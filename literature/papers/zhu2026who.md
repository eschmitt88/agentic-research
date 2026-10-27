---
kind: paper
title: "Who Verifies the Agent's Success Claim? Self-Report Leakage in GUI-Agent Completion Judging"
authors: ["Yunfeng Zhu", "Shengcheng Yu", "Zihan Li", "Zhenyu Chen"]
institutions: ["State Key Laboratory for Novel Software Technology, Nanjing University", "Technical University of Munich (Heilbronn)", "Nanjing Normal University"]
year: 2026
venue: "arXiv 2610.29609 (cs.SE), v1 2026-10-21, preprint (artifact is labelled ICSE27, so probably an ICSE 2027 submission)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.29609"
code_url: "https://anonymous.4open.science/r/ICSE27-3C77/README.md"    # anonymized artifact: labels, splits, masked traces, prompts, schemas, checker code, cached predictions; not opened at ingest
citations: null
source: "raw/papers/zhu2026who.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["self-report-leakage", "llm-judge", "trajectory-judging", "completion-judging", "claim-blind", "report-isolation", "dangerous-misreport", "silent-false-pass", "gui-agent", "androidworld", "evidence-policy-boundary", "deterministic-checker", "policy-replay", "causal-accomplishment", "vacuous-success", "do-nothing", "open-weight-judges", "qwen", "deepseek", "held-out-batch", "declared-vs-verified"]
---

# Who Verifies the Agent's Success Claim? Self-Report Leakage in GUI-Agent Completion Judging

## TL;DR

A GUI agent finishes a run and says "Done". An LLM judge reads the task,
the recorded trace of screen states and, in the usual setup, the agent's
own report, and decides whether the task was completed. The authors hold
the judge, task, trace serialization and scoring fixed and only include
(AWARE) or withhold (BLIND) the agent's self-report. The positive class is a
**dangerous misreport**: an unhedged success claim that the trace does not
show the agent itself causing.

Three results matter for this graph:

1. **The report pulls the judge toward "completed".** On 465 human-labelled
   records (180 dangerous misreports), withholding the report raised the
   judge's trace-only non-completion recall by 43.4 pp (Qwen3.5-35B-A3B) and
   13.3 pp (DeepSeek-284B-A13B). The flips go one way: 79 harmful to 1
   recovery for Qwen, 31 to 7 for DeepSeek. The held-out batch shows the
   same direction (+38.0 / +14.0 pp), significant for Qwen but not for
   DeepSeek (9 vs 2 flips, p = 0.065).
2. **Withholding is not the whole fix.** A claim-blind auditor in which the
   model fills typed evidence fields and deterministic code decides beats a
   claim-blind prompt-level judge (PROMPT-BLIND) on recall and F1: F1
   0.75 / 0.78 vs 0.62 / 0.64 on the combined set, 0.77 / 0.81 vs 0.58 /
   0.63 on held-out Batch 2. It gets there by flagging more, so precision
   drops slightly.
3. **Final-state checks pass do-nothing runs.** The worked failure is a
   "delete the duplicate recipes" task where the list was already empty at
   the start. Replaying the cached evidence under a final-state-only policy
   instead of "the agent caused it" lowers F1 to 0.71 / 0.72 and raises the
   silent-pass rate by 0.07 / 0.13.

Only two open-weight judges of mid size were tested, in one domain (Android
GUI). The larger judge leaked about a third as much as the smaller one.

## Claims

- In a one-shot claim-aware judge, the report "ceases to be merely a claim to
  be verified and instead becomes part of the evidence for its own truth".
  That makes the evaluation path circular.
- Report isolation is "an empirical design constraint rather than an
  optional prompt choice". Claim intent should be read from the report
  alone, and completion judged from the task and trace alone.
- Isolation "should be more than a prompt instruction". Separating evidence
  extraction (model) from policy (deterministic code) makes the audit record
  inspectable, regression-testable and replayable under a new policy without
  new model calls.
- As an audit-stage gate where both FLAG and REVIEW block automatic pass, the
  auditor keeps 74–90% of dangerous misreports from passing silently at
  45–56% inspection load. Over all 1,384 in-scope runs the load is about
  16–19%.
- Most remaining misses are trace-side errors that a human could spot in
  the same trace, so a stronger trace estimator is the main lever.
- "A system under audit should not provide evidence for its own success."

## Methods

