---
kind: paper
title: "The Critic Can Hurt: On the Safety of Critics in LLM-Guided AutoML Search"
authors: ["Gaoyuan Du", "Xiaoyang Liu", "Rex Zhou", "Dengdu Jiang", "Rong Fu", "Meng Ding", "Defu Cao"]
institutions: ["Amazon", "Independent Researcher", "University of Massachusetts Boston", "University of Southern California"]
year: 2026
venue: "arXiv 2611.19037 (cs.LG), v1 2026-11-14, preprint (12-page main text + 13-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.19037"
code_url: null    # CritFault, prompts, cached frontiers and logs "to be released upon publication"
citations: null
source: "raw/papers/du2026critic.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["llm-critic", "critic-safety", "automl", "tabular", "random-search-floor", "no-critic-arm", "four-arm-audit", "fault-injection", "best-so-far", "elitist-selection", "directional-error", "validation-gate", "critfault", "llama-3.1-8b", "low-headroom", "negative-result"]
---

# The Critic Can Hurt: On the Safety of Critics in LLM-Guided AutoML Search

## TL;DR

An LLM reads the search history (configuration names and their
validation AUCs) and the menu of untried configurations, writes a short
critique, and names the next configuration to try. The loop keeps the
best-so-far on validation. The menu is a **cached 96-configuration
tabular frontier**, the budget is **T = 8 rounds**, and the control is
**uniform random search over the same menu at the same budget**. Critic
and proposer are **one LLM call**, so what is audited is the LLM's
whole selection policy, not the critique alone.

Five results matter for this graph:

1. **A Llama-3.1-8B selector lands below random search on 7 of 8 binary
   datasets**, but the sizes are very uneven. ADULT −0.245 (95% CI
   −0.27 to −0.22), JM1 −0.080, ELECTRICITY −0.072, KC1 −0.028, and
   three below 0.01 (PHONEME −0.009, BANK −0.007, CHURN −0.002).
   CREDIT-G is +0.006. Suite-level sign-flip p = 0.0078.
2. **The floor is nearly the ceiling.** The oracle (best untried
   validation pick) beats random search by **0.0013–0.0077 AUC** on
   every dataset, and random search saturates in about 1–3 rounds. So
   no critic can gain more than about 0.005 here, and the "fifty times
   larger downside" holds largely by construction of the frontier.
3. **The harm belongs to one model on one menu, not to "8B critics".** A
   different 2025 8B (Ministral-3-8B) sits at the floor on all six
   ladder datasets (−0.012 to +0.001). Llama-3.3-70B is mildly negative
   on 5 of 6 (to −0.031). Claude Haiku 4.5 is within ±0.006 everywhere:
   harmless, and no help. With a 250-configuration menu and T = 20 the
   same 8B goes from −0.245 to +0.0003 on ADULT. The same 8B on five
   non-tabular or in-domain settings is neutral or slightly positive.
4. **Under keep-best, random errors are free and directional ones spend
   the budget.** That is Prop. 1, and it holds by construction: a
   uniform pick is the floor. The informative half is the measured
   tolerance. Injected worst-pick errors cost more than 0.01 AUC only
   at 60–90% corruption, because random search needs just 1–3 good
   rounds out of 8. The net-benefit condition q(m−1) > β is necessary
   in an idealized model, and its parameters are never estimated for a
   real critic.
5. **Mitigation returns you to the floor, never above it.** A simple
   gate that drops the critique when its last pick failed against the
   incumbent averages **+0.00046** against uniform, against −0.00541
   for the raw critic, on six unnamed datasets. QLoRA fine-tuning on
   oracle picks also lands within ±0.003 of the floor, with no interval
   positive. Since the attainable upside is ≤0.005, no mitigation could
   show retained upside here.

## Claims

- An LLM critic is not a free addition to a propose–evaluate–correct
  loop. Its marginal effect must be audited against an otherwise
  identical no-critic loop at the intended budget.
- A weak proposal is checked by evaluation and discarded by keep-best,
  but a critique is neither scored nor filtered and conditions every
  later proposal. So the critic's cost is misdirected budget, not a
  worse incumbent. (The experiments cannot test this asymmetry, because
  critique and proposal are one call.)
- Harm attenuates with critic capability and with frontier size and
  horizon. There is no capability threshold.
