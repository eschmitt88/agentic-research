---
kind: paper
title: "Evolutionary Task Discovery: Advancing Reasoning Frontiers via Skill Composition and Complexity Scaling"
authors:
  - Liqin Ye
  - Yanbin Yin
  - Michael Galarnyk
  - Yuzhao Heng
  - Sudheer Chava
  - Chao Zhang
institutions:
  - Georgia Institute of Technology
year: 2026
venue: arXiv (preprint, cs.LG)
peer_reviewed: false
url: https://arxiv.org/abs/2605.11666
code_url: https://github.com/liqinye/EvoTD
citations: null
source: "raw/papers/ye2026evolutionary.pdf"
added: "2026-06-03"
relevance: 3
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: [evolutionary-search, skill-composition, data-synthesis, curriculum, reasoning]
---

# Evolutionary Task Discovery: Advancing Reasoning Frontiers via Skill Composition and Complexity Scaling

## TL;DR

EvoTD treats training-data synthesis as a directed evolutionary search over a
dual-axis manifold of algorithmic skills and complexity attributes — crossover
composes skills, parametric mutation scales structural constraints, and a Zone of
Proximal Development filter keeps tasks learnable. (Skim from abstract + PDF intro.)

## Claims

- Inverts the usual framing: it *generates tasks from skills* rather than acquiring
  skills to solve fixed tasks — skill composition is the search primitive.
- Structured evolutionary curricula deliver reasoning gains that generalize across
  model architectures, pretraining regimes, and scales.
- A dynamic ZPD filter avoids both homogeneity collapse and unlearnable difficulty.

## Methods

- Dual-axis manifold: Algorithmic Skills × Complexity Attributes.
- Crossover operator synthesizes novel skill compositions (diversity); Parametric
  Mutation operator scales structural constraints — e.g. input size, tree depth
  (generalization).
- ZPD filter keeps generated tasks in the model's learnable region.

## Results

- Substantial, consistent reasoning gains across architectures and scales (abstract
  claim; specific numbers not read).

## Critique / open questions

- This is data-synthesis-for-reasoning, adjacent to the project's research-agent
  focus rather than central. Relevance is at the architectural pattern level
  (evolution over a skill manifold), not as a research-agent system per se.
- Skim only — no read of operator implementations or benchmark deltas.

## Trust signals

- **Credibility:** 3 — Georgia Tech; preprint, not peer-reviewed; code released
  (github.com/liqinye/EvoTD), which raises confidence. Adjacency to the mission
  caps relevance more than credibility.

## Follow-up

- Compare the crossover-as-skill-composition operator to `skill-library-lifecycle`:
  does treating skills as a generative vocabulary (not a lookup table) change the
  lifecycle stages this project tracks?
- Cross-link with CVEvolve and the evolutionary-search cluster toward a possible MoC.
