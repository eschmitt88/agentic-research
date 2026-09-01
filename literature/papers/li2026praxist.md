---
kind: paper
title: "From Experimental Artifacts to Solution Lineages"
authors: ["Jin Li", "Ahmed Murtadha", "Zhiyu Wang", "Qiwen Chen", "William Chen", "Yifei Wu", "Guan Wang", "Andy L. Siy", "Jiayi Yang", "Mengsha Huang", "Wenhao Li", "Yixuan Liu", "Shuailin Pan", "Mingli Yuan", "Sen Song", "Yuhao Sun"]
institutions: ["Sapient Intelligence", "Nanyang Technological University", "Tsinghua University", "Carnegie Mellon University", "University of Pennsylvania"]
year: 2026
venue: "arXiv (cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.25955"
code_url: "https://github.com/sapientinc/praxist"
citations: null
source: "raw/papers/li2026praxist.pdf"
added: "2026-09-01"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["ml-research-agents", "knowledge-organization", "memory", "evaluation", "compute-economy", "provenance"]
---

# From Experimental Artifacts to Solution Lineages (PRAXIST)

## TL;DR

Autonomous R&D campaigns keep re-learning the same lessons because each
attempt is treated as self-contained — logs and search trees record *what
happened* without establishing *which design element caused an improvement*
or *whether its evidence survived validation*. PRAXIST converts evaluated
artifacts into a **typed evidence graph** that later generations inherit
selectively. On MLE-bench it beats a Claude Code baseline while spending
roughly **one twelfth** as much.

## Claims

- The limitation of current autonomous R&D systems is **structural, not
  capability-bound**: attempts are nearly self-contained, so nothing
  accumulates that a later attempt can inherit.
- Evidence should be **typed and staged**, not merely stored. A result
  carries not just a measurement but how far it can be trusted and what
  should be done about it.
- Separating **local artifact construction** from **cohort-level evidence
  synthesis** is what lets inheritance work — the synthesis step is where
  attribution happens, and it cannot happen inside a single attempt.
- Selective inheritance is a **cost mechanism**, not only a quality
  mechanism: a peer receives only the frontier entries, Gems, and parent
  lineage relevant to its assigned direction, so context stays bounded as
  the campaign grows.

## Methods

The pipeline is a fixed vocabulary: **Artifact → Finding → Frontier →
Agenda → Lineage**, iterated over *generations* (a generation is the
synchronization boundary at which peer-level evidence becomes shared state).

- State inherited into generation *g* is `S_g = (F_g, A_g, G_g, L_g)` —
  frontier, agenda, Gems, lineage trace.
- **Frontier lanes** make validation status explicit. The configuration
  used here: `confirmed` (mature enough to serve as a parent or
  constraint), `candidate` (promising but immature), `diagnostic`
  (failures, controls, failure modes that shape exploration), `validation`
  (scheduled for reproduction or ablation before promotion). Lanes are
  task-owned — the trading campaign declares its own set.
- **Agenda**: a panel of PI roles plus a Chair reads the pooled findings
  and issues per-direction verdicts — **continue, stop, validate, explore**.
- **Gems**: every ρ generations, memory compression distils the balanced
  frontier into a small bounded set of durable lessons (a validated
  mechanism, a rejected assumption, a recurring failure mode, a procedural
  constraint), deliberately preserving control and diagnostic lessons and
  not only high scorers. Off by default; only the trading campaign uses it
  (ρ=6, ≤4 active Gems).
- **Lineage** is materialized as correlated ledgers sharing identifiers,
  with typed relations `derived-from`, `supports`, `challenges`, `updates`.
  Accumulated during the run, not reconstructed afterwards.
- Evaluation: all **75 MLE-bench tasks** against a local Claude Code
  baseline on Claude Opus 4.8, plus four open-ended case studies.

## Results

- **MLE-bench (75 tasks, official grader):** PRAXIST **60 medals (80.0%),
  49 gold**; Claude Code baseline **55 medals (73.3%), 34 gold**.
