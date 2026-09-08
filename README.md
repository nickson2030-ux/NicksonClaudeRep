# NicksonClaudeRep

**UOB IT PMO — Project Task Board**

A single-page IT project management app with a Kanban board, built as an internal demo/training
tool. Vanilla HTML, CSS and JavaScript in one file — no framework, no build step, no dependencies.

> This is a demo/training tool, not an official system. It uses a neutral text wordmark and a
> generic corporate blue palette — no UOB logo, trademarks, or imitation of any real UOB system.

![The board with its eight seeded demo tasks across the Backlog, In Progress, Blocked and Done
columns, with the Add Task form on the left](docs/screenshot.png)

## Live demo

**<https://nickson2030-ux.github.io/NicksonClaudeRep/>** — published from `main` by the
[Pages workflow](.github/workflows/deploy-pages.yml). The page is the repo root, served as-is.

## Run it

It also runs straight off the filesystem. Open [`index.html`](index.html) directly in a browser —
double-click it, or:

```powershell
Start-Process index.html
```

No server, no install, no build.

## Features

- Four-column Kanban board: Backlog, In Progress, Blocked, Done — with live count badges
- Drag and drop between columns (native HTML5 DnD), plus a keyboard-accessible `Move ▸` fallback
- Cards show task ID, title, project/workstream, assignee, priority pill, due date and category,
  colour-coded by priority with an `Overdue` badge where applicable
- Add Task form with inline client-side validation (no `alert()`) and an inline delete confirmation
  (no native `confirm()`)
- Client-side filtering by project, assignee and priority; live summary strip in the header
- New tasks are emailed via the [FormSubmit](https://formsubmit.co) AJAX endpoint, sent optimistically
  so a network failure never breaks the board

## No persistence — by design

Board state lives in a JavaScript array in memory. There is no `localStorage`, `sessionStorage`,
IndexedDB, or cookie use anywhere. **Refreshing the page resets the board to its eight seeded demo
tasks.** This is intended behaviour for a training tool, and the UI says so.

## Configuration

The notification recipient is set in one place, at the top of the `<script>` block in `index.html`:

```js
const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
```

FormSubmit requires a **one-time activation**: the first submission sends a confirmation email to
that address, and nothing is delivered until the link inside it is clicked.

## Development notes

See [CLAUDE.md](CLAUDE.md) for the architecture and the constraints the code is built around.
