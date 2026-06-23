---
name: opendeck
description: Build animated, narrated HTML presentation decks — slides that reveal step-by-step as you click, hover tooltips, a thumbnail rail, fullscreen, plus AI voice narration generated in-browser via ElevenLabs that can be baked into a fully offline file. Use when the user wants a presentation, deck, or slides that animate step-by-step, a self-running or narrated presentation, a deck to present and export, or anything described as "click through and it talks." Can also package the deck as a portable `.deck` file (deck.json + zipped standalone HTML) for playback in a compatible deck player app.
license: MIT
metadata:
  author: Sinisha Djukic
  version: 1.1.7
  created: "2026-06"
---

# OpenDeck — Narrated Animated Presentations

Build an HTML slide deck with **in-slide step animations, hover tooltips, a thumbnail rail, fullscreen, and AI voice narration** — including an in-page "Audio Studio" that generates the voiceover with ElevenLabs and can bake it into a fully offline file.

Use this skill when the user asks for: a presentation/deck/slides that animates step-by-step, a self-running or narrated presentation, a deck they can present and also export, anything describing "click through and it talks," or to **package a deck as a portable `.deck` file** for a compatible player app.

---

## What you ship

The shippable kit lives in this skill's **`assets/`** folder. A deck is a single deck file (`starter-deck.html`, or a `.dc.html` Design Component when building inside a Claude design project) plus five companion files (all copied into the **same folder** as the deck), and a zero-dependency bundler for offline export:

| File | Role | You edit it? |
|---|---|---|
| `deck-stage.js` | Slide engine: auto-scaling, keyboard nav, thumbnail rail, speaker notes. | No — drop in as-is |
| `deck-enhance.js` | `data-step` reveals + step dots, `data-tip` tooltips, Fullscreen (`F`) & Slides (`S`) buttons, edit-mode guard. | No — drop in as-is |
| `narration-script.js` | The narration **text**, one line per slide/step. | **Yes** — this is where you write narration |
| `deck-narration.js` | Narrate + Auto-play controls (merged into the deck overlay) and the Audio Studio. | No — drop in as-is |
| `narration-audio.js` | **Generated**, not shipped. Holds the audio clips as base64 so narration plays offline. Created by the Studio's "Download audio". | No — produced by the user |
| `deck-export.js` | **In-browser bundler**: `deckExport.standalone()` inlines every local script, the baked audio, and the web fonts into one self-contained offline `.html`; `deckExport.deck()` wraps that as a portable `.deck` package (deck.json + zip). Plain browser JS — no build step. | No — drop in as-is |

`deck-animation.css` is the **required CSS** for reveals/dots/tooltips — paste it into the deck's `<style>` (the starter deck already has it inline).

---

## Step 1 — Scaffold the deck

Start from **`assets/starter-deck.html`** (in this skill) — it's a complete, working deck you can open directly: 3 slides, a stepped reveal, tooltips, the rail, and the narration controls. Copy it into the project together with the five companion files from `assets/` (`deck-stage.js`, `deck-enhance.js`, `narration-script.js`, `deck-narration.js`, `deck-animation.css`) — **all in one folder, no subdirectories**. The engine resolves `<script src>` and the `deck-stage` mount relative to the deck, so a flat layout is required. Then replace the demo `<section>`s with the real slides.

### Two wiring shapes — pick the one that matches where you're building

**A. Plain HTML** (what `starter-deck.html` uses; also the shape of a "Save as standalone" export). Mount `<deck-stage>` directly and load the scripts at the **end of `<body>`**, stage first:

```html
<deck-stage width="1920" height="1080">
  <section data-label="Title" data-speaker-notes="…">…</section>
  <section data-label="Agenda" data-speaker-notes="…">…</section>
</deck-stage>

<script src="deck-stage.js"></script>
<script src="deck-enhance.js"></script>
<script src="narration-script.js"></script>
<!-- <script src="narration-audio.js"></script>   ← uncomment once baked -->
<script src="deck-narration.js"></script>
<script src="deck-export.js"></script>
```

