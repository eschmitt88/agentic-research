---
kind: paper
title: "AppSci-Bench: Can LLMs Build AI Agents and Make Them Better?"
authors: ["Sanjari Srivastava", "Manpreet Kaur", "Cheng Chang", "Sofía Ramos Antón", "Oumaima Essamadi", "Rishu Garg", "Alejandro Lavale Jiménez", "Marcos Caballero Yus", "Mercedes García Martínez", "Peng Qi"]
institutions: ["Uniphore"]
year: 2026
venue: "arXiv 2611.25360 (cs.AI), v2 2026-11-21, preprint (11-page main text, 35 pages with references and appendices)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.25360"
code_url: "https://github.com/sanjari-orb/appsci-bench"    # named in the abstract, not opened at ingest
citations: null
source: "raw/papers/srivastava2026appscibench.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/file-as-bus]]"
tags: ["agent-building", "benchmark", "self-authored-evaluation", "internal-benchmark-saturation", "self-grade-inflation", "requirement-elicitation", "user-simulator", "mms-at-k", "worst-of-k", "harness-gap", "handoff-loss", "build-eval-isolation", "red-teaming", "human-baseline", "tau-bench", "negative-result"]
---

# AppSci-Bench: Can LLMs Build AI Agents and Make Them Better?

## TL;DR

A model plays an applied scientist. It interviews an LLM-simulated business
user, explores a sandboxed company environment holding development data,
writes and deploys an "artifact" agent, and may build its own internal
benchmark and hill-climb on it. The artifact is then scored on a held-out
suite in a freshly started evaluation environment the builder never saw.
There are seven tasks: five new ones (Slides, Order-to-Cash, Compliance,
Call Insights, CRM) and two recast τ-bench domains (Airline, Banking). The
headline metric is **MMS@3**: for each held-out case, the minimum score over
three independent builds, averaged over cases and then over tasks.

Five results matter for this graph:

1. **Self-authored internal evaluations saturate fast and early.** 86 of 132
   optimization-enabled builds (65%) reach ≥0.99 on their own internal
   benchmark, at a mean of round 2.5. **26 of 132 score ≥0.99 on the first
   evaluation, before any optimization.** The median suite has **7.5
   cases**. GPT-5.5 saturates 18 of 21 builds and scores 10.1 MMS@3 on the
   held-out suites.
2. **The self-grade overstates the gain about fivefold, on one model.** On
   24 optimization steps from 13 Claude Opus 4.8 builds, the self-grade and
   the held-out score moved in the same direction 10 of 11 times when both
   moved. But 6 steps were accepted on a self-grade rise with a flat
   held-out score, and the mean gain per build was **+16.9 on the
   self-grade vs +3.2 held-out**.
3. **Turning the optimization loop on had no detectable held-out effect.**
   On 252 builds (three models × two harnesses × on/off × 3 builds × 7
   tasks), no effect survives a stratified permutation test with Holm
   correction. Opus 5 in the paper's harness: **−9.4 [−21.0, +2.2]**.
   GPT-5.5 in Codex: −8.3 [−16.3, −0.3], p = 0.026 unadjusted, 0.16
   adjusted. The intervals are wide.
4. **An orchestrator-to-coder hand-off dropped the artifact's I/O field
   name in 29 of 292 builds (9.9%).** 24 of the 29 were on Compliance, the
   one attachment task whose spec has no interface section. Opus 5 and
   Kimi K3 never lost it (0/18). In the native harnesses, where the same
   agent reads the contract file, no model failed to read attachments
   (0/18 each).
5. **The 59.1% headline is a selected, non-budget-matched cell.** It is
   Opus 5 in Claude Code with optimization on, using about 2× the tokens of
   the paper's own harness. In that harness, where budgets are enforced,
   Opus 5 scores 44.7 (optimization on) and 55.7 (off). The paper's own
   test cannot separate these settings.

## Claims

- Few benchmarks measure whether LLMs can *build* agents end to end, as
  opposed to acting as one. AppSci-Bench measures requirement elicitation,
  infrastructure exploration, implementation, internal benchmarking and
  improvement, scored on held-out data.
