---
kind: paper
title: "The Winner's Curse in LLM Self-Improvement Loops: Selection Noise, Lock-in, and Acceptance Rules"
authors: ["Litao Hu", "Yutong Tang"]
institutions: ["Meta", "Microsoft"]
year: 2026
venue: "arXiv 2610.09239 (cs.AI), v1 2026-10-07, preprint; the appendix mentions follow-up studies 'added in response to reviews'"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.09239"
code_url: "https://arxiv.org/src/2610.09239v1/anc"
citations: null
source: "raw/papers/hu2026winners.pdf"
added: "2026-10-12"
relevance: 5
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["winners-curse", "selection-noise", "keep-if-better", "acceptance-rule", "noise-band", "mcnemar", "bayes-gate", "e-process", "select-then-confirm", "lock-in", "incumbent-inflation", "selection-set-size", "held-out-audit", "shrinkage", "prompt-optimization", "gepa", "miprov2", "preregistration", "regressional-goodhart"]
---

# The Winner's Curse in LLM Self-Improvement Loops: Selection Noise, Lock-in, and Acceptance Rules

## TL;DR

The paper treats the keep-if-better step of a self-improvement loop as
selection under measurement noise. Each generation, K = 4 candidate
rewrites of a system instruction are scored on a reused selection set of
n items, paired against the incumbent. Every candidate is also scored on
600 held-out "gold" items the loop never sees. Solvers are Qwen2.5-1.5B/7B
(plus Qwen3.5-4B and a 14B proposer), on TREC-50, Banking77 and GSM8K.

Three results matter for this graph:

1. **The loop's own score is badly inflated at small n.** In a
   pre-registered study (4 settings × 8 seeds), greedy loops' final
   selection-set score exceeded held-out accuracy by 13–20 points with
   n = 16 and 1–5 points with n = 256. A random-effects model (the
   breeder's equation / optimizer's curse with a shared incumbent error)
   predicts the per-decision winner's curse well when given oracle
   quantities.
2. **The tested acceptance rules, including noise-band-like ones, did not
   improve whole runs.** McNemar at α, select-then-confirm and a
   pilot-calibrated Bayes gate all failed to beat greedy over 20
   generations on a reused set. Two were significantly worse in one
   setting each. The same rules *did* beat greedy one decision at a time
   on fresh items, mainly by committing less often.
3. **A small paired audit fixes the bias but not the variance.** Scoring
   the seed and the final instruction on 64 never-used items removes the
   average overstatement of the reported gain (bias ≈ 0), but its RMSE is
   still 5.7 points at n = 16, about the size of the mean true gain (6.2).

## Claims

- **Keep-if-better is selection under noise, and the bias is predictable.**
  Proposition 1: the best-measured candidate's gain over the incumbent is
  overstated by (1 − h²_w)·σ_w·κ_K + E[c]. Here h²_w is the reliability of
  measured differences, κ_K the expected max of K normals, and c the
  incumbent's shared error.
- **Acceptance conditioning adds bias even with one candidate.** For a
  single candidate with true effect −e and noise N(0, e²), the accepted
  mean error is ≈ 1.53e. This applies to `/iterate`'s K = 1 cycles.
- **The Bayes-optimal one-step rule is a threshold** τ* = −μe²/s². It is
  large when most proposals are harmful (μ < 0) and candidates barely differ
  (s ≪ e). A fixed-level test uses τ = z_α·e "regardless of μ and s".
  Greedy (τ = 0) is optimal only if μ = 0 (Prop. 2).
- **Regressional Goodhart (Prop. 3).** With fresh selection sets, s = 0 and
  μ < 0, greedy commits with probability p each generation, and held-out
  drifts down by |μ|p per generation "while every commit reports a positive
  gain."
- **Lock-in on reused sets.** The incumbent carries its own positive
  selection error. Later candidates must beat a lucky number, which acts as
  an implicit threshold. The authors label this "possible" and do not
  identify it causally.
- **Most rewrites after the first are harmful.** After the first rewrite of
  the one-sentence seed, only 4–39% of rewrites improve on the incumbent.
  Mean held-out effects are −0.2 to −13.5 points.
- **"The tested rules did not reliably improve final held-out gains"**, so
  the authors "do not recommend the tested rules as a general remedy for
  adaptive reuse." Their recommendation is to report a held-out score with
  its standard error.

## Methods

