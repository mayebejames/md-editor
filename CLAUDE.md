# Markdown Editor

A plain-text markdown editor that lives in a **single file, `index.html`**. It must work both when double-clicked locally (`file://`) and when served from James's website. It should work well on an iMac, a laptop and a phone.

## Hard constraints

- **One file.** All HTML, CSS and JS inline in `index.html`. No build step.
- **Zero dependencies.** No CDN scripts, no frameworks. Must work fully offline.
- **Plain text only.** No rendered preview and no syntax highlighting. The editor is a native `<textarea>` so native undo, selection, autocorrect and spellcheck all keep working (especially on iOS).
- Edits made by helpers (shortcuts, smart lists, replace) go through `document.execCommand('insertText')` so they stay on the native undo stack. Fall back to `setRangeText` only when that fails.
- The markdown flavour is **Pandoc** (`^sup^`, `::: div` fences, YAML front matter, `[@cite]`).

## Storage (both kinds)

1. **Real files on disk**
   - Open: `showOpenFilePicker` where supported (Chrome/Edge), otherwise `<input type=file>`. Dragging a file onto the window also opens it.
   - Save: in Chrome/Edge, write back to the same file handle, with Save As for a new file. Elsewhere (Safari, Firefox, iOS), use the share sheet on touch devices when `navigator.canShare({files})` works. Otherwise use a blob download.
   - Default filename: the slugified front-matter `title` plus `.md` (e.g. `My Essay` → `my-essay.md`). If there's no title, use `untitled.md`. A file opened from disk keeps its own name.
2. **Browser autosave** (`localStorage`). This holds **one working document**, not a library.
   - The current text is autosaved (debounced) under `mded.autosave`.
   - **Launch always shows the scaffold.** If the previous session's autosave holds real work, it is copied to `mded.previous` and a "Restore previous session" banner/menu item is offered.
   - The browser warns before unload and before New/Open when the text has changed since it was last opened or saved to disk.

## Default scaffold

The scaffold loads on launch and on New. `date` is filled in automatically with today's date in this format: `2^nd^ September 2026`, `14^th^ July 2000` (ordinal suffix in Pandoc superscript, full month name, 4-digit year). The cursor lands right after `title: `.

```
---
title: 
date: 17^th^ October 2026
---




# References

::: References

:::
```

## Editor helpers

- **Formatting shortcuts** (Cmd on Mac, Ctrl elsewhere). Each one wraps the selection, and toggles off if the selection is already wrapped:
  - Cmd+B `**bold**`
  - Cmd+I `*italic*`
  - Cmd+K `[text](url)`
  - Cmd+E `` `code` ``
  - The same actions are on toolbar buttons, so phones can use them too.
- **Smart lists:** Enter continues `- `, `* `, `+ `, `1. ` (incremented), `- [ ] ` and `> `. Enter on an empty item ends the list. Tab/Shift+Tab indent/outdent the current line(s) by 4 spaces.
- **Find & replace:** Cmd+F opens it. It has next/previous, replace, replace all, a case-sensitive toggle and a match count. Matches are highlighted by a mirror layer behind the transparent textarea.
- **Other shortcuts:** Cmd+S save, Cmd+Shift+S save as, Cmd+O open.

## Look & feel

- Light/dark follows the system (`prefers-color-scheme`). There is no manual toggle.
- **Fonts** (switchable in Settings; the choice is remembered in `mded.settings`):
  - Default: system monospace (`ui-monospace, SF Mono, Menlo, Consolas, …`).
  - Proportional: `"Equity A"` (if installed locally) → `equity_a` (web font from the site, see below) → Charter → Georgia → serif.
- **Font size** can be adjusted for both fonts.
- **Text column** is centred, defaults to `90ch`, and its width can be adjusted in Settings. On phones it is the full width minus a 16px gutter.
- On touch devices the textarea font-size never drops below 16px, because iOS zooms on focus otherwise.

### Equity web font (served by the website; ignored gracefully under `file://`)

```css
@font-face { font-family: equity_a; font-style: normal; font-weight: normal; font-display: auto;
  src: url('/src/font/equity_a_regular.woff2') format('woff2'); }
@font-face { font-family: equity_a; font-style: italic; font-weight: normal; font-display: auto;
  src: url('/src/font/equity_a_italic.woff2') format('woff2'); }
@font-face { font-family: equity_a; font-style: normal; font-weight: bold; font-display: auto;
  src: url('/src/font/equity_a_bold.woff2') format('woff2'); }
@font-face { font-family: equity_a; font-style: italic; font-weight: bold; font-display: auto;
  src: url('/src/font/equity_a_bold_italic.woff2') format('woff2'); }
```

## Layout notes

- The page (body) scrolls, not the textarea. The textarea and a mirror `<div>` share one CSS grid cell. The mirror sets the height, so the textarea grows with its content without any JS measuring. This also makes find highlighting line up exactly.
- The header is sticky and has: a file menu (New, Open, Save, Save As, Restore previous, Settings), the filename with an unsaved dot, and buttons for B / I / link / code / find.
- Testing: check at desktop width and at 375px (phone) in light and dark.
