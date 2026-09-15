---
kind: paper
title: "The Double Measurement Confound in Agent Benchmarks: De-Scaffolding, Ground-Truth Scoring, and Reliability Beyond the Mean"
authors: ["Yonghong Zhang", "Shadi Motaali", "Vu Phong Dinh", "Avin Piroutiniya", "Jorge E. López de Vergara", "Luis de Pedro", "Ricardo Correia", "Isabel M. Parra", "Yong Xie"]
institutions: ["Universidad Autónoma de Madrid", "IMDEA Nanociencia", "Universidad Complutense de Madrid", "Spanish National Research Council (CSIC)"]
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.09218"
code_url: "https://github.com/yonghongzhang-io/comtrade-openenv"
citations: null
source: "raw/papers/zhang2026double.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["evaluation", "benchmark-validity", "scaffold", "harness-vs-model", "scorer-validity", "reliability", "worst-case", "cvar", "pre-registration", "negative-result"]
---

# The Double Measurement Confound in Agent Benchmarks: De-Scaffolding, Ground-Truth Scoring, and Reliability Beyond the Mean

## TL;DR

An agent benchmark score is evidence about the model only if (a) the model,
not the harness, makes the execution-critical decisions and (b) the scorer
responds to whether the work is correct rather than how it looks. The
authors audit their own tool-use benchmark (ComtradeBench) and find both
conditions fail. Under the production scaffold, seven models from three
providers submit **SHA-256-identical payloads**. The shipped judge gives a
**fabricated record set the same score as the correct one (0.987 each)**.
The two artifacts hide each other: fixing only the scorer still ties every
model, and removing only the scaffold still misgrades what it separates.
Fixing both turns the leaderboard into a spread from **0.974 to 0.000**, and
worst-case and CVaR statistics then reorder models the mean treats as
equal. The same confound turned their own **pre-registered confirmation**
into a scorer artifact. On τ-bench the scorer is sound, but changing only
the scaffold flag moves the same model **by up to 0.267** and changes the
ranking.

## Claims

- **The confound is joint, not additive.** Under Proposition 1, if the
  scaffold owns every execution-critical decision and the scorer grades only
  shape and self-reported metadata, no function of the leaderboard
  identifies a property of the model. Under Proposition 2, if every such
  decision is model-owned and the scorer is criterion-valid, score
  differences at matched seeds are differences in model behavior.
- **An auditable minimal condition.** A capability claim is interpretable
  only if `D_claimed ⊆ D_model`. That means the decisions the construct
  claims to measure must be ones the harness actually leaves to the model.
  When this fails there are two repairs: return the decisions to the model,
  or relabel the score as system-level performance.
- **Ownership is necessary but not sufficient.** A model-owned channel must
  also be *non-degenerate*: the model's choices must vary (Var(A) > 0), and
  different choices must lead to different outcomes. Their own L1 instrument
  fails this. SKIP was chosen in 0 of 120 episodes, so "no sample size
  repairs it: the effect is non-identified, not merely low-powered."
- **"A number quoted without its scaffolding level is not interpretable."**
  The scaffold level changes what is measured, not just the size of the
  score.
- **The worst case reorders the mean ranking, up to the frontier.** Worst
  case, CVaR@0.2, and Reliability@τ should be reported as first-class
  outputs, and the worst case should be estimated over seeds of a *seeded
  adversary* rather than cited as an anecdote.
- **Across benchmarks, the scorer half is benchmark-specific and the
  scaffold half is "uncontrolled wherever we probed it."**
- **Licensed-claim lattice (the "validity card").** Each benchmark earns
  one claim. It is model capability only if replay, criterion sensitivity,
  and ownership completeness all pass. With an outcome scorer over an
  uncontrolled scaffold, it is model–scaffold system performance (τ-bench).
  With a shape scorer, it is self-report compliance (ComtradeBench as
  shipped). With a spec scorer and the scaffold unprobed, it is
  specification compliance (BFCL). An item marked "not probed" is never
  read as a pass.

## Methods

