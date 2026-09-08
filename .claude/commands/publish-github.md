---
description: Publish this repo to GitHub — secret scan, push, Pages via Actions, README, and repo About/homepage
argument-hint: "[repo URL or owner/name] (optional — omit to use the existing origin)"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Publish to GitHub

Target repo (may be empty): $ARGUMENTS

Run steps 1–5 in order. Step 1 gates everything else: nothing is pushed until the secret scan is
clean. Report progress as you go and finish with the checklist in "Final report".

## Ground rules

- **Never push before the secret scan passes.** If it flags anything, stop and ask the user before
  going further. Do not rewrite git history to bury a secret without explicit instruction — a
  leaked credential must be rotated, not just deleted.
- **Never set git identity globally.** This repo sets `user.name` / `user.email` locally and uses a
  GitHub `@users.noreply.github.com` address on purpose. Preserve that; never replace it with the
  user's real email.
- **Do not invent a repo.** If `$ARGUMENTS` is empty and `git remote -v` has no `origin`, ask the
  user for the URL and stop.
- **`gh` is probably not installed here** (it was not at the time this command was written), and
  `git` only exists inside the Bash tool, not PowerShell. Check before relying on either, and use
  the documented fallbacks rather than failing.
- Respect the repo's hard constraints in [CLAUDE.md](../../CLAUDE.md) — one vanilla file, no
  dependencies, no UOB branding. Publishing must not change the app.

## Step 1 — Scan for sensitive data (gate)

Scan everything that would end up public: tracked files, staged changes, and any untracked file you
are about to add. Search for at least:

- Credentials: `ghp_`, `github_pat_`, `gho_`, `sk-`, `AKIA`, `ASIA`, `xox[baprs]-`, `AIza`,
  `-----BEGIN [A-Z ]*PRIVATE KEY-----`, long `Bearer` tokens
- Assignments that look like secrets: an `api_key`, `apikey`, `secret`, `token`, `password`,
  `passwd` or `client_secret` name followed by `=` or `:` and a quoted value of 8+ characters
- Connection strings: `mongodb+srv://`, `postgres://user:pass@`, `Server=...;Password=`
- Files that should never be committed: `.env*`, `*.pem`, `*.key`, `*.pfx`, `id_rsa*`,
  `credentials`, `*.sqlite`, cloud config directories
- Personal / internal data: real email addresses other than the noreply one, phone numbers,
  internal hostnames, private IP ranges (`10.`, `192.168.`, `172.16`–`172.31`), employee names or
  ticket IDs that should not be public
- History too, for secret-shaped filenames added in earlier commits:
  `git log --diff-filter=A --name-only --pretty=format: | sort -u`

Known and acceptable in this repo: the FormSubmit endpoint address under the `CONFIG` banner in
`index.html` (a deliberate, public recipient address — flag it only if it looks like a personal
inbox rather than the intended one) and the `@users.noreply.github.com` git email.

Then verify `.gitignore` excludes `.env`, `*.key`, `*.pem`, and local editor/OS cruft; create or
extend it if not.

Report findings as a short table (file, line, what matched, verdict). If anything is a real secret,
**stop here** and tell the user what to rotate.

## Step 2 — Upload the code to GitHub

1. Resolve the target repo: use `$ARGUMENTS` if given, otherwise the existing `origin`. Accept
   either a full URL or `owner/name`; normalise to `https://github.com/<owner>/<name>.git`.
2. If `$ARGUMENTS` names a *different* repo than the current `origin`, show both and ask which one
   wins before changing the remote.
3. Confirm the repo-local identity is set (`git config user.name`, `git config user.email`); if
   missing, set it locally to the GitHub username and its noreply address.
4. Make sure the branch is `main`. Stage, review `git diff --cached --stat`, and commit with a
   message describing the actual change (not "publish"). End the message with:
   `Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>`
5. `git push -u origin main`.
6. If the push fails because the remote repo does not exist: `gh repo create` when `gh` is
   available, otherwise tell the user to create the empty repo at https://github.com/new (no
   README, no .gitignore) and re-run. If it fails on auth, point them at a PAT or Git Credential
   Manager — never echo a token into a command that gets logged.
7. If the remote has commits the local branch lacks, `git pull --rebase` and resolve before pushing.
   Do not force-push without asking.

## Step 3 — Create or update the GitHub Pages action

1. If `.github/workflows/deploy-pages.yml` exists, read it and fix only what is wrong. Otherwise
   create it: trigger on `push` to `main` plus `workflow_dispatch`; permissions `contents: read`,
   `pages: write`, `id-token: write`; concurrency group `pages` with `cancel-in-progress: false`;
   steps `actions/checkout@v4`, `actions/configure-pages@v5`, `actions/upload-pages-artifact@v3`
   with `path: .` (this site is static at the repo root — there is no build step), and
   `actions/deploy-pages@v4`.
2. Ensure a `.nojekyll` file exists at the repo root so Jekyll does not eat files.
3. **First-time enablement is manual** — the default `GITHUB_TOKEN` cannot create a Pages site.
   Tell the user to set Settings → Pages → Source = **GitHub Actions** once, then re-run the
   workflow. If a deploy fails with a Pages-not-enabled error, this is the fix, not a workflow bug.
4. Push the workflow, then verify: `gh run list` if available, otherwise give the user the Actions
   URL and `curl -sI https://<owner>.github.io/<name>/` to confirm a 200 once the run finishes.
   The first publish can take a couple of minutes.
5. Record the live URL — `https://<owner>.github.io/<name>/` — for steps 4 and 5.

## Step 4 — Create or update the README

Read `README.md` first and edit it rather than replacing it wholesale. It should cover, accurately:

- Project name and a one-line description
- The demo/training disclaimer (no UOB branding, not an official system)
- **Live demo** link to the Pages URL from step 3
- How to run locally (open `index.html`; no server, no build, no install)
- What it does, and the constraints a reader cares about: single file, vanilla, no persistence —
  the board resets on refresh

No badges pointing at services the repo does not use, and no claims you have not verified.

## Step 5 — Update the repo About and homepage link

Set the description, the homepage to the Pages URL, and a few topics.

- With `gh`:
  `gh repo edit <owner>/<name> --description "<one line>" --homepage "<pages url>" --add-topic kanban --add-topic vanilla-javascript`
- Without `gh`, if the user supplies a PAT with `repo` scope, use the REST API and read the token
  from an environment variable — never inline it:
  `curl -sS -X PATCH -H "Authorization: Bearer $GITHUB_TOKEN" -H "Accept: application/vnd.github+json" https://api.github.com/repos/<owner>/<name> -d '{"description":"...","homepage":"..."}'`
- Otherwise give exact manual steps: repo page → **About** (gear icon, top right) → Description,
  Website, Topics → Save changes. Mention the "Use your GitHub Pages website" checkbox, which fills
  the Website field with the Pages URL automatically.

Ask before requesting a token — do not assume the user wants to create one for this.

## Final report

End with a checklist and real URLs:

- Secret scan: clean, or the findings
- Pushed: commit SHA and branch
- Pages: workflow run status and live URL
- README: created / updated / unchanged
- About: set via gh, via API, or left to the user (with the steps)

Say plainly which steps completed, which need a manual action from the user, and which were skipped
and why.
