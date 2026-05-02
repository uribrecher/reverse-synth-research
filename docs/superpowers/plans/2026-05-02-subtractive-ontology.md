# Subtractive-Synth Ontology Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce a cross-vendor Markdown survey of eight subtractive polysynths plus a hand-authored `subtractive.schema.json` defining the common-core canonical parameter ontology — the comm protocol between the analysis pipeline and `keyboards-mcp`.

**Architecture:** Pure design artifact. Each task produces one deliverable file under `reverse-synth-research/parameter-mapping/`. Eight survey docs (one per device) feed into one canonical JSON Schema and a prose companion. No code, no codegen, no tests-as-code; verification is structural (required sections present) and syntactic (valid JSON / valid JSON Schema). Final cross-check pass confirms every canonical param has at least three surveyed devices that map to it.

**Tech Stack:** Markdown + JSON Schema (Draft 2020-12). Verification with `python3 -m json.tool` (always available) and `node` (for JSON parse). Schema-draft compliance is a manual review per the MVP spec.

**Spec:** [`docs/superpowers/specs/2026-05-02-subtractive-ontology-mvp.md`](../specs/2026-05-02-subtractive-ontology-mvp.md). Plan covers ONLY what is in the spec; the [backlog file](../specs/2026-05-02-subtractive-ontology-backlog.md) is deferred work.

**Working directory for all `git`/`ls`/path commands in this plan:** `/Users/uribrecher/test/sounds-and-recreation/reverse-synth-research`. The current branch is `feature/subtractive-ontology-mvp-spec` and the spec + backlog are already committed there.

---

## File Structure

```
reverse-synth-research/parameter-mapping/
├── parameter-mapping-plan.md          (existing — design plan)
├── survey/
│   ├── README.md                      (Task 1 — index + sourcing notes + survey template)
│   ├── prophet-6.md                   (Task 2)
│   ├── juno-x.md                      (Task 3 — subtractive layer only)
│   ├── prophet-5.md                   (Task 4)
│   ├── ob-x8.md                       (Task 5)
│   ├── moog-muse.md                   (Task 6)
│   ├── polybrute-12.md                (Task 7)
│   ├── minilogue-xd.md                (Task 8 — subtractive layer only)
│   └── novation-summit.md             (Task 9)
├── subtractive.schema.json            (Task 10 — canonical ontology)
└── subtractive-ontology.md            (Task 11 — prose companion)
```

Task 12 is a cross-check sweep — it edits files written in earlier tasks if the sweep finds gaps; it does not create new files.

**Survey-doc template** (used by Tasks 2–9 and defined fully in Task 1):

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

<one paragraph describing the dry signal path, oscillators → mixer → filter → amp, plus modulation>

## Parameters

### Oscillators
| Device-native name | Type | Range | Unit | Enum values | Description |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

### Mixer
| ... |

### Filter
| ... |

### Envelopes
| ... |

### LFO
| ... |

### Voice / Master
| ... |

## Modulation routing

<the destination vocabulary the device exposes for its LFO(s) and mod matrix; sources column for each routing slot; bullet list is fine>

## Quirks

- <knob taper, log vs linear, MIDI 0–127 quantization, special "off" values, asymmetric ranges, anything that affects canonical mapping>
```

Survey docs deliberately keep device-native names. The mapping to canonical names happens in Task 12 (cross-check) and is implicit in `subtractive-ontology.md`.

---

### Task 1: Survey infrastructure (`survey/README.md`)

**Files:**
- Create: `parameter-mapping/survey/README.md`

- [ ] **Step 1: Create the survey directory**

```bash
mkdir -p parameter-mapping/survey
ls parameter-mapping/survey/
```
Expected: empty directory listing (no error).

- [ ] **Step 2: Write `parameter-mapping/survey/README.md`**

Create the file with this exact content:

````markdown
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

````markdown
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
````

## Common-core boundary

This survey informs a "common-core" canonical ontology — see the MVP spec for the rule. Features supported by some surveyed devices but excluded from common-core (cross-mod, sync, ring mod, dual filters, multi-mode filter morphing, full mod-matrix slots, vintage knob simulation) are still recorded in survey docs under quirks/routing — they just do not get canonical params in `../subtractive.schema.json`. They are catalogued in `../../docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md`.
````

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/README.md
```
Expected: `4` (Devices, Sourcing approach, Survey template, Common-core boundary).

```bash
grep -c '^| [0-9] |' parameter-mapping/survey/README.md
```
Expected: `8` (one row per device in the device table).

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/README.md
git commit -m "Add survey/ scaffold and per-device template"
```

---

### Task 2: Prophet-6 survey (`survey/prophet-6.md`)

**Files:**
- Create: `parameter-mapping/survey/prophet-6.md`
- Reference (read-only): `../keyboards-mcp/src/keyboard_models/sequential_circuits/prophet_6/midi-map.ts` (478 lines).

- [ ] **Step 1: Read the keyboards-mcp midi-map for Prophet-6**

```bash
wc -l ../keyboards-mcp/src/keyboard_models/sequential_circuits/prophet_6/midi-map.ts
cat ../keyboards-mcp/src/keyboard_models/sequential_circuits/prophet_6/midi-map.ts
```

Extract a working list of device-native parameter names by section. The file groups params with `// ── Section Name ──` comments — those section comments map directly onto the survey template's "## Parameters" sub-sections.

- [ ] **Step 2: Web-search for the official Prophet-6 MIDI Implementation Chart**

Search query suggestions:
- `Sequential Prophet-6 MIDI Implementation Chart filetype:pdf`
- `Prophet-6 Operation Manual site:sequential.com`

