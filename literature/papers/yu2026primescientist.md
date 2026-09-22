---
kind: paper
title: "PrimeScientist: Strategic Allocation of Research Effort in Autonomous Research"
authors: ["Xinle Yu", "Fan Bai", "Kaiser Sun", "Hengshuo Miao", "Abhay Anand", "Zhongyan Luo", "Kun Zhou", "Zhen Wang"]
institutions: ["UC San Diego", "Johns Hopkins University"]
year: 2026
venue: "arXiv (cs.CL)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.17846"
code_url: "https://github.com/Henri-XYu02/PrimeScientist"
citations: null
source: "raw/papers/yu2026primescientist.pdf"
added: "2026-09-22"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["budget", "resource-allocation", "mcts", "plan-tree", "research-agent", "sample-efficiency", "ablation", "single-run", "self-benchmark", "allocation-policy", "autoresearch"]
---

# PrimeScientist: Strategic Allocation of Research Effort in Autonomous Research

## TL;DR

An autonomous-research harness that keeps competing research plans alive in a
tree and selects the next one with an MCTS-style rule whose **exploration
exponent is a function of the remaining token budget**:
`α_t = min(1/r_t, α_max)`, `r_t = (B − U_t)/B`, `α_max = 10`. Planning and
execution draw on one shared budget (`B = 1.5M` tokens, GPT-5 via Codex CLI),
so proposing a plan competes with running one. The abstract's headline —
**"+10.3% average reward with 50.6% fewer research attempts than
AutoResearch"** — is real arithmetic (0.7738 vs 0.7018; 13.33 vs 27.0
attempts) but comes from **one search per configuration** on **FIRE-Bench, a
benchmark six of the eight authors wrote**. The only multi-seed head-to-head
in the paper (Table 7, 3 searches × 6 AutoLab tasks, GPT-5-mini) gives
**reward parity — 0.4363 vs 0.4359 — and a 24% attempt reduction, not 50.6%**.
At matched 1M tokens (Table 10) the baseline is very slightly *ahead* on mean
reward (0.4357 vs 0.4338). The durable result is therefore **sample efficiency
(fewer complete attempts / less wall-clock), not research quality**. For this
project the load-bearing detail is architectural rather than numeric: the
budget is a **harness-level scalar that never enters any LLM's context**, and
Algorithm 1's only stopping rule is `while U < B` — there is no stall
detection anywhere.

## Claims

- **Strategic effort allocation should be an explicit optimization target.**
  Definition 1 formalizes it as a sequential decision problem with state
  `X_t = (x, P_t, H_t, B − U_t)` and objective `max_π J_B(π) = E_π[V(H_τ)]`,
  where a policy may construct a plan, execute a plan, or stop, and every
  action charges the same budget.
- **Remaining resources should modulate the policy, not just halt it.** "Even
  with the same observed outcomes, different remaining resources can warrant
  different allocations." Implemented as the `α_t = min(1/r_t, α_max)`
  selection exponent: dispersed sampling early, concentrated sampling as the
  budget drains.
- **Plans must outlive the executor's conversation.** An executable plan tree
  stores each plan as a self-contained Markdown "skill", and each edge as an
  executable `diff.py` that regenerates the child plan from the parent — so a
  candidate direction stays selectable while another branch is evaluated, and
  the lineage is reconstructible.
- **Planning must be inside the budget.** `U_t = C_t^exec + C_t^refl`. The
  paper's claim is that this makes the comparison honest, and it is the one
  row where it beats every system in its own Table 1 ("Shared token budget").
- **Adaptive exploration beats fixed exploration** (Table 6) and **the plan-tree
  policy beats UCT / Greedy / Random** (Table 5).
- **Claim the paper does *not* make, and should be read for:** it never claims
  its policy approximates the optimum of its own Definition 1. No regret bound,
  no optimality argument, no derivation of `1/r_t`. The formalism frames the
  heuristic; it does not justify it.

## Methods

- **Backbone.** Executor and reflector are both GPT-5 through the Codex CLI,
  shared budget `B = 1.5 × 10^6` tokens per task. Secondary setting: GPT-5-mini
  both roles, `B = 1 × 10^6`. Attempt caps: 25 (AutoLab, MLE-Bench), 30
  (FIRE-Bench). `m = 3` child proposals per expansion, `α_max = 10`,
  prune threshold `δ = 0.05`.
