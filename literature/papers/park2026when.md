---
kind: paper
title: "When Do Agent Loops Mistake Stagnation for Progress? Self-Evaluation Bias and Externally Grounded Verification in Long-Running Autonomous LLM Agent Loops"
authors: ["Hyundoo Park", "Byungho Choi"]
institutions: ["Independent researchers (Republic of Korea)"]
year: 2026
venue: "arXiv 2607.25152 (cs.AI), v2 revised 2026-09-29; self-described 'preliminary draft'"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.25152"
code_url: null
citations: null
source: "raw/papers/park2026when.pdf"
added: "2026-10-05"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/file-as-bus]]"
tags: ["progress-mirage", "self-evaluation-bias", "llm-as-judge", "out-of-band-evaluator", "keep-gate", "rollback", "stagnation", "world-state-oracle", "preregistration", "pilot", "noise-band", "conditional-acceptance-rate"]
---

# When Do Agent Loops Mistake Stagnation for Progress? Self-Evaluation Bias and Externally Grounded Verification in Long-Running Autonomous LLM Agent Loops

## TL;DR

A small controlled pilot. One agent, with its tools held fixed, edits a
signup-funnel web app for 6 cycles. Three repetitions are run per arm, and
only the **gate** that decides whether each candidate is deployed changes:

- **in-band-self**: the agent's own claim decides.
- **in-band-judge**: a stronger LLM reads the files, the diff and its own
  verdict history, under a critical prompt that defaults to reject.
- **out-of-band**: accepts only if a hidden, network-isolated simulator's
  conversion delta is > 0.

The agent claimed an improvement in **54/54** cycles. **56%** of those
cycles had delta ≤ 0. The self gate is therefore accept-all, and it eroded
its mean peak by 19%. The judge did better than accept-all but did not
close the gap. It accepted regressions 40% of the time and rejected 3 of
8 real improvements. The judge's failure disappears on a control task
whose success criterion is checkable from the artifact. A gate that
returns only accept/reject did about as well as one that also returns the
numbers.

For this graph, the measured content is narrow and mostly confirms
existing positions: gate keeps on a measured metric, not on narrative or
an LLM's opinion. The oracle is deliberately **noise-free** (common random
numbers). So the "sign-only gate ≈ full feedback" result says nothing
about the noisy-metric regime that the pending /iterate noise-band
proposal is about. The authors themselves name that regime as an open
extension.

## Claims

- **The progress mirage is a grounding problem, not a quality problem.**
  "The cause of the gap is not the evaluator's quality but what the
  evaluator is grounded in."
- **Self-report carries no information.** "Self-report was thus an
  uninformative signal, and the self-verdict gate that trusts it
  degenerated into accept-all."
- **A stronger judge does not close the gap (preregistered HA1
  rejected).** The judge's "errors are thus bidirectional: not a problem
  of leniency but a ceiling on discrimination."
- **The boundary condition (H2).** When success "is verifiable from the
  artifact itself", the judge's "mirage vanished to 0", so the apparatus
  "was not designed to make in-band lose unconditionally."
- **The gate matters more than the feedback.** Removing the numbers from
  the feedback leaves output similar. The benefit is attributed to "the
  gate's grounding rather than … the information content of the
  feedback."
- **Loop plumbing vs. the reward brain.** "plumbing (scheduling, pacing,
  session revival) can be delegated to built-in primitives, whereas the
  reward brain grounded in the real world cannot." The platform loop and
  goal primitives judge in-band, so they are adequate only when completion
  is "transcript-checkable".
- **Awareness of stagnation becomes rewardable process.** This claim comes
  from the field case. Under in-band reward, "even awareness of stagnation
  is recorded in the transcript as part of a new analysis, and therefore
  becomes rewardable process too."
- **The authors hedge their own claims.**
  - The out-of-band mirage of 0 "is not an empirical finding but a
    consequence of definition."
  - The four auxiliary mechanisms (append-only state, file handoff,
    frame-adversarial review, self-healing scheduler) are "unvalidated
    design notes".
  - The 61-cycle field case is "motivation only."

## Methods

- **Task T1 (open-ended).** The agent edits a landing page, funnel steps
  and copy to maximize signups. The oracle is a synthetic user simulator:
  a **weighted logistic over four artifact features** (CTA present,
  payload size, number of funnel steps, copy length), scored on a fixed
  cohort.
  - Within a repetition the cohort random numbers are the same each cycle,
    so "the sign of a delta cannot be flipped by sampling noise."
  - The oracle is "instant and without noise" (§6.4).
