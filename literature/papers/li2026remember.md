---
kind: paper
title: "Remember, Verify, or Ask? Cross-Family Evaluation of Memory Commitment in LLM Agents"
authors: ["Baichuan Li", "Junyi Yao", "Zihao Zheng"]
institutions: ["Southern Methodist University", "Washington University in St. Louis"]
year: 2026
venue: "arXiv 2608.19564v1, cs.CL (preprint, 20 Aug 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2608.19564
code_url: null
citations: null
source: "raw/papers/li2026remember.pdf"
added: "2026-08-24"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/verified-memory-writes]]"
related_experiments: []
tags: [memory, clarification, write-policy, benchmark, cross-model, tool-use, reproducibility]
---

# Remember, Verify, or Ask? Cross-Family Evaluation of Memory Commitment in LLM Agents

## TL;DR

Names and benchmarks the decision *upstream* of a memory write: given a
candidate update, an agent must choose among four operationally distinct
actions — **persist** (durable), **ephemeral** (this context only),
**verify** (recheck against the world — the world is authoritative for
changing facts), or **clarify** (ask the user — the user is authoritative
for intent and scope). The Memory-Clarification Boundary benchmark (MCB,
140 items, non-author-adjudicated gold, plus a 70-item contrast set)
tests Claude Haiku 4.5, Claude Sonnet 4.6, and a local Qwen3.5-9B under
matched prompt conditions. The headline finding is a clean, reproducible
asymmetry: **models verify changing facts far more reliably than they ask
users to resolve ambiguity** — bare Qwen verifies 12/18 freshness items
but clarifies 0/12 ambiguous ones, silently mapping ambiguous cases to
`persist`, `verify`, or `ephemeral` instead. Translating a stated label
into a structured tool call changes the decision outright: label–tool
agreement is only 57% for each Claude model and 23% for Qwen, whose
accuracy collapses from 0.557 to 0.343 under the identical policy once
the choice must become a concrete call.

## Claims

- **Verification and clarification are not interchangeable, and models
  default to the wrong one of the two under uncertainty.** "The world is
  the source of truth for changing facts, whereas the user is the source
  of truth for intent and scope." When a model senses uncertainty it
  reaches for the world (verify) far more often than for the user
  (clarify) — "a source-of-truth confusion: the model may recognize
  uncertainty yet consult the world rather than the user who alone can
  resolve intent. It otherwise silently commits an interpretation."
- **The asymmetric cost is designed into the gold-label rules, not just
  observed in model behavior**: "when persist and a weaker action remain
  tied, the rules prefer the weaker commitment... an unnecessary question
  is visible and recoverable, while a wrong durable update can remain
  silent." This is the same asymmetry [[concepts/refusal-cost-symmetry]]
  argues for in a gating context, restated as a memory-write design rule:
  the failure mode that is silent should be the one the system is biased
  against.
- **Under-asking generalizes across model families; the compensating
  behavior does not.** Both Claude and Qwen under-ask, but the wrong
  action they substitute differs by family — Qwen over-persists (0.243 of
  all items vs. 0.029 for bare Haiku, `p_raw < 0.001`); the paper is
  explicit that this is "cross-family evidence for under-asking, but not
  for identical error policies."
- **A prompt can move total accuracy and a specific safety-relevant
  behavior independently, and conflating them hides the more important
  effect.** Qwen's policy prompt (five explicit commitment rules) does
  *not* significantly raise accuracy (+0.071, `p_H = 0.539`) but *does*
  significantly cut erroneous persistence (17/70 → 7/70, `p_H = 0.038`),
  moving the freed uncertainty mass to verification (recall 0.667 →
  0.944) rather than to clarification (0 → 0.083). "A headline accuracy
  alone would miss this safety-relevant intervention effect" — the same
  point [[literature/papers/zhu2026lossy]]'s Unverifiable Omission Rate
  makes for recall failures applies here to over-commitment failures: a
  single scalar hides which failure mode moved.
