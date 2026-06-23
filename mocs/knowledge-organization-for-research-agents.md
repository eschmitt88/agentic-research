---
kind: moc
name: "knowledge-organization-for-research-agents"
status: active
added: "2026-05-11"
concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/web-grounded-literature]]"
tags: [moc, knowledge-organization, agent-memory, architecture]
---

# Knowledge Organization for Research Agents

How autonomous research agents store, structure, curate, and consult
the knowledge they work with — in practice, not in principle. This
MoC ties together the nine concepts that span this question across
three layers: the *substrate* (where memory lives), the *write-side*
(how the library evolves and how outputs are organized), and the
*read-side* (when and how stored knowledge is consulted during
decision-making).

The thread emerged from the 2026-05-11 digest batch, which
delivered four papers (GSAR, ExpWeaver, SkillOS, SkillRet, plus
AutoResearchBench as empirical baseline) each attacking a different
layer of the same architectural question. Reading them together
made the three-layer split explicit and surfaced a working
hypothesis: most autonomous-research-agent work optimizes one layer
in isolation, but the three layers compose and the bottleneck shifts
depending on which is weakest.

## The three layers

### Substrate: where memory lives

The physical substrate that holds the agent's knowledge between
invocations. The empirical move that unifies this layer's papers
is *file-system-as-system-of-record* — markdown files with explicit
provenance, addressable by path, grep-able, git-trackable. No
proprietary databases, no embedding-only stores, no in-memory
state.

- **[[concepts/agent-native-memory]]** — long-term memory of one
  curating agent. Markdown files in a hierarchical layout,
  cross-referenced by explicit wikilinks. Memory operations
  (`/ingest`, `/lint`, `/sync-imports`) are agent-callable skills,
  not external API calls. *Canonical sources:* ByteRover
  ([[literature/papers/nguyen2026byterover]]) — 96.1% on LoCoMo
  with zero external infrastructure; ExpWeaver
  ([[literature/papers/zhao2026expweaver]]) — extends the concept
  to expose memory as an optional resource; SkillOS
  ([[literature/papers/ouyang2026skillos]]) — literal markdown-file
  skill library managed via file I/O; SkillRet
  ([[literature/papers/cho2026skillret]]) — empirical anchor on
  what a library-at-scale looks like (17.8K skills, median 1.6K
  tokens each).

- **[[concepts/file-as-bus]]** — multi-agent coordination case of
  the same pattern. Workspace as coordination substrate;
  role-aligned regions; permission-scoped writes; append-only logs.
  AiScientist ablation: removing file-as-bus drops PaperBench by
  6.41 points and MLE-Bench Lite Any Medal% by 31.82 points
  ([[literature/papers/chen2026toward]]). The strongest published
  evidence that *durable state continuity* — not "more interaction"
  — is what carries an agent through hours- to days-long tasks.

