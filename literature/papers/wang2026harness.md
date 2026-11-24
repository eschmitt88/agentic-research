---
kind: paper
title: "Harness Leverage Across Model Size: A Ladder of Five Local Agent Harnesses from 0.75B to 4.7B"
authors: ["Hao Wang", "Ting Huang"]
institutions: ["School of Mathematics and Physics, University of Science and Technology Beijing", "Honor Device Co., Ltd."]
year: 2026
venue: "arXiv 2611.21407 (cs.AI), v1 2026-11-16, preprint (42-page main text + appendices, 56 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.21407"
code_url: "https://github.com/Mingbird/Mingbird-agent"    # named in the paper (suite, scorer, runner, harness); not opened at ingest. Per-cell archive "on request".
citations: null
source: "raw/papers/wang2026harness.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["agent-harness", "small-models", "local-inference", "completion-gate", "ablation", "quiet-stop", "exit-status", "artifact-scoring", "timeout-scoring", "two-readings", "loop-guard", "repetition", "preregistration", "holm", "collection-drift", "temperature-zero", "deterministic-checker", "partial-oracle", "quantization", "mingbird"]
---

# Harness Leverage Across Model Size: A Ladder of Five Local Agent Harnesses from 0.75B to 4.7B

## TL;DR

Five shipped agent harnesses run 18 artifact-producing tasks on four
local open-weight models (Qwen3.5 0.75B, 1.9B, 4.7B and Granite 4.2 3.7B)
on one 8 GB consumer GPU, one Ollama server and one Windows host. A
deterministic checker scores only the files left in the working
directory. One harness, Mingbird, is the authors' own. It is built around
a **completion gate**: six executable checks must pass before the
harness accepts the model's "done".

Four results matter for this graph:

1. **The completion gate is not shown to carry the harness's lead.**
   Disabling all six checks inside Mingbird changes the score by
   **+0.080 [−0.18, +0.32]**, **+0.117 [−0.05, +0.30]**, **+0.003** and
   **−0.004** at the four rungs. The two small-rung estimates are below
   the design's minimum detectable effect (**0.22–0.39**), and the two
   large-rung ones sit near the ceiling (full arm **0.910** and
   **0.980**). The only clear gate effect is at 1.9B. There, turning it
   off turns **0 of 18** empty runs into **3 of 18**. So the gate's value
   is unresolved where it has room to act and near zero where the
   harness already finishes almost everything.
2. **The lead is a progress effect, and one repeat-blocking rule carries
   much of it at 0.75B.** Removing only the streak rule (it blocks the
   same tool call 8 times in a row, or the same signature 4 times) costs
   **0.411 [+0.218, +0.608]** at 0.75B. On **8 of 18** tasks only the
   full arm writes any artifact, and the reverse never happens (sign test
   p = 0.0078). The baselines' empty runs are mostly one failing shell
   command repeated until the 32K window fills: **273 of 274 identical
   calls** in one pi cell.
3. **The quiet stop.** Of 357 usable cells, **127 leave none of their
   declared artifacts, and 85 of those exit normally**. **40** of the 85
   carry no error line in the transcript. The rate runs from **4 of 70**
   cells (Mingbird) to **34 of 72** (mini-swe-agent). Exit status, the
   transcript's error line and the artifact state are three different
   signals. Only the artifact state is the outcome.
4. **How a timed-out run is scored decided a pre-registered verdict.**
   The deployment reading scores a run that hits the wall budget as 0.
   The rescored reading scores the files it left. The two readings differ
   on only 58 cells, and they change conclusions:
   - On the precision ladder, the registered hypothesis is **contradicted**
     under the deployment reading (slope **+0.139 [+0.037, +0.235]**,
     Holm-adjusted p = 0.048). Under the rescored reading it is
     **undecided** (**−0.030 [−0.152, +0.098]**).
   - The Mingbird–pi gap at 4.85 bits goes from **+0.329** to **+0.023**.

   The paper reports both readings throughout.

## Claims

- Harness choice sets the level at every size. Mingbird has the highest
  mean at all four rungs. Two of the four pre-registered contrasts against
  goose survive Holm correction (0.75B adjusted p = 0.0067; 4.7B 0.0410).
  13 of 16 paired contrasts exclude zero under the deployment reading.
