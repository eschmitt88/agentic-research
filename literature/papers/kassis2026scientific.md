---
kind: paper
title: "Scientific Agent Skills: A Library of Procedural Knowledge for Research Agents"
authors: ["Timothy Kassis", "Vinayak Agarwal", "Yuhuan He", "Darshil Patel", "Aubrey M. Brueckner"]
institutions: ["K-Dense, Inc."]
year: 2026
venue: "arXiv (cs.CL)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.00065"
code_url: "https://github.com/K-Dense-AI/scientific-agent-skills"
citations: null
source: "raw/papers/kassis2026scientific.pdf"
added: "2026-09-07"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["skills", "skill-library", "knowledge-organization", "markdown", "context-budget", "format-portability", "discovery"]
---

# Scientific Agent Skills: A Library of Procedural Knowledge for Research Agents

## TL;DR

"A language-model agent asked to analyse an experiment will usually return
working code. Whether the analysis is defensible is a different question."
163 skills across 16 areas of practice, each a directory around a
versioned Markdown instruction file, encoding the field-specific
procedural conventions that separate running code from a defensible
result. The paper reports **no task-level evaluation and no host selection
rate** — deliberately — and instead measures the documentation corpus's
context cost.

## Claims

- Defensibility rests on procedural choices: "which test the field accepts,
  which identifier namespace is authoritative, and which caveats must
  accompany a result." That knowledge is real, documented, and scattered
  across package docs, reporting guidelines, standards and tacit practice.
- Rediscovering it per task is costly, so record it once. Scientific
  communities already do this for humans (ARRIVE, MIQE) precisely because
  the same procedural mistakes recur across the published literature.
- A skill is portable and reviewable **because it is text**: the convention
  "specifies no runtime, API or service."
- Progressive disclosure is the mechanism — the host keeps only each
  skill's name and short description resident and reads the full file when
  a task appears to call for it.

## Methods

- 163 skills, 16 areas: genomics, cheminformatics, medical imaging, study
  design, scientific communication, and other biology/chemistry/medicine/
  physical-science workflows.
- Each skill is a directory: Markdown instruction file with a short YAML
  header, optionally reference documents, executable scripts, static assets.
- Concrete failure→rule mapping, e.g. `statistical-analysis` requires
  saying which multiple-testing correction was used; `experimental-design`
  states "3 mice with 100 cells each is n = 3 (mice), not n = 300 (cells)";
  `bulk-rnaseq` warns that when treated and control samples run on
  different days "the effect is unrecoverable"; `pysam` fixes that
  "numeric coordinates accepted by pysam APIs are 0-based, half-open."
- Measurement is of the corpus, not of agent behaviour.

## Results

- Always-resident descriptions of all 163 skills cost **7.1% of a
  200,000-token window**.
- Median documented workflow fits within **23.9%** of the window — but
  **29 of 46 would overflow** if every reference file were loaded.
- Openly licensed and released.
- Explicitly: "We report no task-level evaluation and no host selection
  rate." And: "We wrote most of them ourselves, so this is a description of
  our own artifact rather than an independent audit."

## Critique / open questions

- **The null is the finding.** A 163-unit curated library shipped with zero
  evidence that it changes agent behaviour. The honesty is admirable and
  the gap is total: no selection rate means we do not know whether a host
  even *loads* the right skill, which is the first thing that has to work.
- 29 of 46 workflows overflowing on full reference load is a real
  architectural problem stated in passing. Progressive disclosure solves
  the always-resident cost; it does not solve the loaded-workflow cost.
- Authored by the maintainers of the artifact; the failure→rule mapping in
  the introduction is a demonstration, not a sample.
- Commercial entity (K-Dense) publishing its own library, though openly
  licensed.

## Trust signals

- **Credibility:** 3 — arXiv preprint, not peer reviewed, no citations, and
  self-authored artifact description with an acknowledged conflict. Raised
  to 3 by a genuinely **released, openly licensed artifact** at real scale
  (163 skills), reproducible corpus measurements, and unusually disciplined
  scope-setting: it claims exactly what it measured and explicitly disclaims
  the evaluation it did not run.

## Follow-up

- **Relevance:** 5 — the closest published artifact to this project's own
  structure: versioned human-readable Markdown units with YAML headers,
  loaded on demand, portable because they are text. That is
  [[concepts/llm-wiki-pattern]] and [[concepts/shared-skill-namespace]]
  built at 163 units by someone else, and it is the first external
  datapoint on what such a library *costs in context* — 7.1% resident, which
  is directly comparable to this repo's own instruction-corpus accounting
  in NOTES.md.
- Strongest use is as the negative anchor for
  [[concepts/skill-library-lifecycle]]: the concept's other sources measure
  gains from accumulation; this one shows a serious library can be built
  and shipped with *no* evidence of gain. Cite it whenever the concept is
  used to justify accumulation.
- The 29-of-46 overflow result is a live constraint for
  [[concepts/context-eviction-policy]] and
  [[concepts/selective-memory-retrieval]]: description-level indexing is
  cheap, but the retrieved unit itself may not fit. The eviction question
  reappears one level down, inside a single skill.
- Contrast with [[literature/papers/tang2026wikiskill]], which does run the
  ablation isolating library contribution (48.7% → 63.7% when the skill
  proposer has wiki access). Read together they bracket the question:
  wikiskill shows accumulated knowledge helps when co-evolved with skills;
  this shows curation alone, unevaluated, proves nothing.
- Direct action item: 7.1% resident for 163 skills implies ~0.044%/skill,
  a usable budgeting constant for this project's own skill set.
