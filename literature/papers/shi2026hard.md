---
kind: paper
title: "Hard Issue, Weak Model, or Bad Run? Rethinking Failure Prediction for Coding Agents"
authors: ["Xingyue Shi", "Wesley K. G. Assunção"]
institutions: ["North Carolina State University"]
year: 2026
venue: "arXiv 2611.08285 (cs.SE), v1 2026-11-05, preprint (4-page IEEE short-paper format: 'Preliminary findings', 'Future plans'; no venue named)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.08285"
code_url: null    # no code or analysis scripts linked; data are public on Hugging Face (tarsur385/swebench-verified-trajectories, tarsur385/swev-trm-trajectories-25models, nebius/SWE-agent-trajectories)
citations: null
source: "raw/papers/shi2026hard.pdf"
added: "2026-11-16"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/pass-at-k]]"
tags: ["failure-prediction", "trajectory-monitor", "run-blind-baseline", "issue-difficulty", "model-ability", "bad-run", "item-response-theory", "rasch", "2pl", "oracle-gap", "no-specialization-reference", "model-routing", "model-switching", "escalation", "retry", "within-issue-auroc", "pooled-auroc", "conditional-logit", "tost-equivalence", "repeated-runs", "run-to-run-variance", "stall-trigger", "swe-bench-verified", "mini-swe-agent", "negative-result"]
---

# Hard Issue, Weak Model, or Bad Run? Rethinking Failure Prediction for Coding Agents

## TL;DR

A four-page "preliminary findings" paper from the same NC State group as
[[literature/papers/srinivasan2026continue]]. It asks what trajectory-based
failure predictors for coding agents actually predict. A run can fail
because the issue is hard, because the model is weak, or because this
attempt went wrong (a *bad run*). Each cause calls for a different action:
stop or ask a human, switch model, or retry. On 3,492 mini-swe-agent runs
of seven models on SWE-bench Verified (one run per model per issue):

1. **Pooled AUROC is difficulty.** A run-blind difficulty score (the share
   of 16 *other* models that fail the issue) reaches 0.924 pooled AUROC.
   TF-IDF trajectory predictors reach 0.624–0.630, no better than prefix
   length plus model identity (0.607).
2. **Within-issue AUROC is model ability.** Model identity alone gets 0.724
   within an issue; the predictors get 0.604–0.610, and on "upset" pairs
   (the weaker model won) they score 0.42, *below* chance. Adding any of 18
   predictor configurations to difficulty + model changes AUROC by an
   amount equivalent to zero at ±0.02.
3. **The oracle gap is within chance.** Best single model 76.8%, any-model
   oracle 86.8% (10.0 pp). That gap and the upset rate both sit inside the
   range a Rasch/2PL world with *no* per-issue specialization produces.
   Choosing a model per issue gains +0.09 pp on held-out runs; a pre-run
   router +0.41 pp, both CIs spanning zero.
4. **Bad runs are common, but alarms do not find them.** In a repeated-run
   pilot (two Llama models), run-to-run variation is 42% of outcome
   variance. 64–82% of predictor-score variance is between issues, so an
   alarm mostly flags hard issues, where a retry rarely helps.

No stopping, retry or switching policy is run live or replayed under a
budget. "Closed-loop, equal-budget" evaluation is named as future work.

## Claims

- A high pooled AUROC does not show a predictor learned anything about the
  run. In a toy world where success depends only on issue and model, a
  hardness-only score gets 0.90 pooled and 0.50 within issue.
- An oracle gap does not show issue-specific strengths. Toy world: model A
  at 0.7 and B at 0.6 on every issue gives an any-model oracle of 0.88,
  but running A twice gives 0.91. The gap comes from the second attempt.
- An alarm need not mark a bad run. A difficulty score flags failures on
  hard issues (retry succeeds 10% in the toy) and misses the ones on easy
  issues (retry succeeds 90%).
- Real agents largely behave like the toy world. None of the three
  assumptions (AUROC ⇒ run signal; oracle gap ⇒ switchable specialization;
  alarm ⇒ retryable failure) is supported.
- Recommended practice, three tests that need only existing data:
  1. report within-issue AUROC next to pooled, and compare both with
     run-blind references for difficulty, model ability and length;
  2. compare any oracle gap with a no-specialization reference, and
     evaluate routers on held-out runs;
  3. judge an alarm by whether the action it triggers fixes the failure,
     comparing actions at equal numbers of attempts.

## Methods

