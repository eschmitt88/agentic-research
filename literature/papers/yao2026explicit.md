---
kind: paper
title: "An Explicit Feedback Contract for Iterative ML-Agent Harnesses"
authors: ["Kehui Yao", "Chuanbo Hu", "Jiao Chen", "Ramin Giahi", "Reza Yousefi Maragheh", "Lalitesh Morishetti", "Ziwei Hong", "Yiyun Su", "Jianpeng Xu", "Evren Korpeoglu", "Kannan Achan"]
institutions: ["Walmart Global Tech", "Lehigh University", "Rutgers University"]
year: 2026
venue: "arXiv 2610.28798 (cs.LG), v1 2026-10-20, preprint (11 pages, 2 figures)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.28798"
code_url: null     # no repository, no ancillary files on the abs page (checked 2026-10-26)
citations: null
source: "raw/papers/yao2026explicit.pdf"
added: "2026-10-26"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/information-firewall]]"
tags: ["feedback-contract", "adaptive-overfitting", "reusable-holdout", "ladder-mechanism", "thresholded-release", "paired-standard-error", "noise-band", "patience", "stopping-rule", "no-signal-control", "permuted-labels", "val-test-gap", "test-quarantine", "keep-if-better", "ctr-prediction", "learning-to-rank", "criteo", "mslr-web10k", "codex", "claude-sonnet"]
---

# An Explicit Feedback Contract for Iterative ML-Agent Harnesses

## TL;DR

The harness is the subject, not the agent. In today's ML-agent harnesses the
agent gets an exact validation score after every attempt, which turns the
validation split into a reused holdout under adaptive selection. The paper
proposes a five-clause "feedback contract". Its core is Blum & Hardt's Ladder
with a data-dependent margin. A submission's score is released only if it
beats the current champion by at least one *paired* standard error, computed
over per-item loss differences on the validation set. Patience counts only
these released updates. Experiments use one coding agent (GPT-5.6-Sol via
Codex) on Criteo-style CTR, MSLR-WEB10K ranking and a permuted-label control,
with a 30-submission cap.

Three results matter for this graph:

1. **On the no-signal control, exact feedback reports a gain that does not
   exist.** Over 5 seeds the exact-score agent ends reporting +0.0157 gain
   over the base rate, with test −0.0006 (gap 0.0163 ± 0.0026). Under the
   contract the released gain is +0.0033 and test is +0.0033 (gap
   0.0000 ± 0.0025). The contract shrinks the false gain about fivefold. It
   does not remove it.
2. **The stopping difference is large on one backbone and absent on the
   other.** The contract's patience halts at 12.3 ± 4.4 (Criteo),
   4.3 ± 0.7 (MSLR) and 10.4 ± 4.4 (control). Exact runs to 30, 27 and 30.
   The *harness* stops the run in both arms: patience p over consecutive
   non-updates, with p not reported. The agent does not choose to stop.
   Under Claude Sonnet 4.6 on Criteo the two arms stop at 23.4 vs 22.6. The
   paper says those rows ran under an older protocol.
3. **On real tasks the effects are tiny and inside the authors' own noise.**
   The val–test gap narrows by about 8% on both tasks (0.0060 → 0.0055 AUC,
   0.0134 → 0.0123 NDCG@10). Test moves +0.0003 AUC and +0.0005 NDCG. The
   authors say three seeds "cannot separate a small effect from none".

## Claims

- Iterative agent development is adaptive data analysis. A validation score
  is unbiased for a model fixed in advance. Read after every edit in a
  keep-if-better loop, it reports "the smallest score it happened to meet
  along an adaptively chosen path".
- Benchmarks (MLE-bench, MLAgentBench, TML-Bench) fix splits and budgets
  carefully but leave the feedback channel implicit: full precision, as
  often as the agent asks. Table 1 marks no prior agent system as having
  bounded feedback. That omits [[literature/papers/bertran2026fits]], which
  ran a one-bit Ladder with Claude Opus agents on 8 tasks; the paper does
  not cite it.
