---
kind: paper
title: "CADS: Measuring the Incremental Contribution of LLM Proposers in Sequential Biological Screens"
authors: ["Guanxing Chen", "Nihui Shao", "Haohuai He", "Zhenchao Tang", "Pengwei Hu", "Zhi-An Huang"]
institutions: ["City University of Hong Kong (Dongguan)", "The Hong Kong Polytechnic University", "Tencent AI Lab", "Xinjiang Technical Institute of Physics and Chemistry, Chinese Academy of Sciences"]
year: 2026
venue: "arXiv 2610.24597 (cs.AI), v1 2026-10-17, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.24597"
code_url: null     # the PDF describes shipped verifiers, tests and seed-level shards (revision/, experiments/20260717-*) but names no URL
citations: null
source: "raw/papers/chen2026cads.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["attribution", "proposer-removed-arm", "no-llm-control", "null-arm", "displacement", "origin-attribution", "feedback-ablation", "e-process", "e-bh", "fdr", "huber-contamination", "adaptive-sampling", "optional-stopping", "biological-screen", "depmap", "semi-synthetic", "qwen2.5-7b", "mistral-nemo", "llama-3.1-8b", "non-llm-prior", "matched-quota", "delivery-vs-quality", "failed-replication", "post-hoc-primary"]
---

# CADS: Measuring the Incremental Contribution of LLM Proposers in Sequential Biological Screens

## TL;DR

A simulated gene-screen loop. An LLM proposes genes to assay. A non-LLM
policy re-reads candidates that read high, and a seeded random draw fills
any slot the LLM leaves empty. The paper scores each loop's declared hits
with a contamination-robust e-process / e-BH certificate (FDR ≤ q). It
then measures the LLM as the paired difference against the **same loop
with the proposer removed**. Three findings on one DepMap-backed simulator,
three 7–8B open-weight models and 20 paired seeds:

1. **The standard feedback gate passes with no LLM in the loop.** The
   no-LLM loop certifies 2.60 true hits with full feedback and 0.00 with
   absent or permuted feedback (paired 95% interval [1.35, 3.85]). The
   re-selection rule responds to feedback by itself.
2. **Origin counts overstate the LLM because its picks displace the
   fallback's hits.** Qwen's loop certifies 2.95, of which 2.15 are
   LLM-originated. But the no-LLM loop gets 2.60 on its own, so the net is
   +0.35 [−1.05, 1.75].
3. **A competent non-LLM prior beats every LLM.** A truth-isolated
   pan-essentiality ranking nets +10.05 [8.69, 11.41] over the no-LLM loop.
   The best LLM nets about +3. The one LLM effect that excluded zero, an
   informed prompt moving Qwen from +0.35 to +2.90, did not reproduce on
   50 fresh seeds at a matched quota (+0.74 [−0.12, 1.66]).

## Claims

- Raw hit counts "conflate a proposer's value with the discovery capacity
  of the underlying algorithmic harness." The right estimand is "net gain
  over an LLM-free loop, measured between separately certified sets."
- "Only the head-to-head control gives the net." Origin attribution cannot,
  because "an LLM-selected hit may displace a hit that re-selection or
  random fallback would otherwise find."
- Passing the full-vs-none-vs-permuted feedback gate "is consistent with a
  real proposer, an absent proposer, and a duplicate-ridden one alike."
- Listing already-tested candidates in the prompt changes *delivery* (how
  many valid unseen candidates the model returns) in all three models. It
  does not establish a stable net magnitude.
- The certificate controls terminal FDR under arbitrary dependence,
  adaptive allocation and optional stopping. This holds under a declared
  one-sided Gaussian clean null and a per-read conditional contamination
  bound. The authors say the novelty is "the screen contract and its use
  for paired measurement, not new FDR-control theory."
- Recommendation: "report certified net gain over matched no-LLM
  baselines, distinguish delivery from candidate quality, and treat
  cross-loop contrasts as measurements rather than FDR-controlled
  discoveries."

## Methods