- **Selection.** Sample child `v` of `u` with weight
  `w_t(v) = Q(v)^α_t` if evaluated, `(Q(u)·P(v))^α_t` if unvisited, where
  `P(v)` is the reflector's prior. If `Q(u) = 0` the prior is used alone, with
  a positive floor on sampling weights.
- **Backpropagation.** `Q(v)` = **mean** reward over *retained* evaluated
  descendants of `v`, including `v`.
- **Pruning (Algorithm 1 line 8).** A newly evaluated child `s` is excluded
  from subsequent selection iff
  `s ≠ s_0 ∧ δ > 0 ∧ Q(parent(s)) > 0 ∧ ρ < Q(parent(s)) − δ`.
  Note the shape: a **tolerance band**, not a bare comparison. Note also that
  it is one-sided and compares against a *parent branch mean that the pruning
  itself biases upward*.
- **Stopping (Algorithm 1 line 3).** `while U < B`. That is the whole of it.
  The `stop` action in Definition 1 is **never implemented**. Return value is
  `argmax_{s evaluated} h(s)`.
- **Budget never reaches the model.** "The executor receives the plan without
  instructions about the remaining global budget. The reflector can inspect run
  metadata but receives no additional allocation rule." The 27-line reflector
  system prompt (Appendix G) contains no budget token. Budget-awareness lives
  entirely in one arithmetic expression in the harness.
- **Benchmarks.** FIRE-Bench (12 AI-research rediscovery tasks, claim-level
  RAGChecker F1); AutoLab (8 systems/code-optimization tasks, throughput
  reward under correctness gates); MLE-Bench (4 Kaggle competitions).
- **Baselines.** (B1) Single-run agent; (B2) AutoResearch (Karpathy's
  edit-run-keep-or-revert linear loop); (B3) same tree/reflector/pruning with
  Random, Greedy, UCT (`c = √2`), or fixed-exponent selection.
- **Replication.** Tables 2, 3, 4, 5 are **one search per cell**. Tables 6, 7,
  10 are 2–3 searches. No statistical test appears anywhere in the paper.
- **FIRE-Bench scoring.** "Each plan node is executed twice and the higher
  score is recorded" — max-of-2 on a noisy LLM-judged F1, with both runs
  charged to budget and attempt count.

## Results

### The headline, and what survives replication

- **FIRE-Bench (Table 3, n = 1 per cell).** Mean F1 **0.7738 vs 0.7018**
  (+10.26%), attempts **13.33 vs 27.0** (−50.63%). This is the abstract's
  number.
- **The paper's own walk-back, same paragraph.** "Excluding that task leaves
  mean rewards of **0.7533 versus 0.7266**" — i.e. removing MCQ Selection Bias
  (where PrimeScientist scores 1.0000 against 0.4290) drops the quality gap
  from **+10.3% to +3.7%**.
- **Per-task on FIRE-Bench, PrimeScientist *loses* on four of twelve**:
  Activation Control 0.3810 vs 0.4400, QuestBench 0.6820 vs 0.7500, Grokking or
  Not 0.4900 vs 0.5000, Lost in The Middle 0.7690 vs 0.8570. The paper states
  "improve on six tasks and match on two" and leaves the four losses to
  subtraction.
- **AutoLab (Table 2, n = 1).** Mean reward 0.4824 vs 0.4428 (+8.9%). **The
  entire gap is one task**: Smallest Game Player, 0.3224 vs 0.0000, an
  accuracy-gated task the baseline never cleared. The paper discloses this
  directly: "On the remaining seven tasks, average rewards are comparable
  (**0.5052 versus 0.5061**)" — which is a 0.0009 deficit, i.e. the quality
  claim on AutoLab is a single-task artifact.
- **MLE-Bench (Table 4, n = 1, 4 tasks).** "each method leads on two of the
  four competitions." Score parity. Attempts 11–15 vs 21–25.
