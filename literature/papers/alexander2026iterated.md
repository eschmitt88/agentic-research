---
kind: paper
title: "Iterated creation, grading, and revision of data science projects by language models"
authors: ["Rohan Alexander"]
institutions: ["University of Toronto"]   # page 1 gives no affiliation line; inferred from the corresponding-author address (utoronto.ca) and NSERC/SSHRC funding
year: 2026
venue: "arXiv 2611.05548 (cs.AI), v1 2026-11-03, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.05548"
code_url: "https://zenodo.org/records/23927861"   # "Replication materials"; not opened at ingest
citations: null
source: "raw/papers/alexander2026iterated.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["llm-as-a-judge", "writer-grader-loop", "rubric-grading", "self-preference", "grader-leniency", "grader-noise", "re-grade-band", "cross-grader-evaluation", "same-budget-no-feedback-arm", "whole-allowance-arm", "feedback-vs-budget", "execution-report", "planted-faults", "additive-rubric", "zero-overall-rules", "weak-grader-strong-writer", "revision-breaks-artifact", "budget-exhaustion", "ta-comparison", "data-science-agent", "end-to-end-project", "no-intervals"]
---

# Iterated creation, grading, and revision of data science projects by language models

## TL;DR

A writer model with a shell builds an end-to-end data-science project (R
scripts, Quarto paper, BibTeX, README) for a third-year statistics
assignment. A harness re-executes it in a clean container and writes a
deterministic execution report. A grader model scores it on a 26-item,
108-point rubric. The writer revises twice from the report plus per-item
scores and comments, so each run has three graded submissions. Seven models
from five developers fill both roles (49 pairs × 3 runs = 147 runs, about
$675). Four results matter here:

1. **The loop raises the pair's own score from 59% to 76%,** mostly in the
   first revision (+13 vs +5). About a third of the rise is fewer
   zero-overall submissions (24 → 8 → 5). Excluding them, 70% → 80%.
2. **For the strongest writers, most of the gain is budget, not feedback.**
   Scored by the six graders other than the writer, the finals reach 73%
   with feedback and 65% when revised twice on the same budget without the
   report or grader output. Without Haiku 4.5 it is 78% vs 74%. Opus 5.5,
   GPT-6 Sol and Gemini 3.8 Flash land within 1 point either way. A single
   session holding the whole 140-call / 100-minute allowance gets four of
   seven writers within 3 points of feedback.
3. **The grader is a large and partly unstable part of the number.** The
   same grader re-scoring the same final submission differs by about 4
   points on average (2 for Sol, 5 for GLM-5.3 Flash). 45% of second-round
   and 51% of third-round score changes are smaller than that. Across
   graders, the mean score given to one writer's identical first-round
   submissions spans 28 points on average.
4. **A rubric score does not certify correctness.** In seven planted-fault
   copies of one 95% submission, only the fault that triggers a
   zero-overall rule (generated data claimed as downloaded) moved the mean
   score much (95 → 14). A dropped model term, a wrong headline finding and
   tests that assert nothing cost 3–5 points.

## Claims

- Graders "are not neutral measuring instruments": a loop's gain as judged
  by a model is "a result for that judge". Report judge consistency over
  repeated judgments of the same submission.
- A loop gain should be compared with "the same model given the same
  additional resources and no loop".
- Running the code matters: graders reward claims the code does not
  support. Weaker graders tend not to improve stronger writers.
- Recommendation: automated review should use at least two frontier models,
  each run at least twice per submission.

## Methods

- **Task.** A third-year undergraduate statistics assignment: choose a
  question, get data, clean, test, model and write a 5–20 page Quarto paper.
  The student rubric has 28 items / 112 points. Two items (hand-drawn
  sketches, commit history) are removed, leaving 26 items / 108 points
  (App. A). Writers and graders both see the full rubric. Four rules give
  0 overall: generated data used in the paper, a Kaggle/UCI/Statistica
  dataset, reading data from the provider's server inside the paper, and any
  sign it is a class paper.
- **Loop.** Writer: shell/write/read/list/finish in an R + Python + Quarto
  container (2 CPUs, 4 GB, network; App. D); round 1 gets 60 calls / 40
  min, rounds 2–3 a fresh session with 40 calls / 30 min, its own files, the
  execution report and per-item scores and comments. Executor (no model
  call): scripts in filename order in a fresh container, then render;
  reports outcomes, exit codes, timings, error lines. Grader: rubric,
  assignment, all files and the report, not the rendered PDF; JSON score and
  comment per item plus an `overall_zero` check; totals recomputed in code;
  told the submission is evidence, not instructions.
