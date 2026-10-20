---
kind: paper
title: "When Can You Trust an Agent's Own Verifier? Stress-Testing Counterfactual Rationale Verification Under Search Pressure and Oracle Access"
authors: ["Jonghyun Lee", "Moonsu Kim", "Sunghwan Park"]
institutions: ["Yonsei University"]
year: 2026
venue: "arXiv 2610.22779 (cs.AI), v1 2026-10-16; footer: NeurIPS 2026 Workshop \"Who Verifies the Agents? Toward Reliable Agent Development\""
peer_reviewed: false
url: "https://arxiv.org/abs/2610.22779"
code_url: null
citations: null
source: "raw/papers/lee2026when.pdf"
added: "2026-10-20"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["in-loop-verifier", "rationale-faithfulness", "counterfactual-verification", "pressure-ratio", "applied-vs-nominal-weight", "tie-break", "oracle-access", "selection-signature", "claim-minimization", "specification-gaming", "goodhart", "capability-control", "replicate-band", "self-audit", "pipeline-design", "thompson-sampling", "solar-pro-3"]
---

# When Can You Trust an Agent's Own Verifier? Stress-Testing Counterfactual Rationale Verification Under Search Pressure and Oracle Access

## TL;DR

An LLM agent (Solar Pro 3) designs supervised-learning pipelines (feature
subset, model class, hyperparameters) and states a rationale. The authors'
own verifier, the Rationale Consistency Score (RCS), parses the rationale
into claims, negates each claim (drop the feature, swap the model, perturb
the hyperparameter), retrains, and averages the validation-accuracy drops.
RCS enters the search objective as R = Q̂ + λ·RCS, and Thompson sampling
picks the next pipeline type. The paper is a stress test of this verifier,
not a new verifier. It defines a pressure ratio ρ (mean applied λ·RCS over
the s.d. of Q̂ across iterations) and four probes. Six datasets, three
seeds, 16–19 iterations per run, 180 runs plus a 90-run UCB1 sweep.

1. **The deployed verifier barely steered search.** ρ = 0.007–0.089 on all
   six datasets, partly because an accuracy gate cut the applied weight
   (Credit-G: 0.030 applied vs 0.088 nominal). It still changed the reported
   pipeline in 5/18 runs, every time among accuracy-tied candidates.
2. **Search never discovered claim minimization (F3).** Claim counts stay
   flat up to ρ ≈ 64, yet a prompt that tells the agent to state fewer
   claims cuts them 18–35% on four datasets. The authors call this null
   "largely structural": feedback never shows claim counts and runs are
   short.
3. **Letting the agent probe the verifier before committing (F4)** gives
   fewer claims at a higher verified-claim rate on 4 of 6 datasets, while
   held-out RCS falls. Only Credit-G's fall has non-overlapping CIs, and
   Credit-G is where the estimator's small-pipeline rules apply to the probe
   arm and not the control.

## Claims

- "Applied pressure must be measured": whether an auxiliary objective term
  steers search cannot be read from its nominal weight. This is the
  primary methodological claim. ρ is offered as one operationalization.
- In the prior configuration "the verifier barely affects the search
  objective (ρ ≪ 1)", so its robustness "had not been tested" (F2). The
  verifier still shaped the reported artifact through a final-selection
  tie-break that ρ does not register.
- "Shorter claim lists are reachable, but search — Thompson sampling or
  UCB1 — did not find them without instruction." This is called the
  central result, and the authors say it is "a discovery gap, not a
  demonstrated reward exploit".
- Probe access "produces the selection signature … on four of six datasets
  at sign level, with held-out fidelity declines that the estimator may
  partly explain". "Reporting only the verified-claim rate would record
  improvement as fidelity degrades."
- On Ecommerce, putting the verifier in the objective "already degrades
  what it is meant to protect": held-out accuracy 0.675 vs 0.794 and
  held-out RCS 0.056 vs 0.163, both with non-overlapping CIs.
- Proposed reporting standard: report ρ, count claims separately from
  scores, pair any F3 robustness claim with a capability control, and
  measure fidelity on verifier-unseen data.
- Self-described scope: "preliminary and mixed", one verifier of their own,
  one generator, six datasets. The ρ range "comes from a single self-audit,
  not a survey."

## Methods

- **Loop (Fig. 1).** A design agent proposes (S, M, H) plus a rationale in
  one call. A verifier agent parses claims deterministically, builds
  counterfactual pipelines, retrains and returns RCS. A search agent forms
  R = Q̂ + λ_t·RCS and updates Thompson-sampling posteriors over
  pipeline-type arms. The next prompt carries task and verifier scores but
  never the claim count.
