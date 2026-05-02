# Tone Generation — Research Plan

> Classify the synthesis engine and predict its static tone-shaping parameters from a sustain-region audio slice. This is the core ML problem in the pipeline — what was previously "engine detection + inverse synthesis," now scoped down because the volume envelope has been stripped by the amplitude expert and modulation is handled by training-time invariance.

## Role in the Horizontal Pipeline

```
flattened audio ─→ [tone-generation expert] ─→ engine type
                                            └─→ engine-specific static param vector
                                               (osc / noise / filter / voice / master
                                                + filter envelope)
```

This expert classifies the engine and predicts its static tone-shaping parameters. It runs on flattened audio (volume envelope removed by the amplitude expert) **in parallel with the modulation expert** — they consume the same input but answer different questions.

In the canonical schema (see [`parameter-mapping/subtractive-ontology.md`](../parameter-mapping/subtractive-ontology.md)), this expert owns `params.osc`, `params.noise`, `params.filter`, `params.voice`, `params.master`, and `params.envelope.filter`. The amplitude expert fills `params.envelope.amp`; the modulation expert fills `params.lfo.*`. Notably the **filter envelope** (`envelope.filter` ADSR + `filter.lp.envelope_amount`) belongs here, not to amplitude — it is a static tone-shaping behavior that fires identically per note, while the amplitude expert is a DSP volume-envelope follower that produces only the amp envelope.

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

For each engine, define a static-only parameter vector. "Static" means: not the amp envelope (amplitude expert) and not LFO (modulation expert). The filter envelope counts as static here — it is fixed per-program and fires identically per note.

The output of each head is a **canonical-schema-conforming instance** of the engine's `params` shape. Names, units, and structure come from the engine's schema in `parameter-mapping/`.

### Subtractive — schema-grounded (v0.1)

The full target output, per [`subtractive.schema.json`](../parameter-mapping/subtractive.schema.json) and the [ontology doc](../parameter-mapping/subtractive-ontology.md):

| Section | Fields | Required? |
|---|---|---|
| `osc.1` | `shape`, `level`, `detune_cents`, `octave`, `pulse_width_pct`* | required |
| `osc.2` | same five | section optional |
| `osc.sub` | `octave`, `level` | section optional |
| `noise` | `color`, `level` | section optional |
| `filter.lp` | `cutoff_hz`, `resonance`, `envelope_amount`, `key_tracking`, `drive` | required |
| `envelope.filter` | `attack_ms`, `decay_ms`, `sustain`, `release_ms` (ADSR) | optional |
| `voice` | `mode` (mono/poly/unison), `glide_ms`, `unison_voices` | required (`mode`); rest optional |
| `master` | `volume_db` | section optional |

\* `pulse_width_pct` is only meaningful when `shape == 'pulse'`.

Two ML design points the schema's shape forces:

- **Optional sections.** A subtractive instance can omit `osc.2`, `osc.sub`, `noise`, `master` entirely. The head predicts a presence-gate per optional section plus the section's contents; loss for an absent section's contents is masked.
- **Conditional fields.** `osc.{1,2}.pulse_width_pct` is only meaningful when the same oscillator's `shape == 'pulse'`. Either always predict it and ignore at inference when shape mismatches, or use a shape-conditioned masked loss. Same pattern as the optional sections, narrower scope.

### Other engines (FM / organ / wavetable / sample-based)

Currently blocked on parameter-mapping work — only subtractive has a v0.1 canonical schema. The fields below are the **research-plan target** (what the head will eventually predict), not a finalized contract:

| Engine | Static params (target) | Schema status |
|--------|--------------|---|
| **FM** | per-operator frequency ratio, per-operator output level, modulation index, algorithm (discrete) | not yet in `parameter-mapping/` |
| **Organ** | 9 drawbar levels, percussion on / off, percussion harmonic, key click amount | not yet in `parameter-mapping/` |
| **Wavetable** | wavetable position, scan offset, filter cutoff & resonance | not yet in `parameter-mapping/` |
| **Sample-based** | instrument identity (categorical), filter cutoff & resonance, sample brightness / EQ | not yet in `parameter-mapping/` |

These heads should not be trained until their respective ontology schemas exist — parameter names and structure are subject to change. Subtractive ships first, against `subtractive.schema.json` v0.1. Others unblock when their schemas land.

Loss: MSE for continuous params + cross-entropy for discrete + masked across absent optional sections + summed across active heads. Only the head for the predicted engine is supervised on each example.

### Milestone