Goal: confirm CC numbers in the midi-map.ts match Sequential's published chart and capture any params present in the chart that the midi-map does not yet implement (those still get listed in the survey — the survey describes the *device*, not the MCP). Cite at least one URL or document title in the survey's Sources block.

- [ ] **Step 3: Write `parameter-mapping/survey/prophet-6.md`**

Follow the template from Task 1. Required content:

- Header: Manufacturer = Sequential, Year = 2018, Engine type = analog, Polyphony = 6, Sources block (manual + MIDI chart URLs).
- Signal path: one paragraph. Two VCOs → mixer (with sub-osc on Osc 1 and noise) → low-pass + high-pass filters → VCA. ADSR for filter and amp. One LFO. Effects (delay/reverb) are post-engine and out of scope for this survey.
- Parameters tables (one per section), with **device-native names from the midi-map.ts** and ranges. For continuous params with `encoding: RAW` and `min: 0, max: 127`, record range as `0–127 (raw MIDI CC)` and put the acoustic interpretation in the Description column. For discrete params with `labels`, list the enum values verbatim.
- Sections to cover (each its own table): Oscillator 1, Oscillator 2, Mixer, Low-pass filter, High-pass filter, Filter envelope, Amp envelope, LFO, Glide / unison / poly mod, Master.
- Modulation routing: Prophet-6 has a single LFO with selectable destination (one-of) — capture the destination enum verbatim.
- Quirks bullets: monotimbral; CC values are raw 0–127 with knob-taper non-linearity; many discrete switches are CC with threshold semantics (record the threshold convention from the midi-map).

- [ ] **Step 4: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/prophet-6.md
```
Expected: `4` (Signal path, Parameters, Modulation routing, Quirks).

```bash
grep -c '^### ' parameter-mapping/survey/prophet-6.md
```
Expected: `>= 6` (Oscillators sections, Mixer, Filter, Envelopes, LFO, Voice/Master — at minimum six `###` headings).

- [ ] **Step 5: Commit**

```bash
git add parameter-mapping/survey/prophet-6.md
git commit -m "Add Prophet-6 survey"
```

---

### Task 3: Roland JUNO-X survey, Analog Synth engine layer (`survey/juno-x.md`)

**Files:**
- Create: `parameter-mapping/survey/juno-x.md`
- Reference (read-only): `../keyboards-mcp/src/keyboard_models/roland/juno_x/engines/analog-synth.ts`.

The JUNO-X has four engines (AnalogSynth, ZCore, JunoXModel, RDPiano). **Only `AnalogSynth` is subtractive** — the others are sample/model/piano. This survey covers only `AnalogSynth`; the rest are out-of-scope for the subtractive ontology.

- [ ] **Step 1: Read the JUNO-X analog-synth params**

```bash
wc -l ../keyboards-mcp/src/keyboard_models/roland/juno_x/engines/analog-synth.ts
cat ../keyboards-mcp/src/keyboard_models/roland/juno_x/engines/analog-synth.ts
```

Note that param keys are prefixed `as_*` (Analog Synth). Use the `name`, `section`, `min`/`max`, `labels`, and `description` fields directly in the survey tables.

- [ ] **Step 2: Web-search for the JUNO-X Parameter Guide and MIDI Implementation**

Search query suggestions:
- `Roland JUNO-X Parameter Guide filetype:pdf`
- `Roland JUNO-X MIDI Implementation`

Goal: verify the analog-synth.ts coverage against the official Parameter Guide. Cite at least one URL.

- [ ] **Step 3: Write `parameter-mapping/survey/juno-x.md`**

Follow the template. Required content:

- Header: Manufacturer = Roland, Year = 2022, Engine type = virtual analog, Polyphony = up to 8 voices per scene, Sources block.
- Important caveat in the first paragraph: **only the `AnalogSynth` engine is covered**; ZCore, JUNO-X Model, and RDPiano are excluded.
- Signal path: 2 oscillators (with sub osc, ring mod, cross mod, sync available — note these as out-of-common-core), mixer, multi-mode filter, amp, two LFOs, two envelopes (filter + amp), portamento.
- Parameters tables: Oscillator 1, Oscillator 2, Sub-osc, Mixer (incl. ring mod / noise), Filter, Filter envelope, Amp envelope, LFO 1, LFO 2, Voice / portamento, Effects bus parameters relevant to dry signal (mark as out of common-core).
- Modulation routing: per-LFO destination enum from `as_lfo*_destination` (or equivalent). Mod matrix slots (if present in analog-synth.ts) listed under routing with note "out of common-core".
- Quirks bullets: discrete-with-labels params encoding (the `labels: { 0: "SIN", 1: "SAW-DW", ... }` shape is unusual — record the label-to-CC convention); `perPart: true` semantics meaning params are per-voice-part within a scene.

