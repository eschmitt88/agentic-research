---
kind: paper
title: "Good Judgment, Bad Advice: Decision-Model Controllers Raise Agent Cost Without Improving Success"
authors: ["Naman Vats"]
institutions: ["Birla Institute of Technology, Mesra (email domain only)"]    # PDF gives only a bitmesra.ac.in address; no affiliation line, no funding or COI statement
year: 2026
venue: "arXiv 2610.24649 (cs.AI), v1 2026-10-17, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.24649"
code_url: "https://github.com/namanvats/decision-model-controllers"
citations: null
source: "raw/papers/vats2026good.pdf"
added: "2026-10-26"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/spend-forecast-calibration]]"
tags: ["controller", "decision-model", "jev", "typesafe", "placebo-control", "advisory-intervention", "verify-nudge", "abandon", "stop-decision", "change-approach", "calibration", "auroc", "offline-vs-online", "cost-per-attempt", "terminal-bench", "harbor", "sonnet-5.5", "haiku-5.5", "pre-specified", "negative-result", "small-n"]
---

# Good Judgment, Bad Advice: Decision-Model Controllers Raise Agent Cost Without Improving Success

## TL;DR

A single-author, pre-specified study. A Claude Sonnet 5.5 terminal agent
runs on Terminal-Bench 2.1 in one fixed loop, and the only thing that
changes is the controller consulted at four deterministic decision points.
Arms: no controller (native), deterministic rules, Claude Haiku 5.5, the
typed decision model Jev 1.13 (TypeSafe), and a placebo that samples Jev's
development-phase choice frequencies. The controller can only append one of
three fixed advisory messages (investigate / recover / verify), end the run
as a failure (abandon), or allow a finish. The evaluation is 21 held-out
tasks × 5 arms × 3 attempts = 315 trials, and the whole study cost $72.

Three results matter for this graph:

1. **Every informed controller raised cost and none raised the pass rate.**
   Cost ratios against native (geometric mean of per-task ratios): rules
   ×1.24 [1.12, 1.38], Haiku ×1.47 [1.16, 1.86], Jev ×1.19 [1.02, 1.36].
   The placebo was ×0.98 [0.78, 1.16]. Pass rates were 73.0 / 73.0 / 76.2 /
   73.0 / 71.4%. Jev tied rules on cost (0.96 [0.83, 1.11]). So the headline
   is about *advisory control of a strong agent*, not about Jev in
   particular.
2. **The extra cost tracks how often the agent is told to verify.** Verify
   messages per attempt and cost ratio rise together across arms: placebo
   0.33 → 0.98, Jev 0.51 → 1.19, rules 0.92 → 1.24, Haiku 1.30 → 1.47. The
   agent's own finish requests were already right 87% of the time.
   Offline, Jev ranked finish requests best (AUROC 0.78 vs 0.62 rules, 0.60
   Haiku) but put mean P(finish) at 0.50, so it chose verify on 29 of 53
   finish requests. Even the runs it doubted most passed 79% of the time.
3. **No controller ever chose to abandon a run in 315 trials,** including
   the rules arm's deterministic stop rule. Caps and timeouts did all the
   stopping. 10 of native's 17 failures ended at a timeout or cap with no
   finish request, and all 15 runs of one hopeless task ran to the limit.

## Claims

- "The hypothesis that Jev lowers cost is rejected." Jev raised cost per
  attempt by 19% at an identical 73.0% pass rate (confirmatory C1).
- The placebo "matched every informed controller's pass rate without their
  cost". From this the paper concludes "Jev's choices, not the act of
  intervening, add the cost."
- The judgment was informative but the policy was not: "The gap between an
  informative signal and a useful intervention is the policy in between."
  It recommends base-rate-aware thresholds and stopping actions the
  controller will actually use.
- Overhead is not the problem. Jev averaged 0.52 s and 0.11% of spend per
  run (Haiku 1.46 s, 0.47%); one decision cost about $0.00008.
