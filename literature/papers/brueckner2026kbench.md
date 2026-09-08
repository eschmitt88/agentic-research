---
kind: paper
title: "K-Bench: measuring model performance on real scientific agent requests"
authors: ["Aubrey M. Brueckner", "Darshil Patel", "Yuhuan He", "Timothy Kassis"]
institutions: ["K-Dense, Inc."]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.21601"
code_url: null
citations: null
source: "raw/papers/brueckner2026kbench.pdf"
added: "2026-09-08"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/citation-anchoring]]"
tags: ["evaluation", "benchmark", "research-agents", "scientific-discovery", "llm-judge", "overclaiming"]
---

# K-Bench: measuring model performance on real scientific agent requests

## TL;DR

Scientific benchmarks are "written to be scored"; real scientific requests
are underspecified, carry attachments, and **have no ground truth**. K-Bench
scores 1,602 agent runs from live user traffic with three blinded LLM
judges — and finds the leaderboard *does not resolve*: no model clears the
acceptability line under all three judges, and the judges disagree about who
is first. The reported quantity is the **ordering**, not the level.

## Claims

- The informative quantity for a scientific agent is "not a leaderboard
  position but the **joint distribution of what was delivered, what was
  claimed, and what artifacts were produced**."
- The absolute score is an attribute of the *instrument*, not of the system.
  The authors report the top of the table as **unresolved** rather than
  picking a winner.
- Overclaiming is the dominant failure mode, not incapacity.

## Methods

- K-Bench 01: first-turn requests sampled from **live user traffic** on
  K-Dense Web — underspecified, with attachments, no reference solution.
- Nine frontier models run end to end in **identical sandboxes**; 1,602
  completed agent runs.
- Three **blinded** LLM judges score every run on an eight-dimension rubric
  whose 8-anchor is "work a domain scientist would accept with minor edits."
  39,934 scored judgments in total.

## Results

- **No model clears the 8 line under all three judges.** `gpt-5.6-sol` has
  the highest pooled mean at 8.04, but its 95% interval [7.80, 8.23] spans
  the threshold, and **two of the three judges rank `claude-opus-5` first
  instead**.
- 47.6% of all scored judgments fall below the 8-point threshold.
- Rubric difficulty is not uniform: **scientific accuracy averages 6.22 vs
  7.33 for communication**, on identical denominators and in the same
  direction within *every one* of the nine models.
- The single leading failure tag is **overclaiming, on 31.4% of
  assessments**.

## Critique / open questions

- Judges are LLMs scoring LLM output with no ground truth; the paper is
  candid that this bounds the instrument, but the eight-dimension rubric's
  construct validity is still asserted rather than established.
- Live user traffic from the authors' own product is a realistic but
  non-public, non-reproducible task distribution — no code or data link.
- Same group as [[literature/papers/kassis2026scientific]] (Kassis, Patel,
  He, Brueckner). Treat the two as one research programme, not as two
  independent attestations — see the source-independence caveat on
  [[concepts/citation-anchoring]].

## Trust signals

- **Credibility:** 3 — arXiv preprint from a small named company
  (K-Dense, Inc.), not peer reviewed, no released code or data. Method is
  strong where it counts: three *blinded* judges, identical sandboxes, and
  the authors decline to declare a winner when their own instrument cannot
  support one. That refusal is worth more than the affiliation.

## Follow-up

- **Relevance:** 5 — this is the evaluation [[literature/papers/kassis2026scientific]]
  admitted it did not have. That paper shipped 163 curated skills with **no
  task-level evaluation and no host selection rate**; this is the same group
  building the missing instrument. The pair is now a complete story for
  [[concepts/skill-library-lifecycle]] and [[concepts/compression-as-generalization-test]].
- **"Scientific accuracy 6.22 vs communication 7.33, in the same direction
  in all nine models"** is the sharpest evidence yet for
  [[concepts/hce-evaluation]]: agents are systematically better at
  *sounding* right than at *being* right, and a rubric that averages the two
  hides it. Aggregate scores are anti-signal here — the same warning
  [[literature/papers/marsden2026where]] raised about health indicators.
- **Overclaiming at 31.4%** is [[concepts/evidence-gated-completion]]'s
  target failure with a rate attached, measured on real requests rather
  than constructed traps.
- Judge disagreement at the top of the table is a direct caution for
  [[concepts/programmable-evaluator-oracle]]: a *single* fixed judge —
  as in [[literature/papers/yang2026truthinsightbench]] — would have
  reported a clean winner that two other judges contradict.