- [ ] **Step 4: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/juno-x.md
```
Expected: `4`.

```bash
grep -i 'AnalogSynth\|analog synth' parameter-mapping/survey/juno-x.md | head -2
```
Expected: at least one match (the engine-scope caveat must be explicit).

- [ ] **Step 5: Commit**

```bash
git add parameter-mapping/survey/juno-x.md
git commit -m "Add JUNO-X (Analog Synth engine) survey"
```

---

### Task 4: Sequential Prophet-5 Rev 4 survey (`survey/prophet-5.md`)

**Files:**
- Create: `parameter-mapping/survey/prophet-5.md`

No `keyboards-mcp` reference — sourcing is web-only.

- [ ] **Step 1: Web-search for the Prophet-5 Rev 4 Operation Manual + MIDI chart**

Search query suggestions:
- `Sequential Prophet-5 Rev 4 Operation Manual filetype:pdf`
- `Prophet-5 MIDI Implementation Chart filetype:pdf`

Cite at least one URL or document title in Sources.

- [ ] **Step 2: Write `parameter-mapping/survey/prophet-5.md`**

Follow the template. Required content:

- Header: Manufacturer = Sequential, Year = 2020 (Rev 4 reissue), Engine type = analog, Polyphony = 5 (or 10 with optional Prophet-10 firmware — note in caveats), Sources block.
- Signal path: 2 VCOs (with sync available — out of common-core), mixer (incl. noise), 4-pole low-pass filter, VCA, two ADSR envelopes (filter + amp), one LFO, poly-mod section (out of common-core), unison.
- Parameters tables: Oscillator A, Oscillator B, Mixer, Filter, Filter envelope, Amp envelope, LFO, Poly-mod (with "out of common-core" note), Voice (unison/glide), Master.
- Modulation routing: LFO destinations (Freq A, Freq B, Filter — small fixed set on Prophet-5). Poly-mod sources/destinations listed but flagged out-of-common-core.
- Quirks: vintage-knob amount param (Rev 4 specific) — record but flag out-of-common-core; sync switch; the device's MIDI implementation has CCs added in the Rev 4 reissue that the original 1978 hardware lacked.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/prophet-5.md
```
Expected: `4`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/prophet-5.md
git commit -m "Add Prophet-5 Rev 4 survey"
```

---

### Task 5: Oberheim OB-X8 survey (`survey/ob-x8.md`)

**Files:**
- Create: `parameter-mapping/survey/ob-x8.md`

- [ ] **Step 1: Web-search for the OB-X8 manual + MIDI chart**

Search query suggestions:
- `Oberheim OB-X8 Owners Manual filetype:pdf`
- `OB-X8 MIDI Implementation Chart`

Cite at least one URL.

- [ ] **Step 2: Write `parameter-mapping/survey/ob-x8.md`**

Follow the template. Required content:

- Header: Manufacturer = Oberheim, Year = 2022, Engine type = analog, Polyphony = 8, Sources block.
- Signal path: 2 oscillators (with cross-mod, sync — out of common-core), mixer (incl. noise), selectable LP/HP/BP/Notch filter (multi-mode — note the LP slope switching 2-pole/4-pole as out of common-core), VCA, two envelopes (filter + amp), one LFO with multiple destinations, optional second LFO/mod source. Vintage emulation modes (OB-X / OB-Xa / OB-8) — note in caveats.
- Parameters tables: Oscillator 1, Oscillator 2, Mixer, Filter, Filter envelope, Amp envelope, LFO, Voice (unison, glide, vintage), Master.
- Modulation routing: LFO destination set; X-Mod section flagged out-of-common-core.
- Quirks: vintage-mode switch changes envelope curves and filter response — capture; the multi-mode filter exposes a single Cutoff CC but mode is a separate switch.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/ob-x8.md
```
Expected: `4`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/ob-x8.md
git commit -m "Add OB-X8 survey"
```

---

### Task 6: Moog Muse survey (`survey/moog-muse.md`)

**Files:**
- Create: `parameter-mapping/survey/moog-muse.md`

- [ ] **Step 1: Web-search for the Moog Muse manual + MIDI chart**

Search query suggestions:
- `Moog Muse User Guide filetype:pdf`
- `Moog Muse MIDI Implementation`

Cite at least one URL.

- [ ] **Step 2: Write `parameter-mapping/survey/moog-muse.md`**

Follow the template. Required content:

- Header: Manufacturer = Moog Music, Year = 2024, Engine type = analog, Polyphony = 8 (8 voices, bi-timbral), Sources block.
- Signal path: 2 VCOs (with hard sync, FM, ring mod — out of common-core), mixer (incl. noise + feedback), Moog ladder LP filter (with cutoff/resonance/keytrack), VCA, three envelopes (filter, amp, mod — third env is out of common-core), two LFOs (LFO 2 out of common-core), mod matrix.
- Parameters tables: Oscillator 1, Oscillator 2, Mixer, Filter, Filter envelope, Amp envelope, Mod envelope (flag OOC), LFO 1, LFO 2 (flag OOC), Voice (unison, glide, bi-timbral), Master.
- Modulation routing: per-LFO destinations + mod matrix slots (flagged OOC).
- Quirks: feedback path adds saturation that interacts with cutoff; bi-timbral mode splits voice count (4+4 or 6+2 etc.) — record but note that survey covers only single-timbre Layer A.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/moog-muse.md
```
Expected: `4`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/moog-muse.md
git commit -m "Add Moog Muse survey"
```

---

### Task 7: Arturia PolyBrute 12 survey (`survey/polybrute-12.md`)

**Files:**
- Create: `parameter-mapping/survey/polybrute-12.md`

- [ ] **Step 1: Web-search for the PolyBrute 12 manual + MIDI chart**

Search query suggestions:
- `Arturia PolyBrute 12 User Manual filetype:pdf`
- `PolyBrute 12 MIDI Implementation`

Cite at least one URL.

- [ ] **Step 2: Write `parameter-mapping/survey/polybrute-12.md`**

Follow the template. Required content:

- Header: Manufacturer = Arturia, Year = 2024, Engine type = analog, Polyphony = 12, Sources block.
- Signal path: 2 VCOs with the Brute's wave-shaping (metalizer, ultrasaw — out of common-core), mixer (incl. noise + sub), **two filters (Steiner-Parker + Ladder, series/parallel, morphable — out of common-core**; record the Ladder cutoff/resonance only as the common-core LP and note the rest), VCA, three envelopes (out of common-core for the third), three LFOs (LFO 2/3 OOC), 12-slot mod matrix (OOC).
- Parameters tables: Oscillator 1, Oscillator 2, Sub, Mixer, Filter (Ladder primary; Steiner-Parker noted OOC), Filter envelope, Amp envelope, Third envelope (flag OOC), LFO 1, LFO 2/3 (flag OOC), Mod matrix (flag OOC), Voice (poly aftertouch — flag OOC; glide/unison common-core), Master.
- Modulation routing: 12-slot mod matrix listed; common-core covers only LFO 1 → small destination set.
- Quirks: the dual-filter morph is a single morph param but treated as out-of-common-core; the Brute wave-shaping is per-osc nonlinearity; poly-aftertouch is a distinguishing feature but doesn't affect canonical params.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/polybrute-12.md
```
Expected: `4`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/polybrute-12.md
git commit -m "Add PolyBrute 12 survey"
```

---

### Task 8: Korg Minilogue XD subtractive layer survey (`survey/minilogue-xd.md`)

**Files:**
- Create: `parameter-mapping/survey/minilogue-xd.md`

The Minilogue XD has a digital "Multi-Engine" osc (VPM/noise/user wavetable) — only the analog osc + analog filter signal path is "subtractive" in the strict sense. **Survey covers only the subtractive path**; the multi-engine is out of scope and lives in the wavetable/FM backlog.

- [ ] **Step 1: Web-search for the Minilogue XD owner's manual + MIDI chart**

Search query suggestions:
- `Korg Minilogue XD Owners Manual filetype:pdf`
- `Minilogue XD MIDI Implementation Chart`

Cite at least one URL.

- [ ] **Step 2: Write `parameter-mapping/survey/minilogue-xd.md`**

Follow the template. Required content:

- Header: Manufacturer = Korg, Year = 2019, Engine type = hybrid (VCO + Multi-Engine), Polyphony = 4, Sources block.
- Engine-scope caveat in the first paragraph: only the **analog VCO 1 / VCO 2 signal path** is covered. The Multi-Engine is excluded.
- Signal path: 2 VCOs (with sync, ring — out of common-core), mixer (incl. noise + multi-engine — exclude multi-engine), 2-pole resonant LP filter, VCA, two envelopes, one LFO, optional mod sequencer (out of common-core).
- Parameters tables: VCO 1, VCO 2, Mixer (excluding multi-engine), Filter, Amp envelope, EG (filter envelope alternative — describe semantics), LFO, Voice (unison limited; mono/poly/uni mode), Master.
- Modulation routing: LFO destinations are limited (Cutoff, Pitch, Shape on Minilogue XD) — record verbatim. Mod sequencer flagged OOC.
- Quirks: filter envelope is shared with the EG generator (single envelope serves dual purpose); Filter Type switch (2-pole/4-pole) is OOC; the Minilogue XD voice mode list is unusually long (Poly, Mono, Unison, Chord, Arpeggio, etc.) — common-core covers only Poly/Mono/Unison.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/minilogue-xd.md
```
Expected: `4`.

