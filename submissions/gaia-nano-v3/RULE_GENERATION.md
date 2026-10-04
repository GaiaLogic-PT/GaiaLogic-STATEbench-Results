# Rule generation & provenance — compliant_forge_v3 (clean train-only baseline)

This document records exactly how the v3 rule artifact was produced, why it is a
clean train-only / label-free / frozen-before-any-judge baseline, and the full
disclosure of a dev-refinement that was explored and then reverted.

## Inputs (the only learning signal)

- `datasets/train_task_trajectories/<domain>/<id>.json` restricted to the
  **`train_fit`** id subset (customer_support 69, travel 70, shopping 70 of the
  100 train ids). Each file is the recorded `conversation`: user/agent messages
  and `{name, arguments, result}` tool calls the protocol-locked agent observed.
- The complementary `train_dev` slice (31 / 30 / 30) is **held out** and is not
  read during authoring.
- The split is defined in `../compliant_forge_nano/dev_split.json`
  (family-disjoint; deterministic; label-free).

No test tasks, no test labels/outcomes, no ground-truth `state_requirements`, no
judge verdicts, and no simulator output are inputs to any rule.

## Boundary enforcement (compliance-by-construction)

`brains/statebench/compliant_forge/allowed_inputs.py` enforces the boundary at
load time:

- **Path guard** — refuses any path not under a `train_task_trajectories`
  directory.
- **Field guard (recursive)** — refuses any object carrying a benchmark
  label/oracle/judge field (`state_requirements`, `state_requirements_gt`,
  `task_completion_pass`, `state_diff`, `ux_score`, judge fields, …).
- **Projection** — keeps only `conversation`; drops every other top-level key.

So `ground_truth_used=false` / `labels_used=false` is enforced, not merely
asserted, and is independently re-checked by `contamination_proof_v3.md`.

## Two build layers (both offline, judge-free)

1. **Deterministic base rules** — `extract.py` mines per-(domain, family)
   if/else rules from observable regularities in the `train_fit` demos only
   (conduct discipline, tool path, policy branches surfaced verbatim from
   `get_policies` results, disclosure components). No LLM call.
   Emitted as `logs/compliant_forge_v3/<domain>/ruleset_fit.json`
   (`provenance.split = "train_fit only (dev held out)"`).

2. **Enriched per-family playbooks** — `clean_pipeline/playbooks.py`: dense,
   structured operating manuals per family (scope, tool order, policy rails with
   arithmetic, disclosure, common-failure guards) authored offline by reasoning
   over the `train_fit` conversations + their `get_policies` results. Emitted as
   `logs/compliant_forge_v3/<domain>/playbook.json` and rendered into
   `super_prompt_v3.txt`. This exploits the uncapped per-string length under the
   fixed `top_k=3` retrieval cap. It is authored **without any judge / label /
   test / dev-outcome signal**.

Reproduce (deterministic, judge-free):

```bash
cd Submissions/compliant_forge_nano/clean_pipeline
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python3 build_v3.py   # -> logs/compliant_forge_v3/
```

`build_v3.py` reads `playbooks.py` as its authoring source, so the on-disk
artifact is exactly `build_v3(playbooks.py)`.

## Frozen-before-any-judge guarantee

`clean_pipeline/playbooks.py` is git-tracked and was committed **before** any v3
evaluation. The shipped `logs/compliant_forge_v3/` artifact is the deterministic
rebuild of that committed source. The one-shot `--split test` run reads this
frozen artifact and nothing is changed afterward.

## Full disclosure: a dev-refinement was explored and REVERTED

During development, a `train_dev` eval was run and a single family-level
refinement of `playbooks.py` was drafted from the train_dev failure clusters
(judge reasoning + pass rates on the **train_dev** tasks). Because that
refinement was driven by judge/dev-outcome feedback, it is **incompatible with a
frozen-before-any-judge, no-refinement baseline** and was **fully reverted**
(`git restore playbooks.py`) before this submission's artifact was rebuilt. The
refined source/artifact are quarantined under
`logs/_compliant_forge_v3_REFINED_quarantine_<ts>/` and are **not** part of this
submission. The shipped rules contain none of those refinements (verified: the
refine-only marker strings are absent from every `playbook.json`), and the base
`ruleset_fit.json` was never affected by the refinement (byte-identical between
the refined and clean builds — only the authored playbooks differed).

> Note on byte-identity: because `build_v3.py` rebuilds from `playbooks.py`,
> "on-disk artifact == fresh build" only proves the artifact matches the current
> source. The clean guarantee rests on `playbooks.py` being the **git-committed,
> unrefined** source (confirmed by an empty `git diff`), not on byte-identity to
> a rebuild.

## Contamination audit

`clean_pipeline/audit_v3.py` (judge-independent, offline) asserts per domain:
every rule's evidence ids ⊆ `train_fit`; covered ∩ test = 0; train_fit ∩ test =
0; train_dev ∩ test = 0; train_fit ∩ train_dev = 0; and clean provenance. Latest
run: **OVERALL PASS** (see `contamination_proof_v3.md`).
