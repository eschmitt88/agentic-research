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
  - "[[literature/papers/oved2026llm]]"
  - "[[literature/papers/chen2026auditing]]"
  - "[[literature/papers/chen2026cads]]"
  - "[[literature/papers/kim2026elicitation]]"
  - "[[literature/papers/wang2026rsihub]]"
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

## Against fresh sampling at equal tokens, evolution paid on one of five tasks for one model and on all three for another (2026-10-26)

The definition's "losing alternative" is flat single-proposal search. The
cheapest version of it is independent sampling: draw programs from the
task description alone, with no archive, and keep the best.
[[literature/papers/oved2026llm]] compares OpenEvolve, EvoX and AdaEvolve
with that baseline at equal tokens: 40 seeds × 200 iterations per cell,
exact order statistics, and exploited winners removed by re-execution.

On gpt-5-mini, only Heilbronn rewards search, and it does so for all three
strategies (equal-token ratio 1.23–1.28, CI lower bounds ≥ 1.17). Circle
packing, Cloudcast and transaction scheduling sit at 0.99–1.05, and all
three PRISM pairs are at 1.00. Haiku 4.5 on the two math tasks gives the
same ordering (Heilbronn 1.17–1.26, circle packing 1.01–1.07). On
gemini-2.5-flash, the three systems tasks all reward search: Cloudcast
1.53–1.59, transaction scheduling 1.15–1.25, PRISM 1.06–1.15. So whether
the population machinery pays depends on the task, as
[[literature/papers/zou2026fmlbench]] found for topology, and the size of
the payoff depends on the model. The authors' advice, "check whether a
task rewards search before choosing a searcher", has a cheap form: a
fresh-sample arm at the same token budget, priced from the run's own logs.

This bears on guidance 6–7. [[literature/papers/xing2026compute]] budgets
C = T·N in LLM calls. Here a call costs 12.4k–28.1k tokens, and the cost
rises with the archive carried in the prompt. A depth-heavy split is
therefore dearer per call than a breadth-heavy one, so a call-budget
optimum may shift once it is priced in tokens (our inference).

Caveats:
- The window is capped at about a fifth of each evolution run's tokens
  (28–48M), and evolution is still rising at the cap. The margins are
  lower bounds on what full runs would show.
- Ratios are of raw scores. On circle packing, a ratio of 1.037 is the gap
  between OpenEvolve and past the human record in
  [[literature/papers/ishibashi2026effective]].
- The operators are stateless single-call proposals. Search scores only,
  with no held-out workload.

## An evolving-vs-frozen arm cannot show the improver got better; a same-start swap can, and 4 of 26 RSI papers ran one (2026-10-26)

he2026does found, in DGM's logs, that the self-edit vs fixed-editor gap
is a viability effect. [[literature/papers/chen2026auditing]] gives the
design reason a gap of that kind could not show more.

- **Proposition 3, diagonal non-identification.** Suppose the improver is
  a function of the solver state it acts on, as when a loop edits its
  improver only through its own retained changes. Then any number of runs,
  from any starting states, observes each state with only one improver.
  The data are equally well explained by a kernel in which the improver
  has no effect, so "the later improver is better" is not identified.
- **A two-state example.** Two states both score zero. Any improver gains
  0.1 from A and 0.8 from B. An earlier improver seen at A and a later one
  seen at B differ by 0.7, while the true improver contrast is 0 at each
  state. Matching scores does not match states.
- **What identifies the gain.** An off-diagonal observation, most directly
  a swap test. Run the earlier and the later improver from *identical
  starting artifacts* (or paired over a declared set of starts), at the
  same budget, and audit the successors.
- **Evolving-vs-frozen answers a different question.** It compares a run
  where the improver may change with one where it is fixed, as in DGM and
  Hyperagents. Its arms visit different states, so it measures the value
  of letting the improver change over a run. It does not show that a
  later improver beats an earlier one.

The census counts how often the design is used. Of 26 strong-label studies
claiming a better improver:
- 23 report some related comparison, and 22 of those favour the proposal;
- 4 are same-start at a matched budget (STOP, MetaRSI, Sivia, MASS);
- 3 are same-start without a stated budget match;
- 7 are evolving-vs-frozen; 9 other; 3 none.

