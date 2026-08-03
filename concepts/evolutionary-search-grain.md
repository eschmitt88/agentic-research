---
kind: concept
name: "evolutionary-search-grain"
status: seedling
added: "2026-05-16"
source_papers:
  - romeraparedes2024funsearch
  - novikov2025alphaevolve
sources:
  - "[[literature/papers/assumpcao2025codeevolve]]"
  - "[[literature/papers/du2026cvevolve]]"
  - "[[literature/papers/du2026mlevolve]]"
  - "[[literature/papers/li2026apex]]"
  - "[[literature/papers/liu2026oragent]]"
  - "[[literature/papers/pepe2026agentic]]"
  - "[[literature/papers/ye2026evolutionary]]"
  - "[[literature/papers/romeraparedes2024funsearch]]"
  - "[[literature/papers/novikov2025alphaevolve]]"
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/zou2026fmlbench]]"
  - "[[literature/papers/gurkan2026mutation]]"
used_by: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
related_experiments: []
tags: [evolutionary-search, mutation-unit, program-representation, search-grain]
---

# evolutionary-search-grain

## Definition

The **unit of mutation** in an LLM-driven evolutionary code-search
loop — the contiguous span of code the LLM is asked to rewrite each
generation. The grain is a deliberate design choice that trades off
search-space size, interpretability, ease of evaluation, and the
range of solutions the system can express.

## Why it matters

Two end-points of the spectrum are now well-attested:

- **Function-grain** (FunSearch,
  [[literature/papers/romeraparedes2024funsearch]]): the user writes
  a "skeleton" — the program backbone — and the LLM only edits one
  target function inside it. Pros: tiny search space, short readable
  outputs (Kolmogorov-compressible), evaluator can verify cheaply.
  Cons: bounded by what fits in a single function; the human-written
  skeleton is a hard ceiling on the solution structure.
- **Whole-file-grain** (AlphaEvolve,
  [[literature/papers/novikov2025alphaevolve]]): the LLM rewrites
  entire program files via diffs. Pros: can express algorithms
  FunSearch cannot, can refactor structure, can introduce new
  helpers. Cons: larger search space, less interpretable outputs,
  needs a stronger LLM ensemble (Gemini Flash + Pro vs. PaLM 2),
  evaluator must tolerate structural changes.

Intermediate grains exist in principle (multi-function, class-level,
module-level) but are not yet attested as deliberate design choices
in the literature reviewed here.

The grain choice cascades into the rest of the architecture:

- **Smaller grain → less LLM capability required.** FunSearch
  worked with PaLM 2 and later Gemini 1.5 Flash; whole-file work
  needs the Pro tier for the harder edits.
- **Larger grain → richer prompt construction.** The Prompt Sampler
  must include enough context for the LLM to understand
  cross-function relationships; this is why AlphaEvolve's prompt
  pipeline is more elaborate than FunSearch's exemplar-stacking.
- **Smaller grain → cleaner interpretability story.** FunSearch's
  outputs are short enough that a mathematician can read them and
  learn; whole-file outputs trade this for capability.

## Implementation guidance

1. **Pick grain to match the problem, not the model.** Algorithmic
   discovery on a well-bounded function (kernels, heuristics,
   constructions) → function-grain. Code-base optimization, multi-step
   pipelines, refactor-driven improvements → whole-file-grain.

2. **The skeleton is a hyperparameter.** At function-grain, the
   skeleton encodes a strong prior about solution structure.
   Iterating on the skeleton between runs (a meta-loop) is often
   higher leverage than running more generations under a fixed
   skeleton.

3. **Whole-file-grain needs better diff handling.** AlphaEvolve's
   mutations are described as direct file edits; a sloppy diff
   apply (whitespace, partial overlap) breaks the loop silently.
   Tooling for clean diffs is non-negotiable at this grain.

4. **Mixed-grain is plausible but unattested.** A pipeline that
   evolves a function inside a fixed file early and lifts to
   whole-file rewrites once the function has plateaued is intuitive
   but not yet shown in published work. Candidate for downstream
   experimentation.

## The operator, not just the grain

[[literature/papers/gurkan2026mutation]] adds an orthogonal axis this
concept had been assuming away: the mutation operator's *intrinsic
entropy*. Running LLM mutation chains with **no selection pressure**
(every valid mutation accepted), 87% of chains spend >93% of their
mutations revisiting already-seen structural forms — variation
collapses to terminal substitutions inside a frozen control-flow
template — while classical GP subtree mutation on the identical setup
keeps exploring (~143 unique structures vs. <20 for most LLM chains).
The bias is intrinsic to the LLM operator, prompt-sensitive
(semantically similar prompts swing exploration 25×), and
model-dependent (Claude Sonnet 4 collapsed to one structural form;
reasoning-enabled models explore far more). Consequence for grain
choice: a fine grain only buys its theoretical search-space if the
operator actually leaves its attractor — otherwise the effective
search space is the attractor, whatever the grain. Profile the
operator with cheap neutral chains before committing a budget to a
long search.

## Open questions

- **Where does grain actually matter empirically?** No paper isolates
  the grain choice with everything else held constant. A controlled
  comparison on a single problem at both grains would calibrate
  whether grain or model strength is the dominant factor for the
  reported AlphaEvolve > FunSearch results.
  *Partially answered at the adjacent axis:*
  [[literature/papers/zou2026fmlbench]] runs exactly this controlled
  design for search *topology* (greedy vs parallel-linear vs best-first
  tree vs MCTS vs evolutionary; same LLM, same code editor, same step
  budget, 18 tasks). Result: complexity does not predict performance —
  greedy hill-climbing ties the best tree search and principled MCTS
  ranks last — and which topology wins is governed by the task's
  improvement-opportunity density (greedy on dense landscapes, broad
  frontier-keeping search on sparse ones). The same methodology applied
  to grain is still open.
- **Can grain be chosen adaptively per generation?** A meta-controller
  that picks "edit this function" vs "rewrite this file" based on
  current population diversity would unify the two regimes.
  zou2026fmlbench validates the analogous move for topology: its
  AdaptiveSearch runs greedy as a *probe* and irreversibly switches to
  multi-branch exploration when validation improvement stalls for W
  consecutive steps — beating every fixed strategy on both dense and
  sparse task partitions. Stall-triggered escalation is the cheapest
  online signal that works; no prior knowledge of the task's opportunity
  structure is needed.
- **The interpretability cost of larger grain is asserted but not
  measured.** Mathematicians reading FunSearch outputs is a strong
  qualitative claim. Whether AlphaEvolve's whole-file outputs are
  still usable for human-AI collaboration — or whether they require
  a re-distillation step — is open.
