# Claude Code Handoff Pack

This canvas contains three ready-to-use files for your GitHub + Claude Code bug automation setup.

Copy each section into its own file:
- `CLAUDE.md`
- `bug-automation-setup.md`
- `project-board-flow.md`

---

# File: `CLAUDE.md`

```md
# Project context

## Goal
This repo uses GitHub Issues, GitHub Projects, GitHub Actions, and Claude Code Action to automate bug fixing.

Main flow:
- Tester creates a bug issue
- Issue is added to GitHub Project board
- Status starts at `To triage`
- Developer approves automation by adding label `claude-fix-approved`
- Claude creates a PR with a minimal fix
- Issue card moves to `Pending approval`
- After PR merge, issue card moves to `Ready for testing`
- Tester verifies the fix
- If fixed, move to `Done`
- If not fixed, move back to `To triage` and comment `/refix ...details...`

## Stack
- Code hosting: GitHub
- Board: GitHub Projects v2
- Automation: GitHub Actions
- AI coding: Claude Code Action

## Rules for Claude
- Keep fixes minimal and focused only on the reported bug.
- Do not do broad refactors unless explicitly asked.
- Do not change database schema unless the issue clearly requires it.
- Do not merge PRs automatically.
- Create PRs only.
- Prefer targeted edits over sweeping changes.
- Update or add tests if needed.
- In PR body include this exact line:
  `Issue: #<issue-number>`
- Do not use `Fixes #123` or `Closes #123` in the PR body because the issue must remain open until testing is complete.

## GitHub Project values
Project board uses these Status values:
- `To triage`
- `Pending approval`
- `Ready for testing`
- `Done`

## GitHub Action secrets and variables
### Secrets
- `CLAUDE_CODE_OAUTH_TOKEN`
- `PROJECT_PAT`

### Variables
- `PROJECT_ID`
- `STATUS_FIELD_ID`
- `TO_TRIAGE_OPTION_ID`
- `PENDING_APPROVAL_OPTION_ID`
- `READY_FOR_TESTING_OPTION_ID`
- `DONE_OPTION_ID`

## Actual project values
- `PROJECT_ID = PVT_kwHOBYopsc4BTmKH`
- `STATUS_FIELD_ID = PVTSSF_lAHOBYopsc4BTmKHzhA0F_4`
- `TO_TRIAGE_OPTION_ID = f971fb55`
- `PENDING_APPROVAL_OPTION_ID = f75ad846`
- `READY_FOR_TESTING_OPTION_ID = 856cdede`
- `DONE_OPTION_ID = 98236657`

## Important workflow behavior
- New issues should enter the board in `To triage`.
- Label `claude-fix-approved` triggers Claude to implement a fix and open a PR.
- `/refix` comment triggers Claude to create a follow-up PR.
- When Claude creates a PR, move the linked issue card to `Pending approval`.
- When that PR is merged, move the issue card to `Ready for testing`.
- Tester decides whether to move issue to `Done` or back to `To triage`.

## File paths used in this repo
- `.github/workflows/claude-bug-fix.yml`
- `.github/workflows/claude-refix.yml`
- `.github/workflows/project-ready-for-testing.yml`
- `.github/ISSUE_TEMPLATE/bug_report.md`
- `CLAUDE.md`

## Notes
- GitHub Project board movement itself is not the trigger for Claude.
- Issue events and issue comments are the real triggers.
- Board movement is the visual workflow layer.
```

---

# File: `bug-automation-setup.md`

```md
# Bug Automation Setup

This file explains the GitHub + Claude Code automation setup for bug fixing.

## Goal
Build a workflow where:
1. Tester creates a bug issue.
2. Issue appears in the GitHub Project board under `To triage`.
3. Developer approves automated fixing.
4. Claude creates a PR with the code change.
5. The issue card moves to `Pending approval`.
6. After PR merge, the issue card moves to `Ready for testing`.
7. Tester verifies the result.
8. If the bug still exists, tester comments `/refix` and Claude creates a new PR.

## Required GitHub pieces

### 1. Repository secrets
Create these secrets:
- `CLAUDE_CODE_OAUTH_TOKEN`
- `PROJECT_PAT`

### 2. Repository variables
Create these variables:
- `PROJECT_ID = PVT_kwHOBYopsc4BTmKH`
- `STATUS_FIELD_ID = PVTSSF_lAHOBYopsc4BTmKHzhA0F_4`
- `TO_TRIAGE_OPTION_ID = f971fb55`
- `PENDING_APPROVAL_OPTION_ID = f75ad846`
- `READY_FOR_TESTING_OPTION_ID = 856cdede`
- `DONE_OPTION_ID = 98236657`

### 3. Labels
Create this label:
- `claude-fix-approved`

### 4. Issue template
Use a bug issue template so every bug includes:
- bug summary
- steps to reproduce
- expected result
- actual result
- screenshots/logs
- affected module/page/API
- severity

Suggested file path:
- `.github/ISSUE_TEMPLATE/bug_report.md`

## Workflow files used

### A. `claude-bug-fix.yml`
Trigger:
- issue labeled with `claude-fix-approved`