- **Corpus.** 1,384 in-scope runs of two GUI agents (AppAgent and
  Mobile-Agent-v3) on AndroidWorld, an AndroidWorld "extension" that is not
  described further, and Themis tasks. Runs the harness status already
  settles are excluded: clean passes, timeouts, max-round hits, errors, and
  missing or invalid reports. That leaves 465 audit-target records, 180 of
  them dangerous misreports (39%). Each agent contributes exactly 90
  positives (177 and 288 records). The paper does not say whether that
  balance was by design.
- **Gold labels on two blind axes.** Completion y\* is labelled from the task
  and trace without the report. Claim intent c\* is labelled from the report
  without the trace. Positive = c\* = SUCCESS ∧ y\* = NOT COMPLETED. Two
  calibrated annotators who were not involved in developing the auditor
  labelled each axis: κ 0.78 (completion), 0.80 (claim), 0.79 (derived
  label). A third party adjudicated the disagreements. The construct is
  "causal accomplishment": a goal state already present at s0, an empty
  target list, or typed-but-unsaved text does not count.
- **Splits.** Batch 1 (335 records, 130 positives) was used for development.
  Batch 2 (130 records, 50 positives, including all 38 Themis records) was
  fixed by provenance before any scoring and held out from prompt, schema,
  parser, checker and policy development. The specification was frozen
  before Batch 2 was scored. Record-ID lists and validation scripts are in
  the artifact.
- **RQ1 intervention.** The same judge reads the same task and serialized
  trace. AWARE appends the agent's self-report, which is **the final report
  and the per-step self-reports** ("deleted recipe X" after each step).
  BLIND withholds both. Trace-only ΔR is the recall shift before claim
  gating. The claim-gated detector view combines that verdict with a shared
  claim reader. Exact sign tests compare harmful and recovery flips.
- **Structured auditor.** D = A(C(m), G(x, E(x, τ))).
  - C is one temperature-0 call that reads only the final report and labels
    it SUCCESS / FAILURE-OR-INCOMPLETE / UNCLEAR.
  - E reads only the task and trace. It emits five typed fields (action,
    target_id, transition, committed, final_supports_goal), each with a
    value in YES / NO / UNKNOWN / N/A, a cited trace support and a
    rationale. E "is not allowed to output COMPLETED or NOT COMPLETED".
  - G is deterministic. A frozen rule maps the task text to one of six
    classes, and each class has a required-field set. Any required NO gives
    NOT COMPLETED. Otherwise any UNKNOWN gives UNKNOWN. All YES gives
    COMPLETED.
  - A is a fixed table. SUCCESS with NOT COMPLETED gives FLAG. Agreement
    gives PASS. Uncertainty on either side gives REVIEW.
- **Baselines.** Trust-claim, flag-all, claim-gate-only, naive claim-aware,
  matched claim-aware (same claim gate and 5-run aggregation), PROMPT-BLIND
  (report withheld, prompt-level verdict), and an adapted Agent-Eval-Refine
  (AER) prompt. All multi-run methods use 5 temperature-0 runs and a 3-of-5
  vote. CIs are 95% bootstrap intervals over cached predictions. The
  evaluation made 23,500 calls, 99.62% of which parsed cleanly. The median
  trace is about 22K tokens and no trace was truncated.

## Results

- **RQ1, report exposure (Table II).**

  | Scope | Judge | AWARE F1 (R) | BLIND F1 (R) | Trace-only ΔR | Harmful / recovery | p |
  |---|---|---|---|---|---|---|
  | Combined | qwen | 0.37 (0.23) | 0.62 (0.49) | +0.43 | 79 / 1 | < 0.001 |
  | Combined | deepseek | 0.59 (0.44) | 0.64 (0.52) | +0.13 | 31 / 7 | < 0.001 |
  | Batch 2 | qwen | 0.33 (0.20) | 0.58 (0.42) | +0.38 | 19 / 0 | < 0.001 |
  | Batch 2 | deepseek | 0.59 (0.46) | 0.63 (0.52) | +0.14 | 9 / 2 | 0.065 |

  On the claim-gated detector, which is what a pipeline would actually use,
  the recall gain is +26 / +8 pp combined and +22 / +6 pp on Batch 2.
