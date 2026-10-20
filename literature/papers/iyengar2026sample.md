---
kind: paper
title: "Sample, Mix, or Verify? Allocating Test-Time Compute under Imperfect Verifiers"
authors: ["Srinivasan Iyengar"]
institutions: ["Microsoft Research India"]
year: 2026
venue: "arXiv 2610.23255 (cs.LG), v1 2026-10-16, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.23255"
code_url: null
citations: null
source: "raw/papers/iyengar2026sample.pdf"
added: "2026-10-20"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/hce-evaluation]]"
tags: ["test-time-compute", "best-of-n", "imperfect-verifier", "llm-judge", "majority-vote", "precision-ceiling", "correlated-error", "shared-blind-spots", "gaussian-copula", "empirical-bayes", "pilot-calibration", "allocation", "reward-model", "verifier-threshold", "swe-bench-verified", "weaver", "large-language-monkeys", "coverage-vs-selection"]
---

# Sample, Mix, or Verify? Allocating Test-Time Compute under Imperfect Verifiers

## TL;DR

The paper models best-of-N as P(success) = P(a correct candidate exists) ×
P(the verifier picks it). Proposition 1 gives an exact success formula for
uniform choice among accepted candidates. For one generator it is
S_N = (a/s)(1 − (1 − s)^N), so verified success rises to the verifier's
precision a/s and then stops. Coverage keeps rising. The formula has three
correlation terms: task difficulty (beta-binomial or NPMLE prior), blind
spots shared across generators (Gaussian copula) and repeated verifier calls
on one candidate (a per-candidate acceptance propensity). Proposition 3
covers top-score selection. SPLIT fits these terms from a labelled pilot
(32 samples per task on 50% of tasks) by empirical Bayes. It then picks the
configuration with the highest predicted success: generators, mix weights,
sample count, judge calls or threshold.

On the held-out 50% of tasks, the best SPLIT variant in each setting is
0.02–0.65 points below the hindsight-best configuration. Every fixed rule
loses at least 2.5 points somewhere. All data are public releases; nothing
is generated and no API budget is spent. The agentic part is narrow. Agent
selection uses one R2E-Gym agent with 22 rollouts per SWE-bench Verified
issue, so the horizon is 16. The "175 agent systems" test covers coverage
only, with no verifier.

## Claims

- "Coverage … grows smoothly with N … But a system is graded on the one
  answer it submits." Verified success saturates at the verifier's
  precision, so the verifier's true- and false-accept rates must be
  modelled. On MATH, majority vote over Llama-3-8B-Instruct samples stays
  near 43% while coverage reaches 98% (Fig. 1).
- "Tries are correlated." Llama-3-8B-Instruct solves the average MATH
  problem 27% of the time. If every problem were equally hard, 10 tries
  would give 96% coverage; the measured value is 61%. 44 of 128 problems
  are almost never solved. Llama-3-8B and Gemma-7B per-problem solve rates
  correlate at 0.80.
- A third of wrong MATH-500 answers fool at least six of eight LLM judges,
  "so asking again repeats the mistake."
- Proposition 2: k majority-vote judge calls lower the precision ceiling
  relative to one call iff F_k/F_1 > T_k/T_1, i.e. "iff majority voting
  inflates false acceptances by a larger factor than true acceptances."
  A higher FPR alone is not sufficient.
- Proposition 4: planner regret is bounded by the *differential* forecast
  error between the chosen and the best configuration, not by absolute
  error. "The planner only has to rank configurations."
- "No fixed split works everywhere." The best configuration changes with
  budget, models and verifier.
- Limits stated by the author: SPLIT's point-forecast edge over power laws
  is large only under a simulated verifier and SWE-bench thresholds. It is
  0.4–0.6 points with real tests, absent for pure coverage and negative for
  argmax selection. Same-agent horizons stop at 16 rollouts. Pilots must be
  exchangeable with deployment.

## Methods

