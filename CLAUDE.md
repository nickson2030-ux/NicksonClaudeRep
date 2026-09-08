# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Kanban board for a fictional "UOB IT PMO" — an internal demo/training tool, not a real system.
The entire project is one file: [index.html](index.html) (~52 KB). There is no package.json, no
git repo, no dependencies, and no test suite.

It is deliberately branding-neutral: a text wordmark only, generic corporate blue. Do not add a UOB
logo, trademark, or anything that imitates an official UOB system.

## Running it

Open the file directly — `Start-Process index.html` — or double-click it. No server, no build step,
no install. There is nothing to compile, lint, or test; verification is manual in the browser.

Manual checks worth re-running after any change: drag a card between columns; move a card using only
the keyboard (`Move ▸` button); submit the form empty and with a past due date; add a task titled
`<img src=x onerror=alert(1)>` and confirm it renders as literal text; refresh and confirm the board
resets to the 8 seeded tasks.

## Hard constraints

These are requirements of the project, not stylistic preferences. Violating any of them breaks the
deliverable:

- **Vanilla only.** No framework, library, bundler, npm, or build step.
- **One file.** All markup, one `<style>` block, one `<script>` block in `index.html`.
- **Zero external resources.** No CDN scripts, web fonts, or image files — system font stack and
  inline SVG / Unicode glyphs only. The FormSubmit endpoint is the only URL in the file.
- **No persistence.** No `localStorage`, `sessionStorage`, IndexedDB, or cookies. Board state is a
  JavaScript array that resets on refresh; the header says so explicitly. This is intended.
- **No `alert()`, `confirm()`, or `prompt()`.** Validation errors are inline `.field-error` text;
  delete uses an inline `Delete? Yes / No` row inside the card.
- **No `!important`** in the CSS.

## Architecture

Everything lives inside one IIFE with banner comments marking each section (`RENDER — a card`,
`DRAG & DROP`, `SEED DATA`, etc.) — grep for those banners rather than relying on line numbers.

**`state` is the single source of truth.** It holds `tasks`, `filters`, `nextId`, and — importantly
— the *transient* UI flags `movingId` (which card's move menu is open), `confirmingDelete` (which
card is showing its delete confirmation), `dragTaskId`, and `pendingFocus`. Transient per-card UI is
kept in state rather than toggled in the DOM so that a full re-render reproduces it correctly.

**`renderBoard()` is the only function that writes card HTML.** It clears each column and rebuilds
from `applyFilters()`. Nothing else should mutate card contents. Three consequences to respect when
extending this:

1. **Events are delegated.** A single `click` listener on `.board` reads `data-action` / `data-id`
   attributes. Never attach per-card listeners — they would be destroyed on the next render.
   Add new card interactions by emitting a `data-action` in `renderCard()` and adding a `case` to
   `handleBoardClick()`.
2. **Focus must be restored manually.** Re-rendering destroys the focused element. Set
   `state.pendingFocus = { id, action }` before calling `renderBoard()`; `restoreFocus()` re-focuses
   the matching `[data-action][data-id]` button afterwards. `moveTask()` takes a third `keepFocus`
   argument — the keyboard path passes `true`, the drag path deliberately omits it so dropping a
   card doesn't scroll the page.
3. **`state.tasks` changes only through `addTask()`, `moveTask()`, `deleteTask()`, and
   `seedTasks()`.** Route new mutations through a named function that ends in `renderBoard()`.

**Escaping.** `escapeHtml()` must wrap every user-supplied string interpolated into HTML, including
inside attribute values (it escapes quotes for that reason). `renderCard()` is the only place this
matters today.

**Drag and drop** uses the native HTML5 API. The drop-target highlight is driven entirely from
`dragover` — cleared then reapplied — because `dragleave` fires spuriously when the pointer crosses
child elements. Don't reintroduce a `dragleave` handler.

**Dates** are ISO `YYYY-MM-DD` strings compared as plain strings (lexicographic order matches
chronological order), so there is no date library. `todayISO()` builds today's date from local
date parts, not `toISOString()`, to avoid a UTC off-by-one.

## FormSubmit

`FORMSUBMIT_ENDPOINT` sits under the `CONFIG` banner at the top of the script — the only place the
recipient address appears, and the only place it should ever be sent. It uses the AJAX JSON endpoint
so the page never navigates away.

FormSubmit requires a **one-time activation**: the first submission triggers a confirmation email to
the recipient, and nothing is delivered until that link is clicked. Changing the address resets this.

The call is fire-and-forget and wrapped in `try/catch`: `addTask()` puts the card on the board before
the request starts, and a network failure produces a warning toast only. A FormSubmit failure must
never block or break the board.
