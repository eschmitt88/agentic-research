---
kind: paper
title: "MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering"
authors:
  - Jun Shern Chan
  - Neil Chowdhury
  - Oliver Jaffe
  - James Aung
  - Dane Sherburn
  - Evan Mays
  - Giulio Starace
  - Kevin Liu
  - Leon Maksin
  - Tejal Patwardhan
  - Lilian Weng
  - Aleksander Mądry
year: 2024
venue: "ICLR 2025 (arXiv:2410.07095)"
url: "https://arxiv.org/abs/2410.07095"
source: "raw/papers/chan2024mle.pdf"
added: "2026-04-24"
relevance: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
tags:
  - benchmark
  - kaggle
  - mle-bench
  - evaluation
---

# MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering

## TL;DR

MLE-bench curates 75 Kaggle competitions as a benchmark for autonomous
ML engineering agents. Human baselines come from public leaderboards;
agent submissions are scored against the same medal thresholds (bronze,
silver, gold). OpenAI's o1-preview + AIDE scaffold reaches bronze on
~17% of competitions — the headline baseline for every system that
followed.

## Claims

- A credible benchmark for ML engineering agents requires (a) real
  tasks, not toy problems, (b) calibrated human baselines, and
  (c) resistance to pre-training contamination.
- Kaggle competitions, scored by medal thresholds, satisfy all three
  — medals are gold-standard performance markers set by actual
  practitioner effort, not synthetic targets.
- Bronze-rate is the least-noisy comparative metric across competitions
  of very different difficulties.

## Methods

- 75 curated competitions spanning data prep, model training, and
  full-pipeline engineering.
- Open-source evaluation harness; agents submit via a scaffold of the
  author's choice.
- Contamination analysis: check whether pre-training corpora contain
  the competition's leaderboard solutions.

## Results

- o1-preview + AIDE scaffold achieves bronze on ~17% of 75
  competitions — the baseline everyone cites.
- Resource scaling (compute, context length) improves performance
  but with diminishing returns above a threshold.
- Contamination effects are measurable but do not dominate results.

## Critique / open questions

- 17% bronze is now a years-old baseline — interesting as a floor,
  not as a target. FM Agent (43.56%) and AIBuildAI (63.1%) have since
  raised the bar; MLE-STAR reports 64% on MLE-bench *Lite* which is
  a different, smaller set.
- Medal thresholds are coarse. Two agents can both earn bronze with
  very different absolute scores.
- No discipline on held-out splits inside the benchmark — later work
  (AIRA_2) makes HCE separation a first-class concern.

## Follow-up

- Audit the 75-competition list for which are "Lite" (used by
  MLE-STAR) versus full.
- Record baseline numbers as the comparison target for any future
  MLE-bench smoke test in this or downstream projects.
- Cross-check how the benchmark handles validation vs. test split
  inside a single competition — this is the cleanest venue for the
  HCE rule to bite.
