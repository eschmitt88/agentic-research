---
kind: paper
title: "OR-Agent: Bridging Evolutionary Search and Structured Research for Automated Algorithm Discovery"
authors:
  - Qi Liu
  - Ruochen Hao
  - Can Li
  - Wanjing Ma
institutions: ["Tongji University"]
year: 2026
venue: "arXiv:2602.13769 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2602.13769"
code_url: "https://github.com/qiliuchn/OR-Agent"
citations: null
source: "raw/papers/liu2026oragent.pdf"
added: "2026-06-03"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags:
  - evolutionary-search
  - algorithm-discovery
  - research-tree
  - reflection
  - multi-agent
  - open-source
---

# OR-Agent: Bridging Evolutionary Search and Structured Research

## TL;DR

A configurable multi-agent framework that goes beyond mutation-crossover
loops by organizing research as a tree-based workflow with explicit
branching, backtracking, and a hierarchical-optimization-inspired
reflection system (short-term reflections as verbal gradients, long-term as
verbal momentum, memory compression as semantic weight decay).

## Claims

- Automated discovery needs structured hypothesis management, environment
  interaction, and principled reflection — not just iterative program
  mutation.
- A tree-based workflow with branching and systematic backtracking gives
  controllable research trajectories beyond mutation-crossover.
- An evolutionary-systematic ideation mechanism unifies evolutionary
  selection of starting points, plan generation, and coordinated tree
  exploration.

## Methods

- Structured research tree modeling branching hypothesis generation and
  backtracking.
- Hierarchical reflection: short-term = verbal gradients, long-term =
  verbal momentum, memory compression = semantic weight decay.
- Evaluated on classical combinatorial-optimization benchmarks and
  simulation-based cooperative-driving scenarios.

## Results

- Outperforms strong evolutionary baselines while remaining general,
  extensible, and inspectable.

## Critique / open questions

- The optimization-analogy reflection (gradient/momentum/weight-decay) is
  evocative but its mechanism over plain reflection is asserted; ablation
  detail needed on a skim.
- A useful comparison point on `evolutionary-search-grain` — the mutation
  unit is a research-tree node rather than a code edit.

## Trust signals

- **Credibility:** 3 — preprint, single institution (Tongji), but
  open-source code released and evaluated on two distinct domains.

## Follow-up

- Compare tree-based research workflow against CORAL and CodeEvolve search
  grains.
- The reflection-as-optimization framing is a candidate concept seed if
  echoed elsewhere.
