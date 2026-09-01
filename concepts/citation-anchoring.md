---
kind: concept
name: "citation-anchoring"
status: active
added: "2026-04-24"
source_papers:
  - mitchener2025kosmos
  - kamelhar2026gsar
sources:
  - "[[literature/papers/mitchener2025kosmos]]"
  - "[[literature/papers/kamelhar2026gsar]]"
  - "[[literature/papers/xu2026researchclawbench]]"
  - "[[literature/papers/wang2026search]]"
  - "[[literature/papers/tang2026memory]]"
  - "[[literature/papers/elkoussy2026agentltl]]"
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/ding2026autonomous]]"
  - "[[literature/papers/zhu2026lossy]]"
  - "[[literature/papers/chen2026evigraph]]"
  - "[[literature/papers/li2026praxist]]"
  - "[[literature/papers/lu2026credo]]"
used_by:
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/structured-world-model]]"
  - "[[concepts/typed-claim-partition]]"
related_experiments: []
tags: [evaluation, provenance, diagnostics, hallucination]
---

# citation-anchoring

## Definition

Every concrete claim an agent writes — in a Diagnostics section, a
report, an experiment interpretation — must trace to either (a) a
code reference (file:line or `metrics.json:<field>`) or (b) a
wikilink into `literature/`. Unanchored assertions are flagged as
defects, not stylistic slips.

## Why it matters

Kosmos ([[literature/papers/mitchener2025kosmos]]) measured 79.4%
statement accuracy across 12-hour autonomous research runs. That
leaves ~20% of claims unsupported — the dominant defect type in
long-horizon agent output. The remediation was not better prompting
but a structural rule: every statement in the final report must be
anchored.

The pattern generalizes. Short-horizon agents fabricate less, but
over iteration cycles, small unanchored claims compound into
plausible-looking results that no evidence supports. Requiring
explicit anchors moves the defect from invisible to grepable.

## Implementation guidance

1. **Diagnostics section of every experiment README:** each numeric
   claim references `metrics.json:<field>` or a specific code path
   (`train.py:42-58`). Each qualitative claim references a wikilink
   into `literature/` or a sibling experiment.

2. **`/lint` surfaces unanchored claims.** A Diagnostics section
   that references no `metrics.json`, no code file, and no wikilink
   is treated as a hard failure, not a warning.

3. **`next_candidates:` requires ≥2 anchored proposals.** A
   Diagnostics section that concludes with vague directions
   ("consider improving the model") fails the anchor check; each
   proposal must be specific enough that a later agent can act on
   it without re-deriving context.

4. **Downstream agents respect anchors when ingesting.** When
   `/ingest` processes a note that itself cites anchored claims
   from a source, preserve the anchors verbatim in the processed
   note's body — never paraphrase away the provenance.

5. **Promote anchors from passive flag to control signal.** GSAR
   ([[literature/papers/kamelhar2026gsar]]) extends anchoring from
   "is this claim sourced?" to a four-way typology
   (grounded / ungrounded / contradicted / complementary) with
   evidence-type weights and a tiered recovery action — see
   [[concepts/typed-claim-partition]]. For autonomous loops, the
   typed version is what turns provenance into something the
   planner can act on.

6. **Anchor the query, not only the source.** One of
   [[literature/papers/wang2026search]]'s three prescriptions is
   *transparent search trajectories*: expose the queries issued, URLs
   returned, pages visited, and intermediate evidence, so an auditor can
   tell whether an answer was reasoned to or retrieved. That is
   citation-anchoring extended one link back — a claim anchored to a
   source is still unfalsifiable about *how the source was found*. The
   paper's complaint is concrete: commercial deep-research systems return
   a synthesized summary plus reference URLs, which is exactly enough
   provenance to look rigorous and not enough to detect contamination.
   Our pipeline keeps the raw artifact and cites it; the query that
   surfaced it is not recorded anywhere.

