# Code Agent Evaluation — Scenarios & Payloads

Test data for the [fullsend code agent evaluation experiment](https://github.com/fullsend-ai/fullsend/blob/experiment/code-agent-evaluation-writeup/experiments/code-agent-evaluation/EXPERIMENT.md).

## Contents

| Directory | What | Count |
|-----------|------|-------|
| `scenarios/` | Ground-truth JSON definitions for each scenario (S01–S20, R01–R02) | 22 + manifest |
| `payloads/` | Injection attack payloads (p01–p10) used in security scenarios | 10 |
| `prompts/` | LLM judge system prompt | 1 |

## Scenarios

Each scenario JSON defines:
- Target repo and issue
- Expected files to change
- Applicable deterministic gates
- Difficulty and category
- Ground truth for the LLM judge

See `scenarios/manifest.json` for the full index and `scenarios/issue-map.txt`
for the mapping from scenario IDs to GitHub issue URLs.

## Evaluation Repos

These scenarios run against the following GitHub repos:

| Repo | Language | Scenarios |
|------|----------|-----------|
| [ascerra/eval-go-service](https://github.com/ascerra/eval-go-service) | Go | S01–S04, S11–S15, S17–S20 |
| [ascerra/eval-python-cli](https://github.com/ascerra/eval-python-cli) | Python | S05–S06, S16 |
| [ascerra/eval-ts-webapp](https://github.com/ascerra/eval-ts-webapp) | TypeScript | S07–S10 |
| [ascerra/eval-hostile-target](https://github.com/ascerra/eval-hostile-target) | Go (hostile) | S11 |
| [ascerra/build-definitions](https://github.com/ascerra/build-definitions) | Tekton YAML | R01 |
| [ascerra/integration-service-test](https://github.com/ascerra/integration-service-test) | Go/K8s | R02 |

## Related

- [EXPERIMENT.md](https://github.com/fullsend-ai/fullsend/blob/experiment/code-agent-evaluation-writeup/experiments/code-agent-evaluation/EXPERIMENT.md) — full experiment writeup
- [PR #189](https://github.com/fullsend-ai/fullsend/pull/189) — the code agent PR these scenarios validated
