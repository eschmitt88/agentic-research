---
kind: paper
title: "Trustworthy Recursive Self-Improvement with Statistical Guarantees"
authors: ["Zihe Gao", "Zhengling Qi", "Yifan Cui"]
institutions: ["Zhejiang University", "George Washington University"]
year: 2026
venue: "arXiv 2611.10787 (stat.ML), v1 2026-11-07, preprint (ICLR-style template, 9-page body + 35 pages of appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.10787"
code_url: null    # App. H.2 says "the full evaluation procedure can be found in the code", but the PDF gives no link
citations: null
source: "raw/papers/gao2026trustworthy.pdf"
added: "2026-11-16"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: ["acceptance-gate", "recursive-self-improvement", "harness-evolution", "false-evolution-rate", "online-fdr", "e-values", "e-lord", "e-gai", "noninferiority", "validation-reuse", "overlap-correction", "llm-allocator", "anytime-valid", "optional-stopping", "max-drawdown", "greedy-gate", "pace", "sgm", "noise-band", "hotpotqa", "drop", "apps", "deepseek-v4-flash", "qwen-3.8-flash"]
---

# Trustworthy Recursive Self-Improvement with Statistical Guarantees

## TL;DR

A self-improving agent edits its own harness (rules, retrieval routing,
memory, workflow) and a **gate** decides whether each edit replaces the
incumbent. TRUST turns the gate into an online multiple-testing problem.
Each round tests the noninferiority null "the candidate is worse than the
incumbent by at least ε" with a sequential e-value built from paired
per-item score differences. The e-value is corrected for validation items
reused from earlier rounds. A budget allocator (e-LORD inside e-GAI) sets
each round's testing level from a total budget q, and an LLM reading the
history scales each round's share of that budget. The paper proves that the
**false evolution rate** (FER, the expected share of accepted updates that
lose more than ε) stays at or below q. It also proves a true-acceptance
consistency result and a lower bound on final gain. Over 30 rounds × 5
seeds on HotpotQA, DROP and APPS with two cheap models, TRUST's mean
empirical FER0 is 2.78% against Greedy's 45.59%. Mean drawdown is 0.047 pp
against 4.963 pp, and mean final gain is 24.84 pp, the highest of five
gates. Two things qualify that. TRUST's gain lead over Greedy is 1.2 pp on
average, and Greedy is ahead in 2 of 6 settings. And a per-round e-process
gate with no cross-round control (PACE) had an even lower FER0 (1.11%).

## Claims

- A greedy gate (accept if the validation score rises) mistakes finite-
  sample noise and validation reuse for progress. Gates that control the
  family-wise error across rounds (SGM) are so conservative that they
  rarely accept anything.
- FER is the right criterion for a long RSI run: an FDR analogue that
  tolerates a bounded share of bad acceptances in exchange for power.
- **Theorem 2:** under Assumption 1, FER_ε(T) ≤ q. The bound also holds at
  any stopping time of the outer loop, so a data-dependent halt keeps it
  (App. C.2). "The FER guarantee does not rely on the LLM correctly
  identifying beneficial candidates."
- **Theorem 3:** under Assumption 2 (n, N → ∞, n/N → 0, bounded betting
  fractions below 2ε/(1+ε)), TER(T) ≥ 1 − r_{N,n} − T·e^{−κn}. With ε = 0
  no admissible betting fraction exists, so a positive margin is needed for
  power.
- **Theorem 4:** the final gain is at least the proposals' total positive
  effect, discounted by e^{−κn}, minus two accepted-loss terms. One is
  min{loss, ε} per accepted update; the other is α_t times the loss beyond
  ε. Expected maximum drawdown has the same bound (App. E).
- Context-guided allocation makes the gate better at spotting beneficial
  updates while keeping FER control.

## Methods

- **Loop.** One candidate per round. The proposer emits three edits from
  "closed" feedback; they are screened on 20 examples from the previous
  round, and the best is gated (App. G.3). T = 30 rounds. The harness has
  four editable components and 20 operators (App. F). They include adding,
  refining, generalizing and deleting reasoning rules, retrieval top-K and
  fallback, adding up to three closed cases to episodic memory or deleting
  them, and verify/stop/fallback workflow modes.
- **Data.** Per dataset: 20 examples for initial feedback, a 200-example
  validation pool, and 300 held-out test examples used only for reporting.
  Each round draws up to n = 40 validation items without replacement from
  the pool, so items recur across rounds.
- **Test.** Paired differences D ∈ [−1, 1] are mapped to Y = (1 + D)/2.
  The null boundary is p_ε = (1 − ε)/2. The boundary for a reused draw is
  raised by ζ·β, where ζ is the chance the draw was seen before. Evidence
  is a product of betting factors 1 + η(Y − u)/u, with an equal mixture of
  η ∈ {0.10, 0.25, 0.50, 0.75}. The gate accepts at the first m ≤ 40 with
  M ≥ 1/α_t.
