# Prophet-6 — Parameter Survey

**Manufacturer:** Sequential
**Year:** 2018
**Engine type:** analog
**Polyphony:** 6
**Sources:**
- Sequential, *Prophet-6 Operation Manual* v2.1, Appendix C (MIDI Implementation): https://sequential.com/wp-content/uploads/2021/02/Prophet-6-Operation-Manual-2.1.pdf
- Sequential, *Prophet-6 Documentation* product page: https://sequential.com/support/download/prophet-6-documentation/
- Cross-check: keyboards-mcp Prophet-6 midi-map (`../../../keyboards-mcp/src/keyboard_models/sequential_circuits/prophet_6/midi-map.ts`)

## Signal path

Two analog VCOs (Oscillator 1, Oscillator 2) feed a mixer that also receives a square-wave sub-oscillator one octave below Osc 1 plus a noise generator. The mixer output passes into a Curtis-style 4-pole resonant low-pass filter, then into a 2-pole resonant high-pass filter, then into a VCA. A dedicated 4-stage ADSR envelope drives the filter (with bipolar amount and a separate signed Poly Mod amount); a second 4-stage ADSR drives the VCA. A single multi-shape LFO (triangle, sawtooth, reverse sawtooth, square, random; plus a hidden noise mode at random + max rate) is routable to a fixed set of destinations via on/off switches. A separate Poly Mod section provides two extra modulation sources (Filter Envelope, Osc 2) into a fixed destination set — this is out-of-common-core and recorded under Modulation routing / Quirks. Effects (delay, reverb, distortion, etc.) are post-engine and out of scope for this survey except for the single CC-addressable Distortion drive parameter, which is included for completeness.

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit MIDI Continuous Controllers; the underlying program parameter on the device may have higher internal resolution accessible via NRPN (e.g. Osc 1 Shape NRPN range 0–254). The CC encoding maps the full knob travel onto 0–127 with hardware-defined non-linear taper (knob taper). Discrete switches addressed via CC use a threshold convention: `0 = Off / first label`, `≥ 1 = On / next label` (the midi-map records `min: 0, max: 1` for toggles; `max: N` for N+1-way discrete enums where the label table enumerates each integer step from 0 to N).

