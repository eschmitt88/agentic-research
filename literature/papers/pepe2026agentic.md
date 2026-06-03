---
kind: paper
title: "Agentic Discovery of Neural Architectures: AIRA-Compose and AIRA-Design"
authors:
  - Alberto Pepe
  - Chien-Yu Lin
  - Despoina Magka
  - Bilge Acun
  - Yannan Nellie Wu
  - Anton Protopopov
  - Carole-Jean Wu
  - Yoram Bachrach
institutions: ["FAIR at Meta"]
year: 2026
venue: "arXiv:2605.15871 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.15871"
code_url: null
citations: null
source: "raw/papers/pepe2026agentic.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/hierarchical-delegation]]"
tags:
  - aira
  - neural-architecture-search
  - recursive-self-improvement
  - agent-design
  - foundation-models
---

# Agentic Discovery of Neural Architectures: AIRA-Compose and AIRA-Design

## TL;DR

Two complementary AIRA-family frameworks in which LLM agents autonomously
design foundation-model architectures beyond Transformers — AIRA-Compose
for high-level architecture search, AIRA-Design for low-level mechanistic
implementation — with discovered models beating Llama 3.2 and scaling
faster.

## Claims

- LLM agents can autonomously discover architectures and algorithmic
  optimizations that rival or surpass hand-designed baselines.
- Splitting the search into high-level architecture exploration
  (Compose) and low-level implementation (Design) is an effective division
  of the search target.
- This is a concrete step toward recursive self-improvement.

## Methods

- AIRA-Compose: 11 agents explore primitives (Attention, MLP, Mamba) under
  a 24-hour budget; evaluate million-param candidates, extrapolate top
  designs to 350M/1B/3B. Yields 14 architectures (AIRAformers,
  AIRAhybrids).
- AIRA-Design: up to 20 agents write novel attention mechanisms and
  training scripts; evaluated on Long Range Arena and the Autoresearch
  benchmark.

## Results

- 1B AIRA models beat Llama 3.2 and Composer baselines; AIRAformer-D /
  AIRAhybrid-D improve downstream accuracy by 2.4% / 3.8% over Llama 3.2.
- AIRAformer-C scales 54%/71% faster than Llama 3.2 / best Composer
  Transformer; AIRAhybrid-C outscales Nemotron-2 by 23%.
- On Autoresearch, Greedy Opus 4.5 reaches 0.968 validation bits-per-byte,
  surpassing the published minimum.

## Critique / open questions

- 24-hour-budget runs are expensive to reproduce; gains are relative to
  Llama 3.2 / Composer baselines rather than the strongest available.
- "Recursive self-improvement" framing is aspirational — these are
  single-shot discovered architectures, not a closed self-improving loop.

## Trust signals

- **Credibility:** 4 — FAIR at Meta, large rigorous study (55 pages, many
  tables), strong baselines; not peer-reviewed and no code link noted.

## Follow-up

- The Compose/Design (high-level vs low-level) split is a candidate
  concept seed — distinct search grain per agent loop.
- Read alongside hambardzumyan2026aira (AIRA_2) for the AIRA cluster
  trajectory.
