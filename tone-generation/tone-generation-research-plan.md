# Tone Generation — Research Plan

> Classify the synthesis engine and predict its static tone-shaping parameters from a sustain-region audio slice. This is the core ML problem in the pipeline — what was previously "engine detection + inverse synthesis," now scoped down because the volume envelope has been stripped by the amplitude expert and modulation is handled by training-time invariance.

## Role in the Horizontal Pipeline

```
flattened audio ─→ [tone-generation expert] ─→ engine type
                                            └─→ engine-specific static param vector
```

This expert classifies the engine and predicts its static tone-shaping parameters. It runs on flattened audio (volume envelope removed by the amplitude expert) **in parallel with the modulation expert** — they consume the same input but answer different questions.

The audio still contains modulations — LFO sweeps, vibrato, wavetable scanning. The model is trained to be **modulation-invariant**: it learns to predict static tone params despite modulations being present, rather than relying on a "de-modulated" signal from upstream. The decision is documented in the modulation plan: filter LFO and wavetable scanning have no clean underlying spectrum to recover (cycle-averaging gives you the *average filtered* spectrum, not the dry spectrum), so surgical removal is misleading. Training-time invariance is more honest and more robust.

## Why This Is Research

The densest of the three problems. It inherits most of the open challenges from the original `inverse-synth` plan (now in `archive/`):

1. **Many-to-one parameter problem.** Multiple parameter combinations produce perceptually identical sounds. There is no unique correct answer.
2. **Engine-conditional output spaces.** FM ratios, drawbar levels, wavetable position, and subtractive filter cutoff are not the same parameter space. The model needs different output heads per engine.
3. **Real-vs-synthetic generalization.** Training data is SignalFlow-rendered synthetic; real-world input is hardware (Prophet-6, Nord, Juno-X via `keyboards-mcp`) plus recording artifacts.
4. **Modulation invariance as a learned property.** The model must ignore time-varying modulations (LFO filter sweeps, wavetable scanning, vibrato) while still extracting static parameters from the same audio. Achieving robust invariance is a non-trivial training problem.

## Phase 1: Sustain-Region Slice Extraction

Given flattened audio + the ADSR estimate from the amplitude expert, choose the input window for the CNN:

- **Sustain-region slice** (200–500 ms from the held part of the note) — the envelope's attack and release are mostly outside this window; default starting point
- **Plus an attack-region slice** — FM transients (bell-like onset) and subtractive velocity-driven filter envelopes carry significant engine-discriminative information that's gone from sustain alone
- **Multi-slice CNN input** — stack sustain and attack slices as a multi-channel input

The slice still contains any LFO-driven movement that was present during sustain. That's intentional — the model is trained to be invariant to it (Phase 2). Don't try to pick a slice that "happens to" miss the LFO; that's brittle.

### Milestone

A slice-extraction module that, given a flattened audio buffer + ADSR estimate (from the amplitude expert), returns the input tensor format consumed by the CNN.

## Phase 2: Modulation Invariance Strategy

The model has to predict static tone parameters from audio that still contains LFO sweeps, vibrato, and wavetable scanning. Two complementary mechanisms make it modulation-invariant:

**Architectural: temporal pooling.** After the convolutional layers, apply global average pooling across the time axis before the prediction heads. This mathematically discards anything that varies periodically over time and keeps what's stable across the slice. It is the architectural equivalent of "look past the modulation." Without this, the CNN's later layers would see time-varying features and have to learn to ignore them; with it, the variation is averaged out before the prediction heads even see it.

**Training: heavy modulation augmentation.** Render each (engine, static-param) combination many times with varied modulation settings — LFO rate / depth / target sampled across the full range, vibrato variants, "no modulation" as a baseline. The model sees that the same static params produce wildly different time-varying spectra and learns the modulations are irrelevant noise. This is the dataset analog of the architectural choice.

Combined, these make invariance a property of both the architecture and the loss landscape — not a runtime audio-processing step.

### Milestone

Invariance demonstrated on a held-out test set: the same static param vector, rendered under N different modulation settings, produces predictions within tight tolerance (e.g., per-param MSE under 0.05 across modulation variants). This metric proves the invariance worked — without it, you don't actually know whether the model is using the modulation as a shortcut.

## Phase 3: Engine Classifier Head

5-class softmax over: subtractive / FM / organ / wavetable / sample-based.

Backbone candidates: small custom CNN (3–5 conv layers), ResNet-18 (pretrained on AudioSet or ImageNet, fine-tuned), MobileNet. The classifier subsumes what was previously the entire `engine-detection` plan — it's now a head on this expert, not a separate phase.

### Milestone

A trained classifier hitting > 85% accuracy on synthetic test data with the upstream pipeline applied. Confusion matrix that identifies which engine pairs are hardest to distinguish.

## Phase 4: Per-Engine Static-Param Heads

For each engine, define a static-only parameter vector (no envelope, no LFO — those belong to other experts):

