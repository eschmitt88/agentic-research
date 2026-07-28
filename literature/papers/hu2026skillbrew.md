---
kind: paper
title: "SkillBrew: Multi-Objective Curation of Skill Banks for LLM Agents"
authors: ["Wentao Hu", "Zhendong Chu", "Yiming Zhang", "Junda Wu", "Ming Jin", "Xiangyu Zhao", "Yilei Shao", "Yanfeng Wang", "Qingsong Wen"]
institutions: ["City University of Hong Kong", "Squirrel Ai Learning", "University of Science and Technology of China", "University of California, San Diego", "Griffith University", "East China Normal University", "Shanghai Jiao Tong University"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.29440"
code_url: null
citations: null
source: "raw/papers/hu2026skillbrew.pdf"
added: "2026-07-28"
relevance: 4
credibility: 3
status: read
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/selective-memory-retrieval]]"
related_experiments: []
tags: [skills, curation, pareto, multi-objective, pruning, deprecation, counterfactual]
---

# SkillBrew: Multi-Objective Curation of Skill Banks for LLM Agents

## TL;DR

Skill banks grow append-only, accumulating redundant, outdated, and
harmful entries. SkillBrew reframes curation as **constrained
multi-objective optimization**: maximize diversity and coverage on the
Pareto frontier subject to a minimum *utility* constraint, solved by a
bi-level propose-then-verify loop with per-skill counterfactual
leave-one-out credit assignment.

## Claims

- Existing skill-bank work judges skills **in isolation** and collapses
  curation to a single scalar criterion. Bank-*level* properties —
  diversity, coverage of the query distribution — are a distinct and
  under-explored dimension.
- These objectives genuinely conflict and cannot be collapsed: pushing
  coverage introduces redundancy that lowers diversity; pruning for
  diversity reduces coverage. Hence a Pareto formulation rather than a
  weighted sum.
- **Utility is a constraint, not an objective.** A bank must first be
  useful; diversity and coverage act as *structural regularizers* over an
  already-useful bank. The asymmetry is deliberate — organization matters
  only after usefulness is established.
- Treating a skill bank as an object of principled curation rather than an
  ever-growing append-only log is a prerequisite for self-improving agents.

## Methods

- **Objective**: B* ∈ Pareto-argmax ⟨J_util(B), J_div(B), J_cov(B)⟩ subject
  to J_util ≥ η. Utility = does the bank improve task performance;
  diversity = redundancy within the bank; coverage = spread over the query
  distribution.
- **Credit assignment by counterfactual replay**: leave-one-out rollouts
  isolate each skill's marginal contribution, yielding per-skill
  KEEP / REWRITE / REMOVE evidence. A content-addressed cache reduces the
  cost of repeated rollouts.
- **Bi-level propose-then-verify loop** to navigate a non-differentiable,
  open-ended edit space: trajectory evidence on a **support split**
  proposes candidate edits, and Pareto-aware selection on a **held-out
  query split** verifies them — explicitly "preventing overfitting to the
  trajectories that motivated each edit."
- Frozen worker (Qwen2.5-7B-Instruct for the main table); evaluated against
  ten training-free baselines. Training-based methods that jointly update
  the worker policy and bank (e.g. SkillRL) are excluded as
  non-comparable under the frozen-worker setting.

## Results

- Two benchmarks: **ALFWorld** (six household task families requiring
  compositional action) and **WebShop** (score + success rate).
- Outperforms all ten training-free baselines, including **+12.0% over
  append-only Voyager** — the direct accumulation-vs-curation comparison.
- Improves over ReAct on all six ALFWorld task types.
- Cross-worker generalization: a bank curated with one worker transfers to
  workers of different scales, with success rate remaining substantially
  above each worker's own ReAct baseline.

## Critique / open questions

- Authors' own limitations, all fair: the worker stays frozen (no joint
  co-adaptation between agent and evolving bank); leave-one-out replay is
  expensive and may not scale despite the cache; the formulation assumes a
  relatively stable query distribution.
- The "coverage of the query distribution" objective presupposes you know
  the query distribution. In a research-agent setting the distribution is
  precisely what is shifting, and a bank optimized for coverage of
  yesterday's queries is a different failure mode from an append-only log
  but not obviously a smaller one.
- No released artifact.
- The corresponding-author email is at a domain unrelated to any listed
  affiliation, and the affiliation list is unusually long for the
  contribution — minor, but worth noting when weighing provenance.
- Diversity and coverage are measured by the framework's own metrics, so
  "SkillBrew improves diversity and coverage" is partly definitional. The
  load-bearing number is the downstream task improvement, which is
  independent — and it is reported, so this is a presentation caveat rather
  than a validity problem.

## Trust signals

- **Credibility:** 3 — sound formulation, honest baseline scoping
  (excluding training-based methods rather than beating them unfairly),
  ten baselines, cross-worker transfer, candid limitations. Held at 3 by
  the absence of peer review, no released code, two benchmarks both in the
  well-worn ALFWorld/WebShop family, and a diffuse authorship footprint.

## Follow-up

- **Relevance:** 4 — the **deprecation/consolidation half** of
  [[concepts/skill-library-lifecycle]] that SkillOps
  ([[literature/papers/pu2026skillops]]) leaves to policy. The concept has
  been strong on what enters a library and weak on what leaves it; this
  supplies a formal criterion for removal.
- Complements [[literature/papers/shang2026hypothesis]] ingested the same
  day, and the pair covers the lifecycle's two gates cleanly: HDSO governs
  **admission** (does this candidate survive a falsifiable paired test),
  SkillBrew governs **retention** (given a bank, which members earn their
  place). They are not competing designs and could compose directly.
- **Utility-as-constraint is the transferable stance**, not the Pareto
  machinery. It says: never trade measured usefulness for tidiness. The
  same ordering applies to this project's own graph — concept notes should
  be pruned for redundancy only after they are established as load-bearing,
  and `/promote-moc`'s restraint about not manufacturing thin MoCs is the
  same instinct.
- The **support-split / held-out-query-split separation** is
  [[concepts/hce-evaluation]] applied inside the curation loop, and it is
  the discipline [[literature/papers/shang2026hypothesis]] lacks — HDSO
  validates promotion on the same task indices it reports gains on.
  SkillBrew gets this right and says why: the edits must not be verified on
  the trajectories that motivated them.
- Connects to [[concepts/context-eviction-policy]]: both answer "what gets
  dropped when the store outgrows its budget," one for context windows and
  one for skill banks, and both conclude that recency and raw utility are
  insufficient eviction signals on their own.