- **Main data.** mini-swe-agent on the 500 SWE-bench Verified issues, one
  run per model per issue. Seven models: Claude Haiku 4.5, Claude Opus 4.5
  and 4.6, Gemini 3 Flash, GLM-5, GPT-5-mini, MiniMax-M2.5. 3,492 runs
  with outcomes, 994 failures. Two GPT-5.2 sources dropped (identical
  transcripts, conflicting labels). Released resolved labels are the
  ground truth.
- **23-model data.** A companion release of 25 models on the same issues,
  23 after the same exclusion. Used to check generality and, via the 16
  models outside the seven, to build the difficulty reference.
- **Predictors.** Fail-Fast-style. A predictor reads the trajectory after
  t ∈ {10, 20, 40} completed steps, either the last eight steps (R8) or the
  full prefix (ALL). Unlike Fail-Fast it sees neither the issue text nor
  the model name. TF-IDF logistic regression with issue-grouped five-fold
  CV; plus Qwen3-0.6B LoRA on one fold, each paired with a logistic model
  on the same truncated text. 18 configurations in total (10 TF-IDF,
  4 neural, 4 paired logistic). Headline numbers are on the 2,840 runs
  still active at step 20.
- **Run-blind references.**
  - *D_ext*: fraction of the 16 outside models that fail the issue. Uses
    outcomes not available at deployment; a diagnostic, not a predictor.
  - *M*: the model's failure rate on training issues (ability).
  - *Length + M*: M plus prefix length.
- **Pair decomposition.** AUROC split into cross-issue and within-issue
  failed–successful pairs. Within-issue pairs here are necessarily
  *different models on the same issue*. "Consistent" if the overall weaker
  model failed, "upset" otherwise. A run-tracking predictor should beat 0.5
  on both.
- **Tests.**
  - "Adds nothing beyond D_ext and M": the 90% interval of the AUROC gain
    lies within ±0.02 (equivalence test, Holm over 50 comparisons).
  - "Cannot tell which runs of an issue failed": conditional logit, issue
    fixed, does the score add to M.
  - "Oracle gap / upset rate not above chance": a no-specialization
    reference simulated from Rasch (one ability per model, one difficulty
    per issue) and 2PL (issues may discriminate more sharply) fits. Every
    issue ranks the models identically. 500 simulations.
- **Repeated-run pilot.** Nebius SWE-agent trajectories: Llama 70B and 8B,
  ≥4 runs each on 522 issues (SWE-bench-extra and dev). Runs per model and
  issue split into halves A (choose / find upsets) and B (measure), over
  200 random splits, with issue bootstrap repeating the choice per
  resample.

## Results

- **Table I (t = 20, 2,840 runs).**

  | Score | Pooled | Within-issue all | Consistent | Upset |
  |---|---|---|---|---|
  | D_ext | 0.924 | 0.500 | 0.500 | 0.500 |
  | M | 0.567 | 0.724 | 1.000 | 0.000 |
  | D_ext + M | 0.929 | 0.718 | 0.869 | 0.317 |
  | Length + M | 0.607 | 0.648 | 0.756 | 0.373 |
  | TF-IDF R8 | 0.624 | 0.604 | 0.678 | 0.415 |
  | TF-IDF ALL | 0.630 | 0.610 | 0.687 | 0.416 |

- **Pair composition.** 99.66% of the 494,109 failed–successful test pairs
  are cross-issue. Pooled AUROC is almost entirely a ranking of issues.
- **vs Length + M.** All ten 95% intervals of (predictor − Length + M)
  pooled AUROC contain 0.
- **Model ability.** M gets 0.72 within issue on 7 models and 0.82 on 23.
  All ten TF-IDF configurations fall below 0.5 on upsets. Ability explains
  2.5% of outcome variance with 7 models and about 10% with 23.
- **Incremental value.** Predictor added to D_ext + M: pooled change
  equivalent to zero in all 18 configurations (e.g. ALL: −0.001, 90% CI
  [−0.003, 0.000]). Within issue, conditional logit: no configuration helps
  (ALL: −0.011, 95% CI [−0.024, 0.002]).
- **Oracle gap (Fig. 2).** 7 models: best 76.8%, any 86.8%, gap 10.0 pp;
  upset rate 24.6%. 23 models: gap 12.4 pp, upset rate 17.7%. All eight
  comparisons (2 quantities × 2 model sets × Rasch/2PL) are inside the
  simulated range after Holm; with 23 models the Rasch range sits *above*
  the observed values. The authors say this does not show specialization
  is absent.
- **Unattainable share.** 448 of 994 failures (45%) are on the 64 issues no
  model solves. There is nothing to switch to.
