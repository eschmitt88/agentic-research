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
  - "[[literature/papers/zhu2026bad]]"
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
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/liu2026evolvemem]]"
  - "[[literature/papers/gurkan2026mutation]]"
  - "[[literature/papers/xing2026compute]]"
  - "[[literature/papers/yoon2026arcticswarm]]"
  - "[[literature/papers/li2026autorecsys]]"
  - "[[literature/papers/min2026autonomous]]"
  - "[[literature/papers/srikanth2026recursive]]"
  - "[[literature/papers/wang2026rethinking]]"
  - "[[literature/papers/bobadillasuarez2026audit]]"
  - "[[literature/papers/xu2026conflictguide]]"
  - "[[literature/papers/hu2026winners]]"
  - "[[literature/papers/he2026does]]"
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

A caution on where the diversity actually comes from:
[[literature/papers/gurkan2026mutation]] shows the LLM mutation
operator alone converges into attractors (selection-free chains
revisit a handful of structural templates; classical GP mutation
does not), so the population machinery — islands, MAP-Elites,
novelty archives — is what supplies the variation the operator
doesn't. Expansion breadth N is only real breadth if the N children
escape the operator's attractor; phrasing the expansion prompt for
diversity, and checking the children actually differ structurally,
is load-bearing.

The archive machinery can also run in the *other* direction.
[[literature/papers/zhu2026bad]] maintains an archive that **prunes** the
population rather than diversifying it: each entry is a validity-checked
adversarial test, and a candidate is struck out if it loses more than a
tolerance of its advantage over the starting point under any archived test.
The trade is measured. After one test entered the archive, all three
round-two candidates — at +15.31%, +9.18% and +15.31% search-set gain —
became infeasible and the search fell back to its starting harness; round
three adopted a candidate worth only +4.08%. That eleven-point sacrifice is
what transferred: on a separate corpus the constrained system "scores 30.37%,
compared with 26.30% for HarnessCompass" and 27.04% for the unoptimized
baseline. The adversary cost 2.47x as many search attempts and 2.43x as many
tokens.

This bears on any loop here that selects proposals against a fitness signal —
including `/elevate`, which ranks proposals with nothing sealed and no
adversary generating constraints.

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

6. **Allocation is its own lever — treat it as a design choice, not
   a leftover.** [[literature/papers/xing2026compute]] holds model,
   prompt, and evaluator fixed and gets +12.3% mean fitness purely by
   changing how a fixed budget of LLM calls is distributed: "not a
   model improvement, nor a prompt improvement, but an allocation
   improvement." Under a fixed budget `C = T·N`, the depth–breadth
   split is the only remaining degree of freedom, and where it lands
   on the fitness surface is task-dependent — their bilinear fit
   `log(1−V) = β₀ + a·log T + b·log N + c·log T·log N` has an
   interaction term `c` that alone decides the regime: |c|≈0 means the
   optimum sits at the all-depth corner (refine one line deeply),
   large negative `c` means a balanced interior optimum with plateau
   half-width ∝ 1/√|c|. Guidance 3's "N=3, above N=5 the payoff
   fades" is a corner-regime default; it is wrong for interior-ridge
   tasks, and nothing tells you which you have in advance.

7. **Allocate *between* trajectories, not just within one.** The
   sharper finding is that fixed within-run allocation cannot remove
   cross-run heterogeneity: identical configurations produce a
   *distribution*, with some seeds stagnating at low fitness forever.
   xing2026compute's BaSE treats K parallel trajectories as bandit
   arms and routes each next call to the most promising, which
   decomposes cleanly into a **pool effect** (run K trajectories,
   keep the best) and an **allocation effect** (adaptively route
   budget) — both measured to survive independently. Gains concentrate
   exactly where guidance 5's greedy pruning is most dangerous: on
   hard, high-variance tasks. On Heilbronn Triangle, UCB clears
   fitness 0.70 within 60 generations while greedy and two island
   protocols never reach it at all inside 512 calls. Moderate pools
   (K ∈ {5,10,20}) win; K=50 dilutes refinement.

   Caveat before importing: their arms share a seed program and never
   exchange information, so this is not a replacement for island
   migration — it is a scheduler on top of whatever protocol runs
   inside each arm, and the paper shows it composes with OpenEvolve,
   CodeEvolve, and ShinkaEvolve parent sampling rather than replacing
   them.