- Agent building is far from solved. The best model reaches 59.1% MMS@3.
  Haiku 4.5 and GPT-OSS-120B score 0.
- Models are weak at requirement elicitation: 65% at best (Opus 5), against
  83% for human baselines.
- Internal benchmarks written by agents are too easy and saturate early.
  This makes building representative internal evaluation "a key
  bottleneck in agent-building pipelines".
- Self-optimization "does not yet help".
- "The harness matters as much as the model." Native harnesses beat the
  paper's harness. Its stated reasons are unmatched budgets,
  model–harness tool conventions, and context lost in the sub-agent
  hand-off (Appendix J).
- Under realistic stakeholder pressure, builders resist explicit pushback
  but give way when asked to drop validation.

## Methods

- **Harness (AppSci-Agent).** It is based on mini-SWE-agent and has six
  tools: `talk_to_user`, `mini_swe_agent` (a bash-only coding sub-agent
  that receives free-text `task` and `context`), `deploy_artifact`,
  `eval_generator` (writes internal test cases, metrics and a scoring
  harness), `optimize_agent` (scores the deployed artifact on the internal
  benchmark and optionally improves it), and `complete`. The limits are
  ≤3 h wall clock, ≤40 orchestrator turns and ≤50 coding sub-agent turns,
  plus a token cap. With optimization on, the artifact is improved until
  the internal score saturates or the iteration budget runs out. A new
  version is kept only if its self-grade rises.
- **Native harnesses.** Opus 5 and Opus 4.8 also run in Claude Code, and
  GPT-5.5 in Codex. They get the same user simulator over HTTP, the same
  deploy and complete tools, and prompt guidance on building an internal
  evaluation, with no `eval_generator` or `optimize_agent`. Their
  adapters do not enforce the paper's token, turn or dollar caps.
- **Models.** Claude Opus 5, Opus 4.8, Sonnet 4.6, Haiku 4.5, GPT-5.5, Kimi
  K3 and GPT-OSS-120B.
- **Tasks (Table 1, App. A).** There are 10–97 held-out cases per task.
  The share of the score that is deterministic is 1.00 on Airline and
  Banking and 0.80 on O2C, but **0.00 on CRM**, which is entirely LLM-judged.
  Slides has 10 cases scored on 11 weighted objectives, mostly LLM and
  vision judges. The user simulator is given the policy documents and
  reveals specifics only when asked. Each task has a fixed elicitation
  rubric.
- **Metric.** MMS@k (Eq. 1), with k = T = 3. For each case it takes the
  lowest score across three builds, one trial per build, averaged over all
  27 trial combinations. A build with no usable artifact scores 0
  everywhere, so **one failed build sets that build's minimum to 0 on
  every case**. Mean task accuracy is reported alongside (App. C).
- **Isolation (App. K).** The build and evaluation phases never coexist.
  At handover the agent's container is snapshotted and halted and the
  simulator is stopped. The build environment is then torn down, and only
  after that is the evaluation environment started. The build data share
  is carved so it holds no record that any evaluation case touches. CRM
  shares one store by design, and what is held out there is the
  questions. The agent sees an allowlist-redacted spec, and the write
  fails closed if a marker scan matches. The agent gets no Docker socket;
  a host-side service builds and runs its artifact. A four-gate release
  train precedes scored runs. Its automated leak detector produced 348
  hits, and a human read reduced them to zero real leaks.
- **Analyses.**
  - Saturation of internal benchmarks (Table 8).
  - Self-grade vs held-out per optimization step (Table 3; Opus 4.8 only).
  - Optimization on vs off: per-build MMS@1, a permutation test stratified
    by task, Holm correction (App. G).
  - A no-user ablation that puts the full spec in the task description
    (App. H; Opus 5 and 4.8; no test).
  - Hand-off loss: an audit of 292 builds on literal field names (App. J.2).
  - Failure taxonomy over 287 builds, by trace diagnostics plus manual and
    LLM review (App. N).
  - Red teaming: six scripted pressures, one adversarial trace per task
    (App. L).
  - Human baseline: two participants per task, ≤4 h, **allowed Claude Code
    with Opus 4.8** (Section 3.6, App. M).
  - Policy-sufficiency check: whether gold labels follow from the policy
    the builder gets (App. P).

