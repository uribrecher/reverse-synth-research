# Prophet-5 — Parameter Survey

**Manufacturer:** Sequential
**Year:** 2020 (Rev 4 reissue of the 1978 Sequential Circuits Prophet-5; this survey describes the Rev 4 hardware specifically)
**Engine type:** analog
**Polyphony:** 5 (the sibling Prophet-10 Rev 4 is the same panel and firmware with 10 voices — see Quirks)
**Sources:**
- Sequential, *Prophet-5 User's Guide* v1.3, Feb 2021: https://sequential.com/wp-content/uploads/2021/02/Prophet-5-Users-Guide-1.3.pdf
- Sequential, *Prophet-5 MIDI Implementation* v1.4, Mar 2021: https://sequential.com/wp-content/uploads/2021/03/Prophet-5-MIDI-Implementation-1.4.pdf
- Sequential product page (Prophet-5 / Prophet-10): https://sequential.com/product/prophet-5/

No `keyboards-mcp` cross-check is available for this device — sourcing is web-only. CC and NRPN numbers below are taken row-by-row from the Sequential MIDI Implementation v1.4 chart; parameter names are reproduced verbatim from that chart and the User's Guide v1.3.

## Signal path

Two analog VCOs feed a mixer that also receives a dedicated white-noise generator. Oscillator A produces sawtooth and variable-width pulse waves (independently switchable); Oscillator B produces sawtooth, triangle, and variable-width pulse waves (independently switchable, simultaneously). Oscillator A can be hard-synced to Oscillator B via an `OSC SYNC` switch (out-of-common-core). Oscillator B has two extra modes — a `LO FREQ` switch that drops it into LFO range and a `KEYBOARD` on/off switch that disables keyboard tracking — so Osc B can serve as a per-voice modulation oscillator (out-of-common-core).

The mixer output passes into a 4-pole, 24 dB/octave resonant low-pass filter, which can self-oscillate at maximum resonance. The Rev 4 hardware ships with **two** filter implementations selectable per-program via the `REV` switch: `REV 1/2` is a Dave Rossum-designed SSI 2140 (functionally identical to the Rev 1/2 SSM 2040), `REV 3` is a Doug Curtis CEM 3320. Switching `REV` also reshapes the Filter Envelope's response to match the SSM-vs-Curtis envelope generators of the original revisions. The `REV` switch is out-of-common-core (Rev-4-specific reissue feature).

After the filter, audio passes into a VCA. A dedicated 4-stage ADSR envelope drives the filter (with a unipolar `ENV AMOUNT` knob and a separate signed Poly Mod amount); a second 4-stage ADSR drives the VCA. A single LFO (sawtooth, triangle, square; multiple shapes can be enabled simultaneously, summed) is routable to a fixed set of destinations via on/off switches in the `WHEEL-MOD` section, which also mixes a noise generator into the mod path (`SOURCE MIX` LFO↔Noise). A separate Poly Mod section provides two additional per-voice modulation sources (Filter Envelope, Oscillator B) into a fixed destination set with bipolar amounts — out-of-common-core. Unison stacks all five voices on one note (with detune); Glide gives portamento; the `VINTAGE` knob (Rev-4-specific) adds randomness across voices. There is no on-board effects engine.

## Parameters

All ranges shown as `0–120 (raw MIDI CC)` reflect what the published MIDI Implementation chart specifies — Sequential maps these front-panel knobs onto a 0–120 CC range (not 0–127). Exceptions where the chart specifies 0–127 (`OSC B FINE TUNE`, `BRIGHTNESS`, `VINTAGE`, `POLY MOD FILT ENV AMOUNT`) are recorded as such. Toggles use 0–1; multi-step discretes use the chart's stated range. Per the Implementation chart's introduction, **NRPNs are the preferred method of parameter transmission, since they cover the complete range of all parameters, while CCs are limited to a range of 128**. NRPN numbers (when given) are from the *Control NRPN Data* tables on pages 8–9 of the chart. Parameter names below are reproduced verbatim from the chart in `MONOSPACE` casing, and from the User's Guide where the User's Guide provides the panel label (e.g. `Frequency`, `Initial Amount`).

