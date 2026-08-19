# Handoff — Issue 17: Form validation errors are not announced to screen readers

- GitHub issue: #17
- Status: Ready for review
- Branch: `fix/17-form-validation-errors-are-not-announced-to-screen`
- Latest commit: `2b06bf6` — "fix: announce validation errors to screen readers (#17)"
- Updated: 2026-08-19 00:10 UTC

## Objective

Validation errors on the login (`First-file.html`) and forgot-password (`forgot-password.html`)
forms were shown only visually, via a `<span class="error">` toggled with a CSS class. No
`aria-*` attribute or `role` existed anywhere on either page, so a screen reader user got no
indication which field failed validation or why, and no announcement of the forgot-password
success message.

## Current state

Fix implemented and committed on the issue branch. Both HTML files now carry the ARIA wiring the
issue's "Suggested direction" described. No backend/schema/build changes were needed or made —
this repo has no application code behind these two static pages (see `docs/memory/module-map.md`).

## Changes made

- `First-file.html`
  - `#username` and `#password` inputs: added `aria-describedby` pointing at their existing error
    span IDs, and a static `aria-invalid="false"` default.
  - `#username-error` and `#password-error` spans: added `role="alert"`.
  - Submit handler: added `setAttribute('aria-invalid', 'true'|'false')` calls alongside each
    existing `classList.add/remove('invalid')` call, for both fields.
- `forgot-password.html`
  - `#email` input: same `aria-describedby`/`aria-invalid="false"` treatment.
  - `#email-error` span: added `role="alert"`.
  - `#success-message` div: added `role="status"` (polite live region, since it's a success
    message, not an error).
  - Submit handler: added `setAttribute('aria-invalid', ...)` on the email input alongside the
    existing class toggles.

No validation rules, visual styling, or DOM structure otherwise changed.

## Important decisions

- Used implicit ARIA live regions (`role="alert"` for errors, `role="status"` for the success
  banner) instead of explicit `aria-live` attributes — fewer attributes, standard pattern, same
  effect.
- Kept `aria-describedby` static in markup rather than toggling it via JS, since the referenced
  span's `id` never changes and only its visibility does — a screen reader only exposes the
  referenced text once it becomes visible (not `display:none`).
- Set `aria-invalid` via `setAttribute` (value flips `"true"`/`"false"`) rather than
  add/removeAttribute, since `aria-invalid` is a tristate/string ARIA property, not a boolean
  presence attribute.

## Files affected

- `First-file.html`
- `forgot-password.html`
- `docs/tasks/issue-17.md` (new)
- `docs/handoffs/issue-17.md` (new, this file)

## Tests and checks

| Command or check | Result | Notes |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists in this repo — see KP-0001. Declared commands in `CLAUDE.md` have nothing behind them. |
| Manual full read-through of both edited HTML files | Pass | Confirmed `aria-describedby` values match existing error-span `id`s, ARIA role values are valid, and every branch of both submit handlers sets `aria-invalid` consistently with the existing `.invalid` class toggle. |
| Local script-based HTML structural check | Not run | Attempted to run a small Python tag-balance check; `python3` (and `gh`) invocations were denied by this session's Bash tool-approval gate, with no interactive approver available in this run. Only `git` subcommands went through. |
| Real screen reader test (NVDA / VoiceOver / Narrator) | Not run | No assistive technology or browser is available in this sandbox. This is the literal repro step in the issue and it was **not** performed — flagging this explicitly rather than implying it passed. |

**Honest summary**: the fix follows the standard, well-established ARIA pattern the issue itself
suggested (`aria-describedby` + `aria-invalid` + `role="alert"`/`role="status"`), and the resulting
markup was verified by careful manual reading, not by an automated test suite or a real AT. Residual
risk is low because the pattern is simple and standard, but it has not been confirmed with an actual
screen reader.

## Durable memory changes

None. Evaluated against the promotion test in
`.claude/skills/update-project-memory/SKILL.md`: this is a one-off, page-level accessibility fix
with no new reusable pattern, module contract, or confirmed recurring problem. The Bash
tool-approval gate blocking `gh`/`python3` calls in this run is noted here as a blocker for this
run specifically, but was only observed once in one session, so it does not meet the "verified"
bar for `known-problems.md` yet. If a future run hits the same restriction, that would be the
point to promote it as a confirmed, recurring KP.

Memory impact: none.

## Remaining work

- A human/tester should confirm with a real screen reader that:
  - Tabbing to `#username`/`#password`/`#email` with an active error announces the error text.
  - `aria-invalid="true"` is reported by the AT when a field is invalid.
  - The forgot-password success message is announced when it appears.
- Could not post the triage comment to GitHub issue #17 due to the `gh` CLI being blocked in this
  session (see Blockers below). Consider posting the triage content from
  `docs/tasks/issue-17.md`/this handoff manually, or re-running the comment step in an environment
  where `gh` calls are approved.

## Blockers

- `gh` CLI calls (including read-only `gh auth status`) and `python3` invocations were denied by
  the session's Bash tool-approval gate, with no interactive approver present in this run. Only
  `git` subcommands succeeded. This prevented posting the required triage comment to the GitHub
  issue and running a local static-HTML sanity script. Git-based commit/push operations were not
  affected and completed normally.

## Known risks

- The ARIA wiring has not been confirmed with a real screen reader (see Remaining work). The
  pattern used is standard, but "should work" is not the same as "verified working."
- `aria-describedby` is applied statically even while the error span is hidden via `display:none`.
  This is standard, spec-conformant behavior (hidden elements referenced by `aria-describedby` are
  simply not read until they become part of the accessibility tree), but it was not tested against
  a specific AT/browser combination in this run.

## Rollback

Revert commit `2b06bf6` (the ARIA-attribute fix) on this branch, made after the `550ab7d`
task-checkpoint commit. The branch is deleted on merge per
`branch-approve-merge.yml`, so if this has already merged to `main`, revert the merge commit on
`main` instead — do not attempt to recreate the deleted branch.

## Next executable action

A human reviewer should test both pages with a real screen reader (or add the `branch-approved`
label to trigger merge if the code review alone is sufficient), and someone with `gh` access should
post the triage summary from `docs/tasks/issue-17.md` to issue #17 if a comment is still wanted.
