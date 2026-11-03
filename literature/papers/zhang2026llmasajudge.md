---
kind: paper
title: "LLM-as-a-Judge Version Shift: The Instrument Changed, Not the Standard"
authors: ["Guijia Zhang", "Harry Yang"]
institutions: ["The Hong Kong University of Science and Technology"]
year: 2026
venue: "arXiv 2610.32986 (cs.CY), v1 2026-09-07, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.32986"
code_url: null    # the PDF says the 14,882 calls, six locks and scripts are released, but gives no URL
citations: null
source: "raw/papers/zhang2026llmasajudge.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["llm-as-a-judge", "judge-version-change", "evaluator-drift", "measurement-invariance", "anchor-audit", "pre-deployment-check", "resolution-collapse", "rank-instability", "recalibration", "isotonic-calibration", "placebo-control", "information-ladder", "preregistration", "tost-equivalence", "test-retest-floor", "soundnessbench", "research-proposal-scoring", "api-snapshot", "prompt-sensitivity", "revision-loop", "negative-result"]
---

# LLM-as-a-Judge Version Shift: The Instrument Changed, Not the Standard

## TL;DR

The paper treats an LLM judge's version as a measuring instrument. It
freezes three same-family open-weight judge transitions by revision hash:
Qwen 3.5→3.6-27B, Gemma 3-27B→4-31B and Mistral-Small 3.1→3.2-24B. Each
pair scores the same frozen pool of ML research proposals from
SoundnessBench ([[literature/papers/ho2026soundnessbench]]). There are 348
objects and 14,882 judge calls.

Three results matter for this graph:

1. **A judge version change is an evaluator change, and its direction is
   not predictable.** Qwen moves 0.31 SD toward leniency on anchors and
   0.33 SD on the pool (both CIs exclude zero, and the shift survives FDR).
   Gemma is stricter on anchors (+0.30 SD) but only +0.11 on the pool, with
   a CI that crosses zero. Mistral 3.2 collapses: 93% of pool scores land
   on one value, 86% of anchor pairs tie, and clean–flawed separation drops
   from 0.73 to 0.24. Its rationales stay object-specific, and its
   defect-listing probe still discriminates. Only the scalar score broke.
2. **Re-fitting a map from old scores to new ones fixes at most the mean.**
   A preregistered two-parameter map fitted on 24 anchors failed its gate
   on both chains. Fitted on the full pool, a linear map absorbs under 8%
   of object-level gap dispersion. On Gemma the gap flips sign by quality
   tertile (+0.80 SD on weak objects, −0.43 on strong ones), which no
   global map can represent.
3. **A small in-domain anchor audit is proposed as a pre-deployment
   check.** It asks, in order, whether the score channel collapsed
   (modal share > 0.5 or separation < 0.30 → do not deploy), whether ranks
   moved beyond test–retest noise (→ rescore or keep the old judge), or
   whether only the origin moved (boundary disagreement > 10% → calibrate
   on anchors). The thresholds were set after the stage-2 data existed.
   On two later API pairs (GPT-5.4→5.5 and DeepSeek v4-flash→v4-pro), the
   frozen thresholds said "healthy, calibrate". Post-hoc inspection agreed,
   and for GPT a 660-call pool rescore confirmed it. Neither pair was a
   failure case, so the "do not deploy" branch has never been tested out of
   sample.

For HCE clause 4 the point is direct. A hash of the judge script does not
pin the instrument. The weights, the prompt wording and, for API judges,
the provider's serving stack are all part of it. The paper notes that
pinned API snapshots are not weight-frozen.

## Claims

- "Newer judges are harsher" is false in both direction and quality. Of
  three preregistered transitions, two move lenient and one collapses, and
  two API pairs add one stricter and one more lenient.
- A judge version is a function `s_v(x) = g_v(w_vᵀ z(x) + b_v)`. A version
  change can move the origin (`b`), the criterion weights (`w`) or the
  slope (`g`). These need different repairs: calibrate, rescore, or
  withhold. A mean-only drift statistic cannot tell them apart. Appendix M
  proves that a global monotone map exists only when the weights are
  collinear.
- The residual gap is not a knowledge gap. Giving both versions identical
  facts or identical defect certificates does not close it, and a matched
  placebo attributes the small fact effect to packet format, not content.
