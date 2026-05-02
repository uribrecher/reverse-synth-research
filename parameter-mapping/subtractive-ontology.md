# Subtractive Synthesis Canonical Ontology

Companion to [`subtractive.schema.json`](./subtractive.schema.json). Explains the design rules, the common-core boundary, and what each canonical param means acoustically. Audience: engineers wiring the analysis pipeline (`audio-analysis-mcp`) and the device side (`keyboards-mcp`) to this schema.

> **Status:** Schema version `0.1`. Reviewed by design only — no JSON Schema validator fixtures yet. See the [MVP spec](../docs/superpowers/specs/2026-05-02-subtractive-ontology-mvp.md) for the explicit risk acceptance.

## Design rules

1. **Physical units, not normalized stand-ins.** Hz for frequency, ms for time, cents for fine pitch, dB for loudness, 0–1 ratios where there is no natural physical unit. The schema rejects 0–127 MIDI scaling — that is a device-side concern, handled in Phase 4 translators.
2. **Hierarchical names as object structure.** A canonical param like `filter.lp.cutoff_hz` is encoded as nested objects (`params.filter.lp.cutoff_hz`), not a flat string-keyed dict. The schema models the canonical structure directly.
3. **Optional sections for cross-device flexibility.** `osc.2`, `osc.sub`, `noise`, `envelope.filter`, and `voice.glide_ms` are all optional. A minimal Prophet-5-style instance and a Muse-style instance both validate without forcing irrelevant params.
4. **Common-core only.** A canonical param exists only if it can be sensibly mapped to most of the surveyed devices. Anything that requires deep per-device interpretation lives in the backlog, not the schema. See "What is not here" below.
5. **`x-unit` annotation.** A custom JSON Schema keyword (validators ignore unknown keywords) carries the physical unit. Future codegen will use this to emit Pydantic / TypeScript types with the right unit semantics. The keyword is not load-bearing for v0.1 validation.

## Canonical params and their units

The schema's `x-unit` annotations summarize each param's physical unit. Names that already encode the unit (`cutoff_hz`, `attack_ms`, `volume_db`, `detune_cents`, `pulse_width_pct`) carry it explicitly; names without a unit suffix (`level`, `resonance`, `sustain`, `depth`, `key_tracking`, `drive`, `envelope_amount`) are unitless 0–1 ratios (signed where noted).

| Param | Unit | Range | Notes |
|---|---|---|---|
| `osc.{1,2}.shape` | enum | `sine` / `saw` / `square` / `triangle` / `pulse` | |
| `osc.{1,2}.level` | ratio | 0 – 1 | mixer level into engine; 0 = muted |
| `osc.{1,2}.detune_cents` | cents | −1200 – 1200 | fine pitch offset; 1200 cents = 1 octave |
| `osc.{1,2}.octave` | octaves (integer) | −3 – 3 | coarse pitch offset, NOT semitones |
| `osc.{1,2}.pulse_width_pct` | percent | 0 – 100 | only when `shape == 'pulse'`; 50 = perfect square, extremes = narrow pulse |
| `osc.sub.octave` | octaves (integer) | enum: −2 or −1 | sub-osc octave below parent osc |
| `osc.sub.level` | ratio | 0 – 1 | |
| `noise.color` | enum | `white` / `pink` | spectral tilt; pink rolls off 3 dB/octave |
| `noise.level` | ratio | 0 – 1 | |
| `filter.lp.cutoff_hz` | Hz | 20 – 20000 | low-pass cutoff, log-distributed in practice |
| `filter.lp.resonance` | ratio | 0 – 1 | Q / emphasis; 1 ≈ self-oscillation territory |
| `filter.lp.envelope_amount` | signed ratio | −1 – 1 | filter-envelope depth into cutoff; negative inverts the envelope |
| `filter.lp.key_tracking` | ratio | 0 – 1 | how much note pitch tracks cutoff (0 = none, 1 = full key follow) |
| `filter.lp.drive` | ratio | 0 – 1 | pre-filter saturation amount |
| `envelope.{amp,filter}.attack_ms` | ms | 0 – 10000 | |
| `envelope.{amp,filter}.decay_ms` | ms | 0 – 20000 | |
| `envelope.{amp,filter}.sustain` | ratio | 0 – 1 | sustain level as **fraction of peak**, NOT dB |
| `envelope.{amp,filter}.release_ms` | ms | 0 – 20000 | |
| `lfo.1.rate_hz` | Hz | 0.01 – 100 | LFO frequency, log-distributed in practice (sub-audio range) |
| `lfo.1.shape` | enum | `sine` / `triangle` / `square` / `saw` / `ramp` / `sample_hold` | |
| `lfo.1.depth` | ratio | 0 – 1 | global modulation depth |
| `lfo.1.target` | enum | (see destination vocabulary below) | seven-value enum |
| `voice.mode` | enum | `mono` / `poly` / `unison` | |
| `voice.glide_ms` | ms | 0 – 5000 | portamento time |
| `voice.unison_voices` | integer count | 1 – 16 | active voices in unison; ignored if `mode != 'unison'` |
| `master.volume_db` | dB | −60 – 6 | referenced to the device's nominal full-scale output (0 dB = unity, +6 dB = mild boost, −60 dB ≈ silent) |

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
| `filter.resonance` | LFO modulates resonance — uncommon but musical. | Six of eight devices expose it directly; the others (Prophet-5, OB-X8) reach it via mod matrix. |
| `amp.level` | LFO modulates final VCA — output tremolo. | All eight devices, often as a hard-wired LFO destination. |

