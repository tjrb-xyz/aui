# Device model: Roland JUPITER-Xm editor

**Status:** draft 1, 2026-10-01. This is design only; nothing is built. It answers the owner's request in `aui-docs/IDEAS.md` (2026-10-01): "model one for the Juliter XM. As It's current editor and tooling is very annoying."

**What this is:**

- a device model for an editor that drives the owner's real Jupiter-Xm over USB MIDI and SysEx;
- its views, built from aui parts;
- the external pack that styles them.

It is mazika's M1.2 ("the Jupiter-Xm synth manager") seen from aui's side.

**Evidence labels:**

- **confirmed:** from a source read directly.
- **snippet:** seen only in search snippets of Roland's PDFs, which the research proxy blocked.
- **UNVERIFIED:** no source yet; it must be checked on the owner's synth or in Roland's MIDI Implementation PDF.

**Main sources:**

- Roland's JUPITER-X/Xm MIDI Implementation, Parameter Guide and Owner's Manual (snippets only).
- Reviews: Sound on Sound, MusicRadar, Attack.
- The midi.guide data (github.com/pencilresearch/midi).
- github.com/DrKnackerator/RolandZenDecodeXML, which decodes the parameter database shipped with Roland's own editor and predates firmware 3.0.

**Data rule:** Roland's editor database is Roland's file. We read it to learn the structure but never copy or ship it. The profile below is written from the public MIDI Implementation and checked on the hardware.

---

## 1. What the editor fixes

