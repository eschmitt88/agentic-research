---
kind: paper
title: "Generative Skill Composition for LLM Agents"
authors: ["Xinyu Zhao", "Zhen Tan", "Vaishnav Tadiparthi", "Nakul Agarwal", "Kwonjoon Lee", "Ehsan Moradi Pari", "Hossein Nourkhiz Mahjoub", "Tianlong Chen"]
institutions: ["University of North Carolina at Chapel Hill", "Arizona State University", "Honda Research Institute USA"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.32025
code_url: https://skill-composer.github.io/
citations: null
source: "raw/papers/zhao2026generative.pdf"
added: "2026-07-13"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/shared-skill-namespace]]"
tags: [skills, skill-composition, retrieval, small-models, context-efficiency]
---

# Generative Skill Composition for LLM Agents

**Naming caution:** this paper's system is called `SkillComposer`, which
collides with the unrelated `SkillComposer` of
[[literature/papers/zhang2026skillcomposer]] (Qi Zhang et al., 2606.06079,
"Learning to Evolve Agent Skills"). Different authors, different problem
(composition-time selection here vs skill evolution there). Cite by citekey,
not system name.

## TL;DR

Formalizes *structured skill composition* — given a task and a fixed skill
library, jointly predict **which** skills to load, **how many**, and **in
what order** — as closed-vocabulary autoregressive sequence prediction, and
shows a ~3.9M-trainable-parameter specialist decoder beats frontier LLM-judge
and retrieval baselines, closing ~80% of the gap to the gold-skill oracle on
SkillsBench at the lowest prompt-token cost.

## Claims

- Skill selection interfaces used today (expose-everything reasoning, or
  flat embedding retrieval) miss the structural nature of composition:
  subset, count, and order are a joint decision that cannot be decoupled.
- A small task-conditioned decoder over skill identifiers (frozen
  Qwen3-Embedding-0.6B encoder + 3-layer AR decoder + cardinality head +
  set-membership head + TF-IDF retrieval prior fused into decode logits)
  resolves all three jointly in one decoding pass.
- Better skill plans translate to better agent execution: on SkillsBench
  with GPT-5.2-Codex / Gemini-3-Pro-Preview, pass rate +23.1 / +18.2 pp
  over no-skill baseline, beating retrieval (top-3) and matching or
  exceeding *oracle* retrieval, at 1.03M prompt tokens vs 1.27M for
  loading the full library.
- Under distribution shift (synthetic-train → real-task holdout), SFT of
  the full 600M backbone degrades sharply (−27.5 pp) while the specialist
  composer degrades gracefully (−11 pp; +19.3 pp Set F1 gap over SFT) —
  the frozen retrieval-tuned encoder supplies the transfer robustness.
- For closed skill libraries, a small specialist that exploits library
  structure is a more reliable composer than scaling up a generalist LM
  (~154× fewer trainable params than SFT, ~25× less training compute,
  two orders of magnitude lower latency than an API judge).

## Methods

- Definitions: skill = (metadata, applicability condition, procedural
  policy, termination condition, resources), per the open Agent Skills
  standard (agentskills.io); library = fixed set of K skills exposing
  compact metadata for progressive disclosure (metadata at startup, full
  instructions on demand).
- Task-conditioned skill sequence prediction: output vocabulary = skill
  indices + STOP; beam search (width 4) with duplicate-skill constraint;
  fused logits = contextual AR logit + α·TF-IDF relevance + β·set-head
  prior (α=1.0, β=0.5 tuned on validation Set F1).
- Training corpus: 9,872 task–skill-sequence records over SkillsBench's
  196-skill human-curated library — 65 real gold-annotated tasks + 2,880
  single-skill and 6,927 multi-skill synthetic records sampled from a
  skill dependency graph (dependency edges from I/O-type overlap, workflow
  edges from co-occurrence). 90/5/5 train/val/test.
- Evaluation on two axes: composition quality (Set F1, Recall@5, MRR,
  nDCG@5, Set EM) on in-distribution and real-task-holdout regimes, and
  downstream pass rate on 75 SkillsBench tasks with deterministic pytest
  verifiers (Harbor framework), 3 attempts per task at temperature 0.

## Results

- Synthetic test: 73.9 Set F1 vs 71.1 SFT and 61.0 LLM-judge
  (Gemini-2.5-flash); leads MRR and nDCG@5. Trained models beat even
  oracle-cardinality retrieval ceilings.
- Real-task holdout: 62.9 Set F1 vs 43.6 SFT, 59.9 LLM-judge.
- Downstream: 45.3 / 44.0 pass rate on Codex / Gemini vs 22.2 / 25.8
  no-skill, 29.3 / 38.7 all-skills, 44.0 / 41.8 retrieval-top-3;
  gold-skill ceiling 51.1 / 48.4.
- Ablations: every component load-bearing — zeroing set-fusion costs
  −7.1 pp, zeroing retrieval prior −4.6 pp. Sparse TF-IDF beats dense
  embeddings as the decode-time prior (+2.5 pp) because the closed
  library's short syntactic skill names reward token-level precision.

## Critique / open questions

- Library is *fixed* at training time by design ("real agentic deployments
  ship with curated skill packs") — deliberately isolates composition from
  skill creation, but that means the composer must be retrained (or at
  least re-embedded) as the library evolves. How this interacts with a
  living skill-lifecycle (SLIM-style retire/expand) is the open seam.
- Trained on one library (SkillsBench, K=196); cross-library transfer
  untested. Cardinality capped at N_max=8.
- Evaluation hygiene is notably good for the genre: explicit
  train/val/test separation, a real-task holdout regime that removes all
  real tasks from training, deterministic verifiers, and val-only tuning
  of decode weights — consistent with HCE discipline.
- No code repo visible yet (project page only); weights/data pipeline in
  appendices but reproducibility unverified.

## Trust signals

- **Credibility:** 3 — reputable academic groups (UNC, ASU) + industrial
  lab (Honda Research Institute USA); arXiv preprint under review, not yet
  peer-reviewed; project page but no verified code release; no citations
  yet (June 30 2026). Methodologically careful (ablations, holdout regime,
  deterministic verifiers).

## Follow-up

- **Relevance:** 4 — materially strengthens `skill-library-lifecycle`
  (composition/selection as its own lifecycle stage, formalized as joint
  which/how-many/order prediction) and `hybrid-model-backends` (strong
  quantified case for a small specialist beating a frontier generalist on
  a structured sub-decision); also a deployed-standard attestation for
  `shared-skill-namespace` (builds on agentskills.io skill definition and
  progressive disclosure).
- Watch for the code release on the project page; if the dataset-synthesis
  pipeline lands, it is a template for building composition supervision
  from any curated skill pack.
- Pair with SLIM (2605.10923, this digest) — retire/expand lifecycle ops
  upstream of a fixed-library composer is exactly the unresolved seam.
