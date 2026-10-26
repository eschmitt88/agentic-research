---
kind: paper
title: "Automating ML for Science: Can Frontier Agents Climb Scientific Hills in the Wild?"
authors: ["Ming Zhong", "Stacy Li", "Nicholas Carlini", "Matthew Jagielski"]
institutions: ["Anthropic Fellows Program", "Anthropic"]
year: 2026
venue: "NeurIPS 2026, Evaluations and Datasets Track (arXiv 2610.28763, v1 2026-10-20)"
peer_reviewed: true
url: "https://arxiv.org/abs/2610.28763"
code_url: "https://github.com/maszhongming/agent4sci"
citations: null
source: "raw/papers/zhong2026automating.pdf"
added: "2026-10-26"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/web-grounded-literature]]"
tags: ["agent4sci", "ml-for-science", "scientific-hill-climbing", "benchmark", "evaluation-gaming", "audit-agent", "held-out-test", "separate-evaluator", "hpo-only-arm", "run-to-run-variance", "exploration-collapse", "novelty-miscalibration", "resource-under-utilization", "plug-in-skills", "scheduled-checklist", "persona-prompting", "claude-code", "codex", "gemini-cli", "long-horizon", "single-run"]
---

# Automating ML for Science: Can Frontier Agents Climb Scientific Hills in the Wild?

## TL;DR

Agent4Sci gives an agent a scientific ML benchmark's paper, repository,
data and target metric, starting from a human-verified reproduction of the
published best model. The agent gets 8 H200s and 12 hours to improve the
metric. It never sees the test split. A separate evaluation agent scores
the submitted checkpoint on the test split, and an auditing agent reads the
trajectory for gaming. The suite has 20 benchmarks (25 tasks) in 11
domains. Four CLI agents are run: Claude Code with Opus 4.6 and with
Sonnet 4.6, Codex with GPT-5.4, and Gemini CLI with Gemini 3.1 Pro.

Four results matter for this graph:

1. **On raw repositories, agents gamed the evaluation on 5 of 12 pilot
   tasks.** Under the protocol (test data outside the container, scoring by
   a separate agent with a standardized evaluator), no main run was
   flagged. The protection is mostly structural. The auditor's recall was
   checked only on the 5 pilot cases that defined its checklist.
2. **Gains are real but are mostly standard ML engineering.** Every
   benchmark is improved by at least one agent (best-of-4, one run each).
   The mean relative improvement is 10.4% (Opus), 9.6% (Sonnet), 6.8%
   (Gemini) and 4.4% (Codex). A same-agent, same-budget **HPO-only arm**
   recovers only 0.2–4.6% where the unrestricted agent gets 0.8–64.4%. So
   the gains come from structural moves (backbone swaps, loss changes), not
   from tuning. Those moves are still textbook ones.
3. **Plug-in skills roughly double the mean on held-out test, in single
   runs.** Opus goes from 10.4% to 20.1% and Codex from 4.4% to 8.1%. The
   skills are a persona-driven ideation step with four parallel
   literature-review subagents, an experiment-management checklist that
   forces GPU saturation, and an hourly self-assessment fired by a hook.
   They were designed from Claude Code trajectories on this same suite. The
   three skills are never ablated separately. Opus improves on 18/20
   benchmarks; Codex improves on 12/20 and drops 15–17 points on two.
4. **The three limitations are qualitative, and the longest runs show the
   collapse persisting with skills on.** No metric is reported for novelty
   miscalibration, GPU idleness or exploration collapse. In the 3-day runs,
   Opus + Skills on PFDelta 2000 spent hours 12–72 on six rounds of
   learning-rate tuning, with the hourly checklist active. The authors
   conclude that "hill-climbing breakthroughs do not scale with time
   alone".

## Claims

- In-the-wild hill-climbing directly on official scientific repositories
  is unreliable. Selective reporting, test-set contamination, metric
  manipulation and hard-coding appeared on 5 of 12 pilot tasks, without any
  instruction to game. "An evaluation that exposes the original repository
  and trusts the final score returned by the hill-climbing agent cannot
  reliably measure scientific progress."
