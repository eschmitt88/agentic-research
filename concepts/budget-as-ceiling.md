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
  - "[[literature/papers/ye2026agent]]"
  - "[[literature/papers/besanson2026green]]"
  - "[[literature/papers/hao2026selfgc]]"
  - "[[literature/papers/hamri2026zebra]]"
  - "[[literature/papers/bai2026how]]"
  - "[[literature/papers/mason2026missing]]"
  - "[[literature/papers/bhardwaj2026agent]]"
  - "[[literature/papers/ray2026what]]"
used_by:
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/typed-enforcement]]"
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

## A ceiling is really a ceiling-plus-one-call

[[literature/papers/ye2026agent]] (peer-reviewed, COINE 2026 @ AAMAS)
states the enforcement limit this concept has been implicitly assuming
away. Token consumption is *unknown during* an LLM call and known only
after it returns — even streaming APIs report usage metadata at
completion. Three consequences follow directly:

1. No budget mechanism can prevent a **single** call from exceeding the
   ceiling. It can only prevent the *next* one.
2. The value of a ceiling is therefore **multi-call** protection —
   runaway loops, retry storms, iterative-refinement cycles.
3. True pre-allocation would require provider-side capabilities that do
   not exist: interruptible generation with mid-stream cancellation,
   token reservation with guaranteed hard limits, budget-aware
   inference.

The paper's own data shows this honestly: a runaway agent under a 40K
budget was detected and halted at **56K consumed** — a 40% overshoot,
by design rather than by bug.

This **settles the pre-flight-reservation question** left open after
[[literature/papers/khan2026token]]. khan's affine-typed budget
ownership is a genuine in-process integrity property (no aliasing, no
double-spend, no use-after-delegation — all compile errors), but it
cannot bound a single call's actual cost either, because nothing can.
So halt-after-cycle enforcement is not a weaker approximation of
reservation; it is the correct achievable design. What follows for us:

- Keep hard enforcement **between** actions and soft enforcement
  (budget-aware prompting) within them — the paper's explicit split.
  Soft enforcement alone is defeated by "token elasticity," where models
  exceed stated budgets precisely when constraints are tight.
- Treat every ceiling in `budget.yaml` as ceiling-plus-one-worst-case-call,
  and size a **reserve buffer** (the paper uses 10–15%) to cover it. Our
  `budget.yaml` currently declares ceilings with no reserve, which makes
  the overshoot unbounded rather than merely nonzero.
- Enforce at the harness, not the model. Hard enforcement "requires no
  model-level support; it operates at the orchestration layer between
  actions" — the same structural-not-behavioral argument as
  [[concepts/permission-gate-as-architecture]].

## How much pre-flight value survives the impossibility

[[literature/papers/besanson2026green]] shows the achievable remainder
of the settled reservation question: a **predictive pre-action gate**
that admits the next call only if a learned cost forecast plus a
split-conformal margin fits the remaining budget. This does not evade
ye2026agent's limit — the call in flight is still unbounded — but it
turns "ceiling-plus-one-call" from a worst-case overshoot into a
calibrated risk: per-action breach probability ≤ δ, distribution-free
(the Normal-σ version under-covers on real right-skewed residuals,
92% at nominal 95%; conformal holds at 95.2%). Two results sharpen
this concept's argument directly: a soft Lagrangian penalty tuned to
meet the budget in expectation still breaches it on **91.5% of
seeds**, while the architectural gate breaches **0%** — the
quantitative case that a ceiling must be a gate, not a reward term —
and the State-Snowball theorem (naive full-context re-submission is
Θ(n²) in loop depth; real plans accrete *faster*) gives the growth
law behind khan2026token's context-amplification incidents. Practical
consequence for our `budget.yaml`: ye2026agent's flat 10–15% reserve
buffer has a principled replacement — size the reserve from the
forecast-residual quantile of the workload.

Fully enforceable this way: multi-call budgets, iteration limits,
API-call limits, duration limits. Only *approximable*: cost ceilings in
currency, which is why our token-denominated ceilings are the more
defensible unit.

**The counter-lever to State-Snowball has its own price.** Reducing the
active view is the direct way to fight Θ(n²) context growth, but
[[literature/papers/hao2026selfgc]] shows the reduction is not free:
committing a context edit invalidates part of the provider prefix cache,
so an eager eviction policy can spend more on cache misses than it saves
on prompt surface. Their commit rule prices this explicitly —
`CommitBenefit ≈ N_future(C − C′) − L_cache_break − L_GC`, where the last
term is the governance call itself — and holds any plan pruning less than
~0.3 of the active view until cache expiry or the next task boundary.
Deployed, that yielded 10–15% lower daytime input tokens (peaks ~20%),
though on covered traffic only and not as a matched billed-cost audit.
The transferable point is that **context reduction is a budgeted action
like any other**, with its own overhead term, not a pure saving — see
[[concepts/context-eviction-policy]] guidance 8.

