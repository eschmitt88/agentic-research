---
kind: paper
title: "Contain, Conform, Compare (Extended Version): A Red-Teamed Evaluation Kit for Small Coding Agents and the Full Record of Its Experiments"
authors: ["Nguyen Minh Thang"]
institutions: ["VNU University of Science, Hanoi"]
year: 2026
venue: "arXiv 2611.16485 (cs.AI), v1 2026-11-12, preprint; extended version of a Gemma 4 Developer Agent paper-track (Kaggle) writeup"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.16485"
code_url: "https://github.com/MinhThanghus/drydock-eval"
citations: null
source: "raw/papers/thang2026contain.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
tags: ["local-evaluator", "scorer-fidelity", "rendering-trap", "token-id-parity", "container-sandbox", "canary", "red-team", "screen-then-confirm", "frozen-pass-rule", "pre-registered", "wild-cluster-bootstrap", "repository-clusters", "run-to-run-noise", "baseline-rerun", "selection-effect", "winners-curse", "holdout-spending", "grading-controls", "null-result", "gemma-4", "small-model", "swe-tasks", "kaggle", "ai-built"]
---

# Contain, Conform, Compare (Extended Version): A Red-Teamed Evaluation Kit for Small Coding Agents and the Full Record of Its Experiments

## TL;DR

This is a local evaluator and lab notebook for one Kaggle competition, in
which a Gemma 4 agent bundle is scored on hidden GitHub-issue tasks with
one noisy draw per day. The kit, `drydock-eval`, has three parts:

- **contain**: a per-job container runner, attacked by AI verifiers across
  seven versions;
- **conform**: eight checks that compare a local stack's requests and
  rendered prompts with the scorer's configuration, down to token ids;
- **compare**: paired tests with repositories as clusters, and a frozen
  pass rule whose size and power were simulated before use.

The extended version releases the full run record and reports mostly
nulls. Three results matter here:

1. **Screen-then-confirm, done properly, confirmed nothing.** Three E4B
   candidates led 96-run screens by +6.2, +1.0 and +5.2 points. Each was
   tested once on new tasks under a frozen rule. Gains were −0.35, −0.39
   and +4.10, and none passed. Every screen lead is within what the best
   of 2–5 *identical* arms would show (post hoc).
2. **The noise floor was measured by re-running the unchanged baseline.**
   It scored 103 and 97 of 512 on the same 128 tasks (1.17 points). The
   SD of a two-arm difference is 1.44 points, but that is run-to-run noise
   only. The third candidate's +4.10 is 2.9 run-noise SDs, yet one-sided
   p = 0.0405 when repositories are resampled. Fixed-task noise and
   cross-repository generalization are different yardsticks.
3. **The confirmation pool, not compute, ran out.** 479 of 579 validated
   tasks are used. One panel was used for two confirmations. The author's
   conclusion is that the number of confirmations, not screens, limits
   what can be learned.

## Claims

- A local evaluator must answer safety, fidelity and inference before its
  numbers mean anything. Solve counts do not reveal fidelity breaks. A
  wrong tool-result role broke token parity on 114 of 114 calls, while
  solve counts stayed within noise (3 and 5 of 24 vs 4 and 4, Fisher
  p ≈ 0.75).
- Passing scripted escape tests proves little. No attack escaped the
  container's namespaces or privileges, but nine high-severity vectors came
  through what the host shared with it, in designs that had passed their
  own canaries.
- Four documented "rendering traps" changed E4B's solve rate by less than a
  powered, pre-registered experiment could detect.
- A hosted single draw (SD ≈ 1.56 tasks of 58) checks that a file runs and
  cannot rank two files (about 10 draws per file for a two-task gap).

## Methods

- **Setting.** Gemma 4 Developer Agent competition: a google-adk bundle,
  served by vLLM on 4×L4, scored on hidden Python-repo issues (58 public
  tasks, one submission per day). Models: Gemma 4 31B and E4B (4-bit
  QAT). Most E4B runs are on a shared RTX 5090 workstation (8 jobs, about
  250 runs/h); 31B used emulated 4×L4 latency and Kaggle notebooks.