Purpose:
- Claude reads issue
- Claude reads `CLAUDE.md`
- Claude makes code changes
- Claude creates a PR
- workflow updates issue card status to `Pending approval`

Important rule:
- PR body must include:
  `Issue: #<issue-number>`
- PR body must not include:
  `Fixes #123`
  `Closes #123`

### B. `project-ready-for-testing.yml`
Trigger:
- pull request closed

Condition:
- only run if PR was merged

Purpose:
- parse PR body to find `Issue: #123`
- find the linked issue in the GitHub Project
- move issue card status to `Ready for testing`

### C. `claude-refix.yml`
Trigger:
- new issue comment

Condition:
- comment contains `/refix`

Purpose:
- Claude re-analyzes the issue
- Claude creates a follow-up PR
- workflow moves issue card to `Pending approval`

## Recommended permissions

### For fix and refix workflows
```yaml
permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write
```

### For project move after merge
```yaml
permissions:
  contents: read
```

## Claude action behavior
Claude can:
- read issue content
- read repository files
- edit code
- create a PR

Claude should not:
- merge PRs automatically
- broadly refactor unless asked
- close issues automatically before testing is complete

## Why `Issue: #123` is important
The project automation for moving cards after merge needs a reliable way to map the PR back to the issue.

Using:
- `Issue: #123`

is safe.

Using:
- `Fixes #123`
- `Closes #123`

is not safe here because GitHub would auto-close the issue when the PR merges, which breaks the `Ready for testing` stage.

## Recommended tester workflow
1. Create issue
2. Verify issue appears in `To triage`
3. Wait for developer approval
4. After Claude creates PR and it is merged, retest
5. If still broken, move card back to `To triage` and comment `/refix ...details...`
6. If fixed, move card to `Done`

## Recommended developer workflow
1. Review new bug in `To triage`
2. Add label `claude-fix-approved`
3. Review PR in `Pending approval`
4. Merge PR
5. Wait for tester validation in `Ready for testing`
```

---

# File: `project-board-flow.md`

```md
# Project Board Flow

This file explains how the GitHub Project board should be used with the Claude automation.

## Board columns / statuses
Use these statuses in the Project board:
- `To triage`
- `Pending approval`
- `Ready for testing`
- `Done`

## Meaning of each status

### To triage
This means:
- tester has reported a bug
- issue has been added to the board
- developer has not yet approved Claude to make a fix

Actions allowed here:
- developer reviews bug
- developer adds `claude-fix-approved`
- tester may add more context if needed

### Pending approval
This means:
- Claude has created a PR
- code changes exist
- developer needs to review the PR

Actions allowed here:
- review code
- request changes
- merge PR when satisfied

### Ready for testing
This means:
- PR has been merged
- fix is now ready for QA verification

Actions allowed here:
- tester rechecks the bug
- tester verifies edge cases
- tester confirms fixed or still failing

### Done
This means:
- tester verified the bug is fixed
- issue lifecycle is complete

## Main board flow

### Initial bug
```text
Tester creates issue
-> issue appears in To triage
-> developer adds claude-fix-approved
-> Claude creates PR
-> issue moves to Pending approval
-> developer merges PR
-> issue moves to Ready for testing
-> tester verifies
-> if fixed: move to Done
```

### Refix flow
```text
Issue is in Ready for testing
-> tester checks and finds bug still exists
-> tester moves issue back to To triage
-> tester comments /refix with details
-> Claude creates new PR
-> issue moves to Pending approval
-> developer merges PR
-> issue moves to Ready for testing
-> tester verifies again
-> if fixed: move to Done
```

## Important rules

### Rule 1
Moving the card between columns does not itself trigger Claude.

Claude is triggered by:
- issue label `claude-fix-approved`
- issue comment `/refix`

### Rule 2
Board movement is used for workflow visibility.

That means:
- board = process status
- issues/comments/labels = automation triggers

### Rule 3
Do not auto-close the issue when PR is merged.

The issue must stay open while in `Ready for testing`.

### Rule 4
Tester should always add details with `/refix`

Good examples:
- `/refix still failing when password has special characters`
- `/refix fixed on desktop but broken on mobile`
- `/refix API works but UI still shows old validation error`

This gives Claude better context for the second PR.

## Simple operating model

### Tester responsibilities
- create issues clearly
- retest after merge
- move to `Done` if fixed
- move back to `To triage` and use `/refix` if not fixed

### Developer responsibilities
- review issues in `To triage`
- approve Claude by labeling `claude-fix-approved`
- review PRs in `Pending approval`
- merge safe PRs

### Claude responsibilities
- implement minimal safe fix
- create PR
- never merge automatically
- keep changes focused only on the issue

## Best practice
Treat the board as the visible process layer:
- `To triage` = waiting for automation approval
- `Pending approval` = code ready for human review
- `Ready for testing` = code merged, QA validation pending
- `Done` = QA passed

Treat issue comments and labels as the automation control layer:
- `claude-fix-approved` = first fix
- `/refix` = follow-up fix
```

---

You can now copy each section into the exact file names shown above.

