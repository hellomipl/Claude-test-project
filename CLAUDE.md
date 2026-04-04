# Project context

## Architecture
- Backend: NestJS
- Frontend: Angular
- DB: PostgreSQL
- Auth: JWT + refresh tokens

## Commands
- Install: npm install
- Test: npm test
- Lint: npm run lint
- Build: npm run build

## Rules
- Do not change DB schema unless issue explicitly requires it
- Prefer minimal fixes
- Keep API contracts backward compatible
- Add or update tests when fixing bugs

## Bug-fix workflow
- When a bug issue is created, a branch is automatically created from main
- Claude analyzes the bug and posts a triage comment on the issue
- Claude implements the fix and commits directly to the issue branch
- Do NOT create pull requests
- Do NOT merge to main
- Commit and push all changes to the issue branch only
- Developer reviews the branch and adds label `branch-approved` to trigger merge to main
- Keep changes minimal and focused only on the reported bug

## GitHub Project board statuses
- `To triage` — issue created, Claude is analyzing
- `In progress` — Claude is fixing the bug on the branch
- `Review` — fix committed, developer reviews the branch
- `Ready for testing` — branch merged to main, QA pending
- `Done` — tester verified fix works

## GitHub Action variables needed
### Secrets
- `CLAUDE_CODE_OAUTH_TOKEN`
- `PROJECT_PAT`

### Variables
- `PROJECT_ID`
- `STATUS_FIELD_ID`
- `TO_TRIAGE_OPTION_ID`
- `IN_PROGRESS_OPTION_ID`
- `REVIEW_OPTION_ID`
- `READY_FOR_TESTING_OPTION_ID`
- `DONE_OPTION_ID`
