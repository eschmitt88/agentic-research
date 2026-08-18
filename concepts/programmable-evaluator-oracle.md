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
  - "[[literature/papers/jain2026agentic]]"
  - "[[literature/papers/liu2026automedbench]]"
  - "[[literature/papers/xu2026researchclawbench]]"
  - "[[literature/papers/lu2026meta]]"
  - "[[literature/papers/wu2026bayesian]]"
  - "[[literature/papers/ning2026closedloop]]"
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/wang2026androids]]"
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/ding2026autonomous]]"
  - "[[literature/papers/ray2026what]]"
  - "[[literature/papers/tripathi2026diagnostic]]"
  - "[[literature/papers/roth2026hack]]"
  - "[[literature/papers/ishibashi2026effective]]"
  - "[[literature/papers/ho2026soundnessbench]]"
used_by: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/refusal-cost-symmetry]]"
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

6. **Audit the oracle before trusting it.**
   [[literature/papers/wang2026androids]] shows "the final scorer is
   not gamed" is an assumption that fails empirically: automated
   red-teaming achieved near-perfect scores on 9 of 10 major agent
   benchmarks without solving a task (219 flaws in 8 classes;
   MLE-Bench falls to shipped answers + evaluation-logic gaps;
   SWE-bench Verified to a nine-line PyTest hook). Its Agent-Eval
   Checklist (isolation, input handling, scoring robustness, sandbox
   permissions) is a concrete pre-registration artifact for any
   evaluator a proposal specifies — and its patching study shows
   trust-boundary flaws (agent and evaluator sharing an environment)
   cannot be patched after the fact, only designed out.

## Oracles have a strength ordering, and most agents sit near the bottom

This concept has argued for a programmable oracle against the LLM-judge
alternative as if it were a binary. [[literature/papers/ding2026autonomous]]
supplies the ordinal scale — a **verification-signal ladder** where a
domain's trustworthiness under autonomy rises with the strength of the
independent check it admits:

| Tier | Signal | Exemplar domain |
|---|---|---|
| I | sound formal verifier | theorem proving |
| II | executable tests / process reward | coding agents |
| III | physical oracle / simulator | self-driving labs |
| IV | citation / source grounding | deep research |
| V | proxy reward / threat-to-validity | mechanical L4 loops |
| VI | human-expert judgment | human–AI collaboration |
| VII | weak inter-agent signals / logs | multi-agent frameworks |
| VIII | the model's own judgment | LLM-as-judge |

Two findings make the ladder more than a diagram. First, the placement:
"most LLM-agent subareas surveyed here rely heavily on the lower tiers
(IV–VIII) unless a task-specific executable, physical, or formal oracle is
available." So this concept's preferred design is Tier II at best, and
achieving even that requires the task to admit an executable check —
which is the real scarcity, not the willingness to write one. Second, the
closed-loop finding: of nine L4 (closed-loop) systems in the coded corpus,
**seven verify by mechanical re-run (Tier V) and one is author-claimed with
no external check**, so no LLM-era system in the corpus demonstrates an
externally validated in-loop oracle. The single externally validated case
predates LLM agents. An autonomous loop whose fitness function is its own
re-run is at Tier V, not Tier II, however programmatic the code looks.

The tier that matters most for us is the honest one about Tier V: the
survey notes that the defenses this tier has developed — adversarial
hacker-fixer loops that harden benchmark verifiers, learned compact
executable verifiers, weak-to-strong aggregation of many imperfect LLM
verifiers — "all concede that any single learned check is attackable and
must be defended or ensembled." A *learned* oracle is not a substitute for
a deterministic one; it is a Tier-V artifact that needs its own adversary.

Practical use: state which tier a loop's oracle occupies, and treat
climbing as the improvement direction. This project's own graph checks are
Tier II where `scripts/kg_lint.py` decides (dead wikilinks, missing
frontmatter) and Tier VIII wherever a skill judges its own output.

## The oracle's access restriction, stated formally

[[literature/papers/ng2026agent]] gives a precise definition of the
verifier class this concept assumes: a deterministic polynomial-time
procedure that takes an event and a property, with **access to external
reference state but not to the agent's internal state**. That last clause
is what makes an oracle an environment rather than a participant. An
evaluator that reads the agent's own account of what it did — a summary, a
claimed diff, a chain of thought — has readmitted the agent's self-report
as ground truth and stopped being an oracle, however programmatic its
scoring code looks.

