# Example: jupiter-xm

An editor for the owner's Roland JUPITER-Xm. It drives the real synth over USB MIDI and SysEx, and its UI is built with aui.

**Status:** design notes only. Nothing here is built. This folder holds every note about the Jupiter-Xm editor until a separate `synth-modeller` repository exists (see the roadmap below).

| File | What it holds |
|---|---|
| `README.md` | Purpose, the owner's decisions verbatim, the process model, the roadmap, open questions |
| `DEVICE-MODEL.md` | The device model: Scene structure, engines, parameters, SysEx, Scene snapshot files, views, the pack |

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
  - **mazika:** the core (later the native daemon, mazikad). mazika already says "**A client never calls `requestMIDIAccess`.**" (mazika@36bde22:docs/ux-spec.md §6.6.2).
  - **A DAW plugin:** the plugin's own non-audio thread. It is never the audio thread, and never the editor's web view.
- **The web view is a client.**
  - It holds a replica of the device state and proposes edits as messages (`synth.set {param, value}`).
  - It receives state updates (`synth.state`).
  - It can close, reload or crash without losing the synth's state.
- **Why:**
  - **Total recall works with the editor window closed:** a DAW project reload restores the synth from the main side.
  - **No dependence on Web MIDI support inside a host's web view.** Whether WKWebView, WebView2 and WebKitGTK support Web MIDI is being checked.
  - **Slow SysEx dumps never stall the UI.**

## Roadmap

1. **Now: a hardware editor.**
   - The Jupiter-Xm makes the sound. Its audio reaches mazika as a source through USB audio or an interface input.
   - aui supplies the views. The device model and the snapshot format live in this example.
2. **Later: a synth controller device.** Once dsper (tjrb-xyz/dsper) is wired and works properly, build a generic synth controller device that holds *models*; the Jupiter-Xm becomes its first model.
3. **Later: a `synth-modeller` repository.** It is created once aui has the UI elements this example needs (DEVICE-MODEL §7). The notes here move there then.

## Open questions

1. **Firmware:** which exact version is "the latest" on the owner's synth? The research found nothing after 3.x; a check is under way. The editor will read the version from the synth and record it in every snapshot.
2. **Expansions:** does the owner have the JD-800 or Vocal Designer expansions?
3. **Plugin mode:** how a plugin reaches the synth's SysEx (through the host, or by opening the port itself), and sharing the MIDI port with the DAW on Windows. A check is under way.
