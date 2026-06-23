---
kind: system-proposal
slug: digest-scenario-rich-queries
added: "2026-06-23"
target: "claude/skills/digest/SKILL.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [xiong2026autoresearchbench]
evidence_strength: "code-released (github.com/CherYou/AutoResearchBench); credibility ~4"
status: accepted
recommendation: adopt
accepted_on: "2026-06-23"
applied_commit: "claude-system@a8f0f8e"
---

# /digest should synthesize scenario-rich queries, not keyword piles

## The change

`/digest` step 3 ("Gather the active themes") currently tells the skill to
synthesize queries from `concepts/` filenames + `tags:`, the iteration log,
and NOTES, then:

> Pick 2–5 queries that together cover the project's current fronts. Use
> your judgment on how to split — one per active theme is a good default.

It says nothing about the **form** of each query. The natural reading of
"synthesize from filenames + tags" produces keyword-pile queries
(`agent-memory multi-granularity retrieval`), which is exactly the form
AutoResearchBench finds degrades scientific search.

Proposed: append one instruction to step 3 (no new step, no new knob):

> **Form matters: write each query as a 3–5 sentence natural-language
> scenario, not a keyword pile.** Describe what you are looking for —
> the problem, the kind of method/result, the context — in prose, as you
> would brief a research assistant. Short keyword-style queries
> (`topic1 topic2 topic3`) measurably degrade scientific retrieval;
> scenario-rich queries surface the evidence that lives in appendices,
> ablations, and citation contexts. (Evidence: AutoResearchBench,
> [[literature/papers/xiong2026autoresearchbench]].)

## Why (logical case)

`/digest` is the box's unattended literature intake — it runs weekly on
cron across the research projects and feeds the whole curate→ingest→concept
pipeline. Its recall is capped by the quality of the queries it issues. If
the queries are keyword piles, the skill systematically misses exactly the
deep, in-appendix evidence that distinguishes a real attestation from a
title match. Better queries cost nothing extra at runtime (same number of
`WebSearch` calls) and improve every downstream step. This is a pure
quality lift on an existing, frequently-run skill.

## Why (reputable evidence)

- **`xiong2026autoresearchbench`** (AutoResearchBench) — credibility 4,
  **code released** (github.com/CherYou/AutoResearchBench), so it clears
  Gate 1 on the code-released criterion. It is the strongest external
  calibration the graph holds for autonomous literature discovery, and the
  finding is direct and specific: *"Short web-query-style preferences
  degrade scientific search"* — long natural-language queries outperform
  keyword piles. This is a measured result on the exact task `/digest`
  performs, not an analogy.
- The project's own `web-grounded-literature` concept already records this
  as an open lever: *"Digest query synthesis should produce 3–5-sentence
  scenario-rich queries, not `topic1 topic2 topic3` strings."* This
  proposal simply enacts what the curated concept already concluded.

## Simplicity assessment

`adds_surface_area: false`. This is a one-paragraph edit to an existing
step of an existing skill — no new skill, hook, rule, config knob, or
dependency. It does not change the number of queries, the backend, or the
control flow; it constrains the *form* of a string the skill already
produces. The simpler-form test is satisfied because this *is* the simplest
form: a sharper instruction rather than any mechanism. It arguably *reduces*
ambiguity in the current step (which is silent on query form). Strong pass.

## Risks & what could make this wrong

- **WebSearch ≠ DeepXiv.** AutoResearchBench measured this partly on a
  full-text academic retriever; the box's `/digest` uses general
  `WebSearch`, where the magnitude of the gain may be smaller. But the
  *direction* (prose > keyword pile) is a property of the query, not the
  backend, so the edit should not hurt even if it helps less.
- **Token cost.** Slightly longer queries cost a few more tokens per
  `WebSearch`; negligible against the skill's overall cost.
- **Over-specification.** A too-narrow scenario query could reduce recall
  on genuinely broad sweeps. Mitigated by keeping "2–5 queries covering the
  fronts" — breadth comes from query *count/spread*, depth from query
  *form*.

## Recommendation

**Adopt.** It is well-evidenced (code-released, credibility 4), enacts a
conclusion the project's own concept already reached, adds zero surface
area, and improves the most-run intake skill on the box. If you want to be
conservative, adopt it as written but watch the next few `/digest` runs'
candidate quality before propagating the habit downstream — though there is
little to watch, since the change cannot regress control flow.
