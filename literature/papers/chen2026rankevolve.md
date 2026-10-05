---
kind: paper
title: "RankEvolve: A Reliable Multi-Agent Auto-Research Harness for Evolving Ranking Models"
authors: ["Zheng Chen", "Linfeng Liu", "Hong Li", "Hong Yan"]
institutions: ["Meta"]
year: 2026
venue: "arXiv 2609.39551v1 (cs.AI), preprint, 30 Sep 2026 (PDF dated August 1, 2026)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.39551"
code_url: null   # failure-overlap analyzer, leaderboard schema and boost-last-K patch "will be released with the codebase"; no repository named
citations: null
source: "raw/papers/chen2026rankevolve.pdf"
added: "2026-10-05"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["autoresearch", "recsys", "hstu", "meta", "cross-vendor-review", "claude-code", "codex", "error-correlation", "matched-budget", "execution-accuracy", "silent-defects", "state-machine", "runtime-enforcement", "context-scoping", "leakage", "oracle-benchmark", "pre-specified"]
---

# RankEvolve: A Reliable Multi-Agent Auto-Research Harness for Evolving Ranking Models

## TL;DR

A Meta auto-research harness ran twelve iterations of code-level evolution
on the public HSTU generative recommender. It reached NDCG@10 **0.2192**
on MovieLens-20M LARGE (+4.48% over the published 0.2098 anchor), and the
largest gain came from adding a genre side-feature, not from a new
architecture. The paper's real contribution is **ExecML**, an
incident-seeded benchmark with a hidden oracle: 96 private tasks on HSTU
and 96 on LitGPT. On it, an implement→review→repair flow that crosses
products (Claude Code→Codex→Claude Code) beats the **same flow run
entirely in Claude Code** at a matched observable budget. All-oracle
execution accuracy (EA) is 62.5 vs 45.8, a paired +16.7 (CI [6.6, 26.7]).
The silent critical-defect rate falls from 16.7% to 10.4%. Error
correlation between independent patches is ρ = 0.21 across products and
0.58 within Claude Code, at nearly equal marginal failure rates. For
[[concepts/shared-substrate-contagion]], this is the first matched-budget,
capability-matched measurement in the graph showing that a reviewer from a
different vendor buys independence a same-product reviewer does not.
Model and product heterogeneity are confounded (Opus 4.8 inside CC, GPT-5.6
inside Codex), and the baseline family contains no Codex-only arms. The
other headline component, runtime enforcement of the Executable Operating
Protocol (EOP), is **never tested against a no-runtime arm**. Its only
controlled ablation holds the runtime fixed and varies prompt scope.

## Claims

- **Execution accuracy binds.** "Over such long horizons, execution
  accuracy is a binding constraint in our setting: a change can silently
  leak held-out data, omit a layer norm, disconnect a gradient, or leave a
  train/eval flag unwired."
- **The runtime controls the process; the model only performs the steps.**
  "the model performs each step, but the framework controls the process."
  The enforcement contract (Table 7) states the boundary: "runtime control
  constrains admissible transitions but cannot make an agent's scientific
  reasoning or code correct."
- **Different products, not more budget.** "Extra same-product budget helps
  (+10.4 extended; +12.5 homogeneous review) but plateaus below the pair:
  buying different products, not more of the same, delivers the largest
  increment."
- **The mechanism is complementary error, not diversity as such.** "The
  mechanism is thus not 'diversity is good' but complementary error times
  conversion efficiency." Decorrelation slope β̂1 = 0.34, CI [0.12, 0.56].
- **Topology does not matter at fixed budget.** "pairing two different
  products, not the specific topology, is what buys correctness." The
  deployed plan-merge flow ties the simple pair when capped to the same
  budget (paired 0.0, CI [−5.5, 5.5]).
- **Per-step injection beats full-protocol injection, and the gap grows
  with the horizon.** "Because the state machine, not the prompt, carries
  the process, full-protocol injection mainly adds distracting inactive
  instructions the runtime already enforces."