- Whether harness leverage grows or shrinks with model size cannot be
  settled with four rungs. Three design artifacts each produce a
  direction:
  - a floor arm (mini-swe-agent's reply contract) produces a rise in the
    span;
  - a ceiling at 4.7B produces a decline in the winner's margin;
  - a single cross-family rung (Granite at 3.7B) produces a peak.

  "Each is reproducible here by choosing the estimator."
- The completion gate the harness was built around is not the carrier
  "at three of the four rungs". The rest of the lead comes from machinery
  that keeps a weak model working after a tool error or inside a loop:
  a typed tool surface, a prompt that states the platform, loop and
  context guards, and compaction.
- "The failure to design against is the quiet stop." A benchmark that
  reads exit status as the outcome would credit every run that wrote
  nothing. "The fix is to score the artifact, not the process."
- A high checker score means conformance to a written specification, not
  correct work.

## Methods

- **Harnesses (as shipped, own prompts and tools).**
  - Mingbird v3.1.9: plan-execute loop, typed tools, the completion gate.
  - pi 1.3.0: minimal four-tool loop (bash, read, write, edit).
  - OpenCode 1.18.23 and goose 1.48.0: product harnesses.
  - mini-swe-agent 2.4.6: bash-only, needs a bash block in every reply.

  The baselines all stop when the model says it is finished, with a step
  or turn limit. The comparison is between products, so a difference
  between two harnesses cannot be pinned on one design decision.
- **Pinned conditions.**
  - Temperature 0 and thinking off for every arm. Sampling is forced
    through a logging proxy, and thinking-off is verified per rung with a
    probe.
  - 32K context, checked per cell from `/api/ps`.
  - Wall budgets are set per (task, model) and are the same across
    harnesses: 1.3× the maximum completed wall clock from an earlier
    collection, between 10 and 120 minutes.
  - Within a rung, harness order alternates by task.
- **Tasks and scoring.**
  - 18 held-out tasks frozen before the ladder ran: retrieval, synthesis,
    multi-step workflows and long-horizon runs. Examples: a literature
    map, a manuscript audit, a reproducibility audit. Each task mirrors a
    development task by family, with different instances.
  - The score is 4/7 milestone checks plus 3/7 final-artifact checks.
    Every check is binary and reads a named file. The judge dimension is
    disabled.
  - Of the 117 checks, 38% are `exists`, 21% `contains_any`, 15%
    `python_compiles` and 11% `min_words`. Only one runs the task's own
    pytest file.
- **Design.**
  - 360-cell ladder (5 × 18 × 4).
  - 216-cell mechanism gradient: full / guards off / text only at four
    rungs, plus a 54-cell repeat at 3.7B.
  - 72-cell repeat of the 0.75B and 4.7B rungs.
  - 36-cell loop-guard ablation at 0.75B.
  - A 35.5B MoE anchor, reported on the six easiest tasks.
  - 144-cell precision ladder: Granite at four bit widths, Mingbird vs pi,
    run on a second host.
  - Three of the 360 cells were excluded as infrastructure failures, under
    a stated rule.
- **Analysis.**
  - Task-level paired bootstrap (10,000 resamples) and Wilcoxon
    signed-rank tests.
  - A seven-test confirmatory family with Holm: span slope, gap shape,
    guard-contribution slope, and four per-rung Mingbird-vs-goose
    contrasts. goose was fixed as the comparator from an earlier matrix.
    Everything else is descriptive.
  - The minimum detectable effect at 80% power is reported (0.22–0.39).
  - The preregistration file (frozen 2026-10-22) ships with the per-cell
    archive, which is available on request.
- **Instrument checks.**
  - Split-half reliability of the five-arm profile: median 0.92 / 0.81 /
    0.93 / 0.96 by rung.
  - Development vs held-out difficulty correlation: +0.23 (p = 0.36), so
    the mirrored families give no parallel-form evidence.
  - Check-level reach: 112 of 115 check keys separate at least two arms.
  - A DOI-fidelity audit against the fixture.
  - A blind human audit: 60 cells, two reviewers, scored for
    completeness, correctness and use of evidence.
- **Failure coding.** Each cell gets two independent axes, read from the
  working directory and the runner record. One is artifacts: all, some or
  none. The other is termination: finished, wall budget, stall, error and
  so on. A third pass over transcripts flags error lines. Coding was not
  blind to the harness.

## Results

- **Mean artifact score, deployment reading (Table 4):**

  | Model | Mingbird | pi | OpenCode | goose | mini | Span |
  |---|---|---|---|---|---|---|
  | 0.75B | 0.664 | 0.172 | 0.112 | 0.186 | 0.012 | 0.653 |
  | 1.9B | 0.593 | 0.409 | 0.295 | 0.372 | 0.029 | 0.564 |
  | 3.7B | 0.762 | 0.714 | 0.000 | 0.262 | 0.279 | 0.762 |
  | 4.7B | 0.968 | 0.893 | 0.632 | 0.718 | 0.150 | 0.819 |

  Rescored reading: 1.9B Mingbird 0.732, pi 0.552; 3.7B Mingbird 0.841,
  pi 0.869, OpenCode 0.188.
- **Paired contrasts (Table 5).**
  - Smallest lead: pi at 4.7B, +0.075 [+0.012, +0.147].
  - Largest: mini-swe-agent at 4.7B, +0.819.
  - Intervals that include zero: pi at 1.9B (+0.184) and 3.7B (+0.005),
    and goose at 1.9B (+0.221 [−0.002, +0.436]). Rescored, the goose
    contrast at 1.9B becomes +0.360 [+0.131, +0.566], and the pi contrast
    at 3.7B becomes −0.020.
  - Against goose, Mingbird wins 14 / 11 / 11 tasks and loses 0 / 3 / 1
    at the first three rungs. At 4.7B the pair ties on 8 tasks, all at
    1.000.
- **Repeat collections.**
  - Mingbird at 0.75B averages **0.664, 0.503 and 0.431** in three
    collections of one configuration.
  - The gradient's full arm differs from the matrix by 0.233, 0.229,
    0.148 and 0.012 across the rungs.
  - A 72-cell rerun disagreed on 29 of 67 cells. The disagreement is
    bimodal, with a median absolute difference of 0.
  - The paired gap to goose held: +0.508 then +0.476 at 0.75B, and
    +0.250 then +0.256 at 4.7B. Restoring the excluded cells moves the
    0.75B repeat to +0.315 [+0.061, +0.541].
  - pi at 3.7B moved from 0.714 to 0.493 in another collection.
- **Leverage vs size (Table 6).** Slopes on log10 size:
  - span +0.198 [−0.078, +0.476];
  - span without mini-swe-agent +0.011 [−0.302, +0.257];
  - gap to goose −0.195 [−0.494, +0.100];
  - gap to the best baseline −0.591 [−0.836, −0.251].

  The last one falls because the best baseline changes from goose to pi,
  and pi climbs from 0.172 to 0.893.
- **Mechanism gradient (Table 7):**

  | Model | Full | Guards off | Text only | Guards Δ [95% CI] | Empty runs (full/off/text) |
  |---|---|---|---|---|---|
  | 0.75B | 0.431 | 0.351 | 0.386 | +0.080 [−0.18, +0.32] | 7 / 6 / 6 |
  | 1.9B | 0.822 | 0.705 | 0.784 | +0.117 [−0.05, +0.30] | 0 / 3 / 1 |
  | 3.7B | 0.910 | 0.907 | 0.812 | +0.003 [−0.03, +0.04] | 1 / 1 / 1 |
  | 4.7B | 0.980 | 0.984 | 1.000 | −0.004 [−0.05, +0.04] | 0 / 0 / 0 |

  - Slope of the guard contribution: −0.104 [−0.414, +0.220]. Holm-adjusted
    p = 1.000.
  - As a share of the lead over goose, the gate is 16% at 0.75B (the
    interval admits up to 63%), 53% at 1.9B, and under 2% above that.
  - A repeat of the whole 3.7B gradient puts the guard contrast at −0.020.
- **Where the lead lives.**
  - At 0.75B, cells the baseline left empty carry 51% (pi), 68% (goose)
    and 72% (OpenCode) of the gap. On cells both arms produced, Mingbird
    is still +0.443 ahead of pi and +0.439 ahead of goose.
  - At 1.9B, empty cells carry 90–94% of the gap.
  - At 4.7B, pi has no empty cells, and its whole +0.075 gap is depth.
- **Loop guard (0.75B, one batch, interleaved).**
  - Full minus streak-off: +0.411 [+0.218, +0.608]. Full is higher on 12
    tasks and lower on 1. On 8 tasks only the full arm writes an artifact.
  - The authors caution that the mean is close to how far a rerun moves
    a Mingbird cell at this rung (0.37 on average). The artifact split is
    the cleaner result.
  - Across the rung, no Mingbird cell exceeds 29 identical calls. pi has
    12 cells above 100, goose 8 and OpenCode 4.
- **Failure coding (Table 8):**

  | Arm | Complete | Partial | None | None, normal exit | None, budget | None, other |
  |---|---|---|---|---|---|---|
  | Mingbird | 52 | 14 | 4 | 4 | 0 | 0 |
  | pi | 37 | 20 | 15 | 10 | 4 | 1 |
  | OpenCode | 12 | 28 | 32 | 22 | 10 | 0 |
  | goose | 22 | 23 | 26 | 15 | 8 | 3 |
  | mini-swe-agent | 4 | 18 | 50 | 34 | 7 | 9 |
  | all | 127 | 103 | 127 | 85 | 29 | 13 |

  - The quiet stop depends on the rung. 43 of 48 empty cells exit
    normally at 0.75B, but only 10 of 30 at 3.7B, where most empty runs
    hit the budget.
  - No error line: 14 of pi's 15 empty cells and 22 of OpenCode's 32.
    mini-swe-agent mostly stops on its own `RepeatedFormatError` (34 of
    50). All four of Mingbird's empty cells carry an error line.
  - The Windows host matters. The workspace path contains a space and
    the baselines call a Unix shell, so "cd: too many arguments" appears
    in 11 of pi's 18 cells at 0.75B. Dropping the affected tasks leaves
    the gap to OpenCode at +0.494 (11 tasks). The gap to pi becomes
    +0.367 [−0.041, +0.714] on 7 tasks, which is unresolved.
- **The checker's validity.**
  - On HWF-01, Mingbird's artifact carries 28 DOIs, of which 22 are
    invented. pi's carries the fixture's 18. Both score 1.000.
  - In the human audit, inter-rater κ (quadratic-weighted) is 0.58–0.69,
    and the reviewer mean correlates with the checker at 0.57.
  - In **8 of the 34 cells the checker scored 1.000**, both reviewers
    marked the content wrong. The reviewer mean on those 34 cells is 0.56.
  - Human scores return the same ranking except that **goose and Mingbird
    trade the top two places**, and all their intervals overlap.
- **Cost (Table 9).** Mingbird has the lowest minutes per complete
  artifact set at every rung (10 / 11 / 6 / 5). OpenCode takes 195 minutes
  per set at 3.7B.
- **35.5B anchor.** On the six easiest tasks, goose scores 1.000 and
  Mingbird 0.833 (one cell timed out), a difference of −0.167
  [−0.500, 0.000]. Over all 18 tasks they score 0.952 and 0.944. Most of
  the model runs on the CPU.
- **Precision ladder (Table 10), Mingbird − pi.** Deployment reading:
  +0.029 at 3.15 bpw, +0.264 at 3.63, +0.329 at 4.85, +0.292 at 6.50.
  Rescored: +0.000, +0.264, +0.023, +0.004. At 6.50 bits pi hits the
  budget in 7 of 18 cells and Mingbird in none. At 4.85 and 6.50 bits,
  pi's deficit is time, not quality.

## Critique / open questions

- **Against the digest entry: at, with three overreaches and one miss.**
  The PDF is arXiv 2611.21407 by the two named authors, and every number
  in the summary is in it and quoted correctly. The entry has no
  check-at-ingest hint.
  - **"Disabling all six executable checks barely moves the mean"** is
    the abstract's framing, and it is too strong.
    - At 1.9B the point estimate (+0.117) is **about half** of the lead
      over goose. There the gate turns 3 empty runs into 0.
    - At 0.75B the interval allows up to 63% of the lead.
    - Both small-rung estimates are below the design's minimum detectable
      effect of 0.22–0.39, so they are non-detections, not measured
      zeros.
    - The tight zeros are at 3.7B and 4.7B, where the full arm already
      scores 0.910 and 0.980 and there is nothing left for the gate to
      add.

    The fair reading is that the gate's contribution is not resolved
    where it could act, and is near zero at the ceiling.
  - **"Removing one repeat-blocking rule costs 0.411"** comes from one run
    per cell in one batch. The authors themselves say the mean sits near
    the per-cell rerun noise. The robust part is the 8–0 split in which
    tasks produce any artifact. Comparing 0.411 with the gate's 0.080
    also sets contrasts from two different collections side by side. Each
    contrast is paired, so this is acceptable, but it is not one
    experiment.
  - **"A scorer that reads exit status credits work never written"** holds
    by definition. No scorer in the study reads exit status. What the
    paper measures is how common the quiet stop is: 85 of 357 cells
    (24%, our arithmetic), concentrated in mini-swe-agent and OpenCode,
    at 0.75–4.7B on a Windows host whose path breaks Unix shells.
  - **The *Why* line ties the wrong number to `hce-score-coverage-count`.**
    - The 85 quiet stops are not ungraded items. The artifact-reading
      scorer graded them, correctly, as zero. That is a case of *which
      signal the scorer reads*, not of coverage.
    - The coverage-shaped result is the **two readings of timed-out
      cells**. 58 budget-hit cells with work on disk are scored either as
      0 or from that work. The choice decided whether the pre-registered
      precision hypothesis was contradicted or undecided, and whether the
      1.9B goose contrast excluded zero.
    - That is the boundary case the proposal itself flags as its main
      risk: is an agent's own timeout a bad submission or a missing
      grade? The paper's answer is that it is two estimands (finished in
      time vs work present), and both should be reported.
  - **Missed by the entry: the headline ranking rests on the authors' own
    instrument.**
    - The authors wrote Mingbird, all 18 tasks, the scorer and the
      analysis plan.
    - Under blind human scoring, goose and Mingbird swap the top two
      places and the intervals overlap.
    - The checker gave 1.000 to an artifact with 22 of 28 DOIs invented,
      and to 8 of 34 cells that both reviewers judged wrong.

    "Mingbird leads at every size" is a result about checker
    conformance on the authors' suite.