- **RQ2, detection (Table III).**

  | Scope | Model | Matched claim-aware P/R/F1 | PROMPT-BLIND P/R/F1 | Structured P/R/F1 [F1 CI] |
  |---|---|---|---|---|
  | Combined | qwen | 0.84 / 0.42 / 0.56 | 0.85 / 0.49 / 0.62 | 0.82 / 0.69 / 0.75 [0.69, 0.80] |
  | Combined | deepseek | 0.82 / 0.52 / 0.63 | 0.82 / 0.52 / 0.64 | 0.76 / 0.79 / 0.78 [0.73, 0.82] |
  | Batch 2 | qwen | 0.91 / 0.40 / 0.56 | 0.91 / 0.42 / 0.58 | 0.85 / 0.70 / 0.77 [0.67, 0.86] |
  | Batch 2 | deepseek | 0.80 / 0.56 / 0.66 | 0.81 / 0.52 / 0.63 | 0.78 / 0.84 / 0.81 [0.72, 0.88] |

  - Claim-gate-only reaches P 0.45, R 0.77, F1 0.57. It is listed with model
    "n/a", so it is apparently not the auditor's LLM claim reader.
  - Against PROMPT-BLIND, on the combined set, the auditor gains 40 / 49
    positives, loses 3 / 0 and adds 13 / 23 false positives. On Batch 2 it
    gains 15 / 16, loses 1 / 0 and adds 4 / 6 false positives. The paired
    tests give p ≤ 0.001 in all four settings.
  - The single-run F1 is 0.73 ± 0.01 / 0.78 ± 0.01, so the 5-run vote
    mainly stabilizes the result.
  - Against AER on the same backbone, ΔF1 is +0.28 / +0.20 (McNemar
    p < 0.001).
  - On the per-agent subsets F1 is 0.74 / 0.77 (AppAgent) and 0.76 / 0.78
    (Mobile-Agent-v3). The Themis slice (14 positives) is too small to
    report.
- **Deployment gating (Table IV).** With FLAG and REVIEW both blocking:
  - Combined: 138 / 180 blocked at 48% load (qwen) and 159 / 180 at 56%
    (deepseek). Silent-pass rates are 0.23 [0.18, 0.30] and 0.12
    [0.07, 0.17].
  - Batch 2: 37 / 50 and 45 / 50 blocked, at 45% and 54% load.
  - Inspection yield is 0.61–0.64.
- **RQ3, policy replay over cached evidence.**
  - *Final-state policy*: F1 0.71 / 0.72, silent-pass rate +0.07 / +0.13.
  - *Low-load policy*: 15 / 28 fewer inspections, silent-pass rate
    +0.08 / +0.13, F1 0.66 / 0.70.
  - *5-of-5 vote*: deepseek blocked recall rises 0.88 → 0.93 (combined)
    and 0.90 → 0.96 (Batch 2) for about 6 points more load. For qwen it is
    a real trade: silent 0.23 → 0.16, but automatic F1 falls 0.75 → 0.70,
    and on Batch 2 0.77 → 0.65.
  - *Gold claim labels* in place of C: F1 0.82 / 0.84.
- **Where it still fails.** Of 42 qwen silent positives, 6 are claim-side, 31
  are trace-side and visible to a human reader, and 5 are limits of a
  passive trace. The deepseek split is 6 / 12 / 3. Human trace-only readers
  reach F1 0.90 / 0.95 against gold (Table V), against 0.82 / 0.84 for the
  trace channel. By task class, create/edit is weakest (qwen R 0.58) and
  delete is strongest (R 0.84, blocked R 0.95). Of the 79 harmful flips,
  the claim-blind trace channel judges 77 as not completed, and 67 end in
  FLAG.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest entry is in the PDF. Three things are missing.
  1. **The headline uses the bigger metric.** The 43.4 / 13.3 pp are the
     *trace-only* recall shift before claim gating. On the claim-gated
     detector, the view a pipeline uses, the gain is 26 / 8 pp (F1 0.37 →
     0.62 and 0.59 → 0.64). For DeepSeek on Batch 2 it is 6 pp and the flip
     test is not significant (p = 0.065). The digest's "same trend" is
     accurate; "same trend" is all it is.
  2. **"The success report" is two channels.** AWARE adds the final report
     *and* the per-step self-reports. The design cannot say which one leaks.
     For this box the distinction matters. A subagent's final summary and
     the running narrative in `log.md` reach a judge by different routes.
  3. **The structured auditor's gain is recall bought with some precision,
     at one operating point.** PROMPT-BLIND is more precise (0.82 vs 0.76
     for DeepSeek combined). The auditor's closed-world rule (any required
     NO means not completed) is stricter by construction, and it encodes the
     gold construct field by field. The paper does not say whether the
     PROMPT-BLIND prompt states the same causal-accomplishment boundary
     rules (pre-existing goal state, empty list, unsaved edit). If it does
     not, part of the RQ2 gain is construct alignment, not "structure". The
     final-state replay (F1 0.71 / 0.72, still above PROMPT-BLIND) suggests
     structure is worth something without the construct, but that replay
     changes only G, not the prompt.
