---
kind: paper
title: "AgenticSTS: A Bounded-Memory Testbed for Long-Horizon LLM Agents"
authors: ["Xiangchen Cheng", "Yunwei Jiang", "Jianwen Sun", "Zizhen Li", "Chuanhao Li", "Xiangcheng Cao", "Yihao Liu", "Fanrui Zhang", "Li Jin", "Kaipeng Zhang"]
institutions: ["Alaya Lab", "Shanghai Jiao Tong University", "Shanghai Innovation Institute", "Nankai University", "University of Science and Technology of China"]
year: 2026
venue: "arXiv 2607.02255v1, cs.AI (preprint, 2 Jul 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.02255
code_url: https://github.com/AlayaLab/AgenticSTS
citations:
source: "raw/papers/cheng2026agenticsts.pdf"
added: "2026-08-18"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/lossless-context-offload]]"
related_experiments: []
tags: [memory, context-management, bounded-memory, typed-retrieval, ablation, long-horizon, skill-library, benchmark-design, evaluation-methodology]
---

# AgenticSTS: A Bounded-Memory Testbed for Long-Horizon LLM Agents

## TL;DR

The contribution is a **methodology**, and the paper says so plainly rather
than dressing up its numbers. Its reframe: "memory for a long-horizon LLM
agent is a contract about what each future decision is allowed to see." The
usual contract — append prior observations, tool calls, and reflections to
every prompt — makes context "a jumbled mixture in which the effect of any
single memory component is hard to isolate." So it builds the alternative:
every decision gets a **freshly composed** user message assembled from five
typed slots, with **no raw cross-decision transcript appended ever**, which
bounds the prompt at any horizon *and* makes each layer independently
ablatable. Instantiated on Slay the Spire 2 (298 released trajectories).
The headline ablation — 3/10 wins without the skill layer, 6/10 with — is
explicitly **directional, not significant** (Fisher exact p ≈ 0.37).

## Claims

- **Memory is an access contract, not a store.** "For a long-horizon LLM
  agent, memory is not a place to store text; it is a contract about what
  each future decision is allowed to see." The choice of contract
  "determines what evidence the model sees, what stale information can
  re-enter a decision, and which component can be ablated when the agent
  succeeds or fails."
- **Transcript-appending is not just expensive — it is
  *epistemically* opaque.** Because everything accumulates into one
  undifferentiated blob, no component's contribution can be isolated. The
  bounded contract's payoff is framed as an *evaluation* property first and
  a cost property second, which is the inversion worth noticing.
- **The interface, formally.** Each decision `d` composes
  `u_d = π(L1, L2(s_d), L3(s_d), L4(s_d), L5(s_d))` from five layers
  separated by mutability and experimental role:
  - **L1** operator prompts — immutable role and protocol templates per
    state type
  - **L2** state-typed prompts — immutable schemas and legal action
    formats (combat, deckbuilding, map, event, intermission)
  - **L3** game knowledge — enumerable rule data, refreshed by patch,
    filterable
  - **L4** episodic memory — postrun summaries, retrievable, writable
    between runs
  - **L5** skill library — triggered strategic guides distilled from logs,
    each with an **explicit trigger condition**, indexed for retrieval
    across recurring state classes

  The hard rule: **"any information that survives across decisions must
  first be written into a bounded store."** Nothing crosses a decision
  boundary implicitly.
- **Four evaluation handles a raw prompt-history setup hides**: horizon
  growth capped by slot budgets; retrieved evidence labeled by layer;
  L4/L5 toggleable without rewriting the prompt; and runs, stores, prompts,
  and scripts carrying condition tags for reuse. This is the paper's actual
  deliverable.
- **Growth is bounded by construction.** A transcript interface has
  worst-case `Ω(d · s̄)` growth over `d` decisions; the token audit plots
  actual bounded usage against a transcript-appending counterfactual at ¼
  of naive `O(c²)` growth (i.e. already discounted for prompt caching).
  The audit is explicitly labeled as illustrating growth mechanics, not as
  a win-rate comparison — it runs at two runs per cell.
