---
kind: moc
name: "compute-economy"
status: active
added: "2026-08-17"
concepts:
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/context-proprioception]]"
tags: [moc, budget, spend-policy, compute, allocation, forecasting, cost-accounting, resources]
---

# Compute Economy

An autonomous agent spends real money and real wall-clock on every step,
and the spend is not a side effect of the work — it is a decision surface
with its own failure modes. This MoC collects the six concepts that answer
three questions in order: **how does the agent know what a plan will
cost, how should it place that spend, and what stops it when the budget is
gone?**

The theme was declined as a MoC candidate five times between 2026-07-07 and
2026-08-17, always for the same stated reason: the cluster had four solid
members plus [[concepts/async-worker-pool]] by judgment, and a spend MoC
would have been a third framing of [[concepts/budget-as-ceiling]] —
whose halting and parallelism economics [[mocs/autonomous-search-loop]]
already covers, and whose bindingness [[mocs/governance-by-architecture]]
already covers. The declines named a precise ripening condition: *one
genuine spend-policy concept, from the khan2026token pre-flight-reservation
thread or the besanson2026green energy-cost line — both still sources on
other concepts with nothing seeded of their own.*
[[concepts/spend-forecast-calibration]] (2026-08-17) is that concept, and
it arrived from both threads at once —
[[literature/papers/bai2026how]] is the empirical test of pre-flight
self-reservation, [[literature/papers/panigrahy2026energy]] the mechanism
behind the energy-cost line.

What that new member changes is the framing, not just the count. Every
prior view of this material asked *whether spend continues*. The
measurement question — where the projected number comes from, and whose
estimate a gate may act on — is not stated anywhere else in the graph, and
it turns out to be the axis with the strongest empirical results behind it.
Max overlap with any existing MoC is 2 of 6.

## Know the number

Measurement comes first, because everything downstream acts on a figure
that has to be trustworthy.

- [[concepts/spend-forecast-calibration]] — the gate needs a number, and
  it may not be the agent's own. Eight frontier models given full tool
  access, permission to inspect the repository, and an instruction to
  estimate rather than act reach Pearson r ≤ 0.39 against their own actual
  token usage, and **every one underestimates**, worst on input tokens —
  the dominant cost term ([[literature/papers/bai2026how]]). So a
  self-estimated reservation fails in the admitting direction. The
  constructive half is external and calibrated: a learned forecast plus a
  split-conformal margin bounds per-action breach probability
  distribution-free, where a Normal-σ interval under-covers on real
  right-skewed residuals ([[literature/papers/besanson2026green]]). And the
  denominator matters as much as the estimator — normalize by *completed
  goals*, not by calls, or retries vanish from the accounting
  ([[literature/papers/panigrahy2026energy]], where a failed attempt drew
  2,256 J against 1,358 J for the success on the same goal).
- [[concepts/context-proprioception]] — the agent's perception of its own
  *present* resource state: per-block token cost, recency, archive status,
  remaining headroom, surfaced as a machine-maintained ledger rather than
  inferred from prompt text ([[literature/papers/xu2026llm]]), and deployed
  in production as graduated pressure zones that inject fill percentage and
  the largest resident blocks while ~40K tokens of runway remain
  ([[literature/papers/mason2026missing]]). It belongs here because
  spending well requires knowing what you are holding — and it sits
  deliberately *beside* forecasting rather than inside it: present state is
  an interface problem that plumbing fixes, future cost is not.

## Spend it well

Under a fixed budget, placement dominates outcomes — and the win
concentrates exactly where the budget binds.

- [[concepts/hybrid-model-backends]] — different roles, different models.
  Token efficiency turns out to be a property of the *model*, not the
  task: relative usage holds its ranking across both the
  shared-success and shared-failure subsets, and models diverge most when
  they are losing, because they lack a reliable mechanism to recognize an
  unsolvable task and stop ([[literature/papers/bai2026how]]). A backend
  that overspends specifically when failing is a poor fit for any
  unsupervised chain.
