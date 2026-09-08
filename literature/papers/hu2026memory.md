---
kind: paper
title: "The Memory Trust Gap: Capability-Dependent Failures in Persistent-Memory Agents"
authors: ["Jundong Hu", "Shekar Ramachandran"]
institutions: ["PayPal AI"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.01852"
code_url: null
citations: null
source: "raw/papers/hu2026memory.pdf"
added: "2026-09-08"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/constraint-pinning]]"
tags: ["memory", "staleness", "trust", "provenance", "scaling", "diagnostics"]
---

# The Memory Trust Gap: Capability-Dependent Failures in Persistent-Memory Agents

## TL;DR

**A stale stored fact can override current authoritative evidence without
warning** — and the harm is *capability-gated*: the larger models collapse
hardest once a stale note is made to look current. Over-trust, not
confusion. Notably, a recency cue (stale note dated newer) fools the
*bigger* models more.

## Claims

- The failure is **over-trust in the store**, not an inability to
  distinguish sources. Models are not confused; they defer.
- Harm scales *with* capability rather than against it, for the features
  that matter most — the usual "this will be fixed by a better model"
  assumption is wrong here.
- Mitigation is itself capability-dependent, so a single fix does not
  cover a model range.

## Methods

- Frozen, closed-set, **action-scored** benchmark with two suites encoding
  two different meanings of "no memory": a **Benefit** suite (unsolvable
  without the stored fact) and a **Safety** suite (an authoritative tool
  always holds the correct value, so trusting memory is always wrong).
- Same-family model-size series: **Qwen3 0.6 / 1.7 / 4 / 8B**.
- **2×2×2×2 factorial** over note features (label, recency, source
  authority, position).
- Scale interactions confirmed by **direct cross-size contrast tests**
  rather than overlapping per-model intervals.
- Replicated on an independent Llama-Instruct size series and two external
  datasets (RGB, MisBench).

## Results

- Benefit suite: models answer with the stale value **0.92–1.00 of the time
  at every scale** — the store is trusted essentially unconditionally.
- Safety suite: harm below the no-memory baseline (Δ_mem) is
  **capability-gated**, with larger models collapsing most when a stale note
  is dressed to look current.
- **Removing a label amplifies over-trust at every size**; a **recency
  feature (stale dated newer) fools the larger models harder**.
- Source authority is a **weak and scale-flat** signal; position flips from
  positive to negative across the size series.
- Mitigation: exposing metadata helps the capable models, but only
  **pre-resolving the conflict** restores accuracy for the two smaller
  checkpoints.
- Framing control: at the three smaller scales models trust a stale
  *document* more than a stale *memory*; at 8B the difference is not
  significant.

## Critique / open questions

- Sub-10B open-weight models only. "Capability-gated" is established across
  0.6→8B; extrapolating the trend to frontier scale is the paper's implicit
  invitation and is not tested.
- Synthetic traps by construction — the Safety suite is designed so trusting
  memory is *always* wrong, which is not the real base rate. It measures
  susceptibility, not expected harm.
- No released code or benchmark.

## Trust signals

- **Credibility:** 3 — arXiv preprint from an industrial lab (PayPal AI),
  not peer reviewed, no citations, no artifact released. The experimental
  design is well above the band: a factorial with cross-size contrast tests
  instead of interval overlap, plus replication on a second model family
  and two external datasets. That replication is what earns the 3.

## Follow-up

- **Relevance:** 4 — [[concepts/verified-memory-writes]] is about gating
  what *enters* the store. This is the complementary failure: an entry that
  was valid when written silently outranks fresh authoritative evidence at
  *read* time. The concept needs a read-side clause, and this supplies the
  evidence for it.
- **"Removing a label amplifies over-trust at every size"** is a direct
  argument for the frontmatter discipline this project already runs — an
  unlabelled fact is trusted more, not less. It also means a `status:` or
  `added:` field is doing safety work, not just bookkeeping.
- The **recency-cue result is a caution for this repo specifically**: notes
  here carry an `added:` date, and a stale concept whose date was refreshed
  by an unrelated edit is exactly the "stale dated newer" trap. Touching a
  file is not the same as revalidating it.
- Pairs with [[literature/papers/goyal2026does]] on the memory front:
  goyal shows *format* determines whether memory survives a model change,
  hu shows *capability* determines whether surviving memory is trusted too
  much. Together they make model upgrade a two-sided risk for any
  persistent store.
- Also a second read on [[concepts/constraint-pinning]] — a pinned
  constraint is a stored fact, and this says a stale one will win against
  live evidence.
