---
kind: paper
title: "ArcticSwarm: Deferring Early Consensus in Long-Horizon Multi-Agent Research"
authors: ["Soyoung Yoon", "Boyi Liu", "Yite Wang", "Ruofan Wu", "Canwen Xu", "Nikki Lijing Kuang", "Seung-won Hwang", "Yuxiong He", "Zhewei Yao"]
institutions: ["Seoul National University", "Snowflake AI Research"]
year: 2026
venue: "arXiv (cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.01870"
code_url: null
citations: null
source: "raw/papers/yoon2026arcticswarm.pdf"
added: "2026-09-08"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["multi-agent", "research-agents", "search", "consensus", "isolation", "deep-research"]
---

# ArcticSwarm: Deferring Early Consensus in Long-Horizon Multi-Agent Research

## TL;DR

Verifier-free research tasks break the usual multi-agent recipe: with no
oracle to select candidates, majority voting stands in — and **peer access
makes parallel agents converge on an early candidate before alternatives are
tested**. ArcticSwarm deliberately **gates isolation** so selected searches
keep their own prior, and adds review at three commitment boundaries. The
ablation is the finding: 82.6% → 78.8% without gated isolation → 74.5%
without review either.

## Claims

- Multi-agent pipelines that work in coding (parallel candidates + reliable
  verifier) **do not generalise to open-ended research**, because the
  verifier is what made them work.
- Self-consistency used as a proxy verifier is actively harmful here:
  parallel agents re-explore the same evidence, and **peer visibility of
  partial findings causes premature convergence**.
- The fix is architectural — **separate evidence gathering from evidence
  integration**, and restrict peer reads during gathering.
- Confidence should gate propagation: only confident candidates cross a
  commitment boundary.

## Methods

- Subagents publish findings to a shared bulletin board, but **gated
  isolation** lets selected search tasks maintain their own prior.
- **Structured review at three commitment boundaries** enforces that only
  confident candidates propagate.
- Evaluated on BrowseComp-Plus (full set) and live-web BrowseComp, with
  component ablations.

## Results

- **BrowseComp-Plus, open-weight Qwen 3.5-27B: 82.6%**, vs **78.8%** without
  gated isolation and **74.5%** with structured review also disabled.
  Aligned MiroFlow baseline: 70.6%.
- **Live-web BrowseComp with GPT-5: 73.6%**, against a reported provider
  system at 54.9% and MiroFlow at 63.4%.
- The two mechanisms contribute roughly 3.8 and 4.3 points respectively —
  neither dominates.

## Critique / open questions

- BrowseComp is retrieval-heavy question answering with verifiable answers.
  Calling it "research" is a stretch: the paper's own premise is that
  research tasks *lack* a verifier, yet the benchmark has one. The
  mechanism is motivated by verifier-free settings and evaluated in a
  verified one.
- "Gated isolation for selected search tasks" — the selection policy is the
  crux and the abstract does not expose how tasks are chosen for isolation.
- No released code.

## Follow-up

- **Relevance:** 4 — **the deliberate inverse of
  [[literature/papers/paglieri2026case]]**, and the pairing is the valuable
  part. Paglieri shows a shared knowledge library and peer messaging
  propagating a reward-hacking exploit across a research swarm; this shows
  *restricting* those same channels improving accuracy by 3.8 points. The
  shared substrate is not merely a contagion risk to be tolerated for its
  benefits — on long-horizon search its benefits are partly illusory,
  because what propagates fastest is early consensus.
  [[concepts/shared-substrate-contagion]] should carry both directions.
- Supplies [[concepts/information-firewall]] with something it lacked: a
  case where the firewall is erected for **epistemic** reasons (preserve
  independent priors) rather than security ones, with an ablation number.
- Premature convergence is the failure [[concepts/pass-at-k]] and
  [[concepts/evolutionary-expansion]] both circle; attacking it as a
  *scheduling and visibility* property rather than a sampling property is a
  distinct move.
- "Only confident candidates cross a commitment boundary" is
  [[concepts/hierarchical-delegation]] with a gate on the upward edge.

## Trust signals

- **Credibility:** 4 — arXiv preprint, not peer reviewed, no citations yet.
  Strong group: Snowflake AI Research (Yuxiong He, Zhewei Yao — both
  established in efficient LLM systems) with Seoul National University.
  Clean component ablation across two benchmarks including a live-web
  setting, and an open-weight result so the headline does not depend on a
  frontier API. No artifact released, which caps it at 4.
