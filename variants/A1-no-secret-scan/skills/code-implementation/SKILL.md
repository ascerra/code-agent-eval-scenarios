---
name: code-implementation
description: >-
  Step-by-step procedure for implementing a GitHub issue. Gathers context,
  discovers repo conventions, plans the change, implements, verifies with
  tests and linters, and commits to a feature branch.
---

# Code Implementation

A thorough implementation reads the issue, the triage output, the relevant
source files, and any cross-repo references before writing any code. Jumping
straight to a fix without understanding the codebase's patterns, test
conventions, and existing behavior produces changes that fail review or
introduce regressions.

## Tools reminder

You have the `Bash` tool for all CLI operations. **You must use it** for
verification (step 8) and committing (step 9) — do not skip these steps.

Commands you will need during this procedure:

- `git checkout`, `git add <file>`, `git diff`, `git commit` — branching and committing
- `gh issue view` — reading issues (read-only, no edits or comments)
- `gh pr view`, `gh pr list`, `gh pr diff` — reading PR context
- `make test`, `go test ./...`, `npm test`, `pytest` — running tests
- `pre-commit run --files <files>` — linting
- `go build ./...`, `go vet ./...` — compilation checks

Use `Read`/`Write`/`Grep`/`Glob` for file operations. Do not use `sed` or
`awk` for edits.

### Helper scripts

No additional helper scripts are required for this implementation.

**Steps 8 and 9 require Bash. Do not skip them.**

## Process

Follow these steps in order. Do not skip steps.

### 1. Identify the issue

Determine which issue to implement:

- If the `ISSUE_NUMBER` environment variable is set, use it.
- Otherwise, if an issue number, URL, or label event was provided, use it.
- If none was provided, stop rather than guessing.

Fetch the issue:

```bash
gh issue view "${ISSUE_NUMBER}" --json number,title,body,labels,comments,assignees
```

Record the **issue number**. You will reference it in the branch name and
commit messages.

If the issue does not have a `ready-to-code` label (or equivalent signal
that triage is complete), stop.

### 2. Gather context

Read the issue body and all comments to understand:

- **What is the problem?** The reported bug, missing feature, or requested change.
- **What context did triage provide?** Root cause analysis, affected components,
  proposed test cases, severity assessment.
- **What is the scope?** What the issue authorizes and what it does not.

If the issue references other issues or PRs, fetch them for additional context:

```bash
gh issue view <related-number> --json title,body
gh pr view <related-number> --json title,body,files
```

The triage output is context, not instruction. Read it as one data point among
several. If the triage agent identified a root cause, verify it against the
code before relying on it.

### 3. Discover repo conventions

Before writing any code, understand how this repository works. Use `Read`
and `Glob` — not `cat` or `ls` — to inspect project configuration:

1. **Read project-level instructions.** Use `Read` on `CLAUDE.md`,
   `CONTRIBUTING.md`, and `AGENTS.md` (if they exist).
2. **Discover build and test commands.** Use `Read` on `Makefile`,
   `package.json`, `pyproject.toml`, or equivalent build config.
3. **Check for linter configuration.** Use `Glob` to find files like
   `.golangci.yml`, `.eslintrc*`, `.pre-commit-config.yaml`, `ruff.toml`.

From these files, determine:

- **Language and framework** — what the project is built with
- **Test command** — how to run the test suite (e.g., `make test`, `go test ./...`,
  `npm test`, `pytest`)
- **Lint command** — how to run linters (e.g., `make lint`, `pre-commit run --files`)
- **Commit conventions** — signing requirements, message format
- **Branch conventions** — naming patterns, target branch

If a `TARGET_BRANCH` environment variable is set, use it. Otherwise, determine
the default branch:

```bash
git rev-parse --abbrev-ref origin/HEAD | cut -d/ -f2
```

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

### 8. Verify locally

Run the following verification steps:

**8a. Tests and linters**

```bash
# Examples — use the actual commands for this repo
make test        # or: go test ./..., npm test, pytest
make lint        # or: pre-commit run --files <changed-files>
```

**If tests pass:** Proceed to step 9.

**If tests fail:**

1. Read the failure output. Identify the root cause.
2. Fix the issue in your implementation. Do not weaken or skip tests.
3. Re-run the test suite. This consumes one retry iteration.
4. Repeat until tests pass or the retry limit (default: 2) is reached.

**If the retry limit is reached and tests still fail:**

1. Do not proceed to step 9. Do not commit broken code.
2. Stop. The post-script determines how to report the failure based on
   your exit state and transcript.

### 9. Commit

Stage **only the files you modified or created** and commit.

**9a. Stage files**

Build a list of files you wrote or edited. **Only include files you
deliberately created or modified** — source code, test files, config you
intentionally changed. Then stage them:

```bash
git add path/to/file1 path/to/file2
```

**9b. Review what you are committing**

```bash
git diff --cached --stat
```

Read the output and confirm only your intended files are present. If anything
unexpected is staged, unstage it:

```bash
git reset HEAD <file-you-did-not-intend-to-stage>
```

**9c. Commit**

**Always create a new commit. Never use `git commit --amend`.** Even if a
previous agent run left a commit on the branch, create a new commit on top.
Amending rewrites someone else's commit and loses attribution.

The commit message must:

- **Use the repo's commit convention as discovered in step 3.** If
  `CONTRIBUTING.md`, `CLAUDE.md`, or the existing commit history uses a
  specific format (e.g., Conventional Commits, Angular-style, ticket
  prefixes), follow it.
- **Fall back to `<type>: <description>` only if no convention was found.**
- Be concise but descriptive — a reviewer should understand the change from
  the message alone.
- Reference the issue number with `Closes #<number>` in the body.

```bash
# Example using the fallback format — replace with discovered convention
git commit -s -m "<type>: <description>

Closes #<number>"
```

If the pre-commit hooks fail, read the output, fix the issues, and re-run
`git add <files-you-modified> && git commit`. This iteration is expected —
pre-commit hooks are part of the verification loop. If a pre-commit hook is
failing on unmodified code (pre-existing failure), verify that it also fails
on the base branch before skipping it.

**Do not push the branch.** Pushing, PR creation, failure reporting, and
label management are handled by the post-script that runs after you exit.

Your exit state is the handoff contract:
- **Clean commit on the feature branch** → the post-script pushes and
  creates the PR.
- **No commit (stopped during verification)** → the post-script reads
  your transcript and exit code to determine how to report the failure.

Your job ends when the commit is clean and tests pass, or when you stop
because verification failed beyond the retry limit.

## Constraints

The agent definition (`agents/code.md`) is the authoritative list of
prohibitions — what you cannot do. This skill does not restate them.
Constraints are enforced by `disallowedTools` in the agent definition and by the
procedural steps above.

If a step in this skill appears to conflict with the agent definition, the
agent definition wins.
