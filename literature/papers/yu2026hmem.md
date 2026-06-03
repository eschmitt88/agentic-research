---
kind: paper
title: "H-Mem: A Novel Memory Mechanism for Evolving and Retrieving Agent Memory via a Hybrid Structure"
authors:
  - Jiawei Yu
  - Yixiang Fang
  - Xilin Liu
  - Yuchi Ma
institutions: ["The Chinese University of Hong Kong, Shenzhen", "Huawei Cloud"]
year: 2026
venue: "arXiv:2605.15701 [cs.CL]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.15701"
code_url: null
citations: null
source: "raw/papers/yu2026hmem.pdf"
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
  - hybrid-structure
  - temporal-semantic-tree
  - knowledge-graph
  - memory-evolution
---

# H-Mem: Evolving and Retrieving Agent Memory via a Hybrid Structure

## TL;DR

A hybrid agent-memory mechanism that combines a temporal-semantic tree
(short-term memory progressively summarized into long-term) with a
knowledge graph of entity relationships, retrieving over both structures;
reports SOTA on three QA memory benchmarks.

## Claims

- Prior memory work lacks a principled mechanism for modeling how memory
  evolves over time and for retrieving it effectively.
- A tree structure lets short-term memory evolve into summarized long-term
  memory; a parallel knowledge graph captures entity relationships.
- Retrieving over the combined tree + graph structure outperforms flat
  approaches.

## Methods

- Temporal-and-semantic tree for progressive short-to-long-term
  consolidation; knowledge graph for entity relations; hybrid retrieval
  exploiting both.
- Evaluated on three agent-memory QA benchmarks.

## Results

- Claims state-of-the-art on the QA task across the three benchmarks.

## Critique / open questions

- QA-only evaluation; unclear whether the structure helps agentic tasks
  beyond retrieval-QA.
- Pairs with TriMem (sun2026rethinking) as a second same-week attestation
  of multi-structure / multi-granularity memory — verify whether they cite
  each other (likely parallel discovery).

## Trust signals

- **Credibility:** 3 — preprint, but CUHK-Shenzhen + Huawei Cloud, concrete
  mechanism and multi-benchmark evaluation; no released code noted.

## Follow-up

- Third attestation of multi-granularity memory would seed a
  `multi-granularity-memory` concept (with TriMem and the memory surveys).
- Compare tree+KG hybrid against GAM's hierarchical graph memory.
