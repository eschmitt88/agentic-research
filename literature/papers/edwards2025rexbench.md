---
kind: paper
title: "RExBench: Can coding agents autonomously implement AI research extensions?"
authors:
  - Nicholas Edwards
  - Yukyung Lee
  - Yujun Audrey Mao
  - Yulu Qin
  - Sebastian Schuster
  - Najoung Kim
institutions: ["University of Vienna", "Boston University"]
year: 2025
venue: "ACL 2026 (arXiv:2506.22598 [cs.CL])"
peer_reviewed: true
url: "https://arxiv.org/abs/2506.22598"
code_url: null
citations: null
source: "raw/papers/edwards2025rexbench.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags:
  - benchmark
  - research-extension
  - coding-agents
  - data-contamination
  - automatic-evaluation
---

# RExBench: Can coding agents autonomously implement AI research extensions?

## TL;DR

A benchmark of 12 realistic research-extension tasks — each an extension to
an existing paper + codebase with expert-written instructions and automatic
execution-based scoring — testing whether coding agents can extend research,
not just replicate it. All evaluated agents fail the majority of tasks.

## Claims

- Research extension (implementing a novel hypothesis on top of an existing
  paper/codebase) is a critical, distinct capability worth benchmarking.
- The benchmark is robust to data contamination and supports automatic
  execution-based evaluation against success criteria.
- Current agents are far from handling realistic extensions without
  substantial human guidance.

## Methods

- 12 tasks, each an extension to an existing research paper + codebase with
  domain-expert instructions.
- Automatic evaluation infrastructure executes agent outputs against
  success criteria.
- Evaluates 12 LLM agents across two frameworks (aider, OpenHands).

## Results

- All agents fail the majority of extensions; best agent ~33% success.
- With additional human-written hints, best performance still below 44%.

## Critique / open questions

- Complements MLE-bench / PaperBench / AIRA on the "extend, not just
  replicate" axis — a sharper, harder capability test.
- The execution-based oracle and contamination-robust design are directly
  relevant to `hce-evaluation` and `programmable-evaluator-oracle`.

## Trust signals

- **Credibility:** 4 — peer-reviewed (ACL 2026), University of Vienna +
  Boston University, contamination-robust automatic evaluation; v3 on
  arXiv.

## Follow-up

- Position in the research-agent benchmark family alongside MLE-bench and
  PaperBench; note the extension vs replication distinction.
- Inspect the automatic-evaluation harness as a `programmable-evaluator-oracle`
  exemplar.
