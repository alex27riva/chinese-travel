# UI & Visual Review

Review date: 2026-09-09, against v0.11.1. Status updated through v0.16.0.

Method: full read of `style.css` and `index.html`, plus layout checks in Chrome with the body constrained to a 390px column to approximate a phone. Colors below are judged from the CSS values (the test browser had forced dark mode on, so light-mode screenshots were not reliable).

Overall: the warm paper / ink / seal-red / gold identity is distinctive and worth keeping. The editorial hierarchy (red tracked section labels, hanzi-font subsection headers with red underline) works. The problems are mostly mobile layout, text legibility, and dark-mode contrast, not the visual direction.

Items are ordered by impact within each group. See the suggested bundle at the end.

---

## 1. Layout

### 1.1 Too much chrome before content ✅ done in v0.13.0
On a phone the first card sits roughly 500px down: title, three subtitle lines, install banner, tab strip, toolbar. Every launch and every tab switch (which scrolls to top) pays this cost.

- Cut the header to title + one subtitle. Move "hàn + handy" and the tap hint into a small ⓘ / about popover.
- Make the toolbar sticky: `position: sticky; top: 0` with a solid `--paper` background and a hairline bottom border, so search and the toggles stay reachable while scrolling.

### 1.2 Bottom tab bar on mobile ✅ done in v0.12.0
Five uppercase mono tabs at 390px are cramped; "VISA FREE" touches the edge of the strip. Use the native mobile pattern: a fixed bottom bar with hanzi label + tiny English label per tab, `padding-bottom: env(safe-area-inset-bottom)`. It is thumb-reachable, frees the top of the screen, and pairs naturally with the existing swipe gesture. Keep the current top strip for viewports ≥ 700px.

### 1.3 Card text alignment is off ✅ done in v0.14.0
`.hanzi` has `padding-left: 1.8rem` to dodge the star button, but `.pinyin` and `.meaning` sit flush left. Three text lines, two different left edges. Fix: hanzi flush left, and move the action icons into a right-hand column (star top-right, speaker bottom-right, or both stacked). Same fix for `.phrase-hanzi`, which pads both sides.

Icons are also 22px at 0.35 opacity: hard to see and hard to hit. Give them a 36px hit area (keep the glyph 16–18px) and raise resting opacity to about 0.55.

### 1.4 Phrase hanzi too small ✅ done in v0.14.0
`.phrase-hanzi` is 1.25rem for text the user will show to a taxi driver or waiter. Hanzi is the product. Go to 1.6rem with `line-height: 1.25`. Vocab hanzi at 2rem is right.

### 1.5 Floating theme / language pills ✅ done in v0.13.0 (anchored in header, scroll away)
`.theme-btn` and `.lang-switch` are fixed at the screen corners and hover over cards while scrolling. Fold them into the sticky toolbar row, or into a ⚙ sheet opened from the bottom bar.

---

## 2. Typography

### 2.1 Four font families is too many ✅ done in v0.14.0
Hanzi sans, mono for UI and pinyin, serif italic for meanings, sans for buttons. The busiest combination lands on the smallest text: meanings are 0.78rem italic serif in `--muted`, so the translation, the thing beginners need most, is the hardest text to read.

- Keep the hanzi stack.
- One system sans (`--font-sans`) for everything else.
- Mono only for pinyin, if you like the tone-mark alignment. Otherwise sans medium.
- Meaning: 0.85rem, regular weight, `--text-2` (see tokens below). Serif italic can survive on the header subtitle as flavor.

---

## 3. Color

### 3.1 Add a semantic token layer ✅ done in v0.13.1
Today `--red`, `--gold`, `--muted` are used for text, borders and fills interchangeably. Add semantic tokens on top and map dark mode only through those:

```css
:root {
  --text: var(--ink);
  --text-2: #665a4c;        /* secondary text, replaces --muted for copy */
  --surface: var(--paper);
  --surface-2: var(--card-bg);
  --line: var(--border);
  --accent: var(--red);
  --accent-soft: rgba(192,57,43,0.08);
  --accent-2: var(--gold);
}
```

Then components reference `--text-2`, `--line`, `--accent` and never the raw palette. Retuning becomes a token edit instead of a hunt through rules.

### 3.2 Light-mode contrast ✅ done in v0.13.1 (`--text-2: #665a4c`)
- `--muted` `#7a6e5f` on card `#faf6ee` ≈ 4.6:1. Passes at body size, borderline at 0.78rem italic. Darken secondary text to about `#665a4c` (≈ 6:1).
- Gold `#b8960c` on paper is ≈ 2.9:1. Fine for borders and the star fill, never for text. It is not used for text today; keep it that way.
- Red `#c0392b` on card ≈ 5.3:1, fine for pinyin at 0.72rem.

