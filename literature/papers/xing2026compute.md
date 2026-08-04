---
kind: paper
title: "Compute Allocation in Evolutionary Search: From Depth-Breadth to Multi-Armed Bandits"
authors: ["Sixue Xing", "Haoyu He", "Kerui Wu", "Zhuo Yang", "Haozheng Luo", "Tianfan Fu", "Aarthy Nagarajan"]
institutions: ["University of Notre Dame", "Northeastern University", "University of Massachusetts Amherst", "Southeast University", "Northwestern University", "Nanjing University", "Shanghai Artificial Intelligence Laboratory"]
year: 2026
venue: arXiv (cs.CL)
peer_reviewed: false
url: https://arxiv.org/abs/2605.29268
code_url: https://github.com/keruiwu/self-evolving-allocation
citations: null
source: "raw/papers/xing2026compute.pdf"
added: "2026-08-04"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/async-worker-pool]]"
related_experiments: []
tags: [evolutionary-search, compute-allocation, bandits, depth-breadth, fixed-budget, reliability, variance, reporting-discipline, scaling]
---

# Compute Allocation in Evolutionary Search: From Depth-Breadth to Multi-Armed Bandits

## TL;DR

The first systematic fixed-budget study of *how* LLM-guided evolutionary
search should spend its calls, rather than how many it needs. Two
regularities (a fitness–compute envelope on effective FLOPs where model
capability ordering largely collapses, and a bilinear depth–breadth fit
whose interaction term predicts whether the optimum is at a corner or in
the interior), plus BaSE — a multi-armed bandit allocating calls *across*
parallel trajectories — which lifts mean fitness 12.3% over the strongest
island baseline **with the model, prompt, and evaluator held fixed**.

## Claims

- Headline numbers from *Evolve systems are systematically incomparable*:
  FunSearch reports a 4-of-140 hit rate, CodeEvolve displays "only the
  best", AlphaEvolve a single number; reported per-run cost spans ~510×
  (≈150 LLM calls in ShinkaEvolve to 204,800 candidates in ThetaEvolve).
  Few report the run-to-run distribution, so published figures
  "characterize what is achievable on a favorable run, not what a
  practitioner should expect at a finite computational cost."
- **Allocation is an independent lever**, orthogonal to model and prompt
  quality, and it remains meaningful once model, prompt, and evaluator
  are fixed. The 12.3% gain "is not a model improvement, nor a prompt
  improvement, but an allocation improvement."
- Under a fixed budget `C = T · N`, the depth–breadth split determines
  *where on the fitness landscape a run lands*, and that landscape has
  **task-dependent geometry** — some tasks have broad depth-favored
  plateaus, others a narrow interior ridge where only a balanced split
  wins.
- Fixed within-run allocation cannot remove **cross-run heterogeneity**:
  identical configurations produce a *distribution*, with some seeds
  stagnating at low fitness indefinitely. A second lever operating
  *between* trajectories is therefore required.
- **Allocation amplifies an existing signal but cannot create one.**
  Model capability and prompt jointly set a ceiling that allocation only
  governs the approach to.

## Methods

- **Tasks**: three geometric optimization problems from AlphaEvolve,
  shipped with the OpenEvolve example suite — Circle Packing (n=26),
  MinMaxDist (n=16), Heilbronn Triangle (n=11). Evaluator and
  initial-program files used verbatim; raw objectives normalized so
  FITNESS=1.0 is the best published construction.
- **Models**: open-weight Qwen3 at 1.7B/4B/8B/14B (thinking mode on) and
  Llama-3.1-8B; vLLM v0.18, fp8, temp 0.6, top-p 0.95, up to 16 in-flight
  parallel calls on one H100.
- **Sweep**: greedy protocol over `C ∈ {8..512}`, `T ∈ {1,2,4,...,C}`,
  10 seeds per cell. `T=1` recovers best-of-N; `T=C` is pure sequential.
  Island protocols (OpenEvolve, CodeEvolve, ShinkaEvolve) run at C=512.
- **Post-hoc FLOPs accounting** — the methodological centrepiece.
  LLM-call count is *not* a fair cost axis across models or protocols
  because prefix-cache hit rates differ (greedy reuses one parent prefix
  across N siblings; island rewrites the prompt per call). They charge
  `FLOPs_call = 2·P_active·(p_prompt − p_cached + p_out)`.
- **Statistics**: stratified bootstrap standard errors and 95% CIs over
  1000 resamples, following the rliable protocol (Agarwal et al. 2022).
- **BaSE**: K parallel trajectories from the same seed program as bandit
  arms; each pull spends one LLM call extending that trajectory and
  reveals its new fitness. UCB, EXP3.P, and Thompson sampling
  implemented, plus a *random* arm-choice baseline used to separate
  effects.
- **Clean two-step ablation**: `Greedy/Island → Random` isolates the
  **pool effect** (drawing K independent trajectories and returning the
  best); `Random → BaSE` isolates the **allocation effect** (adaptively
  routing compute toward promising arms). Both are shown to survive.

## Results

- **Fitness–compute envelope**: for every model, best-in-sweep fitness
  rises smoothly and monotonically with compute — no phase transitions.
  Re-priced in effective FLOPs, capability ordering largely dissolves
  (MMD 4B/8B/14B envelopes nearly coincide, R²=0.94; CP 8B/14B coincide
  below the ceiling, R²=0.93). Larger models lead per *call*, but the
  lead is mostly paid for in FLOPs.
- **Bilinear regularity**: `log(1−V) = β₀ + a·log T + b·log N + c·log T·log N`
  reaches R² ∈ [0.75, 0.92] vs 0.74–0.78 for a budget-only model. The
  interaction coefficient `c` alone controls geometry — small |c| leaves
  the optimum at the all-depth corner; large negative `c` bends it to a
  balanced interior optimum with plateau half-width ∝ 1/√|c|. MMD sits
  in the interior-ridge regime, CP and HT at the corner limit (|c|<0.03).
