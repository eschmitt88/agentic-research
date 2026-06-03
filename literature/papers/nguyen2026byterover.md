---
kind: paper
title: "ByteRover: Agent-Native Memory Through LLM-Curated Hierarchical Context"
authors:
  - Andy Nguyen
  - Danh Doan
  - Hoang Pham
  - Bao Ha
  - Dat Pham
  - Linh Nguyen
  - Hieu Nguyen
  - Thien Nguyen
  - Cuong Do
  - Phat Nguyen
  - Toan Nguyen
institutions: ["ByteRover"]
year: 2026
venue: "arXiv:2604.01599 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.01599"
code_url:
citations:
source: "raw/papers/nguyen2026byterover.pdf"
added: "2026-04-26"
relevance: 5
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/citation-anchoring]]"
tags:
  - agent-native-memory
  - memory-augmented-generation
  - context-tree
  - hierarchical-knowledge
  - markdown-knowledge-graph
  - progressive-retrieval
  - lifecycle-management
---

# ByteRover: Agent-Native Memory Through LLM-Curated Hierarchical Context

## TL;DR

ByteRover inverts the conventional Memory-Augmented Generation (MAG)
pattern: instead of memory being an external service the agent calls
into (vector DB, graph DB, embedding pipeline), memory operations
(`ADD`, `UPDATE`, `UPSERT`, `MERGE`, `DELETE`) are first-class tools
in the same LLM's reasoning loop. Knowledge lives as plain markdown
files on the local filesystem in a hierarchical *Context Tree*
(`Domain > Topic > Subtopic > Entry`) with explicit `@`-relation
edges, provenance, and an Adaptive Knowledge Lifecycle (importance
score + maturity tiers + recency decay). A 5-tier progressive
retrieval strategy resolves most queries sub-100ms without an LLM
call; the system reaches 96.1% on LoCoMo and 92.8% on LongMemEval-S
with **zero external infrastructure**.

## Claims

- The dominant failure mode of external-service MAG is *the system
  that stores knowledge does not understand it*. Chunking is
  mechanical, embeddings encode surface similarity, and the agent
  has no visibility into how its memory was organized — leading to
  **semantic drift**, **lost coordination context** (multi-agent),
  and **recovery fragility** after crashes.
- Making the curating LLM and the reasoning LLM *the same agent*
  eliminates the drift: the agent that intended a nuance is the
  agent that wrote the markdown that captured it.
- Memory ops returning per-operation status (success / failure /
  message) enables a *stateful feedback loop* — the agent adapts
  in real time. External HTTP services don't carry this.
- A 5-tier retrieval cascade (cache → fuzzy cache → BM25 → optimized
  LLM call → full agentic loop) is essential — not merely a latency
  optimization. Removing it costs **−29.4 pp** absolute accuracy on
  LongMemEval-S.
- File-based markdown with explicit `@relation` edges and BM25 full-
  text search beats vector / graph databases on long-term
  conversational benchmarks while requiring zero external infra.

## Methods

- **Three layers**: (1) Agent Layer — the LLM with `curate` and
  `search_knowledge` as native tools; (2) Execution Layer —
  sequential task queue with sandboxed CurateExecutor (write side)
  and QueryExecutor (read side, 5-tier); (3) Knowledge Layer —
  Context Tree of markdown files + MiniSearch BM25 index + query
  cache. All on local FS.
- **Context Tree entry** is a markdown file with five components:
  Relations (explicit `@domain/topic/file.md` edges), raw Concept
  (provenance: task, changes, sources, timestamp, author), Vault
  (interpreted narrative), Snippets (code, data), Lifecycle metadata.
- **Symbol tree** with five kinds (Domain / Topic / Subtopic /
  Context / Summary) for O(1) path lookup; bidirectional reference
  index (forward + backlinks).
- **Adaptive Knowledge Lifecycle**: importance score
  ι ∈ [0, 100] (access +3, update +5, daily decay 0.995); maturity
  tiers `draft → validated → core` with hysteresis gaps (65/35,
  85/60); recency decay `r = exp(−Δt/τ)` with τ = 30 days.
  Compound retrieval score combines BM25 + normalized importance +
  recency.
- **Five atomic curate operations** (`ADD`, `UPDATE`, `UPSERT`,
  `MERGE`, `DELETE`) each carrying a `reason` field as audit trail.
