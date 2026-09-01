---
kind: concept
name: "llm-wiki-pattern"
status: seedling
added: "2026-07-14"
sources:
  - "[[literature/posts/gist-github-com-karpathy-llm-wiki]]"
  - "[[literature/papers/xu2025amem]]"
  - "[[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]]"
  - "[[literature/repos/eugeniughelbur-obsidian-second-brain]]"
  - "[[literature/repos/agricidaniel-claude-obsidian]]"
  - "[[literature/papers/cao2026agentsk1]]"
  - "[[literature/papers/tang2026wikiskill]]"
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

## Maintenance operations (the pattern's known failure half)

The rebuild critique
([[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]])
supplies the failure model: an **append-only** wiki goes internally
inconsistent past a few hundred sources. Its five gaps, as a checklist —
(1) rewrite-on-ingest with dated recency markers, (2) automatic
contradiction reconciliation (recency × authority × confidence; losing
claims archived, not deleted), (3) unsolicited synthesis passes,
(4) scheduled maintenance instead of on-demand, (5) AI-first note
format (the LLM, not the human, is the primary reader). A-Mem's memory
evolution is the academic echo of (1)–(2): new arrivals trigger LLM
updates to neighboring notes, and its ablation shows the win. Its
companion principle — **reversible automation** (daily diffs + a review
window before scheduled changes stick) — is what makes autonomous
maintenance safe; a git-tracked wiki gets it structurally.

This repo's scorecard: (4) and (5) enacted, (1) partial (concepts
evolve on ingest; literature notes are append-only), (2) and (3)
missing — no contradiction-reconciliation pass, and `/promote-moc` is
the only unsolicited synthesis.

**A second, independent implementation of the critique's checklist.**
The same author's `obsidian-second-brain` tool
([[literature/repos/eugeniughelbur-obsidian-second-brain]]) claims all
five gaps closed: `/obsidian-ingest` rewrites 5–15 existing pages per
source (1), `/obsidian-reconcile` runs on demand and nightly (2),
`/obsidian-synthesize` writes unprompted pattern pages (3), four
cron-scheduled agents plus a post-compaction background agent (4), and
an "AI-first" `## For future Claude` note preamble (5). It adds one
mechanism neither the critique nor A-Mem names: **OKM (Open Knowledge
Metabolism)** — a spec'd, linted rule that every stored fact must be
*timeless*, *dated*, or *a pointer*, so fast-changing facts are never
copied in to go stale, only linked with an `as of` stamp — plus
**bi-temporal facts** that record both when a claim was true and when
the vault learned it, giving reconciliation an audit trail. Still only
one team's self-report, not an ablation or an independent evaluation —
this is corroborating design convergence, not proof any given
mechanism works better than the alternatives.

**A third, larger implementation with a different emphasis.**
`claude-obsidian` ([[literature/repos/agricidaniel-claude-obsidian]],
9.4k stars, the highest-visibility repo in this cluster) converges on
the same three-layer shape but its distinctive contributions are
engineering concerns none of the other sources name: per-file advisory
locks for concurrent ingest sub-agents (`wiki-lock.sh`), a benchmarked
hybrid-retrieval pipeline (BM25 + optional Anthropic contextual-prefix
+ local cosine rerank, self-reported +32pp top-1 accuracy over a
BM25-only baseline), and a web-egress hygiene policy for its autonomous
research loop (URL scheme/host filtering, script stripping,
wikilink-injection defense, body-size caps) — a threat model (fetched
content injecting the LLM-owned wiki) none of the other three sources
raise explicitly.

**The pattern at industrial scale, off the markdown substrate.**
Agents-K1 ([[literature/papers/cao2026agentsk1]], Shanghai AI Lab) is
the largest instance found of compile-time knowledge orchestration:
2.46M scientific papers parsed **once, offline** (MinerU parser + a 4B
GRPO-trained extractor) into Scholar-KG, a persistent typed knowledge
graph that agents then query through an agent-facing interface
(GraphAnything CLI/MCP). The paper states the pattern's core move
explicitly — separating "offline construction of reliable knowledge
representations" from "online use of this knowledge for
evidence-grounded reasoning" — and quantifies the anti-RAG argument
the gist only asserts: the compiled graph beats nine graph-RAG
baselines on multi-hop QA and lifts GPT-5.2 from 25.2% to 39.4% on
FrontierScience-Research. The three layers map cleanly (raw PDFs →
LLM-maintained graph → five-module schema), as do the three operations
(ingest = parser + extractor; query = tri-source CLI; lint = a
per-module LLM-as-judge protocol). The divergence is the substrate:
Neo4j + embeddings rather than markdown the agent owns — evidence that
the pattern's economic core (organization work amortized at compile
time) is substrate-independent, while what survives from the wiki
form is provenance-per-field and typed links, not human readability.

## Open questions

- **Which maintenance operations are essential vs nice-to-have.** The
  critique treats all five as required for production; (2)
  reconciliation now has two independent implementations pointing the
  same direction (A-Mem's evolution ablation, and this repo's
  `/obsidian-reconcile` + bi-temporal audit trail) but still no
  controlled comparison. A weekly reconciliation pass is the obvious
  candidate mechanism to trial here before any `/elevate` proposal —
  and `freshness-policy.md`'s timeless/dated/pointer taxonomy is a
  concrete design worth stealing for that trial rather than inventing
  one from scratch.
- **AI-first vs human-readable tension.** Gap 5 inverts
  [[concepts/agent-native-memory]]'s "human-readable artifacts"
  framing; in practice the formats converge (frontmatter + wikilinks +
  prose), but the *primary-reader* question changes what a lint should
  optimize for.
- **Where compile-time curation breaks down** — for corpora that
  change under you (live codebases, prices), compiled pages go stale;
  the pattern implicitly assumes slowly-accreting sources like
  literature.
