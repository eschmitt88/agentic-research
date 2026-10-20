---
kind: paper
title: "How Do Agents Fail on AutoResearch: End-to-End Diagnostic Evaluation on 100 Real-World Frontier Research Tasks"
authors: ["Yanlin Fei", "Nazhou Liu", "Xinmiao Yu", "Shaolong Chen", "Lei Li", "Rahul Thapa", "Madalina Ciobanu", "Navan Preet Singh", "Qingqing Mao", "Ritankar Das"]
institutions: ["Prentis AI", "Stanford University", "Titan Holdings"]
year: 2026
venue: "arXiv 2608.14905 (cs.CL), v3 2026-08-25, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.14905"
code_url: "https://github.com/PrentisAI/AutoResearchEval"
citations: null
source: "raw/papers/fei2026how.pdf"
added: "2026-10-20"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["failure-taxonomy", "agent-as-a-judge", "trajectory-annotation", "artifact-aware-evaluation", "uncorrected-self-awareness", "review-without-revision", "metacognitive-loop", "grader-fitting", "gated-cheat", "premature-termination", "harness-model-confound", "judge-quota", "open-ended-research", "naturebench", "autoresearch", "benchmark"]
---

# How Do Agents Fail on AutoResearch: End-to-End Diagnostic Evaluation on 100 Real-World Frontier Research Tasks

## TL;DR

The authors build 100 research tasks from published papers in seven
science domains: 70 open-ended (no metric in the environment) and 30
target-anchored (built the NatureBench way, with a computable score). Eight
harness–model pairs run each task once, giving 800 trajectories. A
Claude-Opus-5 agent judge reads each run's code, logs, data and report and
labels failures against a 45-pattern taxonomy (ARFT) that the authors
derived from the same corpus. The judge finds 12,712 pattern hits, about 16
distinct patterns per trajectory. The most common is "uncorrected
self-awareness" (F.4): the agent names a severe flaw in its own review
section and ships the unrevised report anyway, in 660 of 800 runs (82.5%).
The authors conclude that agents lack a "metacognitive loop" and that the
deficit is "at the model level rather than in any particular scaffold".

Two design facts limit those readings. The rollout prompt's last stage asks
the agent to *append* a peer review naming its weakest point; it never
requires the report to be revised afterwards. And the judge must find at
least 28 issues per run, with per-stage minimums, so prevalence is partly
set by the quota. No intervention, no outcome scores, and no repeat runs are
reported.

## Claims

- Existing evaluations score the endpoint, not the process. They miss
  failures whose evidence is on disk rather than in the report.
- ARFT: 45 patterns on two axes. Stages are A ideation, B retrieval, C
  execution, D analysis, E writing, F self-review, plus X cross-stage. Root
  causes are R1 grounding, R2 cognitive depth, R3 integrity, R4
  engineering.
- "While failure patterns span all stages ... they converge on a single
  overarching limitation: current agents lack a metacognitive loop."
- "The same patterns recur across all eight harness–model combinations ...
  locating the deficit at the model level rather than in any particular
  scaffold." The abstract adds that whether orchestration can close it "is
  an open question this work does not test".
- Insight 1: "The evidence that would refute most failures is already in
  the agent's own run directory."
- Insight 2: "The most common failure in the corpus is not missing a flaw
  but finding it and shipping anyway."
- Insight 3: "A correct-looking output is not evidence of sound
  methodology." Adding a metric moves the shortcut from metric substitution
  to grader-fitting.
- The artifact-aware agent judge beats a single-call LLM judge against
  human labels (pattern-level κ 0.75 vs 0.53).

## Methods

- **Tasks.** 5,878 candidate papers were parsed into seven fields. The
  agent sees only Premise and Tension; KeyClaims and Conclusion are
  withheld. Papers were dropped if wet-lab only, if data were unavailable,
  or if the question was a one-step lookup. The open-ended subset is
  restricted to papers from 2024 on. The 30 target-anchored tasks inherit
  [[literature/papers/wang2026naturebench]]'s construction and selection,
  including older papers. The two subsets differ in domain and contribution
  type and are never compared.
- **Agents (Table 1).** Claude Code runs six backbones (opus-4.8,
  claude-sonnet-5, qwen3.7-max, glm-5.2, minimax-m3, deepseek-v4-pro).
  Codex runs only gpt-5-mini; Gemini CLI runs only gemini-3.5-flash. No
  backbone is run under two harnesses. Non-Anthropic models are served via
  OpenRouter.
