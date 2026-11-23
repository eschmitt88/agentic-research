---
kind: paper
title: "The Resolution Hypothesis: Agent Evolution's Key Is Identifying Improvements"
authors: ["Yuanhao Zeng", "Shaocheng Lan", "Ao Lu", "Kuang Yan", "Heng Lin", "Xinang Xu", "Yuan Luo", "Yueting Wang", "Yexin Li", "Ruo Shan", "Kan Ren"]
institutions: ["ShanghaiTech University", "ByteDance", "BIGAI", "SUSTech"]
year: 2026
venue: "arXiv 2611.23068 (cs.AI), v1 2026-11-17, preprint (38 pages with appendices)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.23068"
code_url: null    # "All reproducible materials and trajectory data will be released upon the acceptance of this paper." Project page only: https://lineshogan.github.io/resolution-hypothesis/ (not opened at ingest).
citations: null
source: "raw/papers/zeng2026resolution.pdf"
added: "2026-11-23"
relevance: 5
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["selection-noise", "evaluation-resolution", "variance-decomposition", "run-to-run-variance", "evaluation-noise", "power-analysis", "runs-per-method", "method-ranking", "winners-curse", "in-loop-vs-reevaluated", "quality-at-k", "equal-quality-null", "llm-judge-selection", "pairwise-accuracy", "false-acceptance", "one-rule-test", "evaluation-shortcut", "grader-repair", "gepa", "skillopt-lite", "trace2skill", "reasoningbank", "harnesslens", "meta-harness", "ahe", "terminal-bench", "spreadsheetbench", "officeqa", "alfworld", "livemathematicianbench", "process-diagnostics", "literature-survey"]
---

# The Resolution Hypothesis: Agent Evolution's Key Is Identifying Improvements

## TL;DR

A large reproducibility study of frozen-weight agent evolution (prompts,
skills, memory playbooks, harness code). The authors split a reported
score's variance into evaluation noise σ² (same artifact, re-evaluated)
and run-to-run variance τ² (different artifacts from independent
evolution runs). They measure both for seven methods (SkillOpt-Lite,
GEPA, Trace2Skill, ReasoningBank, HarnessLens, Meta-Harness, AHE) on six
models and four benchmarks: 150 configurations, 1,030 evolution runs,
over 2.1M test trajectories, over 10M LLM calls and 400K CPU core hours.

Four results matter for this graph:

1. **One run cannot rank methods at published margins.** σ is 1.0–3.7 pp
   on SpreadsheetBench, OfficeQA and ALFWorld and 4.5 pp on
   Terminal-Bench; τ is 0–12.3 pp. A single-run score has a median SD of
   1.7–4.1 pp per benchmark. 217 of 341 same-benchmark configuration
   pairs (64%) differ by less than one SD of a single-run comparison. In
   nine evolution papers (171 table columns) the median margin over the
   strongest competitor is 3.4 pp (IQR 1.2–6.6). No column reports an
   error bar, and only Trace2Skill's 6 columns average runs. Detecting
   3.4 pp at 80% power takes 5 (SpreadsheetBench), 10 (OfficeQA), 19
   (ALFWorld) or 23 (Terminal-Bench) runs per method; 1.2 pp on
   Terminal-Bench takes 181.
2. **The large published gains were evaluation shortcuts.** Across 27
   model–benchmark pairs with a reported gain above 3 pp, appending one
   hand-written sentence to the unevolved baseline recovers a median 90%
   of the gain. Repairing the grader (LibreOffice recalculation,
   semantic grading, restoring the official instruction) cuts the median
   gain from 17.6 to 0.8 pp, and re-evolving under the repaired grader
   does not bring it back.
3. **At one rollout per candidate, selection is at chance.** On a frozen
   pool of 100 Terminal-Bench skills, execution-score selection picks the
   better of two skills 49.7% ± 1.9 of the time. Three LLM trajectory
   judges score 45.4–49.0%. A single execution score correlates 0.054
   with posterior quality. No selector gains from seeing more candidates.
4. **In-loop gains are noise exploitation.** Eight SkillOpt-Lite and
   eight GEPA runs on Terminal-Bench (two models) report in-loop gains of
   6.9–9.0 pp. Re-evaluated at eight rollouts, the same artifacts sit at
   +0.4 to −3.2 pp. Run-to-run variance on the validation set grows from
   0 to 4.2–24.5 pp² over rounds.

