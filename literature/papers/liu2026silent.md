---
kind: paper
title: "Silent Instrument Errors Reorder Agent Leaderboards"
authors: ["Yangze Liu"]
institutions: ["Shandong University"]
year: 2026
venue: "arXiv 2611.04399 (cs.AI), v1 2026-11-03, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.04399"
code_url: "https://github.com/YangzeLiu/eval-instrument-disclosure"
citations: null
source: "raw/papers/liu2026silent.pdf"
added: "2026-11-16"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/spend-forecast-calibration]]"
tags: ["instrument-error", "leaderboard-audit", "hal", "tau-bench", "tau2-bench", "graded-fraction", "termination-gated-grading", "cost-double-count", "nested-spans", "root-span", "action-provenance", "grader-replay", "trajectory-archive", "re-grade", "evaluator-versioning", "lenient-replay", "silent-failure", "disclosure", "errata", "artifact-only-audit"]
---

# Silent Instrument Errors Reorder Agent Leaderboards

## TL;DR

The author audits two public agent leaderboards, HAL and τ²-bench, using
only their published artifacts. No agent is run and no model is called.
The audit finds four errors in the run-time pipeline, meaning the code that
triggers grading, logs, prices, replays and versions an evaluation. None of
the four is in the benchmark's task or reward design. Each produced a
plausible number and raised no error:

1. **Half the Generalist's episodes were never graded.** On HAL's TAU-bench
   Airline, the Generalist scaffold was graded in 51% of its episodes,
   against 98% for Tool-Calling. Ungraded episodes were stored as
   failures. Grading them offline under the benchmark's own rule raises the
   Generalist's pooled accuracy from 0.30 to 0.43, which turns a 13-point
   published gap into a tie.
2. **The cost column counts nested calls twice, unevenly.** A wrapper span
   and its provider-client child span carry the same usage, and the
   aggregator sums both. 70 of 134 logged runs come out at exactly 2× and
   46 at 1×.
3. **The archived trajectories contain the grader's actions.** The
   grader's gold replay is appended to the agent's action list in 68% of
   2,334 airline records.
4. **A re-grade is a new measurement.** A correct, announced τ²-bench fix
   reorders the leaderboard. A mode check that misreads four telecom files
   drops their scores by 59–82 points, and the command still prints a
   number.

The HAL maintainers confirmed their three errors. The proposed remedy is
one extra number per pipeline stage, published next to the score.

## Claims

- In agent evaluation, the infrastructure is part of the measurement. It
  changes as fast as the agents do, and its errors produce plausible
  numbers rather than failures. Leaderboards should publish checks on the
  instrument alongside the scores.
- Agent evaluation is more exposed than static QA in four places. Grading
  is triggered by agent behaviour. Logging has layers. A replay writes into
  the records. Evaluators and archives change at different speeds.
- Each error came from a locally sensible decision: summing over all
  spans, grading at episode end, replaying gold through the real
  environment, making replay lenient. The failures came from interactions
  between components written by different people at different times, and
  reviewing any one component would likely have passed it.
- The author expects more such errors elsewhere. This is an argument from
  mechanism, not from a survey.
- Five proposed checks:
  - report the graded fraction per cell;
  - sum cost over root spans only (or deduplicate by response ID) and print
    the all-span/root-span ratio;
  - tag the provenance of every recorded action (agent, user simulator or
    grader), and run the gold replay in a separate environment;
  - pin evaluator and task-data versions, and publish a per-task diff with
    every re-grade, failing when replay mismatches pass a threshold;
  - ship machine-readable errata inside frozen archives.
- "Openness is necessary for auditing the instrument and does not perform
  the audit." Re-running released code reproduces released errors.

## Methods

- **HAL.** HAL publishes 380 encrypted trace archives on Hugging Face,
  and its own script decrypts them with a fixed passphrase. The author
  decrypted 138 (TAU-bench Airline, ScienceAgentBench, CORE-Bench Hard),
  plus the SciCode and SWE-bench Verified Mini archives for the cost case.
  The HAL numbers were recomputed a second time with independently written
  code.
