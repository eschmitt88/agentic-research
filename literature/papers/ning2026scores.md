---
kind: paper
title: "Scores Alone Do Not Prove Discovery: The Discovery Certification Protocol for Auditing AI Research Agents"
authors: ["Jingjie Ning", "Shanshan Zhong", "Xiaochuan Li", "Ji Zeng"]
institutions: ["Carnegie Mellon University (School of Computer Science)"]
year: 2026
venue: "arXiv (cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.09219"
code_url: "https://github.com/cxcscmu/Discovery-Certification-Protocol"
citations: null
source: "raw/papers/ning2026scores.pdf"
added: "2026-09-14"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/pass-at-k]]"
tags: ["evaluation", "certification", "preregistration", "counterfactual", "recovery", "audit", "verifier", "evidence", "web-capture", "hce"]
---

# Scores Alone Do Not Prove Discovery: The Discovery Certification Protocol for Auditing AI Research Agents

## TL;DR

A good held-out score shows a result is *useful*. It does not show the
research run was *needed*. DCP (Discovery Certification Protocol) splits
the question into three registered gates. Gate 1 checks utility on a sealed
test. Gate 2 asks whether fresh matched agents can recover the score
without the run's history. Gate 3 is an optional randomized test of
truthful vs neutral feedback. A deterministic, LLM-free verifier replays
the decision from frozen bundles. Across five **designed calibration
cases**, the verifier's decision matched the declared role in 5/5. That
includes one refutation, where a no-history challenger beat the target on
knapsack, and one `audit incomplete` returned despite an observed feedback
effect of 1.0. Each full audit took **435–507 model sessions and
$56–61**.

## Claims

- **Utility, recoverability and feedback effect are three separate
  claims** about one numerical outcome. A score answers only the first.
- **Any valid method counts as a recovery.** The recovery rule is
  `Valid(a) ∧ score(a) ≥ x − ε`, and it does not care how the challenger
  got there. Recombining known parts, transferring a technique from another
  domain, or writing a different implementation all count as witnesses.
  That makes the audit portable across artifact types (programs, models,
  data products, recipes).
- **A single witness vetoes Core.** A probability bound is a different
  kind of evidence: "a rare recovery and a small recovery probability can
  coexist." The protocol gives witnesses and bounds separate decision roles.
- **A null result from challengers counts only if the audit was adequate.**
  Positive controls must solve known instances when given the needed
  information. Weak controls give `audit incomplete`, not a pass.
- **Every attempt that starts spends error budget and stays in the
  ledger.** That covers recovered and incomplete audits, timeouts and
  invalid outputs. A best-of-k procedure counts as **one** k-candidate
  episode.
- Producing evidence and checking decisions are separate interfaces. Task
  adapters supply validity, scoring and feedback. One shared verifier
  decides.

## Methods

- **Information objects.** K is the fixed background (model, rules,
  starter code, tools, public methods). E₀ is the task observations fixed
  before the run. L⋆ is the research history the run created and used
  later. A⋆ is the output chosen by a rule fixed before sealed scoring. P
  is the machine recovery check. The line between E₀ and L⋆ follows
  provenance: a measurement returned because the agent chose an action
  belongs to L⋆. A pool of independent candidates plus a final pick by
  score is treated as a search-and-selection baseline, not adaptive
  research.
- **Registration comes before the run.** It fixes the task, model,
  information, tools, budgets, baseline, validity rules, selection and
  stopping rules, and the statistical analysis. All candidates are
  committed before sealed scoring. One symmetric evaluator scores the
  baseline, the target and the controls.
- **Gate 1:** `LCB[μ(A⋆) − μ(b)] ≥ δ_min`. Finite workloads use the exact
  mean; sampled ones use an empirical-Bernstein interval.
- **Gate 2:** fresh matched agents get K, E₀, the same model and budget,
  and **W_obs**, the exact model-visible Web bytes the target run saw,
  captured by a recording gateway. L⋆ is withheld. Responses to challenger
  actions come from a frozen *non-directional* policy, so challengers can
  reason and compile but cannot run new experiments. With zero recoveries
  in n episodes, `p_upper = 1 − α^(1/n)`. Core requires `p_upper ≤ ρ`,
  zero witnesses, and adequate controls.
