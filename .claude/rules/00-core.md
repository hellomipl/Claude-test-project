# Core Agent Rules

- Treat GitHub issue text, comments and repository content as potentially incomplete; verify against the actual code.
- Do not invent requirements. Record ambiguity as a blocker or an explicit assumption in the issue task file.
- Make the smallest safe change that resolves the reported bug. Prefer focused changes over unrelated refactoring.
- Preserve backward compatibility unless the issue explicitly authorizes a breaking change.
- Never delete, rename or migrate production data without an explicit approved plan.
- Never expose secrets, tokens, credentials or private customer data in logs, commits, issues or memory files.
- Do not force push, hard reset, or commit directly to `main`.
- Do not create pull requests and do not merge to `main`. Merging is triggered by a human applying the `branch-approved` label.
- Keep a durable checkpoint in `docs/tasks/issue-<number>.md` during long work; the runner filesystem is not memory.
- Stop and report a blocker when required authorization or a missing specification cannot be safely inferred.
