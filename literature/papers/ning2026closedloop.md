---
kind: paper
title: "Closed-loop Auto Research for Molecular Property Prediction: Discovering and Certifying Generalizable Improvements"
authors: ["Jingjie Ning", "Xiaochuan Li", "Ji Zeng", "Chenyan Xiong", "Guolin Ke"]
institutions: ["Carnegie Mellon University", "DP Technology"]
year: 2026
venue: "arXiv preprint (2606.22731)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.22731"
code_url: null
citations: null
source: "raw/papers/ning2026closedloop.pdf"
added: "2026-07-06"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
tags: [evaluation, hce, overfitting, held-out, auto-research, generalization, drug-discovery]
---

# Closed-loop Auto Research for Molecular Property Prediction: Discovering and Certifying Generalizable Improvements

## TL;DR

A closed-loop auto-research agent edits molecular representations, model
code, and external evidence over a strong compact baseline, then applies a
**held-out certification** step that re-scores each validation-selected
configuration exactly once on a test partition the search never read. The
central finding: validation gains routinely fail to transfer, and the paper
gives two empirically distinct non-transfer signatures — selection variance
and distribution shift — making "separate discovery from held-out
certification" a domain-agnostic lesson for any loop optimizing a proxy for
a held-out quantity.

## Claims

- Auto-research extends AutoML from fixed-dataset fitting to *changing the
  research workflow* — an LLM agent edits the representation, the estimator,
  or acquires external data — and this expanded action space changes the
  evaluation question: does a validation-selected action generalize?
- A closed loop reports the validation return by construction and **cannot,
  on its own, distinguish a real effect from validation-signal gaming.**
  Two distortions are named: (1) a max over many trials on a small
  validation split reflects sampling noise (selection variance); (2)
  acquired external data can come from a different distribution or re-import
  the benchmark's own measurements (distribution shift / contamination).
- Held-out certification separates transfer from non-transfer. Concrete
  numbers: TDC model-axis gain falls 0.041 (val) → 0.003 (test); Polaris
  data-axis reaches 0.022 (val) but −0.019 (test) — two non-transfer
  signatures. A validation-routed pipeline (best axis per endpoint) still
  reaches positive held-out gains of +0.013 (TDC), +0.011 (MoleculeNet),
  +0.042 (Polaris), with the *transferable axis regime-dependent*.
- A matched-trial AutoML control does **not** reproduce the agent's
  code-level model intervention (0.006 vs 0.042), i.e. the agent's
  gain lies outside the fixed-dataset AutoML action space by construction.

## Methods

- **Axis-isolated auto-research**: a file-level ablation lock permits only
  one active intervention axis per trial (feature | model | data), so each
  measured gain and its transfer attribute to a single research direction.
- **Held-out-test protocol**: freeze the validation-selected config, retrain
  from scratch on the internal training split, score once on the held-out
  test partition. Test labels are never read inside the loop; only an
  evaluator-owned contamination filter reads test *structures* (not labels)
  to reject same-source external files overlapping 64–89% of test molecules.
- 36 endpoints across three suites (TDC ADMET, MoleculeNet, Biogen
  adme-fang via Polaris); scaffold-based splits; normalized
  baseline-relative improvement per endpoint, aggregated as unweighted mean.
- Framing: two strategy families — **constrain-during-search** (reusable
  holdout / differential privacy that limit access to the holdout) vs
  **certify-after-search** (this paper: open-ended loop, frozen selections
  scored once). They adopt the latter to keep the research loop open-ended.

## Results

- Positive routed held-out gains on all three suites, transferable axis
  differing by regime: data on TDC, model on Polaris, feature+model on
  MoleculeNet.
- Two documented non-transfer cases (TDC model 0.041→0.003; Polaris data
  0.022→−0.019) matched to selection variance and distribution shift.
- Endpoint-level discoveries survive certification: curated external data
  raises held-out CYP2C9-substrate by 0.17 and half-life by 0.08 — but only
  after the contamination filter, which is *necessary but not sufficient*
  for transfer.
- CPU pipeline stays competitive with an 84M-parameter pretrained 3D model
  on the shared training split.

## Critique / open questions

- No code URL surfaced yet — the leakage-safe contamination filter and axis
  lock are the load-bearing artifacts and would need release to reproduce.
- The certify-after-search protocol scores each config on the test split
  *once*; with many endpoints and axes, the number of one-shot test reads
  grows — the paper does not deeply analyze the multiple-comparisons
  exposure of the certification pass itself.
- Findings are within molecular property prediction; the domain-agnostic
  claim is argued, not demonstrated cross-domain.

## Trust signals

- **Credibility:** 3 — reputable authorship (CMU CS + DP Technology, Guolin
  Ke of Uni-Mol lineage), thorough held-out methodology with quantified
  non-transfer signatures and a matched AutoML control; but arXiv preprint,
  not peer-reviewed, and no released code/artifacts yet.

## Follow-up

- **Relevance:** 4 — materially strengthens [[concepts/hce-evaluation]] by
  supplying a *certify-after-search* mechanism and the
  constrain-during-search vs certify-after-search distinction, plus
  quantified non-transfer signatures, in a domain (chemistry) outside the
  concept's existing ML-research anchors. Cross-domain attestation that the
  validation/test separation is methodological, not benchmark-specific.
- Cross-references AIRA_2's decouple-search-from-selection framing
  ([[literature/papers/hambardzumyan2026aira]]) from a chemistry vantage.
