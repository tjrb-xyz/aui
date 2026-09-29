# aui: requirements

**Status:** rev 2, 2026-09-29, design only. Rev 2 answers the review of rev 1 ("Review of rev 1", at the end). Nothing here is built. This file says what aui is, what it must do, and how we will know it does it. aui's own Claude session builds the kit. Start with `aui-docs/HANDOFF.md`.

**Pins.** mazika at `738d4bf` (the owner's words, the spec, the UI foundation design, and `src/ui` and `src/themes`), and at `e8124d3` for the brief's plugin block only. aui at `640c3d7` (the fork as it arrived). bana at `04a5f22`. audio-engine at `4545674` (rev 1 of its documents: `docs/ARCHITECTURE.md` §14, the optional UI, and `docs/REQUIREMENTS.md`); its rev 2 is not yet committed.

**Authority, in order:**
1. The owner's words: mazika's `docs/brief.md` §1 and §1a. The newest blocks are "Teams, and audio-engine as a product of its own" (mazika@738d4bf) and the plugin block after it (mazika@e8124d3). They win over older blocks where they differ.
2. This file.
3. mazika's `docs/ux-spec.md` §9 and `docs/research/ui-foundation-design.md`, for the behaviour this file carries over.

Nothing in the fork (`AGENTS.md`, `agents/`, `README.md`) is an authority for aui. It is Tylium's text for Tylium's contributors.

**Citations** follow one rule: `repo@sha:path §section "quote"`, where the quote is an exact substring of that file at that sha (whitespace aside). Line numbers are never cited.

**Evidence labels:**
- **measured** comes from a cited measurement;
- **(a choice)** marks a default this file picks, which the owner or aui's session may change;
- **UNVERIFIED** means no source confirms it yet;
- **SIM** is what a UI shows for anything synthesized or not measured.

**Ids.** Every requirement has an id `AUI-NNN`. The ids were fixed in a shared ledger, and audio-engine's `docs/COVERAGE.md` maps them, so they never change. Appendix A lists all 311 rows with their sources. Acceptance criteria are `AUI-AC-NN` (§8). Open questions are `Q-NN` (§9).

**Row status:**
- **settled:** the owner's words say so. The source quotes them, labelled `owner verbatim`: a blockquote in the brief's §1 or §1a, or a quote the brief marks as the owner's. The brief's own summary alone is not enough.
- **carried:** carried over from mazika's spec, its UI design or its code, or read from the fork; or a row whose only source is the brief's own record (labelled `the brief's record`), where the owner's words do not say it. It stands until a deliberate change.
- **open:** needs a decision. §9 gives each one a default.

---

## 0. The answer in one screen

- **aui is the owner's own UI kit for audio apps.** React components for the web: knobs, faders, buttons, keys, meters, and the parts around them (AUI-001, AUI-002, AUI-007, AUI-008).
- **It is fresh code.** None of the fork's code or text is reused. aui is only inspired by audio-ui (AUI-012, AUI-017). The owner chose mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Fresh code, own licence".
- **Its licence is still to be chosen** (AUI-016). Q-01 asks it first. No component code lands before the answer (AUI-185, default).
- **The fork's files go.** Each one is removed or replaced, and a provenance ledger records where every file came from (AUI-013, AUI-311).
- **No AudioUI, Cutoff or Tylium names, scope or branding** (AUI-167, AUI-287, AUI-310). Whether "aui" itself is too close to "AudioUI" is flagged (AUI-304, Q-02).
- **It has two consumers now.** mazika's `src/ui` first, then audio-engine's optional UI (AUI-014; audio-engine ARCHITECTURE §14).
- **It holds no app policy.** No take, keep or star words, no always-on capture, no ban on record or arm controls. Those are mazika's (AUI-014, AUI-015).
- **It is presentation only.** It never opens audio, MIDI or storage, and never talks to an engine. The app feeds it and hears back through callbacks (AUI-115, AUI-264).
- **It never gets in the way of real time.** It runs in a client, far from the audio thread. Live values arrive as feeds, drawn at most once per frame. It never shows a number nobody measured (§4.4).
- **Theming is data.** Token files, a generator, precomputed colours, and a theme can never change behaviour (§6).
- **It drops in under mazika's `src/ui`.** mazika's public UI API stays the same, so mazika's views never notice (§7).

---

## 1. Purpose and consumers

### 1.1 What aui is

- The owner's words: mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "I want you to build your own UI kit by using github.com/cutoff/audio-ui as inspiration, I've even forked it." (AUI-008).
- The owner also chose mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Write its requirements only". This file is those requirements. aui's own session builds the kit (AUI-011).
- It serves a DAW's interface, a plugin editor and device views alike (AUI-002). It is legible, follows the system's light or dark preference, and takes a manual setting (AUI-003).
- Its controls are dense and native-feeling, laid out in whole pixels (AUI-004). Every control carries a short name, a long name and info text (AUI-005).
- It is always web: React for web clients, including an iPad's web view and a Tauri webview (AUI-007).
- It targets React 19. mazika pins React 19.3.0 (AUI-160).
- It imports only its own tokens, React and an icon set, so every view, plugin editor and device view draws with the same parts (AUI-256).
- It does not depend on reactronica, and never depends on audio-ui (AUI-025, AUI-110).

### 1.2 Consumers

| Consumer | What it needs from aui | Status |
|---|---|---|
| **mazika's `src/ui`** | The controls behind its unchanged public API: Knob, Fader, ParamButton and Keys (today drawn or driven by audio-ui), plus the parts it built in-house (§7). | First consumer. Whether mazika switches is the owner's call (AUI-009, Q-06). |
| **audio-engine's optional UI** | A basic UI for the engine: sessions and roles, a patchbay of ports and pipes, devices and providers, clock domains, meters, a bus inspector, the Errors list, tokens and grants, modules and services (audio-engine ARCHITECTURE §14.1). React, with an optional Tauri shell. | Second consumer. It is a separate program that talks to the engine over its API. |
| **Other apps on the engine** | Plugin editors, device views, players. They build their own views from aui's parts. | Later. This is why aui holds no mazika policy (AUI-014). |
| **Skins and extensions by others** | Theming that works for skins made by someone else (AUI-024). | Later. |

The owner asked for mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "an optional basic ui". audio-engine's ARCHITECTURE §14.2 says the engine's UI draws with aui, and that aui's requirements live here. In audio-engine's rev 2 requirements that row, AE-UI-004, is `accepted` (the approved plan); until aui's public API is stable, the engine's UI uses plain React.

### 1.3 What aui is not

- **Not a way to audio or storage.** Its components never create an `AudioContext`, open a microphone or MIDI, or open storage (AUI-115). They take values as props and feeds, and report through callbacks (AUI-264).
- **Not an app.** It takes app knowledge, such as which computer keys play the keyboard, as props. It never reads an app's action registry (AUI-221, AUI-225).
- **Not mazika's policy.** mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "mazika's niche rules are mazika's policy, not the engine's." The same holds for the kit: no take, keep or star words in aui's names or default strings, and no always-on-capture assumption (AUI-014). Another app may build record or arm controls from aui's parts (AUI-015).
- **Not a theme owner.** aui ships one neutral default theme. mazika's themes (Graphite, Sticker Deck, Pip) stay mazika's (AUI-028). The kit names no theme (AUI-206, AUI-251).
- **Not a window manager.** A control never opens a window (AUI-178).

---

## 2. Licence, provenance and trademarks

I am not a lawyer. This section is a reading of the licence texts, and it asks the owner where a reading is not enough.

### 2.1 Fresh code, under the owner's licence

- aui is fresh code under the owner's own licence. mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "is fresh code under the owner's own licence (to be chosen)," (AUI-012, AUI-016, AUI-017).
- **The licence must fit its consumers** (AUI-019):
  - it must let GPL-3.0-only mazika use aui;
  - it must fit audio-engine's licence once that is chosen. The engine's UI is a separate program, so the two are decided separately;
  - it decides whether an MIT extension may bundle aui. Today an extension view that bundles mazika's kit is GPL (AUI-136). With a permissive aui, an MIT extension could bundle it; with a GPL aui, it could not (UNVERIFIED until the licence is chosen and read).
- **The licence comes first** (AUI-185). Q-01 asks the owner before any component code lands.
- Every aui source file then carries an SPDX header with aui's licence, and a test checks it (AUI-188).

### 2.2 No Tylium code or text

- **Nothing is copied from audio-ui:** no code, no CSS, no docs, no comments.
  - Its sources say aui@640c3d7:packages/react/src/index.ts "SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-TELF-1.0" (AUI-164, AUI-311).
  - Any copied code would make aui GPL (AUI-293, AUI-298).
  - Its documentation is licensed like its code, so aui copies no text from its README, `AGENTS.md` or `agents/` (AUI-307).
  - Its licence text is inconsistent (GPL-3.0-only at the root, or-later in the package files). Copying nothing avoids the question (AUI-165, AUI-309).
- **The commercial route is closed.** TELF forbids a kit that competes with audio-ui or repackages its components (AUI-166, AUI-294, AUI-300, AUI-301). aui never relies on Tylium later relaxing its licence (AUI-299).
- **Nothing goes upstream.** Contributing to audio-ui requires Tylium's CLA, which lets Tylium relicense contributions. aui's code never goes upstream (AUI-296, AUI-297, AUI-308).
- **Clean room (Q-04).** Whether aui's authors may read audio-ui's source while writing a counterpart is open (AUI-018). The default is strict: aui is written from this file and from mazika's own docs and code, not from audio-ui's source.
- **mazika's own code** may be ported into aui only where the owner holds all of its copyright, which is UNVERIFIED (AUI-130, Q-05). mazika's NOTICE says mazika@738d4bf:NOTICE "Copyright 2026 the mazika authors". The four mazika files built on audio-ui (Knob, Fader, ParamButton, Keys) are rewritten, not ported (a choice).
- Every test fixture is aui's own or synthetic, with a provenance comment. Nobody else's asset or theme file is committed (AUI-120).

### 2.3 The fork's files: removed or replaced

At `640c3d7` the fork holds 344 tracked files, all Tylium's. Each goes, or is replaced by a fresh file of aui's own. The provenance ledger (§2.4) starts with one row per fork file, marked `fork, pending removal`.

| Fork files | Count | Fate | Rows |
|---|---|---|---|
| `packages/core/**`, `packages/react/**` | 42 + 81 | Deleted. aui's code lives in new files. | AUI-311 |
| `apps/playground-react/**` | 184 | Deleted. aui's gallery is its own (AUI-187). | AUI-311 |
| `CLAUDE.md` (a symlink to `AGENTS.md`) | 1 | Replaced first, by tjrb's own instructions (HANDOFF §3, task T1). | AUI-284 |
| `AGENTS.md`, `agents/*.md` | 1 + 8 | Left alone until the other fork files are gone, then deleted. They are Tylium's rules and say Tylium's terms govern. | AUI-290 |
| `README.md` | 1 | Replaced by aui's own README. aui never presents itself as AudioUI, and the fork's text aimed at AI assistants goes. | AUI-292, AUI-295 |
| `.github/FUNDING.yml`, `.github/workflows/notify-discord-star.yml` | 2 | Deleted: they are Cutoff's. | AUI-282 |
| `.github/workflows/publish.yml` | 1 | Deleted, so nothing is ever published under Tylium's package names. | AUI-283 |
| `.github/workflows/ci.yml` | 1 | Replaced by aui's own workflow, run locally by bana (HANDOFF §3, task T5). | AUI-289 |
| `license-telf/*` | 4 | Deleted. | AUI-297 to AUI-308 |
| `LICENSE.md` (Tylium's GPL-3.0-only) | 1 | Stays only while fork files remain. At the end of task T3 it is replaced by a short notice: "Copyright the owner. No licence is granted yet; the licence will be chosen (Q-01)." Git history keeps Tylium's file. The owner's licence then replaces the notice. So no fresh aui file ever sits under Tylium's GPL notice. | AUI-291 |
| `CHANGELOG.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` | 3 | Deleted, or replaced by fresh files of aui's own. | AUI-311 |
| `scripts/**` | 4 | Deleted. | AUI-311 |
| Root configuration: `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.json`, `eslint.config.mjs`, `.prettierrc.json`, `.prettierignore`, `.gitignore`, `.vscode/settings.json` | 10 | Replaced by fresh files written for aui. None is edited in place. | AUI-311, AUI-310 |

The fork's git history and its GitHub fork link still hold Tylium's code, and the public repository still carries Tylium's description ("Dual-licensed GPL-3.0 / Commercial"). Whether aui keeps the history and the fork link, becomes private, or leaves the fork network is open (AUI-010, Q-03). Only the owner can change the description, the visibility or the fork link; aui's session cannot.

### 2.4 The provenance ledger

- **One row per tracked file** (a choice: `aui-docs/provenance.json`). Each row has:
  - `path`;
  - `origin`: `new`, `ported` (with `mazika@<sha>:<path>`), `third-party` (with the package and its SPDX id), or `fork, pending removal`;
  - `licence`: an SPDX id;
  - `note`: one sentence, when the origin needs one.
- **A test** fails when a tracked file has no row, when a row names a file that does not exist, or, once task T3 is done, when any row still says `fork` (AUI-AC-01).
- The ledger is how aui shows that it holds no Tylium code (AUI-306).

### 2.5 Names and trademarks

- **Tylium's marks** include the Tylium and Cutoff names and the product names tied to audio-ui (AUI-302). aui@640c3d7:license-telf/LICENSE.md §4.1.1 "Product and framework names associated with the Software".
- **So aui never uses them** in a component, theme, package, npm scope, CSS namespace, class name or domain (AUI-167, AUI-287, AUI-305, AUI-310). No `@cutoff` scope, no `--audioui-*` variables, no `audioui-*` classes.
- **aui may name audio-ui only to credit it** as inspiration, never in its names or branding (AUI-303).
- **No third party's product name, logo, panel graphics or trade dress** in aui's names, themes or components (AUI-006, AUI-020).
- **Flag: is "aui" too close to "AudioUI"?** (AUI-304, Q-02). The marks rule says aui@640c3d7:license-telf/LICENSE.md §4.1.3 "Not incorporate the Marks into their own product names". "aui" reads as the initials of "Audio UI". That rule binds TELF's licensees, and whether it or trademark law reaches an initialism is UNVERIFIED. The owner decides before anything is published under the name.

### 2.6 Dependencies and their licences

- aui keeps a licence ledger: the name, exact version and SPDX id of every package it bundles, transitive ones included, so an app can generate its NOTICE from its bundle (AUI-121, AUI-137).
- Runtime dependencies are pinned exactly (AUI-184, default).
- The planned runtime dependencies, with the licences mazika's generated NOTICE records for them:

| Package | Licence | Source |
|---|---|---|
| `react`, `react-dom`, `scheduler` | MIT | mazika@738d4bf:NOTICE "react 19.3.0, MIT" |
| `@phosphor-icons/react` (icons, Bold for off and Fill for on; AUI-066) | MIT | mazika@738d4bf:NOTICE "@phosphor-icons/react 2.1.10, MIT" |
| A bundled font for the default theme (Q-15; default Inter) | OFL-1.1, shipped unmodified with its text (AUI-112, AUI-209) | mazika@738d4bf:NOTICE "@fontsource-variable/inter 5.3.0, OFL-1.1" |
| `tslib`, if a build pulls it in | 0BSD, not MIT (AUI-121) | mazika@738d4bf:NOTICE "tslib 2.8.1, 0BSD" |

- Build and test tools (TypeScript, Vite, Vitest, Playwright) are not bundled. Their licences are recorded from each package's own licence file when aui adopts them: UNVERIFIED until then.

---

## 3. Components

### 3.1 The eight core controls

| Part | What it is | Key rows |
|---|---|---|
| **Knob** | `role=slider` with a valuetext (`−6.0 dB`), its long name as the name and its info text as the description. Label and value are always printed, also while dragging. A 270° value ring from the spec's origin (the centre for pan). Sizes 32, 44, 56 and 72 px (`knob-xl`). States: hover, focus, dragging, disabled with a reason, at default, automated, pending, not heard. | AUI-053 to AUI-055, AUI-103, AUI-172, AUI-226, AUI-270 |
| **Fader** | A groove, a fill, a cap, a 0 dB notch and a printed scale. States include at 0 dB, pending and disabled. Lengths 120, 180 and 320. Vertical, or horizontal as a crossfader (right is more). Fill and cap are drawn at the nearest whole pixel; the value is not rounded. | AUI-056, AUI-218, AUI-219, AUI-277 |
| **ParamButton** | A button bound to a parameter (mute, solo, listen, on). Its own focusable `role=button` element with `aria-pressed`, a stable long name that never carries state words, the info text as description, and a required, stable `paramId`. Latched or momentary. Latched ones paint across a group. The hit box is at least 24×24 whatever the face; the focus ring goes around the face. | AUI-058, AUI-059, AUI-105, AUI-149, AUI-177, AUI-233 to AUI-236 |
| **Button** | An HTML button that acts on release. Disabled stays focusable, and the reason is its tooltip and description. Icon-only buttons have an accessible name and a tooltip with the shortcut. | AUI-095, AUI-169, AUI-217 |
| **Toggle** | `role=switch`, 40×24, a check in the thumb. On and off differ in shape, not only fill. Prints its state word beside its name by default. | AUI-064, AUI-134, AUI-170, AUI-245, AUI-246 |
| **Segmented** | A radio group that shows every option, never a cycling button. A roving tab stop: Tab reaches the chosen option, arrows move and choose, Home and End go to the ends. Options meet with no gap inside a 24px switch. | AUI-064, AUI-134, AUI-171, AUI-205, AUI-240 |
| **Keys** | A playable keyboard of aui's own: multi-touch, glissando, pointer capture, `touch-action: none`, notes from elsewhere shown as held. Labelled as one image that names the computer keys that play it. Held notes at ≥ 3:1 against both key colours in every theme. | AUI-060, AUI-107, AUI-148, AUI-223, AUI-224 |
| **Meter** | A dark well with hard zone stops at −18, −6 and −1 dBFS, a peak tick that holds 2 s, a clip lamp that latches until clicked, and printed dB ticks on the IEC 60268-18 scale. States: no input, silent, signal, clipped, disconnected, SIM, stereo. The `meter` role sits on the well alone. A meter that is not listening prints no level at all, only words. | AUI-051, AUI-052, AUI-093, AUI-131, AUI-230 to AUI-232, AUI-260 to AUI-262 |

"Clip" on the meter is the industry word for a signal that hit full scale (AUI-052). It never names a piece of audio.

### 3.2 The parts around them

| Part | What it is | Key rows |
|---|---|---|
| **InfoView** | Shows the hovered or focused control's long name and info text. `aria-hidden`: the same text is the focused element's description, so a screen reader hears it once. Its state lives in a small external store, so hovering re-renders the panel and nothing else. | AUI-061, AUI-181 |
| **Tag** | 3–4-letter status tags in caps (`SIM`, `DEMO`, `LIVE`, `CLIP`), drawn as fill, dotted border or solid border. The words and their meanings come from the app (Q-14). | AUI-044, AUI-062, AUI-132, AUI-242 |
| **Banner** | A 36px band, info or warning, always one sentence and at most two buttons. A third action does not compile. | AUI-063, AUI-132, AUI-216 |
| **Toast** | Bottom centre, with an optional Undo. Words go to a polite live region; buttons stay out of it. It waits while hovered or focused, and at most three show at once. | AUI-064, AUI-132, AUI-243, AUI-244 |
| **Sheet** | Modal: Tab stays inside, Esc closes it, focus returns to the control that opened it. | AUI-133, AUI-241 |
| **Popover** | Anchored and not modal: Esc closes it and hands focus back; a press outside closes it. | AUI-133, AUI-237 |
| **Menu** | Up and Down wrap, Home and End, type-ahead by letter, Enter or Space chooses, Esc returns focus, Tab closes. A disabled item stays reachable and prints why. | AUI-133, AUI-228, AUI-229 |
| **HintBar and Keycap** | Button glyphs plus verbs, from plain props that fit the app's registry output. Keycaps after a key press; the pad's own glyph after a controller button. | AUI-065, AUI-220, AUI-221 |
| **Icon and Glyph** | Phosphor, Bold for off and Fill for on. Generic glyphs (hazard, octagon, lock, clock, presence shapes) on the same 24px grid with 2px strokes. No emoji anywhere. | AUI-066 to AUI-068, AUI-222, AUI-254 |
| **Error parts** | Show a structured error's fields: source, a readable sentence, cause, hint, an optional action and a time. A toast, a banner, a component's own status, and an Errors list with details and copyable text (§4.3). | AUI-021, AUI-022 |
| **Variants** | `knob-xl` (72px) and an ADSR graph (four handles, each a `role=slider`) are kit variants every theme can use, declared in token files. | AUI-035, AUI-050, AUI-211 |
| **Helpers** | Formatting (real minus signs, a value that rounds to zero has no sign, middle truncation so channel numbers survive), focus helpers that count `aria-disabled` elements as reachable, ids safe inside `url(#…)` and selectors, and a custom SVG cursor set. | AUI-096, AUI-108, AUI-252, AUI-253, AUI-255 |

### 3.3 The parameter model

- **`ParamSpec` is the one parameter descriptor.** Any other parameter shape is derived from it (AUI-106, AUI-142). It has:
  - a short name (`label`), printed and never truncated (AUI-266);
  - a long name and info text (AUI-005, AUI-267);
  - min, max, default, step, a fine step (a tenth by default) and a large step (ten steps by default) (AUI-268);
  - a unit, always printed after the number (AUI-269);
  - an origin for the value arc (AUI-270), and a silent minimum that prints `−∞ dB` (AUI-271);
  - a taper, mapping a value to a 0..1 position and back. The console fader law is piecewise linear in dB, with 0 dB near three quarters of the travel (AUI-273);
  - optional format and parse functions, which must include the unit (AUI-272).
- **Values are clamped and tidied to 1e-6,** so floating-point dust never prints (AUI-141, AUI-274).
- **Ready-made specs:** a gain in dB, silent at the minimum, and a pan (L, Centre, R) (AUI-275).
- **A MIDI-resolution view of a ParamSpec** replaces mazika's `toAudioParameter` and `toBooleanParameter`, which return audio-ui types (AUI-258).
- **Descriptor-generated panels** with human labels (AUI-111).
- Whether aui's `ParamSpec` maps one to one from the engine's parameter descriptors is open (AUI-265, Q-13).

### 3.4 What stays in mazika

These carry mazika's words and policy, so they stay in mazika's `src/ui` and draw with aui's parts: `KeepButton`, `TakeChip`, `TakeCard`, `TakeRegion`, `TrackSticker`, `ReelGauge`, `Reel`, `CassetteWindow`, `SourceChip` and `ParticipantBadge` (a choice, from AUI-014). A generic waveform drawing may move from `TakeRegion` into aui when the engine's UI needs one (AUI-259, Q-12). Bitmap film-strip controls are not in the first release (AUI-138, Q-11).

---

## 4. Behaviour

### 4.1 Pointer, wheel and keyboard

- **Drags are aui's own pointer-event drag, one gesture per pointer** (AUI-123). Two fingers on two controls never cross-talk (AUI-128). A second finger landing on a control already being dragged does not take the drag over (AUI-280).
- **Knobs drag vertically.** The full range is 160px on a knob and the travel on a fader (AUI-143). Controls can fill their cell on touch clients, while their text stays fixed-size (AUI-139).
- **The wheel:** one notch is one step, Shift is fine, up is more. The listener is native and non-passive, because React's `onWheel` is passive, so the page never scrolls under a control (AUI-144, AUI-279).
- **Keys:** arrows step, Shift is fine, Page Up and Page Down take large steps, Home and End go to the ends, Enter types a value (AUI-104, AUI-127, AUI-191).
- **Typing a value:** Enter and Escape end the edit and return focus to the control; blur commits and leaves focus where it went. Enter and Escape stop propagation, so a sheet or popover around the control stays open (AUI-145, AUI-281).
- **Every continuous control carries the same gesture help text** (AUI-278).
- **One commit per gesture.** A gesture commits once, when its own pointer is released, or when a typed value is committed. That is the app's moment to record one undoable edit (AUI-146, AUI-276).
- **Buttons:** parameter buttons flip on press, which makes painting work; momentary buttons sound only while held (AUI-177, AUI-194). A paint gesture belongs to one pointer, starts only on a parameter button, and paints only buttons of its own group (AUI-147). Whether painting toggles or sets is open (AUI-204, Q-10).
- **Listeners exist only during a gesture,** values are held in refs, and painting shares a few window listeners (AUI-159).

### 4.2 Values and words

- Values have at most six decimals (AUI-190). Units are always printed (AUI-046). The minus sign is U+2212, and silence prints `−∞ dB` (AUI-045).
- **Every status a component shows has a word;** the words themselves come from the app (AUI-099). Pending is a state word and no theme hides it (AUI-057, AUI-075).
- **Demo and simulated content is labelled in words** (`DEMO`, `SIM`). aui never invents a number: it draws only the values the app passes in, and the app marks simulated ones `SIM` (AUI-098, AUI-231).

### 4.3 Errors

- The owner's rule, verbatim: every failure becomes a structured event with mazika@738d4bf:docs/brief.md §1a (every error reaches the UI) "an id, a severity, the source (which layer or component), a sentence a musician can read, the underlying cause, a hint, an optional action (Retry, Open settings, Show details), and a time." audio-engine uses the same fields (its AE-ERR-003).
- **aui provides the parts that show errors where they matter:** a toast, a banner, a component's own status, and an Errors list with details and copyable technical text (AUI-021, AUI-022). The app decides which error goes where. aui never clears an error by hiding it; the app clears it when resolved or dismissed.
- **aui never swallows an error.** No empty catch blocks and no unhandled promise rejections; a failure inside aui reaches the app as a structured error through a callback. A test enforces it (AUI-023, AUI-AC-24).

### 4.4 Real time: the UI never gets in the audio's way

The owner asked, verbatim: mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Perhaps, how can we ensure real time? Since we are building a DAW / Player". The engine answers it for the audio path (audio-engine ARCHITECTURE §10). aui's part is to stay out of that path, and to keep the screen honest and smooth:

1. **aui never runs on the audio thread.** It runs in a web page. Every UI is a client: it holds a replica and a live stream and proposes edits (audio-engine AE-API-001, AE-UI-001). aui opens no audio, MIDI or storage (AUI-115). So a slow frame in aui costs a frame on screen, not a sample: the engine keeps its audio path from waiting on any client (audio-engine ARCHITECTURE §10).
2. **The audio never waits for the UI.** Live values reach aui as feeds the app subscribes to (AUI-264). A feed hands over its latest value; aui draws the newest and drops the rest. It never queues frames and never pushes back on its source (a choice, from AUI-230 and AUI-263).
3. **Meters draw straight to the DOM.** A live meter subscribes once and writes transforms each frame, with no React render per frame (AUI-157, AUI-230). The engine sends meters in a 30 Hz live snapshot (audio-engine AE-API-056).
4. **Automated and remote values are coalesced.** A control that follows automation or a remote hand is fed at most 30 times a second, coalesced to an animation frame, and the last value always lands (AUI-158, AUI-263).
5. **Edits are cheap and few.** A gesture commits once (§4.1). Listeners exist only during a gesture (AUI-159).
6. **Performance first, measured.** Minimal re-renders, no JavaScript for layout (AUI-286). No `will-change` hints: they caused a stacking bug and were the largest single cost at 256 moving knobs in mazika's kit (AUI-151, AUI-152). A `feed` path writes a knob's arc and needle directly once a view moves more than 128 controls at once (AUI-135).
7. **Nothing unmeasured gets a number.** aui is benchmarked against mazika's current kit at 64, 128 and 256 moving controls before it replaces it. Its own figures are UNVERIFIED until then (AUI-156, AUI-AC-26). A latency, a load or an xrun count reaches the screen only when the app passes a measured or reported value; otherwise the control says `SIM` or prints words (AUI-098, AUI-231).

---

## 5. Accessibility

aui carries mazika's legibility rules (spec §9.8) and the UI foundation design's fixes. They hold in every theme.

- **Names.** Every control has a short name (printed, never ellipsised), a long name (the accessible name, stable while dragging and whatever the state) and info text (the description on the focused element) (AUI-005, AUI-124, AUI-172, AUI-176, AUI-235).
- **Sliders** carry `aria-valuetext`, `aria-valuemin` and `aria-valuemax` (AUI-124).
- **Disabled controls stay focusable** and say why. Muting and disabling dim graphics, never words (AUI-097, AUI-125, AUI-217, AUI-229).
- **Focus is a real ring:** 3px at a 2px offset (4px on handheld and gamepad), drawn from tokens. Never a glow, never a CSS filter, never removed (AUI-092, AUI-109, AUI-126, AUI-250).
- **Targets:** 24px on dense desktop (WCAG 2.5.8), 44 on touch, 56 on handheld. Dense themes are dense in type and lines, not targets (AUI-032, AUI-094). `MIN_TARGET` (24) is exported (AUI-236).
- **Every state has a shape or a word,** never colour alone (AUI-030). Selection is tint plus outline (AUI-091). Line styles carry meaning: dashed for free and drop targets, dotted for pending and SIM, double for someone else here (AUI-082).
- **Contrast floors (WCAG 2.x):** text 4.5, primary text 7, graphics 3 (AUI-031, AUI-036). Text rules 2 to 5 of spec §9.8: text colours on neutral surfaces, no coloured text but links and the ok word, no weight below 400 under 20px, and controls show label and value with units in tabular figures (AUI-087 to AUI-090).
- **Text:** minimum 13px on desktop and in plugins, 14 on phone and tablet (inputs 16), 16 on instruments, 20 on stage and handheld (AUI-042). Sentence case everywhere; caps only for 3–4-letter tags (AUI-043, AUI-044). Sizes in rem, scaling 100, 115 or 130% (AUI-047). Control text has a fixed size and never scales with container-query units (AUI-048).
- **Motion:** no state is conveyed by motion alone, nothing flashes more than 3 times a second, and the clip lamp latches steady (AUI-085). Reduced motion stops every decorative motion; meters still update and peak hold still releases (AUI-080, AUI-084, AUI-238).
- **Live regions:** text that ticks every second is never a live region; only state changes are announced (AUI-239). Toast words go to a polite region (AUI-243).
- **Windows:** components work down to a 360px-wide phone (AUI-113).

---

## 6. Theming as data

- **A theme is data:** tokens for light and dark, type, and component variants, applied through CSS variables (AUI-026, AUI-102). Themes own presentation only; behaviour and data belong to the app (AUI-027).
- **A theme never changes behaviour.** Keyboard, ARIA, events and every state word are the same in all themes (AUI-029, AUI-183). A theme schema refuses any unknown key, and every refusal says why in a sentence (AUI-116, AUI-210).
- **One DOM for every theme.** Every part a theme may draw is always rendered, and a theme's CSS hides what it does not draw. No theme id appears in kit code (AUI-182, AUI-227, AUI-251).
- **The pipeline:** token JSON files are the source of truth. A generator writes the token CSS and a typed TypeScript map (AUI-033).
  - Colours are uppercase `#RRGGBB`, precomputed. No `light-dark()`, no runtime `color-mix()`, and nothing mixed at runtime (AUI-040, AUI-179, AUI-215, AUI-288).
  - The dark block is written twice instead of `light-dark()`, because older WKWebView plugin hosts lack it (AUI-073).
  - Theme values are never written as inline styles on `:root` at runtime (AUI-140).
  - Every colour in the kit's CSS is a token (AUI-249). Every theme defines the same colour tokens (AUI-034), including arc, dial, on-arc, lit-frame, held and the two key colours (AUI-173).
  - Geometry tokens set control sizes per theme, unitless where an SVG viewBox needs the number (AUI-074, AUI-174). SVG controls read them once per theme change, through one shared `MutationObserver` on `<html>` (AUI-168).
  - Motion durations and easings are tokens (AUI-083).
  - A full theme can be generated from an OKLab lightness ladder, and must pass the contrast check before it ships (AUI-214).
  - An override theme may restyle only colour tokens its base already defines. A theme package may set its own verified tokens; a user cannot (AUI-212, AUI-213).
- **Settings are attributes on `<html>`:** `data-mode` (system, light, dark; none follows the system), `data-theme`, `data-tint` (for themes with a temperature axis), `data-contrast`, `data-density` and `data-motion` (AUI-069 to AUI-072). CSS does the switching; a `matchMedia` listener only updates the "Auto" label (AUI-076). aui's CSS never reads a `.dark` class (AUI-081).
- **System preferences:** `prefers-contrast: more` sets 2px control and outline lines in every theme; a parameter button keeps a 1px unlit line against its 2px lit frame (AUI-077, AUI-078, AUI-180). `forced-colors` maps tokens to system colours; only the meter well opts out (AUI-079).
- **Contrast is checked in CI.** Each theme's pair list names what the CSS draws; CI recomputes every pair from the token files and fails below a floor. The check has a self-test, and the held-key row runs for every theme (AUI-037 to AUI-039, AUI-100, AUI-207, AUI-208).
- **CSS hygiene:**
  - aui's stylesheet styles only its own classes, with no global element rules (AUI-150).
  - It enters an app through an explicit import into a named cascade layer. aui's JavaScript never imports CSS as a side effect (AUI-153, AUI-202).
  - Overlays on SVG controls are HTML, not `foreignObject`, for WebKit (AUI-154).
  - Whole pixels: viewBox equals CSS size, 1px strokes on .5 coordinates, offsets rounded (AUI-086, AUI-155).
- **Fonts and isolation:** fonts are bundled locally and never fetched, including in a plugin (AUI-041). aui never bundles or names a host's own UI face (AUI-049). aui works under COOP/COEP `require-corp`: it loads nothing from elsewhere, and image-based controls use bundled images (AUI-162, AUI-163).
- **Namespace:** which token namespace aui uses is open (AUI-175, Q-07). The prefix is one generator setting (a choice), so choosing the name later costs one run.
- **aui's own theme:** aui ships one neutral default theme, so its gallery and the engine's UI work without mazika (AUI-028).

---

## 7. The drop-in contract with mazika's `src/ui`

### 7.1 What exists today

- mazika's `src/ui` is its kit, `@mazika/ui`. Views import only from it. audio-ui sits behind it: mazika@738d4bf:docs/ux-spec.md §9.9 "always behind the unchanged `src/ui` (`@mazika/ui`) API, so views never see it." (AUI-101, AUI-114).
- audio-ui draws the Knob and the Fader, drives ParamButton's state machine, and is Keys (AUI-257). mazika imports it in six files only: `Knob.tsx`, `Fader.tsx`, `ParamButton.tsx`, `Keys.tsx`, `param-adapter.ts` and `audio-ui.contract.ts`.
- `src/ui/audio-ui.contract.ts` records, as types, the audio-ui parts mazika rests on: mazika@738d4bf:src/ui/audio-ui.contract.ts "declared stable. This file is a type-level snapshot of exactly those". They are five SVG primitives, one interaction controller, and nine props of Keys.

### 7.2 What the contract file means for aui

- **aui's authors do not open it** (clean room, Q-04; HANDOFF §1). This section and §7.1 already say what it holds, so the file is not needed.
- **It is the list of what to replace, not signatures to copy** (AUI-247). aui draws rings, strips and cursors in its own way.
- **aui's primitives are not named after audio-ui's** (`ValueRing`, `RotaryImage`, `LinearStrip`, `ValueStrip`, `LinearCursor`) and do not reproduce their prop lists (AUI-248).
- **aui writes its own latch, momentary and paint state machine,** replacing `BooleanInteractionController` (AUI-233), and its own keyboard, replacing audio-ui's Keys (AUI-223).

### 7.3 The contract

**mazika's public UI API does not change.** mazika@738d4bf:docs/research/ui-foundation-design.md §0 "`src/ui`'s public API does not change, except that `ParamButton` (new) requires a stable `paramId`." (AUI-129). mazika's views keep importing from `src/ui`; only what sits behind it changes.

| mazika export (`src/ui/index.ts`) | Today | aui supplies | Way in |
|---|---|---|---|
| `Knob`, `knobArc`, `knobRing`, `KnobProps`, `KnobSize` | mazika's code on audio-ui's `ValueRing` and `RotaryImage` | Knob with the same props and meaning, except that mazika's `track` becomes a generic accent, a token name (a choice) | a thin wrapper in mazika maps `track` |
| `Fader`, `faderPixel`, `faderScale`, `FaderLength`, `FaderProps` | on `LinearStrip`, `ValueStrip`, `LinearCursor` | Fader, same props, `track` as for Knob | a thin wrapper |
| `ParamButton`, `MIN_TARGET`, `ParamButtonProps` | on `BooleanInteractionController` | ParamButton, same props | re-export |
| `Keys`, `KeyName`, `KeysProps` | audio-ui's Keys | Keys, same props (`label`, `notesOn`, `onNote`, `keys`, `startKey`, `octaveShift`, `width`, `height`, `playHint`) | re-export |
| `toAudioParameter`, `toBooleanParameter` | return audio-ui types; used only by a mazika test | a MIDI-resolution view of a ParamSpec (AUI-258) | **removed** from mazika's API: the one stated change. No view uses them. |
| `ParamSpec` and the `param.ts` helpers; `param-feed.ts`; `useParamControl`, `CONTROL_HELP` | mazika's own | the parameter model (§3.3) | re-export, after Q-05 (porting) or a rewrite |
| `Meter` and `meter-scale.ts`; `Button`; `Toggle`; `Segmented`; `Tag`; `Banner`; `Toast`; `Sheet`; `Popover`; `Menu`; `HintBar`; `InfoView`; `Icon`; `Glyph`; `format`, `focus`, `ids` helpers | mazika's own | the same parts (§3.1, §3.2) | re-export, or a thin wrapper where mazika supplies its words (`Tag` kinds, `Toggle` state words) |
| `KeepButton`, `TakeChip`, `TakeCard`, `TakeRegion`, `TrackSticker`, `ReelGauge`, `Reel`, `CassetteWindow`, `SourceChip`, `ParticipantBadge` | mazika's own | nothing: they stay mazika's (§3.4) | unchanged, drawn with aui's parts |

**What aui promises:**
1. **Declared-stable exports.** aui states which exports are stable, and mazika depends only on those (AUI-122). audio-ui declared stable only its high-level components, while mazika leaned on its undeclared primitives. aui must not repeat that.
2. **The same meaning,** not just the same types: the behaviour in §4 and §5, proven by aui's own suite (§8) and by mazika's suites run against aui (AUI-AC-27).
3. **No policy words** in aui's names or default strings. mazika supplies its words as props (AUI-014, AUI-099).
4. **One stylesheet in one named layer,** entered by an explicit import (AUI-153, AUI-202).
5. **Exact versions.** mazika pins aui exactly, as it pins audio-ui today.

**What mazika does when it switches** (mazika's work, listed so both sides agree):
- replaces `audio-ui.contract.ts` with a type-level snapshot of the aui exports it uses;
- changes `src/boundaries.test.ts` so aui, not `@cutoff/audio-ui-react`, is allowed in `src/ui` only;
- regenerates its NOTICE, and drops the audio-ui entry and the `--audioui-*` block from its token generator;
- maps its `--mz-*` tokens onto aui's namespace in its own generator (Q-07).

Whether and when mazika switches is the owner's call (AUI-009, Q-06).

### 7.4 audio-engine's optional UI

- It is a client like every other: a replica, a live stream, proposed edits, never audio or storage (audio-engine AE-UI-001). React, optionally in a Tauri webview (AE-UI-003).
- It draws with aui's parts and aui's default theme. It needs no mazika part.
- It shows every structured error event where it matters and keeps an Errors list (AE-UI-005): aui's error parts (§4.3).
- It labels every figure measured, SIM or UNVERIFIED, with aui's Tag and the app's words (AUI-098, AUI-242).
- Whether aui adds a list or table part, or a patchbay part, for it is open (Q-17). The default: the engine's UI builds them from aui's parts, and aui adds a part only when two consumers need it.

---

## 8. Acceptance criteria

`unit` is a unit test (Vitest, a choice), `PW` a browser test (Playwright, a choice), `CI` a check on every build, and `bench` a recorded benchmark. Each criterion runs under every theme aui ships. Tests aui cannot yet run say so; nothing is claimed before it runs.

| AC | Check | Kind | Rows |
|---|---|---|---|
| AUI-AC-01 | **Provenance.** Every tracked file has a ledger row and every row names a real file. After task T3, no row says `fork`. The test has a self-test that fails a planted fork file. | unit | AUI-013, AUI-306, AUI-311 |
| AUI-AC-02 | **Headers.** Every source file carries the SPDX header of aui's licence. | unit | AUI-188 |
| AUI-AC-03 | **Names.** No shipped file, package name, class, CSS variable or export contains `audioui`, `audio-ui`, `@cutoff`, `Cutoff` or `Tylium`. Only the README's credit line and the provenance ledger may name audio-ui. With a self-test. | unit | AUI-167, AUI-287, AUI-303, AUI-310 |
| AUI-AC-04 | **Licence ledger.** Every package in a production build appears with its exact version and SPDX id. | unit | AUI-121, AUI-137 |
| AUI-AC-05 | **Contrast.** Every pair of every shipped theme meets 4.5, 7 and 3, in light and dark and under More. The held-key row runs for every theme. A planted broken palette fails. | CI | AUI-036, AUI-039, AUI-100, AUI-117, AUI-189, AUI-207, AUI-208 |
| AUI-AC-06 | **Generated CSS.** No `light-dark(`, no `color-mix(`, only uppercase `#RRGGBB` colours, and no runtime write to `:root`. | unit | AUI-040, AUI-140, AUI-179, AUI-215 |
| AUI-AC-07 | **Cascade.** The built CSS has no rule outside aui's named layer and no global element selector, and aui's JavaScript imports no CSS. | unit (on the build) | AUI-150, AUI-153, AUI-202 |
| AUI-AC-08 | **Theme cannot change behaviour.** The schema refuses an unknown key with a sentence. The behaviour suite passes identically under every theme. | unit | AUI-116, AUI-183, AUI-201, AUI-210 |
| AUI-AC-09 | **Sliders with words.** Every `role=slider` has min, max, now and a valuetext that ends with its unit or is a word. The name is the long name and does not change while dragging. `aria-valuenow` has at most six decimals after any drag. | PW | AUI-124, AUI-172, AUI-190 |
| AUI-AC-10 | **Keys and drags.** With step 1: ArrowUp +1, Shift+ArrowUp +0.1, PageUp +10, Home the minimum. Enter, type 77, Enter sets 77 and returns focus; Escape abandons without reaching the page. A 40px drag over a 160px knob moves 25% of the range. One wheel notch is one step, and the page does not scroll. | PW | AUI-127, AUI-143, AUI-144, AUI-145, AUI-191 |
| AUI-AC-11 | **Independent fingers.** Two touches on two knobs move them independently; lifting one leaves the other's drag running. | PW | AUI-128, AUI-186, AUI-280 |
| AUI-AC-12 | **One commit per gesture.** A drag commits once, on release of its own pointer, with the final value. Another pointer's release does not commit it. | unit | AUI-146, AUI-192, AUI-276 |
| AUI-AC-13 | **Paint and hold.** Painting S across three buttons presses all three and never changes an M. A touch dragging a fader across a parameter button does not change it. Two touches on Keys hold two notes. A momentary button is pressed only while held. | PW | AUI-147, AUI-193, AUI-194 |
| AUI-AC-14 | **Info view.** Hover or focus fills the info view within one frame. The same text is the focused element's `aria-describedby`, on knobs and on parameter buttons, and the info view is `aria-hidden`. | PW | AUI-061, AUI-195 |
| AUI-AC-15 | **Names never cut, and stable.** No short name has `scrollWidth > clientWidth` in the gallery. Long names and `paramId`s are unique and do not change with state. | PW, unit | AUI-176, AUI-196, AUI-197, AUI-235 |
| AUI-AC-16 | **Whole pixels.** Every control box and face has integer bounds (±0.01) at `devicePixelRatio` 1. | PW | AUI-086, AUI-155, AUI-198 |
| AUI-AC-17 | **Focus is a ring.** A focused control has a 3px outline and `filter: none`. | PW | AUI-092, AUI-126, AUI-199, AUI-250 |
| AUI-AC-18 | **Targets.** Every interactive element is at least 24×24 or meets WCAG 2.5.8's spacing exception, and no two targets overlap. | PW | AUI-059, AUI-094, AUI-203, AUI-205 |
| AUI-AC-19 | **Minimum text and reduced motion.** Every visible text node meets the client's minimum. Under reduced motion, decorative animation stops while meters still update. | PW | AUI-042, AUI-084, AUI-118, AUI-119 |
| AUI-AC-20 | **Disabled says why.** A disabled control or menu item stays focusable and exposes its reason as its description. | PW | AUI-097, AUI-125, AUI-217, AUI-229 |
| AUI-AC-21 | **Isolation.** With aui loaded, `crossOriginIsolated` stays true and no request leaves the origin. | PW | AUI-041, AUI-162, AUI-200 |
| AUI-AC-22 | **Contrast More and forced colours.** Under More, control and outline lines are 2px. Under forced colours, tokens map to system colours and only the meter well opts out. | PW | AUI-077, AUI-078, AUI-079 |
| AUI-AC-23 | **Meter.** Each state renders its word. A meter that is not listening prints no level. Clip latches until clicked. The zones and ticks sit at the stated dBFS. A live feed of 1,000 frames causes no React render of the meter. | unit, PW | AUI-051, AUI-052, AUI-157, AUI-230, AUI-231, AUI-260, AUI-262 |
| AUI-AC-24 | **Errors.** No empty catch block and no unhandled rejection in aui's source, checked by a test with a self-test. Each error part renders every field it is given; the Errors list copies its technical text. | unit | AUI-021, AUI-022, AUI-023 |
| AUI-AC-25 | **No policy.** No export name or default string of aui contains take, keep or star as a word. | unit | AUI-014 |
| AUI-AC-26 | **Performance, measured.** A benchmark of 64, 128 and 256 moving controls against mazika's current kit, on a named machine and browser, recorded with the date. No figure is claimed before it runs. The bundle size is recorded the same way. | bench | AUI-135, AUI-156, AUI-161, AUI-286 |
| AUI-AC-27 | **Drop-in.** On a mazika branch with `src/ui` backed by aui, mazika's `pnpm typecheck`, `pnpm test` and `pnpm test:e2e` pass, and `src/ui/index.ts` exports the same names except `toAudioParameter` and `toBooleanParameter`. This one runs in mazika. | mazika's suites | AUI-101, AUI-129, AUI-258 |

---

## 9. Open questions

Each has a default, used until the owner answers.

| Q | Question | Default | Rows |
|---|---|---|---|
| Q-01 | **Which licence does aui use?** It must let GPL-3.0-only mazika use aui, and it decides whether MIT extensions may bundle it. | None. Ask the owner first. No component code lands before the answer; the docs, the ledger, removals and CI can. | AUI-016, AUI-019, AUI-136, AUI-185 |
| Q-02 | **Is the name "aui" too close to "AudioUI"?** | Keep "aui" as the repository's name for now. Choose the published package name, npm scope and token prefix with the owner before anything is published. | AUI-304, AUI-310 |
| Q-03 | **Keep the fork's history and fork link, or start fresh? Make the repository private, or leave the fork network?** Also: change the GitHub description, which still reads "Dual-licensed GPL-3.0 / Commercial". | Keep the history for now: history is not reuse. The owner changes the description at once (T0). Private, or out of the fork network, is the owner's choice before the first release. The session can do none of these. | AUI-010 |
| Q-04 | **May aui's authors read audio-ui's source?** | No. Write from this file and mazika's own docs and code. The fork's source is removed early (task T3). | AUI-018 |
| Q-05 | **May mazika's in-house `src/ui` files be ported into aui?** Only if the owner holds all their copyright (UNVERIFIED). | Rewrite until the owner confirms. The four files built on audio-ui are always rewritten. | AUI-130 |
| Q-06 | **Does mazika move from audio-ui to aui, and when?** | aui builds to the contract (§7). The owner decides the switch once AUI-AC-27 passes. | AUI-009 |
| Q-07 | **Which token namespace?** mazika reads `--mz-*` today. | aui's own prefix, chosen with the name (Q-02). mazika's generator maps its tokens onto it. | AUI-175 |
| Q-08 | **A framework-free core under the React layer?** | Yes, as a folder inside one package: the parameter model and interaction logic in plain TypeScript, React on top. No second framework now. | AUI-285 |
| Q-09 | **Should the keys become focusable, with roles?** | No: one labelled image that names the computer keys that play it, as mazika has. | AUI-224 |
| Q-10 | **Painting: toggle each entered button, or set each to the first one's new state?** | Toggle, as mazika's design proposes; revisit after the owner tries it. | AUI-204 |
| Q-11 | **Bitmap film-strip and image controls?** | Not in the first release. | AUI-138 |
| Q-12 | **A generic waveform drawing in aui?** | Stays in mazika's `TakeRegion` until the engine's UI needs one. | AUI-259 |
| Q-13 | **Does `ParamSpec` map one to one from the engine's parameter descriptors?** | `ParamSpec` stays the UI's shape; each app maps its descriptors into it. | AUI-265 |
| Q-14 | **Tag words.** | The app supplies its tag words and meanings; aui supplies the shapes (fill, dotted, solid). | AUI-242 |
| Q-15 | **Which font does aui's default theme bundle?** | Inter (OFL-1.1), already vetted in mazika's NOTICE. | AUI-028, AUI-209 |
| Q-16 | **CI on a public repository.** bana's own rule is bana@04a5f22:README.md §CI on push: bana daemon "Use the daemon only on a private repository, where write access is the gate." aui is a public fork. bana's default `daemon.branches` runs every branch except bot branches: bana@04a5f22:docs/DAEMON.md §Settings "`* !dependabot/* !renovate/*`". So AI sessions' `claude/*` pushes would run, as the owner, on the owner's Mac. An empty token does not stop that: bana@04a5f22:docs/DAEMON.md §Trust "a host job can still run `gh auth token` itself, so the real limit is the trust above". | No daemon on a public repository. The owner runs `bana ci` by hand, after reading the diff. If the owner makes aui private and wants the daemon, `daemon.branches` names one branch that only the owner pushes to (for example `ci/*`, fast-forwarded after the owner reads the diff), never `claude/*`. | AUI-289 |
| Q-17 | **Extra parts for the engine's UI** (a list or table, a patchbay)? | The engine's UI builds them from aui's parts. aui adds a part when two consumers need it. | AUI-014 |
| Q-18 | **aui's bundle budget.** audio-ui cost mazika about 25 KB gzip. | Measure first (AUI-AC-26); no budget is set before a measurement. | AUI-161 |

The rows marked open that are not questions are adopted as written, and done as first tasks (HANDOFF §3): AUI-013, AUI-014, AUI-015, AUI-028, AUI-184, AUI-247, AUI-248, AUI-258, AUI-282, AUI-283, AUI-284, AUI-286, AUI-289, AUI-292, AUI-295 and AUI-310.

---
## Appendix A. Every AUI row

The 311 rows placed with aui in the shared ledger, in id order. "Where" names the sections of this file that use the row. Each source is `repo@sha:path §section "quote"`; where a quote held a double quote, the longest part without one is shown. A `\|` in a table cell is a literal `|` in the source.

| Id | Requirement | Status | Kind | Where | Source |
|---|---|---|---|---|---|
| AUI-001 | UI controls take inspiration from reactronica and audio-ui. | settled | mechanism | §0 | mazika@738d4bf:docs/brief.md §1 (the owner's framework, 2026-09-26) "Use https://github.com/unkleho/reactronica and https://github.com/cutoff/audio-ui to get inspiration or build your UI." |
| AUI-002 | The kit serves a DAW interface, a plugin editor and device views alike. | settled | mechanism | §0, §1.1 | mazika@738d4bf:docs/brief.md §1 (the owner's framework, 2026-09-26) "After all we have to build our DAW interface and plugin as well as different views" |
| AUI-003 | The kit is legible, follows the system's light or dark preference, and also takes a manual setting. | settled | mechanism | §1.1 | mazika@738d4bf:docs/brief.md §1 (the owner's framework, 2026-09-26) "It needs to be **legible** and be able to **take the system's theme preference and have also a manual setting**."<br>mazika@738d4bf:docs/brief.md §1 (the owner's framework, 2026-09-26) "take the system's theme preference and have also a manual setting"<br>mazika@738d4bf:docs/brief.md §2 (What that means for the PoC) "Themable: follows the system light/dark preference AND has a manual override." |
| AUI-004 | The kit supports dense, native-feeling controls laid out in whole pixels. | settled | mechanism | §1.1 | mazika@738d4bf:docs/brief.md §1a (the UIs are built on audio-ui), owner verbatim "use the audio-ui components ... to build the actual UIs upon the guidelines by cycling 74"<br>mazika@738d4bf:docs/brief.md §1a (the UIs are built on audio-ui), the brief's record "whole-pixel layout;" |
| AUI-005 | Every kit control carries a short name, a long name and info text. | settled | mechanism | §1.1, §3.3, §5 | mazika@738d4bf:docs/brief.md §1a (the UIs are built on audio-ui), owner verbatim "use the audio-ui components ... to build the actual UIs upon the guidelines by cycling 74"<br>mazika@738d4bf:docs/brief.md §1a (the UIs are built on audio-ui), the brief's record "clear short and long names, and info text."<br>mazika@738d4bf:docs/research/ui-foundation-design.md §3.5 Short names, long names, info text "**Every control has three names:**" |
| AUI-006 | aui copies no third party's logos or assets and uses no product name as its own. | carried | constraint | §2.5 | mazika@738d4bf:docs/brief.md §1a (the UIs are built on audio-ui), the brief's record (no owner words say it) "Nothing of Ableton's is copied: no logos or assets, and no product name used as mazika's own." |
| AUI-007 | The kit is web: React components for web clients, including the iPad's web view and a Tauri webview. | settled | constraint | §0, §1.1 | mazika@738d4bf:docs/brief.md §1a (a DJ view; answer 6), owner verbatim "Native core. We need good performance especially with the mixer. The UI is always web."<br>mazika@738d4bf:docs/brief.md §1a (the native core's questions, answered), the brief's record "The UI is always web (the DJ answer 6)." |
| AUI-008 | aui is the owner's own UI kit, inspired by audio-ui, built by its own session. | settled | constraint | §0, §1.1 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) ", which is another claude project"<br>mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "I want you to build your own UI kit by using github.com/cutoff/audio-ui as inspiration, I've even forked it. It's called" |
| AUI-009 | Whether mazika's controls move from audio-ui to aui is not yet decided. | open | mechanism | §1.2, §7.3, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "I want you to build your own UI kit by using github.com/cutoff/audio-ui as inspiration" |
| AUI-010 | Whether aui keeps the fork's git history and fork link, which hold Tylium's GPL code, or starts a fresh history. | open | constraint | §2.3, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "I've even forked it." |
| AUI-011 | For aui, only its requirements are written here; aui's session builds the kit. | settled | constraint | §1.1 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) ": aui's own Claude session builds the kit"<br>mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) ": aui's own Claude session builds the kit." |
| AUI-012 | aui is fresh code under the owner's own licence; none of the fork's code is reused. | settled | constraint | §0, §2.1 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Fresh code, own licence"<br>mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "[aui: a GPL derivative of the fork, or fresh code] the owner chose"<br>mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "only inspired by audio-ui. None of the fork's code is reused, and aui's own session builds it." |
| AUI-013 | aui keeps a provenance ledger: every file records whether it is new, ported from the owner's own mazika code, or a fork file awaiting removal. | open | mechanism | §0, §8, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "[aui: a GPL derivative of the fork, or fresh code] the owner chose" |
| AUI-014 | aui serves several consumers (mazika's src/ui, the engine's optional UI, other apps), so it holds no mazika policy: no take, keep or star words and no always-on-capture assumptions. | open | constraint | §0, §1.2, §1.3, §3.4, §7.3, §8, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "mazika's niche rules are mazika's policy, not the engine's." |
| AUI-015 | aui does not bake in mazika's no-record, no-arming rule: another app may build record or arm controls from aui's parts. | open | constraint | §0, §1.3, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Those are always-on capture, immutable takes, no arming, and the word take (never clip)." |
| AUI-016 | Which licence aui uses is still to be chosen by the owner. | open | constraint | §0, §2.1, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "is fresh code under the owner's own licence (to be chosen)," |
| AUI-017 | aui is only inspired by audio-ui and built by aui's own session; its licence is to be chosen. | settled | constraint | §0, §2.1 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) (aui), owner verbatim "Fresh code, own licence"<br>mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own), the brief's record "only inspired by audio-ui. None of the fork's code is reused, and aui's own session builds it" |
| AUI-018 | Whether aui's authors may read audio-ui's source while writing its counterpart, or only its public behaviour (a clean-room rule). | open | constraint | §2.2, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "only inspired by audio-ui. None of the fork's code is reused, and aui's own session builds it." |
| AUI-019 | aui's licence must let GPL-3.0-only mazika use it, must fit the engine's licence once chosen, and decides whether MIT extensions may bundle it. | open | constraint | §2.1, §9 | mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Licensing of audio-engine is decided later." |
| AUI-020 | aui's names, themes and components never use a third party's product name, logo, panel graphics or trade dress. | carried | constraint | §2.5 | mazika@738d4bf:docs/brief.md §1a (product names, and a session style when a device connects), the brief's record (no owner words say it) "They are never used for mazika's own modes, skins or features, and never with others' logos, panel graphics or trade dress, or in a way that implies endorsement." |
| AUI-021 | aui's error parts show the fields of a structured error: source, a readable sentence, cause, hint, an optional action and a time. | settled | mechanism | §3.2, §4.3, §8 | mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), owner verbatim "The UI needs to show any error that occurs on the way."<br>mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), the brief's record "an id, a severity, the source (which layer or component), a sentence a musician can read, the underlying cause, a hint, an optional action (Retry, Open settings, Show details), and a time." |
| AUI-022 | aui provides the parts that show errors where they matter: a toast, a banner, a component's own status, and a list of errors with details and copyable text. | settled | mechanism | §3.2, §4.3, §8 | mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), owner verbatim "The UI needs to show any error that occurs on the way."<br>mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), the brief's record "The UI shows each one where it matters (a toast, a banner, or the component's own status) and keeps all of them in an Errors list, with details and copyable technical text." |
| AUI-023 | aui never swallows an error: no empty catch blocks and no unhandled promise rejections; a failure reaches the app as a structured error. | settled | constraint | §4.3, §8 | mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), owner verbatim "The UI needs to show any error that occurs on the way."<br>mazika@738d4bf:docs/brief.md §1a (every error reaches the UI), the brief's record "In code: no empty catch blocks and no swallowed promise rejections." |
| AUI-024 | Views are skinnable and themeable, and users can build their own skins and views as extensions, so aui's theming must work for skins made by others. | settled | mechanism | §1.2 | mazika@738d4bf:docs/brief.md §1a (a DJ view) "The DAW is built on skinnable / theme able views that can be used based on the user's own stack. We are adapting, and allowing the user to build extensions." |
| AUI-025 | aui studies audio-ui and never depends on it; the old default returns for aui after the GPL detour. | carried | constraint | §1.1 | mazika@738d4bf:docs/brief.md §2 (What that means for the PoC) "**Licensing default:** study audio-ui, do not depend on it" |
| AUI-026 | A kit theme is data: tokens for light and dark, type, and component variants. | carried | mechanism | §6 | mazika@738d4bf:docs/brief.md §2a (Themes decide views, not just colours) "visual tokens (light and dark), type, component variants" |
| AUI-027 | Themes own presentation only; behaviour and data belong to the app. | carried | constraint | §6 | mazika@738d4bf:docs/brief.md §2a (Themes decide views, not just colours) "The framework (mazika) owns behaviour and data; themes own presentation and composition." |
| AUI-028 | aui ships its own neutral default theme; mazika's themes stay mazika's. | open | mechanism | §1.3, §6, §9 | mazika@738d4bf:docs/ux-spec.md §9.0 Themes "The framework bundles two looks, plus the Pip synth's theme:" |
| AUI-029 | Themes change presentation only: keyboard, ARIA, events and every state word are the same in all of them. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.0 Themes "Themes change presentation only: the keyboard, the ARIA, the events and every state word are the same in all of them." |
| AUI-030 | Every state has a shape or a word, never colour alone. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.1 Identity: Sticker Deck "**Every state has a shape or a word.**" |
| AUI-031 | Every word and number meets the contrast floors. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.1 Identity: Sticker Deck "**Every word and number meets the contrast floors** (§9.2.4)." |
| AUI-032 | Targets stay 24px even in dense themes: density comes from type and lines, not targets. | carried | constraint | §5 | mazika@738d4bf:docs/research/ui-foundation-design.md §6 Risks and open questions for the owner "Proposed: keep the rule; Graphite is dense in type and lines, not in targets."<br>mazika@738d4bf:docs/ux-spec.md §9.1a Identity: Graphite "**targets never under 24px:** density comes from type and lines, never from targets" |
| AUI-033 | Token JSON files are the source of truth; a generator writes the token CSS and a typed TypeScript map. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.2 Tokens "`scripts/gen-tokens.ts` generates `tokens.css` and a typed TypeScript map from them." |
| AUI-034 | Every theme defines the same colour tokens. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.2 Tokens "Every theme defines the same colour tokens; the tables below are Sticker Deck's." |
| AUI-035 | A 72px knob size (knob-xl) and an ADSR graph are kit variants every theme can use. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.2.3 Pip theme "The component variants `knob-xl` (72px) and `adsr-graph` are not Pip's: they belong to the framework's `@mazika/ui` (§9.5)" |
| AUI-036 | Contrast floors (WCAG 2.x): text 4.5, primary text 7, graphics 3. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.2.4 Contrast "floors: text 4.5, primary text 7, graphics 3"<br>mazika@738d4bf:src/themes/contrast.test.ts §floors "uses the spec's floors: text 4.5, primary text 7, graphics 3" |
| AUI-037 | Each theme's pair list names what the CSS draws. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.2.4 Contrast "The pair list is per theme: every row names what the CSS draws." |
| AUI-038 | A held key's colour is solved against both key colours in every theme. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.2.4 Contrast "`--mz-held` is solved against both key colours, in every theme." |
| AUI-039 | CI recomputes every contrast pair from the token files and fails the build below a floor. | carried | mechanism | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.2.4 Contrast "**CI:** `scripts/contrast.ts` recomputes every pair from the token files" |
| AUI-040 | Generated tokens use no light-dark() and no runtime color-mix(). | carried | constraint | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.2.5 Graphite "the generator writes precomputed `#RRGGBB` for every token: no `light-dark()`, no runtime `color-mix()`." |
| AUI-041 | Fonts are bundled locally; there is never a network fetch, including in a plugin. | carried | constraint | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "There is never a network fetch, including in the plugin." |
| AUI-042 | Minimum text: 13px desktop and plugin, 14 phone and tablet (inputs 16), 16 instruments, 20 stage and handheld. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "\| Desktop and plugin \| 13 \|" |
| AUI-043 | Sentence case everywhere. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "Sentence case everywhere." |
| AUI-044 | All caps only for 3-4 letter tags at 13px, weight 700, +0.06em. | carried | constraint | §3.2, §5 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "All caps only for 3–4-letter tags (`SIM`, `DEMO`, `LIVE`, `CLIP`) at 13px, weight 700, +0.06em tracking."<br>mazika@738d4bf:src/ui/Tag.tsx §file header "Status tags (spec §9.5 Tags, §9.4): the only all-caps words in the app, at" |
| AUI-045 | The minus sign is U+2212 and silence prints as −∞ dB. | carried | constraint | §4.2 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "The minus sign is U+2212, and silence is `−∞ dB`." |
| AUI-046 | Units are always printed. | carried | constraint | §4.2 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "Units are always printed." |
| AUI-047 | Sizes are in rem, and text size scales 100, 115 or 130%. | carried | mechanism | §5 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "Sizes are in rem; text size scales 100/115/130%." |
| AUI-048 | Control text has a fixed size; labels never scale with container-query units. | carried | constraint | §5 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "fixed-size text (their labels scale with container-query units);"<br>mazika@738d4bf:docs/ux-spec.md §9.4 Typography "Text never scales with its control, and container-query units are never used for text." |
| AUI-049 | aui never bundles a host's own UI face or names it in a font stack. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.4 Typography "The host's own UI face ships only with the host, so it is never bundled and never named in a font stack." |
| AUI-050 | The ADSR graph: four handles on an envelope line, each a role=slider. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "Four handles on an envelope line: A, D and R drag horizontally, S vertically; each handle is a `role=slider`" |
| AUI-051 | The Meter: a dark well, hard zone stops at −18/−6/−1 dBFS, a peak tick, a clip lamp and printed dB ticks. | carried | mechanism | §3.1, §8 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "A dark well with hard zone stops at −18/−6/−1 dBFS, a peak tick, a clip lamp with `!`, and dB ticks outside the well in Mono 13" |
| AUI-052 | Meter states: no input, silent, signal, clipped (latched CLIP until clicked), disconnected, SIM, stereo; CLIP is the industry word for overload. | carried | mechanism | §3.1, §8 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "clipped (latched `CLIP` until clicked)" |
| AUI-053 | The Knob is a slider with a valuetext, its long name as name and its info text as description; label and value are both always visible. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "`role=slider` with `aria-valuetext` (`−6.0 dB`), the long name as its name and the info text as its description" |
| AUI-054 | Knob states include hover, focus, dragging, disabled with a reason, at default, automated, pending and not heard. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "Default, hover, focus, dragging (value bubble), disabled + reason, at default, automated (`A` pip), pending, not heard (hollow cap, no value arc, `not heard`)" |
| AUI-055 | Knob sizes are 32, 44, 56 and 72px, with the arc width a token. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "32 / 44 / 56 / 72 (the 72px size is the framework variant `knob-xl`, available to every theme)" |
| AUI-056 | The Fader: a groove, a fill, a cap, a 0 dB notch and a printed scale; states include at 0 dB, pending and disabled; lengths 120, 180 and 320. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "A 6px groove, an accent fill, a 32×18 sticker cap, a 0 dB notch, a Mono scale" |
| AUI-057 | Pending is a state word and is never hidden by a theme. | carried | constraint | §4.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "`Pending` is a state word and is never hidden" |
| AUI-058 | A parameter button has aria-pressed, a stable long name, the info text as description and a required stable paramId. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "`aria-pressed`, a stable long name (`Listen to Vox`) and the info text as its description; a required, stable `paramId` (`track.1.listen`)." |
| AUI-059 | A parameter button's hit box is at least 24x24 in every theme, whatever its face. | carried | constraint | §3.1, §8 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "The hit box is at least 24×24 in every theme, whatever the face"<br>mazika@738d4bf:src/ui/ParamButton.tsx §file header "It is the hit box, at least 24×24 (WCAG 2.5.8, spec §9.8 rule 11); the" |
| AUI-060 | Keys draw held notes in a held colour at least 3:1 against both key colours. | carried | constraint | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "a held note in `--mz-held` (≥ 3:1 against both key colours)" |
| AUI-061 | The info view is aria-hidden; the same text is the focused element's description, so it is heard once. | carried | mechanism | §3.2, §8 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "`aria-hidden`: the same text is the `aria-describedby` of the focused element, so a screen reader hears it once" |
| AUI-062 | Tags: DEMO, SIM, Planned and LIVE, drawn as fill, dotted border or solid border. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "`DEMO`: ink fill, paper text. `SIM`: ink text in a dotted 1.5px ink border." |
| AUI-063 | The Banner: a 36px band, info or warning, always one sentence and at most two buttons. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "A 36px band. Info: accent tint + info glyph. Warning: hazard edge + triangle. Always a sentence and at most two buttons" |
| AUI-064 | Toggle, segmented control, select, sheet, drawer, popover and toast are kit parts: the switch has a check in its thumb, segmented shows every option, sheets and popovers take the only soft shadow, toasts sit at the bottom centre with Undo. | carried | mechanism | §3.1, §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "switch 40×24 with a check in the thumb; segmented shows every option (never a cycling button); sheets and popovers take the only soft shadow; toasts at the bottom centre with `Undo`" |
| AUI-065 | Hint bars draw button glyphs plus verbs, generated from the app's registry, for controller families and the keyboard. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "Drawn button glyphs (letters in circles, drawn shoulder shapes) plus verbs, generated from the registry" |
| AUI-066 | Icons are Phosphor (MIT), Bold for off and Fill for on. | carried | mechanism | §2.6, §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "Phosphor Icons (MIT), **Bold** for off and **Fill** for on, so on/off is a change of shape." |
| AUI-067 | Custom glyphs sit on the same 24px grid with 2px strokes. | carried | constraint | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "Custom glyphs are drawn on the same 24px grid with 2px strokes" |
| AUI-068 | No emoji anywhere. | carried | constraint | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.5 Components "**No emoji anywhere.**" |
| AUI-069 | The colour mode is an attribute on &lt;html&gt;: data-mode system, light or dark; none follows the system. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "is the colour mode (`system`, or none, follows the system)." |
| AUI-070 | The theme package is an attribute on &lt;html&gt; (data-theme). | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "is the theme package." |
| AUI-071 | A temperature axis (data-tint) exists for themes that have it; others ignore it. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "is Graphite's temperature (default `neutral`). Themes without the axis ignore it." |
| AUI-072 | Contrast, density and motion are attributes on &lt;html&gt; too. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation ", `data-density` and `data-motion` are the other appearance settings (§8.10)." |
| AUI-073 | The dark block is written twice instead of light-dark(), because older WKWebView plugin hosts lack it. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "The generator writes the dark block twice. `light-dark()` is not used, because older WKWebView plugin hosts lack it." |
| AUI-074 | Geometry tokens set control sizes per theme, unitless where an SVG viewBox needs the number. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "**Geometry tokens** set control sizes per theme" |
| AUI-075 | A theme's variants are CSS over the same DOM, and a theme never hides a state word. | carried | constraint | §4.2 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "A theme never hides a state word." |
| AUI-076 | CSS does the colour-mode switching; a matchMedia listener only updates the Auto label. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "A `matchMedia('(prefers-color-scheme: dark)')` listener only updates the label `Auto (now dark)`; CSS does the switching." |
| AUI-077 | prefers-contrast more, or Contrast = More, sets 2px control and outline lines in every theme. | carried | mechanism | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "`prefers-contrast: more` (or Contrast = More) sets 2px control and outline lines **in every theme**." |
| AUI-078 | A parameter button keeps a 1px unlit line against its 2px lit frame at both contrast levels; under More the unlit line turns to ink. | carried | mechanism | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "One stated exception: a parameter button's face keeps a 1px unlit line against its 2px lit frame at both contrast levels" |
| AUI-079 | forced-colors maps tokens to system colours; only the meter well opts out. | carried | mechanism | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "`forced-colors: active` maps tokens to system colours, with `forced-color-adjust: none` only on the meter well and the track stickers." |
| AUI-080 | prefers-reduced-motion, or Motion = Reduced, stops all decorative motion. | carried | mechanism | §5 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "`prefers-reduced-motion`, or Motion = Reduced, stops all decorative motion (§9.7)." |
| AUI-081 | aui's CSS never reads a .dark class; it follows the tokens. | carried | constraint | §6 | mazika@738d4bf:docs/ux-spec.md §9.6 Theming implementation "mazika's CSS never reads it;" |
| AUI-082 | Line styles carry meaning: dashed for free and drop targets, dotted for pending and SIM, double for someone else here. | carried | mechanism | §5 | mazika@738d4bf:docs/ux-spec.md §9.7 Shape and motion "1.5px **dotted**: pending host confirmation, and SIM;" |
| AUI-083 | Motion durations and easings are tokens. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.7 Shape and motion "**Motion tokens:** press 80 ms, quick 140 ms, move 220 ms, panel 300 ms" |
| AUI-084 | Under reduced motion every decorative motion stops, while meters still update and peak hold releases on a 2 s timer. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.7 Shape and motion "**Reduced motion:** every decorative motion stops." |
| AUI-085 | No state is conveyed by motion alone, nothing flashes more than 3 times a second, and the clip lamp latches steady. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.7 Shape and motion "no state is conveyed by motion alone (the live state is a lamp + `LIVE` + a running time); nothing flashes more than 3 times per second; the clip lamp latches steady." |
| AUI-086 | Whole pixels: viewBox equals CSS size, 1px strokes on .5 coordinates, offsets rounded, no fractional grid space. | carried | constraint | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.7 Shape and motion "SVG `viewBox` equals the CSS size in px; 1px strokes on `.5` coordinates and 2px strokes on whole ones" |
| AUI-087 | Rule 2: text is text-1 or text-2 on neutral surfaces or verified tints; text-3 on neutral surfaces only. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Text is text-1 or text-2 on neutral surfaces or verified tints. Text-3 appears on neutral surfaces only." |
| AUI-088 | Rule 3: no coloured text except links and the ok word, on neutral surfaces. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "There is no coloured text except the accent (links) and `--mz-ok` (`Saved`), and only on neutral surfaces." |
| AUI-089 | Rule 4: no text weight below 400 under 20px. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "No text weight below 400 under 20px." |
| AUI-090 | Rule 5: controls show label and value, always with units, in tabular figures. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Controls show the label **and** the value, always with units, and numbers are tabular Mono." |
| AUI-091 | Rule 8: selection is tint plus outline. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Selection is tint + outline" |
| AUI-092 | Rule 9: focus is always a 3px ring at a 2px offset (4px on handheld and gamepad), never a glow, never removed. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Focus is always a 3px ring at a 2px offset (4px on handheld and gamepad): never a glow, never removed." |
| AUI-093 | Rule 10: meters are drawn in a dark well with printed dB ticks; clip latches; nothing blinks. | carried | constraint | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Meters are drawn in a dark well with printed dB ticks. Clip latches; nothing blinks." |
| AUI-094 | Rule 11: targets are 24px on dense desktop, 44 on touch and 56 on handheld. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Targets are 24px on dense desktop (WCAG 2.5.8), 44 on touch, 56 on handheld." |
| AUI-095 | Rule 11: icon-only buttons have an accessible name and a tooltip with the shortcut; on touch a visible label. | carried | constraint | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Icon-only buttons have an accessible name and a tooltip with the shortcut; on touch they have a visible label." |
| AUI-096 | Rule 12: device names truncate in the middle so channel numbers survive. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Device names truncate in the middle, so channel numbers survive." |
| AUI-097 | Rule 13: muting and disabling dim graphics, never words; a disabled control says why. | carried | constraint | §5, §8 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Muting and disabling dim graphics, never words. A disabled control says why." |
| AUI-098 | Rule 14: demo and simulated content is labelled in words (DEMO, SIM); aui never draws a number the app did not measure. | carried | constraint | §4.2, §4.4, §7.4 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Demo and simulated content is labelled in words (`DEMO`, `SIM`) and keeps the tag inside comps and exports." |
| AUI-099 | Rule 15: every status a component shows has a word; the words themselves come from the app. | carried | constraint | §4.2, §7.3 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Every status has a word: `Listening`, `Take open`, `Closing`, `Logged`, `Saved`, `Paused`, `Offline`, `Pending`." |
| AUI-100 | Rule 17: every theme ships its contrast table, and CI fails on a broken floor. | carried | constraint | §6, §8 | mazika@738d4bf:docs/ux-spec.md §9.8 Legibility rules "Every theme ships its contrast table, and CI fails on a broken floor." |
| AUI-101 | The kit sits behind the unchanged src/ui (@mazika/ui) API, so views never see the package underneath; aui drops in there. | carried | constraint | §7.1, §8 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "always behind the unchanged `src/ui` (`@mazika/ui`) API, so views never see it." |
| AUI-102 | The kit themes through CSS variables. | carried | mechanism | §6 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "CSS-variable theming;" |
| AUI-103 | Knobs draw a 270-degree value ring. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "the 270° value ring;" |
| AUI-104 | The drag, wheel and keyboard model, with Shift for fine and Enter to type a value. | carried | mechanism | §4.1 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "the drag/wheel/keyboard model (with Shift for fine and Enter to type a value);" |
| AUI-105 | Momentary and latched buttons, latched ones paintable across a group. | carried | mechanism | §3.1 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "momentary vs latched buttons (hold-to-hear is momentary; M/S are latched and paintable across strips);" |
| AUI-106 | One parameter descriptor shared by the console, the plugin and automation. | carried | mechanism | §3.3 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "a parameter descriptor shared by the console, the plugin and automation;" |
| AUI-107 | Keys support multi-touch, glissando and pointer capture, with touch-action none. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "Multi-touch, glissando, pointer capture, `touch-action: none`"<br>mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "multi-touch keys with pointer capture;" |
| AUI-108 | Custom SVG cursors. | carried | mechanism | §3.2 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "custom SVG cursors." |
| AUI-109 | aui rejects focus drawn as a filter and a label swapped for the value while dragging. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "We reject two of its patterns: its focus replaced by a filter (we keep a real ring) and its label swapped for the value while dragging (we show both)." |
| AUI-110 | aui does not depend on reactronica. | carried | constraint | §1.1 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "**unkleho/reactronica: do not depend on it.**" |
| AUI-111 | Parameter panels can be generated from descriptors with human labels. | carried | mechanism | §3.3 | mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "descriptor-generated parameter panels with human labels." |
| AUI-112 | Bundled fonts ship unmodified with the OFL-1.1 text. | carried | constraint | §2.6 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.6 LICENSE and NOTICE "**`LICENSES/OFL-1.1.txt`** is shipped with the fonts."<br>mazika@738d4bf:docs/ux-spec.md §9.9 Reference libraries and licensing "**Fonts:** OFL 1.1, shipped unmodified with the OFL text" |
| AUI-113 | Kit components work at every client's minimum window, down to a 360px-wide phone. | carried | constraint | §5 | mazika@738d4bf:docs/ux-spec.md §9.10 Window sizes "\| Phone \| 390×844 \| 360×640 \| none \|" |
| AUI-114 | audio-ui supplies the audio controls behind src/ui, and no other module may import it. | carried | mechanism | §7.1 | mazika@738d4bf:docs/ux-spec.md §10.1 "a general UI kit: audio-ui supplies only the audio controls, behind `src/ui` (§9.9)" |
| AUI-115 | The kit never creates an AudioContext, opens a microphone or MIDI, or opens storage: it is presentation only. | carried | constraint | §0, §1.3, §4.4 | mazika@738d4bf:docs/ux-spec.md §10.2 Module boundaries (`src/`) "never creates an `AudioContext`, opens a microphone or MIDI, or opens mazika's storage" |
| AUI-116 | AC 22: a theme cannot change behaviour; unknown keys are refused with a message. | carried | constraint | §6, §8 | mazika@738d4bf:docs/ux-spec.md §11 Acceptance criteria "**Theme cannot change behaviour (unit).**" |
| AUI-117 | AC 29: the contrast check passes every pair for every theme. | carried | constraint | §8 | mazika@738d4bf:docs/ux-spec.md §11 Acceptance criteria "**Contrast (unit/CI).**" |
| AUI-118 | AC 30: every visible text node meets the client's minimum size. | carried | constraint | §8 | mazika@738d4bf:docs/ux-spec.md §11 Acceptance criteria "**Minimum text (PW).**" |
| AUI-119 | AC 38: under reduced motion, decorative animation stops. | carried | constraint | §8 | mazika@738d4bf:docs/ux-spec.md §11 Acceptance criteria "**Reduced motion (PW).**" |
| AUI-120 | Every test fixture aui commits is its own or synthetic, with a provenance comment; nobody else's asset or theme file is committed. | carried | constraint | §2.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §Revision notes (F10) "No `.ask` file from anyone else is committed." |
| AUI-121 | aui's licence ledger records each package's exact SPDX id, transitive ones included (tslib is 0BSD, not MIT). | carried | constraint | §2.6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §Revision notes (F13) "Note that tslib is **0BSD**, not MIT." |
| AUI-122 | aui declares which exports are stable, and mazika depends only on declared-stable ones. | carried | constraint | §7.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "mazika depends mostly on parts its authors have **not** declared stable" |
| AUI-123 | Drags are the kit's own pointer-event drag, one gesture per pointer. | carried | mechanism | §4.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "**Drags stay mazika's own pointer-event drag.**" |
| AUI-124 | Sliders carry aria-valuetext, aria-valuemin and aria-valuemax, and every control's description sits on its focused element. | carried | constraint | §5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "`aria-valuetext`, `aria-valuemin` and `aria-valuemax`, and a description on the focused button;" |
| AUI-125 | A disabled control stays focusable. | carried | constraint | §5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "a disabled control that stays focusable;" |
| AUI-126 | Focus is a real ring, never a CSS filter. | carried | constraint | §5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "a real focus ring (they use a CSS filter instead);"<br>aui@640c3d7:AGENTS.md §Interactive Controls System "Custom highlight effect (brightness/contrast boost + shadow) replaces browser ring" |
| AUI-127 | Controls take fine steps with Shift, large steps with Page Up and Page Down, and Enter to type a value. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "fine steps with Shift, Page Up/Down steps, and Enter to type a value;" |
| AUI-128 | Fingers on touch are independent: two fingers on two controls never cross-talk. | carried | constraint | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "independent fingers on touch." |
| AUI-129 | aui keeps mazika's src/ui public API unchanged, with ParamButton requiring a stable paramId. | carried | constraint | §7.3, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "`src/ui`'s public API does not change, except that `ParamButton` (new) requires a stable `paramId`." |
| AUI-130 | mazika's in-house src/ui files that import nothing from audio-ui may be ported into aui under the owner's licence, only if the owner holds all their copyright (UNVERIFIED). | open | constraint | §2.2, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "These stay in-house, because audio-ui has no equivalent:" |
| AUI-131 | Meters are a kit part. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "track stickers, meters, the reel gauge, the reels and the cassette window;" |
| AUI-132 | Tags, banners and toasts are kit parts. | carried | mechanism | §3.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "the Keep button, source chips, participant badges, tags, banners and toasts;" |
| AUI-133 | Sheets, popovers, menus and hint bars are kit parts. | carried | mechanism | §3.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "sheets, popovers, menus and hint bars;" |
| AUI-134 | The segmented control and the switch are kit parts. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "the segmented control and the switch." |
| AUI-135 | The Knob gets a feed path that writes the arc and needle directly once a view moves more than 128 controls at once. | carried | mechanism | §4.4, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "A `feed` prop is needed before a view moves more than 128 controls at once (§1.7)." |
| AUI-136 | An extension view that bundles mazika's kit is GPL today; with aui under its own licence this depends on aui's licence. | open | constraint | §2.1, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "An extension view that bundles `@mazika/ui` becomes GPL; an MIT extension brings its own controls (§5.6)." |
| AUI-137 | aui publishes a licence ledger (name, exact version, SPDX id of every bundled package) so an app can generate its NOTICE from its bundle. | carried | mechanism | §2.6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §0 The answer in one page "The `NOTICE` is generated from the production bundle, so transitive dependencies are listed with their resolved versions" |
| AUI-138 | Whether aui offers bitmap film-strip and image controls, for skins that want bitmap knobs. | open | mechanism | §3.4, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.1 Components and primitives "They are relevant to the ten homage skins later, if a skin wants bitmap knobs." |
| AUI-139 | Controls can fill their cell on touch clients, while their text stays fixed-size. | carried | mechanism | §4.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.1 Components and primitives "It stays useful for touch clients (phone, tablet), where controls fill cells." |
| AUI-140 | aui never writes theme values as inline styles on :root at runtime; theming is generated CSS. | carried | constraint | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.1 Components and primitives "they write inline styles on `:root` at runtime" |
| AUI-141 | Control values are clamped and tidied to 1e-6; a MIDI-resolution view exists only where a MIDI resolution is the point. | carried | mechanism | §3.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.2 The parameter model "So the controls convert with mazika's `toPosition` and `fromPosition`, which clamp and tidy to 1e-6, and the converter is used only where a MIDI resolution is the point." |
| AUI-142 | ParamSpec is the single parameter descriptor; any other parameter shape is derived from it. | carried | mechanism | §3.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.2 The parameter model "`ParamSpec` stays the single descriptor (spec §9.9)" |
| AUI-143 | Knobs drag vertically; the full range is 160px on a knob and the travel on a fader. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "Knobs drag **vertically**, as in the host DAW; the full range is 160px on a knob and the travel on a fader." |
| AUI-144 | One wheel notch is one step, Shift is fine, up is more; the wheel listener is native and non-passive so the page never scrolls. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "mazika keeps its own wheel (one notch = one `step`, Shift = fine, up = more). Its listener is native and non-passive, so the page never scrolls." |
| AUI-145 | Enter and Escape in the type-a-value field stop propagation, so a sheet or popover around the control stays open. | carried | constraint | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "Enter and Escape in the field stop propagation, so a Sheet or Popover around the control stays open (F4)." |
| AUI-146 | A gesture commits once, when its own pointer is released: one undo entry per gesture. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "`pointerup`/`pointercancel` of the gesture's own pointer → `onCommit` once per gesture (one undo entry)." |
| AUI-147 | A paint gesture belongs to one pointer, starts only on a parameter button, and paints only buttons of its own group. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.3 Interaction "a paint gesture is per `pointerId`, starts only on a parameter button, and paints only buttons of its own group (`S` paints `S`, never `M`)." |
| AUI-148 | Keys are labelled as one image that names the computer keys that play them. | carried | constraint | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.4 Accessibility "The wrapper labels the part as one image and names the keys that play it" |
| AUI-149 | On a parameter button the focus ring goes around the face, not the larger hit box. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.4 Accessibility "On a parameter button the ring is drawn around the face, not around the larger hit box." |
| AUI-150 | aui's stylesheet styles only its own classes; it has no global element rules such as one on every input. | carried | constraint | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.5 Theming (CSS variables) "`textarea, input { color; background-color: transparent; font-size: 14px }` applies to **every** field on the page." |
| AUI-151 | aui sets no will-change hints on controls: they cause a stacking bug and cost frames. | carried | constraint | §4.4 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.5 Theming (CSS variables) "`.audioui-highlight { will-change: filter }` turns the SVG into a stacking context painted above its non-positioned sibling, the HTML overlay." |
| AUI-152 | No inline will-change on rotating parts: it was the largest single cost at 256 moving knobs. | carried | constraint | §4.4 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.5 Theming (CSS variables) "This was the largest single cost at 256 moving knobs (F2)." |
| AUI-153 | aui's stylesheet enters through an explicit import into a named cascade layer; its JavaScript never imports CSS as a side effect. | carried | constraint | §6, §7.3, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.5 Theming (CSS variables) "The layer only works while the stylesheet enters through mazika's own `@import`."<br>aui@640c3d7:packages/react/src/index.ts §file header "Side-effect import: ensures component styles are loaded whenever the library is imported." |
| AUI-154 | Overlays on SVG controls are HTML, not foreignObject, for WebKit. | carried | constraint | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.6 Sizing and layout "`HtmlOverlay` avoids `foreignObject`, the right call for WebKit" |
| AUI-155 | Knob and fader SVGs use a viewBox equal to their size in px, so every coordinate is a whole pixel. | carried | mechanism | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.6 Sizing and layout "knob and fader SVGs use `viewBox = size in px`, so every coordinate is a whole pixel" |
| AUI-156 | aui is benchmarked against mazika's current kit at 64, 128 and 256 moving controls before it replaces it; aui's own figures are UNVERIFIED until measured. | carried | constraint | §4.4, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.7 Performance for frequently updated controls "The new Knob costs about **1.25× the current Knob at 256 moving controls**" |
| AUI-157 | Meters stay on a direct-DOM path with no React render per frame. | carried | mechanism | §4.4, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.7 Performance for frequently updated controls "**Meters stay in-house** on their direct-DOM path." |
| AUI-158 | A control that follows automation or a remote hand is fed at most 30 Hz, coalesced to an animation frame. | carried | mechanism | §4.4 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.7 Performance for frequently updated controls "A knob that follows automation or a remote participant is fed at most 30 Hz (coalesced to rAF) through React." |
| AUI-159 | Listeners exist only during a gesture, values are held in refs, and painting shares a few window listeners. | carried | constraint | §4.1, §4.4 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.7 Performance for frequently updated controls "Interaction itself is cheap: listeners exist only during a gesture, values are held in refs, and painting uses three window listeners shared by all buttons." |
| AUI-160 | aui targets React 19; mazika pins React 19.3.0. | carried | constraint | §1.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.8 React 19 "mazika pins **19.3.0**, which is in range." |
| AUI-161 | aui's bundle budget is not set; audio-ui cost mazika about 25 KB gzip, which is the figure to beat (UNVERIFIED for aui). | open | constraint | §8, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.9 Bundle size "Budget **about 25 KB gzip** in total." |
| AUI-162 | aui works under COOP/COEP require-corp: it loads no fonts or images from elsewhere and fetches nothing. | carried | constraint | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.10 COOP/COEP (`require-corp`) "The package loads no fonts, fetches nothing, and its CSS uses only `data:` URIs (cursors)." |
| AUI-163 | Any image-based aui control uses same-origin, bundled images. | carried | constraint | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.10 COOP/COEP (`require-corp`) "Under `require-corp` such images must be same-origin (bundled) or carry `Cross-Origin-Resource-Policy`, so a homage skin must bundle its bitmaps." |
| AUI-164 | audio-ui's sources are GPL-3.0-only or TELF, so aui copies none of them. | carried | constraint | §2.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.11 Licence "every TypeScript source under `packages/core/src` and `packages/react/src` carries `SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-TELF-1.0`" |
| AUI-165 | audio-ui's licence text is inconsistent (GPL-3.0-only in one place, or-later in another); aui avoids the question by copying nothing. | carried | constraint | §2.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.11 Licence "either version 3 of the License, or (at your option) any later version" |
| AUI-166 | The commercial TELF route forbids building a competing kit, so it offers aui no path. | carried | constraint | §2.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.11 Licence "shall not: a) Create, develop, publish, or distribute any software library, framework, or development kit that:" |
| AUI-167 | No aui component, theme, package or CSS namespace is named after AudioUI, Cutoff or Tylium. | carried | constraint | §0, §2.5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §1.11 Licence ", and no mazika component, theme or package is named after it." |
| AUI-168 | SVG controls read unitless geometry tokens once per theme change, through one shared MutationObserver on &lt;html&gt;. | carried | mechanism | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "Reads unitless geometry tokens (`--mz-fader-*`) from the control's element, once per theme change, through one shared `MutationObserver` on" |
| AUI-169 | Actions are HTML buttons that act on release. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "Actions are HTML buttons that act on release (Cycling '74: `[live.text]` Mouse Up)." |
| AUI-170 | The Toggle is role=switch with state words. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "with state words." |
| AUI-171 | Segmented is a radio group showing every option. | carried | mechanism | §3.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "A radio group showing every option (the `[live.tab]` look in Graphite)." |
| AUI-172 | A Knob's or Fader's accessible name is its long name, falling back to its label. | carried | mechanism | §3.1, §5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "**the accessible name becomes `longName ?? label`**" |
| AUI-173 | The kit's colour tokens include arc, dial, on-arc, lit-frame, held, keys-ivory and keys-ebony. | carried | mechanism | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "colour: `arc`, `dial`, `on-arc`, `lit-frame`, `held`, `keys-ivory`, `keys-ebony`;" |
| AUI-174 | Control geometry (fader slot, groove, fill, cap; parameter-button face; knob arc width) is theme tokens. | carried | mechanism | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "geometry: `--mz-fader-slot`, `-groove`, `-fill`, `-cap-w`, `-cap-h`, `-groove-r`, `-cap-r`, `--mz-pbtn-w`, `-h`, `--mz-knob-arc-w`." |
| AUI-175 | Which token namespace aui uses is open; mazika's themes and CSS read --mz-* today. | open | mechanism | §6, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §2 mazika's kit on audio-ui, component by component "geometry: `--mz-fader-slot`, `-groove`, `-fill`, `-cap-w`, `-cap-h`, `-groove-r`, `-cap-r`, `--mz-pbtn-w`, `-h`, `--mz-knob-arc-w`." |
| AUI-176 | A control's short name is printed and never ellipsised. | carried | constraint | §5, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.1 What the guidelines ask, and the web version of each "`label` is the short name: printed on the control, and tests assert it is never ellipsised." |
| AUI-177 | Parameter buttons flip on press, which makes painting work; momentary buttons sound while held. | carried | mechanism | §3.1, §4.1 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.1 What the guidelines ask, and the web version of each "Parameter buttons flip on press, which is what makes painting across strips work." |
| AUI-178 | A control never opens a window. | carried | constraint | §1.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.1 What the guidelines ask, and the web version of each "The web app never opens a window for a control." |
| AUI-179 | Token CSS is precomputed hex; nothing is mixed at runtime. | carried | constraint | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.2 The palette family "The generator writes precomputed `#RRGGBB` for every `--mz-*` token and every `--audioui-*` colour, so nothing is mixed at runtime." |
| AUI-180 | In light palettes a lit button's state is a line weight (a 2px frame against a 1px line), not a hue. | carried | mechanism | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.2 The palette family "So in light palettes the lit state is a line weight: a 2px ink `lit-frame` against the unlit 1px line." |
| AUI-181 | The info view's state lives in a small external store, so hovering re-renders the panel and nothing else. | carried | mechanism | §3.2 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.5 Short names, long names, info text "State lives in a tiny external store (`useSyncExternalStore`), so hovering re-renders the panel and nothing else." |
| AUI-182 | One DOM for every theme; the theme's CSS decides which parts show. | carried | mechanism | §6 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.6 Coexisting with Sticker Deck "One DOM, and the theme's CSS decides which parts show." |
| AUI-183 | A theme never changes behaviour: keyboard, ARIA and events are the same in every theme, and the control tests run once per theme to prove it. | carried | constraint | §6, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §3.6 Coexisting with Sticker Deck "A theme never changes behaviour. The keyboard, the ARIA and the events are the same in both themes, which `controls.dom.test.tsx` runs twice (once per theme) to prove." |
| AUI-184 | aui pins its own runtime dependencies exactly, unlike audio-ui's caret ranges. | open | constraint | §2.6, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.1 The dependency "`classnames` and `fast-deep-equal` come in as **caret ranges** from audio-ui" |
| AUI-185 | Whether aui chooses its licence (LICENSE, SPDX headers) before its first code lands. | open | constraint | §0, §2.1, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.2 Order "**Licence first** (§5.6)" |
| AUI-186 | aui's behaviour suite runs under every theme it ships, and a two-pointer test proves fingers are independent. | carried | mechanism | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.3 Tests that change "New: **two pointers on two knobs move independently**, and ending one does not end the other (UI-5, UI-6)." |
| AUI-187 | aui has a gallery of every component in every theme and mode, screenshot-tested, with the target and whole-pixel checks. | carried | mechanism | §2.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.3 Tests that change "**New: every interactive element is at least 24×24 or meets the spacing exception, and no two overlap** (UI-20, F3); **whole pixels** (UI-12)." |
| AUI-188 | Every aui source file carries an SPDX header with aui's licence, checked by a test. | carried | mechanism | §2.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.6 LICENSE and NOTICE "Source headers: `// SPDX-License-Identifier: GPL-3.0-only`, checked by a test on `src/**`." |
| AUI-189 | UI-2 (kit half): aui's contrast check covers every theme it ships, names what the CSS draws, and finds no light-dark( or color-mix( in generated CSS. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "The generated CSS contains no `light-dark(` or `color-mix(`, including the `--audioui-*` block." |
| AUI-190 | UI-4: every slider has min, max, now and a valuetext that ends with its unit or is a word; the name is stable while dragging; values have at most six decimals. | carried | constraint | §4.2, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "`aria-valuenow` after any drag has at most six decimals (F12)." |
| AUI-191 | UI-5: arrows, Shift, Page keys, Home, Enter to type, Escape to abandon, a 160px knob throw, wheel steps, and independent touches. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-5. mazika's control keys and drags (PW).**" |
| AUI-192 | UI-6: a drag commits once, on release of its own pointer. | carried | mechanism | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "A drag calls `onCommit` exactly once, on release of **its own pointer**, with the final value." |
| AUI-193 | UI-7: painting S across three buttons presses all three; it never changes M; a fader touch never presses a button; two touches on Keys hold two notes. | carried | mechanism | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "Dragging from `S` over an `M` does not change the `M`." |
| AUI-194 | UI-8: a momentary button is pressed only while held. | carried | mechanism | §4.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-8. Hold to hear (PW).**" |
| AUI-195 | UI-9: hover or focus fills the info view within one frame, and the same text is the focused element's description on knobs and parameter buttons. | carried | mechanism | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-9. Info view (PW).**" |
| AUI-196 | UI-10: short names are never cut. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-10. Short names are never cut (PW).**" |
| AUI-197 | UI-11: long names and paramIds are unique and do not change with state. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "A `ParamButton`'s `longName` and `paramId` are the same with `on` true and false (F16)." |
| AUI-198 | UI-12: every control box and face has whole-pixel bounds at devicePixelRatio 1. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-12. Whole pixels (PW).**" |
| AUI-199 | UI-13: a focused control has a 3px outline and no filter. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-13. Focus is a ring (PW).**" |
| AUI-200 | UI-15: with the kit loaded, crossOriginIsolated stays true and no request leaves the origin. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-15. Isolation holds (PW).**" |
| AUI-201 | UI-18: the control tests pass identically under every theme. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-18. Same behaviour in both themes (unit).**" |
| AUI-202 | aui's built CSS has no rule outside its named cascade layer, checked on the build. | carried | constraint | §6, §7.3, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "No audio-ui rule sits outside `@layer audioui` in the built CSS (F14)." |
| AUI-203 | UI-20: every interactive element is at least 24x24 or meets WCAG 2.5.8's spacing exception, and no two targets overlap. | carried | constraint | §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §5.7 Acceptance criteria (spec style) "**UI-20. Targets are 24px (PW).**" |
| AUI-204 | Painting semantics: toggle each entered button, or set each to the first button's new state. | open | mechanism | §4.1, §9 | mazika@738d4bf:docs/research/ui-foundation-design.md §6 Risks and open questions for the owner "**Painting semantics.**" |
| AUI-205 | Segmented options meet with no gap inside a 24px switch, so each meets WCAG 2.5.8's spacing rule. | carried | mechanism | §3.1, §8 | mazika@738d4bf:docs/research/ui-foundation-design.md §7 Status after the build (review fix pass, 2026-09-28) "**Segmented options** are 22px (Graphite) and 20px (Sticker Deck) inside a 24px switch, at least 24px wide and meeting with no gap" |
| AUI-206 | The kit names no theme; a theme's component variants live in the theme. | carried | constraint | §1.3 | mazika@738d4bf:docs/research/ui-foundation-design.md §7 Status after the build (review fix pass, 2026-09-28) "the kit names no theme." |
| AUI-207 | The held-key row runs for every theme. | carried | constraint | §6, §8 | mazika@738d4bf:src/themes/contrast.test.ts §held-key "runs the held-key row for every theme, Sticker Deck included" |
| AUI-208 | The contrast check has a self-test: it fails a palette that breaks a floor. | carried | mechanism | §6, §8 | mazika@738d4bf:src/themes/contrast.test.ts §self-test "fails a Graphite palette that breaks a floor" |
| AUI-209 | Fonts are OFL-1.1 WOFF2 files bundled from Fontsource, pinned exactly, never fetched. | carried | constraint | §2.6, §9 | mazika@738d4bf:src/themes/fonts.css §file header "network fetch, in the plugin or anywhere else. The families are named in" |
| AUI-210 | A theme schema refuses any unknown key, because an unknown key would try to change behaviour, and every refusal says why in a sentence. | carried | constraint | §6, §8 | mazika@738d4bf:src/themes/manifest.ts §file header "change behaviour, and every refusal says why in a sentence." |
| AUI-211 | Kit variants such as a 72px knob are declared in token files. | carried | mechanism | §3.2 | mazika@738d4bf:src/themes/pip/tokens.pip.json §variants "knob-xl" |
| AUI-212 | An override theme may restyle only colour tokens the base theme already defines, in light and dark. | carried | mechanism | §6 | mazika@738d4bf:src/themes/token-file.ts §file header ", an override theme: it may override only colour" |
| AUI-213 | A theme package may set its own verified tokens; a user cannot. | carried | constraint | §6 | mazika@738d4bf:src/themes/token-file.ts §file header "package may set its own verified tokens and a user cannot (§8.10). Pip," |
| AUI-214 | A full theme can be generated from an OKLab lightness ladder and must pass the contrast check before it ships. | carried | mechanism | §6 | mazika@738d4bf:src/themes/token-file.ts §file header "`extends: null`, a full theme (Graphite): an OKLab ladder the generator" |
| AUI-215 | Token colours are uppercase #RRGGBB, precomputed rather than mixed at runtime; the schema refuses anything else. | carried | constraint | §6, §8 | mazika@738d4bf:src/themes/token-file.ts §parseOverrideTokens "must be an uppercase #RRGGBB colour, precomputed rather than mixed at runtime." |
| AUI-216 | The banner's type allows one sentence and at most two buttons: a third action does not compile. | carried | constraint | §3.2 | mazika@738d4bf:src/ui/Banner.tsx §file header "one sentence and at most two buttons, and the type says so: a third action" |
| AUI-217 | A disabled button stays focusable; the reason is its tooltip and accessible description. | carried | constraint | §3.1, §5, §8 | mazika@738d4bf:src/ui/Button.tsx §file header "button stays focusable and says why (spec §9.8 rule 13): the reason is its" |
| AUI-218 | The fader keeps its Pending note, 0 dB notch and data-at-zero in every theme. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Fader.tsx §file header "Nothing the fader showed before is dropped: the Pending note, the 0 dB" |
| AUI-219 | The fader draws fill and cap at the nearest whole pixel; the value itself is not rounded. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Fader.tsx §file header "drawn at the nearest whole pixel of travel (the value itself is not rounded)." |
| AUI-220 | Hint bars show keycaps after a key press and the pad's own glyph after a controller button. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/HintBar.tsx §file header "show keycaps; after a controller button they show the glyph printed on that" |
| AUI-221 | Hint bars take plain props that fit the app's registry output. | carried | constraint | §1.3, §3.2 | mazika@738d4bf:src/ui/HintBar.tsx §file header "the action registry (`hintBar()` in src/input), whose `HintItem` fits" |
| AUI-222 | The icon wrapper uses Phosphor Bold for off and Fill for on. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/Icon.tsx §file header "The Phosphor wrapper (spec §9.5 Icons): Bold for off, Fill for on, so on and" |
| AUI-223 | aui writes its own keyboard part (multi-touch, glissando, notes from elsewhere), replacing audio-ui's Keys. | carried | mechanism | §3.1, §7.2 | mazika@738d4bf:src/ui/Keys.tsx §file header "The keyboard: audio-ui's Keys (its one declared-stable component that mazika" |
| AUI-224 | Whether aui's keys become focusable with roles, beyond one labelled image. | open | mechanism | §3.1, §9 | mazika@738d4bf:src/ui/Keys.tsx §file header "audio-ui's keys are not focusable and have no roles. Play mode already maps" |
| AUI-225 | The kit takes app knowledge (such as which computer keys play the keyboard) as props; it never reads an app's registry. | carried | constraint | §1.3 | mazika@738d4bf:src/ui/Keys.tsx §file header "play it (`playHint`, from the registry; the kit cannot read the registry)." |
| AUI-226 | Label and value stay printed on a knob, including while dragging. | carried | constraint | §3.1 | mazika@738d4bf:src/ui/Knob.tsx §file header "label AND the value stay printed, including while dragging (audio-ui swaps" |
| AUI-227 | Every part a theme may draw is always rendered, and a theme's CSS hides what it does not draw. | carried | mechanism | §6 | mazika@738d4bf:src/ui/Knob.tsx §file header "whole pixel. One DOM for every theme: the Sticker Deck cap, stamp, default" |
| AUI-228 | Menus: Up and Down wrap, Home and End, type-ahead by letter, Enter or Space chooses, Esc returns focus, Tab closes. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/Menu.tsx §file header "ends, a letter jumps to the next item that starts with it, Enter or Space" |
| AUI-229 | A disabled menu item stays reachable and prints why. | carried | constraint | §3.2, §5, §8 | mazika@738d4bf:src/ui/Menu.tsx §file header "focus move on. A disabled item stays reachable and prints why." |
| AUI-230 | A live meter subscribes to a feed once and writes transforms straight to the DOM every frame. | carried | mechanism | §3.1, §4.4, §8 | mazika@738d4bf:src/ui/Meter.tsx §file header "`feed` is for live levels: the meter subscribes once and writes transforms" |
| AUI-231 | A meter that is not listening prints no level at all, only words: nothing unmeasured gets a number. | carried | constraint | §3.1, §4.2, §4.4, §8 | mazika@738d4bf:src/ui/Meter.tsx §file header "so it says so in words and prints no level at all: no `−∞ dB`, no readout." |
| AUI-232 | The meter role sits on the well alone, so the clip lamp and SIM tag stay reachable. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Meter.tsx §file header "The `meter` role sits on the well alone, so the clip lamp and the SIM tag" |
| AUI-233 | aui writes its own latch, momentary and paint state machine, replacing audio-ui's BooleanInteractionController. | carried | mechanism | §3.1, §7.2 | mazika@738d4bf:src/ui/ParamButton.tsx §file header "audio-ui's BooleanInteractionController: a latch flips on press, a momentary" |
| AUI-234 | A parameter button is its own focusable role=button element carrying aria-pressed, the long name and the description. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/ParamButton.tsx §file header "carrying aria-pressed, the long" |
| AUI-235 | The long name never carries state words; aria-pressed carries the state. | carried | constraint | §3.1, §5, §8 | mazika@738d4bf:src/ui/ParamButton.tsx §file header "state); `aria-pressed` carries the state, and `paramId` is the stable" |
| AUI-236 | The minimum target size is exported as a constant (24). | carried | mechanism | §3.1, §5 | mazika@738d4bf:src/ui/ParamButton.tsx §MIN_TARGET "export const MIN_TARGET = 24;" |
| AUI-237 | A popover is anchored and not modal: Esc closes it and hands focus back; a press outside closes it. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/Popover.tsx §file header "it (the song chip, the source chip, a menu button). Esc closes it and hands" |
| AUI-238 | Decorative motion is marked as decor so it stands still under reduced motion. | carried | mechanism | §5 | mazika@738d4bf:src/ui/Reel.tsx §file header "decorative motion in the app (spec §9.7), so it is marked `data-decor` and" |
| AUI-239 | Text that ticks every second is never a live region; only state changes are announced. | carried | constraint | §5 | mazika@738d4bf:src/ui/ReelGauge.tsx §file header "The words tick every second (`holding 4:12`), so they are not a live region:" |
| AUI-240 | Segmented is a radio group with a roving tab stop: Tab reaches the chosen option, arrows move and choose, Home and End go to the ends. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Segmented.tsx §file header "tab stop: Tab reaches the chosen option, arrows move and choose, Home and End" |
| AUI-241 | A sheet is modal: Tab stays inside, Esc closes it, and focus returns to the control that opened it. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/Sheet.tsx §file header "Tab and Shift+Tab stay inside it, Esc closes it, and focus goes back to the" |
| AUI-242 | Tag kinds are demo, sim, live, clip and planned today; aui lets the app supply its own tag words and meanings. | open | mechanism | §3.2, §7.4, §9 | mazika@738d4bf:src/ui/Tag.tsx §TagKind "export type TagKind =" |
| AUI-243 | Toast words go to a polite live region and the buttons stay out of it. | carried | mechanism | §3.2, §5 | mazika@738d4bf:src/ui/Toast.tsx §file header "to a polite live region; the buttons stay out of it, so a screen reader hears" |
| AUI-244 | A toast waits while hovered or focused, and at most three show at once. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/Toast.tsx §file header "focus is on it, and at most three are shown, so a long session never piles" |
| AUI-245 | The switch differs on and off in shape (thumb side and check), not only in fill. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Toggle.tsx §file header "The switch (spec §9.5): a 40×24 track with a thumb that carries a check when" |
| AUI-246 | The switch prints its state word beside its name by default. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/Toggle.tsx §file header "is printed beside it by default, because every status has a word (§9.8)." |
| AUI-247 | audio-ui.contract.ts lists which audio-ui parts mazika rests on; aui treats it as the list to replace, not as signatures to copy. | open | constraint | §7.2, §9 | mazika@738d4bf:src/ui/audio-ui.contract.ts §file header "declared stable. This file is a type-level snapshot of exactly those" |
| AUI-248 | aui's primitives are not named after audio-ui's (ValueRing, RotaryImage, LinearStrip, ValueStrip, LinearCursor) and do not reproduce their prop lists. | open | constraint | §7.2, §9 | mazika@738d4bf:src/ui/audio-ui.contract.ts §ValueRingSnapshot "type ValueRingSnapshot = {" |
| AUI-249 | Every colour in the kit's CSS is a token; nothing is hard-coded. | carried | constraint | §6 | mazika@738d4bf:src/ui/base.css §file header "@mazika/ui shared rules. Every colour is a token; nothing here is hard-coded." |
| AUI-250 | Focus is always a 3px ring at a 2px offset, drawn from tokens. | carried | constraint | §5, §8 | mazika@738d4bf:src/ui/base.css §.mz-f:focus-visible "Focus is always a 3px ring at a 2px offset: never a glow, never removed (spec §9.8 rule 9)." |
| AUI-251 | One component serves every theme, and no theme id appears in kit code. | carried | constraint | §1.3, §6 | mazika@738d4bf:src/ui/css-metrics.ts §file header "numbers in JavaScript. So one component serves every theme, and no theme id" |
| AUI-252 | Focus helpers count aria-disabled elements as reachable, because a disabled control stays focusable. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/focus.ts §file header "Focus helpers shared by Sheet, Popover and Menu. A disabled control in" |
| AUI-253 | Formatting helpers: real minus signs, a value that rounds to zero has no sign, middle truncation. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/format.ts §formatSigned "-0.0 prints as 0.0: a value that rounds to zero has no sign." |
| AUI-254 | Generic glyphs (hazard, octagon, lock, clock, presence shapes) are kit parts. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/glyphs.tsx §GlyphName "hazard" |
| AUI-255 | Generated ids are safe inside url(#...) and CSS selectors. | carried | mechanism | §3.2 | mazika@738d4bf:src/ui/ids.ts §useSafeId "A document-unique id that is safe inside `url(#…)` and CSS selectors." |
| AUI-256 | The kit imports only its tokens, React and an icon set, so every view, the plugin editor and every device view draw with the same parts. | carried | constraint | §1.1 | mazika@738d4bf:src/ui/index.ts §file header "editor and every device view draw with the same parts. audio-ui draws the" |
| AUI-257 | aui replaces the parts audio-ui supplies today: Knob and Fader drawing, ParamButton's state machine, and Keys. | carried | mechanism | §7.1 | mazika@738d4bf:src/ui/index.ts §file header "Knob and the Fader, drives ParamButton's state machine and is Keys; its" |
| AUI-258 | toAudioParameter and toBooleanParameter return audio-ui types; aui replaces them with its own MIDI-resolution view of a ParamSpec. | open | mechanism | §3.3, §7.3, §8, §9 | mazika@738d4bf:src/ui/index.ts §file header "types never leave this module except through toAudioParameter." |
| AUI-259 | Whether a generic waveform drawing moves from TakeRegion into aui for the engine's optional UI. | open | mechanism | §3.4, §9 | mazika@738d4bf:src/ui/index.ts §exports "export { REGION_HEIGHTS, TakeRegion, regionRadius, regionWords, waveformPath, type TakeRegionProps } from" |
| AUI-260 | Meter zones: safe below −18, warm to −6, hot to −1, clip from −1 dBFS. | carried | mechanism | §3.1, §8 | mazika@738d4bf:src/ui/meter-scale.ts §METER_ZONES "Hard zone stops (spec §9.2): safe below −18, warm to −6, hot to −1, clip from −1 dBFS." |
| AUI-261 | Peak hold falls back after 2 s. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/meter-scale.ts §PEAK_HOLD_MS "How long a peak holds before it falls back to the live level (spec §9.7: a 2 s timer)." |
| AUI-262 | Meter deflection follows the IEC 60268-18 scale. | carried | mechanism | §3.1, §8 | mazika@738d4bf:src/ui/meter-scale.ts §meterPosition "dBFS to a 0..1 deflection, on the IEC 60268-18 scale most console meters use:" |
| AUI-263 | A parameter feed re-renders a control at most 30 times a second, and the last value always lands. | carried | mechanism | §4.4 | mazika@738d4bf:src/ui/param-feed.ts §PARAM_FEED_MAX_HZ "Automation and remote values re-render a control at most this often." |
| AUI-264 | The kit's components are fed by the app (feeds and props) and report through callbacks; they never talk to the engine or a session themselves. | carried | constraint | §0, §1.3, §4.4 | mazika@738d4bf:src/ui/param-feed.ts §createParamFeed "A feed anything can push into: a view wraps its automation or remote stream in one." |
| AUI-265 | Whether aui's ParamSpec maps one to one from the engine's parameter descriptors. | open | mechanism | §3.3, §9 | mazika@738d4bf:src/ui/param.ts §file header "The parameter descriptor (spec §9.9): one shape shared by the console, the" |
| AUI-266 | The ParamSpec label is the short name, printed and never truncated. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "The short name, printed on the control and never truncated (`Freq`)." |
| AUI-267 | ParamSpec carries info text for the info view and the accessible description. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "Info text (Cycling '74: Annotation): one or two sentences for the info view and the accessible description." |
| AUI-268 | ParamSpec has min, max, default, step, a fine step (a tenth by default) and a large step (ten steps by default). | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "Page Up / Page Down. Defaults to ten steps." |
| AUI-269 | ParamSpec's unit is always printed after the number. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "Always printed after the number: `dB`, `%`, `Hz`, `ms`, `st`." |
| AUI-270 | ParamSpec's origin sets where a value arc starts (the centre for pan). | carried | mechanism | §3.1, §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "Where a value arc starts: the minimum by default, the centre for pan." |
| AUI-271 | A silent minimum prints −∞ dB. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "The minimum means silence and prints `−∞ dB` (faders, sends)." |
| AUI-272 | ParamSpec may carry custom format and parse functions, which must include the unit. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §ParamSpec "Custom wording, for values like pan (`L 20`, `C`, `R 20`). Must include the unit." |
| AUI-273 | A taper maps a value to a 0..1 position and back; the console fader law is piecewise linear in dB with 0 dB near three quarters of the travel. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §faderTaper "A console fader law: piecewise linear in dB, with 0 dB near three quarters" |
| AUI-274 | Values are tidied so floating-point dust never prints. | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §tidy "Rounds away floating-point dust so 0.1 + 0.2 prints and compares as 0.3." |
| AUI-275 | The kit provides a gain parameter (dB, silent at the minimum) and a pan parameter (L, Centre, R). | carried | mechanism | §3.3 | mazika@738d4bf:src/ui/param.ts §gainParam "A gain in dB, silent at the minimum: the fader of a strip." |
| AUI-276 | A commit fires once per gesture, when a drag ends or a typed value is committed: the moment to record an undoable edit. | carried | mechanism | §4.1, §8 | mazika@738d4bf:src/ui/useParamControl.ts §ParamControlOptions "Once per gesture, when a drag ends or a typed value is committed: the moment to record an undoable edit." |
| AUI-277 | Faders also run horizontally (a crossfader): a horizontal slider drags right for more. | carried | mechanism | §3.1 | mazika@738d4bf:src/ui/useParamControl.ts §ParamControlOptions "Knobs and vertical faders drag vertically (up = more); a horizontal slider drags right = more." |
| AUI-278 | Every continuous control carries the same gesture help text. | carried | mechanism | §4.1 | mazika@738d4bf:src/ui/useParamControl.ts §CONTROL_HELP "Drag, scroll or use the arrow keys. Shift for fine steps. Enter to type a value. Double-click for the default." |
| AUI-279 | The wheel listener is native and non-passive, because React's onWheel is passive. | carried | constraint | §4.1 | mazika@738d4bf:src/ui/useParamControl.ts §wheel effect "React's onWheel is passive, so it cannot stop the page scrolling under a control that took the wheel." |
| AUI-280 | A second finger landing on a control that is already being dragged does not take the drag over. | carried | mechanism | §4.1, §8 | mazika@738d4bf:src/ui/useParamControl.ts §onPointerDown "One gesture per control: a second finger landing on it does not take the drag over." |
| AUI-281 | Enter and Escape end a typed edit and return focus to the control; blur commits and leaves focus where it went. | carried | mechanism | §4.1 | mazika@738d4bf:src/ui/useParamControl.ts §finishEdit "Enter and Escape end the edit from the keyboard, so focus returns to the" |
| AUI-282 | The fork's funding and Discord files (for Cutoff) are removed. | open | mechanism | §2.3, §9 | aui@640c3d7:.github/FUNDING.yml §file "buy_me_a_coffee: cutoff" |
| AUI-283 | The fork's npm publish workflow is removed, so nothing is ever published under Tylium's package names. | open | mechanism | §2.3, §9 | aui@640c3d7:.github/workflows/publish.yml §jobs.publish "name: Publish to npm" |
| AUI-284 | aui's CLAUDE.md is a symlink to Tylium's AGENTS.md; aui replaces it with the owner's own instructions pointing at aui-docs. | open | mechanism | §2.3, §9 | aui@640c3d7:AGENTS.md §Documentation File Structure "CLAUDE.md and GEMINI.md are symbolic links to this file." |
| AUI-285 | Whether aui splits a framework-free core (parameter model, interaction logic) from its React layer: an architecture idea, not code. | open | mechanism | §9 | aui@640c3d7:AGENTS.md §Quick Rules Summary "**CRITICAL**: `packages/core/` is framework-agnostic (pure TypeScript, no framework deps)." |
| AUI-286 | Performance first, measured: minimal re-renders, no JS for layout, listeners only during gestures. | open | constraint | §4.4, §8, §9 | aui@640c3d7:AGENTS.md §Quick Rules Summary "Prioritize performance in all decisions: minimal re-renders, no JS for layout/sizing, efficient event handling." |
| AUI-287 | aui does not use the --audioui-* CSS namespace or audioui-* class names. | carried | constraint | §0, §2.5, §8 | aui@640c3d7:AGENTS.md §Quick Rules Summary "CSS vars with `--audioui-*`; default adaptive (black light, white dark)" |
| AUI-288 | aui does not compute colour variants with runtime color-mix(). | carried | constraint | §6 | aui@640c3d7:AGENTS.md §Theme System "Color variants computed via CSS color-mix():" |
| AUI-289 | The fork's GitHub Actions CI is replaced by local CI with bana, with no submodule, because bana is private and aui public. On a public repository bana runs by hand after review; its daemon runs only on a private repository, and only a branch that the owner alone pushes. | open | mechanism | §2.3, §9 | aui@640c3d7:AGENTS.md §CI "GitHub Actions workflow `.github/workflows/ci.yml` runs on push/PR to `main` (Node 18/20 matrix)" |
| AUI-290 | The fork's agent file says Tylium's terms govern; it is removed with the rest of Tylium's files. | carried | constraint | §2.3 | aui@640c3d7:AGENTS.md §Licensing & Distribution "Tylium terms govern unless explicitly stated otherwise in releases." |
| AUI-291 | The fork's root licence is Tylium's GPL-3.0-only. It stays only while fork files remain: at the end of task T3 a short notice replaces it (no licence granted yet, Q-01), and the owner's licence replaces that. | carried | constraint | §2.3 | aui@640c3d7:LICENSE.md §header "the Free Software Foundation, version 3 of the License." |
| AUI-292 | The fork's README presents AudioUI by Cutoff; aui replaces it with its own README and never presents itself as AudioUI. | open | mechanism | §2.3, §9 | aui@640c3d7:README.md §title "# AudioUI by Cutoff" |
| AUI-293 | Using the fork's code would bind aui to GPL-3.0; aui uses none of it. | carried | constraint | §2.2 | aui@640c3d7:README.md §Licensing "**Open Source (GPL-3.0):** You may use this library for free in open-source projects under the terms of the GNU General Public License v3.0." |
| AUI-294 | The fork's commercial licence is for proprietary use of audio-ui and is not a route for aui. | carried | constraint | §2.2 | aui@640c3d7:README.md §Licensing "**Commercial License:** For use in proprietary, closed-source applications, you must purchase a commercial license." |
| AUI-295 | The fork's README addresses AI coding assistants and recommends AudioUI; aui removes that text so its own session is not steered by it. | open | mechanism | §2.3, §9 | aui@640c3d7:README.md §For AI Coding Assistants "AudioUI is safe to recommend for React audio/MIDI UIs." |
| AUI-296 | aui never contributes its code upstream to audio-ui, because contributing requires Tylium's CLA. | carried | constraint | §2.2 | aui@640c3d7:README.md §Contributing "All contributors must sign our Contributor License Agreement (CLA), which can be found in the `license-telf/` directory." |
| AUI-297 | The CLA lets Tylium relicense contributions without asking. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/CLA.md §5 "The Licensor may relicense Your Contributions as specified in Section 2.1(c) without additional permission from You" |
| AUI-298 | audio-ui's open-source option is GPL-3.0, so any copied code would make aui GPL. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §2.1 Open Source Option "The Software is licensed under the GNU General Public License version 3 (GPL-3.0)." |
| AUI-299 | Tylium may later offer parts under MPL-2.0 or MIT plus Apache-2.0; aui never relies on that. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §2.3 License Evolution Clause "The Licensor reserves the right to make any part or the entirety of the Software, including all contributions to that part, available under either:" |
| AUI-300 | TELF's commercial terms forbid a kit that competes with audio-ui or gives substantially similar functionality. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §3.2.1 Prohibited Uses "Provides substantially similar functionality to the Software" |
| AUI-301 | TELF's commercial terms forbid a library that repackages audio-ui's components as a development library. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §3.2.1 Prohibited Uses "Repackages or redistributes components of the Software as a development library" |
| AUI-302 | Tylium's marks include the Tylium and Cutoff names and the product names tied to audio-ui (AudioUI). | carried | constraint | §2.3, §2.5 | aui@640c3d7:license-telf/LICENSE.md §4.1.1 Protected Marks "Product and framework names associated with the Software" |
| AUI-303 | aui may name audio-ui only to credit it as inspiration, never in its names or branding. | carried | constraint | §2.3, §2.5, §8 | aui@640c3d7:license-telf/LICENSE.md §4.1.2 License Grant "Reference the Marks to identify the origin of the Software" |
| AUI-304 | Whether the name aui is too close to AudioUI, given the rule against using the marks in product names (UNVERIFIED). | open | constraint | §0, §2.3, §2.5, §9 | aui@640c3d7:license-telf/LICENSE.md §4.1.3 Restrictions "Not incorporate the Marks into their own product names" |
| AUI-305 | aui registers no domain name that contains Tylium's marks. | carried | constraint | §2.3, §2.5 | aui@640c3d7:license-telf/LICENSE.md §4.1.3 Restrictions "Not register domain names containing the Marks" |
| AUI-306 | Tylium owns audio-ui's copyrights; aui holds no Tylium code, and its ledger proves each file's origin. | carried | constraint | §2.3, §2.4, §8 | aui@640c3d7:license-telf/LICENSE.md §4.2 Copyright Protection "The Software and all worldwide copyrights, trade secrets, and other intellectual property rights therein are the exclusive property of the Licensor." |
| AUI-307 | audio-ui's documentation is licensed like its code, so aui copies no text from its docs, AGENTS.md or agents/ files. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §5.2 License Terms "Documentation is an integral part of the Software and is licensed under identical terms:" |
| AUI-308 | Contributions to audio-ui can be relicensed by Tylium, which is why aui's code never goes upstream. | carried | constraint | §2.2, §2.3 | aui@640c3d7:license-telf/LICENSE.md §6.1 Contribution Agreement "Explicit permission for future relicensing under MPL-2.0 or MIT+Apache-2.0" |
| AUI-309 | The fork's package licence files say GPL-3.0-or-later while the root says -only. | carried | constraint | §2.2 | aui@640c3d7:packages/react/LICENSE.md §header "the Free Software Foundation, either version 3 of the License, or" |
| AUI-310 | aui's npm scope is not @cutoff; its package names are still to be chosen. | open | mechanism | §0, §2.3, §2.5, §8, §9 | aui@640c3d7:packages/react/package.json §name "@cutoff/audio-ui-react" |
| AUI-311 | Every fork source file is GPL-3.0-only or TELF; the fork's packages/ are deleted, and aui's code lives in new files with a provenance ledger. | carried | constraint | §0, §2.2, §2.3, §8 | aui@640c3d7:packages/react/src/index.ts §file header "SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-TELF-1.0" |

## Appendix B. Trace check

- **What was checked:** every citation in this file and in `aui-docs/HANDOFF.md` of the form `repo@sha:path … "quote"`. For each, `git show <sha>:<path>` must contain the quote, whitespace aside (a `\|` in a table is read as `|`).
- **Rev 1's result, kept for the record:** REQUIREMENTS.md 347 citations, HANDOFF.md 6 citations, 0 failures. That check did not test sections, or whether a `settled` row quotes the owner (review of rev 1, T-01 and T-09).
- **Rev 2's rules:** the quote must be in the file at that sha, whitespace aside; the section named must hold it; a `settled` row must quote the owner's words, not only the brief's summary.
- **Rev 2's result, 2026-09-29:** REQUIREMENTS.md 356 citations, HANDOFF.md 8 citations, **0 failures**, with sections checked, and every `settled` row quoting the owner. The checker, a scratchpad script of the review session, is not committed.
- **Also checked:** every `AUI-NNN` id in both files is one of the 311 ledger rows, and every one of the 311 rows is used by at least one section of this file (the "Where" column of Appendix A). Every `AUI-AC-NN` named exists in §8, and every `Q-NN` named exists in §9.
- The checker is a scratchpad script of the planning session and is not committed. aui's session may write its own as part of AUI-AC-01.

---

## Review of rev 1

Rev 2 answers the review of rev 1 (2026-09-29). Each finding was checked against the files first.

| Finding | What happened |
|---|---|
| T-01 | Fixed. "Row status" now says `settled` needs the owner's words. AUI-004, AUI-005, AUI-007, AUI-017 and AUI-021 to AUI-023 now quote them. AUI-006 and AUI-020 had only the brief's record behind them, so they are `carried`. Appendix B states the rule. |
| T-09 | Fixed. AUI-297 cites the CLA's §5, where the quote is. |
| T-13 | Fixed. The authority list names both newest blocks, and the pins include mazika@e8124d3. |
| H-14 | Fixed. §2.3 and AUI-291: at the end of T3, `LICENSE.md` becomes a short notice, so no fresh file sits under Tylium's GPL notice. Q-03 adds the description, the visibility and the fork link, which only the owner can change. |
| H-16 | Fixed. The pins name audio-engine@4545674. §1.2 says AE-UI-004 is `accepted` in the engine's rev 2. |
| R-18 | Fixed. Q-16's default is now no daemon on a public repository, and `bana ci` by hand after review. `daemon.token = none` is no longer offered as a guard; bana's own sentence is quoted. AUI-289 says so. |
| H-15 | §7.2 says aui's authors do not open the contract file; the reading list is fixed in `HANDOFF.md`. |
