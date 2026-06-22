---
kind: concept
name: "agent-native-memory"
status: experimental
added: "2026-04-26"
source_papers:
  - nguyen2026byterover
  - zhao2026expweaver
  - ouyang2026skillos
  - cho2026skillret
  - yang2026graph
  - wu2026gam
  - du2026memory
sources:
  - "[[literature/papers/lam2026governing]]"
  - "[[literature/papers/pham2026memorai]]"
  - "[[literature/papers/sun2026rethinking]]"
  - "[[literature/papers/wang2026parness]]"
  - "[[literature/papers/wu2026memory]]"
  - "[[literature/papers/yu2026hmem]]"
  - "[[literature/papers/nguyen2026byterover]]"
  - "[[literature/papers/zhao2026expweaver]]"
  - "[[literature/papers/ouyang2026skillos]]"
  - "[[literature/papers/cho2026skillret]]"
  - "[[literature/papers/yang2026graph]]"
  - "[[literature/papers/wu2026gam]]"
  - "[[literature/papers/du2026memory]]"
  - "[[literature/papers/du2026mlevolve]]"
  - "[[literature/papers/omri2026agent]]"
  - "[[literature/papers/xu2026evoarena]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/posts/paddo-dev-claude-code-leak-harness-exposed]]"
used_by: []
related_concepts:
  - "[[concepts/structured-world-model]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/context-eviction-policy]]"
related_experiments: []
tags: [memory, knowledge-graph, agent-architecture, hierarchical, retrieval, lifecycle]
---

# agent-native-memory

## Definition

A memory architecture in which the same LLM that reasons over a task
also curates and retrieves its knowledge — memory operations are
*tools in the agent's toolkit*, not API calls to an external service
(vector DB, graph DB, embedding pipeline). Knowledge is stored as
human-readable artifacts (typically markdown files) the agent writes
and reads directly, with explicit provenance, cross-references, and
lifecycle metadata.

## Why it matters

ByteRover ([[literature/papers/nguyen2026byterover]]) reports that
making memory agent-native eliminates three failure modes that
plague external-service Memory-Augmented Generation: **semantic
drift** (the chunking/embedding pipeline does not understand what
the agent meant), **lost coordination context** (multi-agent setups
share data but not the *why*), and **recovery fragility** (state
must be reconstructed from query results rather than read from the
filesystem). On long-term conversational benchmarks, ByteRover
reaches 96.1% on LoCoMo and 92.8% on LongMemEval-S — competitive
or state-of-the-art — while requiring **zero external infrastructure**
(no vector DB, no graph DB, no embedding service).

This project is already structurally an agent-native-memory system:
literature notes, concepts, and experiments are markdown files the
ingest agent reads and writes; cross-references are explicit
wikilinks rather than embedding similarity. Naming the pattern
makes the design choice visible to downstream projects that may
otherwise default to bolting on a vector store.

## Implementation guidance

1. **Hierarchical layout, addressable paths.** ByteRover uses
   `Domain > Topic > Subtopic > Entry`. This project uses
   `concepts/ + literature/<kind>/ + experiments/<slug>/` with a
   parallel `_meta/` for indices. The point is: every entry has a
   stable path the agent can name and link to, not an opaque ID
   inside a database.

2. **Explicit relation edges, not implicit similarity.** Each entry
   declares its connections (`[[wikilink]]`, `@import`,
   `related_concepts:`, `source_papers:`). Authors decide what
   relates to what; embedding similarity does not. ByteRover's
   ablation shows relation graphs matter more on explicit multi-hop
   queries than on flat-fact lookups — this is consistent with this
   project's use, where the graph is for cross-paper synthesis.

3. **Provenance per entry.** Frontmatter (`source:`, `source_papers:`,
   `added:`) records where the knowledge came from. This is how
   downstream agents can audit a claim back to its origin and how
   `/lint` catches concepts with no `sources:`.

4. **Memory ops as tools, not services.** `/ingest`, `/sync-imports`,
   `/lint` are agent-callable skills that read and write the
   knowledge graph in-process. There is no external memory service
   to query — the filesystem *is* the memory. This means:
   - Per-operation feedback (what wrote, what failed, why) is
     immediate; the agent adapts in the same turn.
   - Crash recovery is trivial — state is in the files.
   - Knowledge is portable: a `git clone` is the entire memory.

