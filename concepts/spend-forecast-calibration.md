---
kind: concept
name: "spend-forecast-calibration"
status: seedling
added: "2026-08-17"
sources:
  - "[[literature/papers/bai2026how]]"
  - "[[literature/papers/panigrahy2026energy]]"
  - "[[literature/papers/ge2026coverage]]"
  - "[[literature/papers/yu2026primescientist]]"
  - "[[literature/papers/liu2026what]]"
  - "[[literature/papers/wang2026clock]]"
  - "[[literature/papers/kumar2026turncost]]"
  - "[[literature/papers/srinivasan2026continue]]"
  - "[[literature/papers/jain2026monitor]]"
  - "[[literature/papers/kanke2026rtaste]]"
  - "[[literature/papers/liu2026silent]]"
  - "[[literature/papers/shi2026hard]]"
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

## Normalize by outcomes, not attempts

[[literature/papers/panigrahy2026energy]] supplies the denominator this
concept's third design consequence asserted without a source. Its argument
is that the field's unit is simply wrong for agentic work: inference count
is an *implementation artifact* rather than a task property, so a system
that retries four times before succeeding reports the same
energy-per-inference as one that succeeds immediately while consuming ~5×
the energy. Failed attempts are invisible by construction — and they are not
cheap. In a measured GSM8K trace the failed attempt drew **2,256.1 J against
1,358.4 J for the successful one**, so inference-level accounting missed
**62.4% of that goal's true cost**; failures often cost more than successes
because the model exhausts more computation before producing invalid output.

Their replacement is **Energy per Successful Goal**: aggregate every attempt
associated with a goal, divide by the goals an evaluation function accepted.
Measured this way, agentic workflows cost **4.33× more per successful goal**
than matched linear baselines (888.1 J vs 205.3 J across 827 goals), with
the overhead attributed to orchestration structure — retries, intermediate
planning, recovery — rather than to more inference compute.

Two things follow for forecasting. A forecaster fitted to per-call cost, or
to successful runs only, is fitted to the wrong quantity and will underrun
reality by whatever the local retry rate happens to be — compounding the
downward bias bai2026how already measures. And the anti-gaming rule
generalizes: goal granularity must be fixed by the *specification*, never by
the system's internal decomposition, or the unit can be improved by
re-splitting the work. That is the same structural instinct as fixing the
holdout outside the agent's reach.

One methodological caution worth carrying into any forecaster built here:
the paper's *boundary problem*. Allocating cost as rate × wall-time folds in
fixed setup/teardown, and because short runs finish faster that fixed cost is
a larger fraction of their total — biasing any agentic-vs-linear ratio toward
1.0× and hiding real overhead. Measure the window deliberately.

## A meter that sums every logging layer over-counts by a factor that varies with scaffold and provider, so print the raw-to-root ratio on every run (2026-11-16)

This concept's Definition asks for "measured history". The allocation
section says a controller "needs only a meter". Both assume the meter is
right. [[literature/papers/liu2026silent]] shows how a widely used one was
wrong. HAL's cost column, the cost axis of its cost-accuracy frontier,
sums usage over every tracing span. A wrapper call and the provider-client
call it makes show up as parent and child spans with the same response and
the same usage block.
- In 133 of 134 logged runs, the reported usage equals the all-spans sum.
  This identifies the rule.
- Reported tokens are **exactly 2×** the root-span sum in 70 runs and 1×
  in 46.
- 44 of 94 matched leaderboard rows carry a doubled cost. Mean
  overstatement over 133 runs is 36.5%.
- Example: one row lists $149.98 for a run whose root-span cost is $74.99.
  Both look like believable prices.
- **The factor follows the client path**, which depends on scaffold ×
  provider. On SciCode all ten zero-shot runs are clean, while most runs of
  the other two scaffolds are about 2×.

