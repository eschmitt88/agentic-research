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
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/kassis2026scientific]]"
  - "[[literature/papers/chen2026repo]]"
  - "[[literature/papers/singh2026churnbench]]"
  - "[[literature/papers/suresh2026grounding]]"
  - "[[literature/papers/wang2026research]]"
  - "[[literature/papers/khan2026surfacing]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/shared-substrate-contagion]]"
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

**A two-tier variant for a research group, with its alternatives measured
(2026-10-06).** [[literature/papers/wang2026research]] (AfS) is the nearest
published system to this repo's own layout. It has a single tracked
`MEMORY.md` per project worktree and a two-tier knowledge base. Its central
rule is that the organization tier "has no birth channel except promotion".
Promotion is a rewrite, not a move: project parameters become applicability
conditions, the card must be self-contained, and it must state the
mechanism. Three gates apply: the project is closed, the evidence is
frozen, and on the human lane the text contains no project deixis.
`dead_end` cards are called "the highest-yield" type.

Its maintenance rules map onto this page's failure half:

- Delivery is unconditional, with constant byte budgets, because "a channel
  that requires the model to ask is a channel it will sometimes not use".
- Retirement is driven by falsifiability, not age: dead tool references,
  applicability never hit, or contradiction within scope. It is proposed,
  never automatic.
- There is no log layer, because git, transcripts and ledgers already hold
  it.

Its internal audits of the alternatives bear on `raw/_candidates/` here. A
candidate queue awaiting curation left **47% of 800 candidates unprocessed**
(the oldest 49 days), and retirement was never invoked across 24 projects.
Guessing scope at write time left 224 organization claims with zero
promotion provenance.

Limits: these are uncontrolled internal numbers, and promotion's downstream
effect "has not been separated from a ceiling effect". The one replay also
promoted three cards "taken verbatim" from the source project, so the
rewriting step itself went unexercised.

## A recency-first reconciliation is the arm that lost: surfacing the conflict beat collapsing it, so a reconciliation pass should mark conflicts, not pick winners (2026-11-24)

The maintenance checklist above lists gap (2), automatic contradiction
reconciliation ("recency × authority × confidence; losing claims
archived, not deleted"), as missing here. The open question proposes
trialling a weekly reconciliation pass. [[literature/papers/khan2026surfacing]]
measures that design as its collapse arm, on knowledge-graph memory read
by a three-agent gpt-oss-120b debate. Nothing is deleted: supersession
sets an invalid-from time and both versions persist, like archiving the
loser. Competing claims on one key are ranked by (not invalidated,
newest, trust, number of writers), and the reader gets the top one.

- **Collapse handed the reader the lie.** With one planted same-key lie
  dated 30 days later, and conflicts reachable, collapse scored F1
  **0.116**. Showing the reader the whole ranked set with provenance
  scored **0.456**, and attacks fell **22 → 7 of 122**, at equal token
  cost. Archiving the loser preserved the audit trail but kept the
  loser out of the reader's view, and the reader is what got the
  answer wrong.
- **A heuristic supersession step made it worse.** A write-time gate
  that marks incumbents superseded fired 123 invalidations against 3
  rejections and cost −0.075 F1. A reconciliation pass that writes
  "superseded" from a recency or confidence heuristic is that step run
  in batch.
- **Bi-temporal records helped audit, not adjudication.** Recency in
  the ranking was the attack vector, and removing it from the ranking
  changed nothing once the conflict was surfaced. The bi-temporal facts
  in `obsidian-second-brain` above give reconciliation an audit trail.
  This paper gives no reason to let them choose the winner when a writer
  can set the date.

For the open question, the evidence-supported half of gap (2) is
*detection and marking*: find claims that genuinely conflict, and put
both, with provenance, in front of the reader. The paper's false-positive
controls are worth copying. Only single-valued relations can conflict,
judged on source data alone, and claims made at the same time are
elaborations, not disagreements. The hub's dated sections already do
the marking by hand: a later section names the earlier one it contradicts
and leaves both standing. A trial should test that, not a winner-picking
pass.

Limits before leaning on it:
- The reader is a gpt-oss debate over dialogue triples. A Claude
  session over Markdown concepts, whose conflicts are often scope
  differences rather than contradictions, is untested.
- The lie's displayed provenance is weaker by construction (one writer,
  lower trust), so the reader may have used those fields more than
  content.
- Collapse was always recency-first. A trust-first or support-first
  collapse was not run.
- The headline is from a retrieval cap chosen after the first run (+0.043
  at the pre-declared cap). One model family, workshop paper.

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
  literature. Two first data points (2026-09-15), both outside research
  corpora. [[literature/papers/singh2026churnbench]] measures the mechanism
  on the structured-data analog of this pattern: views staged once from
  live sources, with a scheduled refresh tier per entity class. Staleness
  was governed by **refresh-tier width × source mutation rate**, not by how
  long ago the compiled layer was built. Errors stayed flat at 7 / 4 / 4
  across 1 / 14 / 28-day builds and rose to 45 at 28 days only when refresh
  was disabled. Its tiers line up with the OKM taxonomy above: *live*
  entities are never staged (a pointer), and a 30-day *cold* tier behaved as
  timeless and was harmless because contracts barely change. It adds the
  rule the taxonomy lacks: assign a fact's class from its **measured**
  change rate, not from how important it feels. So compile-time curation
  breaks down where a compiled page's refresh interval overlaps its
  source's change rate, not merely where sources change. Every error that
  survived refresh was an aggregate over a faster-changing entity; the
  analogous pages here are MoCs and index lines, which summarize concepts
  that change on every ingest (an inference, not measured).
  [[literature/papers/suresh2026grounding]] adds the compile side from a
  drifting environment: a post-task compiled record index beats
  full-trajectory replay on pass rate at under a third of the input tokens
  (1.69M vs 5.42M on CLBench with a schema migration), and its staleness
  fix is a compiler with read-only access to the live source. The repair
  itself isn't isolated, since the compiler's lead predates the migration,
  so treat it as the mechanism to try, not evidence that it works.
