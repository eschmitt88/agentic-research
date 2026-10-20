---
kind: concept
name: "pass-at-k"
status: experimental
added: "2026-04-24"
source_papers:
  - hambardzumyan2026aira
  - chan2024mle
  - starace2025paperbench
sources:
  - "[[literature/papers/edwards2025rexbench]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/chan2024mle]]"
  - "[[literature/papers/starace2025paperbench]]"
  - "[[literature/papers/jain2026agentic]]"
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/lupidi2026airsbench]]"
  - "[[literature/papers/xing2026compute]]"
  - "[[literature/papers/li2026acm]]"
  - "[[literature/papers/panigrahy2026energy]]"
  - "[[literature/papers/yoon2026arcticswarm]]"
  - "[[literature/papers/ning2026scores]]"
  - "[[literature/papers/zhang2026double]]"
  - "[[literature/papers/kim2026are]]"
  - "[[literature/papers/she2026efficient]]"
  - "[[literature/papers/kim2026divergent]]"
  - "[[literature/papers/agarwal2026fire]]"
  - "[[literature/papers/nguyen2026cliffcompaction]]"
  - "[[literature/papers/wiedmann2026agents]]"
  - "[[literature/papers/wang2026rethinking]]"
  - "[[literature/papers/bobadillasuarez2026audit]]"
  - "[[literature/papers/zhang2026veriharness]]"
  - "[[literature/papers/cai2026forkandflush]]"
  - "[[literature/papers/li2026beyond]]"
  - "[[literature/papers/liu2026what]]"
  - "[[literature/papers/ning2026implementation]]"
  - "[[literature/papers/iyengar2026sample]]"
used_by:
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/hce-evaluation]]"
related_experiments: []
tags: [evaluation, variance, seeds, reproducibility]
---

# pass-at-k

## Definition

Seed variance is a first-class axis of experimental design: any
reported result is a distribution over k seeds, not a single number.
A run with k=1 is a pilot; a claim requires k≥3 and a statement of
the seed distribution, not just its mean.

## Why it matters

MLE-bench ([[literature/papers/chan2024mle]]) and AIRA_2
([[literature/papers/hambardzumyan2026aira]]) both surface that
headline agent performance is seed-sensitive at a magnitude that
rivals architectural changes. A 2-point medal-rate difference
between two agent architectures can vanish when both are run at
k=5; conversely, a true architectural improvement can be invisible
at k=1 if the seed happens to fall in the wrong mode.

The AIRA_2 scaling-laws analysis explicitly treats seed distribution
as signal: predictable scaling across LLM variants emerges only when
each configuration is run at k≥3.

**The same pathology, measured in a different subfield.**
[[literature/papers/xing2026compute]] audits the LLM-guided
evolutionary-search literature and finds the reporting convention is
best-of-an-unspecified-number-of-runs at an unspecified cost: FunSearch
reports a 4-of-140 hit rate, CodeEvolve displays "only the best",
AlphaEvolve a single number, and reported per-run budgets span **~510×**
(≈150 LLM calls to 204,800 candidates). Their conclusion is this
concept's thesis in the authors' own words — existing reports
"characterize what is achievable on a favorable run, not what a
practitioner should expect at a finite computational cost."

Two things make this more than another citation. First, they show the
variance is not incidental: at an *identical* configuration, some seeds
climb to high fitness while others stagnate indefinitely, so a single
run is a draw from a distribution with a heavy bad mode — which is
precisely why k=1 medal rates mislead. Second, they demonstrate that
this variance is *actionable*, not just a reporting hazard: routing
budget across parallel trajectories converts run-to-run heterogeneity
into a reliability gain (see [[concepts/evolutionary-expansion]]
guidance 7). Reporting the distribution and exploiting the distribution
turn out to be the same capability.