- **Loop.** Proposer = solver (unless stated otherwise). The proposer sees
  the current instruction, a task description and 4 failures, and writes
  K = 4 rewrites per generation. T = 20 generations.
- **Selection sets.** n ∈ {8 … 256} items, reused across generations and
  nested across n. Held-out is 600 gold items excluded by item ID; a
  duplicate-text audit found 12 TREC overlaps, and the primary results
  survive their removal. A candidate's held-out effect has SE 1.5–2 points.
- **Studies.**
  1. Exploratory (v1): 3 seeds, oracle-instrumented runs, replicated on a
     second inference stack (llama.cpp, RTX 3080 Ti; Appendix D).
  2. Pre-registered confirmatory (v2): fresh gold set, seeds 11–18, and a
     feedback pool of 256 items separate from the selection set, so that n
     changes only the evaluation. 4 settings × 9 arms × 8 seeds = 336 runs;
     Holm-corrected tests on five hypotheses H1–H5.
  3. Current models: Qwen3.5-4B self-improvement from the seed and from a
     strong start.
  4. Optimizers: GEPA and MIPROv2 in DSPy (Qwen3.5-4B task model,
     Qwen3.8-27B proposer, n_val ∈ {16, 64, 256}, 5 seeds).
- **Acceptance rules tested.** All rules pick the best-measured candidate
  and differ in when they commit.
  - **Greedy**: Δ̂ > 0.
  - **McNemar**: one-sided exact test against the incumbent at α = 0.05,
    0.20, or a pilot-tuned α. Not corrected for choosing among K.
  - **PACE**: the anytime-valid e-process of Shawn (2026), with its
    published fixed bet. Also a GRAPA-style e-process.
  - **Select-then-confirm**: choose on one half of D, commit only if the
    winner also beats the incumbent on the other half.
  - **Threshold on Δ̂, pilot-tuned**: a fixed δ.
  - **Bayes gates (Eq. 3)**: noise terms from the generation's item
    vectors. The prior (μ, s²) comes either from a pilot run with a large
    eval set (pilot-calibrated) or from the loop's own history (online EB
    gate, optionally with "resolution matching", which grows n). Every
    Bayes gate also requires a positive measured gain.
  - **Greedy, fresh D**: a new selection set every generation.
- **Evaluation of rules.** Rules are judged by the held-out value of
  their decisions, not by nominal error rates. The authors note that
  best-of-K and item reuse void the tests' guarantees.
- **Model check (Fig. 1).** The gold set is split 200 times into a pseudo
  selection set and a truth set, over 9 settings. The prediction uses the
  empirical effect distribution (MAE 0.034) or the Gaussian model (MAE
  0.064). A causal within-loop estimate of h²_w is also tested.

## Results

- **H2, self-report gap (Table 1/3).** Greedy proxy−held-out gap at
  n = 16: 17.3, 13.2, 16.6, 19.6 (1.5B TREC, 1.5B Banking77, 7B TREC,
  1.5B GSM8K). At n = 256: 1.4, 2.7, 1.3, 4.9. The paired reduction is
  significant in 3 of 4 settings; Banking77 has p = 0.058.
- **H1, held-out gain with n.** Gains rise on TREC: 13.7 → 24.2 (1.5B) and
  14.0 → 20.4 (7B). They do not rise on GSM8K (+1.6 ± 2.2, p = 0.48), and
  no GSM8K arm gained more than 3.1 points. On Banking77 greedy gains are
  about 0 at every n (−0.6, 0.0, −0.5).
- **H3, Goodhart loss on Banking77, did not replicate.** −0.6 ± 0.7 held-out
  while the selection score rose 12.5.
- **H4/H5, rules vs greedy at n = 16 (whole runs).**

  | Rule − greedy | pooled (32 pairs) | worst setting |
  |---|---|---|
  | Pilot Bayes gate | −2.1 ± 1.1 (p = 0.06; 95% CI [−4.3, 0.1]) | GSM8K −5.7 (p = 0.032) |
  | Select-then-confirm | −1.6 ± 1.2 (p = 0.19) | 7B TREC −6.4 (p = 0.045) |

  Fixed-level McNemar "gave up most of the available gain on TREC"
  (−7.4 and −12.3). At n = 16 it made **3 commits across 32 runs**, against
  greedy's 69. The pooled H4 test had 80% power for a 3.2-point
  difference.
