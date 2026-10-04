# Final Run Manifest — compliant_forge_v3 (clean train-only baseline) nano-high

One-shot `--split test` run registration for the clean, train-only, label-free
v3 baseline. The results rows are filled verbatim from each domain's
`metrics.json` (pass@1/pass^5/UX) and `per_task_metrics/` (pass@5, state/task
requirement satisfaction) — no judge calls in the metrics pass.

## Claim labeling

- Label: **Clean train-only baseline (frozen-before-any-judge, no refinement).**
  The shipped artifact was authored from `train_fit` demonstrations only and was
  not modified after any eval; the single `--split test` touch is the reported
  number. No post-run tuning; no test-failure analysis fed back into any rule
  (that would forfeit the label).

## Registration fields

| Field | Value |
|---|---|
| Claim cycle ID | `compliant_forge_v3_nanohigh_TESTclean_official` |
| Agent class | `CompliantForgeV3Agent` (`agents/compliant_forge_v3_agent.py`) |
| Agent model | `gpt-5.4-nano` |
| Agent reasoning level | `high` |
| Agent provider | `openai` |
| `retrieve_learnings` top-k | `3` |
| Num runs | `5` |
| Num workers | `4` |
| Split | `test` (50 held-out tasks/domain) |
| Registry / artifact root | `logs/compliant_forge_v3` (via `COMPLIANT_FORGE_DIR`) |
| Artifact files | `logs/compliant_forge_v3/<domain>/ruleset_fit.json` + `playbook.json` + `super_prompt_v3.txt` |
| Eval provider (sim + judge) | Azure OpenAI, **locked** `gpt-5.4` (judge reasoning `high`) |
| Protocol id | `state_bench_v0.8.0_gpt54` |
| STATE-Bench commit SHA | `e2c8d7af51ef48fbbea51bb2ce1fb859af36b423` (v0.8.0) |
| Multitude2-0 commit SHA | `a3ee01c951455ad25b7fefa08370257b2c48b2e5` (branch `statebench/compliant-forge-nano-submission`) |
| Domains | `customer_support`, `travel`, `shopping_assistant` |

## Learning method (registration note)

> **Deterministic + offline-authored rule learning from `train_fit`
> demonstrations only.** No LLM/judge is used for learning; the base rule set is
> mined deterministically and the per-family playbooks are authored offline from
> the same `train_fit` conversations, then frozen. The only learning input is
> `datasets/train_task_trajectories/<domain>/` restricted to the `train_fit`
> ids. Task-definition `state_requirements`, judge verdicts, and any scored
> trajectory field are refused at load time by
> `brains/statebench/compliant_forge/allowed_inputs.py`
> (`ground_truth_used=false`, `labels_used=false`). See `contamination_proof_v3.md`
> for the machine-checked covered ∩ test = 0 result and `RULE_GENERATION.md` for
> the frozen-before-any-judge guarantee (and the reverted dev-refinement
> disclosure).

## Artifact fingerprint (train_fit-only rule sets)

| Domain | train_fit demos | families | base if/else rules | covered ids (unique) | covered ∩ test |
|---|---:|---:|---:|---:|---:|
| customer_support | 69 | 7 | 189 | 55 | **0** |
| travel | 70 | 7 | 131 | 53 | **0** |
| shopping_assistant | 70 | 8 | 88 | 46 | **0** |

(v2 mined all 100 train trajectories: cs 189 / travel 155 / shopping 107 base
rules. v3's smaller counts reflect the `train_fit`-only restriction that keeps
`train_dev` a genuine holdout.)

## Exact run command (per domain — frozen config, mirrors v2 official5)

```bash
export PYTHONUTF8=1 PYTHONIOENCODING=utf-8
export MULTITUDE_ROOT=/home/simon/Cursor/Multitude2-0
export COMPLIANT_FORGE_DIR=/home/simon/Cursor/Multitude2-0/logs/compliant_forge_v3
export STATE_BENCH_AGENT_MODEL=gpt-5.4-nano
export COMPLIANT_FORGE_DOMAIN=<DOMAIN>   # customer_support | travel | shopping_assistant

# from the pristine STATE-Bench checkout
uv run python -m state_bench.scripts.run_batch \
  --domain <DOMAIN> --split test \
  --agent-class CompliantForgeV3Agent --retrieve-learnings-top-k 3 \
  --agent-provider openai --agent-model-name gpt-5.4-nano \
  --agent-model-reasoning-level high \
  --num-runs 5 --num-workers 4 \
  --output-dir outputs/<DOMAIN>_compliant_forge_v3_nanohigh_TESTclean_official
```

## Exact metrics command (per domain)

```bash
uv run python -m state_bench.scripts.compute_metrics \
  --domain <DOMAIN> --split test --num-runs 5 \
  --results-dir outputs/<DOMAIN>_compliant_forge_v3_nanohigh_TESTclean_official \
  --output-dir  outputs/<DOMAIN>_compliant_forge_v3_nanohigh_TESTclean_official
```

## Results (file-accurate)

| Domain | pass@1 (mean ± std) | pass^5 | pass@5 | state_req_met | task_req_met | Mean UX | avg cost/task |
|---|---|---|---|---|---|---|---|
| customer_support | 0.57 ± 0.03 | 0.32 | 0.76 | 0.84 | 0.61 | 3.87 | $0.00 |
| travel | 0.66 ± 0.05 | 0.34 | 0.88 | 0.87 | 0.69 | 3.38 | $0.00 |
| shopping_assistant | 0.64 ± 0.02 | 0.48 | 0.80 | 0.92 | 0.66 | 3.79 | $0.00 |
| **Macro avg** | **0.62** | **0.38** | **0.81** | **0.88** | **0.65** | **3.68** | **$0.00** |

### Side-by-side vs v2 official5

| Domain | v2 pass@1 / pass^5 | v3 pass@1 / pass^5 |
|---|---|---|
| customer_support | 0.61 / 0.38 | 0.57 / 0.32 |
| travel | 0.65 / 0.32 | 0.66 / 0.34 |
| shopping_assistant | 0.66 / 0.48 | 0.64 / 0.48 |
| macro | 0.64 / 0.39 | 0.62 / 0.38 |

## Post-completion checklist

- [x] `outputs/<DOMAIN>_..._TESTclean_official/run{1..5}/<task_id>.json` present for all 50 test tasks × 5 runs (250 scored trajectories) per domain.
- [x] `metrics.json` present with protocol id `state_bench_v0.8.0_gpt54`.
- [x] No post-run tuning; frozen artifact unchanged after the run.
- [x] Results tables filled from the metrics files and propagated to `SUBMISSION.md`.