- **The cross-harness ranking at the small end is partly an environment
  effect.** The workspace path contains a space, the host is Windows, and
  the three Unix-shell harnesses fail on it. Mingbird is the only arm whose
  prompt states the platform. The authors keep the asymmetry because "it
  is part of what the system is", and they test it: the OpenCode gap
  survives the exclusion, the pi gap does not resolve. A Linux host would
  probably shrink the 0.75B gaps to pi and OpenCode.
- **The gate and the scorer read similar things.** The gate's
  finish-claim artifact check and artifact-content check test what the
  checker's `exists` and `contains_any` checks test. Even with the gate
  aimed at the scorer, removing it moved little. That points to the base
  rate: the rest of the harness already leaves few empty runs for the
  gate to catch.
- **A gate cannot rescue a run that makes no progress.** At 0.75B the
  full arm leaves 7 of 18 runs empty in the gradient collection, as many
  as with the gate off. A completion refusal cannot make a model that
  loops produce an artifact. What helped at that rung was bounding the
  repeat. Compare [[literature/papers/lee2026dont]]: refusals raise
  quality where the goal is reachable.
- **Levels are not stable objects.** A configuration's mean moved by up
  to 0.233 between collections at temperature 0. The authors handle this
  correctly: only paired, within-collection contrasts carry claims, and
  the decisive pair is re-collected. But only Mingbird and goose were
  re-collected. pi, OpenCode and mini-swe-agent rest on one collection
  each.
