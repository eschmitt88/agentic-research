---
kind: paper
title: "RRSI: Regularized Recursive Self-Improvement of Agent Harnesses"
authors: ["Peng Xia", "Rujun Han", "Zifeng Wang", "Yanfei Chen", "Yufan Zhuang", "Yoonho Lee", "Chengsong Huang", "Han Yu", "Zhongying CuiZhu", "Yifei Ming", "Huaxiu Yao", "Burak Gokturk", "Tomas Pfister", "Chen-Yu Lee"]
institutions: ["Google Cloud AI Research", "UNC-Chapel Hill", "Stanford University", "Washington University in St. Louis"]
year: 2026
venue: "arXiv (cs.LG; v2, 2026-09-23)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.24972"
code_url: "https://github.com/google-research/rrsi"   # resolves (rrsi.py, rrsi/, domains/, tests/); project page regularized-rsi.com claims full round-by-round trajectories
citations: null
source: "raw/papers/xia2026rrsi.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: ["harness-evolution", "recursive-self-improvement", "regularization", "adaptive-overfitting", "noise-band", "calibrated-tolerance", "acceptance-rule", "edit-budget", "annealing", "structural-pruning", "leakage-critic", "ood-transfer", "cost-aware-selection"]
---

# RRSI: Regularized Recursive Self-Improvement of Agent Harnesses

## TL;DR

Automated harness evolution (propose edits to prompts, control flow, tools,
memory, subagents; keep the best on an evolve set) is adaptive reuse of a
finite, noisy test set, so it overfits. RRSI leaves the edit space open and
instead regularizes the **trajectory**: a cosine-annealed cap on how many
atomic edits a candidate may bundle (4 → 1), a proposer that conditions on
the full edit/score history and is redirected to untried component types
when progress stalls, an LLM critic that rejects diffs containing
benchmark-specific literals *before* they are scored, and a selector that
admits a candidate only if it clears a **noise band δ calibrated from
repeated runs of the unchanged base harness**, pays for any token growth
with gain, and passes domain guards. Stale components are flagged to the
proposer for deletion. On the agentic-workspace instance, the unregularized
loop reaches the highest evolve score (92.8) and an OOD average of 40.3,
within a point of no evolution (39.7), at 3.80M tokens/trial. RRSI reaches
90.5 evolve, 43.6 OOD, at 2.42M. The most useful measurement, which the paper
does not foreground, is that the **in-distribution held-out split barely separates
the methods (88.5–89.2) while OOD does (38.0–43.6)**. Every number is one
evolution run per arm, with no seeds or intervals.

## Claims

- **Harness evolution is adaptive empirical optimization.** Candidates at
  round t depend on scores from the same evolve tasks at earlier rounds, so
  apparent gains can be "benchmark-specific fitting, noise chasing, and
  complexity accumulation" (cites Dwork et al. 2015 on holdout reuse).
- **Regularize the search, not the hypothesis space.** Ω(H) stays open, and
  regularizers act on proposal and selection. The L0/L1/L2 labels are
  explicitly "analogous roles", not norm-penalized objectives.
- **Proposal side:** (i) edit cardinality `b_t = b_min + (b_max − b_min)·½(1 +
  cos(πt/T))`, so early rounds explore and late rounds are attributable;
  (ii) evidence-aware credit: every atomic edit is logged with its
  component, hypothesis, diff, ΔS, ΔC and accepted bit, so falsified
  hypotheses stay negative evidence; (iii) a stall (progress over w = 3
  rounds ≤ δ) reserves `m_draft = 1` proposal slot for components in the
  9-type vocabulary that no measured edit has touched yet.
- **Selection side (non-compensatory):** leakage critic before evaluation;
  floor `Ŝ(H′) ≥ S★ − δ` against the **running best**, not the incumbent,
  "prevents the search from walking downhill through a sequence of
  regressions that are individually small enough to be mistaken for
  noise"; if ΔS > δ then `ΔC ≤ β0 + β1·ΔS`; if ΔS ≤ δ, a shaped rule
  `w_s ΔS − w_c ΔC + w_n ν > 0`. The coding instance sets `w_s = 0`, so a
  within-band score gain alone can never admit a candidate. Only lower cost
  or a never-before-accepted structural component type (tool, skill,
  memory, subagent) can.
- **Pruning:** a component whose best measured gain over the last
  `n_prune` rounds is ≤ 0 is sent to the proposer as a deletion target.
  This is advisory; the proposer is "instructed to remove" it.
- Transfer holds under deterministic graders (EngDesign → Frontier-Eng), so
  it is "not an artifact of judge-mediated grading".

## Methods

