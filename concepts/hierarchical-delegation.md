---
kind: concept
name: "hierarchical-delegation"
status: experimental
added: "2026-04-24"
source_papers:
  - zhang2026aibuildai
sources:
  - "[[literature/papers/zhang2026aibuildai]]"
used_by: []
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/structured-world-model]]"
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

- The four-role split is the one AIBuildAI validated. Whether
  three roles (collapse designer + coder) or five (split tuner
  into "trainer" and "hyperparameter-searcher") would work better
  is unknown.
- Status is `experimental` because our current skills do not spawn
  sub-agent hierarchies — `/implement` uses one subagent, not a
  manager. A downstream project that reproduces AIBuildAI's
  architecture would move this to `active`.
