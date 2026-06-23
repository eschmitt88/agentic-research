---
kind: paper
title: "EvolveMem: Self-Evolving Memory Architecture via AutoResearch for LLM Agents"
authors: ["Jiaqi Liu", "Xinyu Ye", "Peng Xia", "Zeyu Zheng", "Cihang Xie", "Mingyu Ding", "Huaxiu Yao"]
institutions: ["UNC-Chapel Hill", "UC Berkeley", "UC Santa Cruz"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2605.13941
code_url: https://github.com/aiming-lab/SimpleMem
citations: null
source: "raw/papers/liu2026evolvemem.pdf"
added: "2026-06-23"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/evolutionary-expansion]]"
tags: [memory, retrieval, evolution, autoresearch, self-improvement, action-space, transfer, long-horizon]
---

# EvolveMem: Self-Evolving Memory Architecture via AutoResearch for LLM Agents

## TL;DR

EvolveMem treats an agent's **retrieval configuration** — fusion mode, per-view
weights, context budget, query-augmentation toggles, answer-generation style,
per-category overrides — as a **structured action space** and evolves it with
an **LLM-powered diagnosis module** that reads per-question failure logs,
categorizes root causes, and proposes targeted config adjustments. A guarded
meta-analyzer applies each proposal with automatic revert-on-regression and
explore-on-stagnation safeguards, realizing an "AutoResearch over your own
memory config" loop: starting from a minimal BM25-only baseline it converges
autonomously and **discovers retrieval dimensions not present in the original
action space**.

## Claims

- Existing memory systems evolve stored *content* (consolidation, forgetting,
  knowledge graphs) but keep the **retrieval infrastructure frozen** at
  deployment — scoring functions, fusion weights, context budgets, and
  answer-generation policies never adapt. The paper argues truly adaptive
  memory must **co-evolve at two levels**: the stored knowledge *and* the
  retrieval mechanism that queries it.
- Different question types need fundamentally different retrieval strategies
  (factual → exact keyword; temporal → recency-weighted; multi-hop → query
  decomposition; adversarial name-swap → person-name stripping), so a single
  frozen config cannot serve all of them — and the mismatch worsens as the
  store grows from dozens to hundreds of records.
- Exposing retrieval as a structured action space and searching it with an
  **LLM diagnosis module** (rather than grid/Bayesian search) lets the system
  conduct the observe→hypothesize→experiment→validate cycle autonomously,
  replacing manual config tuning — an AutoResearch process applied to the
  system's *own* architecture.
- Evolved configs **transfer across benchmarks with positive (not
  catastrophic) transfer**, indicating the loop captures universal retrieval
  principles rather than benchmark-specific heuristics.
- Headline gains: on LoCoMo, +25.7% relative over the strongest published
  baseline and +78.0% relative over the minimal baseline; on MemBench, +18.9%
  relative over the strongest baseline.

## Methods

- **Three layers + a self-evolution feedback loop** (Fig. 2). *Layer 1 —
  Structured Memory Store*: an LLM extractor with retry, chunk-splitting on
  context overflow, and coverage verification populates a typed store; each
  unit is a tuple (content, embedding, type ∈ 6-category taxonomy, metadata
  with importance / entity-reinforcement / entities / topics / timestamp).
  Three consolidation passes — Jaccard dedup, linear importance decay with a
  floor, entity-reinforcement increment on query co-occurrence.
- **Layer 2 — Multi-view retrieval**: three independent views (lexical/BM25,
  semantic/dense, structured-metadata) each return top-k; fused under a mode ∈
  {SUM, WEIGHTED-SUM, RRF}. Final ranking adds memory-intrinsic signals
  (importance, recency, entity-reinforcement). Optional **query augmentation**:
  adversarial entity-swap (strip person names, re-search by topic, union) and
  query decomposition (split multi-hop into single-hop sub-queries, merge via
  RRF). Answer generation has a configurable style plus a second-pass verifier
  for low-confidence responses.
- **Action space** θ = (k_sem, k_kw, k_str, B_ctx, mode, {w_v}, α, {θ_c}_{c∈C}):
  per-view candidate counts, context budget, fusion mode, per-view weights,
  answer-generation style, and **per-category sub-configs θ_c** that can
  override any global parameter. Every dimension is clamped to a safe range
  before a proposal is applied.
- **Layer 3 — Self-Evolution Engine** (Algorithm 1). Each round writes a
  per-question raw log (question, prediction, ground truth, score, retrieved
  sources); the **diagnosis LLM** reads it under a rubric written in terms of
  *failure patterns* (wrong entity retrieved, insufficient context, temporal
  confusion) rather than benchmark specifics — so newly-discovered config
  dimensions become usable immediately. The proposed Δθ goes through a
  **meta-analyzer update rule** with three branches: revert to best-so-far if
  score drops > τ_rev; random perturbation (explore) if score is flat for 2
  rounds; else clamp-and-apply. If diagnosis detects missing coverage it
  triggers targeted re-extraction, closing the loop back to Layer 1.
  Terminates when round-over-round improvement < ε or at R_max (=7 here).
- **Eval**: LoCoMo (multi-session dialogue, token-F1/BLEU-1, 5 QA categories,
  10 conversations / 1,986 pairs) and MemBench (memory-tool-use, exact-match
  accuracy, 7 categories). Backbones GPT-4o and GPT-5.1. Storage SQLite/FTS5,
  embeddings BAAI/bge-base-en-v1.5. θ_0 = BM25-only SUM, semantic+structured
  disabled, k_kw=5, B_ctx=8 — a deliberately minimal start.

