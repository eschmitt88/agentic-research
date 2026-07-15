---
kind: candidates
topic: "agentic second brains — LLM-agent-managed personal knowledge bases (Claude Code + markdown vaults, Obsidian-style PKM with agents, Zettelkasten/MoC practice as agent memory architecture)"
discovered: 2026-07-14
source: discover
n_requested: 10
n_returned: 8
curated: 2026-07-15
---

# Triage: agentic second brains / LLM-managed PKM

Context for the curator: the graph has zero coverage of second-brain /
PKM / Zettelkasten material despite this project *being* an instance of
the pattern — `raw/` (immutable sources) + `concepts/`+`mocs/`
(LLM-maintained wiki layer) + `CLAUDE.md` (schema file) is exactly
Karpathy's three-layer LLM Wiki. Two distinct clusters below: an
academic one (Zettelkasten as agent memory architecture) that plugs
straight into the existing memory concepts, and a practitioner one
(Claude Code + Obsidian vaults) that attests our own architecture from
an independent domain.

## 1. A-MEM: Agentic Memory for LLM Agents

- url: https://arxiv.org/abs/2502.12110
- type: paper
- summary: NeurIPS 2025 paper (code released) that builds agent memory
  explicitly on Zettelkasten principles — atomic notes with generated
  context/keywords/tags, dynamic linking to prior memories, and memory
  *evolution* where new notes trigger updates to old ones; beats SOTA
  baselines across six foundation models.
- reason: The canonical peer-reviewed bridge between PKM practice and
  agent memory architecture — direct new attestation material for
  [[agent-native-memory]], [[multi-granularity-memory]], and
  [[verified-memory-writes]], and the strongest academic anchor for any
  second-brain concept we seed.

## 2. Karpathy's "LLM Wiki" gist

- url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- type: post
- summary: The primary source (April 2026) for the pattern the whole
  practitioner ecosystem builds on: immutable raw sources, an
  LLM-owned markdown wiki layer compiled from them, and a schema file
  (CLAUDE.md/AGENTS.md) governing conventions — framed as doing RAG's
  sorting work once at compile time instead of per-query.
- reason: This repo independently converged on the same three-layer
  architecture; ingesting the primary source lets the graph name and
  cite the pattern instead of embodying it anonymously.

## 3. "I rebuilt Karpathy's LLM Wiki: what's missing"

- url: https://theaioperator.io/p/i-rebuilt-karpathys-llm-wiki-heres
- type: post
- summary: Critical rebuild identifying five gaps in the append-only
  pattern — no rewriting of stale claims, unresolved contradictions at
  scale, no unsolicited synthesis, no scheduled maintenance, and
  human-first notes when "you don't read your own notes, the LLM does"
  (the AI-First Vault Principle).
- reason: The dissenting viewpoint in the cluster, and each named gap
  maps onto a mechanism this project already runs (nightly /curate,
  /lint, /promote-moc) — useful both as validation and as a checklist
  of what we still lack (contradiction reconciliation, unsolicited
  synthesis).

## 4. eugeniughelbur/obsidian-second-brain

- url: https://github.com/eugeniughelbur/obsidian-second-brain
- type: repo
- summary: Cross-CLI skill (Claude Code, Codex, Gemini, OpenCode…)
  turning an Obsidian vault into an "AI-first second brain": 44
  commands, self-rewriting notes, local+hybrid semantic search, and
  scheduled agents that maintain the vault unattended.
- reason: The most developed open implementation of scheduled
  autonomous vault maintenance — directly comparable to this repo's
  cron sweeps and a source of concrete mechanisms for
  [[skill-library-lifecycle]].

## 5. AgriciDaniel/claude-obsidian

- url: https://github.com/AgriciDaniel/claude-obsidian
- type: repo
- summary: Self-organizing second brain for Obsidian + Claude Code
  (v1.9.x): drop any source and the agent reads, links, and files it
  into a connected plain-markdown knowledge graph, with methodology
  modes and audit hardening.
- reason: The highest-visibility Karpathy-pattern implementation;
  its ingest-and-link loop is an independent reinvention of this
  repo's /fetch-paper → /ingest → concept-linking pipeline.

## 6. Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

- url: https://arxiv.org/abs/2504.19413
- type: paper
- summary: Widely-deployed open-source memory layer that incrementally
  extracts, consolidates, and updates memory facts via LLM
  summarization, with a graph-memory variant; reports large latency
  and token savings over full-context baselines.