```bash
grep -i 'Multi-Engine\|multi-engine\|multi engine' parameter-mapping/survey/minilogue-xd.md | head -2
```
Expected: at least one match (the engine-scope caveat must be explicit).

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/minilogue-xd.md
git commit -m "Add Minilogue XD (subtractive layer) survey"
```

---

### Task 9: Novation Summit survey (`survey/novation-summit.md`)

**Files:**
- Create: `parameter-mapping/survey/novation-summit.md`

- [ ] **Step 1: Web-search for the Summit / Peak User Guide + MIDI chart**

Search query suggestions:
- `Novation Summit User Guide filetype:pdf`
- `Novation Peak MIDI Implementation` (Peak is the mono-timbral predecessor; same engine doc applies in places)

Cite at least one URL.

- [ ] **Step 2: Write `parameter-mapping/survey/novation-summit.md`**

Follow the template. Required content:

- Header: Manufacturer = Novation, Year = 2019, Engine type = hybrid (NCO digital oscillators + analog filter), Polyphony = 16 (bi-timbral, 8+8), Sources block.
- Signal path: 3 NCO oscillators per voice (the Summit's third osc is OOC for common-core — survey covers all three but note common-core uses only osc 1 and 2; sub osc available), mixer (incl. ring mod, noise — ring mod OOC), analog multi-mode filter (LP/HP/BP — multi-mode OOC; LP is common-core), VCA, three envelopes (third env OOC), two LFOs (LFO 2 OOC), 16-slot mod matrix (OOC).
- Parameters tables: Oscillator 1, Oscillator 2, Oscillator 3 (flag OOC), Sub (flag OOC if applicable), Mixer (incl. ring mod flagged OOC), Filter, Filter envelope, Amp envelope, Third envelope (flag OOC), LFO 1, LFO 2 (flag OOC), Voice (unison, glide), Master.
- Modulation routing: the 16-slot mod matrix is the routing primary; LFO 1 destinations subset listed for common-core.
- Quirks: bi-timbral split/layer modes; FM between oscillators (OOC); the wavetable-style oscillator algorithms (which Novation markets as "NCO") are still subtractive-compatible but capture in routing notes.

- [ ] **Step 3: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/survey/novation-summit.md
```
Expected: `4`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/survey/novation-summit.md
git commit -m "Add Novation Summit survey"
```

---

### Task 10: Author `subtractive.schema.json`

**Files:**
- Create: `parameter-mapping/subtractive.schema.json`

This is the canonical JSON Schema. Common-core coverage rule applies — anything flagged OOC in the surveys does not get a top-level canonical param.

- [ ] **Step 1: Write `parameter-mapping/subtractive.schema.json`**

Create the file with this exact content:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://reverse-synth-research.local/parameter-mapping/subtractive.schema.json",
  "title": "Subtractive Synthesis Canonical Parameters",
  "description": "Common-core canonical parameter ontology for subtractive polysynths. Comm protocol between the analysis pipeline (audio-analysis-mcp) and the device side (keyboards-mcp). Schema version 0.1 — design-reviewed only, no fixtures yet. Rare/extended features are deferred per the MVP backlog.",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "engine", "params"],
  "properties": {
    "schema_version": { "type": "string", "const": "0.1" },
    "engine": { "type": "string", "const": "subtractive" },
    "params": { "$ref": "#/$defs/SubtractiveParams" }
  },
  "$defs": {
    "SubtractiveParams": {
      "type": "object",
      "additionalProperties": false,
      "required": ["osc", "filter", "envelope", "lfo", "voice", "master"],
      "properties": {
        "osc": { "$ref": "#/$defs/Oscillators" },
        "noise": { "$ref": "#/$defs/Noise" },
        "filter": { "$ref": "#/$defs/Filter" },
        "envelope": { "$ref": "#/$defs/Envelopes" },
        "lfo": { "$ref": "#/$defs/LFOs" },
        "voice": { "$ref": "#/$defs/Voice" },
        "master": { "$ref": "#/$defs/Master" }
      }
    },
    "Oscillators": {
      "type": "object",
      "additionalProperties": false,
      "required": ["1"],
      "properties": {
        "1": { "$ref": "#/$defs/Oscillator" },
        "2": { "$ref": "#/$defs/Oscillator" },
        "sub": { "$ref": "#/$defs/SubOscillator" }
      }
    },
    "Oscillator": {
      "type": "object",
      "additionalProperties": false,
      "required": ["shape", "level"],
      "properties": {
        "shape": {
          "type": "string",
          "enum": ["sine", "saw", "square", "triangle", "pulse"],
          "description": "Oscillator waveshape selector."
        },
        "level": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "x-unit": "ratio",
          "description": "Mixer level for this oscillator. 0 = muted, 1 = full."
        },
        "detune_cents": {
          "type": "number",
          "minimum": -1200,
          "maximum": 1200,
          "x-unit": "cents",
          "description": "Fine pitch offset relative to the played note."
        },
        "octave": {
          "type": "integer",
          "minimum": -3,
          "maximum": 3,
          "x-unit": "octaves",
          "description": "Coarse pitch offset in octaves."
        },
        "pulse_width_pct": {
          "type": "number",
          "minimum": 0,
          "maximum": 100,
          "x-unit": "percent",
          "description": "Pulse width when shape == 'pulse'. 50 = perfect square. Ignored for other shapes."
        }
      }
    },
    "SubOscillator": {
      "type": "object",
      "additionalProperties": false,
      "required": ["level"],
      "properties": {
        "octave": {
          "type": "integer",
          "enum": [-2, -1],
          "x-unit": "octaves",
          "description": "Sub-oscillator octave below the parent oscillator."
        },
        "level": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "x-unit": "ratio"
        }
      }
    },
    "Noise": {
      "type": "object",
      "additionalProperties": false,
      "required": ["level"],
      "properties": {
        "color": {
          "type": "string",
          "enum": ["white", "pink"],
          "description": "Noise spectrum. 'white' is flat, 'pink' rolls off 3 dB/octave."
        },
        "level": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "x-unit": "ratio"
        }
      }
    },
    "Filter": {
      "type": "object",
      "additionalProperties": false,
      "required": ["lp"],
      "properties": {
        "lp": {
          "type": "object",
          "additionalProperties": false,
          "required": ["cutoff_hz", "resonance"],
          "properties": {
            "cutoff_hz": {
              "type": "number",
              "minimum": 20,
              "maximum": 20000,
              "x-unit": "Hz",
              "description": "Low-pass filter cutoff frequency. Log-distributed in practice."
            },
            "resonance": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "x-unit": "ratio",
              "description": "Filter resonance / Q. 1 = self-oscillation territory on most analog ladders."
            },
            "envelope_amount": {
              "type": "number",
              "minimum": -1,
              "maximum": 1,
              "x-unit": "ratio",
              "description": "Signed amount the filter envelope modulates cutoff. Negative inverts the envelope."
            },
            "key_tracking": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "x-unit": "ratio",
              "description": "How much the played note pitch tracks the cutoff. 0 = none, 1 = full key follow."
            },
            "drive": {
              "type": "number",
              "minimum": 0,
              "maximum": 1,
              "x-unit": "ratio",
              "description": "Pre-filter saturation / overdrive amount."
            }
          }
        }
      }
    },
    "ADSR": {
      "type": "object",
      "additionalProperties": false,
      "required": ["attack_ms", "decay_ms", "sustain", "release_ms"],
      "properties": {
        "attack_ms": {
          "type": "number",
          "minimum": 0,
          "maximum": 10000,
          "x-unit": "ms"
        },
        "decay_ms": {
          "type": "number",
          "minimum": 0,
          "maximum": 20000,
          "x-unit": "ms"
        },
        "sustain": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "x-unit": "ratio",
          "description": "Sustain level as a ratio of peak."
        },
        "release_ms": {
          "type": "number",
          "minimum": 0,
          "maximum": 20000,
          "x-unit": "ms"
        }
      }
    },
    "Envelopes": {
      "type": "object",
      "additionalProperties": false,
      "required": ["amp"],
      "properties": {
        "amp": { "$ref": "#/$defs/ADSR" },
        "filter": { "$ref": "#/$defs/ADSR" }
      }
    },
    "LFO": {
      "type": "object",
      "additionalProperties": false,
      "required": ["rate_hz", "shape", "depth", "target"],
      "properties": {
        "rate_hz": {
          "type": "number",
          "minimum": 0.01,
          "maximum": 100,
          "x-unit": "Hz",
          "description": "LFO frequency. Log-distributed in practice."
        },
        "shape": {
          "type": "string",
          "enum": ["sine", "triangle", "square", "saw", "ramp", "sample_hold"]
        },
        "depth": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "x-unit": "ratio"
        },
        "target": { "$ref": "#/$defs/ModulationDestination" }
      }
    },
    "LFOs": {
      "type": "object",
      "additionalProperties": false,
      "required": ["1"],
      "properties": {
        "1": { "$ref": "#/$defs/LFO" }
      }
    },
    "ModulationDestination": {
      "type": "string",
      "enum": [
        "osc.pitch",
        "osc.shape",
        "osc.pwm",
        "osc.amp",
        "filter.cutoff",
        "filter.resonance",
        "amp.level"
      ],
      "description": "Common-core LFO destinations. Full mod-matrix destinations are deferred to the backlog."
    },
    "Voice": {
      "type": "object",
      "additionalProperties": false,
      "required": ["mode"],
      "properties": {
        "glide_ms": {
          "type": "number",
          "minimum": 0,
          "maximum": 5000,
          "x-unit": "ms"
        },
        "mode": {
          "type": "string",
          "enum": ["mono", "poly", "unison"]
        },
        "unison_voices": {
          "type": "integer",
          "minimum": 1,
          "maximum": 16,
          "description": "Active voices in unison mode. Ignored when mode != 'unison'."
        }
      }
    },
    "Master": {
      "type": "object",
      "additionalProperties": false,
      "required": ["volume_db"],
      "properties": {
        "volume_db": {
          "type": "number",
          "minimum": -60,
          "maximum": 6,
          "x-unit": "dB"
        }
      }
    }
  }
}
```

