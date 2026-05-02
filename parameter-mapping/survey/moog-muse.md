# Moog Muse — Parameter Survey

**Manufacturer:** Moog Music
**Year:** 2024
**Engine type:** analog
**Polyphony:** 8 (8-voice, bi-timbral — Timbre A and Timbre B; layered/stacked use halves the per-timbre polyphony to 4+4 per Music Tech and Sound on Sound reviews; this survey covers single-timbre Layer A only.)
**Sources:**
- Moog Music, *Muse Quickstart Guide* (2024-07): https://api.moogmusic.com/sites/default/files/2024-07/MUSE-quickstart_Eng.pdf (mirrored at https://cdn.inmusicbrands.com/Moog/Muse/MUSE-quickstart_Eng.pdf)
- Moog Music, *Muse Manual* v1.4.0: https://cdn.inmusicbrands.com/Moog/Muse/Muse_Manual-1.4.0.pdf (also hosted by Kraft Music: https://files.kraftmusic.com/media/ownersmanual/Moog_Muse_User_Manual.pdf and B&H: https://www.bhphotovideo.com/lit_files/1140244.pdf)
- midi.guide community CC database (Moog Muse): https://midi.guide/d/moog/muse/
- Sound on Sound, *Moog Muse* review: https://www.soundonsound.com/reviews/moog-muse
- Music Tech, *Moog Muse review*: https://musictech.com/reviews/hardware-instruments/moog-muse-review/

No `keyboards-mcp` cross-check is available for this device — sourcing is web-only. CC numbers and value ranges below are taken row-by-row from the midi.guide community chart (which mirrors the MIDI implementation appendix shipped with the Muse Manual v1.4.0). The Muse's MIDI implementation as published does **not document NRPN parameter numbers** — the chart at midi.guide records 0 NRPNs for the device, and the manual's MIDI appendix similarly publishes only CC assignments. **This is a gap relative to the Sequential / Oberheim chart conventions used elsewhere in this survey set, and is recorded under Quirks.** Parameter names below are reproduced verbatim from the midi.guide chart and cross-checked against panel labels described in the Sound on Sound and Music Tech reviews.

## Signal path

Two per-voice analog VCOs (`Oscillator 1`, `Oscillator 2`) with octave-foot-switch (16′ / 8′ / 4′ / 2′), a continuously-variable triangle↔saw `Tri/Saw Mix`, a separate variable-pulse-width pulse-wave path, and a `Wave Mix` blend between the tri/saw output and the pulse output. The two oscillators support hard sync (`Oscillator 2>1 SYNC` — Osc 1 is reset by Osc 2) and **bi-directional FM** between the two oscillators (`Oscillator 2>1 FM`, `Oscillator 1>2 FM`, scaled by `FM Amount`). Per Music Tech the oscillators are "based on the Voyager design" with sync and bidirectional FM. **Hard sync, FM, and ring mod are out-of-common-core.**

A third oscillator, the **Modulation Oscillator**, sits alongside the two main VCOs. It is a free-running oscillator with five waveshapes (`Sine` / `Sawtooth` / `Ramp` / `Square` / `Noise` per CC 28), a frequency knob, and switches for `Audio` (mixes its output into the audio mixer at audio rate), `KB Track` (1V/oct keyboard tracking on/off), `KB Reset` (key-trigger phase reset), and `Unipolar` (force the modulation source to unipolar). Per Music Tech: "tracks the keyboard perfectly" but can disconnect for drone / constant-modulation use. The Modulation Oscillator has **its own dedicated set of mod destinations with per-destination amounts** — `Pitch Amount` → `Pitch OSC 1` / `Pitch OSC 2`, `PWM Amount` → `PWM OSC 1` / `PWM OSC 2`, `Filter Amount` → `Filter 1` / `Filter 2`, `VCA Amount` → `VCA PAN` — and a `Modulation Oscillator Level` parameter for its audio-rate path into the mixer. **The Modulation Oscillator's role as a third audio-rate oscillator and as a fixed-routing modulation source with per-destination amounts is out-of-common-core.**

The mixer is based on the **Moog CP3 module** (per Sound on Sound and Music Tech). It mixes Osc 1, Osc 2, the Modulation Oscillator (when `Audio` is on), the Osc 1/Osc 2 ring modulator (per Sound on Sound: "inspired by the Moogerfooger MF-102"), a white-noise generator, and a `Clipping Level` overload path. Per Sound on Sound: "at low levels, it does this without clipping, but you can overload it to distort and fatten the signal" — the CP3-style mixer provides **soft analog clipping** when overdriven, and the dedicated `Clipping Level` parameter exposes this drive amount as a saved patch parameter. **The Clipping/feedback drive path is out-of-common-core (mixer saturation as a patch parameter that interacts with cutoff perception — see Quirks).**

