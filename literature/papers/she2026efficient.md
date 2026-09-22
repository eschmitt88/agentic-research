---
kind: paper
title: "Efficient Benchmarking in Production: A Study of an Evolving LLM Agent"
authors: ["Yining She", "Lei Lin"]
institutions: ["Carnegie Mellon University", "Meta"]
year: 2026
venue: "arXiv (cs.AI; cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.21267"
code_url: null
citations: null
source: "raw/papers/she2026efficient.pdf"
added: "2026-09-22"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/spend-forecast-calibration]]"
tags: ["efficient-evaluation", "benchmark-subsetting", "item-response-theory", "adaptive-testing", "temporal-holdout", "chronological-split", "operational-simplicity", "cross-agent-transfer", "evaluation-cost", "regression-testing", "production-deployment", "industry-report", "failed-replication", "no-artifact-release"]
---

# Efficient Benchmarking in Production: A Study of an Evolving LLM Agent

## TL;DR

An industry report on making a recurring agent benchmark cheaper. A
519-question, ~3-hour benchmark for a production analytics agent (tens of
thousands of MAU) was run 574 times over 52 days; the runs are split
**chronologically** into 287 calibration (Day 1–28) and 287 held-out test
(Day 29–52) runs, and four partial-evaluation methods are replayed against
the full-run pass rate. Multidimensional 2PL adaptive testing wins: **1.03
pp MAE at k=200 (38.5% of the benchmark)**, and it is not a marginal win —
it has lower MAE *and* higher Spearman *and* higher Kendall than the
difficulty-stratified fixed subset **at all twelve budgets tested**. The
team deployed the difficulty-stratified fixed subsets anyway, for
operational reasons they state plainly, paying **+0.37 pp of MAE at the
k=200 operating point** (1.40 vs 1.03) and between +0.18 and +0.68 pp
across the four deployed budgets. The paper is unusually good about
publishing the numbers that make its own deployment look second-best
(twelve appendix tables, per-family breakdowns) and unusually thin on
reproducibility: **no benchmark, no outcome matrix, no code released.**
Two headline framings need walking back — "transfers without
recalibration" means "beats uniform random sampling *pooled* across 299
runs from five anonymized families, where one family is 49% of the pool",
and per-family it loses rank fidelity to random sampling at the deployed
budget for at least one family; and the abstract's "evolving agent" premise
is never quantified, with the paper's own calibration-window sweep showing
**no detectable instrument decay** over 52 days.

## Claims

- **Partial evaluation can stand in for a full agent benchmark run at a
  fraction of the cost.** Multidimensional 2PL adaptive testing gives the
  best score fidelity: 200 of 519 questions (38.5%) → 1.03 pp pass-rate
  MAE.
- **Rankings survive better than scores.** At k=300 the best adaptive
  configuration reaches 0.992 Spearman against full-benchmark ordering.
- **Difficulty-stratified fixed subsets are the strongest non-adaptive
  method** and were the ones deployed, "because of their operational
  simplicity."
- **The deployed subsets transfer without recalibration to five other agent
  families** and "remain stable across calibration windows as short as one
  day."
- **Historical outcome caching is dominated.** Its MAE curve lies above
  every other method over their shared execution range — though it
  preserves *rankings* far better than its score error suggests.
