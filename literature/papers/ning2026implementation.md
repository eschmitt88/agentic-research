---
kind: paper
title: "The Implementation Lottery: Auditing Idea Reliability in Automated Research"
authors: ["Jingjie Ning", "Shanshan Zhong", "Xiaochuan Li", "Ji Zeng", "Chenyan Xiong"]
institutions: ["Carnegie Mellon University (School of Computer Science)"]
year: 2026
venue: "arXiv 2607.26587 (cs.MA/cs.AI), v1 2026-07-29, v2 2026-10-04, preprint; ancillary protocol files are named for AISTATS 2027 and include a 'reviewer revision' analysis script"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.26587"
code_url: "https://arxiv.org/src/2607.26587v2/anc"
citations: null
source: "raw/papers/ning2026implementation.pdf"
added: "2026-10-13"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["implementation-variance", "idea-vs-artifact", "selection-noise", "re-implementation", "seed-variance", "icc", "max-vs-mean", "selected-code-premium", "replication-allocation", "task-concentration", "itt", "fidelity-judge", "prespecification", "tabular", "openml"]
---

# The Implementation Lottery: Auditing Idea Reliability in Automated Research

## TL;DR

A research agent judges an idea by the score of one program that implements
it. The paper asks how much that evidence depends on which program got
written. It freezes four "mechanism cards" per task (early stopping, feature
filtering, covariance whitening, RBF classifier, each a task-specific variant
over a scikit-learn gradient-boosting baseline), has DeepSeek Flash write many
independent programs per card under two executors (a capped "Bounded"
process and a ~11-turn "Agentic" SDK session), and scores them against a
shared bank of 12 fresh reference implementations per card. There are 3,048
assignments on 31 OpenML tabular tasks; the main follow-up is 18 tasks.

Three results matter for this graph:

1. **Choosing an idea from one implementation was not detectably worse than
   choosing from five.** All four primary contrasts (max-of-3 vs mean-of-3,
   single vs mean-of-5, two executors) are null, Holm p ≥ 0.56. This is
   absence of evidence at low power, not an equivalence result (see Critique).
2. **The selected code beats a fresh implementation of the same idea, by a
   little.** Re-scoring the retained highest-scoring program against fresh
   programs of its card gives a premium of 0.38 (Bounded) and 0.45 (Agentic)
   accuracy-equivalent points under mean-of-5 selection. The median task's
   premium is 0.03 / 0.02, and one task carries most of it.
3. **Implementation noise is about twice seed noise.** In the follow-up,
   implementation SD is 1.17 / 0.98 accuracy points and seed SD is
   0.46 / 0.45. 78% of programs score identically on all three seeds. The
   between-idea share of variance (ICC) is still high, 0.83–0.92, so on these
   tasks ideas differ more than their implementations do.

## Claims

- **Promote the right unit.** "An artifact result supports reuse of its
  code. Idea-level promotion calls for outcomes from fresh realizations
  under the declared executor and card." Reusing a winner and re-implementing
  its idea are different actions with different expected payoffs.
- **The maximum's reference floor cancels in a shared-reference contrast**
  (Prop. 1). Two frozen policies scored against the same fresh bank differ
  by d^T q, so the policy comparison is unbiased and only its variance
  depends on the bank. This is why paired contrasts are the primary
  endpoint.
- **Max aggregation is optimistic at equal means** (Eq. 7): with J ≥ 2
  draws, the expected maximum exceeds the mean by s·c_J, so a max-of-J
  policy favours high-variance ideas.
- **Seeds vs implementations is a two-stage allocation** (Eq. 8, Cochran):
  K* = (σ_R/σ_M)·√(c_M/c_R). A second seed per program helps only when the
  implementation-to-seed cost ratio exceeds 2σ²_M/σ²_R, about 13 (Bounded)
  or 9.5 (Agentic) here.
- **The retained-code premium needs no within-card replication.** Picking
  the best of one draw per card already selects on program noise
  (expected gain βσ_D c_C, App. A.3).

## Methods

- **Tasks.** OpenML-CC18-style classification, one histogram
  gradient-boosting baseline, CPU only, single predictor. Three cohorts:
  archived (13 tasks, 312 assignments, DeepSeek V4 Pro), initial
  prospective (12 tasks, 768), follow-up (12 continuing + 6 new tasks,
  1,968 new assignments, 480 initial programs reused for selection).
