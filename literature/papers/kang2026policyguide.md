---
kind: paper
title: "POLICYGUIDE: From Guarding One Action to Guiding the Whole Workflow for Policy-Compliant LLM Agents"
authors: ["Seongjae Kang", "Taehyung Yu", "Sung Ju Hwang"]
institutions: ["KAIST", "DeepAuto.ai"]
year: 2026
venue: "arXiv 2608.19861v1, cs.AI (preprint, 20 Aug 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2608.19861
code_url: null
citations: null
source: "raw/papers/kang2026policyguide.pdf"
added: "2026-08-24"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
related_experiments: []
tags: [policy, enforcement, workflow, permission-gate, runtime-verification, tau2-bench, adversarial-robustness]
---

# POLICYGUIDE: From Guarding One Action to Guiding the Whole Workflow for Policy-Compliant LLM Agents

## TL;DR

Argues that policy compliance is a **joint safeguarding-and-workflow-enforcement
problem**: an action-triggered guard only ever sees the call it fires on, so
it cannot discover a skipped identification step or a misordered eligibility
check until the final mutating call — by which point the procedural
violation has already happened and cannot be undone. POLICYGUIDE compiles
each domain policy into a workflow graph and runs an external, model-paired
proactive verifier at every user-turn boundary that walks the graph from its
persisted position, reconciles open requests, and returns step-specific
remediation for the first unmet requirement — before a mutating call is
even attempted. Across τ²-bench airline/retail/telecom with a GPT 5.4
agent+verifier, mean Pass⁴ rises 0.42 → 0.62, largest on telecom (0.19 →
0.61, the most workflow-structured domain), and the same frozen workflow
graph transfers unchanged to Claude Sonnet 4.6 and Gemini 2.5 Pro agents.

## Claims

- **Action-triggered coverage is a strict subset of workflow-level
  coverage, formally.** Theorem 1 defines a *reachable first deviation* as
  a point where a compliant prefix can be followed by a policy-violating
  agent event, and shows a firing schedule preserves procedural validity
  throughout an execution iff it covers every such deviation
  (`C_S(G) = D_G`). Corollary 1: an action-triggered schedule covers this
  set only when *every* reachable first deviation happens to fall inside
  the guarded action class `A` — i.e., only when the policy's failure
  modes are all mutating-call failures. Corollary 2 (evaluated boundary
  schedule) states plainly that firing after every user turn plus a
  one-shot correction on an unauthorized mutating call does **not**
  provide an unconditional guarantee — "if a first deviation can occur
  after an intervening agent event and before the next scheduled firing,
  the evaluated schedule does not provide an unconditional guarantee."
  POLICYGUIDE's own deployed configuration is explicitly named as falling
  short of the theorem's ideal case.
- **The two-axis framing (Figure 1) cleanly separates four prior
  approaches.** External-safeguarding × workflow-enforcement, both
  binary: unguided agents (neither), workflow/SOP agents (enforce
  procedure, not safeguard against violating behavior), action verifiers
  like PolicyGuard/ToolGuard (safeguard single actions, no procedure
  awareness), and POLICYGUIDE (both). PolicyGuard is the *same first
  author's* prior action-scoped verifier (Kang et al., 2026,
  arXiv:2606.29225) — this paper is explicitly the workflow-level
  successor to its own baseline, not a third party's.
- **What persists across the interaction is the state, not the model's
  conversational memory.** "Code, rather than the model's conversational
  memory, owns state persistence." The verifier rejects unknown node IDs,
  filters authorization outputs against an enumerated mutating-tool
  inventory, and persists each request's graph position and evidence
  memory between firings — the same code-owns-state move
  [[concepts/typed-enforcement]]'s sources make for budgets and eviction,
  applied here to procedural position.
- **Two separable ingredients, isolated by ablation.** *What* the policy
  compiles into (graph vs. raw policy text: POLICYGUIDE-RAW) and *who*
  tracks progress (external verifier vs. the acting agent itself:
  POLICYGUIDE-SELF) are independently necessary. Giving the frozen graph
  to the actor and removing the external verifier, persisted state, and
  correction intercept (SELF) does not beat plain ReAct on Mut Pass⁴ in
  any domain — access to the workflow representation buys nothing without
  the external tracking that enforces it.
