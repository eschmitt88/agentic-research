---
kind: paper
title: "Are Diversity Metrics Measuring Diversity? A Capability-Controlled Audit of Majority-Vote Gain in LLM Ensembles"
authors: ["Donghwan Kim"]
institutions: ["Aidentyx Inc. (San Jose, CA)"]
year: 2026
venue: "arXiv 2607.20768v1, cs.CL (preprint, 22 Jul 2026; ACL-style with a Limitations section, so likely an ARR submission)"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.20768"
code_url: null   # "planned public release" only; no repository named in the paper
citations: null
source: "raw/papers/kim2026are.pdf"
added: "2026-09-22"
relevance: 3
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["ensembles", "majority-vote", "self-consistency", "diversity-metrics", "capability-confound", "correlated-errors", "construct-validity", "measurement-definition", "selection-gap", "negative-result", "mmlu-pro", "truthfulqa"]
---

# Are Diversity Metrics Measuring Diversity? A Capability-Controlled Audit of Majority-Vote Gain in LLM Ensembles

## TL;DR

Thirty LLMs are run on 500 MMLU-Pro items, all 31,900 size-2–4 subsets are
enumerated, and five "diversity" statistics are audited as predictors of
**majority-vote gain over the best member**. Two results. First, a selection
gap: **oracle gain is positive in 100% of subsets on both benchmarks**, yet
unweighted majority vote beats the strongest member in only **9.98% of
canonical size-3 subsets** (**18.71% ±3.70** when the best member is selected
on held-out items, so the in-sample figure is the pessimistic one). Second, a
measurement result: the standard pairwise contingency-table statistics are
mostly re-expressing member accuracy — ranked best+mean accuracy explains
**98.9%/98.5%** of strict-diversity, **92.2%/88.5%** of disagreement and
**85.7%/83.8%** of double-fault rank variance (MMLU-Pro/TruthfulQA), and the
"strict diversity" proxy is near-collinear with one minus mean accuracy
(**ρ = +0.991 / +0.988**). Three of the five measures are *algebraically*
non-separable (`strict = disagreement + double-fault`), so a joint linear
regression on them is rank-deficient by construction. After control, the only
directionally stable remainder is a modest pairwise co-failure association —
more shared error, less gain (**−0.432** at size 3) — and the paper says
plainly that its magnitude is configuration-dependent (**−0.18 to −0.57**),
near zero in a restricted pool (**−0.038**), and useless as a predictor
(**AUC 0.597**).

**The abstract does not overstate.** This is the rare case where the abstract
is already deflationary and the body is, if anything, harder on the result
than the abstract is. See "Does the body walk back the abstract?" below.

## Claims

- **Latent complementarity is ubiquitous; realized gain is not.** "Oracle gain
  is positive in 100% of subsets on both benchmarks, yet realized
  majority-vote gain is typically negative." The bottleneck is aggregation /
  selection, not candidate quality.
- **Diversity statistics over modern LLM pools are largely capability
  readings.** "Ranked best and mean member accuracy explain 98.9%/98.5% of
  strict, 92.2%/88.5% of disagreement, and 85.7%/83.8% of double-fault
  variation." The paper explicitly refuses to generalize this to all five:
  "the corresponding fractions are 56.4%/42.7% for Jaccard and 45.2%/23.0%
  for focal diversity. We therefore do not claim that all five are
  interchangeable capability proxies."
- **Part of the collapse is algebra, part is empirical, and the paper
  separates them.** `strict = disagreement + double-fault` (max |ε| <
  1.9 × 10⁻¹⁶) and `1 − Acc = DoubleFault + ½ Disagreement` are exact
  identities; so any raw-space linear control that includes mean accuracy
  forces `DoubleFault_res = −½ Disagreement_res` at Pearson r = −1.000. "This
  raw-space one-dimensionality is algebraically inevitable; it is not an
  empirical discovery." What *is* empirical is that the joint-correct rate is
  collinear with mean accuracy in this pool.
- **Raw diversity–gain correlations carry the wrong sign and should not be
  read.** Raw size-3: strict −0.650, disagreement −0.667, Jaccard *error-set
  similarity* **+0.537** — i.e. raw numbers appear to say "diversity hurts,
  overlap helps." The paper attributes this to capability, not to a real
  effect.
