# STATE-Bench leaderboard submission — compliant_forge_v3 (clean train-only baseline)

**Track:** Agent Learning Track (offline, label-free learning from train demonstrations)
**Benchmark version:** 0.8.0
**Evaluation protocol:** `state_bench_v0.8.0_gpt54` (locked Azure GPT-5.4 simulator + judge, judge reasoning `high`)
**Agent under test:** `CompliantForgeV3Agent` — OpenAI `gpt-5.4-nano`, reasoning level `high`
**Label:** Clean, train-only, label-free, **frozen-before-any-judge** baseline (no refinement)

## What this submission is (and is not)

`compliant_forge_v3` is the **strictly train-only, label-free** rebuild of the
compliant_forge learner. Its distinguishing property versus the earlier v2 run:

- **Rules are authored from `train_fit` demonstrations ONLY** — a
  **family-disjoint** subset of each domain's train split (69/70/70 of the 100
  train trajectories). The complementary `train_dev` slice is held out.
- The shipped artifact is **frozen before any evaluation / judge / simulator
  call** and was **not refined** using dev results, judge reasoning, pass rates,
  or any other feedback. (A dev-driven refinement was explored during
  development and then **fully reverted**; the shipped rules are the
  git-committed, train-only source rebuilt deterministically — see
  `RULE_GENERATION.md` and `final_run_manifest.md`.)
- **No test task, test label, test outcome, or ground-truth requirement label**
  informs any rule. This is machine-checked in `contamination_proof_v3.md`
  (covered ∩ test = 0 for all three domains; `ground_truth_used=false`,
  `labels_used=false`, `judge_used=false`).

It is therefore a genuine **imitation-from-demonstrations** baseline: what an
agent learns purely by reading allowed train_fit conversations, with the test
split kept a true holdout touched exactly once.

## Results (50 test tasks × 5 runs per domain)

Full 50×5 held-out coverage (250 scored trajectories per domain) under the
locked GPT-5.4 simulator + judge, one-shot on `--split test`. pass@1 is the
per-run mean ± std across the 5 runs; pass^5 is the all-5-runs-pass rate; pass@5
is the any-of-5-runs pass rate; state_req_met / task_req_met are the mean
per-run requirement-satisfaction rates; UX is the mean judge UX score. All
numbers are read verbatim from each domain's `metrics.json` /
`per_task_metrics/`.

| Domain | pass@1 (mean ± std) | pass^5 | pass@5 | state_req_met | task_req_met | Mean UX | cost/task |
|---|---|---|---|---|---|---|---|
| customer_support | 0.57 ± 0.03 | 0.32 | 0.76 | 0.84 | 0.61 | 3.87 | $0.00 |
| travel | 0.66 ± 0.05 | 0.34 | 0.88 | 0.87 | 0.69 | 3.38 | $0.00 |
| shopping_assistant | 0.64 ± 0.02 | 0.48 | 0.80 | 0.92 | 0.66 | 3.79 | $0.00 |
| **Macro avg** | **0.62** | **0.38** | **0.81** | **0.88** | **0.65** | **3.68** | **$0.00** |

cost/task is $0.00 because the agent runs unpriced (`agent_pricing: null`);
`mean_cost_usd` is 0.0 in every `metrics.json`.

### Reference: v2 official5 (for context, NOT a claim about v2's cleanliness)

| Domain | v2 pass@1 | v2 pass^5 |
|---|---|---|
| customer_support | 0.61 | 0.38 |
| travel | 0.65 | 0.32 |
| shopping_assistant | 0.66 | 0.48 |

v2 mined rules from all 100 train trajectories; v3 restricts authoring to
`train_fit` and is frozen before any judge feedback. The two runs share the
identical eval configuration (below), so v3-vs-v2 is a like-for-like comparison
of the clean train-only baseline against the earlier config.

## Method

`CompliantForgeV3Agent` is a thin `StateBenchAgent` subclass whose only addition
is a read-only `retrieve_learnings(query, top_k=3) -> list[str]` hook. The
harness still owns the tools, the simulated user, and the judge; the agent only
adds retrieval of a frozen, train-derived operating manual.