- **Table II.**

  | A vs B | A − B, pp [95% CI] |
  |---|---|
  | Per-issue choice from past runs vs one model | +0.09 [−0.36, 0.63] |
  | Pre-run router vs best single model | +0.41 [−1.83, 2.85] |
  | Escalate every failure to a fixed 2nd model vs best single | +6.91 [4.67, 9.15] |

  No other hindsight-chosen pair beats the fixed escalation. The +6.91 pp
  assumes a perfect failure detector and spends a second attempt.
- **Specialization is real but sparse.** An upset found in half A recurs
  in half B 44% of the time against 19% under the reference (p = 0.004),
  on about 15 of 522 issues. Variance shares in the pilot: issue 53%,
  run-to-run 42%, issue × model 4.6%.
- **Alarms.** 64–82% of predictor-score variance is between issues.
  Consistent with Fail-Fast's cold restart adding only 0.2 pp.

## Critique / open questions

- **Against the digest entry: below (mild).** Every number in the digest
  summary is in the PDF and is quoted correctly. The full text is thinner
  than the summary reads in five places.
  1. **"A reference score that never looks at the run reaches 0.92."**
     True, but D_ext is built from 16 other models' *outcomes* on the same
     issues. The paper calls it a diagnostic, not a deployable predictor.
     It shows what pooled AUROC measures, not that difficulty is cheaply
     forecastable.
  2. **"Per-issue switching gains nothing on held-out runs."** The +0.09 pp
     comes from the two-model Llama pilot, and its CI reaches +0.63. The
     router's CI reaches +2.85. These are non-significant, not equivalence
     results. The digest also omits the paper's positive finding:
     upsets recur at 44% vs 19% (p = 0.004), so specialization exists on a
     few issues.
  3. **"Retrying rescues bad runs but not issues beyond the model."** No
     retry was run conditioned on an alarm. This is the toy world plus the
     pilot's 42% run-to-run share plus the between-issue score variance.
     It is a well-motivated inference, not a measurement.
  4. **"Gap no larger than random variation."** Stated with its caveat in
     the paper: the reference's power to detect specialization was not
     measured.
  5. **Digest "Why": "on-front for budget-as-ceiling".** The paper runs no
     budget or stopping experiment. Its budget content is the third test
     and the "equal budgets" future-work line. The escalation +6.91 pp is
     *not* budget-matched (two attempts vs one).
- **Compared with srinivasan2026continue (same group).**
  - *Different axis of the same confound.* Srinivasan held the model fixed
    (one model per setting, repeated attempts) and found within-task AUC
    0.50 at 20k tokens. Here each model runs once, so "within-issue" means
    *across models*, and the 0.60 there is fully accounted for by model
    identity (upsets at 0.42). Together: trajectory predictors carry the
    issue and the model, and neither paper finds run-level signal early.
    Neither covers same-model, same-issue discrimination on frontier
    closed models; the cited EsfandyariDoulabi et al. (2610.05572) is
    the within-model work.
  - *Difficulty transfers across models.* Srinivasan's task-rate oracle
    (0.97 pooled) used the same model's held-out attempts. D_ext uses
    other models and still gets 0.924. Difficulty is largely a property
    of the item.
  - *The action differs.* Srinivasan replayed a cold restart under a
    shared token cap and found it loses to continuing. This paper's
    "retry" and "escalation" are extra attempts outside any cap. The two
    are compatible: a bad run is worth retrying given extra budget, while a
    budget-matched cold restart pays to re-do setup. No one here tests a
    budget-matched retry targeted at bad runs.
  - *Unit.* Steps (10/20/40) here, transcript tokens there.
  - *Rigor.* Srinivasan has 147k runs, four settings and an exact
    replay decomposition. This is a short paper with two configurations
    tabulated out of 18, and the neural arm unreported.
- **What it implies for validating a stall trigger** (box rule accepted
  2026-11-14: `docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md`;
  our inference). `/iterate`'s counter counts measured non-improvements,
  not a prediction from text, so the AUROC numbers do not transfer. The
  validation design does.
  1. *Score the trigger against run-blind references, within problem.*
     Pooled over projects, a counter that fires more on hard problems will
     look predictive (99.66% of pairs here were cross-issue). The
     references are: problem difficulty (how other chains or projects on
     that problem fared), backbone (which ideator/implementer models ran
     the chain), and cycle index (the analogue of "length"). The trigger is
     a stall detector only if it adds to all three on the same problem.
  2. *Ask which cause the stall is.* A problem every repeated chain stalls
     on is "hard": halting is right and a pivot costs a cycle for nothing.
     A stall that does not recur on a repeat chain is a "bad run": a pivot
     or a plain re-try can rescue it. A stall another backbone clears is a
     "weak model": a model swap, not an axis pivot, is the fix. The rule
     applies one action to all three, so its value is set by the cause mix,
     which only repeated chains per problem can measure.
  3. *Score the action at equal attempts, against a resample.* The pivot
     cycle replaces the third refinement, so the rule already compares at
     equal cycles. Escalation here gained from the extra attempt, not the
     choice. In the same way, a pivot that "wins" must beat a same-axis
     resample at that cycle, not just a stalled history. With 42% of
     outcome variance run-to-run, single pivot successes are not evidence.
  4. *Log so it can be done later.* Beside `redirect=<axis>`, the log line
     needs the problem identity, the backbone models and the cycle index.
     Otherwise none of the three references can be computed afterwards.
  The proposal's compute-neutral defence still holds. This paper predicts
  the rule's *gain* is small where stalls are mostly difficulty, not that
  it costs anything.
