---
kind: paper
title: "Stopping and Routing LLM Judge Panels"
authors: ["Bin Zhu", "Yi Xie", "Yanghui Rao"]
institutions: ["Sun Yat-sen University, School of Computer Science and Engineering"]
year: 2026
venue: "arXiv 2608.19802v1, cs.CL (preprint, 20 Aug 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2608.19802
code_url: null
citations: null
source: "raw/papers/zhu2026stopping.pdf"
added: "2026-08-24"
relevance: 3
credibility: 3
status: read
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/refusal-cost-symmetry]]"
related_experiments: []
tags: [evaluation, llm-as-judge, panel-construction, cost-allocation, calibration, ensemble]
---

# Stopping and Routing LLM Judge Panels

## TL;DR

Reframes "which judges should I call, and when should I stop" as a
target-relative, conditional allocation problem rather than a fixed
ranking or a call-everyone default. From a small labeled audit set, each
candidate judge is scored by its *conditional* validation-risk reduction
given the current panel — a **copy** (near-zero conditional gain, drop
by default), a **complement** (broad residual gain, add globally), or a
**specialist** (gain concentrated on a declared slice, route only
there). Across eight benchmark settings (hard reasoning, code, safety,
preference, reward-modeling, summarization, math), the resulting policy
matches or beats flat all-judge panels and full-call stacking at a
fraction of the calls — e.g. LLMBar risk 0.2118→0.1884 *and* accuracy
0.6692→0.7334 at 3.46 average judge calls versus a 7-judge flat panel.

## Claims

- **Judge diversity is target-relative and conditional, not a nominal
  property of the pool.** "A reward model that is weak as a standalone
  preference judge can still be a complement after a rubric prompt
  enters the panel. Conversely, a second prompt from the same model
  family can become a copy if its conditional gain vanishes after the
  first prompt." Role is an action-oriented label attached to a
  (candidate, current-panel, target-distribution, slice) tuple, not a
  fixed trait of the judge.
- **The projection-gain identity requires no independence assumption
  among judges** (Lemma 1): the conditional value of adding judge `j` to
  panel `S` is the squared-loss risk reduction from including it,
  provably non-negative, and computable from a labeled audit split
  without assuming judges err independently — the assumption most
  reliability-jury and stacking methods quietly make.
- **Redundant copies should change deployment, not just interpretation.**
  A stress test that adds four exact copies of an existing judge to two
  benchmarks leaves the role policy's risk and cost byte-for-byte
  unchanged (LLMBar 0.1884/3.46, JBB 0.1094/2.29 before and after),
  while a full-call reliability jury's cost rises to 11 calls and its
  risk *worsens* (LLMBar 0.2058→0.2860) because duplicated votes are
  overweighted. Nominal pool size and model-family count can go up while
  the calling policy stays flat.
- **The construction procedure is a validation-gated greedy search with
  an explicit stopping report**, not an optimization run to convergence:
  starting from an empty panel, add the best-conditional-gain candidate
  only if it clears a declared threshold `τ`, repeat; then run the same
  search per slice with the global panel frozen. When no remaining
  candidate clears threshold, "the policy stops and records a
  validation-based stopping report that every unused broad or slice gain
  is below the declared threshold" — explicitly "operational rather than
  asymptotic," i.e. a deployment audit artifact, not a statistical
  guarantee.
- **Two operating regimes, cleanly separated by the results.** Broad-
  complement regimes (hard GSM8K rationale, MBPP overfit audits) are
  where a full-call reliability jury or flat panel is still competitive
  because every signal contributes to the aggregate. Slice-conditional
  regimes (safety proxy/audit slices, LLMBar's adversarial vs. natural
  splits) are where routing wins outright, because "useful judges are
  conditional on declared slices, not merely next in a global quality
  order." The paper's stated contribution is the policy that tells a
  practitioner which regime they're in, not a claim that routing always
  wins.

## Methods

