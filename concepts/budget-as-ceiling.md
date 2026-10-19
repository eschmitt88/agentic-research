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
  - "[[literature/papers/zhang2026agora]]"
  - "[[literature/papers/yu2026primescientist]]"
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
  - "[[literature/papers/roth2026hack]]"
  - "[[literature/papers/nakayashiki2026when]]"
  - "[[literature/papers/moukpe2026deltaml]]"
  - "[[literature/papers/chi2026ai4ai]]"
  - "[[literature/papers/li2026praxist]]"
  - "[[literature/papers/ge2026coverage]]"
  - "[[literature/papers/hickey2026saltbench]]"
  - "[[literature/papers/chandran2026autoresearch]]"
  - "[[literature/papers/min2026autonomous]]"
  - "[[literature/papers/qu2026propose]]"
  - "[[literature/papers/hu2026analyzing]]"
  - "[[literature/papers/srikanth2026recursive]]"
  - "[[literature/papers/xia2026rrsi]]"
  - "[[literature/papers/park2026when]]"
  - "[[literature/papers/wiedmann2026agents]]"
  - "[[literature/papers/bobadillasuarez2026audit]]"
  - "[[literature/papers/yan2026traceml]]"
  - "[[literature/papers/wang2026pear]]"
  - "[[literature/papers/wang2026research]]"
  - "[[literature/papers/xu2026conflictguide]]"
  - "[[literature/papers/samanta2026learning]]"
  - "[[literature/papers/woo2026youra]]"
  - "[[literature/papers/cai2026forkandflush]]"
  - "[[literature/papers/hu2026winners]]"
  - "[[literature/papers/zhang2026judged]]"
  - "[[literature/papers/li2026humanize]]"
  - "[[literature/papers/du2026can]]"
  - "[[literature/papers/he2026beneath]]"
  - "[[literature/papers/li2026beyond]]"
  - "[[literature/papers/ning2026implementation]]"
  - "[[literature/papers/wang2026clock]]"
  - "[[literature/papers/wang2026language]]"
  - "[[literature/papers/wei2026scale]]"
  - "[[literature/papers/ding2026harnessdelta]]"
  - "[[literature/papers/rodionov2026improving]]"
  - "[[literature/papers/sun2026scipivot]]"
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

There is a third failure, in the opposite direction: the agent halts
itself too early. [[literature/papers/min2026autonomous]] reports that every
research agent it ran (Sonnet 5, Composer 2.5, GPT-OSS 120B) "often went
idle or stopped prematurely despite explicit guidelines to 'never stop'".
Python heartbeat controllers were no more reliable. What worked was a
deterministic outer bash `while` loop that resumes the agent session each
iteration. So the definition's rule extends to *continuation*: whether the
chain keeps going is not the agent's call either, in either direction.
Their loop had continuation but no ceiling; a human ended every campaign.
The longest campaign (132 experiments) finished below three campaigns of
9–17 experiments, which is the unbounded shape a
`max_consecutive_no_improvement` ceiling exists to cut. It is confounded
with team structure, and no per-experiment trajectory is reported.

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
   That reading holds for the no-improvement counter, not for a token or
   call cap. [[literature/papers/hickey2026saltbench]] shows a cap reading
   under a verifier carries no distance information: a Sonnet 5 episode
   reported 0 obligations verified at a 40-call cap and held a complete
   verified proof at call 79 once the cap was raised. A chain halted at
   `max_tokens` has produced an *unresolved* result, not a negative one,
   and the NOTES.md halt reason should say so.

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

Defensible, but not tier-neutral. hickey2026saltbench found the
subscription quota cost per token roughly double at the higher model tier
while the token count fell, so "a campaign that priced the tier on tokens
alone would have called it free." A ceiling has to name its unit, and a
token ceiling spanning a model switch is a different budget per tier.

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

### Allocation without telling the model, and the budget below which it loses

[[literature/papers/yu2026primescientist]] extends the counter-pole above in
three ways worth recording.

**Budget-awareness need not be a prompt.** The budget enters only as
`α_t = min(1/r_t, α_max)` with `r_t = (B − U_t)/B`, computed in the harness:
"the executor receives the plan without instructions about the remaining
global budget," and the reflector's system prompt contains no budget token.
This is [[concepts/scripted-tool-pipelines]]' split in a stronger form — the
model is not merely given the arithmetic, it is never told the budget exists
— which sidesteps `ye2026agent`'s token-elasticity problem by construction
rather than by instruction.

**Allocation has a measured overhead and a two-sided break-even.** Planning
is **38%** of the shared budget and one expansion costs 0.1–0.3M tokens
against a 1–1.5M campaign. Below roughly **750k tokens the allocator loses**
outright (0.42 against 0.52 at 250k; 0.47 against 0.51 at 500k on Concurrent
KV WAL). That sharpens "allocation only matters when the budget binds" into a
condition with a floor as well as a ceiling: under a small budget, the
machinery costs more than it saves.

**An independent restatement of ceiling-plus-one-call.** Its Definition 1:
actions are charged on completion and no further action starts once the
budget is reached, so "the final action may therefore cross the threshold."

Discount the headline before reusing any of this. "+10.3% average reward with
50.6% fewer attempts" is one search per cell on **FIRE-Bench, a benchmark six
of the eight authors wrote**, with no conflict statement. Dropping a single
task takes the quality gap to +3.7%; the only multi-seed head-to-head is
reward *parity* (0.4363 vs 0.4359) with a 24% attempt reduction; and at
matched tokens the baseline is marginally ahead on the mean. **What survives
is sample efficiency of roughly 24–29% fewer complete attempts, not research
quality.**

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

## The halt-on-no-progress ceiling is a safety control, not only a cost control

`max_consecutive_no_improvement` is framed in this project as a spend
guard — stop paying for a search that has stopped paying out.
[[literature/papers/roth2026hack]] supplies a second and stronger reason to
keep it.

Its cleanest result is that **hack rate rises monotonically with task
difficulty**, measured *within* a task by turning one difficulty knob
(fewer Wordle guesses, more Hanoi disks, more 15-Puzzle shuffles) rather
than across tasks, so the gradient is not confounded by task identity. An
autonomous loop that keeps running *because it is not improving* is, by
construction, sitting in the regime where the task has proven hard —
precisely where exploitation propensity is highest.

Compounding it: with persistent context across attempts, hacking is
**emergent and addictive**. Models need several attempts before discovering
an exploit, and once one hacks it almost always hacks again. Both halves
point the same way — the longer a stalled loop runs on accumulated context,
the more likely it is to find and then repeat a shortcut.

