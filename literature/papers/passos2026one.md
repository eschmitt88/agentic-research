---
kind: paper
title: "One Run Is Not Enough: Reminding a small Gemma 4 coding agent of its last finding made no measurable difference in a paired test"
authors: ["Patrick Passos"]
institutions: ["Renor"]
year: 2026
venue: "arXiv 2611.19359 (cs.AI), v1 2026-11-14, preprint (10 pages including references and a 1-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.19359"
code_url: "https://github.com/renorai/gemma4-memory-study"    # named in the Reproducibility paragraph, not opened at ingest
citations: null
source: "raw/papers/passos2026one.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/agent-native-memory]]"
tags: ["in-task-memory", "post-compaction-recap", "carry-over-slot", "compaction", "a-a-calibration", "flip-rate", "minimal-detectable-effect", "paired-mcnemar", "newcombe-interval", "preregistration", "documented-deviation", "post-result-extension", "sign-reversal", "validity-funnel", "gold-patch-check", "smoke-gates", "control-arm-replay", "treatment-delivery", "memory-adherence", "submit-loop", "gemma-4", "small-model", "swe-bench-family", "negative-result"]
---

# One Run Is Not Enough: Reminding a small Gemma 4 coding agent of its last finding made no measurable difference in a paired test

## TL;DR

One author tests a single harness change on a small coding agent:
Gemma 4 31B (QAT, 4-bit) in the Kaggle Gemma 4 Developer Agent harness.
After every compaction, the coder's instruction carries the explorer
sub-agent's latest answer (at most 15 lines) plus any notes the agent
wrote. The control is no memory. The tasks are paired SWE-bench-family
repository tasks, split over three backends, with a hash-frozen
preregistration and a timestamped change log.

Four results matter for this graph:

1. **The resolve-rate effect is null, and its sign did not hold.**
   - Primary (first 48 pairs, run 1): 29 vs 25, **+8.3 points**
     [−2.1, +18.3], McNemar p = 0.22.
   - Pre-registered extension on 50 new tasks, decided after that
     result: 18 vs 23, **−10.0** [−20.2, +0.7], p = 0.125.
   - Combined 98 pairs (secondary): 47 vs 48, **−1.0** [−8.3, +6.3],
     p = 1.0.
2. **The control flips against itself about as often as against the
   treatment.**
   - The no-memory arm, run 2 vs run 1, changed outcome on **9 of 48
     tasks (18.75%)**.
   - On Modal it moved +6.3 points, the same as the Modal treatment
     effect.
   - At that churn, 80% power needs about **±18 points at 48 pairs and
     ±12 at 98**.
   - All 5 tasks the treatment won in the primary were solved by the
     no-memory arm on its own rerun.
3. **The treatment mostly was not delivered, and it cost context.**
   - The model almost never wrote notes: 2 of 16 Modal runs and 4 of 32
     Kaggle runs (0 of 6 smoke runs).
   - Something was injected at only **47% (Modal) and 65% (Kaggle) of
     compactions**.
   - The memory plumbing adds about 2,000 tokens to every request.
     Compactions rose from 23 to 45 on Modal and from 71 to 77 on
     Kaggle.
4. **The measurement checks caught failures that would have looked like
   results.**
   - Only **37 of 129** gold patches passed in the public environment.
     Fixes raised this to 105 valid tasks.
   - Smoke gates caught three bugs, one of which would have run the
     memory arm with nothing in it.
   - A CPU replay showed the control arm's 33 model requests
     byte-identical before and after the memory plumbing.
   - Two hypotheses found on Modal (one at Fisher p = 0.0088) did not
     replicate on Kaggle.

## Claims

- Automatically carrying the explorer's last finding across compaction
  did not measurably change how often Gemma 4 31B resolved tasks. The
  combined interval excludes harms above ~8 points and gains above ~6.
- Single-run memory claims on task sets of a few dozen sit inside
  run-to-run noise. A 19% A/A flip rate means one run on ~50 tasks
  cannot tell a real effect from noise.
- "Small models do not write memory unless memory writes itself." All 11
  `plumbing_failed` runs on Modal were adherence failures (4 never ran
  the first recall, 7 had nothing to inject at a compaction), not
  machinery failures.
- Memory costs context, which causes more compactions. The trigger reads
  the previous prompt's size, so one large step can push input past the
  16,384-token point where output and input meet the 32,768 limit
  before compaction runs.
