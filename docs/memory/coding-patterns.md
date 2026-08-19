# Approved Coding Patterns

Patterns verified in this repository and reusable in future tasks.

### Pattern: Move a Projects v2 card status from a workflow

- Applies to: `.github/workflows/*.yml`
- Reference implementation: `.github/workflows/claude-bug-fix.yml`, step "Move issue card to Review"
- Verified: 2026-08-19

**Use when**

A workflow needs to change a GitHub Projects v2 card status.

**Implementation rules**

- Use `actions/github-script` with `github-token` set to the `PROJECT_PAT` secret. The default
  `GITHUB_TOKEN` cannot write Projects v2.
- Pass IDs in through the step's `env:` block from repository variables, then read them via
  `process.env` inside the script. Do not interpolate variables directly into the script body.
- Query `repository.issue.projectItems` and match on `project.id` before mutating — an issue can
  belong to more than one project.
- Choose deliberately between hard failure and skip when the item is not found.
  `claude-bug-fix.yml` skips on the first move, because the card may not be added to the board yet,
  and fails on the later move.

**Avoid**

- Assuming the card already exists at issue-open time.
- Renaming the `PENDING_APPROVAL_OPTION_ID` variable; it maps to the column now named Review.

---

### Pattern: Bash tool permission globs use a colon, not a space

- Applies to: `--allowedTools` in `.github/workflows/*.yml`, and `allowed-tools` in skill frontmatter
- Reference implementation: `.github/workflows/claude-bug-fix.yml`
- Verified: 2026-08-19, empirically, during issue #16

**Use when**

Granting an agent permission to run a specific command family in an unattended run.

**Implementation rules**

- Write the pattern as `Bash(<command>:*)`. The separator before the wildcard is a **colon**.
- A multi-word prefix takes the colon at the end: `Bash(gh issue:*)`.
- In an unattended run nobody can answer an approval prompt, so a non-matching pattern does not
  merely warn — the command is refused and that part of the task silently does not happen.

**Avoid**

- The space form `Bash(gh issue *)`. Proven during issue #16: in one run, with one permission list,
  `Bash(git:*)` matched and every git commit succeeded, while `Bash(gh issue *)` did not match and
  the triage comment was never posted. Only the separator differed.

---

### Pattern: Prevent native submission unconditionally in a form's submit handler

- Applies to: standalone HTML pages at the repository root (`*.html`) with a JS `submit` listener
- Reference implementation: `forgot-password.html`, `First-file.html` (fixed in issue #16)
- Verified: 2026-08-19

**Use when**

Adding or editing a `<form>` whose submission is handled entirely in client-side JS (no real
`action`/`method`, e.g. a static demo page).

**Implementation rules**

- Call `e.preventDefault()` as the *first* statement in the `submit` listener, unconditionally —
  before any validation runs. Do not gate it behind a validity check.
- A form with no `action`/`method` defaults to a GET against the current URL on native submit, which
  serializes every field — including passwords — into the query string, browser history and
  Referer headers.

**Avoid**

- Calling `e.preventDefault()` only inside an `if (!valid)` branch (or any conditional path); that
  leaves the successful/valid path free to trigger a native submit. This was the exact bug in
  issue #16.

---

## Template

### Pattern: Short name

- Applies to: which files or layer
- Reference implementation: `path/to/file`
- Verified: YYYY-MM-DD

**Use when**

**Implementation rules**

**Avoid**
