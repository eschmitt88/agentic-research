---
kind: paper
title: "Multi-Agent Screening: What Agreement and Acceptance Cannot Tell You"
authors: ["Benjamin Chang", "Dell Zhang"]
institutions: ["University of Cambridge", "XSci (London)"]
year: 2026
venue: "arXiv 2611.12705 (cs.MA), v1 2026-11-09, preprint (5 pp. LNCS-format short paper)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.12705"
code_url: null
citations: null
source: "raw/papers/chang2026multiagent.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/hce-evaluation]]"
tags: ["screening", "incomplete-judgments", "recall-estimation", "partial-identification", "selective-labels", "capture-recapture", "lincoln-petersen", "elusion-test", "audit-sampling", "llm-as-judge", "agreement-vs-correctness", "pooled-evaluation", "e-discovery", "ai-reviewing", "theory", "digest-recall"]
---

# Multi-Agent Screening: What Agreement and Acceptance Cannot Tell You

## TL;DR

A five-page theory note. k agents screen a fixed collection; a rule over
their outputs decides which items go to a costly assessor, whose verdict
defines relevance. Only the passed items are judged. The paper asks what
each kind of evidence can identify *even with unlimited data*, so the limits
are identification limits, not sampling error:

1. **Agreement** (the joint distribution of agent outputs) cannot tell a
   right consensus from a wrong one, however many agents (Theorem 1).
2. **Acceptance** (the assessor's verdicts on the pool) gives precision
   exactly, and recall only a lower bound (Proposition 1).
3. **Overlap** (who nominated each accepted item) gives an *upper* bound on
   recall under a positive-dependence assumption (Theorem 2), and a cap on
   what one more exchangeable agent can add (Proposition 2).
4. **Audit** (assessor verdicts on a uniform random sample of the *unpassed*
   items) pins down the missed share with no assumptions about the agents,
   at a sample size that does not depend on k (Proposition 3).

All four are classical results (selective labels, Manski partial
identification, Lincoln-Petersen capture-recapture, Good-Turing,
Hoeffding / e-discovery elusion tests) restated for an LLM-agent pipeline.
There are no experiments. The worked example (AI reviewers at a venue) uses
invented numbers.

For this box: `/digest` → `/curate` is exactly this setup. The digest is the
screen, `/curate` is the assessor, and the graph currently knows only its
precision.

## Claims

- **Agreement shows consensus, not correctness (Thm 1).** Model agents built
  on similar models as sharing a latent representation Z, with outputs O and
  relevance Y conditionally independent given Z. Then (a) any rule's error
  P(A ≠ Y) is at least E min{η(Z), 1 − η(Z)}, an error floor that stays
  positive at every k unless Z determines Y. And (b) the distribution of O
  does not depend on η, so as η varies the accuracy of the rule can be
  anything in [E min{g, 1−g}, E max{g, 1−g}], which is [0, 1] if A is a
  function of Z. No statistic of the outputs separates a right consensus
  from a wrong one. Dawid-Skene-type label-free methods identify error
  rates only by *assuming* the latent class is Y.
- **Acceptance gives precision, not recall (Prop 1).** With a = P(A=1) the
  pooled share and p = P(Y=1 | A=1) the precision, recall is
  r = ap / (ap + H), where H = P(Y=1, A=0) is the relevant-but-unjudged share.
  The data allow any H ∈ [0, 1 − a], so the tight interval is
  [ap / (ap + 1 − a), 1].
- **Overlap gives an upper bound on recall (Thm 2).** Let agents nominate
  separately and the assessor judge the union. For two disjoint groups B, B'
  with all other agents silent, h10 / h01 / h11 are the shares of relevant
  items found by B only, B' only, both. If Cov(B, B' | Y=1, C) ≥ 0 on each
  split ("positive dependence"), then H ≥ L = max h10·h01/h11 (the
  Lincoln-Petersen quantity) and recall ∈ [ap/(ap+1−a), ap/(ap+L)]. If
  L > 1 − a, the data refute the assumption. Little overlap means many
  misses. Specialist agents covering different parts of the collection can
  break the assumption.
- **The next agent (Prop 2).** If the k+1 agents are exchangeable given Y=1,
  agent k+1 adds new relevant items worth at most f1/k of the collection,
  where f1 is the share relevant and nominated by exactly one of the first k.
  An agent of a new kind is not covered.
- **Audit closes the gap (Prop 3).** Let Ĥ be (1 − a) times the relevant
  share of n0 items drawn uniformly without replacement outside the pool.
  Then |Ĥ − H| ≤ ε with probability ≥ 1 − δ once
  n0 ≥ (1 − a)² ln(2/δ) / (2ε²), for any k. The bound is Hoeffding and is
  sufficient, not necessary. For 1,000 items, a = 0.2, δ = 0.05, ε = 0.05 it
  asks for 473 audits, against 192 from an exact hypergeometric calculation.
