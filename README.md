# GaiaLogic — STATE-Bench Results

Public results and documentation for **GaiaLogic**'s submissions to the
[STATE-Bench](https://github.com/microsoft/STATE-Bench) **Agent Learning Track**.

Our system is **`gaia-nano`**: an offline, deterministic, **label-free**
rule-learning agent whose **entire pipeline runs on a single small model,
`gpt-5.4-nano`** — nothing larger anywhere. Rule learning uses **no LLM at all**
(rules are mined deterministically from the allowed train trajectories); the only
model in the loop at evaluation time is the `gpt-5.4-nano` agent under test.

This repository contains the **plain submission artifacts and written
explanations only**. It intentionally **does not** include the pipeline / agent
source code used to produce the results — see
[Reproduction code](#reproduction-code-not-included).

## What's here

```
README.md                       <- this file
REPORT.md                       <- full report: what it is, method, results, compliance
submissions/
  gaia-nano/                    <- primary submission (label-free, test-adapted at design level)
    outputs.zip                 <- 3 domains x 50 test tasks x 5 runs = 750 scored trajectories + metrics.json
    SUBMISSION.md               <- method write-up + results
    final_run_manifest.md       <- run registration, exact eval config, file-accurate results
    harness_integrity.md        <- pristine locked-harness proof + commit SHAs + protocol pin
    contamination_proof.md      <- machine-checked: learned rules ∩ test split = 0
    contamination_proof.json
  gaia-nano-v3/                 <- companion submission (train-only; NEVER saw the test set)
    outputs.zip
    SUBMISSION.md
    final_run_manifest.md
    harness_integrity.md
    RULE_GENERATION.md          <- how the train-only rules were built; frozen-before-any-judge
    contamination_proof_v3.md
    contamination_proof_v3.json
```

## Two submissions

| Submission | What it is | Saw the test set? |
|---|---|---|
| **`gaia-nano`** | Label-free rule learning from all 100 train trajectories per domain. Label-free, but **design-level test-adapted**: an earlier iteration was run on the test split and its results were examined by engineers, which informed general prompt / conduct-rail design changes. No labels or test fields ever entered the agent or the learning harness. | Design informed by inspecting earlier test-set behavior (disclosed). |
| **`gaia-nano-v3`** | Strictly **train-only** rebuild: rules authored from a family-disjoint `train_fit` subset, **frozen before any evaluation / judge / simulator call**, never refined. | **No — never, in any way.** |

Both are evaluated identically: locked Azure `gpt-5.4` simulator + judge
(protocol `state_bench_v0.8.0_gpt54`, judge reasoning `high`), agent
`gpt-5.4-nano` reasoning `high`, `--split test` 50 held-out tasks × 5 runs,
`--retrieve-learnings-top-k 3`.

## Headline results (macro average over the 3 domains)

| System | pass@1 | pass^5 | pass@5 | Mean UX |
|---|---|---|---|---|
| `gaia-nano` (test-adapted) | 0.64 | 0.39 | 0.84 | 3.71 |
| `gaia-nano-v3` (train-only) | 0.62 | 0.38 | 0.81 | 3.68 |

Per-domain tables and full compliance evidence are in [`REPORT.md`](REPORT.md) and
in each submission's `SUBMISSION.md`.

## Compliance in one line

No ground-truth labels, task descriptions, or any other task/environment field —
from train or test — are ever fed into the agent or the learning harness. The
input boundary is enforced at load time and independently machine-checked
(covered ∩ test = 0 for all three domains). `gaia-nano-v3` additionally never saw
the test set at any stage.

## Reproduction code (not included)

By design, this repository ships only the **submission artifacts** (scored
trajectories + metrics) and **written explanations** of the method. The
rule-mining pipeline and agent source code are kept private and are **not**
included here. The documents describe the method, the exact evaluation
configuration, and the compliance guarantees; they reference internal module
names (e.g. `allowed_inputs.py`, `extract.py`) only for explanatory purposes.
