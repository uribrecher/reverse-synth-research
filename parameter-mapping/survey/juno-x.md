# JUNO-X (Analog Synth engine layer) — Parameter Survey

**Manufacturer:** Roland
**Year:** 2022
**Engine type:** virtual analog
**Polyphony:** up to 8 voices per scene part (Roland do not publish a single hard polyphony figure for the JUNO-X — it is engine- and scene-allocation-dependent; the AnalogSynth engine layer is documented as 8 voices per part within a scene)
**Sources:**
- Roland, *JUNO-X Parameter Guide* (2022): https://files.kraftmusic.com/media/ownersmanual/Roland_Juno-X_Parameter_Guide.pdf — Tone Parameters: TONE JUNO-X (pp. 18–19), TONE JUNO-106 (pp. 19–20), TONE JUNO-60 (p. 20)
- Roland, *JUNO-X MIDI Implementation* v1.10 (Apr 26 2022): https://files.kraftmusic.com/media/ownersmanual/Roland_Juno-X_MIDI_Implementation.pdf — SysEx address map "Temporary Tone Analog Synth Model" (offsets `02 10 00 00`–`02 13 00 00`), `[MDLSYN0] Analog Synth Model Parameter` block
- Roland, *JUNO-X Reference Manual* (2022): https://www.fullcompass.com/common/files/73250-JUNOXReferenceManual.pdf
- Sound on Sound, *Roland Juno-X review* (Oct 2022): https://www.soundonsound.com/reviews/roland-juno-x — engine inventory and JUNO-X-vs-JUNO-60/106 differences
- Cross-check: keyboards-mcp JUNO-X analog-synth params (`../../../keyboards-mcp/src/keyboard_models/roland/juno_x/engines/analog-synth.ts`)

