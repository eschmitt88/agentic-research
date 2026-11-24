---
kind: paper
title: "The Between-Round Decision in Autonomous Research: An LLM Controller That Reads the Whole Trajectory"
authors: ["Caoxin Wang", "Shengkai Gan", "Xiang Hu", "Lili Pan"]
institutions: ["University of Electronic Science and Technology of China", "Zhejiang University"]
year: 2026
venue: "arXiv 2611.22661 (cs.AI), v1 2026-11-17, preprint (9-page main text + 7-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.22661"
code_url: "https://github.com/learninginvision/BRED"    # repo exists (created 2026-11-12, ~2.8 MB); contents not inspected at ingest
citations: null
source: "raw/papers/wang2026between.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/constraint-pinning]]"
tags: ["between-round-decision", "research-controller", "champion-and-log", "one-incumbent-loop", "redirect", "restart-pivot", "stop-decision", "context-ablation", "internagent-benchmark", "single-run", "selection-on-reported-metric", "winners-curse", "budget-asymmetry", "held-out-seeds", "daic-woz"]
---

# The Between-Round Decision in Autonomous Research: An LLM Controller That Reads the Whole Trajectory

## TL;DR

BRED is a research loop with one LLM call (deepseek-v4-pro) per round that
decides what the round does. It reads two objects: the **Champion** (the
best verified solution so far, with code, metric and a mechanism
signature) and a **log** of past rounds (decision, every candidate's result
or failure diagnosis, and a facts-only lesson). It picks one of four
actions: start a new line from the clean baseline, extend the Champion with
a retrieved mechanism, refine the Champion without retrieval, or stop. It
also writes a goal-level instruction of at most 40 words and up to three
hard constraints. The other steps run 3–5 candidates per round; a Claude
Code agent backed by MiniMax-M3 does the coding. The Champion is replaced
only on a strict improvement. Guardrails in code force refine-or-stop in
the last 30% of rounds and force a new line after three stalled
extensions.

Five results matter for this graph:

1. **Every reported number is one run, selected on the metric it
   reports.** On the three InternAgent-benchmark tasks, the headline is
   the best of 24–40 candidates on the task's own evaluation, with no
   separate selection split. The final scores are **MAE 0.4172** on ETTh1,
   **95.07%** on SST-2 and **84.63%** on CIFAR-100. Each run lasted ten
   rounds or fewer. No seeds, no intervals and no costs are reported.
2. **The context ablation is a three-step ladder, one run per step, on
   two tasks.** ETTh1 MAE goes 0.4299 / 0.4244 / 0.4172 for no history /
   last round only / full history. CIFAR-100 goes **83.71 / 84.01 /
   84.63**. But "no history" also means **every round restarts from the
   baseline**, so that arm can never extend or refine anything. It is a
   loop with no hill-climbing, not just a controller that reads less.
3. **The same-model control has a large budget asymmetry.** Re-run with
   BRED's models, InternAgent iterates on ideas before execution and then
   **runs three** of them. BRED runs 24–40. On ETTh1 the no-history arm
   (same budget as BRED) still loses to that re-run (0.4299 vs 0.4245). On
   CIFAR-100 it beats it (83.71 vs 83.44), so there the margin can be
   explained by more executions.
4. **Restart pivots rarely beat the incumbent (our tabulation of App. E).**
   After a Champion exists, **1 of 8** new-line-from-baseline rounds
   beat it. **6 of 10** extensions did, and **6 of 11** refinements did
   (two of those six by 0.00003 MAE and 0.003 F1 points). The controller
   left a lineage after **one** failed extension every time (4 of 4), so
   the three-stall code backstop never fired. On CIFAR-100 all the gain
   after R3 (83.91 → 84.63) came in the code-forced refine-only end phase.
5. **The one held-out check is held-out seeds, not held-out data.** On
   DAIC-WOZ depression detection the in-run Champion is 0.6019 macro-F1,
   the maximum over 44 candidates. Re-run on five fresh seeds over the
   same 188 subjects, it scores **0.6164 ± 0.0166**. That is 4.0 points
   above the strongest published system under the same leakage-free
   protocol.

## Claims

