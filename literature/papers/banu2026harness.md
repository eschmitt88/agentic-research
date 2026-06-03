---
kind: paper
title: "Harness Engineering as Categorical Architecture"
authors:
  - Bogdan Banu
institutions: ["Independent"]
year: 2026
venue: "arXiv:2605.12239 [cs.PL]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.12239"
code_url: null
citations: null
source: "raw/papers/banu2026harness.pdf"
added: "2026-06-03"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/hybrid-model-backends]]"
tags:
  - harness-architecture
  - category-theory
  - formal-methods
  - compiler-functor
  - escalation
---

# Harness Engineering as Categorical Architecture

## TL;DR

Argues that the LLM agent harness (prompts, tools, memory, orchestration)
lacks any formal theory and proposes the categorical Architecture triple
(G, Know, Phi) from ArchAgents as that formalization, with a reference
compiler emitting harnesses across five frameworks.

## Claims

- Harness design is ad hoc; no formal theory governs composition,
  preservation under compilation, or cross-framework comparison.
- The four externalization pillars (Memory, Skills, Protocols, Harness)
  map onto the triple: Memory as coalgebraic state, Skills as
  operad-composed objects, Protocols as syntactic wiring G, Harness as the
  Architecture itself.
- Structural guarantees (integrity gates, quality-based escalation,
  convergence checks) are Know-level certificates preserved by structural
  replay (identity + verifier replay), not output-layer correctness.
- Quality-based escalation is model-parametric — the escalation control
  path operates independently of model choice.

## Methods

- Reference implementation with compiler functors targeting Swarms,
  DeerFlow, Ralph, Scion, and LangGraph; the LangGraph compiler emits one
  node per stage using the runtime's native per-stage method.
- An end-to-end escalation experiment with real LLM agents (two models,
  one task).

## Results

- Four configuration compilers preserve three named certificate types by
  identity/replay; LangGraph preserves the same via its shared per-stage
  path.
- Escalation confirmed model-parametric in the two-model, one-task setup.

## Critique / open questions

- The escalation claim rests on a single two-model, one-task experiment —
  thin empirical support for a strong "model-parametric" claim.
- Heavy category-theoretic framing; practical payoff over informal harness
  design is asserted more than demonstrated.

## Trust signals

- **Credibility:** 2 — independent author, preprint, no released code; the
  formalism is interesting but validation is minimal.

## Follow-up

- Fourth axis (compiler-functor-across-harnesses) for the
  harness-architecture cluster — relevant to whether that cluster reaches
  MoC threshold.
- Cross-read with claw-code / KAIROS as concrete harness instances.