## Claims

- **The Resolution Hypothesis.** "The bottleneck of agent evolution is
  identifying which artifacts bring improvement, not proposing them."
  *Resolution* is the smallest true improvement an evaluation pipeline
  can reliably tell from noise. Below it, selection degrades to chance
  and little accumulates.
- Comparing one fixed artifact with a reference is limited by σ² alone.
  Comparing two methods is limited by τ² + σ², because each run draws a
  new artifact. Re-evaluation shrinks only σ².
- Three predictions are tested on Terminal-Bench: P1, the pools contain
  helpful artifacts; P2, common selectors are near chance; P3, per-round
  selection errors compound into run-to-run variance.
- Evolution succeeds where one behaviour lifts nearly every task at once
  (a shortcut, a history-window setting), because only such gains exceed
  resolution. It stalls when only small task-specific gains remain.
- Recommendations: report σ̂ and τ̂ beside point scores; evaluate
  proposers (Quality@K curves) and selectors (selected-artifact quality
  curves on a fixed pool) separately; add low-variance process metrics
  as diagnostics.
- Explicitly scoped out: weight-updating methods (RL), program evolution
  with deterministic scoring (kernels, ARC-AGI-2, Atari), much stronger
  future models, and candidate-level results beyond Terminal-Bench.

## Methods

- **Models.** GPT-5.4 nano, GPT-5.6 Luna and Terra, DeepSeek-V4-Flash,
  Kimi-K3, GLM-5.2. Every role in a method (solving, reflecting,
  rewriting) uses the same model. The cheapest setting uses all six; the
  harness and Terminal-Bench settings use a subset (Table 5).
- **Benchmarks.** ALFWorld (100/70/67 split, halved), SpreadsheetBench
  Verified-400 (80/39/281), OfficeQA (49/49/148), Terminal-Bench v2.1
  (18/18/53, stratified by difficulty). Scores are binary per task.
  Terminal-Bench runs Mini-SWE-agent 2.4.6 offline in task sandboxes.
- **Methods as run.** Each uses its official implementation and the same
  budget on every model. Every validation score is one rollout per task.
  Selection rules (Table 2): SkillOpt-Lite accepts an edit if validation
  beats the current skill by ≥ 2 points; GEPA uses a 3-task training gate
  and Pareto parent sampling; Trace2Skill and ReasoningBank keep every
  patch or memory (no score-based selection on three benchmarks);
  Meta-Harness and AHE take the best of 5 candidates; HarnessLens
  confirms a paired check before a single validation score.
- **Variance measurement (App. B, D).** R = 8 evolution runs per
  configuration (16 for text methods on nano), each artifact evaluated
  M = 16 times (8 on Terminal-Bench and for harness methods). σ̂² is the
  mean within-artifact variance; τ̂² is a random-effects moment estimator
  that subtracts σ̂²/M, set to 0 when negative (9 of 117 configurations).
  Single-run SD is s = (τ² + σ²)^½. Runs needed: N ≥ 2(z₀.₉₇₅ +
  z₀.₈)²(τ² + σ²)/Δ².
- **Ranking simulation.** 20,000 draws per model and benchmark of one run
  and one evaluation per method, compared with the all-run ranking.
- **Literature survey (App. D.3).** Main results tables of nine papers
  (GEPA, ACE, ReasoningBank, Trace2Skill, SkillOpt, SkillOpt-Lite,
  Meta-Harness, HarnessLens, AHE): 178 columns, 171 with an optimization
  competitor. Margin = proposed method − strongest competitor.
- **Gain audit (App. E).** 12 benchmarks: three from the grid plus
  HotpotQA, PUPA, HoVer, IFBench, AIME 2025, LiveBench-Math (GEPA
  artifacts) and LiveMathematicianBench, SearchQA, DocVQA (SkillOpt-Lite
  artifacts); models GPT-5.4 nano, DeepSeek-V4-Flash, GPT-5.6 Luna. An
  LLM agent flags candidate instructions in the evolved artifacts; the
  authors check each by hand and write the single most responsible rule
  *before* running the test, and do not revise it after. Repairs restore
  the official evaluation or replace surface matching by semantic grading
  (exact match, or both of two LLM judges accept). Re-evolution under the
  repaired grader: GEPA and SkillOpt-Lite (Table 12, 8 runs × 8 evals for
  harness methods).