- **Take-home rules (§5).** Rate agents on judgments, not agreement. Report
  precision, and recall as an interval. Let agents nominate separately,
  judge the union and log who found each item, "since a threshold on an
  aggregate score can leave nominations unjudged". Compare a next agent
  against an audit before buying either. Budget audits by the accuracy
  wanted, not by the number of agents. An audit can also reject positive
  dependence.

## Methods

- Pure identification analysis. Probabilities are over an item drawn
  uniformly from a fixed collection. Relevance is Cranfield-style: Y = 1 if
  the assessor would accept the item under a fixed standard, observed only
  in the pool.
- Four evidence types (agreement, acceptance, overlap, audit) are analysed
  for the tight interval each implies, meaning the set of values that fit
  the data and the stated assumptions and no others.
- Proofs are two to four lines each and are given in full. Thm 2 adapts the
  dependence-robust capture-recapture bounds of Sun et al. (2023), adding the
  cap H ≤ 1 − a. Prop 2 replaces Good-Turing's independent sampling with
  exchangeability. Prop 3 is Hoeffding without replacement.
- One worked example with hypothetical numbers (§4). No dataset, no
  simulation, no code.

## Results

The worked example is the only quantitative content. I re-derived each
number:

- Venue, 1,000 submissions. Three AI reviewers nominate 200 (a = 0.2). Human
  experts accept 120 (p = 0.6).
- **Agreement alone:** the nominations fit p = 0, 0.6 or 1 equally unless
  reviewers err independently given acceptability (Thm 1).
- **Acceptance alone:** r ∈ [0.12 / (0.12 + 0.8), 1] = **[13%, 100%]**.
- **Overlap:** of the 120 accepted, each reviewer alone found 30, each pair
  alone 6, all three 12 (90 + 18 + 12 = 120). With reviewer 3 silent,
  reviewers 1 and 2 give 30, 30, 6, so at least 30 × 30 / 6 = **150
  acceptable papers were missed** under positive dependence, and
  r ∈ **[13%, 44%]** (0.12 / (0.12 + 0.15) = 0.444).
- **Next reviewer:** a 4th, if interchangeable, adds at most 90 / 3 = **30**
  acceptable papers.
- **Audit:** 192 of the other 800 estimate H to ±0.05 at 95% confidence and,
  under positive dependence, are expected to find at least
  192 × 150 / 800 = **36** missed papers. The venue's extra budget is 200
  reviews, so the audit "alone certifies the misses but uses nearly all the
  budget". The 4th reviewer needs judgments only for its new nominations.
- Prop 3 check: (0.8)² ln 40 / (2 × 0.05²) = 472.2 → 473. Matches.

## Critique / open questions

What the paper does **not** show:

- **No data.** Nothing is measured. The worked example is invented, and the
  paper reports no real screening system's precision, overlap or audit.
  The contribution is a clean framing and four bounds, each a known result.
- **The audit's ε is on H, in collection-share units, not on recall.** At a
  low relevant share this is the wrong scale. In the paper's own example,
  ±0.05 on H is ±50 papers against 120 accepted, so H = 0.15 ± 0.05 still
  leaves r ∈ [0.375, 0.545]. The paper never converts the audit into a
  recall interval. When relevant items are a few percent of the collection,
  the Hoeffding form is useless (see Follow-up: 17,243 audits for ±8 items
  on this box's numbers) and an exact binomial or hypergeometric bound is
  needed. The paper mentions the hypergeometric figure (192) but gives
  neither its formula nor the zero-hit (rule-of-three) case, which is the
  practical one at low prevalence.
- **"Doesn't grow with the number of agents" is true but beside the point
  for cost.** The audit never touches the agents, so k cannot enter.
  Relative precision on recall grows as roughly 1/prevalence, and the paper
  does not discuss that.
- **The assessor is assumed correct and fixed.** Y is "what the assessor
  would accept". Assessor error, drift, and standards that depend on what
  else has been accepted are outside the model. This box's `/curate` declines
  items partly because a stronger in-graph source exists, so its standard
  is relative to the graph's state.
- **Multi-stage screens and stratified audits are a sentence.** "Sampled
  pools need weighting and add variance." A cascade (category filter →
  keyword filter → title read → abstract read → list) is covered only as
  one composite rule π(O). No stratified audit design or allocation is given.
- **Thm 2 depends on an untestable assumption** except through an audit
  (which the paper says can reject it). Positive dependence fails exactly
  when agents specialise, which is the usual reason to add an agent.