- **ComtradeBench.** Seeded, paginated extraction from a mock API. Faults
  are injected by the environment: duplicates, 429/500 errors, page drift,
  and totals rows. Three MCP tools stay constant across all levels. All data
  is a deterministic function of `(task_id, seed)`, so ground truth can be
  regenerated for free. The benchmark predates the audit and is the first
  author's own (Zhang et al. 2026, second place in the AgentX–AgentBeats
  OpenEnv track), which makes this a retrospective self-audit.
- **Scaffolding spectrum.** At **L0** (production) the harness auto-retries,
  deduplicates, drops totals rows, loops pagination, and submits, and the
  model fills templated slots. At **L1** (half-scaffold) the model owns only
  the retry-or-skip decision on a fault. At **L2** (slim) the model owns
  everything, including re-emitting the cleaned record set and calling
  `submit_results`.
- **Ground-truth scorer.** F1 over distinct ground-truth ids in the
  submission. Duplicates, totals rows, and fabricated rows inflate |S| but
  not |U|, which cuts precision.
- **Reliability metrics.** Worst case (minimum over seeds), CVaR@α (mean of
  the worst α fraction, α = 0.2), Reliability@τ (fraction of runs at or above
  τ = 0.9), a seeded bit-reproducible bootstrap 95% CI, and paired Cliff's δ.
- **E1.** Eight models at L2 on five execution tasks, 10 seeds each
  (n = 50 per model). T = 0 except GPT-5 and Claude-Fable-5, whose endpoints
  reject the parameter and so run at provider defaults (disclosed).
- **2×2 joint intervention.** Harness (L0/L2) × scorer (judge/ground truth)
  on T3_duplicates, 5 seeds per cell, 7 models, every episode double-scored.
- **Forged-submission probe.** Canned correct, empty, fabricated,
  duplicate-laden, and contradictory-metadata submissions are run through
  the released, unmodified `judge.py`.
- **Pre-registered stress test (B1).** Escalating vs matched-mean constant
  stress, frozen 2026-06-17, n = 20 per arm, three models. It includes a
  disclosed instrument deviation and a later pre-registration-faithful
  rerun. The exploratory follow-ups (B2 and a slack dose-response) are
  calibrated against **no-LLM rule policies** (always-retry, always-skip,
  quota-aware).
- **External audits, all zero model calls except the τ-bench slice.**
  τ-bench's reward function is replayed on released trajectories with
  semantic perturbations. A τ-bench scaffold slice covers 3 models × 4
  shipped scaffolds × 15 retail tasks, one trial each. BFCL's AST checker
  gets 400 entries × 5 canned submissions. A scorer-kind census covers nine
  benchmarks: executed for four, classified from source for five.
- **BenchAudit.** A roughly 1,000-line benchmark-agnostic core plus
  111–196-line adapters, with 13 built-in oracle checks. Optional
  LLM-assisted layers (scaffold discovery, harness synthesis, probe design)
  only *propose*. A "number guard" rejects any narrated figure that is
  missing from the audit JSON.

## Results

- **Production invariance.** On the published leaderboard, Kimi and Claude
  both score 97.5 and a no-LLM rule-based script scores 96.8. Qwen2.5-7B and
  Llama-3.3-70B produce byte-identical per-seed reward vectors on the
  multi-page task. In the 2×2 at L0, payloads are SHA-256-identical across
  all 21 model pairs on every seed, reasoning models included.
- **Judge probe (T2, 2,345 rows).** An empty submission with well-formed
  metadata scores **0.648**. Fabricated and correct submissions score
  **0.987** each, with ground-truth F1 of 0.0 and 1.0. Correct data paired
  with a contradictory self-report drops by 0.398, while fabricating the data
  behind a well-formed report costs 0.000. "The judge grades the report, not
  the data."
- **2×2.** At L0 the spread is 0.000 under both scorers. At L2 with the
  judge, among models that submit, the judge compresses a 1.000
  ground-truth spread to 0.425. It also misorders: GPT-5 is ground-truth
  perfect but gets 0.768, below Sonnet's 0.948. Llama's all-wrong
  submissions get 0.523. Only L2 × ground truth separates the models fully.
