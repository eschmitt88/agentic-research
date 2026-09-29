---
kind: paper
title: "Discover, Falsify, Revise: Auditing Input-Use Claims from Source Code to Predictive Contribution in Agent-Discovered Cell Models"
authors: ["Mengran Li", "Bo Li", "Chengyang Zhang", "Yang Yan", "Jinfeng Xu", "Zhenchao Tang"]
institutions: ["Sun Yat-sen University", "University of Macau", "Sichuan University", "Zhejiang University", "University of British Columbia", "Tencent AI Lab"]
year: 2026
venue: "arXiv (cs.LG; q-bio.QM)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.27234"
code_url: "https://github.com/limengran98/CellAudit"   # linked from the project page; docs/REPRODUCIBILITY.md; biological matrices and fitted weights are external inputs
citations: null
source: "raw/papers/li2026discover.pdf"
added: "2026-09-29"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["claim-vs-score", "input-use-audit", "falsification", "permutation-reliance", "shortcut-learning", "trivial-baseline", "agentic-model-discovery", "virtual-cell", "perturbation-prediction", "pre-registration", "claim-generalization", "audit-feedback", "self-audit"]
---

# Discover, Falsify, Revise: Auditing Input-Use Claims from Source Code to Predictive Contribution in Agent-Discovered Cell Models

## TL;DR

Agentic model-discovery loops (CellScientist, AIDE, CellForge, HarmonyCell)
generate predictors of cellular responses to a compound + dose + control
profile, and select them on held-out Global PCC. CellAudit asks whether the
selected model actually *uses* the perturbation it claims to use, via three
layers that can each fail independently: (1) **source consumption** (an AST
checker: can the input reach the cited computation?), (2) **fitted
dependence** (do predictions change under matched input replacement on a
frozen checkpoint?), (3) **target-relevant contribution** (does that
dependence lower loss vs. a reference threshold?). The headline: on
BBBC047, the score-selected model (from the authors' own prior system,
CellScientist) reaches held-out Global PCC **0.3153** and is **exactly
invariant** to compound replacement; a control-profile-only predictor gets
**0.3142** (difference +0.0011, 95% CI [−0.0005, +0.0027]). The cited
compound-conditioned cross-attention has one key and one value, so its
softmax weight is constant. On both tasks, the score-based selection rule
picked an exactly compound-invariant model. Feeding audit diagnostics back
to the agent helps in the mean, but at n = 5 paired trajectories the
paired-t intervals span zero.

## Claims

- **Prediction–claim gap.** "Predictive performance alone does not
  establish perturbation use." A registered claim ("this model uses the
  compound") is contradicted if the cited source cannot carry the input or
  if fitted predictions are invariant to replacing it. "Predictive
  performance affects the importance of the case, not the definition."
- **Three non-equivalent questions.** "A cited pathway can be
  mathematically inactive; an executable pathway may have little effect
  after fitting; and input dependence need not provide predictive
  benefit."
- **Discover–falsify–revise.** Score-only feedback lets search "favor
  high-scoring predictors without testing whether they use the inputs
  specified by their designs." The audit can guide revision during
  development, but "final claims are evaluated only after the selected
  source, checkpoint, and audit rules are frozen."
- **Claim generalization is separate from predictive generalization.**
  A fixed design refit on an independently acquired cohort can keep its
  predictive gain and lose support for its input-use claim.
- **Support does not inherit.** "Support for one fit should not be
  inherited by later refits." Observed-support replacements "do not
  establish biological exchangeability, mechanism, or causal effects."

## Methods

- **Fixed executor.** Candidate code cannot touch data loading, targets,
  folds or metrics. Edits are limited to architecture, fusion, readout,
  loss, optimizer and scheduler. Folds 1–2 fit, Fold 3 gives feedback and
  selects, Folds 4–5 are post-search. One disclosed exception: the
  across-policy choice uses Fold 4 and is then refit and tested once on
  Fold 5. Response-blind Bemis–Murcko scaffold folds hold out compound
  families.
- **Discovery LLM.** Every agentic call uses DeepSeek V4 Flash at
  temperature 0.2, with no fallback. Request/response hashes, prompts and
  token usage are logged per call.
- **Prediction-score study.** 80 ten-slot trajectories (2 tasks: BBBC036
  and its parent BBBC047 × 4 policies × 10 seeds), with a matched initial
  model, executor and budget. Selected sources get 5 paired refits.
