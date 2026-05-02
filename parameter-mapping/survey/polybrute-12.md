# PolyBrute 12 — Parameter Survey

**Manufacturer:** Arturia
**Year:** 2024
**Engine type:** analog
**Polyphony:** 12
**Sources:**
- Arturia, *PolyBrute 12 User Manual* v3.1.2 EN (current at the time of writing): https://dl.arturia.net/products/polybrute-12/manual/polybrute-12_Manual_3_1_2_EN.pdf (also indexed at ManualsLib: https://www.manualslib.com/manual/3488409/Arturia-Polybrute-12.html)
- Arturia, *PolyBrute 12* product overview / details: https://www.arturia.com/products/hardware-synths/polybrute-12/overview and https://www.arturia.com/products/hardware-synths/polybrute-12/details
- midi.guide community CC database, *Arturia PolyBrute MIDI CCs and NRPNs*: https://midi.guide/d/arturia/polybrute/ (the database covers the original PolyBrute; per the Arturia user-forum thread cited below, the PolyBrute 12 deliberately preserves the same CC assignments for back-compatibility with PolyBrute sequences)
- Arturia, *PolyBrute 12 — MPE (MIDI Polyphonic Expression)* support article: https://support.arturia.com/hc/en-us/articles/14545577398556-PolyBrute-12-MPE-MIDI-Polyphonic-Expression
- Arturia user-forum, *Hello! And MIDI Implementation Chart for Polybrute 12?* (community thread documenting that the manual's section 13.1 *MIDI Continuous Controller assignments* lists Morph 3, Ribbon 9, Morphée X 114, Morphée Y 115, Morphée Z 89 — and that the standard `PolyBrute MIDI` USB port does not expose Morphée and Ribbon as mappable CCs; only the `PolyBrute VST` USB port carries them as NRPNs for use by PolyBrute Connect): https://forum.arturia.com/t/hello-and-midi-implementation-chart-for-polybrute-12/5327
- Arturia user-forum, *POLYBRUTE 12 MIDI Control Functionality could be so much better* (community thread documenting non-standard CC assignments that PolyBrute 12 inherits unchanged from the original PolyBrute — "MIDI CC2 (breath controller) affecting reverb level, CC7 (volume) affecting the Steiner filter level, and CC65 (portamento on/off) affecting VCO1 sync"): https://forum.arturia.com/t/polybrute-12-midi-control-functionality-could-be-so-much-better-here-are-some-ideas/5971
- Sound on Sound, *Arturia PolyBrute 12* review: https://www.soundonsound.com/reviews/arturia-polybrute-12

No `keyboards-mcp` cross-check is available for this device — sourcing is web-only. CC numbers below are taken row-by-row from the midi.guide chart (which mirrors the manual's section 13.1 *MIDI Continuous Controller assignments*) and cross-checked against the Arturia user-forum thread for the controller and Morph assignments. Parameter names below are reproduced verbatim from the chart and from the Arturia overview / details pages. **PolyBrute 12 vs original PolyBrute:** the PolyBrute 12 (2024) is a 12-voice analog flagship that doubles the original PolyBrute's (2020) 6-voice polyphony and adds the FullTouch® MPE keyboard plus a third LFO (LFO 3 with shape/symmetry); per Sound on Sound, "much of the PolyBrute 12 is the same as its predecessor" — synthesis architecture (oscillators, filters, mixer, mod matrix size, Morphée, ribbon) is unchanged from the original PolyBrute. Per the user-forum thread, **the MIDI CC assignments are preserved unchanged** between the two models for sequence back-compatibility. Where Arturia's manual sections may be shared between the two devices, this survey describes the PolyBrute 12 specifically and notes any differences in Quirks.

## Signal path

Two analog VCOs per voice (`VCO 1`, `VCO 2`) generate sawtooth, triangle, and square waves simultaneously (per Arturia details). VCO 1 carries a **Metalizer** wavefolding circuit (described by Arturia as "wavefolding on VCO1 to escalate from pure to punchy") and a continuously-variable `Sawtooth/Triangle mix` and `Sawtooth/Square mix` instead of discrete waveform switches — the two mix knobs are the per-osc Brute wave-shaping path. VCO 2 has a **sub oscillator** ("extra lower octave and some added rumble" per Arturia details) with its own mix knob (`Sub-oscillator mix`, CC 14), plus the same Saw/Tri and Saw/Sq mix knobs. **`Linear FM` (`Frequency modulation 2 > 1`, CC 77) routes VCO 2 as an FM source into VCO 1; `Sync` (`VCO 1 Sync`, CC 65) hard-syncs VCO 1 to VCO 2; and `Brute Factor` is a feedback control on VCF 1. The Metalizer, the Brute Factor, the Saw/Tri+Saw/Sq dual-mix paths, the Linear FM, and the hard sync are all out-of-common-core (Brute wave-shaping and oscillator-rate modulation paths beyond a single waveform select).** A separate **noise generator** offers a `Color` knob (CC 22) sweeping continuously from rumble to white noise.

The mixer mixes `Oscillator 1 level` (CC 18), `Oscillator 2 level` (CC 19), and `Noise level` (CC 21) into the filter section.

After the mixer, audio passes into **two analog filters in a routable topology**: **`VCF 1 - Steiner-Parker`** (12 dB/oct, 2-pole, with continuous sweepable slope morphing between Lowpass / Notch / Highpass / Bandpass modes via the `Filter mode` parameter, CC 81; plus a `Brute Factor` feedback control, CC 82, that adds harmonic distortion / self-oscillation character); and **`VCF 2 - Ladder`** (24 dB/oct, 4-pole low-pass, "Moog-inspired … added gain compensation and asymmetrical distortion" per Arturia, with a dedicated `Distortion` knob, CC 85). The two filters are routable in three configurations via the `Series/Parallel` parameter (CC 86), which is — per Arturia — a **continuously variable blend** between Series (VCF 1 → VCF 2) and Parallel (sum) routing rather than a discrete switch. A `Master cutoff` knob (CC 27) jointly drives both filter cutoffs as a bipolar offset relative to each filter's own cutoff knob. **The dual-filter topology, the Steiner-Parker filter mode morph, the Ladder Distortion path, the continuous Series/Parallel blend, and the Master cutoff joint-drive are all out-of-common-core (the canonical schema's common-core has a single LP filter). The Ladder filter cutoff (`Ladder Filter Cutoff`, CC 25) and resonance (`Ladder Filter Resonance`, CC 87) map to the canonical common-core LP cutoff and resonance.**

A pair of dedicated **Filter FM** paths route audio-rate modulation into each filter: `Oscillator 2 to filter 1` (CC 79) routes VCO 2 into VCF 1 cutoff; `Noise to filter 2` (CC 80) routes the noise generator into VCF 2 cutoff. **Both Filter FM paths are out-of-common-core (audio-rate filter modulation).**

After the filter section, audio passes into a VCA. **Three envelope generators** drive the engine: a velocity-sensitive `Filter Envelope` (ADSR — A/D/S/R on CCs 102/103/28/104, plus `Velocity` on CC 94); a velocity-sensitive `Amplifier Envelope` (ADSR — CCs 105/106/29/107, `Velocity` on CC 95); and a third **`Modulation Envelope`** (DADSR — Delay / Attack / Decay / Sustain / Release on CCs 108/109/110/30/111) which Arturia describes as offering "repeat and loop modes" and a delay stage, freely assignable through the mod matrix. **The Modulation Envelope is out-of-common-core (third envelope, with delay stage and loop modes).**

The PolyBrute 12 carries **three LFOs**: `LFO 1` (panel `Phase` CC 90 + `Rate` CC 91 with waveform selection), `LFO 2` (panel `Fade in` CC 92 + `Rate` CC 93, also with waveform selection), and `LFO 3` (panel `Curve` CC 67 + `Symmetry` CC 68 + `Rate` CC 73 — LFO 3 uses Curve/Symmetry continuous waveshape morph instead of discrete waveform select per Arturia details). All three LFOs support tempo sync and retrigger options per Arturia. **LFO 2 and LFO 3 are out-of-common-core (additional LFO sources). LFO 1's panel-fixed routing covers the common-core small destination set; richer per-LFO destinations are reached only via the mod matrix and are out-of-common-core.**

A **12 × 32 modulation matrix** (12 mod sources × 32 destinations × up to 64 active connections per Arturia) sits above the engine, with the Morph A/B knob (`Morph`, CC 3 per the manual section 13.1 / CC 11 per the midi.guide chart — see Quirks for the discrepancy) acting as a global cross-fade between two complete patch states. **The mod matrix is out-of-common-core (the canonical schema's common-core has only fixed per-LFO destination switches, not a free assignable matrix). The Morph A/B feature is out-of-common-core (per-patch dual-state morphing).**

`Glide` (CC 5) handles portamento. The PolyBrute 12 also carries a 64-step polyphonic `Sequencer` (with three parallel modulation tracks and a Motion Recorder), a Matrix Arpeggiator, three stereo digital effects sections (Modulation, Delay, Reverb — Delay with 9 algorithms including BBD; Reverb with 9 algorithms including hall, plate, spring, shimmer), the **Morphée** 3-axis touch+pressure controller (X / Y / Z), a **ribbon controller**, a **Morph A/B** dual-state per-patch system, and the FullTouch® MPE keyboard with five aftertouch modes (Mono AT, Poly AT, FullTouch AT, FullTouch AT > Z, FullTouch Env > AT). **All effects, sequencer/arp, Morphée, ribbon, Morph A/B, and poly-aftertouch / FullTouch are out-of-common-core (effects / performance / sequencer / mod-matrix-source domain).**

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit MIDI Continuous Controllers per the manual section 13.1 / midi.guide chart. The PolyBrute 12 uses two value conventions per the chart: **Centered** (knob center = 64; bipolar parameters such as Tune, Master cutoff, Filter envelope amount, LFO 1 Phase, LFO 3 Symmetry) and **0-based** (knob counter-clockwise = 0; unipolar parameters such as level, rate, time, depth knobs). **No NRPN parameter numbers are published in the public manual section 13.1 for general MIDI use.** The PolyBrute 12 transmits Morphée and Ribbon as NRPNs **only on the `PolyBrute VST` USB MIDI port** (a private port reserved for the PolyBrute Connect software editor) — per the Arturia user-forum thread, the standard `PolyBrute MIDI` USB port does not expose Morphée or Ribbon as mappable CCs; the standard port additionally outputs Morphée X / Y / Z as CC 114 / 115 / 89 and Ribbon as CC 9, but per a separate forum thread (#5971) **incoming CC 9 / 89 / 114 / 115 is currently not received by the engine** — recorded under Quirks.

### Oscillator 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCO 1 Tune | continuous | 0–127 (raw MIDI CC 66) | semitones (knob taper, bipolar — center = 64) | — | VCO 1 base pitch offset. Centered: 64 = no transposition. |
| VCO 1 Sawtooth/Triangle mix | continuous | 0–127 (raw MIDI CC 10) | morph | — | Continuously-variable mix between VCO 1's sawtooth (0) and triangle (127) waveshapes — the panel knob controlling the saw/tri blend. |
| VCO 1 Sawtooth/Square mix | continuous | 0–127 (raw MIDI CC 12) | morph | — | Continuously-variable mix between VCO 1's sawtooth (0) and square (127) waveshapes. |
| VCO 1 Pulse width | continuous | 0–127 (raw MIDI CC 69) | pulse-width | — | Pulse width of VCO 1's square wave path. Square at 12 o'clock; narrow at extremes. |
| VCO 1 Metalizer | continuous | 0–127 (raw MIDI CC 70) | drive | — | Wavefolding amount on VCO 1 — adds harmonic content via Brute-style wavefolder. **Out-of-common-core (Brute wave-shaping nonlinearity).** |
| VCO 1 Sync | toggle / continuous | 0–127 (raw MIDI CC 65) | enum / on-off | — | Hard sync — VCO 1's waveform restarts on each cycle of VCO 2. The chart row publishes the parameter as a 0–127 CC, but the underlying parameter is on/off in the engine; the CC is read as a threshold. (Note: CC 65 is the standard MIDI Portamento On/Off controller — see Quirks.) **Out-of-common-core (oscillator sync).** |

### Oscillator 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| VCO 2 Tune | continuous | 0–127 (raw MIDI CC 72) | semitones (knob taper, bipolar — center = 64) | — | VCO 2 base pitch offset. Centered: 64 = unison with VCO 1. |
| VCO 2 Sawtooth/Triangle mix | continuous | 0–127 (raw MIDI CC 15) | morph | — | Continuously-variable mix between VCO 2's sawtooth (0) and triangle (127). |
| VCO 2 Sawtooth/Square mix | continuous | 0–127 (raw MIDI CC 16) | morph | — | Continuously-variable mix between VCO 2's sawtooth (0) and square (127). |
| VCO 2 Pulse width | continuous | 0–127 (raw MIDI CC 75) | pulse-width | — | Pulse width of VCO 2's square wave path. |
| VCO 2 Sub-oscillator mix | continuous | 0–127 (raw MIDI CC 14) | morph | — | Mix amount of VCO 2's sub-octave oscillator into the VCO 2 output path. |
| VCO 2 Frequency modulation 2 > 1 | continuous | 0–127 (raw MIDI CC 77) | amount | — | Linear FM depth — VCO 2 → VCO 1 frequency modulation. **Out-of-common-core (oscillator-rate FM).** |

### Sub
The VCO 2 sub-oscillator is a built-in sub-octave generator on VCO 2; the mix knob is published in the Oscillator 2 table above as `VCO 2 Sub-oscillator mix` (CC 14). No separate sub-oscillator level / waveshape parameters are published in section 13.1 — the sub follows VCO 2's pitch and the mix knob is the only programmable parameter.

### Noise
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Noise Color | continuous | 0–127 (raw MIDI CC 22) | color | — | Continuous noise color from rumble (0) to white noise (127). |

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mixer Oscillator 1 level | continuous | 0–127 (raw MIDI CC 18) | level | — | VCO 1 (post-Metalizer / waveform-mix) level into the filter section. |
| Mixer Oscillator 2 level | continuous | 0–127 (raw MIDI CC 19) | level | — | VCO 2 (post-sub / waveform-mix) level into the filter section. |
| Mixer Noise level | continuous | 0–127 (raw MIDI CC 21) | level | — | Noise generator level into the filter section. |

### Filter — Ladder (canonical common-core LP)
The Ladder filter is the PolyBrute 12's dedicated 24 dB/oct low-pass filter and is the row that maps to the canonical common-core LP filter. Its cutoff and resonance below are the canonical LP parameters; the Steiner-Parker filter and the dual-filter routing are recorded separately and flagged out-of-common-core.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Ladder Filter Cutoff | continuous | 0–127 (raw MIDI CC 25) | cutoff (knob taper) | — | VCF 2 (Ladder) low-pass cutoff frequency. Maps to canonical common-core LP cutoff. |
| Ladder Filter Resonance | continuous | 0–127 (raw MIDI CC 87) | resonance | — | VCF 2 (Ladder) resonance (with gain compensation per Arturia). Maps to canonical common-core LP resonance. |
| Ladder Filter Distortion | continuous | 0–127 (raw MIDI CC 85) | drive | — | Asymmetrical distortion / overdrive on the Ladder filter input path. **Out-of-common-core (Brute wave-shaping / saturation drive on the filter).** |
| Ladder Filter envelope amount | continuous | 0–127 (raw MIDI CC 26) | amount (bipolar — center = 64) | — | Filter envelope depth into the Ladder cutoff. Bipolar (negative = inverted env). |
| Ladder Filter Level | continuous | 0–127 (raw MIDI CC 8) | level | — | Output level of the Ladder filter when blending Series/Parallel. **Out-of-common-core (per-filter output mix in dual-filter topology).** |

### Filter — Steiner-Parker (out-of-common-core)
**The entire Steiner-Parker section is out-of-common-core (second filter / multi-mode filter / Brute Factor).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Steiner Filter Cutoff | continuous | 0–127 (raw MIDI CC 23) | cutoff (knob taper) | — | VCF 1 (Steiner-Parker) cutoff. **OOC.** |
| Steiner Filter Resonance | continuous | 0–127 (raw MIDI CC 83) | resonance | — | VCF 1 resonance. **OOC.** |
| Steiner Filter Brute Factor | continuous | 0–127 (raw MIDI CC 82) | drive | — | Feedback / saturation amount on the Steiner-Parker filter — adds harmonic content and self-oscillation character. **OOC (Brute Factor — distinguishing PolyBrute saturation feature, not part of common-core).** |
| Steiner Filter mode | continuous | 0–127 (raw MIDI CC 81) | morph | — | Continuous slope/mode morph across Lowpass / Notch / Highpass / Bandpass states. **OOC (multi-mode filter morph).** |
| Steiner Filter envelope amount | continuous | 0–127 (raw MIDI CC 24) | amount (bipolar — center = 64) | — | Filter envelope depth into the Steiner-Parker cutoff. **OOC.** |
| Steiner Filter Level | continuous | 0–127 (raw MIDI CC 7) | level | — | Output level of the Steiner-Parker filter. **OOC. (Note: CC 7 is the standard MIDI Volume controller — non-standard mapping; see Quirks.)** |

### Filter — joint controls (out-of-common-core)
**Both rows out-of-common-core (dual-filter joint-drive / routing).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filters Master cutoff | continuous | 0–127 (raw MIDI CC 27) | cutoff offset (bipolar — center = 64) | — | Bipolar offset that scales both `Steiner Filter Cutoff` and `Ladder Filter Cutoff` together. **OOC.** |
| Filters Key tracking | continuous | 0–127 (raw MIDI CC 71) | tracking | — | Common keyboard-tracking amount applied to both filters. **OOC (joint-filter parameter).** |

### Filter FM (out-of-common-core)
**Both rows out-of-common-core (audio-rate filter FM).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter FM Oscillator 2 to filter 1 | continuous | 0–127 (raw MIDI CC 79) | amount | — | VCO 2 audio-rate FM into the Steiner-Parker filter cutoff. **OOC.** |
| Filter FM Noise to filter 2 | continuous | 0–127 (raw MIDI CC 80) | amount | — | Noise generator audio-rate FM into the Ladder filter cutoff. **OOC.** |

### Filter envelope (ADSR, velocity-sensitive)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Filter Envelope Attack | continuous | 0–127 (raw MIDI CC 102) | time (knob taper) | — | Filter envelope attack time. |
| Filter Envelope Decay | continuous | 0–127 (raw MIDI CC 103) | time (knob taper) | — | Filter envelope decay time. |
| Filter Envelope Sustain | continuous | 0–127 (raw MIDI CC 28) | level | — | Filter envelope sustain level. |
| Filter Envelope Release | continuous | 0–127 (raw MIDI CC 104) | time (knob taper) | — | Filter envelope release time. |
| Filter Envelope Velocity | continuous | 0–127 (raw MIDI CC 94) | amount | — | Velocity sensitivity of the filter envelope amount. |

### Amp envelope (ADSR, velocity-sensitive)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Amplifier Envelope Attack | continuous | 0–127 (raw MIDI CC 105) | time (knob taper) | — | VCA envelope attack time. |
| Amplifier Envelope Decay | continuous | 0–127 (raw MIDI CC 106) | time (knob taper) | — | VCA envelope decay time. |
| Amplifier Envelope Sustain | continuous | 0–127 (raw MIDI CC 29) | level | — | VCA envelope sustain level. |
| Amplifier Envelope Release | continuous | 0–127 (raw MIDI CC 107) | time (knob taper) | — | VCA envelope release time. |
| Amplifier Envelope Velocity | continuous | 0–127 (raw MIDI CC 95) | amount | — | Velocity sensitivity of the VCA envelope amount. |

### Third envelope — Modulation Envelope (DADSR, freely assignable)
**The entire Modulation Envelope section is out-of-common-core (third envelope; Delay stage; loop modes).** Per Arturia: "Mod Envelope: DADSR with delay stage and repeat and loop modes."

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Modulation Envelope Delay | continuous | 0–127 (raw MIDI CC 108) | time (knob taper) | — | Mod-envelope pre-attack delay stage. **OOC.** |
| Modulation Envelope Attack | continuous | 0–127 (raw MIDI CC 109) | time (knob taper) | — | Mod-envelope attack time. **OOC.** |
| Modulation Envelope Decay | continuous | 0–127 (raw MIDI CC 110) | time (knob taper) | — | Mod-envelope decay time. **OOC.** |
| Modulation Envelope Sustain | continuous | 0–127 (raw MIDI CC 30) | level | — | Mod-envelope sustain level. **OOC.** |
| Modulation Envelope Release | continuous | 0–127 (raw MIDI CC 111) | time (knob taper) | — | Mod-envelope release time. **OOC.** |

### LFO 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 1 Phase | continuous | 0–127 (raw MIDI CC 90) | phase (bipolar — center = 64) | — | LFO 1 starting phase at note-on (when retrig is enabled). |
| LFO 1 Rate | continuous | 0–127 (raw MIDI CC 91) | rate (knob taper) | — | LFO 1 rate. |

LFO 1 destinations are reachable through the **12 × 32 mod matrix** (see Modulation routing). The published manual section 13.1 / midi.guide chart **does not expose per-LFO-destination on/off switches as CCs** — the canonical small destination set (pitch, PWM, filter cutoff, VCA) is reached via mod-matrix slots, not via dedicated CCs. **Recorded as a documentation gap relative to the OB-X8 / Prophet-5 / Prophet-6 surveys (no panel-fixed LFO routing CCs).** LFO 1's waveform-select knob is on the panel but the chart does not publish a CC for it (recorded as a documentation gap — see Quirks).

### LFO 2 (out-of-common-core)
**LFO 2 is out-of-common-core (second LFO).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 2 Fade in | continuous | 0–127 (raw MIDI CC 92) | time (knob taper) | — | LFO 2 fade-in (depth ramps up after note-on). **OOC.** |
| LFO 2 Rate | continuous | 0–127 (raw MIDI CC 93) | rate (knob taper) | — | LFO 2 rate. **OOC.** |

LFO 2 waveform-select knob is panel-only; not in the chart.

### LFO 3 (out-of-common-core)
**The entire LFO 3 section is out-of-common-core (third LFO; continuous waveshape morph via Curve/Symmetry).** Per Arturia: "LFO3: waveform shaping using Shape and Symmetry."

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 3 Curve | continuous | 0–127 (raw MIDI CC 67) | shape | — | LFO 3 curve / shape morph. **OOC.** |
| LFO 3 Symmetry | continuous | 0–127 (raw MIDI CC 68) | shape (bipolar — center = 64) | — | LFO 3 symmetry — controls duty-cycle / tilt of the LFO waveform. **OOC.** |
| LFO 3 Rate | continuous | 0–127 (raw MIDI CC 73) | rate (knob taper) | — | LFO 3 rate. **OOC.** |

### Modulation Matrix (out-of-common-core)
**The 12 × 32 mod matrix is out-of-common-core in its entirety.** Per Arturia: "12x32 Modulation Matrix with up to 32 destinations and 64 total connections" — i.e. 12 mod sources, 32 mod destinations, with up to 64 active source→destination connections per patch. **As of the manual's section 13.1, the mod-matrix slot source / destination / amount fields are NOT exposed as individual MIDI CCs** — they are accessible only via the panel UI, sysex preset dump, or the PolyBrute Connect software editor (which uses a private USB MIDI port, `PolyBrute VST`, with NRPN messages). This is a gap relative to e.g. the OB-X8 (which does expose individual mod parameters via NRPN). Recorded under Quirks.

| Field | Range | Notes |
|---|---|---|
| Slot source (per slot, ×64 slots) | 12 sources | **OOC.** Sources include LFO 1/2/3, Mod Env, velocity, aftertouch, Mod Wheel, Morphée X/Y/Z, Ribbon, etc. (specific source list not in section 13.1). |
| Slot destination (per slot, ×64 slots) | 32 destinations | **OOC.** |
| Slot amount (per slot, ×64 slots) | 0–127 (bipolar in mod-matrix UI) | **OOC.** |

### Voice (glide, unison/poly mode, MPE) — common-core covers Glide only
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Glide | continuous | 0–127 (raw MIDI CC 5) | rate (knob taper) | — | Portamento / glide time. Common-core. |
| Voicing mode | discrete | enum (panel + mod matrix) | mode | Mono / Unison / Poly | Voice-allocation mode (panel and patch parameter — not published as a CC in section 13.1). **Out-of-common-core (voice-allocation mode).** |
| FullTouch / poly-aftertouch mode | discrete | enum (Globals) | mode | Mono AT / Poly AT / FullTouch AT / FullTouch AT > Z / FullTouch Env > AT | The PolyBrute 12's FullTouch® MPE keyboard offers five aftertouch modes per Arturia. **Out-of-common-core (per-voice pressure / MPE source — distinguishing PolyBrute 12 feature; doesn't affect canonical params).** |

### Performance / Master / Global
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mod wheel | continuous | 0–127 (raw MIDI CC 1) | level | — | Standard MIDI Mod Wheel — assignable as a mod-matrix source. |
| Reverb level | continuous | 0–127 (raw MIDI CC 2) | level | — | Reverb effect send level. **Out-of-common-core (effects domain). Note: CC 2 is the standard MIDI Breath Controller — non-standard mapping; see Quirks.** |
| Morph | continuous | 0–127 (raw MIDI CC 3 per manual section 13.1; CC 11 per midi.guide chart — see Quirks for the discrepancy) | level | — | Morph A/B knob — cross-fades between two patch-internal sound states. **Out-of-common-core (Morph A/B / per-patch dual-state morphing).** |
| Expression pedal 1 | continuous | 0–127 (raw MIDI CC 3) | level | — | Expression-pedal-1 input. (Same CC 3 as `Morph` per midi.guide; recorded verbatim.) |
| Expression pedal 2 | continuous | 0–127 (raw MIDI CC 4) | level | — | Expression-pedal-2 input. |
| Ribbon | continuous | 0–127 (raw MIDI CC 9) | level | — | Ribbon controller position. **Out-of-common-core (per-patch ribbon controller as mod source / not a tone-generation parameter).** |
| Morphée X | continuous | 0–127 (raw MIDI CC 114) | level | — | Morphée pad X-axis (left-right touch position). **Out-of-common-core (Morphée 3-axis controller as mod source).** |
| Morphée Y | continuous | 0–127 (raw MIDI CC 115) | level | — | Morphée pad Y-axis (top-bottom touch position). **Out-of-common-core.** |
| Morphée Z | continuous | 0–127 (raw MIDI CC 89) | level | — | Morphée pad Z-axis (pressure). **Out-of-common-core.** |
| Modulation intensity | continuous | 0–127 (raw MIDI CC 13) | amount | — | Global mod-matrix intensity scaler (per Arturia panel labelling). **Out-of-common-core (mod-matrix global parameter).** |
| Delay level | continuous | 0–127 (raw MIDI CC 31) | level | — | Delay effect send. **Out-of-common-core (effects).** |
| Delay time | continuous | 0–127 (raw MIDI CC 112) | time (knob taper) | — | Delay time. **OOC.** |
| Delay regeneration | continuous | 0–127 (raw MIDI CC 113) | feedback | — | Delay feedback / regen. **OOC.** |
| Reverb time | continuous | 0–127 (raw MIDI CC 74) | time | — | Reverb decay / time. **OOC.** |
| Reverb damping | continuous | 0–127 (raw MIDI CC 76) | damping | — | Reverb HF damping. **OOC.** |
| Sequencer Rate | continuous | 0–127 (raw MIDI CC 116) | rate | — | Sequencer step rate. **Out-of-common-core (sequencer domain).** |
| Sequencer Gate | continuous | 0–127 (raw MIDI CC 118) | gate length | — | Sequencer gate length. **OOC.** |

## Modulation routing

The PolyBrute 12 has **three LFOs** (`LFO 1`, `LFO 2`, `LFO 3`), **three envelopes** (Filter, Amp, Modulation), and a free **12 × 32 modulation matrix** with up to 64 active connections per patch. The multi-LFO / three-envelope / mod-matrix topology is itself out-of-common-core.

- **LFO 1 destinations:** Configured via the **12 × 32 mod matrix** — the published manual section 13.1 / midi.guide chart does NOT expose per-destination on/off switches for LFO 1 as CCs. Common-core's small destination set (pitch, PWM, filter cutoff, VCA) is reached through mod-matrix slots, **out-of-common-core.**
- **LFO 1 shapes:** Waveform select on the panel (specific shape enumeration not published in section 13.1 as a CC — sysex / Connect-software-only). LFO 1 also has a `Phase` knob (CC 90) for note-on phase reset.
- **LFO 2 destinations:** Same as LFO 1 — configured via the mod matrix. **OOC (second LFO source / mod-matrix routing).**
- **LFO 2 features:** Adds a `Fade in` parameter (CC 92) — depth ramps up after note-on instead of starting at full depth. **OOC.**
- **LFO 3 destinations:** Same as LFO 1/2 — via mod matrix. **OOC.**
- **LFO 3 shape:** Continuous shape morph via `Curve` (CC 67) and `Symmetry` (CC 68) instead of discrete waveform select. **OOC.**
- **Filter envelope → Steiner cutoff** via `Steiner Filter envelope amount` (CC 24, bipolar) — same envelope source.
- **Filter envelope → Ladder cutoff** via `Ladder Filter envelope amount` (CC 26, bipolar) — same envelope source, independent amount per filter.
- **Filter envelope velocity** via `Filter Envelope Velocity` (CC 94).
- **Amp envelope velocity** via `Amplifier Envelope Velocity` (CC 95).
- **Modulation Envelope (DADSR) → any mod-matrix destination** — freely assignable, not panel-fixed. **OOC (third envelope routed through mod matrix).**
- **Morphée X / Y / Z, Ribbon, Mod Wheel, FullTouch poly-aftertouch** are all mod-matrix sources. **OOC.**
- **Filter FM (`Oscillator 2 to filter 1` CC 79; `Noise to filter 2` CC 80)** — audio-rate per-filter FM. **OOC.**
- **Linear FM (`VCO 2 Frequency modulation 2 > 1`, CC 77)** — VCO 2 → VCO 1 oscillator-rate FM. **OOC.**
- **Hard sync (`VCO 1 Sync`, CC 65)** — VCO 1 reset by VCO 2. **OOC.**
- **Brute Factor (`Steiner Filter Brute Factor`, CC 82) and Ladder `Distortion` (CC 85)** — Brute wave-shaping / saturation paths. **OOC.**
- **Metalizer (`VCO 1 Metalizer`, CC 70)** — wavefolding nonlinearity per oscillator. **OOC (Brute wave-shaping).**

Sources for routing: PolyBrute 12 User Manual v3.1.2 (section 13.1 *MIDI Continuous Controller assignments*; modulation-matrix description), Arturia PolyBrute 12 details page (signal-path / mod-matrix dimensions / envelope and LFO descriptions), midi.guide chart (CC-level routing parameters).

## Quirks

- **PolyBrute 12 inherits the original PolyBrute's MIDI CC assignments unchanged** — per the Arturia user-forum thread #5971, "Arturia missed the opportunity to correct the PolyBrute's MIDI specification in the PolyBrute 12 model" and preserved the same non-standard CC assignments to maintain back-compatibility with PolyBrute sequences. The midi.guide chart for the original PolyBrute is therefore the authoritative chart for the PolyBrute 12 as well, with the additions noted under "Mod-matrix dimensions" and "FullTouch / MPE" below.
- **Several CC assignments collide with the Standard MIDI Controller assignments.** Per the user-forum thread #5971: `Reverb level` is on **CC 2** (standard MIDI Breath Controller); `Steiner Filter Level` is on **CC 7** (standard MIDI Volume); `VCO 1 Sync` is on **CC 65** (standard MIDI Portamento On/Off). Hosts that send standard-named CCs (Volume, Breath, Portamento) will inadvertently affect filter level, reverb level, and oscillator sync respectively. **Recorded as a CC-naming-collision risk for hosts cross-targeting multiple devices.**
- **Morph CC discrepancy.** The Arturia user-forum thread #5327 reports that the PolyBrute 12 manual section 13.1 documents `Morph` on **CC 3**, while the midi.guide chart publishes `Morph` on **CC 11** (and `Expression pedal 1` on CC 3). Both values are reproduced in the parameter table above with this note. The discrepancy may reflect a chart staleness on midi.guide (which covers the original PolyBrute) vs an updated mapping in the PolyBrute 12 manual, or a documentation-publication discrepancy in section 13.1. Hosts targeting the PolyBrute 12 specifically should test against the manual section 13.1 table directly.
- **The dual-filter Series/Parallel routing is a continuously-variable blend, not a discrete switch.** Per Arturia: "Filter Routing: Series, Parallel, or Continuous blend modes" — `Series/Parallel` (CC 86) sweeps continuously across the routing topology rather than selecting a discrete mode. The schema's common-core collapses to a single LP filter; **the dual-filter morph is treated as out-of-common-core.**
- **Brute wave-shaping is per-oscillator nonlinearity, not a global drive.** The `VCO 1 Metalizer` (CC 70) is wavefolding on VCO 1 specifically; the `Steiner Filter Brute Factor` (CC 82) is feedback-style saturation on the Steiner-Parker filter; the `Ladder Filter Distortion` (CC 85) is asymmetrical clipping on the Ladder filter input. These three saturators are spread across the signal chain (per-osc, per-filter) rather than collapsed into a single drive parameter. **All three are out-of-common-core (Brute wave-shaping nonlinearities).**
- **Poly-aftertouch / FullTouch® MPE is a distinguishing feature of the PolyBrute 12** but does not affect canonical tone-generation parameters per the task brief. The five aftertouch modes (Mono AT / Poly AT / FullTouch AT / FullTouch AT > Z / FullTouch Env > AT) are accessible only as mod-matrix sources and as MPE-channel data sent over MIDI. **Out-of-common-core.**
- **The 12 × 32 mod matrix is not exposed as CCs in section 13.1.** Per the manual section 13.1 / midi.guide chart, the mod-matrix slot source / destination / amount fields are panel-UI / sysex-only. The published CC chart exposes only the underlying mod sources (LFO 1/2/3, Mod Env, Mod Wheel, Morphée, Ribbon — via their continuous-controller knobs) and fixed-routing panel parameters (e.g. `Filter envelope amount`), not the slot assignments themselves. **Out-of-common-core (the canonical schema's common-core has only fixed per-LFO destination switches, not a free assignable mod matrix); also a documentation gap for hosts that want to drive mod-matrix slots over MIDI in real-time.**
- **No NRPN parameter numbers are published in section 13.1 of the public manual** — per the Arturia user-forum thread #5327, the PolyBrute 12 transmits NRPN messages **only on the `PolyBrute VST` USB MIDI port**, a private port reserved for the PolyBrute Connect software editor. The standard `PolyBrute MIDI` USB port does not expose Morphée or Ribbon as mappable CCs (only as outgoing CC 89 / 114 / 115 / 9), and as of January 2026 Arturia has not published a detailed 14-bit NRPN specification despite user requests. This is a gap relative to the OB-X8 / Prophet-5 / Prophet-6 chart conventions. The original PolyBrute's NRPNs (Ribbon NRPN 4103; Morphée X / Y / Z NRPN 4113 / 4114 / 4115 — community-documented per the Arturia user-forum thread #4296) are **not officially documented for the PolyBrute 12**, though the Connect-software port likely reuses the same NRPN scheme.
- **Incoming Morphée / Ribbon / Morph CC is currently not received by the engine** — per the Arturia user-forum thread #5971: "External incoming CC114/115/89 should affect Morphee they are not. External incoming Cc 9 should affect ribbon, its not currently." The CC values are sent outbound (recorded into a DAW correctly) but the PolyBrute 12 does not respond to them when played back into the device. **This is an open firmware bug as of forum discussion — recorded for hosts that need to play back recorded automation.**
- **PolyBrute 12 vs original PolyBrute (2020).** Synthesis architecture is the same per Sound on Sound ("much of the PolyBrute 12 is the same as its predecessor"). Differences: (a) **polyphony doubles** from 6 to 12 voices; (b) the **FullTouch® MPE keyboard** with five aftertouch modes is new; (c) **LFO 3** (with Curve/Symmetry continuous waveshape morph) is documented in PolyBrute 12 / midi.guide as a third LFO — the original PolyBrute also has three LFOs per midi.guide so this is not a 12-vs-original difference; (d) the **PolyBrute Connect** companion software is new for the PolyBrute 12. The MIDI CC table is shared (per user-forum thread #5971). **Hosts cross-targeting both devices can rely on the same CC chart but must account for the polyphony and MPE differences.**
- **No on-board sample-rate / oversampling mode parameters are exposed via CC.** Globals are panel-UI and Connect-software only.
- **Many Globals are not exposed as CCs.** MIDI channel, MIDI cable mode, voicing mode (Mono/Unison/Poly), playing mode (Single/Split/Layer), aftertouch curve, and similar global / configuration parameters are accessible only via the panel UI or sysex / Connect-software. **Out-of-common-core (system / configuration domain).**
- **The PolyBrute 12's three "stereo digital effects" sections (Modulation, Delay, Reverb) are post-engine** — Modulation effects (chorus / phaser / flanger / ring modulation), Delay (9 algorithms incl. BBD / digital), Reverb (9 algorithms incl. hall / plate / spring / shimmer). Their CC-addressable parameters (`Modulation intensity` CC 13, `Delay level` CC 31, `Delay time` CC 112, `Delay regeneration` CC 113, `Reverb level` CC 2, `Reverb time` CC 74, `Reverb damping` CC 76) are recorded for completeness but are **out-of-common-core (effects domain)**.