- **E1 spectrum (L2, ground-truth F1).** Fable-5 0.974 (worst 0.824),
  Sonnet-4.6 0.943, GPT-5 0.930, Haiku-4.5 0.924, GPT-4o 0.805, GPT-4o-mini
  0.610, Llama-3.3-70B and Qwen2.5-7B **0.000**.
- **Tail statistics separate what the mean does not.** GPT-5 and Haiku-4.5
  differ by 0.006 in mean but 0.079 in CVaR@0.2 (0.720 vs 0.799).
  Reliability@0.9 is 0.720 for GPT-5, 0.600 for Haiku, and 0.480 for GPT-4o.
  GPT-4o ranks fifth by mean but has a worst case of 0.00. GPT-5 has one full
  zero on mixed faults (T8), giving it the suite's largest reliability gap
  (0.930). Haiku never drops below 0.787.
- **The GPT-4o bimodality has an identifiable mechanism.** At n = 30, 14
  runs score 1.0 and 16 score 0.0. Every episode opens with
  `fetch_page(page=0)` against a 1-indexed API, which returns HTTP 422 in
  the same `retry: true` envelope as the injected transient faults. All 14
  solves vary the page argument; none of the 16 failures do. Each failure
  ends in a schema-valid empty submission that truthfully reports the fetch
  failure.
- **The pre-registered test was confirmed by the scorer, not by the data.**
  B1 under the deviating L1 coverage instrument is null for all three models
  (Δ = +0.000), and SKIP fired 0 of 120 times. The pre-registration-faithful
  rerun gives GPT-5 Δ = +0.011, CI [+0.002, +0.023], p = .034, which counts
  as a *confirmation* under the frozen "≥ 1 frontier reasoning model" rule.
  On the identical episodes, ground-truth F1 and coverage deltas are about 0
  and the submitted data is equally correct in both arms. The effect "lives
  entirely in judge dimensions ground truth does not measure."
- **The rule-policy calibration overturns the exploratory result.** In B2
  (enforced quota, zero slack) the escalation penalty is positive for all
  three models. A no-LLM always-retry policy then produces Δ = +0.096,
  **larger than every model's penalty**, so this is resource arithmetic that
  model decisions only attenuate. With slack, the penalty decays for all
  models. "Decision relevance is thus an inverted U in slack."
- **τ-bench.** The scorer is outcome-based: a corrupted DB write and a
  wrong answer both score 0.0. Still, 5 of 115 retail tasks are satisfied
  by an empty do-nothing trajectory. Changing only the scaffold flag moves
  GPT-4o by 0.267 (0.667 → 0.400), react is the worst scaffold for all three
  models, few-shot triples the Sonnet–Haiku gap, and GPT-4o ties Sonnet
  under act but finishes last under react. The per-model sign tests are
  n.s. (n = 15). A test–retest of one cell scores 0.867 against the original
  0.733, noise of the same order as the spread. η² is 0.0293 for scaffold
  vs 0.0080 for model, both small against the task residual.
- **BFCL.** The correct call passes all 400 entries, and every corruption
  fails (0 of 2,000 cells deviate). It is a spec-valid scorer; its scaffold
  was not probed.
- **Census.** Across nine benchmarks the primary scorer kinds are shape 1,
  outcome 6, specification 1, llm-judge 1. Shape scoring and LLM-judging are
  "two routes to the same criterion-validity failure." GAIA's scorer passes
  5/5 correct submissions and 0/5 fabricated ones.
- **Legacy GRPO runs against the L0 judge reward** (supplement only): 7B
  "starts at ceiling with KL = 0" (zero group advantage), 3B learns and then
  collapses, and 1.5B oscillates. The authors read saturation at
  initialization as evidence the reward carries no model-discriminating
  signal.

## Critique / open questions

