---
kind: paper
title: "AIRS-Bench: a Suite of Tasks for Frontier AI Research Science Agents"
authors:
  - Alisia Lupidi
  - Bhavul Gauri
  - Thomas Simon Foster
  - Bassel Al Omari
  - Despoina Magka
  - Alberto Pepe
  - Alexis Audran-Reiss
  - Muna Aghamelu
  - Nicolas Baldwin
  - Lucia Cipolina-Kun
  - Jean-Christophe Gagnon-Audet
  - Chee Hau Leow
  - Sandra Lefdal
  - Hossam Mossalam
  - Abhinav Moudgil
  - Saba Nazir
  - Emanuel Tewolde
  - Isabel Urrego
  - Jordi Armengol Estape
  - Amar Budhiraja
  - Gaurav Chaurasia
  - Abhishek Charnalia
  - Derek Dunfield
  - Karen Hambardzumyan
  - Daniel Izcovich
  - Martin Josifoski
  - Ishita Mediratta
  - Kelvin Niu
  - Parth Pathak
  - Michael Shvartsman
  - Edan Toledo
  - Anton Protopopov
  - Roberta Raileanu
  - Alexander Miller
  - Tatiana Shavrina
  - Jakob Foerster
  - Yoram Bachrach
institutions: ["FAIR at Meta", "University of Oxford", "University College London"]
year: 2026
venue: "arXiv:2602.06855 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2602.06855"
code_url: "https://github.com/facebookresearch/airs-bench"
citations: null
source: "raw/papers/lupidi2026airsbench.pdf"
added: "2026-07-20"
relevance: 3
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
tags: [benchmark, ml-research-agents, evaluation-protocol, seeds, elo, harness, aira-dojo, mlgym]
---

# AIRS-Bench: a Suite of Tasks for Frontier AI Research Science Agents

## TL;DR

Meta FAIR's 20-task benchmark for end-to-end AI research agents, sourced
from 17 SOTA ML papers across 7 domains (NLP, molecules/proteins, time
series, code, math), with no baseline code provided — agents must ideate,
implement, and iterate from scratch against the paper's published SOTA.
Baselines (5 LLMs × one-shot/greedy/ReAct scaffolds in AIRA-dojo and
MLGym, 24h on one H-200, ≥10 seeds each) beat human SOTA on only 4 of 20
tasks; overall valid-submission rate is 55.1% and mean normalized score
24.1% — the benchmark is far from saturated.

## Claims

- Existing agentic-research evaluation suffers an "evaluation crisis"
  driven by three factors: data contamination (LLMs memorize benchmark
  solutions), environmental non-standardization (unclear whether success
  is the agent or the bespoke environment), and computational cost (too
  few trials for statistical significance).
- Withholding the baseline solution is the key design choice: it forces a
  longer reasoning horizon, promotes original research over
  patch-the-starter-code behavior, and de-biases the evaluation. Table 1
  positions this against 14 adjacent benchmarks (MLE-bench, PaperBench,
  MLGym-Bench, RE-Bench, CORE/CSR-Bench, …); few combine full
  scientific-method coverage + long horizon + no baseline + high compute.
- A {problem, dataset, metric} task configuration standard
  (`metadata.yaml` + `project_description.md` + `prepare.py` /
  `evaluate_prepare.py` / `evaluate.py`) makes tasks portable across
  harnesses — demonstrated by programmatic AIRA-dojo→MLGym conversion —
  enabling apples-to-apples comparison of agent frameworks.
- Even where agents exceed human SOTA they stay short of the theoretical
  task ceiling, so headroom exists in both directions (agent vs SOTA,
  SOTA vs optimum).

## Methods

- Task sourcing: PapersWithCode leaderboards filtered to datasets with
  2020–2025 published SOTA, HuggingFace availability, and train/test
  splits; ~100 candidate tasks from ~85 papers manually created,
  reviewed, and verified down to 20 (16 datasets, 7 categories; NLP is
  the largest share at 9/20).
- Agent = LLM × scaffold, instantiated in a harness: AIRA-dojo (tree
  search over Draft/Debug/Improve operators; greedy policy) and MLGym
  (sequential ReAct). LLMs: CWM, GPT-4o, o3-mini, gpt-oss-20b/120b,
  Devstral-Small — 14 agent combinations total.