**B. Design Component** (when building inside a Claude design project, authored with `dc_write`). The deck is a `.dc.html`; the project's own runtime hydrates it. Mount the stage with `<x-import>` and load the other scripts from `<helmet>` in this order:

```html
<x-dc>
<helmet>
  <style> …your slide CSS + the required animation CSS… </style>
  <script src="deck-enhance.js"></script>
  <script src="narration-script.js"></script>
  <!-- <script src="narration-audio.js"></script>   ← uncomment once baked -->
  <script src="deck-narration.js"></script>
  <script src="deck-export.js"></script>
</helmet>

<x-import component-from-global-scope="deck-stage" from="./deck-stage.js"
          width="1920" height="1080" hint-size="100%,100%">
  <section data-label="Title" data-speaker-notes="…">…</section>
  <section data-label="Agenda" data-speaker-notes="…">…</section>
</x-import>
</x-dc>
```

(Do **not** hand-write a `.dc.html` as a raw file and ship it — a DC needs the project's runtime to hydrate. The portable, openable artifact is always the plain-HTML form.)

Rules that matter either way:
- **Slides are the direct `<section>` children** of the stage. Author content at the **1920×1080** design size; the stage scales to fit.
- Each slide: `data-label="…"` (rail label) and `data-speaker-notes="…"` (presenter note, travels with the slide on reorder).
- Don't set `position`/`inset` on `<section>` — the stage positions them.
- Keep the `deck-stage:not(:defined){visibility:hidden}` rule to avoid an unstyled flash.
- `narration-audio.js` is commented out until audio is baked.

---

## Step 2 — Add step animations

Give any element a `data-step="N"` (N = 1, 2, 3…). It starts hidden and reveals when the viewer reaches step N (→ / click / Space). Advancing past the last step moves to the next slide; ← steps back.

```html
<section data-label="Pillars">
  <h2>Three pillars</h2>
  <div class="tile" data-step="1">First</div>
  <div class="tile" data-step="2">Second</div>
  <div class="tile" data-step="3">Third</div>
</section>
```

- **Step dots** are auto-injected (bottom-left) on any slide that has steps — no markup needed.
- Multiple elements can share the same `data-step` to reveal together.
- The **required CSS** (`deck-animation.css`) defines what `.step-visible` / `.step-current` / `.step-past` look like. Without it, `data-step` elements just stay hidden. Paste it into your `<style>` and restyle freely (it's plain fade-and-rise by default).
- Elements with **no** `data-step` are visible on slide entry (and can stagger in via the optional `.frame` rule in the CSS).

### Tooltips
Add `data-tip="explanatory text"` to any element for a smart hover tooltip (auto-flips above/below). Tooltips on a not-yet-revealed step stay suppressed until that step shows.

---

## Step 3 — Write the narration

Edit `narration-script.js`. It exposes `window.__NARRATION` with a `slides[]` array — **one entry per `<section>`, in order**. Each slide's `lines[]` maps to steps:

- `lines[0]` → spoken when the slide first appears (before any reveal)
- `lines[1]` → spoken at `data-step="1"`, `lines[2]` at `data-step="2"`, …

```js
window.__NARRATION = {
  voiceId: "",                       // optional: pre-fill the Studio's Voice ID
  modelId: "eleven_multilingual_v2",
  gapSeconds: 0.5,                   // beat between a clip ending and auto-advance
  slides: [
    { lines: ["Welcome. Here is the plan."] },                  // slide 1, no steps
    { lines: ["", "First pillar.", "Second.", "The payoff."] }, // slide 2, 3 steps
    { lines: ["Thank you."] }
  ]
};
```

Writing guidance (matches how the voice reads best):
- **One idea per sentence**, short sentences.
- **Spell acronyms with periods** so they're read as letters: `"A.I."`, `"U.X."`, `"G.L.M."`.
- Respell numbers a TTS misreads: `"twenty twenty-six"` instead of `2026`.
- Empty string `""` = silent for that step.

> **These respellings are spoken-only — they belong ONLY in `narration-script.js`, never in the slide `<section>` text.** Spoken text and on-screen text are separate sources: the engine reads `narration-script.js` for the voice and never echoes it on screen. So the slide keeps the human-readable form and the narration line carries the pronunciation hint:
>
> | Appears on the slide (`<section>`) | Spoken (`narration-script.js`) |
> |---|---|
> | `Our HR team` | `"Our H.R. team"` |
> | `Shipping in 2026` | `"Shipping in twenty twenty-six"` |
> | `The AI roadmap` | `"The A.I. roadmap"` |
>
> If `H.R.` ever shows up *in the slide*, the respelling leaked into the markup — fix the `<section>`, not the narration. (Optional, if you'd rather keep even the narration source clean: ElevenLabs also honors SSML `<say-as interpret-as="characters">HR</say-as>` and pronunciation dictionaries, but support varies by model — the period-spelling is the simplest reliable default.)

If the user wants you to draft the script, write it from their instructions, keep their tone, and confirm before they generate (generation costs ElevenLabs credits).

---

## Step 4 — Generate the audio (the user does this, in-browser)

Audio is generated **in the browser** so the ElevenLabs key never leaves the user's machine and is never baked into a file.

While authoring, the control bar shows a blue **Studio** button (a different colour from the neutral playback controls, so it reads as a build-time tool). It opens a menu of authoring actions:
- **Audio studio** — a 5-step wizard: **① Connect** (API key + Voice ID) → **② Generate** (one clip per step, progress bar) → **③ Download** (saves `narration-audio.js`) → **④ Place** (move that file next to the deck) → **⑤ Export** (tells the user to ask their AI agent: *"Export this presentation as a standalone file."*). The export step deliberately promotes only the agent path.
- **Cue overview** — a checklist of every step showing which have narration text and which have audio, to spot gaps before exporting.
- **Export standalone HTML** — runs the in-browser bundler (Route B) directly, without the console (power-user shortcut).

The **Studio button is hidden in the exported standalone** (via `window.__DECK_EXPORTED`). The console API still works too (`deckNarration.studio()`), printed on every load.

> Narration text is edited only in `narration-script.js` (the file). There is intentionally **no in-browser narration editor** — browser edits wouldn't survive into the export, so they'd mislead. Change the wording in `narration-script.js`, then re-generate.

Tell the user to:
1. Open the deck, click **Studio → Audio studio** (or run `deckNarration.studio()` in the dev console).
2. **① Connect:** paste their **ElevenLabs API key** and **Voice ID** (ElevenLabs → Voices → the voice → ID), then **Next**.
3. **② Generate:** click **Generate narration** — one clip per non-empty line, with a progress bar, cached in the browser (IndexedDB). On success the wizard advances. To re-do lines: edit `narration-script.js`, reopen, tick **Re-generate clips that already exist**, and Generate.
4. **③ Download:** click **Download audio** to save `narration-audio.js`.
5. **④ Place:** move `narration-audio.js` into the same folder as the deck.
6. **⑤ Export:** ask the AI agent to *"Export this presentation as a standalone file"* (the agent runs the Route A build).

> **Recommend (can't enforce) Chrome or Edge for generation — or serve over http.** The clip cache uses IndexedDB, which **Chromium allows from `file://` but Safari blocks and Firefox treats inconsistently** (the ElevenLabs call itself is fine everywhere — it returns `Access-Control-Allow-Origin: *`). Where the cache is blocked the generated clips live only in memory for that session, so **the user must click "Download audio" before reloading** or they'll regenerate (and re-pay). The Studio detects this and shows an inline warning; the deck also prints the tip to the console. This only affects *generating* — once audio is **baked**, playback reads the inlined map (no IndexedDB), so an exported deck plays offline in **every** browser, including from a double-clicked `file://`.

Console API (all available globally once the deck loads):
- `deckNarration.studio()` — open the generation panel
- `deckNarration.play()` / `.stop()` — start/stop auto-play
- `deckNarration.narration(true|false)` — narration on/off
- `deckNarration.status()` — `{clips, narrationOn, autoplay}`

---

## Step 5 — Playback model (what the two on-screen controls do)

The deck overlay (bottom bar, alongside Prev/Next/Reset/Fullscreen/Slides) gains two controls:

- **Narrate** — master on/off. When on, **every step you land on speaks**, whether you click through manually, use arrows, click a tile, or jump via a thumbnail.
- **Auto-play / Pause** — hands-free: reveals a step, plays its clip, and advances when the clip ends. Turning it on implies Narrate on. **Pausing keeps Narrate on**, so you can keep clicking through and still hear each step.

It is a **single-driver** model: any step change cancels stale audio and any pending auto-advance, then plays the clip for the step you're actually on — so manual clicks during auto-play never double-advance or desync.

---

## Step 6 — Bake audio in & export offline

To make narration play on **any machine, offline, with no key**:

1. In the Studio, click **Download audio** → produces `narration-audio.js` (all clips as base64 data URLs).
2. Move that file into the deck's folder (next to `deck-stage.js` etc.).
3. Bundle everything into one self-contained `.html`. There are two routes — both produce the same single file with **zero tools to install**:

### Route A — ask the skill to build it (recommended; no server, no Python)

The user just says *"build the standalone"* and **you (the agent) do the bundling with your own file tools** — nothing runs on their machine. Procedure:

1. **Read** the deck `.html` and inline every local `<script src="…">` (the kit scripts + `narration-audio.js` if present) by replacing each tag with `<script>…file contents…</script>`. Skip absolute/CDN URLs.
   - **Mark it exported:** always inject `<script>window.__DECK_EXPORTED=true;</script>` *before* the (inlined) `deck-narration.js`. This hides the authoring **Studio** button in the export, and — when no audio was baked — the Narrate + Auto-play buttons too (nothing to play). With baked audio, Narrate/Auto-play stay; the Studio button is always gone.
   - **Activate baked audio:** if `narration-audio.js` is in the folder, also un-comment its `<script>` line so the inlined copy is live. If it's absent, just bundle without it (the `__DECK_EXPORTED` flag handles hiding the now-useless controls).
   - **⚠ Neutralize closing tags:** the engine files contain the literal text `</script>` inside their doc-comments. Before inlining a file, replace `</script` with `<\/script` in its contents, or the first one will close the block early and corrupt the file.
2. **Fonts — ask the user** which they want (use `AskUserQuestion`):
   - **Bake the fonts in (recommended)** — fetch each `woff2` the deck's Google-Fonts `<link>` references and inline it as a base64 `@font-face` rule, then remove the `<link>`/`preconnect`. Result renders identically offline.
   - **Keep the CDN `<link>`** — smaller file, but the custom fonts only render with an internet connection; offline (or if the fonts aren't installed locally) it falls back to system fonts and looks different.
3. **Write** the result as `your-deck.standalone.html`. Expect it to be large (the audio dominates — often several MB).

### Route B — self-service in the browser (`deck-export.js`)

For a user who'd rather not involve the agent: load **`deck-export.js`** (it's one of the kit files) and run this in the browser dev console:

```js
deckExport.standalone()      // downloads your-deck.standalone.html
```

It does the same inlining (scripts, same-origin stylesheet `<link>`s and their `url()` fonts/images, `<img>` sources, baked audio, and Google fonts — with the `</script>` fix) entirely in the browser. It auto-detects `narration-audio.js` and always tries to bake fonts (falling back to the CDN link if offline). Because it `fetch()`es same-origin files, the deck must be **served over http(s)** for this route — not opened via `file://`. *Inside a Claude design project,* the project's own **"Save as standalone HTML"** export is the equivalent.

Without baking audio, the export is a clean silent deck: the **Narrate + Auto-play controls are hidden** (nothing to play), while all the animation, tooltips, and navigation still work. The clips live only in the browser that made them, so voice returns only if someone regenerates with a key (the `deckNarration.studio()` console API stays available for that).

### Route C — package as a portable `.deck` file

When the user wants to present on a phone/tablet, or asks to **"export as a `.deck`"**, package the deck for a compatible deck player app. A `.deck` is a Zip archive with MIME type `application/x-deck` whose root holds:

```
my-talk.deck
├── deck.json       # the manifest — see assets/deck.schema.json
├── index.html      # the standalone export (Route A/B output) — fully self-contained
└── thumbnail.png   # optional preview image shown in the player's library
```

Two routes, same output:

```js
deckExport.deck()   // browser console: builds the standalone, writes deck.json, downloads my-talk.deck
```

…or **ask the agent** to do it with file tools: produce the standalone HTML exactly as in Route A, write it as `index.html`, add a `deck.json`, and zip the two at the archive root as `<id>.deck`. Either way the result is one portable file.

The `deck.json` manifest is defined by **`assets/deck.schema.json`** (JSON Schema) — read that file for the exact field set, types, patterns, and defaults. In short: only `entry` (`"index.html"`) is strictly required; `id` (the filename), `title`, `orientation` (`landscape` for these 1920×1080 decks), and optional `author`/`version` round it out. Add `"$schema": "./deck.schema.json"` to a hand-written `deck.json` for editor validation.

**Optional thumbnail.** Drop a `thumbnail.png` (or `.jpg`/`.webp`/`.svg`) next to the deck before exporting and `deckExport.deck()` bakes it into the package and records `"thumbnail": "thumbnail.png"` in `deck.json`; or pass one explicitly: `deckExport.deck({ thumbnail: "data:image/png;base64,…" })`. Hand-writing the package? Add the image file at the archive root and point `thumbnail` at its relative path. Players that support it (e.g. the OpenDeck app) show the image in their library; players that don't simply ignore the field.

Because the deck is the standalone bundle, **baked audio plays offline** inside the player too — it renders in a real web origin, so all animation, narration, fonts, and tooltips work exactly as in the browser.

---

## Gotchas / notes

- **CSS is mandatory.** If reveals don't show, the animation CSS isn't in the deck's `<style>`. Copy from `deck-animation.css`.
- **Load order & location.** The four scripts go inside `<helmet>` in the given order, and **all files must sit in the same folder as the deck** (the engine resolves `src` paths relative to the deck — a subfolder breaks the helmet scripts). For a plain (non-DC) HTML deck, put the scripts at the end of `<body>` instead, with `deck-stage.js` first.
- **Design tokens.** The CSS ships with plain values; if the deck has a design system, swap them for its tokens (easing, accent color) so reveals/dots/tooltips match.
- **Slide count.** `slides[]` in the script should line up with the `<section>`s; extra/short entries just stay silent — fine while drafting.
- **Don't rename the globals.** `deck-enhance.js`/`deck-narration.js` look for `window.__NARRATION` and `window.__NARRATION_AUDIO`; keep those names.
- **Editing safety.** `deck-enhance.js` suppresses nav keys while you're typing in a field/contenteditable, so authoring text won't flip slides.
- **Hosting & CSP.** A deck — and any standalone export — is fully inline: an inline `<style>` block, an inline `<script>`, and (on export) the whole engine inlined too. It works on any host that doesn't impose a restrictive Content-Security-Policy, which is the default for static hosting (GitHub Pages, S3, Netlify/Vercel, plain nginx) — so most self-hosting just works. If the host *does* set a CSP, the deck needs `script-src 'unsafe-inline'` and `style-src 'unsafe-inline'` (the engine uses no `eval`/`new Function`, so `'unsafe-eval'` is **not** required), plus `data:`/`blob:` for embedded fonts, images, and audio. A minimal working policy: `default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; media-src 'self' data: blob:; font-src 'self' data:`. On Cloudflare `_headers` (and Pages-style headers generally), matching rules *accumulate* rather than override — a strict global CSP plus a relaxed deck CSP emits two headers and the browser enforces the stricter intersection, which silently breaks decks. Scope CSP so exactly one rule matches the deck path.
