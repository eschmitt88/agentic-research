---
kind: paper
title: "Kosmos: An AI Scientist for Autonomous Discovery"
authors:
  - Ludovico Mitchener
  - Angela Yiu
  - Benjamin Chang
  - Mathieu Bourdenx
  - Tyler Nadolski
  - Arvis Sulovari
  - Eric C. Landsness
  - Daniel L. Barabasi
  - Siddharth Narayanan
  - Nicky Evans
  - Shriya Reddy
  - Martha Foiani
  - Aizad Kamal
  - Leah P. Shriver
  - Fang Cao
  - Asmamaw T. Wassie
  - Jon M. Laurent
  - Edwin Melville-Green
  - Mayk Caldas
  - Albert Bou
  - Kaleigh F. Roberts
  - Sladjana Zagorac
  - Timothy C. Orr
  - Miranda E. Orr
  - Kevin J. Zwezdaryk
  - Ali E. Ghareeb
  - Laurie McCoy
  - Bruna Gomes
  - Euan A. Ashley
  - Karen E. Duff
  - Tonio Buonassisi
  - Tom Rainforth
  - Randall J. Bateman
  - Michael Skarlinski
  - Samuel G. Rodriques
  - Michaela M. Hinks
  - Andrew D. White
institutions: ["Edison Scientific", "FutureHouse", "University of Oxford", "Massachusetts Institute of Technology", "Stanford University", "Washington University in St. Louis", "University College London"]
year: 2025
venue: "arXiv:2511.02824 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2511.02824"
code_url:
citations:
source: "raw/papers/mitchener2025kosmos.pdf"
added: "2026-04-24"
relevance: 5
credibility: 4
status: skimmed
related_experiments:
  - world-model-at-short-horizons
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/structured-world-model]]"
tags:
  - ai-scientist
  - world-model
  - citation-anchoring
  - long-horizon-agent
---

# Kosmos: An AI Scientist for Autonomous Discovery

## TL;DR

Edison Scientific's Kosmos runs 12-hour autonomous research cycles
across scientific domains, coordinating a data analysis agent and a
literature search agent through a *structured world model* — the
shared state object both agents read and write. Independent evaluation
found 79.4% of report statements accurate; collaborators said a single
run was equivalent to ~6 months of manual research effort.

## Claims

- Long-horizon agent coherence (200+ iterations, hours of wall time)
  requires explicit shared state, not passing messages alone.
- A structured world model — not free-text scratchpads — is what lets
  two specialist agents (analysis, literature) stay aligned across
  hours.
- Every claim in the final report must be anchored to a citation,
  either internal (code, run result) or external (paper, dataset).
  Unanchored claims are the dominant failure mode.
- Scientific research is bottlenecked by synthesis, not by individual
  experiments; the win is coordination.

## Methods

- Two specialized agents: data analysis (code + run) and literature
  search (retrieval + synthesis).
- Shared structured world model records hypotheses, evidence,
  provenance, open questions.
- Runs scoped to ~12 hours wall time; ~42,000 lines of code executed
  per run; ~1,500 papers reviewed per run.
- External eval: independent reviewers score statement accuracy in
  final reports.

## Results

- 79.4% statement accuracy (external evaluators).
- Collaborators estimate ~6 months of manual research per run.
- Several findings corroborated concurrent unpublished work —
  suggesting the system is converging on real phenomena, not
  hallucinating.

## Critique / open questions

- 79.4% accuracy is high for a 12-hour autonomous run but still means
  ~1 in 5 claims is wrong; operationally, a human has to verify each
  before publication.
- Structured world model's exact schema matters a lot and is
  underspecified in the abstract — this is the lesson worth stealing
  for our own agent.
- Domain breadth makes depth of ablation hard; unclear which agent
  (analysis vs. literature) carries the most weight.

## Trust signals

- **Credibility:** 4 — Edison Scientific / FutureHouse with a broad
  academic consortium (Oxford, MIT, Stanford, WashU, UCL, Broad
  Institute); arXiv preprint, not yet peer-reviewed; no public code
  located. Distinguished by independent external evaluation of report
  accuracy (79.4%) and reproduction of concurrent unpublished
  findings — unusually strong validation for a preprint; held below 5
  only by the lack of peer review and a released artifact.

## Follow-up

- Deep-read for the world model schema.
- Compare "citation anchoring" to the Diagnostics-citation rule in
  our experiment template — this is the paper that motivated it.
- Test whether a structured world model meaningfully improves
  coherence in shorter (1-2 hour) runs, or whether it is only
  load-bearing at 12h+ horizons.