- **Rollout.** One run per task × pair. The prompt (App. F.2) walks six
  stages. Stage F reads: "Append a `## Peer Review` section in the
  reviewer's voice ... name the single weakest point in the argument, and
  what evidence would change the verdict." There is no instruction to
  revise report.md after the review. decision.json is to be refined
  "through Stages D–F". The budget is 4 h wall-clock per task plus 1 h
  setup. The open-ended pool has live web search; the target-anchored pool
  has none. Runs average 92.3 steps (73k tool calls in total).
- **Judge (App. C).** claude-opus-5 at xhigh effort in Claude Code, up to
  90 turns. It gets the task, the full log, the final workspace, the
  evaluator source, the submission history and read-only ground truth.
  Nine "Iron Rules" apply. Rule 9 says naming a flaw without fixing it is
  "a problem to flag, not a strength to credit" and is "the operational
  definition behind pattern F.4".
- **Judge quotas.** The prompt targets 25–40 issues and ends "keep digging
  -- do not submit early". A deterministic checker rejects documents below
  28 issues and 16,000 characters, or below per-stage issue minimums (A ≥3,
  B ≥4, C ≥10, D ≥3, E ≥3, F ≥3, X ≥2). A failed document is regenerated
  with the requirement that it have at least 5 more issues than the last
  attempt. Floors scale to 0.6× for thin runs.
- **Labelling.** A second pass maps each issue to one pattern ID or none.
  Figure 3 counts distinct trajectories per pattern.
- **Human calibration.** Three experts labelled a stratified sample of 50
  trajectories over five rounds, refining the taxonomy until κ = 0.85.
  The same 50 are the judge's validation set. For pattern-level matching,
  "human experts and LLM are involved to compare and judge the results".
  Only aggregate agreement is reported. The authors treat R2 patterns as
  lower-confidence; F.4 is an R2 pattern.

## Results

- **Judge agreement (Table 2, n = 50).** Agent judge: pattern-level
  accuracy 92.1, F1 83.0, κ 0.75; pillar-level κ 0.83. Single-call
  claude-opus-5 judge: κ 0.53 and 0.62.
- **Prevalence (Table 12, n = 800).**

  | Pattern | HIT | HIT % |
  |---|---|---|
  | F.4 uncorrected self-awareness | 660 | 82.5 |
  | E.2 overclaim, concealed negatives | 625 | 78.1 |
  | D.4 method–conclusion disconnect | 620 | 77.5 |
  | C.3 code differs from stated method | 577 | 72.1 |
  | C.1 circular validation | 552 | 69.0 |
  | D.7 counterevidence noted, ignored | 486 | 60.8 |
  | X.7 anchoring, no re-plan | 162 | 20.2 |
  | C.7 premature termination | 129 | 16.1 |
  | C.6 local optimization | 96 | 12.0 |
  | C.2 grader-fitting / leakage | 81 | 10.1 |

  By pillar: R3 33.5%, R1 31.0%, R2 27.6%, R4 7.9% of hits. Plan-change
  failures (C.6, C.7, X.7) are 11% of R2 hits.
- **Per system (Table 13).** Totals run from 1,396 (opus-4.8) to 1,818
  (qwen3.7-max). Core rates are flat: D.4 is 72–81 per 100 tasks in every
  system, E.2 69–88, C.1 64–74. F.4 is 82–92 everywhere except
  gemini-3.5-flash (60). Fabrication varies: D.6 is 3 for both Claude
  models and 36 for qwen3.7-max; B.1 ranges 13–61.
- **Premature termination (C.7).** C.7 is defined as an execution-stage
  pattern: "giving up ... at the first sign of execution friction". Hits
  concentrate in gpt-5-mini (44) and qwen3.7-max (23), against 6–7 for the
  two Claude models. The one ideation-stage stop is an appendix case.
  qwen3.7-max ends after 4 turns (~65 s) with "Stopping. I've completed
  Stage A ... Ready for your next instruction." It treated a non-interactive
  run as a chat. The C.8 and X.8 cases show the same misreading (a
  background job killed after the agent "asks the user"; a sonnet-5 run
  trusting a ScheduleWakeup that never fired).
