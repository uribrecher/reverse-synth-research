# Relocate the subtractive tone-generation training pipeline into `reverse-synth-research`

**Date:** 2026-06-19
**Issue:** [`uribrecher/audio-analysis-mcp#47`](https://github.com/uribrecher/audio-analysis-mcp/issues/47)
**Status:** design (approved, pending spec review)

## Problem

The subtractive **tone-generation training pipeline** currently lives inside the
`audio-analysis-mcp` package, even though no shipped MCP tool / `analysis/` /
`audio/` module / stdio server imports it. It is the **sole reason** `signalflow`
— a heavy native dependency ceilinged at `<0.5.4` for lack of a macOS wheel — is a
dependency of the analysis server. The matching design docs (the subtractive
tone-training MVP/backlog specs and the canonical schema) already live in
`reverse-synth-research`. Co-locating the training code with those docs shrinks the
MCP server's surface and dependency footprint and removes the `signalflow` wheel
headache from the analysis repo entirely.

## Goal

The tone-generation training pipeline (code + scripts + scratch explorers + tests +
its `signalflow` dependency) lives in `reverse-synth-research`, runs there, and is
covered by CI. `audio-analysis-mcp` contains none of it and has zero `signalflow`
reference in its deps, lockfile, or mypy config. Both repos' tests + mypy stay green.

## Non-goals

- The runtime MCP tools / `analysis/` package — untouched.
- The amplitude / modulation / tone analysis *experts* slated to live **in**
  `audio-analysis-mcp` per the umbrella design — untouched (the deferred
  `2026-04-26-amplitude-expert` SignalFlow integration test stays as-is; it is a
  separate, not-yet-implemented concern).
- The installable-packaging work in `audio-analysis-mcp#46`.
- Relocating historical `docs/superpowers/{plans,specs}` records in
  `audio-analysis-mcp` that *mention* signalflow — they are records of completed
  work and stay put.

## Current state (verified 2026-06-19)

### Source: `audio-analysis-mcp`
- `src/audio_analysis_mcp/research/tone_generation/` — `renderer.py` (SignalFlow
  subtractive renderer), `dataset.py`, `model.py`, `schema_io.py`, `README.md`,
  `__init__.py`; plus the parent `research/__init__.py`.
- `scripts/` — `train_tone_generation.py`, `eval_tone_generation.py`,
  `generate_subtractive_dataset.py`, `_eval_helpers.py`.
- `scratch/` — `explore_signalflow.py`, `explore_signalflow_audio_backend.py`,
  **`explore_subtractive_renderer.py`**, **`run_inference_example.py`**.
  (The last two are not in the issue's literal list but import `signalflow` /
  the moved package; left behind they would be dead/broken.)
- `tests/` — **`test_tone_generation_{dataset,model,renderer,schema_io,smoke}.py`**
  (also absent from the issue's list, but import the moved package + signalflow).
- `signalflow` is declared only in the `research` dependency-group of
  `pyproject.toml`, with a `signalflow.*` mypy `ignore_missing_imports` override.
- `schema_io.py` loads `subtractive.schema.json` + `synth-base.schema.json` from the
  sibling repo via `Path(__file__).resolve().parents[5] / "reverse-synth-research" /
  "parameter-mapping"`, with a `TONE_GEN_SCHEMA_DIR` env override.
- CI (`.github/workflows/ci.yml`) vendors a `schemas/` snapshot of those two JSON
  files and points the loader at it via `TONE_GEN_SCHEMA_DIR`, and syncs with
  `--group research` so the tone_generation tests get signalflow.
- All of the above landed in a single squash commit: `250145f` ("Subtractive
  tone-generation training pipeline (MVP) (#15)").

### Destination: `reverse-synth-research`
- Docs-only repo: no `pyproject.toml`, no package, no `uv.lock`.
- Already owns the canonical schema: `parameter-mapping/subtractive.schema.json` +
  `synth-base.schema.json`.
- Already owns the design docs: `docs/superpowers/specs/completed/2026-05-02-subtractive-tone-training-mvp.md`
  and `docs/superpowers/specs/2026-05-02-subtractive-tone-training-backlog.md`, plus
  `tone-generation/tone-generation-research-plan.md`.
- `docs/` is a **GitHub Pages site** (classic `/docs` branch deploy; `docs/index.html`
  + `docs/site-assets/`). A repo-root Python project and a Python CI workflow are
  orthogonal to it.

## Design

### 1. Destination project scaffold

A `uv` Python project at the repo root of `reverse-synth-research`, src-layout:

```
reverse-synth-research/
  pyproject.toml          # NEW
  uv.lock                 # NEW (generated)
  .gitignore              # extend with Python ignores
  .github/workflows/ci.yml# NEW
  src/tone_generation/    # moved package (NOT "tone-generation/" — that is the docs dir)
  scripts/                # moved training/eval/generate scripts + _eval_helpers.py
  scratch/                # moved signalflow/tone explorers
  tests/                  # moved test_tone_generation_*.py + conftest if needed
```

`pyproject.toml` declares:
- Python `==3.11.*` (signalflow cp311 wheels; matches source repo's Basic-Pitch pin).
- Runtime deps, specifiers copied verbatim from `audio-analysis-mcp/pyproject.toml`
  so resolved versions match: `signalflow>=0.4.0,<0.5.4`, `torch>=2.0`,
  `numpy>=1.24`, `scipy>=1.10`, `librosa>=0.10.0`, `soundfile>=0.12`,
  `jsonschema>=4.21`, `referencing>=0.30`. (The moved code imports only `torch` — not
  `torchaudio`/`torchcodec` — so those are omitted.)
- `dev` dependency-group: `pytest`, `mypy`.
- `[tool.pytest.ini_options]` defining the `slow` marker (the moved
  `test_tone_generation_dataset.py` + `test_tone_generation_smoke.py` use
  `@pytest.mark.slow`).
- `[[tool.mypy.overrides]]` with `ignore_missing_imports = true` for exactly the
  untyped imports the moved code uses: `signalflow.*`, `librosa.*`, `soundfile.*`,
  `scipy.*`, `jsonschema.*`, `referencing.*` (ported from the source's override list).
- Build backend `hatchling` (matches the source repo) with `src/` package discovery.

The package name becomes **`tone_generation`** (was `audio_analysis_mcp.research.tone_generation`).

### 2. Files moved (clean copy + delete; no git-history graft)

| From `audio-analysis-mcp` | To `reverse-synth-research` |
|---|---|
| `src/audio_analysis_mcp/research/tone_generation/{renderer,dataset,model,schema_io}.py`, `README.md`, `__init__.py` | `src/tone_generation/` |
| `src/audio_analysis_mcp/research/__init__.py` | dropped (subpackage no longer exists) |
| `scripts/{train_tone_generation,eval_tone_generation,generate_subtractive_dataset,_eval_helpers}.py` | `scripts/` |
| `scratch/{explore_signalflow,explore_signalflow_audio_backend,explore_subtractive_renderer,run_inference_example}.py` | `scratch/` |
| `tests/test_tone_generation_{dataset,model,renderer,schema_io,smoke}.py` | `tests/` |

Per the approved decision, the move is a **clean copy into the destination + delete
from the source**, as ordinary commits whose messages cross-reference the source
SHA `250145f` / PR #15 / issue #47 — not a `git filter-repo`/`format-patch` history
graft (the source is a single squash commit, so little real history exists to carry).

### 3. Rewrites at the destination

- **Imports:** `audio_analysis_mcp.research.tone_generation.X` → `tone_generation.X`
  across moved modules, scripts, and tests. The `scripts/_eval_helpers` sys.path
  import shim (`from _eval_helpers import ...`) is preserved as-is (still works with
  scripts co-located).
- **`schema_io.py` path:** `parents[5] / "reverse-synth-research" / "parameter-mapping"`
  → in-repo `Path(__file__).resolve().parents[2] / "parameter-mapping"`
  (`src/tone_generation/schema_io.py` → `parents[2]` is the repo root). The
  `TONE_GEN_SCHEMA_DIR` env override is kept. Update the module docstring that
  describes the path.
- **Doc links:** `tone_generation/README.md` and the `tone_generation/__init__.py`
  docstring reference the spec/backlog/schema via deep `../../../../../reverse-synth-research/...`
  relative paths. Rewrite to in-repo relative paths, pointing the MVP-spec link at
  its current home under `docs/superpowers/specs/completed/`.
- **`.gitignore`:** add Python/uv ignores (`__pycache__/`, `*.egg-info/`, `.venv/`,
  `.mypy_cache/`, `.pytest_cache/`, and a sensible ignore for generated dataset
  output dirs the scripts write).

### 4. CI at the destination

`.github/workflows/ci.yml` (greenfield — no existing workflows; classic Pages deploy
is unaffected):
- `astral-sh/setup-uv` + Python 3.11.
- `uv sync --dev`.
- `uv run mypy src/`.
- `uv run pytest -m "not slow"` — mirrors the source repo's CI posture. The
  `dataset` + `smoke` tests are marked `slow` (they render full datasets / run a
  train→eval loop); they are excluded on CI but **run locally** during verification.

**No schema vendoring needed:** `parameter-mapping/` is now in-repo, so the loader's
default path resolves under `${{ github.workspace }}` on CI. `signalflow` has
manylinux x86_64/aarch64 wheels (per the source `uv.lock`), so it installs on the
ubuntu runner.

### 5. Removals at the source (`audio-analysis-mcp`)

- Delete every moved file and the now-empty `research/` subpackage directory.
- `pyproject.toml`: remove the `research` dependency-group (`signalflow`) and the
  `signalflow.*` entry from the mypy `module` override list.
- `uv.lock`: relock so `signalflow` (and any deps it alone pulled) are gone. Run via
  `uv lock --no-sources` (the worktree can't resolve the `../SongFormer` path source).
- Delete the vendored `schemas/` directory (`subtractive.schema.json`,
  `synth-base.schema.json`, `README.md`) — it existed solely for the leaving tests.
- `.github/workflows/ci.yml`: remove the `TONE_GEN_SCHEMA_DIR` env var (+ its comment)
  and drop `--group research` from the `uv sync` step.
- `README.md` + `CLAUDE.md`: update the `uv sync --dev --group research --extra service`
  lines to drop the now-gone `research` group.
- **Keep:** `tests/test_packaging_metadata.py`'s `FORBIDDEN` tuple containing
  `signalflow` (it asserts signalflow is **absent** from published metadata — still
  true and still valuable), and all historical `docs/superpowers/{plans,specs}`
  records that mention signalflow.

### 6. Delivery — two PRs, ordered

1. **`reverse-synth-research` first** — adds the Python project, moved
   code/scripts/scratch/tests, import + schema-path rewrites, CI, this spec doc.
   PR body: *"Part of `uribrecher/audio-analysis-mcp#47`"* (non-closing cross-ref —
   GitHub's auto-close ignores negation, so avoid "Closes" here).
2. **`audio-analysis-mcp` second** — deletes moved files, removes the `research`
   group + signalflow + mypy override + vendored `schemas/` + CI workaround, relocks,
   updates README/CLAUDE. PR body: *"Closes #47"*.

Ordering guarantees the code always exists in at least one repo. Each repo: an
isolated worktree on a feature branch off the freshly-fetched `origin/main`, squash
merge, never a direct push to main.

## Testing / verification

- **Destination:** `uv run pytest` green (all 5 moved test modules) + `uv run mypy
  src/` clean. `signalflow 0.5.3` arm64 macOS wheel installs locally, so the renderer
  + smoke tests genuinely exercise SignalFlow on this machine.
- **Source:** `uv run pytest -m "not slow"` + `uv run mypy src/` green; `grep -ri
  signalflow` over `pyproject.toml`, `uv.lock`, and mypy config returns nothing.
- **Cross-check:** `test_packaging_metadata.py` still passes (signalflow absent).

## Risks / open points

- **Resolved-version drift:** copying specifiers (not the exact locked versions) into
  a fresh `uv.lock` could resolve `torch`/`librosa`/etc. to newer releases than the
  source pinned. Mitigation: copy the source's specifiers verbatim; if a moved test
  fails on a version delta, pin to the source's locked version.
- **`signalflow` Linux import side-effects:** the renderer notes SignalFlow touches an
  audio backend (PortAudio/ALSA) on import in some configs. The
  `explore_signalflow_audio_backend.py` scratch documents an offline/dummy-backend
  path. If CI import fails, set the dummy-backend env var the scratch identifies (or
  mark the renderer/smoke tests `slow` and exclude them on CI, mirroring the source's
  posture). Decide empirically during implementation.
- **Historical doc scope:** this design leaves `audio-analysis-mcp`'s historical
  plan/spec docs that mention signalflow untouched. If the intent is a full scrub,
  that is an easy add — flag at spec review.