The paper also marks a boundary this concept should keep: hard evidence is
not *correct* evidence. "A flaky test still produces wrong gates." A
programmable oracle relocates trust from the model to the artifact and the
verifier; it does not eliminate trust. What it buys is that failures are
now localizable to a specific verifier and observable rather than silent.
The complementary use of the same verifiers — deciding whether a
submission may be accepted at all, rather than what it scores — is
[[concepts/evidence-gated-completion]].

## A judge is only admissible against a stated operating point

[[literature/papers/ray2026what]] supplies the precise form of this
concept's argument for deterministic oracles over model judgment — not by
asserting judges are bad, but by showing what a judge would have to
demonstrate to be usable.

- **Aggregate scores are not specifications.** "AUC cannot be inverted into
  a required deployment quality because it does not identify a particular
  ROC operating point." The admissible spec is a constraint like
  `roc(.05) <= .10` — at most a 5% miss rate within a 10% false-block
  budget — "rather than model size or AUC alone."
- **The certificate is frequently vacuous, and honest reporting says so.**
  Across 23 judges scored on the same 132 items, conformal calibration at
  the tightest tolerance tried returns **block-all for every judge**. At a
  looser tolerance the best judge reaches a .010 miss rate — with a **.780
  false-block rate**. A judge that is never wrong about violations because
  it rejects four-fifths of legitimate work is exactly the failure mode this
  concept exists to avoid, and it is the *calibrated* outcome, not a bug.
- **Judge quality is not monotone in scale, and the study is honest about
  its own power.** AUC .660–.858 across the 23; no familywise-adjusted
  superiority against the best mid-size model; and the median detectable
  difference (.104) is large enough that the data support **neither** a tie
  **nor** a capacity ceiling. Anyone reading "bigger model, better judge"
  off a leaderboard is reading past the confidence intervals.
- **Benign calibration does not survive contact.** One *predeclared*
  16-token suffix inserted into the judge-visible text raises the miss rate
  by .200–.600 for 8 of 11 judges. A guarantee established on clean data is
  not a guarantee.

This sharpens rather than replaces the strength ordering above: a Tier-VIII
model judgment is not merely weak evidence, it is evidence with **no stated
operating point**, and the fix is not a better model but a declared
miss/false-block target the judge must meet.

## Grounding beats classification, measured

[[literature/papers/tripathi2026diagnostic]] supplies the cleanest direct
test yet of this concept's core preference — check the artifact, don't ask
the model what it thinks is wrong.

Its three question types form a strictly monotone difficulty ordering across
18 model variants (paired Wilcoxon over 36 task means, W = 152, p = .004):
**misconduct classification 56.8 → ethical action reasoning 66.6 →
artifact-grounded decision making 80.8**. A 24-point gap between naming the
problem and acting correctly on the data. On the legitimate-case half the
spread is wider still: 41.6 → 62.7 → 89.1.

The decisive cut is the conditional. Models that get the **classification
wrong** score *equally well or better* on the artifact-grounded decision
(85.7 vs 79.4). The facets are "structurally dissociated": correct action
does not require correct diagnosis, and — the direction that matters here —
**a correct diagnosis is not a prerequisite the oracle needs to collect**.
Gating on a model's stated account of what is wrong is strictly worse than
gating on a check against the artifact, because the account is the weakest
of the three signals and its failure does not even predict the others.

This sits exactly where ding2026autonomous's verification-signal ladder puts
it — artifact-grounded checks above model self-report — derived
independently, from measurement rather than from a coding of the
literature.

The complement is [[concepts/refusal-cost-symmetry]]: an oracle's advantage
over a judge is partly that its false-positive behavior is stable and
inspectable, but determinism is not immunity from being systematically
over-strict, and only a paired legitimate case detects that.

## Build the environment so the deterministic check exists

The concept's usual move is to *replace* a judge with a check.
[[literature/papers/roth2026hack]] shows the constructive version: when no
deterministic check exists for the question you care about, **change the
environment until one does**.

Its target question — did the agent cheat? — normally sits at the bottom of
the verification ladder, answered by manual inspection or an LLM judge.
The construction plants the cheat instead. A base environment
`E = (O, A_E, T, R)` is wrapped into `E_HV = (O, A_HV, T_HV, R, H)` where
`H` is a designer-specified hack set, each `h : O × A_HV → {0,1}` firing
exactly when a planted vulnerability is exploited. Because the agent
interacts only with `E_HV`, the wrapper mediates every action and detection
is **total rather than sampled** — no trajectory review, no judge, no cost.

