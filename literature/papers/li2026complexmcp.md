---
kind: paper
title: "ComplexMCP: Evaluation of LLM Agents in Dynamic, Interdependent, and Large-Scale Tool Sandbox"
authors:
  - Yuanyang Li
  - Xue Yang
  - Longyue Wang
  - Weihua Luo
  - Hongyang Chen
institutions:
  - Zhejiang Lab
year: 2026
venue: arXiv (preprint, cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2605.10787
code_url: null
citations: null
source: "raw/papers/li2026complexmcp.pdf"
added: "2026-06-03"
relevance: 3
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [benchmark, mcp, tool-use, tool-surface-scaling, sandbox]
---

# ComplexMCP: Evaluation of LLM Agents in Dynamic, Interdependent, and Large-Scale Tool Sandbox

## TL;DR

An MCP-based benchmark of 300+ validated tools across 7 stateful sandboxes that
stresses interdependent, atomic, noise-prone tool use — the regime where
expose-all-tools-in-context collapses. Top models hit ~60% vs. ~90% human.
(Skim from abstract + PDF intro.)

## Claims

- Real-world tools are atomic, interdependent, and prone to environmental noise —
  not the isolated APIs most benchmarks assume.
- A seed-driven architecture simulates dynamic environment states and unpredictable
  API failures for a deterministic-yet-diverse evaluation.
- Three diagnosed bottlenecks: tool retrieval at scale, agent overconfidence, and
  strategic failure rationalization.

## Methods

- Built on the Model Context Protocol (MCP); 300+ systematically validated tools
  derived from 7 stateful sandboxes (office suites to financial systems).
- Seed-driven state simulation for reproducible dynamic conditions.

## Results

- Top-tier models reach ~60% success vs. ~90% human performance.

## Critique / open questions

- This is an external yardstick, not an agent architecture — its value here is as a
  measuring stick for tool-surface scaling and selective retrieval claims.
- Skim only; no public code link surfaced, which limits reuse as a live benchmark.

## Trust signals

- **Credibility:** 3 — Zhejiang Lab et al.; preprint, not peer-reviewed; no code
  link surfaced. Solid empirical framing but unverified artifact availability.

## Follow-up

- Use as the empirical counterpart when arguing `scripted-tool-pipelines` and
  `selective-memory-retrieval` against naive expose-all-tools designs.
- "Tool retrieval at scale" as a named bottleneck is a direct stressor for
  `context-eviction-policy` — does eviction help or hurt retrieval here?
