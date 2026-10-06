---
kind: paper
title: "PEAR: Progressive Evidence-Based AutoResearch for Industrial Search Systems"
authors: ["Yifan Wang", "Shipeng Zhu", "Fei Xiong", "Yuqin Yang", "Yonghui Huang", "Kunyao Wu", "Yue Wang", "Weichao Meng", "Yu Gong"]
institutions: ["ByteDance (Global E-Commerce Agentic Search Team; ByteDance logo on page 1, author emails are gmail addresses)"]
year: 2026
venue: "arXiv 2609.35031v1 (cs.IR), 2026-09-28"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.35031"
code_url: null
citations: null
source: "raw/papers/wang2026pear.pdf"
added: "2026-10-06"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/structured-world-model]]"
tags: ["autoresearch", "industrial-search", "e-commerce", "keep-if-better", "non-stationarity", "concurrent-baseline", "confidence-interval-gate", "promote-retain-stop", "multi-fidelity", "verifier-ladder", "shadow-traffic", "online-ab-test", "proxy-reward-model", "hypothesis-ledger", "winner-selection", "multiplicity", "noise-band", "production-deployment"]
---

# PEAR: Progressive Evidence-Based AutoResearch for Industrial Search Systems

## TL;DR

A ByteDance e-commerce search team runs an LLM AutoResearch loop over
**strategy-level ranking parameters**, such as result-density windows and
position-dependent exposure settings. The paper describes two components.

- **Evidence-driven AutoResearch.** Each strategy task keeps its own research
  state: context, hypotheses, intervention space and evidence. The state
  moves through Plan → Execute → Evaluate → Update. Hypotheses are kept in a
  ledger with Support/Refute/Neutral counters.
- **Confidence-Gated Verifier Ladder.** There are four levels: offline
  replay (L1), shadow traffic (L2), rapid online A/B (L3) and decision-grade
  online A/B (L4). The **same** three-way rule applies at every level. A
  candidate is PROMOTEd if the lower 95% CI bound is above 0, gets a STOP if
  the upper bound is below 0, and otherwise RETAINs at its level for more
  evaluation. Every comparison is against a concurrent baseline.

In the reported runs, L1 was never used (regional data-access rules). The
L2 screen ran 454 experimental groups over three tasks and promoted 13.
Two strategies are reported with significant Main Order/DAU gains at L4:
+2.7336% for Task A and +3.2957% for Task B.

For this graph, the value is a **concrete specification** of the
interval-based keep rule that [[literature/papers/park2026when]] left open,
plus one structural idea: answer non-stationarity with a concurrent
baseline in every round, not a stored champion score. The paper has **no
comparison arm**. Nothing tests the gate against keep-if-better, against no
ladder, or against the human engineers. So it shows that the design was
deployed and found two winners. It does not show that the significance gate
is what found them.

## Claims

- **Keep-if-better assumes comparable scores across rounds.** "Under
  non-stationary traffic, transient gains may be mistaken for persistent
  improvements, impairing reliable accumulation of search knowledge."
- **A unified CI rule across heterogeneous evaluators.** The ladder "applies
  a common confidence-based promotion rule" and keeps each level's signal
  separate "rather than combining heterogeneous results into a single
  score."
- **Cheap levels screen, expensive levels decide.** "Lower-cost evaluations
  support broad exploration, while costly online experiments are reserved
  for promoted strategies."
- **Online evidence redirects the search.** In Task B, L4 failures in
  Phases I–II led to revised hypotheses and a wider Phase III range, "one
  of which subsequently achieves a significant improvement at L4."
- **Proxy screening is directionally valid.** Two traced candidates show
  "directional consistency from proxy-based screening to Decision-Grade
  Online Evaluation". The authors call this "illustrative evidence".
- **Headline.** PEAR strategies "significantly increased Main Order/DAU by
  2.7336% and 3.2957% relative to their respective baselines in two A/B
  experiments."

## Methods

- **Research state** H = (C, M, Ω, D). C is the protocol and scope, M the
  hypotheses, Ω the candidate intervention space and D the evidence linked
  to hypotheses. There is one state per strategy task, and tasks run
  concurrently.