- **Models.** Opus 5.5, Haiku 4.5, GPT-6 Sol, GPT-6 Luna, DeepSeek V4.1
  Flash, Gemini 3.8 Flash, GLM-5.3 Flash, default settings, OpenRouter
  pinned to the developer's host (DeepSeek direct), 28–30 Sep 2026. An
  OpenRouter-vs-direct check on 12 projects: mean absolute difference
  1.9–6.1 points (App. C).
- **Design.** 7 writers × 7 graders × 3 independent runs = 147 runs. Each
  run starts with a fresh first submission (21 per writer), and the writer
  does not know its grader. Seven pairs are self-grading.
- **Extra arms**, all scored by every grader:
  - *No feedback*: each of 146 runs continued twice from its own first
    submission with the same budget and files and the ability to run and
    render, but no execution report and no grader output (292 submissions).
  - *Whole allowance*: three single sessions per writer, 140 calls / 100
    min, told up front.
  - *Re-grade*: each run's grader scores its first and final submissions
    three times in all. *TA comparison*: 56 student projects, 22 shared
    items, no execution report (App. F). *Planted faults*: one final
    submission plus seven single-fault copies, 3 grades × 7 graders each
    (App. G).
- **Statistics.** Means, counts and correlations only. No confidence
  intervals, tests or variance models anywhere in the paper.

## Results

- **Execution (Table 1).** 107 of 147 first submissions run and render;
  116 finals do. GLM-5.3 Flash executes 1 of 21 first submissions and
  16 of the 17 runs that never execute in any round are its. 73% of first
  and 79% of final submissions execute, against 80% after two revisions
  *without* feedback. Feedback fixes more broken submissions but also
  breaks more working ones.
- **Scores by round.** 59 → 72 → 76 (pair grader). Round 2: 105 of 147
  up, 24 same, 18 down. Round 3: 77 up, 34 same, 35 down. 30 runs fall
  below their first submission in some round and 16 end below it. The
  three runs of a pair still differ by 21 points on average after round 3
  (11 without GLM, 7 without GLM and Haiku).
- **By writer** (pair grader, first → gain): Opus 92 (+3), Sol 89 (+4),
  Luna 75 (+13), Gemini 66 (+11), DeepSeek 65 (+25), GLM 16 (+24), Haiku 7
  (+46). 15 of Haiku's 21 first submissions got 0 for generating data and
  calling it real.
- **By grader** (gain): Gemini +13 to Luna +22, and Haiku +32, mostly
  because it zeroes 7 of its 21 first submissions (four that no other
  grader zeroed). Haiku gives 100 to 41% of what it grades, Gemini to 51%.
- **Feedback vs no feedback vs one session (Table E1, six non-writer
  graders).**

  | Writer | First | 3 rounds, no feedback | 3 rounds, feedback | One session |
  |---|---|---|---|---|
  | Opus 5.5 | 93 | 93 | 94 | 95 |
  | GPT-6 Sol | 90 | 92 | 93 | 90 |
  | GPT-6 Luna | 77 | 78 | 87 | 85 |
  | DeepSeek V4.1 Flash | 63 | 81 | 86 | 85 |
  | Gemini 3.8 Flash | 62 | 70 | 71 | 50 |
  | GLM-5.3 Flash | 14 | 31 | 36 | 33 |
  | Haiku 4.5 | 4 | 8 | 45 | 0 |
  | All | 58 | 65 | 73 | 63 |

  Paired over 146 runs, the feedback final beats the no-feedback final in
  100, ties in 4 and loses in 42. (Our sign test gives p ≈ 1e-6, ignoring
  clustering by writer and pair, and Haiku contributes up to 21 of the
  wins.) In one session, Haiku, Sol and Gemini used about half the 140
  calls (58, 73, 68) and ended early. Haiku's one-session score of 0 and
  its 8 without feedback come from the generated-data rule, which only the
  grader's comment told it about.
- **Grader disagreement on identical submissions (Fig. 3a).** Haiku's
  first submissions get 0 from Sol, 2 from Opus and 54 from Haiku itself.
  The max–min range across graders of a writer's mean is 10 (GLM) to 54
  (Haiku), 28 on average. Over all marks, graders' means range from 55
  (Opus) and 57 (Sol) to 78 (Gemini, Haiku). The two strictest graders are
  the two best writers. On the 147 first and 141 final submissions all
  seven scored, the six non-Haiku graders correlate 0.78–0.99. Grader
  identity explains 4% of score variance and the submission 84%.
