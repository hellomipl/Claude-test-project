# Issue 17 — Form validation errors are not announced to screen readers

- GitHub issue: #17
- Status: Ready for review
- Branch: `fix/17-form-validation-errors-are-not-announced-to-screen`
- Last updated: 2026-08-19 00:10 UTC

## Objective

Validation errors on the login (`First-file.html`) and forgot-password (`forgot-password.html`)
forms are shown only visually (a `<span>` toggled with a CSS class). Screen reader users get no
announcement of which field is invalid or why. Wire up ARIA so errors (and the forgot-password
success message) are programmatically associated with their fields and announced when they appear.

## Scope

### In scope

- `First-file.html`: `#username`/`#username-error`, `#password`/`#password-error`
- `forgot-password.html`: `#email`/`#email-error`, `#success-message`
- Adding `aria-describedby`, `aria-invalid`, and live-region roles (`role="alert"` /
  `role="status"`), and toggling `aria-invalid` in the existing submit-handler JS.

### Out of scope

- Any visual/CSS redesign.
- Any change to validation logic/rules themselves (only the announcement mechanism).
- Any backend, schema, or build tooling changes (none exist for this repo — see KP-0001).

## Acceptance criteria

- [x] `#username-error` and `#password-error` are tied to their inputs via `aria-describedby` and
      announced via a live region when they become visible.
- [x] `#email-error` in `forgot-password.html` has the same treatment.
- [x] `aria-invalid` toggles true/false on each input alongside the existing `.invalid` class logic.
- [x] `#success-message` in `forgot-password.html` is announced via a live region when it appears.

## Assumptions

- `role="alert"` (implicit `aria-live="assertive"`) is appropriate for validation error spans —
  this is the standard WCAG pattern for inline form errors, but is unverified against a real
  screen reader in this environment.
- `role="status"` (implicit `aria-live="polite"`) is appropriate for the success banner since it's
  not an error — same caveat.
- Setting `aria-describedby` statically in markup is safe even though the error span is hidden by
  default (`display:none`), because the association only needs to resolve once the span becomes
  visible; this is standard behavior, not verified with an actual AT in this sandbox.

## Blockers / questions

- Could not post the triage comment to GitHub issue #17: `gh` CLI calls (including read-only
  `gh auth status`) were denied by the session's tool-approval gate with no interactive approver
  available in this run. Triage analysis is recorded here instead.
- No `package.json` / test harness exists (KP-0001), and no browser/AT automation is available in
  this sandbox. Verification will be limited to static review of the resulting HTML/JS.

## Context loaded

- [x] `CLAUDE.md` and `.claude/rules/`
- [x] `docs/memory/project-memory.md` and `module-map.md`
- [x] `docs/memory/known-problems.md` and `failed-approaches.md`
- [x] Previous task/handoff for this issue: none existed before this run

## Implementation checklist

- [x] Inspect existing code before editing
- [x] Implement the smallest safe fix
- [x] Add or update tests where a harness exists — none exists (KP-0001), so N/A
- [x] Evaluate durable-memory impact
- [x] Update the handoff
- [x] Commit and push to the issue branch

## Affected files

- `First-file.html`
- `forgot-password.html`

## Findings

- Neither page contains any `aria-*` attribute or `role` (confirmed by reading both files in
  full). Confirmed evidence for the issue report.
- The error/success visibility toggles are driven entirely by adding/removing a `visible` CSS class
  in the inline `<script>` blocks; there is no framework or component abstraction to update.

## Decisions made

- Used `role="alert"` on error spans and `role="status"` on the success banner (implicit live
  regions) rather than explicit `aria-live` attributes — simpler and standard-conformant, no extra
  attribute needed.
- Set `aria-invalid="false"` as the static default on inputs in the markup, and flip it to
  `"true"`/`"false"` via `setAttribute` in the existing submit handlers, right alongside each
  existing `classList.add('invalid')`/`classList.remove('invalid')` call — keeps the two indicators
  (visual class, ARIA state) always in sync in one place.
- Kept `aria-describedby` static in the markup rather than adding/removing it via JS. The error
  span always exists at the same ID; only its visibility toggles. Screen readers only announce the
  referenced text once it's exposed (not `display:none`), so a static association is simpler and
  has the same effective behavior as toggling the attribute.

## Validation results

| Command or check | Result | Evidence |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists (KP-0001) |
| Manual read-through of both HTML files, full file, post-edit | Done | Confirmed each `aria-describedby` value matches its error span's `id`; `role="alert"`/`role="status"` are valid ARIA roles; JS `setAttribute('aria-invalid', ...)` calls added in both the invalid and valid branches of every validation check, mirroring the existing class-toggle branches exactly |
| HTML tag-balance / structural check via a local script | Not run | `python3` and other non-`git` Bash commands were denied by the session's tool-approval gate (see Blockers) — could not execute even a read-only local script |
| Actual screen reader test (NVDA/VoiceOver/Narrator) | Not run | No AT or browser available in this sandbox; cannot claim this was verified end-to-end. This is the one thing the issue itself asks a fixer to do ("turn on a screen reader") and it was not possible here. |

## Durable memory impact

- Candidates: none. This is a one-off page-level fix with no new reusable pattern, invariant, or
  confirmed recurring problem beyond what's already captured in `known-problems.md`. The `gh`/
  non-git Bash approval blocker encountered in this run is noted in the handoff as a blocker for
  this run, but was not independently confirmed as a stable, reproducible constraint of the
  environment (only observed once, in one session), so it does not meet the "verified reusable"
  bar for promotion to `docs/memory/known-problems.md`.

## Next executable action

None — fix implemented, task complete pending human review of the branch.
