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

### KP-0005 — Board column names in the docs do not match the live board

- Status: Open
- First confirmed: 2026-08-19
- Affected modules: `CLAUDE.md`, `docs/context.md`, all three workflows

**Observed behavior**

`CLAUDE.md` lists the second board status as "`Review` (was Pending approval)", implying the column
was renamed. It was not. The live board's Status field offers exactly four options:
`To triage`, `Pending approval`, `Ready for testing`, `Done`.

**Confirmed evidence**

GraphQL query against user project 5 on 2026-08-19 returned the option set above, with
`Pending approval` = `f75ad846` — the value supplied as `PENDING_APPROVAL_OPTION_ID`.

**Why it is harmless today**

The workflows address the column by option **ID**, never by name, so the status moves work
regardless of the label mismatch.

**Why it still matters**

Anyone reading `CLAUDE.md` will look for a `Review` column that does not exist, and may "fix" the
apparent inconsistency by renaming the variable or the column, which would break all three workflows.

**Correct long-term direction**

Pick one: rename the board column to `Review`, or correct the line in `CLAUDE.md`. Then delete
this entry.

---

### KP-0006 — The agent cannot post its triage comment; `gh issue comment` is refused

- Status: **Open**
- First confirmed: 2026-08-19 (issue #16), still reproducing on issue #17
- Affected modules: `.github/workflows/claude-bug-fix.yml`, `.github/workflows/claude-refix.yml`

**Observed behavior**

Step 2 of both workflow prompts tells the agent to post its triage analysis as an issue comment.
It composes the comment and calls `gh issue comment <n> --repo … --body "$(cat <<'EOF' … EOF)"`.
Every attempt returns `This command requires approval`. The run then continues and finishes
**green**, so the missing comment is invisible unless you check the issue.

Issues #16 and #17 both have zero comments.

**Confirmed evidence** (run 32227877961, 20 denials total)

| Command | Pattern intended to cover it | Result |
|---|---|---|
| `git commit -m "$(cat <<'EOF' …)"` | `Bash(git:*)` | allowed |
| `gh issue comment 17 --body "$(cat …)"` | `Bash(gh issue:*)` | denied ×8 |
| `gh auth status` | none | denied ×4 (expected) |
| `python3 /tmp/check_html.py` | none | denied ×8 (expected) |

**What has been ruled out**

- *Not* the space-vs-colon separator. `Bash(gh issue *)` was changed to `Bash(gh issue:*)` in
  commit `7301dd1` and the denial is unchanged. An earlier entry in `coding-patterns.md` claimed
  the separator was the proven cause; that claim was wrong and has been corrected.
- *Not* command substitution. The allowed `git commit` above contains `$(cat <<'EOF' …)`.

**Leading hypothesis, not yet isolated**

Only the *single-word* prefix matched. `Bash(git:*)` worked; the two-word `Bash(gh issue:*)` did
not. Predicted fix: `Bash(gh:*)`. Untested — and it widens access to every `gh` subcommand, so
weigh that before adopting it.

**Second, independent candidate cause**

The Claude step passes no `GH_TOKEN`/`GITHUB_TOKEN` in an `env:` block, so `gh` may be
unauthenticated in the runner regardless of permissions. If so, fixing the pattern alone will
surface an auth error instead of a comment. Both need checking together.

**Current mitigation**

None. Treat the triage comment as not happening. The task file and handoff on the issue branch
carry the same analysis and are committed, so no information is actually lost.