- reason: The production-scale counterpoint to A-MEM's
  Zettelkasten approach — useful attestation for
  [[selective-memory-retrieval]] and for the zhou2026ready lesson that
  memory structure should match workload.

## 7. "How I Took Karpathy's LLM Wiki and Built an AI-Powered Second Brain in Obsidian"

- url: https://aimaker.substack.com/p/llm-wiki-obsidian-knowledge-base-andrej-karphaty
- type: post
- summary: Practitioner walkthrough of implementing the LLM Wiki
  pattern in a real Obsidian vault — compile step, entity/concept
  pages, backlink maintenance, and where the pattern strains in
  day-to-day use.
- reason: Ground-level attestation of the pattern's mechanics and
  failure points from someone who ran it, complementing the gist
  (design) and the rebuild critique (gaps).

## 8. Wes Roth — "Claude Built the Ultimate Second Brain"

- url: https://www.youtube.com/watch?v=cwf2vEAigKA
- type: talk
- summary: Survey video of the Claude-managed second-brain ecosystem
  (LLM Wiki pattern, Obsidian + Claude Code workflows) that is driving
  the current wave of practitioner interest.
- reason: The item that triggered this sweep; low evidence weight on
  its own but useful as the ecosystem-survey entry point and to record
  why this topic entered the graph now.

---

## Curation progress (2026-07-14, same-day user-directed ingest)

- **Ingested #1 A-MEM** → `literature/papers/xu2025amem.md` (rel 4 / cred 5)
- **Ingested #2 Karpathy gist** → `literature/posts/gist-github-com-karpathy-llm-wiki.md` (rel 4); seeded `concepts/llm-wiki-pattern`
- **Ingested #3 rebuild critique** → `literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki.md` (rel 4)
- Entries 4–8 remain for the nightly `/curate` sweep.

## Curation

Dedup check against `literature/**` `source:`/`url:` fields: none of
entries 4–8 were already in the graph (entry 3's URL was the only hit,
already recorded above as ingested). Coordinator headroom verdict at
curation time: `go` / `high` (7.6% weekly spend, resources idle) —
proceeded autonomously per `agency: max`.

- **Ingested #4 eugeniughelbur/obsidian-second-brain** →
  `literature/repos/eugeniughelbur-obsidian-second-brain.md` (rel 4 /
  cred 3); updated `concepts/llm-wiki-pattern` — this is the same
  author's tool implementing all five gaps the rebuild critique (#3)
  named as missing, plus a named OKM freshness policy and bi-temporal
  facts not covered by any prior source.
- **Ingested #5 AgriciDaniel/claude-obsidian** →
  `literature/repos/agricidaniel-claude-obsidian.md` (rel 3 / cred 2);
  updated `concepts/llm-wiki-pattern` — highest-visibility (9.4k star)
  implementation, distinctive for multi-writer advisory locking, a
  benchmarked hybrid-retrieval pipeline, and a web-egress hygiene
  policy none of the other sources name. Credibility capped by a
  promotional "Pro membership" upsell layered on the open-source
  release and an unverified self-reported benchmark.
- **Ingested #6 Mem0 (chhikara2025mem0)** →
  `literature/papers/chhikara2025mem0.md` (rel 4 / cred 4); updated
  `concepts/agent-native-memory` and `concepts/verified-memory-writes`
  — production-scale (60.8k star, deployed) counterpoint to A-Mem,
  with an explicit ADD/UPDATE/DELETE/NOOP write-gate that corroborates
  (but doesn't replicate) TrustMem's write-verification claim.
- **Declined #7 aimaker.substack.com walkthrough** — redundant with
  already-ingested primary source (#2, the gist) and critique (#3);
  a single practitioner blog post with no unique mechanism, benchmark,
  or claim beyond what's now covered by six substantive sources in
  this same batch. Not legally fraught or off-mission, just thin
  relative to what's already in the graph.
- **Declined #8 Wes Roth — "Claude Built the Ultimate Second Brain"**
  — ecosystem-survey video, no code or novel claims; the original
  triage note already flagged it as "low evidence weight on its own,"
  and its role (recording why this sweep was triggered) is now
  satisfied by this candidate file itself. Superseded by the six
  substantive sources ingested above.
