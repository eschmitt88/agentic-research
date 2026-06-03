---
kind: paper
title: "PaperBench: Evaluating AI's Ability to Replicate AI Research"
authors:
  - Giulio Starace
  - Oliver Jaffe
  - Dane Sherburn
  - James Aung
  - Jun Shern Chan
  - Leon Maksin
  - Rachel Dias
  - Evan Mays
  - Benjamin Kinsella
  - Wyatt Thompson
  - Johannes Heidecke
  - Amelia Glaese
  - Tejal Patwardhan
institutions: ["OpenAI"]
year: 2025
venue: "arXiv:2504.01848 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2504.01848"
code_url: "https://github.com/openai/preparedness"
citations:
source: "raw/papers/starace2025paperbench.pdf"
added: "2026-06-03"
relevance: 5
credibility: 5
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
tags:
  - benchmark
  - research-replication
  - mle-bench-adjacent
  - llm-judge
  - rubric-evaluation
  - human-baseline
  - openai
---

# PaperBench: Evaluating AI's Ability to Replicate AI Research

## TL;DR

(Skim of abstract page.) PaperBench is an OpenAI benchmark that
measures whether AI agents can **replicate** 20 ICML 2024 Spotlight
and Oral papers from scratch — understanding contributions, building a
codebase, and running experiments. Each task is hierarchically
decomposed into gradable sub-tasks (8,316 in total) via author-co-
developed rubrics, and an LLM judge auto-grades attempts. Best tested
agent (Claude 3.5 Sonnet (New) with open-source scaffolding) scores
21.0%; top ML PhDs still beat the models.

## Claims

- Replicating SOTA AI research end-to-end (read paper → build codebase
  → run experiments) is a meaningful, hard capability to benchmark for
  AI-research agents.
- Hierarchical rubrics co-developed with the original paper authors
  give objective, fine-grained grading (8,316 individually gradable
  leaf tasks across 20 papers).
- An LLM-based judge can grade replication attempts at scale; its
  reliability is itself measured via a separate judge benchmark.
- Models do **not yet** outperform a human (ML PhD) baseline on a
  subset.

## Methods

- **20 ICML 2024 Spotlight/Oral papers**; agents replicate each from
  scratch.
- **Hierarchical rubric decomposition**: each replication task split
  into smaller sub-tasks with explicit grading criteria; rubrics
  co-developed with paper authors for accuracy/realism. Total 8,316
  gradable tasks.
- **LLM judge** for scalable auto-grading, plus a **separate
  judge-evaluation benchmark** to assess the judge itself.
- **Human baseline**: top ML PhDs attempt a subset for comparison.
- Code open-sourced (`openai/preparedness`).

## Results

- Best tested agent: **Claude 3.5 Sonnet (New)** with open-source
  scaffolding → **21.0%** average replication score.
- Models do not yet outperform the human (ML PhD) baseline on the
  evaluated subset.

## Critique / open questions

- Replication ≠ novel discovery; PaperBench measures faithful
  re-implementation, which is necessary-but-not-sufficient for the
  autonomous-research-agent mission. Complements (does not replace)
  discovery benchmarks.
- LLM-judge grading introduces a judge-reliability dependency; the
  authors mitigate with a separate judge benchmark, but judge bias on
  borderline rubric items is a residual risk worth noting when citing
  scores.
- 20 papers from a single venue/year (ICML 2024) — coverage breadth
  and leakage (papers in training data) are open considerations.

## Trust signals

- **Credibility:** 5 — OpenAI, frontier lab; author-co-developed
  rubrics and a separately-validated LLM judge; **open-sourced code**
  (`openai/preparedness`); explicit human baseline. The methodology is
  unusually rigorous for a capability benchmark. arXiv preprint
  (not formally peer-reviewed), but the lab provenance, released
  artifacts, and validation design put trust at the top of the rubric.

## Follow-up

- Canonical companion to MLE-bench (`chan2024mle`) and AIRA_2
  (`hambardzumyan2026aira`) in the research-agent benchmark cluster,
  and named explicitly in this project's CLAUDE.md mission statement
  ("MLE-bench / PaperBench / AIRA"). Long-overdue addition to the graph.
- The **hierarchical author-co-developed rubric** is a concrete
  instantiation of `hce-evaluation`'s principle that the scoring
  oracle must be defined independently of the search loop — here the
  rubric (held by authors/judge) is the held-out scoring authority the
  agent optimizes toward but does not author.
- The LLM-judge + separate judge benchmark is a transferable pattern
  for any project that auto-scores agent output: validate the judge
  before trusting it. Pairs with `programmable-evaluator-oracle`.
- The 21.0% headline and human-baseline gap are a useful anchor for
  `pass-at-k` framing — single-number capability claims on this kind
  of benchmark should report seed/run variance.
</content>
