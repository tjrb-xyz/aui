# Device model: Roland JUPITER-Xm editor

**Status:** draft 2, 2026-10-01. Design only; nothing is built. The owner's decisions are quoted in full in `README.md`.

**What this is:**

- a device model for an editor that drives the owner's real JUPITER-Xm over USB MIDI and SysEx;
- its views, built from aui parts;
- the external pack that styles them.

**What it is not:**

- **It makes no sound.** The sound comes off the Jupiter-Xm.
- **It is not yet the synth controller device.** That device, with the Jupiter-Xm as one of its models, follows once dsper is wired (`README.md`, roadmap).

Inside mazika this is M1.2, "the Jupiter-Xm synth manager" (mazika@36bde22:docs/HANDOFF.md §4).

**Evidence labels:**

- **MI:** read directly in Roland's *JUPITER-X/Xm MIDI Implementation* Version 1.06 (Apr 19, 2022, 90 pages). This is the newest edition found.
  - Roland's site was unreachable from the research network. The PDF was read from a public mirror (github.com/matchalunatic/jupiter-xm-midi-controller, `midi-spec.pdf`).
  - The PDF is Roland's document. This repo cites it but does not copy it.
- **editor DB:** seen only in Roland's own JUPITER-X Editor parameter database, as decoded by github.com/DrKnackerator/RolandZenDecodeXML and DrKnackeratorStrikesAgain/Roland-Zen-Decode-XML.
  - Used for structure only. It is Roland's file: it is never copied or shipped.
  - Where it disagrees with the MI, the MI wins.
- **snippet:** seen only in search-engine snippets of Roland's support pages.
- **UNVERIFIED:** no source. It must be measured on the owner's synth (§10).

---

## 1. What the editor fixes

**The owner's priorities, in order (2026-10-01):**

1. **See the whole Scene:** the Scene overview (§6.1).
2. **Edit its parts:** the part editor (§6.2).
   - The analog models come first.
   - Complete support of every engine is the goal. §2.2 says what the public SysEx allows.
3. **Save to the computer and reload with every parameter of every part:** Scene snapshots (§4).
   - Writing to the synth's own memory is secondary.