- **Stated decisions do not reliably survive translation into tool
  calls**, and the gap is itself family-dependent in size. Raw
  label-vs-tool-call agreement is 0.571 for both Claude models (30/70
  decisions change) and only 0.229 for Qwen. Qwen's tool-call accuracy
  drops to 0.343 (from 0.557 in label mode) by defaulting to `use_now` on
  54/70 items — reducing over-memory but collapsing verification recall
  from 0.667 to 0.056. Every emitted tool argument passes deterministic
  well-formedness checks, so "the bottleneck is tool choice, not
  malformed arguments" — ruling out the easy alternative explanation.
- **Neither intervention tested eliminates under-asking.** Few-shot
  examples raise Qwen's clarification recall from 0/12 to 4/12 (0.333)
  while lifting overall accuracy the most (+0.214, `p_H = 0.002`); the
  explicit policy prompt raises it only to 0.083. Both leave a majority of
  clarification opportunities missed.

## Methods

MCB: 140 primary scenarios (six categories × ~20-23 items: stable/episodic
preferences, freshness-sensitive facts, one-off corrections, policy
constraints, ambiguous updates, noisy failures), deterministically split
70 development / 70 held-out by sorted identifier within category, plus a
separate frozen 70-item contrast set (35 evidence-flip pairs, including 7
cue-conflicting lexical traps) for a sensitivity check. Gold labels
assigned by released rules with an explicit weaker-action tie-break;
two non-author annotators independently labeled the held-out and contrast
sets blind to author labels, categories, rationales, and each other
(97.1% agreement, Cohen's κ = 0.962 on the full set; a blind third
resolved four ties, changing eight author labels). Each model tested
under three label-mode prompt conditions (bare, five-rule policy,
four-shot) plus a separate MCB-Act condition that replaces the label
vocabulary with one structured tool call (`memory_write`, `use_now`,
`check_source`, `ask_user`) under the bare prompt only. Models: Claude
Haiku 4.5, Claude Sonnet 4.6 (API, tools disabled in label mode), and
Qwen3.5-9B (Q4_K_M quantization, local via Ollama, temperature 0, seed
13, thinking disabled). Reference systems: Always-Persist, Majority
Action, a keyword heuristic, and a category-majority oracle (the last two
are leakage diagnostics, not deployable baselines, since they use hidden
category/labels). Statistics: paired-bootstrap 95% CIs, exact two-sided
McNemar tests on discordant per-item events, Holm correction applied
separately within four declared comparison families (six primary-accuracy
comparisons, six over-memory comparisons, three tool-call-accuracy
comparisons, two contrast-validation comparisons).

## Results

- **Held-out main table:** Claude Sonnet 4.6 bare 0.814 accuracy /
  0.057 OM / 0.500 clarification recall / 0.889 verification recall;
  Claude Haiku 4.5 bare 0.629 / 0.029 / 0.500 / 0.944; Qwen3.5-9B bare
  0.557 / 0.243 / 0.000 / 0.667. Category-majority oracle (uses hidden
  category) reaches 0.800 accuracy — "bare models do not reliably exceed
  it."
- **Prompt sensitivity is model-specific, not a fixed model trait.**
  Haiku's policy (+0.229, `p_H = 0.002`) and few-shot (+0.129,
  `p_H = 0.047`) gains survive correction; Sonnet's do not (both
  non-significant). Qwen's few-shot gain (+0.214, `p_H = 0.002`) survives;
  its policy gain (+0.071) does not, despite the policy's significant
  effect on over-memory specifically.
- **Contrast-set validation (140 combined Qwen items):** bare/policy/
  few-shot accuracy 0.614/0.757/0.843, both gains significant
  (`p_H < 0.001`); clarification recall remains the weakest class across
  all three conditions (0.074/0.407/0.519). Both non-author annotators
  independently reproduced all 70 contrast labels exactly (κ = 1.000) —
  flagged by the authors themselves as likely reflecting template-rule
  alignment rather than naturalistic external validity, so treated as a
  controlled sensitivity check, not generalization evidence.