- **Tasks.** 70 competition dev tasks (opaque IDs) and 579 Berth tasks
  that load and validate. Disjoint sets were drawn from the Berth tasks:
  - screen set S: 48 tasks;
  - small panel C: 72 tasks, 18 repos;
  - large panel: 200 tasks, 20 repos, containing C; H = its 128 tasks
    outside C;
  - case-study panel: 231 tasks, 15 repos.

  Panels were frozen before any outcome on them existed. S is not fresh:
  32 of its 48 tasks had been run before.
- **Grading controls.** Before any agent ran, every task was graded with
  its reference patch (must pass) and with an empty patch (must fail). One
  task whose empty patch passed was removed. 2,494 control rows in total.
  1,016 task × control pairs were graded on more than one machine or
  runner, and 0 changed.
- **contain.** A fresh container per job (no network or capabilities,
  seccomp, read-only root, resource caps). The only channel out is a
  per-job socket to a forwarder that passes allow-listed chat fields.
  Runs start only if eight recorded hashes (image, code, flags, model
  files) match.
- **conform.** Eight checks (role, sampling, tools, shape, template,
  thoughts, newline, compaction). The template check compares token ids,
  because identical strings can tokenize differently. Mutation suite: 103
  mutations × 4 scenarios.
- **compare (the pass rule).** Candidate and baseline run on the same
  tasks and repeats. All five conditions must hold:
  1. mean gain ≥ 2/58 (3.45 pts);
  2. one-sided p ≤ 0.025 under a null-imposed wild cluster bootstrap-t over
     repositories (Rademacher, 9,999 draws);
  3. at least three repositories gain, and every leave-one-repo-out
     estimate is positive;
  4. agent time ≤ 1.2× baseline, and infrastructure errors do not rise;
  5. no repository loses ≥ 2 task-equivalents.

  Two independent re-implementations matched the rule code on 280/280 and
  193/194 datasets.
- **Case study (Sec. 9, pre-registered, protocol sha256 recorded).** Five
  arms on 231 tasks × 4 repeats (4,620 jobs): the scorer path F and four
  divergences:
  - R: tool-result role `tool_responses`;
  - C: compaction unset;
  - H: history thoughts removed;
  - T: the Hugging Face template.

  The primary test is an exact task-level sign-flip, Holm over four arms.
  A frozen checker re-rendered every returned request (75,606) and
  confirmed each run's arm. Outcomes stayed blind until a fixed cut-off.
- **Three candidate programmes (Sec. 10).** Each followed the same steps:
  1. build variants;
  2. pilot on 15–16 tasks (programmes 2 and 3);
  3. screen on S, 2 repeats, against the baseline in one queue, and take
     the leader;
  4. register the candidate, tasks, rule and script, then confirm once on
     a frozen panel disjoint from S.

  Each programme took 1,152–1,402 runs and 3–5 workstation hours.
  Candidates: v-p2 (prompt), v-t6 (file-editing path), v-a3 (added
  sub-agent step). Their text is withheld until 2026-12-03.
- **Who did the work.** AI builders wrote the code, attacks and analyses;
  separate AI verifiers re-ran them adversarially; a lead AI session froze
  protocols; a human owner approved machine time, submissions and release.
  "Verified" means this process, not human review. The models are unnamed.

## Results

- **Serving (4×L4).** One stream decodes 38.1 / 76.1 / 105.8 tok/s
  (31B / 12B / E4B). A LoRA adapter costs 9–13% of decode speed.
- **contain (Table 3).** Canaries passed 52/52 → 200/200 over versions
  1–7. High-severity vectors were 4, 4, 1, then 0. They included:
  - a host-side install that ran the agent's `setup.py`;
  - a symlink that leaked a host file;
  - 10 GiB of host disk reserved in 0.005 s;
  - a model server that fetched URLs named in requests.

  Caveat: the released tree is an identifier-substituted copy that was
  never run in a container as released.
- **conform (Table 4).** Checks caught 317 of 325 drift runs; the 8
  misses were same-length template changes seen by count, all caught by
  token id. Kaggle-side evidence is count-level only (181 of 189 calls).