- **Screen contract (C1–C5).**
  - C1: the candidate family is registered before the campaign.
  - C2: the proposer sees only identifiers and a protocol-released summary
    of past reads. In the LLM runs this summary is high- and low-reading ID
    lists, never values.
  - C3: the certificate reads the numerical channel, "not the agent's
    account of it."
  - C4: the clean null is N(µ, s²) with µ ≤ 0 at a stated s.
  - C5: per-read contamination is at most ε̄, conditional on the past.

  C1–C3 are enforced by construction and covered by contract tests. C4–C5
  are "the lab's to justify." Reference effects and labels sit in a private
  oracle that is used only after the terminal declaration.
- **Certificate.** Each read becomes a capped likelihood ratio
  `e = min(L, C*)`. The cap C* is solved so that the worst-case expectation
  under ε-contamination equals 1. Per-candidate products form an e-process,
  and unobserved candidates get E = 1. e-BH runs over the *complete*
  registered family m, not the observed subset.
- **Optional Audit-CADS.** Blinded controls estimate ε, with flag
  sensitivity κ derived from a declared corruption-support class. It needs
  extra independence and mismatch assumptions.
- **Validity sweeps.**
  - Registry: 6,000 DepMap 26Q1 genes. Simulated reads are N(u_i, 0.5²),
    and with probability ε an upward U(2, 4) is added.
  - Loop: 15 continuation rounds of "re-read top 70%, add 500".
  - Seeds: 1,000 paired seeds on the primary 5 × 3 (ε, q) grid, and 200 on
    each of 8 cross-reference settings (Perturb-seq, PRISM, 6 DepMap
    lines).
  - Comparators: naive BH (a failure diagnostic), uncapped LR e-BH, and two
    Zhou–Liu robust-bandit variants.
- **Proposer measurement (frozen contract for Qwen2.5-7B and
  Mistral-Nemo).**
  - Registry: the full 18,159-gene registry with 1,822 reference hits. It is
    never compared with the 6,000-gene sweeps.
  - Loop: 12 rounds × batch 40, giving 480 reads over 260 fresh slots, with
    re-observation fraction 0.5 and ε = q = 0.10.
  - Arms: 3 feedback modes (full / none / permuted) × 2 prompts (thin /
    informed) × seeds 0–19.
  - Thin prompt: does not say which genes were already tested, so the model
    re-proposes them and they score as duplicates.
  - Informed prompt: lists the tested genes and asks for unseen ones.
  - Under-delivery is filled by a seeded label-free random draw.
  - Llama-3.1-8B (4-bit GGUF) was added later on the same protocol and is
    reported as an extension.
  - The no-LLM control is the same loop with a proposer that returns
    nothing.
  - The pre-specified gate: paired 95% t intervals on certified true hits
    exclude zero for both full−none and full−permuted.
- **Harness validation.** A CPU replay re-derives every certified
  declaration from the stored responses for all 40 thin campaigns, with
  zero mismatches. The no-LLM control and the origin split therefore run
  on the same declaration code.
- **Matched-quota follow-up (App. G).**
  - Pre-specified in a design file frozen before the held-out seeds. It is
    shipped with the code but not lodged with a third-party registry.
  - Seeds 20–69 (50 fresh seeds).
  - A candidate-level noise tape, so every arm sees the same read for the
    same (candidate, visit).
  - A common fallback priority.
  - A fixed quota of K = 5 valid unseen candidates per late round, set on
    seeds 0–19 because Qwen-thin delivers a median of 0.
  - Bootstrap intervals, Holm-corrected across the three models, plus a TOST
    test at a margin of one declaration.
- **Non-LLM prior.** Each gene's mean effect across the 5 DepMap 24Q4
  lines other than the target, fixed before any read. The proposer fills
  fresh slots with the top unseen genes by this prior and is otherwise
  identical to the LLM arms.

## Results

- **Certificate validity (Table 1, 1,000 seeds, ε = q = 0.10).**

  | Method | FDR | Power | True | False |
  |---|---|---|---|---|
  | CADS | 1.33e−4 | 0.396 | 245.7 | 0.033 |
  | Naive BH | 0.372 | 0.816 | 506.9 | 300.8 |
  | Uncapped LR e-BH | 0.251 | 0.711 | 441.8 | 147.9 |
  | Zhou–Liu evidence | 0 | 0 | 0 | 0 |

  - Across the grid, CADS's worst FDR is 6.73e−4.
  - Power at q = 0.10 falls from 0.476 at ε = 0 to 0.152 at ε = 0.30.
  - Across the 8 references, CADS FDR is ≤ 4.51e−4 and power is
    0.189–0.741. Naive BH runs 0.151–0.548.
  - Realized FDR is about 750× below q. The authors read this as "a loose
    certificate in this signal-to-noise regime".
  - The only valid comparator (Zhou–Liu) declares nothing at the matched
    budget of 45,899 reads. Its published burn-in alone needs 3.5M.
