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
