# Project Configuration Rule

Before running install, build, test or lint commands, read the `## Commands` section of `CLAUDE.md`.

- Use the configured commands when they exist and actually run.
- Verify the command is runnable before relying on it. In this repository the declared commands
  (`npm install`, `npm test`, `npm run lint`, `npm run build`) currently have no `package.json`
  behind them — see `docs/memory/known-problems.md` (KP-0001).
- If a configured command cannot run, do not silently skip verification. Record in the task file
  and handoff: the command attempted, the failure, what was validated instead, and the residual risk.
- If a command is missing, inspect `package.json`, `Makefile` or CI workflows before guessing.
- Do not edit `CLAUDE.md` command entries until the replacement command has been executed successfully.
- Never run production deployment or migration commands from an issue workflow.