- **The claims are confirmatory.** "a falsifiable protocol fixed before the
  private runs, so the reported effects are confirmatory rather than
  post-hoc." No external registry is cited.
- **The agent is weakest at novelty.** "a contemporary auto-research agent
  is most reliable at disciplined execution, failure diagnosis, and
  pragmatic information-adding, and least reliable at the modeling novelty
  we most want from it."

## Methods

- **Three layers.**
  - *EOP (authoring).* A semi-structured markdown playbook with
    `depends on`, `branch`, `goto … if`, `requires user input` and
    `Tools [must]` markers. The full text is in Appendix D. It compiles to
    a state machine with state σ = (phase, variables, artifacts, gate
    status, branches, attempt/budget counters, append-only trace), advanced
    by typed events. By default the runtime injects the global preamble
    plus the active phase body only.
  - *Meta-meta-harness (control).* Work nodes bind whole coding-agent
    products (Claude Code, Codex, OpenHands) through an adapter contract:
    task, snapshot, allowed tools and budget go in; patch, status, trace and
    usage come out. Routing, aggregation and gates remain deterministic.
    The patterns are Linear, Dual (propose→review→fix until approved or
    severity < τ, ≤K rounds), Breakdown-then-Aggregate, and
    Plan-then-Implement. The deployed flow is human-authored YAML (App. F),
    and its file defaults bind *every* node to Claude Code (`opus[1m]`).
    Only the two planning lanes are rebound to CC and Codex.
  - *Knowledge layer.* Notes are committed beside the code they describe.
    A central wiki cites those notes and is declared the single source of
    truth. An entity graph spans both tiers. A reconciliation agent demotes
    and back-links superseded notes and escalates undecidable conflicts to a
    human. Retrieval is selective per phase. There is **no memory on/off
    ablation**.