- **A failed replication of prior art.** IRT-feature clustering (adapted
  from Polo et al.'s tinyBenchmarks) performs poorly here, contradicting
  that paper's finding that IRT-based selection is consistently effective.
  The authors offer two explanations, both of which bear on their own
  transfer claim (see Critique).
- **A more expressive IRT model substantially improves adaptive testing**
  — the multidimensional 2PL variant beats the Rasch variant everywhere,
  and the Rasch variant is *worse than uniform random sampling* at small
  budgets.

## Methods

- **System under test.** "A production analytics agent designed to answer
  natural-language questions about data," combining LLM reasoning with
  retrieval/analysis tools, serving "tens of thousands of monthly active
  users." Its "development and monitoring generate tens of thousands of
  evaluation runs." The agent, its models, and the five comparison families
  are all anonymized.
- **Benchmark.** 519 human-curated questions "about data written by
  internal analysts"; binary pass/fail per question from an automated
  grader; one full run takes "approximately three hours." No user data or
  PII in questions or recorded outcomes.
- **Data and split.** 574 runs over 52 days → **287 calibration (Day 1–28)
  and 287 held-out test (Day 29–52)**, split *by time*, not at random. Runs
  are retained only if they have "valid outcomes for at least 80% of the
  benchmark," and each run's reference pass rate is computed over its valid
  outcomes.
- **Four method families**, each defined by a selection rule plus an
  estimator of the full pass rate:
  - *Random sampling* — k drawn uniformly without replacement, unweighted
    mean, averaged over 20 seeds. The baseline.
  - *Historical caching* — reuse the recent majority verdict for any
    question whose 7-day rolling pass rate satisfies `min(ȳ, 1−ȳ) ≤ ε`
    (≥5 historical trials required, rebuilt daily); execute the rest. ε
    swept 0.00–0.30.
  - *Fixed subsets* — chosen once from all calibration runs and reused
    unchanged on every test run. Three selectors (difficulty
    stratification, historical-response K-means, IRT-feature K-means) × two
    estimators (weighted subset; gp-IRT, which blends the weighted estimate
    with a p-IRT reconstruction of the unobserved items). **Difficulty
    stratification uses only the scalar Rasch difficulty β** — sort by β,
    cut into k equal strata, take the median-difficulty question from each,
    weight by stratum share.
  - *Adaptive testing* — Fisher-information item selection with per-run
    ability re-estimation, in a Rasch variant and a multidimensional 2PL
    variant with D-optimal selection; score reconstructed by p-IRT.
- **Metrics.** Pass-rate MAE against the full run (score fidelity) and
  Spearman ρ / Kendall τ between the partial and full orderings of the 287
  held-out runs (ranking fidelity). Efficiency is reported as execution
  fraction |S|/N.
- **Two deployment validations, on the difficulty-stratified subset only:**
  - *Cross-agent transfer* — subsets and Rasch/gp-IRT parameters calibrated
    on the original agent are applied **unchanged** to 299 runs from five
    additional families that "differ in their agent frameworks, execution
    harnesses, tool sets, and model configurations." Family run counts: A
    51, B 146, C 56, D 34, E 12.
  - *Calibration-window sensitivity* — ten nested windows all ending Day
    28, start day advancing in 3-day steps, giving 287 down to **14
    calibration runs** (a one-day window); refit and re-evaluated on the
    same 287 held-out runs.

## Results

### Score fidelity (Table 1, Table 4; 287 held-out runs)

Pass-rate MAE in percentage points, lower better:

| k | exec % | random | difficulty gp-IRT | Rasch adaptive | **2PL adaptive** |
|---|---|---|---|---|---|
| 10 | 1.9 | 10.48 | 7.57 | 10.94 | **5.07** |
| 100 | 19.3 | 2.94 | 2.65 | 2.92 | **1.97** |
| 200 | 38.5 | 1.79 | 1.40 | 1.37 | **1.03** |
| 300 | 57.8 | 1.18 | 0.99 | 0.79 | **0.58** |
| 400 | 77.1 | 0.69 | 0.47 | 0.42 | **0.29** |

- **The abstract's headline is exact.** 2PL adaptive, k=200, 38.5%,
  1.03 pp. Verified in Table 4.
- **2PL adaptive strictly dominates the deployed method.** Across all
  twelve budgets it has lower MAE, higher Spearman, and higher Kendall than
  difficulty-stratified gp-IRT — no crossover anywhere.
- **Rasch adaptive is worse than random sampling at small budgets**
  (k=10: 10.94 vs 10.48; k=20: 9.79 vs 7.22; k=50: 6.46 vs 4.41). Fisher
  selection targets informative rather than representative items, so on a
  single ability axis it can actively hurt a *mean* estimate. It only
  overtakes the fixed subset from k=200.
- **The main comparison figure excludes the winner.** Figure 2 / Table 1 —
  the "headline method comparison" — contains random, difficulty-stratified
  (both estimators), caching and *Rasch* adaptive. Multidimensional 2PL
  appears only in the §5.3.2 "design choices" subsection and Table 4.
  Against Rasch alone, the deployed method looks competitive (it wins at
  k ≤ 150). Against the method that actually won, it never wins.
- **Caching is the clear loser on score.** MAE 7.36 pp at 20.3% execution,
  falling to 0.81 pp only at 81.6%. At matched execution it is beaten
  badly: 1.54 pp at 69.1% versus 0.60 / 0.73 / 0.93 for Rasch adaptive /
  fixed-subset gp-IRT / random at k=350 (67.4%).
- **But caching preserves rankings far better than its scores.** 0.904
  Spearman / 0.732 Kendall at 20.3% execution (Table 2) — i.e. its errors
  are largely a shared bias that cancels in the ordering. The body reports
  these as 0.905/0.733; a one-in-the-last-digit mismatch with its own table.

### Fixed-subset design (Table 3)

- **Difficulty stratification wins overall**, leading under both estimators
  from k=150.
- **Both clustering selectors fall *below* uniform random sampling from
  k=200 onward** under either estimator (k=200: random 1.79 vs 2.51 / 2.12 /
  2.47 / 2.07). Historical-response K-means does lead at very small budgets
  (k=30 through k=100 under gp-IRT).
- **gp-IRT helps the clustering selectors more than difficulty
  stratification**, whose weighted and gp-IRT estimates are "consistently
  similar" — at k=200, 1.43 vs 1.40. The IRT reconstruction is nearly
  redundant for the method that was deployed.

### Cross-agent transfer (§6.1, Tables 6–11)

Pooled over 299 runs from five families, no recalibration:

- **gp-IRT beats random sampling on MAE at 11 of 12 budgets.** Verified
  against Table 6; the exception is k=350 (1.02 vs 0.99).
- At k=200: random 1.86, weighted 1.49, gp-IRT **1.47** pp, "close to its
  1.40 pp MAE on the original 287 held-out runs."
- Pooled ranking: both fixed-subset estimators exceed random in Spearman
  *and* Kendall at every budget. Verified.
- **The paper states the non-uniformity itself:** "The advantage is not
  uniform across every family and computational budget. Nevertheless, gp-IRT
  has lower MAE than random sampling for a majority of budgets within each
  family."
- **Per-family, the ranking claim does not hold.** For **Family A at the
  deployed k=200**, transferred gp-IRT gets Spearman 0.506 / Kendall 0.359
  against random sampling's 0.622 / 0.451 — the transferred subset is
  materially *worse* than random at ordering that family's runs (it also
  loses at k=50, 100 and 250). **Family D** loses at k=100 (Spearman 0.811
  vs 0.886; Kendall 0.667 vs 0.736) and on MAE at k=400 (1.13 vs 0.94).
  Families B, C, E behave well.
