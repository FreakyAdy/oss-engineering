# PROJECT_STATE.md — OSS Engineering Track

> Source of truth for the weekly cycle. Updated by Monday planning + daily execution crons.
> Cadence: ONE project per week, FULLY COMPLETED Mon-Sun. Weekly rotation.

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
Python ≥3.10, pytest (project uses pytest via uv/hatchling), ruff. Local: uv 0.11.16, Windows.

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

### WEEK 1 TASK PLAN (Mon-Sat; deliberately uneven)

**Mon (Day 1) — Setup + exploration. Expected commits: 0**
- Clone `UKGovernmentBEIS/inspect_ai` into `C:\Users\FreakyAdy\oss-engineering\inspect_ai`
- Read README + CONTRIBUTING.md (tiered policy — we're "new contributors": must work from an accepted issue; ours is)
- `uv sync` / install dev deps per CONTRIBUTING (Python ≥3.10; uv will fetch a matching interpreter)
- Run the scorer test subset: `uv run pytest tests/scorer -q` (baseline green)
- Read `src/inspect_ai/scorer/_model.py` fully: find where grader model/role binding happens and where the fallback occurs; note exact functions/lines in PROJECT_STATE progress
- Verify #4695 still open + unclaimed (jina/GitHub page)
- Report: setup status, key findings, file map

**Tue (Day 2) — Core implementation. Expected commits: 2**
- If `gh auth status` authenticated: comment on #4695 claiming it ("I'll implement this — Ady/Friday"). If not: flag auth blocker (do NOT fake the claim)
- Write failing tests first (`tests/scorer/` — warning fires when unbound; silent when bound)
- Run them: confirm FAIL
- Implement the warning in `src/inspect_ai/scorer/_model.py` (follow existing warning/logging patterns)
- Run tests: PASS
- Commit 1: `test: cover warning when model-graded scorer has no grader bound`; Commit 2: `feat: warn when model-graded scorer runs without grader model/role`

**Wed (Day 3) — Edge cases + hardening. Expected commits: 1**
- Cover edge cases: role bound but no model (and vice versa), custom scorers, nested/multi-scorer configs
- Run full scorer suite + `uv run ruff check .` + `uv run ruff format --check .`
- Fix failures at root cause
- Commit: `fix: handle partial grader binding in model-graded scorers` (or fold into feat if no new change — then 0 commits today, fine)

**Thu (Day 4) — Docs + PR. Expected commits: 1-2**
- CHANGELOG.md entry (follow project format) + any scorer docs touch
- Branch: `fix/scorer-no-grader-warning`; final full test run + lint
- Commit: `docs: document grader binding warning for model-graded scorers`
- If authed: `git push -u origin fix/scorer-no-grader-warning`, open PR referencing `Fixes #4695` with summary + test evidence. If not authed: prepare PR body in PROJECT_STATE + flag blocker

**Fri (Day 5) — Maintainer feedback + stretch. Expected commits: 1-2**
- Check PR for maintainer comments; respond + revise if needed (commit fixes as `fix: address review feedback`)
- If no feedback yet / PR clean: start stretch issue [#4770 — ChatMessage.text setter reorders content blocks](https://github.com/UKGovernmentBEIS/inspect_ai/issues/4770) (accepted bug): reproduce with failing test, implement fix, commit (`fix: preserve content block order in ChatMessage.text setter`), prepare second PR
- Only if #4695 is fully submitted and clean

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
- **Ady: run `gh auth login`** (GitHub.com → HTTPS → browser device flow). Without it: no issue claim, no push, no PR. Everything else works locally.
- Rate-limited unauthenticated GitHub API on shared IP — use raw.githubusercontent.com / jina / browser for reads.

### NEXT WEEK (Week 2)
After #4695 ships: chain the accepted-issue queue in inspect_ai (#4770, #4781, #4756) OR new tool from IDEAS.md. Decision at Monday 09:00 cron.
