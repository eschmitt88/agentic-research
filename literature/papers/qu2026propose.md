---
kind: paper
title: "Propose, Don't Judge: An Anytime-Valid Referee for LLM Agents That Mine Investment Factors"
authors: ["Bo Qu", "Mingguang Chen", "Licheng Wang"]
institutions: ["DeepGrounding", "AlphaAvatar"]
year: 2026
venue: "arXiv (cs.AI; q-fin.PM, q-fin.ST)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.27051"
code_url: null      # "available from the corresponding author on request"; market data licensed, not redistributable
citations: null
source: "raw/papers/qu2026propose.pdf"
added: "2026-09-28"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
tags: ["anytime-valid", "e-values", "e-bh", "optional-stopping", "multiple-testing", "proposer-judge-separation", "frozen-evaluator", "factor-mining", "quant-finance", "sequential-testing", "peeking", "negative-portfolio-result"]
---

# Propose, Don't Judge: An Anytime-Valid Referee for LLM Agents That Mine Investment Factors

## TL;DR

An LLM factor-mining loop is split into a **proposer** (script, bandit or
LLM — the "agent surface", free to evolve) and a **frozen referee** the
proposer cannot mount, patch or intercept. The referee scores each candidate
only on daily rank-IC revealed **after submission**, by betting (a test
supermartingale per candidate → online e-BH for admission → an e-detector for
retirement), so FDR over everything ever admitted holds at every stopping time
for any proposer that has no information about the post-submission stream.
Crossing 3 proposers × 4 referees (the valid one plus three deliberately
leaky ones): **who judges sets false admissions, who proposes sets yield.**
On the CSI 500 the frozen referee admits 11.7 realised false factors per
campaign against 86–196 for the leaky arms under a script. The price is
latency — an admitted true factor waits about 500 trading days — and the
certified book's net Sharpe trails the ungated one. A patient fixed-horizon
t-test + BH desk gets most of the false-admission benefit and builds a
*better* book.

## Claims

- **Division of labour.** "The agent may propose. A statistical procedure
  that the agent cannot touch must judge." The trust kernel is the referee,
  the frozen library, the point-in-time data layer, the append-only decision
  log and the cost meter; probes, memory, allocation and diagnosis are a
  swappable surface.
- **Validity for any proposal policy (Proposition 1)** — under four stated
  conditions: bet fractions G_{t−1}-measurable; each candidate bet on
  post-submission data only; library frozen within an epoch; and (iv) the
  proposer has no information about the candidate's post-submission outcomes.
  The authors are explicit that it "follows from the cited results and is not
  a new theorem" (Wang & Ramdas 2022; Waudby-Smith & Ramdas 2024; stopped
  e-BH; e-detectors). Its value "is to rule out one possibility: that an
  endogenous proposal process breaks validity."
- **Named failure: foretelling** — sizing a bet with knowledge of where
  another candidate's stream is about to move; the Wang et al. 2025
  counterexample has E[M] = 1.25. Guarded by the alignment invariant and a
  unit test that reproduces the counterexample.
- **The wait is an information bound, not an engineering gap.** Expected
  wait ≈ ln(1/α)·2σ²/µ² at the Kelly stake, ln(N_v/(kα))·2σ²/µ² under e-BH;
  "no bettor can certify a factor at the threshold in under five years. Only
  a less noisy statistic or a lower bar shortens the wait."
- **The four threats** it names are exactly an adaptive loop's: optional
  stopping, near-copy resubmission (multiplicity), verifier-in-the-loop
  adaptation, knowledge-cutoff leakage. The referee answers the first three
  by construction and explicitly *not* the fourth.
- **The LLM's value is proposing and instrument-making**, not allocation: it
  beats the script on yield, roughly ties the bandit, and writes its own
  diagnostic probes — programs that "never enter a statistical test", so they
  carry no multiplicity cost.

## Methods

- **Referee.** Per-candidate capital W_t = Π(1 + λ_s X_s), λ_s ∈ [0, λ_max,s],
  λ_max,s < 1, stake fixed before X_t is seen (aGRAPA, capped at ϕ = 0.8 of
  the no-bankruptcy bound). The stream is **whitened** (X̃_t = X_t − ρ̂ X_{t−1})
  because AR(1) noise makes a zero-marginal-edge factor beatable: the raw
  referee admits 6.0% (ρ = 0.2) and 15.4% (ρ = 0.4) against a nominal 5%;
  whitening restores 1.8 / 2.0 / 2.4% at ρ = 0 / 0.2 / 0.4. Admission is
  online e-BH over a frozen universe of N_v = 2000 slots (a campaign uses
  about 490), α = 0.05, so the first admission needs capital N_v/α = 40,000.
  Retirement is an e-detector (sum of daily-restarted e-processes against
  the viability threshold δ = 0.015), A* = 1260.
