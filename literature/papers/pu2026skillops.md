---
kind: paper
title: "SkillOps: Managing LLM Agent Skill Libraries as Self-Maintaining Software Ecosystems"
authors: ["Hongji Pu", "Xinyuan Song", "Liang Zhao"]
institutions: ["University of Illinois Urbana-Champaign", "Emory University"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2605.13716
code_url: https://github.com/Hik289/SkillOps
citations: null
source: "raw/papers/pu2026skillops.pdf"
added: "2026-06-23"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
tags: [skill-library, curation, lifecycle, technical-debt, typed-contracts, maintenance, alfworld, rule-based]
---

# SkillOps: Managing LLM Agent Skill Libraries as Self-Maintaining Software Ecosystems

## TL;DR

SkillOps is a method-agnostic, drop-in framework that treats an agent's
skill library as a **software ecosystem with technical debt**, maintained
by two cooperating loops: a **task-time loop** (retrieve + stitch skills
into an executable plan for the current task) and a **library-time loop**
(diagnose library health from execution traces, then apply typed
maintenance actions — merge, repair, retire, add_validator, add_adapter).
Each skill is a **typed contract** `(P, O, A, V, F)` and skills are
organized in a **Hierarchical Skill Ecosystem Graph (HSEG)** with typed
dependency / compatibility / redundancy / alternative edges. The
maintenance pass is **rule-based and uses ~zero LLM calls**, so library
maintenance is a low-overhead architectural layer rather than an extra
inference-time burden.

## Claims

- Skill libraries accumulate **skill technical debt**: library-level
  defects (redundant clones, stale skills, missing validators, missing
  artifacts, interface drift, over-specialized skills) that may not break
  a single skill locally but degrade future retrieval, composition, and
  execution. The key issue is **persistence**: task-time repair fixes one
  episode while the underlying library defect remains and re-causes
  failures on later reuse.
- Prior skill-based agents focus almost entirely on **task-time** use
  (retrieve / compose / validate-during-execution) and assume the library
  is healthy; the **library-time** maintenance problem is underexplored.
  SkillOps separates the two loops cleanly — the central interface is a
  pure library transformation `cleaned_lib = run_maintenance(raw_lib)`.
- As a standalone agent on ALFWorld with a 200-skill library, SkillOps
  reaches **79.5% task success**, beating the strongest baseline by
  **+8.8pp** with **zero additional task-time LLM calls**.
- As a plug-in maintenance layer it consistently helps **retrieval-heavy**
  baselines (+0.68 to +2.90pp) but is **method-conditional** — neutral or
  slightly conflicting for LLM-planning and self-repairing baselines that
  already do task-time filtering.

## Methods

- **Skill Contract** `s = (P, O, A, V, F)`: preconditions, executable
  operation, produced (typed) artifacts, validator over artifacts, and
  known failure modes. `V = ∅` is a "validation gap" — no local
  correctness check. Contracts let the planner check
  relevance/applicability/composability/local-verifiability before
  execution.
- **HSEG**: library as `L = (S, R)` with four typed edge relations —
  `dep` (artifact satisfies a downstream precondition), `comp`
  (output type compatible with downstream input type), `red`
  (equivalent interfaces → near-duplicate), `alt` (same goal, different
  operation). Maintenance actions are typed:
  `merge` (collapse a `red` pair), `repair` (rewrite an operation from
  execution feedback), `retire` (drop obsolete / consistently failing
  skills), `add_validator` (insert validator when `V = ∅`),
  `add_adapter` (insert a type-conversion shim when interfaces are
  dep-linked but not comp-compatible), `instantiate` (bind a task-specific
  argument at task time).
- **Task-Time Loop** (Algorithm 1): skill-match via `λ·BM25 + (1−λ)·sem`,
  filter by satisfied preconditions, **dependency-stitch** a plan that
  enforces *both* dep and comp edges (so a transition is accepted only
  when the produced artifact also matches the next skill's expected input
  type), insert validator/adapter nodes, execute with **local repair**
  (substitute an `alt` neighbour or re-invoke `repair`) and log
  unrecoverable failures for later maintenance.
- **Library-Time Loop** (Algorithm 2): mine skill contracts from
  execution logs, score each skill on **five health dimensions** —
  Utility, Redundancy, Compatibility, Failure-Risk, Validation-Gap — into
  an overall health score `H(L)`; if `ΔH` exceeds a threshold, apply the
  typed maintenance actions. **CGPD (ContractGraph-Propagated Diagnosis)**
  is an optional component that propagates risk along `dep` edges
  (`R^{t+1}(s) = (1−α)R_loc(s) + α·max_{parents} R^{t}(s')`, converging by
  Banach fixed point) for preemptive validator insertion on structurally
  sound skills with high upstream risk.
- **Evaluation**: ALFWorld (text household manipulation, 185 tasks/seed, 3
  seeds, strict-order subgoal grader). Library built from 229 curated
  SkillsBench skills; larger libraries (up to 2000) reach scale by adding
  **synthetically degraded variants** covering six technical-debt patterns
  (redundant/stale clones, missing validators, missing artifacts, wrong
  interfaces, over-specialization). Baselines: ReAct, LLM_Skill_Planner,
  Hybrid_Retrieval, GoS_Style, SkillWeaver — all on GPT-4o-mini.

## Results

- **Standalone (200-skill library)**: SkillOps_Full 79.5% SR (±0.00) vs
  LLM_Skill_Planner 70.6%, GoS_Style 61.1%, Hybrid_Retrieval 58.2%,
  SkillWeaver 50.3%, ReAct 12.8% → +8.8pp over the strongest baseline.
- **Drop-in plug-in (200-skill)**: helps retrieval-heavy methods —
  Hybrid +2.90pp, SkillWeaver +2.46pp, BM25-Only +1.00pp, Dense +1.12pp,
  GoS +0.80pp, LLM_SP +0.50pp; ReAct +0.00 (token-budget dominated by
  action history). Effect is smaller where the baseline already filters.
- **Scale sensitivity (200→2000 skills, degradation 15→90%)**: SkillOps
  stays stable (80.5% at the largest scale) and leads the next-best
  baseline by **>31pp**; task-time-only baselines degrade as the noisy
  candidate pool grows.
- **Token cost**: maintenance is ~zero LLM calls; task-time token change
  is mostly neutral/negative (24 of 35 cells decrease) because library
  pruning yields a cleaner top-k candidate set.
- **Ablations**: removing the Task-Time Loop causes the largest drop
  (79.5→15.7%); removing Library-Time Loop drops to 71.9%; both graph
  levels matter; `add_adapter`, `add_validator`, `repair` are the most
  critical maintenance actions, `merge`/`retire`/CGPD smaller.

## Critique / open questions

- **Half-synthetic library, single benchmark.** Scale and degradation are
  driven by *synthetically* degraded skill variants on ALFWorld only; the
  authors concede broader benchmarks and **real long-running agent logs**
  are needed. The headline "self-maintaining ecosystem" claim is tested
  against engineered debt, not organically accrued debt.
- **Relies on structured contracts + gold PDDL-style arguments**, which
  may not exist in real deployments. The contract-mining step from
  execution logs is the load-bearing assumption that the typed `(P,O,A,V,F)`
  structure can be recovered cheaply — that step is asserted more than
  evaluated.
- **CGPD does not improve task success** in the current setup (validator
  fields aren't consumed during plan-time selection) — the paper's most
  novel component is presently inert.
- The rule-based, zero-LLM maintenance is a strength for cost but the
  authors note it can **miss semantic redundancy or complex skill
  conflicts** that need deeper reasoning — the cheap loop has a known
  blind spot.
- Method-conditional gains mean SkillOps is **not strictly additive**: it
  can be neutral or slightly conflict with agents that already self-repair
  at task time (SkillWeaver token increases at some scales).

## Trust signals

- **Credibility:** 3 — arXiv preprint (v1, not peer-reviewed) from a small
  three-author team at UIUC and Emory, with **code released**
  (github.com/Hik289/SkillOps). Evaluation is rigorous in form (Wilson CIs,
  3 seeds, 9 library scales, ablations + trigger-precision) but rests on a
  **single benchmark (ALFWorld)** and a **largely synthetic** degraded
  library, with GPT-4o-mini-only baselines re-implemented by the authors
  (GoS / GraSP are clean reproductions, not author code). Code release
  lifts it above a bare preprint; single-benchmark + synthetic-debt scope
  holds it at 3.

## Follow-up

- **Relevance:** 4 — a strong fresh attestation of
  [[concepts/skill-library-lifecycle]] and arguably the **cleanest extant
  statement of the curation loop**: it factors the lifecycle into an
  explicit **task-time loop** (retrieve + stitch for the current task) and
  a **library-time loop** (diagnose + maintain after execution), and gives
  the insert/update/delete vocabulary concrete typed operations
  (`merge`/`repair`/`retire`/`add_validator`/`add_adapter`). The
  **skill technical debt** framing — library-level defects that persist
  across episodes even when task-time repair patches one run — sharpens the
  "when to delete / when to prune" write-policy question at the heart of
  that concept, and the five-dimensional health diagnosis
  (utility/redundancy/compatibility/failure-risk/validation-gap) is a
  reusable rubric for *deciding* maintenance actions. Complements
  cho2026skillret / ouyang2026skillos / saha2026skilldex / yang2026skillopt
  by naming the loop split explicitly. Held at 4 (not 5) because it
  strengthens and articulates an existing concept rather than seeding a new
  importable one, and its evidence base is one benchmark on synthetic debt.
- Not asserting [[concepts/shared-skill-namespace]]: SkillOps' typed
  contracts are an *internal* HSEG schema for one library's maintenance,
  not a cross-harness portability convention — the namespace concept is
  about file-format/path portability across harnesses, which this paper
  does not address.
- Candidate follow-up: a downstream experiment could test whether the
  task-time/library-time split and the five-dimensional health rubric
  transfer to *organically* accrued skill debt (real agent logs) rather
  than synthetically injected variants — the paper's main unaddressed gap.
