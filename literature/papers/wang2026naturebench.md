---
kind: paper
title: "NatureBench: Can Coding Agents Match the Published SOTA of Nature-Family Papers?"
authors: ["Yuru Wang", "Lejun Cheng", "Yuxin Zuo", "Kaiyan Zhang", "Ning Ding", "Bowen Zhou"]
institutions: ["Horizon Research, Frontis.AI", "Tsinghua University", "Peking University", "Harvard University"]
year: 2026
venue: "arXiv preprint (cs.CL)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.24530"
code_url: "https://github.com/FrontisAI/NatureBench"
citations: null
source: "raw/papers/wang2026naturebench.pdf"
added: "2026-07-20"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
tags: [benchmark, evaluation, discovery, scientific-agents, sota-anchoring, information-firewall, containerization]
---

# NatureBench: Can Coding Agents Match the Published SOTA of Nature-Family Papers?

## TL;DR

Splits *discovery* from *reproduction* — a distinction MLE-bench-style
optimization and PaperBench-style replication both blur — with 90 tasks
distilled from Nature-family papers by NatureGym, an automated
paper-to-containerized-task pipeline whose information firewall strips
the source method so agents must invent, not re-implement; the best of
twelve frontier agents (Claude Opus 4.7 + Claude Code) surpasses the
published SOTA (g > 0.1) on only 17.8% of tasks and matches it (g ≥ 0)
on 47.8%, succeeding mostly by translating scientific tasks into
familiar supervised prediction rather than by scientific invention.

## Claims

- Existing paper-grounded benchmarks (PaperBench, CORE-Bench,
  ReplicationBench) target reading/reproduction; optimization benchmarks
  (MLE-bench, PostTrainBench) target Kaggle/ML-engineering proxies with
  no natural-science content and fragile, fragmented environments.
  NatureBench claims the unfilled intersection: paper-sourced tasks +
  genuine scientific domains + discovery-oriented scoring against the
  paper's own published SOTA (Table 2's three-way ✓).
- The **information firewall** is what turns reproduction into
  discovery: every file specific to the source algorithm A (its
  preprocessing, intermediate and final outputs) is excluded; only files
  needed "to define the task no matter which method is used" are kept,
  and no package file may reveal the source paper's identity or method.
  Web search is disabled at eval time so the firewall can't be bypassed
  by retrieval.
- Environment fragmentation — not task difficulty — is what limited the
  credibility of prior agent-on-research benchmarks; NatureGym's
  per-task Docker overlays on a shared base image plus a host-side
  evaluation service make every task independently re-runnable.
- Agents that do match SOTA get there by **methodological translation**:
  supervised proxy prediction accounts for 45.5% of validated successes
  and engineering-driven categories 82.7% together; domain-reasoned
  alternatives (8.3%) and method-aligned solutions (9.0%) are rare.
  Same-broad-family-as-paper runs match SOTA at 37.7% vs 29.6% for
  different-family runs — method proximity to the task's scientific
  structure helps even though nothing enforces it.
- Failures concentrate in method choice (45.1%) and insufficient
  compute/time (24.4%), not task misunderstanding (3.1%) — the
  bottleneck is scientific method selection and execution depth, not
  reading comprehension or code generation.
- The six domains form a difficulty gradient shared across all ten
  analyzed agents (Spearman ρ 0.71–1.00 vs consensus): Relational
  Reasoning 60.0% consensus Match-SOTA down to Biomedical Modeling
  17.9%; cross-discipline tasks (15 of 90) widen the gap further
  (pooled median g̃ −0.13 → −0.21, 9/10 agents in that direction).

## Methods

- **NatureGym pipeline**, five phases with review gates: crawl ~5,500
  papers from ten Nature-family journals (2022–2025) → article filter
  ~2,500 → three-level filtering (task extractability, deterministic
  automatable evaluation, complete public data ≤50 GB) ~200 → dataset
  acquisition + verification ~180 → task-package construction ~160 →
  evaluation-time calibration → **90 tasks / 333 instances**. Each stage
  is run by an LLM agent with a verify–repair loop; humans confirm only
  critical corrections. Every task is a tuple T = (Algorithm, Data,
  Metric, SOTA, Baseline) refined across stages.
- **Task package**: agent-visible `problem/` (README, data description,
  inputs with ground truth excluded) vs hidden `evaluation/`
  (evaluator.py, ground_truth/), per-task Dockerfile, metadata.json with
  per-instance SOTA scores. 36 automated build checks + logic/smoke
  tests, including verifying evaluator scores against the authors'
  released outputs.
- **Scoring**: SOTA-normalized relative gap
  g_i = dir_i · (m_i − m_i^sota)/|m_i^sota| on one primary metric per
  instance (81 distinct primary metrics across the corpus), averaged
  across instances; no valid submission → g = −1.0. Surpass-SOTA =
  g > 0.1, Match-SOTA = g ≥ 0.
