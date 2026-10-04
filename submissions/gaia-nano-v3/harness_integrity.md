# Harness-integrity proof — compliant_forge_v3 (clean train-only baseline)

Captured read-only via git inspection. No locked/protocol file is modified; the
only working-tree additions are the allowed `agents/` (and prior `clients/`)
extension dirs STATE-Bench documents for user code.

## Repository SHAs

| Repo | Path | Commit SHA | Branch |
|---|---|---|---|
| STATE-Bench (locked harness) | `/home/simon/Cursor/MultitudeBench/third_party/STATE-Bench` | `e2c8d7af51ef48fbbea51bb2ce1fb859af36b423` | detached `HEAD` |
| Multitude2-0 (learner + agent hook) | `/home/simon/Cursor/Multitude2-0` | `a3ee01c951455ad25b7fefa08370257b2c48b2e5` | `statebench/compliant-forge-nano-submission` |

## STATE-Bench version / protocol pin

- Package version `0.8.0`; protocol id `state_bench_v0.8.0_gpt54`.
- Locked protocol: `split: test`, `num_runs: 5`, simulator `gpt-5.4`, judge
  `gpt-5.4` `reasoning_effort: high`.
- Confirm the protocol id in each domain `metrics.json` (`evaluation_protocol_id`).

## Locked-surface cleanliness

```
git -C <STATE-Bench> status --short
?? agents/
?? clients/

git -C <STATE-Bench> diff --stat        -> (empty)
git -C <STATE-Bench> diff --stat HEAD    -> (empty)
```

**Zero tracked files modified** — no change to `state_bench/` (protocol, scorer,
judge/simulator clients, tasks, task_envs, tools, judge prompts), `datasets/`,
or configs.

### Untracked extension files present

```
agents/compliant_forge_v3_agent.py   <- the agent used by THIS submission (CompliantForgeV3Agent)
agents/compliant_forge_agent.py      <- v2 agent (not used here)
agents/brains_agent.py               <- unrelated prior extension (not used here)
clients/brains_client.py             <- unrelated prior extension (not used here)
```

`agents/compliant_forge_v3_agent.py` injects lightweight `brains` /
`brains.statebench` namespace packages, then imports
`brains.statebench.compliant_forge.agent_hook_v3.CompliantForgeV3Agent` — a
`StateBenchAgent` subclass whose only addition is the read-only
`retrieve_learnings` hook (harness still owns tools, simulator, judge). It
touches no protocol code and reads the frozen `logs/compliant_forge_v3` artifact
via `COMPLIANT_FORGE_DIR`.

## Verdict

- Locked STATE-Bench surface: **UNMODIFIED** (no tracked diffs). PASS.
- Submission uses the allowed `agents/compliant_forge_v3_agent.py` hook. PASS.
- Protocol pin `state_bench_v0.8.0_gpt54`, num_runs=5, top_k=3 (enforced at run
  time — see `final_run_manifest.md`). PASS (config-level; confirm from
  `metrics.json`).

## Reproduce

```bash
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench rev-parse HEAD
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench status --short
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench diff --stat HEAD
git -C /home/simon/Cursor/Multitude2-0 rev-parse HEAD
```
