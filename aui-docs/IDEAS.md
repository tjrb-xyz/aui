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
