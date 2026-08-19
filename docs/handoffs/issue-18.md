# Handoff — Issue 18: No way to create an account from the login page

- GitHub issue: #18
- Status: Ready for review
- Branch: `fix/18-no-way-to-create-an-account-from-the-login-page`
- Latest commit: (see `git log -1` on this branch after the memory/handoff commit)
- Updated: 2026-08-19

## Objective

`First-file.html` offered only "Login" and "Forgot password?" links, so a first-time visitor with
no account had no way to reach account creation.

## Current state

Fix implemented and committed on the issue branch. Ready for human review (`branch-approved`
label triggers merge — this agent did not merge or open a PR).

## Changes made

- Added a new static `register.html` page: same visual pattern as `forgot-password.html`
  (card layout, inline client-side validation, link back to login), with username/email/password/
  confirm-password fields.
- Added a "Create an account" link on `First-file.html`, next to the existing "Forgot password?"
  link, pointing to `register.html`.
- Registered `register.html` in `docs/memory/module-map.md` per `.claude/rules/20-frontend.md`.

## Important decisions

- Named the new page `register.html`, matching the plain descriptive naming already used by
  `forgot-password.html`.
- Reused the existing CSS card pattern verbatim rather than inventing a new style, to keep the
  fix minimal and visually consistent with the rest of the login flow.
- Scoped this as a client-side-only page: no backend/API exists in this repo (confirmed in
  `docs/memory/module-map.md` "Not yet present"), so there is nothing to wire the form up to.
  Treated as an explicit assumption since the issue itself only describes a UI dead end.

## Files affected

- `First-file.html` (modified — added link + CSS class)
- `register.html` (new)
- `docs/memory/module-map.md` (modified — registered new page)
- `docs/tasks/issue-18.md` (new — task checkpoint)
- `docs/handoffs/issue-18.md` (this file)

## Tests and checks

| Command or check | Result | Notes |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` in repo — KP-0001 |
| `python3` HTML well-formedness check | Not run | Command denied without approval (same pattern as KP-0006) |
| Manual read-through of `register.html` and `First-file.html` diff | Pass | Checked markup, CSS, and inline JS validation logic end to end |
| `grep` cross-check of all three page links | Pass | `First-file.html <-> forgot-password.html` and `First-file.html <-> register.html` all resolve to files present at repo root |

No automated or browser-rendered verification was possible in this environment. A human should
open `register.html` (and the updated `First-file.html`) in a browser before approving, to confirm
actual rendering and form-validation behavior.

## Durable memory changes

- `docs/memory/module-map.md`: added `register.html` to the "Static pages" row. This is required
  bookkeeping per `.claude/rules/20-frontend.md`, not a new insight.
- No other memory file changed. The `gh issue comment` denial reproduced again but matches
  KP-0006 exactly (already recorded from issues #16/#17) — no new information to promote.

## Remaining work

- None required for this bug. Optional follow-up (not requested by the issue, not implemented):
  wiring `register.html` to a real account-creation API once a backend exists.

## Blockers

- Could not post the triage comment to issue #18 (`gh issue comment` denied — KP-0006). The
  analysis that would have been posted is captured in `docs/tasks/issue-18.md` under "Findings"
  and in this handoff instead.

## Known risks

- The registration form is presentation-only (no backend). If a reviewer expects a functioning
  signup flow rather than a UI entry point, that would need a separate, larger change — flagging
  this explicitly since the issue text doesn't rule it out.
- No browser-based visual verification was performed (see Tests and checks).

## Rollback

To undo this fix, revert commits on `fix/18-no-way-to-create-an-account-from-the-login-page` that
touch `First-file.html`, `register.html`, and `docs/memory/module-map.md`, or delete `register.html`
and revert the link addition in `First-file.html`. The branch is deleted on merge, so if this is
merged and needs reverting later, revert against `main` instead.

## Next executable action

Human reviewer: open `First-file.html` and `register.html` in a browser, confirm the "Create an
account" link and form validation behave as expected, then apply the `branch-approved` label to
trigger the merge workflow.
