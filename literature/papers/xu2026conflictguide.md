---
kind: paper
title: "ConflictGuide: AutoResearch Improves When Competing Behaviors Are Made Visible"
authors: ["Binqian Xu", "Qiran Zou", "Xiangbo Shu", "Dianbo Liu"]
institutions: ["National University of Singapore", "Nanjing University of Science and Technology"]
year: 2026
venue: "arXiv 2609.39933v1 (cs.AI), 2026-09-30"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.39933"
code_url: null
citations: null
source: "raw/papers/xu2026conflictguide.pdf"
added: "2026-10-06"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["autoresearch", "competing-behaviors", "probe-feedback", "two-stage-search", "branch-point-comparison", "redirect-vs-continue", "keep-rule", "noise-band", "retention-threshold", "feedback-timing", "conflict-taxonomy", "probe-card", "agent-skill", "matched-budget", "proxy-to-formal-gap", "selective-reporting"]
---

# ConflictGuide: AutoResearch Improves When Competing Behaviors Are Made Visible

## TL;DR

The setup is a Karpathy-style AutoResearch loop: Claude Opus 4.6 edits one
training file, and a protected controller keeps an edit if the scalar task
metric improves. It is run on five model families: a spectral FNO for
Navier–Stokes, SNGP on CIFAR-100, GCNII node classification, an echo-state
network, and TCM-Lite image compression.

- **Stage I.** 100 scalar-only proposals (20 for ESN) produce a branch
  point.
- **Stage II.** Two matched 100-proposal continuations (50 for ESN) start
  from that branch point:
  - plain AutoResearch, which keeps any edit with G > 0;
  - ConflictGuide. The prompt also names a model-specific "conflict" (two
    desirable behaviours coupled through one mechanism) and shows
    behaviour-level probe values. The keep rule changes as well: clear
    gains (G > τS) are kept directly, and gains in the marginal band
    [0, τS] are kept only if the probes show alleviation.

The probes come from a "ConflictGuide-Skill". It screens a 110-mechanism
Root–Axis–Mechanism taxonomy against the target model and qualifies probes
against no-edit noise before Stage II.

ConflictGuide's terminal source beats AutoResearch's in most of the 15
model × round cells. The size of the win depends heavily on the model:

- large on SNGP (mean 18%) and on the ESN transfer task (mean 19%);
- 2–4% mean on FNO, GCNII and TCM-Lite, often within one training-seed s.d.

For this graph, it is the first **matched branch-point comparison** of
"continue the same loop" against "change what the loop sees and keeps".
Two limits matter:

- **It is not plateau-triggered.** The switch happens at a fixed iteration
  count chosen in advance.
- **The intervention bundles three things:** new feedback, a narrowed edit
  scope, and a different keep rule.

The ablation suggests that on SNGP most of the gain comes from the keep
rule, not from the visible probes.

## Claims

- **Scalar feedback hides trade-offs.** "scalar feedback supports broad
  exploration early in search, but it does not reveal how edits affect
  competing behaviors."
- **Feedback timing matters.** Feedback from the outset "produced
  conflict-specific edits but concentrated the search on spectral
  compensation". Delayed feedback "sustains progress beyond scalar-only
  plateaus."
- **Headline number.** ConflictGuide "reduces task and conflict-related
  errors by up to 28% and 14%, respectively, relative to scalar-only
  AutoResearch, with gains extending to other code agents."
- **Proposal-level shift.** "The share of proposals improving both
  behaviors rose from 7.2% to 13.1%, while the share degrading both
  behaviors fell from 29.0% to 20.7%."
- **Measured probes beat generic metrics and prompts.** "guidance benefits
  from measuring the instantiated competing behaviors, rather than
  providing additional metrics or describing the trade-off."
- **Formal information argument (App. E).** If probes carry information
  about true alleviation beyond the scalar gain, they strictly reduce the
  optimal prediction risk. The authors add that this "does not imply ...
  that a coding agent necessarily uses the additional information well."
- **Guidance and retention are complementary (ablation).** "guidance
  redirects proposals via conflict-specific feedback, while retention
  filters noisy task gains".

## Methods

- **Loop ownership.** The design differs from upstream autoresearch (App.
  G.6, Table 23):
  - The agent only proposes edits to `train.py`. A protected controller
    owns validation, training, retention, rollback and an append-only
    hash-bound ledger.
  - Each proposal runs in a fresh session with a deterministic
    structured-history view.
  - Probes are computed outside the editable file. Where costs had to be
    matched, they are also computed hidden in the AutoResearch arm.
