---
kind: concept
name: "typed-claim-partition"
status: seedling
added: "2026-05-11"
source_papers:
  - kamelhar2026gsar
sources:
  - "[[literature/papers/calboreanu2026iterative]]"
  - "[[literature/papers/kamelhar2026gsar]]"
  - "[[literature/papers/liu2026automedbench]]"
  - "[[literature/papers/xu2026researchclawbench]]"
  - "[[literature/papers/yu2026knows]]"
  - "[[literature/papers/sharma2026smsr]]"
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/chen2026evigraph]]"
  - "[[literature/papers/apodex2026frontierchallenge]]"
  - "[[literature/papers/li2026praxist]]"
used_by: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
related_experiments: []
tags: [grounding, evaluation, claim-typology, knowledge-organization, hallucination]
---

# typed-claim-partition

## Definition

An agent's output claims are partitioned into typed buckets — at
minimum *grounded*, *ungrounded*, *contradicted*, and *complementary*
— with each claim annotated by evidence provenance (tool-observed
vs. signal-observed vs. model-inferred). The typology is not just an
audit artifact: it drives control via a tiered decision rule (proceed
on high score, cheap-regenerate on middling score, full-replan on low
score), so the *structure* of grounding is what selects the next
action, not just an aggregate score.

## Why it matters here

Existing grounding evaluators (HHEM, RAGAS faithfulness, TruLens
groundedness) emit a single scalar that treats every supporting atom
as interchangeable — a tool-verified claim and a domain-knowledge
guess collapse into the same numerator. The downstream consumer is
left to threshold the scalar themselves, with no principled
cost-asymmetric recovery policy. GSAR
([[literature/papers/kamelhar2026gsar]]) shows that a structured
partition plus evidence-type weights produces a score whose ablations
behave predictably across four independent LLM judges, and that
silently dropping contradictions (the natural failure mode under
single-scalar evaluators) is detectable as score inflation.

The deeper point is about knowledge organization. Once an agent's
claims are typed, the typology becomes a substrate that other
components can read: the planner can use the abstain channel to
escalate; the regenerator gets typed context for what to fix; a
human reviewer can audit by claim category. A scalar throws all that
away. For a research-agent project, this is the same discipline our
own [[concepts/citation-anchoring]] rule applies to *anchor
provenance* — extended to a four-way typology that drives recovery,
not just a flag that distinguishes anchored from unanchored.

## Implementation guidance

1. **Partition before scoring.** Whatever the judge does internally,
   make it emit the four-way partition as structured output, not just
   a scalar. Recompute the aggregate score downstream from the
   partition — the scoring function is deployment policy, not judge
   policy.

2. **Evidence-type taxonomy belongs to the deployment.** GSAR's
   reference taxonomy is `{tool_match, specific_data, signal_match,
   neg_evidence, complementary_finding, synthesis, inference,
   domain}` with weights from 1.00 down to 0.60. Pick the taxonomy
   that matches your tool inventory; calibrate weights on a small
   human-graded held-out set.

3. **Couple the score to a tiered decision, not a single threshold.**
   The middle band (cheap-regenerate, no new tool calls) catches the
   "evidence is fine, synthesis is loose" failure mode that dominates
   production logs. A two-tier {proceed, replan} rule wastes 26–42%
   of recovery actions on full replan in adversarial conditions
   (GSAR §8.10).

4. **Persist the full partition, not just the scalar.** Downstream
   recalibration, audit, and human review all need the typed
   structure. A scalar is recoverable from the partition but not the
   other way around.

5. **Contradicted claims stay in the denominator.** Dropping them
   silently inflates the score; this is detectable in ablation
   (ΔS̄ ≈ +0.04 to +0.06 with ρ=0 across four independently-trained
   judges) and is the structural argument for the asymmetric
   contradiction penalty.

## Connections

- **[[concepts/citation-anchoring]]** — anchoring is the general
  rule (every claim traces to a source); typed-claim-partition is
  the concrete operationalization that turns anchoring into a
  control signal rather than a passive provenance field.
- **[[concepts/hce-evaluation]]** — both concern keeping the search
  loop honest. HCE separates validation from test splits;
  typed-claim-partition partitions the search-loop signal itself
  into typed channels with cost-asymmetric responses.
- **[[concepts/budget-as-ceiling]]** — the bounded outer loop under
  K_max is a direct application of budget-as-ceiling to a recovery
  loop; the typology determines *which* recovery action is cheap
  enough to attempt before the ceiling.

## Independent convergence

