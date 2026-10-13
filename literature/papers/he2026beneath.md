---
kind: paper
title: "Beneath the Diff: Diagnosing and Mitigating Algorithmic Mode Collapse in Code-Level Autonomous Research Loops"
authors: ["Bowei He", "Weixu Zhang", "Yili Jin", "Xue Liu"]
institutions: ["MBZUAI", "McGill University", "MirrorSpace Technology", "Simon Fraser University"]
year: 2026
venue: "arXiv 2609.00077 (cs.CL), v1 2026-08-31, preprint; ACL-style format, and the repo calls it 'our submission'"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.00077"
code_url: "https://github.com/BokwaiHo/arl-mode-collapse"
citations: null
source: "raw/papers/he2026beneath.pdf"
added: "2026-10-13"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["autoresearch", "karpathy", "mode-collapse", "algorithmic-mode-collapse", "proposal-diversity", "mechanism-entropy", "semantic-dedup", "category-reweighting", "faithfulness-gap", "audit-gate", "three-tier-evaluation", "blind-holdout", "goodhart", "keep-if-better", "operator-attractor", "controller-not-prompt", "stall", "redirect"]
---

# Beneath the Diff: Diagnosing and Mitigating Algorithmic Mode Collapse in Code-Level Autonomous Research Loops

## TL;DR

In Karpathy-style autoresearch loops (an LLM edits a training pipeline, the
pipeline runs for 5 minutes, the edit is kept if a validation metric
improves), the *kind* of edit the proposer makes narrows over a few hundred
iterations while the diffs stay lexically diverse. The authors call this
**algorithmic mode collapse**. Under vanilla autoresearch with Claude Opus 4.7
on four small NLP pipelines, mechanism entropy over accepted edits falls from
1.76 to 1.18 nats by iteration 300 on T2. By iteration 200 on T1, 74% of
accepted edits are LR-schedule or AdamW tweaks. Meanwhile only 38–49% of the
in-loop gain shows up on separate audit benchmarks.

The mitigation, DAPS, is enforced by the controller, not the prompt. It has
three parts: reweight sampled candidates toward under-used edit categories
(CCR), reject candidates semantically near the last 200 accepted edits (PEM),
and every 10 iterations revert to the last checkpoint whose in-loop/audit gap
is under a threshold (HVG). DAPS raises the audited faithfulness ratio from
0.44 to 0.79 (0.41 to 0.74 on a blind tier no component reads), with in-loop
gain within 2% of vanilla. Raising temperature and adding a Reflexion summary
do nothing.

## Claims

- **Surface diversity is a poor warning sign.** Mean pairwise edit distance of
  accepted diffs stays flat (0.66–0.68) while semantic clusters fall 50–70%
  and mechanism entropy falls about 0.6 nats. A monitor that watched only diff
  diversity "would have raised no flag".
- **Collapse goes with Goodharting.** The in-loop/audit gap Δ^faith grows
  monotonically from ≈0.05 at t = 60 to ≈0.17 at t = 300, tracking the
  entropy fall "with almost no iteration lag". Across methods, the ones that
  hold entropy also close the gap (Table 4).
- **The mechanism is in the proposer prior times the acceptance rule**
  (Appendix A, conceptual):
  - (F1) the proposer's prior over edit categories is peaked toward
    optimizer/LR edits;
  - (F2) an "improves by more than ε" rule accepts low-variance
    micro-tunings more reliably than high-variance structural edits;
  - (F3) accepted edits in the history pull the next proposals toward
    themselves.
  Six predictions follow, including that temperature will not help (P3)
  and that only a held-out audit can undo proxy-exploiting edits (P6).
- **Data-level diversity fixes transfer only partly.** R-Diverse
  (memory-augmented penalty) and Prism (cluster-coverage reward), moved onto
  edit descriptions, lift ρ^audit to ≈0.58 and cost 3–13% of in-loop gain on
  T2/T3.
- **The pattern holds across proposers** (Opus 4.7, GPT-5.2, Llama-3.3-70B;
  Llama collapses most), across scaffolds (an Aider-based loop matches
  vanilla autoresearch on ρ and H despite higher surface diversity), and over
  a 1,000-iteration run (collapse deepens, ρ 0.31).

## Methods

- **Loop.** Fork of the public `autoresearch` repo. Each iteration the proposer
  emits a one-sentence intent plus a patch. A 5-minute A100 run gives m^opt,
  and the edit is kept if it beats the incumbent by a noise threshold ε, whose
  value is not stated. T = 300. Three seeds, five on T1. About 6,200
  A100-hours in total.
- **Tasks.** T1 GPT-2-small pretraining on 0.5B OpenWebText tokens; T2
  Llama-3.2-1B LoRA on Alpaca (win-rate); T3 Qwen-2.5-1.5B on GSM8K; T4 a
  DSPy-style prompt program for Llama-3.2-3B on ARC-Easy.