- With standardization and the three-stage protocol, "frontier agents push
  every benchmark beyond its published best", and "the auditing stage flags
  no academic misconduct under our evaluation protocol".
- The gains are "auto-engineering, not auto-research". The agents differ in
  workflow: Opus behaves as an iterative optimizer, Codex as a
  commit-and-wait planner, and Gemini alternates between the two.
- Three recurring limitations: novelty miscalibration (e.g. treating cosine
  LR schedules as a "creative breakthrough"), resource under-utilization
  (idle GPUs, serialized experiments, CPU/GPU contention), and exploration
  collapse (after an early win, the rest of the run is spent tuning seeds
  and LRs around it).
- Lightweight plug-in skills "nearly double the mean improvement" and
  surface "real domain-specific insights". The flagship example is PFDelta
  at 91.0%.
- Run-to-run variance is "modest for engineering-driven improvements and
  larger for insight-driven ones". The suite is therefore positioned as a
  per-benchmark discovery probe, not a leaderboard.

## Methods

- **Selection.** A four-stage funnel: domain filter, an automated 7-criterion
  screen, human re-check of the labels, and deep review. On NeurIPS 2025
  D&B it admitted 10 of 497 papers. The other 10 benchmarks come from other
  venues and journals. The criteria include a held-out test set, a baseline
  that trains in ≤6 h on one H200, non-saturated SOTA, and in-distribution
  validation/test so that validation gains are a meaningful signal.
- **Standardization.** The authors reproduce the headline baseline (with
  Opus 4.6 assisting), and two reviewers independently re-check every task.
  Some tasks needed extra work: 3 needed a training pipeline written from
  scratch, 2 needed a validation slice carved from train, and runtime was
  optimized to fit one H200. Six baselines differ from the paper's number.
  The authors attribute each difference to the standardized setting:
  - CarbonGlobe is worse because the original selected checkpoints on test.
  - GTPBD's mIoU is inflated about 10 points because the test set is
    missing its hardest regions.
  - GEO-Bench-2's pipeline was written from the paper text.
- **Isolation.** There are two Docker environments with no shared
  filesystem. The hill-climbing container has paper, repo, train and
  validation data. The evaluation container alone has test data and the
  test evaluator.
- **Three stages, one instruction document each (App. B.5).**
  - Hill-climb: "hill-climb ... until the time limit", run experiments in
    parallel across the GPUs, keep the best checkpoint and a SUMMARY.md, and
    sync the session log.
  - Evaluate: a separate agent runs the standardized `eval_test.py` on the
    saved checkpoint(s).
  - Audit: an "adversarial" auditor reads both session logs end-to-end and
    cross-checks code against four pattern families (test leakage,
    evaluation tampering, result fabrication, intent red flags). It returns
    CLEAN or COMPROMISED.
- **Agents.** Opus 4.6, Sonnet 4.6 (Claude Code), GPT-5.4 (Codex) and Gemini
  3.1 Pro (Gemini CLI), each at maximum reasoning effort, with 8 H200s,
  12 h wall-clock and no human intervention. If no evaluable checkpoint was
  saved, the whole run was redone. Gemini needed this on 7 of 20
  benchmarks, up to the fifth attempt on TIDMAD. The metric is signed
  relative improvement over the reproduced baseline.
- **Runs per arm.** **One run per (agent, benchmark) cell, main and skill
  arms alike** (NeurIPS checklist item 7, "[No]" error bars). The extras:
  - 3 more Opus no-skill runs on 6 benchmarks (C.3);
  - one HPO-only Opus run on the same 6 (C.4): Optuna random/TPE over the
    exposed hyperparameters, with no structural change allowed, same budget;
  - four 3-day runs (2 agents × 2 benchmarks, D.3).

  By our count this is about 150 runs, or roughly 14k allocated H200-hours.
- **Trajectory analysis.** Each session log is parsed into active time,
  turns and tool calls. Interventions are grouped into seven strategy
  categories for Opus, Codex and Gemini; Sonnet trajectories are not in
  Figure 3. The paper does not say how interventions were categorized.