- Error *structure*, not error *rate*, sets the cost: non-directional
  errors act like random exploration; directional ones waste effective
  rounds.
- The net-benefit condition q(m−1) > β (competence q, lookahead m,
  adversarial rate β) is necessary and the leading-order boundary when
  improvements are rare. It is not sufficient (a Bernoulli
  counterexample is given) and does not characterize real critics.
- Breadth (parallel k-wide search) is a partial structural defense.
- Harm tracks mismatch between the critic's priors and the task. This
  is a hypothesis, stated as untested (no name-anonymization arm).
- Observable gating reduces average loss but guarantees nothing per
  dataset. A one-shot pilot-round red flag fails on low-headroom menus.

## Methods

- **Loop.** Each round one LLM call reads the evaluated history
  (configuration names, validation AUCs) and the untried menu, states
  an assessment and names the next configuration. A response naming no
  untried configuration falls back to a uniform draw. Keep-best on
  validation; test AUC is read only for the final incumbent.
- **Frontier.** 96 configurations per dataset (8 model families × 4
  feature-transform sets × 3 hyperparameter presets), every one
  pre-trained and cached on one frozen split. All arms share the cache,
  so training noise and execution failures are excluded.
- **Four arms**, budget-matched:
  1. No critic: uniform random over untried (the floor).
  2. "Random feedback": in the reported runs, a second independently
     seeded random search, *not* a semantic placebo. ∆RF is a second
     draw of ∆NC and isolates nothing.
  3. Real critic.
  4. Oracle: picks the best untried validation score directly.
  All conclusions rest on ∆NC = real − no-critic.
