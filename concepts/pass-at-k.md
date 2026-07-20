---
kind: concept
name: "pass-at-k"
status: experimental
added: "2026-04-24"
source_papers:
  - hambardzumyan2026aira
  - chan2024mle
  - starace2025paperbench
sources:
  - "[[literature/papers/edwards2025rexbench]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/chan2024mle]]"
  - "[[literature/papers/starace2025paperbench]]"
  - "[[literature/papers/jain2026agentic]]"
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/lupidi2026airsbench]]"
used_by:
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/hce-evaluation]]"
related_experiments: []
tags: [evaluation, variance, seeds, reproducibility]
---

# pass-at-k

## Definition

Seed variance is a first-class axis of experimental design: any
reported result is a distribution over k seeds, not a single number.
A run with k=1 is a pilot; a claim requires k≥3 and a statement of
the seed distribution, not just its mean.

## Why it matters

MLE-bench ([[literature/papers/chan2024mle]]) and AIRA_2
([[literature/papers/hambardzumyan2026aira]]) both surface that
headline agent performance is seed-sensitive at a magnitude that
rivals architectural changes. A 2-point medal-rate difference
between two agent architectures can vanish when both are run at
k=5; conversely, a true architectural improvement can be invisible
at k=1 if the seed happens to fall in the wrong mode.

The AIRA_2 scaling-laws analysis explicitly treats seed distribution
as signal: predictable scaling across LLM variants emerges only when
each configuration is run at k≥3.

## Implementation guidance

1. **Metrics files report distributions.** Instead of
   `{"val_acc": 0.84}`, report
   `{"val_acc": {"mean": 0.84, "std": 0.02, "seeds": [42, 43, 44, 45, 46], "values": [...]}}`.
   Skills that rank experiments use the mean; skills that decide
   whether two runs differ must inspect the distribution.

2. **Default k.** For cheap experiments (minutes on CPU), k=5 is a
   reasonable floor. For expensive experiments, k=3 with an
   explicit note in the Diagnostics section. k=1 is allowed only
   for pilots — mark `status: pilot` on the experiment frontmatter.

3. **Seeds are recorded, not re-derived.** Each experiment's
   `config.yaml` lists the seeds used. Re-running does not
   regenerate the seed list; it produces a new run at the same seeds.

4. **Comparison requires overlap in seed distribution.** A vs B is
   comparable only if they ran at matched seed lists. If not, the
   comparison is "A at seeds S1 vs B at seeds S2" and must say so.

## Open questions

- The right k for MLE-bench-scale tasks is unsettled. FM Agent
  ([[literature/papers/li2025fm]]) and AIBuildAI
  ([[literature/papers/zhang2026aibuildai]]) do not specify seed
  protocols explicitly; their medal rates may not be directly
  comparable. NatureBench
  ([[literature/papers/wang2026naturebench]]) runs each of its twelve
  agent configurations exactly once over 90 tasks, so its 17.8%
  Surpass-SOTA headline is a k=1 estimate with no run-to-run variance
  treatment — its cross-model reproduce-mode agreement calibrates the
  SOTA anchors, not the variance of the agents being ranked. Expensive
  4-hour-per-task benchmarks are exactly where the k=1 temptation is
  strongest and where this concept's k≥3 floor remains unmet in
  practice. *Counterexample now attested:* AIRS-Bench
  ([[literature/papers/lupidi2026airsbench]]) runs every agent-task
  pair at ≥10 seeds despite 24h/H-200-per-run cost, reports 95% CIs on
  all headline metrics, folds failed/invalid runs in as normalized-0
  rather than dropping them, and aggregates rankings with an
  order-invariant Bradley–Terry Elo — the first expensive benchmark in
  this graph to exceed the k≥3 floor as protocol rather than
  aspiration.
- `status: experimental` here because this project has no tooling
  that enforces pass@k reporting yet. It is a recommendation backed
  by literature, not an enforced rule. Graduate to `active` when a
  downstream project successfully runs the protocol end-to-end.
