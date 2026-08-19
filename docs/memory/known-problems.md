# Known Problems

Confirmed unresolved or recurring problems. Mark one resolved only through a reviewed, merged change.

### KP-0001 — Declared build and test commands have nothing behind them

- Status: Open
- First confirmed: 2026-08-19
- Affected modules: every agent run

**Observed behavior**

`CLAUDE.md` declares `npm install`, `npm test`, `npm run lint` and `npm run build`. There is no
`package.json` anywhere in the repository, so all four fail immediately.

**Confirmed evidence**

No `package.json`, lockfile or `node_modules` exists at the repository root.

**Current mitigation**

Do not report a fix as "tested". State plainly in the handoff that no test harness exists, and
describe what was verified instead — for example, reading the changed file end to end, or
confirming a link target actually exists.

**Correct long-term direction**

Either add a real `package.json` with runnable scripts, or replace the `## Commands` block in
`CLAUDE.md` with the honest current state, so the agent stops being instructed to run commands
that cannot work.

---

### KP-0002 — No human approval gate before the agent runs

- Status: Open
- First confirmed: 2026-08-19
- Affected modules: `.github/workflows/claude-bug-fix.yml`

**Observed behavior**

The workflow triggers on `issues: [opened]`, with only `if: github.event.issue.pull_request == null`
on the job. Any newly opened issue immediately starts an agent run holding `contents: write` and
`issues: write`. Earlier design notes in `docs/context.md` describe a `claude-fix-approved` label
gate that does not exist in the current workflow.

**Confirmed evidence**

The `on:` block and the job `if:` condition in `claude-bug-fix.yml`.

**Current mitigation**

None in code. Issue bodies are author-controlled text and must be treated as untrusted input, per
`.claude/rules/50-security.md`.

**Correct long-term direction**

Move the trigger to `issues: [labeled]` gated on an explicit approval label, matching the
documented intent. This is a deliberate policy change — record it as an ADR first.

---

### KP-0003 — Branch-prefix match is ambiguous once a refix branch exists

- Status: Open
- First confirmed: 2026-08-19
- Affected modules: `.github/workflows/branch-approve-merge.yml`

**Observed behavior**

The merge workflow picks the first branch whose name starts with `fix/<issue>-`, in whatever order
the branches API returns. After a `/refix` run, both `fix/15-title` and
`fix/15-title-refix-<timestamp>` match that prefix, so the merge can select the stale branch.

**Confirmed evidence**

`branch-approve-merge.yml` builds the prefix `fix/<issueNumber>-` and resolves it with a `.find()`,
while `claude-refix.yml` creates `fix/<number>-<slug>-refix-<timestamp>`.

**Current mitigation**

Only reproduces when the original branch is still unmerged at the moment a refix branch exists.
Check the branch name printed in the merge workflow log before trusting the result.

**Correct long-term direction**

Select the most recently committed matching branch, or make the refix flow reuse the original
branch instead of creating a sibling.

---

### KP-0004 — `docs/context.md` contradicts the implemented workflow

- Status: Open
- First confirmed: 2026-08-19
- Affected modules: documentation

**Observed behavior**

`docs/context.md` describes a pull-request-based flow: a `claude-fix-approved` label, Claude opening
PRs, a `project-ready-for-testing.yml` workflow, and a `Pending approval` board column. None of that
matches the current branch-based, no-PR implementation in `CLAUDE.md` and the three workflow files.
`project-ready-for-testing.yml` does not exist.

**Current mitigation**

Treat `CLAUDE.md` and `.github/workflows/` as authoritative. Do not follow `docs/context.md`.

**Correct long-term direction**

Rewrite or delete `docs/context.md`. It also holds the project and field IDs in plaintext, which are
now supplied as Actions variables.

---

### KP-0005 — `gh` CLI commands are blocked by the approval gate in an unattended run

- Status: Open
- First confirmed: 2026-08-19 (issue #16)
- Affected modules: any agent step that shells out to `gh` (e.g. posting a triage comment)

**Observed behavior**

`gh auth status` and `gh issue comment` both returned `This command requires approval` with no
prompt able to reach a human, even with `dangerouslyDisableSandbox` set. No `.claude/settings.json`
or `.claude/settings.local.json` exists to pre-allow these commands. As a result, the agent could not
post the triage comment described in the bug-fix workflow.

**Confirmed evidence**

Direct tool output during issue #16: repeated `gh auth status` and `gh issue comment` calls both
returned the approval-required error; a plain `echo` in the same session succeeded immediately.

**Current mitigation**

Do not block the rest of the run on a failed `gh` call. Record the blocker in the task file and
handoff, and continue with the branch/commit work, which uses `git` (not `gh`) and is unaffected.

**Correct long-term direction**

Add an explicit allow-rule for the specific `gh` invocations the bug-fix workflow needs (e.g.
`gh issue comment`) to `.claude/settings.json`, so unattended runs don't stall on an approval prompt
nobody can answer.
