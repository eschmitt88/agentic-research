---
kind: concept
name: "evolutionary-expansion"
status: experimental
added: "2026-04-24"
source_papers:
  - li2025fm
  - nam2025mle
  - novikov2025alphaevolve
  - romeraparedes2024funsearch
  - qu2026coral
sources:
  - "[[literature/papers/assumpcao2025codeevolve]]"
  - "[[literature/papers/du2026cvevolve]]"
  - "[[literature/papers/du2026mlevolve]]"
  - "[[literature/papers/li2026apex]]"
  - "[[literature/papers/liu2026oragent]]"
  - "[[literature/papers/pelleriti2026evolutionary]]"
  - "[[literature/papers/pepe2026agentic]]"
  - "[[literature/papers/ye2026evolutionary]]"
  - "[[literature/papers/li2025fm]]"
  - "[[literature/papers/nam2025mle]]"
  - "[[literature/papers/novikov2025alphaevolve]]"
  - "[[literature/papers/romeraparedes2024funsearch]]"
  - "[[literature/papers/qu2026coral]]"
used_by: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/pass-at-k]]"
related_experiments: []
tags: [search, evolution, mcts, proposals, expansion]
---

# evolutionary-expansion

## Definition

Proposal-space search that maintains a population of candidate
experiments, scores them by fitness, selects the strongest, and
mutates them to produce the next generation. Variants include MCTS
over an experiment tree and islands-based evolutionary sampling. The
unifying move is: do not pick one idea and commit; grow many in
parallel and let the fitness signal winnow them.

## Why it matters

FM Agent ([[literature/papers/li2025fm]]) builds its entire
autonomous pipeline around evolutionary sampling and reports
43.56% on MLE-bench (plus strong results in operations research,
GPU kernels, mathematics) with the same recipe across domains — the
generic search plus a swapped domain evaluator is what generalizes.

MLE-STAR ([[literature/papers/nam2025mle]]) achieves a related effect
via ablation-guided targeted refinement: rather than mutating whole
solutions, it identifies which component carries the most signal and
explores variants within that component. Same principle at finer
grain.

In both cases the losing alternative is flat single-proposal search
— commit to one design, run it, iterate linearly. The evolutionary
move expands the frontier earlier, so bad trajectories are cheaper
to abandon.

AlphaEvolve ([[literature/papers/novikov2025alphaevolve]]) is the
canonical attestation of this pattern outside ML engineering: an
evolutionary loop with Gemini Flash + Pro as the mutation operators,
a programs database as the population manager, and user-supplied
evaluators as the fitness signal. Same recipe, applied to pure
mathematics (Strassen-beating matrix multiplication), datacenter
scheduling, GPU kernels, and Verilog hardware design. The agent
architecture is domain-agnostic; the evaluator carries the domain.
This generalizes the FM Agent observation — evolutionary sampling
plus a swapped evaluator is the unit of cross-domain transfer.

FunSearch ([[literature/papers/romeraparedes2024funsearch]]) is the
foundational ancestor of this lineage (Nature 2024): the first
LLM-driven evolutionary program-search system, with an islands-based
programs database for diversity, function-grain mutation, and an
evaluator that doubles as the hallucination filter. The four-subsystem
architecture (programs database, prompt sampler, LLM, evaluator) that
AlphaEvolve later scales to whole-file evolution originates here.
The lineage shows the search pattern is robust across model
generations — PaLM 2 (FunSearch) through Gemini Pro (AlphaEvolve)
through Opus-class agents (FM Agent), the same architecture
produces the same kind of result.

## Implementation guidance

1. **`/expand` is the natural entry point.** Given a proposal,
   produce N child proposals that share the hypothesis but differ
   in approach (architecture, training regime, feature plan,
   evaluation strategy). Each child records `parent:` in
   frontmatter so the tree is reconstructable.

2. **Fitness = validation metric, not final metric.** Evolutionary
   search is inside the HCE search loop; selection signal is
   `metrics.json`, never `final_metrics.json`.

3. **Cap the branching factor per level.** N=3 is a reasonable
   default for `/expand`; above N=5 the context cost of evaluating
   children exceeds the diversity payoff for most budgets.

4. **Mutation operators are explicit.** A child proposal states
   what it changed from the parent and why in its `rationale:`
   field. Opaque mutation ("try a different approach") is not
   useful signal later.

5. **Pruning cadence.** After each generation, lock in the top-k
   children (by validation metric) and discard the rest rather
   than hedging — the search budget for later generations needs
   the headroom.

## Open questions

- Mutation operators for ML-engineering proposals are underspecified
  in both source papers. The right set probably depends on task —
  feature-heavy tasks need feature-mutation operators; architecture-
  heavy tasks need layer-swap operators.
- Whether MCTS over experiment trees meaningfully outperforms flat
  evolutionary sampling at realistic compute budgets is an open
  empirical question — candidate for a downstream experiment.