- **Proposal Pool (§5.1, App. F.1–F.2).** An offline, SkillOpt-Lite-based
  proposer on DeepSeek-V4-Flash, running parallel research sub-agents
  with small local probes (the prompt is printed), writes 100 skills from
  78 trajectories on 10 Terminal-Bench training tasks. Each skill runs 16
  times on 10 disjoint test tasks (16,000 rollouts, 2-hour budget, with
  native-timeout cutoffs re-scored on the same trajectories).
  *Quality@K* is the expected success of the best of K random skills,
  chosen per task, under an empirical-Bayes Beta model with the
  concentration capped at κ_max = 100. The *equal-quality null* refits
  the same estimator on 1,000 simulated pools where all 100 skills share
  the task's pooled rate.
- **Selection Pool (§5.2, App. F.3).** 100 skills from trajectories on
  six Terminal-Bench and three DeepSWE tasks, rolled out on 11
  Terminal-Bench tasks (six source tasks, five held out from skill
  generation), 8 rollouts per artifact–task pair (15 for the ten best).
  Each selector sees the first rollout (two for the native SkillOpt-Lite
  and GEPA rules); quality is posterior on the held-out rollouts.
  Judges: DeepSeek-V4-Flash pointwise (1,111 calls), pairwise (2,200
  calls, both orders), and an ACE-Reflector-style template.
- **Trajectory re-evaluation (§5.3, App. G).** 119 intermediate artifacts
  from 8 SkillOpt-Lite + 8 GEPA runs × DeepSeek-V4-Flash and GPT-5.6
  Luna, re-run on the 18 Terminal-Bench validation tasks at 8 rollouts
  each (evolution used 1). τ̂²_t is computed per round without clipping.
- **Process diagnostic (App. H).** Fraction of check commands whose
  failure output is not masked, per trajectory, on DeepSeek-V4-Flash.

## Results

- **Variance grid (Table 6, Fig. 2).** Per method, 53–79% of comparisons
  fall below one single-run SD (GEPA 54/101, ReasoningBank 61/101,
  SkillOpt-Lite 63/101, Trace2Skill 68/101, AHE 38/57, Meta-Harness
  40/57, HarnessLens 37/47). On Terminal-Bench σ is 2.9–4.5 pp and τ̂ is
  0.0 in 6 of 12 text-method cells. σ is the larger component on three
  of four benchmarks; with many evaluations per artifact, the 3.4 pp
  margin would need about 1, 5, 9 and 3 runs instead of 5, 10, 19 and 23.
- **Ranking a single run (App. D.2).** The top method is right 67%
  (SpreadsheetBench), 51% (OfficeQA), 73% (ALFWorld) and 33%
  (Terminal-Bench) of the time; random is 25%. The full order of four is
  right 33%, 29%, 24% and 8% (random 4%).
- **Survey (Table 8).** Median margin 3.4 pp; median gain over the
  unoptimized baseline 9.3 pp. Per-paper median 3.0 (IQR 2.0–3.8);
  restricting to the 52 columns on the four grid benchmarks gives 3.5.
  152 of 178 columns do not state the run count, 20 state one run. In
  four papers the strongest competitor scores at or below the
  unoptimized baseline in 15 columns.
- **Gain audit (Tables 9–12, Fig. 3, 8).** Shortcuts found in 8 of 12
  evaluations: SpreadsheetBench's grader reads formula cells without
  recalculating (so every formula scores empty); AIME's parser takes only
  a bare integer; LiveMathematicianBench's "a stronger result can be
  proven" option is correct in all 85 of 85 questions where it appears
  (46 of 106 test questions); DocVQA, HotpotQA, SearchQA use near-exact
  match; OfficeQA's SkillOpt-Lite release drops the official bare-value
  instruction. Examples (released → one-rule → repaired gain, pp):
  SpreadsheetBench Luna +38.4 → +39.9 → +5.9; AIME nano +55.3 → +54.7 →
  +1.8; OfficeQA Luna +21.1 → +21.4 → −0.1; LiveMathematicianBench nano
  +27.1 → −0.7 (rule not followed) → −1.7.