- **Allocation.** α_t = min{α_max, ρ_t·B_t·(R_{t−1} + 1)}, the charge is
  α_t/(R_{t−1} + 1), and ρ_t = clip(w_t/(T − t + 1), ρ_min, ρ_max). Here
  w_t is the LLM's multiplier from a G-Eval-style rubric covering safety,
  contract, actionability, generalization and scope (App. H.2). The PDF
  reports neither ρ_min, ρ_max and α_max nor the map from rubric scores to
  w_t.
- **Settings.** q = 0.30, ε = 0.10, β = 0.01. Solver and allocator run at
  temperature 0, the proposer at 0.3 (0.7 for HotpotQA/DeepSeek). Cached
  responses are reused for identical inputs. Seeds: solver-output seeds
  0–4 with data-split seeds 42–46.
- **Baselines.** Greedy (positive paired gain); PACE (bet 0.5,
  candidate-level α = 0.05, no cross-round control); SGM-B and SGM-H
  (Bernstein / Hoeffding lower confidence bound > 0, FWER across rounds,
  δ = 0.5).
- **Metrics on the test set** (App. G.2). FER0 counts accepted updates with
  test Δ̂ ≤ 0, ties included, so it is stricter than the bounded FER_0.10.
  TER is the share of test-improving candidates accepted. MDD is the
  largest peak-to-trough fall of the incumbent's test score.
- **Starting harness.** "Contains prespecified weaknesses that provide
  clear opportunities for improvement." The HotpotQA base protocol, for
  example, says to return the first displayed title as the answer and skip
  bridge reasoning (Listing 14).

## Results

- **Main table (Table 1, mean ± SE over 5 seeds).**

  | Setting | Greedy gain / FER0 | PACE gain / FER0 | TRUST gain / FER0 |
  |---|---|---|---|
  | HotpotQA DS | 7.87 / 64.67 | 10.15 / 0.00 | 11.58 / 10.00±10.00 |
  | HotpotQA Qwen | 20.66 / 44.67 | 26.42 / 6.67 | 27.72 / 0.00 |
  | DROP DS | **36.24** / 35.28 | 29.70 / 0.00 | 31.47 / 0.00 |
  | DROP Qwen | 15.15 / 37.38 | 13.22 / 0.00 | 17.22 / 0.00 |
  | APPS DS | 33.89 / 41.08 | 28.68 / 0.00 | 34.55 / 0.00 |
  | APPS Qwen | **28.03** / 50.44 | 26.24 / 0.00 | 26.53 / 6.67±6.67 |

  Means (our arithmetic): gain 23.64 Greedy, 22.40 PACE, 24.84 TRUST.
  FER0 45.59, 1.11 and 2.78%. TER 50.6, 8.4 and 14.3%. SGM-B never
  accepted an update. SGM-H accepted three in all its runs.
- **Drawdown.** Greedy's MDD is 3.37–8.48 pp; TRUST's is at most 0.27 pp.
  Figure 4: Greedy gains fast and then declines, and TRUST overtakes it
  late.
- **Ablations, HotpotQA/DeepSeek only (Tables 7–9).**
  - q = 0.1 / 0.3 / 0.5: gain 6.82 / 11.58 / 11.33, FER0 0 / 10.00 / 18.00.
  - ε = 0 / 0.05 / 0.10 / 0.15: gain **0.00** / 4.87 / 11.58 / 11.59, FER0
    0 / 0 / 10.00 / **57.68**, MDD 0 / 0 / 0.02 / 5.33. At ε = 0 TRUST
    accepts nothing.
  - β = 0 / 0.01 / 0.03 / 0.05: gain 10.97 / 11.58 / 7.13 / 2.80, FER0
    23.33±14.53 / 10.00±10.00 / 0 / 0.
- **LLM allocator (Fig. 6).** Final score 44.91 ± 1.36% with the LLM and
  43.78 ± 1.18% with w_t ≡ 1. The App. H.2 case is one seed (seed 3). There
  the LLM's "very_conservative" posture rejects a round-2 candidate that the
  w ≡ 1 arm accepts, and the seed finishes at 47.22 against 43.95.
- **Greedy case study (App. G.4, APPS).** An edit to stop at "first
  example pass" won 4 validation items and lost 2 of 40 (+0.04), so Greedy
  accepted it. On the 300 test problems it fell from 75.07% to 67.00%.
- **Overfitting stress test (App. I, Greedy only).** The same 20 examples
  are reused for proposing and validating, over 10 rounds of routing and
  memory edits. Validation gain is +24.44 pp; test gain peaks at +5.06 and
  ends at −2.15. One accepted memory stores a validation question with its
  verified answer. On the test set it is retrieved by a shared keyword,
  routing then truncates a needed paragraph, and Joint F1 falls from 54.55
  to 48.00 (13 items down, none up).

