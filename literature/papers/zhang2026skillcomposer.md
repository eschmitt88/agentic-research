---
kind: paper
title: "SkillComposer: Learning to Evolve Agent Skills for Specification and Generalization"
authors: ["Qi Zhang", "Zhaopeng Feng", "Xiaonan Shi", "Xiaomeng Hu", "Chu Liu", "Pengjun Xie", "Xiaobin Wang", "Jieping Ye", "Bryan Hooi", "Haobo Wang", "Junbo Zhao"]
institutions: ["Zhejiang University", "Tongyi Lab (Alibaba)", "National University of Singapore"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.06079
code_url: null
citations: null
source: "raw/papers/zhang2026skillcomposer.pdf"
added: "2026-06-22"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/shared-skill-namespace]]"
tags: [skill-library, curation, lifecycle, self-evolving, specification, generalization]
---

# SkillComposer: Learning to Evolve Agent Skills for Specification and Generalization

## TL;DR

SkillComposer decomposes skill construction into three learnable
operations — **create**, **improve**, **merge** — and trains a model
(via delta-pass-rate-guided rejection sampling) to self-evolve its skill
library at inference time *without ground-truth supervision*. It names
**specification** (tailoring a skill to a task pattern) and
**generalization** (abstracting across similar tasks) as two orthogonal
quality dimensions, and assigns *merge* to generalization and *improve*
to specification.

## Claims

- Skill quality has two orthogonal axes — **specification** (too-general
  skills give insufficient guidance) and **generalization** (too-specific
  skills fail to transfer) — and prior one-shot skill-extraction methods
  provide no systematic mechanism to achieve either.
- Decomposing skill construction into **create** (extract reusable
  procedural knowledge from trajectories), **merge** (consolidate
  semantically similar skills → drives generalization), and **improve**
  (refine a skill to capture task-specific patterns → drives
  specification) gives an explicit lever for the trade-off.
- *Merge* and *improve* address **complementary** quality dimensions, and
  skill composition is a **transferable meta-ability** (transfers across
  task types unseen during training).
- A small composer can lift a much larger executor: **SkillComposer-4B
  improves a 27B executor by up to +4.5 on agent tasks and +3.4 on code
  tasks.**

## Methods

- Three deployment modes (Fig. 1): **Offline** — build a generalized
  library from a training set via iterative create + merge, then retrieve
  at inference; **Online** — start empty, create-and-improve on the fly,
  no prior data (fits open-ended / streaming); **Hybrid** — initialize
  from an offline library and specialize via online evolution.
- Self-evolution at inference **without ground-truth supervision**; delta
  pass-rate guided rejection sampling synthesizes SFT data to sharpen the
  three abilities.
- Evaluated on τ²-Bench and AppWorld (agent tasks) and LiveCodeBench v6
  (code), across model scales, in-domain and out-of-distribution.

## Results

- Consistent improvements over baselines across benchmarks, scales, and
  ID/OOD settings.
- SkillComposer-4B → +4.5 (agent) / +3.4 (code) over a 27B executor.
- Analysis: merge and improve move orthogonal quality dimensions; skill
  composition transfers across task types.

## Critique / open questions

- No code/artifacts released (as of v1), so the rejection-sampling recipe
  and the 4B composer aren't independently reproducible yet.
- Evaluated on agent/code benchmarks (τ²-Bench, AppWorld, LiveCodeBench),
  not on research-agent tasks — the create/improve/merge arc is plausibly
  transferable but unattested on the ML-research front this project tracks.
- The create/improve/**merge** triple differs from this project's
  insert/update/**delete** lifecycle: *merge* is a consolidation operation
  with no clean analogue to deletion (it fuses rather than prunes). Open
  whether a mature library needs both merge *and* delete, or whether
  merge subsumes delete (the merged-away skill is effectively deleted).

## Trust signals

- **Credibility:** 3 — reputable groups (Zhejiang University, Alibaba
  Tongyi Lab, NUS; Bryan Hooi / Jieping Ye among authors) and a clean
  multi-benchmark evaluation, but an arXiv preprint with no released code
  and no citations yet. Solid-but-unverified.

## Follow-up

- **Relevance:** 4 — material new evidence for
  [[concepts/skill-library-lifecycle]]: an explicit, *learnable*
  three-operation construction loop (create/improve/merge) with the
  specification-vs-generalization framing the existing concept lacked,
  plus a clean [[concepts/hybrid-model-backends]] attestation (a 4B
  composer improving a 27B executor — small strategic model lifting a
  larger executor, echoing SkillOS's curator/executor inversion). Not a 5
  only because it doesn't seed a new importable concept on its own and is
  unattested on research-agent tasks.
