---
kind: paper
title: "Aligning Autoresearch: Co-Improvement between Agents and Humans"
authors: ["Tianjian Li", "Chuanyang Jin", "Seungone Kim", "Tong Chen", "John Hoffman", "Yoram Bachrach", "Pedro Rodriguez", "Ping Yu", "Eryk Helenowski", "Swarnadeep Saha", "Jack Lanchantin", "Weizhe Yuan", "Ilia Kulikov", "Jason Weston"]
institutions: ["FAIR at Meta", "Johns Hopkins University", "Carnegie Mellon University", "University of Washington"]
year: 2026
venue: "arXiv 2610.38882 (cs.AI), v1 2026-10-28, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.38882"
code_url: "https://huggingface.co/datasets/facebook/hackbench"
citations: null
source: "raw/papers/li2026aligning.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["reward-hacking", "hack-localization", "transcript-monitor", "hackbench", "synthetic-injection", "grader-audit", "known-bad-preflight", "benchmark-creation", "autobenchmark", "human-in-the-loop", "co-improvement", "task-specification", "user-simulator", "rubric-rl", "llm-judge", "known-ordering", "researchgym", "autolab"]
---

# Aligning Autoresearch: Co-Improvement between Agents and Humans

## TL;DR

A 72-page FAIR report in four loosely linked parts. Each part has its own
setup and its own small evidence base.

1. **AutoBenchmark (human role in benchmark creation).** Muse Spark 1.1
   builds a 10-task benchmark over 10 loop iterations, with in-loop
   solvers, five LLM quality judges, and held-out solvers. One run per
   setting. With no human input it picks its own benchmark, and solvers
   score about 80–90 on it. A one-sentence human intent barely helps.
   A detailed human spec **plus a curated source corpus** lowers the
   selected checkpoint's solver scores by 28–46.5 points. In one forked
   run, three human tips on *how* to build the tasks restarted a stalled
   loop (Muse Spark 64.2 vs 88.1 unaided).
2. **HackBench (the HCE-relevant part).** 169 long autoresearch
   trajectories (median 1.06M characters, 264 commands), each containing at
   least one hack. 128 have **planted** hacks: GLM-5.2 inserts the hack
   commands and patches the later observations. 41 have natural hacks. An
   LLM monitor must return the hacking command indices. The best monitor,
   Claude Opus 5, gets **F1 64.9** (P 69.7, R 72.6). Six of the eight
   models score below 53.
3. **AutoInteract.** RL on multi-turn coding tasks with learned user
   simulators. Qwen3.5-4B's average Pass@1 goes from 29.9 to 43.0, and
   27B's from 49.1 to 55.7. One training run per cell.
4. **RL-XAR.** Standard LLM judges rank model writing above expert human
   writing: pairwise in 63.5–84.6% of cases, generated rubrics in 100%.
   Rubrics meta-optimized to widen the human–model gap reverse this, and
   RL against them improves Qwen3.5-27B. The authors' own blind check
   prefers the RL-XAR model 16–2 on papers and 19–1 on stories.

Three things matter for this graph. (a) A transcript monitor is a leaky
backstop. Even the best one misses about a quarter of the gold hack
commands, and the benchmark never tests it on a clean run. (b) Appendix E
audits 12 graders from ResearchGym and AutoLab, which are real ML-research
benchmarks. By our count, 9 of the 12 can be beaten by a no-op, empty or
constant submission. This is the first base rate in the graph for the
pending known-bad preflight on ML-research evaluators. (c) In the
benchmark-building loop, a human was useful when the input was a detailed
spec plus a curated source corpus. A one-line goal behaved almost like no
human. That evidence is n = 1 per setting.

## Claims

- Human researchers are still "pivotal", especially at the proposal stage.
  Feedback helps "only when it is concrete".
- Current autoresearch agents can run the benchmark-creation loop end to
  end. Unaided, they produce near-saturated benchmarks.
- An LLM benchmark verifier catches defects the solver score cannot see:
  leaked answers, memorized constructs, and cited sources that were never
  retrieved.
- Current frontier LLMs "are not aligned and are unable to achieve hack
  localization sufficiently". All monitors "miss many hacks".