- [ ] **Step 2: Verify the file is valid JSON**

```bash
python3 -m json.tool parameter-mapping/subtractive.schema.json > /dev/null
```
Expected: exit 0, no output.

- [ ] **Step 3: Verify the file declares Draft 2020-12**

```bash
grep '"\$schema"' parameter-mapping/subtractive.schema.json
```
Expected: `"$schema": "https://json-schema.org/draft/2020-12/schema",` (one match).

- [ ] **Step 4: Verify `$defs` has every referenced type**

```bash
for ref in SubtractiveParams Oscillators Oscillator SubOscillator Noise Filter ADSR Envelopes LFO LFOs ModulationDestination Voice Master; do
  grep -q "\"$ref\":" parameter-mapping/subtractive.schema.json && echo "OK $ref" || echo "MISSING $ref"
done
```
Expected: 13 lines, all `OK <name>`.

- [ ] **Step 5: Manual schema-draft compliance review**

Read the schema file end-to-end. Confirm:
- Every continuous param has explicit `minimum`/`maximum`.
- Every continuous param carries `x-unit` (custom keyword annotation, ignored by validators but consumed by future bindings).
- Every discrete param uses `enum`.
- `additionalProperties: false` is on every object schema.
- The ModulationDestination enum is exactly the seven values from the spec.
- The schema file is self-contained: no external `$ref`s, only intra-file refs to `#/$defs/...`.

