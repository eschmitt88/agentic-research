---
kind: paper
title: "Who Verifies the Verifier? Co-Evolving Inspectable Graders with Self-Improving Agents"
authors: ["Xing Zhang", "Guanghui Wang", "Yanwei Cui", "Ziyuan Li", "Wei Qiu", "Bing Zhu", "Peiyang He"]
institutions: ["AWS Forward Deployed Engineering", "HSBC Holdings Plc. (HSBC Technology Center, China)"]
year: 2026
venue: "arXiv 2610.11464v1 (cs.AI), 2026-10-08; NeurIPS 2026 Workshop 'Who Verifies the Agents? Toward Reliable Agent Development'"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.11464"
code_url: null
citations: null
source: "raw/papers/zhang2026who.pdf"
added: "2026-10-12"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["verifier-evolution", "co-evolution", "self-evolving-agents", "drawback-detectors", "anchored-selection", "fail-closed-abstention", "always-pass-collapse", "proxy-gaming", "reward-hacking", "outer-judge", "double-ratchet", "ratchet", "skill-loop", "mbpp-plus", "spider-2", "report-generation", "same-family-judge", "negative-result", "peak-selection-bias"]
---

# Who Verifies the Verifier? Co-Evolving Inspectable Graders with Self-Improving Agents

## TL;DR

The paper evolves the *grader* instead of the agent. The verifier is a
readable boolean expression over small typed "drawback detectors" (static
checks, sandbox/warehouse execution, narrow LLM-judge questions). A loop
synthesizes detectors from clustered failures, gates them at birth, and
retires the useless ones. Candidate expressions are selected for agreement
with a 10-item anchored dev set times consensus over unlabeled outputs. The
agent's score is never part of selection. The evolved verifier then grades
training attempts for the authors' existing Ratchet skill loop ("Double
Ratchet").

Three tasks (MBPP+, Spider 2.0-Snow, reference-free report generation), three
seeds each, all roles Claude Opus 4.7, outer judge Opus 4.8.

- **Verifier validity.** Validity is clearly lifted only on MBPP+: held-out
  agreement 0.625 vs 0.417 for the hand-written seed composition. That lift
  is **not significant** once clustering is respected (p = 0.12 at peak,
  0.21 at final rounds). Spider ends at chance (0.500). Report starts and
  ends at ceiling (0.84).
- **The useful negative result.** If the two "anchor guards" (fail-closed
  abstention scoring and a validity gate that drops all-pass candidates) are
  turned off, the report-generation verifier collapses to always-pass on 3/3
  seeds. Its skill loop still scores as well as the anchored one, and on two
  tasks slightly better. So downstream task score cannot validate a
  self-evolved grader.