### Oscillator A
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| OSC A FREQUENCY | continuous | 0–120 (raw MIDI CC 3) | semitones (knob taper) | — | Oscillator A base frequency over a 4-octave range, semitone steps. NRPN 0, range 0–120. |
| OSC A LEVEL | continuous | 0–120 (raw MIDI CC 27) | level | — | Oscillator A output level into the mixer. NRPN 14, range 0–120. |
| OSC A SAW ON/OFF | toggle | 0–1 (raw MIDI CC 15) | enum | Off / On | Sawtooth wave enable for Osc A. NRPN 3, 0–1. |
| OSC A SQUARE ON/OFF | toggle | 0–1 (raw MIDI CC 20) | enum | Off / On | Square/pulse wave enable for Osc A. NRPN 4, 0–1. (Multiple shapes can be enabled simultaneously per the User's Guide.) |
| OSC A PULSE WIDTH | continuous | 0–120 (raw MIDI CC 21) | pulse-width | — | Pulse width of Osc A square/pulse wave; square at 12 o'clock, narrow pulse at extremes. NRPN 8, 0–120. |

### Oscillator B
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| OSC B FREQUENCY | continuous | 0–120 (raw MIDI CC 9) | semitones (knob taper) | — | Oscillator B base frequency. 4-octave range with `KEYBOARD ON`; 9-octave range with `KEYBOARD OFF` per User's Guide. NRPN 1, 0–120. |
| OSC B FINE TUNE | continuous | 0–127 (raw MIDI CC 14) | cents | — | Fine tune Oscillator B upward — User's Guide states "range of nearly a semitone." NRPN 2, 0–127. |
| OSC B LEVEL | continuous | 0–120 (raw MIDI CC 28) | level | — | Oscillator B output level into the mixer. NRPN 15, 0–120. |
| OSC B SAW ON/OFF | toggle | 0–1 (raw MIDI CC 30) | enum | Off / On | Sawtooth wave enable for Osc B. NRPN 5, 0–1. |
| OSC B TRI ON/OFF | toggle | 0–1 (raw MIDI CC 52) | enum | Off / On | Triangle wave enable for Osc B (Osc B only — Osc A has no triangle). NRPN 6, 0–1. |
| OSC B SQUARE ON/OFF | toggle | 0–1 (raw MIDI CC 116) | enum | Off / On | Square/pulse wave enable for Osc B. NRPN 7, 0–1. |
| OSC B PULSE WIDTH | continuous | 0–120 (raw MIDI CC 22) | pulse-width | — | Pulse width of Osc B square/pulse wave. NRPN 9, 0–120. |

Out-of-common-core (Osc B dual-role and oscillator sync — recorded under Quirks):
- `OSC SYNC ON/OFF` — toggle 0–1 (raw MIDI CC 23, NRPN 10). Hard-syncs Osc A to Osc B (Osc A is slave, Osc B is master per User's Guide).
- `OSC B LOW FREQ ON/OFF` — toggle 0–1 (raw MIDI CC 24, NRPN 11). Drops Osc B into LFO range, giving five free-running per-voice LFOs that track the keyboard if `OSC B KEYBOARD` is on.
- `OSC B KEYBOARD ON/OFF` — toggle 0–1 (raw MIDI CC 25, NRPN 12). Disables keyboard tracking on Osc B (drone / mod-source mode).

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| NOISE LEVEL | continuous | 0–120 (raw MIDI CC 29) | level | — | White noise generator output level into the mixer. NRPN 16, 0–120. |

(Osc A Level and Osc B Level are also Mixer-section panel knobs but are listed under their respective oscillators above to keep parameter rows next to their owning section.)

### Filter
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| CUTOFF | continuous | 0–120 (raw MIDI CC 73) | cutoff (knob taper) | — | Filter cutoff frequency. NRPN 17, 0–120. |
| BRIGHTNESS | continuous | 0–127 (raw MIDI CC 74) | added cutoff offset | — | Performance/MIDI offset added to filter cutoff frequency (see *Received Controller Messages*: "Brightness: Added to filter cutoff frequency"). 0–127 because it is a standard MIDI controller, not a panel knob. |
| RESONANCE | continuous | 0–120 (raw MIDI CC 31) | resonance | — | Filter resonance; self-oscillates at maximum. NRPN 18, 0–120. |
| FILTER KEYBOARD TRACK OFF/HALF/FULL | discrete | 0–2 (raw MIDI CC 35) | enum | Off / Half / Full | 3-position keyboard-tracking switch into filter cutoff. NRPN 19, 0–2. |
| FILTER REV SELECT | toggle | 0–1 (raw MIDI CC 41) | enum | Rev 1/2 (SSI 2140) / Rev 3 (CEM 3320) | Selects the filter type. **Out-of-common-core (Rev-4-specific feature; also reshapes the Filter Envelope response — see Quirks).** NRPN 20, 0–1. |
| ENVELOPE FILTER AMOUNT | continuous | 0–120 (raw MIDI CC 89) | amount | — | Filter Envelope amount into filter cutoff. Unipolar on this device (User's Guide describes "0...10" range; "Any setting above zero means that each time you strike a key, the filter envelope controls how the filter opens and closes"). NRPN 40, 0–120. |
| ENVELOPE FILTER VELOCITY ON/OFF | toggle | 0–1 (raw MIDI CC 90) | enum | Off / On | Velocity sensitivity for filter envelope into filter cutoff. NRPN 41, 0–1. |

### Filter envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| ATTACK FILTER | continuous | 0–120 (raw MIDI CC 103) | time (knob taper) | — | Filter envelope attack time. NRPN 43, 0–120. |
| DECAY FILTER | continuous | 0–120 (raw MIDI CC 105) | time (knob taper) | — | Filter envelope decay time. NRPN 45, 0–120. |
| SUSTAIN FILTER | continuous | 0–120 (raw MIDI CC 107) | level | — | Filter envelope sustain level. NRPN 47, 0–120. |
| RELEASE FILTER | continuous | 0–120 (raw MIDI CC 109) | time (knob taper) | — | Filter envelope release time. NRPN 49, 0–120. |

### Amp envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| ATTACK VCA | continuous | 0–120 (raw MIDI CC 104) | time (knob taper) | — | Amplifier envelope attack time. NRPN 44, 0–120. |
| DECAY VCA | continuous | 0–120 (raw MIDI CC 106) | time (knob taper) | — | Amplifier envelope decay time. NRPN 46, 0–120. |
| SUSTAIN VCA | continuous | 0–120 (raw MIDI CC 108) | level | — | Amplifier envelope sustain level. NRPN 48, 0–120. |
| RELEASE VCA | continuous | 0–120 (raw MIDI CC 110) | time (knob taper) | — | Amplifier envelope release time. NRPN 50, 0–120. |
| ENVELOPE VCA VELOCITY ON/OFF | toggle | 0–1 (raw MIDI CC 102) | enum | Off / On | Velocity sensitivity for VCA envelope (User's Guide *Velocity Switch: Filt, Amp* — when on, harder strikes increase volume). NRPN 42, 0–1. |
| RELEASE ON/OFF | toggle | 0–1 (raw MIDI CC 111) | enum | Off / On | Front-panel `RELEASE` switch that gates whether the Release knob value is used for both Filter and Amp envelopes (off → release is fast on both). NRPN 51, 0–1. |

### LFO
The Prophet-5 LFO produces sawtooth, triangle, and square waveshapes; **multiple shapes can be enabled simultaneously**, summed (User's Guide, p. 32). The LFO is routed to destinations via on/off switches in the `WHEEL-MOD` section (next subsection).

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO FREQUENCY | continuous | 0–120 (raw MIDI CC 46) | rate (knob taper, ≈ 0.022–500 Hz per User's Guide) | — | LFO rate. NRPN 21, 0–120. |
| LFO INITIAL AMOUNT | continuous | 0–120 (raw MIDI CC 47) | amount | — | Always-on LFO modulation depth applied without mod wheel; the Mod Wheel adds on top of this (User's Guide p. 32). NRPN 22, 0–120. |
| LFO SAW ON/OFF | toggle | 0–1 (raw MIDI CC 117) | enum | Off / On | LFO sawtooth shape enable. NRPN 23, 0–1. |
| LFO TRI ON/OFF | toggle | 0–1 (raw MIDI CC 118) | enum | Off / On | LFO triangle shape enable. NRPN 24, 0–1. |
| LFO SQUARE ON/OFF | toggle | 0–1 (raw MIDI CC 119) | enum | Off / On | LFO square shape enable. NRPN 25, 0–1. |

### Wheel-Mod (LFO/Noise routing)
The `WHEEL-MOD` section is the LFO's destination matrix. It also mixes a noise modulation source into the mod bus.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO SOURCE MIX | continuous | 0–120 (raw MIDI CC 53) | mix | — | Mix between LFO (fully CCW) and Noise (fully CW) as the modulation source. NRPN 26, 0–120. |
| LFO FREQ A ON/OFF | toggle | 0–1 (raw MIDI CC 54) | enum | Off / On | Route LFO/Noise to Osc A frequency. NRPN 27, 0–1. |
| LFO FREQ B ON/OFF | toggle | 0–1 (raw MIDI CC 55) | enum | Off / On | Route LFO/Noise to Osc B frequency. NRPN 28, 0–1. |
| LFO FREQ PW A ON/OFF | toggle | 0–1 (raw MIDI CC 56) | enum | Off / On | Route LFO/Noise to Osc A pulse width. NRPN 29, 0–1. |
| LFO FREQ PW B ON/OFF | toggle | 0–1 (raw MIDI CC 57) | enum | Off / On | Route LFO/Noise to Osc B pulse width. NRPN 30, 0–1. |
| LFO FILTER ON/OFF | toggle | 0–1 (raw MIDI CC 58) | enum | Off / On | Route LFO/Noise to filter cutoff. NRPN 31, 0–1. |

### Poly Mod (out of common-core)
**The entire Poly Mod section is out-of-common-core** (cross-mod / per-voice mod-matrix slot — see Quirks). Two sources (Filter Envelope, Oscillator B) into a fixed three-destination set (Freq A, PW A, Filter) with bipolar amounts.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| POLY MOD FILT ENV AMOUNT | continuous | 0–127 (raw MIDI CC 59) | bipolar amount | — | Filter Envelope amount into the Poly Mod destination set. **Out-of-common-core.** NRPN 32, 0–127. |
| POLY MOD OSC B AMOUNT | continuous | 0–120 (raw MIDI CC 60) | bipolar amount | — | Osc B amount into the Poly Mod destination set (FM at audio rate when Osc B is in audio range). **Out-of-common-core.** NRPN 33, 0–120. |
| POLY MOD FREQ A ON/OFF | toggle | 0–1 (raw MIDI CC 61) | enum | Off / On | Poly Mod destination: Osc A frequency. **Out-of-common-core.** NRPN 34, 0–1. |
| POLY MOD PW ON/OFF | toggle | 0–1 (raw MIDI CC 62) | enum | Off / On | Poly Mod destination: Osc A pulse width. **Out-of-common-core.** NRPN 35, 0–1. |
| POLY MOD FILTER ON/OFF | toggle | 0–1 (raw MIDI CC 63) | enum | Off / On | Poly Mod destination: filter cutoff. **Out-of-common-core.** NRPN 36, 0–1. |

### Voice (unison, glide, key priority)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| GLIDE RATE | continuous | 0–120 (raw MIDI CC 26) | rate (knob taper) | — | Portamento rate. NRPN 13, 0–120. |
| UNISON ON/OFF | toggle | 0–1 (raw MIDI CC 112) | enum | Off / On | Stacks all voices on a single note. **Out-of-common-core (voice-allocation mode).** NRPN 52, 0–1. |
| UNISON VOICE COUNT | discrete | 0–10 (raw MIDI CC 113) | enum | (voice count) | Number of voices stacked in unison. **Out-of-common-core.** NRPN 53, 0–10. (Range goes to 10 to accommodate Prophet-10 firmware on the same panel — see Quirks.) |
| UNISON DETUNE | discrete | 0–7 (raw MIDI CC 114) | amount | — | Detune amount between stacked unison voices. **Out-of-common-core.** NRPN 54, 0–7. |
| RETRIGGER AND UNISON ASSIGN | discrete | 0–3 (raw MIDI CC 71) | enum | Low / Low Retrigger / Last / Last Retrigger | Key Priority Mode for Unison (User's Guide *Key Priority Modes*). NRPN 87, 0–3. **Out-of-common-core (key-assign / voice-allocation behavior).** |

### Master / Global
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| MASTER VOLUME | continuous | 0–120 (raw MIDI CC 7) | level | — | Master MIDI volume. **Note:** the Master Volume knob is a non-programmable physical pot per User's Guide (it sits outside Pot Mode), so this CC reflects MIDI-side master volume only. |
| Mod Wheel | continuous | 0–127 (raw MIDI CC 1) | level | — | Performance Mod Wheel; scales the Wheel-Mod routing depth additively on top of `LFO INITIAL AMOUNT`. |
| Damper pedal | toggle | 0–127, threshold (raw MIDI CC 64) | enum | Off (<64) / On (≥64) | Standard sustain pedal — "Holds envelopes in Sustain if 0100 0000 or higher" per the Received Controller Messages table. |
| Foot Controller | continuous | 0–127 (raw MIDI CC 4) | level | — | Foot Controller — "directly assignable controller" per the Received Controller Messages table. |
| PITCH WHEEL RANGE | discrete | 0–11 (raw MIDI CC 70) | semitones | — | Pitch-bend range in semitones (User's Guide states "1–12"; chart range 0–11). NRPN 86, 0–11. |
| VINTAGE | continuous | 0–127 (raw MIDI CC 85) | amount (panel labelled 4 → 1, stable→vintage) | — | **Vintage knob — Rev-4-specific.** Adds randomness to VCOs, envelopes, LFOs, and amplifiers per voice to emulate Rev 1/Rev 2 vintage drift. **Out-of-common-core.** NRPN 37, 0–127. |
| PRESSURE FILTER | toggle | 0–1 (raw MIDI CC 86) | enum | Off / On | Routes channel aftertouch to filter cutoff (User's Guide *Adding Aftertouch*, panel name `AFTERTOUCH > FILT`). NRPN 38, 0–1. **Out-of-common-core (performance / per-voice pressure routing).** |
| PRESSURE LFO | toggle | 0–1 (raw MIDI CC 87) | enum | Off / On | Routes channel aftertouch to LFO amount (User's Guide *Adding Aftertouch*, panel name `AFTERTOUCH > LFO`; the chart's NRPN row labels this `AFTERTOUCH > AMP` — same parameter). NRPN 39, 0–1. **Out-of-common-core.** |
| VELOCITY > FILTER | toggle | 0–1 (NRPN) | enum | Off / On | NRPN 41, 0–1. (Same parameter as `ENVELOPE FILTER VELOCITY ON/OFF` row above; the chart lists it both ways.) |
| VELOCITY > AMP | toggle | 0–1 (NRPN) | enum | Off / On | NRPN 42, 0–1. (Same parameter as `ENVELOPE VCA VELOCITY ON/OFF` row above; chart lists both names.) |
| TRANSPOSE | discrete | 0–24 (NRPN) | semitones | — | Master transpose ±12 semitones. NRPN 4096, global. |
| MIDI CHANNEL | discrete | 0–16 (NRPN) | channel | All / 1..16 | NRPN 4097, global. |
| ALT TUNINGS | discrete | 0–15 (NRPN) | enum | Normal + 16 alternative scales | Alternative tuning selection (User's Guide Appendix D). NRPN 4107, global. |
| AT RESPONSE | discrete | 0–7 (NRPN) | curve | Curve 0..7 | Aftertouch response curve. NRPN 4109, global. |
| VEL RESPONSE | discrete | 0–7 (NRPN) | curve | Curve 0..6 | Velocity response curve (User's Guide states 0..6 = 7 curves; chart range 0–7). NRPN 4108, global. |
| UNISON NOTE | discrete | 1–10 per slot (NRPN) | chord-memory note | — | Chord Memory note slots (up to 5 notes for Prophet-5). NRPN 55–64, range 1–10. **Out-of-common-core.** |

Pedal Mode (NRPN 4106), MIDI SysEx (NRPN 4101), MIDI Out (NRPN 4102), Local Control (NRPN 4103), Pot Mode (NRPN 4104, Relative/Passthru/Jump), Sustain Mode (NRPN 4105, Release/Hold), Param Xmit (NRPN 4098), Param Rcv (NRPN 4099), MIDI Control (NRPN 4100) are global system parameters, recorded for completeness — out-of-common-core (system / configuration domain, not tone generation).

## Modulation routing

The Prophet-5 has a small fixed set of per-voice modulation paths plus one global LFO with explicit destination switches.

- **LFO destinations (Wheel-Mod section)** — verbatim panel labels: `FREQ A` (Osc A frequency), `FREQ B` (Osc B frequency), `PW A` (Osc A pulse width), `PW B` (Osc B pulse width), `FILTER` (filter cutoff). Each is an independent on/off, so the LFO is broadcast to any subset.
- **LFO source mixed with noise** via `SOURCE MIX` (LFO ↔ Noise blend).
- **LFO depth scaling:** `INITIAL AMOUNT` is the always-on depth; `Mod Wheel` adds on top at performance time.
- **LFO shapes:** `Sawtooth`, `Triangle`, `Square` (User's Guide p. 32) — multiple may be enabled simultaneously, summed.
- **Aftertouch fixed routings** (User's Guide *Adding Aftertouch*, p. 38–39): `Filter` (filter cutoff) and/or `LFO` (LFO amount). Both can be enabled simultaneously. Out-of-common-core.
- **Velocity → Filter envelope** via `ENVELOPE FILTER VELOCITY ON/OFF`.
- **Velocity → VCA envelope** via `ENVELOPE VCA VELOCITY ON/OFF`.
- **Filter envelope → filter cutoff** via unipolar `ENVELOPE FILTER AMOUNT`.

Out-of-common-core (Poly Mod):

- **Poly Mod sources:** `Filter Envelope`, `Oscillator B`. **Poly Mod destinations:** `Freq A` (Osc A frequency), `PW A` (Osc A pulse width), `Filter` (filter cutoff). Bipolar amount per source. Out-of-common-core (per-voice cross-mod / FM / dedicated mod-matrix slot).
- **Osc B as second LFO/mod source:** `OSC B LOW FREQ` + `OSC B KEYBOARD OFF` turns Osc B into a free-running per-voice modulation oscillator that can drive Poly Mod. Out-of-common-core.

Sources for routing: User's Guide v1.3 pp. 31–35 (Low Frequency Oscillator, Wheel-Mod Controls, Poly Mod) and Appendix C of the MIDI Implementation v1.4 chart.

## Quirks

- **Monotimbral.** All program parameters are global to a single program at a time, on a single MIDI channel for all voices. Globals (transpose, MIDI channel, etc.) are separate from program parameters.
- **CC range is 0–120 for most front-panel knobs** (not 0–127). Per the published MIDI Implementation chart, parameters such as `OSC A FREQUENCY`, `CUTOFF`, `RESONANCE`, `LFO FREQUENCY`, the four ADSR knobs of each envelope, etc. are sent and received on `0–120`. Standard MIDI controllers (`BRIGHTNESS`, `Mod Wheel`, `Damper`) and a small set of program parameters (`OSC B FINE TUNE`, `VINTAGE`, `POLY MOD FILT ENV AMOUNT`) use `0–127`. Hosts must respect this difference or risk values rolling off near the top of the knob.
- **NRPN is the preferred transmission method.** The MIDI chart introduction states this explicitly; CCs are quantized to 128 steps while NRPNs cover the full parameter resolution.
- **Vintage knob (`VINTAGE`, CC 85 / NRPN 37, 0–127).** Rev-4-specific. Panel labelled `4 → 1` (stable Rev 4 → temperamental Rev 1). Dials randomness into VCO, envelope, LFO, and amplifier behavior across voices — emulates inter-voice drift. Out-of-common-core (vintage-knob / per-voice analog-modeling parameter).
- **Filter `REV` switch (Rev-4-specific).** Selects between SSI 2140 (Rev 1/2 character) and CEM 3320 (Rev 3 character). The switch also reshapes the Filter Envelope's response to match the original revision's envelope generator (SSM-style flat/linear vs. Curtis-style curved). Out-of-common-core (Rev-4-only feature; the user must emulate this on devices without the dual-filter option).
- **Oscillator hard sync (`OSC SYNC`, NRPN 10).** Osc A is the slave, Osc B is the master per User's Guide p. 18. Out-of-common-core.
- **Osc B dual-role.** `OSC B LOW FREQ` (NRPN 11) drops Osc B into LFO range; `OSC B KEYBOARD ON/OFF` (NRPN 12) disables keyboard tracking. Combined with Poly Mod's Osc-B source, this turns Osc B into five free-running per-voice modulation oscillators. Out-of-common-core.
- **Poly Mod (cross-mod / FM).** Two sources (Filter Env, Osc B) into three destinations (Freq A, PW A, Filter), bipolar amounts. Out-of-common-core (cross-mod / dedicated mod-matrix slot).
- **No on-board effects engine.** No delay, reverb, chorus, distortion, etc. — clean output to the audio jack. Survey scope is therefore the entire audio signal path on this device.
- **Rev-4 reissue MIDI is richer than the 1978 original.** The 1978 hardware predated MIDI; the Rev 4 (2020) reissue ships with a full MIDI Implementation including CC and NRPN coverage of program parameters, plus SysEx for program/bank/global dumps. **Any CC number quoted from this survey applies to the Rev 4 reissue, not the original — the original had no MIDI at all.** This is an important caveat when sourcing legacy patch sheets that pre-date the reissue.
- **Sibling Prophet-10 Rev 4 shares this panel and firmware.** The Prophet-10 Rev 4 is a separate product (twin-board, 10 voices) but uses the same front panel and the same MIDI implementation; Sequential publishes a single set of documents (`Prophet-5/10` is the USB MIDI port name per User's Guide p. 4). The parameter set is identical between the two — only voice count differs. Sequential offered Prophet-10 firmware as a paid upgrade for a window after release per Sequential product communications, but it is sold as a distinct hardware model. The `UNISON VOICE COUNT` CC range of 0–10 reflects this shared firmware. Treat any value labelled "Prophet-5 Rev 4" as belonging to the 5-voice device specifically; values inferred from Prophet-10 sources are valid for parameters but not for polyphony.
- **Master Volume is a non-programmable physical pot.** Per User's Guide p. 14, "Master volume is not programmable, so these [Pot Mode] modes don't apply." CC 7 affects MIDI-side master volume but not the front-panel pot.
- **`Brightness` is a performance offset, not the panel `Cutoff` knob.** CC 74 maps to a *Brightness* MIDI-only parameter that *adds* to the filter cutoff frequency (per the Received Controller Messages table). The panel `CUTOFF` knob itself maps to CC 73 / NRPN 17.
- **Note Off velocity is ignored.** Per the Received Channel Messages table — `Note Off. Velocity is ignored`.
- **Polyphonic Key Pressure is received but the Prophet-5 has only monophonic (channel) aftertouch routing.** The MIDI chart lists `1010 nnnn` (Polyphonic Key Pressure) as a received status byte; the User's Guide describes aftertouch as "monophonic (or 'channel') aftertouch, which means that applying pressure to any key within a chord will apply modulation to *all* notes currently held" (p. 38). Hosts can send poly-pressure but the destination is still channel-wide.
- **Pitch Wheel Range parameter range mismatch.** The MIDI chart records `PITCH WHEEL RANGE` as 0–11 (CC 70 / NRPN 86) while the User's Guide describes it as 1–12 semitones. The likely interpretation: chart value 0 = 1 semitone, chart value 11 = 12 semitones (zero-indexed semitone count). Recorded but not normalized in this survey.
