---
kind: paper
title: "Auto-Research Arena: Benchmarking and Advancing Auto-Research in Language Modeling"
authors: ["Xi Chen", "Chongjian Ge", "Yiran Xu", "Yiqun Mei", "Henrik I. Christensen", "Hao Tan", "Ziwen Chen", "Hanwen Jiang"]
institutions: ["University of California San Diego", "Adobe Research"]
year: 2026
venue: "arXiv 2610.35866 (cs.LG), v2 2026-10-27, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.35866"
code_url: "https://github.com/Auto-Research-Arena/auto-research-arena"
citations: null
source: "raw/papers/chen2026autoresearch.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["benchmark", "auto-research", "karpathy-autoresearch", "lm-pretraining", "admissibility", "constrained-optimization", "multi-objective", "sequential-search", "beam-search", "harness-comparison", "elo", "charged-experiments", "idea-library", "cross-target-transfer", "budget-allocation", "one-run-per-cell"]
---

# Auto-Research Arena: Benchmarking and Advancing Auto-Research in Language Modeling

## TL;DR

Arena takes Karpathy's autoresearch testbed (a ~50M-parameter LM
pretraining program, 10-minute training runs on one A100) and turns it into
nine constrained-optimization tracks. Each track minimizes one resource
(parameters, tokens, updates, training time, FLOPs/token, peak memory,
KV-cache, decode latency, request latency). A candidate is *admissible*
only if validation BPB < 1.05 and the other resource limits hold.
Admissibility is a gate; the target is the score.

Six LLM harnesses (AutoScientists, Arbor, GEAR, Heuresis, and the
authors' own Sequential Search and Beam Search) run under Claude Opus 5 and
GPT-6 Astra, plus TPE under Opus 5 only. Each search gets 100 charged
launches. Results are aggregated by a margin-aware Elo over the nine
tracks.

Three results matter for this graph:

1. **The authors' simple Sequential Search ranks second under both
   models.** Opus 5: AutoScientists 1729.4, Sequential 1687.6, Arbor
   1632.4, GEAR 1622.4, Beam 1438.6, Heuresis 1350.4, TPE 1039.2. GPT-6:
   GEAR 1835.3, Sequential 1702.4, Arbor 1459.6, Beam 1423.3,
   AutoScientists 1298.7, Heuresis 1280.6. Every cell is **one search**
   (seed 42). No repeats, no intervals.
2. **Budget is metered by the evaluator, not the harness.** A launch is
   charged when GPU work starts, including crashes and OOMs. The harness
   reads Arena's remaining budget. Arena independently records the best
   admissible candidate, whatever the harness keeps.
3. **An idea library from the nine searches seeds a 1-hour quality
   search** that lowers training loss at matched data (2.675 → 2.544) and
   matched FLOPs (2.617 → 2.544), and held-out BPB 0.9295 → 0.9112. That
   BPB is the best of 305 single-read candidates on the same evaluator the
   search used. The reference got no search.

## Claims

- Separating admissibility (quality + resource limits) from improvement
  (one target) lets one environment host many research objectives, and
  excludes gains that break a constraint.
- Arena is discriminative: harnesses differ across objectives, and no
  single harness dominates.
- "More elaborate orchestration does not yet consistently translate into
  stronger performance", since Sequential Search ranks second with both
  models.
- TPE, tuned for the original loss-minimization setting, ranks last on
  the broader benchmark, so specialization does not transfer.
- Multi-target search widens the idea frontier: 32 distinct findings, 20
  accepted during search, in nine idea families. Many mirror current LLM
  research (looped Transformers with depth-specific residual gains,
  cross-layer KV sharing, Adam-mini-style second moments).
- Discoveries are reusable: an idea library improves quality at a longer
  horizon, some 10-minute programs keep their advantage for 4–5× the
  horizon, and programs can seed searches on other targets.
- Constraints shape the reachable design space; evaluation depth trades
  against search breadth.

## Methods

- **Testbed.** Karpathy's autoresearch LM pretraining program. Fixed
  ~631.2M-token ClimbMix pool. Each candidate runs on one A100 80GB, seed
  42, up to 600 s of training (compile, validation and first ten warmup
  updates excluded). Reference: 50.33M params, 239.08 MFLOPs/token,
  43.957 GiB peak memory.
- **Formulation (Eq. 1).** Minimize Target_j(p) subject to BPB(p) < τ =
  1.05 and Resources_j(p) ≤ b_j. τ is "slightly above the reference
  program's BPB". Per-track limits are in Appendix A (e.g. FLOPs ≤ 239.1
  M/token, params ≤ 50.3M, memory ≤ 44.0 GiB). Inference tracks add a
  total-variation correctness check (≤ 0.05 against the full forward pass
  at all 513 positions) and a latency or state-memory bound.
