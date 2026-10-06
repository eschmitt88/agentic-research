---
kind: paper
title: "False Frontiers: Diagnosing and Mitigating Co-Cheating in Self-Evolving Search Agents"
authors: ["Meijia Chen", "Hao Li", "Zheng Lu", "Hongshan Lin", "Junbai Tian", "Yichen Liu", "Zijun Tian", "Yufan Zou", "Shuhan Sun", "Hanxin Chen", "Zeyu Zhang", "Weizhi Du", "Yueting Li", "Tianyu Shi", "Alaa Khamis"]
institutions: ["Rutgers University", "Independent Researcher (10 of 15 authors)", "University of California, San Diego", "University of Michigan", "McGill University", "King Fahd University of Petroleum and Minerals"]
year: 2026
venue: "arXiv 2609.39102v2 (cs.CL), 2026-10-03; 'Preprint' header"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.39102"
code_url: null
citations: null
source: "raw/papers/chen2026false.pdf"
added: "2026-10-06"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
tags: ["self-evolution", "proposer-solver", "self-play", "pseudo-labels", "co-cheating", "false-agreement", "cross-fitting", "source-level-exclusion", "evaluator-ancestry", "fixed-bank-replay", "post-hoc-audit", "llm-auditor", "search-agents", "dr-zero", "confirmation-bias"]
---

# False Frontiers: Diagnosing and Mitigating Co-Cheating in Self-Evolving Search Agents

## TL;DR

Dr. Zero-style self-evolution trains a search agent with no human data. A
**proposer** turns source documents into question + pseudo-label pairs. A
**solver** trains on the admitted pairs. The proposer is then rewarded by
how many of 5 solver rollouts match its label. The paper audits this loop
post hoc, using an LLM auditor that the loop never sees. It finds
**co-cheating**: a wrong pseudo-label trains the solver, the solver
reproduces the same wrong answer on later questions from that source, and
the match is paid out as proposer reward.

- **False-agreement mass F** is the share of pairs that agree on the same
  wrong answer. Over 3 rounds it climbs from 0.004 to **0.061** (Qwen3.5-4B)
  and from 0.003 to **0.088** (9B).
- **CrossFit** splits the source documents into two folds. A question from
  fold A is scored by an auxiliary solver trained only on fold B, and vice
  versa. The main solver still trains on everything. This brings F down to
  **0.030 / 0.037**.
- **Downstream Cover-EM** rises from 0.400 to **0.488** (4B) and from 0.428
  to **0.512** (9B) on 7 QA benchmarks.

The best evidence is a **fixed-bank replay** with 5 seeds. It holds the
3,000 questions and labels fixed and varies only what the feedback solver
was trained on. A separate solver that still saw the same source does
nothing (F 0.064 / 0.087). A random row split helps a little (0.050 /
0.062). Only a split by **source ID** removes the effect (0.004 / 0.001).
For this graph, that is a clean measurement that **evaluator independence
must be drawn on data ancestry, not on evaluator identity**. The live-loop
headline numbers come from one training run per arm, and the auditor is
never checked against humans.

## Claims

- **Co-cheating exists and compounds.** "co-cheating becomes increasingly
  severe over successive rounds of self-evolution, with pseudo-label
  correctness stagnating or declining even as the in-loop training signal
  improves."
- **Verification alone is insufficient.** MSV draws 3 source-aware and 3
  source-blind answers from the same model and admits a question only if
  the two majorities agree. It "partially reduces false agreement but
  leaves substantial residual co-cheating," and "six samples from the same
  model can share errors."
- **Provenance matters more than label quality.** "changing the provenance
  of proposer feedback is more consequential than improving pseudo-label
  quality alone."
- **A separate evaluator is not enough.** "A separate evaluator is
  therefore not sufficient when its training data retain the same
  source-derived pseudo-labels."
- **The split must follow source ancestry.** A random split leaves the
  path open "because questions derived from the same document can still
  enter both folds."
