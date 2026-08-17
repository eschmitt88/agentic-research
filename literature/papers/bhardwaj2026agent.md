---
kind: paper
title: "Agent Behavioral Contracts: Formal Specification and Runtime Enforcement for Reliable Autonomous AI Agents"
authors: ["Varun Pratap Bhardwaj"]
institutions: ["Accenture"]
year: 2026
venue: "arXiv 2602.22302v1, cs.AI (preprint, 2026-02-25; 71 pages)"
peer_reviewed: false
url: https://arxiv.org/abs/2602.22302
code_url:
citations:
source: "raw/papers/bhardwaj2026agent.pdf"
added: "2026-08-17"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evidence-gated-completion]]"
tags: [typed-enforcement, design-by-contract, runtime-monitoring, behavioral-drift, dsl, composition, observability, enterprise]
---

# Agent Behavioral Contracts (ABC)

## TL;DR

Design-by-Contract for agent *sessions* rather than function calls: a
contract C = (P, I, G, R) of Preconditions, Invariants (hard and soft),
Governance policies, and Recovery mechanisms, written in a YAML DSL
(ContractSpec), enforced at runtime by a monitor library (AgentAssert), with
a probabilistic (p, δ, k)-satisfaction notion for LLM non-determinism and an
Ornstein–Uhlenbeck drift-bound theorem. Evaluated on 1,980 sessions across
7 models from 6 vendors. **The measured benefit is observability, not
prevention** — and the paper says so plainly: "The value of ABC contracts is
not that they eliminate violations, but that they make violations
measurable."

## Claims

- **The gap is formal semantics.** Traditional software has type systems,
  assertions, and interface specifications; agents are "governed by
  prompts—natural language instructions that carry no formal semantics, no
  verifiable guarantees, and no enforcement mechanisms," and that gap is
  the root cause of behavioral drift, governance violations, and silent
  degradation. Constitutional AI and RLHF shape general tendencies but
  cannot enforce deployment-specific invariants; output guardrails (NeMo,
  Guardrails AI) filter individual responses without session-level state,
  drift detection, or composition.
- **Contract structure.** Preconditions gate entry; Invariants must hold
  across time and are split **hard** (must never be violated) vs **soft**
  (violations tolerated within a bound and recoverable); Governance
  policies constrain actions; Recovery mechanisms respond to soft
  violations by re-prompting. The generalization over prior DbC-for-LLM
  work (Leoveanu-Condrei 2025) is from single LLM invocations to
  *sessions*, adding cross-turn invariants and multi-agent composition.
- **(p, δ, k)-satisfaction.** Contracts hold with probability ≥ p,
  deviations stay within tolerance δ, recovery occurs within k steps — a
  probabilistic compliance notion for a non-deterministic executor, rather
  than the binary satisfaction CPS contracts assume.
- **Stochastic Drift Bound Theorem.** Modeling behavioral drift as an
  Ornstein–Uhlenbeck process with natural drift rate α and contract
  recovery rate γ, Lyapunov analysis gives bounded drift converging to
  D* = α/γ in expectation when γ > α, with Gaussian concentration and a
  closed-form contract-design criterion.
- **Compositionality Theorem.** Four sufficient conditions — interface
  compatibility, assumption discharge, governance consistency, recovery
  independence — under which per-agent contract guarantees compose into
  end-to-end guarantees for multi-agent chains, with quantified
  probabilistic degradation bounds. Extends assume-guarantee reasoning
  (Henzinger et al.) to agent pipelines.
- **Positioned as orthogonal to resource contracts.** Explicitly:
  ye2026agent's Agent Contracts govern *how much* an agent may consume
  (tokens, time, cost, delegation hierarchies with conservation laws),
  while ABC governs *how it must behave*. "The two frameworks address
  orthogonal concerns and could be composed: resource contracts bounding
  computation, behavioral contracts bounding behavior."
- **Specification-first, and drift as a leading indicator.** Contracts are
  declared before deployment (unlike VeriGuard, which verifies generated
  code, or StepShield, which evaluates traces post hoc), and drift
  detection is intended to enable intervention *before* a constraint
  violation materializes.
