# Shared starting state, independent agent worlds

Status: research proposal; no new runtime behavior implemented or benchmark run.

## Idea

Give many agents the same committed database state, let each work in its own writable branch, then inspect, score, retain, or discard the resulting states. The unit of an experiment becomes a database transition: starting commit, task, actions, ending commit, and evaluation.

This connects three uses:

- **Parallel attempts:** multiple agents or policies solve the same task from the same starting state.
- **Synthetic worlds:** generate several coherent database scenarios, freeze each as a starting commit, and sample attempts within each scenario.
- **Selective continuation:** retain a useful outcome as the starting state for the next task, producing a lineage of decisions and consequences.

The research question is whether this makes stateful agent experiments easier to reproduce and cheaper to reset than creating a separate database for every attempt. Cost and throughput are hypotheses to measure.

## What “the same database” means

Agents share a schema and starting data. Independent attempts receive separate branches. Agents intentionally collaborating on one attempt can share that attempt's branch, where they must coordinate ordinary concurrent writes.

Doltgres documents branch-qualified connections and read-only connections to commit revisions. Changes on separate branches remain separate until merged. These mechanisms support the proposal, but do not establish resource isolation or credential restrictions. See [branches and revisions](https://www.doltgres.com/docs/reference/version-control/branches/) and [branch semantics](https://www.doltgres.com/docs/concepts/git/branch/).

```text
schema + deterministic synthetic seed
                  |
          world commit W0
           /      |      \
       attempt A  B      C       independent writable branches
           |      |      |
        result A  B      C       final commits + action traces
           \      |      /
          inspect and evaluate
                  |
       retain B as world W1      explicit selection
```

Selecting a complete result and merging multiple results are different operations. Start with selection. A conflict-free merge does not guarantee a valid business outcome; combining results needs its own invariant checks.

## Existing foundation

`dolt/tooling/lab.py` already implements seeded world commits, serial branch creation, parallel workers with separate connections, rollout commits, row-count checks, and branch deletion. `orchestrator.py` measures creation/work/deletion time, memory and disk around garbage collection. `lab_viewer.py` provides an inspection surface.

The current workers insert synthetic rows; they are not agent trajectories. Verification checks counts rather than complete contents. The separate permission scripts print outcomes rather than enforce an automated contract. No passing permission or performance results are assumed here.

Other gaps worth addressing before building on it:

- Check baseline immutability and cross-branch contamination explicitly.
- Preserve selected results under durable references before deleting temporary branches or running GC; a recorded hash alone is not a retention policy.
- Record verification time as part of end-to-end runtime. The current total sums creation, work, and deletion only.
- Replace global process termination and state wiping in the legacy startup helper with a process and temporary directory owned by each experiment.
- Bound worker concurrency and record partial failures; cleanup must also run when a worker fails.

## First experiment: alternate outcomes for one order book

Use a small deterministic database with customers, orders, order lines, and inventory. The task is to fulfill eligible orders under a fixed stock budget. Include an impossible order and competing orders for the same inventory so that correctness requires more than inserting rows.

1. Generate the fixture from a recorded seed and commit it as W0.
2. Fork 1, 4, 16, and 32 attempts from the exact W0 commit.
3. Run deterministic SQL workers first. Have different attempts update the same primary keys to different values, and include a deliberately invalid outcome.
4. Commit each successful execution and evaluate it independently. Check nonnegative inventory, stock accounting, valid order references, and task completion. Reject the deliberately invalid result.
5. Verify exact expected contents for each branch and verify that W0 is unchanged. Counts alone cannot detect swaps or same-row contamination.
6. Retain one passing result, release the other temporary branches, run GC, and confirm the retained result is still readable. Start a fresh attempt from W0 and check the original contents again.
7. Exercise cancellation and failed SQL to verify cleanup and recorded failure status.

After this lifecycle works, replace the scripted worker with an agent using narrow SQL tools. Keep fixture generation, branch management, and evaluation unchanged. Record model configuration and tool traces; reproducing the database starting state does not imply identical stochastic agent behavior.

### Evidence to collect

Measure create, connect, execute, commit, verify, and release latency separately, plus end-to-end p50/p95 latency, completed attempts per second, peak memory, and disk growth before and after GC. Repeat batches to detect accumulation over time. Record hardware, Doltgres version, schema/fixture version, row counts, and concurrency.

Compare with separately initialized databases using the same schema, data, workload, and verification. Include seed loading in initialization costs and keep SQL behavior equivalent. This establishes whether branching is worthwhile for this workload, rather than presuming it is faster.

Correctness gates: no content contamination, unchanged baseline, invalid outcomes rejected, retained outcomes survive cleanup, and interrupted attempts leave no active workers or untracked temporary branches. Performance results determine the useful concurrency range; do not invent a speedup target before measuring.

## Minimal interface to extract

Keep the first implementation local to `dbenv`, with an explicit lifecycle:

```text
create_world(fixture, seed) -> world_commit
fork_attempt(world_commit, attempt_id) -> attempt_handle
run_attempt(attempt_handle, worker) -> trace
finalize_attempt(attempt_handle) -> result_commit
evaluate(world_commit, result_commit, task) -> checks + score
retain(result_commit) -> durable_reference
release(attempt_handle)
```

The attempt handle owns its connection scope and cleanup. Management operations belong to the controller. If agents receive raw SQL access, test whether their credentials can switch branches or mutate other state before claiming enforced isolation. Version the interface only after this experiment reveals the necessary lifecycle semantics.

Each result should record the world commit, fixture seed/version, task/version, attempt ID, worker or model configuration, trace location, final commit, durable reference if retained, verification results, timing, and failure status. Keep this manifest outside the disposable branch.

## Synthetic database direction

Once one world works, vary dataset size, order density, stock scarcity, and scenario difficulty. Generate valid starting states with deterministic rules and validate them before committing. Language generation can add realistic descriptions, but relational constraints and expected outcomes should remain independently checkable.

Keep held-out fixture seeds for evaluation. A branch creates another version of an existing world; diversity comes from the scenario generator. Store both the generation recipe and committed state so failed cases can be revisited.

The first deliverable is a repeatable experiment and a small lifecycle library. A training integration, a general database service, automatic merging, and a new dashboard can follow once the experiment establishes correctness and useful operating costs.
