---
mode: backlog
parent_topic: subtractive-tone-training
mvp_spec: ./2026-05-02-subtractive-tone-training-mvp.md
---

# Subtractive Tone-Generation Training — Deferred Backlog

Features identified during MVP brainstorm but not in the first slice. Each entry is a stub — when picked up, run `superpowers:brainstorming` (or `mvp-brainstorm` again if it is still big) on it as a fresh topic.

## Schema-coverage expansion

### `osc.2` — second oscillator section + presence-gate

Optional in `subtractive.schema.json`. Adding it unblocks `osc.{1,2}.octave` (which is degenerate with a single oscillator — see MVP spec) and approximately doubles the timbre space the model can predict. Requires the model to learn a per-section presence-gate: "is osc.2 present at all, and if so, what are its params?" — masked loss for the contents when the gate predicts absent. Bring in alongside `osc.1.octave` and `osc.1.detune_cents` so octave-stacking and detuning textures become learnable together.

### `osc.sub` — sub-oscillator + presence-gate

Optional. `octave ∈ {-2, -1}` (enum) and `level ∈ [0, 1]`. Same presence-gate pattern as `osc.2`. Smaller scope than `osc.2` — could ship in a smaller follow-up.

### `noise` — generator section + presence-gate

Optional. `color ∈ {white, pink}` + `level`. Mostly orthogonal to the rest of the spectrum and probably easy to detect; defer until osc.2 / sub-osc are working since those are higher-priority.

### `master.volume_db`

Optional section, single param. Output-stage gain. Hardest of the deferred params to predict because peak normalization in the renderer (and any analogous normalization in real audio) destroys most of the signal. Probably needs the renderer to skip peak normalization for this to be learnable.

### `osc.1.pulse_width_pct` + the conditional-field problem

Schema makes `pulse_width_pct` only meaningful when `shape == 'pulse'`. Two options when picking this up: (a) always predict it and mask the loss when `shape != 'pulse'` at inference, or (b) train a separate head that fires only conditioned on the shape prediction. Decide via a small ablation. Adding `pulse` to the shape enum at the same time gives the head four meaningful target shapes plus pulse, instead of skipping pulse entirely as the MVP does.

### `osc.1.detune_cents`, `osc.1.level`

Both are continuous regression. `level` is fixed at 1.0 in MVP because there's only one oscillator (level vs master volume is unidentifiable). Becomes meaningful once `osc.2` ships and the model has to predict the relative balance.

### `filter.lp.envelope_amount`, `key_tracking`, `drive`

`envelope_amount` is the connection between the filter envelope and the cutoff — but the filter envelope itself is owned by the modulation expert per the latest direction. Once the modulation expert ships filter-envelope prediction, this expert needs to predict the coupling parameter on top of that. `key_tracking` and `drive` are subtler effects that only matter once the simpler params are nailed.

### `voice.mode` / `glide_ms` / `unison_voices` prediction

Currently frozen at `voice.mode = 'poly'`. Predicting `mode` requires renders that include `mono` and `unison` modes, with the model learning to distinguish them from chord/no-chord audio + glide artifacts. `unison_voices` is a count that's hard to recover from audio (multiple detuned voices look like one slightly chorused voice). Probably ships together with detune and `osc.2`.

## Modulation handling

### Modulation invariance training (LFO + vibrato in dataset)

The full plan calls for training-time augmentation that varies LFO rate / depth / target across the same static-param point so the model learns to predict static params despite modulation. The MVP renders without LFO at all — every sample is "no modulation." First step here is to add LFO into the renderer (already in the canonical schema) and confirm the model still recovers static params under the augmentation. The architectural temporal-pool is already in the MVP CNN; it just needs the data augmentation to validate against.

### Filter-envelope coupling

Once the modulation expert ships filter-envelope prediction, this expert has to predict `filter.lp.envelope_amount` (the depth coupling) as a static param. The renderer needs to render with non-zero filter envelopes during dataset generation. Coordinate handoff with the modulation expert's plan when it becomes concrete.