- **Pre-registered constants**: α, δ, A, cap fraction, no-gate incubation,
  grids, analysis plan. Amendments are dated (Table 24). κ̄ = 0.018 and the
  execution layer are declared in-sample.
- **Leaky referees**: *peeking* (rolling one-sided t-test daily after 60 obs,
  admit at first p ≤ α), *adaptive threshold* (e-process but bar relaxes with
  age), *no gate* (admit after 126-day incubation).
- **Three environments**: a synthetic world with planted truth (1260 days,
  AR(1) ρ = 0.2, four families), a probe-authoring environment (5 held-out
  fault types × 20 episodes, 7 model families), and a point-in-time CSI 500
  walk-forward 2015-01 → 2026-08 with 2016–2019 starts. 540 cells (320
  baseline, 220 LLM), five seeds, common random numbers, paired t-tests with
  Holm within model family, families never pooled.
- **Legitimate comparator** (§8.5): a desk that tests each candidate **once**,
  500 days after submission, one-sided t-test on the whitened stream, BH at
  0.05 when each window closes — run counterfactually on the frozen cell's
  recorded submission stream.

## Results

- **Synthetic: the judge decides.** Frozen referee: 0.000 false admissions
  per submission in every seed for script, bandit and LLM; leaky arms
  0.26–0.85; realised FDP 0 vs 0.75–0.95. Yield under the frozen referee is
  identical across proposers (0.095 / 0.096 / 0.096). A hidden-retry
  attacker extracts +0.027 false admissions/submission from peeking, +0.009
  from adaptive threshold, none from frozen.
- **Real data (Table 2, four-family).** False admissions per campaign,
  frozen / peeking / adaptive / no-gate: round-robin 11.7 / 86.2 / 105.2 /
  196.0; LLM (Gemini) 11.7 / 37.4 / 43.5 / 78.0. The proposer only moves the
  count when the referee leaks. None of the frozen referee's 100 four-family
  cells admitted a negative-edge factor.
- **Label sensitivity.** On the after-admission-only label the frozen count
  rises 11.7 → 17.6 and the advantage is **5–11×** (vs 7–17× on the
  registered label); under the LLM **2.6–4.9×** (vs 3–7×). Against each
  family's own break-even the ratio shrinks to "two- to threefold".
- **The comparator nearly closes it on the four-family library**: 16.4
  false admissions vs 11.7 (one-look "At End" variant 16.0), yield 0.358 vs
  0.403, wait 500 vs a median 439. On the nine-family library it is "four
  times less accurate" (41.4 vs 9.8; 0.33 vs 0.12 share false) — and the
  paper attributes that to "the height of its bar", N_v/(kα), not to anytime
  validity.
- **The wait.** Median ~500 trading days for an admitted true factor under
  the frozen referee vs 122–214 under leaky ones. Worked at run parameters:
  546 days at edge 0.03, 1434 at δ = 0.015 (1356 for an uncapped Kelly bettor).
- **Portfolio: a negative result, reported.** Certified net Sharpe +0.33 to
  +0.50 vs ungated +0.59 to +0.77; alpha Newey–West t never reaches one;
  deflated Sharpe 0.49–0.69. The 500-day comparator builds a better book
  (+0.12 to +0.24 Sharpe, paired, every start) because the anytime-valid bar
  favours the statistic strongest *per day* — short-horizon reversal, the
  family that pays worst after costs. Certification does shelve less (13% of
  admitted vs 26% of ungated sleeves).
- **Probes.** Authored probes cut intervention regret by 0.388 / 0.248 /
  0.233 in 3 of 6 evaluable model families (Holm-significant), indistinguishable
  in the other three; one family never produced a probe.
- **Controls.** Six memory arms: 0 of 10 Holm tests significant. Same-policy
  forks reproduce every outcome-level metric in 5 of 5 cells.

## Critique / open questions

- **The theory is borrowed and sound; the empirics are one market.** The
  guarantee is a composition of peer-reviewed Ramdas-group results, stated as
  such. The empirical side is CSI 500 only, four start years sharing most of
  their data, no held-out period beyond the walk-forward.
- **"For any proposal policy" is conditional, and the LLM arms fail the
  condition.** On real data the LLM is a replay over years its training may
  cover, so condition (iv) fails; the authors say those results "are
  therefore read as replay outcomes", not instances of the guarantee. Their
  leakage sign-check (LLM leads the bandit before 2020, trails after — the
  opposite of what leakage predicts) is an interpretation, not a test.
- **The guarantee is on the whitened conditional null.** "The marginal-null
  guarantee under autocorrelation is measured, not proved."
- **The headline label leans toward the frozen referee** (its window overlaps
  the admission window). The after-admission relabel costs about a third of
  the ratio; the synthetic factorial, which has no overlap, carries the
  false-admission claim.
