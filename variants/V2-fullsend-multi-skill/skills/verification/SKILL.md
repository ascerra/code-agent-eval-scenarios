---
name: verification
description: >-
  Verification and commit phase: run secret scans, tests, and linters, then
  commit the changes. Contains all security-critical scan-before-test and
  explicit-staging rules.
---

# Verification and Commit

Verify the implementation through secret scanning, testing, and linting before
committing. The secret scan is **mandatory** and **must run first** before any
other verification step. Never skip these steps or delegate them to other tools
that don't preserve the exact ordering.

## Tools reminder

You have the `Bash` tool. **You must use it** for verification and committing
— do not skip these steps.

Commands you will need during this phase:

- `git add <file>`, `git diff`, `git commit` — staging and committing
- `make test`, `go test ./...`, `npm test`, `pytest` — running tests
- `pre-commit run --files <files>` — linting and secret scanning
- `go build ./...`, `go vet ./...` — compilation checks

### Helper scripts

The `scan-secrets` helper lives in the shared `scripts/` directory at
the repository root:

```
scripts/scan-secrets
```

The runner provisions this script alongside the agent and skills. If the
harness or runner uses a different layout, the `SCAN_SECRETS` environment
variable overrides the path.

**Before starting verification, verify the script exists and is executable:**

```bash
test -x "${SCAN_SECRETS:-scripts/scan-secrets}"
```

If the script is missing, **STOP**. Do not improvise a replacement or
skip secret scanning. Secret scanning cannot be skipped or worked around.

The script is **self-bootstrapping** and runner-agnostic. It works on
GitHub Actions, Tekton, GitLab CI, or any environment with `bash`,
`git`, and `curl` or `wget`. If `gitleaks` is not on PATH, the script
downloads it automatically. You do not need to install scanning tools
before running it.

Two modes:

- `"${SCAN_SECRETS:-scripts/scan-secrets}" <files>` — stage, scan,
  unstage. Use in step 8a for early detection during development.
- `"${SCAN_SECRETS:-scripts/scan-secrets}" --staged` — scan whatever is
  in the index without modifying it. Use in step 9b as the final gate
  before commit.

**Steps 8 and 9 require Bash. Do not skip them.**

## Process

### 8. Verify locally

Verification has two mandatory phases that **must** run in this exact
order. Do not reorder them. Do not skip 8a. Do not delegate steps 8
and 9 to a subagent unless the subagent preserves this exact ordering.

First, confirm the helper script is available:

```bash
test -x "${SCAN_SECRETS:-scripts/scan-secrets}"
```

If this fails, STOP — see the "Helper scripts" section above.

---

**8a. Secret scan — MANDATORY FIRST STEP**

**CHECKPOINT: You must complete this step and confirm it passed before
running any tests, linters, or other commands that produce output.
"Skipped" is not "passed." If the scan cannot run (missing tools,
missing script, error), treat it as a failure and stop.**

Run the secret scan helper against your changed files:

```bash
"${SCAN_SECRETS:-scripts/scan-secrets}" <files-you-modified>
```

The script handles tool installation, staging, scanning, and unstaging
internally. It exits non-zero if secrets are detected or if it cannot
obtain a scanner.

If secret scanning detects secrets in your changes:

1. **Hard stop.** Do not run tests. Do not commit.
2. Remove the secrets from your code. Replace them with environment variable
   references or placeholders.
3. Re-run the secret scan. If it passes, continue to 8b.
4. If you cannot remove the secrets (e.g., the issue itself requires handling
   real credentials), stop. The post-script handles failure reporting.

**Only proceed to 8b after the secret scan passes.**

---

**8b. Tests and linters**

```bash
# Examples — use the actual commands for this repo
make test        # or: go test ./..., npm test, pytest
make lint        # or: pre-commit run --files <changed-files>
```

**If tests pass:** Proceed to step 9.

**If tests fail:**

1. Read the failure output. Identify the root cause.
2. Fix the issue in your implementation. Do not weaken or skip tests.
3. **Re-run the secret scan (8a) first**, then the test suite. This
   consumes one retry iteration. Do not skip the re-scan — your fix may
   have introduced secrets.
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

**9b. Review and scan what you are committing**

```bash
git diff --cached --stat
```

Read the output and confirm only your intended files are present. If anything
unexpected is staged, unstage it:

```bash
git reset HEAD <file-you-did-not-intend-to-stage>
```

Then run the secret scan against the actual staged content:

```bash
"${SCAN_SECRETS:-scripts/scan-secrets}" --staged
```

**This is not a repeat of step 8a.** Step 8a scans the files you *named*.
This scans the files you *actually staged*, which may differ. If the scan
fails, do not commit — fix the issue and re-stage.

**9c. Commit**

**Always create a new commit. Never use `git commit --amend`.** Even if a
previous agent run left a commit on the branch, create a new commit on top.
Amending rewrites someone else's commit and loses attribution.

The commit message must:

- **Use the repo's commit convention as discovered in context-gathering.** If
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
prohibitions — what you cannot do. This skill does not restate them. The
`scan-secrets` helper in the shared `scripts/` directory enforces the
scan-before-test ordering with guaranteed cleanup. Other constraints are
enforced by `disallowedTools` in the agent definition and by the
procedural steps above.

If a step in this skill appears to conflict with the agent definition, the
agent definition wins.
