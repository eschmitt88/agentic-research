---
kind: paper
title: "Harnessing LLM Agents with Skill Programs"
authors:
  - Hongjun Liu
  - Yifei Ming
  - Shafiq Joty
  - Chen Zhao
institutions:
  - New York University
  - Salesforce AI Research
year: 2026
venue: arXiv (preprint, cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2605.17734
code_url: null
citations: null
source: "raw/papers/liu2026harnessing.pdf"
added: "2026-06-03"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [agent-skills, executable-guardrails, sandbox, self-improvement, salesforce, nyu]
---

# Harnessing LLM Agents with Skill Programs

## TL;DR

HASP upgrades skills from advisory text into executable Program Functions (PFs) that
act as guardrails — activating on failure-prone states to modify the next action or
inject corrective context — applicable at inference, in post-training, or for
self-improvement. (Skim from abstract + PDF intro.)

## Claims

- Textual skills remain largely advisory and lack mechanisms for *when/how* to
  intervene; PFs make skills executable guardrails with runtime enforcement.
- HASP is modular across three modes: inference-time intervention, post-training
  supervision, and self-improvement via evolving teacher-reviewed PFs.
- Inference-time PFs alone improve web-search reasoning by 25% over a multi-loop
  ReAct agent; post-training + controlled evolution gives 30.4% over Search-R1.

## Methods

- Convert reusable skills from past experience into executable PFs.
- Candidate PFs are wrapped in a sandbox preamble with sequential checks
  (syntax, interface, mock execution, return type) — a programmable-evaluator-oracle
  mini-instance gating skill admission.
- Mechanism analysis on how PFs trigger, intervene, and get internalized; notes a
  requirement for stable skill-library evolution.

## Results

- +25% web-search reasoning vs. ReAct; +30.4% vs. Search-R1 with
  post-training + controlled evolution. Gains also reported on math and coding.

## Critique / open questions

- "Executable guardrail" is a stronger framing than skills-as-prompts; whether the
  gains come from the guardrail mechanism or from the validation gate is worth
  disentangling.
- Skim only; no code link surfaced, which limits reproduction of the sandbox checks.

## Trust signals

- **Credibility:** 4 — NYU + Salesforce AI Research; preprint, not peer-reviewed; no
  code link surfaced. Concrete, sizeable benchmark deltas and an explicit ablation-
  style mechanism analysis raise confidence.

## Follow-up

- The four-check sandbox preamble (syntax/interface/mock/return) is a clean
  `programmable-evaluator-oracle` instance — compare to this project's acceptance
  gates.
- Skills-as-executable-guardrails may refactor `skill-library-lifecycle` toward
  runtime enforcement rather than lookup; pair with zhou2026comprehensive's lifecycle.
