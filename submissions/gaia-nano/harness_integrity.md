# Harness-integrity proof — compliant_forge (forge v2) nano

Clean Protocol Checklist section **C** (Harness / protocol cleanliness) and
**G** (minimum audit bundle). Captured read-only; no eval was launched.

Captured: 2026-08-14 (Europe/Berlin). All commands below are read-only git
inspection.

## Repository SHAs (Run Registration / D)

| Repo | Path | Commit SHA | Branch |
|---|---|---|---|
| STATE-Bench (locked harness) | `/home/simon/Cursor/MultitudeBench/third_party/STATE-Bench` | `e2c8d7af51ef48fbbea51bb2ce1fb859af36b423` | detached `HEAD` |
| Multitude2-0 (learner + agent hook) | `/home/simon/Cursor/Multitude2-0` | `97cd3f141c4eb82d26d0a80d0078937b547692ec` | `STATEBench` |

STATE-Bench HEAD commit: `e2c8d7af Merge pull request #39 from microsoft/dev`
(2026-06-25).

## STATE-Bench version / protocol pin

- Package version: `0.8.0` (`pyproject.toml` → `version = "0.8.0"`).
- Protocol id: `state_bench_v0.8.0_gpt54` (`DEFAULT_PROTOCOL_KEY = "gpt54"`,
  `build_protocol_id()` → `state_bench_{benchmark_version}_gpt54`).
- Locked protocol config `state_bench/configs/eval_protocols/gpt54.json`:
  - `split: test`, `num_runs: 5`, `official_model: gpt-5.4`
  - simulator `gpt-5.4`; judge `gpt-5.4`, `reasoning_effort: high`
  - simulator + judge prompt sha256 hashes pinned for all three domains.

## Locked-surface cleanliness

Command: `git -C <STATE-Bench> status --short`

```
?? agents/
?? clients/
```

Command: `git -C <STATE-Bench> diff --stat` → **(empty)**
Command: `git -C <STATE-Bench> diff --stat HEAD` → **(empty)**

**Zero tracked files are modified.** No change to any locked surface:
`state_bench/` (protocol, scorer, judge/simulator clients, tasks, task_envs,
domain tools, judge prompts), `datasets/`, or config files. The only working-tree
additions are the two untracked *extension* directories STATE-Bench documents for
user code (`agents/`, `clients/`) — these are NOT locked/protocol files.

### Untracked extension files present (full listing)

```
agents/compliant_forge_agent.py     <- the agent used by THIS submission
agents/brains_agent.py              <- unrelated prior extension (NOT used here)
clients/brains_client.py            <- unrelated prior extension (NOT used here)
agents/__pycache__/*.pyc            <- bytecode caches
clients/__pycache__/*.pyc
```

Notes for the auditor (full disclosure):

- The submission agent is **`agents/compliant_forge_agent.py`** →
  `CompliantForgeAgent`, a `StateBenchAgent` subclass whose only addition is the
  read-only `retrieve_learnings` hook (harness still owns tools, simulator,
  judge). Source reviewed: it injects lightweight `brains` / `brains.statebench`
  namespace packages then imports
  `brains.statebench.compliant_forge.agent_hook.CompliantForgeAgent`. It touches
  no protocol code.
- `agents/brains_agent.py` and `clients/brains_client.py` are leftover extension
  files from an earlier approach; they are **not referenced** by the
  compliant_forge run (`--agent-class CompliantForgeAgent`, default built-in
  client). For a maximally clean pristine checkout they may optionally be removed
  before the official run, but they do not affect the locked surface or the
  scored trajectories because they are never imported by this agent class.

## Verdict

- Locked STATE-Bench surface: **UNMODIFIED** (no tracked diffs). PASS.
- Only user extension dirs added; submission uses the allowed
  `agents/compliant_forge_agent.py` hook. PASS.
- Protocol pin: `state_bench_v0.8.0_gpt54`, num_runs=5, top_k=3 (enforced at run
  time by the eval command — see `final_run_manifest.md`). PASS (config-level).

## Reproduce

```bash
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench rev-parse HEAD
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench status --short
git -C /home/simon/Cursor/MultitudeBench/third_party/STATE-Bench diff --stat HEAD
git -C /home/simon/Cursor/Multitude2-0 rev-parse HEAD
```
