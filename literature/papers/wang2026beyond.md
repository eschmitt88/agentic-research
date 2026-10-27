---
kind: paper
title: "Beyond Gain: Measuring Skill Internalization for Agent Skill Retirement"
authors: ["Yanze Wang", "Pengfei Yao", "Tianyi Sun", "Hongda Chen", "Yunyun Han", "Jun Sun", "Chuanrui Hu", "Yafeng Deng"]
institutions: ["EverMind", "Shanda Group", "Peking University", "Southeast University"]
year: 2026
venue: "arXiv 2610.32322 (cs.AI), v1 2026-10-23, preprint (ACM format, venue not named; 12 pages incl. appendices)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.32322"
code_url: "https://github.com/EverMind-AI/SkillCorpus"
citations: null
source: "raw/papers/wang2026beyond.pdf"
added: "2026-10-27"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/shared-skill-namespace]]"
tags: ["skill-retirement", "internalization", "gain-score", "uncovered-state", "level-rule", "band-rule", "validity-gate", "witness-model", "binary-collapse", "generalizability-theory", "parallel-forms-reliability", "tasks-over-runs", "per-model-verdicts", "skill-hub", "skillsbench", "tessl", "task-synthesis", "llm-judge", "self-judging", "judge-audit", "lords-paradox", "soft-retirement", "qwen", "open-weight"]
---

# Beyond Gain: Measuring Skill Internalization for Agent Skill Retirement

## TL;DR

The paper asks when a skill library should retire a skill for a given model.
Most published rules run the agent with and without the skill and retire when
the gain (aided minus unaided score) is near zero. The gain is also near zero
when the model fails with *and* without the skill. The authors call that state
**uncovered**: neither the model nor the skill delivers. A gain rule cannot
tell it from **internalized** (both arms pass). The paper retires on the
unaided score instead (the **level rule**, U ≥ 0.8). It trusts a high U only
when some other "witness" model gains from the skill on the same task (the
**validity gate**).

Main measured results:

1. **The published band rule (|gain| < 0.10, from "Not All Skills Help")
   removes 57.5% [46.2, 68.7] of uncovered skills** with one task per skill
   and Qwen3.6-27B as target. Under each of the four models as target it
   removes 58–65% on capability scores and 80–99% on binary all-items
   outcomes. The level rule removes none by construction. Read on a different
   task than the state (cross-fitted, n = 29 decisions), the band removes 45%
   and the level rule 14%, a paired difference of −31 points [−52, −7].
2. **The gate matters only at the top.** It admits 72.4% of tasks overall but
   52.6% of those with U ≥ 0.8. So nearly half of apparent internalization
   comes from tasks no model benefits from.
3. **On SkillsBench, 28 of the native band's 31 retirements are binary
   collapse.** These are probes where no trial in either arm ever passed every
   item, so both pass rates are 0 and the gain is 0. Those probes average
   U = 0.555 on capability items. **But deleting the retired skills costs no
   detectable reward under any rule**, and the level rule and the band do not
   differ (Δ_L − Δ_Bbin = −0.8 points [−3.1, 1.7]).
4. **Tasks, not runs, carry the noise.** Test–retest over runs is 0.90.
   Parallel-forms over a second synthesized task is 0.41. Task variance is 39%
   of a single run's variance and run variance is 27%. At three runs per
   skill, three tasks × one run misclassify 11.4% and one task × three runs
   16.1%. The level verdict reproduces across tasks better than the band's
   (κ 0.42 vs 0.23, paired CI on the difference [0.01, 0.38]).
5. **Verdicts are per model.** 37% of skills get contradictory verdicts
   somewhere among the four models. The skill × model interaction explains 15%
   of U variance, and model main effect 0.4%. So shifting the threshold per
   model would not fix it.

The diagnosis is sound and the measurement is unusually careful. What it does
not show is that retiring on the level rule beats the gain rule on any
downstream outcome. Also, Tessl's original two-part condition (gain band
*plus* A ≥ 0.7) does as well as the level rule. The fault lies with gain-only
rules, not with reading the gain at all.

## Claims

- Retirement is "a per-model certificate of unaided capability". A retired
  entry stops being retrieved for that model, so the error is not corrected
  by later use, and an uncovered skill loses the repair flag it should get.
  The costs (c_R for certifying a capability the model lacks ≥ c_K for
  withholding one it has) are a definition, not a measurement.
