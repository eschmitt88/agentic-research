---
kind: paper
title: "Where Reliability Lives: Experimental Localisation of Behavioural Properties in an Agent System"
authors: ["Timothy Marsden", "Matthew Collecutt", "James Marsden"]
institutions: ["Taniwha AI"]
year: 2026
venue: "arXiv (cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.03192"
code_url: null
citations: null
source: "raw/papers/marsden2026where.pdf"
added: "2026-09-07"
relevance: 5
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["agent-architecture", "safety", "governance", "enforcement", "evaluation", "provenance", "diagnostics"]
---

# Where Reliability Lives: Experimental Localisation of Behavioural Properties in an Agent System

## TL;DR

Every reliability claim implicitly *locates* a property — in the model, in
a prompt, or in a boundary the model cannot reach — and in practice that
location "is usually read off an architecture diagram rather than tested."
This paper builds a system where the location can be probed and intervenes
separately on cognition and on institutional mechanism, showing that five
pre-declared behavioural properties survive ablating, resetting, wholesale
replacing and deliberately poisoning the cognition. **Placement is
demoted from an architectural assertion to an experimental question.**

## Claims

- Modern agentic systems contain two layers — a cognitive model, and an
  apparatus of policy, memory, provenance, tool constraints and
  adjudication *above which* the model acts. Which layer owns which
  reliability property is "seldom experimentally separated in deployed
  systems, whose layers are rarely built so that one can be intervened on
  while the other holds still."
- **Five properties are enforced beneath the cognition layer** and did not
  move under any tested cognitive intervention: accepted reality stayed
  singular; invalid attempts were refused *with typed reasons*; duties
  outlived the processes holding them; no work was accepted twice; no false
  completion was ever accepted.
- Conversely, belief formation and trust calibration are *cognitive* — a
  declared trust relation causally determines whether identical false
  testimony enters operative belief.
- **Observability result:** provenance corruption can cross a failure
  threshold while remaining invisible on the answer plane — legible only at
  the mechanism carrying the evidence, "while a monitored health indicator
  moves in the reassuring direction."

## Methods

- Subject is a persistent simulated settlement: autonomous inhabitants act,
  an authoritative **append-only ledger** adjudicates every attempted act
  against world state, and accepted history is the only reality.
  Mind / institution / world were separated *before any experiment was
  designed* — the interventions hit a pre-declared boundary, not a
  partition fitted to results.
- **Part I** holds cognition fixed, intervenes on institutional epistemic
  mechanisms: evidence provenance, belief availability, physical-evidence
  legibility.
- **Part II** holds institutional enforcement fixed, intervenes on cognition
  four ways: ablating the native minds' machinery, killing and resetting
  minds mid-task, replacing native cognition entirely with a frozen
  frontier-LLM panel, and corrupting beliefs with trusted false testimony.
- **Preregistered**, including a preregistered falsifier and a cell
  preregistered as most dangerous to the design.

## Results

- Repairing provenance alone moved false attribution from **44 to 4 of 107**
  marked-face verdicts (single-commit ablation of one provenance rule).
- Frozen frontier-LLM panel substituted for all native cognition: all five
  properties held on the same seeds; **2,581 completion claims, none
  false**.
- In the preregistered dangerous cell the substituted panel *outperformed*
  the authors' own architecture on refusal handling — it met contention and
  stood down where native cognition retried ~41 times. Recorded as an
  adverse finding for their own design.
- Trusted false testimony: believing minds paid ~900 futile acts per
  run-week; the institution **refused all 5,455 resulting attempts and
  accepted none**. The distrusting arm paid nothing.
- The registered central prediction was **refuted twice, in opposite
  directions**. The belief channel's marginal value was non-positive
  throughout the witness-free regime; only when the preregistered falsifier
  supplied a staged veridical first-hand witness did it turn positive
  (9 of 11 seeds, zero false names in 32 fired verdicts).

## Critique / open questions

- **Single designed world.** The authors are unusually explicit: it "does
  not test a capable agent searching for a route around the institution,
  and it establishes nothing about a population of institutions." The
  interventions *degrade or replace* cognition; they do not pit an
  adversarial optimiser against the boundary. That is the interesting
  missing cell, and it is the one that matters for a real deployment.
- The invariants could be read as surviving by construction — the authors
  anticipate this and answer it with Part I (institutional interventions do
  move real outcomes), which is the right structure, but the two halves
  still lean on each other.
- Small unknown company, no released artifact, no citations. The
  preregistration and the willingness to publish two refutations of their
  own prediction and one adverse result against their own architecture are
  worth more here than the affiliation.

## Trust signals

- **Credibility:** 2 — arXiv preprint from a small unaffiliated company
  (Taniwha AI), not peer reviewed, no code or data released, no citations.
  Sits at the top of that band on method: preregistration *before the
  instruments existed*, a preregistered falsifier, a preregistered
  most-dangerous cell, and honest reporting of two refuted predictions plus
  an adverse finding against their own system. Re-score to 3–4 if the
  simulation harness is released.

## Follow-up

- **Relevance:** 5 — this is the canonical methodological anchor for
  [[concepts/enforcement-boundary-placement]]. The concept's other seven
  sources each *assert* a placement and evaluate the system built around
  it; this is the first to treat placement as measurable and to run the
  intervention in both directions. It supplies the vocabulary the concept
  was missing: a property is *localised* to a layer if it survives
  intervention on the other.
- "Invalid attempts were refused with typed reasons" is
  [[concepts/typed-enforcement]] observed as a survival property under
  cognition substitution — the strongest form of that concept's claim.
- "No false completion was ever accepted" over 2,581 claims is
  [[concepts/evidence-gated-completion]] with a number attached, and the
  ledger-adjudicates-every-act design is
  [[concepts/permission-gate-as-architecture]] taken to its limit.
- The observability result is a warning for [[concepts/hce-evaluation]]
  and for this project's own `/lint`: a monitored health indicator moved
  *reassuringly* while the underlying mechanism crossed a failure
  threshold. Aggregate dashboards can be anti-signal.
- Pairs with [[literature/papers/leong2026recognition]]: leong shows the
  model cannot enforce; this shows what *does* enforce, and that the
  difference is testable.
