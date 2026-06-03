---
kind: paper
title: "MemORAI: Memory Organization and Retrieval via Adaptive Graph Intelligence for LLM Conversational Agents"
authors:
  - Hung Pham Van
  - Nguyen Manh Hieu
  - Khang Pham Tran Tuan
  - Nam Le Hai
  - Linh Ngo Van
  - Diep Thi-Ngoc Nguyen
  - Trung Le
institutions:
  - Independent Researcher
  - Hanoi University of Science and Technology
  - VNU University of Engineering and Technology
  - Monash University
year: 2026
venue: ACL Findings
peer_reviewed: true
url: https://arxiv.org/abs/2605.01386
code_url: null
citations: null
source: "raw/papers/pham2026memorai.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
tags: [agent-memory, multi-granularity, provenance, graph-retrieval, pagerank, acl-findings]
---

# MemORAI: Memory Organization and Retrieval via Adaptive Graph Intelligence for LLM Conversational Agents

## TL;DR

A graph-based agent-memory framework with three parts: selective memory filtering
with dual-layer compression, a provenance-enriched multi-relational graph tracking
factual origins at the turn level, and query-adaptive subgraph retrieval via Dynamic
Weighted PageRank. SOTA on LOCOMO and LongMemEval. (Skim from abstract + PDF intro.)

## Claims

- Existing graph memory suffers from information dilution, absent provenance, and
  query-agnostic uniform retrieval; MemORAI targets all three.
- Selective storage + enriched (provenance) representation + adaptive retrieval are
  jointly essential for coherent, personalized agents.
- This is a third independent attestation of the multi-granularity-memory pattern
  (alongside TriMem's raw/atomic/synthesized and H-Mem's tree+KG stacks).

## Methods

- Dual-layer compression filter to retain user-persona-relevant content.
- Provenance-enriched multi-relational graph with turn-level factual origins
  (citation/anchoring at the memory layer).
- Query-adaptive subgraph retrieval with Dynamic Weighted PageRank
  (query-conditioned edge weighting).

## Results

- State-of-the-art on LOCOMO and LongMemEval for both memory retrieval and
  personalized response generation (specific deltas not read).

## Critique / open questions

- Conversational-personalization memory rather than research-agent memory; transfer
  is at the architecture level (granularity stack + provenance graph), which is
  exactly what makes it the third attestation for a concept seed.
- Skim only; no code link surfaced.

## Trust signals

- **Credibility:** 3 — peer-reviewed (ACL Findings), which lifts it above a bare
  preprint; mixed-institution team (HUST, VNU-UET, Monash, independent); no public
  code surfaced. Benchmarks are standard (LOCOMO, LongMemEval).

## Follow-up

- Third attestation to seed `multi-granularity-memory` (TriMem + H-Mem + MemORAI).
  The parent agent handles concept creation/back-linking.
- The turn-level provenance graph is a `citation-anchoring`-at-memory pattern worth
  cross-linking once the concept exists.
