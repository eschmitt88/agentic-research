---
kind: paper
title: "CompAct-Bench: A Benchmark for Working-Context Compaction in Long-Horizon Agent Tasks"
authors: ["Jiayi Chen", "Lizhe Chen", "Yujia Chen", "Shihan Dou", "Ming Zhang", "Chendi Ge", "Shiguang Ni", "Tao Gui", "Qi Zhang", "Xuanjing Huang", "Maxm Pan"]
institutions: ["Fudan University", "Hunyuan Team, Tencent", "Shenzhen International Graduate School (SIGS), Tsinghua University"]
year: 2026
venue: "arXiv 2611.25673 (cs.AI), v1 2026-11-19, preprint (11-page main text + 47-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.25673"
code_url: null    # the PDF names a project website (compaction-eval.github.io/CompAct-Bench); not opened at ingest, code release not established
citations: null
source: "raw/papers/chen2026compactbench.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/file-as-bus]]"
tags: ["compaction", "compaction-benchmark", "post-compaction-continuation", "matched-action-budget", "retention-ratio-control", "iterative-ratio-calibration", "browsecomp", "swe-bench-verified", "companybench", "task-state-persistence", "materialized-state", "re-observability", "epistemic-status", "premature-commitment", "omission", "budget-exhaustion", "llm-annotation", "no-full-history-arm", "twelve-compactors", "claude-opus-5"]
---

# CompAct-Bench: A Benchmark for Working-Context Compaction in Long-Horizon Agent Tasks

## TL;DR

The authors take successful agent trajectories, cut each one at 25%, 50%
or 75% of its actions, and have a model under test compress everything
before the cut to **10% of its characters**. A fixed executor then resumes
from the restored environment state with the original task, the summary,
and **exactly as many actions as the successful run had left (T − b)**. 200
instances: 79 BrowseComp (Search), 96 SWE-bench Verified (Coding), 25 from
an internal CompanyBench (Workspace). Twelve frontier models compact. The
executor is Claude Opus 4.8 for Search and Coding and GPT-5.5 Codex for
Workspace. Every source task was solved in 3 of 3 screening runs.

Five results matter for this graph:

1. **Post-compaction success is 46.0–68.5%, mean 56.7%.** Claude Opus 5 is
   best (68.5 [62.0, 74.5]), Doubao Seed 2.1 Pro worst (46.0 [40.0, 52.5]).
   There is **no full-history continuation arm** under the same budget, so
   how much of the gap to 100% is compaction and how much is a fresh rollout
   under a tight budget is not measured.
2. **Where the task state lives decides what the summary is worth.** With
   no history and no summary, Coding resumed from the restored repository
   scores 66.7 / 58.1 / 71.9% at the three boundaries, against 48.5 / 35.5 /
   15.6% from the initial repository. The summary adds −2.7, +14.6 and +14.2
   points on top. In Search, where the open web keeps no task state, it adds
   +26.0 to +27.2 at every boundary. So Coding gets *easier* at later cuts
   (64.0 → 86.1%) and Search harder (52.9 → 34.6%).
3. **Most failures are omission → re-acquisition → budget exhaustion.**
   91.6% (Search), 92.0% (Coding) and 97.7% (Workspace) of failed
   continuations used up their actions. Critical omission goes with +18.2
   points more repeated work, and the executor tries to recover lost state in
   95.1% of applicable failures.
4. **Status distortion is real but the smaller route.** An LLM annotator
   labels 69.1% of inventory units preserved, 25.3% omitted and **5.6%
   distorted**. "Premature commitment" (an unresolved hypothesis stated as
   settled) appears in 56.7% of early failures against 46.0% of
   budget-exhausted ones. Early failures are only about 2–8% of failures.
5. **The case studies are the strongest qualitative evidence in the graph
   that a carried fact's status matters, not just its presence.** In one
   Search task all twelve summaries contain the same disqualifying fact.
   The three that write it as a decision ("set aside"), and one that lists
   it with no next step, all succeed. Five of the eight that write it as an
   open question exhaust the budget re-checking it (Case F). In another, an assumption the agent had
   introduced into its own queries ("gold-rush town") is carried by all
   twelve compactors as if it were part of the task (Case C).