- **The replicated comparison (Table 7, 3 searches, GPT-5-mini, AutoLab).**
  Mean reward **0.4363 vs 0.4359** — a dead tie. Mean attempts **21.6 vs
  28.4**, a **~24%** reduction, less than half the headline's 50.6%. Per-task,
  PrimeScientist is *behind* on Gaussian Blur (0.4740 vs 0.4859), Hash Join
  (0.6259 vs 0.6291), Flash Attention (0.2939 vs 0.2960) and FFT (0.5002 vs
  0.5133), and ahead on Concurrent KV WAL (0.5830 vs 0.5608) and Sha256
  (0.1410 ± 0.1523 vs 0.1304 ± 0.1540 — a std larger than the mean).
- **Matched tokens (Table 10, 1M, 3 searches, 6 AutoLab tasks).** Computed
  means: PrimeScientist **0.4338**, AutoResearch **0.4357**. The baseline is
  marginally ahead. The prose quotes only the two tasks where PrimeScientist
  wins (Concurrent KV WAL 0.583 vs 0.561, Sha256 0.141 vs 0.130) and calls the
  set "comparable."
- **Matched attempts (Table 11).** The table caption states it "shows the
  **three improved cases** from a six-task comparison." The other three are not
  reported at all.

### The budget-adaptivity ablation — the claim this project cares about

Table 6, GPT-5-mini, 1M shared budget, 6 tasks, `n = 3` for the adaptive policy
and `n = 2` for each fixed exponent. Mean max reward (population std):

| Task | Adaptive | Random α=0 | Fixed α=3 | Greedy α=10 |
|---|---|---|---|---|
| Activation Control | **0.314** (0.010) | 0.245 (0.034) | 0.203 (0.016) | 0.279 (0.045) |
| QuestBench | 0.555 (0.085) | **0.664** (0.064) | 0.467 (0.005) | 0.431 (0.031) |
| LLM Value Consistency | 0.850 (0.108) | 0.856 (0.067) | 0.656 (0.071) | **0.962** (0.038) |
| Concurrent KV WAL | **0.583** (0.020) | 0.369 (0.084) | 0.499 (0.018) | 0.451 (0.196) |
| Hash Join | **0.626** (0.003) | 0.582 (0.029) | 0.617 (0.002) | 0.624 (0.008) |
| Flash Attention | 0.294 (0.037) | 0.294 (0.030) | 0.270 (0.030) | **0.297** (0.088) |
| **Average** | **0.537** | 0.502 | 0.452 | 0.507 |

Read carefully this is a **weak positive**: adaptive leads on 3/6, is beaten by
α=0 on QuestBench by 0.109 and by α=10 on LLM Value Consistency by 0.112, and
the average margin over the best fixed alternative is **0.030** with n = 2–3
and per-cell stds up to 0.196. Its actual defensible claim is the one the
authors make — "achieves the strongest overall reward **without choosing a
separate constant for each task**" and "never ranks last." That is a
*robustness-to-hyperparameter* argument, not a demonstration that `1/r_t` is
the right schedule.

Table 5 (policy comparison, GPT-5, n = 1 per cell, 8 tasks): adaptive 0.598,
Greedy 0.563, UCT 0.551, Random 0.546. Single-run; Random wins two of eight.

### Cost of allocation, and where it stops paying

- **Planning is 38% of the budget** (Table 8): coding-agent execution 62%,
  reflection + tree context + child-plan generation 38%. "The reflector alone
  uses **21% to 37%** of tokens per run, and a plan expansion consumes **0.1 to
  0.3 million tokens**" — i.e. one expansion can be 10–30% of the whole
  1–1.5M-token campaign.
- **Attempt reduction under a fixed 1M budget (Table 9):** 112 vs 158 across six
  AutoLab tasks, ~29% fewer, lower on every task.
- **Budget sweep (Table 12), the most useful scope condition in the paper.**
  PrimeScientist / AutoResearch at 250k / 500k / 750k / 1M:
  Concurrent KV WAL 0.42/0.52, 0.47/0.51, 0.52/0.49, 0.57/0.54;
  Flash Attention 0.30/0.30, 0.26/0.33, 0.30/0.32, 0.27/0.30;
  Sha256 0.02/0.02, 0.01/0.03, 0.19/0.19, 0.20/0.18.
  **Below ~750k tokens the allocator loses**, because the 38% planning tax is
  not amortized. The overhead has a break-even and the paper measures it.
- **Elapsed time (Table 13, 12 searches/method):** mean 32 vs 42 min, but the
  **median is 34 min for both**. The time win is entirely tail behaviour. The
  prose reports the mean and not the median coincidence.

