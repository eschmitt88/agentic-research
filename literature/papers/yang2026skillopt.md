---
kind: paper
title: "SkillOpt: Executive Strategy for Self-Evolving Agent Skills"
authors:
  - Yifan Yang
  - Ziyang Gong
  - Weiquan Huang
  - Qihao Yang
  - Ziwei Zhou
  - Zisu Huang
  - Yan Li
  - Xuemei Gao
  - Qi Dai
  - Bei Liu
  - Kai Qiu
  - Yuqing Yang
  - Dongdong Chen
  - Xue Yang
  - Chong Luo
institutions:
  - Microsoft
  - Shanghai Jiao Tong University
  - Tongji University
  - Fudan University
year: 2026
venue: arXiv (preprint, cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2605.23904
code_url: https://aka.ms/skillopt
citations: null
source: "raw/papers/yang2026skillopt.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/hce-evaluation]]"
tags: [agent-skills, text-space-optimizer, harness-portable, validation-gated, microsoft]
---

# SkillOpt: Executive Strategy for Self-Evolving Agent Skills

## TL;DR

A systematic, controllable text-space optimizer for agent skills: a separate
optimizer model turns scored rollouts into bounded add/delete/replace edits on a
single skill document, accepting an edit only when it strictly improves a held-out
validation score — with zero added inference-time model calls. Tested across three
execution harnesses (direct chat, Codex, Claude Code). (Skim from abstract + PDF.)

## Claims

- The skill should be trained as the external state of a frozen agent, with the same
  discipline that makes weight-space optimization reproducible.
- First systematic controllable text-space optimizer for skills; bounded
  add/delete/replace edits accepted only on strict held-out validation improvement —
  a search-vs-accept separation (an `hce-evaluation` microcosm at the skill-doc level).
- Skills are harness-portable: best or tied on all 52 (model, benchmark, harness)
  cells across six benchmarks, seven target models, three harnesses; +19.1 to +24.8
  points on GPT-5.5; beats human, one-shot LLM, Trace2Skill, TextGrad, GEPA, EvoSkill.

## Methods

- Optimizer model converts scored rollouts into bounded edits on one skill document.
- Textual learning-rate budget, rejected-edit buffer, epoch-wise slow/meta updates
  for stable "training"; zero inference-time overhead at deployment.

## Results

- Best/tied on all 52 evaluated cells; +19.1 to +24.8 pts on GPT-5.5; transfers
  across model scales and execution environments.

## Critique / open questions

- "Harness-portable" is the strong claim — the empirical backbone of
  `shared-skill-namespace`. Worth checking how much per-harness adaptation the
  pipeline actually needs in practice.
- Skim only; code behind an aka.ms redirect (Microsoft), not directly inspected.

## Trust signals

- **Credibility:** 4 — Microsoft + SJTU/Tongji/Fudan; preprint, not peer-reviewed;
  code released (aka.ms/skillopt). Broad eval grid (52 cells, 7 models, 3 harnesses)
  and strong deltas raise confidence; not peer-reviewed caps it below 5.

## Follow-up

- This is the measured backing for `shared-skill-namespace` (same skill doc gains
  under direct chat, Codex, AND Claude Code) — promote the concept from aspirational
  to attested.
- The validation-gated bounded-edit loop is a clean `hce-evaluation` analogue;
  compare its accept-test to this project's metrics.json/final_metrics.json split.