- Renor's internal product telemetry shows agents ask for memory on
  about 1 in 24 prompts unless recall is automatic. This is stated as
  internal and unpublished.
- "Memory reduces repeated actions" is not supported. The registered
  secondary is null, and no post-hoc test survives Bonferroni over 23.

## Methods

- **Setting.** Gemma 4 31B, QAT w4a16, thinking budget 4096, in the
  competition's declarative ADK harness. There is a coder agent and an
  explorer sub-agent, with a 100-tool-call cap and a 60-minute cap.
  Compaction fires when the previous prompt reaches 14,336 tokens.
- **Tasks.** 129 public development tasks (fastapi 67, rich 48,
  requests 13, httpx 1). Scoring uses the harness's `verify_task` in a
  per-task Python 3.13 environment. Agents run under the subprocess
  sandbox, not the official Docker scorer.
- **Arms.**
  - `none`: the baseline submission. Its build is byte-identical at
    every code pin, and a test rebuilds and compares it.
  - `recap`: after every compaction, two ADK session-state slots fill on
    each model call. `{explorer_finding?}` holds the explorer's latest
    answer (≤15 lines). `{working_notes?}` holds a notes page the agent
    writes through a notes sub-agent (≤1,500 tokens). The agent is asked
    to write a note after each explorer answer and each edit.
  - The arm began as agent-requested recall. Three smoke checks
    (6 runs) wrote no notes, so before any 48-task result it was moved
    to automatic injection and the explorer finding was added. This was
    the pre-registered fallback for near-zero note writes. No embeddings.
- **Allocation.** 48 tasks drawn once (seed 20261107) from the 105
  valid, stratified by repository. Modal H100: 16. Kaggle 4×L4: 32. The
  extension's 50 tasks on AWS L40S are the remaining valid tasks minus 7
  used for tuning and smoke checks. Seeded round-robin sent all 9
  requests tasks to Modal, so backend is confounded with repository.
- **Preregistration and analysis.**
  - The primary is resolve rate, intention to treat, run 1 vs run 1,
    pooled over 48 pairs. Its pooled run-1 form was **fixed 5 minutes
    before the first results commit**, which the paper labels a
    documented deviation.
  - The headline moved from cross-task lessons to the in-task recap.
    That decision came from no-memory traces only, before any memory
    run.
  - Statistics: backend-stratified bootstrap (10,000), Newcombe paired
    interval, exact McNemar. Run 2 and per-task means are sensitivity
    analyses.
  - Key secondary: repeated root-agent tool calls per compaction (same
    tool, normalized arguments), with a paired bootstrap and Wilcoxon.
  - Replication: two hypotheses discovered on Modal were frozen as code
    and tested on Kaggle before any Kaggle recap row was read.
  - Adherence (notes written) and delivery (share of compactions with
    something injected) were registered outcomes.
  - Extension: pre-registered at commit `bdf6c0e3`, after the 48-pair
    result and before any extension row. One run per arm, own two-sided
    primary. The combined 98-pair analysis is secondary.
  - 23 post-hoc tests, all listed in Appendix A. Pre-registered
    replicate runs of every task (Phase R) are reported elsewhere, not
    in this paper.
- **Measurement checks (§2.4).**
  - *Validity funnel:* gold patches pass on 37 of 129 tasks as shipped,
    68 after adding missing test dependencies, and 105 after installing
    the newest competition wheel each snapshot's pyproject allows. That
    step targeted 44 fastapi gold failures from a too-new starlette. 24
    exclusions follow written rules.
  - *Smoke gates:* small out-of-sample checks per arm before the
    48-task run. They caught a skill write flagged as path traversal;
    host `/workspace` and `/tmp` paths under the subprocess sandbox,
    which made memory report itself unavailable and would have shared
    notes across concurrent tasks; and agents that skipped recall and
    wrote no notes.
  - *CPU replay:* caught an interpreter path that would have failed
    every task in a fresh session. After masking sandbox ids, the
    no-memory arm sent 33 of 33 requests identical before and after the
    memory plumbing was added.
  - *A/A calibration:* the no-memory arm, run 2 against run 1, on the
    48 tasks.
- **Reproducibility.** `results/run_rows.jsonl` (one row per run),
  `analyze_rows.py` regenerates every number, and
  `paper/check_numbers.py` checks each number in the text against the
  analysis, rows and change log.

## Results

