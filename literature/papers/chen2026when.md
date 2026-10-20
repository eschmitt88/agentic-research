---
kind: paper
title: "When Agent Evaluation Becomes Unreliable: An Empirical Study of Oracle Failures and Configuration-Dependent Measurement in an LLM Agent Benchmark"
authors: ["Junjie Chen"]
institutions: ["Nanjing University of Science and Technology"]
year: 2026
venue: "arXiv 2610.16067 (cs.SE), v1 2026-10-12, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.16067"
code_url: "https://github.com/JunjieChen0/SimGlot"
citations: null
source: "raw/papers/chen2026when.pdf"
added: "2026-10-20"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
tags: ["oracle-integrity", "stateful-oracle", "known-answer-controls", "state-invariants", "fault-injection", "mutation-testing", "replay-correction", "tau-bench", "user-simulator", "configuration-sensitivity", "localisation", "trajectory-collapse", "pre-registration", "negative-result", "replication-package", "evaluator-audit"]
---

# When Agent Evaluation Becomes Unreliable: An Empirical Study of Oracle Failures and Configuration-Dependent Measurement in an LLM Agent Benchmark

## TL;DR

A single-author software-engineering study that treats an agent benchmark's
harness, and especially its reward function, as the system under test. It
has three parts, and the author ranks their evidential weight in this order:

1. **RQ1, oracle integrity (primary).** Eight checks run against a
   four-operation oracle interface: five state invariants (INV-1..5) and
   three known-answer controls (CTL-1..3). A battery of 7 author-built
   defect classes (21 variants × 3 targets = 63 combinations) measures what
   the checks catch. All 7 classes are caught. At the variant level the
   suite catches 54 of the 57 combinations a probe could separate (56/63
   conservatively). The one miss is the same on all three targets: an
   oracle that swallows its own exception. On τ-bench, `calculate_reward()`
   resets the database to replay the reference solution and never restores
   it. The value it returns is correct, and τ-bench's own runner is
   unaffected. The author's harness read the environment again after the
   episode, so it graded the reference solution's state. Replaying the
   4,320 archived episodes' tool calls against a clean database cuts
   database passes from 4,194 to 2,883 and success from 94.3% to 65.5%.
2. **RQ2, configuration sensitivity (secondary).** On τ-bench retail,
   translating the agent's policy, tool descriptions or error strings into
   Chinese lowers the *collapse* rate (a mechanical degenerate-trajectory
   label) by 5.3–9.2 pp. The base rate is 9.81% and comes from one agent.
   Success rises 5.1–9.6 pp. The three effects do not add. In the airline
   domain, the user-language effect on collapse has the opposite sign.
3. **RQ3, boundaries.** The suite could not be ported to a second,
   unnamed framework, because an external harness cannot rebuild its
   reference terminal state. A blind LLM judge from another family agrees
   with the collapse label on 98.1% of 210 items.

## Claims

- An agent benchmark's reward function is software and needs the tests
  other software gets. It is treated as definitional, so its defects are
  "invisible by construction."
- Two checks would have caught the defect immediately at no cost.
  Repeat-scoring (INV-2): score the same finished episode twice and require
  the same verdict. Terminal-state stability (INV-5): after the session
  ends, the environment must still hold the agent's state. Neither needs a
  reference implementation, a model or a human. The author proposes both
  for harness CI.
- The τ-bench defect is "an integration failure": "the unsafe side effect
  is upstream, the second reading is ours", with "no claim that the
  upstream runner inflates its own success rates."
- Replay, not re-run, is the audit primitive when providers offer no
  deterministic seed or immutable model version.
- Localising the apparatus is a property of the measurement, not of the
  model. The reversal across domains "retires" any claim that Chinese makes
  an agent better or worse.
- Two pre-registered expectations failed and are reported as such. Error
  strings were expected to carry the language effect and carry the least.
  The retail language effect does not survive a change of domain.

## Methods

- **Oracle checks.** INV-1: a direct call must not change the graded state.
  INV-2: two calls agree in verdict and resulting state. INV-3: the verdict
  does not depend on scoring order. INV-4: if the oracle raises, state is
  restored and the exception propagates. INV-5: terminal-state stability.
  CTL-1: a do-nothing episode, and one that says the required outputs
  without doing the writes, must fail. CTL-2: the reference solution must
  pass. CTL-3: an accepted alternative spelling must pass.