- **Data (Table 1).** All public:
  - MATH (128 tasks) and CodeContests (140) from the Large Language Monkeys
    release, 10,000 samples per task. For CodeContests the author re-ran
    80,403 programs against public tests (the verifier) and all tests (the
    truth).
  - HumanEval (164, 17 generators) and MBPP (399, 10 generators) from
    Stroebl et al. 2024. Base tests are the verifier; EvalPlus is the truth.
  - MATH-500 and GPQA from the Weaver release: 100 Llama-3.1-70B-Instruct
    candidates per task, verdicts from 8 LLM judges, 19 reward-model scores
    and the Weaver score. "A random judge" is a stochastic verifier whose
    per-candidate propensity is the fraction of the 8 judges that accept.
  - SWE-bench Verified: R2E-Gym 32B agent, 492 issues × 22 rollouts, with
    an execution-free LLM patch verifier P(yes).
  - SWE-bench Verified agent pool: 175 submitted systems × 500 issues, one
    attempt each, coverage only.
- **Simulated verifier.** τ = 0.9, per-task FPR ~ Beta(mean 0.1,
  concentration 2), κ = 4. It drives the MATH and CodeContests rows of
  Table 2.
- **Allocation protocol.** Calibrate on a random 50% of tasks from an
  n-sample pilot (32 by default) and score the chosen configuration on the
  other 50%. Regret is oracle success minus achieved success, in points.
  There are 5 splits for simulated verifiers and 10 for real ones. Budgets
  run 4–256 cost units per task (16–1,024 for judges). The grid covers
  single generators and pairs with weights {1/4, 1/2, 3/4}, k ∈ {1, 3, 5, 9}
  judge calls and SWE-bench thresholds {none, 0.3, 0.5, 0.7} for
  N ∈ {1, 2, 4, 8, 16}.
- **Verifier models.** Pooled (one TPR, beta prior over per-task FPR),
  task-varying (3-D Gaussian copula over p, TPR and FPR), automatic choice
  between the two on a split pilot, and for judges an "empirical task
  population" that averages over the calibration tasks' own candidates.
- **Costs.** Parameter count, or (parameters)^γ × measured output tokens
  with γ ∈ {1, 0.7, 0.5, 0} for compute, price, latency and flat pricing.
  A judge call costs one generation.
- **Baselines.** Strongest model, most accurate per cost, uniform ensemble,
  an adaptation of Singhi et al. 2025's compute-optimal exponents
  (S ∝ C^0.57, V ∝ C^0.39), generation only (k = 1), maximum verification
  (k = 9), fixed threshold 0.5, no verifier, and SPLIT with independent
  errors as an ablation.
- **Statistics.** Paired Wilcoxon signed-rank tests and bootstrap 95% CIs
  over shared (model, pilot, seed) or (dataset, seed, budget) units. Table
  11 lists every headline comparison. Everything runs on one 8-core laptop
  CPU.

## Results

- **Allocation regret, samples and generators (Table 2, points).**

  | Method | Sim. MATH | Sim. CodeContests | Real tests, param cost | Real tests, compute-matched |
  |---|---|---|---|---|
  | SPLIT | 0.54 | 0.82 | 0.62 | 0.46 |
  | SPLIT, independent errors | 1.69 | 1.01 | 1.46 | 1.34 |
  | Singhi et al. rule | 11.95 | **0.78** | — | — |
  | Strongest model | 12.35 | 0.99 | 0.94 | 0.94 |
  | Uniform ensemble | 16.73 | 3.62 | 2.33 | 2.53 |
  | Most accurate per cost | 19.81 | 5.31 | 3.74 | 3.35 |

  The 12-point MATH losses for strongest-model and Singhi come from small
  budgets. There one Llama-3-70B sample (cost 7.8–8.75) is unaffordable, so
  those rules submit nothing. Two key comparisons are not significant:
  SPLIT vs strongest model with real tests (−0.32, p = 0.43) and SPLIT vs
  Singhi on simulated verifiers (p = 0.13; 0.09 compute-matched).