- **Three evaluation tiers.**
  1. m^opt, the only signal the proposer and acceptance rule see.
  2. m^audit, two capability-overlapping benchmarks per task (e.g. MMLU +
     IFEval for T2). It is read only by HVG's scalar test and by τ_h
     calibration over the first 30 iterations, so the authors call it a
     validation signal. The gate fires 2.7 ± 0.9 times per run.
  3. m^blind, one benchmark per task (WikiText-103, Dolly win-rate, SVAMP,
     OpenBookQA), scored once, offline.
- **Faithfulness ratio** ρ = audited gain / in-loop gain, reported only
  when the in-loop gain exceeds ε.
- **Four diagnostic axes.**
  - Surface: Levenshtein over a window of 20 accepted diffs.
  - Semantic clusters: SBERT + HDBSCAN on intent descriptions.
  - Mechanism entropy: a 9-category taxonomy frozen from 187 external
    autoresearch log edits, assigned by keyword parser; κ = 0.84 vs an LLM
    annotator, 0.81 vs humans.
  - Corpus similarity to arXiv abstracts. The authors call this one
    descriptive only.
- **Baselines** (all tuned on a 3-point grid over T2's first 50 iterations):
  vanilla, HiTemp (0.7 → 1.2), R-Diverse-A, Prism-A, Reflexion, RandSearch
  (uniform draws from 200 iteration-1 edits), HO-EarlyStop and HO-Revert. The
  last two match DAPS's audit access, cadence and feedback format.
- **Construct checks.**
  - 12 known-good published edits give ρ = 0.93 ± 0.06, and 8 deliberately
    proxy-only edits give 0.11 ± 0.08.
  - The cluster decay ΔC is stable under swapped summarizer, embedder and
    clustering (Spearman > 0.93).
  - An AST detector with no LLM reproduces the 74% figure (71%).
  - Coarse, fine, random and human taxonomies all show the drop.

## Results

- **Main table (T = 300; ρ^audit, vanilla → DAPS).** T1 0.41 → 0.78, T2
  0.46 → 0.80, T3 0.38 → 0.74, T4 0.49 → 0.82. In-loop gain is unchanged
  within 1 s.d. on every task.
- **Matched audit access (Table 3, averaged over tasks; ρ^audit / ρ^blind /
  ΔC).**

  | Method | audit read | ρ^aud | ρ^bli | ΔC |
  |---|---|---|---|---|
  | Vanilla | none | 0.44 | 0.41 | 0.68 |
  | CCR + PEM | none | 0.71 | 0.67 | 0.28 |
  | HO-EarlyStop | K = 10 | 0.62 | 0.57 | 0.66 |
  | HO-Revert (HVG alone) | K = 10 | 0.67 | 0.62 | 0.55 |
  | DAPS | K = 10 | 0.79 | 0.74 | 0.21 |

  The two diversity components *without any audit access* already beat both
  published mitigations, and both audit-only baselines, on the blind tier.
- **Absolute audited gains (Table 12, our arithmetic).** Mean audit-component
  gain is +4.1 vs +7.1 points on T2, +4.5 vs +8.5 on T3 and +4.7 vs +7.6 on
  T4. On T1 the log-perplexity reduction is 0.058 vs 0.108 nats. These agree
  with ρ × in-loop gain, so the ratio headline and the raw numbers are
  internally consistent. DAPS's real gain is about 1.6–1.9× vanilla's.
- **RandSearch is the non-adaptive ceiling.** Uniform draws from a fixed edit
  corpus reach ρ^audit 0.74–0.81, about DAPS's level, with roughly half the
  in-loop gain. Most of vanilla's shortfall is induced by adaptive proposals.
  The task pairs themselves allow near-full transfer.
- **Ablation (Table 5).** CCR alone 0.58, PEM alone 0.61, HVG alone 0.67.
  The three are sub-additive (0.98 predicted, 0.79 observed). HVG alone
  costs the most in-loop gain (0.953 normalized). PEM's similarity threshold
  has a cliff: at τ_m = 0.95, ρ falls back to 0.51 (Table 8).
- **Overhead.** About 1.4% wall-clock; iterations to reach 80% of vanilla's
  terminal gain are 154 vs 163 on T2.
- **Outside ML (Appendix M, 150 iterations, 2 seeds).** A runtime-optimization
  loop shows the same signature: clusters 11 → 5, 66% caching or
  micro-optimizations, ρ 0.58 vs 0.81.

## Critique / open questions

- **Against the digest framing: BELOW.** The digest put it "directly on the
  stall front". It is not a stall paper.
  - The in-loop metric never stalls. Figure 4's curve keeps rising through
    t = 300, and the failure is that it *keeps rising* on a narrowing set of
    micro-tunings.
  - There is no halt arm, no plateau trigger and no redirect-vs-halt
    comparison. HVG's revert is triggered by the audit gap on a fixed
    K = 10 schedule.
  - What it gives the stall front is a complement. It describes the regime
    a no-improvement counter cannot see, a cheap controller-side signal
    (mechanism entropy over accepted edits) that does see it, and one more
    controller-beats-prompt result (CCR + PEM vs Reflexion and temperature).
    That is valuable, but adjacent.
- **Abstract against body: roughly AT, with one probable mislabel.** The
  "81.6% audited / 83.7% blind" figures check out: they are means of
  per-task ratio-of-ratio improvements (e.g. T3 0.38 → 0.74 = +95%). The
  plainer statement is ρ 0.44 → 0.79 and about 1.7× the absolute audited
  gain. The "2.0 to 2.6×" overstatement is 1/0.49 to 1/0.38, the full range.
  But the "69.1%" cluster-decay cut is (0.68 − 0.21)/0.68. Tables 3 and 4
  label 0.68/0.21 "averaged across tasks", yet Table 14 gives exactly
  0.68 (0.21) for T2 alone. Every Table 4 entry (S 0.66, H 1.18/1.71,
  Δ^faith 0.171/0.067) also equals the T2 row of Table 6 and Figure 2. Either
  T2 happens to sit at the four-task mean on eight numbers, or the headline
  collapse figures are single-task values labelled as averages. Figure 1
  also plots 500 iterations "over 3 seeds" on all four tasks, while the
  paper's runs are T = 300, plus T = 1000 with one seed on T2 only.
- **The faithfulness ratio compares different units.** It divides, for
  example, MMLU points by Alpaca win-rate points. The known-good and
  proxy-only controls (0.93 vs 0.11) are the only calibration that ρ ≈ 1
  means "transfers". That is a fair check, but it covers 20 hand-built
  edits.
- **The acceptance threshold is the lever named in F2, and it is never
  varied.** ε is unstated and fixed. If "keep only above a noise floor"
  sharpens the accepted distribution toward low-variance micro-tunings,
  then a noise band and a diversity collapse interact. The paper predicts
  this direction and does not test it. That is the open question for the
  `/iterate` band.
- **"Audit" is contaminated by design, and the authors say so.** HVG reads
  m^audit, so the blind column is the honest one. The audit-to-blind drop
  (0.03–0.05) is the same for vanilla, which never reads audit, and that
  supports the authors' claim that the gate does not leak.
- **Small-scale pipelines and a frontier proposer.** The targets are
  124M–3B models with a 5-minute budget, three seeds, and a 9-category
  taxonomy built for ML edits. Baselines were tuned on T2's first 50
  iterations only. Verbatim logs are promised, not released. The code repo
  exists (created 2026-05-26).
- **The mechanism is a conceptual model.** F1–F3 are argued, and the six
  predictions are checked directionally, not fitted. Corpus similarity
  correlates only moderately with human novelty ratings (ρ = −0.47) and is
  rightly kept descriptive.

## Trust signals

- **Credibility:** 3 — academic groups (MBZUAI, McGill, SFU) plus a startup;
  code repo public; not peer reviewed (ACL-format submission); three seeds;
  an unusually complete robustness appendix (construct controls, LLM-free
  detector, taxonomy and pipeline swaps, matched-audit baselines, blind
  tier). Held below 4 by the apparent T2-as-average mislabel in the headline
  collapse tables, the 500-iteration figure with no matching runs, and logs
  not yet released.

## Follow-up

- **Relevance:** 4. It is the first controlled study in the graph of the
  autoresearch loop shape that `/iterate` uses, measured over hundreds of
  iterations against audit and blind tiers. It adds a measured,
  controller-enforced fix for the operator attractor
  ([[literature/papers/gurkan2026mutation]],
  [[literature/papers/yan2026traceml]],
  [[literature/papers/min2026autonomous]]'s lr/epochs default), with matched
  baselines for the prompt-level and temperature fixes. Not 5 because it
  never tests halting or redirect-before-halt, and its gate leans on a
  validation-grade audit split.
- **For `/iterate`:**
  - Log a category label per accepted edit and compute windowed mechanism
    entropy. It costs little and it fires in the regime where
    `max_consecutive_no_improvement` stays at zero.
  - A redirect, if built, should act like CCR/PEM, by reweighting or
    rejecting candidates in the controller. Prompting the proposer did not
    work here.
  - Test whether the noise band deepens collapse (F2).
- **For HCE:** the RandSearch arm is a cheap calibrator. A non-adaptive
  search over the same task pair says how much of the val/test gap is benign
  mismatch and how much is loop-induced.
- **Candidates** (cited, not in the graph):
  - Trehan & Chopra 2026, "Why LLMs aren't scientists yet" (arXiv
    2601.03315), six failure modes across four end-to-end autonomous ML
    attempts.
  - Li et al. 2026, R-Diverse (arXiv 2602.13103), diversity illusion in
    self-play.
  - Mishra 2026, curriculum collapse (arXiv 2603.13309).
  - Gupta & Pruthi 2025, plagiarism in AI-generated research (ACL 2025).
