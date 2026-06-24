---
kind: moc
name: "autonomous-search-loop"
status: active
added: "2026-06-24"
concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/budget-as-ceiling]]"
tags: [moc, autonomous-search-loop, evolutionary-search, search, infrastructure, autonomy]
---

# The Autonomous Experiment-Search Loop

How does an autonomous research agent search a vast space of candidate
experiments — or programs — *productively*: expanding the frontier instead
of committing to one guess, keeping the hardware busy instead of idling
between trials, and stopping on a real ceiling instead of running forever?
This MoC collects the five concepts that, together, describe one machine:
the loop that proposes many candidates, scores them against ground truth,
and winnows. Where the sibling [[mocs/evaluation-integrity]] asks whether
the *signal* the loop reads is honest, and [[mocs/agent-architecture]] asks
how the agent is *built*, this MoC asks how the agent *searches* — the
strategy, the grain it mutates, the oracle it optimizes against, and the
two operational disciplines (parallelism and halting) that make a search
campaign runnable at all.

The lineage is unusually well-attested: FunSearch
([[literature/papers/romeraparedes2024funsearch]], Nature 2024) →
AlphaEvolve ([[literature/papers/novikov2025alphaevolve]]) → FM Agent
([[literature/papers/li2025fm]]) trace the same four-subsystem recipe
(programs database, prompt sampler, LLM, evaluator) across model
generations, and AIRA_2 ([[literature/papers/hambardzumyan2026aira]])
names two of this MoC's concerns — synchronous execution and unbounded
runs — among the dominant bottlenecks in research agents.

## The search strategy and its grain

The core move is to refuse single-shot commitment: grow a population of
candidates and let fitness winnow them.

- [[concepts/evolutionary-expansion]] — maintain a population of candidate
  experiments, score them, select the strongest, and mutate them into the
  next generation (MCTS over an experiment tree, or islands-based sampling).
  FM Agent ([[literature/papers/li2025fm]]) builds its whole pipeline on
  this and reports 43.56% on MLE-bench with the *same* recipe across ML
  engineering, operations research, GPU kernels, and mathematics; MLE-STAR
  ([[literature/papers/nam2025mle]]) is the same principle at finer grain
  (ablation-guided refinement of the highest-signal component). The losing
  alternative is flat single-proposal search — commit, run, iterate
  linearly — which abandons bad trajectories late and expensively.
- [[concepts/evolutionary-search-grain]] — the *unit of mutation*: the span
  of code the LLM rewrites each generation. Function-grain (FunSearch — a
  human skeleton, one edited function: tiny search space, cheap evaluation,
  weaker LLM suffices) versus whole-file-grain (AlphaEvolve — diff-based
  rewrites: richer solutions, larger space, needs a stronger model
  ensemble). The grain is a hyperparameter that cascades into model tier,
  prompt construction, and interpretability — pick it to match the problem,
  not the model.

## The oracle that defines what's discoverable

A search loop is only as good as the thing it optimizes against. In these
systems that thing is not part of the agent — it is the environment.

- [[concepts/programmable-evaluator-oracle]] — a user-supplied function that
  verifies, runs, and scores every candidate and serves as the loop's *only*
  ground truth. FunSearch uses it as a hallucination filter (invalid
  programs never enter the population, so the LLM is free to be noisy);
  AlphaEvolve makes it the *unit of domain transfer* (swap the evaluator,
  keep the core). The consequence is that agent competence collapses into
  evaluator quality — a subtly-buggy oracle steers the entire search wrong
  without tripping its own success signal. This concept also anchors
  [[mocs/evaluation-integrity]]: it is the seam where "how the loop searches"
  meets "whether the signal is honest," and it earns a place in both views.

## Running the loop productively — parallelism and halting

A search strategy and an oracle still need an execution substrate. These two
make a campaign actually runnable: one keeps the hardware busy, the other
guarantees it stops.

- [[concepts/async-worker-pool]] — parallelism comes from a pool of workers
  sharing a job queue, not from one worker running faster. AIRA_2
  ([[literature/papers/hambardzumyan2026aira]]) names synchronous execution
  as a dominant bottleneck: a single-worker loop is GPU-idle while it
  proposes, analyzes, and writes, and GPU-bound only while training; the
  pool overlaps those phases. FM Agent's headline depends on population
  sizes that are infeasible synchronously. The win survives even at pool
  size 2 on a single-GPU box, because it is about overlapping phases, not
  multi-GPU training.
- [[concepts/budget-as-ceiling]] — the loop halts because an *explicit*
  resource ceiling was hit (wall-hours, tokens, consecutive
  no-improvement, disk), not because a heuristic decided the work looked
  done. Stored in `budget.yaml`, read by every skill that spawns compute.
  FM Agent's distributed search could run indefinitely; ceiling discipline
  is what makes it practically runnable, and AIRA_2 reports 24h/72h results
  separately precisely because its wall ceilings predict behavior. Hitting
  a ceiling is information — "this direction is exhausted" — that feeds the
  next proposal.

## Open thread

The five compose into a single dependency chain: the **grain** sets how big
each mutation is, the **strategy** decides how the population evolves, the
**oracle** scores what survives, the **worker pool** sets how fast the
generations turn over, and the **ceiling** decides when to stop. Two of
these — the oracle and the ceiling — are also the loop's main failure
surfaces: a corrupt oracle optimizes the wrong target with full machinery
behind it, and a missing ceiling either runs forever (benign) or burns the
real budget silently (costly). The working hypothesis worth testing in this
project is whether grain and strategy are *separable* knobs or a coupled
pair — i.e., whether finer grain (MLE-STAR-style component refinement) is
just a different mutation operator inside the same evolutionary loop, or
whether it demands a different population manager and prompt sampler
altogether. AIRA_2's bottleneck list also implies the async-pool win and
the HCE win ([[mocs/evaluation-integrity]]) are *additive* — two of three
independent levers — which this MoC's concepts could be used to ablate.
