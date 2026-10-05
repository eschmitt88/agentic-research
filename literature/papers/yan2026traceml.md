---
kind: paper
title: "TraceML: What Auto-Research Agents Miss in Long-Horizon ML Development"
authors: ["Jiarui Yan", "Weiwei Sun", "Sijie Li", "Wenhan Li", "Yiming Yang"]
institutions: ["Carnegie Mellon University"]
year: 2026
venue: "NeurIPS 2026, Track on Evaluations and Datasets (per the v3 page-1 footer); arXiv 2608.26086v3 (cs.LG, 2026-09-28)"
peer_reviewed: true
url: "https://arxiv.org/abs/2608.26086"
code_url: "https://github.com/JerryYan123/TraceML"
citations: null
source: "raw/papers/yan2026traceml.pdf"
added: "2026-10-05"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["trajectory-dataset", "human-agent-comparison", "kaggle", "mle-bench", "process-metrics", "pivot-rate", "solution-revisit", "setback-recovery", "stagnation", "redirect-before-halt", "codex-cli", "mlevolve", "planning-prompt", "human-distilled-skill", "prohibition-overshoot", "labeler-distillation"]
---

# TraceML: What Auto-Research Agents Miss in Long-Horizon ML Development

## TL;DR

A version-level trajectory corpus of 4,465 human Kaggle development
histories across 134 competitions. Every saved version carries its code,
score and timestamp, and every transition carries action, intent, edit size
and score-effect labels from distilled Qwen3-1.7B labelers. Seven of the
competitions are also worked by two agent scaffolds on the same
gpt-5.4-mini backend under a 12-hour budget: Codex CLI, a single
edit-run loop, and MLEvolve, an evolutionary tree search. The process view
finds that each scaffold settles into one band. Codex re-weights an
ensemble it never grows; MLEvolve mutates its model in place. Humans pivot
on 25% of transitions, Codex on 9% and MLEvolve on 58%, and only the human
pivots pay. Top humans return to an abandoned approach on 9.1% of eligible
versions; across all agent runs this happens once. A ~1k-token planning
prompt distilled from human practice moves the three behaviours it names
and improves scores on 5 of 7 competitions (single runs).

