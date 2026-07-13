---
kind: concept
name: "skill-library-lifecycle"
status: seedling
added: "2026-05-11"
source_papers:
  - ouyang2026skillos
  - cho2026skillret
sources:
  - "[[literature/papers/lam2026governing]]"
  - "[[literature/papers/liu2026harnessing]]"
  - "[[literature/papers/yang2026skillopt]]"
  - "[[literature/papers/ye2026evolutionary]]"
  - "[[literature/papers/zhou2026comprehensive]]"
  - "[[literature/papers/ouyang2026skillos]]"
  - "[[literature/papers/cho2026skillret]]"
  - "[[literature/papers/zhang2026skillcomposer]]"
  - "[[literature/papers/wu2026bayesian]]"
  - "[[literature/papers/pu2026skillops]]"
  - "[[literature/papers/belikova2026managing]]"
  - "[[literature/papers/zhao2026generative]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/papers/zhao2026agenticos]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/evolutionary-expansion]]"
related_experiments: []
tags: [skill-library, curation, knowledge-organization, write-policy, procedural-memory, lifecycle]
---

# skill-library-lifecycle

## Definition

A *skill library* is a curated, evolving collection of reusable
procedural-memory artifacts — markdown files with explicit
`name`/`description` frontmatter and structured body sections —
maintained through three explicit operations: **insert** (add a new
skill), **update** (refine an existing skill), and **delete** (prune
redundant or harmful skills). The lifecycle policy — when to insert
vs update vs delete, and how that balance shifts over the library's
maturation — is itself a load-bearing design decision that can be
heuristic, prompt-driven, or learned via RL with downstream-task
reward.

## Why it matters here

Most agent-memory work treats curation as a side effect of
representation choice: pick a good skill format and the library
takes care of itself. SkillOS
([[literature/papers/ouyang2026skillos]]) shows this is wrong on
both ends. Holding the experience-construction pipeline of three
existing frameworks (ReasoningBank, MemP, AWM) fixed and changing
*only the curation policy* yields the largest reported gains in the
self-evolving-agents literature to date. Conversely, the empirical
curator-evolution pattern (insert-dominant early → update-dominant
mid → small-but-rising delete late) demonstrates that *good*
curation policies have a recognizable shape: libraries that only
grow (never refine or prune) underperform libraries that consolidate.

For a research-agent project specifically, this is the missing piece
between [[concepts/agent-native-memory]] (the substrate — markdown
files in a git-tracked directory) and [[concepts/selective-memory-retrieval]]
(the read-side policy — when to consult). Skill-library-lifecycle
is the *write-side policy*: how the library is grown, refined, and
pruned over the course of an agent's deployment. Together the three
concepts span a complete memory architecture.

This project already enacts a manual version of the pattern. `/ingest`
is the insert operation; concept-merge edits during ingestion (like
this one) are update; the implicit "retire stale notes" practice is
delete. The concept names what is currently implicit so that future
work can sharpen it — e.g., a `/lint` extension that surfaces
"concepts that should be merged" rather than only orphan-flagging.

## Implementation guidance

1. **Three operations, not one.** Insert alone gives an
   ever-growing library that becomes noise. Update alone never
   admits new content. Delete alone is degenerate. The lifecycle
   needs all three, with the *balance* between them shifting as the
   library matures. SkillOS's empirical schedule: insert-dominant
   for the first ~10 cycles, update-dominant from ~15 onward,
   delete a small but persistent fraction throughout.

2. **Markdown-file substrate keeps operations transparent.** Each
   skill is one file. Operations are file I/O (write / overwrite /
   delete). This means: every operation produces a git diff; every
   change is grep-able and auditable; recovery from a bad operation
   is `git revert`. SkillOS uses single-file Markdown for tractability;
   Anthropic's full SKILL.md spec allows multi-file hierarchical
   skills, but the single-file version is sufficient for most
   research-memory uses and is what this project's `concepts/` and
   `literature/` directories enact.

3. **Curate against the executor, not abstract quality.** SkillOS's
   strongest empirical finding: a trained 8B curator beats
   Gemini-2.5-Pro used directly as curator, *on the same executor*.
   The reason is that good curation is reader-specific. For research
   notes, this means: optimize note structure for what *future
   Claude sessions consulting the graph* will actually retrieve and
   use — not for what looks intrinsically clean to a current human
   reader.

