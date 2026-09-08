---
kind: paper
title: "Repo-To-Skill: Distilling GitHub Repositories Into AI4AI Skills"
authors: ["Jianlyu Chen", "Yuyang Hu", "Hongjin Qian", "Jiawei Liu", "Wenqing Wei", "Xiaolong Chen", "Defu Lian", "Zhicheng Dou", "Chaozhuo Li", "Qiwei Ye", "Zheng Liu"]
institutions: ["Beijing Academy of Artificial Intelligence", "University of Science and Technology of China", "Renmin University of China", "Hong Kong Polytechnic University"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.02749"
code_url: null
citations: null
source: "raw/papers/chen2026repo.pdf"
added: "2026-09-08"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/compression-as-generalization-test]]"
tags: ["skills", "knowledge-organization", "ml-research-agents", "mle-bench", "paperbench", "distillation", "capability-layer"]
---

# Repo-To-Skill: Distilling GitHub Repositories Into AI4AI Skills

## TL;DR

Names the missing layer in ML-research agents **operational knowledge** —
"the know-how that separates knowing a method from making it work" — and
argues it already exists in repositories but in a form too large to load.
DisCo distils it into verified skills, yielding a **5,000+ skill library
from 1,000 ML repositories**, and with backbone, harness and execution
budget **held fixed** scores **+134.3% on MLE-bench** and **+34.4% on
PaperBench**.

## Claims

- Agent architecture today is backbone + harness (planning, execution,
  memory, verification), and this **leaves domain-specific know-how outside
  the agent entirely**. Operational knowledge is a third layer, not a
  property of either.
- The knowledge is not missing from the field — it is in repos and papers,
  written for human readers and too bulky to load during a task. The problem
  is **distillation and retrieval, not acquisition**.
- Distillation has two complementary modes: **task-agnostic** (condense the
  ecosystem's widely-used repos ahead of time) and **task-oriented**
  (produce the skills a concrete task calls for).
- Gains come from **adding distilled operating context**, not from a better
  model or more compute — the fixed-budget comparison is the whole argument.

## Methods

- DisCo: a skill-powered research agent that both creates and uses skills.
- Task-agnostic distillation across the open ecosystem → the **AREX-Skill
  Library**: 5,000+ *verified* skills from 1,000 widely-used ML
  repositories, organised into **20 areas and 178 capability families**.
- Evaluation holds **GPT-5.5 backbone, research harness, and downstream
  execution budget fixed**, varying only skill availability.

## Results

- **MLE-bench +134.3%**, **PaperBench +34.4%**, FrontierCS +9.2%,
  PassNet +14.0% over the identical agent without skills.
- The spread across benchmarks is itself informative: the largest gain is on
  MLE-bench (execution-heavy, where operational know-how binds most) and the
  smallest on FrontierCS.

## Critique / open questions

- "Verified" is doing heavy lifting for 5,000 skills and the verification
  procedure's strength is the paper's central unstated variable. At that
  scale verification is necessarily automated, so the library's quality is
  bounded by an unreported oracle.
- No library or code link in the paper despite the artifact being the
  contribution — this is the biggest gap.
- +134.3% is a *relative* gain; MLE-bench absolute baselines are low enough
  that large relative movements are easier than the number suggests.
- Distilling 1,000 repos into a shared library is precisely the
  substrate [[literature/papers/paglieri2026case]] shows can carry a
  defect to every consumer. Nothing here audits for that.

## Trust signals

- **Credibility:** 4 — arXiv preprint, not peer reviewed, no citations yet,
  but a strong multi-institution group led by BAAI with USTC / RUC / HK
  PolyU, and Zheng Liu as senior author. The fixed-backbone /
  fixed-harness / fixed-budget ablation is the right experimental design for
  the claim being made. Held off 5 by the absent artifact — for a paper
  whose contribution *is* a library, non-release matters.

## Follow-up

- **Relevance:** 5 — the **most on-mission item in the 09-07 backlog**:
  ML-research agents evaluated on MLE-bench and PaperBench, which
  `CLAUDE.md` names as this project's dominant focus.
- **This is the direct counter-evidence to
  [[literature/papers/kassis2026scientific]]'s honest null.** That paper
  shipped 163 curated skills with *no task-level evaluation and no host
  selection rate*; this one ships 5,000 with a fixed-budget ablation and
  large gains. The pair now brackets
  [[concepts/skill-library-lifecycle]]: a curated library can be
  load-bearing, but only the one that measured it can say so. That is
  exactly what [[concepts/compression-as-generalization-test]] demands, and
  it is the first time the concept has both a positive and a null instance
  on the same question.
- Scale changes the [[concepts/shared-skill-namespace]] problem
  qualitatively: 163 units is a namespace a human can hold, 5,000 organised
  into 20 areas and 178 families is not. The **178 capability families are
  the retrieval index** — the analogue of this project's `mocs/` layer, and
  independent evidence that a flat concept list stops working somewhere
  between those two numbers.
- Reads as the ingest direction this project's own `/fetch-paper` repo path
  already takes, at three orders of magnitude more volume.
