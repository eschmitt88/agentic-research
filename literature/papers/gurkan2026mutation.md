---
kind: paper
title: "Mutation Without Variation: Convergence Dynamics in LLM-Driven Program Evolution"
authors: ["Can Gurkan", "Forrest Stonedahl", "Uri Wilensky"]
institutions: ["Northwestern University", "Augustana College"]
year: 2026
venue: "GECCO '26 Workshop on LLMs for and with Evolutionary Computation (arXiv 2606.05408)"
peer_reviewed: true
url: https://arxiv.org/abs/2606.05408
code_url: https://github.com/can-gurkan/lmca
citations: null
source: "raw/papers/gurkan2026mutation.pdf"
added: "2026-08-03"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hybrid-model-backends]]"
tags: [evolutionary-search, mutation-operator, diversity, attractors, dynamical-systems, genetic-programming]
---

# Mutation Without Variation: Convergence Dynamics in LLM-Driven Program Evolution

## TL;DR

Strip selection pressure away and study the LLM mutation operator
alone: repeated LLM mutation of programs converges into small
attractor regions of program space — in 87% of chains, >93% of
mutations revisit a previously seen structural form, with variation
mostly confined to terminal substitutions inside a fixed template.
Classical GP subtree mutation on the same setup shows no comparable
convergence: the bias is intrinsic to the LLM operator, not the
search space.

## Claims

- LLM-based mutation is not a neutral variation operator: it carries
  a systematic bias toward canonical structures favored by the
  training distribution, producing "mutation without variation."
- Convergence is two-level: lexically distinct programs keep
  appearing while the *skeleton* (control-flow structure) freezes —
  median chain visits ~10 unique skeletons in 300 steps; 87% of
  chains visit <20 (GP baseline: ~143).
- Transition-graph structure: 2-cycles dominate at the program level,
  self-loops at the skeleton level (some prompts: mean skeleton cycle
  length 1.04 — chains spend 300 steps cycling inside one template).
- Prompt wording modulates severity strongly but unpredictably —
  semantically similar prompts produce very different convergence
  profiles; only a small fraction of 50 prompts sustained
  exploration.
- Model choice matters and is prompt-invariant: Claude Sonnet 4
  collapsed to 6 unique programs / 1 skeleton; GPT-5 Mini *with
  reasoning* sustained near-complete exploration (301 programs / 225
  skeletons); reasoning-enabled variants consistently out-explore
  their base counterparts.
- Since prior work (LLaMEA behavior-space analysis, Digital Red
  Queen) observed clustering *under* selection, selection may *mask*
  structural convergence rather than prevent it.

## Methods

- Neutral mutation chains (every valid mutation accepted, no
  fitness), 300 steps, in a strongly typed Lisp-like gridworld DSL;
  dual representation: full program vs. skeleton AST (terminals
  abstracted to type categories).
- Experiments: 50 prompt variants × 3 initial program sizes (150
  chains); 30 replications × 4 representative prompts; 7 models × 4
  prompts. Classical GP subtree mutation (max depth 4) as baseline,
  same initial programs and chain length.
- Measures: cumulative unique programs/skeletons, directed transition
  graphs with cycle-length distributions, degree entropy, pairwise
  Levenshtein distance.

## Results

- 71% of LLM chains visit <100 unique programs and 87% <20 unique
  skeletons over 300 steps; GP baseline ~270 programs / ~143
  skeletons on identical conditions — DSL size is not the limiting
  factor.
- Convergent prompts plateau within the first 50–100 steps; the
  most convergent prompt produced <10 unique programs in 300 steps.
- The four representative prompts (Table 3) differ only in phrasing
  ("exploratory modifications" vs. "a small mutation applied") yet
  span near-linear exploration to total collapse.

## Critique / open questions

- Single constrained DSL; generalization to rich languages (more
  strongly represented in pretraining — plausibly *stronger*
  attractors) is untested.
- Purely genotypic analysis: structurally identical programs can
  behave differently, and structural convergence with behavioral
  variation would matter less. The Digital Red Queen comparison cuts
  both ways.
- Model-sensitivity experiment is 1 chain per (model, prompt) — no
  replication; the striking Claude-collapse and reasoning-model
  results are single observations.
- No test of whether selection pressure, islands, or QD archives
  actually escape the attractors — flagged as future work, which
  limits the actionable conclusion to "diversity machinery is
  load-bearing," not "which machinery suffices."

## Trust signals

- **Credibility:** 3 — accepted at a GECCO '26 workshop (light peer
  review), code released, established EC group (Wilensky/NetLogo
  lineage); constrained toy domain and unreplicated model-comparison
  cells keep it below 4.

## Follow-up

- **Relevance:** 4 — names the axis
  [[concepts/evolutionary-search-grain]] assumes away: the mutation
  *operator's* entropy, not the grain, can bind the search. A
  fine-grained diff operator that only ever proposes moves inside an
  attractor explores less than its grain suggests. Also the
  mechanistic justification for why islands / MAP-Elites in
  FunSearch/AlphaEvolve ([[concepts/evolutionary-expansion]]) are
  load-bearing rather than decorative: the operator supplies less
  variation than assumed, so the population machinery must.
- The reasoning-vs-base model diversity gap is a new criterion for
  [[concepts/hybrid-model-backends]]: ideator-model choice affects
  search *diversity*, not just proposal quality — worth a line when
  that concept next updates.
- Method to reuse: neutral-chain (selection-free) probing of a
  mutation operator before trusting it in a long search — cheap to
  run for any downstream evolutionary experiment; prompt-profile the
  operator first (the paper shows prompt choice swings exploration
  by 25×).