- **The dramatic half of the headline comes from a benchmark with an
  unusually bad judge, built by the same first author.** Their own census
  finds exactly **one of nine** benchmarks with a pure shape scorer, and it
  is ComtradeBench. The byte-identical ties, the fabricated = correct tie,
  and the 2×2 masking result are real, but they show what a
  self-report-grading judge does. They don't show that agent benchmarks
  generally have one. The authors say the scorer half is
  benchmark-specific, and the abstract still leads with the joint confound.
- **"Uncontrolled wherever we probed it" rests on one external benchmark.**
  BFCL's scaffold was not probed, so the external scaffold evidence is the
  τ-bench slice alone: 15 tasks, one trial, n.s. sign tests, test–retest
  noise (0.133) about the size of the spread, and η² shares that are both
  tiny. The authors call it "estimation only". It *is* consistent with
  [[literature/papers/wang2026act]]'s 16-combination harness × model grid,
  where the minimalist harness beat feature-rich ones on the same model.
- **"Nearly flat leaderboard" is truer per task than across the suite.**
  The L0 suite means are Llama 0.893, GPT-5 0.932, and Sonnet 0.975 (Table
  A19), and the legacy T9 adaptive-adversary task already spread systems by
  79 judge points. The byte-level invariance is shown on T3 and the
  multi-page task. There is also an internal inconsistency. Figure A7 says
  Claude-Fable-5 "was added to the suite at E1 (L2) time and never run under
  the production or half scaffold", yet Table 1 reports Fable at L0 (0.987 /
  1.000). The simplest reading is a separate later 2×2 batch, but the paper
  doesn't say.
- **De-scaffolding trades one construct-irrelevant floor for another.** At
  L2 the floored tier's zeros are marshaling failures (Llama fetches around
  0.83 of rows at L1 and then submits an empty payload), not decision
  quality. T7 is excluded because the output token budget caps F1 at
  0.26/0.46 regardless of model. In the 2×2, much of the "1.000 spread" on
  T3 is submit vs don't-submit. L2 × ground truth measures execution *within
  interface bounds*, as the validity card states, and that is not the same
  as "the model's decisions."
