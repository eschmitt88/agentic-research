---
kind: paper
title: "Toward Autonomous Long-Horizon Engineering for ML Research"
authors:
  - Guoxin Chen
  - Jie Chen
  - Lei Chen
  - Jiale Zhao
  - Fanzhe Meng
  - Wayne Xin Zhao
  - Ruihua Song
  - Cheng Chen
  - Ji-Rong Wen
  - Kai Jia
institutions: ["Renmin University of China", "AweAI"]
year: 2026
venue: "arXiv:2604.13018 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.13018"
code_url:
citations:
source: "raw/papers/chen2026toward.pdf"
added: "2026-04-26"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/file-as-bus]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/structured-world-model]]"
tags:
  - long-horizon-agent
  - ml-research-engineering
  - paperbench
  - mle-bench
  - file-as-bus
  - hierarchical-orchestration
  - durable-state
---

# Toward Autonomous Long-Horizon Engineering for ML Research

## TL;DR

AiScientist is a multi-tier agent system for autonomous long-horizon
ML research engineering, built on the principle of *thin control over
thick state*: a top-level Orchestrator routes work to specialist agents
through concise summaries while project state lives in a permission-
scoped shared workspace ("File-as-Bus"). It improves PaperBench by
+10.54 average points over the best matched baseline and reaches
81.82% Any Medal on MLE-Bench Lite; ablating File-as-Bus alone costs
6.41 / 31.82 points respectively.

## Claims

- Long-horizon ML research engineering is primarily a *systems*
  problem of coordinating specialized work over durable project
  state, not a local-reasoning problem.
- Strong long-horizon performance requires both **structured
  orchestration** and **durable state continuity**; either alone is
  insufficient.
- Treating the workspace itself (paper_analysis/, submission/,
  agent/) as the system of record outperforms compressing state into
  conversational handoffs.
- Hierarchical orchestration contributes materially beyond just
  "more interaction" — IterativeAgent has more interaction than
  BasicAgent yet still trails AiScientist-without-File-as-Bus.

## Methods

- **Three tiers**: Tier-0 Orchestrator (stage-level decisions,
  concise summaries, workspace map); Tier-1 specialists (Paper
  Comprehension, Prioritization, Implementation, Experimentation,
  Generic Helper); Tier-2 subagents spawned on demand (e.g. Plan,
  Explore, EnvSetup, Resource Download).
- **Agent-as-Tool**: each Tier-1 specialist is exposed to the
  Orchestrator as a callable tool alongside Bash/Python/WebFetch,
  so delegation is selective rather than mandatory.
- **File-as-Bus**: shared workspace organized into three role-
  aligned regions — `paper_analysis/`, `submission/`, `agent/`
  (with append-only `impl_log.md`, `exp_log.md`). Specialists
  re-ground from these artifacts on each invocation rather than
  inheriting full context.
- **Permission scoping**: each region has read-only / read+limited
  write / read+full write tiers per agent role.
- **Progressive disclosure**: agents start from a workspace map and
  read task-relevant artifacts on demand; private specialist context
  is re-initialized per invocation.
- Evaluated on PaperBench (full) and MLE-Bench Lite under
  Gemini-3-Flash and GLM-5 backbones.

## Results

- **PaperBench**: +10.54 average points over the best matched
  baseline (BasicAgent or IterativeAgent), per-task gains visible
  on every task in Table 1 (e.g. bbox +18.36 under Gemini-3-Flash,
  bridging-data-gaps +13.96 under GLM-5).
- **MLE-Bench Lite**: 81.82% Any Medal under both Gemini-3-Flash
  and GLM-5; +4.55 / +18.18 points over the best matched baseline.
  Beats every official leaderboard row in the paper's reference
  set (highest is 75.76%).
- **File-as-Bus ablation**: removing the protocol drops PaperBench
  by 6.41 points and MLE-Bench Lite Any Medal% by 31.82 points;
  the impact is larger for later-round refinement than for getting
  to a minimally competitive starting point.
- **vs. simpler organizations**: even File-as-Bus-removed AiScientist
  beats BasicAgent / AIDE by ~5–10 points, suggesting hierarchy
  contributes independently of state continuity.

## Critique / open questions

- The role split (Paper Comprehension, Prioritization, Implementation,
  Experimentation) is well-suited to PaperBench's paper-replication
  shape. Whether the same split transfers to other long-horizon
  research-engineering settings (open-ended research, ML competitions
  without a target paper) is not directly tested.
- Permission scoping is asserted as load-bearing but not separately
  ablated; its contribution beyond the File-as-Bus protocol itself
  is unclear from the paper.
- "Workspace map" is mentioned as the Orchestrator's substitute for
  full-state context, but its update cadence and representation are
  not documented in detail — likely a key implementation question
  for replication.
- File-as-Bus and citation-anchoring (per Kosmos) are likely
  complementary; this paper does not enforce citation discipline on
  workspace writes.

## Trust signals

- **Credibility:** 3 — Renmin University of China (Gaoling School of
  AI) + AweAI; arXiv preprint, not yet peer-reviewed; page-1 GitHub
  link advertised but no concrete repo URL in the text. Strong
  benchmark results (PaperBench, MLE-Bench Lite) from a reputable
  academic group, but reproducibility unverified.

## Follow-up

- This is direct prior art for any downstream project that wants a
  durable-artifact substrate; consider whether the workspace layout
  (`paper_analysis/`, `submission/`, `agent/`) maps onto our existing
  experiment folder shape (`experiments/<slug>/{config.yaml, log.md,
  notes.qmd, results/}`) — they appear close.
- The +6.41 / +31.82 ablation gap between "with vs. without File-as-Bus"
  is the cleanest published evidence we have that durable state continuity
  matters; cite this when justifying the meta-project's
  artifacts-over-conversation defaults.
- Compare File-as-Bus to Kosmos's structured world model: both are
  shared state, but File-as-Bus is filesystem-as-bus (heterogeneous
  artifacts, role-aligned regions) while structured-world-model is a
  schema-indexed object. Worth a concept-level distinction.