- **One residual survives, modestly.** Rank-residual pairwise co-failure stays
  negative under all six linear control specifications, nonlinear decile
  controls, matched/stratified controls, plurality voting, per-subset
  denominators, a less-filtered slice, TruthfulQA, and removal of the upper
  accuracy cutoff.
- **Explicit non-claims.** "We do not claim diversity never helps, that any
  single statistic is uniquely 'correct,' or that our residual signal is
  large. The claims are measurement-level and associational, not causal."
- **Prescription.** "(i) Evaluate against the strongest member; (ii) control
  for capability level and spread, including nonlinear specifications, before
  crediting diversity; (iii) treat contingency-table measures as algebraically
  coupled; (iv) validate selection on held-out items."

## Methods

- **Pool.** 30 benchmark-specific model routes via OpenRouter — Anthropic,
  OpenAI, Google, Meta, Mistral, DeepSeek, Qwen, x-AI and several
  Chinese-origin routes. MMLU-Pro inference collected April 2026 with a July
  2026 retry confined to previously unparsed responses; TruthfulQA collected
  in the July pipeline. Inclusion filter: parse rate ≥ 0.90 and full-500
  accuracy in **[0.40, 0.92]**, an interval the paper calls "a pragmatic
  construction choice, not a literature-derived or formally preregistered
  standard."
- **Subsets.** All size 2–4: 435 + 4,060 + 27,405 = **31,900**. TruthfulQA
  reproduction: 29 models, 338 common items, 27,811 subsets.
- **Target.** `g(S) = Acc_MV(S) − Acc_best(S)`, majority vote under a
  correct-count rule (exact even-size ties count incorrect). Size 2 is
  structurally 0% under this rule and is excluded from gain headlines.
- **Five measures.** Strict diversity `(b+c+d)/N` (the author's own term —
  "not a named measure in Kuncheva and Whitaker's catalogue"), disagreement
  `(b+c)/N`, double-fault `d/N`, mean pairwise Jaccard error-set similarity
  `d/(b+c+d)`, and focal diversity (TOPLA, Tekin et al. 2024) — the last
  deliberately chosen because it is *not* in the algebraic family, so any
  attenuation it shows is empirical rather than forced.
- **Controls.** Six linear rank-space specifications (best; best+mean;
  best+min; best+median; best+std; full sorted accuracy vector), plus
  nonlinear decile-indicator controls and a matched/stratified estimator
  (exact best-member accuracy, mean binned at 0.01, ≥8 subsets per stratum;
  retains 3,551/4,060 size-3 subsets across 134 strata).
- **Inference discipline — a notable choice.** "The 31,900 subsets reuse the
  same 30 models and are highly overlapping; they are not independent
  observations, and subset-level p-values would be inflated. We therefore do
  not report subset-level significance." The evidentiary standard is instead
  direction + effect size + model-level resampling intervals (B = 1,000) +
  leave-one-model-out. The resampling procedure deduplicates drawn models, so
  the author declines to call its percentiles confidence intervals, reporting
  them as "robustness intervals."
- **Common-parse slice.** The analysis slice is the intersection of items
  parsed by all 30 models: **500 → 356 items**, and the filtering is not
  neutral (retained mean accuracy 0.791 / disagreement 0.178; dropped 0.659 /
  0.316; domain composition differs, χ²(13) = 61.65, p = 2.7 × 10⁻⁸). The
  initial 500-item sample itself is not category-skewed (χ²(13) = 19.9,
  p = 0.10), so the shift comes from the parse filter.
- **Pipeline self-audit.** "We audited every gain-computing script in the
  codebase and regenerated all reported numbers from a single consistent
  pipeline; regenerated values match the analysis files to rounding (max
  |diff| ≤ 5×10⁻⁵ over all 31,900 rows)."

## Results

### The selection gap

| gain > 0 (%) | all (2–4) | size 3 | size 4 |
|---|---|---|---|
| in-sample best | 1.27 | **9.98** | 0.00 |
| held-out best (20 seeds) | 3.44 (±1.18) | **18.71 (±3.70)** | 1.23 (±0.95) |

- Size-4's 0.00% is a rule artifact: "size 4 is 0% largely because exact 2–2
  ties count as incorrect." Excluding size-4 tie items gives **8.50%**; an
  answer-plurality rule gives **4.52%** at size 4.
- The pooled **1.27%** is the number a reader is most likely to quote and the
  paper flags it as the least informative: it "mixes structural size-2 zeros
  and size-4 tie effects with the more informative size-3 rate (9.98%)."
