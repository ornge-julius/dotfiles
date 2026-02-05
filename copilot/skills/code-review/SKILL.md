---
name: code-review
description: Reviews recent git commits on the current branch for bugs, security issues, and regressions, then writes findings and fixes to code-review.md. Use when the user asks for a code review of recent changes or wants a review based on git commits.
---

# Code Review (Recent Changes)

## Scope
- Default scope is commits on the current branch compared to the base branch.
- If the user asks, also include staged and unstaged working tree changes.
- If multiple repositories are open, confirm which repo to review.

## Workflow
1. Identify the repo root for the active workspace and confirm the target repo if needed.
2. Determine the base branch:
   - Prefer the remote default branch (origin/HEAD).
   - Fallback to main, then master.
3. Collect recent changes:
   - List commits in base..HEAD.
   - Review the full diff for base..HEAD.
   - If the user requested working tree changes, include staged/unstaged diffs.
4. Review for:
   - Correctness and edge cases
   - Security risks (injection, auth, secrets, unsafe deserialization)
   - Data integrity and backward compatibility
   - Performance regressions and avoidable O(n^2)
   - Missing tests or insufficient coverage
   - Logging quality (useful, not noisy, no secrets)
5. For Ruby/Rails changes, ensure adherence to the project's established style and conventions as documented in the ruby-rails-style-guide skill.
6. Write results to code-review.md in the repo root.

## Output format (code-review.md)
Use this template:

```markdown
# Code Review

## Scope
- Repo: <repo name>
- Base branch: <base branch>
- Compared range: <base>..HEAD
- Included working tree: <yes|no>

## Findings
- [Severity] <short title>
  - Location: <file path and lines>
  - Issue: <what is wrong>
  - Risk: <why it matters>
  - Fix: <concrete fix or patch idea>

## Tests
- Ran: <tests or none>
- Gaps: <missing tests or areas to add>

## Notes
- <optional additional context>
```

## Expectations
- If no issues are found, write "No findings" under Findings.
- Keep fixes concrete and actionable.
- Keep the review comprehensive but focused on defects and risks.
