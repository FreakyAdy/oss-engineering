# OSS Engineering Track — Research Log

> Phase 1 output (2026-08-10). All data verified live against GitHub API / pages that day.
> Cadence: one project per week, fully completed Mon-Sun.

## Final shortlist (scored 1-10; WF = one-week completability)

| # | Repo | RVU | ED | OSCP | MA | PV | LV | WF | TOTAL |
|---|------|-----|----|------|----|----|----|----|-------|
| 1 | **UKGovernmentBEIS/inspect_ai** (Python, MIT, 2.5k★) | 9 | 9 | 9 | 9 | 9 | 9 | 8 | **62** ✅ |
| 2 | modelcontextprotocol/python-sdk (Python, MIT, 24k★) | 9 | 7 | 10 | 9 | 9 | 8 | 7 | 59 |
| 3 | EleutherAI/lm-evaluation-harness (Python, MIT, 13.6k★) | 10 | 8 | 8 | 8 | 9 | 8 | 6 | 57 |
| 4 | simonw/llm (Python, Apache-2.0, 12.3k★) | 8 | 6 | 9 | 9 | 8 | 7 | 8 | 55 |
| 5 | promptfoo (TS, MIT, 24k★) | 9 | 7 | 6 | 7 | 9 | 7 | 6 | 51 |
| 5 | comet-ml/opik (Py+TS, Apache-2.0, 21k★) | 8 | 7 | 7 | 8 | 8 | 7 | 6 | 51 |
| 5 | langchain-ai/langgraph (Python, MIT, 39k★) | 9 | 9 | 5 | 8 | 8 | 7 | 5 | 51 |
| 8 | copier-org/copier (Python, MIT, 3.5k★) | 7 | 7 | 8 | 8 | 6 | 7 | 7 | 50 |
| 9 | commitizen-tools/commitizen (Python, MIT, 3.5k★) | 7 | 6 | 8 | 8 | 6 | 6 | 8 | 49 |
| 10 | ag2ai/ag2 (Python, Apache-2.0, 4.8k★) | 6 | 6 | 7 | 8 | 6 | 7 | 7 | 47 |

RVU=Real User Value · ED=Engineering Depth · OSCP=OSS Contribution Potential · MA=Maintainer Activity · PV=Portfolio Value · LV=Learning Value · WF=one-Week Fit

## Corroboration (delegation batch deleg_07f917e2, same day)
All three parallel research tracks returned; their top picks map 1:1 to this shortlist: eval track → inspect_ai (independently confirmed #4695 as clean MVP; noted it also carries the `good first issue` label), agent-infra track → python-sdk (#2), dev-tooling track → simonw/llm (#4). No changes to selection.

## Why inspect_ai won
- **Formal external-PR pipeline**: `accepted` label = "maintainer-approved for external PR"; CONTRIBUTING.md documents a tiered policy — new contributors MUST start from an accepted issue; unrequested PRs are auto-closed. 20+ open accepted issues, several clean junior-sized ones (#4695, #4770, #4781, #4756). This is the highest-signal on-ramp found in all three research tracks.
- **Fills the identified portfolio gap**: Ady's internship audit flagged "no public LLM-eval/observability artifacts". inspect_ai = agent/LLM evaluation — his differentiator.
- **Engineering + docs culture**: fully typed Python, pytest, daily merges, issues triaged within hours, 200+ prebuilt evals.
- **Skill fit**: Python strong; LLM evaluation systems is a stated skill; offline testable (mocks, no API keys needed).

## Runner-ups (revisit when applicable)
- **python-sdk**: 7 labeled issues incl. `ready for work` tags; MCP is Ady's domain. Pick for a future week (e.g. #348 isError flag for tool results).
- **lm-evaluation-harness**: "add a benchmark task" = classic one-week complete; 931-issue tracker noise is the cost.
- **simonw/llm**: #409 "fix skipped tests on Windows" — Ady is on Windows; Simon's reviews are fast and kind.
- **IDEAS.md** backlog (agent-eval CLI, promptlint, cost estimator, etc.) for new-tool weeks.

## Rejection log (Phase 10 discipline)
| Repo | Rejected because |
|------|-----------------|
| opencode-ai/opencode | **Archived** (Sept 2025) |
| Aider-AI/aider | Stale — last push 2026-05-22 |
| Textualize/rich | Stale — last push 2026-06-23 |
| cruft/cruft | Dead — last push 2024-12-25 |
| Ragas | Stale — last push 2026-02-24 |
| openai/evals | Stale — last push 2026-04-14 |
| AgentOps | Stale — last push 2026-06-25 |
| THUDM/AgentBench | Stale — last push 2026-02-08 |
| Braintrust core | NOT open source (only proxy/SDK/cookbook are) |
| deepeval | Active but 6 help-wanted issues are 1-2.5 yrs old, weak triage |
| crewAI / elizaOS / llama_index | 0 good-first-issue/help-wanted labels; mega-repos |
| mastra-ai | Open-core license (Apache-2.0 core + ee/ commercial) — must read LICENSE before contributing; thin junior issue pool |
| fastapi/typer | Very active but 0 labeled issues, 1 open issue — PR-only entry |
| langfuse | 478 open PRs = review bottleneck; custom dual license |
| promptfoo (deferred, not rejected) | 392 open PRs = review backlog flag; strong fallback for TS weeks |
