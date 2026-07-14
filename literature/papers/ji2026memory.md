---
kind: paper
title: "Memory is Reconstructed, Not Retrieved: Graph Memory for LLM Agents"
authors: ["Shuo Ji", "Yibo Li", "Bryan Hooi"]
institutions: ["National University of Singapore"]
year: 2026
venue: ICML 2026 (PMLR 306)
peer_reviewed: true
url: https://arxiv.org/abs/2606.06036
code_url: https://github.com/Ji-shuo/MRAgent
citations: null
source: "raw/papers/ji2026memory.pdf"
added: "2026-07-14"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
tags: [memory, graph-memory, active-reconstruction, retrieval-policy, granularity, long-horizon]
---

# Memory is Reconstructed, Not Retrieved: Graph Memory for LLM Agents

## TL;DR

Replaces the static "retrieve-then-reason" memory pipeline with **active
reconstruction**: MRAgent organizes memory as a Cue–Tag–Content
associative graph with episodic / semantic / topic layers, and puts LLM
reasoning *inside* the memory-access loop — iteratively selecting
traversal actions, pruning branches, and stopping when accumulated
evidence suffices — with a proof that stateful active retrieval is
strictly more expressive than any fixed passive policy.

## Claims

- Existing memory systems — similarity top-k (RAG, Mem0), predefined
  N-hop graph expansion (A-Mem, Zep), hierarchical stores (MemoryOS) —
  are all *passive* policies: the retrieval set is a fixed function of
  the query, so intermediate evidence discovered mid-reasoning cannot
  redirect the search.
- Cognitive framing: human recall is reconstruction — contextual cues
  reactivate engrams that progressively rebuild the memory — not a
  one-shot readout; MRAgent operationalizes this with cue-initiated,
  evidence-conditioned graph traversal.
- Theorem 4.1: for any retrieval budget T ≥ 2, the passive hypothesis
  class is strictly contained in the active one — LMs with adaptive
  retrieval can implement everything passive retrieval can, not vice
  versa.
- Deferring relational reasoning to retrieval time lets construction stay
  lightweight: relations are resolved query-specifically instead of
  being exhaustively precomputed and summarized at write time.

## Methods

- **Cue–Tag–Content graph**: fine-grained cues (entities, attributes)
  link to memory contents through typed relation *tags*; tags act as
  semantic bridges so the LLM selects retrieval paths before touching
  full content, avoiding combinatorial n-hop expansion.
- **Multi-granular layers**: episodic (event-specific units on a unified
  timeline), semantic (stable knowledge distilled across events,
  anchored to entity cues), abstraction (topic nodes summarizing
  recurring patterns, with top-down topic→episode transitions).
- **Reconstruction loop**: state = (active set, accumulated context);
  per step the LLM selects forward (cue→tag, (cue,tag)→content) or
  reverse (content→cue,tag) traversal actions, the graph executes them,
  and an LLM routing function prunes candidates and decides
  sufficiency/termination.
- Population via LLM distillation: episodes segmented from the stream;
  tags + cues extracted per unit; semantic and topic layers distilled
  above them.
- Eval: LoCoMo and LongMemEval, Gemini-2.5-Flash and Claude-Sonnet-4.5
  backbones, vs RAG / A-Mem / MemoryOS / LangMem / Mem0.

## Results

- LoCoMo overall LLM-judge: 68.31 → 84.21 (Gemini, +23.3% relative over
  best baseline Mem0) and 69.02 → 88.32 (Claude); largest gains on
  multi-hop (75.17 vs 68.79 J) and temporal questions.
- LongMemEval: 72.95 overall vs 54.92 best baseline (+32% relative).
- Cost: 118k prompt tokens per sample vs 245k (Mem0) and 632k (A-Mem);
  runtime second-best (586 s vs Mem0's 533 s) — the structure *reduces*
  token cost because tags gate access to expensive episodic content.
- Ablations: reasoning-in-the-loop is the primary factor (with-reasoning
  beats structure-only for every memory variant); associative tags help
  monotonically (CE → CTE → CTC) even without reasoning; removing the
  semantic layer clearly degrades multi-hop.
- Multi-turn analysis: multi-hop recall improves >30% across successive
  reasoning turns; Max Valid Turns closely tracks Average Turns — the
  agent's own sufficiency judgment decides when to stop, and extra
  parallel retrieval budget cannot substitute for deeper reconstruction.

## Critique / open questions

- Construction is static: no update, consolidation, or forgetting — the
  graph grows monotonically with interaction history, which the authors
  flag as a limitation for long-lived deployments. The write-side
  lifecycle (eviction, consolidation) is exactly what it lacks.
- Reconstruction cost scales with exploration depth; many-step queries
  pay real latency vs single-shot retrieval, and the token win reported
  is against notably heavyweight baselines.
- Benchmarks are conversational QA (LoCoMo, LongMemEval); nothing shows
  the mechanism on task-execution or research-agent memory, where cue
  extraction from non-dialog artifacts may be harder.
- The expressivity theorem is about hypothesis-class containment, not a
  guarantee the LLM router *finds* the better policy; the empirical gap
  carries the argument.

## Trust signals

- **Credibility:** 4 — peer-reviewed at ICML 2026, known group (NUS,
  Bryan Hooi), code released; not yet cited or independently reproduced,
  and results were produced with closed-model backbones.

## Follow-up

- **Relevance:** 4 — third-paradigm attestation for
  [[concepts/multi-granularity-memory]]: alongside MemGAS's
  entropy-routed grains (xu2026single) and H-MEM's hierarchy, MRAgent
  supplies *reconstruction-vs-retrieval* — the grains (episodic /
  semantic / topic) are traversed by in-loop reasoning rather than
  routed to by a similarity heuristic. Also extends
  [[concepts/selective-memory-retrieval]] from "when to read" to "how
  long to keep reading": termination is an evidence-sufficiency judgment
  by the agent, and the routing signal is accumulated evidence, not
  query similarity.
- The tag-gated two-stage access pattern (select cheap associative tags
  before touching expensive content) is the same shape as this project's
  index-then-body read discipline over concept notes — one-line
  `index.md` pointers as tags, note bodies as content.
- Watch citations; try the released code if a downstream project builds
  an agent-memory experiment.
