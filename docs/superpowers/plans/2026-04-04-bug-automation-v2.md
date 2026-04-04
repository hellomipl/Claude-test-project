# Bug Automation V2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rework the bug automation pipeline so Claude fixes bugs immediately on auto-created branches, developers review branches directly, and approve via label to merge to main.

**Architecture:** 3 GitHub Actions workflows replace the current 4. Issue creation triggers branch creation + Claude triage + fix. A label triggers merge + cleanup. `/refix` creates a new branch for follow-up fixes.

**Tech Stack:** GitHub Actions, GitHub Projects v2 GraphQL API, Claude Code Action, `actions/github-script@v8`

---

## File Map

| Action | File | Responsibility |
|---|---|---|
| Rewrite | `.github/workflows/claude-bug-fix.yml` | On issue open: create branch, triage, fix, commit, move board card |
| Create | `.github/workflows/branch-approve-merge.yml` | On `branch-approved` label: merge branch to main, delete branch, move card |
| Rewrite | `.github/workflows/claude-refix.yml` | On `/refix` comment: create new branch, fix, commit, move card |
| Delete | `.github/workflows/claude-bug-triage.yml` | Merged into claude-bug-fix.yml |
| Delete | `.github/workflows/project-ready-for-testing.yml` | Replaced by branch-approve-merge.yml |
| Update | `CLAUDE.md` | Reflect new flow and rules |

---

### Task 1: Rewrite `claude-bug-fix.yml`

**Files:**
- Rewrite: `.github/workflows/claude-bug-fix.yml`

- [ ] **Step 1: Replace the entire content of `.github/workflows/claude-bug-fix.yml`**

```yaml
name: Claude Bug Fix

on:
  issues:
    types: [opened]

permissions:
  contents: write
  issues: write
  id-token: write

jobs:
  fix:
    if: github.event.issue.pull_request == null
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Generate branch name
        id: branch
        uses: actions/github-script@v8
        with:
          result-encoding: string
          script: |
            const number = context.issue.number;
            const title = context.payload.issue.title;
            const slug = title
              .toLowerCase()
              .replace(/[^a-z0-9\s-]/g, '')
              .replace(/\s+/g, '-')
              .replace(/-+/g, '-')
              .replace(/^-|-$/g, '')
              .substring(0, 50);
            return `fix/${number}-${slug}`;

      - name: Create and push branch
        run: |
          git checkout -b ${{ steps.branch.outputs.result }}
          git push origin ${{ steps.branch.outputs.result }}

      - name: Move issue card to To triage
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          TO_TRIAGE_OPTION_ID: ${{ vars.TO_TRIAGE_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.TO_TRIAGE_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              console.log("Issue not yet in project, skipping status move.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });

      - name: Claude triage and fix
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          show_full_output: true
          prompt: |
            A new bug issue has been created.

            Repository: ${{ github.repository }}
            Issue #${{ github.event.issue.number }}
            Title: ${{ github.event.issue.title }}
            Branch: ${{ steps.branch.outputs.result }}

            Issue body:
            ${{ github.event.issue.body }}

            Read CLAUDE.md and repository context.

            Task:
            1. First, analyze the bug: identify the likely root cause, impacted modules/files, and your fix approach. Post this analysis as a comment on the issue.
            2. Then, checkout the branch: git checkout ${{ steps.branch.outputs.result }}
            3. Implement the smallest safe fix on this branch.
            4. Update or add tests if needed.
            5. Commit all changes directly to the branch and push.
            6. Do NOT create a pull request.
            7. Do NOT merge anything to main.
            8. Keep changes minimal and focused only on this bug.
          claude_args: >-
            --max-turns 20
            --allowedTools Read,Edit,Write,Glob,Grep,WebFetch,WebSearch,Bash(git:*),Bash(gh issue *),Bash(npm:*),Bash(node:*)

      - name: Move issue card to In progress
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          IN_PROGRESS_OPTION_ID: ${{ vars.IN_PROGRESS_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.IN_PROGRESS_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              core.setFailed("Issue is not in the target project.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });

      - name: Move issue card to Review
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          REVIEW_OPTION_ID: ${{ vars.REVIEW_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.REVIEW_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              core.setFailed("Issue is not in the target project.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });
```

- [ ] **Step 2: Commit**

```bash
git add .github/workflows/claude-bug-fix.yml
git commit -m "feat: rewrite claude-bug-fix to auto-create branch and commit directly"
```

---

### Task 2: Create `branch-approve-merge.yml`

**Files:**
- Create: `.github/workflows/branch-approve-merge.yml`