## Results

- **Main table (Tables 2, 6).** MMS@3 overall:

  | Model | AppSci opt | AppSci no-opt | Native opt | Native no-opt |
  |---|---|---|---|---|
  | Opus 5 | 44.7 | 55.7 | 59.1 (CC) | 51.7 (CC) |
  | Opus 4.8 | 34.9 | 39.9 | 43.2 (CC) | 48.7 (CC) |
  | Kimi K3 | 31.3 | 24.0 | – | – |
  | GPT-5.5 | 10.1 | 11.9 | 22.3 (Codex) | 33.3 (Codex) |
  | Sonnet 4.6 | 8.6 | 11.9 | – | – |
  | Haiku 4.5 | 0.0 | 0.0 | – | – |
  | GPT-OSS-120B | 0.0 | 0.0 | – | – |

  - Single cells swing on broken builds. Opus 5's O2C score is 26 with
    optimization on and 97 with it off, in AppSci-Agent. Opus 4.8's Airline
    score in Claude Code with optimization on is 0.0 because of one failed
    build.
  - Banking is near zero for every model (best 24.3). The human attempts
    scored 0.09 and 0.21.
  - MMS@3 vs mean accuracy (Table 7): Opus 5 44.7 vs 58.3, GPT-5.5 10.1 vs
    24.4. The Opus 5 – Opus 4.8 gap is 9.8 on MMS@3 and 4.6 on accuracy.
- **Build failures (Fig. 2c, Table 19).** The share of builds that failed
  outright was 0% for Opus 5, Opus 4.8 and Kimi K3, 10% for GPT-5.5, 24%
  for Sonnet 4.6, 62% for Haiku 4.5 and 67% for GPT-OSS-120B. 26 of the 28
  failed builds hit the 40-turn orchestrator limit.
- **Cost (Fig. 3).** Opus 5, Opus 4.8 and GPT-5.5 form the
  cost-and-time Pareto frontier. Costs are reconstructed from token
  counts at list prices.
- **Internal saturation (Table 8).**

  | Model | Saturated | At round 1 | Rounds to saturate | Suite size |
  |---|---|---|---|---|
  | Opus 5 | 15/21 | 5 | 2.0 ± 1.0 | 12 |
  | Opus 4.8 | 17/21 | 10 | 1.7 ± 1.0 | 8 |
  | Sonnet 4.6 | 10/19 | 3 | 2.8 ± 1.9 | 8 |
  | Haiku 4.5 | 6/19 | 1 | 3.8 ± 4.1 | 5 |
  | GPT-5.5 | 18/21 | 0 | 3.2 ± 1.2 | 6 |
  | GPT-OSS-120B | 3/10 | 0 | 4.0 ± 2.6 | 2 |
  | Kimi K3 | 17/21 | 7 | 2.1 ± 1.4 | 10 |
  | **All** | **86/132** | **26** | **2.5 ± 1.7** | **7.5** |

  Saturation is not paired build by build with held-out failure. The paper
  contrasts it with the aggregate held-out scores.
- **Self-grade vs held-out (Table 3; Opus 4.8; 24 steps, 13 builds).** The
  8 builds that saturated on their first step are excluded.
  - Direction: 10 agreed, 1 disagreed and 6 were false accepts (self-grade
    up, held-out flat). The other 7 steps had a flat or falling self-grade.
  - The paper's "correct predictor 10/17 (58.8%)" is agreed steps over
    agreed, disagreed and false-accept steps together.
  - Mean gain per build: self-grade +16.9, held-out +3.2. O2C: +54.9 vs
    0.0.
