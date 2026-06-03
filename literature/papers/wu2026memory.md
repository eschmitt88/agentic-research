---
kind: paper
title: "Memory in the LLM Era: Modular Architectures and Strategies in a Unified Framework [Experiment, Analysis & Benchmark]"
authors:
  - Yanchen Wu
  - Tenghui Lin
  - Yingli Zhou
  - Fangyuan Zhang
  - Qintian Guo
  - Xun Zhou
  - Sibo Wang
  - Xilin Liu
  - Yuchi Ma
  - Yixiang Fang
institutions: ["The Chinese University of Hong Kong, Shenzhen", "CUHK", "HITSZ", "BIT"]
year: 2026
venue: "arXiv:2604.01707 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.01707"
code_url: null
citations: null
source: "raw/papers/wu2026memory.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
tags:
  - agent-memory
  - benchmark
  - unified-framework
  - modular-architecture
  - empirical-comparison
---

# Memory in the LLM Era: Modular Architectures in a Unified Framework

## TL;DR

An experiment/analysis/benchmark paper that places existing agent-memory
methods in a single unified framework, compares representative methods
head-to-head under identical settings on two benchmarks, and assembles a
new memory method from existing modules that beats the SOTA.

## Claims

- Existing memory methods have not been systematically compared under the
  same experimental settings.
- A high-level unified framework can incorporate all existing agent-memory
  methods, exposing them as composable modules.
- A new method assembled from existing modules outperforms the
  state-of-the-art — i.e. the gains live in module composition.

## Methods

- Summarize a unified framework covering existing memory methods.
- Compare representative methods on two well-known benchmarks under matched
  settings.
- Construct a byproduct method by recombining modules from existing
  methods.

## Results

- Thorough effectiveness analysis across methods; the recombined method
  outperforms the prior SOTA.

## Critique / open questions

- Two benchmarks only; the "assembled from modules beats SOTA" result
  echoes CodeEvolve's "interaction of components drives results" — worth
  noting the recurring composition-over-novelty pattern.
- Yardstick value is high: a matched-settings comparison is exactly what
  the memory cluster lacked.

## Trust signals

- **Credibility:** 3 — preprint, CUHK-Shenzhen / CUHK / HITSZ / BIT,
  matched-condition empirical comparison plus a benchmark; no code link
  noted.

## Follow-up

- Use as the comparison yardstick for memory designs (H-Mem, TriMem, GAM).
- The unified modular framing is a candidate anchor for a memory MoC.