The last point matters most for this concept. liu2026what and
kumar2026turncost (sections above) say a forecaster should be fitted per
harness × backend. An error whose size changes with exactly those keys
does not cancel within a stratum. It biases each stratum's fit by its own
factor and reorders strata against each other. A uniform 2× would only
rescale the axis.

This box has hit the same class of bug twice:
- 07-26: the Stop hook stored cumulative session totals as per-turn rows,
  about 51×.
- 11-14: the hook summed usage over transcript records. One response is
  written as several records repeating its usage (11,934 records for 6,490
  responses in the largest transcript), and subagent transcripts were not
  read. The net error was about 2.4×. Counting now goes through a
  deduplicated reader in `coordinator/token_log.py`.

Liu's fix matches ours: count once per response ID, or over root spans
only. Liu adds a canary we lack: **print the ratio of raw to deduplicated
usage on every run.** "A ratio of exactly two is a bug signature that any
reader can recognise." Here the healthy ratio is not 1. It is whatever the
transcript format currently yields, roughly 1.8 records per response on
the 11-14 sample. So the alarm should be a change in the ratio, not a
fixed value.

Our inference:
- Have the token reader log `records / unique_response_ids` and the raw
  and deduplicated totals for each session.
- Flag a step change, which would mean Claude Code changed how it writes
  transcripts.
- Tag every `token_events` row with a meter version. Any forecaster or
  ceiling calibrated on rows from before 11-14 is fitted to an inflated
  history whose size varied, and needs a refit, not a rescale.

Limits:
- HAL's meter is a tracing library over provider clients. The box's meter
  reads Claude Code transcripts. The mechanism is analogous (one response,
  several records), not identical, and the box figures come from the 11-14
  system review, not from the paper.
- The paper measures metering error only. It fits no forecast and does
  not show the downstream bias in a cost model. The stratum argument above
  is ours.
- HAL has disclosed the overstatement but has not re-derived the costs,
  so no corrected cost frontier exists to compare against.

## Outcome difficulty forecasts from the item's history with other agents, not from the item; a pre-run router gained +0.41 pp (2026-11-16)

The 11-02 section said task difficulty is forecastable before the run and
attempt outcome is not, and flagged that deployable pre-execution features
reached only 0.51–0.57. [[literature/papers/shi2026hard]] (same NC State
group) separates where the forecast comes from. Data: 3,492 mini-swe-agent
runs of seven models on SWE-bench Verified.
- **Difficulty transfers across models.** The share of 16 *other* models
  that fail an issue ranks failures at 0.924 pooled AUROC, at step 20.
  [[literature/papers/srinivasan2026continue]]'s 0.97 used held-out
  attempts by the *same* model. So the forecastable part is a property of
  the item, carried by any population of agents that has met it. It needs
  outcomes, and the paper calls it a diagnostic, not a deployable
  predictor.
- **From item features, the forecast buys almost nothing.** A router that
  predicts each model's chance from model identity and the issue text,
  before any run, beat the best single model by only +0.41 pp
  [−1.83, 2.85]. Choosing per issue from that issue's own past runs gave
  +0.09 pp [−0.36, 0.63] on new runs (two-model Llama pilot).
- **Model ability is the second forecastable term.** A model's failure
  rate on training issues alone gives 0.724 within-issue AUROC (0.82 over
  23 models). Trajectory predictors added nothing beyond difficulty plus
  ability (equivalent to zero at ±0.02, 18 configurations).
- **A floor no forecast removes.** In the repeated-run pilot,
  run-to-run variation is 42% of outcome variance. That share is
  unforecastable before the run by construction.

For this concept (our inference): the admission/allocation rule stands,
with a narrower input. An outcome forecast at admission is calibrated only
for items with a history: benchmark items, or a project with earlier
chains. A new `/iterate` project is a new item. Before it runs, the usable
terms are the backbone's base rate and a per-model margin, as in
[[literature/papers/jain2026monitor]] and [[literature/papers/kanke2026rtaste]].
Item text gives little. Record per-problem outcomes so that later chains
on the same problem have a history to forecast from.