- Controllers should be "evaluated end to end on cost, against rules and a
  placebo, not only on judgment accuracy."
- Two harness lessons: a naive "did the agent verify?" feature misses
  agents that check their work by running their own programs, and a safety
  refusal returns an empty reply that a loop can mistake for a finish
  request.

## Methods

- **Loop (Fig. 1).** Each step is one Sonnet 5.5 call (reasoning effort
  high, prompt caching, no temperature control) that may call `bash` or
  `finish`. A reply with no tool call counts as a finish request. Commands
  run via Harbor 0.24.0 in a fresh shell in a Daytona sandbox built from the
  task image. Output is truncated to the first 4,000 and last 8,000
  characters, and the context is never compacted.
- **Triggers (logged in every arm).** T1: ≥2 consecutive failing commands.
  T2: the same normalized command twice in the last 5, or the same output
  twice in the last 3. T3: a finish request. T4: every 10 steps. At most one
  trigger fires per step, and a trigger type has a 3-step cooldown.
- **Questions.** At T1/T2/T4, Q-progress: continue / investigate / recover /
  abandon. At T3, Q-finish: finish / verify. The guidance messages never
  contain commands, paths or test information (App. A). "Recover" is
  "Step back, identify why it is failing, and try a different approach." The
  controller sees a compact state built from the transcript (≤24k chars). It
  never sees tests, verifier output or reference solutions.
- **Controllers.**
  - Rules: recover at every T1/T2, continue at T4, verify at T3 unless a
    verification-like command ran after the last edit, and abandon after
    four consecutive failures once 40% of the step budget is used.
  - Haiku 5.5: reasoning off, JSON-schema single choice.
  - Jev: `typesafe/jev-1.13`, served via OpenRouter's "System One"
    endpoint.
  - Placebo: samples Q-progress at continue 0.625 / investigate 0.125 /
    recover 0.25 / abandon 0, and Q-finish at finish 0.652 / verify 0.348,
    seeded per trial. These are Jev's frequencies on the 8 development
    tasks (8 progress and 23 finish decisions).
- **Guards override every controller.**
  - G1: hard caps (60 steps, $1.00, 30M tokens, 4 h, 30 controller calls).
  - G2: abandon is allowed only at ≥40% of steps with a failure streak or
    repeated no-progress; otherwise it becomes recover.
  - G3: the third finish request is accepted after two verifies.
  - G4: a fallback on timeout or malformed output.
- **Protocol.** Terminal-Bench 2.1 has 89 tasks, 86 eligible. 30 tasks were
  calibrated with native ×2 and stratified, giving 8 development tasks and
  21 held-out tasks (14 easy, 3 mid, 4 hard). The held-out list was
  checksummed and configurations fingerprinted. Arm order was randomized
  within each task–attempt block.
- **Statistics.** Paired by task, 10,000 task bootstraps. The cost effect is
  the geometric mean of per-task USD/attempt ratios. C1 (Jev vs native)
  tests cost and checks pass rate against a −10 pp noninferiority margin
  (one-sided 95%). C2–C4 are Holm-adjusted. The offline analysis (4.3) and
  the cost decomposition (4.4) are exploratory.
- **Offline replay (Table 3).** Every trigger state logged by native was
  replayed to each controller without acting on the answer, and labelled
  with that trial's outcome: 53 finish states and 72 progress states.
- **Deviations (App. D), all disclosed:**
  - 21 held-out tasks instead of the planned 30–40.
  - Caps raised above every calibration run, because p90 caps would have
    acted as an abandon policy.
  - No provider pinning (94.6% AWS / 5.4% Azure) and no temperature 0.
  - The verification feature was widened before development.
  - The open-weight replication (C5) was dropped.

## Results