For this graph the important part is narrower than the digest framing. The
paper gives the **first human base rate for pivoting and for revisiting**
in ML development, and an operational definition of a pivot. It does
**not** measure redirect versus halt. The baseline prompt forbids halting,
and pivot rates are never conditioned on a stall. Its one instructed
anti-stall check ("stuck on the same task category for 3+ consecutive
steps?") went with *less* mode switching, not more.

## Claims

- **Outcome benchmarks hide the cause of the gap.** "They grade the final
  submission without seeing the sequence of edits behind it, so two runs
  with the same score look identical even when one experimented carefully
  and the other tuned blindly."
- **Each scaffold collapses into a narrow loop.** "Codex spends its steps
  re-weighting ensembles and tuning submissions, MLEvolve mutates its model
  in place, and neither pivots at the human rate nor reopens abandoned
  work." Neither changes direction or checks it: checkpoint swaps,
  pretrained-source swaps and re-running unchanged code to verify "all stay
  an order of magnitude below the human rate."
- **The scaffolds miss the human pivot rate from opposite sides.** "Codex
  rarely turns; MLEvolve turns without gain."
- **The agents' search has no memory.** "Recovering a score by tuning
  forward and returning to an earlier approach are different capabilities,
  and the agents have the first without the second." Together with the pivot
  result: "from a given state the agent does not turn, and it does not go
  back."
- **Ensembling is not one behaviour.** "a checklist asking only whether the
  agent ensembles would rank Codex above the top human cohort while its
  ensemble work does nothing."
- **Instruction reaches only part of the gap.** "A practice transfers when
  the instruction names a level the agent has not already passed. It does
  not transfer when the instruction is a direction with no destination,
  which is what a prohibition is, nor when the agent already stands beyond
  the human value." The abstract's form: "instruction closes only the part
  of the gap that reduces to instructions."
- **The fixes are design problems, not model problems (§6, untested).** For
  memory: "retrieval over a run's earlier states" is "a concrete thing to
  build". For control: "what is wanted is a controller that reads where the
  run stands rather than a stronger base model." Neither is implemented.

## Methods

- **Human corpus.** Reconstructed from Meta Kaggle and Meta Kaggle Code
  public notebook save-versions. Lineage is rebuilt as a DAG over notebook
  histories, forks and code similarity, with one canonical parent per
  version. Filters drop post-deadline edits, chains shorter than 5 versions
  or 3 days or with no score, and score-fishing resubmissions with unchanged
  code. 4,465 of 5,048 kernels survive. Medal rates are similar in the
  removed (43.3%) and retained (47.3%) sets, so the filter does not drop a
  weak tail. 23.7% of trajectories share near-duplicate code with another
  kernel in at least one version, and 8.2% have a fork parent.
- **Agent corpus.** 11 baseline Codex runs (codex-cli 0.146.0), 7 harnessed
  Codex runs and 13 MLEvolve searches, all on gpt-5.4-mini with the same
  task prompt and the same wall-clock and GPU budget. The paper says this
  "leaves search topology as the difference between them". Every agent
  version is re-graded with the held-out MLE-bench grader. Codex is tracked
  through sidecar git commits. MLEvolve journals are linearized into
  root-to-leaf branches (189 branches from 13 runs).
- **Unit alignment.** A human version is a deliberate Kaggle save. Its agent
  analogue is a submission-producing commit with adjacent identical states
  collapsed; in one Codex run "1,290 graded commits reduce to 16 versions".
  Median versions per trajectory are 20 for humans, 18 for Codex and 5 for
  MLEvolve. Behaviour-outcome analyses use within-competition percentile.
- **Schema and labelers.** Version state uses 8 coarse and 136 fine tags.
  Transitions get action (10 coarse, 85 fine), intent (6 classes),
  magnitude (4 levels) and score effect. A gpt-5.4-mini teacher labels
  curated traces and two Qwen3-1.7B students are distilled from it.
  Reliability (Table 9):
  - Teacher-to-student: coarse state F1 0.978, coarse action F1 0.733,
    intent accuracy 0.772.
  - Cross-model intent agreement: κ 0.611.
  - 100-item human audit of intent: 81% agreement (κ 0.68). It is no lower
    on agent transitions (Codex 85%, MLEvolve 75% at n = 20) than on human
    ones (80%).
- **Pivot.** Defined at the action level as "an edit that changes the
  backbone, the representation, the objective, or the validation scheme".
  Payoff is measured by coding each of the three steps after a pivot as +1
  (improving) or −1 (regressing).
- **Revisit vs recovery (App. D.3).** A *solution revisit* is a version
  whose state signature is closer to an earlier non-adjacent version than to
  its predecessor: Jaccard ≥ 0.6, a margin of 0.1, and something below 0.5
  similarity reached in between, so a plateau does not count. A *setback
  recovery* is a run that falls below its own best and climbs back, judged
  on scores alone.
- **Scopes (Table 6).** All §4 comparisons use the twelve-hour scope:
  7 competitions, 430 human trajectories, **10 Codex runs and 3 MLEvolve
  runs** (107 branches). Intervals use a two-stage cluster bootstrap over
  competitions, then human fork-lineage groups and agent runs, with
  MLEvolve branches that share nodes resampled together (B = 4000).
- **Harness experiment (§5, App. F).** Only the prompt varies, on Codex CLI
  with a 12 h budget on all 7 competitions. There are four arms:
  - baseline;
  - harness: a ~1k-token skill block of anti-loop prohibitions, human-prior
    practices, a 7-question self-check, and task-type priors, re-injected
    every 30 min;
  - Abl-A: a single content block;
  - Abl-B: the 30-min cadence with the planning content stripped.
  The skill operationalizes six features that correlate with final rank on
  humans (Table 16): K-fold, no plain hold-out, ensemble attention, early
  first ensemble, OOF predictions, and blending.
- **Baseline prompt forbids halting.** "There is always something worth
  trying next; don't exit voluntarily before the timer ends... If you think
  you're done, use the remaining time to try another backbone / more folds /
  more features / TTA / ensembling."

## Results

- **Action profiles (§4.1, Fig. 3).**
  - At coarse grain the separation is partial. MLEvolve-best sits
    0.09–0.12 bits JSD from every human cohort. Codex sits about as close
    to top humans as the human cohorts sit to each other.
  - At fine grain the separation is sharp. Coarse ensemble mass is 19.7%
    for Codex against 6.3% for humans. Model plus training mass is 38.4%
    for MLEvolve against 21.9% for humans.
  - Every gap survives three fairness treatments with CIs excluding zero
    (Table 11): scored-only humans, agents filtered by the human retention
    rules, and added new runs.
- **Pivot rates (§4.2).**
  - Per transition: **humans 25%, Codex 9%, MLEvolve 58%.** Matched to
    human versions in the same code state, "Codex is still out-pivoted
    three to one."
  - Post-pivot payoff, averaged over the three following steps: **matched
    humans +0.089, MLEvolve −0.008.** No figure is given for Codex in §4.2.
    §6 asserts that Codex "gains what a human gains on the rare occasions it
    does change direction".
  - Per-run Codex pivot rates (Table 7). The six runs longer than 90
    transitions sit at **2.0–4.8%**, and the pooled rate is **2.8%**. The
    four runs under 10 transitions read 0%, 0%, 57.1% and 62.5%.
- **Revisits vs recoveries (Table 14).**

  | | Top humans | Other humans | Codex | MLEvolve |
  |---|---|---|---|---|
  | Solution revisit, % eligible versions | 9.1% | 7.2% | 0.2% (1 of 658) | 0.0% (0 of 344) |
  | Revisits that beat the version returned to | 78.5% | 63.9% | n too small | — |
  | Setbacks recovered | 79.4% | 72.1% | **89.0%** | **41.1%** |
  | Recoveries that set a new best | 95.9% | 95.2% | 56.2% | 100.0% |

  The top-human rate predicts about 60 Codex and 31 MLEvolve revisits.
- **Ensembling (§4.4).** 78% of Codex's ensemble edits re-weight a member
  set it never grows. In human runs, an ensemble step that adds or changes a
  member raises the chance the next version improves by 6.4 points, and a
  re-weight-only step lowers it by 5.8. Neither kind moves the next-step
  odds for Codex.
- **Harness behaviour (Fig. 4; read as top-10% humans / Codex baseline /
  Codex + skill).**
  - Re-weight ensemble, % of transitions: 8.9 / 50 / 11.
  - Ensembling attention: 21 / 26 / 21. The text says the agent "shifts
    attention toward ensembling", but the bar actually *falls* onto the
    human value.
  - Small early edits: 43 / 5.6 / 61, an overshoot past humans.
  - Plain hold-out: 24 / 10 / 0. The prohibition overshoots.
  - K-fold: 51 / 57 / 59, and OOF: 33 / 79 / 83. Both were already
    saturated before the prompt asked for them.
- **What the prompt does not reach (App. F.3).** Dominant-working-mode
  switches per step are **2.2%** with the harness, against **7.1%** for
  baseline Codex and **13.2–14.1%** for every human cohort. The skill
  explicitly asks "Have I been stuck on the same task category for 3+
  consecutive steps?", yet mode switching fell. The paper calls the 2.2%
  "close to the 7.1% of prior Codex"; it is in fact further from the human
  value.
- **Harness scores (Table 18, best valid score per run).**

  | Competition (metric) | Baseline | Harness | Abl-A | Abl-B |
  |---|---|---|---|---|
  | commonlit (RMSE ↓) | 0.510 | 0.505 / 0.517 | 0.520 | 0.512 |
  | equity (C-index ↑) | 0.675 | 0.670 / 0.680 | 0.672 | 0.670 |
  | gquest (Spearman ↑) | 0.371 | 0.429 | 0.372 | 0.388 |
  | aes2 (QWK ↑) | 0.771 | 0.817 / 0.808 | 0.806 | 0.796 |
  | hms (KL ↓) | 1.050 | 0.718 | 0.795 | 1.377 |
  | ranzcr (AUC ↑) | 0.545 | 0.877 | 0.542 | 0.583 |
  | amex (Amex ↑) | 0.023 | 0.781 | — | 0.022 |

  - The noise floor comes from two duplicated harness cells: 0.012 RMSE and
    0.009 C-index, "roughly 0.01".
  - "Five of the seven competitions improve, two are within noise, and none
    regress."
  - Cadence alone (Abl-B) lands "at or below the baseline everywhere". A
    single content block (Abl-A) recovers part of the gain on aes2 and hms
    and none on gquest or ranzcr.
- **Harness scores as percentiles (Fig. 5, final percentile read off the
  plot, baseline → skill).** commonlit 0.61 → 0.63, equity 0.30 → 0.28,
  aes2 0.26 → 0.60, amex 0.00 → 0.33, gquest 0.59 → 1.00, hms 0.10 → 0.30,
  ranzcr 0.20 → 0.20. In most panels the harnessed run reaches its final
  value within the first ~15–30% of the budget and then stays flat.
- **What predicts rank, on humans (Table 16).** K-fold averaging (ρ −0.59
  paired, −0.35 humans-only), plain hold-out (+0.50 / +0.34), and early
  first ensemble. These are correlational and "co-vary with trajectory
  length".
- **Toolkit.** It has been applied to Codex CLI, MLEvolve, AIDE, Claude Code
  and Gemini CLI. A 1-hour Claude Code run (haiku-4.5) on commonlit reached
  RMSE 0.733, above 52% of humans, with 36% exploration intent against 8%
  for top humans.

## Critique / open questions

- **This is not a redirect-vs-halt measurement.** The baseline prompt
  forbids voluntary exit, so no arm halts. Pivot rates are unconditional
  per transition, or matched on code state, never on being in a stall.
  Figure 1's "pivots when behind" is a *stylized* panel, and no number in
  the body backs it. For the held redirect-before-halt idea, the paper
  supplies a base rate and a definition, not an effect size.
- **The instructed anti-stall check moved the wrong way.** Self-check item 7
  is the closest thing in the literature to a prompt-level
  redirect-before-halt rule. With it, mode switching fell from 7.1% to 2.2%
  (humans: 13–14%), and the harnessed runs flatten early in Fig. 5. One
  backend and n = 1 per cell. Still, it is the only direct datum on whether
  *asking* for a different direction produces one, and the answer here is
  no.
- **The agent sample is much smaller than "207 agent trajectories"
  suggests.**
  - The §4 comparisons rest on **10 Codex runs and 3 MLEvolve runs**. The
    189 MLEvolve "trajectories" are branches of 13 trees that share nodes.
  - Of the 10 Codex runs, four have 3–8 transitions. Those four are the
    commonlit, amex and equity runs plus one aes2 run. So the informative
    Codex runs cover only **4 of the 7** competitions: gquest ×2, ranzcr ×2,
    hms, aes2.
  - The authors cluster by run and flag the short runs. The abstract still
    says "seven" and "207".
- **The headline Codex pivot rate is not reconciled with the per-run
  table.** §4.2 reports 9%. Table 7 gives a pooled marginal rate of 2.8%,
  and every long run sits between 2.0% and 4.8%. Our own arithmetic: the
  unweighted run mean is 13.4%, so neither obvious aggregation gives 9%. The
  direction (Codex < humans) is robust either way. The size of the gap
  (roughly 3× vs roughly 9×) is not pinned down.
- **MLEvolve's zero revisits may be partly a linearization artifact.** In a
  tree search, "going back" usually means expanding an older node. That
  shows up as a *new branch*, not as a revisit along one root-to-leaf path.
  So "MLEvolve never reopens abandoned work" is measured with an instrument
  that cannot see the scaffold's native way of reopening. The 58% pivot
  rate may similarly reflect sibling-candidate generation. Codex's 1 in 658
  is the clean result.
- **"Agents recover scores" is Codex only.** MLEvolve recovers 41.1% of
  setbacks against 72–79% for humans. Codex recovers more often (89.0%), but
  only 56.2% of its recoveries set a new best, against about 95% for humans.
  It climbs back to where it was rather than past it.
- **"Only the human pivots pay" rests on a small effect.** +0.089 on a ±1
  scale over three steps means roughly 54% of post-pivot steps improve
  rather than 50%, if plateaus are excluded (our arithmetic). There is no CI in the main text and no Codex figure, yet
  §6 claims Codex gains what humans gain.
- **The score lift is weaker than "lifts scores" suggests.**
  - Single runs per cell, apart from two duplicated harness cells. The noise
    bound comes from those two duplicates and is not measured for the
    baseline.
  - Two baselines were effectively broken: amex 0.023 at the 0th
    percentile, and ranzcr AUC 0.545, near chance. Two of the five "wins"
    are therefore recoveries from failed runs.
  - In the paper's own stated performance measure (percentile, Fig. 5),
    ranzcr does not move (0.20 → 0.20) and equity falls slightly.
  - The real lifts are gquest, aes2 and hms, plus amex from zero.
- **The prompt is not purely distilled from the data.** Parts 1–3 follow the
  Table 16 correlates. Part 4 (DeBERTa/RoBERTa, TTA, groupby features) is
  generic Kaggle lore. The correlates are associations on humans, and they
  co-vary with trajectory length.
- **The human reference is not budget-matched.** Humans work for weeks;
  agents for 12 h. The paper says so, and treats humans as "a reference
  distribution, not a control". Per-transition rates depend on what a
  version is, though, and the agent collapse (1,290 → 16) is aggressive. The
  fairness checks (Table 11) bound this for the action-mass gaps, not
  explicitly for the pivot rate.
- **One small backend.** gpt-5.4-mini only. Whether a frontier model pivots
  or revisits more is open. The paper's "design problem, not model problem"
  reading is asserted, not tested across models.
- **The digest said "v2 revised 2026-09-28". The PDF is v3 of 2026-09-28.**

## Trust signals

- **Credibility:** 4. CMU (Yiming Yang's group), with the NeurIPS 2026
  Evaluations & Datasets track on the v3 footer. Everything is released:
  dataset (HF), labeler weights, code, and the exact prompts. Statistics are
  careful: run- and lineage-clustered bootstrap, GEE cross-check, three
  fairness treatments, a stratified human audit of the labels, and an
  explicit limitations section. The agent-side claims sit nearer 3. There
  are 3 MLEvolve runs and 6 informative Codex runs in §4, n = 1 harness
  cells, two broken baselines counted as wins, an unreconciled 9% vs 2.8%
  pivot figure, and one mini-scale backend. The 4 is earned by the human
  corpus and the instrument, not by the agent comparisons.

## Follow-up

- **Relevance:** 4. Material new evidence for the stall-handling thread in
  [[concepts/budget-as-ceiling]] and for [[concepts/evolutionary-search-grain]].
  It is the first human base rate for pivots (25% of transitions) and
  revisits (9.1% of versions, 78.5% of which beat the version returned to)
  in ML development. It is a second, independent observation, after
  [[literature/papers/min2026autonomous]], that agents never cross to
  structurally different moves (backbone or checkpoint swaps). Not a 5: it
  never measures halting, and its agent n is small.
- **Bearing on the held redirect-before-halt idea (`docs/system-proposals/_index.md`).**
  - *Supports the premise.* An unconditioned single-loop agent (Codex,
    the /iterate shape) under-pivots by roughly 3× in matched states and
    almost never returns to an abandoned line.
  - *Constrains the mechanism, in three ways.*
    1. Redirect must be defined structurally, not left to the model. The
       paper's pivot test is directly reusable: does the next candidate
       change the backbone, the representation, the objective or the
       validation scheme?
    2. Asking is not enough. The prompt-level "stuck 3+ steps?" self-check
       went with *less* mode switching. The paper's own prescription is "a
       controller that reads where the run stands", i.e. the stall counter
       must *force* the next candidate's category, not remind the agent.
    3. Redirect without memory is MLEvolve: frequent pivots, net payoff
       about 0. The redirect step should be allowed to pick a *previously
       abandoned* line from the run's own history (the 78.5% revisit win
       rate in humans), not only a novel one.
  - *Still missing.* A redirect-vs-halt ablation. This paper does not
    supply one, and its baseline prompt rules halting out by design.
- **Converges with** [[literature/papers/zou2026fmlbench]] (greedy until
  stagnation, then broaden) and [[literature/papers/chandran2026autoresearch]]
  ("paradigm shifts require external redirection"). It adds the human
  reference rate neither had. [[literature/papers/hu2026analyzing]]'s
  DevSkill 7 ("take one materially different action") is a prompt-level
  rule of the kind §5 finds reaches named practices but not rhythm. Note the
  contrast with hu2026analyzing's finding that developer-written skills
  *work* on cost: here a human-written skill works on the named features and
  on score, but not on direction-change dynamics.
- **For [[concepts/skill-library-lifecycle]].** A skill that names a target
  *level* transfers. A prohibition overshoots past the human value (plain
  hold-out 10 → 0 against a human 24). A prescription the agent already
  exceeds does nothing. Cadence without content is at or below baseline.
  Those are three testable admission criteria for a skill clause.
- **For [[concepts/selective-memory-retrieval]].** "Search without memory":
  the agent has its history on disk (git commits) and never retrieves an
  earlier state. The paper says retrieval over a run's earlier states is the
  thing to build, which is a stall-triggered retrieval case.
- **Candidates:** MLEvolve (Du et al. 2026, arXiv 2606.06473), RE-Bench
  (Wijk et al. 2025, human-agent time-budgeted pairs), CoMind (Li et al.
  2025, arXiv 2506.20640, same group), HORIZON / "long-horizon task mirage"
  (Wang et al. 2026, arXiv 2604.11978). The TraceML toolkit could be run on
  this box's own `/iterate` traces against the human cohorts.