- **The headline result is a transparency effect.** Contracted agents
  register **5.23–6.83 soft violations per session**; uncontracted
  baselines register **0.00–0.30** — all seven models, all pairwise
  differences p < 0.0001, Cohen's d from 6.70 (Llama 3.3 70B) to 33.82
  (GPT-5.2). The paper's own reading: without a contract "an agent's
  behavioral compliance is undefined. With a contract, it is quantified,
  tracked, and actionable." The contract did not remove violations; it
  created the yardstick that makes them countable.
- **Platform guardrails and contracts can be incompatible.** A separate
  finding: "organizations cannot simply layer behavioral contracts on top
  of platform guardrails without compatibility testing," motivating a
  three-layer analysis (no guardrails / platform default / platform
  strict).
- **Adversarial stress produces defensive tightening.** Under prompt
  injection, GPT-5.2 (−2.07), Llama 3.3 70B (−0.70), and Mistral Large 3
  (−3.17) showed *negative* soft-violation spikes — they answered
  injection attempts with more conservative, more compliant behavior.

## Methods

- 7 models from 6 vendors (GPT-5.2, Claude Opus 4.6, DeepSeek-R1, Grok-4
  Fast, Llama 3.3 70B, Mistral Large 3, GPT-4o-mini), all via Azure AI
  Foundry for a uniform API. 30 sessions per model per condition.
- Four experiments: **E1** contracted vs uncontracted (6-turn sessions);
  **E2** drift over 12-turn extended sessions (4 models, 240 sessions);
  **E3** governance under adversarial prompt injection; **E4** structural
  ablation of each contract component (600 sessions).
- Conditions differ only in whether contract rules are injected and
  enforced: contracted = domain context + full rules + active AgentAssert
  monitor + LLM re-prompting recovery; uncontracted = domain context only,
  passive evaluation, no recovery.
- AgentContract-Bench: 200 scenarios over 7 domains and 6 stress profiles.
- Reproducibility: fixed seed (42) for task ordering, full JSON session
  traces recording every turn, constraint evaluation, violation event, and
  recovery attempt.

## Results

- **E1**: soft-violation deltas +5.23 to +6.63 per session, all p < 0.0001
  under Bonferroni-corrected Welch's t-tests. Hard compliance was
  **already near ceiling in both conditions** — five of seven models at
  C_hard ≥ 0.989 contracted, GPT-5.2 and GPT-4o-mini perfect in *both*
  conditions — so the contract's contribution to hard compliance is not
  demonstrated. Mistral Large 3 is the exception in the wrong direction:
  4.23 hard violations per *contracted* session.
- **Reliability index Θ**: 0.908 (Mistral Large 3) to 0.956 (Llama 3.3
  70B), all > 0.90. Notably the open-weight Llama 3.3 70B leads on Θ,
  lowest drift (0.073), and fewest soft violations — "contract compliance
  does not require proprietary alignment techniques."
- **E2 (12-turn sessions)**: drift stays bounded, max observed D_max =
  0.264 (Mistral), mean D̄ = 0.139; OU model fit R² = 0.49–0.75. Recovery
  success is highly model-dependent: 1.00 for GPT-5.2 and Claude Opus 4.6,
  **0.50 for Llama 3.3 70B, 0.17 for Mistral Large 3** (mean 0.67). Cost
  per 12-turn session $0.91–$2.71.
- **E3**: contracts surface adversarial effects invisible to uncontracted
  monitoring (uncontracted detects 0.03–0.77); recovery under stress
  ranges 0.33–0.91 for Claude Opus 4.6. GPT-5.2's 0% stress recovery rate
  is an artifact of never incurring a hard violation to recover from.