- **Time-to-quality targets** (tokens, updates, time) use the first
  30-second validation checkpoint with BPB < 1.05. Times are rounded to
  30 s, so ties are common.
- **Budget.** 100 charged launches per search, the reference included, so
  ≤ 99 candidates. Charging starts at a GPU-work marker. Failures, OOMs
  and abandoned runs are charged. Sequential uses one worker; Beam uses
  four (8 candidates per generation, ≤ 4 at once).
- **Baselines (Appendix B).** Both run a persistent controller with an
  ideator, an executor and a reviewer, in fresh sessions per round with
  history supplied from files. Sequential keeps one incumbent and accepts
  a strict target improvement. Ties are broken on a task metric, then BPB.
  It has an allowance of four "unchanged-target" quality attempts, reset
  only after a strict target gain. Beam keeps up to four programs, two
  children each per generation.
- **Other harnesses.** AutoScientists (multi-agent), Arbor
  ([[literature/papers/jin2026toward]], hypothesis tree), GEAR
  (evolutionary), Heuresis (diversity-preserving archive). They are
  integrated through a minimal interface; the paper does not report how
  faithful each integration is or who tuned it. Both models run at medium
  reasoning effort.
- **Scoring (Appendix D).** Best admissible value per objective. Pairwise
  log-ratio outcomes, margin multiplier capped at 3, K = 32, equal
  objective weights. Averaged over 4,000 random comparison orders (seed
  20260902). Elo is within-model.
- **Follow-up studies.** A BPB threshold sweep (1.02 / 1.05 / 2.0, 30
  completed candidates per search). A training-time-per-candidate sweep
  under a 20 GPU-hour budget (Beam, BPB < 2.0). Three cross-target
  continuations. A transfer test (selected programs retrained 60 min). A
  quality-driven search over an idea library (10B-token pool, ≤ 1 h or
  2.217B tokens per candidate, six rounds).
- **Not reported:** LLM tokens or dollar cost per harness, wall-clock per
  search, realized GPU-hours per harness, BPB noise across seeds, and the
  harness and model used for the quality-driven search.

## Results

- **Table 2 per-objective winners.** Opus 5: AutoScientists wins 4 of 9
  (updates 267, FLOPs 52.15, decode 49.70 ms, request 29.46 ms).
  Sequential wins 3 (tokens 152.46M, time 210 s, KV cache 1.000 KiB).
  GEAR wins params (13.20M), Arbor wins memory (0.373 GiB). GPT-6: GEAR
  wins 7 of 9; Sequential wins params (15.28M) and memory (0.646 GiB).
- **Charged experiments used (mean over objectives).** Opus 5:
  AutoScientists 84.7, Sequential 87.7, Arbor 54.0, GEAR 100, Beam 96.9,
  Heuresis 100, TPE 100. GPT-6: AutoScientists **30.2**, Sequential 100,
  Arbor **14.7**, GEAR 100, Beam 95.8, Heuresis 86.8.
- **Who loses to Sequential Search.** Opus 5: Arbor (−55 Elo), GEAR (−65),
  Beam (−249), Heuresis (−337), TPE (−648). GPT-6: Arbor (−243), Beam
  (−279), AutoScientists (−404), Heuresis (−422). Under GPT-6, the two
  largest "losses" to Sequential are harnesses that used 15–30% of the
  budget Sequential used.
- **Search trajectories (Fig. 3).** On the Opus training-time track,
  GEAR, Arbor, Beam and Heuresis all end at 270 s but first reach it at
  12, 39, 50 and 99 charged experiments.
- **Threshold sweep (Table 5).** Relaxing BPB from 1.05 to 2.0 lets
  Sequential cut params from 18.94M to 0.69M and memory from 0.83 to
  0.035 GiB. Winners sit at the gate: BPB 1.0497 under < 1.05, 1.9963
  under < 2.0, and 1.0195 / 1.0185 under < 1.02.
