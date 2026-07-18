# UI Modernization Design — deskmatee

**Date:** 2026-07-18
**Status:** Approved
**Approach:** A — Incremental refactor in place

## Goal

Modernize the warm "filing cabinet" aesthetic of deskmatee's frontend, make it
fluidly responsive across desktop/tablet/phone, redesign file rows as icon-rich
cards, and add automatic dark mode — with **zero new dependencies** and **no
changes to JS application logic** (Tauri commands, AI chat, sharing, preview all
stay intact).

## Constraints

- No new npm/Rust dependencies.
- No framework (Tailwind etc.) — plain CSS in a separate `styles.css`.
- All existing JavaScript behavior must remain unchanged. Only class names and
  CSS change.
- Local-first: no external network requests (SVG icons inline, not from a CDN).

## Decisions (from brainstorming)

1. **Direction:** Modernize current style (keep warm palette, polish it).
2. **Dark mode:** Auto via `prefers-color-scheme` (no manual toggle).
3. **CSS structure:** Separate `src/styles.css` imported by Vite.
4. **Priority areas:** Fluid responsiveness + File list redesign.
5. **Icons:** Inline SVG (no emoji, no icon library) using `currentColor`.

## File structure

| File | Change |
|------|--------|
| `src/styles.css` | **NEW** — all CSS extracted from the `<style>` block, plus theming, responsive, and card styles. |
| `src/index.html` | Remove `<style>` block; add `<link rel="stylesheet" href="/src/styles.css">`; convert inline `style="..."` to semantic classes; add SVG icon sprite; rename `.frow` → `.file-card`; add empty/loading states. |
| `src-tauri/src/lib.rs` | No change. |

## Design detail

### 1. Theming layer

- Keep all existing light-mode variables (`--paper`, `--paper-deep`, `--manila`,
  `--manila-dark`, `--ink`, `--ink-soft`, `--rust`, `--forest`, `--line`,
  `--card`, `--shadow`) exactly as they are.
- Add a `@media (prefers-color-scheme: dark)` block overriding those variables
  with dark, readable variants (e.g. `--paper: #1c241e`, `--card: #252e27`,
  `--ink: #e8e0cf`, `--ink-soft: #a9b3a3`, `--line: #3a463c`).
- Soften the hard offset shadows (`3px 4px 0 rgba(...)`, `3px 3px 0 var(--rust)`)
  into blurred equivalents (e.g. `0 4px 12px rgba(...)`) while keeping a tactile
  feel. `--shadow` variable is updated accordingly.
- SVG icons use `fill="currentColor"` so they adapt to the theme automatically.

### 2. Responsive (mobile-first, fluid)

- Base styles target phone (~360px); scale up via `min-width` media queries
  (replacing the current `max-width` approach).
- Use `clamp()` for fluid type and spacing (e.g. `h1: clamp(28px, 5vw, 44px)`)
  so sizes scale smoothly instead of jumping at breakpoints.
- Three tiers:
  - **Phone (<600px):** single-column; file cards stack full-width; metadata
    wraps below the name (no column hiding).
  - **Tablet (600–899px):** category cabinet becomes a horizontal scrollable
    bar at the top.
  - **Desktop (≥900px):** two-column `.stage` (cabinet + main) as today.
- Update AI sidebar and share panel responsive rules to match the new tiers.

### 3. File list → cards

- Rename `.frow` → `.file-card`. Update `renderFileList()` markup and all CSS
  selectors that reference `.frow`.
- Card contents:
  - SVG **type icon** (from the icon set, by category)
  - File name + relative path
  - A **wrapping metadata row**: category/tag · date · size
  - **Hover actions**: "Preview" and "Open" buttons appear on hover (desktop),
    always visible on touch devices
  - Duplicate files keep the rust left-border accent (`.file-card.dupe`)
- **Empty state:** friendly centered message with an icon.
- **Loading state:** skeleton shimmer while scanning (replaces the plain
  "Scanning…" text) via a `.loading` class.
- Keep `handleFileClick(i)` behavior; move `cursor:pointer` from inline style
  into `.file-card` CSS.

### 4. Inline SVG icon set

- A hidden `<svg>` sprite with `<symbol>` definitions for: pdf, doc, sheet,
  slides, image, video, audio, archive, code, installer, other (11 icons).
- A helper `fileIcon(category)` returns the `<svg><use href="#icon-..."></svg>`
  markup so cards render the right glyph.
- Icons use `fill="currentColor"` (or `stroke` where appropriate) so they follow
  the active theme.

### 5. Inline-style cleanup

- Convert the 25+ inline `style="..."` attributes to semantic classes:
  `.hidden`, `.flex-gap`, `.share-start-btn`, etc.
- Remove inline `cursor:pointer` from generated file rows.
- Behavior must stay identical.

## Out of scope

- No CSS framework, no motion/micro-interaction library, no component-system
  rewrite, no changes to backend or app behavior.

## Verification

- `npx vite build` succeeds.
- `cargo check` (no Rust change, sanity only).
- Manual smoke test: scan a folder, toggle OS dark mode, resize through the
  three tiers, confirm preview / share / AI chat still function.
- Commit and push.