- **Gains that survive repair.** HotpotQA: GEPA keeps 5.5–7.6 pp, traced
  to retrieval (supporting-article recall up ~10 pp). HoVer: 8–13 pp from
  retrieval strategies, and one rule recovers only 41–43% for two models.
  Harness re-evolution: Meta-Harness and AHE gain 16–32 pp on ALFWorld,
  but a single change of the history window (the initial agent keeps the
  last two steps) gains 29 pp alone; AHE gains 11–13 pp on the 151
  formula tasks of SpreadsheetBench and 3–4 pp elsewhere. HarnessLens
  returns its initial harness in 47 of 71 runs.
- **Headroom.** Terminal-Bench leaves 28 pp (DeepSeek) and 46 pp (Luna),
  and the best method gains 1.0 and 3.6. SearchQA is answerable closed
  book (removing all snippets: 84.2 → 81.9, 89.4 → 90.2, 89.8 → 90.7).
- **Proposal Pool (Fig. 4a, 9, 11).** Quality@K rises 49.2% → 61.8%
  (K = 1 → 100) at 2 h and 39.7% → 51.6% at native timeouts. The excess
  over the equal-quality null is +3.3 to +3.4 pp at K = 100, separating
  from K ≥ 5, at every timeout cutoff. Sensitivity to the prior cap: +2.20
  pp with overlapping intervals at κ_max = 30, +3.41 at 100, +4.75 at
  300, +6.42 unbounded. The best shared skill scores 57.5% against a pool
  mean of 49.2% and a worst of 41.9%.
- **Selection Pool (Table 1, Fig. 4b, 12).** Pairwise accuracy (%):
  random 50.0, execution 49.7 ± 1.9, native SkillOpt-Lite 48.6 ± 2.6,
  native GEPA 48.6 ± 2.6, judge pointwise 45.4 ± 4.5, pairwise 49.0 ± 2.2,
  ACE 46.9 ± 3.5. False acceptance (share of accepted baseline
  replacements that are worse): random 30.5 ± 14.0, execution 62.1 ±
  42.9, SkillOpt-Lite 59.4 ± 46.2, GEPA 31.6 ± 14.5, judges 39.0–57.6.
  The oracle beats every selector's K = 100 quality within K < 5.
- **Rounds (Fig. 5, 6).** In-loop gains 6.9–9.0 pp; re-evaluated +0.4 to
  −3.2 pp. Noise-corrected τ̂²_t at the final round 4.2–24.5 pp².
- **The one intervention (App. F.3).** SkillOpt-Lite, 16 runs, up to 8
  rounds: 16 validation rollouts per candidate instead of 1 raises the
  mean final test score 61.4% → 64.4% and the best run 64.8% → 72.7%
  (32 test rollouts per task). With one rollout, a lucky incumbent
  (e.g. 80% validation) blocks later candidates, and evolution stalls
  within four rounds. No interval, model or benchmark is stated.
- **Process metric (Table 13).** Evaluation noise 13–57× smaller than
  the terminal reward. Trace2Skill raises post-edit verification +8.73 pp
  (z = 2.17) on DeepSeek; AHE lowers it −17.98 pp (z = −4.19) on Luna;
  neither shows in task success.

## Critique / open questions