- **Per arm (Table 1, 63 attempts each).**

  | Arm | Pass % | USD/attempt | USD/solved | Steps | Decisions | Verify |
  |---|---|---|---|---|---|---|
  | native | 73.0 | 0.177 | 0.243 | 13.3 | – | – |
  | rules | 73.0 | 0.191 | 0.262 | 15.6 | 3.14 | 0.92 |
  | Haiku 5.5 | 76.2 | 0.213 | 0.280 | 17.8 | 3.65 | 1.30 |
  | Jev | 73.0 | 0.206 | 0.283 | 16.0 | 2.79 | 0.51 |
  | placebo | 71.4 | 0.156 | 0.218 | 13.2 | 2.32 | 0.33 |

- **Paired contrasts (Table 2).**

  | | Δ pass (pp) | Cost ratio | p |
  |---|---|---|---|
  | C1 Jev vs native | +0.0 [−11.1, +12.7] | 1.19 [1.02, 1.36] | 0.026 |
  | C2 Jev vs Haiku | −3.2 [−11.1, +4.8] | 0.81 [0.68, 0.95] | 0.019 |
  | C3 Jev vs rules | +0.0 [−7.9, +9.5] | 0.96 [0.83, 1.11] | 0.623 |
  | C4 Jev vs placebo | +1.6 [−4.8, +9.5] | 1.22 [1.08, 1.39] | 0.001 |

  All Holm-adjusted pass-rate p-values are 1.00. The C1 one-sided lower
  bound is −9.5 pp, just inside the −10 pp margin. The paper reads this as
  "no evidence of a large drop rather than as noninferiority."
- **Spread.** Jev cost more than native on 15 of 21 tasks.
  Leave-one-task-out keeps the ratio within 1.16–1.25. By stratum the ratio
  was 1.29 (easy), 1.48 (mid) and 0.76 (hard). The hard-task savings came
  from runs that hit the timeout or cap, where there is no finish request
  and so nothing to verify.
- **Identical pass rate, different tasks (App. C).** Jev and native differ
  on three tasks:
  - install-windows-3.11: 0/3 → 3/3 (Haiku also 3/3, rules 1/3);
  - filter-js-from-html: 3/3 → 1/3 (Haiku 0/3);
  - caffe-cifar-10: 1/3 → 0/3.

  The tie is a net of swaps, not of no effect.
- **Cost decomposition (exploratory).** Jev added $0.029 per attempt. Of
  that, $0.011 was spent after the first verify message. The rest came from
  longer trajectories (16.0 vs 13.3 steps), which fits Jev choosing
  investigate or recover at 30 of 92 progress decisions. The paper calls
  this part correlational.
- **Offline (Table 3).**
  - Q-finish (n = 53; base rate 87% pass): AUROC/ECE/accuracy were Jev
    0.78/0.37/0.55, rules 0.62/0.55/0.45, Haiku 0.60/0.70/0.30.
    Runs with Jev's P(finish) ≥ 0.5 passed 23 of 24; runs below 0.5 passed
    23 of 29.
  - Q-progress (n = 72): Jev's P(abandon) AUROC was 0.55. Rules and Haiku
    were 0.50, since they never abandon.
  - The 111 calibration states replicate the finish ranking (Jev 0.77,
    Haiku 0.46).
- **Abandon.** It was never chosen in 315 trials, so G2 never blocked one.
  make-doom-for-mips failed all 15 runs, 11 at the Harbor timeout and 4 at
  the $1.00 cap.

## Critique / open questions

