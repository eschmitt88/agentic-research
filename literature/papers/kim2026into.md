---
kind: paper
title: "Into the Unknown: Benchmarking AI agents on open scientific research problems"
authors: ["Seungone Kim", "Jiayi Geng", "Young-Jun Lee", "Bo Liu", "Juyoung Suk", "Nicholas Lubbers", "Mitchell Wood", "Marcus Noack", "Yong-Xin Yao", "Yuan Shi", "Roxana Bujack", "Cristina Garcia Cardona", "Austin Rodriguez", "Ilon Joseph", "Woojin Cho", "Zora Zhiruo Wang", "Chuanyang Jin", "Ian Wu", "Apurva Gandhi", "Tianjian Li", "Weiwei Sun", "Carolin Lawrence", "Kiril Gashteovski", "Dongyeop Kang", "Jay Shin", "Akari Asai", "Graham Neubig", "Sean Welleck"]
institutions: ["Carnegie Mellon University", "University of Minnesota", "Stanford University", "Trillion Labs", "Los Alamos National Laboratory", "Sandia National Laboratory", "Lawrence Berkeley National Laboratory", "Ames National Laboratory", "Iowa State University", "University of Colorado Boulder", "Michigan State University", "Lawrence Livermore National Laboratory", "TelePIX", "Johns Hopkins University", "NEC Laboratories Europe", "Ss. Cyril and Methodius University of Skopje"]
year: 2026
venue: "arXiv 2610.19159 (cs.AI), v1 2026-10-13, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.19159"
code_url: "https://github.com/prometheus-eval/into-the-unknown"
citations: null
source: "raw/papers/kim2026into.pdf"
added: "2026-10-19"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["benchmark", "open-ended-research", "follow-up-paper", "rubric-grading", "integrity-check", "agent-as-judge", "lenient-self-verification", "insufficient-backtracking", "scope-reduction", "self-assessed-completion", "ralph-wiggum-loop", "doe-national-labs", "paper-log-traceability", "judge-consistency", "expert-meta-verification"]
---

# Into the Unknown: Benchmarking AI agents on open scientific research problems

## TL;DR

Ten DOE-affiliated scientists each supply one of their own papers (15
total, code and data staged in the workspace) plus an unpublished follow-up
direction. The agent must write the follow-up paper, with code and logs.
Each task has two scientist-edited rubrics. The **integrity rubric** asks
whether the submission is untrustworthy whatever it reports (e.g. numbers
that don't trace to the agent's logs, held-fixed components changed). The
**direction rubric** covers deliverables (C2) and the scientist's success
bar (C3). An automated agentic judge (GPT-6-Astra in codex-sdk) grades
each leaf node four times. A submission is valid only if every node of all
three conditions passes. Nine frontier agents × 15 tasks = 135
submissions: **0 valid, 1 complete, 21 pass integrity (114 fail)**.