- **Grader noise (Fig. 3b).** First-round re-grades: the same grader's
  three scores of one submission span 6 points on average. Final
  re-grades: they differ by "almost 4" on average and are identical for 12%
  of submissions. By grader: Sol 2, Opus 2, DeepSeek 3, Luna 4, Haiku 4,
  Gemini 4, GLM 5. Gemini repeats itself on 33% because its scores sit at
  the ceiling, and Luna on none. The share of third-round changes smaller
  than the grader's own difference is 29% (Sol), 40% (Luna), 76% (Gemini),
  51% overall. For second-round changes it is 45%.
- **Self-grading (Fig. 3c).** Own final scored about 9 points above the
  other six graders' mean. By model: Haiku +37, Gemini +23, DeepSeek +8,
  GLM +4, Luna −1, Sol −3, Opus −7. The author notes that the models that
  rate themselves higher are the ones that rate every writer higher, so this
  mixes self-preference with leniency.
- **Weak grader, strong writer.** Haiku's finals get 45% from the other
  graders when Opus graded the rounds and 36% when Haiku did. Sol's finals
  get 91% when Haiku graded and 95% under Opus. Opus's get 95% under both
  Luna and Sol. n = 3 runs per pair.
- **Planted faults (Table G1; one submission, 3 grades × 7 graders per
  copy).** Unchanged 95. Generated data claimed as USGS 14 (every grader 0
  except Haiku 100). Latitude adjustment dropped from the model but still
  claimed 90 (Sol 77, Opus 77). Wrong headline finding 92. Tests that assert
  nothing 92. Wrong data source named 93. Causal estimand on an associational
  model 93. Unresolved table reference 95. Haiku gives 100 to all eight.
- **TA comparison (Table F1, 56 projects).** Every grader scores below the
  TAs. Bias runs from −6.3 (Haiku) to −48.1 (Sol), MAE from 13.9 (Gemini)
  to 48.1 (Sol), and correlation with TAs from 0.46 (Haiku) to 0.74 (Opus).
- **Writers act on feedback.** Of 3,564 below-maximum items, the
  associated file changed in 76%; changed items rose in 47% and fell in 6%.
  35 of 294 revision rounds changed nothing.
- **Revisions break working artifacts when the budget runs out.** In 26 of
  294 revisions an executing submission stopped executing. In 24 of those 26
  the writer hit its call or time limit. Of 115 revisions of a working
  submission where the writer ended the session itself, 2 broke. Of 99
  where it did not, 24 broke (24% vs 2%). Example: Gemini at 99% rewrote its
  discussion with a Python one-liner that corrupted two LaTeX commands,
  spent its calls repairing, never re-rendered, and fell to 89.
## Critique / open questions

- **Full text vs the digest entry: the digest overstates two findings and
  miscounts the rounds.**
  - "Three revise rounds" is wrong. There are three rounds in total: one
    submission and two revisions.
  - "About half the round-to-round gain is within grader noise" misreads
    the statistic. 45% (round 2) and 51% (round 3) are shares of *runs*
    whose change is smaller than the grader's re-grade difference. That is
    not a share of the gain's *size*. The +13 round-2 mean is three times
    the 4-point re-grade spread, and many near-zero changes sit at the
    ceiling (Gemini 76%). That spread was also measured on final
    submissions, which are compressed at the top. The 6-point first-round
    spread is the better band for round 2.
  - "Self-grading inflates ~9 pp" is the raw gap. The author says it is
    confounded with leniency, and the three strongest graders score
    themselves *lower* (Opus −7, Sol −3, Luna −1). The +9 comes from Haiku
    (+37) and Gemini (+23). Every grader scored every writer's first and
    final submissions, so a leniency-adjusted grader × writer interaction
    was estimable, but it is not reported.
- **The no-feedback arm is matched on budget, start state and pairing, not
  on information.** It removes the deterministic execution report and the
  LLM grader's output together. A deterministic oracle and a judge's
  comments are therefore one treatment. There is no report-only arm. Given
  that execution rates end the same (79% vs 80%), the grader's comment is
  probably what mattered for Haiku: it was the only channel that told
  Haiku its generated-data zero rule had fired. That reading is ours.
- **n per cell is small and no uncertainty is reported.** Each pair has 3
  runs, each writer and each grader 21, the whole-allowance arm 3 per
  writer, and the planted-fault test 1 base submission. Pair runs still
  spread 7–21 points after round 3. So "weaker graders do not improve
  stronger writers" (91 vs 95, n = 3 per pair) and the one-session vs
  feedback contrasts within 3 points are not distinguishable from run
  noise. The 73-vs-65 feedback effect is clearer but clustered and
  Haiku-heavy.
