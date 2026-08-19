# Frontend Rules

Applies to: `src/app/**`, `frontend/**`, `web/**`, `*.component.ts`, `*.html`, `*.scss`, and the
standalone HTML pages at the repository root.
Skip this file when the issue does not touch UI code.

- Preserve accessibility, keyboard navigation, focus order and readable error states.
- Do not hide missing or failed data without a clear user-visible state.
- Keep business rules out of presentation-only components where possible.
- Avoid duplicate subscriptions, listeners, timers and socket handlers; unsubscribe on destroy.
- Verify loading, empty, error and permission-restricted states, not just the happy path.
- Keep API models typed and handle nullable/optional fields explicitly.
- When a page links to another page, confirm the target actually exists before shipping the link.
- Add or update component tests and affected end-to-end coverage where a harness exists.
