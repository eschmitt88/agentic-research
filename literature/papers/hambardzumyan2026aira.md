---
kind: paper
title: "AIRA_2: Overcoming Bottlenecks in AI Research Agents"
authors:
  - Karen Hambardzumyan
  - Nicolas Baldwin
  - Edan Toledo
  - Rishi Hazra
  - Michael Kuchnik
  - Bassel Al Omari
  - Thomas Simon Foster
  - Anton Protopopov
  - Jean-Christophe Gagnon-Audet
  - Ishita Mediratta
  - Kelvin Niu
  - Michael Shvartsman
  - Alisia Lupidi
  - Alexis Audran-Reiss
  - Parth Pathak
  - Tatiana Shavrina
  - Despoina Magka
  - Hela Momand
  - Derek Dunfield
  - Nicola Cancedda
  - Pontus Stenetorp
  - Carole-Jean Wu
  - Jakob Nicolaus Foerster
  - Yoram Bachrach
  - Martin Josifoski
institutions: ["Meta AI", "University College London", "University of Oxford"]
year: 2026
venue: "arXiv:2603.26499 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2603.26499"
code_url:
citations:
source: "raw/papers/hambardzumyan2026aira.pdf"
added: "2026-04-24"
relevance: 5
credibility: 4
status: skimmed
related_experiments:
  - hce-discipline-ablation
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/async-worker-pool]]"
tags:
  - aira
  - bottlenecks
  - hidden-consistent-evaluation
  - async
---

# AIRA_2: Overcoming Bottlenecks in AI Research Agents

## TL;DR

Meta FAIR names three structural bottlenecks in AI research agents —
synchronous execution limiting throughput, validation-based selection
causing overfitting to the search signal, and weak fixed single-turn
LLM operators — and addresses each with specific machinery:
asynchronous multi-GPU workers, a Hidden Consistent Evaluation (HCE)
protocol, and ReAct agents with dynamic scoping. 81.5% mean percentile
rank on MLE-bench-30 at 24h, 83.1% at 72h, versus 72.7% best baseline.

## Claims

- Autonomous research loops overfit to their own validation signal
  across many iterations — this is a dominant failure mode, not a
  corner case.
- The fix is a Hidden Consistent Evaluation protocol: a held-out
  test split that the search loop never sees, revealed only at chain
  end. Validation metrics drive search; test metrics score it.
- Synchronous execution (one experiment at a time) caps throughput
  far below hardware capacity. Async workers with a shared job queue
  eliminate this.
- Fixed single-turn LLM operators are brittle; ReAct agents that can
  re-scope dynamically within a step are strictly better.
- All three matter. Ablations confirm each component's necessity.

## Methods

- HCE protocol: strict separation of validation split (visible to the
  search loop, written to `metrics.json`) and test split (invisible,
  revealed only at chain-end final-scoring pass, written to
  `final_metrics.json`).
- Async multi-GPU worker pool with shared job queue; workers claim
  experiments, report metrics, free themselves for the next job.
- ReAct agents as LLM operators with dynamic scope adjustment inside
  a single step.
- Evaluation on MLE-bench-30 (the 30-competition subset) and
  AIRS-Bench.

## Results

- MLE-bench-30: 81.5% mean percentile rank at 24h, 83.1% at 72h,
  versus 72.7% strongest baseline.
- AIRS-Bench: exceeds human baseline on 6 of 20 tasks.
- Ablations: removing HCE, async workers, or ReAct operators each
  individually degrades performance significantly.
- Scaling laws across LLM variants are predictable (noise is not
  dominating signal).

## Critique / open questions

- HCE separation is a discipline rule, not a hard constraint —
  enforcement depends on trust in the framework or static checks.
  Our project enforces via `~/.claude/rules/evaluation.md` and
  `/lint`.
- "Mean percentile rank" is a different metric than MLE-bench medal
  rate; direct comparison to FM Agent (43.56% medal) and AIBuildAI
  (63.1% medal) requires care.
- Async worker infrastructure is heavy — at-home reproduction may
  need to trade parallelism for simpler sync loops.

## Trust signals

- **Credibility:** 4 — FAIR at Meta, with University College London
  and University of Oxford; arXiv preprint (v2), not yet
  peer-reviewed; no released code located on the front matter. A
  major-lab work with careful ablations and a transferable scaling
  law — strong on most signals, short only on peer review and a
  public artifact.

## Follow-up

- This is the canonical citation for HCE. Load into
  `~/.claude/rules/evaluation.md` context (already done).
- Deep-read the ReAct-with-dynamic-scoping section; the paper frames
  this as the biggest win per ablation-dollar.
- Extract the exact HCE protocol wording for downstream projects to
  quote in their evaluation docs.