- The authors' interpretation: the transfer "indicates that the IRT model
  captures intrinsic aspects of question difficulty that generalize across
  agent systems, even when calibrated on a single agent."

### Calibration-window sensitivity (§6.2, Table 12)

- **MAE does not degrade monotonically as the window shortens**, and
  barely degrades at all. At k=200, gp-IRT ranges **1.35–1.59 pp** across
  ten windows; the **one-day window (14 runs) gets 1.41 pp against 1.40 pp
  for the full four weeks.** Verified.
- Variation is largest at k=100 (2.04–2.97). Spearman/Kendall show the same
  non-monotonic, essentially flat pattern.
- **The authors flag this as possibly an artifact of their own setting:**
  "This stability may reflect the relatively mature development stage of
  the agent and limited day-to-day change during the study."

### Practical recommendations (§7) — the deployment decision

- **Deployed: difficulty-stratified fixed subsets at k ∈ {100, 200, 300,
  400}**, "allowing users to choose an execution–fidelity tradeoff."
- Stated rationale, in the paper's own terms:
  - "The subset is constructed once and requires neither sequential
    selection nor run-specific ability updates."
  - "Executing the same questions in every run also supports direct
    question-level comparisons and diagnosis as the production agent
    evolves."
  - "Fixed subsets reveal the workload before execution" — the cost is
    known in advance rather than discovered during the run.
  - **"They also avoid additional sequential serving logic in an
    evaluation system designed to execute questions in parallel."** This is
    the load-bearing one: adaptive testing is inherently sequential and the
    existing harness is parallel, so the better method demands an
    architectural change to the surrounding system, not just a new script.
  - Caching by contrast "retains the full question set, but requires recent
    outcomes and monitoring for stale verdicts."