- **Case study (Table 18/23).** F solved 23.8% (220/924). No arm differed
  (smallest Holm p 0.44):

  | Arm | Change, pts | 95% interval |
  |---|---|---|
  | R | −0.3 | [−2.8, +2.2] |
  | C | +0.2 | [−2.0, +2.5] |
  | H | −2.1 | [−4.4, +0.4] |
  | T | +0.8 | [−1.4, +2.9] |

  - Simultaneous 98.75% intervals allow at most −5.1 (H) and +3.5 (T).
  - The divergences changed 85–90% of requests in R, H and T, and 21% in C.
  - 135 of 231 tasks were solved in no run of any arm and 13 in every run,
    so 83 tasks carry every difference.
  - H runs took 137 s against 78 s and hit the 5-minute limit in 11.5% of
    runs against 1.8%. With limit rows treated as missing, H is −1.25
    (p = 0.34).
- **31B configurations (Table 7, one run per task).** On the 70 dev tasks,
  BF5 solved 37, W1/W2 39, W3 38, SH1/SH3 41, SH5 41 and SH6 44.
  - The largest difference against BF5 is SH6: 10 tasks only it solved, 3
    only BF5 (p = 0.09, uncorrected for 5 comparisons). SH6 was developed
    while looking at these same tasks.
  - The first-30 leads (W1 20 vs BF5 17; SH1 20 vs W3 16) did not
    reappear on the next 40.
  - Union of six configurations: 50 tasks, against 44 for the best one.
    The author calls this inflated by run noise.
- **Baseline repeatability (Table 15).** E4B baseline, same tasks:
  - S: 19, 21 and 20 of 96 (range 2.1 pts).
  - H: 103 and 97 of 512 (1.17 pts), per-task r = 0.92. 83 of 128 tasks
    were solved in 0 of 8 runs and 10 in 8 of 8.
  - From within-task variation, run-to-run SD is 1.01 pts for one arm and
    1.44 for a two-arm difference on H.
  - The independent-binomial formula gives 1.75 per arm. Within-task
    variation is 34% of that variance.
  - The SD of a screen difference on S is 2.9 pts.
- **Repositories (Table 16).** The baseline solved 323 of 1,600 runs
  (20.2%). By repository: 7–34% among repos with 10–24 tasks, 53% in the
  best, 0 in three small ones.
- **Machines.** Eight local runs of one 24-task list solved 2–5, against
  4 and 4 on Kaggle (each interval > 33 pts wide). Solve-level parity is
  "not refuted, not established".
- **Screens (Table 29).** In screen 1, all five variants beat the baseline
  (22–25 vs 19). The shared baseline drew its lowest of three sessions,
  which lifts every lead at once. Pilots ranked nothing: v-t4 had the best
  pilot (7/15) and then trailed in its screen. All three leaders used more
  agent time (75 vs 64, 89 vs 72, 77 vs 66 s).
- **Confirmations (Table 30).** No candidate passed.

  | Candidate | Panel | Solved, base / cand. of 512 or 288 | Gain, pts | Clustered one-sided p |
  |---|---|---|---|---|
  | v-p2 | C (72 tasks, 18 repos) | 63 / 62 | −0.35 | 0.60 |
  | v-t6 | H (128 tasks, 19 repos) | 103 / 101 | −0.39 | 0.60 |
  | v-a3 | H, second use | 97 / 118 | +4.10 | 0.0405 |

  v-a3 met the size, ≥3-repos, leave-one-out and time conditions and
  failed only p ≤ 0.025. By task it was ahead on 24, behind on 11 and
  level on 93. The three largest-gaining repositories hold 27 tasks and
  +4.25 of its +5.25 task-equivalents. Its pre-registered power was about
  0.5. v-a3 was built after the v-t6 result on H was known, and two
  mechanical reads of earlier runs fed its repair. Both are disclosed in
  its registration.
- **Simulated rule behaviour (Table 5).** The false-pass rate depends on
  the null:
  - no effect, exchangeable tasks: 0.23–1.90% (small panel), 0–0.50%
    (large);
  - no mean effect, but the candidate breaks 20% of repos: 7.5% / 8.4%;
  - worst tasks dropped after seeing outcomes: 31.75% / 40.75%.

  Power: an even 3/58 gain passes about 60% / 90% of the time. A gain
  confined to four repositories passes 15.9% / 24.9%.