- **BaSE fitness (C=512)**: best or near-best across all cells. Qwen3-8B
  HT — Thompson 0.8736 vs greedy 0.6780 and ShinkaEvolve 0.7379. Llama
  HT — Thompson 0.4387 while every non-BaSE method stays below 0.2538
  (OpenEvolve and CodeEvolve collapse to exactly 0.0). Qwen3-14B CP —
  1.0003, i.e. above the published construction.
- **Threshold-reaching efficiency** is the more striking result. On MMD,
  UCB reaches τ=0.95 in 92 generations vs greedy's 485. On HT, UCB
  reaches τ=0.70 within 60 generations while greedy, OpenEvolve, and
  CodeEvolve **never reach it** within budget. Thompson reaches the same
  thresholds with ~40% fewer generations than random allocation across
  the seven reached cells.
- **Saturation caveat, stated by the authors**: on CP several baselines
  already approach the ceiling (greedy 0.9985, ShinkaEvolve 0.9986), so
  "the CP gains are necessarily small." Gains concentrate on hard,
  high-variance settings.
- **Arm-pool ablation**: K ∈ {5,10,20} is the sweet spot; very small
  pools under-explore, K=50 dilutes refinement across too many shallow
  trajectories.
- **Prompt confound, honestly surfaced**: ShinkaEvolve's apparent CP
  strength is traced to its system prompt explicitly suggesting
  `scipy.optimize`, and its HT prompt carries a "CRITICAL — degeneracy
  warning" about collinear-triplet failures that no other baseline has.
  Their own greedy run, using OpenEvolve's prompt, reaches 0.9985 — a
  +0.17 gap over OpenEvolve — by the LLM eventually discovering `scipy`
  despite the prompt discouraging it.

## Critique / open questions

- **Three geometric tasks from one example suite.** CP/MMD/HT are all
  low-dimensional continuous-geometry problems with cheap deterministic
  evaluators. Whether the bilinear regularity or the `c`-coefficient
  regime classification transfers to ML-engineering search (expensive,
  noisy, long-horizon evaluation) is entirely untested — and that is the
  setting this project cares about.
- **Open-weight models up to 14B only.** No frontier model appears
  anywhere. The capability-ceiling finding is precisely the one most
  likely to change at frontier scale, and the FLOPs-collapse result is
  fitted within a single family plus one Llama.
- **Evaluation cost is assumed free.** The entire cost model counts LLM
  FLOPs; `Eval` is treated as costless. For research agents where a
  fitness evaluation is a training run, the depth–breadth arithmetic is
  dominated by a term this paper omits.
- **Bandit arms share a seed program and are independent thereafter** —
  no migration, crossover, or cross-arm information flow, unlike real
  island protocols. So "BaSE vs island" compares allocation against a
  protocol that also does something else.
- The `c`-coefficient is fitted post hoc per (model, task) cell; there is
  no method for predicting the regime *before* spending the sweep budget,
  which is what a practitioner actually needs.
- No limitations section in the main body.

## Trust signals

- **Credibility:** 4 — code released, 10 seeds per cell with stratified
  bootstrap 95% CIs under the rliable protocol, a genuine FLOPs-based
  cost axis that corrects for prefix-cache asymmetry between protocols,
  and a clean pool-vs-allocation ablation; the authors surface their own
  confounds (prompt-induced baseline advantage, task saturation, capability
  ceiling) rather than burying them. Held below 5 by: no peer review, a
  seven-institution academic group with no independent replication,
  three narrow tasks, and no model above 14B.

## Follow-up

- **Relevance:** 4 — supplies quantitative evidence for
  [[concepts/evolutionary-expansion]]'s allocation question and hard
  numbers for [[concepts/pass-at-k]]'s reporting-discipline argument,
  and connects both to [[concepts/budget-as-ceiling]] by making the
  search policy a budget-allocation policy. Not 5 only because the task
  suite is far from research-agent workloads.
- The strongest importable claim for this project is the **reporting
  discipline**, not the bandit: reporting best-of-unspecified-runs at
  unspecified cost is the norm in this literature and it is unreliable
  as a guide to single-run performance. That is exactly
  [[concepts/pass-at-k]]'s thesis, arriving from the evolutionary-search
  side with a measured 510× budget spread as the evidence.
- The **threshold-reaching metric** (earliest generation at which 90% of
  bootstrap samples reach fitness τ) is a better fit for this project's
  own `/iterate` loop than final fitness: our ceilings in `budget.yaml`
  are wall-clock and token ceilings, so "how fast does 90% of the seed
  distribution clear the bar" is the operational question, not "what
  does the best seed eventually reach."
- Pairs directly with [[literature/papers/gurkan2026mutation]]: that
  paper shows LLM mutation operators collapse population *diversity*;
  this one shows allocation can route budget away from the collapsed
  trajectories. Together they suggest diversity loss and allocation are
  the two halves of one problem — worth checking whether BaSE's gains on
  high-variance settings are simply it detecting gurkan's structural
  attractors early.
- The prefix-cache term in their FLOPs accounting is the same economics
  [[literature/papers/hao2026selfgc]] prices for context eviction — cache
  behaviour is becoming a first-class cost axis in two unrelated
  subfields at once.
- Open thread: `Eval`-cost-aware allocation. Every result here assumes
  free evaluation; for ML-research agents the natural extension is a
  bandit over trajectories where each pull's cost is dominated by the
  fitness evaluation, which changes the arm-pool sizing conclusion.