- **Thin points.**
  - The 53/42/4.6% variance shares sum to 99.6%, implying ≈0.4% for model
    between a 70B and an 8B Llama. That seems low for that capability gap,
    and the decomposition method is not given.
  - The router is described in one sentence ("model identity and the issue
    text"); which data it runs on is not stated clearly.
  - The neural predictors are trained on one fold and reported only as
    entering the tests.
  - One benchmark, one scaffold, released labels. The main trajectory
    corpora come from an individual Hugging Face uploader; the paper does
    not describe how they were generated.
- **Open questions.**
  - Does a predictor trained to separate the three causes exist, and does
    acting on its cause label beat one action for all alarms at equal
    budget? (The authors' plan.)
  - How much specialization could the Rasch/2PL reference detect?

## Trust signals

- **Credibility:** 3.
  - For:
    - NC State, Assunção's trace-mining group (also srinivasan2026continue,
      Mehtiyev & Assunção, EsfandyariDoulabi et al.).
    - Public data, so the analysis is re-runnable in principle.
    - Equivalence tests with a stated margin and Holm correction, a
      conditional logit for the within-issue claim, and a simulated null
      for the oracle gap. The toy-world arithmetic checks (0.90, 0.88,
      0.91).
    - The abstract matches the body. Limits (non-rejection ≠ absence;
      D_ext not deployable; pilot uses older models) are stated.
  - Against:
    - Preprint, short-paper format with "preliminary findings"; no code.
    - Two of 18 configurations tabulated. Router and neural arm thinly
      described. The variance decomposition method is unstated.
    - No intervention: no retry, switch or stop is executed or replayed.
  - Not 4: compared with srinivasan2026continue (cred 4), far less detail
    and no replay. Not 2: a known group, public data, and careful null
    construction.

## Follow-up

- **Relevance:** 4. It adds the model-ability confound and the
  cause → action map to the hub's stall-trigger evaluation bar, and it gives a
  chance null for oracle gaps. It adds material evidence to:
  - [[concepts/budget-as-ceiling]]: how a stall trigger and its action
    must be validated (run-blind references within problem; actions at
    equal attempts), for the accepted redirect-before-halt rule.
  - [[concepts/spend-forecast-calibration]]: outcome difficulty forecasts
    from the item's history with other agents, not from item features. A
    pre-run router gains +0.41 pp.
  - [[concepts/pass-at-k]]: a cross-model oracle gap is what chance
    produces under no specialization; best-model pass@2 is the baseline.

  Not 5: no intervention or budget experiment, SWE not ML-research loops,
  a short preliminary paper. It sharpens srinivasan2026continue's bar
  rather than setting a new one.
- **Possible concept:** `failure-cause-attribution`. A failure is a hard
  item, a weak model or a bad run; each has its own action (stop, switch,
  retry); and any trigger is judged against run-blind references for the
  first two. Sources would be this paper, srinivasan2026continue and
  jain2026monitor. Not created.
- **Candidates** (cited, not yet in the graph):
  - EsfandyariDoulabi et al. 2026, "Disentangling task difficulty from
    run-level failure in agent failure prediction" (arXiv 2610.05572).
    Same group, the within-model repeated-run half of this argument.
  - Ge et al. 2026, "Agent psychometrics: task-level performance
    prediction in agentic coding benchmarks" (arXiv 2604.00594). IRT for
    agent benchmarks.
  - Wang, Lyu et al. 2026, "Fail-Fast, Restart-Smart" (arXiv 2608.03222),
    already a candidate from srinivasan2026continue.
  - Son et al. 2026, "SWE-Router" (arXiv 2607.00053), and Kim & Yoo 2026,
    "Atropos" (ISSTA 2026). The switching work the RQ2 null is aimed at.