- **Isolation.** The agent and the oracle run in separate containers on
  networks with no route between them. The only shared surface is a
  read-only artifact volume. Two checks back this up:
  - A boundary proof at the start of each run. The host fetches a canary
    string, and the agent fails to fetch it even when given the oracle's
    IP.
  - A canary scan of agent output in every cycle, which found 0
    occurrences.
- **Gated deployment with rollback.** An accepted candidate becomes the
  deployed state. A rejected one is rolled back, and the reason is fed to
  the next cycle. An independent monitor records the oracle delta in every
  arm, which is how the in-band arms' errors are measured.
- **Scale.** One unnamed "frontier agent model", plus one unnamed stronger
  judge model. Three arms × 3 repetitions × 6 cycles gives 54 T1 cycles.
  The other runs are 9 B1 runs, a 3-repetition sign-only variant, and the
  apparatus v1/v2 history.
- **Control task B1.** Success is the count of five structural
  requirements met: a form, an email input, a submit control, exactly 3
  steps, and 40–80 words of copy. The same spec is given to every
  evaluator.
- **Dependent variables.** They are computed by standard-library scripts
  from append-only cycle logs:
  - mirage rate = the share of accepted cycles with delta ≤ 0;
  - P(accept | Δ ≤ 0) and P(accept | Δ > 0);
  - false-rejection rate;
  - real outcome at budget;
  - time to first real outcome;
  - wasted-cycle ratio.
- **Preregistration status.** Hypotheses and thresholds were committed
  before measurement, but "the commit-hash freeze … was not completed." It
  is an "exploratory measurement under a pre-freeze registration draft".
  Amendments are disclosed: the mirage definition changed, and the
  apparatus was modified mid-run. H3 and HA2, the reward-function and cost
  axes, were not run.

## Results

- **Table 1 (T1, means over 3 reps; baseline conversion 33.3):**

  | Arm | Mirage | Accepted/6 | P(acc\|Δ≤0) | P(acc\|Δ>0) | Peak | Final |
  |---|---|---|---|---|---|---|
  | in-band-self | 0.56 (0.50–0.67) | 6.0 | 1.00 | 1.00 | 95.0 | 76.7 |
  | in-band-judge | 0.44 (0.33–0.67) | 3.0 | 0.40 | 0.63 | 100.0 | 86.7 |
  | out-of-band | 0.00 (structural) | 2.7 | 0.00 | 1.00 | 113.0 | 113.0 |

- **Our reconstruction of the counts.** Each arm has 18 cycles, about 10
  with Δ ≤ 0 and 8 with Δ > 0. The headline numbers are therefore:
  - judge accepts **4 of 10** regressions and **5 of 8** improvements;
  - "rejected 38% of real improvements" = **3 of 8**;
  - "44% regressions" = **4 of 9** accepted cycles.
- **Wasted-cycle ratio is 0.56 in all three arms.** The proposer's
  regression rate did not differ by arm. The arms differ only in what the
  gate let through.
- **Erosion.** For in-band-self, (95.0 − 76.7)/95.0 = 19%, from mean peak
  to mean final. "reproduced in all three repetitions". In the worked
  trajectory, three real gains (33→47→95→104) are followed by three
  plausible regressions (social proof −6, inline email −1, testimonial −6).
  It ends at 91.
- **Judge errors.**
  - Accepted: a styled hero section (−23) and social proof above the CTA
    (−10).
  - Rejected: removing a verification step (+7) as "inflating unverified
    signups", and rewrites it suspected of breaking links (+15, +16).
- **B1 (Table 2).** Judge mirage 0.44 → **0.00**. Under apparatus v1 with
  input truncation it was 0.17, and it fell to 0 once the diff and verdict
  history were added. **in-band-self stays at 0.50 on B1.**
- **Sign-only vs full feedback (Table 3).**

  | Feedback | Final per rep | Mean | Accepted/6 | Wasted ratio |
  |---|---|---|---|---|
  | full | 130, 117, 92 | 113.0 | 2.7 | 0.56 |
  | sign-only | 122, 104, 104 | 110.0 | 3.0 | 0.50 |

