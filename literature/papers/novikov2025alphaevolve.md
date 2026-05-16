---
kind: paper
title: "AlphaEvolve: A coding agent for scientific and algorithmic discovery"
authors:
  - Alexander Novikov
  - Ngân Vũ
  - Marvin Eisenberger
  - Emilien Dupont
  - Po-Sen Huang
  - Adam Zsolt Wagner
  - Sergey Shirobokov
  - Borislav Kozlovskii
  - Francisco J. R. Ruiz
  - Abbas Mehrabian
  - M. Pawan Kumar
  - Abigail See
  - Swarat Chaudhuri
  - George Holland
  - Alex Davies
  - Sebastian Nowozin
  - Pushmeet Kohli
  - Matej Balog
year: 2025
venue: "arXiv:2506.13131"
url: "https://arxiv.org/abs/2506.13131"
source: "raw/papers/novikov2025alphaevolve.pdf"
added: "2026-05-16"
relevance: 5
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hybrid-model-backends]]"
tags: [evolutionary-search, coding-agent, deepmind, gemini, automated-discovery, evaluator-fitness, funsearch-successor]
---

# AlphaEvolve: A coding agent for scientific and algorithmic discovery

## TL;DR

AlphaEvolve is DeepMind's evolutionary coding agent that orchestrates a
pipeline of Gemini models against automated evaluators to iteratively
improve whole programs. It produced the first improvement to Strassen's
algorithm for 4×4 complex matrix multiplication in 56 years (48 scalar
multiplications), recovered ~0.7% of Google's worldwide compute via a
new Borg scheduling heuristic, sped a Gemini training kernel by 23%
(1% total training-time reduction), and improved FlashAttention by up
to 32.5%.

## Claims

- An evolutionary loop with LLM-generated mutations and programmable
  evaluators can discover **novel, provably-correct algorithms** at
  the frontier of mathematics and computer science.
- The agent operates on **whole code files**, not single functions —
  explicitly positioned as the scope extension of FunSearch
  (Romera-Paredes et al., 2023).
- A **fast-model + strong-model ensemble** (Gemini Flash for breadth,
  Gemini Pro for depth) outperforms either alone — the fast model
  "maximizes the breadth of ideas explored," the strong model
  "provides critical depth."
- The same architecture **transfers across domains** with no change
  beyond the evaluator: pure math, hardware design (Verilog), GPU
  kernel optimization, datacenter scheduling, and circuit design.
- **Evaluator quality is the binding constraint.** The agent can only
  optimize what the evaluator measures, so framing the problem as a
  scoring function is the dominant work.

## Methods

Four named subsystems:

1. **Prompt Sampler** — assembles prompts by drawing high-performing
   programs from the programs database and composing them as in-context
   exemplars for the next generation. The prompt is the
   information-bus between population history and the next mutation.
2. **Language Model Ensemble** — Gemini Flash + Gemini Pro running in
   tandem. Flash supplies high-volume cheap proposals; Pro supplies
   fewer but higher-quality refinements. Both write directly to the
   program-file representation.
3. **Evaluators** — user-supplied programs that **verify, run, and
   score** a candidate. Multi-objective scoring is supported. The
   evaluator is the only fitness signal.
4. **Programs Database** — stores all evaluated candidates with their
   scores; implements the evolutionary-algorithm logic (selection,
   ancestry, diversity). Acts as the population manager.

The inner loop:

1. Programs Database selects candidates.
2. Prompt Sampler builds a prompt containing those candidates plus
   problem context.
3. LLM Ensemble proposes a mutated program (full file or diff).
4. Evaluator runs the mutated program against the scoring function.
5. Programs Database stores the result; selection biases toward
   high-fitness ancestors for the next iteration.

Programs are represented as executable code files in whatever language
the evaluator runs — Python, C++, Verilog, JAX, CUDA. The agent is not
tied to a fixed AST or DSL.

## Results

