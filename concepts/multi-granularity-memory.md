---
kind: concept
name: "multi-granularity-memory"
status: seedling
added: "2026-06-23"
source_papers:
  - yu2026hmem
  - sun2026rethinking
  - omri2026agent
  - pham2026memorai
  - kerestecioglu2026human
  - xu2026single
sources:
  - "[[literature/papers/yu2026hmem]]"
  - "[[literature/papers/sun2026rethinking]]"
  - "[[literature/papers/omri2026agent]]"
  - "[[literature/papers/pham2026memorai]]"
  - "[[literature/papers/kerestecioglu2026human]]"
  - "[[literature/papers/xu2026single]]"
  - "[[literature/papers/zhou2026ready]]"
  - "[[literature/papers/ji2026memory]]"
  - "[[literature/papers/xu2025amem]]"
  - "[[literature/papers/gao2026mempoison]]"
  - "[[literature/papers/tang2026memory]]"
  - "[[literature/papers/lee2026minteval]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/verified-memory-writes]]"
related_experiments: []
tags: [memory, granularity, retrieval, knowledge-organization, consolidation, multi-store]
---

# multi-granularity-memory

## Definition

An agent's memory is stored at **several coexisting levels of grain
simultaneously** — raw segments / turns, atomic facts, synthesized
summaries, keywords; or working / episodic / semantic tiers — and
retrieval *selects or weights across those grains per query* rather than
reading a single flat store at one fixed resolution. The two load-bearing
moves are (1) **constructing** the same experience at multiple
granularities and associating them, and (2) **routing** retrieval to the
grain that fits the current query.

## Why it matters here

Single-granularity memory is a recurring failure mode. Store everything
raw and retrieval drowns in noise and token cost; store only summaries and
the fine detail a later query needs is already gone. Across six
independent attestations the same fix recurs — keep multiple grains and
decide which to read at query time:

- **xu2026single (MemGAS, ICLR 2026)** is the cleanest anchor: four
  co-stored granularities (session / turn / summary / keyword), a
  cross-granular GMM **association graph**, and an **entropy-based router**
  that weights each grain by the certainty of its similarity distribution.
  It shows single-granularity retrieval is the failure mode and supplies
  the canonical mechanism.
- **kerestecioglu2026human** gives the cognitively-motivated version: three
  biological tiers (PFC hot-cache → hippocampal episodic → neocortical
  semantic graph) with **consolidation between tiers** and lifecycle
  mechanisms (sleep-phase consolidation, engram maturation,
  reconsolidation).
- **yu2026hmem**, **sun2026rethinking** (raw segments / atomic facts /
  synthesized granularities), **omri2026agent** (per-turn / session / topic
  construction granularity), and **pham2026memorai** (multi-granularity +
  graph retrieval) are the corroborating attestations from distinct groups
  and venues.