- **Matched branch construction.** For each of 3 search rounds, one Stage-I
  trajectory is shared by both Stage-II arms. Both arms share the same
  source hash, history prefix and policy, evaluator, seeds and budget. The
  budget is counted in proposals, including crashed or invalid ones, not
  in wall clock.
- **Keep rules (App. G.3, Table 21).**
  - AutoResearch: keep iff G > 0.
  - ConflictGuide: keep iff G > τS, or G ∈ Bc ⊆ [0, τS] and the probe
    alleviation passes its guards. G < 0 is never kept.
  - Per-model thresholds:
    - FNO: τS = 5e-4, with the probe route requiring a ≥1% relative drop in
      non-dominant-mode error.
    - SNGP: τS = 1e-3.
    - GCNII and ESN: ties are admitted via probes.
    - TCM-Lite: the probe route applies to exact ties only.
  - Probe thresholds come from the 95th percentile of paired no-edit
    variation and are frozen before Stage II.
- **What the ConflictGuide prompt adds.** It names the conflict and "the
  conflict-relevant components available for modification", shows probe
  values with their desirable directions, and gives keep/reject reasons.
  This is a scope restriction as well as a new signal.
- **ConflictGuide-Skill.** It has three modes (ANALYZE, IMPLEMENT,
  QUALIFY) and a five-stage workflow: context card, taxonomy screen,
  conflict instantiation, probe design, and qualification. It abstains
  when the evidence is thin. Probes are read-only measurements and may not
  touch the objective, gradients, data or retention rule. Its output is a
  machine-checkable "Conflict Probe Card". Conflict selection is barred
  from using evolution trajectories or test results.
- **Proxy search, then formal evaluation.** Search runs on reduced-cost
  proxies, for example 200 Navier–Stokes trajectories and a width-32 FNO,
  or a single Chameleon split for GCNII. The frozen terminal source is then
  retrained from scratch at full scale on held-out data. Reported s.d. is
  over training seeds or splits of **one winner per round**, not over
  search rounds.
- **Baselines.**
  - Upstream-style scalar-only AutoResearch, run in the same controlled
    harness.
  - Two feedback baselines on SNGP and GCNII, round 2 only (App. H): a
    multi-metric arm (generic calibration, OOD or robustness metrics with
    task-based retention), and a one-sentence "consider trade-offs" prompt.
  - Agent transfer on FNO: Kimi K2.7 Code (seed 2), and AIDE with
    o4-mini/GPT-4.1-mini and with GLM-5.3 (seed 0). One run each.

## Results

- **Formal results, % reduction in task error vs AutoResearch's winner, by
  round** (our arithmetic from Tables 2–6):

  | Model (task metric) | R0 | R1 | R2 | Mean |
  |---|---|---|---|---|
  | SpecB-FNO (test NRMSE) | 9.9 | 0.7 | 0.3 | 3.6 |
  | SNGP (CIFAR-100 NLL) | 19.6 | 6.7 | **28.2** | 18.1 |
  | GCNII (Wisconsin clean NLL) | 3.0 | 1.4 | 2.3 | 2.2 |
  | ESN (Mackey–Glass h=84 NRMSE, transfer) | **28.0** | 21.1 | 8.9 | 19.3 |
  | TCM-Lite (Kodak bpp) | 4.1 | 1.6 | 2.5 | 2.7 |

  - The conflict-metric **14%** is FNO round 0 non-dominant-mode error.
    Rounds 1 and 2 give 0.8% and 0.4%, and TCM zdetail gives 2.0, 1.0 and
    1.5%.
  - FNO rounds 1 and 2, and every GCNII round, differ by less than one
    training-seed s.d. For example, GCNII NLL s.d. is about 0.08–0.24
    against gaps of 0.01–0.02.
- **AutoResearch's SNGP winners are worse than the unmodified reference in
  2 of 3 rounds.** Reference NLL is 1.0434; AutoResearch scores 1.0578 in
  R0 and 1.1777 in R2. Against the reference, ConflictGuide's R2 is a 19%
  reduction, not 28%. The proxy-selected scalar-only edits did not
  transfer to full-scale training.
- **Exploratory timing study (FNO, App. A).**
  - Probes from the outset: test NRMSE 0.0465, **worse than the reference
    (0.0458)**. Scalar-only reached 0.0379.
  - Delayed probes: 0.0344 vs 0.0382, a 9.9% reduction.
  - Table 11 (delayed) has the same numbers as Table 2 round 0, so the run
    that motivated the design is also one of the three evaluation rounds.
