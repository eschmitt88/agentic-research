---
kind: paper
title: "Addressable Recall Compaction for Long Context-Window Control in AI Agents"
authors: ["Thang Dang", "Yuma Ichikawa", "Sakina Fatima", "Koichi Shirahata"]
institutions: ["Fujitsu Research of America", "Fujitsu Limited Japan", "RIKEN Center for AIP"]
year: 2026
venue: "arXiv preprint (2607.25066, cs.AI)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.25066
code_url: null
citations: null
source: "raw/papers/dang2026addressable.pdf"
added: "2026-08-11"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [memory, context-window, compaction, eviction, lossless, addressability, recall, serving-cost, formal-guarantee]
---

# Addressable Recall Compaction for Long Context-Window Control in AI Agents

## TL;DR

ARC separates "what to keep" (everything, verbatim, in an append-only
content-addressed store) from "what to show" (a bounded active view
where evicted observations become `§id` citations the agent can
`_recall` on demand), proves an informal theorem that the active view
stays ≤ budget while every observation remains exactly recoverable,
and closes the recall gap lossy compaction leaves open: 99.40% vs
88.12% best-baseline on needle-in-a-haystack, +1.6–2.3 points on
LongBench-v2 Hard, while cutting HBM traffic 38.8–73.5%.

## Claims

- All four mainstream compaction paradigms — truncation, LLM summary,
  structured state, RAG memory — are *lossy by construction*: once
  content is pruned, paraphrased, or vectorized, exact recovery is no
  longer guaranteed. The organizing axis for compaction methods is
  whether a discarded value is **recoverable** (some strategy exists
  by which the agent can retrieve its exact original form) or lost.
- Every baseline conflates two decisions that need not be coupled:
  what to keep and what to show the model right now. ARC keeps
  everything, always, in a content-addressed store; omission from the
  visible transcript leaves an explicit dereferenceable pointer.
  "Recovery is then a decision that the agent makes, not a
  probability that the retriever gets right."
- **Theorem 1 (informal ARC guarantee)**: with injective fixed-width
  ids, store-before-remove, an external citation catalog with one
  token-bounded page materialized in-prompt, and enforced view budget
  K ≤ L, every model input is ≤ K tokens regardless of elapsed turns,
  and any stored observation is exactly reconstructible from
  ⌈|O|/q⌉ non-overlapping recall chunks. The appendix also argues
  linear external-memory growth is *unavoidable* for worst-case exact
  recovery — ARC's append-only growth matches the lower bound.
- The guarantee is deliberately scoped: address-conditioned
  recoverability (the mechanism's property) is distinct from
  end-to-end task success (the agent must still choose the right
  address).

## Methods

- **Store**: every tool observation hashed (SHA1 over action signature
  + observation) into an append-only `ObsStore`; visible ids are 8-hex
  prefixes extended only on collision. Records keep full content,
  head/tail previews, length metadata, return code, creation step.
- **Compaction routine is deterministic — no LLM call**: truncates
  older thoughts, keeps short observations inline, replaces long ones
  with `§id` citations (head preview + tail preview + metadata +
  `_recall §id` hint), retains recent turns verbatim; if still over
  budget, already-cited entries collapse to bare `§id` stubs. Once an
  id has appeared as a recall handle, later compactions preserve the
  handle (`Cited` set) rather than re-summarizing.
- **Recall is agent-initiated and budgeted**: `_recall §id`
  intercepted by the framework; invalid ids return a nearest-match
  suggestion instead of failing silently. Per-step recall limit
  (default 2), total materialized-recall budget, per-recall max
  length; LRU-recalled bodies evict back to citation stubs. A lighter
  `_recall_meta` restores citation-level metadata only.
- Evaluated on Qwen3-8B (16k) / Qwen3-32B (32k) under vLLM with
  reactive compaction on context-overflow; NIH (1000 tasks × 3 seeds)
  with an anti-self-doubt guardrail (banner-labeled needle, scored on
  submitted `NEEDLE=<value>` token), LongBench-v2 Hard (311 × 3
  seeds), McNemar paired tests throughout; serving cost via an H200
  roofline model (prefill FLOPs + decode bytes streamed / HBM
  bandwidth).

## Results

- NIH: 99.00% (8B) / 99.80% (32B) vs best baseline RAG_memory 79.57% /
  96.67%; Full_context collapses (36.8% / 2.93%) — larger windows
  alone do not solve precise retrieval. Lowest seed variance (0.17%).
- LongBench-v2 Hard: 27.47% / 32.47%, +1.64 / +1.80 over the
  strongest baseline; no-answer rate 16.33 / 7.33 vs 23–74 for every
  baseline. Margin narrows vs NIH because hard tasks add multi-step
  reasoning on top of recall — ARC's advantage is a lower bound on
  how much failure is attributable to recall.
- Serving: 38.8% (8B) / 73.5% (32B) HBM-traffic savings vs
  sliding-window on LongBench; on NIH, 80.3% traffic cut with total
  time 6.47s → 1.26s. Only method that improves accuracy *and*
  reduces memory traffic/decoding cost.
- All paired comparisons significant (p ≪ 0.001), odds ratios 2.2–∞.

## Critique / open questions

- Qwen3-only; the `_recall` convention's dependence on
  instruction-following across model families is explicitly open.
- No released code found in the paper — the formal appendix helps,
  but the deterministic-compaction details (preview lengths, budget
  split across the five view components) would be fiddly to
  re-implement faithfully.
- Only tool *observations* are stored verbatim; discarded reasoning
  traces are recoverable only if separately stored — the theorem
  explicitly excludes them. An agent whose needle lives in its own
  thought stream is unprotected.
- Implicit retrieval still wins on dialogue-history tasks (their own
  admission) — explicit and implicit recall look complementary, not
  competing.

## Trust signals

- **Credibility:** 3 — industrial lab (Fujitsu Research × RIKEN AIP)
  with formal appendix and unusually careful paired statistics, but
  arXiv-only, no released code, single model family, no citations yet
  (v1 late July 2026).

## Follow-up

- **Relevance:** 4 — fourth attestation for
  [[concepts/lossless-context-offload]] and the strongest *formal*
  one (theorem + matching lower bound); also the cleanest statement
  of the keep/show decoupling, and the first to price the invariant
  in serving cost (HBM roofline), not just accuracy.
- The split policy locus is new evidence on the open axis: write-side
  compaction is fully deterministic (no LLM), read-side recall is
  agentic — contrast ACM (agent-initiated both sides, trained) and
  CWL (deterministic both sides).
