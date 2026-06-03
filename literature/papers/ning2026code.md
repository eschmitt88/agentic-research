---
kind: paper
title: "Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems"
authors:
  - Xuying Ning
  - Katherine Tieu
  - Dongqi Fu
  - Tianxin Wei
  - Zihao Li
  - Jiaru Zou
  - Mengting Ai
  - Zhining Liu
  - Ting-Wei Li
  - Bingxuan Li
  - Cheng Qian
  - Gaotang Li
  - Xiao Lin
  - Zhichen Zeng
  - Yifan Sun
  - Xiyuan Yang
  - Ruida Wang
  - Rui Pan
  - et al. (42 authors)
institutions:
  - University of Illinois Urbana-Champaign
  - Meta
  - Stanford University
year: 2026
venue: arXiv (survey, cs.CL)
peer_reviewed: false
url: https://arxiv.org/abs/2605.18747
code_url: https://github.com/YennNing/Awesome-Code-as-Agent-Harness-Papers
citations: null
source: "raw/papers/ning2026code.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [survey, agent-harness, plan-execute-verify, multi-agent, code-as-harness, uiuc]
---

# Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems

## TL;DR

A 42-author survey framing code as the foundational infrastructure layer for agents,
organized into three layers — harness interface, harness mechanisms
(planning/memory/tool use), and scaling to multi-agent systems — with
Plan-Execute-Verify loop semantics over sandboxed, permissioned execution and
deterministic/human-review verification gates. (Skim from abstract + PDF intro.)

## Claims

- Code is foundational agent infrastructure, not just an output modality —
  "executable, verifiable, stateful" is the organizing thesis.
- Three-layer organization (interface / mechanisms / scaling) is a candidate
  top-level frame for a harness Map of Content.
- Plan-Execute-Verify with sandboxed/permissioned execution and human-review gates
  synthesizes permission-gate-as-architecture and executable-guardrail framings at
  the survey level.

## Methods

- Literature survey; companion GitHub paper collection
  (github.com/YennNing/Awesome-Code-as-Agent-Harness-Papers).

## Results

- Coverage across coding assistants, GUI/OS automation, embodied agents, scientific
  discovery, personalization/recommendation, DevOps, enterprise workflows
  (no empirical result of its own).

## Critique / open questions

- Forty-two authors and survey breadth mean the three-layer frame is consensus-
  shaped; verify it isn't just a taxonomy of convenience before adopting as the MoC
  spine.
- Skim only — the execution (planning/memory) and scaling (multi-agent) sections are
  the parts to read for `hierarchical-delegation`.

## Trust signals

- **Credibility:** 4 — large multi-institution effort (UIUC-led, incl. Meta);
  preprint survey, not peer-reviewed; maintained companion paper collection released.
  Breadth and author count make it the strongest single literature attestation for
  the harness-architecture cluster this cycle; not peer-reviewed caps it below 5.

## Follow-up

- Strongest single fetch toward promoting the harness-architecture cluster to a MoC;
  the three-layer organization is a candidate MoC spine.
- Read the verification section: deterministic sensors + human-review gates is a
  `programmable-evaluator-oracle` + permission-gate synthesis worth anchoring.
