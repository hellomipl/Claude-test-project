# Backend and API Rules

Applies to: `src/**`, `server/**`, `api/**`, `backend/**` — NestJS controllers, services, guards, DTOs.
Skip this file when the issue does not touch backend code.

- Keep transport/controller logic separate from business logic in services.
- Validate all external input at the boundary (DTO + validation pipe), not deep in a service.
- Preserve stable API contracts unless the issue authorizes a versioned change.
- Return consistent error shapes; never leak internal exceptions or stack traces to clients.
- Add timeouts, cancellation and bounded retries for outbound calls.
- Make retryable operations idempotent where practical.
- Do not introduce unbounded concurrency, queues or connection pools.
- Auth changes must keep JWT and refresh-token handling separate: verify the access path and the
  refresh path independently.
- Add tests for success, validation failure, authorization failure and important edge cases.
