---
kind: paper
title: "APEX: Autonomous Policy Exploration for Self-Evolving LLM Agents"
authors:
  - Yibo Li
  - Jiashuo Yang
  - Zhi Zheng
  - Zhiyuan Hu
  - Yuan Sui
  - Shizun Wang
  - Yufei He
  - Bryan Hooi
institutions: ["National University of Singapore", "Beijing University of Posts and Telecommunications"]
year: 2026
venue: "arXiv:2605.21240 [cs.LG]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.21240"
code_url: "https://github.com/liushiliushi/APEX"
citations: null
source: "raw/papers/li2026apex.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hce-evaluation]]"
tags:
  - self-evolving-agents
  - exploration-collapse
  - strategy-dag
  - exploration-exploitation
  - webarena
---

# APEX: Autonomous Policy Exploration for Self-Evolving LLM Agents

## TL;DR

Names "exploration collapse" — as memory grows, self-evolving agents
concentrate on familiar high-reward routines and stop exploring — and
counters it with APEX, which maintains an explicit strategy space as a DAG
of milestones, expanding it via Fork Discovery and choosing paths via
Policy Selection.

## Claims

- Self-evolving agents that accumulate memory/reflection across episodes
  suffer exploration collapse as memory grows.
- An explicit strategy map (DAG of milestones with prerequisite edges)
  plus exploration-grounded forking sustains exploration.
- APEX outperforms all baselines on text-adventure and web-interaction
  benchmarks.

## Methods

- Strategy map: directed acyclic graph of milestones with prerequisite
  dependency edges.
- Fork Discovery expands the map with evidence-grounded unexplored
  directions; Policy Selection balances exploration/exploitation in
  planning.
- Evaluated on nine Jericho text-adventure games and WebArena.

## Results

- Outperforms all baselines; ablations validate each component and show
  robustness across settings.

## Critique / open questions

- "Exploration collapse" intersects `evolutionary-search-grain` — narrow
  operators collapse to local optima — and AIRA2's "loop overfits its own
  signal" bottleneck.
- The DAG-of-strategies framing is a candidate dual to lineage-aware
  sampling in evolutionary search; transfer to ML-research loops is
  untested here (game/web tasks only).

## Trust signals

- **Credibility:** 3 — preprint, NUS (Bryan Hooi group) + BUPT, code
  released, ablation-backed; benchmarks are games/web, not research tasks.

## Follow-up

- Read with pelleriti2026evolutionary — both critique what
  self-improvement actually produces (recycling vs. exploration collapse).
- Consider whether the strategy-DAG maps onto experiment-proposal lineage.