- Agents differ from leaderboard submitters in three ways a threshold alone
  does not cover. They can read intermediate files, so readable files must
  be specified. There is no closing date, so a stopping rule is needed. They
  work inside the repository, so anything not actively withheld can leak.
- The contract's stopping rule "is guaranteed to fire". The agent halts
  "because no further improvement can be certified, not because a budget
  ran out" (Prop. 2, for a fixed margin).
- Budget exhaustion "is usually read as diligence. It is better read as a
  symptom: when the feedback channel is unbounded, noise keeps supplying
  improvements to chase."
- The guarantee is "narrow, deliberately". It covers the released
  transcript, not the final test score: "the contract makes the reported
  number trustworthy, not better."

## Methods

- **Setting.** Splits T (open), V (reachable only through the harness), H
  (unreachable until the run stops). Each round the agent submits one
  candidate. The harness scores it on V privately and releases one pair
  (u_t, R_t): whether the released score changed, and its current value.
- **Five clauses.**
  - C1 splits: V's labels, per-example losses and prediction files stay in
    the harness.
  - C2 released feedback: one pair per round. A rejected submission is not
    told how far it missed.
  - C3 stopping: budget T_max and patience p fixed in advance. Halt at the
    budget or after p consecutive non-updates. Patience counts updates,
    not score changes.
  - C4 test quarantine: name the final model (default the champion) before
    any test feedback, evaluate H once, release aggregates only.
  - C5 recorded artifacts: configuration, hidden loss vectors, transcript,
    seeds, agent IDs and final metrics, enough to replay the transcript
    without rerunning the agent.
- **Release rule (Algorithm 1).** Keep the champion's per-item loss vector
  L*. For a candidate, compute r_t (mean loss) and
  s_t = std(L_t − L*). Update iff r_t < R_{t−1} − s_t/√n, i.e. the
  candidate wins by one paired SE over the validation *items*. Release the
  score rounded to a multiple of 1/n. The deployed margin is
  max{s_t/√n, η_min}, with η_min = c·n^{−1/3}·log^{1/3}(kn). The value of
  c is not reported.
- **Theory.** Both propositions are for a fixed margin η. Prop. 1 (Blum &
  Hardt 2015): pverr = O(n^{−1/3} log^{1/3}(kn)) w.h.p., logarithmic in the
  number of submissions k. Prop. 2: at most ⌈1/η⌉ + 1 updates, so a run
  halts by min{T_max, p(⌈1/η⌉ + 2)}. The paper notes that without a margin
  an adaptive submitter can push observed loss to 1/2 − Ω(√(k/n)) on
  no-signal data, so a patience rule on raw improvements need not fire.
- **Data.**
  - Criteo-style CTR: 100k train / 25k val / 50k test, AUC.
  - MSLR-WEB10K Fold1: 6,000 / 2,000 / 2,000 queries, macro NDCG@10.
    The effective n for the bound is about 2,000 queries, not rows.
  - No-signal control: labels permuted before splitting, Criteo-sized.
    Gain is measured against the base-rate predictor; true gain is zero.
- **Arms.** *Exact* returns the validation score after every submission.
  *Ours* is Algorithm 1. Patience applies to both. Exact has no update
  events, so its patience "falls back on observed improvement", which is a
  bare `>` counter. The arms therefore differ in three things at once: what
  is released, what becomes champion, and what patience counts.
- **Runs.** 30-submission cap. Three "attack seeds" for Criteo and MSLR,
  five for the control. Table 3 uses five seeds; Table 4 does not state
  seed count. The agent backbone for the main table is GPT-5.6-Sol (Codex),
  per Table 5.