- **Fault injection.** There are 7 classes, each one dimension away from a
  correct oracle: unrestored mutation; restore on the success path only;
  restore that forgets the recorded actions; order dependence via a module
  cache; always-pass; ignore the state component; strict output match. The
  targets are τ-bench retail, τ-bench airline (the same oracle code on
  different tasks) and a synthetic domain. A differential-probe gate
  decides which variant–target pairs are separable before detection is
  counted. Unresolved pairs are never counted as equivalent.
- **Replay correction.** Every archived tool call is re-executed against a
  clean database and the terminal hash is compared with the reference.
  Faithfulness is checked two ways. All 137 step-limit episodes, which
  never reach the defective path, reproduce their labels. All 33,402 tool
  observations reproduce character for character.
- **Main matrix (pre-registered, archived).** 30 retail tasks × 2 user
  languages × 6 simulators × 4 agents × 3 repetitions = 4,320 episodes.
  Later additions, frozen before their runs: an environment-language arm
  (1,080, plus 1,080 length-padded), a per-component ablation (3,240), an
  airline repeat (50 tasks × 6 simulators × 2 languages × 2 reps = 1,200)
  and a collapse adjudication (212 items).
- **Collapse** means at least 5 consecutive reply steps, each spending at
  most 2 completion tokens, with no tool call.
- **Statistics.** Pairing is within (task, simulator, user language,
  repetition) cells. Intervals are task-clustered with t(G−1), G = 30
  (retail) or 50 (airline). An interval containing zero is reported as "no
  detected difference." The component contrasts are exploratory, with no
  multiplicity correction.
- **Deviations, logged.** A cross-vendor agent arm was dropped after 0/120
  pilot collapses. The adjudication became a census of 106 collapsed
  episodes, not 200. Two defects were found in the author's own validity
  gate.
- **Models.** "Four agents and six simulator models, all served by one
  provider." The paper says models are specified by name throughout, but
  no model is named anywhere in the text.

## Results

- **Detection.** All 7 classes are caught. At the variant level, 54/57
  separable (56/63 conservative). The miss is exception swallowing, which
  makes INV-4 record "not applicable" instead of a violation. In the
  author's words: "a check whose precondition is destroyed by the defect it
  is meant to detect will record that defect as inapplicable." Four checks
  are necessary for at least one class: INV-3 (9 unique detections), CTL-1
  (15), INV-4 (6), CTL-3 (5). INV-1, INV-2, INV-5 and CTL-2 are each
  redundant across this battery. Invariants alone miss a state-clean but
  wrong oracle, and controls alone miss a state-dirty one that happens to
  be right.
- **Minimal counter-example.** An agent that writes nothing and says
  goodbye is graded a database pass on the archived code path, and fails
  after the repair.
- **Correction.** Database passes 4,194 → 2,883 and success 94.3% → 65.5%,
  pooled over 4 agents, 6 simulators and 2 languages. There is no
  per-agent breakdown of the correction.
- **Component ablation (Table 2, retail, 1,080 paired cells per arm,
  corrected scorer).**

  | Arm | Collapse | Δ collapse [95% CI] | Δ success [95% CI] |
  |---|---|---|---|
  | English baseline | 9.81% (106/1080) | — | — |
  | Tool descriptions zh | 0.65% | −9.17 [−11.61, −6.73] | +9.63 [+5.09, +14.17] |
  | All three zh | 1.76% | −8.06 [−10.57, −5.54] | +5.56 [+0.45, +10.67] |
  | All three zh + padding | 1.30% | −8.52 [−10.77, −6.28] | +5.80 [+0.56, +11.04] |
  | Policy zh | 2.59% | −7.22 [−9.46, −4.98] | +5.09 [+0.80, +9.38] |
  | Error strings zh | 4.54% | −5.28 [−8.03, −2.52] | +5.19 [+1.31, +9.06] |

  Tool descriptions alone beat all three by 1.11 pp [0.36, 1.86]. The
  author's own power simulation gives that contrast 17% power, and the two
  arms ran in different batches at least a day apart. Padding to English
  length moves collapse by −0.46 pp, so length is not the explanation at
  the whole-environment level. Per-document length was not controlled.
