# Issue 16 — Login form leaks password into the URL on successful submit

- GitHub issue: #16
- Status: Ready for review
- Branch: `fix/16-login-form-leaks-password-into-the-url-on-successf`
- Last updated: 2026-08-19 00:30 UTC

## Objective

On a successful login submit, `First-file.html` does not call `e.preventDefault()`, so the browser
performs a native GET submit and appends `username`/`password` to the URL query string, leaking the
password into the address bar, browser history, and (in a real deployment) access logs/Referer
headers.

## Scope

### In scope

- `First-file.html` — the `submit` listener on `#login-form`.

### Out of scope

- Any backend/API work (none exists; this is a static demo page).
- Validation logic or UI changes beyond preventing default navigation.
- `forgot-password.html` (already correct; used as the reference pattern).

## Acceptance criteria

- [x] Submitting the login form with valid username/password does not navigate the page or add
      `username`/`password` to the URL, in any code path (valid or invalid). Verified by code
      read-through (see Validation results); not exercised in a live browser — see handoff.

## Assumptions

- Verified: This is a static demo page with no backend, per the issue's own "Severity" note and
  confirmed by `docs/memory/module-map.md` ("Static pages ... none (no build step)").
- Verified: `forgot-password.html` calls `e.preventDefault()` unconditionally as the first statement
  of its submit handler (line 158), confirmed by reading the file directly.

## Blockers / questions

- `gh` CLI commands (`gh issue comment`, `gh auth status`) are blocked by the harness's approval
  gate in this session, with no interactive user available to approve. Could not post the triage
  comment on issue #16 via CLI. Proceeding with the branch work regardless, since git operations are
  unaffected. Recorded in the handoff.

## Context loaded

- [x] `CLAUDE.md` and `.claude/rules/`
- [x] `docs/memory/project-memory.md` and `module-map.md`
- [x] `docs/memory/known-problems.md` and `failed-approaches.md`
- [x] Previous task/handoff for this issue — none existed prior to this run.

## Implementation checklist

- [x] Inspect existing code before editing
- [x] Implement the smallest safe fix
- [x] Add or update tests where a harness exists (none exists — see KP-0001; no test added)
- [x] Evaluate durable-memory impact
- [x] Update the handoff
- [x] Commit and push to the issue branch

## Affected files

- `First-file.html`

## Findings

- Root cause confirmed: the submit handler in `First-file.html` (lines 137–165) only calls
  `e.preventDefault()` inside the `if (!valid)` branch (line 163). When validation passes, nothing
  stops the native form submission. The `<form id="login-form">` (line 119) has no `action` and no
  `method`, so the browser defaults to a GET against the current page URL with all named fields
  serialized as query params.
- `forgot-password.html` (line 158) calls `e.preventDefault()` unconditionally as the very first
  statement in its handler — this is the correct, existing pattern in this codebase to follow.

## Decisions made

- Fix by moving `e.preventDefault()` to the top of the login form's submit handler, matching
  `forgot-password.html`'s pattern exactly, rather than adding a `method`/`action` to the form (which
  would still leak the password via GET) or introducing other structural changes. This is the
  smallest change that fully closes the leak for both the valid and invalid paths.

## Validation results

| Command or check | Result | Evidence |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists — see KP-0001. Declared commands have nothing behind them. |
| Manual code read-through of `First-file.html` after fix | Verified | Confirmed `e.preventDefault()` is now the first statement in the submit handler, executed on every submit regardless of `valid`, and no other code path can trigger native submission (form has no `action`/`method`, no other event listeners). |
| Browser interaction | Not run | No browser/display available in this non-interactive session. Static-analysis / code read-through was used instead; see handoff for residual risk. |

## Durable memory impact

- Promoted two items (both pass the five-point promotion test in
  `.claude/skills/update-project-memory/SKILL.md`):
  1. `docs/memory/known-problems.md` KP-0005 — `gh` CLI commands are blocked by the approval gate
     in an unattended run, with no `.claude/settings.json` allow-rule to unblock them. Confirmed by
     direct tool failures this run; reusable for any future run that needs to call `gh`.
  2. `docs/memory/coding-patterns.md` — "Prevent native submission unconditionally in a form's
     submit handler." Reusable convention for any future form added to these static HTML pages.

## Next executable action

None — fix implemented and verified by code read-through; task and handoff written. A human
reviewer should exercise the login form in an actual browser before applying `branch-approved`,
since no browser was available in this session to confirm the fix visually.