4. **Long-horizon supervision via task grouping.** Curation reward is
   indirect and delayed: a skill inserted today only proves its value
   by helping later related tasks. SkillOS's *grouped task streams*
   trick — train curation policy on bundles of related tasks where
   earlier-task curations are graded by later-task success — converts
   delayed signal into dense GRPO reward. In a research-memory
   context, this maps to ingesting *clusters* of related papers in
   one curation cycle so the inter-paper relationships exercise the
   schema. The Map-of-Content threshold (≥5 concepts on a theme)
   is the project's natural unit of grouping.

5. **Behavior dynamics over the library's life are diagnostic.** A
   library that is always insert-dominant signals an under-pruned
   knowledge graph. A library that suddenly tilts to delete
   suggests the schema has destabilized. `/lint` (or any other
   periodic auditor) should track the ratio of insert/update/delete
   operations over time as a health metric, not just structural
   defects.

6. **Execution-time curation, not only batch-time.** SkillOS curates
   between task groups; ExpWeaver curates implicitly through gated
   retrieval. Hermes Agent
   ([[literature/repos/nousresearch-hermes-agent]]) adds a third
   mode: **skills self-improve during use** — a skill executed
   today may be edited (by the agent) as a side-effect of execution,
   not as a separate curation pass. This is closer to continuous
   refinement (ReasoningBank-style) than to scheduled curation. The
   three modes — execution-time, retrieval-time, batch-time — are
   not mutually exclusive; a mature library likely uses all three.

7. **The construction operations can be *learned*, and the lifecycle
   triple is not unique.** SkillComposer
   ([[literature/papers/zhang2026skillcomposer]]) decomposes skill
   construction into three *learnable* operations — **create**, **improve**,
   **merge** — trained at inference time without ground-truth supervision
   (delta-pass-rate-guided rejection sampling). Its triple is not this
   project's insert/update/delete: *create* ≈ insert and *improve* ≈
   update, but **merge** (consolidate semantically similar skills) is a
   consolidation operation with no clean analogue to *delete* — it fuses
   rather than prunes. SkillComposer names the axis the consolidation
   serves: **specification** (refine to a task pattern; the job of
   *improve*) vs **generalization** (abstract across tasks; the job of
   *merge*) are *orthogonal* quality dimensions, and a good lifecycle
   needs both levers, not just an insert/prune balance. This sharpens the
   write-side picture: the open question is no longer only *when* to
   insert/update/delete but *which consolidation operation* (merge vs
   delete) applies — and whether a mature library needs both.

8. **The *operation* should be selected by verified evidence, not
   reflection.** Bayesian-Agent ([[literature/papers/wu2026bayesian]])
   treats each skill as a hypothesis and maintains a feature-conditioned
   posterior over its success/failure modes from verified trajectory
   evidence; that posterior — not an LLM's self-critique — picks the
   rewrite action from a five-way set: **patch** (failure-mode fix),
   **split**, **compress**, **retire** (prune), **explore**. Two lessons
   for this project's write-side policy. First, the operation set is
   *richer* than insert/update/delete: *split* and *compress* are
   structural reorganizations (decompose an overloaded skill; consolidate
   a verbose one) that neither this project's triple nor SkillComposer's
   create/improve/merge names explicitly — a `/lint` extension might
   surface "concepts that should be split" or "notes that should be
   compressed", not only merged or pruned. Second, and sharper: the
   curation *decision* should be **evidence-driven**. Where SkillOS
   curates against the executor's behavior and SkillComposer learns the
   operations, Bayesian-Agent makes the *choice of which operation to
   apply* a function of accumulated verified outcomes — the antidote to
   uncalibrated growth where every run just appends another note. The
   manual analogue here: a note repeatedly retrieved-but-unused (evidence
   of low value) is a *retire* candidate; a note that keeps being
   partially-cited across unrelated themes is a *split* candidate.