- An LLM can make the between-round decision from accumulated evidence,
  with no tree, score, schedule or filter around it.
- Performance improves monotonically as the decision step reads more of
  the accumulated evidence (no history < last round < full history).
- BRED beats published systems on three benchmark tasks and beats
  InternAgent re-run with the same models, "so the gap is not a matter of
  the backbone."
- On ETTh1 the margin over re-run InternAgent "appears only when the
  controller reads accumulated evidence, not from running more
  experiments", because the no-history arm at the same budget loses to
  the re-run.
- Unproductive rounds "cost the loop nothing it has already won",
  because the Champion never regresses.
- The log's lessons record only what was measured and may not attribute
  an outcome to a mechanism. Interpretation is left to the controller, so
  "one noisy measurement does not become a causal story that later
  rounds inherit." This is asserted, not ablated.
- Several structures are left out (pattern–action table, a forest of
  dormant directions with revive, a composite mechanism stack, multi-axis
  round plans). App. A says this is for simplicity and that the ablation
  "does not test whether the structures ... would add further gains."

## Methods

- **Loop (Alg. 1).** DECIDE → GENERATE → RUN → RECORD, for T rounds
  (T = 10 in every run).
  - **DECIDE** is one deepseek-v4-pro call at temperature 0.7. The prompt
    holds the task, the floor, the Champion, the log (one line per round
    plus full detail for the last five), remaining budget, a stall count,
    and a **frontier survey** of published lines and their results. The
    survey is the same in every round and every ablation setting.
    Fig. 1 labels it as DeepSeek-generated; how it is built is not stated.
    Output fields: situation, mode, base, instruction (≤ 40 words,
    goal-level, "not a recipe"), 0–3 hard constraints (passed verbatim to
    every candidate, compliance logged), rationale and stop_reason. The
    full prompts are given (App. C.2).
  - **GENERATE.** For exploration, a search planner writes six plans, a
    retriever reads the top three of up to 30 arXiv papers per plan in
    full, and an idea forge drafts one idea per plan. A strict LLM
    reviewer scores feasibility and grounding, and the N most feasible
    passing ideas run. Refinement skips retrieval and drafts five
    incremental edits to the Champion's code. These rarely pass the
    grounding gate, so refinement candidates are usually just the N most
    feasible.
  - **RUN.** Each candidate gets its own workspace copy. Claude Code
    (claude-agent-sdk) backed by MiniMax-M3 implements, executes and
    repairs it under the frozen protocol.
  - **RECORD.** An LLM reads the candidate's actual code and writes a
    ≤ 12-word mechanism signature. Diverged results are discarded. The
    round's best replaces the Champion only on a strict improvement, with
    no noise margin. Failures and their diagnoses are logged.
- **Guardrails in code.**
  - Rounds start from the baseline until a first Champion exists.
  - The last 30% of rounds (R8–R10 at T = 10) allow only refine or stop.
  - Before that, after three consecutive extensions without progress, a
    further extension is turned into a new line (the prompt suggests
    switching after about two).
  - A stop is refused if the Champion improved in either of the last two
    rounds.