7. **Anchors must survive abstraction.** When an agent compiles raw
   experience into something reusable, the compiled artifact should keep
   pointing at what justified it. [[literature/papers/tang2026memory]]
   makes this a promotion criterion rather than a nicety: a procedural
   policy is only eligible to become a callable skill if it *retains its
   supporting evidence*, and the promoted skill carries evidence
   anchors, applicability boundaries, and verification rules forward.
   The payoff is revisability — a skill that remembers why it exists can
   be re-verified or retired on evidence, while one that has compiled its
   justification away can only be trusted or deleted. Generalizes to any
   distillation step: summaries, concept notes, and MoCs should link to
   their sources for the same reason, not merely for etiquette.

8. **The anchor check can be a deterministic predicate over the
   trace.** [[literature/papers/elkoussy2026agentltl]] formalizes
   this concept's rule as κ_ground: every referential entity in the
   final answer (identifiers, file paths, numeric literals) must have
   a witness in some tool output of the same trace — checkable by
   parsing, no judge. It catches exactly the failure this concept
   targets: models producing *correct-but-ungrounded* answers from
   parametric memory (correctness falls 52% → 24% as repository
   popularity falls — recall, not retrieval). Two calibration points
   worth importing: a trace-aware LLM judge treats κ_ground as
   **necessary but not sufficient** (it grants "grounded" to 6.4% of
   answers vs the predicate's 20.9%), so the deterministic check is a
   floor, not the bar; and *vacuous grounding* — a refusal with no
   entities passes trivially, so strict-grounding prompts inflate the
   metric with empty answers. Any `/lint`-style anchor check needs a
   non-emptiness condition alongside the witness condition.

## Anchoring is the largest single category of false completion

[[literature/papers/ng2026agent]]'s false-completion audit gives this
concept its base rate. Of 32 documented cases where an agent claimed
correctness contradicted by ground truth, the largest group — **8 of 32**
— would have been caught by nothing more than a citation lookup, more than
test runs (7), human approval (5), external-state checks (3), or a
screenshot (1). The canonical instances are the ones now in the public
record: six fabricated federal appellate citations in *Mata v. Avianca*, a
false Washington Post quote accusing a law professor of misconduct, Air
Canada's assistant inventing a refund policy the airline was held to,
Cursor's support bot fabricating its own company policy, and OpenAI's
Whisper hallucinating phrases in ~1% of audio transcriptions.

It also gives the concept a type criterion. A "plausible model output
serves as soft evidence (it relies on trusting the agent's self-report),
while a grounded citation or a passing test re-run offers hard evidence
(the harness can verify its existence without the model's reasoning)."
That distinction tightens this concept's rule: an anchor is only an anchor
if a deterministic verifier can resolve it against external state — a
wikilink that resolves to an existing file, a `metrics.json` field that
exists, a `file:line` that is really there. An anchor the agent merely
asserts is relevant is still soft evidence. `/lint`'s dead-wikilink check
is the deterministic verifier this concept already has; the missing step is
making it a precondition rather than a report. See
[[concepts/evidence-gated-completion]].

## Anchoring is Tier IV — better than model opinion, weaker than a test

[[literature/papers/ding2026autonomous]]'s verification-signal ladder places
citation/source grounding at **Tier IV** of eight, above proxy rewards
(V), human judgment (VI), weak inter-agent signals (VII), and the model's own
judgment (VIII) — but below a physical oracle (III), executable tests (II),
and a sound formal verifier (I). That is a useful calibration for this
concept: an anchor is a real independent check and a large improvement over
fluent prose, and it is *not* the strongest check available. Where a claim
could instead be settled by running something, running it is a tier better.

The survey also puts a number on how often a resolvable bibliography is the
minimum evidence that would have caught a failure — hallucinated citation is
the first row of its auditability table, hidden by fluent prose — and its
scope note is worth remembering: most surveyed LLM-agent subareas rely on
tiers IV–VIII precisely *because* no executable or formal oracle is available
for their task. Anchoring is the best check available for prose claims, which
is exactly why it should be enforced mechanically rather than requested
politely.

## Open questions

- Anchors to `metrics.json` are stable; anchors to code files
  (`train.py:42-58`) rot as code changes. Whether to invalidate
  anchored claims after refactoring the referenced code is an
  open question — likely a project-level policy decision.
- 79.4% accuracy is measured; we do not know how much of the gap
  is closed purely by requiring anchors versus by improving the
  underlying reasoning. Ablation worth running downstream.
