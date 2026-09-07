---
kind: paper
title: "ACLE-MCP: Attested Capability Leases for Execution-Time Trust in Remote LLM Tool Use"
authors: ["Zhiyang Ding", "Yang Luo", "Guangpu Chen", "Qingni Shen", "Zhonghai Wu"]
institutions: ["Peking University"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.02690"
code_url: null
citations: null
source: "raw/papers/ding2026acle.pdf"
added: "2026-09-07"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
tags: ["safety", "governance", "enforcement", "permission-gate", "tool-use", "agent-architecture", "runtime-approval"]
---

# ACLE-MCP: Attested Capability Leases for Execution-Time Trust in Remote LLM Tool Use

## TL;DR

OAuth answers *whether a client may access a resource*, not *whether the
workload that ends up executing the call is still the one you agreed to
trust*. ACLE-MCP names this the **post-authorization execution trust gap**
and closes it by binding authority to workload state at invocation time,
with a provider-side Execution Gate that consumes a short-lived lease
immediately before the protected tool logic runs.

## Claims

- "OAuth authorization alone does not ensure that a later tool call is
  executed by the provider-side workload that the relying party intended to
  trust." An endpoint may stay authorized after execution shifts to a
  substituted workload, relies on stale appraisal, reuses authority
  transferred from another sender, or traverses an undeclared downstream
  component.
- Attestation obtained at registration or connection time **goes stale
  before the authority is consumed** — so the check must sit at execution
  time, not connect time.
- Parameter and operation bounds are explicitly *not* the central problem;
  they exist so that authority issued after workload appraisal "cannot be
  widened when consumed."

## Methods

- Invocation-scoped architecture coupling delegated authorization, workload
  appraisal, and resource-side execution admission.
- For protected calls, issues a short-lived **sender-constrained capability
  lease** binding expected workload, freshness requirement, operation,
  object and parameter bounds, downstream constraints, and receipt
  obligations.
- Provider-side **Execution Gate** consumes the lease immediately before
  protected tool logic begins.
- Runnable prototype: Keycloak/OIDC validation, MCP Python SDK server,
  optional vTPM quote-verification backend.
- Controlled security experiments comparing weaker authorization and
  connect-time attestation modes against full ACLE-MCP, plus an agent
  tool-use extension.

## Results

- Weaker authorization and connect-time attestation modes each leave
  **distinct** post-authorization attack families open; full ACLE-MCP blocks
  all evaluated families while preserving all benign tasks.
- Locally simulated agent extension: request-level pooled **p95 latency
  +25.7%** on normal allowed calls relative to OAuth-only.
- Motivating scenario is concrete: an agent authorized to `update_ticket` on
  `T-17` where the provider rolls the serving container back to a vulnerable
  image, routes through an undeclared proxy, or accepts a lease copied from
  another sender.

## Critique / open questions

- "All evaluated attack families" is the authors' own taxonomy; no adaptive
  adversary and no independent red-team.
- Evaluation is locally simulated, so the 25.7% p95 figure is a lower bound
  — real remote deployments add network and attestation-service round trips.
- vTPM backend is optional, which is where most of the actual trust would
  come from; the paper does not separate results with and without it
  clearly enough to tell how much rests on hardware attestation.
- No released code despite a "runnable prototype."

## Trust signals

- **Credibility:** 3 — Peking University (established security group,
  corresponding author Yang Luo), a working prototype against real
  components (Keycloak/OIDC, MCP Python SDK, vTPM), and a reported
  overhead number rather than only a success rate. Held at 3 by: arXiv
  preprint with no peer review, no released artifact, no citations, and a
  self-defined attack taxonomy.

## Follow-up

- **Relevance:** 4 — a sixth distinct placement for
  [[concepts/enforcement-boundary-placement]]: **resource-side, at
  execution time**, downstream of everything the agent's own harness
  controls. The concept currently spans placements from the system prompt
  outward to an external reference monitor; this adds the case where the
  boundary sits inside the *tool provider*, past the point the calling
  agent can influence at all.
- Its central argument — that a check valid at connect time is stale by
  consumption time — is a temporal axis the concept does not yet name.
  Placement has a *when* as well as a *where*, and this is the first source
  to make that the whole point.
- Gives [[concepts/permission-gate-as-architecture]] its first quantified
  cost (+25.7% p95). Across 24 sources that concept has argued the gate is
  necessary without pricing it; a number makes the tradeoff arguable.
- Read with [[literature/papers/zheng2026continuity]] (same week, same
  problem class, different answer): CONTINUITY distributes the boundary
  across all interfaces, ACLE-MCP concentrates it at the last one before
  execution.
