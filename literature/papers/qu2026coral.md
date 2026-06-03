---
kind: paper
title: "CORAL: Towards Autonomous Multi-Agent Evolution for Open-Ended Discovery"
authors:
  - Ao Qu
  - Han Zheng
  - Zijian Zhou
  - Yihao Yan
  - Yihong Tang
  - Shao Yong Ong
  - Fenglu Hong
  - Kaichen Zhou
  - Chonghe Jiang
  - Minwei Kong
  - Jiacheng Zhu
  - Xuan Jiang
  - Sirui Li
  - Cathy Wu
  - Bryan Kian Hsiang Low
  - Jinhua Zhao
  - Paul Pu Liang
institutions: []
year: 2026
venue: "arXiv:2604.01658 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.01658"
code_url: "https://github.com/Human-Agent-Society/CORAL"
citations:
source: "raw/papers/qu2026coral.pdf"
added: "2026-06-03"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags:
  - evolutionary-search
  - multi-agent
  - open-ended-discovery
  - autonomous-agents
  - shared-memory
  - evaluator-separation
  - asynchronous
---

# CORAL: Towards Autonomous Multi-Agent Evolution for Open-Ended Discovery

## TL;DR

(Skim of abstract page.) CORAL is presented as the first framework for
*autonomous multi-agent evolution* on open-ended problems. It replaces
the fixed heuristics and hard-coded exploration rules of prior
LLM-evolution methods (FunSearch / AlphaEvolve lineage) with
long-running agents that explore, reflect, and collaborate via shared
persistent memory, asynchronous multi-agent execution, and
heartbeat-based interventions. On math, algorithmic, and systems
optimization tasks it sets SOTA on 10 tasks with 3–10x higher
improvement rates and far fewer evaluations than fixed evolutionary
baselines. On Anthropic's kernel engineering task, four co-evolving
agents improve the best-known score from 1363 to 1103 cycles.

## Claims

- Existing LLM-evolution methods are bottlenecked by fixed heuristics
  and hard-coded exploration rules that cap agent autonomy; replacing
  rigid control with long-running, reflecting, collaborating agents
  improves open-ended discovery.
- Greater agent autonomy *and* multi-agent co-evolution each
  substantially improve discovery (mechanistic analyses attribute
  gains to knowledge reuse and multi-agent exploration/communication).
- Achieves 3–10x higher improvement rates with **far fewer
  evaluations** than fixed evolutionary search — efficiency, not just
  ceiling, improves.

## Methods

- **Long-running agents** that explore, reflect, and collaborate
  through **shared persistent memory** (the knowledge-accumulation
  substrate) — a multi-agent generalization of the evolution loop.
- **Asynchronous multi-agent execution** plus **heartbeat-based
  interventions** for liveness/control of long-running agents.
- **Practical safeguards**: isolated workspaces, **evaluator
  separation**, resource management, and agent session/health
  management.
- Evaluated across mathematical, algorithmic, and systems optimization
  tasks, including Anthropic's kernel engineering task.

## Results

- New SOTA on **10 tasks**; **3–10x** higher improvement rates with
  far fewer evaluations than fixed evolutionary-search baselines.
- Anthropic kernel engineering: four co-evolving agents improve the
  best-known score from **1363 → 1103 cycles** (lower is better).
- Mechanistic analyses attribute gains to knowledge reuse and
  multi-agent exploration/communication.

## Critique / open questions

- "First autonomous multi-agent evolution" is a strong primacy claim;
  worth checking against concurrent multi-agent FunSearch/AlphaEvolve
  variants before repeating it.
- Heartbeat-based intervention and shared-memory coordination are the
  novel mechanisms but the abstract is thin on how interventions are
  triggered — deep read needed before lifting the design.
- "Far fewer evaluations" is the most interesting efficiency claim for
  this project's budget discipline; the eval-count comparison
  methodology should be verified.

## Trust signals

- **Credibility:** 4 — Large MIT/NUS-adjacent author roster including
  senior researchers (Cathy Wu, Bryan Kian Hsiang Low, Jinhua Zhao,
  Paul Pu Liang); **open code released**
  (`Human-Agent-Society/CORAL`); concrete, externally-anchored result
  on Anthropic's kernel task (1363→1103). arXiv preprint (revised),
  not yet peer-reviewed, which is the only thing holding it below 5.

## Follow-up

- The single most on-mission item in this digest for the evolutionary
  cluster: CORAL is the multi-agent, autonomy-maximizing successor to
  FunSearch/AlphaEvolve already in the graph
  (`novikov2025alphaevolve`, `romeraparedes2024funsearch`). Directly
  extends `evolutionary-expansion` from single-population search to
  asynchronous multi-agent co-evolution.
- **Evaluator separation** is a first-class safeguard here — strong
  corroboration for `programmable-evaluator-oracle` (the evaluator as
  environment, kept distinct from the search agents) and adjacent to
  `hce-evaluation`'s search/scoring separation.
- "Far fewer evaluations" pairs with `budget-as-ceiling`: a worked
  example that better coordination can cut evaluation spend, not just
  raise the ceiling. Candidate for a derive-experiment in a downstream
  evolutionary-search project.
- Code is open — a downstream project could read the shared-memory and
  heartbeat-intervention implementation directly.
</content>
