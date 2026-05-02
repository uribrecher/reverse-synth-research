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