- **Budget allocation (Table 8).** At 20 GPU-hours with 1,200 / 2,400 /
  4,800 / 6,000 s per candidate (60 / 30 / 15 / 12 candidates), the
  smallest admissible models are 123K / 3.19M / 11.54M / 7.86M params at
  BPB 1.986 / 1.202 / 1.061 / 1.152. One search per setting. The
  authors say this shows neither an optimal allocation nor a causal effect.
- **Cross-target (Table 6).** Params → FLOPs: params held at 14.09M,
  FLOPs −33.7%. FLOPs → decode latency: −87.5%. Params → training time:
  time −21.1%, params +24.6%. One search each, no restart-from-reference
  control (the authors say so).
- **Transfer beyond the horizon (Table 4, Fig. 4).** A GEAR token program
  reaches 0.9593 BPB at 30 min with 1.004B tokens; the reference needs
  ~47.6 min and 1.771B (43.3% fewer tokens). Its lead lasts to ~51 min. At
  60 min the reference is better (0.9295 vs 0.9478). An AutoScientists
  time program saves 34.5%. The six programs were "selected"; the
  selection rule is not given.
- **Idea-library quality search (Figs. 1, 5, 6; Tables 9, 10).** Training
  loss −4.9% at matched data and −2.8% at matched FLOPs. BPB 0.9295 →
  0.9112 at 50.33M params. 0/56 round-1 candidates beat the reference,
  then 44/56, 29/56, 50/56, 55/56, 25/25 (203/305 overall). The top two
  differ by 8 × 10⁻⁶ BPB. The authors say the single measurement cannot
  separate them. Ideas came from four harnesses and four objective
  classes. Ideas did not combine additively: one reuse step (0.934888)
  was worse than the reference.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every digest
  sentence is in the paper. What it leaves out changes how far the
  headline can be read.
  1. **"Several sophisticated harnesses lose to a simple
     sequential-search baseline."** True at the Elo level, but Sequential
     is second under both models. It wins 3 of 9 objectives under Opus 5
     and 2 of 9 under GPT-6. Every number is one search at seed 42 with no
     interval. The Opus 5 gaps to Arbor and GEAR (55 and 65 Elo) are the
     kind a rerun could flip.
  2. **Compute is not matched in practice.** The budget is a launch
     *cap*, not a launch count used. Under GPT-6, Arbor used 14.7 and
     AutoScientists 30.2 of 100 launches, against Sequential's 100. LLM
     tokens, dollars and wall-clock are not reported for any harness. Beam
     runs four GPUs at once. So "simple beats sophisticated" is partly
     "a harness that spends its budget beats one that stops early". The
     cleanest losses are under Opus 5, where GEAR and Heuresis used all
     100 launches and still ranked below Sequential.
  3. **Sequential Search is the authors' own baseline.** It has a
     reviewer, a tuned tie-break rule and a quality-attempt allowance. The
     paper does not report how much tuning went into it, or into the
     integrations of the other four harnesses.
  4. **"Lower held-out BPB" from reused discoveries.** 0.9112 is the best
     of 305 single reads of the same evaluator the quality search was
     shown. There is no untouched test split. The reference had one 1-hour
     run and no search at equal GPU-hours (roughly 305 or more candidate
     hours). The authors do not over-read the top candidate, but the
     1.97% gain itself carries selection inflation of unknown size.
- **How held-out BPB is protected.** Only structurally. The evaluator
  code and protected files are checked by Arena, and harnesses cannot
  edit them. But the same held-out BPB is the admissibility gate every
  harness sees after each launch. It is a validation set the search
  selects on, not a sealed test. Nothing is re-scored on fresh data or
  seeds.
- **The admissibility gate is read once at its edge.** Six of the 12
  best admissible programs in Table 5 sit within 0.004 of the BPB
  threshold. Sequential's parameter winners sit 0.0005, 0.0003 and 0.0037
  below it (1.0195, 1.0497, 1.9963). Seed-to-seed BPB noise at 10 minutes is not reported. A
  winner at 1.0497 may not be admissible on a second seed. A binding
  constraint will push winners to the edge anyway, so this is expected.
  But it means the gate selects on noise as well as on quality.
- **An extreme result goes unaudited.** Opus 5 Sequential's KV-cache
  allocation is 1.000 KiB against the reference's 33,790 KiB. The
  correctness check (TV ≤ 0.05) and latency bound presumably held, but the
  paper does not describe the mechanism or audit it. A 4.5-order cut is
  the kind of result a known-bad / known-good preflight is for.
