---
mode: backlog
parent_topic: subtractive-ontology
mvp_spec: ./2026-05-02-subtractive-ontology-mvp.md
---

# Subtractive-Synth Ontology — Deferred Backlog

Features identified during MVP brainstorm but not in the first slice. Each is a stub — when picked up, run standard `superpowers:brainstorming` (or `mvp-brainstorm` again if it is still big) on it as a fresh topic.

## Other engine families (FM, organ, wavetable, sample-based, additive, granular)
Each engine family needs its own canonical schema and survey. FM (DX-style: operators, ratios, modulation index, feedback) and organ (drawbars, percussion, key click, rotary speaker) are the most likely next targets given `keyboards-mcp` already has Nord Electro 5D (organ + EP) and JUNO-X (which exposes more than its subtractive layer). Wavetable and sample-based are further-out. Each engine family is roughly the size of this MVP slice.

## Nord Electro 5D and other non-subtractive `keyboards-mcp` devices
Excluded from the subtractive survey because the device is organ + electric piano + sampled instruments, not subtractive. Pick this up alongside the organ ontology.

## Wider canon — expanding the survey beyond eight devices
The shortlist is "fixed" specifically so the MVP has a finish line. As the inverse-synth scope grows, more devices may need representation: Oberheim OB-6, DSI Pro 2 / Pro 3, Behringer Deepmind / Poly D, Korg Prologue, Modal Argon8, Waldorf Iridium, Yamaha CS-80 reissues, Moog One, ASM Hydrasynth, etc. Treat each addition as a delta against the existing schema — if it introduces nothing new, it just gets a survey doc; if it forces a schema extension, version-bump the schema.

## Union-coverage features explicitly excluded by "common core"
The common-core boundary parks several musically-real features that some surveyed devices support but most do not:

- Oscillator cross-modulation, hard sync, ring modulation, oscillator-FM
- Dual filters (series, parallel, morphing topologies, multi-mode)
- Sub-oscillator wave shape variation beyond fixed square/sine
- Vintage-knob noise / drift simulation
- Velocity-sensitive envelopes beyond a single per-envelope velocity-amount param
- Aftertouch routing
- Per-voice / per-key modulation
- Full mod-matrix slots (source × destination × amount × polarity)
- Effects chains (chorus, delay, reverb, EQ) — these are post-engine and arguably belong in a separate "fx" ontology

Each is a candidate for either a v0.2 schema extension or a separate "subtractive-extended" schema. Decide when a downstream consumer actually needs them.

## Auto-generated Pydantic and TypeScript bindings
The MVP ships JSON Schema only. When `audio-analysis-mcp` (Python) needs to validate analysis output and `keyboards-mcp` (TypeScript) needs to type translator inputs, generate bindings from the schema. Tooling candidates: `datamodel-code-generator` for Pydantic, `json-schema-to-typescript` for TS. Decision deferred until there is a real consumer asking for the bindings.

## Codegen / build pipeline
Once bindings exist, the build flow needs to regenerate them when the schema changes, and downstream repos need to pick them up. Possible approaches: shared package published from a chosen home repo, or per-repo regenerate-on-build. Tied to the "physical home" question below.

## Example fixtures + JSON Schema validator integration (the Q5-B option)
Hand-author 2–3 representative canonical instances ("Prophet-6 brass pad", "Muse lead", "JUNO-X bass") and run them through a JSON Schema validator as part of CI for the schema. The user explicitly took the design-review-only path for MVP and accepted the risk of latent schema bugs. Pick this up if the schema starts breaking when downstream consumers wire it in.

## Schema versioning and deprecation policy
The schema is versioned `0.1` in the MVP. A real versioning policy — semver semantics, what counts as a breaking change, deprecation windows, how analysis-side and translator-side advertise compatible versions — needs to exist before the schema gets second consumer. The plan calls for "Analysis side commits to a schema version; translators advertise the versions they support."

## Canonical schema's physical home
MVP parks the schema under `reverse-synth-research/parameter-mapping/`. That is fine for design, but once `audio-analysis-mcp` and `keyboards-mcp` consume it, the schema needs a clear source-of-truth location. Options: shared package (separate repo or workspace), source-of-truth in one repo with the other importing, auto-generated bindings copied into each repo. This is the open question already noted in `parameter-mapping-plan.md`.

## Phase 3 — aligning the three expert plans with the canonical schema
Update `amplitude/`, `modulation/`, and `tone-generation/` plans so each Deliverable section emits canonical names + physical units. Specify training-time 0–1 → deliverable-time canonical-unit denormalization at the boundary. Contract-level change to existing plans, not a code change. Should follow shortly after MVP approval since each expert plan currently has its own ad-hoc parameter names.

## Phase 4 — per-device translators inside `keyboards-mcp` model folders
For each supported device, write a `translate(canonical_params) → mcp_calls` adapter that lives in the model's folder alongside `device.ts`, `midi-map.ts`, and the system prompt. Includes name mapping, range/scale mapping, enum mapping, and an explicit per-device capability table. Validated by round-trip tests: send through `keyboards-mcp`, read state back via `list_parameters`, confirm match within tolerance. One translator per device; Prophet-6 is the natural first target because of its rich `midi-map.ts`.

## Phase 5 — fallback and capability policy table
Document, per canonical-param × device, what happens on mismatch: hard-fail, clamp, approximate, skip-with-warning, leave-current-state. Explicit and per-param, not global. The MVP schema gives translators the input vocabulary; this stage gives them the failure semantics.

## Two-level rejection wiring (agent-level + translator safety net)
- Agent-level (primary): `sound-recreation-agent` reads each `keyboards-mcp` model's system prompt and matches predicted engine + canonical params to a feasible target device. If nothing fits, the agent rejects with an explanation. Implemented as a multi-device-management skill in the agent.
- Translator-level (backup): if the agent's choice was wrong, the translator fails loud at translation time. Safety net only.

## Approximation similarity scoring
When fallback policy permits approximation, how do we score the result? Options: round-trip render through SignalFlow + spectral similarity (expensive but principled), per-param distance metric (cheap but blind to perceptual nonlinearities), perceptual loss using a pretrained audio embedding. Decide based on what fallback policy actually wants to call "good enough."

## Full routing-destination canonicalization (mod-matrix destination union)
The MVP exposes seven LFO destinations. Real subtractive synths route many more sources (envelopes, velocity, aftertouch, mod wheel, key tracking, random) to many more destinations (filter resonance, drive, individual oscillator pitches, pulse-widths, FX sends, voice pan). Define the canonical destination union, decide on a hierarchical naming scheme that resists bloat, and let device translators subset it. This is the routing-canonicalization open question already in `parameter-mapping-plan.md`.

## Analysis-side denormalization integration
The analysis experts train on 0–1 normalized targets (better loss landscape) but must emit canonical Hz / ms / dB at the deliverable boundary. Implement denormalization layers in the experts' inference paths, with the canonical schema as the unit-of-truth. Tied to Phase 3.
