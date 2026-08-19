# Testing and Verification Rules

- Map each acceptance criterion to a code change and a verification method.
- Prefer focused tests first, then broader regression commands.
- Do not claim a test passed unless the command actually completed successfully.
- Record exact commands and their real results in the task file and handoff.
- If a test cannot run, record why, what was validated instead, and the residual risk. Never let
  "no test suite exists" read as "verified".
- A mocked test proves your code builds the right request, not that the real system accepts it.
  Say so explicitly when mocks are the only evidence.
- Add a regression test for a confirmed bug when a test harness exists.
- Do not delete or weaken existing tests to make a change pass.
- Treat a flaky test as a documented risk, not an automatic pass.