- **Where the guarantee breaks (App. E–F).**
  - At ε̄ = ε the cap is exactly tight. A null mean shift of +0.05, a null
    sd 10% low, or a same-mean, same-variance null with kurtosis 9.7 each
    push E_null[e] above 1.
  - Setting ε̄ = 0.20 absorbs a +0.10 shift or a 1.5× scale error.
  - Six adversarial contamination channels at the same marginal rate hold
    the bound (worst FDR 0.0095). One of them fires only on candidates the
    agent chose to re-read.
  - A clustered channel (one coin per campaign) violates the conditional
    bound. FDR rises two orders of magnitude to 0.0474 [0.0201, 0.0747],
    which is still below q. The paper claims only the precondition.
- **Proposer grid (Table S8, 20 seeds per cell).**
  - CADS FDR is 0.000 in all 18 cells. Naive BH on the same streams runs
    0.129–0.628.
  - Certified true hits under full feedback:

    | Arm | Thin | Informed |
    |---|---|---|
    | Qwen | 2.95 | 5.50 |
    | Mistral-Nemo | 2.95 | 3.90 |
    | Llama | 5.40 | 5.65 |
    | No-LLM | 2.60 | – |

  - Permuted feedback yields exactly 0 in every seed for every model, so
    every full−permuted interval is driven by its full arm.
- **Displacement (thin prompt).**
  - Qwen fills 57.7 of 260 fresh slots and the fallback fills the other 202.
    The LLM originates 2.15 hits and the fallback finds 0.80. Alone, the
    fallback fills 260 slots and finds 2.60, so the LLM displaced 1.80.
    Net +0.35 [−1.05, 1.75].
  - Mistral-Nemo: 2.35 + 0.60, displaced 2.00, net +0.35 [−1.53, 2.23].
- **Prompt effect (20 seeds).**
  - Qwen's net goes from +0.35 to +2.90 [1.26, 4.54], a prompt effect of
    +2.55 [1.40, 3.70]. The bootstrap agrees.
  - Mistral-Nemo: +1.30 [−0.62, 3.22].
  - Llama: +2.80 [0.50, 5.10] thin and +3.05 [1.12, 4.98] informed. Its
    prompt effect is +0.25 [−1.80, 2.30].
  - Qwen was named primary *after* inspecting the estimates, and its
    intervals are uncorrected. The paper says so.
  - Informed delivers 175.8 vs 57.7 slots, at per-slot precision 0.226 vs
    0.215 (+0.010 [−0.037, 0.057]).
- **Matched-quota replication (50 fresh seeds, 700 campaigns).**
  - Thin misses 40.7 / 26.0 / 13.1 of 55 late quota slots (Qwen / Mistral /
    Llama). Informed misses ≤ 0.7.
  - Fixed-quota prompt effects: Qwen +0.74 [−0.12, 1.66], Mistral +0.16,
    Llama −0.64. Holm p ≥ 0.34, and TOST does not establish equivalence.
  - Qwen's natural prompt effect attenuates from +2.55 to +1.10
    [−0.10, 2.28].
  - Rerunning seeds 0–19 under the new noise tape gives +2.00 [0.50, 3.40].
    The authors decline to attribute the drop to seeds or to noise model.
  - Certified per campaign at matched quota:

    | Arm | Certified |
    |---|---|
    | Random | 1.74 |
    | Qwen thin / informed | 3.58 / 4.32 |
    | Mistral thin / informed | 3.42 / 3.58 |
    | Llama thin / informed | 5.08 / 4.44 |
    | External prior | 16.12 |

  - Every LLM informed arm beats random by +1.84 to +2.70.
  - The prior beats each informed arm by 11.68–12.54.
  - The FDR bound over the 700 campaigns, with zero false declarations, is
    a Clopper–Pearson 0.058 at n = 50 seeds. Seeds, not campaigns, are the
    independent unit.