### What the paper says about stopping — essentially nothing

Definition 1 admits a `stop` action and a stopping time `τ_π`. Algorithm 1
implements neither: the loop is `while U < B`, and the returned artifact is the
argmax over evaluated nodes. There is **no stagnation counter, no
no-improvement ceiling, no convergence test**. The one tolerance-shaped
construct is the prune threshold `δ = 0.05`, which is asserted in
"Implementation details" with no derivation, no sensitivity study, and no
connection to the evaluator's measurement noise.

Definition 1 does, however, state the overshoot property independently and
correctly: "Actions are charged on completion, and no further action starts
once the budget is reached; **the final action may therefore cross the
threshold**."

## Critique / open questions

- **The headline benchmark is the authors' own.** FIRE-Bench is reference [58]:
  Wang, Bai, Luo, Su, Sun, **Yu**, Liu, **Zhou**, Cardie, Dredze, Hu, Xing —
  six of PrimeScientist's eight authors, including both corresponding authors.
  The 10.3% / 50.6% abstract number is a single-run result on a self-authored
  benchmark scored by an LLM-judged claim-matching F1. On the two **external**
  benchmarks (AutoLab, MLE-Bench) the quality result is parity. This is not
  disclosed as a conflict anywhere in the paper.
- **Abstract overstatement, precisely located.** "improves average reward by
  10.3% with 50.6% fewer research attempts" generalizes an n = 1 self-benchmark
  cell. The replicated version of the same comparison is *parity + 24%*. A
  reader who stops at the abstract gets a reward claim the body does not
  support at any level of replication.
- **Selective prose over honest tables.** Tables 2, 7, 10 and 12 all contain the
  unfavourable rows and the authors deserve credit for printing them, including
  the 0.5052 / 0.5061 seven-task null. But Table 10's prose quotes only the two
  winning tasks, Table 13's prose quotes only the mean, and Table 11 reports
  "the three improved cases" out of six with the rest omitted. The pattern is
  consistent enough to matter.
- **No statistical testing at all**, which the Limitations section concedes:
  "Statistical significance and backbone effects require separate analysis" and
  "Larger repeated-search panels would enable more precise estimates of small
  reward differences." Every reported difference outside Table 7's attempt
  counts is within plausible run-to-run spread.
- **Winner's curse is structurally baked in.** Reward is a noisy LLM-judged F1;
  FIRE-Bench takes the **max of two** executions per node; `Q(v)` is a mean over
  **retained** (i.e. post-pruning) descendants, so pruning biases the value
  every later selection conditions on; and the final output is
  `argmax_s h(s)` over all evaluated nodes. Four separate max-over-noise
  operators compound, and none is quantified. The paper never reports the
  evaluator's own reproduction spread, so it is impossible to tell how much of
  a 0.03 average gap is search and how much is selection on noise.
- **The mechanism is borrowed.** The remaining-budget exponent is cited to
  BAVT (Li et al., [33], arXiv 2603.12634) at the exact line where it is
  introduced: "uses the remaining-budget ratio to adjust how strongly selection
  favors high-value branches [33]." The novelty claimed is the *setting* —
  plan construction and complete research attempts under one shared budget —
  not the rule.
- **`1/r_t` and `α_max = 10` and `δ = 0.05` are all unjustified constants.**
  Definition 1 poses a well-formed optimization; the policy is a heuristic with
  no stated relationship to it. `α_max` is never ablated. `δ` is never ablated.
  A sensitivity study on either would have been cheaper than Table 5.
- **Direct tension with [[literature/papers/zou2026fmlbench]].** FML-Bench's
  controlled topology study (same LLM, same editor, same step budget, 18 tasks)
  found **MCTS ranked last** among search topologies and greedy hill-climbing
  tied the best tree search. PrimeScientist reports its MCTS variant beating
  Greedy and UCT on n = 1 cells over 8 tasks. FML-Bench is cited ([69]) but this
  contradiction is not addressed. FML-Bench's design is the stronger one.
- **Scope of the budget.** "Here B measures inference tokens, excluding elapsed
  time and CPU/GPU hours... allocation under separate resource constraints
  remains future work." A single-resource allocator. Our box's binding
  constraint is often GPU-hours, which this does not model.