- **Thm 1 is in a single-factor latent model.** It shows agreement cannot
  certify correctness *without assumptions*, not that it is uninformative
  under partial independence (kohli2026nine's n_eff ≈ 2 measures that).

## Trust signals

- **Credibility:** 3.
  - For: proofs short, complete and checkable, and every worked-example
    number re-derives. The results rest on standard, well-cited statistics
    (Manski, Lakkaraju selective labels, Fienberg / Sun et al.
    capture-recapture, Hoeffding, Lewis et al. 2021 TAR certification,
    Webber 2013 recall CIs). Dell Zhang is an established IR researcher.
    The claims are scoped to identification, and the abstract does not
    overstate.
  - Against: not peer reviewed (LNCS formatting suggests a conference
    submission; venue not named). No experiments or code. Two authors, one a
    small company (XSci). One self-citation to a book from a press tied to
    the authors.
  - Not 4: no empirical validation, no venue, no artifacts. Not 2: theory
    of this kind needs no code, and the derivations hold on inspection.

## Follow-up

- **Relevance:** 4. It is the missing measurement design for the
  `/digest` → `/curate` pipeline that
  [[concepts/web-grounded-literature]] describes. The graph currently reports
  only the screen's precision, and the paper says why that cannot bound
  recall and what can. It also gives a theorem behind
  [[concepts/hce-evaluation]]'s "panel unanimity is not a certificate"
  (kohli2026nine) and "self-agreement is not validity" (im2026when). Not 5:
  no new empirical evidence, and every bound is classical. Its value here is
  the mapping, which the box has to do itself.

- **Mapping onto this box** (11-16 digest, `raw/_candidates/2026-11-16-digest.md`):
  - Collection: 9,265 new `2611.` IDs from OAI-PMH `set=cs`. Screens in
    order: eleven-category filter, then keyword rule (845 hits), then all
    845 titles read, 90 abstracts read, 20 listed plus 8 near misses.
  - Pool sent to the assessor (`/curate`): 28 = 20 listed + 8 near misses.
    `_meta/log.md` shows 5 curate runs judging 25–29 items each and
    ingesting 11–17 (pooled 78 / 136 = 0.57 for listed plus near misses).
    That is p, the precision. Recall is unidentified, and Prop 1 gives only
    r ≥ 17 / (17 + 817) ≈ 2% within the keyword hits.
  - The digest's four WebSearch queries are a second "agent". They found no
    in-window paper, so h11 = 0 and Thm 2 gives nothing. If the digest logged
    which channel found each listed item, overlap would become usable.
  - The near-miss list is the paper's "threshold on an aggregate score":
    items read in full and cut by rank. `/curate` does judge them, which
    is the paper's recommendation.

- **How large an audit, at this box's numbers** (exact hypergeometric, 95%;
  computed for this note, not in the paper). Take R_in ≈ 17 accepted per
  digest and N = 817 unlisted keyword hits.
  - *Prop 3 as stated (Hoeffding, a = 28/845):* ε = 0.01 (±8 missed items)
    needs 17,243 audits; ε = 0.05 (±42 items) needs 690. Unusable at a ~2%
    relevant share.
  - *One-sided certificate, zero hits found:* recall ≥ 0.33 needs 66
    judged; ≥ 0.50 needs **124**; ≥ 0.68 needs 231; ≥ 0.81 needs 368.
  - *Two-sided, true recall 0.5 (17 missed):* n = 100 → [0.24, 0.85];
    150 → [0.28, 0.77]; 300 → [0.35, 0.68]; 400 → [0.39, 0.63]. ±0.1 needs
    about half the stratum.
  - *Pooled:* 50 per digest over six digests (n = 300 of ~4,900) gives
    [0.33, 0.73] at recall 0.5 and [0.54, 0.99] at 0.8, about the same as
    300 at once. This is the affordable form if recall is stable over weeks.
  - Cost: an abstract judgment is a few hundred tokens, so 50–125 per digest
    is well under 100k tokens against the 5M ceiling. Abstract-level
    positives must still go to the full-text `/curate` standard. Abstracts
    read above their PDFs (98 of 116 in the 11-03 count), so abstract
    negatives are probably true negatives, and abstract positives would
    otherwise inflate H.

- **Which population.** Both, stratified, because the keyword filter is
  itself a screen.
  - *Stratum A, the 817 unlisted keyword hits.* Highest prevalence, cheapest
    signal. It estimates the recall of the reading stage given the filter.
    By the paper's Prop 1 logic, misses outside A can only add to H, so
    this is an **upper bound on end-to-end recall**, not the recall.
  - *Stratum B, the 8,420 non-hits* (9,265 − 845, including the
    category-filtered records). Prevalence is presumably well under 1%.
    Certifying with zero hits that the filter loses ≤ 17 relevant items per
    digest needs ~1,290 judged; ≤ 34 needs ~690. Not affordable per
    digest at abstract level. Pooled at ~100 per digest, it reaches a
    recall ≥ 0.5 certificate in about a quarter.
  - Weight the strata by their sizes when combining (the paper's "sampled
    pools need weighting"). Papers that never enter the harvest (stat.ML-only
    records outside `set=cs`, venue papers not on arXiv) are outside both
    strata, so even a full audit bounds recall relative to the harvest.

- **Assessor hygiene.** The audit reader should apply `/curate`'s standard,
  not the digest's. By Thm 1, a second pass by the same model in the same
  context agrees with the digest without showing it was right. Mix a few
  listed items into the audit batch, unmarked, so the reader cannot tell
  what was cut.

- Open: a recall estimate is relative to `/curate`'s graph-dependent
  standard, so an item can become "relevant" once the graph changes.
  Record the audit verdict with the graph commit it was judged against.
