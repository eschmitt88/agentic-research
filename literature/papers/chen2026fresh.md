---
kind: paper
title: "Fresh Memory, Stale Plans: Dependency-Scoped Validation for Distributed LLM-Agent Memory"
authors: ["Evan Chen", "Shiqiang Wang", "Christopher G. Brinton"]
institutions: ["Purdue University", "University of Exeter"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.03340"
code_url: null
citations: null
source: "raw/papers/chen2026fresh.pdf"
added: "2026-09-08"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["memory", "staleness", "provenance", "multi-agent", "enforcement", "validation"]
---

# Fresh Memory, Stale Plans: Dependency-Scoped Validation for Distributed LLM-Agent Memory

## TL;DR

**State freshness does not establish that the plan authorizing an action is
still valid.** An executor can hold the newest shared facts and still act on
a plan derived from a superseded requirement. PlanFence makes plans **cite
the exact records they used** and validates only those records before an
external action — closing a failure that a freshness check cannot see.

## Claims

- *Stale-plan execution* is a distinct failure class: the planner derives an
  action from record `r3`, another agent commits `r4`, the executor receives
  `r4` but never replaces the plan built on `r3`. Every individual component
  is up to date; the composite is not.
- Validation should be **dependency-scoped** — check the records that can
  affect *this* pending action, not global state. Scoping is what makes the
  check affordable as the shared keyspace grows.
- Incomplete validation must **block**, not proceed. The gate sits
  immediately before the external effect.

## Methods

- PlanFence: plans cite the exact public records used; the executor
  validates only records that can affect the pending external action, then
  replans once or blocks if validation is incomplete.
- 30 controlled live workflows, each containing a post-plan revision.
- Controlled replay to map the boundary against proactive synchronization
  across churn rates and keyspace sizes.

## Results

- A **freshness-only executor acts on the obsolete plan in every one of the
  30 tasks**; PlanFence completes all tasks with no invalid action.
- Two conditional boundaries: proactive sync gives lower coordination stall
  at **low churn**; PlanFence wins **as churn grows** (avoids repeated
  update-path coordination) and **as the keyspace grows** (avoids validating
  unrelated state).
- The authors are explicit these are **controlled safety and systems-cost
  results, not general task-accuracy gains**.

## Critique / open questions

- 30 workflows, all constructed to contain a post-plan revision — a 30/30
  baseline failure rate is a property of the scenario design, not a measured
  incidence in the wild. The paper is honest about this; it establishes the
  mechanism, not its frequency.
- Requires plans to cite their records, which presumes a planner that emits
  structured dependencies. Retrofitting that onto a free-text planner is the
  unaddressed engineering cost.
- No released code.

## Trust signals

- **Credibility:** 3 — arXiv preprint, not peer reviewed, no citations, no
  artifact. Solid academic group (Purdue / Exeter; Brinton is established in
  distributed learning). Credit for scoping the claim tightly and explicitly
  disclaiming accuracy gains rather than overselling — the failure mode
  [[literature/papers/brueckner2026kbench]] finds on 31.4% of agent output.

## Follow-up

- **Relevance:** 4 — supplies the **temporal axis** that
  [[concepts/enforcement-boundary-placement]] gained this cycle, in its
  cleanest form: a check valid at authorization is stale at consumption, so
  placement has a *when* as well as a *where*. This is the same shape as
  [[literature/papers/ding2026acle]]'s execution-time leases, arrived at
  from the memory side rather than the capability side.
- Directly relevant to this project's `/lint` staleness check, which
  approximates staleness by file age. Dependency-scoped validation is the
  principled version: a concept is stale when **a source it cites has
  changed**, not when it is merely old. That is implementable here —
  concept notes already cite their sources.
- "Validate only what can affect the pending action" is the affordability
  argument [[concepts/permission-gate-as-architecture]] needs; blanket
  revalidation is what makes gates get skipped.
- Pairs with [[literature/papers/hu2026memory]]: hu shows a stale *fact*
  beats live evidence, chen shows a stale *plan* survives a live fact.