- Uniform budget per run: 24h, one H-200 GPU, ≥10 seeds per agent-task
  pair; local HF checkpoint cache (nothing newer than 2021-dated cache
  policy for foundation models) to mitigate rate limits; agents get no
  information about the SOTA paper's methodology or score. Test labels
  are stripped from the agent-visible split; only `evaluate.py` sees them.
- Three-metric evaluation protocol: mean valid submission rate; average
  normalized score under a "march of 9s" transform
  (φ(s) = −log10|s − s_opt|, mapping worst-observed→0 and SOTA→1, so
  0.99→0.999 counts like 0.9→0.99), with failed/invalid runs scored 0;
  and Elo via an order-invariant Bradley–Terry fit over pairwise
  task-level comparisons, with human SOTA included as an extra "player".

## Results

- 4/20 tasks have an agent exceeding human SOTA; only 1.58% of all
  agent-task combinations exceed SOTA, driven mostly by greedy search.
  Case study: Greedy gpt-oss-120b beats SOTA on SICK NLI classification.
- Overall VSR 55.1% — merely *submitting* a valid solution stretches
  current agents. Best agents by VSR: ReAct CWM 91.0%, Greedy o3-mini
  85.6%; worst: One-Shot Devstral 5.8%.
- Mean normalized score 24.1% overall; Greedy gpt-oss-120b/20b and
  Greedy o3-mini lead (~0.39–0.40). Reasoning models win in both
  one-shot and greedy settings; model size barely matters once
  test-time search is added (Greedy gpt-oss-120b ≈ 20b); tree search
  helps open and closed models alike.
- VSR and score rank agents differently — agents "willing to take risks"
  submit less reliably but score higher when they do.
- The Elo gap between the human-SOTA player and the best agent is large;
  tasks bucket into easy/medium/hard/expert with expert-bucket scores
  near zero across all 14 agents.

## Critique / open questions

- "Uncontaminated" is argued from no-baseline-code and label stripping,
  but the tasks come from published 2020–2025 papers that frontier LLMs
  have plausibly read — idea-level contamination (remembering the SOTA
  paper's method) is mitigated by withholding the citation, not
  eliminated. The claim deserves a probe (e.g. asking models to name the
  SOTA paper from the task description).
- The uniform 24h/H-200 budget is acknowledged by the authors as
  under-resourcing some tasks; capability and budget-fit are conflated
  at the task level.
- Counting failed/invalid runs as normalized 0 folds reliability into
  capability — deliberate, but it means score comparisons partly re-measure
  VSR.
- SOTA anchors are sourced from PapersWithCode leaderboards, which can
  be stale or contested; the authors did manually re-verify each number
  against the cited paper, which is better practice than most.
- Small internal inconsistency: body text reports overall VSR 55.1%,
  Figure 7's caption says 59.3% across all runs and agents.
- Baseline-free design cuts both ways: it tests ideation, but makes
  per-task failure analysis harder (no reference implementation to diff
  against).

## Trust signals

- **Credibility:** 4 — FAIR at Meta with Oxford and UCL; task
  definitions and evaluation code released
  (github.com/facebookresearch/airs-bench); careful protocol (≥10 seeds,
  CIs, order-invariant BT-Elo, human verification of SOTA anchors); but
  arXiv-only, not peer-reviewed, no citations established yet.

## Follow-up

- **Relevance:** 3 — the citation anchor for AIRS-Bench claims already
  in the graph: [[literature/papers/hambardzumyan2026aira]] (AIRA_2)
  evaluates on this suite ("exceeds human baseline on 6 of 20 tasks" —
  note this paper's own baselines reach 4/20, so AIRA_2's 6/20 is an
  agent improvement on the same yardstick). Useful prior art on the
  ML-research-agent benchmark theme; strengthens
  [[concepts/pass-at-k]] with a working ≥10-seed protocol on expensive
  tasks but shifts no concept's architecture.
- The shared author set with AIRA_2 (Hambardzumyan, Toledo, Baldwin,
  Foster, Al Omari, Josifoski, Bachrach, Foerster, …) makes this
  effectively the eval-suite half of the same FAIR program — read
  results across the two papers as one lab's benchmark + agent story,
  not independent confirmation.
- Watch for: adoption of the task configuration standard by third-party
  harnesses (the portability claim's real test), and whether the
  march-of-9s transform gets picked up as a normalization convention for
  heterogeneous-metric agent benchmarks.
