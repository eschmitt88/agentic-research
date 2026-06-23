---
kind: paper
title: "From Single to Multi-Granularity: Toward Long-Term Memory Association and Selection of Conversational Agents"
authors: ["Derong Xu", "Yi Wen", "Pengyue Jia", "Yingyi Zhang", "Wenlin Zhang", "Yichao Wang", "Huifeng Guo", "Ruiming Tang", "Xiangyu Zhao", "Enhong Chen", "Tong Xu"]
institutions: ["University of Science and Technology of China", "City University of Hong Kong", "Huawei Technologies Ltd.", "Dalian University of Technology"]
year: 2026
venue: "ICLR 2026"
peer_reviewed: true
url: https://openreview.net/forum?id=i2yIvZARnG
code_url: https://github.com/Applied-Machine-Learning-Lab/ICLR2026_MemGAS
citations: null
source: "raw/papers/xu2026single.pdf"
added: "2026-06-23"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
tags: [memory, multi-granularity, entropy-routing, retrieval, GMM, personalized-pagerank, conversational-agents, long-term-memory]
---

# From Single to Multi-Granularity: Toward Long-Term Memory Association and Selection of Conversational Agents

## TL;DR

**MemGAS** is a training-free long-term memory framework for conversational
agents that stops treating memory as a single fixed granularity. Each session
is stored as **four co-existing granularity levels** — session, turn,
LLM-generated summary, and LLM-generated keywords — that are wired together by
a **Gaussian Mixture Model** clustering new memories against history into
accept/reject sets, and retrieved through an **entropy-based router** that
adaptively weights each granularity by how *certain* the query's similarity
distribution is at that level. Retrieval runs Personalized PageRank over the
multi-granular association graph, then an LLM filter removes redundancy.

## Claims

- Existing retrieval-augmented memory systems depend on a **single fixed
  granularity** (session chunks, turn-level segments, or LLM summaries). This
  causes either *partial retrieval* of useful information (missing
  cross-granular links) or *substantial noise*, both yielding suboptimal QA.
- Two specific gaps: **(i) insufficient multi-granularity connection** —
  methods organize memory into graphs/trees at a *single* scale (entities OR
  session summaries) and so fail to model cross-granular interactions; and
  **(ii) lack of adaptive multi-granular selection** — fixed
  session/turn/summary strategies cannot pick the best granularity per query.
- An empirical analysis (Fig. 1a) shows that adaptively choosing the
  best-suited granularity per query — balancing noise reduction in summaries
  /keywords against information retention in raw sessions — yields substantial
  gains, motivating granularity *selection* as the central lever (recall jumps
  from ~41–45 at any single granularity to ~63 with "suited gran").
- MemGAS outperforms SOTA baselines on **both QA and retrieval** across four
  long-term memory benchmarks, consistently across query types and top-K.

## Methods

- **Multi-granular memory units.** For each session, an LLM generates a
  summary `U_i` and keywords `K_i`, and the session is segmented into turns
  `T_i`; the stored unit `M_i = {S_i, T_i, U_i, K_i}` holds all four
  granularities (session / turn / summary / keyword).
- **Dynamical memory association via GMM.** When a new memory is added, every
  granularity of every element is encoded to a dense vector (Contriever).
  Pairwise similarities between the new memory and all history are clustered
  by a **Gaussian Mixture Model** into an **accept set** (relevant, gets edges
  in the association graph) and a **reject set** (excluded). Edges are added
  per-granularity, so the new memory's session/turn/summary/keyword nodes each
  connect to matching history nodes — mimicking human-like memory
  consolidation by selectively reinforcing related memories.
- **Entropy-based router.** For a query, similarity scores to all memories at
  each granularity `g` are softmax-normalized into a distribution; its
  **Shannon entropy `H^g`** measures matching uncertainty. Granularity weights
  are inverse-entropy normalized (`w^g ∝ 1/H^g`), so granularities where the
  query matches *confidently* (low entropy → a clear correspondence) are
  up-weighted — no manual granularity choice needed. `λ` controls entropy
  temperature.
- **Retrieval + filter.** Each granularity node gets an initial relevance
  score `w^g · sim(q, M_i^g)`; the top-α become seed nodes for **Personalized
  PageRank** over the association graph, propagating relevance to nodes both
  query-relevant and densely connected to high-value nodes. Top-K nodes are
  passed through an **LLM-based redundancy filter** to discard
  irrelevant/repetitive content before generation.
