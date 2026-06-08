---
kind: paper
title: "Agent Memory: Characterization and System Implications of Stateful Long-Horizon Workloads"
authors: ["Yasmine Omri", "Ziyu Gan", "Zachary Broveak", "Robin Geens", "Zexue He", "Alex Pentland", "Marian Verhelst", "Tsachy Weissman", "Thierry Tambe"]
institutions: ["Stanford University", "MICAS, KU Leuven", "Massachusetts Institute of Technology"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.06448
code_url: null   # phase-aware profiling harness announced "to be open-sourced"; no URL yet
citations: null
source: "raw/papers/omri2026agent.pdf"
added: "2026-06-08"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
tags: [memory, systems, taxonomy, profiling, retrieval, write-path, long-horizon, benchmark]
---

# Agent Memory: Characterization and System Implications of Stateful Long-Horizon Workloads

## TL;DR

The first *systems* characterization of agent memory (as opposed to the
accuracy-only benchmarks that dominate). Introduces a four-axis taxonomy
(construction / storage / retrieval / mutability), a phase-aware profiling
harness that attributes cost to construction, retrieval, and generation, and
characterizes ten representative memory systems across two benchmark suites —
yielding 10 deployment-level system recommendations. The key reframing:
agent memory is "a mutable state produced by the agent's own interaction
stream… with a write path, a search path, and an ongoing maintenance policy,"
not a static retrieval target.

## Claims

- Existing memory benchmarks evaluate only downstream accuracy, leaving
  system-level behavior (cost, bandwidth, latency, scalability)
  uncharacterized — yet those costs "dominate at deployment scale" and are
  invisible to accuracy metrics.
- Design choices redistribute cost *asymmetrically*: compressing history into
  atomic facts slashes query-time prompt length but pays orders-of-magnitude
  more in construction-time LLM prefill; a graph-based memory improves
  relational retrieval but multiplies embedding traffic and storage footprint.
- The construction (write) path takes one of four forms — **absent** (raw
  history in prompt), **deterministic** (chunk/index, no LLM),
  **LLM-mediated** (extraction/summary at predefined points), or **agentic**
  (LLM-controlled writes, tool invocation, record mutation).
- Memory naturally factors into a two-tier hierarchy: short-term working
  memory (current context + retrieved entries) and long-term persisted
  memory, with explicit "Forget" operations on both tiers.

## Methods

- Seven-stage execution pipeline: ingestion → memory construction → storage →
  retrieval → prompt assembly → generation → maintenance.
- Four-axis system-oriented taxonomy with predicted cost signatures per
  paradigm.
- Phase-aware profiling harness ("to be open-sourced") tracking tokens, model
  calls, GPU utilization, and latency across construction, retrieval, and
  generation.
- Characterizes ten representative systems over two benchmark suites; derives
  10 recommendations on construction scheduling, capability floors,
  amortization via query volume, freshness-latency tradeoffs, and footprint.

## Results

- Construction cost, capability thresholds, amortization structure, freshness
  scheduling, footprint growth, and retrieval tail behavior all vary sharply
  by paradigm and are not predicted by accuracy benchmarks.
- 10 concrete system recommendations for agent-memory serving infrastructure,
  scheduling, and system selection.

## Critique / open questions

- A characterization/taxonomy paper, not a new mechanism — its value is the
  cost model and the vocabulary, not a SOTA number.
- Profiling harness is announced as open-source but not yet released; the 10
  systems and two suites are not named in the front matter, so reproducibility
  hinges on the eventual release.
- The two-tier "Forget" framing maps cleanly onto `context-eviction-policy`
  but the paper treats eviction as a serving-cost lever, not a fidelity lever
  — worth reconciling with the recall-degradation ("U-shaped" context) view.

## Trust signals

- **Credibility:** 4 — strong, diverse institutions (Stanford, MIT/Pentland,
  KU Leuven/Verhelst, Weissman) and a careful systems methodology, but an
  arXiv preprint with the profiling harness not yet released and no
  independent reproduction.

## Follow-up

- **Relevance:** 4 — material new *systems-cost* evidence for the memory
  cluster: the four construction paradigms sharpen `agent-native-memory`
  (its "agentic" paradigm is exactly agent-native write-path), the read-path
  cost analysis grounds `selective-memory-retrieval`, and the two-tier
  "Forget" model grounds `context-eviction-policy`. A supporting attestation
  toward a memory Map of Content.
- The two-tier hierarchy + per-turn/session/topic construction granularity is
  a fifth attestation for the still-unseeded `multi-granularity-memory`
  concept NOTES flagged — re-check MoC-ripeness once that concept is created.
- Watch for the profiling-harness release; if it lands, it is a candidate
  reference for any downstream memory experiment's instrumentation.
