---
kind: system-proposal
slug: lint-consolidation-check
added: "2026-06-28"
target: "claude/skills/lint/SKILL.md"
change_type: edit
adds_surface_area: true
evidence_citekeys: [ouyang2026skillos, pu2026skillops, yang2026skillopt, liu2026harnessing, wu2026bayesian, zhang2026skillcomposer]
evidence_strength: "code-released (SkillOps github.com/Hik289/SkillOps, SkillOpt aka.ms/skillopt) + 3+ independent attestations at credibility 3-4; credibility ~4 on balance"
status: proposed
recommendation: adopt-with-changes
---

# `/lint` should surface consolidation candidates, not only growth gaps

## The change

Add one always-on, **report-only** check to `/lint` (a new check #8,
renumbering the experiments-mode checks). The current always-on set
(checks 1–7) flags only *absence* (orphans, sourceless concepts, dead
links) or *growth* (check #5 MoC candidates — "≥5 concepts on a theme,
promote them"). Nothing in the graph ever flags *over-growth*: two
concepts that have drifted into redundancy, one note that should be
split, a mature concept that should be retired. The graph is
append-only by construction — `/ingest` inserts and updates, but no
skill surfaces a consolidation signal.

Proposed addition to the "Always-on checks" section, immediately after
check #5 (MoC candidates), as its consolidation counterpart:

> ### 5b. Consolidation candidates
>
> The mirror of check #5: where #5 flags a cluster that should *grow*
> into a MoC, this flags pairs/notes that should *shrink*. Report-only,
> conservative, never auto-applied — a human decides. Surface a concept
> **pair** as a merge candidate only when **all** of:
>
> - they list each other in `related_concepts:`, **and**
> - their `sources:` lists overlap by ≥60% (mostly the same papers), **and**
> - they share ≥2 frontmatter `tags:`.
>
> Surface a **single** concept as a *split* candidate when its body is
> long (>~400 lines) AND its `tags:` span two clearly disjoint themes —
> a sign one note carries two concepts. Print the evidence (shared
> sources, shared tags) so the reasoning is auditable; suggest a human
> review, not an action. Do **not** flag deliberately-paired
> complements (e.g. a documented read-side / write-side split) — high
> `related_concepts:` overlap alone is not redundancy.

Optionally (a smaller alternative if even 5b is too much surface): a
one-sentence amendment to check #5 noting that a cluster which is
*already* covered by a MoC but keeps accreting near-duplicate concepts
is a consolidation signal, not a new-MoC signal.

## Why (logical case)

This box's knowledge graph is a procedural/declarative memory library,
and it only grows. `/ingest` and `/digest` insert; concept-merge
during ingestion is the only update path; there is no systematic
delete/merge signal. The NOTES history shows the *symptom*: the
maintainer repeatedly exercises manual restraint against over-growth —
"declined a new agent-memory MoC and a safety MoC," "the memory cluster
is still *not* ripe... enrich-existing was the correct move." That
judgment is currently unaided by tooling. Every other graph-health
defect (orphans, dead links, sourceless concepts, ripe clusters) has a
`/lint` check; the over-growth defect does not. A report-only check
closes that asymmetry without changing any behavior — it just makes a
consolidation opportunity *grepable* the same way check #5 makes a
promotion opportunity grepable.

## Why (reputable evidence)

The `skill-library-lifecycle` concept rests on a strong base and names
this exact enactment twice ("a `/lint` extension that surfaces
'concepts that should be merged'"; "a `/lint` extension might surface
'concepts that should be split'"):

- **`ouyang2026skillos`** (SkillOS) — credibility 4. The concept calls
  it "the largest reported gains in the self-evolving-agents literature
  to date," obtained by holding the experience-construction pipeline
  fixed and changing *only the curation policy*. Its empirical curator
  arc — insert-dominant early, update-dominant mid, a small-but-rising
  *delete/consolidate* fraction late — is direct evidence that
  libraries which only grow underperform libraries that consolidate.
- **`pu2026skillops`** (SkillOps) — credibility 3, **code released**
  (github.com/Hik289/SkillOps): treats a skill library as a
  self-maintaining software ecosystem, i.e. one needing pruning/merge,
  not just insertion. Clears Gate 1 on the code-released criterion.
- **`yang2026skillopt`** (SkillOpt) — credibility 4, code (aka.ms/skillopt).
- **`liu2026harnessing`** (Skill Programs) — credibility 4.
- **`wu2026bayesian`** (Bayesian-Agent) — names the precise failure this
  check guards against: "uncalibrated growth where every run just
  appends another note," and a five-way operation set (patch / **split**
  / **compress** / **retire** / explore) richer than insert/update/delete.
- **`zhang2026skillcomposer`** — names **merge** (consolidate
  semantically similar artifacts) as a first-class lifecycle operation
  distinct from prune.

Gate 1 is cleared by ≥3 independent attestations at credibility ≥3
(SkillOS 4, SkillOpt 4, Skill Programs 4, SkillOps 3) *and* two
code-released anchors. This is not a single weak preprint.

## Simplicity assessment

`adds_surface_area: true` — so it must clear the higher bar, and I hold
it to that honestly.

- **Simpler form considered.** The genuinely simplest form is the
  optional one-sentence amendment to check #5 (flag a MoC-covered
  cluster that keeps accreting duplicates). I propose 5b as the primary
  because the amendment-only form can't catch a *pairwise* redundancy
  that never reached cluster size, which is the more common case here.
  But the amendment is a legitimate fallback if the reviewer wants the
  smallest possible change.