Mark this step `[x]` once the manual read confirms all five bullets.

- [ ] **Step 6: Commit**

```bash
git add parameter-mapping/subtractive.schema.json
git commit -m "Add subtractive.schema.json (canonical ontology, v0.1)"
```

---

### Task 11: Author `subtractive-ontology.md`

**Files:**
- Create: `parameter-mapping/subtractive-ontology.md`

- [ ] **Step 1: Write `parameter-mapping/subtractive-ontology.md`**

Create the file with this exact content (the references to surveys are concrete file paths because the survey docs were written in Tasks 2–9):

````markdown
# Subtractive Synthesis Canonical Ontology

Companion to [`subtractive.schema.json`](./subtractive.schema.json). Explains the design rules, the common-core boundary, and what each canonical param means acoustically. Audience: engineers wiring the analysis pipeline (`audio-analysis-mcp`) and the device side (`keyboards-mcp`) to this schema.

> **Status:** Schema version `0.1`. Reviewed by design only — no JSON Schema validator fixtures yet. See the [MVP spec](../docs/superpowers/specs/2026-05-02-subtractive-ontology-mvp.md) for the explicit risk acceptance.

## Design rules

1. **Physical units, not normalized stand-ins.** Hz for frequency, ms for time, cents for fine pitch, dB for loudness, 0–1 ratios where there is no natural physical unit. The schema rejects 0–127 MIDI scaling — that is a device-side concern, handled in Phase 4 translators.
2. **Hierarchical names as object structure.** A canonical param like `filter.lp.cutoff_hz` is encoded as nested objects (`params.filter.lp.cutoff_hz`), not a flat string-keyed dict. The schema models the canonical structure directly.
3. **Optional sections for cross-device flexibility.** `osc.2`, `osc.sub`, `noise`, `envelope.filter`, and `voice.glide_ms` are all optional. A minimal Prophet-5-style instance and a Muse-style instance both validate without forcing irrelevant params.
4. **Common-core only.** A canonical param exists only if it can be sensibly mapped to most of the surveyed devices. Anything that requires deep per-device interpretation lives in the backlog, not the schema. See "What is not here" below.
5. **`x-unit` annotation.** A custom JSON Schema keyword (validators ignore unknown keywords) carries the physical unit. Future codegen will use this to emit Pydantic / TypeScript types with the right unit semantics. The keyword is not load-bearing for v0.1 validation.