- Rank pipelines are not exempt. Monotone maps leave within-judge Kendall τ
  unchanged, but replacing the judge reorders objects.
- Reliability belongs to an operating point, not a model name. The
  Mistral collapse depends on prompt surface form and on domain. It does
  not appear on code review.
- An anchor audit of about 200 stratified calls classifies the failure
  before the new judge scores anything that matters.

## Methods

- **Judges.** Three same-family open-weight pairs, pinned by revision hash.
  Qwen and Mistral are same-size patches; Gemma is a generation jump with a
  size change. Decoding is T = 0.6, top-p 0.95, with per-call CRC seeds.
  Prompt-repeat reliability is quadratic-weighted κ = 0.86–0.93. All
  output is JSON with a final score line.
- **Objects.** 348 frozen proposal-style objects from 2022–2026 ML
  conference submissions, with reviewer soundness scores and verified risk
  factors (SoundnessBench). They form three disjoint sets: 48 anchors
  (12 clean, 12 flawed, 24 natural), an 80-object intervention pool and a
  220-object expansion pool. "Full pool" means the 300 non-anchor objects.
  62 masked-title variants are rewrites, not extra samples.
- **Information ladder.** Stateless randomized calls under L0 (rubric
  only), L1 (plus a frozen atomic-fact packet built by a third-family
  model, phi-4, identical for both judges), L1P (matched placebo facts),
  L2 (plus a defect certificate distilled from reviewer risk factors) and
  L2P (donor-mismatched certificate matched for length and style). B is a
  commit-first defect listing with no score, and R is a title-verified
  recognition probe. Mistral was added at stage 2 and runs L0, R, B, L2
  and L2P only.
- **Estimand.** `Δ_ver(C)` is the object-clustered mean of (old − new)
  divided by the transition's pooled L0 SD. Positive means the new judge is
  stricter. Object-clustered bootstrap CIs (20,000 draws).
- **Preregistration.** Six internal locks (stages 0a, 0b, 1a, 1b, 2, 3),
  each a SHA-256 freeze of configs, prompts, code hashes, kill gates and
  analysis scripts, written before any judge output of that stage. There
  is no third-party registry; the locks ship with the artifacts. Seventeen
  amendments (A1–G7) are logged in Appendix I, each before unblinding the
  affected endpoint. Every test is marked confirmatory or exploratory in
  Table 4. Nulls get TOST exclusion bounds and an 80%-power MDE, and the
  authors say the margins are data-derived, not pre-set SESOIs. BH-FDR is
  applied post hoc to 11 scalar endpoints.
- **The anchor audit (Table 3, §7, App. F.7, J).** As run on the API
  pairs: the 48 frozen anchors, L0 prompt verbatim, 3 repeats per judge.
  That is 288 calls per pair, 144 of them from the new judge. The frozen
  rule had two thresholds: separation < 0.30 raises the degradation flag,
  and accept-boundary (score ≥ 6.0) disagreement > 10% triggers
  calibration. "About 200 calls" is the paper's round figure, not a run
  count. A stress test (G3) puts the minimum near 24 stratified anchors for
  a healthy judge. Detecting the Mistral collapse needs ≥ 40 anchors on
  separation alone, but 12–32 if the modal-share statistic is added.
  Unstratified selection at n = 8 leaves the health check undefined in 62%
  of draws, so the stratification carries weight.
- **Other studies.** A pool-level same-version retest for all six judges
  (F3, 1,384 calls) separates sampling noise from version change. There is
  a cross-family replacement read on the shared pool (G2), a code-review
  replication on 48 HumanEvalPack objects (G6, 864 calls), a perturbation
  matrix of prompt × temperature on the Mistral collapse (E1, 816 calls),
  a counterfactual edit study (G7), and a six-round revise-against-judge
  loop (App. E, n = 40).
- **Call accounting (Table 16).** 13,646 open-weight and 1,236 API calls.
  The AI-use statement says generative AI built the orchestration and
  analysis code, generated the intervention artifacts and helped draft.

## Results