- **The authors scope it.** CrossFit "does not turn the auxiliary solver
  into a truth oracle." Shared pretraining, overlapping web evidence and
  an adaptive proposer "can still induce correlated mistakes." They
  borrow cross-fitting's "exclusion principle, not its asymptotic
  guarantees." Lower F "can also result from rejecting difficult tasks
  rather than improving learning."

## Methods

- **Loop.** Dr. Zero (Yue et al., COLM 2026). Proposer reward is
  f(k) = (5 − k)/4 for 0 < k < 5 and zero otherwise, where k is the
  number of matching solver responses out of 5. The schedule is 3 rounds,
  each with 18 proposer and 25 solver updates. That gives 129 audited
  steps and 1,600 admitted questions per round.
- **Backbones.** Qwen3.5-4B and Qwen3.5-9B, all starting from the same
  public checkpoint. No human-annotated QA training data is used.
- **Post-hoc audit.** At every step the authors save the source, the
  adopted label and the 5 solver responses used for reward.
  `gpt-6-astra/high` builds an evidence-backed reference from the source
  and judges the saved outputs. Unsupported cases are left unresolved.
  Coverage J/E is about 0.86. The audit never affects admission, updates
  or reward. Metrics:
  - T_P: label truth;
  - T_S: solver truth;
  - A: in-loop agreement;
  - F: pairs matching on the *same* wrong answer;
  - L: correct responses denied credit by a wrong label.
- **Arms.** Dr. Zero (coupled), MSV, CrossFit (25 auxiliary updates per
  fold per round) and MSV + CrossFit. Reproduced baselines (Prompting,
  R1-Instruct, Search-R1) use the same backbones and the same 1,325-question
  evaluation set: 200 each from NQ, TriviaQA, PopQA, HotpotQA, 2WikiMQA and
  MuSiQue, plus 125 from Bamboogle. Each question gets one greedy
  trajectory, scored by Cover-EM.
- **Fixed-bank replay (§5.2–5.3).** 3,000 saved questions and adopted
  labels are held fixed while the feedback solver's training provenance
  varies. There are 8 conditions, including same-source auxiliary,
  full-data auxiliary, random-row OOF, source-ID OOF, half-budget
  source-ID and the MSV crosses. Results are mean ± SD over **5 seeds**.
- **Cost (Table 3).** All costs are reserved H200-hours.
  - Dr. Zero: 379 (4B) / 476 (9B).
  - MSV: 719 / 903.
  - CrossFit, 25 per fold: 650 / 854.
  - CrossFit, 25 total: 515 / 665.
  - MSV + CrossFit: 990 / 1,281, which is 2.6–2.7× Dr. Zero.

## Results

- **Live-loop audit, round 3 (Table 4):**

  | Arm | T_P 4B / 9B | T_S 4B / 9B | A 4B / 9B | F 4B / 9B | L 4B / 9B |
  |---|---|---|---|---|---|
  | Dr. Zero | .747 / .737 | .669 / .689 | .710 / .752 | .061 / .088 | .020 / .025 |
  | MSV | .747 / .770 | .708 / .727 | .745 / .788 | .057 / .072 | .020 / .011 |
  | CrossFit | .819 / .851 | .686 / .712 | .679 / .709 | .030 / .037 | .038 / .041 |
  | MSV + CrossFit | .843 / .878 | .727 / .754 | .708 / .732 | .020 / .017 | .038 / .039 |

- **Coupled dynamics (Fig. 5a).** At 4B, in-loop agreement rises from
  about 62% to 71% across rounds, while solver truth falls from about 74%
  to 67%. Round 3 sits above the A = T_S diagonal, so feedback is
  "optimistic". All arms share round 1.
