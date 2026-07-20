---
kind: paper
title: "Agents-K1: Towards Agent-native Knowledge Orchestration"
authors: ["Zongsheng Cao", "Bihao Zhan", "Jinxin Shi", "Jiong Wang", "Fangchen Yu", "Zhijie Zhong", "et al. (29 authors)", "Bo Zhang", "Lei Bai"]
institutions: ["Shanghai Artificial Intelligence Laboratory", "East China Normal University", "Fudan University"]
year: 2026
venue: "arXiv preprint (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.13669"
code_url: "https://github.com/InternScience/GraphAnything"
citations: null
source: "raw/papers/cao2026agentsk1.pdf"
added: "2026-07-20"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/citation-anchoring]]"
tags: [knowledge-graph, knowledge-orchestration, multimodal, information-extraction, grpo, graph-rag, scientific-agents, mcp, provenance]
---

# Agents-K1: Towards Agent-native Knowledge Orchestration

## TL;DR

Shanghai AI Lab's end-to-end pipeline for turning raw papers into
agent-native multimodal knowledge graphs: a MinerU-based parser with a
five-module typed schema, a 4B GRPO-trained extraction backbone that
roughly closes the gap to a 32B base, and a tri-source "GraphAnything"
CLI (web search + graph retrieval + cross-document traversal, exposed
as CLI/MCP/Python). Run over 2.46M papers it yields **Scholar-KG** (1M
subset released), lifting GPT-5.2 from 25.2% to 39.4% on
FrontierScience-Research and topping nine graph-RAG baselines on
multi-hop QA.

## Claims

- Research agents have advanced through *agent* orchestration while
  *knowledge* orchestration stagnated: existing scholarly infrastructure
  reduces papers to abstracts, surface mentions, and flat `cites` edges,
  losing claims, evidence, mechanisms, and method lineages — so agents
  re-extract structure from raw PDFs at every query and cannot trace
  answers back to exact evidence.
- Agent-native knowledge orchestration has three requirements:
  full-paper multimodal coverage (figures/tables/equations as
  first-class evidence, not captions), typed knowledge (entities,
  claims, mechanisms, citation *intent*, inter-entity relations), and
  auditable retrieval (every answer traceable to stable graph
  identifiers + exact evidence spans).
- The framework's core separation: **offline construction of reliable
  knowledge representations vs online use of that knowledge** — do the
  organization work once, then amortize it across every downstream
  query.
- A middle "semantic anchor" layer (modality-agnostic summary nodes per
  content unit) avoids brittle direct cross-modal entity alignment
  while preserving fine-grained provenance; theoretical propositions
  (identifier-preserving joins, cross-view reachability) argue why one
  connected graph beats searching separate text fragments.
- Targeted RL recovers most of raw parameter scaling for structured
  extraction: the 4B GRPO backbone beats the 8B base on 8/10
  benchmarks and matches-or-exceeds Qwen3-32B on NER, lagging it only
  0.99 F1 overall.

## Methods

- **KG layer:** MinerU modality-aware decomposition of PDFs into
  {text, figure, table, equation} content units; three-layer
  heterogeneous graph (fine-grained entities → semantic anchors →
  document structure) with `grounded_in`/`belongs_to` cross-layer
  edges. Five-category schema: A meta/factual (each field stored with
  provenance ⟨doc, section/page, span⟩ and a calibrated confidence
  score), B textually mentioned entities, C implicit/abstracted
  knowledge (contributions, findings, limitations), D citation
  relations with intent, E durable inter-entity knowledge triples.
- **LLM layer:** Qwen3-4B-Instruct trained with multi-reward GRPO
  (format compliance + JSON validity + task-conditioned F1) on IEPile;
  roughly one hour on a single 8-GPU node.
- **CLI layer:** GraphAnything — tri-source retrieval fusing real-time
  web search, multimodal graph retrieval, and cross-document network
  traversal; graph operators (seed resolution, comparative retrieval,
  gap detection, lineage reconstruction, idea grounding); access as
  Python package, unified CLI, or MCP server (Claude Code / Codex
  skill). Storage: Neo4j + vector/graph index + BM25.
