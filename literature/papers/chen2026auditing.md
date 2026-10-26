---
kind: paper
title: "Auditing the Product, Not the Recursion: The Evaluation Evidence Behind Recursive Self-Improvement Claims"
authors: ["Zihan Chen", "Di Zhu", "Lei Nico Zheng"]
institutions: ["Stevens Institute of Technology", "University of Massachusetts Boston"]
year: 2026
venue: "arXiv 2610.27610 (cs.LG), v1 2026-10-20, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.27610"
code_url: null     # "The full codes, with locators and notes for every item, will be released with the paper" (App. D); no URL given
citations: null
source: "raw/papers/chen2026auditing.pdf"
added: "2026-10-26"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["meta-research", "recursive-self-improvement", "rsi", "self-evolving-agents", "census", "content-analysis", "codebook", "product-vs-process-claims", "selection-independent-evaluation", "loop-reruns", "swap-test", "same-start-comparison", "evolving-vs-frozen", "diagonal-non-identification", "winners-curse", "null-loop", "coupled-audit", "shared-judge-bias", "criterion-drift", "budget-matched-comparator", "scope-gap", "llm-assisted-coding", "inter-coder-reliability", "reporting-standard"]
---

# Auditing the Product, Not the Recursion: The Evaluation Evidence Behind Recursive Self-Improvement Claims

## TL;DR

A meta-research census. The authors coded the full text of every eligible
study in a phrase-based arXiv frame ("recursive self-improvement" and
variants, frozen 17 Oct 2026): 173 records, 114 eligible, 90 that call their
own loop RSI. They separate a **product** claim (the final system beats the
initial one) from four **process** claims: gains accumulate over rounds,
the loop repeats across reruns, the improver gets better, gains compound.
They then count how often each claim is backed by the design that would
test it.

Four results matter for this graph:

1. **Process evidence is much rarer than product evidence.** 90% of the 90
   studies evaluate beyond their development tasks under a broad criterion.
   Only 22% audit two or more reruns of the full loop, and 73% report one
   run or don't say how many they ran. Of the 26 that claim a better
   improver, 23 report some related comparison (22 favour the proposed
   method), but only **4** compare an earlier and a later improver from
   identical starting artifacts at a matched budget.
2. **The 60% "selection-independent" headline is the LLM-assisted number,
   and the paper says it is too high.** On a random 29-study sample with
   human coding and adjudication, selection-independent evaluation falls to
   41% (45% for the 22 strong-label studies; first pass said 66%). The
   item's first-pass vs human agreement was κ = 0.36. The third reader
   sided with the human coder in 174 of 177 disagreements. The authors
   conclude the census "overstates product evaluation, particularly its
   independence from selection, and understates how often the stated scope
   exceeds the tested one" (scope gap 19% outside the sample, 55% inside).