## Critique / open questions

- **Against the digest framing: at, with two corrections.** The headline
  numbers are in the PDF and quoted correctly. The digest asked for the
  run count: 30 rounds × 5 seeds in six settings, with ablations on one
  setting at 5 seeds. Two readings need fixing.
  1. **"This is the noise band as a formal procedure" overstates the
     match.** `/iterate`'s band (skills-experiment/iterate/SKILL.md, "What
     counts as an improvement") is a **stop-counter** rule built from
     seed std. It decides whether a cycle resets
     `--consecutive-no-improvement`. It does not decide what is kept: the
     next cycle seeds from the most recent implemented experiment. TRUST is
     a **keep gate** on paired per-item outcomes. They meet only if
     `/iterate` ever grows a keep rule, which
     [[concepts/budget-as-ceiling]]'s 10-12 section argues against at
     current sample sizes.
  2. **The cross-round control is proven, not shown to be what lowers
     false acceptance.** PACE, a per-round test at α = 0.05 with no
     cross-round budget, had a lower mean FER0 than TRUST (1.11 vs 2.78%).
     What TRUST adds over PACE in the data is power (TER 14.3 vs 8.4%,
     +2.4 pp gain).
- **The margin probably does most of the work, and the baselines lack it.**
  TRUST tests noninferiority at ε = 0.10, ten points on a [0, 1] score.
  SGM tests superiority (lower bound > 0), and the PDF does not state
  PACE's null. At ε = 0, TRUST also accepted nothing (Table 8), like SGM-B.
  Theorem 3 says the same: power needs ε > 0. So "TRUST beats the FWER
  gates" mixes two changes, the error criterion (FER vs FWER) and the null
  (noninferiority vs superiority). No baseline is run with the same margin.
- **What ε = 0.10 allows (our arithmetic).** Under the null boundary
  p_ε = 0.45, a tied item (Y = 0.5) counts as evidence *for* acceptance.
  Forty ties give the paper's betting mixture an e-value of about 9.5
  (about 5.5 once most of the pool has been seen). With w_t ≡ 1 the budget
  recursion gives α_t = (q/T)·(R_{t−1} + 1) = 0.01·(R + 1). The first
  threshold is therefore 100, and a pure no-op cannot pass it. After about
  10–18 acceptances, or with an LLM multiplier of 3 after three or four
  (if ρ_max and α_max allow it; the PDF gives neither), it can. That is by design: a no-op is "tolerated", not a false evolution.
  But a gate that admits do-nothing edits, and whose later thresholds
  loosen with every admission, is not a keep-only-if-better gate. The
  empirical FER0 (ties count as failures there) suggests this rarely
  happened in these runs.
- **Can the LLM step loosen the proven bound? No, if two conditions hold
  in code.** The Theorem 2 proof (App. C.1) uses only two facts. α_t is
  fixed before the round's validation draws (F^pre_t-measurable). And
  Σ α_t/(R_{t−1} + 1) ≤ q, which holds because ρ_t is clipped below
  ρ_max < 1 and charged against the remaining budget. w_t's accuracy never
  enters. So the allocator can only move budget between rounds. The
  conditions are that the allocator never sees the current round's draws
  and that the clip and the budget ledger are enforced in code, not left
  to the LLM. What the LLM *can* worsen is everything except FER:
  - The gain and MDD bounds (Thm. 4): their third term is
    Σ α_t·(loss beyond ε), so generous levels on harmful candidates enlarge
    it.
  - Power: front-loaded spending leaves later rounds starved down to the
    ρ_min floor.
  - The no-op channel above: a large w_t in a round with many prior
    acceptances lowers that round's threshold toward the tie e-value.
  The levers that really loosen the guarantee are the human-set ones. ε
  and q define it (ε = 0.15 gave FER0 57.68%, which the theorem allows,
  since it bounds only losses beyond 15 points). β is an *assumed* bound
  on reuse optimism. "The present study does not address how to identify
  an exact value of overlap correction strength from data." Assumption 1
  is untested, and β = 0 gave FER0 23.33 ± 14.53%.
- **The LLM's benefit is not shown.** 44.91 ± 1.36 vs 43.78 ± 1.18 in one
  setting at 5 seeds. The SEs overlap, and the case study is one seed.
- **Gains mostly undo planted damage.** Each starting harness is broken on
  purpose (e.g. "return the first displayed title"). Gains of 10–35 pp are
  repairs of an obvious fault, which a gate passes easily. The late-run
  regime, where real edits are small, is the one a box loop lives in. Only
  the drawdown numbers speak to it.
