# Contamination + Split-Discipline Proof (v3) — compliant_forge_nano CLEAN pipeline

Judge-independent, offline. Asserts the true-holdout guarantees for the v3
`train_fit`-authored artifact. Reproduce: `python audit_v3.py`.

**OVERALL: PASS**

| Domain | fit | dev | test | covered | covered⊆fit | covered∩test | fit∩test | dev∩test | fit∩dev | CS-caveat∉dev | prov | verdict |
|---|--:|--:|--:|--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| customer_support | 69 | 31 | 50 | 55 | OK | OK | OK | OK | OK | OK | OK | PASS |
| travel | 70 | 30 | 50 | 53 | OK | OK | OK | OK | OK | OK | OK | PASS |
| shopping_assistant | 70 | 30 | 50 | 46 | OK | OK | OK | OK | OK | OK | OK | PASS |

- `covered⊆fit`: every rule evidence id is a train_fit id (rails authored from fit only).
- `covered∩test`, `fit∩test`, `dev∩test`: zero test contamination (test is a true holdout).
- `fit∩dev`: fit and dev are disjoint (dev is a clean generalization holdout).
- `CS-caveat∉dev`: the baseline-used / prior-analysis CS ids never appear in the dev holdout.
- `prov`: artifact provenance has `ground_truth_used=false`, `labels_used=false`,
  `input_boundary=train_task_trajectories only`.