- **Domain reversal.** For the one agent with a non-zero base rate,
  collapse with an English vs Chinese simulator is 6.48% vs 13.15% in
  retail and 9.67% vs 4.33% in airline. The difference of effects is
  −12.00 pp [−16.33, −7.67]. On airline success the difference is −2.00 pp
  [−7.05, +3.05], so no direction.
- **Adjudication.** 98.1% agreement (Wilson [95.2, 99.3]), κ = 0.962: 98.6%
  in the English arm and 97.9% in the Chinese arm. The judge is an LLM, the
  sample is enriched for positives, and the judge reads Chinese text in the
  Chinese arm.
- **Portability.** On the second framework, the 3 of 21 held-out tasks
  that looked usable as positive controls were false positives from the
  same state-reconstruction problem.

## Critique / open questions

- **Against the digest framing: BELOW (moderate).** Every number in the
  entry is in the paper. Three of its four sentences read stronger than the
  paper allows.
  1. *"Finds a reward function that mutates environment state before
     scoring. Correcting it drops measured success 94.3% → 65.5%."* The
     paper's own wording is that the oracle returns the correct value, and
     τ-bench's runner never sees a wrong label. The 28.8-point inflation
     was produced by the author's own harness reading the environment a
     second time. So it measures the cost of an integration bug in one
     project, not an error in published τ-bench numbers. The defect was
     also *discovered* in an earlier study. The suite was validated
     afterwards against a deliberate reproduction. "Applied to a real
     benchmark, it finds" reverses that order.
  2. *"Catches all seven injected oracle-defect classes."* True at the
     class level. The author himself calls that summary "more generous
     than the result": 54/57 at the variant level, with a structural blind
     spot. Every variant is author-built, so the figure is "a conditional
     property of this battery, not a detection rate for unknown defects."
  3. *"Tool descriptions and error strings shift reliability by 5–9
     points."* These are shifts in the collapse rate. Collapse is a
     degenerate-trajectory label carried by one of four agents, on 30
     retail tasks, and the ordering is labelled exploratory. Localising
     *reduced* collapse and *raised* success.
  4. The language reversal is reported accurately.
- **Ambiguity about the miss.** The Table 1 caption puts the undetected
  variant in the "saturated oracle" class. The text calls it an
  exception-swallowing oracle. Both can be true if the swallowing variant
  passes everything, but the paper does not say.
- **Models are unnamed.** All models come from one provider and none is
  named in the text, despite the declaration that they are. This weakens
  RQ2 more than RQ1. RQ1's evidence (code path, minimal counter-example,
  replay) does not depend on which model ran.
- **The paper cannot separate configuration from model.** One agent
  carries the collapse effect, and the other three have zero base rate in
  both arms. The airline domain changes task set, tools and base rate
  together. The author states both limits.
- **The pre-registration is internal.** It ships in the package, not on an
  external registry, so the timestamps cannot be checked independently.
  The prose refers to "an earlier draft" and "earlier work in this line"
  without citing either. The manuscript was LLM-assisted (declared).
- **What it means for the hub's fronts.**
  - *HCE clause 4.* The rule makes the final pass run a **pristine** copy
    of the evaluator. A pristine copy of a stateful grader with this bug
    would still mislabel if anything re-reads the environment afterwards.
    Clause 4 guards against tampering, not against a wrong evaluator. The
    paper's two cheapest checks (score twice; a do-nothing episode must
    fail) are a chain-start pre-flight that clause 4 lacks. See the
    concept edits.
  - *Noise band.* Translating tool descriptions alone moved retail
    success by +9.6 pp [+5.1, +14.2], and the domain change reversed the
    collapse effect. An `/iterate` gain that comes only from rewording
    prompts or tool text is therefore configuration-bound until it is
    checked on a second task suite. This is a general point, not specific
    to research loops.
  - *Redirect-before-halt.* Only tangential. The proposal lists the
    validation scheme as a pivot axis. This paper and
    [[literature/papers/sun2026scipivot]]'s LiveCodeBench case both say
    that an implausible score (94% here, all zeros there) calls for
    auditing the grader before any model change.
  - *Shared substrate.* Grader and agent share one mutable database, and
    the error arose from a second reader of that state. The paper offers
    nothing on correlated errors across agents.
