---
kind: paper
title: "Pro² Agent: Scientific Problem Probing for Autonomous AI R&D"
authors: ["Yixiong Chen", "Liqian Peng", "Tian Tan", "Ziqian Lin", "Blake Jianhang Chen", "Chao Zhao", "Yaojie Liu", "Alec Go", "Alan Yuille"]
institutions: ["Google", "Johns Hopkins University"]
year: 2026
venue: "arXiv 2611.25357 (cs.AI), v1 2026-11-19, preprint (17-page main text, no appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.25357"
code_url: null    # no code, prompts or run logs named in the PDF
citations: null
source: "raw/papers/chen2026pro2.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/structured-world-model]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/hce-evaluation]]"
tags: ["mle-bench", "mle-bench-lite", "posttrainbench", "problem-state", "research-state", "hypothesis-testing", "belief-update", "ablation", "single-run-headline", "best-in-hindsight", "web-retrieval", "search-time-contamination", "pi-agent", "reviewer-agent", "git-backed-state", "mcts"]
---

# Pro² Agent: Scientific Problem Probing for Autonomous AI R&D

## TL;DR

Pro² keeps an explicit **Problem State** alongside the codebase. It is a
list of diagnosed performance bottlenecks. Under each bottleneck sit
candidate fixes, each with a belief score in [0, 1] and its evidence.
The state is built before any training code, from runtime profiling and
a web/literature search keyed to the bottlenecks. It seeds the first
solution. Single-variable hypothesis tests then raise or lower the
beliefs, and a final MCTS phase tunes hyperparameters only. A Gemini 3.1
Pro "PI" agent keeps the state. Gemini 3.5 Flash implements and reviews.

Four results matter for this graph:

1. **The headline is one run, beside rows that average several.** 70.7%
   any-medal on all 75 MLE-bench tasks at 24 h (53 tasks) and 76.0% at
   48 h (57 tasks), one run per budget. The 70.7% ties AIBuildAI-2 (Opus
   4.7, A100, 24 h). Seven comparator rows are not multiples of 1/75
   (e.g. ScienceFlow 70.2, Iris 64.9, AIBuildAI 63.1), so they are means
   over runs (our arithmetic). Budgets, GPUs and models differ by row.
   The 48 h "highest reported" is twice the standard budget. Its gain
   over 24 h is 4 tasks (3 bronze, 1 gold), and above-median is
   unchanged.
2. **The controlled evidence is a 3-run ablation on MLE-bench Lite**
   (22 tasks, 24 h, same models and budget; mean ± SE). Out of 66
   task-runs, medals at the first solution / at the end of the budget:

   | Variant | Initial | Final |
   |---|---|---|
   | Codebase State | 7 (10.6%) | 37 (56.1%) |
   | Problem State, no grounding | 31 (47.0%) | 51 (77.3%) |
   | Static (grounded, never updated) | 37 (56.1%) | 55 (83.3%) |
   | Full Pro² | 38 (57.6%) | 59 (89.4%) |

   Most of the gain is in **building the state before the first code**.
   Updating it in the loop adds 4 task-runs of 66 at the end (about 1.3
   tasks per run).
3. **The node-indexed metrics are best-in-hindsight.** Medal@3, Medal@5,
   Node AUC and the ablation's "Final Medal" score the best node reached
   so far. They are not the artifact the run submits. The paper doesn't
   say whether that node's score is validation or hidden-test. Its
   Figure 4 plots validation scores against the Kaggle test thresholds.
4. **Grounding means open web search on public Kaggle tasks, with no
   reported blocklist.** Retrieval alone (Search-Augmented vs Direct
   Coding Loop) moves first-solution medals from 12.1% to 28.8%. The
   paper's "expert technique recall" counts overlap with Kaggle winning
   solutions. Nothing in the paper rules out that search found those
   write-ups.

## Claims

- The usual AI-research agent's primary state is the codebase, its
  score and its edit history. That state records what was tried, not why
  the task is hard, so the agent stays in the wrong solution region.
- Keeping an explicit, evidence-grounded, revisable Problem State
  produces stronger first solutions and faster convergence. It also
  keeps converting extra research time into medals instead of
  saturating (24 h → 48 h).