- **Downstream average Cover-EM (Table 1):**

  | | 4B | 9B |
  |---|---|---|
  | Base | 0.384 | 0.409 |
  | Search-R1 | 0.401 | 0.434 |
  | Dr. Zero | 0.400 | 0.428 |
  | MSV | 0.407 | 0.436 |
  | CrossFit | 0.488 | 0.512 |
  | MSV + CrossFit | 0.491 | 0.515 |

  Gains are 10.0 / 10.9 points on multi-hop tasks against 7.3 / 5.2 on
  single-hop. CrossFit's lead over Dr. Zero grows from 4.2 / 4.3 after
  round 2 to 8.8 / 8.4 after round 3. Micro averages order the arms the
  same way.
- **Fixed-bank replay (Fig. 7, §5.2–5.4), F at 4B / 9B:**
  - coupled 0.058 / 0.073;
  - same-source auxiliary 0.064 / 0.087;
  - full-data auxiliary 0.058 / 0.069;
  - random-row OOF 0.050 / 0.062;
  - **source-ID OOF 0.004 / 0.001**;
  - half-budget source-ID 0.005 / 0.002;
  - coupled + MSV 0.043 / 0.056;
  - source-ID + MSV 0.005 / 0.000.

  Replay accuracy for source-ID OOF goes from 88.1% to 91.5% (4B) and from
  87.0% to 91.7% (9B). Probe solver truth goes from 0.687 to 0.770 and
  from 0.717 to 0.868.

## Critique / open questions

- **The digest's 0.4% / 0.1% comes from a different experiment.** Both the
  abstract ("further reduces false agreement to 0.4% and 0.1%") and the
  digest place it after the live-loop 3.0% / 3.7%, which reads as a third
  step down. It is actually the fixed-bank replay, whose own coupled
  baseline is 5.8% / 7.3%, not 6.1% / 8.8%. In the live loop the best
  result is **MSV + CrossFit at 2.0% / 1.7%**. CrossFit alone stalls at
  3.0% / 3.7%, which is roughly 8–37× the replay floor. The paper does not
  explain that gap. Its own limitations section suggests one: the
  adaptive proposer and shared pretraining re-correlate errors that a
  frozen bank cannot.
- **"+8–9 points" is the Dr. Zero comparison.** Against Search-R1, the
  supervised baseline, the gains are +8.7 (4B) and **+7.8** (9B). The
  reproduced Dr. Zero is slightly *below* reproduced Search-R1 at both
  scales (0.400 vs 0.401, 0.428 vs 0.434), and only 1.6 / 1.9 points above
  the untrained Base. A weak coupled baseline makes the CrossFit gap look
  larger.
- **The live-loop numbers are one training run per arm per scale.** The
  appendix says the round summaries are "arithmetic means of the 43
  displayed step rates … not additional runs or uncertainty estimates."
  Table 1 has no seeds, no CIs and no training-run variance. A gap of
  about 8 points on 1,325 items is well beyond eval-sampling noise, but
  run-to-run RL variance is unmeasured. Only the replay has seeds.
- **The downstream gain is not tied to the mechanism.** The replay
  isolates "less F" but never measures downstream accuracy. CrossFit also
  changes the curriculum: label truth T_P rises by 7.2 / 11.4 points even
  though CrossFit never touches labels, because the proposer is now
  writing different questions. Solver truth on those questions moves only
  1.8 / 2.2 points (Fig. 6). So "less co-cheating → +8.8" rests on a
  single-run trajectory correlation. The proposer's curriculum shift is
  an equally live explanation.
- **CrossFit trades false agreement for lost credit.** Lost credit L
  roughly doubles (0.020 → 0.038 and 0.025 → 0.041), because a half-data
  auxiliary solver disagrees more often. Total label–response errors
  (F + L) fall only from 0.081 to 0.068 at 4B and from 0.113 to 0.078 at
  9B. The proposer's signal swaps one kind of noise for another, and the
  swap is net-positive mostly at 9B.
- **The whole diagnosis rests on one unvalidated LLM auditor.** F, T_P
  and T_S all come from `gpt-6-astra/high` judging against a reference it
  builds itself. The paper reports no human agreement check, no second
  auditor and no recall estimate. This is the same single-learned-check
  weakness ding2026autonomous puts at Tier V.
