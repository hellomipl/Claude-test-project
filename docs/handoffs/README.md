# Agent Handoffs

Each issue gets a continuation-ready handoff:

```text
docs/handoffs/issue-123.md
```

A handoff is not an activity log. It is the minimum complete state another engineer or agent needs
to continue safely without the previous conversation.

Write it before the run ends, and whenever work stops while incomplete.

Why this matters here: `/refix` creates a **new branch from `main`**, not a continuation of the
previous fix branch. A handoff that was never merged is invisible to the refix run. If a fix is
likely to need a second pass, the handoff must be committed and the branch merged for that context
to survive.
