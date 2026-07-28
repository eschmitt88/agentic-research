---
kind: paper
title: "Formal Policy Enforcement for Real-World Agentic Systems"
authors: ["Nils Palumbo", "Sarthak Choudhary", "Jihye Choi", "Guy Amir", "Prasad Chalasani", "Somesh Jha"]
institutions: ["University of Wisconsin–Madison", "Cornell University", "Langroid"]
year: 2026
venue: "arXiv preprint (v3)"
peer_reviewed: false
url: "https://arxiv.org/abs/2602.16708"
code_url: null
citations: null
source: "raw/papers/palumbo2026formal.pdf"
added: "2026-07-28"
relevance: 5
credibility: 4
status: read
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
related_experiments: []
tags: [policy, enforcement, datalog, formal-methods, multi-agent, security, aspect-oriented]
---

# Formal Policy Enforcement for Real-World Agentic Systems

## TL;DR

Policy enforcement is a **cross-cutting concern**, so treat it as aspect
weaving: specify policies in Datalog over abstract predicates, maintain
those predicates through an observability service bound by a formal
assume/guarantee contract, and consult a reference monitor at every
policy-relevant action. FORGE realizes this over *unmodified* agents and
eliminates policy violations by construction — prompt-injection attack
success 100% → 0%, τ²-bench compliance 58% → 98%, unauthorized accesses
40 → 0 — at 19–38% latency overhead.

## Claims

- Embedding natural-language policy in a system prompt and delegating
  compliance to the agent's reasoning "admits no formal enforcement
  guarantee." It is the dominant practice and it is unsound.
- Prompt-based policy also **cannot express history-dependent policy** —
  constraints whose satisfaction depends on the causal history of an
  execution. This gap becomes acute in multi-agent systems, where relevant
  events are only *partially ordered* and existing trace-based approaches
  (which assume linear traces) are ill-suited.
- Enforcement correctness can be stated as a theorem with an explicit
  precondition: **when the environment contract holds**, enforcement
  decisions coincide with the policy's intended semantics. The contract is
  what each deployment must discharge.
- Claimed first: a framework combining automatic enforcement of declarative
  authorization policies with formal correctness guarantees in the
  *multi-agent* setting, and the first supporting recursive queries.

## Methods

- **Aspect-oriented framing.** Policy is not woven into agent logic; it is
  an aspect applied at every policy-relevant join point. Consequence: it
  works over arbitrary agentic deployments *without modifying the agents*
  — the property that makes it deployable rather than a redesign.
- **Datalog as the policy language**, chosen for four properties that each
  do real work: declarative rules, recursion (needed for policies over
  transitive relationships), deterministic enforcement, and — the
  underrated one — **tractable static analysis for contradiction,
  redundancy, subsumption, and conditional reachability**, so a policy
  author can verify intent and surface the ambiguities latent in a
  natural-language specification *before* deployment. Stratified negation
  plus foreign functions.
- **Observability service under an assume/guarantee contract.** The paper
  identifies a limitation in existing observability systems: they were not
  built to be *sound and sufficient* on the facts a policy decision depends
  on. The contract requires exactly that, per (context, action) pair.
- **Reference monitor** enforcing complete mediation, consulted at each
  action to produce the enforcement decision.
- Threat model makes no assumptions about entity internals (agents may be
  LLMs, deterministic programs, or tool executors); the adversary may
  manipulate downstream entities and inject instructions via external input.
- `llm_check` foreign function for semantic predicates that Datalog cannot
  express structurally.

## Results

Three quantitative case studies plus two real-world deployments (OpenClaw,
VS Code Copilot Chat):

- **Prompt-injection / information-flow**: attack success 100% → 0%, with
  **no false positives** on benign workflows.
- **τ²-bench customer service**: policy compliance 58% → 98% across three
  frontier models.
- **MALADE multi-agent pharmacovigilance**: unauthorized FDA accesses
  40 → 0, with approval workflows expressed as policy.
- **Overhead**: end-to-end latency +19–38%; per-trial cost increase under
  $0.05.

## Critique / open questions

- The correctness theorem is conditional on the environment contract, and
  discharging that contract is deployment-specific work the paper
  acknowledges rather than automates. The guarantee is real but relocated:
  "enforcement is correct if observability is sound and sufficient" moves
  the trust burden into the observability layer.
- Authors flag that the concurrency assumption breaks for method bodies
  that internally launch concurrent tasks producing messages — a
  realistic pattern in agent frameworks.
- `llm_check` is the escape hatch and it is explicitly probabilistic; any
  policy needing semantic judgment inherits an unbounded false-negative
  rate. The formal guarantee covers the Datalog skeleton, not the semantic
  predicates hanging off it — worth being precise about when citing "formal
  enforcement."
- 19–38% latency is described as modest and is defensible for
  security-critical deployments, but it is not free, and the paper does not
  characterize how overhead scales with policy size or agent count.
- No FORGE artifact URL in the paper — hence `code_url: null`, despite two
  named real-world deployments.

## Trust signals

- **Credibility:** 4 — strong formal-methods and security provenance
  (Somesh Jha's group at UW–Madison, with Cornell and Langroid); a stated
  correctness theorem with an explicit contract rather than an informal
  claim; three quantitative case studies plus two real deployments; v3
  indicates revision. Held below 5 by the absence of peer review and of a
  released implementation.

## Follow-up

- **Relevance:** 5 — the formal anchor for the **typed-enforcement thread**
  the 07-20 NOTES flagged for concept ripeness (with khan2026token,
  zhao2026agenticos, louck2026securing, madatha2026deterministic, and
  mondl2026autoformalization). This is the source that states the thread's
  central claim as a theorem: prompt-resident policy has no enforcement
  semantics, and the fix is a declarative artifact checked by a reference
  monitor outside the agent's reasoning.
- Directly extends [[concepts/permission-gate-as-architecture]] along the
  axis madatha2026deterministic opened (the gate must be deterministic and
  testable, not further LLM orchestration) by supplying the *language* that
  determinism should be written in, plus static analyses that let a policy
  be checked before it ever runs.
- The **history-dependence** point is new to this graph. Every gate the
  concept has collected so far decides on the current action plus
  accumulated risk state; Datalog with recursion over a partially-ordered
  event set is a strictly more expressive gate, and multi-agent
  delegation — [[concepts/hierarchical-delegation]] — is exactly where
  point-in-time gates stop being sufficient.
- Most importable idea for this project as it stands: **static analysis of
  the policy itself**. This repo's rules (`~/.claude/rules/evaluation.md`,
  `agency.md`, `CLAUDE.md`) are natural-language policy whose
  contradictions and redundancies are found only when an agent hits them.
  Datalog's contradiction/subsumption/reachability checks are the
  machine-checkable analogue, and the gap between the two is precisely what
  [[literature/papers/mondl2026autoformalization]] tries to bridge.
- Note for `/elevate`: the honest read is that FORGE-style enforcement is
  far heavier than this single-operator box needs. What is *cheap* and
  worth stealing is the contradiction/subsumption check over the rule set,
  not the reference monitor.
