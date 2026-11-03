---
kind: paper
title: "Automatic Construction of Strong Simple Baselines"
authors: ["Caleb N. Ellington", "Jiaqi Wang", "Sohan Addagudi", "Elijah Cole"]
institutions: ["Carnegie Mellon University", "Sunday Scientific", "University of Washington"]
year: 2026
venue: "arXiv 2610.40700 (cs.LG), v1 2026-10-28, preprint (marked 'Preliminary version')"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.40700"
code_url: "https://github.com/cnellington/automatic-baselines"
citations: null
source: "raw/papers/ellington2026automatic.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/information-firewall]]"
tags: ["simple-baselines", "baseline-adequacy", "trivial-floor", "known-bad-floor", "competent-prior", "scientific-ml", "nature-family", "claude-opus-5", "single-attempt", "test-time-compute", "context-ablation", "validation-test-gap", "replicate-spread", "harness-audit", "target-score-leak", "normalized-ratio"]
---

# Automatic Construction of Strong Simple Baselines

## TL;DR

The authors take 36 supervised prediction tasks from Nature-, Science- and
Cell-family papers (2024–2026) that propose a deep learning (DL) method.
Claude Opus 5 at medium effort gets one shot per task: one script, 30
minutes, 4 CPUs, no GPU, no pretrained weights, no network. Each task is
run 5 times (180 attempts, 157 scored). The score is *simple baseline
strength* (SBS): (baseline − trivial floor) / (published DL score − trivial
floor), on the task's one held-out test fold.

Three results matter for this graph:

1. **A single cheap shot gets most of the published gain.** Median task SBS
   is 0.955 (mean 0.857). On 12 of 36 tasks the median replicate matches or
   beats the published number. Six tasks sit below 0.5. Failed attempts are
   dropped from the medians; scoring them as 0 gives mean 0.796, median
   unchanged.
2. **More compute did not clearly help.** On 9 tasks at high effort, mean SBS
   went 0.88 (30 min) → 0.88 (150 min) → 0.90 (validation-picked best of 5)
   → 0.92 (ten rounds) → 0.96 (five-hour iteration). No arm differs from the
   single attempt (smallest Holm-adjusted p = 0.16), and the arms also differ
   in what documentation the agent could read.
3. **Data, not the paper, carries the result.** On 9 tasks with 3 attempts
   each, removing all paper text moved SBS 0.88 → 0.82 (n.s.). Removing the
   training labels dropped it to 0.17, and removing test inputs too gave
   0.00.

The baseline never sees test labels; grading runs outside the container.
But the task README states the published score in at least 23 of 36 tasks,
and every attempt read it. The DL reference is a reported number, not a
rerun, and is usually the best entry in the paper's table.

## Claims

- Missing or weak baselines inflate DL gains in science (cited cases:
  perturbation prediction, time series, tabular, recommenders), and peer
  review has not kept up.
- An LLM can build a strong simple baseline per task at modest cost. This
  "is a practical and scalable check on deep learning contributions".
- SBS is unitless, so tasks with different metrics can be pooled. The
  authors warn that its value depends on the trivial predictor and the
  metric, and that equal SBS steps are not equal scientific value.
- A modest budget gives an informative comparison: one 30-minute attempt
  reached 91.5% of the five-hour arm's mean SBS.
- Context ablations are a general way to measure how much guidance an agent
  needs to reach a fixed, verifiable outcome.

## Methods

- **Task selection (§3.1, App. B, F).** Open-access articles from 2024-01-31
  to 2026-09-18 in nine journals: Nature, Nature Methods, Nature Machine
  Intelligence, Science, Cell, Cell Systems, Nature Communications, Nature
  Biotechnology, Nature Genetics. An LLM screens abstracts for a supervised
  DL contribution and assigns a field. A discovery agent reads full text and
  code and inventories candidate tasks. One task per paper, chosen as the
  one that "best represents its core contribution". The policy forbids
  choosing a task "where the new method happens to win". Only papers with
  public data qualify. The 36 are "the first complete set" that gave ≥3
  tasks in each of 8 fields and ≥3 per Nature-family journal. "Task
  decisions did not use any baseline results." AI agents (Claude Code,
  Codex) extracted tasks and ran experiments; humans verified the tasks.