- **Cards and executors.** Four mechanism families, task-conditioned, with
  admission review before any code. Bounded: four model calls, 64K branch
  tokens, six inspections, two repairs. Agentic: fresh Read/Edit SDK
  session, requested 20-turn cap, median 11 turns used.
- **Splits.** Development selection trains on folds 2–9 and scores fold 1.
  Final evaluation trains on folds 1–9 and scores fold 0, three seeds
  (0, 1001, 2001). Test sets range from 53 to 2,000 items.
- **Utility.** Relative gain over baseline minus a fidelity penalty (0.02
  for drift, 0.10 for invalid), −0.10 for failures; reported ×100·baseline as
  "accuracy-equivalent points". ITT keeps every assignment; "faithful"
  populations require the blind DeepSeek Flash fidelity judge's label.
- **Policies.** Single draw, mean or max of 3, mean or max of 5, enumerated
  over every within-card subset (625 / 10,000 / 1 worlds). Loss = reference
  optimum minus the selected card's fresh-bank mean.
- **Prespecification.** Local protocol files with UTC freeze stamps
  (analysis plan frozen 10:51:03 on 2026-10-02, policy weights 12:28:58,
  final evaluation 12:30:11). The follow-up was designed after the initial
  results, and continuing tasks' fold-0 outcomes had been seen. Table 13
  labels each later analysis as post-outcome.
- **Inference.** Tasks are the resampling unit: exact paired sign-flip
  tests and 20,000-draw task bootstraps, with task-deletion ranges and
  paired item bootstraps as secondary checks.

## Results

- **Primary contrasts (follow-up, 18 tasks, ITT).** Max3 − Mean3 loss:
  −0.14 [−0.59, 0.29] Bounded, 0.12 [−0.10, 0.44] Agentic. Single − Mean5:
  0.00 [−0.42, 0.34] and 0.14 [−0.01, 0.31]. Positive/tied/negative task
  counts are 3/9/6, 4/11/3, 5/8/5 and 6/11/1. Holm-adjusted p = 1, 1, 1,
  0.5625.
- **Most tasks contribute nothing.** Seven Bounded and ten Agentic tasks
  have zero loss under every policy. CNAE-9 alone is 61% of Bounded Mean5
  loss, and deleting one task moves that mean between 0.200 and 0.514.
- **Selected-code premium (prespecified secondary, Table 2/39–42).**
  - ITT Mean5: 0.38 [0.08, 0.82] Bounded, 0.45 [0.05, 1.12] Agentic;
    ITT Single: 0.27 and 0.21.
  - Faithful Mean5: 0.29 [0.03, 0.75] Bounded, 0.33 [−0.05, 1.00] Agentic
    (p = 0.43).
  - Medians 0.005–0.052. Under ITT the premium stays positive after every
    single-task deletion (0.20–0.40 and 0.14–0.48).
  - In the faithful population one task carries 74% (gesture, Bounded) and
    97% (CNAE-9, Agentic) of the signed total; deleting CNAE-9 takes the
    Agentic faithful mean from 0.328 to 0.012.
- **Total development-to-fresh overstatement.** Mean5 development gain
  minus fresh-draw gain is 0.61 (Bounded) and 1.12 (Agentic). It splits
  into split transfer (0.24 / 0.68, CIs crossing zero) plus the premium.
- **Variance components (Table 14).** Implementation SD / seed SD in
  accuracy points: archived 1.10 / 0.75 and 2.65 / 0.82; initial
  2.16 / 0.43 and 0.93 / 0.43; follow-up 1.17 / 0.46 and 0.98 / 0.45.
  Within-card program deviations correlate across splits at 0.59 and 0.68,
  so part of an implementation's luck persists to the final split.
- **Ranking flips (Table 30).** Against a leader fitted on six programs,
  a single-draw choice picks a different card in 25.6% (Bounded) and
  19.1% (Agentic) of 6/6 partitions; mean-of-5 cuts this to 17.4% and
  12.3%. A plug-in Gaussian model under-predicts these rates by 0.02–0.06.
- **Replication budget (Fig. 3, Table 27).** Implementation-to-seed
  duration ratios are 2.6 and 3.7, far below the switch points of 13 and
  9.5, so one seed per program minimises fitted variance. At the measured
  budget, 21 programs × 1 seed has fitted variance 0.075 vs 0.120 for the
  12 × 3 design actually used. These are model-based planning values.
