---
kind: paper
title: "CVEvolve: Zero-Code Autonomous Algorithm Discovery for Unstructured Scientific Data Processing"
authors:
  - Ming Du
  - Xiangyu Yin
  - Yanqi Luo
  - Dishant Beniwal
  - Songyuan Tang
  - Hemant Sharma
  - Mathew J. Cherukara
institutions:
  - Advanced Photon Source, Argonne National Laboratory
year: 2026
venue: arXiv (preprint, cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2605.11359
code_url: null
citations: null
source: "raw/papers/du2026cvevolve.pdf"
added: "2026-06-03"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [evolutionary-search, scientific-discovery, agentic-harness, holdout-testing, argonne]
---

# CVEvolve: Zero-Code Autonomous Algorithm Discovery for Unstructured Scientific Data Processing

## TL;DR

An autonomous agentic harness that discovers data-processing algorithms for
unstructured scientific image data via a multi-round search whose operator set is
small and named — discovery vs. improvement actions — with lineage-aware stochastic
candidate sampling and holdout-test tracking to avoid over-optimization.
(Skim from abstract + PDF intro.)

## Claims

- A zero-code interface lets domain scientists turn noisy, sparsely-labeled,
  loosely-specified scientific image data into practical algorithms.
- The search alternates between discovery and improvement actions and uses
  lineage-aware stochastic candidate sampling to balance exploration/exploitation.
- Holdout test tracking identifies candidates that generalize better than later
  over-optimized alternatives — an explicit search-vs-accept separation.

## Methods

- Agentic harness with tools for code execution, evaluation implementation,
  history management, holdout testing, and optional inspection of scientific
  data and visual outputs.
- Demonstrated on X-ray fluorescence microscopy image registration, Bragg peak
  detection, high-energy diffraction microscopy segmentation, and hybrid
  analytical/learning affine registration.

## Results

- Across these tasks CVEvolve discovers algorithms that improve over baselines.
- Holdout tracking flags over-optimized candidates so generalizing solutions win.

## Critique / open questions

- Skim only — no read of the search-controller internals. The "lineage-aware
  stochastic candidate sampling" selection policy is the most concept-relevant
  detail and is undocumented here beyond the abstract framing.
- Domain is scientific CV, not ML-research agents; transfer is at the
  architecture level (typed small operator set + holdout-guarded acceptance).

## Trust signals

- **Credibility:** 4 — Argonne National Laboratory / Advanced Photon Source
  (a national lab, strong domain prior); preprint, not peer-reviewed; no public
  code link surfaced. Institution is a prior, not a verdict.

## Follow-up

- Read the search controller: is the operator set genuinely two actions
  (discovery/improvement) or the three (`generate`/`tune`/`evolve`) the digest
  reported? Resolve against `evolutionary-search-grain`.
- The holdout-test-tracking mechanism is a clean `hce-evaluation` analogue worth
  comparing to this project's search-vs-final-scoring separation.
