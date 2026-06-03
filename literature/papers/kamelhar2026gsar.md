---
kind: paper
title: "GSAR: Typed Grounding for Hallucination Detection and Recovery in Multi-Agent LLMs"
authors: ["Federico A. Kamelhar"]
institutions: ["Oracle Corporation"]
year: 2026
venue: "arXiv 2604.23366"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.23366"
code_url:
citations:
source: "raw/papers/kamelhar2026gsar.pdf"
added: "2026-05-11"
relevance: 5
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/typed-claim-partition]]"
tags: [grounding, hallucination, multi-agent, evaluation, replanning, evidence-typing, faithfulness, llm-as-judge, fever, aiops]
---

# GSAR: Typed Grounding for Hallucination Detection and Recovery in Multi-Agent LLMs

## TL;DR

GSAR partitions every agent claim into a four-way typology (grounded /
ungrounded / contradicted / complementary), weights each by evidence
provenance (`tool_match`=1.00 down to `inference`=0.60), and couples
the resulting scalar S to a three-tier decision rule {proceed,
regenerate, replan} that drives a bounded outer loop under explicit
compute budget K_max — turning grounding from a passive score into
an operational control signal.

## Claims

The paper makes and empirically evaluates five testable claims:

- **C1 — Four-way partition is strictly more expressive than NLI.**
  Collapsing the complementary class K into ungrounded U drops the
  grounded-output rate from 100/200 to 18/200 (−82 pp) under Opus 4.7
  on FEVER.
- **C2 — Evidence-typed weighting is benefit-sensitive to atom
  variety.** Indistinguishable from default at n=50 single-mode (one
  atom per claim); large ΔS̄ ≈ −0.08 at n=200 adversarial-mode under
  Opus and Sonnet judges.
- **C3 — Asymmetric ρ prevents score inflation by contradiction
  suppression.** At n=1000, three of four single-mode judges converge
  to ΔS̄(ρ=0) = +0.058 — same three-decimal result across
  independently-trained models. M4 contradiction catch-rate is
  unchanged, so ρ modifies scoring without modifying identification.
- **C4 — Three-tier {proceed, regenerate, replan} saves compute
  vs. two-tier.** Adversarial-mode n=1000 avoids 283–455 extra replan
  dispatches per 1000 claims relative to a two-tier baseline.
- **C5 — Implementable on a proprietary production stack.** End-to-end
  on Locus SDK + Oracle Database 26ai AI Vector Search + OCI
  Generative AI; 339s wall-clock for n=200 Opus run with zero
  resilience-wrapper retries.

## Methods

- **Four-way claim partition** (Def. 4): grounded G, ungrounded U,
  contradicted X, complementary K. Complementary = non-redundant,
  non-conflicting alternative perspectives — first-class, not
  subsumed by G ∪ X.
- **Evidence-type weight map** w: T → [0,1]. Reference values:
  `tool_match`=1.00, `specific_data`=0.95, `signal_match`=0.90,
  `complementary_finding`=0.85, `synthesis`=0.80, `neg_evidence`=0.70,
  `inference`/`domain`=0.60.
- **Scoring function** (Eq. 2):
  S = [W(G) + W(K)] / [W(G) + W(U) + ρ·W(X) + W(K)] ∈ [0,1].
  Contradicted claims stay in the denominator scaled by ρ — silently
  dropping them inflates S (Property 5).
- **Three-tier decision** δ: [0,1] → {proceed, regenerate, replan}
  with τ_proceed=0.80, τ_regenerate=0.65 reference. Regenerate
  rewrites the synthesis from the existing claim set (~1 extra LLM
  call); replan revises the plan and re-dispatches specialists (1–2
  orders of magnitude more expensive).
- **Bounded outer loop** (Alg. 1) with K_max ∈ {2,3}; degraded-flag
  exit when budget exhausted.
- **Structured judge protocol** emits partition + scalar s + abstain
  channel + natural-language explanation; explanation is fed into
  plan-revision prompts on replan. Judge's own scalar is kept for
  audit/calibration but downstream system recomputes S from the
  partition (w is a deployment parameter, not a judge parameter).
- **Six structural properties** of S formally proved: boundedness,
  monotonicity in G, monotonicity in X, complementary-claim value,
  contradiction non-suppression, inference-observation asymmetry.

## Results

- **Property P5 ablation is judge-invariant.** At n=200, ρ=0 inflates
  ΔS̄ by +0.04 to +0.06 across gpt-5.4 / Sonnet-4.6 / Opus-4.7 /
  Gemini-2.5-pro; at n=1000 three of four converge to +0.058 ± 0.001.
  M4 unchanged in every cell.
