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
  - "[[literature/papers/kerestecioglu2026human]]"
  - "[[literature/papers/liu2026evolvemem]]"
  - "[[literature/papers/xu2026single]]"
  - "[[literature/papers/zhou2026ready]]"
  - "[[literature/papers/xu2025amem]]"
  - "[[literature/papers/chhikara2025mem0]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/posts/paddo-dev-claude-code-leak-harness-exposed]]"
  - "[[literature/papers/alzahrani2026persistent]]"
  - "[[literature/papers/tang2026memory]]"
  - "[[literature/papers/yu2026recursive]]"
  - "[[literature/papers/tang2026wikiskill]]"
  - "[[literature/papers/he2026stored]]"
  - "[[literature/papers/goyal2026does]]"
  - "[[literature/papers/hu2026memory]]"
  - "[[literature/papers/das2026selfpropagating]]"
  - "[[literature/papers/pham2026reliable]]"
  - "[[literature/papers/qin2026autoresearch]]"
  - "[[literature/papers/jayaram2026how]]"
used_by: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
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

### The portability assumption, now measured

The pattern has always carried an untested premise: that a
plain-Markdown store **outlives the model that wrote it**.
[[literature/papers/goyal2026does]] is the first source to measure it,
and the result splits this concept's own artifact in two. Under a
writer swap:

- **fixed-schema structure transfers essentially perfectly** —
  KG-fixed accuracy moved by +0.0004 ± 0.0020;
- **model-compressed natural-language notes are strongly
  model-coupled** — NOTES shifted by **+9.91 or −13.28 points**, and
  *asymmetrically by migration direction*, so one upgrade test does
  not generalise to the reverse.

A note in this repository is both at once: the **YAML frontmatter is
KG-fixed** and the **Markdown prose body is NOTES**. The durable layer
is therefore the frontmatter, and the prose is the part that degrades
when the writing model changes. That is a reason to push load-bearing
facts *into* structured fields rather than leaving them in prose — an
argument this concept had no evidence for before.

The second finding is more consequential than the first: **80% of the
NOTES deficit is information destroyed at write time**, and store-only
repair failed to reach 90% recovery in all 48 cases, whereas retaining
the raw source history succeeded in 34 of 48. A compressed store is
not self-sufficient. This is an independent performance argument for
the `raw/` is immutable rule, which was adopted on provenance grounds:
`raw/` *is* the retained source history that makes a degraded note
repairable at all.

Caveat on strength: goyal's histories are synthetic with randomized
answer codes, which is maximally compression-hostile and likely
overstates the prose penalty for real notes carrying semantic
redundancy. The direction is credible; the magnitude is an upper
bound.

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

9. **Link and evolve at write time — new entries rework old ones.**
   A-Mem ([[literature/papers/xu2025amem]], NeurIPS 2025, code) is the
   peer-reviewed anchor for building agent memory on explicit
   Zettelkasten principles: every new note gets LLM-generated
   keywords/tags/context, a cheap embedding filter proposes neighbor
   candidates, an LLM decides which links are real, and — the
   distinctive move — the LLM then **retroactively updates the
   neighbors** (memory evolution) so the network reorganizes as
   knowledge accumulates. Its ablation is directly load-bearing:
   removing link generation collapses multi-hop QA (27.0 → 9.7 F1),
   while evolution adds refinement on top — i.e. the *links* are where
   the value lives, which is this project's wikilink discipline
   attested with numbers. `/ingest`'s update-existing-concepts-on-
   arrival step is memory evolution enacted; A-Mem's caveat (it
   rewrites neighbors in place, losing what they said before) is
   answered here by guidance #8 — evolve *through* git history, not
   over it.

   Mem0 ([[literature/papers/chhikara2025mem0]], 60.8k-star deployed
   system) is the production-scale sibling to A-Mem's academic design:
   same principle (an LLM decides how new information changes existing
   memory) at conversation scale rather than corpus scale, via a
   coarser four-way ADD/UPDATE/DELETE/NOOP tool-call instead of A-Mem's
   note-linking-plus-evolution. It trades A-Mem's richer cross-linking
   for 91% lower p95 latency and >90% token savings vs. full-context —
   a concrete data point that *how much* structure a write policy
   carries is a tunable cost/quality knob, not a fixed requirement.
   See [[concepts/verified-memory-writes]] for how its write gate
   compares to TrustMem's decomposed verification.

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