- [[concepts/scripted-tool-pipelines]] — move the arithmetic out of the
  model. The division of labor (judgment in the model, computation in
  code) is a cost argument as well as a correctness one: handing the
  budget *split* itself to an LLM loses 4+ points given the same elicited
  curves ([[literature/papers/hamri2026zebra]]), and orchestration
  overhead inverts below 1.0× on tool-augmented tasks — agentic dispatch
  replacing token generation is strictly cheaper
  ([[literature/papers/panigrahy2026energy]]).
- [[concepts/async-worker-pool]] — throughput per wall-hour, the only
  lever that buys more search inside a fixed time ceiling
  ([[literature/papers/hambardzumyan2026aira]]). Two cautions from this
  cluster: wall-clock-based cost attribution mis-prices a design whose
  point is overlapping waits (measured power drops to 1.0 W during API
  wait against 0.2 W during active planning), and a *rate* ceiling can
  bind before a token ceiling — one production session died on the API
  rate limit while thrashing, not on context exhaustion
  ([[literature/papers/mason2026missing]]).

## Stop when it's gone

- [[concepts/budget-as-ceiling]] — explicit ceilings in `budget.yaml`,
  read by every skill that spawns compute, with halting as information
  rather than error. The concept's own hard-won limits are what keep this
  MoC honest: no mechanism can bound a *single* call, so a ceiling is
  really ceiling-plus-one-worst-case-call and needs a sized reserve
  ([[literature/papers/ye2026agent]]); a soft budget penalty tuned to meet
  the budget in expectation still breaches it on 91.5% of seeds while an
  architectural gate breaches 0%
  ([[literature/papers/besanson2026green]]); budgets survive delegation by
  conservation law, Σ c_j ≤ B, which is what makes recursive delegation
  safe; and the honest headline is variance, not savings — 90% token
  reduction came with **525× lower variance** and a statistically
  insignificant quality change. Argue ceilings on tail risk.
- **Scope discipline.** A ceiling bounds *consumption*, not *conduct*. It
  cannot prevent an agent doing the wrong thing cheaply, and a chain that
  halted at `max_tokens` has said nothing about whether its work was
  correct ([[literature/papers/bhardwaj2026agent]]'s orthogonality between
  resource contracts and behavioral contracts). Behavioral constraints
  live in [[mocs/governance-by-architecture]]; this MoC stays narrower on
  purpose.

## Relation to neighbouring maps

Three members appear elsewhere and the split is deliberate.
[[mocs/autonomous-search-loop]] holds `budget-as-ceiling` and
`async-worker-pool` as *search* machinery — how a loop halts and how it
parallelizes; here they are spend machinery.
[[mocs/capability-layer]] and [[mocs/agent-architecture]] hold
`hybrid-model-backends` and `scripted-tool-pipelines` as *action-surface*
components; here they are allocation choices.
[[mocs/runtime-memory-policy]] holds `context-proprioception` as a memory
interface; here it is the agent's cost sensor. The organizing question —
know / place / stop — is what this view adds.

## Open thread

**Nothing in this repo forecasts anything yet.** The cluster's strongest
new claim is that budget policy should act on a calibrated external
forecast, and the ingredients are already on this box —
`~/.claude/state.db` holds measured per-session usage and the coordinator's
agency CLI already reports weekly consumption — but no forecaster exists,
so `spend-forecast-calibration` is currently a constraint on what *not* to
build (self-estimated reservations) rather than a component to import.

Two related gaps. Ceilings on this project rarely bind, and the allocation
literature is explicit that the win concentrates when they do
([[literature/papers/hamri2026zebra]]: 94.4% vs 88.1% of unconstrained
quality at half budget, gap *growing* as budget tightens) — so an
allocator here would buy little until a downstream chain routinely halts at
`max_tokens`. And the coupling this cluster has only started to trace is
between spend and context: cache reads dominate billed cost in every phase
of an agentic trajectory ([[literature/papers/bai2026how]]), so an eviction
policy is a spend policy, and churn that looks like savings on the
token-surface metric can be net negative on the bill
([[literature/papers/mason2026missing]]). Whether that deserves its own
concept or stays a cross-link between [[concepts/context-eviction-policy]]
and this map is the next thing to test.
