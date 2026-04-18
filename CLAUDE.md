# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Offline-capable PWA for Chinese travel vocabulary and phrases. Designed to be installed on iOS via Safari → Add to Home Screen. No build system, no dependencies, no tests.

Three files make up the app:

- `index.html` — app shell (markup, inline CSS, renderer JS). Contains no vocabulary data.
- `content.json` — all vocabulary and phrase data (tabs → sections → entries).
- `sw.js` — service worker providing cache-first offline support.

## Running / previewing

There is no build step, but the app **must be served over HTTP(S)** — opening `index.html` via `file://` will break both the `fetch('./content.json')` call and the service worker. Quick local server:

```
python3 -m http.server
```

Then browse to `http://localhost:8000/`. For the install-to-home-screen flow and native `zh-CN` TTS voices, test on real iOS Safari — desktop browsers have different voice availability.

## Architecture

### Data flow
On load, `index.html` fetches `content.json` and renders the entire tabbed card UI from it. There is no static card markup in the HTML. The rendering pipeline is: `renderTabs()` + `renderPanels()` → `wireTabs()` → `wireCards()` (TTS click handlers).

### Content shape (`content.json`)
```
{ "tabs": [ { "id", "label", "hanziLabel", "cardClass", "sections": [ { "title", "entries": [ {"hanzi","pinyin","meaning"}, … ] } ] } ] }
```
- `cardClass` is either `"card"` (vocabulary style — compact, no accent) or `"phrase-card"` (phrase style — wider, gold left border). The renderer derives everything else from this: field classnames get a `phrase-` prefix for phrase cards, and the grid container becomes `.phrase-grid` vs `.grid`.
- Tone marks in `pinyin` must be real diacritics (`nǐ hǎo`), not numbered (`ni3 hao3`).
- To add a tab, section, or entry: edit `content.json`. No code change needed.

### TTS
One generic click handler on `.card, .phrase-card` reads `.hanzi` or `.phrase-hanzi` and speaks it via `window.speechSynthesis`. Voice selection prefers `zh-CN`, falls back to any `zh-*`. `synth.cancel()` runs on tab switch and before each utterance to prevent overlap. The `.speaking` class is the visual-feedback hook.

### Offline / PWA
- `sw.js` precaches `./`, `./index.html`, `./content.json` on install and serves cache-first with a network fallback that also populates the cache. Bump the `CACHE` constant (`chinese-travel-v1` → `v2`) when shipping a change you want users to pick up — otherwise the old version stays cached indefinitely.
- iOS install uses the `apple-mobile-web-app-*` meta tags and inline SVG data-URI icons. No `manifest.json` — iOS doesn't need one, and the app isn't targeting Android install.
- `@media (display-mode: standalone)` hides the install hint when launched from the home screen; the hint is also dismissible with `localStorage.hintDismissed`.

## Editing conventions

- **Content edits go in `content.json`.** Do not reintroduce hardcoded card markup in `index.html`.
- No external fonts or network assets. Fonts come from the system stacks defined in `:root` (`--font-hanzi`, `--font-mono`, `--font-serif`, `--font-sans`). Don't add `<link>` to Google Fonts etc.
- If you add a new static asset (e.g. an image, another JSON file), add its path to `ASSETS` in `sw.js` and bump `CACHE` so existing installs refetch.