A strict rule moved two comparisons out of same-start, because their arms
carried solver artifacts across: SwarmRSI's inherited research state, and
Hyperagents' transfer, which moves the task agent with the meta agent.
Results depend on conditions: STOP's improved improver helps with GPT-4
but not GPT-3.5 or Mixtral.

For this concept's own sources:
- srikanth2026recursive's ignition test is coded same-start and matched,
  one of the four correctly designed tests in the census. It returned
  0.780 vs 0.782, inconclusive.
- The paper's null-loop proposition gives the acceptance count to expect
  from noise alone. With no true gain, i.i.d. errors and accept-if-better,
  N proposals yield Σ_{i=2}^{N+1} 1/i acceptances: 2.9 at N = 25, 4.2 at
  N = 99. AIDE² accepted 7 rewrites in 100 nodes. Its argmax-on-mean rule
  does not meet those assumptions, so this is a scale reference, not a
  test (our arithmetic).

Rule: an evolving-vs-frozen win may be reported as "letting the improver
change helped this run". "The improver improved" needs a same-start swap
at matched budget, with successors audited on held-out tasks and the
contrast's sign and uncertainty reported. An inconclusive swap is a tested
claim, not a missing one.

Caveats:
- Prop. 3 assumes a deterministic, separable improver. Random updates or
  different histories that reach the same state supply some overlap.
- Census counts come from an independent blind coder, but the claimant
  group is small (4/26, CI 6–34%).

## The baseline to beat is a competent non-LLM prior, and a random fallback both flatters the LLM and is displaced by it (2026-10-27)

oved2026llm's fresh-sample arm asks whether a task rewards *search*.
[[literature/papers/chen2026cads]] asks the matching question about the
*proposer*: is it worth more than a cheap non-LLM rule in the same loop?
The setting is a simulated gene screen with 12 rounds of 40 reads, 7–8B
open-weight proposers and 20 paired seeds. Results are certified true hits
per campaign:

| Proposer | Certified hits (20 seeds) | Net over random fill |
|---|---|---|
| Random fill | 2.60 | – |
| LLMs | 2.95–5.65 | +0.35 to +3.05 |
| Pan-essentiality prior | 12.65 | +10.05 [8.69, 11.41] |

The prior is each gene's mean effect across other cell lines, fixed before
any read. On 50 fresh seeds at a matched quota, it certified 16.12 against
the best LLM row's 5.08.

Two consequences for the losing-alternative arm in the definition:
- **Random is the floor, not the baseline.** Every LLM arm beat random
  fill at matched quota (+1.84 to +2.70, unadjusted). A report that
  compared only with random would have credited the LLM. A domain
  heuristic of the kind a practitioner would write first beat all of them
  by about 3×. The concurrent Gupta et al. 2025 result, which the paper
  cites, reaches the same conclusion for LLM-guided Bayesian optimization.
- **An LLM that fills some slots displaces the fallback that fills the
  rest.** With the thin prompt, Qwen filled 58 of 260 slots and originated
  2.15 hits. The fallback, squeezed to 202 slots, found 0.80 instead of
  2.60. Counting "LLM-originated winners" in an archive therefore
  overstates the LLM by whatever the mutation or random-restart path would
  have found in those slots.

Our inference: in an ML-research search, the analogue of the prior is the
obvious strong default, such as a tuned GBDT, the published recipe, or
the last champion re-run with a standard sweep. It belongs in the
comparison next to oved2026llm's fresh-sample arm, at equal reads or
tokens.

Caveats:
- Semi-synthetic, one simulator, small open models.
- Pretraining leakage is untested. A frontier model's proposals here could
  be recall of the same essentiality that the prior encodes.
- The prior uses other cell lines from the same reference resource. It is
  truth-isolated for the target line, not domain-naive.

## Fitness on validation pass@1 selects reliability, and re-weighting the task mixture did not buy reach (2026-10-27)

