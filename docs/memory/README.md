# Durable Project Memory

Reviewed, reusable project knowledge, stored in Git so it survives across GitHub Actions runs.

## Taxonomy

- `project-memory.md` — concise index and critical invariants.
- `module-map.md` — module responsibilities, dependencies and entry points.
- `known-problems.md` — confirmed unresolved or recurring issues.
- `failed-approaches.md` — approaches that were tried and failed, plus the preferred alternative.
- `coding-patterns.md` — approved reusable implementation patterns.

## Promotion pipeline

```text
Observation
  -> docs/tasks/issue-N.md          (working memory, may contain unverified notes)
  -> verified by code or a run      (not by assumption)
  -> docs/handoffs/issue-N.md       (continuation state)
  -> docs/memory/ or docs/decisions/ (durable, only if reusable)
  -> human review via `branch-approved`
  -> merged to main
  -> loaded by every future run
```

## Visibility rule

A future issue run starts from `main`. Memory written on an unmerged branch is invisible to it.
Only merged memory is global truth.

## Never store here

Secrets, tokens, raw logs, temporary environment values, personal data, or a root cause that was
guessed rather than confirmed.
