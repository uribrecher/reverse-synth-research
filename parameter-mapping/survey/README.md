# Subtractive-Synth Survey — Index

Cross-vendor parameter inventory for eight subtractive polysynths. Used to inform the design of `../subtractive.schema.json`. Scope: subtractive engines only — devices whose primary architecture is osc → mixer → filter → amp with LFO/envelope modulation. Per the MVP spec, this survey is bounded to exactly eight devices; widening the canon is in the backlog.

## Devices

| # | Device | Manufacturer | Engine type | Polyphony | File |
|---|---|---|---|---|---|
| 1 | Prophet-6 | Sequential | analog | 6 | [prophet-6.md](./prophet-6.md) |
| 2 | JUNO-X (Analog Synth engine layer) | Roland | virtual analog | up to 8 per scene | [juno-x.md](./juno-x.md) |
| 3 | Prophet-5 Rev 4 | Sequential | analog | 5 | [prophet-5.md](./prophet-5.md) |
| 4 | OB-X8 | Oberheim | analog | 8 | [ob-x8.md](./ob-x8.md) |
| 5 | Muse | Moog | analog | 8 | [moog-muse.md](./moog-muse.md) |
| 6 | PolyBrute 12 | Arturia | analog | 12 | [polybrute-12.md](./polybrute-12.md) |
| 7 | Minilogue XD (subtractive layer) | Korg | hybrid | 4 | [minilogue-xd.md](./minilogue-xd.md) |
| 8 | Summit | Novation | hybrid (poly) | 16 | [novation-summit.md](./novation-summit.md) |

## Sourcing approach

For every device, primary sources are the manufacturer's user manual and MIDI implementation chart. Cross-checks for the two devices we control directly via `keyboards-mcp`:

- **Prophet-6:** cross-check against `../../../keyboards-mcp/src/keyboard_models/sequential_circuits/prophet_6/midi-map.ts`.
- **JUNO-X:** cross-check against `../../../keyboards-mcp/src/keyboard_models/roland/juno_x/engines/analog-synth.ts` (the Analog Synth engine — the only subtractive engine in the JUNO-X).

Survey docs keep device-native parameter names. The mapping to canonical names happens in `../subtractive-ontology.md`.

## Survey template

Each per-device survey doc follows this template:

```markdown
# <Device Name> — Parameter Survey

**Manufacturer:** <name>
**Year:** <year>
**Engine type:** <analog | virtual analog | hybrid>
**Polyphony:** <n voices>
**Sources:**
- <source 1: URL or citation>
- <source 2: URL or citation>

## Signal path

<one paragraph: oscillators → mixer → filter → amp, plus modulation>

## Parameters

### Oscillators
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

### Mixer
(same column shape)

### Filter
(same column shape)

### Envelopes
(same column shape)

### LFO
(same column shape)

### Voice / Master
(same column shape)

## Modulation routing

<destinations the device's LFO(s) and mod matrix can target; sources are bullet list>

## Quirks

- <knob taper, log vs linear, MIDI 0–127 quantization, special "off" values, asymmetric ranges>
```

## Common-core boundary

This survey informs a "common-core" canonical ontology — see the MVP spec for the rule. Features supported by some surveyed devices but excluded from common-core (cross-mod, sync, ring mod, dual filters, multi-mode filter morphing, full mod-matrix slots, vintage knob simulation) are still recorded in survey docs under quirks/routing — they just do not get canonical params in `../subtractive.schema.json`. They are catalogued in `../../docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md`.

## Coverage matrix (cross-check sweep)

Each canonical param vs. the eight surveyed devices. ✓ = direct support, ≈ = near-equivalent (concept present but in different units, constrained range, single-source-of-many, or NRPN/SysEx-only addressability), ✗ = absent. Count column is total ✓+≈. Per the MVP spec's definition of done, every canonical param must hit ≥3 devices. The sweep applies the rule: ≥3 keeps the param, 2 demotes to optional, 0–1 removes. **All canonical params survive the sweep — no demotions, no removals.**

