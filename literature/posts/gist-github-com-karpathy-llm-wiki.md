---
kind: post
title: "LLM Wiki: A Pattern for Personal Knowledge Bases"
author: Andrej Karpathy
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
source: "raw/web/gist-github-com-karpathy-llm-wiki.md"
added: "2026-07-14"
relevance: 4
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/agent-native-memory]]"
tags: [second-brain, pkm, llm-wiki, knowledge-organization, markdown, compile-time-curation]
---

# LLM Wiki: A Pattern for Personal Knowledge Bases

## TL;DR

Karpathy's gist (2026-04-04, 5,000+ stars — the primary source behind
the current second-brain wave) names the pattern where an LLM builds and
maintains a persistent markdown wiki from immutable raw sources under a
schema file's conventions: do RAG's sorting work **once, at compile
time**, and let the wiki compound instead of re-deriving per query.

## Key points

- **Three layers**: immutable raw sources → an LLM-owned wiki layer
  (summaries, entity/concept pages, cross-references) → a schema file
  (CLAUDE.md/AGENTS.md) holding conventions and workflows. This project
  independently converged on exactly this: `raw/` → `literature/` +
  `concepts/` + `mocs/` → `CLAUDE.md` + `_meta/templates/`.
- **Three operations**: ingest (process a source, update related pages,
  keep cross-references), query (synthesize with citations; good answers
  can become pages), lint (contradictions, stale claims, orphans, dead
  links). Again 1:1 with `/ingest`, graph queries, and `/lint`.
- **Anti-RAG framing**: retrieval-augmented generation re-sorts raw data
  on every question; the wiki is "a persistent, compounding artifact"
  that fits a long context window once compiled.
- **Key files**: a content catalog (`index.md`) and an append-only
  greppable log (`log.md`) — this repo's `_meta/index.md` and
  `_meta/log.md`.
- **Production reports from the comments**: schema files are what keep
  consistency; cross-reference drift is the dominant failure mode (lint
  is essential); at ~4,000 concepts hybrid search (SQLite FTS5 +
  embeddings) beats flat indexes.

## Follow-up

- **Relevance:** 4 — primary source for [[concepts/llm-wiki-pattern]]
  (seeded from this note); the graph previously embodied the pattern
  anonymously and can now name and cite it. Not a 5 because it's a
  practitioner gist, not evidence — the evidence lives in the academic
  bridge ([[literature/papers/xu2025amem]]) and the critique
  (theaioperator rebuild, next in the ingest queue).
- The ~4,000-concept hybrid-search comment is a concrete scale threshold
  to remember if this graph (23 concepts) ever grows two orders of
  magnitude.
