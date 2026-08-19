# Database Rules

Applies to: `database/**`, `db/**`, `migrations/**`, `*.sql`, `*migration*`.
Skip this file when the issue does not touch the database.

- Do not change the schema unless the issue explicitly requires it (see `CLAUDE.md`).
- Never modify an applied migration; add a new one.
- Every schema change needs forward behavior, rollback/mitigation notes and data-impact analysis.
- Avoid full-table rewrites and long locks without an explicit operational plan.
- Preserve tenant/user scope and authorization in every query.
- Use parameterized queries; never concatenate untrusted input into SQL.
- Consider indexes, query plans, cardinality and connection-pool impact.
- Record irreversible changes and operational prerequisites in the handoff.