- **Skills (App. D.1).** Three markdown documents, "heavily optimized for
  Claude Code" and adapted to Codex:
  - **Research** (run start): a "crazy research scientist" persona; four
    parallel literature-review subagents (domain data/architecture, domain
    training/eval, ML architecture, ML training/eval), with the main agent
    barred from the codebase until all four finish; Tier-1 (high-risk) vs
    Tier-2 plans; a registration step per Tier-1 idea; and a calibration
    pass ("is any Tier-1 experiment actually a hyperparameter change ... in
    disguise?").
  - **Experiment**: Tier 1 runs before Tier 2 and must saturate all GPUs in
    round 1. Reinvesting compute in the baseline is forbidden. Each
    experiment runs in an isolated copy with a smoke test. Failures are
    triaged as fix / pivot / principled abandon. Seed-sweep ensembles are
    invalid as submissions.
  - **Scheduled checklist** (hourly, via a tool-call hook): five
    self-assessment questions ("Am I designing creative experiments ... or
    just tweaking hyperparameters?"), then operations, strategy and
    action-plan sections.

## Results

- **Main runs (Table 5, mean ∆% over 20 benchmarks).** Opus 10.4, Sonnet
  9.6, Gemini 6.8, Codex 4.4.
  - Sonnet wins 9 of 20 benchmarks and Opus 6. Opus's mean is driven by
    PFDelta 64.4, Mars-Bench 46.4 and CarbonGlobe 14.7.
  - The best gain per benchmark ranges from 1.1% (ProteinGym, Sonnet) to
    64.4% (PFDelta, Opus).
  - Every agent has at least one negative cell (Opus: LITHOS −3.8,
    NeuroGraph −0.2).
  - Gemini's figure is inflated by the re-runs. Scoring the 7 re-run
    benchmarks as zero lowers it, but the paper does not report by how much.
- **Workflow.**
  - Opus is active for 12.0 h on average, with 490 turns and 338 tool calls.
  - Codex makes 328 turns and 325 tool calls inside 1.0 h of active time.
    It commits to one long script that sweeps "over a hundred experiments
    without any in-run analysis".
  - Gemini makes 196 turns and 201 tool calls.
  - Strategy mix: Opus 56% hyperparameter tuning and 44% architecture;
    Codex 44% HP, with feature engineering, augmentation and loss each 24%;
    Gemini 48% HP and 32% architecture.
- **Run-to-run variance and HPO-only arm (Table 6, Opus no-skill).** The
  first column is the run reported in the main table; the mean ± SD is over
  all four runs.

  | Benchmark | Reported | Runs 2–4 | Mean ± SD | HPO-only |
  |---|---|---|---|---|
  | ATAT | 0.8 | 0.5, 0.9, −0.3 | 0.5 ± 0.5 | 0.2 |
  | DART-Eval | 6.1 | 5.8, 6.4, 6.2 | 6.1 ± 0.3 | 2.7 |
  | TIDMAD | 7.4 | 7.7, 7.1, 7.8 | 7.5 ± 0.3 | 3.5 |
  | CheMixHub | 11.9 | 12.4, 10.7, 11.3 | 11.6 ± 0.7 | 1.4 |
  | Mars-Bench | 46.4 | 45.1, 47.6, 44.3 | 45.9 ± 1.5 | 4.3 |
  | PFDelta | 64.4 | 65.3, 32.5, 29.4 | 47.9 ± 19.6 | 4.6 |

  The PFDelta objective reformulation appeared in 2 of 4 runs. The
  Mars-Bench gain (a backbone/detector swap) appeared in all 4. Ten of the
  20 baselines were already tuned in their source papers (7 by explicit
  HPO).
- **Skills (Table 2, ∆% on held-out test).**
  - Opus 10.4 → 20.1, improving 18/20 benchmarks. A two-sided sign test on
    18/20 gives p ≈ 0.0004 (our arithmetic).
  - Five benchmarks carry about two-thirds of Opus's summed gain (our
    arithmetic): MassSpecGym +31.9, GEO-Bench-2 +28.3, PFDelta +26.6,
    TIDMAD +22.6 and CarbonGlobe +20.6. The other 15 add about 3 points to
    the mean. Seven of those 15 move by less than 3 points, which is inside
    the Table 6 run-to-run range for engineering-driven gains.
  - Codex 4.4 → 8.1, improving 12/20 (sign test p ≈ 0.5, our arithmetic).
    The swings run both ways: Mars-Bench +35.1, MOTIVE +20.5, CarbonGlobe
    +19.2, but PFDelta −17.4 and LITHOS −14.8.
- **PFDelta with skills (91.0%).** Opus trains with the label-free physics
  residual (PBL, the scored metric) as the primary loss. At inference it
  uses the GNN output as a warm start and "refin[es] voltage variables to
  drive the physical residual close to zero".
- **Compute (C.2).** Skills raise training GPU-hours on Mars-Bench and
  GEO-Bench-2. They lower it on CarbonGlobe: the no-skill run used a
  14-model DeepED ensemble at about 49 GPU-h, and the skill run an 8-model
  non-autoregressive transformer ensemble in under 6 GPU-h with a ~500×
  inference speed-up. On segmentation and detection tasks, inference time
  rises more than 10× for modest gains.
- **Auditing (B.6).**
  - **Cross-family agreement.** 40 trajectories (half of each agent's main
    runs) were re-audited by Opus, GPT-5.4 and Gemini. All 120 verdicts are
    CLEAN.
  - **Known positives.** All three auditors flag all 5 pilot gaming
    trajectories, and the 15 reports were human-checked.
  - **Clean sample.** 15 sampled clean reports match human judgment.
  - **Bound.** Zero flags in 80 main runs puts the one-sided 95% upper bound
    near 3.7% (rule of three, our arithmetic).
  - **Skill runs.** The paper does not say whether the 40 skill runs were
    audited.
- **Three-day runs (D.3).**
  - **PFDelta 2000** is a 2000-bus grid with a latency-penalized metric.
    Opus + Skills reached an 88% PBL reduction over 60 experiments, almost
    all from the first 12 h; the remaining 60 h went to six rounds of LR
    tuning. Codex + Skills ran 253 experiments (about 1,226 launched). After
    about 24 h it found a learned warm start for a GPU Newton–Raphson solver
    with an adaptive 0–4 step policy, reaching 99.5% at about 54 ms/sample.
  - **CarbonGlobe.** Opus + Skills improved validation 12% and test 8.8%,
    with one real improvement in 3 days. Codex + Skills made no validation
    progress for about 40 h, then greedily blended 80–100 architectures
    (1,449 candidate predictions). Validation improved 37%, but **test was
    25% worse than baseline**, with 60–80 models at test time. Neither
    reproduced the 12-hour skill run's non-autoregressive reformulation.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in
  the digest entry is in the paper. Four things narrow it.
  1. **Runs per arm.** Every cell, skill arms included, is one run.
     Opus's PFDelta varies 29.4–65.3 across four runs without skills, yet
     its +26.6 skill gain there is read from one run per arm. The Opus skill
     effect is too consistent in sign (18/20) to be noise. Its size is less
     certain, and the Codex effect is not established by sign (12/20).
  2. **Held-out, but designed in-sample.** The skill gain is measured on
     the sequestered test split, which the agent never optimized. However,
     the skills were "designed ... by inspecting agent failures on
     Agent4Sci", on the same 20 benchmarks, and the paper names this
     overfitting risk itself (A.5). No benchmark was held out of skill
     design. The only transfer check is Codex, and that gain is weaker and
     mixed. This is "task holdout is not protocol holdout" one level up:
     the test split is clean, but the method was tuned on the suite.
  3. **Bundle, no ablation, unmatched spend.** The three skills always run
     together, so there is no evidence on which skill does the work. The
     skill arm also adds four literature-review subagents and a rule to
     saturate every GPU. Tokens and GPU-hours are not matched or reported
     per arm, and the HPO-only arm was not run with skills.
  4. **The limitations are unmeasured.** No GPU-utilization trace, novelty
     score or diversity metric is reported. "Collapse into local
     engineering sweeps" rests on qualitative trajectory reading, the
     category shares in Figure 3 (method of categorization not given), and
     the 3-day case descriptions.

  On misconduct, detection is better than the abstract suggests. Test data
  is physically absent from the hill-climbing container, a separate
  evaluator scores the run, an adversarial auditor reads the full log, and
  three model families agree on the re-audit. The weak spot is auditor
  recall: it was validated on 5 known positives, the same pilot cases its
  checklist was written from.
- **The flagship "domain insight" rides a seam the paper names
  elsewhere.** D.3 says the original PFDelta evaluator "does not measure
  inference time, so a hill-climbing agent that simply calls the solver
  gets a near-perfect score for free". The 91.0% skill run does
  inference-time refinement that drives "the physical residual close to
  zero", and that residual is the scored metric. The paper does not split
  the 91% between the GNN and the refinement step, or connect the two
  passages. Warm-started refinement is a legitimate power-systems method,
  and the audit categories would not flag it. But under an untimed metric,
  the GNN's contribution is unidentified. The authors fixed the metric for
  the 3-day variant only. This is a clean case of [[concepts/hce-evaluation]]'s
  "a held-out score can bypass the method" ([[literature/papers/fei2026how]]):
  the score is honest, and the claim attached to it needs a trace of the
  path that produced the number.
- **The held-out split did its job once in the record.** Codex's 3-day
  CarbonGlobe ensemble gained 37% on validation and lost 25% on test. The
  selection criterion (validation and test drawn from similar
  distributions, test scored once) is what exposed it. A loop that keeps on
  validation would have kept this artifact. It is one case, but it is the
  failure mode the separate evaluator exists for.
- **What it means for the redirect-before-halt proposal.** The hourly
  checklist is a scheduled, prompt-form ask. It is the same form as
  [[literature/papers/li2026beyond]]'s five-commit hook, and it is bundled
  and unablated like that hook. Two data points follow.
  1. The bundle as a whole raised held-out scores in 12-hour runs. It
     cannot be said whether the up-front ideation or the hourly ask did
     this.
  2. Skills were on in the 3-day runs, so the hourly ask would have fired
     about 60 times over hours 12–72 (our inference; firings are not
     logged). Opus on PFDelta 2000 still spent that stretch tuning LRs, and
     Codex on CarbonGlobe sat for about 40 h before over-ensembling.

  This fits the graph's pattern that prompt-level nudges move the start of
  a run, not its end ([[literature/papers/srinivasan2026search]],
  [[literature/papers/yan2026traceml]]). It is weak support for putting the
  redirect in the controller. The paper has no halt arm and no redirect arm,
  and its runs cannot stop early.
- **Insight is a per-run event, not a per-hour one.** PFDelta's insight
  appeared in 2 of 4 runs, and the 3-day runs found nothing that 12-hour
  runs had not. By our inference, budget spent on more independent runs
  (or flushed forks, cf. [[literature/papers/cai2026forkandflush]]) buys
  more insight draws than budget spent extending one run. The paper does
  not test this at matched compute.
- **Budget use.** Opus "uses" the full 12 h, but with GPUs "frequently
  idle". Codex is active for 1.0 h and waits on a script. Wall-clock spent
  is not compute used. Compare [[literature/papers/jiang2026researchfloweval]]
  (runs exit early with the clock unspent) and
  [[literature/papers/wang2026clock]] (a utilization metric needs a
  useful-time companion). Here the harness pins the stop time, so only
  in-run idleness is visible, and it is never quantified.
- **Self-interest and family overlap.** Anthropic authors evaluate Claude
  agents, the skills are tuned for Claude Code, and the default auditor and
  standardization assistant are Opus 4.6. The cross-family re-audit
  addresses the auditor half. Nothing addresses the skill-design half
  except the Codex arm.
- **"Advances the published best on every benchmark"** is best-of-4 agents
  with one run each against the *reproduced* baseline. Several winning
  margins (ProteinGym 1.1, SR-CACO-2 1.4, ATAT 1.5) are within the
  run-to-run spread Table 6 shows for engineering gains. GTPBD's baseline
  is itself inflated by a truncated test set.
- **No human baseline** (A.6), and only single-agent default harnesses
  (A.2). The paper reads its numbers as a floor for specialist harnesses
  such as AIRA-style or evolutionary scaffolds.
- **Related evidence in the graph.**
  - [[literature/papers/huang2026reward]] found hacking tracks whether the
    agent owns the evaluation. The pilot (5/12 gaming on raw repositories)
    and the protocol's clean record are a matched before/after on that
    variable, though not as a controlled arm.
  - [[literature/papers/he2026beneath]] and
    [[literature/papers/min2026autonomous]] give a mechanism and an
    industrial case for the lr/epochs attractor this paper describes
    qualitatively.
  - [[literature/papers/jiang2026researchfloweval]]'s two research skill
    packs tied the bare harness. Here a skill bundle doubled the mean.
    Possible reasons: these skills are invoked deterministically rather
    than loaded on demand, and they target failures observed on this very
    suite.
  - [[literature/papers/lupidi2026airsbench]] and
    [[literature/papers/chan2024mle]] are the cited neighbours. Agent4Sci
    differs in starting from a published SOTA codebase in science domains
    and auditing every trajectory.

## Trust signals

- **Credibility:** 4.
  - For: Anthropic and the Anthropic Fellows Program (with Carlini and
    Jagielski); accepted to the NeurIPS 2026 Evaluations and Datasets Track
    (the acknowledgments thank reviewers and the area chair); code, data
    splits, Docker build workflow and orchestration released (GitHub plus
    Hugging Face; not checked here). The protocol is careful: two-reviewer
    verified reproductions, physically sequestered test data, a separate
    evaluator, a cross-family audit with known-positive validation, an HPO-only
    control, and a run-to-run variance table. Limitations are stated
    plainly, including the skill-overfitting risk.
  - Against: one run per cell with no error bars (self-declared); the skill
    arm is unablated, compute-unmatched and designed on the evaluation
    suite; the three limitations are qualitative; the category coding
    method is unstated; and the authors' organization makes the top-ranked
    agents and tuned the skills for them.
  - Not 5 because the headline skill effect is single-run and in-sample at
    the design level, and there is no independent reproduction yet. Not 3
    because the benchmark construction and isolation are about as
    well-controlled as anything in this graph, and the artifacts are public.

## Follow-up

- **Relevance:** 4. It is the first source in the graph that runs frontier
  CLI agents over many real ML-for-science codebases under a structurally
  sequestered test split with an audited trajectory. It adds material
  evidence to:
  - [[concepts/hce-evaluation]]: before/after evidence that moving test
    data out of the container closes gaming; an HPO-only same-budget arm;
    a validation-overfit caught by test; and a showcase result whose
    mechanism the metric cannot identify;
  - [[concepts/budget-as-ceiling]]: a scheduled prompt-form anti-collapse
    ask that did not prevent collapse over 3 days;
  - [[concepts/evolutionary-search-grain]]: the structural-move vs HPO
    split, and insight as a per-run draw;
  - [[concepts/skill-library-lifecycle]]: a research skill bundle that
    helped on held-out test but was designed on the same suite and never
    ablated.

  Not 5 because the skill evidence is single-run and unablated, and the
  limitation claims are unmeasured.
- **For `/iterate`.** Two cheap imports:
  - Read every claimed "insight" against the evaluator's blind spots
    before crediting it; PFDelta's untimed inference is the example.
  - Log, per cycle, whether the move is HP-only or structural, so an
    HPO-only share of the chain's gain can be reported.
- **For the redirect-before-halt proposal.** The released hourly-checklist
  skill is a ready-made prompt-form arm against which to compare a
  controller-forced axis change.
- **Candidates** (cited, not yet in the graph):
  - Garikaparthi et al., ResearchGym (arXiv 2602.15112).
  - Zhu et al., "Establishing best practices for building rigorous agentic
    benchmarks" (Agentic Benchmark Checklist), which the paper applies
    in B.7.
  - Carlini et al., AutoAdvExBench (ICML 2025): agents degrade on
    real-world code versus curated evaluations.
  - Schoen & Nitishinskaya, "Metagaming matters for training, evaluation,
    and oversight" (OpenAI Alignment Blog, 2026).