| Canonical param | Unit | Prophet-6 | JUNO-X | Prophet-5 | OB-X8 | Muse | PolyBrute 12 | Minilogue XD | Summit | Count |
|---|---|---|---|---|---|---|---|---|---|---|
| osc.1.shape | enum | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| osc.1.level | ratio (0–1) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| osc.1.detune_cents | cents | ✗ | ≈ | ✗ | ✗ | ≈ | ≈ | ✓ | ✓ | 5 |
| osc.1.octave | octaves (int) | ≈ | ≈ | ≈ | ≈ | ✓ | ≈ | ✓ | ✓ | 8 |
| osc.1.pulse_width_pct | % | ✓ | ≈ | ✓ | ✓ | ✓ | ✓ | ≈ | ≈ | 8 |
| osc.2.shape | enum | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 7 |
| osc.2.level | ratio (0–1) | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 7 |
| osc.2.detune_cents | cents | ✓ | ✗ | ✓ | ✓ | ≈ | ≈ | ✓ | ✓ | 7 |
| osc.2.octave | octaves (int) | ≈ | ✗ | ≈ | ≈ | ✓ | ≈ | ✓ | ✓ | 7 |
| osc.2.pulse_width_pct | % | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ≈ | ≈ | 7 |
| osc.sub.octave | octaves (int) | ≈ | ≈ | ✗ | ✗ | ✗ | ≈ | ✗ | ✗ | 3 |
| osc.sub.level | ratio (0–1) | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | 3 |
| noise.color | enum | ≈ | ≈ | ≈ | ≈ | ≈ | ≈ | ✗ | ≈ | 7 |
| noise.level | ratio (0–1) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | 7 |
| filter.lp.cutoff_hz | Hz | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| filter.lp.resonance | ratio (0–1) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| filter.lp.envelope_amount | ratio (−1 to 1) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| filter.lp.key_tracking | ratio (0–1) | ✓ | ✓ | ≈ | ✓ | ≈ | ✓ | ≈ | ✓ | 8 |
| filter.lp.drive | ratio (0–1) | ≈ | ✗ | ✗ | ✗ | ✓ | ✓ | ✓ | ✓ | 5 |
| envelope.amp.attack_ms | ms | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| envelope.amp.decay_ms | ms | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| envelope.amp.sustain | ratio (0–1) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| envelope.amp.release_ms | ms | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| envelope.filter.attack_ms | ms | ✓ | ≈ | ✓ | ✓ | ✓ | ✓ | ≈ | ≈ | 8 |
| envelope.filter.decay_ms | ms | ✓ | ≈ | ✓ | ✓ | ✓ | ✓ | ≈ | ≈ | 8 |
| envelope.filter.sustain | ratio (0–1) | ✓ | ≈ | ✓ | ✓ | ✓ | ✓ | ✗ | ≈ | 7 |
| envelope.filter.release_ms | ms | ✓ | ≈ | ✓ | ✓ | ✓ | ✓ | ✗ | ≈ | 7 |
| lfo.1.rate_hz | Hz | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| lfo.1.shape | enum | ✓ | ✓ | ≈ | ✓ | ✓ | ≈ | ✓ | ≈ | 8 |
| lfo.1.depth | ratio (0–1) | ✓ | ≈ | ✓ | ≈ | ✓ | ≈ | ✓ | ≈ | 8 |
| lfo.1.target | enum | ✓ | ✓ | ✓ | ✓ | ≈ | ≈ | ✓ | ✓ | 8 |
| voice.mode | enum | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ≈ | ✓ | 8 |
| voice.glide_ms | ms | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| voice.unison_voices | int (1–16) | ✓ | ≈ | ✓ | ✓ | ≈ | ≈ | ≈ | ≈ | 8 |
| master.volume_db | dB | ≈ | ≈ | ≈ | ≈ | ≈ | ✗ | ✗ | ≈ | 6 |

Unit column reflects the schema's `x-unit` annotation (in `../subtractive.schema.json`). `ratio (0–1)` means a unitless normalized scale; `ratio (−1 to 1)` is signed; `cents` = 1/100 semitone; `octaves (int)` is an integer octave count; `enum` means a string-valued discrete vocabulary (the schema's `enum` field has the allowed values). See `../subtractive-ontology.md` for the per-param ranges and acoustic meanings.

### Notable ≈ judgement calls