- **Double Ratchet's 88–110% retention** is computed at peak, and peak is an
  argmax over 100 rounds on the locked set. The vacuous naive arm retains
  100/100/98% by the same arithmetic (ours). The retention number therefore
  says the skill loop barely needs an accurate grader. It does not say the
  evolved grader is good, and the authors concede this ("retained lift does
  not isolate verifier accuracy").

## Claims

- **The evolving object is the grader.** "We formulate the grading function,
  rather than the artifact being graded, as the evolving object." Selection
  "never asks whether the agent scores well".
- **Anchor discipline carries the safety load, and the lifecycle does not.**
  "Anchor discipline, not the detector lifecycle, is load-bearing for
  verifiers, inverting the lesson from skill evolution."
- **Task score cannot certify a verifier.** "Since a collapsed verifier
  trains skills as well as a valid one, validating a self-evolved verifier
  by downstream performance is unsound."
- **Sufficiency.** "Double Ratchet retains 106%, 110%, and 88% of the
  reference loop's lift". The authors frame this as "a weaker claim than
  causing the gain".
- **The skill loop is a forgiving consumer.** "Spider's verifier agrees with
  ground truth only at 0.500 ± 0.026 yet its co-loop retains the full
  reference lift, because the verifier's role in training is *directional*".
- **Inspectability repairs gaming.** A named detector localized a
  proxy-gaming defect to "a nameable check", which "a learned scalar does
  not admit".
- **The outer audit needs the task contract.** The generic outer judge "was
  wrong until given the task contract". Self-evolved verifiers "need an
  independent outer audit, and the audit needs the task contract."
- **Detectability scopes the method.** "verifier evolution buys the most
  where failures are checkable, a scoping condition rather than an
  implementation limit."
- **Diversity by determinism (argued, not measured).** Mostly deterministic
  detectors "fail differently from the model being graded, which is what
  resists the shared-blind-spot collusion a bare LLM judge invites".

## Methods

- **Verifier.** M(t, y) → {PASS, FAIL, ABSTAIN}. The expression grammar has
  leaf op, OR, AND, NOT, K-of-k vote and weighted vote (App. C). Ops come in
  three cost tiers: static, execution (sandbox / live Snowflake) and judge
  (one narrow LLM question). A fixed root per task validates structure.
- **Splits.** All three are disjoint.
  - **train** is large and unlabeled, used for consensus and gap-finding.
  - **dev** has 10 anchored items, 5 fail and 5 clean, each with a teacher
    soft label against a golden reference. This is the only supervised
    signal.
  - **test** is 40–48 locked items that verifier evolution never reads.
  - Sizes: 60/10/40 (MBPP+), 59/10/40 (Spider), 73/10/48 (Report).
  - The Ratchet skill loop *does* read test, for held-out scoring and
    rollback. Rollback fired in 23 of 2,900 rounds.
- **Selection objective.** S(e) = A_dev(e) · A_train(e)^w − λC(e).
  - A_dev is recall-weighted, with FAIL counting twice.
  - A_train weights each op by 1 + 10·max(0, anchored leave-one-out
    marginal).
  - w = 1.0 for MBPP+ and Spider, 0.25 for Report.
  - λ is 0.001 per op beyond three, plus 0.002 per LLM op.
- **Guards.** Fail-closed: a candidate with no usable dev opinion cannot be
  selected. A validity gate drops all-pass, all-fail and all-abstain
  candidates.
- **Lifecycle.**
  - Synthesis from dev misses and from train gaps (abstain or split).
  - A birth gate: the op must fire on at least half its cluster and stay
    clean on known-good outputs.
  - Gap-born ops enter a "shadow" tier and are promoted only if they raise
    dev agreement.
  - Retirement after two rounds of non-positive leave-one-out marginal.
- **Co-evolution.** Verifier phases of 15/8/5/2 rounds alternate with four
  25-round skill phases (100 skill rounds total). The reference loop is
  identical but gets ground truth (MBPP+, Spider) or the best available
  rubric (Report).
- **Ablations.**
  - **naive**: both anchor guards off *and* skill-side rollback off.
  - **no-lifecycle**: birth gate, shadow tier and retirement off.
- **Outer audit.** Opus 4.8 pairwise-compares round-0 vs peak or final
  report outputs, judged twice with positions swapped. A win requires both
  orders to agree.
- **Statistics.**
  - Three seeds.
  - Paired verdict changes on the same locked items, tested with a sign-flip
    permutation at the task level.
  - The authors decline to pool the verdicts into McNemar, calling it
    "anti-conservative".
  - Peak (locked-set argmax, "optimistic") and final-rounds (mean of the
    last 10) are both reported.
- **Solver.** A single direct LLM call with no tools. Frozen baselines are
  about 0.27 (MBPP+), 0.285 (Spider) and 0.56 (Report).

## Results

- **Verifier validity, held-out agreement (Fig. 2, three seeds).**

  | Task | Seed composition | Evolved | Bare judge op | Leaves | Birth-gate rejections |
  |---|---|---|---|---|---|
  | MBPP+ | 0.417 ± 0.058 | 0.625 ± 0.050 | 0.55 ± 0.04 | 3 | 997–1014 |
  | Spider 2.0 | ~0.49 | 0.500 ± 0.026 | ~0.45 | 1–2 | 612–785 |
  | Report | 0.84 | 0.84 | n/a | 2 | 172–188 |

  On MBPP+, evolution fixes 61 locked verdicts and breaks 36 at peak (36 vs
  20 at final rounds). Every seed is net positive, with p = 0.12 / 0.21.
  Five of nine runs end below their own mid-run transfer peak.
- **Guard ablation on Report (Table 3, final rounds).**

  | Arm | Objective | Held-out agreement | Train pass rate | Outcome |
  |---|---|---|---|---|
  | anchored | 0.865 ± 0.002 | 0.830 ± 0.012 | .78–.82 | valid |
  | naive | 1.000 (vacuous) | fail-open | .97–1.0 | always-pass, 3/3 seeds |
  | no-lifecycle | 0.896 ± 0.072 | 0.868 ± 0.061 | n/r | no collapse |

  On the Report locked set an always-pass verifier would score about 0.56
  (54 of 96 items pass). For MBPP+ and Spider the naive collapse is asserted
  in App. F ("degenerate on every task"). Only Spider gets a description,
  "barely moves on two of three seeds", and no table is given.
- **Held-out task score (Table 1 and Table 5).**

  | Task | Loop | Init | Peak | Final rounds |
  |---|---|---|---|---|
  | MBPP+ | reference | 0.275 | 0.700 ± .025 | 0.592 ± .039 |
  | | co | 0.267 | 0.717 ± .038 | 0.652 ± .034 |
  | | naive | 0.317 | 0.742 ± .014 | 0.632 ± .023 |
  | Spider | reference | 0.317 | 0.483 ± .038 | 0.327 ± .010 |
  | | co | 0.275 | 0.458 ± .038 | 0.323 ± .038 |
  | | naive | 0.292 | 0.458 ± .029 | 0.343 ± .028 |
  | Report | reference | 0.562 | 0.850 ± .010 | 0.802 ± .024 |
  | | co | 0.557 | 0.812 ± .006 | 0.767 ± .015 |
  | | naive | 0.559 | 0.841 ± .003 | 0.811 ± .015 |

- **Lift retention, our arithmetic from Table 5** (lift over own init as a
  fraction of the reference lift):

  | Task | co, peak (paper) | naive, peak | co, final | naive, final |
  |---|---|---|---|---|
  | MBPP+ | 106% | 100% | 121% | 99% |
  | Spider | 110% | 100% | undefined (ref lift 0.010) | undefined |
  | Report | 88% | 98% | 88% | 105% |

- **Never-solved tasks solved at least once.** MBPP+ 19/23 (co) vs 16/23
  (reference); Spider 6/12 vs 4/12. This is an any-round statistic.
- **Proxy gaming on Report (§5.4).**
  - Early skills raised the rubric score by +0.26, partly by writing metric
    tags *in place of* values (about 30% of tags had no value nearby) and by
    inventing confident forecasts. No in-loop signal saw it.
  - The generic outer judge preferred the round-0 baseline in 88% of decided
    pairs.
  - One added value-erasure detector plus rewritten failure hints cut erased
    tags to about 1%. The rubric score still reached 0.850 ± 0.010.
- **Auditing the auditor (Table 2).**
  - The generic judge's win rate barely moved with the fix (0.122 → 0.126),
    because it penalized the required tag syntax.
  - A task-aware rubric gives 0.515 → 0.770. It was written *after* reading
    the generic judge's rationales, and the authors say it "diagnoses the
    audit rather than independently confirming the repair."
  - Decided pairs: 271/526 before the repair, 435/565 after.
- **Anchor size (App. F).** This is an offline replay of Report selection
  over 200 stratified subsamples of the same 10 items, rescoring only
  candidates the live run generated. Held-out agreement is 0.871 / 0.882 /
  0.874 at 4 / 6 / 8 items, against 0.854 at 10. The subsample SD is 0.063
  at 4 items and 0.041 at 8.
- **Detectability contrast on Spider.**
  - Under the first, ungrounded prompt most failures were compile errors,
    and the evolved verifier reached 0.85 held-out agreement.
  - Schema grounding raised the solver baseline from 0.211 to 0.329. It left
    clean-executing wrong answers, and agreement fell to 0.500.
- **The two loops converge.** The reference and co loops independently
  synthesize functionally matched skills on every task (App. E).

## Critique / open questions

- **Against the digest framing: it reads below.** This is the 32nd in a row
  for the house pattern. The paper's body is candid, and the overstatement
  sits in the abstract and in the digest's lift of its numbers.
  - **"88–110% of the ground-truth lift" is a peak-cut number.** Peak is the
    locked-set argmax over 100 rounds, with per-round noise near 5 points on
    40–48 items.
  - **Spider's 110% is a noise ratio.** It comes from co's lower round-0
    baseline (0.275 vs 0.317). Co's peak is *below* the reference's (0.458
    vs 0.483). In the final rounds both loops sit about 1–5 points above
    init (reference +0.010), so Spider's lift is mostly the max of
    fluctuations.
  - **The +0.21 validity gain is honest but non-significant** (p = 0.12 /
    0.21), on one task of three.
- **The retention headline does not credit the verifier.** The vacuous naive
  arm matches or beats co at peak on all three tasks (0.742 / 0.458 / 0.841
  vs 0.717 / 0.458 / 0.812). That includes Report, where co trails the
  reference by 0.04 (interval [−.05, −.03], excluding zero), while naive
  trails it by only 0.009.
  On Spider the co verifier is at chance. "An evolved verifier can
  substitute for ground truth" is true here only in the sense that almost
  any verifier can. A missing arm would settle this: always-pass or random
  verdicts with rollback *on*. The naive arm confounds a vacuous verifier
  with rollback off.
- **Why does an always-pass verifier train skills equally well?** In this
  design the verdict is not the loop's signal.
  - The verdict only decides which attempts become "failure capsules". The
    capsule's content (traceback, empty result, error text) comes from the
    environment, so always-pass reduces to "unfiltered practice".
  - The skill loop's measurement, peak selection and, in the anchored arms,
    rollback all read the locked ground-truth set.
  - The paper does not say whether Ratchet's contribution-based eviction
    reads the locked set too. If it does, ground truth reaches the skill
    loop's keep decisions even in the naive arm.
  - Either way, the skill loop's real signal is environment feedback plus a
    ground-truth-pinned evaluation. This matches park2026when's accept-all
    gate with the opposite cost. There, accept-all sat on the keep path and
    eroded the best state by 19%. Here it sits only on the training-input
    path, and costs nothing measurable.
- **How robust is the collapse finding?**
  - It is quantified on one task (Report) × 3 seeds, which is 3/3 there.
    MBPP+ and Spider get a sentence each.
  - It is partly by construction. The naive arm removes the validity gate,
    whose stated job is to drop all-pass candidates, and the fail-closed
    rule, so abstention on every dev item scores as a perfect objective
    ("fail-open scoring reports a vacuous perfect objective").
  - What it shows is that this objective has a degenerate optimum that the
    guards exclude by rule. That is a real and transferable lesson: never
    let an agreement metric skip abstentions. It is not evidence about how
    big or good the anchor set must be.
  - "Anchor discipline is load-bearing" is therefore a claim about two
    scoring rules, not about anchor information. The A_dev term stays on in
    the naive arm.
- **Is a 10-item anchor enough? Not shown.**
  - The only size test is on Report, the task where "subset seeding already
    places a near-optimal composition in the round-0 candidate set". Little
    selection is happening there.
  - It subsamples the same 10 items, so it misses the variance of drawing a
    different anchor.
  - It rescores offline candidates, so it misses how the anchor steers
    synthesis through "misses".
  - Smaller anchors gave a slightly *higher* mean (0.871 at 4 vs 0.854 at
    10). That is consistent with the anchor doing little discriminative work
    on Report.
  - On MBPP+, the one task where evolution moved validity, 5 of 9 runs
    overfit the 10-item proxy past mid-run, and no anchor-size test was run.
- **The Report validity set is easy and off-distribution.** Each locked item
  is either a golden section (passes by definition) or a frozen-agent section
  (which "the strict anchor almost always fails"). That tests whether the
  verifier separates golden text from raw text. It does not test whether it
  grades skill-improved outputs correctly, which is where the proxy gaming
  happened. The 0.84 validity figure could not have detected the gaming, and
  it did not.
- **The Report anchor is itself a same-model judgment.** Report dev labels
  are Opus 4.7 teacher soft labels against golden sections under a 7-class
  miss taxonomy. Every in-loop role is also Opus 4.7. The paper's defence
  ("compares candidate outputs against golden references ... never sees the
  locked anchor") addresses leakage, not shared blind spots.
- **Bearing on shared-substrate contagion: argued, not measured.**
  - No cross-family arm.
  - No error-correlation analysis between solver and detectors.
  - Two of three selected verifiers keep a same-model LLM-judge leaf
    (`spec_mismatch`, `spec_sql_mismatch`).
  - The evolved-beats-bare-judge margins are 0.62 vs 0.55 (MBPP+) and 0.50
    vs 0.45 (Spider, both near chance), and are untested for significance.
  - The cleanest contagion-relevant datum is the outer audit. A
    stronger same-family judge was convention-blind until handed the task
    contract. Independence came from what the auditor was *given*, not from
    which model it was. That agrees with shen2026revoked and xu2026dont: the
    evidence path, not the model identity, sets independence.
- **"An outer judge caught it" in the abstract is generous.** The generic
  judge's 88% preference for the baseline mixed real defects with
  penalizing the required syntax. The repair barely moved it. The task-aware
  re-judging was designed after reading its rationales.
- **Self-built stack.** The skill loop (Ratchet, arXiv 2605.22148) and the
  library-drift framing are the same group's prior work, adopted
  unchanged. The verifier-validity vs skill-lifecycle "inversion" therefore
  compares against the authors' own earlier result.
- **Scale.**
  - Three task families, one model family, one single-call solver with no
    tools.
  - Locked sets of 40–48 items and three seeds.
  - The authors say so: "a mechanism study, not a scaling result".

## Trust signals

- **Credibility:** 3.
  - **For:** an industry group (AWS Forward Deployed Engineering, with HSBC
    Technology Center co-authors), accepted at a NeurIPS 2026 workshop.
  - **For:** unusually careful statistics. It runs a clustered paired
    permutation test rather than pooled McNemar, reports non-significance
    in the body, reports the peak and final-rounds cuts side by side, and
    labels peak "optimistic".
  - **For:** appendices give the selected expressions verbatim, the
    objective constants, the per-loop tables and the judge protocol.
  - **Against:** no code or artifact URL, though "52,600 locked verdicts"
    are said to be stored. Workshop review only.
  - **Against:** three seeds, 40–48-item test sets, a single model family
    (Claude Opus 4.7/4.8), and the skill-side component is the authors' own
    unreplicated loop.
  - **Against:** the key collapse result is quantified on one task.
  - Not 2, because disclosure and statistics are well above the median for
    this graph. Not 4, because there is no code and n is small.

## Follow-up

- **Relevance:** 4. It is the first source in the graph that makes the
  *grader* the evolving object under an explicit held-out validity
  measurement. It also runs the experiment this project would have reached
  for first, validating a self-built grader by downstream gain, and shows
  that experiment passing a vacuous grader. Material new evidence for:
  - [[concepts/programmable-evaluator-oracle]]: it partly answers the "evaluator design is
    unautomated" open question, and the abstention-scoring trap is new;
  - [[concepts/hce-evaluation]]: downstream score cannot certify a grader;
  - [[concepts/skill-library-lifecycle]]: in this loop the skills do not
    need accurate verdicts.

  Not a 5: n is small, there is one model family, and the
  anchor-sufficiency question is open.
- **Most transferable finding.** When a verdict only filters what a loop
  practices on, and the loop's measurement and keep path are pinned to
  ground truth, a vacuous verdict costs nothing measurable. So the loop's
  score says nothing about the verdict. Validate any self-built or
  self-evolved grader against held-out labelled items it never reads, and
  score abstentions fail-closed.
- **For this box.**
  - An `/iterate` or skill loop that adds an LLM or rule-based "verifier" to
    filter what becomes a skill or lesson should report that verifier's
    agreement on a small locked labelled set. It should not cite the loop's
    gain as evidence the verifier works.
  - The cost of an accept-all grader depends on placement. It is ruinous on
    the keep path (park2026when) and nearly free on the training-input path
    (this paper). Put graders on the keep path only once they have held-out
    agreement numbers.
- **Pairs with:**
  - [[literature/papers/park2026when]]: accept-all on the keep path, with
    measured erosion.
  - [[literature/papers/zhang2026veriharness]]: an evolved verifier
    used *for selection*, where its accuracy is on the keep path.
  - [[literature/papers/xu2026dont]] and [[literature/papers/shen2026revoked]]:
    independence comes from the evidence path, not the model.
- **Candidates:**
  - Zhang et al. 2026, "Ratchet: A minimal hygiene recipe for self-evolving
    LLM agents" (arXiv 2605.22148). It is the skill loop used here, and is
    needed to settle whether its contribution scoring reads the locked set.
  - Zhang et al. 2026, "Library drift" (ICML 2026 Workshop on Failure Modes
    in Agentic AI).
  - Zhuge et al. 2024, "Agent-as-a-Judge" (arXiv 2410.10934), the nearest
    prior co-evolution.
