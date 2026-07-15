---
kind: paper
title: "Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory"
authors: [Prateek Chhikara, Dev Khant, Saket Aryan, Taranjeet Singh, Deshraj Yadav]
institutions: ["Mem0.ai"]
year: 2025
venue: "arXiv preprint (2504.19413)"
peer_reviewed: false
url: https://arxiv.org/abs/2504.19413
code_url: https://github.com/mem0ai/mem0
citations: null
source: "raw/papers/chhikara2025mem0.pdf"
added: "2026-07-15"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/multi-granularity-memory]]"
tags: [memory, consolidation, write-policy, retrieval, latency, production-deployment, graph-memory, LOCOMO]
---

# Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

## TL;DR

An incremental memory pipeline (extraction → LLM-adjudicated
ADD/UPDATE/DELETE/NOOP against retrieved neighbors) plus a graph-memory
variant, evaluated on LOCOMO against six baseline categories including
A-Mem; the deployed, most-starred (60.8k GitHub stars) open-source
memory layer in this graph's corpus so far, trading a chunk of quality
(vs. full-context) for large latency and token savings.

## Claims

- Full-context re-processing of entire conversation history is not a
  viable production baseline: attention degrades over distant tokens,
  and real conversations interleave unrelated topics so relevant facts
  get buried even within generous context windows.
- A memory system should be evaluated on more than accuracy: latency
  (search + total, p50/p95) and token cost are first-class production
  constraints, not afterthoughts.
- Graph-structured memory (entities as nodes, relationships as edges)
  helps specifically on multi-hop queries that require traversing
  relational paths across memories — the accuracy gain over base Mem0
  is small overall (~2%) but concentrated there.

## Methods

Two-phase incremental pipeline, processing one message pair
`(m_{t-1}, m_t)` at a time (no full re-ingestion):

1. **Extraction.** An LLM extraction function φ takes a prompt
   `P = (S, {m_{t-m}...m_{t-2}}, m_{t-1}, m_t)` — a periodically
   refreshed async conversation summary `S` plus a recency window of
   raw messages — and extracts candidate salient facts `Ω`.
2. **Update.** For each candidate fact, retrieve the top-*s* most
   similar existing memories by vector embedding, then hand both to
   the LLM via a tool-call interface that must pick exactly one of
   four operations: **ADD** (no semantic match), **UPDATE** (augments
   an existing memory), **DELETE** (contradicted by new information),
   **NOOP** (no change needed).
3. **Mem0^g (graph variant)** stores memories as a directed labeled
   graph instead of/alongside flat records, for relational multi-hop
   queries.

Evaluated on LOCOMO (Maharana et al. 2024) against six baseline
categories: established memory-augmented systems (LoCoMo, ReadAgent,
MemoryBank, MemGPT), RAG at varying chunk sizes/*k*, full-context,
A-Mem (open-source memory solution), a proprietary system (OpenAI),
and a dedicated memory platform (Zep/LangMem-class). Judge: LLM-as-a-
Judge (J) plus BLEU-1/F1, using gpt-4o-mini for evaluation consistency.

## Results

- Mem0 beats OpenAI's system by **26% relative** on the LLM-as-a-Judge
  metric; Mem0^g scores **~2% higher** than base Mem0 overall.
- **91% lower p95 latency** and **>90% token cost savings** vs. the
  full-context baseline.
- Head-to-head against A-Mem on LOCOMO (Table 1, LoCoMo/temporal
  slice): A-Mem's B1/J scores (27.02 / 20.09, best-tuned variant "A-Mem*"
  20.76/14.90) trail the strongest RAG and Mem0 configurations in this
  paper's harness — a data point for the design tradeoff below, though
  cross-paper benchmark numbers should be read with the usual caution
  (different harnesses, prompts, judges).

## Critique / open questions

- **This is the production-scale counterpoint to A-Mem's Zettelkasten
  design**, not a refutation of it: A-Mem
  ([[literature/papers/xu2025amem]]) optimizes for atomic, richly
  cross-linked notes with LLM-driven retroactive evolution; Mem0
  optimizes for low-latency incremental consolidation at conversation
  scale with an explicit four-operation write policy. Both use an LLM
  to decide how new information changes existing memory — Mem0's
  ADD/UPDATE/DELETE/NOOP is a coarser-grained, cheaper cousin of the
  coverage/preservation/faithfulness verification in
  [[literature/papers/yang2026trustmem]]
  ([[concepts/verified-memory-writes]]): both gate writes on a
  judgment call rather than appending unconditionally, but Mem0's gate
  is a single LLM tool-call with no explicit faithfulness check against
  the new evidence.
- **Self-reported, industry benchmark.** Not peer-reviewed; authors are
  the company selling this as a product (Mem0.ai). The LOCOMO
  evaluation is a real, external benchmark (not the authors' own), and
  the 60.8k-star, actively-maintained (pushed within the last day) open
  repo is unusually strong reproducibility evidence for an arXiv
  preprint — but the specific accuracy/latency numbers are the
  authors' own harness and haven't been independently reproduced in
  this graph.
- **No discussion of retrieval-time gating** — Mem0 always retrieves
  top-*s* similar memories on write (for dedup/update) but the paper
  doesn't address *read*-time selectivity (whether every query
  consults memory) the way
  [[literature/papers/zhao2026expweaver]] does for
  [[concepts/selective-memory-retrieval]]. Worth checking whether
  Mem0's production deployment has a read-side gate not covered here.

## Trust signals

- **Credibility: 4** — not peer-reviewed and self-benchmarked by a
  company with a commercial stake in the result, which caps it below
  5. But: code released and *very* widely deployed (60,849 GitHub
  stars, actively maintained), evaluated on an external benchmark
  (LOCOMO) against real baselines rather than only ablations, and the
  qualitative direction (incremental consolidation beats full-context
  on latency/cost) is architecturally uncontroversial even without
  independent reproduction of the exact percentages.

## Follow-up

- **Relevance: 4** — strengthens [[concepts/agent-native-memory]] and
  [[concepts/verified-memory-writes]] with a second, production-scale
  write-policy design (LLM-adjudicated ADD/UPDATE/DELETE/NOOP) distinct
  from A-Mem's evolution mechanism and TrustMem's three-axis
  verification; the graph-memory variant's multi-hop-specific gain is
  a useful data point for [[concepts/multi-granularity-memory]].
- Consider whether `verified-memory-writes`' coverage/preservation/
  faithfulness rubric could be retrofit onto Mem0's four-operation gate
  as a stricter alternative to the raw LLM tool-call decision — an
  open design question, not yet explored here.