- **HSTU deployment.** The deployment ran twelve iterations in three
  stages, with ~55 leaderboard entries across 49 experiment directories on
  1–8 H200s. A human operator sat at the gates and also steered: they
  vetoed an over-budget run and gave three directives ("favor new methods
  over stacks of add-ons", compare like with like, no test leakage). All
  numbers are best checkpoints from one run, not seeds. Canonical full-test
  NDCG@10 is kept separate from a subset evaluation that "ran ≈0.005–0.010
  higher".
- **ExecML (the oracle benchmark).**
  - Each repository has 96 private tasks and 32 development tasks
    (Table 8). Half are implementation and half repair, balanced over six
    incident families: data provenance/leakage, tensor routing, gradient
    flow, train/eval mode, metric semantics, and configuration wiring.
  - HSTU tasks are "seeded by the case study's logged incidents". The paper
    does not say how the LitGPT tasks were sourced, beyond the same six
    families.
  - The oracle passes only if all four checks pass: regression suite ∧
    behavioral checks ∧ scientific-safety invariants ∧ evaluator-integrity.
    The curator reference patch must pass, and removing the intended
    mechanism must fail at least one semantic check. A second curator
    validates each task. Hidden tests, reference patches, git history and
    network are kept out of the sandbox, and evaluator files are read-only
    and hash-checked. Mutation testing plus a stratified human audit
    estimate missed defects.
  - Silent critical-defect rate (CDR): the patch runs, but the defect can
    invalidate a scientific conclusion.
- **Is it really heterogeneous vs homogeneous composition at matched
  budget? Mostly yes, with two holes.**
  - Six stage-1 conditions, randomized order, identical snapshot, tools,
    network, prompt and timeout:
    1. one-pass CC (deliberately unmatched, cheap reference)
    2. one continuous CC session given the whole budget B
    3. CC best-of-N with oracle-blind selection and repair
    4. **CC→CC→CC** implement–review–repair
    5. **CC→Codex→CC** (same topology)
    6. Codex→CC→Codex
  - Conditions 2–6 share envelope B, frozen on development tasks: same
    three roles, per-node limits, tool permissions, action caps, wall-clock
    and spend caps. Timeouts and overruns count as failures. Realized cost
    is $0.95–1.02 per task and 90–109 tool actions. The paper says
    "matched observable inference budget", conceding that provider-side
    FLOPs cannot be matched.
  - So the primary contrast (5 vs 4) is heterogeneous vs homogeneous review
    at the same topology and budget, not composition vs a single call. The
    abstract calls 45.8% the "best single-product" number. That means
    single-*product* composition, not a single call.
  - **Hole 1: the baseline family is all Claude Code.** There is no
    Codex one-pass, no Codex extended-budget and no Codex→Codex→Codex row
    in Table 4. "Beats every budget-matched single-product baseline" means
    every *CC* baseline. Table 9 gives Codex a slightly *worse*
    independent critical-failure rate (0.58 vs 0.54), so a
    Codex-homogeneous flow probably would not have won, but it was not
    run.
  - **Hole 2: product and model are confounded.** CC runs Claude Opus 4.8
    and Codex runs GPT-5.6, both at maximum effort. No arm crosses harness
    and model (Codex on Claude, or CC on GPT), so "different products"
    cannot be separated from "different model vendors".
- **Statistics.** The primary test is a single max-over-B contrast. Per-baseline
  contrasts are Holm-corrected secondaries (not reported individually). The
  tests are exact McNemar or paired randomization, with bootstrap intervals
  clustered on related source incidents. A smallest effect of interest was
  frozen after a development-only power analysis, and **its value is never
  stated**.
- **Mechanism (RQ3).** CC, Codex and OpenHands each produce independent
  replicate patches *before any product sees another's*, which gives
  marginals p and phi correlation ρ. The paper defines
  D_{a←b} = P(E_a=1, E_b=0), the error mass available for rescue, and
  G_{a←b}, the realized rescue minus the harm the reviewer introduces. It
  fits G ~ D with repository and fault-family random effects. The
  pre-registered criterion is *both* a slope CI that excludes zero *and*
  held-out prediction beating a marginals-only model.
- **Context-scope ablation (RQ4).** At each scored phase, the same runtime
  checkpoint is forked into two sandboxes. One prompt is G‖H‖A_i; the other
  is G‖H‖A_{−i}‖A_i, with the active body repeated last to control recency.
  Runtime, state, tools, budgets, scripted gate responses and product are
  all identical. A length-matched neutral-padding arm and a whole-episode
  randomization are described in the design.

## Results

- **ExecML stage 1 (Table 4, n = 96 tasks per repository). Rows are
  HSTU EA / HSTU CDR / LitGPT EA / LitGPT CDR / $ per task.**
  - CC one-pass: 22.9 / 35.4 / 20.8 / 37.5 / $0.41
  - CC extended budget: 33.3 / 27.1 / 29.2 / 29.2 / $0.98
  - CC best-of-N, blind selection: 43.8 / 18.8 / 39.6 / 20.8 / $0.95
  - CC→CC→CC: 45.8 / 16.7 / 43.8 / 18.8 / $0.97
  - **CC→Codex→CC: 62.5 / 10.4 / 56.2 / 12.5 / $1.02**
  - Codex→CC→Codex: 56.2 / 12.5 / 50.0 / 14.6 / $1.00
- **Primary contrast.** +16.7 on HSTU, CI [6.6, 26.7], p < 0.001. There are
  20 discordant tasks, 18 rescued and 2 broken. Our check: exact two-sided
  McNemar on 18/2 gives p ≈ 0.0004, which matches. The LitGPT transfer is
  +12.5, CI [3.0, 22.0], p = 0.008. Order matters: the reversed pair trails
  by 6.2.
- **Stage 2, the deployed plan-merge topology, specified after
  unblinding.**
  - At natural cost ($2.40 per task, 250 actions, ≈2.4× the envelope) it
    reaches EA 70.8 and CDR 6.2. Against a re-run comparator the gain is
    +8.3, with 12 tasks rescued and 4 regressed, p = 0.077 (our McNemar
    check agrees), CI [−0.7, 17.3]. That is not significant.
  - Capped to B, it scores 62.5, a tie with the pair.
  - An equal-spend three-role control at $2.40 scores 66.7.
  - The extra 8 points therefore come from spending more, not from the
    topology. The authors justify the spend as worth ~10³× its cost in
    averted GPU waste. That figure is asserted, not measured.
- **Mechanism (Table 9). Rows are p_a / p_b / ρ / D / G / rescue–harm.**
  - CC←CC: 0.54 / 0.54 / **0.58** / 0.104 / **−0.01** / 0.04–0.05
  - CC←Codex (HSTU): 0.54 / 0.58 / **0.21** / 0.175 / **0.061** /
    0.09–0.03
  - Codex←Codex: ρ 0.55, G 0.004
  - Our recomputation of D from p and ρ reproduces the CC rows exactly.
  - The marginals are near-equal, so this is close to a
    capability-matched comparison. It avoids the
    [[literature/papers/kim2026are]] trap, where disagreement statistics
    turn out to be accuracy gauges.
  - Slope β̂1 = 0.34, CI [0.12, 0.56].
- **Context scope (Table 11). Per-step minus full-protocol, in EA
  points.** Early +2.1 (64.6 vs 62.5), middle +6.2, late **+10.4**
  (52.1 vs 41.7). CDR −6.2, wrong promotion −8.3, input tokens 4.9k vs
  9.8k, cost $0.71 vs $1.28.
- **Process-drift pilot (Table 10, Figure 2).** Ten runs received the
  whole procedure as one in-context playbook with no runtime. **0/10 were
  fully faithful.** All 10 ran only a subset of the selected proposals,
  4 stopped early despite recommending to continue, 4 skipped a gate, and
  3 called tools off-spec. Violations "proliferate from iteration 2". This
  is an uncontrolled historical pilot, and the authors call it that.
- **HSTU deployment.**
  - ML-20M: LARGE 0.2192 (+4.48%), BASE 0.1948 (+2.80%). The ML-32M rows
    are against *internal* reproductions, not published anchors.
  - Stage I (compose SSD-style compression + FiLM conditioning +
    multi-token head) reached 0.2161 (+3.0%). Stage II was "mostly within
    noise".
  - The 0.2192 comes from an additive genre side-feature fine-tuned from
    the 0.2140 stack. Genre, year and popularity channels all saturate at
    ≈0.219.
  - Recipe transfer to Foursquare, Gowalla and Yelp gained +5.9 to +25%,
    one run per cell.
- **Leakage incidents inside the deployment.**
  - The genre feature's v1 put genre into the loss-side embedding of
    positives only. Training loss collapsed to zero and eval spiked to
    0.2163.
  - DPO used target-derived negatives and collapsed NDCG by −35.9%. The
    leak-free re-derivation gave Δ ≈ 0.
  - Naive multi-position eval inflated 0.2192 to 0.2517/0.2577.
  - The paper states that the genre and DPO bugs "surfaced not from a
    failed unit test but as anomalous metric trajectories" read off the
    persistent leaderboard.
  - Subset-vs-full eval separation was "the study's most important
    methodological discipline: without it, the system repeatedly chased
    phantom 0.005-scale 'gains'".

## Critique / open questions

- **The digest's EOP claim is not tested by this paper's controlled
  evidence.** "the runtime enforces the compiled state machine" is the
  architectural headline. But no arm compares runtime enforcement with no
  runtime enforcement. The pilot "lacks a matched EOP-runtime arm and
  changes control and context delivery together". RQ4 holds the runtime
  fixed *in both arms* and varies only prompt scope. So the paper shows:
  *given* a runtime, less prompt is better. It does not show that the
  runtime buys faithfulness or correctness. Process violations in RQ4 are
  "only a manipulation check", identical across arms by construction.
- **RQ4 has no inferential statistics.** Table 11 gives point estimates
  only: no intervals, no p-values, and no per-stratum n. The design
  promises two further results, and neither appears in the paper:
  - The length-matched padding arm, which would separate interference
    from length. **No result is reported.**
  - The "whole-episode randomization (full table in Appendix C)". Appendix
    C contains no such table.
  The abstract presents this ablation alongside the CI-bearing results.
- **Half of the decorrelation criterion is unreported.** The rule requires
  the slope interval to exclude zero *and* held-out prediction to beat a
  marginals-only model. The text says "the slope condition holds" and is
  silent on the second. The Conclusion nonetheless says "the decorrelation
  condition is met".
- **Tables 4 and 9 do not reconcile.** Table 9 says homogeneous CC←CC
  review nets **G = −0.01**: it rescues 4% of critical defects and
  introduces 5%. Table 4 says CC→CC→CC lifts EA from 22.9 (one-pass) and
  33.3 (extended) to 45.8, and cuts CDR from 35.4 and 27.1 to 16.7. Both
  can be true only if Table 9's "independent pre-review patch" baseline
  differs from Table 4's arms, for example in per-node budget. The paper
  does not explain the gap. Table 9's p_CC = 0.54 critical failure also
  sits oddly beside one-pass CC's 77% all-oracle failure. "Critical" vs
  "all-oracle" may cover it, but neither is broken down.
- **The benchmark may be selected toward Claude Code's failures.** HSTU
  tasks are seeded from incidents logged in a deployment whose YAML
  defaults bind every node to Claude Code. If the seed incidents are
  disproportionately CC-authored defects, a non-CC reviewer is favored by
  construction. The LitGPT replication (+12.5) partly answers this, but
  its task provenance is not described.
- **Model and product heterogeneity are confounded, and that matters for
  [[concepts/shared-substrate-contagion]].** The result shows that
  (Opus 4.8 + CC harness) and (GPT-5.6 + Codex harness) err less
  correlatedly than two CC runs. It cannot say whether the harness, the
  weights or the vendor's training data buys the independence. That makes
  it complementary to [[literature/papers/zheng2026engineering]] (a model
  swap alone gave 11.3 pp against 40.9 pp for an independent data source),
  not a refutation of it. Here the reviewer reads the *same* repository
  and the *same* patch, so its evidence path is not independent. Yet it
  still rescues. The rescue must come from the reviewer's priors, not
  from new evidence.
- **"Pre-specified" rests on the authors' word.** There is no
  registration link, and the smallest effect of interest and the Holm
  secondary results are not reported. The stage-2 null and the
  "reported whatever its outcome" LitGPT framing are honest moves. Still,
  "confirmatory" here means "the authors say they froze it".
- **A numeric oddity, not proof of anything.** Every one of the 30
  percentage cells in Table 4 is an *even* count out of 96, a multiple of
  1/48 (for example 22/96, 44/96, 60/96, 10/96). Under independence that
  is roughly 2⁻³⁰. Table 11's CDR cells (11/96, 17/96) are odd, so the
  grid is specific to Table 4. Possible benign causes include paired
  attempt averaging or an effective n of 48. The paper does not explain
  it, and an artifact release would.
- **Composition did not stop leakage in the deployment.** The two worst
  incidents (the genre-v1 leak and the DPO target-derived negatives)
  passed through whatever review the deployment used and were caught
  downstream by the metric trajectory on the leaderboard. Reviews did
  not catch them. The ExecML numbers are single-patch, oracle-scored and
  human-free. The claim that composition "makes long-horizon iteration
  reliable" is an extrapolation from single patches to the loop.
- **The deployment result is weak as an ML result.** It is one run with
  best checkpoints, a human steering at the gates, and a win that adds
  metadata the anchor did not use. Novel methods "did not finish within
  the study". The authors are candid about all of this (App. I.2: "no
  genuine architectural novelty").
- **The name collides with an existing system.** The authors footnote
  Nian et al. 2026 (arXiv 2602.16932), a different RankEvolve that evolves
  retrieval algorithms.

## Trust signals

- **Credibility:** 3. In favor:
  - Meta authors with real H200 deployment logs.
  - A carefully specified oracle and budget-matching protocol, with
    failures kept in the denominators.
  - McNemar values that we re-derived and that match.
  - Table 9's D algebra reproduces.
  - Honest reporting: a non-significant stage-2 result, an explicit
    "Not guaranteed" column, and stated limitations (no memory ablation,
    human operator confound, single-run transfer).

  Held at 3 because:
  - Not peer-reviewed, no code or benchmark released, no external
    preregistration.
  - Half of the decorrelation criterion is unreported.
  - The RQ4 ablation has no intervals, and its promised length-matched and
    whole-episode results are missing.
  - The Table 4/Table 9 tension is unreconciled.
  - The baseline family is CC-only.
  - Table 4 has an unexplained 1/48 grid.

## Follow-up

- **Relevance:** 4. It is material new evidence for
  [[concepts/shared-substrate-contagion]]: the first matched-budget,
  same-topology, near-capability-matched comparison of heterogeneous and
  homogeneous review, with **no-channel** error correlations measured
  before review (independent replicate patches). Same-product review sits
  at ρ ≈ 0.55–0.58 with near-zero net rescue. Cross-vendor review sits at
  ρ ≈ 0.21–0.26 with positive net rescue. It also adds a controlled
  prompt-scope result to [[concepts/enforcement-boundary-placement]] and
  [[concepts/typed-enforcement]], and an explicit enforced/not-guaranteed
  contract. It is not a 5: it is an unreleased preprint with unreported
  pre-specified analyses, and the enforcement claim most relevant to this
  repo (runtime vs prompt) is untested.
- **Implication for this repo's review path.** The ingest→concept pipeline
  here is homogeneous: Opus writes, and Opus re-reads if anything does.
  The ρ = 0.58 figure for same-product independent patches is the
  number to beat. A cross-vendor re-read (Codex or GPT on the concept
  diff) is the cheapest instance of the "source or model independence" fix
  that shao2026language says correlated priors need. Per zheng2026engineering,
  an independent *evidence path* (re-reading the raw PDF without the
  literature note) should still beat a vendor swap.
- **Implication for EOP-like enforcement.** The pilot's failure list
  matches [[literature/papers/nepal2026faithful]] and
  [[literature/papers/chandran2026autoresearch]]'s "memory decay": present
  procedural rules ignored as context grows. That is evidence for moving
  phase/gate order out of the prompt. But the paper supplies the
  motivation, not the measurement. The RQ4 horizon pattern (+2.1 → +10.4)
  supports injecting only the active phase *when a runtime already holds
  the order*. That bears on skills that load a whole multi-phase procedure
  at once.
- **Bearing on [[concepts/hce-evaluation]].** ExecML's oracle design is a
  worked instance of "plant the failure so you do not have to adjudicate
  it". Each task is a planted incident family with a mechanism-removal
  check against vacuous tests, plus mutation testing to estimate false
  passes. The deployment's lesson that subset vs full eval produces
  "phantom 0.005-scale gains" is another field datum for noise-band
  discipline, alongside chandran2026autoresearch.
- **Candidates:**
  - Wang et al. 2026, "Self-evolving recommendation system" (arXiv
    2602.10226, YouTube).
  - Kim et al. 2026, Self-EvolveRec (arXiv 2602.12612).
  - Yao et al. 2026, Harness-Bench (cited as showing "the harness, not
    just the model, determines outcomes"; no arXiv id given).
  - Meta's Ranking Engineer Agent blog post (Kumar et al. 2026).
  - StateFlow (Wu et al. 2024) as the prior art for state-specific
    prompting.
