# Relocate Tone-Generation Training Pipeline — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move the subtractive tone-generation training pipeline (package + scripts + scratch + tests + the `signalflow` dependency) out of `audio-analysis-mcp` and into `reverse-synth-research`, where it runs under its own uv project + CI, leaving `audio-analysis-mcp` with zero signalflow footprint.

**Architecture:** Two repos, two PRs, executed in order. **PR1** scaffolds a `uv` Python project in the (currently docs-only) `reverse-synth-research`, copies the code in, rewrites imports + the schema path, and adds CI. **PR2** deletes the originals from `audio-analysis-mcp` and strips the `research` dependency-group, the signalflow mypy override, the vendored `schemas/` CI workaround, and the doc commands. The move is a clean copy + delete (no git-history graft); every commit cross-references source SHA `250145f` / PR #15 / issue #47.

**Tech Stack:** Python 3.11, `uv`, `hatchling` (src-layout), `pytest`, `mypy`, `signalflow`, `torch`, `librosa`, `scipy`, `numpy`, `soundfile`, `jsonschema`, `referencing`. GitHub Actions for CI.

**Spec:** `reverse-synth-research/docs/superpowers/specs/2026-06-19-tone-generation-relocation-design.md`

## Global Constraints

- **Source-of-truth paths.** Source repo: `/Users/uribrecher/test/sounds-and-recreation/audio-analysis-mcp`. Destination worktree (PR1, already created off fresh `origin/main`, branch `relocate-tone-generation-pipeline`): `/Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate`. PR2 worktree (to create): `/Users/uribrecher/test/sounds-and-recreation/.worktrees/aam-relocate`.
- **Python:** `requires-python = ">=3.11,<3.12"` in both repos (signalflow cp311 wheels).
- **Dependency specifiers** (copied verbatim from source `pyproject.toml`): `signalflow>=0.4.0,<0.5.4`, `torch>=2.0`, `numpy>=1.24`, `scipy>=1.10`, `librosa>=0.10.0`, `soundfile>=0.12`, `jsonschema>=4.21`, `referencing>=0.30`. The moved code imports only `torch` (not `torchaudio`/`torchcodec`).
- **Package rename:** `audio_analysis_mcp.research.tone_generation` → `tone_generation` everywhere.
- **Commits:** Conventional Commits (both repos gate on it). Sign every commit (SSH/1Password); commit+push must run with the sandbox disabled or the signature is silently dropped — verify with `git cat-file commit HEAD | grep -c gpgsig` (expect `1`). Never push to `main`; branch + squash-merge PR only.
- **Commit/PR trailers:** end commit messages with the `Co-Authored-By` + `Claude-Session` trailers; PR1 body says "Part of `uribrecher/audio-analysis-mcp#47`" (non-closing — auto-close ignores negation), PR2 body says "Closes #47".
- **Cross-repo ordering:** land PR1 before opening PR2 so the code always exists somewhere.
- **uv source caveat:** in the `audio-analysis-mcp` worktree, `[tool.uv.sources]` points `songformer` at `../SongFormer` which the worktree can't resolve — use `uv lock --no-sources` / `UV_NO_SOURCES=1 uv sync` there. The `reverse-synth-research` project has no such source — plain `uv lock` / `uv sync` works.

---

# PR1 — `reverse-synth-research` (destination)

Worktree: `.worktrees/rsr-relocate` (branch `relocate-tone-generation-pipeline`, already created off `origin/main`; the design spec + this plan are already committed there). **All paths in PR1 tasks are relative to that worktree** unless prefixed with `SRC =` (the source repo).

`SRC = /Users/uribrecher/test/sounds-and-recreation/audio-analysis-mcp`

### Task 1: uv project scaffold + move the `tone_generation` package

**Files:**
- Create: `pyproject.toml`
- Create: `.gitignore` additions (extend existing)
- Create: `src/tone_generation/{__init__.py,renderer.py,dataset.py,model.py,schema_io.py,README.md}` (copied from `SRC`)
- Generate: `uv.lock`