Fix a target distribution `P` and a finite candidate judge pool `J`
producing per-item signals `Z_j`. For a panel `S ⊆ J`, define the oracle
predictor `η_{P,S}(z) = E[Y | Z_S = z]` and its squared-loss oracle risk;
the conditional value of adding judge `j ∉ S` is the risk reduction
`g_P(j|S)`, extended per-slice as `g_{P_f}(j|S)` for declared slices `f
∈ F` (e.g. LLMBar's adversarial/natural subsets). Audit data splits into
construction-fit / construction-validation / final-test; pattern
calibrators estimate `η` by cell means on the fit split, selection uses
validation gain only, and final numbers come from the held-out test
split alone. Global construction is greedy: add the best cost-adjusted
validation-gain candidate while it clears threshold `τ_P`, then run the
same search per slice with the global panel fixed; the deployed policy
routes `π(x) = S ∪ S_{f(x)}` using only pre-available metadata, verifier
outputs, classifier outputs, or already-observed judge disagreement —
never ground-truth labels, which exist only for audit-time analysis.
Judge pool: Qwen2.5 Instruct 7B, Llama 3.1 Instruct 8B, Mistral v0.3 7B,
Prometheus 2 v2.0 7B, Gemma 3 IT 12B, Atla Selene Mini (Llama 3.1, 8B),
and DeepSeek V4 Flash (284B total / 13B active) — a deliberately mixed
pool of small local judges plus one much larger MoE judge. Benchmarks:
hard GSM8K rationale audits, MBPP public-overfit audits, JailbreakBench
safety, LLMBar preference, RewardBench, Arena100K, SummEval, MATH-500.
Baselines: single best validation judge, flat all-judge panel, matched-K
top-k, correlation-diverse and quality-diverse panels, full-call
ridge/logistic stacking, Dawid–Skene-style reliability juries, and
FrugalGPT/RouteLLM-style confidence cascades. All results averaged over
10 random splits; τ_P = τ_f = 0.005 unless noted.

## Results

- **Main comparison (Table 3):** role-routed policy beats or matches
  both single-best and flat-all on 6/8 settings while using far fewer
  than a full 7-judge call. Flagship case, LLMBar-7: risk 0.2118 (flat)
  → 0.1884 (role) *and* accuracy 0.6692 → 0.7334, at 3.46 average calls.
  On Arena100K and SummEval, the policy correctly collapses to the
  single-best judge (matching it exactly), since the flat panel there is
  *worse* than the single best (Arena100K flat risk 0.2462 > single
  0.2321) — expanding the panel would have actively hurt.
- **Strong-baseline comparison (Table 4):** full-call aggregation
  remains the best absolute risk endpoint in some settings (RewardBench
  0.0201 @ 7 calls vs. role 0.0291 @ 1.80; MATH-500 0.0536 @ 5 vs. role
  0.0678 @ 1.70), which the paper reports as the honest boundary case
  rather than hiding it — "role policies solve the deployment problem of
  deciding when to buy a small stopped panel, when to route specialists,
  and when to keep the full-call endpoint," not a claim to always
  dominate.
- **Redundant-copy stress test (Table 9):** adding four exact copies of
  an existing DeepSeek judge leaves the role policy's risk/cost
  unchanged on both LLMBar (0.1884/3.46) and JBB (0.1094/2.29), while
  the full-call jury's cost rises 7→11 and its risk on LLMBar worsens
  0.2058→0.2860 from overweighted duplicate votes.
- **Threshold sensitivity (Table 8):** `τ` behaves as a transparent
  risk-cost dial in the expected direction on most settings (higher `τ`
  → fewer calls, usually flat or slightly worse risk), with LLMBar the
  stated exception — a more conservative threshold there *improves*
  risk by avoiding sparse or redundant expansions.
- **Routed-specialist stability (Table 7):** on LLMBar, the same
  judge-slice route (e.g. DeepSeek anchor → llama3_8b_v on the
  `adversarial_gptinst` slice) is selected in 5-6 of 10 random splits,
  not all 10 — the paper's own honest reading is a deployment
  recommendation to "collect more audit labels before relying on
  low-frequency specialists," not a claim of route determinism.

## Critique / open questions

- **The judge pool conflates model scale with prompt/role diversity**:
  six small (7-12B) open models plus one much larger MoE judge
  (DeepSeek V4 Flash) sit in the same candidate pool, and no ablation
  isolates how much of the routing gain comes from *having the strong
  judge available to route to* versus the allocation policy itself
  choosing well among comparably-weak judges.
- **Risk is squared-loss calibration risk on binary labels**, not task
  accuracy directly (accuracy is reported as a secondary, "legible"
  metric) — a policy tuned to minimize calibration risk is not
  automatically the policy an accuracy-focused deployment would want,
  and the two occasionally diverge in the reported tables (e.g. MBPP:
  role has the best accuracy and best risk simultaneously, but this
  need not hold in general for a squared-loss objective).
- **No comparison against a single strong judge with self-consistency
  or majority voting** — an obvious, much simpler alternative to
  building a full role-conditioned policy, and the paper's own
  RewardBench/MATH-500 results (where full-call aggregation wins) leave
  open whether a cheaper strong-judge-plus-resampling baseline would
  close most of the gap the routed policy claims.
- **Single academic group, no released code or artifacts visible** in
  the paper itself — the stopping/routing policy and audit-set
  construction are described precisely enough to reimplement, but
  nothing is currently checkable against a reference implementation.
- The paper explicitly shares its benchmark judge-output matrices and
  judge pool with a companion paper ("A Finite-Calibration Regime Map
  for LLM Judge Panels") that addresses a related but distinct
  deployment decision (panel prefix + aggregation family, once outputs
  exist) — worth fetching alongside this one if the allocation question
  proves load-bearing here.

## Trust signals

- **Credibility:** 3 — a reputable but non-flagship group, a
  methodologically careful evaluation (8 benchmark settings, an
  identity lemma with a clean non-independence property, an honest
  reporting of the boundary cases where full-call aggregation still
  wins, a redundant-copy stress test that is exactly the adversarial
  check this kind of claim needs). Held below 4: no released code,
  single preprint with no independent replication, and a judge pool
  that mixes model scales in a way the ablations do not disentangle.

## Follow-up

- **A genuinely different lever on the same problem
  [[concepts/refusal-cost-symmetry]] names.** That concept's sources
  (ho2026soundnessbench, ray2026what) show recall collapsing under a
  *stricter threshold* on a single judge or gate. This paper attacks
  the same false-approval/false-rejection tradeoff from **panel
  composition** instead of strictness — adding, dropping, or routing
  *which* judges get called, holding each judge's own threshold fixed.
  Concretely relevant to `/elevate`'s pending elevate-paired-control
  proposal: if a panel-routing policy can hold recall while a stricter
  single-judge threshold cannot, it is a candidate alternative or
  complement to the paired-control re-test that proposal currently
  specifies — worth a closer read before that proposal is implemented,
  not after.
- **For [[concepts/programmable-evaluator-oracle]], this is evidence at
  the *meta* level — designing the evaluator-calling policy — rather
  than the evaluator itself.** Most of that concept's sources build a
  single oracle/fitness function; this paper assumes several already
  exist and asks which to invoke, when, and at what cost. The
  copy/complement/specialist taxonomy and the "drop redundant signals
  even as nominal diversity grows" result generalize past LLM judges to
  any multi-oracle evaluation setup (e.g. this project's own multi-
  dimension `/code-review` panels) — the label is target-relative and
  conditional there too, and a redundant dimension-check should be
  droppable by the same conditional-gain logic rather than by fixed
  configuration.
- Not proposed as a concept seed: the mechanism is squarely an instance
  of designing the evaluator layer, which the two linked concepts
  already cover from complementary angles.