- **A recalibration and audit policy, stated explicitly:** "recalibrat[e]
  after material changes to its models, prompts, tools, or execution
  system, or after sustained shifts in question-level outcomes. Developers
  should also periodically run the full benchmark and compare its pass rate
  with the fixed-subset estimate to measure the error introduced during
  ongoing monitoring."

## Critique / open questions

- **The MAE numbers are uninterpretable in absolute terms, because the
  paper never reports the pass rates.** Nowhere in 28 pages is the agent's
  pass rate, its spread across the 574 runs, or its trajectory over the 52
  days given. A 1.03 pp MAE is only "good" relative to the effect sizes a
  developer needs to resolve. If run-to-run pass rate varies by a few
  points, 1.03 pp is close to useless; if it varies by 30 points, it is
  excellent. The reader cannot tell. This is the single largest gap.
- **No test–retest noise floor.** The introduction motivates the work by
  noting "stochasticity may also require repeated trials," and Related Work
  observes that "non-deterministic agent outcomes resemble flaky tests" —
  but the study never measures the MAE between **two full runs of the same
  configuration**. That number is the floor any estimator is competing
  against, and without it there is no way to know whether 1.03 pp is
  approaching irreducible noise or leaving a lot on the table. See
  [[concepts/pass-at-k]].
- **The "evolving system" premise is asserted, never measured.** The
  framing — and the justification for the chronological split — is that the
  agent changes under the instrument. The paper never quantifies how much
  it changed during the 52 days. Its one relevant measurement (the
  calibration-window sweep) is a **null**: a 14-run, one-day calibration
  window is as good as a 287-run, four-week one, which is what you would
  expect if the agent barely moved. The authors say as much. So this is
  best read as *evidence that the temporal protocol is the right design*,
  not as evidence that instrument decay was observed and handled.
- **"Transfers without recalibration" is doing more work in the abstract
  than in the body.** Three qualifications the abstract omits: (i) the bar
  is *beating uniform random sampling at matched budget*, not matching
  original-agent fidelity; (ii) the pooled result is dominated by **Family
  B, which is 146 of 299 runs (49%)** — the pooled curve is substantially a
  Family B curve; (iii) per-family, the subset loses rank fidelity to
  random sampling at the deployed k=200 for Family A and at k=100 for
  Family D. The body's own sentence ("not uniform across every family and
  computational budget") and its fully honest per-family tables are the
  correction; the abstract does not carry it. Family E's rank correlations
  are computed over **12 runs** and should not be read at all.
- **Cross-population MAE comparison is not apples to apples.** "gp-IRT
  remains close to its 1.40 pp MAE on the original 287 held-out runs" (1.47
  pooled) compares error on two different run populations whose pass-rate
  distributions are never reported. At k=100 the *transferred* MAE (2.50)
  is actually **lower** than the original-agent MAE (2.65), which is a hint
  that the two populations differ in ways that make the comparison soft.
