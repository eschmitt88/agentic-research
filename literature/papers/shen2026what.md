---
kind: paper
title: "What Eviction Destroys: A Restore-Counterfactual Audit of Forgetting in Agent Memory"
authors: ["Chen Shen"]
institutions: ["Megagon Labs"]
year: 2026
venue: "COLM 2026 Workshop on Context Beyond the Window (arXiv cs.CL)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.08279"
code_url: "https://github.com/megagonlabs/restore-counterfactual"
citations: null
source: "raw/papers/shen2026what.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["memory", "eviction", "forgetting", "retrieval", "counterfactual", "diagnostics", "longmemeval", "negative-result"]
---

# What Eviction Destroys: A Restore-Counterfactual Audit of Forgetting in Agent Memory

## TL;DR

When a memory store is capped below the history size, lost accuracy has
two causes with opposite fixes. Either eviction **destroyed** the evidence,
and only more retention helps, or the evidence **survived and the retriever
missed it**, and a better retriever helps. A budget–accuracy curve can't
tell these apart. The paper's *restore counterfactual* reinstates each
question's gold evidence and reruns the same frozen reader. It then sorts
every oracle-answerable error into **irreversible**, **recoverable**, or
**residual** (still wrong with the gold present). On LongMemEval-S, among
restore-fixable errors, destruction is **0.67–0.73** for FIFO, random, and
redundancy-aware eviction at 80k tokens, **0.60** for LLM-importance, and
**1.00 for every policy at 8k**. At matched accuracy, the tested policies
show no detectable difference. The paper presents an instrument, not a
ranking of policies.

## Claims

- **An accuracy drop conflates destruction with retrieval misses.** "An
  all-recoverable frontier and an all-irreversible frontier can look
  identical on the accuracy axis but require different interventions."
- **Destruction is most of the fixable loss.** Every evicting policy's
  two-bin share irr/(irr+rec) exceeds 0.5 at 80k. Only LLM-importance has
  a CI that crosses 0.5. The share rises to ≈1.00 as the budget tightens.
- **Frontiers are not comparable unless the retrieval regime is reported.**
  Forced-gold reading empties the recoverable bin by construction. Top-k
  reading does not. The same policy and budget therefore decompose
  differently under the two regimes.
- **At matched accuracy, no policy dissociates.** Across 0/9 baseline pairs
  and 0/6 LLM-importance comparisons, no irreversible-rate difference is
  detected at a resolution of 1.2–6 pp. A deliberately destructive control
  *is* detected in 9/9 contrasts. The paper reports this as "no dissociation
  detected", not as equivalence.
- **Residual errors are reader-utilization failures.** They are mostly
  cross-session counting and summation, and they shrink with a stronger
  reader.
- **Under tight budgets, retention comes before retrieval.** "A better
  retriever offers little benefit until more evidence is retained."

## Methods

- **Estimator.** restore_gain(q) = acc(restored) − acc(policy).
  irreversible_loss(q) = max(0, restore_gain) · 1[G_q ⊄ S_P]. Reader,
  prompt, T = 0 decoding, judge, and ranker are held fixed across the two
  conditions.
- **Denominator.** *Oracle-answerable policy errors*: questions the reader
  gets right under clean full-gold injection but wrong under the policy.
  Each error falls into exactly one bin. Irreversible: the restore flips it
  and at least one gold unit was evicted. Recoverable: the restore flips it
  and all gold survived. Residual: still wrong after the restore.
- **Data.** LongMemEval-S (cleaned release, sha-pinned). Histories are
  ≈102k tokens. The 470 evidence-labelled questions are used and abstention
  questions excluded. After the oracle filter, N = 336 (332 for the
  LLM-importance grid, which was run separately). Units are unmodified
  turns and sessions; summary stores are explicitly out of scope.
- **Policies.** FIFO, random (3 seeds), redundancy-aware, and
  LLM-importance. The last is a frozen GPT-4o-mini rater giving a general
  1–10 score that **never sees the question**. There is also a no-evict
  reference and an information-density positive control that evicts gold
  at ≈3× the baseline rate. No policy reads gold.
- **Budgets and regimes.** Store caps of 80k, 30k, and 8k tokens, all
  binding. Regime (a) forces surviving gold into context. Regime (b) is
  pure top-k (k = 60) from a frozen **BM25-lite** lexical ranker with
  recency tiebreak. Injection is capped at 2,000 tokens. A restore forces
  the full gold set, then fills the rest of the cap with ranker results,
  ordered chronologically.
- **Models.** GPT-4o-mini is both reader and judge. GPT-5.4-mini is a
  robustness reader. GPT-5.5 is an independent re-grader on a stratified
  sample of 396 items.
- **Statistics.** The protocol was frozen before measurement. The paper
  uses a question-cluster bootstrap (10k resamples) with Holm–Bonferroni
  within families. H1 (share > 0 under forced gold) is a construct check.
  H3 is the regime (b)−(a) recoverable delta. H2, the matched-accuracy
  test, was **amended after the freeze** because the original statistic
  was degenerate. The paper flags this.