- **Against the digest framing: BELOW (mild).** Every number in the digest
  entry is in the paper and stated correctly. The "Why" reads past the
  paper in three places:
  1. **"The closest design yet to the redirect arm."** The change-approach
     action is advisory prompt text ("try a different approach") appended
     to the transcript. The trigger is owned by the controller, but the
     action is not. Nothing forces an axis change, and no redirect or halt
     was ever executed because abandon never fired. It is a trigger-owned
     *advisory* arm, which is the form yan2026traceml and
     cai2026forkandflush already found inert. It is not the forced-pivot
     arm the redirect-before-halt proposal needs.
  2. **"Without improving success" is a power statement.** The pass-rate
     intervals are ±11–13 pp. 14 of the 21 tasks are easy and pass about
     3/3 in every arm, and Jev and native differ on only three tasks. By
     [[literature/papers/liu2026what]]'s arithmetic, nothing under about
     55 paired tasks is decision-grade. Haiku's +3.2 pp and
     install-windows' 0/3 → 3/3 cannot be judged either way. The cost
     result is solid (15/21 tasks, leave-one-out 1.16–1.25). The pass
     result is not, and the paper says so.
  3. **The title singles out decision models, but the rules arm did
     worse.** Deterministic rules cost ×1.24, with an interval that
     excludes 1, and tied Jev (C3). What the paper shows is that
     intervening on this agent costs money. It does not show that learned
     controllers are worse than rules.
- **Check-at-ingest answers.**
  - *Power:* see point 2. The C1 guardrail sits at the design's resolution.
  - *Did rules fire "change approach"?* Yes, by construction: the rules arm
    answers recover at every T1/T2. The paper never reports the count or
    what followed. It decomposes cost only for Jev. Rules: 73.0% pass,
    $0.191/attempt, ×1.24 cost, 15.6 steps, 0.92 verifies per attempt.
    Rules' own abandon rule (4 consecutive failures at ≥40% of steps) never
    met its condition.
  - *Affiliation with Jev's maker:* none found. The PDF gives only a
    bitmesra.ac.in email and no funding or conflict statement. The repo
    README never mentions TypeSafe and credits no API access. A web search
    for the author with TypeSafe/Jev returned nothing. An unverified
    LinkedIn profile puts the author at Sentient Labs, Bengaluru. Jev was
    reached through OpenRouter, not a vendor channel, and the result goes
    against the vendor's product.
- **The placebo controls content, not dose, and the doses differ.** The
  placebo samples Jev's *development* frequencies (verify 0.348, from 23
  finish decisions). At evaluation, Jev chose verify more often: 29 of 53
  finish states offline, and 0.51 vs 0.33 verifies per attempt online.
  The placebo also made fewer decisions (2.32 vs 2.79 per attempt). So the
  C4 gap (1.22) mixes "Jev's choices" with "Jev intervened more". The
  paper's own Limitations say the placebo "tests decision content, not
  intervention frequency". Its Discussion sentence attributing the cost to
  Jev's choices goes further than that. Across all five arms, cost rises
  monotonically with verify dose. The simplest reading of Table 1 is a dose
  effect, with little left for content (our inference). A yoked placebo,
  firing the same number of interventions at the same moments in a paired
  run, would separate the two.
- **AUROC 0.78 rests on about 7 failures.** 87% of 53 finish states passed,
  which leaves roughly 7 failing runs. No interval is given. The 111-state
  calibration replication (0.77) helps, but those states predate the final
  verification feature. Labels are factual trial outcomes, not the
  counterfactual outcome of the other decision. The paper notes this.
- **The verify content is generic.** The verify message asks the agent to
  "verify each explicit requirement". It names nothing that is missing.
  [[literature/papers/agarwal2026fire]] found on the same benchmark that a
  generic refusal is worth about zero and a specific one is what helps. This
  paper adds the bill for the generic form on a strong agent. The three arms
  that used it at 0.5–1.3 verifies per attempt cost 19–47% more per attempt
  for no detectable gain, though their progress messages are bundled into
  that figure.
- **Q-progress accuracy is unclear.** Rules, Haiku and Jev all score 0.36
  against a 0.64 base rate. That fits a convention where any non-continue
  answer counts as predicting failure, but the paper does not define it.
- **What it does not show.**
  - An ML-research loop. These are terminal tasks with about 13-step runs
    and a 60-step cap, far from `/iterate` cycles.
  - A strong controller action. Only advice and an abandon that was never
    used are available.
  - Any weaker agent. On an agent with a lower finish-request precision,
    verification could pay. The 87% base rate drives the result.
  - Thresholded policies. Base-rate-aware thresholds are proposed and not
    run.