Limits:
- No cost forecasts. This is outcomes only, with binary resolve labels.
- The router is described in one sentence, with shallow issue features.
  A richer pre-run model could do better; the paper's CI allows +2.85 pp.
- One benchmark and one scaffold. The pilot uses older Llama models, and
  the variance-decomposition method is not given.

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

## Turn count carries the cross-run cost variance, and a token count is not a bill (2026-10-20)

[[literature/papers/kumar2026turncost]] re-prices 600 published
coding-agent trajectories (2 scaffolds × 2 open-weight models) under a
simulated prefix cache. On log–log axes, turn count alone explains
R² = 0.844 of per-trajectory cost (exponent 1.23). Mean observation size
alone explains 0.040, with a slightly negative slope. Cost per turn is
fairly stable (median 0.005 normalized units, CV 0.28). From the first to
the tenth turn-count decile, median turns rise 36 → 168 and median cost
about 7×.

This sharpens the section above. liu2026what found step count can outweigh
harness preamble. Here the per-turn cost is steady enough that a forecaster
can be (turn-count quantile) × (per-turn cost), fitted per scaffold ×
backend, which is what this page's implementation guidance already asks
for. It is still not a forecast from inside the agent. bai2026how's
r ≤ 0.39 for self-estimates stands, and the turn count is the unknown.

The paper also puts a number on units. Flat input pricing overstates the
cached bill by 6.5–7.8× (mean 7.14×), rising from 5.33× below 40 turns to
7.99× above 130. At 90% cache efficiency the gap is 4.0×, and at 50% it is
1.45×. A gate and a forecast must use the same units. A raw token ceiling
that counts cache reads at par with fresh input is a much looser limit on
billed spend than it looks on long sessions.

Caveats:
- The 84% is partly mechanical, since cost is a sum over turns. The
  authors say so.
- Adding observation size to turn count raises R² by 0.101. "4%" is the
  univariate figure, not observation size's whole contribution.
- The cache is simulated (h_t = C_{t−1}), not metered. liu2026what's
  metered 96–99% hit rate is the better anchor for this box.
- The corpus excludes runs that hit the iteration limit. Those are the
  long-tail runs a ceiling most needs to catch.

Rule: fit the turn-count distribution, not observation size, and state
whether a token budget counts cached reads.

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


## Forecasting is required for admission, not for allocation

A boundary this concept has not drawn. Its core claim — a budget *gate* needs
a calibrated external forecast — is about deciding whether to *start* an
action whose cost is not yet known. [[literature/papers/yu2026primescientist]]
shows an *allocator* needs no forecast at all: its control signal
`r_t = (B − U_t)/B` is realized consumption over a known constant, with no
prediction of the next action's cost anywhere, and its Definition 1 states
outright that costs "may be unknown before completion."

So: **forecasting is a requirement for admission decisions, not allocation
decisions.** Deciding *whether* this action fits needs an estimate of its
cost; deciding *how widely to explore given what is left* needs only a meter.

This is the one design point `bai2026how`'s r ≤ 0.39 does not undermine —
that result says self-estimates of cost are poorly correlated with actual
cost, which is fatal for admission and irrelevant to a controller reading a
counter. Where this repo needs budget-sensitive behaviour, prefer the meter
over the estimate.

## Outcomes split the same way: task difficulty forecasts before the run, attempt outcome does not until late (2026-11-02)

The admission/allocation split above is about cost.
[[literature/papers/srinivasan2026continue]] finds the same split for
*outcomes*, on 147,002 coding-agent runs in four settings.
- **Task-level outcome is forecastable before execution.** Outcomes
  cluster strongly by issue (intraclass correlation 0.71–0.82). A held-out
  task solve rate alone gives pooled AUC 0.97. Pre-execution issue
  features give 0.51–0.57, and early prefix counts add only 0.02–0.04.