8. **Allocation cannot manufacture signal.** The same paper's negative
   result is the boundary condition: model capability and prompt
   jointly set a ceiling, and allocation only governs how efficiently
   a run approaches it. On Llama-3.1-8B/HT the best allocator reaches
   0.4387 while Qwen3-8B reaches 0.8736 on the same task — an order of
   magnitude of headroom that no scheduler recovers. Where a model
   fails to cross the capability threshold, depth gains are
   statistically indistinguishable from selection noise. Fix
   capability first, then spend effort on allocation.

## Pointed at the searcher itself: what the loop converges on (2026-09-29)

[[literature/papers/srikanth2026recursive]] (AIDE²) makes the population
member a whole research agent. An outer loop proposes rewrites of the
agent's harness code, grades each rewrite by running it under a fixed dollar
budget on a task suite, and keeps the argmax. Over 100 nodes it accepted
seven rewrites. Two results bear on this concept.

- **Evolution over search policies re-derives guidance 7 and FML-Bench's
  stall rule.** The final agent runs UCB1 over five drafting-strategy arms
  and forks the global best under a new arm every five steps. The first is
  xing2026compute's bandit allocation. The second is close to
  zou2026fmlbench's AdaptiveSearch (greedy until a stall, then widen).
  Independent convergence from a blind search is weak but real evidence that
  these are the right defaults. It also means little here is new.
- **A negative list, graded under a fixed budget.** Among rejected proposals
  (pooled over three runs), island migration scored −0.021, pairwise
  LLM-judge tournaments −0.090 and majority-vote ensembles up to −0.031.
  Ensembling "was explored … but never retained", and the loop's own analyses
  noted it "consumes budget that could otherwise fund additional search
  steps." That backs guidance 5 and 6: at a fixed budget, depth-buying
  changes beat breadth-for-robustness changes.

**The recursive claim is unproven.** When a discovered agent drives the
outer loop (the "ignition test"), endpoints are 0.780 vs 0.782 for the
human-built driver over 3 seeds, which the authors call "inconclusive." And
the argmax keep rule has no noise band. In this setting a false accept does
more than crown the wrong champion. It becomes the code every later proposal
edits, so noise compounds down the lineage (see the noise-floor discussion
in [[concepts/budget-as-ceiling]]).

**The missing arm in AIDE².** [[literature/papers/wang2026rethinking]] says the
comparator for any harness-level search is the same budget spent re-running
the seed harness. On Terminal-Bench, neither per-task nor cross-task harness
evolution beat that arm by more than noise, and its held-out gain was exactly
+0.0. Per-task "Harness Scaling" does win on long-horizon games (ARC-AGI-3
RHAE 43.3 → 77.5), but there the edits are test-time adaptation scored
best-so-far on the same instance, not a reusable artifact. AIDE²'s
0.703 → 0.778 has no matched re-run arm.

[[literature/papers/bobadillasuarez2026audit]] names the regime AIDE² occupies.
Rewriting the harness with frozen weights can grow the reachable edit set,
while the weights cap the eventual ceiling, so the loop should saturate with
jumps along the way. Both halves are assumptions, not measurements. The
governance point: in a scaffold-rewriting loop, log whether the tool or
verifier set grew, not just whether the weights changed.

**The cost of a G > 0 keep rule, seen at formal scale (2026-10-06).** In
[[literature/papers/xu2026conflictguide]] the scalar-only AutoResearch arm
keeps any proxy improvement with G > 0, from a single proxy evaluation.
On SNGP its retrained winners were worse than the unmodified reference in
2 of 3 rounds: CIFAR-100 NLL 1.0578 and 1.1777 vs 1.0434. This is the
lineage-compounding false accept described above, observed rather than
argued. The competing arm keeps directly only when G > 1e-3, and needs
probe corroboration below that. It did better, though the thresholded
rule was never tested without the probes.

## A population selected on validation inflates beyond the single-step curse (2026-10-12)