- **Journal mix.** Nature Communications 18, Nature Machine Intelligence 6,
  Nature Methods 6, Nature 3, and one each from Science, Cell and Cell
  Systems. Nature Biotechnology and Nature Genetics are on the list but
  contribute no task.
- **Task interface.** `setup` gives training features and labels, test
  features and an example submission. `submit` writes predictions. `grade`
  runs outside the container against private labels. Splits are the
  paper's own or a documented reconstruction. Each task runs on **one
  fold**. Sixteen source studies used several folds; for 13 of them the
  published reference is one number, sometimes a fold average, so SBS
  compares one fold with that number.
- **DL reference (App. D.3).** By default the best result in the paper's
  table of results. On three tasks a comparator scores higher and the
  paper's own method is used instead. On `gosai_machine_2024` the reference
  is the best of 488 reported hyperparameter configurations (0.8867,
  against 0.883 for the headline model).
- **Trivial predictor (§3.3).** A constant, a training-set summary, or a
  fixed rule with no fitted parameters. It may use prior knowledge, and
  grouped tasks get one per group. Floors are measured on the test fold,
  not assumed. Most sit at chance, but not all: `lv_learning_2026` has a
  trivial PCC of 0.862 against a DL score of 0.9726 (Table 2).
- **Agent.** Claude Opus 5 (`claude-opus-5`) through the Claude Agent SDK
  0.2.132, 60 turns, no shell. One harness tool runs the script. The prompt
  asks for "a reasonable baseline model", not a simple one; simplicity comes
  from the environment. 47 of 157 scored scripts used PyTorch and 85 used
  gradient-boosted trees. The script must print a self-computed validation
  score on a split the agent holds out from training data.
- **Isolation (App. E).** The container has no network except a relay to
  three Anthropic API endpoints. Web search, fetch and code execution tools
  are refused. Test labels, the trivial-floor file and the reported-results
  file are outside the container. The task's source code and documentation
  are inside, and the README states the reported score for most tasks.
  Unlabeled test features are available, so semi-supervised use is
  possible.
- **Replicates and failures.** 5 attempts per task. Infrastructure failures
  are retried. Agent failures are final and excluded from the task median.
  All 23 unscored attempts were the agent's script crashing before writing
  predictions (22 ordinary bugs, 1 missing library).
- **Compute comparison (§4.2, App. E).** 9 tasks, 4 shared with the main
  cohort, chosen from earlier single-attempt results as a mix of easy, hard
  and saturated tasks. High effort. Arms: 30-min single (5 reps), 150-min
  single (3 reps), validation-selected best of 5, ten rounds (3 reps),
  five-hour budget with a 50-round cap (2 reps). Iterative arms end with a
  judge session that picks the round. A tenth task was dropped because row
  repeats in its test file revealed the label. Paired two-sided Wilcoxon,
  Holm over four comparisons.
- **Context ablations (§4.3, App. D).** The same 9 tasks under 8 levels,
  L1 (full paper, all data) to L8 (column names only). 3 attempts per cell,
  no task README. Reported scores that appear verbatim in the paper text
  are masked. 207 attempts started, 190 scored. Paired Wilcoxon against L1,
  Holm over seven comparisons.
- **Audit.** Every relay request was logged: 2,610 (main), 7,641 (compute),
  1,577 (context). None was refused and none was a web search. Eight
  attempts asked for a withheld file, which was absent. Code released, with
  a pinned image recipe.

## Results

- **Main cohort (Fig. 1, Table 2).** Median SBS 0.955, mean 0.857 over 36
  task medians. 12 of 36 at SBS ≥ 1, up to 1.49 (`attia_data_2025`, RMSE).
  157 of 180 attempts scored. 16 tasks had at least one unscored attempt;
  `zhang_enhancing_2026` had 4 of 5 fail, and `matange_deep_2026` and
  `wefers_comprehensive_2026` had 3 of 5. With unscored attempts as 0: mean
  0.796, median 0.955.
- **Low tasks.** Six medians are below 0.5 (0.16–0.46). All six had at least
  one unscored attempt, and all six take raw sequences, structures or atomic
  coordinates. Other tasks with the same input types reach ≈ 1.
- **By field and journal (Figs. 2–3).** Field means range 0.73–1.08 and
  journal means 0.72–0.92 on 3–18 tasks per group. Not tested; the authors
  draw no conclusion.
