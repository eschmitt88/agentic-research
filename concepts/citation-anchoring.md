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

## Open questions

- Anchors to `metrics.json` are stable; anchors to code files
  (`train.py:42-58`) rot as code changes. Whether to invalidate
  anchored claims after refactoring the referenced code is an
  open question — likely a project-level policy decision.
- 79.4% accuracy is measured; we do not know how much of the gap
  is closed purely by requiring anchors versus by improving the
  underlying reasoning. Ablation worth running downstream.
