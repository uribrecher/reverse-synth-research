# Parameter Mapping — Design Plan

> The bridge between the analysis pipeline (which speaks a canonical, device-agnostic language) and `keyboards-mcp` (which speaks a different per-device language for each keyboard). Two artifacts: a canonical synthesis-parameter ontology, and per-device translators that map canonical params to device-specific MCP tool calls.

## Why This Stage Exists

The three analysis experts (`amplitude/`, `modulation/`, `tone-generation/`) predict synthesis parameters in physical units (Hz, ms, dB, %) with named enums. `keyboards-mcp` is per-device — Prophet-6, Roland JUNO-X, and Nord Electro 5D each expose their own parameter vocabulary. There is significant overlap (subtractive synths share most of their language), but no unified schema exists.

Without this stage, either:

- The analysis experts have to be device-aware — impractical, because the cross product of devices × engines × parameters explodes; or
- Every consumer of the analysis output has to write its own translation — duplicated logic, brittle, and the canonical schema becomes implicit and inconsistent.

This stage solves the problem with two artifacts:

1. **A canonical ontology** — a single schema describing synthesis parameters in physically-grounded, device-agnostic terms. It is the contract between analysis and translation.
2. **Per-device translators** — small adapter modules that map canonical params to specific `keyboards-mcp` tool calls per supported device.

This is not an ML research problem. It is schema design + adapter engineering. But it needs a plan because the canonical schema is a contract that everything else in the pipeline depends on, and getting it wrong has expensive downstream consequences.

## Position in the Pipeline

```
analysis pipeline ─→ unified canonical params ─→ [parameter-mapping] ─→ keyboards-mcp tool calls
                                                  ↑           ↑
                                                  │           └─ per-device translators (adapters)
                                                  └─ canonical ontology (schema)
```

Last stage of the pipeline. The only stage that knows about specific devices.

## Phase 1: Survey keyboards-mcp Parameter Schemas

Goal: ground truth on what each device speaks. No design decisions yet — just collect data.

For each device supported by `keyboards-mcp` (Prophet-6, Roland JUNO-X, Nord Electro 5D, mock):

- Run `list_parameters` and `list_synth_engines` against the device
- Collect parameter inventory: name, type (continuous / discrete), range, unit, enum values, brief description
- For routing / destination params (LFO targets, mod matrix slots), capture the device's destination vocabulary
- Note device-specific quirks (e.g., MIDI 0–127 scaling, log vs linear knob taper, special "off" values)

### Milestone

A canonical-ready survey document — one section per device — that's the input to Phase 2 ontology design.

## Phase 2: Design the Canonical Ontology

Goal: a single schema that describes synthesis parameters in device-agnostic, acoustically-grounded terms.

Organize by engine family (subtractive / FM / organ / wavetable / sample-based) plus cross-cutting categories (envelopes, modulation, mix, master). For each canonical param, define:

- **Name** — hierarchical, e.g., `filter.lp.cutoff_hz`
- **Type** — continuous / discrete / categorical
- **Unit** — Hz, ms, dB, ratio, percent, named enum
- **Range** — physical, not normalized (e.g., 20–20000 Hz)
- **Enum values** when applicable (e.g., `osc.shape ∈ {sine, saw, square, triangle, pulse}`)
- **Brief description** — what it acoustically means

Special attention to:

- **Routing params.** LFO targets and modulation destinations vary wildly across devices. Define a canonical destination vocabulary that is the *union* of common routing options, with a hierarchical naming scheme (e.g., `osc.1.amp`, `osc.1.pitch`, `filter.cutoff`); device translators subset it.
- **Engine-specific params.** Drawbar levels (organ), operator ratios (FM), wavetable position (wavetable). Per-engine sub-schemas, but follow the same canonical conventions.
- **Versioning.** The schema must evolve as devices are added without breaking analysis-side outputs. Semantic versioning + deprecation policy. Analysis side commits to a schema version; translators advertise the versions they support.

Output format: JSON Schema (language-neutral) with auto-generated Pydantic / TypeScript bindings. The JSON Schema is the canonical artifact.

### Milestone

A versioned canonical schema covering all engine families. Reviewed against the Phase 1 survey: every device parameter that has acoustic meaning has a canonical representation, and every canonical param has at least one device that supports it.

## Phase 3: Align Expert Outputs with the Canonical Schema

Goal: make the three analysis experts emit canonical params, not raw 0–1 vectors.

For each expert (amplitude, modulation, tone-generation):

- Map predicted output to canonical names + units
- Specify denormalization at the deliverable boundary: the model trains on 0–1 (better loss landscape), but the deliverable returns Hz / ms / etc.
- Update each expert's plan Deliverable section to reflect canonical output

This is a contract-level change to the existing plans, not a code change yet — it locks in what each expert must produce when implementation starts.

### Milestone

All three expert plans updated. Each Deliverable example uses canonical names and physical units, with an explicit reference to the canonical schema version.

## Phase 4: Per-Device Translators

Goal: adapter modules that take canonical params and emit `keyboards-mcp` tool calls per device.

**Location is decided.** Translators live inside `keyboards-mcp` itself, in the per-model folder for each device. `keyboards-mcp` already organizes device-specific code by model (one folder per keyboard), so the translator sits alongside the device driver, MIDI implementation, and the model's system prompt. This keeps all device-specific knowledge for a given keyboard colocated — when the model changes, everything that needs to change is in one place.

For each supported device (Prophet-6, JUNO-X, Nord Electro 5D, mock):

