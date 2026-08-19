# Git and Branch Rules

This repository uses a branch-based review flow. There are no agent-created pull requests.

- Default branch is `main`. Never commit to it directly.
- Branch format is created by the workflow: `fix/<issue-number>-<slug>`.
  Refix runs use `fix/<issue-number>-<slug>-refix-<timestamp>`.
- Always work on the branch the workflow checked out. Do not create an extra branch.
- Commit logical checkpoints; avoid one-file-per-commit noise.
- Commit message format: `fix: <imperative summary> (#<issue-number>)`.
- Push every commit. Uncommitted runner files are lost when the run ends and are not memory.
- Never force push. Never hard reset.
- Do NOT create a pull request. Do NOT merge to `main`.
- A human merges by applying the `branch-approved` label, which triggers
  `.github/workflows/branch-approve-merge.yml`. That workflow merges the branch and deletes it.
- Because the branch is deleted on merge, anything not committed to it is gone. Commit the task
  file and handoff before the run ends.