- **Size is confounded with calendar time and quantization.** The rungs
  were collected in sequence. The two smallest Qwen tags are Q8_0 and the
  rest Q4_K_M. The precision ladder measures the quantization effect on
  one Granite checkpoint and states that it does not transfer to other
  rungs.
- **The failure coding was not blind** (directory names carry the
  harness). It is mechanical (file existence plus the runner record), so
  the risk is low.
- **Small models only.** Nothing here measures frontier models. The
  related-work section cites the opposing results (Starace 2026 on GAIA:
  the strongest model gained most from scaffolding; Hu et al. 2026:
  direction depends on task type). This ladder cannot adjudicate between
  them, and says so.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/li2026who]]: removing a commit gate moved Pass
    by about 3.5 pp, and most of the gain came from the feedback loop.
    Here the gate moved little, and most of the lead came from progress
    mechanisms.
  - [[literature/papers/vats2026good]]: a check on an agent whose finish
    requests are mostly right is pure cost. Here, at the top rungs, a
    gate on a harness that already finishes adds nothing, though it
    costs nothing measurable either.
  - [[literature/papers/liu2026silent]]: the same kind of
    process-vs-outcome error, in the opposite direction. There, ungraded
    episodes were stored as zero. Here, a normal exit could pass for
    completion.