- **Focused expansion (exploratory replay).** Expanding only the two
  leading cards to five programs (12 draws) recovers most of Mean5's
  20-draw loss in Agentic (0.322 vs 0.273); in Bounded single draw and
  Mean5 both lose 0.485.
- **Fidelity judging.** Two blind prompts of the same Flash model agree on
  968/984 and 975/984 follow-up assignments, and all 144 mismatched-card and
  144 empty-patch controls get the expected label. One program that both
  prompts called faithful crashed on a wrong scorer signature.

## Critique / open questions

- **Against the digest framing: AT.** The digest entry is accurate and
  already deflationary: it leads with the null primaries, calls the
  premiums small and fragile, and quotes the one-task deletion. Two
  corrections, neither of which moves the verdict. The 0.33 → 0.01 drop is
  the fidelity-conditioned Agentic cut; under ITT the premium survives every
  single-task deletion (minimum 0.14). And the digest omits the most
  transferable numbers: implementation SD ≈ 2× seed SD and an idea ICC of
  0.83–0.92.
- **The nulls are low-power absence of evidence.** Eight to eleven of 18
  tasks are exact ties in each primary contrast, so the sign-flip test has
  only 7–10 informative tasks. There is no equivalence margin or power
  analysis. The Agentic single − Mean5 contrast (0.14 [−0.01, 0.31],
  6/11/1) points toward Mean5. Its faithful-population version is 0.21
  [0.03, 0.44], p = 0.031, which is secondary and unadjusted, and is
  correctly not headlined.
- **Low headroom drives many ties.** Six of 18 follow-up baselines are
  ≥ 98.8% and several are above 99% (pendigits 99.18, wall-robot 99.08,
  banknote 99.28, HAR 99.71). When the baseline is near ceiling, no idea
  can lose much. The headroom split (Table 18) is post-outcome, and its
  below-98.8% subgroup is still null. The null is about these 18 tasks,
  not about idea selection in general.
- **Most-favourable-cut check.** The abstract's premium is ITT Mean5, the
  largest of the four Single/Mean5 × ITT/faithful cuts for Bounded (0.38 vs
  0.27 / 0.10 / 0.29) and the second-largest for Agentic, below only Max5's
  0.52. All cuts are tabulated, with medians and deletion ranges, so the
  headline is a favourable cut that is fully disclosed. The typical task's
  premium is a few hundredths of a point. CNAE-9 has 108 test items, so
  its 5.7-point Agentic premium is about six test items.
- **The "ideas" are small.** Four hyperparameter-scale mechanisms (early
  stopping, a feature filter, whitening, an RBF swap) on top of one
  gradient-boosting baseline. Programs are 20–35-second Flash sessions, not
  research implementations. Implementation freedom is therefore narrow, and
  that may be why the ICC is so high. A real `/iterate` hypothesis gives an
  implementer much more room, so expect the implementation term to be
  larger there and the ICC lower. This is our inference; the paper does
  not test it.
- **Seed noise is unusually small here.** 69–83% of programs give identical
  accuracy at all three seeds, because histogram gradient boosting is
  mostly deterministic and the seed only touches early-stopping splits.
  The 2:1 implementation-to-seed ratio is a floor for stochastic training,
  not a transferable constant. What transfers is the decomposition, and
  the rule that the seeds-vs-implementations split follows K* from measured
  costs and variances.
- **Fidelity rests on one model judging itself.** Generator, card author
  and both judges are DeepSeek Flash. The judges agree with each other and
  catch planted violations, but there is no human fidelity audit. A
  separate runtime check found a "faithful" program that cannot run. The
  ITT primary avoids depending on the label.
- **Several design choices came after results, and the paper says so.**
  The follow-up was built after the initial cohort. Its 12 continuing
  tasks reuse 480 initial programs whose fold-0 outcomes had been seen.
  Premium robustness, transport, calibration and allocation analyses are
  all post-outcome (Table 13). Prespecification is local files with
  timestamps, not a public registry.