**Interfaces:**
- Produces: the importable package `tone_generation` with public symbols `tone_generation.renderer.render_chord`, `tone_generation.dataset.{ToneGenerationDataset,sample_dataset_config}`, `tone_generation.model.ToneGenerationCNN`, `tone_generation.schema_io.*` (consumed by Tasks 2 & 3).

- [ ] **Step 1: Create `pyproject.toml`**

```toml
[project]
name = "tone-generation-training"
version = "0.1.0"
description = "Subtractive tone-generation training pipeline (SignalFlow renderer → synthetic dataset → conditioned CNN). Research code for reverse-synth-research."
requires-python = ">=3.11,<3.12"
readme = "src/tone_generation/README.md"
license = { text = "GPL-3.0-or-later" }
dependencies = [
  "signalflow>=0.4.0,<0.5.4",
  "torch>=2.0",
  "numpy>=1.24",
  "scipy>=1.10",
  "librosa>=0.10.0",
  "soundfile>=0.12",
  "jsonschema>=4.21",
  "referencing>=0.30",
]

[dependency-groups]
dev = [
  "pytest>=8.0",
  "mypy>=1.10",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/tone_generation"]

[tool.pytest.ini_options]
markers = ["slow: marks tests as slow (deselect with '-m not slow')"]
testpaths = ["tests"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true

[[tool.mypy.overrides]]
module = [
  "librosa.*",
  "soundfile.*",
  "scipy.*",
  "signalflow.*",
  "jsonschema.*",
  "referencing.*",
]
ignore_missing_imports = true
```

- [ ] **Step 2: Extend `.gitignore`** — append Python/uv ignores below the existing IDE block:

```gitignore

# Python / uv
__pycache__/
*.py[cod]
*.egg-info/
.venv/
.mypy_cache/
.pytest_cache/

# Generated training artifacts
/datasets/
/runs/
*.wav
```

- [ ] **Step 3: Copy the package modules from source**

```bash
cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate
mkdir -p src/tone_generation
cp "$SRC"/src/audio_analysis_mcp/research/tone_generation/{__init__.py,renderer.py,dataset.py,model.py,schema_io.py,README.md} src/tone_generation/
```
(`SRC=/Users/uribrecher/test/sounds-and-recreation/audio-analysis-mcp`)

- [ ] **Step 4: Rewrite intra-package imports** — in `src/tone_generation/`, the only cross-module import is `dataset.py:32` (`from audio_analysis_mcp.research.tone_generation.schema_io import (`). Rewrite all occurrences:

```bash
cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate
grep -rl "audio_analysis_mcp.research.tone_generation" src/tone_generation/ \
  | xargs sed -i '' 's/audio_analysis_mcp\.research\.tone_generation/tone_generation/g'
```

- [ ] **Step 5: Fix the schema path in `schema_io.py`** — replace the `_DEFAULT_SCHEMA_DIR` block (currently `parents[5] / "reverse-synth-research" / "parameter-mapping"`) with the in-repo path. `src/tone_generation/schema_io.py` → `parents[2]` is the repo root:

```python
_DEFAULT_SCHEMA_DIR = (
    Path(__file__).resolve().parents[2]
    / "parameter-mapping"
)
```
Also update the module docstring (lines ~3–5) that says the schema lives in the "sibling `reverse-synth-research/parameter-mapping/` directory" — it now lives in this repo's `parameter-mapping/`. Keep the `TONE_GEN_SCHEMA_DIR` env override untouched.

