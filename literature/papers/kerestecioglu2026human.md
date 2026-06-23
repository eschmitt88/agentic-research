---
kind: paper
title: "Human-Inspired Memory Architecture for LLM Agents"
authors: ["Doga Kerestecioglu", "Alexei Robsky", "Clemens Vasters", "Anshul Sharma", "Yitzhak Kesselman"]
institutions: ["Microsoft"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2605.08538
code_url: null
citations: null
source: "raw/papers/kerestecioglu2026human.pdf"
added: "2026-06-23"
relevance: 3
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/context-eviction-policy]]"
tags: [memory, multi-store, consolidation, forgetting, neuroscience-inspired, knowledge-graph, longmemeval]
---

# Human-Inspired Memory Architecture for LLM Agents

## TL;DR

A **biologically-grounded, multi-tier memory architecture** for LLM agents
that maps three memory stores (short-term hot cache / medium-term episodic
store / long-term semantic graph) to three biological tiers (prefrontal
cortex / hippocampus / neocortex) and implements **six cognitive
mechanisms**: (1) sleep-phase **consolidation** (scheduled batch dedup +
promotion to long-term), (2) interference-based **forgetting** (TTL decay
+ retrieval interference), (3) engram **maturation** (newly consolidated
memories start "silent" and become retrievable over ~1-2 weeks), (4)
**reconsolidation** on retrieval (labile window, contradiction-aware
update), (5) entity **knowledge graphs** (semantic relationships), and (6)
**hybrid multi-cue retrieval** (episodic + GraphRAG). A **synthetic
calibration** method derives all pipeline thresholds from LLM-generated
corpora — no benchmark exposure — to avoid evaluation leakage. Evaluated on
a VSCode issue-tracking dataset (retention precision) and LongMemEval
(first streaming M-tier eval, 475 sessions).

## Claims

- Current agent memory falls into three deficient categories: **stateless**
  (loses context), **context-window** (scales cost not intelligence,
  cannot prioritize/forget/learn; rolling summarization compounds loss),
  and **RAG** (treats all info equally, no consolidation/forgetting, cannot
  evolve memories). A principled memory system needs the full *lifecycle*.
- **Consolidation by deduplication, not summarization**, is the dominant
  quality lever: on VSCode it yields **97.2% retention precision** with a
  **58% store reduction** (+21.8 pp over a keep-everything baseline);
  aggressive clustering/merge consolidation is *destructive* (drops
  LongMemEval S-tier to 48.4%) because merging turns destroys the specific
  details factual QA needs.
- The full pipeline is **non-destructive** on a benchmark it was *not*
  designed for: LongMemEval S-tier 76.8% vs 78.4% raw-RAG (overlapping 95%
  CI), M-tier 70.1% vs 71.2% at a 200K-token budget — the −1.1 pp gap is
  within sampling noise. Lower token budgets expose a tunable
  accuracy/store-size operating curve.
- Synthetic calibration eliminates **threshold leakage**: percentile-based
  thresholds (near-dedup at P99, interference at P90 of within-session
  similarity) transfer across domains without retuning.
- Directional wins where the design fits: **+13.3 pp S-tier preference
  recall** (dedup+recon), **+1.2 pp multi-session** and **+3.0 pp
  temporal-reasoning** at M-tier.

## Methods

- **Three-tier store.** Short-term hot cache (in-memory, TTL min-hrs);
  medium-term episodic store (full-fidelity vector store, TTL days-weeks,
  tiered caching); long-term semantic **knowledge graph** (entity
  relationships, multi-hop traversal). One unified data layer.
- **Consolidation pipeline** (sharp-wave-ripple analog, batched ~every 6h):
  five-factor **importance score** (recency, frequency, Bayesian surprise,
  entity salience, outcome) classifies events into promote (top 20%) /
  retain (60%) / prune (20%); temporal validation quarantines out-of-order
  / duplicate / causally-inverted events; promoted events become LLM-
  clustered semantic gists integrated into the graph.
- **Forgetting.** Passive exponential decay (λ=0.001, half-life ≈29 days)
  for un-consolidated events; **interference-based forgetting** computes a
  similarity-weighted interference score (retroactive 0.6 / proactive 0.4)
  and forgets high-interference low-value memories; **graceful
  degradation** through six fidelity levels (L0 full record 100% → L2
  summary 50% → L3 gist 25% → L5 tombstone 0%).
- **Maturation.** Consolidated memories enter the graph at
  activation_strength = 0 ("silent") and rise via a sigmoid (half-life 168h):
  silent → retrievable at ~1 week → mature at ~2 weeks; below threshold they
  still exert implicit *priming* on relevance scoring (implicit vs explicit
  memory).
- **Reconsolidation.** Retrieved memories enter a labile window (default
  60 min) and can be blended/updated when contradiction or new context is
  detected, with adaptive strength based on confidence/recency/severity.
- **Hybrid retrieval (GraphRAG).** Priority-ordered across tiers: hot cache
  → warm episodic vector store (importance-filtered) → semantic graph
  (activation-filtered); merged, deduped, recency-boosted.