- **Why this form, not a new skill/hook/rule.** It is an edit to an
  existing skill, reusing machinery `/lint` already has — check #5
  already computes shared-tag clusters and reads `related_concepts:`
  and `sources:`. 5b adds a conjunctive overlap test on the same fields.
  No new file, hook, rule, config knob, or dependency.
- **Net direction is toward a *smaller* system.** The check's entire
  purpose is to drive consolidation of the graph (fewer, tighter
  concepts). It adds one report-only check whose output reduces surface
  area elsewhere. This is the "strongly prefer consolidate/clarify"
  case, instantiated as the tooling that *enables* consolidation —
  which is why I judge it past Gate 2 despite `adds_surface_area: true`.
- **Downside is low** — report-only, conservative conjunctive trigger,
  never auto-merges. A human reviews every flag.

## Risks & what could make this wrong

- **Transfer validity.** The evidence is about *skill* libraries
  (procedural how-to artifacts); this graph is *concept/literature*
  notes (declarative). `skill-library-lifecycle`'s own open questions
  flag this: "Does the same insert→update→delete arc apply to factual
  knowledge... the schema may differ." The merge *operation* plausibly
  transfers (redundant declarative notes are as real as redundant
  skills), but the empirical *cadence* from SkillOS does not, and I do
  not claim it does — this proposal borrows only the existence of a
  consolidation operation, not its schedule.
- **False positives on deliberate complements.** `agent-native-memory`
  and `selective-memory-retrieval` are intentionally distinct read/write
  sides yet share sources and tags. The trigger must exclude documented
  complements; if the heuristic is noisy it will erode trust in `/lint`.
  The conservative all-of conjunction and the explicit "don't flag
  documented complements" clause are the mitigation, but tuning may be
  needed after the first few runs.
- **Lint length.** `/lint` already runs 13 checks; this is a 14th. The
  marginal cognitive load is real, though small for a report-only check.
- **Could be premature.** With ~22 concepts, the graph is not yet large
  enough that over-growth bites hard. The counter-argument: tooling that
  guards against uncalibrated growth is cheapest to add *before* the
  graph is large, not after.

## Recommendation

**Adopt-with-changes.** The change is well-evidenced (two code-released
anchors plus three credibility-4 attestations), enacts what the project's
own concept names as its intended `/lint` extension, fills a real
structural asymmetry (every defect type has a check except over-growth),
and serves the simplicity bias by being consolidation-*enabling* tooling.
The "changes" are the guardrails: keep it report-only, keep the trigger a
conservative all-of conjunction, explicitly exclude documented
read/write-style complements, and treat the magnitude/cadence findings
from SkillOS as out of scope (only the *existence* of a merge operation
transfers). If the reviewer wants the minimum, adopt the optional
check-#5 amendment instead of full 5b.