- **Ablation (Table 7).** It uses FNO R0 and SNGP R2 as baselines, the most
  favourable round for each model.

  | Config | FNO NRMSE | SNGP NLL |
  |---|---|---|
  | Baseline (AR) | 0.0382 | 1.1777 |
  | Guidance only | 0.0348 | 1.0381 |
  | + Retention (full) | 0.0344 | 0.8455 |

  On FNO, guidance does nearly all the work. On SNGP, guidance alone only
  gets back to about the reference (1.0434), and the retention route
  supplies the remaining 0.19 NLL.
- **Feedback baselines (Table 8, round 2).**

  | Config | GCNII clean NLL | SNGP NLL |
  |---|---|---|
  | AR (Tables 3–4) | 0.5669 | 1.1777 |
  | Multi-metric feedback | 0.5662 | 0.8918 |
  | Trade-off prompt only | 0.5694 | 1.1959 |
  | ConflictGuide | 0.5540 | 0.8455 |

  - The prompt-only reminder does nothing or slightly worse.
  - On SNGP, generic multi-metric feedback with plain task-based retention
    recovers about 86% of ConflictGuide's gain over AR. That beats
    ConflictGuide's own guidance-only ablation (1.0381). The authors flag
    that the multi-metric agent saw CIFAR-10 examples during search, the
    same OOD domain used in evaluation.
- **Proposal analysis.** Relief rate rose 10.8% → 21.5% on SNGP and
  20.0% → 28.8% on GCNII. Edits shift toward the named mechanism: on FNO,
  spectral edits rose 12.3% → 43.4%. The pooled 7.2% → 13.1% joint-improve
  figure is given without per-model counts.
- **Scalar ambiguity (App. J.1).** Among positive-gain scalar-only
  proposals, behaviour-outcome entropy is 0.89 (max 1) in every task-gain
  quartile. Similar scalar gains therefore hide different behaviour
  changes. The authors limit this to a "limited claim" (no causal
  reading).
- **Agent transfer (Fig. 4, FNO test NRMSE, scalar → probe).**
  - Kimi: 0.0433 → 0.0388.
  - AIDE (GLM): 0.0429 → 0.0419.
  - AIDE (o4-mini): 0.03444 → 0.03431, a 0.4% difference. In this run
    "probe-informed parent selection did not activate".
- **Probe cost.** It is matched to zero for FNO, ESN and TCM, where probes
  are computed hidden in both arms. SNGP adds 2.6% and GCNII adds 12.8%.
  The effort of designing and qualifying probes for each model is excluded
  from all cost comparisons (App. K).

## Critique / open questions

- **The running finding holds (now 32/32).** The PDF supports less than the
  digest entry, and the gap comes from the most favourable cut:
  - "Up to 28%" is the maximum over 15 model × round cells. One source is
    SNGP R2, where the baseline winner had regressed **below the unmodified
    reference**. The other is ESN R0, measured on a cross-task transfer
    benchmark, not the search task.
  - "14%" is the single FNO round that doubled as the design-motivating
    exploratory run.
  - Median per-round task gain on three of the five families is 2–3%,
    mostly inside training-seed noise.
- **"Switching as gains diminish" is a fixed schedule, not a plateau
  detector.** Stage I runs exactly 100 proposals (20 for ESN), and "the
  stage boundary T1 is fixed before evolution". The paper never measures
  whether the branch point is actually a plateau. Adaptive switching is
  listed as future work. So it does not test the trigger, only the
  intervention.
- **The redirect changes more than the feedback signal.** The digest's "the
  redirect is 'change the feedback signal' rather than 'change the idea'"
  understates it. The ConflictGuide arm also:
  1. narrows the prompt to "conflict-relevant components available for
     modification", an enforced scope redirect;
  2. replaces AR's G > 0 keep with a thresholded rule plus a probe
     tie-break.

  Point 2 is effectively a noise band: SNGP's direct route needs G > 1e-3.
  On SNGP the ablation puts most of the effect there. The multi-metric
  baseline (generic metrics, plain retention) gets close to ConflictGuide
  on SNGP. So "the specific instantiated conflict matters" is supported on
  GCNII by a small margin and is weak on SNGP.
- **The baseline's failure to transfer is part of the headline.**
  AutoResearch's proxy winners regress at full scale on SNGP in 2 of 3
  rounds. A strict G > 0 keep on a single-seed proxy admits noise, and that
  noise compounds down the lineage. ConflictGuide's higher direct threshold
  plausibly filters some of it, and the authors' own gloss is that
  retention "filters noisy task gains". This is our inference: a fair
  comparator would be AR with the same τS and no probes. That arm is
  missing, and it is the one this graph most wants.
- **n is small and the dispersion is the wrong kind.** There are 3 search
  rounds per model. The reported s.d. is over retraining seeds of one
  winner, so it says nothing about search-to-search variance. No
  significance tests are reported. Agent transfer is one seed per setting,
  and one of the three AIDE/Kimi cases is a 0.4% difference with the probe
  mechanism inactive. "Across all settings ... consistently improves"
  overstates this.
