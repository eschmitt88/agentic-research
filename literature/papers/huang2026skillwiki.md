---
kind: paper
title: "SkillWiki: A Living Knowledge Infrastructure for Agent Skills"
authors: ["Dingcheng Huang", "Yuda Ding", "Bingshuo Liu", "Qingbin Liu", "Xi Chen", "Jiang Bian", "Hongliang Sun", "Zhiying Tu", "Dianhui Chu", "Xiaoyan Yu", "Dianbo Sui"]
institutions: ["Harbin Institute of Technology", "Tencent", "Nanyang Technological University"]
year: 2026
venue: arXiv (cs.CL, demo-style systems paper)
peer_reviewed: false
url: https://arxiv.org/abs/2606.16523
code_url: https://github.com/Huangdingcheng/SkillWiki
citations: null
source: "raw/papers/huang2026skillwiki.pdf"
added: "2026-07-14"
relevance: 3
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
tags: [skills, infrastructure, governance, provenance, versioning, lifecycle, registry]
---

# SkillWiki: A Living Knowledge Infrastructure for Agent Skills

## TL;DR

Argues that agent skills lack what Wikipedia gives knowledge and GitHub
gives software — a shared infrastructure for large-scale production,
governance, and evolution — and demos SkillWiki: skills as *governed
capability assets* with an eight-state lifecycle (Raw Experience →
Candidate → Draft → Verified → Released → Degraded → Deprecated →
Archived), a provenance graph linking every skill to its originating
evidence, git-style reviewed changes, and health-monitoring-driven
maintenance proposals.

## Claims

- The bottleneck in skill ecosystems is no longer acquisition,
  retrieval, or evolution mechanisms (all well-studied) but the absence
  of *unified infrastructure* for organizing, tracing, governing, and
  evolving an ever-growing skill collection — a systems problem, not an
  algorithms problem.
- Knowledge and skills should be decoupled layers: original knowledge
  materials (trajectories, docs, API specs, scripts, historical skills)
  are preserved as long-term evidence assets, and skills are
  continuously *derived* from them, each skill explicitly linked back
  to its sources.
- Skill changes should never be direct overwrites: creation, repair,
  decomposition, composition, and version updates are candidate changes
  processed through snapshots, structured diffs, reviews, and releases
  (git-style governance), keeping evolution traceable, reviewable, and
  reversible.
- Skill *health* is measurable from runtime signals (execution/success
  counts, recurring-failure clusters, reflection memories) and should
  automatically generate maintenance proposals routed through the same
  governance workflow.

## Methods

- Unified skill schema: identity, classification (atomic / functional /
  strategic), lifecycle state, interface spec, implementation,
  evaluation contract, provenance, graph relations, runtime metrics.
- Skill Provenance Graph: heterogeneous graph over knowledge sources,
  skills, tools, executions, validations, versions, and inter-skill
  dependencies.
- Self-management agents per lifecycle stage: Builder (extract +
  construct), Auditor (audit + verify, with deterministic verifiers —
  schema checks, harness validation gates for S2→S3), Harness/Runtime
  (execute + monitor), Maintainer (repair + evolve), Librarian
  (deprecate + archive). Human decisions take precedence over
  autonomous governance actions.
- Web UI + CLI exposing the same infrastructure for automation.
- Eval: 125 knowledge artifacts (25 each of trajectories, documents,
  API specs, scripts, historical skills; parsing via DeepSeek-V4-Flash)
  → 99 governed skill candidates (79%); plus one full-chain case study
  traversing all S0–S7 states including degradation, repair, version
  update, deprecation, and archival.

## Results

- Knowledge-to-skill conversion works across all five source types
  (highest yield from trajectories and API specs at 24/25, lowest from
  historical skills at 16/25).
- The case study demonstrates the complete closed loop: execution
  failures cluster → repair proposal → reviewed version update
  (1.0.0 → 1.1.0) → eventual deprecation and archival with provenance
  retained.
- No downstream-task evaluation: the paper measures whether the
  *workflow* functions, not whether governed skills make agents better.

## Critique / open questions

- Demo-scale evidence only: 125 artifacts, one lifecycle case study;
  behavior at the tens-of-thousands scale the motivation invokes
  (SkillRet's 17.8k crawl) is explicitly untested.
- Effectiveness is measured as infrastructure throughput (candidates
  produced, states traversed), never as agent capability improvement —
  the governance layer's cost/benefit on real task suites is unknown.
- The self-management agents are themselves LLM agents governing other
  agents' assets; the paper doesn't address who audits the auditor
  (cf. the deterministic-core argument in the permission-gate cluster —
  though its S2→S3 verification gate is notably deterministic).
- Long-term stability of a continuously self-evolving skill ecosystem
  is named as open by the authors themselves.

## Trust signals

- **Credibility:** 3 — HIT + Tencent + NTU team, code and live demo
  released, honest limitations; but a demo-style preprint whose
  evaluation is feasibility, not effectiveness.

## Follow-up

- **Relevance:** 3 — the first *infrastructure-level* attestation in
  the skill cluster: [[concepts/skill-library-lifecycle]] covers the
  write-policy operations (insert/update/retire and their learned
  variants) per library; SkillWiki names the layer above — lifecycle
  *states* as governance objects, provenance as a graph, changes as
  reviewed releases. Useful prior art and vocabulary, but demo-grade
  evidence doesn't shift the concept's architecture.
- Speaks to [[concepts/shared-skill-namespace]]'s open registry
  question: SkillWiki is a concrete proposal for what a PyPI-style
  skill registry would need (versioning, lineage, health, governed
  contribution) beyond the current curated-link-list reality of
  agentskills.io.
- The S0–S7 state machine maps neatly onto this project's own note
  lifecycle (candidate → ingested → concept → MoC → retired); the
  Degraded state — detected from runtime failure signals rather than
  age — matches SLIM's evidence-based retirement
  ([[literature/papers/shen2026dynamic]]).
- Watch whether the demo grows a real evaluation; revisit if a
  downstream benchmark lands.