- **Cost.** For each run, the reported usage is compared with the
  all-calls sum and the root-calls-only sum. In 133 of 134 runs the
  reported figure equals the all-calls sum, which identifies the
  aggregation rule.
- **TAU-bench re-grade.** The author installs the τ-bench commit that HAL
  pins and replays recorded actions with the LLM client and user simulator
  stubbed. So only the database comparison and the output check are
  evaluated, not any user-side criterion.
  - Grader actions are separated from the agent's by aligning the agent's
    own tool calls, parsed from logged model outputs, in order against the
    recorded action list by tool name and arguments.
  - An episode counts as graded if its record holds an injected gold copy
    anywhere, or if its recorded reward is positive.
  - HAL's fork differs from the pinned gold on 3 of 50 tasks. The re-grade
    was repeated with each record's embedded gold, and the pooled figures
    did not change.
- **Validation of the classification.**
  - Re-running the grader at each recorded ending event reproduces the
    recorded reward for all 1,951 graded records.
  - Replaying only the agent's own actions on graded episodes reproduces
    the reward for 97% of Tool-Calling, 97% of Few-Shot and 84% of
    Generalist records. Almost every Generalist disagreement is a grader
    pass that the offline replay fails, so the offline re-grade, if
    anything, understates the Generalist.
- **τ²-bench.** Four builds were installed side by side (0.2.0, 1.0.0, the
  commit before 1.0.1, and 1.0.1). Public trajectory files were re-graded
  under each, varying evaluator code and task data independently. Fidelity
  check: the 1.0.1 re-grade of claude-opus-4-5 on banking gives 0.2474,
  against the official 24.74%.
