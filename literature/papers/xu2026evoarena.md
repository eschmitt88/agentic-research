---
kind: paper
title: "EvoArena: Tracking Memory Evolution for Robust LLM Agents in Dynamic Environments"
authors: ["Jundong Xu", "Qingchuan Li", "Jiaying Wu", "Yihuai Lan", "Shuyue Stella Li", "Huichi Zhou", "Bowen Jiang", "Lei Wang", "Jun Wang", "Anh Tuan Luu", "Caiming Xiong", "Hae Won Park", "Bryan Hooi", "Zhiyuan Hu"]
institutions: ["National University of Singapore", "Singapore Management University", "University of Washington", "University College London", "University of Pennsylvania", "Nanyang Technological University", "Recursive", "MIT"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.13681
code_url: https://github.com/Aiden0526/EvoArena
citations: null
source: "raw/papers/xu2026evoarena.pdf"
added: "2026-06-22"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/context-eviction-policy]]"
tags: [memory, evolution, version-aware, benchmark, provenance, git, long-horizon]
---

# EvoArena: Tracking Memory Evolution for Robust LLM Agents in Dynamic Environments

## TL;DR

EvoArena is a benchmark suite that models environments as **chains of
progressively evolving releases** (same goal, changing interfaces / rules
/ workflows / code / user preferences) across terminal, software, and
social domains. The companion method **EvoMem** is a **git-like,
append-only patch history** over an agent's memory: each patch stores the
pre-update memory, post-update memory, rationale, and supporting evidence,
making memory evolution *traceable* and enabling version-aware retrieval.

## Claims

- Agents are usually evaluated on **static environment snapshots**, but
  deployment is dynamic; a reliable agent must know *what changed, what
  still holds, and how to act under the current version*.
- The dominant failure mode is **state collapse**: most memory-based
  agents keep memory as *a single latest state* (retrieved bank /
  episodic store), which works when new info cleanly supersedes old but
  is brittle when different versions need different behaviors — an update
  can overwrite a rule still valid for an older release, losing both the
  prior behavior and the context explaining when it was valid.
- The fix is **version-aware state tracking**: retain the latest memory,
  recover relevant prior states, and reason over *why* each update
  occurred.
- Current agents struggle on EvoArena — average accuracy 39.6% across
  evolving terminal/software/social-preference domains.

## Methods

- **EvoArena** organizes each environment into a chain of evolving
  releases; sub-benchmarks: **Terminal-Bench-Evo**, **SWE-Chain-Evo**,
  **PersonaMem-Evo**. Measures both *forward adaptation* (to new changes)
  and *version compatibility* (still-valid prior knowledge). Two scores:
  step-level (local adaptation) and chain-level (sustained reliability
  across a consecutive sequence of related subtasks).
- **EvoMem** augments a standard memory system with an **append-only
  patch history** (git-style). Each patch = pre-update memory + post-update
  memory + update rationale + supporting evidence from the triggering
  context. At inference it retrieves from the latest memory by default and
  **selectively retrieves relevant patches** only when a query depends on
  overwritten states, conflicting evidence, or earlier versions.

## Results

- EvoMem: average **+1.5%** on EvoArena across agents/backbones; **+6.1%
  on GAIA**, **+4.8% on LoCoMo**; **chain-level +3.7%** (where success
  needs a consecutive sequence of related evolutionary subtasks).
- Mechanistic analysis: EvoMem improves *evidence capture* in memory —
  better preservation of complete evolving environment states.

## Critique / open questions

- Average EvoArena gain is modest (+1.5%); the larger wins are on
  chain-level reliability and on standard long-horizon benchmarks (GAIA,
  LoCoMo). The patch-history overhead vs. that gain isn't cost-accounted.
- "Append-only patch history" is unbounded by construction — how the
  patch store is pruned/compacted over very long deployments (the
  [[concepts/context-eviction-policy]] question) isn't addressed; EvoMem
  trades state collapse for unbounded growth.
- Domains are terminal/SWE/social-preference; the version-aware angle is
  plausibly central to research-agent memory (papers get retracted,
  methods get superseded) but is unattested there.

## Trust signals

- **Credibility:** 4 — strong, broad collaboration (NUS, MIT, UWashington,
  UCL, UPenn, NTU; Caiming Xiong / Bryan Hooi among authors) with **full
  artifacts released** — code (github.com/Aiden0526/EvoArena), a project
  page, and a HuggingFace dataset collection. An arXiv preprint (v2, not
  yet peer-reviewed), so short of 5, but well above a no-artifact preprint.

## Follow-up

- **Relevance:** 4 — a near-literal attestation of
  [[concepts/agent-native-memory]]'s core thesis ("git is the memory
  layer; every write carries provenance"): EvoMem makes the git-patch
  metaphor explicit (pre/post/rationale/evidence per change). It adds a
  *temporal/version* gating axis to
  [[concepts/selective-memory-retrieval]] (retrieve patches only when the
  query depends on overwritten/conflicting/earlier state — a new gating
  signal orthogonal to uncertainty) and a concrete instance of
  version-aware state for [[concepts/structured-world-model]]. Borderline
  5; held at 4 because it strengthens existing concepts rather than
  seeding a new importable one.