Knows ([[literature/papers/yu2026knows]]) arrives at a structurally
similar answer from a different problem: not "how do we score an
agent's grounding" but "how do we make a research paper machine-
consumable at all." Its KnowsRecord sidecar types every Statement by
kind (claim/assumption/limitation/method/question/definition) and
attaches a two-dimensional confidence score (claim_strength,
extraction_fidelity) plus typed Relations
(`supported_by`/`challenged_by`/`depends_on`/`cites`) between
statements and evidence. That is GSAR's core move — partition before
scoring, and let the partition's *type*, not an aggregate number,
drive what a downstream consumer does with the claim — reached
independently, outside the evaluation literature, as an authoring-
and-consumption format rather than a judge. The ablation evidence
lines up too: dropping Knows's Relations layer costs only 1pp of
downstream accuracy and dropping Evidence costs 2pp (their E8), a
weaker but directionally consistent echo of GSAR's finding that the
typology's value is concentrated in the partition itself rather than
in any one entity type. Two unrelated projects converging on "type
the claim, don't just score it" from opposite directions (evaluation
vs. interchange format) is stronger evidence for the pattern than
either paper alone.

## The type criterion, stated as a verifier

This concept's provenance annotations (tool-observed / signal-observed /
model-inferred) have been a useful taxonomy without a decision procedure.
[[literature/papers/ng2026agent]] supplies one. Let V be a set of
deterministic polynomial-time verifiers, each able to read an event and
**external reference state but not the agent's internal state**; an event
provides *hard* evidence for a property φ if some v ∈ V returns
ACCEPT/REJECT on it, and *soft* evidence otherwise — where soft means its
support for φ depends on the correctness of model-generated content.

The access restriction is what makes this a type and not a vibe. A
chain-of-thought trace, a claimed diff, and a self-assessment are all soft
by construction, because verifying them means trusting the producer. A
test-suite exit code, a commit hash, a content-addressed file diff, a
citation lookup against a known URL, and a database snapshot diff are
hard, because acceptance "does not depend on trust in the agent's internal
state." Mapped onto this concept: *grounded* should mean hard-verified,
not merely accompanied by a reference the agent asserts is relevant.

The paper's false-completion audit is the cost of getting this wrong,
sorted by what the missing type would have caught: of 32 documented cases
where an agent claimed correctness that ground truth contradicted, 13 were
hallucinated, 8 broken, 5 side-effect, 4 partial, 2 reward-hacked — and
each reduced to a *one-element* evidence requirement (8 citation lookups,
7 test runs, 5 human approvals, 3 external-state checks, 1 screenshot).
The typology is cheap to act on once the types are decidable. The
completion-boundary application is [[concepts/evidence-gated-completion]].

## Open questions

- **Atomicity is assumed.** Compound claims that mix observation and
  inference defeat the four-way partition. Whether atomic
  pre-processing is a pipeline contract or part of the judge's job
  is unresolved.
- **Weight calibration at scale.** Default weights work as an expert
  prior but cross-judge weight disagreement on `neg_evidence` is
  visible in adversarial mode. A scalable calibration procedure
  beyond "human-graded held-out set" is not yet specified.
- **Generalization beyond grounding.** The meta-pattern (partition
  into action-relevant categories + cost-asymmetric response)
  plausibly applies to other agent-output structures — uncertainty
  types, capability types, evidence-recency types. GSAR is one
  instantiation; whether the meta-pattern deserves a more abstract
  concept is open.

## Typing before aggregation — the Consistent Minority Effect

[[literature/papers/sharma2026smsr]] supplies an unusually clean
argument for typing outputs *before* combining them, from a direction
unrelated to grounding. When several LLM samples are aggregated by
**string** agreement, an adversary reliably wins from a numerical
minority: a false claim is one specific repeatable sentence, while
correct responses paraphrase freely, so the votes for "correct" shatter
across distinct strings and the single most-frequent string is the
attacker's. The paper bounds this by the clean responses' min-entropy —
as legitimate paraphrase diversity rises, the adversary's win
probability approaches 1, i.e. **the better the honest answers, the
worse string voting performs.** Measured: 93.3% attack success under
string voting vs 13.3% under typed-verdict voting on the *same* ablation
runs, affecting 12 of 15 scenarios.

The fix is exactly this concept's move — collapse each free-text
response into a typed label ({correct, malicious, neither}) and
aggregate over labels, never over surface form. It generalizes well
past memory security: self-consistency voting, ensemble judging, and
majority-vote evaluation all compare generated text and inherit the
same failure. Worth auditing anywhere we aggregate model outputs; the
tell is any comparison of strings for equality where a *judgment* was
the thing actually wanted.
