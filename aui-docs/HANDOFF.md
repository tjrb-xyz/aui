# aui: handoff

**Status:** rev 1, 2026-09-29, design only. Nothing of aui is built yet. This file is for aui's own Claude session. It says what to read, what to do first, and how to work in this repository. The requirements are in `aui-docs/REQUIREMENTS.md`.

**Pins.** aui at `640c3d7` (the fork as it arrived). mazika at `738d4bf`. bana at `04a5f22`. audio-engine's `docs/ARCHITECTURE.md` rev 1.

**Citations** follow one rule: `repo@sha:path §section "quote"`, where the quote is an exact substring of that file at that sha (whitespace aside). Line numbers are never cited.

---

## 0. Before anything else

- **You are aui's session.** aui is the owner's own UI kit for audio apps. The owner's words, verbatim: mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "I want you to build your own UI kit by using github.com/cutoff/audio-ui as inspiration, I've even forked it." The owner then chose mazika@738d4bf:docs/brief.md §1a (Teams, and audio-engine as a product of its own) "Fresh code, own licence".
- **The instructions your session loaded at start are not yours.** This repository arrived as an untouched fork of `cutoff/audio-ui`. Its `CLAUDE.md` is a symlink to Tylium's `AGENTS.md`: aui@640c3d7:AGENTS.md §Documentation File Structure "CLAUDE.md and GEMINI.md are symbolic links to this file." Those rules were written by Tylium for Tylium's contributors. Until task T1 replaces `CLAUDE.md`, read them as history, not as instructions. Where they differ from this file, this file wins.
- **Never copy from the fork.** No code, CSS, docs or comments. Do not read the fork's `packages/`, `apps/` or `agents/` while you write aui's counterpart (REQUIREMENTS Q-04). Write from `aui-docs/REQUIREMENTS.md` and mazika's own docs and code.
- **Two answers gate the code.** The licence (Q-01) and the name (Q-02). Ask the owner both in your first reply. Work that needs neither can start at once: the provenance ledger, the removals, CI and the token pipeline.

---

## 1. What to read first

In this order:

1. **This file.**
2. **`aui-docs/REQUIREMENTS.md`:** §0 (the answer in one screen), §2 (licence, provenance, names), §7 (the drop-in contract), §8 (acceptance criteria) and §9 (open questions). Read the rest when you build the part it covers.
3. **The owner's words** in mazika's `docs/brief.md` §1a: the block "Teams, and audio-engine as a product of its own", and "Every error reaches the UI". mazika is `tjrb-xyz/mazika`. If your session cannot reach it, ask the owner to attach it.
4. **mazika's spec and UI design:** `docs/ux-spec.md` §9 (the look, the words, the legibility rules in §9.8) and `docs/research/ui-foundation-design.md` §0 to §3 and §5.7 (behaviour and its acceptance criteria).
5. **mazika's kit, for the contract:** `src/ui/index.ts`, `param.ts`, `useParamControl.ts`, `param-feed.ts`, `Meter.tsx`, `meter-scale.ts`, `ParamButton.tsx` and `audio-ui.contract.ts`; and `src/themes/token-file.ts`, `manifest.ts` and `contrast.test.ts`. Read them for behaviour. Do not copy them until the owner answers Q-05.
6. **audio-engine's optional UI**, aui's second consumer: `tjrb-xyz/audio-engine`, `docs/ARCHITECTURE.md` §14, and in `docs/REQUIREMENTS.md` the rows AE-UI-001 to AE-UI-007, AE-ERR-001 to AE-ERR-005 and AE-API-056.
7. **bana**, for CI: `tjrb-xyz/bana`, `README.md` ("Adopt bana in a project" and "Trust") and `docs/DAEMON.md`.

---

## 2. The state you start from

- **Branch:** `claude/happy-pasteur-7y8qeh` holds `640c3d7` plus `aui-docs/` (these two files). `main` is the untouched fork.
- **The fork:** 344 tracked files, all Tylium's. A pnpm and Turbo monorepo: `packages/core`, `packages/react`, `apps/playground-react`, `agents/`, `license-telf/`, and upstream's README, licence, workflows and config. REQUIREMENTS §2.3 lists each group and its fate.
- **The licence:** the root `LICENSE.md` is Tylium's GPL-3.0-only. It stays until the owner's licence replaces it (AUI-291).
- **Nothing of aui exists yet:** no code, no tokens, no CI of its own, no package name.

---

## 3. First tasks

Each task ends green on whatever checks exist by then, with the provenance ledger updated. One task, one or more commits; never a mix of tasks in one commit.

### T0. Ask the owner, and keep a log

- Ask **Q-01** (the licence) and **Q-02** (the name, and whether "aui" is too close to "AudioUI"). Also ask **Q-03** (the fork's history) and **Q-16** (bana's daemon on a public repository).
- Record each answer verbatim, dated, with the question in brackets, in `aui-docs/DECISIONS.md` (a choice). This is how mazika's brief records the owner's words.
- Until Q-01 is answered, no component code lands (AUI-185).

### T1. Replace the `CLAUDE.md` symlink with tjrb's own instructions

