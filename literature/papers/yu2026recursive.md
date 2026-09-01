---
kind: paper
title: "Recursive Experiential–Working Memory Evolution for Long-Horizon Agent Harnesses"
authors: ["Zhaochen Yu", "Yingcheng Wu", "Zhenfei Yin", "Kaiyuan Chen", "Zhe Zhao", "Mengdi Wang", "Shuicheng Yan", "Ling Yang"]
institutions: ["National University of Singapore", "Stanford University", "University of Oxford", "Princeton University"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.24876"
code_url: "https://github.com/Gen-Verse/Recuris"
citations: null
source: "raw/papers/yu2026recursive.pdf"
added: "2026-09-01"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/structured-world-model]]"
tags: ["memory", "agent-architecture", "self-improvement", "long-horizon", "skills"]
---

# Recursive Experiential–Working Memory Evolution for Long-Horizon Agent Harnesses (Recuris)

## TL;DR

Long-horizon agents fail because a growing history **obscures the current
task state**, which in turn misaligns skill selection. Recuris separates
**Working Memory** (tracks progress, grounds skill selection in current
need rather than full history) from **Experiential Memory** (the skill
store), and adds a Meta-Agent that attributes each failure to a *specific
memory component* and patches only that one. The coupling is what makes
failures localizable.

## Claims

- State-grounded memory use is a **prerequisite** for recursive
  self-improvement in long-horizon tasks — without a tracked task state,
  skill invocation is misaligned and failures cannot be attributed.
- Coupling WM to EM turns execution into **structured evidence** that
  localizes failures to a component, rather than producing an
  undifferentiated trace.
- Repair should be **scoped per edit, not per round**: a candidate patch
  revises every component the diagnosis implicates and copies the rest
  unchanged, so each edit is tied to a diagnosed failure and confined to
  the component that failure was attributed to.
- Patch admission is **validation-gated**, forming a bounded recursive loop
  rather than open-ended self-modification.
- **The critical memory component differs across domains** — there is no
  single memory mechanism to optimize.

## Methods

- Four components form the evolving memory-control layer and define the
  patch space: Working Memory, Experiential/Skill Memory, and the
  selection and state-update mechanisms.
- A **fixed** Meta-Agent (not itself evolved) reads structured execution
  traces, diagnoses which memory component is implicated, and emits a
  scoped patch.
- **Validation-gated patch admission**: patches are admitted only if they
  survive validation, bounding drift.
- Four long-horizon benchmarks (τ²-Bench / τ²-Retail, SkillFlow, and two
  others) covering tool-use dialogue and lifelong skill learning, across
  **ten models** from 3B open-weight to frontier.
- Ablations remove each harness mechanism in turn and re-run both domains;
  a separate ablation removes one component from a fixed Skill Memory.

## Results

- Improves task success in **35 of 37 completed model–benchmark pairs**.
- τ²-Bench: **+17.8** points for GPT-5.6 Sol; **+15.6** for Claude Opus 5,
  taking Opus 5 to **87.9%**.
- SkillFlow: **+16.6 / +13.5** points for Qwen3.6-27B / 35B.
- **The advantage widens with horizon length**, reaching **+32.2 points on
  the longest tasks** — the gain is specifically a long-horizon effect, not
  a uniform lift.
- Common long-horizon failures fall by **up to 80%**.
- Component-level failure localization is substantially more accurate than
  the alternatives compared.
- Evolved Skill Memory **transfers across tasks and across models**.
- Ablation: the critical mechanism **differs between domains** — removing
  the same component helps in one domain and hurts in another.

## Critique / open questions

- The gains are largest exactly where measurement is hardest (longest
  horizons, most expensive runs), so the +32.2 headline rests on the
  smallest and costliest slice of the evaluation.
- The Meta-Agent is fixed and hand-designed. The system self-improves its
  *memory*, not its *improver* — a reasonable and honest scoping, but it
  means "recursive self-improvement" is claimed for a bounded inner loop
  only.
- "35 of 37 **completed** pairs" implies three of forty were not completed;
  what failed to complete, and why, is not surfaced in the abstract.
- Domain-dependence of the critical component is reported as a finding, but
  it also undercuts transferability: a practitioner cannot know in advance
  which memory component to invest in.
- Benchmarks are tool-use dialogue and skill learning, not research
  workflows. The transfer to ML-research agents is by analogy.

## Trust signals

- **Credibility:** 4 — four strong institutions (NUS, Stanford, Oxford,
  Princeton) including Mengdi Wang, Shuicheng Yan, and Ling Yang; code
  released at `Gen-Verse/Recuris`; ten models × four benchmarks with
  per-mechanism ablations and cross-task/cross-model transfer studies; a
  negative-ish finding (domain-dependent critical component) reported
  rather than smoothed. Not peer reviewed.

## Follow-up

- **Relevance:** 5 — this fills the gap
  [[concepts/multi-granularity-memory]] explicitly leaves open: the
  *transformation between tiers*. Both that concept and
  [[concepts/agent-native-memory]] describe tiers and their contents; the
  WM↔EM coupling here is a mechanism for how one feeds the other, with
  ablation evidence that the coupling — not either tier alone — is what
  produces the gain.
- **Failure localization is the transferable idea, independent of the
  training.** "Attribute the failure to a memory component, patch only that
  component" is a discipline this project can adopt in prose: when a
  session goes wrong, the question becomes *which* layer failed —
  `raw/` capture, the literature note, the concept, or the skill — rather
  than a general retrospective. `/lint` currently checks structural
  integrity but attributes nothing.
- Validation-gated patch admission is a second independent attestation
  (with [[literature/papers/tang2026wikiskill]], same curate pass) of the
  same rule: **an autonomous edit to the memory layer must clear a gate
  before it persists**. That converges with
  [[concepts/verified-memory-writes]] from the self-improvement side rather
  than the security side, and the pair of them is a stronger case than
  either alone.
- The Working Memory as an explicit, maintained task-state object relates
  to [[concepts/structured-world-model]] and to
  [[concepts/context-proprioception]]: the agent reads its state from a
  maintained artifact instead of re-deriving it from a growing transcript.
  Compare [[literature/papers/badhe2026skill]] (same curate pass), which
  reaches the same "replace append-only history with explicit mutable
  state" conclusion at the runtime level.