### 3.3 Dark mode is muddy ✅ done in v0.13.1
Paper `#141210` and card `#201c14` are too close, so cards do not lift off the page, and border `#2e2820` is nearly invisible.

Suggested values:

| Token | Current | Suggested |
|---|---|---|
| `--paper` | `#141210` | `#12100d` |
| `--card-bg` | `#201c14` | `#1e1a15` |
| `--border` | `#2e2820` | `#3a3128` |
| `--gold` | `#e0b828` | `#d9b544` (less neon) |
| `--red` | `#f07060` | keep |
| `--muted` | `#b8a890` | keep |

Add a faint inset top highlight on cards in dark mode for edge definition, e.g. `box-shadow: inset 0 1px 0 rgba(255,255,255,0.03)` (as a token, per the no-hardcoded-color rule).

### 3.4 `theme-color` meta is static ✅ done in v0.13.1
`<meta name="theme-color" content="#1a1208">` stays ink-dark in light mode, giving a dark Android status bar over a cream page. Update it from `setTheme()`, or use two meta tags with `media="(prefers-color-scheme: …)"`.

### 3.5 Two hardcoded colors remain ✅ done in v0.13.1
`.quiz-btn.active` uses raw `rgba(184,150,12,0.1)` and its dark override raw `rgba(224,184,40,0.12)`. Move to a `--gold-soft` token. These are the only violations of the "all colors are tokens" rule.

---

## 4. Micro-polish

- ~~**Press feedback.**~~ done in v0.16.0 (`scale(0.98)`, 80ms, on `.card` / `.phrase-card`).
- ~~**Speaking state.**~~ done in v0.14.0 (speaker pulses while speaking).
- ~~**Quiz mode placeholder.**~~ done in v0.16.0 (dashed `::after` line, label from `--quiz-hint`, EN/IT).
- **Card radius.** 4px is a deliberate print / seal feel. Resting `0 1px 2px var(--shadow)` added in v0.16.0; the border-opacity half is still open.
- **Tab active state.** Solid ink block × 5 tabs is heavy. Try a 2px red underline with ink text in the top strip; reserve solid fill for the bottom bar.
- ~~**Search pill width.**~~ done in v0.13.0.
- ~~**Viewport meta.**~~ done in v0.16.0.

---

## 5. Suggested v0.12 bundle

Order by value for effort:

1. ~~Card alignment + icon hit areas (1.3)~~ done in v0.14.0
2. ~~Phrase hanzi size (1.4)~~ done in v0.14.0
3. ~~Typography consolidation (2.1)~~ done in v0.14.0
4. ~~Contrast and dark-mode retune + semantic tokens (3.1–3.3, 3.5)~~ done in v0.13.1
5. ~~Compact header + sticky toolbar (1.1)~~ done in v0.13.0
6. ~~Bottom tab bar on mobile (1.2)~~ done in v0.12.0
7. ~~Fold theme / language controls into the toolbar or a sheet (1.5)~~ done in v0.13.0 (kept in header, no longer fixed)

Items 1–4 are CSS-only and about an hour of work. Items 5–7 are a mobile shell rework, roughly two hours, and should ship together.

---

## Appendix: remaining items from the code review

Fixed in v0.10.2–v0.11.1: tone-insensitive search, iOS PNG icon, swipe guard in favorites overlay, docs sync, Numbers content, long-press copy.

Still open:

- ~~**Update toast.**~~ done in v0.15.0.
- ~~**Single version source.**~~ done in v0.16.0. `CACHE = 'handy-v0.16.0'` in `sw.js` is the only literal; `showAppVersion()` reads it back via `caches.keys()`.
- ~~**`alert()` in `speak()`**~~ done in v0.16.0 (toast, `CHROME.ttsUnsupported`).
- **Missing zh voice detection.** With no `zh-*` voice, iOS reads hanzi with the default voice. Show a one-time hint pointing to Settings → Accessibility → Spoken Content.
- ~~**`lang="zh-CN"` on hanzi elements**~~ done in v0.16.0 (all hanzi, incl. tab labels and static glyphs).
- ~~**Accessibility.**~~ done in v0.16.0 (`role="button"` + `tabindex` + Enter/Space on cards, `aria-selected` / `aria-controls` / `role="tabpanel"`, Escape closes favorites, `:focus-visible` ring).
- **Duplication.** Panel-reset logic exists in both `filterCards('')` and `handleCardClick`; `refreshFavoritesPanel` and `showFavorites` are near-identical.
- **Content validator + CI.** Script that checks pinyin diacritics, `en`/`it` presence, and duplicate hanzi per tab; run on push.
- **Content ideas.** Emergency numbers (110 / 120 / 119), show-mode (fullscreen hanzi), slow TTS on long-press, quiz shuffle, search-result highlight after navigation.
- **Maskable icon** for Android adaptive icons.
- ~~**`.gitignore`**~~ done in v0.16.0.
