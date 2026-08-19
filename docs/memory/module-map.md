# Module Map

Current verified state as of 2026-08-19.

| Module | Responsibility | Main paths | Depends on |
|---|---|---|---|
| Bug automation | Issue opened -> branch -> Claude fix -> board status | `.github/workflows/claude-bug-fix.yml` | `CLAUDE_CODE_OAUTH_TOKEN`, `PROJECT_PAT`, project vars |
| Refix loop | `/refix` comment -> new branch -> follow-up fix | `.github/workflows/claude-refix.yml` | same as above |
| Merge gate | `branch-approved` label -> merge, delete branch, board move | `.github/workflows/branch-approve-merge.yml` | `PROJECT_PAT`, `READY_FOR_TESTING_OPTION_ID` |
| Static pages | Login, forgot-password and registration screens | `First-file.html`, `forgot-password.html`, `register.html` | none (no build step) |
| Agent instructions | Always-on rules and memory policy | `CLAUDE.md`, `.claude/rules/` | — |

## Not yet present

No NestJS backend, Angular frontend, PostgreSQL schema, `package.json` or test suite exists in this
repository, despite the stack declared in `CLAUDE.md`. The backend and database rules in
`.claude/rules/` are therefore currently inert.

## Cross-module contracts

- Board status IDs are supplied as GitHub Actions **variables**: `PROJECT_ID`, `STATUS_FIELD_ID`,
  `TO_TRIAGE_OPTION_ID`, `PENDING_APPROVAL_OPTION_ID`, `READY_FOR_TESTING_OPTION_ID`,
  `DONE_OPTION_ID`. All three workflows depend on them; changing the project board breaks all three.
- `PENDING_APPROVAL_OPTION_ID` (`f75ad846`) is the ID of the board column literally named
  **Pending approval**. Verified against the live board on 2026-08-19.
  `CLAUDE.md` describes this column as "Review (was Pending approval)", but **no rename has
  happened on the board** — the option is still `Pending approval`. Trust the board, not that line.
  Do not rename the variable without updating both workflows that read it.
- The branch prefix `fix/<issue>-` is the contract between `claude-bug-fix.yml` (writer) and
  `branch-approve-merge.yml` (reader).

## High-conflict files

- `CLAUDE.md` — loaded by every run; parallel issue branches will conflict on it.
- `docs/memory/*.md` — same. Rebase on `main` before finalizing a memory edit.