- **Transitions (Table 2).**

  | Transition | Δ_ver anchors [95% CI] | sep. | Δ_ver pool [95% CI] | τ | top-30% flip | ties |
  |---|---|---|---|---|---|---|
  | Qwen 3.5→3.6 | −0.310 [−0.487, −0.146] | 1.88→3.17 | −0.332 [−0.432, −0.236] | 0.613 | 37% | 18% |
  | Gemma 3→4 | +0.303 [+0.014, +0.607] | 1.00→3.92 | +0.110 [−0.034, +0.258] | 0.506 | 39% | 26% |
  | Mistral 3.1→3.2 | −0.650 [−0.961, −0.339] | 0.73→0.24 | −0.450 [−0.616, −0.290] | 0.175 | 59% | 86% |

  The pool is 220 objects for Qwen and Gemma and 300 for Mistral. After
  BH-FDR, Qwen (p = 0.0004) and Mistral (p = 0.00004) survive, and Gemma
  (p ≈ 0.045) does not.
- **Noise floor (F3).** Same-version pool test–retest τ is 0.70/0.68
  (Qwen old/new), 0.70/0.82 (Gemma) and 0.34/0.76 (Mistral).
  Cross-version τ falls below both floors on every chain. The
  version-attributable excess top-30% flip is +11.9 (Qwen), +34.8 (Gemma)
  and +13.8 (Mistral) points. So most of Mistral's 59% flip is noise from
  the old judge (τ = 0.34), not the version change.
- **Mistral collapse (F.1, Table 10).** 0 of 252 pool calls fail to parse.
  Rationales are object-specific (mean 3-gram Jaccard 0.047). The B-probe
  separates clean from flawed by 0.55 SD for 3.2, against −0.27 for 3.1.
  The collapse is stable across T ∈ {0, 0.6, 1.0}: between-object SD is
  0.31–0.35, against the predecessor's 0.94. A semantically equivalent
  prompt paraphrase partly restores it (SD 0.45–0.55, modal share
  0.90→0.50–0.73) and leaves the old judge almost unchanged. On code review
  the same pair does not collapse (modal share 31%) and moves lenient by
  0.79 SD.
- **Facts and certificates (§6, Tables 5, 6, 14).** Δ_fact = +0.060 SD
  [−0.039, +0.158], n = 128, TOST ±0.143. The content-specific fact effect
  is −0.088 [−0.190, +0.012], TOST ±0.174, and placebo packets alone move
  the gap more than real facts on Qwen. The preregistered certificate
  primary is inconclusive (+0.181 [−0.014, +0.375], p = 0.069). An
  exploratory paired contrast came out significant (+0.214), and its
  preregistered Mistral replication was null (−0.082, p = 0.66, n = 33).
  The fact packets lexically cover only 5.3% of risk factors, against
  29.2% for certificates.
- **Recalibration (§6, Tables 7, 9, 11).** The anchor-fitted two-parameter
  map explains −0.15 of the off-anchor gap on Qwen and −12.5 on Gemma
  (gate: ≥ 2/3). Refit on random pool subsets, Qwen reaches a median
  explained fraction of 0.76–0.81 for k ≥ 40, and Gemma still fails 77% of
  draws at k = 80. On the 258-object non-anchor set, the best mapping MAE
  is 0.811 against 0.833 for identity on Qwen. On Gemma every mapping is
  worse than identity (1.011).
- **Audit, in sample (Fig. 4).** The diagnose-then-act rule matches the
  post-hoc best F1 arm on Qwen and Gemma and raises the degradation flag on
  Mistral. The thresholds were "fixed by convention but formalized after
  stage-2 data existed". The paper calls the 3/3 match "an in-simulation
  demonstration", and its leave-one-transition-out check shows
  "insensitivity, not external validity". Mistral's anchor separation
  (0.243) has a 51% bootstrap flag rate, so separation alone is not a
  reliable detector; modal share is.
- **Audit, out of sample (App. J).**
  - GPT-5.4→5.5 (pinned dated snapshots, provider-default decoding): anchor
    Δ_ver = +0.452 SD [+0.319, +0.572], separation 3.52→3.11, boundary
    disagreement 12.5%, giving "healthy, calibrate". A pool rescore of 660
    calls gives Δ_ver = +0.735 [+0.641, +0.824] and disagreement 32.7%,
    which confirms the trigger. The anchor audit understated the shift. The
    frozen payoff endpoint (top-30% F1) is null by construction (0.727
    both). A post-hoc contrast shows isotonic calibration cuts boundary
    disagreement from 32.7% to 19.1%. Anchor subsample agreement on the
    trigger plateaus near 70%, because 12.5% sits close to the 10% line.
  - DeepSeek v4-flash→v4-pro (a tier swap): −0.42 SD [−0.60, −0.25],
    separation 1.78→2.12, disagreement 16.7%, giving "healthy, calibrate".
    There was no pool rescore.