- **TPE is a straw baseline on most tracks.** It searches a fixed
  hyperparameter space, so it cannot change the architecture or the
  inference code that the KV-cache, latency and parameter tracks need.
  "Specialization does not transfer" is true by construction there.
- **Not shown:** variance across search seeds; any compute-matched
  comparison in tokens or dollars; whether harness rankings hold under a
  second model at the same reasoning effort (the two models already
  disagree on AutoScientists: first under Opus 5, fifth under GPT-6); a
  causal effect for cross-target transfer or the idea library.
- **What is good.** The admissibility/target split is clean and reusable.
  Evaluator-side budget metering, with failures charged and the harness
  reading the remaining budget from Arena, removes self-accounting. Arena
  records the best admissible result independently of what the harness
  keeps. The appendix states its own limits (Tables 8 and 6, C.3)
  honestly.

## Trust signals

- **Credibility:** 3.
  - For: UCSD and Adobe Research. Code released. A full protocol appendix
    (metric definitions, charging rule, baseline selection policies, Elo
    formula with seeds). Per-objective raw values next to Elo. Charged
    experiments per harness are disclosed, which is what exposes the
    GPT-6 budget gap. The appendix flags its own non-causal studies.
  - Against: preprint, a week old, no citations. One search per
    harness × target × model, no intervals. No token, dollar or
    wall-clock accounting. The headline baseline is the authors' own. The
    quality-search BPB is selected on the reported metric, and its
    harness and model are not named.
  - Not 4 because the ranking claims rest on single runs with
    unequal realized budgets. Not 2 because the benchmark, scoring and
    accounting are well specified, and the code is out.

## Follow-up

- **Relevance:** 4. This is the closest benchmark in the graph to this
  box's `/iterate` loop: Karpathy's autoresearch, short GPU training runs,
  a fixed launch budget, keep-if-better. It adds material evidence to:
  - [[concepts/hce-evaluation]]: an admissibility gate separate from the
    target, evaluator-side best-admissible recording, and a gate read
    once at its edge on an unsealed validation set.
  - [[concepts/budget-as-ceiling]]: evaluator-metered launch budget with
    failures charged; a launch cap is not matched spend.
  - [[concepts/evolutionary-expansion]]: a one-incumbent loop ranks
    second against evolutionary, tree and archive harnesses, with
    caveats.
  Not 5 because every ranking is one run per cell with unequal realized
  budgets.
- **Bearing on live fronts.**
  - Known-bad preflight (2026-10-25 proposal): Arena's admissibility
    check is the "is this candidate even valid" gate, and the 1.000 KiB
    KV result is an unaudited outlier a preflight would probe. A
    known-good check (re-run the reference through the evaluator and
    confirm it scores as admissible) is cheap here.
  - Noise band (2026-09-27): Sequential Search accepts on bare strict
    improvement with no band. Winners at BPB 1.0497 under a 1.05 gate are
    the case a band, or a second-seed confirmation of admission, would
    catch.
  - Redirect-before-halt (2026-10-18): no redirect arm. Sequential stops
    only on budget exhaustion. Fig. 3's plateaus (TPE early, GEAR and
    AutoScientists still improving late) are descriptive, not a test.
- **Cheap local test.** Run Sequential and Beam on two Arena tracks at
  three search seeds each, and log LLM tokens and GPU-hours. That tells us
  whether the Opus 5 Sequential-vs-GEAR/Arbor gap survives a rerun.
- **Candidates** (cited, not yet in the graph):
  - Ferreira et al. 2026, "Can LLMs beat classical hyperparameter
    optimization algorithms? A study on autoresearch" (TPE baseline; also
    listed as a candidate in [[literature/papers/lange2026agentic]]).
  - Jeddi et al. 2026, GEAR: Genetic AutoResearch (arXiv 2605.13874).
  - Gao et al. 2026, AutoScientists (arXiv 2605.28655).
  - Antoniades et al. 2026, Heuresis (arXiv 2606.25198).
  - Intology 2026, NanoGPT-Bench.
  - Chen et al. 2026, "Are performance-optimization benchmarks reliably
    measuring coding agents?" (arXiv 2607.01211), on hardware sensitivity
    and score aggregation.