Full mod-matrix destinations (envelope-as-source, mod wheel, aftertouch, per-osc-pitch, FX sends, etc.) are deferred — see the backlog.

## Per-section walkthrough

### Oscillators (`params.osc`)

`osc.1` is required; `osc.2` and `osc.sub` are optional. Each oscillator has `shape`, `level`, `detune_cents`, `octave`, and `pulse_width_pct` (used only when `shape == 'pulse'`).

Mapping notes across the surveyed devices:
- **Prophet-6, Prophet-5, OB-X8, Moog Muse:** two analog VCOs; `shape` enums map directly. PWM is per-osc continuous.
- **JUNO-X (Analog Synth):** two virtual analog oscillators. JUNO-X exposes more shape variants (e.g. `SAW-DW`) — these collapse to `saw` for canonical purposes.
- **PolyBrute 12:** two oscillators with extra wave-shaping (Brute / Metalizer / Ultrasaw). The wave-shaping params are out of common-core; the oscillator's underlying shape still maps to `shape`.
- **Minilogue XD:** two VCOs, plus the Multi-Engine which is excluded from this ontology entirely.
- **Novation Summit:** three NCOs per voice. Common-core uses only osc.1 and osc.2; the third oscillator is captured in the survey but not in the schema.

### Noise (`params.noise`)

Optional. `color` is `white` or `pink`. Devices that only produce one color set `color` to whichever they ship.

### Filter (`params.filter.lp`)

Required. The schema models a single low-pass filter with `cutoff_hz`, `resonance`, `envelope_amount` (signed), `key_tracking`, `drive`. Multi-mode filters (LP/HP/BP/Notch on OB-X8 / Summit) and dual filters (PolyBrute 12) are out of common-core — the canonical schema treats them as their LP setting.

### Envelopes (`params.envelope`)

`envelope.amp` is required; `envelope.filter` is optional. Both are ADSR with attack/decay/release in ms and sustain as a 0–1 ratio. Devices with a third "modulation envelope" (Moog Muse, PolyBrute 12) leave that envelope unrepresented in the canonical params; analysis-side outputs simply do not target it.

### LFO (`params.lfo.1`)

One LFO is required: `rate_hz`, `shape`, `depth`, `target`. Devices with multiple LFOs (JUNO-X, Muse, PolyBrute 12, Summit) expose only their primary LFO through this canonical slot for v0.1.

### Voice and master (`params.voice`, `params.master`)

`voice.mode` is required and is one of `mono`, `poly`, `unison`. `voice.glide_ms` and `voice.unison_voices` are optional. `master.volume_db` is required, in dB referenced to the device's nominal output.

## What is not here, and why

The "common-core" rule deliberately omits features that some devices have but others do not. These all have stub entries in the [backlog](../docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md):

- **Cross-modulation, hard sync, ring modulation, oscillator FM** — present on PolyBrute 12, JUNO-X Analog Synth, Muse, OB-X8, Summit; absent on Prophet-6, Minilogue XD, Prophet-5 (without poly-mod). Including them would force per-device fallbacks into the schema.
- **Dual filters in series/parallel/morph (PolyBrute 12), multi-mode filter (OB-X8, Summit)** — modeled as a single LP for v0.1.
- **Multiple LFOs and full mod matrix** — only `lfo.1` and the seven canonical destinations for v0.1.
- **Third (modulation) envelope** — analysis side does not yet predict it.
- **Vintage-knob noise simulation, Brute wave-shaping, NCO algorithms** — device-specific flavor that does not survive translation.
- **Effects bus parameters (chorus, delay, reverb, EQ)** — post-engine, belongs in a separate fx ontology.
- **Aftertouch / poly-aftertouch routing, velocity-amount params, per-voice modulation** — analysis-side cannot yet identify these from audio reliably.

When the inverse-synth pipeline matures past its first iteration and downstream consumers (translators, codegen, fixtures) start asking for these features, version-bump the schema and pull from the backlog.
