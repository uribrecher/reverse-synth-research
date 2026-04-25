# Amplitude Envelope — Research Plan

> Extract the amplitude envelope (ADSR estimate) from audio and produce a "flattened" signal for downstream stages. This is mostly DSP, not deep learning — the research questions are narrow and concern envelope representation, fitting, and how flattening interacts with downstream models.

## Role in the Horizontal Pipeline

```
audio ─→ [amplitude expert] ─→ ADSR estimate
                            └─→ flattened audio ─→ [modulation] → [tone-generation]
```

This is the outermost layer of the onion. Two responsibilities:

1. Estimate the volume contour and fit it to an ADSR-style description.
2. Produce a normalized/flattened audio stream so the modulation and tone-generation experts don't have to learn around volume changes.

Removing the amplitude envelope before downstream analysis is a deliberate architectural choice: ADSR is a confounding variable for engine classification (it shapes the audio but tells you nothing about the engine), and a CNN tasked with finding an attack time is wildly overpowered for the job.

## Why This Is Mostly Not Research

Amplitude envelope estimation is well-understood DSP — RMS-with-windowing, Hilbert envelope, peak follower. The narrow research questions:

1. **What flattening operation actually helps downstream?** Naive `audio / envelope` blows up noise in silent regions. Soft-knee normalization, floor-clipped division, and frequency-domain spectral whitening are all candidates and may behave very differently for the downstream models.
2. **Handling "natural" envelopes.** Acoustic / sample-based instruments (piano, Rhodes, Wurlitzer) have engine-intrinsic decay that isn't separable from the timbre. Flattening these may destroy the very feature that identifies them.
3. **ADSR fitting from a measured envelope curve.** Real envelope curves don't always conform to a four-segment piecewise-exponential model — multi-stage envelopes, slow attack with sharp peak, etc.

## Phase 1: Envelope Extraction Algorithms

Implement and compare:

| Method | Strength | Weakness |
|--------|----------|----------|
| **RMS over sliding window** (10–50 ms) | Smooth, robust to noise | Smears fast transients |
| **Hilbert envelope** | Captures fast attacks accurately | Noise-sensitive |
| **Peak follower with attack/release time constants** | Closer to what hardware envelope detectors do | Tunable — hyperparameters matter |

Evaluation: SignalFlow-rendered synthetic patches with fixed tone params and known ADSR — measure reconstruction error of each algorithm against ground truth.

### Milestone

A Python module that, given a WAV file, returns an envelope curve at audio rate. Side-by-side comparison of the three methods on a synthetic ADSR test set.

## Phase 2: ADSR Fitting

Given an envelope curve, fit:

- Attack time (silence → peak)
- Decay time (peak → sustain level)
- Sustain level (held value during note)
- Release time (note-off → silence)

Approaches:

- **Heuristic segmentation** — find peak, detect sustain region by variance, fit exponentials per segment
- **Least-squares fit** to a four-segment piecewise-exponential model
- **Lightweight regression model** (small MLP on envelope features) if heuristics are not robust enough

### Milestone

Per-clip ADSR estimate with reasonable accuracy (e.g., attack within ±10 ms, sustain within ±0.05) on synthetic data.

## Phase 3: Audio Flattening

Produce a "volume-normalized" version of the audio for the modulation and tone-generation experts. Candidate approaches:

- **Envelope division with floor** — `audio / max(envelope, ε)` where ε prevents noise blow-up in tails
- **Soft-knee compression** with very fast attack/release — approximates flattening without division artifacts
- **Spectral whitening** — frequency-domain normalization, conceptually different from time-domain envelope removal; may be more useful to downstream models that consume mel spectrograms anyway

The downstream stages should be trained and evaluated on the flattening method that wins on round-trip metrics — i.e., does the modulation expert detect LFOs more accurately on flattened-by-method-A vs method-B audio?

### Milestone

A flattening method selected based on how well it improves downstream metrics, not on how clean the flattened audio sounds in isolation.

## Open Questions

- Does spectral whitening replace time-domain envelope-based flattening for downstream models?
- For acoustic / sample-based instruments, the "envelope" includes engine-specific decay characteristics (piano string decay, Rhodes tine ring-out). Should we even attempt to flatten these, or pass the original audio through to the tone-generation expert when this engine class is suspected?
- Filter envelopes (which don't show up as volume changes) are explicitly out of scope here — they belong to the modulation or tone-generation expert.
- Polyphonic input: a chord with staggered note-onsets has overlapping envelopes. Is per-note onset detection needed before envelope extraction, or can the system operate on the aggregate?

## Deliverable

A Python module exposing:

```python
from amplitude import AmplitudeAnalyzer

analyzer = AmplitudeAnalyzer()
result = analyzer.analyze("path/to/audio.wav")
# result = {
#   "envelope_canonical": {
#     "envelope.amp.attack_ms":  12,
#     "envelope.amp.decay_ms":   380,
#     "envelope.amp.sustain":    0.62,
#     "envelope.amp.release_ms": 220,
#   },
#   "envelope_curve":  np.ndarray,  # raw envelope at audio rate (debugging / validation)
#   "flattened_audio": np.ndarray,  # normalized audio for downstream stages
# }
```

`envelope_canonical` keys and units conform to the canonical schema defined in `parameter-mapping/` — physical units (ms), named hierarchical keys, no normalized 0–1 floats. Consumed by the audio-analysis-mcp pipeline as the first stage of analysis. Canonical envelope params flow into the unified canonical param vector; flattened audio flows into the modulation and tone-generation experts.