[[literature/papers/hu2026winners]] instruments GEPA (reflective
evolutionary prompt search with a Pareto pool) and MIPROv2 on a held-out
set the optimizers never see. The validation score GEPA selects on
overstated the returned program's held-out gain heavily at 16 validation
items: +38.8 vs +19.2 on TREC, and +18.8 vs +1.2 on Banking77, at 1000
metric calls. From a strong start, it reported +25.0 on both tasks for
real gains of 1.6 and 4.1. The single-step winner's-curse model
under-predicts GEPA's overstatement by a factor of 1.1–1.9. The authors
attribute the gap to the pool itself being selected on the validation
set, so selection compounds across the archive and not only at the final
pick.

The budget trade is the import for this concept. A small validation set
made each evaluation cheap: about 10× more reflections and a pool of 91
programs against 9. GEPA's held-out gain still did not grow with
n_val. MIPROv2, which mostly holds its proposal pool fixed, doubled its
TREC held-out gain going from 16 to 256 items (11.4 vs 5.8). Spending the
budget on more candidates per noisy evaluation bought archive size, not
transfer. A population method's reported best should be re-scored on
items outside its selection set before it is credited (see
[[concepts/hce-evaluation]]). Caveat: Qwen3.5-4B task model, 5 seeds, two
tasks, and the GEPA reported-gain measure was added after eight runs had
been seen.

## In DGM's logs, edits are not aimed at the failure shown, and selection over zero-mean edits reproduces the curves (2026-10-20)

[[literature/papers/he2026does]] tests the premise every self-modifying
population here relies on: that an edit written after seeing a failure is
aimed at that failure. DGM draws the failure shown at random from the
parent's failures, so the target and the parent's other failures are
exchangeable under the null, and an e-value stays valid however the archive
adapts. Over 145 eligible modifications in the six public DGM runs, 12
repaired their target against 16.4 expected (8.3% vs 11.6%; e-value 0.15,
where 20 is needed at 5%). Five of six runs point the wrong way. Power is
0.72 for a 6-point directed boost, so small effects are not excluded.

Two further results change how a population's rising best should be read:
- **Selection over zero-mean edits is sufficient for DGM's curves.** A
  simulation with mean-zero edit effects, each run's measured edit spread
  (about 6.4 points for DGM), its viability and DGM's own parent-sampling
  and archive rules gives a median best of 0.51 (5–95%: 0.37–0.65) against
  0.50 observed, 0.44 vs 0.39 for the fixed editor, and 0.24 vs 0.23 with
  no archive. This is a sufficiency check calibrated on the same runs, not
  proof that the edits had zero mean.
- **The editor's measurable contribution is viability, not repair.** Edit
  gain did not grow with parent quality (slope −0.15, CI −0.35 to +0.02)
  and fell 2.1 points per 10 generations. What separated self-editing from
  a fixed editor was how often an edit yielded a working agent: 49% vs 33%
  on SWE-bench, 75% vs 23% on Polyglot (interaction z = 4.2).

This is the population-level counterpart of hu2026winners' per-decision
winner's curse, and it gives srikanth2026recursive's inconclusive ignition
test a mechanism: in the one system with public logs, "self-improvement
compounding" is a survival-rate effect. Before crediting an evolved best to
directed search, compare it with a zero-mean-edit simulation under the same
selection rule, and log per edit whether the failure shown was repaired
against the parent's other failures.

Caveats:
- One system (o1 proposer, Claude 3.5 Sonnet implementer that never sees the
  failure), 80 iterations per run, single evaluations per agent.
- In the paper's controlled studies, where the editor sees the failed
  question and its answer, edits *were* directed (+24.5 points on a clean
  playbook). The null is about this pipeline, not about feedback in general.

## Open questions

- Mutation operators for ML-engineering proposals are underspecified
  in both source papers. The right set probably depends on task —
  feature-heavy tasks need feature-mutation operators; architecture-
  heavy tasks need layer-swap operators.
- Whether MCTS over experiment trees meaningfully outperforms flat
  evolutionary sampling at realistic compute budgets is an open
  empirical question — candidate for a downstream experiment.
