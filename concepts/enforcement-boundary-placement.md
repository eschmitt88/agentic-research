---
kind: concept
name: "enforcement-boundary-placement"
status: growing
added: "2026-09-01"
sources:
  - "[[literature/papers/leong2026recognition]]"
  - "[[literature/papers/zhan2026auto]]"
  - "[[literature/papers/guo2026when]]"
  - "[[literature/papers/rahman2026framing]]"
  - "[[literature/papers/song2026string]]"
  - "[[literature/papers/chi2026ai4ai]]"
  - "[[literature/papers/esakkiraja2026starharness]]"
  - "[[literature/papers/marsden2026where]]"
  - "[[literature/papers/zheng2026continuity]]"
  - "[[literature/papers/ding2026acle]]"
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/yang2026sok]]"
  - "[[literature/papers/chen2026fresh]]"
  - "[[literature/papers/zheng2026engineering]]"
  - "[[literature/papers/shen2026revoked]]"
  - "[[literature/papers/kapner2026scanning]]"
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
related_experiments: []
tags: ["safety", "governance", "agent-architecture", "evaluation"]
---

# enforcement-boundary-placement

## Definition

**Where** an agent's enforcement boundary physically sits — which artifact
holds the constraint and which component evaluates it — treated as an
independent design axis from *what form* the constraint takes and *what
role* it plays in the control loop.

The axis runs from placements inside the agent's own reasoning (a system
prompt) outward: into the artifact being invoked, into the runtime between
induction and execution, at the data's destination, at the provenance
origin of the instruction, into the resource being called, and finally
entirely outside the agent's workspace. The consensus across sources is
negative and strong — enforcement cannot live in the model — while the
positive answers disagree, and it is that disagreement that makes
placement a separate question worth naming.

The axis has a **temporal** component as well as a spatial one.
[[literature/papers/ding2026acle]] is the first source to make this the
whole point: a check that was valid when the connection was authorized can
be stale by the time the authority is consumed, so *when* the boundary is
evaluated is as much a placement decision as *where* it sits.

Since 2026-09-07 the axis is also **empirically testable** rather than only
arguable. [[literature/papers/marsden2026where]] localises a property to a
layer by intervening on one layer while holding the other fixed, and
reports five behavioural properties that survive ablating, resetting,
wholly replacing and poisoning the cognition. That converts "where does
this property live" from an architecture-diagram claim into a measurement,
and it is the methodological anchor this concept previously lacked.

## Why it matters here

Two existing concepts each cover a different dimension of the same
mechanism and neither covers this one:

- [[concepts/typed-enforcement]] covers the **form**: the constraint is a
  machine-checkable artifact in a language with decidable analyses, not
  prose. It is silent on where that artifact lives.
- [[concepts/permission-gate-as-architecture]] covers the **role**: the
  gate is a stateful regulator over the agent's policy rather than a
  bolt-on filter. It is silent on what the gate is attached to.

A design can satisfy both and still be broken by placement. A typed policy
evaluated by a stateful regulator that reads its inputs from inside the
agent's context window inherits every channel-forgery weakness
[[literature/papers/leong2026recognition]] documents, because the thing
being checked is reachable by the thing being constrained. Placement is
what decides whether the boundary is a *security boundary* or, in that
paper's phrase, merely a *capability*.

[[concepts/information-firewall]] is best understood as the **evaluation
special case** of this axis — placing the boundary such that the oracle is
unreachable from the agent's workspace — which is why
[[literature/papers/chi2026ai4ai]] arrives at an identical structure from a
measurement motivation rather than a security one.

## The attested placements

Eleven sources, all agreeing enforcement leaves the model, disagreeing on
where it goes:

