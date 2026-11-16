---
kind: paper
title: "Agent Self-Improvement with Frozen Weights: The Added Value and Limits of Failure Traces"
authors: ["Arkid Mitra", "Snehil Saluja"]
institutions: ["Hopit AI", "Overlayy"]
year: 2026
venue: "arXiv 2611.11342 (cs.LG), v2 2026-11-11, preprint (6-page short report; an extended version is available only from the authors)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.11342"
code_url: "https://github.com/hopit-ai/harness-mistakes-artifacts"    # named in the PDF, not opened at ingest. Analysis code, pre-registration records, frozen defect lists, derived outcome tables (PolyForm Noncommercial). Experiment kits on request.
citations: null
source: "raw/papers/mitra2026agent.pdf"
added: "2026-11-16"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["failure-traces", "lesson-writing", "harness-notes", "frozen-weights", "no-evidence-control", "matched-search-arm", "neutral-note", "placebo-arm", "randomized-inclusion", "hierarchical-bayes", "per-context-non-inferiority", "harm-bound", "admission-gate", "winners-curse", "held-out-replication", "scorer-audit", "benchmark-defects", "preregistration", "tau2-bench", "bfcl", "qwen3.5-9b", "decision-vs-episode-gap", "negative-result"]
---

# Agent Self-Improvement with Frozen Weights: The Added Value and Limits of Failure Traces

## TL;DR

The agent is a frozen Qwen3.5-9B (thinking off). A larger teacher model
writes short "WHEN situation THEN action" lessons into one notes slot of
its system prompt. The question is how much of a learned harness's gain
needs the agent's failure traces. The control is a **matched
no-evidence search**: same writer, prompts, propose / validate /
keep-or-revert loop, budget ceilings and development feedback, with the
episodes replaced by a fixed sentence. Every primary contrast was
pre-registered in a hash-chained registry.

Four results matter for this graph:

1. **Traces add a little on isolated decisions, and the loop without
   them does most of the work.** On 2,997 sealed τ² retail decision
   cases: standard harness 70.3%, no-evidence search 76.8%, trace lessons
   80.9%. Trace − search is +4.0 [+2.3, +5.7]: it passes superiority but
   sits below the registered 5-point worthwhile gain. The search reaches
   6.5 of the 10.5-point gain (~60%) without reading a failure. All of the
   control's gain is on the one task whose rule was stated in the writer's
   task description.
2. **None of it shows up in full episodes.** In official τ² retail
   (114 tasks × 4 trials), pass^1 is 82.9% standard vs 81.6% with lessons,
   −1.3 [−5.9, +3.3]. The lessons did change the targeted behaviour:
   conversations that changed items before the address fell from 6 to 0.
3. **A strict single-lesson gate is a screen, and its one tested winner
   did not replicate.** Lessons are switched on and off at random, and a
   hierarchical Bayesian model requires gain > 1 point, per-context
   non-inferiority (1-point margin) and bounded harm. On τ² it admitted 2
   of 12 candidates and rejected 4 "ask or confirm again" lessons, each of
   which credibly hurt some context. On BFCL multi-turn **0 of 8 trace
   lessons passed**. The one that passed came from the no-evidence search
   ("when the current directory is unknown, call pwd", +4.9 CrI
   [+1.9, +8.0]). On 120 held-out roots its gain was −0.31 [−3.65, +3.12].
4. **A neutral note beat both lesson sets, and most BFCL items are
   defective.** The neutral note is a fixed note with no procedure. On
   held-out BFCL it scored +4.5 [+0.7, +8.4] over the standard harness
   (registered descriptive secondary; the comparison with the lessons was
   not registered). A blind audit, frozen before outcomes were opened,
   judged 146 of 240 held-out instances (60.8%) and 42 of 60 gate
   instances clearly defective (κ = 0.84). On clean held-out items, both
   lesson arms sit about 6 points *below* the standard harness.

## Claims

- Comparing a learned harness with its starting point mixes **search**
  (propose texts and keep the best on dev) with **evidence** (the writer
  read failures). Only a matched arm with the evidence withheld separates
  them. Even that only partly separates them: the control's actual search
  path can differ, and its gain may come from search, selection,
  restating supplied rules or added text.
- Without the matched search, the decision-level gain would have looked
  2.6× larger (10.5 vs 4.0).
