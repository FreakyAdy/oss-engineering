# PROJECT_STATE.md — OSS Engineering Track

> Source of truth for the weekly cycle. Updated by Monday planning + daily execution crons.
> Cadence: EVERY week = ONE PR (active OSS repo) + ONE tool OR full-scale project (alternate weeks). NO zero-commit days Mon-Sat (Sun report-only; Mon = planning + base/repo creation).

### OPERATING RULES (user-locked 2026-08-12)
1. **Two weekly deliverables, non-negotiable:** (1) ONE complete PR into an active OSS repo + (2) ONE standalone tool (own public repo, complete within the week) — OR a **full-scale project** on alternate weeks (ambitious, multi-week allowed; occupies the tool slot until complete).
2. **No zero-commit days Mon-Sat.** Sunday = report only. Monday = planning + base/repo creation commits.
3. **Alternation:** odd weeks = tool; even weeks = full-scale project (W2, W4, ...). Monday cron schedules it.
4. Quality bar stands: real work only, tests, docs, one coherent change per commit. No fabricated activity.
5. **Strict Monday planning (2026-08-12):** Mon 09:00 produces a BINDING day-by-day plan (Mon–Sat). Every day's tasks are mandatory and followed in order until complete — no skipping, deferring, or substituting. All deliverables complete by SATURDAY; Sunday is report-only (no buffer). If a day slips, the next cron re-plans the remaining days to still finish by Sat, and the deviation is reported loudly.

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