- **Data.** 12 OpenML classification sets (700–8,400 train rows);
  multiclass targets binarized. Each analysis uses a different subset:
  audit 8 binary, fault injection 9, capability ladder 6 (chosen partly
  on the 8B's result), CritFault 12.
- **Critics.** Llama-3.1-8B as the primary "naive" critic;
  capability ladder of Ministral-3B, Llama-3.1-8B, Ministral-3-8B,
  Ministral-14B, Qwen3-32B, Llama-3.3-70B and Claude Haiku 4.5, via
  Amazon Bedrock, temperature 0, ≤1,536 tokens. 8 seeds per cell. With
  deterministic decoding, what the seeds vary is not stated.
- **Fault injection.** A clean reference policy is corrupted with
  probability p per round: RANDOM (uniform untried), WORST (global
  argmin untried) or MISLEAD (worst one-component neighbour of the
  incumbent). Both adversarial modes use privileged cached scores.
  30 seeds.
- **Theory.** Prop. 1 (uniform corruption preserves the floor, by
  exchangeability), Prop. 2 (resilience bound via effective-round
  count, p* ≈ 1 − Tsat/T), Lemma 5.1 (effective sample size m·Kc + Ku),
  Prop. 3 (q(m−1) > β necessary). Idealized: i.i.d. scores, independent
  round types.
- **Prevention.** One gated policy: if gate = 0 the next proposal comes
  from a critique-free fallback.
  - VGC: the gate opens only if the last critic-conditioned pick is
    within τ of the incumbent's validation score. The adaptive version
    widens τ when bootstrap rank stability of tried configurations is
    low, and may abstain to uniform.
  - A gradient-boosted detector on three decision-time features
    (proposal validation score, change against the incumbent, round
    fraction).
  - A "legacy" protocol whose reject branch picked the best *untried*
    validation score (privileged). Its results (Table 8, Fig. 4, the
    VGC column of Table 11) are flagged exploratory by the authors.
- **CritFault.** About 3,840 fault-injection transitions (1,920
  WORST-neighbour picks vs 1,920 ideal-critic picks), leave-one-dataset-
  out. Plus a 384-transition real-harm relabelling of Llama-3.1-8B
  trajectories.
- **Extra checks.** CAAFE re-run with Llama-3.1-8B; chain-of-thought and
  5× self-consistency; QLoRA fine-tuning on oracle picks
  (leave-one-dataset-out); four search backbones and two regression
  sets under a *modeled* adversarial critic; five cross-domain settings
  with the real 8B; AutoGluon-Tabular comparison.

## Results

- **Upside (Table 15, deterministic ideal critic, 30 seeds).** +0.0013
  (ADULT) to +0.0053 (ELECTRICITY), every CI above zero, suite Wilcoxon
  p = 0.001. Near-zero headroom (≤0.0003) on MFEAT, SEGMENT, SPLICE.
- **Floor quality (Table 6).** The in-space oracle matches or beats
  AutoGluon-Tabular (120 s, medium_quality) on 5 of 8 datasets.
- **Real 8B critic (Fig. 1; Table 11 raw column).** ADULT −0.245,
  JM1 −0.080, ELECTRICITY −0.072, KC1 −0.028, PHONEME −0.009, BANK
  −0.007, CHURN −0.002, CREDIT-G +0.006. Sign-flip p = 0.0078. The
  harm-rate metric (negative *and* CI excluding zero) is defined in
  App. A but never reported, so how many of the seven are individually
  distinguishable from zero is not given. "Persists under stochastic
  decoding" is asserted without numbers.
- **Parsing is not the cause (App. D).** ADULT had 0 fallbacks in 64
  rounds. CREDIT-G had the most (18/64) and was the one dataset not
  harmed, because a fallback is a uniform draw.
- **Capability ladder (Table 1).**

  | | 3B'25 | 8B'24 | 8B'25 | 14B | 32B | 70B | Haiku 4.5 |
  |---|---|---|---|---|---|---|---|
  | adult | −.210 | −.245 | +.001 | +.001 | +.001 | −.011 | +.002 |
  | electricity | −.064 | −.072 | −.012 | +.012 | +.008 | −.031 | +.006 |
  | jm1 | −.008 | −.080 | −.003 | −.002 | −.000 | −.011 | +.002 |
  | kc1 | +.000 | −.028 | −.000 | −.001 | −.001 | −.023 | −.000 |
  | bank-mkt | – | −.007 | +.000 | +.000 | +.001 | −.007 | +.001 |
  | credit-g | – | +.006 | −.005 | −.012 | −.007 | +.006 | −.001 |

  14B beats 8B'24 on 5 of 6 (p = 0.047). Not monotone in size or date.
- **Search setting.** 250 configurations and T = 20 together: ADULT
  −0.245 → +0.0003, ELECTRICITY −0.072 → +0.0003, JM1 −0.080 → −0.004.
  Size and horizon are confounded. Under a modeled adversarial critic on
  a space with real headroom (1.7–5.6%), enlarging 96 → 250 *increased*
  harm on all five datasets (Table 7).
- **Fault injection (Table 14).** RANDOM at p = 1 costs ≤0.005 on every
  dataset. WORST threshold p* (first p with >0.01 drop) is 0.6 on
  CREDIT-G, ADULT, CHURN; 0.9 on five others; 1.0 on LETTER. Full-
  corruption cost 0.019–0.093. Prop. 2 predicts p* ∈ [0.63, 0.88] from
  Tsat ∈ [1, 3]. 90.5% of adversarial picks failed to raise the
  incumbent (Assumption 1).
- **Real-critic signature.** Family-selection KL to the oracle's family
  distribution: 8.4 for Llama-3.1-8B vs 3.2 for Haiku 4.5. In the one
  shown ADULT trajectory (reasoning-mode prompt, App. G), the critic
  praised LightGBM every round while picking worse configurations
  (0.930 → … → 0.651). Keep-best held the 0.930 incumbent, so this
  trajectory illustrates misdirection but not the −0.245 outcome.
- **Criterion check (App. B).** In simulation at T = 8 the sign of
  q(m−1) − β matched the sign of net benefit on 43 of 48 grid cells. The
  sign-flip boundary for q = 0.3, m = 3 is 0.42 / 0.53 / 0.56 / 0.60 at
  T = 8 / 32 / 128 / 512, against 0.60 predicted.
- **Backbones under a modeled adversarial critic (Table 9, 12 datasets,
  30 seeds).** Mean ∆: sequential −0.0183 (9/12 below −0.002),
  multi-agent −0.0095 (9/12), UCB1 tree −0.0080 (9/12), parallel k-wide
  −0.0004 (1/12). k and whether evaluations are matched are not stated.
- **Cross-domain, real 8B (Table 2).** Only tabular AutoML (ADULT) is
  harmed. Credit scoring (96) +0.000, 20NG text −0.003 and +0.003 (CIs
  span zero), M4 forecasting +0.001 and RAG config +0.007 (both CIs
  above zero).
- **CAAFE with Llama-3.1-8B.** Negative on 3 datasets (ADULT −0.030,
  BANK −0.054, CREDIT-G −0.090), mean −0.021, no gain above +0.004.
- **Prompting (Table 10).** ADULT: naive −0.278, CoT −0.041,
  self-consistency (40 calls) −0.071. CoT and self-consistency stay
  negative on 3 of 4. The zero-call floor beats all of them.
- **Prevention, deployable (§6.6).**
  - Paired VGC, 8 datasets × 8 seeds: fixed − uniform −0.00032
    [−0.00140, +0.00072]; adaptive − uniform −0.00038 [−0.00166,
    +0.00078]; adaptive − fixed −0.00006. Non-inferior at a 0.002
    margin on average only. No raw-critic arm is reported in this audit.
  - Six datasets: raw −0.00541, simple non-improvement gate +0.00046,
    learned detector −0.00015, all vs uniform.
- **Fine-tuning (Table 11).** QLoRA on oracle picks from seven datasets:
  every held-out dataset within ±0.003 of the floor, none with a
  positive interval.
- **Detector (Tables 4, 13).** LODO AUC 0.95 on 12 datasets (0.982 on
  nine, 0.851 on near-zero-headroom). Best single feature 0.916,
  logistic 0.921. Leave-one-model-out 0.988. Deployable three-feature
  detector on real-harm labels 0.982.

## Critique / open questions

- **Against the digest entry: at, with two overreaches.** The numbers
  quoted are in the PDF and correct, and "Tabular AutoML only" is right.
  - **"A naive 8B critic"** should read "Llama-3.1-8B". A different
    2025 8B is at the floor everywhere (TL;DR item 3).
  - **"7 of 8"** counts point-estimate signs. Three of the seven are
    under 0.01 AUC, and the per-dataset harm rate is defined but never
    reported.
- **The "critic" is the selector.** One LLM call critiques and picks.
  The paper's motivating asymmetry (a proposal is checked by evaluation,
  a critique is not) is therefore never tested, and the "random
  feedback" arm is a second random search, not a shuffled-critique
  placebo. What is shown is narrower: on these menus, LLM-chosen picks
  did worse than uniform picks. The authors say so in §3.2 and §7. The
  title and abstract don't.