- **Sensitivity studies.** Public-validation size on Criteo (nested subsets
  2k–20k of the 25k split, "Signed Boosting", attack strength chosen per
  seed and method on validation). Fixed η ∈ {0.001, 0.005, 0.010} vs
  paired SE on Criteo. A second backbone (Claude Sonnet 4.6) on Criteo.

## Results

- **Main table (Table 2, mean ± SE over seeds, cap 30).**

  | Task | Arm | Stop round | Test | Val–test gap |
  |---|---|---|---|---|
  | Criteo (AUC) | Exact | 30.0 ± 0.0 | 0.7820 ± 0.0001 | 0.0060 ± 0.0001 |
  | | Ours | 12.3 ± 4.4 | 0.7823 ± 0.0000 | 0.0055 ± 0.0000 |
  | MSLR (NDCG@10) | Exact | 27.0 ± 0.0 | 0.4446 ± 0.0003 | 0.0134 ± 0.0003 |
  | | Ours | 4.3 ± 0.7 | 0.4451 ± 0.0001 | 0.0123 ± 0.0001 |
  | No-signal (gain) | Exact | 30.0 ± 0.0 | −0.0006 ± 0.0018 | 0.0163 ± 0.0026 |
  | | Ours | 10.4 ± 4.4 | 0.0033 ± 0.0020 | 0.0000 ± 0.0025 |

  Validation reported = test + gap. On the control that is +0.0157
  (Exact) vs +0.0033 (Ours).
- **Validation size (Table 3, Criteo, 5 seeds).** Ours has the higher test
  AUC and smaller gap at every size. The advantage shrinks with n:
  - 2,000 items: test 0.7782 vs 0.7816; gap 0.0117 ± 0.0047 vs
    0.0051 ± 0.0044.
  - 20,000 items: test 0.7818 vs 0.7821; gap 0.0055 vs 0.0048.

  The gap SEs overlap at every size.
- **Fixed vs paired margin (Table 4, Criteo).**

  | Margin | Stop | Test | Gap |
  |---|---|---|---|
  | η = 0.001 | 14.6 ± 1.9 | 0.7831 ± 0.0013 | 0.0060 ± 0.0006 |
  | η = 0.005 | 9.8 ± 1.1 | 0.7684 ± 0.0090 | 0.0066 ± 0.0012 |
  | η = 0.010 | 7.8 ± 0.4 | 0.7727 ± 0.0025 | 0.0054 ± 0.0003 |
  | paired SE | 14.2 ± 2.4 | 0.7840 ± 0.0003 | 0.0053 ± 0.0003 |

  The paired rule runs as long as the smallest fixed margin, so its
  advantage "is not that it stops sooner". Large fixed margins lose test
  AUC, non-monotonically. This table's configuration differs from Table 2's
  (Ours test 0.7840 here vs 0.7823 there).
- **Backbone (Table 5, Criteo).** Codex rows are the main-table rows. Claude
  Sonnet 4.6: Exact stops at 23.4 ± 3.4, Ours at 22.6 ± 1.9; test 0.7831 vs
  0.7835; gap 0.0058 vs 0.0053. "Those runs have not been rerun under the
  current protocol and should be read separately."

## Critique / open questions

- **Against the digest framing: below framing.** Every number in the digest
  entry is in the paper. Three things the entry leaves out narrow it.
  1. The headline result is the stopping effect, and it is one backbone.
     Under Claude Sonnet 4.6 the two arms stop at nearly the same round
     (23.4 vs 22.6). The paper sets those rows aside as an older protocol
     rather than explaining them.
  2. "Halted after 4–12" is the harness's patience rule firing on
     thresholded updates. The value of p is never reported. Exact's patience
     counts raw observed improvements, and Prop. 2 / Blum & Hardt Thm 5.1
     say that kind of counter need not fire. So most of the stopping
     difference follows from how the counter is defined. It is not an
     observation about agent behaviour.
  3. The real-task effects (~8% gap narrowing, +0.0003 AUC, +0.0005 NDCG)
     are within what the authors say three seeds cannot resolve.
