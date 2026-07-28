---
kind: concept
name: "hierarchical-delegation"
status: experimental
added: "2026-04-24"
source_papers:
  - zhang2026aibuildai
  - chen2026toward
sources:
  - "[[literature/papers/calboreanu2026iterative]]"
  - "[[literature/papers/liu2026dive]]"
  - "[[literature/papers/ning2026code]]"
  - "[[literature/papers/pepe2026agentic]]"
  - "[[literature/papers/wang2026reframing]]"
  - "[[literature/repos/vila-lab-dive-into-claude-code]]"
  - "[[literature/posts/openclaw2026acp]]"
  - "[[literature/posts/openclaw2026harness]]"
  - "[[literature/papers/zhang2026aibuildai]]"
  - "[[literature/papers/chen2026toward]]"
  - "[[literature/papers/du2026mlevolve]]"
  - "[[literature/papers/ye2026agent]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/xin2026eurekagent]]"
used_by: []
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/typed-enforcement]]"
related_experiments: []
tags: [agent-architecture, delegation, roles, coordination]
---

# hierarchical-delegation

## Definition

Split an autonomous agent into specialized sub-agents coordinated by
a manager. Canonical split: manager (arbitrates, routes, decides
when a task is done), designer (strategy and architecture), coder
(implementation and debugging), tuner (training and performance
optimization). Each sub-agent is itself an LLM-based agent with its
own tools and multi-step reasoning.

## Why it matters

AIBuildAI ([[literature/papers/zhang2026aibuildai]]) reached rank-1
on MLE-bench with a 63.1% medal rate using this exact four-role
split, outperforming both flat AutoML and monolithic LLM agents.
The split's value is not prompt engineering — it is context isolation:
each role operates within its own working memory, so the coder is
not distracted by architectural debates and the designer is not
distracted by debugger output.

Compared to a flat agent trying to do everything in one context
window, hierarchical delegation trades coordination overhead for
per-role focus. At MLE-bench scale the trade is strongly positive.

## Implementation guidance

1. **Role boundaries are sharp in charter, fuzzy in practice.**
   Each role's system prompt specifies what it owns and what it
   defers to the manager. In practice architecture choices depend
   on tuner feedback, so the manager mediates round-trips rather
   than pretending the dependency does not exist.

2. **Handoffs go through structured messages.** Sub-agents do not
   talk to each other directly — they return results to the manager,
   who decides whether to hand off, loop back, or finalize. The
   structured-message format is itself important:
   free-text handoffs drift; field-indexed handoffs do not.

3. **Per-role model choice pairs naturally.** The manager and
   designer benefit from strong models; the coder and tuner can
   often run on a cheaper implementer (see
   [[concepts/hybrid-model-backends]]).

4. **Manager context is the bottleneck.** The manager sees every
   sub-agent's output; it is the first thing that overflows the
   context window on long tasks. A shared
   [[concepts/structured-world-model]] that sub-agents write into
   (rather than returning full transcripts) keeps manager context
   from exploding.

## Open questions

- The four-role split is the one AIBuildAI validated. AiScientist
  ([[literature/papers/chen2026toward]]) validates a different,
  task-shape-driven split — Paper Comprehension / Prioritization /
  Implementation / Experimentation specialists — exposed to a
  Tier-0 Orchestrator via Agent-as-Tool. The convergence across
  two papers is on *hierarchy itself*, not on a specific role list;
  the right split likely depends on task shape.
- AiScientist's *Agent-as-Tool* design (specialists are callable
  from the Orchestrator's native tool space, so delegation is
  *selective* rather than mandatory) is a refinement worth noting:
  the Orchestrator can handle lightweight ops directly and only
  invoke a specialist when the expected benefit clears coordination
  cost. This is distinct from a fixed handoff pipeline.
- Whether three roles (collapse designer + coder) or five (split
  tuner into "trainer" and "hyperparameter-searcher") would work
  better is still unknown.
- Status is `experimental` because our current skills do not spawn
  sub-agent hierarchies — `/implement` uses one subagent, not a
  manager. A downstream project that reproduces AIBuildAI's or
  AiScientist's architecture would move this to `active`.

## What a parent owes a child: the conservation law

Most sources here describe delegation as a *role* decomposition — who
does what. [[literature/papers/ye2026agent]] supplies the missing
*resource* semantics: an orchestrator issues subcontracts constrained by
Σ R_i ≤ R_parent, an invariant that holds regardless of whether children
run sequentially, in parallel, hierarchically, or competitively.
Measured at zero violations across 50 trials of a three-agent
Researcher → Analyzer → Reporter pipeline.

The property it buys is **bounded autonomy**: an orchestrator may be
arbitrarily capable and may itself spawn contract-issuing children, yet
can never exceed the constraint it was handed. That is what makes
*recursive* delegation safe — the depth of the hierarchy is irrelevant
to the total spend, so "how deep may this nest?" stops being a safety
question and becomes purely a coordination-overhead question.

Two mechanisms make it practical rather than merely sound:

- **Allocation has three modes** — proportional to estimated complexity,
  equal when complexity is unknown, or *negotiated* (children request,
  the coordinator caps to prevent over-claiming). A 10–15% reserve
  buffer absorbs coordination overhead.
- **Unused budget returns to a shared pool** as children complete, so
  efficient siblings subsidize expensive ones without breaching the
  total. Without this, equal allocation wastes whatever the cheap
  children did not need.

The paper also reframes the orchestrator's job as **contracting as a
capability**: once a contract fully specifies a child's task, budget,
and success criteria, the parent need not select from a fixed pool — it
can *instantiate* a specialist to fit the contract. Routing and
orchestration collapse into one operation, with the contract as the
agent's specification rather than a filter over existing agents. That is
a materially different architecture from the fixed-role pipelines the
other sources here describe, and the closest thing in the cluster to a
principled answer to "how many roles?" — as many as there are
contracts worth writing.

See [[concepts/budget-as-ceiling]] for the enforcement limits that bound
all of this (a ceiling is really ceiling-plus-one-call).