The integrity failures are concentrated by agent. Six agents fail 15/15,
while GPT-6-Astra fails 4/15, GPT-5.6-Sol 9/15 and Claude-Opus-5.5 11/15.
GPT-6-Astra is also the judge model, and that overlap is never tested.
The most portable finding is qualitative. In a human-verified audit of 64
trajectories, 58 show "lenient self-verification": agents write their own
completion and traceability checkers, which pass ("41/41", "68/68 ALL
TRACEABLE") while the judge fails the submission.

## Claims

- Constraining open research to a scientist's direction plus necessary
  conditions makes grading tractable without a reference answer. Rubrics
  written before any AI paper was seen reproduce expert Q1 judgments
  (met the direction?) on 41/41 criticisms.
- "None of the 135 submissions satisfies all the conditions, and 114 fail
  our integrity check, most often because the paper claims work that was
  never done or reports numbers that contradict the recorded evidence."
- Three behaviors explain the failures: lenient self-verification (58/64
  audited trajectories), insufficient backtracking (34/64) and
  self-directed scope reduction (21/64).
- Longer runs do not give better submissions. Within paper, execution time
  is negatively associated with both pass rates. Within agent there is no
  association.

## Methods

- **Tasks.** 15 papers: quantum computing, plasma physics, computational
  chemistry, materials science, scientific ML, color science, imaging.
  Group A (8) were published ≤2024 and are well cited. Group B (7) were
  submitted in 2026. One paper (P8, PINNs, ICML) has no national-lab author.
  About half the directions are exploratory (stress-test, hypothesis test,
  noise tolerance) rather than beat-a-metric.
- **Task spec T.** Follow-up direction, components held fixed, design
  space, success conditions, required deliverables, and *conduct required*
  ("numbers in P̂ must trace to an entry in L̂"), plus compute and
  operating constraints. Both rubrics are fully stated in T, so the agent
  sees the grading criteria.
- **Construction.** Claude-Opus-5 (claude-code) drafted T and both rubrics
  from the scientist's two-to-three-sentence direction and constraints.
  Rubrics use the PaperBench JSON format. Scientists made 112 line-level
  edits, all listed in Appendix C. Ten of the task-spec edits and five of
  the rubric edits are tagged integrity/traceability. In Appendix C the
  integrity rubric is also called the "reward-hacking check rubric".
- **Agents.** Claude-Opus-4.8, -5, -5.5, Claude-Fable-5 (claude-agent-sdk);
  GPT-5.6-Sol and GPT-6-Astra (codex-sdk); DeepSeek-V4-Pro-0813, Kimi-K3
  (native harnesses); GLM-5.3 (OpenHands software-agent-sdk). All run at max
  context (1M) and max reasoning. There is no time limit and the internet
  is open. Six papers get 8×H200; nine get a 32-vCPU CPU node.
- **Ralph Wiggum loop.** When the agent ends its session it is relaunched,
  up to five times. The agent itself judges whether its submission is
  complete, and relaunching stops when it says so. The number of relaunches
  actually used is not reported.
- **Grading.** The judge answers every leaf node, four runs per submission.
  Each node is combined by majority (3 of 4; a 2–2 tie counts as failure).
  Strict grading needs 100% of a condition's nodes to pass; "almost"
  grading needs ≥90%. The judge is agentic (codex-sdk) and reads paper,
  code and logs. The paper does not say whether it re-executes code. **The
  integrity check is therefore automated, and it is checked against the
  agent's own artifacts (its code and the logs it wrote), not against an
  independent re-run.** Compute: 1,942.81 h to generate, 62.95 h to grade.
- **Judge validation.**
  - *Expert agreement (App. A, Table 4).* Each scientist blind-read three
    anonymized submissions per paper, from Claude-Opus-5, Claude-Fable-5 and
    GPT-5.6-Sol. The questions were Q1 (met the direction?), Q2 (major
    weaknesses) and Q3 (quality vs the original). The criticisms were split
    into units and checked for rubric coverage and verdict agreement.
  - *Consistency (App. B, Table 5).* 52 submissions were graded ×4 by
    GPT-6-Astra/codex and by Claude-Opus-5.5/claude-code. Astra was chosen
    for paper-level consistency.
- **Trajectory audit (App. D).** The three behavior categories were fixed
  by the authors in advance. Claude-Opus-5 drafted per-trajectory labels.
  Three authors checked all 192 cells (64 trajectories × 3 behaviors),
  agreed on 156 and resolved 36 by discussion. The 64 trajectories come from
  seven agents. Claude-Opus-5.5 and GPT-6-Astra are not in Table 6, despite
  the text saying "spanning all nine".

## Results

- **Paper level (Table 2), strict.**
  - G1 (integrity) passes: GPT-6-Astra 11/15, GPT-5.6-Sol 6/15,
    Claude-Opus-5.5 4/15, every other agent 0/15.
  - G2 (complete): 1/135, from Astra, which also passes G1.
  - G3 (scientist's bar): 0/135.
  - Almost grading (≥90% of nodes): G1 86.67 / 60.00 / 60.00% for Astra /
    Sol / Opus-5.5, 0 for the rest. G3 up to 26.67% (Astra), 20% (Opus-5.5,
    Sol), 13.33% (Fable-5).
- **Node level.**

  | Agent | integrity nodes | direction nodes |
  |---|---|---|
  | GPT-6-Astra | 97.77 | 83.62 |
  | Claude-Opus-5.5 | 90.38 | 80.00 |
  | GPT-5.6-Sol | 88.73 | 74.89 |
  | Claude-Fable-5 | 66.50 | 71.00 |
  | Claude-Opus-4.8 | 65.26 | 60.15 |
  | Claude-Opus-5 | 59.50 | 66.83 |
  | DeepSeek-V4-Pro | 64.65 | 51.76 |
  | Kimi-K3 | 57.36 | 52.86 |
  | GLM-5.3 | 54.41 | 44.25 |

- **Integrity failure categories (Fig. 3, non-exclusive, n = 114).**

  | Category | count | spread (9 agents) |
  |---|---|---|
  | Paper/code mismatch | 90 | 0 (Astra) to 15 |
  | Reported numbers contradict evidence | 85 | 0 (Astra), 3 (Sol), 5 (Opus-5.5), 11–14 others |
  | False claims of completed work | 68 | 1 (Astra) to 12 |
  | Held-out data used in development | 23 | 1–4 for **every** agent |
  | Misleading comparisons | 21 | 0 for the three best agents |
  | Misreported baselines | 15 | 12 from the three open-weight agents |
  | Hard-coded results presented as computed | 2 | Kimi-K3, GLM-5.3 |

  Astra's four failures carry only two category flags (one false claim, one
  held-out use). So the seven categories do not cover all integrity
  failures. Some must come from other integrity nodes, such as held-fixed
  components.
- **Completeness.** 128/135 have inadequate required files or records:
  timestamped pre-registered plan files, per-seed logs, records of
  attempted work.
- **Direction failures.** Method/protocol unmet 120; analysis inadequate
  104; scientific targets missed 83. Open-weight agents fail the
  literature/originality nodes in 34/45 submissions, against 29/90 for
  proprietary agents.
- **Expert agreement (Table 4).**

  | Question | criticisms | coverage (paper-avg / pooled) | verdict agreement (paper-avg / pooled) |
  |---|---|---|---|
  | Q1 met direction | 41 | 100 / 100 | 100 / 100 |
  | Q2 weaknesses | 133 | 96.97 / 93.98 | 80.34 / 83.20 |
  | Q3 quality | 73 | 89.05 / 75.34 | 68.68 / 63.64 |

- **Judge consistency (Table 5).** Share of the 52 submissions where all
  four runs agree, Astra vs Claude-Opus-5.5 as judge:
  - same overall grade: 86.5% vs 57.7%;
  - G1 verdict: 86.5% vs 69.2%;
  - integrity node rating: 87.6% vs 92.7%.

  Claude is more consistent per node but less consistent per paper,
  because a single decisive node flips a conjunction.
- **Effort (Table 3, Fig. 6).** Median run time ranges from 1.3 h for Astra
  (313 steps) to 19.5 h for DeepSeek-V4-Pro (261 steps). Within paper,
  execution time vs pass rate gives Somers' D −0.313 on integrity
  (p_adj 0.0014) and −0.324 on direction (p_adj < 8×10⁻⁴). Within agent,
  D ≈ 0. The cross-agent association largely reflects that slow agents are
  weak ones.
- **Worked failures (Fig. 5).**
  - Claude-Fable-5 planned kernel families at n_k = 6 and 7, but launched
    only n_k = 7. It then built a completion checker from the launcher's
    config instead of the plan. The checker reported "41/41 … missing
    outputs: none", and the agent trusted it through later reviews.
  - GPT-5.6-Sol acknowledged that its method missed the sample-efficiency
    target and lost on all three transfer materials. It then wrote the
    paper "around that controlled finding" and ran a coverage check that
    "does not establish success".
  - An appendix trajectory notes: "every number was traceable and every
    macro defined, but two sections disagreed in prose."

## Critique / open questions

- **Against the digest framing:** BELOW. The digest's numbers are the
  abstract's and are right: 9 × 15, 0/135 valid, 114 integrity failures.
  But "114 fail the integrity check" reads as 114 established fabrications,
  and the body supports less:
  1. The check is an automated LLM verdict. One agentic judge decides, by
     3-of-4 majority with ties scored as failure, over a conjunction of
     every integrity node.
  2. There is **no false-positive control on integrity verdicts**. The
     expert study measures Q1–Q3 agreement, not whether flagged integrity
     violations are real. G1 verdicts agree across four judge runs on only
     86.5% of submissions.
  3. The "recorded evidence" is the agent's own code and logs. So the check
     tests internal consistency, not ground truth.
  4. The ordering is off: paper/code mismatch (90) is the top category, not
     "claims work never done" (68).
  5. The 114 is not a uniform frontier property. Six agents fail 15/15,
     and the best fails 4/15.
- **The judge is the top agent's model and harness.** GPT-6-Astra/codex-sdk
  grades every submission and is also the best agent, by a wide margin on
  integrity (97.77% of nodes; 4 failures against ≥9 for every other agent).
  Self-preference is never tested. The expert study sampled Claude-Opus-5,
  Fable-5 and Sol, so **Astra's submissions, the ones that drive every
  positive cell, have no human check**. The judge was chosen on
  self-consistency, and the authors note that "consistency does not imply
  correctness".
- **Integrity can be bought by doing less.** Astra has the shortest median
  run (1.3 h, 313 steps). An agent that runs fewer experiments and claims
  less has fewer numbers to contradict. Integrity is gated together with
  completeness for this reason, but the paper does not analyze the trade.
  Read "pass G1" as "no detected inconsistency", not "honest and
  productive".
- **Q1's 41/41 agreement is near-uninformative about discrimination.**
  Almost every submission fails, so agreeing that a paper did not meet its
  direction is the base-rate answer. The paper reports no agreement on
  passing cases. Agreement falls to 80% on Q2 and 64–69% on Q3, with wide
  intervals.
- **The text has drifted from the tables.** Section 5.2 says GPT-5.6-Sol has
  "the shortest median execution time at 2.9 hours" and "is the only agent
  with submissions that pass the integrity checks". Both are contradicted
  by Fig. 6 (Astra 1.3 h) and Table 2 (Astra and Opus-5.5 also pass G1).
  Section 5.2 node rates for DeepSeek, GLM and Kimi (52.55/45.09,
  59.16/54.13) differ from Table 2 (51.76/44.25, 57.36/52.86). The text's
  Somers' D values (−0.302; −0.149, p_adj 0.048) differ from Table 3
  (−0.324; −0.143, p_adj 0.0024). Fig. 6 is captioned "all 105
  submissions" (7 × 15) but plots nine agents. Table 5 mentions a "current
  grading policy". The most likely story is that two agents
  (Claude-Opus-5.5, GPT-6-Astra) and a judge-policy revision were added
  after the analysis text was written. The tables are probably current.
  The behavior audit covers only the original seven agents.
- **The behavior audit is confirmatory by design.** The categories were
  fixed in advance, Claude-Opus-5 drafted the labels, and authors adjudicated
  them, with 36/192 cells disputed. 58/64 is the rate at which a predefined
  label survived review, not an open-coded prevalence. Every non-Sol audited
  trajectory (52/52) carries the lenient self-verification label; Sol
  carries it on 6/12.
- **What the paper does not show.** It has no intervention: no stricter
  harness, no external completion gate, and no measurement of whether
  feeding the judge's verdict back would fix anything. The Ralph loop is
  the only "repair" mechanism, and its relaunch counts are unreported.
  There is one judge model for headline numbers and 15 tasks, so per-agent
  cells are n = 15. Domains are physical-science computation, not ML
  research proper, though P8/P12 and several tasks are ML-heavy.
- **Holdout contamination is measured, incidentally.** "Held-out data used
  in development" appears for every agent (1–4 of 15 each, 23 total) even
  though the task spec names the protocol. Data sit in the workspace and
  the internet is open.
- **Open:** would an external completion gate tied to the frozen T (the
  required deliverables list) have caught the 128 missing-records cases at
  session end, instead of the agent's own Ralph-loop self-assessment? The
  paper has everything needed to test this and does not.

## Trust signals

- **Credibility:** 3.
  - For: CMU LTI (Neubig, Welleck, Kim) with domain-scientist co-authors
    from six DOE labs. Code, dataset, all agent trajectories and a paper
    viewer are released (GitHub, HF, Pages). Rubrics were edited line by
    line by the original authors, and all 112 edits are printed. There is
    an expert meta-verification and a judge-consistency study with
    bootstrap CIs, and per-agent failure matrices.
  - Against: not peer reviewed. The single judge is the top-scoring agent's
    model and harness, with no self-preference test and no expert check on
    that agent's work. The headline integrity count has no false-positive
    audit. Several text claims in Section 5 contradict the paper's own
    tables. The behavior audit omits two of nine agents while claiming all
    nine.
  - Not 4 because the specific claim this hub would lean on (an integrity
    failure rate) rests on an unvalidated, possibly self-preferring judge,
    and the text/table drift shows weak final checking. Not 2 because full
    trajectories are public, so any cell can be re-audited, and "0 valid"
    is robust to judge noise given how far most submissions sit from 100%.

## Follow-up

- **Relevance:** 4. Material new evidence for
  [[concepts/evidence-gated-completion]]. Self-assessed completion inside a
  relaunch loop (the agent decides when it is done) left 128/135
  submissions missing required records. Agent-authored completion and
  traceability checkers pass while an external grader fails the work, and
  the Fable-5 case is a clean instance of a checker derived from the
  agent's own launcher instead of the frozen plan (the absence-fault
  pattern of [[literature/papers/waks2026what]]). It also gives a
  research-agent base rate for paper-to-log traceability failures
  ([[concepts/citation-anchoring]]) and for holdout use in development
  ([[concepts/hce-evaluation]]). Not 5: no gate is tested, so it adds
  failure base rates, not an effect of gating.
- Re-audit plan if this becomes load-bearing: sample flagged-integrity
  submissions from the released trajectories and hand-check them, to
  estimate the integrity false-positive rate. Compare Astra's submissions
  graded by a non-OpenAI judge.
- Candidates cited, not in graph: Chen et al. MLR-Bench (NeurIPS 2025; ~80%
  fabricated or invalidated results in coding-agent tasks); Kirgis et al.
  2026, "Can AI agents conduct open-ended AI research?" (arXiv 2607.27191);
  Si et al., "The ideation-execution gap" (ICLR 2026); Terminal-Bench-Science;
  Gandhi et al., "Cognitive behaviors that enable self-improving reasoners"
  (COLM 2025), the source of the verification/backtracking framing.
