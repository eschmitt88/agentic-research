---
kind: paper
title: "Monitor Accuracy Isn't the Answer: A Decision-Value Framework for Evaluating AI Agent Restart Monitors"
authors: ["Siddharth Jain", "Venkat N Vedam"]
institutions: ["Intuit"]
year: 2026
venue: "arXiv 2610.36544 (cs.AI), v1 2026-10-27, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.36544"
code_url: null    # the PDF names an analysis package ("RestartLab", App. E: sanitized traces, frozen protocols, replay and audit code, tests) but gives no URL
citations: null
source: "raw/papers/jain2026monitor.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/context-proprioception]]"
tags: ["restart-policy", "continue-vs-restart", "decision-value", "value-of-information", "stall-trigger", "abstention", "clopper-pearson", "simultaneous-intervals", "opportunity-ceiling", "fixed-cutoff", "cross-model-transfer", "context-flush", "failed-branch-reuse", "swe-bench-verified", "mini-swe-agent", "small-n", "negative-result"]
---

# Monitor Accuracy Isn't the Answer: A Decision-Value Framework for Evaluating AI Agent Restart Monitors

## TL;DR

The paper argues that a failure monitor should be scored by the value of
the action it triggers, not by its accuracy. For one clean restart, that
value depends on how often the alarm fires on would-be failures and
would-be successes, *and* on how often a fresh attempt succeeds with the
budget left (its "restartability" p). Accuracy never measures p.

It gives three tools: an exact alarm-to-action identity, a criterion for
which progress signals (or compressed contexts) keep the information a
decision needs, and a binomial audit over existing traces that returns
"restart", "continue" or "abstain".

The empirical part is small and tests **no monitor**. It is 194 runs of
mini-swe-agent on 19 hand-picked SymPy issues (two cheap hosted models,
$4.77 total), with fixed-time cutoffs and simple recovery tricks. Every
intervention comes out null or negative. The audit of 130 of those runs
abstains on all 103 live decisions. At five runs per cell that outcome
was fixed before any data came in (see Critique).

## Claims

- **Accuracy does not set restart value (Prop. 4.1).** Let q be the chance
  that continuing succeeds, p the chance a clean restart succeeds in the
  residual budget, τ the alarm's true-positive rate on failures and φ its
  false-positive rate on successes. The success change from restarting on
  alarm is G = (1 − q)τp − qφ(1 − p). With q = 0.5, τ = 0.9, φ = 0.1 the
  monitor is 90% accurate for every p, and G = 0.5p − 0.05: −5 pp at
  p = 0, 0 at p = 0.1, +35 pp at p = 0.8. This is an analytical example,
  not a measured monitor.
- **Opportunity ceiling (Cor. 4.2).** No alarm can gain more than
  (1 − q)p. If restarts rarely succeed in the residual budget, even a
  perfect failure predictor adds nothing.
- **Regret is advantage-weighted (Prop. 3.2).** A stopping rule's regret is
  E[|Δ| · 1{wrong action}], where Δ is continue-minus-restart value. A
  detector can be accurate where the two actions are near-tied and still
  lose most of the value where |Δ| is large.
- **No fixed cutoff dominates (Prop. 3.3, Cor. 3.4).** For any budget and
  cutoff there is a completion-time distribution where the cutoff loses to
  never restarting, and one where it wins. A finite schedule with longest
  episode s fails outright if every completion takes longer than s.
- **The trace can be aliased (Prop. 3.5, Cor. 3.6).** Two regimes
  ("late but viable" and "fast or never") can produce the same no-success
  history yet need opposite actions. Any rule reading only that history
  has worst-case regret ≥ p/(1 + p).
- **Decision-sufficient compression (Prop. 4.3).** Coarsening a progress
  signal Z to G = g(Z) loses value ½{E|D(Z)| − E|E[D(Z) | G]|}, where D is
  restart advantage. Nothing is lost if each coarse cell holds advantages
  of one sign. So a compressed context should keep the distinctions that
  change the preferred action, not every token. This is a criterion; the
  paper builds no compressor.
- **A certificate with abstention (Prop. 4.4).** Per task j and decision
  time t, estimate continuation success q̂ from attempts still live at t
  and fresh success p̂ from all attempts within B − t. Put a two-sided
  Clopper–Pearson interval at error α/(2M) on each, for M comparisons.
  Return "restart" if L_R > U_C, "continue" if U_R < L_C, else "abstain".
  Under IID attempts within a task, every non-abstaining call has the right
  sign with probability ≥ 1 − α. Abstaining does not certify continuing.

