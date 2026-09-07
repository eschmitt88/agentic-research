---
kind: paper
title: "CONTINUITY: Security-Context Contracts for Composable LLM Agent Controls"
authors: ["Chris Zheng", "Geng Yang"]
institutions: ["ZAST.AI"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.05269"
code_url: "https://github.com/zast-ai/continuity"
citations: null
source: "raw/papers/zheng2026continuity.pdf"
added: "2026-09-07"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/information-firewall]]"
tags: ["safety", "governance", "enforcement", "agent-architecture", "provenance", "types", "formal-methods"]
---

# CONTINUITY: Security-Context Contracts for Composable LLM Agent Controls

## TL;DR

Agent stacks now chain several independently-correct controls — provenance
tracking, task-scoped authorization, policy gateways, protocol adapters,
effect-bound execution — and the *composition* fails even when each part
holds. CONTINUITY names this failure class **security-context
discontinuity** and answers it with assume-guarantee contracts requiring
every externally realised effect to carry a verifiable witness back to an
authenticated origin.

## Claims

- Controls "are commonly specified and tested in isolation. Their
  composition can nevertheless fail when a boundary drops a source label,
  accepts a self-declared authority root, widens a delegation, changes an
  approved field without a valid transformation relation, or executes a
  stale or replayed permit."
- **End-to-end consequence integrity (ECI):** every externally realised
  effect must have a verifiable witness connecting the exact effect to an
  authenticated chain origin, principal, task, field-level provenance, root
  grant, component contracts, current policy, finality sink, and
  single-use execution state.
- The enforcement boundary is therefore **not a chokepoint but a property
  of every component interface** — each transition must produce a
  verifiable artifact.

## Methods

- Assume-guarantee contract model plus a reference system enforcing: signed
  root grants; role-bound component identities; RFC 6901-style leaf paths;
  signed provenance and context manifests; bounded, source- and value-bound
  typed releases; independently verifiable transformation witnesses; and
  subject-, action-, policy-, revocation- and replay-bound finality permits.
- **1.5-KLOC trusted core.** Deterministic conformance suite: 32 cross-layer
  fault classes × 4 domains × 20 parameterized instances, plus 700 benign
  and 200 ambiguous tasks.
- Targeted ablations per omitted invariant.

## Results

- Across **3,460 scenarios / 24,220 system–scenario runs**: contains all 128
  fault–domain classes; **0 harmful effects in 2,560 attack instances**;
  completes all benign tasks; escalates all ambiguous tasks.
- The strongest *incomplete* reference configuration commits a harmful
  effect in **65.6%** of attack instances — the composition argument's
  main empirical support.
- Ablations reopen 4–24 fault–domain classes depending on the omitted
  invariant.
- Median proof verification 4.21 ms; median end-to-end transition
  production + verification + permit issuance + finality 7.17 ms.

## Critique / open questions

- The authors state the scope limit themselves and it is the important
  sentence: these are "deterministic conformance results under an explicit
  trusted-computing-base model, not an estimate of real-world attack
  probability or a claim to solve semantic correctness." Same caveat class
  as [[literature/papers/leong2026recognition]]'s reference monitor — the
  0/2,560 is correctness-by-construction, not adversarial robustness.
- 32 fault classes are the authors' own enumeration; nothing establishes
  coverage of the real fault space.
- Per-effect witness chains imply a real engineering tax beyond the quoted
  millisecond latencies (key management, manifest signing, component
  identity provisioning) that the paper does not cost.
- Commercial vendor publishing an evaluation of its own architecture.

## Trust signals

- **Credibility:** 3 — arXiv preprint, not peer reviewed, from a startup
  (ZAST.AI) publishing on its own system, so an obvious conflict of
  interest. Raised from 2 by a **released artifact**
  (`github.com/zast-ai/continuity`), a small auditable trusted core
  (1.5 KLOC), a deterministic and reproducible suite at real scale
  (24,220 runs), reported ablations, and an explicit self-stated scope
  limit.

## Follow-up

- **Relevance:** 4 — a distinct answer on the axis
  [[concepts/enforcement-boundary-placement]] exists to track. All seven
  of that concept's current sources name a *single* site for the boundary
  (reference monitor, skill artifact, runtime middleware, destination
  allow-list, provenance origin). This one argues the boundary is
  distributed across every component interface and that single-site
  placement is exactly what produces the discontinuity class. That
  disagreement is the concept's substance, so this materially extends it.
- The typed-release / transformation-witness machinery is
  [[concepts/typed-enforcement]] carried across component boundaries rather
  than applied at one gate.
- "Drops a source label" as a named fault class is
  [[concepts/information-firewall]]'s failure mode given a name and a test.
- Note for a future [[concepts/enforcement-boundary-placement]] revision:
  read alongside [[literature/papers/ding2026acle]] from the same week,
  which places the gate at execution time on the resource side. The two are
  compatible but answer "where" differently, which is the point.