- **Isolated decisions on fresh items (Table 13, exploratory).** Value per
  decision at n = 16: greedy −0.47 [−0.82, −0.03]; McNemar α = .05 +0.06;
  α = .20 +0.12; pilot Bayes gate +0.28; tuned McNemar +0.19. Pilot gate −
  greedy = +0.77 [0.60, 0.93]. The rules avoid greedy's loss "mainly by
  committing less often". Greedy commits in 44% of decisions, the pilot
  gate in 8%. About half of the rules' commits are still harmful. By
  n = 128 greedy is positive (+0.50) and the advantage shrinks to ≈ 0.1.
- **Prior transfer (Table 14).** A pilot from another model on the same
  task keeps the protection. A pilot from another task protects against
  greedy's losses but is no better than McNemar α = .05.
- **Native-loop commits (Table 5, post hoc).** First-generation commits
  average 9.1 observed vs 9.0 predicted at n = 16, 3.2 vs 3.8 at 64, and
  0.1 vs 1.3 at 256. Single settings are off by several points (1.9 vs 5.9
  on 1.5B TREC; 12.9 vs 7.0 on Banking77). Later commits overstate *less*
  than the fresh-set reference (5.2 vs 9.7 at n = 16), which the authors
  read as consistent with lock-in.
- **Incumbent inflation accumulates (Table 4).** Confirmatory, generations
  11–20: 13.6 ± 1.7 (n = 16), 7.2 ± 1.1 (64), 1.7 ± 0.6 (256). The share of
  candidates whose measured effect is positive falls from 61% in generation
  1 to 1.7% in generations 11–20, so greedy nearly stops committing late in
  the run.
- **Fresh D does not help (exploratory, one setting, 3 seeds).** Fresh-D
  greedy commits twice as often (33 vs 16), with 39% vs 25% harmful, and
  gains less held-out (+14.4 vs +21.3).
- **Strong starts (Qwen3.5-4B, Table 8).** From a competent instruction,
  only 5% of TREC rewrites improve (mean effect −14). At n = 16 the loop
  reported +14.8 (TREC) and +11.7 (Banking77) while held-out moved −1.5 and
  +0.2. The extra four seeds and the combined p-values are exploratory.
- **GEPA/MIPROv2 (Tables 9–12).** At 1000 metric calls with 16 validation
  items, GEPA's validation score claimed +38.8 on TREC for a real +19.2,
  and +18.8 on Banking77 for a real +1.2. Its held-out gain did not grow
  with n_val: 16 items let it make about 10× more reflection calls, with
  no held-out benefit. Proposition 1 tracks how overstatement varies but
  under-predicts GEPA by a factor of 1.1–1.9, because GEPA's pool is itself
  selected on validation. MIPROv2 at 256 items doubled the TREC held-out
  gain (11.4 vs 5.8, p = 0.001).
- **Reporting the gain (Table 7, 151 runs at n = 16).**

  | Estimate of seed→final gain | extra evals | bias / RMSE (n = 16) |
  |---|---|---|
  | self-report on selection set | 0 | +10.8 / 15.5 |
  | shrink each commit by loop's own ĥ²_w | 0 | +0.7 / 10.7 |
  | paired audit, 16 items | 32 | −0.0 / 10.7 |
  | paired audit, 64 items | 128 | −0.0 / 5.7 |
  | paired audit, 128 items | 256 | −0.1 / 4.4 |

  Shrinkage leaves setting-level bias of −8.5 to +5.8. "Resolving gains of
  a few points would take a few hundred items."
- **Allocation (Table 17).** Splitting B = 256 evaluations per generation
  into K × n from 16×16 to 1×256: all differences are within seed
  variation.

## Critique / open questions

- **House-pattern check: the paper reads at the digest framing, and its
  abstract does not overstate.** This breaks the run of overstating
  abstracts. The 13–20 / 1–5 figures are the pre-registered H2 means over
  all four settings, not a best cut. The abstract also reports the
  unfavourable results: H1 fails on GSM8K, the model matches native loops
  "though not setting by setting", and the rules lose. The one max-cut is
  in the introduction, not the abstract. "26 and 7 points" for GEPA and
  MIPROv2 are the largest cells (26.5 at 2250 calls; MIPROv2's 6.6 on
  Banking77). At the common 1000-call budget GEPA's TREC inflation is
  14.1.
