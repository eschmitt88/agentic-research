---
kind: paper
title: "GAM: Hierarchical Graph-based Agentic Memory for LLM Agents"
authors:
  - Zhaofen Wu
  - Hanrong Zhang
  - Fulin Lin
  - Wujiang Xu
  - Xinran Xu
  - Yankai Chen
  - Henry Peng Zou
  - Shaowen Chen
  - Weizhi Zhang
  - Xue Liu
  - Philip S. Yu
  - Hongwei Wang
institutions: []
year: 2026
venue: "arXiv:2604.12285 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.12285"
code_url:
citations:
source: "raw/papers/wu2026gam.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags:
  - agent-memory
  - graph-memory
  - hierarchical-memory
  - memory-consolidation
  - retrieval
  - long-horizon
---

# GAM: Hierarchical Graph-based Agentic Memory for LLM Agents

## TL;DR

(Skim of abstract page.) GAM is a hierarchical graph-based memory
framework that explicitly **decouples memory encoding from
consolidation** to resolve the tension between fast context
perception and stable long-term retention. Ongoing dialogue is
isolated in an *event progression graph* and integrated into a *topic
associative network* only upon a detected semantic shift, minimizing
interference. A graph-guided multi-factor retrieval strategy sharpens
context precision. Beats SOTA baselines on LoCoMo and LongDialQA on
both reasoning accuracy and efficiency.

## Claims

- Unified stream-based memory systems update context easily but are
  vulnerable to interference from transient noise; discrete structured
  memory retains knowledge robustly but adapts poorly to evolving
  narratives. GAM's hierarchy is the proposed resolution.
- Decoupling encoding (event progression graph) from consolidation
  (topic associative network), and consolidating only on semantic
  shift, minimizes interference while preserving long-term consistency.
- A graph-guided, multi-factor retrieval strategy improves context
  precision over flat retrieval.

## Methods

- **Two-tier graph**: (1) an *event progression graph* that isolates
  ongoing dialogue (the encoding/working tier); (2) a *topic
  associative network* into which events are consolidated only upon a
  detected semantic shift (the stable long-term tier).
- **Semantic-shift-gated consolidation**: the trigger for moving
  content from the event graph into the topic network is a semantic
  shift, not a fixed schedule — this is the eviction/consolidation
  policy.
- **Graph-guided multi-factor retrieval** for context selection at
  read time.

## Results

- On **LoCoMo** and **LongDialQA**, GAM consistently outperforms
  state-of-the-art baselines in both reasoning accuracy and
  efficiency (per abstract; specific numbers not extracted from the
  skim).

## Critique / open questions

- No code URL located on the abstract page; reproducibility unverified
  at skim time.
- "Semantic shift" as the consolidation trigger is the load-bearing
  design choice but the abstract does not specify how the shift is
  detected — worth a deep read before lifting the mechanism.
- Evaluated on conversational/dialogue benchmarks (LoCoMo,
  LongDialQA), not research-graph synthesis; generalization to this
  project's use case is plausible but extrapolated (same caveat as
  ByteRover).

## Trust signals

- **Credibility:** 3 — Strong author roster (Philip S. Yu, Xue Liu are
  well-known senior ML researchers), suggesting a UIUC/McGill-adjacent
  group; "18 pages, 6 figures" Comments imply a full paper. arXiv
  preprint, not peer-reviewed; no released code found at skim. Senior
  authorship is a prior, not a verdict.

## Follow-up

- Direct mechanistic counterpart to ByteRover and to the
  `context-eviction-policy` concept: GAM's encoding/consolidation split
  with a semantic-shift trigger is a concrete instantiation of
  "what stays hot vs. what gets consolidated to a colder tier."
  Worth a side-by-side with ByteRover's AKL when a downstream project
  wants a consolidation policy.
- Pairs with `yang2026graph` (the survey) as a worked example of its
  "evolution" life-cycle stage — consolidation-on-semantic-shift.
- A deep read should extract the semantic-shift detector and the
  multi-factor retrieval scoring; both could inform `/digest` and
  `/lint` consolidation heuristics.
</content>