- **How much to verify (Table 3).** With LLM judges, the best SPLIT variant
  has 0.04 regret on MATH-500 (judge-level IRT) and 0.02 on GPQA (empirical
  task population). Generation only is 0.10 / 2.58 and maximum verification
  1.69 / 3.84. On SWE-bench thresholds, the task-varying verifier is at
  0.33, the auto-selected model 0.56, the pooled verifier 2.36, a fixed
  0.5 threshold 2.62 and no verifier 3.58. With one rollout, a 0.5
  threshold *lowers* the resolve rate from 32.5% to 25.1%, because it
  rejects correct patches with nothing to fall back on.
- **Modelling correlation pays.** Assuming independent errors raises regret
  1.2–4.8× (0.54 → 1.69 on MATH, 0.11 → 0.53 on GPQA; p ≤ 0.003 on MATH,
  CodeContests and real tests).
- **Other prices (Fig. 3a, Table 11).** On CodeContests with real tests,
  SPLIT's regret stays between 0.45 and 0.62 under all five cost models.
  Under flat or latency-like prices (γ ≤ 0.5), "strongest model" is
  optimal and beats SPLIT by 0.45–0.50 points (p ≤ 0.001).
- **Online re-planning (Table 4).** Stopping a task at its first accepted
  candidate and moving the savings to unresolved tasks adds +0.56 points
  over the static plan (p < 10⁻⁵). That ties the static hindsight oracle
  (+0.06, p = 0.48). Most of the gain is early stopping with a pooled
  budget. This is simulated from per-task rates on CodeContests only.
- **More judges can hurt (Fig. 4, §6.3).** On MATH-500, going from k = 1 to
  k = 9 multiplies wrong-answer acceptance by 1.052 (0.507 → 0.534) and
  correct-answer acceptance by 1.026 (0.931 → 0.955). Nine calls lower the
  precision ceiling on 61% of tasks. Measured verified success moves by
  only −0.3 to +0.1 points. On GPQA, wrong-answer acceptance falls (×0.75),
  the ceiling rises on 86% of tasks, and nine calls add +1.2, +3.3 and +4.8
  points at N = 4, 16 and 64. Fitted judge concentrations are κ_w ≈ 14 on
  MATH-500 and 60–270 on GPQA (κ → ∞ means independent calls).
- **Pooled verifier rates mislead (Table 5).** For the gain from 9 judge
  calls instead of 1 on GPQA (real +1.2 / +3.3 / +4.8), pooled judge rates
  predict +3.7 / +7.3 / +10.0, about twice the real gain. The difficulty
  prior is not the cause. Per-task TPR and FPR have Spearman 0.54 and 0.65
  with task difficulty, and pooling erases that coupling. The empirical
  task population predicts within 0.2 points.
- **Selector choice.** The Weaver aggregate score reaches 86–93% on
  MATH-500 against about 81% for judge-based selection. On GPQA it reaches
  about 43%, below a single judge call (46–49%).
- **Generator capability and FPR (Fig. 5).** For fixed tests, FPR rises with
  generator pass@1: Spearman ρ = 0.97 over 17 models on HumanEval+ (1.5%
  for Vicuna-7B to 31% for GPT-4o) and 0.95 over 10 on MBPP+. This
  replicates Stroebl et al. 2024. With 19 reward models at one fixed
  threshold, the 70B generator's wrong answers pass more often than the
  8B's for 16 of 19 (p = 0.004). Re-tuning the threshold to equal TPR (0.9)
  removes the effect (9 of 19, p = 1).
- **Mixing generators.** Pilot copula correlations are 0.67–0.96. For
  Llama-3-8B + Gemma-7B on MATH (ρ̂ = 0.83), independence predicts +5
  points from mixing. The real gain is +1.6 and SPLIT predicts +2.4. Mean
  error on the mixing gain is 2.81 points under independence and 0.43 for
  SPLIT. Among agents, 26 of 500 SWE-bench Verified issues are solved by
  none of the 175 systems.
