# OSS Engineering — Idea Backlog

> One project per week (Mon selection → Sun completion). Pick unbuilt candidates here first;
> validate the problem at selection time. Research-agent findings get merged in when they land.

## New-project candidates (unvalidated until selection)

| Idea | Problem it solves | Stack | Notes |
|------|-------------------|-------|-------|
| agent-eval CLI | Devs need a minimal harness to run an LLM agent against a task suite and get a pass/fail JSON report without wiring 5 tools | Python CLI, pytest | Fills Ady's "no public eval artifacts" gap; natural follow-on: CI integration |
| MCP server for a developer tool | Agents need standardized access to dev tools (e.g. gh, local test runners, eval results) | Python/TS, MCP SDK | Active ecosystem, real users, small surface |
| promptlint | Static checks on prompts in code: unresolved vars, missing output format, token-budget warnings | TS or Python, regex/ast | Real DX pain, pure logic = easy tests |
| LLM cost estimator CLI | Estimate the cost of a prompt file / dataset / test-suite run across providers before running | Python CLI, tabular | Pure computation, great test surface, tiny |
| SQLite experiment tracker | Tiny local-first run/experiment logger for LLM evals (no server, no cloud) | Python, sqlite3 | Scoped small vs MLflow/Langfuse; CLI + JSON export |
| Dataset integrity checker | Verify eval/benchmark datasets: schema, hashes, duplicates, label drift | Python CLI | Reproducibility angle, testable |
| GitHub Actions issue triager | Auto-label/route new issues by content for small repos | YAML + TS/Python | Real audience: single-maintainer repos |

## Existing-repo contribution candidates (from Phase 1 research)

- Research agents' findings (agent infra / LLM eval / dev tooling) land in RESEARCH.md — merge here at selection time.
