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
  - "[[literature/papers/ishibashi2026effective]]"
  - "[[literature/papers/moukpe2026deltaml]]"
  - "[[literature/papers/chi2026ai4ai]]"
  - "[[literature/papers/esakkiraja2026starharness]]"
  - "[[literature/papers/ge2026coverage]]"
  - "[[literature/papers/min2026autonomous]]"
  - "[[literature/papers/yu2026primescientist]]"
  - "[[literature/papers/xia2026rrsi]]"
  - "[[literature/papers/yan2026traceml]]"
  - "[[literature/papers/xu2026conflictguide]]"
  - "[[literature/papers/cai2026forkandflush]]"
  - "[[literature/papers/farahbakhsh2026agentdiscover]]"
  - "[[literature/papers/he2026beneath]]"
  - "[[literature/papers/wei2026scale]]"
  - "[[literature/papers/oved2026llm]]"
  - "[[literature/papers/zhong2026automating]]"
  - "[[literature/papers/chen2026auditing]]"
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

## How much to spend per individual, and the answer is "more than you think"

This concept frames the grain of *code* — how large a span the LLM rewrites
each generation. [[literature/papers/ishibashi2026effective]] measures an
orthogonal grain: the **effort budget per candidate**, under a fixed total.

Its finding runs against the evolutionary intuition. At an identical 40M
token budget on Circle Packing, harnesses that spent **54K–465K tokens per
algorithm and produced 87–742 candidates** beat one that spent ~25K tokens
each across ~1,500 candidates, by a wide margin (2.636 vs 2.541 on the same
model). "Scaling the quality of each individual is more budget-efficient
than scaling the number of evolutionary generations." The good harness
passed the weak one's *final* score at roughly one eighth of the budget.

The mechanism the paper points at is that a coding agent — stateful, able
to run and test its own candidate — is a categorically different mutation
operator than a stateless single-shot API call, which is what OpenEvolve
and comparable reimplementations use. So the depth is not "think longer
before emitting"; it is "iterate on this individual against the evaluator
before submitting it."

Two caveats. The comparison confounds four harness changes at once
(coding-agent integration, hack detection, worktree isolation, database
observation) and only the latter two are ablated, so *depth* is the
plausible explanation rather than the isolated one. And it is 2 runs per
condition on a single problem with a known optimum. Treat the direction as
a strong prior for setting per-candidate budgets, not as a tuned parameter.

## Spend per evaluation, not only per proposal (2026-10-13)

The section above prices *proposal* effort per candidate.
[[literature/papers/wei2026scale]] varies the other per-individual spend:
how many rollouts score each candidate and feed the next proposal. The
optimizer, a coding agent, reads full trajectories of B rollouts and makes
one bounded commit per round. With rounds fixed at 30, B = 100 produced the
only held-out gain over baseline (67% vs 51%). B = 5–20 stayed within noise
of baseline or overfit the training cases.

B bundles three levers that the paper does not separate:
- more failure evidence for the proposer;
- finer resolution on the promotion decision;
- 20× the rollout spend of B = 5, with no budget-matched arm.

It points the same way as [[literature/papers/ishibashi2026effective]]:
more spend per individual pays. But the candidate count was held at 30, so
this was extra spend, not a reallocation under a fixed total. Caveat: one
campaign per B, robotics simulation, and no rerun noise.

## In a research loop, the grain is the declared search space

[[literature/papers/min2026autonomous]] observes where agentic search stops
being narrow, on an industrial retrieval task whose search space covers
document representation, training-pair generation and architecture, not
just hyperparameters. The single-agent autoresearch loop could edit the
whole repository, yet without declared axes agents "default to optimizing
standard hyperparameters such as learning rate and number of training
epochs." Declaring the axes as pre-set variables got them off lr/epochs, at
the cost the FunSearch skeleton predicts: "agents primarily toggle
pre-defined candidate options rather than implementing novel code
modifications." Real wins landed *inside* the declared axes (2× oversampling
of ticket-to-ticket pairs, which the human systems lacked). Nothing crossed
to the system-composition level. Across 8 campaigns and 3 agents, no agent
attempted re-ranking, data augmentation or ensembling, the components that
separate its R@1 of 0.343 from the human SoTA's 0.380, **even when the
documentation named them**.

Two consequences for this concept. First, the edit span is not the
effective grain; the declared space is. Guidance 1's pairing of pipeline
problems with whole-file grain needs a qualifier: a larger grain *permits*
structural change but does not *produce* it. Second, the skeleton ceiling
isn't lifted by telling the agent what lies beyond it. Only putting the
component into the declared space might lift it, and the authors
deliberately didn't test that. A greedy keep/revert loop adds a second
barrier: a system-level change that first lowers the metric is reverted
before it can be tuned. Caveats: one task, one campaign per cell, no seeds;
the informed/uninformed gap on one agent (0.07) is larger than the
cross-model spread, so the paper's "model and informedness don't matter"
nulls are unresolved. Compare [[literature/papers/chi2026ai4ai]], where
more reasoning effort raised structural attempts from 8% to 64% on a
benchmark that scores only structural change. The two fit together if the
declared space dominates willingness.

