# NicksonClaudeRep

**UOB IT PMO — Project Task Board**

A single-page IT project management app with a Kanban board, built as an internal demo/training
tool. Vanilla HTML, CSS and JavaScript in one file — no framework, no build step, no dependencies.

> This is a demo/training tool, not an official system. It uses a neutral text wordmark and a
> generic corporate blue palette — no UOB logo, trademarks, or imitation of any real UOB system.

![The purple-themed board with its eight seeded demo tasks across the Backlog, In Progress, Blocked
and Done columns — each column topped by a coloured rail and status dot — with the Add Task form in
a sidebar on the left](docs/screenshot.png)

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

## Client-side hardening

There is no backend, so none of this is a substitute for server-side validation — it is what a
single static file can actually enforce:

- **Content Security Policy** via `<meta>`. `connect-src` pins outbound requests to the FormSubmit
  endpoint, so injected script has nowhere to exfiltrate to; `form-action`, `base-uri` and
  `object-src` are locked off. `'unsafe-inline'` is unavoidable here (the app *is* one inline
  block) and is deliberately accepted — the containment above is what the policy buys.
- **Output escaping.** `escapeHtml()` wraps every user string interpolated into HTML, attribute
  values included.
- **Input sanitising.** Control characters and Unicode bidi overrides are stripped from free text,
  so a title cannot be made to read differently from what it contains.
- **Allow-listed selects.** Project, category, priority and status are re-checked against their
  lists rather than trusted because the markup offered no other option.
- **Drag payloads are origin-checked.** Drops carry a private MIME type cross-checked against the
  id recorded on `dragstart`, so text dragged in from another tab is ignored.
- **Bounded network call.** The FormSubmit request aborts after 10s, sends no cookies and no
  referrer, and its response body is never read back into the page. Submissions are rate-limited.

Two risks are known and accepted: the FormSubmit endpoint is public and unauthenticated (anyone
reading the source can POST to it — not fixable client-side), and `frame-ancestors` is ignored in a
meta CSP, so clickjacking cannot be blocked from inside the file. Neither matters much for a board
with no auth, no privileged actions and no real data.

## Configuration

The notification recipient is set in one place, at the top of the `<script>` block in `index.html`:

```js
const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
```

FormSubmit requires a **one-time activation**: the first submission sends a confirmation email to
that address, and nothing is delivered until the link inside it is clicked.

## Development notes

See [CLAUDE.md](CLAUDE.md) for the architecture and the constraints the code is built around.