Per-engine param prediction with round-trip audio similarity (render predicted canonical params through SignalFlow → mel-spectrogram cosine similarity to input) > 0.85 on a held-out synthetic test set. Subtractive must additionally produce schema-valid instances (every prediction validates against `subtractive.schema.json`).

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
- **Synthetic dataset coverage vs schema common-core.** SignalFlow can render features that the canonical schema deliberately omits as out-of-common-core (cross-modulation, hard sync, ring mod, multi-mode filters, multi-LFO, third envelope — see the "What is not here" section of the subtractive ontology). Two options: (a) render only common-core features so labels match the schema directly, or (b) render the full feature space and clip labels to common-core, accepting the model may see audio whose "ground truth" can't be expressed canonically. Probably (a) for v0.1 — keeps labels honest; revisit when the schema grows.
- **Schema-conformance as a training signal.** Subtractive predictions should validate against `subtractive.schema.json`. Either enforce structurally (architecture mirrors the schema with presence-gates per optional section), or post-hoc (mask invalid combinations at inference). Architectural enforcement is cleaner; post-hoc is faster to iterate. Decide when implementation starts.

(Note: parameter vector → device params, formerly an open question here, is now addressed by the dedicated `parameter-mapping/` stage — this expert emits canonical params, and per-device translation is owned by parameter-mapping.)

## Deliverable

Each prediction is a partial canonical instance (this expert's slice — osc / noise / filter / voice / master / `envelope.filter`). The amplitude expert fills `envelope.amp`; the modulation expert fills `lfo.*`. The pipeline merges them into one fully-formed canonical instance before handoff to `parameter-mapping/`.

```python
from tone_generation import ToneGenerationModel

model = ToneGenerationModel.load("path/to/checkpoint.pt")
result = model.predict(flattened_audio, top_k=3)
# result = {
#   "engine": "subtractive",
#   "engine_confidence": 0.91,
#   "schema_version": "0.1",
#   "predictions": [
#     {
#       "confidence": 0.87,
#       "params_partial": {
#         "osc": {
#           "1": {"shape": "saw",   "level": 0.85, "octave": 0,  "detune_cents": 0},
#           "2": {"shape": "pulse", "level": 0.6,  "octave": 0,  "detune_cents": 7,
#                  "pulse_width_pct": 44}
#         },
#         "noise": {"color": "pink", "level": 0.05},
#         "filter": {
#           "lp": {
#             "cutoff_hz": 1200, "resonance": 0.6,
#             "envelope_amount": 0.4, "key_tracking": 0.3, "drive": 0.0
#           }
#         },
#         "envelope": {
#           "filter": {"attack_ms": 5, "decay_ms": 240, "sustain": 0.3, "release_ms": 180}
#         },
#         "voice":  {"mode": "poly"}
#       }
#     },
#     {"confidence": 0.71, "params_partial": {...}},
#     ...
#   ]
# }
```

### Canonical Output Contract

- **Schema source:** [`parameter-mapping/subtractive.schema.json`](../parameter-mapping/subtractive.schema.json) (extends [`synth-base.schema.json`](../parameter-mapping/synth-base.schema.json)). Schema version `0.1`.
- **Structure:** nested objects per the schema, not a flat dot-keyed dict. `params.osc.1.shape`, not `"osc.1.shape"`.
- **Names:** exact canonical names — e.g. `pulse_width_pct` (not `pwm_pct`), `detune_cents` (not `detune`), `cutoff_hz` (not `cutoff`).
- **Units:** physical — Hz, ms, cents, percent (0–100), dB; ratios are 0–1 (signed −1..1 for `filter.lp.envelope_amount`). No 0–127 MIDI scaling, no raw 0–1 stand-ins for parameters that have a physical unit. The model's training-time normalized outputs are denormalized at the deliverable boundary.
- **Enums:** named values — `saw` / `square` / `triangle` / `pulse` / `sine` for `osc.shape`, `white` / `pink` for `noise.color`, `mono` / `poly` / `unison` for `voice.mode`.
- **Optional sections:** `osc.2`, `osc.sub`, `noise`, `master`, `envelope.filter` may be omitted entirely; the head's per-section presence-gate decides.
- **Validation:** every emitted prediction must validate against `subtractive.schema.json` after merge with the other experts' outputs — this is a CI gate, not a stretch goal.

Per-engine the canonical schema differs (FM will have `op.N.ratio`, organ `drawbar.N.level`, etc.); only subtractive is formalized in v0.1 — see Phase 4 for the others' status.

Consumed by the audio-analysis-mcp pipeline. Runs in parallel with the modulation expert on the same flattened audio. Canonical engine + static params combine with canonical ADSR (from amplitude expert) and canonical modulation params (from modulation expert) into the unified canonical param vector that flows into `parameter-mapping/` for per-device translation.