The cost axis matters too. They argue LLM-call count is not comparable
across systems because prefix-cache hit rates differ by protocol, and
re-price everything in effective FLOPs — at which point much of the
apparent capability ordering between model sizes dissolves. Any k-run
comparison in this project should state the budget unit alongside k;
"k=3" at unspecified and unequal cost is not a controlled comparison.

**Decompose the distribution: pass@k vs passᵏ.**
[[literature/papers/li2026acm]] reads the same k runs two ways —
pass@4 (any-of-4 succeeds) as a proxy for the *capability boundary*,
pass⁴ (all-of-4 succeed) as a proxy for *consistency* — and shows an
intervention can move them independently: agentic context management
lifts pass⁴ from 34.1 to 59.3 on BrowseComp-Plus while pass@4 moves
only 73.5 → 82.0. The mean (pass@1) conflates the two; a treatment
that "helps" on the mean may be widening the capability boundary,
tightening consistency, or both, and those imply different follow-ups
(harder tasks vs variance reduction). Where k runs already exist, both
statistics are free — `metrics.json` distributions should let a reader
compute each.

**Report the tail, and check the variance is the model's.**
[[literature/papers/zhang2026double]] adds three refinements. First, the
mean can tie what the tail separates. GPT-5 and Claude-Haiku-4.5 differ by
0.006 in mean ground-truth F1 but by 0.079 in CVaR@0.2, and GPT-4o ranks
fifth by mean with a worst case of 0.00. Worst case, CVaR@α (mean of the
worst α fraction), and Reliability@τ (fraction of runs clearing τ, the
continuous-score analogue of passᵏ) belong beside the mean. Second, **a
tight distribution is not evidence of reliability.** Under a scaffold that
owned every execution decision, unrelated models produced byte-identical
per-seed vectors. The variance was zero because the model was not being
measured, and k seeds at matched lists cannot reveal that. Third, **"seed"
needs a referent.** Seeding the environment's adversary makes the worst
case reproducible, but it is a different axis from sampler seeds. GPT-4o's
solve-or-fail split occurred at T = 0 on byte-identical prompts, so the
seed alone does not explain it. A related caution from the same paper: a
power study "certifies the statistical test and says nothing about whether
the instrument could have picked up a behavioral effect". Their
well-powered B1 null came from a channel that never fired (0/120).

## Implementation guidance

1. **Metrics files report distributions.** Instead of
   `{"val_acc": 0.84}`, report
   `{"val_acc": {"mean": 0.84, "std": 0.02, "seeds": [42, 43, 44, 45, 46], "values": [...]}}`.
   Skills that rank experiments use the mean; skills that decide
   whether two runs differ must inspect the distribution.

2. **Default k.** For cheap experiments (minutes on CPU), k=5 is a
   reasonable floor. For expensive experiments, k=3 with an
   explicit note in the Diagnostics section. k=1 is allowed only
   for pilots — mark `status: pilot` on the experiment frontmatter.

3. **Seeds are recorded, not re-derived.** Each experiment's
   `config.yaml` lists the seeds used. Re-running does not
   regenerate the seed list; it produces a new run at the same seeds.

4. **Comparison requires overlap in seed distribution.** A vs B is
   comparable only if they ran at matched seed lists. If not, the
   comparison is "A at seeds S1 vs B at seeds S2" and must say so.
   Matched seeds are necessary but not sufficient: runs under different
   scaffolds (who retries, selects, and submits) are not comparable at any
   k, and a result should carry its scaffold level alongside its seed list.

5. **Report time-to-threshold, not only final metric.**
   [[literature/papers/xing2026compute]] reports, for each method, the
   earliest generation and cumulative FLOPs at which **90% of bootstrap
   samples reach a target fitness τ**. This is a better fit for a
   budget-capped loop than final fitness: it answers "how reliably does
   this clear the bar within budget" rather than "how high did the
   luckiest seed get." It also exposes failures a mean hides — in their
   results, several baselines simply *never reach* a threshold within
   budget, which a mean-and-std summary reports as a merely lower score.
   Where an experiment has a meaningful target, `metrics.json` should
   carry the threshold, the fraction of seeds reaching it, and the cost
   at which they did.