- **Forecasting beyond the pilot (Table 6, MAE in points).** SPLIT is
  better than a power law under a simulated verifier (1.9 vs 7.6 at
  N = 8,192), on CodeContests real tests (2.13 vs 2.77), on HumanEval
  (1.69 vs 2.13) and on SWE-bench thresholds (2.74 vs 6.70). It is worse
  for pure coverage (3.3 vs 2.6) and SWE-bench argmax (3.28 vs 1.58,
  p = 0.006). On the 175-system pool it ties (2.23 vs 1.94, p = 0.57) and
  has larger end-point error (2.9 vs 2.0, p = 0.004). With a pilot of the
  16 oldest systems, every method badly under-predicts.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). The digest's numbers are
  in the abstract, and the abstract hedges ("we report where it loses").
  The digest drops those hedges and puts the agent pool under the
  allocation claim.
  1. "Within 0.02–0.8 points" uses the best SPLIT variant per setting,
     and the best variant differs by setting: judge-level IRT on
     MATH-500, empirical task population on GPQA, task-varying verifier
     on SWE-bench. The pooled-verifier SPLIT is at 2.36 on SWE-bench.
     The automatic model choice, the variant a user would deploy without
     hindsight, is significantly behind the better fixed model on
     CodeContests and SWE-bench (Table 7).
  2. The "175 agent systems" do not test allocation or selection. They
     test the coverage prior, where SPLIT ties the power law and is worse
     at the longest horizon. Agentic selection is one 32B agent with 22
     rollouts per issue.
  3. "Explains when adding more LLM judges lowers precision" is exact, and
     the condition is met on MATH-500. The measured cost in verified
     success is −0.3 to +0.1 points. The mechanism is real and small here.
  4. Fixed rules do lose ≥ 2.5 points somewhere. But SPLIT vs strongest
     model with real tests is not significant (p = 0.43), and under flat
     or latency pricing strongest model beats SPLIT.

  The digest's link to `spend-forecast-calibration` is loose. SPLIT
  forecasts *success* at a fixed budget, not spend.
- **The judges are a recorded panel, not repeated calls.** "k calls" means
  k draws from the 8 Weaver judges, so propensity is the panel's agreement
  rate on a candidate. That measures cross-judge shared blind spots well.
  It does not measure one judge sampled k times at temperature, which is
  how most agent loops re-ask. The paper does not list which models the 8
  judges are, so it cannot say whether shared blind spots come from shared
  model families.
- **Conditional independence within a task is assumed (Assumption 1).**
  The author flags duplicated or near-duplicate answers as unmodelled and
  a likely bias at large N. Agent rollouts from one scaffold often
  converge on the same patch, so this matters most where this project
  would use it.
- **Simulated verifier does much of the headline work.** The biggest
  margins (MATH, about 12 points over fixed rules) come from a synthetic
  verifier and from small budgets where some rules cannot afford one
  sample. With real verifiers the margins are 0.3–4 points.
- **The regret guarantee is vacuous at these sizes.** Proposition 4(c)
  gives 60 points at T = 70, |C| = 13. The author says it explains
  consistency, not the observed regret. The empirical argument (pairwise
  error 0.6–2.8 points against max absolute error 3.2–8.5) is the useful
  part, and it is checkable.
- **Online re-planning is simulated.** Samples are drawn from each
  evaluation task's measured rates, the same rates that score the static
  plan. The policy ignores the risk that a false acceptance ends a task.
- **Code is promised, not released.** No URL is given. The appendix
  describes 35 unit tests, 26 numerical checks of the propositions, fixed
  seeds, and a pipeline for 64-rollout same-agent runs (32,000 agent runs)
  that has not been run.
