---
kind: concept
name: "typed-enforcement"
status: seedling
added: "2026-07-28"
sources:
  - "[[literature/papers/palumbo2026formal]]"
  - "[[literature/papers/mondl2026autoformalization]]"
  - "[[literature/papers/khan2026token]]"
  - "[[literature/papers/ye2026agent]]"
  - "[[literature/papers/zhao2026agenticos]]"
  - "[[literature/papers/madatha2026deterministic]]"
  - "[[literature/papers/louck2026securing]]"
  - "[[literature/papers/semenov2026beyond]]"
  - "[[literature/papers/chen2026governance]]"
  - "[[literature/papers/philippov2026glite]]"
  - "[[literature/papers/hao2026selfgc]]"
  - "[[literature/papers/elkoussy2026agentltl]]"
  - "[[literature/papers/ng2026agent]]"
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/verified-memory-writes]]"
related_experiments: []
tags: [policy, enforcement, formal-methods, types, determinism, governance]
---

# typed-enforcement

## Definition

An agent's constraints — permissions, budgets, workflow order, data-flow
rules — are expressed as a **machine-checkable artifact in a language with
decidable static analyses**, held outside the agent's reasoning and
enforced by a deterministic checker. The artifact is the policy; the
agent's compliance is a consequence of the checker, not of the agent
honoring prose.

The contrast class is the dominant practice: policy written as
natural-language instructions in a system prompt, with compliance
delegated to the model. That form admits no enforcement semantics, cannot
be checked for contradiction before it runs, and degrades exactly when
optimization pressure is highest.

## Why it matters here

Seven sources converge on this from four unrelated starting points —
programming languages, operating systems, formal verification, and an
incident corpus — which is the main reason to treat it as a concept rather
than a restatement of [[concepts/permission-gate-as-architecture]]. That
concept says the gate is architecture and describes *where* it sits. This
one is about *what the policy is written in*, and the payoff is that a
formal language admits checks a prompt cannot support.

**The language, not just the check.** [[literature/papers/palumbo2026formal]]
is the clearest statement: policies in Datalog over abstract predicates,
maintained by an observability service under an assume/guarantee contract,
consulted by a reference monitor at every action, with a correctness
theorem conditional on that contract. Datalog is chosen for four
properties, and the underrated one is **tractable static analysis for
contradiction, redundancy, subsumption, and conditional reachability** — a
policy author can find out that their rules conflict *before* deployment.
FORGE eliminates violations by construction (prompt-injection success
100% → 0% with no benign false positives; τ²-bench compliance 58% → 98%;
unauthorized accesses 40 → 0) at 19–38% latency overhead, and it does this
over unmodified agents.

**Types as the checker.** [[literature/papers/khan2026token]] reaches the
same place from Rust: an affine `Budget` type makes cloning,
double-spending, or use-after-delegating a budget a *compile error*. The
enforcement is not a runtime monitor at all — the illegal program does not
exist. [[literature/papers/ye2026agent]] supplies the algebra one level
up: a contract seven-tuple with a **conservation law** guaranteeing
delegated budgets never exceed the parent's, with zero conservation
violations across 50 multi-agent trials.

**What a policy signal may legitimately bind to.**
[[literature/papers/louck2026securing]] is the negative theorem of the
cluster, machine-checked in TLA+: any authority signal derived from
*content* or a *derivation edge* is malleable and can be laundered, so
only non-malleable origin binding is sound. This is the constraint on what
predicates a typed policy may be written over — it is not enough for the
policy language to be formal if its atoms are attacker-influenceable.

**Why not just use an LLM to check.**
[[literature/papers/madatha2026deterministic]] argues the enforcement layer
must be ordinary testable code — hashing, state machines, blocklists — and
explicitly **not further LLM orchestration**, on the grounds that a
non-deterministic component cannot be a trustworthy control for another
non-deterministic one. Its prevalence study is the empirical motivation:
across 6,145 real agent-config files, **fewer than 1% declare any
permission boundary at all.**

**The compiler from prose.** The obvious objection to all of the above is
that nobody writes Datalog for their agent, and hand-coded enforcement
does not scale — prior work implemented 23 of an 88-rule policy.
[[literature/papers/mondl2026autoformalization]] closes the gap with a
generator–critic pipeline compiling prompts, tool schemas, and policy
documents into Cedar, paired with a deterministic hard critic (syntax,
schema, vacuity, rule conflict) and a semantic soft critic. Coverage, not
soundness, was the binding constraint, and autoformalization is the answer
to coverage.

**The whole stack, as an OS.** [[literature/papers/zhao2026agenticos]] is
the maximal version: agents never touch POSIX primitives, they submit a
structured intent Manifest, and the system synthesizes a least-capability
environment from it, with resource budgets bound into capability tokens.