- **Plan-level granularity only** (the paper's own limitation): "cannot
  reallocate effort among intermediate implementation decisions during an
  ongoing execution." The executor is a black box that consumes an
  unpredictable amount of the shared budget.
- **PDF rendering note.** `pdftotext` emits "Mismatch between font type and
  embedded font file." One glyph is displaced: UCT's exploration constant
  renders as `c = 2` with the `√` orphaned onto the preceding line — read as
  `c = √2`. All numbers quoted above were grep-verified against the extracted
  text; no minus signs appear in any results table, so sign-dropping is not a
  risk here.

## Trust signals

- **Credibility:** 3. Two reputable but non-frontier academic groups (UC San
  Diego, Johns Hopkins), a public code repo that resolves
  (`github.com/Henri-XYu02/PrimeScientist`), a substantial and genuinely candid
  four-part Limitations section that names the small panels, the max-of-2
  scoring, the absent significance testing and the single-resource scope, and
  several tables that print their own nulls (the 0.5052/0.5061 seven-task tie,
  the 0.4363/0.4359 replication tie, the 250k–500k budget losses, the identical
  34-min medians). Held at 3 rather than 4 by: an arXiv preprint with no peer
  review and no citations; **every headline table is n = 1**; zero statistical
  tests in the paper; the abstract's quality claim rests on a benchmark
  six of eight authors wrote, with no conflict statement; and at least three
  places where the prose reports the favourable half of a table it printed in
  full (Table 10's two wins, Table 13's mean-not-median, Table 11's
  "three improved cases" of six).

## Follow-up

- **Relevance:** 4. Not for its numbers, which are weak, but because it occupies
  a gap [[concepts/budget-as-ceiling]] explicitly named. ZEBRA
  ([[literature/papers/hamri2026zebra]]) established allocation-under-a-ceiling
  but was one-shot, pre-execution, and *explicitly excluded phaseless agentic
  loops* — our dominant shape. PrimeScientist runs a **sequential, reactive,
  forecast-free** allocator over exactly that shape, with the planning cost
  inside the budget, and it measures the overhead (38%) and the break-even
  (~750k tokens) that ZEBRA could not. Not 5 because the policy is borrowed
  from BAVT, the adaptivity evidence is n = 2–3 with no test, and it contributes
  nothing to the repo's live stall-detection question.

- **[[concepts/budget-as-ceiling]] — the counter-pole section extends to
  phaseless loops.** Three specific additions, in descending confidence:
  (1) **Architectural, and the strongest claim here.** Budget-awareness need not
  be a prompt. PrimeScientist's budget enters only as `α_t = min(1/r_t, α_max)`
  computed in the harness; "the executor receives the plan without instructions
  about the remaining global budget" and the reflector prompt contains no budget
  token. This is ZEBRA's "LLM estimates, solver optimizes"
  ([[concepts/scripted-tool-pipelines]]) in a stronger form — the LLM is not
  even told the budget exists — and it sidesteps ye2026agent's "token
  elasticity" failure mode by construction, since there is no soft constraint to
  be elastic about.
  (2) **Allocation has a measured overhead and a measured break-even.** Planning
  is 38% of the shared budget, a single expansion 0.1–0.3M tokens, and below
  ~750k total tokens the allocator *loses* to a plain greedy loop (Table 12:
  0.42/0.52 and 0.47/0.51 on Concurrent KV WAL at 250k/500k). This sharpens the
  concept's existing "allocation only matters when the budget binds" point into
  a two-sided condition: the budget must bind **and** be large enough to
  amortize the controller.
  (3) **Independent formal restatement of ceiling-plus-one-call.** Definition 1:
  "Actions are charged on completion, and no further action starts once the
  budget is reached; the final action may therefore cross the threshold."
  Arrived at independently of [[literature/papers/ye2026agent]], which measured
  it (40K budget, halted at 56K).

- **[[concepts/budget-as-ceiling]] — what it does *not* supply, and this is the
  honest headline for the digest's framing.** The repo's open question is
  whether `max_consecutive_no_improvement: 3` needs a tolerance term. This paper
  **has no stall detector and no stopping rule**: Algorithm 1 is `while U < B`,
  and the `stop` action in its own Definition 1 is never implemented. It is
  therefore a *worked example of the failure mode*
  [[literature/papers/zhang2026agora]] documented — a search that can only
  terminate by exhausting its budget — rather than a fix for it. The one
  relevant artifact is the prune rule, `ρ < Q(parent(s)) − δ` with `δ = 0.05`:
  the **right shape** (a tolerance band around a reference value rather than a
  bare `>`) applied in the right place, but with `δ` asserted as a constant,
  never ablated, and never related to the evaluator's reproduction spread —
  precisely the calibration zhang2026agora showed is the whole difficulty. So:
  a shape precedent, not evidence. Worth one or two sentences in the
  "noise floor" subsection saying that a second system independently reaches for
  a tolerance band and independently fails to calibrate it, which raises the
  priority of the local fix rather than resolving it.

- **[[concepts/spend-forecast-calibration]] — a boundary clarification, small
  but real.** The concept states that a budget *gate* needs a calibrated
  external forecast. PrimeScientist shows an *allocator* needs none: `r_t` is
  realized consumption over a known constant, involving no prediction of what
  the next action will cost — and Definition 1 says outright that costs "may be
  unknown before completion." Worth recording the distinction: forecasting is a
  requirement for **admission** decisions (besanson's pre-action gate), not for
  **allocation** decisions, which can react to realized spend instead. This is
  a genuinely cheaper design point and it is the one thing here that
  bai2026how's r ≤ 0.39 result does not undermine. Two sentences; no other
  change warranted, since the paper measures nothing about forecasting.

- **[[concepts/evolutionary-search-grain]] — a new grain point, weakly
  attested.** PrimeScientist's mutation unit is **not code**: it is the
  *experimental plan*, stored as a self-contained Markdown skill, and the
  mutation operator is an LLM-written `diff.py` that reads the parent
  `skill.md` and emits a complete child `skill.md` ("The output skill.md must be
  a FULL self-contained experimental plan... NOT a delta"). Code editing is
  delegated wholly to a downstream coding agent. This is
  [[literature/papers/min2026autonomous]]'s "the declared space is the effective
  grain" taken to its conclusion — the search space *is* the declared plan —
  and it sits above whole-file grain on the spectrum. The reflector is also
  explicitly instructed toward operator diversity ("Propose between 1 and N
  CONTROVERSIALLY DIFFERENT skill variants... Never pad with minor variations"),
  which is the prompt-level countermeasure to
  [[literature/papers/gurkan2026mutation]]'s entropy-collapse finding — but
  unmeasured here, so it is a design datum, not evidence. Note also the
  unaddressed tension with [[literature/papers/zou2026fmlbench]] above. Suggest
  2–3 sentences adding plan-grain to the spectrum, flagged as single-system and
  n = 1.

- **[[concepts/evidence-gated-completion]] — one quotable line, optional.** The
  reflector prompt instructs: "**The agent often claims success -- trust
  packet.json, not the narrative.**" A deployed harness encoding the
  self-report/oracle split directly in its supervisor prompt. Nice corroborating
  quote; adds no new evidence.

- **Digest candidates.** The most valuable is **[33] Li, Deng, Li & Li, "Spend
  less, reason better: Budget-aware value tree search for LLM agents"
  (arXiv 2603.12634)** — the *actual source* of the remaining-budget exponent
  PrimeScientist uses, so it is where any derivation or ablation of `1/r_t`
  would live. Currently only name-checked in
  [[literature/papers/hamri2026zebra]]. Also worth fetching: **[22] AlphaLab
  (arXiv 2604.08590)** — the closest prior art, whose Strategist shifts from
  exploration to refinement using the *remaining experiment count*, which is a
  direct analogue of a `max_experiments` ceiling entering a policy; **[38] BATS,
  "Budget-aware tool use enables effective agent scaling" (arXiv 2511.17006)** —
  continuous remaining-tool-call awareness, the finest granularity on this axis;
  and **[6] MARS (arXiv 2602.02660)** — cost-constrained MCTS for AI research
  that backpropagates execution *time* into the reward, i.e. the multi-resource
  case this paper defers. **[58] FIRE-Bench (arXiv 2602.02905)** is worth a look
  separately for [[concepts/hce-evaluation]] — a claim-level RAGChecker-based
  rediscovery benchmark whose construct validity is now load-bearing for this
  paper's headline.
