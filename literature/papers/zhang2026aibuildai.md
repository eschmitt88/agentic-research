---
kind: paper
title: "AIBuildAI: An AI Agent for Automatically Building AI Models"
authors:
  - Ruiyi Zhang
  - Peijia Qin
  - Qi Cao
  - Li Zhang
  - Pengtao Xie
year: 2026
venue: "arXiv:2604.14455 [cs.AI]"
url: "https://arxiv.org/abs/2604.14455"
source: "raw/papers/zhang2026aibuildai.pdf"
added: "2026-04-24"
relevance: 5
status: skimmed
related_experiments:
  - per-role-model-downgrade
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
tags:
  - mle-bench
  - hierarchical-agent
  - manager-designer-coder-tuner
  - agent-architecture
---

# AIBuildAI: An AI Agent for Automatically Building AI Models

## TL;DR

UCSD's AIBuildAI reaches rank-1 on MLE-bench with a 63.1% medal rate
via hierarchical delegation: a manager agent coordinates three
specialist sub-agents — designer (modeling strategy), coder
(implementation + debugging), tuner (training + performance). Each
sub-agent is an LLM agent with its own multi-step reasoning and tools.

## Claims

- End-to-end ML engineering is a coordination problem, not a single
  monolithic LLM task. Splitting roles lets each specialist use its
  appropriate prompt, tools, and (implicitly) its appropriate model.
- The four-role split — manager, designer, coder, tuner — mirrors
  how human ML teams actually work: strategy, architecture, code,
  optimization.
- Hierarchical delegation beats flat AutoML (which restricts itself
  to narrow slices like hyperparameter search) and beats monolithic
  LLM agents (which try to do everything in one context).

## Methods

- Manager agent: receives task description + training data, assigns
  sub-tasks, arbitrates between specialists.
- Designer: proposes modeling strategies (architectures, feature
  plans).
- Coder: implements the designer's spec, debugs failures.
- Tuner: optimizes hyperparameters, schedules training runs, reports
  metrics.
- Each sub-agent is an LLM with multi-step reasoning and tool access;
  handoffs happen via structured messages through the manager.

## Results

- 63.1% medal rate on MLE-bench — rank-1 at time of publication,
  outperforming prior flat-agent and AutoML approaches.
- Said to "match the capability of highly experienced AI engineers"
  on the benchmark.

## Critique / open questions

- Manager agent becomes the context bottleneck; unclear how it
  scales to longer / more complex tasks.
- Role boundaries (designer vs. coder, coder vs. tuner) are sharp in
  principle but fuzzy in practice — e.g. architecture choice often
  depends on tuner feedback. Paper's handoff protocol is
  underspecified in the abstract.
- No mention of a HCE-style separation; if the tuner sees test-split
  metrics during its loop, the 63.1% may be partly driven by
  validation-signal overfitting.

## Follow-up

- Deep-read for the manager's arbitration logic — this is where
  hierarchical delegation either works or degrades to "four prompts
  stapled together."
- Cross-check whether the role split supports per-role model choice
  (ideator-vs-implementer backend split is a natural fit here).
- Compare to AIRA_2's ReAct-with-dynamic-scoping: delegation vs.
  scope-adjustment are two different answers to the same
  "fixed single-turn operator" bottleneck.