- **Run protocol**: isolated container, 4-hour wall clock (paused during
  scoring), GPU matched to task metadata (RTX 3090/4090 or A800); the
  agent iteratively queries a host-side service (/evaluate, /best_score,
  /time_remaining) it cannot read the internals of; a post-hoc Claude
  Sonnet 4.6 validity judge screens for fabrication, rule substitution,
  answer recovery, feedback gaming, and training bypass, voiding flagged
  runs.
- **Calibration**: base-mode diagnosis with Opus 4.6, then a
  reproduce-mode audit (agent additionally gets the source paper and
  must faithfully reproduce) with Opus 4.6 and DeepSeek-V4-Pro; 45 tasks
  dropped for systematic defects, 17 repaired. On the final 90, Opus 4.6
  reproduces 30 and DeepSeek 21; on the 16 tasks both reproduce, g
  clusters tightly at zero (median −0.0026), calibrating the SOTA
  anchors.
- Twelve model/harness configurations (Claude Code ×9, Codex CLI ×2,
  Gemini CLI ×1), each run once over all 90 tasks; 900-run behavioral
  annotation over the ten v1-analyzed configs.

## Results

- Surpass-SOTA: Claude Opus 4.7 17.8%, GLM-5.2 and Gemini 3.5 Flash
  15.6%, GPT-5.5 14.4%, down to MiniMax-M2.7 at 1.1%. Match-SOTA: Opus
  4.7 47.8%, GPT-5.5 44.4%, GLM-5.2 41.1%.
- Median relative gap g̃_all runs −0.007 (Opus 4.7) to −0.40
  (MiniMax-M2.7): most tasks land modestly below SOTA rather than
  failing badly; heavy negative means (−4.5 to −12.5) are
  normalization artifacts of near-ceiling SOTA denominators, audited and
  found not to be task errors.
- Validity audit is discriminative: both Claude Opus agents show 100%
  completion and score rates with zero invalid submissions; GPT-5.5
  attempts shortcuts most often (13 invalid submissions filtered by the
  judge), and its remaining scores stay genuine — the only non-negative
  median over judge-accepted tasks (+0.001).
- High-frequency submission to the scorer is "overwhelmingly legitimate
  iteration"; the rare genuine exploit is caught by the judge.

## Critique / open questions

- The evaluation service scores the agent's submissions against the
  held-out ground truth *during* the run — the search signal is the test
  signal. That is a deliberate design (peak-capability measurement
  against a fixed anchor) but the opposite of HCE discipline; the
  feedback-gaming residual is bounded only by a post-hoc LLM judge, and
  the paper concedes the risk rather than eliminating it.
- Single run per agent configuration (k=1): the 17.8% headline has no
  seed-variance treatment, and MLE-bench/AIRA showed run-to-run variance
  at magnitudes rivaling architecture differences. The reproduce-mode
  cross-model agreement (16 doubly-reproduced tasks) is a calibration
  check, not a variance estimate.
- SOTA-relative g inherits the source paper's metric slice: a large
  positive g can mean the agent optimized the one scored facet of a
  multi-objective method (authors audit extreme gaps and acknowledge
  this).
- Firewall integrity rests on the same LLM-agent pipeline that builds
  the tasks; "some agent-visible inputs are inherently coupled to their
  targets" on public data means partial answer readout is possible in
  principle. Corporate lab (Frontis) hosting its own leaderboard is a
  mild conflict, mitigated by maintainer-side reproduction and released
  artifacts.
- Four of ten source journals retain zero tasks after filtering — the
  pipeline's feasibility constraints (deterministic metric, public
  ≤50 GB data) select for ML-shaped science, which is also the fairest
  reading of "six domains": scientific *ML* tasks, not wet-lab science.

## Trust signals

- **Credibility:** 4 — Tsinghua/Frontis with Peking and Harvard
  contributors, code + HuggingFace data + public leaderboard released
  with maintainer-side reproduction and unusually thorough build
  verification, but arXiv-only (v2, not peer reviewed) and no citations
  yet.

## Follow-up

- **Relevance:** 4 — canonical evidence for the discovery-vs-reproduction
  split that MLE-bench and PaperBench conflate: seeds
  [[concepts/information-firewall]], attests a third enforcement family
  (seal-and-audit) in [[concepts/hce-evaluation]], and partially closes
  [[concepts/programmable-evaluator-oracle]]'s "evaluator design is
  unautomated" open question via NatureGym.
- Watch for: whether the leaderboard diverges once web search is allowed
  (firewall bypass by retrieval); FrontierCS/ALE-Bench as the
  engineering-side counterparts; the promised use of the substrate as
  training data for scientific-discovery agents.