- **xu2025amem (A-Mem, NeurIPS 2025)** attests the *write-side* half
  (guidance #1) from the Zettelkasten direction: every stored experience
  is constructed at multiple representations up front (raw content +
  LLM-generated keywords, tags, and contextual description, jointly
  embedded), and its 85–93% per-question token reduction vs flat-context
  baselines is the efficiency case for paying construction cost at write
  time.
- **ji2026memory (MRAgent, ICML 2026)** adds the third routing paradigm:
  where MemGAS routes by entropy and H-MEM by hierarchy, MRAgent
  *reconstructs* — the grains (episodic / semantic / topic layers over a
  Cue–Tag–Content graph) are traversed by LLM reasoning inside the
  retrieval loop, with intermediate evidence redirecting which grain is
  read next and a sufficiency judgment deciding when to stop. Comes with
  an expressivity proof (active/stateful retrieval strictly contains
  passive) and a cost result worth quoting: tag-gated access to grains
  *cut* per-sample tokens to 118k vs 245–632k for flat or
  summarize-everything baselines — multi-granularity as an efficiency
  mechanism, not just a recall one.

For this project the pattern maps directly onto the knowledge graph: a
concept note already exists at several grains — the one-line `index.md`
pointer, the `## Definition`, the full body, the `sources:` list, the git
history. A retrieval policy that pulls the *right grain* for the query
(definition for a quick check, full body for synthesis, history for "how
did this claim change") is the read-side counterpart to
[[concepts/selective-memory-retrieval]].

## Implementation guidance

1. **Construct at multiple grains at write time.** When an experience is
   stored, also derive and link its coarser forms (a summary, keywords) so
   the coarse grain exists *before* it is needed — don't summarize lazily
   at read time.
2. **Associate across grains, don't silo them.** MemGAS's GMM graph and
   the human-memory consolidation links both make the point: the grains
   must be cross-referenced so retrieving one can surface its neighbors at
   other resolutions.
3. **Route, don't concatenate.** Weight or select grains per query
   (entropy of the match distribution is one concrete signal); dumping all
   grains into context reintroduces the single-store noise problem.
4. **Bound each grain independently.** The fine grain needs an eviction /
   consolidation policy ([[concepts/context-eviction-policy]]) or it grows
   unbounded; the coarse grain is cheap to keep. Different grains warrant
   different retention budgets.

## Connections

- **[[concepts/selective-memory-retrieval]]** is the read-side dual:
  multi-granularity-memory says memory exists *at several grains*;
  selective-memory-retrieval says *when and which* to consult. MemGAS's
  entropy router is literally an instance of both — uncertainty-gated
  selection *across granularities*.
- **[[concepts/agent-native-memory]]** is the write-side substrate:
  agent-native-memory says memory is a first-class self-managing
  subsystem; multi-granularity is one axis along which that subsystem is
  organized.
- **[[concepts/context-eviction-policy]]** is what keeps the fine grain
  from exploding; consolidation-between-tiers (kerestecioglu2026human) is
  eviction reframed as promotion to a coarser grain.
- **[[concepts/structured-world-model]]** — the coarse semantic grain is a
  compressed world model the agent reasons over without re-reading raw
  history.

## Open questions

- **Who picks the grain set?** All six sources hand-design the granularity
  ladder (turn / summary / keyword, or working / episodic / semantic).
  Whether the optimal set of grains can itself be learned or evolved
  (cf. [[concepts/evolutionary-expansion]]) is open.
- **Grains can differ in *kind*, not only in resolution.**
  [[literature/papers/tang2026memory]]'s ladder is L1 grounded step
  traces → L2 induced procedural policies → L3 declarative environmental
  cognition. That is not a compression ladder: L2 is a *procedure* and L3
  is a *model of the environment*, neither of which is a summary of L1.
  It suggests the concept's framing (same content at several
  resolutions) may be a special case of a broader one — several
  representations of different type, associated by evidence links. Its
  flat-memory ablation is also the strongest quantitative argument in the
  cluster that the ladder is load-bearing rather than decorative:
  collapsing it drops information-retrieval Pass@1 from 26.15 to 10.77,
  below several baselines, while *increasing* cost.
- **Routing signal robustness.** Entropy of a similarity distribution is
  cheap but proxy; whether it degrades on adversarial or out-of-distribution
  queries (where confident-looking matches are wrong) is untested.
- **Cost accounting.** Constructing every experience at N grains multiplies
  write cost; none of the sources fully account for the construction
  overhead against the retrieval-quality gain.
- **Grain choice is also a security parameter — with an unresolved sign.**
  [[literature/papers/gao2026mempoison]] measures poisoning resistance
  across three substrates and finds it tracks granularity: a decomposed
  fact store is most robust (56.6% behavioral corruption), hierarchical
  notes intermediate (63.1%), flat chunks worst (67.9%). The mechanism is
  "decomposition-induced dilution" — breaking an entry into atomic facts
  dissolves an injected payload that survives intact in a raw chunk, and
  summarization attenuates it partially. That is a second, independent
  argument for a design this concept already favours on retrieval
  grounds.

  But the sign is not uniform, and the same paper supplies the
  counter-case: its hardest attack class works by *splitting* a payload
  across several individually plausible records that only become harmful
  when co-retrieved. Fine granularity dilutes single-record poisons and
  is the native medium for compositional ones. Whether decomposition is
  net protective depends on which attack class dominates, and no source
  yet measures the tradeoff directly — worth resolving before treating
  fine grain as a safety win.
