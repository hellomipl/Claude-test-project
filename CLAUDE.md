# Project context

## Architecture
- Backend: NestJS
- Frontend: Angular
- DB: PostgreSQL
- Auth: JWT + refresh tokens

## Commands
- Install: npm install
- Test: npm test
- Lint: npm run lint
- Build: npm run build

> **These commands do not currently run.** There is no `package.json` in this repository.
> See `docs/memory/known-problems.md` (KP-0001). Do not report a fix as tested until this is fixed —
> say what you verified instead.

## Rules
- Do not change DB schema unless issue explicitly requires it
- Prefer minimal fixes
- Keep API contracts backward compatible
- Add or update tests when fixing bugs

## Source of truth

The Git repository is the durable source of truth. A GitHub Actions runner is temporary — only
files that are **committed and pushed** to the issue branch survive the run. Do not rely on
machine-local auto-memory for shared project knowledge.

## Before starting any issue

1. Read the full issue, its labels and its comments.
2. Read `docs/memory/project-memory.md` and `docs/memory/module-map.md`.
3. Read `docs/memory/known-problems.md` and `docs/memory/failed-approaches.md`.
4. Read `docs/memory/coding-patterns.md` when writing code that resembles an existing pattern.
5. Read `docs/tasks/issue-<number>.md` and `docs/handoffs/issue-<number>.md` if they exist.
6. Create `docs/tasks/issue-<number>.md` from `docs/tasks/ISSUE-TASK-TEMPLATE.md` **before**
   implementing, and commit it early. If the run stops mid-way, that file is the only record.

Load only what is relevant. Do not read every document for a one-line fix.

## Memory placement

| What | Where |
|---|---|
| Temporary findings, current progress | `docs/tasks/issue-<number>.md` |
| Continuation state, remaining risk | `docs/handoffs/issue-<number>.md` |
| Verified, reusable facts | `docs/memory/` |
| Significant or hard-to-reverse decisions | `docs/decisions/` |
| Mandatory behavior and standards | this file, or `.claude/rules/` |

Follow `.claude/skills/update-project-memory/SKILL.md` when deciding what to promote.

## Memory quality rule

Never promote a guess, a temporary debugging note, a secret, or an unverified root cause into
`docs/memory/`. If nothing durable was learned, write `Memory impact: none` in the handoff. That is
the normal outcome for most bug fixes.

## Completion gates

Before the run ends, confirm:

- The reported bug is fixed, or the blocker is stated explicitly.
- Validation was attempted and the **real** result is recorded — including "could not run, because".
- `docs/tasks/issue-<number>.md` is current.
- `docs/handoffs/issue-<number>.md` is written.
- Durable-memory impact was evaluated, even if the answer is none.
- No secret or generated dependency directory is committed.
- Everything is committed and pushed to the issue branch.

## Bug-fix workflow
- When a bug issue is created, a branch is automatically created from main
- Claude analyzes the bug and posts a triage comment on the issue
- Claude implements the fix and commits directly to the issue branch
- Do NOT create pull requests
- Do NOT merge to main
- Commit and push all changes to the issue branch only
- Developer reviews the branch and adds label `branch-approved` to trigger merge to main
- Keep changes minimal and focused only on the reported bug

## GitHub Project board statuses
- `To triage` — issue created, Claude is analyzing
- `Review` (was Pending approval) — fix committed, developer reviews the branch
- `Ready for testing` — branch merged to main, QA pending
- `Done` — tester verified fix works

## GitHub Action variables needed
### Secrets
- `CLAUDE_CODE_OAUTH_TOKEN`
- `PROJECT_PAT`

### Variables
- `PROJECT_ID`
- `STATUS_FIELD_ID`
- `TO_TRIAGE_OPTION_ID`
- `PENDING_APPROVAL_OPTION_ID` (used for Review status)
- `READY_FOR_TESTING_OPTION_ID`
- `DONE_OPTION_ID`

## Engineering rules

These are imported so they load in every session, including GitHub Actions runs. Each file states
what it applies to — skip the ones that do not touch the current issue.

@.claude/rules/00-core.md
@.claude/rules/05-project-config.md
@.claude/rules/10-backend.md
@.claude/rules/20-frontend.md
@.claude/rules/30-database.md
@.claude/rules/40-testing.md
@.claude/rules/50-security.md
@.claude/rules/60-git-branch.md
@.claude/rules/70-documentation-memory.md

## Stale documentation

`docs/context.md` describes an older pull-request-based flow with a `claude-fix-approved` label and
a `project-ready-for-testing.yml` workflow. **It is out of date and contradicts this file.** This
file and `.github/workflows/` are authoritative. See KP-0004.
