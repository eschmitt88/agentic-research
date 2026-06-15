---
kind: paper
title: "EurekAgent: Agent Environment Engineering is All You Need for Autonomous Scientific Discovery"
authors: ["Amy Xin", "Jiening Siow", "Junjie Wang", "Zijun Yao", "Fanjin Zhang", "Jian Song", "Lei Hou", "Juanzi Li"]
institutions: ["Tsinghua University", "Renmin University of China"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.13662
code_url: https://github.com/THU-Team-Eureka/EurekAgent
citations: null
source: "raw/papers/xin2026eurekagent.pdf"
added: "2026-06-15"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/file-as-bus]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/hierarchical-delegation]]"
tags: [environment-engineering, permissions, artifact-management, budget, human-in-the-loop, reward-hacking, circle-packing, mle-bench]
---

# EurekAgent: Agent Environment Engineering is All You Need for Autonomous Scientific Discovery

## TL;DR

A position-plus-system paper arguing that as base agents (Claude Code, Codex)
get stronger, the bottleneck for autonomous discovery shifts from *prescribing
agent workflows* to *engineering the environment* — the resources, constraints,
and interfaces that shape agent behavior. EurekAgent engineers the environment
along four dimensions: **permissions** (bounded execution + isolated
evaluation), **artifacts** (filesystem + Git-based collaboration), **budget**
(budget-aware exploration), and **human-in-the-loop** (low-friction
supervision/intervention). It amplifies productive affordances (open-ended
exploration, accurate rewards, inter-agent coordination) and suppresses harmful
ones (reward hacking, evaluation tampering). Sets new SOTA on math (26-circle
packing, found for <$11 total API cost), kernel engineering (TriMul), and an
MLE-Bench ML subset (85.71%).

## Claims

- The bottleneck is shifting from agent-workflow design to **environment
  engineering**; much of the useful research capability already resides in a
  strong base agent given a clear task and an optimizable metric (cites
  ResearchClawBench showing general agents outperform research-specific
  scaffolds).
- Task performance alone does not make a reliable autonomous researcher:
  agents may contaminate evaluations, manipulate artifacts, or skip procedural
  constraints — observability/reward-hacking failures. The environment, not the
  prompt, is where you constrain this.
- Analogy (Gibson's affordances / a capable PhD student): productivity comes
  from accountability, autonomy, accurate feedback, and supervision, not
  minute-by-minute instructions.
- Four engineered dimensions — permissions, artifacts, budget, human-in-loop —
  are individually ablatable levers on agent reliability.

## Methods

- **Permissions engineering**: bounded agent execution and isolated evaluation
  (sandboxed scoring the agent cannot tamper with).
- **Artifact engineering**: a filesystem + Git-based workspace as the
  collaboration substrate for multiple agents.
- **Budget engineering**: budget-aware exploration with explicit cost tracking
  (headline: 26-circle packing SOTA for <$11).
- **Human-in-the-loop engineering**: low-friction interfaces for supervision
  and intervention.
- Evaluated on metric-driven tasks across mathematics, kernel engineering, and
  an MLE-Bench ML subset.

## Results

- Math: new SOTA on 26-circle packing (2.635999 vs prev. best AI 2.635986),
  Erdős minimum overlap, first-autocorrelation inequality.
- Kernel engineering: TriMul 2005.03 µs (beats prev. best 2096.04 µs human /
  2247.78 µs AI).
- Machine learning: 85.71% on the evaluated MLE-Bench subset (vs 71.43% prev).
- Circle-packing SOTA achieved at <$11 total API cost.

## Critique / open questions

- "All you need" framing overstates; the paper itself shows base-agent capability
  is the other half. The contribution is the taxonomy of environment levers, not
  evidence that workflow design is dispensable.
- Permissions = "isolated evaluation the agent can't tamper with" is the
  operational core of HCE (no test leakage / no reward hacking) recast as an
  environment property — a useful reframing of why the holdout must be
  externally enforced, not agent-honored.
- Per-dimension ablations are claimed; how much each lever independently
  contributes (vs the strong GPT-5.5-class backbone) is the number to verify.

## Trust signals

- **Credibility:** 4 — Tsinghua University (KEG/CS&T) + Renmin University of
  China, open-source code (github.com/THU-Team-Eureka/EurekAgent) and released
  results; arXiv preprint (v2 Jun 12 2026), not peer-reviewed.

## Follow-up

- **Relevance:** 5 — a clean multi-concept anchor. Artifact engineering
  (filesystem + Git collaboration) is `file-as-bus`; budget engineering is
  `budget-as-ceiling` (cost ceiling as a first-class environment lever);
  isolated/tamper-proof evaluation + reward-hacking suppression is
  `hce-evaluation`; inter-agent collaboration is `hierarchical-delegation`.
  Permissions engineering is also a fifth strong attestation for the
  not-yet-seeded **permission-gate-as-architecture** concept (cf. liu2026dive,
  wang2026reframing, the deferred FinHarness) — now well past the threshold to
  crystallize.
- "Environment engineering" is a candidate new concept in its own right (the
  environment, not the prompt or workflow, as the primary design surface);
  watch for a second independent attestation before seeding.
