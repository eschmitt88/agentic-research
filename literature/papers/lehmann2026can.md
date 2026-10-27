---
kind: paper
title: "Can LLMs Improve Their Own Harness? A Controlled Study on ARC-AGI-3"
authors: ["Jens Lehmann", "Sahar Vahdati"]
institutions: ["Amazon (work done outside Amazon)", "Technische Universität Dresden", "Universität Hannover", "TIB – Leibniz Information Centre for Science and Technology and University Library"]
year: 2026
venue: "arXiv 2610.30499 (cs.AI), v1 2026-10-21, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.30499"
code_url: null     # artifact ships as arXiv ancillary file harness-adaptation-review-artifact.zip (App. G); no repo URL in the PDF; not opened at ingest
citations: null
source: "raw/papers/lehmann2026can.pdf"
added: "2026-10-27"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/context-eviction-policy]]"
tags: ["harness-adaptation", "self-improvement", "recursive-self-improvement", "arc-agi-3", "rhae", "held-out-games", "all-snapshot-evaluation", "loop-reruns", "adaptation-gap", "dev-heldout-gap", "no-abstention", "keep-parent", "post-hoc-checkpoint-selection", "fixed-harness-rerun", "noise-band", "score-cost-pareto", "outcome-blind-coding", "llm-annotator", "process-checks", "opencode", "human-steered-comparison", "null-result"]
---

# Can LLMs Improve Their Own Harness? A Controlled Study on ARC-AGI-3

## TL;DR

Nine model–effort conditions (GPT-6 Astra, GPT-5.6 Sol/Terra/Luna, Claude
Fable 5, Opus 5, Sonnet 5; Sol and Opus at two efforts) each build an
ARC-AGI-3 harness from a minimal repository through a fixed coding bootstrap
(OpenCode). Each harness is revised twice from its own traces on 5
development games. That is done three times per condition: 27 lineages and
81 frozen snapshots. Every snapshot is scored once on 20 public games that
never feed construction, revision or selection.

Three results matter for this graph:

1. **The pooled held-out gain is not distinguishable from zero.** Revision 2
   raises held-out RHAE by **+2.80 (95% CI −1.44 to 7.35)**. Gains and
   losses split 15/12 (sign test p = .70), and 9/3/15 with a ±5-point band.
   Development RHAE rises +8.42. The dev–held-out gap is **5.62
   (CI 1.78–9.77)**, but omitting one development game (r11l) cuts it to
   3.02.
2. **The noise band is measured, once.** Ten evaluations of one unchanged
   harness on the same 20 games give a suite SD of **3.99** RHAE (mean
   absolute pairwise difference 4.73). The median *per-game* SD is **18.78**.
   A 2.80-point mean gain is smaller than the typical difference between
   two evaluations of the same harness.
3. **No updater ever kept its parent.** Retaining the parent was allowed.
   All 54 revisions changed the code. 22 of 27 first revisions reoriented
   the strategy, and every lineage grew (median 2,161 → 2,912 Python lines).
   All 54 episodes inspected item-level evidence, targeted an
   evidence-backed issue and passed their operational checks, and held-out
   outcomes still went either way. A post hoc "pick the snapshot with the
   best dev score" rule recovers +4.53 over H0 (CI 0.81–8.68).

## Claims

- Harness adaptation is a capability separate from harness construction.
  The authors define the **harness adaptation gap**
  G_t = [J_dev(H_t) − J_dev(H_0)] − [J_test(H_t) − J_test(H_0)] as the
  quantity to report. Positive G means dev feedback overstates transfer.
- Models "autonomously produce diverse harnesses spanning zero to
  near-ceiling performance". Initial held-out RHAE ranges from 0 to 97.14
  (mean 24.54). "Yet their revisions do not consistently turn development
  experience into held-out gains."
- "Reductions in inference cost can come at the expense of task
  performance." 9 of the 18 revision-2 descendants that got cheaper also
  lost RHAE.
