---
kind: paper
title: "From Memory to Skills: Evidence-Grounded Co-Evolution Governance for Long-Horizon LLM Agents"
authors: ["Bo Tang", "Yang Zhang", "Guomian Zhuang", "Wenqiang Wei", "Gaoyang Zheng", "Lindong Xie", "Yanchao Tan", "Feiyu Xiong", "Qingyu Yang", "Edward Chung", "Zhiyu Li"]
institutions: ["MemTensor", "University of Science and Technology of China", "Hong Kong Polytechnic University", "Fuzhou University", "Xi'an Jiaotong University"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.16621"
code_url: "https://github.com/MemTensor/MemOS"
citations: null
source: "raw/papers/tang2026memory.pdf"
added: "2026-07-27"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/agent-native-memory]]"
tags: [skills, memory, promotion, lifecycle, evidence-anchoring, self-evolution, credit-assignment, training-free]
---

# From Memory to Skills: Evidence-Grounded Co-Evolution Governance for Long-Horizon LLM Agents

## TL;DR

Argues that distilling skills directly from raw trajectories is the
mistake — raw traces carry failed attempts, blind exploration, and
environment-specific artifacts, so skills built from them are brittle
and fire in the wrong contexts. MSCE instead promotes skills out of a
*governed* three-level memory: L1 grounded step traces → L2 recurring
cross-episode policies → L3 declarative environmental cognition, with
only evidence-backed, positive-gain, internally-consistent L2 policies
crystallizing into callable skills. Training-free. Best or tied-best on
all five EvoAgentBench domains (+15.4 pts on software engineering)
while usually *reducing* cost.

## Claims

- **Memory is currently wasted as passive context.** Most systems
  retrieve prior traces and make the agent re-reason over them. The
  paper's example is concrete: after repeatedly installing plugins, an
  agent still re-lists directories to find tests and build scripts,
  burning tokens re-deriving what it has already established.
- **Skills need more than a working procedure.** A reusable skill
  requires triggers, applicability boundaries, verification criteria,
  and lifecycle maintenance. Anything less overfits to the trajectory it
  came from.
- **Promotion should be gated, and the gate is three-part.** An L2
  policy becomes a skill only if it (a) retains supporting evidence,
  (b) shows positive estimated gain, and (c) remains consistent across
  its trigger, procedure, and applicability boundary. Skills carry
  evidence anchors, verification rules, and reliability estimates
  forward — the provenance survives promotion rather than being
  compiled away.
- **Reflection-weighted value backfilling addresses step-level credit
  assignment.** Terminal feedback is sparse and delayed, so MSCE
  propagates it backward over grounded traces using *dense local
  self-reflections* as weights, producing evidence-calibrated trace
  values. One signal then governs everything downstream: retrieval,
  policy induction, environmental abstraction, skill promotion, and
  revision.
- **Separating the three levels is what makes governance possible** —
  evidence (L1), procedure (L2), and environmental knowledge (L3) are
  distinct kinds of thing, and collapsing them removes the ability to
  verify or revise any of them independently.

## Methods

- Training-free; no base-model updates. GPT-5.2 as agent backbone,
  GPT-4o for auxiliary prompted operators (reflection scoring, L2
  induction, L3 abstraction). All methods on the OpenClaw v2026.5.7
  runtime with identical tool access, interaction budgets, and task
  order.
- **EvoAgentBench** across five domains: information retrieval
  (BrowseComp-Plus), mathematical reasoning (OmniMath), software
  engineering (SWE-Bench), code implementation (LiveCodeBench),
  knowledge work (GDPVal). Plus **LoCoMo** for long-term dialogue
  memory, reported separately because it stresses memory consistency
  rather than tool-use skill execution.
- Seven baselines in three families: memory-based (EverOS, Memento,
  MemSkill), trajectory-to-skill (EvoSkill, OpenSpace,
  SkillFlow-Evolve), and a vanilla no-memory agent.
- Metric is Pass@1 (macro-averaged per-task success) with cost reported
  as turns or characters. The paper explicitly warns that lower cost may
  reflect premature termination, so cost must be read jointly with
  Pass@1 — an unusually careful framing.

## Results

Pass@1 on EvoAgentBench (best non-MSCE baseline in parentheses):

| Domain | MSCE | Δ vs best baseline | Cost |
|---|---|---|---|
| Information retrieval | 26.15 | +4.61 (EverOS 21.54) | 8.7 vs 14.0 ↓ |
| Mathematical reasoning | 47.00 | +4.00 (EverOS 43.00) | 1140.9 vs 1745.3 ↓ |
| Software engineering | 53.85 | **+15.39** (38.46) | 40.8 vs 37.3 ↑ |
| Code implementation | 61.54 | tie (EvoSkill 61.54) | **2.0 vs 3.9** ↓ |
| Knowledge work | 53.45 | +5.17 (EverOS 48.28) | 15.0 vs 19.1 ↓ |

- Gains are not bought with inference budget: cost falls on four of five
  domains. The exception is SE, where +15.4 Pass@1 costs +3.5 turns —
  attributed to the extra navigation/editing/testing that *successful*
  SE solutions actually require.
