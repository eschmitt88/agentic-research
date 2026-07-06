---
kind: paper
title: "Are We Ready For An Agent-Native Memory System?"
authors: ["Wei Zhou", "Xuanhe Zhou", "Shaokun Han", "Hongming Xu", "Guoliang Li", "Zhiyu Li", "Feiyu Xiong", "Fan Wu"]
institutions: ["Shanghai Jiao Tong University", "Tsinghua University", "MemTensor (Shanghai) Technology"]
year: 2026
venue: "arXiv preprint (2606.24775)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.24775"
code_url: "https://github.com/OpenDataBox/MemoryData"
citations: null
source: "raw/papers/zhou2026ready.pdf"
added: "2026-07-06"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [memory, benchmark, data-management, taxonomy, lifecycle, eviction, consolidation, retrieval-routing]
---

# Are We Ready For An Agent-Native Memory System?

## TL;DR

A systematic empirical study that reframes LLM-agent memory as a
**data-management system** rather than a retrieval add-on, decomposes it
into four modules (representation/storage, extraction, retrieval/routing,
maintenance), and benchmarks 12 representative memory systems + 2 baselines
across 5 workloads and 11 datasets. Headline: no single architecture
dominates — effectiveness depends on aligning memory structure to the
workload bottleneck — and existing end-to-end metrics hide the system-level
concerns (operational cost, update robustness, long-horizon stability) that
actually decide production fitness.

## Claims

- Prior memory evaluation benchmarks end-to-end task success (F1, BLEU) and
  treats memory as a monolithic black box, hiding index-construction time,
  query latency, update correctness, and architectural trade-offs. Memory
  should be evaluated per-module as data-management infrastructure.
- **No memory system is effective across all workloads** — composite hybrid
  systems lead on conversational QA; graph-based methods excel at single-hop
  factual recall but struggle with targeted overwrites.
- **Retrieval accuracy degrades as the temporal distance between evidence
  and query grows** — a limitation of similarity-based retrieval that
  explicit query planning only partly mitigates.
- **Graph-based methods handle updates most reliably;** append-only stores
  and fact-extraction plugins suffer "hallucinations of the past" (stale
  facts) without lifecycle management.
- **Highly structured systems incur orders-of-magnitude higher index
  construction time and query latency than lightweight stores, without
  proportional accuracy gains** — a direct cost/benefit warning.
- **Localized maintenance is more cost-efficient than global
  reorganization;** conservative consolidation is the safe default, and
  delayed flushing creates a deceptive coverage-vs-answerability trade-off.

## Methods

- Four-module analytical framework $\mathcal{M}_{sys} = \langle
  \mathcal{R}, \mathcal{S}, \mathcal{Q}, \mathcal{U} \rangle$:
  Representation/storage, extraction, retrieval/routing, maintenance. Each
  module has a structured taxonomy (Table 1) categorizing the 12 systems
  (MemoChat, Mem0, MEM1, MemAgent, MemTree, Zep, Memⁿᵍ, Cognee, LightMem,
  SimpleMem, MemOS, A-MEM, Letta) by design principle.
- End-to-end evaluation across five perspectives: task effectiveness (RQ1),
  retrieval fidelity (RQ2), dynamic-update robustness (RQ3), long-horizon
  stability (RQ4), operational cost (RQ5).
- Fine-grained ablations that modify one module at a time to isolate its
  effect on representation fidelity, routing precision, update correctness.
- Maintenance taxonomy: conflict resolution/versioning, capacity management
  (constraint-based hard eviction e.g. FIFO/token-limit, or score-based
  priority eviction via temporal decay), semantic consolidation.

## Results

- Six findings (RQ1–RQ6, quoted above): workload-dependence, temporal-gap
  retrieval decay, graph superiority on updates, cost non-proportionality,
  append-only long-horizon collapse, and progressive information loss at
  each abstraction level (compression/summarization/extraction).
- Notably: for time-dependent queries, **raw long-context retrieval still
  outperforms most memory-backed approaches** because standard semantic
  consolidation destroys chronological cues.
- Code and a curated paper list released
  (github.com/OpenDataBox/MemoryData, awesome-agent-memory).

## Critique / open questions

- A benchmark/survey, not a new method — its value is the taxonomy and the
  cost/robustness measurements, not a proposed architecture.
- The five workloads and 11 datasets are still largely conversational-QA and
  factual-recall shaped; whether the module-level findings transfer to
  research-agent / coding-agent memory (this project's dominant interest) is
  argued by analogy, not measured.
- arXiv v1, not peer-reviewed.

## Trust signals

- **Credibility:** 3 — strong database-systems authorship (SJTU + Tsinghua,
  Guoliang Li a data-management name), broad controlled benchmark (12
  systems / 11 datasets) with released code; offset by preprint status (not
  peer-reviewed) and no independent reproduction yet.

## Follow-up

- **Relevance:** 4 — the four-module data-management decomposition and the
  cost/robustness findings materially strengthen
  [[concepts/agent-native-memory]] (memory as an agent-owned data system,
  with quantified evidence for the lifecycle-metadata and localized-
  maintenance guidance the concept already argues), and the maintenance /
  eviction taxonomy attests [[concepts/context-eviction-policy]] and
  [[concepts/multi-granularity-memory]]. The "graph handles updates, raw
  long-context still wins on temporal queries" result sharpens
  [[concepts/selective-memory-retrieval]]'s read-side policy question.
- The title poses the same "are we agent-native yet?" question this project
  answers structurally — a useful external audit of the pattern's maturity.
