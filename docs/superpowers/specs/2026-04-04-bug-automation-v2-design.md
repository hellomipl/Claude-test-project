# Bug Automation V2 — Direct Branch Fix Flow

## Summary

Rework the bug automation pipeline so Claude fixes bugs immediately on auto-created branches without waiting for developer approval or creating PRs. Developers review the branch directly and approve via label to merge to main.

## Flow

```
Issue created
  → Branch fix/<number>-<title-slug> auto-created from main
  → Board: "To triage"
  → Claude analyzes bug, posts triage comment
  → Claude fixes code, commits directly to branch
  → Board: "In progress" → "Review"
  → Developer reviews branch
  → Developer adds label "branch-approved"
  → Branch auto-merges to main, branch deleted
  → Board: "Ready for testing"
  → Tester verifies
  → If fixed → "Done"
  → If not → /refix comment → new branch from main → cycle repeats
```

## Board Statuses

| Status | Meaning |
|---|---|
| To triage | Issue created, Claude is analyzing |
| In progress | Claude is committing the fix to the branch |
| Review | Fix committed, developer reviews the branch |
| Ready for testing | Branch merged to main, QA pending |
| Done | Tester verified fix works |

## Workflow Files

### 1. `claude-bug-fix.yml`

**Trigger:** Issue opened

**Steps:**
1. Create branch `fix/<issue-number>-<title-slug>` from main
2. Checkout the new branch
3. Move issue card to `To triage`
4. Claude analyzes the bug and posts a triage comment (root cause, impacted files, fix approach)
5. Claude implements the fix and commits directly to the branch
6. Move issue card to `In progress` then to `Review`

**Claude instructions:**
- Read CLAUDE.md for project context
- First analyze the bug: identify root cause, impacted modules/files, suggest fix approach — post as a comment
- Then implement the smallest safe fix on the branch
- Commit directly — do not create a PR
- Update or add tests if needed

**Branch naming:** `fix/<issue-number>-<slugified-issue-title>`
- Slug: lowercase, spaces to hyphens, remove special chars, truncate to 50 chars

### 2. `branch-approve-merge.yml`

**Trigger:** Issue labeled with `branch-approved`

**Steps:**
1. Identify the issue branch (`fix/<issue-number>-*`)
2. Merge the branch to main
3. Delete the issue branch
4. Move issue card to `Ready for testing`
5. Remove the `branch-approved` label (clean up for potential refix cycle)

### 3. `claude-refix.yml`

**Trigger:** Issue comment containing `/refix`

**Steps:**
1. Create a new branch `fix/<issue-number>-<title-slug>-refix-<timestamp>` from main
2. Checkout the new branch
3. Move issue card to `In progress`
4. Claude re-analyzes bug with original issue + refix comment context
5. Claude implements follow-up fix, commits directly to branch
6. Move issue card to `Review`

**Claude instructions:**
- Read CLAUDE.md for project context
- Read the original issue and all comments
- Understand why the previous fix was insufficient
- Implement the smallest safe follow-up fix
- Commit directly — do not create a PR
- Update or add tests if needed

### 4. `project-ready-for-testing.yml`

**Status:** Removed. No longer needed — `branch-approve-merge.yml` handles moving the card to `Ready for testing` after merge.

## Labels

| Label | Purpose |
|---|---|
| `branch-approved` | Developer signals the branch is ready to merge to main |

**Removed:** `claude-fix-approved` — no longer needed since Claude starts immediately.

## GitHub Project Variables

Existing variables remain. New option IDs needed for the two new statuses:

| Variable | Purpose |
|---|---|
| `PROJECT_ID` | Existing |
| `STATUS_FIELD_ID` | Existing |
| `TO_TRIAGE_OPTION_ID` | Existing |
| `IN_PROGRESS_OPTION_ID` | **New** — for "In progress" status |
| `REVIEW_OPTION_ID` | **New** — for "Review" status |
| `READY_FOR_TESTING_OPTION_ID` | Existing |
| `DONE_OPTION_ID` | Existing |

**Removed:** `PENDING_APPROVAL_OPTION_ID` — status no longer exists.

## Secrets

No changes:
- `CLAUDE_CODE_OAUTH_TOKEN`
- `PROJECT_PAT`

## Permissions

### claude-bug-fix.yml
```yaml
permissions:
  contents: write
  issues: write
  id-token: write
```

### branch-approve-merge.yml
```yaml
permissions:
  contents: write
  issues: write
```

### claude-refix.yml
```yaml
permissions:
  contents: write
  issues: write
  id-token: write
```

## Key Differences from V1

| Aspect | V1 (Current) | V2 (New) |
|---|---|---|
| Trigger for Claude to fix | Developer adds `claude-fix-approved` label | Automatic on issue creation |
| Fix delivery | PR created | Direct commit to issue branch |
| Developer review | Review PR | Review branch directly |
| Merge trigger | Developer merges PR manually | `branch-approved` label triggers auto-merge |
| Branch creation | Manual or by PR | Auto-created on issue open |
| Branch cleanup | Manual | Auto-deleted after merge |
| Triage | Separate workflow | Same workflow, first step before fixing |
| Board statuses | 4 (To triage, Pending approval, Ready for testing, Done) | 5 (To triage, In progress, Review, Ready for testing, Done) |
| Refix | New PR on same branch context | New branch from main |

## Files to Create/Modify

- **Modify:** `.github/workflows/claude-bug-fix.yml` — complete rewrite
- **Create:** `.github/workflows/branch-approve-merge.yml` — new workflow
- **Modify:** `.github/workflows/claude-refix.yml` — rewrite for direct commit flow
- **Delete:** `.github/workflows/claude-bug-triage.yml` — merged into claude-bug-fix.yml
- **Delete:** `.github/workflows/project-ready-for-testing.yml` — replaced by branch-approve-merge.yml
- **Update:** `CLAUDE.md` — reflect new flow and rules
- **Update:** `docs/context.md` — reflect new flow