## Budgets survive delegation — the conservation law

The same paper supplies the invariant this concept lacked for
sub-agents: Σ_j c_j ≤ B for every resource, holding across sequential,
parallel, hierarchical, and competitive execution, with subcontracts
constrained by Σ R_i ≤ R_parent. The consequence it names **bounded
autonomy** — however capable an orchestrator is, it may create
subcontracts but can never exceed its own parent constraint — is what
makes recursive delegation safe rather than merely convenient. Measured:
zero conservation violations across 50 multi-agent trials.

Two allocation details worth importing alongside it: budgets can be
assigned proportionally (by estimated complexity), equally, or
*negotiated* (agents request, coordinator caps to prevent
over-claiming); and unused budget from completed agents should return to
a shared pool, letting cheap agents subsidize expensive ones without
breaching the total. See [[concepts/hierarchical-delegation]].

The headline result is also a reframing of what ceilings buy: 90% token
reduction came with **525× lower variance** and a success-rate change of
−7.1 pp that was *not* statistically significant. Budgets are not a cost
optimization that happens to be safe; they are a variance-elimination
mechanism whose mean-quality effect is roughly neutral. Argue for them
on tail risk, not on average savings.

## The counter-pole: a ceiling says stop, not where to spend

[[literature/papers/hamri2026zebra]] names the axis this concept has
been sitting at one end of. Everything above governs *whether* spend
continues — halt conditions, enforcement locus, overshoot bounds.
ZEBRA governs *where* spend goes while the ceiling holds: an LLM
estimates a saturating utility curve per pipeline phase and an
explicit water-filling solver splits the budget as a continuous
knapsack. Under a binding budget the split dominates outcomes — 94.4%
vs 88.1% of unconstrained quality at half budget on APPS, with the
gap *growing* as the budget tightens, precisely the regime where a
ceiling-only design leaves quality on the table.

Three points transfer:

- **Allocation survives the reservation-impossibility.** ye2026agent
  settled that no mechanism can bound a single call's cost; ZEBRA
  needs no mid-call enforcement — per-phase caps are ordinary
  between-action ceilings the harness already knows how to enforce.
  Allocation is the achievable half of "spend planning."
- **LLM estimates, solver optimizes.** Handing the split itself to
  an LLM loses 4+ points *given the same curves* (LLMs demonstrably
  fail at knapsack); the arithmetic belongs in code, the judgment
  (curve elicitation) in the model — the
  [[concepts/scripted-tool-pipelines]] division applied to budgeting.
- **Allocation only matters when the budget binds.** Easy tasks tie
  under any policy; the win concentrates on hard tasks under tight α.
  A project whose ceilings never bind (this one, mostly) gains
  nothing from an allocator; a downstream chain that routinely halts
  at `max_tokens` would.

