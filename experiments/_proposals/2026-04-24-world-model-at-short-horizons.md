---
kind: proposal
slug: world-model-at-short-horizons
date: 2026-04-24
status: proposed
hypothesis: "On a 2-hour autonomous research loop (1 data-analysis agent + 1 literature-search agent, shared NOTES.md + structured world-model file vs. NOTES.md alone), the structured-world-model arm produces equal or lower statement-inconsistency rate in the final report as measured by a blinded grader. The predicted effect size is small or zero — the Kosmos gains are dominated by the 12-hour horizon."
rationale: "Kosmos motivates the structured world model for 12-hour runs and ~200 agent iterations, where coherence decay across free-text handoffs is severe. Whether the same mechanism is load-bearing at 1-2 hour horizons — the scale most downstream projects will actually operate at — is an open question the Kosmos abstract does not address. A cheap negative result frees downstream projects from building schema-heavy coordination state when a NOTES.md tail suffices; a cheap positive result says the schema is worth implementing even at short horizons."
reads:
  - "[[literature/papers/mitchener2025kosmos]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/citation-anchoring]]"
expected_metric:
  name: statement_inconsistency_rate_delta
  target: -0.02
  direction: lower-is-better
design_sketch:
  - Pick a well-scoped literature-synthesis task (e.g. "summarize recent findings on X" with 20-30 candidate papers).
  - Arm-A (control): two agents coordinate through NOTES.md tail + message passing. No schema.
  - Arm-B (treatment): same two agents plus `_meta/world.yaml` with fields {hypotheses, evidence, open_questions, rejected_claims}. Agents read and write scoped fields per role.
  - Cap each run at 2h wall. Identical compute budget, LLM operator, retrieval tool.
  - Final report graded by a separate blinded agent on statement-anchor consistency (each claim traced to a citation?) and internal consistency (claims don't contradict).
  - Run k=5 seeds per arm. Primary metric is delta in inconsistency rate; report distribution, not just mean.
risks:
  - 2h runs may be too short to accumulate enough coordination failures to measure a delta — a null result could reflect too-small horizon, not mechanism irrelevance. Budget for a 4h replication if the 2h delta is within noise.
  - Blinded grading is expensive and subjective; the grader agent must itself be anchored to a rubric.
  - Arm-B's schema is project-designed; different schemas likely give different results. The experiment compares "*a* reasonable schema" to "no schema," not "the optimal schema" to anything.
related_prior: []
estimated_runtime: "~2h wall per run × 2 arms × 5 seeds ≈ ~20h plus grader time."
---

# world-model-at-short-horizons

Kosmos is the standout recent paper on long-horizon agent coherence,
and its core architectural contribution — the structured world model
shared between data-analysis and literature-search agents — is
exactly the kind of pattern that feels universally right but might
only matter at a specific scale. The 12-hour horizon Kosmos operates
at is unusual; most research-agent deployments today run for minutes
to a couple of hours.

The interesting outcome is either of the two extremes. If the
structured world model produces no measurable consistency improvement
at 2 hours, downstream projects can skip the schema-design cost and
use a NOTES.md tail as the coordination surface for short runs — a
material simplification. If it produces even a small improvement at 2
hours, the schema is worth paying for always, because the cost is a
one-time design and the benefit compounds with every run.

The null hypothesis — "no effect at 2h" — is the stronger prior
given Kosmos's 12h emphasis. A surprising positive result would be
informative in a different direction: it would suggest the schema's
benefit is not about long horizons at all but about forcing explicit
fields, which would in turn suggest that prompt engineering alone
could capture most of the gain via "write your output in these
fields" instructions.

This experiment is realistic on a single-GPU box if the retrieval
backend is `/discover`-style web search rather than training a
retriever from scratch. It is also a natural fit to pair with the
citation-anchoring concept — the grader rubric can reuse the same
anchor-validity check we expect in experiment Diagnostics sections.