- **Selection (Sec. 10.3, post hoc).** The best of five equal arms leads
  the baseline by 2.4 pts on average, and the best of two by 1.2. P(lead ≥
  observed | all equal) is 0.09, 0.59 and 0.10. Ranking variants would
  need the difference SD to fall from 2.9 to about 1 pt, which takes about
  830 runs per arm.
- **Hosted vs local (Table 11, Sec. 11.3).** Hosted scores were 0.22–0.34
  of the local solved share. Both arm pairs with both numbers reversed
  order between local and hosted (W3/BF5 tied locally, 0.12 vs 0.18
  hosted; v-a3 +4.10 locally, 0.05 vs 0.06 hosted). Neither hosted
  difference is distinguishable from noise.
- **Failure shape.** 93% of unsolved E4B runs handed in a wrong patch;
  0.3% hit the time limit (31B: 12.6%). All 104 decision quantities
  recompute from the released rows.

## Critique / open questions

- **Against the digest: slightly above.** Most numbers are right. Three
  phrasings claim more than the paper does.
  1. "9,660 agent runs": these are *rows*, 5,024 runs plus 4,636
     case-study rows, 16 of which are interrupted-job infrastructure rows.
     9,644 runs were analysed.
  2. "Matching a hosted competition scorer down to rendered tokens": token
     ids match the *public Kaggle template* on local requests. Kaggle-side
     evidence is count-level on 189 calls, the hosted google-adk release is
     inferred, and solve-level parity is explicitly "not established".
     Local solve rates are 3–5× the hosted ones.
  3. "Each retested once on new tasks": the tasks were new to each
     candidate, but the third confirmation was the panel's second use, and
     the candidate was built after the first result there.
- **What this is not.** It is not an autonomous research loop. Humans and
  a lead AI session chose the screens, built the variants and set the
  rules. No candidate was ever confirmed, so the paper shows what a
  protocol rejects, not that it keeps real gains. Pre-registered power was
  about 0.5 for the third confirmation, and the simulated power against
  repository-concentrated gains is 16–25%. A null here is weak evidence of
  no effect, and the author says so.
- **One baseline, one model, one task source.** The noise numbers
  (1.44-point difference SD, 65% never-solved) are for one E4B bundle on
  Berth's 22 repositories. A stronger model has fewer never-solved tasks
  and probably more run-to-run noise per task. The share of uncertain
  tasks is what sets the run SD, and it will not transfer.
- **The interesting candidate is undecided.** v-a3's +4.10 is 2.9 SDs
  against run noise, met every repository condition but one, and its
  hosted draw is uninformative. The paper correctly does not call it an
  improvement. Neither a fresh-panel test nor a selection-adjusted
  estimate exists.
- **The selection analysis is a planning guide, not a test.** It uses a
  normal approximation, a noise estimate from six baseline runs per task,
  and assumes variants share the baseline's noise. The leaders used
  10–24% more agent time, so "equal noise" is itself untested.
- **AI built it, AI verified it.** "Verified" means AI verifiers, not
  humans. Prompts and bundles are withheld until 2026-12-03. Arms cannot
  be rebuilt, and the candidates' changes are described only by category.
- **Execution blemishes, all disclosed.** The case study ran under a
  second GPU workload (6.0% of runs over 240 s, against a 5% threshold)
  and stopped three times (open-file limit, external SIGTERM, OOM). Rows
  of two runs are missing. Some 31B arms ran in their own blocks under
  different load, and some task sets were picked with results in view.