- **Replicate spread (App. C).** Over the 35 tasks with ≥ 2 scored
  replicates, the median within-task range of SBS is 0.10. Seven tasks have
  a range above 0.2.
- **Validation vs test (App. C).** Over the 153 attempts that printed a
  validation score, Spearman ρ = 0.40 between validation and test SBS.
  Validation exceeded test in 102 of 153. Picking each task's replicate by
  validation score gives mean SBS 0.92, against 0.88 for the median
  replicate and 0.93 for the hindsight best.
- **Compute (Fig. 4).** Means 0.88 / 0.88 / 0.90 / 0.92 / 0.96, none
  significant (smallest adjusted p = 0.16, five-hour arm). Over 45 iterative
  runs the judge's pick and the best-validation round both averaged 0.94,
  against 0.95 for the hindsight-best round. The single and best-of-5 arms
  ran on image builds that redacted the docs; the other arms could read the
  reported score, and in two five-hour attempts the agent used reported
  scores to estimate its own test score.
- **Context (Fig. 5, Table 4).** Medians L1–L8: 0.88, 0.83, 0.82, 0.80,
  0.62, 0.17, 0.00, 0.00. Only L6–L8 differ from L1 (Holm p = 0.039, 0.027,
  0.027). L5 (10% of rows) is −0.118 median paired difference, p = 0.445.
  On `baracat_spiking_2026` L1 scored 0.15 against 1.01 in the main cohort,
  because only the main cohort's README told the agent to fit a separate
  decoder per subject and direction.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest is in the paper and quoted correctly (median 95.5%, mean 85.7%, 12
  of 36). What it leaves out changes how far the numbers carry.
  1. **"Simple" is a resource limit, not a model class.** The prompt asks
     for "a reasonable baseline model". 47 of 157 scripts trained PyTorch
     networks. What is simple is the setting: CPU, 30 minutes, no
     pretrained weights.
  2. **The 95.5% is one test fold against a reported number.** The
     reference is usually the table's best entry, once the best of 488
     configurations, and for 13 multi-fold tasks a single (maybe averaged)
     figure. The DL method is not rerun under the same budget or fold. That
     probably biases SBS *down* (a test-selected reference), but the size is
     unknown.
  3. **Failures are dropped, and the target score was visible.** 23 of 180
     attempts are excluded from medians. The README stated the published
     score in at least 23 of 36 tasks, and every attempt read it. Labels
     never leaked, and no main-cohort script contained the score, but the
     agent knew the number to beat.
  4. **"Known-good sibling of the known-bad preflight" is half right.** The
     paper's trivial predictor *is* the known-bad floor, measured per task
     on the test fold. The auto-baseline is not a known-good probe. Its
     score is not known in advance and varies (median range 0.10, 7 tasks
     > 0.2), so it cannot certify an evaluator. It is a competent
     comparator for crediting a gain, the role
     [[literature/papers/chen2026cads]]'s non-LLM prior plays. The 10-27
     curation made the same correction for chen2026cads.
- **Leakage audit, answering the brief.**
  - *Test labels:* held outside the container, scored outside, logged.
    Clean as far as the logs show.
  - *Reported result:* visible in the README for ≥ 23 tasks (main), and in
    the iterative compute arms it was used. Not visible in the context
    ablations (no README, scores masked in the paper text). On the 4 tasks
    shared with the main cohort, L1 matched the main-cohort median on 3; the
    fourth (`baracat`) differs because of a README modelling hint, not the
    score. That is weak evidence that seeing the target did not drive the
    main result. It is 4 tasks, not a controlled test.
  - *Pretraining:* the papers are 2024–2026, so Opus 5 may have seen them.
    L7/L8 scoring 0.00 rules out recall of test labels from the identifiers.
    It does not rule out recalled method or feature knowledge.
  - *Task selection:* explicitly blind to baseline results, but limited to
    papers with public data and to "the first complete set" meeting quota.
    The README is written from the paper and can carry method hints
    (`baracat`: 0.15 → 1.01).
- **SBS is a normalized ratio, with the known problems.** Mean and median
  differ by 10 points. SBS > 1 values (up to 1.49) pull the mean up. A task
  whose floor is near the DL score (`lv_learning`: 0.862 vs 0.973 PCC) can
  reach SBS ≈ 1 on a small raw gain. The paper gives raw floors and
  references (Table 2) but not raw baseline scores in the main text. See
  [[concepts/pass-at-k]] on normalized headlines.