- **Audit.** 32 fixed input-replacement maps per checkpoint (compound
  derangements within same-control-profile rows, dose preserved). Status
  (qualified / unsupported / inconclusive) compares the 2.5–97.5 effect
  quantiles against a model-specific threshold, the 95th percentile of
  |pairwise differences| among 32 reference scores. A five-refit panel
  needs 4/5 agreement. An input with too few legal interventions is **not
  identifiable**: BBBC047 dose has 0/4380 eligible rows on Fold 4. Known-
  truth fixtures cover 6,400 simulated datasets.
- **Two revision formulations.** *Path-constrained*: an LLM emits a
  design card and a compiler guarantees an explicit compound route.
  *Falsification-guided*: a residual ŷ = g(c) + α·h(c, p, a) over a frozen
  control-only g, with a replacement-loss qualification gate on Fold 3.
  Both change the model space as well as the feedback, and the authors
  say these "test the joint design changes."
- **Feedback study (sci-Plex3).** Score vs. Audit feedback with a shared
  context+dose anchor, 5 paired trajectory seeds, and 5 new slots each.
  Model space, trainer and PCC-first selection are held fixed. Audit adds
  grouped errors, source checks and development-set compound/dose
  replacement effects to the next prompt.
- **Transfer.** Ten LINCS-selected designs are frozen and refit on LKCP
  Batch 2, an independent acquisition. Dose and compound contributions get
  a Shapley split. A Norman CRISPRa double-perturbation task tests a
  bundled pair-identity input.

## Results

- **The headline model is a control-only model.** The BBBC047
  score-selected source scores 0.3035 on Fold 4 and 0.3153 on Fold 5.
  Compound shuffling changes PCC by 0.0000 in every refit, and
  prediction-space distance is exactly zero. Control replacement costs
  0.1932 / 0.2004. The finding survives refitting with physically
  disjoint control wells for input and target reference (0.3085 / 0.3193
  vs. control-only 0.3082 / 0.3185).
- **It is not one model.** In the Fold-4 panel (Table 37), none of the 8
  policy-selected sources passes the compound-use criterion: 6
  unsupported, 2 inconclusive. CellScientist is exactly invariant on both
  tasks, and HarmonyCell is too on BBBC036. The across-policy score rule
  carried an exactly invariant source to Fold 5 on **both** tasks
  (HarmonyCell on BBBC036, CellScientist on BBBC047). On BBBC036 the
  control-only predictor (0.3098 ± 0.0024) matches CellScientist's
  selected model (0.3096 ± 0.0029).
- **The flagship defect is partly a CellScientist artifact.** The
  singleton-attention certificate fires on 15/99 and 17/94 CellScientist
  slots and on 0 for AIDE, CellForge and HarmonyCell. CellScientist's
  checker coverage is also the lowest (60.6% / 51.1% resolved; the other
  policies are ≥98.9%).
- **48-candidate linked audit (Table 60).** Stratified and descriptive:
  - 47/48 change predictions under compound replacement on both folds.
  - 20/48 have positive target-loss intervals on both folds.
  - One *implemented* MultiTowerFusion source is exactly invariant.
  - The split is by cohort: **BBBC036 0/24** positive vs. **BBBC047
    20/24**.
- **Explicit route ≠ contribution.** All 200 path-constrained evaluations
  pass the source checks. Mean compound effects are still only 0.0083 /
  0.0074, and 6/10 and 7/10 models fall below the reference threshold.
- **Falsification-guided residual.** It beats control-only by +0.0037 /
  +0.0028 PCC, with positive compound effects (0.0053 / 0.0048) and joint
  intervals above zero. Yet **0/10** models clear the reference-relative
  criterion on either fold. Positive mean contribution and passing the
  per-model criterion are different results.
- **Audit feedback (sci-Plex).** Audit minus Score PCC is +0.0024
  (paired-t 95% CI [−0.0013, +0.0060]) on Fold 4 and +0.0015
  ([−0.0005, +0.0034]) on Fold 5, with 4/5 and 3/5 wins. MSE is 9.58% /
  6.10% lower. A joint bootstrap excludes zero, which the authors flag as
  an artifact of interval construction at five pairs. The anchor alone
  already reaches PCC 0.9664 (Score 0.9756, Audit 0.9779). Audit costs
  **592,347 vs. ≥286,995** tokens (38 vs. 36 calls), roughly 2×.
- **Claim generalization fails where prediction transfers.** Frozen
  LINCS designs refit on LKCP still beat control-only by +0.0104 / +0.0211.
  Dose allocation persists (+0.0237 / +0.0341), and all 50 refits pass dose
  at every boundary. Compound identity collapses (−0.000070 / +0.0033), and
  0 refits pass it on LKCP. Even within LINCS the compound pass rate moves
  from 35/50 on Fold 4 to 10/50 on Fold 5 while the mean stays positive.