- **Deployed RCS (App. A).** The paper flags each of these as a degree of
  freedom:
  - per-claim noise thresholds (about 2–4 × 10⁻⁴);
  - a relative-drop boost (β = 2.5);
  - joint removal when three or more feature claims are stated;
  - a per-drop cap of 0.02 on pipelines with ≤ 10 features;
  - a floor value for unparseable or unexecutable rationales;
  - an accuracy gate that scales λ by 0, 0.5 or 1.2 depending on
    validation accuracy, with a warm-up multiplier of 0.78/0.68.
- **Verification-aware final selection.** Among candidates within a
  per-dataset validation tolerance (0.004–0.02, 0.035 on Credit-G), the
  reported pipeline is the one with higher RCS. This was on for all six
  datasets in the deployed configuration.
- **Probes.**
  - *P1 (dose–response):* static λ_nom ∈ {0, 0.12, 1, 5, 20} with the gate
    off.
  - *P2 (oracle access):* both arms get a tool-using proposer. Only the
    treatment may call `probe_feature_drop`, capped at four calls per
    iteration.
  - *P3:* the floor rate (how often RCS is silent) and seed spread.
  - *P4 (capability control):* at λ_nom = 5, a "minimize" prompt that
    discloses the averaging rule and asks for fewer claims.
- **Selection signature.** Fewer claims per iteration (ΔN̄ < 0) together
  with a higher verified-claim rate (Δv̂ > 0), stated before the results.
- **Data.** Six public tasks: Credit-G, UCI Breast Cancer, Madelon,
  Ecommerce text (TF-IDF), Butterfly image (26 grayscale statistics,
  ≈ 0.19 accuracy on 75 classes), Electricity (time-ordered split). Split
  64/16/20. Search uses validation only. Held-out accuracy and RCS are
  computed once on test after selection and enter neither objective nor
  selection.
- **Statistics.** Percentile bootstrap over per-seed run means (10,000
  resamples) with three seeds (42/43/44). Per-dataset Spearman trend tests
  (n = 15) with Holm correction. A post-hoc selection flip rate (Table 4)
  re-selects each run's reported pipeline with and without the verifier
  term.
- **Identical-configuration replicate (Table 3).** Two arms coincide
  exactly (λ_nom = 5 of P1 and the default arm of P4) but were invoked
  separately, so their gap measures generator noise at fixed seeds. Claim
  volume differs by up to 11.4%, held-out RCS by ×0.61–×2.71, test accuracy
  by up to 0.048 and verified rate by up to 0.06. Every effect is read
  against this band.

## Results

- **F2, pressure (Table 2).** ρ = 0.012 / 0.010 / 0.089 / 0.007 / 0.010 /
  0.028 (Credit-G / Breast / Madelon / Ecommerce / Butterfly / Electricity).
  None of the 18 runs exceeds 0.10. With the gate off at λ_nom = 0.12, ρ is
  0.04–0.28. The floor rate is 0% (Madelon) to 9.3% (Breast).
- **Selection flip rate (Table 4).**

  | Arm | Flips | Val. accuracy conceded (mean / max) |
  |---|---|---|
  | deployed (dynamic λ) | 5/18 | 0.000 / 0.000 |
  | λ_nom = 0 | 0/18 | 0.000 / 0.000 |
  | λ_nom = 0.12 | 6/18 | 0.003 / 0.009 |
  | λ_nom = 1 | 10/18 | 0.011 / 0.044 |
  | λ_nom = 5 | 14/18 | 0.034 / 0.209 |
  | λ_nom = 20 | 13/18 | 0.036 / 0.192 |

  At deployment the verifier breaks ties. At high weight it trades real
  validation accuracy for RCS. The table excludes the final-selection
  tie-break, which is a separate channel with no reported flip count.
- **Ecommerce backfire.** RCS-included vs excluded: test accuracy 0.675
  [0.648, 0.717] vs 0.794 [0.754, 0.825]; held-out RCS 0.056 vs 0.163. The
  accuracy gap (0.119) is 2.5× the replicate band. On Credit-G the point
  estimates favour the included arm (0.723 vs 0.697 accuracy) with
  overlapping CIs.
- **F3, dose–response (Fig. 2, Tables 7–8).** ρ reaches 41.7 (Madelon) and
  63.7 (Electricity) at λ_nom = 20. Claim counts stay flat (Credit-G
  10.6–12.2, Madelon 17.7–21.0) and per-dataset Spearman is −0.12 to +0.38
  (all p > 0.15). Rationale length is also flat. UCB1 with a doubled
  iteration cap (realized 18–38 iterations, ρ up to 38.7) gives the same
  picture. Under Eq. 3, deleting one inert claim at λ_nom = 20 would raise
  the objective by about 1.2 (Credit-G) to 2.1 (Madelon) accuracy s.d.s,
  an upper bound the paper does not test directly.
