---
kind: paper
title: "Your Agent's Memories Are Not Its Own: Forged Reasoning Attacks on LLM Agent Memory and Defenses"
authors: ["Neeraj Karamchandani", "Piyush Nagasubramaniam", "Sencun Zhu", "Dinghao Wu"]
institutions: ["The Pennsylvania State University"]
year: 2026
venue: "arXiv preprint (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.05029"
code_url: null
citations: null
source: "raw/papers/karamchandani2026your.pdf"
added: "2026-07-20"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [memory, write-policy, security, memory-poisoning, provenance, verification]
---

# Your Agent's Memories Are Not Its Own: Forged Reasoning Attacks on LLM Agent Memory and Defenses

## TL;DR

Identifies an agent's *reasoning store* (decision logs, rationales,
self-reflections) as a distinct memory-poisoning attack surface: FARMA
injects forged "prior validation already done" traces and amplifies them
into false precedent, hitting up to 100% attack success; SENTINEL, a
5-layer write-path defense whose Reasoning Guard structurally scores
candidate entries, drops success to 0% with zero false positives on 326
benign traces.

## Claims

- Prior memory-poisoning work (AgentPoison, MINJA, MemoryGraft,
  PoisonedRAG) targets retrieved knowledge/examples/experiences; the
  agent's *own remembered reasoning* is an unexplored, higher-trust
  surface — retrieved rationales are treated as evidence of the agent's
  own past work, so a forged "validation complete" trace makes the agent
  skip the step *because its memory says it already did it*.
- Two-phase attack: (1) seed injection of evasively-worded,
  structurally-mimetic decision-log entries (no "skip"/"bypass" tokens);
  (2) amplification — new forged entries cite earlier ones ("consistent
  with N prior runs"), which raises retrieval probability, becomes the
  consensus (defeating consensus-deviation defenses like A-MemGuard),
  and manufactures precedent.
- Defense must run **at write time, not only retrieval time** — SENTINEL's
  first design principle. Entries are screened before commit: keyword
  filter → provenance/IFC taint labeling → taint threshold → pattern/risk
  screen → Reasoning Guard (5 weighted structural signals: provenance
  anomaly, self-reference mismatch, suspicious pattern density,
  implausible claim checks, structure check).
- Ablation: the Reasoning Guard alone is necessary and sufficient against
  FARMA (all other layers removed → 0% ASR; guard removed → 100%); the
  lightweight layers only cover orthogonal/naive variants.

## Methods

- Threat model: attacker has memory-write access (compromised plugin,
  shared store, insider) but cannot control the agent, observe execution,
  or modify weights/prompts. Metadata spoofing allowed (source=AGENT,
  trust=VERIFIED).
- Eval: 3 agent domains with distinct memory-usage patterns (EHRAgent
  decision-log reuse, ReAct-QA reasoning-chain reuse, RAP
  preference-pattern reuse) × GPT-4o-mini/GPT-4o/Llama-3.3-70B, 50 trials
  per cell, Wilson CIs; differential clean-vs-poisoned protocol so only
  attack-caused behavior change counts.
- Reasoning Guard is deliberately heuristic (regex/scoring, <1 ms
  per-write) so it can sit synchronously on the memory-write path without
  a second LLM call.

## Results

- FARMA: 100% ASR on EHRAgent across all three models and against
  KeywordFilter and A-MemGuard (both heuristic and LLM-judge modes);
  52%/48% on ReAct-QA/RAP undefended. Binary-decision domains
  (import-vs-validate) are most susceptible; domains where competing
  evidence is visible at decision time resist better.
- SENTINEL: 0–6% ASR across domains; 0% FPR across 326 benign traces at
  τ_guard = 0.5.
- Amplification is what kills consensus defenses: flooding makes forged
  traces the cluster center, so outlier detection inverts.

## Critique / open questions

- A paraphrased adaptive attacker that knows SENTINEL's patterns already
  defeats the Reasoning Guard (authors admit "did not provide significant
  protection") — the heuristic defense is brittle; the arms race is one
  step deep. LLM-as-judge in the guard is future work.
- Simulated environments only; single-agent, single-store. The multi-agent
  shared-memory case (one compromised agent's writes become another's
  reads) is named but unevaluated.
- No code released, which limits reproducibility of an otherwise careful
  eval (differential protocol, CIs, ablations).
- The store separation (reasoning store vs. general memory store) does
  real defensive work here — `looks_like_reasoning` reclassification
  closes the cross-store evasion — which quietly supports typed
  partitioning of memory as an architectural choice, not just an
  implementation detail.

## Trust signals

- **Credibility:** 3 — established security group (Penn State; Zhu/Wu),
  careful differential eval with CIs and ablations, but arXiv-only, no
  code, no citations yet.

## Follow-up

- **Relevance:** 4 — first adversarial attestation for
  [[concepts/verified-memory-writes]]: prior sources motivate write-time
  gating from benign drift (TrustMem's coverage/preservation/faithfulness,
  Mem0's ADD/UPDATE/DELETE/NOOP); this paper shows the write path is also
  the *security* boundary and that retrieval-time defense structurally
  fails against amplified writes.
- Watch for: the OWASP Agentic Top-10 2026 entry (ASI06 memory/context
  poisoning) as a practice-side anchor; A-MemGuard and MemoryGraft as the
  adjacent attack/defense pairs if the memory-security thread thickens
  toward its own concept.
