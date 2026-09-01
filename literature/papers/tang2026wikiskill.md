---
kind: paper
title: "WikiSkill: Compiling Agent Experience into Persistent Knowledge for Skill Evolution"
authors: ["Liyan Tang", "Cyrus Rashtchian", "Chun-Sung Ferng", "Andrew Tomkins", "Da-Cheng Juan", "Tu Vu"]
institutions: ["Google Research", "Virginia Tech"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.27454"
code_url: null
citations: null
source: "raw/papers/tang2026wikiskill.pdf"
added: "2026-09-01"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/shared-skill-namespace]]"
tags: ["knowledge-organization", "skills", "memory", "agent-architecture"]
---

# WikiSkill: Compiling Agent Experience into Persistent Knowledge for Skill Evolution

## TL;DR

Skill-evolution systems throw away the *reasoning* that produced a skill
update — insights stay "scattered across optimization histories". WikiSkill
inserts a **persistent wiki layer between raw traces and executable skills**,
so each iteration's understanding survives even when the skill edit it
motivated is rolled back. Ablations confirm the wiki, not the skill loop, is
what carries the gains.

## Claims

- A structured knowledge layer belongs **between** raw experience and
  executable procedure. Skills alone lose the rationale; traces alone are
  unconsolidated.
- Skill updates should be **gated and reversible**, but the knowledge
  extracted while making them should be **durable**. The asymmetry is the
  design's core: skills roll back, the wiki does not.
- **Skill discovery and skill execution are distinct capabilities** — a
  model can benefit from a skill it could not itself have discovered.
- Skill evolution **complements** model scaling rather than substituting
  for it: larger models benefit more from evolved skills.
- Explicitly positioned as an implementation of Karpathy (2026)'s "LLM
  Wiki" argument — compile experience into persistent, reusable knowledge.

## Methods

Three layers over the agent workspace:

- **Raw Layer** — immutable execution traces.
- **Wiki Layer** — structured, consolidated knowledge.
- **Skill Layer** — evolving procedural knowledge; a skill is a *modular,
  filesystem-based directory* of instructions, scripts, and resources.

Four components in a continual loop: an **Inference Agent** (runs rollouts
under current skills), a **Wiki Maintainer** (consolidates traces into the
wiki), a **Skill Proposer** (proposes skill updates from wiki + traces), and
a **Gating and Rollback** mechanism (retains only updates that improve
validation performance). Tasks are split into disjoint train / validation /
test sets; gating is measured on validation.

Five benchmarks (LiveMathematicanBench, SealQA, spreadsheet manipulation,
ALFWorld, and others) across five models spanning the Qwen and Gemini
families.

## Results

- Within the Qwen family, average performance improves by **12.3% / 17.5% /
  23.9%** for 4B / 9B / 27B — gains *increase* with scale.
- **Skills substitute for scale**: Qwen-3.5-9B + WikiSkill beats
  Qwen-3.6-27B with no skills (**47.4% vs 39.4%**).
- **Cross-model transfer, sometimes beating self-evolution**: on ALFWorld,
  Qwen-3.5-9B reaches **70.2%** using a skill evolved by Qwen-3.6-27B,
  versus **63.4%** using its own.
- Outperforms prior skill-evolution methods (EvoSkill, SkillOpt) and
  no-skill baselines in most model-benchmark settings.
- **Ablation: the persistent wiki is critical.** Removing knowledge
  accumulation degrades skill evolution — the wiki is doing the work, not
  the skill-editing loop alone.

## Critique / open questions

- No released code, which for a Google Research paper is a real gap and the
  main thing holding credibility below 5. The wiki's content and schema —
  the part most worth copying — are therefore not inspectable.
- "Improves in *most* model-benchmark settings" is doing quiet work; the
  failures are not foregrounded in the abstract.
- The wiki is maintained by an LLM ("Wiki Maintainer") with no described
  verification step. This is precisely the unverified-write surface that
  [[concepts/verified-memory-writes]] argues against, and
  [[literature/papers/wu2026evomal]] shows is exploitable when the artifact
  is executable. WikiSkill gates the *skill* update but not the *wiki*
  write.
- Gains increasing with model scale is reported as encouraging, but it also
  means the benefit for the small-model regime — where scaffolding matters
  most economically — is the weakest case.

## Trust signals

- **Credibility:** 4 — Google Research with Andrew Tomkins and Da-Cheng
  Juan, five benchmarks × five models, disjoint train/val/test splits, a
  gating mechanism measured on held-out validation, cross-family transfer
  experiments, and an ablation that isolates the paper's central claim.
  Not peer reviewed and no code released.

## Follow-up

- **Relevance:** 5 — this is the **third independent construction** of
  [[concepts/llm-wiki-pattern]], after the Karpathy-adjacent post and
  huang2026skillwiki, and it is the first to *cite the same origin
  explicitly* while adding a controlled ablation showing the wiki layer is
  load-bearing. The pattern is now attested by construction, not just
  advocacy.
- **The three-layer structure is this project's structure.** Raw Layer
  (immutable traces) / Wiki Layer (structured knowledge) / Skill Layer
  (executable) maps almost exactly onto `raw/` (immutable per
  `.claude/rules/data.md`) / `literature/` + `concepts/` / `~/.claude/skills/`.
  This is the closest external validation of the project's own layout that
  has been ingested.
- **The rollback asymmetry is the importable mechanism.** This project
  currently has no equivalent: when a skill edit is reverted, the reasoning
  that motivated it is lost with it. WikiSkill's rule — revert the
  artifact, keep the knowledge — is a concrete addition to
  [[concepts/skill-library-lifecycle]], which today describes promotion but
  not what survives a demotion. This is a strong `/elevate` candidate.
- "Skill discovery and skill execution are distinct capabilities", plus
  skills transferring across model families and beating self-evolved
  skills, is direct evidence for [[concepts/shared-skill-namespace]]'s
  benefit case — and an interesting counterweight to
  [[literature/papers/wu2026evomal]], which names the same namespace as the
  contamination channel. Both are true; the namespace is a high-value,
  high-risk surface.
- Extends [[concepts/agent-native-memory]]: the wiki is written *for* the
  agent's own future consumption, not for a human reader, and its value is
  measured by downstream task success rather than by human legibility.