The JUNO-X is a multi-engine workstation that ships four sound engines per scene-part: **AnalogSynth** (Roland's internal "Analog Synth Model" — the only subtractive-style engine, presenting three sub-tone variants on the panel: JUNO-60, JUNO-106, and JUNO-X), **ZCore** (a sample/PCM-plus-VA partial-based ZEN-Core engine that supports OSC sync, ring modulation, and cross-modulation between partials), **JunoXModel** (the modeled tone variant of the AnalogSynth — exposed as a separate engine slot in the `keyboards-mcp` mapping), and **RDPiano** (a piano model). **This survey covers only the AnalogSynth engine.** ZCore, JunoXModel, and RDPiano are out of scope for the subtractive ontology: ZCore's primary architecture is sample-plus-partial-routing rather than a single OSC→filter→amp path, RDPiano is a physical-model piano, and JunoXModel as a separate engine slot is a piano/key-model preset variant rather than a parametric subtractive engine.

A note on `keyboards-mcp` scope: the `analog-synth.ts` file exposes a single union of CC parameters whose CC numbers match the **TONE JUNO-X** variant column of the official Parameter Guide (CC 122 for FLT KEY FOLLOW, CC 25 for SUB OSC PSHIFT, CC 46 for SUPER SAW — all JUNO-X-specific). The JUNO-106 and JUNO-60 sub-tone variants have a near-identical parameter set with some CC reassignments (notably FLT KEY FOLLOW = CC 82) and a reduced switch set (PW SWITCH / SAW SWITCH instead of continuous level mixers, OSC RANGE 16'/8'/4' instead of bipolar OSC PITCH); those are recorded under Quirks for completeness but the survey's primary tables follow the keyboards-mcp / JUNO-X-variant CC assignments.

## Signal path

Each AnalogSynth voice has **one oscillator section** with selectable mixed sources: a pulse / Super-SAW source (`PW LEVEL / SSAW LEVEL`), a sawtooth source (`SAW LEVEL`), a square sub-oscillator one octave below (`SUB LEVEL`), and a white-noise source (`NOISE LEVEL`). When `SUPER SAW` is ON the pulse channel becomes a Super-SAW (multiple detuned sawtooth oscillators) and `PW LEVEL` doubles as Super-SAW level / detune-depth target. The four mixed sources feed a 4-step **HPF** (`HPF-STEP`, 0–3) → a **vintage-modeled LPF** with three response curves (`VINTAGE FLT TYPE` R / M / S) → a **VCA**. A single multi-shape **LFO** (sine, saw-down, square, sample-and-hold) is routable to OSC pitch (`OSC LFO MOD`), filter cutoff (`FILTER MOD`, addressed in `analog-synth.ts` as `as_flt_lfo_mod` CC 28), and amp level (`AMP MOD`, addressed as `as_amp_lfo_mod` CC 30). A single 4-stage ADSR drives the VCA and (with a separate signed `FLT ENV DEPTH`) the LPF cutoff. A **separate 4-stage pitch envelope** (`PENV ATTACK/DECAY/SUSTAIN/RELEASE`) modulates oscillator pitch — this is unusual for a strict 2-osc subtractive box and is recorded under Quirks. Portamento and key/voice-mode (POLY / SOLO / UNISON / SL-UNISON) sit between note input and voice allocation. Effects (chorus, delay, reverb, drive, MFX) are scene-level and post-engine — out of scope for this survey except for the bend-routing parameters (`BEND PITCH`, `BEND FILTER`).

The AnalogSynth signal path is a **single-OSC** subtractive structure (mixed sources, not two independent VCOs). It does **not** offer cross-modulation, oscillator hard sync, or ring modulation between oscillators — those features exist on the JUNO-X but only inside the **ZCore** partial-based Tone PMT structure (`SYNC`, `RING`, `XMOD`, `XMOD2`), which is outside this survey's scope. **Mod matrix**: the AnalogSynth engine has no general-purpose mod matrix; it has only the fixed LFO routings and the aftertouch fixed routings (AFT LFO / AFT FREQ / AFT LEVEL) listed under Modulation routing. Roland's general-purpose Matrix Control (8 slots) lives in the ZCore partials only.

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit Continuous Controllers; the Parameter Guide lists wider underlying parameter ranges (e.g. `LFO RATE` 0–1023, `FLT ENV DEPTH` -1023..+1023, `BEND PITCH` 0–1200) that are reachable at full resolution only via SysEx into the `[MDLSYN0] Analog Synth Model Parameter` block. The CC pathway is quantized to 128 steps with a hardware-defined non-linear taper (knob taper), and the `analog-synth.ts` map encodes every continuous parameter as `{ kind: "raw", min: 0, max: 127 }`. Discrete switches and enums use a label-table convention (see Quirks): `min: 0, max: N` with `labels: { 0: "VAL0", 1: "VAL1", … N: "VALN" }`, where each integer 0..N selects exactly one label and intermediate CC values are rounded to the nearest label step. Every `as_*` parameter is `perPart: true`, meaning the parameter applies to a specific part within the active scene (the parameter map is parameterized over a part index when the device sends SysEx).

### Oscillator (single-OSC mixed-source)
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| OSC PITCH | `as_osc_pitch` | continuous | 0–127 (raw MIDI CC) | semitones (knob taper, underlying -12..+12) | — | Adjusts the coarse pitch of the oscillator in semitones (CC 20). Bipolar on the panel knob; the CC value collapses bipolar onto 0–127 with 64 = center. |
| OSC DETUNE | `as_osc_detune` | continuous | 0–127 (raw MIDI CC) | cents (knob taper, underlying -50..+50 of a "split detune" between SAW and SUB) | — | Detunes the SAW vs. SUB-OSC against each other for a thicker chorus-style detune (CC 21). Bipolar on the panel knob; 64 = center. |
| PW LEVEL / SSAW LEVEL | `as_pw_level` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–255) | — | Pulse-wave level when `SUPER SAW = OFF`; doubles as Super-SAW source level when `SUPER SAW = ON` (CC 16). |
| SAW LEVEL | `as_saw_level` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–255) | — | Sawtooth-wave level into the OSC mixer (CC 17). |
| SUB LEVEL | `as_sub_level` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–255) | — | Square sub-oscillator (one octave below) level into the OSC mixer (CC 18). |
| SUPER SAW | `as_super_saw` | toggle | 0–1 (raw MIDI CC) | enum | OFF / ON | Replaces the pulse channel with a Super-SAW (detuned saw stack) (CC 46). |
| OSC LFO MOD | `as_osc_lfo_mod` | continuous | 0–127 (raw MIDI CC) | depth (knob taper, underlying 0–100) | — | Depth of LFO modulation routed to oscillator pitch — vibrato (CC 26). |

