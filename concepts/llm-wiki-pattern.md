---
kind: concept
name: "llm-wiki-pattern"
status: seedling
added: "2026-07-14"
sources:
  - "[[literature/posts/gist-github-com-karpathy-llm-wiki]]"
  - "[[literature/papers/xu2025amem]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/web-grounded-literature]]"
related_experiments: []
tags: [second-brain, pkm, knowledge-organization, compile-time-curation, markdown, zettelkasten]
---

# llm-wiki-pattern

## Definition

Do knowledge organization **once, at ingest/compile time** — an LLM
distills immutable raw sources into a persistent, cross-referenced
markdown wiki it owns, governed by a schema file — instead of re-sorting
raw data at every query the way RAG does. Three layers: raw sources
(immutable), wiki (LLM-maintained, compounding), schema
(conventions/workflows); three operations: ingest, query, lint.

## Why it matters here

This project *is* an instance of the pattern and converged on it
independently: `raw/` → `literature/`+`concepts/`+`mocs/` →
`CLAUDE.md`+templates, with `/ingest`, graph queries, and `/lint` as the
three operations and `_meta/index.md`/`_meta/log.md` as the key files.
Naming it does three things:

1. **Grounds the design choice for downstream projects** — the pattern
   is now citable (Karpathy gist, 5k+ stars, plus the whole
   Claude-Code-and-Obsidian practitioner ecosystem it spawned) rather
   than embodied anonymously.
2. **Imports the ecosystem's failure reports.** Production comments on
   the gist converge on: cross-reference drift as the dominant failure
   mode (lint is load-bearing, not optional), schema files as the
   consistency mechanism, and hybrid search (FTS + embeddings) beating
   flat indexes past ~4,000 concepts.
3. **Connects PKM practice to the memory literature.** A-Mem
   ([[literature/papers/xu2025amem]], NeurIPS 2025) is the peer-reviewed
   bridge: Zettelkasten-style atomic notes + LLM link generation +
   retroactive evolution, with ablations showing the *links* carry the
   multi-hop value — quantitative backing for the wiki's cross-reference
   discipline.

The distinctive claim vs [[concepts/agent-native-memory]] (substrate:
markdown the agent owns, ops as tools) is **when the organization work
happens**: compile-time curation producing a compounding artifact, vs
per-query retrieval over unorganized data. The anti-RAG argument is the
pattern's economic core — sorting work amortizes across all future
queries.

## Connections

- **[[concepts/agent-native-memory]]** — the substrate this pattern
  runs on; llm-wiki-pattern adds the compile-vs-query-time framing and
  the three-layer separation.
- **[[concepts/multi-granularity-memory]]** — a compiled wiki is the
  coarse grain(s) constructed at write time; raw sources remain the
  fine grain.
- **[[concepts/selective-memory-retrieval]]** — read-side policy over
  the compiled artifact; the gist's scale reports (hybrid search past
  4k pages) are a retrieval-policy datapoint.
- **[[concepts/web-grounded-literature]]** — ingest is the boundary
  where fresh web material enters the wiki.

## Open questions

- **Append-only vs self-rewriting.** The gist's pattern is largely
  append-and-link; the practitioner critique (theaioperator rebuild —
  pending ingest) argues append-only wikis go internally inconsistent
  past a few hundred sources without rewriting, contradiction
  reconciliation, scheduled maintenance, and unsolicited synthesis.
  A-Mem's memory evolution is the academic version of the same fix.
  Which maintenance operations are *essential* vs nice-to-have is the
  open design question — this repo runs scheduled lint/curate but has
  no contradiction-reconciliation pass.
- **Where compile-time curation breaks down** — for corpora that
  change under you (live codebases, prices), compiled pages go stale;
  the pattern implicitly assumes slowly-accreting sources like
  literature.