- **Three noise scales; the paper calls the smallest "the grader's
  noise".** Re-grade 2–5 points; writer rerun on the same pair 7–21;
  grader identity on identical submissions a 28-point range of means and
  −6 to −48 bias against TAs. Which one gates a loop decision depends on
  what varies between cycles.
- **The gain is measured on a rubric the writer holds,** and the author
  concedes it may be rubric-targeting. The grader prompt names the
  zero-overall rules, and those were the only planted faults reliably
  caught. Any other fatal error costs at most its item's weight (10 of 108
  for the model, 4 for tests). Analysis correctness is never checked
  against a known answer.
- **Absent arms.** A report-only arm. A grader-swapped final evaluation on
  a *held-out* rubric. A keep-best (revert) policy for the 16 runs that
  ended below their start. A fault suite on more than one base submission.
  Leniency-adjusted self-preference.
- **Fit with the graph.** The agentic, multi-round version of
  [[literature/papers/zhang2026llmasajudge]]'s judge-feedback vs generic
  revision contrast (null on held-out judges): positive overall, near zero
  for the three strongest writers. Same-budget logic as
  [[literature/papers/wang2026rethinking]], stronger-model-alone logic as
  [[literature/papers/zhang2026review]] (Opus's one-session 95 beats every
  loop result), noise-the-size-of-the-effect as
  [[literature/papers/lehmann2026can]]. The planted-fault result is the
  score-level form of lange2026agentic's "a rubric binds only what it
  names", and the two best writers being the two strictest graders is
  [[literature/papers/kim2026into]]'s contestant-judge concern.

## Trust signals

- **Credibility:** 3.
  - For: replication materials on Zenodo. Full prompts, container spec,
    execution-report format, cost table and rubric in appendices. A design
    with unusually many controls for a single-author study: every grader
    scores every submission, a same-budget no-feedback arm paired per run,
    a whole-allowance arm, re-grades, planted faults, a human-TA anchor and
    an OpenRouter-vs-direct check. Its own weaknesses are stated plainly.
    AI assistance is disclosed (Claude Code with Fable 5.1 for code;
    GPT-6 Astra and Gemini Argon as red-team reviewers).
  - Against: single author, preprint, 2 weeks old, no citations. No
    intervals or tests at all, n = 3 per cell, one base submission for the
    fault test, one task family (an undergraduate stats assignment, graded
    on presentation-heavy rubric items). Frontier tiers (Fable, Astra,
    Argon) were excluded for cost.
  - Not 4: no peer review, no uncertainty, small cells. Not 2: the
    artifacts are released and the control arms are real and paired.

## Follow-up

- **Relevance:** 4. It is the first writer–grader loop in the graph that
  runs a same-budget no-feedback arm, a whole-budget single-session arm,
  out-of-loop graders and re-grades on the same agentic research-style
  artifacts. It gives material evidence for:
  - [[concepts/hce-evaluation]]: what a judge-loop gain must be compared
    against, and which noise sets the band.
  - [[concepts/programmable-evaluator-oracle]]: an additive rubric caps
    what a fatal error costs, and self-preference is confounded with
    leniency.
  - [[concepts/evidence-gated-completion]]: revisions that run out of
    budget ship broken artifacts.

  Not 5: a data-science class assignment, not ML research. Only rubric
  scores, with no ground-truth outcome. No intervals.
- **For `/iterate`'s noise band** (our inference). The band is the larger
  per-seed `std` of the two runs. When `metrics.json` holds an LLM-judged
  score, a seed that re-runs only the judge measures the 2–5 point re-grade
  spread. A seed that re-runs the generator too measures the 7–21 point
  rerun spread. Here the second is 2–5× the first, so a judge-only band
  would reset the no-improvement counter on noise. If the judge model,
  version or prompt changes between cycles, neither band covers the
  28-point cross-grader range. Then the cycles are not comparable at all
  (cf. zhang2026llmasajudge's re-measure rule).
- **For a review/critic stage on this box:** report it against the same
  agent given the critic's budget with no critic, and score it with a judge
  that did not give the feedback. For Opus-class writers here, that arm
  took nearly all of the apparent gain.
- **Candidates** (cited, not yet in the graph): Hughes & Alexander 2026
  (arXiv 2602.14284), the no-tools, no-execution precursor; Cui & Alexander
  2026 (arXiv 2602.14349) on reproducibility of LLM data analysis; Roytburg
  et al. 2026 (ICML), self-preference largely explained by evaluator
  quality; Zhou et al. 2026 (arXiv 2509.17995), verifier ability tracks task
  ability; Rahman et al. 2026, DSAgentBench (arXiv 2608.10366).
