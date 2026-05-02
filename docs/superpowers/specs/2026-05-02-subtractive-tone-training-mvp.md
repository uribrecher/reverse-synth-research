---
mode: mvp
parent_topic: subtractive-tone-training
backlog: ./2026-05-02-subtractive-tone-training-backlog.md
---

# Subtractive Tone-Generation Model — Training Pipeline MVP

> **Scope discipline:** This spec is intentionally a thin slice. Deferred features live in the backlog file linked above. `writing-plans` should plan ONLY what is in this spec — do not pull from the backlog.

## What this slice is

A single PyTorch training script that takes synthetic subtractive-engine renders, conditioned on ground-truth played pitches, and learns to predict three canonical synth params (oscillator shape, filter cutoff, filter resonance) from a sustain-region log-mel spectrogram. The MVP closes the full loop end-to-end — SignalFlow renderer → dataset generator → mel preprocessing → conditioned CNN → per-param eval + round-trip mel cosine — on a deliberately reduced parameter space.

## Why this cut

Three free parameters is the smallest set that exercises both prediction types the full model will need (classification + regression), so the architecture is meaningfully validated. Pitch + polyphony enter as conditioning inputs (ground-truth from the renderer), which removes pitch-invariance from the model's learning burden and matches how the deployed pipeline will work (`note_transcribe` → tone-generation). Fixed amp ADSR / inert filter envelope / no LFO removes every confounder the other experts will eventually own, so failures are unambiguously this expert's. The output emits a schema-conformant `subtractive.schema.json` instance from day one, exercising the contract with `parameter-mapping/` even on this small slice.

## In scope

### Renderer