- **E4 ablation — the awkward one.** Removing soft constraints ("Hard
  Only") *increases* Θ by +0.025, and "Drift Only" by up to +0.036,
  because Θ is computed from detected soft violations: ablating the
  detector improves the score. Conversely "Soft Only" and "No Recovery"
  both cost −0.20 Θ. The ablation therefore demonstrates that the metric
  is entangled with the mechanism it evaluates more clearly than it
  isolates component value. Hard-constraint enforcement and drift
  monitoring are confirmed structurally independent of the soft/recovery
  path (identical C_hard and D̄ across conditions).
- Overhead: < 10 ms per action.

## Critique / open questions

- **The result is not the one a typed-enforcement argument wants.** This
  paper is frequently summarized as showing that formal contracts prevent
  violations prompts miss. It shows something weaker and more interesting:
  contracts make violations *countable*. Hard compliance was near ceiling
  without them; the large effect sizes are on a quantity that is
  undefined in the control condition, so d = 33.82 measures the presence
  of a measuring instrument, not a behavior change. Cite it for
  observability, not for prevention.
- **Artifact not available.** The title footnote reads: "Patent pending.
  Reference implementation and benchmark suite available subject to
  intellectual property clearance." So AgentAssert and
  AgentContract-Bench are described, not released — which is a hard
  problem for a paper whose contribution is a library and a benchmark.
- **The benchmark grades the specification language against itself.** The
  paper concedes it: AgentContract-Bench uses synthetic traces with
  pre-computed feature fields, so a scenario with `pii_detected: true`
  tests whether the evaluator can check `pii_detected == false` — not
  whether the PII detector is accurate. "The benchmark achieves high
  accuracy by design, since it evaluates the enforcement logic against its
  own specification language." Behavioral detection end-to-end is
  explicitly not covered.
- **Judge confound.** GPT-4o-mini is both a model under test and the
  LLM-as-Judge evaluator for *all* experiments. The paper checks for
  self-evaluation bias in a human-annotation study, but a single model
  scoring the entire evaluation including itself is a structural weakness.
- **One domain.** Every experiment uses the financial-advisor contract, so
  the seven-model, 1,980-session breadth is across *executors*, not across
  task types. Generalization to research-agent behavior is untested.
- The OU drift model is a fit (R² = 0.49–0.75) to a metric the paper also
  defines; the Drift Bounds Theorem is a property of the model, not an
  independently verified property of agents.
- Recovery is LLM re-prompting, i.e. soft enforcement — and it works
  0.17–1.00 of the time depending on the model. The framework's
  deterministic half (hard invariants) is the part that behaves like an
  enforcement mechanism; the recovery half inherits the executor's
  reliability, which is exactly the dependency typed enforcement is
  supposed to remove.

## Trust signals

- **Credibility:** 2 — a genuinely large evaluation (1,980 sessions, 7
  models, 6 vendors, four experiments including an ablation and an
  adversarial condition), pre-registered thresholds, fixed seeds, full
  trace capture, and unusually candid limitation sections. But: single
  author, industry practitioner with no academic co-author, 71-page
  unreviewed preprint, **no released code or benchmark** (patent-pending,
  IP-gated), a benchmark that grades its own DSL, a judge model that is
  also a subject, one task domain, and a headline reliability metric that
  the paper's own ablation shows improves when its detector is removed.
  The evidence is real but the artifacts are unavailable and the metric is
  entangled — hence 2 rather than 3.

## Follow-up

- **Relevance:** 4 — the largest cross-vendor evaluation of declarative
  runtime behavioral contracts surfaced so far, and a useful *negative*
  refinement of [[concepts/typed-enforcement]]: the demonstrated value of
  a machine-checkable specification here is that it defines the
  measurement, not that it changes the behavior. That distinction matters
  for how this project argues the concept, and it pairs with
  [[literature/papers/elkoussy2026agentltl]]'s finding that enforcement
  can regress strong models — together they say the case for typed
  enforcement should rest on observability and auditability rather than on
  a claimed compliance lift.
- The hard/soft invariant split maps cleanly onto this repo's own
  structure: HCE violations are hard (halt), lint findings are soft
  (report and recover). Worth noting that the paper's hard-invariant pass
  is deterministic and its soft-recovery path is an LLM re-prompt with
  0.17–1.00 success — the same division of labor
  [[concepts/scripted-tool-pipelines]] recommends, arrived at
  independently.
- Its orthogonality claim against ye2026agent (resource contracts bound
  *how much*, behavioral contracts bound *how*) is a clean framing for
  [[concepts/budget-as-ceiling]]'s scope: budget ceilings are not a
  behavioral governance mechanism and should not be argued as one.
- The guardrail-incompatibility finding is worth remembering for
  [[concepts/permission-gate-as-architecture]]: stacking a contract layer
  on top of an existing platform gate needs compatibility testing, which
  is the practical face of ng2026agent's non-disjoint-monitor caveat.
- Not a candidate for `/elevate`: the mechanism is unavailable, the
  benchmark is self-referential, and the claim this project would want
  (enforcement prevents what prose misses) is not what the data shows.