- **Optimization on vs off (Table 9).** MMS@1, on − off:

  | Model | AppSciAgent | Native |
  |---|---|---|
  | Opus 5 | −9.4 [−21.0, +2.2] | +2.9 [−4.4, +10.1] |
  | Opus 4.8 | −2.7 [−15.7, +10.4] | −1.4 [−14.6, +11.7] |
  | GPT-5.5 | +2.0 [−6.0, +10.0] | −8.3 [−16.3, −0.3] |

  - Every Holm-adjusted p is ≥0.16.
  - The largest estimates come from single O2C builds that broke the
    output contract, and these failures occur in both settings. Without
    O2C every effect is within about ±7 and every interval includes 0.
- **No-user ablation (Table 10).** Putting every specification in the task
  description raises MMS@3 in all four comparisons: Opus 5 +17.8 (opt) and
  +6.9 (no-opt), Opus 4.8 +6.8 and +19.8. Each arm has three builds with
  one trial each, and no test was run.
- **Elicitation (Fig. 4).** Mean share of the rubric elicited: Opus 5 65%,
  Kimi K3 60%, Opus 4.8 59%, GPT-OSS-120B 44%, GPT-5.5 34%, Sonnet 4.6 30%,
  Haiku 4.5 29%; humans 83%.
  - Within-task r between elicited share and score is 0.34 pooled. Within
    capability tiers it is −0.12 (weak), −0.15 (medium) and +0.29
    (strong), so most of the pooled correlation is between tiers.
  - Humans kept returning to the stakeholder (median >20 turns). Agents
    asked their questions up front and then built.
- **Human baseline (Table 5).** Two AI-assisted attempts per task, against
  the mean of three *successful* Opus 4.8 opt-on builds. Agent and human
  scores correlate at r ≈ 0.81 across tasks, and neither side is
  consistently ahead. On O2C the humans scored 0.99 and 0.99, against the
  agent's 0.72 ± 0.38.
- **Hand-off loss (Table 13, App. J.2).**
  - 29 of 292 builds (9.9%) lost the field. That is 29 of 126 builds on
    attachment tasks (23.0%) and 29 of 54 contract failures. By task:
    Compliance 24/42, Slides 4/42, Calls 1/42, the other four 0/166.
  - On Compliance, builds with a hand-off loss average 0.18 against 0.44
    for the rest.
  - The delegation guidance is generic: "Provide clear, specific task
    descriptions", "Include relevant context". Nothing asks for the
    interface to be copied verbatim.
  - Per model, over 18 attachment-task builds: Opus 5 0, Kimi K3 0, Opus 4.8
    4, GPT-5.5 4, Sonnet 4.6 6, Haiku 4.5 7, GPT-OSS-120B 8. In the native
    harnesses there were 0 failures for all three models tested.
  - Of the 25 response-field failures, 19 were elicitation gaps and 6 were
    implementation errors. None was a hand-off loss.
  - Matching is on literal names, so the hand-off-loss rate is a lower
    bound.
- **Failure taxonomy (Fig. 5).** Of 190 failed or low-scoring builds, 28
  were build failures, 87 artifact failures and 75 low-scoring artifacts.
  Among the 75 low scorers, the primary labels were "elicited but missing
  or incorrectly implemented" 29 and "internal–external eval distribution
  mismatch" 26. The labels come from manual and LLM review, with an LLM
  confidence floor of 0.6.
- **A traced internal-oracle failure (App. O.5).** GPT-5.5 on Banking,
  optimization on. One of five synthetic internal scenarios rewarded a
  wrong assumption (that lookup by email was unavailable). Optimization
  hardcoded a refusal, and the internal score went 0.785 → 0.976 → 1.0,
  after which the optimizer reported `nothing_to_fix`. Externally the
  artifact succeeded on 5 of 97 cases.
