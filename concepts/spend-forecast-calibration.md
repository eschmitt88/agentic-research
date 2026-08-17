---
kind: concept
name: "spend-forecast-calibration"
status: seedling
added: "2026-08-17"
sources:
  - "[[literature/papers/bai2026how]]"
used_by: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/context-eviction-policy]]"
related_experiments: []
tags: [budget, spend-policy, forecasting, calibration, self-modeling, cost-accounting]
---

# spend-forecast-calibration

## Definition

A budget gate needs a *number* to gate on, and that number must come
from a calibrated external forecast — never from the agent's own
estimate of what it is about to spend. Agent self-estimates are weakly
correlated with actuals and biased downward, so they are admissible only
as coarse tier or alert signals; anything that admits or refuses work
must be driven by measured history plus an explicit uncertainty margin.

## Why it matters here

[[concepts/budget-as-ceiling]] answers *whether and where* spend halts.
It does not answer where the projected-spend number comes from, and
under `agency: max` this project routinely acts on such numbers — the
coordinator's headroom verdict, `suggested_session_tokens`, the reserve
buffer sized against ceiling-plus-one-call. If those inputs are an
agent's self-report, the ceiling inherits the self-report's bias.

[[literature/papers/bai2026how]] measures that bias directly and it is
the unhelpful direction. Given full tool access, permission to inspect
the repository, and an instruction to estimate rather than fix, eight
frontier models reach Pearson r ≤ 0.39 against their own actual token
usage (best case: output tokens, Sonnet 4.5; input tokens are harder for
every model but Kimi-K2 at 0.38), and **every model systematically
underestimates** — most severely on input tokens, which are the
dominant cost term (input/output ratio 153.85 in agentic coding). The
bias persists with no in-context example. So a self-estimated reservation
does not fail symmetrically around the truth; it fails by admitting work
that will overrun.

Three design consequences:

1. **Self-estimates are a tier signal, not an allocation.** The paper's
   own reading is that self-prediction is "a useful coarse-grained
   signal of relative cost" — good enough to raise a budget alert or
   sort cheap-vs-expensive work, not to size a reservation. Use it to
   decide *whether to ask*, never to decide *how much to grant*.
2. **The forecast belongs outside the agent.**
   [[literature/papers/besanson2026green]] supplies the constructive
   half from the other side: a learned cost forecast plus a
   split-conformal margin gives a distribution-free per-action breach
   bound (δ), where the Normal-σ version under-covers on real
   right-skewed residuals (92% at nominal 95%) and conformal holds
   (95.2%). Pairing the two papers gives the whole claim — the agent
   cannot supply the estimate, and a calibrated external predictor can.
   The asymmetry of bai's bias is also the argument for a *one-sided*
   margin rather than a symmetric interval.
3. **Calibrate on the right denominator.** Because retries and failures
   are part of what a budget actually pays for, a forecast fitted to
   per-call or per-success-only cost understates the real draw. Cost
   normalized by *completed goals* is the honest unit.

The self-prediction result is not simply a capability gap to be waited
out, and it is worth separating from
[[concepts/context-proprioception]]. Proprioception concerns the agent's
*present* resource state — token cost per block, recency, remaining
headroom — and is an interface problem: surface a machine-maintained
ledger and the meta-decisions improve. Forecasting concerns *future*
spend, and bai's agents already had the privileged access an interface
would give them (they could read the repo and run preliminary commands)
and still missed low. Present state is fixable by plumbing; future cost
is not, because the trajectory's own branching is what sets it. Keep the
two claims distinct: expose state, do not trust prediction.

## Why the numbers are hard to forecast at all

bai2026how also measures *why*, which bounds how good any forecaster can
get:

- **Heavy tails within a single task.** The most expensive of four runs
  on the *same* problem roughly doubles the cheapest (up to 30× in total
  tokens on some tasks), and cross-run variance is largest exactly on
  the high-cost problems. A point forecast is the wrong object; a
  quantile is the right one.
- **Human difficulty is a weak feature.** Kendall τ_b = 0.32 against
  expert-estimated resolution time; 6.7% of "<15 min" tasks cost more
  than the average ">1 hr" task. Difficulty labels — including our own
  in experiment `README.md` frontmatter — should not be used as a cost
  proxy.
- **Cost is model-idiosyncratic.** Relative token efficiency holds
  across both the shared-success and shared-failure subsets, so a
  forecast must be fitted per backend, not per task family. See
  [[concepts/hybrid-model-backends]].
- **Failure is where models diverge most.** The success→failure cost
  increase ranges from <0.5M tokens (GPT-5/5.2) to ~2M (Kimi-K2),
  because models lack a reliable mechanism to recognize an unsolvable
  task and stop. Forecast error therefore concentrates on the runs a
  ceiling most needs to catch.

## Implementation guidance

1. **Fit on measured history, per backend.** This box already records
   the input: `~/.claude/state.db` holds actual usage and the
   coordinator's agency CLI already reports weekly consumption. That is
   a forecaster's training set; the agent's guess is not.
2. **Gate on an upper quantile, one-sided.** Given the documented
   downward bias and right-skewed residuals, size the reserve from a
   residual quantile rather than a symmetric interval or a flat
   percentage — the concrete replacement for the flat 10–15% buffer
   `budget.yaml` currently lacks entirely.
3. **Let a self-estimate trigger a question, never an admission.** A
   skill may ask "does this look expensive?" and escalate to the user or
   to a cheaper backend on a yes. It may not convert the answer into a
   token grant.
4. **Report forecast error, not just spend.** A run that stayed under
   its ceiling but blew past its forecast is a calibration failure worth
   logging even though nothing halted; that is the data the next
   forecast needs.

## Connections

- [[concepts/budget-as-ceiling]] — enforcement; this concept supplies
  the number enforcement acts on. The two are deliberately separate:
  a ceiling with a bad forecast is still safe (it halts late), but a
  *reservation* with a bad forecast is unsafe (it admits too much).
- [[concepts/context-proprioception]] — present state vs future cost;
  see above for why the two should not be merged.
- [[concepts/hybrid-model-backends]] — token efficiency is inherent to
  the backend, so forecasts and routing share the same per-model
  parameters.
- [[concepts/context-eviction-policy]] — cache reads dominate billed
  cost, so any forecast denominated in token *surface* rather than
  billed cost will misprice eviction decisions.

## Open questions

- How much of the self-prediction gap is prompt/compute-limited? bai
  ran one prediction prompt and concedes better prediction may be
  reachable; the ceiling on self-estimation is unestablished, only its
  current level. Worth re-testing if a cheap self-estimate ever looks
  attractive.
- Does the downward bias survive on research-agent trajectories, where
  a single training run can dominate the bill? The coding-agent phase
  mix (Explore+Fix ≈ 64% of rounds) probably does not transfer; the bias
  direction likely does.
- No forecaster exists in this repo yet. Until one does, this concept is
  a constraint on what *not* to build (self-estimated reservations)
  rather than a component to import.