- **The worst-case reorder for GPT-4o may not be a property of the seeded
  adversary.** Its failure branch is a mock-API design choice: a
  non-transient 422 sent in a `retry: true` envelope, which invites the
  perseveration it then penalizes. GPT-4o runs at T = 0, and the forensic
  episodes have byte-identical prompts and an identical opening call. The
  solve/fail split therefore plausibly comes from provider-side
  nondeterminism on an identical prefix rather than from the seed. That
  undercuts the stated advantage of conditioning on adversary seeds ("making
  the worst case estimable rather than anecdotal") for this case.
- **The pre-registration story is the most valuable part, and it is
  reported with unusual candor.** The deviation is disclosed, the
  instrument's blindness to the pre-registered channel is analyzed, the
  faithful rerun is completed and reported under the frozen rule, and the
  authors explicitly don't dismiss the p = .034 as noise. The lesson
  generalizes: **pre-registration fixes the analysis, not the validity of
  the scorer it names.** The power study makes the same point: power
  "certifies the statistical test and says nothing about whether the
  instrument could have picked up a behavioral effect."
- **The no-LLM rule-policy baseline is a cheap, general control.** A fixed
  always-retry policy beating every model's effect is what exposed B2 as
  budget arithmetic. Any claim that a model *decided* something should
  first check what a trivial policy scores in the same harness.
- **Small n throughout.** There are 10 seeds per task at L2, 5 per 2×2 cell,
  15 single-trial τ-bench tasks, and a single stylized ETL domain. Scores
  are point-in-time on pinned endpoints.
- **Submission residue.** Artifact paths reference `paper/aaai27/`, and a
  legacy figure is titled "AnonBench". It looks like a venue submission and
  is not yet reviewed.
- **Open question.** The paper names a controlled ownership decomposition on
  a benchmark the authors don't own as future work. For research-agent
  benchmarks (MLE-bench / AIDE, PaperBench scaffolds), the `D_claimed ⊆
  D_model` audit has never been run. The obvious version would list which
  of submission formatting, retry-on-crash, and final-checkpoint selection
  the scaffold does for the model.

## Trust signals

- **Credibility:** 3. An arXiv preprint (cs.SE, apparently an AAAI-27
  submission) from reputable Spanish public research institutions (UAM,
  CSIC, IMDEA, UCM) with no track record in this graph. It is not
  peer-reviewed and no citations are established. Raised to 3 by a released
  benchmark repository, a result bundle where every table is regenerated by
  script, bit-reproducible seeded bootstraps, a dated pre-registration, and
  forthright negatives: the deviation and faithful rerun, the confirmation
  explained away by their own scorer, the rule policy that beats every
  model, the T7 solvability exclusion, and the few-shot contamination
  caveat. Held at 3 because it is a self-audit of the authors' own
  benchmark, one domain, small n, and the external generalization claim
  rests on a single under-powered slice.

## Follow-up

- **Relevance:** 4. It adds a new axis to two load-bearing concepts. It
  does not seed a new concept, and its test domain is tool-use ETL rather
  than ML research, so it scores 4 rather than 5.
- **[[concepts/hce-evaluation]].** That concept guards the number against
  leakage, evaluator tampering, retrieval of the answer, overfitting, and
  (via [[literature/papers/ray2026what]]) closed-loop non-identification.
  None of those defenses asks **who made the decisions being scored** or
  **whether the scorer responds to correctness at all**. This paper
  supplies both as auditable conditions: `D_claimed ⊆ D_model` with
  non-degeneracy, and a forged-submission probe. It adds the cleanest
  negative result in the graph that a pre-registered, frozen-rule
  confirmation can be a scorer artifact. For a research-agent harness the
  scaffold question is concrete: a result produced by AIDE's
  retry/debug/select loop is an AIDE+model result.
- **[[concepts/pass-at-k]].** That concept treats variance as sampling
  seeds over repeated runs and asks for distributions and bootstrap CIs. It
  gains three things here. (1) **Tail statistics as first-class outputs**
  (worst case, CVaR@α, Reliability@τ), with a measured case where the mean
  calls two models equivalent and CVaR does not. (2) **A tight distribution
  can mean the model is not being measured.** Byte-identical per-seed
  vectors across unrelated models are zero variance *because* the scaffold
  owns the outcome, so matched seeds and k ≥ 3 don't help. (3) **"Seed"
  needs a referent.** Environment seeds (a seeded adversary) and sampler
  seeds are different axes, and at T = 0 the GPT-4o bimodality apparently
  came from neither. Relates to [[literature/papers/li2026acm]]'s pass@k vs
  passᵏ split: Reliability@τ is the thresholded, continuous-score analogue
  of passᵏ.
- **[[concepts/programmable-evaluator-oracle]].** This is a measured case of
  [[literature/papers/ng2026agent]]'s access-restriction clause: `judge.py`
  is deterministic and programmatic, grades `metadata.row_count` against
  the submitted line count, and scores fabricated = correct. **Determinism is
  not criterion validity.** A scorer that reads the agent's self-report sits
  at the self-report tier however programmatic it looks. The
  forged-submission battery (correct / empty / fabricated / duplicate /
  contradictory metadata) generalizes [[literature/papers/he2026swegate]]'s
  "ship a non-compliant patch alongside the gold patch" into a reusable
  pre-flight check. The legacy GRPO saturation (reward at ceiling, KL = 0)
  is a weak, supplement-only hint of what an invalid oracle does inside a
  loop: it doesn't mislead the search, it gives it no gradient.
- **Independence.** The scaffold-sensitivity half converges with
  [[literature/papers/wang2026act]] (harness × model decomposition,
  Xi'an Jiaotong) and cites Harness-Bench (Yao et al. 2026, arXiv:2605.27922)
  and HAL (Kapoor et al. 2026, ICLR), none of which are in this graph. The
  scorer half converges with [[literature/papers/wang2026androids]]'s flaw
  classes (weak matching, evaluation-logic gaps). Harness-Bench and
  "Benchmarking the Benchmarks" (Vaghasiya et al. 2026, arXiv:2607.02577, a
  validity audit of tool-calling evaluation) are candidates for a later
  digest.