- **Proposition 1.** Any criterion φ with φ(p, p) = 0 gives one verdict on
  the whole diagonal p_A = p_U. That covers the difference, the log-ratio,
  arm–outcome mutual information, and single-skill leave-one-out and Shapley.
  Such a criterion cannot tell uncovered from internalized. This is close to
  a tautology once stated. The paper frames it as Lord's paradox: a
  difference and a level answer different questions.
- **Proposition 2.** With more runs the band converges to 1[|p_A − p_U| < δ],
  so replication removes the noise in its error, not the error.
- **Proposition 3.** Error variance is σ_t²/n_t + σ_r²/(n_t n_r). At fixed
  n_t it cannot fall below σ_t²/n_t, and the optimal runs per task is
  n_r* = √(σ_r² c_t / σ_t² c_r) = 0.83 √(c_t/c_r). So one run per task is
  optimal unless generating a task costs more than about three runs. Extra
  tasks belong near τ: a skill 0.2 from τ needs about four tasks and one 0.3
  away at most two (α = 0.1, one run per task).
- The gain suits **ranking** skills, where agent strength and judge leniency
  cancel in the difference (they cite Kevin et al. and SLIM). It does not
  suit **retiring** them, where the agent's unaided strength is the quantity
  being measured.
- Recommendation (Algorithm 1): measure U on capability items with three
  runs; if |U − τ| ≤ 0.2 add two same-batch tasks; consult witnesses only
  before retiring; regenerate the task once if no witness gains; run the
  aided arm only when U < 0.4, to flag uncovered skills for repair. Use a
  high τ and a reversible soft retirement.

## Methods

