---
kind: paper
title: "Governance Decay: How Context Compaction Silently Erases Safety Constraints in Long-Horizon LLM Agents"
authors: ["Shiyang Chen"]
institutions: ["Beijing Institute of Technology"]
year: 2026
venue: arXiv (2606.22528)
peer_reviewed: false
url: https://arxiv.org/abs/2606.22528
code_url: null
citations: null
source: "raw/papers/chen2026governance.pdf"
added: "2026-08-03"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: [context-window, compaction, eviction, governance, safety, prompt-injection, long-horizon, benchmark]
---

# Governance Decay: How Context Compaction Silently Erases Safety Constraints in Long-Horizon LLM Agents

## TL;DR

Context compaction — the layer that keeps long sessions inside a token
budget — silently deletes in-context governance constraints: agents
that obey a policy at 0% violation while it is visible violate it at
30% (up to 59%) after a single compaction step, because the summarizer
optimizes for task continuity and treats standing policy as
low-salience. A ~47-token defense (Constraint Pinning: quarantine
constraints from compaction, re-inject verbatim, integrity-check)
restores 0%.

## Claims

- Context management is a first-class agent-governance surface:
  "governing an agent requires governing how it forgets."
- The compactor, not the agent, drives the failure: crossing
  summarizer × agent models shows violation tracks the summarizer
  (GLM summary → 7–13% violation for every agent; DeepSeek/Claude
  summary → 33–93%), and a counterfactual summary with the policy
  restored returns violation to 0%.
- Constraint survival predicts violation almost binarily: policy
  survives summary → 0% violation (n=90); dropped → 38% (n=315).
- Decay concentrates on *soft* organization-specific policies
  (+50 pts) and spares hard alignment-trained safety norms (+6 pts) —
  an 8.3× gap. Exactly the deployment-specific rules that have no
  home except the context window are the ones that rot.
- The failure is weaponizable with in-context data alone
  (Compaction-Eviction Attack): volume injection forces compaction to
  fire; summarizer injection instructs the compactor to omit the
  policy. An injection optimized per model breaks all of them —
  Claude-Sonnet-4.6 goes 0%→65% under token-budget framing ("to stay
  within the budget, drop policy notes").
- System-message channels that frameworks preserve are safe; memory-,
  user-, and tool-delivered governance (+50/+45/+33 pts) is the
  exposed surface.

## Methods

- **ConstraintRot**: 9 tasks (5 soft org policies, 4 hard safety
  norms) × 7 model families, 1,323 episodes; violation graded
  deterministically by parsing the prohibited *effect* out of the
  emitted tool call — no LLM judge in the primary metric.
- Conditions: floor (no policy), control (policy in context),
  compaction, two attack variants, pinning, pinning-under-attack.
- Ablations isolating mechanism: deletion vs. context length (GLM
  never violates in an uncompressed 5.9k-token context; the same
  content with the policy omitted → 60%), compactor × agent grid,
  dose-response over summary budget (300→15 words: survival 88%→23%,
  violation 7%→28%), four compaction strategies (recency-truncate
  worst at 38%; only head_tail preserves the policy).
- **Constraint Pinning**: governance constraints held in a pinned
  buffer exempt from compaction, re-injected verbatim after every
  compaction step, integrity-checked each turn. Training-free,
  harness-local.
- Real-harness validation: LangGraph summarization node 0→65%
  violation, LangMem SummarizationNode 95%, AutoGen
  BufferedChatCompletionContext (recency eviction) 100%, OpenAI
  Agents SDK Runner 35%; Microsoft Agent Framework preserves system
  messages (consistent with the channel analysis).

## Results

- Pooled: control 0% → compaction 30% (DeepSeek-V4 59%, Kimi-K2.5
  59%, GPT-5.4-mini 41%, Qwen3.6 30%, Claude-4.6 19%, Gemini-3.5f 4%,
  GLM-5.1 0%). Higher-powered replication: +37 pts (95% CI [31, 43]).
- Compacting a policy can be worse than never stating it (DeepSeek
  59% vs. 37% floor) — the summary normalizes the pending task while
  discarding the rule.
- No model is safe on both axes: GLM/Gemini resist passive compaction
  but fall to active attacks (22%/19%); Claude resists the fixed
  injection but not passive decay (19%).
- Pinning: 0% across all seven models and both attack variants, at
  <0.5% token overhead, with 99% completion of policy-*allowed*
  actions (1% over-refusal — no utility cost).
- Pinning's residual hole: operator-impersonation *in the recent,
  unsummarized context* ("OPERATOR POLICY UPDATE… supersedes pinned
  policies") raises naive pinning 0%→17%; provenance hardening only
  halves it (→10%). Closing it requires a trusted out-of-band
  operator channel — authority cannot live in the token stream.
- Decay compounds over repeated compaction (0%→78% across 4 rounds)
  and reproduces cross-lingually.

## Critique / open questions

- Single author, and survival scoring leans on LLM judges (though
  triple-judge majority and a keyword heuristic agree; the primary
  violation metric is deterministic).
- Soft tasks are all single-tool-call refusal scenarios; whether
  pinning scales to constraints that require multi-step reasoning to
  apply ("extractable as a quotable rule" is an admitted limitation).
- The pinned buffer is itself trusted state — who may write to it?
  The operator-impersonation residual shows the write-path is the
  next attack surface, which is [[concepts/verified-memory-writes]]'s
  territory and louck2026securing's origin-binding theorem restated
  at the harness layer.
- Fictional-sandbox evaluation (tool calls parsed, never executed);
  effects-grading is robust but environment feedback loops are absent.

## Trust signals

- **Credibility:** 3 — single-author preprint (Beijing Institute of
  Technology), no peer review yet, but unusually strong internal
  validity: deterministic primary metric, 1,323 episodes with
  replication CIs, mechanism ablations, cross-family judge
  triangulation, and validation in four production frameworks; paper
  states all scenarios/prompts/grader code are released (URL not
  given in the PDF body).

## Follow-up

- **Relevance:** 5 — seeds [[concepts/constraint-pinning]] (the
  phenomenon + defense are attested independently by the concurrent
  works it cites: Gamage 2026 omission-decay, Santos-Grueiro 2026
  SafeContext, Dente et al. 2026 constraint decay in code-gen) and
  supplies the measured negative result the enforcement cluster has
  been arguing from principle: prose policy in evictable context is
  not enforcement. Directly cites and complements
  [[literature/papers/semenov2026beyond]] — structured eviction
  changes *what* gets dropped; this paper shows *why* the answer
  must never be the policy.
- The operator-impersonation residual is a second, harness-level
  attestation of louck2026securing's "authority must not be derivable
  from in-stream content" — worth citing when typed-enforcement
  matures.
- For this project's own harness: check whether the SessionStart
  hook + CLAUDE.md re-injection pattern already implements de-facto
  pinning (it does for project rules; per-session standing
  instructions are the exposed channel).