- **Where the digest framing over-reads.** "The tested acceptance rules did
  not beat greedy over whole runs" is accurate, but it is easy to read as
  "thresholds don't help". The body is narrower in three ways:
  1. The null is for *whole runs on a reused set*. One decision at a time
     on fresh items, every calibrated rule beat greedy (Table 13).
  2. The whole-run result is not a clean equivalence. The pilot gate's
     pooled CI is [−4.3, +0.1], which leans *negative*, and two rules were
     significantly worse in one setting each.
  3. The losses are concentrated where real gains were large and
     resolvable only in aggregate (TREC). Strict tests "rarely commit at
     n = 16 and therefore stay near the seed, which is optimal on Banking77
     … and costly on TREC".
- **"Overstate held-out by 13–20 points" is a level gap, not a gain gap.**
  It is the final proxy minus held-out accuracy, and it includes the seed's
  own offset on 16 items. The overstatement of the *reported gain* across
  all rules is +10.8 at n = 16 (Table 7). Greedy's offset-corrected
  incumbent inflation is 13.6 (Table 4).
- **The noise model covers item sampling only.** Scoring is deterministic,
  with greedy decoding, so v is per-item disagreement between candidate and
  incumbent. Seed-to-seed training noise, the kind `/iterate`'s `--seeds`
  band measures, is absent. Remark (iii) concedes that stochastic rewards
  add a variance term, and Section 6 says candidate-specific rollout noise
  predicts a *larger* curse. GSM8K's 9.8% flipped items under batched
  inference show what happens when it creeps in: GSM8K is the least
  consistent setting in Fig. 1.
- **The domain is far from ML research loops.** The artifacts are one-line
  seed instructions for 1.5B/7B classifiers, where the first rewrite does
  most of the work (gen-1 is 38–112% of the final gain). Proposal effects
  are heavy-tailed (excess kurtosis up to 7.6) with catastrophic rewrites.
  None of the tasks are agentic or code tasks, the largest proposer is
  27B, and n ≤ 256. An ML-research loop with a 2,000-row validation split
  sits at a different reliability h²_w. The transferable quantity is the
  ratio of measurement SE to proposal spread s, not n itself.
- **The model's validation leans on oracle data.** Fig. 1 uses gold-set
  quantities from 256-item trajectories, and its intervals "omit run-to-run
  variation". The native-loop check (Table 5) is post hoc, covers four
  Qwen2.5 settings, and the authors call it "agreement on average, not
  calibration."
- **The Bayes gate is impractical as specified.** It needs a pilot with a
  large evaluation set on the same model and task. The online version,
  which estimates the prior from the loop's own history, failed: its s²
  collapsed to zero after two generations and it committed measured
  regressions. The "never commit a measured regression" safeguard was added
  post hoc.
- **The pre-registration was imperfect but candid.**
  - The analysis script was written 10 minutes into the runs (16/336
    done).
  - Pooled tests were implemented as one-sample t rather than with setting
    fixed effects (p 0.054 vs 0.062).
  - The strong-start extension, its one-sided direction and its stage
    weights were chosen after seeing results.
  - GEPA's reported-gain measure was added after eight runs had been seen.

  All of this is disclosed, and the H1/H2 conclusions do not depend on any
  of it.
- **"Harmful" has a resolution floor.** The gold set resolves about 1.5–2
  points, so "harmful commit" (held-out < 0) includes noise. The ≤ −1.8
  column is the honest one. 14% of late candidates sit at 0–1.8 points,
  "consistent with no gain".
- **Lock-in is asserted, not identified.** It is offered as a "possible
  mechanism" and inferred from commits overstating less than the fresh-set
  reference. Table 4's collapse in late-generation commit rate fits it, but
  so would proposal exhaustion.
- **Open for this graph:** no rule was tested that sets the threshold from
  *rerun spread of the same artifact*, which is what the accepted noise
  band does. McNemar's τ = z_α·e is the nearest analogue, with e from
  per-item discordance. The halting use of a band (stop after k
  sub-threshold cycles) was never tested; every rule here runs a fixed
  T = 20.

## Trust signals