9. **Composition-time selection is its own lifecycle stage — and it
   couples to the write-side.** Generative Skill Composition
   ([[literature/papers/zhao2026generative]]; system confusingly also
   named "SkillComposer" — distinct from
   [[literature/papers/zhang2026skillcomposer]]) formalizes the
   *deployment* end of the lifecycle: given a fixed curated library,
   jointly predict **which** skills to load, **how many**, and **in
   what order** as closed-vocabulary sequence prediction. A ~3.9M-param
   specialist decoder beats flat retrieval and a frontier LLM-judge,
   closing ~80% of the gap to the gold-skill oracle on SkillsBench at
   the lowest prompt-token cost. Two lessons. First, selection is
   *structural*, not similarity ranking — how many skills and in what
   order are decisions a flat retriever cannot express (an unordered
   top-k under- or over-provisions the context). Second, the paper
   deliberately freezes the library to isolate composition from
   creation — which exposes the open seam with the write-side
   operations above: every insert/merge/retire invalidates a trained
   composer's output vocabulary. A living library needs either a
   composer cheap enough to retrain per curation cycle or a
   composition interface robust to library churn.

6. **Empirical reference point for what a library looks like at
   scale.** SkillRet ([[literature/papers/cho2026skillret]]) curated
   17,810 community-contributed Claude skills from a 22,795-entry
   crawl (78.1% retention through 5-stage filtering). Median skill
   length: 1,583 tokens; 95th percentile: 5,531 tokens. Category
   distribution is heavily imbalanced (Software Engineering 62.2%,
   Information Retrieval 3.1% — a 20× spread). This is the natural
   shape of unsupervised community-curated libraries, and any
   curation policy needs to handle the long-tail without
   collapsing it.

## Connections

- **[[concepts/agent-native-memory]]** is the substrate this lifecycle
  operates on. Agent-native-memory says memory lives in markdown
  files with explicit provenance; skill-library-lifecycle says how
  those files are added, refined, and removed over the agent's
  deployment.
- **[[concepts/hybrid-model-backends]]** — SkillOS's
  curator/executor architecture is a direct instantiation of the
  ideator/implementer pattern, with the empirical twist that the
  curator (ideator-role) should be *smaller and trained on the
  executor's behavior*, not larger and frontier-grade. This
  inverts the "use the biggest model for the strategic role"
  default and is worth flagging as evidence that the hybrid-backend
  split benefits from *role-specific training*, not just role-specific
  model selection.
- **[[concepts/selective-memory-retrieval]]** is the read-side
  complement. SkillOS handles write-side curation but uses
  fixed BM25 top-k retrieval at every step — exactly the
  always-on pattern ExpWeaver shows is suboptimal. A complete
  memory architecture combines a trained write-side curator with
  a gated read-side retriever; neither paper does both.
- **[[concepts/evolutionary-expansion]]** — the grouped-task-stream
  training has flavors of fitness-based selection (later-task
  success serves as fitness signal for earlier-step curation
  policy). It's not population-based search (no parallel curator
  siblings), but the "earlier decisions evaluated by later
  outcomes" structure is shared.

## Open questions

- **Joint read/write training.** SkillOS trains write-side
  (curator); ExpWeaver trains read-side (gate). Neither trains
  both. Whether the optimal curator policy *depends on* the read
  policy (and vice versa) is open. Plausibly yes: a curator that
  knows its skills will be selectively-retrieved should curate
  differently than one expecting always-on retrieval.
- **Lifecycle policy across distinct memory types.** SkillOS's
  pattern is shown for procedural skills (how-to knowledge). Does
  the same insert→update→delete arc apply to factual knowledge
  (literature notes), structural knowledge (concept relationships),
  or evaluative knowledge (experimental diagnostics)? The schema
  may differ; the project's three storage tiers
  (`concepts/`, `literature/`, `experiments/`) might each need
  their own lifecycle calibration.
- **Reader-specific calibration at population scale.** SkillOS's
  curator-executor-mismatch finding holds for a single fixed
  executor. In a multi-reader setting (multiple Claude sessions
  with different active contexts, multiple human researchers, or
  multiple downstream projects @importing the concepts), curation
  must balance multiple readers. How to do this without falling
  back to lowest-common-denominator notes is unresolved.
- **When does a library reach steady state?** SkillOS's behavior
  curves (Fig. 4) are reported over ~50 training steps; whether
  the insert/update/delete balance converges or continues to drift
  on longer horizons isn't shown. For a research-memory project
  with multi-year horizons, the steady-state question matters
  practically.