## Methods

- **Runtime (Table 3).** mini-swe-agent 2.4.6; `openai/gpt-5.6-luna` for
  everything, plus `deepseek/deepseek-v4-flash-0731` in the first cohort
  only. Temperature 0.7. The budget unit is model calls (horizons 12, 30
  or 60). Network-off Bubblewrap sandbox. Collected 2026-08-31 and
  2026-09-06.
- **Verification.** The hidden fail-to-pass tests run outside the
  sandbox after every environment action, so the first passing call is
  recorded exactly. Each task is gated before use: the start state must
  fail and the reference patch must pass. Every successful diff was
  checked for tampering with the verifier; none was found.
- **The ladder (Table 2, App. B).** Six stages, each frozen before its
  data. Later stages were chosen after earlier results.

  | Stage | Model runs | Tasks | Comparison |
  |---|---|---|---|
  | Initial restart replay | 70 | 7 | 5 per task × 2 models; cutoffs 4–30, geometric, Luby |
  | Prospective transfer | 60 | 12 | Luna only; fixed 16 vs fixed 30 |
  | Horizon extension | 20 | 4 | one 60-call vs repeated 30 |
  | Context rejuvenation | 40 | 4 | baseline vs fresh context, 5 + 5 per task |
  | Patch algebra | 0 | 4 | 19 verifier-only composites of failed patches |
  | Semantic resolver | 4 | 4 | one 12-call run per task |

  The 19 tasks overlap across stages. The four "stress" tasks
  (16597, 17630, 21612, 21930) carry the last four stages.
- **Restart replay.** Plug-in success for repeated cutoff c under total
  budget 30, from each task's empirical completion-time CDF over its five
  runs (Eq. 20). Unused calls are released after an explicit protocol
  failure. Cutoff selection is in-sample and leave-one-task-out.
- **Decision audit (§5.8).** Retrospective, designed after seeing the
  data, not preregistered. 130 Luna and DeepSeek 30-call runs form 26
  task-model cells of five runs each. Decision times t ∈ {4, 8, …, 24}
  give M = 156 comparisons, α = 0.05. The action is one clean restart with
  the residual budget, not the repeated-cutoff policy of the replay.
- **Statistics.** Task-then-attempt bootstrap, 10,000 replicates, for
  policy contrasts. Stage intervals are not adjusted for multiplicity;
  only the audit uses the family allocation.

## Results

- **Initial cohort (Table 5).** Luna solves 29/35, mostly by call 16.
  DeepSeek solves 24/35, mostly late (21–30 calls).
- **The in-sample gain is one trajectory (§6.3, Eq. 22).** Luna's best
  cutoff (16) gives 85.14% vs 82.86% for one 30-call run: +2.29 pp
  [0.00, 6.86], bootstrap positive fraction 0.504. All of it is task 23413,
  where one of five runs was still live at call 16 and 4/5 fresh runs
  finish in 14 calls: (1/7)(1/5)(4/5) = 2.2857 pp. Leave-one-task-out
  selection gives 75.54%, −7.31 pp vs the baseline.
- **The cutoff inverts across models.** The same 16-call cutoff on
  DeepSeek: 68.57% → 26.29%, −42.29 pp [−69.14, −17.71], on the same 7
  tasks × 5 runs. DeepSeek's best cutoff is 30, the no-restart policy.
  Luby schedules give 45.72–67.40% on Luna and zero on DeepSeek; none has
  an episode long enough for DeepSeek's completions.
- **No prospective replication (Table 6).** The high-ID cohort (7 tasks,
  35 runs) is solved by call 15 in every run: 100% vs 100%, no live
  decision. The stress cohort (5 tasks, 25 runs) has one success, at call
  23: 0% vs 4%. Pooled over 19 tasks: 68.21% vs 68.42%, −0.21 pp
  [−4.21, 2.53].
- **The audit abstains everywhere.** 103 of 156 cells have a live run;
  none is certified. At call 16, 13 cells are live. All six live DeepSeek
  cells favour continuing on the point estimate. Under the plug-in
  perfect alarm, the prospective cohorts and initial DeepSeek gain 0.00
  (Table 7): there was nothing to recover.