- TruthfulQA agrees: **1.09%** pooled, **0.98%** with the upper accuracy
  cutoff removed.
- A descriptive "oracle-gain capture ratio" is reported only in the appendix
  and explicitly not used as a headline: mean **−131.7%**, pooled **−120.6%**.
  Negative, i.e. the vote typically loses ground relative to the best member.

### Capability entanglement

- `strict ≈ 1 − mean member accuracy`: **ρ = +0.991** (MMLU-Pro), **+0.988**
  (TruthfulQA). After best+mean rank-space projection, "only **1.1%** of
  MMLU-Pro and **1.5%** of TruthfulQA strict-diversity variance remains."
- Table 3, size 3 (n = 4,060), Spearman with gain, Raw → | best →
  | best+mean:

| Measure | Raw | \| best | \| best+mean |
|---|---|---|---|
| strict diversity | −0.650 | −0.799 | **+0.339** † |
| disagreement | −0.667 | −0.742 | **+0.292** † |
| Jaccard (pairwise) | +0.537 | +0.530 | −0.226 |
| focal (TOPLA) | −0.537 | −0.532 | **+0.049** |
| double-fault (co-failure) | −0.449 | −0.815 | **−0.432** |

  († the paper's own footnote marks these as full-pool-only and
  roster-dependent.) Size 4 (Table C1, n = 27,405) repeats the pattern:
  double-fault −0.635 / −0.862 / **−0.380**.
- **The sign flips are treated as evidence of confounding, not as findings.**
  "Strict diversity flips to +0.339 under linear best+mean control, shrinks to
  +0.092 under nonlinear control, and vanishes in the 16-model pool (−0.006)."
  And: "a coefficient that reverses under a linear control and nearly
  disappears under a nonlinear one is consistent with residual capability
  confounding or specification sensitivity, rather than a robust standalone
  diversity effect."
- Focal diversity, which is *not* algebraically coupled, still attenuates to
  **+0.049** at size 3 and **−0.138** at size 4 — so the collapse is not
  purely an artifact of the identity.

### The surviving residual, and how small it is

Robustness of the controlled double-fault association (Table C2):

- Direction holds across every specification: best-only −0.815; best+min
  −0.616; best+median −0.587; best+std −0.628; full accuracy vector −0.338;
  nonlinear deciles −0.474; matched/stratified −0.510; per-subset denominators
  −0.418; TruthfulQA −0.553 (−0.570 with no upper cutoff).
- **Magnitude is not stable.** Point estimates "span approximately **−0.18 to
  −0.57** across slice, roster, benchmark, and threshold configurations."
- **Boundary conditions the paper volunteers:**
  - Size-4 MMLU-Pro model-resampling interval **crosses zero: [−0.523,
    +0.054]** (leave-one-model-out stays negative, [−0.431, −0.269]).
  - Non-circular difficulty split (15 models define difficulty, the disjoint
    15 are evaluated): easy **−0.851/−0.747**, medium **−0.855/−0.790**, hard
    **−0.219/−0.182**. The signal is weakest exactly where diversity would
    matter most.
  - Restricted nine-route Chinese-origin pool (n = 84 subsets): double-fault
    **−0.038**, strict **+0.001**. Essentially nothing.
  - Held-out predictive use: "size-3 pairwise co-failure **AUC 0.597**.
    This is a diagnosis of shortfall, not a recipe for winning ensembles."
  - The high-parse 16-model / 451-item slice roughly halves it: **−0.236**
    (size 3) / **−0.195** (size 4).
- **Pairwise, not all-member.** Replacing pairwise double-fault with the
  fraction of items on which *every* member is wrong gives **−0.101** versus
  −0.432 — which the paper uses to distinguish its axis from the concurrent
  "all-member co-failure ceiling" work (Chen, 2026).
- **Item noise attenuates rather than creates the effect.** Split-half
  reliabilities: gain 0.757, double-fault 0.860; attenuation-corrected
  ≈ **−0.535** versus observed −0.432. The author flags the correction as
  approximate because it treats the controls as error-free.

### Does the body walk back the abstract?