- **[[concepts/structured-world-model]]** — schema-indexed shared
  state for multi-agent coordination over long horizons. Composes
  with the above: file-as-bus is the workspace; the structured
  world model is one (schema'd) artifact in it.

### Write-side: how the library evolves, how outputs are organized

How knowledge gets *into* the substrate and how it gets *shaped*
over time. Three sub-concepts cover different write-time concerns.

- **[[concepts/skill-library-lifecycle]]** — `insert/update/delete`
  as a learned curation policy over the library's life. Empirical
  schedule: insert-dominant early → update-dominant mid →
  small-but-rising delete late. SkillOS
  ([[literature/papers/ouyang2026skillos]]) demonstrates this with
  GRPO + grouped task streams, and the strongest practical finding
  — *a trained 8B curator beats Gemini-2.5-Pro used directly as
  curator on the same executor* — is the empirical anchor for
  curator-executor calibration: good curation is reader-specific,
  not absolutely good or bad.

- **[[concepts/typed-claim-partition]]** — agent *output* claims
  organized into typed buckets (grounded / ungrounded / contradicted
  / complementary) with epistemic-strength weights and tiered
  recovery decisions. GSAR
  ([[literature/papers/kamelhar2026gsar]]) operationalizes typed
  grounding as a control signal rather than a passive audit field.
  This is the dual of skill-library-lifecycle: lifecycle organizes
  *stored* skills over time; typed-claim-partition organizes
  *emitted* claims at production time.

- **[[concepts/citation-anchoring]]** — every claim traces to a
  source (file:line, `metrics.json:<field>`, or wikilink). The
  general rule that the other two write-side concepts specialize:
  skill-library-lifecycle's `sources:` frontmatter, GSAR's
  evidence-typed weighting, AutoResearchBench's full-text
  constraint verification all rest on this discipline.

### Read-side: when and how stored knowledge is consulted

How an agent decides *whether* and *what* to consult from its
substrate at decision time. The most under-theorized layer in the
literature; the two papers from this batch attack it head-on.

- **[[concepts/selective-memory-retrieval]]** — read-side policy:
  retrieval is a runtime decision, not a fixed schedule. Triggered
  by uncertainty signals (token entropy, repeated tool failures,
  explicit `[Retrieve]` tokens emitted mid-reasoning). ExpWeaver
  ([[literature/papers/zhao2026expweaver]]) shows that this gating
  beats both initialization-only and always-on schedules across 4
  frameworks × 7 backbones × 3 environments — and that the gating
  policy is learnable via RL with task-success reward.

- **[[concepts/multi-granularity-memory]]** — the *what-to-read* axis
  that complements selective-memory-retrieval's *whether-to-read*.
  Memory is stored at several coexisting grains (turn / summary /
  keyword, or working / episodic / semantic tiers) and retrieval
  routes to the grain that fits the query. MemGAS
  ([[literature/papers/xu2026single]], ICLR 2026) is the canonical
  anchor: four co-stored grains, a cross-granular GMM association
  graph, and an *entropy-based router* that weights each grain by the
  certainty of its match distribution — making granularity selection
  literally an instance of uncertainty-gated retrieval. The
  cognitively-motivated variant
  ([[literature/papers/kerestecioglu2026human]]) reframes
  consolidation-between-tiers as the write-side dual: promoting fine
  episodic detail into a coarse semantic grain is eviction
  ([[concepts/context-eviction-policy]]) seen from the grain axis. Maps
  onto this project's own graph, where a concept already exists at
  several grains — the `index.md` one-liner, the `## Definition`, the
  full body, the git history — and the right grain depends on the
  query.

- **[[concepts/web-grounded-literature]]** — read-side policy for
  *external* knowledge specifically: continuous intake of primary
  sources (`/discover`, `/fetch-paper`, `/digest`) rather than
  reliance on training-time priors. AutoResearchBench
  ([[literature/papers/xiong2026autoresearchbench]]) provides the
  realistic SOTA bound: frontier models max out at ~9% on
  autonomous literature-discovery tasks. The implication isn't
  that automation has failed — it's that the *curator-in-the-loop*
  pattern (this project's design) matches what the evidence
  supports.

## Cross-layer hypotheses

Reading the layers together (rather than as siloed contributions
each in isolation) surfaces a few hypotheses worth testing:

**1. The three layers are largely orthogonal but compose
non-additively.** A good substrate (markdown files, agent-native)
doesn't fix a bad read-policy (always-on retrieval) or a bad
write-policy (insert-dominant forever). Conversely, a learned
curator (SkillOS) running over a poor substrate (vector-DB-only,
no explicit provenance) is limited by what the substrate can
express. The combinations that show up in the literature are
typically just *one* layer at a time:

| Paper | Substrate | Write-side | Read-side |
|---|---|---|---|
| ByteRover | markdown-files | implicit | tiered cache+BM25 |
| GSAR | (orthogonal) | typed partition | (orthogonal) |
| ExpWeaver | inherits framework | inherits framework | learned trigger |
| SkillOS | markdown-files | learned `{insert, update, delete}` | fixed BM25 top-k |
| SkillRet | markdown-files | empirical anchor only | fine-tuned retriever |
| AutoResearchBench | DeepXiv full-text | (orthogonal) | ReAct agent loop |

None of the papers do all three. The natural next experiment is
combining a *trained read-side gate* (ExpWeaver's contribution)
with a *trained write-side curator* (SkillOS's contribution) over
a *markdown-file substrate* (agent-native-memory's contribution).

**2. Curator-executor calibration generalizes beyond the curator
role.** SkillOS's finding that a small trained curator beats a
frontier zero-shot curator is, generalized: *good knowledge
organization is reader-specific*. The natural extensions:

- A retriever should be calibrated to *what its consumer agent
  actually uses*, not to general-purpose IR benchmarks (SkillRet
  evidence: MTEB rank doesn't predict skill-retrieval rank).
- A claim-grounding score function should be calibrated to *how its
  downstream consumer plans to act on the score*, not to abstract
  faithfulness (GSAR §4.4).
- For research-memory projects: note structure should be calibrated
  to *what future Claude sessions consulting the graph actually
  retrieve and use*, not to what looks clean to a current human
  reader.

**3. The read-side is the most actionable improvement frontier for
this project right now.** The substrate is already in place
(this is enacted: markdown files, frontmatter, wikilinks, immutable
`raw/`). The write-side is in place too (the `/ingest` / `/lint` /
`/wrap` cycle, plus this batch's new concept structure). What's
explicitly *not* in place yet is the read-side policy: `/digest`
and `/iterate` today use initialization-only loading. ExpWeaver's
evidence + AutoResearchBench's empirical bounds suggest the highest
leverage tweaks are:

- `/digest` query synthesis should produce longer, scenario-rich
  natural-language queries (AutoResearchBench finding).
- `/digest` retrieval backend could be specialized arxiv full-text
  rather than general WebSearch (AutoResearchBench finding).
- Sub-agents in `/iterate` could emit `[Retrieve]` triggers when
  proposals stall, rather than always loading the top-5 lit notes
  upfront (ExpWeaver finding).

These are concrete, implementable, and rooted in the empirical
evidence from this batch.

## How this MoC will evolve

This MoC is `status: active` rather than `experimental` because the
substrate and write-side layers are already enacted by the project's
existing skills and the literature evidence is strong. The read-side
recommendations are concrete experiments rather than enacted
practice; they may shift as `/digest` and `/iterate` evolve.

Concepts that *don't* belong here but might at next batch:

- **`hybrid-model-backends`** is tangentially related (the curator/
  executor split in SkillOS is a model-role split), but it's
  primarily about cost/model selection, not memory organization.
  Keep it as a sibling concept, not as a knowledge-org member.
- **`hierarchical-delegation`** governs *who* writes/reads what,
  which interacts with this MoC's substrate layer (file-as-bus
  permission scoping) but is its own concern.
- **`evolutionary-expansion`** and **`budget-as-ceiling`** are
  orthogonal to knowledge organization (they govern search and
  resource policy respectively).

If a future batch produces a concept on *agent-readable
knowledge-graph synthesis* (taking many literature notes and
producing a synthesized survey or MoC automatically), that would
be a natural addition here.

## Connections out

- **[[concepts/hce-evaluation]]** — bounded outer-loop discipline
  that any knowledge-org system needs to avoid overfitting to its
  own internal signal. The HCE discipline (separate validation
  and test) maps to memory architectures too: validation-time
  reads can use the full graph, but final-scoring reads should
  use a frozen snapshot to avoid the system curating itself to
  look good on its own evaluator.
- **[[concepts/pass-at-k]]** — AutoResearchBench's test-time
  scaling result (pass@16 ≈ 28% on Deep vs 13% on Wide) is a
  domain-specific instantiation of pass@k that informs *how* to
  parallelize discovery work.