- "Screens are not certifications." A winner selected on one split
  should shrink on fresh data, and one split cannot tell a small true
  effect from selection on noise. τ² noise is large: 33–55% of task
  verdicts flip across four identical trials of published configurations
  (citing tau2-bench issue #540).
- Note effects may be non-specific, so a learned note should be compared
  with a neutral note, not only with no note.
- Benchmark scorers need a blind audit frozen before outcomes are seen.
- Recommended practice: report a matched no-evidence search, compare
  with a neutral note, confirm gate winners on held-out data, and audit
  the scorer blind.
- Explicitly not claimed: that the lessons have no effect (intervals
  allow a few points either way); that the two τ² lessons are certified
  or that their gain comes from traces; generality beyond one agent, two
  benchmarks and one teacher family.

## Methods

- **Agent.** Qwen3.5-9B on vLLM, thinking mode off, weights frozen. The
  standard harness (B1) is each benchmark's reference agent. On τ² the
  optional think tool is added, and learned or control text goes in one
  notes slot of the system prompt. On BFCL a text is one added system
  message. An empty text gives B1 exactly.
- **τ² decision level.** Synthetic conversations cut at one decision
  (address, variant, sequence tasks), scored against fixed answers. 2,997
  sealed test cases. "Wrong write" = a write call that is not exactly the
  correct one.
- **τ² conversation level.** Official retail, 114 tasks × 4 trials,
  τ²-bench v1.0.1 (commit 672227c6). τ-bench 2.0.0 is not
  score-comparable, so the numbers hold for the pinned version. 40 of the
  114 tasks were never used in development.
- **BFCL multi-turn.** 200 roots, each with a Base and a Long-Context
  instance: 50 discovery, 30 gate (60 instances), 120 held-out (240
  instances). Every needed fact is in the request and tools, so asking the
  user cannot replace the right calls.
- **Writers.** On τ², a teacher (Qwen3.8-Flash-Next-FP8) runs propose →
  validate → keep or revert, 12 seeded runs per task, one library per run.
  The writers' task descriptions **stated the target rules**. On BFCL,
  Qwen3.5-397B-A17B reads 198 discovery episodes. The registered teacher
  was replaced before any result.
- **Controls.** τ²: no-evidence search, matched on everything except the
  episodes (a fixed sentence). BFCL: matched writer, prompts, loop, step
  count, budget and screen, but given the **function documentation**
  instead of episodes. Neutral note (BFCL only): "You can call the
  functions provided in this conversation. The user will make one or more
  requests, one turn at a time. Handle each request using the functions as
  needed, then reply."
- **Gate.** Each candidate lesson is switched on or off at random across
  many runs, so its effect is estimated against a random background of the
  other lessons. A hierarchical Bayesian model pools across contexts (task
  × decision type on τ²; domain family on BFCL). To pass, a lesson needs a
  credible gain > 1 point, enough exposure in every context, P(loss > 1
  point) < 0.10 in every context, and bounded harm (wrong write on τ²,
  wrong final state on BFCL). On τ² it must pass under three priors
  (sceptical, weakly informative, slightly negative). τ² gate: 12
  candidates, 20,400 decisions over 1,200 training prefixes. BFCL gate: 60
  instances, 2,820 episodes. BFCL held-out: 240 instances, 5,760
  episodes.
- **Scorer audit.** Two independent model-based reviewers labelled all 300
  gate and test instances from benchmark material only, never from
  trajectories. Defective = a plausible, natural response would have its
  official verdict flipped, and the reviewers judge the case clear. Lists
  were frozen and registered before outcomes were opened. Official scores
  stay primary; audited estimates are reported beside them and never
  choose anything.
- **Execution.** GPU runs were done by an external collaborator from
  tagged experiment kits; the authors held the answer keys and scored.
  Disclosed deviations: the τ² gate data were re-run in full after the
  first run's folder was lost; in that run, read calls were answered by
  the original database in 0.27% of rows (removing them leaves the
  admitted lessons unchanged); BFCL used temperature 0.7, not the official
  0.001; a scorer serialisation fix was applied after the run, with
  verdicts byte-identical wherever they could be compared.

## Results

- **Decision level (Fig. 1a–b, Table 1).**

  | Arm | Accuracy | Wrong writes |
  |---|---|---|
  | B0 model only | 68.6 | 15.5 |
  | B1 standard harness | 70.3 | 15.0 |
  | No-evidence search | 76.8 | 8.2 |
  | Trace lessons | 80.9 | 5.1 |
  | Person-written reference | 80.9 | 2.3 |

  Trace − search: +4.0 [+2.3, +5.7] accuracy, −3.1 [−4.6, −1.7] wrong
  writes. Trace − B1: +10.5 [+8.5, +12.5]. A person-written note matches
  the trace lessons on accuracy and has half their wrong writes.
- **Conversations (Fig. 1c).** pass^1 −1.3 [−5.9, +3.3]; on the 40
  never-used tasks −0.6 [−6.9, +6.3]. pass^4 is 66.7 (B1) vs 57.0
  (lessons), −9.6 [−18.4, −0.9]. That gap is exploratory and confounded,
  because the learned arm used a different library in each trial. The
  no-evidence search was not run at this level.
- **τ² gate.** "Change the address first" +4.7 [+4.0, +5.4], "list only
  the first change" +2.7 [+2.0, +3.4], both +7.4 [+6.3, +8.4] (90% CrIs,
  on the selection data). No held-out test was run. The first lesson
  states the rule the no-evidence search also wrote "almost word for
  word".