**The pattern outside security.** [[literature/papers/semenov2026beyond]]
attests the same shape in a domain with no adversary: context management.
The agent declares typed structure (`expl`/`act` episodes with dependency
edges) and a deterministic, LLM-free policy — not the model — decides
what survives eviction, explicitly to rule out compression-induced
hallucination "by construction." Same division of labor as
madatha2026deterministic: the model authors the typed artifact, ordinary
code enforces over it. Evidence is design-paper grade (one demo), but as
an eighth source it shows the concept generalizes beyond
permissions/budgets to any policy the agent must not be trusted to
execute at inference time.
[[literature/papers/chen2026governance]] then supplies the measured
cost of *not* doing this: prose policy left in evictable context goes
from 0% to 30% violation (up to 59%) after one compaction pass,
because a model — the summarizer — was trusted with policy survival.
Its Constraint Pinning defense ([[concepts/constraint-pinning]]) is
typed-enforcement applied to the context layer: harness code, not
model judgment, guarantees the invariant.

**The pattern at research-campaign scale.**
[[literature/papers/philippov2026glite]] deploys the thesis on the
research process itself — "the rules of the research process live in
code that fails loudly when violated, not in prose that agents are
merely asked to follow" — and reports the operating point: across
thousands of agent invocations, prompt-encoded rules never reach zero
failure ("we tried"), while deterministic verifiers (versioned
artefact specs + stable error codes gating every merge) cost ~1% of
wall-clock and let a 12-agent, 273-task campaign win an external
refereed shared task with zero cross-task corruption. It also states
this cluster's escape hatch in its most concrete form yet: the
verifiers are ordinary scripts in the agent-writable repository, so
they stop accidental drift, not an agent instructed to rewrite the
verifier — enforcement that lives inside the agent's write scope is
advisory under adversarial pressure (cf. louck2026securing's
origin-binding and chen2026governance's operator-impersonation
residual).

**How often the model would actually breach the invariant.**
[[literature/papers/hao2026selfgc]] is the first source here to measure
the quantity this cluster keeps assuming. Its context-governance planner
is well-prompted, given an explicit object-action contract with hard
rules and few-shot examples, and told never to touch the latest visible
user turn. Across three backbones it proposed cutting that protected
turn in **25/330 (Qwen3.6-Plus), 15/330 (Qwen3.7-Max), and 12/328
(GLM-5.1)** parsed plans — roughly 4–8% of the time. The paper's own
reading is the concept in one line: "the prompt usually works, but the
residual risk justifies mandatory last-turn protection."

That number is worth carrying because it cuts both ways. It is small
enough to explain why prompt-only enforcement *looks* fine in
development and in demos, and large enough that at production volume it
is a steady stream of violations — which is exactly the shape
philippov2026glite reports qualitatively ("we tried"; prompt-encoded
rules never reach zero). It also partially answers this concept's open
question about a head-to-head between formal enforcement and a
well-prompted frontier model: on a single, simple, structurally
checkable invariant, the model's error rate is a few percent and the
deterministic check costs nearly nothing. It is only a partial answer —
one invariant, non-adversarial, and the harness check is trivial to
write — but it is a real measurement where the cluster previously had
only argument.

The paper's other contribution is the cleanest statement of the division
of labor: "the model supplies semantic judgment about future value,
while the harness enforces runtime invariants such as recoverability and
protocol validity." Note the corollary its deployment result supplies —
because the harness owns the invariants, a **mid-tier planner suffices**
(all three backbones exceed 90% no-impact). Deterministic enforcement is
not only a safety property; it is what makes the expensive model
optional.

**One spec, three uses — and the first negative enforcement result.**
[[literature/papers/elkoussy2026agentltl]] extends the pattern along
the temporal axis: procedural rules (ordering, branching, iteration,
grounding) as FO-LTL constraints over the tool-call trace, yielding a
deterministic judge-free compliance score that a *single*
specification drives three ways — offline measurement, online
pre-execution gating, and a dense RL reward. The shared language is
the point: evaluation, deployment, and training cannot drift apart
when they read the same artifact, and finetuning against the score
transfers structurally (+38pp accuracy on held-out patterns with
unseen tool aliases — the model learns the procedure, not the tool
names). But it is also the cluster's first measured evidence that
**enforcement placement can hurt**: block-and-warn improves 5/7 models
(most on the weakest) yet regresses two strong models already near
their compliance ceiling, and a kill-switch variant (terminate after 3
violations) is the *worst* setting for most models because forced
termination prevents recovery the gate would otherwise permit. The
gap between the two settings is itself a diagnostic — it measures how
much non-compliance is locally recoverable. Deterministic checking of
the policy is uncontested across the cluster; *what the checker does
on violation* (log, warn-and-retry, block, kill, roll back) is now an
open design axis with evidence that the harshest response is usually
wrong, and that in irreversible workflows mid-procedure blocking can
itself leave the system inconsistent.

## The honest limit: every instance has a semantic escape hatch

This is the part worth carrying forward, because each paper states it only
about itself and the pattern is only visible across the set:

- FORGE has `llm_check`, an explicitly probabilistic foreign function for
  predicates Datalog cannot express structurally.
- The autoformalization pipeline's soft critic is an LLM-as-judge
  validating LLM-generated policy — generator and semantic verifier share
  a failure mode; only the hard critic is independent, and it can catch
  mechanical faults but never a *wrong* rule.
