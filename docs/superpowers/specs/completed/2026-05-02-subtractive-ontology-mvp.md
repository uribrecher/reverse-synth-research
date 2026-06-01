---
mode: mvp
parent_topic: subtractive-ontology
backlog: ./2026-05-02-subtractive-ontology-backlog.md
---

# Subtractive-Synth Ontology — MVP Slice

> **Scope discipline:** This spec is intentionally a thin slice. Deferred features live in the backlog file linked above. `writing-plans` should plan ONLY what is in this spec — do not pull from the backlog.

## What this slice is

A cross-vendor Markdown survey of eight subtractive polysynths plus a hand-authored `subtractive.schema.json` that defines the common-core canonical parameter ontology — names, types, units, ranges, enums, and a small canonical destination vocabulary for routing. Pure design artifact: no codegen, no bindings, no translators, no fixtures. Reviewed and approved by the user as the comm protocol between the analysis pipeline and `keyboards-mcp`.

## Why this cut

The ontology is the single most load-bearing contract in the analysis-to-device pipeline. A reviewable, cross-vendor-grounded schema is the smallest artifact that proves the contract is real. Bindings, translators, fallback policy, and engine families beyond subtractive are valuable but downstream — they only make sense once the ontology exists. The "common core" coverage rule and the eight-device shortlist together give the slice a concrete finish line that fits the inverse-synth's own minimal first-iteration scope.

## In scope

- **Survey of exactly these eight devices** (subtractive engines only):
  1. Sequential Prophet-6 (analog, in `keyboards-mcp`)
  2. Roland JUNO-X — subtractive engine layer only (digital VA, in `keyboards-mcp`)
  3. Moog Muse (poly analog)
  4. Arturia PolyBrute 12 (poly analog)
  5. Oberheim OB-X8 (poly analog)
  6. Korg Minilogue XD — subtractive layer only (hybrid analog + digital osc)
  7. Novation Summit (poly hybrid)
  8. Sequential Prophet-5 Rev 4 (poly analog)
- **Survey content per device:** parameter inventory (name, type, range, unit, enum values where applicable, brief acoustic description), modulation routing destinations the device exposes, device-specific quirks worth recording (knob taper, MIDI scaling, special "off" values).
- **Survey sourcing:** online research (manufacturer manuals, MIDI implementation charts, public synth-engine writeups) plus, for Prophet-6 and JUNO-X, cross-checking against `keyboards-mcp`'s `list_parameters` and existing `midi-map.ts` files.
- **Canonical ontology** — a two-file split: `synth-base.schema.json` (engine-agnostic — `ADSR`, `Voice`, `Master`, `Envelopes` shape, top-level wrapper) plus `subtractive.schema.json` (subtractive-specific — extends the base via `allOf`, constrains `engine` to `subtractive`, adds oscillators / noise / filter / LFO / `SubtractiveEnvelopes`). The split is a v0.1 architectural commitment so future engine schemas (FM, organ, wavetable, …) reuse the base. Together they cover the common-core subtractive signal path:
  - Engine identifier and schema version
  - Oscillators (1–2 main + optional sub) with shape, level, detune, octave/coarse, pulse-width
  - Noise generator (optional) with level and color
  - Mixer levels per source
  - One low-pass filter with cutoff (Hz), resonance, envelope amount, key tracking, drive/saturation
  - Amplitude envelope (ADSR, ms + 0–1 sustain)
  - Filter envelope (ADSR, ms + 0–1 sustain)
  - One LFO with rate (Hz), shape enum, depth, and a small canonical destination vocabulary
  - Voice/global parameters: glide, voice mode (mono/poly/unison), unison voices, master volume
- **Hierarchical canonical names** (e.g. `osc.1.shape`, `filter.lp.cutoff_hz`, `lfo.1.target`).
- **Physical units** — Hz, ms, cents, dB, 0–1 ratios, named enums. No normalized 0–127 or 0–1 stand-ins where a physical unit applies.
- **Canonical destination vocabulary** for the LFO target enum, restricted to the common-core set: `osc.pitch`, `osc.shape`, `osc.pwm`, `osc.amp`, `filter.cutoff`, `filter.resonance`, `amp.level`. This is the *common-core* destination list — full mod-matrix destinations are deferred.
- **A short `subtractive-ontology.md` companion doc** that walks through the schema in prose: design rules, naming conventions, why each param exists, and the explicit "common core" boundary.
- **Deliverable location:** all artifacts live under `reverse-synth-research/parameter-mapping/`.

## Out of scope (see backlog)

- Other engine families: FM, organ, wavetable, sample-based, additive, granular
- Nord Electro 5D and other non-subtractive `keyboards-mcp` devices
- Expanding the survey beyond the eight-device shortlist
- Union-coverage features (cross-mod, sync, ring mod, dual filters, multi-mode filter morphing, full mod-matrix slots, vintage-knob noise simulation)
- Auto-generated Pydantic and TypeScript bindings
- Codegen / build pipeline
- JSON-Schema-validator-backed example fixtures
- Schema versioning and deprecation policy
- Canonical-schema physical home decision (shared package vs. source-of-truth in one repo)
- Phase 3: aligning the three expert plans with the canonical schema
- Phase 4: per-device translators inside `keyboards-mcp` model folders
- Phase 5: fallback and capability policy table
- Two-level-rejection wiring (agent-level skill + translator safety net)
- Approximation similarity scoring
- Full routing-destination canonicalization beyond the common-core LFO targets
- Analysis-side denormalization integration (training-time 0–1 → deliverable-time canonical units)