### 1. Compliance-by-construction input boundary

All learning signal flows through `allowed_inputs.py`, which accepts **only**
`datasets/train_task_trajectories/<domain>/<id>.json` (the recorded
`conversation` — messages + `{name, arguments, result}` tool calls the
protocol-locked agent observed), refuses any path outside a
`train_task_trajectories` directory, and refuses any object carrying a
benchmark label/oracle/judge field (`state_requirements`,
`state_requirements_gt`, `task_completion_pass`, `ux_score`, judge fields, …).
v3 additionally restricts mining to the `train_fit` id subset.

### 2. Deterministic (no-LLM) base rules from train_fit

`extract.py` mines per-(domain, family) if/else rules from observable
regularities in the `train_fit` demos only (conduct, tool path, policy branches
surfaced verbatim from `get_policies` results, disclosure components). No LLM
call — the base build is free, offline, and reproducible.

### 3. Enriched per-family playbooks (authored offline from train_fit)

`retrieve_learnings` is hard-capped at `top_k=3`, but each retrieved string has
no length limit, so each family carries a dense procedural playbook (scope,
tool order, policy rails with arithmetic, disclosure, failure guards) authored
offline from the `train_fit` trajectories + their `get_policies` results. This
is the legitimate lever, and it is authored **without any judge/label/test
signal**.

### 4. Agent wiring

`agent_hook_v3.py` exposes `CompliantForgeV3Agent`; `retrieval_v3.py` layers the
per-family playbook on top of the deterministic base block. The checkout entry
`agents/compliant_forge_v3_agent.py` injects lightweight `brains` /
`brains.statebench` namespace packages, then imports the hook. The domain is
taken from `COMPLIANT_FORGE_DOMAIN`; the artifact root from `COMPLIANT_FORGE_DIR`
(`logs/compliant_forge_v3`). All new modules are additive — the live v2
`super_prompt.py` / `agent_hook.py` and the `logs/compliant_forge*` v1/v2
artifacts are untouched.

## Exact evaluation configuration (mirrors v2 official5)

- Pristine STATE-Bench **v0.8.0** checkout
  (`e2c8d7af51ef48fbbea51bb2ce1fb859af36b423`); no tracked-file modifications
  (see `harness_integrity.md`); only the allowed `agents/` extension is added.
- Simulator + judge: locked Azure `gpt-5.4` (judge reasoning `high`), protocol
  `state_bench_v0.8.0_gpt54`.
- Agent: `CompliantForgeV3Agent`, OpenAI `gpt-5.4-nano`, reasoning `high`.
- `--split test` (50 held-out tasks/domain), `--num-runs 5`,
  `--retrieve-learnings-top-k 3`, `--num-workers 4` — identical to v2 official5.
- Metrics via `state_bench.scripts.compute_metrics --split test --num-runs 5`.

Reproduction commands are in `final_run_manifest.md`.

## Compliance / clean-protocol evidence (bundled)

- `contamination_proof_v3.md` / `.json` — machine-checked covered ⊆ train_fit,
  covered ∩ test = 0, train_fit ∩ test = 0, train_dev ∩ test = 0,
  train_fit ∩ train_dev = 0 for all three domains; provenance
  `ground_truth_used=false`, `labels_used=false`, `judge_used=false`.
- `RULE_GENERATION.md` — the exact train-only build (base miner + offline
  playbooks), the frozen-before-any-judge guarantee, and the reverted-refinement
  disclosure.
- `harness_integrity.md` — pristine STATE-Bench surface (empty tracked diff) +
  both repo SHAs + protocol pin.
- `final_run_manifest.md` — full run registration with the exact frozen run +
  metrics commands and the file-accurate results.

## Attachments

- `outputs.zip` — scored trajectories (`outputs/<domain>/runN/<task_id>.json`)
  and `metrics.json` for all three domains from the one-shot clean test run.