Out-of-common-core (Tone PMT level on partial-based engines, NOT exposed at the AnalogSynth level): `OSC SYNC`, `RING modulator`, `XMOD / XMOD2` cross-modulation. These exist on the JUNO-X but only inside ZCore Tone PMT (`Struct12`, `Struct34` switches), not in `analog-synth.ts`.

### Mixer (noise + bend routings)
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| NOISE LEVEL | `as_noise_level` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–255) | — | White-noise level into the mixer (CC 19). |

The AnalogSynth engine does **not** expose a separate ring-mod level CC at the engine level (ring/cross-mod live in ZCore partials, out-of-common-core).

### High-pass filter
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| HPF-STEP | `as_hpf_step` | discrete | 0–3 (raw MIDI CC) | enum | 0 / 1 / 2 / 3 | Sets the high-pass filter cutoff in four discrete steps (0 = effectively off, 3 = highest cutoff) (CC 79). The HPF is **stepped, not continuous**, faithfully modeling the original JUNO-60 / JUNO-106 HPF behavior. |

### Low-pass filter
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| VINTAGE FLT TYPE | `as_vintage_flt_type` | discrete | 0–2 (raw MIDI CC) | enum | R (Roland) / M (Moog-style) / S (sharp) | Selects one of three response curves modeling vintage analog LPFs (CC 108). Multi-mode filter (out-of-common-core if the canonical schema fixes a single LPF behavior). |
| CUTOFF | `as_cutoff` | continuous | 0–127 (raw MIDI CC) | cutoff (knob taper, underlying 0–1023) | — | Low-pass cutoff frequency (CC 3). |
| RESONANCE | `as_resonance` | continuous | 0–127 (raw MIDI CC) | resonance (knob taper, underlying 0–1023) | — | Low-pass resonance / Q (CC 9). |
| FLT ENV DEPTH | `as_flt_env_depth` | continuous | 0–127 (raw MIDI CC) | bipolar amount (knob taper, underlying -1023..+1023) | — | Filter envelope amount into low-pass cutoff (CC 81). Bipolar on the panel knob; the CC value collapses bipolar onto 0–127 with 64 = center (out-of-common-core: signed amount knob behavior — see Quirks). |
| FLT KEY FOLLOW | `as_flt_key_follow` | continuous | 0–127 (raw MIDI CC) | tracking (knob taper, underlying -200..+200) | — | Keyboard tracking amount into low-pass cutoff (CC 122 on the JUNO-X variant; CC 82 on the 106/60 sub-tone variants). Bipolar on the panel knob; 64 = center. |
| FILTER MOD | `as_flt_lfo_mod` | continuous | 0–127 (raw MIDI CC) | depth (knob taper, underlying 0–100) | — | Depth of LFO modulation routed to LPF cutoff — filter wah (CC 28). |
| FLT VSENS | `as_flt_vsens` | continuous | 0–127 (raw MIDI CC) | sensitivity (knob taper, underlying -100..+100) | — | How much key velocity affects LPF cutoff (CC 53). Bipolar; 64 = center. |

### Amp (VCA)
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| AMP LEVEL | `as_amp_level` | continuous | 0–127 (raw MIDI CC) | level | — | Overall VCA output level (CC 110). |
| AMP MOD | `as_amp_lfo_mod` | continuous | 0–127 (raw MIDI CC) | depth (knob taper, underlying 0–100) | — | Depth of LFO modulation routed to VCA level — tremolo (CC 30). |
| AMP VSENS | `as_amp_vsens` | continuous | 0–127 (raw MIDI CC) | sensitivity (knob taper, underlying -100..+100) | — | How much key velocity affects VCA level (CC 54). Bipolar; 64 = center. |

### Amp / filter envelope (single shared ADSR)
The AnalogSynth engine has **one shared ADSR** that drives both the VCA and (scaled by `FLT ENV DEPTH`) the LPF cutoff. This is faithful to the original JUNO-60 / JUNO-106 architecture and **differs from most of the other devices in this survey, which expose two independent envelopes**. Recorded under Quirks.

| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| ENV ATTACK | `as_env_attack` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | ADSR attack time (CC 89). |
| ENV DECAY | `as_env_decay` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | ADSR decay time (CC 90). |
| ENV SUSTAIN | `as_env_sustain` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–1023) | — | ADSR sustain level (CC 102). |
| ENV RELEASE | `as_env_release` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | ADSR release time (CC 103). |

### Pitch envelope (out-of-common-core)
A **second, independent 4-stage envelope** modulates oscillator pitch. This is unusual for a JUNO-style box and out-of-common-core if the canonical schema fixes "filter env + amp env" as the only two envelope slots.

| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| PENV ATTACK | `as_penv_attack` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | Pitch envelope attack time (CC 83). Out-of-common-core. |
| PENV DECAY | `as_penv_decay` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | Pitch envelope decay time (CC 80). Out-of-common-core. |
| PENV SUSTAIN | `as_penv_sustain` | continuous | 0–127 (raw MIDI CC) | level (knob taper, underlying 0–1023) | — | Pitch envelope sustain level (CC 85). Out-of-common-core. |
| PENV RELEASE | `as_penv_release` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | Pitch envelope release time (CC 86). Out-of-common-core. |

### LFO (single LFO)
The AnalogSynth engine has **one** LFO with three fixed depth-knob destinations (OSC pitch via `OSC LFO MOD`, LPF cutoff via `FILTER MOD`, VCA level via `AMP MOD`) — not a destination-selector enum. The Parameter Guide does not document a second LFO at the AnalogSynth tone level; LFO 2 / per-partial LFOs / general-purpose mod matrix exist only inside ZCore (out-of-common-core).

| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| LFO WAVEFORM | `as_lfo_waveform` | discrete | 0–3 (raw MIDI CC) | enum | SIN / SAW-DW / SQR / S&H | LFO shape (CC 35). The label-to-CC convention is the JUNO-X-specific encoding (see Quirks). |
| LFO RATE | `as_lfo_rate` | continuous | 0–127 (raw MIDI CC) | rate (knob taper, underlying 0–1023) | — | LFO cycle speed (CC 29). |
| LFO DELAY TIME | `as_lfo_delay_time` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | Delay from key-on to LFO modulation onset (CC 27). |
| LFO SYNC | `as_lfo_sync` | toggle | 0–1 (raw MIDI CC) | enum | OFF / ON | Syncs LFO RATE to MIDI clock / sequencer tempo (CC 117). When ON, the underlying parameter is `LFO NOTE` (note-length enum: 1/64T … 4) addressable via SysEx, not via CC. |

LFO depth-routing knobs `OSC LFO MOD` (CC 26), `FILTER MOD` (CC 28), `AMP MOD` (CC 30) are listed under their respective sections (Oscillator, LPF, Amp) since each is a destination-side depth, not a source-side LFO parameter.

### Performance / voice / portamento
| Device-native name | `analog-synth.ts` key | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|---|
| BEND PITCH | `as_bend_pitch` | continuous | 0–127 (raw MIDI CC) | semitones (knob taper, underlying 0–1200 cents) | — | Pitch bend range (CC 41). |
| BEND FILTER | `as_bend_filter` | continuous | 0–127 (raw MIDI CC) | depth (knob taper, underlying -63..+63) | — | How much pitch bend modulates LPF cutoff (CC 14). Bipolar; 64 = center. |
| PORTA MODE | `as_porta_mode` | toggle | 0–1 (raw MIDI CC) | enum | OFF / ON | Portamento on/off (CC 118). |
| PORTA TIME | `as_porta_time` | continuous | 0–127 (raw MIDI CC) | time (knob taper, underlying 0–1023) | — | Portamento glide time (CC 5). |
| KEY MODE | `as_key_mode` | discrete | 0–3 (raw MIDI CC) | enum | POLY / SOLO / UNISON / SL-UNISON | Voice-allocation mode (CC 119). UNISON and SL-UNISON are out-of-common-core (voice-allocation feature). |

