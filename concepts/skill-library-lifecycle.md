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
  - "[[literature/papers/zhan2026auto]]"
  - "[[literature/papers/wu2026evomal]]"
  - "[[literature/papers/esakkiraja2026starharness]]"
  - "[[literature/papers/yu2026recursive]]"
  - "[[literature/papers/badhe2026skill]]"
  - "[[literature/papers/tang2026wikiskill]]"
  - "[[literature/papers/lu2026credo]]"
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/kassis2026scientific]]"
  - "[[literature/papers/chen2026repo]]"
  - "[[literature/papers/brueckner2026kbench]]"
  - "[[literature/papers/piriyakulkij2026subagents]]"
  - "[[literature/papers/li2026autorecsys]]"
  - "[[literature/papers/kapner2026scanning]]"
  - "[[literature/papers/gao2026agentic]]"
  - "[[literature/papers/suresh2026grounding]]"
  - "[[literature/papers/hu2026analyzing]]"
  - "[[literature/papers/cheng2026scope]]"
  - "[[literature/papers/yan2026traceml]]"
  - "[[literature/papers/zhang2026veriharness]]"
  - "[[literature/papers/zhang2026who]]"
  - "[[literature/papers/lumer2026reflective]]"
  - "[[literature/papers/ding2026harnessdelta]]"
  - "[[literature/papers/pham2026reliable]]"
  - "[[literature/papers/giampieri2026cortex]]"
  - "[[literature/papers/he2026does]]"
  - "[[literature/papers/jiang2026researchfloweval]]"
  - "[[literature/papers/liu2026lost]]"
  - "[[literature/papers/zhong2026automating]]"
  - "[[literature/papers/xu2026stale]]"
  - "[[literature/papers/razmara2026one]]"
  - "[[literature/papers/wang2026beyond]]"
  - "[[literature/papers/so2026masked]]"
  - "[[literature/papers/zhou2026does]]"
  - "[[literature/papers/nakayashiki2026lessons]]"
  - "[[literature/papers/treude2026how]]"
  - "[[literature/papers/mitra2026agent]]"
  - "[[literature/papers/gao2026trustworthy]]"
  - "[[literature/papers/zhang2026rsi]]"
  - "[[literature/papers/mehtiyev2026when]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/shared-substrate-contagion]]"
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

## A single-lesson admission gate with per-context non-inferiority is a screen: its one held-out-tested winner did not replicate, and it came from the writer that never saw a failure (2026-11-16)

The 10-19 section above admits lesson *blocks* by randomized inclusion
(pham2026reliable). [[literature/papers/mitra2026agent]] applies the same
design to *single* lessons and adds two harm terms. The lessons are written
for a frozen Qwen3.5-9B by a larger teacher, as "WHEN … THEN" notes in one
system-prompt slot. Each candidate is switched on or off at random across
many runs. A hierarchical Bayesian model pools contexts (τ² task ×
decision type; BFCL domain family). A lesson passes only with all of:
- a credible gain > 1 point;
- enough exposure in every context;
- P(loss > 1 point) < 0.10 in every context (non-inferiority);
- bounded harm (τ²: wrong writes; BFCL: wrong final states);
- on τ², all of this under sceptical, weakly informative and slightly
  negative priors.

What it did:
- **τ²: 2 of 12 admitted, on the selection data.** "Change the address
  first" +4.7 [+4.0, +5.4] and "list only the first change" +2.7
  [+2.0, +3.4] (90% CrIs, 20,400 decisions). **Four lessons pushing toward
  asking or confirming again were rejected,** each because it credibly
  hurt at least one context. This complements the 11-02 section, where
  52% of optimizer-added rules ask the agent to verify. Here a per-context
  harm term is what kept such rules out. No held-out test was run.
- **BFCL: 0 of 8 trace-written lessons passed.** The one that passed came
  from the no-evidence writer: "when the current directory is unknown,
  call pwd", +4.9 [+1.9, +8.0]. On 120 held-out roots it gave −0.31
  [−3.65, +3.12]. The gate estimate lies outside that interval. Running
  all eight trace lessons without the gate gave +0.52 [−3.33, +4.38] over
  the standard harness (sign flipped from the paper's
  standard − lessons). Both are null on held-out, so the gate's selection
  added nothing measurable there.

Three points for this concept:
- **Admission on the selection split is a screen, not a certificate.**
  The gate's design is stricter than any in the 10-19 sections. It still
  produced a winner that shrank to zero on fresh roots. Pham's design
  estimates blocks on a practice split and evaluates on a separate one.
  The held-out step is what this gate needs, and it is what showed the
  shrinkage here.
- **An admitted lesson's provenance is not its source.** The first τ²
  lesson states the rule the no-evidence writer also wrote "almost word
  for word". The BFCL winner was not trace-written at all. A library that
  records "learned from failure X" for an admitted lesson can be wrong
  about why it is there. Compare treude2026how (11-03 section), where
  proposals came from the skill's checklist rather than the session.
- **"None" should be a neutral text, not only an empty slot.** On
  held-out BFCL a fixed note with no procedure scored +4.5 [+0.7, +8.4]
  over the standard harness, more than either lesson set (descriptive,
  unadjusted). Pham's "keep the empty library as a scored candidate"
  (10-19) generalizes. Score a content-free note of similar length too, or
  a library's gain may be the presence of a note rather than its lessons.

A fourth, weaker point extends "Fired is not the same as effective" (11-03
section) one step further. The τ² lessons cut the targeted error from 6
conversations to 0, and pass^1 did not move (−1.3 [−5.9, +3.3]). A lesson
that fires and changes the behaviour it targets has still not been shown
to change the outcome.

Limits:
- One frozen 9B model with thinking off, one teacher family; customer
  service and function calling, not ML research.
- The τ² writers' task descriptions stated the target rules, so the τ²
  gate screens phrasings of known rules, not discovered ones.
- The BFCL search writer read the function documentation and the trace
  writer did not.
- The τ² gate data were re-run after a data loss, and BFCL ran at
  temperature 0.7, not the official 0.001. 146 of 240 held-out BFCL
  instances were audited as defective; on the 18 clean gate instances the
  winner's estimate was +3.9 [−0.4, +8.3].
- The hierarchical model's specification and the rejected lessons' texts
  are in an extended report available only on request.

## A noninferiority admission gate admits edits that change nothing, and its ratio guarantee loosens later thresholds with each admission (2026-11-16)

The admission sections above ask that a candidate show a positive
estimated gain (tang2026memory), keep "none" as a scored candidate
(pham2026reliable), and pay for the turns it adds (11-02).
[[literature/papers/gao2026trustworthy]] (TRUST) is a formally guaranteed
gate over exactly these edit types: adding, refining, generalizing,
promoting and deleting reasoning rules; adding up to three closed cases
to memory or archiving them; and routing and workflow switches. It
admits on a different criterion. The test is **noninferiority**: the
null is "the edit loses at least ε", with ε = 0.10 on a [0, 1] score. The
guarantee bounds the share of admitted edits that lose more than ε.

What that means for a library (our arithmetic and extension):
- **A tie is evidence for admission.** The null boundary is
  p_ε = (1 − ε)/2 = 0.45, so an item where the edit changes nothing
  (Y = 0.5) raises the e-value. Forty ties give the paper's betting
  mixture an e-value of about 9.5, or about 5.5 once most of the pool has
  been seen.