- **Engine:** SignalFlow node graph. Single oscillator → low-pass filter → fixed amp ADSR → output. No noise, no sub-osc, no `osc.2`, no LFO. `filter.lp.envelope_amount = 0` (filter envelope inert; modulation expert's territory anyway).
- **Free synth parameters (per render):**
  - `osc.1.shape` ∈ `{sine, saw, square, triangle}` — discrete, uniform
  - `filter.lp.cutoff_hz` ∈ [50, 10000] Hz — continuous, log-distributed
  - `filter.lp.resonance` ∈ [0, 1] — continuous, uniform
- **Frozen synth parameters:** `voice.mode = 'poly'`, `osc.1.level = 1.0`, `osc.1.detune_cents = 0`, no `osc.2` / `osc.sub` / `noise` / `master` sections, fixed amp ADSR baseline preset (single "pad"-style preset, exact values pinned in `schema_io.py`).
- **Note rendering:**
  - Per render: sample `n_voices ∈ {1, 2, 3}` uniformly.
  - Sample `n_voices` distinct MIDI pitches from [36, 84] (C2–C6); for `n_voices > 1`, all pitches must lie within a 12-semitone window (musical chord-density default).
  - Render each voice with the same synth params; sum the buffers; normalize to peak amplitude ~0.95.
  - Total render duration: 1.2 s (100 ms attack + ≥ 800 ms sustain + 100 ms release; pinned in renderer config).
  - Sample rate 44.1 kHz, mono, float32.
- **Sampling strategy:**
  - Continuous synth params (`cutoff_hz`, `resonance`): Sobol via `scipy.stats.qmc`. `cutoff_hz` is log-distributed by sampling its log uniformly in [log 50, log 10000] via Sobol then exponentiating.
  - Discrete params (`shape`, `n_voices`): independent uniform integer.
  - Pitches: sampled independently per voice slot, uniform integer over MIDI [36, 84], rejecting samples that violate the 12-semitone window constraint.

### Dataset generator

- **Output layout:** one directory per dataset:
  - `samples/{idx:06d}.wav` — rendered audio, 16-bit PCM, 44.1 kHz, mono
  - `labels.jsonl` — one JSON object per line: `{idx, params_canonical, midi_pitches, n_voices}` where `params_canonical` is the schema-conformant nested instance
  - `manifest.json` — dataset metadata (sample count, schema version, master seed, rendering config, renderer git SHA)
- **Size:** 10,000 samples for MVP (8 K train / 1 K val / 1 K test by `idx % 10`).
- **Determinism:** seeded Sobol + seeded pitch / shape / n_voices sampler. Same master seed → byte-identical dataset.
- **Location:** dataset path configurable via CLI flag. Default `audio-analysis-mcp/scratch/tone_gen_dataset/` (gitignored).

### Preprocessing (online, in the training Dataset)

- **Sustain slice:** fixed 300 ms window starting 100 ms after note-on (skips attack, lands solidly mid-sustain).
- **Mel-spectrogram:** log-mel, 128 mel bins, 25 ms window, 10 ms hop, output shape `(128, ≈30)` for the 300 ms slice. Computed on the fly per batch (not pre-computed).
- **Pitch encoding:** 88-dim multi-hot vector over MIDI 21–108. `n_voices` is implicit as popcount; not passed separately.
- **Param normalization (training-time only):**
  - `cutoff_hz` → `cutoff_norm` ∈ [0, 1] via `(log(cutoff_hz) - log(50)) / (log(10000) - log(50))`
  - `resonance` ∈ [0, 1] passes through
  - `shape` → integer label 0–3 (sine=0, saw=1, square=2, triangle=3) for cross-entropy
  - Denormalized at inference back to canonical units.

### Model

- **Architecture:** small custom CNN.
  - Input: mel-spec `(1, 128, ~30)`
  - 3 conv blocks: Conv2d 3×3 → BatchNorm → ReLU → MaxPool 2×2, channels 32 → 64 → 128
  - Global average pool over the time axis only (modulation-invariance principle from cross-cutting setup), then spatial flatten
  - Concat with 88-dim pitch multi-hot
  - 2-layer MLP head splitting into:
    - `shape_logits` — 4-dim (cross-entropy)
    - `cutoff_norm` — 1-dim, sigmoid-clamped to [0, 1]
    - `resonance` — 1-dim, sigmoid-clamped to [0, 1]
- **Parameter count:** target ~few hundred K. CPU-trainable in < 30 min on a developer machine.
- No pretraining, no transfer learning.

### Training loop

- **Framework:** PyTorch. Single-process, auto-detect device — prefer `mps` (Apple Silicon GPU via Metal) on the developer's M3 Pro, fall back to `cuda` if available, otherwise `cpu`. Determinism flags (`torch.use_deterministic_algorithms`) are best-effort under MPS — set them but do not gate on full reproducibility.
- **Loss:** unweighted sum of cross-entropy(shape) + MSE(cutoff_norm) + MSE(resonance).
- **Optimizer:** Adam, `lr=1e-3`, `weight_decay=1e-4`. Hard-coded; no scheduler.
- **Batch size:** 64.
- **Epochs:** 50 with early stop on val-loss plateau (patience=5).
- **Determinism:** seeded torch + numpy. CuDNN deterministic mode if GPU.
- **Logging:** stdout only — per-epoch train loss, val loss, val per-param metrics. No tensorboard / wandb.

### Eval harness

- **Per-param metrics on test set:**
  - `osc.1.shape`: top-1 accuracy + 4×4 confusion matrix
  - `filter.lp.cutoff_hz`: MSE in log-Hz + median relative error in linear Hz
  - `filter.lp.resonance`: MSE
- **Round-trip metric:** for each test sample, render predicted canonical params through the same SignalFlow voice (using the same `midi_pitches` + `n_voices`), compute mel-spec, mel-cosine vs input. Report mean and median.
- **Schema-validation gate:** every emitted prediction's denormalized canonical instance must validate against `subtractive.schema.json`. Eval reports a count of failures; > 0 is an error.
- **Output:** `eval_report.json` with full numbers + a one-screen `eval_summary.txt` to stdout.

### Deliverables

1. **Dataset generator script:** `scripts/generate_subtractive_dataset.py` — renders + writes `samples/` + `labels.jsonl` + `manifest.json`. CLI: `--n-samples`, `--seed`, `--out-dir`.
2. **Training script:** `scripts/train_tone_generation.py` — reads dataset, trains, writes `checkpoint.pt` + `eval_report.json`. CLI: `--dataset-dir`, `--checkpoint-out`, `--seed`.
3. **Eval-only script:** `scripts/eval_tone_generation.py` — reloads checkpoint + dataset, re-runs eval. CLI: `--checkpoint`, `--dataset-dir`.
4. **Module code:** `src/audio_analysis_mcp/research/tone_generation/` package containing `renderer.py`, `dataset.py`, `model.py`, `schema_io.py`, plus a short `README.md` documenting the workflow.

## Out of scope (see backlog)

- `osc.2`, `osc.sub`, `noise`, `master` optional sections + per-section presence-gates
- `osc.1.pulse_width_pct` + the conditional-field problem
- `osc.1.detune_cents`, `osc.1.level` prediction (level fixed at 1.0)
- `osc.1.octave` (degenerate with single oscillator)
- `filter.lp.envelope_amount`, `key_tracking`, `drive`
- `voice.mode` / `glide_ms` / `unison_voices` prediction
- `master.volume_db` prediction
- Modulation invariance training (LFO + vibrato in data)
- Engine classifier head (5-class softmax)
- Attack-region multi-slice input
- Top-K predictions with per-slot confidence
- Variable amp envelope (sample from preset bank per render)
- Velocity variation
- Pitch range expansion beyond C2–C6
- Wider polyphony (4+ voices, denser chord textures)
- Real-keyboard recordings via `keyboards-mcp` (synthetic→real gap)
- Running `note_transcribe` at dataset time + transcription-noise augmentation
- Validating `note_transcribe` accuracy on synthetic subtractive renders
- Richer pitch encoding (pitch-class histogram + sorted-pitch sequence) or FiLM conditioning
- Schema-validation by construction (architecture mirrors schema with presence-gates)
- Pre-pivot baseline (vs archived two-phase vertical pipeline)
- Shared encoder with the modulation expert
- Larger backbones (ResNet-18 pretrained, MobileNet)
- Rejection sampling (silence / DC click / clipping)
- Hyperparameter sweeps + dataset-size scaling
- Loss weighting / scheduling
- Pre-computed mel-spec cache to disk
- Fix `tone-generation-research-plan.md` to drop `envelope.filter` from this expert's ownership (modulation expert owns it per latest direction)

## Architecture

```
Dataset generation (offline, one-shot)
─────────────────────────────────────
  Sobol sampler ─┐
  uniform RNG ───┴──► params_canonical, midi_pitches[], n_voices
                              │
                              ▼
                     SignalFlow node graph
                  (osc → LP → fixed amp ADSR → ×N voices summed)
                              │
                              ▼
                  audio buffer (float32, 44.1 kHz)
                              │
                              ▼
                samples/{idx}.wav  +  labels.jsonl entry

Training (PyTorch, online preprocessing)
────────────────────────────────────────
  labels.jsonl ─► Dataset class ─► DataLoader
                                        │
                                        ▼
                       audio + pitches + params
                                        │
                ┌───────────────────────┼─────────┐
                ▼                       ▼         ▼
         sustain slice           pitch multi-hot  target params
                │                       │              │
                ▼                       │              │
          log-mel spec                  │              │
                │                       │              │
                └────► CNN encoder ◄────┘              │
                            │                          │
                global avg pool over time              │
                            │                          │
                     concat with pitches               │
                            │                          │
                        MLP heads                      │
                            │                          │
            [shape_logits, cutoff_norm, resonance] ────┴──► loss
                            │
                            ▼
                checkpoint.pt + per-epoch metrics

Eval (loads checkpoint + dataset)
─────────────────────────────────
  test samples ─► model ─► predictions
                       │
                       ▼
              denormalize → schema-conformant params_canonical
                       │
              ┌────────┴────────┐
              ▼                 ▼
        per-param metrics    re-render via SignalFlow
                                (same pitches, same n_voices)
                                   │
                                   ▼
                           mel-cosine vs input
                                   │
                                   ▼
                            eval_report.json
```

## Components

### `renderer.py`

- `class SubtractiveRenderer`: builds a SignalFlow `AudioGraph` per render call, renders to `Buffer`, returns `np.ndarray` (float32, mono).
- `def render_chord(params_canonical, midi_pitches, sample_rate, total_duration_s) -> np.ndarray`: builds N parallel voices in one graph, sums them, normalizes peak. Single-voice is just `n_voices=1`.
- Reuses the SignalFlow API patterns proven in `audio-analysis-mcp/scratch/explore_signalflow.py` (`AudioGraph`, `ADSREnvelope`, `render_to_new_buffer`).

### `dataset.py`

- `def sample_dataset_config(n_samples, seed) -> Iterable[DatasetItem]`: Sobol over `(cutoff_hz_log, resonance)` + uniform integer over `(shape, n_voices, pitches)`. Returns dataclass items with `params_canonical`, `midi_pitches`, `n_voices`.
- `def build_canonical_instance(shape, cutoff_hz, resonance) -> dict`: builds the `subtractive.schema.json`-shaped nested dict including the frozen `voice.mode` and frozen amp ADSR.
- `class ToneGenerationDataset(torch.utils.data.Dataset)`: reads `labels.jsonl` + `samples/{idx}.wav`, returns `(mel_spec, pitch_multihot, target_dict)` per item.

### `model.py`

- `class ToneGenerationCNN(nn.Module)`: 3-block CNN + temporal pool + concat + MLP heads. `forward(mel_spec, pitch_multihot)` returns `dict[shape_logits, cutoff_norm, resonance]`.

### `schema_io.py`

- `BASELINE_AMP_ADSR`: dict literal of the frozen amp ADSR preset.
- `def normalize_params(params_canonical) -> dict[str, Tensor]`: canonical → training-time normalized tensors.
- `def denormalize_predictions(preds) -> dict`: predictions → schema-conformant canonical instance.
- `def validate_canonical(instance) -> None`: JSON Schema validation against `subtractive.schema.json`. Schema loaded once and cached.
- Schema is loaded from `reverse-synth-research/parameter-mapping/subtractive.schema.json` via a path resolved at import time (env var override for non-monorepo layouts).

## Error handling

- **Renderer build failures** (SignalFlow node graph fails, unsupported param value): fail loudly with the offending params. No silent fallbacks.
- **Schema-validation failures during eval:** each invalid prediction logged with offending instance + validator error. Eval reports a count.
- **NaN / Inf / DC-only audio in dataset generation:** fail loud, do not skip. (Rejection sampling is a backlog item.)
- **Out-of-range Sobol samples:** assert in the sampler. Should not happen given fixed bounds.
- **Dataset integrity:** training script verifies `len(labels.jsonl) == manifest.n_samples` and that every referenced WAV exists.

## Testing

- **Unit tests** (fast, run by default in `pytest`):
  - `test_renderer.py`: render a saw at A4 with `cutoff_hz=2000`, `resonance=0.5`. Assert audio length matches `total_duration_s`, peak amplitude in [0.1, 1.0], no NaN / Inf.
  - `test_dataset.py`: `build_canonical_instance` produces schema-valid output for representative param tuples; `normalize_params` ↔ `denormalize_predictions` is round-trip stable for in-range inputs.
  - `test_model.py`: forward pass on dummy `(mel_spec, pitch_multihot)` returns expected output shapes / dtypes.
- **Integration tests** (slow, marked `@pytest.mark.slow`, opt-in):
  - Generate a 100-sample dataset, train for 5 epochs, assert val loss decreases below a generous threshold. Smoke test the loop, not a quality assertion.
- **No CI gate on model accuracy.** MVP success is "loop runs end-to-end with sane numbers," not "model hits target accuracy."
- **Schema validation:** every test that emits a canonical instance asserts validation against `subtractive.schema.json`.

## Definition of done

- Dataset generator produces 10 K samples + valid `labels.jsonl` + `manifest.json` from a fresh run.
- Training script runs to completion on the 10 K dataset on the developer machine in < 30 min CPU / < 5 min GPU.
- Eval harness produces `eval_report.json` with all metrics; schema-validation pass count equals test-set size.
- Unit tests pass; the slow integration test passes on a 100-sample mini dataset.
- A short `README.md` in `research/tone_generation/` explains how to regenerate the dataset, train, and re-eval.

## Implementation notes (for writing-plans)

- **Module location:** `audio-analysis-mcp/src/audio_analysis_mcp/research/tone_generation/`. Create the `research/` subpackage if it does not already exist.
- **Schema source of truth:** read `subtractive.schema.json` from `reverse-synth-research/parameter-mapping/`. Do not duplicate the schema file. Resolve path via env var with a sane default (this monorepo's relative layout).
- **Pre-implementation scratch (mandatory):** before dispatching the implementer for the renderer, write `audio-analysis-mcp/scratch/explore_subtractive_renderer.py` that exercises the SignalFlow polyphonic-sum-of-voices pattern end-to-end and confirms the audio looks right (peak amplitude reasonable, expected harmonics present in spectrum, no clipping after sum). The existing `scratch/explore_signalflow.py` only covers monophonic ADSR — polyphonic summation needs its own scratch verification per the project's "scratch first" rule.
- **PyTorch dependency:** add `torch` to `audio-analysis-mcp/pyproject.toml` if not already present. The standard macOS wheel includes MPS support — no special build required.
- **No `keyboards-mcp` integration in MVP.** Purely synthetic data + offline training.
