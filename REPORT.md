# GaiaLogic on STATE-Bench — Report

**Track:** Agent Learning Track
**Benchmark:** STATE-Bench v0.8.0
**Evaluation protocol:** `state_bench_v0.8.0_gpt54` — locked Azure `gpt-5.4`
simulator + judge, judge reasoning `high`
**Agent under test:** OpenAI `gpt-5.4-nano`, reasoning level `high`
**Coverage:** 3 domains (`customer_support`, `travel`, `shopping_assistant`) ×
50 held-out test tasks × 5 runs = **750 scored trajectories per submission**

---

## 1. What this is

**`gaia-nano`** is an offline, deterministic, **label-free** rule-learning system
for the STATE-Bench Agent Learning Track. Its defining properties:

- **The whole agentic system runs on one small model, `gpt-5.4-nano`** — and
  nothing larger anywhere.
  - **Rule learning uses no LLM at all.** Per-(domain, family) if/else rules are
    mined deterministically from observable regularities in the allowed train
    trajectories (conduct discipline, tool paths, and policy branches surfaced
    verbatim from the `get_policies` environment documents the agent observed).
  - **The only model in the loop at eval time is the `gpt-5.4-nano` agent.** It
    receives the mined rules through a read-only
    `retrieve_learnings(query, top_k=3)` hook. The harness still owns the tools,
    the simulated user, and the judge.
  - We use **no `gpt-5.4` (full), no separate planner/reasoning model, and no LLM
    for rule induction**. (The `gpt-5.4` simulator and judge are the locked
    STATE-Bench harness, not part of our system.)

This makes the submission cheap, reproducible, and easy to audit: the "memory" is
a frozen, human-readable rule set, and the agent is a thin retrieval layer on top
of a small model.

## 2. The two submissions

We provide two variants that differ only in how the rules were authored and
whether the test set was ever observed.

### `gaia-nano` (primary — label-free, design-level test-adapted)

Rules are mined from **all 100 train trajectories** per domain. It is **label-free**
— no labels, task descriptions, or any other task/environment field (train or
test) ever enter the agent or the learning harness. 

In the interest of full transparency: this variant is **test-adapted at the
design level**. An *earlier* iteration of the system was run on the test split and
our engineers examined those results; based on that examination we made general
architectural changes (primarily prompt engineering / conduct-rail design) that
yielded a moderate improvement. This was a one-time, human-in-the-loop design
adaptation. The submitted frozen artifact was not changed after its scored run,
but its *design* was informed by inspecting prior test-set behavior. We disclose
this explicitly rather than leave it implicit.

### `gaia-nano-v3` (companion — strictly train-only, never saw test)

A clean rebuild that is **frozen before any evaluation / judge / simulator call**
and **never refined**:

- Rules are authored from a **family-disjoint `train_fit` subset** of each
  domain's train split (69 / 70 / 70 of the 100 train trajectories); the
  complementary `train_dev` slice is held out.
- **No test task, test label, test outcome, judge verdict, or ground-truth
  requirement** informs any rule, at any stage.
- A dev-driven refinement was explored during development and then **fully
  reverted**; the shipped rules are the git-committed, train-only source rebuilt
  deterministically. (Full disclosure in `submissions/gaia-nano-v3/RULE_GENERATION.md`.)

This is a genuine imitation-from-demonstrations baseline: what the agent learns
purely by reading allowed train conversations, with the test split kept a true
holdout touched exactly once.

## 3. Results

Both variants use the identical evaluation configuration, so they are directly
comparable. pass@1 is the per-run mean ± std across the 5 runs; pass^5 is the
all-5-runs-pass rate; pass@5 is the any-of-5-runs pass rate; state/task_req_met
are the mean per-run requirement-satisfaction rates; UX is the mean judge UX
score. Numbers are read verbatim from each domain's `metrics.json` and
`per_task_metrics/`.

### `gaia-nano` (test-adapted)

| Domain | pass@1 (mean ± std) | pass^5 | pass@5 | state_req_met | task_req_met | Mean UX |
|---|---|---|---|---|---|---|
| customer_support | 0.61 ± 0.05 | 0.38 | 0.80 | 0.88 | 0.64 | 3.92 |
| travel | 0.65 ± 0.05 | 0.32 | 0.86 | 0.85 | 0.71 | 3.40 |
| shopping_assistant | 0.66 ± 0.04 | 0.48 | 0.86 | 0.93 | 0.67 | 3.82 |
| **Macro avg** | **0.64** | **0.39** | **0.84** | **0.89** | **0.67** | **3.71** |

### `gaia-nano-v3` (train-only, never saw test)

| Domain | pass@1 (mean ± std) | pass^5 | pass@5 | state_req_met | task_req_met | Mean UX |
|---|---|---|---|---|---|---|
| customer_support | 0.57 ± 0.03 | 0.32 | 0.76 | 0.84 | 0.61 | 3.87 |
| travel | 0.66 ± 0.05 | 0.34 | 0.88 | 0.87 | 0.69 | 3.38 |
| shopping_assistant | 0.64 ± 0.02 | 0.48 | 0.80 | 0.92 | 0.66 | 3.79 |
| **Macro avg** | **0.62** | **0.38** | **0.81** | **0.88** | **0.65** | **3.68** |

### Side by side (pass@1 / pass^5)