The complaints about the current tools (snippets from Sweetwater, gearnews and Gearspace, and the owner's own words):

| Complaint | This model's answer |
|---|---|
| Menu diving on a tiny LCD; edit modes are confusing | Every parameter of the selected part sits on one screen, sectioned like a panel; no hidden pages for the common controls (§4.2) |
| Roland's JUPITER-X Editor UI is oversized and needs constant scrolling | Pages fit the screen and never scroll (mazika ux-spec §6.4.4 "**No page scrolls.**"); a compact view and a performance view for small screens |
| The editor crashes while rearranging; its librarian lagged behind firmware 3.0 | The librarian is our own: files on disk, search, tags, A/B, backups (§4.7). No dependence on Roland Cloud Manager |
| Two-way SysEx is unreliable | A tolerant sync layer: echo guard, value parsing that accepts the synth's quirks, a visible state per control, and a "Read from synth" resync (§3) |
| The whole Scene is hard to see at once | A Scene overview: 5 part strips side by side, like a mixer (§4.1) |

**The owner should confirm or reorder this list** (IDEAS, open question). The first build follows their top annoyance.

---

## 2. The device model

### 2.1 Structure

**Scene.** A Scene is what the synth calls up; there are 512 user Scenes on firmware 3.0+ (synthanatomy, confirmed). It holds:

- **Parts 1–4: synth parts.** Each holds one Tone, and each Tone uses one engine (§2.2). DUAL layers parts 1 and 2 (snippet).
- **Part 5: the drum part,** which holds a drum kit.
- **Per-part settings:** level, pan, key range, octave, MIDI channel, the arpeggio / I-Arpeggio / step-sequencer mode per part (firmware 3.0), MFX and Part EQ.
- **Shared effects:** Overdrive, Chorus (4 types), Delay (5), Reverb (7). Chorus can feed Reverb. There is also mic input processing (noise suppressor, compressor), vocoder / Vocal Designer, and a Master EQ and compressor (Sound on Sound, Kraft Music; snippet).
- **Arpeggio settings:** arpeggio and I-Arpeggio (65 types and 65 rhythms on 3.0, snippet), and the step sequencer per part with Probability (3.0).

**System.** Global settings such as MIDI, Edit Tx and the device ID. These are never part of a Scene.

**Libraries:** 512 user Tones (3.0+), the preset banks and the Roland Cloud expansions.

### 2.2 Engines (a Tone's kind)

The engines differ radically, so the model treats **engine kind** as a field. Each kind has its own parameter set and its own panel.

| Kind | Models | Shape (from the decoded editor database) |
|---|---|---|
| `analog` | JP-8, JX-8P, JUNO-106, SH-101, JUNO-60 (enum seen; firmware UNVERIFIED), JUPITER-X model (v3.0, 4 oscillators; layout UNVERIFIED) | One flat block. A `model` byte selects which subset of parameters is in use. Contents: <br>• OSC1–3: wave, feet, coarse ±48, fine ±50, PW, PWM <br>• noise <br>• mod mode (off, sync, ring, x-mod) <br>• mixer <br>• HPF <br>• filter freq/reso 0–1023 with env depth and key follow <br>• amp <br>• ENV1/ENV2 ADSR 0–1023 <br>• LFO with 11 waves <br>• portamento <br>• key mode (poly, solo, unison, solo-unison) <br>• drift and condition |
| `zen` | ZEN-Core / PCM / XV-5080-style tones | Common, MFX and a partial mix table, plus 4 partials. Each partial has: <br>• oscillator type: PCM, VA, PCM-Sync, SuperSAW or noise <br>• filter type <br>• pitch, filter and amp envelopes <br>• 2 LFOs with 16-step patterns <br>• EQ <br>• a matrix-control section |
| `drum` | Drum kits | Kit common, MFX and 6 compressors, plus 88 key partials (instrument, level, pan, sends, mute group) and 88 EQs |
| `jd800` | JD-800 Model Expansion (paid, Roland Cloud) | Common block, partials (2 LFOs, pitch/TVF/TVA envelopes), and its own effect block |
| `vocal` | Vocal Designer (expansion) | About 11 parameters: algorithm, carrier, mic and vocoder levels |

The editor offers a kind only if the synth reports it as installed (expansions are optional). **UNVERIFIED:** how the synth reports installed expansions over MIDI.

### 2.3 A parameter descriptor

Each editable value is described as data. This extends mazika's synth-profile format (ux-spec §6.7) with a SysEx address:

```json
{
  "id": "analog.filter.freq",
  "label": "Cutoff",
  "group": "Filter",
  "kinds": ["analog"],
  "models": ["JP8", "JX8P", "JUNO106", "SH101", "JUNO60"],
  "address": { "area": "temp.part", "offset": "00 00 4F" },
  "size": 4,
  "encoding": "nibble4",
  "range": [0, 1023],
  "display": { "kind": "number" },
  "cc": 74,
  "hero": true
}
```

- **`address`** is relative to an area such as "temporary tone of part *n*". The base of each area comes from one table. This keeps the 4-byte map in one place.
- **`encoding`:**
  - `byte7` (0–127);
  - `nibble4` (a wide value split over 4 nibbles; Roland's "L4");
  - `signed` with an offset.
- **`models`** lets one analog block serve all analog models: the panel shows only the parameters the selected model uses.
  - This is the device model deciding what exists, not a skin hiding controls, so it keeps aui's "a skin never hides a control" rule (AUI-182, mazika DJ design §4.6).
- **`cc`:** only a few panel controls have CCs. midi.guide lists:
  - 71 resonance, 72 release, 73 attack, 74 cutoff, 75 decay
  - 76–78 vibrato
  - 84 portamento
  - 91 reverb, 93 chorus
  - the usual 1, 5–7, 10, 11 and 64–68
  
  No NRPNs are documented. Everything else is SysEx.

The full parameter table is an **open task**: write it from Roland's MIDI Implementation PDF (not from Roland's editor files), then check each row on the synth.

---

## 3. Talking to the synth

**Messages (snippet + GitHub sources):**

- **Header:** `F0 41 <dev> 00 00 00 65 <cmd> aa aa aa aa … sum F7`.
  - The model ID is `00 00 00 65` and the default device ID is `10`.
  - `cmd` is `11` (RQ1, request) or `12` (DT1, data set).
- **Checksum:** Roland's usual rule, (128 − (sum of address and data bytes) mod 128) mod 128. This is the general convention, UNVERIFIED for this model.
- **Temporary Scene:** starts at `01 00 00 00`. A request for it returns each part's model type (snippet).
- **A known DT1 example** sets the analog filter frequency at `02 10 00 4F`. The base address of each part's temporary tone is UNVERIFIED.
- **Large dumps** go in packets of 256 bytes or less, about 20 ms apart (snippet).
- **Scene and Tone selection** (snippet):
  - Scenes: bank select MSB 85, LSB 0–3 + program change (512 Scenes).
  - User Tones on parts 1–4: MSB 71, LSB 3.
  - JUPITER-X model tones: MSB 97, LSB 81–82.
  - Other banks are UNVERIFIED.

**Rules** (mazika's, carried over: HANDOFF M1.2 and ux-spec §6.4.5 and §6.4.6):

- **Edit buffer only.** The editor writes only the temporary areas. Writing user memory needs the **"Save to synth…"** confirmation. The confirmation names the slot it overwrites and offers to back up the old contents to a file first.
- **Editor → synth:** at most one message per control per 10 ms, the latest value wins, and the final value is always sent.
- **Synth → editor:** with System "Edit Tx" on, panel moves arrive as DT1 messages (snippet).
  - An incoming value equal to one we sent in the last 100 ms is an echo and is ignored.
  - The parser accepts the synth's quirks, such as nibble values that reportedly read as zero in some receivers.
- **Control states** (ux-spec §6.4.5): Heard, Not heard, Pending send, Turned on the synth, Not modelled. Each is shown on the control.
- **Resync:** `Read from synth` requests the temporary Scene and every part's Tone (RQ1).
  - It runs on connect and after a Scene change on the synth.
  - Any time drift is suspected, it is one button.
- **Bluetooth MIDI** (Xm has BLE-MIDI) works for notes and CCs but is slow for dumps. The editor prefers USB and says so when only Bluetooth is connected.
- **Not in undo:** parameter changes belong to the synth, not to the console's undo history (ux-spec §6.4.5). The editor keeps its own A/B compare and "revert to last read".

---

## 4. Views

All views are built from aui parts. They are generated from the device model, so a firmware update that adds parameters needs only a profile update, not new screens. The pack (§5) styles them; it cannot add, hide or move behaviour.

### 4.1 Scene overview (the default page)

- **5 part strips side by side, like mixer channels.** Each strip shows:
  - the part number;
  - engine kind and model (e.g. `Analog · JP-8`);
  - Tone name;
  - level fader, pan, mute/solo;
  - key range (a small keyboard bar);
  - octave;
  - arp/step mode;
  - MFX name;
  - sends to chorus, delay and reverb;
  - a part activity LED (notes on that channel).
- **Above the strips:** Scene name, the Scene picker and DUAL / SPLIT.
- **Click a strip** to open its part editor (§4.2).

### 4.2 Part editor (the front panel)

One screen per engine kind:

- **`analog`:**
  - Sections: OSC 1–3, Mod (sync, ring, x-mod), Mixer, HPF, Filter, Amp, ENV 1, ENV 2, LFO, Voice (key mode, portamento, drift).
  - ENV sections show an ADSR curve drawn from the values. LFO shows its wave shape.
  - Switching the model (JP-8 → SH-101) re-forms the panel to that model's parameters.
- **`zen`:**
  - Common and Matrix Control sections.
  - 4 partial tabs (with a "compare all 4" table view), each with Osc, Filter, Amp, 3 envelope curves, 2 LFOs with step patterns, and EQ.
- **`drum`:**
  - An 88-key map as a grid of pads (by note).
  - Selecting a pad edits its instrument, level, pan, sends and mute group.
  - Kit-wide compressors and MFX.
- **`jd800`, `vocal`:** their own sections, shown only when the expansion is installed.
- **Every part editor also has:** the part's MFX (type + its parameters) and Part EQ.

### 4.3 Effects and routing

- **The Scene's effect chain as a signal flow:** parts → MFX → Part EQ → sends → Overdrive / Chorus / Delay / Reverb → Master EQ / compressor → out.
- **Mic in** feeds noise suppressor → compressor → vocoder / Vocal Designer.
- **In aui terms,** this is the synth's internal bus view. The console view (IDEAS 2026-09-29) shows it as the synth's own patchbay.
- **What can be patched:** the Jupiter-Xm's routing is fixed. The patchable parts are its sends, levels and on/off switches.

### 4.4 Arpeggio and step sequencer

- **Per part:** mode (off, arp, I-Arp, step), type, rhythm and motif.
- **The step sequencer** is a row of step keys with Probability, using aui's parameter-button painting gesture (AUI-147).

### 4.5 Performance view (phone, tablet, on stage)

- Scene picker (large).
- 4–8 macro knobs (owner-chosen parameters, saved per Scene in the editor's library, not on the synth).
- Keys, pitch and mod wheels.
- Part mute buttons.
- Arp on/off.

### 4.6 Compact view and back panel

- **Compact tile:** Scene name, 5 part LEDs, connection state (USB / Bluetooth / not connected), and 4 macros.
- **Back panel:** generated from the synth's declared I/O: USB (audio + MIDI), MIDI in/out, L/R outputs, phones, mic in, pedal jacks and Bluetooth.
  - **The exact jack list is UNVERIFIED** and must be checked against the owner's unit.
  - In mazika, the synth's audio arrives as a source through USB audio or an interface input (ux-spec §6.6.3).

### 4.7 Librarian

- **Browse and search:** all 512 user Scenes and Tones, plus presets, with tags, favourites and search by engine/model.
- **Comparing:** A/B and "audition without saving".
- **Backups:** full backup and restore to files on disk, in the portable per-user folder (IDEAS 2026-09-30). The editor never depends on Roland Cloud Manager.
- **Writing to the synth** always goes through "Save to synth…".

---

## 5. The pack (the look)

- **The pack styles the faces only:**
  - panel colours and materials;
  - knob caps, section legends, LED colour;
  - the LCD window (glass, as on real gear).
- **Two layers kept apart:**
  - **The device profile** (§2 parameter table, §3 messages) is device data. It ships with the editor, because it is the editor's function.
  - **The skin pack** is the external, editable, text-only look (IDEAS 2026-09-30).
  - A user can swap skins without touching the profile.
- **Homage, not replica, in what we ship.** No Roland logos, product names on the panel, or copied panel graphics. Naming "Jupiter-Xm" to say which synth the editor drives is fine (mazika brief §1a).
  - The first pack gets its own name.
  - A closer replica can only ever be a user-made external pack (IDEAS 2026-09-30).
- **No plugin.** This pack is look-only: the sound is the real synth.
  - A later software synth (the instrument-plugin idea) can reuse the same faces and the same parameter-descriptor shape.

---

## 6. aui parts this needs

- **Already in the requirements:**
  - Knob, Fader, Toggle, Segmented
  - ParamButton (painting)
  - Keys, Meter
  - InfoView
- **New:**
  - Envelope curve (ADSR and multi-stage, read-only drawing from parameter values)
  - LFO shape indicator
  - Step-pattern row (LFO steps, sequencer)
  - Part strip (Scene overview)
  - Key-range bar
  - Pad grid (drum map)
  - Signal-flow diagram (effects chain)
  - LCD/display window (glass material)
  - Librarian list with tags
  - Control-state marks (heard, pending, turned on the synth, not modelled)

---

## 7. Unknowns and checks on the owner's synth

1. The full address map: the top-level areas and each part's temporary-tone base. Read it from the MIDI Implementation PDF (v3.x) on a network that can reach static.roland.com, then check it on the synth.
2. The JUPITER-X model (firmware 3.0) block layout, which the decoded editor data predates.
3. Firmware version on the owner's unit, and whether anything after 3.x exists (none found).
4. How installed expansions (JD-800, Vocal Designer) are reported.
5. Whether "Edit Tx" sends every panel move, for every engine and edit mode.
6. The nibble-value quirk on received messages.
7. The exact rear-panel jack list.
8. Whether program change for preset banks behaves as the snippets say.

These belong on mazika's M1.2 hardware checklist ("Jupiter-Xm: its checklist comes with M1.2", mazika@36bde22:docs/HANDOFF.md).

## 8. Open questions for the owner

1. Which annoyance first: Scene overview, part editing without menu diving, reliable two-way sync, or the librarian and backups?
2. Which engines do you use most? JP-8 / JUNO / SH-101 analog models, ZEN-Core tones, drums, or the expansions? The part editor for those comes first.
3. Do you own the JD-800 or Vocal Designer expansions?
4. Does the editor run in mazika only (M1.2), or also as a standalone app or a plugin in other DAWs?
5. What firmware is your Jupiter-Xm on?
