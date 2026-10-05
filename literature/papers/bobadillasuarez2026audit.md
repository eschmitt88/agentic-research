---
kind: paper
title: "Audit the Scaffold, Not the Checkpoint: A Stationarity Dichotomy for Recursive Self-Improvement in Agentic Coding"
authors: ["Sebastian Bobadilla-Suarez", "Bob Suh", "Ryan Fortin"]
institutions: ["OnCorps"]
year: 2026
venue: "arXiv 2609.34924 (cs.LG) preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.34924"
code_url: null
citations: null
source: "raw/papers/bobadillasuarez2026audit.pdf"
added: "2026-10-05"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
tags: ["recursive-self-improvement", "scaffold-expansion", "boosting", "diminishing-returns", "refinement-loop", "best-of-k", "majority-vote", "correlated-errors", "same-family-pool", "swe-bench-lite", "idempotence", "production-telemetry", "churn", "lean-formalization", "governance", "evaluator-audit"]
---

# Audit the Scaffold, Not the Checkpoint: A Stationarity Dichotomy for Recursive Self-Improvement in Agentic Coding

## TL;DR

Three authors from OnCorps, a small enterprise-software company, read
iterative code refinement as boosting on the residual (the patch). From that
reading they derive a **bounded-monotone-sequence** result. If quality is
bounded and the reachable set of edits is fixed, per-round gains must go to
zero. Escape needs that set to keep growing, and a scaffold rewrite can grow
it while the weights stay frozen. Hence the governance slogan "audit the
scaffold, not the checkpoint."