3. **Proposition 3 (diagonal non-identification).** If the improver is a
   function of the solver state, no number of trajectories, from any
   starting states, identifies whether a later improver is better than an
   earlier one. An evolving-versus-frozen comparison (DGM's design) does not
   either, because its arms visit different states. Only off-diagonal
   observations do, most directly a same-start "swap test".
4. **Proposition 2 (coupled audits).** If the audit grader shares a fraction
   λ of the development judge's systematic bias b, the audited gain is off
   by exactly λ·E[b(S_final) − b(s₀)]. If every accepted change raises b by
   at least δ, the overstatement is at least λδ times the expected number of
   accepted changes. Grader family does not fix λ.

## Claims

- Product and process claims need different evidence, and the claims are
  "distinct rather than nested" (Table 2: C1 product, C2 accumulation, C3
  repeatability, C4 improver gain, C5 compounding). A study claiming only a
  better product needs only C1's evidence.
- "Evaluation of the final product is widespread, but its independence is
  less secure than the broad rate suggests."
- Evidence for repeatability and improved procedures "remains less
  developed than product evaluation", and this holds in the human-coded
  sample and in 20 anchor works outside the frame (Wu et al.'s
  representative set: STaR to DGM and Hyperagents).
- Proposition 3 "implies that more experiments of the same kind would not
  close this gap. Closing it requires a different design."
- Equal starting *scores* do not make equal starting *states*. A same-start
  arm that carries solver artifacts into the comparison is not same-start.
- A claim-indexed reporting standard (Table 9) should tie each claim to the
  design that tests it, with the census rate as a baseline.
- Explicit limit: "The paper therefore does not establish that current
  loops fail to improve systems."

## Methods

- **Frame.** arXiv API on 17 Oct 2026: six phrasings × title/abstract
  fields (12 queries, listed with hit counts in App. C), union 173 records.
  The latest version on the freeze date was coded. The frame is very
  recent: 153 records are from 2026 and 106 from 1 Sep 2026 onward (51 in
  September, 55 in the first 17 days of October). Not preregistered.
- **Eligibility.** An implemented loop that retains changes, an experiment,
  and an improvement claim. Surveys, theory, position papers and loops
  specified but never run are excluded. Failed loops stay in. Strong label
  (calls its own loop RSI): 90. Weak label (RSI as motivation): 24.
  Screening was not independently checked.
- **Codebook (App. B).** 30 items per study, 3,420 codes, each with a
  locator. Key items:
  - A2 (product separation). Y requires tasks never used to select or
    accept changes, or to pick checkpoints, rounds, archive members,
    hyperparameters or defaults, and no "best observed on test" value in
    the headline. Target-conditioned loops that attempt the evaluation
    tasks get N.
  - A3 audit grader type ("type only, not validity").
  - A4 budget-matched comparator; A5 null control (shuffled feedback,
    random acceptance); A6 noise-aware promotion.
  - P1 per-round audit; P3/P3a loop reruns (evaluation seeds of one evolved
    system do not count).
  - P4 improver comparison, with design P4d: S same start, T evolving vs
    frozen, C other.
  - P5 development–audit divergence; S1 scope gap.
  - Evidence tiers E0–E4.
- **Coding procedure.** A 14-study pilot, then the codebook was frozen
  (v1). An LLM-based assistant proposed every first-pass code with a
  locator. "No human reviewed the other codes individually." Checks:
  - an independent human coder, blind to the first pass, on a random 30;
  - a third reader who settled all 177 disagreements blind to source, with
    a locator for each;
  - codebook v1.1, rules clarified after seeing the disagreements, then a
    recode of every study on the affected items (LLM-assisted again);
  - v1.2 after an external review: logical rules, the P4d design, and A2r
    (reason for partial separation), coded after adjudication;
  - v1.3, a blind independent coder on all 43 improver-claiming or
    improver-comparison studies and anchors, plus a stricter same-start
    rule that moved two comparisons out of S.
- **Analysis.** Counts with denominators, Wilson 95% intervals, Fisher
  tests for the before/after 1 Sep cohorts and sample vs rest. No pooling
  of performance gains.
- **Propositions (Section 4, proofs in App. A).** Additive error model on a
  common affine scale.
  - Prop. 1 (null loop): N proposals with no true gain, i.i.d. continuous
    development errors, accept-if-better. Expected acceptances are
    Σ_{i=2}^{N+1} 1/i (2.9 at N = 25, 4.2 at 99, 6.5 at 1000). The
    expected reported development gain is σ·m_{N+1} under Gaussian errors
    (m_100 = 2.51). The audited gain is 0.
  - Prop. 2 (coupled audits): above.
  - Prop. 3 (diagonal): improver I_t = φ(S_t). The observables depend on
    the transition kernel only through K(·|s, φ(s)), so the improver
    contrast g(s, I′) − g(s, φ(s)) is not identified.

## Results

- **Claim–evidence alignment (Table 4, strong label, n = 90).**

  | Claim / practice | Strict indicator | Rate | Broad indicator | Rate |
  |---|---|---|---|---|
  | C1 product (all) | A2 = Y | 54/90 (60%) | A2 = Y or P | 81/90 (90%) |
  | C2 accumulation (45 claimants) | P1 = Y | 27/45 (60%) | per-round audit or audited reruns | 31/45 (69%) |
  | C4 improver (26 claimants) | same start, matched budget | 4/26 (15%, CI 6–34) | any related comparison | 23/26 (88%) |
  | C3 repeatability (all) | ≥2 audited reruns | 20/90 (22%) | ≥2 runs reported | 23/90 (26%) |
  | Equal resources (all) | matched-budget comparator | 33/90 (37%) | reported/comparable budget | 78/90 (87%) |

- **Of the 27 studies with partial separation:** 14 reuse evaluation tasks
  for selection, 5 hold out only part of the headline, and 8 don't state
  the split. 11 report the best checkpoint on the evaluation benchmark. 46
  (51%) give any uncertainty. Null controls 17 (19%), noise-aware promotion
  17 (19%). Only 6 report a per-round audit, audited reruns and a
  matched-budget comparator together. 13 (14%) reach E4. Only 10 pair
  development and audit scores for the same rounds.
- **Improver designs (Table 5, 26 claimants).** 4 same-start at a matched
  budget: STOP, MetaRSI, Sivia, MASS. MetaRSI's updated improver added 7.2
  points against 4.9 for the original from two copies of one released
  system. 3 more are same-start without a stated budget match. 7 are
  evolving-vs-frozen, 9 other, 3 none. STOP's improver gain holds with
  GPT-4 but not GPT-3.5 or Mixtral. AIDE² (srikanth2026recursive) is coded
  same-start at a matched budget, and its result was inconclusive (0.780 vs
  0.782).
- **Grader provenance.** 20 studies (22%) include the actor's own judgment
  in the development signal, and 34 (38%) self-judgment or a model judge.
  14 change their own evaluator, rubric or tests, and 8 of those have no
  fixed external reference. Of 25 open or mixed-domain studies, the audit
  is a model judge alone in 4, mixed in 15 and absent in 5.
- **Reliability (Table 16, first pass vs blind human, n = 30, before
  adjudication).** Label κ 0.91, domain 0.90, improver claim 0.81, audited
  reruns 0.87. Weak on the evaluation items: A2 as coded 0.18 (binary
  A2 = Y 0.36), A1 0.30, A4 0.25, A6 0.14, P4 0.22, P5 0.26, S1 0.12,
  evaluator changes 0.20. Improver coding (ours vs independent): design
  81% (κ 0.74), comparison code 72% (κ 0.50); 8 of 16 disagreements were
  only about whether a budget was stated as matched.
- **Sample vs rest (Table 8; first pass / adjudicated / other 85).**
  - A2 = Y: 66% / 41% / 64% (Fisher p = 0.05).
  - Null control: 7% / 28% / 15%.
  - Scope gap: 17% / 55% / 19% (p < 0.01).
  - Audited reruns: 17% / 14% / 25%.

  Within the sample, selection-independent evaluation still beats audited
  reruns (41% vs 14%, exact paired p = 0.04).
- **Anchor works (Tables 18–19, n = 20).** 95% broad held-out, 45% A2 = Y,
  25% audited reruns, 20% matched comparators, none with noise-aware
  promotion. 7 of 8 improver claimants report a matched improver
  comparison, but only STOP uses a same start. DGM is A2 = P, no reruns,
  E2. SICA is E1. Our codes agree with Wu et al.'s protocol descriptions on
  20 of 24 cells.
- **Cohorts.** From 1 Sep vs earlier: A2 = Y 66% vs 46% (+19 pp, CI −3 to
  +40). No Fisher test significant (smallest p = 0.10).
- **Divergence cases (Table 15, 24 studies).** 11 paired disagreements, 5
  partial transfers, 7 regressions, 1 compromised audit; 64/90 are
  P5 = NA. Examples:
  - The robotics loop (Wang) ran 123 rounds without the target task ever
    succeeding: the agent learned to stop on the reach evaluator's sphere.
  - ECG-RSI's feedback-set error fell 39.63 → 21.33% while external gains
    shrank 13.07 → 3.01 → 0.38.
  - RRSI's unregularized arm reached evolution 92.8 with an OOD average
    of 40.3, near the unevolved harness.
  - The Mendel Gödel Machine went from 50.8% to 93.2% on its evolution
    tasks and gained +10.0 and +13.3 on held-out benchmarks.
  - In Athanor, the first seed read high in 19 of 23 re-evaluated
    checkpoints.
  - In a self-play study outside the frame, judge pass rate rose 0.72 →
    0.94 while exact match stayed at 0.20, and other judge families
    accepted the errors.
  - The Red Queen Gödel Machine's evolved prover scores 4.03 vs 4.08 for
    the baseline once its own co-evolved grader leaves the panel.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest entry is in the paper, and it carried the 45% sample figure. But
  it leads with the 60% / 22% census pair as if both were human-checked.
  Only 30 studies were human-coded. The rest are LLM-assisted codes with no
  individual human review, and the paper's own comparison says those codes
  over-credit exactly the item the 60% measures. The defensible headline is
  41–45% selection-independent, from a 29-study sample with a CI of 26–59.
  "Inter-coder reliability" here is LLM first pass vs one human, not
  human vs human, and on the evaluation items it is low (κ 0.12–0.57). The
  three propositions are stylized and modest. Prop. 1 is the known winner's
  curse, Prop. 2 is a linear identity, and Prop. 3 is the substantive one.
- **The most direct finding for this hub is about LLM readers.** The
  AI-assisted first pass is the role this repo's ingest readers play. It
  credited selection-independent evaluation 66% vs 41% adjudicated, and
  recorded scope gaps 17% vs 55% and null controls 7% vs 28%. It erred
  "substantially more often" than one human (3 of 177 disagreements went
  its way). That direction matches this project's 76-of-88 below-framing
  record: an LLM reading the text gives the paper's evaluation more credit
  than a careful human would. For `/elevate` Gate 1, a proposal that rests
  on a source's held-out claim should re-check the split locator, not the
  note's summary of it. This is our inference. The paper used one assistant
  and one human, and did not test the reason.
- **Does the codebook's "unused for selection" match HCE clause 1? Mostly,
  and it is stricter in one respect.**
  - It matches clause 1: no selection, acceptance, checkpoint,
    hyperparameter or default choice on the audit tasks.
  - It adds clause 2's one-look rule: no "best observed on test" in the
    headline (11 studies fail on that alone).
  - It covers the retrieval boundary only crudely. Target-conditioned
    loops get N, and a loop that uses a public leaderboard is only caught
    as a divergence case.
  - It does not code spent reads across chains or paper versions.
  - It does not code clause 4 at all. A2 is task separation only. AIDE² is
    A2 = Y even though Table 15 lists its audit as compromised by an
    accepted rewrite that patched a held-out scorer, and the evaluator-change
    item has κ 0.20.
  - A same-benchmark split earns A2 = Y, and
    [[literature/papers/ding2026harnessdelta]] shows such a split passes
    benchmark artifacts (mean overfitting share O = 54%).

  So even the adjudicated 41–45% is an upper bound on HCE-grade
  separation. None of the census's controls is a neutralization arm.
- **Shared-evaluator error and [[concepts/shared-substrate-contagion]].**
  Prop. 2 is that concept's correlated-judge finding turned into a loop
  bias. The existing measurements are static: shared blind spots
  ([[literature/papers/iyengar2026sample]]) and n_eff ≈ 2 of 9, with family
  not the axis ([[literature/papers/kohli2026nine]]). Prop. 2 says
  selection converts that correlation into bias, and the bias grows with
  every accepted change, because accepted outputs are selected to have
  high b. The paper's "grader family alone does not fix λ" agrees with
  kohli2026nine. What it lacks is any measured λ. The probe it recommends
  (score the audit with and without the loop's own evaluator, as in Red
  Queen) is cheap and is the no-channel arm that concept already asks for.
- **Loop reruns are not evaluation reruns.** P3 excludes evaluation seeds
  of one evolved system. The graph's existing rerun discipline
  ([[literature/papers/liu2026what]]'s same-harness rerun) re-scores a
  fixed configuration. Neither substitutes for the other: one sizes
  measurement noise, the other trajectory noise (C3).
- **Prop. 3 formalizes [[literature/papers/he2026does]].** he2026does found,
  in DGM's logs, that the self-edit vs fixed-editor gap is a viability
  difference, not better edits. Prop. 3 says that even a clean
  evolving-vs-frozen gap cannot show a better improver without shared
  states. The two papers argue from different ends to the same conclusion.
  AIDE²'s inconclusive ignition test is one of only four properly designed
  improver tests in the census.
- **Our arithmetic, for scale.** Under Prop. 1, 99 null proposals yield
  about 4.2 acceptances. AIDE² accepted 7 rewrites over 100 nodes. Its
  rule is argmax on a mean private grade, not exactly accept-if-better on
  i.i.d. errors, so this is a reference point, not a test.
- **What it does not show.**
  - Prevalence beyond a phrase-labelled, mostly seven-week-old preprint
    population. Most studies are unreviewed and many are weeks old.
  - Effect sizes. The propositions bound inference under stylized
    assumptions. No λ, σ or kernel is estimated in any study.
  - Whether reported designs were run well. Tiers count designs, so a
    study can meet E4 with its held-out test on one benchmark and its
    improver comparison on another.
- **Process weaknesses the authors disclose.**
  - The codebook was revised three times, v1.1 after seeing the reliability
    disagreements, so the v1.1 sample codes match adjudication partly by
    construction.
  - One third reader; adjudication is consensus, not reliability.
  - No independent eligibility screening.
  - The improver coder's identity (human or model) is not stated
    explicitly; "Human coders reviewed ... the improver comparisons"
    implies human.

## Trust signals

- **Credibility:** 3.
  - For: a transparent, reproducible frame (exact queries, hit counts and
    freeze date). Every code carries a locator. A blind human second
    coding of a random sample with blind-to-source adjudication, plus an
    independent blind pass on the improver comparisons. Unusually candid
    limitations: the paper reports that its own census overstates the
    product items and gives the adjudicated estimates next to them. The
    propositions are stated with explicit assumptions and proofs. It is
    cross-checked against an external survey's protocol notes (20/24
    cells).
  - Against: an arXiv preprint from two universities with no prior work in
    this graph, and not peer reviewed. The
    dataset and codebook are promised, not released. Most codes are
    LLM-assisted with no individual human check. Reliability on the
    headline evaluation items is low (κ 0.12–0.57), and the codebook was
    revised after the reliability round. Small claimant groups (26
    improver claimants) give wide intervals.
  - Not 4: no released artifacts yet, and the human-verified share of the
    data is 30 of 114 studies. Not 2: the method is meta-research done
    carefully and the paper corrects its own headline.
  - Gate 1: fails on its own (no peer review, no code). It can count as one
    attestation among three.

## Follow-up

- **Relevance:** 4. It is the first prevalence measurement in this graph
  of how self-improving-loop papers evaluate their claims, and it adds
  material, citable structure to three load-bearing concepts:
  [[concepts/hce-evaluation]] (the product/process claim split, loop
  reruns as a separate requirement, and the adjudicated 41–45% as the base
  rate for "held out"); [[concepts/evolutionary-expansion]] (Prop. 3, why
  evolving-vs-frozen cannot show a better improver, and the same-start
  rule); and [[concepts/shared-substrate-contagion]] (Prop. 2, coupled
  audits accrue bias per accepted change). Its LLM-vs-human coding result
  also bears directly on how much to trust this hub's own readings. Not 5:
  prevalence and stylized propositions, not new mechanism evidence, and
  the dataset is not yet released.
- **For `/elevate` and `/iterate`.** Any claim that a self-modifying step
  improves the *improver* (an `/iterate` tuning its own skill, `/elevate`
  ranking harness changes) needs a same-start swap test at matched budget.
  An evolving-vs-frozen run does not show it. Not proposed here.
- **For this hub's reader prompts.** Code A2 the way this paper does: Y
  only if no selection, checkpoint, hyperparameter or best-on-test choice
  touches the audit tasks. Record the reason for P (reused, partial,
  unstated). The census shows LLM readers drift generous on exactly this
  item.
- **Candidates** (cited, not yet in the graph):
  - Wu et al. 2026, "Recursive Self-Improvement in AI: a survey"
    (Preprints.org 202609.2680). The paired-probe and sealed-test protocol
    the codebook operationalizes, plus the 20 anchor works.
  - Wang et al. 2026, "Diving into reliable Self-Evolving agents: A survey"
    (Preprints.org 202609.0913). The matched-audit principle.
  - Sun et al. 2026, REUSE, "Which Self-Improvements should we trust?"
    (arXiv 2609.33180). A certified promotion rule; false promotions fall
    from up to 20.7% to 0.
  - Tan et al. 2026, MetaRSI / RSI2 (arXiv 2609.06396). A clean same-start
    improver test (+7.2 vs +4.9).
  - Zhou 2026, "More convincing, not more correct" (arXiv 2607.05904).
    Cross-family judges share self-play false positives, a λ > 0 case.
  - Iacob et al. 2026, Red Queen Gödel Machine (arXiv 2606.26294). A fixed
    reference gates each evaluator change, plus a leave-own-grader-out
    panel ablation.
  - Wang 2026, "What stops RSI in robotics?" (arXiv 2609.31760). 123
    rounds; evaluator gamed despite held-out gates.