[[literature/papers/yan2026traceml]] independently reproduces the
min2026autonomous pattern, using 430 paired human trajectories as the
reference. Both scaffolds settle into one band. Codex re-weights an ensemble
it never grows (78% of its ensemble edits), and MLEvolve mutates layers,
epochs and seeds in place. The cross-family moves (checkpoint swap,
pretrained-source swap, re-running unchanged code to verify) "all stay an
order of magnitude below the human rate". This holds for both a single loop
and an evolutionary tree on the same backend, so topology alone does not
widen the grain. Caveat: 3 MLEvolve runs and 6 informative Codex runs, all on
gpt-5.4-mini.

## A run-level attractor that compaction does not break (2026-10-12)

[[literature/papers/cai2026forkandflush]] finds gurkan2026mutation's operator
attractor one level up, at the grain of a whole multi-day agent run. Eight
independent runs of one coding agent on one task plateau at very different
scores. On an LLM-judged functional-similarity map, each run's submissions
stay in one region (an "idea basin"), and periodic context compaction does
not move them. The paper's move is go-with-the-winners at run grain: fork K = 4
continuations of the leader's workspace, start each with an empty chat
context, keep the best. A node is an episode chain over a workspace and the
value estimate is the task grader.

Two points for this concept. First, the attractor is carried by the
*conversation*, not the code. Branches that keep the context move about one
median step from the fork point, while a flushed branch can jump 10–43
steps into another basin (one hand-picked polyomino fan-out; the size
depends on the embedding). Second, it agrees with zou2026fmlbench that
structure matters most on sparse landscapes. The large gains are on AHC014
(+142%) and order-add (+101%). On dense tasks such as relay-lay and
LLM-SQL the gain is under 1%. How often a winning branch actually changes
basin is never counted, so "escaping basins" is illustrated rather than
measured.

## The operator attractor under selection, and a controller fix that holds (2026-10-13)

[[literature/papers/gurkan2026mutation]] measured the LLM operator's
attractor with no selection. [[literature/papers/he2026beneath]] measures it
inside a real autoresearch keep/revert loop (Opus 4.7, GPT-5.2 and
Llama-3.3-70B, four small NLP pipelines, 300 iterations). Diff-level
diversity stays flat while mechanism entropy over a 9-category edit taxonomy
falls about 0.6 nats. Late edits are LR, warmup and β tweaks on different
lines of `train.py`. This is min2026autonomous's lr/epochs default,
reproduced with a stated mechanism: a peaked proposer prior, times an
acceptance rule that passes low-variance tweaks more reliably, times history
conditioning. An Aider loop with structurally different diffs collapses the
same way, so the edit span is not what sets the effective grain.

The fix acts on what is *proposed*, not on the span. The controller samples
N candidates, reweights them by inverse category frequency, and drops any
candidate within cosine 0.85 of the last 200 accepted intents. This kept
entropy at 1.71 nats and cut cluster decay 0.68 → 0.28 at about 1% in-loop
cost. Temperature 1.2 did nothing. Reading for this concept: profile the
operator by category, not by diff, and widen it in the controller.
Caveats: three seeds, ML-specific taxonomy, 124M–3B targets.

## The gain is in the structural moves, and the insight is a per-run draw (2026-10-26)

[[literature/papers/zhong2026automating]] reproduces the lr/epochs attractor
(min2026autonomous, he2026beneath) on 20 ML-for-science codebases with
off-the-shelf CLI agents. Opus's interventions are 56% hyperparameter
tuning; Codex sweeps "over a hundred experiments" from one script. The
paper adds two controls that this concept lacked.

- **An HPO-only arm separates the attractor from the gain.** Same agent,
  same budget, restricted to Optuna over the exposed hyperparameters, on
  6 benchmarks:

  | Benchmark | HPO-only | Unrestricted |
  |---|---|---|
  | CheMixHub | 1.4 | 11.9 |
  | Mars-Bench | 4.3 | 46.4 |
  | PFDelta | 4.6 | 64.4 |
  | DART-Eval | 2.7 | 6.1 |
  | TIDMAD | 3.5 | 7.4 |

  About half the gain survives on DART-Eval and TIDMAD. Elsewhere the gain
  comes from a few structural moves: a backbone or detector swap, or a loss
  reformulation. The sweep that fills the rest of the run is mostly
  residual. So the attractor is a waste of budget, not where the gain comes
  from.
