---
kind: paper
title: "A Comprehensive Survey on Agent Skills: Taxonomy, Techniques, and Applications"
authors:
  - Yingli Zhou
  - Shu Wang
  - Yaodong Su
  - Wenchuan Du
  - Yixiang Fang
  - Xuemin Lin
institutions:
  - "The Chinese University of Hong Kong, Shenzhen"
year: 2026
venue: arXiv (survey, cs.IR)
peer_reviewed: false
url: https://arxiv.org/abs/2605.07358
code_url: https://github.com/JayLZhou/Awesome-Agent-Skills
citations: null
source: "raw/papers/zhou2026comprehensive.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [survey, agent-skills, lifecycle, taxonomy, cuhk]
---

# A Comprehensive Survey on Agent Skills: Taxonomy, Techniques, and Applications

## TL;DR

A survey that reifies the agent-skills sub-field's vocabulary: defines skills as
"reusable procedural artifacts that coordinate tools, memory, and runtime context
under task-specific constraints" and organizes the literature into a four-stage
lifecycle — representation, acquisition, retrieval, evolution. (Skim from
abstract + PDF intro.)

## Claims

- Agents and skills are complementary: agents do high-level reasoning/planning;
  skills are the operational layer enabling reliable, reusable, composable execution.
- The four-stage lifecycle (representation/acquisition/retrieval/evolution) is the
  canonical organizing frame — directly relevant to `skill-library-lifecycle`, which
  currently uses fuzzier terms.
- Names open challenges: quality control, interoperability, safe updating, long-term
  capability management.

## Methods

- Literature survey; companion GitHub resource collection
  (github.com/JayLZhou/Awesome-Agent-Skills) as a prior-art map.

## Results

- A taxonomy + method/resource/application review across the four lifecycle stages
  (no empirical result of its own — it's a survey).

## Critique / open questions

- Surveys reify vocabulary but do not adjudicate competing claims; useful as a frame,
  not as evidence for any single mechanism.
- Reading it may motivate renaming/refactoring `skill-library-lifecycle` to match the
  canonical four-stage terminology — a decision worth recording if taken.

## Trust signals

- **Credibility:** 4 — CUHK-Shenzhen (incl. IEEE Member/Fellow co-authors); preprint
  survey, not peer-reviewed; companion GitHub resource collection released. Breadth
  and maintained resource map raise confidence as a reference frame.

## Follow-up

- Map this project's `skill-library-lifecycle` onto the four canonical stages;
  decide whether to adopt the terminology (record as a decision if so).
- The Awesome-Agent-Skills index is a candidate cross-check for the concept index's
  coverage of the skills front.
