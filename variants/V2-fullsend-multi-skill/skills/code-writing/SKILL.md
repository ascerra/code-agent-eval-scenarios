---
name: code-writing
description: >-
  Code implementation phase: write the actual changes following repository
  patterns, include necessary tests, and prepare for verification.
---

# Code Writing

With a concrete plan in place, implement the change following the repository's
existing patterns and conventions. Every behavioral change must include
corresponding tests to prevent regressions.

## Tools reminder

You have the `Bash` tool for compilation checks and basic validation.

Use `Read`/`Write`/`Grep`/`Glob` for file operations. Do not use `sed` or
`awk` for edits.

## Process

### 7. Implement the change

Write the code change:

- **Follow existing patterns.** If the repo uses a specific error handling idiom,
  use it. If controllers follow a specific reconciliation pattern, follow it. If
  test files use a specific helper library, use it.
- **Do not introduce new dependencies without justification.** If the change can
  be made with the existing dependency set, prefer that.
- **Do not refactor adjacent code.** Keep changes scoped to what the issue
  authorizes. If you notice problems in nearby code, note them in the commit
  message as follow-up work — do not fix them in this change.
- **Write or update tests.** Every behavioral change must have a corresponding
  test change. If the issue includes a proposed test case from triage, evaluate
  it critically — use it if it's good, improve it if it's not, replace it if
  it's wrong.
- **Document non-obvious changes.** If the fix involves a subtle invariant or
  a non-obvious design choice, add a code comment explaining why.

## Completion

When implementation is complete, proceed to the `verification` skill.