## Architecture

```
reverse-synth-research/parameter-mapping/
├── parameter-mapping-plan.md          (existing — design plan)
├── survey/
│   ├── README.md                      (index + sourcing notes)
│   ├── prophet-6.md
│   ├── juno-x.md
│   ├── moog-muse.md
│   ├── polybrute-12.md
│   ├── ob-x8.md
│   ├── minilogue-xd.md
│   ├── novation-summit.md
│   └── prophet-5.md
├── subtractive.schema.json            (canonical ontology — JSON Schema)
└── subtractive-ontology.md            (prose companion to the schema)
```

## Components

### Survey docs (`survey/<device>.md`)

Each file follows a fixed template so cross-device comparison is mechanical:

- **Header:** device name, manufacturer, year, engine type (analog / VA / hybrid), polyphony, sources cited.
- **Signal path summary:** one paragraph describing the dry signal path.
- **Parameter inventory tables:** one table per stage (oscillators, mixer, filter, envelopes, LFO, voice, master). Columns: param name (device-native), type, range, unit, enum values, brief acoustic description.
- **Modulation routing:** the destination vocabulary the device exposes for its LFO(s) and mod matrix.
- **Quirks:** anything that affects canonical mapping — knob taper, log vs. linear scaling, MIDI 0–127 quantization, special "off" values, asymmetric ranges.

`survey/README.md` is the index: list of devices, sourcing approach (manuals + MIDI charts + `keyboards-mcp` cross-check for the two devices we control), the explicit "subtractive engines only" rule, and the common-core boundary the survey was used to inform.

### Canonical schema (`subtractive.schema.json`)

JSON Schema (draft 2020-12). Top-level shape:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "subtractive.schema.json",
  "title": "Subtractive Synthesis Canonical Parameters",
  "type": "object",
  "required": ["schema_version", "engine", "params"],
  "properties": {
    "schema_version": { "const": "0.1" },
    "engine": { "const": "subtractive" },
    "params": { "$ref": "#/$defs/SubtractiveParams" }
  },
  "$defs": {
    "SubtractiveParams": { ... },
    "Oscillator": { ... },
    "Filter": { ... },
    "ADSR": { ... },
    "LFO": { ... },
    "ModulationDestination": { "enum": ["osc.pitch", "osc.shape", ...] }
  }
}
```

Param style:

- Hierarchical dotted names live as object keys, not as string identifiers — the schema models the canonical structure, so `params.osc.1.shape` is a real path.
- Continuous params declare `type: number`, `unit` (custom keyword in `x-unit`), and explicit `minimum`/`maximum`. Examples: `cutoff_hz` (20–20000), `attack_ms` (0–10000), `detune_cents` (−1200–1200), `level` (0–1), `volume_db` (−60–6).
- Discrete params declare `enum`. Examples: `osc.shape ∈ {sine, saw, square, triangle, pulse}`, `lfo.shape ∈ {sine, triangle, square, saw, ramp, sample_hold}`, `voice.mode ∈ {mono, poly, unison}`.
- Optional sections (sub-osc, noise, filter envelope) are top-level optional objects under `params` so a minimal Prophet-5-style instance and a Muse-style instance can both validate.

### Ontology companion (`subtractive-ontology.md`)

Prose doc explaining the schema:

- Design rules (physical units, hierarchical names, optional sections, common-core boundary).
- The canonical destination vocabulary and why these seven targets (and not more).
- Per-section walkthrough: oscillators, filter, envelopes, LFO, voice/master — what each canonical param means acoustically and how it generally maps across the surveyed devices.
- An explicit "what is *not* here and why" section pointing into the backlog so future readers do not mistake the absence of a param for an oversight.

## Data flow

This slice produces no runtime data flow. The artifact is the contract:

```
(survey/) ──→ informs design of ──→ subtractive.schema.json ──→ reviewed by user ──→ MVP done
                                              ↑
                                              └─ subtractive-ontology.md (prose companion)
```

Future consumers (Phase 3 expert alignment, Phase 4 translators) live in the backlog and are not implemented in this slice.

## Error handling

Not applicable — this is a design artifact, not running code. The "errors" the slice can produce are:

- Schema mistakes (a canonical param that no surveyed device maps to, or a survey param with no canonical home). These surface during the user review and are fixed before approval.
- Out-of-scope features sneaking in. The "common core" boundary in `subtractive-ontology.md` and the Out-of-scope list above are the guardrails.

## Testing

Validation is **pure design review**, by explicit user choice (Q5-A). No fixtures, no JSON-Schema-validator runs, no round-trip translator. The user reads the survey + schema + ontology doc, sanity-checks each canonical param against ≥3 of the surveyed devices, and approves. Schema mistakes that survive review will surface when Phase 3 (expert alignment) or Phase 4 (translators) get built; the user has explicitly accepted that risk.

## Definition of done

- All eight survey docs written and consistent in template.
- `subtractive.schema.json` parses as valid JSON Schema (verified manually) and covers the in-scope param set above.
- `subtractive-ontology.md` walks through the schema with explicit common-core boundary notes.
- User has read the artifacts and explicitly approved the schema as the comm protocol between the inverse-synth engine and `keyboards-mcp`.