6. **Use a distribution-aware statistical protocol.** xing2026compute
   follows rliable (Agarwal et al. 2022) with stratified bootstrap
   standard errors and 95% CIs over 1000 resamples at 10 seeds per
   cell. For any comparison this project treats as load-bearing, a
   bootstrap CI over the seed distribution is the minimum bar; a mean
   ± std over k=3 does not support a claim that A beats B.

## The same denominator argument, in energy

[[literature/papers/panigrahy2026energy]] makes this concept's move on the
cost side rather than the score side, which is a useful independent
statement of the principle. Its objection to energy-per-inference is exactly
pass@1's problem with a single sample: the unit counts *attempts* and so is
blind to the distribution of outcomes across them. Its fix — Energy per
Successful Goal, aggregating every attempt including failures and retries
and dividing by goals actually accepted — is the cost-side analogue of
scoring a method by what it delivers over k tries rather than by one run.

Measured consequence: a failed attempt drew 2,256.1 J against 1,358.4 J for
the successful one on the same goal, so per-inference accounting missed
62.4% of true cost; agentic workflows come out at 4.33× the energy per
successful goal of matched linear baselines. Two rules transfer. Always
normalize by outcomes, never by attempts — on both axes, since a method that
looks efficient per call and a method that looks strong per sample can be
the same method measured twice charitably. And **granularity must be fixed
by the specification, not by the system**: the paper pins one benchmark row
to one goal precisely so a system cannot improve its number by
re-decomposing the work, the same defense pass@k needs against redefining
what counts as an attempt.

## Latent complementarity is abundant; selection is the scarce resource

This concept's case for k > 1 is that a single sample understates what a
method can do. [[literature/papers/kim2026are]] measures the gap between that
latent capacity and what a selector actually harvests, and the gap is the
whole story. Over 31,900 subsets of 30 models on MMLU-Pro, **oracle gain is
positive in 100% of subsets on both benchmarks** — some member is right
essentially always — **yet unweighted majority vote beats the strongest
member in only 9.98% of canonical size-3 subsets** (18.71% ±3.70 when the
best member is picked on held-out items). The descriptive oracle-capture
ratio is *negative*: −131.7% mean, −120.6% pooled.

Read precisely, this bounds the **rule**, not selection in general — majority
vote over answer labels is the weakest selector available, and the setting is
zero-shot MCQ with frozen correctness matrices, not agent trajectories. But
the shape generalizes: the same author's companion result finds that
"iteration produces better candidates that a reference-free LLM judge largely
fails to select" (arXiv:2607.13347).

**The operational point for this project.** pass@k with an oracle and pass@k
with a real selector are different quantities, and the first is close to
uninformative about the second — here they differ by roughly an order of
magnitude in how often the ensemble wins. A reported pass@k that is silent on
how the winning run would have been *chosen* without the answer key is
reporting the oracle. Where this project cares about k (medal rates,
best-of-k experiment selection), the selector belongs in the specification
alongside k, and [[concepts/programmable-evaluator-oracle]] is the component
that has to carry it.

On agent trajectories, a trained selector does much better than vote, and
it comes with a new trap: where it was trained.
[[literature/papers/nguyen2026cliffcompaction]] reports pass@1, a learned
"Practical" selector and the Oracle for each configuration on
Terminal-Bench 2.0 (Kimi K2.6, k=3). The selector is a LightGBM over
trajectory features plus within-group line and symbol agreement. It
harvests 8.3 of a 12.8-point oracle gap (61.4 → 69.7, oracle 74.2) with
compaction, and 4.8 of 11.6 without (59.2 → 64.0, oracle 70.8). But the
selector was trained on another model's rollouts **of the same 89 tasks**.
The paper's own 5-fold instance-level CV (Table 16) gives 64.4 and 60.7
for those two cells, 5.3 and 3.3 points lower. That turns a claimed
Opus-4.7 match into a result below GPT 5.3 Codex. So add to the
specification: **a selector's number is only a deployment estimate when it
was trained on disjoint tasks.** Cross-model training does not substitute
for task holdout.

