---
kind: paper
title: "AutoMedBench: Towards Medical AutoResearch with Agentic AI Models"
authors: ["Junqi Liu", "Selena Song", "Yuhan Wang", "Jiawei Mao", "Hardy Chen", "Xiaoke Huang", "Tianhao Qi", "Pengfei Guo", "Yucheng Tang", "Yufan He", "Can Zhao", "Andriy Myronenko", "Dong Yang", "Daguang Xu", "Yuyin Zhou"]
institutions: ["University of California, Santa Cruz", "NVIDIA"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.01961
code_url: https://github.com/AutoMedBench/AutoMedBench
citations: null
source: "raw/papers/liu2026automedbench.pdf"
added: "2026-06-08"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [benchmark, medical-ai, autoresearch, workflow, evaluation, verification, stage-scoring, error-codes]
---

# AutoMedBench: Towards Medical AutoResearch with Agentic AI Models

## TL;DR

A workflow-aware benchmark for autonomous medical-AI research that decomposes
each long-horizon task into a five-stage workflow (Plan, Setup, Validate,
Inference, Submit) and scores both final task performance *and* per-stage
(S1–S5) execution. Spanning segmentation, image enhancement, VQA, report
generation, and lesion detection (tasks averaging 33 agent turns, two
difficulty tiers), it finds that **Validate is the weakest stage** and that
verification + submission failures dominate errors — the empirical case that
autonomous research agents are far better at *implementing* pipelines than at
*verifying their own reliability*.

## Claims

- Final-output-only agent benchmarks are insufficient for long-horizon
  workflows: failures emerge at different stages and compound before being
  collapsed into a single end score, hiding where the agent actually broke.
- Stage-level scoring across thousands of runs shows **Validate is the weakest
  workflow stage on average; Setup is the strongest** — agents build runnable
  pipelines but cannot reliably verify them.
- Post-run error analysis: verification failures (37.7%) and submission
  failures (38.1%) dominate fired error codes (~76% combined), while
  task-understanding errors are rare (0.9%).
- Error codes are diagnostic, not cosmetic: a run with even one fired error
  code scores 48% lower overall on average than a clean run.
- Strong medical-research agents must combine high-quality domain knowledge
  with robust engineering — specifically intermediate validation and error
  recovery throughout the workflow.

## Methods

- Five-stage unified workflow (S1 Plan → S2 Setup → S3 Validate → S4 Inference
  → S5 Submit), each stage independently scored.
- Two difficulty tiers (Lite / Standard) sharing data and metrics but differing
  in the amount of task-brief scaffolding — an explicit scaffolding ablation.
- Joint scoring of final task performance + S1–S5 stage scores, linked to a
  taxonomy of diagnostic error codes (failed model loading, shape bugs,
  skipped validation, empty outputs, malformed submissions).
- Five research tracks across medical imaging + multimodal inference; ~33
  agent turns per task; thousands of recorded runs analyzed.

## Results

- Validate weakest, Setup strongest stage on average.
- Verification 37.7% + submission 38.1% of fired error codes; understanding
  0.9%; one-error runs −48% overall score.
- Stage-level + error-code linkage surfaces "hidden breakdowns" invisible to
  final-output metrics.

## Critique / open questions

- Domain is medical AI, but the architectural lesson (stage-decomposed scoring
  + diagnostic error codes exposing a *verification* bottleneck) is the
  transferable part for general research-agent evaluation — cf. MLE-bench /
  PaperBench, which score mostly final artifacts.
- "Validate is weakest" is the inverse of the HCE worry: HCE guards against
  over-trusting the search signal, while AutoMedBench shows agents *under*-
  validate. Both point to validation as the load-bearing discipline.
- The error-code taxonomy is a candidate typed-claim partition for *failures*
  rather than claims; whether it generalizes beyond medical pipelines is open.

## Trust signals

- **Credibility:** 4 — UC Santa Cruz + NVIDIA, code released
  (github.com/AutoMedBench/AutoMedBench) with a public leaderboard
  (automedbench.github.io), but an arXiv preprint, not yet peer-reviewed.

## Follow-up

- **Relevance:** 4 — in-scope as general scientific-research automation whose
  architecture transfers (cf. Kosmos in biology). The verification-bottleneck
  finding is strong new evidence for `hce-evaluation` (validation is the
  fragile stage), the diagnostic error-code taxonomy attests
  `typed-claim-partition` (a typed partition of failure modes), and the
  stage-level scoring rig is a `programmable-evaluator-oracle` instance
  (per-stage fitness + diagnostics).
- The Lite/Standard scaffolding tiers are a clean design for a downstream
  experiment on how much task-brief scaffolding a research agent needs.