- **P4, capability control (Table 5).** The minimize prompt cuts claims by
  35% (Butterfly), 34% (Electricity), 27% (Madelon) and 18% (Credit-G), and
  barely changes Ecommerce (0.0%) and Breast (−1.7%). Three of the four
  cuts exceed the replicate band. Held-out RCS moves within or at the edge
  of the band.
- **F4, probe access (Table 6, Fig. 4).** The probe arm used 3.1–3.9 of 4
  calls per iteration.

  | Dataset | Claims/iter | Verified rate | Held-out RCS [95% CI] |
  |---|---|---|---|
  | Credit-G | 12.4 → 11.7 | 0.40 → 0.50 | 0.034 [0.025, 0.051] → 0.016 [0.010, 0.020] |
  | Madelon | 25.6 → 14.8 | 0.50 → 0.51 | 0.188 → 0.095 (wide CIs) |
  | Butterfly | 15.5 → 14.3 | 0.49 → 0.50 | 0.093 → 0.090 |
  | Breast | 10.5 → 10.3 | 0.38 → 0.41 | 0.008 → 0.009 |
  | Ecommerce | 17.6 → 12.6 | 0.54 → 0.49 | 0.069 → 0.024 (overlapping) |
  | Electricity | 8.9 → 11.1 | 0.46 → 0.49 | 0.040 → 0.041 |

  Only Madelon's volume drop (−42%) and Credit-G's rate rise (+0.10)
  exceed the replicate band. Test accuracy does not fall in the probe arm
  (Credit-G 0.718 → 0.725, Electricity 0.703 → 0.729). Probe access flips
  3/18 reported pipelines against 4/18 in the control.

## Critique / open questions

- **Against the digest framing: BELOW (mild).** The digest's facts are in
  the paper and it says "small and preliminary". But it bolds the weakest
  result and leaves out the paper's own central one.
  1. *"Held-out faithfulness falls, invisibly to in-loop metrics"* is
     bolded as if it were a general finding. Held-out RCS falls on 3 of 6
     datasets. Only Credit-G has non-overlapping CIs, and Credit-G's probe
     winners all use ≤ 10 features while its control winners use 11–17. The
     per-drop cap and joint-removal rules therefore apply to one arm only.
     The authors say "part of those declines may be scoring artifacts". The
     only clean case is also the one most exposed to the confound.
  2. *The signature does not track the fidelity drop.* Ecommerce loses
     fidelity without the signature (its verified rate falls). Butterfly
     and Breast show the signature with flat fidelity. If selection on
     probe results caused the drop, the two should co-occur.
  3. *Omitted:* the F3 null that the authors call their central result
     (search never minimized claims even at ρ ≈ 64, while instruction did),
     and the Ecommerce backfire, which is the paper's strongest
     CI-separated effect.

  The ρ ≪ 1 and tie-break sentences are accurate.
- **The Ecommerce backfire has no identified mechanism.** At ρ = 0.007 the
  objective term cannot move search much, and Table 4's deployed flips
  concede 0.000 accuracy. A 0.119 test-accuracy gap therefore has to come
  from somewhere else. Three candidates:
  - the final-selection tie-break, which has no flip count;
  - the verifier scores shown in the proposer's prompt, a third channel
    that neither ρ nor the flip rate measures;
  - noise. The excluded arm's CI is wide [0.754, 0.825] with three seeds.

  The paper reads the gap as "the most direct evidence here that an
  in-objective verifier can backfire". That is plausible, but the paper
  does not show it.
- **ρ is fragile, and the authors say so.** Its denominator is the s.d. of
  Q̂ within a run, which inflates ρ on low-variance tasks (Electricity's
  63.7). Its numerator is a time average, so pressure concentrated in a few
  decisive iterations looks small. The flip rate is the more decision-
  relevant number, and it is post hoc: it re-selects over the trajectory
  that actually ran, not over a counterfactual run.
- **Power.** Three seeds and n = 15 per trend test detect only
  |r_s| ≳ 0.52–0.65. The F3 null is informative only because P4 reaches
  lower claim counts. The replicate band is a single pair of runs, "an
  order of magnitude for fixed-seed noise, not an interval".