- **Three domains, eight benchmarks.** Evolve on Terminal-Bench 2.1 (89
  tasks), Harvey LAB (120 evolve / 40 ID held-out; ~14,000 criterion
  verdicts, Gemini-3.5-Flash judge), EngDesign (61 license-free tasks,
  simulator-graded, no ID split). OOD, run unchanged: SWE-bench Verified;
  JobBench, GDPval (3-judge panel vs human expert), APEX-Agents (480 tasks,
  pass@1); Frontier-Eng (Medal score, 38 of 47 tasks contribute credit).
- **Policy and meta-agents** are all Claude Opus 4.8 (policy, proposer,
  failure analyst, leakage critic). A second coding run uses Gemini 3.5
  Flash as the policy.
- **Baselines** are Meta-Harness, AHE, TTHE and HarnessX, all run by the
  authors from the same H0 with the same frozen policy, evolve set and
  candidate budget. Reimplementation details are in Appendix B, which is
  prose only.
- **Hyperparameters (Table 5)**, coding / workspace / engineering: T = 20 /
  20 / 40 rounds; k = 2 / 2 / 4 trials per task; δ = 0.017 / 0.004 / 0.020;
  b_max = 4 / 3 / 4, b_min = 1; n_prune = 4 / 4 / 5. In the paper's units, δ
  is 3 passes of 178 trials, 60 criteria of ~14,100, and 5 passes of 244.
  All are "fixed without consulting held-out or OOD benchmarks".

## Results

- **Main (Fig. 3, RRSI vs H0).** Evolve: Terminal-Bench 74.2 → 80.2
  (+6.0), Harvey 89.4 → 90.5 (+1.1), EngDesign 50.0 → 54.9 (+4.9).
  Held-out: SWE-bench Verified +1.8, Harvey ID +2.3, JobBench +4.7, GDPval
  +3.5, APEX-Agents +3.7, Frontier-Eng 17.7 → 22.0 (+4.3, 24.3%
  relative). No held-out split regresses.
- **Vs baselines (Table 1, workspace only).** OOD average RRSI 43.6 vs H0
  39.7. Meta-Harness is +0.9 over H0, HarnessX is flat, and AHE and TTHE
  end below H0 (TTHE by 1.7). RRSI has the *lowest* evolve score of any
  evolved arm (90.5 vs 90.7–93.0). On the Harvey ID held-out split every
  arm lands at 88.5–89.2, and RRSI ties Meta-Harness at 89.2.
- **Ablation (Table 2, single run each).** w/o proposal regularizers:
  90.7 / 88.8 / OOD 41.9 / 2.69M tokens. w/o acceptance regularizers:
  91.5 / 88.7 / 41.0 / 3.59M. Unregularized: 92.8 / 88.9 / 40.3 / 3.80M.
  RRSI: 90.5 / 89.2 / 43.6 / 2.42M. Across arms, evolve score and OOD
  average move in opposite directions.
- **Policy robustness (Table 3).** Under Gemini 3.5 Flash, Terminal-Bench
  goes 64.6 → 78.7 (**+14.1**) and SWE-bench Verified 76.8 → 79.0 (+2.2).
  **Cross-model (Table 4):** that harness, run with the unseen Gemini 3.1
  Flash Lite, goes 11.2 → 14.6.
- **Cost (Fig. 4, workspace evolve split).** RRSI uses 2.42M tokens and
  26.3 steps per trial, against 1.56M and 21.2 for H0 and 27.3–34.6 steps for
  the baselines. AHE uses 3.82M, "58% more than ours, for 4.4 points less
  out of distribution".
- **Case study (Table 6).** Coding R0-A was accepted at +3.93. R0-B, a
  near-duplicate, was rejected at +1.69 (inside δ = 1.7 pts) with +26.1%
  cost. R8-B, which "pins the original task instruction into the completion
  gate", was rejected by the floor at −2.81 despite −13.6% cost.

## Critique / open questions

- **One run per arm, no intervals.** None of the comparative numbers has
  a seed count or CI. The paper's own δ gives the scale. On coding, δ = 1.7
  points against a +1.8 SWE-bench Verified gain (a different set, but the
  same noise order). On the workspace ablation, the OOD differences of
  1.7–3.3 points separating the arms are single draws on judge-graded
  benchmarks.
- **δ's estimator is unspecified.** The paper says the base harness is
  evaluated "repeatedly", but not how many times or which statistic (sd,
  range, max deviation) becomes δ. The workspace δ of 60 of ~14,100
  criteria treats criteria as the unit, but criteria cluster inside 120
  tasks. If the estimator is criterion-level, δ is probably understated.
  Harvey's +1.1 evolve gain is then only ~2.75δ.
- **The regularizers are ablated as two groups, not individually.** The
  edit anneal, credit history, stall exploration, critic, floor, cost rule
  and pruner have no separate contributions. The leakage critic's reject
  rate is never reported.