- **Hypothesis ledger (App. B.1).**
  - The agent adds at most 3 new hypotheses per round, and each hypothesis
    gets at most one verdict per round.
  - Score = Support − Refute. A score ≥ 2 makes the hypothesis active, ≤ −2
    marks it risky, and superseded hypotheses are archived.
  - Each hypothesis carries a **strategy-version fingerprint**, and it is
    reused only on a version match.
  - The authors say the scores "are not verifier confidence estimates."
    The verdicts are LLM-assigned.
- **Lessons and memory (B.2–B.3).**
  - Lessons are appended per round. Each has an outcome of
    keep/discard/iterate/pivot/crash.
  - "When summaries disagree with measured evidence or experimental
    records, correct the summaries."
  - The final decision is based "on the confidence-evaluation record, not
    legacy score logs".
  - `best_selected_by: llm_from_confidence_pipeline`, meaning an LLM
    picks the round's best from the CI records.
- **Candidate generation (B.4).** The paper calls this
  "Bayesian-optimization-inspired" reasoning, but it is **prompt-level**:
  - The LLM labels the search state as cold_start, local_refine, stagnant
    or pivot.
  - It reasons in a GP-, TPE- or SMAC-inspired style, with no fitted
    surrogate.
  - It builds a pool at least 2× the budget, and increases exploration when
    stagnant.
  - Exact replications come from a separate `confirmation_queue`, kept
    apart from new candidates.
- **Gate (§5.2, App. A).**
  - L1 uses a paired t-test over shared historical requests.
  - L2–L4 use a Welch t-test over independent groups.
  - The two-sided CI is at α = 0.05 by default, and the PROMOTE/RETAIN/STOP
    rule is identical at every level.
  - L1 and L2 score ranked lists with a learned list-level reward model R_ψ.
    L3 and L4 use observed user outcomes.
  - L1–L3 take hours; L4 takes days.
- **Humans gate exposure.** "Before any candidate is exposed to real-user
  traffic, the proposed change and supporting evidence undergo human review
  and require explicit human confirmation" (§9). So L3 and L4 promotions are
  human-approved.
- **Unstated:** the LLM, the traffic sizes, the L4 windows, the L4 CIs,
  and the unit of analysis for the ratio metric.

## Results

- **L2 search scale (Table 1).** Promotion rate = promoted ÷ groups.

  | Task phase | Rounds | Groups | Unique configs | Promoted from L2 | Rate |
  |---|---|---|---|---|---|
  | A | 19 | 133 | 20 | 2 | 1.50% |
  | B–I | 12 | 84 | 23 | 5 | 5.95% |
  | B–II | 2 | 14 | 6 | 1 | 7.14% |
  | B–III | 21 | 147 | 53 | 2 | 1.36% |
  | C | 16 | 76 | 54 | 3 | 3.95% |

  Totals (our sum): 70 rounds, 454 groups, 156 unique configs and 13
  promoted. That is 2.9% of groups, or 8.3% of unique configs.
- **Task B at L4 (Table 3).**
  - Phase I: 5 promoted, "No significant gain".
  - Phase II: 1 promoted, "Positive, not significant".
  - Phase III: 2 promoted, one with a "Significant order gain".
  - So **1 of 8** Task B candidates sent to L4 cleared.
- **Task A trajectory (Table 2).** G = (window, capacity), baseline (2, 1).
  - Round 1: only (3, 2) gives a significant order signal.
  - Round 2: replications of (3, 2) give "mixed directional effects".
  - Round 3: capacity-only (2, 2) is strongest.
  - Round 4: (3, 3) is "directionally favorable … but is not significant".

  This is the paper's only concrete instance of a gain that failed to
  replicate and was caught.
- **Proxy calibration (Table 4).** The paper compares aggregate means only,
  not candidate effects. Signed relative errors are click −16.02%, order
  −3.49% and GMV −18.23%.
- **Proxy → online, two Task A candidates (Table 5).**

  | Candidate | L2 order proxy | L3 observed order | L4 Main Order/DAU |
  |---|---|---|---|
  | A | +45.88% | +15.48% | +2.73% |
  | B | +61.64% | +29.76% | +2.16% |

