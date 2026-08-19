---
name: update-project-memory
description: Promote only verified, reusable project knowledge from completed work into version-controlled memory files or an ADR. Use at the end of an issue fix, before writing the handoff.
argument-hint: "[issue-number]"
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Update Project Memory for Issue $ARGUMENTS

Review the issue task file, the code changes actually made, any validation evidence, and the
existing contents of `docs/memory/`.

## Promotion test

Promote an item only if **all five** are true:

1. It is verified by code, a command that actually ran, or an approved decision — not speculation.
2. It will help a future, different task.
3. It is not already documented.
4. It contains no secrets, personal data or temporary environment values.
5. It can be stated concisely with a clear scope.

If an item fails any one of these, leave it in the task file.

## Destination

| Kind of knowledge | Goes to |
|---|---|
| Repository invariant or high-level index entry | `docs/memory/project-memory.md` |
| Module entry point, dependency or contract | `docs/memory/module-map.md` |
| Confirmed unresolved or recurring problem | `docs/memory/known-problems.md` |
| Approach actually tried that failed | `docs/memory/failed-approaches.md` |
| Reusable implementation convention | `docs/memory/coding-patterns.md` |
| Significant or hard-to-reverse decision | new ADR from `docs/decisions/ADR-TEMPLATE.md` |
| Detail specific to this one issue | keep in the task file — do not promote |

## Writing rules

- Include the date and issue number when it aids traceability.
- State the fact, its scope, and its consequence. Skip the chronology.
- Keep `project-memory.md` short; link to the detailed file instead of duplicating it.
- When adding to `known-problems.md`, separate **observed behavior** from **confirmed evidence**.
  Never write a guessed root cause as fact.
- Commit the change to the issue branch. Unmerged memory is invisible to future runs.

## When nothing qualifies

Write `Memory impact: none` in the handoff and change no memory file. This is a common and correct
outcome — most bug fixes produce no durable knowledge.