- **Tightness checks on the 2,276 irreversible cases.** (i) Restore only
  the *surviving* gold. (ii) Inject the *entire* retained store. (iii) Run
  an equal-length non-gold placebo at the gold position.

## Results

- **Table 1, top-k regime, primary reader.** Each cell gives err / irr /
  rec / res and the two-bin share:
  - no-evict: 95 / 0 / 38 / 57 → 0.00.
  - FIFO: 80k 124 / 55 / 22 / 47 → 0.71 (CI .61–.81); 30k → 0.99; 8k
    299 / 262 / 0 / 37 → 1.00.
  - random (pooled, N = 1,008): 80k → 0.73; 8k → 1.00.
  - redundancy-aware: 80k → 0.67; 8k → 1.00.
  - LLM-importance: 80k 102 / 34 / 23 / 45 → 0.60 (CI .47–.72); 8k → 1.00.
  - control: 80k → 1.00.
- **Irreversible rate irr/N** (the share of all answerable questions lost
  for good). FIFO: 0.16 at 80k, 0.59 at 30k, 0.78 at 8k. LLM-importance:
  0.10 / 0.59 / 0.80. Control: 0.61 / 0.82 / 0.88.
- **Three-bin shares are much smaller.** Counting residuals, irreversible
  errors are only ≈0.40–0.44 of errors at 80k for the three baselines, and
  residuals are ≈0.38–0.41.
- **Layout matters.** Single-session questions are ~95% irreversible.
  Multi-session and temporal-reasoning questions carry 23–34% residual.
- **R2, regime gap.** The recoverable-share difference (b)−(a) at 80k is
  FIFO +0.29 (CI .19–.39), random +0.27, and redundancy-aware +0.33, all
  p_adj = .007. Random at 30k adds +0.019. That makes 4/12 evicting cells.
  The gap sits at 80k because tighter budgets destroy evidence rather than
  leave it unretrieved.
- **R3, matched accuracy.** Baseline pairs differ by |Δrate| ≤ 0.036 with
  every CI spanning 0. LLM-importance leads at 80k (accuracy 0.69 vs ≈0.62,
  irr-rate 0.10 vs 0.15–0.17), but those pairs fall outside the 0.05
  caliper and don't count. At 30k and 8k, where accuracy is matched, it
  does not separate from the baselines. The null replicates with
  GPT-5.4-mini.
- **Tightness.** A surviving-gold-only restore reclassifies **0.3%** (7).
  87% of irreversible cases had no surviving gold. Injecting the entire
  retained store (mean coverage 1.00) recovers **2.0%** (45). The placebo
  flips **8.3%** (190), against 100% for the gold restore.
- **Judge.** GPT-5.5 agrees at 95.7% (κ = 0.90; κ = 0.83 on restored
  answers). The disagreements are mostly GPT-5.5 grading more strictly
  (14 cases). Pre-run self-consistency was 1.0.
- **Reader robustness.** GPT-5.4-mini gives identical Holm counts (12/12
  H1, 4/12 H3) and the same policy ordering. Residuals more than halve
  (no-evict @80k, forced gold: 59 → 28), and the ones fixed are exactly the
  aggregation errors.

## Critique / open questions

- **"Destruction is the majority" is true on a denominator that leaves out
  most errors.** At 80k, FIFO makes 124 errors, but no-evict already makes
  95 (38 retrieval misses, 57 residual). Eviction's *marginal* cost at 80k
  is ≈29 errors, about 9 pp on the answerable slice. The abstract's
  0.67–0.73 is irr/(irr+rec). The three-bin figure (≈0.40–0.44) is
  disclosed only in the Fig. 1 caption and §5.1. Both numbers are correct,
  but the headline invites the wrong one.
- **The recoverable bin is a property of a weak ranker.** BM25-lite with a
  2,000-token cap misses 38/336 answerable questions *with nothing
  evicted*. A dense or reranked retriever would likely shrink the
  recoverable bin and push the two-bin share toward 1.00 at 80k too. The
  0.67–0.73 is therefore not a constant of eviction. The R2 conclusion
  still holds: report the retrieval regime.
- **The residual bin is partly distraction, not only utilization.** A
  residual question is, by the oracle filter, one the reader answers
  correctly from *clean gold alone*. It fails when the same gold arrives
  with top-k filler packed to the cap. The paper attributes residuals to
  cross-session aggregation, which fits the examples. Read against the
  setup, though, they are also a measured cost of injecting unrequested
  context alongside the needed evidence. This is this note's reading, not
  the paper's.
- **The null covers question-blind policies only.** All four arms decide
  without the question, which is zhu2026lossy's write-before-query barrier
  in its purest form. Dependency-aware or task-aware eviction
  ([[literature/papers/hao2026selfgc]],
  [[literature/papers/semenov2026beyond]]) was not tested. So "no policy
  dissociates" says nothing about whether *keying on future use* helps. The
  paper scopes the null accordingly: "specific to the tested policies and
  budgets."
- **Evaluation discipline is good and worth copying.** The protocol was
  frozen before measurement, and the degenerate H2 statistic was caught
  and amended in the open. A positive control establishes that the test
  can fire. A placebo controls for placement. CI half-widths are reported
  as resolution in place of a claim of equivalence, and a TOST is
  explicitly deferred.
