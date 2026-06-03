---
kind: paper
title: "Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems"
authors:
  - Jiacheng Liu
  - Xiaohan Zhao
  - Xinyi Shang
  - Zhiqiang Shen
institutions:
  - "VILA Lab, Mohamed bin Zayed University of Artificial Intelligence (MBZUAI)"
  - University College London
year: 2026
venue: arXiv (tech report, cs.SE)
peer_reviewed: false
url: https://arxiv.org/abs/2604.14228
code_url: https://github.com/VILA-Lab/Dive-into-Claude-Code
citations: null
source: "raw/papers/liu2026dive.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [agent-harness, claude-code, reverse-engineering, permission-gate, compaction, mbzuai]
---

# Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems

## TL;DR

A source-level reverse-engineering of Claude Code v2.1.88 (TypeScript) that traces
five human values through thirteen design principles to implementation, framing the
system as a simple while-loop wrapped in deterministic infrastructure: a seven-mode
permission system with an ML classifier, a five-layer compaction pipeline, four
extensibility mechanisms, subagent delegation, and append-oriented session storage.
Compares against OpenClaw. (Skim from abstract + PDF intro.)

## Claims

- The core is a simple while-loop (call model → run tools → repeat); most of the
  code lives in the *systems around* the loop — i.e. harness engineering, not model
  choice, is the locus of capability. (The companion repo states this quantitatively:
  98.4% deterministic infrastructure vs. 1.6% AI decision logic.)
- Five motivating values: human decision authority, safety/security, reliable
  execution, capability amplification, contextual adaptability.
- The permission system is a first-class component: seven modes plus an ML-based
  classifier — a strong attestation for permission-gate-as-architecture.
- The same recurring design questions yield different answers under a different
  deployment context (OpenClaw): per-action safety vs. perimeter access control,
  CLI loop vs. embedded runtime, context-window extension vs. gateway capability
  registration.

## Methods

- Analysis of publicly available TypeScript source (v2.1.88) plus a values →
  principles → implementation analytical framework; OpenClaw as comparison baseline.

## Results

- Identifies the five-layer compaction pipeline (a concrete `context-eviction-policy`
  reference), the four extensibility mechanisms (MCP/plugins/skills/hooks), and the
  subagent delegation/orchestration mechanism (a `hierarchical-delegation` datum).
- Names six open design directions for future agent systems.

## Critique / open questions

- Reverse-engineering of closed source: fidelity depends on the authors' reading of
  the TypeScript; v2.1.88-specific and will drift as Claude Code evolves.
- Skim only — the thirteen design principles and six open directions are the parts
  worth a full read for the harness-architecture cluster.

## Trust signals

- **Credibility:** 4 — VILA Lab / MBZUAI + UCL; preprint tech report, not
  peer-reviewed; substantive companion GitHub artifact (CC BY-NC-SA 4.0) with the
  full analysis released. The quantitative 98.4%/1.6% claim and source-grounded
  comparison raise confidence; closed-source reverse-engineering caps it below 5.

## Follow-up

- This is the load-bearing fourth attestation the harness-architecture cluster needs
  for permission-gate-as-architecture (alongside OpenHarness, Hermes, and the AHI
  security survey wang2026reframing).
- Read the five-layer compaction pipeline as the reference implementation to anchor
  `context-eviction-policy`.
- See the companion repo note `literature/repos/vila-lab-dive-into-claude-code.md`.