- **Name mapping** — canonical → device-specific (`filter.lp.cutoff_hz` → Prophet-6's `LP_CUTOFF`)
- **Range / scale mapping** — canonical physical units → device's preferred scale (linear, log, MIDI 0–127, etc.)
- **Enum mapping** — canonical enum values → device-specific enum values
- **Capability subsetting** — explicitly report which canonical params have no device equivalent on this device

Each translator is a thin module: a dictionary of mappings + small conversion functions + a capability table.

### Milestone

A `translate(canonical_params, device) → mcp_calls` function for each supported device, validated by round-trip tests: send the result through `keyboards-mcp`, read state back via `list_parameters`, confirm it matches the input canonical params (within tolerance).

## Phase 5: Fallback and Capability Policy

Goal: define what happens when canonical params can't map cleanly to the target device.

Cases:

| Situation | Options | Default |
|-----------|---------|---------|
| **Canonical has no device equivalent** (e.g., `wavetable.position` → Prophet-6) | Hard-fail / approximate / skip with warning | Per-param policy; default = warn + skip |
| **Device requires a param the canonical doesn't specify** | Use device default / leave current state | Leave current state |
| **Engine mismatch** (predicted subtractive → target Nord Electro, no subtractive synth) | Hard-fail | Hard-fail; orchestration should have caught this earlier |
| **Continuous canonical value outside device range** (e.g., 20 kHz cutoff on a device that caps at 16 kHz) | Clamp / fail | Clamp + warn |

Policy is documented per-canonical-param × per-device, not global. The translator is the last line of defense — it should fail loudly rather than silently approximate when correctness matters, and approximate explicitly (with a similarity score) when it makes sense.

### Two-Level Rejection

Feasibility is checked at two levels, not one:

1. **Agent-level (primary).** Each keyboard model in `keyboards-mcp` ships a **system prompt** describing its synth engine and full signal path. `sound-recreation-agent` has a multi-device-management skill that reads these prompts and matches predicted engine + canonical params to the most appropriate target device. If no device fits the analyzed sound, the agent rejects the request with an explanation. This is the right place for the decision — the agent has full context and human-readable capability descriptions to reason over.

2. **Translator-level (backup).** If the agent's choice was wrong — e.g., the device's signal path doesn't actually support a specific routing despite the system prompt suggesting it might — the per-device translator fails loud at translation time rather than silently approximating. Safety net, not main mechanism.

Don't try to make translators smart enough to recover from engine mismatches. That's the agent's job, with better information and better context.

### Milestone

A documented policy table covering every canonical param × device combination. No silent failures; every translation either succeeds, fails loudly, or warns explicitly.

## Boundaries with Other Repos

Several decisions sit outside this stage and are owned elsewhere:

- **Translator code physically lives in `keyboards-mcp`**, in each model's folder, alongside the device driver, MIDI code, and the model's system prompt. This plan defines what translators do; the code itself ships there. (See Phase 4.)
- **Engine-to-device feasibility** is owned by `sound-recreation-agent`. Each keyboard model's system prompt in `keyboards-mcp` describes its synth engine and signal path; the agent uses these to match predicted engine + canonical params to a feasible device. The translator does not attempt to recover from category errors — it just fails loud as a safety net. (See Two-Level Rejection in Phase 5.)
- **Multi-device routing** is also owned by `sound-recreation-agent`. The agent has a skill for managing multiple devices and choosing the best target per acoustic detection. The translator API takes a single concrete device target — multi-device fan-out is the agent's concern, not the translator's.

## Open Questions

- **Schema source language.** JSON Schema is language-neutral; Pydantic is Python-only and richer; TypeScript types are TS-only. Likely JSON Schema as the canonical source, with Pydantic / TS bindings auto-generated — analysis side is Python, translator side is TypeScript, so neither native is strictly better.

- **Where does the canonical schema itself physically live?** Translators live in `keyboards-mcp`, but the canonical schema is shared between `audio-analysis-mcp` (validates analysis output) and `keyboards-mcp` (validates translator input). Options: a shared package, source-of-truth in one repo with the other importing, or auto-generated bindings in each. Decision deferred to implementation.

- **Routing param canonicalization.** The set of LFO destinations varies wildly across devices. Defining a canonical "union" risks bloat; defining a canonical "intersection" loses expressiveness. Likely solution: a tagged hierarchical destination vocabulary with each device translator subsetting; capture in Phase 2.

- **Approximation similarity scoring.** When the policy is "approximate," how do we score approximation quality? Round-trip rendering + spectral similarity is one option, but expensive. A lightweight per-param distance metric may be enough.

## Deliverable

```python
# Canonical params produced by the analysis pipeline (merge of all three experts' outputs):
canonical = {
    "schema_version": "1.0",
    "engine": "subtractive",
    "params": {
        "osc.1.shape": "saw",
        "osc.1.pwm_pct": 44,
        "osc.2.detune_cents": 7,
        "osc.2.level": 0.6,
        "filter.lp.cutoff_hz": 1200,
        "filter.lp.resonance": 0.6,
        "envelope.amp.attack_ms": 12,
        "envelope.amp.decay_ms": 380,
        "envelope.amp.sustain": 0.62,
        "envelope.amp.release_ms": 220,
        "lfo.1.rate_hz": 4.2,
        "lfo.1.depth": 0.6,
        "lfo.1.target": "filter.cutoff",
    },
}

# Per-device translation:
from parameter_mapping import translate

result = translate(canonical, device="prophet-6")
# result = {
#   "mcp_calls": [
#     {"tool": "set_parameters", "args": {"LP_CUTOFF": 64, "LP_RES": 76, ...}},
#     ...
#   ],
#   "unsupported": [],          # canonical params with no device equivalent
#   "warnings": [],             # clamps, approximations, etc.
# }
```

The canonical schema is the contract between the analysis side and the device side. Once stable, it should change only through versioned updates with deprecation paths.