- **Tasks.** Three from the InternAgent benchmark, with its datasets,
  baselines and metrics: AutoTSF (ETTh1, DLinear, MAE averaged over four
  horizons), AutoClsSST (SST-2 accuracy) and AutoCls2D (CIFAR-100 top-1).
  The fourth, AutoDepDAIC, is outside the benchmark: DAIC-WOZ speech
  depression detection, macro-F1, under HAREN-CTC's leakage-free protocol
  (188 pooled subjects, subject-stratified 5-fold CV over five seeds, all
  choices made inside each fold's training part).
  - CIFAR-100's baseline is the **published** WideResNet number (81.20%).
    The in-repo baseline is a three-epoch smoke run (49.23%) used only as
    the floor.
  - Candidates per round: 3 (TSF, SST), 4 (CIFAR), 5 (DAIC).
- **Comparators (Table 1).** Dolphin and InternAgent numbers are copied
  from InternAgent's paper. Those were obtained with GPT-4o and
  Claude-3.7-Sonnet, on other hardware, as maxima across runs. InternAgent
  was also re-run from its public repo (commit fa8c3ee) with BRED's models
  and coding agent on the same hardware. Its protocol iterates on ideas
  before execution and then runs the top three.
- **Ablation (§4.4).** It varies only what DECIDE sees: none (no
  Champion, no log, every round from the baseline), the last round only
  (not the Champion), or everything. GENERATE's ledger of tried directions
  is rendered from the full log in all three. Only AutoTSF and AutoCls2D
  were run, one run per setting.
- **Hardware.** Five RTX 2080 Ti (11 GB). LangGraph orchestration.

## Results

- **Main results (Table 1).**

  | Task | Baseline | Dolphin | InternAgent pub. | InternAgent re-run | BRED |
  |---|---|---|---|---|---|
  | ETTh1 MAE ↓ | 0.4382 | 0.4627 | 0.4331 | 0.4245 | **0.4172** |
  | SST-2 Acc ↑ | 91.63 | 92.50 | 93.50 | 92.78 | **95.07** |
  | CIFAR-100 Acc ↑ | 81.20 | 82.00 | 83.30 | 83.44 | **84.63** |
  | DAIC macro-F1 ↑ | 56.10 | — | — | — | **61.64** (HAREN-CTC 57.6) |

  - Against the strongest prior entry: −1.72% MAE, +1.57, +1.19 and +4.04
    points.
  - The abstract's "+4.80% / +3.44 / +3.43" are gains over each task's
    baseline. CIFAR-100's baseline is a published number, not one
    measured in this stack.
- **Ablation (Table 2), best verified result of one run each.**

  | Decide reads | ETTh1 MAE ↓ | CIFAR-100 Acc ↑ |
  |---|---|---|
  | No history | 0.4299 | 83.71 |
  | Last round | 0.4244 | 84.01 |
  | Full (= the main run) | 0.4172 | 84.63 |

  - The full-history row equals the main-run numbers, so it is the same
    run reused.
  - On ETTh1, no history reaches 0.4299 at R4 and makes no further gain.
    "Some directions are repeated across rounds."
  - First round to beat the floor (App. G): R1 for last-round, R2 for
    no-history, R4 for full history. The authors treat first crossing as
    a diagnostic, not a quality measure.
- **Trajectories (App. E, Tables 5–8).**
  - **ETTh1.** R1–R3 open three survey lines from the baseline; none beats
    the floor. R4 opens a cross-channel line, which gives the first
    Champion (0.4226). R5–R7 extensions improve it each round to 0.4172.
    The R8 refinement "improves" it 0.41724 → 0.41721. The loop stops at
    R9, one round early.
    - The first Champion carries 74% of the final gain over the floor
      (our arithmetic).
  - **SST-2.** R1 follows the survey to a RoBERTa-base encoder: 94.61%,
    already above InternAgent's published 93.50%. R3, a new line with
    consistency regularisation, gives 94.95. The R8 refinement gives
    95.07. It stops at R10.
    - **R1 carries 2.98 of the 3.44-point gain over baseline (87%)**,
      before any history existed (our arithmetic).
  - **CIFAR-100.** R1–R3 build one WideResNet lineage to 83.91. R4's
    extension fails, and R5–R7 open three new lines from the baseline
    (81.62, 80.51, 82.05). End-phase refinements R8 and R10 lift the
    Champion to 84.29, then 84.63. The run uses the full budget.
  - **DAIC-WOZ.** The Champion reaches 59.94 at R2. R4 and R5 return no
    valid metric from any of their five candidates. R7 gives 60.19. R8
    "improves" it 60.188 → 60.191. It stops at R10.
- **Pivot outcomes (our tabulation, Tables 5–8, rounds after the first
  Champion).**

  | Action | Rounds | Beat the Champion |
  |---|---|---|
  | New line from baseline | 8 | 1 (SST R3, +0.34) |
  | Extend the Champion | 10 | 6 |
  | Refine the Champion | 11 | 6 (2 of them < 0.001 in metric units) |

  - The controller's four switches from extending to a new line each came
    after a single failed extension (CIFAR R4, SST R6, DAIC R3 and R5).
  - On CIFAR it then ran three losing new lines in a row. It returned to
    the Champion at R8, which is the first round in which the guardrail
    allows only refine or stop.
- **Stop decisions.** The LLM stopped one round early on ETTh1 (R9), SST
  and DAIC (R10), and never stopped on CIFAR. There is no run-to-budget
  counterfactual, so the value of stopping is not measured.
- **DAIC held-out seeds.** Seeds 100–104 give 0.6164 ± 0.0166 macro-F1 and
  0.6902 ± 0.0151 AUC. Every seed is ≥ 0.5987.
  - The fresh-seed mean is *above* the selected in-run 0.6019, and the
    per-seed SD (1.66 points) is more than five times the 0.3-point
    refinement moves the controller kept promoting.
  - The protocol penalises signal-free models. An all-positive predictor
    scores ≈ 0.23. A port of DepAudioNet scores at chance (0.474 / AUC
    0.467).

## Critique / open questions

- **Against the digest entry: below.** Its numbers are correct. It
  leaves out the confounds that decide what they mean, and its *Why* line
  claims a test the paper does not run.
  - "With no history the same loop gains less" is the paper's framing.
    The no-history arm also cannot build on any candidate, so the
    ablation conflates **what the controller reads** with **whether the
    loop hill-climbs**. The no-history arm is closer to repeated restarts
    at matched candidate count, with GENERATE still reading the ledger of
    tried directions. The 0.92-point CIFAR gap is about the size of the
    0.72 points the full run won in its forced refine-only phase, which
    no-history may not have been able to use. The paper does not say what
    the end-phase guardrail does when the controller cannot see a
    Champion.
  - **"The nearest thing yet to the missing redirect-vs-halt arm"**
    overreaches. No arm varies halting, and none compares redirecting
    with continuing at a fixed budget. The ablation varies context. The
    nearest real evidence is descriptive: the per-round tables
    (tabulated above) show restart pivots losing to the incumbent in 7
    of 8 rounds.
  - **Check at ingest: seeds.** There are none on the three benchmark
    tasks. Each table cell is one run, scored as the best of its 24–40
    candidates on the evaluation it reports. Table 1's caption and §1
    say the main figures follow the benchmark's "maximum performance
    across runs" / "best final Champion" convention. How many
    full-history runs were made, and whether the reported one was the
    best of several, is not stated. If it was,
    the ablation ladder compares a selected run with single runs. Only
    DAIC has a repeat (five fresh seeds on the same subjects).
  - **Check at ingest: baseline budget.** It is not matched. InternAgent
    re-run executes three ideas against BRED's 24–40 candidates. With
    selection on the reported metric, that is also a winner's-curse
    asymmetry: max of 3 against max of 30+. The paper's answer (the
    no-history arm loses to the re-run on ETTh1) holds on one task, at
    one run each. On CIFAR it goes the other way. Dolphin's numbers
    come from other models and hardware. No arm is matched by tokens,
    dollars or wall-clock, and none of these is reported.
  - The digest's "beats task baselines" is right. Its "ahead of the
    published results" holds for the published figures, but the
    CIFAR-100 baseline is itself a published number, not measured in this
    stack.
  - The PDF is the work the entry describes: arXiv 2611.22661v1, four
    authors, 2026-11-17, code linked. The entry's ID, date and quoted
    numbers are correct.