- **What it does not show.** There is no research-agent or ML-engineering
  task, no multi-step selection (choosing which experiment to keep in a
  loop) and no same-model self-judging. Every verifier is external to the
  generator.

## Trust signals

- **Credibility:** 3.
  - For: Microsoft Research India. Every sample and verdict comes from a
    public release (Large Language Monkeys, Stroebl et al., Weaver, R2E-Gym,
    SWE-bench experiments), so results can be rerun on a laptop CPU in
    under 10 hours. The propositions come with proofs. Table 11 gives paired
    CIs and p-values for every headline comparison, including the
    non-significant ones and the ones SPLIT loses. The limitations section
    names the settings where SPLIT is worse than a power law.
  - Against: a single author, no peer review (v1, four days old). The code
    is promised with no URL. Headline regret is reported for the best of
    several SPLIT variants per setting. Agentic evidence is one agent with a
    16-rollout horizon plus a coverage-only pool.
  - Not 4 because the code is not released and the cross-setting headline
    picks variants in hindsight. Not 2 because the data are public, the
    statistics are complete, and the paper reports its own losses.

## Follow-up

- **Relevance:** 3. This is the cleanest quantitative statement in the
  graph of why a selector caps a best-of-N gain. It adds three numbers to
  [[concepts/shared-substrate-contagion]] and [[concepts/pass-at-k]]:
  - a measured shared blind spot across a judge panel (a third of 11,015
    wrong MATH-500 answers pass at least 6 of 8 judges);
  - an exact condition for when more votes hurt;
  - a 2× forecast bias from pooling judge accuracy across tasks.

  It agrees with [[literature/papers/kim2026are]] (selection, not
  generation, is the scarce resource), with
  [[literature/papers/zhang2026veriharness]] (agreement is not correctness)
  and with [[literature/papers/bobadillasuarez2026audit]] (votes do not
  beat the average worker), and it gives the mechanism the first two lack.
  Not 4 because every task is single-shot math, QA or code. No agent loop
  selects among its own experiments here, and the agent-side evidence is
  thin.
- **Bearing on live fronts.**
  - *Shared-substrate correlated error:* a direct measurement on real
    judges, and the first source here that turns "judges share blind
    spots" into a test you can run on a pilot (Proposition 2). It also
    gives [[literature/papers/zhang2026who]]'s untested "detector
    diversity" claim a concrete check: measure F_k/F_1 against T_k/T_1 on
    labelled candidates before adding a judge.
  - *HCE / LLM judges in loops:*
    - judge accuracy is coupled to task difficulty, so a judge validated
      on pooled data overstates its value on hard items;
    - a scored verifier's threshold must be re-tuned when the generator
      changes. The evaluation rule does not say this yet. Noted here, not
      proposed.
    - The 50/50 calibrate-then-evaluate protocol is a clean holdout.
  - *Redirect-before-halt proposal, noise band, skill scoping:* no direct
    bearing. Proposition 4 (rank accuracy, not absolute accuracy, decides
    regret) fits the existing note in
    [[concepts/spend-forecast-calibration]] that allocation needs less
    than admission. It does not change the noise band.
- **Candidates** (cited, not in the graph):
  - Stroebl, Kapoor & Narayanan 2024, "The limits of inference scaling
    through resampling" (arXiv 2411.17501).
  - Saad-Falcon et al. 2025, "Shrinking the generation-verification gap
    with weak verifiers" (Weaver, arXiv 2506.18203).
  - Singhi et al. 2025, "When to solve, when to verify" (arXiv
    2504.01005).
  - Brown et al. 2024, Large Language Monkeys (arXiv 2407.21787).
  - Schaeffer et al. 2025, power-law origins of repeated sampling (arXiv
    2502.17578).
  - Chen et al. 2024, "Are more LLM calls all you need?" (arXiv
    2403.02419). Voting accuracy can be non-monotone in the number of
    calls.