- **Pitch-fine vs semitones vs cents.** Several devices expose pitch tuning as semitones (P6 `Osc 1 Freq`, JUNO-X `OSC PITCH`, P5 `Osc A Freq`, OB-X8 `OSC 1/2 FREQ`, PB12 `VCO Tune`) rather than cents. Counted as ≈ for `detune_cents` per the rule that "cents vs semitones" near-equivalents count as ✓.
- **Octave switches vs continuous semitone ranges.** `osc.*.octave` is canonically an integer octave count. Devices with continuous semitone-stepped frequency knobs spanning multiple octaves (P6, JUNO-X, P5, OB-X8, PB12) are scored ≈; devices with explicit foot-switch octave selectors (Muse, Mlg-XD, Summit) are ✓.
- **Pulse width on shape-morph oscillators.** Mlg-XD `VCO SHAPE` and Summit `Osc N Shape` are continuously-variable waveshape morphs; on a pulse waveform the SHAPE knob acts as duty-cycle. Counted as ≈ for `pulse_width_pct`. JUNO-X has `PULSE WIDTH MOD/SSAW DETUNE` SysEx-only — also ≈.
- **Sub-osc octave constrained to -1.** Schema permits {-2, -1}; P6 / JUNO-X / PB12 sub-oscs are fixed one octave below the parent, equivalent to -1 only. Concept and value-within-allowed-range present, so ≈.
- **Noise color.** Schema enum is `["white", "pink"]`. Most devices ship white noise only with no color selector — counted as ≈ (color "fixed" to white). PB12 has a continuous `Noise Color` knob (rumble→white) which is also ≈ since it's not a discrete enum. Mlg-XD has no analog noise (digital MULTI ENGINE only) — ✗.
- **JUNO-X shared ADSR.** AnalogSynth has a single shared ADSR that drives both VCA and (scaled by FLT ENV DEPTH) the LPF cutoff. Filter-envelope rows are scored ≈ on JUNO-X because the same envelope plays both roles via a depth knob.
- **Mlg-XD `EG` is AD-only, not ADSR.** Shared 2-stage AD envelope time-shared between filter and pitch via `EG TARGET`. `envelope.filter.attack_ms` and `envelope.filter.decay_ms` are ≈ (AD present); `envelope.filter.sustain` and `envelope.filter.release_ms` are ✗ (no S/R stages on this envelope).
- **Summit Mod Env 1 as filter envelope.** No dedicated filter envelope — Mod Env 1 fills the role via the `Filter Source` switch. Mod Env 1's AHDSR sliders are top-level CCs, scored ≈ for `envelope.filter.*`.
- **Mlg-XD voice mode lacks MONO.** Voice modes are `POLY/UNISON/CHORD/ARP-LATCH` per Owner's Manual p.15 — no dedicated MONO. Closest analogue is UNISON with depth=0. Scored ≈.
- **`voice.unison_voices` and CC addressability.** P6, P5, OB-X8 expose explicit voice-count CCs (or NRPNs); JUNO-X / Muse / PB12 / Mlg-XD / Summit have unison modes but no top-level voice-count CC published — scored ≈.
- **`lfo.1.depth` and `lfo.1.target` panel-fixed vs mod-matrix.** Devices with mod-matrix-only LFO routing (Muse, PB12) have the concept but no panel-fixed CCs for destination switches — ≈. Devices with multi-bus depth (OB-X8, Summit) or per-destination depth knobs (JUNO-X) are also ≈ for `depth` since the canonical schema models a single global depth.
- **`filter.lp.drive` is "pre-filter saturation."** P6's `Distortion Amount` is post-VCA — concept of drive present but not in the canonical position; ≈. JUNO-X / P5 / OB-X8 have no in-engine drive parameter — ✗. Muse `Clipping Level`, PB12 `Ladder Distortion`, Mlg-XD `DRIVE`, Summit `Filter Overdrive` are all pre-filter — ✓.
- **`master.volume_db`.** PB12 and Mlg-XD have non-programmable master-volume knobs not exposed via CC; scored ✗. Summit `PatchLevel` is NRPN-only — ≈. P6/JUNO-X/P5/OB-X8/Muse expose CC 7 (or AMP LEVEL CC 110 for JUNO-X) but as a 0–127 level rather than dB — ≈ since unit differs.

### Sweep result summary

- ≥3 ✓: **all 35 canonical params** (lowest counts are `osc.sub.octave` = 3 and `osc.sub.level` = 3, exactly at threshold; both already optional in the schema).
- 2 ✓: none — no demotions applied.
- 0–1 ✓: none — no removals applied.

The schema (`../subtractive.schema.json`) and ontology companion (`../subtractive-ontology.md`) are unchanged by this sweep.