- ye2026agent's negative result is sharper still: **true pre-flight spend
  reservation is impossible with current provider APIs**, because token
  consumption is unknown until a call returns. Contracts can stop the next
  expensive call, not the current one.
- mondl2026autoformalization concedes some enforceable constraints (FHIR
  rules) are not derivable from the policy text at all and had to be
  transcribed from the implementation.

So the guarantee typed enforcement provides covers the **formal skeleton**,
and the semantic predicates hanging off it are as reliable as the model
evaluating them. "Formally enforced" should be read as "formally enforced
modulo the escape hatches," and a design's quality is largely a question of
how much it pushes into the skeleton versus the hatch. This is the
cluster's open frontier, not a footnote.

## Composition: when several checkers are cheap, and when they are not

[[literature/papers/ng2026agent]] answers a question this concept has been
silent on: what happens when you deploy *several* deterministic checkers
at once. Model each as a deterministic finite automaton over an
observation alphabet (a harness monitor) and each evidential gate as an
evidence-chain checker; the composed harness enforces the conjunction of
their properties. The cost of verifying the composition is **polynomial
when the observation alphabets are pairwise disjoint** (standard parallel
composition of finite-state monitors, with disjointness guaranteeing
non-interference) and in the disjoint or sequential cases stays tractable
for the small-state monitors typical of deployed harnesses. When monitors
**share events**, disjointness fails, non-interference is no longer free,
and one falls back to assume-guarantee reasoning — exponential in the
general case.

That condition is the load-bearing caveat for us, because a real hook
layer violates it by default: a permission gate and a trajectory monitor
both observe `tool_call`. So "add another checker" is not automatically a
free move, and a claude-system hook set whose members observe overlapping
events has no polynomial guarantee behind it. Worth knowing before
treating hook composition as costless.

The paper is also careful about what a checker buys, in terms this concept
should adopt: the guarantee is not that a checked artifact is correct — "a
flaky test still produces wrong gates" — but that **architectural
responsibility** moves, with the burden of producing the artifact on the
agent and the burden of verifying it on the harness, "not to a reasoning
chain inside the model." Its hard/soft evidence criterion gives the
type-level version: a property is checkable if some deterministic
polynomial-time verifier can decide it from an event plus *external
reference state, without access to the agent's internal state*. That
access restriction is the cleanest formal statement of what makes an
artifact enforceable rather than merely asserted. See
[[concepts/evidence-gated-completion]].

## Connections

- [[concepts/permission-gate-as-architecture]] is the sibling concept:
  that one covers gate placement, statefulness, and lifetime; this one
  covers the policy artifact's language and its static checkability. Most
  sources appear in both, from different angles — read them together.
  louck2026securing's malleability theorem is the hinge: it constrains
  what a gate may key on *and* what a typed policy may quantify over.
- [[concepts/budget-as-ceiling]] is the instance this box actually runs.
  `budget.yaml`'s ceilings are prose-adjacent config honored by skills;
  khan2026token and ye2026agent describe what it would mean for them to be
  a type or a conserved quantity instead.
- [[concepts/hierarchical-delegation]] is where the difference bites:
  point-in-time gates and prompt-resident policy both fail across agent
  boundaries, which is exactly what conservation laws and multi-agent
  reference monitors are built for.
- Tension with [[concepts/scripted-tool-pipelines]]: both move work out of
  the model into deterministic code, but scripted pipelines do it for
  reliability and cost, while typed enforcement does it for guarantee.
  They will often be the same refactor.

## Open questions

- **Cost/benefit at single-operator scale is unestablished.** Every result
  here comes from a multi-tenant or security-critical setting. For this
  box, the cheap importable piece is not a reference monitor — it is
  **static analysis of the rule set**: this project's `CLAUDE.md` and
  `.claude/rules/` are a natural-language policy that has never been
  checked for contradiction, redundancy, or vacuity, and `/lint` checks
  graph hygiene rather than rule consistency. That is the concrete
  proposal this concept generates, and it is a `/elevate` candidate.
- No source measures **false positives from autoformalized policy**.
  mondl2026autoformalization explicitly evaluates coverage rather than
  utility, which leaves the failure mode most likely to sink the approach
  unmeasured.
- No head-to-head between a formal policy engine and a well-prompted
  frontier model on the same policy set. The prompt-based baseline is
  argued to be unsound (correctly), but "unsound" and "worse in practice"
  are different claims and only the first is established.
  **Partially answered** by [[literature/papers/hao2026selfgc]]'s 4–8%
  cut-turn violation rate across three planner backbones — the first
  measured error rate for a well-prompted model on an invariant a
  deterministic check enforces for free. Still open in the general case:
  one simple structural invariant, no adversary, and no comparison on a
  policy set rich enough that the deterministic checker is itself hard to
  write. The interesting regime is where the invariant is *expensive* to
  check deterministically, and nobody has measured that.
- Whether static analysis over an *autoformalized* policy is meaningful
  when the policy was generated from prose the analysis never sees —
  contradiction-checking the output does not detect that the input was
  misread.
