---
kind: paper
title: "Governance Architecture for Autonomous Agent Systems: Threats, Framework, and Engineering Practice"
authors: ["Yuxu Ge"]
institutions: ["University of York"]
year: 2026
venue: arXiv (cs.CR)
peer_reviewed: false
url: https://arxiv.org/abs/2603.07191
code_url: null
citations: null
source: "raw/papers/ge2026governance.pdf"
added: "2026-07-14"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [safety, governance, permission-gate, intent-verification, prompt-injection, skill-plugins, audit-log, defense-in-depth]
---

# Governance Architecture for Autonomous Agent Systems: Threats, Framework, and Engineering Practice

## TL;DR

Proposes LGA, a four-layer defense-in-depth governance stack for agent
systems — L1 OS-level execution sandbox, L2 LLM-judge intent verification
per tool call, L3 zero-trust inter-agent capability tokens, L4 immutable
audit log — and quantifies L2 on a 1,081-sample bilingual tool-call
benchmark: LLM judges intercept 93–98.5% of injection/poisoning attacks
(NLI baselines <10%), but a base-rate analysis shows judge precision
collapses at realistic attack prevalence, so the judge cannot be the sole
gatekeeper.

## Claims

- Existing defenses (Llama Guard, NeMo Guardrails) operate at the
  *text-generation* layer; execution-layer threats — unauthorized tool
  invocations from benign-looking text — pass through. Governance must
  intercept at the tool-call level.
- Three execution-layer threat classes are formalized: TC1 prompt
  injection, TC2 RAG poisoning, TC3 malicious skill plugins (runtime
  behavior exceeds self-declared permissions).
- Governance boundaries are architectural invariants ("specified by
  system architects, not learned from data"): model capability
  improvements cannot substitute for OS-level isolation or cryptographic
  authorization, because their correctness derives from legal/contractual
  requirements, not optimizable objectives.
- Isolation belongs at the OS layer, not the software layer —
  software-layer access controls are subject to semantic manipulation.
- TC3 is structurally harder for intent-alignment judges (75–94% IR vs
  ~92–100% on TC1/TC2): a malicious plugin performs its declared
  operation *plus* a covert side-channel, so the call is partially
  consistent with the task.

## Methods

- LGA layers: L1 sandbox (seccomp/containers; prototype uses `unshare`
  namespaces), L2 judge `Verify(a,T) = J(ρ(a,T)) ∈ {ALLOW, BLOCK}` on
  each high-risk call against the original task description, L3
  min-privilege capability tokens with TTL + HMAC-SHA256 + JSON-schema
  validation between agents, L4 append-only `fsync` audit log.
- Benchmark: 1,081 bilingual samples (Chinese original, machine-translated
  English) over 4 tool types — 504 benign, 457 TC1/TC2 malicious, 120 TC3
  plugin calls. Generated with GPT-4o-mini; labels verified by a 10%
  single-inspector sample (the author).
- Judges: BART-MNLI / mDeBERTa-NLI baselines, four local LLMs
  (Qwen3.5-4B/9B, Llama-3.1-8B, Qwen2.5-14B via Ollama on an RTX 5080),
  GPT-4o-mini cloud, plus two post-hoc two-stage cascades.
- OpenClaw case study: static compliance analysis of an open-source local
  agent framework against the four layers.
- McNemar significance tests on all pairwise judge comparisons; PPV
  base-rate analysis under π ∈ {50%, 5%, 1%} attack prevalence; external
  generalization on InjecAgent (n=100).

## Results

- All five LLM judges ≥93% overall IR on TC1/TC2; zero-shot NLI <10% —
  textual entailment is insufficient for tool-call authorization.
- Best local balance Qwen2.5-14B (98.2–98.5% IR, 9.7–20.1% FPR); cloud
  cascade (Qwen3.5-9B→GPT-4o-mini) 91.9–92.6% IR at 1.9–6.7% FPR; fully
  local cascade (→Qwen2.5-14B) 94.7–95.6% IR at 6.0–9.7% FPR — a viable
  data-sovereign deployment.
- Non-judge layers are nearly free: L1+L3+L4 add ~5–18 ms total; the LLM
  judge dominates latency (~200–2,200 ms; output-token capping gives a
  6.5× speedup at negligible IR cost). End-to-end 4-layer pipeline: 96%
  IR, P50 980 ms.
- **Base-rate collapse**: at 1% attack prevalence even the best single
  judge yields 22.7% PPV — 3 of 4 blocks are false alarms. The paper's own
  conclusion: L2 must not be the sole gatekeeper; the sandbox is the
  always-on containment boundary and judge escalations get human review
  or rollback, not silent blocking.
- TC3 per-subtype: privilege escalation best-detected (86.7–93.3%);
  exfiltration and persistence hardest (46.7–96.7%) — covert
  side-channels blend into legitimate-looking telemetry fields.
- Hand-crafted adversarial probe (30 samples) drops IR to 50–63%
  (authority-injection evasions hardest at 30–50%) — the black-box
  non-adaptive threat model is a real boundary, not a formality.
- InjecAgent generalization: 99–100% judge-level interception, evidence
  the capability is not overfit to the synthetic benchmark.
- OpenClaw compliance: SOUL.md behavioral constraints are soft semantics
  (L2: None), plugin permissions self-declared with no runtime
  verification (G2), no inter-agent authorization (L3: None), mutable
  `logs/` directory (G4).

## Critique / open questions

- Single-author preprint; no code or benchmark release (an open-source
  LGA reference implementation is listed as future work), so nothing is
  independently runnable.
- The benchmark is synthetic and GPT-4o-mini-generated while GPT-4o-mini
  is also a judge; the circularity analysis (Qwen3.5-9B matches or beats
  it) bounds the IR bias but the FPR-side bias is acknowledged as
  unbounded without regenerating the benchmark with an independent model.
- English set is machine-translated; a native-English TC3 ablation shows
  11–17 pp lower IR, so translated-EN numbers likely flatter the judges.
- L3 and L1 are microbenchmarked for latency but never attacked — only
  L2 gets adversarial evaluation; the defense-in-depth claim for TC3
  ("L1/L3 compensate for L2's weakness") is argued, not measured.
- The adaptive-attacker gap is explicitly out of scope, and the
  preliminary evasion probe suggests it is the binding constraint.

## Trust signals

- **Credibility:** 2 — single-author arXiv preprint from a group with no
  agents-security track record, no released code or data, synthetic
  self-verified benchmark; partially offset by unusual methodological
  transparency (McNemar tests, circularity and base-rate analyses, a
  seven-item limitations section) — but nothing is reproducible today.

## Follow-up

- **Relevance:** 4 — strengthens [[concepts/permission-gate-as-architecture]]
  with the first *quantified* judge-as-gate benchmark in the cluster and
  a base-rate (PPV) argument that independently supports
  madatha2026deterministic's deterministic-core thesis from the
  statistics side: a probabilistic judge behind a realistic attack prior
  mostly blocks legitimate work, so hard invariants belong in
  deterministic layers.
- The risk-stratified routing policy (fast NLI pre-filter for low-risk
  calls, LLM judge only for high-risk) is the same shape as Claude Code's
  permission classifier — a deployable pattern worth citing when the
  permission-gate concept graduates to a system proposal.
- TC3's formalization (declared permission set vs runtime behavior) gives
  [[concepts/shared-skill-namespace]] its missing threat model: a shared
  skill registry with self-declared permissions is exactly the G2 gap.
- Watch for the promised open-source LGA reference implementation and
  venue acceptance; revisit `credibility` if either lands.