## Common-core LFO destination vocabulary

The `ModulationDestination` enum in the schema is exactly seven values:

| Destination | Acoustic meaning | Why common-core |
|---|---|---|
| `osc.pitch` | LFO modulates oscillator pitch — vibrato. | Every surveyed device exposes this. |
| `osc.shape` | LFO modulates wave shape — texture/timbre wobble. | All eight devices. |
| `osc.pwm` | LFO modulates pulse width — classic PWM movement. | All eight devices when shape == pulse. |
| `osc.amp` | LFO modulates oscillator amplitude — tremolo at the source. | All eight devices. |
| `filter.cutoff` | LFO sweeps the LP cutoff — wah/swell movement. | Every surveyed device exposes this; it is the canonical "filter wobble". |
| `filter.resonance` | LFO modulates resonance — uncommon but musical. | Six of eight devices expose it directly; the others (Prophet-5, OB-X8) reach it via mod matrix. |
| `amp.level` | LFO modulates final VCA — output tremolo. | All eight devices, often as a hard-wired LFO destination. |

Full mod-matrix destinations (envelope-as-source, mod wheel, aftertouch, per-osc-pitch, FX sends, etc.) are deferred — see the backlog.

## Per-section walkthrough

### Oscillators (`params.osc`)

`osc.1` is required; `osc.2` and `osc.sub` are optional. Each oscillator has `shape`, `level`, `detune_cents`, `octave`, and `pulse_width_pct` (used only when `shape == 'pulse'`).

Mapping notes across the surveyed devices:
- **Prophet-6, Prophet-5, OB-X8, Moog Muse:** two analog VCOs; `shape` enums map directly. PWM is per-osc continuous.
- **JUNO-X (Analog Synth):** two virtual analog oscillators. JUNO-X exposes more shape variants (e.g. `SAW-DW`) — these collapse to `saw` for canonical purposes.
- **PolyBrute 12:** two oscillators with extra wave-shaping (Brute / Metalizer / Ultrasaw). The wave-shaping params are out of common-core; the oscillator's underlying shape still maps to `shape`.
- **Minilogue XD:** two VCOs, plus the Multi-Engine which is excluded from this ontology entirely.
- **Novation Summit:** three NCOs per voice. Common-core uses only osc.1 and osc.2; the third oscillator is captured in the survey but not in the schema.

### Noise (`params.noise`)

Optional. `color` is `white` or `pink`. Devices that only produce one color set `color` to whichever they ship.

### Filter (`params.filter.lp`)

Required. The schema models a single low-pass filter with `cutoff_hz`, `resonance`, `envelope_amount` (signed), `key_tracking`, `drive`. Multi-mode filters (LP/HP/BP/Notch on OB-X8 / Summit) and dual filters (PolyBrute 12) are out of common-core — the canonical schema treats them as their LP setting.

### Envelopes (`params.envelope`)

`envelope.amp` is required; `envelope.filter` is optional. Both are ADSR with attack/decay/release in ms and sustain as a 0–1 ratio. Devices with a third "modulation envelope" (Moog Muse, PolyBrute 12) leave that envelope unrepresented in the canonical params; analysis-side outputs simply do not target it.

### LFO (`params.lfo.1`)

One LFO is required: `rate_hz`, `shape`, `depth`, `target`. Devices with multiple LFOs (JUNO-X, Muse, PolyBrute 12, Summit) expose only their primary LFO through this canonical slot for v0.1.

### Voice and master (`params.voice`, `params.master`)

`voice.mode` is required and is one of `mono`, `poly`, `unison`. `voice.glide_ms` and `voice.unison_voices` are optional. `master.volume_db` is required, in dB referenced to the device's nominal output.

## What is not here, and why

The "common-core" rule deliberately omits features that some devices have but others do not. These all have stub entries in the [backlog](../docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md):

- **Cross-modulation, hard sync, ring modulation, oscillator FM** — present on PolyBrute 12, JUNO-X Analog Synth, Muse, OB-X8, Summit; absent on Prophet-6, Minilogue XD, Prophet-5 (without poly-mod). Including them would force per-device fallbacks into the schema.
- **Dual filters in series/parallel/morph (PolyBrute 12), multi-mode filter (OB-X8, Summit)** — modeled as a single LP for v0.1.
- **Multiple LFOs and full mod matrix** — only `lfo.1` and the seven canonical destinations for v0.1.
- **Third (modulation) envelope** — analysis side does not yet predict it.
- **Vintage-knob noise simulation, Brute wave-shaping, NCO algorithms** — device-specific flavor that does not survive translation.
- **Effects bus parameters (chorus, delay, reverb, EQ)** — post-engine, belongs in a separate fx ontology.
- **Aftertouch / poly-aftertouch routing, velocity-amount params, per-voice modulation** — analysis-side cannot yet identify these from audio reliably.

When the inverse-synth pipeline matures past its first iteration and downstream consumers (translators, codegen, fixtures) start asking for these features, version-bump the schema and pull from the backlog.
````

- [ ] **Step 2: Verify required sections**

```bash
grep -c '^## ' parameter-mapping/subtractive-ontology.md
```
Expected: `4` (Design rules, Common-core LFO destination vocabulary, Per-section walkthrough, What is not here).

- [ ] **Step 3: Verify the file references the schema and the surveys**

```bash
grep -c 'subtractive.schema.json' parameter-mapping/subtractive-ontology.md
```
Expected: `>= 2`.

