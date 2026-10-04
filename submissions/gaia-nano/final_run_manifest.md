# Final Run Manifest — compliant_forge (forge v2) nano-high

Run registration per Clean Protocol Checklist **D** (and appendix **H2**). The
metrics rows below are the file-accurate official5 results, read directly from
each domain's `metrics.json` and aggregated from the already-scored
`per_task_metrics/` files (no judge calls in this pass).

## Claim labeling

- Label: **Label-free, test-adapted (design-level)** (frozen, label-free
  artifact; pristine locked STATE-Bench surface; one-shot 50×5 holdout). The
  frozen artifact was not modified after its scored run (checklist E) and no
  per-task holdout failure drove a post-freeze change. However, the artifact's
  *design* (prompt engineering / conduct rails) was informed by a one-time,
  human-in-the-loop inspection of an earlier iteration's test-set behavior. No
  labels or test-set fields were ever fed into the agent or the learning harness.
  See the "Test-set adaptation disclosure" in `SUBMISSION.md`.

## Registration fields (D / H2)

| Field | Value |
|---|---|
| Claim cycle ID | `compliant_forge_v2_nanohigh_official5` |
| Agent class | `CompliantForgeAgent` (`agents/compliant_forge_agent.py`) |
| Agent model | `gpt-5.4-nano` |
| Agent reasoning level | `high` |
| Agent provider | `openai` |
| `retrieve_learnings` top-k | `3` |
| Num runs | `5` |
| Num workers | `4` |
| Split | `test` (50 held-out tasks/domain) |
| Registry / artifact root | `logs/compliant_forge_v2` (via `COMPLIANT_FORGE_DIR`) |
| Artifact files | `logs/compliant_forge_v2/<domain>/ruleset.json` + `super_prompt.txt` |
| Eval provider (sim + judge) | Azure OpenAI, **locked** `gpt-5.4` (judge reasoning `high`) |
| Protocol id | `state_bench_v0.8.0_gpt54` |
| STATE-Bench commit SHA | `e2c8d7af51ef48fbbea51bb2ce1fb859af36b423` (v0.8.0) |
| Multitude2-0 commit SHA | `97cd3f141c4eb82d26d0a80d0078937b547692ec` (branch `STATEBench`) |
| Domains | `customer_support`, `travel`, `shopping_assistant` |

## Learning method (registration note)

> **Deterministic, label-free rule mining from train trajectories.** No LLM is
> used for learning; the rule set is mined offline and read-only at eval time,
> so the "memory" is frozen by construction. The only learning input is
> `datasets/train_task_trajectories/<domain>/` (conversation + tool calls the
> protocol-locked agent observed). Task-definition `state_requirements`, judge
> verdicts, and any scored trajectory field are refused at load time by
> `brains/statebench/compliant_forge/allowed_inputs.py`
> (`ground_truth_used=false`, `labels_used=false`). See `contamination_proof.md`
> for the machine-checked covered ∩ test = 0 result.

## Artifact fingerprint (mined rule sets)

| Domain | train demos | families | if/else rules | covered ids (unique) | covered ∩ test |
|---|---:|---:|---:|---:|---:|
| customer_support | 100 | 7 | 189 | 65 | **0** |
| travel | 100 | 7 | 155 | 70 | **0** |
| shopping_assistant | 100 | 8 | 107 | 61 | **0** |

## Exact run command (per domain — frozen config)

The agent routes to the correct domain rule set via `COMPLIANT_FORGE_DOMAIN` and
reads the v2 artifact root via `COMPLIANT_FORGE_DIR`. Run from the pristine
STATE-Bench checkout.

```bash
export PYTHONUTF8=1 PYTHONIOENCODING=utf-8
export COMPLIANT_FORGE_DIR=/home/simon/Cursor/Multitude2-0/logs/compliant_forge_v2
export MULTITUDE_ROOT=/home/simon/Cursor/Multitude2-0

# DOMAIN in {customer_support, travel, shopping_assistant}
export COMPLIANT_FORGE_DOMAIN=<DOMAIN>

uv run python -m state_bench.scripts.run_batch \
  --domain <DOMAIN> --split test \
  --agent-class CompliantForgeAgent --retrieve-learnings-top-k 3 \
  --agent-provider openai --agent-model-name gpt-5.4-nano \
  --agent-model-reasoning-level high \
  --num-runs 5 --num-workers 4 \
  --output-dir outputs/<DOMAIN>_official5
```

## Exact metrics command (per domain)

```bash
uv run python -m state_bench.scripts.compute_metrics \
  --domain <DOMAIN> --split test --num-runs 5 \
  --results-dir outputs/<DOMAIN>_official5 \
  --save-filepath outputs/<DOMAIN>_official5/metrics.json
```

## Results (official5, file-accurate)

> pass@1 is the per-domain mean ± std across the 5 runs; pass^5 is the
> all-5-runs-pass rate; state_met / task_met are the mean per-run
> requirement-satisfaction rates aggregated from `per_task_metrics/`; UX and
> pass@1/pass^5 come from the standardized `metrics.json` block. pass@5 is the
> any-of-5-runs pass rate.

| Domain | pass@1 (mean ± std) | pass^5 | pass@5 | state_req_met | task_req_met | Mean UX | avg cost/task |
|---|---|---|---|---|---|---|---|
| customer_support | 0.61 ± 0.05 | 0.38 | 0.80 | 0.88 | 0.64 | 3.92 | $0.00 |
| travel | 0.65 ± 0.05 | 0.32 | 0.86 | 0.85 | 0.71 | 3.40 | $0.00 |
| shopping_assistant | 0.66 ± 0.04 | 0.48 | 0.86 | 0.93 | 0.67 | 3.82 | $0.00 |
| **Average** | **0.64** | **0.39** | **0.84** | **0.89** | **0.67** | **3.71** | **$0.00** |

avg cost/task is $0.00 because the agent runs unpriced (`agent_pricing: null`);
`mean_cost_usd` is 0.0 in every domain `metrics.json`.

## Post-completion checklist (E / F / G)

- [x] `outputs/<DOMAIN>_official5/run{1..5}/<task_id>.json` present for all 50
      test tasks × 5 runs (250 scored trajectories) per domain.
- [x] `outputs/<DOMAIN>_official5/metrics.json` present and carries protocol id
      `state_bench_v0.8.0_gpt54`.
- [x] No post-run tuning; the frozen artifact was unchanged after the run.
- [x] Results table filled from the metrics files and propagated into
      `SUBMISSION.md`.
