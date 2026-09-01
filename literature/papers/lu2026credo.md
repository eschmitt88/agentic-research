---
kind: paper
title: "Credo: Reusable Declarative Primitives for Agentic Workflows"
authors: ["Duo Lu", "Andrew Crotty"]
institutions: ["Brown University", "Northwestern University"]
year: 2026
venue: "arXiv (cs.AI) — vision/agenda paper"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.27790"
code_url: null
citations: null
source: "raw/papers/lu2026credo.pdf"
added: "2026-09-01"
relevance: 3
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/citation-anchoring]]"
tags: ["skills", "agent-architecture", "knowledge-organization", "provenance", "position-paper"]
---

# Credo: Reusable Declarative Primitives for Agentic Workflows

## TL;DR

A searched harness encodes real knowledge — which logical steps work, which
runtime signals matter, which prompt strategies are effective — but the
artifact is **an opaque block of imperative code**, so the next task starts
the search from scratch. Credo proposes recovering a **structured
declarative description** of a searched harness, tagging each extracted
primitive with metadata and **provenance**, and compiling stored primitives
into harnesses for new tasks. Explicitly a **vision paper with preliminary
results**.

## Claims

- An LLM application depends on both a model and a **harness** — the
  program deciding what each call sees, how many calls to make, and which
  answers to trust.
- Harness search produces reusable knowledge but stores it in a
  non-reusable form: logical steps, runtime signals, physical execution
  decisions, and prompt strategies all remain implicit and task-specific.
- The missing artifact is a **catalog**: primitives tagged with metadata and
  provenance, bindable by a compiler.
- This is a **database problem**, and the DB community is well positioned
  for it — cost-based compilation over declarative catalogs, and catalog
  maintenance under model and workload drift.

## Methods

- Extract structured declarative primitives from a searched harness.
- Tag each with relevant metadata; catalog all of it **with provenance**.
- A compiler binds stored primitives to generate harnesses for new tasks
  without restarting the search.
- The paper presents preliminary results and lays out a research agenda
  rather than a full system evaluation.

## Critique / open questions

- **This is a position paper and should be weighted as one.** The paper
  says so plainly ("provides preliminary results demonstrating the
  potential of our approach and lays out a related research agenda"), and
  the 08-31 digest's summary implied a working system it does not claim to
  be. No benchmark, no baseline, no released code.
- The central difficulty is asserted away: whether a searched imperative
  harness *can* be faithfully decompiled into recombinable declarative
  primitives is the whole question, and preliminary results are not
  evidence that it can at useful fidelity.
- "Catalog maintenance under model and workload drift" is correctly named
  as the hard open problem — a catalog of primitives tuned to one model
  generation may not survive the next.
- Two authors, no evaluation, no artifacts.

## Trust signals

- **Credibility:** 2 — reputable universities (Brown, Northwestern; Andrew
  Crotty is an established database researcher), and the framing is
  coherent and honestly labelled. But there is no system evaluation, no
  released code, no peer review, and no baseline: on the trust rubric this
  is a preprint from a small group with no reproducible artifact, and the
  self-described "preliminary" status is the dominant signal.

## Follow-up

- **Relevance:** 3 — useful prior art on an active theme that does not
  shift a concept. It names the **promotion step** that
  [[concepts/skill-library-lifecycle]] describes but does not mechanize —
  a one-off script becoming a catalogued primitive — and it is the only
  ingested source arguing that the promoted artifact should carry
  *provenance and metadata* rather than just being moved.
- The provenance requirement connects to [[concepts/citation-anchoring]]
  from an unexpected direction: a catalogued primitive should record which
  search produced it and on what task, for the same reason a claim should
  record its source.
- Held at relevance 3 rather than 4 deliberately.
  [[literature/papers/esakkiraja2026starharness]] (same curate pass)
  demonstrates harness evolution working, with released code and held-out
  generalization tests; Credo proposes what to do with the *output* of such
  a search but does not demonstrate it. Revisit if a full Credo system
  paper appears — the idea is good and the evidence is not yet there.
- The "catalog maintenance under model and workload drift" problem is worth
  carrying forward independently: this project's own skills were authored
  against particular model behaviour, and nothing tracks which of them
  encode assumptions that a model generation could invalidate.
