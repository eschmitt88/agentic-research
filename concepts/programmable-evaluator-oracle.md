---
kind: concept
name: "programmable-evaluator-oracle"
status: seedling
added: "2026-05-16"
source_papers:
  - romeraparedes2024funsearch
  - novikov2025alphaevolve
  - li2025fm
  - qu2026coral
sources:
  - "[[literature/papers/assumpcao2025codeevolve]]"
  - "[[literature/papers/du2026cvevolve]]"
  - "[[literature/papers/edwards2025rexbench]]"
  - "[[literature/papers/liu2026harnessing]]"
  - "[[literature/papers/liu2026oragent]]"
  - "[[literature/papers/ning2026code]]"
  - "[[literature/papers/pelleriti2026evolutionary]]"
  - "[[literature/papers/romeraparedes2024funsearch]]"
  - "[[literature/papers/novikov2025alphaevolve]]"
  - "[[literature/papers/li2025fm]]"
  - "[[literature/papers/qu2026coral]]"
  - "[[literature/papers/liu2026automedbench]]"
  - "[[literature/papers/xu2026researchclawbench]]"
  - "[[literature/papers/wu2026bayesian]]"
used_by: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hce-evaluation]]"
related_experiments: []
tags: [evaluation, fitness-signal, oracle, evolutionary-search, evaluator-defined-problem]
---

# programmable-evaluator-oracle

## Definition

A user-supplied programmable function that **verifies, runs, and
scores** every candidate produced by an autonomous search loop, and
serves as the loop's *only* ground truth. The evaluator is not a
component of the agent — it is the **environment** the agent
optimizes against, and it implicitly defines what the agent can
discover.

## Why it matters

In evolutionary LLM-program-search systems the evaluator is the
binding constraint:

- **FunSearch** ([[literature/papers/romeraparedes2024funsearch]])
  uses the evaluator to neutralize LLM hallucination — invalid
  programs never enter the population, so the LLM is free to be
  noisy. The evaluator is what makes correctness emerge from
  unreliable generation.
- **AlphaEvolve** ([[literature/papers/novikov2025alphaevolve]])
  scales the same pattern across domains (pure math, Verilog,
  CUDA kernels, datacenter scheduling) with **no architectural
  change** — only the evaluator swaps. This makes the evaluator
  the unit of domain transfer.
- **FM Agent** ([[literature/papers/li2025fm]]) reports the same
  cross-domain transfer property (ML engineering, operations
  research, GPU kernels, mathematics) under a generic search core
  with domain-specific evaluators. Same insight from a different
  research lineage.

The architectural consequence: **framing the problem as a scoring
function is the dominant up-front work**, and the agent's competence
collapses into the quality of the evaluator. A poor evaluator
produces a poor agent regardless of search budget or model strength.

This is also why the paradigm transfers awkwardly to open-ended ML
research — much of that work *is* the problem of figuring out what
to measure, and the evaluator cannot be written before the research
is done. The right boundary is captured by
[[concepts/hce-evaluation]]: structurally separate the search-signal
evaluator (`metrics.json`) from the held-out truth
(`final_metrics.json`), so the search loop cannot game its own
oracle.

## Implementation guidance

1. **The evaluator is part of the problem statement.** When a
   downstream project proposes an evolutionary-search experiment,
   the evaluator design goes in the proposal, not the implementation.
   It is the contract the search loop optimizes against.

2. **Evaluator latency caps iteration count.** Wall-clock per
   evaluation × population size × generations ≤ budget. Cheap
   evaluators are not a nice-to-have; they are the gate on whether
   the paradigm fits.

3. **Multi-objective is allowed; aggregation must be explicit.**
   AlphaEvolve evaluators report multiple scores; the programs
   database needs an aggregation rule (Pareto, weighted sum, primary
   + tie-breakers). Hidden aggregation in the evaluator is a
   debugging trap.

4. **Verify before scoring.** The evaluator's first job is to
   *reject* invalid candidates (syntax, type, runtime errors,
   constraint violations). Score-only evaluators that silently
   accept broken programs are a known failure mode.

5. **Evaluator-as-oracle and HCE coexist.** During search the
   evaluator is `metrics.json`. At chain end a separate final-scoring
   pass writes `final_metrics.json`. The evaluator is allowed to be
   gamed; the final scorer is not.

## Open questions

- The pattern works cleanly for problems with crisp objective
  functions (algorithm performance, mathematical bounds, kernel
  speed). Whether it can be adapted to fuzzy domains via *learned*
  evaluators (LLM-as-judge, reward models) without inheriting the
  noise-and-collapse problems of those judges is an open empirical
  question. Bayesian-Agent ([[literature/papers/wu2026bayesian]])
  offers a partial answer in the *skill-evolution* setting: instead of an
  LLM-as-judge, it accumulates a **feature-conditioned Bayesian
  posterior** over each skill's verified success/failure outcomes and
  lets that calibrated posterior — not a one-shot judge call — drive the
  rewrite policy. This sidesteps the noise-and-collapse trap by treating
  the fitness signal as *evidence accumulated over many runs* rather than
  a single fuzzy score, at the cost of needing enough trajectory evidence
  to calibrate (a cold-start problem the crisp-oracle systems don't have).
- Evaluator design itself is unautomated in current work. A
  meta-loop that searches over evaluator specifications given a
  high-level goal is a natural extension but no published system
  does it.
- How brittle the agent is to small evaluator bugs is undocumented.
  An evaluator with a subtle scoring error directs the entire
  search toward the wrong target — and the search's own success
  signal won't catch it. [[concepts/hce-evaluation]] is the
  structural defense.
