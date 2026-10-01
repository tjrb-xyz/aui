# Example: jupiter-xm

An editor for the owner's Roland JUPITER-Xm. It drives the real synth over USB MIDI and SysEx, and its UI is built with aui.

**Status:** design notes only. Nothing here is built. This folder holds every note about the Jupiter-Xm editor until a separate `synth-modeller` repository exists (see the roadmap below).

| File | What it holds |
|---|---|
| `README.md` | Purpose, the owner's decisions verbatim, the process model, the roadmap, open questions |
| `DEVICE-MODEL.md` | The device model (draft 2): the synth as seen over SysEx, the address map, Scene snapshots, plugin mode, views, the pack, what mazika must change, checks on the synth |

---

## Purpose

- **Fix what makes the current tools annoying.** The owner's priorities, in order:
  1. see the whole Scene;
  2. edit its parts;
  3. save to the computer and reload it with every parameter of every part.
- **Support the analog models first** (JP-8, JX-8P, JUNO-106, SH-101, JUNO-60, the JUPITER-X model), with complete support of every engine as the goal.
- **Run inside mazika, and potentially as a plugin in other DAWs.**
- **The sound comes off the Jupiter-Xm itself.** This example makes no sound of its own.
- **Prove aui's parts** (knobs, faders, envelope curves, part strips, the glass display window, the librarian) on a real device.

## The owner's decisions (verbatim)

**2026-10-01, modelling request:**

> Yes, I want you to model one for the Juliter XM. As It’s current editor and tooling is very annoying.

**2026-10-01, priorities, engines and hosts:**

> 1. It’s mainly about seeing the whole scene, editing its parts, and saving it, not necessarily on synth memory but computer memory and being able to reload the save with all the parameters and parts we’ve edited.
> 2. All engines are used but the analog models are the primary models to support here yet a complete support is desired
> 3. would love if this runs as part of mazika and potentially other daws as well
> 4. My jupiter is running the latest firmware

**2026-10-01, this PR, threads, the roadmap:**

> this should be its own PR. call it “jupiter-xm” and place under an example. The web view should be separate from the main thread which should handle midi. the sounds now come off the jupiter xm, and later on once dsper is wired and works properly we can build a synth controller device which will contain jupiter xm as model. let’s keep all of these notes in the jupiter-xm PR for now. Later on we can create a synth-modeller repo once we have the necessary UI elements here.

**2026-10-01, firmware:**

> I have version 3.02 but planning to upgrade to 3.03

The earlier ideas behind this example are logged with dates in `aui-docs/IDEAS.md` on the `owner-ideas` branch:
- the synth as the first pack;
- the pack sitting beside its plugin;
- packs editable as text;
- homage rather than replica.

## Process model

The owner's rule: **the web view is separate from the main thread, and the main thread handles MIDI.**

```
 ┌────────────── main side (native) ──────────────┐        ┌──────── web view ────────┐
 │ MIDI port (USB / Bluetooth)                     │        │ aui views                │
 │ SysEx framing, checksum, packet pacing          │ ─────▶ │ replica of device state  │
 │ echo guard, Edit Tx parsing                     │ state  │ proposes edits           │
 │ device state: the current Scene                 │ ◀───── │ never touches MIDI       │
 │ snapshot capture and recall, files on disk      │ edits  │                          │
 └─────────────────────────────────────────────────┘        └──────────────────────────┘
```

- **The main side owns everything that talks to the synth or the disk:**
  - the MIDI port;
  - SysEx encoding and pacing;
  - the device state;
  - Scene snapshot files.
  
  Which process this is depends on the host:
  - **mazika:** the core that owns the synth.
    - Today that is mazika's browser core page, using Web MIDI.
    - Later it is the native daemon, mazikad.
    - mazika already says "**A client never calls `requestMIDIAccess`.**" (mazika@36bde22:docs/ux-spec.md §6.6.2).
  - **A DAW plugin:** the plugin's own non-audio thread, which opens the synth's MIDI port itself rather than going through the DAW (DEVICE-MODEL §5). It is never the audio thread, and never the editor's web view.
- **The web view is a client.**
  - It holds a replica of the device state and proposes edits as messages (`synth.set {param, value}`).
  - It receives state updates (`synth.state`).
  - It can close, reload or crash without losing the synth's state.
- **Why:**
  - **Total recall works with the editor window closed:** a DAW project reload restores the synth from the main side.
  - **No dependence on Web MIDI inside a host's web view.** WebKit has none, so WKWebView (macOS, iOS) and WebKitGTK (Linux) cannot reach MIDI. WebView2 (Windows) needs a SysEx permission (DEVICE-MODEL §3.1).
  - **Slow SysEx dumps never stall the UI.**

## Roadmap

1. **Now: a hardware editor.**
   - The Jupiter-Xm makes the sound. Its audio reaches mazika as a source through USB audio or an interface input.
   - aui supplies the views. The device model and the snapshot format live in this example.
2. **Later: a synth controller device.** Once dsper (tjrb-xyz/dsper) is wired and works properly, build a generic synth controller device that holds *models*; the Jupiter-Xm becomes its first model.
3. **Later: a `synth-modeller` repository.** It is created once aui has the UI elements this example needs (DEVICE-MODEL §8). The notes here move there then.

## Open questions

The full list, with defaults, is in DEVICE-MODEL §11.

1. **Firmware: answered.** The owner's synth runs 3.02, with an upgrade to 3.03 planned.
   - Roland's newest MIDI Implementation (v1.06) dates from 3.00. The research found no parameter or SysEx changes in 3.01–3.03 (DEVICE-MODEL §2.6).
   - The editor records the identity reply and the confirmed version in every snapshot.
   - **Before upgrading,** take Roland's own backup as well (and a snapshot backup once the editor exists).
2. **Recall confirmation:** ask "Send · Cancel" on every recall, as mazika does (the default), or recall without asking?
3. **Engines the public SysEx does not cover:** drum kits, the vocoder and the JD-800 / Vocal Designer expansions are saved by reference for now.
   - Does the owner have either expansion?
   - Does the synth show JUNO-60 tones? Roland's document does not list a JUNO-60 model for this synth.
4. **mazika before native MIDI:** is it acceptable for the editor's main side to run in mazika's browser core page until mazikad has native MIDI?