- **Disclosure.** The HAL items were reported on 3 September 2026
  (HF discussion #9 and an email). Issues were filed as tau-bench #92 and
  tau2-bench #502.

## Results

- **Cost (§3.1, App. A).**
  - Reported tokens are exactly 2× the root-span tokens in 70 of 134 runs
    and 1× in 46.
  - 44 of the 94 leaderboard rows that match a logged archive carry a
    doubled cost.
  - Over 133 runs, the reported cost exceeds the root-span cost by 36.5% on
    average.
  - The doubling follows the client path, which depends on scaffold ×
    provider. On SciCode, all ten zero-shot runs are clean, while most runs
    of the other two scaffolds are about 2×.
  - Example: Tool-Calling with Claude Opus 4.1 (high reasoning) is listed
    at $149.98 for a run whose root-span cost is $74.99.
  - HAL's own paper drops "redundant calls related to weave logging" in its
    log analysis, but not in the cost column.
- **Graded fraction (§3.2, Fig. 1, Table 2).**
  - Mechanism: the Generalist sends an unsolicited closing message and
    exits. The simulator usually does not answer it with the stop token, so
    `done` never fires.
  - Ungraded episodes have no ending event and stop at a median of nine
    actions, far below any step limit.
  - On the 41 tasks whose gold has a non-terminal action, pooled over nine
    models: the Generalist goes from 0.30 to 0.43, and Tool-Calling moves
    by at most 2.5 points.
  - **Most of the pooled gain is on tasks whose gold only reads the
    database.** The benchmark's rule passes those whenever the database is
    left unchanged. On write-required tasks the Generalist rises from 0.28
    to 0.33, and each of the 23 newly passing ungraded episodes contains
    the required write among the agent's own calls.
  - Per model: the Generalist trails for 8 of 9 models as published and 7
    of 9 after re-grading. Only gemini-2.0-flash changes order. Run by run,
    48 of 351 pairwise orderings among 27 runs reverse.
  - Each model row rests on 40–82 Generalist records, and the author claims
    only the direction of the shift.
  - Including the withdrawn Few-Shot traces, the Generalist is last for 9
    of 10 models as published and last for 6 after re-grading.
- **Grader actions in trajectories (§3.3).**
  - Grader actions appear in 68% of 2,334 airline records, usually at the
    end.
  - In Generalist episodes that ended with a transfer, they sit in the
    middle, because the harness appends its closing message after the
    grader runs.
  - In 58 records the agent made no tool call, yet the record shows the
    full gold sequence.
  - Replaying stored actions verbatim reproduces the recorded reward in
    70% of records whose gold has ≥ 2 non-terminal actions. Stripping the
    grader's actions first raises this to 92%.
- **Re-grading (§3.4).**
  - Banking, release 1.0.1 (announced): no pass became a fail. All movement
    comes from evaluator code, none from task data. Still, 4 of 136
    pairwise orderings among 17 submissions reverse, and gemini-3-flash
    moves from 9th to a tie for 5th.
  - Airline, release 1.0.0 (listed, not flagged): the criteria of 14 of 50
    tasks changed in substance. Through the documented re-grade command, 77
    of 800 simulations flip in both directions, per-model scores move −7 to
    +2.5, and gpt-4.1-mini and claude-3-7-sonnet swap order.
  - Telecom (silent): the solo-mode check matches a single agent name.
    Four files recorded with a related solo agent fall through, and the
    replay environment is built without user tools. Under the 1.0.1
    lenient replay, every user call warns and continues. Scores fall by
    59–82 points (e.g. 0.72 → 0.10 in issue #502). A companion file with
    the recognised agent name re-grades to its recorded score.
- **After disclosure (§3.5).**
  - HAL reproduced all three items and added web-page disclosures, dated
    5 October and announced 7 October.
  - The archives are unchanged and costs have not been re-derived.
  - As of 3 November, the τ-bench and τ²-bench issues have no maintainer
    reply, and community fix PRs #508 and #573 are open.

## Critique / open questions

- **Against the digest entry: at (slightly above) the digest.** Every
  digest number is in the PDF and is quoted correctly. The full text adds
  four qualifications that change how the result should be read:
  1. **"Closes the gap" is pooled, and much of it rests on a lenient rule.**
     Most of the 0.30 → 0.43 gain comes from read-only-gold tasks, where
     τ-bench passes any episode that leaves the database unchanged. That
     rule passes a do-nothing episode, as Zhu et al.'s checklist and
     [[literature/papers/chen2026when]]'s "says goodbye, gets a pass"
     counter-example both show. Both scaffolds benefit from it equally, so
     the *comparison* is fair. The solid "solved tasks recorded as
     failures" evidence is the write-task shift from 0.28 to 0.33 (23
     episodes). Per model, the Generalist still trails for 7 of 9 models.
  2. **The figures were corrected after HAL confirmed them.** The
     September report counted an episode as graded only if gold formed the
     end of its record. That missed 73 Generalist episodes with interior
     copies, which is the very trap §3.3 warns about. The repo says the
     per-model reordering first reported "does not hold". HAL's 7 October
     confirmation predates this correction, and the HF thread shows no
     maintainer reply to it. So "HAL confirmed three" confirms the three
     *errors*, not the current figures. (Checked on HF discussion #9 and
     the GitHub repo on 2026-11-16. The paper dates the correction
     2 November, while the repo and thread say 1 November.)
  3. **The re-grade is partial.** The user simulator is stubbed, so only
     database and output criteria are checked (Limitations). Scaffold
     differences remain: different step budgets, and the policy sits in
     the system prompt in one scaffold and in a file in the other.
  4. **There is a fifth check the digest omits:** errata inside the
     archive.
- **The digest's "graded fraction is a cheap sibling of the do-nothing
  preflight" is half right.** Both checks are cheap, but they catch
  different defects. An ungraded episode stored as zero lands at the
  floor, so a do-nothing preflight *passes* it by construction. The graded
  fraction measures whether the scorer ran (coverage). The preflight
  measures what it returns when it does run (value). τ-bench fails the
  do-nothing preflight on read-only tasks for the opposite reason: there,
  doing nothing passes. This is our reading.
- **Two independent reports describe the same τ-bench function.**
  `calculate_reward` replays gold through `self.step`. chen2026when found
  that it leaves the database holding the gold state. This paper finds
  that it appends gold to `self.actions`. Neither paper cites the other.
  Two single-author audits from different groups converge on one
  mechanism, which strengthens it.
- **Fit to this box.** Two of the four modes transfer directly: cost
  metering (the box's Stop hook had the same class of bug twice, 07-26 and
  11-14) and evaluator versioning across re-scored experiments. The box's
  evaluators mostly score an artifact once, so termination-gated grading
  and replay-into-record have weaker analogues. They apply wherever a
  downstream project grades an interactive episode, or reads its own
  transcripts as evidence about agent behaviour.
- **Generality is argued, not measured.** The evidence covers two
  leaderboards and four cases found in "a few days". The prevalence claim
  is explicitly by mechanism.
- **Open question.** Does fail-closed scoring (HCE clause 5: a crash
  scores as a failure) create the graded-fraction problem inside our own
  loops? See the hce-evaluation edit.

## Trust signals

- **Credibility:** 4. A single author from Shandong University, not peer
  reviewed, and only 13 days old. But almost every claim is a
  recomputation from public artifacts, with reproduction scripts and
  commit IDs released. The HAL numbers were computed twice with
  independent code, and an exact fidelity check matches the official
  banking score (0.2474). The HAL maintainers independently reproduced all
  three HAL errors on their own archive, which we verified on the HF
  thread. The author published a correction that weakened their own
  earlier claim.
  - Against: the per-model results rest on 40–82 records per cell; the
    re-grade stubs the user simulator; the τ²-bench items have no
    maintainer confirmation; the prevalence claim is by mechanism.
  - Not 5: preprint, no peer review, and the τ² findings are not
    externally confirmed.
  - Not 3: maintainer reproduction is a stronger signal than most
    major-lab preprints carry.

## Follow-up

- **Relevance:** 4. This is material new evidence for
  [[concepts/hce-evaluation]]: the grading trigger, logging, replay and
  evaluator version all belong to "the evaluator", and each fails silently.
  It also adds to [[concepts/programmable-evaluator-oracle]] (the oracle's
  invocation and its writes must be observable) and to
  [[concepts/spend-forecast-calibration]] (an uneven meter multiplier
  corrupts the history a forecaster fits; this is the HAL twin of the
  box's own Stop-hook bug).
  - Not 5: the box's own evaluators are mostly one-shot artifact scorers,
    where two of the four modes have only weak analogues, and the box has
    already fixed its metering bug.
- **Design implications (our inference).**
  - Record `n_scored / n_attempted` next to `metrics.json`.
  - Have the token meter print its raw-to-deduplicated ratio on every run.
  - Never rank experiments scored under different evaluator hashes without
    re-scoring them and diffing per task.
- **Candidates** (cited, not yet in the graph):
  - Liu & Han 2026, "Asking earns nothing: Scoring the decision to act in
    BFCL multi-turn" (arXiv 2610.04429). A companion instrument-vs-model
    case.
  - Masters, Liu & Albrecht 2026, "Rollout Cards: A reproducibility
    standard for agent research" (arXiv 2605.12131). None of 50 repos
    reports failed or skipped runs, and the reporting rule alone reverses
    rankings.
  - Kirgis et al. 2026, "Log analysis is necessary for credible evaluation
    of AI agents" (arXiv 2605.08545).
  - Kapoor et al. 2025, "Holistic Agent Leaderboard" (arXiv 2510.11977).
  - Zhu et al. 2025, "Establishing best practices for building rigorous
    agentic benchmarks" (ABC, arXiv 2507.02825).
  - Chen et al. 2026, "Emerging trends in AI agent evaluation" (Princeton
    SAgE survey, 19 October 2026).
