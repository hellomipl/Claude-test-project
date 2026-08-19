# Issue 18 — No way to create an account from the login page

- GitHub issue: #18
- Status: Ready for review
- Branch: `fix/18-no-way-to-create-an-account-from-the-login-page`
- Last updated: 2026-08-19

## Objective

`First-file.html` (the login page) offers only "Login" and "Forgot password?" — a first-time
visitor with no account has no way to reach account creation.

## Scope

### In scope

- Add a visible "Create account" link on `First-file.html`.
- Add a new static `register.html` page (client-side only, matching the existing
  `forgot-password.html` look and structure) that the link points to.
- Register the new page in `docs/memory/module-map.md` per `.claude/rules/20-frontend.md`.

### Out of scope

- Any real backend/API for account creation — no backend exists in this repo (KP-0001).
- DB schema changes — none exist to change, and none are required for this fix.

## Acceptance criteria

- [x] Login page (`First-file.html`) has a clear, visible link to create an account.
- [x] The link target (`register.html`) exists and renders a usable registration form.
- [x] Registration page links back to the login page.
- [x] New page registered in `docs/memory/module-map.md`.

## Assumptions

- No backend exists, so "create an account" means presenting a registration form with the same
  client-side validation pattern as `forgot-password.html`, not a working account-creation API.
  Unverified against the issue author, but consistent with repo reality per `module-map.md`
  ("Not yet present": no NestJS backend). Stated as an explicit assumption since the issue itself
  doesn't specify backend behavior — it's scoped as a UI dead-end bug.

## Blockers / questions

- None blocking. Note: `gh issue comment` is refused per KP-0006 — triage comment could not be
  posted to the issue; analysis is recorded here and in the handoff instead.

## Context loaded

- [x] `CLAUDE.md` and `.claude/rules/`
- [x] `docs/memory/project-memory.md` and `module-map.md`
- [x] `docs/memory/known-problems.md` and `failed-approaches.md`
- [x] Previous task/handoff for this issue — none existed before this run

## Implementation checklist

- [x] Inspect existing code before editing (`First-file.html`, `forgot-password.html`)
- [x] Implement the smallest safe fix
- [x] Add or update tests where a harness exists (none exists — KP-0001)
- [x] Evaluate durable-memory impact
- [x] Update the handoff
- [x] Commit and push to the issue branch

## Affected files

- `First-file.html` — add "Create account" link
- `register.html` — new file
- `docs/memory/module-map.md` — register new page

## Findings

- No `package.json`/test harness in this repo (KP-0001, confirmed still true).
- `gh issue comment` denied (KP-0006, reproduced again on issue #18).
- Repo has exactly two prior static pages, no build step — confirmed via direct file listing.

## Decisions made

- New page named `register.html` (not `signup.html`) to match the plain, descriptive naming of
  `forgot-password.html`.
- Reused the exact CSS card pattern from `forgot-password.html` rather than introducing a new
  visual style, per "prefer minimal fixes" and to keep the two auth-adjacent pages consistent.

## Validation results

| Command or check | Result | Evidence |
|---|---|---|
| `npm test` / `npm run lint` / `npm run build` | Not run | No `package.json` exists (KP-0001) |
| `python3 -c "html.parser...".feed(...)` well-formedness check | Not run | `Bash` denied without approval, same as KP-0006 pattern for `python3` |
| Manual read-through of `register.html` and updated `First-file.html` | Pass | Reviewed HTML/CSS/JS end to end for syntax and link correctness |
| `grep -n "register.html\|forgot-password.html\|First-file.html" First-file.html register.html forgot-password.html` | Pass | All three cross-page links resolve to files that exist at repo root: `First-file.html` -> `register.html` -> `First-file.html`, and `First-file.html` -> `forgot-password.html` -> `First-file.html` |

**Residual risk:** no browser or automated render check was possible in this environment (no
headless browser tool, and `python3`/other non-`git` commands are denied per KP-0006). The page was
only verified by reading the markup/CSS/JS and confirming link targets — a human should open
`register.html` in a browser before approving, to confirm rendering and the JS validation
interactions actually behave as intended.

## Durable memory impact

- Applied: registered `register.html` in `docs/memory/module-map.md` (required by
  `.claude/rules/20-frontend.md` for any new root-level HTML page).
- No other promotion: the `gh issue comment` denial reproduced again (matches KP-0006 exactly,
  same as issues #16/#17), so it adds no new information to promote. Everything else about this
  fix (page content, link wiring) is specific to issue #18 and stays in this task file.

## Next executable action

Implement `register.html`, update `First-file.html`, update `module-map.md`, then commit and push.