- **Against the digest framing: at, with two overstatements and one
  thing the *Why* line should not lean on.** The summary's numbers and
  structure match the abstract and body.
  1. **"Helpful candidate artifacts exist" rests on +3.4 pp, per task.**
     The digest asked how it was established. It is not oracle
     re-evaluation of evolved artifacts. It is an offline pool of 100
     skills from a richer proposer than any evaluated method (parallel
     research sub-agents plus local probes), on DeepSeek-V4-Flash only,
     scored at 160 rollouts each. Quality@K lets every task pick its own
     best skill, so it measures per-task routing headroom. The excess over
     a proper equal-quality null is +3.3 to +3.4 pp at K = 100, and +2.2
     with overlapping intervals under a tighter prior cap. That is real
     but about the size of the median published margin.
  2. **The "shareable artifacts exist" paragraph has no null, and noise
     alone predicts its spread.** Our arithmetic: with per-task rates read
     from Fig. 10 (2 h) and independent Bernoulli rollouts, a skill's
     160-rollout mean has SD ≈ 3.1 pp. The expected max and min of 100
     identical skills are then about ±7.7 pp from the pool mean. The
     paper reports +8.3 (57.5 vs 49.2) and −7.3 (41.9). The "more than 15
     pp" span is what 100 equal skills would show. The +3.4 pp
     Quality@K result is the evidence for P1; this paragraph is not.
  3. **"Selection is no better than random" holds at one rollout per
     candidate.** The paper's own App. F.3 shows 16 validation rollouts
     lifting mean final test by 3.0 pp and the best run by 7.9 pp. Also,
     about half of the Selection Pool tasks leave almost nothing to
     choose between (T8, T10, T11 above 96%; T2, T3 below 6%; read from
     Fig. 12 axes), which pulls pairwise accuracy toward 50%
     mechanically. The 62.1% false acceptance for execution selection
     has a ±42.9 interval; the paper says it may not differ from random.
  4. **"Gap between two independent runs" was measured per method, but
     on Terminal-Bench it is mostly evaluation noise, not different
     artifacts.** τ and σ come from 8 (or 16) runs × 8–16 evaluations in
     every configuration (Table 6). On the Terminal-Bench test split τ̂ is
     0.0–1.5 pp for SkillOpt-Lite and GEPA except GEPA-Luna (6.0), and σ
     is 3.5–4.0 for those cells. Yet §5.3 reports validation-set τ̂² of
     4.2–24.5 pp² (τ ≈ 2–5 pp) for the same method–model pairs; only
     GEPA-Luna agrees between the two (≈ 25 pp² vs 6.0²). Our reading: runs
     drift apart on the 18 validation tasks they were selected on, and
     much of that does not carry to the 53 test tasks. So P3's chain
     ("selection errors compound into the run-to-run variance behind
     ranking flips") is weaker on held-out data than presented. On
     Terminal-Bench the flips come mainly from σ, the final evaluation's
     own noise, which re-evaluation can fix without changing the loop.
  5. **"Rankings often come down to luck"** is strongest on
     Terminal-Bench (33% vs 25% random). On the other three benchmarks a
     single run picks the top method 51–73% of the time.
  6. **The *Why* line** ("a single-run margin is no evidence") is right
     for benchmark margins of ~3 pp. It does not follow that every score
     is unusable: σ-only comparisons of one fixed artifact against a
     reference (an `/elevate`-style before/after on the same artifact)
     need only re-evaluation, not more evolution runs.
- **The one-rule test is the additive mirror of
  [[literature/papers/ding2026harnessdelta]]'s neutralization arm, and
  the two agree on LiveMath and ALFWorld.** Ding disables each detected
  benchmark-specific mechanism and measures the drop; this paper adds the
  single most responsible rule to the baseline and measures the rise.
  Both find LiveMath's always-correct option (21/21 there, 85/85 here)
  and an ALFWorld gain from one behaviour. The rule is written after
  reading the artifacts, so it shows sufficiency only (the authors say
  so), and it fails when the model does not follow one sentence (LiveMath
  with nano: recovers 0%).
- **No neutral-text arm.** On the Selection Pool a random skill already
  sits roughly +8.6 pp over the no-skill baseline (read from Fig. 4b), and
  all three printed skills (App. I) open with the same six generic
  software-engineering bullets. How much of the common gain a fixed
  generic text would give is untested; compare
  [[literature/papers/mitra2026agent]]'s neutral note (+4.5 on held-out
  BFCL).
- **The variance grid mixes graders.** The text-method OfficeQA rows of
  Table 6 match Table 11's *released* (shortcut-bearing) evaluation
  (e.g. GPT-5.6 Luna baseline 35.5 in both, against 61.0 repaired), while
  SpreadsheetBench is recalculated and harness rows use repaired graders.
  σ and τ remain valid noise estimates, but those method means include
  the shortcut.
- **The one fix tested is thin.** The 16-rollout selection result has no
  interval, model or benchmark named, and no matched-compute alternative
  (16× more candidates at one rollout, or fewer rounds).