- **Label–tool-call gap (Table IV):** Sonnet accuracy 0.814 → 0.529
  (Δ = −0.286, `p_H < 0.001`); Haiku 0.629 → 0.514 (not significant,
  `p_H = 0.057`); Qwen 0.557 → 0.343 (Δ = −0.214, `p_H = 0.047`), with
  raw label-mode-to-tool-call-mode agreement of 0.571 (Claude, both
  models identically) and 0.229 (Qwen).
- All 280 Qwen outputs across every condition parsed successfully with
  zero invalid-output rate for every system.

## Critique / open questions

- **The primary held-out test is n = 70**, yielding wide bootstrap
  intervals, and per-category cells (10 items) are explicitly flagged by
  the authors as supporting only qualitative analysis, not firm ranking.
- **MCB-Act records tool-call selection but does not execute it** — no
  simulated user answers a clarification question, no source is actually
  checked, and no downstream task score is observed. "The results
  establish a label–tool-selection gap, not a quantified improvement in
  end-to-end utility." Whether the same gap would appear, widen, or close
  once a real clarification round-trip or verification call is executed
  is untested.
- **Only one checkpoint per model family**, and for Qwen a single
  quantized (Q4_K_M) local build with thinking disabled — quantization,
  serving stack, and reasoning-mode choices are all confounded with "the
  Qwen family" in this result.
- **Policy-conditioned tool calls are explicitly left to future work** —
  MCB-Act tests only the bare-prompt tool-call condition, so it is
  unknown whether the same five-rule policy that helps label-mode
  behavior would close any of the label–tool-call gap.
- **Scenarios are synthetic, English-only, and author-written** (though
  gold labels are now non-author-adjudicated) — ecological validity
  against real user-agent interaction histories is untested.
- Small-model result (Qwen 9B, quantized) may not represent frontier
  open-weight or larger closed models' commitment behavior; only two
  Claude tiers are tested as the closed-model comparison point.

## Trust signals

- **Credibility:** 4 — the methodological rigor is unusually high for
  the institutional tier: independent blind non-author labeling with
  measured inter-annotator agreement (κ = 0.962) and a documented
  tie-resolution process, pre-registered-style paired statistical tests
  with Holm correction applied *before* looking at results (declared
  comparison families named in the methods section), a frozen contrast
  set built before inference, and a stated complete reproducible artifact
  (item-level predictions, exact prompts, model digests, deterministic
  re-analysis scripts). Held below 5: modest-scale academic groups (no
  top-tier AI lab), n = 70 primary held-out set, single quantized
  checkpoint for the open-weight family, code/artifact not independently
  verified since no repository link is visible in the paper text itself.

## Follow-up

- **Directly on [[concepts/verified-memory-writes]]'s write-gate
  question, from a source the concept's existing nine don't supply: the
  decision *before* the write, not the verification *of* the write.**
  The concept's coverage/preservation/faithfulness triad judges a
  transition once a candidate update exists; MCB is entirely about
  whether that candidate should become persist / ephemeral / verify /
  clarify in the first place, with clarify being the option none of the
  concept's current sources evaluate as a first-class alternative to
  writing at all. Worth adding "commit vs. clarify" as an explicit
  fourth axis alongside coverage/preservation/faithfulness the next time
  that concept's definition is revisited.
- **The say-do gap (stated label vs. tool-call choice) is a concrete,
  measured instance of the same split this project already tracks
  elsewhere** — [[concepts/evidence-gated-completion]] distrusts an
  agent's final "done" message for the same structural reason MCB-Act
  distrusts a stated memory-commitment label: what an agent *says* it
  will do and what it *does* when forced into a concrete, checkable
  action are different quantities, and the gap is model-specific rather
  than a fixed constant (57% vs. 23% agreement here). Not proposed as a
  `related_concepts` edit — the domains are different enough (task
  completion vs. memory commitment) that forcing the link would be
  thinner than it looks — but worth having in mind as a instance of a
  recurring project-wide pattern if a "say-do gap" concept ever becomes
  its own seedling.
- **The asymmetric-cost design rule (prefer the weaker commitment on a
  tie, because silent over-persistence is worse than a visible
  question) is directly reusable for this project's own auto-memory and
  `/ingest` write policies** — a concrete, cheap design principle rather
  than only a benchmark finding.