| Domain | `gaia-nano` | `gaia-nano-v3` |
|---|---|---|
| customer_support | 0.61 / 0.38 | 0.57 / 0.32 |
| travel | 0.65 / 0.32 | 0.66 / 0.34 |
| shopping_assistant | 0.66 / 0.48 | 0.64 / 0.48 |
| **macro** | **0.64 / 0.39** | **0.62 / 0.38** |

**Takeaway.** The strictly train-only `gaia-nano-v3` is only marginally behind the
test-adapted `gaia-nano` on macro average (0.62 vs 0.64 pass@1), and on `travel` it
is actually slightly ahead. The design-level test adaptation bought a small,
honest improvement — not a large one — which is consistent with the deterministic,
label-free nature of the learner.

All agents run unpriced (`agent_pricing: null`), so reported cost/task is $0.00 in
every `metrics.json`.

### Comparison to the verified leaderboard (same benchmark version, v0.8.0)

The verified Agent Learning Track entries on the same benchmark version (v0.8.0),
from the
[public leaderboard](https://microsoft.github.io/STATE-Bench/leaderboard/?track=agent-learning),
with our two submissions beneath (percentages, macro average):

| System | Organization | pass@1 | pass^5 | UX | Status |
|---|---|---|---|---|---|
| GPT-5.4 + Foundry Memory | Microsoft Foundry | 54.5% | 33.6% | 3.75 | verified |
| GPT-5.4, no memory (baseline) | OpenAI | 51.3% | 29.9% | 3.31 | verified |
| **gaia-nano** (ours) | GaiaLogic | 64.0% | 39.3% | 3.71 | **not verified** |
| **gaia-nano-v3** (ours) | GaiaLogic | 62.3% | 38.0% | 3.68 | **not verified** |

On the same version, both of our runs score above the verified entries — including
a full-size GPT-5.4 with a dedicated memory system — from a much smaller model
(`gpt-5.4-nano`) carrying only a frozen, human-readable rule set. We present this
with deliberate caution: our numbers are **self-reported on the locked protocol and
not yet verified by Microsoft**, and a result of this shape warrants independent
scrutiny before it is taken as established. Only same-version entries are compared
here; the older v0.4.4 leaderboard numbers (e.g. GPT-5.1 + Foundry Memory at 58.3%)
run a different evaluation protocol and are not directly comparable. As a wider
bearing, the strongest Main-Track model on v0.8.0 (GPT-5.5, high reasoning) sits at
58.9% pass@1. Even taken at face value, the pass^5 figures (~38–39%) show that the
agent still fails at least one of five runs on the majority of tasks — there is
substantial headroom left.

## 4. Method (summary)

1. **Compliance-by-construction input boundary.** All learning signal flows
   through an input boundary that accepts only
   `datasets/train_task_trajectories/<domain>/<id>.json` (the recorded
   conversation + tool calls the protocol-locked agent observed), refuses any path
   outside a `train_task_trajectories` directory, and refuses any object carrying a
   benchmark label/oracle/judge field (`state_requirements`,
   `state_requirements_gt`, `task_completion_pass`, `ux_score`, judge fields, …).
   Only `conversation` is projected forward. So `ground_truth_used=false` /
   `labels_used=false` is **enforced at load time**, not merely asserted.
2. **Deterministic (no-LLM) rule mining** into per-(domain, family) if/else rules.
3. **Per-family operating playbooks** served through the read-only
   `retrieve_learnings` hook (hard-capped at `top_k=3`; each returned string is a
   dense procedural manual: scope, tool order, policy rails with arithmetic,
   disclosure, failure guards).
4. **Thin agent wiring.** The agent is a `StateBenchAgent` subclass whose only
   addition is the retrieval hook; it adds no tools and does not touch protocol
   code.

## 5. Compliance / clean-protocol evidence

Each submission bundles:

- **`contamination_proof.md` / `.json`** — machine-checked: every rule's evidence
  ids come only from train trajectories, and the covered train-id set has **zero
  overlap** with the test split (covered ∩ test = 0) for all three domains;
  `ground_truth_used=false`, `labels_used=false`. For `gaia-nano-v3` it
  additionally asserts `train_fit ∩ test = 0`, `train_dev ∩ test = 0`, and
  `train_fit ∩ train_dev = 0`, with `judge_used=false`.
- **`harness_integrity.md`** — the locked STATE-Bench surface is unmodified (empty
  `git diff` on tracked files); both repository commit SHAs; the protocol pin
  (`state_bench_v0.8.0_gpt54`, num_runs=5, top_k=3). Only the documented `agents/`
  user-extension is added.
- **`final_run_manifest.md`** — run registration: the exact `run_batch` /
  `compute_metrics` commands, the frozen configuration, the mined-artifact
  fingerprint per domain, and the file-accurate results.
- **`RULE_GENERATION.md`** (v3 only) — the exact train-only build, the
  frozen-before-any-judge guarantee, and the reverted-refinement disclosure.

## 6. What is and isn't in this repository

- **Included:** the submission artifacts (`outputs.zip` = scored trajectories +
  `metrics.json`) and the written method / compliance documents.
- **Not included:** the rule-mining pipeline and agent source code (kept private).
  The documents reference internal module names only to explain the method; this
  repository is not intended to reproduce the results from source.