- **The floor is near-optimal by construction.** Headroom is <0.01 AUC
  on every dataset and Tsat ≈ 1–3 of 8 rounds. Three results follow
  from that, not from anything about critics:
  - the bounded upside (≤0.005);
  - the "fifty times" asymmetry;
  - the wide 60–90% error tolerance.
  In an ML-research loop with large headroom, Prop. 2 predicts a much
  narrower envelope, and Table 7's modeled critic shows harm growing
  with space size. The real-critic evidence for large-headroom settings
  is just the 250/T = 20 run, where harm vanished, with size and horizon
  confounded.
- **How −0.245 happens is not shown.** Random search on ADULT reaches
  0.93 in a few draws, so a final test AUC of 0.68 means eight picks
  with no good configuration among them. The one trajectory shown uses
  a different prompt and holds the 0.930 incumbent. Table 10 gives the
  same cell as −0.278 in a "single controlled run".
- **The mitigations were tested where the raw harm was small.** In the
  deployable rerun the raw policy averages −0.0054, about a tenth of the
  8B audit's mean (−0.055 over the eight datasets, our arithmetic). The
  six datasets and the critic for that rerun are not named. The VGC
  audit reports no raw arm, so its effect against raw is not given.
- **The detector's AUC is largely by construction.** Adversarial labels
  are low-score picks and clean labels are oracle picks, and feature f0
  is the pick's validation score. The real-harm label ("fails to raise
  the incumbent and leaves ≥0.01 on the table") is close to feature f1
  (change against the incumbent). One feature already reaches 0.916. The
  authors say AUC does not establish a safe filter, and the online
  result (−0.00015 vs uniform) bears that out.
- **The theory is mostly bookkeeping.** Prop. 1 is exchangeability. Prop.
  2 and Lemma 5.1 delete rounds that cannot raise a max. Prop. 3's
  condition is necessary only. (q, m, β) are never estimated for a real
  critic, and CREDIT-G has a harmed-looking β proxy with a positive
  delta. The useful quantitative content is Tsat and the measured p*.