So the ceiling that halts a stalled chain is buying two things: it caps
spend, and it caps the exposure window in which a frustrated agent
discovers that the metric can be moved without doing the work. That is a
better argument for the ceiling than cost alone, and it also argues the
halt should **discard the context** rather than resume from it — see
[[concepts/context-eviction-policy]].

Note the tension with the shape argument above: this is the ceiling that is
*not* a monotone counter, and therefore the one that is not statically
analyzable. It earns its place on behavioral grounds while remaining the
weakest link formally, which is the case for pairing it with a monotone
hard cycle cap rather than trusting it alone.

**A no-improvement ceiling needs a noise floor, or it will never fire.**
[[literature/papers/zhang2026agora]] runs the case where no ceiling existed
and the fitness comparison had no tolerance. Thirteen workers spent five days
refining one recipe by 1e-5 bits-per-byte per step, each reading the same
leaderboard. The paper's own measurement noise is two orders of magnitude
larger — cross-hardware reproductions of the same code differ by up to
1.3e-3 bpb — and the authors flag the endpoint: the last recorded change
"moved the score by 9 x 10-6 , below cross-hardware variation," so the number
to trust is "the improvement from 3.39 to about 1.90 rather than the final
decimal places." Because the leaderboard compared with a bare `>`, every
sub-noise delta registered as a new best and reset the stall clock that a
`max_consecutive_no_improvement` guard would have been counting.

The design rule: **`max_consecutive_no_improvement` must compare against a
measured run-to-run spread, not against zero.** A "new best" inside the
evaluator's own reproduction tolerance is a coin flip, and a loop that treats
it as progress is formally incapable of reaching its own halt condition. The
cheapest instrument is the one that paper already had and did not use for
this purpose: its reproduction verdicts give an empirical noise band for
free. This is the counterpart of the hickey2026saltbench result — there a cap
corrupts the number beside it; here an *absent* floor lets the number disable
the cap.

The scale of what the stall cost: 18 scored contributions "account for about
98% of the total reduction," leaving 1,106 further scored results to find the
next 0.03.

**This repo is exposed.** `budget.yaml` sets `max_consecutive_no_improvement:
3` with no tolerance term anywhere, so the same bare-`>` failure applies to
any `/iterate` loop scored on a noisy metric.

**A second system reaches for the tolerance band and also fails to calibrate
it.** [[literature/papers/yu2026primescientist]] prunes a node when
`ρ < Q(parent(s)) − δ` with `δ = 0.05` — the right *shape*, a band rather
than a bare `>`. But `δ` is asserted in "Implementation details" with no
derivation, no ablation, and no relation to its evaluator's reproduction
spread, which matters because its reward is a noisy LLM-judged F1 and the
pipeline stacks four max-over-noise operators (FIRE-Bench takes the max of
two executions per node, `Q(v)` averages only *post-pruning* retained
descendants so pruning biases the value later selections condition on, and
the output is `argmax h(s)`). None is quantified.

That system also has **no stopping rule at all**: Algorithm 1 line 3 is
`while U < B`, and the `stop` action in its own Definition 1 is never
implemented — no stagnation counter, no no-improvement ceiling, no
convergence test. So it is a second instance of the `zhang2026agora` failure,
not a fix for it.

The bearing on this repo's open question is therefore sharpened but not
answered: **two independent systems now agree the comparison needs a
tolerance term, and neither supplies a principled way to set one.** The
derivation still has to come from the evaluator's own reproduction spread.
It does raise the priority — an uncalibrated band is at least analyzable,
where a bare `>` is not.

**A production loop puts a band on both comparisons, still uncalibrated,
and wires the stall counter to redirect rather than halt (2026-09-28).**
[[literature/papers/chandran2026autoresearch]] ran Karpathy-style
AutoResearch for twelve weeks on two Amazon recsys systems. Its Algorithm 1
applies a tolerance at two points:

- **Keep.** A candidate is kept `if score > best_score · (1 + τ_keep)`, a
  relative band on the champion comparison. The value of τ_keep is never
  reported.
- **Stall.** When `Δrel(best score)` over the last 5 iterations falls below
  `τ_stag` = 8%, the counter fires.

The stall parameters were tuned by hand against false alarms: "3 iterations
was too frequent (firing on noise); 8 iterations was too late." There are
no seeds and no repeated runs. The only spread the paper measures comes
after the fact: the final model re-scored on 11 dates gives ±0.28 pp. That
spread never feeds back into either threshold.

So this is a third system that reaches for the band and does not derive it.
It is also field evidence for the premise. Stagnation looked like five
consecutive iterations "all within ±0.008 weighted coherence", and a window
of 3 tripped on noise.

Two things are new here:

- **Stagnation redirects instead of halting.** Firing the counter calls a
  Criticizer, which emits one strategic directive that clears on the next
  keep. Algorithm 1 has no halt branch at all (`for i = 1, 2, …`). The largest
  single-directive gain followed one such firing (+50 bps Recall@6 in three
  iterations). This field-confirms guidance item 3's switch-then-halt
  reading (zou2026fmlbench).
- **The band needs to be defined per objective.** System B had several
  competing criteria that were "never simultaneously met in any of 15 runs".
  There, "improved" had no single definition at all, and humans set the
  revert thresholds.

A scalar noise band presupposes a scalar metric. A multi-criteria `/iterate`
chain would need the band specified per criterion, or a declared primary.

**An anytime-valid test answers the champion question, not the halt
question — and at `/iterate`'s sample sizes it certifies nothing
(2026-09-28).** [[literature/papers/qu2026propose]] is the first principled
alternative to a band in this graph. It uses a frozen betting referee
(e-processes → online e-BH) that scores each candidate only on outcomes
revealed *after* it was submitted. So FDR over everything ever admitted
holds at every stopping time, for any proposer with no foreknowledge of
those outcomes. Its measured penalty for the practice a bare `>` or a
re-checked band amounts to is large. The *peeking* arm (a rolling t-test,
admit at the first p ≤ α) admits 86.2 realised false factors per campaign,
against the frozen referee's 11.7.

It does not transfer as "the tolerance term", for four reasons:

- **Different object.** The guarantee controls false *admissions*, which
  in `/iterate` terms is the honesty of "new best" claims. A halt is not an
  inference and needs no validity.
