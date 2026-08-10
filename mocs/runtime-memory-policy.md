---
kind: moc
name: "runtime-memory-policy"
status: active
added: "2026-08-10"
concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/agent-native-memory]]"
tags: [moc, memory, runtime-policy, context-window, eviction, retrieval, agent-memory]
---

# Runtime Memory Policy

While the agent is *reasoning*, a second set of decisions runs
alongside the task: what stays in the context window, what leaves and
under what guarantee, what gets pulled back in, and what is allowed to
persist. This MoC collects the seven concepts that govern that
inference-time traffic. It is a split from
[[mocs/knowledge-organization-for-research-agents]], whose center of
gravity is *compile-time* curation — how a knowledge library is
structured, evolved, and kept honest between sessions. The sub-theme
outgrew the parent when the runtime side reached seven concepts with
its own coherent question and two members
([[concepts/constraint-pinning]], [[concepts/lossless-context-offload]])
that never belonged to the parent at all. Five concepts remain in
both; this view collects them by *when the policy runs* (mid-episode,
under token pressure, per decision) rather than by where knowledge
lives. Two members also appear in [[mocs/governance-by-architecture]]
— the runtime memory layer is one of that MoC's enforcement surfaces.

## The window — what stays hot

- [[concepts/context-eviction-policy]] — the load-bearing axis: what
  leaves the window when the working set would overflow, decided by
  what a *future* turn will depend on, not by position or type. Its
  central open question is now a three-way policy-locus split —
  deterministic LLM-free eviction
  ([[literature/papers/semenov2026beyond]]), planner-proposed under
  harness validation ([[literature/papers/hao2026selfgc]]), and
  trained agent-initiated management
  ([[literature/papers/li2026acm]]).
- [[concepts/constraint-pinning]] — the exemption list: standing
  policies are quarantined from eviction and re-injected verbatim
  after every compression, because a summarizer trusted with policy
  survival measurably drops it (0%→30% violation after one compaction
  pass, [[literature/papers/chen2026governance]]).
- [[concepts/lossless-context-offload]] — the invariant on leaving:
  eviction is an offload to a stable address, never a deletion, so
  the summary left behind can be traded back for the raw span on
  demand. The one claim all three eviction-locus camps agree on.

## The flux — reads and writes across the hot/cold boundary

- [[concepts/selective-memory-retrieval]] — reading is a runtime
  decision gated on the agent's own reasoning state (uncertainty,
  staleness, continuation), not a schedule; always-on injection
  measurably underperforms
  ([[literature/papers/zhao2026expweaver]]).
- [[concepts/verified-memory-writes]] — persisting is a verified
  state transition (coverage / preservation / faithfulness) enforced
  at consolidation time, because retrieval-time filtering cannot see
  compositional corruption
  ([[literature/papers/gao2026mempoison]]).

## The store — what the runtime policies operate against

- [[concepts/multi-granularity-memory]] — the cold side keeps the
  same experience at several grains and routes each query to the
  grain that fits, so eviction has somewhere faithful to spill and
  retrieval has something precise to target.
- [[concepts/agent-native-memory]] — the substrate discipline:
  memory operations as tools over human-readable, provenance-carrying
  artifacts the reasoning agent itself curates — which is what makes
  every policy above auditable after the fact.

## Open thread

The unresolved question that spans the whole MoC is **who runs the
policy**: harness code, a side-channel planner, or the trained agent.
The eviction cluster now has measured evidence in all three
directions, including the first negative result — enforcement placed
too aggressively regresses strong models
([[literature/papers/elkoussy2026agentltl]]) — and the
cache-economics constraint that eager eviction can cost more than it
saves ([[literature/papers/hao2026selfgc]]). A likely convergence,
already visible in Self-GC and ACM, is the division of labor: the
model supplies semantic judgment about future value; the harness owns
recoverability, protected content, and commit timing. Whether that
split holds for the read and write sides too — trained gates over
harness-verified stores — is the natural next thing this cluster's
sources should be watched for.