- "Abstention is part of adaptation." A reliable updater must recognise
  when evidence favours keeping the current harness. Candidate revisions
  should be compared with a preserved baseline and rolled back when not
  justified (cites RRSI's noise- and cost-aware acceptance).
- Human-steered harnesses show headroom. Tycho (the authors' own system)
  scores 100 on all 25 public games at the same Opus 5 xhigh setting where
  these revision-2 harnesses average 39.95. The paper says this "does not
  isolate the contribution of human steering", because Tycho selected among
  policies on all 25 games.
- Explicitly not claimed: that revisions *reduce* expected performance. The
  15/12 counts are "observed suite outcomes, not estimates of how often
  revisions reduce expected performance."

## Methods

- **Two harnesses, one model.** The same model profile acts as builder
  (through OpenCode, fixed for the campaign) and as runtime model (through
  the generated harness). The starting repository holds the task contract,
  a local engine client, an LLM client, model and pricing docs, and a
  `run.sh` contract. No solver architecture, game rules or examples are
  given. The build prompt (App. B) asks for high RHAE "while using
  inference economically".
- **Lifecycle (Fig. 1).** Construct → smoke-qualify on 2 dev games (50
  actions, no score threshold, up to 10 repair cycles) → freeze H0 → run 5
  dev games → revision 1 gets a fixed bundle (its own source, traces,
  scores, actions, costs) and may change anything or keep the parent →
  qualify → freeze H1 → same again for H2. No human edits or guidance
  between stages. Coding sessions have no access to the live game or the
  runtime-model gateway.
- **Split.** 5 dev games (tr87, vc33, r11l, bp35, ft09) "selected at random
  before the campaign"; the other 20 public games are feedback-held-out.
  This is the same 5/20 split the authors' Tycho paper uses.
- **Evaluation.** Every frozen snapshot is scored once on all 25 games
  (2,025 game evaluations). Traces are replayed locally and through the
  official ARC hosted scorer; all 81 validations agree on score. The
  primary endpoint is fixed: mean held-out RHAE, H0 → H2, over all lineages
  completing three stages. All 27 did. CIs come from 100,000 whole-lineage
  bootstrap resamples (pairing preserved, fixed seed). Between-condition
  comparisons are labelled descriptive.
- **Costs.** Dollar inference cost per held-out game is recomputed from
  token ledgers at list prices frozen 2026-09-13. The program cost at least
  $18,129, $16,130 of it gameplay. There is no common dollar cap on
  gameplay.
- **Noise study (App. D.1).** Sol-high replicate 1 at H2 was rerun 9 more
  times after the campaign. That snapshot was chosen post hoc, after
  outcomes were known, because GPT-5.6 is fast.
- **Revision coding (Section 6.2, App. F).** All 54 revision episodes were
  packaged with model identity and all downstream outcomes masked, then
  coded twice by GPT-5.6 Sol xhigh under a frozen 12-item codebook.
  Consensus was hashed before outcome linkage. Items below κw 0.67 are
  dropped from outcome analysis. There is no human validation.
- **Strategy audit (App. E).** A separate, post hoc, outcome-visible label
  for each transition: Preserve / Extend / Reorient / Replace.
- **Integrity (App. G).** Snapshot fingerprints, prompt constancy and
  hosted-validation agreement were rechecked against retained records.
  2,130 game attempts were made for 2,025 slots. 63 slots had several
  attempts, and each slot has exactly one clean attempt, which is the one
  used. The records show no choice among multiple clean scores.

## Results

- **Per condition (Table 3, Table 4; H0 → H2 held-out mean, n = 3 each).**
  Rows not in Table 4 are our arithmetic from Table 3.

  | Condition | H0 | H2 | Change |
  |---|---|---|---|
  | GPT-6 Astra high | 82.92 | 94.37 | +11.45 |
  | Opus 5 xhigh | 31.27 | 39.95 | +8.68 |
  | Opus 5 high | 11.71 | 15.43 | +3.72 |
  | Fable 5 high | 21.11 | 23.79 | +2.68 |
  | Sol high | 26.15 | 28.58 | +2.43 |
  | Sonnet 5 high | 0.09 | 0.35 | +0.27 |
  | Sol xhigh | 41.98 | 41.94 | −0.05 |
  | Luna high | 1.45 | 0.97 | −0.48 |
  | Terra high | 4.16 | 0.69 | −3.47 |

  Six lineages (Sonnet, Luna) sit at the floor. Their sub-point moves count
  as strict gains or losses in the 15/12 tally. Leave-one-condition-out
  means run from +1.72 to +3.59. With equal family weights the mean is
  +2.55; the condition-stratified CI is −1.12 to 7.01.
- **Within a condition, replicates disagree.** All three Sol-high
  replicates follow the same R → E strategy path. Their held-out outcomes
  run from +33.18 to −15.18.
- **Per game (Table 12).** 13 held-out games gain on average and 7 lose.
  The largest moves are lp85 +20.44 and sb26 −15.03. Complete games rise
  73 → 94 of 540 (13.5% → 17.4%). Levels completed rise 34.5% → 37.1%.
- **Threshold sensitivity (Table 9).**

  | ε (RHAE) | Improve | Regress | Indeterminate | Pareto gain |
  |---|---|---|---|---|
  | 0 | 15 | 12 | 0 | 9 |
  | 1 | 11 | 8 | 8 | 7 |
  | 5 | 9 | 3 | 15 | 5 |

- **Noise (Table 10).** Ten evaluations: mean 48.53, SD 3.99, range
  43.38–55.37. The original campaign observation (53.56) is the second
  highest of the ten, 5.03 above the mean. All ten stay above the lineage's
  H0 (20.38), so that showcase gain is real. Replacing its H2 with the
  ten-run mean moves the pooled endpoint from +2.80 to +2.62. The largest
  per-game SDs are sb26 30.76, m0r0 28.33 and tn36 27.13.
- **Post hoc checkpoint selection (Table 5).**

  | Policy | Held-out RHAE | Gain vs H0 | $/game | +/−/= |
  |---|---|---|---|---|
  | Retain H0 | 24.54 | 0 | 10.30 | — |
  | Deploy H2 | 27.34 | +2.80 | 7.57 | 15/12/0 |
  | Best dev snapshot | 29.06 | +4.53 [0.81, 8.68] | 9.15 | 13/5/9 |
  | Held-out oracle | 30.66 | +6.13 | 9.80 | 18/0/9 |

  The dev rule picks H0/H1/H2 in 9/6/12 lineages and beats always-H2 by
  +1.72 [0.38, 3.67]. The rule was evaluated after the fact, and the paper
  labels it so.
- **Score–cost (Fig. 4, Tables 6–7).** H0 → H2: 9 Pareto gains, 6 better
  and dearer, 9 cheaper and worse, 3 worse on both. Mean cost per game falls
  $10.30 → $7.57 (−$2.72). The median change is only −$0.54.
- **What the revisions changed (App. E, Tables 13–14, Fig. 5).**
  - First revisions: 22 Reorient, 4 Extend, 1 Preserve. Second: 12 / 12 / 3.
    No transition replaces the harness family.
  - What moved: context persistence, replay flow, model-consultation
    cadence, action batching, budget allocation, memory, perception.
  - Every lineage grows: median 2,161 → 2,604 → 2,912 Python lines, mean
    +1,052. Large and small diffs fall on both sides of zero held-out
    change.
  - T001 (Sol high): H1 hit its own 100-decision cap on 3 dev games and
    wrote cache entries it rarely read. Revision 2 raised the cap to 120,
    removed explicit cache writes, and added animation-aware observations,
    object geometry and completed-level memory. Held-out went 29.24 → 53.56
    at about the same cost.
  - T007 (Sol xhigh): revision 1 replaced response-ID chaining and history
    compaction with independent calls carrying a bounded notebook. Held-out
    fell 43.95 → 18.49 (21.99 at H2), and cost fell $31.39 → $4.89 ($7.18).
    Dev RHAE fell too (37.11 → 31.73).
- **Process codes (Tables 16–17).** All 54 episodes inspected item-level
  evidence, targeted at least one evidence-backed issue and passed proximal
  checks, so those items are invariant and cannot discriminate. Broad
  rewrites average +3.75 immediate held-out change and contained ones
  −0.20 (contrast −3.94, CI −9.09 to 1.22). Episodes where a post-edit
  result triggered further edits, reverts or test changes (18) average
  −0.76; clean hand-offs (36) average +2.48 (contrast −3.24, CI −8.54 to
  2.38). The reactive episodes start from stronger parents (32.94 vs 21.33).
  Diagnosis (κw 0.35), incumbent protection (0.47) and resource
  metareasoning (0.29) fail the reliability bar.

## Critique / open questions

- **Against the digest framing: at framing.** The digest called this "a
  harness self-improvement datum with a held-out design", which is
  accurate. The paper's own abstract leads with the near-null result. The
  four ingest checks come out as follows.
  1. **Runs per arm.** 3 independent lineages per condition, 27 in total.
     Each snapshot is scored **once** on the 25 games. The only
     repeated-evaluation data are 10 runs of one snapshot picked post hoc.
     Per-condition means rest on n = 3, and the paper treats them as
     descriptive.
  2. **Are the 20 held-out games untouched by adaptation?** Yes, as far as
     the loop goes. No held-out score reaches the builder, the
     qualification gate, the stage lifecycle or any selection. Coding
     sessions cannot reach the live engine. Recovery reruns never chose
     among clean scores. By [[literature/papers/chen2026auditing]]'s A2 item
     the primary endpoint is a Y. Three softer leaks remain:
     - The games are public and may be in pretraining (acknowledged).
     - The experimenters know all 25 games well. Tycho, their earlier
       system, selected policies on all 25. The minimal repository and
       `docs/arc_rules.md` were written by them. The build prompt is
       generic, so this is a protocol-holdout caveat only ("Task holdout is
       not protocol holdout" in [[concepts/hce-evaluation]]).
     - The secondary analyses that touch held-out scores are post hoc and
       labelled so: the dev-selection rule, the noise-study snapshot and
       the strategy labels.
  3. **Does the gain survive on held-out?** The pooled gain does not, at
     the stated confidence: +2.80, CI −1.44 to 7.35. It is about half the
     dev gain and concentrated in two conditions (Astra +11.45, Opus xhigh
     +8.68). Several individual lineages gain large amounts that survive.
     Sol-high 1 stays above its H0 in all ten re-evaluations, and Astra 2
     goes 58.03 → 92.62. So "self-revision sometimes works and you can't
     tell when from the dev score" is the defensible reading.
  4. **What the two revisions changed.** Mostly the control layer rather
     than game knowledge: context continuity, replay, consultation cadence,
     batching and budget. Revisions add code every time and never abstain.
     See Results.
- **The paper's "loss" category is partly what the prompt asked for.** The
  build prompt scores the system on "the tradeoff between RHAE and inference
  cost" with no exchange rate. A revision that cuts cost by 77% and loses
  RHAE (T007) is a contraction on one axis, not necessarily a failure on
  the stated objective. The paper reports score and cost jointly, which is
  right. Calling 9 of 27 "performance contractions" is fair as a
  description, but it should not be read as "half of cost-cutting is
  error".
- **The updater was given single-run, 5-game feedback with per-game noise
  SD near 19.** Our inference: much of what an updater reads as an
  "evidence-backed issue" on a dev game is one draw from a very wide
  distribution, so fitting it is noise-chasing. This is a mechanism for the
  positive G that the paper does not test. A repeated-dev-evaluation arm
  would test it. So would feeding the updater the measured noise band.
- **Missing arms.**
  - Same-budget, no-revision arm (wang2026rethinking): re-run H0 with the
    revision budget spent on extra attempts.
  - Null-feedback or shuffled-feedback revision (chen2026auditing's A5).
  - A prospective acceptance gate: the post hoc dev rule, pre-registered,
    or an RRSI-style noise band from repeated H0 runs.

  The last is the arm the paper's own conclusion asks for and does not run.
- **The noise study is n = 1 harness, chosen post hoc,** and it is a
  mid-range harness (RHAE about 48). The ε = 5 band in Table 9 is scaled
  from it. Floor harnesses (Sonnet, Luna) and near-ceiling ones (Astra)
  surely have different spreads. The pooled CI already absorbs evaluation
  noise through the lineage bootstrap, so the headline is not affected.
- **The adaptation gap estimate leans on one dev game.** r11l alone moves G
  from 5.62 to 3.02. The authors report this. With 5 dev games, the gap's
  CI understates split uncertainty: it conditions on the fixed split, as
  the paper says.
- **The process-coding layer is weak, and the paper says so.** The coder
  is a single model family that is also a studied condition (GPT-5.6 Sol),
  and there is no human check. Half the planning labels are missing, all
  of them for the Opus and Sonnet conditions. The two retained associations
  have CIs spanning zero and are unadjusted. The robust process finding is
  the invariant one: every episode looks diligent, and that does not sort
  outcomes.
- **Self-comparison.** The human-steered reference point is the authors'
  own Tycho. They report the selection-procedure difference fairly, but the
  "headroom" claim is benchmarked against their own prior work.
- **Fit with the graph.**
  - [[literature/papers/wang2026rethinking]] found +0.0 held-out from
    AHE-style harness evolution on Terminal-Bench (single runs, no CIs).
    This is the same direction on a different domain, with 27 lineages and
    a lineage bootstrap.
  - [[literature/papers/hu2026winners]]: here there is no acceptance step
    at all (always deploy H2), so the gap is adaptive overfitting to dev
    feedback, not a winner's curse from selection. That makes it a
    complementary measurement. The two together suggest the dev score is
    unreliable for both proposing and accepting.
  - [[literature/papers/xia2026rrsi]] is the noise-band acceptance design
    this paper recommends and does not test.
  - [[literature/papers/rodionov2026improving]] reported ARC-AGI-3
    prompt-line gains of 1.4–7.3 RHAE on 3 blocks. Against a single-harness
    suite SD of 3.99 measured here, the low end of that range is inside one
    evaluation's spread.
  - [[concepts/context-eviction-policy]] and
    [[concepts/compression-as-generalization-test]]: T007 is one confounded
    case of a self-revision that dropped context continuity (response
    chaining plus compaction, replaced by stateless calls with a bounded
    notebook) to save cost, and lost half its held-out score. Separately,
    every lineage grew in code and nothing in the loop penalised it; size
    and churn did not sort outcomes.

## Trust signals

- **Credibility:** 4.
  - For:
    - The primary endpoint is pre-specified (always-H2 vs H0, all
      lineages). Held-out never touches the loop, and the lineage-level
      bootstrap keeps pairing.
    - Three full-loop reruns per condition. This is rarer than a held-out
      split: 22% in chen2026auditing's census.
    - A measured fixed-harness noise band, threshold sensitivity, and
      leave-one-condition-out and leave-one-dev-game-out checks.
    - Hosted-scorer replay of every trace, a fingerprint-level integrity
      audit and an explicit attempt-selection audit.
    - It reports a near-null result in its own abstract. The limitations
      section is specific (no repeated evaluations, LLM coders unvalidated,
      condition comparisons descriptive).
    - Full artifact (snapshots, prompts, splits, analysis scripts,
      checksums) as an arXiv ancillary file.
    - Established researchers (TU Dresden, TIB/Hannover; Amazon affiliation
      with work done outside it).
  - Against:
    - Not peer reviewed.
    - One evaluation per snapshot.
    - n = 3 per condition.
    - The noise study is one post hoc harness.
    - The process coding is LLM-only and uses a studied model family.
    - The headroom comparison is to the authors' own system.
    - Lower-bound cost accounting with unreconciled recovery meters.
    - The artifact was not opened at ingest.
  - Not 5 because there is no peer review, single-shot snapshot evaluation,
    and the arms that would explain the gap (null feedback, same budget,
    prospective acceptance) are absent. Not 3 because the design is cleaner
    than most lab harness-evolution papers in this graph. The held-out wall
    is real, and the result goes against the authors' interest.

## Follow-up

- **Relevance:** 4. It is the cleanest test in this graph of a frontier
  model revising its own harness from its own traces with a strict holdout
  and loop reruns. That is the shape of this box's own self-application
  loop (`/elevate` proposing claude-system edits from session evidence). It
  adds material evidence to:
  - [[concepts/hce-evaluation]]: a measured dev–held-out gap with a CI,
    under no selection.
  - [[concepts/evolutionary-search-grain]]: an updater that may keep its
    parent never does, so acceptance has to live outside the proposer.
  - [[concepts/evidence-gated-completion]]: every revision passes the
    process checks, and those checks do not predict transfer.

  Not 5 because it is a game benchmark, not an ML-research loop, and the
  prescriptive arm (an acceptance gate) is untested.
- **For the noise-band / compute-matching front.** It supplies a measured
  suite SD (3.99 on 20 games) and a per-game SD (18.78) for one harness.
  Our inference for this box: any `/elevate`-style "the revised skill is
  better" claim scored on a handful of single-run tasks is inside noise
  unless the same unchanged configuration is re-run first. That is
  liu2026what's same-harness rerun arm, and here it is cheap relative to
  the revision.
- **For `/elevate` and self-revision generally.** "Keep the parent" was
  always allowed and never taken. A proposal step should not be trusted to
  abstain. The decision to adopt has to be a separate, scored step against a
  preserved baseline.
- **Candidates** (cited, not yet in the graph):
  - Ursekar et al. 2026, "HarnessOpt-Bench: Evaluating LLMs at harness
    optimization" (arXiv 2608.06301). Best validation score seen during
    search is optimistic relative to the nominated harness's test score.
  - Zheng et al. 2026, "SEAGym: An evaluation environment for self-evolving
    LLM agents" (arXiv 2606.17546). 39/40 harness updates complete while
    test success falls 14.5 pp.
  - Zhang et al. 2026b, "HarnessCompass" (arXiv 2608.01918). Minimal-seed
    strict test over five rounds.
  - Lin et al. 2026b, "Harness updating is not harness benefit" (HUIB,
    arXiv 2605.30621). Separates producing an update from using it.
  - Lee et al. 2026, "Meta-Harness" (arXiv 2603.28052), and Lin et al.
    2026a, "Agentic harness engineering" (AHE, arXiv 2604.25850).
  - Lehmann, Aioanei & Vahdati 2026, "Tycho: Active abstraction with
    programmable world models for ARC-AGI-3" (arXiv 2607.28287). The
    human-steered comparator.
  - Sarafian et al. 2026, "Workspace optimization: How to train your agent"
    (arXiv 2605.09650). In-game revision of world models under a fixed
    harness.
  - Skoutnev et al. 2026, "Twin: Playing an unknown game with a test-time
    digital twin" (arXiv 2608.14490). Off-the-shelf Codex 61.1 vs 93.3 with
    a world-model harness, same model.