- **The three checks the digest asked for.**
  - *Runs per arm:* 3 seeds on Criteo and MSLR, 5 on the control, 5 in the
    validation-size study, unstated for the fixed-margin study. The 0.0000
    SEs on Criteo test and gap at n = 3 suggest rounding or near-identical
    runs.
  - *Where the paired SE comes from:* validation **items**, not seeds. It is
    the per-item std of champion-vs-candidate loss differences over the n
    validation examples. This is the term that
    [[literature/papers/hu2026winners]] said a `--seeds` band misses: the
    item-sampling error of a reused split, e = √(v/n). It does not include
    training-seed or re-implementation noise, which
    [[literature/papers/wiedmann2026agents]] and
    [[literature/papers/ning2026implementation]] show is larger at the
    hypothesis level. The two are complementary terms. The paper uses only
    the item term.
  - *Who halts:* the harness, under C3 patience, in both arms. The agent
    never decides to stop.
- **The three levers are confounded in the main comparison.** Exact vs
  Ours changes (a) the released information (exact score vs a rounded,
  thresholded best-so-far), (b) the keep rule (bare `>` vs one paired SE),
  and (c) what patience counts. No arm releases exact scores while gating
  patience on a threshold. That arm is what `/iterate`'s accepted noise
  band actually is. Table 4 varies only the margin, inside the contract.
- **The contract still releases a false gain on null data.** +0.0033 was
  released on permuted labels. A one-SE one-sided bar admits about 16% of
  equal-quality candidates per round (our arithmetic). Over 10–30 rounds
  several false updates are expected, and each resets patience. The η_min
  floor exists to stop this, but its constant c is unreported. The control's
  zero gap comes partly from the test gain also landing at +0.0033
  (1.65 SE above zero on data where population gain is zero), which the
  paper does not discuss. "Reports nothing learned" in the conclusion
  overstates its own table.
- **"Parameter-free" is not quite true.** p, c (in η_min) and the
  one-SE multiplier are all choices. Only the per-round margin is
  data-dependent. Table 4 compares against fixed margins, not against
  other multipliers or a z_α bar like McNemar's.
- **Protocol consistency is weak.**
  - Tables 2, 3 and 4 use different Criteo configurations (Ours test 0.7823
    / 0.7816–0.7821 / 0.7840).
  - "Attack seeds", "Signed Boosting" and "attack strength … selected …
    using validation only" suggest that at least the validation-size study
    used a scripted adaptive attacker in the Blum–Hardt style rather than
    (or alongside) the LLM agent. The paper does not say which.
  - The Discussion says exact feedback "ran to the budget on every dataset";
    Table 2 has MSLR at 27.
- **The firewall is specified, not tested.** C1 and Definition 1 exclude
  validation labels, per-example losses and prediction files. The paper
  stresses that classic reusable-holdout guarantees assume the submitter
  cannot see raw data. It never says how the harness enforces C1, what form
  a submission takes (predictions, a model artifact, or code the harness
  runs on V), or whether anyone audited the agent's file access. If the
  harness runs agent code on V, [[literature/papers/farahbakhsh2026agentdiscover]]'s
  candidate-side leak applies (our inference).
- **Bearing on the accepted noise band (counter only, champion stays
  `>`).** This is the first controlled comparison in the graph of a
  thresholded patience counter against a bare-`>` counter in an ML-agent
  loop with held-out ground truth. Under Codex it supports the band's
  premise: on null data the bare counter never fired (30.0 ± 0.0 over 5
  seeds). It adds that the band's threshold should include the paired
  item-SE term, not only seed std. The Claude rows are the counter-evidence
  and are under-explained. It also applies the threshold to the
  *keep rule*. [[literature/papers/hu2026winners]] found thresholded keep
  rules neutral to worse over whole runs at n ≤ 256. Here, at n = 2,000–25,000,
  they cost nothing on test. That is consistent: at large n the threshold
  rarely blocks a real gain. It does not reopen the band-on-champion
  extension for small validation splits.