- **What RQ1 shows well.** It is a clean single-factor intervention with a
  very one-sided flip pattern (79 : 1). The gold labels are built so the
  claim never enters the completion label. This is the controlled version of
  [[literature/papers/xu2026dont]] (a supportive log raised false accepts
  from 0.07–0.19 to 0.78–0.90 on fixed frames) in a second domain, from an
  independent group. Like xu2026dont it uses open-weight judges only, and
  the effect falls with judge size (Qwen 43 pp, DeepSeek 13 pp). xu2026dont
  found frontier judges (GPT-5.5, Opus 5) nearly flat. So for this box's
  Claude judges the size of the leak is unmeasured and probably smaller.
  The case for isolation is that it costs nothing and holds without
  measuring the judge.
- **The audit-target set is enriched and conditioned on the harness.** 66%
  of runs are excluded before the auditor runs, and the positive rate in
  what remains is 39%. The authors say so and report full-stream load
  separately. Precision and yield do not transfer to a stream with a
  different base rate. The "clean passes" the harness settles through
  "status and report-integrity checks" are not characterized, so we cannot
  tell how many misreports that upstream filter itself passes.
- **The load comparison with trivial baselines is underspecified.**
  Claim-gate-only has recall 0.77 at precision 0.45. Our arithmetic: about
  308 flags (66% load) for about 139 positives caught. That is about the
  same catch as qwen's gate (138) at 18 points more load. DeepSeek's gate
  is clearly better (159 at 56%). But the residual analysis puts only 6
  silent positives on the claim side. If C reads about 97% of positives as
  SUCCESS, claim-gate-only cannot be using C, and its definition is not in
  the text.
- **The human ceiling is probably optimistic.** Table V's "Human annotator
  A / B" appear to be the gold annotators scored against the adjudicated
  labels that they produced, which inflates their F1. The headroom estimate
  (0.90–0.95 vs 0.82–0.84) is an upper bound.
- **Evidence fields are not validated field by field.** "Checkable" means
  the record can be inspected, not that the fields are right. The dominant
  residual error is the extractor emitting all-YES on a trace a human reads
  as not completed. That is a model error inside the trusted half, and the
  deterministic policy cannot catch it. Compare
  [[literature/papers/guo2026groundability]]: small models misread the
  decisive line even when it is in front of them.
- **The trace is assumed honest.** The auditor trusts the recorded trace and
  scopes out trace manipulation and omission. On a harness where the agent
  can write its own transcript ([[literature/papers/qin2026llm]]), the
  "independent evidence channel" holds only if the trace is recorded
  outside the agent's reach. On Android that is plausible, since the screen
  states come from the device. In a coding or research loop it is not
  automatic.
- **Domain and scale.** The domain is Android GUI only, with two agents and
  six task classes. Batch 2 has 50 positives, and a single Themis record
  moves F1 by about 0.07. Model names (Qwen3.5-35B-A3B, DeepSeek-284B-A13B)
  are hosted endpoints, and API drift affects reruns but not the cached
  metrics. No closed or frontier judge was tested.
- **Fit with the graph.** This agrees with
  [[literature/papers/sanayei2026evidencefirst]]: a judge with the trace but
  not the state "believed the account", and the active ingredient was
  reading the object the claim is about. It also agrees with
  [[literature/papers/xu2026what]], where the report is a lossy view of the
  trajectory. zhu2026who adds the reverse direction: the report is not
  only thin, it actively bends a reader who also has the full trace. The
  "extractor may not emit a verdict" rule is
  [[literature/papers/qu2026propose]]'s "instruments, not verdicts", one
  level down: the model may write evidence, and code holds the verdict.

## Trust signals

