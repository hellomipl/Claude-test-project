# Handoff — Issue 16: Login form leaks password into the URL on successful submit

- GitHub issue: #16
- Status: Ready for review
- Branch: `fix/16-login-form-leaks-password-into-the-url-on-successf`
- Latest commit: (see `git log -1` on this branch after this handoff is pushed)
- Updated: 2026-08-19 00:30 UTC

## Objective

On a successful login submit, `First-file.html`'s submit handler did not call
`e.preventDefault()` unless validation failed, so the browser performed a native GET submit and
leaked `username`/`password` into the URL query string.

## Current state

Fix implemented and pushed. `e.preventDefault()` is now the first statement in the `#login-form`
submit handler, so the page never navigates on submit, regardless of validation outcome. This
mirrors the existing correct pattern in `forgot-password.html`.

## Changes made

- `First-file.html`: added `e.preventDefault();` as the first statement inside the `submit` event
  listener for `#login-form` (before the `valid` check begins). No other lines changed — the
  pre-existing `if (!valid) { e.preventDefault(); }` block was left in place; it is now a harmless
  no-op duplicate call, not a bug. Removing it would have left an unused `valid`-tracking flow
  cleanup outside the scope of this bug, so it was deliberately left untouched.
- `docs/tasks/issue-16.md`: created as the run checkpoint, then updated with final results.
- `docs/memory/known-problems.md`: added KP-0005 (see below).
- `docs/memory/coding-patterns.md`: added a pattern entry (see below).

## Important decisions

- Chose to move `e.preventDefault()` to the top of the handler rather than adding
  `method="post"` / `action` to the form. Adding a method/action would still leak the password (POST
  bodies are safer than GET query strings but this is a static page with no real endpoint to post
  to), and it's a larger, less targeted change than the one-line fix that matches the codebase's own
  existing correct pattern in `forgot-password.html`.
- Left the now-redundant `if (!valid) { e.preventDefault(); }` block untouched rather than removing
  it, to keep the diff to the single line required to fix the bug (see CLAUDE.md "Prefer minimal
  fixes"). Removing it would have surfaced an unused `valid` variable, pulling in unrelated cleanup.

## Files affected

- `First-file.html`
- `docs/tasks/issue-16.md`
- `docs/memory/known-problems.md`
- `docs/memory/coding-patterns.md`
- `docs/handoffs/issue-16.md` (this file)

## Tests and checks

| Command or check | Result | Notes |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists in the repo — see KP-0001. The declared commands in `CLAUDE.md` have nothing behind them. |
| Manual code read-through of the fixed submit handler | Pass | Confirmed `e.preventDefault()` now executes unconditionally as the first statement, before any validation logic, for every submit event. The form has no `action`/`method` and no other listeners that could trigger native submission. |
| Live browser interaction (steps to reproduce from the issue) | Not run | No browser/display available in this non-interactive session. This is the one gap in verification — a human should open `First-file.html`, submit valid credentials, and confirm the URL does not change, before merging. |
| `gh issue comment` (triage comment) | Failed | Blocked by the harness's approval gate with no interactive user to approve, even with sandbox checks disabled. See KP-0005. The triage analysis that would have been posted is preserved in `docs/tasks/issue-16.md` under Findings. |

## Durable memory changes

- `docs/memory/known-problems.md`: added **KP-0005** — `gh` CLI commands are blocked by the approval
  gate in an unattended run; no `.claude/settings.json` allow-rule exists to unblock them. Affects
  any future run that needs to shell out to `gh` (e.g. posting issue comments).
- `docs/memory/coding-patterns.md`: added **"Prevent native submission unconditionally in a form's
  submit handler"** — call `e.preventDefault()` as the first statement, never gated behind a
  validity check, for any standalone HTML page's client-side-only form handler.

## Remaining work

- None on this branch. A human reviewer should exercise the login form in a real browser (submit
  valid credentials, confirm the address bar doesn't change) before applying `branch-approved`,
  since that step could not be performed in this session.

## Blockers

- Could not post the triage comment on issue #16 via `gh issue comment` — see KP-0005 and the Tests
  and checks table above. The intended comment content is preserved in `docs/tasks/issue-16.md`
  under "Findings" and can be posted manually or by a future run once KP-0005 is addressed.

## Known risks

- Low. The change is a single added line, behavior-only, no markup/style/API/schema changes, and
  directly mirrors an existing correct pattern already in the repo. The only residual risk is the
  lack of live-browser confirmation (see Remaining work).

## Rollback

Revert commit(s) on this branch that touch `First-file.html` (see `git log -- First-file.html` on
`fix/16-login-form-leaks-password-into-the-url-on-successf`), or simply re-remove the added
`e.preventDefault();` line at the top of the `#login-form` submit listener. The branch is deleted on
merge per `branch-approve-merge.yml`, so if this needs reverting after merge, revert the merge commit
on `main` instead.

## Next executable action

A human reviewer opens `First-file.html` in a browser, submits valid credentials, confirms the URL
stays clean, then applies the `branch-approved` label to trigger the merge.