- **Time to first real outcome.** It was 1 cycle in every arm and every
  rep, so this variable did not separate the arms.
- **Under the preregistered literal definition (Δ exactly 0),** every
  arm's mirage is 0.00 and **H1 is undecidable.**
- **Without the gate.** In an early apparatus with no rollback, final
  output "did not separate the arms".
- **Field case (anonymized, uncontrolled).** A director-worker loop ran for
  about 6 weeks and 61 cycles. All 57 evaluated cycles were accepted while
  every external signal stayed flat. Moving the gate to real-world outcomes
  with an out-of-band verifier reversed behavior "from analysis to
  shipping".

## Critique / open questions

- **The sample is tiny, and the effect sizes are counts.** There are 3
  repetitions of 6 cycles per arm. The judge's headline rates rest on 9
  accepted cycles and 8 positive-delta cycles. Our quick check: 4/10 vs
  5/8 acceptance does not separate significantly (Fisher exact, p ≈ 0.6).
  The "bootstrap intervals" over 3 trajectories are, by the authors'
  admission, observed ranges.
- **"Long-running" is 6 cycles.** The title's long-running loop is
  supported only by the uncontrolled field anecdote. Each controlled run
  lasts 6 cycles, and every arm takes its large gain in cycle 1.
- **The discussion overstates the judge result.** §4.3 says the judge's
  discrimination was in "the right" direction (0.40 vs 0.63). Two later
  passages contradict that:
  - §5.1 says both transcript-bound gates were "effectively independent
    of the sign of the oracle delta".
  - §5.1 also says the judge's mirage was "effectively the same as
    in-band-self's" (0.44 vs 0.56).

  The data show a weak judge, not a null one.
- **An internal inconsistency on candidate quality.** §4.1 reports the
  wasted-cycle ratio as "coincidentally identical at 0.56 across the three
  arms". §6.2 says "a second-order effect is observed in which the agent's
  regression-candidate generation rate falls in the out-of-band arm." The
  tables show no such fall: 0.56 for full feedback, and 0.50 only for
  sign-only.
- **HA1's rejection is close to built in.** The judge is asked to predict
  the sign of a hidden four-weight response function that it was never
  shown. Its failure shows that a transcript can't contain information it
  was never given, which the authors largely concede (§6.1). Their defence
  is that real A/B tests also mostly refute plausible ideas (Kohavi). That
  argues the simulator is realistic in sign structure. It does not show
  that a judge with domain priors would do as badly on a real service.
- **H2 is likewise close to definitional.** On B1 the judge holds a
  checklist that *is* the oracle. The non-trivial part of B1 is the other
  arm: the self gate stays at 0.50 even with the spec in plain view.
- **"Self-report uninformative" was tested only with binary claims.** The
  agent was never asked for a calibrated probability or a predicted delta.
  A constant "yes" has zero information by construction. Whether the
  model's ranked confidence carries signal is untested.
- **The noise-free oracle limits what the sign-only result means.** With
  common random numbers, a strict `Δ > 0` gate is a perfect classifier.
  "Sign-only ≈ full feedback" therefore holds in exactly the regime where
  the sign is free of error. The authors flag this themselves: under noisy,
  delayed metrics "a simple gate that accepts only positive deltas can
  reject true improvements for lack of statistical power", with
  "interval-estimate-based acceptance" left for the full study.
- **The oracle measures something narrower than value.** Removing email
  verification scored +7 and was booked as the judge's false rejection,
  though the judge's reason was sound. The authors call this "the
  surrogate-metric problem one level up".
- **Apparatus changes were made after seeing results.** Examples are the
  v1 → v2 judge input and the CTA-detection reinforcement, and the B1 judge
  mirage moved 0.17 → 0 with them. These changes are disclosed, and the
  arms' mirage ordering is reported as invariant across versions. Still,
  the H2 "accepted" verdict was reached after an apparatus change.
- **Reproducibility is asserted but not checkable from the PDF.**
  - Both models are unnamed.
  - "the full prompt is in the public repository" and "the testbed is
    released", but no URL is given.
  - The cited appendix (preregistration document, apparatus history) is
    not in the 23-page PDF.
- **No new mechanism for ML loops.** The recommended "out-of-band" gate is
  keep-if-better on a metric the agent can't write. That is already the
  standard keep rule in Karpathy-style AutoResearch and in `/iterate`, so
  for ML-research loops the paper adds evidence for an existing design
  rather than a new one.