- **Budget.** The no-LLM loop at 1× / 2× / 4× reads certifies 2.60 /
  11.30 / 20.20. Absolute power in the LLM runs is 0.0016–0.0031, roughly
  3–6 of 1,822 reference hits.

## Critique / open questions

- **Against the digest framing: BELOW (mild).** Every number in the
  digest entry is in the paper and stated correctly. The "Why" overreaches
  in one place and the entry omits three things.
  1. **It is not the same shape as the `hce-known-bad-preflight`
     proposal.** The proposal scores a do-nothing *submission* with the
     search evaluator and expects the floor. Landing above the floor means
     the evaluator is wrong. CADS's LLM-free arm is a working *search
     loop* (random fill plus feedback-driven re-reads). It is expected to
     score above the floor, and it does: 2.60, rising to 20.20 at 4× reads.
     That is not an evaluator defect. It is the harness's own contribution,
     to be subtracted from the proposer's. The two arms answer different
     questions: "is the scorer right?" versus "who earned the score?". A
     reader who merged them would flag a sound evaluator as broken. The
     arm in CADS that does land at the floor is permuted feedback, with
     exactly 0 in every seed. That is a no-signal arm like
     [[literature/papers/yao2026explicit]]'s permuted labels, and it tests
     the loop, not the scorer.
  2. **Omitted: a non-LLM prior wins by about 3×.** Net +10.05 against the
     LLMs' +0.35 to +3.05, and 16.12 against 5.08 at matched quota. This
     is the paper's strongest result, and it matters more than the failed
     prompt replication.
  3. **Omitted: the LLM net is not uniformly null.** Llama-3.1-8B nets
     +2.80 [0.50, 5.10] on the thin prompt, and every informed arm beats
     random at matched quota (+1.84 to +2.70, unadjusted). "The +2.90 did
     not persist" is about Qwen's *prompt effect* and its magnitude.
  4. **Omitted: the setting.** It is semi-synthetic, with simulated
     Gaussian reads on public reference effects. The models are only
     7–8B open-weight. Absolute power is about 0.2–0.3%.
- **Check at ingest: how far it transfers outside bio screens.**
  - *Transfers: the proposer-removed arm and the displacement accounting.*
    Any loop in which an LLM fills some slots and a non-LLM mechanism fills
    or re-ranks the rest has this structure. Examples: an `/iterate` chain
    with keep-best selection, evolutionary search with a random-mutation
    fallback, and HPO where the LLM proposes and a default sampler fills.
    In each, crediting the LLM with "its" winners overstates it by the
    winners the fallback would have found.
  - *Transfers: a full-vs-no-feedback ablation does not isolate the LLM.*
    It does so only when no non-LLM component reads the feedback. Keep-best
    on validation reads it. This sharpens zhang2026double's "report what a
    no-LLM rule policy scores in the same harness" (hce-evaluation guidance
    8). The same harness without the LLM is the subtrahend, not a side
    number.
  - *Transfers: delivery vs quality.* A matched *quota* equalises what each
    arm is asked for, not what it returns. This is the
    [[literature/papers/oved2026llm]] currency point in another form:
    match on delivered work, and report under-delivery as a separate
    effect.
  - *Does not transfer: the certificate.* It needs a pre-registered finite
    family, one numerical read per assay with a known clean-null law, and
    a terminal declaration. An ML-research chain has an open-ended
    candidate space and few, expensive, non-Gaussian reads. The paper's
    own App. E shows validity breaks at a 10% sd error when ε̄ = ε. This
    matches [[literature/papers/qu2026propose]]'s conclusion for
    `/iterate`: an anytime-valid referee answers the "is this a real
    discovery?" question, at a latency the box's run counts cannot pay.
  - *Does not transfer: the LLM magnitudes.* 7–8B models on a gene list
    that frontier models have likely seen (pretraining leakage untested,
    as the paper says). A frontier model's "proposal" here may be recall
    of DepMap essentiality, which is what the winning prior encodes.
- **The replication changed four things at once.** Fresh seeds, a
  candidate-level noise tape, a common fallback priority and a quota. The
  authors' seeds-vs-tape check (+2.00 on old seeds under the new tape) is
  honest but untested for difference, and they say so. Read it as "the
  20-seed magnitude was a favourable draw, selected post hoc", which is
  [[literature/papers/hu2026winners]]'s winner's curse at the
  paper-writing level, not only the loop level.
