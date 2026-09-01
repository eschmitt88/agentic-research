---
kind: paper
title: "SKILL.state: Scalable Long-Horizon Agent Skills"
authors: ["Sanket Badhe", "Priyanka Tiwari", "Jonghyun Chung"]
institutions: ["Google LLC", "Purdue University"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.26263"
code_url: null
citations: null
source: "raw/papers/badhe2026skill.pdf"
added: "2026-09-01"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: ["context-window", "agent-architecture", "skills", "memory", "long-horizon"]
---

# SKILL.state: Scalable Long-Horizon Agent Skills

## TL;DR

Agent runtimes maintain execution by appending observations, actions, and
reasoning traces to an ever-growing conversation history — causing latency
degradation and **context poisoning**. SKILL.state replaces that with an
**explicit mutable execution state**: each step the model sees only the
immutable skill spec, the current structured state, and the latest
observation. Intermediate reasoning is **discarded** after producing a
validated state update.

## Claims

- Long-horizon execution is a **systems problem, not a reasoning problem**.
  The failure is representational, not a capability limit.
- Append-only history has two costs, and the second is the important one:
  token growth, and **context poisoning** — early errors persist in the
  window and keep influencing later steps.
- Discarding intermediate reasoning after a *validated* state update is
  safe, because execution should depend on the current world state rather
  than on replaying the historical trajectory.
- The abstraction is **architecture-agnostic** — a runtime property, not a
  model or prompt trick.

## Methods

- Execution is reformulated as explicit state transitions over
  `(P, Σ_t, O_t)` — immutable procedural specification, structured
  execution state, latest observation.
- Each step produces a **validated state update**; the reasoning trace that
  produced it is discarded immediately.
- Claimed complexity: **O(1) prompt footprint**, **O(T) cumulative token
  consumption**, versus O(T) prompt and O(T²) cumulative for append-only.
- Evaluated on synthetic and real-world stateful environments (including
  Sierra τ-Bench), across proprietary and open-weight models, testing
  scaling behaviour, noise tolerance, and **state recovery**.

## Results

- Improves task accuracy **while** substantially reducing cumulative token
  consumption — the two usually trade off.
- Maintains bounded prompt sizes across execution horizons where
  history-based baselines grow without bound.
- Outperforms history-based runtimes on both accuracy and cost across
  multiple horizons and both model classes.

## Critique / open questions

- No released code, and the paper is thin on the **schema question** that
  decides whether this works: what belongs in Σ_t. Get it wrong and
  discarding reasoning becomes lossy in exactly the cases that matter.
- "Validated state update" carries the whole safety argument, but the
  validator is underspecified. If validation is itself an LLM call, the
  discard step is only as trustworthy as that call — and the discarded
  trace is precisely the evidence needed to audit it.
- Irreversibility is the real trade: append-only history is
  *reconstructible*, and an explicit state is not. The paper measures task
  accuracy but not post-hoc debuggability, which is what a research
  workflow needs.
- Small author group, no peer review, and the specific numbers are not
  foregrounded in the abstract.

## Trust signals

- **Credibility:** 3 — a Google author plus Purdue, a clean complexity
  argument, evaluation across synthetic and real environments (τ-Bench) and
  across both proprietary and open-weight models, with explicit noise and
  state-recovery probes. Held down by no released code, no peer review, a
  small team, and an underspecified validation step.

## Follow-up

- **Relevance:** 4 — reframes [[concepts/lossless-context-offload]] in a
  useful way. That concept treats offload as *moving context out* while
  preserving it; this argues the deeper win is making the retained
  representation **mutable** rather than append-only, so that superseded
  state is *overwritten* instead of accumulated. Those are different
  mechanisms with different failure modes, and "lossless" is precisely what
  SKILL.state gives up on purpose.
- Sharpens [[concepts/context-eviction-policy]]: eviction assumes the
  question is *what to drop from a growing log*. If state is mutable, most
  of that log never exists, and the policy question becomes *what to
  represent* — a schema decision made once, not a runtime decision made
  repeatedly.
- **Context poisoning as a named cost is the argument that transfers.**
  This project's `NOTES.md` / `_meta/log.md` discipline is deliberately
  append-only, and the `_meta/log.md` tail read at the start of this
  session shows the cost concretely: several successive `/promote-moc`
  entries restate and then *correct* figures inherited from earlier
  entries (the "11/17/4 in N MoCs" membership distribution was repeated
  across six entries before being corrected to 13/15/5 on 08-31). An
  append-only log propagated a wrong number for eleven days. The 08-31
  entry's own remedy — "recompute from the files rather than inherit" — is
  this paper's thesis reached independently by this repository.
- Converges with [[literature/papers/yu2026recursive]] (same curate pass),
  whose Working Memory is the same construct arrived at from the
  self-improvement side: an explicit maintained task state that grounds
  skill selection instead of a replayed transcript. Two independent
  attestations in one week.
