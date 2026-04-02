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
- First analyze likely root cause
- Mention impacted files/modules
- Suggest minimal safe fix
- Only create code changes after explicit approval