- **Cross-family replacement (G2).** This is at least as disruptive as a
  within-family upgrade. Location gaps reach 1.6 SD, τ = 0.18–0.56 and
  boundary disagreement up to 59%. Gemma 4 tracks reviewer soundness best
  of the six judges (ρ = 0.59), so the best judge can also be a rank break.
- **Revision loop (App. E, Fig. 5).** Six stateless rounds of
  revise-with-judge-feedback by a neutral third-family reviser on 40
  objects. Fresh-seed gain is real (+0.304 SD [+0.096, +0.527]), but it is
  not judge-specific: judge-feedback minus generic revision on held-out
  judges is +0.062 [−0.174, +0.302], TOST ±0.262. Winner's curse is null.
  An interim n = 9 contrast looked significant and vanished at the full
  preregistered sample.

## Critique / open questions

- **Against the digest framing: below framing (moderate).** The
  descriptive findings hold and are quoted correctly. The operational claim
  the digest leads with does not hold as stated.
  1. **"Correctly flagged two later API judge replacements."** Neither
     pair was flagged as failing. Both got "healthy, calibrate". So the
     out-of-sample test covers the calibrate branch only. The "do not
     deploy" branch rests on one in-sample case (Mistral), with thresholds
     formalized after that case was seen. The GPT verdict was confirmed by
     a pool rescore. The DeepSeek verdict rests on "post-hoc inspection
     agrees". The GPT pair's frozen payoff endpoint was null.
  2. **The order row was not in the frozen rule.** Table 3 has three rows,
     but the API amendment froze only separation and boundary
     disagreement. On the GPT pair, cross-version τ on anchors is 0.85,
     below the same-version floors of 0.89 and 0.92. At pool scale it is
     0.661 against a retest of 0.824, a 9-point excess flip. By Table 3's
     own order row that would mean "rescore", not "calibrate". The paper
     does not discuss this.
  3. **"~200-call anchor audit."** As run it was 288 calls per pair, 144
     from the new judge. The 200 figure is a round number. G3 suggests a
     smaller floor (about 24 stratified anchors for a healthy judge), but
     that floor needs labelled clean and flawed anchors.
  4. **"Global recalibration fixes the mean."** True on Qwen only, and
     only when fitted on ≥ 40 pool objects. The anchor-fitted map failed
     its preregistered gate on both chains. On Gemma no map beats identity.
  5. **"Preregistered."** True in a specific sense: six internal SHA-256
     locks per stage, released (per the paper) with the calls. There is no
     public registry timestamp, no URL in the PDF, and 17 amendments. Some
     amendments are post-draft and the script names (`reviewer2_analyses`,
     `rebuttal_analyses`) suggest a review cycle. All the amendments are
     disclosed and labelled exploratory where they should be.
  6. **"59% of the top-30% selection set changes."** Correct as a raw
     number. But old Mistral's own retest τ is 0.34, and the
     version-attributable excess is about 14 points. The paper says so in
     F3; the digest does not.
- **What the evidence does not show.** It does not show which criteria
  moved: `w` is latent, and the classes are read from signatures. It does
  not show that the audit prevents a bad deployment out of sample, because
  no out-of-sample pair was bad. It does not show that expert-written
  evidence fails to close the gap, since the fact packets are
  model-extracted and avoid evaluative content by design. And it does not
  show judge-specific overfitting under stateful or deep optimization; the
  null holds only at a stateless six-round mild operating point.
- **The instrument includes the prompt.** The Mistral collapse partly
  recovers under a paraphrase that leaves the old judge unchanged. So
  "pin the judge" means pinning weights, prompt text and decoding together.
  It also means a prompt edit can be a version change on its own. This
  ties to [[concepts/hce-evaluation]]'s "identical code is not identical
  measurement".
- **The audit is domain-bound.** The Mistral pair that collapses on
  proposals works on code review. An anchor audit certifies only the
  domain its anchors come from.