- **The writing is hard to audit.** The 8-page main text compresses a
  34-page appendix into dense, idiosyncratic terms ("accuracy-equivalent
  utility points", "cards", "worlds", "banks"), and the paper has no
  limitations section. The AI-use statement says AI tools helped with
  manuscript drafting. Several numbers in this note were recoverable only
  from appendix tables.
- **Open for this graph:** no source yet measures implementation variance
  for a research-scale hypothesis (a new loss, an architecture change)
  with an agent like `/implement`. That number decides whether `/iterate`
  should re-implement a kept idea before crediting it.

## Trust signals

- **Credibility:** 4.
  - For: CMU SCS with Chenyan Xiong as senior author. 147+ ancillary files
    on arXiv: the full harness and evaluator, analysis scripts, 27 test
    files, `reproduce.sh`, protocol documents and an `evidence.tar.xz` with
    assignment-level scores (listing verified on the abs page). UTC-stamped
    freeze records, ITT primary analysis with Holm correction, exact tests,
    task-deletion and item-bootstrap checks, and every policy × population
    tabulated. Planted-violation controls for the fidelity judge.
  - Against: not peer reviewed (the ancillary files point at an AISTATS
    2027 submission). One generator/judge model, toy-scale mechanisms, a
    near-ceiling task set, and a follow-up designed after seeing results.
    The first author's 2026 output is large and heavily self-cited (six
    Ning et al. 2026 citations, including
    [[literature/papers/ning2026scores]] and
    [[literature/papers/ning2026closedloop]]), so this is one group's audit
    programme, not independent confirmation of it.
  - Not 5: no peer review, no independent reproduction, no citations
    yet. Not 3: the release and the disclosure are well above typical
    preprints in this graph.

## Follow-up

- **Relevance:** 4. It is the first source in this graph to separate
  *implementation* variance from seed variance for the same frozen idea in
  an ML-research-agent setting, and to price "keep this artifact" against
  "this idea works". That materially strengthens
  [[concepts/hce-evaluation]] (artifact vs idea claims),
  [[concepts/budget-as-ceiling]] (what the `/iterate` noise band measures)
  and [[concepts/pass-at-k]] (max-of-k vs mean-of-k for selection). It is
  not a 5: the task set is toy-scale and near ceiling, so it gives a
  decomposition and an allocation rule, not transferable magnitudes.
- **Relation to [[literature/papers/wiedmann2026agents]].** Both find
  re-implementation noise above seed noise. Wiedmann measures it as
  within-configuration spread over whole agent runs (54% of variance).
  This paper isolates it for a frozen idea spec with a shared reference
  bank, and finds it about 2× the seed SD, with ideas still dominating
  (ICC 0.83–0.92). Read together: the `--seeds` band understates
  hypothesis-level noise, by an amount that depends on how much freedom
  the spec leaves the implementer.
- **Relation to [[literature/papers/hu2026winners]].** Hu measures the
  winner's curse in a *sequential* keep loop on reused items. This paper
  measures it *across implementations* of a fixed idea on a fresh split.
  The two overstatements add: development-to-fresh gaps of 0.6–1.1 points
  here, on top of whatever the loop's own item reuse contributes.
- **For `/iterate`:** when a cycle keeps a change, the stored score
  certifies the *artifact*. Before the chain reports "idea X helps", one
  fresh re-implementation of X, scored on the held-out split, measures the
  premium directly. At these cost ratios, spend replication budget on
  extra implementations at one seed rather than on extra seeds of one
  implementation. With σ_R/σ_M ≈ 0.45, K* = (σ_R/σ_M)·√(c_M/c_R) stays
  below 1 unless an implementation costs more than about 5 seeds.
- **Candidates** (cited, not in the graph):
  - Ning, Li, Kong, Li 2026a, "WhatWorkedBench: Benchmarking experimental
    understanding in AI agents" (arXiv 2609.27490).
  - Ning & Li 2026, "Same agent, different answers: a repeat-aware audit of
    corpus-induced answer churn in RAG" (arXiv 2608.22856).
  - Bertran, Fogliato, Wu 2026, "Many AI analysts, one dataset: navigating
    the agentic data science multiverse" (arXiv 2602.18710).
  - Rabanser et al. 2026, "Towards a science of AI agent reliability"
    (ICML 2026).
  - Mustahsan et al. 2025, "Stochasticity in agentic evaluations:
    quantifying inconsistency with intraclass correlation" (arXiv
    2512.06710).
