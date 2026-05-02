# Subtractive Synthesis Canonical Ontology

Companion to [`subtractive.schema.json`](./subtractive.schema.json). Explains the design rules, the common-core boundary, and what each canonical param means acoustically. Audience: engineers wiring the analysis pipeline (`audio-analysis-mcp`) and the device side (`keyboards-mcp`) to this schema.

> **Status:** Schema version `0.1`. Reviewed by design only — no JSON Schema validator fixtures yet. See the [MVP spec](../docs/superpowers/specs/2026-05-02-subtractive-ontology-mvp.md) for the explicit risk acceptance.

## Schema architecture: base + engine

The schema is split across two files:

- [`synth-base.schema.json`](./synth-base.schema.json) — engine-agnostic base. Defines the top-level wrapper (`schema_version`, `engine`, `params`) and the `$defs` that every synth engine has in common: `ADSR`, `Voice`, `Master`, the `Envelopes` shape (with `amp` required), and a `CommonParams` container. Stays stable as new engine schemas are added.
- [`subtractive.schema.json`](./subtractive.schema.json) — extends the base via `allOf`. Constrains `engine` to `const: "subtractive"`, attaches a subtractive-specific `params` (`SubtractiveParams`), and adds engine-specific `$defs`: `Oscillators`, `Noise`, `Filter`, `LFO` with subtractive destination vocabulary, and `SubtractiveEnvelopes` (which extends the base `Envelopes` with an optional `filter` envelope).

```
synth-base.schema.json                         (engine-agnostic)
├── ADSR                                       reusable envelope shape
├── Voice                                      mono / poly / unison + glide + unison_voices
├── Master                                     volume_db
├── Envelopes  (requires amp)
└── CommonParams  (requires envelope, voice; master optional)
                              ▲
                              │ allOf
                              │
subtractive.schema.json                        (engine-specific)
├── SubtractiveParams = CommonParams + { osc, noise, filter, lfo, envelope.filter }
├── SubtractiveEnvelopes = base Envelopes + { filter (optional) }
├── Oscillators, Oscillator, SubOscillator
├── Noise
├── Filter (with lp)
├── LFO, LFOs
└── ModulationDestination (subtractive-specific 7-value enum)
```

Composition uses `allOf` plus `unevaluatedProperties: false` (Draft 2020-12). The `unevaluatedProperties` keyword is `allOf`-aware, so a subtractive instance with `envelope.filter` validates cleanly: the base's `Envelopes` accepts `amp` (required), the subtractive `SubtractiveEnvelopes` accepts the additional `filter`, and nothing outside that union is allowed.

Future engine schemas (FM, organ, wavetable, sample-based, …) will follow the same pattern: `allOf` the base, constrain `engine` to a different `const`, attach their own `params` extending `CommonParams`, and add engine-specific `$defs`.

**v0.1 instance compatibility:** any document valid against the previous monolithic schema validates against this split-schema design unchanged. The split is purely structural.

## Design rules

1. **Physical units, not normalized stand-ins.** Hz for frequency, ms for time, cents for fine pitch, dB for loudness, 0–1 ratios where there is no natural physical unit. The schema rejects 0–127 MIDI scaling — that is a device-side concern, handled in Phase 4 translators.
2. **Hierarchical names as object structure.** A canonical param like `filter.lp.cutoff_hz` is encoded as nested objects (`params.filter.lp.cutoff_hz`), not a flat string-keyed dict. The schema models the canonical structure directly.
3. **Optional sections for cross-device flexibility.** `osc.2`, `osc.sub`, `noise`, `envelope.filter`, `voice.glide_ms`, and the entire `master` section are optional. A minimal Prophet-5-style instance, a Muse-style instance, and a PolyBrute-12-style instance (no MIDI-addressable master volume) all validate without forcing irrelevant params.
4. **Common-core only.** A canonical param exists only if it can be sensibly mapped to most of the surveyed devices. Anything that requires deep per-device interpretation lives in the backlog, not the schema. See "What is not here" below.
5. **`x-unit` annotation.** A custom JSON Schema keyword (validators ignore unknown keywords) carries the physical unit. Future codegen will use this to emit Pydantic / TypeScript types with the right unit semantics. The keyword is not load-bearing for v0.1 validation.

