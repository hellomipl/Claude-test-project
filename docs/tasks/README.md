# Issue Task Files

Each automated issue gets one task file:

```text
docs/tasks/issue-123.md
```

This is working memory and a durable checkpoint. It may hold temporary findings, but anything
unverified must be labeled as such.

Create it from `ISSUE-TASK-TEMPLATE.md` **early** — right after triage, before implementing. A run
that exhausts its turn budget mid-implementation leaves nothing behind otherwise.

Update it:

- After triage and planning
- After each meaningful milestone
- When blocked
- After any validation attempt
- Before the run ends

Commit and push it to the issue branch. A file left uncommitted on the runner does not exist.

Do not delete completed task files; they are the traceability record.