- **BFCL gate.** 0 of 8 trace lessons pass; all estimates within ±2.5. The
  search lesson passes at +4.9 [+1.9, +8.0]; on the 18 clean gate
  instances +3.9 [−0.4, +8.3].
- **BFCL held-out.** Standard − search lesson +0.31 [−3.12, +3.65], Holm
  p 1.000; the gate's +4.9 lies outside the interval. Standard − all eight
  trace lessons without the gate −0.52 [−4.38, +3.33], Holm p 1.000.
  Neutral note − standard +4.5 [+0.7, +8.4]; on clean items +3.4
  [−1.9, +9.1].
- **Clean held-out items (94 of 240).** Standard − search lesson +6.0
  [+0.2, +12.0] (Holm p 0.149); standard − trace lessons +6.2
  [−0.5, +13.2] (Holm p 0.153). By the registered rule the official-score
  primary stands.

## Critique / open questions

- **Against the digest framing: at framing for the summary; the "Why"
  line misreads the mechanism.** Every number in the digest summary is in
  the PDF and is quoted correctly. Three things the digest leaves out or
  gets wrong:
  1. **"The selection-only null from zhou2026does, applied to
     lesson-writing, with the same result. Most 'learning' is
     selection."** Not quite. The two papers run different nulls.
     [[literature/papers/zhou2026does]] simulates candidates *identical to
     the seed* through the optimizer's real acceptance rule, which asks
     whether the dev gain is noise picked by the gate. Here the control
     is a *real run* that writes different text with the evidence
     withheld, which asks whether the gain needed the traces. The
     control's +6.5 is measured on 2,997 sealed cases. It is not a
     winner's-curse artifact; it is a real gain on the decision
     distribution, from a writer phrasing a rule its task description
     already stated. And unlike BiPA's +0.1, the trace arm here *beats*
     its control (+4.0, passes superiority). Zhou's pattern (a gate gain
     that vanishes on fresh data) appears only in the BFCL gate: +4.9 →
     −0.31 held-out.
  2. **The writers were told the target rules.** So the decision-level
     result measures whether a teacher can phrase a known rule so the 9B
     follows it, not whether traces let it discover one (the authors'
     own limitation). The digest does not say this, and it is the main
     qualifier on finding (1).
  3. **The one admitted lesson tested on new tasks was not
     trace-written.** It came from the no-evidence search, and 0 of 8
     trace lessons passed the BFCL gate. The abstract says this; the
     digest drops it.
- **What zhou2026does and this paper show together.** They are
  complementary nulls, and neither paper runs the other's. A loop's gain
  needs (a) the identical-candidate replay through its own gate (is the
  gain noise?), (b) a matched no-evidence arm (is it the evidence?), (c) a
  neutral-text arm (is it any text at all?), and (d) held-out confirmation
  of whatever the gate admits. Both agree that the gate's own estimate
  does not survive fresh data. They differ on whether the learned text
  helps at all: zhou starts from a strong hand-written seed with little
  headroom, this paper from an empty notes slot.
- **The clean-item reversal is the most interesting number, and the paper
  does not decompose it.** Our arithmetic, assuming the estimates are
  instance-weighted means: if the all-item difference is +0.31 and the 94
  clean items give +6.0, the 146 defective items give about −3.4 for
  standard − search lesson. For the trace lessons the figure is about −4.9.
  So the lessons gain on items where a correct answer is scored wrong and
  lose on items scored correctly. That is consistent with lessons
  tracking scorer quirks (the paper cites Huang et al. 2026 on learners
  fitting the verifier), but both clean-item contrasts have Holm p ≈ 0.15.