- **Adversarial set.** 3,000 skills drawn uniformly from SkillCorpus (the
  authors' own public corpus, ~10⁵ entries). For each skill DeepSeek-V4-Flash
  (a different family from the target) synthesizes one executable task with
  hidden rubrics in Tessl's format. The prompt asks it to state the goal but
  not the exact API, parameter, mapping or ordering the rubric tests. Median
  task: 234 words, 78% with input files, 10 rubric items. A completeness
  critic rejects tasks that cannot be performed (708 dropped). A difficulty
  gate regenerates tasks the target passes on more than half the rubric
  unaided, and drops the skill if the retry is also too easy (859 dropped).
  The critic and the gate are Qwen3.6-27B, the target. Counts: 1,323 scored
  once → 1,251 with both arms × 3 runs (released) → 1,165 computable → 844
  past the validity gate.
- **Models.** Target Qwen3.6-27B. Witnesses and alternative targets:
  Qwen3.8-27B, Qwen3.5-397B, DeepSeek-V4-Flash. All open-weight, three of
  four Qwen. Single-agent tool-use loop with shell, ≤ 40 turns. Unaided arm 3
  runs for every model; aided arm 3 runs for the target, 2 per witness.
- **Judge.** One judge, **Qwen3.6-27B (the target itself)**, scores every run
  of every model and arm.
- **Capability items.** A text adjudicator (Qwen3.5-397B) keeps items whose
  pass needs information stated in the skill that a competent engineer could
  not reliably supply: 69.2% of items (median 8 of 10). Against 120 items
  annotated by the authors it has 0.85 precision and 0.81 recall. Scoring all
  task-completion items instead correlates 0.93 with the capability score,
  and the audit reproduces on it (band still retires 51.7% of uncovered).
- **States.** High = 0.7, low = 0.4. Internalized: A, U ≥ 0.7. Skill works:
  A ≥ 0.7, U < 0.4. Uncovered: A, U < 0.4. Level rule τ = 0.8. Witness gate
  γ = 0.1 (max over witnesses of A_w − U_w).
- **Audited rules.** gain < 0 (SKILLC-like), gain ≤ 0 (SKILL0-like),
  |gain| < 0.10 on capability scores and on binary all-items passes
  (ASSAY / "Not All Skills Help"). With one task per skill, ASSAY's step that
  splits high-spread skills into conditional variants cannot run; the paper
  argues it would not reach uncovered skills anyway. SLIM, SkillOps, Library
  Drift and SkillOS are discussed but not audited, because their guards use
  in-deployment evidence.
- **Reliability.** 135 skills got a second task from a later generation
  batch (124 scored on both). A pilot gave 40 mid-range skills four extra
  same-batch tasks.
- **Judge checks.** DeepSeek-V4-Flash re-scored 444 unaided trajectories.
  Claude Opus 5.5, blind to the judge's marks, regraded 99 items near the
  threshold on the evidence the judge saw.
- **SkillsBench execution.** One probe per SkillsBench skill (200 → 182
  scorable). This judge runs code in a sandbox and recomputes numeric and
  set-valued items. Witnesses: Qwen3.6-35B-A3B and Qwen3.8-27B. Before
  running, the analysis was fixed to the 92 probes that give no rubric item
  away in the task text; the gate admits 59. Each rule's retired skills are
  deleted from every SkillsBench task that ships them. Cost Δ is mean reward
  lost over 81 tasks (2 excluded in advance because their tests check the
  skill's procedure, not the output). At least 5 runs per touched task per
  deletion, 9 for the full library, 2,000-draw task bootstrap. The primary
  comparison Δ_L − Δ_Bbin was fixed in advance.
- **Replication.** Tessl-single: 849 tasks over 414 skills, both arms × 3,
  669 scorable tasks over 354 skills, ungated (no witness runs).
- **Retrieval check.** A dense retriever plus reranker ranks each task's own
  skill among 10,000 distractors; utility is the skill's held-out gain when
  it ranks top-3.

## Results

- **Table 1 (target Qwen3.6-27B, gated adversarial set).**

  | Rule | retired share of library | uncovered share of retired | uncovered removed | internalized removed |
  |---|---|---|---|---|
  | gain < 0 | 9.4% | 11.4% | 12.3% | 10.7% |
  | gain ≤ 0 | 25.0% | 18.5% | 53.4% | 45.4% |
  | \|gain\| < 0.10 | 27.8% | 17.9% | 57.5% | 61.3% |
  | U ≥ 0.8 | 26.4% | 0 | 0 | 77.1% |

  On Tessl-single the band retires 15.0% and removes 45.2% of uncovered; the
  level rule removes 67.1% of internalized. The uncovered state is 8.6% of
  the gated adversarial library and 5.9% of Tessl-single. It is 2.1× as
  dense inside the band as in the library. Within the band U spans 0 to 1
  (SD 0.331). The level rule removes 15.9 points [10.9, 20.7] more of the
  internalized state than the band.
- **Restoring Tessl's level half.** A band that also requires A ≥ 0.7
  removes no uncovered skill under any of the four targets, and 7% in the
  cross-fitted audit. Its cross-task κ is 0.47 against the level rule's 0.42.
  Every skill it retires has U ≥ 0.61. It keeps 49 internalized skills the
  level rule retires, 39 of which the skill still lifts by ≥ 0.1.
- **Across targets (Table 3).** Uncovered base rate 8.6–15.9%. The band
  removes 57.5–65.4% of uncovered on capability scores and 79.5–98.7% on
  binary outcomes. The level rule removes 13–16 points more internalized.
  The set was made adversarial only for Qwen3.6-27B, so the other rows are a
  partial out-of-design check.
- **Difficulty-gate correction.** Regenerating ungated tasks for 115 of the
  859 dropped skills: 69.6% reach τ unaided (vs 40.4% of retained).
  Post-stratifying lowers the uncovered base rate from 8.6% to 7.0%, and the
  band still removes 57.2% [45.7, 68.3] of it. The band's retirements rise
  to 40.3% of the library.
- **Gate.** Admits 844 of 1,165 tasks (72.4%); 52.6% of the 424 with U ≥ 0.8;
  76–89% elsewhere. Two witnesses recover 85–90% of what three admit, one
  witness 61–74%. The strongest witness admits the fewest high-U tasks. A
  text-only predictor separates high from low U at AUC 0.798 ungated and
  0.656 gated, and at 0.575 [0.470, 0.673] on the 160 skills with the
  strongest witness evidence.
- **Reliability.** Internal consistency 0.907 (upper bound), test–retest 0.90
  at three runs, parallel-forms 0.41 (0.364 on Tessl-single). Variance
  components: skill 0.046, task 0.053, run 0.036. Same-batch task pairs
  correlate 0.549, different-batch 0.285. Cross-task reproducibility of the
  gain is 0.15. The gain's cross-task covariance is 0.011 after the two arms'
  shared 87% cancels. Adding tasks only within 0.2 of τ (41% of the library)
  reaches 11.1% misclassification at three quarters of the cost of tripling
  tasks everywhere. Run-level state stability: an uncovered skill moved to
  internalized on a held-out run in 2 of 205 cases.
- **Judge.** DeepSeek re-scoring: r = 0.89, mean difference 0.009, same
  retirement verdict on 91.4% of trajectories, disagreements split 20/18.
  Opus 5.5 audit: 9% of the judge's decidable full marks unsupported (15%
  counting partly met criteria), mostly where a number had to be recomputed;
  2 of 20 zero marks wrong. These unsupported marks lift U near τ by about
  0.1, which the paper notes is the margin between τ and h.
- **Item validity.** 10.0% of the 6,802 capability items in gated tasks are
  passed unaided by all four models on every run. They sit in 29.4% of
  skills (median 30% of an affected skill's items); no skill has all its
  items in this class.
- **Across models.** Pairwise verdict flips 14–23%; 37% of skills
  contradictory somewhere among the four; resampling one model's runs flips
  5–9%. 19–32% of what any rule retires under one model is a skill that
  works for another. Re-labelling for a new model reuses tasks, items and
  witness evidence: 61% cheaper than labelling from scratch.
- **SkillsBench (Table 4, 59 admitted clean probes).**

  | Rule | retired | tasks | slots | shared with g_L | Δ (pts) | interval |
  |---|---|---|---|---|---|---|
  | level g_L | 18 | 17 | 18 | — | −0.2 | [−2.7, 2.4] |
  | native binary band | 31 | 25 | 29 | 10 | +0.6 | [−1.7, 3.1] |
  | band, all items | 15 | 14 | 14 | 7 | −0.5 | [−2.0, 0.7] |
  | band, volume-matched | 18 | 17 | 17 | 9 | −0.8 | [−3.2, 1.7] |

  Deleting every skill costs 19.1 [11.3, 27.0], so the benchmark can
  register loss. Per touched task, the level rule's 95% upper limit is about
  11 points. Full passes are rare: 10% of unaided runs and 37% of aided. On
  probes that give an item away, the level rule retires 43% against 31% on
  clean probes.
- **Retrieval.** The level rule retires 19–29% of the library across three
  models and keeps 91–97% of retrieval utility, against 70–80% for random
  retirement of the same size.
- **Automatic retrieval on SkillsBench** (83 tasks, 2 harnesses × 2 models):
  6% of pairs pass both arms, 13% rescued by the skill, 4% broken, 77% fail
  both. So the gain is zero for 83% of pairs, nearly all of them double
  failures.

## Critique / open questions

- **Against the digest framing: at framing.** Every number in the digest
  entry is in the PDF and is quoted correctly. The digest leaves out three
  things that limit how far the result reaches.
  1. **No downstream outcome separates the rules.** On SkillsBench no rule
     shows a cost, and the pre-fixed primary comparison is −0.8 [−3.1, 1.7].
     The case for the level rule rests on *which* skills each rule removes
     and on cross-task κ. The κ gap's interval, [0.01, 0.38], barely clears
     zero. The cost of retiring an uncovered skill (losing the repair flag,
     withholding from other models) is defined in the loss function, not
     measured.
  2. **A gain rule with a level guard does as well.** The band restricted to
     A ≥ 0.7 (Tessl's original two-part condition) removes no uncovered
     skill and reproduces at κ 0.47. The title's "beyond gain" is really
     "beyond gain *alone*". The two rules then differ mainly on internalized
     skills the skill still lifts, which the level rule retires and the
     restricted band keeps. Which is right depends on c_R vs c_K, and that is
     not measured either.
  3. **"Over half" is under an adversarial, one-task design.** The set was
     built so the target fails unaided (difficulty gate), which inflates the
     uncovered and in-band mass. The 57.5% share is conditional on the state,
     so it barely moves under the gate correction (57.2%). The volume shares
     do move: the band's share of the library goes from 27.8% to 40.3%.
     ASSAY's own guard (splitting high-spread skills) cannot run with one
     task per skill. The paper argues it would not catch uncovered skills
     anyway; that is reasoned, not tested.
- **The digest's ingest check, task synthesis quality: mixed, and the paper
  measures most of it.**
  - The synthesized task is the largest noise source. Parallel-forms
    reliability is 0.41, task variance is the largest component, and
    different generation batches correlate only 0.285. Across two tasks the
    level verdict agrees with itself 73% of the time (κ 0.42), the band's
    62% (κ 0.23). The paper's remedy (extra same-batch tasks near τ) raises agreement
    *within* a batch. It may also share the batch's bias, which the
    0.549-vs-0.285 gap suggests is real.
  - Leakage is common. In the SkillsBench probes, 90 of 182 (49%) gave a
    rubric item away in the task text, and give-aways raise U (43% vs 31%
    retired). The adversarial set's give-away rate is not reported. There,
    leaked items are only dropped if the adjudicator flags them.
  - Non-diagnostic items survive. 10% of capability items are passed by
    every model on every run, concentrated in 29% of skills. These inflate U
    for the skills where they sit.
  - The critic and the difficulty gate are the target model. The
    adjudicator is only checked against 120 items the authors annotated
    themselves.
  - So task synthesis is good enough for library-level shares and weak for
    per-skill verdicts. Near τ a single task decides badly, which the paper
    says itself.
- **The target judges itself, and the audit cuts against the level rule
  where it matters.** The DeepSeek re-score shows no *systematic* self-bias
  (r = 0.89, 20/18 split). But the Opus audit found 9–15% unsupported full
  marks near τ, worth about +0.1 on U, mostly on items needing recomputation.
  A false pass inflates U, and U is the quantity the level rule thresholds.
  So judge leniency pushes the level rule toward *over*-retirement, which is
  the opposite of the "Blind Curator" result the paper cites (a false-pass
  judge silently *disables* net-success retirement). The SkillsBench judge
  recomputes numbers in a sandbox, but the 1,251-skill dataset's judge does
  not. Our inference: the headline audit tables rest on the weaker judge.
- **Validity gate: what it certifies.** "Some other model gains > 0.1"
  shows the task is sensitive to the skill's content. It is a positive
  control on the instrument. It does not show that the *target* would lose
  anything without the skill, and the gate becomes harder to pass as
  witnesses improve, which is why the paper keeps a weaker witness. With 2
  aided runs per witness, a gain > 0.1 from noise alone is plausible on some
  tasks. The paper does not report a witness false-admission rate (e.g., the
  gate applied to a placebo skill).
- **Model coverage.** Four open-weight models, three of them Qwen, all
  between 27B and ~400B. No frontier closed model is a target. The
  "verdicts differ by model" result (skill × model 15%, model 0.4%) is
  exactly the kind of result that might change with a wider model spread,
  in either direction.
- **The Cortex link the digest draws needs care.**
  [[literature/papers/giampieri2026cortex]]'s `/prune` deletes when removing
  the item costs no runs (with exposure and protected-task guards). That is
  a gain rule, and a fail-both item that fired would be deleted. But Cortex
  serves one model in one repo. There, deleting a fired item that helps
  nothing is the pruning goal that Wang et al. concede gain rules serve
  ("pruning entries that do not help"). Their objection bites on Cortex's
  *other* use: "a model change is a retirement trigger", where a deletion
  after an upgrade is read as the model having absorbed the convention.
  Cortex's two flagged items after the Sonnet 5 upgrade could be
  internalized or uncovered, and only the unaided level tells which.
- **Fit with the rest of the graph.** SLIM
  ([[literature/papers/shen2026dynamic]]) warned that "removal is not
  internalization" and forced-to-zero libraries lose 92.2 → 76.6. This paper
  gives that warning a mechanism (the diagonal), a measurement (8.6% base
  rate, 57.5% removed by the band) and a fix. The reliability result extends
  [[concepts/pass-at-k]]'s "k bounds stochastic error, not systematic error"
  (kim2026divergent): more runs of one task do not buy a verdict, more tasks
  do. The binary collapse is a floor effect of all-tests-pass scoring. The same
  floor shows in the paper's automatic-retrieval run on SkillsBench, where
  83% of task pairs have zero gain and nearly all are double failures. Our
  inference: a library-mean skill gain on a hard benchmark is mostly
  computed over pairs that cannot move.

## Trust signals

- **Credibility:** 4.
  - For: an industry lab (EverMind / Shanda) with PKU and Southeast
    University co-authors. Unusually careful measurement: a G-theory variance
    decomposition, all three reliability facets, an attenuation bound on
    validation, a cross-fitted audit that removes shared-measurement
    dependence, a difficulty-gate correction with regenerated tasks, skill-
    and task-level bootstrap intervals throughout, an independent re-judge,
    and a blind third-family (Opus 5.5) audit of the judge. The SkillsBench
    analysis set and primary comparison were fixed before running, and the
    exclusions were fixed before any retirement decision. Many caveats are
    stated by the authors (the gate's limits, item validity, judge lift
    near τ). Code, prompts, 1,251 skills with tasks, rubrics and item scores
    under four models are released (github.com/EverMind-AI/SkillCorpus,
    huggingface EverMind-AI/skillcorpus-lifecycle, named in the PDF; not
    opened at ingest).
  - Against: not peer reviewed, no citations yet (4 days old). The skill
    source is the authors' own corpus. The target is also judge, critic and
    difficulty gate. Three of four models are Qwen, all open-weight. The
    downstream test detects no difference between rules. The κ advantage
    barely clears zero, and the cross-fitted audit rests on 29 decisions.
    The paper's own audit finds the judge's false full marks shift U near τ
    by the full τ − h margin.
  - Not 5 because there is no peer review or outside replication, and the
    only end-to-end outcome is a null. Not 3 because the core diagnosis is
    a proof plus a measurement that survives cross-fitting, the gate
    correction, four targets and a second library, and the authors measured
    most of the threats themselves.

## Follow-up

- **Relevance:** 4. It is the most careful treatment in this graph of the
  exit gate in [[concepts/skill-library-lifecycle]]. It tells this box what a
  retirement verdict must establish: unaided level, a task shown sensitive
  to the skill, per model, with tasks rather than runs as the replication
  unit. That applies directly when a model upgrade raises the question of
  which claude-system skills or rule clauses are now redundant. It also
  adds a measured case to [[concepts/pass-at-k]] (tasks buy more than runs;
  all-items binary collapse) and a positive-control analogue to the pending
  `hce-known-bad-preflight` proposal ([[concepts/hce-evaluation]]). Not 5
  because the setting is open-weight coding/tool tasks rather than ML
  research, and the level rule's downstream advantage is not shown.
- **Cheap local use (our inference, not tested).** For a "retire after the
  upgrade" pass over this box's skills, the paper's design maps onto what
  is already here. Run the current model without the skill on a few
  synthesized tasks per skill (more tasks, one run each). Use an older or
  smaller model (Haiku, Sonnet) as the witness that must gain from the
  skill. Retire softly (move, don't delete) only when U is high and a
  witness gains. A skill that fails with and without is a repair item, not
  a retirement.
- **Candidates** (cited, not yet in the graph):
  - Wang (Yixuan) et al. 2026, "Not All Skills Help: Measuring and
    Repairing Agent Knowledge" (ASSAY, arXiv 2606.15390). The audited band
    rule, with its spread-split guard.
  - Li et al. 2026, "SkillsBench" (arXiv 2602.12670). Used by
    [[literature/papers/li2026who]] but not ingested itself; the paper's
    downstream test bed.
  - Shaposhnikov et al. 2026, "A Framework for Evaluating Agentic Skills at
    Scale" (Tessl, arXiv 2606.17819). The two-part level-plus-gain
    condition this paper restores.
  - Zhang (Xing) et al. 2026, "The Blind Curator: How a Biased Judge
    Silently Disables Skill Retirement" (arXiv 2607.07436), and "Library
    Drift" (arXiv 2605.19576, ICML 2026 workshop; in the 06-29 digest, not
    ingested). Same group as the Ratchet loop in
    [[literature/papers/zhang2026who]]. Judge bias against retirement.
  - Zhan, Ma & Haddadi 2026, "Safe Skill Retirement for Physical Agents"
    (arXiv 2609.29543). Cited by Cortex too; clause-level retirement with a
    protected-effect guard.
  - Tank & Nama 2026, "The Regression Tax: Decomposing Why Skills Help and
    Hurt LLM Agents" (arXiv 2607.22520). Residual fail-both failures per
    model.
  - Han et al. 2026, "SWE-Skills-Bench" (arXiv 2603.15401). 39 of 49 skills
    with no pass-rate gain, split 24 pass-both / 15 imperfect-both: the
    field instance of the internalized/uncovered confusion.
  - Wang (Yanze) et al. 2026, "SkillCorpus" (arXiv 2607.15557). The skill
    source; the authors' own corpus.
