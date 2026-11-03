---
kind: paper
title: "Lessons from Luck: Do LLM-Written Lessons Carry Chance into an Agent's Next Decision?"
authors: ["Kazuki Nakayashiki"]
institutions: ["Glasp"]
year: 2026
venue: "arXiv 2610.33029 (cs.LG), v1 2026-10-12, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.33029"
code_url: "https://doi.org/10.5281/zenodo.23253562"    # frozen protocol, code, worlds, prompts; locked run records at doi:10.5281/zenodo.23272809
citations: null
source: "raw/papers/nakayashiki2026lessons.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["memory-writing", "lessons", "outcome-bias", "chance-outcomes", "reasoningbank", "failure-routed-instruction", "decision-quality-note", "no-change-option", "self-registration", "zenodo", "opentimestamps", "betting-test", "difference-in-differences", "same-model-writer-reader", "noise-band", "write-side-noise", "per-model-heterogeneity"]
---

# Lessons from Luck: Do LLM-Written Lessons Carry Chance into an Agent's Next Decision?

## TL;DR

A model writes a lesson from one scripted run whose outcome was assigned by
chance. A fresh session of the same model then picks the first tool for the
next run, holding the full run log and the previous result. After a chance
failure of the better tool, the lesson pushed readers to the worse tool in
**11.6%** of next runs, against **0.4%** with an irrelevant memory item and
**0.4%** with the lesson written from a success (panel interaction P2
+0.112 [+0.097, +0.126], 256 worlds, 11 endpoints). The effect is all in 6
of 11 models. **Claude Opus 5.5, Claude Sonnet 5.5 and GPT-6.1 Sol never
chose the worse tool in that cell** (0/255, 0/256, 0/256). After a lucky
success of the worse tool, the mirror effect was bounded below 0.10
(+0.002). Two writing-instruction changes lowered the panel rate: a
three-sentence decision-quality note to 7.5%, and one outcome-neutral
instruction with a "say so if nothing needs to change" option to 3.1%. Both
met the registered harm-reduction rule on the panel. Neither met it at
every model that carried the chance (3 of 6 and 5 of 6).

## Claims

- An outcome-routed lesson-writing step (ReasoningBank's released
  success/failure instructions) turns a chance failure of a sound decision
  into advice that moves a later decision. No adversary is needed.
- The lesson moves the reader only together with the reported failure.
  Read after a reported success, the same failure-written lesson gave a
  0.2% worse-tool rate (L2b +0.001 [−0.001, +0.003]).
- A lesson can outweigh the record it summarises. Every reader held the
  authoritative log in which the better tool stayed better.
- Most of the effect sits where the log does not say that failures are
  independent (descriptive split; not a test of the sentence).
- The two kinds of luck differ. A chance failure of a good decision
  propagates; a lucky success of a bad decision did not, on the panel.
- The same pipeline behaves very differently across models.
- A planted-directive test alone would miss the effect: two models ignored
  an unreasoned "use the worse tool" directive but still carried the
  chance from their own lessons.
- The paper claims no mechanism and no prevalence in real agents.

## Methods

- **Worlds.** 256 synthetic recurring operational tasks (16 domains × 16).
  Each has two interchangeable tools of equal cost that differ only in
  recorded reliability: better tool 0.80–0.93, worse 0.50–0.70, gap at
  least 0.15. The log shows the last 180–365 runs of each tool. The
  generator guarantees the better tool stays better after any single
  further outcome. Half the worlds (128, "explicit") add one sentence
  saying failures are independent one-off events; the other half
  ("implicit") show counts only.
- **Scripted previous runs.** Four per world: good or bad decision ×
  success or transient non-diagnostic failure. The good decision states
  its reason ("the more reliable tool"); the bad decision gives none.
- **Writers.** Three instructions, each producing up to three memory items:
  - **RB**: ReasoningBank's success and failure instructions, verbatim from
    the released code (commit and SHA-256 given), with two declared
    replacements. The released failure instruction asks for "concrete,
    actionable recovery procedures"; it differs from the printed paper in
    three places.
  - **RBD**: RB plus one bullet: judge whether the decision was sound
    given only what was known before the outcome; one outcome is not by
    itself evidence; recommend a different action only if the decision was
    unsound.
  - **N**: one instruction for both outcomes, with the outcome-specific
    phrases made neutral and "If nothing about how the task should be done
    needs to change, say so." N vs RB is a compound comparison (no routing
    and an explicit no-change option).
