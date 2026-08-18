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
  - "[[literature/papers/shen2026dynamic]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/papers/zhao2026agenticos]]"
  - "[[literature/papers/shang2026hypothesis]]"
  - "[[literature/papers/hu2026skillbrew]]"
  - "[[literature/papers/huang2026skillwiki]]"
  - "[[literature/papers/tang2026memory]]"
  - "[[literature/papers/cheng2026agenticsts]]"
  - "[[literature/papers/kim2026why]]"
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

10. **Retirement gets a mechanism — and a warning: removal is not
   internalization.** SLIM ([[literature/papers/shen2026dynamic]]) treats
   the active skill set as a trainable *external capability boundary*,
   jointly optimized with the policy. Each audited skill's
   **marginal external contribution** is measured by leave-one-skill-out
   validation on the tasks routed to it (EMA-smoothed); retire fires only
   when contribution is negligible *and* the skill has sufficient
   cumulative exposure *and* a persistent low-contribution streak — the
   guard that protects low-frequency long-tail skills from premature
   pruning. Three findings sharpen this concept. (a) The healthy endpoint
   is **non-monotonic**: expand → fluctuate → stabilize at a compact
   non-empty set (38→46→21), refuting both always-grow (SkillOS's
   insert-dominant early phase is a *phase*, not the end state) and
   prune-to-zero. (b) Forcing the set to zero drops validation 92.2%→76.6%
   — a removed skill was not necessarily absorbed; retirement decisions
   must be measured, not assumed. (c) Frequency is not value in either
   direction: frequently-selected skills can be near-internalized
   (disabling costs ~0.06) while globally-rare skills are locally
   indispensable (−0.250) — so retire-by-LRU or retire-by-age heuristics
   are the wrong policy, and a random-audit ablation is the *worst*
   variant. Manual analogue for this project: a note is a retire candidate
   only after repeated audited exposure with persistently negligible
   contribution, never merely because it is old or rarely linked. Bonus:
   SLIM's *expand* operation emits Anthropic-style `SKILL.md` artifacts
   from inside the RL loop — the lifecycle and the shared skill format
   ([[concepts/shared-skill-namespace]]) are converging.

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

11. **Above the operations sits an infrastructure layer.** SkillWiki
   ([[literature/papers/huang2026skillwiki]], demo-grade) names what
   the per-library operations above presuppose: lifecycle *states* as
   first-class governance objects (Raw → Candidate → Draft → Verified →
   Released → Degraded → Deprecated → Archived), a provenance graph
   linking every skill to the knowledge evidence it was derived from,
   and change-as-reviewed-release (snapshot → structured diff → review
   → release) instead of direct overwrites. Its Degraded state is
   detected from runtime failure clustering, echoing SLIM's
   evidence-based retirement rather than age-based pruning. Evidence is
   feasibility-only (125 artifacts, one full-chain case study), so
   treat this as vocabulary and architecture, not validated mechanism —
   but the vocabulary maps directly onto this project's own
   candidate → ingested → concept → MoC → retired pipeline, where git
   already provides the snapshot/diff/review substrate.

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

## Distilling a skill from a failure rather than from a success

Most sources in this cluster build the library from trajectories that
*worked*. [[literature/papers/cheng2026agenticsts]] populates its skill
layer two ways, and the second is the unusual one: **mistake-driven
discovery**, which reads combat losses against per-enemy baselines and
distills a guide from what went wrong. (The other is human-authored seed
skills.)

The entries themselves are also more structured than most: each guide
carries an **explicit trigger condition**, a prose body, and an index over
recurring state classes, so retrieval is keyed rather than semantic — the
skill fires when its trigger matches the current state type, not when an
embedding is nearby.

Two findings temper the import. The skill layer is where the paper's
largest observed win-rate difference sits (3/10 → 6/10) but the difference
is **not significant** (Fisher p ≈ 0.37, n = 10 per cell), so this is not
evidence that a skill library helps. And the frozen store is
**backbone-sensitive**: the skills were distilled from Gemini 3.1 Pro
trajectories and transplanting them to other model families is reported as
a separate diagnostic rather than pooled, because they do not transfer
cleanly. A distilled skill library may be a per-model artifact, which would
be a real constraint on [[concepts/shared-skill-namespace]] — worth
watching for a second source.

## The loading function is a first-class component, and an unscoped library is worth nothing

The strongest result in this cluster, and it is a negative one.
[[literature/papers/kim2026why]] holds a **159-skill inventory, model,
pipeline and budget constant** across 8 MLE-Bench competitions and varies
only *which skills enter context*:

| Loading | Medal rate | Output tokens | Tokens/medal |
|---|---|---|---|
| Tiered (scope-matched, 10–60 entries) | **8/8** | 2.27M | **284K** |
| Flat (all 159, ~145K chars) | 5/8 | 3.78M | 756K |
| Empty (no skills) | 5/8 | 1.86M | 371K |

**Flat loading performs exactly as well as loading nothing, at twice the
token cost.** A skill library without a scoping function is not a weak
library — it is inert, and per token spent it is worse than having no
library. Flat also ran the *most* experiments (75 vs 60) at a slightly
higher execution success rate, so the additional attempts were simply
poorly directed.

Three mechanisms, diagnosed from logs rather than asserted:

- **Signal dilution** — relevant skills buried among entries scoped to
  unrelated tasks.
- **Context budget displacement** — 145K characters of skill dump "crowds
  out the agent's own reasoning and code analysis."
- **Overconfident priors** — the sharpest instance: on one NLP task the
  flat-loaded agent repeatedly attempted DeBERTa-v3-large and hit OOM,
  while the **skill-free** agent picked simpler models and scored *higher*
  (0.985 vs 0.981). A domain prior actively misled it.

The scoping axis is **applicability scope**, and it is made to coincide
with the agent hierarchy: global skills (5) load for every specialist,
domain skills (19 tabular / 12 NLP / 15 vision) only for the matching
specialist, competition skills (108) only on a re-run of that task. An
agent structurally cannot see out-of-scope knowledge — an information
boundary enforced by architecture rather than by relevance ranking.

**Promotion keeps contradictions instead of resolving them.** Each new
learning is classified `skip` / `competition` / `domain` / `global` /
`conflict`, and "when two learnings conflict, both are kept and annotated
with conditions" — e.g. ensembling helps when correlation is below 0.95 but
hurts when a weak member drags down a strong one. That is the right default
for a store that accumulates across contexts, and it is the opposite of a
last-write-wins update; compare [[concepts/verified-memory-writes]].