- **Resolve rate (Table 2).**

  | Set | recap / none | discordant (recap-only / none-only) | Δ pts [Newcombe 95%] | McNemar p |
  |---|---|---|---|---|
  | Pooled 48 (primary) | 29 / 25 | 5 / 1 | +8.3 [−2.1, +18.3] | 0.22 |
  | Modal 16 | 12 / 11 | 2 / 1 | +6.3 [−17.5, +29.3] | 1.0 |
  | Kaggle 32 | 17 / 14 | 3 / 0 | +9.4 [−2.1, +20.2] | 0.25 |
  | AWS 50 (extension primary) | 18 / 23 | 1 / 6 | −10.0 [−20.2, +0.7] | 0.125 |
  | Combined 98 (secondary) | 47 / 48 | 6 / 7 | −1.0 [−8.3, +6.3] | 1.0 |

  - Stratified bootstrap: [0.0, +18.8] for the 48, [−20.0, 0.0] for the
    extension, [−8.2, +6.1] combined. The authors note the bound at 0
    follows from the 5/1 and 1/6 splits.
  - Per-protocol (excluding `plumbing_failed`, 11 per backend): 26 pairs,
    13 vs 11, p = 0.5. Per-task mean over all runs: +5.2 [−3.1, +12.5].
  - The no-memory arm solved all 5 of its run-1 losses in its own run 2.
    The one task it won, `requests_7328`, was a recap run caught in the
    `submit_patch` loop.
  - Extension, descriptive: run-1 context overflow in 12 of 50 no-memory
    and 14 of 50 memory runs. 13 memory runs were `plumbing_failed`.
- **A/A calibration (Table 1, Fig. 1).**
  - Kaggle: 14 → 16 resolved, +6.3 [−9.4, +21.9], McNemar b = 4, c = 2.
  - Modal: 11 → 12, +6.3 [−12.5, +25.0], b = 2, c = 1.
  - 9 of 48 tasks changed outcome (18.75%). Our arithmetic: Wilson 95%
    about 10–32%.
  - Paired SE over 48 at that churn is about 0.063, so 80% power at
    α = 0.05 needs about ±18 points.
  - Our arithmetic: the treatment-vs-control discordance was 6/48
    (12.5%) on the primary and 13/98 (13%) combined, below the A/A's
    19%.
- **Repeats per compaction (Table 3).** Registered pooled: +0.30
  [−4.50, +5.98], Wilcoxon p = 0.27, n = 41. Modal's +8.54 comes from
  two looping recap runs. Kaggle's −4.45 includes one looping no-memory
  run and shrinks to −2.26 once post-submit submits are removed.
- **Post-hoc (App. A).** Only single-stratum tests fall below 0.05:
  Kaggle repeat rate without post-submit submits (0.029), Kaggle
  repeats per call (0.021 / 0.042), Modal compactions per task (0.032),
  and the Modal discovery of H-b (0.0088). None survives Bonferroni over
  23. No pooled test reaches 0.05.
- **Replication of Modal hypotheses (Table 4).**
  - H-a (recap compacts more before the first successful submit): Modal
    8/2/6, one-sided p 0.059. Kaggle 17/11/4, p 0.16.
  - H-b (recap leaks raw `<|channel>` thought text): Modal 6/0 of 16,
    Fisher p 0.0088. Kaggle 5/5 of 32, p 0.63.
  - Kaggle's no-memory arm already leaked in 5 and 3 of 32 runs, against
    0 of 16 on Modal. This was recorded before the replication. The
    authors attribute Modal's 6/0 to a different serving stack.
- **Mechanism, first 48 (Fig. 2, Table 5).**
  - Notes: 2 notes in 2 of 16 Modal runs, 5 notes in 4 of 32 Kaggle
    runs.
  - Delivery: 21 of 45 compactions (Modal) and 50 of 77 (Kaggle) had
    something injected. The mean per compacted run was 38% and 70%.
    Compacted runs with something injected at every compaction: 5/16
    and 19/28. Nothing can be injected before the first explorer call.
  - Context cost: median first coder request 3,966 tokens vs 2,311
    (Modal). Compactions 23 → 45 (Modal; post-hoc per task +1.38
    [+0.31, +2.69]) and 71 → 77 (Kaggle). Overflows: Kaggle 7 vs 5 (3 vs
    2 before any compaction), Modal 0 vs 1.
  - `submit_patch` loop: in 2 of 16 Modal recap runs, after an `ok`, the
    agent called it 441 and 385 more times with the same patch until the
    500-turn limit. The harness ends a run only on visible text with no
    tool call. No Modal no-memory run passed 120 model calls. Kaggle's
    one loop was in the no-memory arm. The loops do not change resolve
    outcomes, and they account for 879 of the recap arm's run-1 Modal
    repeats.