## Trust signals

- **Credibility:** 3.
  - For:
    - The task suite, fixtures, deterministic scorer, runner and harness
      are public. The per-cell archive is available on request.
    - A preregistration with a stated confirmatory family, Holm
      correction, and a minimum detectable effect reported beside every
      null.
    - Both readings are reported at every contrast, and the authors
      report the result that contradicts their own design hypothesis.
    - Repeat collections, an exclusion sensitivity check, split-half
      reliability, check-level reach, a DOI audit and a blind two-reviewer
      human audit.
    - Served artifacts are verified per cell (context window, processor
      split, thinking-off probe).
  - Against:
    - Not peer reviewed, eight days old, no citations.
    - A small group (a university mathematics school plus an industry
      co-author), with no established agent-evaluation record.
    - The authors built one of the five harnesses, every task and the
      scorer. Human scores do not reproduce the top-place result.
    - One run per cell, 18 tasks, a Windows-specific failure mix, and
      small models only.
  - Not 4: unreviewed, and the main ranking is measured on the authors'
    own suite and instrument. Not 2: released code and data, a
    preregistration, and an analysis that reports against its own
    hypothesis.

## Follow-up

- **Relevance:** 4.
  - It is a within-harness, pre-registered **removal arm for an
    executable completion gate**, run at four model sizes with intervals,
    on a suite held out from development. That is the kind of evidence
    [[concepts/evidence-gated-completion]]'s open hold asks for, and the
    measured delta is small to unresolved.
  - It is the first measured case in the graph where the scoring of
    timed-out runs flips a pre-registered verdict. That bears on
    [[concepts/hce-evaluation]] and on the pending
    `hce-score-coverage-count` proposal.
  - It adds a temperature-0 re-collection drift figure (0.23) for
    [[concepts/pass-at-k]].
  - Not 5: small local models, not ML-research agents, and the main
    ranking depends on the authors' own instrument.