- **Bearing on redirect-before-halt.** No redirect arm, so the
  redirect-vs-halt comparison is still missing. Two indirect points.
  - On MSLR the contract halted at round 4.3 and lost nothing on test
    against 27 rounds of exact feedback (0.4451 vs 0.4446). On these
    tasks, continuing past a thresholded stall bought nothing. A redirect
    has to beat a halt that forfeits about zero here, though the base
    learners (hashed LR, HistGB) may simply be saturated.
  - Under C2 a rejected submission is not told how far it missed.
    `/iterate` step 3 chooses the next candidate from `delta_from_prior`. A
    contract-style channel would remove that signal. A pivot rule would
    have to work from the transcript of update bits alone.
- **Second instance of ladder feedback with LLM agents.**
  [[literature/papers/bertran2026fits]] (Claude Opus, 8 tasks × 3 runs)
  found one-bit ladder feedback at least as good as scalar feedback on
  holdout. This paper is independent and uncited by it. It adds a
  data-dependent margin, a no-signal control and a stopping rule, but on
  two tabular tasks only.

## Trust signals

- **Credibility:** 2.
  - For: an established industrial applied-ML group (Walmart Global Tech,
    with Lehigh and Rutgers). The theory is the peer-reviewed Ladder (Blum &
    Hardt, ICML 2015) applied correctly, and the paper is clear about the
    gap between theorem (fixed η) and deployed rule (computed margin). The
    limitations paragraph is candid about seed count and effect size, and
    it includes a no-signal control.
  - Against: preprint, no code or artifacts, even though C5 claims
    replayable transcripts. Patience p and the floor constant c are
    unreported. Three seeds on the real tasks. Configurations differ across
    tables. The second-backbone rows run under a different protocol and
    contradict the headline. There is ambiguity about whether some studies
    used a scripted attacker. It omits the closest prior agent work
    (bertran2026fits).
  - Not 3 because the protocol under-specification and mixed setups prevent
    re-deriving the headline from the paper. Not 1 because the group is
    identifiable, the theory is sound, and the control experiment is
    well designed.

## Follow-up

- **Relevance:** 4. It is the first source to compare a thresholded stop
  counter with a bare-`>` counter inside an ML-agent loop. It has a
  permuted-label control where the true gain is known to be zero. And it
  implements the paired item-SE term that hu2026winners inferred the
  accepted `/iterate` noise band was missing. That adds material evidence
  to [[concepts/budget-as-ceiling]] and [[concepts/hce-evaluation]], and a
  second instance of mechanism 2 to
  [[concepts/compression-as-generalization-test]]. Not 5 because the
  stopping effect does not hold on the second backbone, the real-task
  effects are inside noise, and the protocol is under-specified.
- **For the noise band (our inference, not proposed here).** Define the band
  as max(seed std, paired item SE between champion and candidate). The item
  term costs nothing: `/implement` already writes validation predictions,
  so per-item losses for both runs are available. A one-SE bar admits
  about 16% of null candidates, so a z ≈ 1.64 bar, or the η_min floor,
  would make the counter fire more reliably.
- **Missing arm worth asking for (or running downstream).** Exact feedback
  with a thresholded patience counter. That isolates the counter from the
  channel and keep rule, and it matches what `/iterate` does after 10-11.
- **Candidates** (cited, not in the graph):
  - Blum & Hardt 2015, "The Ladder: A reliable leaderboard for machine
    learning competitions" (ICML). Peer-reviewed anchor, also flagged by
    hu2026winners.
  - Dwork et al. 2015, "The reusable holdout" (Science).
  - Pinchuk 2026, TML-Bench (arXiv 2603.05764).
  - Wang et al. 2026, "Self-evolving recommendation system" (arXiv
    2602.10226), an agentic recommender loop with exact feedback.
