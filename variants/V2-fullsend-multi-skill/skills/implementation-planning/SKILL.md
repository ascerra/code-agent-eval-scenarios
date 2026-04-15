---
name: implementation-planning
description: >-
  Implementation planning phase: check for existing branches, create feature
  branch, and form a concrete implementation plan before writing code.
---

# Implementation Planning

With context gathered and repository conventions understood, create the
appropriate branch and plan exactly what changes will be made. Proper
planning prevents scope creep and ensures all necessary tests are identified
before implementation begins.

## Tools reminder

You have the `Bash` tool for all CLI operations. Commands you will need
during this phase:

- `git checkout`, `git branch`, `git fetch` — branching operations
- `gh pr view`, `gh api` — reading cross-repo references
- Additional git operations for branch management

Use `Read`/`Write`/`Grep`/`Glob` for file operations.

## Process

Follow these steps in order. Do not skip steps.

### 4. Check for existing branch

Before creating a new branch, check whether a branch already exists for this
issue from a previous run:

```bash
git branch -a | grep "agent/<number>-"
```

**If a branch exists:** Check it out and work on top of it.

**If no branch exists:** Proceed to step 5.

### 5. Create branch

If the `BRANCH_NAME` environment variable is set, use it:

```bash
git fetch origin
git checkout -b "${BRANCH_NAME}" origin/<target-branch>
```

Otherwise, create a feature branch from the target branch:

```bash
git fetch origin
git checkout -b agent/<number>-<short-description> origin/<target-branch>
```

The branch name must follow the `agent/<issue-number>-<short-description>`
convention. Keep the description to 2-4 lowercase hyphenated words derived
from the issue title.

### 6. Plan the implementation

Before writing code, form a concrete plan:

1. **Read affected files in full** — not just the lines mentioned in the issue.
   Understand the surrounding context, imports, types, and call sites.
2. **Read test files** that cover the affected code. Understand how the existing
   tests are structured, what patterns they follow, what helpers exist.
3. **Read related files** — if the change touches an API handler, read the
   router, middleware, and model files. If it touches a controller, read the
   reconciler pattern and RBAC config.
4. **Follow cross-repo references** — if the issue, docs, or triage comments
   link to other repos (e.g., an e2e test suite, a dependent service, a
   related PR in another repo), read those references to understand the full
   picture. Use `gh issue view`, `gh pr view`, or
   `gh api repos/{owner}/{repo}/contents/{path}` to fetch what you need.
   Do not chase every import — focus on references that the issue context
   points you toward.
5. **Identify what to change** — list the specific files and functions you will
   modify or create.
6. **Identify what tests to write or update** — new behavior needs new tests;
   changed behavior needs updated tests.
7. **Assess risk** — will this change affect other callers? Does it change a
   public interface? Could it break downstream consumers?

Do not start writing code until you can articulate: what you will change, why,
and how you will verify it works.

## Completion

When your plan is complete, proceed to the `code-writing` skill.