- **Selection at chance is shown on Terminal-Bench only,** with one
  proposer model and one judge model (DeepSeek-V4-Flash). Six of the 11
  Selection Pool tasks are the skills' own source tasks.
- **Relation to the graph's selection-noise papers.** This is the
  method-level complement to [[literature/papers/hu2026winners]]
  (candidate-level winner's curse, pre-registered, small classifiers)
  and [[literature/papers/zhou2026does]] (selection-only replay of
  unchanged candidates). P3's in-loop vs re-evaluated gap is hu's
  inflation measured on Terminal-Bench, and Fig. 4b is
  [[literature/papers/chen2026auditing]]'s Prop. 1 seen from the
  selector's side. What is new is the per-method τ/σ split, the power
  table, the 171-column survey, and the one-rule plus grader-repair audit.
  It does not test a statistical gate such as
  [[literature/papers/gao2026trustworthy]] (TRUST) or PACE, which it cites.

## Trust signals

- **Credibility:** 3.
  - For: ShanghaiTech with ByteDance, BIGAI and SUSTech; very large
    replicated design (8–16 runs × 8–16 evaluations per configuration);
    unbiased variance estimators stated and derived; a proper
    equal-quality null with a prior-sensitivity check; held-out
    rollouts separate selector evidence from quality truth; one-rule
    texts written before testing; a frank limitations section and an
    explicit "sufficient, not necessary" scope for the audit.
  - Against: preprint, six days old, no citations. Code and trajectories
    are promised only "upon acceptance". The candidate-level evidence is
    Terminal-Bench and DeepSeek-V4-Flash only. The shared-skill spread
    lacks a null, the 16-rollout fix has no interval, and the variance
    grid's OfficeQA rows use the shortcut-bearing grader without saying
    so in §4.
  - Not 4: no released code or data, and the rubric weighs that above
    the institutions. Not 2: the replicated design and controls are well
    above the preprint norm here.

## Follow-up

- **Relevance:** 5. It becomes the canonical method-level evidence for
  [[concepts/hce-evaluation]]'s "a single-run margin is not a result"
  rules (τ/σ split, runs per method, no variance in 171 published
  columns). It adds the cleanest selector-on-a-frozen-pool test to
  [[concepts/evolutionary-expansion]] and
  [[concepts/skill-library-lifecycle]]. Its one-rule audit independently
  confirms [[literature/papers/ding2026harnessdelta]]. Not an ML-research
  loop, but the loop it studies is the propose–score–accept loop
  `/iterate` and `/elevate` both run.
- **For `/elevate`'s evidence gate.** (a) A source paper's reported
  margin of a few points on a single run is not evidence for adopting a
  method; require repeated runs or a σ/τ report. (b) An LLM judge
  reading a trajectory did not select better skills than chance here, so
  a proposal whose only evidence is a model's reading of a run log should
  not pass. (c) The cheapest effective fix shown is more evaluations per
  candidate, not a better judge. (d) Before crediting any large gain,
  run the one-rule arm: append the single most responsible instruction
  to the baseline.
- **For this box (our inference).** `/iterate` compares candidates on one
  draw each. The paper's Eq. 2 with a measured σ gives the number of
  re-evaluations needed before a keep decision; on Terminal-Bench-like
  noise (σ ≈ 4 pp) a 3 pp difference is not decidable from one draw.
- **Candidates** (cited, not yet in the graph):
  - Shawn 2026, "PACE: Anytime-valid acceptance tests for self-evolving
    agents" (arXiv 2606.08106). Cited by three notes; no note of its own.
  - Ye et al. 2026, "On the fragility of self-improving agents: variance,
    task order, and underspecification" (arXiv 2608.18066v2).
  - Shang et al. 2026, "When self-evolution backfires: pre-commit gating
    against skill contamination" (arXiv 2608.05810).
  - Shen et al. 2026, "SkillOpt-Lite" (arXiv 2607.03451), the method
    whose released OfficeQA and LiveMathematicianBench setups carry two
    of the audited shortcuts.
  - Ni et al. 2026, "Trace2Skill" (arXiv 2603.25158), and Xu et al.
    2026, "HarnessLens" (arXiv 2608.27311).
  - Zhang et al. 2025, "ACE: Agentic context engineering" (arXiv
    2510.04618), the highest median margin in the survey (10.4 pp).