- **Fit with the graph.** [[literature/papers/alfaqeeh2026beyond]] swapped
  a judge across vendors and saw pass counts double while group order held.
  On this paper's taxonomy that reads as mostly an origin shift, with some
  order change at node level (cross-judge κ 0.38). That is our reading;
  neither paper tests it. [[literature/papers/iyengar2026sample]] showed that a scored
  judge's operating point moves when the generator changes. This paper
  shows it also moves when the judge changes, and that the move need not
  be a shift a threshold re-tune can absorb.
- **Scale.** Three preregistered transitions and two descriptive API pairs.
  These refute universal claims and do not estimate the rate of drift. The
  Gemma direction is suggestive only. All open-weight judges are 24–31B.

## Trust signals

- **Credibility:** 3.
  - For: weight-frozen pairs pinned by revision hash; staged locks with
    disclosed amendments; a placebo at every rung; same-version retest
    floors at pool scale; TOST bounds on every null, with the caveat that
    they are data-derived; BH-FDR; the paper reports its own failed
    replication (certificates) and a vanished interim result (revision
    loop); scope limits stated plainly; 14,882 calls with per-table
    accounting. HKUST.
  - Against: not peer reviewed; two authors; no code or data URL in the
    PDF despite the release claims; the audit thresholds were set after
    seeing the data; the out-of-sample test covers one branch of the rule;
    API pairs use provider-default decoding and are not weight-frozen;
    generative AI wrote the orchestration and analysis code (reviewed by
    the authors).
  - Not 4 because nothing is linked and the headline protocol is validated
    in sample. Not 2 because the design is unusually careful and every
    null and failure is reported.

## Follow-up

- **Relevance:** 4. It is the first evidence in this graph that swapping a
  judge's weights under a fixed prompt and rubric re-instruments the score.
  It gives a taxonomy (origin / order / resolution) that tells which repair
  can work, and a cheap anchor-audit protocol. It adds material evidence to:
  - [[concepts/hce-evaluation]]: clause 4's hash does not pin an LLM
    judge, and the known-bad preflight has an LLM-judge analogue that
    includes the known-good half.
  - [[concepts/programmable-evaluator-oracle]]: an operating point moves
    when the judge changes, not only when the generator changes, and
    re-tuning a threshold repairs only an origin shift.
  Not 5 because the audit's withhold branch is validated only in sample,
  and the setting is proposal scoring, not an agent loop's metric.
- **Preflight bearing (`hce-known-bad-preflight`).** The anchor audit is
  the known-bad preflight generalized to an LLM judge. It includes the
  known-good half that reviewers asked to restore: clean–flawed separation
  is "known-good above known-bad" on 12 + 12 anchors. It costs about 72–144
  new-judge calls (24–48 anchors × 3 repeats), plus cached old-judge
  scores. That is minutes and cents, cheap enough for chain start. It needs
  a labelled in-domain anchor set, which most ML-research tasks lack. And
  it should run at the final-scoring pass too, because for a served judge
  the "same pinned evaluator" may not be the same instrument. The pending
  proposal dropped the final-pass repeat for deterministic code. That
  reason does not hold for an API judge.
- **Cheap local test.** For any downstream project that scores with an LLM
  judge: freeze 24–48 labelled anchors (half known-good, half known-bad),
  log modal share, separation and Kendall τ against a same-version retest
  at chain start and at the final pass, and record the model ID and prompt
  hash next to the evaluator hash.
- **Candidates** (cited, not yet in the graph):
  - Yang, Hou, Yang 2026, "When the judge changes, so does the
    measurement: Auditing LLM-as-judge reliability" (arXiv 2607.08535).
    This is the closest prior audit.
  - Don-Yehiya et al. 2026, "Mediocrity is the key for LLM-as-a-judge
    anchor selection" (ACL 2026). It covers anchor choice.
  - Li 2026, "Who drifted: the system or the judge? Anytime-valid
    attribution in LLM evaluation pipelines" (arXiv 2606.15474).
  - Liu 2026, "A diagnostic framework and multi-evaluator audit of
    evaluator-driven preference dynamics in self-adapting LLM agents"
    (arXiv 2606.29719).
  - Zhou 2026, "More convincing, not more correct: Self-play reward hacking
    of reference-free LLM judges" (arXiv 2607.05904).