- **A naturalistic deployment now attests the pattern at scale, not
  just in benchmark or architecture-description form.** Alzahrani
  ([[literature/papers/alzahrani2026persistent]]) is a 115-day
  self-observed case study of one physician-scientist's persistent
  agent environment: 502 memory-related files, 17 configured agent
  directories (role specialization), 57 skill files (reusable
  procedures) — markdown/file-based memory the agent reads and writes
  directly, no vector DB or embedding service named anywhere in the
  paper. That is guidance #1 and #4 above (hierarchical addressable
  paths; memory ops as tools, not services) observed in the wild
  rather than benchmarked. It is n=1 and self-report, so it cannot
  attest *effectiveness* the way ByteRover's LoCoMo/LongMemEval-S
  numbers do — but it is independent ecological evidence that the
  agent-native pattern is something a real, non-CS-lab researcher
  actually built and ran for four months, not only something ML labs
  demonstrate on benchmarks.
- **The paper also surfaces a token-economics signature of a mature
  agent-native memory system that this concept does not yet model.**
  Its May 2026 telemetry subset was 82.9% cache-read tokens
  (Cache-Dominance Ratio); the authors read this as evidence that once
  a persistent workflow has accumulated enough reusable memory and
  procedure, most "work" is *reading back* durable state rather than
  generating fresh inference — and argue evaluation should therefore
  price cost-per-completed-artifact rather than cost-per-token. That's
  a plausible mechanical consequence of guidance #6 (tiered/cheap
  retrieval before agent loops) at the token-accounting layer, but it
  rests on one 25-day window from one deployment. Worth tracking as a
  candidate future concept (token-economics-of-persistent-memory or
  similar) if a second independent source reports a comparable
  cache-dominance pattern; not yet promoted on a single data point.
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

## Any persistent file is memory; auto-loading is what makes it act (2026-10-13)

This concept treats memory as Markdown files the agent writes and reads
directly. [[literature/papers/das2026selfpropagating]] shows the
security consequence of that definition: removing the memory tool does
not remove memory. With no memory tool and no mention of memory, agents
that wanted a later session to act wrote the goal to ordinary files
(`/soc/handover.md`, `STANDING_PRACTICE.md`, `persistent_memory.md`) at
almost the rate they had written to memory (13.1 vs 13.5 of 20
scenarios). A tool that exists but errors on every write made no
difference.

The variable that mattered was the **read path**. End-to-end
propagation fell from 12.5 to 4.2 scenarios per model, almost entirely
because the successor had to find and open the file, while memory is
injected into context every session. So for a file-based memory the
security boundary is the set of auto-loaded files, not the memory API.
In this project that set is CLAUDE.md, NOTES.md as read by `/wrap`, and
concept files imported downstream. A directive that lands there has the
memory-open properties. One that lands in an unindexed file mostly does
not. This is our inference: the paper's harness had no auto-loaded files
outside memory.

## A self-curated store served whole can cost more than no store (2026-10-19)

This concept's defining move is that the agent doing the task also
writes and curates its own memory. [[literature/papers/pham2026reliable]]
measures what that produces when the store is served whole. Across 33
ACE growth runs (11 settings × 3), 14 by our count score more than 5
points below no playbook, and the worst scores 69.6 below. The same
procedure on the same tasks in another order gave 82.5, 58.9 and 30.0
against 78.4 with no playbook. A Dynamic Cheatsheet store (rewritten
whole each batch) behaved the same way: +13.2 and −14.0 on one setting.
Top-k similarity retrieval instead of whole-serving did not help (ACE
50.0 vs whole 51.2 vs none 54.4). So ByteRover's benchmark wins
([[literature/papers/nguyen2026byterover]]) do not show that a
self-curated store is safe to auto-load.

The fix that carried most of the gain is simple: grow more than one
store, and keep "load nothing" as a candidate scored on fresh practice
episodes. Applied to this project (our inference, untested here): the
auto-loaded set of CLAUDE.md, NOTES.md and imported concepts is a
playbook served whole and has never been compared with a no-memory
control. Caveat: the paper needs hundreds of comparable practice
episodes. Research tasks are mostly one-off, so the comparison would have
to run on recurring jobs, not on unique investigations.

## Agent-written insight cards from the same stream moved the first solution, not the final one, and a correct lesson without its conditions overapplied (2026-11-17)

The 10-19 section ([[literature/papers/pham2026reliable]]) found that a
self-curated store served whole is not reliably better than none.
[[literature/papers/qin2026autoresearch]] adds the control pham lacked:
memory of the same size but from an unrelated task stream. On one
forecasting stream (4 transfer tasks, 4 frontier agents, one run each),
each task starts a fresh session and receives all agent-written cards
(claim, evidence, confidence, scope, actions, failure conditions) from
earlier tasks, cards from another stream, or none.
- **Relevance moved the start.** Matched minus shuffled first-solution
  gain: +2.6 Opus 5, +3.9 GPT-5.6-Sol, +5.9 GPT-5.4, −2.9 Sonnet 4.6.