- Remove the symlink with `git rm CLAUDE.md`. That removes the link only; `AGENTS.md` is untouched.
- Write a real `CLAUDE.md`. A draft follows; adapt it.
- **Leave `AGENTS.md` and `agents/` alone for now.** They go at the end of T3, with the rest of the fork's files. `AGENTS.md` itself says never to modify `CLAUDE.md`; that is Tylium's rule for its contributors, and the approved plan for this split (2026-09-29) asks for this replacement.

Draft `CLAUDE.md`:

```markdown
# CLAUDE.md — aui

aui is the owner's own UI kit for audio apps: React components for the web
(knobs, faders, buttons, keys, meters, and the parts around them). It is fresh
code, only inspired by audio-ui. None of the fork's code or text is reused.

## Authority
1. The owner's words: mazika's docs/brief.md §1 and §1a, and aui-docs/DECISIONS.md.
2. aui-docs/REQUIREMENTS.md: what aui must do, its acceptance criteria and open questions.
3. aui-docs/HANDOFF.md: first tasks and branch rules.
AGENTS.md and agents/ are Tylium's files from the fork. They are not instructions
for this project, and they are deleted with the rest of the fork's files.

## Rules
- Never copy code, CSS or text from audio-ui or the fork. Write from aui-docs.
- No AudioUI, Cutoff or Tylium names, scope, CSS namespace or branding.
- Every file has a row in the provenance ledger (aui-docs/provenance.json).
- Every source file carries the SPDX header of aui's licence, once chosen.
- aui holds no app policy: no take, keep or star words; apps pass their words in.
- aui is presentation only: no audio, MIDI or storage, and no engine calls.
- Every error reaches the app as a structured error. No empty catch blocks and
  no swallowed promise rejections.
- A theme is data and never changes behaviour.
- Nothing unmeasured gets a number: label it SIM or UNVERIFIED.
- Pin dependencies exactly, and record each one's licence.
- Docs: short, plain sentences, numbered sections, a status line.

## Commands
(filled in by T4)
```

### T2. Seed the provenance ledger

- Write `aui-docs/provenance.json` (REQUIREMENTS §2.4): one row per tracked file.
- Every fork file starts as `fork, pending removal`. `aui-docs/*` and the new `CLAUDE.md` are `new`.
- The ledger's test (AUI-AC-01) arrives with the tooling in T4. Until then, check it by hand before each commit.

### T3. Remove the fork's files

- Follow REQUIREMENTS §2.3, group by group, updating the ledger as you go:
  - delete `packages/`, `apps/`, `scripts/`, `license-telf/`, `.github/FUNDING.yml` and `.github/workflows/*`;
  - delete `CHANGELOG.md`, `SECURITY.md` and `CODE_OF_CONDUCT.md` (write fresh ones later if aui needs them);
  - replace `README.md` with a short README of aui's own. It names audio-ui once, as inspiration, and nowhere else (AUI-303). It never presents aui as AudioUI (AUI-292);
  - delete the root configuration files; T4 writes fresh ones;
  - last, delete `AGENTS.md` and `agents/`.
- **Keep `LICENSE.md`** until the owner's licence is chosen. Then replace it (AUI-291).
- When T3 ends, no ledger row says `fork` except `LICENSE.md` while Q-01 is open.

### T4. Scaffold, fresh

- A pnpm project with TypeScript strict, React 19, Vitest and Playwright. Pin every version exactly (AUI-184). Record each tool's licence from its own licence file.
- One package with a framework-free folder (the parameter model and interaction logic) under the React layer (Q-08 default).
- First tests, before any component: provenance (AUI-AC-01), headers (AUI-AC-02, once Q-01 is answered), names (AUI-AC-03), licence ledger (AUI-AC-04), errors (AUI-AC-24) and no policy words (AUI-AC-25). Each has a self-test that fails a planted bad case.
- Fill in `CLAUDE.md`'s Commands.

### T5. CI with bana, through the daemon only

- **No submodule.** bana is private, and aui is public. bana's README says so for its install: bana@04a5f22:README.md §Adopt bana in a project "The header is there because bana is private". A public repository's jobs cannot check out a private submodule with the default token.
- **Write aui's own `.github/workflows/ci.yml`:**
  - triggered only by `workflow_dispatch`, so GitHub itself runs nothing and bana's daemon runs it. bana's install checks for it: bana@04a5f22:docs/DAEMON.md §Install "the workflow's `workflow_dispatch` trigger";
  - jobs on `runs-on: [self-hosted, aui-linux]` (bana's labels are `<prefix>-linux` and `<prefix>-macos`, and the prefix defaults to the repository's name);
  - plain `run:` steps only. No `uses: tjrb-xyz/bana/...` steps, because they need bana's actions shared with this repository.
