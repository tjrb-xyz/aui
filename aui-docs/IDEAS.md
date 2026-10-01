# aui — owner ideas

A running log of the owner's ideas for aui. Each entry has the date, the owner's words verbatim, then short notes. The notes are observations and open questions, not decisions.

---

## 2026-09-29 — Visual direction: clean, glass, one UI engine with skins

### Owner's words

> In terms of UI, I want it to be clean, minimalistic, and following the latest trends of design in terms transparent UIs, I want this to be able to be used reliable to build +100 integrations / extensions for audio players, daws, and audio drivers. I want it to be inspired by liquid glass. Take some inspirations from nuclear https://nuclearplayer.com - but don;t make it as animated. Take some inspirations from https://subvert.fm, obviously from ableton. However, I can provide you with additional elements. Here's 2 failing UI projects, I;ve started - I am saying failing because the UI of the player and of the DAW are very different. I do like the DAW UI quite a bit, but it needs vast improvements and polishing to look more  inspiring, yet minimal, and professional. It needs to allow for skins to mimic different elements, and focus on further flexiblity in terms of view, it should be a UI engine with tons of custom assets available

Attached screenshots (stored in `ideas/2026-09-29/`):

| File | What it shows |
| --- | --- |
| `jukebox-host.webp` | Jukebox host screen: now playing, up next, guest rotation, output, join QR |
| `jukebox-guest.webp` | Jukebox guest view (phone width): rotation position, speaker pairing, picks, playlists |
| `tapedeck-8track.webp` | Tapedeck 8-Track: light "cassette recorder" hardware panel inside the dark shell |
| `tapedeck-arrange.webp` | Tapedeck Arrange: track lanes, clips, R/M/S buttons, loop recorder side panel |
| `tapedeck-synth.webp` | Tapedeck Synth manager: knob groups, ADSR faders, keyboard |

### Notes

**Goals as stated**

- Clean, minimal, professional, yet inspiring.
- Translucent UI in the spirit of liquid glass.
- References: Nuclear (with much less animation), Subvert, Ableton.
- One visual language across players, DAWs and audio-driver tools. The two prototypes feel like different products, and that is the failure to fix.
- Built to carry 100+ integrations and extensions reliably.
- Skins that can mimic other gear and elements, plus flexible views.
- Framed as a UI *engine* with a large library of custom assets, not just a component kit.

**What the two prototypes already share.** This is a good base for one language:

- Dark neutral shell.
- Small uppercase, wide-tracked monospace section labels (NOW PLAYING, UP NEXT, TAPE COUNTER, LOOP RECORDER).
- Monospace figures for time and counters.
- Rounded cards with thin borders.
- Status dots before labels.

**Where they drift apart**

- Accent colour. Jukebox uses amber plus teal. Tapedeck is mostly greyscale, with red only for record.
- Surface. Jukebox uses flat cards with visible borders. Tapedeck uses softer pill shapes and lighter raised surfaces.
- Density. Jukebox is spacious and editorial, with big titles. Tapedeck is dense and tool-like.
- The same control drawn differently: the jukebox progress bar is segmented LED blocks, while tapedeck uses a playhead line and a slider.
- Knobs: the jukebox output knobs and the synth knobs are the same widget with different styling.

**The TAPEHAUS 424 panel is already a skin.** A light, hardware-styled panel sits inside the dark app shell. That is the "skins mimic different elements" idea working on a small scale. It suggests two layers:

- Themes: tokens such as colour, radius, type and glass level, applied app-wide as data.
- Skins: a different look for a component or a panel, such as a knob drawn as a Moog knob or a counter drawn as a mechanical tape counter. A skin should keep the component's behaviour, parameter model and accessibility unchanged.

**Tension to settle early: glass vs legibility.** Heavy translucency over busy content such as waveforms, clips or album art lowers contrast. mazika's §9.8 accessibility and legibility rules apply. One possible rule: glass for chrome and floating layers (toolbars, sidebars, popovers, transport, panels over art); solid or near-solid for data surfaces (lanes, meters, parameter values). Contrast is checked against the worst-case backdrop. The kit also needs a reduced-transparency mode.

**"Not as animated" as Nuclear.** Motion should be limited to state changes and meters. There should be no decorative or ambient motion, and `prefers-reduced-motion` should be honoured. This also fits the audio-app performance budget.

**"UI engine" for 100+ extensions.** This implies:

- A stable, versioned component and token contract that extensions build against.
- An asset registry (icons, knob and fader art, meter styles, device images) that extensions can add to.
- Possibly declarative layouts, so that an extension can describe a panel as data and the engine renders it.

This needs to be squared with the requirements drafted in `REQUIREMENTS.md` on the mazika session's branch once that branch is pushed.

**Open questions put to the owner**

1. Glass placement: chrome and overlays only, or everywhere?
2. Skins: does "mimic different elements" mean hardware look-alikes (like TAPEHAUS 424), bitmap and filmstrip art, whole-app looks, or all three?
3. Extensions: do third parties write React components against the kit, or declare UI as data (a manifest) that the engine renders?
4. Light mode: first-class alongside dark, or dark-first?

---

## 2026-09-29 — Answers: glass inside, gear replicas, SDK primitives, device views with sound

### Owner's words