- **Fit with the graph.**
  - [[literature/papers/hu2026winners]] and
    [[literature/papers/zhou2026does]] computed screen-lead inflation in
    simulation. This is a real three-programme instance, including a
    shared baseline drawn low (zhou's baseline-was-the-worst-run case).
  - It is the cleanest case in the graph of the 10-06
    [[concepts/hce-evaluation]] rule ("the decision-grade test is a
    selection step too"): a separate screen set, then one untouched
    confirmation.
  - The contain record is a second case, after
    [[literature/papers/hickey2026saltbench]], of probes that pass while
    the boundary leaks through a layer they do not exercise.
  - The 50-vs-44 union gap is the oracle-gap-under-noise point that
    [[concepts/pass-at-k]] already holds from
    [[literature/papers/shi2026hard]].

## Trust signals

- **Credibility:** 3.
  - For: code (Apache-2.0, 435 tests) and every run-level row released,
    with scripts that recompute every table and stop on mismatch. Protocol
    hashed before data, size and power simulated before use, outcomes
    blind to a cut-off. Nulls reported as nulls, post hoc analyses
    labelled, missing rows disclosed. Grading controls on every task.
  - Against: single author, one university, not peer reviewed (competition
    paper track), 5 days old, no citations. Done and verified by unnamed
    AI agents, with no human review of the verification. Bundles and
    prompts withheld. One small model in one competition.
  - Not 4: no major lab or venue, and no outside reproduction. Not 2: the
    artifact and reporting discipline is unusually complete, and the
    headline numbers can be recomputed from released rows.

## Follow-up

- **Relevance:** 4. It is the first fully reported real-agent case in this
  graph of screen → frozen-rule confirmation, with the noise floor
  measured by re-running the unchanged baseline. That is material new
  evidence for [[concepts/hce-evaluation]] (confirmation slots are the
  spent resource; the resampling unit decides the verdict) and for
  [[concepts/budget-as-ceiling]] (the size of the `/iterate` band against
  the comparison it gates). Not 5: a Kaggle coding agent, not an
  ML-research agent; no autonomous loop; one small model; and no
  candidate passed, so the protocol's power on real gains is not shown.
- **What it bears on for `/iterate` (our inference).** The band decides
  when a chain stops, not what is kept (established 11-16 from
  [[literature/papers/gao2026trustworthy]]). Three of the paper's numbers
  touch that stop counter.
  - **Per-seed std vs a difference of means.** The band is a per-seed std
    ("larger of the two std values"), and the counter compares the
    difference of two N-seed means against it. Here the difference of two
    single draws had √2 times the per-arm run SD (1.44 vs 1.01), and
    pairing on the same tasks did not shrink it, because the noise is
    within-task. So a null cycle clears the band with probability about
    P(Z > √(N/2)): 16% at N = 2, 11% at N = 3 (our arithmetic, normal
    approximation, against a fixed incumbent).
  - **Run noise vs task noise.** A seed std measures run-to-run noise on
    fixed items. The clustered SE implied for v-a3 (about 2.1 pts, our
    arithmetic from bootstrap-t 1.954) is about 1.5× the run SD of the
    difference (1.44). This supports the 10-26 suggestion that the band
    be max(seed std, paired item SE).
  - **Binomial overstates run noise.** The binomial SE is 1.7× the true
    run SD, because 65% of tasks never move. A band computed as a
    binomial SE is not a seed band.

  No `/iterate` keep rule exists, so the screen-lead discount is a
  keep-side matter that the box does not yet have.
- **For the HCE final pass (our inference).** The paper's confirm step is
  the box's final-scoring pass with a decision rule attached. The
  analogue of its exhausted pool is `evaluation.md` clause 2's "a test
  score, once read, is spent". The paper's registration of a reused panel
  ("second use of that task set", with the candidate built after the
  first result) is the disclosure clause 2 asks for, done in advance
  rather than at reporting.
- **Candidates** (cited, not yet in the graph):
  - Miller 2024, "Adding error bars to evals" (arXiv 2411.00640):
    clustered and paired errors and power analysis for evals.
  - Heineman et al. 2025, "Signal and noise" (NeurIPS 2025, arXiv
    2508.13144): benchmark SNR.
  - Mobassir 2026, "Too small to see" (Kaggle paper track): resolution
    limits on Gemma 4 31B tuning.
  - Marchand et al. 2026, "Quantifying frontier LLM capabilities for
    container sandbox escape" (ICML 2026, SandboxEscapeBench).
  - Zhu et al. 2025, "Establishing best practices for building rigorous
    agentic benchmarks" (NeurIPS 2025, arXiv 2507.02825).
