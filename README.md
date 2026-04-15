# Code Agent Evaluation — Scenarios, Payloads & Variants

Test data for the [fullsend code agent evaluation experiment](https://github.com/fullsend-ai/fullsend/blob/experiment/code-agent-evaluation-writeup/experiments/code-agent-evaluation/EXPERIMENT.md).

## Contents

| Directory | What | Count |
|-----------|------|-------|
| `scenarios/` | Ground-truth JSON definitions for each scenario (S01–S20, R01–R02) | 22 + manifest |
| `payloads/` | Injection attack payloads (p01–p10) used in security scenarios | 10 |
| `prompts/` | LLM judge system prompt | 1 |
| `variants/` | Agent + skill definitions for all evaluated variants (V1–V7) | 7 |

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

## Variants

Each variant directory contains the agent definition (`agents/code.md`), skill
(`skills/code-implementation/SKILL.md`), and helper scripts used during that
variant's evaluation trials. See `variants/` for the full set.

| ID | Name | Description |
|----|------|-------------|
| V1 | fullsend-single-skill | PR #189 original — agent + single skill + scan-secrets |
| V2 | fullsend-multi-skill | PR #189 with skill split into 4 pieces |
| V3 | vanilla-claude | No guardrails baseline — just a prompt |
| V4 | claudemd-only | CLAUDE.md instructions only — stopped early |
| V5 | apex | Enhanced V1 with reasoning protocol, self-review, minimal diff |
| V6 | apex-github | V5 hardcoded for GitHub (rigidity hurts) |
| V7 | ultimate | V5 + "understand before you act" + reproduction step |

> **V8 (hybrid)** is the variant proposed in PR #189's latest commit. Its files
> live in the [fullsend repo](https://github.com/fullsend-ai/fullsend/tree/experiment/code-agent-evaluation-writeup/experiments/code-agent-evaluation/variants/V8-hybrid)
> rather than here since it's the active deliverable.

## Related

- [EXPERIMENT.md](https://github.com/fullsend-ai/fullsend/blob/experiment/code-agent-evaluation-writeup/experiments/code-agent-evaluation/EXPERIMENT.md) — full experiment writeup
- [PR #189](https://github.com/fullsend-ai/fullsend/pull/189) — the code agent PR these scenarios validated