| Placement | Source | Mechanism |
|---|---|---|
| Outside the model entirely | [[literature/papers/leong2026recognition]] | External reference monitor: authenticated source routing + capability-gated execution |
| Inside the invoked artifact | [[literature/papers/zhan2026auto]] | Typed invocation policy co-packaged into the Skill itself |
| In the runtime, between induction and authorization | [[literature/papers/guo2026when]] | Provenance recorded on the Observation side; authorization decided separately against the user objective |
| At the data's destination | [[literature/papers/rahman2026framing]] | Destination allow-list, or a capability-isolating planner/reader split |
| At the provenance origin | [[literature/papers/song2026string]] | Privilege follows topic type in code: remote documents may call HTTP but never the shell |
| Outside the workspace | [[literature/papers/chi2026ai4ai]] | Evaluator fixed before the first run, no access to the agent's workspace |
| At the change's scope check | [[literature/papers/esakkiraja2026starharness]] | Validator checks a candidate git diff for scope and imports; ground-truth tables and hidden-state access forbidden |
| Beneath cognition, in the institution | [[literature/papers/marsden2026where]] | Append-only ledger adjudicates every attempted act against world state; five properties survive full cognition substitution |
| Distributed across *every* component interface | [[literature/papers/zheng2026continuity]] | Assume-guarantee contracts; each transition emits a verifiable witness, so there is no single site to drop a label at |
| Resource-side, at execution time | [[literature/papers/ding2026acle]] | Provider-side Execution Gate consumes a short-lived sender-constrained lease immediately before protected tool logic runs |
| Nowhere — the negative case | [[literature/papers/paglieri2026case]] | A system prompt forbade cheating and was "not actively enforced beyond the autograder check"; 34 of 71 conjectures fell in 27 minutes |

Two properties recur across the successful placements and are the
candidate design rules:

1. **The check is payload-blind.** [[literature/papers/rahman2026framing]]
   is the sharpest evidence: ten overt exfiltration attempts are refused,
   the same leak reframed as an integrity signature succeeds 100% of the
   time, and both structural defenses that ignore the payload entirely
   score 0%. Any placement whose check must *understand* the request
   inherits the reframing attack.
2. **The constrained component cannot reach the constraining one.** This is
   the property [[concepts/information-firewall]] states for evaluation and
   that [[literature/papers/chi2026ai4ai]] and
   [[literature/papers/esakkiraja2026starharness]] both implement
   incidentally — the evaluator is fixed before the run, the diff cannot
   touch hidden state. [[literature/papers/marsden2026where]] supplies the
   direct test: substitute the entire cognition with a frozen frontier-LLM
   panel and see whether the property moves. Over 2,581 completion claims,
   none false.
3. **The boundary must be evaluated at the moment authority is used, not
   when it is granted.** [[literature/papers/ding2026acle]]'s
   post-authorization execution trust gap, and
   [[literature/papers/zheng2026continuity]]'s replayed-permit and
   widened-delegation fault classes, are the same observation from two
   directions. A placement that is spatially correct and temporally early
   is still bypassable. [[literature/papers/zheng2026engineering]]
   sharpens "at the moment authority is used" into **"inside the
   transaction that applies the effect"** and backs it with a controlled
   live measurement. In a forced after-check race (36 episodes, HTTP
   service over SQLite, separate writer process), every verifier-only
   policy commits the same 12 unsafe effects. The same partial guard run
   as a non-atomic preflight just before the write commits 16; moved
   inside the write transaction it commits 9 — exactly the clauses it
   does not express. The full-predicate atomic guard commits 0 across 216
   episodes (95% upper bound 1.4%) at 81.0% safe completion. A check
   immediately before the effect is still early, and a guard in the right
   place still covers only the clauses it can express. This is a
   long-established database principle; the new part is head-to-head
   evidence that verification portfolios do not substitute for it.

## Why this repository has the question and not the answer

This project runs three de facto trust domains distinguished only by prose
convention:

- `raw/` — immutable after ingest, per `.claude/rules/data.md`.
- `literature/`, `concepts/`, `mocs/` — agent-written knowledge.
- `~/.claude/skills/` — executable, and shared across every project on the
  box.

Nothing enforces the boundaries between them. `raw/`'s immutability is a
rule the agent is asked to honor, which is exactly the form
[[concepts/typed-enforcement]] identifies as admitting no enforcement
semantics. [[literature/papers/song2026string]] enforces the analogous
distinction in code — privilege derived from where a document came from —
and [[literature/papers/wu2026evomal]] demonstrates why the executable
domain is the one that matters: a shared skill namespace with
imitation-based authoring is a self-propagating medium, and this project
has a shared skill namespace.

## Open questions