**Complaints and answers** (snippets from Sweetwater, gearnews and Gearspace, plus the owner's words):

| Complaint | This model's answer |
|---|---|
| The whole Scene is hard to see at once | A Scene overview: 5 part strips side by side, like a mixer (§6.1) |
| Menu diving on a tiny LCD; confusing edit modes | Every parameter of the selected part on one screen, sectioned like a panel (§6.2) |
| Saving and reloading an edited Scene is awkward | Snapshots on the computer that restore every part exactly, verified by reading back (§4) |
| Roland's JUPITER-X Editor UI is oversized and needs constant scrolling | Pages fit the screen and never scroll (mazika ux-spec §6.4.4 "**No page scrolls.**"); compact and performance views |
| Roland's editor crashes while rearranging; its librarian lagged behind firmware 3.0 | Our own librarian of snapshot files (§6.7); no dependence on Roland Cloud Manager |
| Two-way SysEx is unreliable | A tolerant sync layer: timeouts and retries, echo guard, partial-group merging, read-back after every recall, and a visible state per control (§3) |

**Prior art to beat:**

- **Roland's free JUPITER-X Editor** (May 2021, standalone, via Roland Cloud Manager; snippet). It has a librarian with "save to file", a Scene Builder and a Part Editor.
- **Sound Quest MIDI Quest** sells a Jupiter-X editor/librarian, including as AU/VST3/AAX plugins (snippet).

---

## 2. The synth as seen over SysEx

### 2.1 Structure

- **A Scene does not contain its Tones; it refers to them** (MI).
  - Each Scene Part stores its Tone's bank MSB, bank LSB and program number.
  - The Tone's data lives in a per-part **temporary Tone area**, one per engine (§2.3).
  - Roland's support confirms that writing a Scene does not save edited Tones (snippet).
- **The temporary Scene** (MI, 36 blocks, 12,490 data bytes):
  - **Scene Common.**
  - **5 Scene Parts**, each with:
    - part switch, mute, level, pan, tune;
    - mono/poly, legato, bend and portamento (each with a "TONE" option);
    - the Tone reference;
    - MIDI settings.
  - **Per part:** Part EQ, MFX and Zone.
  - **Shared effects:** Chorus, Delay, Reverb and Drive.
  - **Arpeggio:** settings per part, including Arp Mode (I-ARP / ARP / STEP), Step Seq Mode and Probability, which arrived in firmware 3.00.
  - **Five user arpeggio/step patterns,** 2,112 bytes each. These are 10,560 of the 12,490 bytes.
- **Parts 1–4 are synth parts; part 5 is the drum part.**
- **Only in System, not in a Scene** (MI):
  - Master EQ and Master Comp;
  - mic noise suppressor, compressor and sends;
  - the System chorus, delay and reverb;
  - the SCENE/SYS source switches that decide whether chorus, delay, reverb, tempo and controllers come from the Scene or from System.
- **Unreachable over SysEx** (editor DB marks them not SysEx-addressable):
  - mic input gain, master level;
  - Rx Exclusive, Tx Edit;
  - device ID.
  
  The editor labels these "set on the synth".
- **Storage counts** (firmware 3.00+; MI and snippet): 512 user Scenes and 512 user Tones.

### 2.2 Engines

| Engine | Models / content | Temporary area (per part) | Size per part | Public SysEx? |
|---|---|---|---|---|
| Analog model (`SynMdl`) | JUPITER-8, JX-8P, JUNO-106, SH-101 (MI `Model` field 0–4: `---, JP8, JX8P, JUNO106, SH101`) | `02 10 00 00` + part (parts 1–4) | 334 bytes, 3 blocks (parameters, MFX, Tone common) | **Yes (MI).** First priority. |
| JUPITER-X model (`MdlJPX`, firmware 3.00+) | Jupiter-8-inspired: 4 oscillators, 7 waveforms, osc pan and delay | `02 50 00 00` + part | 436 bytes, 3 blocks | **Yes (MI).** First priority. |
| ZEN-Core (`ZCore`) | PCM / XV-5080 / VA / SuperSAW tones with 4 partials | `02 00 00 00` + part | 2,124 bytes, 33 blocks | **Yes (MI)** |
| RD Piano | Piano tones | `02 20 00 00` (part 1 only) | 20 bytes | **Yes (MI)** |
| Vocoder | 2 preset tones (bank 71/70) | no temporary area found | — | **No:** captured by reference only |
| Drum kit (part 5) | 17 preset kits (bank 86/64) plus common kits | none in the MI. The editor DB names one, but its address is unknown | ~4,969 bytes (editor DB) | **No:** captured by reference, plus the Scene-level part settings |
| JD-800 (paid expansion) | JD-800 model | `02 30 00 00` + part (editor DB) | 566 bytes (editor DB) | **Experimental** until measured |
| Vocal Designer (expansion) | Vocal Designer | `02 40 00 00` (editor DB) | 178 bytes (editor DB) | **Experimental** until measured |

**JUNO-60 is not documented for this synth.**

- The MI's analog `Model` field stops at SH-101, and there is no JUNO-60 tone bank.
- The editor DB adds `JUNO60` as model 5.
- Draft 1 listed JUNO-60 as an analog model. That is withdrawn until the owner's synth shows it (§10).

**"Complete support of every engine" has a public-SysEx limit:**

- The analog, JUPITER-X, ZEN-Core and RD Piano engines are fully editable from the MI.
- The drum kit, vocoder and expansions are not documented. They are captured by reference (bank and program) and restored with their Scene settings.
- Going further means measuring undocumented areas on the owner's synth (§10), never copying Roland's editor files.

**Engine detection per part:**

- **Preset Tones:** the engine follows from the Tone's bank (MI bank table):
  - 97/64 JUPITER-8, 97/66 JUNO-106, 97/68 JX-8P, 97/70 SH-101;
  - 97/81–82 JUPITER-X;
  - 71/69 RD-PIANO, 71/70 VOCODER, 71/67 PR-X;
  - 87/xx ZEN-Core banks (PR-A…D, XV-5080, COMMON);
  - 86/64 drum kits.
- **User Tones** (71/3–6) do not reveal their engine. The editor reads the temporary area of each candidate engine and keeps the one that answers. Whether an inactive area answers with stale data, defaults or silence is UNVERIFIED (§10).
- **Analog models:** the `Model` byte of the analog block says which of the four it is.

### 2.3 Address map (MI)

| Address | Area |
|---|---|
| `00 00 00 00` | System (Common, Control, chorus/delay/reverb, Master EQ/Comp, mic, Model Bank Assign) |
| `00 10 00 00` | Setup (3 bytes: the current Scene's bank and program) |
| `01 00 00 00` | Temporary Scene |
| `02 00 00 00` – `02 03 00 00` | Temporary Tone, ZEN-Core, parts 1–4 |
| `02 10 00 00` – `02 13 00 00` | Temporary Tone, analog model, parts 1–4 |
| `02 20 00 00` | Temporary Tone, RD Piano (part 1) |
| `02 50 00 00` – `02 53 00 00` | Temporary Tone, JUPITER-X model, parts 1–4 (firmware 3.00+) |
| `40 00 00 00` – `49 7B 00 00` | User Scenes 1–256, one every `00 05 00 00` |
| `50 00 00 00` – `57 7E 00 00` | User Tones 1–512, one every `00 02 00 00`. These are opaque data blocks, not named parameters. |
| `60 00 00 00` – `69 7B 00 00` | User Scenes 257–512 (firmware 3.00+) |
| `7F 00 00 00` | "Editor". Undocumented in the MI. The editor DB names commands here, all UNVERIFIED: <br>• version request <br>• Scene and Tone lists <br>• "Notify Scene Load / Tone Load" <br>• "Edit State" <br>• a write message <br>• an Edit Tx override at `7F 00 7F 01` |

**Rules for using the map:**

- Addresses and sizes are 4 bytes of 7 bits each, so offsets carry at 128.
- **The capture and recall path only ever touches** `00 00 00 00`, `00 10 00 00`, `01 …` and `02 …`.
- **The storage areas** `40 …`, `50 …` and `60 …`, and any write command in `7F …`, are used only by "Save to synth…" (§3.3).

### 2.4 Message format and encoding (MI)

- **DT1 (data set):** `F0 41 <dev> 00 00 00 65 12 <a a a a> <data…> <sum> F7`.
- **RQ1 (request):** `F0 41 <dev> 00 00 00 65 11 <a a a a> <s s s s> <sum> F7`.
  - The model ID is `00 00 00 65`. The MI's reception tables misprint the last byte as `52`; ignore that.
  - The device ID is `10`–`1F` (default `10`); `7F` is also accepted.
- **Checksum:** `(128 − (sum of address and data or size bytes) mod 128) & 0x7F`.
  - The MI's example `F0 41 10 00 00 00 65 12 01 00 00 10 4A 25 F7` (temporary Scene level = 74) checks out.
- **Requests:**
  - An RQ1 must use a block start address and size from the map.
  - A wrong request gets **silence, not an error**, so every read needs a timeout and a retry.
- **Packets:**
  - Data over 256 bytes travels in packets of 256 bytes or less, about 20 ms apart. This holds both ways.
  - A long reply arrives as several DT1 messages, each with its own address and checksum, and is reassembled by address.
- **Values:**
  - A plain value is one 7-bit byte.
  - A value marked `#` is split into 4-bit nibbles, high nibble first. Groups are 2 bytes (8-bit values) or 4 bytes (16-bit values), never 3.
  - **The address of a `#` value is its group's first byte.**
- **Signed values are stored with an offset,** for example:
  - `FILTER KEY FOLLOW 824–1224` means −200…+200;
  - `FILTER ENV DEPTH 1–2047` means −1023…+1023.
  
  A few fields' signed encodings are unclear and need measuring (§10).
- **Other traffic:**
  - Filter Active Sensing (`FE`) on input, and never send it.
  - The synth has a second USB MIDI port for DAW control. SysEx uses the main port.

### 2.5 A parameter descriptor

Every editable value is data. This needs a v2 of mazika's synth-profile format (§9). The example is the analog model's filter frequency, from the MI's `[MDLSYN0]` table:

```json
{
  "id": "analog.filter.freq",
  "label": "Cutoff",
  "group": "Filter",
  "engine": "analog",
  "models": ["JP8", "JX8P", "JUNO106", "SH101"],
  "area": "temp.tone.analog",
  "offset": "00 00 4F",
  "encoding": "nibble4",
  "raw": [0, 1023],
  "display": { "kind": "number", "min": 0, "max": 1023 },
  "hero": true
}
```

- **`area`** names a base that depends on the part: `temp.tone.analog` for part *n* is `02 1(n−1) 00 00`. One table holds every base. The address is base + offset, with 7-bit carry.
- **`encoding`** is one of `byte7`, `nibble2` or `nibble4`. **`raw`** is the wire range, and **`display`** maps it to what the person sees, including signed offsets.
- **`models`** lists the analog models that use the parameter. The panel shows only those parameters.
  - This is the device model deciding what exists, not a skin hiding controls, so aui's "a skin never hides a control" rule holds (AUI-182; mazika DJ design §4.6).
- **No `cc` here.** The synth's CCs act on the *part*, not the Tone (MI):
  - CC 74 changes the Scene Part's "Cutoff Offset" relatively;
  - CC 71–73 and 75 do the same for resonance and the envelope.
  
  So they are performance offsets, not ways to set tone parameters. Draft 1 mapped CC 74 to the tone's filter frequency; that is withdrawn.
- **The full table is generated from the MI** (a parser over the MI's text, as the RoMi project does), not typed by hand, and then checked on the synth.

### 2.6 Firmware and identity

- **Versions** (snippet unless marked):
  - **3.03** (March 2025) is the newest found. Its only change: the Auto Off default becomes 20 minutes.
  - **3.02** (October 2022) reduced popping noise. **3.01** (May 2022) fixed problems connecting to Roland's editor.
  - **3.00** (April 2022) is the last release that changed parameters, engines or storage:
    - the JUPITER-X model;
    - 512 Scenes and Tones;
    - the new Scene block at `60 …`;
    - I-ARP 55→65 types and 44→65 rhythms;
    - per-part arp/step mode and Probability.
  - The MI v1.06 is dated the day 3.00 came out (MI), and no later edition is indexed.
  - **The owner runs 3.02 and plans 3.03.** One schema (MI v1.06) is expected to cover both. This is to be confirmed by measuring block sizes on each (§10).
- **Identity** (MI):
  - The Universal Identity Request `F0 7E 7F 06 01 F7` is answered with `F0 7E <dev> 06 02 41 65 03 00 00 <rev ×4> F7`.
  - The family `65 03 / 00 00` identifies a JUPITER-X or Xm.
  - The MI is inconsistent about the revision bytes: `01 01 00 05` vs `00 01 00 05`. They do not map reliably to "3.02" or "3.03", and they may or may not tell X from Xm.
  - So the editor:
    - stores the raw reply;
    - asks the person to confirm the version shown under MENU › INFORMATION, once per synth;
    - records both in every snapshot.
- **Auto Off** (3.03 default: 20 minutes) means an idle synth can switch itself off. The editor handles the port going away and coming back: identify again, re-read, and offer to resend the edit buffer.

---

## 3. Talking to the synth

### 3.1 Who talks to the synth

The owner's rule (2026-10-01): **"The web view should be separate from the main thread which should handle midi."**

- **The main side owns the conversation with the synth.** That covers:
  - the MIDI port;
  - SysEx framing, checksums and pacing;
  - timeouts and retries;
  - the echo guard;
  - the decoded device state;
  - snapshot capture and recall (§4);
  - every file.
  
  It runs whether or not a view is open. It is written once, as one "Jupiter core" library, and used in every host:
  - **mazika:** the core that owns the synth.
    - **Today** that is mazika's browser core page, using Web MIDI with SysEx permission granted in the core's own window. mazika's rule is "**A client never calls `requestMIDIAccess`.**" (ux-spec §6.6.2).
    - **Later** it is the native daemon mazikad, which needs a MIDI and SysEx design it does not have yet (§9).
  - **A DAW plugin:** the plugin's native, non-audio thread, which **opens the synth's USB MIDI port itself** (§5). It never uses the audio thread, the host's MIDI path or the editor's web view.
- **The web view is a client.** It renders aui, holds a replica of the state and proposes edits. It never opens a MIDI port or a file.
- **The research supports this:**
  - WebKit has no Web MIDI, so neither does WKWebView (macOS and iOS hosts) nor WebKitGTK (Linux): confirmed in the WebKit source tree and browser-compat data.
  - WebView2 (Windows) has Web MIDI only behind a SysEx permission.
  - A web view that owned MIDI would fail on two of three desktop systems and would lose the synth whenever the editor window closed.

### 3.2 Messages

The same contract serves mazika and the plugin, built on mazika's synth session messages (ux-spec §6.4–§6.6):

- **Client → main side:**
  - `synth.set {param, value}`: one edit;
  - `synth.do {action}` with these actions:
    - `read` (read everything from the synth);
    - `capture` (save a snapshot);
    - `send {snapshot}` (recall into the edit buffer);
    - `import` / `export`;
    - `save-to-synth {slot}` (behind its confirmation).
- **Main side → client:**
  - `synth.state`: the decoded state, sent as per-area changes rather than whole snapshots;
  - progress and results for long actions;
  - structured errors (timeout, checksum, partial reply, port lost).
- **Ownership:** `synth.claim`, `synth.claimed`, `synth.refused` and `synth.released`, mazika's existing rule that the first claim wins (§5.3).
- **mazika already uses "scene" for jam scenes** (`scene.recall`). So the messages say `synth.*`, and mazika's UI calls a saved Scene snapshot a **sound** (its glossary word). "Scene" appears only when naming the Jupiter's own concept.

### 3.3 Rules

**Carried over from mazika** (HANDOFF M1.2; ux-spec §6.4.5, §6.4.6):

- **Edit buffer only.** Editing and recall write only the temporary areas.
- **Writing the synth's memory needs the "Save to synth…" confirmation:**
  - it names the slot it overwrites;
  - it offers to back up the slot's old contents to a snapshot first;
  - it reminds the person that edited Tones must be saved with their Scene ("Scene & Tone"), since a Scene only refers to its Tones.
- **Editor → synth:**
  - at most one message per control per 10 ms;
  - the latest value wins;
  - the final value is always sent.
- **Synth → editor:**
  - With **Tx Edit** on, panel moves arrive as DT1 messages (snippet). They arrive at nibble-group start addresses, so partial groups are merged before decoding.
  - Tx Edit cannot be set over documented SysEx. The editor either asks the person to set System › MIDI TX › Tx Edit = ON, or uses the editor area's override if measuring shows it works (§10).
  - An incoming value equal to one sent in the last 100 ms is an echo and is ignored.
- **Control states** (ux-spec §6.4.5), each shown on the control:
  - Heard;
  - Not heard;
  - Pending send;
  - Turned on the synth;
  - Not modelled.
- **Not in undo.** Synth edits stay out of the console's undo history. The editor has its own A/B compare and "revert to last read".

**Setup the editor checks and explains on first connect:**

- USB MIDI port found (on Windows, Roland's VENDOR driver mode where needed);
- System › MIDI RX › **Rx Exclusive = ON**; without it every RQ1 and DT1 is ignored;
- **Tx Edit = ON** for a live mirror of the synth's panel;
- the device ID, if not the default;
- the firmware confirmation (§2.6).

**Bluetooth MIDI** (the Xm has BLE-MIDI) carries notes and CCs but is slow for dumps. The editor prefers USB and says so.

---

## 4. Scene snapshots (save to the computer)

The owner's third priority: "saving it, not necessarily on synth memory but computer memory and being able to reload the save with all the parameters and parts we’ve edited."

### 4.1 What a snapshot holds

**Always:**

- **The temporary Scene:** all 36 blocks (MI).
  - The 5 user patterns (10,560 bytes) are included by default, so nothing is lost.
  - A "light" snapshot can skip them when they equal the defaults.
- **For each of parts 1–4,** the temporary Tone blocks of its *active engine* (§2.2):
  - 334 bytes for an analog model;
  - 436 bytes for the JUPITER-X model;
  - 2,124 bytes for ZEN-Core;
  - 20 bytes for RD Piano.
- **For each part,** its Tone reference (bank, program, name) and engine. For an analog Tone, also its model.
- **Part 5 (drums), the vocoder and expansion Tones:** the reference and every Scene-level setting (part, EQ, MFX, zone, arpeggio, pattern).
  - Their inner data is not in the public SysEx (§2.2).
  - JD-800 and Vocal Designer data are captured as *experimental* raw blocks once measuring confirms their areas.
- **Metadata:**
  - the identity reply;
  - the firmware the person confirmed;
  - the MI edition the schema follows (v1.06);
  - the editor schema version;
  - the Setup block (which stored Scene it came from);
  - the expansions it depends on;
  - name, time, tags, notes.

**Optional, as "System context":**

- System chorus/delay/reverb, Master EQ and Master Comp;
- mic processing;
- the SCENE/SYS source switches.

These affect every Scene, so they are saved for reference and **restored only when the person ticks "Also restore System settings".**

**Size:** about 12.5 KB for the Scene, plus up to 4 × 2.1 KB for Tones, plus about 1.6 KB of System. That is roughly 23 KB at most, or about 2 KB plus Tones without patterns. This fits a mazika sound, a plugin's saved state, or a file.

### 4.2 File format

- **One JSON text file per snapshot:** `<name>.jupiter-xm.json`. It is also mazika's sound export format (§9), so mazika and the plugin share one file.
- **`blocks`** is the authoritative part. It is a list of `{address, size, bytes}` with the size *the synth actually returned*.
  - Recall replays exactly what was read and never assumes sizes. The MI and the editor DB already disagree on some sizes (Arpeggio Part 61 vs 59 bytes; System Common 48 vs 47).
- **`values`** holds decoded values by name for the parameters the profile knows (for example `"part1.analog.filter.freq": 812`). They are for reading, comparing and editing in a text editor.
  - On load, a value that differs from its bytes is re-encoded into the block, and each such change is listed before sending.
  - Blocks the profile cannot decode stay opaque and round-trip unchanged.
- **Canonical form:**
  - The JSON is canonical: sorted keys, whole numbers only, bytes as hex.
  - The snapshot's **hash** is computed over the canonical form, so a take can refer to a snapshot by hash (§9).
- **`.syx` export:** a plain sequence of DT1 messages, for any SysEx librarian.
- **Roland's own formats** (`.SVD` backups and Scene exports, `.SVZ` Tones, the editor's librarian files) are undocumented. They are not written or read in this version.

### 4.3 Capture

1. Read Setup (`00 10 00 00`) and the temporary Scene, one RQ1 per block at the MI's start and size.
2. For each part, find the engine (§2.2) and read that engine's temporary Tone blocks.
3. Read System context.
4. Each read has a timeout and one retry. A block that never answers is reported by name. **A snapshot is never saved silently partial:**
   - the person either retries;
   - or saves it marked "N blocks missing". mazika's equivalent is "Saved with N not heard".
5. Show progress. A full capture is about 40–80 block reads (UNVERIFIED timing; §10).

### 4.4 Recall

1. **Check the snapshot's model and firmware against the connected synth.** A mismatch warns, then allows recall.
2. **For each part, select its Tone** by bank and program, and wait until the Tone has loaded. This uses a "Notify Tone Load" if measuring confirms one; otherwise a measured delay.
3. **Write that part's temporary Tone blocks.**
   - If a referenced *user* Tone slot now holds something else, the editor selects a factory Tone of the same engine and model as a "carrier", then writes the saved Tone data over it.
4. **Write the remaining Scene blocks.**
   - Scene Part blocks are split so the Tone selection is not sent again after the Tone data, which would reload the Tone and wipe the edit (UNVERIFIED behaviour; §10).
5. **Restore System context only if ticked.**
6. **Read everything back and compare.** Show "Restored · all parts match", or list each mismatch with a retry.
   - SysEx on this synth is known to be unreliable, so verifying is part of recall.
7. **Nothing is written to the synth's memory.** Keeping the Scene on the synth is "Save to synth…".

Paced at 256-byte packets about 20 ms apart, a full recall takes a few seconds. It runs on the main side, so the UI stays live and shows progress.

### 4.5 Confirming a recall (open)

mazika confirms every send of a sound ("… Send · Cancel", ux-spec §6.4.6). Draft 1 proposed no confirmation, since recall writes only the edit buffer.

**Default here: follow mazika.** A recall asks once, saying what it replaces:

- *"Replace the synth's current sound with 'Night Brass'? Unsaved edits on the synth will be lost. Send · Cancel"*;
- with "Save current first" offered when the edit buffer has unsaved changes.

Stepping through sounds with `Shift+,` / `Shift+.` uses the same rule (mazika §6.4.6). The owner may choose otherwise (README, open questions).

### 4.6 Where snapshots live

- **In the store of whichever main side owns the synth, never in a client** (mazika ux-spec §10.8: sounds are stored by the owning core).
  - **mazika:** the core's own store, which must survive new participant ids (§9). mazika does not have one yet; its SDK design proposes `mazika-device`.
  - **A DAW plugin:** the snapshot *is* the plugin's saved state, inside the DAW project (§5.2). Its library of other snapshots lives in the portable per-user folder:
    - macOS: `~/Library/Application Support/<vendor>/`
    - Windows: `%APPDATA%\<vendor>\`
    - Linux: `$XDG_DATA_HOME/<vendor>/`
- **Moving snapshots between computers or hosts** is export and import of the same `.jupiter-xm.json` file.

---

## 5. Running in other DAWs (plugin mode)

### 5.1 How the plugin reaches the synth

**Never through the DAW.** The plugin's native main side opens the synth's USB MIDI port directly. The research found host routing of plugin SysEx too unreliable:

- **Formats:** VST3, CLAP and AU can all carry SysEx in principle.
- **Hosts** (snippets and source, 2026):
  - Bitwig strips SysEx at the track and device level;
  - Live passes it only through Max for Live;
  - Logic's External Instrument does not pass it;
  - developers reported VST3 SysEx output failing in Cubase 12 and REAPER (a JUCE framing bug fixed in 2023; current hosts are untested).
- **clap-wrapper 0.16.0** forwards outgoing SysEx to AUv2 and AUv3 but drops it in its VST3 adapter (confirmed in its source).
- **Commercial editors with total recall do the same** (snippet): MIDI Quest, Soundtower and Ctrlr open their own MIDI ports.

**Opening the port alongside the DAW:**

- **macOS:** CoreMIDI is multi-client.
- **Linux:** use the ALSA sequencer (multi-client), never rawmidi (exclusive).
- **Windows:**
  - Windows MIDI Services makes ports multi-client by default and routes the older WinMM and WinRT APIs through itself (Microsoft's docs, 2026).
  - Ports are exclusive again in its "Legacy" mode, on Windows 10, and on unupdated systems. There the editor detects the mode and shows the known fix: disable the synth's port in the DAW.
  - Whether Roland's VENDOR driver joins the multi-client service is UNVERIFIED.

### 5.2 Total recall

- **The plugin's saved state** (CLAP state, VST3 `getState`, AU `fullState`) **is the Scene snapshot.**
- **On load, the main side schedules a paced recall and a read-back**, with or without the editor window open:
  - the recall waits until the synth's port appears;
  - CLAP's state-context tells a project load from a *duplicate*, so duplicating a track does not resend to the hardware.
- **Options**, as MIDI Quest offers:
  - recall on project load (default);
  - recall on first play;
  - manual.
- **A recall started during playback** finishes in the background and says so. It never touches the audio thread.

### 5.3 One owner per synth

**One synth has exactly one owner at a time**, whichever of these is running:

- several plugin instances;
- a host that runs plugins in separate processes (for example Bitwig's sandboxes);
- mazika and a DAW at once.

**How:**

- **The first claim wins** (mazika's `synth.claim` rule). The others show "Owned by …" and act as clients.
- **When mazikad runs,** a plugin can hand ownership to it over loopback, the way mazika's Max for Live device already acts as a session client of the core.
- **Across processes,** a lock keyed by the synth's port and identity enforces it.

### 5.4 Build

- **CLAP first,** wrapped to VST3 and AUv2 with clap-wrapper. Its VST3 SysEx gap does not matter, since SysEx bypasses the host.
- **AUv2 before AUv3,** until an AUv3 sandbox is shown to reach CoreMIDI hardware ports.
- **The Jupiter core** (§3.1) is a native library with a C interface, usable from a C++ plugin shell and from mazikad (Rust).
- **The editor UI is aui in the plugin's web view,** bridged to the core with the toolkit's native calls (choc's `bind`, JUCE's native functions, or equivalent).
  - The plugin's web view follows the older-WKWebView rules already in aui's requirements (AUI-073): no `light-dark()`, precomputed colours.
- **Pro Tools (AAX)** is out of scope at first (signing barrier; IDEAS 2026-09-30).

### 5.5 Echo loops

With multi-client ports, the DAW's MIDI track listening to the synth also receives the synth's Tx Edit SysEx. The editor's setup notes tell the person to turn off SysEx thru on that track.

---

## 6. Views

**All views are built from aui parts and generated from the device model.**

- A firmware change that adds parameters needs a profile update, not new screens.
- The pack (§7) styles the views. It cannot add, hide or move behaviour.

**Inside mazika, pages follow mazika's layout rules:**

- no scrolling;
- a 56 px unit;
- a 1024×576 reference size, 800×480 minimum.

### 6.1 Scene overview (the default page)

- **5 part strips side by side, like mixer channels.** Each strip shows:
  - part number;
  - engine and model (e.g. `Analog · JP-8`, `Drums · by reference`);
  - Tone name;
  - level, pan, mute/solo;
  - key range bar, octave;
  - arp/step mode;
  - MFX name;
  - sends to chorus, delay and reverb;
  - a note-activity LED.
- **Above the strips:**
  - Scene name and picker;
  - DUAL / SPLIT;
  - **Capture** (save a snapshot) and **Send** (recall);
  - the firmware and connection badge.
- **The SCENE/SYS switches** for chorus, delay, reverb, tempo and controllers sit next to the shared effects, so the person sees which settings actually apply.
- **Click a strip** to open its part editor.
- **At 800×480** the strips compress to name, engine, level and mute. Details are one tap away.

### 6.2 Part editor (the front panel)

- **Analog model:**
  - sections for OSC, Mod (sync, ring, x-mod), Mixer, HPF, Filter, Amp, ENV 1, ENV 2, LFO, and Voice (key mode, portamento, drift);
  - envelopes drawn as curves from their values; the LFO shows its wave;
  - switching the model (JP-8 → SH-101) re-forms the panel to that model's parameters.
  - Whether writing the `Model` byte switches the model, or only selecting a Tone from that model's bank does, is UNVERIFIED (§10).
- **JUPITER-X model:** its four oscillators, waveforms, osc pan and delay, plus the same filter, amp, envelope and LFO sections.
- **ZEN-Core:**
  - Common and Matrix Control;
  - 4 partial tabs, plus a "compare all 4" table;
  - per partial: Osc, Filter, Amp, three envelope curves, 2 LFOs with step patterns, EQ.
- **RD Piano:** its few parameters.
- **Drums, vocoder, expansions:**
  - the Scene-level part settings (level, pan, EQ, MFX, zone, arp) are editable;
  - the inner data shows "Not in the synth's public SysEx: saved by reference".
- **Every part also has** its MFX (type and parameters) and Part EQ.

### 6.3 Effects and routing

- **The signal flow:** parts → MFX → Part EQ → sends → Drive / Chorus / Delay / Reverb → Master EQ / Comp → out.
- **Mic in** → noise suppressor → compressor → vocoder.
- **Settings that live in System** are marked "System · affects every Scene".
- **Settings SysEx cannot reach** are marked "set on the synth".
- **In aui terms,** this is the synth's internal bus view (IDEAS 2026-09-29). Its routing is fixed; sends, levels and switches are editable.

### 6.4 Arpeggio and step sequencer

- **Per part:** Arp Mode (I-ARP / ARP / STEP), type, rhythm, motif and Step Seq Mode.
- **The step pattern** is a row of step keys with Probability, using aui's parameter-button painting gesture (AUI-147).

### 6.5 Performance view (phone, tablet, on stage)

- Scene picker, large.
- 4–8 macro knobs. The owner picks the parameters, saved per snapshot in the editor's library, not on the synth.
- Keys and wheels.
- Part mutes.
- Arp on/off.

### 6.6 Compact view and back panel

- **Compact tile:** Scene name, 5 part LEDs, connection (USB / Bluetooth / off / "Owned by …"), and 4 macros.
- **Back panel:** generated from the synth's declared I/O:
  - USB (audio + MIDI, two MIDI ports);
  - MIDI in/out;
  - L/R out, phones;
  - mic in, pedal jacks;
  - Bluetooth.
  
  The exact jack list is UNVERIFIED (§10). In mazika, the synth's audio enters as a source through USB audio or an interface input (ux-spec §6.6.3).

### 6.7 Librarian

- **Snapshots are the library's unit.** The library offers:
  - search by name, tag, engine and model;
  - favourites;
  - A/B compare;
  - "audition", which sends without saving;
  - a diff between two snapshots, by part and parameter, from their decoded values.
- **A full backup** reads every user Scene and every user Tone into a folder of files.
  - User Tones in synth memory are opaque blocks (§2.3), so a full backup restores bit-exact but is not editable parameter by parameter.
  - Editing a stored user Tone means loading it into a part first.
- **Writing to the synth** always goes through "Save to synth…".

---

## 7. The pack (the look)

- **The pack styles faces only:**
  - panel colours and materials;
  - knob caps, section legends, LED colour;
  - the LCD window (glass, as on real gear).
- **Two layers stay apart:**
  - **The device profile** (§2) is device data that ships with the editor, because it *is* the editor's function.
  - **The skin pack** is the external, text-only look (IDEAS 2026-09-30).
- **Homage, not replica, in what we ship.** No Roland logos, panel product names or copied panel graphics.
  - Naming "JUPITER-Xm" to say which synth the editor drives is fine (mazika brief §1a).
  - The first pack gets its own name.
  - A closer replica can only be a user-made external pack.
- **No plugin sound.** The sound is the real synth.
  - The later synth controller device (README, roadmap) can reuse the same faces and the same descriptor shape.

---

## 8. aui parts this needs

- **In aui's requirements already:**
  - Knob, Fader, Toggle, Segmented;
  - ParamButton (painting);
  - Keys, Meter;
  - InfoView.
- **New:**
  - envelope curve (ADSR and multi-stage, drawn from values);
  - LFO shape indicator;
  - step-pattern row (with probability);
  - part strip;
  - key-range bar;
  - pad grid;
  - signal-flow diagram;
  - LCD/display window (glass material);
  - librarian list with tags and diff;
  - control-state marks (heard, not heard, pending, turned on the synth, not modelled);
  - a progress-and-verify panel for long transfers;
  - "set on the synth" and "by reference" badges.

---

## 9. What mazika needs to change

The research read mazika@36bde22. These are mazika spec changes for mazika's session to make. They are listed here so nothing is lost.

1. **M1.2's target.** HANDOFF names the Jupiter-Xm, but ux-spec §6.7 and the research still build M1.2 around the Roland S-1. The spec needs the Jupiter-Xm profile.
2. **First-party, not an extension.** The extension SDK forbids SysEx (`midi.send` is 3 bytes), has no MIDI input, and caps storage. So the editor's main side is mazika core code (`src/synth/`), with a sound-design client view.
3. **Synth profile v2.** v1 (`mazika/synth-profile/v1`) assumes:
   - one channel;
   - CCs per parameter;
   - a flat list, with unknown keys refused.
   
   v2 needs:
   - SysEx areas with bases per part, offsets and encodings;
   - device ID and checksum rules;
   - 4 synth parts and a drum part with their own channels;
   - parameters that depend on the engine and model;
   - raw blocks outside the page-mapped list.
4. **The sound's shape.** A sound is a flat `values` array in profile order today. A Jupiter-Xm sound is the snapshot of §4: addressed blocks plus decoded values, in canonical JSON, hashed.
5. **Takes refer to sounds by hash.** `Take.synthSound` embeds full values today. Pointing to a content-addressed snapshot by hash avoids putting ~23 KB in every take and `take.announce` (ux-spec §10.4–§10.5).
6. **`synth.do` gains actions:** `read`, `capture`, `send`, `import`, `export`, `save-to-synth`.
7. **`synth.state` sends changes per area,** not whole value lists at 20 per second.
8. **A store that belongs to the core** and survives new participant ids. Sounds cannot live in the per-participant database.
9. **MIDI and SysEx in mazikad.** No design yet says how the native daemon owns MIDI on macOS, Windows and Linux. Until it does, the Jupiter core runs in mazika's browser core with Web MIDI. Whether a headless core can get the SysEx permission is UNVERIFIED (mazika §13.2).
10. **Words:**
    - "sound", never patch, program or preset;
    - "scene" stays for jam scenes;
    - no "Record" labels.

---

## 10. Checks on the owner's synth

These belong on mazika's M1.2 hardware checklist ("Jupiter-Xm: its checklist comes with M1.2"). Each takes minutes with a SysEx monitor.

1. **Identity reply** on 3.02, and again after the upgrade to 3.03. Record the four revision bytes.
2. **Block sizes returned on 3.02 and 3.03** against MI v1.06, especially:
   - Arpeggio Part (61 vs 59);
   - System Common (48 vs 47);
   - the JUPITER-X model block;
   - the 2,112-byte user pattern.
3. **What an RQ1 to an inactive engine's temporary area returns** (silence, stale data or defaults).
4. **Whether one RQ1 can read a whole area,** and how many requests can be in flight at once.
5. **Recall order:**
   - how long after a Tone selection the temporary Tone can be written;
   - whether re-sending a Scene Part reloads the Tone and wipes edits;
   - whether writing Setup loads a Scene.
6. **Whether writing the analog `Model` byte switches the model.**
7. **Tx Edit:**
   - what a panel move sends, per engine;
   - whether whole or partial nibble groups arrive;
   - whether the synth echoes host DT1;
   - whether the `7F 00 7F 01` override works.
8. **Whether the synth sends "Notify Scene Load" / "Notify Tone Load"** when the panel changes Scene or Tone.
9. **Pacing:** the sustained rate of single DT1 edits without drops or audio glitches, and the real time of a full recall.
10. **Undocumented areas** (only if the owner wants them):
    - the drum kit;
    - the vocoder;
    - JD-800 at `02 30 …` and Vocal Designer at `02 40 …` (needs the expansions installed);
    - whether a JUNO-60 model exists on this unit.
11. **Signed-value encodings** that the MI leaves ambiguous (e.g. AMP MOD −100…100 in two nibbles).
12. **Whether System-area changes** (Master EQ/Comp, mic) persist across power cycles without a write.
13. **USB MIDI port names per OS and driver mode,** the device ID set on the synth, and the rear jack list.
14. **What the synth shows when DT1 data arrives:** the "EDITED" mark, and names on the display.
15. **Whether selecting another Scene on the panel discards edits without asking,** and what it sends.

---

## 11. Open questions for the owner

**Answered (2026-10-01; see `README.md`):**

- the whole Scene, its parts and saving to the computer come first;
- all engines, the analog models first;
- runs in mazika and potentially other DAWs;
- firmware 3.02, with 3.03 planned;
- MIDI on the main side, the web view separate.

**Still open:**

1. **Recall confirmation:** follow mazika and ask "Send · Cancel" on every recall (the default here), or recall without asking, since only the edit buffer changes?
2. **Engines the public SysEx does not cover:** drum kits, the vocoder and the expansions are saved by reference. Is that enough for now, or should measuring the undocumented areas on your synth be planned? Do you own JD-800 or Vocal Designer? Does your synth show JUNO-60 tones?
3. **mazika today:** until mazikad has native MIDI, the Jupiter core in mazika runs in mazika's browser core page, which is the owning core, never a client view. Is that acceptable for the first version, or should the editor wait for native MIDI?
4. **Snapshot files:** one JSON file per Scene with raw blocks plus readable values, shared by mazika and the plugin. Is that right, or do you also want Roland-compatible files later?