- **L4 results (Table 6).** Bold = p < 0.05.

  | Task | SearchPV/DAU | ASN | GMV/DAU | Main Order/DAU | SKU Order/DAU | PayPV/PV | Main OPMS | SKU OPMS |
  |---|---|---|---|---|---|---|---|---|
  | A | +0.0063% | **+1.1512%** | +2.2136% | **+2.7336%** | **+2.7259%** | +1.0441% | **+2.7407%** | **+2.7375%** |
  | B | +0.0470% | **+1.3841%** | +1.6633% | **+3.2957%** | **+2.9826%** | **+1.6298%** | **+3.2595%** | **+2.9448%** |

  **GMV/DAU is not significant for either task.**
- **Task C.** Three candidates were promoted from L2. **No L3 or L4
  outcome is reported.**

## Critique / open questions

- **The paper reads below its digest framing, and the usual cause applies.**
  The two headline numbers match the PDF exactly. They are also the
  favourable cut:
  - **Task B** reports the 1 of 8 L4 candidates that cleared.
  - **Task A** reports the better of the 2 candidates at L4. Candidate B's
    +2.16% has no stated significance.
  - **Task C** has 3 L2 promotions and no reported outcome.
  - **GMV/DAU**, the revenue metric, is not significant for either winner.
  - **No CIs or p-values** are given for any L4 effect, only boldface.

  If the 8 Task B candidates had no true effect, the chance that at least
  one would clear a one-sided 2.5% bar is about 1 − 0.975⁸ ≈ 18% (our
  arithmetic). The candidates were pre-screened, so the true prior is
  better than null. Even so, the reported magnitude is a selected maximum,
  and nothing in the paper replicates it.
- **"The gate gets stricter as evaluations get more expensive" is not in the
  paper.** It applies one rule at one α (0.05) at every level. Higher
  levels are more trustworthy only because the signal is real outcomes over
  longer windows, not because the bar rises. A two-sided 95% CI at every
  level also means each level's false-pass rate on a null candidate is
  2.5%. The ladder's protection against noise comes from having to pass
  *successive independent* tests, which is a product of per-level rates.
  The paper never states this.
- **Keep-if-better is never run as a comparison.** The motivating claim, that
  keep-if-better under non-stationarity mistakes transient gains for
  persistent ones, is argued and not measured. There is:
  - no keep-if-better arm and no no-ladder arm;
  - no human-engineer baseline, although humans approve every exposure;
  - no count of how many L2 "wins" a bare `>` rule would have promoted.

  The one supporting datum is Task A round 2, a gain that failed to
  replicate. It shows the value of **replication against a concurrent
  baseline**, which is ordinary A/B practice (Kohavi et al., cited). It
  does not show the value of the significance gate specifically. As a
  "third independent source that keep-if-better needs a significance gate"
  after [[literature/papers/chandran2026autoresearch]] and
  [[literature/papers/qu2026propose]], it counts as a design endorsement
  with no data behind it. qu2026propose remains the only one of the three
  with a measured false-admission cost.
- **The actual non-stationarity fix is the concurrent baseline, and the
  abstract undersells it.** Every evidence record carries its own
  stage-specific baseline b_ℓ, and B.3 says "Retain one baseline per
  round." Comparing against a concurrent control is what makes round t and
  round t+1 comparable. The CI rule only sizes the uncertainty around that
  comparison. For `/iterate` the transferable form is this: re-run the
  champion next to the challenger in the same round, rather than comparing
  against the champion's stored score.
- **RETAIN without a stopping rule amounts to optional stopping.** An
  inconclusive candidate "is retained for further evaluation at the current
  stage", and nothing says how many re-looks are allowed or whether α is
  corrected. qu2026propose measured this exact pattern (re-test until
  p ≤ α) at 86.2 vs 11.7 false admissions per campaign. Across 454 L2 groups
  with RETAIN re-looks, the L2 false-promotion rate is not 2.5%. The paper
  does not report it.
- **The proxy inflates and loses rank order.** In Table 5 the L2 effects
  are 17× and 29× the L4 effects. Candidate B leads at L2 (+61.6 vs +45.9)
  and at L3 (+29.8 vs +15.5), then trails at L4 (+2.16 vs +2.73). The paper
  reads n = 2 as "directional consistency". It equally shows that the proxy
  ranks candidates unreliably. Table 4 does not help: it tests aggregate
  means in a separate experiment, and it bolds the best of three
  components (order, −3.49%) while click and GMV are off by 16–18%.
