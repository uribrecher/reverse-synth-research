# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Directory Is

Primarily a **research-design repository** — design documents for the ML analysis pipeline (the three analysis experts + parameter mapping) destined for the sibling `audio-analysis-mcp` Python directory. `docs/` is a GitHub Pages site.

It **also hosts the subtractive tone-generation training pipeline** — a `uv` Python project relocated here from `audio-analysis-mcp` (issue #47) to sit beside its specs:

- `src/tone_generation/` — renderer / dataset / model / schema_io (the training package)
- `scripts/` — `generate_subtractive_dataset.py`, `train_tone_generation.py`, `eval_tone_generation.py`
- `tests/` — `test_tone_generation_*.py`; `scratch/` — SignalFlow explorers

```bash
uv sync --dev          # install (incl. signalflow, torch)
uv run pytest          # full suite (-m "not slow" skips the train/eval loop)
uv run mypy src/       # type-check
```

## Architecture: Three Analysis Experts + Parameter Mapping

Audio analysis decomposes into three orthogonal experts (each stripping its concern from the audio), and the unified output flows through a final parameter-mapping stage that translates canonical params to per-device `keyboards-mcp` calls:

```
audio ─→ [amplitude] ─→ ADSR (canonical)
              ↓
      flattened audio ──┬──► [modulation]      ─→ LFO / mod params (canonical)
                        │
                        └──► [tone-generation] ─→ engine + static tone params (canonical)
                                                  (modulation-invariant via training)

                           unified canonical param vector (merged from all three)
                                            ↓
                                  [parameter-mapping]
                                            ↓
                              per-device keyboards-mcp tool calls
```

| Subdir | Method | Output |
|--------|--------|--------|
| `amplitude/` | DSP envelope follower (no ML) | ADSR estimate (canonical), flattened audio |
| `modulation/` | RNN / LSTM / 1D CNN on spectrogram time-series | LFO rate / depth / target (canonical) |
| `tone-generation/` | CNN on sustain-region slice; modulation-invariant via temporal pooling + augmentation | Engine + static tone params (canonical) |
| `parameter-mapping/` | Schema design + per-device adapters (no ML) | Canonical ontology + device translators → keyboards-mcp tool calls |

Engine classification is a softmax head on the tone-generation expert — there is **no separate engine-detection phase**.

This replaces an earlier two-phase vertical pipeline (detect-engine → predict-all-parameters). The pivot was driven by the chat in `chat-with-gemini.txt` — read it before making structural changes; it's the historical record of why the architecture is what it is.

## Why Horizontal

The vertical pipeline was a cascading-error system: misclassification in phase 1 produced garbage in phase 2. The horizontal pipeline avoids this by treating ADSR, modulation, and tone generation as orthogonal phenomena that can be analyzed independently. ADSR in particular is a confounding variable — it changes the shape of the audio but tells you nothing about the engine — so stripping it before classification meaningfully helps the downstream model.

The horizontal claim is also a measurable hypothesis, not a foregone conclusion. The tone-generation plan calls out a baseline comparison against the archived vertical pipeline — the pivot's value should be verified empirically once both are runnable.

## Cross-Cutting Setup

These choices apply to the training pipelines for `modulation/` and `tone-generation/` (the amplitude expert is DSP-only, no training). Captured once here, referred back from each plan:

- **Renderer:** SignalFlow (Python, batch-friendly, pythonic node graph). VST plugins were considered and rejected — too heavy for batch dataset generation.
- **Parameter sampling:** Sobol sequences or Latin Hypercube via `scipy.stats.qmc`. Naive `random.uniform` produces poor coverage in high-dimensional parameter spaces ("curse of dimensionality").
- **Frequency / time params:** sample logarithmically (filter cutoffs, LFO rates, envelope times). Linear sampling biases the dataset toward unmusical extremes.
- **Rejection sampling:** filter generated audio for silence / pure noise / DC clicks before adding to dataset (RMS amplitude floor + zero-crossing-rate ceiling).
- **Baseline ADSR presets — the orthogonality rule.** Define 5–10 envelope presets (pad, pluck, stab, lead, etc.). **Use the exact same baseline preset across every engine type.** This is the single most important rule of the dataset pipeline: it prevents shortcut learning where the model identifies engines by their envelopes (which it can do trivially) rather than by their timbre (which is what we actually want). The rule applies wherever envelopes appear in training data.

## How the Plans Relate

- `amplitude/` runs first and is mostly DSP. It strips the volume envelope, producing flattened audio that both downstream experts consume.
- `modulation/` and `tone-generation/` run **in parallel** on the flattened audio — they are not sequential. Strict de-modulation of the audio for the tone-generation expert was considered and rejected: filter LFO and wavetable scanning have no clean underlying spectrum to recover (cycle-averaging gives you the average filtered spectrum, not the dry spectrum), so surgical removal is misleading. Instead, the tone-generation expert is trained to be modulation-invariant via temporal pooling and data augmentation.
- Both consume mel spectrograms and may share an encoder — that's an open architectural question called out in both plans.
- All three experts emit **canonical** parameters — physical units (Hz, ms, dB, %) and named enums — defined by the schema in `parameter-mapping/`. This is the contract between the analysis side and the device side. Without it, experts would have to be device-aware (impractical) or every consumer would have to write its own translation (brittle).
- `parameter-mapping/` owns the canonical ontology and the per-device translator design. At implementation time, **the translators live inside `keyboards-mcp`** in each model's folder (alongside the device driver, MIDI code, and the model's system prompt) — `keyboards-mcp` already organizes device-specific code by model. The canonical schema is a shared artifact; its physical home (shared package vs. one-repo-source-of-truth) is still open.
- **Engine-to-device feasibility and multi-device routing** are owned by `sound-recreation-agent`, not the translators. Each keyboard model's system prompt in `keyboards-mcp` describes its synth engine and signal path; the agent reads these via its multi-device-management skill to match predicted engine + canonical params to a feasible target device. If nothing fits, the agent rejects rather than asking a translator to approximate. The translator only handles a single concrete target and fails loud on incompatibility as a safety net.
- All four feed into the same final output: device-specific `keyboards-mcp` tool calls that recreate the analyzed sound.

## Archive

`archive/` contains the previous two-phase research plans (`engine-detection/`, `inverse-synth/`). They are kept for reference — substantial content (Sobol sampling, SignalFlow rendering rationale, per-engine parameter spaces, evaluation metrics) carries over to the horizontal plans. The vertical pipeline they describe is no longer the architecture; the archive is documentation of why we pivoted, not a parallel design.

## Where Code Will Eventually Live

These are **plans, not implementations**. When implementing:

- Code goes into `../audio-analysis-mcp/` (the Python MCP server), not here.
- This directory remains the design source of truth — update plans when scope or approach changes.
- The Python interfaces at the bottom of each plan are the contracts with `audio-analysis-mcp`.

## Editing the Plans

- Preserve the phased structure and the Milestone / Open Questions / Deliverable sections — downstream readers rely on them.
- Cross-cutting setup (renderer, sampling, baseline ADSR rule) lives here in CLAUDE.md, not in each plan — keep it DRY. Refer back from plans, don't duplicate.
- The Python interfaces at the bottom of each plan are contracts with `audio-analysis-mcp` — changes there imply MCP-tool changes downstream.

## Parent Context

Parent (`../`) has its own CLAUDE.md describing the four-repo system (`keyboards-mcp`, `sound-recreation-agent`, `audio-analysis-mcp`, `macos-packager`). Read it when work touches anything outside this directory — particularly the audio-analysis-mcp consumer side.