- **Jev elsewhere in the graph.** In [[literature/papers/zhang2026review]],
  JEV as a closed judge choosing between two answers came out +4.1 above
  the stronger model for cents per 1,000 questions. In
  [[literature/papers/giampieri2026cortex]], a Jev relevance judge
  replayed offline did not separate kept from killed items. Here it ranks
  well and is miscalibrated. Taken together, Jev's scores order outcomes
  reasonably, and its argmax is not a policy.

## Trust signals

- **Credibility:** 3.
  - For:
    - A pre-specified design with a checksummed held-out list, fingerprinted
      configs and randomized arm order.
    - A deviations table (App. D), and a limitations section that
      downgrades its own pass-rate result.
    - Placebo and rules baselines, paired task bootstrap, Holm correction,
      leave-one-out.
    - Per-task results in App. C.
    - Harness, configs, prompts and analysis scripts released under MIT.
      Result files were not visible in the repo listing (checked
      2026-10-26).
    - Cheap to reproduce ($72 total).
    - A negative result against a commercial product, with no apparent
      vendor tie.
  - Against:
    - Single author with no affiliation stated beyond an email domain.
    - Not peer reviewed. The repo shows an AES Symposium submission branch.
    - 21 tasks, 14 of them at ceiling.
    - One agent, one decision model, unpinned hosts, no temperature
      control.
    - The plan is self-held, not externally registered.
    - The placebo's dose does not match the evaluation-time dose.
  - The same author's "The scaffold effect in coding agents" (Vats & Golev,
    arXiv 2607.22585) is a cited candidate in
    [[literature/papers/liu2026what]].
  - Not 4: no institutional or peer-review signal, and a small effective n
    on pass rate. Not 2: the design and its disclosures are better than
    most lab preprints in this graph, and the code is released.

## Follow-up

- **Relevance:** 4. It is the first placebo-controlled, end-to-end cost test
  in this graph of a cheap controller over a strong agent loop. It adds
  material evidence to four concepts:
  - [[concepts/hybrid-model-backends]]: a structured specialist on a closed
    decision was cheaper than a small LLM and ranked better, and still lost
    to no controller. Its cost is the main-model steps its advice causes,
    not its own price.
  - [[concepts/evidence-gated-completion]]: generic verify nudges on an
    agent that is 87% right at finish are pure cost.
  - [[concepts/budget-as-ceiling]]: no learned or rule stop fired, and the
    caps did all the stopping. An advisory change-approach on stall
    triggers bought nothing measurable.
  - [[concepts/hce-evaluation]]: offline AUROC did not predict online
    value, and the controller needs a placebo arm at matched dose.

  Not 5: terminal tasks, not an ML-research loop; there is no forced
  redirect or halt arm; and the pass-rate evidence is weak.
- **For the redirect-before-halt proposal.** This is evidence about the
  *action*, not the trigger. A deterministic, controller-owned stall trigger
  attached to prompt-level "try a different approach" text (the rules arm)
  bought nothing measurable and cost more. The proposal's forced axis change
  remains untested. Neither redirect nor halt was ever executed here, so
  there is still no redirect-vs-halt arm.
- **For [[concepts/spend-forecast-calibration]].** A controller's line item
  (0.11% of spend) says nothing about its bill. The bill is the induced
  main-model steps (+2.7 steps per attempt). Price a controller end to end.
- **Candidates** (cited, not yet in the graph):
  - Vats & Golev 2026, "The scaffold effect in coding agents" (arXiv
    2607.22585). Already listed under liu2026what.
  - Huang et al. 2024, "Large language models cannot self-correct reasoning
    yet" (ICLR 2024). This is the prior for a verify nudge without new
    external information.
  - Kadavath et al. 2022, "Language models (mostly) know what they know"
    (arXiv 2207.05221).