- **What actually buys the false-admission gap?** The comparator says
  mostly *validity + multiplicity correction + a high bar*, not anytime
  validity per se. Anytime validity adds three things (§8.5): read on any
  day, strong factors pass early, and the guarantee survives an adaptive
  proposer. The wait is set by noise and by the N_v = 2000 bar — the
  comparator waits 500 days too.
- **Author overlap and self-citation.** Two cited works are the authors'
  own (Chen, Wang, Qu 2026 survey; Qu & Chen 2026 CLQT). Many cited
  e-value papers are 2026 preprints not in this graph.
- **Transfer to `/iterate` (the reason this was ingested) is weaker than the
  digest framed it** — see Follow-up.

## Trust signals

- **Credibility:** 2 — small independent groups (DeepGrounding, AlphaAvatar;
  gmail contact), unreviewed arXiv preprint, code/prompts/recorded calls "on
  request" only, market data unredistributable, days old. It is the top of 2:
  the statistical core is inherited from peer-reviewed work and stated as
  not new, constants were pre-registered with a dated amendment log, CRN +
  Holm throughout, a legitimate comparator is run that *narrows the
  headline*, and the portfolio result is reported as a loss to the ungated
  book. Trust the mechanism and the wait arithmetic more than the 5–11×.

## Follow-up

- **Relevance:** 4. The first anytime-valid / e-value source in the graph,
  and the cleanest measured factorial of "who judges vs who proposes" in an
  agent loop. Not a 5: the domain is finance, and its transfer to this
  project's `/iterate` counter is argued (below), not measured — and on
  inspection it is narrower than the digest's "answers the tolerance
  question more strongly than a std band".
- **[[concepts/budget-as-ceiling]] — the noise-band question (09-27
  proposal `iterate-no-improvement-noise-band`).** Honest verdict: the
  paper answers a *different* question. Its guarantee is FDR over
  *admissions* under optional stopping — in `/iterate` terms, the honesty of
  "new best" / champion claims — not when a chain should halt (a halt is not
  an inference). Used as the counter's reset rule, a betting referee needs
  (a) a bounded metric on a known range, (b) a *stream* of fresh
  post-decision observations per candidate — i.e. new seeds run *after* the
  cycle declares a candidate, since the run that made it look good is
  pre-submission evidence and inadmissible — and (c) enough of them. Each
  bounded observation multiplies capital by (1 + λX) with λ < 1, so one run
  can at most double it; reaching 1/α = 20 is impossible from a single run.
  From the paper's own wait formula (my arithmetic, not the paper's):
  T ≈ 6(σ/µ)² fresh evaluations at α = 0.05 with no multiplicity, ≈ 12(σ/µ)²
  with a 20-experiment chain's e-BH bar — about 6 runs for a 1σ improvement,
  24 for 0.5σ, ~10⁵ for Agora's sub-noise steps. With `seeds_run: 1` the
  e-gate never certifies, so the counter never resets and the chain halts
  at cycle 3: correct but degenerate — a hard cycle cap. So the std band
  stays the right cheap edit; this paper adds (1) the reason not to peek
  and add seeds against a band until it clears (the peeking arm is the
  measured penalty: 86.2 vs 11.7), (2) a lower bound on evaluations per
  certified improvement, and (3) the upgrade path if chains ever run many
  cheap seeds: a sequential e-test on paired seed deltas. Its own comparator
  also vindicates the proposal's rejected heavy form — fixed `--seeds N` + a
  real test gets most of the benefit.
- **[[concepts/hce-evaluation]].** Measured price of peeking and of the
  one-look discipline; see the concept's 2026-09-28 section.
- **[[concepts/programmable-evaluator-oracle]].** "Agent writes
  instruments, not verdicts": probes that never enter the test are free
  of multiplicity cost and help in 3 of 6 families; the LLM as its own
  partial judge (no-gate arm) cuts 196.0 → 78.0 but stays far above 11.7.
- **[[concepts/information-firewall]].** Post-submission scoring is the
  prospective limit of the time boundary — the only form in which the
  adaptive-proposer guarantee is a theorem — and historical replay quietly
  voids it.
- **Candidates.** Waudby-Smith & Ramdas 2024 (JRSSB, betting CIs) and
  Wang & Ramdas 2022 (JRSSB, e-BH) are the peer-reviewed anchors any
  anytime-valid concept here should cite directly. Sengupta 2026 (SEA) and
  Shawn 2026 (PACE) — anytime-valid acceptance tests for a self-modifying
  agent's *own* modifications — are the closer fit for `/iterate` champion
  acceptance and are worth a digest check. Gonuguntla 2026 ("static replay
  of logged agent trajectories is invalid once policies diverge") bears on
  any replay-based harness comparison.
