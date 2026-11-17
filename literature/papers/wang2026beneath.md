---
kind: paper
title: "Beneath the Surface: What We Overlook When We Talk About Agent Evolution"
authors: ["Xiaobo Wang", "Zimu Wang", "Siyu Ye", "Jiaqi Li", "Min Tang", "Jun Bai", "Yuling Shi", "Ruobing Wang", "Jingjing Zhang", "Chaofan Wang", "Kai Wu", "Mengnan Qi", "Kai Cai", "Xiaodong Gu", "Qi Liu", "Zilong Zheng"]
institutions: ["University of Science and Technology of China", "University of California, Berkeley", "Simple Agent Lab", "Beijing Institute for General Artificial Intelligence", "Shanghai Jiao Tong University"]
year: 2026
venue: "arXiv 2611.07624 (cs.AI), v1 2026-11-05, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.07624"
code_url: "https://github.com/simple-agent-lab/RSIHub"
citations: null
source: "raw/papers/wang2026beneath.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["harness-evolution", "agent-evolution", "rsi", "ahe", "hyperagents", "a-evolve", "gepa", "terminal-bench", "tau-bench", "coverage-churn", "task-turnover", "avg-at-k", "pass-at-k", "deployment-cost", "component-ablation", "generic-reminder", "interaction-effect", "parent-selection", "admission-gate", "regression-cap", "gate-replay", "transfer", "single-lineage", "no-null-churn", "rsihub-followup"]
---

# Beneath the Surface: What We Overlook When We Talk About Agent Evolution

## TL;DR

This is a follow-up to [[literature/papers/wang2026rsihub]] by the same group,
on the same grid: four harness-evolution recipes (AHE, HyperAgents,
A-Evolve, GEPA) × two harnesses (MiniSWE 2.4.5, source-editable; Codex
0.143.0, outer-layer-only) × two benchmarks (τ³-Banking, 97 tasks;
Terminal-Bench 2, 89 tasks), all on GPT-5.4. That gives 16 evolution
lineages, each run once for up to 20 generations. Each frozen Champion and
its Seed then get **eight full passes**, plus four diagnostic studies:
1. **Turnover.** All 16 Champions both gain and lose covered tasks
   (Pass@8), with gains of 1–15 and losses of 2–10. Reliability (Avg@8) rises
   in 11 of 16 and coverage in 10. 8 of the 11 reliability gains cost more
   per trial.
2. **Mechanism diagnostics.** In each of 4 four-Parent pools, the
   highest-scoring Parent is not the one whose children improve most.
   Evolved "Components" beat one generic reminder sentence in 15 of 16
   comparisons. Two HyperAgents Champions show significant negative
   interaction between their components. A stricter gate (strict gain plus
   a regression cap) keeps the best Candidate in 6 of 16 replayed pools,
   against 15 under the original rules.
3. **AGE.** The authors then revise A-Evolve's select, analyze, mutate and
   gate operators ("EvoAudit" → "AGE"). One 20-generation run per cell beats
   all four methods on Avg@8 in every cell.

The diagnostics are useful and mostly described honestly. Two cautions
matter for this graph. First, the turnover headline has **no Seed-vs-Seed
null**: most lost tasks were solved once in eight by the Seed. Second,
the eight-pass re-measurement **undercuts the RSIHub single-draw
headlines** that our wang2026rsihub note partly accepted (see Critique).

## Claims

- A higher score "do[es] not mean a more useful agent": it hides lost
  tasks, cost and off-distribution behavior. Report gains and losses
  separately, and reliability (Avg@k) separately from coverage (Pass@k).
- Efficiency is cost per attempt × attempts needed; a pricier Champion can
  reach a coverage level more cheaply.
- Gains often reach the held-out Test split; external transfer
  "selectively preserves gains".
- Current score does not identify improvement potential; specific changes
  beat generic guidance; changes interfere; a stricter gate can discard
  useful alternatives. Hence "evidence must match the decision": select,
  mutate and gate each need their own evaluation.

## Methods

- **Definitions.** Agent A = (M, H) with a frozen model. The recipe is
  seven operators: select, rollout, analyze, mutate, validate (optional),
  gate, record. Avg@k is the mean success over N tasks × k passes. Pass@k
  is the share of tasks with ≥1 success in k. Turnover counts tasks
  gained or lost at Pass@8 relative to Seed. Std@k is the population SD of
  per-pass rates. Cdep@k = k × mean cost per trial.