- LoCoMo is a much narrower win: overall judge 61.23 / F1 49.89 vs
  SkillFlow-Evolve's 59.22 / 48.71 (+2.01 / +1.18). Best on single-hop,
  multi-hop, and temporal; second on open-domain.
- **Cross-domain transfer is positive on all six pairs tested**, +2.56
  to +5.13 Pass@1 — skills and memory evolved on one domain help on
  another rather than interfering.
- **Ablations (Pass@1 on IR / Math / SE / Code / KW):**

| Variant | IR | Math | SE | Code | KW |
|---|---|---|---|---|---|
| Full MSCE | 26.15 | 47.00 | 53.85 | 61.54 | 53.45 |
| Flat memory | **10.77** | **31.00** | **34.62** | 56.41 | **37.93** |
| w/o skill crystallization | 20.00 | 39.00 | 42.31 | 53.85 | 44.83 |
| w/o reflection weighting | 21.54 | 40.00 | 46.15 | 56.41 | 46.55 |
| w/o L3 | 23.08 | 43.00 | 50.00 | 58.97 | 48.28 |
| w/o value calibration | 24.62 | 44.00 | 50.00 | 58.97 | 50.00 |

  Flat memory is catastrophic — it drops below several baselines and
  costs more than any other ablation. Skill crystallization is the
  second-largest contributor; value calibration the smallest.

## Critique / open questions

- **The ablation undercuts the title.** "From Memory to Skills" frames
  skill crystallization as the contribution, but flattening the memory
  hierarchy costs far more than removing skills entirely (IR 26.15 →
  10.77 vs → 20.00). The dominant effect is the governed L1/L2/L3
  substrate; skill promotion is a real but secondary increment. Read as
  evidence for hierarchical memory first, skills second.
- LoCoMo gains (+2.01 judge) are within the range where LLM-judge noise
  matters, and no variance or repeat-run statistics are reported
  anywhere — no error bars on any table. With single runs on
  small per-domain sets (SE and IR appear to be tens of tasks), several
  of the smaller deltas are not clearly outside noise. The SE result is
  large enough to survive this concern; the 4–5 point ones may not be.
- The evaluation stack is unusually self-referential: an
  agent-self-evolution benchmark, judged partly by LLM, run on a
  commercial agent runtime, by a team whose product is a memory OS. Not
  disqualifying — the baselines are real and the code is public — but
  the comparison is on the authors' home turf.
- "Positive estimated gain" as a promotion criterion is the crux and is
  where the paper is thinnest in the main text; how gain is estimated
  before a skill has been reused determines whether the gate is
  meaningful or self-fulfilling.
- **No retirement evidence.** The abstract promises "lifecycle
  maintenance" and reliability estimates, but the experiments measure
  accumulation and transfer, not what happens when a skill goes stale or
  when the library grows large enough to interfere with retrieval —
  which is the failure mode [[concepts/skill-library-lifecycle]] cares
  most about.

## Trust signals

- **Credibility:** 4 — MemTensor with USTC, HK PolyU, Fuzhou, and XJTU;
  code released under the established MemOS repository. Seven baselines
  across three families, two benchmarks, five-domain coverage, a
  component-level ablation, and a cross-domain transfer study. Held
  below 5 by the absence of peer review, the absence of any variance
  reporting, and a benchmark/runtime choice adjacent to the authors' own
  product.

## Follow-up

- **Relevance:** 4 — strengthens [[concepts/skill-library-lifecycle]]
  with the mechanism it has been missing on the *entry* side. Existing
  sources cover retirement (SLIM's leave-one-skill-out marginal
  contribution) and curation; this supplies a principled **admission
  gate**: a candidate procedure must retain evidence, show positive
  estimated gain, and be internally consistent across trigger,
  procedure, and boundary. Cheaper than retiring a bad skill later.
- **Skills that keep their evidence anchors** is
  [[concepts/citation-anchoring]] applied to procedural memory, and the
  design choice worth stealing: the promoted skill points back at the L1
  traces that justified it, so it can be re-verified or revised rather
  than trusted blindly. Our own skill files assert procedures with no
  link to what established them — the same gap, one level up.
- The L1/L2/L3 split (grounded evidence → induced procedure →
  declarative environment model) is a cleaner articulation of
  [[concepts/multi-granularity-memory]]'s ladder than the
  turn/summary/keyword framings already in the concept, because the
  grains differ in *kind* rather than in compression level — and the
  flat-memory ablation is the strongest quantitative argument in the
  cluster for why the ladder is load-bearing (IR 26.15 → 10.77).
- Open thread this does not close: whether promotion gates prevent the
  retrieval-interference failure that shows up as libraries grow. MSCE
  measures accumulation and transfer over a bounded run; candidate #14
  from today's digest (HDSO, 2606.22330) and the SkillBrew/SkillPyramid
  consolidation line address the same question from the pruning side.
  Worth ingesting one of those next to see whether admission gating and
  consolidation are substitutes or complements.