> 1. If inner parts need glass then we should think of it.
> 2. Exactly, replicas of real gear such as the 808, JUPITER 8, NORD,ACCESS VIRUS, MPC, SP202, DJM, CDJs,  KORG, etc..., it should just be the inner part of the look. The outer should always look unbiased, yet it should be customizable. Say I want to put pink glass borders around the apps, I need to be possible.
> 3. Yes, we will write React on top of it. But also it will be used as an SDK, so it needs to have primitives.
> 4. Yes, either light mode, or toggle between 2 views - Say 2 different device views support this option - Say for DJ Mixer - Allen & Heath Xone 96 and DJM A9 support similar features in different types of view. However, they also introduce  a certain sound, so obviously with a skin, we should also be able to enable a certain sound emulation - That would be extension emulation code - Depending if the backend support's it for this option.

### Notes

**Two layers of look, named for clarity** (working names):

- **Frame:** the outer app chrome (shell, sidebars, toolbars, panel borders). It is always neutral and unbiased, but fully customisable through theme data. "Pink glass borders around the apps" is a frame theme setting: border tint, glass blur and opacity, radius.
- **Faceplate:** the inner device surface, where the gear replica lives (808, Jupiter-8, Nord, Virus, MPC, SP-202, DJM, CDJ, Korg...). A skin only ever touches the faceplate.

**Glass inside.** Glass is not banned from faceplates. It is opt-in per skin and held to the same legibility rules (§9.8) and reduced-transparency mode as the frame.

**SDK means two tiers:**

- **Primitives:** knob, fader, button, LED, meter, jog wheel, pad, screen or display, segment counter, keys, jack, panel, label and screw, plus glass and surface materials. These are what skin and extension authors compose.
- **Composed components:** mixer channel, transport, deck and so on, built from the primitives. React teams use either tier.

**Device model vs device view** (the DJ mixer example):

- A *device model* is behaviour and parameters: channels, EQ, filters, crossfader, sends.
- A *view* is a faceplate layout plus a skin bound to that model. "Xone 96 style" and "DJM-A9 style" are two views of one mixer model.
- A view declares which parameters it shows. Parameters it lacks stay reachable, for example through a generic panel, so switching views never loses state.
- Light and dark is the same idea one level up: a frame theme toggle.

**Sound follows skin (optionally):**

- A skin or view can reference a *sound character*: an emulation extension such as "Xone-style filter" or "DJM-style EQ curves".
- aui does no DSP. It asks the backend whether that emulation is available (a capability query to audio-engine or the host) and shows the toggle: enabled, disabled with a reason, or hidden.
- The look must work without the sound, and the sound without the look.

**Flag: brand names and trade dress.** Replicas named and styled as Roland, Akai, Pioneer DJ, Allen & Heath, Korg, Clavia, Access and so on can raise trademark and trade-dress issues once distributed. This is the same spirit as keeping Cutoff and Tylium names out of aui. Options:

- Unbranded "in the style of" skins with generic names.
- Keeping brand assets in private or user-supplied skin packs.
- Licensing.

Worth deciding before the asset library grows.

**Open questions put to the owner**

1. Replicas: branded (names, logos) or unbranded homages?
2. Skin authoring: code (React) only, or asset packs (SVG/bitmap + layout data) that designers can make without code?
3. View switch and sound switch: linked by default (pick DJM view → DJM sound), or always independent?
4. First proving case: the two-view DJ mixer?

**Cross-check with `REQUIREMENTS.md` rev 1** (branch `claude/happy-pasteur-7y8qeh`, commit `3731dee`):

- **Conflict: gear replicas vs AUI-020 (settled).** AUI-020 is settled from the owner's own earlier words in mazika's brief §1a. It rules out any third party's product name, logo, panel graphics or trade dress in aui's names, themes or components. The requirements also speak of "ten homage skins". Today's "replicas of real gear" goes further. The owner needs to choose one of these:
  - homages: evoke the character (layout, colour family, knob feel) with no names, logos or copied panels;
  - replicas: which reopens AUI-020.
- **Tension: glass vs the colour pipeline.**
  - The requirements fix colours as precomputed opaque `#RRGGBB`, with no runtime `color-mix()` (AUI-040, AUI-215, AUI-288).
  - Glass needs alpha colours and `backdrop-filter`.
  - `backdrop-filter` also costs GPU time in dense views and in older WKWebView hosts (AUI-073 already guards those hosts).
  - Likely fix: allow `#RRGGBBAA` tokens and a small set of glass material tokens (blur, tint, opacity, border). Include a solid fallback, used by the reduced-transparency mode, older hosts and the contrast check, which measures against the worst backdrop.
- **Tension: faceplate skins vs Q-11 (default: no bitmap film-strip controls in the first release).** Convincing gear homages usually want image assets. Q-11 may need revisiting, or skins stay SVG-plus-tokens at first.
- **Fits as written:**
  - skins and views by others (AUI-024, settled);
  - themes as data with light and dark (AUI-026);
  - one DOM per theme, so skins cannot change behaviour (AUI-029, AUI-182);
  - aui never touches audio (AUI-115), so sound emulation lives in the engine and aui only shows a capability toggle.
- **New scope not yet in the requirements:**
  - the frame vs faceplate split;
  - frame customisation such as tinted glass borders;
  - device model vs multiple views;
  - sound character linked to a skin;
  - public SDK primitives: primitives as declared-stable exports, extending AUI-122.

---

## 2026-09-29 — A skin's sound as a filter on the output

### Owner's words

> A skin might enable a custom sound as a "filter" on the output

### Notes

