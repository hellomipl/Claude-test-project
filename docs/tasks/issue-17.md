# Issue 17 — Form validation errors are not announced to screen readers

- GitHub issue: #17
- Status: In progress
- Branch: `fix/17-form-validation-errors-are-not-announced-to-screen`
- Last updated: 2026-08-19 00:00 UTC

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

- [ ] `#username-error` and `#password-error` are tied to their inputs via `aria-describedby` and
      announced via a live region when they become visible.
- [ ] `#email-error` in `forgot-password.html` has the same treatment.
- [ ] `aria-invalid` toggles true/false on each input alongside the existing `.invalid` class logic.
- [ ] `#success-message` in `forgot-password.html` is announced via a live region when it appears.

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

- [ ] Inspect existing code before editing
- [ ] Implement the smallest safe fix
- [ ] Add or update tests where a harness exists — none exists (KP-0001)
- [ ] Evaluate durable-memory impact
- [ ] Update the handoff
- [ ] Commit and push to the issue branch

## Affected files

- `First-file.html`
- `forgot-password.html`

## Findings

- Neither page contains any `aria-*` attribute or `role` (confirmed by reading both files in
  full). Confirmed evidence for the issue report.
- The error/success visibility toggles are driven entirely by adding/removing a `visible` CSS class
  in the inline `<script>` blocks; there is no framework or component abstraction to update.

## Decisions made

- Plan: use `role="alert"` on error spans and `role="status"` on the success banner (implicit live
  regions) rather than explicit `aria-live` attributes — simpler and standard-conformant.
- Plan: set `aria-invalid="false"` as the static default on inputs and flip it to `"true"`/`"false"`
  via `setAttribute` in the existing submit handlers, alongside the existing class toggles.

## Validation results

| Command or check | Result | Evidence |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists (KP-0001) |

## Durable memory impact

- Candidates: TBD after implementation.

## Next executable action

Implement the ARIA attribute changes in `First-file.html` and `forgot-password.html`.