- [ ] **Step 6: Fix doc links in `src/tone_generation/README.md` and `__init__.py`**
  - `__init__.py` docstring: change `spec: reverse-synth-research/docs/superpowers/specs/2026-05-02-subtractive-tone-training-mvp.md` → `spec: docs/superpowers/specs/completed/2026-05-02-subtractive-tone-training-mvp.md` and drop the now-irrelevant `plan: audio-analysis-mcp/...` line (or repoint it to `docs/superpowers/plans/2026-06-19-relocate-tone-generation-pipeline.md`).
  - `README.md`: rewrite the three `../../../../../reverse-synth-research/...` links to in-repo relative paths from `src/tone_generation/`:
    - spec → `../../../docs/superpowers/specs/completed/2026-05-02-subtractive-tone-training-mvp.md`
    - backlog → `../../../docs/superpowers/specs/2026-05-02-subtractive-tone-training-backlog.md`
    - schema → `../../../parameter-mapping/subtractive.schema.json`

- [ ] **Step 7: Lock + sync**

Run: `cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate && uv lock && uv sync --dev`
Expected: resolves and installs all deps including `signalflow` (0.5.3 arm64 macOS wheel) and `torch`. No errors.

- [ ] **Step 8: Verify the package imports + types**

Run: `uv run python -c "import tone_generation; from tone_generation import renderer, dataset, model, schema_io; print('ok')"`
Expected: `ok`

Run: `uv run mypy src/`
Expected: `Success: no issues found`

- [ ] **Step 9: Commit**

```bash
git add pyproject.toml uv.lock .gitignore src/tone_generation/
git commit -m "feat: scaffold uv project + move tone_generation package

Copied from audio-analysis-mcp@250145f (PR #15). Imports rewritten to the
tone_generation top-level package; schema_io now resolves the canonical
schema from this repo's parameter-mapping/ dir.

Part of uribrecher/audio-analysis-mcp#47"
```
(append the `Co-Authored-By` + `Claude-Session` trailers; run committing with the sandbox disabled; verify `git cat-file commit HEAD | grep -c gpgsig` → `1`.)

### Task 2: Move + green the 5 tests

**Files:**
- Create: `tests/test_tone_generation_{dataset,model,renderer,schema_io,smoke}.py` (copied from `SRC/tests/`)

**Interfaces:**
- Consumes: `tone_generation.*` (Task 1) and the `scripts/` (Task 3) for the subprocess-based slow tests. **Note:** the slow tests (`test_tone_generation_smoke.py`, plus `test_generate_dataset_script_smoke` in `test_tone_generation_dataset.py`) shell out via `sys.executable` to `Path(__file__).resolve().parents[1] / "scripts" / "..."` — i.e. they require `scripts/` at repo root (Task 3). They pass once Task 3 lands; the non-slow tests pass after this task.

- [ ] **Step 1: Copy the test modules**