- **Some evaluations are reported only partly.**
  - GCNII is formally evaluated on seven datasets, and "the cross-dataset
    claim is supported by the complete seven-dataset protocol". Only
    Wisconsin appears in the PDF.
  - ESN's in-domain NARMA-30 evaluation and its h = 1 Mackey–Glass horizon
    are specified but not reported. Only the h = 84 transfer appears.
  - On Wisconsin, the formal quantity that matches the probe
    (contamination-induced ΔNLL) improves "only in round 2" (App. F.5).
- **Feedback is not free, and the paper shows it.** Probe feedback from the
  outset made FNO *worse than the unmodified reference*, while scalar-only
  improved it 17%. That is a clean negative result on one model: a
  narrowing signal introduced too early collapses exploration. It should be
  carried as such, not hidden under the delayed-feedback win.
- **"Packaged as a reusable skill" is unverifiable for now.** The skill and
  code are "included in the supplementary material". The taxonomy index,
  schemas and validators are to be released "upon acceptance". No URL is
  given. Probe design for each model is human-heavy, and its cost is
  excluded.
- **Credit where due.** The harness discipline is better than most
  AutoResearch papers:
  - an external controller and append-only hash-bound ledger;
  - probes frozen before Stage II and calibrated against no-edit noise;
  - hidden probes in the control arm to match cost;
  - test data never online, and winners retrained from scratch.

  App. E is honest that information gain does not imply better search.

## Trust signals

- **Credibility:** 3. In its favour:
  - a reputable group (Dianbo Liu's NUS lab, the same group as
    zou2026fmlbench);
  - an unusually careful controlled harness;
  - five heterogeneous model families;
  - frozen thresholds and held-out formal retraining.

  Against it:
  - an unreviewed v1 preprint with no public code (release "upon
    acceptance");
  - 3 search rounds per model, with dispersion over training seeds only;
  - headline numbers taken from the most favourable cells;
  - partial reporting of the GCNII and ESN formal evaluations;
  - the motivating study reused as an evaluation round;
  - an ablation run only on the most favourable rounds.

## Follow-up

- **Relevance:** 4. It is the first matched branch-point experiment in this
  graph where a stalled-ish scalar loop either continues or changes what it
  sees and keeps, under an equal proposal budget, across five model
  families. It is material new evidence for:
  - the redirect-before-halt idea in [[concepts/budget-as-ceiling]];
  - the "richer diagnostics in the feedback, verdict on the frozen score"
    rule in [[concepts/programmable-evaluator-oracle]].

  It is not a 5 for three reasons: the trigger is untested, the
  intervention is a bundle, and per-model probe engineering makes it
  costly to import.
- **Bearing on the pending `/iterate` changes.**
  - *Redirect vs continue:* supportive in direction. A matched continuation
    from the same branch point is the right design, and a controller-
    enforced scope change plus new evidence beat plain continuation in most
    cells. Prompt-only redirection ("consider trade-offs") did not help,
    which agrees with yan2026traceml. It still does not test redirect
    against **halt**, or a stall-detected trigger.
  - *Noise band:* indirectly supportive. A G > 0 keep on a noisy proxy
    produced SNGP winners worse than the reference, and the thresholded
    keep did better. But the band is confounded with probes. The missing
    arm, AR + τS with no probes, would isolate it.
  - *Practical import:* a cheap version for `/iterate` is to log one or two
    secondary diagnostics per run and show them to the proposer only after
    the stall counter fires. Keep the keep/reject verdict on the primary
    metric, with a band. Do not show diagnostics from the start: the FNO
    from-outset result says early narrowing can hurt.
- **The Conflict Probe Card format** (behaviour pair, coupling, what the
  scalar aggregates away, frozen probes, null-calibrated thresholds) is a
  reusable spec shape for "instruments the agent may read but not write".
  This is the same boundary as programmable-evaluator-oracle's "agent may
  write instruments, not verdicts".
- **Candidates:**
  - Kim et al. 2026, Self-EvolveRec (arXiv 2602.12612): directional
    diagnostic feedback for recommenders.
  - Yin et al. 2026, EvoPINN: training-dynamics-conditioned proposals.
  - Liu et al. 2026a, GoalEvolve: localizing multi-objective gaps.
  - Liu et al. 2026c, NOVA: an "architecture gradient" from history plus
    verification diagnostics.
  - Abueidda et al. 2026, "Physics-audited agentic discovery in scientific
    ML" (arXiv 2607.07379).
