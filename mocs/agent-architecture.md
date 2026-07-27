---
kind: moc
name: "agent-architecture"
status: active
added: "2026-06-03"
concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [moc, agent-architecture, orchestration, long-horizon, architecture]
---

# Agent Architecture for Long-Horizon Autonomy

How an autonomous research agent is *built and run* over a long horizon —
the structural choices that decide whether it stays coherent across many
invocations or collapses under its own context. Where the sibling MoC
[[mocs/knowledge-organization-for-research-agents]] is about the knowledge
the agent works *with* (claims, sources, retrieval), this one is about the
machine that does the work: where state lives, how labor is divided, and
how the agent spends its scarcest resource — context budget.

Five concepts span the question across three layers. Two of them
(`agent-native-memory`, `file-as-bus`) also appear in the
knowledge-organization MoC — they are the seam where "how knowledge is
stored" and "how the agent is structured" meet, and they earn a place in
both views.

## Substrate: where durable state lives

The agent has to survive its own context window resetting. Both substrate
concepts answer "what is the system of record when the conversation isn't?"

- [[concepts/agent-native-memory]] — the same LLM that reasons also curates
  and retrieves its knowledge, as human-readable markdown it writes and
  reads directly (no external vector/graph DB). Memory operations are tools,
  not service calls. Evidence: ByteRover
  ([[literature/papers/nguyen2026byterover]]) eliminates three
  external-MAG failure modes by making memory agent-native.
- [[concepts/file-as-bus]] — the shared filesystem, not the conversational
  handoff, is authoritative across invocations. Specialists write durable,
  role-scoped artifacts and re-ground from them each call. Ablating it drops
  PaperBench by 6.4 points ([[literature/papers/chen2026toward]]).

## Orchestration: how the work is divided

Once state is durable, the question is who does what — and with which model.

- [[concepts/hierarchical-delegation]] — split the agent into specialized
  sub-agents (manager / designer / coder / tuner) under a coordinator. The
  win is **context isolation**, not prompt engineering: AIBuildAI
  ([[literature/papers/zhang2026aibuildai]]) hit rank-1 on MLE-bench (63.1%
  medal) with exactly this split. Context isolation is only half the
  story, though: delegation also divides *resources*, and
  [[literature/papers/ye2026agent]] supplies the invariant — subcontracts
  constrained by Σ R_i ≤ R_parent, holding across sequential, parallel,
  hierarchical, and competitive execution (zero violations, n=50). The
  property it buys, **bounded autonomy**, is what makes recursive
  delegation safe: a child may itself spawn contract-issuing children and
  still cannot exceed the constraint its parent was handed, so nesting
  depth stops being a safety question. See
  [[concepts/budget-as-ceiling]] in
  [[mocs/autonomous-search-loop]] for the enforcement limits that bound
  this — a ceiling is really ceiling-plus-one-call, because token spend is
  unknown until a call returns.
- [[concepts/hybrid-model-backends]] — let each role pick its own model
  family; a cheaper implementer often matches a frontier ideator's quality
  on mechanical sub-tasks at a fraction of the cost. Directly composes with
  the delegation roles above (coder/tuner ≠ designer in required capability).

## Execution: spending context efficiently

Even with good roles and durable state, a long task drowns in tool-call
round-trips. This layer is the runtime discipline that keeps it cheap.

- [[concepts/scripted-tool-pipelines]] — chain multi-step tool use as a
  short *script* through a programmatic interface, so only the script and
  its final return re-enter the conversation; intermediate results stay in
  local scope. Reclaims the context budget that frontier agents otherwise
  burn on round-tripping.

## Open thread

The three layers compose, and — as in the knowledge-organization MoC — the
bottleneck shifts to whichever is weakest. A system with agent-native memory
and role delegation but naive per-turn tool use will still suffocate on
context; one with scripted pipelines but no durable bus will lose state
across invocations. The working hypothesis: long-horizon autonomy needs all
three layers at once, and most published systems optimize one or two and
inherit the third's ceiling. The `hybrid-model-backends` × `hierarchical-
delegation` pairing (this project's own ideator/implementer split) is the
most directly testable interaction.