- **Attempt-level outcome is not, until most of the budget is gone.**
  Within the same issue, every predictor sits at AUC 0.50 at 20k tokens.
  That includes a task-conditional model and a fine-tuned 0.6B monitor
  (three seeds, 32,406 attempts). Signal reaches 0.54 at 40k and 0.56 at
  60k, when 23% of runs are still active.
- **Mid-run reallocation gained only what admission could have.** The one
  setting-dependent gain of a learned restart over a random one (≤ 1.98 pp)
  is between-task. It comes from stopping hard issues, which a
  pre-execution screen does for zero tokens.

For this concept: put outcome-aware spending at admission and routing
time, where the forecast exists. The paper's own example is scaffold choice
(14.06 pp on shared tasks, against ≤ 1.98 pp for restarts). Mid-run, read a
meter, not a prediction. This is the same rule as above, now for outcomes.

The paper also shows the unit point again. It counts cost as transcript
length, and says so, which ignores caching and context re-reads. A
restart-vs-continue cost comparison in transcript tokens is not one in
billed tokens ([[literature/papers/kumar2026turncost]]).

Limits:
- External monitors over recorded runs. No agent self-forecast was tested.
- Coding issues with binary resolve labels and open-weight models. Whether
  ML research cycles (one training run per cycle) show the same task
  dominance is untested.
- "Forecastable before execution" is shown as pooled AUC with a task-rate
  oracle. Deployable pre-execution features reached only 0.51–0.57.

## A harness bill is three numbers, and the harness sets two of them (2026-10-13)

bai2026how says a trajectory's cost cannot be forecast from inside the
agent. [[literature/papers/liu2026what]] shows how much of it is fixed by
the *harness* before the first step. Per trial, the bill is roughly
(first-call preamble) + (growth per step) × (step count). On GLM-5.3-Flash
the preamble (system prompt plus tool schemas) is 16.6k tokens in Claude
Code, 7.0k in OpenCode and 0.8k in mini-SWE-agent, and it is paid on every
step. Per-call tool returns are about equal across harnesses, and growth
per step is within 1.5×. The same model and tasks cost 1.81 / 0.97 / 0.56
CNY. On DeepSeek-V4-Flash the step counts diverge (104 / 141 / 88), and the
lightest preamble ends up most expensive (1.02 CNY), so step count can
outweigh preamble. 96–99% of input is cache hits. The cache price scales
the bill but does not reorder the harnesses. For a forecaster here, fit
preamble size and step-count distribution per harness × backend. A preamble
measured at session start is a known term, not a forecast.

## The downward bias holds on ML-agent wall-clock, and a declared timeout is enough to gate (2026-10-13)

[[literature/papers/wang2026clock]] answers this page's second open
question for one research-agent setting. Qwen3.6-27B on MLE-bench Lite was
stopped mid-run in 15-minute episodes and asked how long the whole run
would take. It underestimated by −735 ± 546 s without a clock and
−554 ± 538 s with one (n = 9–20 probes per cell). That is the same
direction as bai2026how, now in seconds on training-heavy trajectories,
and present-state perception did not remove it. Action durations also
vary from 30 s to 3 min under GPU and system load, so even an external
forecaster faces a noisy target.

The paper's working gate needs no forecast of cost. Its clamp hook
compares a shell command's *declared* timeout, or the harness default if
none is given, with the remaining budget, and lowers the timeout to fit.
That is an admission decision made from a meter plus a bound the action
carries with it. It extends the section above: admission needs a cost
estimate, unless the action can be made to carry a hard upper bound that
the harness enforces. The cost is lost work. A clamped training run ends
early, and quality fell slightly (46.8 vs 49.4 percentile, within noise).

## A completion-time cutoff calibrated on one model inverted on another (2026-11-03)