- **Run-to-run variance separates kinds of move.** Over four Opus runs,
  engineering gains reproduce (SD 0.3–1.5 points). The PFDelta objective
  reformulation appears in 2 of 4 runs (29.4–65.3, SD 19.6).

The 3-day runs fit this: the extra time went to sweeps and found no
insight that the 12-hour runs had missed. Our inference for grain: insight
behaves like a low-probability draw per independent trajectory, not like
progress per hour. Extra budget is better spent on more independent starts
(or flushed forks, cai2026forkandflush) than on extending one run. Not
tested at matched compute; n = 4 runs on one insight benchmark.

The skill bundle widened what was proposed (persona, tiered plan, a
calibration pass that demotes "a hyperparameter change ... in disguise").
That is a prompt-side widening, unlike he2026beneath's controller
reweighting. In the 3-day runs it did not hold past about hour 12.

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


## A grain above code: the plan as the mutation unit

[[literature/papers/yu2026primescientist]] sets the grain one level higher
than any other source here. The mutation unit is **not code** — it is the
experimental plan, written as a self-contained Markdown skill, and the
operator is an LLM-authored `diff.py` that regenerates the *entire* child
from the parent (the prompt insists the output "must be a FULL self-contained
experimental plan... NOT a delta"). Code editing is delegated wholly
downstream to the executor.

That is `min2026autonomous`'s "the declared space is the effective grain"
taken to its conclusion: if the plan is the only thing the search operator
can touch, no amount of executor skill puts an unplanned idea in the pool.

Two cautions. The evidence is n = 1 per cell. And there is an **unaddressed
tension with [[literature/papers/zou2026fmlbench]]**: FML-Bench's controlled
study — same LLM, same editor, same step budget, 18 tasks — ranked MCTS
*last*, while this paper reports its MCTS beating Greedy and UCT on n=1 cells
over 8 tasks. It cites FML-Bench and never addresses the conflict. FML-Bench's
design is the stronger of the two, so treat the search-algorithm comparison
here as unsupported and take only the grain point.

## Grain on a schedule: count the edits, then anneal the count (2026-09-29)

[[literature/papers/xia2026rrsi]] (RRSI, harness evolution) adds a third
grain axis alongside span and per-candidate effort: **how many
independently attributable edits one candidate may bundle**. It puts that
number on a cosine schedule, `b_t = b_min + (b_max − b_min)·½(1 +
cos(πt/T))`, from 3–4 edits in round 0 down to 1 by the last round. The
rationale is about generalization, not capability. A bundle of unrelated
edits "can fit more idiosyncrasies of the current feedback, and any
measured change is difficult to attribute". Late single-edit rounds make
the per-edit credit log (component, hypothesis, diff, ΔS, ΔC, kept?)
attributable.

This is the first attested *scheduled* grain, which Implementation
guidance 4 said was unattested. It runs in the opposite direction from the
one guessed there: coarse-to-fine, not function-first-then-file. It pairs
with two things above:

- **Stall-triggered redirection toward components never touched** (w = 3,
  one reserved slot). This is the operator-attractor fix gurkan2026mutation
  calls for, and it uses zou2026fmlbench's stall trigger. The authors'
  named attractor is "repeatedly rewriting prompts while leaving agent
  structural mechanisms untouched".
- **A search space left fully open** (prompt, control flow, config,
  context, tools, skills, memory, subagents), with the declared component
  vocabulary serving only to steer exploration.

The evidence is thin for the grain itself. The anneal, credit history
and stall exploration are ablated together as "proposal regularizers" in
one run. Removing them leaves the evolve score flat (90.5 → 90.7), lowers
the OOD average (43.6 → 41.9), and raises tokens per trial (2.42M →
2.69M). So the direction matches the rationale, but the contribution of
the edit schedule alone is unmeasured.

**A scheduled feedback signal, coarse-to-fine like the edit schedule above
(2026-10-06).** [[literature/papers/xu2026conflictguide]] schedules what the
proposer *sees* rather than the edit size:
- Stage I: 100 proposals on the scalar metric alone.
- Stage II: 100 proposals that also see behaviour-level probes and are
  told which components the conflict lives in.

On FNO, the share of mechanism-targeted (spectral) edits rose
12.3% → 43.4%, and SNGP and GCNII show similar shifts. The order matters.
The same probes shown from iteration 0 concentrated proposals early
(spectral edits 18.1% → 31.8%), and the resulting model was *worse than
the unmodified reference*: test NRMSE 0.0465 vs 0.0458, while scalar-only
search reached 0.0379. This is one model and one run, but it is the first
attested cost of narrowing search too early. Like the anneal above, the
schedule is fixed in advance (T1 = 100), and the only sensitivity check
(50/100/150 on FNO R0) favours the middle split.

## The selection rule as a query the agent may ignore (2026-10-13)

[[literature/papers/farahbakhsh2026agentdiscover]] (AgentDiscover) moves one
level above the grain. The coding agent chooses which past candidates to
build on, not only what to rewrite. Every idea and candidate is stored in a
graph database, so MAP-Elites parent selection is a single Cypher query
that the agent may use, combine or skip. The agent also runs its own local
experiments before each submission. On Anthropic's kernel take-home at
matched spend, it submitted 15 candidates against 128–166 for the
population baselines, and it reached 960 cycles against 1,098. This fits
ishibashi2026effective's per-candidate-depth finding above: fewer, deeper
individuals.

The evidence does not separate the two levers. No arm holds the coding
agent fixed and forces a fixed selection rule on it. Local
experimentation, a named target, prediction-before-submit, steering and a
meta agent all travel together. The authors say they have not checked
whether the agents rediscover MAP-Elites or UCT. Main results are single
runs. Treat "let the agent own selection" as compatible with the data, and
"go deeper per candidate" as the better-supported reading.

## The context an evolve prompt carries is a price, and on circle packing single-call evolution barely beats fresh samples (2026-10-26)

This concept says a larger grain needs "richer prompt construction".
[[literature/papers/oved2026llm]] puts a price on the prompt. An evolve
proposal carries the program being edited and part of the archive, and
both grow as the run proceeds. On gpt-5-mini one proposal costs
15.6k–38.6k tokens, and a fresh sample from the task description costs
4.1k–6.0k (4.6–7.2× less). The prompt's context is the main cost of a
proposal, and it rises with depth.

Read beside the per-individual-effort section above, circle packing gives
a three-way comparison across two papers:
- Single-call OpenEvolve on gpt-5-mini costs 22.8k tokens/proposal. That
  matches the 26.5k ishibashi2026effective logged on gpt-5.2-codex. Its
  equal-token ratio against independent sampling is 1.01 (EvoX and
  AdaEvolve 1.04).
- [[literature/papers/ishibashi2026effective]]'s Vesper spends 54K–465K
  tokens per individual through a stateful coding agent. At 40M tokens it
  scored 2.636 against OpenEvolve's 2.541 (×1.037).

So on this task, the population over cheap single-call proposals added
about 1–4% over having no population. A deeper operator added a further
3.7% on top of OpenEvolve. That supports spending the budget on depth per
individual rather than on more stateless generations, at least where a
fresh sample is nearly as good as an edit. Heilbronn is the exception in
the same paper: there, editing a good program keeps paying (1.23–1.28 over
sampling).

Caveats:
- Different models and harnesses across the two papers, so the 1–4% and
  3.7% are not additive or directly comparable.
- The oved2026llm window is capped at about a fifth of each run's tokens.
- The grain itself (function vs file) is not varied by either paper.

## Selection inflation scales with candidates scored, so breadth-vs-depth wins read on the dev score lean toward breadth (2026-10-26)

The two spend-per-individual sections above compare few deep candidates
with many shallow ones. [[literature/papers/chen2026auditing]]'s
Proposition 1 prices one side of that comparison. Take a loop that scores
N proposals with no true gain, with i.i.d. development errors of SD σ,
accepting whenever a candidate beats the incumbent. Then:
- the reported development gain has expectation σ·m_{N+1} (Gaussian
  errors), where m_n is the expected maximum of n standard normals;
- the audited gain is zero;
- the inflation "grows with the number of candidates scored, not with the
  number of rounds."

Our arithmetic for the counts in this concept: m ≈ 2.46 at 88 candidates
and ≈ 3.36 at 1,500. On a noisy dev score, the 1,500-candidate arm starts
about 0.9σ ahead of the 88-candidate arm before either finds anything.

- **ishibashi2026effective is unaffected.** Circle Packing is scored
  deterministically, so σ = 0. Its fewer-deeper result stands as measured.
- **wei2026scale's B cuts σ itself.** Scoring each candidate on more
  rollouts shrinks the noise, which is one plausible part of why B = 100
  was the only setting that transferred. The paper did not separate this.
- **hu2026winners is the measured case.** GEPA's small validation set
  bought a 91-program pool against 9 and a larger reported gain, but not
  a larger held-out one.

Rule: when a comparison of candidate counts or per-candidate spend is read
on the selection score and the evaluator is noisy, a win for fewer, deeper
candidates is conservative. A win for more, shallower ones needs a
re-score outside the selection set. The paper does not discuss grain; this
is our application of its Prop. 1. Real loops have correlated errors on
shared dev items, which shrinks the effect, so treat these as upper-end
reference values.
