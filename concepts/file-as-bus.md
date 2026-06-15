---
kind: concept
name: "file-as-bus"
status: experimental
added: "2026-04-26"
source_papers:
  - chen2026toward
sources:
  - "[[literature/papers/chen2026toward]]"
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/xin2026eurekagent]]"
used_by: []
related_concepts:
  - "[[concepts/structured-world-model]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/agent-native-memory]]"
related_experiments: []
tags: [agent-architecture, durable-state, coordination, workspace, long-horizon]
---

# file-as-bus

## Definition

A coordination protocol where the shared filesystem workspace —
not the conversational handoff and not an in-memory schema — is the
authoritative system of record across agent invocations. Specialists
write durable, role-aligned artifacts (paper analyses, code,
configs, append-only logs, experiment outputs) into permission-
scoped regions and re-ground from them on each call rather than
inheriting full conversational context.

## Why it matters

AiScientist ([[literature/papers/chen2026toward]]) reports that
removing File-as-Bus from an otherwise identical system drops
PaperBench score by 6.41 points and MLE-Bench Lite Any Medal% by
31.82 points. The ablation is among the cleanest published evidence
that *durable state continuity* — not just "more interaction" or
"better prompts" — is what carries an agent through hours- to
days-long ML research engineering tasks.

The pattern's core trade is: pay a per-call cost (read the workspace
map, fetch task-relevant artifacts) to dodge the alternative cost
(carry the whole project history in every active context). At
12-hour horizons the trade is strongly positive; the ablation
suggests it widens for later-round refinement, where conversational
state has accumulated enough drift to corrupt decisions.

## Implementation guidance

1. **Organize the workspace into role-aligned regions.** AiScientist
   uses three: `paper_analysis/` (structured paper understanding),
   `submission/` (the runnable artifact), `agent/` (planning and
   logs). For our experiment shape, the equivalent is the
   per-experiment folder: `config.yaml`, `notes.qmd`, `results/`,
   `log.md`, `metrics.json`. The point is that each region has a
   single owning role and a documented purpose.

2. **Permission-scope writes per role.** Read-only / read+limited
   write / read+full write tiers prevent specialists from
   clobbering each other's regions. In our setup, this maps to:
   `/ingest` writes to `literature/` and `concepts/` but not
   `experiments/`; `/implement` writes to its experiment folder
   but not `concepts/`. The boundaries are already there;
   File-as-Bus formalizes them.

3. **Append-only logs over rewritable ones for handoff state.**
   AiScientist's `impl_log.md` and `exp_log.md` are append-only —
   later rounds read prior failures rather than overwriting them.
   Our `_meta/log.md` and per-experiment `log.md` follow the same
   pattern; this is not coincidental.

4. **Re-ground, do not inherit.** A specialist's private context
   is re-initialized at each invocation. Continuity is carried by
   workspace artifacts plus a *workspace map* — a small, stable
   index that the Orchestrator passes by reference. The
   `_meta/index.md` + `MEMORY.md` of a project is the analogous
   map; keep it small enough to fit in any specialist's context
   on entry.

5. **The workspace is the system of record.** If a fact is not
   written to the workspace, it does not exist for the next
   round. This is the discipline `/wrap` enforces and the
   SessionEnd hook backstops.

## Connections

- Related to but distinct from
  [[concepts/structured-world-model]]: a structured world model is
  a *schema-indexed object* that sub-agents query by field; File-as-Bus
  is a *workspace-as-coordination-substrate* of heterogeneous
  artifacts. They compose — the world-model file is one artifact
  in the bus.
- Pairs with [[concepts/hierarchical-delegation]]: File-as-Bus is
  the substrate the orchestrator uses to keep its own context thin
  while specialists do thick local work.
- Writes to the bus benefit from
  [[concepts/citation-anchoring]] discipline so downstream agents
  can audit claims back to source artifacts.

## Open questions

- Does the workspace-map representation (the Orchestrator's index of
  the bus) need to be hand-designed, or can it be derived from the
  filesystem tree? AiScientist's paper does not document the format
  in detail.
- File-as-Bus is asserted as load-bearing primarily on long horizons
  (PaperBench tasks run hours per submission). Whether it pays off
  on shorter (~1h) tasks is open.
- Status is `experimental` because while the project's existing
  experiment-folder shape and log discipline already implement most
  of the pattern, no skill yet enforces permission scopes between
  agent roles. Graduate to `active` when a downstream project runs
  an Orchestrator+specialist split with explicit per-role write
  permissions.
