---
kind: paper
title: "Lingering Authority: Revocable Resource-and-Effect Capabilities for Coding Agents"
authors: ["Igor Santos-Grueiro"]
institutions: ["International University of La Rioja"]
year: 2026
venue: arXiv (cs.CR)
peer_reviewed: false
url: https://arxiv.org/abs/2606.22504
code_url: https://anonymous.4open.science/r/portico/
citations: null
source: "raw/papers/santosgrueiro2026lingering.pdf"
added: "2026-07-13"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
tags: [safety, permission-gate, capabilities, revocation, coding-agents, governance]
---

# Lingering Authority: Revocable Resource-and-Effect Capabilities for Coding Agents

## TL;DR

Names the failure mode **lingering authority** — a temporary capability
that remains exposed to the agent's planner after the subgoal that
justified it has closed — and builds PORTICO, a reference monitor that
gives permissions a *lifetime*: task contracts compile to an initial
capability envelope plus grant rules and closure predicates; grants mint
epoch-bound opaque handles; closure removes them from the next planner
interface and rejects stale replay before side effects.

## Claims

- Existing defenses (sandboxing, allowlists, execution-time checks)
  answer *whether* an action is admissible, but leave open *when*
  temporary authority should disappear. A broad option left visible in
  the planner interface is itself security state — the model keeps
  planning around it even if a monitor would reject the call.
- Three obligations formalize the fix: **exposure minimality** (only
  currently-justified capabilities visible), **justified expansion**
  (boundary work enters only through explicit grants matched by rule),
  and **authority closure** (a grant's authority disappears at the first
  planner turn after its closure event; replay of the old handle is
  denied before effects).
- Closure is the load-bearing difference: a non-revoking comparator with
  the *same* envelope, grants, and execution policy matches PORTICO on
  task success, scope compliance, and all pre-closure decisions — then
  permits 10/10 post-closure reuses and executes 6/6 forbidden stale
  writes where PORTICO permits/executes 0.
- Trust provenance matters: closure events are trusted runtime signals
  (focused test passes, orchestrator subgoal transitions, authenticated
  revocation) — a planner-authored "subgoal_complete" *claim* is
  untrusted text and cannot close or extend a grant.

## Methods

- Task contract (YAML manifest: initial allowed resources, declared
  escalation paths with triggers, compiled lifetime predicates, forbidden
  resources/hosts) compiled to policy P = (initial envelope E₀, grant
  rules Γ, closure predicates Λ, global denies G).
- Request–grant–invoke lifecycle: planner requests authority; a matching
  grant rule mints epoch-bound opaque handles for the covered
  resource+effect; later calls execute only through live handles;
  closure consumes an observable trusted event and removes the handles
  before the next planner turn. Handles are task-local server-side
  references validated against task, epoch, resource, privilege, effect.
- Shell/git/package/network calls mediated through a typed catalog
  mapping each invocation to an authority descriptor (tool, privilege,
  resource, side-effect class, intent, phase); unclassifiable commands
  require explicit approval or are denied.
- Evaluation: three controlled fixture suites (exposure lures, legitimate
  boundary grants, hostile context) + fourteen pinned real-repository
  contracts (Flask, Requests, Click, Jinja, ...). Main model
  Qwen3-Coder-30B; frontier slice GPT-5.5, Gemini 3.5 Flash, Claude
  Opus 4.8. Comparators isolate each mechanism: full access, static
  allowlist, coarse sandbox, PORTICO-core (envelope only), task-wide
  non-revoking, all-visible same-policy.
- Artifact: anonymized repo with checked-in contracts, frozen traces,
  reproduction scripts, and `portico-tool` — a standalone MCP server
  exposing the request–grant–invoke lifecycle to MCP-capable agents.

## Results

- Suite A (exposure lures): full access violates every run (avg blast
  radius 6.65); static allowlist and sandbox still violate at 0.82/0.86;
  both PORTICO variants: 0.00 violations, 0.00 blast radius across 0/17
  scenarios and 0/85 repeats.
- Suite B (legitimate boundary work): envelope-only is safe but brittle
  (0.21 success); grants recover it to 0.87 with 1.00 scope compliance
  and zero violations. Measurable utility cost remains (161/185
  completions vs 180/185 for baselines).
- Closure slice: PORTICO denies 10/10 late rereads, non-revoking allows
  10/10; deterministic stale-write audit 0/6 vs 6/6 executed forbidden
  effects; same split replicated in scripted and model-live traces across
  file writes, git mutation, and network egress.
- Visibility has planning cost even under identical execution policy:
  the all-visible comparator generates 84 blocked proposals vs
  PORTICO's 67 — broad visible options produce pressure toward
  forbidden detours.
- Real repositories: all six conditions complete 14/14 contracts; only
  PORTICO-full materializes temporary grants on all 14, with 0
  executed contract-forbidden effects.

## Critique / open questions

- Single-author preprint from a lesser-known institution; evaluation is
  on self-constructed fixtures with self-defined metrics (though the
  comparator design — same policy, only revocation removed — is unusually
  clean causal isolation).
- Guarantees assume a linearizable mediation point and serialized planner
  requests; concurrent multi-agent integrations must revalidate handles
  immediately before side effects, which the paper acknowledges but does
  not evaluate.
- Contract authoring is the soft underbelly: undergranting blocks
  legitimate work, overgranting re-inflates exposure, and the monitor
  inherits wrong expansions. Who writes good contracts for open-ended
  research tasks is unanswered — a research agent's subgoals are less
  predictable than "propagate a timeout parameter".
- Utility cost is real (Suite B 161/185) and would compound over long
  autonomous research runs; the allow/verify/refuse operating curve is
  reported at one point.

## Trust signals

- **Credibility:** 3 — single-author arXiv preprint, institution not a
  known agents-security group, no citations yet; but a full anonymized
  artifact is released (contracts, traces, reproduction scripts, MCP
  server), the comparator methodology is rigorous, and results replicate
  across scripted, model-live, and frontier-model slices.

## Follow-up

- **Relevance:** 4 — adds the missing *temporal* dimension to
  [[concepts/permission-gate-as-architecture]]: existing sources govern
  whether an action passes the gate; this one governs *how long the
  permission itself lives*. First source in the cluster with a released
  runnable artifact (MCP server), directly relevant to Claude Code-style
  harness design.
- The "visible option = planning pressure" finding (84 vs 67 blocked
  proposals under identical execution policy) is independently
  interesting: interface exposure shapes model behavior even when
  enforcement is constant.
- Watch for de-anonymization/venue acceptance; try `portico-tool`
  against a Claude Code PreToolUse-hook setup if the permission-gate
  concept graduates to a system proposal.