- **Opus 4.7 n=200 no-K ablation:** proceed crashes 100/200 → 18/200;
  mean S̄ 0.53 → 0.12; bootstrap 95% CI on Δproceed is [−96, −68].
- **Adversarial-mode dominance:** the no-K effect is ≤2.1 pp on
  single-mode FEVER but reaches −26.1 pp on gpt-5.4 adversarial at
  n=1000. The complementary class earns its keep only when generators
  produce multi-claim diverse output, not on atomic benchmark claims.
- **Judge-selection trade-off:** GPT family routes more aggressively
  to X (higher M4 contradiction catch); Opus uses K more (higher M5
  complementary separation). The scoring function is judge-agnostic
  given the partition.
- **R1/R2 reproducibility:** independent re-runs on separate days
  agree within ±1.5 pp proceed and ±0.002 S̄ at n=1000 — with the
  caveat that one mid-study OCI endpoint flip silently routed all
  four judges to a deterministic rule-based fallback, caught only
  by a partition-shape fingerprint audit (see L5 below).
- **M3 calibration vs. independent Opus grader:** Spearman ρ = +0.53
  in single-mode (95% CI [+0.34, +0.70], 100% proceed agreement);
  ρ ≈ −0.25 in adversarial-mode (robust across strict and role-aware
  grader prompts) — flagged as a per-deployment calibration issue
  for `w(neg_evidence)`, not a structural failure.
- **HHEM-2.1 / RAGAS head-to-head:** HHEM wins on M4 (in-distribution
  on FEVER, binary threshold optimal for two-way SUPPORTS/REFUTES);
  GSAR's claim is not M4 dominance but the structural additions —
  four-way partition, tiered decision, typed weights — that neither
  baseline provides.

## Critique / open questions

- **Single-author paper from Oracle, no peer review yet.** The Locus
  SDK that the empirical stack runs on is internal/proprietary,
  planned-for-release. Reproducibility depends on either Locus
  availability or downstream adopters re-implementing the
  abstractions. The portable fallback (sentence-transformers +
  in-memory cosine) is mentioned but underspecified.
- **The adversarial generator is itself prompt-engineered to
  overreach.** Each summary is constructed to contain ≥1 fabrication;
  the ρ ≈ −0.25 negative correlation under that condition is a
  meaningful structural finding (cross-model weight disagreement on
  `neg_evidence`) but the framing of "negative correlation"
  overstates the case — "default weights need recalibration under
  adversarial conditions" is the more honest gloss.
- **w as expert prior, not learned.** Per-deployment recalibration is
  recommended but no scalable procedure is shown. Section 11.1's
  RLHF/DPO/PRM proposals are speculative future work.
- **Claim atomicity is assumed.** GSAR inherits FActScore-style
  atomic decomposition as pre-processing; compound claims defeat the
  partition.
- **Silent failure as the dominant reliability risk (L5)** is a
  striking practitioner takeaway: four cross-judge runs silently fell
  through to a rule-based fallback during the study itself. The
  fingerprint audit defense is included but the underlying issue
  (LLM-as-judge endpoint can change behavior without warning) is
  structural and unresolved.

## Trust signals

- **Credibility:** 2 — single-author paper (Oracle Corporation);
  arXiv preprint, not peer-reviewed. The empirical stack runs on the
  internal/proprietary Locus SDK, so results are not independently
  reproducible; the portable fallback is underspecified. Reputable
  corporate affiliation is a weak prior here given the absence of
  peer review and released code.

## Follow-up

**Relevance: 5** — directly seeds the new concept
[[concepts/typed-claim-partition]] (four-way typology + epistemic-
weight scoring + tiered control coupling) and provides canonical
operational evidence for [[concepts/citation-anchoring]] (promoting
passive provenance to active control) and [[concepts/hce-evaluation]]
(bounded outer loop under K_max budget is a concrete instantiation
of search-time discipline).

**Connections to our own implementation.** The L1–L7 practitioner
takeaways in §11 read as a checklist for evolving this project's own
knowledge organization. *Persist the full partition not just S* (L5)
maps to our concept-and-literature-note split. *Judge ≠ generator*
(L4) maps to our [[concepts/hybrid-model-backends]] ideator/implementer
separation. *Treat the abstain channel as a first-class planner
input* (L7) doesn't yet have an analogue in our `/digest` or
`/iterate` skills and could be borrowed.

**Suggested next reads** from the 2026-05-11 digest batch:
- ExpWeaver (`zhao2026expweaver`, pending) — read-side experience
  policy, complements GSAR's write-side typology.
- SkillOS (`ouyang2026skillos`, pending) — skill curator/executor
  split as another typed-knowledge organization pattern.
