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
  - "[[literature/papers/yu2026knows]]"
  - "[[literature/papers/alzahrani2026persistent]]"
  - "[[literature/papers/ravindran2026portable]]"
  - "[[literature/papers/philippov2026glite]]"
  - "[[literature/papers/ishibashi2026effective]]"
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

[[literature/papers/philippov2026glite]] runs the pattern at
multi-week campaign scale with two refinements worth naming: a
**corrections overlay** (completed task folders are immutable;
downstream fixes are correction files in the fixing task's folder,
applied at read time by aggregators — the original record is never
rewritten) and a **materialized overview** (a committed, regenerated
dashboard is the only human-facing summary; agents read through
aggregator scripts, never by walking the workspace). Both exist
because hand-maintained cross-task summaries drift ("Completed Tasks
(75 of 64)"); the canonical view must be recomputable from the files.

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

## Same model, same budget, different harness — the cleanest instance yet

[[literature/papers/ishibashi2026effective]] is the component-level
attestation this concept has been waiting for, and its control is tight
where it counts: **identical model, identical 40M-token budget**, only the
harness differs. On Circle Packing (n=26), OpenEvolve with `gpt-5.2-codex`
scores 2.54142; Vesper with the same model scores 2.63599 — past the human
best (2.6340) and level with AlphaEvolve (2.6358). The good harness passed
the weak harness's *final* score at roughly one eighth of the budget.

The sharper form of the claim: **the cheap model on the good harness beats
the strong model on the weak one.** Vesper with `gpt-5.1-codex-mini` (2.636)
outperforms OpenEvolve with `gpt-5.2` (2.419), at $42 against $27. If the
harness can invert a model-capability ordering, the harness is not a
multiplier on model quality — it is a comparable term.

The stated design premise is the same one this concept encodes: existing
implementations "use LLMs as stateless code generators through single-shot
API calls, without leveraging the capabilities of coding agents." A
stateful agent that can run and test its own candidate against the evaluator
is a different operator than a text generator, and the harness is what makes
that difference available.

**Caveat that should travel with the citation:** Vesper differs from
OpenEvolve in four ways at once — coding-agent integration, evaluation-hack
detection, Git worktree isolation, and database observation — and only the
last two are ablated. So "the harness matters" is well supported and *which
part* is not. Read alongside philippov2026glite, which makes the same
argument at research-campaign scale with a different mechanism.

## Open questions

- **A naturalistic (non-benchmark) instance of the pattern now exists.**
  Alzahrani ([[literature/papers/alzahrani2026persistent]]) is a
  115-day self-observed case study of one physician-scientist running
  a persistent multi-role agent environment — 502 memory files, 17
  configured agent directories, 57 skill files, scheduled jobs, and a
  governance layer that "became part of the operating environment
  rather than an after-the-fact policy appendix." That is the same
  shape as this concept's implementation guidance (#1 role-aligned
  regions, #3 provenance-bearing durable artifacts, #5 workspace as
  system of record) observed in the wild rather than asserted from an
  architecture description or measured via ablation. It attests the
  *coordination-substrate* half more directly than Knows does — 17
  distinct role directories is closer to File-as-Bus's multi-region,
  multi-role claim than Knows' one-sidecar-per-document design — but
  it is n=1, self-report, with no counterfactual (no matched non-agent
  workflow, no controlled removal of the memory layer to see what
  breaks). It cannot substitute for AiScientist's ablation numbers; it
  can and does substitute for the thing ablations can't provide —
  evidence the pattern survives contact with real, messy, multi-month
  academic work rather than a benchmark harness.
- **A document-grain data point now exists alongside the
  workspace-grain one.** Knows ([[literature/papers/yu2026knows]])
  arrives at the same core commitment — a durable file, not
  conversational context, is the thing agents actually read — from a
  different direction: not multi-agent workspace coordination but
  single-document consumption. Its KnowsRecord is a YAML sidecar that
  coexists with a PDF; agents read the sidecar and re-ground from it
  rather than re-parsing the prose on every invocation, which is
  exactly guidance #4 here ("re-ground, do not inherit") applied to a
  paper instead of a project workspace. The gains are large at that
  grain too — weak models gain +29 to +42pp accuracy and cut
  29–86% of tokens reading the sidecar instead of the PDF — a
  second, independently-motivated confirmation that a structured
  durable artifact beats re-deriving structure from unstructured
  context on every read, even when the "bus" is a single file rather
  than a multi-region workspace. It does not attest the *coordination*
  half of this concept (permission-scoped regions, multiple roles
  writing to distinct areas) — Knows has one sidecar per artifact, no
  role-based write partitioning — so it strengthens the re-grounding
  principle specifically, not the full workspace-bus pattern.
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

## Across a trust boundary, the file needs to carry its own credentials

Every instance above assumes a shared filesystem inside one trust domain:
the bus works because everyone touching it is already trusted.
[[literature/papers/ravindran2026portable]] is the version where that
assumption is dropped — memory serialized to a signed artifact that a
*different vendor's* agent re-hydrates, with capability-scoped tokens
controlling which parts the receiver may read. The file is still the bus;
it just has to authenticate itself now.

Two ideas transfer even though the paper's evidence is thin (see the note
for why): the artifact carries **its own authorization** rather than
relying on filesystem permissions, and redaction **preserves DAG position**
with typed tokens so erasure does not break downstream references — a
deletion story `file-as-bus` currently lacks entirely. See
[[concepts/verified-memory-writes]] for why the paper's provenance layer
verifies the wrong property.