- [ ] **Step 4: Commit**

```bash
git add parameter-mapping/subtractive-ontology.md
git commit -m "Add subtractive-ontology.md prose companion"
```

---

### Task 12: Cross-check sweep — every canonical param hits ≥3 surveyed devices

**Files (reference, possibly edit):**
- Read: `parameter-mapping/survey/*.md`, `parameter-mapping/subtractive.schema.json`, `parameter-mapping/subtractive-ontology.md`.
- Edit (if gaps found): any of the above.

This step enforces the spec's definition of done: "sanity-check each canonical param against ≥3 of the surveyed devices." The output is a coverage matrix appended to `survey/README.md` and any inline corrections to schema or surveys triggered by gaps the sweep finds.

- [ ] **Step 1: Build a working coverage matrix**

In your scratch buffer (or a temp file outside git), build a table with one row per canonical param and one column per device. Mark each cell:
- ✓ — the device exposes the param directly (per the survey).
- ≈ — the device exposes a near-equivalent (within tolerance — e.g. a `cutoff_hz` exposed only as raw 0–127 still counts).
- ✗ — the device does not expose it.

Canonical params to check (one row each):
- `osc.1.shape`, `osc.1.level`, `osc.1.detune_cents`, `osc.1.octave`, `osc.1.pulse_width_pct`
- `osc.2.*` (each of the same five) — note that `osc.2` is optional so absence is fine
- `osc.sub.octave`, `osc.sub.level` — optional
- `noise.color`, `noise.level` — optional
- `filter.lp.cutoff_hz`, `filter.lp.resonance`, `filter.lp.envelope_amount`, `filter.lp.key_tracking`, `filter.lp.drive`
- `envelope.amp.*` (4 params), `envelope.filter.*` (4 params, optional)
- `lfo.1.rate_hz`, `lfo.1.shape`, `lfo.1.depth`, `lfo.1.target`
- `voice.mode`, `voice.glide_ms`, `voice.unison_voices`
- `master.volume_db`

- [ ] **Step 2: Identify gaps and decide per-row**

For each canonical param, count ✓ marks across the eight devices. Apply this rule:

- **≥ 3 ✓:** keep as required (or as currently declared).
- **2 ✓:** demote to **optional** in the schema (move out of any `required` list). Add a note in `subtractive-ontology.md`'s "Per-section walkthrough" explaining the demotion.
- **0–1 ✓:** **remove** the param from the schema entirely and add a stub entry to the backlog explaining why it was pulled.

For canonical params where a device exposes the *concept* but with non-equivalent units (e.g. cents vs semitones, dB vs raw), count ≈ as ✓.

- [ ] **Step 3: Apply changes**

If any param needed demotion or removal:
- Edit `parameter-mapping/subtractive.schema.json` to update `required` arrays or remove `$defs` entries / properties.
- Edit `parameter-mapping/subtractive-ontology.md` to reflect the new shape.
- For removed params, append a stub to `docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md` under a new heading `## Demoted / removed during cross-check`.

If no changes are needed, skip to Step 5.

- [ ] **Step 4: Re-verify the schema is still valid JSON**

```bash
python3 -m json.tool parameter-mapping/subtractive.schema.json > /dev/null
```
Expected: exit 0.

- [ ] **Step 5: Append the coverage matrix to `survey/README.md`**

Add a new section at the end of `parameter-mapping/survey/README.md`:

```markdown
## Coverage matrix (cross-check sweep)

Each canonical param vs. the eight surveyed devices. ✓ = direct support, ≈ = near-equivalent, ✗ = absent.

| Canonical param | Prophet-6 | JUNO-X | Prophet-5 | OB-X8 | Muse | PolyBrute 12 | Minilogue XD | Summit | Count |
|---|---|---|---|---|---|---|---|---|---|
| osc.1.shape | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 8 |
| ... | | | | | | | | | |
```

Fill in every canonical param row from Step 1 with the marks from Step 2. The Count column is the total ✓+≈ across the eight columns.

- [ ] **Step 6: Verify the matrix is complete**

```bash
grep -c '^| ' parameter-mapping/survey/README.md
```
Expected: increased by at least 28 lines (≈ one per canonical param row + header + separator). The exact count depends on how many params survive the sweep.

- [ ] **Step 7: Commit**

```bash
git add parameter-mapping/survey/README.md parameter-mapping/subtractive.schema.json parameter-mapping/subtractive-ontology.md docs/superpowers/specs/2026-05-02-subtractive-ontology-backlog.md
git commit -m "Cross-check sweep: coverage matrix + any demotions/removals"
```

(If only `survey/README.md` changed because nothing was demoted, the commit is just that one file — `git add` ignores files that have no changes.)

---

## Self-Review Notes

This plan covers every requirement in the MVP spec:

- **Eight survey docs** — Tasks 2–9, one per device, each with the same template structure and explicit sourcing.
- **`subtractive.schema.json` covering common-core subtractive params** — Task 10, with the full schema content embedded.
- **`subtractive-ontology.md` companion** — Task 11, prose walkthrough plus "what is not here" cross-references to the backlog.
- **Common-core boundary enforced** — Task 12 cross-check applies the ≥3-device rule that the spec implies, with explicit demotion/removal procedure.
- **Verification path matches Q5-A (pure design review)** — JSON parse + structural greps, plus a manual schema-draft compliance read in Task 10 Step 5. No fixtures, no validator-backed examples.

Out-of-scope items from the spec are not implemented and not implicitly pulled in: no codegen, no Pydantic / TS bindings, no translators, no fallback policy, no Phase 3 expert-plan updates.
