---
kind: paper
title: "PARNESS: A Paper Harness for End-to-End Automated Scientific Research with Dynamic Workflows, Full-Text Indexing, and Cross-Run Knowledge Accumulation"
authors:
  - Yuchen Wang
  - Zhongzhi Luan
institutions: ["Beihang University"]
year: 2026
venue: "arXiv:2605.05258 [cs.SE]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.05258"
code_url: "https://github.com/gtrhythm/PARNESS"
citations: null
source: "raw/papers/wang2026parness.pdf"
added: "2026-06-03"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/agent-native-memory]]"
tags:
  - research-harness
  - dag-workflow
  - full-text-indexing
  - knowledge-graph
  - cross-run-memory
  - open-source
---

# PARNESS: A Paper Harness for End-to-End Automated Scientific Research

## TL;DR

A framework for end-to-end automated research built on a thin DAG kernel
plus user-editable YAML workflows, full-text PDF indexing, and a
scenario-typed knowledge graph over papers/ideas/experiments/code — the
closest external analogue to this project's own skills + concepts +
literature + experiments architecture.

## Claims

- Existing autonomous-research systems each hard-code one control-flow
  shape (linear pipeline, state machine, single-agent loop, fixed skill
  pack), which is too rigid because real research workflows are dynamic
  and discipline-specific.
- A thin DAG kernel with a four-field Agent contract decouples scheduling
  from domain semantics, so any discipline's loop is expressible as
  user-editable YAML rather than orchestrator code.
- Full-text PDF parsing (bodies, figures, tables as typed objects) with
  abstract-only fallback grows reading capability monotonically with use.
- A scenario-typed knowledge graph (similar / contradictory / cross-domain
  / counter-intuitive retrieval) surfaces a focused slice of the cumulative
  corpus into each LLM call.

## Methods

- Four design moves: (i) DAG kernel + Agent contract; (ii) full-text
  literature-library subsystem; (iii) knowledge-graph index with
  scenario-typed retrieval; (iv) a small extension surface so any coding
  agent (Claude Code, Cursor, etc.) can swap modules without a plug-in.
- Open-source; appendix includes a verbatim end-to-end-generated paper.

## Results

- Demonstrated via the appendix's generated paper; no held-out benchmark
  reported in the abstract.

## Critique / open questions

- Self-reported demonstration rather than a comparative benchmark — hard to
  gauge quality of generated research vs. AI-Scientist or ResearchAgent.
- The "cumulative corpus does the work" argument depends on retrieval
  quality that the abstract does not quantify.

## Trust signals

- **Credibility:** 3 — preprint (not peer-reviewed), single mid-tier
  institution (Beihang), but open-source code released and an unusually
  detailed 31-page writeup with a worked example.

## Follow-up

- Compare PARNESS's DAG-kernel-plus-YAML design against this project's
  skill/concept architecture — validation or counter-design points.
- Inspect scenario-typed retrieval as a candidate refinement of
  `web-grounded-literature`.
