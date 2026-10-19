---
kind: paper
title: "Harness-Delta Attribution: Dissecting Gains From Harness Evolution"
authors: ["Wenxuan Ding", "Qin Lu", "Changlong Yu", "Sha Li", "Shuowei Jin", "Xin Liu", "Zhenyu Shi", "Greg Durrett"]
institutions: ["Amazon", "New York University"]
year: 2026
venue: "arXiv 2610.22466 (cs.AI), v1 2026-10-15, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.22466"
code_url: "https://github.com/Wenwen-D/HarnessDeltaAttribution"
citations: null
source: "raw/papers/ding2026harnessdelta.pdf"
added: "2026-10-19"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["harness-evolution", "gain-attribution", "overfitting", "benchmark-artifact", "shortcut-neutralization", "test-time-scaling", "compute-matched-baseline", "held-out-transfer", "validation-selection", "meta-harness", "skillopt", "ahe", "livemath", "alfworld", "swe-bench-verified", "create", "proposer-distillation"]
---

# Harness-Delta Attribution: Dissecting Gains From Harness Evolution

## TL;DR

Harness-Delta Attribution (HDA) is an evaluation protocol. It splits a
harness-evolution search-set gain ΔS = S(Ĥ) − S(H0) into three parts using
two constructed variants:

- **T (test-time scaling):** S(H0^scale) − S(H0). H0^scale is the baseline
  run at Ĥ's average inference budget, e.g. k samples plus majority vote or
  pass@k.
- **O (overfitting):** S(Ĥ) − S(Ĥ^neutral). Ĥ^neutral is the evolved harness
  with each *detected* benchmark-specific mechanism disabled by the smallest
  possible edit.
- **G (residual):** S(Ĥ^neutral) − S(H0^scale), so that ΔS = O + T + G.

The authors call O a lower bound on overfitting and G an upper bound on
generalizable improvement. They apply HDA to 25 settings: 3 frameworks
(Meta-Harness, SkillOpt, AHE), 11 executors and 4 benchmarks (LiveMath,
ALFWorld, CREATE, SWE-bench Verified). 21 settings have a significant
search-set gain. Across those 21, the mean capped shares are O = 54% and
T = 14%, and O + T > 50% in 14. The averages depend heavily on the
benchmark: ALFWorld and LiveMath are almost all O, CREATE is mostly T, and
SWE-bench is mostly G. Held-out gains are usually much smaller than
search-set gains. Choosing the harness by an aggregate validation score
helps in 10 of 16 settings but does not close the gap.

## Claims

- Harness evolution gains come from three sources that produce similar
  benchmark numbers but mean different things: benchmark-specific
  overfitting, extra inference compute, and generalizable improvement.
- "Among the 21 settings with statistically significant gains, HDA
  attributes at least 54% of the gain to overfitting and 14% to test-time
  scaling on average; their combined share exceeds 50% in 14 settings"
  (intro). The abstract states the same numbers without "at least".