- **The critic targets literals, and zhu2026bad's shortcut class has
  none.** It rejects diffs that name tasks, entities, values or answers. A
  benchmark-wide protocol regularity contains no task name
  ([[literature/papers/zhu2026bad]]), so the critic cannot see it by
  construction. Only OOD evaluation can.
- **"Fewer tokens" is relative to unregularized evolution, not the
  base.** The evolved harness costs about 55% more than H0 (2.42M vs 1.56M),
  which the authors acknowledge. The margin over most baselines is small:
  read off Fig. 4a, Meta-Harness is at about 2.7M and TTHE about 2.5M.
  Token cost does not track transfer across methods either. TTHE is almost
  as cheap as RRSI and has the worst OOD average.
- **Baselines were reimplemented by the authors** under a shared budget,
  with no check that they reproduce their papers' numbers.
- **Minor inconsistencies.** §4.3 calls the cost-acceptance rule "the
  L1-style budget", but §3.3 labels it Ridge/L2. GDPval is "185 tasks",
  yet "each judge therefore issues 204 comparisons per harness", when two
  orders × 185 = 370. RRSI's Harvey evolve gain (+1.1) is smaller than its
  ID held-out gain (+2.3), and the paper leaves that unexplained.
- **Interesting single datum for [[concepts/constraint-pinning]]:** R8-B,
  which pinned the literal task instruction into the completion gate, was
  a −2.81 regression on Terminal-Bench. This is one noisy candidate and
  not evidence against pinning, but the pinned text went into a
  *completion gate*, not into context.

## Trust signals

- **Credibility:** 3. This sits at the top of 3. The prior is strong:
  Google Cloud AI Research with Pfister and C.-Y. Lee as senior authors,
  plus Stanford, UNC and WashU, and the first author is a UNC student
  researcher. The code repo resolves and has substance (`rrsi.py`,
  `rrsi/`, `domains/`, `tests/`), and the project page claims full
  per-round diffs and critic decisions. Hyperparameters were fixed on the
  evolve set. The design includes deterministic-grader domains and a
  cross-family policy check. It is held below 4 by the statistics. Every
  comparative claim is one evolution run with no seeds or CIs, δ's
  estimator is unstated, the ablation is two lumps, baselines are author
  reimplementations, and there are several small internal inconsistencies.
  It is an unreviewed preprint, days old. Trust the mechanism design and
  the direction of the ID-vs-OOD contrast more than any specific point
  gap.

## Follow-up

- **Relevance:** 4. It is the first system in the graph that
  **calibrates** its tolerance band from base-harness reruns and uses it
  at three decision points, which is the gap
  [[concepts/budget-as-ceiling]] recorded as open. It is also a
  released-benchmark measurement of task holdout failing to separate
  methods that OOD separates ([[concepts/hce-evaluation]]). It is not a 5
  because the statistical design cannot anchor a load-bearing number.
- **[[concepts/budget-as-ceiling]].** Floor against best-so-far minus δ,
  the `w_s = 0` rule that within-band gains earn nothing, and
  stall → redirect rather than halt. These go directly into the `/iterate`
  noise-band proposal. The next step is to state δ's estimator
  explicitly, which this paper does not.
- **[[concepts/hce-evaluation]].** ID held-out spread 0.7 points vs OOD
  spread 5.7 points across the same six arms.
- **[[concepts/evolutionary-search-grain]].** The annealed edit
  cardinality is the first attested *scheduled* grain. It is ablated only
  inside the proposal group, where OOD moves 43.6 → 41.9.
- **[[concepts/compression-as-generalization-test]].** This is not the
  wanted second mechanism paper. The complexity proxy is runtime tokens,
  not description length, and Fig. 4 shows it does not predict transfer
  across methods.
- **[[concepts/skill-library-lifecycle]].** The same retention logic
  (delete if no positive measured gain in a window) applies at harness
  component grain. The skill-grain sibling, SkillEvoReg (2609.30861:
  dropout, complexity regularization, counterexample validation), was
  declined from the 09-28 digest as same-direction. Revisit it if this
  concept's retention gate needs a second source.
- **Candidates.** These are the harness-evolution generalization papers
  RRSI cites that the graph lacks: Ding et al. 2026, a delta-attribution
  blog ("What evolves when we talk about harness evolution?"); Wang et al.
  2026b, "Rethinking the evaluation of harness evolution" (COLM workshop);
  Lin et al. 2026b, "Harness updating is not harness benefit"
  (2605.30621); HarnessCompass (2608.01918, generalization as an explicit
  objective); Luo et al. 2026, a gated quality-diversity archive
  (2607.13683); and Meta-Harness (Lee et al., COLM 2026), the strongest
  baseline and the only peer-reviewed one.
