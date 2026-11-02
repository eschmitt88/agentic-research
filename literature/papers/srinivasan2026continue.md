---
kind: paper
title: "Continue or Restart? The Limits of Early Failure Prediction in Coding Agents"
authors: ["Srinath Srinivasan", "Lunxiao Li", "Tim Menzies", "Wesley Assunção"]
institutions: ["North Carolina State University"]
year: 2026
venue: "arXiv 2610.35971 (cs.SE), v1 2026-10-26, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.35971"
code_url: null    # no code or analysis scripts linked in the PDF; both trace corpora are public on Hugging Face (Nebius SWE-rebench OpenHands Trajectories, NVIDIA Open-SWE-Traces)
citations: null
source: "raw/papers/srinivasan2026continue.pdf"
added: "2026-11-02"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/context-proprioception]]"
tags: ["early-stopping", "restart-policy", "continue-vs-restart", "failure-prediction", "within-task-auc", "pooled-auc", "task-difficulty-confound", "matched-random-control", "shared-token-cap", "recorded-attempt-replay", "cold-restart", "stall-trigger", "swe-rebench", "open-swe-traces", "openhands", "swe-agent", "trace-mining", "tost-equivalence", "negative-result"]
---

# Continue or Restart? The Limits of Early Failure Prediction in Coding Agents

## TL;DR

The authors mine 147,002 released coding-agent traces from two public
corpora. There are four model–framework settings, all open-weight models on
SWE-rebench-style Python issues. They ask whether the first 5k–20k tokens of
a run can tell a run that will solve its issue from a run on the *same*
issue that will not. Then they replay stopping policies under a hard
token cap.

Three results matter for this graph:

1. **Early prefixes rank tasks, not attempts.** Prefix features lift
   pooled AUC a little (0.57 → 0.61 at 20k on the primary corpus), because
   they track how hard the issue is. Within a task, every predictor sits at
   chance at 20k: count models, frozen embeddings, task-conditional
   logistic regression, cross-setting transfer, and a LoRA-tuned
   Qwen3-0.6B monitor (mean within-task AUC 0.50 [0.48, 0.51], three seeds,
   32,406 attempts). Signal appears only late: 0.52 at 30k, 0.54 at 40k,
   0.56 at 60k on the primary corpus, when 23% of runs are still active.
2. **A learned restart beats a random one only by being a task filter.**
   Under a 75k cap with one cold restart at 20k, learned restart minus
   matched random restart is +1.98 and +0.77 pp in the two Qwen3.5
   settings and −0.46 and −0.66 pp in the other two. An exact
   decomposition puts almost all of it in *which tasks* get stopped
   (between-task term −0.70 to +1.75 pp). The within-task term is −0.11 to
   +0.24 pp.
3. **Continuing beats both restart policies.** At 75k, continuation has a
   strictly higher solve rate and strictly lower mean token cost than all
   56 restart configurations (4 settings × 7 thresholds × 2 policies). The
   gap to learned restart at the median threshold is 4.6–8.1 pp. Over
   four caps it Pareto-dominates 216 of 224 configurations.

The restart tested is cold: a fresh, independently recorded attempt on the
same issue with the same model and scaffold. No redirect, no
state-preserving restart, and no halt-vs-redirect arm is run.

## Claims

- Task identity dominates outcomes. Intraclass correlation of binary
  outcomes is 0.71–0.82 across the four settings. So a monitor that only
  learns issue difficulty can score well on pooled AUC while giving every
  attempt on an issue the same score.
- Early failure prediction should be judged by within-task AUC (pairs of
  runs on the same issue), not pooled AUC.
- At 20k tokens agents have already done real work: on the primary corpus
  87.51% of active runs have run a test command and 32.80% have edited
  code. So chance-level discrimination is not explained by "nothing has
  happened yet".
- Abort-only termination at 20k with within-task AUC ≈ 0.5 is a delayed
  task filter. It is strictly dominated by filtering before execution at
  zero token cost.