```bash
cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate
mkdir -p tests
cp "$SRC"/tests/test_tone_generation_{dataset,model,renderer,schema_io,smoke}.py tests/
```
(Do **not** copy `tests/conftest.py` — the tone_generation tests use only the built-in `tmp_path` fixture, verified, not the source's custom audio fixtures.)

- [ ] **Step 2: Rewrite imports in the tests**

```bash
grep -rl "audio_analysis_mcp.research.tone_generation" tests/ \
  | xargs sed -i '' 's/audio_analysis_mcp\.research\.tone_generation/tone_generation/g'
```

- [ ] **Step 3: Run the non-slow tests (verify green now)**

Run: `uv run pytest -m "not slow" -v`
Expected: PASS. The fast tests in `renderer`, `model`, `schema_io`, and the non-slow `dataset` tests all pass. (Slow `dataset`/`smoke` tests are deselected — they need Task 3.)

- [ ] **Step 4: Commit**

```bash
git add tests/
git commit -m "test: move tone_generation tests (non-slow green)

From audio-analysis-mcp@250145f. Part of uribrecher/audio-analysis-mcp#47"
```
(signed; trailers.)

### Task 3: Move + fix scripts and scratch explorers

**Files:**
- Create: `scripts/{train_tone_generation.py,eval_tone_generation.py,generate_subtractive_dataset.py,_eval_helpers.py}`
- Create: `scratch/{explore_signalflow.py,explore_signalflow_audio_backend.py,explore_subtractive_renderer.py,run_inference_example.py}`

**Interfaces:**
- Consumes: `tone_generation.*` (editable-installed) + the `_eval_helpers` sys.path shim (preserved as-is — scripts insert their own dir on `sys.path`).
- Produces: the runnable training/eval/generate scripts the slow tests in Task 2 invoke.

- [ ] **Step 1: Copy scripts + scratch**

```bash
cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/rsr-relocate
mkdir -p scripts scratch
cp "$SRC"/scripts/{train_tone_generation.py,eval_tone_generation.py,generate_subtractive_dataset.py,_eval_helpers.py} scripts/
cp "$SRC"/scratch/{explore_signalflow.py,explore_signalflow_audio_backend.py,explore_subtractive_renderer.py,run_inference_example.py} scratch/
```

- [ ] **Step 2: Rewrite `tone_generation` imports in scripts + scratch**

```bash
grep -rl "audio_analysis_mcp.research.tone_generation" scripts/ scratch/ \
  | xargs sed -i '' 's/audio_analysis_mcp\.research\.tone_generation/tone_generation/g'
```

- [ ] **Step 3: Guard the amplitude imports in `scratch/explore_signalflow.py`** — this file's SignalFlow render half is what this repo cares about; its amplitude round-trip depends on `audio_analysis_mcp.{schemas,analysis.amplitude}` (which stay in the MCP server). Replace the block starting at `# Fabricate a single-cluster triage.` so both `audio_analysis_mcp` imports are wrapped and the comparison no-ops when the package is absent:

```python
        # The amplitude round-trip below depends on audio_analysis_mcp's
        # analyze_amplitude + schemas, which live in the analysis MCP server
        # (not this research repo). Skip it if that package isn't importable —
        # the SignalFlow render/probe above is the part this repo cares about.
        try:
            from audio_analysis_mcp.schemas import (
                CandidateCluster, CandidateNote, NoteEvent, NoteTriageFileData,
            )
            from audio_analysis_mcp.analysis.amplitude import analyze_amplitude
        except ImportError:
            print()
            print("=== amplitude round-trip skipped ===")
            print("  audio_analysis_mcp is not installed in this repo;")
            print("  the SignalFlow render probe above is the relevant part here.")
            return
```
Leave the body that builds the triage and calls `analyze_amplitude(...)` (and the tolerance-check prints) unchanged, after the guard.

- [ ] **Step 4: Verify scripts + scratch compile and the scripts' `--help` works**

```bash
uv run python -m py_compile scripts/*.py scratch/*.py && echo "compile-ok"
uv run python scripts/generate_subtractive_dataset.py --help
uv run python scripts/train_tone_generation.py --help
uv run python scripts/eval_tone_generation.py --help
```
Expected: `compile-ok`, then each `--help` prints usage (confirms `tone_generation` + `_eval_helpers` import paths resolve).

- [ ] **Step 5: Run the FULL test suite (slow included) — verifies the subprocess wiring end-to-end**

Run: `uv run pytest -v`
Expected: PASS, all 5 test modules including the `slow` smoke/dataset tests (these run the real generate→train→eval loop via SignalFlow on the local arm64 wheel).

- [ ] **Step 6: Commit**

```bash
git add scripts/ scratch/
git commit -m "feat: move tone_generation scripts + scratch explorers

Scripts/scratch imports repointed to the tone_generation package.
explore_signalflow.py's amplitude round-trip is guarded (audio_analysis_mcp
not present here). From audio-analysis-mcp@250145f.

Part of uribrecher/audio-analysis-mcp#47"
```
(signed; trailers.)

### Task 4: CI workflow + open PR1

**Files:**
- Create: `.github/workflows/ci.yml`

- [ ] **Step 1: Create `.github/workflows/ci.yml`**

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  test-and-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v4
        with:
          python-version: "3.11"
      - run: uv sync --dev
      - run: uv run mypy src/
      # Slow tests (full generate→train→eval loop) are deselected on CI, matching
      # audio-analysis-mcp's posture. The schema loads from this repo's
      # parameter-mapping/ dir (in the checkout), so no vendoring is needed.
      - run: uv run pytest -m "not slow" -v
```

- [ ] **Step 2: Sanity-check the CI commands locally** (they are the exact CI steps)

Run: `uv sync --dev && uv run mypy src/ && uv run pytest -m "not slow" -v`
Expected: all green.

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/ci.yml
git commit -m "ci: add Python CI (mypy + pytest) for tone_generation

Part of uribrecher/audio-analysis-mcp#47"
```
(signed; trailers.)

- [ ] **Step 4: Push + open PR1 via the `personal-pr` skill.** PR title (Conventional Commit): `feat: relocate tone-generation training pipeline from audio-analysis-mcp`. PR body references the spec, lists what moved, and ends with "Part of `uribrecher/audio-analysis-mcp#47`" (NOT "Closes"). Let CI go green and resolve any Copilot comments per the skill. **Land PR1 before starting PR2.**

---

# PR2 — `audio-analysis-mcp` (source removal)

Create the worktree only after PR1 has merged.

### Task 5: Remove the pipeline + signalflow footprint from `audio-analysis-mcp`

**Files:**
- Worktree: create `.worktrees/aam-relocate` off fresh `origin/main`, branch `remove-tone-generation-pipeline`.
- Delete: `src/audio_analysis_mcp/research/` (whole dir, incl. `__init__.py` + `tone_generation/`); `scripts/{train_tone_generation,eval_tone_generation,generate_subtractive_dataset,_eval_helpers}.py`; `scratch/{explore_signalflow,explore_signalflow_audio_backend,explore_subtractive_renderer,run_inference_example}.py`; `tests/test_tone_generation_{dataset,model,renderer,schema_io,smoke}.py`; `schemas/` (whole dir).
- Modify: `pyproject.toml`, `uv.lock`, `.github/workflows/ci.yml`, `README.md`, `CLAUDE.md`.

- [ ] **Step 1: Pre-flight + worktree**

```bash
cd /Users/uribrecher/test/sounds-and-recreation/audio-analysis-mcp
git fetch origin
git worktree add /Users/uribrecher/test/sounds-and-recreation/.worktrees/aam-relocate -b remove-tone-generation-pipeline origin/main
cd /Users/uribrecher/test/sounds-and-recreation/.worktrees/aam-relocate
```

- [ ] **Step 2: Delete the moved files + dirs**

```bash
git rm -r src/audio_analysis_mcp/research \
  scripts/train_tone_generation.py scripts/eval_tone_generation.py \
  scripts/generate_subtractive_dataset.py scripts/_eval_helpers.py \
  scratch/explore_signalflow.py scratch/explore_signalflow_audio_backend.py \
  scratch/explore_subtractive_renderer.py scratch/run_inference_example.py \
  tests/test_tone_generation_dataset.py tests/test_tone_generation_model.py \
  tests/test_tone_generation_renderer.py tests/test_tone_generation_schema_io.py \
  tests/test_tone_generation_smoke.py \
  schemas
```

- [ ] **Step 3: Remove the `research` dependency-group from `pyproject.toml`** — delete the block (the comment lines + group):

```toml
# Training-only (tone_generation renderer). Leaves the published package entirely.
# Temporary home until #47 moves the training pipeline to reverse-synth-research.
research = [
  "signalflow>=0.4.0,<0.5.4",
]
```

- [ ] **Step 4: Remove the signalflow mypy override** — delete the `"signalflow.*",` line from the `[[tool.mypy.overrides]]` `module` list.

- [ ] **Step 5: Update `.github/workflows/ci.yml`**
  - Delete the `TONE_GEN_SCHEMA_DIR: ${{ github.workspace }}/schemas` env line and its preceding explanatory comment block (the lines describing the vendored `schemas/` snapshot).
  - Change `- run: uv sync --dev --group research --extra service` → `- run: uv sync --dev --extra service`.

- [ ] **Step 6: Update doc commands**
  - `CLAUDE.md`: `uv sync --dev --group research --extra service` → `uv sync --dev --extra service`.
  - `README.md`: `uv sync --dev --group research --extra service   # dev tools + signalflow (tone_generation tests)` → `uv sync --dev --extra service   # dev tools + service deps` (drop the signalflow note).

- [ ] **Step 7: Relock**

Run: `uv lock --no-sources`
Expected: `uv.lock` updates; `signalflow` (and any deps it alone pulled) removed.

- [ ] **Step 8: Verify no signalflow references remain in config**

Run: `grep -rin "signalflow" pyproject.toml uv.lock README.md CLAUDE.md .github/`
Expected: **no output** (historical `docs/` plans/specs are intentionally untouched and not in this grep).

Run: `grep -rn "research" src/ tests/`
Expected: no `audio_analysis_mcp.research` references remain.

- [ ] **Step 9: Verify the source repo is still green**

```bash
UV_NO_SOURCES=1 uv sync --dev --extra service
uv run mypy src/
uv run pytest -m "not slow" -v
```
Expected: mypy clean; tests pass. `test_packaging_metadata.py` still passes (its `FORBIDDEN` signalflow assertion is now trivially true).

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "refactor: remove tone-generation training pipeline + signalflow

Moved to reverse-synth-research (see uribrecher/reverse-synth-research). Drops
the research dependency-group, the signalflow mypy override, the vendored
schemas/ CI snapshot + TONE_GEN_SCHEMA_DIR env, and the research-group doc
commands. Source of the move: this repo @250145f (PR #15).

Closes #47"
```
(signed; trailers; verify gpgsig.)

- [ ] **Step 11: Push + open PR2 via the `personal-pr` skill.** PR title (Conventional Commit, drives `cz bump` — this is a `refactor:` with no release impact, which is fine): `refactor: remove tone-generation training pipeline + signalflow dep (#47)`. Body notes the destination repo + that this completes the move; ends with "Closes #47". Watch the `changeset`/`pr-title` + CI checks; resolve Copilot comments per the skill.

---

## Self-Review

**Spec coverage** (each spec section → task):
- Destination scaffold (spec §1) → Task 1 (pyproject, uv.lock, .gitignore, src layout). ✓
- Files moved (spec §2) → Tasks 1 (package), 2 (tests), 3 (scripts + scratch). ✓
- Rewrites (spec §3): imports → Tasks 1–3 sed steps; schema path → Task 1 Step 5; README/`__init__` links → Task 1 Step 6. ✓
- CI at destination (spec §4) → Task 4. ✓
- Source removals (spec §5) → Task 5 (files, research group, mypy override, schemas/, CI env + `--group research`, README/CLAUDE; keep `test_packaging_metadata.py`). ✓
- Delivery two PRs ordered (spec §6) → Task 4 Step 4 + Task 5 Step 11. ✓
- Testing/verification → Task 2 Step 3, Task 3 Step 5, Task 4 Step 2, Task 5 Steps 8–9. ✓
- `explore_signalflow.py` amplitude-coupling decision (move + guard) → Task 3 Step 3. ✓ (deviation from spec's open point, resolved by user.)

**Placeholder scan:** No TBD/TODO; every code/edit step shows exact content or exact command + expected output.

**Type/name consistency:** Package symbols named consistently across tasks (`render_chord`, `ToneGenerationDataset`, `sample_dataset_config`, `ToneGenerationCNN`). `sed` rename pattern identical in Tasks 1–3. Schema-path `parents[2]` matches the `src/tone_generation/` depth. Test→script `parents[1]` anchor matches the root-level `scripts/` + `tests/` layout.

**macOS `sed` note:** steps use `sed -i ''` (BSD sed). On a Linux executor use `sed -i` (no `''`).