[[literature/papers/wang2026rethinking]] gives the agent-trajectory version. On
5 Terminal-Bench samples a self-judge lifted pass@1 from 68.2 to 72.3. With
unit tests as the selector, the same budget reached 82.0. Harness evolution's
advantage appeared only in pass@5, so its gains are attempt-count gains.
Watch the column definitions: with tests, Parallel Sampling's "pass@1" equals
its pass@5 in every cell, so for that arm it is oracle best-of-5.

A majority vote does no better. [[literature/papers/bobadillasuarez2026audit]]
scores 30 one-vendor workers on 55 SWE-bench Lite tasks: Sonnet/Haiku 4.5 ×
3 prompts × 5 seeds, at temperature 0.25.
- Within a tier the vote equals the *average* worker (Sonnet 0.145 vs 0.147;
  Haiku 0.545 vs 0.565).
- Pooling tiers makes it worse (0.418 vs 0.356).

The abstract's "majority fails 23/55" figure is mostly this pooling artifact
(its own Appendix C).

## Report the repeat-run distribution before reporting an approximation error

A small but clean instance of this concept's rule, in an unusual place.
[[literature/papers/she2026efficient]]'s entire contribution *is* an error
bar — how far a benchmark subset's estimate sits from the full-run score —
and it never establishes the **test-retest noise floor** of the thing it is
approximating, despite flagging the agent's non-determinism twice. Its
headline 1.03 pp MAE is therefore uninterpretable in the one direction that
matters: if two full runs of the same agent differ by more than 1.03 pp, the
subset is already inside the noise and the comparison is free.

The rule generalizes beyond k: **any claim of the form "this cheap measure
approximates the expensive one to within ε" is empty until the expensive one's
own run-to-run spread is reported.** Same argument as the noise floor
[[concepts/budget-as-ceiling]] needs for `max_consecutive_no_improvement`,
applied to an instrument rather than a stopping rule.

## k bounds stochastic error, not systematic error (2026-09-28)

This concept's floor (k ≥ 3, report the distribution) treats run-to-run
spread as the uncertainty. [[literature/papers/kim2026divergent]] shows what
a large k with a tight distribution certifies when every run shares an
input. Sixteen replicas of one research-agent configuration, on one frozen
database, gave **15 of 16** the same wrong champion, a defective curated
entry. Their values spread by an SD of 0.12 cm³ cm⁻³, below the simulation's
own uncertainty. That is k = 16 and a near-zero spread on a wrong answer.
zhang2026double's "a tight distribution is not evidence of reliability" had
a scaffold-level cause (the model was not being measured). This is a second,
independent cause: **the model was measured, and the shared input was
wrong.** In the paper's words, "Replication reduced stochastic uncertainty
but not this systematic error."

It also sharpens the selector point above (kim2026are). Draw three of the
sixteen: "there was a 98% chance that at least one would report the leading
retained material, but certainty that a majority would name the excluded
entry." pass@3 over the legitimate frontier is near 1, and maj@3 picks the
common-mode error every time. On shared inputs, majority vote over k
replicas does not wash out the defect. It *selects* it.

Rule for this project: a distribution over k seeds of one
configuration on one input set supports a claim about **stability to
resampling**, and only that. A validity claim needs a varied input or source
(a second data split, an independently curated set, a different model),
which k does not supply at any size.

## Two marginals hide churn; unfired tasks are a free placebo (2026-09-29)

[[literature/papers/agarwal2026fire]] is a second pass@k-vs-passᵏ
separation after li2026acm, at k = 2. Harness runtime policies on
Terminal-Bench 2.1 raise Sol's pass∧2 from 64.4 to 73.6 while pass@2 moves
only 86.2 → 87.4. The paper's own task-transition table (its Table 6)
shows two things the marginals do not.