- **Recipes (Table 11; adaptations, as in RSIHub).** AHE: newest Parent,
  debugger reports, admits all. HyperAgents: score-weighted sampling,
  raw traces, may edit its own operators, admits all. A-Evolve: greedy
  Parent, reward-blind trajectory judge, strict-improvement gate. GEPA:
  per-task Pareto sampling, strict-improvement validate on the Train batch.
- **Splits.** τ³: 50/20/27 Train/Val/Test. TB2: 50/19/20. Table 3 reports
  each split's *contribution* to the full-benchmark delta, using the full N
  as the denominator. So 1 task = 1.03 pp (τ³) or 1.12 pp (TB2) of
  contribution.
- **Re-measurement.** 8 passes per Seed and Champion. That is 7,760 τ³ and
  7,120 TB2 executions; 776 or 712 logical trials per configuration. A
  pass-half (1–4 vs 5–8) stability check.
- **Transfer.** Frozen Champions with no re-evolution. τ³ → full τ²-bench
  (278 tasks, equal-weight mean over retail/airline/telecom, 8 passes). TB2
  → DeepSWE v1.1 (113 tasks, 8 passes).
- **Select study (D.1).** Each benchmark × harness pool holds the 4
  Champions as Parents. Each Parent spawns 3 branches through the *same*
  A-Evolve analyze/mutate, giving 48 Candidates. The best Parent maximizes
  mean child Test Avg@8 minus its own. The paper reports the hit rate of the
  HyperAgents, A-Evolve and GEPA select rules against a 25% random
  baseline. Test has 27 or 20 tasks.
- **Component study (D.2, D.3, D.5).** In 8 Champions (7 implementation-
  centric), the authors split the changes into two dependency-closed groups
  A and B, frozen before evaluation. They score Seed+A, Seed+B, Champion,
  and Seed + a generic reminder, all at Avg@8 on the full benchmark.
  - The τ³ reminder: "Inspect the available runtime tool interface, use
    tools when needed, verify the resulting state, and complete every
    requested item before ending."
  - The TB2 MiniSWE reminder adds: run "the closest exact end-to-end check
    on the final artifact" before submitting.
  - Interaction I = (S_AB − S0) − [(S_A − S0) + (S_B − S0)], with 95%
    task-paired intervals, on the four τ³ decompositions only.
- **Gate replay (D.4).** For each lineage, a pool of four frozen artifacts:
  Seed, Champion, last legal Candidate, and the Candidate nearest 50% of
  cumulative cost. These are replayed in a fixed diagnostic order (not the
  historical one) under the original gate and under a "Constrained" gate:
  strict mean improvement plus incumbent-relative mean regression L ≤ 0.05.
  Decisions use passes 1–2 on 20 or 19 decision tasks. The empirical best is
  scored on a separate 27- or 20-task reporting panel at 8 passes. Metric:
  H/N, the best-retained indicator divided by retained-set size.
- **AGE (§7).** Four revisions in A-Evolve, proposed by an agent workflow
  (EvoAudit), reviewed by 8 experts. Select breaks exact score ties by
  fewest Seed-relative per-task regressions. Analyze: reward-blind judge,
  Observed/Inference-separated verdicts. Mutate must state observed
  behavior → mechanism → replacement → activation conditions → expected
  trade-off, then make one dependency-closed change. Gate admits equal
  scores only with a verified cost reduction. 20 generations, search seed 0.

## Results

- **Table 1 (Avg@8 / Pass@8 Δ vs Seed, pp).**

  | Bench | Harness | Seed Avg / Pass | AHE | HyperAgents | A-Evolve | GEPA |
  |---|---|---|---|---|---|---|
  | τ³ | MiniSWE | 27.4 ± 3.1 / 54.6 | +5.4 / +11.3 | +5.2 / +3.1 | −1.2 / +1.0 | +0.8 / +4.1 |
  | τ³ | Codex | 30.3 ± 4.3 / 56.7 | +5.9 / +6.2 | +6.6 / +9.3 | −2.2 / +1.0 | −0.8 / −3.1 |
  | TB2 | MiniSWE | 65.9 ± 4.8 / 86.5 | +4.4 / +1.1 | +1.8 / −1.1 | +2.1 / 0.0 | +2.7 / +2.2 |
  | TB2 | Codex | 69.9 ± 2.3 / 89.9 | +2.1 / +1.1 | +2.7 / −3.4 | −2.0 / −2.2 | −1.8 / −3.4 |

  ± is the SD of per-pass rates. Reliability improves in all 8
  implementation-centric lineages, but in only 3 of 4 source-editable and
  0 of 4 outer-layer-only reflection-centric ones. 8 lineages improve both
  metrics, 4 move them in opposite directions, and 3 reduce both.
