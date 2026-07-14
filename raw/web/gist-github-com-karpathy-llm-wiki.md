# LLM Wiki: A Pattern for Personal Knowledge Bases

- source_url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- fetched: 2026-07-14
- author: Andrej Karpathy (@karpathy)
- gist_file: llm-wiki.md
- created: 2026-04-04
- engagement: 5,000+ stars/forks at fetch time
- capture_note: retrieved via WebFetch (summarizing fetcher); structure
  and key wording preserved, not a verbatim byte copy.

## Core Concept

The document describes a pattern where LLMs incrementally build and
maintain a persistent wiki — a structured markdown knowledge base —
rather than repeatedly retrieving and synthesizing from raw sources on
every query. The key insight: "the wiki is a persistent, compounding
artifact."

## Three Layers

1. **Raw Sources** (immutable): Articles, papers, documents — the
   source of truth. The LLM reads them but never edits them.
2. **The Wiki** (LLM-maintained): Markdown files with summaries, entity
   pages, concept pages, cross-references. The model owns this layer
   entirely.
3. **The Schema** (configuration): A file like CLAUDE.md that defines
   wiki structure, conventions, and workflows. Evolves with use.

## Operations

- **Ingest**: Process new sources, extract key information, update
  related pages, maintain cross-references.
- **Query**: Search the wiki, synthesize answers with citations;
  valuable findings can become new pages.
- **Lint**: Periodically check for contradictions, stale claims, orphan
  pages, missing cross-references.

## Key Files

- **index.md**: Content-oriented catalog (links, summaries, metadata).
- **log.md**: Append-only chronological record with greppable
  timestamps.

## Notable Insights from Comments

Production implementations report:

- Schema files become critical for maintaining consistency.
- Drift (outdated cross-references) is the main failure mode; lint
  passes are essential.
- At scale (~4,000 concepts), hybrid search (SQLite FTS5 + embeddings)
  outperforms flat indexes.
- The compounding effect enables synthesis impossible from raw sources
  alone.

Multiple developers have instantiated versions for research, book
reading, team knowledge management, and specialized domains
(accounting, behavioral verification, product management).
