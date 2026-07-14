---
kind: paper
title: "A-Mem: Agentic Memory for LLM Agents"
authors: [Wujiang Xu, Zujie Liang, Kai Mei, Hang Gao, Juntao Tan, Yongfeng Zhang]
institutions: ["Rutgers University", "Independent Researcher", "AIOS Foundation"]
year: 2025
venue: "NeurIPS 2025"
peer_reviewed: true
url: https://arxiv.org/abs/2502.12110
code_url: https://github.com/WujiangXu/AgenticMemory
citations: null
source: "raw/papers/xu2025amem.pdf"
added: "2026-07-14"
relevance: 4
credibility: 5
status: read
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/multi-granularity-memory]]"
tags: [memory, zettelkasten, note-linking, memory-evolution, second-brain, retrieval, agent-architecture]
---

# A-Mem: Agentic Memory for LLM Agents

## TL;DR

Builds LLM-agent memory explicitly on Zettelkasten principles — each
experience becomes an atomic note with LLM-generated keywords, tags, and
a contextual description; new notes trigger **link generation** (embed →
top-k candidates → LLM decides connections) and **memory evolution**
(the LLM retroactively updates neighboring notes' context/keywords/tags),
so the memory network reorganizes itself as knowledge accumulates.

## Claims

- Fixed memory operations and predefined schemas (MemGPT, MemoryBank,
  Mem0-style graph DBs) limit adaptability across diverse tasks; agentic,
  self-organizing note networks generalize better.
- On LoCoMo long-term conversational QA, A-Mem beats LoCoMo, ReadAgent,
  MemoryBank, and MemGPT baselines across six foundation models
  (GPT-4o/4o-mini, Qwen-2.5 1.5B/3B, Llama-3.2 1B/3B), with the largest
  margins on multi-hop reasoning (at least 2× on GPT models).
- Selective top-k note retrieval cuts per-question token cost by 85–93%
  (~1,200 tokens vs ~16,900 for the flat-context baselines).
- Retrieval stays fast at scale: 3.7 μs at 1M memories, linear O(N)
  storage — no overhead vs simpler stores.

## Methods

- **Note construction**: note = {content, timestamp, keywords, tags,
  contextual description, embedding, links}; the semantic components are
  LLM-generated at write time (Zettelkasten atomicity), the embedding
  encodes their concatenation.
- **Link generation**: cosine top-k over embeddings as a cheap candidate
  filter, then an LLM judges which connections are real — "boxes" of
  interlinked notes emerge without predefined rules; one note can live in
  several boxes.
- **Memory evolution**: for each near neighbor of a new note, an LLM
  decides whether to rewrite its context/keywords/tags in light of the
  new arrival — retroactive refinement, not append-only accumulation.
- **Retrieval**: passive cosine top-k per interaction (k≈10; ablated
  10–50 with plateau-and-decline at large k).
- Evaluated on LoCoMo (7,512 QA pairs, 5 question types) and DialSim;
  ablation removes link generation (LG) and memory evolution (ME).

## Results

- Table 1: A-Mem best-average ranking on nearly every model; e.g.
  GPT-4o-mini multi-hop F1 27.02 vs 9.15–26.65 for baselines at ~7×
  fewer tokens.
- Ablation (Table 3): removing both LG and ME collapses multi-hop
  (27.02 → 9.65 F1); LG alone recovers most of it — **link generation is
  the load-bearing module**, evolution adds refinement on top.
- DialSim: F1 3.45 vs LoCoMo 2.55 / MemGPT 1.18 (+35% / +192%).
- Cost: <$0.0003 per memory operation (GPT-4o-mini); 1.1 s per op on a
  locally hosted Llama-3.2 1B.

## Critique / open questions

- Evaluation is entirely conversational QA (LoCoMo, DialSim); no
  research-agent or task-execution workload, so transfer of the linking
  win to knowledge-graph synthesis (this project's use case) is
  extrapolation.
- Memory evolution rewrites neighbors **in place** — no provenance trail
  of what a note said before evolution. EvoMem
  ([[literature/papers/xu2026evoarena]]) diagnoses exactly this state-
  collapse risk; a git-tracked substrate fixes it for free.
- The LLM link judgment is unvalidated against human linking; the "boxes"
  are only inspected via t-SNE.
- Baselines are 2023–24 systems; contemporary graph-memory systems
  (Zep, Mem0-graph) are discussed but not run.

## Trust signals

- **Credibility:** 5 — peer-reviewed at a top venue (NeurIPS 2025),
  Rutgers-anchored author group, both benchmark and production code
  released (AgenticMemory + A-mem-sys), widely cited as the canonical
  "Zettelkasten for agent memory" reference (count not pinned —
  Semantic Scholar rate-limited at ingest time).

## Follow-up

- **Relevance:** 4 — the canonical peer-reviewed bridge between PKM /
  second-brain practice (Zettelkasten) and agent memory architecture;
  materially strengthens [[concepts/agent-native-memory]] (retroactive
  link generation + evolution, with ablations) and
  [[concepts/multi-granularity-memory]] (multi-representation note
  construction at write time). Likely `sources:` anchor for a future
  llm-wiki / second-brain concept once the Karpathy-pattern cluster is
  ingested.
- When the second-brain cluster (Karpathy gist, rebuild critique) is in,
  decide whether "self-evolving note network" deserves its own concept or
  stays as guidance inside agent-native-memory.
