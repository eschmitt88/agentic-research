---
kind: paper
title: "CodeEvolve: an open-source evolutionary framework for algorithmic discovery and optimization"
authors:
  - Henrique Assumpção
  - Diego Ferreira
  - Leandro Campos
  - Fabricio Murai
institutions: ["Inter&Co", "Federal University of Minas Gerais", "Worcester Polytechnic Institute"]
year: 2025
venue: "arXiv:2510.14150 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2510.14150"
code_url: "https://github.com/inter-co/science-codeevolve"
citations: null
source: "raw/papers/assumpcao2025codeevolve.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags:
  - evolutionary-coding
  - open-source
  - map-elites
  - island-model
  - alphaevolve
  - llm-ensemble
---

# CodeEvolve: an open-source evolutionary coding agent

## TL;DR

An open-source evolutionary coding framework coupling LLMs with
island-based search, inspiration-based crossover, meta-prompting, and
depth-based refinement on a CVT-MAP-Elites archive; matches or beats
reported AlphaEvolve on 5/9 problems and outperforms OpenEvolve /
ShinkaEvolve on 6/9 under matched conditions.

## Claims

- A released, reproducible framework can match closed-source AlphaEvolve on
  several benchmark problems.
- The interaction between components (crossover, meta-prompting,
  refinement, archive, ensemble), not any single operator, drives results.
- With an open-weight backbone it surpasses AlphaEvolve on CirclePacking at
  roughly an order of magnitude lower cost than a frontier closed ensemble.

## Methods

- Island-based evolutionary search over a CVT-MAP-Elites archive with a
  weighted LLM ensemble.
- Inspiration-based crossover, meta-prompting, depth-based refinement.
- Evaluated on the AlphaEvolve benchmark suite; ablations isolate
  component interaction.

## Results

- Matches/surpasses AlphaEvolve on 5/9; beats OpenEvolve & ShinkaEvolve on
  6/9 under matched conditions.
- Surpasses AlphaEvolve on both CirclePackingSquare instances with
  Qwen3-Coder-30B at ~10x lower cost; competitive with EoH on
  heuristic-design without retuning.

## Critique / open questions

- High-credibility reference implementation for the evolutionary cluster
  precisely because the code is released and benchmarked against
  AlphaEvolve.
- Read against pelleriti2026evolutionary's critique — are CodeEvolve's
  gains new structure or recycling/re-tuning? The mechanism question
  applies here too.

## Trust signals

- **Credibility:** 4 — preprint (v5), but open-source code released,
  matched-condition comparisons against multiple frameworks, ablation-backed.

## Follow-up

- Candidate reference implementation if a downstream project explores
  evolutionary search.
- Diagnose gains via EvoReplay-style mechanism analysis (pelleriti2026).
