---
kind: paper
title: "What Fits (Into Few Tokens) Doesn't Overfit: Compression and Generalization in ML Research Agents"
authors: ["Martin Andres Bertran", "Aaron Roth", "Zhiwei Steven Wu"]
institutions: ["Amazon Responsible AI", "University of Pennsylvania", "Carnegie Mellon University"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.11045
code_url: null
citations: null
source: "raw/papers/bertran2026fits.pdf"
added: "2026-07-07"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/hce-evaluation]]"
tags: [evaluation, overfitting, hce, compression, ml-research-agents, adaptive-data-analysis]
---

# What Fits (Into Few Tokens) Doesn't Overfit: Compression and Generalization in ML Research Agents

## TL;DR

Explains why benchmark-reusing ML research avoids worst-case adaptive
overfitting — successful strategies are highly compressible — and turns
that explanation into an operational, falsifiable *test*: if an ML-research
agent's gains survive compression into a ~32-token prompt handed to a fresh
reproducer with no validation access, they generalize; validation-exploiting
gains fail to reproduce (100% sensitivity, 91% specificity).

## Claims

- **Output compression preserves performance.** An explorer agent's
  adaptively-optimized strategy, compressed to a 32–64-token prompt and
  handed to a fresh reproducer with only the training data (no validation
  set, no code, no transcript), reproduces the explorer's validation
  performance on 38/41 improvement checkpoints (92.7% under a one-sided 5%
  relative-gap criterion) across 8 datasets.
- **Input compression suffices.** Replacing scalar validation feedback with
  a one-bit "ladder mechanism" ("did this model improve on the running
  best?", after Blum & Hardt 2015) matches or exceeds scalar-feedback
  search on holdout performance on all 8 datasets — much of the useful
  validation signal is coarse.
- **The ladder certifies progress.** Because the ladder's binary transcript
  bounds description length, it yields simultaneous 95% confidence
  intervals (half-width ≈0.6–2.5pp at K_max=7, T_max=50) over every
  improvement checkpoint; observed validation–holdout gaps (0.0–1.9pp) sit
  inside them. Non-overlapping CIs certify genuine progress rather than
  validation-set wins.
- **Falsifiable prediction confirmed.** Agents prompted to exploit
  validation ("maximize validation accuracy at all costs", sample-level
  validation access) overfit at 38/102 checkpoints (>10% relative
  validation–holdout gap); those advantages disappear under prompt
  compression, separating honest from exploiting checkpoints with 100%
  sensitivity and 91% specificity.
- Together this supports a description-length explanation for the lack of
  overfitting in benchmark-driven ML: successful strategies occupy a
  low-complexity region of strategy space.

## Methods

- Explorer / compressor / reproducer roles, each a Claude Opus instance
  driven by Claude Code. Bottlenecks enforced by the experiment harness
  (validation reachable only via an evaluation entry point; reproducer runs
  in an isolated workspace with training data + compressed prompt only) —
  not by prompt instructions.
- Theory: adaptive description-length generalization (Dwork et al.) — a
  B-token message class gives a sup-bound ~sqrt(((B+1)ln|V| + ln(2/δ))/2n);
  the ladder's transcript count gives per-checkpoint simultaneous CIs
  (Chernoff-optimized Bernoulli-KL form for classification).
- 8 tasks spanning tabular (Folktables, Gene-Expr), NLP (SST-2), vision
  (CIFAR-10, ImageNet-1K), generative (CIFAR-100 diffusion), reward
  modeling (HH-RLHF), and language modeling (WikiText-103); disjoint
  train/validation/holdout splits, holdout used only post hoc.

## Results

- 32-token reproducers match explorers on 92.7% of improvement checkpoints;
  failures concentrate on ImageNet/CIFAR-10 where the short prompt drops
  detail. Empty-prompt baseline recovers only first-iteration performance,
  so the transfer is genuine.
- One-bit ladder feedback ≥ scalar feedback on holdout across all 8
  datasets (three runs each).
- Overfitting-detection rule (10% val–holdout gap flagged by failure of a
  128-token reproduction): 38/38 exploiting checkpoints flagged, 6/64 false
  positives.

## Critique / open questions

- No released code or artifacts; the harness (role prompts, compressor
  audit) is specified in appendices but not runnable — reproduction cost is
  nontrivial.
- The uniform token-count bound is loose (LM shorthand is far from uniform
  over V^B); the paper flags a PAC-Bayes prior over agent-written prompts
  as open.
- Detection needs a reproducer run per checkpoint — cheap relative to
  exploration but not free; unclear how it scales to multi-day pipelines
  where "hand the training data to a fresh agent" is expensive.
- Single model family (Claude Opus) for all roles; compressibility of
  strategies found by weaker/stronger explorers untested.

## Trust signals

- **Credibility:** 3 — heavyweight authors in adaptive data analysis
  (Roth, Wu; Amazon Responsible AI / UPenn / CMU) with a real theory
  backbone, but an arXiv preprint: not peer-reviewed, no released code, no
  citation record yet.

## Follow-up

- **Relevance:** 4 — seeds `compression-as-generalization-test` (an
  operational certify-after-search mechanism downstream HCE projects could
  adopt as a cheap overfitting audit) and materially extends
  `hce-evaluation` beyond split hygiene with two enforcement-layer
  mechanisms (ladder feedback, reproducer certification).
- The ladder mechanism is a *constrain-during-search* remedy and the
  reproducer test a *certify-after-search* remedy — the same two families
  [[literature/papers/ning2026closedloop]] frames; this paper supplies the
  formal bounds side.
- If a downstream MLE-bench-style project wants an overfitting audit
  beyond HCE split discipline, the 128-token reproduce-or-flag rule is
  directly implementable with existing skills (/implement in an isolated
  worktree).
