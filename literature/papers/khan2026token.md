---
kind: paper
title: "Token Budgets: An Empirical Catalog of 63 LLM-Agent Budget-Overrun Incidents, with an Affine-Typed Rust Mitigation as a Case Study"
authors: ["Sajjad Khan"]
institutions: ["Independent researcher (MSc, University of the West of England, Bristol)"]
year: 2026
venue: "arXiv preprint (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.04056"
code_url: "https://github.com/sajjadanwar0/token-budgets"
citations: null
source: "raw/papers/khan2026token.pdf"
added: "2026-07-20"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: [budget, cost-control, failure-catalog, incident-taxonomy, type-systems, affine-types, rust, delegation, reliability]
---

# Token Budgets: An Empirical Catalog of 63 LLM-Agent Budget-Overrun Incidents, with an Affine-Typed Rust Mitigation as a Case Study

## TL;DR

The first incident-corpus study of LLM-agent budget overruns: 63
confirmed production incidents across 21 frameworks / 18 ecosystems
(2023–2026), each backed by a quoted GitHub issue and, where reported, a
dollar loss, organized into an eight-cluster mechanism taxonomy
(two-rater Cohen's κ = 0.837 on case labels, N = 113); as one mitigation
it builds `token-budgets`, a 1,180-line no-`unsafe` Rust crate whose
affine `Budget` type makes cloning, double-spending, or
use-after-delegating a budget *compile errors*, isolating its value to
non-bypassability in multi-agent delegation.

## Claims

- Budget overruns are a documented production failure class, not an
  anecdote: 63 confirmed incidents (plus 47 supplementary structural
  entries — 28 maintainer-acknowledged gaps, 14 feature requests, 5
  borderline) recur across 21 independently-developed projects. The
  paper is explicit that this establishes *recurrence, not incidence* —
  the sampling frame selects on the dependent variable (threat C1) and
  dollar aggregates are lower bounds.
- **Reactive fixes dominate.** Fixes ship fast once reported (median
  days; several same-day), but the catalog contains **no case where a
  budget overrun was prevented before at least one user paid for it**.
  Every deployed mitigation the catalog surfaces is post-hoc (budget
  alerts, `BudgetExceeded` circuit breakers, HTTP 402 at the payment
  layer) — none catches the spend before the API call commits.
- Eight architectural mechanism clusters (rows/frameworks):
  M-retry-loop (27/12), M-cost-observability (22/7),
  M-context-amplification (13/7), M-storage-amplification (13/5),
  M-budget-primitive-missing (12/6), M-delegation-fanout (11/6),
  providerOptions-silently-dropped (6/3), M-multimodal-cost-amplification
  (6/2). The cluster partition is flagged as exploratory (blind
  second-rater cluster agreement κ = 0.44; only cost-observability and
  multimodal reliably re-identified); the validated labels are the
  four-class case scheme (κ = 0.837).
- First-party vendor tools exhibit the class too: two claude-code
  compaction-loop incidents (CCDE-001: ≈$235 in four days for one user,
  ≈$1,760/month extrapolated; CCDE-002 with 10+ sibling issues), a
  Pydantic AI multi-agent docs example that fails its own
  `total_tokens_limit` default, OpenAI Agents SDK maintainer admitting
  no graceful degradation when `max_turns` is exceeded.
- The affine discipline's honest scoping is the paper's sharpest move:
  on single-agent workloads a **4-line Python counter matches the Rust
  crate at 0/30 overshoot**, so the type system buys nothing there. The
  distinguishing value is non-bypassability under operator error in
  multi-agent delegation: the delegation-fanout race documented in 11
  incidents is a compile error under the borrow checker, while the same
  pattern under `asyncio` overshoots 30/30 (three disciplined Python
  alternatives: 0/30 each — a structural fix for one pattern, not a
  general upgrade).
- Three-layer enforcement taxonomy for deployed mitigations:
  compile-time (this work), software layer (AgentGuard-style callbacks,
  LiteLLM proxies — after the spend), transport layer (ATXP's HTTP 402 —
  after the request is in flight). Only the compile-time layer catches
  integrity violations (aliasing, double-spend, use-after-delegation)
  before any external resource is consumed.
- A decision matrix (Table 3) deliberately positions the crate as *one
  cell*: provider-side kernel caps (`max_completion_tokens`, Bedrock
  budget actions) are preferred where they exist; Python frameworks
  should use runtime caps; reasoning models structurally violate the
  output-cap assumption (hidden thinking tokens unbounded by
  `max_output_tokens`), demoting the approach to defense-in-depth there.

## Methods

- Corpus: 21 sub-projects from GitHub topics (`llm-agent`,
  `agent-framework`, `ai-agent`, `llm-orchestration`), ≥1,000 stars as
  of Jan 2026, filtered to projects exposing a budget/cost primitive or
  having an overrun issue. Issue-tracker search on failure keywords: 167
  candidate URLs → 110 retained rows (each with verbatim quotation, one
  of five labels); full survey including triaged-out rows ships in
  `catalogue.csv`.
- Reliability: an independent second coder (no prior catalog exposure,
  blinded to original codings) re-annotated all rows under a frozen
  codebook: κ = 0.837 (95% CI [0.745, 0.919]) on the four-class scheme,
  κ = 0.943 on the both-confirmed subset; the 12 disagreements and
  adjudications ship in the artifact (`irr/`).
- Selection-bias bounding: an independent keyword-neutral baseline
  cohort — top-20 starred "LLM agent" projects, 3,461 issues pulled, 186
  body-read under the same codebook — finds the mechanism clusters recur
  in 12/20 projects (60%; Wilson CI [40%, 77%]), though single-coder.
- Mitigation eval: five production runtimes (LangGraph, CrewAI, AutoGen,
  AgentGuard-style callback, LiteLLM proxy) plus concurrent work (Agent
  Contracts), three providers, 382 live-API sessions plus a calibrated
  simulation (2,628 trials), and a temperature-stratified live test
  (T ∈ {0.0, 0.3, 0.7, 1.0}, N = 160, two production models).
- Cost of the discipline: default static estimator over-reserves 4–6×
  (2.51× median); `AdaptiveEstimator` tightens to 2.11× median at zero
  latency; tokenizer-direct reaches ~1.0–1.1× at 939–1,749 ms per-spend
  latency. Break-even analysis for prepay vs post-pay deployments.

## Results

- Live cap behavior: zero cap violations and zero false refusals across
  the temperature-stratified sweep; at a discriminating cap
  (B₀ = 2,000 uc) 0/30 overshoot vs 30/30 baseline; at a sub-floor cap
  0/30 pre-flight refusal vs 30/30 baseline post-hoc overshoot;
  cap-sweep robustness 30/30 across 10 caps; multi-agent delegation 0/60
  aggregate, 0/180 per-child.
- The Forgetful-Operator experiment isolates the contribution: a
  properly locked Python counter also achieves 0/30 — the affine layer's
  delta is that the *incorrect* version does not compile, so the
  delegation guarantee no longer depends on the operator getting
  concurrency right.
- What stays open: binary-level cap-soundness on the running Tokio
  binary (Conjecture 1, explicitly unproven); estimator assumption A1
  must be re-calibrated per provider/model (up to 9.97× over-reservation
  on an adversarial hold-out); provider-reported `actual_charge`
  truthfulness is a trust assumption shared with every client-side cost
  accounting layer.

## Critique / open questions

- Solo independent author; the IRR second rater is a hired coder, not an
  institutional collaborator — the reliability study is real but thinner
  than it reads. arXiv-only, no citations yet.
- The catalog is a convenience sample of public, English-language GitHub
  issues; closed-source platforms (Cursor, Replit Agent) absent; the
  paper is commendably explicit about all of this, but it means the
  eight-cluster structure should be treated as existence proof, not
  distribution.
- The Rust mitigation applies directly to a small minority of the
  surveyed surface (~7–8 of every 10 retained incidents are in Python
  frameworks); the transferable asset for non-Rust projects is the
  catalog + the three-layer taxonomy + the pre-flight-reservation
  principle, not the crate.
- The 47 supplementary rows do useful corroborating work but invite
  headline inflation ("110 cases"); the paper polices the distinction
  carefully — readers of second-hand summaries may not.

## Trust signals

- **Credibility:** 3 — independent single author, arXiv preprint, no
  peer review or citations; but the full artifact is public and
  auditable (crate source, `catalogue.csv` with per-row quoted evidence,
  IRR coding sheets + adjudications + scripts), the two-coder
  reliability study and construct-validity threat analysis are unusually
  rigorous for a solo preprint, and every headline number can be
  re-derived from the shipped CSV — reproducibility carries the score
  per the rubric's institution-is-a-prior-not-a-verdict clause.

## Follow-up

- **Relevance:** 5 — first empirical (incident-corpus) anchor for
  [[concepts/budget-as-ceiling]], whose six prior sources are all
  design-position papers: the catalog converts the ceiling from a
  sensible stance into a documented necessity, and its
  no-overrun-prevented-before-payment finding is the canonical argument
  for pre-flight (refuse-before-the-call) enforcement over post-hoc
  counters. The M-context-amplification cluster independently attests
  [[concepts/context-eviction-policy]] as a dollar-cost control.
- **/elevate note:** the evidence here is strong by the /elevate bar —
  63 quoted, independently re-coded, artifact-auditable production
  incidents plus a keyword-neutral replication cohort. It empirically
  motivates the `budget.yaml` ceiling discipline claude-system already
  has; the elevate-worthy increment is *pre-flight reservation* (check
  the projected spend and refuse before issuing the call) versus the
  current halt-after-the-cycle counters, since the catalog found no
  overrun ever prevented post-hoc before a user paid.
- Watch: whether the typed-enforcement thread (this paper's affine
  capability, zhao2026agenticos's capability tokens,
  louck2026securing's non-malleable gate data,
  madatha2026deterministic's deterministic core) thickens into its own
  concept — three-plus independent sources now argue that hard agent
  invariants belong in deterministic/typed substrate below the agent;
  today it lives distributed across budget-as-ceiling and
  permission-gate-as-architecture.