## Claims

- Working-context compaction is an execution-critical capability distinct
  from summarization, and current models are substantially lossy at it.
- Difficulty is not explained by history length. Within a domain, prefix
  length is uncorrelated with success (|ρ| ≤ 0.03). It depends on where task
  state is kept (environment vs context) and on how faithfully the
  remaining state, including its epistemic status, is carried.
- Two failure pathways: omissions lead to repeated work and budget
  exhaustion; state distortions lead to unverified commitment and premature
  termination.
- Failures are not a weak-model tail: the four strongest compactors jointly
  fail 15.2% of Search, 7.3% of Coding and 32.0% of Workspace tasks.
- The construction generalizes to any benchmark with resumable environment
  state and an automatic evaluator, with no reference summaries needed.

## Methods

- **Source tasks.** Run the fixed executor 3 times per sampled task. Keep
  tasks solved in all three (pass^3) with ≥8 rounds. One successful rollout
  becomes the reference trajectory. Mean trajectory: Search 35.3K tokens /
  30.4 rounds; Coding 8.8K / 19.4; Workspace 122.8K / 58.5.
- **Boundaries.** One boundary per trajectory (cost control), stratified by
  length tertile within each benchmark. 26–33 tasks per Search/Coding cell,
  8–9 per Workspace cell. Mean compaction input grows with the boundary:
  Search 8.2K → 29.3K tokens, Coding 2.3K → 5.5K, Workspace 41.2K → 90.0K.
- **Compaction.** One shared prompt for all models: "Please summarize the
  above history concisely. Target length: approximately {r}% … Preserve key
  facts, decisions, tool call results, and context needed for continuing …
  Focus on tool call and tool response." The system prompt and task are a
  verbatim retained head outside the budget (about 30K characters for
  Workspace). Each model runs at its strongest reasoning setting.