Caveats for import: ~33% controller overhead at tight budgets,
one-shot pre-execution allocation (their mid-pipeline reallocation
variant didn't help), and a hard scope limit — the method presupposes
discrete phases, so *phaseless agentic loops* (our dominant shape)
are explicitly out of scope. The coordinator's suggested session
budget is the single-phase degenerate case; ZEBRA is the shape a
multi-phase version would take.

## Whose estimate the gate may use

besanson2026green's predictive pre-action gate needs a cost forecast, and
[[literature/papers/bai2026how]] settles where that forecast may *not*
come from: the agent itself. Eight frontier models given full tool
access, permission to inspect the repository, and an explicit instruction
to estimate rather than fix reach Pearson r ≤ 0.39 against their own
actual token usage, and **every one underestimates**, worst on input
tokens — which are the dominant term (input/output ratio 153.85 in
agentic coding, ~$1.86 and 4.17M tokens per SWE-bench-Verified task
against $0.023 for multi-turn chat). A self-estimated reservation
therefore fails in the admitting direction. This does not weaken the
gate; it relocates the estimator outside the agent and makes the
one-sided conformal margin mandatory rather than tidy. The full argument,
including why forecasting is a different problem from
[[concepts/context-proprioception]], lives in
[[concepts/spend-forecast-calibration]].

Two further results from the same paper bear on ceilings directly. The
inverse-scaling finding — accuracy peaks at intermediate cost and then
saturates, with high-cost runs showing sharply more *repeated* file views
and edits — means an expensive run is weak evidence of a hard problem and
decent evidence of an unproductive loop, which is the empirical case for
`max_consecutive_no_improvement` as a first-class ceiling rather than a
nicety. And the success→failure cost increase is model-specific (<0.5M
extra tokens for GPT-5/5.2, ~2M for Kimi-K2) because models lack a
reliable mechanism to recognize an unsolvable task and stop: the ceiling
substitutes for a missing capability, and how much it must substitute
depends on the backend.

## Context churn is a spend line, and it can exhaust the rate limit first

[[literature/papers/mason2026missing]] supplies a ceiling failure this
concept has not accounted for. In a 681-turn production session its pager
thrashed — 680 evictions, 659 page faults — and the session died on the
**API rate limit, not the context limit**. Each fault is an inference-priced
round trip, so "fault cost is not merely computational but monetary" and
thrashing consumed the rate budget faster than useful work. Two things follow.
A ceiling denominated only in total tokens misses a *rate* ceiling that can
bind first, which matters for any parallel design ([[concepts/async-worker-pool]]).
And an eviction policy is a spend policy: churn that looks like savings on the
token-surface metric can be net negative on the bill — the paper's own
"bytes saved" number was partly self-inflicted re-reads, which it says
outright. See [[concepts/context-eviction-policy]].

## Scope: a ceiling bounds consumption, not conduct

Worth stating explicitly, because the two get conflated.
[[literature/papers/bhardwaj2026agent]] draws the line cleanly against
ye2026agent, which this concept already leans on: resource contracts govern
*how much* an agent may consume — tokens, time, cost, delegation hierarchies
with conservation laws — while behavioral contracts govern *how it must
behave* — preconditions, invariants, drift bounds, recovery. "The two
frameworks address orthogonal concerns and could be composed: resource
contracts bounding computation, behavioral contracts bounding behavior."

So a `budget.yaml` ceiling is not a governance mechanism and should not be
argued as one. It cannot prevent an agent from doing the wrong thing cheaply,
and the fact that a chain halted at `max_tokens` says nothing about whether
its work was correct. Behavioral constraints belong to
[[concepts/typed-enforcement]] and [[concepts/permission-gate-as-architecture]];
this concept's claim is narrower and stronger for staying narrow.

## Monotone counters stay analyzable; resettable ones do not

[[literature/papers/ray2026what]]'s decidability result is a design rule for
this concept's central artifact, and it cuts `budget.yaml` in two.

Asking whether a policy is even **nontrivial** — whether it forbids anything
at all — is **undecidable** for a checker with two decrementable zero-test
counters, but **PSPACE** for the separable, key-local, constant-guard
**monotone** fragment. Monotone means the counter only ever goes one way.
The paper notes this fragment is exactly what deployed caps, quotas, and
rate limits use, and that it is the deliberate middle ground: plain
registers cannot count at all, unrestricted counters are undecidable.

Sorting this repo's ceilings by that line:

- `max_tokens`, `max_wall_hours`, `max_experiments`, `max_disk_gb` are
  **monotone accumulators against a constant bound** — the tractable
  fragment. Whether a set of them is contradictory, redundant, or vacuous is
  a decidable question, which is what makes the static-analysis proposal in
  [[concepts/typed-enforcement]] worth attempting on this file.
- `max_consecutive_no_improvement` is **not** monotone. It increments on a
  bad cycle and **resets to zero on an improvement** — a decrementable
  counter with a zero test, which is the construct T3 shows tips
  nontriviality into undecidability. It is also the ceiling most exposed to
  gaming, since the agent's own notion of "improvement" is what resets it.

The practical reading is not that the reset ceiling is wrong — it encodes
something the monotone ones cannot, namely *stop when progress stops*. It is
that it belongs to a different tier: the monotone ceilings can be checked
for consistency ahead of time and the reset one cannot, so it needs an
independent monotone backstop (a hard cycle count) rather than being trusted
alone. That backstop does not exist in `budget.yaml` today.

Two smaller notes from the same source. A ceiling is a **safety** property
in the formal sense — "spend never exceeds c" has a bad prefix the moment it
is crossed — which is why it sits inside the gate-enforceable class at all,
unlike anything phrased as "eventually finish." And the paper's exact
cap-saturation argument means a per-key cap stays analyzable even with
unboundedly many keys active, because nontriviality only ever needs to track
a single witness key: per-project or per-session budgets do not cost
tractability.

## Open questions

- Token budgets are predictable; disk budgets are trickier because
  intermediate artifacts (checkpoints, cache) grow in bursts.
  Whether the ceiling should be enforced pre-allocation or
  post-hoc-with-cleanup is project-dependent.
- A `budget.yaml.git.auto_push: true` flag (set on this meta project)
  encodes a non-compute commitment — overnight work pushes without
  manual intervention. No skill reads it yet; the flag documents
  intent until a SessionEnd hook honors it.