- **Turnover (Fig. 2).** Examples of gained/lost tasks: τ³ MiniSWE AHE
  15/4; τ³ MiniSWE HyperAgents 13/10, with reliability still +5.2 (the
  abstract's case); τ³ Codex GEPA 6/9; TB2 Codex HyperAgents 2/5. Reading
  the bottom row of Fig. 2: 7 of HyperAgents' 10 τ³ MiniSWE losses are
  tasks the Seed solved 1/8 times, and AHE's 4 losses there are at 1/8
  and 2/8. 6 of AHE's 15 gains are 0 → 1/8. The paper notes that "some
  aggregate directions differ between halves on τ³-Banking" in the
  pass-half check, and gives no per-lineage detail.
- **Behavior (Table 2; 22 extreme task–lineage pairs, agent-assisted
  coding).** Increases are mostly state/dependency management (7 of 13);
  decreases mostly exact grounding (wrong date, one wrong character; 4 of
  9) and premature completion (3), e.g. stopping after opening checking
  (5 → 0/8).
- **Cost (Table 13).** Mean cost per trial is $0.64–1.11. All 5
  reliability-improving τ³ Champions cost more. 3 of 6 reliability-improving
  TB2 Champions cost less, all on MiniSWE. τ³ MiniSWE AHE reaches 56.7%
  coverage at k = 4 (Seed: 54.6% at k = 8) at lower Cdep. Evolution cost
  is $723–2,373 per lineage.
- **Generalization (Table 3, contribution pp).** The Test ΔAvg
  contribution is positive in 10 of 16, ranging from −1.5 to +1.8.
  Multiplying by N/N_test (×3.6 for τ³, ×4.45 for TB2) gives the
  Test-split-only change; e.g. TB2 MiniSWE AHE +1.8 → ≈ +8 pp on its 20
  Test tasks (our arithmetic).
  - Transfer Pass@8 on τ²-bench falls in 7 of 8, but by only 0.3–1.0
    points from a 96–98% ceiling (Table 14). Transfer Pass@8 on DeepSWE
    rises in 7 of 8, by up to +8.0 (Codex AHE: 77.0 → 85.0).
  - Transfer Avg@8 is mixed: on DeepSWE, all four MiniSWE Champions lose
    0.2–2.3 points.
- **Select (Table 4, D.1).** Original rules hit the best Parent 17.5% and
  8.8% of the time on τ³ (MiniSWE, Codex), and 59.5% and 55.7% on TB2,
  against a 25% random baseline. In all 4 pools the top-Test Parent is not
  the best-improvement Parent. Best-Parent mean gains: τ³ −2.0 (MiniSWE),
  +19.6 (Codex); TB2 +4.0, +1.5.
- **Components vs reminder (Tables 5–6).** The reminder moves the Seed by
  +0.4 (τ³ MiniSWE), **−1.7 (τ³ Codex)**, 0.0 (TB2 MiniSWE; Pass@8 −2.2)
  and **−4.2 (TB2 Codex)**.
  - 15 of 16 Components beat the reminder. The exception is GEPA's shell
    adaptation (64.0 vs 65.9).
  - Against the *Seed* rather than the reminder, 11 of 16 Components
    improve Avg@8 (our count). Prose-only Components gained +4.5 to +6.9
    on τ³ but at most +1.8 on TB2. Both Codex TB2 instruction Components
    sat at or below Seed (70.2 and 69.1 vs 69.9).
- **Combination (D.3).** 6 of 8 Champions beat either isolated Component.
  The exceptions are a τ³ MiniSWE HyperAgents interface Component (34.4
  vs Champion 32.6) and a τ³ Codex HyperAgents orchestration Component
  (37.2 vs 36.9). In TB2 Codex HyperAgents, both parts fall below Seed
  (69.1, 68.0) but the whole reaches 72.6.
  - Interaction, AHE: 0.0 [−4.5, 4.9] (MiniSWE) and +2.1 [−2.2, 6.2]
    (Codex).
  - Interaction, HyperAgents: **−6.3 [−11.7, −1.2]** (MiniSWE) and
    **−6.6 [−12.6, −0.6]** (Codex).
- **Gate replay (Tables 7, 15).** The best Candidate is retained in 15 of
  16 pools under the original gates and 6 of 16 under Constrained.
  - Size-normalized retention: MiniSWE 31.3 → 25.0 (τ³) and 25.0 → 12.5
    (TB2); Codex 18.8 → 25.0 and 43.8 → 50.0.
  - The Codex increases come from pools where the empirical best is the
    Seed. In 3 of 16 pools the reporting-panel best *is* the Seed.
- **AGE vs methods (Table 10, Avg@8 / Pass@8 Δ; tasks gained/lost).**

  | Cell | AGE | A-Evolve | best other |
  |---|---|---|---|
  | τ³ MiniSWE | +6.6 / +11.3 (13/2) | −1.2 / +1.0 (7/6) | AHE +5.4 / +11.3 (15/4) |
  | τ³ Codex | +7.9 / +10.3 (15/5) | −2.2 / +1.0 (9/8) | HyperAgents +6.6 / +9.3 (11/2) |
  | TB2 MiniSWE | +5.9 / +3.4 (5/2) | +2.1 / 0.0 (2/2) | AHE +4.4 / +1.1 (4/3) |
  | TB2 Codex | +3.5 / +2.2 (3/1) | −2.0 / −2.2 (1/3) | HyperAgents +2.7 / −3.4 (2/5) |

  AGE's evolution cost ($983–1,481) is close to A-Evolve's. Its deployment
  cost is at or below Seed in 2 of 4 cells.
- **Expert review (Table 9).** 8 experts. Direction agreement averaged 6/8,
  evidence support 6.25/8, problem addressing 6.5/8, endorsement 6.75/8.

## Critique / open questions

- **No null for turnover.** "All sixteen Champions gain and lose covered
  tasks" is offered as a finding about evolution. The paper never runs a
  Seed against a second eight-pass Seed draw. Many losses sit at Seed
  1/8. If a task's true per-trial rate is ~1/8, the chance of 0/8 on a
  fresh eight passes is (7/8)^8 ≈ 0.34 (our arithmetic). So some turnover
  at the margin is expected with no change at all. The paper's own
  pass-half check says τ³ aggregate directions are unstable. Our
  wang2026rsihub note and agarwal2026fire (see
  [[concepts/pass-at-k]]) found 12–20% per-trial flips on reruns.
  The turnover *pattern* is plausible. The claim that every Champion
  really loses tasks is not shown. A placebo arm would have settled it,
  and the 8-pass ledger exists to compute one.
- **One lineage per cell.** Every Champion comes from one search run
  (search seed 0 for AGE; unstated for the others). The ±SDs are per-pass
  spread *within* one Champion, not across runs. The only intervals in the
  paper are the four interaction estimates. Many Table 1 deltas (+0.8,
  −0.8, +1.8, ±1.1 Pass) are inside one per-pass SD. The 2–4 point
  differences between methods are not shown to exceed run-to-run
  variation, which [[literature/papers/hu2026winners]] shows can be large
  at these split sizes.
- **AGE is designed on its own test bed.** EvoAudit read diagnostics from
  the same two benchmarks, harnesses and model, then AGE was run once per
  cell on them and compared against archived runs. There is no transfer
  evaluation of AGE, no second seed, and no fresh-sampling or best-of-K
  arm at matched spend ([[literature/papers/wang2026rethinking]]). "AGE
  improves over A-Evolve in all four settings" is four single runs against
  four single runs. The abstract's "achieving the best performance" claims
  more than that.
- **"Specific beats generic" is not a clean specificity test.**
  - The Components are evolved, benchmark-fitted content (a τ³ banking
    skill, JSON payload-formatting rules, a uv verifier bootstrap). Several
    are code, not prose. They were selected from Champions, and are scored
    on the full benchmark, *including the 50 Train tasks they were evolved
    on*.
  - The control is one fixed sentence: not tuned, not length-matched, and
    itself harmful on Codex.
  - What is shown: fitted changes beat an untuned generic sentence,
    in-sample. Against Seed, 5 of 16 Components do not help.
  - The intent-briefing reading is in the concept draft.
- **The RSIHub single-draw numbers, revisited (our cross-check).**
  Table 13's Pass@1 column (the first pass) reproduces
  [[literature/papers/wang2026rsihub]]'s single-run Seed scores exactly in
  all four cells (27.8, 24.7, 56.2, 69.7). It also reproduces 7 of its 16
  Champion scores. The paper does not say whether pass 1 *is* the RSIHub
  run.
  - RSIHub's headline +13.5 (A-Evolve, TB2 MiniSWE) becomes **+2.1
    Avg@8 and 0.0 Pass@8** here.
  - τ³ Codex A-Evolve goes from +4.2 to −2.2, and GEPA from +8.3 to −0.8.
  - The TB2 MiniSWE Seed's eight-pass mean is 65.9. RSIHub's three-rep
    RSIHub mean was 56.9, and the single draw was 56.2. So our RSIHub
    note's reading ("the seed draw is near its mean… +4/+5 is probably
    real") was wrong for that cell: the draw was ~10 points low.
  - Either the environment drifted between papers or the earlier
    replicates differ in condition. Neither paper reconciles them. The
    regression-to-the-mean reading in our RSIHub note holds more strongly
    than it said.
- **Transfer "coverage falls" on τ²-bench is at the ceiling.** Seeds are
  at 96.2% and 97.5%, and drops are 0.3–1.0 points of an equal-weight
  three-domain mean. That is a handful of tasks, with no interval. The
  DeepSWE coverage gains (up to +8) are the more informative half.
- **The gate replay is under-powered by construction.** The Constrained
  rule decides on passes 1–2 over 19–20 tasks. A single task falling from
  2/2 to 1/2 contributes 0.025 to L, so two such tasks reject the
  candidate. At these sample sizes the 0.05 cap mostly measures noise.
  The fixed diagnostic order (Seed → Champion → last → mid-cost) is not
  the historical order either. "Stricter admission discards useful
  alternatives" is shown for this rule at this n. It is not shown for
  regression caps in general. (Contrast [[literature/papers/so2026masked]],
  where the gate's failure is *masked*, i.e. invisible to the measured
  score; here the regressions are visible but noisy.)
- **Smaller points.** The select study's "below random on τ³, above on
  TB2" rests on which of four Parents won in each of four pools (27- or
  20-task Test panels). The behavioral audit codes 22 cherry-picked
  extremes with no inter-rater check. The 8 experts' relation to the
  authors is unstated, and they judged directions, not outcomes. There is
  no limitations section.
