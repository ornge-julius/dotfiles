---
applyTo: '**'
---
Provide project context and coding guidelines that AI should follow when generating code, answering questions, or reviewing changes.

- Always avoid introducing bugs or breaking changes.
- Preserve existing behavior unless the task explicitly requires change; document intentional behavior changes.
- Prefer minimal, targeted edits; avoid formatting-only churn or unrelated refactors.
- Understand context before editing: locate relevant code paths, call sites, and configuration.
- Maintain API stability (public methods, routes, payloads, and schemas); deprecate rather than remove.
- Add or update tests when behavior changes or regressions are possible; keep tests focused and reliable.
- Respect existing architecture, naming conventions, and style; follow project patterns.
- Handle error cases explicitly; avoid silent failures and ensure actionable error messages.
- Validate assumptions with existing code or docs; do not guess behavior when evidence exists.
- Prefer standard library and existing dependencies; add new dependencies only when clearly justified.
- Keep performance and memory impacts in mind; avoid $O(n^2)$ where $O(n)$ is sufficient for expected data sizes.
- Ensure security best practices: validate inputs, avoid injection, and protect secrets.
- Be careful with data migrations and background jobs: ensure backward compatibility and safe rollouts.
- Avoid breaking changes in configuration, environment variables, or deployment scripts.
- Keep changes reversible when possible; avoid destructive operations without explicit approval.
- Ensure logs are useful but not noisy; never log secrets or sensitive data.
- Prefer explicitness over magic; minimize global state and side effects.
- When updating interfaces, update all usages consistently.
- Keep documentation accurate when behavior, configuration, or usage changes.