## Canonical params and their units

The schema's `x-unit` annotations summarize each param's physical unit. Names that already encode the unit (`cutoff_hz`, `attack_ms`, `volume_db`, `detune_cents`, `pulse_width_pct`) carry it explicitly; names without a unit suffix (`level`, `resonance`, `sustain`, `depth`, `key_tracking`, `drive`, `envelope_amount`) are unitless 0–1 ratios (signed where noted).

The **Source** column indicates where each param's `$def` lives: `base` = `synth-base.schema.json` (shared with future engine schemas), `subtractive` = `subtractive.schema.json` (engine-specific). Future engine schemas will reuse every `base` row unchanged.

| Param | Unit | Range | Source | Notes |
|---|---|---|---|---|
| `osc.{1,2}.shape` | enum | `sine` / `saw` / `square` / `triangle` / `pulse` | subtractive | |
| `osc.{1,2}.level` | ratio | 0 – 1 | subtractive | mixer level into engine; 0 = muted |
| `osc.{1,2}.detune_cents` | cents | −1200 – 1200 | subtractive | fine pitch offset; 1200 cents = 1 octave |
| `osc.{1,2}.octave` | octaves (integer) | −3 – 3 | subtractive | coarse pitch offset, NOT semitones |
| `osc.{1,2}.pulse_width_pct` | percent | 0 – 100 | subtractive | only when `shape == 'pulse'`; 50 = perfect square, extremes = narrow pulse |
| `osc.sub.octave` | octaves (integer) | enum: −2 or −1 | subtractive | sub-osc octave below parent osc |
| `osc.sub.level` | ratio | 0 – 1 | subtractive | |
| `noise.color` | enum | `white` / `pink` | subtractive | spectral tilt; pink rolls off 3 dB/octave |
| `noise.level` | ratio | 0 – 1 | subtractive | |
| `filter.lp.cutoff_hz` | Hz | 20 – 20000 | subtractive | low-pass cutoff, log-distributed in practice |
| `filter.lp.resonance` | ratio | 0 – 1 | subtractive | Q / emphasis; 1 ≈ self-oscillation territory |
| `filter.lp.envelope_amount` | signed ratio | −1 – 1 | subtractive | filter-envelope depth into cutoff; negative inverts the envelope |
| `filter.lp.key_tracking` | ratio | 0 – 1 | subtractive | how much note pitch tracks cutoff (0 = none, 1 = full key follow) |
| `filter.lp.drive` | ratio | 0 – 1 | subtractive | pre-filter saturation amount |
| `envelope.{amp,filter}.attack_ms` | ms | 0 – 10000 | base (`ADSR`) | |
| `envelope.{amp,filter}.decay_ms` | ms | 0 – 20000 | base (`ADSR`) | |
| `envelope.{amp,filter}.sustain` | ratio | 0 – 1 | base (`ADSR`) | sustain level as **fraction of peak**, NOT dB |
| `envelope.{amp,filter}.release_ms` | ms | 0 – 20000 | base (`ADSR`) | |
| `lfo.1.rate_hz` | Hz | 0.01 – 100 | subtractive | LFO frequency, log-distributed in practice (sub-audio range) |
| `lfo.1.shape` | enum | `sine` / `triangle` / `square` / `saw` / `ramp` / `sample_hold` | subtractive | |
| `lfo.1.depth` | ratio | 0 – 1 | subtractive | global modulation depth |
| `lfo.1.target` | enum | (see destination vocabulary below) | subtractive | seven-value enum |
| `voice.mode` | enum | `mono` / `poly` / `unison` | base | |
| `voice.glide_ms` | ms | 0 – 5000 | base | portamento time |
| `voice.unison_voices` | integer count | 1 – 16 | base | active voices in unison; ignored if `mode != 'unison'` |
| `master.volume_db` | dB | −60 – 6 | base | required *if* `master` is present (the whole `master` section is optional). Referenced to the device's nominal full-scale output (0 dB = unity, +6 dB = mild boost, −60 dB ≈ silent). |