- **Iterative Ratio Calibration (IRC).** To match lengths, Stage 1
  repeatedly asks the compactor to shrink **its own current candidate**.
  Stage 2 then gives it the full history and the best candidate, and asks
  for small edits. Up to K = 30 calls, tolerance ±3% relative. Over 2,400
  compactions the median realized ratio is 0.0986 and 86.7% land in
  tolerance (Table 9's "All" row says 86.0%). Workspace adherence is 59.7%.
  Mean calls per compaction: 4.6 (Kimi K3) to 11.8 (Opus 5).
- **Continuation.** The environment is rebuilt by replaying the reference
  run's first b tool calls (Coding, Workspace). Search needs none. The
  executor gets the head, the summary and a domain-specific resume line.
  Budget: at most T − b executor actions. Each summary is generated once and
  continued 3 times (Avg@3). Strict success means within budget, a
  successful terminal status, and acceptance by the benchmark evaluator
  (Gemini 3.5 Flash judge for Search, repository tests for Coding, the
  native evaluator for Workspace).
- **Persistence controls (App. F).** "Without H_b": the executor gets only
  the head and the boundary state, with no history and no summary, on
  Search and Coding. Plus a from-initial-state arm for Coding.
- **Error annotation (App. G).** GPT-5.6-Sol, at high effort, in three
  blinded stages.
  - A: list the 17–44 units needed to continue, from H_b alone, typed as
    factual / progress / epistemic status / constraints-and-goals.
  - B: label each unit in the summary preserved / omitted / distorted /
    uncertain, plus six error labels (CO, FD, PC, LR, PSL, CGD), with the
    continuation hidden.
  - C: code seven behaviours in the continuation, with the summary hidden.

  Quotes must occur verbatim. Human review covered "a subset", with no size
  or agreement reported. Statistics cover **failed continuations only**.
- **Statistics.** Stratified task-level bootstrap (10,000 resamples) for
  every model × domain. Spearman for rank associations, Mann–Whitney for
  two-group contrasts.

## Results

- **Main table (Table 2 / 12), strict Avg@3 %:**

  | Compactor | Search | Coding | Workspace | Overall |
  |---|---|---|---|---|
  | Claude Opus 5 (Max) | 62.0 | 81.2 | 40.0 | **68.5** |
  | Gemini 3.8 Flash (High) | 57.0 | 75.0 | 56.0 | 65.5 |
  | Kimi K3 (Max) | 50.6 | 79.2 | 32.0 | 62.0 |
  | GPT-5.6-Sol (Max) | 51.9 | 76.0 | 20.0 | 59.5 |
  | … (8 more) | 31.6–44.3 | 66.7–77.1 | 8.0–44.0 | 46.0–57.8 |

  The top and bottom Overall intervals do not overlap. Workspace intervals
  are about ±20 points (n = 25). Cross-domain Spearman over 12 models: 0.62
  (Search–Coding), 0.53 (Search–Workspace), 0.26 (Coding–Workspace).
- **Boundary trends (Table 13).** Coding: every compactor peaks at 75%
  (81.2–90.6%). Ten of twelve rise monotonically. Search: 11 of 12 are lower
  at 75% than at 25%, and late Search spans 7.4% (GLM 5.2) to 55.6% (Opus
  5). Workspace: 9 of 12 peak at 50%.
- **Persistence (Table 14).**

  | | 25% | 50% | 75% |
  |---|---|---|---|
  | Coding, no H_b, initial repo | 48.5 | 35.5 | 15.6 |
  | Coding, no H_b, boundary repo | 66.7 | 58.1 | 71.9 |
  | Coding, with M_b (12-model mean) | 64.0 | 72.7 | 86.1 |
  | Search, no H_b | 26.9 | 15.4 | 7.4 |
  | Search, with M_b | 52.9 | 42.6 | 34.6 |

  Observationally, Coding tasks with a file edit before the boundary score
  86.9% against 53.0% without (p = 3 × 10⁻⁸). Prior test runs add nothing
  given an edit (86.5% vs 87.2%). The edit share rises with the boundary
  (30% → 71% → 88%), so edit and stage are confounded. At the late Coding
  boundary 72% of tasks are solved by every compactor, and 41% of successes
  finish within two actions.
- **Failure composition (App. E.4).** Budget exhaustion is 91.6% / 92.0% /
  97.7% of failed continuations. Early failures cluster at the 25% boundary
  (20.8% of Search failures, 14.8% of Coding). Successful continuations use
  a median 55% / 86% / 93% of the source remainder. When the answer string
  survives into a Search summary (8% of contexts), success is 60% against
  42%. Remaining budget does not predict Search success (ρ = 0.15, 0.00,
  0.35, none significant).
- **Errors in failed continuations (Tables 16–17).**

  | Error | Early failure | Budget exhaustion |
  |---|---|---|
  | Critical omission (CO) | 47.8 | 48.8 |
  | Factual distortion (FD) | 40.3 | 35.2 |
  | Premature commitment (PC) | 56.7 | 46.0 |
  | Lost resolution (LR) | 0.0 | 3.6 |
  | Progress/state loss (PSL) | 55.2 | 44.6 |
  | Constraint/goal distortion (CGD) | 25.4 | 24.3 |
  | Any | 92.5 | 85.8 |

  Behaviour shifts with an error present vs absent, among failures:
  - repeated work: CO +18.2, LR +14.2, PSL +13.8;
  - unverified commitment: CGD +10.5, FD +7.3, PC only +5.2;
  - PC goes with −10.9 on constraint deviation.
- **Cases (App. H, selected for clarity).**
  - **A:** small settled state. Every summary works, in 2–5 actions against
    the source's 11, partly from parametric recall.
  - **B:** a 700K-character, 145-round Search prefix. 11 of 12 summaries
    still lead to success. GPT-5.6-Sol's summary promoted two results the
    source agent had seen and never opened.
  - **C:** asserted summaries lead to 2-round wrong submissions. Hedged ones
    lead to longer searches that still close on the next-best wrong
    candidate. Only the two that kept alternatives alive succeed, and one of
    those found a web page reproducing the benchmark question.
  - **D:** faithful summaries, a 140-action budget, and 11 of 12 compactors
    still fail early. What did not cross the cut was the source agent's
    *implicit* rule of testing the candidate against the one discriminating
    clue before submitting.
  - **E:** a 3-action budget. The answer appeared only inside results the
    agent had labelled spam. Nine summaries drop it, three keep it as spam,
    and every continuation fails.
  - **F:** the decision-vs-question contrast in TL;DR item 5.
  - **G:** a SWE-bench task where the helper function was located but no
    edit made. The 4 summaries that omit it all fail, re-locating it with 4
    of 8 actions. 4 of the 8 that name it succeed.

## Critique / open questions

- **Against the digest entry: at, with two overreaches.**
  - The PDF is the entry's work: arXiv 2611.25673, 11 authors, v1 dated
    19 Nov, BrowseComp / SWE-bench / internal CompanyBench, retention
    controlled. It is **not** the 2610.31710 "Compact-Bench" listed as a
    near miss in [[literature/papers/salle2026benchmarks]].
  - **"Compaction stays highly lossy even for frontier models"** rests on
    the pass^3 screen, not on a measured no-compaction arm. App. C.2 says
    the benchmark "measures robustness of a successful execution to
    replacing accumulated history with M_b". The full-history continuation
    under the same T − b budget is the missing control, so the 31.5–54
    point "loss" mixes compaction with rollout variance under a budget set
    by one successful run. Some budgets are tiny (Case E: 3 actions).
    Fresh continuations often beat the source's own remainder (Cases A, B),
    so the bias's direction is not obvious either.
  - **The summary's headline failure is the minor route.** "Unresolved
    issues turned into apparent conclusions" (PC) is the paper's
    attention-grabbing finding. Early failures are 2–8% of failures, PC is
    nearly as common among budget exhaustions (46.0% vs 56.7%), and
    distortion is 5.6% of unit labels against 25.3% omitted. The dominant
    route is omission, re-acquisition and running out of actions.
  - "Established results forgotten" maps best to progress/state loss.
    The paper's "lost resolution" label (a settled question reopened) is
    rare (0% and 3.6%), apart from Case F.
- **The error statistics have no base rate.** All error and behaviour rates
  are computed over failed continuations. How often successful
  continuations carry the same errors is not reported, so "92.5% of early
  failures inherit an error" cannot be read as a cause. The annotator is one
  of the twelve compactors, identity-blinded. Human validation is "a subset"
  with no count or agreement.
- **One generic prompt at one ratio.** "Summarize … concisely" at 10% of
  characters is a harsh setting next to production compaction prompts (for
  example Claude Code's structured sections). The absolute numbers should not
  be carried to any harness. App. B.6 calibrates other ratios but runs no
  downstream continuations at them.
- **IRC is itself repeated compression.** Stage 1 shrinks the model's own
  previous output, the note-from-note step
  [[literature/papers/liu2026agents]] found costly. Stage 2 then re-supplies
  the full history, which is liu's fix. The ranking measures "model + up to
  30 calibration rounds", and the models used very different numbers of
  rounds (4.6–11.8). How the procedure interacts with quality is not tested.
- **Confounds the paper names, and two it doesn't.**
  - Named: stage vs edit presence. The edit indicator is observational and
    rises with the boundary.
  - Not named: the executor for Search and Coding is Claude Opus 4.8 and the
    best compactor is Claude Opus 5, so a same-family style match is
    possible. GPT-5.6-Sol is weak on the GPT-executed Workspace (20.0%),
    which argues against a simple family effect.
  - Not named: the late-Coding "improvement" is partly little work left.
    41% of successes finish within two actions.
- **Thin cells.** Workspace boundary cells hold 8–9 tasks, and Workspace
  intervals span ±20 points. CompanyBench is internal and unreleased.
  Cross-domain correlations rest on 12 points.
- **Case studies are selected** "for mechanistic clarity". Case F is the
  cleanest status contrast but is one task. Its "settled" contexts succeed
  in 4, 28 and 36 rounds, so "fast success" holds for one of three.
- **What it adds against neighbours.**
  - [[literature/papers/fan2026empirical]] and
    [[literature/papers/salle2026benchmarks]] found explicit archives
    redundant in workspace-backed tasks. This paper measures the
    environment's share directly (the no-H_b arm) across 12 compactors, and
    shows that a re-derivable omission is still paid for in actions.
  - [[literature/papers/chu2026when]] measured sentences a summary adds.
    Here a frontier compactor set (Opus 5 included) carries an
    agent-introduced assumption as task (Case C). That is one case, not a
    rate.
  - [[literature/papers/liu2026agents]] found a confirmed / ruled-out /
    open ledger instruction did not help in toy worlds. This paper shows the
    status matters to the reader, but tests no instruction that would set
    it.

## Trust signals

- **Credibility:** 3.
  - For:
    - Fudan NLP (Tao Gui, Qi Zhang, Xuanjing Huang) with Tencent Hunyuan.
    - Twelve current frontier compactors, three domains, length-matched
      outputs with reported adherence.
    - Task-level bootstrap intervals for every cell.
    - A persistence control (no-H_b, two environment states).
    - An explicit alternative-explanations table, and candid scoping of
      what the conditioning on pass^3 means (App. C.2).
  - Against:
    - Unreviewed, five days old, code not established.
    - No full-history arm under the matched budget.
    - Error analysis by one LLM annotator, failures only, with
      unquantified human review.
    - One prompt, one ratio, one executor per domain.
    - The third domain is internal.
    - A small internal inconsistency (86.7% vs 86.0%).
  - Not 4: unreviewed, and the headline loss lacks its control. Not 2: a
    reputable group with careful, well-instrumented controls on most
    secondary claims.

## Follow-up

- **Relevance:** 4.
  - The first multi-model benchmark in the graph that measures compaction
    by whether execution survives it, with output length held fixed. It
    bears on context-eviction-policy's open "compaction fidelity" question.
  - It measures lossless-context-offload's re-observability condition
    across domains.
  - Not 5: one generic prompt at a harsh ratio, no full-history control,
    no intervention on the compaction itself, and no ML-research tasks.
- **For `/elevate` (narrow).**
  - The PreCompact hook's "persist anything load-bearing to disk" gets
    indirect, observational support. Materialized edits are the strongest
    predictor of post-compaction success in Coding (86.9% vs 53.0%), and the
    restored repository alone carries 58–72%. These are code edits, not
    notes, so this is not a test of writing findings to disk.
  - "Unresolved → concluded" is not yet gate-grade evidence for a new
    bullet: LLM-labelled, failures-only, no base rate, and the minor
    failure route. liu2026agents' ledger null cuts against an instruction
    fix.
  - For any box evaluation of a compaction change: include a full-history
    arm under the same action budget, and score a carried fact's status as
    well as its presence.
- **Candidates** (cited, not yet in the graph):
  - Chen, Pan, Dai & Netravali 2026, "Slipstream: trajectory-grounded
    compaction validation for long-horizon agents" (arXiv 2605.08580).
    Also listed in chu2026when's follow-ups.
  - Kang et al. 2026, "ACON: optimizing context compression for
    long-horizon LLM agents" (ICML 2026). Its prompts are already tested in
    zha2026do, but it has no note of its own.
  - Li et al. 2026, "CompactionRL" (arXiv 2607.05378). RL with compaction
    in the loop.
  - Tirmazi, Markelon, Bishop & Mitzenmacher 2026, "Context compaction
    theory" (arXiv 2608.01326).
  - Wang et al. 2026, "Context compression for LLM agents: a survey of
    methods, failure modes, and evaluation" (Preprints 202605.2065).