- **Each admission lowers the bar for the next.** With the LLM multiplier
  at 1, the budget recursion gives α_t = (q/T)·(R + 1) = 0.01·(R + 1),
  where R counts prior admissions. The first threshold is 100, which a
  no-op cannot reach. After about 10–18 admissions, or with a multiplier
  of 3 after a few, it can.
- **The guarantee cannot see what accumulates.** FER is a ratio, so more
  admissions also enlarge its denominator. Theorem 4 makes the cost
  explicit: each admitted edit may lose up to ε with no charge to the
  error budget. That is fine for a one-incumbent prompt. For a library
  that grows by admission, it lets do-nothing rules in, and the 11-02
  section prices those in turns, not in score.

So a margin buys power (TRUST at ε = 0 admitted nothing, Table 8), and the
price is the write-side "none" control that pham2026reliable and
nakayashiki2026lessons found does most of the work. Our rule, untested: if
an admission gate uses a loss margin, a noninferior candidate must still
beat "no change" on a cost axis (turns or tokens per task) before it
enters.

A second datum is the verbatim-memory route, under Greedy only (App. I).
With 20 examples reused for proposing and validating, an accepted memory
entry stored a validation question with its verified answer. On a test
question, a shared keyword ("album") retrieved it. Routing then dropped a
needed paragraph, and Joint F1 fell from 54.55 to 48.00 (13 items down,
none up). Validation gain was +24.44 pp while test gain ended at −2.15.
This is "an optimizer that can write anything will write the answer key"
(10-13), here through memory retrieval rather than rule text.

Limits:
- The paper reports no acceptance breakdown by operator or component, so
  how many admitted edits were rules, memories or no-ops is unknown. Its
  low empirical FER0 (ties count as failures there) suggests few no-ops
  were admitted in these runs.
- One incumbent harness, not a library with retrieval. The noninferiority
  argument is about the gate's design, not an observed library effect.
- App. I has no TRUST arm. The tie arithmetic assumes the paper's fixed
  betting mixture. ρ_max and α_max, which cap the LLM's effect, are not
  reported.

## Admission by validation can cost more than serving the stream, and one unvalidated bounded note did as well in one run (2026-11-17)

The 11-02 section (zhou2026does) prices an admitted rule by the turns it
causes. [[literature/papers/zhang2026rsi]] prices the *admission process*.
EvoSkill's frontier with validation-based selection ran on two models and
four streams, one run each:
- Validation was 78.1% of EvoSkill's 2,103M self-improvement tokens;
  writing the skills was 0.2% (4.59M).
- On DeepSeek/terminal, re-scoring the 3-program frontier per support
  window cost 497M, against 182M to serve all 89 tasks.
- A single note, rewritten from failed traces every four tasks, capped at
  2,048 tokens, always loaded and adopted without validation (SRN), matched
  or beat EvoSkill's observed accuracy in all four settings (margins 8, 1,
  6, 0 tasks) at 0.04–0.36% overhead.

Three points for this concept:
- **The paper's "subsequent use" is Cortex's "fired" (10-19) in token
  units.** 37.9% of EvoSkill's tax and 63.6% of ReasoningBank's went to
  skills or memories that never entered a later context. One ReasoningBank
  bundle went unretrieved through 34 remaining tasks. SRN's 100% use holds
  by construction, since its note is always injected. Exposure is not effect
  (treude2026how, 11-03); the authors make the same point.
- **The bounded note has no admission gate and failure-only routing.** That
  is the write policy nakayashiki2026lessons (11-03) showed turns chance
  failures into rules. The prompt hedges it, but nothing measures it. No
  content-free-note control was run. mitra2026agent's fixed note (+4.5 on
  held-out, 11-16) means SRN's +4.7 pp over Static may partly be the
  presence of a note.
- **"Keep the empty library" (10-19) gains a cheap neighbour:** a bounded,
  replace-not-append note, compared on total tokens.

Rule (our inference): an admission gate's validation runs are charged
against the expected number of later tasks that will load the item. A
gated library is reported against (a) no library, (b) one bounded note
and (c) a content-free note of the same length, all on total spend.

Limits:
- One run per cell, no CIs, the authors' own baseline. EvoSkill ran at one
  iteration, with a rolling refresh its original lacks and no ground truth.
- 15-task chains reset state, so a skill has ≤12 later tasks to repay its
  validation. No code or traces released.

## For a stale instruction line, correcting beats deleting, and no agent repaired the file unasked: 0 of 732 runs (2026-11-17)

The 2026-10-26 section "Staleness in an always-loaded instruction file is
quiet" ([[literature/papers/xu2026stale]]) left the removal gate with two
gaps. It had no deletion arm, and its proposed repair signal was the
agent's own conflict report. [[literature/papers/mehtiyev2026when]] fills
both, on real repos, with Claude Code (Haiku 5.5, Sonnet 5.5). Three real
stale lines (a test command, a code location, a procedure) were each
tested over 22 bug-fix tasks as stale / corrected / removed: 732 runs.

- **Correct, don't delete.** Stale-path use was 78/80 (command) and 55/80
  (procedure) with the stale line, 0 with the corrected line, and 1/80 and
  20/80 with the line removed. Without the line, agents rebuilt the old
  folder from the code's layout. Sonnet's effort on the command case was
  the same removed as stale (7.8 calls vs 5.8 corrected). Deletion had
  lower median effort in only one unresolved cell (Procedure pooled: 8.9 vs
  9.6 calls). Fix rates were flat everywhere (p = 1.0).
- **The repair signal mostly never forms.** Agents recovered in 130/133
  encounters, mostly by searching (98). Only 21/133 final replies named
  the mismatch: Sonnet 17/40 on the command case, 0/31 on the procedure
  case. 0/133 proposed a repair, and 0/732 edited an instruction file.
  Haiku noticed *other* stale lines in 22/120 procedure runs and left them
  "because it was not part of the bug". That is scope discipline, not
  inability.
- **Stale lines survive edits.** The three lines outlived 2, 13 and 4
  later edits of their files. An edit is not a review.

Rule (our inference): when a removal or revision gate here finds a stale
path or command in CLAUDE.md, `.claude/rules/` or a skill, the action
should be *rewrite to current*, not *delete*. Deletion leaves the agent
to rediscover the old structure. Harvesting the agent's conflict reports
(the 2026-10-26 proposal) will catch convention conflicts more often than
dead paths, because a dead path that is routed around goes unreported
in 112 of 133 encounters.

Limits:
- The tasks asked for a code fix only. Whether an agent *told* it may fix
  the instruction file would do so is untested. The authors say this is
  default behaviour, not behaviour when asked.
- Three cases, with instruction kind, repo and language confounded; one
  vendor, one harness; easy single-bug tasks.
- The maintainer's cost of finding the correct line, which is where
  deletion is cheaper, is not measured.

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

**Second attestation, from a different group and benchmark.**
[[literature/papers/piriyakulkij2026subagents]] reproduces the shape on
SkillsBench with Qwen3.5-9B (accuracies read from its bar charts, ≈±0.01).
With every synthesized skill exposed flat, inline accuracy is ≈0.22. Lazy
loading through a two-level task hierarchy (pick the task node, then see
only its skills) raises it to ≈0.50, matching the zero-distractor condition
(≈0.48). In a separate sweep, inline accuracy drops monotonically as
unrelated skills are added — gpt-5.4-mini ≈0.80 → ≈0.55 at 263 distractors.
This is independent of kim2026why (Alberta/Amii, MLE-Bench), but it shares
both of that source's limits: the flat baseline is again dump-everything
rather than retrieval at matched context size, and the best hierarchy
groups skills by their source task, which is close to oracle scoping that a
novel task would not have. LLM-built trees did worse, and depth 3 was worse
than depth 2.