- **The task is hard but unsaturated.** At the easiest difficulty (A0), a
  public benchmark of frontier LLMs reports **zero wins across five model
  configurations**, against a developer-reported **16% human win rate**.
- **The skill layer (L5) is where the observed lift sits, and episodic
  memory (L4) does not separate.** All three L5-enabled cells win 6/10;
  the no-scaffold baseline wins 3/10. Notably, full-frozen (*with* L4) and
  mode-a (*without* L4) **both win 6/10**, so L4 contributes nothing
  detectable at this scale.
- **A frozen memory stack is backbone-sensitive.** The L4+L5 stores were
  distilled from Gemini 3.1 Pro trajectories; transplanting them to other
  model families is reported as a separate diagnostic stream, not pooled
  into the headline, because the stack does not transfer cleanly.

## Methods

Training-free harness over Slay the Spire 2, a closed-rule stochastic
deck-building roguelike chosen because the rule space is "closed, symbolic,
and text-readable" — so game facts and legal actions ship as structured
records rather than pixels — while a run still requires hundreds of tactical
and strategic decisions with delayed consequences. The fixed-A0 ablation is
a balanced 50-game matrix (five conditions × 10 completed games): no-scaffold
baseline; a prompt-strictness variant; mode-a (human-authored L5 seed
bodies); mode-b-frozen (stub-template-filled L5 bodies); full-frozen (mode-a
plus the frozen L4 store). L5 is populated two ways — mistake-driven
discovery from combat losses (self-evolve) and human-authored seeds. Cell
win rates use Wilson 95% intervals; continuous scores use 5,000-sample
bootstrap. Separate, smaller streams cover the difficulty ladder (A0→A10
with stores updating between runs) and a cross-backbone probe. Released:
**298 condition-tagged trajectories, SHA-anchored L4/L5 snapshots,
decision-time prompt records, and the analysis scripts**, plus a HuggingFace
dataset.

## Results

- Fixed-A0, N = 10 per cell: no-scaffold **3/10** (Wilson [10.8%, 60.3%]),
  prompt-strictness variant 4/10 ([17%, 69%]), all three L5 cells **6/10**
  ([31.3%, 83.2%]).
- Decomposed: Δ_prompt = +1/10 (strictness and wrappers), Δ_L5 = +2/10 at
  the same prompt setup.
- **Neither result is significant.** Fisher exact on 3/10 vs 6/10 gives
  **p ≈ 0.37**; pooling all scaffolded vs unscaffolded cells (18/30 vs
  7/20) gives **p ≈ 0.148**; the Wilson intervals overlap. The paper's own
  reading: "L5 as the layer with the largest observed difference in the
  balanced matrix — a directional [finding]."

## Critique / open questions

- **The central comparison was not run, and the paper admits it.** There is
  no same-codebase accumulating-context cell. Everything about the bounded
  contract's *performance* rests on external, differently-configured public
  baselines, which the paper repeatedly labels "operational comparisons
  rather than controlled tests of the contract variable itself." Limitations
  names this as "the cleanest direct comparison to the bounded contract"
  and leaves it to future work with the artifact provided. So the paper
  establishes that the contract is *ablatable*, not that it is *better*.
- **n = 10 per cell against a stochastic roguelike.** Slay the Spire runs
  have enormous variance from card draws, events, and enemy sampling alone.
  A 3/10 → 6/10 swing is three games. The paper handles this correctly
  (Wilson intervals, Fisher tests, refusing the significance claim) but it
  means no layer-level conclusion survives.
- **Single character, single game.** Headline runs use one character
  (Silent) to keep the typed substrate self-consistent; the L3/L4/L5 stores
  must be repopulated per character. Cross-game transfer is a stated
  non-target.