Guidance 2 and 5 rank candidates on the validation metric.
[[literature/papers/kim2026elicitation]] shows what that metric selects
when it is pass@1. The setting is Meta-Harness with an Opus 5 proposer,
20 iterations × ≤2 proposals, and solver weights fixed.
- **All 40 candidates of one HotpotQA run (Qwen3-8B)** stay below the
  Qwen3-32B baseline on hard questions at every k up to 128. The selected
  harness has the best hard pass@1 of the 40 but not the best hard
  pass@128.
- **Re-optimizing on a hard-focused task mixture** raised hard pass@1 from
  56.88% to 59.28%, against 32B's 59.51%. Hard pass@128 stayed at 73.17%
  (30/41 questions), against 78.05% for 32B. It was one run.
- **Static tasks.** The gain concentrates where the baseline already
  succeeds: 84% of LawBench's pass@1 gain is on tasks H0 already reached.
  The authors liken this to RL's Matthew effect.
- **Interactive tasks** did gain reach over the initial harness: AppWorld
  hard went from 52/63 to 56–58/63 at k = 128. So the loop is not purely
  eliciting. It just did not pass the stronger reference's reach.

For a population loop whose purpose is reach, a pass@1 fitness is the
wrong selection target. The paper proposes rewarding "validated successes
on previously uncovered hard tasks" and retaining "candidates with
complementary successes", but does not test either. Our inference: the
cheap version is a selection score that counts validation tasks a
candidate solves which no archived candidate solves, at a stated k. That is
a coverage-novelty term next to the mean, as in quality-diversity
archives. It has no measured evidence in this graph yet.

Caveats:
- One optimizer, two solver families.
- The main runs were selected under scorers the paper later replaced. The
  aligned-scorer rerun keeps the hard-task pattern.
- Hard-subset gaps are 1–2 questions with no intervals.
- QA, classification and app tasks, not ML-research loops.

## Four evolution policies from one seed under one evaluator: no ranking holds across agent or benchmark, at one run per cell (2026-10-27)

[[literature/papers/wang2026rsihub]] writes AHE, Hyperagents, A-Evolve and
GEPA as recipes over shared operators. The seed artifact, target and
mutator model (GPT-5.4), evaluator, splits and generation cap (≤10, one
child per generation) are all held fixed. That makes it a same-start
comparison of *methods*. [[literature/papers/chen2026auditing]]'s swap
test compares earlier and later *improvers*, which is a different
question, but it needs the same infrastructure.

- The best method changes in every cell. On Terminal Bench 2 it is
  Hyperagents and A-Evolve for MiniSWE (+13.5 overall) and AHE for Codex
  (+2.2). On τ³ Banking it is Hyperagents for MiniSWE (+5.2, with a sealed
  regression) and AHE for Codex (+9.3).
- The only cell where all four methods clearly gain held-out is TB2
  MiniSWE, at +4 to +5 sealed tasks of 20. There the seed draw is near its
  own mean. Codex TB2 sealed moves 0 to +2.
- Budget is matched by generation count only. GEPA adds a minibatch
  Validate, Mutate may trial on train tasks, and no token or dollar totals
  are reported. By [[literature/papers/oved2026llm]]'s standard the arms
  are not compute-matched. There is also no fresh-sampling arm on the seed
  ([[literature/papers/wang2026rethinking]]).

Our inference: with one run per cell and a single-draw baseline (see
[[concepts/hce-evaluation]]), "rankings vary" cannot be separated from
"the cells are inside the noise". The value here is the harness: a
one-operator swap at a fixed seed and evaluator is a config change. The
evidence about which policy expands better is close to nil.

Caveats:
- The four recipes are adaptations of "selected policies", not
  reproductions of the original systems.
- The agent-driven mode (an outer agent rewriting the recipe) is described
  but not evaluated.

## Open questions

- Mutation operators for ML-engineering proposals are underspecified
  in both source papers. The right set probably depends on task —
  feature-heavy tasks need feature-mutation operators; architecture-
  heavy tasks need layer-swap operators.
- Whether MCTS over experiment trees meaningfully outperforms flat
  evolutionary sampling at realistic compute budgets is an open
  empirical question — candidate for a downstream experiment.