- A stopping rule must be compared against matched random stopping *and*
  against uninterrupted continuation, and its gain split into between-task
  and within-task parts.
- Design advice: route and budget before execution, since task difficulty
  is predictable then. Choice of scaffold alone moves solve rate by
  14.06 pp on shared Qwen3.5 tasks (SWE-agent 47.30% vs OpenHands 33.24%,
  CI [12.26, 15.95]). That dwarfs the best restart gain. Mid-run recovery
  should keep state rather than restart cold.

## Methods

- **Corpora (Table I).** Primary: Nebius SWE-rebench, OpenHands +
  Qwen3-Coder-480B-A35B (67,074 runs, 6,306 tasks). Three slices of NVIDIA
  Open-SWE-Traces: SWE-agent + Qwen3.5-122B-A10B (15,098 runs),
  OpenHands + Qwen3.5 (30,957), OpenHands + MiniMax-M2.5 (33,873). The four
  sum to 147,002. Outcome is the benchmark harness's resolved / unresolved
  label.
- **Progress unit.** Transcript tokens: the total length of all recorded
  messages under each setting's own tokenizer. It is a length, not a bill.
  Caching and the cost of re-reading a growing context are ignored, which
  the paper concedes.
- **Landmarks.** b ∈ {5k, 10k, 15k, 20k, 30k, 40k, 60k}. Only runs longer
  than b are eligible at b. 20k is the primary point, since nearly all
  runs are still active there (Fig. 1b).
