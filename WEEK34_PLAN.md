# Week 34 Plan: External OSS PRs + Next Weekly Project

## External PR Targets (Saturday + Sunday)

### Target Repos & Issues (AI/ML/Data focus)

**High Priority (core ML libraries):**
1. **huggingface/transformers#46015** - Outdated examples about RAG (docs, good first issue)
2. **huggingface/transformers#44016** - Syntax error in Transformer section 3 notebook (bug, good first issue)
3. **huggingface/datasets#6755** - Small typo on documentation (good first issue)
3. **huggingface/datasets#5653** - Doc: save_to_disk num_proc affects num_shards (docs, good first issue)
4. **huggingface/peft#2835** - Update deprecated torch_dtype argument (good first issue)
5. **huggingface/accelerate#3639** - Update low-precision training docs for MS-AMP (docs, good first issue)

**Evaluation/Benchmarking (directly relevant):**
6. **EleutherAI/lm-evaluation-harness#3005** - zeno_visualize.py can't parse model_args (bug, good first issue)
7. **EleutherAI/lm-evaluation-harness#2552** - Hendrycks Math extraction rule seems too strict (validation, good first issue)
8. **huggingface/evaluate#326** - Evaluate usage in other frameworks (good first issue)

**RL/Environment (relevant to background):**
9. **Farama-Foundation/Gymnasium#1118** - Allow to specify dtype for Discrete (enhancement, good first issue)
10. **Farama-Foundation/Gymnasium#1085** - Allow frame stack for stacks of size 1 (enhancement, good first issue)

**Optimization/Tools:**
11. **optuna/optuna#6305** - Use f-string {var_name=} instead of .format (code-fix, good first issue)
12. **optuna/optuna#6136** - Use return a, b instead of return (a, b) (code-fix, good first issue)

---

## Saturday Plan (Aug 16)

### Morning (9:00-13:00) - PR #1: huggingface/transformers
- [ ] Fork huggingface/transformers
- [ ] Clone locally, set up dev environment
- [ ] Read CONTRIBUTING.md
- [ ] Pick issue #44016 (syntax error in notebook) - concrete, testable
- [ ] Fix the syntax error in the notebook
- [ ] Run relevant tests
- [ ] Open PR with proper description referencing issue

### Afternoon (14:00-18:00) - PR #2: huggingface/datasets
- [ ] Fork huggingface/datasets
- [ ] Pick issue #6755 (typo in docs) or #5653 (doc improvement)
- [ ] Fix documentation
- [ ] Run doc build/tests
- [ ] Open PR

### Evening (19:00-21:00) - Review & Polish
- [ ] Check both PRs for quality
- [ ] Respond to any immediate feedback
- [ ] Log in GHOST_ACTIVITY.md

---

## Sunday Plan (Aug 17)

### Morning (9:00-13:00) - PR #3: EleutherAI/lm-evaluation-harness
- [ ] Fork lm-evaluation-harness
- [ ] Pick issue #3005 (zeno_visualize.py bug) - directly relevant to our work
- [ ] Fix the model_args parsing bug
- [ ] Add test if possible
- [ ] Open PR

### Afternoon (14:00-18:00) - PR #4: huggingface/peft or accelerate
- [ ] Fork peft or accelerate
- [ ] Pick peft#2835 (torch_dtype) or accelerate#3639 (docs)
- [ ] Implement fix
- [ ] Open PR

### Evening (19:00-21:00) - Week 34 Weekly Project Planning
- [ ] Choose Week 34 project (see options below)
- [ ] Create week34-<project-name> repo
- [ ] Write PLAN.md
- [ ] Scaffold project structure
- [ ] Open week-plan issue

---

## Week 34 Project Options (Monday Start)

### Option A: LLM Benchmark Dataset Curator
- Tool to download, clean, and format benchmark datasets (MMLU, GSM8K, HumanEval, etc.)
- Output standardized JSONL for llm-eval-harness
- CLI + Python API
- Relevance: Directly complements Week 33 project

### Option B: Model Quantization Comparison Tool
- Compare GGUF quantization levels (q4_k_m vs q5_k_m vs q8_0) on benchmarks
- Automate llama.cpp quantization + evaluation
- Generate comparison reports
- Relevance: ML engineering, local LLM optimization

### Option C: RL Environment Wrapper for LLM Agents
- Gymnasium-compatible env for LLM agent benchmarks
- Tools: web search, code exec, file ops
- Integrates with llm-eval-harness for eval
- Relevance: RL + LLM agents

### Option D: Prompt Optimization Framework
- Automated prompt engineering: MIPRO, APE, or simple grid search
- Works with OpenRouter/local models
- Evaluates using llm-eval-harness
- Relevance: Prompt engineering, optimization

### Option E: Training Data Quality Scorer
- Score/filter training data for LLMs (dedup, perplexity, toxicity, quality)
- Integrates with datasets library
- Relevance: Data engineering for LLM training

---

## Decision: Option A (LLM Benchmark Dataset Curator)
**Rationale:**
- Directly extends Week 33's llm-eval-harness
- High visibility: every LLM eval needs good benchmarks
- Clear scope: download → clean → format → export
- Uses existing skills: datasets, HuggingFace Hub, CLI tools
- Can be done in 1 week with solid MVP

---

## Monday (Aug 18) - Week 34 Start

1. Create repo `week34-benchmark-curator`
2. Write PLAN.md with:
   - Scope: Download, clean, format benchmark datasets
   - Milestones Tue-Sun
   - DoD: CLI works, tests pass, CI green, docs
3. Scaffold: README, LICENSE, .gitignore, pyproject.toml, CI
4. Open week-plan issue
5. Push

---

## GHOST_ACTIVITY.md Tracking (Private Repo)

Create/update in private notes repo:
```markdown
# GHOST_ACTIVITY.md

## 2026-08-16 (Sat)
- PR: huggingface/transformers#44016 - Fix syntax error in notebook
- PR: huggingface/datasets#6755 - Fix typo in documentation

## 2026-08-17 (Sun)
- PR: EleutherAI/lm-evaluation-harness#3005 - Fix zeno_visualize.py model_args parsing
- PR: huggingface/peft#2835 - Update deprecated torch_dtype argument

## Week 34 Project: week34-benchmark-curator (LLM Benchmark Dataset Curator)
```