- **The reuse stress test was not run on TRUST.** App. I (20 reused
  examples, verbatim answer memorized) uses Greedy only. So the setting
  where Assumption 1 is most strained has no TRUST arm.
- **The labels are noisy too.** FER0 and TER label a candidate "inferior"
  from its Δ̂ on 300 test items, ties included. An item's execution noise
  is near zero (temperature 0, cached responses), so the noise in the
  experiments is item sampling only.
- **Transfer to ML-research loops (our inference).** The e-value needs
  per-item paired outcomes that are independent given the history.
  Training-seed noise in an ML experiment is shared by every item that
  model scores. So an item-level e-value on one trained model certifies
  "this checkpoint beats that one", not "this configuration beats that
  one", which is what `/iterate` selects. Seed replicates would be the
  unit, and at `/iterate`'s 1–3 seeds that is
  [[literature/papers/qu2026propose]]'s "certifies nothing" regime.
- **Fit with the graph.** [[literature/papers/hu2026winners]] found that
  fixed-level McNemar and PACE-like gates rarely commit. This paper agrees
  for FWER gates and gets power back mainly through the margin.
  [[literature/papers/zhou2026does]]'s repeat-the-win rule is the cheap
  rival, and no one has compared the two.
  [[literature/papers/so2026masked]] adds that a margin-tolerant gate is
  the kind that lets a cheap safeguard deletion through, since a deletion
  that costs under ε is "tolerated". [[literature/papers/mitra2026agent]]
  (digest item 4) also uses per-context noninferiority, at the lesson
  level.

## Trust signals

- **Credibility:** 3.
  - For: complete proofs in the appendix (supermartingale validity at
    stopping times, FER at outer stopping times, the TER rate, the gain and
    MDD bounds), written carefully enough to check line by line. Mean ± SE
    over 5 paired seeds. Four baselines including two published
    statistical gates. Honest limitations: β cannot be identified, the
    allocator can err, and the bound is conservative. Two worked case
    studies show the failure mechanism. The method lineage (e-GAI,
    e-LOND, contextual online FDR) is cited and used correctly.
  - Against: preprint, 9 days old, no citations. Code is referenced but
    not linked. Several parameters that matter (ρ_min, ρ_max, α_max, the
    rubric-to-multiplier map, PACE's null) are unreported. Ablations
    cover one setting. The headline comparison is not matched on the
    noninferiority margin. Starting harnesses are deliberately broken.
    Two cheap models only. The LLM benefit is within SE.
  - Not 4 because there is no code and the key empirical contrast is
    confounded by the margin. Not 2 because the theory is sound, and the
    empirical claims that do stand (Greedy's drawdown, the FWER gates'
    paralysis) are consistent across six settings.

## Follow-up

- **Relevance:** 4. It is the first source here with a proven cross-round
  error rate for an RSI keep gate, plus a proof that the guarantee
  survives a learned allocator. It adds material evidence to the keep-rule
  thread in [[concepts/budget-as-ceiling]], to
  [[concepts/hce-evaluation]] (reuse correction as an assumption, and
  where a model may sit inside a statistical gate), and to
  [[concepts/skill-library-lifecycle]] (a noninferiority admission gate
  admits no-ops by design). Not 5 because `/iterate` has no keep gate for
  it to replace. The band it was picked against is a stop rule, and the
  test does not transfer to seed-level ML noise without replicates the
  box does not run.
- **Not drafted for [[concepts/evolutionary-expansion]].** The digest
  marked it on-front, but TRUST is a single-incumbent loop with no
  population. Its contribution is the selection rule, which the
  budget-as-ceiling keep-rule thread already holds.
- **Design implication for this box (our inference).** If a box loop ever
  adds a keep gate on per-item outcomes, three parts transfer:
  - re-measure the incumbent paired on each round's draw, never against a
    stored best;
  - fix the round's evidence threshold before the draw, in code;
  - set the loss margin explicitly, since it governs power more than the
    error criterion does.
  If an LLM sets the threshold, the clip and the ledger must be code.
- **Candidates** (cited, not yet in the graph):
  - Shawn 2026, "PACE: anytime-valid acceptance tests for self-evolving
    agents" (arXiv 2606.08106), the per-round baseline that matched TRUST
    on FER0.
  - Wu et al. 2025, "SGM: a statistical Gödel machine for risk-controlled
    recursive self-modification" (arXiv 2510.10232).
  - Sengupta 2026, "Self-evolving agents with anytime-valid certificates"
    (arXiv 2607.00871).
  - Nguyen, Nguyen & Vuong 2026, "Recursive self-evolving agents via
    held-out selection" (RSEA, arXiv 2606.28374).
  - Zhang et al. 2025b, "e-GAI: e-value-based generalized α-investing for
    online FDR control" (ICML 2025), the allocator TRUST adapts.
