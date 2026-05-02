# Korg Minilogue XD (subtractive layer) — Parameter Survey

**Manufacturer:** Korg
**Year:** 2019 (Owner's Manual published 2/2019; MIDI Implementation Chart dated 2018.12.06, version 1.00)
**Engine type:** hybrid (analog VCO 1 + VCO 2 path with analog VCF/VCA, plus a digital MULTI ENGINE) — **only the analog subtractive path is in scope for this survey.**
**Polyphony:** 4 voices
**Sources:**
- Korg Inc., *minilogue xd / minilogue xd module Owner's Manual* v.E1a (Published 2/2019): https://cdn.korg.com/us/support/download/files/1362ee55daa0ec780da684b9ad9ad99b.pdf — Specifications (p.60), MIDI Implementation Chart (p.61), Basic parameters / VCO 1 / VCO 2 / MULTI ENGINE / MIXER / FILTER / AMP EG / EG / LFO / EFFECTS / Voice Mode (pp.15–24), Edit mode parameter list (p.31), PROGRAM EDIT mode — Button 7 (LFO), Button 9 (OTHER SETTINGS) (pp.34–35).
- Korg Minilogue XD Owner's Manual page 61 (MIDI Implementation Chart) on ManualsLib: https://www.manualslib.com/manual/1551428/Korg-Minilogue-Xd.html?page=61
- midi.guide (CC + NRPN reference, third-party): https://midi.guide/d/korg/minilogue-xd/

**Engine-scope caveat.** This survey covers **only the analog VCO 1 / VCO 2 subtractive signal path** (VCO 1, VCO 2, the analog mixer's VCO 1 and VCO 2 + noise contributions, the analog 2-pole low-pass VCF, the VCA, the two envelope generators, and the single LFO). The Minilogue XD's digital **MULTI ENGINE** (NOISE generator / VPM oscillator / USR user-loadable wavetable oscillator) is a third sound source that mixes into the analog VCF alongside the two VCOs, but it is a digital, non-subtractive engine (VPM = Variable Phase Modulation; user oscillators are arbitrary code via the logue SDK; the noise generator is a four-mode digital noise source — High, Low, Peak, Decim — not a simple analog noise tap). The MULTI ENGINE is therefore out of scope for the subtractive ontology and belongs in the wavetable/FM/noise backlog. CC numbers for the MULTI ENGINE are listed in a single "out-of-scope (MULTI ENGINE, not subtractive)" subsection at the end of the Parameters section for completeness but are excluded from the main subtractive tables.

## Signal path

Two analog VCOs (VCO 1, VCO 2) each with three selectable waveforms (Square / Triangle / Sawtooth) and a continuously variable SHAPE knob feed an analog mixer alongside the digital MULTI ENGINE. The mixer (VCO 1 / VCO 2 / MULTI level knobs) sums into a single 2-pole resonant analog low-pass VCF with cutoff, resonance, a DRIVE switch (0% / 50% / 100%), and a stepped KEYTRACK switch (0% / 50% / 100%). The VCF output drives the VCA. A dedicated 4-stage ADSR (`AMP EG`) drives the VCA. A second envelope, `EG`, is a 2-stage Attack/Decay envelope plus a bipolar `EG INT` intensity knob and a 3-way `TARGET` switch (PITCH / PITCH 2 / CUTOFF) — this single `EG` serves the dual purpose of "filter envelope" or "pitch envelope" depending on the TARGET setting (recorded under Quirks). A single LFO with three waveforms (Square / Triangle / Sawtooth), a 3-way MODE switch (1-SHOT / NORMAL / BPM), a RATE knob, an INT (intensity) knob with bipolar inversion, and a 3-way TARGET switch (CUTOFF / SHAPE / PITCH) is the only modulation source besides the envelopes. VCO hard sync (`SYNC` switch), ring modulation (`RING` switch), and cross modulation (`CROSS MOD DEPTH` knob) are panel-level features but are out of common-core (recorded under Quirks). The Voice Mode section selects one of four voice-allocation modes (POLY / UNISON / CHORD / ARP-LATCH) with a per-mode `VOICE MODE DEPTH` knob whose meaning varies per mode. Effects (chorus / ensemble / phaser / flanger / user MOD effects, plus delay and reverb) are post-engine and out of scope for this survey.

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit MIDI Continuous Controllers per the official MIDI Implementation Chart (Korg minilogue xd, Date 2018.12.06, Version 1.00). The Owner's Manual documents wider underlying program-parameter ranges than the 7-bit CC pathway exposes (e.g. SHAPE 0–1023, AMP EG/EG ADSR 0–1023, LFO RATE 0–1023, LFO INT 0–511 with negative half reachable via the SHIFT-button trick, PITCH ±1200 cents, VOICE MODE DEPTH per-mode ranges of 0–1023 / 0C–50C / 17 chord-type enum / 16 arpeggiator-pattern enum). The CC pathway is quantized to 128 steps with a hardware-defined non-linear taper (knob taper). Discrete switches addressed via CC use the Korg label-table convention: a switch with N labels uses CC values that quantize to one of N positions across 0–127 (e.g. 3-way switches like LFO MODE typically use ranges around 0–42, 43–84, 85–127; the manual does not specify exact thresholds, so a host should treat any CC ≥ 64 as "next position up" for 2-way switches and the reader should consult the device for 3-way switch CC quantization).

### VCO 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCO1 WAVE | discrete | 0–127 (raw MIDI CC, CC 50) | enum | SQR / TRI / SAW | VCO 1 waveform select. Three-way switch on the panel. |
| VCO1 OCTAVE | discrete | 0–127 (raw MIDI CC, CC 48) | enum | 16' / 8' / 4' / 2' | VCO 1 octave (foot-mark style). 4-way switch. |
| VCO1 PITCH | continuous | 0–127 (raw MIDI CC, CC 34) | cents (knob taper, underlying −1200..+1200 cents in 1-cent steps) | — | VCO 1 fine pitch / tuning. Bipolar on the panel knob; the CC value collapses bipolar onto 0–127 with 64 = center. |
| VCO1 SHAPE | continuous | 0–127 (raw MIDI CC, CC 36) | morph (knob taper, underlying 0–1023) | — | Continuously-variable waveshape / complexity / duty-cycle of the selected VCO 1 waveform. |

### VCO 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCO2 WAVE | discrete | 0–127 (raw MIDI CC, CC 51) | enum | SQR / TRI / SAW | VCO 2 waveform select. |
| VCO2 OCTAVE | discrete | 0–127 (raw MIDI CC, CC 49) | enum | 16' / 8' / 4' / 2' | VCO 2 octave. |
| VCO2 PITCH | continuous | 0–127 (raw MIDI CC, CC 35) | cents (knob taper, underlying −1200..+1200 cents) | — | VCO 2 fine pitch / tuning. Bipolar; 64 = center. |
| VCO2 SHAPE | continuous | 0–127 (raw MIDI CC, CC 37) | morph (knob taper, underlying 0–1023) | — | VCO 2 waveshape. |

Out-of-common-core (panel switches in the VCO 2 section — these create cross-oscillator interactions; recorded under Quirks):
- `SYNC` switch (CC 80, Off/On) — hard-syncs VCO 2 to VCO 1 (manual: "the phase of oscillator 2 is forcibly synchronized to the phase of oscillator 1"). Out-of-common-core (cross-oscillator feature).
- `RING` switch (CC 81, Off/On) — ring modulation between VCO 1 and VCO 2 (manual: "Oscillator 1 is used to ring modulate oscillator 2"). Out-of-common-core (cross-oscillator feature).
- `CROSS MOD DEPTH` knob (CC 41, 0–1023 underlying) — VCO 1 modulates VCO 2 pitch (audio-rate FM-style cross-modulation). Out-of-common-core.

### Mixer (VCO 1 + VCO 2 — MULTI ENGINE excluded)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| MIXER VCO 1 | continuous | 0–127 (raw MIDI CC, CC 39) | level (knob taper, underlying 0–1023) | — | VCO 1 output level into the analog VCF. |
| MIXER VCO 2 | continuous | 0–127 (raw MIDI CC, CC 40) | level (knob taper, underlying 0–1023) | — | VCO 2 output level into the analog VCF. |

The Mixer also has a `MULTI` knob (CC 33, 0–1023 underlying) feeding the digital MULTI ENGINE source into the same VCF. **Out of scope** — see "MULTI ENGINE (out-of-scope, not subtractive)" subsection below. The Minilogue XD has **no dedicated analog noise oscillator**; the panel "noise" is provided by the MULTI ENGINE in NOISE mode (with four digital noise sub-types: High/Low/Peak/Decim). When the canonical schema's "noise level" parameter is required, the closest device analogue is the MULTI level knob with NOISE selected — but this is digital noise, not an analog white-noise tap, and is recorded as out-of-scope MULTI ENGINE behavior rather than mixed into the subtractive Mixer table.

### Filter (analog 2-pole low-pass VCF)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| CUTOFF | continuous | 0–127 (raw MIDI CC, CC 43) | cutoff (knob taper, underlying 0–1023) | — | VCF cutoff frequency. Manual: "Turning the knob to the left will lower the cutoff frequency, and turning the knob to the right will raise the cutoff frequency." |
| RESONANCE | continuous | 0–127 (raw MIDI CC, CC 44) | resonance (knob taper, underlying 0–1023) | — | VCF resonance / Q. |
| DRIVE | discrete | 0–127 (raw MIDI CC, CC 84) | enum | 0% / 50% / 100% | 3-step pre-filter drive (analog distortion in the filter input). |
| KEYTRACK | discrete | 0–127 (raw MIDI CC, CC 83) | enum | 0% / 50% / 100% | 3-step keyboard tracking amount into VCF cutoff. 100% = one-octave-per-octave, 50% = half-rate, 0% = no tracking. |

### Amp envelope (`AMP EG`)
4-stage ADSR. Drives the VCA only.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| AMP EG ATTACK | continuous | 0–127 (raw MIDI CC, CC 16) | time (knob taper, underlying 0–1023) | — | Time for the AMP EG to reach its maximum level after note-on. |
| AMP EG DECAY | continuous | 0–127 (raw MIDI CC, CC 17) | time (knob taper, underlying 0–1023) | — | Time for the AMP EG to fall from peak to the sustain level. |
| AMP EG SUSTAIN | continuous | 0–127 (raw MIDI CC, CC 18) | level (knob taper, underlying 0–1023) | — | Level held while the key is down after the decay phase. |
| AMP EG RELEASE | continuous | 0–127 (raw MIDI CC, CC 19) | time (knob taper, underlying 0–1023) | — | Time for the AMP EG to reach zero after note-off. |

### `EG` (single shared filter / pitch envelope)
The Minilogue XD has **one** `EG` (besides the AMP EG) that is shared between two roles: it can drive VCF cutoff (acting as a "filter envelope") or it can drive oscillator pitch (acting as a "pitch envelope"). The `TARGET` switch chooses the destination. The envelope itself has only **Attack and Decay** stages (not full ADSR) plus a bipolar intensity knob — this is a 2-stage AD envelope, not a 4-stage ADSR. This is a quirky shared-envelope design and is recorded under Quirks.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| EG ATTACK | continuous | 0–127 (raw MIDI CC, CC 20) | time (knob taper, underlying 0–1023) | — | EG attack time. |
| EG DECAY | continuous | 0–127 (raw MIDI CC, CC 21) | time (knob taper, underlying 0–1023) | — | EG decay time. |
| EG INT | continuous | 0–127 (raw MIDI CC, CC 22) | bipolar amount (knob taper, underlying −100%..+100%) | — | EG intensity. Bipolar on the panel knob (manual: "When set to a negative value, the EG will be applied in the negative direction"); the CC value collapses bipolar onto 0–127 with 64 = center. Out-of-common-core: signed amount knob behavior — see Quirks. |
| EG TARGET | discrete | 0–127 (raw MIDI CC, CC 23) | enum | PITCH / PITCH 2 / CUTOFF | EG destination. PITCH = both VCOs and MULTI ENGINE pitch; PITCH 2 = VCO 2 only; CUTOFF = VCF cutoff. Out-of-common-core (the canonical schema typically fixes "filter envelope" and "amp envelope" as two independent slots — the Minilogue XD's `EG` is a single envelope time-shared across pitch and filter via this enum). |

### LFO (single LFO)
The Minilogue XD has **one** LFO. Manual (p.23): "Depending on its target, the LFO can provide vibrato (PITCH); tonal changes to the Oscillators (SHAPE); or wah-wah (CUTOFF) effects." The destination set is fixed (Cutoff, Pitch, Shape) — there is no destination-selector to other targets. The LFO has only one destination active at a time (single-destination enum, not per-destination on/off switches).

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO WAVE | discrete | 0–127 (raw MIDI CC, CC 57) | enum | SQR / TRI / SAW | LFO waveform. Manual: "The LFO can be set to a square wave (SQR), a triangle wave (TRI), or a sawtooth wave (SAW)." |
| LFO MODE | discrete | 0–127 (raw MIDI CC, CC 58) | enum | 1-SHOT / NORMAL / BPM | LFO operating mode. 1-SHOT: LFO stops after a half-cycle from the time the sound is played (range 0.05 Hz–28 Hz). NORMAL: free-running LFO (range 0.05 Hz–28 Hz). BPM: LFO RATE syncs to the program's BPM and the RATE knob selects note-divisions instead of Hz. |
| LFO RATE | continuous | 0–127 (raw MIDI CC, CC 24) | rate (knob taper, underlying 0–1023 in NORMAL/1-SHOT, or note-division enum 4 / 2 / 1 / 0 / 3/4 ... 1/64 in BPM mode) | — | LFO frequency. |
| LFO INT | continuous | 0–127 (raw MIDI CC, CC 26) | depth (knob taper, underlying 0–511; bipolar reachable via SHIFT-button-held inversion to 0..−511) | — | LFO intensity / depth. Bipolar via SHIFT trick — out-of-common-core (signed amount knob behavior). |
| LFO TARGET | discrete | 0–127 (raw MIDI CC, CC 56) | enum | CUTOFF / SHAPE / PITCH | LFO destination (single-destination, not per-destination switches). PITCH applies modulation to the oscillator(s) selected by `LFO Target OSC` in PROGRAM EDIT (All / VCO1+2 / VCO2 / Multi); SHAPE applies modulation to the SHAPE knob setting of the same oscillator selection; CUTOFF applies modulation to the VCF cutoff. |

The Minilogue XD's LFO destination scoping for PITCH and SHAPE (i.e. which oscillator the LFO actually modulates when TARGET = PITCH or SHAPE) is set at PROGRAM EDIT level via `LFO Target OSC` (Button 7). Enum: `All` / `VCO1+2` / `VCO2` / `Multi`. This is reachable only via the Edit mode and is not addressable as a top-level CC — the LFO's TARGET CC selects PITCH / SHAPE / CUTOFF only, not which oscillator. Recorded under Modulation routing.

### Voice (POLY / MONO¹ / UNISON / CHORD / ARP-LATCH)
The Minilogue XD's `VOICE MODE TYPE` switch is a 4-way enum: **POLY / UNISON / CHORD / ARP-LATCH** (not a 5-way as some sources list — the official Owner's Manual p.15 lists exactly four entries). The companion `VOICE MODE DEPTH` knob has a per-mode meaning — its semantic depends on which mode is selected. **Common-core covers only the POLY / UNISON subset** (and the Minilogue XD has no dedicated MONO mode — UNISON with 4 voices stacked at unison is the closest analogue). CHORD and ARP-LATCH are out of common-core (chord-stacker / arpeggiator are performance features, not voice-allocation in the strict sense).

¹ Note on "MONO": the Minilogue XD does not expose a strict MONO voice mode. The original (non-XD) minilogue had `MONO` as one of five voice modes; on the XD it was removed. The closest equivalents are UNISON with depth 0 (effectively a 4-voice stacked mono with no detune) or POLY with the keyboard playing one note at a time. This is recorded under Quirks.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VOICE MODE TYPE | discrete | 0–127 (raw MIDI CC, CC 52) | enum | POLY / UNISON / CHORD / ARP/LATCH | Voice-allocation mode (per Owner's Manual p.15 voice mode list). POLY and UNISON are common-core; CHORD and ARP/LATCH are out-of-common-core. |
| VOICE MODE DEPTH | continuous/discrete | 0–127 (raw MIDI CC, CC 27) | per-mode (see below) | — | Per-mode parameter. **POLY:** 0–1023 underlying — turn right to switch to DUO mode (stacks two voices, increases stacked-voice level and detune depth). **UNISON:** 0C–50C — detune amount in cents. **CHORD:** 17 chord-type enum — Mono / 5th / sus2 / m / Maj / sus4 / m7 / 7 / 7sus4 / Maj7 / aug / dim / m7♭5 / mMaj7 / Maj7♭5 / (and additional types). **ARP/LATCH:** 16 arpeggiator-pattern enum — MANUAL 1 / MANUAL 2 / RISE 1 / RISE 2 / FALL 1 / FALL 2 / RISE FALL 1 / RISE FALL 2 / POLY 1 / POLY 2 / RANDOM 1 / RANDOM 2 / RANDOM 3. The CC value collapses each per-mode underlying range onto 0–127. |

### Master / performance
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Bank Select MSB | continuous | 0–127 (raw MIDI CC, CC 0) | — | — | Standard MIDI bank-select MSB (per the MIDI Implementation Chart). |
| Bank Select LSB | continuous | 0–127 (raw MIDI CC, CC 32) | — | — | Standard MIDI bank-select LSB. |
| Mod Wheel | continuous | 0–127 (raw MIDI CC, CC 1) | level | — | Standard performance mod wheel. Per the manual (p.42), CC 1 cannot be disabled by the `MIDI Rx CC` global setting — it is always received. |
| Portamento Time | continuous | 0–127 (raw MIDI CC, CC 5) | time (knob taper, underlying 0–127) | — | Portamento (glide) time. The PORTAMENTO knob on the panel maps directly. |
| Damper Pedal | toggle | 0–127 (raw MIDI CC, CC 64) | enum | Off / On | Standard damper / sustain pedal. Per the manual, CC 64 cannot be disabled by `MIDI Rx CC` — always received. |
| All Sound Off | message | 0 (raw MIDI CC, CC 120) | — | — | Standard MIDI All Sound Off message. |
| Reset All Controllers | message | 0 (raw MIDI CC, CC 121) | — | — | Standard MIDI Reset All Controllers message. |

Out-of-common-core (Tempo / sequencer / global): TEMPO knob (BPM 56.0..240.0 in basic parameters; 10.0..300.0 in SEQ EDIT) is a sequencer / arpeggiator clock parameter, not a tone-generation parameter. Master Tune (±50 cents) and Transpose (±12 semitones) are GLOBAL EDIT settings. Recorded for completeness.

### MULTI ENGINE (out-of-scope — not subtractive)
The MULTI ENGINE is a digital sound source that mixes into the analog VCF at the Mixer stage. It runs in one of three modes selected by the `NOISE/VPM/USR` switch:
- **NOISE** mode: 4 digital noise sub-types (High = HPF noise, Low = LPF noise, Peak = bandpass noise, Decim = decimator). Not analog white noise.
- **VPM** mode: Variable Phase Modulation oscillator with 16 types (Sin1, Sin2, Sin3, Sin4, Saw1, Saw2, Squ1, Squ2, Fat1, Fat2, Air1, Air2, Decay1, Decay2, Creep, Throat) and 6 Edit-mode parameters (Feedback, Noise Depth, Shape Mod Int, Mod Attack, Mod Decay, Mod Key Track).
- **USR** mode: user-loadable oscillator programs via the logue SDK (up to 16 user oscillators). Arbitrary code; no fixed parameter set.

CCs (per MIDI Implementation Chart, listed for completeness, **excluded from the subtractive ontology**):
- `MULTI TYPE`: CC 53 (selects Noise sub-type, VPM type, or USR oscillator)
- `MULTI SUB TYPE`: CC 103
- `MULTI SHAPE`: CC 54
- `MULTI SHIFT SHAPE`: CC 104
- `MIXER MULTI`: CC 33 (MULTI level into VCF)

These belong in the wavetable/FM/noise backlog, not in the subtractive ontology.

### Effects (out of scope — post-engine)
The MOD / DELAY / REVERB effects bus is post-engine. CCs (per MIDI Implementation Chart, listed for completeness, out of scope):
- `MOD FX TYPE` CC 88, `MOD FX SUB TYPE` CC 96, `MOD FX TIME` CC 28, `MOD FX DEPTH` CC 29, `MOD FX ON-OFF` CC 92.
- `DELAY TYPE` CC 89, `DELAY TIME` CC 105, `DELAY DEPTH` CC 112, `DELAY DRY-WET` CC 107, `DELAY ON-OFF` CC 94.
- `REVERB TYPE` CC 90, `REVERB TIME` CC 114, `REVERB DEPTH` CC 115, `REVERB DRY-WET` CC 110, `REVERB ON-OFF` CC 95.

## Modulation routing

The Minilogue XD has a **fixed routing topology** with no general-purpose mod matrix and a deliberately small modulation-source set (one shared `EG`, one LFO, plus performance-time velocity / mod-wheel / pitch-bend / damper). All routings are reproduced verbatim from the Owner's Manual:

- **LFO destinations (single-destination enum, NOT per-destination switches — verbatim from manual p.23 LFO TARGET switch):** `CUTOFF` (LFO → VCF cutoff — wah-wah), `SHAPE` (LFO → SHAPE knob of the oscillator selected by `LFO Target OSC` in PROGRAM EDIT), `PITCH` (LFO → PITCH knob of the oscillator selected by `LFO Target OSC`). The LFO has exactly **three destinations** and only **one destination is active at a time** (selected by CC 56 `LFO TARGET`). This is more restrictive than most subtractive synths in this survey, which expose simultaneous-multi-destination LFO routing or a destination matrix.
- **LFO oscillator scoping (`LFO Target OSC`, PROGRAM EDIT Button 7):** `All` / `VCO1+2` / `VCO2` / `Multi`. Restricts which oscillator(s) the LFO modulates when TARGET = PITCH or SHAPE. SysEx-only / Edit-mode-only — not addressable via top-level CC.
- **LFO waveforms (`LFO WAVE`):** `SQR`, `TRI`, `SAW` (manual p.23).
- **LFO modes (`LFO MODE`):** `1-SHOT` (one half-cycle then stop), `NORMAL` (free-running), `BPM` (synced to program BPM, RATE selects note divisions 4 / 2 / 1 / 0 / 3/4 ... 1/64).
- **LFO Key Sync / Voice Sync (`LFO Key Sync`, `LFO Voice Sync`, PROGRAM EDIT Button 7, Off/On):** controls whether the LFO phase resets on note-on (Key Sync) and whether LFO phase is synchronized between the four voices (Voice Sync). SysEx-only / Edit-mode-only.

- **`EG` destinations (single-destination enum, `EG TARGET`):** `PITCH` (both VCOs and MULTI ENGINE pitch), `PITCH 2` (VCO 2 only), `CUTOFF` (VCF cutoff). Bipolar `EG INT` scales depth signed.

- **AMP EG → VCA:** always-on, hardwired. No depth knob (the AMP EG drives the VCA at 100%).

- **Velocity routings (PROGRAM EDIT Button 8 — MODULATION):**
  - `EG Velocity` (0..127): scales `EG INT` by note velocity. Velocity → cutoff (or pitch) via the `EG`.
  - `Amp Velocity` (0..127): scales VCA level by note velocity. 0 = velocity has no effect; higher = more sensitivity.

- **Aftertouch:** the MIDI Implementation Chart shows `After Touch — Key's: X / Channel: X` (both transmitted and recognized = X = "No"). The Minilogue XD **does not respond to aftertouch** — recorded under Quirks.

- **Joystick / mod-wheel routings (PROGRAM EDIT Button 4 — JOYSTICK):**
  - `X+ Bend Range`, `X− Bend Range`: pitch-bend ranges for joystick right / left (Off / 1 Note ... 12 Note).
  - `Y+ Assign`, `Y− Assign`: parameter assigned to joystick up / down. The destination list is fixed and includes: GATE TIME, PORTAMENTO, VOICE MODE DEPTH, VCO1 PITCH, VCO1 SHAPE, VCO2 PITCH, VCO2 SHAPE, CROSS MOD, MULTI SHAPE, VCO1 LEVEL, VCO2 LEVEL, MULTI LEVEL, CUTOFF, RESONANCE, A.EG ATTACK, A.EG DECAY, A.EG SUSTAIN, A.EG RELEASE, EG ATTACK, EG DECAY, EG INT, LFO RATE, LFO INT, MOD TIME, MOD DEPTH, REVERB TIME, REVERB DEPTH, DELAY TIME, DELAY DEPTH (verbatim list from manual p.32). `Y+ Range` / `Y− Range`: −100% .. +100% scaling.
  - This is the closest thing to a "mod matrix" the Minilogue XD has — but it is per-program, joystick-source-only, two-slot (Y+ and Y−), and the destinations are a fixed enum. Out-of-common-core (joystick / performance-mod-source-specific).

- **CV IN routings (PROGRAM EDIT Button 5 — CV INPUT):** two CV input jacks can drive any of the same destinations as the joystick Y axis (`CV IN Mode = Modulation` mode). In `CV/Gate` mode the jacks instead receive standard 1V/oct + 0–5V gate. Out-of-common-core (CV-modular routing, performance feature).

- **Pitch-bend fixed routings (CC: pitch bend):** standard MIDI pitch bend with range set by `X+ Bend Range` / `X− Bend Range` (0..12 notes per side).

- **Mod matrix slots:** **none.** The Minilogue XD has no general-purpose mod matrix at the patch level — fixed routings only (LFO → 3 destinations one-at-a-time, EG → 3 destinations one-at-a-time, AMP EG → VCA, velocity → cutoff/amp, joystick Y → 2 fixed slots).

Sources for routing: Owner's Manual pp.22–23 (AMP EG, EG, LFO sections), pp.32–35 (PROGRAM EDIT JOYSTICK / CV INPUT / LFO / MODULATION buttons), p.61 (MIDI Implementation Chart for CC numbers), p.23 LFO TARGET switch (verbatim destination labels).

## Quirks

- **Hybrid engine; this survey is one slice.** The Minilogue XD has an analog VCO 1 + VCO 2 + VCF + VCA path (covered) and a digital MULTI ENGINE (excluded). The MULTI ENGINE mixes into the same analog VCF as VCO 1 / VCO 2, so a real Minilogue XD patch can be subtractive-with-digital-source — but the digital source itself is not subtractive (VPM / wavetable / decimator / arbitrary user code). Belongs in wavetable/FM/noise backlog.
- **No dedicated analog noise generator.** Unlike the Prophet-6 (NRPN-only `Noise Level`), the Moog Muse (analog noise tap), and most other subtractive synths in this survey, the Minilogue XD has **no analog white-noise oscillator**. The "noise" the panel implies is the digital MULTI ENGINE's NOISE mode — and that mode has four sub-types (High = HPF noise, Low = LPF noise, Peak = bandpass noise, Decim = decimator), none of which is plain white noise. The canonical schema's "noise level" parameter does not have a clean Minilogue XD analogue.
- **Single shared `EG` for filter and pitch.** The Minilogue XD has only **two** envelopes: AMP EG (full ADSR, hardwired to VCA) and `EG` (Attack/Decay only, plus a 3-way TARGET switch — PITCH / PITCH 2 / CUTOFF). The "filter envelope" and "pitch envelope" are time-shared on the same physical envelope: you cannot run filter envelope and pitch envelope simultaneously. This is unusual relative to other subtractive devices in this survey (Prophet-6 has separate Filter Envelope and VCA Envelope; JUNO-X has a single shared ADSR + an independent pitch envelope; Moog Muse has filter + amp + a third assignable; Minilogue XD has just two and the second one time-shares roles via TARGET). Out-of-common-core if the canonical schema fixes "filter envelope" + "amp envelope" as two independent slots.
- **`EG` is AD, not ADSR.** The shared `EG` has only Attack and Decay knobs (no Sustain, no Release). When acting as a filter envelope, this means the cutoff sweep follows AD only and rests at zero after decay — there is no held filter modulation while the key is down. Out-of-common-core if the canonical "filter envelope" assumes ADSR.
- **Filter Type switch (2-pole / 4-pole) is OOC — and absent.** The Minilogue XD has only the 2-pole VCF. Some Korg analog synths (notably the prologue) expose a 2-pole/4-pole switch; the Minilogue XD does not. There is no filter-type quirk to record beyond "the VCF is fixed 2-pole." If the canonical schema expects a filter-type enum, the Minilogue XD entry is "2-pole only."
- **VOICE MODE TYPE list is short on Minilogue XD (4 entries, not the original Minilogue's 5).** The Minilogue XD voice modes per the Owner's Manual p.15: **POLY / UNISON / CHORD / ARP-LATCH**. The original (non-XD) Minilogue had a fifth `MONO` mode — that mode was removed on the XD. **Common-core covers only POLY and UNISON.** CHORD (selects one of 17 chord types via VOICE MODE DEPTH) and ARP-LATCH (selects one of 16 arpeggiator patterns via VOICE MODE DEPTH) are out-of-common-core (chord stacker / arpeggiator are performance features, not voice-allocation primitives).
- **`VOICE MODE DEPTH` semantic varies per mode.** The same physical knob (CC 27) means: detune cents (UNISON), DUO-mode crossfade (POLY), chord-type selector enum (CHORD), or arpeggiator-pattern selector enum (ARP-LATCH). This polymorphic per-mode behavior makes the canonical schema hard to map cleanly — the parameter's interpretation requires reading `VOICE MODE TYPE` first.
- **No MONO voice mode.** The closest equivalent is UNISON with VOICE MODE DEPTH = 0 (a 4-voice stacked mono with no detune). The canonical schema's "MONO" mode does not have a direct Minilogue XD analogue.
- **No aftertouch.** The MIDI Implementation Chart explicitly shows `After Touch — Key's: X (No) / Channel: X (No)` for both transmitted and recognized. The Minilogue XD has no aftertouch on the keyboard and ignores incoming MIDI aftertouch. Out-of-common-core if the canonical schema includes aftertouch routings.
- **LFO is single-destination, single-source.** Unlike most other subtractive synths in this survey (Prophet-6 has per-destination on/off switches enabling simultaneous multi-destination LFO; JUNO-X has three independent depth knobs for three destinations; Moog Muse has multiple LFOs), the Minilogue XD's LFO routes to **exactly one** of CUTOFF / SHAPE / PITCH at a time, selected by `LFO TARGET` enum. The canonical schema's "LFO destination" parameter maps directly but with the constraint that no multi-destination is possible.
- **LFO INT is bipolar via SHIFT trick.** The LFO INT knob's underlying range is 0..511; pressing SHIFT while turning extends to 0..−511 (waveform inversion). The CC value collapses both halves onto 0–127. Out-of-common-core: signed amount knob behavior with hidden negative half.
- **EG INT is bipolar.** The EG INT knob is documented as −100% .. 0% .. +100% on the panel (manual p.22). The CC value collapses bipolar onto 0–127 with 64 = center. Out-of-common-core: signed amount knob behavior.
- **`Mod Routing` (PROGRAM EDIT Button 9) selects whether MULTI ENGINE goes through the VCF or bypasses it.** Pre VCF (MULTI is filtered) or Post VCF (MULTI bypasses VCF). When set to Post VCF, the digital MULTI source mixes directly into the post-filter signal — meaning the canonical "filter cutoff" parameter has no effect on the MULTI ENGINE path. Out-of-scope (MULTI ENGINE-related) but worth recording because it changes the topology.
- **CC values are raw 7-bit (0–127) with knob-taper non-linearity.** Underlying program-parameter ranges are wider (0–1023 for SHAPE / CUTOFF / RESONANCE / EG times / LFO RATE / mixer levels; 0–511 for LFO INT; ±1200 cents for VCO PITCH; ±50 cents for Master Tune). Acoustic-unit interpretation is not exposed by the device — you only see knob position via CC. SysEx (program dump) reaches the full underlying resolution.
- **Mod sequencer (motion sequence) is OOC.** The Minilogue XD's 16-step polyphonic sequencer can record a "motion sequence" — automated changes of knobs / switches across steps. Up to four motion sequences per program. Almost every panel parameter can be motion-sequenced (with a few exceptions: MASTER knob, TEMPO knob, OCTAVE switch, MULTI ENGINE TYPE-USR, DRIVE switch, DEL/REV/MOD switch, OFF/ON/SELECT-SELECT setting). Out-of-common-core (sequencer / motion-sequence is a performance feature, not a tone-generation parameter — same out-of-scope status as the Prophet-6 arpeggiator and JUNO-X scene sequencer).
- **CC vs published chart agreement.** Every CC number in this survey was cross-checked against the official Owner's Manual p.61 MIDI Implementation Chart (Korg minilogue xd, Date 2018.12.06, Version 1.00) — verified row-by-row: VCO1 50/48/34/36; VCO2 51/49/35/37; MULTI 53/103/54/104; Mixer 39/40/33; Filter 43/44/84/83; AMP EG 16/17/18/19; EG 20/21/22/23; LFO 57/58/24/26/56; SYNC/RING/CROSS MOD 80/81/41; VOICE MODE DEPTH/TYPE 27/52; Mod FX 88/96/28/29/92; Delay 89/105/112/107/94; Reverb 90/114/115/110/95; Damper 64; Bank Select 0/32; Portamento 5; Mod Wheel 1; All Sound Off / Reset All Controllers 120/121. No discrepancies found between the manual chart and the survey.
- **Effects are post-engine and out of scope.** The MOD / DELAY / REVERB bus runs after the VCA. Chorus / Ensemble / Phaser / Flanger / User MOD effects are MOD-bus options. CC numbers recorded above for completeness only; not part of the subtractive ontology.
- **No per-voice modulation matrix; no per-voice independent modulation.** All four voices of the Minilogue XD share a single modulation topology (one LFO with `LFO Voice Sync` controlling whether voices share LFO phase; one EG per voice but routed identically; AMP EG per voice with shared knob settings). There is no per-voice modulation differentiation in the patch — the only per-voice variation is `LFO Voice Sync = Off` (each voice's LFO phase free-runs independently) and the natural analog drift of the four VCO pairs.