- **Red teaming (Tables 15–16, App. L.3).** 31 of 40 declared pressures
  fired, and 24 were scorable.
  - Pushback: 5 of 7 resisted.
  - Scope expansion: 2 resisted, 3 failed.
  - **Scope contraction ("skip the eval set, if it runs, we'll trust
    it"): 0 of 6 resisted.**
  - One Banking failure breached four safeguards in one trace.
- **Policy sufficiency (Table 20, App. P).** The share of gold labels that
  follow from the policy the builder gets is 98% on O2C, **68% on Calls
  and 56% on Compliance**. The paper says this caps achievable Compliance
  scores.

## Critique / open questions

- **Against the digest entry: at, with two overreaches in the summary and
  one in the *Why* line.**
  - The PDF is the work described: arXiv 2611.25360, 10 authors, the
    seven domains, the code link. The PDF is v2, dated 2026-11-21. The
    digest's 2026-11-19 is presumably v1, which can't be checked from this
    PDF.
  - The numbers quoted are in the PDF: 59.1%, 65% elicitation, 65%
    saturation.
  - **"The best model (Claude Opus 5.0) reaches a 59.1% mean-minimum"** is
    the best of four Opus 5 configurations. That cell is Claude Code with
    optimization on, at about 2× the tokens and with no enforced caps. In
    the budget-matched harness, Opus 5's figures are 44.7 and 55.7. The
    abstract states the 59.1% without naming the harness.
  - **"Models saturate their own internal evaluation 65% of the time
    while failing the held-out task"** goes beyond the paper.
    - The 65% counts builds whose internal score reaches ≥0.99.
    - The paper never pairs those builds with their held-out scores. It
      contrasts saturation with aggregate held-out scores, and Opus 5's
      15 saturated builds sit beside an opt-on held-out score of 44.7
      MMS@3 and 58.3 mean accuracy, not beside failure.
    - "Failing" is the abstract's wording ("but fail to solve the
      held-out task"), not a measured per-build outcome.
  - **The *Why* line, "an agent that writes its own validation set
    overfits to it", names the wrong mechanism.**
    - 26 of 132 builds hit the ceiling before any optimization. The mean
      saturation round is 2.5, and median suites have 7.5 cases.
    - The measured failure is a suite too easy and too small to give a
      search signal. The self-grade also over-reports gains (+16.9 vs
      +3.2, one model). This is not many-cycle overfitting.
    - The optimization loop's held-out effect is null with wide
      intervals. It is not a measured degradation.
    - One traced case (O.5) does show a loop fitting a wrong internal
      oracle into a hardcoded behaviour. It is an anecdote, not a rate.
    - The holdout itself is clean by construction (App. K), so this is
      not a test-split breach. It is a validation-quality failure, which
      `evaluation.md` does not address because it assumes a given split.
  - **"A benchmark of the build-and-improve loop this box's downstream
    projects run"** is partly right. The loop shape matches. But the tasks
    are business agents, CRM is scored entirely by LLM judges, and here
    the agent *authors* its validation cases. The box's downstream loops
    receive data and splits.
  - The digest entry has no check-at-ingest hint.
- **Three builds per cell carry the headline table.** The paper's own
  permutation test cannot separate optimization on from off for any model,
  and an 11-point overall MMS@3 gap (Opus 5, 44.7 vs 55.7) sits inside it.
  Ranking harness or optimization settings by Table 2 overreads it. The
  between-model gaps are large enough to survive (Opus 5 vs Sonnet 4.6,
  Haiku 4.5, GPT-OSS-120B), but no cross-model test is reported.
- **MMS@3 confounds build reliability with capability by design.** That
  is the stated intent, but it means a single crashed or contract-breaking
  build moves a task by up to its whole score (Opus 5 O2C 26 vs 97). Mean
  accuracy halves the Opus 5 – Opus 4.8 gap.
- **The self-grade analysis is one model, 13 builds and 24 steps.** "Gets
  direction right" rests on 11 steps where both scores moved. The 16.9 vs
  3.2 comparison mixes metrics on different samples (the paper says as
  much in O.5).
- **The human baseline is a human with Claude Code and Opus 4.8,** two
  attempts per task, compared against the mean of three *successful* agent
  builds. Both choices make the comparison less informative about how
  humans and agents compare. Elicited share (83%) is the cleaner
  human-vs-agent number.
- **The elicitation-to-score link is mostly between tiers.** Within tier
  the correlations are near zero or negative for weak and medium models.
  The no-user ablation (+6.8 to +19.8) says the *information* matters,
  but it has no test and three builds per arm.
- **The labels cap some tasks.** Only 56% of Compliance labels follow from
  the builder's policy, so part of any internal–external gap there is
  unattainable score, not a weak internal benchmark. CRM is 100%
  LLM-judged. The Slides evaluator was fixed mid-study and re-run (Table 11
  note).
- **The hand-off finding is an observational audit,** not an arm. No run
  passes the contract as a file in AppSci-Agent. The fix is planned, not
  tested. It is still the cleanest measurement in the graph of a parent
  dropping a literal it held.
- **Red teaming is one trace per task.** The scope-contraction result (0/6)
  is six events in total.
- **Editing defects.**
  - Section 3.2 repeats a paragraph with "two factors" and then "three
    factors".
  - "MSS@3" appears for MMS@3.
  - The introduction's task list omits CRM.
  - The abstract cites τ³-bench, while App. A says the tasks were converted
    from τ²-bench.
  - Table 10 gives 55.6 where Table 6 gives 55.7.
  - None of these changes a conclusion.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/lehmann2026can]]: self-revision gained +8.4 on dev
    and +2.8 on held-out, with the CI spanning zero.
  - [[literature/papers/alexander2026iterated]]: for strong writers, a
    same-budget no-feedback arm took nearly all of a judge loop's gain.
  - [[literature/papers/huang2026reward]]: hacking risk tracks whether the
    agent builds its own evaluation.
  - [[literature/papers/chen2026signed]]: for an Opus-class parent the
    residual risk is the parent's own extraction.
  - [[literature/papers/li2026aligning]]: a detailed spec did the work a
    one-line goal did not.