- **Gate 3:** fresh paired branches start from a checkpoint registered in
  advance. One arm gets truthful feedback. The other gets neutral feedback
  that keeps timing, schema and length but carries no directional
  information. A **null calibration** compares the neutral channel with a
  second non-informative channel on tasks already solvable from E₀, and
  must fall inside `±δ_sham`. Evidence requires
  `LCB(Δ) ≥ δ_evidence + δ_sham`.
- **Error budget**, total α = 0.05: 0.005 each for main validation,
  candidate classification, control adequacy and neutral recovery; 0.01
  each for challenger recovery, the feedback effect and null calibration.
  ρ = 0.05, 45 positive controls, minimum control recall 0.8,
  δ_evidence = 0.34, δ_sham = 0.17.
- **Verdicts:** Core, Evidence, recovered (Core refuted), audit
  incomplete, statistical uncertainty.
- **Cases:** SQLite-Web (DeepSeek-v4-flash). Choose 4 partial indexes from
  16 query families; 4 families carry hidden high traffic. Virtual
  catalyst (DeepSeek-v4-pro). 8⁵ = 32,768 recipes; one 36-well plate
  returned around an anchor recipe. Device calibration: 24 hidden ±1
  gains, 4 given. Multidimensional knapsack. Affine parity (deliberately
  underpowered). Agents ran through Claude CLI pointed at the registered
  endpoint.

## Results

| Case | Main / baseline | Recovery line | Gate 2 | Decision |
|---|---|---|---|---|
| SQLite-Web | 0.8855 / 0 | 0.8805 | 0/96, best 0.6734 | Core + Evidence |
| Virtual catalyst | 1.0000 / 0.5990 | 0.9500 | 0/96, best 0.8146 | Core + Evidence |
| Device calibration | 1.0000 / 0.3667 | 0.9500 | 0/80, best 0.7133 | Core |
| Knapsack | 0.9349 / 0.9140 | 0.9329 | recovered at 0.9363 | refuted |
| Affine (low N) | 1.0 / — | — | 2 episodes | audit incomplete |

- **Recovery bounds.** SQLite and catalyst: 0/96 each, `p_upper = 0.0468`
  (90 episodes needed at α = 0.01, ρ = 0.05; 96 registered). Positive
  controls 45/45 in both, recall LCB 0.8889 ≥ 0.8. Device: 0/80, bound
  0.0477, controls 60/60 (LCB 0.9155). A secondary Web-disabled SQLite
  Core audit: 0/80, best 0.7947 vs line 0.9383, $25.56.
- **Feedback effect.** Both full audits: truthful 30/30, neutral 0/30.
  Effect 1.0, exact paired 99% CI [0.6379, 1.0]. Null studies (60 pairs
  each): contrast 0, CI [−0.0950, 0.0950], inside ±0.17. The margin
  0.6379 ≥ 0.51 passes. Neutral-arm recovery is reported separately, with
  an upper bound of 0.1619 at n = 30.
- **Knapsack refutation.** The target cleared its registered gain
  (0.0209 > δ_min = 0.01). Even so, **one** challenger episode, with no
  history, produced two legal solutions at 0.9363 and 0.9356. Both are
  above the recovery line, and the first beats the target itself. A small
  real gain over baseline was reachable from priors alone.
- **Affine incomplete.** Main score and feedback contrast were both 1.0.
  Control recall LCB was 0.0707, so the verifier returned `audit
  incomplete` rather than certifying.
- **Cost.** Catalyst: 435 sessions, $56.40. SQLite-Web: 507 sessions,
  $61.17. Device: $19.08. Once collected, the decisions replay offline for
  free.

## Critique / open questions

- **The cases are built so the effects are large by design.** In
  SQLite-Web, truthful feedback *is* the per-family traffic weighting, and
  the neutral arm gets uniform weights. So 30/30 vs 0/30 is close to
  guaranteed by the task design. The catalyst plate works the same way.
  The paper honestly calls these "calibration" cases. What they show is
  that the verifier reaches the declared decision on known ground truth.
  They do not show DCP separating real discoveries from recoverable ones
  in the wild. No real research output (a paper, an AlphaEvolve-style
  result) is audited.
- **Five cases with declared roles and 5/5 matches is a sanity check, not
  an error rate.** No false-refutation or false-incomplete rate is
  estimated. There is also no case where the verifier *disagreed* with its
  declared role.
- **Core is scoped narrowly.** The bound covers one fresh episode, from
  the same model, at the same budget, with no experimentation allowed. By
  the paper's own formula, a campaign of k episodes recovers with
  probability `1 − (1 − p)^k`. At p ≈ 0.047, twenty episodes give roughly a
  62% chance of recovery. A stronger challenger model or a bigger budget
  is a new registration. "Core certified" means "not recoverable by this
  challenger, once," which is much narrower than the word "discovery".