- **An internal tension that resolves into the paper's most interesting
  unstated point.** §5.3.1 explains IRT-feature clustering's failure by
  arguing that "recurring configurations of one production agent may
  contain fewer distinct response patterns than diverse LLM populations,
  making multidimensional item parameters less reliable." §6.1 then
  concludes the IRT model "captures intrinsic aspects of question
  difficulty that generalize across agent systems." These are consistent —
  what transfers is the **scalar Rasch difficulty ordering**, not the 2PL
  structure — but the consequence is worth naming: **the property that made
  the deployed instrument simple and transferable (it is a coarse scalar)
  is the same property that caps its fidelity.** The robust instrument and
  the accurate instrument are different instruments, and the paper chose
  robustness without framing it that way.
- **The authors concede they never measured the actual decision.**
  Limitations: MAE and aggregate rank correlation "do not directly measure
  regression detection or release-gate decisions at operational
  thresholds. Future work should add threshold-based analyses that measure
  missed regressions, false alarms, and agreement with decisions based on
  the full benchmark." The deployed use case is gating releases; the
  evidence is about reconstructing a scalar. A 1.40 pp MAE with an unknown
  error distribution says nothing about missed-regression rate.
- **Nothing is reproducible.** "We do not release the benchmark questions,
  the question-level outcome matrix, or the implementation." One
  organization, one benchmark, anonymized agent families. The methods are
  fully specified and replayable on *someone else's* recorded outcome
  matrix, which is a real mitigation, but no claim in this paper can be
  checked.
- **Selection of the 574 is under-specified.** Development and monitoring
  generate "tens of thousands of evaluation runs"; the study uses 574 over
  52 days. Beyond the ≥80%-validity filter and a reference to "routine
  runs," how the analyzed population was drawn from the operational one is
  not stated. Dropping runs with <80% valid outcomes also plausibly drops
  the most broken agent builds — exactly the regressions an evaluation
  system exists to catch.
- **A small internal inconsistency worth noting for anyone quoting §5.3.2:**
  the text says multidimensional 2PL's "largest advantage" over Rasch and
  random is "at k=100 (19.3%): 1.97 pp versus 2.92 and 2.94 pp." By its own
  Table 4 the advantage is larger at every smaller budget, absolutely and
  relatively (k=10: 5.07 vs 10.94 and 10.48; k=20: 3.90 vs 9.79 and 7.22).
- **Open question the paper raises implicitly.** Adaptive testing was
  rejected because the harness executes in parallel. Nobody tested a
  *batched* adaptive scheme (select a block of k/b items, execute in
  parallel, re-estimate, repeat) which would recover most of the
  information gain at a handful of synchronization points. The
  simplicity/fidelity tradeoff as presented is a binary between two
  endpoints of a spectrum that was never explored.

## Trust signals