- **It did not move the finish.** Final outcome: +1.0, −3.2, +0.2, −0.2.
  Time-convergence was worse with matched cards for three agents (−15.5,
  −27.3, −0.9) and better for Sonnet (+25.6).
- **Any cards beat none about equally.** Over no memory, matched cards
  gave +14.7 (Sonnet) and +5.0 (GPT-5.4); shuffled gave +14.9 and +4.8.
- **Two failure cases.** 21 matched cards describing the exact schema
  anchored Sonnet on one model family (RMSE 0.588 vs 0.543 shuffled, 0.538
  no memory). In the main persistent-session runs, Opus 5 turned one real
  failure into the standing rule "write a valid final within the first ~3
  minutes, before any tuning". On the next tasks it made 4 valid attempts
  vs 28 for the reset run and lost (ROC-AUC 0.978 vs 0.998).

Rule (our inference): a lesson the agent writes for its future sessions
should carry the condition it was formed under and what would retire it,
not only the rule. The box's auto-memory feedback entries and NOTES.md
"Findings" are this kind of store, served whole. Neither paper shows that
relevant self-written memory raises a final result; both show it changes
behaviour, in either direction.

Limits:
- One stream, four tasks, one run per cell. Cards are always all served,
  never retrieved. The ablation does not compare cards with the persistent
  session, so it cannot say whether insights or carried code and
  transcript drove the main stateful effect.
- The overapplied-lesson case is one run, on a stream with host contention.

## A lossless self-written store served whole stayed at or above no memory for five models, and a 12-rule compression kept a third of the gain (2026-11-24)

The 10-19 section ([[literature/papers/pham2026reliable]]) found ACE
playbooks grown online scoring below no playbook in 14 of 33 runs.
[[literature/papers/jayaram2026how]] serves a differently built store
whole. Each model's own trajectories on AppWorld train/dev yield 1–3
lessons each. Success or failure is labelled by the evaluator, not by
the model. Near-duplicates are merged into guidelines that each carry a
support count, and nothing the merge misses is dropped. The store is
built once and frozen. The same model writes, merges and reads.

- **Served whole, it never fell below none.** Under the rule fixed
  before any test run (inject everything), TGC on 168 tasks moved +9.5
  (DeepSeek-V3.2, 191 guidelines), +11.3 (gpt-oss-120b, 199), +2.9
  (GPT-5.5), +1.2 (Gemma-4-31B) and 0.0 (GLM-5). DeepSeek's gain holds
  on the 417-task hard split (60.4 → 72.9). In a second scaffold,
  GLM-5.3-Flash went 87.8 → 93.2 over two runs per arm, with
  non-overlapping ranges.
- **Compression lost most of the gain.** A consolidation prompt asking
  for "a small set of highly relevant rules" gave 12 rules and +2.9 for
  DeepSeek, against +9.5 for the lossless 191. That is 11 tasks apart.
  The 47 guidelines with support ≥ 2 gave +5.9, so the 144 support-one
  guidelines added about 6 tasks in one run (our arithmetic). The
  direction is consistent with [[literature/papers/guo2026how]]: a
  model's rewrite drops first what it would not write itself.
- **More memory cost input tokens, not steps.** The full set cost +78%
  tokens per task, re-sent every step. The support ≥ 2 set cost +27%.

What it does and doesn't support:
- **It narrows the 10-19 warning rather than contradicting it.** The
  stores differ on three axes at once: offline batch mining versus
  online growth, outcome labels from the evaluator versus Reflector
  tags, and lossless merging versus incremental edits. Any of them could
  explain why this store was safe to serve whole. The paper tests none
  separately. pham2026reliable's tags-versus-effect result (rank
  correlation 0.21) points at the label source as one candidate.
- **It does not show "keep everything" is right for every reader.** The
  only Claude model, Opus 4.6, was run with the 12-rule set alone
  (+4.1), so "do not compress" is untested on the strongest model in
  the study.
- **For this box (our inference):** the auto-loaded set (CLAUDE.md,
  NOTES.md, imported concepts) is a store served whole that has never
  been compared with no memory. This paper is weak evidence that a
  lossless, outcome-labelled store served whole is not the dangerous
  case. It is not evidence for this box's prose stores, which are
  rewritten and summarised.

Limits:
- Single runs, no intervals, configurations compared on test. The fixed
  rule is the cleanest number. Identical DeepSeek runs flipped ~32 of
  168 tasks.
- AppWorld tasks recur and have an authoritative verdict. A store mined
  from a train split has no direct analogue for one-off research tasks.
- Deployment report from the library's authors; peer review unknown.
