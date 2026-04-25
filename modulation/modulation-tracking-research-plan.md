# Modulation Tracking — Research Plan

> Detect time-varying modulation (LFOs, vibrato, filter sweeps, tremolo) from flattened audio. This is a temporal-pattern recognition problem — the right model class is sequence-aware (RNN/LSTM, 1D CNN, transformer), not a static-image CNN.

## Role in the Horizontal Pipeline

```
flattened audio ─→ [modulation expert] ─→ LFO / mod params (rate, depth, target)
```

This expert identifies the low-frequency structure imposed on top of the static timbre — LFOs sweeping the filter, vibrato modulating pitch, tremolo modulating amplitude (anything that survived envelope flattening), wavetable scanning.

It runs **in parallel with the tone-generation expert** on the same flattened audio. Earlier versions of this plan had the modulation expert produce a "de-modulated" audio frame for downstream — that idea was dropped. Filter LFO and wavetable scanning have no clean underlying spectrum to recover (cycle-averaging gives you the average filtered spectrum, not the dry spectrum), so attempted surgical removal is misleading. Instead, the tone-generation expert is trained to be modulation-invariant via temporal pooling and data augmentation; this expert's job is to identify and report what's there, not to clean it up for downstream.

## Why This Is Research

There is no off-the-shelf "LFO detector" for arbitrary audio. Specific challenges:

- **Distinguishing LFO from filter envelope.** Both produce slow filter movement. The discriminator is periodicity, but a single 3-second clip may not contain enough cycles of a 0.5 Hz LFO to confirm periodicity.
- **Multiple simultaneous modulations.** Pitch vibrato + filter LFO + chorus is common in real patches — the model has to disentangle them.
- **Modulation target detection.** Is the LFO modulating amplitude (tremolo), pitch (vibrato), filter cutoff, or wavetable position? Each leaves a different spectral signature.

## Phase 1: Feature Design

Candidate temporal representations (likely a multi-stream input combining several):

| Feature | What it reveals |
|---------|-----------------|
| **Sequence of mel spectrogram frames** | General — what an RNN or 1D CNN consumes by default |
| **Spectral centroid time-series** | Filter sweeps appear as a 1D oscillation |
| **Pitch contour (f0 tracking)** | Vibrato appears as periodic pitch deviation |
| **Per-harmonic amplitude tracking** | Wavetable scanning shows up as harmonic-by-harmonic morphing |
| **RMS time-series** (residual after flattening) | Tremolo / amplitude LFO that survived imperfect flattening |

### Milestone

A standardized feature extraction pipeline producing a multi-channel temporal representation from flattened audio.

## Phase 2: Architecture Exploration

| Architecture | Strength | When to try |
|--------------|----------|-------------|
| **Classical sinusoidal regression** on spectral-centroid time-series | Interpretable, no training, fast baseline | First — often this is enough for periodic LFOs |
| **1D CNN on multi-channel time-series** | Captures local temporal patterns; fast | Strong baseline if classical fails |
| **LSTM / GRU** | Captures long-range periodicity | If LFO rates < 1 Hz are important and classical fails |
| **Small Transformer** | Models phase / period structure well | If recurrent models plateau |

Output heads:

- **LFO rate** (continuous, log-scaled — e.g., 0.1–20 Hz)
- **LFO depth** (continuous)
- **LFO target** (categorical: pitch / filter / amp / wavetable / none)
- **Vibrato rate / depth** (separate from LFO since it's conceptually different — pitch-only modulation)
- **Filter sweep direction** (one-shot vs periodic — this is the LFO-vs-envelope discriminator)

### Milestone

A trained model that, on synthetic data with known modulation params, predicts rate within ±10%, depth within ±0.1, and target with > 90% accuracy.

## Phase 3: Dataset

Render synthetic data using SignalFlow with the cross-cutting setup from CLAUDE.md (Sobol sampling, log-scaled time/frequency, rejection filter). Specific to this expert:

- **Fixed baseline ADSR** — one of the shared envelope presets, identical across the dataset, so the model can't use envelope cues
- **Fixed tone-generation params per engine** — locked to a "neutral" patch per engine; the only thing varying within a class is the modulation
- **Sobol-sampled modulation params** — LFO rate (log 0.1–20 Hz), depth (linear 0–1), target (categorical), vibrato (separate), and a "no modulation" class as negative examples

This isolates the modulation signal in the dataset: the model can't cheat using static timbre.

### Milestone

5,000+ (flattened audio, modulation params) pairs covering all engine types and the "no modulation" class.

## Open Questions

- **3-second clips and slow LFOs.** A 0.5 Hz LFO completes 1.5 cycles in 3 seconds — is that enough to confirm periodicity? May need to require longer input clips, or accept that very slow LFOs will be missed.
- **Filter envelope vs filter LFO disambiguation.** Both produce filter movement; one is one-shot and one is periodic. With short clips, this distinction may be unreliable.
- **Modulation matrix routing.** Real synths route modulation through a matrix (LFO 1 → filter cutoff at depth X, LFO 1 → pitch at depth Y). Can the model recover routing, or only the audible signature?
- **Polyphonic / chord input.** Per-voice modulation tracking on chord-based clips is significantly harder than monophonic. Probably out of initial scope.
- **Shared encoder with tone-generation expert.** Both stages consume mel spectrograms — a shared spectrogram encoder may regularize better than two independent models. Open architectural question, called out also in the tone-generation plan.

## Deliverable

```python
from modulation import ModulationAnalyzer

analyzer = ModulationAnalyzer.load("path/to/checkpoint.pt")
result = analyzer.analyze(flattened_audio)
# result = {
#   "modulations_canonical": {
#     "lfo.1.rate_hz":  4.2,
#     "lfo.1.depth":    0.6,
#     "lfo.1.target":   "filter.cutoff",
#     "vibrato.rate_hz": 5.1,
#     "vibrato.depth":   0.05,
#   },
# }
```

`modulations_canonical` keys, units, and target vocabulary conform to the canonical schema defined in `parameter-mapping/`. The `lfo.N.target` field uses the canonical destination vocabulary (e.g., `filter.cutoff`, `osc.1.pitch`, `osc.1.amp`); device translators subset it per device.

Consumed by the audio-analysis-mcp pipeline. Runs in parallel with the tone-generation expert on the same flattened audio. Canonical modulation params flow into the unified canonical param vector; the tone-generation expert ignores modulations through training-time invariance, not by consuming a de-modulated signal from this stage.