The same paper adds a write-side property this concept has not named: **a
skill's interface decides how it can be executed.** The descriptions that
let a skill run as an isolated subagent state an "Expected input" and an
"Output"; curated skills without them did no better as subagents than
inline, and worse for strong models (see
[[concepts/hierarchical-delegation]]). So the skill format is not neutral
about execution. Its synthesized skills were also distilled from successful
trajectories on the very tasks they are scored on — the holdout gap flagged
below.

**Third attestation (2026-09-29), with a different mechanism: interference,
not dilution.** [[literature/papers/cheng2026scope]] shows that scope matters
even when nothing is diluted. There is one 360-character skill and the same
prompt budget either way. A frozen model edits the skill after each round of
a 12-round code-repair stream over nine recurring task families, and a
paired, execution-grounded gate (ORC) admits or rejects each edit. Every
admission passed its same-family probes. Deployed as *the* global skill,
though, the round-0 rule scored +0.398 on its own family and −0.461 /
−0.234 / −0.148 / −0.117 on four others. The gated agent ended below
never updating: 0.713 vs Static 0.775 mean hidden trajectory utility over
8 streams. The same eight edits, stored only under the family that produced
them, give 0.816, with 0 harmful instead of 6. A prespecified 27-stream
randomized-order extension reruns everything end to end and finds scoped
minus global = +0.063 [0.037, 0.094]. Scoped memory makes 63 admissions to
global's 12, 19/27 streams keep admitting (vs 2/27), and harmful/admitted is
0/63 vs 6/12.

What this adds is a **coupling rule between the admission gate and the
loading function.** An admission gate certifies an edit only for the scope
its evidence sampled, and loading must not deploy the edit wider. The
admission sections below (tang2026memory, shang2026hypothesis) treat
admission as yes/no. This source makes it yes/no *plus where*, and argues
that the "where" should be recorded at admission rather than rediscovered
by retrieval. Limits, ordered by how much they matter for import:

- **"Global" is a single-slot overwrite.** Each admitted edit replaces the
  whole skill for every family. Scoped keeps nine slots. An additive
  library, where a new rule is one entry among many, is untested.
- **The scope key is free here.** Families carry opaque tokens and
  templated prompts, so a character n-gram router scores 243/243.
  Wrong-slot routing is untested. For real tasks, choosing the key is the
  hard part (piriyakulkij's LLM-built trees did worse than source-task
  grouping).
- **Scoping forbids positive transfer.** That is in tension with
  tang2026memory's cross-domain gains on all six pairs. "Retrieve where
  certified, widen on cross-scope evidence" is the untested middle.
- **Some of the effect is structural, and the world is small.** A scoped
  edit's non-current-family effect is 0.000 by construction. The 8-stream
  0.816 recomposes archived outputs. The 18-stream single-update
  replication is not significant on its own (0.041 [−0.004, 0.107],
  p = 0.1875). ORC itself did not beat accept-everything (Holm p = 1.0).
  There is one model and nine code-contract families.

For this box, `~/.claude/` user-global skills, the global `CLAUDE.md` and
auto-memory are the global slot, and project `CLAUDE.md` plus path-scoped
`.claude/rules/` are the family slots. Promoting a lesson validated in one
project straight to the global layer is the Global-ORC move. Record where a
rule was validated when promoting it.

## Which clauses of a human-distilled skill transfer (2026-10-05)

[[literature/papers/yan2026traceml]] probes a ~1k-token planning skill built
from human Kaggle practice, clause by clause.
- A clause that names a level the agent has not reached transfers: ensemble
  re-weighting fell from 50% to 11%, against a human 8.9%.
- A prohibition overshoots: plain hold-out went from 10% to 0, against a
  human 24%, "a direction with no destination".
- A prescription the agent already exceeds does nothing.

That gives three admission tests for a skill clause: it states a target
level, the agent is below that level, and it is more than a ban. The score
evidence is one run per competition.

## Failure-driven evolution of a verifier's skills, with a held-out split (2026-10-06)

[[literature/papers/zhang2026veriharness]] evolves a *verifier's* skill
library from development failures. Each failure package holds the task, the
evidence record, the delivered artifact and the grader's per-item verdicts.
The admission protocol is worth copying:

- tasks sharing a workspace or template are grouped before a ~75/25
  dev/held-out split;
- a candidate library replaces the current one only on a non-negative
  paired dev gain;
- a filter rejects task IDs, benchmark names, dev filenames and reference
  values;
- round schedules are fixed before any held-out score is inspected;
- held-out grades are never fed back to the proposer.

Held-out results (Opus 4.8, one run each, no significance testing):

| Benchmark | Empty library | Human-authored | Evolved from empty | Evolved from human |
|---|---|---|---|---|
| APEX-Agents | 38.6 | 45.3 | 49.6 | 52.1 |
| SpreadsheetBench 2 | 32.8 | 35.7 | 38.5 | 39.4 |

The paper compared all 95 evolved checks with the human library by hand:

| Relation to the human library | Count |
|---|---|
| Restates a human check | 5 |
| Makes a human check concrete (script, constant, document type) | 48 |
| New | 42 |

So evolution mostly specializes a general check. Two caveats: both
trajectories regress on held-out tasks along the way, and the
"human-authored" baseline was itself written from development failures.

## A vacuous grader trains the library as well as a valid one (2026-10-12)

[[literature/papers/zhang2026who]] puts a self-evolved verifier in front of
the authors' Ratchet skill loop. The verifier decides which attempts become
failure capsules for skill synthesis. With the verifier's guards removed it
collapses to always-pass, yet held-out peak scores match or beat the
anchored arm on all three tasks: 0.742 / 0.458 / 0.841 vs 0.717 / 0.458 /
0.812. By our arithmetic it retains 100 / 100 / 98% of the ground-truth
loop's lift. On Spider the anchored verifier sat at chance (0.500) and the
loop still kept its lift. The paper's reading is that the verdict is only
"directional". The capsule's content (tracebacks, empty results) comes from
the environment, and measurement and peak selection read the locked ground
truth. This qualifies the 2026-10-06 section above. There, an evolved
verifier sits on the *keep* path and its accuracy is the result. Here it
sits on the *training-input* path, where the loop barely uses it. So the
admission and retirement gates this concept recommends need their verdicts
validated directly. A rising library score cannot show the admitting grader
works. The paper also reports that the lifecycle (birth gate, shadow tier,
retirement) is not load-bearing for a *detector* pool. Junk detectors stay
inert unless selection picks them, whereas junk skills get routed into
prompts. Caveats: three seeds, one model family, and the skill loop is the
same group's own.

## An optimizer that can write anything will write the answer key (2026-10-13)

[[literature/papers/lumer2026reflective]] gives the admission gate a
concrete failure to block. GEPA, rewriting a deep-research agent's system
prompts from training traces, added "Known Benchmark Facts" blocks that
pair verbatim training questions with their answers and source URLs.
These blocks entered at the first accepted edit. They reached 52% of the
selected orchestrator prompt and 72% of one rejected researcher prompt.
The key earned no reward even on the questions it encodes. It still
inflated the prompt, and it would be an active hazard in a domain where
the same entities recur.

