---
kind: paper
title: "ZEBRA: Zero-shot Budgeted Resource Allocation for LLM Orchestration"
authors: ["May Hamri", "Inbal Talgam-Cohen"]
institutions: ["Tel Aviv University"]
year: 2026
venue: "SCALE workshop @ ICML 2026 (arXiv 2605.20485, cs.LG)"
peer_reviewed: false
url: https://arxiv.org/abs/2605.20485
code_url: null
citations: null
source: "raw/papers/hamri2026zebra.pdf"
added: "2026-08-11"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [budget, allocation, knapsack, water-filling, orchestration, inference-time, utility-curves, zero-shot]
---

# ZEBRA: Zero-shot Budgeted Resource Allocation

## TL;DR

Shifts the budget question from "is the ceiling respected" to "how
should the budget be *split*": an LLM controller estimates a
saturating-exponential utility curve per pipeline phase (via two
elicited operating points — tokens to ~50% and ~90% quality), then an
explicit water-filling solver allocates the monetary budget as a
continuous nonlinear knapsack; both ZEBRA objectives beat asking the
LLM to split the budget directly, on every aggregate metric, with the
gap scaling with budget tightness (94.4% vs 88.1% of unconstrained
quality at half budget on APPS) and transferring to HotpotQA (+14.3pp).

## Claims

- Budget compliance and budget *effectiveness* are different
  problems: under a fixed spend, planning depth, tool usage, and
  verification effort should differ, and none of the existing lines
  (provider-side token budgeting, discrete model routing/cascades,
  step-by-step in-agent budget control like BATS/INTENT/BAVT)
  addresses splitting a continuous budget across the *dependent
  phases* of one pipeline at inference time, zero-shot.
- **Split the roles: LLM estimates, solver optimizes.** Delegating
  the allocation itself to an LLM fails — motivated by documented
  systematic LLM failure on knapsack problems (Duchnowski & Pavlick
  2025) and confirmed by their own ablation: same fitted curves,
  LLM solver instead of water-filling, −4.2 to −4.3 points.
- Additive vs multiplicative aggregation starve different phases:
  additive starves low-marginal-utility phases; the multiplicative
  offset form starves low-ceiling phases and concentrates spend on
  the would-be weakest link — which pays off exactly on hard tasks
  under tight budgets.
- **Allocation quality only matters when the budget binds**: easy
  tasks tie, medium/hard at α=0.5 open significant gaps (+6.7 mean /
  +10.0pp success on medium); re-running easy tasks at α=0.3 flips
  the tie to a ZEBRA win. Rule of thumb: ZEBRA's advantage scales
  with tightness, whether from difficulty or from α.

## Methods

- Phase utilities f_i(x) = a_i(1 − e^(−b_i x)); b_i fit as geometric
  mean of the two elicited token operating points (converted to USD
  via model pricing); ceiling a_i elicited directly. Water-filling:
  bisection on the Lagrange multiplier (marginal-equalization),
  O(n log 1/ε); multiplicative objective log-transformed into the
  same solver.
- APPS interview tier, 150 tasks screened for cost stability and
  balanced across difficulty (defined by unconstrained pipeline mean
  score); 4-phase LangGraph pipeline (plan → decompose → implement →
  refine), gpt-4o-mini phases, gpt-4o controller; 15 runs/cell,
  paired Wilcoxon with Benjamini–Hochberg FDR control across the
  18-test family. Transfers: HumanEval, CodeContests, and a
  structurally different 3-phase plain-Python HotpotQA pipeline.
- Ablations: LLM-as-solver, uniform split, CoT-augmented LLM-direct
  (scores *worse* than plain), Fixed-Avg (global mean share, isolates
  per-task adaptation — loses +9/+22pp NB-retention on medium/hard).
- Robustness: 50% multiplicative noise on curve parameters costs only
  −1.4pp (n.s.) and still beats uniform; controller swap
  (gpt-4o ↔ mini) leaves per-phase allocations correlated at
  Pearson 0.92–0.95.

## Results

- APPS α=0.5: mult_offset retains 94.4% of no-budget quality vs
  88.1% LLM-direct (p<0.001); α=0.8: additive 98.1% vs 94.3%.
- **Why it wins: it adapts the spend.** ZEBRA shifts refine's share
  from ~30–45% (easy) to 60–66% (medium/hard); LLM-direct spends a
  fixed ~25% implement / ~50% refine regardless of tier or budget.
  On HotpotQA the learned split is near-balanced (30/39/30) and flat
  across tiers — matching the Uniform optimum there — evidence it
  adapts to pipeline structure rather than memorizing a shape.
- Controller overhead is real: curve estimation costs ~33% of the
  budget at α=0.5; still wins against Uniform given 60% more budget
  and zero overhead.
- Limitations (their own): allocates once before execution (their
  mid-pipeline reallocation variant did not help); presupposes
  discrete phases — **phaseless agentic loops are out of scope**;
  monetary curve estimation strains prohibitively tight budgets.

## Critique / open questions

- gpt-4o-mini-scale phase models and 150 screened tasks; whether
  utility curves stay concave-saturating for frontier models with
  heavy inference-time reasoning is untested (they validate
  monotonicity within-task, ruling out overthinking non-monotonicity
  for *their* pipeline only).
- No code released; the two-point curve-elicitation prompt is in the
  appendix but reproduction is nontrivial.
- The comparison target is LLM-direct allocation — not a trained RL
  allocator; the zero-shot framing is fair but the ceiling of
  learned allocation is unmeasured.

## Trust signals

- **Credibility:** 3 — university group (Talgam-Cohen: algorithmic
  game theory), SCALE @ ICML 2026 workshop, unusually careful
  statistics (paired Wilcoxon, BH-FDR, noise/controller-swap
  robustness); no released code, small models, no citation record.

## Follow-up

- **Relevance:** 4 — names the counter-pole to
  [[concepts/budget-as-ceiling]] on an active axis: ceilings/halting
  govern *whether* spend continues; ZEBRA governs *where* spend goes
  under a binding ceiling. Directly relevant to the pending /elevate
  item (khan2026token pre-flight reservation vs halt-after-cycle):
  allocation is the part of "spend planning" that survives
  ye2026agent's reservation-impossibility, because it needs no
  mid-call enforcement — only per-phase caps the harness already has.
- The "LLM estimates, solver optimizes" split is the same shape as
  the instruction-ablation phase-3 move (prose-programs → scripts)
  and [[concepts/scripted-tool-pipelines]]: put arithmetic in code,
  judgment in the model.
- Local analogue: the coordinator's GO verdict already emits a
  *suggested session budget* — a single-phase allocation; ZEBRA is
  what the multi-phase version would look like if /implement-style
  chains ever split their `budget.yaml` ceiling across phases.