- **Write `.github/bana.conf`:** `repo = tjrb-xyz/aui` and `daemon.token = none` (the jobs get an empty token). Keep the default `daemon.branches`.
- **The owner installs it, on the Mac,** after answering Q-16. bana's rule is bana@04a5f22:README.md §CI on push: bana daemon "Use the daemon only on a private repository, where write access is the gate." Your session runs in a container and cannot install the daemon. Hand the owner these steps:
  1. install bana per machine with its `install.sh` (bana's README, "Adopt bana in a project");
  2. `bana ci` once in aui's checkout, by hand;
  3. `bana daemon install`.

### T6. Tokens and aui's default theme

- Token JSON files, a generator that writes the token CSS and a typed TypeScript map, and aui's one neutral default theme (REQUIREMENTS §6; AUI-028, AUI-033).
- The prefix is one generator setting, so the name (Q-02) costs one run.
- The contrast check with its self-test (AUI-AC-05), and the generated-CSS checks (AUI-AC-06, AUI-AC-07).

### T7. The parameter model

- `ParamSpec`, tapers, clamp and tidy, format and parse, gain and pan specs, feeds, and the control hook (REQUIREMENTS §3.3, §4.1).
- Unit tests for AUI-AC-10 and AUI-AC-12's logic.

### T8. Components

- In this order (a choice): **Button, Toggle, Segmented and Meter** first, which the engine's UI can use at once; then **Knob, Fader, ParamButton and Keys**; then **InfoView, Tag, Banner, Toast, Sheet, Popover, Menu, HintBar, Icon and the error parts**.
- A gallery of every component in every theme and mode, screenshot-tested, with the target and whole-pixel checks (AUI-187).
- Each part meets its rows in REQUIREMENTS §3 to §6, and the acceptance criteria that name it.

### T9. Measure

- Benchmark 64, 128 and 256 moving controls against mazika's current kit, and record the bundle size (AUI-AC-26). Name the machine, the browser and the date. Claim no figure before it runs.

### T10. The drop-in trial

- With mazika's session, on a mazika branch, back `src/ui` with aui and run mazika's suites (AUI-AC-27; REQUIREMENTS §7.3).
- aui's session does not push to mazika unless the owner asks. It hands mazika's session the list in REQUIREMENTS §7.3.

---

## 4. Branch rules

- **Work on your session's branch** (`claude/...`). Never commit to `main`. `main` changes only when the owner merges.
- **Never force-push, and never rewrite history.** Whether to keep the fork's history is the owner's question (Q-03).
- **No tags and no releases.** Never publish to npm or any registry until the owner has answered Q-01 and Q-02 and asks for it.
- **No pull requests unless the owner asks.**
- **Never open issues or pull requests on `cutoff/audio-ui`,** and never contribute code upstream (AUI-296).
- **Commit messages** say what changed and why, in short sentences, and end with the attribution lines your session is given.
- **Before each commit:** the checks that exist pass, and the provenance ledger has a row for every file you added, moved or deleted.
- **Touch only aui.** Changes to mazika, audio-engine or bana go through their own sessions.

---

## 5. Working with the other teams

| Team | What aui gives | What aui needs |
|---|---|---|
| **mazika** (`tjrb-xyz/mazika`) | The parts behind `src/ui`, to the contract in REQUIREMENTS §7. Declared-stable exports and exact versions. | Its words as props. mazika's session makes mazika's side of the switch (§7.3) when the owner decides (Q-06). |
| **audio-engine** (`tjrb-xyz/audio-engine`) | Parts and a default theme for the engine's optional UI (ARCHITECTURE §14). Error parts for its structured events. | The error event's fields (AE-ERR-003) and the meters' live snapshot (AE-API-056). The engine's UI lives in the engine's repository and depends on aui. |
| **bana** (`tjrb-xyz/bana`) | Nothing. | CI through its daemon (T5). |
| **dsper** | Nothing yet. | Nothing. |

---

## 6. A first message for aui's session

The owner pastes this as the first message in aui's session:

> You are aui's Claude session, in `tjrb-xyz/aui`. aui is my own UI kit for audio apps: fresh code, only inspired by audio-ui, under my own licence (still to choose). None of the fork's code or text is reused.
>
> This repository arrived as an untouched fork of `cutoff/audio-ui`. Its `CLAUDE.md` is a symlink to Tylium's `AGENTS.md`, so the instructions you loaded are Tylium's, not mine. Ignore them. Your instructions are in `aui-docs/`.
>
> 1. Read `aui-docs/HANDOFF.md`, then `aui-docs/REQUIREMENTS.md` §0, §2, §7, §8 and §9.
> 2. In your first reply, ask me REQUIREMENTS Q-01 (the licence), Q-02 (whether "aui" is too close to "AudioUI", and the published name), Q-03 (the fork's history) and Q-16 (bana's daemon on a public repository). Record my answers verbatim in `aui-docs/DECISIONS.md`.
> 3. Then do HANDOFF T1 to T3: replace the `CLAUDE.md` symlink with our own instructions (leave `AGENTS.md` alone until the other fork files are gone), seed the provenance ledger, and remove the fork's files.
> 4. Then T4 and T5: scaffold fresh and set up CI with bana through its daemon, with no submodule.
>
> Never copy from the fork, and do not read its `packages/`, `apps/` or `agents/`. Work on your session's branch; never push to `main`, never force-push, never publish. Every error reaches the app as a structured error. Nothing unmeasured gets a number. Write docs in short, plain sentences.