- **Setup.** Training-free; backbone `gpt-4o-mini-2024-07-18`, temperature 0,
  zero-shot, Contriever embeddings, top-3 sessions, consistent across all
  baselines.

## Results

- Benchmarks: **LoCoMo, LongMemEval-s, LongMemEval-m, LongMTBench+**. Metrics:
  GPT4o-as-Judge, F1, BLEU-4, ROUGE-1/2/L, BERTScore (QA); Recall@k / NDCG@k
  (retrieval). MemGAS is **best on most QA metrics on every dataset** — e.g.
  LongMemEval-s GPT4o-J 60.20 (next best 57.60 HippoRAG2), F1 20.38 (vs 14.73);
  LongMTBench+ F1 41.49 vs 37.69. Wins are large on the underlying retrieval
  task: **Recall@3 78.51 vs 75.53** (LongMemEval-s), and it takes the top
  Recall/NDCG at every k on all retrieval datasets.
- Efficiency is competitive: avg tokens ~8.8k (vs 103k–128k for Full History),
  latency 1.9–3.9s, comparable to or below several baselines (A-Mem,
  RecurSum); LLM API calls account for >98% of latency, so the framework's own
  overhead is minimal.
- **Ablation (LongMemEval-s):** removing any module degrades both QA and
  retrieval. Removing all (GMM + PPR + MA + Router) drops F1 from 20.38 → 13.78
  and Recall@3 from 78.51 → 71.06. The latency each module adds is tiny
  (≤0.019s QA, ≤0.008s retrieval), so the gains are nearly free.

## Critique / open questions

- The end-to-end QA gains over the strongest baseline are sometimes modest
  (GPT4o-J +2.6 on LongMemEval-s, +2.0 on LongMTBench+ over Full History); the
  decisive, consistent advantage is on the **retrieval** sub-task, which is
  the cleaner claim. Whether better retrieval always converts to better answers
  is dataset-dependent.
- All four granularities are **generated up-front by an LLM at write time** —
  the summary/keyword generation cost (and its dependence on a capable
  backbone) isn't separated from retrieval cost, and the framework is only
  tested with `gpt-4o-mini` as both generator and judge (GPT4o-as-Judge with a
  GPT-family generator risks evaluator-generator affinity).
- The association graph grows per-write with per-granularity edges; like other
  append-style memory, **graph growth / pruning over very long deployments**
  is not addressed (the context-eviction question).
- Scope is **conversational personal-assistant memory**, not research-agent
  memory; the multi-granular + entropy-gating mechanism is architecture-level
  and plausibly transfers, but that is unattested here.

## Trust signals

- **Credibility:** 4 — **peer-reviewed and published at ICLR 2026**, with code
  released (github.com/Applied-Machine-Learning-Lab/ICLR2026_MemGAS), a strong
  evaluation across four established long-term-memory benchmarks against nine
  competitive baselines (HippoRAG2, RAPTOR, A-Mem, SeCom, …), and a clean
  ablation isolating each module's contribution. Academia–industry team
  (USTC, CityU HK, Huawei, Dalian UT). Held at 4 rather than 5 only because
  evaluation leans on a single small backbone (`gpt-4o-mini`) used as both
  generator and judge.

## Follow-up

- **Relevance:** 4 — this is an almost-exact anchor for the new
  [[concepts/multi-granularity-memory]] concept: it makes the case empirically
  (single-granularity is the failure mode) and supplies a concrete mechanism
  (four co-stored granularities + a cross-granular association graph + a router
  that *selects* granularity per query). The entropy-based router is a clean
  **uncertainty-gated retrieval** signal for
  [[concepts/selective-memory-retrieval]] — it gates *which granularity* to
  trust by the certainty of the query's match distribution, a gating axis
  distinct from EvoArena's version/temporal gating. The LLM-generated
  summaries/keywords-as-memory and the consolidation-graph framing also
  instantiate [[concepts/agent-native-memory]]. Held at 4 (not 5) because the
  domain is conversational assistants rather than research agents, so it
  strengthens importable concepts rather than directly driving an experiment
  here.