- **L4's null result is under-discussed.** That the episodic layer changes
  nothing (6/10 either way) is the more interesting finding for this
  project than L5's directional lift, since episodic summarization is what
  most agent memory systems actually ship. At this sample size it is a null
  result with no power behind it, not evidence of absence — but it deserves
  more than a sentence.
- **Backbone-sensitivity of the frozen stack is a real limit on the
  "reusable methodology" claim.** If distilled skills don't transfer across
  model families, the archive's value is as a protocol rather than as
  content.
- Some conditions differ in more than one mechanism — the paper flags this
  in a figure caption ("not every neighboring pair is a single-mechanism
  isolation"), which is honest but does dull the ablation surface it is
  selling.
- Game-agent domain. Whether a contract tuned for turn-based discrete
  decisions with enumerable legal actions transfers to open-ended research
  work — where the action space is not enumerable and there is no `L2`
  legal-action schema to write — is exactly the untested case here.

## Trust signals

- **Credibility:** 4 — unusually disciplined reporting: the abstract itself
  states the headline is directional and gives the p-value, figure captions
  mark which panels are illustrative, external comparisons are labeled
  operational rather than controlled, non-peer-reviewed references are
  annotated as such in the bibliography, and the Limitations section leads
  with the missing comparison rather than burying it. Full artifact release
  (code, 298 trajectories, SHA-anchored store snapshots, prompt records,
  analysis scripts, HF dataset). Held below 5: preprint; every quantitative
  claim is underpowered; single game and character; and the comparison that
  would validate the central thesis is absent.

## Follow-up

- **Relevance: 4** — the framing sentence is the import: *memory is a
  contract about what each future decision is allowed to see*. That is a
  cleaner statement of what [[concepts/context-eviction-policy]] is
  actually about than the concept's current framing (a rule for what stays
  when the buffer would overflow), because it makes the policy primary and
  the overflow incidental. A bounded contract has no overflow event at all
  — the question is never "what do we drop" but "what was allowed in."
- **The strongest idea here is that a memory architecture determines what
  you can measure.** Append-everything makes component attribution
  impossible; typed slots make each layer toggleable. This connects
  [[concepts/hce-evaluation]] to the memory cluster in a direction neither
  had: *ablatability is a design criterion for the harness, not just a
  property of the experiment*. Any harness whose memory is an
  undifferentiated transcript cannot answer "which part of the context
  earned this result" — which is the question every autonomous-loop
  post-mortem in this project wants to ask.
- **The "bounded store" rule is directly adoptable and stricter than
  anything the graph currently states**: *any information that survives
  across decisions must first be written into a bounded store*. Compare
  [[concepts/lossless-context-offload]], which moves material to a colder
  tier while keeping it addressable; this forbids implicit survival
  entirely, so the offload is mandatory rather than opportunistic. The two
  are the same mechanism with opposite defaults, and the difference is
  worth stating in that concept.
- **L5's trigger-indexed guides are
  [[concepts/skill-library-lifecycle]] with an explicit retrieval key** —
  each guide carries a trigger condition, a prose body, and is indexed for
  recurring state classes, populated by *mistake-driven discovery* from
  logged losses. That population mechanism (distill a skill from a failure,
  not from a success) is the part worth carrying; most skill-library
  sources distill from successful trajectories.
- **Treat the numbers as a null result, not support.** Nothing here
  strengthens any claim in the graph — 3/10 vs 6/10 at p ≈ 0.37 is
  compatible with no effect, and the L4 cells are flat. Cite this for the
  contract, the token-growth argument, and the released archive; do **not**
  cite it as evidence that typed retrieval beats accumulation. That
  comparison remains open, and this paper is the clearest statement of
  *why* it is open.
- **A watch item for the runtime-memory cluster's policy-locus axis.** This
  is the far deterministic pole: the memory policy is fixed slot budgets and
  typed retrieval with the model given no say at all, sitting opposite
  model-initiated compaction. Between mason2026missing's proxy and this,
  the axis now has both endpoints instrumented; what nothing measures is
  whether the model's own judgment about what to keep beats a fixed
  schedule.