| Engine | Static params |
|--------|--------------|
| **Subtractive** | osc1 / osc2 shape (discrete), osc2 detune & level, filter cutoff, resonance, env-to-filter amount, noise level |
| **FM** | per-operator frequency ratio, per-operator output level, modulation index, algorithm (discrete) |
| **Organ** | 9 drawbar levels, percussion on / off, percussion harmonic, key click amount |
| **Wavetable** | wavetable position, scan offset, filter cutoff & resonance |
| **Sample-based** | instrument identity (categorical), filter cutoff & resonance, sample brightness / EQ |

Loss: MSE for continuous params + cross-entropy for discrete + summed across active heads. Only the head for the predicted engine is supervised on each example.

### Milestone

Per-engine param prediction with round-trip audio similarity (render predicted params through SignalFlow → mel-spectrogram cosine similarity to input) > 0.85 on a held-out synthetic test set.

## Phase 5: Architecture Decisions

The biggest open question for this expert:

| Option | Pros | Cons |
|--------|------|------|
| **One model per engine** (original inverse-synth plan's approach) | Simple, well-isolated | 5× model count; classifier output gates which model runs |
| **Shared encoder + classifier head + 5 engine-specific param heads** | One model, encoder learns general timbre representations, conditional on classifier | More complex training; head imbalance issues |
| **Joint encoder with the modulation expert** | Maximum sharing across the pipeline; both consume mel spectrograms | Couples two stages; harder to iterate independently |

Recommend benchmarking shared-encoder against fully-independent baselines before committing. Independent is the safe default; sharing is an optimization.

### Milestone

Decision document comparing the three approaches on a fixed evaluation set.

## Phase 6: Evaluation

Inherited from the archived inverse-synth plan, scoped to static params only:

| Metric | Computation |
|--------|-------------|
| Engine classification accuracy | Confusion matrix, per-class F1 |
| Per-param MSE (continuous) | Standard regression metric |
| Per-param accuracy (discrete) | Exact-match for waveform type, FM algorithm, etc. |
| Round-trip audio similarity | Render predicted params → mel spectrogram → cosine sim with input |
| Modulation invariance | Per-param MSE across modulation variants of identical static params (see Phase 2 milestone) |
| Top-K hit rate | Best of K predicted param vectors against ground truth |
| Real-world generalization | Same metrics on hardware recordings via `keyboards-mcp` |

Baselines:

- Random parameter prediction within valid ranges
- Nearest-neighbor in encoder embedding space
- **Pre-pivot baseline:** the archived two-phase vertical pipeline (engine-detection → full inverse-synth) — measure whether the horizontal decomposition actually helps on the same test set

That last baseline is critical. The whole pivot's value rests on the claim that stripping the envelope upstream and learning modulation invariance makes this stage's job easier — that needs to be measured, not assumed.

### Milestone

Quantitative results on all metrics + comparison against the archived vertical baseline. Identification of failure modes per engine.

## Open Questions

- **Is temporal pooling enough, or does the model need explicit modulation conditioning?** An alternative is to feed the modulation expert's predictions as side information ("there's a 4 Hz filter LFO at depth 0.6 — ignore it"). Adds coupling between experts but may improve accuracy. Probably not needed if invariance training works, but worth keeping as a fallback.
- **Shared encoder with the modulation expert** — open architectural question, also flagged in the modulation plan.
- **Sample-based instruments.** Is parameter prediction even meaningful for "Rhodes" or "piano," or should this engine class collapse to instrument identification? Probably the latter for v1.
- **Top-K plausibility.** The many-to-one problem means returning a single param vector is often wrong. Returning top-K with confidence and letting the orchestrating agent pick (or render and audition) is likely better.

(Note: parameter vector → device params, formerly an open question here, is now addressed by the dedicated `parameter-mapping/` stage — this expert emits canonical params, and per-device translation is owned by parameter-mapping.)

## Deliverable

```python
from tone_generation import ToneGenerationModel

model = ToneGenerationModel.load("path/to/checkpoint.pt")
result = model.predict(flattened_audio, top_k=3)
# result = {
#   "engine": "subtractive",
#   "engine_confidence": 0.91,
#   "predictions": [
#     {"confidence": 0.87, "params_canonical": {
#         "osc.1.shape":          "saw",
#         "osc.1.pwm_pct":        44,
#         "osc.2.detune_cents":   7,
#         "osc.2.level":          0.6,
#         "filter.lp.cutoff_hz":  1200,
#         "filter.lp.resonance":  0.6,
#         "noise.level":          0.05,
#     }},
#     {"confidence": 0.71, "params_canonical": {...}},
#     ...
#   ]
# }
```

`params_canonical` keys, units, and enum values conform to the canonical schema defined in `parameter-mapping/` — physical units (Hz, percent, cents), named enums (`saw`, `square`, etc.), no normalized 0–1 floats. Per-engine the canonical schema differs (FM has `op.N.ratio`, organ has `drawbar.N.level`, etc.); the model's training-time normalized 0–1 outputs are denormalized at the deliverable boundary.

Consumed by the audio-analysis-mcp pipeline. Runs in parallel with the modulation expert on the same flattened audio. Canonical engine + static params combine with canonical ADSR (from amplitude expert) and canonical modulation params (from modulation expert) into the unified canonical param vector that flows into `parameter-mapping/` for per-device translation.