- **Synthetic calibration.** Two LLM-generated corpora (a similarity corpus
  of 88 turns, an importance corpus of 483 turns) derive thresholds and
  signal weights via percentiles and per-signal ROC AUC — zero benchmark
  exposure. (Notably, AUC analysis collapsed the five-factor score to an
  effective four-signal weighted sum; frequency and outcome were dropped as
  degenerate on LongMemEval.)
- **Evaluation.** VSCode: 13,127 real GitHub issues / 120K events, retention
  precision (no QA task). LongMemEval: S-tier (50 sessions, 9-config
  ablation w/ bootstrap CIs) and the first **streaming M-tier** (475
  sessions, ~540K turns). GPT-4o judge, text-embedding-3-large, Azure AI
  Foundry; lifecycle mechanisms are deterministic (no LLM in the loop).

## Results

- **VSCode:** 97.2% retention precision, 58% store reduction, +21.8 pp over
  the 75.4% keep-everything baseline; store self-regulates to 300-500 events
  regardless of input volume. Graph retrieval and maturation are *not yet
  integrated* into this pipeline, so the result is a lower bound.
- **LongMemEval S-tier:** moderate configs (dedup-only, dedup+recon, etc.)
  76.2-76.8% overall, CIs overlapping the 78.4% raw-RAG baseline → non-
  destructive. Aggressive consolidation 48.4% → destructive. +13.3 pp
  preference recall (directional, wide CI at n=30).
- **LongMemEval M-tier:** dedup-adaptive at 200K tokens 70.1% vs raw-RAG
  71.2% (within noise); 115K→65.6%, 50K→49.2%, 25K→38.8% — a tunable
  accuracy/store-size curve. M-tier beats raw-RAG directionally on
  multi-session and temporal-reasoning.
- Two of six mechanisms (**maturation, reconsolidation**) have *no ablation
  evidence* — by the authors' admission LongMemEval's construction
  structurally lacks the repeated retrieval and cross-session contradictions
  they require; they remain design-rationale claims.

## Critique / open questions

- **Half the mechanisms are unvalidated.** Maturation and reconsolidation
  are unsupported by any experiment in the paper; the authors are candid
  that the benchmarks cannot exercise them. So "six mechanisms" is really
  "four with ablation + two by argument." The cognitively-motivated story is
  richer than the evidence.
- **No code released** and unknown provenance beyond "Microsoft" affiliation
  (no lab, no named senior author with a memory-systems track record). Not
  independently reproducible.
- **Headline benchmark is a custom retention-precision proxy**, not a
  downstream task — the VSCode dataset has no questions, so 97.2% measures
  "did we keep the events future activity references," not task success. The
  authors flag the gap and propose an end-to-end issue-triage agent as
  future work.
- On the *standard* QA benchmark (LongMemEval) the pipeline at best matches
  raw RAG and never beats it on aggregate; the honest framing is a "non-
  destruction bound" on a benchmark the architecture was not designed for.
  The win is store *compression* at near-parity accuracy, not accuracy.
- Interference-based forgetting + multi-level graceful degradation is a
  principled eviction story but its parameters (interference weights, decay
  λ, degradation triggers) are hand-set or synthetically calibrated, not
  learned or validated against retention outcomes.

## Trust signals

- **Credibility:** 2 — an independent arXiv preprint (v1, not peer-reviewed)
  with **no code or artifacts released**, authored under a bare "Microsoft"
  affiliation with no named lab or recognizable senior memory-systems
  researcher. The methodology has genuinely good ideas (synthetic
  calibration to dodge threshold leakage; first streaming M-tier
  LongMemEval) and the negative results are reported honestly (aggressive
  consolidation destructive; two mechanisms unvalidated; aggregate parity
  not superiority). But the central architectural claim — six cognitive
  mechanisms — is only ~40% backed by ablation, the headline number rests on
  a self-defined proxy metric, and nothing is reproducible. A
  conscientiously-written but unverifiable preprint.

## Follow-up

- **Relevance:** 3 — a **fourth independent attestation** of cognitively-
  motivated **multi-store memory** (working/episodic/semantic tiers with
  consolidation *between* tiers): this is the most explicit instantiation
  yet of [[concepts/multi-granularity-memory]], naming three biological
  tiers, scheduled sleep-phase consolidation, engram maturation, and
  interference-based forgetting as distinct lifecycle stages — strong
  material to seed that concept and help tip the memory cluster to
  MoC-ripeness. It also attests [[concepts/agent-native-memory]] (memory as
  a first-class, self-managing subsystem with its own lifecycle rather than
  a passive RAG store) and supplies a concrete eviction story for
  [[concepts/context-eviction-policy]] — interference-based forgetting +
  six-level graceful degradation (full→summary→gist→tombstone) is a
  consolidation-as-eviction mechanism distinct from the append-only-growth
  problem flagged in [[literature/papers/xu2026evoarena]]. Held at 3 (not 4)
  because half the mechanisms lack evidence, nothing is reproducible, and
  the empirical wins are compression-at-parity rather than capability gains
  — it strengthens an existing cluster rather than introducing a validated
  importable technique.