- **Cost: US$3,054 vs US$38,370** — roughly one twelfth, at a better
  result. This is the number that makes the paper interesting.
- Four open-ended case studies with mixed, honestly-reported outcomes:
  - *Quantitative trading*: 53% walk-forward CAGR vs 23% for a paired
    equal-weight baseline (2.3×).
  - *Rocket landing*: deterministic first-contact landing constructed.
  - *LiDAR-inertial-visual SLAM*: shows SOTA LIVO systems over-spend visual
    compute; cuts it at no cost in trajectory accuracy — but the two arms
    stamp poses differently, so accuracy columns are reported side by side
    rather than as a gain.
  - *Tokamak magnetic control*: leads on survival but **not** on
    full-horizon precision.

## Critique / open questions

- The case studies are explicitly **not** standardized cross-system
  comparisons, and the authors say so. The rocket study's external
  reference point is described as "an outside reference point rather than a
  matched-model or matched-compute comparison". The MLE-bench result is the
  only one with a controlled baseline.
- The cost comparison is against *a local Claude Code baseline*, not
  against a cost-optimized baseline. A twelvefold spend gap is large enough
  to survive some slack, but "Claude Code run naively" is a soft
  denominator, and the baseline's per-task costs are relegated to an
  appendix.
- The vocabulary is large — findings, lanes, stages, agendas, Gems, four
  typed relations, per-task-configurable lane sets. The paper argues these
  are "method-level roles, not literal strings", but the ablation of *which
  parts of the vocabulary earn their keep* is not the paper's focus.
- Gems — the part most transferable to a note-taking knowledge graph — are
  off by default and exercised in exactly one case study. The
  memory-compression claim is the least evidenced part of the system.

## Trust signals

- **Credibility:** 4 — code released at `sapientinc/praxist` plus a project
  site, all 75 MLE-bench tasks under the finalized official grader (not a
  self-selected subset), a controlled same-harness baseline, recorded
  dollar spend, and case studies whose negative results are reported rather
  than buried (tokamak precision, SLAM's non-comparable pose stamps). Not
  peer reviewed, and the author list is led by a company with obvious
  interest in the result, which is what holds it at 4 rather than 5.

## Follow-up

- **Relevance:** 5 — this is the closest thing in the literature to *this
  project's own thesis, built and measured*. The `Artifact → Finding →
  Frontier → Agenda → Lineage` chain is structurally the same move as
  `raw/ → literature/ → concepts/ → mocs/`, and the frontier lanes are a
  sharper version of what [[concepts/typed-claim-partition]] argues for.
- **The lane vocabulary is directly importable.** This project's concept
  notes currently carry `status: active | seedling | retired`, which
  conflates "how mature is the idea" with "what should be done about it".
  PRAXIST's split — `confirmed` / `candidate` / `diagnostic` / `validation`
  — separates trust level from operational disposition, and the
  `diagnostic` lane names something this graph has no home for: recorded
  failures and negative controls that shape future exploration.
  [[literature/papers/ge2026coverage]] (same curate pass) is exactly a
  `diagnostic`-lane artifact with nowhere to sit.
- The `continue / stop / validate / explore` agenda verdicts map onto
  `/promote-moc`'s and `/elevate`'s decision vocabulary, both of which
  currently emit a binary promote-or-decline. "Validate" in particular is a
  disposition `/elevate` keeps reaching for and expressing as a prose hold.
- Extends [[concepts/selective-memory-retrieval]] with a scaling argument:
  inheritance is selective *because* a long run produces more evidence than
  any context can hold, so relevance filtering is forced by the
  architecture rather than chosen.
- Bears on [[concepts/budget-as-ceiling]]: a twelvefold spend reduction
  attributed to process rather than model is the strongest available
  argument that the ceiling should be spent on synthesis between attempts
  rather than on more attempts.
- Pairs with [[literature/papers/moukpe2026deltaml]] (same curate pass):
  both conclude that scaffolding, not model choice, is the dominant term —
  moukpe on integrity, li on cost and quality.
