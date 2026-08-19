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

### Pattern: Use a single-word prefix in `Bash()` permission patterns

- Applies to: `--allowedTools` in `.github/workflows/*.yml`, and `allowed-tools` in skill frontmatter
- Reference implementation: `.github/workflows/claude-bug-fix.yml`
- Verified: 2026-08-19, from the issue #17 run log

**Use when**

Granting an agent permission to run a command family in an unattended run.

**Implementation rules**

- Write the pattern as `Bash(<command>:*)` with a **colon** before the wildcard.
- Keep the prefix to a **single word**. `Bash(git:*)` works. A two-word prefix did not.
- In an unattended run nobody can answer an approval prompt, so a non-matching pattern does not
  merely warn — the command is refused and that part of the task silently does not happen, while
  the run still reports success.

**Evidence (issue #17 run 32227877961, one run, one permission list)**

| Command | Pattern that should cover it | Result |
|---|---|---|
| `git commit -m "$(cat <<'EOF' …)"` | `Bash(git:*)` — 1 word | **allowed** |
| `gh issue comment 17 --body "$(cat …)"` | `Bash(gh issue:*)` — 2 words | **denied** ×8 |
| `gh auth status` | none | denied ×4 (correct) |
| `python3 /tmp/check_html.py` | none | denied ×8 (correct) |

**Avoid**

- Assuming a multi-word prefix such as `Bash(gh issue:*)` narrows access safely — it appears to
  match nothing at all, which fails closed but silently.
- Concluding that command substitution is what blocks a command. It is not: the allowed `git commit`
  above contains `$(cat <<'EOF' …)` and ran fine.

**Open, not yet verified**

The single-word-prefix explanation fits every row above but has not been isolated by a controlled
test. `Bash(gh:*)` is the predicted fix and is untested. Separately, the Claude step sets no
`GH_TOKEN`, so `gh` may also be unauthenticated in the runner — that would be a second, independent
cause. See KP-0006.

---

## Template

### Pattern: Short name

- Applies to: which files or layer
- Reference implementation: `path/to/file`
- Verified: YYYY-MM-DD

**Use when**

**Implementation rules**

**Avoid**