## Results

- **LoCoMo (GPT-4o)**: overall F1 0.543 vs SimpleMem 0.432 (strongest
  baseline) = +25.7% relative; largest category gains temporal (+63.4%) and
  single-hop (+68.7%). On GPT-5.1 it leads every column (overall 0.572,
  +36.8% rel over SimpleMem). Gains consistent across backbones → not
  model-specific.
- **MemBench**: best overall on both backbones (67.9% GPT-4o, 71.4% GPT-5.1),
  +18.9%/+11.0% rel over the strongest baseline. Gains concentrate in Recall
  (+40.0%) and Reasoning (+33.4%); **Robustness is the weakest dimension** and
  failure-log inspection localizes it to POST_PROCESSING where relevant
  memories are *absent from the store* — a coverage limit retrieval-level
  tuning cannot fix.
- **Evolution trajectory (Table 4)**: 30.5% (R0) → 54.3% (R7) on LoCoMo,
  fully autonomous. R2 shows the revert guard (a proposed MMR-diversity change
  regressed F1 and was rolled back). Three dimensions the diagnosis LLM
  *discovered* — adversarial entity-swap (R3), query decomposition (R5),
  answer verification (R7) — jointly contribute +7.77 F1 beyond the initial
  action space, each independently verifiable in ablation.
- **Transfer (Table 5)**: a config evolved on LoCoMo zero-shots to 54.3% on
  MemBench (vs evolving from scratch 67.9%); *continued* evolution from the
  LoCoMo prior reaches 79.2% MemBench (+16.6% rel over scratch) **while also
  improving LoCoMo 0.543→0.593** — a Pareto improvement on both.
- **Ablations (Table 6)**: removing extraction-quality control is most damaging
  (−23.22 F1, nearly halves extraction yield), then semantic search (−10.32),
  then **LLM-powered diagnosis (−9.63)** — i.e. replacing the diagnosis module
  with random search over the same action space costs ~9.6 F1, confirming that
  reading per-question failure logs provides meaningful search signal. No
  single component dominates the −23 to −2 range → discovered components are
  complementary, not redundant.

## Critique / open questions

- **Diagnosis cost is not accounted.** Every round runs a full evaluation pass
  plus an LLM diagnosis call over per-question logs; up to R_max=7 rounds. The
  paper relegates efficiency to an appendix and the body never states the
  token/wall-clock cost of evolution relative to the F1 gained — the central
  trade-off for adopting this over a one-shot tuned config.
- **The "novel dimensions" are drawn from a known menu.** Entity-swap, query
  decomposition, and answer verification are all standard adaptive-RAG moves;
  "not present in the original action space" means not in *θ_0*, not novel to
  the literature. The diagnosis LLM is selecting from techniques it has seen,
  so "discovery" is really guided activation, not invention.
- **Robustness ceiling is a coverage problem, by the authors' own analysis** —
  retrieval evolution cannot help when the memory simply lacks the content.
  This bounds the framework: it optimizes the *retrieval* half of the
  co-evolution claim far more than the *store-content* half.
- **Two benchmarks, both QA-over-dialogue.** Transfer is shown LoCoMo↔MemBench;
  whether evolved retrieval principles generalize to genuinely different agent
  workloads (tool use, code, multimodal — flagged as future work) is untested.
- **Code link points at SimpleMem**, the authors' prior repo / strongest
  baseline, not an `EvolveMem`-named artifact — release completeness for the
  evolution engine specifically is unverified from the paper.

## Trust signals

- **Credibility:** 3 — coherent UNC-Chapel Hill / UC Berkeley / UCSC
  collaboration with a recognized senior author (Huaxiu Yao) and a released
  code repo, plus thorough ablations and a transfer study. Held at 3 (not 4):
  an arXiv v1 preprint with no peer review, only two same-genre benchmarks,
  the headline framing ("discovers new dimensions", "AutoResearch") oversells
  what is guided selection from a standard adaptive-RAG menu, and the released
  code is the prior SimpleMem repo rather than a clearly distinct EvolveMem
  artifact.

## Follow-up

- **Relevance:** 4 — this is a clean instance of **retrieval policy as a
  learned/evolved object rather than a fixed heuristic**, the core thesis of
  [[concepts/selective-memory-retrieval]]: the gating decisions (which view,
  what fusion, whether to decompose, whether to verify) are per-category and
  *searched*, not hand-set, and the ablation isolates that the LLM-diagnosis
  search beats random search by ~9.6 F1. It instantiates
  [[concepts/agent-native-memory]] on the retrieval side — the memory system
  carries and reasons over its own per-question failure logs to rewrite its
  config. And the EVALUATE→DIAGNOSE→PROPOSE→GUARD loop with revert-on-regression
  and explore-on-stagnation is a memory-flavored
  [[concepts/evolutionary-expansion]] search: a guarded select/mutate/keep loop
  over a config action space, with positive cross-benchmark transfer as the
  generalization signal. Held at 4 rather than 5 because it strengthens three
  existing concepts rather than seeding a new importable one, and the
  cost-accounting gap leaves the practical adoption case under-argued.
