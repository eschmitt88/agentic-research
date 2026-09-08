---
kind: paper
title: "Does Your Agent's Memory Survive a Model Upgrade? A Controlled Study of Memory Portability"
authors: ["Ankit Goyal", "Jaideep Ray"]
institutions: ["LinkedIn"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.05339"
code_url: null
citations: null
source: "raw/papers/goyal2026does.pdf"
added: "2026-09-08"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/compression-as-generalization-test]]"
tags: ["memory", "portability", "format", "provenance", "model-upgrade", "diagnostics"]
---

# Does Your Agent's Memory Survive a Model Upgrade? A Controlled Study of Memory Portability

## TL;DR

"Model upgrades are routine; memory migrations are not." Four memory
representations are held against a **writer swap**. Fixed-schema structure
transfers essentially perfectly; **model-compressed natural-language notes
are strongly model-coupled and shift by up to 13 points, asymmetrically by
direction**. The decisive finding for any notes-based store: 80% of the
deficit is information destroyed *at write time*, and store-only repair
cannot recover it.

## Claims

- An agent can keep the same memory store and still forget. Three distinct
  mechanisms: the new model reads old notes differently, mixed embedding
  versions break retrieval, and repair fails when the original evidence is
  gone.
- **Migration is direction-specific.** The same pair of models gives
  opposite-signed results depending on which way you migrate, so a single
  upgrade test does not generalise.
- Retaining the **raw source history** is what makes memory repairable;
  a compressed store is not self-sufficient.

## Methods

- Four representations of the same history: **LC-RAW** (verbatim,
  long-context), **RAG** (chunked + retrieved), **NOTES** (model-compressed
  natural language), **KG-fixed** (normalised fixed-schema knowledge graph).
- 48 synthetic histories with randomized answer codes and **exact scoring**
  (no judge in the loop).
- Two open-weight sub-10B models; the writer is swapped and the reader held,
  isolating write-side coupling.
- Diagnostic decomposition separates construction loss from retrieval failure.

## Results

- **KG-fixed: +0.0004 ± 0.0020** accuracy change after a writer swap —
  effectively immune.
- **NOTES: +9.91 or −13.28 percentage points**, depending on migration
  direction. Highly model-coupled and asymmetric.
- RAG: a 50/50 mixed embedding index captures only 4.96 of the 11.90-point
  gain from full re-embedding — **partial migration forfeits most of the
  benefit**.
- Attribution: **80% (0.467 ± 0.014) of the NOTES deficit is information lost
  during initial construction**; 81% (0.364 ± 0.012) of the RAG deficit is
  retrieval failure. Different representations fail for different reasons.
- **Store-only repair of NOTES fails to reach 90% recovery in all 48 cases**;
  retaining raw source history succeeds in 34/48 for one direction.

## Critique / open questions

- Synthetic histories with randomized answer codes buy exact scoring at the
  cost of realism — real notes carry semantic redundancy that may make
  compression far more robust than a random code, which is maximally
  compression-hostile. This likely **overstates** the NOTES penalty.
- Two sub-10B open-weight models. Whether frontier-scale readers are equally
  coupled to their writer is exactly what the result would need to establish
  before it binds a production design.
- No code released.

## Trust signals

- **Credibility:** 3 — arXiv preprint from an industrial lab (LinkedIn), not
  peer reviewed, no citations, no released artifact. Method is the strength:
  a controlled writer swap with exact scoring, error bars throughout, and a
  diagnostic decomposition that attributes the deficit rather than just
  reporting it.

## Follow-up

- **Relevance:** 5 — **this is load-bearing for this repository's own
  design and it is not flattering.** The project's memory model is
  plain Markdown with flat YAML frontmatter, and
  [[concepts/agent-native-memory]] (30 sources) rests on an untested
  assumption that such a store outlives the model that wrote it. This is
  the first source to measure that assumption, and the split it finds runs
  straight through our format: the **YAML frontmatter is KG-fixed**
  (fixed-schema, transfers at +0.0004), while the **Markdown prose body is
  NOTES** (model-compressed, ±13 points).
- The practical consequence is that the frontmatter is the durable layer and
  the prose is not — which is an argument for pushing load-bearing facts
  *into* structured fields, and an argument this project has not previously
  had a reason to make.
- **"80% of the deficit is construction loss, and store-only repair cannot
  fix it"** is the strongest available justification for the `raw/` is
  immutable rule: `raw/` *is* the retained source history, and this paper
  shows it is the only thing that makes a compressed note repairable. The
  rule was adopted on provenance grounds; this supplies an independent
  performance argument for it.
- Direction-specific migration testing is a requirement
  [[concepts/verified-memory-writes]] does not currently carry.
- Sharpens [[concepts/compression-as-generalization-test]]: compression is
  not just a test of understanding, it is a **coupling to the compressor**.
- Pairs with [[literature/papers/hu2026memory]] — if memory utility is also
  capability-dependent, then portability across an upgrade is doubly
  non-neutral.