- **Reader and judge are the same model**, and the only independent judge
  is from the same provider. The authors say so. With κ = 0.90 this is
  probably fine for binary QA grading.
- **Deployability is limited.** The audit needs gold evidence labels, so it
  is a benchmark instrument. One benchmark, one ranker, two OpenAI readers.
  The "policies" are policy classes, not published systems (EMBER, OSL-MR,
  mem0 are not run).
- **The restore is not a pure add.** Forcing gold ahead of ranker results
  under a fixed cap also displaces non-gold units. The 8.3% placebo figure
  bounds that effect, and the upper-bound framing (A3) is honest.
- **The paper suggests a use it doesn't pursue: auditing mandated
  deletion.** Under privacy erasure, the same decomposition prices whether
  a required deletion is recoverable or irreversible task loss.
- **Open question.** Run the instrument over a store that *claims* lossless
  offload (fold with a pointer, content-addressed recall). The irreversible
  bin should be empty by construction, which makes a non-zero count a
  direct test of the claim.

## Trust signals

- **Credibility:** 3. Single author from an industry NLP lab (Megagon
  Labs). Accepted at a COLM 2026 *workshop*, so not a peer-reviewed
  main-track venue (`peer_reviewed: false` per the rubric). No citations
  yet; it was posted 2026-09-08. The method sits at the upper end of 3:
  **code, per-question records for both readers, CI tables, and the
  calibration report are released** (repo resolves). The protocol was
  frozen before measurement with a disclosed post-freeze amendment, and
  the paper includes a positive control, a placebo, and three tightness
  ablations. Held at 3 by the single benchmark, a single weak lexical
  ranker, a same-model reader/judge, question-blind policies only, and a
  headline framed on the favourable denominator.

## Follow-up

- **Relevance:** 4. It adds material new measurement to three existing
  concepts and seeds none. The load-bearing contributions are (i) the
  **budget-dependent split** between destruction and retrieval misses,
  (ii) a **negative result**: question-blind importance scoring does not
  change irreversible loss at matched accuracy under pressure, and
  (iii) a **reporting requirement** for any budget–accuracy comparison.
- **Independence: not a companion to [[literature/papers/shen2026revoked]].**
  The 09-14 digest and that note's last Follow-up bullet (corrected
  2026-09-15) described this as "same first author". The PDFs disagree. This paper is by **Chen Shen,
  Megagon Labs**. shen2026revoked is by **Yi Ting Shen**, Kentaroh Toyoda,
  and Alex Leung at **Vulcan Research, AIFT (Singapore)**. They are
  different people at different institutions, working on different
  problems (eviction loss vs revocation enforcement), and should be
  counted as independent. The two notes share a surname and a week, not a
  programme.
- **Convergence on "retention loss can't be repaired downstream".** There
  are now three groups:
  - [[literature/papers/goyal2026does]] (LinkedIn): 80% of the NOTES
    deficit is construction loss, and store-only repair misses 90% recovery in all 48
    cases.
  - [[literature/papers/zhu2026lossy]]: Unverifiable Omission Rate of
    14.7–23.3% on LoCoMo, ~30% on LongMemEval.
  - This paper: the entire retained store recovers only 2% of irreversible
    cases.

  The mechanisms differ (compression, summary tier, capacity eviction), and
  so does the conclusion each supports: once it's gone, no read-side policy
  gets it back. This paper cites neither of the other two, so the agreement
  is independent.
- **[[concepts/lossless-context-offload]].** This is the price of breaking
  the invariant, measured per question. The contrast with
  [[literature/papers/mason2026missing]] is instructive. Mason's
  deliberately minimal FIFO runs at a 0.0254% fault rate *with* a fault
  path, while FIFO here, with no address left, loses 16% of answerable
  questions at 80k and 78% at 8k. The workloads differ, so this is not a
  controlled comparison. Combined with R3, it still supports the concept's
  claim that the invariant is the more settled and more important thing to
  import than the choice of eviction rule. The paper cites Mason as
  "context-paging employs restores as a serving mechanism", so it knows
  the contrast.
- **[[concepts/context-eviction-policy]].** It partly measures that
  concept's open question "when to evict vs. when to retrieve". The concept
  defines itself on the in-context buffer. This paper evicts from a
  budgeted *store* read through top-k into a 2k-token context, so the
  mapping is structural, not exact. The R2 reporting rule is the same kind
  of measurement discipline as that concept's "count garbage collection
  separately from paging".
- **[[concepts/selective-memory-retrieval]].** It qualifies "the bottleneck
  has moved from acquisition to access" ([[literature/papers/lee2026minteval]]).
  Which one binds depends on how much was retained. The residual bin also
  bears on the concept's always-on distraction failure mode (see the
  critique above).
- **For this repository.** The `raw/` immutability rule is the
  no-evict-with-addressable-store arm, which is why curation loss here is
  recoverable by re-reading `raw/` rather than irreversible. The applicable
  lesson is R2: a claim that a memory design "loses X" is uninterpretable
  without saying how it was read back.