- **Longer horizon (Table 8).** On four tasks that were 0/5 at 30 calls,
  20 runs at 60 calls give one pass, at call 17, and none after call 30
  (one-sided upper bound 13.9%, unclustered). 17 of 20 end in repeated
  format errors, against 1 of 20 at horizon 30 (Fisher p = 4.06e−7,
  descriptive). Repeated 30 scores 9% vs 5% for one 60-call run, and
  "recycle the budget after an explicit failure" also scores 9%. So the
  only restart gain comes from reusing calls after a visible protocol
  failure, not from cutting a live run.
- **Fresh context (Table 9).** After three consecutive invalid tool
  responses, the treatment keeps the repository, drops the conversation,
  opens a new client and supplies the issue plus `git status` and a
  diff-stat. It fired in 18/20 runs. Success 0/20 vs 1/20 at baseline.
  Format-error terminals fall from 90% to 0%; limit terminals rise from 5%
  to 100%. Mean cost +56.7% ($0.0379 → $0.0593), wall time +65.7%. The
  frozen bar (+5 pp or +2 successes, ≤ +20% cost) is not met.
- **Failed-branch reuse (Table 10).** 38 failed patches give 51 clean
  novel pairwise composites; the 19 chosen blind all fail. Four resolver
  runs that saw every failed patch fail too, though their tests reach
  76/79, 21/22, 99/100 and 47/52.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in
  the digest entry is in the PDF and quoted correctly, and the entry
  already flags small n and the missing replication. It misses three
  things.
  1. **No monitor is evaluated.** The title and digest say "restart
     monitors". The empirical part tests fixed-time cutoffs, schedules, a
     protocol-failure trigger and patch reuse. The paper says so (§1,
     §7.3: "The empirical audit uses task identity and elapsed calls, not
     rich trace features"). The −42.29 pp is a 16-call restart schedule
     moved from one model to another, not a monitor's loss.
  2. **"0 of 103 certified" is arithmetic, not a finding.** With the
     paper's allocation (tail error α/(4M)), an arm with 5/5 successes has
     lower bound 0.152 and an arm with 0/5 has upper bound 0.848 at
     M = 156. Even perfect separation cannot certify. It needs at least 14
     runs per arm at M = 156, 12 at M = 26, and 7 for a single decision
     (our computation from Eq. 17). The continuation arm is smaller still,
     since it holds only runs still live at t (often 1–5). The paper says
     the cells are "too sparse for this conservative task-specific rule"
     (§7.2), but the abstract lists the zero as a result.
  3. **The real n.** 194 runs counts executions, not independent
     evidence. The audit is 130 of them in 26 cells of 5. The headline
     gain rests on one run. The cross-model result is 7 tasks × 5 runs
     per model. The four stress tasks carry four of the six stages.
- **Is the audit a usable template for a stall trigger?** Partly.
  - *Usable:* the identity and the ceiling. Before judging any trigger,
    estimate q (does the default action still succeed from the stall
    point?) and p (does the alternative action succeed in the budget
    left?). If (1 − q)p is near zero, no trigger can help, however
    accurate. Score triggers by advantage-weighted regret, not hit rate.
    Report "abstain" as its own outcome, and never read it as "continue
    is right".
  - *Not usable as a per-decision gate here:* the certificate needs IID
    replicate attempts at the same task and decision time, in both arms.
    An `/iterate` chain is one trajectory per project, and each cycle is
    a training run. Even pooled across projects, cells would hold a few
    runs, below the 7-per-arm floor for a single decision. The identity
    also assumes a clean restart independent of the current attempt. A
    redirect that keeps the workspace breaks that assumption, and the
    paper says the general form G = E[A(Y_R − Y_C)] then needs outcomes
    for the recovery action itself.
  - Our inference: the closest cheap estimate is observational. Past
    `/iterate` logs give q directly (the share of cycles with counter at
    ceiling − 1 that improved). The `redirect=<axis>` field in the
    redirect-before-halt proposal would give p over time.
- **For redirect-before-halt
  (`docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md`).**
  Still no redirect-vs-halt arm. The nearest arm is the fresh-context
  intervention, a failure-triggered flush that keeps the workspace. It is
  null at 0/20 vs 1/20 and costs 57% more. That is a counterpoint to
  [[literature/papers/cai2026forkandflush]], where a scheduled flush
  helped on average. But it ran on the four tasks the baseline almost
  never solves, so p was near zero by selection, and Cor. 4.2 says
  nothing could help there.
- **For the noise band
  (`docs/system-proposals/2026-09-27-iterate-no-improvement-noise-band.md`).**
  A simultaneous-interval rule at small n never acts. That is the same
  pattern as the significance-cut keep rules in
  [[literature/papers/hu2026winners]] and [[literature/papers/zhou2026does]].
  It supports the accepted scope: a band on the stop counter, not a
  significance gate on each keep or pivot.
- **For the known-bad preflight
  (`docs/system-proposals/2026-10-25-hce-known-bad-preflight.md`).** The
  task gate is the proposal plus its reviewers' known-good check: the
  start state must fail the evaluator and the reference patch must pass.
  One more attestation, in a small study. Separately, the opportunity
  ceiling works as a preflight for an *intervention*: check that the
  alternative action can succeed before measuring a trigger.
- **Other limits.**
  - Two cheap hosted models, one harness, SymPy only; "Luna" and "flash"
    tiers cost about a cent per run. Hosted model behaviour can drift
    under a fixed ID.
  - The theory is elementary and the paper says so: Blackwell's comparison
    of experiments and Clopper–Pearson intervals, specialised to two
    actions.
  - The independence assumption behind Prop. 4.1 fails for any restart
    that keeps edits, which is the recovery type that did pay in
    FailFast–RestartSmart (as cited in
    [[literature/papers/srinivasan2026continue]]).
- **Open questions.**
  - What is p for a state-keeping pivot at ceiling − 1 in an ML research
    chain? Nothing in the graph measures it.
  - Does a hierarchical (pooled) version of the certificate resolve
    decisions at realistic n, and how would it be validated? The paper
    names this and does not try it.

## Trust signals

- **Credibility:** 3.
  - For:
    - The claims are tightly scoped and the paper states its own limits
      (three evidence levels kept apart, §7.1; threats appendix).
    - Every stage was frozen before its data. Nulls are kept, not
      dropped.
    - Verification is external and per action, with tamper audits and
      base-fail / reference-pass task gates.
    - The arithmetic checks: Eq. 22 gives 2.2857 pp; Fig. 2B's counts
      (20/42/86, 35/73/149, 43/90/184) match Eq. 18; the 13.9% bound and
      the Fisher p reproduce (checked at ingest).
  - Against:
    - Preprint, not peer reviewed. Two authors at Intuit, not a group
      known for this line of work.
    - Very small n. The analysis package is described but not linked.
    - The audit is post hoc. Stage intervals are not multiplicity
      adjusted. Two models, 19 selected tasks.
  - Not 4: n is tiny and no code link is given. Not 2: the stated claims
    stay inside what the data and the (elementary) proofs support.

## Follow-up

- **Relevance:** 4. It is the first source in this graph to give a
  formal yardstick for a stall trigger: score it by the value of the
  action it fires, bound that value by (1 − q)p before testing, and
  report abstention separately. It also adds one failure-triggered
  same-model flush, which is null. It qualifies four concepts:
  - [[concepts/budget-as-ceiling]]: a pivot's value is set by its own
    success chance in the remaining budget, not by how well the counter
    detects stalls.
  - [[concepts/hce-evaluation]]: the decision audit, its per-arm sample
    floor, and "a protocol label moved, the outcome did not".
  - [[concepts/spend-forecast-calibration]]: a completion-time cutoff
    calibrated on one model inverts on another.
  - [[concepts/context-eviction-policy]]: a failure-triggered flush
    replaced the failure label and not the failure.

  Not 5: the evidence is a framework plus a tiny empirical study. No
  monitor and no redirect-vs-halt arm is tested.
- **Pairs with** [[literature/papers/srinivasan2026continue]]. That paper
  has the n (147k traces) and finds no within-task signal at 20k tokens.
  This one explains why that alone does not settle the restart question
  (p matters too) and why the trace may be unable to settle it (aliasing,
  Prop. 3.5). Both find continuing beats cold restart in their settings.
- **Cited and in the graph:** [[literature/papers/park2026when]] (external
  grounding of progress), [[literature/papers/li2026acm]] (agentic context
  management).
- **Candidates** (cited, not yet in the graph):
  - Wang, Lyu et al. 2026, "Fail-fast, restart-smart" (arXiv 2608.03222).
    The PDF lists Chenyu Wang as first author;
    srinivasan2026continue's note cites it as "Lyu et al.".
  - Guo et al. 2026, "EET: Experience-driven early termination for
    cost-efficient software engineering agents" (arXiv 2601.05777).
  - Shrivastava 2026, "Semantic early-stopping for iterative LLM agent
    loops" (arXiv 2606.27009).