Mostly no — the abstract is already hedged ("modest residual," "magnitude is
configuration-dependent," "with one exception, unstable under control"). Two
places where the body is *more* informative than the abstract, and a reader
who stops at the abstract gets a slightly rosier picture of the co-failure
residual:

1. **The headline slice is the one most favorable to the surviving result.**
   Section 5.6 concedes it directly: "the common-parse filter
   disproportionately removes items from a regime in which this particular
   signal is less informative" — 63.2% of dropped items fall in the hard band
   where the association is −0.18 to −0.22. The less-filtered 451-item slice
   gives −0.236/−0.195, about half the headline. The abstract's phrase
   "configuration-dependent" covers this but does not quantify it.
2. **The abstract leads with 9.98% and mentions 18.71% parenthetically.** The
   held-out-selection number is the deployment-relevant one and is nearly
   double.

Conversely, the abstract *understates* one thing: the 9.98% rate is over all
size-3 subsets including hopelessly mismatched rosters (gpt-4o-mini at 49.4%
paired with claude-sonnet-4.6 at 83.0%), so part of what it measures is roster
heterogeneity rather than an intrinsic ceiling on voting.

## Critique / open questions

- **Not an agent paper, and the transfer is by analogy.** Zero-shot
  multiple-choice prompts, frozen per-item correctness matrices, unweighted
  majority vote over answer labels. No trajectories, no tools, no iteration.
  The rule audited — unweighted majority vote on MCQ labels — is the weakest
  plausible aggregator and is not what any research-agent architecture in this
  graph actually uses. The paper is explicit that it audits "that classical
  intuition rather than learned aggregators, routers, or judges."
- **Reproducibility is promised, not delivered.** There is no repository link
  anywhere in the paper — only a "planned public release" of audit/generation
  scripts, the binary correctness and parse-status matrices, item IDs and the
  roster, with "raw prediction caches remain[ing] conditional on
  redistribution constraints." Worse for replication: "Provider selection
  followed OpenRouter's default routing and was not pinned; provider metadata
  beyond the requested route ID was not recorded." The correctness matrices
  are therefore not regenerable even by the author.
- **Exploratory by the author's own admission.** "The analysis was exploratory
  and iteratively refined. The exact accuracy-band endpoints were not formally
  preregistered or literature-derived." With six control specifications, two
  benchmarks, two sizes and a dozen robustness slices, the space of reportable
  numbers is large; the defense is that the *direction* is invariant across
  all of them, which is a reasonable defense but not a preregistered one.
- **The parse filter is doing real work and is partly a prompting artifact.**
  144 of 500 items are dropped because at least one of 30 models failed to
  emit a parseable label. The July retry re-queried unparsed cases "with
  progressively simplified prompts, so the final correctness matrix mixes a
  small number of recovered responses with the initial prompting regime." The
  clean sensitivity check — initial responses only — is impossible: "the
  pre-retry intersection of items parsed by all 30 models is only **18
  items**."
- **The measurement result is stronger than the causal one and should be cited
  that way.** "Ranked best+mean accuracy explains 84–99% of the trio's
  variance" is robust, cheap to check, and largely algebra-plus-one-empirical-
  fact. "More shared error → less gain" is, as the paper concedes, both
  unsurprising ("errors shared by multiple members directly reduce the cases a
  vote can recover") and weakly estimated. Cite the first; treat the second as
  a direction.
- **Near-total collinearity makes the residual analysis structurally thin.**
  Limitation 3 says it outright: "only 1.1%/1.5% of its rank variance survives
  best+mean control on MMLU-Pro/TruthfulQA; estimates on this residual are
  inherently less stable... a structural limit."
- **Open question the paper names.** Whether the diversity–capability
  relationship is prompt-invariant: "All models were evaluated under a common
  zero-shot chain-of-thought instruction. Different prompting or reasoning
  regimes may alter both member capability and error dependence."

## Trust signals

- **Credibility:** 3. On provenance alone this is a rubric-2 artifact — a
  single-author preprint from Aidentyx Inc., a small company with no track
  record in this graph, no peer review, **no released code** (only a planned
  release), no citations, and unpinned OpenRouter provider routing that makes
  the underlying correctness matrices unregenerable. It is lifted to 3 by
  method quality that is well above that floor: algebraic identities verified
  numerically to machine precision, a deliberate refusal to report
  subset-level p-values because the subsets are dependent, six control
  specifications plus nonlinear and matched estimators, model-level resampling
  and leave-one-model-out, a cross-benchmark reproduction on TruthfulQA, a
  non-circular difficulty split, a split-half attenuation correction, an
  item-subsampling stability curve, a pipeline self-audit reproducing every
  number to ≤ 5×10⁻⁵, and a seven-item Limitations section that pre-empts
  nearly every objection a reviewer would raise — including the one that most
  damages its own headline (the parse filter favors the slice where the effect
  is strongest). Held below 4 by the missing code and the unpinned inference.

## Follow-up

- **Relevance:** 3 — one point below the digest's 4. This is useful prior art
  on an active theme, and it is cite-worthy, but it does not shift any concept
  in this project's architecture. It studies frozen multiple-choice answers
  from 30 API models under unweighted majority vote; there are no agents, no
  trajectories, and no research task. Its two transferable facts — the
  oracle-vs-realized selection gap, and "your diversity metric is mostly an
  accuracy metric" — strengthen arguments this graph already makes rather than
  introducing a new one.
- **[[concepts/pass-at-k]]** — the strongest transfer, and the one edit I
  would make. The concept currently argues seed variance and the
  best-of-unreported-many reporting problem ([[literature/papers/xing2026compute]]).
  This adds the cleanest available statement of the *selection* half: latent
  complementarity is not the scarce resource, selection is. "Oracle gain is
  positive in 100% of subsets on both benchmarks," yet the vote beats the best
  member in 9.98% of size-3 subsets (18.71% ±3.70 with held-out best
  selection), and the descriptive oracle-capture ratio is negative (−131.7%
  mean). Note that this measures *majority vote* specifically, the weakest
  selector, and that the author's own companion result generalizes the shape
  to a stronger one: "in closed-loop table recognition, iteration produces
  better candidates that a reference-free LLM judge largely fails to select"
  (Kim, 2026, arXiv:2607.13347).
- **[[concepts/hce-evaluation]]** — a second, smaller edit. Adds a
  construct-validity failure mode not yet recorded there: **a metric family
  that presents as several independent signals can be one signal by
  construction.** `strict = disagreement + double-fault` exactly, and
  `1 − Acc = DoubleFault + ½ Disagreement` exactly, so "joint raw-space linear
  regressions treating strict diversity, disagreement, and double-fault as
  independent predictors are rank-deficient by construction" — any linear
  control including mean accuracy forces `DoubleFault_res =
  −½ Disagreement_res` at r = −1.000. The reusable discipline is the paper's
  own: check the algebra before crediting a residual, and refuse to report
  significance over a population of overlapping, non-independent units.
- **[[concepts/shared-substrate-contagion]]** — a narrower claim than the
  digest proposed; see the framing note below. What this paper actually
  supplies is the *no-channel limit* of
  [[literature/papers/shao2026language]]'s peer-hidden control, generalized
  and quantified. Kim's setting has no channel at all by construction — 30
  models answering independently — and there, agreement structure is still 84–99%
  explained by member capability. So the baseline shao demands is not
  zero-agreement; it is capability-determined agreement, and an observer who
  measures pool disagreement to argue a substrate is or is not herding its
  agents is mostly reading an accuracy gauge. Corollary for this repo's
  reviewer-independence question: swapping model family to buy diversity buys
  less the stronger both models are, which is the mechanism behind
  [[literature/papers/zheng2026engineering]]'s 11.3pp (model swap) versus
  40.9pp (independent source).
- **[[concepts/hybrid-model-backends]]** — record, do not edit. The concept is
  about splitting *roles* across model families (ideator/implementer); Kim
  audits *aggregating* heterogeneous models by vote. Different mechanism, and
  nothing here bears on role-splitting.
- **Digest candidates from the reference list** (none currently in this
  graph). Strongest first: **Goel et al. 2025, "Great models think alike and
  this undermines AI oversight" (ICML 2025)** — peer-reviewed, and its subject
  is exactly the verifier-independence question
  [[concepts/shared-substrate-contagion]] and
  [[literature/papers/zheng2026engineering]] are circling. **Kim et al. 2025,
  "Correlated errors in large language models" (ICML 2025)** — peer-reviewed,
  the empirical foundation of the confound this paper measures. **Donghwan Kim
  2026, arXiv:2607.13347**, the same author's LLM-as-a-judge-as-optimization-
  signal paper, which is far closer to this project's
  [[concepts/programmable-evaluator-oracle]] and
  [[concepts/evidence-gated-completion]] than the ensembling paper is. Lower
  priority: Josef Chen 2026 (arXiv:2606.27288, co-failure ceiling across 67
  frontier models), Ali 2026 (arXiv:2607.17384), Turkmen et al. 2026
  (arXiv:2602.08003).