- **Matrix multiplication.** Found a procedure for 4×4 complex matrix
  multiplication using **48 scalar multiplications**, beating
  Strassen's 1969 bound after 56 years.
- **Borg scheduler.** Discovered a heuristic that recovers ~0.7% of
  Google's worldwide compute resources continuously — directly
  deployed in production.
- **TPU hardware design.** Proposed a Verilog modification removing
  unnecessary bits in a matrix-multiplication arithmetic circuit;
  shipping in an upcoming TPU.
- **Gemini training.** Optimized matrix-multiplication kernel; 23%
  speedup on that kernel translated to 1% reduction in Gemini's
  end-to-end training time.
- **FlashAttention.** Up to **32.5% speedup** on the FlashAttention
  kernel.
- **Kissing-number problem.** New lower bound in 11 dimensions —
  a non-algorithmic mathematical discovery.

## Critique / open questions

- **Evaluator design is the moat.** AlphaEvolve only works when the
  problem admits a precise programmable scoring function. The paper
  shows the agent on tasks where this is feasible; whether it can be
  adapted to fuzzy research tasks (where the scoring function is
  itself unclear) is open. This is the same boundary
  [[concepts/hce-evaluation]] enforces structurally — separate the
  signal from the held-out truth.
- **No published architecture diagram.** The blog post is the most
  detailed public account of the four-subsystem split. The paper
  itself (per the abstract) emphasizes results over architecture;
  deeper ablations on prompt-sampling strategy, selection policy,
  and Flash:Pro ratio are not in the public material.
- **Closed system.** Gemini Flash + Pro are the LLMs; no public code
  or weights for the orchestrator. Reproduction requires reimplementing
  the four subsystems against a different model family.
- **Compute footprint unstated.** The Strassen result and FlashAttention
  speedup each likely consumed substantial compute — wall-time and
  token cost per discovery would help calibrate when this approach is
  economical vs. human algorithm design.

## Follow-up

**Relevance:** 5 — canonical attestation for [[concepts/evolutionary-expansion]]
(previously sourced only from FM Agent + MLE-STAR; AlphaEvolve is the
flagship example) and sixth attestation for [[concepts/hybrid-model-backends]]
(Gemini Flash + Pro = explore/exploit at the model-tier level, a
distinct framing from ideator/implementer role split).

**Concepts seeded on this ingest** (jointly with FunSearch
[[literature/papers/romeraparedes2024funsearch]]):

- [[concepts/programmable-evaluator-oracle]] — the discipline that
  the evaluator function *defines* what the agent can optimize, and
  framing the problem as a scoring function is the dominant work.
  Three attestations: FunSearch + AlphaEvolve + FM Agent.
- [[concepts/evolutionary-search-grain]] — programs-as-whole-files
  (AlphaEvolve) vs programs-as-single-functions (FunSearch). The
  grain is a deliberate design dimension with consequences for
  LLM strength, interpretability, and solution scope.

**Candidate concept still deferred:**

- **`prompt-as-population-bus`** — the Prompt Sampler injects past
  best programs into the LLM context as in-context exemplars,
  making the prompt itself the carrier between generations. Distinct
  from typical evolutionary algorithms where genetic information
  lives in a structured population. FunSearch attests this too, but
  the mechanism is largely implicit in evolutionary-expansion's
  population dynamics — wait for a third explicit attestation
  before seeding.

**Connections to track:**
- The Flash:Pro ratio question is the **explore-exploit budget split**
  at the model-tier level. Worth a downstream experiment if a project
  ever needs to schedule a multi-model agent: when does Flash earn its
  keep vs. just running Pro twice?
- AlphaEvolve's success rests on an evaluator that's cheap, reliable,
  and unambiguous. Most ML-research tasks fail at least one of those —
  this is why MLE-bench has Kaggle as the grader. The "good evaluator"
  prerequisite is the hidden cost of applying this paradigm to research
  agents broadly.
