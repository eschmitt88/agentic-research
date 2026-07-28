---
kind: paper
title: "MINTEval: Evaluating Memory under Multi-Target Interference in Long-Horizon Agent Systems"
authors: ["Hyunji Lee", "Justin Chih-Yao Chen", "Joykirat Singh", "Zaid Khan", "Elias Stengel-Eskin", "Mohit Bansal"]
institutions: ["UNC Chapel Hill", "The University of Texas at Austin"]
year: 2026
venue: "arXiv preprint (v2)"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.18565"
code_url: "https://github.com/amy-hyunji/MINTEval"
citations: null
source: "raw/papers/lee2026minteval.pdf"
added: "2026-07-28"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/agent-native-memory]]"
related_experiments: []
tags: [memory, benchmark, interference, long-horizon, retrieval, evaluation]
---

# MINTEval: Evaluating Memory under Multi-Target Interference in Long-Horizon Agent Systems

## TL;DR

Existing memory benchmarks test **static, independent recall**. Real
agents face information that is repeatedly *revised* and that *interferes*
across memories. MINTEval measures the difference: 15.6k QA pairs over
contexts averaging 138.8k tokens (up to 1.8M), across four domains, with
both single-target recall and multi-target aggregation. Seven
representative systems average **27.9% accuracy**; the best reaches 33.4%.
The dominant bottleneck is **retrieval and memory construction**, not
answering.

## Claims

- New information usually does not overwrite prior information — it
  *revises* or *builds on* it. Benchmarks that treat facts as independent
  miss the whole dynamic.
- The hard cases are (i) **long-range lookback** — recalling an earlier
  fact that later updates have interfered with — and (ii) **multi-target
  aggregation** — reasoning over several relevant pieces at once.
- Performance degrades monotonically **as the number of intervening
  updates increases**: the further back the target, the worse the recall.
- The bottleneck sits earlier in the pipeline than most work assumes. Real
  memory is not just a long-context retrieval problem; it requires
  faithful preservation of evolving state, fine-grained updates, and
  reasoning over temporally distributed evidence.

## Methods

- Four domains chosen for genuinely different revision dynamics: state
  tracking, multi-turn dialogue, **Wikipedia revisions**, and **GitHub
  commits**. The last two are naturally-occurring revision histories rather
  than synthesized interference.
- Five question types spanning single-target recall (Simple, History) and
  multi-target aggregation (Ordering, Counting, Multihop). 1.8k of the
  pairs are multi-target aggregation.
- Seven systems across three families: vanilla long-context LLMs,
  retrieval-augmented generation, and memory-augmented agent frameworks
  (including SimpleMem, AtomMem, Full Context, RAG).
- **Decomposition analysis** treating 100% as the upper bound — all
  questions are generated directly from the source material, so every
  answer is present by construction. This lets them attribute the shortfall
  between retrieval/memory-construction and answer generation separately.

## Results

- Average accuracy **27.9%**; strongest system **33.4%**. Far from
  saturated.
- Single-target recall is easier than multi-target aggregation (avg.
  26.5% on aggregation). History questions requiring long-range lookback
  average **21.0%**.
- **Retrieval and memory construction is the primary bottleneck**;
  answering errors add a further 25.2% drop. All four decomposed systems
  share an answering agent, yet differ substantially — so memory
  construction quality, not the reader, explains most of the spread.
- **Memory-based agents degrade less** than Full Context and RAG as
  intervening updates accumulate; the authors attribute this to better
  temporal encoding. Full Context shows the largest degradation from first
  to last revision.
- Limited cross-domain generalization: systems that do well on bAbI (short,
  simple contexts) degrade substantially on Wikipedia revisions.

## Critique / open questions

- The 100%-upper-bound framing is clean but slightly generous to the
  benchmark: questions generated directly from the source guarantee the
  answer is *present*, not that it is *unambiguously identifiable* after
  many revisions. Some aggregation questions may be genuinely
  under-determined, and the paper does not report a human ceiling.
- "Retrieval and memory construction is the bottleneck" is a decomposition
  over four systems, all using the same answering agent. A stronger reader
  might shift the attribution; the claim is about current systems, not a
  structural result.
- No memory system in the comparison was designed for revision-heavy input
  specifically, so the result partly measures a gap nobody has targeted yet
  rather than a hard limit.
- Interference is measured but not *characterized* — the paper establishes
  that performance falls with intervening updates without distinguishing
  the mechanisms (retrieval dilution vs stale-entry preference vs
  aggregation failure), which is what a fix would need.

## Trust signals

- **Credibility:** 4 — strong provenance (Mohit Bansal's group at UNC,
  with UT Austin), **code and data released** at
  `amy-hyunji/MINTEval`, large and carefully decomposed evaluation across
  seven systems and four domains, v2 revision, honest about the benchmark
  being unsaturated. Not peer-reviewed.

## Follow-up

- **Relevance:** 4 — quantifies the failure
  [[concepts/selective-memory-retrieval]] and
  [[concepts/context-eviction-policy]] exist to prevent, which those
  concepts have so far argued rather than measured. It also supplies a
  framing the memory cluster needs: **the bottleneck has shifted from
  acquisition to access.** Most of the cluster's sources are about what
  gets written; this one shows that even with everything written, the
  retrieval-and-construction step loses most of the value.
- The "revision, not overwrite" model is the durable idea. It reframes
  memory as **state evolution** rather than fact accumulation, which makes
  eviction and retrieval fundamentally temporal problems — you cannot
  decide whether an entry is stale without knowing what superseded it.
  [[concepts/multi-granularity-memory]] is the natural home for that.
- Directly relevant to this project's own graph, which *is* a revision-heavy
  store: concept notes accumulate attestations that revise earlier claims
  (e.g. `verified-memory-writes` now carries an impossibility result that
  qualifies an earlier defense). MINTEval's finding is that retrieval over
  such a store degrades with revision depth — an argument for why concept
  notes must be *rewritten* rather than appended to indefinitely, and a
  caution about how long the append-a-paragraph-per-source pattern stays
  readable.
- Wikipedia revisions and GitHub commits as evaluation substrates are a
  reusable construction trick: real revision histories are free,
  timestamped, and adversarially messy in the way synthetic interference
  is not.
