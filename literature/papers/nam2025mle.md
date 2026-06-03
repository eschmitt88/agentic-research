---
kind: paper
title: "MLE-STAR: Machine Learning Engineering Agent via Search and Targeted Refinement"
authors:
  - Jaehyun Nam
  - Jinsung Yoon
  - Jiefeng Chen
  - Jinwoo Shin
  - Sercan Ö. Arık
  - Tomas Pfister
institutions: ["Google Cloud", "KAIST"]
year: 2025
venue: "arXiv:2506.15692 [cs.LG]"
peer_reviewed: false
url: "https://arxiv.org/abs/2506.15692"
code_url:
citations:
source: "raw/papers/nam2025mle.pdf"
added: "2026-04-24"
relevance: 5
credibility: 4
status: skimmed
related_experiments:
  - ablation-guided-refinement-on-toy-mle
related_concepts:
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/evolutionary-expansion]]"
tags:
  - mle-bench
  - agent-architecture
  - web-retrieval
  - ablation
---

# MLE-STAR: Machine Learning Engineering Agent via Search and Targeted Refinement

## TL;DR

MLE-STAR builds an ML engineering agent that (1) seeds an initial
solution by retrieving effective models from the web rather than
relying on LLM priors, then (2) iteratively refines specific code
components, guided by ablation studies that identify which blocks
carry the most signal. Medals on 64% of MLE-bench Lite competitions.

## Claims

- LLM-only ML agents are bottlenecked by stale, general training-time
  knowledge; web retrieval surfaces task-specific state-of-the-art.
- Whole-solution revision is a coarse exploration strategy; targeted
  refinement of individual components (feature engineering, model
  architecture, training loop) is higher signal per token.
- Ablation studies — running the current solution with each block
  removed — produce a cheap, legible ranking of which component to
  refine next.
- A novel ensembling method derived from MLE-STAR's own suggestions
  lifts headline performance further.

## Methods

- Initial solution phase: query a search engine for the task, extract
  candidate architectures and recipes from results, compose an
  initial code draft.
- Refinement loop: per cycle, run ablations on the current solution to
  rank components by contribution; pick the highest-signal block and
  explore variations (feature engineering, hyperparameters, layer
  swaps) within just that block.
- Ensemble: aggregate multiple refined solutions via a strategy
  suggested by the agent itself.

## Results

- 64% medal rate on MLE-bench Lite, substantially above the paper's
  best alternative baseline.
- Reports suggest component-targeted refinement outperforms full-code
  revision at matched compute.

## Critique / open questions

- Ablation cost scales with the number of code blocks. Budget
  implications when the codebase is large are underexplored here.
- Web retrieval quality depends on the search engine; reproducibility
  over time is an open concern as search results drift.
- "Agent-suggested ensembling strategy" risks optimizing to the
  validation signal — HCE discipline (see [[literature/papers/hambardzumyan2026aira]])
  becomes load-bearing.

## Trust signals

- **Credibility:** 4 — Google Cloud and KAIST; arXiv preprint (v3),
  not yet peer-reviewed; no code URL located on the scanned front
  matter. Strong MLE-bench Lite result from a major lab + reputable
  university; short only on peer review and a located public artifact.

## Follow-up

- Deep-read the refinement-loop section for the exact stop criterion.
- Compare the retrieval prompt against what `/discover` currently emits.
- Check how ablation rankings handle correlated blocks (two components
  each necessary but neither sufficient).