- Different frameworks rediscover the same shortcut on their own. On
  LiveMath, one answer choice ("one of the remaining options is correct,
  but a stronger result can be proven") appears in 21 of 35 search
  questions and is correct in all 21. Meta-Harness wrote a string match
  for it, SkillOpt made it a default skill rule, and AHE added a prompt
  rule to choose it.
- "Held-out evaluation alone is not enough." When the artifact is shared
  across splits, an overfit harness transfers. ALFWorld's zero-LLM
  controller reaches 93.3% held-out success. On LiveMath, the 0.8B
  Meta-Harness executor scores 53/53 on test questions that contain the
  meta-option and 22/71 (31%) on the rest.
- A stronger proposer can distil benchmark knowledge into a weak
  executor's harness: Opus 4.8 writing deterministic ALFWorld controllers
  for 0.8B–35B Qwen executors. The authors present this as a separate
  overfitting route.
- "Validation-based signals are insufficient to prevent overfitting."
- Recommendation: report HDA alongside held-out results, account for
  test-time scaling and state cost limits during evolution, and use rich,
  low-artifact benchmarks with real distribution shift.

## Methods

- **Evolution loop (Alg. 1).** A proposer edits H using the history of
  versions, scores and traces stored on the filesystem. Ĥ is the argmax
  on the search set, with an optional validation set. Meta-Harness and
  SkillOpt use Claude Opus 4.8 as proposer with Qwen3/3.5/3.6
  (0.8B–35B-A3B) or Claude Haiku 4.5 executors. AHE uses Opus 4.8 as both
  proposer and executor, as in the original paper. Settings: 16
  Meta-Harness, 6 SkillOpt, 3 AHE.
- **Splits (App. B.2).**
  - LiveMath: 35/18/124 (SkillOpt's split). All three splits share the
    same construction, including the meta-option.
  - ALFWorld: 50 search games; 36 validation games covering all 6 task
    types but only some type–room pairs; 120 test games. The test set
    has new instances of the search types, 20 games in a held-out
    bedroom, and 3 unseen task types × 25.
  - CREATE: 120/80/561. 454 of the test items come from held-out
    relation types.
  - SWE-bench Verified: 48/24/50, seed 42, filtered to tasks resolvable
    within one hour.
- **Significance gate.** A paired bootstrap on H0 vs Ĥ search-set scores.
  A non-significant ΔS is reported as noise and gets no attribution.
  4 of 25 settings were not significant.
- **Identifying overfitting.** An LLM analysis agent follows the HDA skill
  (shipped as `src/hda/HDA_SKILL.md`). It audits the harness code, prompts
  and execution evidence for two kinds of mechanism:
  - **direct shortcuts:** no LLM call, constant answers, item-specific
    gates, lookup tables, string matching;
  - **benchmark-derived heuristics:** thresholds fitted to benchmark
    statistics, reverse-engineered construction rules, dataset priors,
    hard-coded recipes.

  For each flagged mechanism the agent makes the smallest edit that
  disables it and leaves everything else in place. A human reviews this
  at checkpoint 2.
- **Compute matching.** The agent proposes a cost measure (LLM calls or
  tokens) and an aggregation rule for H0^scale. A human approves it at
  checkpoint 1. If Ĥ uses no more compute than H0, then H0^scale = H0.
- **Reporting.** Each component is divided by ΔS. The O share is capped
  to [0%, 100%] and the T share to whatever remains, and G gets the rest.
  The raw signed components satisfy ΔS = O + T + G, but the paper does not
  tabulate them.
- **Validation arm (App. B.3, Meta-Harness only).** The proposer sees only
  the aggregate validation score, not per-instance results or
  trajectories. Both Ĥ (best on search) and Ĥ^val (best on validation)
  are scored on test.

## Results

- **Table 1 by benchmark** (O/T/G shares; per-benchmark means are our
  arithmetic from Table 1):

  | Benchmark | significant settings | mean O | mean T | O+T > 50% |
  |---|---|---|---|---|
  | ALFWorld | 6 (5 MH + 1 SO) | 89% (98% over the 5 MH) | 0% | 5/6 |
  | LiveMath | 7 (5 MH + SO + AHE) | 83% | 7% | 7/7 |
  | CREATE | 3 (2 MH + AHE) | 0% | 75% | 2/3 |
  | SWE-bench | 5 (4 MH + SO) | 5% | 6% | 0/5 |

  13 of the 21 settings are ALFWorld or LiveMath, so the headline 54% / 14%
  are unweighted means over a mix dominated by artifacts. The three
  numbers in the abstract check out against Table 1 (sum of O shares
  1133/21 = 54.0; sum of T shares 302/21 = 14.4; 14 settings with
  O + T > 50%).
- **ALFWorld.** All five Meta-Harness executors reach 100% on the search
  set. For the 0.8B executor the harness grew from 66 to 503 lines (7.6×)
  and makes no LLM calls (O/T/G = 100/0/0). SkillOpt's ALFWorld skill
  encodes the same recipes and priors in natural language (44/0/56).
- **CREATE.** On Meta-Harness Haiku 4.5, a compute-matched baseline
  recovers 49% of the gain (20 calls per item). On Qwen3.6-35B-A3B the
  baseline does better than the evolved harness (T = 100). AHE went from
  1.04 to 2.46 calls per item, and with k = 3 matching the split is
  0/75/25. SkillOpt's rollout is fixed at one call, so it cannot scale
  compute, and neither of its CREATE gains is significant.
- **SWE-bench.** This benchmark has the clearest residual. Qwen3-4B:
  23/23/54, made of a patch-size envelope fitted to the search set
  (ADD/DEL/DUP thresholds 12/8/3), 1.44× more LLM calls, and a no-op-edit
  guardrail. Qwen3-30B-A3B: 0/6/94, from an action-block parser guardrail
  with no benchmark content. Qwen3.6-27B and Qwen3-Coder-30B both have
  0/0/100 attributions and **both regress on held-out**.
- **Held-out transfer (Fig. 4).** Excluding ALFWorld, the held-out gain is
  smaller than the search-set gain in 13 of 15 settings and less than half
  of it in 8. Ĥ does worse than H0 on test in 4 of those 15. Table 1 marks
  5 regressions in total, including Meta-Harness ALFWorld 35B-A3B at −6.7.
- **Validation selection (Fig. 5, 16 MH settings).** Compared with Ĥ,
  Ĥ^val changes mean held-out performance by +8.0 pp on ALFWorld, +0.8 on
  LiveMath, +8.5 on SWE-bench and −0.4 CU on CREATE. It helps in 10 of 16
  settings and hurts in 6. In 14 of 16, Ĥ^val's held-out gain is still
  below Ĥ's search-set gain.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). The digest repeats the
  abstract's numbers exactly, and they match Table 1. The body narrows
  them in three ways the digest leaves out:
  1. "54% on average" is an unweighted mean of capped shares, and 13 of
     the 21 settings come from two artifact-heavy benchmarks. One is a
     synthetic simulator; the other has a single answer-choice artifact.
     On SWE-bench Verified, the most realistic task, O averages about 5%
     and no setting has O + T > 50%.
  2. "Overfitting" is the share an LLM auditor flagged and a human
     approved. It is not identified by an independent instrument.
  3. The residual is labelled an upper bound, but it is a point estimate
     by subtraction. It is a bound only if the auditor never over-flags
     and the compute match is exact.

  The validation sentence is fair as far as it goes. It was tested only
  as aggregate-score selection inside Meta-Harness, with small validation
  sets (18 LiveMath, 24 SWE-bench), and on LiveMath the validation set
  contains the artifact by construction.
- **How overfitting is identified, and the "lower bound" assumption.** O
  is a lower bound only if every neutralized mechanism really is
  benchmark-specific and disabling it removes nothing general. The paper
  gives no precision check for that:
  - no planted-shortcut validation (compare zhu2026bad's synthetic
    benchmark);
  - no second auditor or inter-rater agreement;
  - no count of mechanisms that humans rejected or edited at checkpoint 2.

  Some flagged items are arguable. SkillOpt's ALFWorld skill has "If an
  action returns 'Nothing happens', do not repeat it" highlighted as
  overfitting (Fig. 9), which reads like a general simulator rule. The
  object-location priors ("search countertops … before cabinets") are
  arguably world knowledge. If the auditor over-flags, O is overstated
  and G is understated, so the direction of the bound is not guaranteed.
  The [0, 100%] cap also hides cases where Ĥ^neutral scores below H0, and
  no raw signed components are reported.
- **T may be generous in places.** The compute-matched baseline can use
  pass@k. Fig. 12 says matching "raises the baseline from pass@2 to
  pass@3" on SWE-bench. Pass@k picks the winner with the oracle, so it is
  not a deployable baseline at the same budget, and in those cells T is
  an optimistic estimate. On CREATE, pooling answers is legitimate because
  the metric rewards the pool.
- **The decomposition depends on order.** T is measured from H0 and O from
  Ĥ, so any interaction ends up in G. An overfit mechanism that also
  saves compute (zero-LLM controllers) makes "compute-matched" ambiguous.
  In those cases H0^scale = H0 and the scaling question goes away.
- **There are no intervals on the components.** Only ΔS is significance
  tested. Each setting is apparently one evolution run with one Ĥ, so O,
  T and G have no seeds and no CIs. Splits are small (35 LiveMath search
  items, 48/50 SWE-bench search/test), which puts single-run component
  shares near the rerun-noise floor that liu2026what measured at n = 45.
- **Held-out vs search comparisons mix in distribution shift.** The test
  splits are deliberately shifted (ALFWorld's held-out room and task
  types; CREATE's held-out relations). "Held-out gain < search gain" is
  therefore partly a harder test set, not only overfitting. The paper
  treats it as confirming overfitting without separating the two.
- **What the paper does not show.**
  - That HDA's attribution predicts transfer. G = 100% settings regress
    on held-out, which the authors concede.
  - Any intervention during search that lowers O. HDA is post hoc.
  - Results for research-agent or ML-engineering tasks.
  - A comparison with wang2026rethinking's matched-budget sampling arms on
    the same settings.
- **Proposer distillation is a real confound for this project.** If the
  proposer already knows the benchmark (Opus 4.8 on ALFWorld), the evolved
  harness can encode that knowledge, and a held-out split from the same
  benchmark rewards it. `/elevate` and `/iterate` use a frontier proposer
  on tasks it may have seen. That raises the same issue.

## Trust signals

- **Credibility:** 3.
  - For: Amazon and NYU, with Greg Durrett as senior author. Public code
    (verified 2026-10-19): `src/hda/HDA_SKILL.md`, the paired-bootstrap
    `sig_test.py`, the Meta-Harness engine, and per-benchmark run folders
    with every proposed harness, `evolution_summary.jsonl` and selected
    frontiers. 25 settings over 3 frameworks and 11 executors. The
    mechanism excerpts are concrete and checkable. The authors state the
    limits themselves (O as a lower bound, G not a guarantee).
  - Against: no peer review. Single runs with no intervals on the
    components. Overfitting is identified by an LLM audit with unreported
    human-review statistics and no precision validation. Raw per-item
    logs and trajectories are not released (the README cites multi-GB
    size). The repo shows no license.
  - Not 4 because the central quantity (O) comes from an unvalidated
    judgment procedure with no uncertainty attached. Not 2 because the
    code and harness artifacts let anyone re-audit the flagged mechanisms.

## Follow-up

- **Relevance:** 4. This is the first source in the graph that splits a
  harness-evolution gain into overfitting, compute and residual on
  released benchmarks across several frameworks. It is material evidence
  for [[concepts/hce-evaluation]]:
  - It adds a **neutralization arm** next to wang2026rethinking's
    same-budget arm and liu2026what's rerun arm.
  - It gives a measured released-benchmark case for zhu2026bad's
    "task holdout is not protocol holdout": the LiveMath meta-option is
    53/53 on test vs 31% elsewhere.
  - It is a counter-example to SkillOpt's guards (bounded edits, no
    hard-coded values, validation gate) as anti-answer-key protection,
    because SkillOpt still wrote the meta-option rule (O = 88%).

  Not 5 because the overfitting share comes from an unvalidated audit with
  no intervals, and no task here is an ML-research loop.
- **For `/elevate` and `/iterate`.** Before crediting a self-modification,
  ask three questions:
  1. Does it spend more compute per call? Compare against a
     compute-matched rerun.
  2. Does it encode anything specific to the eval set? Grep for literal
     strings, thresholds fitted to the split, and lookup tables, then
     disable them and rescore.
  3. Does it survive a different benchmark?

  The HDA skill file could be adapted as a checklist. Not proposed here.
- **Candidates** (cited, not yet in the graph):
  - Lee et al. 2026, Meta-Harness (arXiv 2603.28052).
  - Lin et al. 2026a, AHE (arXiv 2604.25850).
  - Lin et al. 2026b, "Harness updating is not harness benefit"
    (arXiv 2605.30621).
  - Huang et al. 2026b, Evo-bench (arXiv 2608.09096).
  - Zhang et al. 2026e, HarnessCompass (arXiv 2608.01918).
  - Tu et al. 2026, Benchguard (arXiv 2604.24955).
  - Wang et al. 2026a, automated benchmark auditing (arXiv 2605.26079).