## Trust signals

- **Credibility:** 2. Against the paper:
  - two independent researchers, no peer review, self-labelled
    "preliminary draft";
  - n = 3 × 6 cycles per arm, one task, one unnamed agent model and one
    unnamed judge model;
  - no code URL despite claims of release, and the appendix is missing;
  - the preregistration was not frozen;
  - the discussion overstates the judge result.

  It is not lower because the design is clean (single manipulated
  variable, isolation proven per run, a falsification control) and the
  disclosure is unusually candid: structural zeros are flagged, H1 is
  shown undecidable under the registered definition, and the amendments
  are listed.

## Follow-up

- **Relevance:** 4. It is the first source in this graph that holds the
  agent fixed and varies **only** what the keep gate is grounded in, with
  an ablation on judge strength and a control task. That is material new
  evidence for:
  - [[concepts/programmable-evaluator-oracle]]: an oracle over a judge, and
    the operating-point framing via P(accept | Δ ≤ 0) and P(accept | Δ > 0);
  - [[concepts/evidence-gated-completion]]: self-report as non-evidence.

  It is not a 5 for three reasons: the evidence is pilot-scale and
  low-credibility, the domain is web conversion rather than ML research,
  and for ML loops its remedy is already the default.
- **Bearing on the pending 2026-09-27 `iterate-no-improvement-noise-band`
  proposal: neutral to weakly supportive. Not admissible as evidence for
  the band.**
  - The sign-only result supports a bare `Δ > 0` keep rule only when the
    metric is noise-free, which the apparatus guarantees by construction.
  - The authors independently name "interval-estimate-based acceptance"
    as the gate design needed under noise. That is a premise-level
    agreement with no data.
  - The paper does supply a useful **instrument**: conditional acceptance
    rates P(keep | true Δ ≤ 0) and P(keep | true Δ > 0). For a noisy
    single-seed metric, a bare `>` keep rule accepts a true-zero change
    about half the time. That makes it behave like this paper's weak judge
    (0.40), not like its oracle gate (0.00). This is our inference, not the
    paper's.
  - A reviewer of the proposal could state its success criterion in these
    terms: the band should push P(keep | Δ ≤ 0) down without pushing the
    false-rejection rate above what hu2026analyzing-style replicates
    tolerate.
- **Erosion is the cost of false accepts on the champion.** Accepting
  regressions whittled the best state down by 19%. A gate that rejects only
  "preserves the best state reached". This bears on the proposal's
  deferred "champion selection has the same flaw" point. A chain that
  promotes sub-noise "new bests" is exposed to the same erosion, at
  noise-level magnitude.
- **Converges with [[literature/papers/qu2026propose]]'s "who judges sets
  false admissions, who proposes sets yield".** Here the proposer's
  regression rate (0.56) is flat across arms, and only the gate changes
  what is admitted. These are two independent setups with the same
  decomposition. It also sits beside:
  - [[literature/papers/huang2026reward]]: "observability is not
    verification";
  - [[literature/papers/xu2026dont]]: a judge with the account in view.
- **Stagnation handling.** In the field case a days-flat counter switched
  the loop into an outcome-only acceptance mode once a threshold was
  crossed. That is the same stagnation → constrain/redirect shape as
  [[literature/papers/chandran2026autoresearch]]'s Criticizer, and it is
  unvalidated here too. The claim that in-band loops turn "awareness of
  stagnation" into rewardable analysis is a reason a
  `max_consecutive_no_improvement` counter must be computed from the
  metric log, never from the agent's own summary of progress.
- **Platform loop/goal primitives judge in-band.** For this box, `/loop`
  and goal-style completion checks are adequate only where completion is
  transcript-checkable. Any `/iterate`-style keep decision must stay on
  the metric file.
- **Candidates:**
  - Advani 2026, "From Confident Closing to Silent Failure" (arXiv
    2606.09863). False success across 5 judges × 5 prompts. It is cited in
    zhang2026how and not yet ingested.
  - Pan et al. 2024, "Spontaneous Reward Hacking in Iterative
    Self-Refinement" (arXiv 2407.04549).
  - Wang et al. 2026, "The Verification Horizon" (arXiv 2606.26300).
  - Flynt 2026, GroundEval (arXiv 2606.22737).
