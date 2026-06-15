---
kind: paper
title: "Toward Generalist Autonomous Research via Hypothesis-Tree Refinement"
authors: ["Jiajie Jin", "Yuyang Hu", "Kai Qiu", "Qi Dai", "Chong Luo", "Guanting Dong", "Xiaoxi Li", "Tong Zhao", "Xiaolong Ma", "Gongrui Zhang", "Zhirong Wu", "Bei Liu", "Zhengyuan Yang", "Linjie Li", "Lijuan Wang", "Hongjin Qian", "Yutao Zhu", "Zhicheng Dou"]
institutions: ["Renmin University of China (Gaoling School of AI)", "Microsoft Research"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.11926
code_url: https://github.com/RUC-NLPIR/Arbor
citations: null
source: "raw/papers/jin2026toward.pdf"
added: "2026-06-15"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/hce-evaluation]]"
tags: [autonomous-research, hypothesis-tree, mle-bench, coordinator-executor, worktree, held-out, long-horizon, search-state]
---

# Toward Generalist Autonomous Research via Hypothesis-Tree Refinement

## TL;DR

Arbor is a general framework for *Autonomous Optimization* (AO) — improving an
initial research artifact through iterative experimentation without step-level
human supervision. It separates a **long-lived coordinator** (owns global
research state, decides the search frontier) from **short-lived executors**
that each test one hypothesis in an **isolated git worktree** and return
structured evidence. The two-level loop is made cumulative by **Hypothesis
Tree Refinement (HTR)**: a persistent tree whose nodes bind a hypothesis, the
artifact version realizing it, the experimental evidence, and a distilled
insight; as results return, Arbor writes evidence back, abstracts findings
upward, and expands/prunes/merges the tree — promoting a candidate to "current
best" only when it improves a **held-out** evaluation. Across six real AO tasks
(model training, harness engineering, data synthesis) Arbor wins all six with
>2.5× the average relative held-out gain of Codex and Claude Code, and reaches
86.36% Any-Medal on MLE-Bench Lite with GPT-5.5.

## Claims

- Long execution alone does not produce research *progress*: an agent that
  treats each trial as an independent local attempt loses the structure of the
  research process. The missing mechanism is a persistent research state that
  records what was tried, what evidence resulted, and how each result reshapes
  the hypothesis space.
- The hypothesis tree is simultaneously the search frontier, the memory of past
  attempts, and the audit trail for verified improvement — one structure
  serving all three roles.
- Promotion to "current best" is gated on a held-out evaluation, not the
  development signal the executors optimize against — an explicit guard against
  overfitting the search signal.
- Coordinator/executor separation with worktree isolation lets many hypotheses
  be tested concurrently without cross-contaminating the shared state.
- Gains come from the HTR mechanism itself (shown via ablations, backbone
  studies, transfer experiments, cost analyses), not merely from a stronger
  backbone model.

## Methods

- **Autonomous Optimization (AO)** formalization: initial artifact + NL
  objective + task-native metric + a dev/test protocol that separates
  exploratory feedback from final scoring.
- **Coordinator** (long-lived): owns global strategy, decides which frontier
  nodes to expand; **executors** (short-lived): implement and test one
  hypothesis each in an isolated worktree, return structured evidence.
- **Hypothesis Tree Refinement (HTR)**: nodes = (hypothesis, artifact,
  evidence, insight); operations = expand / prune / merge; evidence written
  back to executed nodes and abstracted upward into reusable lessons.
- Six AO tasks across model training (optimizer/architecture design), harness
  engineering (TerminalBench, BrowseComp), and data synthesis (search-agent,
  math-reasoning); MLE-Bench Lite as an external check.

## Results

- Best held-out result on all six AO tasks; >2.5× the average relative held-out
  gain of Codex and Claude Code under the same interface and budget.
- MLE-Bench Lite: 86.36% Any-Medal with GPT-5.5 — strongest in their comparison.
- Per-task held-out gains range from ~+2.7% (optimizer design) to +22.34%
  (BrowseComp) over the initial artifact.

## Critique / open questions

- "Living technical report for an ongoing project" — numbers may shift; treat
  the held-out gains as a snapshot, not a frozen result.
- The held-out promotion gate is exactly the HCE discipline applied *inside*
  the search loop; how strictly dev/test separation is enforced per task (and
  whether executors can leak test info) is the load-bearing detail to verify.
- HTR generalizes MCTS-style expansion to a hypothesis-grained tree (cf.
  MLEvolve's Progressive MCGS) — the open question is whether the grain
  (hypothesis vs code-diff vs program) is what drives the gain or whether the
  coordinator's abstraction step is.

## Trust signals

- **Credibility:** 4 — Renmin University of China (Gaoling School of AI) +
  Microsoft Research, open-source code (github.com/RUC-NLPIR/Arbor); arXiv
  preprint (Jun 10 2026), self-described living report, not peer-reviewed.

## Follow-up

- **Relevance:** 5 — a multi-concept anchor on the dominant front. The
  coordinator/executor split is `hierarchical-delegation`; the persistent
  hypothesis tree (frontier + memory + audit trail) is a textbook
  `structured-world-model`; expand/prune/merge over the tree is
  `evolutionary-expansion` with the hypothesis as the `evolutionary-search-grain`;
  and the held-out promotion gate is `hce-evaluation` applied inside the loop.
  Worktree-isolated executors also attest `file-as-bus`.
- Strong candidate for a downstream experiment: ablate the search grain
  (hypothesis-node vs code-diff) to test whether HTR's gain is grain-driven or
  abstraction-driven.