- **Selection on the reported metric (HCE §4).** For the three benchmark
  tasks the Champion is chosen and reported on the same evaluation, as
  the benchmark convention does. The authors saw the problem for DAIC
  ("0.6019 is the maximum over 44 candidates") and re-ran there only.
  [[literature/papers/hu2026winners]] and
  [[literature/papers/zeng2026resolution]] show how large this inflation
  can be. The sub-point margins on CIFAR and SST are of the size such
  inflation produces.
- **The strict-improvement keep promotes noise.** Two Champion updates
  are 0.00003 MAE and 0.003 macro-F1 points. On DAIC the per-seed SD is
  1.66 points. The "never regresses" guarantee is about the recorded
  number, not the true quality. This is the `band=none` case of
  [[concepts/budget-as-ceiling]]'s noise-band rule.
- **Much of the gain is from the survey and the first rounds, not
  between-round control.** On SST, 87% of the gain came at R1 from the
  survey's encoder choice. The survey of published lines and results is
  task knowledge given to the controller in every setting. Whether
  InternAgent's re-run got equivalent knowledge is not stated.
- **Redirects happen readily here, unlike in
  [[literature/papers/yan2026traceml]] and
  [[literature/papers/vats2026good]].** The prompt names the actions,
  gives a stall count, and says to switch after about two stalls. The
  controller switched after one, every time. But the restarts mostly
  lost, which matches [[literature/papers/zhou2026athanor]] and
  [[literature/papers/srinivasan2026continue]]. So a named action menu
  can elicit pivots; it does not make them pay.