- **Credibility:** 3. A CMU + Meta preprint (Yining She at CMU, "work done
  while at Meta"; Lei Lin at Meta) reporting first-hand production data at
  real scale — 574 full runs of a 3-hour benchmark against an agent with
  tens of thousands of MAU is evidence almost nobody publishes. The
  methodology is sound where it counts: a **chronological** calibration/test
  split rather than a random one, a 20-seed random baseline, a held-out
  transfer population, a nested calibration-window sweep, and twelve
  appendix tables that report every number behind every figure — including
  the per-family breakdowns that undercut the abstract's own transfer
  framing, and the Table 4 numbers that show the deployed method is
  strictly dominated. The Limitations section names the right gaps
  unprompted (one organization, one benchmark, fixed-subsets-only transfer
  evidence, no threshold/regression-detection analysis). Held to 3 rather
  than 4 by the reproducibility floor: **no peer review, no code, no
  benchmark, no outcome matrix, anonymized agent and comparison families**
  — nothing here is independently checkable, and the rubric weights
  released artifacts at least as heavily as affiliation. Also held back by
  the un-reported pass-rate distribution, which leaves the headline MAE
  without a scale, and by an abstract that states the transfer result
  without the qualification its own body supplies.

## Follow-up

- **Relevance:** 3. Useful prior art on an active theme that does not shift
  any concept's architecture. The domain is off-centre for this project —
  a production analytics agent doing natural-language data Q&A, not an
  ML-research agent — and the transferable content is an evaluation-cost
  technique rather than an agent-architecture pattern. It earns 3 rather
  than 2 on three counts: it is a rare first-hand production deployment
  report; it supplies a concrete, reusable **audit discipline** that
  [[concepts/hce-evaluation]] currently lacks; and it is a documented,
  quantified instance of choosing operational simplicity over measured
  fidelity, which is direct evidence for the `/elevate` simplicity gate.
  Not 4 because no concept here changes shape as a result, and the two
  claims that would have justified 4 — instrument decay under an evolving
  system, and clean cross-agent transfer — are respectively a null and a
  pooled result with per-family exceptions.

- **[[concepts/hce-evaluation]] — proposed addition, the strongest of the
  four.** The concept covers hiding a holdout from a search loop. This adds
  a distinct construct-validity axis: **when the system under test evolves,
  the measuring instrument is itself calibrated on that system's history,
  so the holdout must be held out along the axis of change (time), not at
  random.** A random split across the 574 runs would have let item
  difficulties be fitted on runs of the same agent build being scored — the
  evaluation analogue of look-ahead bias in a backtest. The paper's Day
  1–28 / Day 29–52 protocol is the correct construction and is worth
  naming as a reusable rule. The second, more actionable half is §7's audit
  policy: **a cheap proxy metric requires a scheduled re-run of the
  expensive ground truth, and the proxy-vs-truth gap is itself a reported
  number** ("periodically run the full benchmark and compare its pass rate
  with the fixed-subset estimate to measure the error introduced during
  ongoing monitoring"), plus event-triggered recalibration on material
  changes to models, prompts, tools, or execution system. Both belong in
  the concept's implementation guidance. **Record the caveat alongside
  them:** the paper's own evidence for instrument aging is a null — a
  one-day, 14-run calibration window matched a four-week one — which the
  authors attribute to a mature agent with limited day-to-day change. The
  discipline is justified by construction, not by an observed decay.

- **[[concepts/programmable-evaluator-oracle]] — proposed addition to
  implementation guidance 2.** That guidance currently says "Evaluator
  latency caps iteration count. Wall-clock per evaluation × population size
  × generations ≤ budget." This paper supplies a measured lever that
  guidance does not currently offer: **you can cut evaluator cost by
  subsetting the item set instead of weakening the evaluator**, and the
  fidelity price is quantifiable in advance from recorded per-item
  outcomes. Concrete anchors: 38.5% of items → 1.03 pp MAE and 0.982
  Spearman; 19.3% → 1.97 pp and 0.954. Two caveats that make this a
  qualified addition rather than a recommendation: the technique needs a
  history of per-item outcomes to calibrate against (a cold-start
  evaluator cannot use it), and **the estimator's error must be small
  relative to the selection margins the loop acts on** — which this paper
  cannot establish for itself, because it never reports its pass-rate
  spread. Also worth recording: **item selection can be worse than random**
  (Rasch adaptive at k≤50; both clustering selectors from k=200), so a
  clever subset is not automatically better than a uniform one.

- **[[concepts/pass-at-k]] — proposed addition, small but clean.** This is
  a worked example of the concept's core complaint in a new place. The
  paper's whole contribution is an error bar on a score estimator, and it
  never establishes the **test–retest noise floor**: the MAE between two
  full runs of the same configuration. Without it, "1.03 pp MAE" cannot be
  compared to anything. The generalizable rule for this graph: *an
  estimator's error is only interpretable against the variance of the thing
  it estimates, so report the repeat-run distribution before reporting the
  approximation error.* The paper flags run-to-run non-determinism twice
  (stochasticity requiring repeated trials; "non-deterministic agent
  outcomes resemble flaky tests") and then models every cell as a single
  deterministic binary verdict, which is precisely the gap.

- **[[concepts/spend-forecast-calibration]] — record and decline to edit.**
  There is a structural rhyme: an external predictor fitted on measured
  history (the IRT model) estimating a quantity too expensive to measure
  directly, which is the concept's own prescription. But the object is a
  *score*, not a *spend*, and the concept's load-bearing content — the
  downward bias of agent self-estimates, the one-sided conformal margin —
  has no counterpart here. The one point of contact is a negative and is
  already captured in the Critique: the paper reports a **mean absolute
  error with no uncertainty bound**, and its own Limitations concede it
  never measured decisions at operational thresholds. A gate needs a bound,
  not an MAE. That is a sentence the concept already implies; it does not
  need a new source to say it. Not worth an edit.

- **Not a candidate for a new concept.** "Subset the benchmark to make
  recurring evaluation cheap" is a technique with one production data
  point, no released artifacts, and a deployed variant that its own tables
  show is dominated. It belongs as evidence inside
  [[concepts/hce-evaluation]] and
  [[concepts/programmable-evaluator-oracle]], not as a concept downstream
  projects would import.

- **Direct evidence for the `/elevate` simplicity gate.** The skill's Gate 2
  treats simplicity as a first-class acceptance criterion rather than a
  tiebreaker, and defaults to rejecting net-new complexity. This paper is a
  citable production instance of exactly that judgment, with the sacrifice
  measured: the deployed method is beaten by 2PL adaptive on MAE, Spearman
  and Kendall at **all twelve** budgets, costing +0.37 pp MAE at k=200
  (1.40 vs 1.03) and +0.68 / +0.41 / +0.18 pp at the other three deployed
  budgets. The decisive reason was architectural, not statistical —
  adaptive testing "require[s] sequential orchestration, run-specific
  state, and repeated ability updates" and would "avoid additional
  sequential serving logic in an evaluation system designed to execute
  questions in parallel." That is the simplicity gate's actual shape: the
  better method lost because adopting it meant changing the surrounding
  system, not because the numbers were close. Two secondary reasons are
  also reusable — a fixed subset "reveal[s] the workload before execution"
  (predictable, forecastable cost, cf.
  [[concepts/budget-as-ceiling]]) and executing identical questions every
  run "supports direct question-level comparisons and diagnosis," i.e. the
  simpler instrument is also the more *diagnosable* one. The honest
  counterweight for any proposal citing this: the paper never tested a
  **batched** adaptive scheme, so it did not actually rule out a
  middle option.

- **Cited work worth a future digest.** Five efficient-evaluation papers
  none of which are in this graph, and the cluster is coherent enough to be
  worth a single sweep rather than five: **Polo et al. 2024,
  tinyBenchmarks** (ICML) — the direct parent of the fixed-subset methods
  here and the paper this one *fails to replicate* on IRT-feature
  selection, which makes it the highest-value read; **Truong et al. 2025**
  (ICML), the amortized model-based adaptive-evaluation procedure this
  paper adapts and extends to 2PL; **Kipnis et al. 2025, metabench**
  (ICLR), a sparse IRT-derived benchmark; **Perlitz et al. 2024, Efficient
  Benchmarking (of language models)** (NAACL); and **Vivek et al. 2024,
  Anchor Points** (EACL). All are peer-reviewed, which this paper is not —
  if the graph wants a credible anchor for benchmark-subsetting, it should
  come from these rather than from here. Separately, **Kapoor et al. 2026,
  Holistic Agent Leaderboard** (ICLR) is cited for agent-evaluation
  infrastructure and is squarely on this project's theme. Confirmed absent
  from this graph as of ingest, as are all five efficient-evaluation
  papers above.