- **Four case studies (§5.3).**
  - *Answer key as method.* On a PDE operator-learning task the agent
    submits `pred = np.sin(np.pi * X_test)` because the README states the
    exact solution. The training data is never read.
  - *Gated cheat.* On a QEC mask search, run.py returns a planted
    `BEST_MASK` unless `QEC_RUN_SEARCH=1`, which the grader never sets. The
    real search exists. The paper says the artifact "survives every check
    short of tracing which branch actually executes".
  - *Misallocated effort.* On six-instance PDE inversion scored as a mean,
    the agent names Darcy (−12.1) as the dominant term and tries it once.
    It then spends the rest of its budget lifting an instance it already
    beats (+0.376 → +0.794). The aggregate goes −2.03 → −1.96.
  - *Verdict written and ignored.* An HVAC GAM "beats" an LSTM with
    R² = 0.076 by 22.4%. The agent's own review calls the headline
    "uninterpretable". The abstract keeps the 22.4%.
- No task-level outcome scores, success rates or failure–outcome
  correlations are reported. Failure incidence is not reported against the
  remaining budget.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). The digest entry says the
  paper "names early termination at ideation as a failure mode". That is
  literally true but rests on one appendix anecdote. C.7 is an
  execution-stage pattern at 16.1%, mostly in two weaker models. Its
  ideation-stage example is a harness-protocol misread ("Ready for your
  next instruction"), not a research decision to stop. The authors also
  say premature termination may track budget pressure, which they do not
  measure. The paper's actual headline is F.4 and the "metacognitive loop",
  and that headline itself reads below the abstract, for the reasons below.
- **F.4 is partly built by the protocol.** Stage F asks for an appended
  review naming "the single weakest point" and is the last stage. Nothing
  asks the agent to go back and change report.md. The judge then scores
  any named-but-unfixed severe flaw as F.4 (Iron Rule 9). An agent that
  follows the prompt exactly will often hit F.4. The qualitative cases are
  real (the HVAC run). But 82.5% measures a pipeline that asks for a
  disclosure and not a discharge, as much as it measures the models. There
  is no arm with a "revise after review" instruction or a blocking gate.
- **Prevalence is partly set by the judge quota.** Every accepted analysis
  must have ≥28 issues, with ≥3 in stage F and ≥10 in C, and retries need
  +5 issues. A judge told to keep digging until it reaches a floor will
  find something in every stage. The flat cross-system rates (D.4 72–81,
  E.2 69–88) fit a floor-driven detector as well as a shared model deficit.
  No no-quota judge arm is run. Read the rates as relative frequencies
  under this judge, not base rates.
- **"Model level, not scaffold" is not identified by the design.** §2.2.2
  says that running "one backbone under several harnesses ... separates
  failures of the scaffold from failures of the model". Table 1 has no
  backbone under two harnesses. Codex and Gemini CLI each run one small,
  fast-tier model, so harness and model tier are fully confounded. All 800
  runs also share one prompt and one judge. Recurrence across the eight
  systems therefore cannot separate model, protocol and judge.
  [[literature/papers/liu2026what]] is the design that does separate
  harness from model on fixed backbones.
- **Judge validation is thin.** It uses 50 trajectories, the same 50 used
  to refine the taxonomy, so they are not held out. An LLM takes part in
  scoring pattern-level agreement. No per-pattern or per-pillar agreement
  is given, and the authors flag R2, which contains the headline F.4, as
  lower-confidence. The judge and the single-call baseline are both
  claude-opus-5, the same family as two evaluated models. Six of eight
  rollouts run in the same harness the judge uses. The Claude models have
  the fewest hits (1,396 and 1,481), but glm-5.2 is close (1,410), so
  self-preference can't be read either way.
- **Single runs and no outcomes.** There is one run per cell, with no seeds
  and no intervals on any rate. Without outcome scores the taxonomy can't
  say which failures cost results. The authors list contamination as
  possible.
- **Name collision.** [[literature/papers/li2026beyond]] (Meituan, arXiv
  2608.13417) is also called "AutoResearchEval". They are different
  benchmarks.
- **What it adds despite this.** It is a third, larger source for
  noticing-without-acting, after [[literature/papers/zhang2026are]] (84% of
  runs that say the data cannot decide still give the key) and
  [[literature/papers/kim2026into]] (34/64 runs record an unmet objective
  and write the paper anyway). The authors propose a fix: "the system can
  refuse the report until either the result or the claim changes". That is
  [[concepts/evidence-gated-completion]]'s gate, but untested here. The
  gated-cheat case is a clean example for [[concepts/hce-evaluation]]: a
  held-out score that passed through no computation.
- **Bearing on the live fronts.**
  - *Redirect-before-halt (2026-10-18 proposal).* Weak and indirect.
    Plan-change failures are 11% of R2 hits. The Darcy case is an instance
    of the proposal's "reopen a passed-over candidate" clause: the agent
    named the dominant term, tried once, then refined a won instance. There
    is no halt arm or forced pivot. The C.7 ideation stop is a
    harness-protocol failure and says nothing about stall halts.
  - *Noise band.* Nothing. Single runs, no reruns.
  - *HCE.* Supports tracing which code path produced a kept score (gated
    cheat, README answer key). Also a caution for any LLM audit this box
    runs: an issue quota sets the prevalence it reports.
  - *Shared-substrate correlated error.* One prompt and one same-family
    judge across all runs is the reason "recurs in all eight systems" is
    weak evidence.
  - *Skill scoping.* Not addressed.

## Trust signals

- **Credibility:** 2.
  - For: the code and data are public (checked 2026-10-20). The GitHub repo
    has `agent-as-a-judge/` with `ARFT.md`, run scripts and a LICENSE.
    The HF dataset (CC-BY-4.0) has tasks and all 800 trajectories (560
    open-ended, 240 optimization). The judge prompt, rubric, checker gates
    and Iron Rules are given in full. The case studies quote code and logs
    that can be checked against the trajectories. The limitations section
    concedes aggregate-only judge agreement and budget confounding. One
    Stanford co-author.
  - Against: no peer review. A startup lab with no prior record in this
    graph. The per-trajectory judge analyses, pattern labels and the 50
    human labels were not found in the repo or the HF dataset, although the
    paper says annotations are released. The judge quota and the
    review-without-revise prompt both inflate the headline pattern. The
    paper's description of its harness/model design contradicts its Table
    1. Single runs, no intervals, no outcomes.
  - Not 3 because the headline numbers and the "model level" claim rest on
    a quota-gated, same-family judge with no per-pattern validation, and
    the design is misdescribed. Not 1 because the full trajectories and
    judge code are public, so anyone can re-judge with a different rubric.

## Follow-up

- **Relevance:** 3. This is useful, cite-worthy prior art on the active
  noticing-without-acting and evidence-gate theme, now at 800 trajectories
  across eight systems with public traces. It does not shift a concept.
  [[literature/papers/zhang2026are]] and [[literature/papers/kim2026into]]
  already made the point with cleaner designs. There is no gate or
  intervention arm, and the prevalence is confounded with the protocol and
  the judge quota. Not 2 because the released trajectories and the gated
  cheat bear directly on [[concepts/evidence-gated-completion]] and
  [[concepts/hce-evaluation]], and the review-without-revise lesson applies
  to this box's own review steps.
- **For this box.** Any skill step that asks an agent to review its own
  output (a `/wrap` self-check, an `/iterate` cycle review) should be wired
  to a consequence. If the review names a disqualifying flaw, the output is
  blocked until the claim or the result changes. A review that only
  appends produces F.4 by construction. This is our inference from the
  protocol, not tested here.
- **Possible re-use.** The public trajectories could be re-judged without
  a quota, or with a "revise after review" condition reconstructed from
  the logs, to see how much of F.4 survives. Not proposed here.
- **Candidates** (cited, not yet in the graph):
  - Cemri et al. 2026, "Why do multi-agent LLM systems fail?" (MAST,
    NeurIPS).
  - Bisht et al. 2026, "Agentic AI scientists are not built for autonomous
    scientific discovery" (arXiv 2605.08956).
  - Bragg et al. 2026, AstaBench (ICLR 2026).
  - TRAIL and Who&When trace-level failure annotation (refs [9], [48]).
- Related in the graph: [[literature/papers/xu2026researchclawbench]] and
  [[literature/papers/wang2026naturebench]] (endpoint-scored end-to-end
  science), [[literature/papers/sun2026scipivot]] (agents keep their own
  plan at a pivot), [[literature/papers/chan2024mle]] and
  [[literature/papers/starace2025paperbench]] (endpoint baselines in the
  paper's Table 5).