- **Is there a placement that dominates, or is the right answer always
  several?** [[literature/papers/rahman2026framing]] reports two
  independent placements both reaching 0%, which suggests redundancy is
  cheap here. **Two head-to-head comparisons now exist (2026-09-14)**,
  each for one threat, from independent groups:
  - [[literature/papers/zheng2026engineering]] compares pre-action model
    verification, a non-atomic preflight guard, an in-transaction partial
    guard, a full atomic guard and deferral, all on identical proposals
    with preregistered comparisons. When the whole predicate is checkable
    at commit, the full atomic guard dominates and extra model checks add
    no observed safety. When the guard is partial, a cost-aware portfolio
    buys availability (+0.27 safe completion) but not lower risk (Holm
    p=0.51). Most live-arm differences miss significance with only 12
    contracts, so the dominance result rests on the zero, not on a
    ranking.
  - [[literature/papers/shen2026revoked]] runs the same scenarios, models
    and trials under placements along one memory pipeline, against a
    benign stale-memory threat (a soft-revoked policy still returned by
    retrieval). *Before the read*, a store-level status filter scores
    0/1,620 unsafe actions, and the authors' guard matches it (not
    independent evidence: both read the same status field). *After the
    read, in context*, prompt hardening scores 37.2%, against 43.1% with
    no defense. *At output*, a filter scores 18.1% even when given the
    revoked texts. *At the tool*, a shell guardrail leaves dangerous-tool
    invocation at 17.2%, the same as no defense, because the revoked
    policy licenses actions the guardrail does not classify as dangerous.

  The two agree on direction, and it is design rule 3: the check that sits
  where the record or authority is *consumed* — the read filter for
  memory, the write transaction for effects — wins. Both also bound it the
  same way. The winning placement holds only if it is applied at *every*
  use: an unfiltered reviewer role reintroduces shen's failure, and a
  preflight one step before the write lets zheng's race through. And
  anything that moves state past the check defeats it: agent write-back
  takes shen's filtered unsafe rate to 71.6-83.1% at later hops (direct
  write-back mode) — zheng2026continuity's dropped-label class in a memory
  store — just as the after-check race defeats every earlier placement in
  zheng2026engineering.

  Before these, every paper argued for its own placement, and the question
  was partly answered from the other side:
  [[literature/papers/zheng2026continuity]] argues that
  single-site placement is what *produces* the failure class, since a
  composed stack of individually-correct controls fails when any one
  boundary drops a label (65.6% harmful-effect rate for the strongest
  incomplete configuration). If that holds, "several" is not redundancy
  but the minimum viable answer.
- **What does placement cost?** Now partly answered:
  [[literature/papers/ding2026acle]] reports +25.7% p95 latency over
  OAuth-only, and [[literature/papers/zheng2026continuity]] reports 4.21 ms
  median proof verification against a 1.5-KLOC trusted core. These are the
  first numbers attached to the axis; both are from the systems' own
  authors and neither prices the operational tax (key management, manifest
  signing, component identity provisioning). Each attested placement adds a
  component.
  The 08-30 `/elevate` cycle held `context-proprioception` specifically
  because no viable form avoided adding surface area, and the same
  objection applies to most of these. The one placement that adds nothing —
  deriving privilege from a location that already exists — is
  [[literature/papers/song2026string]]'s, which is the reason it is the
  most interesting for this repository.
- **Does placement survive the artifact being agent-authored?**
  [[literature/papers/zhan2026auto]] puts the policy inside the Skill; if
  the Skill is written by the agent, the policy is too. This is the
  unresolved tension between placement-in-the-artifact and
  [[concepts/verified-memory-writes]]. The same tension has a measured,
  non-agent form: [[literature/papers/kapner2026scanning]] finds that a
  placement inside the artifact carries *grants* as easily as
  *constraints*. 3.7% of published skill collections ship a skill whose
  `allowed-tools` pre-approves the shell, and that pre-approval moves with
  the artifact into every repository that installs it. So the question is
  not only whether the artifact is agent-authored, but whether a boundary
  co-packaged with a third-party artifact can ever widen authority.
  zhan2026auto's placement is safe only if the packaged policy can
  restrict and never grant.

## Connections

- Depends on [[concepts/typed-enforcement]] (form) and
  [[concepts/permission-gate-as-architecture]] (role); this concept is the
  third dimension, not a restatement of either.
- [[concepts/information-firewall]] is the evaluation-side instance.
- [[concepts/file-as-bus]] becomes load-bearing if privilege is derived
  from file location — the substrate stops being merely a coordination
  convenience and becomes the policy carrier.
- Bears on [[concepts/hce-evaluation]]: an evaluator the agent cannot reach
  is a placement decision, and
  [[literature/papers/moukpe2026deltaml]] measures what happens without one
  (specification gaming up to 47.9%).
  [[literature/papers/paglieri2026case]] is the field observation of the
  same thing: the boundary lived only in a syntactic autograder, and once
  one agent found the seam the collective consumed the remaining benchmark
  in 27 minutes.
- The negative case now has a name of its own — see
  [[concepts/shared-substrate-contagion]] for what happens *after* a
  misplaced boundary is breached in a system with shared infrastructure.
