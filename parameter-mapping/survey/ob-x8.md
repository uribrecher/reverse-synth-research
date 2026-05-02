# OB-X8 — Parameter Survey

**Manufacturer:** Oberheim
**Year:** 2022
**Engine type:** analog
**Polyphony:** 8
**Sources:**
- Oberheim, *OB-X8 User's Guide* v1.1, August 2022 (covers Chapter 2 panel controls, Chapter 3 Page 2 parameters, and the OB-X8 MIDI Implementation chart at the back of the manual): https://oberheim.com/wp-content/uploads/2022/08/OB-X8-Users-Guide-1.1.pdf
- Oberheim, *OB-X8 Manual* v2.2 (Sept 2023, current edition at the time of writing): https://oberheim.com/wp-content/uploads/2023/09/OB-X8-Manual-V2_2.pdf
- Oberheim, *OB-X8 OS 2.0 New Features Addendum* (July 2024): https://oberheim.com/wp-content/uploads/2024/07/OB-X8-OS2.0-Addendum.pdf
- midi.guide community CC database (cross-check, names verbatim from Oberheim's published MIDI implementation chart): https://midi.guide/d/oberheim/ob-x8/

No `keyboards-mcp` cross-check is available for this device — sourcing is web-only. CC and NRPN numbers below are taken row-by-row from the *Additional Continuous Controllers Transmitted/Received* table and the *Control NRPN Data* table at the back of the User's Guide v1.1; parameter names are reproduced verbatim from those tables (the chart uses ALL-CAPS naming, the prose chapters use Title Case — both are kept for traceability, since the MIDI chart table names are the canonical row identifiers).

## Signal path

Two analog VCOs (`OSC 1`, `OSC 2`) per voice, each generating sawtooth, pulse (variable width), and triangle waves — sawtooth and pulse are independently switchable, and with both `SAW` and `PULSE` LEDs unlit the oscillator generates a triangle. Pulse width is shared by a single panel `PULSE WIDTH` knob (with a per-oscillator override accessed by holding the `OSC 1`/`OSC 2` waveform switch while turning the knob, and per-oscillator levels available as Page 2 parameters). The oscillators feed a mixer that also receives a noise generator (`NOISE` switch in the Filter section) and offers per-source on/off + Page 2 level for `OSC 1`, `OSC 2`, and `NOISE`.

The mixer output passes into a multi-mode filter section. The OB-X8 ships with three legacy low-pass filter implementations selectable via the front-panel `TYPE` switch — `OB-X` (a discrete 2-pole, 12 dB/oct state-variable lowpass per the original OB-X), `OB-Xa/8 2-pole` (CEM 3320 in 2-pole mode), and `OB-Xa/8 4-pole` (CEM 3320 in 4-pole, 24 dB/oct lowpass) — plus three additional state-variable modes from the SEM (high-pass, band-pass, notch) selectable via Page 2 `Filter Type`. **The 2-pole/4-pole slope switching and the SEM HP/BP/Notch modes are out-of-common-core (multi-mode filter and slope-mode beyond a single LP).** Filter cutoff (`FILTER FREQUENCY`), resonance (`FILTER RESONANCE`), and a single bipolar `MODULATION` knob (which simultaneously sets both filter-envelope amount into cutoff and filter-envelope amount into Osc 2 frequency when `F-Env` is enabled in the Oscillators section) sit on the panel. `KBD` toggles 1V/octave keyboard tracking into cutoff; a Page 2 `Filter Keybd Track` parameter scales tracking from 0–127 (0–100%).

After the filter, audio passes into a VCA. Two dedicated 4-stage ADSR envelope generators (`FILTER ENVELOPE`, `VOLUME ENVELOPE`) drive filter cutoff and VCA respectively; both envelope contours can be reshaped via Page 2 `Envelope Type` to match either OB-X/OB-Xa (curved/exponential) or OB-8 (more linear) response.

A single front-panel `MODULATION` LFO with five waveshapes (`SINE`, `SQUARE`, `S/H`, `SAW UP` = Sine + Square pressed together, `SAW DOWN` = Square + S/H pressed together) feeds **two independent destination busses** — `DESTINATION 1` (`DEPTH 1` knob → `OSC 1`, `OSC 2`, `FILTER`) and `DESTINATION 2` (`DEPTH 2` knob → `PWM 1`, `PWM 2`, `VOLUME`). Each destination on either bus is an independent on/off, so the LFO is broadcast to any subset of the six destinations with a per-bus depth.

A second LFO and arpeggiator live in the Lever Box / `MOD BOX` section: the `MOD BOX LFO 2` is a vibrato-dedicated LFO with its own rate, shape (six waveforms — Triangle, Square, Saw Up, Sample and Hold, Saw Down, Noise — set via Page 2 `Vibrato LFO Wave`), depth (`DEPTH` knob), and a fixed destination set of `OSC 1` and/or `OSC 2` pitch (with an `OSC 2 ONLY` bend mode for the pitch lever). The `MOD BOX` also exposes pressure / aftertouch routing (`Filter Pressure`, `LFO Pressure`) and an arpeggiator. **The Mod Box LFO 2 is out-of-common-core (second LFO).**

X-Mod (`OSC 1 XMOD`) cross-modulates Osc 1's frequency with Osc 2's sawtooth, with an additional Page 2 `Osc 2 Tri. XMod` providing Osc 1 cross-mod from Osc 2's triangle wave (independent depth — `Page 2 OSC XMod Amt`). Hard sync is available via `OSC 1 SYNC` (Osc 1 is reset on each cycle of Osc 2 — note: front panel labels read `SYNC` under the OSC 1 column, and the User's Guide describes the result as "hard sync of Oscillator 2 by Oscillator 1" / "Oscillator 2 to restart its waveform on each cycle of Oscillator 1"; CC and NRPN names use `OSC 1 SYNC` / `OSC SYNC`). **X-Mod and Sync are out-of-common-core.**

The OB-X8 emulates three vintage modes via the Page 2 `Envelope Type` (`OB-X/Xa` vs `OB-8`) and Page 2 `LFO Type` (`OB-X/Xa` vs `OB-8`) parameters, plus the front-panel filter `TYPE` switch (which selects between OB-X, OB-Xa, and OB-Xa/8 4-pole filter implementations). **The vintage-mode emulation switches are out-of-common-core — captured under Quirks.**