The authors name two guards they did not run:
- [[literature/papers/yang2026skillopt]]'s update rule, which bounds
  edits, forbids hard-coded task-specific values and gates on held-out
  validation;
- a token-overlap novelty penalty between candidate text and training
  questions, applied at scoring time.

Both apply to any text artifact a loop writes, skill files included. The
cheap version for this project is a lint that flags verbatim task-specific
strings (answers, IDs, dates) in an evolved skill before admission. The
held-out split only discounts such content after the fact.

## A bounded, validation-gated optimizer still writes the benchmark's regularity (2026-10-19)

The section above cites [[literature/papers/yang2026skillopt]]'s update
rule as a guard against answer keys: bounded edits, no hard-coded
task-specific values, and acceptance gated on validation.
[[literature/papers/ding2026harnessdelta]] runs SkillOpt and shows the
guard does not cover the next class up. On LiveMath, the skill learned
"The meta-option is your working default … otherwise, the meta-option
wins". That rule names no answer and no item. It encodes a construction
regularity that holds across the search, validation and test splits, so
the validation gate passed it. Its audited overfitting share is O = 88%.
On ALFWorld, SkillOpt wrote appliance recipes, location priors and a rule
distilled from one failed search episode ("this is exactly how the lettuce
task was lost"), O = 44%.

A second route: when the proposer is stronger than the executor (Opus 4.8
writing for Qwen 0.8B–35B), it can distil its own knowledge of the
benchmark into the skill. A same-benchmark holdout then rewards that
knowledge.

For admission:
- the verbatim-string lint proposed above is necessary but not sufficient;
- also flag clauses that state a default answer, a fitted threshold or an
  environment-specific prior, then disable each one and rescore.

Caveats: the "overfit" labels come from an LLM audit with human review
and no measured precision. Some flagged clauses ("do not repeat an action
that returns 'Nothing happens'") read as general rules. Each setting is
one run.

## Admission by randomized inclusion: measure blocks, and keep "none" as a candidate (2026-10-19)

[[literature/papers/pham2026reliable]] (RBC) estimates every lesson
group's causal effect from one experiment. It does not test candidates
one at a time. Lessons are clustered into 8 blocks. On each practice
episode every block is included independently with p = 1/2, and a ridge
regression with one intercept per task gives each block's average
marginal effect. The estimate is unbiased whatever the interactions, and
it needs 4.5× fewer episodes than leave-one-out for the same precision.
That is a cheaper form of hu2026skillbrew's per-skill leave-one-out
credit and of shang2026hypothesis's S vs S∪{c} pairing. It also answers
this note's "skill promotion needs a holdout too": all selection runs on
a practice split, and evaluation tasks are never scored by any method.

The decomposition is the transferable part. Over 19 settings, curating
one run scores 61.6. Choosing among 3 growth runs plus "no playbook" on
practice episodes (Best-Full, RSEA's rule) scores 66.4. Both together
score 66.9. **Keeping the empty library as a scored candidate does most of
the work.** Block curation adds about 1 point on ACE, nothing on Dynamic
Cheatsheet (rewritten lessons are not separable), and +3.1 when ACE
playbooks grow long. A 40% token cap beat serving every positive block
(65.2 vs 63.8), so measured-positive is still not enough to serve.

Caveats: there are no CIs on the 0.5-point margin over Best-Full. It
needs about 2,800–3,250 practice episodes per setting on 57–300
comparable practice tasks. The unit of admission is a cluster of 10–21
lessons, not one lesson. Playbooks are frozen once grown, and the models
are 9–31B open models.

## One exposure rule for admission and removal: never-fired means unmeasured (2026-10-19)

[[literature/papers/giampieri2026cortex]] (Cortex) applies one rule to both
gates. It compares the library with and without an item only on tasks where the
item entered context (a rule loaded, or a skill was listed), and unknown
exposure blocks the verdict.

- **Admission (`/evolve`)** is shang2026hypothesis's paired S vs S ∪ {c},
  scored by a script at k = 3 runs per task and arm. A candidate is kept only
  if it has a positive gain, at least 2 runs won net, no task losing 2 of its
  3 runs, no always-passing task losing any run, and at least one rollout where
  it fired (rule loaded or skill opened).
- **Removal (`/prune`)** reverses the arms. An item that never fired gets
  UNMEASURED and stays.

This extends SLIM's exposure guard (shen2026dynamic), which protects only
retirement, to admission. It also takes the "removal is not internalization"
warning above in the other direction: absence of use is absence of evidence,
not evidence of no value.

What the exposure gate demonstrably did:
- In a known-answer test, 10 candidates showed a positive screen gain without
  ever firing, 9 of them from a single run on a single task. The fired gate
  rejected them.
- 0/20 placebos and 0/5 harmful items were kept.

What it did not do:
- No placebo or harmful skill was ever opened, and harmful rules were screened
  only on failing tasks. The regression gates never met harm that loads.
- Only 2 of 4 known-good items were kept.
- `/prune` kept a placebo that was opened 0 times, because 2 tasks lost runs at
  k = 3 when it was removed.

Two qualifications for this concept:

1. **The holdout cannot see scope errors.** Cortex has the holdout this cluster
   lacked: 30 never-played scenarios, +55.0 [+34.1, +74.1] over no harness
   (Haiku 4.5, 4 runs). But they test the same four conventions, and a gate on
   harvested tasks cannot see scope errors on new ones. R3's money rule,
   narrowed to 5 named files, passed every gate and then loaded in 0 of 30
   held-out rollouts. This is cheng2026scope's coupling rule from the other
   side: an admission certifies scope only where its evidence sampled, whether
   the scope is too wide or too narrow.
2. **A model change is a retirement trigger.** Value tracked how often the model
   broke the convention. The same harness added +5.6 [−2.8, +13.9] under
   Sonnet 5, and one `/prune` pass after the upgrade flagged 2 of 4 items.

Single author, synthetic repo, Haiku in every role, no harm test.

## Failure-driven revision rarely removes the rule that causes the failures (2026-10-20)

[[literature/papers/he2026does]] plants three erroneous rules in a
playbook and lets the model revise it from its own failures. The result
bears on whether failure-driven update can serve as the removal half of a
library's lifecycle. Mostly it cannot.

- **Editors blame non-compliance, not the rule.** None of the 18 coded
  diagnoses of rule-caused failures said a rule was wrong. For the
  "write counts in words" rule, 7 of 12 diagnoses mentioned it, and 6 of
  those said the agent had *not followed* it, although the answer ("five")
  had followed it. None of the 12 children neutralized the rule, and two
  stated it more firmly. With the format restrictions lifted, it was
  neutralized in 1 of 11 (vs 0 of 11).
- **Misapplication framing works when it fits.** For the rounding rule, 5
  of 6 diagnoses said it was applied where not needed, restricted it, and
  repaired the target. Scoping the rule down is what failure reflection
  found, not deleting it.
- **A rule that causes no failure is never examined.** The inert
  navigation rule survived 27 of 28 failure-driven revisions; all 4
  children given no evidence removed it while rewriting.

The removal mechanisms already in this concept do not depend on the editor
diagnosing the rule: hu2026skillbrew's counterfactual leave-one-out,
pham2026reliable's randomized inclusion, and giampieri2026cortex's paired
with/without comparison. This paper is evidence for keeping it that way.
A rule or skill that the agent is visibly following looks, to a reflective
editor, like a rule it should follow better. The inert-rule result is the
reflection-side twin of Cortex's "never fired means unmeasured": no
failure, no pressure, no removal.

Rule: retire skills and instruction clauses by ablation against a scored
holdout, not by asking the agent to reflect on its failures.

Caveats:
- Qwen3-4B as both agent and editor, artificial rules by design, 18
  rule-caused cases coded by one coder (the author).
- The revision prompt told the editor to keep final-answer format rules,
  which may have protected the number-words rule; the small control (11
  per arm) does not estimate that effect precisely.

## Skill optimizers only add rules, and a rule's cost is the turns it causes, not its length (2026-11-02)

The section above found that failure-driven revision rarely removes a rule.
[[literature/papers/zhou2026does]] measures the same thing at optimizer
scale, and prices it. GEPA, SkillOpt and SkillAdam were run from one
580-word seed. 58 of 59 DeepSeek edits lengthened the skill (33 of 38
under Luna), and one shortened it. Skills grew to 654–2.4k words. Of 337
added rules, 52% ask the agent to verify, 18 serialize calls, and none
limits reading. SkillOpt can delete and replace, and it still grew.

The cost lands in turns:
- Across 59 edits, the change in turns explains 93% of the variance in
  token change. Tool calls explain 79%, and skill length 10%.
- Turn terms carry 73% of the extra input tokens; skill text 21%.
- Each turn re-sends the history, so 10% more turns read 14–16% more
  tokens.
- Loading on demand did not help (ρ 1.230 vs 1.226 resident).

This reframes pham2026reliable's 40% token cap. A cap on a skill's *length*
prices the smaller term. What should be charged is the turns the rule
induces, measured per task against the unaided run. Learned skills
"sharpened" the agent (agreement rose where success rose) but covered 57
new tasks and lost 62. Their cost went with them to other models and
benchmarks, and their gains did not (8 of 9 transfers below the seed).

Rules:
- Admission charges a candidate rule for the turns and tokens it adds per
  task. Only 5 of 59 edits kept success without adding tokens.
- The editor is shown earlier rules' costs and may delete and merge. This
  is the paper's Principle 3–4, untested.

Limits:
- One seed, itself a verification skill, which may bias edits toward
  "verify".
- One optimizer run per cell; SkillOpt at one pass of its upstream four.
- The 93% figure is across edits of one model (DeepSeek-V4.1-Flash).
- No-compaction harness. Under re-billed compaction, ρ is still 1.05–1.36.

## Two research skill packs tied the bare harness by trading evidence for write-up (2026-10-20)

[[literature/papers/jiang2026researchfloweval]] gives a no-skill arm for
two off-the-shelf research skill packs. ARIS and Orchestra each run on
Codex/GPT-5.5 with one added instruction line. They are compared with
Codex/GPT-5.5 alone, on the same task, budget and judge.

- Overview: 44.5 and 45.2 against 45.1.
- Paper writing: 52.5 and 47.5 against 32.5.
- Result quality: 20.0 for both, against 30.0.
- Claim completion: 32.5 and 37.5 against 42.5.
- Met claims: 1 each, the same as the bare harness.
- Self-reports: each pack reported 8 claims attained within half an hour.
  The bare harness reported 0.

The overall tie hides opposite-signed shifts. The packs moved effort into
the write-up and into confident self-reports, not into evidence. This is
the empty-library candidate that [[literature/papers/pham2026reliable]]
says must always be scored, and here it ties a full pack. It adds one
point to that note: score a pack on the dimension it claims to improve,
not on the aggregate.

For this box it is weak support for scoping research skill groups to the
projects that need them. A general pack loaded everywhere can raise how
finished the output looks without raising what it establishes.

Caveats:
- One run per arm, on one task.
- The bare harness itself stopped after 0.30 h.
- The skill arms ran on a different host.
- Which skills actually fired is not reported, so
  [[literature/papers/giampieri2026cortex]]'s exposure rule cannot be
  applied.

## Large libraries suppress loading through description interference, and matched-size preselection recovers it (2026-10-20)

[[literature/papers/liu2026lost]] holds SkillsBench tasks, verifiers and
golden skills fixed and adds 0 to 10,000 random distractor skills from a
20,000-skill market pool. The agent uses progressive disclosure: it sees
every skill's name, description and location, and decides during
execution what to open. Aggregate loaded-skill recall (82 tasks × 3
open-weight models × 3 runs) is 73.6 → 68.2 → 53.3 → 27.4% at 0 / 10 /
100 / 1,000 distractors. The zero-load rate goes 9.3 → 51.6%.

This is the fourth source on the loading function, after kim2026why,
piriyakulkij2026subagents and cheng2026scope. It adds three things they
lacked:

- **The failure is under-loading, not wrong loading.** Non-golden loads
  occur in 2.32% of evaluations. When agents do load at 1,000
  distractors, 76–89% of loads are golden. The agent stops opening
  skills; it does not open the wrong ones.
- **The mechanism is description semantics, not context budget.** On a
  10-task subset at 1,000 distractors, replacing distractor metadata with
  length-matched `<mask>` tokens raises loaded recall 18.6 → 63.9%. The
  no-distractor value is 73.5%. Masking descriptions alone gives 63.3%.
  So kim2026why's "context budget displacement" is not the main channel
  here. Plausible-sounding entries are.
- **Retrieval at matched exposed size is now tested.** The limit noted
  above was "flat vs dump-everything, not retrieval at matched size."
  Preselecting ≤5 skills before execution keeps loaded recall at 62.0%
  (LLM pick from full list) and 67.1% (hybrid: LLM-written queries,
  embedding pool, LLM pick) at 1,000. Hybrid keeps 55.2% at 10,000, with
  zero-load at 9–11%. A prompt instruction to plan skills first recovers
  only 44.6%.

One contrast with kim2026why: here even the full 1,000-skill list beats
no skills (Pass@3 29.3–43.9 vs 26.8–34.1). Flat loading hurt loading
but was not inert.

Caveats:
- Custom JSON terminal harness with all metadata in the system prompt,
  not Claude Code's `Skill` tool. Effect size may not transfer.
- Distractors are random, not topically clustered. There is no tiered or
  hierarchical arm to compare against kim2026why's tiers.
- No CIs. The mechanism arms are 10 tasks; field-level masking is 1 run
  per task and model. Code URL returned 404 on 2026-10-20.

Rule: above a few dozen skills, narrow the visible list before the task
starts (scope or preselect); do not rely on the agent to route from the
full list mid-task.

## A failure-distilled research bundle doubled held-out gains, but it was designed on the suite that scored it (2026-10-26)

[[literature/papers/zhong2026automating]] is the counter-case to
jiang2026researchfloweval's two research packs, which tied the bare harness
(above). Agent4Sci's three skills raise Opus's mean held-out gain over 20
ML-for-science benchmarks from 10.4 to 20.1% (18/20 improved) and Codex's
from 4.4 to 8.1% (12/20). The three skills are research ideation,
experiment management and an hourly checklist. Three design differences
may explain why this bundle helped where those packs did not:
- **Distilled from observed failures.** Each skill targets one failure
  seen in this suite's trajectories: novelty miscalibration, idle GPUs,
  exploration collapse. See "Distilling a skill from a failure" above.
- **Invoked, not loaded.** Ideation runs at the start, experiment
  management after it, and the checklist by hook every hour. The loading
  function that liu2026lost and kim2026why found to fail is bypassed.
- **Scored on the dimension the bundle claims.** The score is a
  sequestered test metric computed by a separate evaluator, not a
  write-up judge.

Four limits keep this from being an admission-grade result:
- **Designed in-sample.** The skills came from Claude Code trajectories
  on the same 20 benchmarks, with no benchmark held out of skill design.
  The authors flag this (A.5). The only transfer check is a second agent,
  Codex, where six benchmarks move by more than 14 points, two of them
  down (PFDelta −17.4, LITHOS −14.8).
- **No empty or partial bundle.** The three skills are never ablated, so
  pham2026reliable's per-block credit cannot be assigned.
- **One run per arm.** No-skill PFDelta alone spans 29.4–65.3 over four
  runs.
- **The benefit does not persist.** In 3-day runs with skills on, both
  agents fell back into incremental tuning or over-ensembling after the
  first day.

For this box: a skill that targets a measured failure of *this* loop and
is triggered deterministically is the shape that has positive evidence.
Admitting it needs a held-out task set and a no-skill arm with more than
one run.

## Removal judged on success alone strips redundant safeguards; retention needs a one-layer ablation on harm (2026-11-02)

The retention gates in this concept ask whether a member still earns
its place: [[literature/papers/hu2026skillbrew]]'s counterfactual
leave-one-out and the randomized-inclusion and paired with/without
checks all score *utility*. [[literature/papers/so2026masked]] shows
what that criterion does to safety clauses that back each other up. A
clause whose harm is also blocked by another clause has near-zero
leave-one-out harm (0.016 vs 0.422 when the deletion exposes the harm)
and may cost task success. Judged on utility, it looks like dead weight.
Removing it is the masked deletion, and the library's margin falls by
one without any signal.

What the paper measured, on harnesses of four written safeguards:
- Success-gated loops removed redundant clauses until protection was
  gone (greedy, 5 of 6 loops). A paired significance test stopped
  removals that only looked good by chance, but not removals that truly
  raised success (paired loops lost protection in 3–6 of 6 when the
  clause had a cost).
- Natural proposers asked to make the harness cheaper mostly *rewrote*
  clauses rather than deleting them. 7 of the 10 accepted rewrites
  relaxed a check, and rewritten clauses let through 0.19 more harm.
- The **margin-aware gate** is the retention rule that held: accept a
  child only if every one-layer ablation of it stays protected on a
  harm suite. 0 of 6 loops ended unprotected (greedy 5 of 6), with no
  loss of clean success in AppWorld, but 0.35 lost in OpsSim, where it
  kept three costly layers.

This extends the existing "verdicts are per model" point
([[literature/papers/wang2026beyond]]): which clauses are redundant
depends on the executor. Layers interchangeable for DeepSeek were not
for Qwen3-30B, which relied mainly on one, so a library pruned under
one model can be unsafe under the next.

Rule (our inference): for clauses that carry safety or governance, do
not use utility leave-one-out as the removal criterion. Score removal
on harm with the clause's siblings also ablated, or exempt such clauses
from automatic removal.

Limits:
- Four safeguards per environment, two environments, one harm each; the
  clauses are prompt rules, not callable skills.
- The margin-aware gate checks clause removal, not wording; it let a
  rewrite weaken VERIFY in all three natural loops it gated.
- 6 loops per arm; the cross-model harm is large only for weak models.

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
| admission, world-grounded ([[literature/papers/suresh2026grounding]]) | is this candidate still true of the environment? | curator re-queries the world with read-only tools before create/update |

The third row is a different kind of check. HDSO and SkillBrew test a
candidate's *utility* against task rollouts. Environment probing tests its
*claim* against the world: does this join, file location, or table name
hold on slices the source trajectory never touched, and is it still
current? That is far cheaper (no rollouts) but certifies correctness and
currency, not usefulness, so it complements the other two rather than
replacing them. It is also a second answer to tang2026memory's "don't
distill from raw trajectories": re-observe the environment instead of
governing a memory hierarchy. Its increment over trajectory-only curation
is modest (CLBench 70 → 73% pass with overlapping CIs; better than
trajectory-only in 5/6 APEX worlds), and the curator's own cost goes
unreported. See [[concepts/verified-memory-writes]] for the same mechanism
as a memory-write gate.

Worth noting that only SkillBrew separates the split it optimizes on from
the split it verifies on. A library whose gates are tuned on the tasks it
is then scored against drifts exactly the way [[concepts/hce-evaluation]]
describes — **skill promotion needs a holdout too**, and this is the
clearest open gap in the cluster.

**A holdout test of who should write the skill (2026-09-29).**
[[literature/papers/hu2026analyzing]] builds skills from analysis-split
trajectories and scores them only on held-out tasks. The holdout has two
parts: a date split within SWE-bench Verified, and a cross-benchmark
SWE-bench Pro. It compares two arms:

- **SynSkills:** ungated Trace2Skill distillation, 23–41 rules per
  configuration.
- **DevSkills:** seven principles the authors wrote, such as "reuse
  context", "persist artifacts", and "rerun only if code changed".

Both sets were preloaded into the system prompt, and the target was cost.
DevSkills cut cost robustly in 6 of 8 settings (7.88–41.73%), against 3 of
8 for SynSkills (8.86–22.32%). SynSkills come out worse than this once
synthesis is charged: after amortization only one robust reduction is
left, and the headline −22.32% cell becomes −2.70%. Both arms also shrink
on the cross-benchmark holdout.

The result sharpens this concept on two points:

- **Distilling from raw traces yields trace-specific rules.** The
  synthesized rules cover failed string substitutions and shell quoting,
  as tang2026memory predicted. That is a measured instance on a holdout.
- **Synthesis has a price to charge.** It costs 5–23% of the analysis-set
  solve cost ($4.27–$37.59). A self-authoring library has to beat the
  human-written one net of what it cost to write.

Three limits apply before reading this as "humans write better skills":

- **Rule count is confounded.** Seven principles against 23–41 rules, all
  loaded flat, is kim2026why's dilution setting.
- **The human authors had more information.** They wrote after seeing every
  configuration's diagnoses.
- **The ranking flips on Claude Code.** DevSkills win on Verified
  (−13.94%). SynSkills win on Pro (−8.86%), where DevSkills' −1.51% is
  noise.

Against validation-gated optimization, which yang2026skillopt reports beats
both human-written skills and Trace2Skill, the defensible reading is
narrower: **ungated distillation loses to human abstraction.** Gated
self-authoring is untested here.

The standing caution on SkillBrew's own framing: coverage of the query
distribution presupposes a known, stable query distribution. For a research
agent the distribution is what shifts, so a bank optimized for yesterday's
coverage is a different failure mode from an append-only log, not
necessarily a smaller one.

## What staleness looks like in production: loud regression, fast recovery

The sources above test accumulation and gating over bounded runs; none
shows what happens when a mature library's *world* changes under it —
the question the tang2026memory caveat left open.
[[literature/papers/li2026autorecsys]] (Meta, Auto-RecSys) is the first
naturalistic data point. Per-model playbooks of dead ends, pipeline recipes
and a crystallized submission config were evolved over 31 iterations on
one production recommendation model:

| Phase | Iterations | Operational fixes / iter | Zero-fix rate |
|---|---|---|---|
| Bootstrap | 1-4 | 4.0 | ~25% |
| Stabilized | 5-20 | 1.3 | ~50% |
| New baseline | 21-25 | ~4.2 | 0% |
| Post-shift | 26-31 | 0.5 | 83% |

The baseline change at iteration 21 made correct entries **wrong** —
hardware choice, package versions, entitlements — and turned six
previously positive ideas into failures. Two things are worth carrying.
First, the regression was *loud*, not silent: five iterations in a row
needed recovery, and the playbook reached better than its pre-shift level
within about five iterations. Second, the authors skip a validation gate
because dead ends are "correct by construction", since they record real
failures. Their own shift refutes that: **write-time truth does not
guarantee read-time validity** once a dependency moves. See
[[concepts/verified-memory-writes]].

A second transferable point: **structure transfers where contents do
not**. The first model's mature playbook (six fixed categories) becomes a
template whose slots are filled interactively for the next model. This is
a different axis from cheng2026agenticsts's backbone-sensitivity worry
above: here the target system changes, not the LLM. The three tiers (a
model-agnostic orchestrator skill, per-model playbook, per-iteration state
file) are a second instance of kim2026why's applicability-scope hierarchy,
but without its loading ablation.

Limits before citing: one model, n=31, an author-defined metric that
excludes implementation debugging, no no-playbook or frozen-playbook
control (so maturation and infrastructure settling are confounded), and
inconsistent dead-end counts (19 vs 49). Treat it as the shape of
staleness in the wild, not as evidence that the playbook caused the
recovery.

## Staleness in an always-loaded instruction file is quiet, and the agent may repair the world toward it (2026-10-26)

li2026autorecsys's playbook staleness was loud: a shifted baseline made
stale entries fail, and the loop recovered within about five iterations.
[[literature/papers/xu2026stale]] shows the other regime, where the stale
entry does not fail because the agent obeys it. In 40 synthetic Python
projects, CLAUDE.md or AGENTS.md describes the opposite convention from the
code (16 kinds, e.g. logging vs print, Enum vs string constants).

- **Stale text is followed, not detected.** 90.1% of choices matched the
  stale file. Nothing in the run fails. Claude Code's tests passed 80/80,
  and the only signal is prose: it named the conflict in 79/80 final
  replies.
- **Or the agent rewrites the world to match the stale text.** Codex CLI
  rewrote existing modules toward AGENTS.md in 20/80 runs (median 198 lines
  over 9 files). The restored original tests failed in 17. The agent
  reported "14 passed" on tests it had rewritten.

This sharpens the "what leaves" gate in two ways.
- **A loaded-and-followed entry can still be stale.** Exposure-based
  removal ("never-fired means unmeasured", giampieri2026cortex) does not
  catch it, since the stale line fires every session and is obeyed.
- **The removal signal already exists, but in the wrong place.** It is the
  agent's own conflict report. The paper's team recommendation is to harvest
  those reports to correct the file. Wired into a loop, that is a
  reconcile-against-world step of the kind [[literature/papers/suresh2026grounding]]
  runs at write time, here run at read time.

The paper's own fix is a precedence rule (user-level: existing code wins,
report each inconsistency). It moved code-following to 91.4% without
anyone first finding the stale line. Field prevalence is cited, not
measured: 73.8% of AI config files are never modified after the first
commit, and 23.0% of repos have stale code references in instruction
files.

Limits:
- Synthetic projects where the file is always the stale side, so the
  precedence rule's cost on intended migrations is unknown.
- Single feature-task runs.
- Sonnet 5.5 and GPT-6 Sol only.

## Merge quality is set by the merger and by the merged file's one example (2026-10-27)

Guidance 7 above treats *merge* as a consolidation operation
(zhang2026skillcomposer; pu2026skillops merges identical bodies). Both score
it by task pass rate. [[literature/papers/razmara2026one]] scores a merge
rule by rule, against code checkers and a proven-achievable reference. It
merges two same-job skills (6 rules + 1 worked example each, 16 jobs) and
has seven readers from 4B to GPT-4.1 load the result.

- **Who merges matters more than how.** With Qwen3-4B as the fixed reader,
  merges written by Qwen3-4B, Llama-3.1-8B, Qwen3-235B, GPT-4.1 and Claude
  Opus 5.5 scored 79, 87, 94, 92 and 98% of achievable rules. Claude's
  files reached 96–99% on all seven readers. A model merging for itself
  stayed 5–25 points below that.
- **Structure helps only with correct labels.** A step-by-step merge
  labels interacting rule pairs first. Claude labelled 48/48 pairs right
  with no false pairs. The other seven mergers got 2–39 of 48 right and
  invented 8–61 false pairs. Given the true labels, the weak merger's file
  gained 8–10 points.
- **The example carries most of the gain.** Removing the example from
  Claude's file costs 9–25 points on the small readers. Adding it to a
  code-built rule union adds 9–24. Across all merged files, a correct
  example adds 12–17 points and an incorrect one adds roughly nothing. The
  sources' own examples *hurt*: added to the union, they cost 0–14 points
  and bring takeover back to 32–55%.
- **A checker-fed loop is a partial substitute for a strong merger.** Six
  GEPA-style rounds lift Qwen3-4B's union +13.8 [10.4, 17.5] points to 92%.
  But the loop needs per-rule checks, which real skills lack.

For this project's write-side policy:
- A merge is a write that needs verification like any other. The paper
  proposes checking every combined rule and the example against the
  source rules before saving. This is untested, but it is the same
  reconcile-before-write shape as [[literature/papers/suresh2026grounding]].
- Our inference: the merged file's example is the part to review first.
  It is the single highest-leverage line in the file, and its correctness
  is checkable against the source rules.

Caveats:
- Synthetic author-built skills, greedy single samples, and CIs over 16
  jobs. Gaps of about 1 point should be read as ties.
- No frontier reader.
- Single-turn generation, not an agent harness.

## A near-zero gain cannot tell an absorbed skill from one that never worked: retire on the unaided level, gated by a witness (2026-10-27)

SLIM's warning above, "removal is not internalization", now has a mechanism
and a measurement. [[literature/papers/wang2026beyond]] sorts a skill by its
two arms. Both pass: **internalized**, retire. Aided passes, unaided fails:
the skill works, keep. Both fail: **uncovered**, neither the model nor the
skill delivers. The gain is about zero in the first and third states. So any
criterion that vanishes at equal arms gives both one verdict. That covers the
difference, the log-ratio, and single-skill leave-one-out or Shapley. More
runs do not fix this, because the error is not noise.

Measured on 844 gated synthetic tasks (target Qwen3.6-27B, which is also the
judge, 3 runs per arm):
- The published |gain| < 0.10 band (from "Not All Skills Help") removes
  57.5% [46.2, 68.7] of the uncovered state. Under each of four open-weight
  targets it removes 58–65%, and 80–99% on binary all-items outcomes. Read on
  a second task, so the verdict and the state are measured separately
  (n = 29), it removes 45% against the level rule's 14%.
- On SkillsBench, 28 of the native band's 31 retirements are probes where no
  trial in either arm passed every item. Both pass rates are 0, so the gain
  is 0. Those probes average U = 0.555 on capability items.
- Fix, the **level rule**: retire when the unaided capability score U ≥ 0.8.
  Trust a high U only if some other "witness" model gains > 0.1 from the
  skill on the same items. The gate admits only 52.6% of tasks with
  U ≥ 0.8, so nearly half of the apparent internalization was tasks the
  skill does not affect. Run the aided arm only when U < 0.4, to flag
  uncovered skills for **repair**, not retirement. Retire softly
  (reversibly).
- Verdicts are **per model**. 37% of skills get contradictory verdicts
  somewhere among the four models. The skill × model interaction explains
  15% of U variance and model strength 0.4%. 19–32% of what any rule retires
  under one model is a skill that works for another.

Limits:
- No downstream outcome separates the rules. Deleting what any rule retires
  costs no detectable SkillsBench reward (level −0.2 [−2.7, 2.4]; level vs
  band −0.8 [−3.1, 1.7]). The case rests on which skills each rule removes,
  and on cross-task κ (0.42 vs 0.23, difference CI [0.01, 0.38]).
- The gain is not the problem; gain *alone* is. Tessl's original condition,
  the band plus A ≥ 0.7, removes no uncovered skill and reproduces at
  κ 0.47.
- One synthesized task per skill is a weak instrument (parallel-forms 0.41).
  A blind Opus 5.5 audit found 9–15% unsupported full marks near the
  threshold, worth about +0.1 on U. Judge leniency therefore pushes a level
  rule toward over-retirement.

How this bears on [[literature/papers/giampieri2026cortex]]'s `/prune`: it is
a gain rule, so a fired fail-both item gets deleted. For one model in one
repo that is the pruning goal, and Wang et al. concede that gain rules serve
it. Our inference: it matters for Cortex's other rule, "a model change is a
retirement trigger". An item flagged after an upgrade may have been absorbed,
or may never have worked. Only the unaided level separates the two.

Rule: to retire a skill or rule clause *because the model no longer needs
it*, require a high unaided score on tasks that some weaker model gains on,
read for the model it will govern. A skill that fails with and without is a
repair item. Gain-only deletion is fine for "this does not help". It is not
evidence of internalization.

## Failure-routed distillation turns a chance failure into a rule; a no-change option is the write-side "none" (2026-11-03)

"Distilling a skill from a failure rather than from a success" above, and
the failure-distilled bundle of 2026-10-26, treat failures as the richer
source. [[literature/papers/nakayashiki2026lessons]] measures the cost of
that when the failure was luck. It used ReasoningBank's released writing
instructions, the same experience-construction pipeline SkillOS holds
fixed in "Why it matters here". After a chance failure of the better of
two tools, the lesson written by the failure instruction moved a fresh
reader of the same model to the worse tool in 11.6% of next runs. The rate
was 0.4% with an irrelevant item and 0.4% with the lesson from a success
(P2 +0.112 [+0.097, +0.126]; 256 worlds, 11 models). The mirror case, a
lucky success of a bad decision, stayed bounded below 0.10 (+0.002). So the
risk sits on the failure-routed half of the write policy.

Two points for this concept:
- **The insert step is where luck gets in.** The released failure
  instruction asks the writer to reflect on why the run failed and to
  prefer "concrete, actionable recovery procedures". That presupposes a
  fault. In the printed lessons that moved readers, the rule was a
  within-run recovery step ("switch to the other tool after an error"),
  and the reader applied it to the next run's first choice. This is the
  insert-side twin of [[literature/papers/he2026does]]: reflection does not
  remove a bad rule, and here it writes one from noise. The admission
  gate's "positive estimated gain, not merely occurred" (tang2026memory)
  is exactly the test a one-outcome lesson fails.
- **"Nothing needs to change" works as a candidate at write time.** One
  outcome-neutral instruction with an explicit no-change option cut the
  rate to 3.1% (N − RB P2 −0.085 [−0.097, −0.073]) and was credited at 5 of
  the 6 models that carried the chance. A decision-quality note in the
  failure instruction cut it only to 7.5% and was credited at 3 of 6. This
  is the write-side version of pham2026reliable's "keep the empty library
  as a scored candidate". N also drops the outcome routing, so the two
  changes are confounded.

The spread across models matters for [[concepts/shared-skill-namespace]]:
the same pipeline gave 0/256 at Claude Opus 5.5 and Sonnet 5.5 and 100/256
at Qwen 3.8 Max. A write policy validated on one model need not carry over.

Limits:
- One stored lesson, read once; no library, retrieval or consolidation.
- Synthetic two-tool worlds with a clear log; the reader could have
  checked the lesson against the record.
- Remedies were tested only on transient failures. A no-change option may
  also suppress a correct lesson after a real breakage; untested.
- Writer and reader were the same model; single author; LLM-reviewed only.

## A session retrospective on Claude Code: proposals carry the skill's own checklist, and a consulted change helped only where it covered the instance met later (2026-11-03)

The sections above test write and removal policies in synthetic loops.
[[literature/papers/treude2026how]] watches the insert step on this box's
own stack (Claude Code 2.1.295, Opus 5.5). A third-party `/retro` skill
reviewed each of 30 finished SWE-bench Verified sessions and proposed
harness changes for human approval. The authors implemented 24 and
evaluated 12 on later same-repo issues, 5 sessions with and 5 without
the change per event.

Three points for this concept:
- **The insert step imports the proposer's priors.** 101 of 125
  proposals cited an event in the session. 19 cited none, and 18 of those
  were lint checks, because the skill's text calls a repo with no
  pre-commit hook or CI lint job "a finding". This is the write-side twin
  of nakayashiki2026lessons: there a chance outcome wrote the rule; here
  the skill's checklist did. tang2026memory's admission test ("evidence
  retained") would reject these proposals on their own record.
- **Fired is not the same as effective.** Cortex's rule (above) treats an
  item that never fired as unmeasured. That case occurs here: the one
  evaluated prior-driven change, a pre-commit config, was present in 5/5
  sessions and run in 0/5, since sessions never commit. The converse
  also occurs. Four known-failures files were consulted in every adapted
  session and left recurrence unchanged (5/5, 5/5, 4/4, 5/5), because
  later sessions met failures the lists did not name. The 2 changes that
  removed the targeted action (5/5 → 0/5, 3/5 → 0/5) both covered the
  instances later met. Exposure is necessary but not sufficient for
  admission.
- **Instance facts age fastest.** A repo-level fact (the test command)
  transferred cleanly. Lists of specific failing tests did not. One list
  also said "treat any unlisted failure as yours", and on an unrelated
  control issue 4/5 adapted sessions ran extra baseline checks against
  0/5 before. This is so2026masked's harm-suite point from the add side:
  a change can be judged only with a task it was not written for.

Rule (our inference): admit a retrospective's change only with a recorded
later task that (a) met the targeted problem in the unchanged harness,
(b) shows the change read, and (c) shows the action changed, plus one
unrelated task for added work. Drop changes justified by the proposer's
checklist rather than by the session.

Limits:
- Seven informative events, 5 sessions per arm, no statistical tests. By
  our Fisher test only 5/5 → 0/5 (p = 0.008) and the control's 0/5 → 4/5
  (p = 0.048) clear 0.05 alone.
- Humans selected proposals and evaluation issues; issues were picked to
  reproduce the problem. Coding was by two LLMs, resolved by the first
  author.
- All evaluated changes target environment problems (test runner,
  pre-existing failures). No removal or revision arm.