- **What it does not show:** that any recipe ranking holds beyond one
  lineage; that turnover exceeds rerun noise; that AGE's revisions
  generalize off the audited benchmarks; that a regression-aware gate with
  adequate n (e.g. more passes or a paired test) loses alternatives.

## Trust signals

- **Credibility:** 3 — Multi-institution (USTC, UC Berkeley, BIGAI, SJTU,
  Simple Agent Lab). The code repo is named (RSIHub; not opened at ingest).
  The scale of re-measurement is real: 8 passes, ~15k primary executions,
  ~31k transfer outcomes. Component boundaries were frozen before
  evaluation, and interrupted runs are disclosed. Against: not peer
  reviewed, 12 days old. One evolution lineage per cell and almost no
  intervals. No Seed-vs-Seed null for the headline turnover claim. AGE was
  designed and tested on the same benchmarks. The authors' own framework
  is the substrate. Its first-pass numbers silently coincide with the
  earlier paper's single draws, and the two papers' Seed means disagree by
  ~9 points with no reconciliation. Not 4 because the headline phenomena
  lack a null and replication. Not 2 because the measurement design (8
  passes, separated reliability and coverage, task-paired interaction CIs,
  frozen-pool replays) is sound and openly described.

## Follow-up

- **Relevance:** 4 — Material new evidence for
  [[concepts/evolutionary-expansion]]:
  - the first eight-pass re-measurement of four same-start evolution
    recipes;
  - a parent-selection study showing current score ≠ improvement
    potential;
  - a gate replay;
  - significant negative interaction between evolved components.
  It also corrects this graph's own reading of wang2026rsihub. It feeds
  [[concepts/pass-at-k]] (turnover reported as a transition matrix,
  Avg@k vs Pass@k, coverage-cost curves) and gives
  [[concepts/intent-level-briefing]] a hedged data point: an untuned
  generic "verify and finish everything" reminder *lowered* Codex
  reliability. Not 5 because one lineage per cell can't anchor a concept,
  and the turnover claim has no null.
- **For this box (our inference).**
  - `/iterate`'s noise band should be estimated from a repeated
    *baseline* at the k used for decisions, as this paper's Seed SDs
    (2.3–4.8 pp per pass) and the RSIHub reconciliation show.
  - A rising score with a per-task regression count is cheap to log.
    AGE's tie-break uses exactly that count.
- **Cheap check.** The released ledger should allow a Seed passes 1–4 vs
  5–8 turnover count per cell. That is the missing placebo for "all
  sixteen lose tasks".
- **Candidates** (cited, not yet in the graph):
  - Lin et al. 2026a, AHE; Zhang et al. 2026c, HyperAgents; Lin et al.
    2026b, A-Evolve; Agrawal et al. 2026, GEPA. All are already listed as
    candidates in wang2026rsihub.
  - Yue et al. 2025, small-k success vs large-k coverage in RLVR. Cited as
    the analytical frame.
  - Kim et al. 2025, interactions among model, harness and task structure.