- Eval: LLM-as-judge protocol (DeepSeek-V3 as "Expert Scientific
  Editor") with per-module judicial criteria over 100 papers × 6
  domains; a geoscience KG built review-centric (114 surveys → 7,219
  papers → 602K nodes / 610K edges) judged by GPT-5.2 against review
  ground truth; FrontierScience-Research; HotpotQA / 2WikiMultiHopQA /
  MuSiQue against nine graph-augmented retrieval baselines.

## Results

- Extraction quality: AVG F1 79.07–87.11% across six domains; Module C
  (implicit knowledge) is the most robust component (F1 87.22–94.59%),
  Module E (knowledge relations) the most variable (70.54% physics →
  89.33% CS) — fine-grained relational extraction tracks each field's
  reporting norms.
- Geoscience QA: research-question rationale accuracy for Gemini-3
  rises 52.3% → 69.5% (answer 61.0% → 71.5%) with the graph vs the
  bare LLM.
- FrontierScience-Research: GPT-5.2 overall 25.2% → 39.4% (physics
  9.0% → 46.7%); Gemini-3 7.9% → 24.6%.
- Multi-hop QA: best GPT-judged accuracy on all three benchmarks
  (67.8 / 64.8 / 36.2) over KGP, RAPTOR, LightRAG, HippoRAG(2),
  GFM-RAG, E²GraphRAG, G-retriever; the margin is largest on MuSiQue,
  where flat-structure baselines collapse.
- Backbone: +3.3 F1 over its own 4B base (0.5316 → 0.5647), above the
  8B base, within 0.99 F1 of the 32B base and ahead of it on both NER
  regimes; relation extraction is the remaining gap (CoNLL04 0.3181 vs
  0.4768).

## Critique / open questions

- The headline extraction and QA numbers are all LLM-as-judge
  (DeepSeek-V3, GPT-5.2, GPT-4o-mini); no human evaluation is reported,
  and the judged systems include the judge families — circularity risk
  is unexamined.
- The tri-source CLI's web-search arm is not ablated separately in the
  main experiments, so the marginal value of live web vs the compiled
  graph is unquantified — exactly the number this project would want.
- 29 authors and a heavy stack (MinerU, 8-GPU GRPO run, Neo4j +
  vector index): "the same pipeline can be replayed on arbitrary
  corpora" is claimed, but replay is an infrastructure project, not a
  script.
- Openness is partial: 1M-paper Scholar-KG subset on HF, full 2.46M
  graph only via their hosted SCP platform.
- The substrate is external-service memory (Neo4j, embeddings, BM25)
  fronted by agent-native *interfaces* (CLI/MCP) — the inverse of
  [[concepts/agent-native-memory]]'s files-the-agent-owns stance. What
  it keeps from that pattern is provenance-per-field and typed links;
  what it drops is human readability of the store itself.

## Trust signals

- **Credibility:** 4 — major lab (Shanghai AI Lab, the
  InternAgent/Intern-Discovery line, with ECNU and Fudan), code +
  dataset (1M-paper Scholar-KG subset) + model weights all released on
  GitHub/HF; arXiv-only, not peer-reviewed, no citation record yet.

## Follow-up

- **Relevance:** 4 — the largest-scale attestation yet for
  [[concepts/llm-wiki-pattern]]'s compile-time thesis (parse once
  offline → persistent structured artifact → amortized queries, stated
  in the paper as offline-construction-vs-online-use) and the first
  *published* full schema for the knowledge half of
  [[concepts/structured-world-model]] — material new evidence for two
  existing concepts rather than a new seedling.
- Watch for: FrontierScience-Research as a research-reasoning
  benchmark for downstream agents; MinerU as a candidate parser if
  this project ever needs figure/table-level extraction from
  `raw/papers/`; whether the GraphAnything MCP server is usable
  against the public Scholar-KG subset as a `/discover` backend.
