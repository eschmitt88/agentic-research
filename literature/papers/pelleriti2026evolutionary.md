---
kind: paper
title: "What Do Evolutionary Coding Agents Evolve?"
authors:
  - Nico Pelleriti
  - Sree Harsha Nelaturu
  - Zhanke Zhou
  - Zongze Li
  - Max Zimmer
  - Bo Han
  - Sebastian Pokutta
institutions: ["Zuse Institute Berlin", "TU Berlin", "Hong Kong Baptist University", "RIKEN AIP"]
year: 2026
venue: "arXiv:2605.20086 [cs.NE]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.20086"
code_url: null
citations: null
source: "raw/papers/pelleriti2026evolutionary.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
tags:
  - evolutionary-coding
  - critical-methodology
  - evaluator-overfitting
  - mechanism-analysis
  - diagnostic-evaluation
---

# What Do Evolutionary Coding Agents Evolve?

## TL;DR

A critical-methodology paper: benchmark gains in LLM evolutionary coding
agents arise from qualitatively different mechanisms, only some of which
are new algorithmic structure. Introduces EvoTrace (a trace dataset) and
EvoReplay (replay-based intervention) to diagnose what actually drives
score gains — and finds heavy recycling and constant re-tuning.

## Claims

- The best-score-under-an-evaluator summary conflates new structure,
  re-tuning, recombining internal knowledge, and evaluator overfitting.
- About 30% of code lines added during search are byte-identical
  reintroductions of previously-deleted lines (a deterministic cycling
  pattern present in nearly every run).
- Most score gains come from a small subset of nine annotated edit types;
  final benchmark scores mask the actual driving mechanism.

## Methods

- EvoTrace: traces across four evolutionary frameworks, reasoning and
  non-reasoning models, 16 math/algorithm tasks.
- EvoReplay: reconstructs local search states behind high-scoring
  solutions, runs controlled interventions (adjust constants, remove
  components, substitute models/prompts).
- Every edit annotated with one of nine edit types via an LLM-as-judge
  pipeline validated against blind human re-annotation.

## Results

- Gains concentrate in a few edit types; pervasive code recycling; evidence
  of evaluator overfitting — i.e. apparent "evolution" is often re-tuning
  and recombination, not novel structure.

## Critique / open questions

- Directly tempers how this project should read AlphaEvolve / FunSearch /
  CodeEvolve / CORAL: final-metric deltas are not the load-bearing signal;
  the mechanism behind the delta is.
- Strengthens the `hce-evaluation` case — search-time evaluator
  overfitting is exactly the failure mode HCE guards against.

## Trust signals

- **Credibility:** 4 — preprint, but strong group (Pokutta / ZIB + TU
  Berlin), rigorous diagnostic methodology with human-validated
  annotation; no code link noted.

## Follow-up

- Re-read the evolutionary-search cluster notes with this critique applied;
  consider an ADR on "mechanism over final metric."
- Pairs with APEX (li2026apex) — both critique what self-improvement loops
  actually produce.