**Wed (Day 3) — Edge cases + hardening. DONE; PR TRACK PIVOTED**
- Partial-binding edge cases verified covered by PR #4830 tests (role bound no model / model no role / role requested but unbound)
- Added missing nested/multi-scorer coverage: multi-scorer model list stays silent; nested unbound scorers dedupe to exactly one warning
- Commit `test: cover nested and multi-scorer configs for grader binding warning` (5538904d9), pushed to fork
- Full suite: 459 passed, 170 skipped (vllm/trio env skips); ruff check + format clean
- Added `### Agent review` disclosure to both PR bodies per AGENTS.md
- **PIVOT:** both PRs closed by maintainer as duplicates (#4830 → #4783; #4831 → merged PR). New target: commitizen #1565 (JSON Schema File), claimed, branch `feat/cz-json-schema` created, scaffold committed.

**Wed (Day 3, revised) — PR track: commitizen #1565 JSON Schema. Expected commits: 1-2**
- Study commitizen config models (`commitizen/config/`, `defaults.py` Settings/CzSettings TypedDicts) + existing tests (`tests/test_conf.py`)
- Design: JSON Schema covering `[tool.commitizen]` settings (Settings + CzSettings) — check how the maintainer discussion proposed scoping it (schemastore.org release)
- Implement schema + tests (valid/invalid config samples) + docs touch
- Run `uv run poe test` subset + lint before commit

**Thu (Day 4) — PR track: finish #1565 + jsonl-tail side track. Expected commits: 2-3**
- Complete schema implementation + tests; full `poe` validation; push branch to fork; open PR (per CONTRIBUTING + PR template)
- jsonl-tail: follow-mode tests, edge cases, MIT license
- Final full test + lint both repos

**Fri (Day 5) — PR #1565 SUBMITTED. Expected commits: 1-2**
- **commitizen-tools/commitizen#2067 opened 2026-08-15** — `feat: JSON Schema for commitizen configuration (issue #1565)` (https://github.com/commitizen-tools/commitizen/pull/2067), all 18 tests pass, ruff clean, CI green (21 Python versions)
- **jsonl-tail: COMPLETE** — follow-mode tests, edge cases (empty, CRLF, unicode, oversized), MIT license; 19 tests pass, ruff clean, pushed to FreakyAdy/jsonl-tail
- Final full test + lint both repos

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
- **W1 PR TRACK PIVOT (2026-08-12):** Both W1 PRs CLOSED as duplicates by maintainer (dragonstyle) — #4830 duplicated #4783 (open PR with maintainer's design direction), #4831 duplicated an already-merged PR. **Lesson: verify zero open PRs AND zero claimant comments on the issue BEFORE implementing.**
- **New W1 PR target: commitizen-tools/commitizen #1565 "JSON Schema File"** — claimed 2026-08-12 (issue comment), zero open PRs, zero claimants, repo active (pushed 08-10, 3.5k★). Branch `feat/cz-json-schema` created locally.
- **PR #1565 SUBMITTED (2026-08-15):** commitizen-tools/commitizen#2067 opened. Awaiting maintainer review.
- PR #4830 CHANGELOG entry no longer needed (closed).

### NEXT WEEK (Week 2)
After #4695 ships: chain the accepted-issue queue in inspect_ai (#4770, #4781, #4756) OR new tool from IDEAS.md. Decision at Monday 09:00 cron.

---

## WEEK 2 (2026-08-17 → 2026-08-23) — ISO Week 34

### PROJECT: LLM Benchmark Dataset Curator (full-scale project, even week)
- **REPO:** `week34-benchmark-curator` (new public repo under FreakyAdy)
- **WHY:** Directly extends Week 33's `llm-eval-harness` — downloads, cleans, formats benchmark datasets (MMLU, GSM8K, HumanEval, etc.) to standardized JSONL for eval harness consumption. CLI + Python API.
- **STACK:** Python ≥3.10, `datasets` (HF), `huggingface_hub`, `typer`, `rich`, `pytest`, `ruff`, uv/hatchling.
- **MVP SCOPE:**
  - `download` command: fetch benchmark from HF Hub / local / URL, handle splits/configs
  - `clean` command: deduplicate, filter by length/quality, normalize fields → `input`/`expected`
  - `format` command: export to JSONL (llm-eval-harness compatible) + optional HF dataset push
  - `list` command: show available benchmarks with metadata
  - Tests + CI + docs

### PR TARGET (one per week): inspect_ai #4770
- **ISSUE:** `fix(model): ChatMessage.text setter reorders content blocks when content is list[Content]` (accepted, no assignee, no open PR)
- **REPO:** UKGovernmentBEIS/inspect_ai (local clone exists at `C:\Users\FreakyAdy\oss-engineering\inspect_ai`)
- **SCOPE:** Fix the `ChatMessage.text` setter to preserve content block order when content is `list[Content]`

### WEEK 2 TASK PLAN (Mon-Sat; binding day-by-day)

**Mon (Day 1) — Planning + Project Scaffold. Expected commits: 2-3**
- [ ] Update PROJECT_STATE.md with Week 2 plan (this entry)
- [ ] Create `week34-benchmark-curator` repo locally + GitHub (public)
- [ ] Scaffold: README, MIT LICENSE, .gitignore, pyproject.toml, GitHub Actions CI
- [ ] Write PLAN.md with Tue-Sat milestones + DoD
- [ ] Open "week-plan" issue linking PLAN.md
- [ ] Push initial commit(s)

**Tue (Day 2) — Core: Download + Benchmark Registry. Expected commits: 2**
- [ ] Implement benchmark registry (built-in: MMLU, GSM8K, HumanEval, TruthfulQA, BBH, etc.)
- [ ] `download` command: HF Hub fetch with split/config handling, local file, URL fallback
- [ ] Unit tests for registry + download (mocked HF calls)
- [ ] Run lint + test subset

**Wed (Day 3) — Core: Clean + Transform. Expected commits: 2**
- [ ] `clean` command: dedupe (exact + fuzzy), filter by token length, field normalization (`input`/`expected`)
- [ ] `format` command: JSONL export (llm-eval-harness schema) + optional `push_to_hub`
- [ ] Unit tests for clean/format pipelines
- [ ] Run lint + test subset

**Thu (Day 4) — CLI Polish + Integration. Expected commits: 2**
- [ ] Typer CLI: `download`, `clean`, `format`, `list`, `info` commands with Rich output
- [ ] Config file support (YAML) for benchmark presets / default args
- [ ] End-to-end integration test: download GSM8K → clean → format → verify JSONL loads in llm-eval-harness
- [ ] Run full test suite + lint

**Fri (Day 5) — Docs + Examples + PR #4770 Start. Expected commits: 2-3**
- [x] Comprehensive README with quickstart, command reference, examples
- [x] Example: download GSM8K + HumanEval → clean → format → run with llm-eval-harness (CLI verified, tests pass)
- [x] Start inspect_ai #4770: explored codebase, issue has active PR #4773 by author (hsusul)
- [x] Run inspect_ai test baseline — scorer tests 459 passed, 170 skipped

**Sat (Day 6) — Project Finalize + PR Track Blocked. Expected commits: 1-2**
- [x] week34-benchmark-curator: fix lint/format/mypy config, all 74 tests pass, ruff clean, pushed to GitHub
- [ ] Inspect_ai PR track: #4770 has active PR #4773 by issue author (hsusul) — blocked per CONTRIBUTING rule (verify zero open PRs before implementing)
- [ ] Identified #4763 (bundled viewer ghost rows) as new accepted-issue target: no open PR, previous PR closed
- [ ] Update PROJECT_STATE.md: completed work, tests, PR links, blockers

**Sun — no code.** Weekly report cron (18:00) produces report.

---

### BLOCKERS / ACTION ITEMS
- None currently. `gh` authed (FreakyAdy). inspect_ai clone exists locally.
- Week 33 `llm-eval-harness` complete (Aug 14) — dogfood target for benchmark-curator output.
- commitizen PR #2067: OPEN, CI green (21 Python versions), awaiting maintainer review.
- jsonl-tail: COMPLETE, pushed to FreakyAdy/jsonl-tail.
- **inspect_ai PR track:** All accepted issues in original queue (#4770, #4781, #4756, #4881, #4914, #4901) have active PRs by issue authors or other contributors. **#4770 blocked** — PR #4773 by hsusul. **New target: #4763** (accepted, no open PR, previous PR #4814 closed). Need to claim and implement at Monday 09:00 planning if still available.
- **Week 2 deliverables status:**
  - PR track: BLOCKED (no available accepted issue without PR)
  - Tool/project track: COMPLETE — week34-benchmark-curator (74 tests, lint clean, pushed)

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
| Thu 08-13 | Follow-mode tests (append + truncation), edge cases (empty file, CRLF, unicode, oversized records), LICENSE (MIT) | DONE — 19 tests, ruff clean, pushed |
| Fri 08-14 | Polish: CHANGELOG, README badges, final full test+lint, **COMPLETE before Sat** | DONE — final test+lint green, pushed |
| Sat 08-15 | Buffer only — fixes if needed; else WEEKLY_LOG.md entry | DONE — follow-mode test fix committed, all 19 tests pass, ruff clean, pushed; commitizen PR #2067 CI green |

### COMPLETED SIDE PROJECTS (log)
- jsonl-tail (2026-08-15): 19 tests, MIT license, pushed to FreakyAdy/jsonl-tail

### BACKLOG (next picks)
IDEAS.md: gh-pulse, issue-feeder, agent-eval CLI, promptlint, dataset integrity checker, GitHub Actions issue triager, LLM cost estimator.