This concept asks for "measured history plus an explicit uncertainty
margin". [[literature/papers/jain2026monitor]] shows how narrowly the
history has to match. A restart cutoff is a forecast of spend-to-success:
it bets that runs that have not finished by call c will not finish soon.

- **Same tasks, same harness, other model.** On 7 SymPy issues with
  5 runs each, Luna finishes mostly by call 16, so a 16-call cutoff gains
  +2.29 pp [0.00, 6.86] in-sample. DeepSeek finishes mostly at calls
  21–30. The same cutoff takes it from 68.57% to 26.29%: −42.29 pp
  [−69.14, −17.71]. Its best cutoff is no cutoff.
- **Same model, other tasks.** Leave-one-task-out selection gives −7.31 pp
  on Luna. On 12 new tasks the 16-call cutoff ties (one cohort is solved
  by call 15 every time) or loses (0% vs 4%). Pooled: −0.21 pp
  [−4.21, 2.53].
- **No schedule escapes this (Prop. 3.3).** For any fixed cutoff there are
  completion-time distributions where it beats running to the budget and
  others where it loses. Luby schedules scored zero on DeepSeek, because
  no episode was long enough.

For this concept: a forecast used to cut work short must be calibrated on
the same model, harness and task population, and checked held out by
task. A model swap is a recalibration event. The paper notes that a
hosted model ID plus a date does not fully identify the runtime. This
matches the admission/allocation split above: the cutoff is a forecast
used mid-run, and it failed to transfer.

Limits:
- 7 tasks × 5 runs per model for the cross-model result. The interval is
  wide, though it excludes zero.
- Model calls are the budget unit; cost in tokens or dollars is not
  modelled.
- Fixed cutoffs only. No adaptive or learned schedule was tested.

## An agent's forecast of its own experiment outcomes was no better than a fresh reader of the registration, and the bias direction split by model (2026-11-03)

The 11-02 section above found outcomes forecastable at the task level, with
the limit "no agent self-forecast was tested". [[literature/papers/kanke2026rtaste]]
tests one. Six models each registered five small experiments with one
pre-stated event (a statistic against a threshold), then gave each a
probability before any ran. A fresh instance of the same model saw only the
seed paper and the five registrations (no code, no search, no reasoning)
and forecast the same events. Brier scores over 415 MVEs, lower is better:

| Forecaster | Brier |
|---|---|
| Base rate (other seed papers) | 0.240 |
| The proposing agent | 0.192 |
| Single fresh-reader judges | 0.179–0.216 |
| Mean of six fresh readers | 0.179 |

Agent minus same-model reader: −0.006 (95% CI −0.021 to 0.009).

What this adds:
- **The proposer's extra context bought no measurable forecasting edge.**
  Its own reading and search did not beat a reader who saw only the
  registration. This is the outcome analogue of the Definition: an
  external forecast is at least as good as the self-report, and a mean of
  several external ones is better.
- **The bias direction is model-specific, not uniformly optimistic.**
  GPT-6 Astra, GPT-6.1 Sol, GPT-6 Luna and Muse Spark 1.3 stated higher
  probabilities than their event rates. Claude Fable 5.1 and Opus 5.5
  stated lower ones. [[literature/papers/bai2026how]]'s uniform downward bias was about cost;
  for outcomes, a correction margin has to be fitted per model, like
  [[literature/papers/jain2026monitor]]'s cutoff.
- **A complete registration carries the forecast.** The paper suggests
  the null may be built in: each registration had to be complete enough
  for a separate executor. For `/propose`, that is a design rule. An
  `expected_metric:` written to that standard can be forecast by any cheap
  second reader.

Limits:
- One run per condition. The CI is across seven seed papers.
- Events are binary thresholds the proposer chose. Their occurrence rate
  depends on event form (CI-bound vs plain threshold), which differs by model.
- NLP experiments that run on one GPU with ≤8B models. No cost forecasts
  were elicited.
