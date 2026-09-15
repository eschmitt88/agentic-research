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
  - "[[literature/papers/kim2026why]]"
  - "[[literature/papers/yoon2026arcticswarm]]"
  - "[[literature/papers/piriyakulkij2026subagents]]"
  - "[[literature/papers/gao2026agentic]]"
  - "[[literature/papers/bouras2026authority]]"
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

## A hierarchy over knowledge scope, coupled to the agent hierarchy

[[literature/papers/kim2026why]] is an instance where the hierarchy is not
over task decomposition but over **what each level is allowed to know** —
and the two are deliberately made to coincide. An orchestrator assigns
competitions to domain specialists (tabular / NLP / vision), and each
specialist loads only the skill tier matching its scope: global skills for
everyone, domain skills for the matching specialist only, task-specific
skills only on a re-run of that task.

The consequence is that an agent *structurally cannot see* out-of-scope
knowledge — the delegation boundary and the information boundary are the
same boundary. That is the same move [[concepts/permission-gate-as-architecture]]
makes for capability, applied to context.

The measured payoff is large and comes from a controlled ablation with the
skill inventory held fixed: scope-matched loading medals 8/8 where flat
loading of the identical 159 skills medals 5/8 — the same as loading
nothing — at 2× the tokens. So the hierarchy is not organizational tidiness;
it is where the benefit lives.

The orchestrator's own role is worth noting as a delegation pattern: it
does classification, scheduling, and **promotion** (deciding which of a
specialist's learnings rise to domain or global scope). Promotion is
upward information flow across the delegation boundary, which most
hierarchical-agent designs in this cluster lack — subagents report results,
not generalizations. The paper concedes the promotion step is itself
unevaluated.

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

The same invariant has an authority form, with one difference.
[[literature/papers/bouras2026authority]] derives each sub-agent's
capability store as a subset of a task ceiling frozen before any untrusted
input is read. Authority is **attenuated but not conserved**: siblings may
hold overlapping capabilities, and nothing is spent. This gives delegation
a second payoff besides context isolation. With the topology fixed, giving
the runner that reads poisoned test output no write capability takes
injected effects from 33/75 (one shared task allowlist) to 3/75, at no
repair cost. It also gives delegation a new failure mode. In 2 of 75 runs
the runner saw the correct fix, was refused the write, and the workflow
never rerouted the edit to the write-authorized patcher. Once the role
boundary is an authority boundary, handoff routing becomes load-bearing
for completion. See [[concepts/permission-gate-as-architecture]].

## Isolation pays only across an interface, and mostly under pressure

The rationale above — the split's value "is context isolation" — has so
far rested on system-level wins where role structure, prompts and tooling
all changed together. [[literature/papers/piriyakulkij2026subagents]] is
the first controlled test. It keeps the skill package and base model fixed
and varies only whether the skill runs **inline** (its `SKILL.md` loaded
into the main context) or as a **subagent** (fresh context seeded with the
skill body, returning only its final message). On SkillsBench across 7
models the answer is conditional in three ways. (The paper reports
accuracies only as bar charts; values below are read from its Figs. 2, 5
and 6 and are approximate, ≈±0.01.)

- **It needs a contract.** With curated skills that lack stated inputs and
  outputs, the two modes are within a few points of each other, and inline
  clearly wins for the stronger models (gpt-5.3-codex ≈0.71 vs ≈0.61).
  With skills whose descriptions carry "Expected input:" and "Output:"
  sections, subagents roughly double accuracy for weaker models
  (Qwen3.5-9B ≈0.22 → ≈0.46, Mistral-Large ≈0.26 → ≈0.54). This is
  ye2026agent's "the contract is the specification" argument with a
  measurement attached. Caveat: the contract sets also differ in content,
  and no ablation separates the two.
- **It needs context pressure.** Those gains appear to come from the
  condition where all 263 unrelated skills are exposed. With only the
  task's own skills, the subagent edge is ≈+0.02 to +0.03 for gpt-5.4-mini
  and Mistral-Large, and ≈−0.02 for gpt-5.3-codex. Frontier models get no
  accuracy benefit at all (codex ≈0.85 vs ≈0.85), yet still pay roughly
  **1.3-1.7× total tokens** (also read from a chart). What they do get is
  a lower peak context, on 95.3% of codex tasks.
- **Routers must not be delegated.** In a hierarchical library (Qwen3.5-9B
  only), running every node as a subagent collapses accuracy to ≈3-7%.
  Running routing nodes inline and leaf skills as subagents is best
  (≈0.64). A node with no procedure and no contract has nothing to
  delegate.

This sharpens guidance 2 and the Agent-as-Tool open question: delegation
should be selective, and the selection criterion is whether the unit has a
stated input/output interface, not whether it is a distinct role. It also
bounds the manager-context claim in guidance 4: isolation lowers *peak*
context but raises *total* spend, because each child must be re-supplied
context the parent already holds.