5. **Lifecycle metadata on every entry.** The `status:` field
   (`seedling | growing | mature` for concepts;
   `active | experimental | retired` for the
   import-contract status; `running | done | abandoned` for
   experiments) is a coarse-grained Adaptive Knowledge Lifecycle.
   ByteRover formalizes this with importance scoring, hysteresis-
   bound maturity tiers, and recency decay; we have the pattern
   without the math. A downstream project that wants quantitative
   graduation criteria could lift the AKL formulas:
   `ι ∈ [0,100]` with access bonus + update bonus + daily decay,
   `recency = exp(-Δt/τ)`, hysteresis gaps to prevent oscillation.

6. **Tiered retrieval before agent loops.** ByteRover's largest
   single ablation (−29.4 pp absolute) was removing the cache /
   BM25 cascade and routing every query to the agentic loop. The
   takeaway transfers: pre-filter cheaply (grep, frontmatter scan,
   path-based lookup) before invoking any LLM-driven skill. `/lint`
   and `/digest` already follow this pattern; preserve it.

7. **Refuse rather than partially match.** When a query has no
   strong match in the corpus, signal "no relevant knowledge"
   rather than returning the least-bad fit. ByteRover's
   out-of-domain detection prevents hallucinated answers from
   tangential retrievals — same principle should apply to `/discover`
   and `/digest` candidate selection.

8. **Memory as a git-style patch history, not a latest-state snapshot.**
   EvoMem ([[literature/papers/xu2026evoarena]]) takes "git is the memory
   layer" literally: rather than keeping memory as a single overwritten
   latest state, it maintains an **append-only patch history** where each
   change records pre-update memory, post-update memory, the rationale,
   and the supporting evidence. This directly diagnoses a failure mode for
   any overwrite-in-place memory — **state collapse**, where an update
   clobbers a fact still valid for an older context and *also* loses the
   provenance explaining when it was valid. The lesson for this project's
   markdown-file memory: the substrate is already git-tracked, so the
   patch history is *free* (`git log` on a concept file is exactly the
   pre/post/rationale trail EvoMem reconstructs) — but only if commit
   messages carry the rationale and the change keeps the superseded claim
   recoverable rather than silently rewriting it. Provenance (guidance #3)
   should extend across *time*, not just to the originating source.

## Connections

- Composes with [[concepts/structured-world-model]]: agent-native
  memory is the *substrate* (markdown files the agent owns);
  structured-world-model is one possible *schema* for organizing
  fields within an entry. The two answer different questions:
  "where does memory live?" vs. "how is its state shaped?"
- Composes with [[concepts/file-as-bus]]: file-as-bus is the
  multi-agent coordination case of the same pattern — same
  filesystem-as-system-of-record idea, but specifically as a bus
  among several specialists rather than as the long-term memory
  of one curating agent. ByteRover is one agent over time;
  AiScientist's File-as-Bus is multiple agents within a session.
- Pairs with [[concepts/citation-anchoring]]: every write benefits
  from an anchor (source paper, run id, snippet) so future
  retrievals carry provenance.
- Pairs with [[concepts/selective-memory-retrieval]]: this concept
  covers the *write-side and storage substrate* (where memory
  lives, how it's organized, how it's curated); selective-retrieval
  covers the *read-side policy* (when the agent decides to consult
  it during a reasoning trajectory). ExpWeaver
  ([[literature/papers/zhao2026expweaver]]) shows that the two
  axes are largely orthogonal — improvements on one side don't
  obviate the other.

## Open questions

- The relation-graph contribution ablation in ByteRover was small
  (−0.4 pp) on LongMemEval-S; whether explicit edges materially
  beat dense retrieval on *research-graph* synthesis (vs.
  long-conversation retention) is untested. Our use case is closer
  to the multi-hop synthesis side, where the value should be larger.
- AKL's specific parameters (importance increments, τ = 30, hysteresis
  gaps) lack sensitivity analysis. Whether the math matters, or any
  reasonable scheme suffices, is open.
- Status is `experimental` because the project enacts the pattern
  implicitly but does not yet enforce it as discipline. Graduate to
  `active` when a downstream project lifts the AKL math (or another
  formalization) to drive its own status transitions.