Conventions:

- **`ratio`** is a unitless 0–1 normalized scale (or signed −1 to 1 where noted). The schema deliberately rejects 0–127 MIDI scaling for canonical params — that's a device-side concern.
- **`cents`** = 1/100 of a semitone; 1200 cents = 1 octave. Use `detune_cents` for fine pitch and `octave` (integer) for coarse pitch — they are independent.
- **`Hz`** is used for both filter cutoff (audio range) and LFO rate (sub-audio range); the range bounds disambiguate.
- **`ms`** time params have wide ranges (up to 10–20 s) to accommodate long pad envelopes.
- **`dB`** for `volume_db` is referenced to nominal full-scale output of the device, not absolute SPL.
- The `x-unit` JSON Schema keyword in `subtractive.schema.json` is the machine-readable source of truth; this table mirrors it for human readers.

## Common-core LFO destination vocabulary

The `ModulationDestination` enum in the schema is exactly seven values:

| Destination | Acoustic meaning | Why common-core |
|---|---|---|
| `osc.pitch` | LFO modulates oscillator pitch — vibrato. | Every surveyed device exposes this. |
| `osc.shape` | LFO modulates wave shape — texture/timbre wobble. | All eight devices. |
| `osc.pwm` | LFO modulates pulse width — classic PWM movement. | All eight devices when shape == pulse. |
| `osc.amp` | LFO modulates oscillator amplitude — tremolo at the source. | All eight devices. |
| `filter.cutoff` | LFO sweeps the LP cutoff — wah/swell movement. | Every surveyed device exposes this; it is the canonical "filter wobble". |
| `filter.resonance` | LFO modulates resonance — uncommon but musical. | Reachable on Muse, PolyBrute 12, and Summit via their mod matrices (three of eight devices — barely common-core, the weakest entry in this enum). The other five (Prophet-6, JUNO-X, Prophet-5, OB-X8, Minilogue XD) expose only filter cutoff as a direct LFO destination, with no panel- or matrix-level path to resonance. Translators on those devices must mark this destination as unsupported. Kept in the canonical enum because the analysis side may genuinely detect it. |
| `amp.level` | LFO modulates final VCA — output tremolo. | All eight devices, often as a hard-wired LFO destination. |

Full mod-matrix destinations (envelope-as-source, mod wheel, aftertouch, per-osc-pitch, FX sends, etc.) are deferred — see the backlog.

## Per-section walkthrough

### Oscillators (`params.osc`)

`osc.1` is required; `osc.2` and `osc.sub` are optional. Each oscillator has `shape`, `level`, `detune_cents`, `octave`, and `pulse_width_pct` (used only when `shape == 'pulse'`).

Mapping notes across the surveyed devices:
- **Prophet-6, Prophet-5, OB-X8, Moog Muse:** two analog VCOs; `shape` enums map directly. PWM is per-osc continuous.
- **JUNO-X (Analog Synth):** **single oscillator section with mixed sources** (pulse / Super-SAW + sawtooth + square sub-osc + noise) per the JUNO-X survey, not two distinct oscillators. The canonical schema's `osc.2.*` fields therefore have no clean JUNO-X mapping (coverage matrix marks them ✗). Translators should populate only `osc.1.*` and `osc.sub.*` for the JUNO-X. JUNO-X exposes more shape variants (e.g. `SAW-DW`) — these collapse to `saw` for canonical purposes.
- **PolyBrute 12:** two oscillators with extra wave-shaping (Brute / Metalizer / Ultrasaw). The wave-shaping params are out of common-core; the oscillator's underlying shape still maps to `shape`.
- **Minilogue XD:** two VCOs, plus the Multi-Engine which is excluded from this ontology entirely.
- **Novation Summit:** three NCOs per voice. Common-core uses only osc.1 and osc.2; the third oscillator is captured in the survey but not in the schema.

