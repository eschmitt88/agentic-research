---
kind: concept
name: "async-worker-pool"
status: experimental
added: "2026-04-24"
source_papers:
  - hambardzumyan2026aira
  - li2025fm
sources:
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/li2025fm]]"
used_by: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/budget-as-ceiling]]"
related_experiments: []
tags: [infrastructure, async, parallelism, distributed]
---

# async-worker-pool

## Definition

Autonomous experiment execution runs on a pool of asynchronous
workers sharing a job queue. Workers claim experiments, run them,
report metrics, and free themselves. The orchestrator proposes new
experiments faster than any single worker can execute them;
parallelism comes from the pool, not from a single worker running
faster.

## Why it matters

AIRA_2 ([[literature/papers/hambardzumyan2026aira]]) names
synchronous execution as one of three dominant bottlenecks in AI
research agents. At steady state, a single-worker loop is GPU-idle
most of the time (proposing, analyzing, writing) and GPU-bound only
during training; an async pool keeps the hardware utilized by
overlapping proposals with training. The ablation shows this is
material, not marginal — comparable to the HCE fix.

FM Agent ([[literature/papers/li2025fm]]) runs distributed workers
across its evolutionary sampling loop, and its MLE-bench headline
(43.56%) depends on population sizes that would be infeasible on a
single synchronous worker.

## Implementation guidance

1. **Shared job queue.** Workers pull experiments from a queue
   rather than being assigned round-robin. Fast experiments unblock
   the queue; slow experiments run to completion without blocking
   the next-in-line.

2. **Metric reporting is write-once per experiment.** Workers write
   `metrics.json` atomically when the experiment completes. The
   orchestrator polls or subscribes; no worker waits on another
   worker.

3. **Budget ceilings are shared.** A pool-wide
   `max_consecutive_no_improvement` counter needs a single owner —
   usually the orchestrator, with workers reporting their own
   terminal results. Per-worker ceilings (wall hours) are independent.

4. **HCE separation holds per-worker.** Workers do not coordinate
   through the test split; each only writes validation results
   during search. The final-scoring pass runs after the pool halts.

5. **Hardware sizing.** An at-home box with one GPU gets the
   easiest win: the async-pool is really about overlapping the
   "propose / analyze / write" phases with GPU time, not about
   multi-GPU training itself. Benefit remains even at pool size 2.

## Open questions

- At-home single-GPU setups benefit less than multi-GPU labs;
  whether the complexity is worth it below 4 GPUs is unclear.
- Queue starvation modes (short experiments starving long ones, or
  vice versa) are not addressed in either source paper. A priority
  scheme likely emerges as needed.
- Status is `experimental` because no current skill implements this
  pool — `/implement` runs one subagent at a time. A downstream
  project that adds an async worker harness moves it to `active`.