- **Smaller issues.**
  - The breadth result (Table 9) uses a modeled critic with privileged
    argmin access. k and evaluation matching are not stated.
  - The ladder's six datasets were picked partly on the 8B's result
    (disclosed).
  - Our arithmetic from Table 11's rounded values gives a sign-flip
    p of 3/256 ≈ 0.012, not 0.0078. Either way it is below 0.05.
  - The CAAFE run swaps in the 8B for CAAFE's original model, so it
    shows "weak model in a published loop", not a CAAFE defect.
  - "Typical ~0.03" is the median of the seven negatives (KC1 −0.028).
- **Consistent with neighbours in the graph.**
  - [[literature/papers/chen2026cads]]: random is the floor, a random
    fallback flatters the LLM, and small open LLMs net about zero over
    the proposer-removed loop. Here the 8B goes *below* that floor, and
    CREDIT-G shows the fallback pulling a weak selector back to it.
  - [[literature/papers/alexander2026iterated]]: for strong writers a
    same-budget no-feedback arm took nearly all the feedback gain. Here
    Haiku 4.5 is likewise indistinguishable from no critic.
  - [[literature/papers/zeng2026resolution]]: selection at chance. That
    is about selecting among scored candidates; this is about choosing
    which to score.

## Trust signals

- **Credibility:** 3.
  - For:
    - An Amazon-led group (four of seven authors) with UMass Boston and
      USC co-authors.
    - Every arm shares one cached frontier, so training noise is gone.
    - Unusually careful self-limiting text. The abstract and §7 state
      the necessary-not-sufficient status, the merged critic-proposer,
      the placebo that is not one, the ladder's selection and the
      legacy-protocol leak.
    - Many robustness arms: seven critics, CoT and self-consistency,
      fine-tuning, CAAFE, cross-domain, backbones.
  - Against:
    - Unreviewed, ten days old, no citations.
    - Code, prompts, cache and CritFault are all "to be released".
    - The deployable mitigation rerun's datasets and critic are
      unnamed, and its raw harm is a tenth of the headline's.
    - The per-dataset harm rate is never reported.
    - The headline cell appears as −0.245 and −0.278.
    - The case study does not show the headline mechanism.
  - Not 4: no artifacts and unreviewed. Not 2: a reputable industry
    group, a clean shared-cache design and disclosures that most
    preprints in the graph lack.

## Follow-up

- **Relevance:** 4.
  - It is the first measurement in the graph of an LLM-guided selection
    policy going **below** a same-budget uniform-random floor inside a
    keep-best loop, with a measured fault-tolerance envelope.
  - Material new evidence for [[concepts/evolutionary-expansion]] (the
    random floor), [[concepts/programmable-evaluator-oracle]] (what a
    correct keep-best oracle does and does not protect) and
    [[concepts/hce-evaluation]] (the no-critic arm at the intended
    budget).
  - Not 5: tabular menus with <0.01 headroom, eight rounds, weak open
    models. The one frontier-class critic (Haiku 4.5) neither helped
    nor hurt.
- **For `/iterate` (our inference, untested).** Where a step's search
  space is enumerable (a sweep grid, a fixed menu of recipes), a
  uniform pick over it at the same budget is the cheapest floor arm.
  The paper and chen2026cads both show it is not a formality for small
  models. For a frontier proposer on an open-ended space the evidence
  here is silent.
- **Candidates** (cited, not yet in the graph):
  - Hollmann, Müller and Hutter 2023, CAAFE (NeurIPS). The published
    LLM feature-engineering loop re-run here.
  - Chi et al. 2024, SELA (arXiv 2410.17238). Tree-search LLM AutoML
    agent with a search-policy ablation.
  - Itkin 2026, "Delayed verification destabilizes multi-agent LLM
    belief" (arXiv 2606.27409). Placement of the corrector.
  - Kiyani et al. 2026, "When to trust the cheap check: weak and strong
    verification for reasoning". Verifier-reliability gating.
  - Khalaf et al. 2025, inference-time reward hacking (NeurIPS).
  - Huang et al. 2024, "LLMs cannot self-correct reasoning yet" (ICLR);
    Kamoi et al. 2024 (TACL). Background for the self-correction nulls.