- **A flat pass@k can be net churn, not stasis.** Sol's +1 task of reach
  is 4 tasks gaining reach (0/2 → ≥1/2) minus 3 losing it (1/2 → 0/2).
  Its +8 in passᵏ includes 3 tasks that went straight from 0/2 to 2/2,
  which is not "reachable made repeatable". Report the baseline-by-
  treatment transition matrix of per-task success counts; pass@k and passᵏ
  are two projections of it.
- **Where an intervention fires on a known subset, the unfired tasks
  measure the noise floor for free** (my reading, not the paper's
  analysis). Terra's portfolio fired on 15 of 87 tasks, yet 32 tasks
  changed their success count (21 up, 11 down). So at least 17 tasks moved
  with no intervention in the treatment arm. For Luna, at least 13 of 25
  changed tasks were never touched. At k = 2 with cross-date runs, about
  one task in five moves from resampling and drift alone. That is the scale
  a suite-level passᵏ delta has to clear, and the paper reports no interval
  on passᵏ at all. Its randomized panel does the same thing by design: 16
  pre-registered silent tasks where the policy never fired (21/32 vs 20/32).

Also a caution on reading the separation as mechanism. The clean
"passᵏ up, pass@k flat" pattern appears only in the tier whose policies
fired on 81/87 tasks at +47.7% cost. The narrowly targeted tier moved
reach more than repeatability (+6.9 vs +5.7 pp). None of the three
suite-level pass@1 deltas is significant (p = .43, .17, .09).

## Split the hurdle, report absolute spread, not a variance share (2026-10-05)

[[literature/papers/wiedmann2026agents]] separates a binary "genuine attempt"
indicator from the continuous score given success (a Cragg-style hurdle).
Without the split, "rare but severe failures inflate MS_within far out of
proportion". Its headline "54% of variance is run-to-run noise" is an ICC
share over a deliberately wide 432-configuration grid; a narrower comparison
would show a larger noise share. The absolute figures are the ones that transfer:
- Per-configuration SD is about 11–13% of the reference-to-trivial range on
  well-behaved tasks (85% on the worst).
- The 95% CI half-width is still about 13% of that range at k = 5.
- Better configurations are less noisy (ρ = −0.79), so a spread measured on
  weak runs overstates noise near the frontier.

## An agentic selector harvests about 30% of the oracle gap (2026-10-06)

[[literature/papers/zhang2026veriharness]] gives the selector-vs-oracle gap
for a strong agentic verifier. The setting is ten rollouts per task on five
workspace benchmarks, with the verifier being the same model as the
generator. Its evidence-checking selector lifts the pool mean by +4.4 (Gemini
3.5 Flash) and +4.1 (Opus 4.8). That is 28–30% of the oracle headroom
(oracle 62.7 / 63.1). Its margin over the best competing selector is only
+2.1 for both models.

Within one model's samples, agreement is not correctness. This was measured
on APEX-Agents with Opus only:
- 34% of unanimous claims were wrong;
- among disputed claims, the modal value was right only 47% of the time,
  although 74% contained a correct candidate;
- on unanimous pools, selection gained +0.6.

This adds to the specification above. Report pass@k together with the
selector's capture ratio, (selected − mean)/(oracle − mean). Do not treat
maj@k agreement as a confidence signal. The gains rest on 3 verifier seeds,
and several runner-up margins sit within 1–2 SD.

## A normalized-ratio headline can hide a 7% median (2026-10-12)

