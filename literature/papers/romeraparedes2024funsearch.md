---
kind: paper
title: "Mathematical discoveries from program search with large language models"
authors:
  - Bernardino Romera-Paredes
  - Mohammadamin Barekatain
  - Alexander Novikov
  - Matej Balog
  - M. Pawan Kumar
  - Emilien Dupont
  - Francisco J. R. Ruiz
  - Jordan S. Ellenberg
  - Pengming Wang
  - Omar Fawzi
  - Pushmeet Kohli
  - Alhussein Fawzi
year: 2024
venue: "Nature 625, 468–475"
url: "https://www.nature.com/articles/s41586-023-06924-6"
source: "raw/papers/romeraparedes2024funsearch.pdf"
added: "2026-05-16"
relevance: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: [evolutionary-search, llm-program-search, deepmind, palm-2, automated-discovery, evaluator-fitness, foundational]
---

# Mathematical discoveries from program search with large language models (FunSearch)

## TL;DR

FunSearch (DeepMind, Nature 2024) is the foundational LLM-driven
evolutionary program-search system: a pre-trained LLM proposes
single-function mutations against a user-supplied evaluator, with an
islands-based programs database managing the population. It produced
the first non-trivial advance on the cap-set problem in 20 years and
new heuristics for online bin packing.

## Claims

- An LLM paired with an automated evaluator can discover **new,
  human-interpretable, provably-correct** solutions to open problems
  in mathematics and combinatorics — the first such discovery
  attributed to an LLM.
- Searching at **function granularity** (not whole programs)
  constrains the space, biases toward short / Kolmogorov-compressible
  solutions, and yields interpretable artifacts a researcher can
  read and learn from.
- The **evaluator is the only ground truth**: it runs each candidate
  and assigns a fitness score, and this is what neutralizes LLM
  hallucination — invalid programs are rejected by the loop, not by
  the LLM's reasoning.
- The same architecture transfers between pure mathematics (cap set)
  and applied optimization (bin packing) with no change to the
  evolutionary machinery — only the specification, skeleton, and
  evaluator change.
- **Diversity is structural, not incidental:** an islands-based
  population prevents collapse onto a single ancestor.

## Methods

User supplies three things at problem-definition time:

1. **Specification** — code that defines the problem (objective,
   constraints, problem-instance generator).
2. **Skeleton** — a fixed-by-the-user "backbone" that calls into a
   target function; the LLM only edits the target function, not the
   skeleton. This is the **human/LLM separation boundary**.
3. **Evaluator** — runs candidate programs against test instances
   and returns a fitness score.

The inner loop:

1. **Programs Database** holds evaluated candidates organized into
   *islands* (independently-evolving subpopulations) to preserve
   diversity. Periodically the worst islands are reset and reseeded
   from the best.
2. **Prompt Sampler** picks one or more high-scoring programs from
   the current island, formats them as in-context exemplars (often
   labeled `v0`, `v1` so the LLM sees the improvement trajectory),
   and emits a prompt asking for a `v2`.
3. **LLM** (PaLM 2 in the paper; later variants used Codey, Gemini
   1.5 Flash) writes a candidate function body.
4. **Evaluator** runs the candidate and computes fitness.
5. **Programs Database** decides whether to admit; admitted
   candidates can seed future prompts.

Programs are represented as **single function bodies** in Python — a
strict scope choice that defines FunSearch's identity and that
AlphaEvolve later lifts to whole-file evolution.

## Results

- **Cap-set problem** (extremal combinatorics: largest subset of
  $(\mathbb{Z}/3\mathbb{Z})^n$ containing no three collinear points):
  largest known cap sets in $n=8$ and improved asymptotic lower
  bounds; the **largest advance in 20 years**.
- **Online bin packing**: discovered a heuristic beating "best-fit"
  and "first-fit" baselines on standard benchmark distributions
  (Weibull, etc.); transfers cleanly to production-style data center
  job placement.
- Solutions are short, human-readable Python — the cap-set construction
  fits on one page and is interpretable enough that mathematician
  Jordan Ellenberg (co-author) used it as a starting point for further
  human-driven analysis.

## Critique / open questions

- **Evaluator-defined problem.** FunSearch only works where you can
  write a fast, unambiguous scoring function. Research tasks where
  evaluation is itself the open problem (most of empirical ML
  research) are out of scope unless the evaluator can be approximated.
- **No comparative ablation on islands.** The paper claims islands
  matter for diversity but doesn't isolate the contribution vs.
  flat-population evolution at matched compute.
- **Single-function scope is also a ceiling.** Many useful algorithms
  cannot be expressed as a single function calling into a fixed
  skeleton. AlphaEvolve ([[literature/papers/novikov2025alphaevolve]])
  is the explicit scope extension; the cost is that whole-file
  search is harder to direct and harder to interpret.
- **LLM dependency is modest.** Later work used Gemini 1.5 Flash with
  no specialized code training; the architecture is what's
  load-bearing, not the model.

## Follow-up

**Relevance:** 4 — foundational attestation for `evolutionary-expansion`
in the LLM-program-search lineage and the second attestation (with
AlphaEvolve) that seeds two new concepts: `programmable-evaluator-oracle`
and `evolutionary-search-grain`. Scored 4 rather than 5 because the
specific architecture is now superseded by AlphaEvolve for the
research-agent use case; FunSearch's value is canonical reference
and contrast.

**Connections:**
- Direct architectural lineage to AlphaEvolve
  ([[literature/papers/novikov2025alphaevolve]]) — same four
  subsystems (programs database, prompt sampler, LLM, evaluator),
  scope lifted from function to whole file, model upgraded from
  PaLM 2 to Gemini Flash + Pro ensemble.
- Same evolutionary-with-evaluator principle attests in FM Agent
  ([[literature/papers/li2025fm]]) but applied to ML-engineering
  proposals rather than pure-algorithm search.
- The skeleton/function split is a precursor to today's "agent
  scaffold + LLM judgment" architectures, and the human-written
  skeleton is the analogue of a project's CLAUDE.md + skill
  contracts: a fixed program shell the LLM is constrained to fill.

**Open angles for downstream experiments:**
- The cost of one cap-set-class FunSearch run (compute, tokens, wall
  time) is not surfaced in the paper. Replicating bin packing on a
  modern open model would calibrate whether the paradigm is
  approachable at small-lab scale.
- Whether islands meaningfully outperform flat populations at modern
  LLM mutation quality is an empirical hole worth a targeted study.