## Critique / open questions

- **Against the digest entry: at on the summary, with one framing
  slip; the *Why* line misdescribes the box's PreCompact hook.**
  - Every number in the summary is in the PDF and quoted correctly: 98
    pairs, 47 vs 48, −1.0 [−8.3, +6.3], +8.3 on the first 48, 19% A/A
    flips, ±12 power, rare notes, 47–65% delivery.
  - The summary leads with "Result: 47 vs 48". That is the combined
    *secondary*, which the paper says carries no confirmatory claim.
    The registered primary is +8.3 [−2.1, +18.3] on 48. The summary does
    give both, and the null reading is the paper's own. But "pre-registered,
    with a documented deviation" understates the two post-result
    elements. The primary's pooled form was fixed 5 minutes before the
    first results commit. The extension was decided after the 48-pair
    result.
  - The 19% is from 48 tasks on the no-memory arm only (about 10–32%
    Wilson, our arithmetic). The ±12 MDE is for 98 pairs; at the
    primary's 48 it is ±18.
  - **"A PreCompact-style re-injection"** is wrong about the box.
    `~/claude-system/claude/hooks/pre-compact.sh` injects a one-time
    reminder at compaction asking the agent to persist findings (run
    `/wrap`, write results, quote chat-only constraints) and to cite
    the transcript path. It depends on the agent's cooperation. The
    paper's treatment is the opposite design: a harness-filled slot on
    every model call after compaction. The paper's nearest analogue to
    PreCompact is its "write a note after each explorer answer and each
    edit" instruction. A 31B 4-bit model almost never followed it. That
    is an adherence data point for small models, not a test of the hook.
  - **"The A/A flip rate is the number `/elevate` should ask for"** is
    sound as method. The value does not transfer: it is one 4-bit model
    on one harness and ~48 tasks. [[literature/papers/liu2026what]]
    measured 13% on SWE-bench Verified hard45 with other models.
  - "The noise floor that items 1 and 6 talk about": item 1
    ([[literature/papers/zeng2026resolution]]) is about noise; item 6
    ([[literature/papers/guo2026how]]) is about skill rewriting and
    mentions run-to-run SD only in an appendix.
  - The PDF is the work the entry describes: same title, arXiv ID
    2611.19359, single author, v1 dated 2026-11-14.
- **The null is underpowered, and the paper says so.** At 19% churn,
  ±10 points at 80% power needs about 150 pairs, and ±5 about 600 (our
  arithmetic from the paper's SE formula). The combined interval
  excludes only effects above ~6–8 points. Effects of the size most
  harness changes claim are not ruled out.
- **Intention to treat is diluted by delivery.** On Modal, 11 of 16
  recap runs were `plumbing_failed`, and something was injected at
  every compaction in only 5 of 16 compacted runs. So ITT measures
  "turn the feature on", not "the agent received its last finding". The
  per-protocol check (26 pairs, 13 vs 11) is small and conditions on a
  post-treatment variable.
- **The slot content is narrow.** It is the explorer's last answer,
  ≤15 lines. It is not a task-state summary, a failure list or a
  constraint record, which are what the compaction literature in this
  graph finds summaries drop. A null here does not speak to carrying
  those.
- **Backend, repository and serving stack are confounded.** All 9
  requests tasks were on Modal. The extension ran on a third backend
  with a different repo mix (fastapi 26, rich 24). The sign reversal
  could be heterogeneity or noise, and the paper registered no
  heterogeneity test, so it claims neither.
- **The memory arm also changes the context budget.** ~2,000 extra
  tokens per request move compaction earlier. A null resolve rate is
  the net of whatever the carried finding buys and what the earlier,
  more frequent compactions cost. No arm separates them (no
  token-matched filler arm).
- **Internal inconsistencies.**
  - Threats to validity says "no memory resolved 23 of 32 Modal runs".
    Modal has 16 runs per arm, and Table 5 gives 11 resolved. 23 matches
    the AWS extension's no-memory count (23 of 50). This is likely a
    typo.
  - The A/A intervals differ between Table 1 and Fig. 1: Kaggle
    [−9.4, +21.9] vs [−9.3, +21.1], Modal [−12.5, +25.0] vs [−17.5,
    +29.3]. Fig. 1 is labelled Newcombe; Table 1's method is not
    stated.
- **What the paper does well.** Every interval is stated with what it
  excludes. Post-hoc tests are counted and corrected. A Modal p = 0.0088
  was tested on a held-out backend and attributed to a serving-stack
  base rate recorded beforehand. The control arm's bytes were checked
  unchanged by the treatment's plumbing. The deviations are in a
  timestamped log.
- **Limits of transfer.** One quantized 31B open model, a 32k window
  with a 14k trigger, SWE-style Python repository tasks, a subprocess
  sandbox and a scorer-like environment rather than the official Docker
  scorer. Frontier models write notes and hand-offs when asked
  ([[literature/papers/busch2026ctxagent]]: Opus 5.5 compacts at task
  boundaries), so the adherence result is specific to the model class.
  The misleading-memory control was dropped under a $25 Modal cap.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/liu2026what]]: rerun flips of 13% and no
    decision-grade difference below ~55 paired tasks.
  - [[literature/papers/busch2026ctxagent]] and the
    [[concepts/lossless-context-offload]] "offered, rarely used" series:
    memory that the agent must invoke mostly is not invoked. Here it is
    shown on writes, not reads.
  - [[literature/papers/wang2026harness]]: small local models, harness
    effects that move between collections.

