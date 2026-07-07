---
kind: paper
title: "AgenticOS: An Intent-Oriented Secure Operating System Architecture for Autonomous AI Agents"
authors: ["Zhen Zhao", "Yu Zhang", "Yanpeng Zhu", "Jia Wang", "Songqiao Tao", "Xin Cheng", "Jiexin Gao"]
institutions: ["Tencent (Technology Shanghai / Shenzhen Computer System / Cloud Computing)", "South-Central Minzu University"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.21129
code_url: null
citations: null
source: "raw/papers/zhao2026agenticos.pdf"
added: "2026-07-07"
relevance: 3
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: [governance, permission-gate, capability-security, os-architecture, least-privilege, information-flow, budgets]
---

# AgenticOS: An Intent-Oriented Secure Operating System Architecture for Autonomous AI Agents

## TL;DR

Design paper reconstructing the OS from a "resource manager" into an
"intent filter" for autonomous agents: agents never touch POSIX
primitives; they submit a structured intent Manifest, the system
synthesizes a least-capability environment from it, and all external
effects flow through mandatory mediation, auditing, and information-flow
constraints — with resource budgets bound directly into capability tokens.

## Claims

- The security boundary for highly autonomous agents should be the
  *semantic* question "does this external effect conform to the declared
  task intent?", not the low-level "may this process access this
  resource?" — POSIX-style compositional closure (open/socket/exec/fork)
  lets a compromised agent combine individually-legal primitives into
  behavior far beyond the authorized task.
- Prompt filtering, tool allowlists, and application-layer sandboxes are
  insufficient for long-running multi-tool agents; the boundary must be
  lowered into the OS capability model. Sandboxing alone can't stop
  "legal capabilities abused in combination" — that needs intent
  mediation, information-flow labels, and capability-composition analysis.
- Explicit threat model: fully compromised agent process, supply-chain
  poisoning, tool-output poisoning, capability-composition attacks, covert
  channels. The design assumes the agent is *not* trustworthy and that no
  LLM can reliably identify all malicious instructions — a compromised
  agent still cannot cross the semantic capability boundary.

## Methods

- Four one-way layers: **Ghost Kernel** (minimal trusted isolation root —
  encrypted per-capsule memory, deterministic time-slice scheduling to
  shrink timing covert channels, measurement/attestation root; "no exposed
  interface"); **Logic Shutter** (intent validation, policy mediation,
  unforgeable capability tokens bound to agent identity + Manifest +
  request chain + **resource budget**, information-flow labeling,
  immutable audit events); **Agent Capsule** (Manifest-Only Runtime: the
  capsule's ABI is generated from its Manifest — undeclared capabilities
  have no interface at all; no fork/exec/mmap/socket); **Semantic Boundary
  Gateway** (the only path to the external world: protocol proxying,
  gateway-hosted credentials, schema-validated semantic channels, output
  normalization/size-padding/pacing against exfiltration and covert
  channels).
- **Intent ABI**: six interface families (e.g. FetchJSON, UploadArtifact,
  SendMessage) with a non-generalizability principle — composition must
  not re-express arbitrary I/O. POSIX semantics permanently excluded.
  Network permissions reified as ⟨domain, path prefix, method, data type,
  budget⟩ five-tuples; Manifests declare inference budgets, data-retention
  policies, and human-confirmation points for high-risk effects.
- **Skill admission**: AgentOS-native Skills go through generation,
  registration, and admission; agents cannot bypass admission or
  self-expand capabilities at runtime.
- Weaver-based dynamic capability generation synthesizes the per-capsule
  interface table at startup.

## Results

- None — a pure architecture/position paper. No implementation,
  evaluation, or measurements; the paper explicitly declines to elaborate
  a specific engineering implementation (suggests WASM/eBPF/CFI/CHERI as
  candidate capsule substrates).

## Critique / open questions

- Design-only: no prototype, no attack-defense evaluation, no performance
  cost data. The hard part — whether a semantic Intent ABI can stay
  "non-generalizable" under real workload pressure without strangling
  agent usefulness — is asserted, not demonstrated.
- Intent recognition in the Logic Shutter still has to parse and validate
  declared intent; the paper keeps this structural (Manifest schema, not
  NL inference), but Manifest expressiveness vs least-privilege tightness
  is unexplored.
- The "trusted policy inputs" assumption (Manifests, human approvals,
  registration info) relocates rather than solves the governance problem
  for the policy-authoring layer.

## Trust signals

- **Credibility:** 2 — sizable corporate team (Tencent, multiple
  subsidiaries) lends institutional weight, but it is a design-only arXiv
  preprint: no peer review, no implementation or code, no evaluation, no
  citation record.

## Follow-up

- **Relevance:** 3 — cite-worthy architectural attestation that links two
  governance concepts (permission gate as the *only* action path; budgets
  reified inside capability tokens rather than as loop-halting conditions)
  and adds a skill-admission gate, but design-only with no evidence, so it
  sharpens framing more than it shifts any concept.
- The budget-in-the-capability-token move (⟨…, budget⟩ five-tuple,
  per-Manifest inference budgets) is the admission/runtime-primitive
  reading of [[concepts/budget-as-ceiling]] — worth citing if that concept
  ever grows an "enforcement locus" section.
- If the governance cluster ripens toward a MoC, this + 
  [[literature/papers/madatha2026deterministic]] +
  [[literature/papers/jia2026finharness]] span the
  deterministic-vs-semantic gate axis; DEMM-Bench (2606.20634, declined
  2026-07-07) would be the evaluation anchor to fetch then.