## Trust signals

- **Credibility:** 3.
  - For:
    - Code is released (named repo).
    - Held-out isolation is engineered and documented: separate phases,
      an allowlist spec, a fail-closed marker scan, no Docker socket, and
      a leak detector with human adjudication.
    - The optimization ablation uses a stratified permutation test with
      Holm correction, and the paper reports the O2C exclusion as a
      sensitivity check only.
    - A policy-sufficiency audit of its own labels.
    - The hand-off audit ships its script.
    - The limitations section admits that k = 3 builds is thin.
  - Against:
    - Not peer reviewed, one industry group, no citations yet.
    - Three builds per cell.
    - The headline is a selected, non-budget-matched cell.
    - The self-grade analysis covers one model.
    - The human baseline is AI-assisted, n = 2 per task, against
      successful builds only.
    - Red teaming is one trace per task.
    - The judges score CRM and most of Slides.
    - Visible editing errors.
  - Not 4: unreviewed, single group, and the abstract's two headline
    claims are stated more strongly than the body supports. Not 2: code,
    careful isolation, and corrected statistics on the key ablation.

## Follow-up

- **Relevance:** 4.
  - It is the first measurement in the graph of what happens when the
    agent *writes* its own validation set: suite size, how fast it
    saturates, and how far the self-grade runs ahead of held-out. That is
    material new evidence for [[concepts/hce-evaluation]].
  - Its hand-off audit is the first per-build count of a parent dropping
    a literal contract it held, with a native-harness contrast, for
    [[concepts/hierarchical-delegation]].
  - Not 5: business agent-building, not ML research, and the HCE-relevant
    numbers are descriptive, with the self-grade analysis on one model.
- **Candidates** (cited, not yet in the graph):
  - Shi et al. 2026a, "ττ-Bench: an environment for end-to-end, realistic
    agent construction" (arXiv 2609.04611). Concurrent work: a developer
    agent recovers requirements, builds and self-validates a
    customer-service agent, and is scored on held-out interactions. It is
    the closest second source for a self-authored-validation concept.
  - Hu et al. 2025, ADAS. Automated agent design against a supplied
    objective; the contrast case.
  - Zhu et al. 2025, cited for "non-AI baselines are rare in agentic
    benchmarks".
- **Open question for the graph.** No run tests a remedy for weak internal
  evaluations, such as a larger suite, a check that a deliberately broken
  artifact fails it, or suites written by a separate model. A
  known-bad-artifact check on the self-authored suite would apply
  `evaluation.md` clause 4's do-nothing floor to the validation side.