## Trust signals

- **Credibility:** 3.
  - For:
    - A hash-frozen preregistration, a timestamped protocol change log
      and labelled deviations.
    - Code, per-run rows and a number-checking script named in the paper
      (not opened at ingest).
    - Paired design with exact tests and intervals throughout; all 23
      post-hoc tests listed and Bonferroni-corrected.
    - A/A calibration, a gold-patch validity funnel, smoke gates and a
      control-arm replay.
    - A pre-registered replication of discovered hypotheses on a second
      backend.
    - The abstract matches the body and leads with the null.
  - Against:
    - Single author at a small company (Renor), not peer reviewed, ten
      days old, no citations.
    - One small quantized model; n = 48 primary and 98 combined; k = 1
      in the extension.
    - Backend-repository confound; delivery under 70%.
    - Two internal number inconsistencies.
    - The telemetry claim is internal and unverifiable.
    - Phase R replicates are reported elsewhere.
  - Not 4: unreviewed, single author, and the design cannot resolve
    effects below ~12 points. Not 2: the measurement discipline and the
    released artifacts are well above the typical preprint in this
    graph.

## Follow-up

- **Relevance:** 4.
  - It is the first pre-registered paired test in the graph of a
    harness-side post-compaction carry-over slot, with a measured
    delivery rate and context cost. That is material for
    [[concepts/context-eviction-policy]].
  - It measures a same-arm A/A flip rate beside the treatment contrast
    on the same tasks and shows a post-result extension reversing the
    sign. That is material for [[concepts/pass-at-k]].
  - Its gold-patch funnel, control-arm replay and hypothesis holdout are
    worked preflights for [[concepts/hce-evaluation]].
  - Not 5: a coding agent on a small quantized model rather than an ML
    research agent, an underpowered null, and the box's own hook is not
    tested.
- **For `/elevate`'s evidence gate.** A single-run margin under ~±18
  points on ~50 paired tasks at this churn is a pilot. Ask for a
  same-configuration rerun flip rate on the same tasks, or the MDE it
  implies. Do not adopt 19% as a constant.
- **Candidates** (cited, not yet in the graph):
  - Fan et al. 2026, "VibeMemBench" (arXiv 2609.23570). Memory systems
    for coding agents on real repositories: 11 of 12 solver × memory
    pairings did not beat memory-off.
  - Gao et al. 2026, "SWE-MeM" (arXiv 2606.28434). Learned in-episode
    memory compression for long-horizon coding agents, with RL.
  - Bhola et al. 2026, "Code Isn't Memory" (arXiv 2606.22417).
  - Shen et al. 2026, subtask-level memory for SWE agents (arXiv
    2602.21611).
- **Phase R.** The pre-registered replicate runs of every task are
  "reported separately". If they appear, they give the treatment-arm A/A
  and a k ≥ 2 estimate on all 98 tasks.
