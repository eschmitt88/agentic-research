---
kind: paper
title: "Bayesian-Agent: Posterior-Guided Skill Evolution for LLM Agent Harnesses"
authors: ["Xiaojun Wu", "Cehao Yang", "Honghao Liu", "Xueyuan Lin", "Wenjie Zhang", "Zhichao Shi", "Xuhui Jiang", "Chengjin Xu", "Jia Li", "Jian Guo"]
institutions: ["IDEA Research", "Hong Kong University of Science and Technology (Guangzhou)", "DataArcTech Ltd."]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.08348
code_url: https://github.com/DataArcTech/Bayesian-Agent
citations: null
source: "raw/papers/wu2026bayesian.pdf"
added: "2026-06-22"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
tags: [skill-library, lifecycle, harness, bayesian, posterior, evidence, sop]
---

# Bayesian-Agent: Posterior-Guided Skill Evolution for LLM Agent Harnesses

## TL;DR

Bayesian-Agent treats a frozen model's reusable skills and SOPs as
**Bayesian hypotheses** about whether the model will succeed under a
given prompt/context/harness, records verified trajectory evidence, and
maintains a feature-conditioned categorical posterior per skill. The
posterior drives inspectable rewrite actions — **patch, split, compress,
retire, explore** — reframing skill evolution as posterior-guided harness
optimization rather than uncalibrated prompt accumulation.

## Claims

- Reusable skills/SOPs should be **evidence-bearing hypotheses**, not
  artifacts revised by ad-hoc reflection: "under a frozen model and a
  given inference environment, what should we believe about this skill
  after combining prior assumptions with verified evidence?"
- Sparse agent trajectories are *not* i.i.d. — the same skill can be
  reliable in one benchmark context, harmful in another, ambiguous after
  a few runs; a narrow Bayesian question handles this better than asking
  an LLM to judge a skill in isolation.
- A unified Bayesian view of prompt, context, and harness engineering,
  instantiated with an efficient categorical evidence model, a
  posterior-guided rewrite policy, a native backend, and an adapter
  boundary for external harnesses.

## Methods

- Per-skill **feature-conditioned categorical posterior** over success /
  failure modes, updated from verified trajectory evidence.
- Posterior state maps to five inspectable rewrite actions: **patch**
  (failure-mode fix), **split**, **compress**, **retire** (prune),
  **explore**. Model-facing prompts receive *executable guardrails* and
  failure-mode patches; raw posterior numbers stay available only for
  audit/ranking/debug.
- Two modes: **full** (registry evolves online from scratch) and
  **incremental** (an existing run supplies evidence; only failed tasks
  are repaired).
- Native backend plus adapter boundary; external backends (GenericAgent,
  mini-swe-agent, Claude Code) sit behind the same trajectory-evidence
  boundary. Studied with deepseek-v4-flash and deepseek-v4-pro.

## Results

- Incremental repair (deepseek-v4-flash): **SOP-Bench 80→95%**, **Lifelong
  AgentBench 90→100%**, **RealFin-Bench 45→65%**.
- Reports positive, negative, saturated, and case-study settings — the
  authors frame skill evolution as posterior-guided optimization rather
  than a guaranteed monotone gain.

## Critique / open questions

- The posterior is a *frequency/Bayesian* signal over verified
  trajectories — a principled middle ground between crisp deterministic
  oracles and noisy LLM-as-judge. How it degrades when trajectory
  evidence is very sparse (the cold-start a skill faces on day one) isn't
  fully characterized.
- Benchmarks are agent/SOP/finance (SOP-Bench, Lifelong AgentBench,
  RealFin-Bench), not ML-research tasks — the rewrite-action policy is
  plausibly transferable to a research-note/skill library but unattested
  there.
- The five rewrite actions (patch/split/compress/retire/explore) are
  richer than this project's insert/update/delete; whether all five are
  load-bearing or some collapse together in practice is not ablated.

## Trust signals

- **Credibility:** 3 — reputable groups (IDEA Research, HKUST-GZ) **with
  released code** (github.com/DataArcTech/Bayesian-Agent), a clean
  multi-benchmark evaluation reporting negative/saturated cases honestly,
  but an arXiv preprint, not yet peer-reviewed or cited. Code release
  pushes this above a no-artifact preprint.

## Follow-up

- **Relevance:** 4 — strengthens [[concepts/skill-library-lifecycle]]
  with a third, *evidence-driven* operation set (patch/split/compress/
  retire/explore selected by a calibrated posterior, not heuristic
  reflection) and gives [[concepts/programmable-evaluator-oracle]] a
  concrete instance of the "learned/probabilistic evaluator" its open
  questions ask about. Has runnable code. A 5 would require a
  research-agent attestation or a new importable concept.
