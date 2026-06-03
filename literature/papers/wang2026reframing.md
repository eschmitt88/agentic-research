---
kind: paper
title: "Reframing LLM Agent Security as an Agent-Human Interaction Problem"
authors:
  - Peiran Wang
  - Ying Li
  - Yuan Tian
institutions:
  - University of California, Los Angeles (UCLA)
year: 2026
venue: arXiv (preprint, cs.CR)
peer_reviewed: false
url: https://arxiv.org/abs/2605.24309
code_url: null
citations: null
source: "raw/papers/wang2026reframing.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/context-eviction-policy]]"
tags: [agent-security, permission-gate, runtime-approval, industry-academia-gap, ucla]
---

# Reframing LLM Agent Security as an Agent-Human Interaction Problem

## TL;DR

A systematic analysis of 59 academic papers, 21 production agent systems, and 26
security plugins (as of April 2026) arguing that agent security is fundamentally an
agent-human interaction problem: industry converges on human-centric mechanisms
(policy specification, runtime approval, scope configuration) while academia favors
intent anchoring and trust labeling that see zero deployment. (Skim from abstract.)

## Claims

- The three widely deployed human-centric mechanisms each appear in ≥14 of 21
  production systems (policy spec 14, runtime approval 15, scope config 16) — a
  quantitative attestation that runtime-approval / permission-gating is the dominant
  industry mechanism.
- The most-researched academic approaches (intent anchoring, trust labeling) see zero
  production deployment — a pronounced industry-academia mismatch.
- Human participation in agent security decisions is indispensable given current
  capabilities, despite an approval-fatigue vs. uncontrolled-autonomy trade-off.

## Methods

- Systematic comparison of LLM-based vs. human-based intent alignment across the
  three corpora (academic papers, production systems, security plugins).

## Results

- Quantified deployment counts (14/15/16 of 21 systems) backing runtime approval and
  policy specification as primary mechanisms.
- A three-direction research agenda calling for AHI security as a first-class area.

## Critique / open questions

- A position-and-survey paper, not a new mechanism; its value is the quantitative
  attestation, not a buildable artifact.
- Skim only; "as of April 2026" snapshot will date as the production landscape shifts.

## Trust signals

- **Credibility:** 4 — UCLA (Yuan Tian's group); preprint, not peer-reviewed; no
  code (survey/position). Large, explicitly-counted corpus (59+21+26) raises
  confidence as an attestation; not peer-reviewed caps it below 5.

## Follow-up

- This is the fourth attestation for permission-gate-as-architecture (with
  liu2026dive's Claude Code permission system, OpenHarness, and Hermes) — the cleanest
  quantitative formulation yet. Parent agent handles concept seeding/back-linking.
- The approval-fatigue vs. autonomy trade-off is the design tension `budget-as-ceiling`
  and this framework's `respects:` declarations both navigate.
