# Novation Summit — Parameter Survey

**Manufacturer:** Novation (Focusrite Group)
**Year:** 2019
**Engine type:** hybrid (Numerically Controlled Oscillators — Novation's "New Oxford Oscillators" / NCOs — feeding an analog signal path: analog dual filters, analog VCAs, three stages of analog distortion per voice)
**Polyphony:** 16 voices total, **bi-timbral** (Single mode = up to 16 voices on one Patch; Multi mode splits 8 + 8 across Part A and Part B with three sub-modes — Layer / Split / Dual)
**Sources:**
- Novation, *Peak User Guide* (v1.2 firmware, current at the time of writing) — covers the shared Peak/Summit synthesis engine (NCO architecture, mixer, dual-filter section, three envelopes, four LFOs, 16-slot mod matrix, FX matrix). The Summit shares the Peak's per-voice synth engine essentially unchanged: https://files.kraftmusic.com/media/ownersmanual/Novation_Peak_User_Guide.pdf
- Novation, *Summit User Guide* (Focusrite-hosted PDF, English original, v1.0 / firmware 1.0): https://files.kraftmusic.com/media/ownersmanual/Novation_Summit_User_Guide.pdf
- Novation, *Summit Get Started Guide* (v1.0, Focusrite CDN — establishes Summit-specific bi-timbral architecture: Single / Multi modes, Multi sub-modes Layer / Split / Dual, Patch A + Patch B, Part A / Part B, Animate buttons): https://fael-downloads-prod.focusrite.com/customer/prod/s3fs-public/downloads/Summit%20Get%20Started%20Guide%201.0.pdf
- Novation, *Summit & Peak v2.0 Firmware Update Addendum* (Focusrite-hosted) — adds firmware-2.0 features layered on top of the original engine: https://fael-downloads-prod.focusrite.com/customer/prod/downloads/Summit%20&%20Peak%202.0%20Firmware%20Update%20Addendum%20V2%20English%20-%20EN.pdf
- midi.guide community CC + NRPN database, *Novation Summit and Peak* (cross-check of CC numbers; the Summit and Peak share the per-engine CC map verbatim — see Quirks): https://midi.guide/d/novation/summit-and-peak/

No `keyboards-mcp` cross-check is available for this device — sourcing is web-only. **Summit vs Peak documentation:** the Summit's per-engine synth architecture (oscillators, mixer, filter, envelopes, LFOs, mod matrix, FX) is essentially the Peak engine duplicated and addressed bi-timbrally; per the Summit Get Started Guide and the midi.guide CC reference (which is published as a single combined "Summit and Peak" map), the Peak User Guide v1.2 is the authoritative source for the per-engine parameters and the Summit User Guide layers Summit-specific bi-timbral controls (Single / Multi mode, Patch A + Patch B, Part A / Part B, MIDI channel per part, Animate per part, separate stereo audio outputs) on top. Where the two guides diverge, this survey describes the Summit and notes the Peak dependency. **The Peak's "v1.2 firmware" engine documentation is the underlying spec for both devices' per-voice engine** — the Summit ships at firmware 1.0 against the same engine parameter set.

**Engine-scope caveat.** Per the Peak User Guide intro, the NCOs are described by Novation as "FPGA-based Numerically Controlled Oscillators running at 24 MHz [generating] waveforms indistinguishable from those produced by analogue oscillators" — the basic waveforms (sine, triangle, sawtooth, pulse) are subtractive-compatible and behave like virtual-analog VCOs. The same oscillator block also exposes 60 **wavetables** ("More" / `BS sine`, `BS Tri`, etc.) that move the engine into wavetable / hybrid territory. **The basic (non-wavetable) waveform set is in scope for the subtractive ontology**; the **wavetable algorithm set is recorded but flagged out-of-common-core (wavetable / multi-shape / non-subtractive synthesis)**. This is consistent with the Minilogue XD survey's MULTI ENGINE handling: capture the parameter, mark out-of-scope.

## Signal path

Three NCO oscillators per voice (`Osc 1`, `Osc 2`, `Osc 3`) — each generating Sine, Triangle, Sawtooth, or Pulse waveforms (selected by the per-osc `Wave` switch), plus 60 selectable wavetables when `Wave = More` (out-of-common-core wavetable algorithms). Each oscillator carries a continuously-variable `Shape` knob with three modulation sources (Mod Envelope 1, LFO 1, or Manual — selected by the `Source` switch); a panel-knob `Shape Amount` parameter; per-osc `Pitch` (Range / Coarse / Fine), per-osc `Vsync` (virtual-oscillator hard-sync amount, 0–127 — drives the slave oscillator from a virtual master whose frequency is offset by the `Vsync` parameter); and per-osc `Mod 1`, `Mod 2`, `LFO 1`, `LFO 2` modulation depths into pitch and shape. **Vsync (virtual oscillator hard-sync) and the `More` wavetables are out-of-common-core**. There is no dedicated panel sub-oscillator on the Summit / Peak — sub-octave behavior is reached by transposing one of the three NCOs down via its `Range` (16' / 32') or `Coarse` knob.

The `SawDense` and `DenseDet` parameters in the Per-Oscillator menu add up to seven additional copies of a sawtooth oscillator (with detuning) inside a single voice — Novation's documentation cites this as an alternative to Unison-mode voice-stacking that does not consume polyphony. **`SawDense` / `DenseDet` are out-of-common-core (per-osc oscillator-stacking, similar in spirit to the Prophet-5 "Slop" or Moog "Drift" but with discrete copies rather than detuning).**

Three pairs of audio-rate **Frequency Modulation** paths route each oscillator into another oscillator's frequency (`FM Osc1 > Osc2`, `FM Osc2 > Osc3`, `FM Osc3 > Osc1`) — these provide cross-oscillator FM synthesis on top of the basic subtractive paths. **All three FM paths are out-of-common-core (oscillator-rate FM / cross-modulation).**

The mixer (5 inputs) sums `Osc 1`, `Osc 2`, `Osc 3`, the **Ring Modulator** output (Osc 1 × Osc 2), and the **Noise** generator into the analog filter section. **The Ring Modulator is out-of-common-core (cross-oscillator multiplier).**

The filter is a single analog **multi-mode** filter per voice with a 3-position `Shape` switch selecting **Low-Pass (LP)**, **Band-Pass (BP)**, or **High-Pass (HP)** mode, plus a continuously-variable slope (12 dB/oct ↔ 24 dB/oct) on the LP path. **Low-Pass is the common-core mode; HP and BP modes, the variable-slope morph, the pre-filter `Filter Overdrive` analog distortion stage, the post-filter `Filter Post Drv` analog distortion stage, and per-voice `Filter Diverge` parameter are all out-of-common-core (multi-mode filter / variable-slope / drive / per-voice analog drift simulation).** Filter `Cutoff`, `Resonance`, and `Filter Track` (keyboard tracking) are common-core. The filter envelope amount is set per-source via `Source` switch + amount knobs (Mod Envelope 1, Amp Envelope, LFO 1 — see Modulation routing).

After the filter, audio passes into the analog VCA. **Three envelope generators** drive the per-voice engine: a dedicated **Amp Envelope** with five phases (AHDSR — Attack / Hold / Decay / Sustain / Release; Hold accessible via the Envelope menu), four panel sliders (A / D / S / R) hardwired to the VCA; and **two Mod Envelopes** (`Mod Env 1` and `Mod Env 2`) with the same AHDSR five-phase structure, sharing one set of four sliders via a `Select` button. The AHD phases can be looped repeatedly (per Novation's spec: "AHD envelope phases can be looped repeatedly"). **The third envelope (Mod Env 2) and the AHD-loop / repeat modes are out-of-common-core (third envelope / envelope loop).**

The Summit / Peak carries **four LFOs**: `LFO 1` and `LFO 2` are exposed on the panel (each with `Waveform`, `Range` High/Low/Sync, `Rate`, `Fade Time` controls and dedicated routing on the panel — LFO 1 to filter / oscillator Shape destinations, LFO 2 to oscillator pitch destinations); `LFO 3` and `LFO 4` are reachable via menu only and are addressable through the Modulation Matrix. **LFO 1 is the common-core LFO; LFO 2, LFO 3, and LFO 4 are out-of-common-core (additional LFO sources).**

The 16-slot **Modulation Matrix** (with two sources per slot — i.e. 16 routing slots × 2 sources = 32 source assignments × 16 destinations) is the routing primary for any modulation outside the panel-fixed paths. A separate FX Modulation Matrix (4 additional slots) routes mod sources to FX parameters. **The mod matrix is out-of-common-core (free assignable matrix beyond fixed per-LFO destination switches). The FX mod matrix is also out-of-common-core (FX-routing extension).**

The Summit / Peak also includes Glide (portamento with dedicated time control), per-voice Unison (1, 2, 3, 4, 8 voices), Unison Detune, Unison Spread (stereo placement), Pre-Glide (chromatic pre-glide ±12 semitones), Polyphony Mode (Mono / MonoLG / Mono2 / Poly / Poly2), an Arpeggiator, two **Animate** buttons (per-Part on Summit) for live performance spot effects, and a Power-Up FX section (Distortion / Chorus / Delay / Reverb). **Effects are post-engine and out of scope**, except where they share CC numbers with subtractive parameters (none do on the Summit / Peak — the FX CCs are in a separate range, see Master / FX section).

**Summit-specific bi-timbral architecture** (per the Get Started Guide): the Summit has two complete copies of the Peak engine (Part A and Part B) sharing 16 voices total. **Single mode** allocates all 16 voices to one Patch (A or B). **Multi mode** divides voices 8 + 8 between two simultaneously-active Parts, with three sub-modes:
- **Layer**: both Parts play across the keyboard simultaneously (stacked, voices split 8 + 8).
- **Split**: keyboard divided into two zones, each triggering a different Part (each zone gets up to 8 voices).
- **Dual**: each Part responds on a separate MIDI channel (independent multi-timbral).

Patch A and Patch B are per-Part patch-storage locations; in Single mode only the active Patch is used. **All Multi-mode features (Layer / Split / Dual, Part A / Part B, per-part MIDI channel, per-part Animate) are out-of-common-core (bi-timbral split / layer / multi-timbral mode — the canonical schema's common-core covers a single mono-timbral Patch).**

## Parameters

All ranges shown as `0–127 (raw MIDI CC)` are 7-bit MIDI Continuous Controllers per the midi.guide *Novation Summit and Peak* CC reference. The Summit / Peak engine additionally supports **NRPN** with a documented "two CCs per parameter" trick that yields 0–16383 underlying resolution for many parameters (per midi.guide: "MOD MX1-MX16, FX MX1-MX8 ... [0–16383] across various MSB values"). The CC pathway is quantized to 128 steps with a hardware-defined non-linear taper (knob taper). **The Summit and Peak share the same CC map verbatim** — the midi.guide reference is published as a single combined chart titled "Novation Summit and Peak" and the Peak User Guide's appendix lists the same parameter numbers as the Summit User Guide. Per-Part addressing on the Summit is via the per-part MIDI channel (Multi: Dual mode) — the same CC number sent on Part A's channel addresses Part A's engine, and likewise for Part B.

### Oscillator 1
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 1 Range | discrete | 0–127 (raw MIDI CC 3) | enum | (octave-foot range select) | Osc 1 octave / range select. Foot-mark style octave selector (per Peak User Guide oscillator section). |
| Osc 1 Coarse | continuous | 0–127 (raw MIDI CC 14) | semitones (knob taper, bipolar — center = 64) | — | Osc 1 coarse pitch offset. Bipolar; the CC value collapses bipolar onto 0–127 with 64 = center. |
| Osc 1 Fine | continuous | 0–127 (raw MIDI CC 15) | cents (knob taper, bipolar — center = 64) | — | Osc 1 fine tune. |
| Osc 1 Shape | continuous | 0–127 (raw MIDI CC 12) | morph (knob taper) | — | Osc 1 shape amount. Continuously-variable waveshape (e.g. pulse-width on Pulse, harmonic content morph on wavetables). |
| Osc 1 Mod 1 Shape | continuous | 0–127 (raw MIDI CC 119) | depth | — | Mod Env 1 → Osc 1 Shape modulation depth. **Out-of-common-core (panel-fixed routing beyond LFO 1 destination set — captured in Modulation routing).** |
| Osc 1 LFO 1 Shape | continuous | 0–127 (raw MIDI CC 33) | depth | — | LFO 1 → Osc 1 Shape modulation depth (panel knob). The common-core LFO destination set; recorded under Modulation routing. |
| Osc 1 Mod 2 Pitch | continuous | 0–127 (raw MIDI CC 9) | depth | — | Mod Env 2 → Osc 1 pitch modulation depth. **Out-of-common-core (Mod Env 2 is the third envelope).** |
| Osc 1 LFO 2 Pitch | continuous | 0–127 (raw MIDI CC 16) | depth | — | LFO 2 → Osc 1 pitch modulation depth (panel knob). **Out-of-common-core (LFO 2).** |
| Osc 1 Vsync | continuous | 0–127 (raw MIDI CC 34) | sync amount | — | Virtual-oscillator hard-sync amount. The virtual oscillator is silent; its frequency offset re-triggers the audible Osc 1 (Peak User Guide p.~25). When `Vsync` is a multiple of 16, the virtual frequency is a musical harmonic of the main oscillator — values between multiples produce discordant overtones. **Out-of-common-core (oscillator hard sync).** |

### Oscillator 2
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 2 Range | discrete | 0–127 (raw MIDI CC 37) | enum | (octave-foot range select) | Osc 2 octave / range. |
| Osc 2 Coarse | continuous | 0–127 (raw MIDI CC 17) | semitones (knob taper, bipolar — center = 64) | — | Osc 2 coarse pitch offset. |
| Osc 2 Fine | continuous | 0–127 (raw MIDI CC 18) | cents (knob taper, bipolar — center = 64) | — | Osc 2 fine tune. |
| Osc 2 Shape | continuous | 0–127 (raw MIDI CC 39) | morph (knob taper) | — | Osc 2 shape amount. |
| Osc 2 Mod 1 Shape | continuous | 0–127 (raw MIDI CC 40) | depth | — | Mod Env 1 → Osc 2 Shape depth. |
| Osc 2 LFO 1 Shape | continuous | 0–127 (raw MIDI CC 41) | depth | — | LFO 1 → Osc 2 Shape depth. |
| Osc 2 Mod 2 Pitch | continuous | 0–127 (raw MIDI CC 38) | depth | — | Mod Env 2 → Osc 2 pitch depth. **OOC.** |
| Osc 2 LFO 2 Pitch | continuous | 0–127 (raw MIDI CC 19) | depth | — | LFO 2 → Osc 2 pitch depth. **OOC.** |
| Osc 2 Vsync | continuous | 0–127 (raw MIDI CC 42) | sync amount | — | Osc 2 virtual hard sync. **OOC.** |

### Oscillator 3 (out-of-common-core — third oscillator)
**The entire Oscillator 3 section is out-of-common-core (the canonical schema's common-core covers two oscillators only — the Prophet-6, JUNO-X, OB-X8, Prophet-5, Moog Muse, PolyBrute 12, and Minilogue XD all have ≤ 2 main subtractive oscillators per voice; the Summit / Peak's third NCO is a Summit-specific extension).** Recorded for device completeness.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 3 Range | discrete | 0–127 (raw MIDI CC 65) | enum | (octave-foot range select) | Osc 3 octave / range. **OOC. (Note: CC 65 is the standard MIDI Portamento On/Off controller — this is a non-standard mapping; see Quirks.)** |
| Osc 3 Coarse | continuous | 0–127 (raw MIDI CC 20) | semitones (knob taper, bipolar — center = 64) | — | Osc 3 coarse pitch. **OOC.** |
| Osc 3 Fine | continuous | 0–127 (raw MIDI CC 21) | cents (knob taper, bipolar — center = 64) | — | Osc 3 fine tune. **OOC.** |
| Osc 3 Shape | continuous | 0–127 (raw MIDI CC 71) | morph (knob taper) | — | Osc 3 shape amount. **OOC.** |
| Osc 3 Mod 1 Pitch | continuous | 0–127 (raw MIDI CC 72) | depth | — | Mod Env 1 → Osc 3 pitch depth. **OOC.** |
| Osc 3 LFO 1 Shape | continuous | 0–127 (raw MIDI CC 73) | depth | — | LFO 1 → Osc 3 Shape depth. **OOC.** |
| Osc 3 Mod 2 Pitch | continuous | 0–127 (raw MIDI CC 43) | depth | — | Mod Env 2 → Osc 3 pitch depth. **OOC.** |
| Osc 3 LFO 2 Pitch | continuous | 0–127 (raw MIDI CC 22) | depth | — | LFO 2 → Osc 3 pitch depth. **OOC.** |
| Osc 3 Vsync | continuous | 0–127 (raw MIDI CC 44) | sync amount | — | Osc 3 virtual hard sync. **OOC.** |
| Osc 3 Filter | continuous | 0–127 (raw MIDI CC 76) | depth | — | Osc 3 → filter cutoff modulation depth (Osc 3 used as a modulation source into filter cutoff at audio rate, similar to the PolyBrute 12's `Oscillator 2 to filter 1` Filter FM). **OOC (oscillator-rate filter modulation).** |

### Sub
The Summit / Peak has **no dedicated panel sub-oscillator**. Sub-octave behavior is reached by transposing one of the three NCOs (typically Osc 3) down via its `Range` switch (e.g. 32') or `Coarse` knob. There is no separate sub-osc level / waveshape parameter to record. (The canonical schema's "sub osc level" parameter does not have a clean Summit / Peak analogue — flag accordingly when mapping.)

### Mixer
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Osc 1 Mix | continuous | 0–127 (raw MIDI CC 23) | level | — | Osc 1 level into the filter section. |
| Osc 2 Mix | continuous | 0–127 (raw MIDI CC 24) | level | — | Osc 2 level into the filter section. |
| Osc 3 Mix | continuous | 0–127 (raw MIDI CC 25) | level | — | Osc 3 level into the filter section. **Out-of-common-core if the canonical mixer has only Osc 1 + Osc 2 + Sub + Noise slots.** |
| Ring Mod Mix | continuous | 0–127 (raw MIDI CC 26) | level | — | Ring Modulator (Osc 1 × Osc 2) output level into the filter section. **Out-of-common-core (cross-oscillator multiplier).** |
| Noise Mix | continuous | 0–127 (raw MIDI CC 27) | level | — | Noise generator level into the filter section. White noise. |

### Filter (analog multi-mode — LP common-core; HP / BP / variable-slope / drive OOC)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Cutoff | continuous | 0–127 (raw MIDI CC 29) | cutoff (knob taper) | — | Filter cutoff frequency. Common-core when `Filter Shape` = LP. |
| Resonance | continuous | 0–127 (raw MIDI CC 79) | resonance | — | Filter resonance. The Peak User Guide notes that high resonance settings cause the filter to self-oscillate (sine wave at the cutoff frequency). |
| Filter Track | continuous | 0–127 (raw MIDI CC 75) | tracking | — | Keyboard tracking amount into filter cutoff. |
| Filter Overdrive | continuous | 0–127 (raw MIDI CC 80) | drive | — | Pre-filter analog overdrive (one of three stages of analog distortion per voice — Peak User Guide). **Out-of-common-core (filter drive / saturation — not in canonical common-core).** |
| Filter Post Drv | continuous | 0–127 (raw MIDI CC 36) | drive | — | Post-filter analog drive. **Out-of-common-core.** |

The 3-position **`Filter Shape`** switch (LP / BP / HP) and the **continuous slope morph** (12 dB/oct ↔ 24 dB/oct on the LP path) are addressed via NRPN / SysEx, not by a top-level CC in the published midi.guide chart. Recorded under Modulation routing / Quirks. **Both are out-of-common-core (multi-mode filter and variable-slope).**

### Filter envelope (driven by Mod Env 1 and/or Amp Env via Source switch)
The Summit / Peak does **not** have a dedicated "filter envelope" — instead, the `Source` switch in the Filter section selects which of `Mod Env 1`, `Amp Env`, or both is routed into filter cutoff (Peak User Guide, Filter Source: "selects whether the filter frequency is to be varied by Mod Envelope 1 (Mod Env 1) and/or the Amp Envelope (Amp Env): note that these two sources may be combined"). The shape of the envelope-into-cutoff modulation is therefore the Mod Env 1 ADSR (or the Amp Env ADSR, or both summed). The depth of this routing is set by `Amp Filter` (Amp Env → Filter Cutoff) and `Mod 1 Filter` (Mod Env 1 → Filter Cutoff) panel knobs. **The Mod-Env-1-as-filter-envelope role is the closest device analogue to the canonical "filter envelope"; the dual-source Filter Source switch (Mod Env 1 + Amp Env summing into cutoff) and the Amp-Env-into-Cutoff path are out-of-common-core (the canonical schema typically fixes "filter envelope" and "amp envelope" as two independent dedicated envelopes).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mod 1 Filter | continuous | 0–127 (raw MIDI CC 78) | bipolar amount | — | Mod Env 1 → Filter Cutoff modulation depth. The "filter envelope amount" knob (in Mod-Env-1-as-filter-envelope role). Bipolar on the panel knob (negative = inverted envelope into cutoff); the CC value collapses bipolar onto 0–127 with 64 = center. |
| Amp Filter | continuous | 0–127 (raw MIDI CC 77) | bipolar amount | — | Amp Env → Filter Cutoff modulation depth. **Out-of-common-core (Amp-Env-as-filter-envelope dual-role routing).** |
| Mod 1 Attack | continuous | 0–127 (raw MIDI CC 90) | time (knob taper) | — | Mod Env 1 attack time (in Mod-Env-1-as-filter-envelope role). |
| Mod 1 Decay | continuous | 0–127 (raw MIDI CC 91) | time (knob taper) | — | Mod Env 1 decay time. |
| Mod 1 Sustain | continuous | 0–127 (raw MIDI CC 92) | level | — | Mod Env 1 sustain level. |
| Mod 1 Release | continuous | 0–127 (raw MIDI CC 93) | time (knob taper) | — | Mod Env 1 release time. |

The Mod Env 1 also has a `Hold` parameter (the H in AHDSR) accessible via the Envelopes menu — Hold is **not** addressable via top-level CC. The AHD repeat / loop modes are also menu-only. **Hold and AHD-loop are out-of-common-core (extra envelope phase; envelope loop).**

### Amp envelope
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Amp Attack | continuous | 0–127 (raw MIDI CC 86) | time (knob taper) | — | Amp Envelope attack time. |
| Amp Decay | continuous | 0–127 (raw MIDI CC 87) | time (knob taper) | — | Amp Envelope decay time. |
| Amp Sustain | continuous | 0–127 (raw MIDI CC 88) | level | — | Amp Envelope sustain level. |
| Amp Release | continuous | 0–127 (raw MIDI CC 89) | time (knob taper) | — | Amp Envelope release time. |

The Amp Envelope is hardwired to the VCA at full depth (per Peak User Guide). It also has a menu-accessible `Hold` parameter (the H in AHDSR) — not addressable via top-level CC. **Hold is out-of-common-core (extra envelope phase). The AHD-loop / repeat modes (also menu-only) are out-of-common-core (envelope loop).**

### Third envelope (Mod Env 2 — out-of-common-core)
**The entire Mod Env 2 section is out-of-common-core (third envelope).** Mod Env 2 shares the same AHDSR five-phase structure as Mod Env 1 and Amp Env, and its slider controls share hardware with Mod Env 1 (selected via the `Select` button between Mod Env 1 and Mod Env 2). Routing Mod Env 2 anywhere requires the Modulation Matrix; the panel `Mod 2 Pitch` knobs (CCs 9 / 38 / 43 — already listed under each Osc) are the per-osc panel-fixed routings.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mod 2 Attack | continuous | 0–127 (raw MIDI CC 94) | time (knob taper) | — | Mod Env 2 attack time. **OOC.** |
| Mod 2 Decay | continuous | 0–127 (raw MIDI CC 95) | time (knob taper) | — | Mod Env 2 decay time. **OOC.** |
| Mod 2 Sustain | continuous | 0–127 (raw MIDI CC 117) | level | — | Mod Env 2 sustain level. **OOC.** |
| Mod 2 Release | continuous | 0–127 (raw MIDI CC 103) | time (knob taper) | — | Mod Env 2 release time. **OOC.** |

### LFO 1 (panel LFO, common-core)
LFO 1 has panel-fixed routing into a small destination set (filter cutoff via `LFO 1 Filter`, plus per-oscillator Shape via `Osc N LFO 1 Shape` already listed under each Osc). Richer destinations are reachable via the Modulation Matrix only.

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 1 Filter | continuous | 0–127 (raw MIDI CC 28) | depth | — | LFO 1 → Filter Cutoff modulation depth (panel knob). Common-core LFO destination. |
| LFO 1 Rate | continuous | 0–127 (raw MIDI CC 30) | rate (knob taper) | — | LFO 1 frequency. |
| LFO 1 Sync Rate | continuous | 0–127 (raw MIDI CC 81) | rate / note-division enum | — | LFO 1 sync rate (when `LFO 1 Range` = Sync, the rate knob selects clock-divisions; this CC addresses the synced-mode rate). |
| LFO 1 Fade | continuous | 0–127 (raw MIDI CC 82) | time (knob taper) | — | LFO 1 fade-in time (ramp the LFO depth up over a programmed duration after note-on). |

LFO 1 `Waveform` and `Range` (High / Low / Sync) switches are addressed via NRPN / SysEx and not exposed as top-level CCs in the published midi.guide chart. The Peak User Guide lists the LFO waveforms as Sine, Triangle, Sawtooth, Square, Sample-and-Hold, plus additional waveform types reachable from the LFO menu (per the Peak feature spec). **LFO 1 Waveform is recorded under Modulation routing.**

### LFO 2 (out-of-common-core — second LFO)
**The entire LFO 2 section is out-of-common-core (additional LFO).**

| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| LFO 2 Rate | continuous | 0–127 (raw MIDI CC 31) | rate (knob taper) | — | LFO 2 frequency. **OOC.** |
| LFO 2 Range | discrete | 0–127 (raw MIDI CC 83) | enum | High / Low / Sync | LFO 2 range select. **OOC.** |
| LFO 2 Sync Rate | continuous | 0–127 (raw MIDI CC 84) | rate / note-division enum | — | LFO 2 sync-mode rate. **OOC.** |
| LFO 2 Fade | continuous | 0–127 (raw MIDI CC 85) | time (knob taper) | — | LFO 2 fade-in time. **OOC.** |

**LFO 3 and LFO 4** are menu-only and reachable through the Modulation Matrix; there are no dedicated CC numbers for their rate / fade / waveform on the panel, and per midi.guide they are addressed via the NRPN-only modulation-matrix slot parameters (MOD MX1–MX16). **OOC.**

### Voice (unison, glide, polyphony mode)
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Glide Time | continuous | 0–127 (raw MIDI CC 5) | time (knob taper) | — | Portamento (glide) time. |
| Glide On | toggle | 0–127 (raw MIDI CC 35) | enum | Off / On | Glide on/off. (Note: this is a non-standard mapping — CC 35 is not the MIDI standard CC for glide on/off, which is CC 65 — but the Summit / Peak uses CC 35 here and CC 65 for Osc 3 Range; see Quirks.) |

`Unison` (1 / 2 / 3 / 4 / 8 voices), `UniDeTune` (0–127), `UniSpread` (0–127), `Mode` (Mono / MonoLG / Mono2 / Poly / Poly2), `PreGlide` (Off / −12 to +12 semitones), `PatchLevel`, `FltPostDrv`, `FltDiverge` are addressed via NRPN / SysEx (the Voice menu in the Peak User Guide) and are **not** exposed as top-level CCs in the published midi.guide chart. Recorded under Quirks.

**Common-core covers the `Poly` mode and `Unison = 1`; all other Polyphony Mode values (Mono / MonoLG / Mono2 / Poly2) and Unison ≥ 2 are out-of-common-core (voice-allocation modes beyond mono and poly-1).**

### Master / FX / performance
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| Mod Wheel | continuous | 0–127 (raw MIDI CC 1) | level | — | Standard performance mod wheel. |
| Distortion Level | continuous | 0–127 (raw MIDI CC 104) | level | — | FX section: distortion send / level. **Out-of-scope (post-engine FX).** |
| Chorus Level | continuous | 0–127 (raw MIDI CC 105) | level | — | FX: chorus level. **Out-of-scope.** |
| Chorus Rate | continuous | 0–127 (raw MIDI CC 118) | rate | — | FX: chorus rate. **Out-of-scope.** |
| Chorus Feedback | continuous | 0–127 (raw MIDI CC 107) | feedback | — | FX: chorus feedback. **Out-of-scope.** |
| Delay Level | continuous | 0–127 (raw MIDI CC 108) | level | — | FX: delay level. **Out-of-scope.** |
| Delay Time | continuous | 0–127 (raw MIDI CC 109) | time | — | FX: delay time. **Out-of-scope.** |
| Delay Feedback | continuous | 0–127 (raw MIDI CC 110) | feedback | — | FX: delay feedback. **Out-of-scope.** |
| Reverb Level | continuous | 0–127 (raw MIDI CC 112) | level | — | FX: reverb level. **Out-of-scope.** |
| Reverb Time | continuous | 0–127 (raw MIDI CC 113) | time | — | FX: reverb time. **Out-of-scope.** |
| Animate 1 Hold | toggle | 0–127 (raw MIDI CC 114) | enum | Off / On | Animate 1 button hold-state (locks Animate effect). **Out-of-common-core (live-performance spot-effect button — per-Part on Summit).** |
| Animate 2 Hold | toggle | 0–127 (raw MIDI CC 115) | enum | Off / On | Animate 2 button hold-state. **OOC.** |
| Arp Clock Gate | continuous | 0–127 (raw MIDI CC 116) | gate | — | Arpeggiator clock gate length. **Out-of-scope (arpeggiator / sequencer domain).** |

NRPN-only parameters (per midi.guide MSB/LSB pairs): wavetable selection per Osc (`Osc N WaveMore` — selects which of 60 wavetables when `Wave = More`), the 16 mod matrix slots (MOD MX1–MX16, depths 0–16383), the 8 FX matrix slots (FX MX1–MX8, depths 0–16383), oscillator and filter additional menu parameters (Filter Shape switch LP/BP/HP, Filter Slope morph 12/24 dB), `SawDense` / `DenseDet`, `Diverge` / `Drift` / `KeySync`, Voice menu (`Unison` / `UniDeTune` / `UniSpread` / `Mode` / `PreGlide` / `PatchLevel` / `FltPostDrv` / `FltDiverge`), Envelope menu (Hold per envelope, AHD loop modes), LFO menu (waveform, range, key sync, voice sync, additional LFO 3 / LFO 4), Tuning Table, Bend Range. Recorded under Modulation routing / Quirks. **All NRPN-only mod-matrix and FX-matrix parameters are out-of-common-core.**

### Summit-specific bi-timbral controls (out-of-common-core)
**All Summit-specific bi-timbral parameters are out-of-common-core (the canonical schema's common-core covers a single mono-timbral patch).** Per the Summit Get Started Guide:

| Device-native parameter | Type | Description |
|---|---|---|
| Single / Multi mode | discrete enum | Top-level voice-allocation mode: Single (one Patch, up to 16 voices) vs Multi (two Parts, voices split 8 + 8). **OOC.** |
| Multi sub-mode | discrete enum (Layer / Split / Dual) | Multi-mode topology: Layer (both Parts play across the keyboard), Split (keyboard divided into two zones), Dual (each Part on a separate MIDI channel). **OOC.** |
| Patch A | program-select | Patch loaded into Part A. **OOC.** |
| Patch B | program-select | Patch loaded into Part B. **OOC.** |
| Part A MIDI channel | discrete | MIDI channel listened on by Part A (Multi: Dual mode). **OOC.** |
| Part B MIDI channel | discrete | MIDI channel listened on by Part B (Multi: Dual mode). **OOC.** |
| Split Point | discrete (note number) | Keyboard split point in Multi: Split mode. **OOC.** |
| Animate 1 / Animate 2 (per-Part) | toggle | Per-Part Animate buttons for live-performance spot effects. **OOC.** |

These are addressed via NRPN / SysEx through the Settings menu — no top-level CCs are published for the Summit-specific bi-timbral controls in the midi.guide chart (the chart is shared with the Peak, which does not have bi-timbral capability).

## Modulation routing

The Summit / Peak's modulation routing has two layers: a small set of **panel-fixed routings** (LFO 1 / LFO 2 / Mod Env 1 / Mod Env 2 / Amp Env into a fixed destination set on the panel via dedicated knobs and Source switches) and a free-assignable **16-slot Modulation Matrix** (the routing primary for any modulation outside the panel-fixed paths). The mod matrix is the device's general-purpose routing — and is out-of-common-core.

**Panel-fixed routings (verbatim from the Peak User Guide and the per-Osc / Filter / Mixer panel knobs):**

- **LFO 1 destinations (panel-fixed):** `Osc 1 Shape` (per-osc depth knob, CC 33), `Osc 2 Shape` (CC 41), `Osc 3 Shape` (CC 73), `Filter Cutoff` (panel `LFO 1 Filter`, CC 28). LFO 1 is the **common-core LFO destination subset**: filter cutoff + oscillator shape (subset of the canonical schema's expected LFO destinations). Each destination has its own depth knob — multiple destinations may be enabled simultaneously (the depth knobs operate independently).
- **LFO 2 destinations (panel-fixed) — OOC:** `Osc 1 Pitch` (CC 16), `Osc 2 Pitch` (CC 19), `Osc 3 Pitch` (CC 22). LFO 2 is dedicated to oscillator pitch on the panel. **OOC (LFO 2 is the second LFO).**
- **LFO 1 / LFO 2 waveforms (NRPN-only):** Sine, Triangle, Sawtooth, Square, Sample-and-Hold, plus additional shapes from the LFO menu. Per the Peak User Guide LFO section, the waveform select is reached via menu navigation, not a panel switch.
- **LFO 1 / LFO 2 range (NRPN-only):** High / Low / Sync. In Sync mode, the Rate knob selects clock-division values instead of free-running Hz.
- **LFO 3 / LFO 4 — OOC:** menu-only, reachable through the Modulation Matrix as mod sources. No dedicated panel destinations.
- **Mod Env 1 destinations (panel-fixed):** `Osc 1 Shape` (CC 119), `Osc 2 Shape` (CC 40), `Osc 3 Pitch` (CC 72 — note: Osc 3's panel-fixed Mod Env 1 destination is **pitch**, not shape, in contrast to Osc 1 / Osc 2), `Filter Cutoff` (panel `Mod 1 Filter`, CC 78). Mod Env 1 is the closest device analogue to the canonical "filter envelope" because of the `Filter Source` switch — see Filter envelope subsection.
- **Mod Env 2 destinations (panel-fixed) — OOC:** `Osc 1 Pitch` (CC 9), `Osc 2 Pitch` (CC 38), `Osc 3 Pitch` (CC 43). **OOC (third envelope).**
- **Amp Env destinations:** hardwired to the VCA at full depth (no depth knob — the canonical "amp envelope → VCA" routing). Additionally routable via the `Filter Source` switch into Filter Cutoff at the `Amp Filter` depth (CC 77) — **the Amp-Env-into-Cutoff routing is out-of-common-core (Amp-Env-as-filter-envelope dual-role).**
- **Filter envelope source select (`Filter Source` switch):** `Mod Env 1`, `Amp Env`, or both summed. Per the Peak User Guide: "selects whether the filter frequency is to be varied by Mod Envelope 1 (Mod Env 1) and/or the Amp Envelope (Amp Env): note that these two sources may be combined." Common-core when set to `Mod Env 1` only; **the `Amp Env` and `Mod Env 1 + Amp Env` settings are out-of-common-core.**
- **Oscillator Shape source select (per-Osc `Source` switch):** `Mod Env 1`, `LFO 1`, or `Manual` (Shape Amount knob is the static value). When set to `Manual`, the Shape Amount knob is the static shape control; when set to `Mod Env 1` or `LFO 1`, the Shape Amount knob acts as a **modulation depth** control. Common-core when set to `LFO 1` (LFO 1 → Shape destination is in the common-core small set); **the `Mod Env 1 → Shape` and `Manual` settings are quirks of the panel control, recorded under Quirks.**

**Modulation Matrix (out-of-common-core — free assignable matrix):**

- **16 slots × 2 sources per slot × 1 destination per slot.** Each slot has one destination, two sources combined multiplicatively (or via a slot-level operator), and a depth (0–16383 via NRPN). Mod Matrix sources include the 4 LFOs (LFO 1, LFO 2, LFO 3, LFO 4), 3 envelopes (Mod Env 1, Mod Env 2, Amp Env), velocity, aftertouch (channel pressure), key tracking, modwheel, expression pedal, and a constant. Mod Matrix destinations include essentially every per-voice engine parameter — Osc N Pitch / Shape / Vsync / Mix, Ring Mod Mix, Noise Mix, Filter Cutoff / Resonance / Track / Overdrive / Post Drv, the AHDSR slots of all three envelopes, LFO 1 / LFO 2 Rate, Pan, FM amount, and others. The mod-matrix destination list per the Peak User Guide includes labels visible in the menu such as `Ns>FiltF`, `O3>FiltF`, `FM O3>O1`, `FM O2>O3`, `FM O1>O2`, `ModEnv2R`, `ModEnv2D`, `ModEnv2A`, `ModEnv1R`, ... — verbatim destination labels are not all enumerated in the Peak User Guide's main text (they are reached via the Mod Matrix menu navigation).
- **FX Modulation Matrix (4 + 4 = 8 additional slots, FX MX1–MX8 per midi.guide):** routes mod sources to FX parameters (chorus, delay, reverb, distortion). **OOC (FX-routing extension).**
- The two-sources-per-slot design is unusual relative to other surveyed devices — the PolyBrute 12 has a 12 × 32 matrix with single-source slots; the Moog Muse has a 16-slot single-source matrix; the OB-X8 has a Mod Box 2-LFO routing; the Summit / Peak combines two sources multiplicatively per slot, enabling sidechain-like routings (e.g. velocity × LFO 1 → Filter Cutoff) without consuming two slots.

**Aftertouch routings:** channel aftertouch is recognized and is available as a Mod Matrix source. **No fixed-panel aftertouch destinations** — aftertouch must be routed through the Mod Matrix on the Summit / Peak. **OOC (channel-pressure routing).**

**Velocity routings:** velocity is available as a Mod Matrix source. The Filter Source switch combined with `Amp Filter` and `Mod 1 Filter` depth knobs gives implicit velocity-into-filter routing via the envelope routing (the Amp Env and Mod Env 1 are velocity-sensitive — see Quirks). **Direct velocity → filter / amp routing without going through the Mod Matrix is not exposed on the panel.**

**Pitch-bend fixed routings:** standard MIDI pitch bend with range set per-Osc by the `BendRange` parameter (−24 .. +24 semitones, NRPN-only, in the per-Oscillator menu).

Sources for routing: Peak User Guide v1.2 (Oscillators / Mixer / Filter / Envelopes / LFOs / Modulation Matrix sections, plus the per-Oscillator and Voice menus), Summit Get Started Guide (Single / Multi modes), midi.guide *Novation Summit and Peak* CC + NRPN reference.

## Quirks

- **Hybrid engine; subtractive-compatible NCO oscillators + analog signal path.** The Summit / Peak's "NCO" oscillators are described by Novation as "FPGA-based Numerically Controlled Oscillators running at 24 MHz [generating] waveforms indistinguishable from those produced by analogue oscillators." The basic Sine / Triangle / Sawtooth / Pulse waveforms are subtractive-compatible (the digital implementation is musically equivalent to a virtual-analog VCO). The 60 wavetables (`Wave = More` + `WaveMore` selector, NRPN-only) move the engine into wavetable / hybrid territory when used. **Out-of-common-core: wavetable algorithms (60-table set), per-osc `SawDense` / `DenseDet` (osc-stacking inside one voice), per-osc `Diverge` (per-voice oscillator drift simulation), per-osc `Drift` (vintage drift).**
- **Bi-timbral 16-voice (Summit only — Peak is mono-timbral 8-voice).** Single mode allocates all 16 voices to one Patch. Multi mode splits 8 + 8 across Part A and Part B in three sub-modes: Layer (both Parts across the keyboard), Split (keyboard divided into two zones), Dual (separate MIDI channels per Part). Each Part runs its own complete copy of the per-voice engine (oscillators, filter, envelopes, LFOs, mod matrix, FX). **Out-of-common-core: bi-timbral split / layer / dual mode (the canonical schema's common-core covers a single mono-timbral patch).** The Peak shares the per-engine architecture and CC map with the Summit but is mono-timbral 8-voice — most of this survey applies to both devices, with the Summit-specific bi-timbral controls flagged as such.
- **CC numbers are shared between Summit and Peak.** The midi.guide reference is published as a single combined "Novation Summit and Peak" chart, and the Peak User Guide v1.2 appendix lists the same parameter numbers as the Summit User Guide. In Multi: Dual mode, the same CC sent on Part A's MIDI channel addresses Part A's engine, and the same CC sent on Part B's channel addresses Part B's engine. The Summit-specific bi-timbral controls (Single / Multi mode, Patch A / B, Split Point, per-Part MIDI channel) are NRPN / SysEx only — no top-level CCs published.
- **Three NCO oscillators per voice, but common-core covers only two.** The Summit / Peak's per-voice oscillator count is **3** (`Osc 1`, `Osc 2`, `Osc 3`), exceeding the canonical schema's common-core 2-oscillator subset. Osc 3 is fully out-of-common-core. Most of the surveyed devices have 2 main subtractive oscillators (Prophet-6: 2 + sub; JUNO-X analog engine: 2; OB-X8: 2; Prophet-5: 2; Moog Muse: 2 + sub + noise; PolyBrute 12: 2 + sub; Minilogue XD: 2 + multi-engine), making the Summit / Peak the outlier with 3 main NCOs (no panel sub).
- **No dedicated sub-oscillator.** Sub-octave behavior is reached by transposing one of the three NCOs down via its `Range` switch (32') or `Coarse` knob. The canonical schema's "sub osc level" parameter does not have a clean Summit / Peak analogue.
- **NCO wavetables ("Wave = More") are wavetable-style oscillator algorithms but addressable in the same panel position as the basic waveforms.** Per the Peak User Guide: "the Wave switch selects different areas of the wavetable. When Source is set to Mod Env 1 or LFO 1, [Shape Amount] acts as a Modulation Depth control" — meaning the wavetable position can itself be modulated by Mod Env 1 or LFO 1, giving wavetable-scanning behavior. Although Novation markets these as "NCO" oscillators (and they are subtractive-compatible when set to the basic waveforms), the wavetable algorithm set is **not** subtractive synthesis — it is a wavetable engine layered on top of the subtractive panel. **Out-of-common-core (wavetable / multi-shape oscillator algorithms).**
- **Multi-mode filter (LP / BP / HP) with continuously-variable slope (12 dB/oct ↔ 24 dB/oct on the LP path).** Per the Peak User Guide: "There are three basic filter types, all of which are available in Peak: low-pass, band-pass, and high-pass... On Peak, the filter type is selected with the Shape switch." The slope morph is on the LP path only. **Out-of-common-core: HP and BP modes, slope morph (the canonical common-core covers a single LP filter at a fixed slope).**
- **Three envelopes per voice (Amp Env + Mod Env 1 + Mod Env 2), all AHDSR with five phases.** The H (Hold) phase is menu-only — not addressable via top-level CC. AHD phases can be looped repeatedly (Novation's spec calls this out as a feature). **Out-of-common-core: third envelope (Mod Env 2), Hold phase, AHD-loop / repeat modes.**
- **No dedicated filter envelope — Mod Env 1 plus a `Filter Source` switch fills the role.** The `Source` switch in the Filter section selects whether Filter Cutoff is modulated by Mod Env 1, Amp Env, or both summed. The dual-source design is unusual; most surveyed devices have a dedicated "filter envelope" wired only into the filter. The closest device analogue to the canonical "filter envelope" is Mod Env 1 with the `Filter Source` switch set to `Mod Env 1` only. **Out-of-common-core: dual-source filter envelope (Mod Env 1 + Amp Env summing into Cutoff).**
- **Four LFOs (LFO 1, LFO 2 panel; LFO 3, LFO 4 menu-only) — common-core covers LFO 1 only.** Common-core LFO 1 destination subset on the panel: filter cutoff + per-Osc Shape (3 destinations × dedicated depth knobs). LFO 2's panel-fixed destinations are oscillator pitch (Osc 1 / 2 / 3 Pitch via dedicated depth knobs) — out-of-common-core. LFO 3 / LFO 4 are reachable through the Modulation Matrix as mod sources only. **Out-of-common-core: LFO 2, LFO 3, LFO 4 — and any LFO routing not in the panel-fixed common-core subset.**
- **16-slot Modulation Matrix with two sources per slot.** The mod matrix is the routing primary for any modulation outside the panel-fixed paths. The two-sources-per-slot design (sources combined multiplicatively per slot) is unusual — most surveyed devices have single-source matrix slots (PolyBrute 12: 12×32 single-source; Moog Muse: 16 single-source; OB-X8: dedicated Mod Box). The Summit / Peak's design enables sidechain-like routings (e.g. velocity × LFO 1 → Filter Cutoff) without consuming two slots. **Out-of-common-core: free assignable matrix.**
- **FX Modulation Matrix (4–8 additional slots, FX MX1–FX MX8).** Routes mod sources to FX parameters. **Out-of-common-core (FX routing extension).**
- **FM between oscillators (FM O1>O2, FM O2>O3, FM O3>O1) — three audio-rate FM paths.** Each FM path's depth is a Modulation Matrix destination (visible labels `FM O1>O2` / `FM O2>O3` / `FM O3>O1` in the matrix destination list). Per the Peak User Guide: "Frequency Modulation of oscillators (FM) by filter, other oscillators or Noise" — meaning Filter and Noise can also FM the oscillators (via the matrix). **Out-of-common-core: oscillator-rate FM, Filter-FM (cutoff modulating an oscillator's frequency at audio rate), Noise-FM. Although the Peak marketing positions these as "subtractive-compatible" features, FM and audio-rate cross-modulation are not subtractive synthesis primitives.**
- **`Vsync` (virtual-oscillator hard sync) per oscillator.** Per the Peak User Guide: "The virtual oscillators are not heard, but the frequency of each is used to re-trigger that of the main oscillator. The Vsync parameter controls the frequency offset of the virtual oscillator relative to the (audible) main oscillator." When `Vsync` is a multiple of 16, the virtual frequency is a musical harmonic of the main oscillator — values between multiples produce discordant overtones. The virtual oscillator is private to that one main oscillator (no master-slave between Osc 1 and Osc 2 — each Osc has its own private master). **Out-of-common-core: virtual-oscillator hard sync.**
- **Ring Modulator inputs are fixed (Osc 1 × Osc 2).** No selectable inputs. **Out-of-common-core: ring modulator.**
- **Three stages of analog distortion per voice.** Per the Summit Get Started Guide / Peak feature spec: "three stages of analogue distortion per voice" — `Filter Overdrive` (CC 80, pre-filter), `Filter Post Drv` (CC 36, post-filter), and the FX Distortion stage (CC 104, post-engine). The `Filter Overdrive` and `Filter Post Drv` are per-voice and pre-FX; the FX Distortion is post-engine. **Out-of-common-core: filter-stage analog distortion stages (the canonical common-core does not include filter saturation).**
- **Non-standard CC mappings.** The Summit / Peak uses some non-standard CC numbers for its own parameters: **CC 65** is used for `Osc 3 Range` (the standard MIDI Portamento On/Off controller), and **CC 35** is used for `Glide On` (a non-standard controller for glide on/off — the standard would be CC 65). Hosts targeting the Summit / Peak must respect the device-native mapping; sending standard MIDI Portamento On/Off on CC 65 will set Osc 3 Range, not toggle glide. **Documented quirk — out-of-common-core if the canonical schema assumes standard MIDI controller assignments.**
- **NRPN "two CCs per parameter" trick for 0–16383 resolution.** Per midi.guide: "The Peak uses a clever MIDI trick to allow finer control of parameters by assigning 2x CC messages to 1x parameter, allowing for 0-255 instead of just 0-127." The published mod-matrix and FX-matrix slots use NRPN-LSB/MSB pairs to reach 0–16383 underlying resolution. The CC pathway (single 7-bit CC) is quantized to 128 steps with hardware-defined non-linear taper.
- **CC values are raw 7-bit (0–127) with knob-taper non-linearity.** Underlying program-parameter ranges are wider for many parameters (mod matrix slot depths reach 0–16383 via NRPN). Acoustic-unit interpretation (Hz, ms, dB) is not exposed by the device — you only see knob position via CC.
- **Polyphony Mode list (Mono / MonoLG / Mono2 / Poly / Poly2) is unusual.** Five polyphony modes — three monophonic (Mono = standard monophonic, MonoLG = legato-only Glide, Mono 2 = voice-rotation mono with full-envelope-per-note) and two polyphonic (Poly = standard, Poly 2 = voice-rotation poly). Common-core covers `Poly` only. **Out-of-common-core: Mono / MonoLG / Mono2 / Poly2.**
- **Unison range (1, 2, 3, 4, 8 voices) — non-contiguous enum.** The Unison parameter does not include 5, 6, or 7 voices — the enum is `1, 2, 3, 4, 8`. With `Unison = 8`, Peak becomes a multi-voice monophonic synth (all 8 voices stacked on a single note). **Out-of-common-core: Unison ≥ 2 (voice-stacking is a voice-allocation extension).**
- **Per-Part Animate buttons.** The Summit's two Animate buttons (per-Part) provide live-performance spot effects — pressing Animate triggers a programmed parameter change while held; `Hold` (CC 114 / 115) locks the Animate state on. **Out-of-common-core (live-performance spot-effect feature).**
- **CC vs published chart agreement.** Every CC number in this survey was cross-checked against the midi.guide *Novation Summit and Peak* combined chart. The Peak User Guide v1.2's MIDI Implementation appendix and the Summit User Guide's MIDI section list the same parameter numbers. No discrepancies found between midi.guide and the Peak User Guide for the parameters in this survey. The Summit-specific bi-timbral controls are NRPN / SysEx only — no top-level CCs published — and were sourced from the Summit Get Started Guide.
- **Effects are post-engine and out of scope.** The Summit / Peak's Distortion / Chorus / Delay / Reverb FX bus runs after the VCA. CC numbers recorded above for completeness only; not part of the subtractive ontology.