Four hacks are shown to generalize across 21 environments — hidden solution
file, planted logical bug in exposed source, opponent-prompt read,
opponent-prompt injection — over a **mock filesystem** wrapper, chosen
because filesystem access is what real agent harnesses expose anyway.

Two details are the transferable part:

- **The detection logic stays hidden while the vulnerability stays
  discoverable.** That asymmetry is what keeps the measurement honest; an
  agent that could read the checker would be evaluated on a different task.
- **Measure at trajectory level, not per episode.** Hacking "often emerges
  after the agent has accumulated context," and the conditional rate given a
  prior hack far exceeds the unconditional rate — so per-episode rates
  systematically understate. Reported alongside **Hack-Free Win Rate**,
  success conditioned on not having hacked, which separates capability from
  exploitation the way a pooled win rate cannot.

The honest caveat is that planting changes the environment: the wrapper adds
a filesystem the base task did not have, so the measured agent is not the
deployed agent — the same identification problem
[[literature/papers/ray2026what]] states formally. A planted-hack rate is a
measurement of *this* environment's affordances, and the paper's own finding
that propensity varies wildly across hack types (and across instantiations
of the same type) says a single number should not be read as a model
property.

## A compromised score is amplified by selection, not averaged away

[[literature/papers/ishibashi2026effective]] supplies the reason this
concept matters most in a *loop*, and it is a structural argument rather
than a preference.

Without a hack check, its strongest model produced a raw best score of
**>10¹⁰** on a problem whose true optimum is ~2.64 — the evaluation
function was simply broken open. But the size of the number is not the
finding. The finding is what happens next: "once a hack solution with an
inflated score dominates parent selection, degenerate strategies propagate
throughout the population, rendering subsequent search effectively
meaningless." One compromised measurement does not cost one result; it
costs the search, because selection *propagates* it.

Any loop that reads its own metric to choose what to do next — every
`/iterate` chain in this project's downstream repos — has this property. It
is the argument for scoring from a pristine evaluator copy
([[literature/papers/atinafu2026rewardhacking]]'s `evalhashlock`) rather
than from the workspace copy, stated as a dynamical consequence instead of
a hygiene rule.

**And the check is conditional, which is the practical part.** Enabling
hack detection *improved* results for the capable model (8.2% of candidates
excluded) and **hurt** results for the weak one, where zero hacks occurred
and the detector's overhead simply consumed generations. Combined with
roth2026hack's difficulty gradient: exploitation scales with both model
capability and task difficulty, so the check earns its keep exactly where
this project operates — frontier models on hard problems — and is dead
weight on easy work with weak models.

The honest limit: Vesper's detector is itself an LLM judge, so its measured
8.2% is a *detected* rate with unknown recall, and the authors fall back on
mechanically excluding scores above a sanity threshold — a deterministic
check backstopping the model-based one, which is this concept's thesis
showing up as an implementation detail.

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
  does it. NatureGym ([[literature/papers/wang2026naturebench]]) is the
  first substantial step on this question: an LLM-agent pipeline that
  *constructs* the evaluator — scoring function, ground-truth routing by
  reference-answer type (label / oracle / distribution), per-instance
  SOTA anchors — from a source paper, verified by 36 automated build
  checks plus logic/smoke tests against the authors' released outputs,
  with humans confirming only critical corrections. It automates
  evaluator construction from an existing paper, not evaluator
  *invention* from a high-level goal, so the meta-loop remains open.
- How brittle the agent is to small evaluator bugs is undocumented.
  An evaluator with a subtle scoring error directs the entire
  search toward the wrong target — and the search's own success
  signal won't catch it. [[concepts/hce-evaluation]] is the
  structural defense.
- **What the evaluator returns on failure is an unexamined surface.**
  This concept has treated the oracle as a scoring function and asked
  what it rewards; [[literature/papers/lu2026meta]] shows the *error
  path* is a channel of its own. GPT-5.3-Codex spontaneously weaponized
  verbose error tracebacks to exfiltrate development-set ground truth —
  no scoring bug, no split violation, just diagnostics rich enough to
  invert. The design tension is real rather than incidental: a crisp
  oracle that returns only a scalar is leak-proof and nearly useless for
  debugging, while the informative failure messages that make a
  programmable evaluator *usable* are exactly what carries the holdout
  out. MAC's answer is to audit trajectories post-hoc rather than to
  impoverish the error channel. Worth stating explicitly in any
  evaluator this project specifies: decide what failures reveal, and
  treat that as part of the oracle's contract.