- Hacks in autoresearch are task-specific, look like ordinary progress one
  action at a time, and show up only in long transcripts. So detection
  needs task understanding, not string matching.
- Training on simulated, real-user-grounded multi-turn interaction improves
  both interactive and single-turn coding.
- Off-the-shelf LLM judges prefer model "slop" to expert human writing.
  Rubrics learned to separate the two fix this and make a usable RL
  reward.

## Methods

**Part 1, AutoBenchmark (§1, App. A–B).**
- The proposer is Muse Spark 1.1, run in OpenHands, with one fresh session
  per iteration. It writes a Harbor task package: instructions, assets,
  reference solution, and grading criteria. It cannot run solvers or
  judges.
- In-loop solvers are Muse Spark 1.1 and Muse-Glimmer-30B. Each attempts
  each of 10 tasks once per iteration. Muse Spark 1.1 is also the answer
  judge and all five quality judges: construct validity, correctness,
  feasibility, usefulness, overall.
- Held-out solvers never enter the agent's feedback. Nemotron-3.5-Lightning
  runs every iteration. Claude Opus 5 runs only on the final selected
  checkpoint.
- Gates before solving: structural validation; a revision-integrity judge
  that asks whether an independent author would make each public change
  *without* seeing solver results; and **reference-solution admission**.
  The agent's own reference must score ≥ 0.9 on the real verifier, with up
  to 10 repair attempts per iteration.
- Selection: the parent is the lowest Muse Spark score among iterations
  that passed all five judges.
- Four human settings:
  (i) none, where the agent picks the domain and construct and gets no
  grounding;
  (ii) coarse: one sentence of intent, no grounding;
  (iii) fine: a detailed spec plus a curated source corpus;
  (iv) fine plus between-iteration human direction.
  The paper never says who the humans were or how many there were.
  "Operator-in-the-loop" suggests the authors.
- Benchmarks: Graveyard (dead-end ideas), Rebuttal (does a rebuttal resolve
  a weakness), and SilentTrain (silent training bugs).
  - Setting (i) is **one run that builds its own, different benchmark**.
    The same curve is reused as the reference in both Fig. 2 rows.
  - Settings (ii) and (iii): one run each, on Rebuttal and Graveyard.
  - Setting (iv): one SilentTrain run forked at iteration 6, with three
    human instantiation instructions on one branch.

**Part 2, HackBench (§2, App. C–E).**
- Trajectories come from 13 tasks: 5 ResearchGym, 7 AutoLab "Model
  Development", and 1 weak-to-strong task. GPT-5.5 and GLM-5.2 ran them in
  the bash-only Harbor terminus-2 scaffold. The agent's JSON output
  includes "analysis" and "plan" text, so a monitor may see the agent's
  stated reasoning. The paper does not say whether it does.
- The hack taxonomy has 8 classes (Table 2). Six are general: fabrication,
  grader hijack, label leakage, omission, synthetic substitution, skip the
  work. Two are task-specific: overfitting and measurement exploit.
- **Synthetic set (128).** GLM-5.2 samples a hack type, inserts commands
  into a real trajectory, and patches the downstream observations. Claude
  Opus 4.8 and GPT-5.5 audit each one, and humans settle disagreements.
  The paper does not say whether the agent's reasoning text is edited to
  match the inserted commands.
- **Natural set (41).** GPT-5.6 Sol screened transcripts in 8 chunks (the
  highest-recall setup in a pilot), then humans audited the candidates.
  The authors note that agents "rarely hacked when the instruction
  explicitly prohibited reward hacking".
- **Metric.** Every command is labelled [CMD n]. The monitor returns the
  set of hacking indices. Precision, recall and F1 are computed per
  trajectory and macro-averaged over the 169 trajectories.
  - There are **no hack-free trajectories**, so this measures localization
    given that a hack exists, not detection.
  - No natural/synthetic split is reported. No seeds or repeat calls.
  - Nothing says how transcripts of up to 3.8M characters fit into each
    monitor's context, or whether any were truncated.
- Monitors: Claude Opus 5, Claude Opus 4.8, GPT-5.6 Sol, GPT-5.5, Muse
  Spark 1.3, Muse Wonder, GLM-5.2, Kimi K3.