Out-of-common-core (in the Parameter Guide AnalogSynth tone block but not in `analog-synth.ts` — SysEx-only): `PORTA CRV` (ORIGINAL / LINEAR / EXP1 / EXP2 portamento curve), `AFT LFO` / `AFT FREQ` / `AFT LEVEL` (aftertouch fixed routings, -63..+63 each), `PITCH DRIFT` (0–255, vintage-detune simulation), `CONDITION` (0–100, vintage-aging simulation), `PARAM EXPANSION` (OFF/ON, expanded LFO RATE / CUTOFF / RESONANCE range), `OSC PAN SPLIT` (-50..+50 stereo splay across PW/SAW/SUB/NOISE, CC 55), `AMP ENV SEL` (ENV F&A / G-AMP — switches whether the VCA follows the envelope or a gate), `PENV VSENS` / `PENV DEPTH`. Recorded for completeness; not addressable via the keyboards-mcp CC pathway.

### Effects bus (out-of-common-core)
The AnalogSynth tone block exposes one bend-into-filter routing (`BEND FILTER`, CC 14, listed under Performance) and the portamento parameters; everything else in the FX path (chorus, delay, reverb, drive, MFX, system EQ/Comp) is at the **scene** level — addressed via SysEx into `Scene Chorus` / `Scene Delay` / `Scene Reverb` / `Scene Drive` blocks (offsets `00 50 00 00`–`00 53 00 00` per `engine-types.ts`) — and is post-engine, **out-of-common-core**, recorded only under Modulation routing.

## Modulation routing

The AnalogSynth engine has a **fixed routing topology** — there is no general-purpose mod matrix. All routings are listed verbatim from the Parameter Guide:

- **LFO destinations (depth-knob per destination, not a destination-selector enum):**
  - LFO → OSC pitch via `OSC LFO MOD` (CC 26, 0–127 depth) — vibrato.
  - LFO → LPF cutoff via `FILTER MOD` (CC 28, 0–127 depth, exposed as `as_flt_lfo_mod`) — filter wah.
  - LFO → VCA level via `AMP MOD` (CC 30, 0–127 depth, exposed as `as_amp_lfo_mod`) — tremolo.
  - LFO → pulse width / Super-SAW detune via `PW MODE = LFO` (SysEx-only enum: LFO / MANUAL / ENV; the depth knob is `PULSE WIDTH MOD / SSAW DETUNE`, CC 50, exposed via SysEx-only on `analog-synth.ts`).
- **LFO shapes (`as_lfo_waveform` enum, `labels: { 0: "SIN", 1: "SAW-DW", 2: "SQR", 3: "S&H" }`):** SIN, SAW-DW (sawtooth-down), SQR, S&H (sample-and-hold).
- **LFO depth scaling at performance time:** `MODULATION LFO` (mod-wheel-driven, -63..+63, SysEx-only on `analog-synth.ts`) scales LFO depth from the mod wheel.

- **Velocity routings (continuous depth knobs):**
  - Velocity → LPF cutoff via `FLT VSENS` (CC 53, bipolar).
  - Velocity → VCA level via `AMP VSENS` (CC 54, bipolar).
  - Velocity → pitch envelope depth via `PENV VSENS` (-100..+100, SysEx-only) — out-of-common-core.

- **Envelope-as-mod-source routings:**
  - Single ADSR → VCA (always-on, scaled by `AMP ENV SEL = ENV F&A`).
  - Single ADSR → LPF cutoff via `FLT ENV DEPTH` (CC 81, bipolar).
  - Single ADSR → pulse width / Super-SAW detune via `PW MODE = ENV`.
  - Pitch ENV → OSC pitch via `PENV DEPTH` (-100..+100, SysEx-only) — out-of-common-core.

- **Aftertouch fixed routings (SysEx-only on `analog-synth.ts`):** `AFT LFO`, `AFT FREQ`, `AFT LEVEL` (each -63..+63). Out-of-common-core (per-voice pressure mod).

- **Pitch-bend fixed routings:** `BEND PITCH` (CC 41) → OSC pitch; `BEND FILTER` (CC 14) → LPF cutoff.

- **Mod matrix slots:** **none at the AnalogSynth engine layer.** Roland's general-purpose Matrix Control (8 slots × source × destination × depth) lives in the ZCore partial-based Tone, not in AnalogSynth. Out-of-common-core for this survey.

The Parameter Guide JUNO-X / JUNO-106 / JUNO-60 tone blocks (pp. 18–20) document the full routing topology; SysEx address map for the routing block is `[MDLSYN0]` → `Analog Synth Model Parameter` per the MIDI Implementation v1.10.