The `VINTAGE` knob in the `CONTROL` section dials in randomized inter-voice variation across VCOs, filters, envelopes, LFOs, and amplifiers (mirroring the per-voice drift of the original Curtis-era hardware), and also scales Page 2 `Voice Detune Range`. **Vintage knob is out-of-common-core (vintage-knob simulation).** No on-board effects engine.

## Parameters

The OB-X8 publishes **two complementary control surfaces** for every program parameter:
- **CC range is 0–127 for almost every front-panel control** (per the *Additional Continuous Controllers Transmitted/Received* table in the User's Guide v1.1), with a small set of exceptions called out below where the published range is narrower (e.g. `MOD BOX LFO 2 BEND AMOUNT` 0–255 via 14-bit MSB+LSB, `OSC 1/2 FREQ` 0–63 raw on CC, `FILTER FREQUENCY` 0–175 because the underlying filter cutoff has more than 7 bits of program-parameter range).
- **NRPN ranges (from the *Control NRPN Data* table) reflect the underlying program-parameter resolution** — typically 0–63 for oscillator frequency in semitones, 0–127 for level/level-like parameters, 0–175 for the filter cutoff (extended sub-semitone resolution), 0–255 for the four "long" envelope time stages (Filter Env Attack / Decay / Release; Volume Env Attack / Decay / Release), and 0–1 for toggles. Where CC and NRPN ranges differ (e.g. `OSC 1 FREQ` is CC 0–63 but NRPN 0–63 too because the parameter is naturally semitone-stepped; `FILTER ENV ATTACK` is CC 0–255 reachable via Bank-Select-style 14-bit handling per the chart, where the chart lists the range as 0–255 directly), the survey records what each table publishes and notes the discrepancy in Quirks.

Per the User's Guide MIDI Implementation prose: **NRPNs are the preferred method of parameter transmission, since they cover the complete range of all parameters, while CCs are limited to a range of 128.** Both CC Send and NRPN Send paths are independently configurable in Globals (`MIDI Param Send`, `MIDI Param Receive`).

Toggles below use the convention `0–1 (CC/NRPN)` with `Off / On` enum labels. Discrete enums (e.g. `FILTER TYPE` 0–5, `LFO 1 SHAPE` 0–4) record the chart range and the panel/Page 2 enum label set. Parameter names below are the chart's verbatim row name; where the User's Guide text uses a slightly different casing or panel label, that is recorded in the Description column.

### Oscillator 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| OSC 1 FREQ | continuous | 0–63 (raw MIDI CC 15) | semitones (knob taper, 5-octave + minor third) | — | Oscillator 1 base frequency, semitone steps. NRPN 1, range 0–63. User's Guide: "5-octave+minor third range (0-+63 semitones)". |
| OSC 1 WAVEFORM | discrete | 0–3 (raw MIDI CC 19) | enum | (Triangle / Saw / Pulse / Saw+Pulse) | Combined waveform-switch state for Osc 1: triangle when both `SAW` and `PULSE` LEDs are off, saw with `SAW` on, pulse with `PULSE` on, saw+pulse with both on. NRPN 5, 0–3. |
| OSC 1 PW | continuous | 0–127 (raw MIDI CC 21) | pulse-width | — | Pulse width of Osc 1 pulse wave. The shared front-panel `PULSE WIDTH` knob also writes to this; per the User's Guide, holding the Osc 1 `PULSE` switch while turning `PULSE WIDTH` writes to this Osc-1-only parameter. NRPN 7, 0–127. |
| OSC 1 LEVEL | toggle | 0–1 (raw MIDI CC 25) | enum | Off / On | Osc 1 level switch into the filter mixer (front-panel `OSC 1` switch in the Filter section). On = full level (127); per-program continuous level is set by Page 2 `OSC 1 ON LEVEL`. NRPN 19, 0–1. |
| Page 2 OSC 1 ON LEVEL | continuous | 0–127 (raw MIDI CC 27) | level | — | Continuous Osc 1 level applied when the panel `OSC 1` switch is engaged. Default 127 (matches legacy OB-X full-on). NRPN 97, 0–127. |
| OSC 1 XMOD | toggle | 0–1 (raw MIDI CC 12) | enum | Off / On | X-Mod on/off — Osc 2 sawtooth cross-modulates Osc 1 frequency. **Out-of-common-core (cross-mod).** NRPN 9, 0–1. The CC table lists CC 12 twice (X-Mod and Filter Env Mod) — see Quirks. |
| OSC 1 FILTER ENV MOD | toggle | 0–1 (raw MIDI CC 12) | enum | Off / On | F-Env on/off — routes the filter envelope to Osc 2 frequency. (Front-panel `MOD` button cycles X-Mod off → X-Mod on → F-Env on → both → off; `OSC 1 FILTER ENV MOD` is the F-Env-on bit.) **Out-of-common-core (cross-mod / per-voice envelope-as-pitch-modulator).** NRPN 12, 0–1. |
| Page 2 OSC XMod AMT | continuous | 0–127 (raw MIDI CC 13) | amount | — | Independent depth for `Osc 2 Tri. XMod` — Osc 2 triangle into Osc 1 frequency, additive to the panel-switched X-Mod. **Out-of-common-core.** NRPN 11, 0–127. |

### Oscillator 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| OSC 2 FREQ | continuous | 0–63 (raw MIDI CC 16) | semitones (knob taper) | — | Oscillator 2 base frequency, semitone steps over the same 5-octave+minor third range as Osc 1. NRPN 2, 0–63. |
| OSC 2 DETUNE | continuous | 0–63 (raw MIDI CC 17) | cents (±49.2 cents centered) | — | Osc 2 detune relative to Osc 1, ±49.2 cents (half the Master Tune range), with finer resolution near the knob center. NRPN 3, 0–63. |
| OSC 2 WAVEFORM | discrete | 0–3 (raw MIDI CC 20) | enum | (Triangle / Saw / Pulse / Saw+Pulse) | Combined waveform-switch state for Osc 2 (same encoding as Osc 1). NRPN 6, 0–3. |
| OSC 2 PW | continuous | 0–127 (raw MIDI CC 22) | pulse-width | — | Pulse width of Osc 2 pulse wave; shares the panel `PULSE WIDTH` knob with Osc 1 (independent override via held Osc 2 `PULSE` switch + knob, or Page 2 `Osc 2 Pulse Width`). NRPN 8, 0–127. |
| OSC 2 LEVEL | toggle | 0–1 (raw MIDI CC 26) | enum | Off / On | Osc 2 level switch into the filter mixer. NRPN 20, 0–1. |
| Page 2 OSC 2 ON LEVEL | continuous | 0–127 (raw MIDI CC 28) | level | — | Continuous Osc 2 level when the panel `OSC 2` switch is engaged (default 127; setting 49 emulates the legacy OB-X "half" position). NRPN 98, 0–127. |
| OSC 1 SYNC | toggle | 0–1 (raw MIDI CC 24) | enum | Off / On | Hard sync (Osc 1's waveform restarts on each cycle of Osc 2 per the User's Guide; the chart row is named `OSC 1 SYNC` and NRPN row is named `OSC SYNC`). **Out-of-common-core (oscillator sync).** NRPN 13, 0–1. |
| OSC SQUARE MODE | toggle | 0–1 (raw MIDI CC 23) | enum | OB-X/Xa level / OB-8 level | Selects whether the oscillators' pulse/square wave level matches OB-X/Xa (louder) or OB-8 (softer) — see User's Guide p. 22. **Out-of-common-core (vintage-mode emulation).** NRPN 10, 0–1. |

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| NOISE LEVEL | toggle | 0–1 (raw MIDI CC 29) | enum | Off / On | Noise switch into the filter mixer. On = full level (127); Page 2 `NOISE ON LEVEL` provides a programmable level. NRPN 21, 0–1. |
| Page 2 NOISE ON LEVEL | continuous | 0–127 (raw MIDI CC 30) | level | — | Continuous noise level applied when the panel `NOISE` switch is engaged. (Setting 49 emulates the legacy OB-X "half" Noise switch.) NRPN 99, 0–127. |

### Filter
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| FILTER FREQUENCY | continuous | 0–175 (raw MIDI CC 33) | cutoff (knob taper) | — | Filter cutoff frequency. The chart specifies a 0–175 range on this CC because the underlying program parameter has more than 7 bits of resolution; the published range is `00-175` per the *Additional Continuous Controllers* table. NRPN 22, 0–175. |
| BRIGHTNESS (FILTER CUTOFF - RELATIVE) | continuous | 0–175 (raw MIDI CC 74) | added cutoff offset | — | Standard MIDI Brightness controller — added to filter cutoff frequency (per *Received Controller Messages* table: "Brightness: Added to filter cutoff frequency"). |
| FILTER RESONANCE | continuous | 0–127 (raw MIDI CC 34) | resonance | — | Filter resonance. Per User's Guide p. 24 the OB-X8 filters do **not** self-oscillate (unlike the Prophet-5/6). NRPN 23, 0–127. |
| FILTER MODULATION | continuous | 0–127 (raw MIDI CC 31) | amount | — | Single bipolar `MODULATION` knob in the Filter section: amount of filter envelope into filter cutoff, **and** simultaneously amount of filter envelope into Osc 2 frequency when `F-Env` is enabled in the Oscillators section. (User's Guide p. 24: "The Modulation knob controls the amount of positive modulation that the Filter Envelope will have on the filter frequency. This same control also determines the amount of positive filter envelope modulation on Oscillator 2 if the F-Env function is on.") NRPN 59, 0–127. |
| FILTER TYPE | discrete | 0–5 (raw MIDI CC 35) | enum | OB-X/SEM 2-Pole LP / SEM 2-Pole HP / SEM 2-Pole BP / SEM 2-Pole Notch / OB-Xa/8 2-Pole LP / OB-Xa/8 4-Pole LP | Filter mode selector. Front-panel `TYPE` switch toggles OB-X 2-pole LP (yellow LED), OB-Xa/8 2-pole LP (red LED), OB-Xa/8 4-pole LP (both LEDs). The SEM HP/BP/Notch modes are reachable only via Page 2 `Filter Type`. **Out-of-common-core (multi-mode filter; LP slope switch). The schema later collapses to LP common-core.** NRPN 24, 0–5. |
| FILTER KEYBOARD | toggle | 0–1 (raw MIDI CC 36) | enum | Off / On | Front-panel `KBD` switch — enables 1V/oct keyboard tracking into filter cutoff. NRPN 25, 0–1. |
| Page 2 FILTER KBD ON LEVEL | continuous | 0–127 (raw MIDI CC 37) | tracking | — | Programmable keyboard-track amount applied when `KBD` is on (0–100% / 0–1V/oct). The original OB synths only offered on/off; Page 2 enables continuous values. NRPN 123, 0–127. |
| FILTER PRESSURE | toggle | 0–1 (raw MIDI CC 44) | enum | Off / On | Aftertouch → filter cutoff routing (`AFTERTOUCH > FILT` panel labelling under the `TOUCH` switch in the Control section). **Out-of-common-core (per-voice pressure routing).** NRPN 70, 0–1. |
| FILTER ENV VELOCITY ON/OFF | toggle | 0–1 (raw MIDI CC 43) | enum | Off / On | Velocity → filter envelope routing (`VELO` switch, Filt setting). NRPN 68, 0–1. |

### Filter envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| FILTER ENV ATTACK | continuous | 0–255 (raw MIDI CC 39) | time (knob taper) | — | Filter envelope attack time. NRPN 60, 0–255. (The CC range matches NRPN range — Oberheim's chart publishes 0–255 for the long envelope-time parameters; values are transmitted via the standard 14-bit Data Entry MSB/LSB pairing where CC 39 carries the parameter selector and the value is reconstructed. Per the chart row.) |
| FILTER ENV DECAY | continuous | 0–255 (raw MIDI CC 40) | time (knob taper) | — | Filter envelope decay time. NRPN 62, 0–255. |
| FILTER ENV SUSTAIN | continuous | 0–127 (raw MIDI CC 41) | level | — | Filter envelope sustain level. NRPN 64, 0–127. |
| FILTER ENV RELEASE | continuous | 0–255 (raw MIDI CC 42) | time (knob taper) | — | Filter envelope release time. NRPN 66, 0–255. |

### Amp envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VOLUME ENV ATTACK | continuous | 0–255 (raw MIDI CC 45) | time (knob taper) | — | Volume envelope attack time. NRPN 61, 0–255. |
| VOLUME ENV DECAY | continuous | 0–255 (raw MIDI CC 46) | time (knob taper) | — | Volume envelope decay time. NRPN 63, 0–255. |
| VOLUME ENV SUSTAIN | continuous | 0–127 (raw MIDI CC 47) | level | — | Volume envelope sustain level. NRPN 65, 0–127. |
| VOLUME ENV RELEASE | continuous | 0–255 (raw MIDI CC 48) | time (knob taper) | — | Volume envelope release time. NRPN 67, 0–255. |
| VOLUME ENV VELOCITY ON/OFF | toggle | 0–1 (raw MIDI CC 49) | enum | Off / On | Velocity → volume envelope routing (`VELO` switch, Vol setting). NRPN 69, 0–1. |
| ENV TYPE | toggle | 0–1 (raw MIDI CC 50) | enum | OB-X/Xa contour / OB-8 contour | Selects envelope contour shape for both Filter and Volume envelopes simultaneously: OB-X/OB-Xa = curved (more exponential attack), OB-8 = more linear. (User's Guide Chapter 3 §13: "This parameter allows the volume and filter envelope models to be set to either OB-X/Xa or OB-8 contours.") **Out-of-common-core (vintage-mode emulation — envelope curve switch).** NRPN 96, 0–2 in the NRPN table (the CC range is 0–1 per the *Additional Continuous Controllers* row; the NRPN row publishes 0–2 — see Quirks). |

### LFO 1 (front-panel Modulation)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 1 RATE | continuous | 0–127 (raw MIDI CC 51) | rate (knob taper, 0.0667 Hz – 50 Hz per User's Guide) | — | Front-panel LFO `RATE`. NRPN 29, 0–127. |
| LFO 1 SHAPE | discrete | 0–4 (raw MIDI CC 52) | enum | Sine / Square / Sample-and-Hold / Saw Up / Saw Down | LFO waveform. (User's Guide: Sine, Square, S/H, Saw Up reached via Sine+Square pressed together, Saw Down reached via Square+S/H pressed together.) NRPN 30, 0–4. |
| LFO 1 DEPTH 1 | continuous | 0–127 (raw MIDI CC 53) | amount | — | `DEPTH 1` knob — overall depth applied to Destination 1 bus (Osc 1, Osc 2, Filter destinations). NRPN 31, 0–127. |
| LFO 1 DEPTH 2 | continuous | 0–127 (raw MIDI CC 54) | amount | — | `DEPTH 2` knob — overall depth applied to Destination 2 bus (PWM 1, PWM 2, Volume destinations). NRPN 32, 0–127. |
| LFO 1 DEST 1 OSC 1 | toggle | 0–1 (raw MIDI CC 55) | enum | Off / On | Destination 1 bus → Osc 1 frequency. NRPN 33, 0–1. |
| LFO 1 DEST 1 OSC 2 | toggle | 0–1 (raw MIDI CC 56) | enum | Off / On | Destination 1 bus → Osc 2 frequency. NRPN 34, 0–1. |
| LFO 1 DEST 1 FILTER | toggle | 0–1 (raw MIDI CC 57) | enum | Off / On | Destination 1 bus → filter cutoff. NRPN 35, 0–1. |
| LFO 1 DEST 2 PWM 1 | toggle | 0–1 (raw MIDI CC 58) | enum | Off / On | Destination 2 bus → Osc 1 pulse width. NRPN 36, 0–1. |
| LFO 1 DEST 2 PWM 2 | toggle | 0–1 (raw MIDI CC 59) | enum | Off / On | Destination 2 bus → Osc 2 pulse width. NRPN 37, 0–1. |
| LFO 1 DEST 2 VOLUME | toggle | 0–1 (raw MIDI CC 60) | enum | Off / On | Destination 2 bus → final VCA (amplitude modulation / tremolo). NRPN 38, 0–1. |
| LFO PRESSURE | toggle | 0–1 (raw MIDI CC 61) | enum | Off / On | Aftertouch → LFO depth (combined LFO/Mod-Box pressure routing per User's Guide; the chart names it `LFO PRESSURE` while the NRPN row names it `PRESSURE > LFO`). **Out-of-common-core.** NRPN 71, 0–1. |
| Page 2 LFO TYPE | toggle | 0–1 (raw MIDI CC 62) | enum | OB-X/Xa LFO / OB-8 LFO | Selects whether the front-panel LFO models OB-X/Xa (analog sine, negative-going square, VCF-mod path inverted relative to other paths) or OB-8 (digitally-generated triangle in place of sine, positive-going square, VCF-mod path not inverted). **Out-of-common-core (vintage-mode emulation).** NRPN 27, 0–1. |
| Page 2 LFO SAMPLE/HOLD IN | toggle | 0–1 (raw MIDI CC 63) | enum | Noise / Vibrato LFO | Source for the Sample-and-Hold LFO mode — chooses noise (default) or the Mod-Box Vibrato LFO waveform as the S/H input. NRPN 28, 0–1. |

Page 2 LFO advanced parameters (out-of-common-core — capture for completeness; all are CC-addressable per the *Additional Continuous Controllers* table):

- `Page 2 LFO 1 TRIGGER` (CC 102, NRPN 39, 0–1) — keyboard-trig the LFO on first note (mono-trigger) instead of free-run.
- `Page 2 LFO TRIG PHASE` (CC 103, NRPN 40, 0–127) — phase of the LFO waveform at trigger time (0 = waveform start, 127 = 50% of cycle).
- `Page 2 LFO 1 KBD TRACK` (CC 104, NRPN 41, 0–1) — 1/4 keyboard tracking on LFO speed (LFO speed doubles every 4 octaves).
- `Page 2 LFO 1 MOD 1 RAMP DELAY` (CC 105, NRPN 42, 0–127) — Mod 1 ramp delay (0–4.5 s linear) before depth-1 fades in.
- `Page 2 LFO 1 MOD 1 RAMP UP` (CC 106, NRPN 43, 0–127) — Mod 1 attack ramp time (0–4.5 s exponential) for depth-1 fade-in.
- `Page 2 LFO 1 RAMP INVERT` (CC 107, NRPN 44, 0–1) — invert Mod 1 envelope (fade-out instead of fade-in).
- `Page 2 LFO 1 MOD 1 QUANTIZE` (CC 108, NRPN 45, 0–1) — quantize Mod 1 path to semitones.
- `Page 2 LFO 1 OSC 1 MOD 1 INVERT` (CC 109, NRPN 46, 0–1) — invert LFO phase to Osc 1 freq vs Osc 2 freq (Osc 1 down while Osc 2 up).
- `Page 2 LFO 1 MOD 2 RAMP DELAY` (CC 110, NRPN 47, 0–127) — Mod 2 ramp delay.
- `Page 2 LFO 1 MOD 2 RAMP UP` (CC 111, NRPN 48, 0–127) — Mod 2 attack ramp time.
- `Page 2 LFO 1 MOD 2 RAMP INVERT` (CC 112, NRPN 49, 0–1) — invert Mod 2 envelope.
- `Page 2 LFO 1 MOD 2 QUANTIZE` (CC 113, NRPN 50, 0–1) — quantize Mod 2 path to semitones.
- `Page 2 LFO 1 OSC 1 MOD 2 INVERT` (CC 114, NRPN 51, 0–1) — invert LFO phase to PWM 1 vs PWM 2.
- `Page 2 MOD 2 RAMP TO LFO SPEED` (CC 115, NRPN 52, 0–1) — Mod 2 envelope modulates LFO rate (depth set by `LFO 1 DEPTH 2`).
- `Page 2 LFO SHIFT VOICES 5-8` (CC 116, NRPN 53, 0–2) — phase shift of voices 5–8 LFO relative to voices 1–4 (None / 90° / 180°). Voices 1–4 and 5–8 use separate LFOs in OB-8 emulation mode — see Quirks.

### LFO 2 / Mod Box (vibrato LFO + arp + bend)
**The entire Mod Box LFO 2 section is out-of-common-core** (second LFO source / per-voice pressure / arp).

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| MOD BOX LFO 2 RATE | continuous | 0–127 (raw MIDI CC 75) | rate | — | Vibrato LFO rate. NRPN 54, 0–127. |
| MOD BOX LFO 2 SHAPE | discrete | 0–5 (raw MIDI CC 76) | enum | Triangle / Square / Saw Up / Sample-and-Hold / Saw Down / Noise | Vibrato LFO waveform (per Page 2 `Vibrato LFO Wave`). NRPN 55, 0–5. |
| MOD BOX LFO 2 DEST OSC 1 | toggle | 0–1 (raw MIDI CC 77) | enum | Off / On | Vibrato LFO → Osc 1 pitch. NRPN 56, 0–1. |
| MOD BOX LFO 2 DEST OSC 2 | toggle | 0–1 (raw MIDI CC 78) | enum | Off / On | Vibrato LFO → Osc 2 pitch. NRPN 57, 0–1. |
| MOD BOX LFO 2 DEPTH | continuous | 0–127 (raw MIDI CC 79) | amount | — | Vibrato LFO initial depth (independent of Mod-lever depth which is summed on top). NRPN 58, 0–127. |
| MOD BOX LFO 2 BEND OSC 2 ONLY | toggle | 0–1 (raw MIDI CC 80) | enum | Off / On | Pitch lever bends Osc 2 only (guitar-bend effect). NRPN 92, 0–1. |
| MOD BOX MODE | toggle | 0–1 (raw MIDI CC 81) | enum | Mod / Arp | Lever-Box mode switch — selects whether the bottom-row buttons control modulation routing or arpeggiator. NRPN 77, 0–1. |
| MOD BOX LFO 2 LOWER | toggle | 0–1 (raw MIDI CC 82) | enum | Off / On | Apply Mod-Box LFO to Lower preset in Split/Double mode. NRPN 78, 0–1. |
| MOD BOX LFO 2 UPPER | toggle | 0–1 (raw MIDI CC 83) | enum | Off / On | Apply Mod-Box LFO to Upper preset in Split/Double mode. NRPN 79, 0–1. |
| MOD BOX LFO 2 BEND AMOUNT | continuous | 0–255 (raw MIDI CC 1, 14-bit via CC 1 MSB / CC 33 LSB convention) | semitones (knob taper, ¼ semitone – 12 semitones) | — | Pitch-lever bend amount. NRPN 72, 0–12 (per the *Control NRPN Data* table — the NRPN row publishes the semitone range directly). |

Arpeggiator parameters (`MOD BOX ARP ON` CC 84 / NRPN 80, `MOD BOX ARP SPEED` CC 85 / NRPN 81 (NRPN range 0–27), `MOD BOX ARP HOLD KBD` CC 86 / NRPN 82 (range 0–2), `MOD BOX ARP DOWN/UP` CC 87 / NRPN 83 (range 0–3), `MOD BOX ARP LOWER` CC 88 / NRPN 84, `MOD BOX ARP UPPER` CC 89 / NRPN 85, `ARP TRANSPOSE` CC 90 / NRPN 86 (range 0–5)) are CC- and NRPN-addressable but treated as a performance feature, not a tone-generation parameter. Recorded for completeness — out-of-common-core (sequencer / arpeggiator domain).

### Voice (unison, glide, vintage)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VINTAGE | continuous | 0–127 (raw MIDI CC 3) | amount (panel labelled "stable → vintage") | — | **Vintage knob.** Dials in randomized inter-voice variation across VCOs, filters, envelopes, LFOs, amplifiers — emulates the calibration drift of OB-X / OB-Xa / OB-8 era hardware. Also scales `Voice Detune Range` Page 2 parameter. **Out-of-common-core (vintage-knob simulation).** NRPN 26, 0–127. |
| Page 2 VOICE DETUNE RANGE | continuous | 0–127 (raw MIDI CC 18) | cents | — | Per-voice detune amount (scales with Vintage). **Out-of-common-core.** NRPN 4, 0–127. |
| PORTAMENTO RATE | continuous | 0–127 (raw MIDI CC 5) | rate (knob taper) | — | Portamento / glide rate. NRPN 14, 0–127. |
| Page 2 PORTAMENTO MODE | discrete | 0–66 (raw MIDI CC 91) | enum | Normal/Linear / Exponential / Equal Time / Bend (-12.0 .. +19.5 semitones) | Portamento mode. The Bend mode is special — the high range encodes a bend interval rather than a glide curve. NRPN 15, 0–66. |
| Page 2 PORTAMENTO MATCH | toggle | 0–1 (raw MIDI CC 92) | enum | Off / On | Match all voices' portamento times (off = each voice glides at slightly different rate, emulating OB-X analog drift). NRPN 16, 0–1. |
| Page 2 PORTAMENTO QUANTIZE | toggle | 0–1 (raw MIDI CC 93) | enum | Off / On | Quantize all portamento modes to semitones. NRPN 17, 0–1. |
| Page 2 PORTAMENTO LEGATO | toggle | 0–1 (raw MIDI CC 94) | enum | Off / On | Glide only when playing legato (Unison-mode only per User's Guide). NRPN 18, 0–1. |
| UNISON | toggle | 0–1 (raw MIDI CC 70) | enum | Off / On | Unison mode (mono with up to 8 voices stacked). **Out-of-common-core (voice-allocation mode).** NRPN 74, 0–1. |
| UNISON VOICE COUNT | discrete | 0–7 (raw MIDI CC 71) | voice count | (1 / 2 / 3 / 4 / 5 / 6 / 7 / 8) | Number of voices stacked in unison. **Out-of-common-core.** NRPN 75, 0–7. |
| UNISON KEY MODE | discrete | 0–5 (raw MIDI CC 72) | enum | (lowest / highest / last + retrigger variants) | Key-priority mode for Unison. **Out-of-common-core.** NRPN 76, 0–5. |

### Master / Global
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| MASTER VOLUME | continuous | 0–127 (raw MIDI CC 7) | level | — | Master MIDI volume. The front-panel `MASTER VOL` knob is a non-programmable physical pot per User's Guide. |
| PROGRAM VOLUME | continuous | 0–127 (raw MIDI CC 8) | level | — | Per-program saved volume (different from the Master Vol knob). NRPN 73, 0–127. |
| TRANSPOSE UP/DOWN | discrete | 0–2 (raw MIDI CC 9) | enum | Down / Center / Up | Transpose switch in the Lever Box (–1 oct / 0 / +1 oct). NRPN 122, 0–2. |
| Mod Wheel (CC 1) | continuous | 0–127 (raw MIDI CC 1) | level | — | Performance Mod Wheel — directly assignable per *Received Controller Messages*. (The `MOD BOX LFO 2 BEND AMOUNT` row above also occupies CC 1 in the *Additional Continuous Controllers* table because the OB-X8 uses CC 1 for the mod-lever's directly-assignable controller pathway; see Quirks.) |
| EXPRESSION PEDAL | continuous | 0–127 (raw MIDI CC 11) | level | — | Expression-pedal input — directly assignable. |
| FOOT CONTROLLER | continuous | 0–127 (raw MIDI CC 4) | level | — | Foot Controller — directly assignable per *Received Controller Messages*. |
| Damper pedal | toggle | 0–127, threshold (raw MIDI CC 64) | enum | Off (<64) / On (≥64) | Standard sustain pedal — "Holds envelopes in Sustain if 0100 0000 or higher" per the chart. Behavior is shaped by the Global `Sustain Pedal Mode` (Normal/Release vs Note On). |
| SUSTAIN | toggle | 0–1 (raw MIDI CC 64) | enum | Off / On | Programmable sustain-pedal-on bit recorded as a row in the *Additional Continuous Controllers* table (NRPN N/A — sustain is performance-only). |
| Master Tune | discrete | -12..+12 (NRPN, global) | semitones | — | Global Master Tune (±12 semitones, 0 = A-440). Global parameter (not in the Control NRPN Data table — set via Globals menu §1). |
| MIDI Channel | discrete | 0–16 (NRPN, global) | channel | All / 1..16 | Global MIDI Channel (Globals §3). |
| Pitch Wheel Range | continuous | 0–127 (raw MIDI CC 1 LSB) | semitones (¼ semitone – 12 semitones) | — | See `MOD BOX LFO 2 BEND AMOUNT` row above — the OB-X8 sets pitch-bend range via the Mod-Box `AMOUNT` switch + a key from C0 (¼ semitone) to C1 (12 semitones), exposed on the chart as `MOD BOX LFO 2 BEND AMOUNT`. |
| ALT TUNINGS | discrete | 0–63 (NRPN, global) | enum | Equal Temperament + 63 alt scales | Alternative tuning selection (Globals §14). NRPN scope only. |
| Aftertouch Curve | discrete | 0–7 (NRPN, global) | curve | Curve 1..8 | Aftertouch response curve (Globals §6). |
| Velocity Curve | discrete | 0–6 (NRPN, global) | curve | Curve 1..7 | Velocity response curve (Globals §7). |

Globals exposed only via the Globals menu (Local Control, Pot Edit Mode, Page 2 Edit Mode, Stereo/Mono Out, Pitch Lever Direction, MIDI Param Send/Receive, MIDI Sysex Cable, etc.) are recorded for completeness in the User's Guide §1–35; **out-of-common-core (system / configuration domain).**

## Modulation routing

The OB-X8 has **two LFOs** (front-panel `MODULATION` LFO 1 with two destination busses, and Mod-Box `LFO 2` Vibrato LFO with a fixed two-destination set), plus an aftertouch path with two switchable destinations (Filter, LFO), a velocity path with two switchable destinations (Filter Env, Volume Env), and a pair of cross-modulation paths (X-Mod, F-Env into Osc 2). The two-LFO topology is itself out-of-common-core.

- **LFO 1 destinations:** `OSC 1` (Osc 1 frequency), `OSC 2` (Osc 2 frequency), `FILTER` (filter cutoff) — these three on the `DESTINATION 1` bus, scaled by `DEPTH 1`. `PWM 1` (Osc 1 pulse width), `PWM 2` (Osc 2 pulse width), `VOLUME` (final VCA / amplitude modulation) — these three on the `DESTINATION 2` bus, scaled by `DEPTH 2`. Each destination is an independent on/off so the LFO is broadcast to any subset.
- **LFO 1 shapes:** `Sine`, `Square`, `Sample-and-Hold`, `Saw Up` (Sine + Square pressed together), `Saw Down` (Square + S/H pressed together). Five total. The Sine shape is internally a Triangle in OB-8 emulation mode (User's Guide p. 32: "the Sine wave in OB-8 mode is actually a Triangle wave").
- **LFO 2 (Mod Box / Vibrato) destinations:** `OSC 1` pitch and/or `OSC 2` pitch (independent on/off). Out-of-common-core (second LFO).
- **LFO 2 shapes:** `Triangle`, `Square`, `Saw Up`, `Sample-and-Hold`, `Saw Down`, `Noise`. Six total. Out-of-common-core.
- **LFO depth scaling:** `DEPTH 1`, `DEPTH 2` are the always-on per-bus depths for LFO 1; `MOD BOX LFO 2 DEPTH` is the always-on depth for LFO 2 with the Mod Lever scaling on top.
- **Aftertouch fixed routings (TOUCH switch in Control section):** `Filter` (channel-pressure → filter cutoff via `FILTER PRESSURE`) and/or `LFO` (channel-pressure → LFO depth via `LFO PRESSURE`). Both can be enabled simultaneously. **Out-of-common-core (per-voice pressure routing).**
- **Velocity → Filter envelope** via `FILTER ENV VELOCITY ON/OFF`.
- **Velocity → Volume envelope** via `VOLUME ENV VELOCITY ON/OFF`.

Out-of-common-core (cross-modulation):
- **X-Mod (`OSC 1 XMOD`, `Page 2 OSC XMod AMT`).** Two cross-mod paths into Osc 1 frequency: panel-switched X-Mod from Osc 2 sawtooth (depth not directly programmable on the legacy panel — depth comes from per-voice variation per User's Guide), and Page-2-controllable continuous-amount X-Mod from Osc 2 triangle. Both can be on simultaneously.
- **F-Env into Osc 2 frequency (`OSC 1 FILTER ENV MOD`).** Routes the filter envelope to Osc 2 pitch (the "pitch blip" hard-sync-lead trick — User's Guide Chapter 4, Synth Brass example). Depth is shared with the filter's `MODULATION` knob.
- **Osc Sync (`OSC 1 SYNC`).** Hard-sync between the two oscillators. Out-of-common-core.

Sources for routing: User's Guide v1.1 Chapter 2 (Oscillators, Filter, Envelopes, Modulation LFO, Master, Control, Lever Box), Chapter 3 (Page 2 Parameters), and the *Additional Continuous Controllers Transmitted/Received* + *Control NRPN Data* tables in the OB-X8 MIDI Implementation appendix.

## Quirks

- **Three vintage emulation modes (OB-X / OB-Xa / OB-8) are spread across multiple parameters — there is no single "vintage mode" enum.** The emulation is reconstructed per program from the combination of (a) `FILTER TYPE` (chart range 0–5: OB-X 2-pole SVF, OB-Xa/8 2-pole CEM, OB-Xa/8 4-pole CEM, plus three SEM-only HP/BP/Notch modes), (b) `Page 2 ENV TYPE` (OB-X/Xa curved vs OB-8 linear envelope contour for both Filter and Volume envelopes), (c) `Page 2 LFO TYPE` (OB-X/Xa analog-style sine + negative-going square + inverted VCF-mod path, vs OB-8 digital-style triangle + positive-going square + non-inverted VCF-mod path), and (d) `OSC SQUARE MODE` (OB-X/Xa louder square level vs OB-8 softer square level). A patch is "OB-X-style" when `FILTER TYPE = OB-X 2-Pole LP` + `Page 2 ENV TYPE = OB-X/Xa` + `Page 2 LFO TYPE = OB-X/Xa` + `OSC SQUARE MODE = OB-X/Xa`; "OB-Xa-style" when `FILTER TYPE = OB-Xa/8 2-Pole LP` + same envelope/LFO/square levels; "OB-8-style" when `FILTER TYPE = OB-Xa/8 4-Pole LP` + `Page 2 ENV TYPE = OB-8` + `Page 2 LFO TYPE = OB-8` + `OSC SQUARE MODE = OB-8`. **Out-of-common-core: the schema's filter/envelope/LFO common-core does not capture vintage-mode-specific shape differences.** Hosts targeting OB-X8 should treat these four parameters as a coupled "vintage-mode" preset.
- **Multi-mode filter with single Cutoff CC and a separate `FILTER TYPE` enum.** Per the task brief: there is one `FILTER FREQUENCY` CC (33) and one `FILTER RESONANCE` CC (34) regardless of which of the six modes is active; the mode is selected by `FILTER TYPE` CC 35 (range 0–5). The schema later collapses the LP modes (OB-X 2-pole LP, OB-Xa/8 2-pole LP, OB-Xa/8 4-pole LP) to a common-core LP filter; the SEM HP/BP/Notch modes are out-of-common-core. **The 2-pole vs 4-pole slope switching is itself out-of-common-core (slope-mode selector).**
- **Filters do not self-oscillate.** Per User's Guide p. 24: "none of the OB-X8's filter types can self-oscillate to generate a pitch." This contrasts with the Prophet-5 / Prophet-6 surveys.
- **Two LFOs.** The front-panel `MODULATION` LFO 1 plus the Mod-Box `LFO 2` Vibrato LFO. Out-of-common-core (the canonical schema's common-core has a single LFO).
- **LFO 1 has TWO destination busses with independent depth** (`DEPTH 1` → Osc 1, Osc 2, Filter; `DEPTH 2` → PWM 1, PWM 2, Volume). Out-of-common-core (multi-bus / multi-depth LFO).
- **Voices 1–4 and 5–8 use separate LFO oscillators in OB-8 emulation mode** (User's Guide §17: "to stay authentic to the OB-8 design, a separate LFO is generated for voices 1 through 4, and for voices 5 through 8, which can result in some pseudo-random (but authentic) results when modulating the LFO speed or retriggering waveforms from the keyboard"). The phase shift between the two halves is set by `Page 2 LFO Shift Voices 5-8` (None / 90° / 180°). Out-of-common-core.
- **CC range is 0–127 for almost every panel control**, with these documented exceptions from the *Additional Continuous Controllers Transmitted/Received* table:
  - `MOD BOX LFO 2 BEND AMOUNT` (CC 1) is published as 0–255 (14-bit pitch-bend amount, transmitted via MSB+LSB).
  - `OSC 1 FREQ` and `OSC 2 FREQ` (CC 15, 16) are 0–63 because the natural parameter is semitone-stepped over 64 semitones.
  - `OSC 2 DETUNE` (CC 17) is 0–63.
  - `OSC 1 WAVEFORM` and `OSC 2 WAVEFORM` (CC 19, 20) are 0–3 (4-state encoding: triangle, saw, pulse, saw+pulse).
  - `FILTER FREQUENCY` (CC 33) is 0–175 because the underlying program-parameter has more than 7 bits of resolution.
  - `FILTER TYPE` (CC 35) is 0–5 (six modes).
  - `FILTER ENV ATTACK / DECAY / RELEASE` and `VOLUME ENV ATTACK / DECAY / RELEASE` (CC 39, 40, 42, 45, 46, 48) are published as 0–255 — the chart row range. (NRPNs for the same parameters are also 0–255 per the *Control NRPN Data* table.)
  - `LFO 1 SHAPE` (CC 52) is 0–4 (five shapes); `MOD BOX LFO 2 SHAPE` (CC 76) is 0–5 (six shapes).
  - `BRIGHTNESS` (CC 74) is 0–175.
  - `UNISON VOICE COUNT` (CC 71) is 0–7; `UNISON KEY MODE` (CC 72) is 0–5.
  - `Page 2 PORTAMENTO MODE` (CC 91) is 0–66 (Normal + Exponential + Equal Time + Bend with –12.0 to +19.5 semitone range in 0.5-semitone steps).
- **NRPN is the preferred transmission method.** Quoting the chart: "NRPNs are the preferred method of parameter transmission, since they cover the complete range of all parameters, while CCs are limited to a range of 128." Hosts must speak NRPN to reach the long envelope-time ranges (0–255), the filter-cutoff range (0–175), and any program-parameter where 7-bit CC truncates the resolution.
- **CC 12 is reused for two parameters** — `OSC 1 XMOD` and `OSC 1 FILTER ENV MOD` both occupy CC 12 in the *Additional Continuous Controllers* table (this is reproduced from the published chart and is not a survey transcription error). The two parameters are addressable independently via NRPN (NRPN 9 = X-Mod; NRPN 12 = Filter Env Mod). The shared CC reflects that the front-panel `MOD` button cycles X-Mod/F-Env/both with a single increment-style controller. **Recorded but not normalized in this survey.**
- **The single bipolar `MODULATION` knob in the Filter section drives two destinations** — filter envelope amount into filter cutoff, **and** filter envelope amount into Osc 2 frequency when `OSC 1 FILTER ENV MOD` (`F-Env`) is enabled. Out-of-common-core (the canonical schema treats env→cutoff and env→pitch as separate amounts).
- **`ENV TYPE` is published as range 0–1 in the CC table but 0–2 in the NRPN table** (NRPN 96, range 0–2). The User's Guide text only describes two contour options (OB-X/Xa vs OB-8). The third value's meaning is not documented in the v1.1 User's Guide; it may be reserved for a future emulation mode (or for the OB-X "type" specifically distinct from OB-Xa, since the Page-2 `Envelope Type` parameter labels its options as "OB-X/Xa" and "OB-8" while the front-panel Filter `TYPE` LED labels distinguish OB-X from OB-Xa). Recorded as 0–1 toggle here, with the NRPN-side range noted.
- **No on-board effects engine.** The OB-X8 has no delay, reverb, chorus, distortion, or BBD — clean output to the audio jacks. Survey scope is therefore the entire audio signal path of the device.
- **Vintage knob (`VINTAGE`, CC 3 / NRPN 26, 0–127).** Dials in randomized inter-voice variation across VCO, VCF, envelope, LFO, and amplifier behavior — emulates the per-voice calibration drift of the original analog-era OB-X / OB-Xa / OB-8 hardware. Also scales `Page 2 Voice Detune Range`. **Out-of-common-core (vintage-knob simulation).**
- **8 voices, monotimbral by default, bi-timbral in Split / Double mode.** Split and Double layer two programs (Lower / Upper) with 4 voices each (or per the saved Page 2 split balance), which exposes a separate set of Split-Mode Page 2 parameters (Balance, Split Point, Lower Transpose, Upper Transpose, Lower Detune, Pan Mode, Pan Width, Hold/Chord Params, Lever Box Params, Arp Params). Out-of-common-core (split / dual-timbral mode).
- **Master Volume is a non-programmable physical pot** (per User's Guide p. 34). CC 7 affects MIDI-side master volume but not the front-panel pot.
- **Note Off velocity is ignored.** Per *Received Channel Messages* table — `Note Off. Velocity is ignored`.
- **`Polyphonic Key Pressure` is received but the OB-X8 has only monophonic (channel) aftertouch routing.** Per the *Received Channel Messages* table, status `1010 nnnn` is listed. The User's Guide describes aftertouch as "monophonic (or 'channel') aftertouch" — applying pressure to any key in a chord modulates all held notes. Hosts can send poly-pressure but the destination is still channel-wide.
- **Pitch-bend amount is a discrete-stepped, knob-set value.** Set via Mod-Box `AMOUNT` switch + a keyboard key from C0 (¼ semitone) to C1 (12 semitones) per Hidden/Panel Access Functions §6. The chart row name is `MOD BOX LFO 2 BEND AMOUNT` (CC 1, 0–255; NRPN 72, 0–12). Recorded but not normalized.
- **OS 2.0 (July 2024) Addendum exists and adds further parameters** — this survey describes the v1.1 User's Guide (Aug 2022) and v2.2 Manual (Sept 2023) baseline. The OS 2.0 addendum is cited as a source for completeness but its delta is not folded into the parameter rows above; any post-v2.2 features should be cross-checked against the addendum before patching.
