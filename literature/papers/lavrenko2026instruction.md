---
kind: paper
title: "Instruction Duplication as an Inference-Time Control Primitive"
authors: ["Victor Lavrenko"]
institutions: ["PeaceTech VC"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.04024"
code_url: null
citations: null
source: "raw/papers/lavrenko2026instruction.pdf"
added: "2026-09-08"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/typed-enforcement]]"
tags: ["prompting", "constraint-pinning", "inference-time", "placement", "negative-result", "diagnostics"]
---

# Instruction Duplication as an Inference-Time Control Primitive

## TL;DR

Repeating **only the procedural instruction** — no retraining, no decoding
change — raises a deterministic 8-test diagnostic from 90.22% to 93.17%
while leaving **final-answer accuracy exactly unchanged at 60.21%**. The
effect is real, placement-sensitive, and narrow: it improves the *legibility
of the trajectory*, not the answer. The author's own blinded audit **fails
its prespecified criterion**.

## Claims

- Instruction duplication is a minimal black-box control primitive; the
  intervention is a second copy of the instruction and nothing else.
- **It does not make the model more correct.** It makes the generated
  trajectory more inspectable, which matters only when a downstream system
  consumes that trajectory.
- The effect is **placement-sensitive** — where the copy goes changes the
  result — so it is not a "more instruction is better" phenomenon.
- Value is realised through the consuming system, not the model.

## Methods

- Seven instruction-tuned models, 300 medical MCQs, **eight placement
  conditions**, **16,800 scheduled generations**.
- Deterministic All-8 diagnostic: responses passing all eight observable
  tests. Holm-adjusted significance testing throughout.
- A **blinded challenge audit** with a **prespecified** 28/30 confirmation
  criterion.
- Downstream case study in "Answer Engineering" (AE), where explicit
  trajectory state drives local repair.

## Results

- All-8: **90.22% → 93.17%** (+2.95pp), eliminating 30.2% of residual
  failures. TF-IDF recall 73.44% → 74.81% (+1.38, Holm p < .001).
- **Final-answer accuracy remains exactly 60.21%.**
- **Premature commitment gets worse: 1.52% → 2.30%** (Holm p = .00536).
- **The blinded audit yields 10/30 directional confirmations, 20/30
  perceptual ties, no reversals — its prespecified 28/30 criterion is NOT
  met.** The author reports this rather than dropping it.
- Downstream AE: an SSNHL endpoint published at 25.1% reproduced at 84.2%
  with system-only AE, and 97.1% with the trailing duplicate. But for
  conductive branch preservation the same duplicate gives a **within-AE
  decrease** (78.6% → 73.8%), still above the 58.9% no-editing baseline.

## Critique / open questions

- **Single author at a venture-capital firm**, no institutional lab, no
  released code, no citations. The affiliation supplies no methodological
  prior in either direction.
- 300 medical MCQs in one domain; the AE case study is a further narrow
  clinical endpoint. Two of the headline downstream numbers move in
  opposite directions.
- The paper's most useful property is that it **argues against its own
  strongest reading**: an unmet preregistered audit criterion, a worsened
  premature-commitment rate, and a within-AE decrease are all reported.
- Whether a "diagnostic that passes 8 observable tests" is worth anything
  when the answer is unchanged is the open question, and it is exactly the
  [[concepts/hce-evaluation]] worry in reverse — a metric moving without the
  outcome moving.

## Trust signals

- **Credibility:** 2 — arXiv preprint, single unaffiliated author (a VC
  firm), no peer review, no code, no citations. Raised to 2 rather than 1 by
  scale and discipline: 16,800 generations across 7 models and 8 placement
  conditions, Holm correction throughout, a preregistered audit criterion,
  and forthright reporting that the criterion **failed**. The negative
  results are the most trustworthy content in the paper.

## Follow-up

- **Relevance:** 4 — a **fourth attestation for
  [[concepts/constraint-pinning]]** (previously 3 sources) and the first
  with a controlled effect size, placement ablation, and a null.
- **Bears directly on the pending 09-06 `/elevate` proposal**
  (`session-start-limit-first-reading`, built on nakayashiki2026when). Three
  findings here are evidence its reviewer should weigh: the effect is
  **placement-sensitive** (so "repeat the limits at session start" is a
  claim about a specific position, not about repetition), it leaves
  **final-answer accuracy exactly unchanged**, and it **increases premature
  commitment**. The proposal's benefit is trajectory legibility, which is a
  real but narrower benefit than "better adherence." See
  `docs/system-proposals/`.
- The placement sensitivity is the transferable result for
  [[concepts/context-eviction-policy]]: *where* a surviving instruction sits
  after eviction is not neutral.