- Evidence grounding and continual revision each add value beyond a
  static plan (Table 4).
- The gain is in breadth: more bronze and silver tasks, not more golds
  (MARS+ gold 33.8% vs Pro²'s 25.3%).
- Pro² with a Flash implementer is "comparable to Claude Code powered by
  Opus 4.7" on PostTrainBench.
- Each run leaves an inspectable research record, with each bottleneck
  ending resolved, resource-limited or unresolved.

## Methods

- **State (Eq. 1).** S_t = {P_i, E^p_i, {C_ij, b_ij, E^c_ij}}. P_i is a
  bottleneck and E^p_i its profiling evidence. C_ij is a candidate fix
  with belief b_ij and evidence E^c_ij from literature or runs. Every
  state version maps one-to-one to a git snapshot. So the Problem State
  sits *beside* the codebase; it does not replace it.
- **Phase 1, initialization.** Probe scripts run inside the sandbox on
  the raw data and measure latency, memory and structure. A visual
  diagnosis is added for multimodal tasks. The results are extrapolated
  to full scale against the compute budget, which yields bottlenecks
  such as OOM or I/O starvation. Each bottleneck maps to web and academic
  search queries. Retrieved fixes become candidate hypotheses with
  initial beliefs. A baseline is built, sanity-checked on mini-batches
  and committed.
- **Phase 2, scientific exploration (Alg. 1).** Best-first over a tree of
  hypothesis branches. Priority = b / (N(P) + 1), where N(P) counts past
  attempts on that bottleneck. Each hypothesis names a target gain
  δ_target. A Reviewer agent checks single-variable isolation. The branch
  is run once.
  - If Δh ≥ δ_target, it is merged and b rises by Δb_pos.
  - Otherwise, or on a crash, it is rolled back and b falls by Δb_neg.

  The insights are then propagated to re-score pending hypotheses. The
  values of δ_target, Δb_pos, Δb_neg and γ are **not reported**.
- **Phase 3, engineering.** UCT-MCTS over hyperparameters, schedules and
  ensemble weights, rewarded by Δh − γ·ln(T_exec). An "audit firewall"
  rejects any diff the Reviewer maps to model, algorithm or data
  components. The highest-validation trajectory is merged.
- **Agents.** The PI agent (Gemini 3.1 Pro) owns the state, sets goals
  and budgets, and inspects the implementer's trajectory every 10 steps.
  The Implementation agent (Gemini 3.5 Flash) is a ReAct loop. The
  Reviewer (Gemini 3.5 Flash) audits code before any full run. An
  Experience Bank stores error signatures and fixes. A sentinel kills
  runs on NaN or stalled I/O.
- **Grading.** Validation is an HTTP service outside the sandbox, backed
  by isolated labels. Final scores come from MLE-bench's hidden test.
  Submissions that used test-set feedback are excluded, following the
  leaderboard.
- **Evaluations.**
  - Full MLE-bench: 75 tasks, one A100 per task, 24 h and 48 h, **one run
    per budget**. The budget includes analysis, web research and
    training.
  - MLE-bench Lite (22 tasks): 24 h, **3 runs per task**. The matched
    variants are a Direct Coding Loop (no Problem State) and a
    Search-Augmented agent (retrieves methods, no problem organisation).
    Both use the same models, tools, environments, splits and budgets.
    The Table 4 "Codebase State" variant is not separately described,
    and its initial medal rate (10.6) differs from the Direct Coding
    Loop's (12.1).
  - MARS rows come from its public trajectories (Gemini 3 Pro Preview),
    so they are not model-matched.
  - PostTrainBench: 28 configurations × 3 runs at 24 h on an A100. The
    published rows ran 10 h on an H100.
  - Expert-technique recall: 5 hardest tasks, 48 h, against a "w/o
    Problem State" baseline. The technique lists come from Kaggle
    winning solutions. The matching procedure is not described.
- **Not reported:** token or API cost, number of runs behind the 48 h
  row (or whether it continues the 24 h run), prompts, belief-update
  constants, code.

## Results

- **Full MLE-bench (Table 1).**

  | | Valid | >Median | Bronze | Silver | Gold | Any |
  |---|---|---|---|---|---|---|
  | Pro² 24 h | 100 | 84.0 | 25.3 | 21.3 | 24.0 | **70.7** |
  | Pro² 48 h | 100 | 84.0 | 29.3 | 21.3 | 25.3 | **76.0** |
  | AIBuildAI-2 (Opus 4.7, A100, 24 h) | – | – | – | – | – | 70.7 |
  | ScienceFlow (DeepSeek-V4-Flash, ≤2 GPUs, 24 h) | – | – | – | – | – | 70.2 |
  | MLEvolve (Gemini 3.1 Pro, H200, **12 h**) | 100 | 76.0 | – | – | 34.7 | 65.3 |
  | MARS+ (Gemini 3 Pro, **2×H100**, 24 h) | 100 | 74.2 | 12.4 | 16.4 | 33.8 | 62.7 |

  - In counts: 53 medals at 24 h (19 / 16 / 18) and 57 at 48 h
    (22 / 16 / 19). Above-median is 63 tasks in both.
  - Pro²'s bronze + silver is 50.7% against MARS+'s 28.8%, while its gold
    rate is lower (25.3 vs 33.8).
- **PostTrainBench (Table 2).** Weighted average **30.69 ± 3.1**.
  - That is 7th of 10 agent rows. Above it: Fable 5 41.79, GPT 5.6 Sol
    36.23, Opus 5 34.06, Opus 4.8 32.90, Kimi K3 31.96, GLM 5.2 31.70.
    Below it: Opus 4.7 28.56 ± 1.7, GPT 5.5 27.23 and Gemini 3.1 Pro
    (OpenCode) 21.99. Official instruct models score 51.14.
  - Pro² beats Opus 4.7 on 5 of 7 sub-benchmarks, with a 2.4× longer
    budget on a different GPU.
- **Lite, first solution and early trajectory (Table 3, 3 runs, ± SE).**

  | | >Median | Any medal | Gold | G_bronze ↓ | Medal@3 | Medal@5 | Node AUC |
  |---|---|---|---|---|---|---|---|
  | Direct Coding Loop | 25.8 | 12.1 | 4.5 | 1.56 | 21.2 | 30.3 | −0.60 |
  | Search-Augmented | 39.4 | 28.8 | 12.1 | 1.23 | 30.3 | 36.4 | −0.44 |
  | MARS (public runs) | 50.0 | 28.8 | 12.1 | 0.96 | 34.8 | 39.4 | −0.39 |
  | Pro² | **81.8** | **57.6** | **27.3** | **−0.57** | **63.6** | **66.7** | **0.58** |

  - First-solution build time: Pro² 11.2 min vs MARS 14.8 min.
  - Medal@k counts the best of the first k nodes. MARS evaluates more
    than 70 repository variants per task, so a "node" is not the same
    unit of cost across methods.
- **Lite ablation (Table 4).** Initial / final medal rate / Node AUC:
  - Codebase State: 10.6 ± 1.5 / 56.1 ± 3.0 / 0.06.
  - Problem State without grounding: 47.0 ± 1.5 / 77.3 ± 2.6 / 0.27.
  - Static Problem State: 56.1 ± 1.5 / 83.3 ± 3.0 / 0.45.
  - Full Pro²: 57.6 ± 1.5 / 89.4 ± 1.5 / 0.66. That is 20, 20 and 19
    medalled tasks of 22 in the three runs.

  Our arithmetic, in task-runs of 66:
  - Building the state lifts the first solution by +24 (7 → 31).
  - Grounding adds +6 initial and +4 final.
  - Updating adds +1 initial (≈ 0 by construction) and **+4 final**.
  - The paper's SEs imply a per-run SD of roughly 2.6–5.2 points.
- **Technique discovery (Fig. 5, 5 tasks, figure only).** Pro² starts
  with higher recall of winning-solution techniques on every task. On
  some tasks it reaches about 0.8–0.9 by 48 h, read off the plot. At
  equal recall its solution quality is higher, judging from regression
  lines over a few dozen points.
- **Cases (Fig. 6, one trajectory each).**
  - BMS molecular translation: bronze, test score 1.8860 against a
    validation score of 1.7257. No AIRA or MARS run the authors
    inspected medalled on BMS.
  - RSNA mammography: above median. The larger model ran past the 4 h
    per-hypothesis timeout, and the state logged it "resource-limited".
  - Gemma3-4B GSM8K: 0 → 56.5% via CoT SFT, then GRPO.
  - Qwen3-1.7B AIME: 6.67%, with the bottleneck logged "unresolved".

## Critique / open questions

- **Against the digest entry: above.** The bullet's numbers are right.
  Its cut reason ("method-first") fits the headline but misses the 3-run
  matched ablation on Lite, which is the useful part.
  - "Ties the best MLE-bench medal rate" is one run, compared with rows
    that are partly multi-run means. Those rows differ in model, GPU
    (A100 vs 2×H100 vs H200) and budget (12 h vs 24 h). It ties one
    system at 53 of 75 tasks. ScienceFlow's 70.2 is 0.5 points, under
    one task, away.
  - "76.0% at 48 h" doubles the budget relative to every row it is
    ranked against. The paper's "continues to turn time into medals
    rather than saturating" rests on 4 tasks at one run each. It does
    not say whether the 48 h row is the same run continued. The
    identical above-median (63) and silver (16) counts are consistent
    with a continuation.
  - "Replaces the codebase as the loop's primary state" is the paper's
    framing. Mechanically the state is coupled one-to-one to git
    snapshots and sits beside the code.
- **What "Problem State as primary loop state" adds beyond
  [[concepts/structured-world-model]].** Hypothesis-and-evidence states
  are already in the graph. [[literature/papers/wang2026pear]] uses
  support/refute counters. [[literature/papers/woo2026youra]] uses
  `verification_state.yaml`, with an ablation. Arbor's hypothesis tree
  ([[literature/papers/jin2026toward]]) has git worktrees and a held-out
  gate. Two things here are new:
  1. **Indexing.** The state is keyed by diagnosed *bottleneck*, with
     fixes nested under each. It is built before any solution code,
     from measurement as well as literature.
  2. **A decomposition ablation on MLE-bench Lite** that separates
     building, grounding and updating at 3 runs.

  The ablation does not support the "primary loop state" framing
  strongly. The large step is building a diagnosis before the first
  code. In-loop revision adds about 1.3 tasks per run. A static plan
  already reaches 83.3% final. This is closer to "diagnose before you
  code" than to "keep a state as the loop's spine".
- **The Codebase State arm is underspecified.** It is not described, and
  its initial rate differs from the described Direct Coding Loop's. So
  it is unknown whether it got profiling, the PI agent, the Experience
  Bank or Phase 3. Without that, 10.6 → 47.0 cannot be credited to the
  state rather than to a planning phase.
- **Budget-matched in wall clock, not in tokens.** The Lite variants
  share models and the 24 h budget. Token and API spend are never
  reported for any arm. The Problem State arms add a Gemini 3.1 Pro PI
  that inspects every 10 steps.
- **Best-in-hindsight metrics** (see TL;DR 3). Medal@k, Node AUC and
  "Final Medal" take the best node so far. If that node is picked by
  test score, they measure exploration and omit selection loss
  ([[concepts/pass-at-k]] 10-27). If it is picked by validation score
  against test thresholds, it isn't a medal. The BMS case shows a 9%
  validation–test gap. Only the Table 1 rows look like submitted
  artifacts.
- **Single-read acceptance.** A hypothesis is accepted on one run's
  Δh ≥ δ_target, judged on the same validation service that later
  selects the final. There is no second holdout inside the loop. The
  paper lists "repeated measurements for high-stakes hypotheses" as
  future work. The belief constants are unreported, so the update rule
  can't be reproduced.
- **Search-time contamination is not addressed.** MLE-bench tasks are
  past Kaggle competitions with public winning write-ups. Pro²'s
  grounding step sends bottleneck-keyed queries to the open web. The
  paper reports no domain blocklist, no query log and no
  plagiarism/rule check. Its technique-recall metric scores overlap
  with those same write-ups, so it cannot separate "diagnosed the
  bottleneck" from "found the answer". Pure retrieval was the second
  largest matched step (12.1 → 28.8 initial). This is not evidence of
  leakage, only that the paper cannot exclude it
  ([[literature/papers/wang2026search]]).
- **The PostTrainBench "comparable to Opus 4.7" claim** picks the
  comparator it beats. Six agent rows score higher. The budget is 24 h on
  an A100 against 10 h on an H100, and "±" is not defined as SD or SE.
  The PI model alone in OpenCode scores 21.99. The 8.7-point harness lift
  is confounded with the 2.4× budget.
- **The model split is not ablated.** "Strong PI, Flash implementer" is
  presented as a design principle (§5). No arm uses all-Flash or all-Pro.
  Table 1 lists the backbone as "Gemini 3.5 Flash" with the PI in a
  footnote.
- **Cases are illustrative.** One trajectory each, chosen by the
  authors. The four Fig. 4 tasks have no stated selection rule.
- **Consistent with neighbours.** Its Lite drop from removing the state
  (89.4 → 56.1 final) is the same size as AiScientist's File-as-Bus
  ablation (−31.82 Lite medal points, [[literature/papers/chen2026toward]]).
  Its "diagnose first" finding fits TraceML's observation that agents
  rarely pivot ([[literature/papers/yan2026traceml]]), though nothing
  here measures pivots.

## Trust signals

- **Credibility:** 3.
  - For:
    - Google with Johns Hopkins (Alan Yuille).
    - The full 75-task benchmark plus PostTrainBench.
    - A matched 3-run ablation with SEs that separates building,
      grounding and updating.
    - Matched Direct and Search-Augmented baselines.
    - The grading service sits outside the sandbox.
    - Arithmetic in the tables is internally consistent (medal counts
      reconcile).
  - Against:
    - Not peer reviewed. No code, prompts or logs.
    - The headline is one run against multi-run means with unmatched
      budgets and models.
    - The Codebase State arm is undefined.
    - No token cost.
    - Best-in-hindsight node metrics, with the score source unstated.
    - No contamination control for web search.
    - Belief constants unreported.
  - Not 4: no code, and the headline comparison is unmatched. Not 2: a
    major lab, and the Lite ablation is controlled and repeated.

## Follow-up

- **Relevance:** 4.
  - It is a method on the hub's core benchmark, with the first
    build/ground/update decomposition of a research state there.
  - It is material new evidence for [[concepts/structured-world-model]],
    [[concepts/pass-at-k]] (how MLE-bench headlines are formed) and
    [[concepts/web-grounded-literature]] (search-time contamination on
    MLE-bench).
  - Not 5: the load-bearing arm is undefined and not token-matched, and
    the headline is k = 1.
- **Checks a future reader can run cheaply.**
  - When citing any MLE-bench rate, check whether it is a multiple of
    1/75 (one run) or not (a mean). Note budget and hardware beside it.
  - If code or trajectories appear, check whether the grounding queries
    hit kaggle.com competition pages or GitHub write-ups named for the
    competition.
- **Candidates** (cited, not yet in the graph):
  - AIBuildAI-2 (Zhang et al. 2026b, arXiv 2605.27873). It ties the
    headline, a "knowledge-enhanced" agent on Opus 4.7. Same
    contamination question.
  - Iris (Fu et al. 2026, arXiv 2608.02143), "Beyond solution-centric
    search: adaptive inquiry and knowledge revision". By title, the
    closest prior idea to the Problem State. It appears here only as a
    leaderboard row (64.9%, 12 h). Worth reading to see whether "problem
    state" is new.
  - MARS (Chen et al. 2026, arXiv 2602.02660), the main trajectory
    comparator. Same Google group. Cited in yu2026primescientist only.
  - Hypothesis Evolution Protocol (Takahara & Mizoguchi 2026, arXiv
    2607.09195). An auditable hypothesis registry with belief
    probabilities.
  - SAGE (Ma et al. 2026, arXiv 2606.31478). Multi-hypothesis failure
    attribution.
  - ScienceFlow (arXiv 2608.14354), cited in li2026humanize.