- **For `/elevate`.** See the reader's report. In short:
  - the timeout-reading result bears on the wording of the coverage
    proposal's risk paragraph;
  - the coordinator's fail streak reads unit exit status, which is the
    signal a quiet stop passes;
  - the paper is a second source, after houlamy2026who, for adding a
    constructed wrong answer to the known-bad preflight;
  - it is no support for adding a completion gate as a first lever.
- **Candidates** (cited, not yet in the graph):
  - Wang and Huang 2026, "Mingbird: A local-first agent harness enabling
    small open models to complete real tasks" (arXiv 2610.02001). The
    design paper and the 288-cell development matrix.
  - Cho 2026, "It's not the size: Harness design determines operational
    stability in small language models" (arXiv 2605.12129). Three harness
    levels on 2–3B models. A minimal shell can be worse than raw
    prompting.
  - Hu et al. 2026, "Beyond the model: Demystifying harness effects in
    software engineering agents" (arXiv 2609.32459). Ten models, harness
    gains by capability.
  - Starace 2026, "Scaffold effects on GAIA: A controlled comparison"
    (arXiv 2606.08529). Pre-registered. The strongest model gained most.
  - Zhao, Poskitt and Lo 2026, "Pairing matters" (arXiv 2610.32131).
    Six harnesses × three coding models on SWE-bench Verified.
  - Park et al. 2026, "System optimizations for cost-efficient agent
    harnesses" (arXiv 2611.12122). Accuracy depends mostly on the model,
    and the harness moves cost by up to 2.6×.
  - Zhang et al. 2026, "Stop comparing LLM agents without disclosing the
    harness" (arXiv 2605.23950).
  - Gai, Block and Risteski 2026, computational separations for
    scaffolds (arXiv 2610.27157). Theory predicting a harness effect that
    shrinks with size.