- **Norman CRISPRa.** All 5 refits pass the pair-bundle criterion, and
  shuffling costs 0.5578 / 0.6280 PCC. This is a positive case: the audit
  does not only return failures.

## Critique / open questions

- **The strongest result is the least novel, and the most transferable.**
  An agent loop picked a model that ignores the intervention and scores
  like a trivial baseline. This is permutation reliance plus
  shortcut-learning (Fisher 2019; Geirhos 2020) applied to agent-selected
  models. The contribution is doing it as a pre-registered, frozen-
  checkpoint layer on an agent loop, and showing that score selection
  lands there repeatedly (both tasks, two different policies).
- **The "revise" half is weak evidence.** The BBBC residual formulation
  changes the model space (frozen control-only anchor + compiler) at the
  same time as the feedback. Its gains are +0.003 PCC, and none of its
  models passes the paper's own criterion. The one feedback-only
  comparison is sci-Plex: n = 5, a single cheap model, CIs spanning zero,
  about 2× the tokens, and a ceiling where the anchor already reaches 0.966.
  "Discover–falsify–revise" is argued for more strongly than it is
  measured, although the paper itself hedges it ("suggestive rather than
  conclusive").
- **Self-audit cuts both ways.** Three authors (Mengran Li, Bo Li,
  Chengyang Zhang) wrote CellScientist, and the flagship invariant model is
  CellScientist's. It is a credible negative result on their own system.
  The paper still ranks CellScientist first on Best@10 (Table 19), and the
  singleton-attention failure is concentrated in its candidates.
- **Evaluation surface is narrow for an agent claim.** There is one
  discovery LLM (DeepSeek V4 Flash) and ten-slot budgets. The competitor
  policies are the authors' "common-executor instantiations", not the
  original systems.
- **Threshold dependence.** Status depends on reference-context variation.
  The authors show that "holding compound benefit fixed while changing
  reference-context variation can also change qualification." That is why
  they report the continuous effect, across-model uncertainty and
  reference-relative status separately, and a reader should too.
- **Open:** would a cheaper check have caught the headline case? A
  control-only baseline in the selection panel would have tied it, so
  that is a one-line guard (see Follow-up).

## Trust signals

- **Credibility:** 4. Supporting it: universities plus Tencent AI Lab,
  released code with a reproducibility guide, checkpoint replay with hash
  verification, and every call's request/response hash logged. The audit
  rules, maps and thresholds were frozen before post-search folds, and
  known-truth fixtures (6,400 simulated datasets) calibrate the
  instrument. The paper reports a negative result on the authors' own
  system and hedges its weakest claim in the abstract. Holding it at the
  bottom of 4: unreviewed and days old, a single discovery LLM, and the
  feedback study has n = 5. The data are public, but fitted weights are an
  external input.

## Follow-up

- **Relevance:** 4, up from the digest's 3. It is the first measured case
  in this graph of an agentic model-discovery loop whose *held-out
  score* was genuine while the *claim attached to the selected model* was
  false, and where score selection landed on a trivial-baseline-equivalent
  model on both tasks. That is a new axis for
  [[concepts/hce-evaluation]], which protects the number and not the
  claim. Not a 5: the domain is cell biology, and the architectural half
  (audit feedback that improves discovery) is not established.
- **[[concepts/hce-evaluation]]:** new section, "A held-out score
  certifies the number, not the claim attached to the model."
- **[[concepts/typed-claim-partition]]:** mechanism claims have
  evidence layers that diverge. Hard evidence at the source layer does
  not certify behaviour, and "not identifiable" is a fourth outcome.
- **[[concepts/programmable-evaluator-oracle]]:** a correct, immutable
  scalar oracle selected input-ignoring models. Returning audit diagnostics
  is the constructive side of the "what the evaluator returns" question,
  with a measured price of about 2× tokens and an unresolved gain.
- **Cheap import for `/iterate`:** when a chain's "new best" carries a
  mechanism claim ("uses feature X", "the new module helps"), run two
  checks. First, put a baseline without that component in the same
  held-out panel. Second, record a one-shot input-replacement delta on the
  frozen checkpoint. Both are inference-only.
- **Candidates:** POPPER (Huang et al. 2025, agentic sequential
  falsification with error control) is the closest agent-side anchor and
  is not in the graph. CellScientist (arXiv 2605.07335) is the audited
  system. Ahlmann-Eltze et al. 2025 (Nature Methods; deep perturbation
  predictors do not beat linear baselines) is the peer-reviewed domain
  anchor for the trivial-baseline finding.
