---
kind: paper
title: "Agent Contracts: A Formal Framework for Resource-Bounded Autonomous AI Systems"
authors: ["Qing Ye", "Jing Tan"]
institutions: ["Independent researcher"]
year: 2026
venue: "COINE 2026 (16th Int'l Workshop on Coordination, Organizations, Institutions, Norms and Ethics for Governance of Multi-Agent Systems), co-located with AAMAS 2026 — accepted for oral presentation"
peer_reviewed: true
url: "https://arxiv.org/abs/2601.08815"
code_url: "https://github.com/flyersworder/agent-contracts"
citations: null
source: "raw/papers/ye2026agent.pdf"
added: "2026-07-27"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: [budget, resource-governance, multi-agent, delegation, conservation-laws, contract-net, enforcement, coordination]
---

# Agent Contracts: A Formal Framework for Resource-Bounded Autonomous AI Systems

## TL;DR

Revives the 1980 Contract Net Protocol as a *resource-governance*
mechanism: a seven-tuple contract C = (I, O, S, R, T, Φ, Ψ) with an
explicit lifecycle, plus a conservation law guaranteeing that budgets
delegated to sub-agents can never exceed the parent's. Measured 90%
token reduction with **525× lower variance** and zero conservation
violations across 50 multi-agent trials. Most valuable to us is the
honest negative section: **true pre-flight spend reservation is
impossible with current provider APIs**, because token consumption is
unknown until a call returns — so contracts cannot stop one expensive
call, only the next one.

## Claims

- **Resource governance is a systematic gap, not an implementation
  bug.** A survey of eight major frameworks (LangGraph, AutoGen, CrewAI,
  OpenAI Agents SDK, Google ADK, Bedrock, LlamaIndex, smolagents) finds
  all provide *operational* controls — max iterations, timeouts, rate
  limits — and **none** provide formal governance: cost budgets,
  temporal deadlines, success criteria, or conservation laws for
  delegation. MCP/A2A standardize connectivity; none formalize how much
  an agent may consume.
- **Specification and governance are the same object.** The contract
  states both what the agent *should do* (I, O, S, Φ) and how much it
  may consume doing it (R, T, Ψ). The distinction between I and R is
  load-bearing: input size does not predict consumption, since an agent
  can receive little and reason expensively, or receive much and
  transform cheaply.
- **Conservation law for delegation.** Σ_j c_j^(r) ≤ B^(r) for every
  resource r, invariant across sequential, parallel, hierarchical, or
  competitive execution. An orchestrator issues subcontracts with
  Σ R_i ≤ R_parent, which yields **bounded autonomy**: however capable
  an orchestrator is, it can create subcontracts but cannot exceed its
  own parent constraint. Recursive delegation follows, since a
  sub-agent that can contract is itself contract-bounded.
- **"Contracting as a capability."** Once contracts are first-class, the
  orchestrator need not select from a fixed agent pool — the contract
  becomes the *specification for instantiating* an agent, blurring
  routing and orchestration, and enabling meta-governance (contracts
  governing the creation of contracts).
- **The single-call enforcement limit — the paper's most useful
  claim.** Token consumption is `unknown` during an API call and
  `c_actual` only after it returns; even streaming APIs report usage
  metadata only at completion. Three consequences, stated plainly:
  contracts **cannot** prevent one expensive call from blowing the
  budget; they **can** prevent subsequent calls; therefore the value is
  *multi-call* protection. True pre-allocation would need provider-side
  changes that do not exist — interruptible generation with mid-stream
  cancellation, token reservation with guaranteed hard limits, and
  budget-aware inference.
- **Two enforcement layers with unequal strength.** *Soft* enforcement
  (budget-aware prompting, dynamic status updates) is best-effort and
  defeated by "token elasticity" — LLMs exceed stated budgets when
  constraints are tight. *Hard* enforcement is an external monitor at
  the orchestration layer that halts on breach without model
  cooperation. Fully enforceable: multi-call budgets, iteration limits,
  API-call limits, duration limits. Only *approximable*: cost ceilings.
- **The benefit is variance, not mean.** Repeatedly framed: contracts
  "transform unpredictable agent behavior into bounded, auditable
  operations." Mean quality is unchanged; catastrophic tails are removed.

## Methods

- Reference implementation on Google ADK (three experiments) and LiteLLM
  (one), Gemini 2.5 Flash / Flash-Lite. Bootstrap CIs, 10,000 resamples,
  percentile method. Within-subjects design (each problem run under both
  conditions).
- **Contamination-aware benchmark choice**, worth noting given today's
  other ingest: LiveCodeBench problems released *post*-February 2025 and
  OpenR1 puzzles from February 2025, both after the model knowledge
  cutoff.
- Four experiments: Code Review (n=70, runaway prevention), Research
  Pipeline (n=50, conservation laws via a Researcher → Analyzer →
  Reporter chain with `DelegatingAdkAgent`), Strategy Modes (n=50,
  satisficing tradeoffs), Crisis Communication (n=24, failure
  prevention).
- Budget allocation strategies: proportional (by estimated complexity),
  equal, or negotiated (agents request, coordinator caps to prevent
  over-claiming), with a 10–15% reserve buffer and **dynamic
  reallocation** — unused budget from completed agents returns to a
  shared pool, so efficient agents subsidize expensive ones without
  breaking the total.