- **The "facts-only lesson" rule is untested.** No arm lets the lesson
  writer attribute causes. The authors list "a wrong lesson would still
  reach later decisions" as a limitation.
- **One controller model, two ablation tasks, three of the benchmark's
  twelve tasks** (the authors' own limits). The project-level tasks were
  not attempted.
- **AI-assisted writing disclosed.** Claude drafted text, code and
  figures. The prose is unusually clean and well-hedged in places, and it
  still frames the ablation as a pure context effect.

## Trust signals

- **Credibility:** 3.
  - For:
    - University group (UESTC, corresponding author Lili Pan; Zhejiang
      University) with a public code repo.
    - A same-model re-run of the main comparator from a pinned commit.
    - Full DECIDE prompts and output schema, and per-round tables for
      every main run.
    - A leakage-free protocol on DAIC with a held-out-seed re-run and
      chance-level sanity baselines.
    - Limitations stated plainly, including the left-out structures and
      the untested lesson rule.
  - Against:
    - Not peer reviewed, one week old, no citations.
    - One run per cell everywhere, with the Champion selected on the
      reported metric for three of four tasks.
    - Budgets not matched against the comparator, and no costs reported.
    - The ablation's no-history arm changes the loop, not only the
      context.
    - Whether the reported full-history runs are best-of-several is
      unstated.
  - Not 4: unreviewed, and the comparative and ablation claims rest on
    single selected maxima. Not 2: code, prompts, a same-model control and
    the per-round record are more than most preprints in the graph give.

## Follow-up

- **Relevance:** 4.
  - It studies the exact decision `/iterate` makes each cycle: refine
    the best, extend it, start over, or stop. It is the first paper in
    the graph to vary how much trajectory the deciding call reads.
  - Its per-round tables add a third source to
    [[concepts/budget-as-ceiling]]'s finding that restart pivots lose to
    the matured lineage. It adds a one-incumbent data point to
    [[concepts/evolutionary-expansion]]'s losing-alternative arm, and an
    ablation of the log for [[concepts/structured-world-model]].
  - Not 5: single runs, a confounded ablation and test-set selection,
    so it is not canonical evidence for any concept.
- **For the redirect-vs-halt question.** Still open. A cheap reanalysis
  of the released logs would give a per-round q (the chance that
  continuing the current action improves) and p (the chance a pivot
  improves) in [[literature/papers/jain2026monitor]]'s terms. That
  assumes the repo ships the run logs, which was not checked.
- **What a clean version of the ablation needs.** Hold the build-from-
  Champion ability fixed and vary only what DECIDE reads. Use ≥ 3 runs
  per arm and score the Champion on a split it was not selected on.
- **Candidates** (cited, not yet in the graph):
  - Si et al. 2026, "Towards execution-grounded automated AI research"
    (arXiv 2601.14525). Evolutionary search with an annealed exploit
    share over hundreds of parallel executions per epoch, and GRPO on
    execution reward that raised the mean but not the max.
  - InternAgent / NovelSeek (arXiv 2505.16938). The benchmark and main
    comparator here.
  - Virk et al. 2026, "SwarmResearch" (arXiv 2607.02807), and Hogan et
    al. 2026, "AlphaLab" (arXiv 2604.08590). LLM-chosen
    explore/refine with memory or playbook ablations.
  - Juneja et al. 2026, "ARTS" (arXiv 2606.21891). Declined from the
    2026-07-13 digest. This paper cites its memory ablation.
  - Li et al. 2025, R&D-Agent-Quant (arXiv 2505.15155). Cited for an LLM
    scheduler losing to a bandit on a two-way choice.