Two honest limits before importing. The promotion step — the thing that
*builds* the hierarchy — is explicitly unevaluated ("we do not yet measure
how often it assigns the correct tier"), so the evidence is for loading
given a hierarchy, not for constructing one. And the flat baseline is
dump-everything, not **retrieval over a flat pool at matched context
size**, which is what most systems would actually build. The finding
supports *scope your loading*; it does not yet establish that a hand-built
tier hierarchy beats good retrieval.

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

## The admission gate — governing what becomes a skill at all

Most sources here work the *exit* side: which skills to retire, merge,
or deprecate once the library is already noisy.
[[literature/papers/tang2026memory]] works the entry side, and its
premise is that the noise is avoidable. Distilling skills straight from
raw trajectories is the error — traces carry failed attempts, blind
exploration, and environment-specific artifacts, so the resulting skills
are over-specific and fire in the wrong contexts. Instead, promote from
*governed* memory: grounded step traces (L1) → induced cross-episode
procedural policies (L2) → declarative environmental knowledge (L3),
with only L2 policies crystallizing into callable skills.

The gate is three-part, and worth stating as a checklist:

1. **Evidence retained** — the policy still points at the traces that
   support it.
2. **Positive estimated gain** — it demonstrably helps, rather than
   merely having occurred.
3. **Internal consistency** — its trigger, procedure, and applicability
   boundary agree with each other.

A skill that passes carries its evidence anchors, applicability
boundary, verification rules, and a reliability estimate forward.
Provenance survives promotion instead of being compiled away — see
[[concepts/citation-anchoring]], of which this is the procedural-memory
instance. Our own skill files assert procedures with no link to whatever
established them; the same gap, one level up.

Their answer to sparse credit assignment is **reflection-weighted value
backfilling**: terminal feedback is propagated back over grounded traces
using dense local self-reflections as weights, and the resulting
evidence-calibrated trace value is the single signal governing
retrieval, induction, promotion, and revision alike. That unification is
the more transferable idea than any individual component.

Results: best or tied-best on all five EvoAgentBench domains (+15.4 pts
on software engineering), *lower* cost on four of five, and positive
cross-domain transfer on all six pairs tested (+2.6 to +5.1) — evolved
skills help on unrelated domains rather than interfering.

**Read the ablation against the paper's own framing, though.** Removing
skill crystallization costs less than flattening the memory hierarchy
(IR 26.15 → 20.00 vs → 10.77). The dominant effect is the governed
L1/L2/L3 substrate; the skill layer is a real but secondary increment.
The honest lesson for this concept is that **skill quality is
downstream of memory organization** — a library curated well at the exit
cannot rescue procedures induced from an ungoverned trace store.

Caveat on scope: MSCE measures accumulation and transfer over bounded
runs and reports no variance anywhere. It does not test what this
concept most cares about — what happens when skills go stale, or when
the library grows large enough that retrieval interference sets in.
Whether admission gating and consolidation/retirement are substitutes or
complements is open.

## The admission rule, made testable

This concept has been source-rich on acquisition, composition, retrieval,
and consolidation, and thin on the one decision that determines whether a
library degrades: **what earns a place in it.**
[[literature/papers/shang2026hypothesis]] (HDSO) states that rule in a form
that can be executed rather than asserted.

A candidate skill must declare what behavior it will change, when it
applies, what evidence motivated it, what risks it introduces, and **what
observations would falsify it**. It is then tested *prospectively and
paired*: run the current repository *S* against *S ∪ {candidate}* on the
same task indices, and promote only if the behavior delta supports the
proposed mechanism. The promotion gate logs success delta, step delta, and
invalid-action delta — and deliberately declines to make the
evidence-strength statistic a hard threshold, because the question is
whether the *mechanism* held, not whether a number cleared a bar.

Two design choices are worth copying independently of the rest:

- **The unaugmented path stays live.** The executor consumes skills through
  progressive disclosure (compact cards first, detail on request) and the
  executor-only path is preserved when no skill is selected. The library
  can therefore only help by being invoked, and its absence is a permanent
  control rather than a condition someone has to reconstruct.
- **Rejected hypotheses persist as auditable negative evidence** in a
  ledger, so later cycles do not rediscover the same dead ends. This is
  [[concepts/typed-claim-partition]] applied to procedural memory, and it
  is nearly free — the evidence already exists once validation ran.
  Compare this project's own `/curate` discipline, where a decline with a
  recorded reason is a curation decision rather than an absence.

Measured on ALFWorld: +6.9 Avg. SR for Qwen3-8B, +4.0 for Qwen3.6-27B, and
+7.1 preserved under 20% flipped success/failure feedback. Treat the
magnitudes cautiously — the noisy-feedback gain exceeding the clean one,
with no variance reported, suggests run-to-run spread comparable to the
effect. The *design* is what to import, not the numbers.

Open, and relevant to how this concept should be applied: HDSO validates
promotion on the same task indices it reports gains on, so its admission
signal and its headline metric are not separated. A library whose admission
gate is tuned on the tasks it is then scored against will drift the way
[[concepts/hce-evaluation]] describes. Skill promotion needs a holdout too.

## The other gate: what leaves

Admission is only half a lifecycle, and this concept has been weakest on
the other half — what a library *removes*. [[literature/papers/hu2026skillbrew]]
supplies a criterion. Its framing is that existing work judges skills **in
isolation** and reduces curation to one scalar, leaving bank-level
properties unexamined; it instead optimizes **diversity** and **coverage of
the query distribution** on a Pareto frontier, subject to a minimum
**utility** constraint. The asymmetry is the point: usefulness is a
constraint that must hold before organization is worth optimizing, so
tidiness never buys itself at the cost of measured performance.

Mechanically: per-skill **counterfactual leave-one-out replay** produces
KEEP / REWRITE / REMOVE evidence for each member, and a bi-level
propose-then-verify loop proposes edits from trajectory evidence on a
**support split** and verifies them by Pareto selection on a **held-out
query split** — explicitly so that edits are not validated on the
trajectories that motivated them. Against ten training-free baselines on
ALFWorld and WebShop it beats append-only Voyager by 12.0%, and curated
banks transfer across worker models of different scales.

Together with HDSO the lifecycle's two gates are now both covered by
sources, and they compose rather than compete:

| gate | question | mechanism |
|---|---|---|
| admission ([[literature/papers/shang2026hypothesis]]) | should this candidate enter? | falsifiable hypothesis + paired control/treatment |
| retention ([[literature/papers/hu2026skillbrew]]) | given a bank, who stays? | leave-one-out credit + Pareto selection under a utility floor |

Worth noting that only SkillBrew separates the split it optimizes on from
the split it verifies on. A library whose gates are tuned on the tasks it
is then scored against drifts exactly the way [[concepts/hce-evaluation]]
describes — **skill promotion needs a holdout too**, and this is the
clearest open gap in the cluster.

The standing caution on SkillBrew's own framing: coverage of the query
distribution presupposes a known, stable query distribution. For a research
agent the distribution is what shifts, so a bank optimized for yesterday's
coverage is a different failure mode from an append-only log, not
necessarily a smaller one.