- **Behaviour changed and outcome did not.** The address-ordering lesson
  took the targeted error from 6 conversations to 0, and pass^1 did not
  move. Decision-level accuracy is a proxy whose transfer to episode
  success is itself unmeasured. The gap here is between task levels, not
  between dev and test.
- **The search control on BFCL is not matched on information.** It read
  the function documentation; the trace writer did not. So "only the
  search lesson passes" may reflect documentation, not the lack of traces.
- **The person-written reference is the strongest decision-level arm**
  (equal accuracy, wrong writes 2.3 vs 5.1). The paper does not discuss
  it, and it is absent from Table 1.
- **Thin as published.** Six pages; the extended report with all
  registered analyses, the lesson texts beyond the three quoted, and the
  hierarchical model's specification are not public. The audit uses LLM
  reviewers, so its precision is unmeasured.
- **One citation runs against our note of it.** The paper cites
  [[literature/papers/yu2026recursive]] as agreeing that small open
  models show no improvement. Our note records Recuris improving 35 of 37
  model–benchmark pairs, including τ²-Retail. The citation may refer to
  the two exceptions; we cannot tell from our note.

## Trust signals

- **Credibility:** 3.
  - For: every primary contrast, interval method and threshold
    pre-registered in a hash-chained registry, with run-kit commit hashes.
    Answer keys held by the authors while an external collaborator ran
    GPUs. Analysis code, registry records, frozen defect lists and outcome
    tables released. Intervals and Holm adjustment throughout; three
    priors on the τ² gate; deviations disclosed in detail; a "what we do
    not claim" list; the abstract matches the body.
  - Against: two authors from small companies (Hopit AI, Overlayy) with
    no track record in the graph. Not peer reviewed, five days old, no
    citations. One frozen 9B model, thinking off, one teacher family. The
    extended report and experiment kits are on request only;
    non-commercial licence. The scorer audit is LLM-labelled. BFCL ran at
    temperature 0.7, and the τ² gate was re-run after data loss.
  - Not 4: unknown group and the full analyses are not public. Not 2:
    pre-registration and released analysis artifacts are stronger than
    most preprints here.

## Follow-up

- **Relevance:** 4. It is the first real-run, matched no-evidence arm in
  the graph for lesson writing, and it adds the neutral-note arm and a
  held-out replication of a gate winner. It adds material evidence to
  [[concepts/hce-evaluation]], [[concepts/skill-library-lifecycle]] and
  [[concepts/verified-memory-writes]]. Not 5: a 9B customer-service and
  function-calling agent, not an ML-research loop, and the decision-level
  rule was given to the writers. zhou2026does stays the canonical
  selection-null evidence.
- **For `/elevate`'s evidence gate.** It strengthens the case for the
  accepted known-bad preflight and the paired-control rule. It adds one
  testable rule that no accepted proposal yet carries: a lesson or skill
  edit that claims to come from traces is compared with the same writer
  run without them, and with a neutral text of similar length. One paper
  at credibility 3; any proposal should say that.
- **For this box (our inference).** `/iterate`'s Diagnostics and
  `/ingest`'s concept edits are written from the run's own record. The
  matched no-evidence version is cheap for a skill edit: rerun the editor
  with the trace replaced by a fixed sentence and score both.
- **Candidates** (cited, not yet in the graph):
  - Zhao et al. 2026, "Self-evolving agents can repair failures but
    struggle to generalize from them" (arXiv 2610.22984).
  - Singh et al. 2026, "Agent plasticity" (arXiv 2610.08902). Frontier
    agents that do improve from their own artefacts.
  - Huang et al. 2026, "Learning the task or learning the verifier?"
    (arXiv 2611.08318). Criteria exposure produces verifier-shaped gains.
  - Xu et al. 2026, "SelfImprove-Bench" (arXiv 2611.02797), and Wang et
    al. 2026, "Beneath the surface" (arXiv 2611.07624, digest item 12).
    Regressions on previously solved tasks.
  - ALTK-Evolve (IBM Research blog, 2026-10-20) and altk-evolve issue #426:
    a τ² retail result whose adapter mined test-split tasks by default.
  - tau2-bench issue #540: 33–55% of task verdicts flip across four
    identical trials of published retail configurations.
  - Epoch AI's BFCL v4 review: score-changing defects in 8 of 15 sampled
    multi-turn items.