[[literature/papers/cai2026forkandflush]] reports "+66% over single-run and
+44% over best-of-4" as a ratio of min–max normalized means across 13
tasks, with n = 5 per cell and no intervals. Min–max scaling stretches each
task to the observed range. A 0.3% raw gain (relay-lay) therefore weighs as
much as a 142% one (AHC014), and the ratio depends on which runs set the
range. The same fork-and-flush runs score 0.78 in one table and 0.71 in
another with a different pool. The median raw per-task gain is about 7% over
single-run and 4% over best-of-4, and one task (LWE-rec) goes backwards.
Rule for this project: report per-task raw deltas and their spread
alongside any normalized aggregate. Never quote a ratio of normalized
means as a relative improvement.

Two further reporting points from the same paper. Its best-of-4 baseline
splits a budget that was sized to the single-run plateau, so each draw
gets a quarter of it, and its max falls below single-run's on 7 of 13
tasks. Best-of-N at one N set by a budget tuned to another arm is a weak
comparator; sweep N. Its baseline counts are also inconsistent (5, 6, 7
or 8 runs depending on the table), and figures show excluded single runs
scoring above the reported single-run maxima. Report the per-arm k and
any exclusion rule next to the result.

## avg@k separates models that best@k ties (2026-10-13)

[[literature/papers/li2026beyond]] runs 7 frontier models × 36 AutoLab
tasks × 3 rollouts. The highest-to-lowest gap is 0.237 on avg@3 but 0.122
on best@3. Kimi's best@3 is within 0.03 of GLM and Gemini, while its
avg@3 trails both by about 0.07–0.10. Harness swaps move avg@3 by up to
0.055 and best@3 by at most 0.035. So the reliability axis is where models
and harnesses actually differ, and a best-of-k leaderboard hides it.

The paper reports no SE, CI or test anywhere, with k = 3 per cell. Its
own Table 2 reverses most harness effects at category level. Under this
concept's rule, the second-tier ordering (GLM 0.682, GPT 0.663, Gemini
0.652) is unresolved. Report avg@k and best@k together, with the per-task
spread. A model whose rank changes between the two metrics is a
reliability finding, not a capability one.

## Flip rate puts a rerun, an intervention and a model swap on one scale (2026-10-13)

[[literature/papers/liu2026what]] reduces any pair of runs to its flip
rate, (b + c)/n, the share of tasks whose outcome differs. That makes the
k ≥ 2 noise floor directly comparable to the effect being claimed. On
SWE-bench Verified hard45, reruns of one cell flip a median 13% (16% over
the ladder models), harness swaps 13%, and model swaps 22%. Equal churn
does not by itself rule out a directional effect, so the paper adds a
replication check: does a task split in run 1 split the same way in run 2?
For harness pairs, 5 of 12 did and 7 reversed (p = .77). For a model swap,
10 of 15 repeated. Rule for this project: when k = 2 is all the budget
allows, report the flip rate of a same-configuration pair next to the
treatment pair, plus the direction of repeated splits. Caveat: the
replication test rests on 12 tasks, so the pooled McNemar and TOST on
447 tasks carry the "no harness effect" conclusion.

## Choosing ideas: max-of-k is not better than mean-of-k, and the null is weak (2026-10-13)

[[literature/papers/ning2026implementation]] tests k as a *selection*
policy: rank four frozen ideas by one implementation, the mean of 3 or 5,
or the max of 3 or 5, then score the chosen idea against 12 fresh
implementations. On 18 tabular tasks and two executors, all four primary
contrasts (max3 − mean3, single − mean5) are null, Holm p ≥ 0.56. Max
aggregation is optimistic by construction: at equal means it favours the
higher-variance idea (E[max] = μ + s·c_J). A single draw picks a different
idea than a 6-program fitted leader in 19–26% of partitions, against
12–17% for mean-of-5.

Read the null with care. 8–11 of 18 tasks are exact ties in each contrast,
seven to ten tasks have zero loss under every policy, and several
baselines sit above 99%, where no idea can lose much. There is no
equivalence margin. The Agentic single − mean5 contrast leans toward
mean-of-5 (0.14 [−0.01, 0.31], 6/11/1). Rule for this project: report the
positive/tied/negative task counts and the minimum attainable p next to a
null selection contrast. A sign-flip test over 7 informative tasks cannot
show that k does not matter.