- **Credibility:** 4.
  - For: code, saved run records, unit tests and a `reproduce.py` ship as
    62+ arXiv ancillary files (verified on the abs page). There is a dated
    pre-registration with deviations itemised, an independent
    second-inference-stack replication (Appendix D), a duplicate-question
    sensitivity audit, an inference-nondeterminism re-score, a power
    analysis, and per-arm tables for every condition. The model is
    standard selection theory (breeder's equation, optimizer's curse) with
    proofs.
  - Against: two authors with Meta and Microsoft affiliations, but the
    compute is a side-project budget (about 28 rented GPU-hours plus a
    consumer RTX 3080 Ti), so it is not a lab product. It is not peer
    reviewed: the appendix mentions reviews, but no venue is named. Models
    are small, artifacts are instruction-only, and there are three
    classification/math tasks.
  - Not 5 because there is no peer review, no independent reproduction and
    no citations yet. Not 3 because the reproducibility and disclosure are
    well above what most preprints in this graph provide.

## Follow-up

- **Relevance:** 5. It is the first source in this graph that directly
  measures the premise of [[concepts/hce-evaluation]]: a loop's selection
  score overstates held-out performance, by how much, as a function of n.
  It has held-out ground truth for every candidate, a predictive model and
  a pre-registered test. It also gives concrete numbers for that concept's
  open question on validation-split size. It is the first source to test
  noise-band-like acceptance rules against greedy keep-if-better. That
  bears directly on the accepted 2026-09-27 `/iterate` noise-band proposal
  and on [[concepts/budget-as-ceiling]]. Transfer to ML-research loops
  needs care (see Critique), which keeps it from being a mechanism-level
  anchor; it is a statistics-level one.
- **Bearing on the accepted `iterate-no-improvement-noise-band` proposal:
  does not undercut it; it supports the premise and warns against the
  proposal's deferred extension.**
  - The proposal uses the band only for the *halting counter*. Champion
    selection stays a bare `>`. The paper's whole-run null is about the
    *keep rule's* effect on final held-out gain, which is a different
    lever. The paper never tests a band-gated stop.
  - What it does show supports the counter's premise. On fresh noise,
    sub-threshold "improvements" are net harmful when most proposals are
    (greedy −0.47 per decision at n = 16), and band-like rules recover
    that loss mainly by declining to call them improvements. That is
    exactly what the counter needs.
  - It warns against the proposal's last risk bullet, extending the band
    to champion selection. On a reused validation split, thresholded keep
    rules were neutral to worse over whole runs. When real gains are large
    but noisy (TREC), a strict band forfeits them.
  - It exposes a gap in the band's construction. A `--seeds` std is
    measured on the *same* validation items, so it captures training noise
    but not the item-sampling error of a reused split. That error is what
    accumulates into incumbent inflation (13.6 points at n = 16).
    Per-item paired disagreement between champion and candidate gives the
    missing term (e = √(v/n)) from predictions `/implement` already
    produces. This is our inference.
  - For `/iterate`'s K = 1 cycles, the best-of-K term vanishes (κ_1 = 0),
    but acceptance conditioning and champion reuse remain. The stored
    champion score is a lucky draw, which is the lock-in analogue.
- **For HCE's held-out set and a fresh re-score set:**
  - Score the *starting* configuration and the final champion on the same
    held-out items (a paired audit). This makes the reported gain unbiased.
  - Size the held-out set for the gain you need to resolve: 64 items gave
    RMSE about 6 points on gains averaging 6.
  - Report the SE next to the number.
  - Do not re-draw validation per cycle as a substitute. It did worse.
  - Shrinkage from the loop's own logs is free and fixes the average, not
    the single run.
- **For population search** ([[concepts/evolutionary-expansion]]): GEPA's
  pool is selected on the validation set, so its overstatement exceeds the
  single-step prediction. A small validation set bought 10× more search and
  no held-out gain.
- **Candidates** (all cited, none in the graph):
  - Shawn 2026, "PACE: Anytime-valid acceptance tests for self-evolving
    agents" (arXiv 2606.08106). An anytime-valid acceptance test; its
    reported degradation did not replicate here.
  - Xu et al. 2026b, "Towards reliable LLM evaluation: correcting the
    winner's curse in adaptive benchmarking" (arXiv 2605.05973), SIREN.
  - Xu et al. 2026a, "Phantom gains: auditing self-improvement against a
    measured null" (arXiv 2608.20290).
  - Arce 2026, a 41-paper methods audit of self-improving-agent evaluation
    (alphaXiv 2609).
  - Ye et al. 2026, "On the fragility of self-improving agents" (arXiv
    2608.18066).
  - Dwork et al. 2015, "The reusable holdout" (Science), and Blum & Hardt
    2015, "The Ladder". Peer-reviewed anchors for constrain-during-search
    that this graph lacks.