- **Agent validation is optimistic.** Self-chosen validation splits beat
  test in two of three attempts and rank weakly (ρ = 0.40). The most likely
  reason is that the source papers' test splits are harder (scaffold,
  temporal or out-of-distribution) than a random holdout. The paper does
  not analyse this.
- **Statistics.** No interval or test on the headline. The compute and
  context studies use 9 tasks and 2–5 replicates. The compute comparison is
  confounded by documentation access, which the authors say. The selection
  of the 9 tasks used earlier single-attempt results.
- **Scope.** One model, one provider, supervised tabular-style prediction
  only. No GPU, so tasks needing large models are effectively scored against
  a hobbled baseline, which makes the ≈ 1 results more striking and the low
  ones less informative.
- **What it does not show.** That the published DL methods are wrong or
  useless: SBS compares one number, not interpretability or transfer. Nor
  that an auto-baseline is a reliable *per-task* verdict: the within-task
  spread is often as big as the gaps it would adjudicate.

## Trust signals

- **Credibility:** 3.
  - For: code and a pinned image recipe released; a careful isolation
    design with a full request-log audit that reports what was *reachable*
    as well as what was used; failures, exclusions and the
    documentation-access confound are all disclosed; per-task floors and
    references in Table 2; a task dropped for a label leak in its test
    file; Holm-corrected paired tests on the secondary studies; the
    selection policy is printed in full. CMU and UW authors.
  - Against: preprint, 6 days old, marked "Preliminary version"; one model;
    no interval on the headline; failures excluded from medians; the DL
    reference is a reported (often table-best) number on a different
    protocol; the README exposes the target score; the secondary studies are
    9 tasks.
  - Not 4 because the headline comparison is not same-protocol and has no
    uncertainty. Not 2 because the harness and audit are unusually well
    built and the artifacts are released.

## Follow-up

- **Relevance:** 4. It is the first measurement in this graph of how much
  of a published DL gain a cheap, single-shot, LLM-built baseline recovers,
  across 36 real science tasks. It adds material evidence to:
  - [[concepts/hce-evaluation]]: a measured floor and a competent
    comparator are different checks, and agent validation is optimistic;
  - [[concepts/evolutionary-expansion]]: a single shot reached 91.5% of a
    five-hour iterative budget, n.s. on 9 tasks;
  - [[concepts/pass-at-k]]: validation selection captured most of the
    best-of-5 headroom, and dropped failures move the mean;
  - [[concepts/information-firewall]]: paper text barely mattered once data
    was present, while the README carried both the target score and a
    method hint.
  Not 5 because the headline is not same-protocol and the in-loop use this
  box cares about (a comparator inside `/iterate`) is not tested.
- **`/elevate` bearing (our inference, not tested by the paper).** Two
  candidates, neither ready:
  - A *competent-comparator* clause beside the pending known-bad preflight:
    before crediting a chain's gain, score ≥ 3 replicates of a one-shot
    baseline through the pinned evaluator on the pinned validation split.
    Report the chain's gain as a margin over that mean. Cost ≈ 3 × 30
    CPU-minutes. The replicate spread (median range 0.10) is why one draw is
    not enough, as in [[literature/papers/wang2026rsihub]].
  - For the known-bad preflight itself: compute the floor; don't assume it.
    One task here has a trivial PCC of 0.862, and the paper's policy forbids
    inventing floors ("a training-mean predictor does not generally have
    held-out R2=0").
- **Cheap local test.** On one downstream MLE-style task: run the paper's
  single-attempt prompt 3× against the project's own split, then compare
  the chain's best `metrics.json` with the baseline mean and band. That
  costs no GPU and decides whether the comparator ever overturns a credited
  gain here.
- **Candidates** (cited, not yet in the graph):
  - Ahlmann-Eltze et al. 2025, "Deep-learning-based gene perturbation effect
    prediction does not yet outperform simple linear baselines" (Nature
    Methods).
  - Wolfrath et al. 2024, "Stronger Baseline Models" (arXiv 2409.12116).
  - Wen, Ma & Liang 2026, "Priced Guidance: Can Language Models Generate
    Future Research Ideas?" (arXiv 2610.04976), which measures guidance
    needed to reach a target.
  - Miller et al. 2025, BioML-bench (bioRxiv), and BioXArena (arXiv
    2605.15766).