## Quirks

- **Multi-engine workstation; this survey is one slice.** The JUNO-X presents four engines per scene part (AnalogSynth, ZCore, JunoXModel, RDPiano) plus scene-level FX, vocoder, and rhythm; only AnalogSynth is subtractive. ZCore's partial-based features (OSC sync, ring mod, cross-mod, 8-slot mod matrix, dual LFOs per partial) are excluded — they belong in a separate "ZCore subtractive-where-applicable" survey if ever needed.
- **AnalogSynth is single-OSC mixed-source, not 2-OSC.** Unlike most other devices in this survey, AnalogSynth exposes one oscillator section with mixed sources (PW/SSAW + SAW + SUB + NOISE), not two independent VCOs. This faithfully models the original JUNO-60 / JUNO-106 architecture. The implication for the canonical ontology is that "Oscillator 2" parameters from a 2-OSC schema have no AnalogSynth analogue (the SUB-OSC is one octave below OSC 1, fixed).
- **Single shared ADSR drives both VCA and LPF cutoff.** There is no separate Filter Envelope; `FLT ENV DEPTH` (bipolar) is an amount control that scales the shared ADSR into the LPF. This affects how the canonical schema's "filter envelope" slot maps to AnalogSynth.
- **Pitch envelope is a third, independent envelope.** `PENV ATTACK / DECAY / SUSTAIN / RELEASE` is a fully independent 4-stage envelope dedicated to pitch modulation — out-of-common-core (most subtractive ontologies fix two envelopes).
- **CC values are raw 7-bit (0–127); underlying parameters are wider.** The Parameter Guide documents underlying ranges of 0–1023 (envelope times, LFO rate, cutoff, resonance), -1023..+1023 (FLT ENV DEPTH), 0–255 (level CCs, pitch drift), 0–1200 (BEND PITCH, in cents), -200..+200 (FLT KEY FOLLOW), -100..+100 / -50..+50 (bipolar amounts), -63..+63 (aftertouch / mod-wheel routings). The CC pathway is quantized to 128 steps with a hardware-defined non-linear taper. Full resolution is reachable via SysEx into the `[MDLSYN0]` block. Every continuous CC in `analog-synth.ts` is `encoding: { kind: "raw" }` with `min: 0, max: 127`.
- **Bipolar amounts collapse onto 0–127 with 64 = center.** Knobs documented as -1023..+1023 / -200..+200 / -100..+100 / -63..+63 / -50..+50 (FLT ENV DEPTH, FLT KEY FOLLOW, FLT/AMP VSENS, BEND FILTER, OSC PITCH, OSC DETUNE, etc.) all collapse onto 7-bit unsigned via the JUNO-X CC convention with 64 = center.
- **Discrete-with-labels encoding (label-to-CC convention).** The `analog-synth.ts` file encodes every multi-step discrete parameter as `min: 0, max: N` with an explicit `labels: { 0: "VAL0", 1: "VAL1", … N: "VALN" }` table. Examples: `as_lfo_waveform` `{ 0: "SIN", 1: "SAW-DW", 2: "SQR", 3: "S&H" }`; `as_vintage_flt_type` `{ 0: "R", 1: "M", 2: "S" }`; `as_hpf_step` `{ 0: "0", 1: "1", 2: "2", 3: "3" }`; `as_key_mode` `{ 0: "POLY", 1: "SOLO", 2: "UNISON", 3: "SL-UNISON" }`; `as_super_saw` / `as_lfo_sync` / `as_porta_mode` `{ 0: "OFF", 1: "ON" }`. This is the JUNO-X label-to-CC convention: each integer 0..N selects exactly one label (no implicit thresholding within 128 steps), and CC values intermediate between integers are rounded to the nearest label step. This convention contrasts with the Prophet-6 (which uses `min: 0, max: 1` for toggles and the same enum-by-integer scheme for multi-step discretes; the labels are similar in shape, but the JUNO-X explicitly uses string labels including non-numeric tokens like `"SAW-DW"`, `"SL-UNISON"`, `"R"/"M"/"S"`).
- **`perPart: true` semantics.** Every `as_*` parameter in `analog-synth.ts` is flagged `perPart: true`, meaning the parameter's value is scoped to a specific part within the active scene. The JUNO-X scene holds five parts (per `engine-types.ts` `PART_COUNT = 5`), each of which can run any of the four engines. SysEx addressing for AnalogSynth tones uses bases `02 10 00 00` … `02 13 00 00` (four AnalogSynth tone slots in temporary memory). The parameter map is parameterized over a part index when sending SysEx; CC pathway uses the part's MIDI channel.
- **Three sub-tone variants share the AnalogSynth engine; CCs vary.** Within AnalogSynth, the panel offers three sub-tone variants: TONE JUNO-X, TONE JUNO-106, TONE JUNO-60. They share most CCs but differ on a handful: `FLT KEY FOLLOW` (CC 122 on JUNO-X, CC 82 on 106/60); `OSC PITCH` is bipolar -12..+12 on JUNO-X but a 3-step `OSC RANGE` enum (16'/8'/4' or DOWN/NORMAL/UP) on 106/60; `PW LEVEL / SAW LEVEL` are continuous mixers on JUNO-X but `PW SWITCH` / `SAW SWITCH` (OFF/ON only) on 106/60; `MODULATION LFO` range is -63..+63 on JUNO-X but 0–63 on JUNO-106. The keyboards-mcp `analog-synth.ts` exposes the **JUNO-X-variant** CC set; routing the same `as_*` parameter to a 106 or 60 sub-tone may invoke a slightly different parameter at the device. The MCP layer assumes JUNO-X variant — out-of-common-core if the canonical schema needs to address a specific sub-tone.
- **Vintage filter type is a multi-mode switch.** `VINTAGE FLT TYPE` (CC 108) selects R / M / S response curves modeling Roland / Moog / sharp LPFs respectively. Out-of-common-core if the canonical schema fixes a single LPF behavior — though it can be mapped to a "filter character" enum if the schema supports multi-mode filters.
- **HPF is stepped, not continuous.** `HPF-STEP` (CC 79) has only four values (0–3) — faithful to JUNO-60 / JUNO-106 architecture. The canonical schema's "high-pass cutoff" continuous parameter does not have a direct AnalogSynth analogue; mapping is a 4-step quantization.
- **Super SAW reuses the pulse channel.** When `SUPER SAW = ON`, the `PW LEVEL` slider becomes Super-SAW source level, the `PULSE WIDTH MOD` knob becomes Super-SAW detune-depth, and `PW MODE = LFO/MANUAL/ENV` selects Super-SAW detune-modulation source. This dual-role-knob behavior is captured in the parameter `name`s ("PW Level / SSAW Level"). Out-of-common-core if the canonical schema separates "pulse width" from "detune depth".
- **Vintage simulation parameters (`PITCH DRIFT`, `CONDITION`, `PARAM EXPANSION`, `AMP ENV SEL = G-AMP`).** Modeled after vintage-knob behavior; SysEx-only on the keyboards-mcp side. Out-of-common-core (vintage-knob simulation, parallel to Prophet-6's `Slop`).
- **Effects out of scope.** Scene-level chorus (`SCENE_CHORUS_OFFSET 00 50 00 00`), delay (`00 51 00 00`), reverb (`00 52 00 00`), drive (`00 53 00 00`), MFX, system EQ/Comp, and vocoder are post-engine and out of scope for this subtractive-engine survey.
- **No general-purpose mod matrix at the AnalogSynth layer.** Routing is fixed (LFO → 3 destinations; pitch ENV → pitch; ADSR → VCA + LPF; velocity → LPF + VCA + pitch ENV; aftertouch → LFO + LPF + VCA; pitch bend → pitch + LPF). The 8-slot Matrix Control on the JUNO-X belongs to ZCore partials only — out-of-common-core for AnalogSynth.
- **CC vs Parameter Guide agreement (verified row-by-row against `analog-synth.ts`):** LFO CCs 35/29/27/117; OSC CCs 20/21/16/17/18/19/46/26; HPF CC 79; Filter CCs 108/3/9/81/122/28/53; Amp CCs 110/30/54; Env CCs 89/90/102/103; Pitch ENV CCs 83/80/85/86; Performance CCs 41/14/118/5/119. All match the **TONE JUNO-X** column of the Parameter Guide. No discrepancies found between `analog-synth.ts` and the public guide for the JUNO-X variant.