### Oscillator 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 1 Freq | continuous | 0–127 (raw MIDI CC) | semitones (knob taper) | — | Oscillator 1 base frequency over a 9-octave range (16 Hz – 8 kHz, semitone steps; underlying NRPN range 0–60) |
| Osc 1 Level | continuous | 0–127 (raw MIDI CC) | level | — | Oscillator 1 level into the mixer |
| Osc 1 Shape | continuous | 0–127 (raw MIDI CC) | morph | — | Continuously-variable waveshape (triangle → sawtooth → pulse). NRPN range 0–254. |
| Osc 1 Pulse Width | continuous | 0–127 (raw MIDI CC) | pulse-width | — | Pulse width when Osc 1 is in pulse mode (square at 12 o'clock, narrow pulse at extremes). NRPN range 0–255. |

Out-of-common-core (NRPN-only on Prophet-6, not in the keyboards-mcp midi-map): `Osc 1 Sync` (NRPN 1, 0–1) — hard-syncs Osc 1 to Osc 2. Recorded under Quirks.

### Oscillator 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 2 Freq | continuous | 0–127 (raw MIDI CC) | semitones (knob taper) | — | Oscillator 2 base frequency (NRPN range 0–60, semitone steps) |
| Osc 2 Freq Fine | continuous | 0–127 (raw MIDI CC) | cents | — | Fine tune ±50 cents (1/2 semitone), centered at 64 (NRPN range 0–254) |
| Osc 2 Level | continuous | 0–127 (raw MIDI CC) | level | — | Oscillator 2 level into the mixer |
| Osc 2 Shape | continuous | 0–127 (raw MIDI CC) | morph | — | Continuously-variable waveshape (triangle → sawtooth → pulse). NRPN range 0–254. |
| Osc 2 Pulse Width | continuous | 0–127 (raw MIDI CC) | pulse-width | — | Pulse width when Osc 2 is in pulse mode (NRPN range 0–255) |

Out-of-common-core (NRPN-only on Prophet-6, not in the keyboards-mcp midi-map): `Osc 2 Low Freq` (NRPN 10, 0–1) — drops Osc 2 into LFO range; `Osc 2 Key On/Off` (NRPN 11, 0–1) — disables keyboard tracking on Osc 2 (drone / mod-source mode).

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Sub Osc Level | continuous | 0–127 (raw MIDI CC) | level | — | Square sub-oscillator one octave below Osc 1 (CC 8) |

Out-of-common-core (NRPN-only on Prophet-6, not in the keyboards-mcp midi-map): `Noise Level` (NRPN 32, 0–127) — white noise into mixer; `Slop` (NRPN 33, 0–127) — randomized per-voice detuning to emulate vintage oscillator drift (vintage-knob behavior).

### Low-pass filter
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Low-Pass Freq | continuous | 0–127 (raw MIDI CC) | cutoff (knob taper) | — | 4-pole low-pass cutoff frequency (CC 102; NRPN range 0–164) |
| Low-Pass Resonance | continuous | 0–127 (raw MIDI CC) | resonance | — | Low-pass resonance (CC 103; NRPN range 0–255) |
| Low-Pass Key Amt | continuous | 0–127 (raw MIDI CC) | tracking | — | Keyboard tracking amount into low-pass cutoff (CC 104; underlying NRPN is 0–2 stepped — see Quirks) |
| Low-Pass Vel On/Off | toggle | 0–1 (raw MIDI CC) | enum | Off / On | Velocity sensitivity for filter envelope routing into low-pass (CC 105) |
| Low-Pass Env Amt | continuous | 0–127 (raw MIDI CC) | bipolar amount | — | Filter envelope amount into low-pass cutoff (CC 47). Bipolar on the panel knob (12 o'clock = 0); the CC value collapses bipolar onto 0–127 with 64 = center (out-of-common-core: signed amount knob behavior — see Quirks) |

### High-pass filter
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| High-Pass Freq | continuous | 0–127 (raw MIDI CC) | cutoff (knob taper) | — | High-pass cutoff frequency (CC 106; NRPN range 0–164) |
| High-Pass Resonance | continuous | 0–127 (raw MIDI CC) | resonance | — | High-pass resonance (CC 107; NRPN range 0–255) |
| High-Pass Key Amt | continuous | 0–127 (raw MIDI CC) | tracking | — | Keyboard tracking amount into high-pass cutoff (CC 108; underlying NRPN is 0–2 stepped) |
| High-Pass Vel On/Off | toggle | 0–1 (raw MIDI CC) | enum | Off / On | Velocity sensitivity for filter envelope routing into high-pass (CC 109) |
| High-Pass Env Amt | continuous | 0–127 (raw MIDI CC) | bipolar amount | — | Filter envelope amount into high-pass cutoff (CC 54). Bipolar on the panel knob with CC 64 = center (out-of-common-core: signed amount knob behavior) |

### Filter envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter Env Attack | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | Filter envelope attack time (CC 50) |
| Filter Env Decay | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | Filter envelope decay time (CC 51) |
| Filter Env Sustain | continuous | 0–127 (raw MIDI CC) | level | — | Filter envelope sustain level (CC 52) |
| Filter Env Release | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | Filter envelope release time (CC 53) |

### Amp envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCA Env Amt | continuous | 0–127 (raw MIDI CC) | level | — | VCA envelope amount (CC 40). Acts as overall envelope-driven gain into the VCA. |
| VCA Vel Amt | continuous | 0–127 (raw MIDI CC) | sensitivity | — | VCA velocity sensitivity amount (CC 41) |
| VCA Env Attack | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | VCA envelope attack time (CC 43) |
| VCA Env Decay | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | VCA envelope decay time (CC 44) |
| VCA Env Sustain | continuous | 0–127 (raw MIDI CC) | level | — | VCA envelope sustain level (CC 45) |
| VCA Env Release | continuous | 0–127 (raw MIDI CC) | time (knob taper) | — | VCA envelope release time (CC 46) |

### LFO
The Prophet-6 LFO is exposed via NRPN only (it is NOT in the keyboards-mcp midi-map's CC table — listed here per the published Sequential MIDI chart since the survey describes the device, not the MCP). All entries below are NRPN program-parameter numbers from Appendix C of the Operation Manual.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO Freq | continuous | 0–254 (NRPN) | rate (knob taper) | — | LFO rate (NRPN 88) |
| LFO Initial Amt | continuous | 0–255 (NRPN) | amount | — | LFO initial modulation depth applied without mod wheel (NRPN 89) |
| LFO Shape | discrete | 0–4 (NRPN) | enum | Triangle / Sawtooth / Reverse Sawtooth / Square / Random | LFO waveshape (NRPN 90). A sixth "noise" shape is reachable as Random + max-rate (out-of-common-core hidden mode). |
| LFO Sync | toggle | 0–1 (NRPN) | enum | Off / On | LFO sync to arpeggiator / sequencer / MIDI clock (NRPN 91) |
| LFO Freq 1 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to Osc 1 frequency (NRPN 93) |
| LFO Freq 2 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to Osc 2 frequency (NRPN 94) |
| LFO PW 1, 2 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to combined Osc 1+2 pulse width (NRPN 95) |
| LFO Amp Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to amplitude (tremolo) (NRPN 96) |
| LFO Low-pass Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to low-pass cutoff (NRPN 97) |
| LFO High-pass Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Route LFO to high-pass cutoff (NRPN 98) |

### Glide / unison / poly mod
Glide-on/off and Glide Mode are CC-addressable; Unison and Poly Mod are NRPN-only on the Prophet-6 (and are not in the keyboards-mcp midi-map). Listed for device completeness.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Glide On/Off | toggle | 0–1 (raw MIDI CC) | enum | Off / On | Glide (portamento) on/off (CC 65) |
| Glide Mode | discrete | 0–3 (raw MIDI CC) | enum | Fixed Rate / Fixed Rate A / Fixed Time / Fixed Time A | Portamento mode (CC 5). "A" variants apply only between legato-played notes. |
| Glide Rate | continuous | 0–127 (NRPN) | rate (knob taper) | — | Glide rate (NRPN 30) |
| Unison On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Stack all voices on a single note (NRPN 156). Out-of-common-core (voice-allocation mode). |
| Unison Mode | discrete | 0–6 (NRPN) | enum | (voice count: 1, 2, 3, 4, 5, 6, 12) | Number of voices stacked in unison (NRPN 157). 12 reflects poly-chained pair. Out-of-common-core. |
| Key Mode | discrete | 0–5 (NRPN) | enum | (low-note / high-note / last-note + retrigger variants) | Key-assign priority mode (NRPN 158). Only relevant in Unison. Out-of-common-core. |
| PolyMod Filter Env Amt | continuous | 0–254 (NRPN) | bipolar amount | — | Filter-envelope-as-mod-source amount into Poly Mod destination set (NRPN 143). Bipolar. Out-of-common-core. |
| PolyMod Osc 2 Amt | continuous | 0–254 (NRPN) | bipolar amount | — | Osc 2 (audio rate) as mod source, including FM at audio rate (NRPN 144). Bipolar. Out-of-common-core. |
| PolyMod Freq 1 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Poly Mod destination: Osc 1 frequency (NRPN 145). Out-of-common-core. |
| PolyMod Shape 1 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Poly Mod destination: Osc 1 waveshape (NRPN 146). Out-of-common-core. |
| PolyMod PW 1 Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Poly Mod destination: Osc 1 pulse width (NRPN 147). Out-of-common-core. |
| PolyMod Low-Pass Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Poly Mod destination: low-pass cutoff (NRPN 148). Out-of-common-core. |
| PolyMod High-Pass Dest On/Off | toggle | 0–1 (NRPN) | enum | Off / On | Poly Mod destination: high-pass cutoff (NRPN 149). Out-of-common-core. |

### Master
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| MIDI Volume | continuous | 0–127 (raw MIDI CC) | level | — | Master MIDI volume (CC 7) |
| Mod Wheel | continuous | 0–127 (raw MIDI CC) | level | — | Performance mod wheel; multiplies LFO Initial Amt routing (CC 1) |
| BPM | continuous | 0–127 (raw MIDI CC) | tempo (knob taper, underlying 30–250 BPM) | — | Tempo for arpeggiator, sequencer, FX sync, LFO sync (CC 3; NRPN 167 with native range 30–250 BPM) |
| Damper Pedal | toggle | 0–1 (raw MIDI CC) | enum | Off / On | Standard damper / sustain pedal (CC 64) |
| Distortion Amount | continuous | 0–127 (raw MIDI CC) | drive | — | Analog distortion drive (CC 9). Post-VCA — included only for CC completeness; the rest of the FX section (delay, reverb, chorus, phaser, etc.) is post-engine and out of scope for this survey. |
| Master Coarse Tune | discrete | 0–24 (NRPN) | semitones | — | Master coarse tune ±12 semitones (NRPN 1025) |
| Master Fine Tune | continuous | 0–100 (NRPN) | cents | — | Master fine tune (NRPN 1024) |
| Pbend Range | discrete | 0–24 (NRPN) | semitones | — | Pitch-bend range (NRPN 31) |

Arpeggiator parameters are CC-addressable but treated as a performance feature, not a tone-generation parameter — recorded for completeness: Arp On/Off (CC 58), Arp Mode (CC 59, enum: Up / Down / Up-Down / Random / Assign), Arp Range (CC 60, enum: 1 Oct / 2 Oct / 3 Oct), Arp Time Signature (CC 62, enum: Half / Qtr / 8th / 8th D / 8th S / 8th T / 16th / 16th S / 16th T / 32nd). Out-of-common-core (sequencer / arpeggiator domain).

## Modulation routing

The Prophet-6 has **one** LFO with a fixed destination set selected by per-destination on/off switches (multiple destinations may be enabled simultaneously — i.e. the LFO is broadcast to any subset of the destinations, not strictly one-of). Destination enum (verbatim from Appendix C / panel labels):

- **LFO destinations:** `Freq 1` (Osc 1 frequency), `Freq 2` (Osc 2 frequency), `PW 1+2` (combined pulse width), `Amp` (amplitude / tremolo), `LP Filter` (low-pass cutoff), `HP Filter` (high-pass cutoff). Hidden sixth shape `Noise` is reachable via `Random` + max rate.
- **LFO shapes:** `Triangle`, `Sawtooth`, `Reverse Sawtooth`, `Square`, `Random` (sample-and-hold). Hidden `Noise` mode (out-of-common-core).
- **LFO depth scaling:** `Initial Amount` is the always-on depth; `Mod Wheel` scales it additively at performance time.

Additional (out-of-common-core) modulation sources / paths:

- **Poly Mod sources:** `Filter Envelope`, `Oscillator 2` (audio-rate). **Poly Mod destinations:** `Freq 1`, `Shape 1`, `PW 1`, `LP Filter`, `HP Filter`. Bipolar amount per source. Out-of-common-core (cross-mod / per-voice mod-matrix).
- **Aftertouch fixed routings:** `Freq 1`, `Freq 2`, `LFO Amt`, `Amp`, `Filter`. Out-of-common-core (performance / per-voice pressure).
- **Velocity → filter envelope** via `Low-Pass Vel On/Off` and `High-Pass Vel On/Off` toggles.
- **Velocity → VCA** via `VCA Vel Amt` continuous.
- **Filter envelope → low-pass / high-pass** via signed `Low-Pass Env Amt` / `High-Pass Env Amt` (panel knob is bipolar; CC 64 = center).

Sources for routing: Operation Manual v2.1 pp. 33–37 (Low Frequency Oscillators, Poly Mod) and Appendix C NRPN tables.

## Quirks

- **Monotimbral.** All parameters are global (single program at a time, single MIDI channel for all voices). The keyboards-mcp midi-map header note confirms this (`mono-timbral analog synth — all parameters are global`).
- **CC values are raw 7-bit (0–127) with knob-taper non-linearity.** All continuous CCs in the keyboards-mcp midi-map use `encoding: RAW` with `min: 0, max: 127`. Acoustic-unit interpretation (Hz, ms, dB) is not exposed by the device — you only see knob position. NRPN values reach higher resolution (0–254 / 0–255 / 0–164) for some parameters but the CC pathway is quantized to 128 steps.
- **Discrete switches via CC use threshold semantics.** From the keyboards-mcp midi-map: toggles are `min: 0, max: 1` with explicit `labels: { 0: "Off", 1: "On" }`; multi-step discretes are `min: 0, max: N` with a label table enumerating each integer 0..N. The convention is "any non-zero value selects the next label up to max" — finer thresholds within 0–127 are a host-side concern and are not part of the Prophet-6 spec.
- **CC vs published chart agreement:** every CC number recorded in the keyboards-mcp midi-map matches Appendix C of the Operation Manual v2.1 (verified row-by-row: Osc CCs 67/69/70/71/75/76/77/78/79; sub 8; LP 102/103/104/105/47; HP 106/107/108/109/54; FilterEnv 50/51/52/53; VCA 40/41/43/44/45/46; Distortion 9; Glide 5/65; Damper 64; Volume 7; ModWheel 1; BPM 3; Arp 58/59/60/62). No discrepancies found.
- **Filter Env Amt is bipolar on the panel knob** (12 o'clock = 0, sweeps positive or negative). The single 7-bit CC value collapses both halves onto 0–127 with 64 = center. This is out-of-common-core if the canonical schema treats env amount as unipolar.
- **Vintage-trim parameter (`Slop`, NRPN 33).** Adds randomized per-voice oscillator detuning to emulate vintage oscillator drift. Out-of-common-core (vintage knob simulation).
- **Oscillator hard sync (`Osc 1 Sync`, NRPN 1).** Osc 1 is the slave, Osc 2 is the master (note: this is the inverse of the panel labelling convention on some other Sequential synths). Out-of-common-core.
- **Osc 2 dual-role.** `Osc 2 Low Freq` (NRPN 10) drops Osc 2 into LFO range; `Osc 2 Key On/Off` (NRPN 11) disables keyboard tracking. Combined with Poly Mod's Osc-2-as-source path, this gives the Prophet-6 a second free-running modulation oscillator (the "manual key mod / extra LFO" feature). Out-of-common-core.
- **Poly Mod (cross-mod / FM).** A dedicated per-voice mod section with two sources and a fixed destination set, using bipolar amounts. Out-of-common-core (cross-mod / dedicated mod-matrix slot).
- **Hidden LFO noise shape.** Selecting `Random` and turning the `LFO Freq` knob fully clockwise generates a noise modulation source that is not exposed as a sixth `LFO Shape` enum value (it is a side-effect of Random's quantization at maximum rate). Out-of-common-core.
- **LFO is exposed only via NRPN** — there is no LFO CC in the published chart or in the keyboards-mcp midi-map. Any host that needs to drive Prophet-6 LFO over MIDI must speak NRPN.
- **Effects out of scope.** Only `Distortion Amount` (CC 9) is exposed via CC; the rest of the FX engine (delay, reverb, chorus, phaser, BBD, spring) is addressed via NRPN (`FX 1 Select` NRPN 119, `FX 2 Select` NRPN 127, etc.) and is post-engine — out of scope for this subtractive-engine survey.
