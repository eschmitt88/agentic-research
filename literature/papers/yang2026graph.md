---
kind: paper
title: "Graph-based Agent Memory: Taxonomy, Techniques, and Applications"
authors:
  - Chang Yang
  - Chuang Zhou
  - Yilin Xiao
  - Su Dong
  - Luyao Zhuang
  - Yujing Zhang
  - Zhu Wang
  - Zijin Hong
  - Zheng Yuan
  - Zhishang Xiang
  - Shengyuan Chen
  - Huachi Zhou
  - Qinggang Zhang
  - Ninghao Liu
  - Jinsong Su
  - Xinrun Wang
  - Yi Chang
  - Xiao Huang
institutions: ["Hong Kong Polytechnic University"]
year: 2026
venue: "arXiv:2602.05665 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2602.05665"
code_url: "https://github.com/DEEP-PolyU/Awesome-GraphMemory"
citations:
source: "raw/papers/yang2026graph.pdf"
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
  - knowledge-graph
  - survey
  - taxonomy
  - retrieval
  - memory-lifecycle
---

# Graph-based Agent Memory: Taxonomy, Techniques, and Applications

## TL;DR

(Skim of abstract page.) A survey that organizes LLM-agent memory from
a graph-structured perspective: it offers a taxonomy (short-term vs.
long-term, knowledge vs. experience, non-structural vs. structural),
then walks the memory life cycle — extraction, storage, retrieval,
evolution — through graph techniques, and catalogs open-source
libraries and benchmarks. A companion resource list lives at
`DEEP-PolyU/Awesome-GraphMemory`.

## Claims

- Graph is a uniquely powerful substrate for agent memory because it
  natively models relational dependencies, organizes hierarchical
  information, and supports efficient retrieval.
- Agent memory is best understood through its life cycle —
  extraction → storage → retrieval → evolution — and graph techniques
  map onto each stage.
- A three-axis taxonomy (short-term/long-term, knowledge/experience,
  non-structural/structural) covers the design space, with graph-based
  memory as the structural endpoint.

## Methods

- Survey methodology: organizes prior work into a taxonomy plus a
  life-cycle decomposition; no new system or empirical evaluation.
- Catalogs open-sourced libraries and benchmarks supporting
  self-evolving agent memory, and surveys application scenarios.

## Results

- No experimental results (survey). Contribution is the taxonomy, the
  life-cycle technique map, and the curated resource collection.

## Critique / open questions

- Survey — no independent evaluation; claims about graph superiority
  are aggregated from cited work, not tested here.
- Affiliations not shown on the abstract page; the
  `DEEP-PolyU` GitHub org points to a Hong Kong PolyU group (DEEP lab),
  consistent with several authors (Xiao Huang, Qinggang Zhang).

## Trust signals

- **Credibility:** 3 — Large multi-author survey with a maintained
  open resource repo (`DEEP-PolyU/Awesome-GraphMemory`), which raises
  reproducibility of its claims-as-pointers. arXiv preprint, not
  peer-reviewed; no Comments/venue field. Survey credibility rests on
  coverage rather than a testable result.

## Follow-up

- Strong candidate anchor for a future agent-memory Map of Content:
  the extraction/storage/retrieval/evolution life cycle is a clean
  spine that the existing `agent-native-memory`,
  `selective-memory-retrieval`, and `context-eviction-policy` concepts
  already populate.
- The "non-structural vs. structural" axis is a useful framing for
  why this project's markdown-wikilink graph (a structural, file-native
  memory) sits where it does relative to vector-store designs.
- Mine the cited benchmark list for evaluation yardsticks when
  assessing future memory-mechanism papers (pairs with `hce-evaluation`).
</content>
