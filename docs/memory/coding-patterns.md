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

## Template

### Pattern: Short name

- Applies to: which files or layer
- Reference implementation: `path/to/file`
- Verified: YYYY-MM-DD

**Use when**

**Implementation rules**

**Avoid**