After the mixer, audio passes into **two discrete Moog ladder filters** (per Music Tech: "Moog 904a-based; 24 dB/octave"). `Filter 1` is a multi-mode filter with both a `High Pass` cutoff knob and a `Cutoff` (low-pass) knob (per Sound on Sound: "Filter 1 offers high-pass and low-pass"); `Filter 2` is low-pass only with `Frequency` and `Resonance`. Both filters publish a `Resonance` parameter and a 3-position `KB Tracking` enum (`Off` / `Half` / `Full`). The two filters are routable in three configurations via `Filters Order`: `Serial` (Filter 1 → Filter 2), `Stereo` (one filter per stereo channel), or `Parallel` (sum). A `Link Filters` toggle ties the two filters' parameters together. **The dual-filter topology, the slope, the high-pass mode of Filter 1, and the serial/stereo/parallel routing matrix are all out-of-common-core (the canonical schema's common-core has a single LP filter).**

After the filter section, audio passes into a VCA. **Two ADSR envelopes** drive the engine: a `Filter Env` ADSR with `Attack` / `Sustain` / `Decay` / `Release` (note the chart's row-order quirk — `Decay` is published as CC 81 between `Sustain` (CC 80) and `Release` (CC 82); the parameter is still a standard ADSR decay-time per the manual), plus `Loop` and `Velocity` toggles; and a `VCA Env` ADSR with the same A/S/D/R + Loop + Velocity layout (CCs 86–91). Per Music Tech the envelopes are "dual ADSR envelopes per-voice with variable curves, looping, velocity response, and assignable to any destination."

**The Muse does NOT have a dedicated third per-voice "Modulation Envelope" or "Mod Env" ADSR.** This contradicts a working assumption in the upstream task brief — the published MIDI chart (midi.guide) and the Music Tech / Sound on Sound reviews consistently describe **two** per-voice envelopes (filter, amp), both loopable. Firmware 1.4 (May 2025 per Gearnews) added two **global** envelopes available as modulation sources in the mod matrix, but they are global (not per-voice), and the global envelopes are not exposed as MIDI CCs in the published chart. **The "third envelope" common-core flag is therefore not applicable to the Muse — the device has two per-voice envelopes, not three.** The closest equivalent is the per-voice Pitch LFO, which can be configured as an attack-decay envelope per the Music Tech review ("a pitch LFO which can operate as an attack decay envelope") — see LFO 2 / Pitch LFO subsection below.

The Muse exposes **two per-voice modulation LFOs** (`LFO 1`, `LFO 2`) plus a third dedicated **Pitch LFO** for vibrato. Each of `LFO 1` and `LFO 2` has a `Rate`, an `Amount`, and a `Waveform` enum with five shapes (`Triangle` / `Saw` / `Square` / `Random` / `User Wave` per CC 14, CC 17). The Pitch LFO is a vibrato-dedicated LFO with `Rate`, `Shape`, `Amount`, plus a fixed destination set (`Pitch LFO OSC 1`, `Pitch LFO OSC 2`, `Pitch LFO Mod OSC`, `Pitch LFO Detune` — each is an independent on/off). **LFO 2, the Pitch LFO, and the multi-LFO-per-voice topology are all out-of-common-core.**

A **16-slot modulation matrix per Timbre** (per Music Tech and Sound on Sound: "16-slot modulation matrix per Timbre" with "17 sources, 17 controllers, 14 mathematical transforms, and 50 destinations" per Music Tech) gives the Muse a true mod-matrix architecture beyond the fixed-destination per-LFO routing. **The mod matrix slots are all out-of-common-core (the canonical schema's common-core has only fixed per-LFO destination switches, not a free assignable mod matrix).** As of the published v1.4 MIDI implementation chart, the mod-matrix slots' source/destination/amount fields are **not exposed as individual CCs** — they are accessible only via the panel UI, sysex preset dump, or the Muse Editor. This is a gap relative to e.g. the OB-X8's NRPN-addressable parameters; recorded under Quirks.

A `Voice Detune`, `Voice Unison`, `Voice Mono`, and a `Glide Time` knob handle voice allocation and portamento. The Muse also has on-board delay (Time Left/Right, Feedback, Character, Mix, Clock Sync, plus per-Timbre Send), a sequencer, and an arpeggiator — all post-engine and **out-of-common-core (effects / sequencer / arpeggiator domain)**. The Delay's `Feedback` parameter is distinct from the mixer's `Clipping Level` — they are two separate paths in the signal chain (CP3 mixer drive vs. delay-line feedback gain). See Quirks.

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit MIDI Continuous Controllers per the midi.guide chart (which mirrors the manual's MIDI appendix). The Muse uses two range conventions for switches and discrete enums per the chart: (a) toggles are `0-63: Off; 64-127: On` (threshold convention); (b) multi-position discretes (e.g. octave foot-switch, tracking, filter ordering) split 0–127 into equal sub-ranges with the chart explicitly labelling each. Continuous knobs use `[0–127]` raw with hardware-defined non-linear taper. **No NRPN parameter numbers are published by Moog for the Muse as of the v1.4 manual** — the canonical control surface is CC-only at the documented level, plus sysex preset dumps for full-resolution patch transfer (sysex format not reproduced here).

### Oscillator 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Oscillator 1 Octave | discrete | 0–127 (raw MIDI CC 44) | enum | 0-31: 16'; 32-63: 8'; 64-95: 4'; 96-127: 2' | Osc 1 octave foot-switch, four positions encoded as quartiles of the 0–127 CC range. |
| Oscillator 1 Frequency | continuous | 0–127 (raw MIDI CC 45) | semitones (knob taper) | — | Osc 1 fine frequency offset within the selected octave. |
| Oscillator 1 Tri/Saw Mix | continuous | 0–127 (raw MIDI CC 46) | morph | — | Continuously-variable mix between triangle (0) and sawtooth (127) waveshapes for Osc 1's tri/saw waveform path. |
| Oscillator 1 PW | continuous | 0–127 (raw MIDI CC 47) | pulse-width | — | Pulse width of Osc 1's pulse waveshape. Square at 12 o'clock, narrow at extremes. |
| Oscillator 1 Wave Mix | continuous | 0–127 (raw MIDI CC 48) | morph | — | Mix between Osc 1's Tri/Saw output (0) and its Pulse output (127) — selects which waveform path is sent to the mixer (or blends them). |
| Oscillator 1 Level | continuous | 0–127 (raw MIDI CC 58) | level | — | Osc 1 level into the CP3 mixer. |

### Oscillator 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Oscillator 2 Octave | discrete | 0–127 (raw MIDI CC 49) | enum | 0-31: 16'; 32-63: 8'; 64-95: 4'; 96-127: 2' | Osc 2 octave foot-switch (same encoding as Osc 1). |
| Oscillator 2 Frequency | continuous | 0–127 (raw MIDI CC 50) | semitones (knob taper) | — | Osc 2 fine frequency offset within the selected octave. |
| Oscillator 2 Tri/Saw Mix | continuous | 0–127 (raw MIDI CC 51) | morph | — | Continuously-variable mix between triangle (0) and sawtooth (127) waveshapes for Osc 2. |
| Oscillator 2 PW | continuous | 0–127 (raw MIDI CC 52) | pulse-width | — | Pulse width of Osc 2's pulse waveshape. |
| Oscillator 2 Wave Mix | continuous | 0–127 (raw MIDI CC 53) | morph | — | Mix between Osc 2's Tri/Saw output and its Pulse output. |
| Oscillator 2 Level | continuous | 0–127 (raw MIDI CC 59) | level | — | Osc 2 level into the CP3 mixer. |

### Oscillator interactions (sync / FM)
**All three rows below are out-of-common-core (oscillator sync, oscillator-rate FM).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Oscillator 2>1 SYNC | toggle | 0–127 (raw MIDI CC 54) | enum | 0-63: Off; 64-127: On | Hard sync — Osc 1 is reset on each cycle of Osc 2 (Osc 2 is master). **Out-of-common-core.** |
| Oscillator 2>1 FM | toggle | 0–127 (raw MIDI CC 55) | enum | 0-63: Off; 64-127: On | Routes Osc 2 as FM source into Osc 1 frequency. **Out-of-common-core.** |
| Oscillator 1>2 FM | toggle | 0–127 (raw MIDI CC 56) | enum | 0-63: Off; 64-127: On | Routes Osc 1 as FM source into Osc 2 frequency. **Out-of-common-core.** |
| FM Amount | continuous | 0–127 (raw MIDI CC 57) | amount | — | Shared FM depth scaling for both `Osc 2>1 FM` and `Osc 1>2 FM`. **Out-of-common-core.** |

### Modulation Oscillator
**The entire Modulation Oscillator section is out-of-common-core** (third audio-rate oscillator with its own fixed-routing modulation matrix).

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Modulation Oscillator Frequency | continuous | 0–127 (raw MIDI CC 25) | rate (knob taper) | — | Mod Osc base frequency. **OOC.** |
| Modulation Oscillator Audio | toggle | 0–127 (raw MIDI CC 26) | enum | 0-63: Off; 64-127: On | Routes Mod Osc into the audio mixer (audio-rate use as a third VCO). **OOC.** |
| Modulation Oscillator KB Track | toggle | 0–127 (raw MIDI CC 27) | enum | 0-63: Off; 64-127: On | 1V/oct keyboard tracking for Mod Osc. **OOC.** |
| Modulation Oscillator Waveform | discrete | 0–127 (raw MIDI CC 28) | enum | 0-24: Sine; 25-49: Sawtooth; 50-74: Ramp; 75-99: Square; 100-127: Noise | Mod Osc waveshape (5 positions). **OOC.** |
| Modulation Oscillator KB Reset | toggle | 0–127 (raw MIDI CC 29) | enum | 0-63: Off; 64-127: On | Phase-reset Mod Osc on each key trigger. **OOC.** |
| Modulation Oscillator Unipolar | toggle | 0–127 (raw MIDI CC 30) | enum | 0-63: Off; 64-127: On | Force Mod Osc modulation source to unipolar. **OOC.** |
| Modulation Oscillator Pitch Amount | continuous | 0–127 (raw MIDI CC 31) | amount | — | Mod Osc → pitch routing depth (shared scaler for Pitch OSC 1 / Pitch OSC 2 destinations). **OOC.** |
| Modulation Oscillator Pitch OSC 1 | continuous | 0–127 (raw MIDI CC 33) | amount | — | Mod Osc → Osc 1 pitch destination amount. **OOC.** |
| Modulation Oscillator Pitch OSC 2 | continuous | 0–127 (raw MIDI CC 34) | amount | — | Mod Osc → Osc 2 pitch destination amount. **OOC.** |
| Modulation Oscillator PWM Amount | continuous | 0–127 (raw MIDI CC 35) | amount | — | Mod Osc → pulse-width routing depth (shared scaler for PWM OSC 1 / PWM OSC 2). **OOC.** |
| Modulation Oscillator PWM OSC 1 | continuous | 0–127 (raw MIDI CC 36) | amount | — | Mod Osc → Osc 1 PW destination amount. **OOC.** |
| Modulation Oscillator PWM OSC 2 | continuous | 0–127 (raw MIDI CC 37) | amount | — | Mod Osc → Osc 2 PW destination amount. **OOC.** |
| Modulation Oscillator Filter Amount | continuous | 0–127 (raw MIDI CC 39) | amount | — | Mod Osc → filter cutoff routing depth (shared scaler for Filter 1 / Filter 2 destinations). **OOC.** |
| Modulation Oscillator Filter 1 | continuous | 0–127 (raw MIDI CC 40) | amount | — | Mod Osc → Filter 1 cutoff destination amount. **OOC.** |
| Modulation Oscillator Filter 2 | continuous | 0–127 (raw MIDI CC 41) | amount | — | Mod Osc → Filter 2 cutoff destination amount. **OOC.** |
| Modulation Oscillator VCA Amount | continuous | 0–127 (raw MIDI CC 42) | amount | — | Mod Osc → VCA / pan routing depth. **OOC.** |
| Modulation Oscillator VCA PAN | continuous | 0–127 (raw MIDI CC 43) | amount | — | Mod Osc → stereo pan destination amount. **OOC.** |
| Modulation Oscillator Level | continuous | 0–127 (raw MIDI CC 61) | level | — | Mod Osc audio-path level into the CP3 mixer (active when `Audio` is on). **OOC.** |

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Ring Mod Level | continuous | 0–127 (raw MIDI CC 60) | level | — | Osc 1 × Osc 2 ring modulator output level into the mixer. Per Sound on Sound: "inspired by the Moogerfooger MF-102." **Out-of-common-core (ring mod).** |
| Noise Level | continuous | 0–127 (raw MIDI CC 62) | level | — | White-noise generator output level into the mixer. |
| Clipping Level | continuous | 0–127 (raw MIDI CC 65) | drive | — | Soft-analog-clipping overload drive on the CP3 mixer output. Per Sound on Sound: "at low levels, it does this without clipping, but you can overload it to distort and fatten the signal." **Out-of-common-core (mixer saturation drive — see Quirks for cutoff interaction).** |

### Filter 1 (multi-mode: HP + LP, Moog ladder)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter 1 High Pass | continuous | 0–127 (raw MIDI CC 66) | cutoff (knob taper) | — | High-pass cutoff frequency of Filter 1. **Out-of-common-core (HP path on dual-mode filter).** |
| Filter 1 Cutoff | continuous | 0–127 (raw MIDI CC 67) | cutoff (knob taper) | — | Low-pass cutoff frequency of Filter 1 (Moog ladder, 24 dB/oct per Music Tech). |
| Filter 1 Resonance | continuous | 0–127 (raw MIDI CC 68) | resonance | — | Filter 1 resonance. |
| Filter 1 Envelope Amount | continuous | 0–127 (raw MIDI CC 69) | amount | — | Filter envelope amount routed to Filter 1 cutoff. |
| Filter 1 KB Tracking | discrete | 0–127 (raw MIDI CC 70) | enum | 0-42: Off; 43-84: Half; 85-127: Full | 3-position keyboard-tracking switch into Filter 1 cutoff. |

### Filter 2 (Moog ladder, LP only)
**Filter 2 is out-of-common-core when treated as a second filter; its individual rows are recorded for completeness.**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter 2 Frequency | continuous | 0–127 (raw MIDI CC 72) | cutoff (knob taper) | — | Filter 2 LP cutoff (Moog ladder, 24 dB/oct). **OOC (second filter).** |
| Filter 2 Resonance | continuous | 0–127 (raw MIDI CC 73) | resonance | — | Filter 2 resonance. **OOC.** |
| Filter 2 Envelope Amount | continuous | 0–127 (raw MIDI CC 75) | amount | — | Filter envelope amount routed to Filter 2 cutoff. **OOC.** |
| Filter 2 KB Tracking | discrete | 0–127 (raw MIDI CC 76) | enum | 0-42: Off; 43-84: Half; 85-127: Full | 3-position keyboard-tracking switch into Filter 2 cutoff. **OOC.** |

### Filter configuration
**Both rows out-of-common-core (dual-filter routing matrix).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Link Filters | toggle | 0–127 (raw MIDI CC 77) | enum | 0-63: Off; 64-127: On | Tie Filter 1 and Filter 2 parameters together so a single knob movement updates both. **OOC.** |
| Filters Order | discrete | 0–127 (raw MIDI CC 78) | enum | 0-42: Serial; 43-84: Stereo; 85-127: Parallel | Routing of the two filters: Serial (F1 → F2), Stereo (one per channel), or Parallel (sum). **OOC.** |

### Filter envelope (loopable ADSR)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter Env Attack | continuous | 0–127 (raw MIDI CC 79) | time (knob taper) | — | Filter envelope attack time. |
| Filter Env Sustain | continuous | 0–127 (raw MIDI CC 80) | level | — | Filter envelope sustain level. |
| Filter Env Delay | continuous | 0–127 (raw MIDI CC 81) | time (knob taper) | — | Filter envelope **decay** time — the chart row is named `Filter Env Delay` but its CC ordering (between Sustain at CC 80 and Release at CC 82) and the manual's ADSR description identify it as the standard decay-time stage. The naming "Delay" appears to be a typo in the published chart — recorded verbatim, with the correction noted under Quirks. |
| Filter Env Release | continuous | 0–127 (raw MIDI CC 82) | time (knob taper) | — | Filter envelope release time. |
| Filter Env Loop | toggle | 0–127 (raw MIDI CC 83) | enum | 0-63: Off; 64-127: On | Loop the envelope (re-trigger from Attack after Release completes). **Out-of-common-core (looping envelope as patch parameter).** |
| Filter Env Velocity | toggle | 0–127 (raw MIDI CC 85) | enum | 0-63: Off; 64-127: On | Velocity sensitivity for filter envelope amount. |

### Amp envelope (loopable ADSR)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCA Env Attack | continuous | 0–127 (raw MIDI CC 86) | time (knob taper) | — | VCA envelope attack time. |
| VCA Env Sustain | continuous | 0–127 (raw MIDI CC 87) | level | — | VCA envelope sustain level. |
| VCA Env Delay | continuous | 0–127 (raw MIDI CC 88) | time (knob taper) | — | VCA envelope **decay** time — same naming quirk as `Filter Env Delay`; recorded verbatim from the chart. |
| VCA Env Release | continuous | 0–127 (raw MIDI CC 89) | time (knob taper) | — | VCA envelope release time. |
| VCA Env Loop | toggle | 0–127 (raw MIDI CC 90) | enum | 0-63: Off; 64-127: On | Loop the VCA envelope. **Out-of-common-core (looping envelope).** |
| VCA Env Velocity | toggle | 0–127 (raw MIDI CC 91) | enum | 0-63: Off; 64-127: On | Velocity sensitivity for VCA envelope amount. |

### LFO 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 1 Rate | continuous | 0–127 (raw MIDI CC 12) | rate (knob taper) | — | LFO 1 rate. |
| LFO 1 Amount | continuous | 0–127 (raw MIDI CC 13) | amount | — | LFO 1 modulation depth (always-on; mod-matrix slots scale individual destinations on top). |
| LFO 1 Waveform | discrete | 0–127 (raw MIDI CC 14) | enum | 0-24: Triangle; 25-49: Saw; 50-74: Square; 75-99: Random; 100-127: User Wave | LFO 1 waveshape (5 positions). The `User Wave` slot is a user-defined custom waveshape per the Muse Manual v1.4. |

LFO 1 destinations are NOT addressed by per-destination on/off switches in the published CC chart — destinations are configured via the **mod matrix** (see below) which is **not exposed as MIDI CCs** in the published chart. **This is a gap relative to the OB-X8 / Prophet-5 / Prophet-6 surveys — see Quirks.**

### LFO 2
**LFO 2 is out-of-common-core (second LFO).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 2 Rate | continuous | 0–127 (raw MIDI CC 15) | rate (knob taper) | — | LFO 2 rate. **OOC.** |
| LFO 2 Amount | continuous | 0–127 (raw MIDI CC 16) | amount | — | LFO 2 modulation depth. **OOC.** |
| LFO 2 Waveform | discrete | 0–127 (raw MIDI CC 17) | enum | 0-24: Triangle; 25-49: Saw; 50-74: Square; 75-99: Random; 100-127: User Wave | LFO 2 waveshape (same 5-position enum as LFO 1). **OOC.** |

### Pitch LFO (vibrato + AD-envelope-mode)
**The entire Pitch LFO section is out-of-common-core (third LFO source / per-voice vibrato with fixed routing).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Pitch LFO Rate | continuous | 0–127 (raw MIDI CC 18) | rate (knob taper) | — | Pitch LFO rate. **OOC.** |
| Pitch LFO Shape | continuous | 0–127 (raw MIDI CC 19) | morph | — | Pitch LFO waveshape (continuous control — the Music Tech review notes the Pitch LFO can also operate as an attack-decay envelope, suggesting Shape morphs across LFO/AD-env behavior). **OOC.** |
| Pitch LFO Amount | continuous | 0–127 (raw MIDI CC 20) | amount | — | Pitch LFO depth. **OOC.** |
| Pitch LFO OSC 1 | toggle | 0–127 (raw MIDI CC 21) | enum | 0-63: Off; 64-127: On | Route Pitch LFO to Osc 1 frequency. **OOC.** |
| Pitch LFO OSC 2 | toggle | 0–127 (raw MIDI CC 22) | enum | 0-63: Off; 64-127: On | Route Pitch LFO to Osc 2 frequency. **OOC.** |
| Pitch LFO Mod OSC | toggle | 0–127 (raw MIDI CC 23) | enum | 0-63: Off; 64-127: On | Route Pitch LFO to Modulation Oscillator frequency. **OOC.** |
| Pitch LFO Detune | toggle | 0–127 (raw MIDI CC 24) | enum | 0-63: Off; 64-127: On | Route Pitch LFO to per-voice detune amount. **OOC.** |

### Voice (unison, glide, mono)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Glide Time | continuous | 0–127 (raw MIDI CC 5) | rate (knob taper) | — | Portamento / glide time. |
| Voice Detune | continuous | 0–127 (raw MIDI CC 92) | amount | — | Per-voice oscillator detuning (vintage-drift simulation). **Out-of-common-core (vintage-knob-style behavior).** |
| Voice Unison | toggle | 0–127 (raw MIDI CC 108) | enum | 0-63: Off; 64-127: On | Unison mode — stack voices on a single note. **OOC (voice-allocation mode).** |
| Voice Mono | toggle | 0–127 (raw MIDI CC 109) | enum | 0-63: Off; 64-127: On | Monophonic mode. **OOC.** |
| Pan | continuous | 0–127 (raw MIDI CC 10) | bipolar | — | Active-Timbre pan position (bipolar; 64 = center). |
| Pan Spread | continuous | 0–127 (raw MIDI CC 9) | amount | — | Stereo voice spread amount across the 8 voices. **OOC.** |

### Master / Global
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mod Wheel | continuous | 0–127 (raw MIDI CC 1) | level | — | Standard MIDI Mod Wheel — assignable as a mod-matrix source. |
| Mute | toggle | 0–127 (raw MIDI CC 3) | enum | 0-63: Off; 64-127: On | Clickless mute of the active Timbre. |
| Timbre Volume | continuous | 0–127 (raw MIDI CC 7) | level | — | Active-Timbre volume (the Muse uses CC 7 as per-Timbre level rather than global master volume per the published chart). |
| Low Cut | continuous | 0–127 (raw MIDI CC 8) | cutoff | — | Global low-cut filter (post-engine high-pass on the master output). |
| Expression | continuous | 0–127 (raw MIDI CC 11) | level | — | Standard MIDI Expression — assignable as a mod-matrix source. |
| Sustain Pedal | toggle | 0–127 (raw MIDI CC 64) | enum | 0-63: Off; 64-127: On | Standard sustain pedal — holds envelopes in Sustain. |
| Hold | toggle | 0–127 (raw MIDI CC 71) | enum | 0-63: Off; 64-127: On | Patch-saved hold / latch toggle. |
| Clock Tempo | continuous | 0–127 (raw MIDI CC 116) | tempo (knob taper) | — | Internal clock tempo — drives sequencer / arpeggiator / delay clock-sync / LFO sync. |

Delay-section parameters (`Delay Time Left` CC 93, `Delay Time Right` CC 94, `Link Delays` CC 95, `Delay Clock Sync` CC 102, `Delay Feedback` CC 103, `Delay Character` CC 104, `Delay Mix` CC 105, `Delay Timbre A` CC 106, `Delay Timbre B` CC 107) are CC-addressable but **post-engine and out-of-common-core (effects domain)** — recorded for completeness. Note `Delay Feedback` (CC 103) is the delay-line feedback gain, distinct from the mixer's `Clipping Level` (CC 65) — see Quirks.

Sequencer / arpeggiator parameters (`Sequencer Clock Div` CC 110, `Arpeggiator Clock Div` CC 111, `Arpeggiator On/Off` CC 112, `Arpeggiator FW/BK` CC 113, `Arpeggiator Direction` CC 114 with enum Order/Pattern/Random, `Arpeggiator Octave Range` CC 115 with enum 1/2/3/4 octaves) are CC-addressable but treated as performance features, not tone-generation parameters. **Out-of-common-core (sequencer / arp domain).**

## Modulation routing

The Muse has **two modulation LFOs** (`LFO 1`, `LFO 2`) plus a third dedicated **Pitch LFO** for per-voice vibrato, plus a **third audio-rate Modulation Oscillator** with its own fixed-routing modulation matrix, plus a **16-slot mod matrix per Timbre** — i.e., a true free-routing mod-matrix architecture beyond the fixed-destination per-LFO topology of the OB-X8 / Prophet-5 / Prophet-6. The multi-LFO and mod-matrix topology is itself out-of-common-core.

- **LFO 1 destinations:** Configured via the **16-slot mod matrix per Timbre** — the published CC chart does NOT expose per-destination on/off switches for LFO 1. LFO 1 is one of the 17 mod-matrix sources per Music Tech / Sound on Sound.
- **LFO 1 shapes:** `Triangle`, `Saw`, `Square`, `Random`, `User Wave`. Five total. **The `User Wave` slot is a user-defined custom shape (out-of-common-core).**
- **LFO 2 destinations:** Same as LFO 1 — configured via the mod matrix. **Out-of-common-core (second LFO source / mod-matrix routing).**
- **LFO 2 shapes:** Same 5-position enum as LFO 1.
- **Pitch LFO destinations (fixed):** `Pitch LFO OSC 1`, `Pitch LFO OSC 2`, `Pitch LFO Mod OSC`, `Pitch LFO Detune` — each independently switchable. The Pitch LFO is broadcast to any subset of the four destinations. **Out-of-common-core.**
- **Modulation Oscillator destinations (fixed, with per-destination amounts):** Pitch (Osc 1, Osc 2 — scaled by `Pitch Amount` plus per-osc amount), PWM (Osc 1, Osc 2 — scaled by `PWM Amount` plus per-osc amount), Filter (Filter 1, Filter 2 — scaled by `Filter Amount` plus per-filter amount), VCA / Pan (`VCA Amount`, `VCA PAN`). **Out-of-common-core.**
- **16-slot mod matrix per Timbre.** Per Music Tech and Sound on Sound: **17 sources, 17 controllers, 14 mathematical transforms, 50 destinations** — a free-routing matrix where any source can drive any destination with a per-slot amount. **All 16 slots × 2 Timbres = 32 slots are out-of-common-core.** As of the v1.4 manual, mod-matrix slot assignments are **not addressed via individual CCs**; they are configured via the panel UI or via sysex preset dump only.
- **Velocity → filter envelope** via `Filter Env Velocity` (CC 85).
- **Velocity → VCA envelope** via `VCA Env Velocity` (CC 91).
- **Filter envelope → Filter 1 cutoff** via `Filter 1 Envelope Amount` (CC 69), and **→ Filter 2 cutoff** via `Filter 2 Envelope Amount` (CC 75) — same envelope feeds both filters with independent amounts.

Additional out-of-common-core paths:
- **Hard sync (`Oscillator 2>1 SYNC`).** Osc 1 reset by Osc 2.
- **Bi-directional FM (`Oscillator 2>1 FM`, `Oscillator 1>2 FM`, scaled by `FM Amount`).** Either oscillator can be the FM source for the other.
- **Ring mod (`Ring Mod Level`).** Osc 1 × Osc 2 ring modulator.
- **CP3 mixer overload drive (`Clipping Level`).** Saturation as a patch parameter — see Quirks for cutoff interaction.

Sources for routing: Muse Manual v1.4.0 (mod matrix description), Sound on Sound review (filter / mixer / oscillator description), Music Tech review (mod-matrix slot count and source/destination/transform counts), midi.guide chart (CC-level routing parameters).

## Quirks

- **No NRPN parameter numbers are published for the Muse as of the v1.4 manual.** The midi.guide community chart lists 0 NRPNs for the device, and the manual's MIDI implementation appendix similarly publishes only CC assignments. This is a gap relative to the Sequential / Oberheim chart conventions used elsewhere in this survey set — for the Muse, the canonical control surface is **CC + sysex preset dump**, not CC + NRPN. Hosts that need to address mod-matrix slot assignments, the per-Timbre split / layer / dynamic-voice-allocation modes, the Layer A vs Layer B selector, the User Wave slot definition, the global LFOs and global envelopes added in firmware 1.4, or any other parameter not in the CC chart must use sysex preset dumps. **Recorded as a documentation gap for the Muse (the device may have a more complete MIDI surface than the published chart exposes — Moog has historically expanded MIDI implementations across firmware updates, e.g. the v1.4 firmware update which added new modulation sources without adding new CCs).**
- **Bi-timbral with 8 voices, split as 4+4 when layered.** Per Music Tech / Sound on Sound: every patch contains two Timbres (`Timbre A`, `Timbre B`) which can be split across the keyboard, layered (stacked), or switched between. When stacked, "a single key press will trigger two voices, allowing for four-note polyphony" — i.e. 4+4. When split, the voice allocation is **dynamic** rather than a fixed ratio (the manual describes "dynamic voice allocation" rather than a hard 6+2 / 4+4 split). **This survey covers single-Timbre Layer A only — bi-timbral / layered / split mode is out-of-common-core.** The published CC chart does not expose a Timbre-A/B selector or split-point parameter; these are sysex-only.
- **Two per-voice envelopes, not three.** Despite a working assumption upstream that the Muse has filter / amp / mod envelopes, the device has only **two per-voice ADSR envelopes** (Filter Env CCs 79–85, VCA Env CCs 86–91), both loopable. The Pitch LFO (CC 18–24) can be configured as an attack-decay envelope per Music Tech, but it is not a full ADSR. Firmware 1.4 added two **global** envelopes available as mod-matrix sources, but they are global (not per-voice) and not exposed as CCs in the published chart. **The "third envelope" common-core flag is not applicable to the Muse as published.**
- **CP3 mixer drive (`Clipping Level`) is a saturation path that interacts with filter cutoff perception.** Per Sound on Sound: the CP3-style mixer overloads into soft analog clipping when overdriven. Because the saturation is upstream of the filter, increasing `Clipping Level` adds harmonic content that the filter then sweeps — hosts perceive a different filter timbre at the same cutoff value depending on `Clipping Level`. **The mixer drive is a patch parameter (CC 65) but its perceptual interaction with cutoff is out-of-common-core — recorded as a coupling that the canonical schema does not capture.**
- **Two filters with three routing modes plus Link Filters.** `Filter 1` is a multi-mode filter offering both a high-pass cutoff (`Filter 1 High Pass`, CC 66) and a low-pass cutoff (`Filter 1 Cutoff`, CC 67). `Filter 2` is low-pass only. Both are Moog ladder 24 dB/oct designs (Music Tech: "904a-based"). The `Filters Order` parameter (CC 78) selects Serial / Stereo / Parallel routing; `Link Filters` (CC 77) ties parameters together. The schema's common-core collapses to a single LP filter; **the dual-filter topology, the high-pass-on-Filter-1 path, and the routing matrix are all out-of-common-core.**
- **Loopable envelopes.** Both the Filter Env and the VCA Env have a `Loop` toggle (CC 83 and CC 90 respectively) — when on, the envelope re-triggers from Attack after Release completes, turning the ADSR into a quasi-LFO. **Out-of-common-core (looping envelope as patch parameter).**
- **`Filter Env Delay` and `VCA Env Delay` chart row names are likely typos for "Decay".** The midi.guide chart publishes `Filter Env Delay` (CC 81) and `VCA Env Delay` (CC 88), positioned between Sustain and Release in CC order. The Muse Manual v1.4.0's envelope description publishes the standard ADSR (Attack / Decay / Sustain / Release) layout, not a DADSR (Delay-Attack-Decay-Sustain-Release). The CC parameter at position 81 is the **decay-time** stage, mis-labelled in the published chart. Recorded verbatim as `Delay` here for traceability with the chart, with this note clarifying the actual function.
- **Modulation Oscillator is a third audio-rate oscillator with its own fixed-routing modulation matrix.** Per Music Tech: "tracks the keyboard perfectly" (when `KB Track` is on) but can disconnect for drone use. With `Audio` on, it becomes a third VCO into the CP3 mixer at audio rate — the Muse can produce triple-oscillator patches. With `Audio` off, it is a per-voice modulation source with its own pitch / PWM / filter / VCA destination paths and per-destination amounts. **The combined audio + modulation role and the dedicated per-destination-amount routing are out-of-common-core.**
- **Pitch LFO is a third LFO with vibrato-only fixed routing.** Per Music Tech: "a pitch LFO which can operate as an attack decay envelope" — `Pitch LFO Shape` (CC 19) is described as morphing across LFO and AD-envelope behavior. **Out-of-common-core.**
- **16-slot mod matrix per Timbre (32 total slots) is not exposed as CCs.** Mod-matrix slot source / destination / amount / controller / transform fields are panel-UI / sysex-only. The published CC chart exposes only the underlying mod sources (LFO 1, LFO 2, Pitch LFO, Mod Osc) and global controllers (Mod Wheel, Expression), not the slot assignments themselves. **Out-of-common-core (the canonical schema's common-core has only fixed per-LFO destination switches, not a free assignable mod matrix); also a documentation gap for hosts that want to drive mod-matrix slots over MIDI in real-time.**
- **`User Wave` LFO shape slot.** Both LFO 1 and LFO 2 have a 5th waveform position (`User Wave`, 100–127 in CC 14 / CC 17) referencing a user-defined custom waveshape stored elsewhere in the patch. The waveshape data itself is not exposed as a CC — it is panel-UI / sysex-only. **Out-of-common-core (user-defined LFO shape).**
- **Voice Detune is a vintage-drift simulation parameter.** Adds randomized per-voice detuning across the 8 voices, mirroring the per-voice oscillator drift of the Voyager-era hardware. **Out-of-common-core (vintage-knob-style behavior).**
- **No on-board reverb engine; on-board delay is post-engine.** The Muse has a stereo delay (CCs 93–107) and chorus / diffusion (per firmware 1.4 release notes — Synth Magazine), all of which are post-engine and out of scope for this subtractive-engine survey. The mixer-section `Clipping Level` is the only saturation/drive parameter inside the engine path (pre-filter); everything else is post-VCA.
- **`Mute` (CC 3) is an in-band MIDI controller.** Sequential / Oberheim use CC 3 for parameters like Vintage / BPM elsewhere — on the Muse, CC 3 is a clickless mute toggle for the active Timbre. Recorded as a CC-naming-collision risk for hosts cross-targeting multiple devices.
- **Firmware 1.4 (May 2025 per Gearnews) added six new global modulation sources** — two global LFOs, two global envelopes, two global random trigger sources — accessible only via the mod matrix. **Not exposed as CCs in the v1.4 published chart.** Out-of-common-core.