- **Related work in the graph.**
  [[literature/papers/zheng2026benchshield]] states lifecycle invariants
  for the whole run (setup → reward → release), enforced by host probes.
  This paper states invariants for the oracle function alone and validates
  them by mutation. The two are complementary.
  [[literature/papers/zhang2026double]] reaches the same "audit the
  scorer" lesson on its own benchmark. Its fabricated-submission probe is
  CTL-1 under another name. It also found τ-bench's scorer "sound", which
  is consistent here: the upstream runner is fine.
  [[literature/papers/gomez2026can]] has a defective oracle that *hid*
  solves (about 32% → 53% after validation). This one *inflated* them.
  Oracle defects run in both directions.
  [[literature/papers/giampieri2026cortex]]'s "a plant tests only the
  gates it reaches" has the same shape as the INV-4 blind spot.

## Trust signals

- **Credibility:** 3.
  - For: a code and data release (GitHub repo created 2026-10-11, MIT
    license, verified 2026-10-20, containing `artifact/`, `manifests/`,
    `results/` and `vendor/`). Two Zenodo DOIs hold frozen per-episode
    records (232 MB) with SHA-256 manifests, and one offline entry point
    recomputes every statistic with no model calls. The pre-registration
    and deviation log ship in the package. Statistics are careful:
    task-clustered paired intervals, a direct test of each contrast rather
    than overlapping intervals, a power simulation that undercuts the
    author's own contrast, and two denominators reported side by side. The
    write-up is unusually candid about negative results and limits. The
    RQ1 defect has a minimal reproduction and matches τ-bench's code
    structure.
  - Against: a single author, no peer review (v1, 8 days old), and no
    model named. One benchmark with two domains of one oracle
    implementation. The second framework is unnamed. The pre-registration
    is internal. The detection rates come from an author-built battery.
    The secondary results rest on one agent.
  - Not 4: single author, unreviewed, unnamed models, and RQ2/RQ3 rest on
    one agent. Not 2: the frozen records, offline recompute and
    self-limiting analysis are well above the median preprint, and RQ1 can
    be checked from code alone.

## Follow-up

- **Relevance:** 3. It adds a named failure mode for
  [[concepts/programmable-evaluator-oracle]]: a stateful oracle that
  returns the right value and leaves the wrong state, which a second
  reader then turns into wrong labels. It also adds two zero-cost checks
  (repeat-scoring, terminal-state stability) and a do-nothing control that
  HCE clause 4 does not require. It sharpens existing points rather than
  shifting a concept. The graph already has evaluator-audit sources
  (zheng2026benchshield, zhang2026double, farahbakhsh2026agentdiscover).
  The setting is a customer-service tool-use benchmark, not an ML-research
  loop, and the headline inflation is a self-inflicted integration bug.
- **Possible `/elevate` input (not proposed here).** A chain-start
  evaluator pre-flight could check three things: (a) the evaluator scores
  a known-bad (no-op or empty) submission as failing; (b) it scores the
  same artifact twice with the same result; (c) the scored artifact's hash
  is unchanged after scoring. This would complement clause 4's hash pin.
  zhang2026double and gomez2026can are the other evidence for (a).
- **Candidates** (cited, not in the graph):
  - Bellibatlu et al. 2026, "Do agent benchmarks do what they say? An
    executable-contract audit of tool-using agent environments" (arXiv
    2609.37315).
  - Seshadri et al. 2026, "Lost in simulation: LLM-simulated users are
    unreliable proxies for human users in agentic evaluations" (ACL 2026).
  - Khanal et al. 2026, "Beyond pass@1: A reliability science framework
    for long-horizon LLM agents" (arXiv 2603.29231).
  - Bjarnason et al. 2026, "On randomness in agentic evals" (arXiv
    2602.07150), already cited inside [[literature/papers/liu2026what]].