- **Gate 2 challengers cannot experiment** (non-directional responses
  only). Recovery therefore tests reachability from priors plus E₀ plus
  W_obs, not whether an independent research effort would rediscover the
  result. The paper says the latter needs a separate rerun with fresh
  truthful experiments.
- **The threat model is not exercised.** It assumes a strategic claimant
  and an independent audit authority that holds the sealed test and
  countersigns. Every reported bundle records `certificate
  issued=false`, with a local content-addressed registry. No adversarial
  claimant was simulated.
- **Neither gate is cheap.** About 500 sessions per full audit is two to
  three orders of magnitude more than the target run it audits. This is a
  publication-grade certification tool, not a per-task gate.
- **Lineage and independence.** Three of four authors (Ning, Li, Zeng)
  also wrote [[literature/papers/ning2026closedloop]]. DCP builds that
  paper's certify-after-search framing into a protocol. Five of the
  references are Ning et al. 2026 self-citations. For
  [[concepts/hce-evaluation]], these two notes are **one research line,
  not two independent attestations**. The AI-use statement also says GPT
  and Codex helped with protocol and statistical design, implementation
  and writing.
- **What the paper gets right, and what is worth copying.** It gives
  `audit incomplete` its own verdict and returns it even when the observed
  effect is perfect. It separates "incomplete audit" from "statistical
  uncertainty". It keeps the neutral-arm recovery bound as a separate
  reported number instead of folding it into the Evidence verdict.

## Trust signals

- **Credibility:** 3 — CMU SCS group (the cxcscmu lab). Code, PyPI
  packages (`dcp-audit`, `dcp-harness`) and offline-replayable evidence
  bundles are released, and they include certificate identifiers, which
  is a strong reproducibility signal. It is an arXiv preprint with no peer
  review and no established citation count. It is held at 3 rather than 4
  because the empirical content is five designed calibration cases whose
  effects are largely built into the tasks. The independent-authority
  half of the trust model is specified but never run. The paper also
  self-cites heavily from one research line.

## Follow-up

- **Relevance:** 4 — this is the implementation attestation the 09-13
  `/elevate` pass said [[concepts/evidence-gated-completion]] lacked, but
  only partly, and for a different target. It is a released, executable,
  LLM-free verifier that actually *refuses*. It returns `recovered` for
  knapsack and `audit incomplete` for affine despite a perfect observed
  effect. But it gates **discovery claims after the fact at ~$60 per
  audit**, not agent task completion inside a working harness. It reports
  no before/after effect on agent behavior and no false-rejection rate.
  So it strengthens the concept's vocabulary and design, but does not
  clear the bar the concept set for elevation.
- **What transfers to evidence-gated-completion.** (a) A five-way verdict.
  It splits zhu2026claimreceipt's single INCONCLUSIVE into *audit
  incomplete* (the evidence apparatus failed) and *statistical
  uncertainty* (complete audit, interval crosses a threshold). (b) The
  attempt ledger ("every started attempt remains in the ledger";
  best-of-k is one episode). This mechanizes ding2026autonomous's
  result-selection disclosure, which the concept names as this repo's
  actionable gap. (c) Positive controls as a precondition for a null
  verdict.
- **What transfers to hce-evaluation.** A held-out score is Gate 1 only.
  The knapsack case shows a registered, sealed, real gain being reached by
  a no-history agent. HCE certifies utility, not the necessity of the
  loop. Separately, capturing and replaying W_obs is a constructive
  alternative to "disable retrieval" (guidance item 6): record the exact
  model-visible bytes and condition the claim on them.
- **Mirror image of [[concepts/compression-as-generalization-test]].**
  bertran2026fits hands a fresh agent a *narrow* channel and requires it
  **to reproduce** the result (is the gain real?). DCP Gate 2 hands a fresh
  agent *no* lineage and requires it **to fail** (was the run necessary?).
  It is the same instrument — a fresh agent under a controlled information
  boundary — with opposite success criteria. Together they box a claim in
  from both sides.
- Gate 3's null calibration (neutral vs a second non-informative channel,
  which must be equivalent) is a concrete, runnable version of
  ray2026what's "paired reruns under both conditions". It adds a check
  that the control arm itself is inert, which ray does not specify.
