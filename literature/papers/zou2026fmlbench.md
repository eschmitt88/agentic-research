---
kind: paper
title: "FML-bench: A Controlled Study of AI Research Agent Strategies from the Perspective of Search Dynamics"
authors: ["Qiran Zou", "Hou Hei Lam", "Wenhao Zhao", "Tingting Chen", "Yiming Tang", "Samson Yu", "Yingtao Zhu", "Srinivas Anumasa", "Zufeng Zhang", "Tianyi Zhang", "Chang Liu", "Zhengyao Jiang", "Anirudh Goyal", "Dianbo Liu"]
institutions: ["National University of Singapore", "Tsinghua University", "University of Minnesota", "Weco", "Meta"]
year: 2026
venue: arXiv (cs.LG)
peer_reviewed: false
url: https://arxiv.org/abs/2605.17373
code_url: https://github.com/qrzou/FML-bench
citations: null
source: "raw/papers/zou2026fmlbench.pdf"
added: "2026-07-14"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
tags: [benchmark, research-agents, search-dynamics, greedy-vs-tree, process-metrics, controlled-comparison, mle]
---

# FML-bench: A Controlled Study of AI Research Agent Strategies from the Perspective of Search Dynamics

## TL;DR

A benchmark of 18 real ML-research tasks that holds the LLM and the
execution infrastructure fixed so *search strategy* is the only variable,
plus 12 process-level metrics over the search trajectory. Findings: a
greedy hill-climber (Autoresearch) statistically matches the best
tree-search agent (AI Scientist v2) while principled MCTS (AIRA) ranks
last; which strategy wins depends on the task's improvement-opportunity
density; and a simple adaptive agent that runs greedy until stagnation
then forks broader exploration beats everything.

## Claims

- Prior agent comparisons confound strategy with infrastructure — each
  agent ships its own code editor, runner, and metric plumbing — and
  report only final scores, compressing the whole trajectory into one
  scalar. Both must be controlled to say anything about strategy.
- **Strategy complexity does not predict performance** (Finding 1):
  four-stage best-first tree search with journal memory (TAS v2, 0.193
  mean normalized test improvement) ≈ single-incumbent greedy
  hill-climbing (Autoresearch, 0.192), both well above AIDE (0.178),
  OpenEvolve (0.151), TAS v1 and UCT-MCTS AIRA (0.132 each).
- **Opportunity structure decides** (Finding 2): greedy excels where
  small code modifications frequently pay off (dense), stalls where most
  modifications fail (sparse); tree/evolutionary strategies hold
  multiple frontiers and break plateaus. Post-hoc partition: greedy
  ranks first on dense tasks, sixth on sparse ones.
- **What predicts success** (Finding 3): AUC-over-steps (ρ=+0.784),
  early first improvement (ρ=−0.291), exploration reach (+0.115), and
  directional concentration (effective dim, −0.140) are significant;
  solution diversity/uniqueness (ρ≈+0.01), token cost (ρ≈0), and
  wall-clock time are not. How compute is allocated matters; how much
  is spent does not.

## Methods

- 18 tasks over 10 domains (generalization, data efficiency,
  representation/continual/federated learning, causality, robustness,
  privacy, fairness, unlearning), each improving an established baseline
  in a public research codebase; a validation run fits in 40 min on one
  GPU so a 100-step run finishes in days.
- Shared infrastructure: one thin code editor (single LLM call per
  edit, no agent-specific scaffolding), a unified step definition (one
  step = one execution of the task's validation command), unified metric
  rendering, and **framework-enforced val/test separation** — agents see
  only validation metrics during search; the best-validated codebase is
  scored once on the held-out test at the end.
- Six agents spanning the topology space (greedy, parallel linear,
  best-first tree ×2, UCT-MCTS, MAP-Elites-style evolutionary), all on
  GPT-5.4, 3 rounds × 18 tasks × 100 steps (324 runs).
- 12 process metrics over GraphCodeBERT embeddings of each valid
  codebase state: exploration spread/reach/uniqueness/effective-dim,
  val-test gap, valid-step ratio, AUC-over-steps, first/best-improvement
  step, late-gain fraction, token cost, wall-clock.
- AdaptiveSearch: Phase 1 = greedy incumbent; when normalized
  improvement stalls for W consecutive steps, irreversibly fork top-N
  candidates into round-robin multi-branch exploration.

## Results

- AdaptiveSearch: 0.208 mean improvement, 58.6% pairwise win-rate —
  above TAS v2 (0.193/56.2%) and Autoresearch (0.192/56.2%) under the
  same 100-step budget, and top-two on *both* the dense and sparse task
  partitions.
- Two paths to the top: TAS v2 is slow-start but compensates with the
  highest exploration reach along few directions (lowest effective dim);
  Autoresearch has the fastest first improvement and highest AUC. TAS v1
  is the cautionary case — second-highest reach and spread but
  directionally scattered, finishing fifth.
- Cost is decoupled from performance: the most expensive agent (TAS v1,
  2.29M tokens) ranks fifth; the cheapest (OpenEvolve, 0.93M) ranks
  fourth.
- Exploration *uniqueness* (distinct solution families visited) shows
  essentially zero correlation with outcome — empirical pushback on
  novelty-search/quality-diversity intuitions in this setting.

## Critique / open questions

- To control the comparison, every agent's native code editor and
  auxiliary subsystems were removed — the authors acknowledge this may
  underestimate agents whose strength lives in their scaffolding
  (AIRA's last place should be read with this caveat).
- Single LLM (GPT-5.4), fixed 100-step budget; strategy rankings could
  shift with model strength or longer horizons (TAS v2's late-gain
  pattern suggests budget sensitivity).
- Opportunity density is measured post-hoc from the runs themselves;
  an a-priori task-level predictor is future work, and AdaptiveSearch's
  W and N were presumably tuned on the same 18 tasks.
- 3 rounds per (agent, task) is thin for the per-task variances shown
  (some ±.2 on means of .2).

## Trust signals

- **Credibility:** 4 — multi-institution team (NUS, Tsinghua, Meta,
  Weco — the AIDE lineage), code released, careful controlled design
  with process-level metrics and stated limitations; preprint, not yet
  peer-reviewed or cited.

## Follow-up

- **Relevance:** 4 — the first controlled strategy-vs-strategy
  comparison for research agents, directly answering
  [[concepts/evolutionary-search-grain]]'s open question about isolating
  search-design choices (at the topology level rather than the mutation
  grain), and validating its "adaptive meta-controller" speculation:
  stall-triggered switching from exploit to explore beats any fixed
  topology.
- Direct import for `/iterate`: our `budget.yaml`
  `max_consecutive_no_improvement: 3` uses the stagnation signal to
  *halt*; FML-bench shows the same signal is better spent triggering a
  strategy *switch* (fork the top-N proposals and broaden) before
  halting. See [[concepts/budget-as-ceiling]].
- The process-metric result set is a ready-made diagnostics vocabulary
  for iteration logs: AUC-over-steps and first-improvement step are the
  signals worth tracking; token spend is not a health metric.
- The framework-enforced val/test separation is another
  [[concepts/hce-evaluation]] attestation: separation as benchmark
  *infrastructure*, not agent virtue.