- [ ] **Step 1: Create `.github/workflows/branch-approve-merge.yml`**

```yaml
name: Branch Approve and Merge

on:
  issues:
    types: [labeled]

permissions:
  contents: write
  issues: write

jobs:
  merge:
    if: github.event.issue.pull_request == null && github.event.label.name == 'branch-approved'
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Find and merge issue branch
        id: merge
        uses: actions/github-script@v8
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const issueNumber = context.issue.number;
            const prefix = `fix/${issueNumber}-`;

            const { data: branches } = await github.rest.repos.listBranches({
              owner: context.repo.owner,
              repo: context.repo.repo,
              per_page: 100
            });

            const issueBranch = branches.find(b => b.name.startsWith(prefix));

            if (!issueBranch) {
              core.setFailed(`No branch found matching prefix: ${prefix}`);
              return;
            }

            const branchName = issueBranch.name;
            core.setOutput('branch_name', branchName);
            console.log(`Found branch: ${branchName}`);

            // Merge the branch to main
            try {
              await github.rest.repos.merge({
                owner: context.repo.owner,
                repo: context.repo.repo,
                base: 'main',
                head: branchName,
                commit_message: `Merge ${branchName} into main (Issue #${issueNumber})`
              });
              console.log(`Merged ${branchName} into main`);
            } catch (error) {
              core.setFailed(`Merge failed: ${error.message}`);
              return;
            }

            // Delete the branch
            try {
              await github.rest.git.deleteRef({
                owner: context.repo.owner,
                repo: context.repo.repo,
                ref: `heads/${branchName}`
              });
              console.log(`Deleted branch: ${branchName}`);
            } catch (error) {
              console.log(`Warning: Could not delete branch: ${error.message}`);
            }

      - name: Remove branch-approved label
        uses: actions/github-script@v8
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            try {
              await github.rest.issues.removeLabel({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                name: 'branch-approved'
              });
            } catch (error) {
              console.log(`Warning: Could not remove label: ${error.message}`);
            }

      - name: Move issue card to Ready for testing
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          READY_FOR_TESTING_OPTION_ID: ${{ vars.READY_FOR_TESTING_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.READY_FOR_TESTING_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              core.setFailed("Issue is not in the target project.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });

      - name: Comment on issue
        uses: actions/github-script@v8
        with:
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `Branch \`${{ steps.merge.outputs.branch_name }}\` has been merged to main and deleted. Issue moved to **Ready for testing**.`
            });
```

- [ ] **Step 2: Commit**

```bash
git add .github/workflows/branch-approve-merge.yml
git commit -m "feat: add branch-approve-merge workflow for label-triggered merge"
```

---

### Task 3: Rewrite `claude-refix.yml`

**Files:**
- Rewrite: `.github/workflows/claude-refix.yml`

- [ ] **Step 1: Replace the entire content of `.github/workflows/claude-refix.yml`**

```yaml
name: Claude Refix

on:
  issue_comment:
    types: [created]

permissions:
  contents: write
  issues: write
  id-token: write

jobs:
  refix:
    if: >
      github.event.issue.pull_request == null &&
      contains(github.event.comment.body, '/refix')
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Generate refix branch name
        id: branch
        uses: actions/github-script@v8
        with:
          result-encoding: string
          script: |
            const number = context.issue.number;
            const title = context.payload.issue.title;
            const slug = title
              .toLowerCase()
              .replace(/[^a-z0-9\s-]/g, '')
              .replace(/\s+/g, '-')
              .replace(/-+/g, '-')
              .replace(/^-|-$/g, '')
              .substring(0, 50);
            const timestamp = Date.now();
            return `fix/${number}-${slug}-refix-${timestamp}`;

      - name: Create and push branch
        run: |
          git checkout -b ${{ steps.branch.outputs.result }}
          git push origin ${{ steps.branch.outputs.result }}

      - name: Move issue card to In progress
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          IN_PROGRESS_OPTION_ID: ${{ vars.IN_PROGRESS_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.IN_PROGRESS_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              core.setFailed("Issue is not in the target project.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });

      - name: Claude refix
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          show_full_output: true
          prompt: |
            A tester requested a refix for an existing bug.

            Repository: ${{ github.repository }}
            Issue #${{ github.event.issue.number }}
            Title: ${{ github.event.issue.title }}
            Branch: ${{ steps.branch.outputs.result }}

            Original issue body:
            ${{ github.event.issue.body }}

            Refix comment:
            ${{ github.event.comment.body }}

            Read CLAUDE.md and repository context.

            Task:
            1. Checkout the branch: git checkout ${{ steps.branch.outputs.result }}
            2. Re-analyze the bug using the original issue and the new tester comment.
            3. Identify why the previous fix may have failed.
            4. Implement the smallest safe follow-up fix.
            5. Update or add tests if needed.
            6. Commit all changes directly to the branch and push.
            7. Do NOT create a pull request.
            8. Do NOT merge anything to main.
            9. Keep changes minimal and focused only on this bug.
          claude_args: >-
            --max-turns 20
            --allowedTools Read,Edit,Write,Glob,Grep,WebFetch,WebSearch,Bash(git:*),Bash(gh issue *),Bash(npm:*),Bash(node:*)

      - name: Move issue card to Review
        uses: actions/github-script@v8
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          STATUS_FIELD_ID: ${{ vars.STATUS_FIELD_ID }}
          REVIEW_OPTION_ID: ${{ vars.REVIEW_OPTION_ID }}
        with:
          github-token: ${{ secrets.PROJECT_PAT }}
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const issueNumber = context.issue.number;

            const projectId = process.env.PROJECT_ID;
            const statusFieldId = process.env.STATUS_FIELD_ID;
            const optionId = process.env.REVIEW_OPTION_ID;

            const result = await github.graphql(`
              query($owner:String!, $repo:String!, $number:Int!) {
                repository(owner:$owner, name:$repo) {
                  issue(number:$number) {
                    projectItems(first: 20) {
                      nodes {
                        id
                        project { id }
                      }
                    }
                  }
                }
              }
            `, { owner, repo, number: issueNumber });

            const item = result.repository.issue.projectItems.nodes.find(
              n => n.project.id === projectId
            );

            if (!item) {
              core.setFailed("Issue is not in the target project.");
              return;
            }

            await github.graphql(`
              mutation($projectId:ID!, $itemId:ID!, $fieldId:ID!, $optionId:String!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId,
                  itemId: $itemId,
                  fieldId: $fieldId,
                  value: { singleSelectOptionId: $optionId }
                }) {
                  projectV2Item { id }
                }
              }
            `, {
              projectId,
              itemId: item.id,
              fieldId: statusFieldId,
              optionId
            });
```

- [ ] **Step 2: Commit**

```bash
git add .github/workflows/claude-refix.yml
git commit -m "feat: rewrite claude-refix to use direct branch commits instead of PRs"
```

---

### Task 4: Delete old workflows

**Files:**
- Delete: `.github/workflows/claude-bug-triage.yml`
- Delete: `.github/workflows/project-ready-for-testing.yml`

- [ ] **Step 1: Delete the old triage workflow**

```bash
git rm .github/workflows/claude-bug-triage.yml
```

- [ ] **Step 2: Delete the old project-ready-for-testing workflow**

```bash
git rm .github/workflows/project-ready-for-testing.yml
```

- [ ] **Step 3: Commit**

```bash
git commit -m "chore: remove old triage and project-ready-for-testing workflows"
```

---

### Task 5: Update `CLAUDE.md`

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Replace the content of `CLAUDE.md`**

```markdown
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

## Rules
- Do not change DB schema unless issue explicitly requires it
- Prefer minimal fixes
- Keep API contracts backward compatible
- Add or update tests when fixing bugs

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
- `In progress` — Claude is fixing the bug on the branch
- `Review` — fix committed, developer reviews the branch
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
- `IN_PROGRESS_OPTION_ID`
- `REVIEW_OPTION_ID`
- `READY_FOR_TESTING_OPTION_ID`
- `DONE_OPTION_ID`
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md to reflect v2 branch-based workflow"
```

---

### Task 6: Setup checklist (manual steps)

These are GitHub configuration steps that must be done manually in the repository settings.

- [ ] **Step 1: Add new board statuses in GitHub Project**
  - Add `In progress` status to the project board
  - Add `Review` status to the project board
  - Remove `Pending approval` status (or keep for backward compatibility)

- [ ] **Step 2: Get the new option IDs**
  - Use the GitHub GraphQL API to get the option IDs for `In progress` and `Review`
  - Update repository variables:
    - Add `IN_PROGRESS_OPTION_ID`
    - Add `REVIEW_OPTION_ID`
    - Remove `PENDING_APPROVAL_OPTION_ID`

- [ ] **Step 3: Create the `branch-approved` label**
  - Create label `branch-approved` in the repository

- [ ] **Step 4: Remove the `claude-fix-approved` label (optional)**
  - Remove label `claude-fix-approved` if no longer needed