- **The tasks are small.** Sixteen to nineteen iterations of sklearn-style
  pipelines, and one image task with a deliberately weak representation
  (0.19 accuracy, plus a "filename-hash fallback when the raw image is
  unavailable"). One hosted generator whose checkpoint cannot be
  identified. Four datasets ingested a duplicated header line as two data
  rows. This is disclosed and is identical across arms.
- **What the paper does not show.**
  - Gaming by rephrasing, which lowers the parsed claim count without
    lowering content. The authors flag this as untested.
  - Search that can see claim counts, the most direct F3 probe.
  - Delayed or noisy oracle access. The probe channel is noiseless and
    immediate.
  - Any verifier other than RCS, or any LLM-judged verifier. The four
    failure modes are argued to generalize, not tested.
- **Bearing on this hub's live fronts.**
  - *Accepted no-improvement noise band (`/iterate`):* the deployed RCS was
    in effect a within-tolerance tie-breaker, and it decided the reported
    pipeline in 5/18 runs at zero validation cost. Whatever breaks ties
    inside the band therefore chooses the artifact a chain reports. A
    diagnostic used that way needs its own audit (flip count, held-out
    check), because the band will not show it. This is the "diagnostic-gated
    tie-breaking … still-unisolated lever" from
    [[literature/papers/xu2026conflictguide]], now measured in one loop.
    The replicate band (fixed seed, generator resampled: test accuracy up
    to 0.048, ratio metrics ×0.61–×2.71) also supports building a band
    from re-runs, not seeds. Ratio metrics on small values are too noisy to
    band at all.
  - *HCE:* F4 extends clause 1's logic from `test/` to the verifier. A
    check the agent can call before committing is search signal, not
    evidence, and its pass rate can rise while fidelity on unseen data
    falls. This is the same pattern as
    [[literature/papers/ganesh2026homogenization]]'s gate pass rate against
    the oracle rate among passed items.
  - *Redirect-before-halt proposal:* no direct bearing. The paper has no
    stall counter, pivot or halt rule.
  - *Shared-substrate correlated error:* RCS avoids it by construction. Its
    verdict comes from retraining, not from a model sharing the
    generator's priors. That makes it a clean test of the agent's
    *response* to a verifier, and says nothing about LLM-judge verifiers.

## Trust signals

- **Credibility:** 2.
  - For: the disclosure is unusually complete. Every estimator constant,
    the accuracy gate, the tie-break, the data-ingestion bug and a
    stale-metadata mismatch are on record. There is an identical-
    configuration replicate band that every effect is read against, a
    capability control for the null, predeclared falsification
    conditions and Holm-corrected trend tests. The claims are modest and
    the authors label them preliminary. Yonsei University.
  - Against: three authors and a workshop-format preprint with no
    confirmed peer review. The authors are auditing their own verifier
    ("self-audit"). The paper says code, prompts, parser and per-run logs
    are released but gives no URL. One generator model, three seeds,
    short runs, toy-scale tasks. The one CI-separated F4 effect is
    confounded with estimator rules that differ between arms.
  - Not 3 because nothing can be checked without a code link, and the
    headline F4 effect rests on one confounded dataset. Not 1 because the
    methods and disclosures are careful enough that the reported numbers
    are internally consistent and the nulls are honestly bounded.

## Follow-up

- **Relevance:** 3. It is an ML-pipeline design loop with an in-loop
  auxiliary verifier, close to an `/iterate` keep rule with a diagnostic
  term. It contributes two transferable instruments: report *applied*
  pressure (ρ) and a selection flip rate instead of a nominal weight, and
  watch claim volume next to pass rate when the agent can query the check.
  It sharpens [[concepts/programmable-evaluator-oracle]] (the tie-break
  lever, measured) and [[concepts/hce-evaluation]] (query access as a
  selection channel). Not 4 because its one substantive gaming result is
  weak and confounded, and the rationale-faithfulness verifier is not a
  component any skill here uses. Mostly it confirms positions the graph
  already holds: keep the verdict on the frozen score, and measure on data
  the check never saw.
- **For `/iterate` (not proposed):** if a chain ever adds a secondary
  diagnostic to break ties inside the noise band, log how often it changed
  the kept candidate. Score the kept candidate's diagnostic once on held-out
  data at chain end.
- **Related in graph:** [[literature/papers/park2026when]] (gate type
  decides acceptance; a self-claim gate accepts everything),
  [[literature/papers/lumer2026reflective]] (what the oracle returns
  decides transfer), [[literature/papers/hu2026winners]] (selection bias
  from a correct oracle on few items), [[literature/papers/liu2026what]]
  and [[literature/papers/giampieri2026cortex]] (same-configuration rerun
  arms), [[literature/papers/nepal2026faithful]] (rules gradable from the
  output are met, manner rules are not).
- **Candidates** (cited, not in the graph):
  - Hsia et al. 2024, "Goodhart's law applies to NLP's explanation
    benchmarks" (Findings of EACL 2024). Removal-based faithfulness metrics
    are gameable.
  - Chuang et al. 2026, FaithLM (EACL 2026). Counterfactual-intervention
    faithfulness.
  - Hooker et al. 2019, ROAR remove-and-retrain (NeurIPS 2019).