## Results

- **Code Review (n=70)**: 3,461 vs 34,606 tokens (−90%, p=0.0007);
  variance 10.1M vs 5.29B (**525× lower**); iterations −43%; LLM calls
  −50%. Success rate 52.9% vs 60.0% — **−7.1 pp, p=0.13, not
  significant**, and reported as such. Governance value scales with
  difficulty: 92% token savings on medium problems vs 76% on easy.
- **Research Pipeline (n=50)**: zero conservation violations across all
  trials. One trial caught a runaway agent that had exceeded its 40K
  budget — halted at 56K consumed, which is the single-call limitation
  visible in the data. Quality variance 26.7× lower (σ 1.75 vs 9.07);
  excluding the one catastrophic uncontracted failure, still 1.4× lower
  (88.5% Bayesian probability).
- **Crisis Communication (n=24)**: 23% fewer tokens (p=0.005) at
  statistically equivalent quality (p=0.32). One uncontracted agent
  failed outright, stuck in an evaluation loop without submitting; the
  contracted version succeeded — iteration governance prevents failures,
  not merely cost.
- **Strategy Modes (n=50)**: a clean quality-resource gradient —
  URGENT 70% success / 0 reasoning tokens / 26% timeout; ECONOMICAL 76%
  / 718 / 14%; BALANCED 86% / 1,519 / 10%. 75% more tokens buys 16 pp
  success (p ≈ 0.05).

## Critique / open questions

- **Success rate went down, not up.** −7.1 pp is not significant at
  n=70, but neither is it evidence of no harm; the study is not powered
  to exclude a real quality cost. The paper's framing (variance, not
  mean) is the right one, but a reader who wants "governance is free"
  will not find it demonstrated here.
- The runaway-agent anecdote is offered as evidence enforcement works,
  and it also demonstrates the limitation: the agent consumed 56K
  against a 40K budget — a 40% overshoot — *before* the monitor could
  halt it. That is exactly the single-call gap, and it means a ceiling
  is really a ceiling-plus-one-call.
- Both authors are independent researchers with gmail/live addresses;
  no institutional backing. Offsetting: accepted for oral presentation
  at a peer-reviewed AAMAS workshop, code released, bootstrap CIs,
  within-subjects design, post-cutoff benchmarks, and non-significant
  results reported as non-significant.
- Evaluation is entirely on Gemini 2.5 Flash / Flash-Lite. Token
  elasticity and budget-adherence behavior plausibly differ on frontier
  reasoning models, which is the regime we actually run in.
- Quality assessment in the multi-agent experiment is multi-judge LLM
  evaluation; no human validation reported.
- The formalism is a well-organized vocabulary rather than a proof
  system — "conservation laws" are enforced by an allocator that caps
  subcontract sums, not derived as theorems about agent behavior. The
  contribution is architectural discipline, and the framework is
  self-described as "under active development."

## Trust signals

- **Credibility:** 3 — peer-reviewed venue (COINE 2026 @ AAMAS, oral)
  and released code, against no institutional affiliation and a
  single-model-family evaluation. Methodologically careful in ways that
  matter: within-subjects design, 10,000-resample bootstrap CIs,
  deliberately post-cutoff benchmarks, and honest reporting of a
  non-significant (and directionally unfavourable) success-rate result.
  A high 3 rather than a 4 because the empirical base is one model
  family and the "formal framework" is definitional rather than proved.

## Follow-up

- **Relevance:** 5 — the formal object that connects
  [[concepts/budget-as-ceiling]] to [[concepts/hierarchical-delegation]].
  Our coordinator hands ceilings to subagents with no stated invariant
  about what a child may spend relative to its parent; the conservation
  law Σ R_i ≤ R_parent plus dynamic reallocation from a reserve pool is
  a directly adoptable design. Also the first source in the cluster with
  a peer-reviewed venue.
- **Resolves the open `/elevate` question carried in NOTES since
  2026-07-20.** khan2026token argued for *pre-flight spend reservation*
  over our halt-after-cycle counters. This paper's §7.1 says pre-flight
  reservation is **not implementable** against current provider APIs —
  consumption is unknown until the call returns, and token reservation
  with guaranteed hard limits is listed among the *missing* provider
  capabilities. The two are reconcilable: khan's affine-typed ownership
  prevents *double-spending an allocation* in typed source, which is a
  real in-process integrity property, but it cannot bound a single
  call's actual cost either. Conclusion for the next `/elevate` cycle:
  our halt-after-cycle design is the correct achievable one, and the
  honest fix is to (a) keep hard enforcement between actions, (b) treat
  every ceiling as ceiling-plus-one-call, and (c) size the reserve
  buffer to cover one worst-case call rather than pretending the
  boundary is exact. That is a smaller and better-founded change than
  building a reservation layer.
- Concrete gap in our own `budget.yaml`: it declares ceilings but no
  **reserve buffer** and no delegation invariant. The paper's 10–15%
  reserve is a cheap addition that would make the overshoot bounded
  rather than unbounded.
- The satisficing contract-modes result (URGENT/ECONOMICAL/BALANCED as
  a declared quality-resource point) is a cleaner interface than our
  per-skill model-role defaults. Worth considering if we ever want
  `/iterate` to trade depth for breadth explicitly rather than by
  editing model roles.