- **Credibility:** 3.
  - For:
    - A clean single-factor RQ1 design with paired sign tests.
    - Gold labels on two mutually blind axes, from annotators independent
      of the auditor's development. κ is about 0.79 before adjudication.
    - A provenance-fixed held-out batch, including a source (Themis) absent
      from development, with a specification frozen before scoring.
    - Bootstrap CIs, single-run stability, and a leave-one-agent-out check.
    - Negative and trade-off results reported, e.g. qwen's 5-of-5 F1 drop
      and the Themis slice declared too small.
    - An anonymized artifact with labels, splits, prompts, checker code and
      cached predictions (named in the PDF; not opened at ingest).
    - The group (Nanjing University SE lab, Zhenyu Chen; corresponding
      author now at TUM) has a long record in mobile GUI testing.
  - Against:
    - Not peer reviewed (probably an ICSE 2027 submission). No citations
      yet (6 days old).
    - Two open-weight judges, one domain.
    - The leaked channel bundles final and per-step reports.
    - The RQ2 comparison may confound structure with construct alignment.
    - The audit set is enriched and conditioned on an uncharacterized
      upstream filter.
    - The human ceiling is likely optimistic.
    - The related-work novelty claim ("not aware of prior agent-evaluation
      work that treats report exposure as an information-flow hazard")
      misses xu2026dont (2026-09).
  - Not 4: unreviewed, and no frontier judge, which is the case this box
    depends on. Not 2: the RQ1 effect is large, one-directional,
    tested on a split frozen before scoring, and reproducible from the
    released cached predictions.

## Follow-up

- **Relevance:** 3. It is the second controlled measurement, after
  xu2026dont and from an independent group, that a verifier given the
  agent's own account drifts toward accepting it. It is also the first to
  pair that measurement with a claim-blind, deterministic-policy auditor
  and a held-out test. It adds material evidence to:
  - [[concepts/evidence-gated-completion]]: the account has to be absent,
    not merely outweighed, and the model should emit fields, not the
    verdict.
  - [[concepts/information-firewall]]: a new boundary, between the claim
    and its own verifier.
  - [[concepts/typed-claim-partition]]: claim intent and trace support are
    typed separately and combined by a fixed table, with UNKNOWN routed to
    review.
  - [[concepts/hce-evaluation]]: a final-state check passes a do-nothing
    run when the start state already satisfies the goal. This bears on the
    pending `hce-known-bad-preflight`.

  Not 4: GUI domain, open judges, and no completion gate inside a live
  loop. It grades fixed traces, so the hold on a refusal-driven outcome
  delta in evidence-gated-completion is untouched.
- **For this box (our inference).** Any skill step where a model judges
  whether work is done while reading the worker's own summary has this
  shape. Examples are an orchestrator reading reader-subagent reports, or
  `/elevate` reading a concept's dated claims. The cheap version is
  PROMPT-BLIND: hand the judge the artifact and the trace, not the
  worker's "done". That already beat claim-aware judging on recall in both
  judges. The structured version needs a fixed required-evidence list per
  task type. That fits `/ingest`, whose outputs are typed files, better
  than open-ended research steps.
- **For `hce-known-bad-preflight` (pending).** The vacuous-delete case is the
  do-nothing shape the proposal targets, from a fifth group, with a price:
  final-state judging adds 0.07–0.13 silent passes. Our inference: the
  do-nothing submission should be scored from the task's *actual starting
  state*. In an ML workspace that means one that may already contain
  predictions or checkpoints, not a blank one. Otherwise an evaluator that
  reads leftover state passes the pre-flight. A one-clause refinement, not
  new evidence for Gate 1.
- **Candidates** (cited, not yet in the graph):
  - Lù et al. 2025, "AgentRewardBench: Evaluating automatic evaluations of
    web agent trajectories" (arXiv 2504.08942).
  - Pan et al. 2024, "Autonomous evaluation and refinement of digital
    agents" (AER, arXiv 2404.06474). This is the adapted comparator.
  - Ye et al. 2026, "Claw-Eval: Towards trustworthy evaluation of
    autonomous agents" (arXiv 2604.06132).
  - Sogani et al. 2026, "When web agents finish but still fail: Reproducible
    triggers and trace diagnostics for parallel web exploration" (arXiv
    2606.20724).
  - Li et al. 2026, "Preference leakage: A contamination problem in
    LLM-as-a-judge" (arXiv 2502.01534).
  - Same digest, item 19 (2610.25947, "Claim Laundering"). The
    decision-model side of the same failure: records outweigh observations.