- **Readers.** A fresh session of the **same model** as the writer. It
  sees the task, the authoritative log, the previous-run line ("X was used
  and it failed/succeeded"), one memory item, and must name `first_tool`
  in structured JSON. Nothing is executed. Eight memory arms: matched
  lesson (RB, RBD, N), swapped lesson from the other outcome (RBx, RBDx,
  Nx), NEUTRAL (true irrelevant fact) and PLANTED (unreasoned "use the
  worse tool" directive). 32 reader requests per world and endpoint.
- **Panel.** 11 endpoints through OpenRouter, each pinned to one provider
  without fallbacks: Claude Opus 5.5, Claude Sonnet 5.5, Claude Haiku 5.5,
  GPT-6.1 Sol, GPT-6 Luna, Gemini 3.8 Flash, Grok 4.7, Qwen 3.8 Max,
  DeepSeek V4 Pro (served by Alibaba), Mistral Large 4, Kimi K3. Provider
  default sampling; no sampling or reasoning parameters sent. One
  generation per cell. All calls on 9 October 2026. Each endpoint wrote
  the lessons its own readers read, so writing and reading effects cannot
  be separated.
- **Estimands.** P2 (good decision) is the outcome × memory
  difference-in-differences, RB arm minus NEUTRAL arm. P3m is the mirror
  for the bad decision. Qualifiers: A2 (lesson vs irrelevant item in the
  failure cell) and L2 (failure-written vs success-written lesson in the
  failure cell). Intervals are Student-t over the 256 worlds. A detection
  needs Holm-adjusted p < 0.05 on both a t test and a finite-sample betting
  test (Waudby-Smith & Ramdas). "Bounded below 0.10" needs the t upper limit
  below 0.10 and a betting p ≤ 0.025. Endpoint readings are exploratory.
  Remedies are credited only if both their P2 and A2 differences vs RB have
  95% intervals below 0.
- **Registration.** Self-registered, not in a third-party registry. A
  74-file package (registration, code, worlds, prompts, analysis code,
  pilot records, design reviews) was frozen under a SHA-256 manifest,
  deposited publicly on Zenodo (doi:10.5281/zenodo.23253562) and
  OpenTimestamps-attested in Bitcoin block 970581 (2026-10-09 04:23 UTC)
  before any confirmatory call. The launcher refused to call any model
  unless it re-verified the manifest, the attestation and the public
  deposit. The run was launched once and analysed once through a
  completion manifest also timestamped (block 970623). Locked records are
  on Zenodo (doi:10.5281/zenodo.23272809).
- **Pre-freeze changes from a development pilot (disclosed).** Five
  development runs on other seeds shaped the design. After the second
  attempt's descriptive results were read, the memory-only reader was
  dropped, the whole writer response became the stored memory, and P3m
  and the planted control were added. The world-level cue came from a
  writer that repeated the independence statement. No pilot data enter
  the analysis. Protocol reviews were done by LLM reviewers.
- **Operating characteristics (simulated before the run).** Under null
  scenarios a false detection of either primary occurred in at most 0.6%
  of replications. The t rule alone reached 10.6% in the heterogeneous
  null, which is why the betting test is required. Per endpoint, a true P2
  of 0.10 was detected in 96.8% of replications at background discordance
  0.02 and 56.3% at 0.10.

## Results

- **Run.** 33,792 writer requests gave 33,789 lessons (1 refusal, 2 empty).
  90,106 reader requests gave 90,084 valid answers and 22 unparsed. No
  stop, exclusion or contract violation. Cost 543.41 USD.
- **Primary (panel, RB).**

  | Contrast | Estimate [95% CI] | Reading |
  |---|---|---|
  | P2 (good decision) | +0.112 [+0.097, +0.126]; Holm p_t 1.8e-36, p_bet 7.0e-5 | luck-carrying |
  | A2 | +0.112 [+0.097, +0.127] | robustly > 0 |
  | L2 | +0.112 [+0.098, +0.126] | robustly > 0 |
  | P3m (bad decision) | +0.002 [−0.001, +0.005]; Holm p_t 0.252 | bounded below 0.10 |
  | Planted increment | +0.137 [+0.130, +0.143] | above 0.10 |

  Panel worse-tool rates in the good-decision failure cell: RB 11.6%,
  RBx 0.4%, RBD 7.5%, RBDx 0.6%, N 3.1%, Nx 0.4%, NEUTRAL 0.4%,
  PLANTED 17.6%. In the bad-decision success cell: RB 0.4%, NEUTRAL 0.3%.
- **Per endpoint (exploratory; P2 with worse-tool counts in the failure
  cell under RB).**

  | Endpoint | P2 [95% CI] | RB failure-cell count | Reading |
  |---|---|---|---|
  | Claude Opus 5.5 | 0.000 [−0.029, +0.029] | 0/255 | bounded |
  | Claude Sonnet 5.5 | 0.000 [−0.029, +0.029] | 0/256 | bounded |
  | Claude Haiku 5.5 | +0.047 [+0.019, +0.075] | 13/256 | bounded |
  | GPT-6.1 Sol | 0.000 [−0.029, +0.029] | 0/256 | bounded |
  | GPT-6 Luna | +0.031 [+0.010, +0.053] | 8/256 | bounded |
  | Gemini 3.8 Flash | +0.125 [+0.084, +0.166] | 32/256 | luck-carrying |
  | Grok 4.7 | +0.105 [+0.068, +0.143] | 27/256 | luck-carrying |
  | Qwen 3.8 Max | +0.352 [+0.286, +0.417] | 100/256 | luck-carrying |
  | DeepSeek V4 Pro | +0.277 [+0.222, +0.333] | 71/256 | luck-carrying |
  | Mistral Large 4 | +0.166 [+0.120, +0.212] | 43/254 | luck-carrying |
  | Kimi K3 | +0.125 [+0.084, +0.166] | 32/256 | luck-carrying |

  Haiku and Luna had t intervals above zero and robustly positive A2/L2,
  but betting p (Holm) was 0.051 and 0.157, so they stay "bounded". One
  Haiku answer whose `reason` contradicts its `first_tool` decides that
  verdict. P3m was bounded at all 11; DeepSeek came closest (+0.031
  [+0.010, +0.053]; 8/256 vs 0/256 with the irrelevant item).
- **Remedies (secondary).** On the panel, both met the harm-reduction rule.

  | Remedy | P2 diff vs RB | A2 diff vs RB | Failure-cell rate | Residual P2 | Credited at carrying endpoints |
  |---|---|---|---|---|---|
  | RBD (decision-quality note) | −0.039 [−0.053, −0.026] | −0.041 [−0.054, −0.027] | 7.5% | +0.072 [+0.061, +0.082] | 3/6 (Gemini, Grok, DeepSeek) |
  | N (one instruction, no-change option) | −0.085 [−0.097, −0.073] | −0.086 [−0.098, −0.074] | 3.1% | +0.027 [+0.020, +0.034] | 5/6 (not Mistral) |

  By endpoint, failure-cell rates NEUTRAL / RB / RBD / N were: Gemini
  0.0 / 12.5 / 0.0 / 0.4%; Grok 0.0 / 10.5 / 2.0 / 6.3%; Qwen
  4.3 / 39.1 / 33.1 / 6.6%; DeepSeek 0.0 / 27.7 / 16.0 / 3.1%; Mistral
  0.0 / 16.9 / 18.4 / 15.2%; Kimi 0.0 / 12.5 / 10.5 / 0.4%. RBD made
  Mistral slightly worse (+0.020 [−0.045, +0.085]). No registered test
  compares the two remedies.
- **Independence cue (secondary, descriptive).** P2 +0.042 [+0.031,
  +0.053] in explicit worlds vs +0.181 [+0.160, +0.203] in implicit worlds.
  This compares different worlds; it is not a test of the sentence's
  effect. In implicit worlds a worse-tool choice is a deviation from the
  log-based benchmark, not a clear error.
- **What the lessons said.** The share of lessons naming the worse tool did
  not predict the reading: 67% of Claude Opus 5.5's failure lessons named it,
  with zero effect; Grok's named it 0% of the time and carried the chance.
  The three printed lessons that moved readers recommend switching to the
  other tool after an error *within a run*; the reader applied it to the
  next run's first choice. The printed Opus lesson says "One failure is
  expected variance (roughly 1 in 8 runs) and does not show the selection
  was wrong."
- **Planted directive.** Carried by 4 endpoints (Gemini, Qwen, DeepSeek,
  Mistral). Kimi and Grok named the worse tool after it in 0 and 4 of
  1,024 answers, yet both carried the chance from their own lessons.
- **Checks.** A separate agent re-implemented the analysis from the
  registration alone: 3,480 numbers agreed within 1.5e-15, 0 of 36
  readings disagreed. It did not recompute the bootstrap, cue splits or
  descriptives. Results were also reviewed by Codex and a separate Claude
  instance.

## Critique / open questions

- **Against the digest framing: at.** Every number in the digest entry is
  in the body and quoted correctly (256 worlds, 11 endpoints, 90,084
  answers, 11.6% vs 0.4%, +0.112 [+0.097, +0.126], +0.042 / +0.181, 7.5%,
  3.1%, "neither met the registered harm-reduction rule at every
  endpoint"). It is one of the few entries that reports a limit of its own
  remedy. Three omissions matter for this box:
  1. **The effect is model-specific, and the models this box runs show
     none.** Opus 5.5 and Sonnet 5.5 gave 0/255 and 0/256. The 11.6% is a
     panel mean driven by six non-Anthropic, non-OpenAI-flagship endpoints.
  2. **The cue split reads as causal in the digest ("the effect is
     +0.042 when the log states…").** The paper registers it as
     descriptive, across different worlds. Mild.
  3. **The remedies did meet the rule on the panel.** The digest gives
     only the per-endpoint shortfall, and omits that RBD failed at half
     the carrying endpoints while N failed at one.
- **Registration and deviations are unusually well documented.** The
  protocol was frozen, deposited on Zenodo and Bitcoin-timestamped before
  any call, and the launcher checked the deposit before running. It is a
  self-registration, not a third-party registry, and the protocol was
  reviewed only by LLMs. Two deviations are reported. (1) The report's
  tables were generated before the registered independent
  re-implementation comparison; they were regenerated after it, with no
  disagreement. (2) The registration calls the remedy "one-sentence"; it
  is three sentences. The bigger disclosed caveat is that the pilot
  reshaped the design (reader, memory format, P3m, planted control) after
  descriptive results were read, before the freeze.
- **The setting favours the reader.** The reader holds a long,
  authoritative, "stable for months" log with a gap of at least 0.15
  between tools, and the better tool stays better after any single
  outcome. That is the easiest possible case for ignoring a lesson. A
  reader without the log, or with a noisy metric and no clear rate (an ML
  experiment chain), was not tested. The effect could be larger there, or
  the "correct" answer could be unclear.
- **One lesson, read once.** No retrieval among many items, no
  accumulation, no consolidation, no execution. Accumulated lessons could
  amplify or dilute the effect.
- **Writer and reader are the same model.** The per-model spread cannot be
  attributed to writing or reading. Claude's lessons named the worse tool
  67% of the time with no effect, which hints that reading matters, but
  that is a single descriptive.
- **The two routes are not separated.** The printed carrying lessons give
  a within-run recovery rule (switch tools after an error), which readers
  misapplied across runs. That is partly a scope error in the lesson, not
  pure outcome bias. The study cannot say which dominates.
- **Remedies were tested only with non-diagnostic outcomes.** No outcome
  signalled a real reliability change. N's "nothing needs to change"
  option could also suppress learning from a tool that really broke. This
  is the over-correction risk any adoption would carry, and it is
  untested.
- **The good and bad decisions differ in more than quality.** Only the
  good decision states a reason, so the asymmetry between P2 and P3m
  could partly come from that.
- **Statistics are careful but not replicated.** One generation per cell,
  intervals over worlds, endpoint readings unadjusted (about a 6% chance
  of one false endpoint detection), one day of serving. The finite-sample
  betting test is the right conservative choice and it changed two
  endpoint verdicts.
- **Fit with the graph.** This is the memory-write analogue of
  [[literature/papers/hu2026winners]]'s winner's curse: there a lucky
  score becomes the incumbent; here a lucky outcome becomes a durable
  lesson. It adds a benign, true-record failure to the write-side cluster
  next to [[literature/papers/khezresmaeilzadeh2026claim]] (records
  written from intent) and [[literature/papers/pham2026reliable]]
  (usefulness tags that do not track effect). Here the record is true —
  the failure did happen — and the error is in what the writer infers from
  it. It also qualifies [[literature/papers/li2026beyond]]'s "distilled
  lessons beat raw history": with the raw record in hand, a distilled
  lesson overrode it at 6 of 11 models. Same author as
  [[literature/papers/nakayashiki2026when]], with the same frozen-protocol
  discipline; that paper is about reading stale inherited memory, this
  one about writing noisy new memory.

## Trust signals

- **Credibility:** 3.
  - For: frozen, publicly deposited and Bitcoin-timestamped protocol with
    a launcher that enforced it; one launch, one analysis; locked records
    and analysis code public on Zenodo; finite-sample betting tests on top
    of t tests, with simulated operating characteristics; missing-data
    bounds; an independent re-implementation that matched to floating-point
    precision; disclosed deviations, pilot influence, a `first_tool`/reason
    disagreement that decides one verdict, and a long limitations section
    that matches the body. ReasoningBank's instructions are taken verbatim
    from the released code, with the diffs to the printed version listed.
  - Against: single author at a small company (Glasp), not peer reviewed,
    no human reviewer at any stage (protocol, results and re-implementation
    were all done by LLM agents; the author used Claude Code for most of
    the work). One generation per cell, one day, no replication.
    Self-registration, not a third-party registry.
  - Not 4: no peer or human review and no replication. Not 2: the
    reproducibility package and pre-commitment are stronger than most
    lab preprints in this graph. Same score as
    [[literature/papers/nakayashiki2026when]], for the same reasons.

## Follow-up

- **Relevance:** 4. It is the first controlled measurement in this graph of
  chance entering memory at the *write* step, with the decision and the
  information held fixed. It adds material evidence to:
  - [[concepts/verified-memory-writes]]: a true record plus an
    outcome-routed writer gives a false lesson, which coverage /
    preservation / faithfulness gates would pass.
  - [[concepts/skill-library-lifecycle]]: failure-routed distillation
    ("concrete, actionable recovery procedures") is the risky half, and a
    no-change option is the write-side "keep none as a candidate".
  - [[concepts/selective-memory-retrieval]]: a lesson can outweigh the
    authoritative record it summarises.

  Not 5: one lesson read once, synthetic two-tool worlds, and zero effect
  at the Claude models this box runs.
- **For the noise band (accepted 2026-09-27).** This is its write-side
  twin. The band stops a sub-noise delta from resetting the
  no-improvement counter. It does not stop `/implement`'s Diagnostics
  (`delta_from_prior … attributed to <cause>`, `next_candidates`) from
  turning the same single-run delta into a durable causal claim that the
  next `/iterate` cycle reads. That is our inference, not the paper's
  setting. On this paper's evidence the exposure is low for Opus 5.5 in a
  clean setting, so it is not grounds for a proposal. If it were ever
  tested, the cheap version is RBD's wording in the Diagnostics
  instruction ("one run is not evidence that the change helped or hurt;
  attribute a cause only beyond the band"), and the risk to watch is N's:
  suppressing a real signal.
- **Known-bad preflight (pending).** The paper's instrument is a model for
  that proposal's shape: a null control (irrelevant item) and a positive
  control (planted directive) around the real arm. Its finding (iv) is a
  caution for it: a positive control that a system resists does not show
  the subtler failure is absent (Kimi, Grok).
- **Candidates** (cited, not yet in the graph):
  - Nakayashiki 2026b, "Plan pointers and record-directive form in budgeted
    verification of inherited agent memory" (arXiv 2609.03450).
  - Ouyang et al. 2025, ReasoningBank (arXiv 2509.25140): the writing
    instructions tested here.
  - Wang et al. 2026d, "Phantom guardrails: when self-improving agent
    harnesses fix failures that never happened" (arXiv 2607.13083).
  - Ye et al. 2026, "On the fragility of self-improving agents"
    (arXiv 2608.18066; cited by hu2026winners).
  - Zhu et al. 2026, "Escaping the self-confirmation trap:
    Execute-Distill-Verify" (arXiv 2606.24428).
  - Wang et al. 2026b, MemGuard (arXiv 2608.21867).
