---
kind: paper
title: "ACM: Agentic Context Management for Long Horizon Tasks"
authors: ["Xiaochuan Li", "Ryan Ming", "Meng Chu", "Shuai Shao", "Rong Jin", "Chenyan Xiong"]
institutions: ["Carnegie Mellon University", "Meta"]
year: 2026
venue: "arXiv preprint (2607.23809, cs.AI)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.23809
code_url: https://github.com/lixiaochuan2020/agentic-context-management
citations: null
source: "raw/papers/li2026acm.pdf"
added: "2026-08-10"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/pass-at-k]]"
tags: [memory, context-window, compaction, eviction, lossless, agent-initiated, post-training, distillation]
---

# ACM: Agentic Context Management for Long Horizon Tasks

## TL;DR

Gives the agent two tools — `manage_context` (compress history, offload
the *raw* messages to external memory under stable identifiers) and
`query_memory` (recall by identifier on demand) — so the agent itself
decides when and how to manage its context, losslessly; a
teacher–student post-training pipeline then teaches the timing, lifting
Qwen3.5-9B by 27%/16%/8% on BrowseComp-Plus / DeepSearchQA / SWE-bench
Verified over ReAct while cutting peak tokens ~20%.

## Claims

- Existing compression is (a) lossy and (b) triggered by external
  heuristics (fixed thresholds), misaligned with the agent's reasoning
  focus. Table 1 positions ACM as the only approach that is
  simultaneously compact, trainable, lossless, agent-initiated, and
  open-data.
- **Lossless offload**: discarded messages are never deleted — each
  summary carries a unique identifier mapping to the raw messages in
  external storage; `query_memory` sends a query + identifier to a
  querier LLM over the raw span.
- **Agent-initiated timing** beats forced/threshold compression: even
  untrained, tool-equipped Qwen3.5-9B (0.635 BCP) beats ReSum (0.608)
  and ACON (0.614) summary agents.
- Frontier models under-use context management without training —
  GPT-5.5 makes near-zero `manage_context`/`query_memory` calls —
  so proactive management is a *trained* skill, not free-riding on
  scale.
- Context management improves **consistency, not only capability**:
  under ReAct, pass@4 (capability boundary) and pass⁴ (consistency)
  diverge sharply; ACM post-training raises pass@1/pass⁴ substantially
  while pass@4 moves modestly (Fig. 5: pass⁴ 34.1 → 59.3 on BCP).

## Methods

- Two-tool framework (`manage_context`, `query_memory`); compression
  runs up to the previous summary boundary via a summarizer LLM.
- **Dual-constraint teacher annotation**: student rolls out *without*
  ACM tools and the teacher injects management calls where warranted
  (loops, redundancy, context pressure); student rolls out *with*
  tools and the teacher *removes* premature/unnecessary calls,
  replacing them with search/commit actions. Trains both when to call
  and when not to.
- On-policy distillation (top-K=20 teacher token distributions,
  Qwen3.5-397B-A17B teacher → Qwen3.5-9B student, 3 epochs); rejection
  sampling keeps only trajectories where the student fails some
  trials; content filters ensure teacher rationales cite structural
  cues (redundant queries, cyclic exploration) without leaking the
  reference answer.
- Evaluated on BrowseComp-Plus, DeepSearchQA (out-of-domain, live
  search), SWE-bench Verified. Code, data, checkpoints released.

## Results

- Qwen3.5-9B + ACM post-trained: BCP 0.570 → 0.727 (near
  Qwen3.5-397B-A17B's 0.653 / Gemini3-Flash's 0.733 — a ~40× larger
  model), DeepSearchQA 0.367 → 0.425, SWE-bench Verified 0.489 →
  0.530. Peak tokens down ~20% (63k → 54k BCP).
- Sawtooth context curves (Fig. 3): compression fires well before the
  limit, driven by reasoning state; ReAct crosses the 128K limit and
  dies on questions ACM finishes at 98K peak actual (222K raw).
- Positive correlation between pass@1 and tool calls: management
  unlocks *more* exploration (most search/get_document calls of all
  agents), not less.
- Ablation vs plain distillation (Table 3): distillation alone 0.623
  BCP, ACM alone 0.727, both 0.734 — the context-management data, not
  the teacher signal, carries the gain.

## Critique / open questions

- Post-training evidence is single-family (Qwen3.5-9B student; a 4B
  appendix check) — no cross-family replication of the trained policy.
- The querier LLM at recall time is an extra failure surface (its
  errors are the new lossiness) and its cost isn't broken out; the
  "lossless" property is storage-side, not retrieval-side.
- No comparison against structured/deterministic eviction
  (semenov2026beyond's CWL) — the trained-agent-locus vs
  deterministic-policy question is posed by Table 1 but not tested
  head-to-head.
- Benchmarks are search/coding; no multi-day or revision-heavy
  workload (the lee2026minteval regime) where identifier drift would
  stress addressable recall.

## Trust signals

- **Credibility:** 4 — CMU (Xiong group) + Meta advisory; preprint
  (not yet peer-reviewed), but code, data, and checkpoints are all
  released, and the training-data pipeline is open-sourced (explicitly
  contrasted with AgentFold's closed pipeline). No citations yet
  (submitted 2026-07-26).

## Follow-up

- **Relevance:** 4 — seeds [[concepts/lossless-context-offload]] (the
  runtime-memory concept the knowledge-organization MoC split has been
  waiting on) and supplies the third-source attestation
  [[concepts/context-eviction-policy]]'s status note asked for on the
  policy-locus question — agent-initiated + trained, vs CWL's
  deterministic policy and Self-GC's plan-under-harness-validation.
- The pass@4 / pass⁴ capability-vs-consistency decomposition is worth
  importing into [[concepts/pass-at-k]] independent of the memory
  content.
- Head-to-head with semenov2026beyond-style deterministic eviction is
  the natural derive-experiment if a downstream project adopts either.