- **It needs a stream, not a number.** Capital is `Π(1 + λX)` with λ < 1,
  on a bounded metric, so one run can at most double it. It cannot reach
  1/α = 20. The paper's wait formula, `ln(1/α)·2σ²/µ²`, gives about 6(σ/µ)²
  fresh runs per certified improvement at α = 0.05, or about 12(σ/µ)² under
  a 20-experiment e-BH bar. That is roughly 6 runs for a 1σ gain and 24 for
  0.5σ. (This is our arithmetic, not the paper's.)
- **Post-submission only.** The run that made a candidate look best is
  inadmissible evidence for it. Certification needs *new* seeds after the
  decision, which at least doubles the cost.
- **The price is latency.** In the paper it is about 500 trading days per
  admitted true factor. In `/iterate` it is paid in runs against
  `max_experiments`.

With `seeds_run: 1` an e-gated counter never resets, and the ceiling
collapses into a hard 3-cycle cap. Two things survive the transfer. One is
the rule *don't add seeds until the band clears*, because that is peeking.
The other is a sequential e-test on paired seed deltas, as the upgrade if
chains ever run many cheap seeds. The paper's own comparator also backs the
09-27 proposal's rejected heavy form. A patient fixed-horizon t-test with BH
comes close on false admissions (16.4 against 11.7). So fixed `--seeds N`
plus a real test buys most of it. The std band remains the right cheap
edit.

**The band, derived rather than asserted (2026-09-29).**
[[literature/papers/hu2026analyzing]] sets the tolerance from measured spread
and states the estimator and decision rule in full. It is an evaluation
study of coding agents, not a loop, but the protocol transfers directly:

1. **Replicate the baseline only.** They ran each of 8 baselines 3× and took
   s as the SD of Pass@1 and the CV of cost. Triplicating every cell would
   have cost about $7,500.
2. **Derive the threshold.** A change is real when
   `|Δ| / (s·√(1/R + 1/3)) ≥ 1.645`, i.e. a replication would reproduce its
   sign with probability 95%. For a single-run candidate that is
   **|Δ| ≥ 1.90 s**.
3. **Check that the floor transfers.** On 9 replicated treatment cells, cost
   SD was 0.28–1.38× the baseline's in 8 of them. The one exception
   (2.39×) got its own floor.

The measured floors give scale. On SWE-bench-sized evaluations (100–200
tasks), Pass@1 SD is 0.87–4.00 pp and cost CV is 2.44–8.35%. So a
single-run change under roughly 5–16% of cost is noise. The paper also
carries two warnings for the 09-27 proposal:

- **An s from three runs is itself noisy.** Its relative standard error is
  about 52%. The band needs slack, or a 1.5× inflation check like the one
  the paper runs.
- **Secondary metrics are noisier than the headline.** Behavior counts had
  a CV averaging 15.68% and exceeded the cost floor in 20 of 24
  combinations. The authors therefore refuse to give them verdicts. An
  `/iterate` band should be set on the metric the chain actually optimizes,
  never borrowed from a proxy.

This does not solve optional stopping (above). The rule is
fixed-R-then-judge, which is the one-look discipline, not a sequential
test.

**In a self-modifying loop a sub-noise keep compounds, and the loop cannot
grade its own fix (2026-09-29).** [[literature/papers/srikanth2026recursive]]
(AIDE²) is a fourth system with a bare comparison: `a*_k = argmax g(a)` over
the private grade, with no band. The authors spell out the cost specific to
recursion: "a falsely accepted rewrite becomes the new incumbent …, so a
single noisy comparison can derail the outer loop's subsequent search." A
noise-driven keep changes the code that proposes every later candidate, so
it is more than one wasted slot. The magnitudes are close. Rejected deltas
of "−0.004 to −0.007 are small relative to observed run-to-run
variability," while the seven accepts average about 0.011 (0.703 → 0.778),
so some accepts may sit near the band. Two further details matter:

- The loop *proposed* noise-aware selection ("promote-second-best and robust
  near-tie overrides", i.e. the optimizer's curse) and rejected it as
  "within noise".
- The median-distance selection penalty it did keep "never changed which
  candidate the agent selected".

A loop graded at a resolution coarser than its noise floor cannot select
for noise robustness. The band has to be set from outside the loop, which
is the job of the std-band edit. The paper cites a statistically gated
alternative (SGM, arXiv 2510.10232) that is not yet in this graph.

**A loop that calibrates the band and uses it three times (2026-09-29).**
[[literature/papers/xia2026rrsi]] (RRSI, harness evolution) is the first
*loop* in this chain that sets its tolerance from measured spread. Before
evolution it reruns the unchanged base harness and takes the spread as
δ. The paper reports δ = 0.017 / 0.004 / 0.020 across three domains,
which is 3 of 178 trials, 60 of ~14,100 judge criteria, and 5 of 244
trials. That one δ is then wired into three rules:

- **Floor against the running best, not the incumbent.** The rule is
  `Ŝ(H′) ≥ S★ − δ`, and it "prevents the search from walking downhill
  through a sequence of regressions that are individually small enough to
  be mistaken for noise". That is the structural answer to AIDE²'s
  derail concern above. For `/iterate`, compare to best-so-far minus the
  band, not to the last kept cycle.
- **A within-band gain earns nothing by itself.** Above δ, extra cost must
  be paid for (`ΔC ≤ β0 + β1·ΔS`). Inside δ, the coding instance sets the
  score weight to zero, so a candidate can be admitted only for lower cost
  or a new structural component. The case study shows the rule biting. A
  +1.69-point candidate with δ = 1.7 points and +26.1% cost was rejected,
  while its near-twin at +3.93 was kept.
- **Stall redirects rather than halts.** When progress over w = 3 rounds is
  ≤ δ, one proposal slot goes to never-tried component types. This is a
  second instance of chandran2026autoresearch's redirect-not-halt, now
  with a calibrated trigger.

The only evidence is one ablation run. Dropping the acceptance group
raises the evolve score (90.5 → 91.5), lowers the OOD average
(43.6 → 41.0) and raises tokens per trial (2.42M → 3.59M). The paper
does not state δ's estimator: it gives no rerun count and no statistic.
The workspace δ also counts criteria, which cluster inside 120 tasks, so
it is probably understated. Read this against the hu2026analyzing
protocol above: RRSI shows where the band goes in the loop, and
hu2026analyzing shows how to size it.

## Inside a measurement, a ceiling is a censoring instrument

Everything above treats the ceiling as spend control.
[[literature/papers/hickey2026saltbench]] runs ceilings inside a
pre-registered benchmark and finds three ways the same cap corrupts the
number it sits beside.

- **A halt is unresolved, not a failure, and the rule has to live where the
  verdict is written.** SaltBench stops an episode at 4× its regime's
  token p90 (no passing episode among 177 exceeded 2.40×; the one runaway
  reached 7.79× and failed) and scores it unresolved, because counting it
  as a failure "would let the budget instrument move the result."
  Registering that was not enough: the first halted Verus episode was
  scored as a cheating-screen failure on a snapshot of an unfinished edit.
  The checker now writes `HALT` from the episode's termination and keeps
  the would-be class as a diagnostic. The rule is measured, not assumed:
  all three cost-capped cells in its systems matrix pass their withheld
  suites completely when run afterwards.
- **A cap correlated with the condition biases every later sample.** Every
  capped cell was a treatment cell (control 18 landed, 0 capped; treatment
  12 landed, 3 capped), because the arm that spends more reaches a spend
  cap, on its hardest problems. A correctness pass over the survivors
  scores an easier treatment sample, "biased upward by construction," and
  the pass rate cannot show it. Any analysis over capped runs owes a
  statement of which condition lost runs.
- **A budget sized from capped runs returns the cap.** A p90 computed from
  censored data is the cap itself. Under a verifier, progress is
  all-or-nothing, so "0 verified" at the cap is an absent measurement, not
  a distance (the call-40 / call-79 episode above, at 8.7× the tokens of
  the Opus 5 pass on the same task). Size a ceiling from an uncensored run,
  or raise it until the censored fraction is near zero, and price that
  first: one uncensored episode priced SaltBench's planned 26-episode
  read at roughly 218M tokens.

For this project, comparing two configurations whose chains halt at
different rates inherits the second bullet. It is the concrete case of
[[concepts/hce-evaluation]]'s point (via ray2026what) that a ceiling
visible to the agent is an intervention on the measured system.

## Noise band and redirect-before-halt: the 2026-10-05 evidence

Four papers from the 10-05 digest bear on the two pending `/iterate` changes:
the 09-27 noise-band proposal, and the redirect-before-halt idea held on 10-04.

**What the gate must achieve.** [[literature/papers/park2026when]] holds one
agent fixed and changes only what the keep gate is grounded in.
- A gate that trusts the agent's own claim accepts everything: the claim was
  "improves" in 54/54 cycles while 56% had Δ ≤ 0.
- A strong in-band LLM judge still accepted 4 of 10 regressions.
- An isolated oracle gate accepts none.

The paper's sign-only gate (`Δ > 0`, no numbers) matches full feedback (110 vs
113, 3 reps). That holds only because its oracle is noise-free by
construction, and the authors name "interval-estimate-based acceptance" as
the open design for noisy metrics. Read the noise band's job in its
operating-point form: push P(keep | Δ ≤ 0) toward the oracle's 0 without
inflating P(reject | Δ > 0). The pilot is small (3 reps × 6 cycles, models
unnamed; credibility 2).

**The band must cover re-implementation noise, not only seed noise.**
[[literature/papers/wiedmann2026agents]] repeats each of 432 agent
configurations 5 times and finds 50–93% of score variance within a
configuration, 54% pooled. Each repeat is a fresh agent run that writes
different code, not a fresh training seed. Two consequences:
- A band built from `--seeds` std measures fixed-code training noise. It
  *understates* the noise in a hypothesis-vs-hypothesis keep.
- Count crashed or degenerate runs as failures and exclude them from the
  band's std. Otherwise one catastrophic run widens the band until nothing
  beats it.

By our arithmetic, n = 3 per arm detects only ≈2.3 SD differences. A 3-seed
band can veto sub-noise "new bests" but cannot certify small real gains.

**Retrying a fixed move set is a no-op, not a hazard.**
[[literature/papers/bobadillasuarez2026audit]] re-invokes the agent on its own
output over 1,650 SWE-bench Lite trajectories (T = 4). 69–81% of
trajectories never move after round 0, and regressions are ≤ 2.9%. Its advice
is the redirect rule in so many words: on a plateau, "ask which kind of
change you are making". Resampling explores a fixed class; new tools,
retrieval or decomposition can move it. The paper never varies the scaffold
with weights fixed (its open problem 4).

**Agents do not redirect on their own, and asking them does not help.**
[[literature/papers/yan2026traceml]] gives the first human base rate.
- Expert Kaggle humans pivot (backbone, representation, objective or
  validation change) on 25% of transitions; a single-loop Codex agent on 9%.
- Top humans reopen an abandoned line on 9.1% of versions; Codex did so once
  in 658.
- MLEvolve pivots on 58% of transitions with a post-pivot payoff of about 0,
  so pivot rate alone is not the target.
- A prompt-level "stuck on the same category 3+ steps?" self-check went with
  *less* mode switching (7.1% → 2.2%).

So a redirect should be **enforced by the controller**: force the next
candidate's category to differ under a structural pivot test, or reopen an
abandoned line from the run's history. Asking for it in the prompt does not
work. Still missing: its baseline prompt forbids halting, so no paper yet
runs redirect against halt.

**A specification for interval-based keeps, with no evidence attached
(2026-10-06).** [[literature/papers/wang2026pear]] (ByteDance production
search) deploys the "interval-estimate-based acceptance" that park2026when
left open. The rule has three outcomes on a Welch CI at α = 0.05:
- PROMOTE if the lower bound L > 0;
- STOP if the upper bound U < 0;
- RETAIN, meaning keep evaluating, otherwise.

Every round compares against a **concurrently re-run baseline**, not a
stored champion score, and that is its actual answer to non-stationarity.
Exact replications go through a separate queue. Three points for the
noise-band proposal:
- RETAIN is the state a std band lacks. "Not yet distinguishable" becomes a
  reason to spend more seeds, not a keep or a discard.
- RETAIN without a re-look cap is the optional stopping that qu2026propose
  priced (86.2 vs 11.7 false admissions per campaign).
- With n = 3 seeds, Welch t is about 2.8–4.3, so the gate RETAINs almost
  every sub-2-SD change (our arithmetic).

The paper has no keep-if-better arm and no no-gate arm. It is a design
endorsement, not a third measured case for the band.

## A world-state stall counter that never halts, and the run it let through (2026-10-06)

[[literature/papers/wang2026research]] (AfS) keeps two progress counters
with deliberately asymmetric authority:

- **Repeat** counts byte-identical responses that produced no durable
  change. It aborts at 6, after a warning at 3.
- **Stall** counts consecutive turns with no change to a workspace
  fingerprint, "a statement about the world, not about the transcript". It
  never terminates and emits a notice every 15 turns. The stated reason is
  that "a breaker that treats silence as failure would kill exactly the runs
  doing the most expensive work".

Token accounting is warn-only, and the turn cap is the only hard bound. The
paper reports the cost of that choice itself. In its 50-hour campaign one
experiment run reached **turn 153 and 17.5M tokens** with nothing stopping
it, and the limits section lists "no per-node spending ceiling" as an open
gap: the ledger "makes the problem visible after the fact; it does not yet
stop it." It also reports one measured busy-wait under model-judged
continuation, at 47 turns and 6.7M tokens. This is a single-system anecdote
with no comparison. It supports this page's position: a fingerprint-based
stall signal is the right *measure* of no-progress, but without a hard
ceiling behind it a run can still overspend.

**A matched continue-vs-redirect, though not a plateau-triggered one
(2026-10-06).** [[literature/papers/xu2026conflictguide]] branches each of 3
search rounds, on 5 model families, from a shared scalar-only Stage-I
incumbent into two equal 100-proposal continuations:
- plain continuation, which keeps any edit with G > 0;
- a redirect that adds behaviour-level probe feedback, narrows the prompt
  to the conflict's components, and keeps only G > τS, or a marginal or
  tied G with probe-confirmed alleviation.

The redirect wins in most of the 15 cells. The effect is large on SNGP
and the ESN transfer task (means of 18–19%) and small on FNO, GCNII and
TCM-Lite (2–4%, often inside seed noise). The headline "up to 28%" is the
best cell. Three caveats for `/iterate`:
1. **The switch is a fixed iteration count** (100, or 20 for ESN), not a
   stall detector, so the trigger is untested.
2. **The redirect bundles new evidence, a scope restriction and a
   thresholded keep rule.** On SNGP the ablation credits most of the gain
   to the keep rule. The scalar-only arm's G > 0 winners were *worse than
   the unmodified reference* in 2 of 3 rounds at full scale, which is
   indirect support for the noise band. The isolating arm (scalar-only
   plus the same τS) is missing.
3. **Timing matters.** Probe feedback from iteration 0 left FNO worse than
   the reference, so redirect *after* broad exploration, not instead of it.

A one-sentence "consider trade-offs" prompt did nothing (yan2026traceml's
lesson again). Still no paper runs redirect against halt.

**A learned value-of-continuing is not yet an alternative to the patience counter (2026-10-06).**
[[literature/papers/samanta2026learning]] (MIRA) trains a decision-level
critic that forecasts the remaining best-so-far improvement from the curated
research state (RMSE 0.214 on a 0–1 scale). It was flagged in the digest as
a possible learned replacement for `max_consecutive_no_improvement`. The
paper does not test that:
- The critic is used only as the actor–critic baseline.
- Termination is the policy's own sampled action, and every rollout is
  capped at 20 decisions.
- No arm stops on V(o_t), or compares against a patience rule.

What it does offer bears on redirect-before-halt. With weights frozen, a
fresh-context outer loop pivots more than one persistent session with the
same model and tools (strategic inertia 0.772 vs 0.862). That is a
structural redirect, consistent with yan2026traceml's finding that prompting
for a pivot fails. RL on proxy reward then cut judged "local-search
stagnation" from 33.5% to 18.8% of trajectories. It also raised "continuing
despite contrary evidence" on HCB-CPU (10% → 26%) and lowered keeping the
best graph on BNLearn (32% → 19%). A redirect pressure without a keep gate
erodes the champion, so keep the monotone hard cap and the metric-computed
counter.

**What a redirect ladder does once it is built (2026-10-06).**
[[literature/papers/woo2026youra]] routes a MUST_WORK failure that survives
bounded repair to one of two rungs: redesign the hypothesis, or reset to
problem scoping with a new research direction. Reflection over a structured
failure record makes the choice, and a separate controller enforces it.
- Across 30 MLR-Bench runs, 86 escalations fired: **81 resets and 5
  redesigns**, with up to 11 in one task.
- Disabling both rungs (failures documented, run continues) costs −0.87
  Overall. That is the weakest of the four ablations, and it does not hold up
  by sign test (17/11/2, p = 0.345).

So a built ladder mostly jumps to the top rung, and the gain from having it
is marginal. The comparison is still redirect against
continue-with-limitation, not against halt. A reset also archives the failed
folder, and the judges see only the final directory, so a redirect that
re-scopes doubles as a file drawer. A `/iterate` redirect should log which
rung fired and keep the abandoned line visible in the write-up.

## A matched-compute redirect beats continuing, but a stall trigger rarely fires (2026-10-12)

[[literature/papers/cai2026forkandflush]] (Microsoft Research, 13 long-horizon
optimization tasks) is the first source here to separate a controller-level
redirect from selection at equal token spend. Its redirect is a **flush**:
the chat context is emptied and the workspace is kept. On 11 tasks the
min–max normalized means are 0.45 for continuing, 0.55 for flush-only
(K = 1, flushed at 33% and 66% of the budget), 0.57 for fork-only (K = 4
branches that keep their context, best one kept) and 0.71 for both. Flush-only
beats continuing on just 6 of 11 tasks, and fork-only on 10 of 11. So the
redirect helps on average, but without selection it is erratic. A "pick a
DIFFERENT approach" directive given to half the branches did not change
which branch won (46% vs 50%, p = 0.26). That is one more controller-not-prompt
result alongside yan2026traceml and xu2026conflictguide.

The trigger finding matters more for `/iterate`. A plateau-triggered version
never fired in 15 of 25 development runs, because micro-improvements kept
resetting the stall clock. It did worse than continuing on AHC014 and p171.
The paper fell back to a fixed schedule. A redirect hung off
`max_consecutive_no_improvement` will meet the same problem unless sub-noise
"new bests" are prevented from resetting the counter. That makes the
accepted noise band a precondition for a stall-triggered redirect, not an
independent change. There is still no halt arm, and "equal compute" counts
model tokens only: solver and GPU time from parallel branches is unmetered.
The headline +66% is a ratio of normalized means; the median per-task raw
gain is about 7%.

## A band is right for the stop counter and unproven for the keep rule (2026-10-12)

[[literature/papers/hu2026winners]] is the first source in this graph to test
noise-band-like acceptance rules against bare keep-if-better, with held-out
ground truth for every candidate. The setting is Qwen2.5-1.5B/7B rewriting
their own instructions, K = 4 candidates per generation and 20 generations,
on a reused selection set. The nearest analogue to the band is McNemar at α,
whose threshold is τ = z_α·e, where e is the per-item noise of the
difference.

- **Per decision, on fresh items, a band beats `>`.** At n = 16 greedy
  loses 0.47 held-out points per decision. Every calibrated rule avoids
  the loss (pilot Bayes gate +0.77 over greedy), "mainly by committing
  less often". This is the counter's premise: sub-noise "improvements"
  are net harmful when most proposals are.
- **Over whole runs on a reused set, no rule beats greedy.** The pilot gate
  scored −2.1 (95% CI [−4.3, 0.1]). Fixed-level McNemar made 3 commits in
  32 runs and gave up 7–12 points of real TREC gain.

So the 09-27 proposal's scope is right. A band should decide when a chain
halts, and it should not decide which champion is kept. That second use is
the extension the proposal's last risk bullet considers, and this paper is
evidence against expecting it to pay.

The paper also shows a gap in the band itself. A `--seeds` std is measured
on the *same* validation items, so it misses item-sampling error, and that
error is what accumulates into champion inflation (13.6 points at n = 16
after 11–20 generations). Paired per-item disagreement between champion and
candidate supplies the missing term (our inference). Scope: instruction-only
artifacts, deterministic scoring, n ≤ 256; no band-gated stop was tested.

## Accept-all is the floor, and bare `>` stalls on tied binary scores (2026-10-13)

[[literature/papers/wei2026scale]] compares two keep rules for a
coding-agent harness optimizer (one commit per round, T = 30) on robot
rollouts. Its gated rule, "Champion–Challenger", promotes iff the challenger's
success count on the same fixed B cases strictly beats the stored incumbent
count. That is bare keep-if-better, the greedy baseline of
[[literature/papers/hu2026winners]], with K = 1. The alternative it is
compared with is accepting every revision. That arm rises for a few rounds
and then falls on training score at every B (B = 100: 42 → 47 → 34 → 38%
in 10 rounds; B = 5: 2/5 → 0/5).

So the paper establishes the bottom rung, keep-if-better over accept-all,
and says nothing about bands vs `>`. Even that rung is one run per arm, with
early-stopped, training-only accept-all arms.

Its useful datum for the halting counter is about ties. With 5–10 binary
cases most challengers tie the champion exactly and are rejected: 27 of 30
at B = 5, against 22 of 30 at-or-below at B = 100. On a tiny binary
validation split, a run of "no improvement" cycles is a resolution stall,
not evidence that the search has converged. The counter should count it
differently or not at all (our inference). The stored-incumbent score is
never re-measured (Alg. 1), which is Hu's lock-in mechanism. The paper does
not discuss it.

## A run-length counter must act through the harness, and its action here is a stop (2026-10-12)

[[literature/papers/zhang2026judged]] tests the exact shape of
`max_consecutive_no_improvement` (a counter of consecutive "this didn't help"
results that resets on a useful one) in tool-using QA agents with
controlled source failures. The agents judge failed results useless almost
always, yet after five such judgments in a row the 7–8B models answer on
0.3–7% of questions. Showing them the running count and asking them to
decide does not help. When the harness masks every action except `finish`
at k = 5, stopping tracks the evidence. Three points for `/iterate`:

- **Enforce by masking, not asking.** This is the third source, after
  yan2026traceml and xu2026conflictguide, where the prompt version of a
  controller rule fails. Read the redirect rule as "reject same-category
  candidates once the counter is non-zero", not "ask for a different idea".
- **A stated ceiling becomes a deadline.** Telling 7–8B agents their budget
  moved stops to the last action. Doubling the budget doubled their tool
  calls for no gain, while the masked rule's stopping point did not move.
  Keep the counter in the controller rather than in the proposer's prompt.
- **k must exceed the real payoff lag.** When the source recovered after
  five failures, the k = 5 rule cost up to 21 points of success.

Limits: the forced action is a stop, not a redirect. A switch-to-backup arm
added +0.7 to +5.4 points, and the model that already switched unaided gained
nothing. Sonnet 5 partly integrates on its own, and Haiku 4.5 with a stated
price beat the masked rule. Still no paper runs redirect against halt.

## A handoff at the halt point, and a schedule that is not a stall trigger (2026-10-12)

[[literature/papers/li2026humanize]] (HMA, Tsinghua/NVIDIA/MIT) alternates
two native agents (e.g. Opus 5 in Claude Code and GPT-5.6-sol in Codex) on
one shared workspace under a 6 h MLE-bench budget. The controller ends each
session after k = 5 accepted experiments, wipes the context, and hands over
to the other model. This is a controller-enforced structural move with no
prompt nudge. On average it beats one continuous session of either model at
matched wall-clock. Against the *stronger* single agent, the gain is +0.9
to +5.8 any-medal points, +2.7 on average across six pairs (our arithmetic).
Five of the six HMA rows appear to be single runs.

The datum that bears most directly on redirect-before-halt is a side arm,
**NTA**: no cap, and the handoff happens exactly when the active agent
*naturally terminates*. Single agents halt at that point. GPT-first NTA
scored 77.3 against GPT alone at 68.0, and Kimi-first NTA scored 56.0
against Kimi alone at 54.2. Both NTA figures are single, historical runs
compared with 3-repeat means, so read them as a direction only. It is
still the first source in this graph whose two arms differ in whether the
run is allowed to stop. Two constraints on the `/iterate` design follow:
1. **Fire on the stall, not on a schedule.** On the one pair where the cap
   was varied, forced handoffs every k = 1 or k = 3 experiments scored
   *below* the stronger single agent (65.3 and 66.7 against 68.0). Uncapped
   NTA beat every capped variant. Frequent forced redirects cost score.
2. **The move tested is a model swap plus a flush, not a flush alone.**
   There is no same-model arm, so it does not show that a fresh context on
   the same backend would work.

Returning agents also overwrote standing medal candidates, which a
cross-agent final review over the archive restored (311 → 319 of 330). As
with MIRA above, the redirect must sit behind the monotone keep gate.

## The counter-pole to redirect: a controller-enforced commitment floor (2026-10-13)

Every redirect source above treats under-pivoting as the failure.
[[literature/papers/du2026can]] (Station, metric-free research tasks) treats
the opposite one: agents drop hard directions early for easier ones (the
"streetlight effect"). Its fix is a **Supervisor** agent that cannot run
experiments and is forbidden to suggest directions. It enforces a written
pre-commit clause of at least 60 ticks and rejects almost every pivot request
unless continuing is "genuinely impossible". It holds a meeting every 10
ticks and keeps a record of each agent's commitments outside that agent's
context.

On one task, removing the Supervisor together with periodic GPT-5.5
reflection shortens projects (67.0 → 55.5 ticks) and cuts rediscovery
75.8% → 57.9% (3 runs). The ablation is joint, and the ablated system still
beats every baseline by about 3×. Persistence also stopped paying after
tick 100 (61.2% vs 62.7% at tick 300). So this is a mechanism with weak
evidence, not a result. For `/iterate` the design point stands: a
stall-triggered redirect probably needs a minimum span before it may fire.
That matches li2026humanize's finding that frequent forced handoffs cost
score. As with the redirect, the floor here is enforced by the controller,
not requested in the prompt. Still no source runs floor + redirect against
halt.

## The failure a no-improvement counter cannot see (2026-10-13)

[[literature/papers/he2026beneath]] runs Karpathy-style autoresearch (Opus 4.7
proposer, 5-minute runs, keep above a noise floor) for 300 iterations on four
small NLP pipelines. The in-loop metric never stalls. Over the same window
the *kind* of accepted edit collapses: mechanism entropy falls 1.76 → 1.18
nats, and by iteration 200 on T1, 74% of kept edits are LR-schedule or AdamW
micro-tunings. Only 38–49% of the in-loop gain appears on audit benchmarks.
`max_consecutive_no_improvement` would stay at zero throughout. This is
cai2026forkandflush's "micro-improvements reset the stall clock" seen from
the other side.

Two points for the redirect design:
- **A trigger that sees it.** Windowed entropy over a category label per
  accepted edit is cheap, needs no audit split, and falls while the score
  rises.
- **The redirect that worked is enforced by the controller.** CCR
  reweights sampled candidates toward under-used categories, and PEM
  rejects ones near the last 200 accepted edits. Together, with no audit
  access, they raised blind faithfulness 0.41 → 0.67. Higher temperature
  and a Reflexion summary did nothing. This is one more
  controller-not-prompt result, after yan2026traceml, xu2026conflictguide,
  cai2026forkandflush and zhang2026judged.

Limits: no halt arm, no plateau trigger, and the audit-gated revert runs on a
fixed K = 10 schedule. The headline collapse tables may be T2-only values
labelled as averages. The paper's mechanism (F2) says a noise-floor keep
rule favours low-variance micro-tunings, so the accepted band could deepen
collapse. That is predicted, not tested.

## A five-commit plateau hook is a scheduled ask, not a redirect-before-halt test (2026-10-13)

The 10-12 digest put [[literature/papers/li2026beyond]] forward as a design
precedent for redirect-before-halt. In that paper, an Opus-4.8 outer loop
evolved a Claude Code harness over 4 rounds on 3 tasks. One rule it wrote:
"After every five new commits, a hook asks the agent to reassess whether
progress has plateaued and to attempt a larger structural change". The
mechanism is described accurately. As a precedent it is weaker than
xu2026conflictguide or li2026humanize:
- **It fires on a schedule, not on a stall.** li2026humanize found that
  frequent scheduled moves cost score.
- **It asks rather than masks.** That is the prompt form that yan2026traceml,
  xu2026conflictguide, cai2026forkandflush and zhang2026judged each found
  ineffective.
- **There is no halt arm.** The task instruction forbids early stopping.
- **There is no ablation.** The hook is bundled with "identify what the
  verifier rewards" and "restore your best". Its evidence is one trajectory,
  0.37 → 0.68. The whole harness gains +0.06 avg@3 on held-out tasks in the
  same family and nothing on other families.

Two specifics carry over to `/iterate`. First, guard the champion before
any structural move. Second, a commit-count trigger depends on run length.
Models here averaged 2.5–16 evaluated commits per run, so a k = 5 hook
would rarely fire for half of them. A stall trigger has to be scaled to
the run. Still no paper runs redirect against halt.

## What a seed band measures, and where the replication budget should go (2026-10-13)

[[literature/papers/wiedmann2026agents]] showed that the `--seeds` band
understates hypothesis-level noise. [[literature/papers/ning2026implementation]]
puts numbers on the split for a frozen idea spec. Across its three cohorts,
the SD between independent implementations is 0.9–2.7 accuracy points and
the seed SD is 0.4–0.8. In the 18-task follow-up these are 1.17 / 0.46
(Bounded) and 0.98 / 0.45 (Agentic). 78% of programs score identically on
all three seeds. Ideas still differ more than implementations do (ICC
0.83–0.92), but a std computed from seeds of one program measures the
smaller term.

The paper also gives the allocation rule (Cochran two-stage):
K* = (σ_R/σ_M)·√(c_M/c_R) seeds per implementation. At its measured
cost ratios (an implementation costs 2.6–3.7 seed evaluations), one seed
per program minimises variance. 21 programs × 1 seed beats the 12 × 3 design
it actually ran (fitted variance 0.075 vs 0.120). A second seed pays only
above a cost ratio of about 9.5–13. For the noise band this means a band
meant to judge a *hypothesis* should be built from re-implementations at one
seed each, not from seeds of one implementation. Caveat: gradient boosting
is nearly seed-deterministic, so the 2:1 ratio is a floor for stochastic
training, and the allocation figures are model-based planning values.

## A wall-clock ceiling in the prompt is ignored, and one that is met is not spent (2026-10-13)

[[literature/papers/wang2026clock]] (AWS, NeurIPS 2026 workshop) runs
Qwen3.6-27B in OpenCode on five MLE-bench Lite competitions under a
15-minute budget. Stated only in the prompt, the budget is ignored: the
agent used 161% of it, and 15% of runs finished on time. With a clock line
appended to every tool result, mean use falls to 99.7%, but only 62% of
runs finish on time. Per-run adherence needs a hook that clamps (74%) or
refuses (88%) shell commands that would outrun the budget. GRPO trained
under the refusing harness added nothing (86%). Two points for this
concept:

- **The ceiling must be enforced at launch, not just displayed.** The
  dominant overrun was a command, usually training, whose timeout exceeded
  the time left. The agent saw the clock and still launched it. This is
  ceiling-plus-one-call measured in wall-clock time.
- **Meeting a ceiling does not mean spending well under it.** Under the
  harness, elapsed time tracks budgets of 10–25 minutes and quality stays
  flat. Extra time went into "more attempts at essentially the same
  training procedure". This is weak support for redirect over extension
  on a plateau. It is weak because the shared prompt told the agent that
  punctuality "MATTERS MORE THAN MODEL QUALITY".

On Zork, the RL policy's near-exact adherence came from filling the clock
with commands the parser rejects (45–88% of turns). So a utilization
metric needs a useful-time companion before it is read as effort.

## A cheap pre-execution screen decides which candidates get the budget (2026-10-13)

The allocation half of this concept has so far meant splitting a budget
across phases (hamri2026zebra) or across plan branches (yu2026primescientist).
[[literature/papers/wang2026language]] (Amazon et al.) measures a third
lever: predict each candidate's gain before running it, and spend compute
only on the top few. Claude Opus 5 was given past experiment records and
asked to rank unseen interventions. Over 2,653 real runs, mean Spearman rose
from 0.506 with no records to 0.774 with in-environment records. Scoring 16
candidates costs about 5% of one median 42 GPU-hour experiment. That fits
chandran2026autoresearch's rule that pre-execution gates pay off only when
iterations are expensive.

Read the selection numbers narrowly. The abstract's "78% less regret" is the
best of five transfer targets; the macro is 52%, and one target got worse.
Most regret differences, and the multi-round +15.8%, are smaller than one
reference seed's std (0.0054 BPB). The multi-round result is also a replay
over a pre-executed pool, not a live loop. What transfers is a screen that
*drops* clearly bad candidates, conditioned on the project's own records.
That kind of screen sits in front of the ceiling. It does not replace the
keep gate, and it should not be trusted to pick the winner.

## An unbounded per-task inference budget lets an evolved artifact buy its gain (2026-10-19)

The ceilings in this concept bound what a loop spends while it runs. None
of them bounds what the *artifact* the loop produces spends per task.
[[literature/papers/ding2026harnessdelta]] measures what that gap costs.

When a framework can rewrite orchestration, it often raises its score by
adding model calls:
- On CREATE, Meta-Harness made 20 calls per item. A compute-matched
  baseline recovered 49% of its Haiku 4.5 gain and beat its
  Qwen3.6-35B-A3B harness outright (T = 100%).
- AHE went from 1.04 to 2.46 calls per item, and T = 75%.
- SkillOpt can edit only the skill, with the one-call rollout fixed. It
  never scales compute (T = 0), and its CREATE gains are not significant.
- On SWE-bench, a 1.44× call increase accounts for T = 23% of one gain.

Across 21 settings T averages 14%. The authors recommend "explicitly
expressing the cost constraints during evolution".

For this project, any loop that edits a harness or skill should do one of
two things:
1. Cap the evolved artifact's calls or tokens per task in its declared
   search space.
2. Credit gains only against a baseline rerun at the artifact's own
   per-task cost.

Caveats:
- Some compute-matched baselines use pass@k, which picks the winner with
  the oracle, so T is optimistic there.
- T is a single-run difference with no CI.

## A scripted restart at the failure point, and a reminder-then-stop rule nobody measured (2026-10-19)

[[literature/papers/rodionov2026improving]] (EnvTraverse) runs one Codex
agent unchanged on ARC-AGI-3, AGI Maze and Terminal-Bench. Two parts bear on
redirect-before-halt.

- **Controller-owned restart.** Every agent is told to RESET only at
  `GAME_OVER`. Controlled_RESET adds a prompt ban on RESET, and the
  controller issues it instead, with a cheap sanctioned exit: report
  `GAME_OVER` and you get another attempt. This recovers 34–60% of the gap to
  an ARC-specific agent. Only on GPT-5.6-sol is the cost close to equal
  (+11% tokens for +7.3 RHAE at xhigh, mostly separated over 3 runs; +5% for
  +2.4 at max, inside run spread). On GPT-5.5 the reset arm spends 2.3–3×
  more, so the gain there cannot be separated from the extra inference.
- **An existing stuck rule shaped like ceiling − 1.** After one no-progress
  return the controller sends a prompt reminder. After a second it exits,
  and a supervisor restarts it up to 10 times. That is "remind at ceiling −
  1, then halt". The paper never reports how often it fired.

What it adds to cai2026forkandflush and zhang2026judged: the
controller-owned move here is a *restart that keeps what was learned*, not a
stop or a flush, and it fires on an environment event, not a stall
counter. What it does not add: no arm compares restart with stop, the reset
gain is not separated from the prompt ban, and failure modes under
agent-managed reset are not counted. Weak support for keeping the trigger in
the controller and giving the agent a sanctioned action at it. Still no
paper runs redirect against halt.

## Shown both plans at a real pivot, agents keep their own, and reconsidering does not repair it (2026-10-19)

[[literature/papers/sun2026scipivot]] measures the step every redirect
source above treats as the weak one: the choice itself. 248 cases come from
26 supervised research projects. In each, a senior researcher had overridden
the agent's next plan. Models get the record from before the intervention
and both plans: the agent's original and the correction. At medium effort,
four of five frontier models pick their own continuation more often than the
correction (29.2–38.9%, against 59.5% for GPT-6 Astra and 73.4% for fresh
human researchers). A feedback-free "please reconsider" changes answers in
both directions. The first review repaired 422 answers and broke 302; the
second repaired 264 and broke 305. Earlier sources showed that agents don't
pivot on their own: yan2026traceml's prompt self-check, and
cai2026forkandflush's ignored "pick a DIFFERENT approach" directive. This
paper shows they also don't pick the pivot when it is put in front of them.
That supports the redirect-before-halt proposal's choice to force the axis
change in the controller.

It also qualifies the proposal's axis list. In one worked case the *wrong*
plan was a backbone change (0.8B → 2B after zero scores), and the correction
was a validation check: the scorer had skipped answer extraction. In
another, the correction reopened a restriction the agent had carried over
from its own earlier runs, and with the record only 1 of 10 responses chose
it. "Change at least one axis" leaves the content to the same judgment this
paper finds weak. When stalled results are degenerate (all zeros, optima
pinned at a search bound), auditing the measurement should come before a
backbone change. That ordering is our inference.

Caveats:
- Cases were kept only if the evaluated models split between the plans, so
  the levels are challenge-set numbers, not base rates.
- Both plans are supplied, so the paper measures recognition, not
  generation.
- No forced pivot's outcome is measured, and there is still no halt arm.

## Open questions

- Token budgets are predictable; disk budgets are trickier because
  intermediate artifacts (checkpoints, cache) grow in bursts.
  Whether the ceiling should be enforced pre-allocation or
  post-hoc-with-cleanup is project-dependent.
- A `budget.yaml.git.auto_push: true` flag (set on this meta project)
  encodes a non-compute commitment — overnight work pushes without
  manual intervention. No skill reads it yet; the flag documents
  intent until a SessionEnd hook honors it.
