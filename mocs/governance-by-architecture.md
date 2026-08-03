---
kind: moc
name: "governance-by-architecture"
status: active
added: "2026-08-03"
concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/verified-memory-writes]]"
tags: [moc, governance, enforcement, policy, determinism, safety, architecture]
---

# Governance by Architecture

When an autonomous agent must obey a constraint — a permission, a
budget, a standing policy, a memory-integrity rule — what actually
guarantees compliance? The five concepts here converge on one answer
from five directions: **the constraint is enforced by deterministic
code at a structural site in the harness, never by prose the model is
asked to honor.** Prose policy admits no enforcement semantics and
degrades exactly when optimization pressure is highest; the measured
failure rates are now known — in-context policy goes 0%→30% violation
after one compaction pass ([[literature/papers/chen2026governance]]),
soft budget penalties breach the ceiling on 91.5% of seeds
([[literature/papers/besanson2026green]]), prompt-encoded campaign
rules never reach zero failure across thousands of invocations
([[literature/papers/philippov2026glite]]), and fewer than 1% of
6,145 real agent configs declare any permission boundary at all
([[literature/papers/madatha2026deterministic]]).

Where the sibling [[mocs/capability-layer]] asks how the agent's
action surface is *built* and [[mocs/evaluation-integrity]] asks
whether the numbers coming out can be *trusted*, this MoC asks what
makes a stated constraint *binding*. Two members appear in
capability-layer, one each in autonomous-search-loop and
knowledge-organization — this view collects them by their shared
enforcement logic rather than their home subsystem.

## The policy artifact — what the constraint is written in

- [[concepts/typed-enforcement]] — the root of the cluster: policy as
  a machine-checkable artifact (Datalog, affine types, Cedar, intent
  manifests) with decidable static analyses, enforced by a
  deterministic checker outside the agent's reasoning. Anchored by
  FORGE's conditional correctness theorem
  ([[literature/papers/palumbo2026formal]]) and the negative theorem
  on what a policy may bind to
  ([[literature/papers/louck2026securing]]): authority signals
  derived from in-stream content are malleable; only non-malleable
  origin binding is sound.

## The gates — where the constraint meets the action

- [[concepts/permission-gate-as-architecture]] — the approval gate as
  a stateful regulator inside the control loop (risk cumulants,
  typed tool registries, escalation cascades), not a yes/no wrapper
  in front of it.
- [[concepts/budget-as-ceiling]] — resource constraints as hard
  ceilings checked between actions, with the achievable pre-flight
  remainder now mapped: exact reservation is impossible
  (ceiling-plus-one-call, [[literature/papers/ye2026agent]]), but a
  calibrated cost forecast turns the overshoot into a bounded risk
  ([[literature/papers/besanson2026green]]).

## The state — keeping constraint and record intact over time

- [[concepts/constraint-pinning]] — enforcement of constraint
  *presence*: governance that lives in evictable context is silently
  deleted by compaction, so the harness quarantines it from
  compression and re-injects it verbatim. The newest member, and the
  reason this cluster tipped ripe.
- [[concepts/verified-memory-writes]] — enforcement on the durable
  store's *write path*: coverage / preservation / faithfulness
  checked at consolidation time, because a flawed write compounds
  across every later retrieval.

## The shared honest limit

Every member hits the same wall at the top of the stack: **authority
cannot be authenticated from inside the token stream.** Constraint
pinning is defeated by operator impersonation in recent context;
in-repo verifiers can be rewritten by the agent they govern
([[literature/papers/philippov2026glite]], stated plainly); a typed
policy over attacker-influenceable atoms is formal but unsound
(louck2026securing). The open thread is the trusted out-of-band
channel — authority bound to origin, delivered outside the
agent-writable universe — and, until that exists, the design rule is
to keep the enforcement code and its write path outside the agent's
scope. A second open thread: every deterministic gate has a semantic
escape hatch (the constraint that cannot be stated as a checkable
predicate); mapping which constraints are checkable and which
irreducibly need model or human judgment is where this cluster is
still growing.