- This partly answers the earlier question 3 (does a view's sound follow its look?): a skin *may* bring a sound. The skin supplies it as a processing stage on the output, not as changes to the controls' behaviour.
- In engine terms this is a "character" insert on an output bus, hosted by audio-engine. aui still does no audio work (AUI-115). A skin's manifest can name the insert it wants. The engine reports whether it can run it, and aui shows:
  - on/off (bypass) for A/B listening;
  - greyed out with a reason when the backend lacks it;
  - nothing when the skin brings no sound.
- The look must never depend on the sound. Turning the filter off keeps the skin, and turning the skin off removes its filter.
- Possible UI needs:
  - a visible marker that the output is being coloured, so no one mistakes it for the dry signal;
  - the insert's latency and CPU cost, if the engine reports them.
- Open: which output does the filter sit on?
  - That device's own output only, such as the mixer view's master out.
  - Or the whole app's master.
  - And, whichever it is: is the filter on or off by default when the skin is chosen?

---

## 2026-09-29 — Filter placement, a global bus and a bus view

### Owner's words

> The filter, sits right after, where the device sits in the chain. Thus we need a global bus, as well as a global bus view, to show how the busses are all flowing

### Notes

- **Placement is settled by this answer.** A skin's sound filter is inserted immediately after its device, at that device's position in the signal chain. It does not go on the app master. Example: mixer skin → DJM-style filter right after the mixer, before whatever follows it.
- **The global bus is engine scope.** It is a single routing model for all devices, buses, inserts (including skin filters) and outputs, owned by audio-engine. aui renders it and proposes edits to it. It never processes audio (AUI-115).
- **The global bus view is new aui scope.** It is a view showing how all buses flow. It relates to REQUIREMENTS Q-17, which leaves a patchbay part open, with the default "the engine's UI builds it from aui's parts; aui adds a part when two consumers need it". mazika and audio-engine's UI would both want this view, which meets that bar.
  - Worth proposing the bus view as a first-class aui part, built from primitives: node, port, connection, insert slot, meter-on-wire.
- **A skin filter should be visible in the bus view** as its own node or insert right after the device, marked as a coloured (emulated) stage, with its bypass.
- **Open questions put to the owner**
  1. What should the bus view look like?
     - a node graph, where boxes and wires can go anywhere;
     - a left-to-right signal-flow diagram;
     - a mixer-style strip view with sends and returns;
     - or several of these as switchable views of the same model.
  2. Is it view-only, or can you reroute from it (drag wires, reorder inserts, bypass)?
  3. Should it show live signal: levels on wires, clip markers, latency per stage?

---

## 2026-09-29 — Neutral analog look; routing on back panels with patch cables

### Owner's words

> i want it to look as close as possible to something neutral and analog where possible. for routing we should use back consoles and wire patches between the different parts

### Notes

- **Overall direction: neutral and analog where possible.** Everyday physical studio gear with no brand character: plain panels, printed legends, real-looking jacks and knobs, and restrained colour.
- This sits alongside the earlier "liquid glass" direction. One reading: glass for the frame (app chrome, floating layers) and neutral analog for devices and routing. This needs the owner's confirmation.
- **Routing view = the back of the gear.**
  - Devices flip or turn to show a rear console with input and output jacks.
  - The user patches cables between them, like the back of a studio rack. The best-known software examples are rear-rack views with patch cables.
  - This view is the "global bus view" from the previous entry.
  - Global buses could appear as a neutral patchbay unit: rows of jacks, normalled pairs, and bus sends and returns.
  - Skin filters appear as their own small unit or insert right after their device (see the placement entry above).
- **New primitives this implies:**
  - jack/socket (input, output, stereo pair, with a type: audio, MIDI, CV/modulation, clock);
  - cable (curved sag, colour, highlight on hover);
  - rear panel (a grid of jacks with printed labels);
  - patchbay (rows of jacks with normalling).
- **No animation beyond the essentials.** This follows the earlier "not as animated" direction: cables are drawn as static curves with no wobble physics. At most the flip transition animates, and only if `prefers-reduced-motion` allows it.
- **Skins and back panels:**
  - either each skin also defines its rear panel;
  - or every device gets a generic neutral back generated from its declared inputs and outputs.
  - The second scales far better to 100+ extensions. It could be the default, with custom backs optional.
- **Accessibility (§9.8) and scale:**
  - Dragging cables needs a keyboard and screen-reader path: pick a source jack, then a destination, and every connection is readable as text ("Mixer out L → Tape in 1").
  - A list view of all connections doubles as that path.
  - Cable clutter needs controls: show all, show only the selected device's cables, or fade the others.
  - Many SVG cables need a performance check.
- **Open questions put to the owner**
  1. Glass and analog: glass for the frame, analog for devices and back panels?
  2. Back panels: generated from each device's inputs and outputs by default, with custom backs optional per skin?
  3. Global buses as a patchbay unit in the rack?
  4. Front and back: flip one device at a time, or turn the whole rack around?

---

## 2026-09-29 — Answers: glass vs analog (delegated), generated backs, console view, many ways to flip

### Owner's words

> 1. You Decide, what would be more unified?
> 2. Yes
> 3. Yes in a different "console-like" view under settings -  with ability to add shortcut in this session or all sessions.
> 4. Different ways

### Notes

**1. Glass and analog: proposed rule, delegated to this session; the owner can overturn it.** Glass is the enclosure; analog is the instrument.

- **The enclosure is glass.** This covers the app frame (sidebars, toolbars, transport bar), floating layers (popovers, menus, sheets) and the rack case around the devices. It is one frosted material with themeable tint and border. "Pink glass borders around the apps" lives here.
- **The instrument is analog.** This covers device faceplates, rear panels, the console view, jacks, cables and knobs: neutral, opaque, matte panels with printed legends. They are opaque because data and controls sit on them, which keeps legibility (§9.8) simple.
- **Glass inside a device appears only where real gear has glass or acrylic:** display windows, meter windows, cassette and tape windows, LED lenses. This is how glass enters the inner parts (the earlier answer 1: "If inner parts need glass then we should think of it") without breaking the analog look. It also matches the TAPEHAUS 424 prototype's dark window on a light panel.
- **Why this is the more unified choice:**
  - Every surface has exactly one material by role, so any extension or skin knows which one to use.
  - Glass is never under dense data.
  - The reduced-transparency mode only has to replace one material, with a solid fallback.
- **A theme exposes two material families:** `glass.*` (tint, blur, opacity, border, highlight) and `panel.*` (surface, legend ink, screw/edge detail, wear level: none by default).

**2. Back panels: settled.** Every device gets a generated neutral rear panel from the inputs and outputs it declares. A skin may supply a custom back. The generated back is the default for all 100+ extensions.

**3. Global buses: settled, with more detail.**

- The global buses and patchbay live in a separate console-like view under Settings, not in the rack itself.
- Users can add a shortcut to that view in two scopes:
  - **this session only:** stored with the project or session;
  - **all sessions:** stored in user preferences.
- aui's part in this is generic: a "pin shortcut" affordance with a scope choice (session / all sessions). Where the shortcut is stored is the app's concern (mazika, the engine UI).
- This pin-with-scope pattern is probably reusable for any view, not only the console.

**4. Flipping: several ways, all over one routing model.**

- Flip a single device to its back.
- Turn the whole rack around.
- Open the console view (global buses and patchbay).
- The connections list (the accessible text path).

These are views of one connection graph in the engine, so a patch made in any of them appears in all of them. The flip transition is the only motion, and it respects `prefers-reduced-motion` (an instant swap).

**Open questions put to the owner**

1. Does "session" mean a DAW project/session (mazika's session), or an app launch?
2. Does the console view show only global buses and the patchbay, or also every device's back in a compact list (a full routing overview)?
3. Still open from earlier:
   - replicas vs unbranded homages (AUI-020);
   - skins in code only, or also as designer asset packs;
   - the two-view DJ mixer as the first test case.

---

## 2026-09-29 — Device list with compact views; tablet split layout

### Owner's words

> it should also have a list of devices which can show each device as a compact view to fit on the screen. if we in the tablet it would show the previous related info next to it

### Notes

- **Reading (to confirm):** the console view also holds a list of all devices. Each device can show as a compact view sized to fit the screen. On a tablet, that list sits side by side with the routing information from the previous entries (global buses, patchbay, connections), instead of on a separate screen.
- **A third face per device.** Each device then has three representations over the same model:
  1. front (full faceplate, skinnable);
  2. back (generated rear panel, custom optional);
  3. compact (a small tile: name, a few key controls, I/O status, level meter, bypass or power).
- **Like the back, the compact face should be generated by default.** It is built from what the device declares: a short list of "key parameters" or macros, plus its I/O. A skin may still restyle it. With generated compact faces, 100+ extensions get a usable compact face for free.
- **Adaptive layout by available space, not by device type.** This uses CSS container queries, no JavaScript sizing.
  - **Narrow (phone):** device list → tap → that device's routing or detail.
  - **Medium (tablet):** split view. The device list is on one side, and the related routing information (buses, patchbay, the selected device's connections) is on the other.
  - **Wide (desktop):** possibly list + routing + the selected device's front or back together.
- **aui primitives implied:**
  - a split/master-detail layout that collapses by container width;
  - a compact device tile;
  - selection linking, so selecting a device in the list highlights its jacks and cables in the routing pane.
- **Open questions put to the owner**
  1. Is the reading right: on a tablet, device list and bus/patchbay side by side?
  2. What should a compact device show: a few chosen key controls, only meters and status, or should the device author pick?
  3. Can you edit from the compact view (turn a key knob, bypass), or is it view-only?
  4. On desktop, all panes at once (list + routing + device)?

---

## 2026-09-29 — Replica homages live in external, editable skin packs

### Owner's words

> replica homages are not in the binary but an external editable pack of skins

### Notes

- **This answers the replicas vs homages question with a split:**
  - aui itself (the package, and any app binary built on it) ships only neutral, unbranded skins. That keeps AUI-020 true for everything aui and its apps distribute.
  - Replica homages of real gear live outside, in external skin packs that users can edit. This matches the "user-supplied skin packs" option noted earlier.
- **Consequence: the skin pack is a public format.** aui must define it as data, and any pack aui loads must also follow the kit's other rules:
  - **Manifest:** name, version, author, licence, target device model(s), which faces it skins (front / back / compact), and an optional sound filter reference.
  - **Assets:** SVG and bitmaps, plus token overrides (`panel.*`, `glass.*` for windows, geometry).
  - **Validation:** the schema refuses unknown keys and says why (AUI-116, AUI-210). The contrast check runs on the pack's tokens before it is applied (AUI-214).
  - **Hard rule:** a skin never changes behaviour, ARIA or state words (AUI-029). A pack can only restyle the one DOM.
- **Loading external packs under COOP/COEP (AUI-162, AUI-163).** Remote images are blocked unless same-origin. So packs are imported (a file or folder picked by the user) into app storage and served locally, for example from IndexedDB through `blob:` URLs. They are never hot-linked from the web.
- **"Editable" implies:** packs are plain files a designer can open and change. A later in-app skin editor could live on top of the same format.
- **Sound filters stay separate.** A pack can *reference* an emulation filter. The filter itself is engine extension code, installed and trusted separately, never executed from a skin pack.
- **Remaining legal note (flag only).** Whoever makes and shares a branded pack carries its trademark risk. If the owner later runs an official pack gallery or marketplace, hosting branded packs there would bring that risk back.
- **Open questions put to the owner**
  1. Are packs data only (manifest + SVG/images + tokens), or may they include code (custom React faces)? Data-only is safer and simpler to validate; code allows more.
  2. How do users get packs: file import only, or an in-app pack browser?
  3. Editing: by hand in files for now, with an in-app skin editor later?

---

## 2026-09-29 — Wrapper VST beside the pack, first-party pack, text-editable, editor API?

### Owner's words

> it has a wrapper vst / next to it with the instrument modulation details. we as the developers are making the first pack. should be editable with text editor. should we have an editor api as part of aui

### Notes

- **Sound side:** a pack comes with, or sits next to, a wrapper VST (plugin) that carries the instrument's modulation and emulation details. This fits the split already noted:
  - the skin pack is look only;
  - the plugin is sound;
  - the pack references the plugin by id.
  
  The engine hosts the plugin at the device's position in the chain. aui shows its state (available, bypassed, missing).
- **First pack is first-party.** The team makes the first pack. That makes the team the pack author, so the earlier trademark note applies to this pack directly, even though it ships outside the binary. Naming it and describing its gear "in the style of" keeps the risk down.
- **Text-editable format (proposal):**
  - The manifest and tokens are JSON with a published JSON Schema (`"$schema": ...`), so any text editor with JSON support gets autocompletion, hover docs and error squiggles for free.
  - Artwork is SVG where possible, which is text too. Bitmaps are allowed but are the only non-text part.
  - Stable part ids name every skinnable piece (for example `knob.cap`, `knob.arc`, `panel.legend`), so a pack author can target parts by name.
- **Editor API: recommendation is yes, but headless.**
  - aui ships the pack tooling as a framework-free API, not a full editor app:
    - `schema` (the JSON Schema);
    - `validate(pack)` (plain-sentence errors, AUI-210);
    - `load(pack)` / `apply(pack)` / `unapply()`;
    - `diff(a, b)`;
    - contrast check on the pack's tokens;
    - a dev "live reload" that re-applies a pack when its files change.
  - React adds an **inspect mode**: hover or click a part in the running UI to see its part id and the tokens that style it, which tells a text-editor author what to change.
  - A visual skin editor can be built later on this same API: an app, or a mode in the playground.
  - **Why one API:** the same validator then runs in CI, on import in the app, in the text editor (via the schema) and in any future visual editor. That gives one source of truth, and 100+ extensions cannot drift from it.
- **Open questions put to the owner**
  1. Does the pack *contain* the wrapper VST, or sit next to it and reference it by id? Referencing keeps packs text-only and lets one plugin serve many skins.
  2. Which device is the first pack for? The two-view DJ mixer, or a synth like the Jupiter-style one in the tapedeck prototype?
  3. Plugin format: VST3 only, or also AU/CLAP? This is mostly an audio-engine question, noted here for completeness.

---

## 2026-09-30 — Pack beside the VST; first device delegated; cross-DAW format

### Owner's words

> yeah the pack sit next to the VST meaning they can be updated separately.
> you decide which device goes first
> 3. the format should to be used on other computers with other daws as well

### Notes

**Settled: the pack sits next to the plugin, not inside it.** A pack references its plugin by id (and a version range). The two are versioned and updated separately. A pack with its plugin missing still loads, and the sound toggle shows "not installed". A plugin with no pack still runs with a generated neutral face.

**First device: the tape machine.** The owner delegated this; they can overturn it. The pack would be a multitrack tape recorder with a tape-character plugin. Why it goes first:

1. **It is the heart of the first consumer.** mazika's main view is the 8-track recorder (brief §1: "The main view should be an **8 track recorder**"). The tape view is already built (`src/views/tape`), with in-house parts such as `CassetteWindow`, `Reel` and `ReelGauge`. The owner's tapedeck prototype already has a tape panel (the TAPEHAUS 424 screenshot).
2. **It is not blocked.** The DJ view, and so the DJ mixer, waits on mazika's native core: mazika@36bde22:docs/HANDOFF.md §5 "Nothing is built against a simulated core first: the owner answered \"wait for native\"."
3. **Its sound is exactly the "filter after the device" the owner described.** A tape-character effect (saturation, wow and flutter, hiss, speed-dependent tone) runs right after the recorder in the chain.
   - An effect is also the easiest plugin to make work in other DAWs on other computers: audio in, audio out, no MIDI.
   - Hosts differ in their MIDI-effect support, as mazika's own brief notes for its recorder plugin.
4. **It exercises almost every primitive and idea from this log:**
   - Front controls: transport buttons, a segment counter, VU meters behind glass meter windows, and reels behind a cassette window. This is the "glass only where real gear has glass" rule.
   - Settings: knobs (drive, bias, flutter), switches (tape speed, tape type) and 8 channel strips with faders and record-arm buttons.
   - The generated back panel: 8 inputs, 8 outputs, sends.
   - The compact view: VU pair, counter, transport.
5. **Two views of one model come naturally.** A "cassette multitrack" view and a "reel-to-reel studio deck" view share one tape model. This proves the device-model-vs-views idea without waiting for the DJ mixer.
6. **Low trade-dress risk.** Reels and VU meters are generic to the whole category, so a homage without logos is easy.
   - The prototype's "424" matches a well-known cassette multitrack's model number, which the homage rule rules out. The first pack needs its own name.

**Suggested order after it:**

- **The DJ mixer**, once mazika's native core lands. It exercises two views, many buses and sends.
- **A synth**, which fits mazika's planned M1.2 Jupiter-Xm synth manager.
  - That view edits real hardware over MIDI and SysEx, so its pack is look-only and needs no emulation plugin. It is a good test of a pack without sound.
  - Naming the connected product to refer to it is allowed (mazika brief §1a).

**Alignment note, mazika DJ design §4.6.** mazika@36bde22:docs/research/dj-view-design.md §4.6 says "a skin may style a mode's controls, and may never pick a mode, add a control or hide one". The owner's "two device views" (Xone-style vs DJM-style) fit this as follows:

- the *view/layout* is a user-chosen mode or setting, named for what it puts first, never after a product;
- the *skin* only styles it.

So a pack can *suggest* a view, but the user picks it. aui's pack format should keep "view" and "skin" as separate fields for this reason.

**Cross-DAW: "the format should to be used on other computers with other daws as well."** This applies to both halves: the plugin and the skin pack. These are research notes, checked 2026-09-30. Where a claim rests on secondary sources, that is marked. The plugin side is audio-engine's call.

*Plugin formats (the sound half):*

- **Proposal:** build one CLAP plugin, and use `clap-wrapper` (MIT) to produce VST3 and AUv2 from it, plus a standalone app. Covering the major formats this way means:
  - **VST3** covers Live, Cubase, FL Studio, Studio One, REAPER and Bitwig. The VST3 SDK is MIT-licensed from 3.8.0 (2025-10-20), per KVR's news report (kvraudio.com, "Steinberg moves VST 3 SDK to MIT").
  - **AU** is required for Logic Pro, GarageBand and MainStage, which do not load VST3.
  - **CLAP** is native in Bitwig, REAPER, FL Studio 2024+ and partly Studio One 7. It is not supported in Cubase, Live, Logic or Pro Tools (secondary sources).
  - **Linux:** LV2 (Ardour, Carla, REAPER Linux) can come later, if wanted.
- **Pro Tools (AAX) is left out at first.** It needs Avid's developer programme, an NDA with PACE and iLok or cloud signing, which is the highest barrier of any format. `clap-wrapper` can produce AAX later if someone takes on the signing.
- **The plugin's UI is aui in a web view:** WKWebView (macOS), WebView2 (Windows), WebKitGTK (Linux).
  - The requirements already guard for older WKWebView hosts (AUI-073). Known web-view pitfalls for aui and the engine to design for:
    - **WebView2 data folders:** each plugin instance needs its own, or later instances render blank.
    - **Keyboard focus:** the host and the web view compete for keys, and keyboard shortcuts can fire twice.
    - **Blank editors on reopen in some hosts.**
    - **WebView2 runtime:** missing on some Windows 10 machines, so ship its installer.
    - **WebKitGTK:** must be present on Linux.
  - Add an aui acceptance test: the kit runs inside each host web view, with keyboard focus behaving correctly.
- **Framework licences affect the licence choice (Q-01):**
  - JUCE 8 is AGPLv3 or commercial;
  - iPlug2 is zlib-like, DPF is ISC, choc (a single-header web view) is ISC;
  - nih-plug's VST3 export uses GPLv3 bindings (unconfirmed whether that has changed).
  
  A plugin that bundles aui inherits aui's licence terms, so the licence decision (REQUIREMENTS Q-01) should consider plugins sold or shared for other DAWs.

*Skin packs (the look half):*

- **A pack must be portable:** one self-contained folder (or a zip of it) with only relative paths, text files plus images, identical on macOS, Windows and Linux.
- **Packs are found by id, not path.** A plugin's saved state (the host's project) stores the pack's id, version and content hash, never an absolute path. This lets a project opened on another computer find the same pack, or show "pack missing" and fall back to the neutral face.
- **Per-user pack folders:**
  - macOS: `~/Library/Application Support/<vendor>/packs`
  - Windows: `%APPDATA%\<vendor>\packs`
  - Linux: `$XDG_DATA_HOME/<vendor>/packs`
  
  The mazika app and the plugins on the same computer can share this folder.
- **Presets** travel through each format's own mechanism (VST3 `.vstpreset`, AU `.aupreset`, CLAP state). The pack only references them.

**Open questions put to the owner**

1. Is the tape machine as the first pack OK?
2. Pro Tools: needed early, or fine to add later?
3. Linux: needed for the plugins, or macOS and Windows first?
4. Name for the first pack. It needs its own name: not TAPEHAUS 424, and no model numbers.

---

## 2026-09-30 — The VSTs are instruments

### Owner's words

> VSTs are more like drum machines and synths

### Notes

- **Correction to the first-device choice above.** The plugin beside a pack is an *instrument* (drum machine, synth), not an effect such as tape character. The tape machine stays as mazika's built-in main view (it can still take a skin) but is not the first *pack*.
- **Revised first device: the drum machine**, an instrument plugin plus its pack. The owner delegated this; they can overturn it.
  - **It is already in mazika's plan:** mazika@36bde22:docs/brief.md "the owner kept the device views and multiplayer, the drum machine, the ten homage skins and the Strudel extension in the plan."
  - **It exercises the most of aui:**
    - pads;
    - a step sequencer, which puts the parameter-button painting gesture from the requirements (AUI-147, AUI-204) to real use;
    - per-voice knobs (tune, decay, tone, level);
    - an accent and swing section;
    - a pattern display, which is a glass window on the instrument.
  - **Its back panel is the best routing test:** a stereo out plus individual outputs per voice, MIDI in and sync in. Each voice can be patched to its own bus in the console view.
  - **The compact view is obvious:** pattern number, play state, an output meter and a few macro knobs.
  - **Drum voices are simpler to synthesise than a full polysynth**, which keeps the first plugin small.
  - **Low trade-dress risk:** a generic step-sequencer drum machine is a whole category of instruments. It needs its own name, no model numbers and no copied panel colours.
  - **A second view of the same model:** "pads" (an MPC-style grid) vs "steps" (a row of 16 step keys).
- **Next: a synth.** It fits the owner's "instrument modulation details" (envelopes, LFOs, a mod matrix, as in the owner's synth-manager prototype) and mazika's M1.2 Jupiter-Xm editor.
- **Instruments in other DAWs.** The earlier MIDI caveat was about MIDI *effects*. Instrument plugins with MIDI in are supported everywhere, so the CLAP → VST3/AU plan stands.
- **Open question: how do instrument plugins and the skin sound filter fit together?** There seem to be two kinds of sound a pack can bring:
  1. **The instrument itself** (the plugin *is* the drum machine or synth);
  2. **A character filter** after a device (the earlier "custom sound as a filter on the output").
  
  Are both meant, or only the instrument?

---

## 2026-10-01 — The synth goes first

### Owner's words

> I think a sync is a better example here

(interrupted by the owner, then:)

> i think that synth is a better choice for us for now

### Notes

- **Settled by the owner: the first pack is a synth** (instrument plugin + its pack). This replaces the drum machine choice above; the drum machine moves to second.
  - "sync" in the first message reads as a typo for "synth", which the second message confirms.
- **Why it fits:**
  - **It matches the owner's "instrument modulation details"** (envelopes, LFOs, mod routing).
  - **It matches the owner's synth-manager prototype** (`ideas/2026-09-29/tapedeck-synth.webp`): oscillator, filter, amp envelope and effects sections with knobs, ADSR faders and keys.
  - **It matches mazika's M1.2,** the Jupiter-Xm synth manager.
- **What the synth pack exercises:**
  - **The front panel:**
    - knobs, faders, switches and segmented selectors (wave shape, octave);
    - keys;
    - pitch and mod wheels;
    - a glass display window (patch name, value readout);
    - LEDs.
  - **Modulation needs new primitives:**
    - an envelope display (an ADSR curve drawn from the parameter values);
    - an LFO rate and shape indicator;
    - a mod matrix (source → destination → amount).
    - Possibly also modulation rings on knobs showing the range a modulator moves them through.
  - **The back panel:** stereo out, MIDI in, possibly a CV/gate or external audio in for filter processing, and sync.
  - **The compact view:** patch name, an output meter and 4 macro knobs.
  - **Two views of one model:** a "full panel" view of every section, and a "performance" view with macros, keys and wheels. The second suits phones and tablets.
- **One look, two kinds of device.** The same synth skin could face both:
  - the *software* synth (the instrument plugin);
  - a *hardware* synth editor (mazika M1.2, editing a real synth over MIDI and SysEx, with no plugin needed).
  
  That is a cheap way to show a pack is look only.
- **Open questions put to the owner**
  1. What kind of synth first: a polyphonic analog-style synth (pads, brass), a mono bass synth, or one plugin with both modes? The prototype's parts suggest all three.
  2. Should the same pack also skin the hardware synth editor (mazika M1.2)?
  3. Still open: a character filter after a device as well as instruments, or instruments only?

---

## 2026-10-01 — Model the first synth on the Jupiter-Xm

### Owner's words

> Yes, I want you to model one for the Juliter XM. As It’s current editor and tooling is very annoying.

### Notes

- **This answers two open questions:**
  - The first synth pack is modelled on a real synth, the owner's Roland Jupiter-Xm.
  - It covers the hardware editor ("Yes": the same pack skins the editor that drives the real synth).
  - "Juliter XM" reads as a typo for Jupiter-Xm.
- **Motivation, in the owner's words:** the synth's "current editor and tooling is very annoying". The model should aim at a better editor than the vendor's.
- **Fits mazika's plan:**
  - M1.2 ("the Jupiter-Xm synth manager. Edit buffer only; **never write the synth's memory without the "Save to synth…" confirmation**", mazika@36bde22:docs/HANDOFF.md §4);
  - the synth-manager prototype screenshot;
  - mazika's ux-spec §6.4 (sound-design screen) and §6.6 (synth manager).
- **Naming.** Naming the connected product to refer to it is allowed (mazika brief §1a). Roland's logos and panel graphics stay out of aui and mazika. A Jupiter-style look lives only in the external pack.
- **Deliverable:** a device model, drafted in `aui-docs/devices/jupiter-xm.md`, covering:
  - the parameter tree;
  - parts and engines;
  - the views (front, performance, compact, back);
  - how edits reach the synth;
  - what the pack styles.

---

## 2026-10-01 — Jupiter-Xm editor: priorities, engines, where it runs

### Owner's words

> 1. It’s mainly about seeing the whole scene, editing its parts, and saving it, not necessarily on synth memory but computer memory and being able to reload the save with all the parameters and parts we’ve edited.
> 2. All engines are used but the analog models are the primary models to support here yet a complete support is desired
> 3. would love if this runs as part of mazika and potentially other daws as well
> 4. My jupiter is running the latest firmware

### Notes

- **Priority, settled:**
  1. see the whole Scene;
  2. edit its parts;
  3. save to the *computer* and reload with every parameter of every part.

  Writing to the synth's own memory is secondary.
  - The core feature is a **Scene snapshot file**: a complete capture of the temporary Scene and all five parts' tones, effects and arpeggio/step data.
  - Reloading sends it back into the synth's edit buffer. That does not touch the synth's memory, so it needs no "Save to synth…" confirmation.
- **Engines, settled:** the analog models come first (JP-8, JX-8P, JUNO-106, SH-101, JUNO-60, JUPITER-X model), and complete support for every engine is the goal. The snapshot must capture every engine from day one, even before each engine has its own editing panel, so nothing is lost on reload.
- **Where it runs, settled:** inside mazika, and potentially as a plugin in other DAWs.
  - In a DAW, the project can hold the Scene snapshot in the plugin's saved state. Reopening the project then restores the synth ("total recall"), which serves the save/reload goal directly.
  - Feasibility questions: how a plugin reaches the synth's SysEx (through the host, or by opening the MIDI port itself) and port sharing with the DAW. Researched in the device model's next revision.
- **Firmware: "the latest".** The research so far found nothing after 3.x. The exact latest version and its changes need checking; the version is also recorded in every snapshot file.
- **Next:** device model draft 2 in `aui-docs/devices/jupiter-xm.md`, reworked around the snapshot file, capture and recall, and the plugin mode.

---

## 2026-10-01 — Jupiter-Xm gets its own PR; MIDI off the web view; roadmap

### Owner's words

> this should be its own PR. call it “jupiter-xm” and place under an example. The web view should be separate from the main thread which should handle midi. the sounds now come off the jupiter xm, and later on once dsper is wired and works properly we can build a synth controller device which will contain jupiter xm as model. let’s keep all of these notes in the jupiter-xm PR for now. Later on we can create a synth-modeller repo once we have the necessary UI elements here.

### Notes

- **Where the notes live:** all Jupiter-Xm notes move to their own branch and PR, `jupiter-xm`, under `examples/jupiter-xm/`:
  - `README.md`: purpose, the owner's decisions verbatim, the process model, the roadmap;
  - `DEVICE-MODEL.md`: the device model, moved from `aui-docs/devices/jupiter-xm.md`, which is removed from this branch.
  
  This log keeps only the owner's words and a pointer.
- **Process model, settled:**
  - The main (native) side owns MIDI: port, SysEx, pacing, device state, snapshot files.
  - The web view only renders aui and exchanges messages with the main side.
  - In a plugin, the main side is a non-audio thread.
- **Sound, settled:** for now the sound comes off the Jupiter-Xm itself; the editor makes none.
- **Roadmap, settled:**
  1. now: a hardware editor;
  2. later: a synth controller device, with the Jupiter-Xm as one model, once dsper is wired and works;
  3. later: a `synth-modeller` repository, once aui has the UI elements this needs.

---

## 2026-10-01 — The Jupiter-Xm controller moves to tjrb-xyz/synthctrlr

### Owner's words

> I’ve outsourced the jupiter xm controller into a new repo, which has to have a new session and use “aui”. The new name is “tjrb-xyz/synthctrlr”. Can you open a new session as well as pose the requirements you have to “aui” in the documents you’ve been writing thus far.

### Notes

- **New consumer of aui: synthctrlr** (tjrb-xyz/synthctrlr), with its own session. It replaces the "synth-modeller" repository named in the jupiter-xm roadmap. Its first model is the Jupiter-Xm editor designed in PR #1 (`examples/jupiter-xm/`, branch `jupiter-xm`).
- **What synthctrlr needs from aui** is written down in `examples/jupiter-xm/AUI-REQUIREMENTS.md` (branch `jupiter-xm`). In short:
  - **Kit contracts:**
    - presentation only (AUI-115);
    - declared-stable exports (AUI-122);
    - runs in plugin web views (AUI-073, AUI-040, AUI-179);
    - loads nothing from elsewhere (AUI-162, AUI-163);
    - a parameter model a device descriptor maps into (AUI-265);
    - control states with app-supplied words;
    - fast external updates (AUI-135, AUI-286).
  - **New parts:**
    - Scene overview: part strip, key-range bar;
    - part editor: section panel layout, multi-stage envelope, LFO indicator, display window;
    - save and reload: progress-and-verify panel, librarian list with diff;
    - later: step-pattern row, signal-flow diagram, pad grid, badges, compact tile and back panel.
  - **Theming and packs:** the public skin-pack format, material tokens (`glass.*`, `panel.*`) and the headless pack API, all logged above (2026-09-29, 2026-09-30).
  - **Gates:** aui's licence (Q-01), since synthctrlr's plugins bundle aui; and aui's published name (Q-02).
- **The ideas above that synthctrlr inherits:**
  - glass enclosure vs analog instrument;
  - homage packs outside the app, the first one an unbranded homage;
  - packs beside plugins;
  - the MIDI main side separate from the web view.
- **Settled (owner: "you should close it once the notes are brought over"):** synthctrlr imported the notes (branch `docs-from-aui`, commit `504486d`), and PR #1 is closed. The `jupiter-xm` branch stays as the record.