- **Predictors (RQ1–2).** Eight pre-execution issue count features; plus
  18 prefix behaviour counts (16 on Open-SWE); L2 logistic regression with
  five-fold *unseen-repository* cross-validation. A task-conditional
  logistic model on mixed-outcome tasks only. Frozen all-MiniLM-L6-v2
  embeddings of the issue, the first assistant turn and the first
  observation. A task-rate oracle (empirical solve rate from a held-out
  half of a task's attempts). A Qwen3-0.6B LoRA monitor adapted from
  FailFast–RestartSmart, trained on balanced success/failure pairs from
  the same task with BCE plus a pairwise ranking loss. It runs on the
  primary corpus only, at 20k, on tasks with ≥11 active attempts (11
  sampled per task, 2,946 tasks), three-fold, three training seeds.
- **Statistics.** 95% cluster bootstrap (tasks or repositories), label
  permutation within task as a null check, and TOST equivalence at bounds
  [0.45, 0.55] and [0.47, 0.53]. Holm–Bonferroni only for subgroup and
  multi-landmark checks; the primary-landmark intervals are unadjusted.
- **Replay (RQ3).** Tasks with ≥2 recorded attempts. An initial attempt
  runs to b = 20k. If the policy stops it, a cold replacement is drawn
  uniformly from the *other recorded attempts on the same task*, in the
  same setting. Results are exact expectations over all replacements, not
  sampled, so the replay has no seeds. Only one restart is allowed.
  - **Shared token budget.** A hard cap B ∈ {30k, 50k, 75k, 100k} on the
    initial run plus the replacement. The replacement gets B − b. It
    earns credit only if its recorded run solved the issue within B − b
    (Eq. 2). Its cost is b + min(L_j, B − b) (Eq. 3). Continuation earns
    credit if the original run solved within B.
  - **Learned restart (L).** Stop if the out-of-fold score from the
    pooled issue-plus-prefix count model is below a percentile threshold
    (seven thresholds, 2nd–98th). This score includes the pre-execution
    issue features, so it carries task difficulty by design.
  - **Matched random restart (R).** Stop the same overall fraction of
    active runs at b as L, chosen uniformly at random *across the whole
    setting*. The fraction is matched per setting, not per task.
  - **Continuation (C).** No intervention, same cap B.
  - **Uncapped replay** is also run (runs go to their recorded lengths).
- **Decomposition.** V(L) − V(R) splits exactly into a between-task term
  (task stop rate × task mean net gain) and a within-task covariance term
  (Eqs. 4–6).

## Results

- **RQ1.** Primary corpus, pooled AUC: issue-only 0.56 → issue+prefix 0.58
  at 5k, 0.57 → 0.61 at 20k. Log loss unchanged (0.69). Frozen embeddings
  give pooled 0.62 in both arms, so the embedding adds nothing over the
  issue text. Same-repository cross-task pairs score 0.56 [0.55, 0.59]
  at 20k, against 0.49 for same-task pairs. Task-category labels add about
  +0.02 pooled and < 0.01 within-task.
- **RQ2, counts and embeddings.** Within-task AUC at 5k is 0.50
  [0.49, 0.51]. At 20k all four settings are at 0.49–0.50. The task-rate
  oracle reaches pooled 0.97 with within-task 0.50 by construction, and
  adding prefix counts to it leaves within-task at 0.50 (0.53 at 60k).
- **RQ2, task-conditional.** 0.50 [0.49, 0.51] over 38,447 pairs from
  1,737 mixed-outcome tasks (primary). Open-SWE settings: 0.50, 0.51,
  0.48, all CIs spanning 0.50. Cross-setting transfer ≤ 0.50 in all three
  pairs. TOST equivalence is not claimed for SWE-agent/Qwen3.5 because its
  permutation control drifted from 0.50.
- **RQ2, neural monitor (Table II).** Per fold, issue+prefix within-task
  AUC is 0.500, 0.496, 0.490. Its log loss is 1.788, 1.012 and 0.879, all
  *worse* than the constant 0.693 of the issue-only arm. So the monitor
  is also miscalibrated, not just uninformative.
- **Stratified by task solve rate (Table III).** Low / medium / high
  solvability: 0.513 / 0.488 / 0.496 (counts), 0.504 / 0.491 / 0.503
  (neural). All CIs contain 0.50. Easy tasks are not easier to call.
- **End of run.** Reference-hunk recall (an oracle on the final edit)
  gives within-task AUC 0.62 [0.61, 0.64]. Attempts do diverge, but only
  late.
- **Replay, uncapped.** L and R are within 0.20 pp in all four settings.
  A replacement on the same task keeps the task's expected solve rate.
- **Replay, 75k cap, median threshold (Table IV).**

  | Setting | Continue | Learned | L − R [95% CI] | Between | Within |
  |---|---|---|---|---|---|
  | OH / Qwen3-Coder | 46.15 | 41.53 | −0.458 [−0.677, −0.219] | −0.494 | +0.036 |
  | SA / Qwen3.5 | 28.71 | 20.66 | +1.984 [+1.313, +2.600] | +1.749 | +0.235 |
  | OH / Qwen3.5 | 27.19 | 20.16 | +0.768 [+0.302, +1.221] | +0.881 | −0.113 |
  | OH / MiniMax-M2.5 | 41.05 | 35.76 | −0.663 [−1.014, −0.333] | −0.704 | +0.042 |

  The components sum to the totals. Learned stopping concentrates stops on
  hard tasks, where both replacement and continuation rarely succeed.
  That forfeits fewer continuation successes than random stopping. The
  sign of the net depends on the setting.
- **Efficiency frontier (Table VI).** At 30k: 49 dominated, 7 exact ties.
  At 50k and 75k: 56 of 56 dominated. At 100k: 55 dominated and one
  trade-off (OH/MiniMax, 2nd-percentile threshold, +0.0033 pp for +376
  mean tokens). "Dominated" means at least as good on both solve rate and
  cost and strictly better on one. The "strictly better on both" wording
  is stated only for the 75k cap. Example: on SWE-agent/Qwen3.5,
  continuation solves 28.71% at 68.10k mean tokens and median learned
  restart 20.66% at 69.93k.
- **Why cold restart loses.** The paper gives two budget mechanisms. A
  replacement has only B − b tokens, and at the 30k and 50k caps 91–100%
  of trajectories need more than that. The replacement also re-pays the
  setup and localization the stopped run had already done.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest summary is in the PDF and is quoted correctly. The 0.50
  [0.48, 0.51] figure is the neural monitor at 20k on the primary corpus
  only; the "5k–20k" span comes from the count and embedding models. The
  reading goes past the paper in four places.
  1. **"The first restart-vs-continue comparison at matched tokens."** The
     paper does not claim that. It cites Bulut 2026 (arXiv 2609.03436):
     continuing beats cold restarts under matched budgets in math
     reasoning. It also cites concurrent same-group work (EsfandyariDoulabi
     et al., arXiv 2610.05572) on the same Nebius corpus. The paper's
     contribution is the coding-agent replication, the random control and
     the exact decomposition.
  2. **"Matched tokens" means a matched cap, not matched spend.**
     Continuation and restart share the cap B, and continuation spends
     *less* on average. So the comparison does not favour continuation on
     compute. The unit, though, is transcript length, not billed tokens. A
     long continuation re-reads a long context on every turn, and a fresh
     replacement does not. Under billed cost the "lower token cost" half
     of the result may narrow (our inference; [[literature/papers/kumar2026turncost]] measures
     how far billed cost diverges from token surface).
  3. **"Learned restarts beat matched random by ≤ 1.98 pp" reads as a
     gain.** In two of the four settings learned restart is significantly
     *worse* than random (−0.46 and −0.66 pp, CIs exclude 0). The
     "matched" random policy matches the stop fraction per setting, not
     per task, so any task-difficulty signal in the learned score shows up
     as a between-task gain. The decomposition is the paper's own
     correction for this. The digest summary states the decomposition but
     leads with the gain.
  4. **"Continuation beats both … in 216 of 224."** It is Pareto
     dominance: strictly better on one axis, at least equal on the other.
     Strict on both axes is shown for the 75k cap only. More importantly,
     the dominance is largely budget arithmetic once within-task
     selection is at chance. Nearly every run is longer than 20k, so the
     continuing run has to fit in B and the cold replacement in B − 20k.
     The paper says this itself in its "two coupled budget mechanics". It
     is a careful measurement of how large the cost of cold restart is,
     not an independent finding that continuing is better.
- **What the replay is.** One decision point (20k), one restart, the
  replacement from the same model and scaffold, and replacements are
  recorded runs from agents that never saw a cap. A budget-aware agent, or a
  restart into a different scaffold (worth 14 pp here), could behave
  differently. The policy grid varies only the threshold, not the
  landmark.
- **The neural arm is a weakened reproduction.** It drops the original
  FailFast–RestartSmart test-output supervision and state transfer, and its
  log loss is worse than a constant. The paper discloses both. So the
  three-seed 0.50 shows that this monitor fails. It does not show that
  every learned monitor fails at 20k. The count and conditional models at
  ~0.50 across all four settings carry the general claim.
- **What it shows for redirect-before-halt
  (`docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md`).**
  - *Does not bear on:* redirect against halt, or a forced change of
    approach. The only restart is a cold resample of the same policy. It
    does not cover state-preserving recovery. The paper cites
    FailFast–RestartSmart there: transferring saved edits took resolution
    from 66.6% to 71.8%, while a cold restart gave 66.8%. Nor does it cover
    ML research loops or multi-cycle chains. It also does not test
    triggers built on *realized* outcomes. `/iterate`'s counter counts
    measured non-improvements; it is not a prediction from a partial
    trajectory. So the AUC ≈ 0.5 result does not transfer to that counter
    directly.
  - *Does bear on:* (a) the evaluation bar. A stall trigger should be
    compared against a matched-random trigger and against continuing, with
    the between/within split. Across projects, a counter that fires more
    on hard problems can look useful only because it filters. (b) The form
    of the pivot. A pivot that discards the chain's state and re-pays
    setup is a cold restart under a shared cap, and here that loses every
    time. This agrees with [[literature/papers/zhou2026athanor]], where restart pivots lost to
    the matured lineage. (c) Timing. Within-attempt signal appears late,
    which is weakly consistent with firing at ceiling − 1 after two
    noise-banded failures rather than early.
- **Open questions.**
  - Does continuation still dominate under billed (cached) tokens, and
    against a restart into a different scaffold or model?
  - Is within-task signal at 30k–40k (0.52–0.54) enough to pay for a
    *state-preserving* recovery, where the replacement does not re-pay b?
  - The paper gives no per-threshold L − R numbers beyond the median, and
    it gives no continuation-vs-restart comparison in the uncapped replay.

## Trust signals

- **Credibility:** 4.
  - For:
    - NC State; Menzies is a long-standing empirical-SE author, and
      Assunção leads the related trace-mining line ([8], [9]).
    - Very large n (147k traces, four settings) from public corpora, so
      the data are re-obtainable.
    - Unseen-repository cross-validation, within-task permutation nulls,
      TOST equivalence, cluster bootstrap.
    - An exact decomposition whose components sum to the reported totals
      (checked at ingest for all four settings). The run totals sum to
      147,002, and 224 = 4 caps × 4 settings × 7 thresholds × 2 policies.
    - The abstract is accurate against the body, which is unusual here.
    - A threats section that names the weakened neural monitor, the
      transcript-token unit and the SWE-agent permutation drift.
  - Against:
    - Preprint, not peer reviewed. No code or analysis scripts linked.
    - Offline replay of recorded runs, not live restarts. One landmark,
      one restart.
    - Open-weight models on Python issues only. Primary-landmark CIs are
      unadjusted.
  - Not 5: no released analysis code and no live intervention. Not 3: the
    design is careful, n is large, the data are public, and the claims stay
    inside what the analysis supports.

## Follow-up

- **Relevance:** 4. It is the cleanest test in this graph of whether an
  early, prefix-based stall signal can justify abandoning an attempt. It
  sets the evaluation bar for any stall trigger: matched random, plus
  continuation, plus the between/within split. It adds material evidence
  to three concepts:
  - [[concepts/budget-as-ceiling]]: cold restart under a shared cap loses
    to continuing, and a learned stop beats random only as a task filter.
  - [[concepts/spend-forecast-calibration]]: task difficulty is
    forecastable before execution, attempt outcome is not until late. That
    is admission vs allocation again, for outcomes.
  - [[concepts/context-proprioception]]: an external monitor reading the
    full prefix cannot predict which attempt will succeed, so perception is
    not prediction for outcomes too.

  Not 5: SWE coding rather than an ML research loop, a cold restart rather
  than a redirect, and no redirect-vs-halt arm.
- **For the redirect-before-halt proposal.** The proposal is still without
  a redirect-vs-halt arm at matched compute. This paper supplies the
  evaluation template (random-timed trigger control, continuation arm,
  between/within decomposition) and one negative constraint: a pivot must
  keep the chain's state, or it pays the cold-restart penalty measured
  here.
- **Pairs with** digest item 7 (Jain & Vedam, arXiv 2610.36544, a
  decision-value audit for restart monitors with an abstain option). Not
  yet in the graph.
- **Candidates** (cited, not yet in the graph):
  - Bulut 2026, "It's the problem, not the path" (arXiv 2609.03436).
    Continuation beats cold restart under matched budgets in math
    reasoning.
  - EsfandyariDoulabi et al. 2026, "Disentangling task difficulty from
    run-level failure in agent failure prediction" (arXiv 2610.05572). Same
    group, same corpus, abort-only stopping.
  - Lyu et al. 2026, "Fail-fast, restart-smart" (arXiv 2608.03222). The
    state-preserving recovery result (66.6% → 71.8%) that this paper
    leans on and does not reproduce.
  - Mehtiyev & Assunção 2026, "Beyond resolution rates" (arXiv
    2604.02547). Trajectory length vs failure once task identity is
    controlled.
