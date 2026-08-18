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
  - "[[literature/papers/lam2026governing]]"
  - "[[literature/papers/li2026complexmcp]]"
  - "[[literature/papers/pham2026memorai]]"
  - "[[literature/papers/sun2026rethinking]]"
  - "[[literature/papers/wu2026memory]]"
  - "[[literature/papers/yu2026hmem]]"
  - "[[literature/papers/zhou2026comprehensive]]"
  - "[[literature/papers/zhao2026expweaver]]"
  - "[[literature/papers/yang2026graph]]"
  - "[[literature/papers/wu2026gam]]"
  - "[[literature/papers/du2026memory]]"
  - "[[literature/papers/omri2026agent]]"
  - "[[literature/papers/xu2026evoarena]]"
  - "[[literature/papers/liu2026evolvemem]]"
  - "[[literature/papers/xu2026single]]"
  - "[[literature/papers/zhou2026ready]]"
  - "[[literature/papers/ji2026memory]]"
  - "[[literature/papers/gao2026mempoison]]"
  - "[[literature/papers/lee2026minteval]]"
  - "[[literature/papers/li2026acm]]"
  - "[[literature/papers/wu2026remember]]"
  - "[[literature/papers/cheng2026agenticsts]]"
  - "[[literature/papers/zhu2026lossy]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/verified-memory-writes]]"
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
   [[literature/papers/li2026acm]] adds an *addressed* variant: its
   `query_memory` tool retrieves by the identifier a prior compression
   step left in context, so the gate targets a known span rather than
   a similarity search — and its trained agent interleaves such probes
   with compression mid-task (see
   [[concepts/lossless-context-offload]]).

2. **Gate on uncertainty signals, not schedules.** Triggers that
   correlate with reasoning difficulty work — token-level entropy,
   repeated tool failures, dead-ends in search, explicit "I'm not
   sure" reasoning traces. Avoid time-based or step-count-based
   triggers; they reintroduce the always-on failure mode.

   *A second gating axis: temporal/version dependence.* EvoMem
   ([[literature/papers/xu2026evoarena]]) retrieves from the latest
   memory by default but **selectively pulls in historical patches only
   when the current query depends on overwritten state, conflicting
   evidence, or an earlier environment version**. This is a gating signal
   independent of uncertainty: the trigger is "this decision references a
   fact that has since changed," not "I am unsure." For a research graph
   it maps to consulting a concept's *history* (a superseded method, a
   retracted source) rather than only its current head — retrieve the
   `git log` of a note, not just the note, when the query is about how a
   claim evolved.

   *A third gating axis: continuation.* MRAgent
   ([[literature/papers/ji2026memory]]) extends the runtime decision from
   *whether* to read to *how long to keep reading*: retrieval is an
   iterative reconstruction loop where, each step, the agent judges
   whether accumulated evidence suffices and either stops or selects the
   next traversal action conditioned on what it has found so far. Its
   analysis shows the self-termination is well calibrated (max useful
   turns ≈ average turns taken) and that spending the same budget as
   parallel one-shot retrieval cannot substitute for adaptive depth.

   *A fourth gating axis: push-side intervention.* The Proactive
   Memory Agent ([[literature/papers/wu2026remember]]) moves the gate
   out of the acting agent entirely: a parallel memory agent observes
   the trajectory, maintains a structured bank, and each interval
   either injects one targeted reminder or explicitly stays silent
   (`<no_intervention/>`). Its ablations are the cleanest test of
   this concept's core claim so far — selective intervention (64.3
   macro on τ²-Bench) beats passive full-bank exposure (61.5),
   always-on injection (63.5), bank-less advisor guidance (61.0),
   and Mem0-style general retrieval (62.1). The framing also names
   the failure mode precisely: **behavioral state decay** — state
   can still sit *inside the window* and yet stop influencing
   decisions, which is why the read-side policy is not reducible to
   storage or retrieval quality. Push vs pull is now an open
   sub-axis: an observer that decides when to remind, or an actor
   that decides when to ask.

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

  **This is now measured, and the conclusion inverts the usual framing:
  retrieval is not only a risk surface but a necessary defense stage.**
  [[literature/papers/gao2026mempoison]] shows write-time gates have a
  structural ceiling against harm that emerges from *co-retrieval* of
  individually plausible records — no per-record admission check can see
  it, because it is not in any record. Retrieval-time source-reliability
  reweighting is the single best defense on that class (corruption 51.7%
  → 17.0%), beating every write-time method, while being among the worst
  on directly-harmful single records (43.7% vs 4.8%). The two stages fail
  on **disjoint** attack classes. So the right reading is not "retrieval
  gating is dangerous, filter at write time" but "the retrieval policy is
  where compositional harm must be caught, because nothing upstream can."
  See [[concepts/verified-memory-writes]].

- **Which memory actually caused the answer?** MemPoison's Mechanistic
  Influence Decomposition is a reusable diagnostic for exactly the claim
  this concept keeps needing to make: leave-one-out counterfactuals over
  the retrieved slate measure a single item's contribution;
  leave-*pair*-out isolates interaction effects between items; and
  comparing an item's influence across contexts measures whether it is
  conditionally activated. Better grounding for "this retrieval policy is
  working" than end-task accuracy, and applicable outside security.
- **Cross-skill transfer.** Whether the gating policy learned in
  one task (e.g., embodied interaction) transfers to another
  (e.g., literature curation, code generation) is untested. The
  paper's evidence is within-task only.

## The bottleneck has moved from acquisition to access

[[literature/papers/lee2026minteval]] is the measurement this concept has
been arguing without. Its premise is that real information does not
*overwrite* prior information — it **revises** it — so recall degrades
through interference from intervening updates, and existing benchmarks
that test static independent recall miss the entire dynamic.

The numbers are stark. Across seven systems (long-context LLMs, RAG, and
memory-augmented agent frameworks) on 15.6k questions over contexts
averaging 138.8k tokens and reaching 1.8M, average accuracy is **27.9%**
and the best system reaches **33.4%** — against a 100% upper bound that
holds by construction, since every question is generated directly from the
source material. Long-range lookback questions average 21.0%; multi-target
aggregation 26.5%.

The decomposition is what matters here: **retrieval and memory
construction is the dominant bottleneck**, with answer generation adding a
further 25.2% drop. All four decomposed systems share an answering agent
and still differ substantially, so the spread is attributable to how memory
was built and retrieved, not to the reader. Memory-based agents degrade
*less* than Full Context and RAG as intervening updates accumulate, which
the authors credit to better temporal encoding — the first direct evidence
in this cluster that structured memory beats raw context specifically under
revision pressure, rather than merely at length.

Consequence for this concept: retrieval quality cannot be evaluated on
static stores. A gate that selects the right entry from a fixed corpus may
still select a **superseded** entry from an evolving one, and staleness is
not detectable from the entry itself — only from what came after it.
Selection has to be temporal. See [[concepts/multi-granularity-memory]] for
the state-evolution framing this implies, and
[[concepts/context-eviction-policy]] for the eviction-side counterpart.

Caveat worth carrying: none of the seven systems was designed for
revision-heavy input, so this measures a gap nobody has targeted yet rather
than an established hard limit.