- **"Internal reward rises" needs care.** What rises is the solver-side
  agreement A. The proposer's reward f(k) is *non-monotone* in agreement:
  it peaks at k = 1 and is zero at k = 5. Figure 1's "training signal" is
  A.
- **The "increasingly severe" trend has 3 points.** Each is one run, and
  round 1 is shared by every arm.
- **The digest understates MSV in one place.** At 9B, MSV cuts F by 1.6
  points (18% relative) and raises T_P by 3.3 points, which is more than
  "barely". MSV + CrossFit gives the lowest live F, and the digest omits
  it. On downstream accuracy, however, "barely" is fair: +0.7 / +0.8.
- **Running-finding check: it holds, so 32/32.** The most striking number
  in the digest (0.4% / 0.1%) comes from the most favourable cut, a
  frozen replay with its own baseline, and is presented as a continuation
  of the live-loop result. "+8–9" is taken against the weaker of the two
  baselines. MSV is the one place where the digest is less generous than
  the paper.
- **Domain transfer.** This is RL training of a QA search agent, not an
  ML-research agent. The transferable part is the replay ablation:
  evaluator independence comes from **source ancestry**, not from having a
  separate evaluator or more evaluator compute.

## Trust signals

- **Credibility:** 2, at the upper end of 2. Against the paper:
  - 10 of 15 authors list "Independent Researcher". The academic
    affiliations are Rutgers (first author), UCSD, UMich, McGill and
    KFUPM, the last two corresponding.
  - It is an unreviewed preprint with no code or artifact URL. The
    reproducibility statement points only to the appendix.
  - There is one training run per arm.
  - The headline metrics come from a single LLM auditor with no human
    validation.
  - The abstract juxtaposes non-comparable numbers.

  It is not lower because the replay controls are well designed and
  seeded: same-source vs full-data vs random-row vs source-ID, plus a
  half-budget control for "more compute". The cost table is unusually
  complete (H200-hours, tokens, judge calls, wall-clock). The authors
  state their limits plainly, including the difficulty-filtering
  confound on F.

## Follow-up

- **Relevance:** 4. It gives [[concepts/shared-substrate-contagion]] a
  within-training-loop instance with a seeded ablation that pins the
  channel. The shared substrate is the **source document**, and an
  evaluator's independence is set by whether it learned from that source.
  Evaluator identity and evaluator compute do not set it. It also
  supplies [[concepts/information-firewall]] with a new boundary, the
  evaluator's training ancestry, plus a measured result that the
  boundary must be drawn at the source level rather than the row level.
  It is not a 5: one run per arm, an unvalidated auditor, and a domain
  (QA RL) one step removed from research agents.
- **Answers part of li2026benchmark's open point.** Source-ID exclusion
  is a concrete mechanism for *evaluator* independence. Its residual
  (live F 3.0 / 3.7 against a replay floor of 0.4 / 0.1) is the same shape
  as li2026benchmark's F6. Lineage, or here fold assignment, stops copies
  of a source-specific error. It does not stop errors that are correlated
  through shared pretraining.
- **Repo read.** Concurrent `/ingest` agents reading the same PDF with the
  same model are the "full-data auxiliary" arm here: a separate evaluator
  with the same ancestry, and that arm gave no reduction (0.058 vs 0.058).
  A genuinely independent check would need a reader that has *not* seen
  the source-derived note, matching the no-channel suggestion in
  shared-substrate-contagion's open questions.
- **Candidates (cited, not in this graph):**
  - Yue et al. 2026, "Dr. Zero: Self-evolving search agents without
    training data" (arXiv 2601.07055, COLM 2026);
  - Tan et al. 2026, "SearchMaster: Grounded and Regulated Self-Play for
    Search Agents" (arXiv 2608.01822), described as the closest
    complementary diagnosis;
  - Lu et al. 2026, "Search Self-Play";
  - Huang et al. 2025, "R-Zero" (arXiv 2508.05004);
  - Liu et al. 2026b, CAFE (co-evolving feedback).
