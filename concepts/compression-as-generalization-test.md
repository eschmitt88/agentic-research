---
kind: concept
name: "compression-as-generalization-test"
status: seedling
added: "2026-07-07"
sources:
  - "[[literature/papers/bertran2026fits]]"
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
related_experiments: []
tags: [evaluation, overfitting, compression, certification, hce]
---

# compression-as-generalization-test

## Definition

An ML-research agent's improvement generalizes iff its strategy survives
compression through a deliberately narrow information channel: compress the
explorer's strategy into a short prompt (tens of tokens) and hand it to a
fresh reproducer with training data only — no validation set, code, or
transcript. Gains that came from exploiting the validation set cannot fit
through the channel and fail to reproduce.

## Why it matters here

[[concepts/hce-evaluation]] is split *hygiene* — it prevents the loop from
reading the holdout but says nothing about whether validation-selected
gains are real. This concept supplies the missing *audit*:
[[literature/papers/bertran2026fits]] shows a 128-token
reproduce-or-flag rule separates honest from validation-exploiting
checkpoints with 100% sensitivity / 91% specificity across 8 tasks, and
that a one-bit "did it improve?" ladder channel matches scalar validation
feedback while yielding simultaneous confidence intervals that *certify*
progress. Both instantiate a description-length principle: what fits into
few tokens didn't overfit.

Two importable mechanisms:

1. **Reproducer audit (certify-after-search)** — at chain end (or per
   improvement checkpoint), compress the winning strategy to ≤128 tokens
   and re-run it via a fresh isolated agent on training data alone; flag
   any checkpoint whose compressed reproduction loses >5% relative while
   its validation–holdout gap exceeds 10%.
2. **Ladder feedback (constrain-during-search)** — expose the search loop
   to one-bit improvement feedback instead of raw metric values; the
   binary transcript bounds information leakage and gives per-checkpoint
   generalization CIs for free.

These are the two remedy families
[[literature/papers/ning2026closedloop]] identifies, here with formal
bounds and an agent-native enforcement story (harness-level access
control, not prompt instructions).

## Connections

- Extends [[concepts/hce-evaluation]] from structural split separation to
  active certification of individual improvements.
- Complements [[concepts/pass-at-k]]: pass@k treats seed variance as the
  noise axis; the reproducer audit treats validation-dependence as the
  signal-integrity axis — both ask "is this number real?" of a single
  reported result.
- Single-source seedling: needs a second attestation (another compression
  or reusable-holdout mechanism in an agentic loop) before promotion to
  growing.
