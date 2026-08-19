# Documentation and Durable Memory Rules

The Git repository is the durable source of truth. A GitHub Actions runner is temporary; only
committed and pushed files survive.

Promote information into `docs/memory/` only when it is:

- Verified by code, tests, logs or an approved decision — not speculation.
- Reusable in a future task, not specific to one issue.
- Not already documented.
- Free of secrets, private data and temporary environment values.
- Statable concisely as a fact, constraint, pattern or confirmed failure mode.

Destinations:

- `docs/memory/project-memory.md` — concise index and repository invariants.
- `docs/memory/module-map.md` — module ownership, dependencies and entry points.
- `docs/memory/known-problems.md` — confirmed unresolved or recurring issues.
- `docs/memory/failed-approaches.md` — approaches actually attempted that failed, plus the better alternative.
- `docs/memory/coding-patterns.md` — approved reusable implementation patterns.
- `docs/decisions/` — significant decisions with alternatives and consequences.

Keep issue-specific narrative in `docs/tasks/issue-<number>.md` and `docs/handoffs/issue-<number>.md`.
Do not promote it to global memory.

If nothing qualifies, write `Memory impact: none` in the handoff. That is a valid and common outcome.