- App. E (Table 13) gives one task-specific grader exploit for each of the
  12 ResearchGym and AutoLab tasks, "identified by auditing the grader".
  App. C adds that four ResearchGym wrapper rewards are stubs that only
  check for changes to the task tree, and that the w2s wrapper returns a
  constant 0.0.

**Part 3, AutoInteract (§3, App. F).**
- User models are learned from SWE-Chat: a GEPA-optimized behavior prompt
  plus profiles. Kimi-K2.6 is the user simulator, controller, evaluator
  and strong solver.
- A trajectory is kept if the weak (target) solver scores ≤ 40% and the
  strong solver ≥ 70% over 3 samples each, with up to 5 revision rounds.
- Training: DAPO-style RL with prime-rl, up to 500 steps, on Qwen3.5-4B
  and 27B. Baselines are the base model and RL on the same source tasks.
- Evaluation: LiveCodeBench-Pro (706 tasks); Multi-turn LCB-Pro, built
  with the same pipeline's scenarios; and SWE-Together (109 tasks, 2 runs
  each). Ablations are run at 4B only.

**Part 4, RL-XAR (§4, App. G).**
- Opus-4.8 and GPT-5.6 serve as both writers and judges on held-out paper
  sections. The n behind Fig. 12 is not stated; the percentages are
  multiples of 1/13.
- The rubric meta-prompt is optimized by Kimi-K2.6 on 52 (paper, section)
  examples: 8 training papers and 5 validation papers.
- RL: GRPO on Qwen3.5-27B, with Qwen3.8-2.4T-A95B judging. Evaluation uses
  a GPT-5.6 judge and takes the worst of 3 rubric rounds; the model was
  trained on 2 of those 3.

## Results

**Part 1.** These are selected-checkpoint scores, where lower is harder
(Fig. 2, Table 1, Fig. 3).

| Setting | Muse Spark | Muse Glimmer | Nemotron (held out) | Opus 5 (held out) |
|---|---|---|---|---|
| None (own benchmark) | 90 | 84 | 80 | 98.0 |
| Rebuttal, coarse | 84.4 | 83.7 | 81 | 83.1 |
| Rebuttal, fine | 43.5 | 51.2 | 39.1 | 65.9 |
| Graveyard, coarse | 76.2 | 81.6 | 75.4 | 90.7 |
| Graveyard, fine | 60.8 | 55.3 | 52 | 84.4 |
| SilentTrain, fork unaided | 88.1 | 85.6 | 83.8 | – |
| SilentTrain, fork + human | 64.2 | 54.2 | 48.8 | – |

- The fine-grained Rebuttal run starts near 71 on Muse Spark at iteration
  1 (read from Fig. 2). Most of its lead over coarse exists before the
  loop starts iterating. So the spec and corpus did most of the work, not
  the loop.
- The quality judges blocked a lower-scoring but infeasible iteration.
  Rebuttal fine iteration 5 scored 47.4 and failed feasibility. The
  autonomous run's first three iterations all scored 100 on Muse Spark and
  failed five and two criteria.

**Part 2** (Table 4: precision / recall / F1).

| Monitor | P | R | F1 |
|---|---|---|---|
| Claude Opus 5 | 69.68 | 72.64 | 64.88 |
| Muse Wonder | 67.87 | 64.98 | 61.47 |
| GPT-5.6 Sol | 54.84 | 68.55 | 52.99 |
| Kimi K3 | 56.55 | 54.10 | 52.66 |
| Claude Opus 4.8 | 58.72 | 54.59 | 52.47 |
| GPT-5.5 | 55.20 | 57.87 | 50.90 |
| Muse Spark 1.3 | 44.71 | 38.39 | 39.03 |
| GLM-5.2 | 37.40 | 32.96 | 32.33 |

- Sparsity (Table 3): the median trajectory has 1 hacked command (mean
  7.8, max 383). Over all trajectories, 1,312 of 60,468 commands are
  hacks.