## Architecture variations

### Engine classifier head (5-class softmax)

The full plan has a top-level engine classifier (subtractive / FM / organ / wavetable / sample-based) gating which per-engine head fires. MVP is subtractive-only. Picking up this item is gated on at least one more engine schema landing in `parameter-mapping/` (FM is the natural next target).

### Attack-region multi-slice input

The current MVP uses a single 300 ms sustain slice. The full plan stacks an attack-region slice as a second channel into the CNN — captures FM transients (when FM ships) and the spectral evolution that the filter envelope produces. Modest architectural change; defer until `osc.2` and the optional sections are working so we know we need more discriminative signal.

### Top-K predictions with per-slot confidence

The many-to-one parameter problem means a single predicted vector is often "wrong" even when other plausible vectors would also reproduce the audio. Returning top-K with confidence and letting the orchestrating agent pick (or render and audition) is likely better than argmax. Requires changing the heads to emit distributions or sampling-from-Gaussian regression heads.

### Schema-validation by construction

The MVP uses post-hoc schema validation: emit predictions, denormalize, validate, count failures. Stricter alternative: design the model architecture so it can only emit schema-valid instances by construction (per-section presence-gates wired structurally, conditional fields only sampled when the gating field's prediction matches). Cleaner but more work to iterate on; keep post-hoc until the MVP converges.

### Larger backbones

ResNet-18 pretrained on AudioSet or ImageNet, MobileNet, etc. The MVP's small custom CNN is the simplest thing that works. Backbone swap is a hyperparameter sweep, not new architecture work — defer until the pipeline is validated and we know what bottleneck (capacity vs data) is limiting accuracy.

### Music-pretrained encoder backbones (MERT / AudioMAE / BEATs)

A more targeted upgrade path than ImageNet/AudioSet ResNets. Two phased landing points worth tracking separately:

- **v0.2 — MERT-v1-95M** ([HF page](https://huggingface.co/m-a-p/MERT-v1-95M)). Music-domain self-supervised pretraining means timbre features come for free, which is exactly what subtractive synthesis turns into in the spectrum. Drop-in replacement for the encoder: `AutoModel.from_pretrained("m-a-p/MERT-v1-95M")` → mean-pool over the sustain region → 768-dim → concat with the 88-dim pitch multihot (now 856 dim) → 2-layer MLP → same 3 heads. Native HF transformers loader, raw-waveform input (no spectrogram preprocessing in our code path). 95M params is small enough to fine-tune end-to-end on a single mid-tier GPU. Worth picking up alongside `osc.2` / `noise` / modulation invariance, when the parameter space is big enough that custom-CNN capacity starts to bind.
- **v0.3 — AudioMAE** ([HF page](https://huggingface.co/hance-ai/audiomae)). Self-supervised on AudioSet, ViT-based with a spatial 8×64 latent map (not just a pooled vector) — gives a 2D feature map for the modulation-invariance temporal-pooling recipe. The right call once we're scaling to multi-engine + full modulation augmentation. BEATs (Microsoft, [unilm repo](https://github.com/microsoft/unilm/tree/master/beats)) is in the same class but ships weights via manual unilm checkpoint download instead of an HF native loader; revisit if MS publishes an HF-native release.

**Skip:** CLAP is a contrastive embedder optimized for semantic-category retrieval — it's tuned for "this is a flute" not "this is a saw at 2 kHz cutoff," so the embeddings won't capture our micro-timbre regression targets well. VGGish/YAMNet have 128/1024-dim outputs and are old; not worth the engineering vs MERT.

Decision deferred until baseline numbers from the 10K/50-epoch MVP run are in.

### Shared encoder with the modulation expert

Both experts consume mel-spectrograms of the same flattened audio. Sharing the encoder + branching into expert-specific heads is the maximum-sharing option. Cross-cutting decision flagged in both plans; defer until each expert ships independently and we have something to compare against.

### Richer pitch encoding / FiLM conditioning

The MVP encodes played pitches as 88-dim multi-hot concatenated to the post-pool bottleneck. Alternatives: pitch-class histogram (12-dim, more compact), sorted-pitch sequence with positional embeddings (handles polyphony order), or FiLM (Feature-wise Linear Modulation) gating on the conditioning vector. Promote if multi-hot proves under-expressive on richer scenes.

## Dataset improvements

### `note_transcribe` at dataset-generation time + transcription-noise augmentation

The MVP uses ground-truth pitches (free, deterministic). Real-audio inference will use `note_transcribe` (Basic Pitch) — synthetic→real gap on the conditioning input. Two ways to close it: (a) run `note_transcribe` on every rendered sample at dataset time and use its output as the label (slower, more realistic), or (b) add controlled noise to the ground-truth pitches at training time (drop notes, octave shifts, false positives) to simulate transcription error. Probably (b) first since it's cheaper and more controllable.

### `note_transcribe` accuracy validation on synthetic subtractive renders

Basic Pitch was trained on real instruments — synthetic subtractive audio (especially saw / square at extreme cutoffs) may have edge cases. Before relying on `note_transcribe` at dataset time, run a small audit: render N samples with known pitches, run `note_transcribe`, measure pitch accuracy + n_voices accuracy. Establishes the noise floor we'd need to model in (b) above.

### Variable amp envelope (sample from preset bank)

The MVP fixes the amp ADSR at a single "pad" preset. The cross-cutting setup calls for 5–10 baseline ADSR presets (pad, pluck, stab, lead, etc.) used identically across engines. Add this when amp-envelope variation becomes a confounder in eval — currently the amp expert is fully responsible so this expert never sees the envelope.

### Velocity variation

The MVP renders every voice at a single fixed velocity. Real audio has velocity-driven amp + filter response. Defer until amp + velocity-amount params come online.

### Pitch range expansion beyond C2–C6

C2–C6 is a 48-semitone range covering most practical playing zones. Real audio extends to C0–C8. Pitch range matters most when the harmonic spectrum at extreme pitches hides or reveals filter cutoffs the model would otherwise pin. Expand once the rest of the schema is filled in.

### Wider polyphony (4+ voices, denser chord textures)

MVP samples `n_voices ∈ {1, 2, 3}`. Pads and chord stabs in real audio routinely have 4–6 voices. Adding voices is cheap rendering-wise but may stress the conditioning input encoding and the model's chord invariance.

### Rejection sampling for silent / DC-only / clipping audio

The MVP fails loud on bad renders rather than rejecting them. Once the parameter space grows (e.g. extreme low-cutoff + low-resonance can produce near-silent audio at high frequencies), we'll need to filter generated audio for RMS floor + zero-crossing rate ceiling per the cross-cutting setup. Defer until silent / clipping samples actually start showing up.

### Pre-computed mel-spec cache

The MVP computes mel-specs on the fly during training. For larger datasets / longer training runs, caching mel-specs to disk speeds up training significantly. Defer until training time becomes a bottleneck.

## Real-world integration

### Real-keyboard recordings via `keyboards-mcp` (synthetic→real gap)

The MVP is purely synthetic. Real audio recorded from Prophet-6 / JUNO-X / Nord via `keyboards-mcp` is the eventual target. Synthetic→real gap is well-documented in inverse-synth research and likely needs domain adaptation, fine-tuning on a small real set, or generative augmentation (impulse responses, room sims, codec artifacts). Big enough that it's its own project; defer until the synthetic-only model is validated.

## Training infrastructure

### Cloud training via HF Jobs + dataset hosting on HF Datasets

The MVP trains locally on the developer's M3 Pro because the workload (10K samples, 707K-param CNN) finishes in well under 15 minutes — cloud at MVP scale is friction, not speed-up. The two scale-up signals that warrant moving off-local: (a) dataset >100K samples + multi-engine renders pushing single-machine render time past an hour, (b) backbone swap to MERT-v1-95M / AudioMAE bumping the trainable param count high enough that MPS becomes the bottleneck.

When that lands, the lightest path for a one-developer research project is:

- **HF Jobs** ([docs](https://huggingface.co/docs/huggingface_hub/guides/jobs)) for managed-GPU training: `hf jobs run --flavor a10g-large train_tone_generation.py`. Per-second billing on T4 / A10G / A100 / H100, no VM management. Requires HF Pro tier ($9/mo).
- **HF Datasets** for the synthetic dataset: `dataset.push_to_hub("uribrecher/sub-synth-mvp", private=True)` after a one-time local render. Collaborators `load_dataset(...)` and skip the local render entirely. Solves the "everyone re-renders the same 5 minutes of synthesis" friction.
- **HF Hub** for checkpoint hosting: `model.push_to_hub(...)` from inside the training job. Separate model versions per branch / experiment.

**Skip:**
- AutoTrain — no-code path doesn't support custom heads (we have 3 task-specific heads), so it doesn't fit our model anyway.
- Modal Labs ($3.95/h H100) — beautiful Python-native serverless, but the premium pays off for many short jobs, not occasional research runs.
- RunPod / Lambda Labs — cheaper raw compute ($2.49/h H100 on RunPod) but you manage Pods/SSH/dataset-transfer/checkpoint-upload yourself; the integration time eats the savings.
- Colab Pro — fine for one-off notebooks, friction at dataset >5–10 GB and reproducibility is poor.
- AWS SageMaker — way too heavy for one developer.

**Dataset generation stays local** even after this lands. SignalFlow rendering is CPU-bound; `--n-samples 100000` takes ~50 minutes locally, which is fine. Push the result to HF Datasets afterwards. Only revisit cloud rendering if we ever need 1M+ samples in a one-shot batch.

## Deployment / runtime

### CoreML export for deployed inference (Apple Neural Engine)

The MVP trains and runs inference in PyTorch (MPS for Apple Silicon GPU). For deployed inference inside `audio-analysis-mcp` — especially when packaged into the macOS app via `macos-packager/` — the right runtime is CoreML, which dispatches to the M3's Neural Engine (ANE) for substantially better latency and power efficiency than PyTorch MPS at inference. Path: train in PyTorch → freeze the checkpoint → convert via `coremltools.convert()` → ship a `.mlpackage`. Pick this up once the trained model is stable; benchmark ANE inference vs PyTorch MPS to confirm the win is real on this CNN size, and decide whether to keep PyTorch as the MCP-server runtime (smaller deps, hot-reloadable) or switch fully to CoreML (faster, harder to iterate on). Drops the PyTorch runtime dependency from the shipped app entirely if we go all-in.

## Comparisons / baselines

### Pre-pivot baseline (vs archived two-phase vertical pipeline)

The horizontal-pipeline pivot's value rests on the claim that stripping the amp envelope upstream and removing modulation as a confounder makes this stage's job easier. The archived `inverse-synth/` plan is the comparison point — same test set, different architecture. Run once both are runnable and the MVP has stabilized.

### Hyperparameter sweeps + dataset-size scaling experiments

Hard-coded hyperparams in MVP. Real sweeps (LR, batch size, model capacity, dataset size) come after the loop is validated and we know which axes actually move the metrics.

### Loss weighting / scheduling

MVP uses unweighted-summed loss across heads. Per-head weighting matters once one head dominates the others' gradients (often the regression heads vs the classification head). Defer until baseline numbers reveal a clear imbalance.

## Plan corrections

### Fix `tone-generation-research-plan.md`: drop `envelope.filter` ownership

The recently-merged plan (PR #2, commit `bf44bdc`) claims this expert owns `params.envelope.filter`. Latest user direction: modulation expert owns `envelope.filter` (alongside `lfo.*`). The Role section and the Phase 4 subtractive table both need to be updated. Small text-only PR; do alongside the next plan refresh.
