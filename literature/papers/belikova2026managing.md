---
kind: paper
title: "Managing Procedural Memory in LLM Agents: Control, Adaptation, and Evaluation"
authors:
  - Julia Belikova
  - Rauf Parchiev
  - Evgeny Egorov
  - Grigorii Davydenko
  - Gleb Gusev
  - Andrey Savchenko
  - Maksim Makarenko
institutions: []   # no affiliation block on the title page; authors (Savchenko, Gusev, Davydenko) are an industrial AI-lab cluster, but not stated in-PDF — left empty rather than guessed
year: 2026
venue: "arXiv:2606.23127 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.23127"
code_url: "https://huggingface.co/datasets/DavydenkoGr/AFTER"   # AFTER benchmark released as a HF dataset
citations: null
source: "raw/papers/belikova2026managing.pdf"
added: "2026-06-29"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/pass-at-k]]"
tags:
  - procedural-memory
  - skill-library
  - skill-transfer
  - benchmark
  - generalization
  - agent-skills-format
---

# Managing Procedural Memory in LLM Agents: Control, Adaptation, and Evaluation

## TL;DR

Introduces **AFTER**, a benchmark of 382 realistic enterprise tasks
across six professional roles and 22 procedural skills, designed to
measure how procedural-memory skills *transfer* — across tasks, roles,
and model backbones — rather than just whether they help in their
source setting. Headline finding: procedural memory gives consistent
gains, but skills evolved from *narrow* experience over-specialize
(higher specificity, degraded generality), while skills evolved from
*diverse* multi-model traces reach high-specificity **and**
high-generality.

## Claims

- Existing systems and benchmarks **conflate local improvement with
  true transfer**; the field lacks a controlled setting for "when does
  experience produce reusable procedural structure, and when does it
  merely overfit to where it was observed?"
- Procedural memory should be studied along two axes — **specialization**
  vs **generalization** — and evaluated under task, role, and model
  shift, not just on the source task.
- Diverse-experience skills transfer; narrow-experience skills over-fit
  their source context. Cross-role transfer reveals a complementary
  failure: skills can over-specialize to a role and lose effectiveness
  when moved to another role.

## Methods

- **AFTER benchmark**: 382 tasks (318 single-skill, 64 multi-skill),
  six roles (Data Engineer, Data Scientist, GenAI Engineer,
  Infrastructure Engineer, Project Manager, Software Engineer), 22
  skills over five capability areas (document, data-ops, ML/AI, infra,
  SW-eng). 56 tasks adapted from 13 prior benchmarks (SWE-bench
  Verified/Pro, MLE-bench, SkillsBench, Terminal-Bench, …) rewritten as
  self-contained workplace requests with pytest verification; 326 newly
  authored.
- Each skill is a **self-contained `SKILL.md` artifact in the Agent
  Skills format** — "a versionable, retrievable unit of procedural
  memory at uniform granularity." Skill annotations are *fixed at task
  definition*, separating skill quality from retrieval (retrieval can be
  studied separately on the same tasks).
- Controlled **transfer splits**: local improvement (in-context gain),
  cross-task, cross-role, cross-model held-out splits. Skills are
  evolved with a Hermes memory-update operator under an EVOLUTION
  protocol and scored in both source and shifted contexts.

## Results

- A single refinement round improves aggregate performance by
  **3.7–6.7 points**.
- Static benchmark: procedural skills improve full-pass accuracy by
  **+2.8 pts** on average; one refinement round adds a further
  **+5.2 pts** across model scales.
- Cross-model transfer: skills evolved from diverse multi-model traces
  reach **73.1% test accuracy**, exceeding the best single-model trace
  source by **+13.7 pts**.
- Cross-role: skills can over-specialize to role-specific workflows and
  lose effectiveness under transfer (the specialization/generalization
  trade-off made measurable).

## Critique / open questions

- No author affiliations on the title page; credibility leans on the
  released artifact (AFTER on HuggingFace) rather than venue/institution.
- "Uniform granularity" is a deliberate design choice that brackets the
  multi-granularity question — interesting tension with
  `multi-granularity-memory`, where routing across grains is the point.
  AFTER fixes granularity to isolate the transfer question; whether the
  over-specialization finding survives a multi-grain store is untested.
- Retrieval is factored out by fixing skill annotations at task
  definition — clean for studying skill *quality*, but real deployments
  pay retrieval cost the benchmark sidesteps.

## Trust signals

- **Credibility:** 3 — arXiv preprint (not peer-reviewed), but a
  substantial 17-page benchmark contribution with a **released dataset
  artifact** (AFTER on HuggingFace) and tasks adapted from 13
  recognized benchmarks; recognizable author cluster. No citations yet.
  Reproducibility (released benchmark) weighed above the absent
  affiliation block.

## Follow-up

- **Relevance:** 4 — strengthens `skill-library-lifecycle` with material
  new evidence: a transfer-oriented benchmark plus a measured
  specialization-vs-generalization trade-off that sharpens the
  write/refine/prune-policy question (when is an evolved skill worth
  keeping or sharing vs. when has it over-fit its source). Directly
  attests `shared-skill-namespace` (skills as portable `SKILL.md` Agent
  Skills artifacts) and `hce-evaluation` (controlled held-out transfer
  splits separating local improvement from true transfer = the
  source-signal-overfitting failure mode).
- Pairs naturally with the prior skill-library batch (SkillOps,
  SkillOpt, SkillComposer, Library-Drift-adjacent work). A future
  /derive-experiment is out of scope here (meta project), but AFTER is a
  candidate evaluation harness for any downstream skill-curation work.