- **The L2 units may be pseudo-replicated.** L2 runs a Welch t-test over
  independent shadow groups of *requests*. Requests from the same user are
  correlated, so a request-level SE understates the variance and makes
  PROMOTE too easy. Main Order/DAU is a ratio metric, and the paper does not
  say whether L4 used a delta-method or user-level variance.
- **Autonomy is partial and the agent is not identified.** Every real-user
  exposure needs human sign-off, and the LLM is never named. How much of
  the Phase III redirection in Task B came from the agent, and how much from
  the reviewing engineers, cannot be separated.
- **The search space is narrow and the "BO" is a prompt.** The interventions
  are a few integer ranking parameters, e.g. G = (w, q) ∈ small grids. The
  "Bayesian-optimization-inspired" generation is prompt text with no
  surrogate, and it is unablated. The stagnation response ("increase
  exploration") is a prompt instruction too. [[literature/papers/yan2026traceml]]
  found that prompt-level anti-stall checks go with *less* mode switching.
- **No artifacts.** There is no code, the prompts are summarised "omitting
  deployment-specific identifiers", and the tasks are anonymised. The
  system cannot be reproduced outside ByteDance's traffic, which the authors
  acknowledge in their limitations.

## Trust signals

- **Credibility:** 3, at the low end. In favour:
  - a major industrial lab (ByteDance branding; team-name byline);
  - production A/B tests at p < 0.05 on live traffic, which are real
    measurements of real systems;
  - the gate math is specified in full (App. A), and the hypothesis and
    lesson schemas are given (App. B);
  - Task B's L4 failures are reported, which a pure success story would
    have left out.

  Against it:
  - an arXiv preprint with no peer review;
  - no code, an unnamed LLM, anonymised tasks and no CIs on headline
    effects;
  - Task C's L4 outcome is missing;
  - no comparison arm for any methodological claim.

  The numbers are probably accurate. What they show about the method is
  thin.

## Follow-up

- **Relevance:** 3, below the digest's 4.
  - The paper supplies a **specification** that fits the pending
    2026-09-27 `iterate-no-improvement-noise-band` proposal well: the
    three-way CI rule, a concurrent baseline per round, a separate
    replication queue, and a version-fingerprinted hypothesis ledger.
  - It supplies **no evidence** that the gate beats keep-if-better. The
    domain is production search ranking, not ML research.
  - Under the rubric, 4 requires "material new evidence/ablations", and
    there are no ablations.
- **For the noise-band proposal:** the paper supports the band's design but
  provides no evidence for it.
  - It offers a concrete alternative to a std band: **PROMOTE / RETAIN /
    STOP on a CI against a concurrently re-run champion.** RETAIN is the
    useful extra state. "Not yet distinguishable" becomes a reason to spend
    more seeds, not a reason to keep or discard.
  - RETAIN needs a cap on re-looks, or a pre-committed single look, to
    avoid the peeking cost qu2026propose measured. The paper gives no
    such cap.
  - With n = 3 seeds the Welch df is about 2–4, so t ≈ 2.8–4.3. That gate
    will RETAIN nearly every sub-2-SD change. Without a seed budget the
    gate collapses into "never keep". This is our arithmetic.
- **A possible concept, not seeded:** a *multi-fidelity promotion ladder*.
  Successive cheap-to-expensive evaluators each gate on their own CI
  against their own concurrent baseline, and no heterogeneous score is
  ever pooled. The graph has no multi-fidelity concept, and the ML-loop
  analogue is clear: short-schedule proxy run → full run → held-out seeds.
  One unablated source is not enough. Revisit if a second source measures
  ladder vs single-stage.
- **Candidates:**
  - Wang et al. 2026, "Self-Evolving Recommendation System" (arXiv
    2602.10226): a fast offline loop plus a slow online loop.
  - Lao et al. 2026, AgentX (arXiv 2606.26859): agent-driven self-iteration
    of industrial recommenders with online feedback.
  - Cheng et al. 2026, Sortify (arXiv 2603.27765): estimates the
    offline-to-online relationship, which is the proxy-calibration question
    PEAR leaves at n = 2.
