---
kind: paper
title: "MLEvolve: A Self-Evolving Framework for Automated Machine Learning Algorithm Discovery"
authors: ["Shangheng Du", "Xiangchao Yan", "Jinxin Shi", "Zongsheng Cao", "Shiyang Feng", "Zichen Liang", "Boyuan Sun", "Tianshuo Peng", "Yifan Zhou", "Xin Li", "Jie Zhou", "Liang He", "Bo Zhang", "Lei Bai"]
institutions: ["Shanghai Artificial Intelligence Laboratory", "East China Normal University"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.06473
code_url: https://github.com/InternScience/MLEvolve
citations: null
source: "raw/papers/du2026mlevolve.pdf"
added: "2026-06-08"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/hierarchical-delegation]]"
tags: [mle-bench, evolutionary-search, mcts, graph-search, agent-memory, hierarchical-control, algorithm-discovery, self-evolving]
---

# MLEvolve: A Self-Evolving Framework for Automated Machine Learning Algorithm Discovery

## TL;DR

An LLM-based self-evolving multi-agent framework for end-to-end MLE that
attacks three failure modes of existing MLE agents — inter-branch
information isolation, memoryless search, and lack of hierarchical control —
with three mechanisms: Progressive MCGS (graph-structured tree search with
cross-branch edges and an entropy-scheduled explore→exploit shift),
Retrospective Memory (cold-start knowledge base + dynamic global experience
store), and Hierarchical Planning with Adaptive Code Generation (planner
decides *what* to modify, coder decides *how*, picking full-rewrite /
stepwise / diff editing by search state). Achieves SOTA on MLE-Bench under a
12h budget (half standard runtime) and beats AlphaEvolve on math algorithm
optimization.

## Claims

- Existing MLE agents fail at long-horizon self-evolution for three reasons:
  (1) linear/tree search confines successful strategies to a single branch
  (information isolation); (2) search is memoryless — scalar rewards only, no
  reuse of insights from earlier similar attempts; (3) planning and code
  generation are fused into one-shot rewrites, conflating *what to modify*
  with *how to implement*.
- Progressive MCGS replaces the search tree with a graph: reference edges let
  information flow across branches, and an entropy-inspired progressive
  schedule moves the search from broad exploration to focused exploitation.
- Retrospective Memory pairs a curated domain-prior knowledge base
  (cold-start) with a dynamic global memory that automatically accumulates
  and retrieves task-specific experience throughout the search.
- Adaptive Code Generation selects among full rewrite, stepwise (module-by-
  module), and diff (patch-edit) coding modes according to the current search
  state — the mutation grain is chosen, not fixed.

## Methods

- Four-stage evolving loop: Plan → Generate → Execute → Evolve.
- Progressive Monte Carlo Graph Search (MCGS): graph-based cross-branch
  sharing + adaptive exploration that shifts explore→exploit over time.
- Retrospective Memory: knowledge base (domain priors) + global experience
  (search records), retrieved per planning decision.
- Hierarchical Planning: a planner produces a detailed "what" spec; a coder
  realizes it under one of three coding modes selected by search state.
- Best solution surfaced via submission.csv + top-k candidate tracking.

## Results

- SOTA on MLE-Bench (75 Kaggle competitions) across average medal rate and
  valid submission rate, achieved under a 12-hour budget — half the standard
  runtime used by baselines.
- Outperforms specialized algorithm-discovery methods including AlphaEvolve
  on mathematical algorithm-optimization tasks, evidencing cross-domain
  generalization.

## Critique / open questions

- Author affiliations are a strong group (Shanghai AI Lab) but the paper is
  an arXiv preprint, not yet peer-reviewed; MLE-bench numbers are
  self-reported.
- The "12h = half budget" headline conflates efficiency with quality —
  worth checking whether full-budget baselines close the medal-rate gap.
- Adaptive coding-mode selection is the most transferable idea but the
  selection policy (when to diff vs full-rewrite) is under-specified in the
  abstract/intro; the mechanism detail is what `evolutionary-search-grain`
  needs.

## Trust signals

- **Credibility:** 4 — Shanghai AI Laboratory (strong group) + ECNU, code
  released at github.com/InternScience/MLEvolve, but arXiv preprint (not
  peer-reviewed) with self-reported MLE-bench results and no independent
  reproduction yet.

## Follow-up

- **Relevance:** 5 — provides fresh, mechanism-level evidence for two
  load-bearing concepts at once: `evolutionary-search-grain` (adaptive
  full-rewrite/stepwise/diff coding modes = an explicit, state-dependent
  choice of mutation unit) and `evolutionary-expansion` (Progressive MCGS as
  a graph-structured, schedule-controlled successor to FunSearch/AlphaEvolve
  tree search). Also attests `agent-native-memory` (in-loop retrospective
  memory) and `hierarchical-delegation` (planner/coder what-vs-how split).
- Resolve the adaptive coding-mode selection policy from the full PDF (§
  methods) — it is the sharpest new datum for `evolutionary-search-grain`.
- Compare Progressive MCGS against qu2026coral and the AIRA MCTS/evolutionary
  operator results already in the graph.