- **The honest limitation is stated in the paper's own words, not
  extracted by this project:** "each node judgment is probabilistic, so
  compliance is empirical rather than guaranteed... verifier exceptions
  are fail-open, deployments requiring hard guarantees need an additional
  deterministic monitor for the formally expressible policy subset." This
  is new, explicit evidence for [[concepts/typed-enforcement]]'s "escape
  hatch" pattern, from a source that is *not* claiming to be
  deterministic in the first place — the paper positions itself as a
  probabilistic advisory layer, one honest rung below the typed-
  enforcement cluster's Datalog/Rust-type sources, and names the gap to
  the deterministic case explicitly rather than eliding it.

## Methods

Offline: a six-stage pipeline (extract tool specs → derive request
types/shared procedures/ordered subflows → review → generate + schema-
validate subflows → validate schema conformance, tool inventory, graph
composition, edge arity, reachability → prune unused subflows) compiles a
domain policy into a frozen workflow graph once per domain, authored by
GPT 5.4 and reused unchanged across all evaluated agent families. Online:
`Algorithm 1` fires the verifier at every user-turn boundary and once more
after intercepting an unauthorized mutating call; for each open request it
walks the graph from its persisted position, checking each node's
satisfying condition against the interaction history, stopping at the
first unsatisfied node, whose required action becomes the remediation
injected as a guidance message before the agent acts. Evaluated on
τ²-bench airline (50 tasks), retail (114), telecom (114), model-paired
verifier (verifier drawn from the actor's own model family), n=4 trials,
Pass^k reliability metric. Baselines: ReAct (no guard), ToolGuard (static
code, Airline-only), PolicyGuard (the same authors' prior action-scoped
verifier), and — for the closest workflow-aware comparison — FlowAgent
compiled to the same frozen graph via deterministic PDL translation
(no LLM re-authoring), isolating runtime-control differences from
policy-representation differences. Robustness: CRAFT adversarial
red-teaming (persuasive users injecting false eligibility premises).
Procedural compliance: an author-designed Telecom ordered-trace rubric
(Step-TCR / Trace-TCR / process-valid rate), since τ²-bench's own
evaluator scores only final database state plus natural-language
assertions, not intermediate temporal order.

## Results

- **Main (Table 1, Pass⁴):** POLICYGUIDE 0.620/0.614/0.614 (airline/
  retail/telecom) vs. ReAct 0.460/0.596/0.193, PolicyGuard 0.580/0.360/
  0.202. Telecom carries the largest gap (+0.412 over ReAct), consistent
  with its diagnose→instruct→verify chains being the most ordered-
  procedure-dependent of the three domains (54.0% of telecom's policy
  requirements are ordered, vs. 4.7% airline / 3.6% retail per the
  source-policy analysis).
- **Ablations (Table 2):** POLICYGUIDE-SELF's Mut Pass⁴ does not exceed
  ReAct in any domain (e.g. airline 0.154 vs ReAct's 0.192). POLICYGUIDE-
  RAW (external tracking retained, raw text substituted for the compiled
  graph) sits between SELF and full POLICYGUIDE in all three domains —
  the compiled structure and the external tracking each contribute, with
  external tracking being the larger of the two.
- **Matched workflow-controller comparison (Table 3, Telecom test
  split):** Pass⁴ 0.675 (POLICYGUIDE, external graph verifier) vs. 0.350
  (FlowAgent, PDL + API control) vs. 0.325 (PolicyGuard, action-local
  check) vs. 0.250 (ReAct, actor only) — the comparison that isolates
  "external persisted verifier" from "workflow-execution-integrated
  control" on identical graph representation.
- **Cross-agent-family transfer (Table 4):** the GPT-5.4-authored Airline
  graph, reused *unchanged*, still improves Claude Sonnet 4.6 (Mut Pass⁴
  0.500 → 0.577) and Gemini 2.5 Pro (0.231 → 0.462) over each model's own
  ReAct/PolicyGuard baselines. Author-side workflow generalization across
  models/seeds is explicitly *not* established — one frozen GPT-5.4-
  authored graph per domain throughout.
- **Adversarial robustness (CRAFT, Airline, 20 attack tasks):** lowest
  attack-success rate at every k; per-trial ASR 0.087 vs. 0.125
  (PolicyGuard) vs. 0.200 (ReAct) — 91.3% of tested attacks prevented,
  while simultaneously *improving* benign completion rather than trading
  completion for stricter blocking.
- **Procedural trace compliance (Table 5, Telecom):** process-valid rate
  56.2% vs. 17.5% (ReAct) vs. 13.1% (PolicyGuard) — the paper's stated
  "sole end-to-end measure," distinct from the Pass^k statistics above
  because it grades intermediate ordering, not just final state.
- **Cost:** guide calls run ~$0.40/conversation; the paper states this
  directly as unavoidable overhead, reducible with smaller models or
  sparser invocation but not eliminable.

## Critique / open questions

- **The paper's own Corollary 2 undercuts its main-result framing.** The
  evaluated firing schedule (user-turn boundaries + one-shot correction)
  is explicitly *not* the ideal workflow-level schedule the theorem
  proves complete coverage for; deviations that occur mid-turn, after an
  intervening agent event and before the next scheduled firing, are
  outside the stated guarantee. The empirical gains are real, but the
  theory section proves a stronger property than the deployed system
  claims to have.
- **Single frozen policy-author model (GPT 5.4) throughout**, evaluated
  only against itself for workflow faithfulness (manual verification
  against source policy, Appendix E) — no test of graph quality when a
  weaker or different model authors it.
- **Fail-open verifier exceptions** mean an error in the verifier itself
  (not just a missed policy edge case) silently reverts to unguarded
  behavior; the paper does not report an exception rate.
- **CRAFT robustness result is Airline-only** (retail/telecom CRAFT sets
  don't exist in a paper-faithful form per the authors' own account), so
  "POLICYGUIDE is robust to red-teaming" is a one-domain claim generalized
  in the abstract.
- **Code/prompts/schemas not yet released** ("we will release..."), so
  none of the numbers above are independently reproducible yet.
- Relative to the same authors' own PolicyGuard baseline, this reads as
  the second paper in a self-authored progression (action-scoped →
  workflow-scoped) rather than independent replication — worth knowing
  when weighing how much the PolicyGuard-vs-POLICYGUIDE comparison
  counts as a second attestation.

## Trust signals

- **Credibility:** 4 — KAIST + DeepAuto.ai; thorough evaluation design
  (3 domains, 3 agent families, matched-controller comparison, two
  targeted ablations, adversarial red-teaming, a hand-designed
  procedural-trace audit, and a formal intervention-coverage theorem with
  two corollaries that the authors use to bound their own claim rather
  than only to support it). Held below 5: unreviewed preprint, code/
  artifacts not yet released, single policy-author model, and one of the
  three main comparison baselines (PolicyGuard) is the same lab's own
  prior work.

## Follow-up

- **This is new evidence for [[concepts/permission-gate-as-architecture]]'s
  core claim** — "the gate... carries state across turns (accumulating
  intent, drift, and risk)... making the gate part of the agent's control
  loop rather than a wrapper around it" is exactly POLICYGUIDE's
  mechanism, generalized from risk-cumulant scoring (jia2026finharness)
  to graph-position tracking. The matched-controller result (Table 3)
  is the cleanest evidence yet in that concept's source set that
  *external, persisted* state-tracking — not workflow-execution-
  integration (FlowAgent) and not action-local checking (PolicyGuard) —
  is what the architecture buys.
- **For [[concepts/typed-enforcement]], this is a source that argues its
  own limitation rather than one this project has to infer.** The
  concept's "honest limit" section catalogs escape hatches each prior
  source states only about itself; POLICYGUIDE adds a clean fifth
  instance and, unusually, names the exact fix ("an additional
  deterministic monitor for the formally expressible policy subset")
  without claiming to be that monitor. Read alongside
  elkoussy2026agentltl's finding that harsh enforcement responses can
  regress strong models, POLICYGUIDE's design choice — remediate via a
  guidance message rather than hard-block — is evidence for the same
  "gentler enforcement response" lesson, now on the workflow-coverage
  axis rather than the response-to-violation axis.
- **Corollary 2 is worth citing directly if this project ever specs a
  gate's firing schedule**: "evaluated at turn boundaries" is a common,
  cheap design and this paper gives the formal reason it is incomplete —
  a distinction worth keeping in mind for any future `/lint`-style static
  check that fires on a similar cadence rather than continuously.
- Not proposed as a concept seed: the workflow-graph-as-persisted-state
  mechanism is a specific instance of permission-gate-as-architecture's
  general claim, not a new axis of its own.
