---
kind: concept
name: "selective-memory-retrieval"
status: seedling
added: "2026-05-11"
source_papers:
  - zhao2026expweaver
  - yang2026graph
  - wu2026gam
  - du2026memory
sources:
  - "[[literature/papers/zhao2026expweaver]]"
  - "[[literature/papers/yang2026graph]]"
  - "[[literature/papers/wu2026gam]]"
  - "[[literature/papers/du2026memory]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/context-eviction-policy]]"
related_experiments: []
tags: [memory, retrieval-policy, knowledge-organization, uncertainty-gating, runtime-decisions]
---

# selective-memory-retrieval

## Definition

The agent decides at decision time *whether* to consult its memory,
rather than following a fixed schedule (inject once at task start,
or inject at every step). The gating signal is the agent's own
reasoning state — typically uncertainty, ambiguity, or repeated
failure on the current sub-goal — surfaced via a runtime trigger
(e.g., emitting a `[Retrieve]` token mid-reasoning) that activates
retrieval just for the current decision.

## Why it matters here

Most "memory for LLM agents" work concentrates on the *write side*:
how experience is constructed, represented, organized, and updated
([[concepts/agent-native-memory]] focuses there too). But two failure
modes show that the *read-side* policy is its own load-bearing
axis:

- **Initialization-only** under-utilizes memory: relevant experience
  loaded at task start goes stale as the trajectory unfolds, and
  later-stage failures cannot reach back into the repository.
- **Always-on** over-utilizes it: dumping retrieved memory into
  every step injects irrelevant context, distracts the policy, and
  in RL settings actually *reduces* performance below Init-only
  (zhao2026expweaver, Fig. 4: Always-on 67.9 vs Init-only 70.6 on
  Qwen3-4B ALFWorld GRPO).

The fix is to make retrieval a *runtime decision*, not a fixed
schedule. ExpWeaver ([[literature/papers/zhao2026expweaver]])
demonstrates this with a minimal prompt-engineering intervention,
shows the resulting gate is task-aware (more retrieval on long-horizon
embodied tasks than on QA) and model-aware (stronger backbones
retrieve less), and shows the behavior is learnable through GRPO.

For a research-agent project this is directly actionable. The
project's own skills today (`/digest`, `/iterate`,
sub-agents in `/implement`) mostly use initialization-only context
loading. Migrating to a gated read-policy — where the sub-agent
emits a trigger when it judges the current context is insufficient
— is a small change with potentially large effect.

## Implementation guidance

1. **Expose memory as optional, not default.** Inside the agent's
   reasoning loop, frame memory as an opt-in resource the agent
   *can* invoke, not a context block that arrives automatically.
   A literal trigger token (`[Retrieve]`, `[Consult]`,
   `[Need-Guidance]`) is the simplest implementation; structured
   tool-call invocations are equivalent.

2. **Gate on uncertainty signals, not schedules.** Triggers that
   correlate with reasoning difficulty work — token-level entropy,
   repeated tool failures, dead-ends in search, explicit "I'm not
   sure" reasoning traces. Avoid time-based or step-count-based
   triggers; they reintroduce the always-on failure mode.

3. **Cheap-default, escalation-on-trigger.** The default action
   path should run *without* memory consultation. Retrieval kicks
   in only when the gate fires — so the cost of retrieval is paid
   only on the subset of decisions where it actually helps.

4. **Persist the gate decisions for audit.** Because retrieval is
   explicit (emitted by the agent rather than triggered by a
   middleware layer), every invocation is visible in the
   trajectory. Log when retrieval fired, what was retrieved, and
   the post-retrieval decision — this is essential for downstream
   recalibration and is the "observability" property the paper
   highlights.

5. **The policy is learnable.** A prompt-only implementation works
   for capable backbones (GPT-5.2-class, Opus-class). For weaker
   models, the gating policy can be amplified via RL with
   task-success reward — the trigger becomes part of the trained
   action space rather than relying on instruction-following alone.

## Connections

- **[[concepts/agent-native-memory]]** is the write-side complement.
  Agent-native-memory says *where* memory lives and how it's
  organized; selective-memory-retrieval says *when* to read it.
  Both concepts are necessary for a complete memory architecture;
  neither subsumes the other.
- **[[concepts/typed-claim-partition]]** is the write-side dual:
  one structures *what is grounded* in the agent's output, the
  other structures *when to consult* the agent's stored experience.
  Together they cover the read/write split of agent memory.
- **[[concepts/evolutionary-expansion]]** — the RL-amplified version
  of selective retrieval is a population-level capability:
  trajectories that invoke memory at high-value points outperform
  and propagate, giving the gating policy itself as a learned
  artifact across the search loop.

## Open questions

- **Joint optimization of construction and utilization.** The paper
  intentionally holds construction fixed to isolate the read-side
  effect. Whether the *optimal* construction depends on the
  utilization policy (or vice versa) is open. A repository tuned
  for "always-on" reads may be wrong for selective retrieval, and
  vice versa.
- **Gating signals beyond uncertainty.** Token entropy is one
  reasoning-state signal; others (confidence on intermediate
  conclusions, contradiction detection, sub-goal failure) plausibly
  carry independent information. Whether to compose multiple gates
  or rely on a single learned trigger is unresolved.
- **Safety surface.** Selective retrieval amplifies whatever the
  repository contains. If memory holds stale, biased, or
  adversarially-injected content, the gate becomes a vulnerability
  surface — the system invokes bad memory *exactly when uncertain*,
  which is the worst time. Pairing with provenance / freshness
  filters at retrieval time may be necessary.
- **Cross-skill transfer.** Whether the gating policy learned in
  one task (e.g., embodied interaction) transfers to another
  (e.g., literature curation, code generation) is untested. The
  paper's evidence is within-task only.
