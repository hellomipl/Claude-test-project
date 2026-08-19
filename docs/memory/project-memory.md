# Project Memory

Keep this file concise. Link to detailed documents rather than duplicating them.

## Project identity

- Project: Claude-test-project — a testbed for GitHub Issues + Projects + Claude Code Action bug automation.
- Primary purpose: exercise the issue -> branch -> fix -> human-approved-merge pipeline.
- Default branch: `main`.

## Critical invariants

- The agent never creates a pull request and never merges to `main`. Merge is triggered by a human
  applying the `branch-approved` label. (Verified: `.github/workflows/branch-approve-merge.yml`.)
- Every fix lands on `fix/<issue>-<slug>`, created by the workflow before Claude starts.
  (Verified: `.github/workflows/claude-bug-fix.yml`.)
- The issue branch is deleted immediately after merge. Anything not committed to it is lost.
  (Verified: `branch-approve-merge.yml` calls `git.deleteRef` after `repos.merge`.)
- The issue stays open through `Ready for testing`; a tester closes it. Do not use `Fixes #N` or
  `Closes #N` in commit messages, or GitHub will close the issue early.
- `claude-bug-fix.yml` triggers on `issues: opened` with no approval label. There is currently no
  human gate before the agent runs. (Verified 2026-08-19; see KP-0002.)

## Repository reality vs. declared stack

`CLAUDE.md` declares NestJS / Angular / PostgreSQL / JWT. As of 2026-08-19 the repository contains
only two standalone HTML pages (`First-file.html`, `forgot-password.html`) and no application code,
no `package.json` and no test harness. Treat the declared stack as the intended target, not the
current state. See KP-0001 before running any `npm` command.

## Project board (verified 2026-08-19 via GraphQL)

- Board: user project **5** under `hellomipl` (`PVT_kwHOBYopsc4BTmKH`).
- Status field: `PVTSSF_lAHOBYopsc4BTmKHzhA0F_4`.
- Status options, exactly as named on the board:
  `To triage` (`f971fb55`), `Pending approval` (`f75ad846`), `Ready for testing` (`856cdede`),
  `Done` (`98236657`).
- There is **no** `Review`, `In progress` or `Blocked` column, despite what `CLAUDE.md` and
  `docs/context.md` imply. See KP-0005.
- New repository issues are added to this board automatically; issues #13, #14 and #15 all carry
  a project item without anyone adding them by hand.

## Architecture index

- Module map: `docs/memory/module-map.md`
- Known problems: `docs/memory/known-problems.md`
- Failed approaches: `docs/memory/failed-approaches.md`
- Coding patterns: `docs/memory/coding-patterns.md`
- Decisions: `docs/decisions/`

## Operational commands

See the `## Commands` section of `CLAUDE.md`, and read KP-0001 first — the declared commands do
not currently execute.

## High-risk areas

- `.github/workflows/` — a broken workflow silently disables the whole automation.
- Branch-prefix matching in `branch-approve-merge.yml` (see KP-0003).

## Maintenance

Add only verified, reusable facts. Detailed issue history belongs in task and handoff files.
