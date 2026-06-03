---
kind: paper
title: "Rethinking How to Remember: Beyond Atomic Facts in Lifelong LLM Agent Memory"
authors:
  - Jingwei Sun
  - Jianing Zhu
  - Jiangchao Yao
  - Tongliang Liu
  - Bo Han
institutions: ["Hong Kong Baptist University", "UT Austin", "Shanghai Jiao Tong University", "University of Sydney"]
year: 2026
venue: "arXiv:2605.19952 [cs.CL]"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.19952"
code_url: "https://TMLR-TriMem.github.io"
citations: null
source: "raw/papers/sun2026rethinking.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
tags:
  - agent-memory
  - multi-granularity
  - atomic-facts
  - textgrad
  - lifelong-learning
---

# Rethinking How to Remember: Beyond Atomic Facts (TriMem)

## TL;DR

Argues the dominant "extract atomic facts" memory paradigm discards
detail and can't reason over scattered facts; proposes TriMem, which keeps
three coexisting granularities (raw segments, atomic facts, synthesized
profiles) and uses TextGrad prompt optimization to evolve extraction
without parameter updates.

## Claims

- Fact-centric memory discards fine-grained detail and fails at deep
  reasoning over scattered isolated facts; static prompts can't keep
  consistent extraction granularity across dialogue styles.
- Maintaining three granularities simultaneously — raw segments (fidelity),
  atomic facts (retrieval), synthesized profiles (deep reasoning) —
  outperforms single-representation memory.
- TextGrad-based prompt optimization achieves lifelong evolution without
  any parameter updating.

## Methods

- Three-granularity store with source-anchored raw segments; profiles
  aggregate dispersed facts into holistic semantic understanding.
- TextGrad iteratively refines extraction/profiling prompts via
  response-quality feedback.
- Evaluated on LoCoMo and PerLTQA across multiple LLM backbones.

## Results

- Consistently outperforms strong memory baselines on both benchmarks.

## Critique / open questions

- Maintaining three coexisting representations multiplies storage and
  extraction cost; the abstract doesn't quantify the overhead.
- Strong second attestation (with H-Mem) that the field is converging on
  multi-granularity memory — the raw/atomic/profile stack is itself a
  candidate concept seed.

## Trust signals

- **Credibility:** 3 — preprint, multi-institution (HKBU TMLR group + UT
  Austin + SJTU + Sydney), code page released, multi-backbone evaluation.

## Follow-up

- Third attestation would seed `multi-granularity-memory`.
- The raw/atomic/profile split maps onto the
  "what stays in working set vs. what gets compressed" question behind
  `context-eviction-policy`.
