# Contamination / Clean-Protocol Proof (H4) — compliant_forge (forge v2)

Machine-checked assertion that the learned rule sets were built ONLY from `datasets/train_task_trajectories/<domain>/` and that the set of training ids cited as rule evidence has **zero overlap** with the official STATE-Bench `test` split (covered ∩ test = 0).

- Ruleset root: `/home/simon/Cursor/Multitude2-0/logs/compliant_forge_v2`
- Split files: `/home/simon/Cursor/MultitudeBench/third_party/STATE-Bench/state_bench/domains/<domain>/splits/train_test.json`
- Reproduce: `PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python contamination_check.py`

**OVERALL: PASS**

## Per-domain result

| Domain | train ids | test ids | covered (unique) | covered∩test | outside-split | provenance clean | verdict |
|---|---:|---:|---:|---:|---:|---|---|
| customer_support | 100 | 50 | 65 | 0 | 0 | yes | PASS |
| travel | 100 | 50 | 70 | 0 | 0 | yes | PASS |
| shopping_assistant | 100 | 50 | 61 | 0 | 0 | yes | PASS |

## Provenance boundary (allowed_inputs re-affirmation)

### customer_support

- `input_boundary`: `datasets/train_task_trajectories/ only (conversation + tool calls)`
- `ground_truth_used`: `False`
- `labels_used`: `False`
- `extraction`: `deterministic (no LLM); policy branches surfaced from get_policies env docs`
- `learner`: `compliant_forge`  •  `n_train_demos`: `100`  •  `n_families`: `7`
- covered ∩ test = **0**; outside-split = **0**

### travel

- `input_boundary`: `datasets/train_task_trajectories/ only (conversation + tool calls)`
- `ground_truth_used`: `False`
- `labels_used`: `False`
- `extraction`: `deterministic (no LLM); policy branches surfaced from get_policies env docs`
- `learner`: `compliant_forge`  •  `n_train_demos`: `100`  •  `n_families`: `7`
- covered ∩ test = **0**; outside-split = **0**

### shopping_assistant

- `input_boundary`: `datasets/train_task_trajectories/ only (conversation + tool calls)`
- `ground_truth_used`: `False`
- `labels_used`: `False`
- `extraction`: `deterministic (no LLM); policy branches surfaced from get_policies env docs`
- `learner`: `compliant_forge`  •  `n_train_demos`: `100`  •  `n_families`: `8`
- covered ∩ test = **0**; outside-split = **0**

The full covered-id and test-id sets per domain are in `contamination_proof.json`.

