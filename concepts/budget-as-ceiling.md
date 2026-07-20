---
kind: concept
name: "budget-as-ceiling"
status: active
added: "2026-04-24"
source_papers:
  - li2025fm
  - hambardzumyan2026aira
  - kamelhar2026gsar
  - khan2026token
sources:
  - "[[literature/papers/li2025fm]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/kamelhar2026gsar]]"
  - "[[literature/papers/xin2026eurekagent]]"
  - "[[literature/papers/jia2026finharness]]"
  - "[[literature/papers/zhao2026agenticos]]"
  - "[[literature/papers/khan2026token]]"
used_by:
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/typed-claim-partition]]"
related_experiments: []
tags: [budget, halting, resources, autonomy]
---

# budget-as-ceiling

## Definition

Autonomous chains halt because an explicit resource ceiling was hit
— wall-clock hours, token count, consecutive no-improvement
iterations, disk — not because a heuristic decided the work looked
done. The ceiling is a project-level commitment stored in
`budget.yaml` and read by every skill that spawns compute.

## Why it matters

FM Agent ([[literature/papers/li2025fm]]) operates a distributed
evolutionary search that could run indefinitely. Its ceiling
discipline is what makes the system practically runnable: explicit
wall-time and cost limits per experiment and per campaign,
distributed workers respect them, the campaign terminates when any
one is hit. AIRA_2 ([[literature/papers/hambardzumyan2026aira]])
reports 24h and 72h results separately because its async worker
pool has explicit wall ceilings that predict behavior reliably.

Without a ceiling, autonomous chains fail in two ways: they run
forever (the benign failure), or they blow through the user's real
budget without the user knowing (the costly one). The ceiling makes
both failure modes structural — if the budget is wrong, the user
edits one file; the skill does not need to learn a new halting
heuristic.

AgenticOS ([[literature/papers/zhao2026agenticos]]) generalizes the
enforcement locus: budgets there are not loop-halting conditions read by
skills but **admission/runtime primitives bound into capability tokens** —
network permissions are ⟨domain, path prefix, method, data type, budget⟩
five-tuples and Manifests declare per-agent inference budgets, so
exceeding a budget doesn't halt a loop, it invalidates the capability.
Design-only, but it shows the same ceiling discipline enforced below the
agent instead of inside it.

Until [[literature/papers/khan2026token]], every source above argued for
the ceiling from design experience. That paper supplies the empirical
base: 63 confirmed production budget-overrun incidents across 21
frameworks and 18 ecosystems (2023–2026), each backed by a quoted GitHub
issue and, where reported, a dollar loss (≈$2,150 for a single user in
the worst case), with an eight-cluster mechanism taxonomy (two-rater
κ = 0.837 on case labels). Two findings land directly on this concept.
First, the costly failure mode above is the documented norm, not a
hypothetical: framework fixes ship within days of a report, but the
catalog contains *no case in which an overrun was prevented before at
least one user paid* — every deployed mitigation was post-hoc, which is
the strongest argument yet that the ceiling should be enforced
**pre-flight** (refuse the call whose projected spend would exceed the
cap) rather than as a counter checked after the spend. Second, a whole
cluster (M-budget-primitive-missing, 12 of 110 rows across six
frameworks) documents frameworks that ship *no first-class budget
primitive at all* — the absence a project-level `budget.yaml` exists to
fill. The paper's three-layer enforcement taxonomy (compile-time type
system / software middleware / transport-layer 402) extends the
AgenticOS locus point: its affine-typed Rust `Budget` makes cloning,
double-spending, or use-after-delegating a budget compile errors, so the
delegation-fanout race it documents in 11 incidents cannot be written —
the same deterministic-substrate instinct as
[[concepts/permission-gate-as-architecture]]'s deterministic core,
applied to spend integrity. Its M-context-amplification cluster (13
incidents, including Claude Code's own compaction-loop bugs) is the
dollar-cost face of [[concepts/context-eviction-policy]].

## Implementation guidance

1. **`budget.yaml` at the project root.** Hard ceilings:
   ```yaml
   max_wall_hours: 24
   max_tokens: 5000000
   max_experiments: 20
   max_consecutive_no_improvement: 3
   max_disk_gb: 500
   ```
   Skills read this before spawning work; chains halt when any
   ceiling is hit.

2. **Per-skill responsibility.** `/iterate --chain` checks
   `max_consecutive_no_improvement` after each cycle. `/implement`
   checks `max_wall_hours` and `max_tokens` before starting.
   `/propose` checks `max_disk_gb` headroom if the proposal
   implies large downloads.

3. **Hitting a ceiling is information, not an error.** A chain that
   halts at `max_consecutive_no_improvement=3` is reporting "this
   search direction is exhausted," which feeds the next `/propose`
   call. The halt reason goes in the session's NOTES.md.
   [[literature/papers/zou2026fmlbench]] sharpens this: stagnation is
   first a *switch* signal, then a halt signal. Its AdaptiveSearch
   treats W consecutive non-improving steps as the trigger to fork the
   top-N candidates and broaden the search — outperforming both pure
   greedy and pure tree search — and only a stall that survives the
   broader phase is true exhaustion. A chain-runner that can afford it
   should escalate strategy at the stagnation counter before killing
   the chain.

4. **Raising ceilings is explicit.** A proposal that requires
   raising `max_wall_hours` beyond the current ceiling must say so
   in `risks:` and the user explicitly edits `budget.yaml` before
   running. No silent overrides.

## Open questions

- Token budgets are predictable; disk budgets are trickier because
  intermediate artifacts (checkpoints, cache) grow in bursts.
  Whether the ceiling should be enforced pre-allocation or
  post-hoc-with-cleanup is project-dependent.
- A `budget.yaml.git.auto_push: true` flag (set on this meta project)
  encodes a non-compute commitment — overnight work pushes without
  manual intervention. No skill reads it yet; the flag documents
  intent until a SessionEnd hook honors it.
