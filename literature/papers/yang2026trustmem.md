---
kind: paper
title: "TrustMem: Learning Trustworthy Memory Consolidation for LLM Agents with Long-Term Memory"
authors: ["Tianyu Yang", "Sudipta Paul", "Vijay Srinivasan", "Vivek Kulkarni", "Srinivas Chappidi"]
institutions: ["Samsung Electronics AI Center Mountain View", "University of Notre Dame"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.25161
code_url: null
citations: null
source: "raw/papers/yang2026trustmem.pdf"
added: "2026-07-07"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [memory, consolidation, write-verification, trustworthiness, rl, grpo, hallucination]
---

# TrustMem: Learning Trustworthy Memory Consolidation for LLM Agents with Long-Term Memory

## TL;DR

Shifts memory-agent supervision from terminal task outcomes to the
individual memory *transition*: a frozen-LLM Memory Transition Verifier
scores every write/revise/prune update on coverage, preservation, and
faithfulness before it persists, and Transition-Ranked GRPO trains the
memory policy on those scores — cutting persistent memory omission,
corruption, and hallucination by 40–79% while improving downstream task
performance.

## Claims

- Memory agents that actively edit external memory (write / revise /
  delete) introduce **persistent transition-level errors** — omission
  (dropping key input facts), corruption (distorting valid prior memory),
  and hallucination (writing unsupported content) — that terminal
  task-success rewards cannot localize (severe credit-assignment gap: a
  wrong final answer may stem from an early omission; a right one may mask
  latent unsafe writes).
- Verifying each local transition (c_t, M_{t-1}, a_t, M_t) **before it
  becomes persistent** fixes this: TrustMem achieves SOTA across
  MemoryAgentBench (+6.5 over strongest baseline), HaluMem (+12.14 F1 on
  memory extraction), and the Mem-α validation set.
- Transition-level error rates drop to 8.08% omission / 0.14% corruption /
  0.01% hallucination — 40.1%, 79.1%, and 50.0% reductions vs the
  strongest baseline per error type (baselines: MemAgent, MEM1, Mem-T,
  AtomMem).

## Methods

- **Memory Transition Verifier**: a frozen LLM sees a compact view of each
  transition (current chunk, generated editing actions, touched memory
  entries — not the full memory bank) and scores it on three diagnostic
  dimensions: *coverage* (important input information preserved),
  *preservation* (valid prior memory kept without unjustified deletion or
  distortion), *faithfulness* (new content supported by chunk or prior
  state) → scalar trustworthiness v_t ∈ [0,1].
- **Multi-granularity reward**: sample-level terms (downstream task reward
  via fixed retriever + answer model; efficiency reward penalizing memory
  bloat) combined with transition-level terms (action executability,
  content specificity, verifier score).
- **Transition-Ranked GRPO**: GRPO over the scalar reward, plus a
  DPO-style ranking loss on verifier-ranked preference pairs — highest- vs
  lowest-scored candidate transitions generated from the *same* memory
  state (verifier scores treated as relative preferences, not calibrated
  absolutes), with KL regularization to a frozen reference policy.
- Unified action space {WRITE, REVISE, PRUNE} applied by a deterministic
  memory executor; chunks processed chronologically over an input stream.

## Results

- SOTA on all three evaluation settings (MemoryAgentBench, HaluMem, Mem-α
  validation set) covering accurate retrieval, test-time learning,
  long-range understanding, and memory reliability.
- Both utility and safety improve together — the verifier-guided ranking
  does not trade task performance for conservatism.

## Critique / open questions

- No released code or artifacts; results not independently reproducible
  yet.
- The verifier is itself a frozen LLM — the trust chain bottoms out in an
  unverified judge; adversarial or systematically-biased chunks could fool
  coverage/faithfulness checks (the memory-injection-attack literature the
  paper cites is a defense-oriented complement, not addressed here).
- Requires RL fine-tuning of the memory policy — heavyweight for
  harness-level agents whose memory behavior lives in prompts and skills;
  the *verifier-at-write-time* pattern transfers, the training recipe less
  so.
- Evaluated on conversational/document-stream memory; untested on
  research-agent workloads (experiment logs, skill libraries) where
  transitions are sparser and higher-stakes.

## Trust signals

- **Credibility:** 3 — reputable industry lab (Samsung AI Center Mountain
  View, with Notre Dame), solid benchmark coverage and named baselines,
  but an arXiv preprint with no peer review, no released code, no citation
  record yet.

## Follow-up

- **Relevance:** 4 — seeds `verified-memory-writes`, filling the
  consolidation-trust gap the memory cluster previously covered only on
  the retrieval side ([[concepts/selective-memory-retrieval]]); the
  write-time-verification pattern maps directly onto skill-library
  promotion and this project's own ingest discipline.
- The coverage / preservation / faithfulness triad is a reusable rubric
  for any "should this update persist?" gate — including
  [[concepts/skill-library-lifecycle]] promotion decisions.
- Watch for a code release or peer-reviewed version; also watch whether
  HaluMem-style transition-level error metrics get adopted by
  memory-agent benchmarks the graph already tracks.