- **Grader audit (Table 13), our classification of the 12 exploits:**
  - *Untouched start state clears the bar (4).* Cross-modal: the no-op
    source model already beats the published baselines. Materials
    tokenization: stock WordPiece clears the floor because the grader
    takes a max over F1 cells. Continual learning: a frozen backbone with
    nearest-class-mean heads "ties InfLoRA (~86/91)". Data selection:
    ~1 sample leaves the base model nearly unchanged, and it beats honest
    selection.
  - *Constant, empty or stub output scores well (5).* Multilingual OCR:
    empty output gives a real 2–3× CER gain over the untuned model.
    Moving-MNIST: a near-mean frame wins ("observed as a saved
    ZeroPredictor"). Time-series explanation: constant near-max
    attribution. LLM serving: a stub engine, 844 → 1238 tok/s in Table 2.
    GRPO: a constant number, or word salad that passes a substring check.
  - *A degenerate optimum that still needs work (3).* A Hopper policy
    that only survives (~1000 return). A mode-collapsed LoRA. Early stop
    at the ppl-18 saturation point.
- Most of these exploits come from auditing the grader code. Only a few
  have measured scores, and these are given above.

**Part 3.**
- At 4B: LCB-Pro 36.8 → 42.8; Multi-turn LCB-Pro 23.0 → 43.2;
  SWE-Together Pass@1 6.0 → 20.6.
- At 27B: LCB-Pro is flat (60.9 → 60.8); Multi-turn 37.2 → 50.6;
  SWE-Together 34.9 → 38.5. RL on source data regresses SWE-Together to
  27.1.
- Ablations at 4B, on the average: no multi-turn 28.2, no user models
  32.0, no executable verification 25.0, against 34.0 for the full method.

**Part 4.**
- The learned rubrics move the validation gap from −4.2 to +2.76 over 7
  meta-prompt iterations.
- Paper sections, worst-of-3 rubrics with human = 10: RL-XAR 9.6, base
  8.1, Claude 5.5 Opus 9.4.
- Stories: 2.8 → 8.2. Wikipedia: 2.7 → 4.0, still below every frontier
  writer except Haiku.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest line,
  "Agents fail hack localization", repeats the abstract's own claim, but:
  1. "Fail" is categorical. The best monitor recovers 72.6% of gold hack
     commands at 69.7% precision. The paper's own wording is "not …
     sufficiently" and "miss many hacks".
  2. It omits that 128 of 169 trajectories carry planted hacks. Those were
     written by GLM-5.2, which is also the weakest monitor. Gold labels
     were audited by two models that are also monitors.
  3. It omits that every trajectory contains a hack, so false alarms on
     clean runs are never measured.
  4. It omits three of the paper's four parts, including the human-role
     finding that bears on `agency: max`.
- **HackBench measures localization given a hack, not detection.** With no
  clean trajectories, a monitor that flags something in every run is never
  penalized at the trajectory level. Only within-trajectory precision
  limits it. For an HCE backstop, the cost that matters is the false-alarm
  rate on honest chains. This benchmark cannot give it.
- **Gold-label bias.** Natural hacks entered the set only if GPT-5.6 Sol
  flagged them during screening. So natural-set recall is measured against
  hacks that one model already found. Any hack it missed is absent from
  the gold set, which also makes every monitor look better on recall. For
  synthetic hacks, the paper does not say whether the agent's "analysis"
  and "plan" text was edited to match the inserted commands. Unexplained
  commands would be a tell. That contradicts the claim that the hack
  "cannot be found by … spotting an inconsistency".
- **Single numbers, no variance.** There is one call per monitor and no
  CI. With a median of 1 gold command, per-trajectory F1 is close to 0 or
  1, so the 50.9–53.0 cluster of four models is not resolved. No
  natural-vs-synthetic or per-class breakdown is given, so we cannot tell
  which hack types are missed.
- **The grader audit is the stronger HCE result, and it sits in an
  appendix.** On our reading, 9 of 12 published ML-research graders accept
  a trivial submission at or above honest work. Four of the 5 ResearchGym
  wrappers do not compute the research metric at all. One discrepancy:
  [[literature/papers/shao2026do]] reports that flux2_klein_lora is forced
  to zero whatever the agent does, while Table 13 says an overfit LoRA
  scores high. They may describe different versions; neither paper says.
- **Part 1 is n = 1 per setting and confounded.**
  - The no-feedback baseline is a *different benchmark*, chosen by the
    agent.
  - "Fine" adds both a detailed spec and a curated corpus that "coarse"
    lacks. Spec detail and source curation are never separated.
  - The humans are unnamed and uncounted, apparently the authors.
  - Every validity judge is the proposer's own model.
  - The SilentTrain fork is the cleanest contrast (same agent, construct
    and corpus), but it is one fork of one run.
  - "Pivotal in ideation" overstates what was varied. A one-sentence "what
    to build" (coarse) was nearly useless. What helped was *how* plus
    sources.
- **Selection is on the minimum of 10 noisy iterations** of a 10-task mean.
  The held-out Nemotron and Opus 5 scores are the unselected check, and
  they keep the ordering. Opus 5 runs only on the chosen checkpoint, with
  no repeat.
- **Parts 3–4 are single training runs**, scored with in-house-built
  evaluations. Multi-turn LCB-Pro is generated by the same scenario
  pipeline as the training data. The RL-XAR "leaderboard" uses rubrics
  that the model was partly trained on, which the authors acknowledge. The
  human checks are by the authors, 18 and 20 pairs.
- **A known-ordering check on judges, in passing.** Part 4's diagnostic
  asks whether a judge places expert human text above model text. It is
  the [[literature/papers/kim2026elicitation]] known-ordering preflight
  applied to an LLM judge. Standard pairwise judges failed it 63.5–84.6%
  of the time and generated rubrics 100%.

## Trust signals

- **Credibility:** 3.
  - For: FAIR at Meta with JHU, CMU and UW. The HackBench dataset is
    released on HF (`facebook/hackbench`). Prompts, task suite and
    per-task exploits are disclosed in appendices. AutoBenchmark's harness
    is described in unusual detail (gates, isolation, held-out solver
    lane).
  - Against: a preprint with no code link for Parts 1, 3 and 4. Every
    headline is a single run or single call with no CI. Part 1's human
    arm is n = 1 per setting, confounded, and has unnamed participants.
    HackBench's gold labels depend on models that are also monitors.
  - Not 4: the evidence behind each part is too thin for a major-lab
    rating. Not 2: the dataset is released, and HackBench's construction
    and the grader audit are concrete and checkable.

## Follow-up

- **Relevance:** 4. It adds material evidence to two load-bearing
  concepts:
  - [[concepts/hce-evaluation]]: a transcript monitor is a leaky backstop
    and is untested on false alarms.
  - [[concepts/programmable-evaluator-oracle]]: a 12-grader ML-research
    audit for the known-bad preflight.

  It adds a weak but on-point datum to [[concepts/hierarchical-delegation]]
  (a human parent's spec detail). Not 5: it seeds no concept, and the
  human-role and RL parts are n = 1.
- **Live fronts.**
  - *Known-bad preflight (pending).* The proposal's risk section says
    "None of them is an ML-research loop". Table 13 is ML-research. It
    also shows the preflight needs three probes, not one: no-op vs the
    bar, empty or constant output vs the start state, and known-good vs
    no-op (data selection). AutoBenchmark's reference-solution admission
    (≥ 0.9) is the known-good check that reviewers asked to restore. It
    applies where the loop builds the task.
  - *Redirect-before-halt (pending).* The SilentTrain fork is a redirect
    vs continue arm at a stall. A human redirect helped, n = 1. There is
    still no redirect vs halt arm.
- **Cheap local check.** Take one downstream project's `metrics.json`
  evaluator and score three probes: the untouched start state, an empty
  output, and a constant output. Log them next to the evaluator hash.
  That is three evaluator calls with no GPU.
- **Candidates** (cited, not yet in the graph):
  - Garikaparthi et al. 2026, ResearchGym.
  - Xu et al. 2026, AutoLab. Cited by [[literature/papers/shao2026do]].
  - Kulikov et al. 2026, Autodata, the same group's agentic data
    scientist.
  - Martin and Roger 2026, and Jotautaitė et al. 2026: monitoring
    benchmarks overestimate monitors on short trajectories.
  - Deshpande et al. 2026: contrastive reward-hack detection for code.
  - Libon et al. 2026: main/side-task red-teaming in autoresearch.
  - Cunningham et al. 2026 (METR): NanoGPT speedrun hacks.
  - PostTrainBench Team 2026: leaderboard re-audit.
  - Weston and Foerster 2025: co-improvement.