- **Curation pipeline**: preprocess (read + validate) → pre-compaction
  (escalating L1/L2/L3 summarization with deterministic truncation
  fallback so curation always terminates) → sandboxed LLM curation.
  Atomic write-to-temp-then-rename for crash safety.
- **5-tier retrieval** (Tier 0: exact cache hash; Tier 1: Jaccard
  fuzzy cache; Tier 2: high-confidence BM25; Tier 3: pre-fetched
  context + single optimized LLM call; Tier 4: full agentic loop).
  Tiers 0–2 issue zero LLM calls.
- **Out-of-domain detection**: when significant query terms (length
  ≥ 4) match nothing and normalized BM25 falls below θ_OOD = 0.85,
  system explicitly rejects rather than hallucinating.
- Exposed as MCP tools `brv-query` and `brv-curate`.

## Results

- **LoCoMo overall**: 96.1% (next-best HonCho 89.9%, +6.2 pp).
  Multi-hop +9.3 pp over best, temporal 97.8%. Beaten only on
  open-domain (85.9% vs Hindsight 95.1%).
- **LongMemEval-S overall**: 92.8% (best in table; Chronos 92.6%,
  Hindsight 91.4%). Strongest on knowledge-update (98.7%) and
  single-session preference (96.7%); weakest on multi-session
  (84.2%) where Chronos's event-ordering wins.
- **Latency**: median cold-query 1.2s on 272-doc corpus, 1.6s on
  23,867-doc corpus. p99 stays under 2.5s — tiered retrieval bounds
  search cost as corpus grows.
- **Ablations on LongMemEval-S**:
  - w/o tiered retrieval: **−29.4 pp** (largest drop by far);
    multi-session 84.2 → 47.4. Tiered retrieval is the load-bearing
    component, not just a latency knob.
  - w/o OOD detection: −0.4 pp; mostly affects temporal queries.
  - w/o relation graph: −0.4 pp on this benchmark; expected to
    matter more on explicit multi-hop tests.

## Critique / open questions

- The write path is LLM-driven and therefore expensive — the
  paper itself flags this as a limitation. For research-knowledge
  curation (low write rate) this is fine; for high-throughput
  ingestion it isn't.
- The relation-graph ablation showed only −0.4 pp on LongMemEval-S,
  raising the question of how much the explicit edges actually
  contribute vs. the BM25 + cache machinery doing the real work.
  Authors note this likely flips on benchmarks with deeper multi-hop
  demands; not directly tested here.
- AKL parameters (importance increments, decay τ = 30, hysteresis
  gaps) are presented as design choices without sensitivity analysis.
  Whether the specific numbers matter or just any reasonable
  hysteresis-with-decay scheme works is unanswered.
- Conversational-memory benchmarks (LoCoMo, LongMemEval) test long
  dialogue retention, not the kind of cross-session research-graph
  use case this project actually has — generalization to durable
  research notes is plausible but extrapolated.

## Trust signals

- **Credibility:** 2 — ByteRover, a single commercial startup
  (byterover.dev); arXiv preprint, not peer-reviewed. Strong SOTA
  claims on LoCoMo/LongMemEval but no open code located (the system
  is a commercial product exposed via MCP tools), so results are not
  independently reproducible. Vendor-paper framing warrants caution.

## Follow-up

- **This project is structurally a ByteRover instance.** The
  Context Tree's `Domain > Topic > Subtopic > Entry` layout maps
  closely onto our `concepts/ + literature/<kind>/ + experiments/`
  shape; `@relation` edges are functionally identical to our
  `[[wikilink]]` cross-references and `@import` directives;
  status tiers (`seedling | growing | mature` and `experimental |
  active | retired`) are a coarse-grained AKL. Worth writing this
  out explicitly so the parallel is visible to downstream projects.
- The −29.4 pp tiered-retrieval ablation is the cleanest published
  evidence that *cached / cheap retrieval before agent loops* is
  load-bearing, not just an optimization. Cite this when justifying
  why `/lint` and `/digest` should pre-filter aggressively before
  any LLM-driven step.
- Compare AKL's `importance + recency + maturity` formula to our
  current implicit lifecycle (`status:` field + manual review). A
  downstream project that wants quantitative graduation criteria
  could lift the AKL math directly.
- The OOD-detection idea (refuse rather than partial-match) is a
  good pattern for `/discover` and `/digest` — when no candidate
  matches the active concepts well, return zero rather than the
  least-bad fit.