## Verified success stops at the selector's precision, and pooled verifier rates overstate it (2026-10-20)

[[literature/papers/iyengar2026sample]] makes this concept's
selector-in-the-specification point exact. Under uniform choice among
accepted candidates, one generator's verified success is
S_N = (a/s)(1 − (1 − s)^N). It rises to the verifier's precision a/s and
stops there, while coverage keeps rising. On MATH, majority vote over
Llama-3-8B-Instruct samples stays near 43% while coverage reaches 98%.

The paper then fits the selector's error rates from a 32-sample labelled
pilot on half the tasks and scores the other half. Three findings bear on
how this project would report or plan k:
- **Modelling correlation matters.** Assuming independent errors raises
  allocation regret 1.2–4.8× (e.g. 0.54 → 1.69 points on MATH).
- **Pooled verifier rates overstate gains.** Per-task judge TPR and FPR
  correlate with task difficulty (Spearman 0.54 and 0.65 on GPQA). Pooled
  rates predict +10.0 points from 9 judge calls at N = 64; the real gain is
  +4.8. Using the calibration tasks' own candidates predicts within 0.2.
- **A fixed acceptance threshold can make k = 1 worse than no verifier.**
  On SWE-bench Verified with an LLM patch verifier, a 0.5 threshold at one
  rollout lowers the resolve rate from 32.5% to 25.1%. It rejects correct
  patches with nothing to fall back on. The right threshold changes with N.

This adds the mechanism behind kim2026are, zhang2026veriharness and
bobadillasuarez2026audit. Their selectors capture little of the oracle
gap. Here the gap is a/s times coverage, and a/s can be estimated
before the run.

Caveats:
- The "within 0.02–0.65 points of hindsight-best" headline uses the best
  SPLIT variant per setting. The automatically chosen variant is
  significantly behind the better one on two of three datasets.
- With real CodeContests tests, SPLIT vs "strongest model" is not
  significant (p = 0.43). Under flat or latency pricing, strongest model
  wins by about 0.5 points.
- Agentic selection is one R2E-Gym agent at N ≤ 16. Conditional
  independence within a task is assumed, and converging agent rollouts
  break it.

Rule: when k is spent on a selector, report the selector's per-task
acceptance rates on correct and wrong candidates from a labelled pilot. Do
not pool them across tasks.

## Open questions

- The right k for MLE-bench-scale tasks is unsettled. FM Agent
  ([[literature/papers/li2025fm]]) and AIBuildAI
  ([[literature/papers/zhang2026aibuildai]]) do not specify seed
  protocols explicitly; their medal rates may not be directly
  comparable. NatureBench
  ([[literature/papers/wang2026naturebench]]) runs each of its twelve
  agent configurations exactly once over 90 tasks, so its 17.8%
  Surpass-SOTA headline is a k=1 estimate with no run-to-run variance
  treatment — its cross-model reproduce-mode agreement calibrates the
  SOTA anchors, not the variance of the agents being ranked. Expensive
  4-hour-per-task benchmarks are exactly where the k=1 temptation is
  strongest and where this concept's k≥3 floor remains unmet in
  practice. *Counterexample now attested:* AIRS-Bench
  ([[literature/papers/lupidi2026airsbench]]) runs every agent-task
  pair at ≥10 seeds despite 24h/H-200-per-run cost, reports 95% CIs on
  all headline metrics, folds failed/invalid runs in as normalized-0
  rather than dropping them, and aggregates rankings with an
  order-invariant Bradley–Terry Elo — the first expensive benchmark in
  this graph to exceed the k≥3 floor as protocol rather than
  aspiration.
- `status: experimental` here because this project has no tooling
  that enforces pass@k reporting yet. It is a recommendation backed
  by literature, not an enforced rule. Graduate to `active` when a
  downstream project successfully runs the protocol end-to-end.
