---
kind: paper
title: "AgentLTL: A Trace-Verification Framework for Measuring, Enforcing, and Training Procedural Compliance in Tool-Using LLM Agents"
authors: ["Laïla Elkoussy", "Julien Perez"]
institutions: ["LRE, EPITA", "Bpifrance"]
year: 2026
venue: "arXiv preprint (2607.02599, cs.SE)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.02599
code_url: null   # paper states framework + benchmark + training corpus + adapters + scripts are released ("available here"); link target not resolvable from the PDF text
citations: null
source: "raw/papers/elkoussy2026agentltl.pdf"
added: "2026-08-10"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [formal-methods, ltl, enforcement, compliance, trace-verification, grounding, hallucination, rl-reward, tool-use]
---

# AgentLTL: Trace-Verification for Measuring, Enforcing, and Training Procedural Compliance

## TL;DR

A FO-LTL-derived constraint language over tool-call traces yields a
deterministic, judge-free compliance score that one specification
drives three ways — offline scoring, online pre-execution gating, and
a dense GRPO reward — with block-and-warn enforcement improving
compliance on 5/7 models and compliance-reward finetuning adding
+38/+17.5pp accuracy/compliance on held-out patterns including unseen
tool aliases.

## Claims

- Final-answer accuracy and LLM judges cannot see *how* an answer was
  produced; two traces with identical answers can differ in whether
  any tool evidence supports them (Fig. 1: a parametric-memory
  shortcut scores 0.00 compliance while being "correct").
- One spec, three uses: score completed traces; gate proposed calls
  online by checking each prefix (block-and-warn / soft-block); use
  C(τ,G_P) ∈ [0,1] as a dense RL reward. A shared constraint language
  avoids drift between evaluation, deployment, and training.
- **Enforcement is not uniformly beneficial**: block-and-warn helps
  5/7 models (largest gains on the weakest) but *regresses* two strong
  models near their compliance ceiling; soft-block (escalate + kill
  after 3 violations) is the worst setting for 4/7 because forced
  termination prevents recovery. The block-and-warn/soft-block gap is
  itself a diagnostic: it measures how much non-compliance is locally
  recoverable.
- Compliance and correctness diverge in both directions: small models
  compliant-but-wrong, large models correct-but-ungrounded
  (DeepSeek-V4-Pro; parametric shortcut). Sequence ordering (L3/L4) is
  the weakest layer for every model — 55–77% of weighted loss.
- Grounding predicate κ_ground (∀ entity in the answer, a witness in
  some tool output) detects parametric-memory hallucination
  deterministically; a trace-aware LLM judge agrees 77.6% and treats
  κ_ground as necessary but not sufficient. Correctness under default
  prompts falls 52% → 24% as repository popularity falls — models
  answer from pretraining recall, not tools.
- Finetuning with the compliance reward transfers structurally:
  +38.5pp accuracy on unseen-pattern and diverse-tool splits (unseen
  tool-name aliases), gains on no-coverage templates — procedural
  structure, not memorized call sequences.

## Methods

- Language: FO-LTL fragment over trace tuples (tool, args, result,
  index) — atomic propositions (CalledWith, occurrence, ordering,
  return-value checks), temporal operators over finite traces, data
  quantifiers with eager instantiation, user-definable open predicates.
- Weighted constraint sets organized in six layers (L1 branching, L2
  literal args, L3 global sequence, L4 pair ordering, L5 call counts,
  L6 exact args) so one arithmetic error can't collapse the score;
  structural layers carry ~73% of weight.
- Benchmark: 12 templates × 5 difficulty levels, deterministic
  synthetic arithmetic tools, gold traces 2–50 calls; 7 zero-shot
  models (26B–1T) × 360 runs × 3 harness settings via smolagents.
- Finetuning: Qwen3-4B-Instruct, LoRA, GRPO, reward = 0.5 compliance +
  0.25 answer + 0.25 trace-distance (weighted Levenshtein to gold);
  300 examples from 15 simplified templates; two templates held out
  entirely.
- Repository-QA grounding study: 160 (repo, question) pairs over 16
  Python repos, 1,917 traces, κ_ground vs a 5-call trace-aware judge.

## Results

- Harnessing: block-and-warn compliance ↑ on 5/7 (Gemma-4-26B .617 →
  .717, Gemma-4-31B .789 → .961), accuracy +0.019 avg; Qwen3-Next-80B
  and Kimi-K2.6 regress under it; soft-block worst for most (DeepSeek
  V4-Flash .596 → .530).
- Finetuning: validation compliance .511 → .770 during training;
  held-out answer correctness more than triples across all four
  splits (e.g. unaugmented 4.8% → 67.5%); full benchmark +43.8pp
  accuracy / +21.3pp compliance; finetuned model uses *fewer* tool
  calls at higher compliance (base cycles without progressing).
  Remaining failure: nested_loops+branching stays at 0% — deep
  nesting is not recovered by training on simpler motifs, and the
  reward gives no gradient when no partial trace is produced.
- Grounding: strict-grounding prompts raise grounding for all models
  but the extra grounded traces are mostly refusals (vacuous
  grounding: κ_ground is satisfied trivially by an answer with no
  entities); CG stays ≤4% per popularity bucket for Qwen3.5-9B.

## Critique / open questions

- Synthetic arithmetic tool environment — transfer to noisy real tool
  outputs, partial failures, ambiguous arguments is explicitly open.
- Finetuning evidence is one 4B model with LoRA; no scale or
  cross-family replication, and stronger parametric recall may resist
  the anti-shortcut reward.
- Constraint authoring cost is shifted, not removed: amortized for
  generated benchmarks, but deployment needs domain experts to author
  FO-LTL, and mis-specified procedures propagate into both scores and
  training signal (the escape-hatch pattern typed-enforcement already
  tracks).
- Blocking mid-procedure can leave real systems inconsistent (their
  own limitation) — richer enforcement semantics (rollback, escalate)
  than block/kill are not evaluated; constraints are fixed before
  execution, no reactive respecification mid-trace.
- FO-LTL can't express hyperproperties (non-interference between
  parallel tool calls) or timing/resource/privacy constraints.

## Trust signals

- **Credibility:** 3 — small independent French group (EPITA LRE +
  Bpifrance), preprint with no citations yet; but full artifact
  release is claimed (framework, benchmark, training corpus, adapters,
  scripts, fixed seeds), the compute is disclosed (317 GPU-hours), and
  the ACL-style ethics/reproducibility statements are unusually
  thorough for a preprint.

## Follow-up

- **Relevance:** 4 — strengthens [[concepts/typed-enforcement]] with
  the measure/enforce/train unification from a single formal spec and
  the cluster's first *negative* enforcement evidence (strong models
  regress under gating; kill-switches worst), and gives
  [[concepts/citation-anchoring]] a deterministic per-trace grounding
  predicate with a measured judge comparison.
- The L1–L6 layered weighting is a reusable design for any composite
  deterministic score (cf. kg_lint's check taxonomy).
- Watch for the artifact link resolving (code_url) and any follow-up
  on real-tool transfer; pairs naturally with philippov2026glite's
  verifier-scripts operating point.