- **The validity sweeps are generous to the method.** The frozen
  contamination is i.i.d., one-sided, cap-saturating and far above the
  audit threshold. The paper says this and runs six unkind channels, which
  is to its credit. The ~750× gap below q means power, not validity, is
  the binding question, and the only valid comparator declared nothing.
  So "CADS controls FDR" is shown. "CADS is a good instrument" is not.
- **Fixed rounds only.** Optional stopping is proved, not exercised.
- **What it does not show.** Any frontier model. Any wet-lab or
  prospective screen. Any ML-research loop. A stable model ranking (the
  paper says this itself).

## Trust signals

- **Credibility:** 3.
  - For:
    - Unusually self-critical. Limitations, App. E–F and App. G report the
      failed replication, the post hoc primary contrast, the uncorrected
      intervals, where validity breaks, and that Zhou–Liu's zero is budget
      incompatibility, not dominance.
    - Artifact integrity is described in detail: one-command validators
      that recompute every mean from seed rows, SHA-256-bound paper
      artifacts, 62 contract tests, and a CPU replay with zero mismatches
      on 40 campaigns.
    - A frozen two-model contract, with the third model labelled as an
      extension. A pre-specified follow-up design on fresh seeds, with Holm
      and TOST.
    - Large seed counts on the statistics (1,000 / 200), plus 1,180
      proposer campaigns.
    - Proofs in App. A for the non-standard pieces.
  - Against:
    - Not peer reviewed.
    - The PDF names no code URL, so the released artifacts could not be
      checked.
    - Semi-synthetic, single simulator, three small open models, 20 seeds
      for the headline.
    - The primary contrast was chosen from the estimates.
    - The qualitative findings are concurrent with Gupta et al. 2025, as
      the paper concedes.
    - An AI-use disclosure covers editing, engineering and review.
  - Not 4: no venue, no checkable code, and a simulated setting with small
    models. Not 2: four institutions including Tencent AI Lab, careful
    disclosures, and the negative replication reported in the abstract.

## Follow-up

- **Relevance:** 3. The instrument is bio-screen-specific. What transfers
  is attribution methodology: a proposer-removed arm as the subtrahend,
  displacement accounting, a feedback gate that a non-LLM loop passes, a
  strong domain prior as the real baseline, and delivery vs quality. It
  adds material evidence to:
  - [[concepts/hce-evaluation]]: the proposer-removed arm, and why it is
    not a known-bad pre-flight.
  - [[concepts/evolutionary-expansion]]: a competent non-LLM prior as the
    baseline, and displacement of the fallback.

  It touches [[concepts/programmable-evaluator-oracle]] (naive BH is a
  correct-for-fixed-n rule that adaptive sampling makes wrong, 0.13–0.63
  FDR) and [[concepts/budget-as-ceiling]] (it agrees with qu2026propose
  that anytime-valid certification is expensive). Neither gets an edit,
  since both points are already in those files. Not 4: no ML-research
  loop, no frontier model, and the certificate does not transfer.
- **For the `hce-known-bad-preflight` proposal.** Neutral, and not
  evidence for it. It neither supports nor weakens the do-nothing check.
  It adds one caution for the proposal's wording. "Do-nothing" must mean
  an empty or constant *submission* scored once. It must not mean a
  proposer-free *loop*: the loop's harness earns real score, and that score
  is the attribution baseline, not the floor. The proposal's text already
  says "submission", so no change is needed. If it is ever widened to
  "null arm", this paper is the counterexample.
- **Candidates** (cited, not yet in the graph):
  - Gupta, Hartford & Liu 2025, "LLMs for Bayesian optimization in
    scientific domains: Are we there yet?" (Findings of EMNLP 2025). It is
    the concurrent source: non-LLM baselines match LLM feedback, and part
    of the hybrid gain comes from showing which candidates were tested.
  - Wainrib et al. 2026, "Can AI scientist agents learn from lab-in-the-loop
    feedback? Evidence from iterative perturbation discovery" (arXiv
    2603.26177).
  - Roohani et al. 2025, BioDiscoveryAgent (ICLR 2025, arXiv 2405.17631).
  - Xu, Wang & Ramdas 2021, "A unified framework for bandit multiple
    testing" (NeurIPS 2021). It is the e-process / e-BH foundation that
    qu2026propose also builds on.
