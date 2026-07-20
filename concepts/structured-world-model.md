---
kind: concept
name: "structured-world-model"
status: experimental
added: "2026-04-24"
source_papers:
  - mitchener2025kosmos
  - chen2026toward
sources:
  - "[[literature/papers/mitchener2025kosmos]]"
  - "[[literature/papers/chen2026toward]]"
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/xu2026evoarena]]"
  - "[[literature/papers/cao2026agentsk1]]"
used_by: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/agent-native-memory]]"
related_experiments: []
tags: [world-model, long-horizon, coordination, shared-state]
---

# structured-world-model

## Definition

A shared, field-indexed state object that multiple sub-agents read
and write to coordinate over long horizons. Not a free-text
scratchpad, not a message log — a schema'd object whose fields
(hypotheses, evidence, provenance, open questions, open experiments)
are named and addressable. Sub-agents query and update fields rather
than exchanging free-form messages.

## Why it matters

Kosmos ([[literature/papers/mitchener2025kosmos]]) reports 12-hour
autonomous runs spanning ~200 agent iterations, ~42,000 lines of
code executed, ~1,500 papers reviewed per run. Coherence at that
horizon is the hardest problem — sub-agents that pass free-text
messages drift, forget, or contradict each other. The paper's
finding is that a structured world model is the mechanism that
keeps the data-analysis agent and the literature-search agent
aligned across this horizon.

The equivalent without a world model would be either passing the
full transcript (context-window death) or compressing the transcript
(information loss). Field-indexed state dodges both: each sub-agent
reads only the fields relevant to its role, and the schema enforces
consistency across rounds.

## Implementation guidance

1. **Define the schema explicitly.** For a research project, at
   minimum:
   ```yaml
   hypotheses: []         # active + rejected + pending
   evidence: []           # {claim, anchor, source}
   open_experiments: []   # slugs, status, expected metric
   open_questions: []
   recent_findings: []    # last N results
   ```
   Extend per-project. The point is that every field is named so
   downstream prompts can reference it precisely.

2. **Sub-agents read field subsets, not the whole model.** The
   coder sees `open_experiments` and relevant `evidence`; the
   literature agent sees `open_questions` and `recent_findings`.
   Scoped reads keep each sub-agent's context manageable.

3. **Every write carries provenance.** An agent that updates
   `evidence:` cites the anchor (see
   [[concepts/citation-anchoring]]). Unsourced writes are defects.

4. **The world model is the NOTES.md tail plus.** In the simplest
   realization, NOTES.md + `_meta/iteration_log.md` +
   frontmatter-indexed experiment READMEs together already form a
   crude world model. A dedicated file (e.g. `_meta/world.yaml`)
   is worth promoting when manual maintenance of the implicit one
   starts failing.

## Open questions

- Kosmos does not publish the schema. Re-deriving a good one from
  the paper's reported behavior is partly inference; the right
  schema likely varies by domain.
- **A published, full-scale schema now exists for the knowledge half
  of the state.** Agents-K1 ([[literature/papers/cao2026agentsk1]])
  publishes what Kosmos withholds: a complete five-category schema
  (meta/factual, textually-mentioned, implicit/abstracted, citation
  relations, knowledge relations) for structured scientific knowledge,
  with per-field provenance ⟨doc, section/page, span⟩ plus a
  calibrated confidence score — guidance #3's "every write carries
  provenance" enacted at 2.46M-paper scale — and its propositions
  (identifier-preserving joins, cross-view reachability) formalize
  *why* one connected, field-addressable graph beats reasoning over
  separate text fragments. Scope caveat: Scholar-KG is a read-mostly
  knowledge substrate, not a mutable coordination object — it attests
  the schema'd-state half of this concept (typed fields,
  addressability, provenance), not the multi-agent read/write
  coordination half.
- AiScientist ([[literature/papers/chen2026toward]]) takes a
  related but distinct stance: the durable state lives as
  *heterogeneous artifacts in a workspace* (paper analyses, code,
  configs, append-only logs) rather than as a single field-indexed
  object. Its File-as-Bus ablation drops PaperBench by 6.41 points
  and MLE-Bench Lite Any Medal% by 31.82 points — strong evidence
  that *durable state continuity* is what matters, with the schema
  vs. workspace question being a design knob within that. See
  [[concepts/file-as-bus]] for the workspace variant.
- Whether the structured-world-model approach is load-bearing at
  shorter horizons (1-2 hours) or only at 12h+ is unknown — could
  be the kind of ablation a downstream project runs.
- **The world model's state may need to be versioned, not just
  field-indexed.** EvoArena ([[literature/papers/xu2026evoarena]]) shows
  that when the *environment itself* evolves (interfaces, rules, code,
  user prefs change across releases), a latest-state world model suffers
  **state collapse** — overwriting a field loses a value still valid for
  an older version and the rationale for the change. Its fix
  (version-aware state tracking via an append-only patch history) suggests
  schema fields here should arguably carry their own change history, not
  just current values — a `hypotheses:` field that records *why* a
  hypothesis was rejected and under what conditions, recoverable later, is
  more robust than one that just deletes rejected entries.
- Status is `experimental` because no current skill materializes
  a world-model file; NOTES.md + log files substitute implicitly.
  Graduate to `active` when a downstream project uses a dedicated
  world-model artifact end to end.