The theory is close to tautological, and the authors say so ("The proofs are
elementary … and that is the point"). The value is in two honest,
modest measurements:
- **Part B.** A 1,650-trajectory SWE-bench Lite refinement loop (55 tasks,
  T = 4 rounds). Refinement **goes idempotent rather than wrong**: 80.7%
  (Haiku) and 69.1% (Sonnet) of trajectories never change after round 0, and
  at most 2.9% of rounds lower quality.
- **Part C.** 401 production sessions at the authors' own company show
  geometric churn decay. A pre-AI human baseline shows the same shape, so it
  identifies nothing about mechanism.

The digest's "majority fails 23/55 (42%)" is real but is mostly a
**tier-pooling artifact**, as the body concedes. Within a single tier the vote
equals the average worker almost exactly.

## Claims

- **The dichotomy.** "iterative self-modification hits strict diminishing
  returns whenever the agent's reachable set of edits stays fixed, and can
  escape only if that set expands." Body: "a criterion, not a convergence
  rate."
- **Frozen weights are the wrong invariant.** "frozen weights buy an eventual
  ceiling but no stationarity along the way." "An auditor who checks only
  whether weights are frozen catches test-time training and misses scaffold
  expansion entirely."
- **Width buys rate, not budget.** "Best-of-k selection attains V*_orch =
  max_i V*(W, C_i) exactly—an unconditional equality." Orchestration "selects
  which existing ceiling to realize and reaches it sooner; it does not create
  a higher one."
- **The vote needs diversity the pool lacks.** "on 30 same-family workers the
  failure overlap sits at its maximum, and a majority fails 23/55 (42%) of
  tasks." The body qualifies this: "same-family workers, so this settles only
  the pessimistic half."
- **A reused pool has a horizon.** Either some nonnegative combination of the
  pool already fits the work, or "the edge hypothesis fails by round
  log(1/ϵ0)/2γ²". This holds only under AdaBoost reweighting, which, as the
  paper says, "no deployed loop maintains".
- **The mechanism is not identified.** "What we measure is saturation … a
  shape shared with a pre-AI human baseline that establishes the regime
  without identifying its cause."
- **Practitioner guidance** (§6, G.3), stated as "hypotheses a practitioner
  can cheaply test":
  - "spend on the first attempt, not the refinement loop".
  - "reach for capability before orchestration".
  - "When a loop has stalled, ask which kind of change you are making: only
    the second kind [new reach: tools, retrieval, ability to run tests,
    decomposition] can move a hard plateau."

## Methods

**Theory (§3–4, Apps. A–C).** Lean 4 checks every result, with no axiom and
no `sorry`. The authors say the check only "verifies the algebra, not the
modeling assumptions". They also disclose that the manuscript was
"substantially drafted with AI assistance" and that Lean is used as "a
quality-control layer against AI-hallucinated derivations".
- **Thm. 2, the refinement game.** If each round improves expected quality by
  η_t > 0 and V ≤ 1, then Σ η_t ≤ 1 − V(z0). This is a telescoping sum.
- **Prop. 3, the dichotomy.**
  - (i) A fixed bound B forces η_t → 0.
  - (ii) A uniform edge η_t ≥ c makes E[V] unbounded (Archimedean property).
- **Cor. 4.** The system is S = (W, C), with frozen weights and a mutable
  scaffold. Two stated assumptions:
  - (M1) the reachable class H_t(C) can expand under scaffold rewrites with
    W fixed.
  - (M2) an ultimate ceiling V*(W) holds over any constructible scaffold.

  Neither is measured.
- **Prop. 5 (best-of-k).** "sup and finite max commute." Lean refuted the
  authors' own conjectured strict inequality.
- **Prop. 6.** An unweighted majority vote has zero training error iff every
  task is failed by fewer than k/2 workers (m* < k/2). The paper calls this
  "bookkeeping".
- **Prop. 7, Props. 19–21.** These cover the horizon for reusing a pool under
  AdaBoost.
- **What the theory does not do.** The classical AdaBoost results transfer
  "verbatim", but the paper "rel[ies] on none". Deployed loops compose patches
  (R_T ∘ … ∘ R_1) rather than retaining and voting on them, and they keep no
  distribution over specs.

**Part A (App. D).** Decision stumps on synthetic labels over SWE-bench
TF-IDF features. It is only an implementation sanity check and "claim[s]
nothing about code or LLM behavior".

**Part B: the source of the 30 workers and 55 tasks.**
- **Loop.** mini-swe-agent is the per-round learner inside a T = 4 refinement
  loop. T was chosen "from pilot evidence that the edge collapses by round 1".
- **Tasks.** K = 55 SWE-bench Lite tasks from 10 repos. Django and SymPy are
  excluded because the authors' own evaluator is pytest-only.
- **Design.** A 2 × 3 grid (1,650 trajectories) × 5 seeds at temperature
  0.25:
  - Tier: "Claude Sonnet" (version not stated) vs Claude Haiku 4.5.
  - Feedback mode: independent (no memory), blind (prior patch only), or
    diagnostic (prior patch plus the still-failing FAIL_TO_PASS tests).
- **The 30 "workers"** are those 6 arms × 5 seeds, all scored on the same 55
  tasks.
- **"Same-family"** means one vendor (Anthropic), two capability tiers, three
  prompt variants and low-temperature reseeding. The authors judge the pool's
  effective diversity "nearer two configurations than 6", and call it "about
  as weak a diversity test as one could field".
- **Scoring.** A digest-pinned Docker harness reads `--junit-xml`. "Solved"
  means at least τ = 0.6 of tests passing (τ = 1.0 also reported), so these
  are **not** SWE-bench resolve rates.
- **Stop-on-pass.** The loop stops calling the model once a task passes and
  back-fills the remaining rounds with q = 1.
- **Evaluator audit.** Applying gold patches to the first 50-task pool showed
  **23/50 (46%) failing deterministically for evaluator reasons**:
  - wrong runner for Django/SymPy (9);
  - an unsupported `--no-header` flag (3);
  - ANSI colour codes breaking the parser (3);
  - parametrize-ID drift (7);
  - a network-dependent test (1).

  These were fixed or excluded, 15 gold-verified tasks were backfilled, and
  the pool was rebuilt to K = 55.

**Part C: the source of the 401 sessions.**
- **Data.** Structural git metrics only (diff sizes, commit counts,
  model trailer) from the authors' employer, "an enterprise software
  company". The acknowledgments name OnCorps production code.
- **Sessions.** A session is a run of consecutive Claude-co-authored commits
  with gaps under 4 h. That gives 401 sessions, 1,211 commits and 23 repos.
  50.9% of sessions are a single commit.
- **Controls.**
  - A pre-AI human baseline from a disjoint cohort of 122 older repos
    (2021-11 to 2022-09): 9,395 sessions.
  - A within-era AI vs non-AI contrast: 487 vs 1,874 sessions over the same
    developers.
- **Metrics.** Churn trajectory per session, the late/early churn ratio, the
  survivor ratio ψ = |net lines| / total churn, and a fitted geometric decay
  factor ρ.

## Results

- **Capability dominates, and feedback mode is an underpowered null.**
  - RM-ANOVA: tier F(1,54) = 53.46, partial η² = 0.497; feedback mode
    p = 0.61; interaction p = 0.45.
  - Solve rates at τ = 0.6: Haiku 44–49%, Sonnet 85–89%. At τ = 1.0: Haiku
    29–38%, Sonnet 71–75%.
  - Power for the mode effect is 11% (17% for the interaction). 80% power
    needs K ≈ 364–467.
  - Seed-to-seed SD within a cell is ≈0.096, about 1.7× the between-mode
    signal (≈0.057).
- **Refinement saturates by going idempotent.**
  - Mean per-round improvement on the running best: Haiku 0.097 → 0.036 →
    0.033; Sonnet 0.156 → 0.068 → 0.029.
  - Tie rates on the diagnostic arm run 80–95% per round. Net of back-fill,
    genuine no-ops are 52–59% (Haiku) and 13–21% (Sonnet). Regressions are at
    most 2.9%.
  - The per-round edge on live cells only is negative for Haiku throughout
    (−0.345, −0.435, −0.411). For Sonnet (−0.084, −0.143, −0.276) the first
    two intervals include zero.
  - The pooled edge is an artifact: back-filled cells score as "failed to
    improve", which makes the stronger tier look worse.
- **The overlap and vote numbers behind the 42% (App. C).**
  - Some task is failed by all 30 workers (m* = 30 = k) at every round.
  - 23/55 tasks have 2m_i ≥ k, so vote error is 0.418 [0.291, 0.545].
  - Mean per-worker error is 0.356. The paired gap is +0.062
    [−0.018, +0.142].
  - **Within a single tier**, m* = k persists, but the vote matches the
    average worker: Sonnet 0.145 vs 0.147, Haiku 0.545 vs 0.565.
  - The paper concludes the excess "is a pooling effect rather than a fact
    about voting".
  - The headline is the same at k = 6 (one worker per arm; 28/55 violate) and
    at τ = 1.0.
- **Production churn converges.**
  - 70.0% of multi-commit sessions show decreasing churn. The median
    late/early ratio is 0.49 [0.400, 0.793].
  - The commit-order permutation test gives p < 0.0001, on the n = 70
    sessions with at least 5 commits.
  - Decay factor ρ ≈ 0.77 [0.65, 0.92] for AI sessions vs 0.86 [0.82, 0.91]
    for the human baseline. **The shape is the same.**
  - The fitted decay model fails held-out KS in 6 of 8 seeds.
- **Opus and Sonnet do not differ on survivor ratio.**
  - ψ ≈ 0.55 for both; gap −0.003 [−0.075, +0.069].
  - The null survives four de-confounding checks and a version-resolved
    trend test, but it is "a bounded null". The interval still admits a
    7-step swing of up to 0.155.
  - The authors read it as the proxy failing, not the models being equal.
    ψ separates *whether* an agent wrote the code (human 0.39, a gap that
    survives random effects) but not *which* agent.

## Critique / open questions

- **The theorem mostly restates its assumptions.**
  - Thm. 2 and Prop. 3(i): a bounded, monotone sequence has summable
    increments.
  - Prop. 3(ii): sustained positive increments diverge. Under Thm. 2's own
    V ≤ 1 this case is vacuous, and G.5 concedes that boundedness "is the
    load-bearing assumption".
  - The step that matters, "fixed reachable edit set ⇒ fixed ceiling B < 1",
    is not a theorem. It is (M2), and (M1) and (M2) are both unmeasured.
  - The criterion runs one way only. Expansion is "necessary but not
    sufficient", so the diagnostic "over-flags by construction".
  - The paper admits it lacks an estimator for the reachable class and that
    the reasoning/tool-creation boundary "is not operationally sharp".
  - So "audit the scaffold" is a useful reframing of the governance question,
    not a derived result.
- **Prop. 5 is definitional.** The orchestrator's reachable class is defined
  as the union of the workers' classes, and the ceilings are sups over
  infinite attempts. Hard selection also presumes an oracle V for picking
  outputs.
  - The finite-sample point that matters for [[concepts/pass-at-k]] is
    untouched. Coverage at a given k still grows with width (Brown et al.).
  - "Width buys rate" is the honest content: the per-round max of edges
    dominates any one worker. The paper also separates oracle coverage from
    verifier-free voting.
- **m* = k is a weak diversity statistic.**
  - Prop. 6's condition is for *zero* training error. Any real pool fails it
    as soon as one task is beyond every member, and SWE-bench has such tasks
    for every model family.
  - Against an independent-errors null (Sonnet ε ≈ 0.15; 0.15¹⁵ ≈ 0), m* = k
    does show correlated failure. But per-task difficulty heterogeneity is
    the trivial explanation, and it would also bind a cross-family pool.
  - The more useful number is vote error against the average or best
    worker within a tier, and there the vote is neutral.
  - "Same-family" here is narrower than in kim2026are's 30-model pool. It is
    one vendor resampled at temperature 0.25 with prompt tweaks. Seeds are
    near-replicates, so the pool is ~2 effective configurations, not 30.
- **The digest's 42% is mostly Haiku.** With 15 Haiku and 15 Sonnet workers,
  a task that every Haiku worker fails already gives 2m_i ≥ k. The figure is
  the weak tier's failure rate diluted by pooling, and App. C says as much.
  Read it as "pooling a weak tier poisons a majority vote", not as a
  same-family correlation magnitude.
- **The decay in Part B is short and survivorship-shaped.**
  - The series has 3 transitions (T = 4).
  - Stop-on-pass removes solved tasks, so later rounds are a smaller, harder
    population (Sonnet 113 → 49 live rows).
  - The authors flag this as their critical caveat and call for a re-run that
    "declines to stop on success" (open problem 3).
  - The decline is still consistent with saturation. But it cannot
    distinguish a fixed reachable class from "the easy residuals are gone".
- **Part C is not evidence for the theory, and the paper says so.** Geometric
  front-loading of churn is how editing works (human ρ = 0.86). The one
  theory-specific discriminator, Opus vs Sonnet on ψ, is null. The
  observational tier assignment is confounded with task type, because Opus
  is used for planning. The data come from the authors' own employer.
- **The decisive experiment is not run.** "Hold weights fixed and vary the
  scaffold" (open problem 4) would test the paper's actual claim. All three
  feedback modes are within-class variants by the paper's own account
  (G.3 item 4), so the null on them is predicted, not tested.
- **The evaluator audit is the most transferable piece.** A home-built
  per-round evaluator wrongly failed 46% of gold patches before audit. Every
  one of those failures would have been scored as a model error. This is a
  direct instance of [[concepts/programmable-evaluator-oracle]] and
  [[concepts/hce-evaluation]] discipline: verify the scorer against
  known-correct answers before trusting it.
- **"Computable in advance" is never computed.** Prop. 7's horizon needs
  ϵ0 (the best cone-combination error) and γ under AdaBoost reweighting,
  which no deployed loop runs. The paper never computes it for its pool.

## Trust signals

- **Credibility:** 3. Against it:
  - an unreviewed preprint from a three-person industry group;
  - substantially AI-drafted (disclosed);
  - one vendor and one benchmark, with T = 4;
  - the production data are the authors' own company's, and the cohort
    roster cannot be released;
  - the abstract oversells the vote result relative to its own App. C.

  For it, the empirical work is unusually careful and self-critical:
  - a gold-patch evaluator audit with root causes;
  - power analysis and required-K for its nulls;
  - cluster bootstraps;
  - tier-pooling and back-fill artifacts exposed by the authors themselves;
  - a pre-AI and within-era control for Part C;
  - Lean-checked algebra, with explicit statements of what Lean cannot
    certify.

  A supplementary repository with code, data and the LLM response cache is
  claimed, but no URL is given.

## Follow-up

- **Relevance:** 3. It is useful prior art and framing for the `/iterate`
  stall question, not canonical evidence. It is SWE-bench coding rather than
  ML research. Its theory restates its assumptions, and it does not run the
  experiment that would test "change the move set, not the retry". It is not
  a 4 because the materially new data, retry idempotence and
  same-vendor overlap, are short-horizon and single-vendor, and the
  production data cannot discriminate mechanism.
- **For the redirect-before-halt idea in [[concepts/budget-as-ceiling]].**
  - G.3 item 4 is that idea stated from first principles: on a plateau,
    "ask which kind of change you are making". Re-sampling, "think again" and
    more rounds explore a fixed class; new tools, retrieval, test execution
    and decomposition can move the plateau.
  - Part B gives the quantitative premise. Retrying with a fixed move set is
    mostly a no-op (52–59% / 13–21% genuine ties per round, 69–81% of
    trajectories never move) rather than harmful (≤ 2.9% regressions). So a
    stall counter that retries wastes budget but rarely damages work.
  - It is a fourth independent source pointing the same way as
    chandran2026autoresearch, zou2026fmlbench and RRSI. But it is framing
    plus an untested prediction, not evidence that redirect works.
- **For [[concepts/shared-substrate-contagion]] and
  [[concepts/pass-at-k]].**
  - It is the agent-trajectory counterpart to kim2026are's MCQ result. A
    same-vendor resampled pool's majority vote equals its average member, so
    voting buys nothing within a tier.
  - Pooling a weaker tier into a vote actively hurts. A task every weak worker
    fails already ties the vote.
  - Add the measurement it proposes and leaves unmade (open problem 2):
    report m* and vote-minus-mean-worker error for any cross-family ensemble
    before claiming diversity.
- **For [[concepts/evolutionary-expansion]] and
  [[literature/papers/srikanth2026recursive]].**
  - AIDE² is the (M1) case in the wild: frozen-weight harness rewriting that
    expands the reachable class.
  - Cor. 4 predicts it saturates at V*(W). AIDE²'s inconclusive ignition test
    (0.780 vs 0.782) is consistent with that but does not discriminate it.
  - Both papers independently reject or neutralize majority-vote ensembling.
- **Evaluator hygiene for downstream projects.** Gold-patch-audit any
  home-built per-round scorer. A 46% false-fail rate went undetected until
  this check was run.
- **Candidates** (none ingested):
  - Xue & Yang 2026, "Rethinking self-evolving agents" (online composition of
    the improvement process, the purest frozen-weight scaffold expansion).
  - Chen, Wang & Qu 2026, an RSI survey (bounded self-refinement vs
    autonomous research loops).
  - He et al. 2026, "Harness engineering for language agents".
  - Papamarkou et al. 2026, "Bayesian control for coding agents" (stopping
    as sequential hypothesis testing).
  - Zhang et al. 2025, Darwin Gödel Machine.
