---
kind: repo
name: "claude-obsidian"
url: https://github.com/AgriciDaniel/claude-obsidian
commit: HEAD @ 2026-05-28
source: "raw/repos/agricidaniel-claude-obsidian.md"
added: "2026-07-15"
relevance: 3
status: scanned
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
tags: [second-brain, pkm, obsidian, claude-code, hybrid-retrieval, multi-writer-locking, web-egress-hygiene]
---

# claude-obsidian

## Purpose

The highest-visibility (9,400 stars) open-source implementation of
Karpathy's LLM Wiki pattern for Obsidian + Claude Code — explicitly
positioned in its own README as "Based on Karpathy's LLM Wiki pattern."
Same shape as [[literature/repos/eugeniughelbur-obsidian-second-brain]]
(drop sources → Claude extracts entities/concepts → structured vault
→ lint for orphans/dead-links/staleness), but with a different set of
distinctive engineering choices worth recording independently: explicit
multi-writer concurrency control, a benchmarked hybrid-retrieval
pipeline, and a hardened autonomous web-research loop.

## Shape

- `.raw/` — immutable source landing zone (same convention as this
  project's `raw/`).
- `wiki/` — the LLM-maintained layer; structure varies by "Methodology
  Mode" (LYT / PARA / Zettelkasten / Generic — configurable rather than
  fixed, unlike this project's literature/concepts/mocs split).
- `hot.md` — a session-boundary context cache, read first on every
  query (`hot → index → pages`) to bound token cost.
- `scripts/wiki-lock.sh` — per-file advisory locks for concurrent
  ingest sub-agents.
- `skills/wiki-retrieve/` — opt-in hybrid retrieval (BM25 + optional
  Anthropic contextual-prefix + local-ollama cosine rerank).
- `skills/autoresearch/` — a 3-round web-research loop (broad search →
  gap-fill → synthesis) with a documented egress-hygiene policy.
- Companion repo `claude-canvas` for visual-canvas orchestration.

## Useful bits

- **Multi-writer safety (v1.7+).** When a user batches multiple
  sources for parallel ingestion, sub-agents can target the same wiki
  page. `wiki-lock.sh` gives per-file advisory locks — one writer
  acquires, others wait and retry next pass — and the PostToolUse
  auto-commit hook checks the lock list before staging, deferring the
  commit while writes are in flight. This is a concrete answer to a
  problem this project doesn't yet face (single-agent, serial
  `/ingest`) but would need the moment `/curate` or `/digest` fanned
  ingestion out to parallel subagents.
- **Hybrid retrieval with a reported benchmark.** BM25 sparse search
  is always-on; a consent-gated (`--allow-egress`) contextual-prefix
  tier sends page bodies to the Anthropic API per [Anthropic's
  contextual retrieval technique](https://www.anthropic.com/news/contextual-retrieval);
  cosine rerank runs on a local Ollama model by default; results are
  `--explain`-traceable. The README reports a 50-query internal
  benchmark: **+32 percentage points top-1 accuracy, +41% error
  reduction** vs. the prior (BM25-only) version. Self-reported, not
  independently reproduced — but it's a rare *quantified* claim in a
  space (PKM tooling) that mostly ships qualitative demos.
- **Web egress hygiene for the autonomous research loop.** The
  `/autoresearch` skill's fetch layer rejects `file://` / `javascript:`
  / RFC1918-host URLs, strips `<script>` tags and wikilink-injection
  attempts from fetched content, and caps fetch bodies at 50KB — a
  concrete threat model (prompt/wikilink injection via fetched web
  content into an LLM-owned, auto-committed wiki) that neither the
  gist, the critique, nor A-Mem name explicitly.

## Follow-up

**Relevance: 3** — solid prior art on the active `llm-wiki-pattern`
theme (a second, larger, independently-built implementation, useful as
a design-choice comparison point against
[[literature/repos/eugeniughelbur-obsidian-second-brain]]), but its
strongest ideas (locking, hybrid retrieval, egress hygiene) are
adjacent engineering concerns rather than direct evidence for any of
this graph's existing memory-architecture claims, so it doesn't shift a
concept the way A-Mem or the rebuild critique did.

**Credibility: 2** — MIT-licensed, real star count and usage evidence,
concrete version history (v1.0 → v1.8.2) suggesting sustained
iteration. Weighed down for this repo specifically: the README
advertises a paid "AI Marketing Hub Pro" membership tier with an
early-access private mirror, which is a promotional signal layered
on top of the open-source release — worth knowing when citing this as
evidence rather than treating it as a neutral engineering artifact.
The retrieval benchmark (50 queries, no methodology detail, no
baseline release) is a self-report, not a verified result.

- Not explored: no local install/run performed.
- Possible future pull: if this project ever parallelizes `/ingest` or
  `/curate` across subagents, `wiki-lock.sh`'s advisory-lock + deferred-
  commit pattern is a concrete mechanism to borrow rather than design
  from scratch.
