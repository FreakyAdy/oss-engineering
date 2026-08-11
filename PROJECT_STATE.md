# PROJECT_STATE.md — OSS Engineering Track

> Source of truth for the weekly cycle. Updated by Monday planning + daily execution crons.
> Cadence: EVERY week = ONE PR (active OSS repo) + ONE tool OR full-scale project (alternate weeks). NO zero-commit days Mon-Sat (Sun report-only; Mon = planning + base/repo creation).

### OPERATING RULES (user-locked 2026-08-12)
1. **Two weekly deliverables, non-negotiable:** (1) ONE complete PR into an active OSS repo + (2) ONE standalone tool (own public repo, complete within the week) — OR a **full-scale project** on alternate weeks (ambitious, multi-week allowed; occupies the tool slot until complete).
2. **No zero-commit days Mon-Sat.** Sunday = report only. Monday = planning + base/repo creation commits.
3. **Alternation:** odd weeks = tool; even weeks = full-scale project (W2, W4, ...). Monday cron schedules it.
4. Quality bar stands: real work only, tests, docs, one coherent change per commit. No fabricated activity.

---

## WEEK 1 (2026-08-10 → 2026-08-16)

### PROJECT: inspect_ai — runtime warning for unbound model-graded scorers
- **REPOSITORY:** https://github.com/UKGovernmentBEIS/inspect_ai (UK AI Security Institute, MIT, 2.5k★, active)
- **LOCAL PATH:** `C:\Users\FreakyAdy\oss-engineering\inspect_ai` (clone during Day 1)
- **ISSUE:** [#4695 — model-graded scorers should warn at runtime when no grader model/role is bound](https://github.com/UKGovernmentBEIS/inspect_ai/issues/4695) (labeled `accepted` = maintainer-approved for external PR, plus `good first issue`; author MattFisher)

### PROBLEM
Model-graded scorers (`model_graded_qa`, `self_critique`, `scale_critique`, `act_eval`, etc.) can run with **no explicit grader model or role bound**. When that happens, the *same model and role being evaluated* grades itself — silently. Users get confident scores produced by the model grading its own output, with no signal that this happened.

### CURRENT BEHAVIOR
No warning. A scorer configured without a grader model/role silently falls back to the target model, and the eval proceeds as if the scores were independent.

### WHY IT MATTERS
Silent self-grading corrupts evaluation results — the exact thing inspect_ai exists to prevent. Users unknowingly publish self-graded numbers. A runtime warning is the minimal, safe intervention: it preserves behavior but makes the risk visible.

### PROPOSED SOLUTION
Emit a runtime warning (`warn()` from inspect's logging/`util` module — follow existing patterns in the codebase) when a model-graded scorer is invoked without an explicit grader model or role bound, while keeping the existing fallback behavior (no functional change). Cover all model-graded scorer entry points in `src/inspect_ai/scorer/_model.py`.

### MVP SCOPE
- Detect "no grader model/role bound" at scorer invocation time for all model-graded scorers in `src/inspect_ai/scorer/_model.py`
- Emit one clear warning (include scorer name + how to fix: pass `model=`/`grader_role=` or configure a default)
- Unit tests asserting the warning fires (and that explicit binding silences it)
- Update relevant docs/CHANGELOG entry

### OUT OF SCOPE
- Changing fallback behavior (still self-grades — just loudly)
- New CLI flags / config surface
- Other scorer families (only model-graded ones)

### TECH STACK
Python ≥3.10, pytest (project uses pytest via uv/hatchling), ruff. Local: venv Python 3.13.13, ruff in `.venv/Scripts/`, Windows.

### TEST STRATEGY
- Failing test first: instantiate a model-graded scorer without grader binding → assert warning raised (pytest `warns`/`caplog`)
- Assert explicit `model=`/role binding produces no warning
- Run full scorer test suite + repo lint (`ruff`) before commit
- Offline-only tests: no real model API calls in our tests (mock the model)

### SUCCESS CRITERIA (completion =)
1. Code + tests + docs changes committed on branch `fix/scorer-no-grader-warning`
2. Full scorer test suite green locally
3. PR opened referencing `Fixes #4695`, with issue claimed via comment first (per CONTRIBUTING: new contributors must start from an accepted issue)
4. If `gh` auth unavailable: local branch + tests green + PR body prepared, auth flagged as blocker

---

### WEEK 1 TASK PLAN (Mon-Sat; deliberately uneven) — WITH ACTUALS

**Mon (Day 1) — Setup + exploration. Expected commits: 0. DONE**
- Cloned `UKGovernmentBEIS/inspect_ai`, read README + CONTRIBUTING + AGENTS.md (tiered policy — we're "new contributors": must work from an accepted issue; ours is)
- `uv sync` / venv ready (Python 3.13.13 in `.venv`)
- Scorer test subset baseline green
- Read `src/inspect_ai/scorer/_model.py`: all entry points (`model_graded_fact` → `model_graded_qa` → `_model_graded_qa_single`) route through one shared `score()`; fallback happens at model resolution (`get_model(role=...)` → default model)
- #4695 confirmed open + accepted

**Tue (Day 2) — Core implementation. Expected commits: 2. DONE (ahead of schedule)**
- `gh` authenticated (FreakyAdy); issue claimed via PR
- Implemented warning in `src/inspect_ai/scorer/_model.py` (`warn_once` from `inspect_ai._util.logger`; fires when `model is None and (model_role is None or model_role not in model_roles())`)
- Tests: warning fires for unbound / explicit `model_role=None` / role-requested-but-not-bound; silent for explicit `model=` and bound role
- **PR #4830 opened 2026-08-11 20:40** — `feat: warn when model-graded scorer runs without a grader model or role` (https://github.com/UKGovernmentBEIS/inspect_ai/pull/4830), `Fixes #4695`, gate CI green
- Bonus: **PR #4831 opened 2026-08-11 20:40** — `fix: map POSIX 'no such file or directory' to FileNotFoundError in docker sandbox` (https://github.com/UKGovernmentBEIS/inspect_ai/pull/4831), 2 commits, gate CI green (from the accepted-issue queue)

**Wed (Day 3) — Edge cases + hardening. Expected commits: 1. DONE**
- Partial-binding edge cases verified covered by PR #4830 tests (role bound no model / model no role / role requested but unbound)
- Added missing nested/multi-scorer coverage: multi-scorer model list stays silent; nested unbound scorers dedupe to exactly one warning
- Commit `test: cover nested and multi-scorer configs for grader binding warning` (5538904d9), pushed to fork
- Full suite: 459 passed, 170 skipped (vllm/trio env skips); ruff check + format clean
- Added `### Agent review` disclosure to both PR bodies per AGENTS.md

**Thu (Day 4) — Docs + PR. Expected commits: 1-2**
- CHANGELOG.md entry (follow project format) + any scorer docs touch — pending; PR opened early on Tue, so docs commit can land here
- Final full test run + lint before any further push

**Fri (Day 5) — Maintainer feedback + stretch. Expected commits: 1-2**
- Check PRs for maintainer comments; respond + revise if needed (commit fixes as `fix: address review feedback`)
- If no feedback yet / PR clean: stretch issue [#4770 — ChatMessage.text setter reorders content blocks](https://github.com/UKGovernmentBEIS/inspect_ai/issues/4770) (accepted bug): reproduce with failing test, implement fix, commit, prepare second PR
- Only if #4695 PR is fully submitted and clean

**Sat (Day 6) — Finalize. Expected commits: 0-1**
- Verify all commits pushed, PR state accurate, no dangling work
- Update PROJECT_STATE.md: completed work, tests added, PR links, blockers
- Any final review-feedback commits

**Sun — no code.** Weekly report cron (18:00) produces Phase-12 report.

---

### IMPORTANT DECISIONS
- Selected inspect_ai over python-sdk/llm because: formal `accepted`-label external-PR pipeline, fills Ady's public agent-eval evidence gap, Python + eval = core skill, docs culture, reasonable review bar (2.5k★)
- Completion = PR submitted + tests green (merge depends on maintainers — out of our control)
- Stretch #4770 only if #4695 is fully shipped first — never two half-done PRs

### BLOCKERS / ACTION ITEMS
- ~~**Ady: run `gh auth login`**~~ — RESOLVED 2026-08-11. Authed as FreakyAdy (ssh protocol; scopes: admin:public_key, gist, read:org, repo).
- Rate-limited unauthenticated GitHub API on shared IP — use raw.githubusercontent.com / jina / browser for reads.
- Both PRs await maintainer review (reviewDecision: REVIEW_REQUIRED). No comments as of 2026-08-12 02:30.
- PR #4830 CHANGELOG entry still pending (Day 4 task).

### NEXT WEEK (Week 2)
After #4695 ships: chain the accepted-issue queue in inspect_ai (#4770, #4781, #4756) OR new tool from IDEAS.md. Decision at Monday 09:00 cron.

---

## SIDE-PROJECT TRACK (parallel to weekly PR, from 2026-08-12)

> No-zero-days mandate (Ady, 2026-08-12): every Mon-Sat has a real commit. When the weekly PR is done/blocked,
> the side-project track supplies the day's work. Each side project = its own public repo, built till it's complete.

### SIDE PROJECT #1: jsonl-tail
- **REPO:** https://github.com/FreakyAdy/jsonl-tail (public) — created 2026-08-12
- **WHY:** tail JSONL files (eval logs, datasets) with pretty-printed JSON, filters, live follow. Name free on PyPI + npm (verified 2026-08-12). Directly useful for LLM eval work; stdlib-only; dogfoods the eval-log workflow.
- **STACK:** Python ≥3.10, stdlib only, uv + hatchling, pytest, ruff.

| Day | Task | Status |
|-----|------|--------|
| Wed 08-12 | Repo + scaffold + core: pretty/compact print, -n/-a, stdin, --filter/--regex/--key, invalid-line passthrough | DONE — 13 tests, ruff clean, pushed |
| Thu 08-13 | Follow-mode tests (append + truncation), edge cases (empty file, CRLF, unicode, oversized records), LICENSE (MIT) | pending |
| Fri 08-14 | Polish: CHANGELOG, README badges, final full test+lint, **COMPLETE before Sat** | pending |
| Sat 08-15 | Buffer only — fixes if needed; else WEEKLY_LOG.md entry | pending |

### COMPLETED SIDE PROJECTS (log)
- (none yet)

### BACKLOG (next picks)
IDEAS.md: gh-pulse, issue-feeder, agent-eval CLI, promptlint, dataset integrity checker, GitHub Actions issue triager, LLM cost estimator.