### Noise (`params.noise`)

Optional. `color` is `white` or `pink`. Devices that only produce one color set `color` to whichever they ship.

### Filter (`params.filter.lp`)

Required. The schema models a single low-pass filter with `cutoff_hz`, `resonance`, `envelope_amount` (signed), `key_tracking`, `drive`. Multi-mode filters (LP/HP/BP/Notch on OB-X8 / Summit) and dual filters (PolyBrute 12) are out of common-core — the canonical schema treats them as their LP setting.

### Envelopes (`params.envelope`)

The `Envelopes` shape lives in `synth-base.schema.json` and requires `amp`; the subtractive schema extends it via `SubtractiveEnvelopes` to add an optional `filter` envelope. Both reuse the base `ADSR` `$def` (attack/decay/release in ms, sustain as a 0–1 ratio of peak). When future engine schemas (FM, organ, …) are added, they will reuse the same `ADSR` shape and add their own engine-specific envelope slots (e.g. per-operator envelopes for FM).

The PolyBrute 12 has a third per-voice envelope which is unrepresented in the canonical params; analysis-side outputs simply do not target it. The Moog Muse has only two per-voice envelopes (Filter Env + VCA Env per its survey); two additional envelopes added in firmware 1.4 are global mod-matrix sources, not per-voice ADSRs, and so are also unrepresented.

### LFO (`params.lfo.1`)

One LFO is required: `rate_hz`, `shape`, `depth`, `target`. Devices with multiple LFOs (JUNO-X, Muse, PolyBrute 12, Summit) expose only their primary LFO through this canonical slot for v0.1.

### Voice and master (`params.voice`, `params.master`)

`Voice` and `Master` both live in `synth-base.schema.json` — they are engine-agnostic and apply unchanged to every synth engine.

`voice.mode` is required and is one of `mono`, `poly`, `unison`. `voice.glide_ms` and `voice.unison_voices` are optional. The whole `master` section is **optional** — PolyBrute 12 and Minilogue XD expose master volume only as a non-programmable physical pot (not addressable over MIDI), so requiring it would reject canonical instances for those devices. When `master` is present, `master.volume_db` is required and is in dB referenced to the device's nominal full-scale output.

## What is not here, and why

The "common-core" rule deliberately omits features that some devices have but others do not. These all have stub entries in the [backlog](../docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md):

- **Cross-modulation, hard sync, ring modulation, oscillator FM** — excluded from the schema regardless of which devices have them. Most surveyed devices expose at least one of these (PolyBrute 12, JUNO-X Analog Synth, Muse, OB-X8, Summit have a rich set; Prophet-6 has hard sync via NRPN; Minilogue XD has SYNC / RING / CROSS MOD DEPTH; Prophet-5's Poly-Mod hits a similar surface). Each device implements them differently with different routing topologies, so any common-core canonicalization would force per-device fallbacks. Surveys document each device's specific support; translators handle the gap by leaving these unset.
- **Dual filters in series/parallel/morph (PolyBrute 12), multi-mode filter (OB-X8, Summit)** — modeled as a single LP for v0.1.
- **Multiple LFOs and full mod matrix** — only `lfo.1` and the seven canonical destinations for v0.1.
- **Third (modulation) envelope** — analysis side does not yet predict it.
- **Vintage-knob noise simulation, Brute wave-shaping, NCO algorithms** — device-specific flavor that does not survive translation.
- **Effects bus parameters (chorus, delay, reverb, EQ)** — post-engine, belongs in a separate fx ontology.
- **Aftertouch / poly-aftertouch routing, velocity-amount params, per-voice modulation** — analysis-side cannot yet identify these from audio reliably.

When the inverse-synth pipeline matures past its first iteration and downstream consumers (translators, codegen, fixtures) start asking for these features, version-bump the schema and pull from the backlog.
