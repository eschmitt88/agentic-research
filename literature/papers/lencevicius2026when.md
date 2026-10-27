---
kind: paper
title: "When the harness decides success: state that cannot be re-observed"
authors: ["Raimondas Lencevicius"]
institutions: ["Microsoft"]    # PDF author block; the paper states the work reflects the author's own views, not Microsoft's
year: 2026
venue: "arXiv 2610.26632, v1 2026-10-19, preprint (14 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.26632"
code_url: "https://github.com/raimondasl/persistgym"
citations: null
source: "raw/papers/lencevicius2026when.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/context-proprioception]]"
tags: ["truncation", "sliding-window", "compaction", "summary", "note-taking-instruction", "reasoning-replay", "re-observability", "recoverable-vs-not", "derived-vs-observed", "prompt-caching", "cache-break", "arc-agi-3", "harness-gap", "persistgym", "pre-registered", "amendments", "single-run", "run-to-run-variance", "gpt-6-luna", "claude-haiku-5.5", "gemini-3.8-flash", "abstention"]
---

# When the harness decides success: state that cannot be re-observed

## TL;DR

A single-author, pre-registered study in two parts.

1. **A synthetic probe (PersistGym-lite).** A fact needed at a late step is
   either shown once or derived from clues, and can either be looked up
   again with a lookup action or not. With a truncated window, gpt-6-luna
   loses nothing in success when it can look the fact up (100%, 201
   lookups in 200 episodes). When it cannot, it falls to 7.5% (15/200),
   against 10% chance. The difference between the two penalties is
   +92.5 points (90% CI 89.5–95.5, 200 templates, confirmatory, Holm
   across H1 and H3). A replication on claude-haiku-5-5 gives the same
   contrast (0/200 vs 200/200), but it sits outside the confirmatory
   family. The paper says itself that the success penalty is expected by
   construction. The finding is the contrast with the recoverable case.
2. **A decomposition of ARC-AGI-3's harness gap.** One model (Gemini 3.8
   Flash, high thinking), four games picked for a large published gap, one
   counted run per configuration and game. Going from the Standard harness
   (drop oldest turns, notes line in the prompt) to summary compaction
   with the notes line takes the mean score from 9.24 to 63.08. The
   Provider Adapter, with native reasoning replay, scores 71.83. So the
   summary plus notes step recovers 86% of a 62.6-point gap. Removing the
   notes line drops the mean to 36.32, and replay recovers that and more.
   The replay share (57%) hinges on one run that was replaced under an
   amendment adopted after outcomes were seen. Under the frozen rules
   the share is 35% and neither registered prediction holds.

Two side results matter more for this graph than the headline. Dropping
turns is cheaper in tokens but 2.9–4.0× more expensive in dollars, because
it breaks the prompt cache (94% cached input under full history against
48% under the window). And a summary with a notes line carries a derived
fact far better than an observed one (92% vs 43% success).

## Claims

- Harness context policies are "chosen for cost and latency, on the
  implicit assumption that a capable model can recover whatever it
  loses". That fails when the needed information was derived during the
  episode or shown once and cannot be shown again. Then "the harness, as
  much as the model, decides whether the agent succeeds."
- H1 (confirmatory): truncation costs success when the fact cannot be
  re-observed, and costs only effort when it can. Supported.
- H3 (confirmatory control): multi-turn structure itself costs at most a
  few points when the fact survives. Supported, but both arms sit at
  ceiling, so it bounds the cost for this model and task and does not
  show it is zero.
- H4: on ARC-AGI-3, the loss policy (drop vs summarize), the note-taking
  instruction, and native reasoning replay each carry a share of the
  Standard-to-Adapter gap. On the point estimates, after amendment R7,
  prediction B (replay carries at least half) holds and prediction A
  (loss policy large, replay at most half, notes near zero) does not.
  Under the rules as frozen before R7, neither holds.
- H5: the Adapter's advantage in clearing a level, once reached, is
  larger at deep levels than at early ones. The author says this "largely
  restates where runs stopped" and does not separate a growing advantage
  from a constant one.
- Neither prior work it cites (Fan et al. 2026, Mehta & Datta 2026,
  LOCA-bench) separates information the agent can look up again from
  information it cannot. The paper claims to isolate that case.
- Nothing in the paper ranks models or providers.

## Methods

- **PersistGym-lite episodes.** A sequence of steps. *Routine* steps
  depend only on the current observation. A *reveal* step shows the
  target fact (observed), or shows clues across steps plus the rule that
  combines them (derived). A later *need* step asks for an action whose
  parameter is the fact, from K = 10 options. In the recoverable variant
  a *lookup* action re-shows the fact or clues at the cost of one step.
- **Task cells.** Observed/derived × recoverable/not × near/far (whether
  the evidence lies inside or outside the truncated window at the need
  step), 8 cells. T = 200 templates, one episode per template per cell,
  and every episode is played under every harness.
- **Harness cells (-0 = no reasoning replay).**
  - F-0: full history.
  - W-0: a window of the most recent turns, W = 24,000 tokens, oldest
    dropped.
  - W-n-0: W-0 plus the one-sentence notes instruction, "Include any
    context you want to carry forward in your reply, along with the
    action you want to take." This is the sentence ARC-AGI-3's Standard
    harness uses.
  - Σ-n-0 (exploratory): dropped turns are replaced with a model-written
    summary, and the notes instruction is kept.
  - Controls: a single-prompt flattening of an F-0 episode (H3), and a
    restated-at-need ceiling.
- **Validity gate (before the sweep).** With the evidence restated at the
  need step, 160/160. With it withheld, 19/160 (11.9%, gate 20%). The
  programmatic oracle scored 100% and no memory-free heuristic beat 10%.
- **Model and protocol.** gpt-6-luna at medium reasoning effort on
  OpenAI's Flex tier. 200 templates at repeat 0, 20 of them again at
  repeat 1. Amendment R6 capped T at 200. The power rule would have set
  290, and at T = 200 simulated power at the worst Holm step was 0.65 for
  H1 and 0.78 for H3, below the registered 0.80. The test is a one-sided
  template-cluster bootstrap (10,000 resamples), Holm across H1 and H3. A
  model check uses logistic regression with template fixed effects, and
  Firth's penalized fit where a cell has no variation.
- **Replication (amendment R8, 2026-10-18).** claude-haiku-5-5 at medium
  effort with adaptive thinking, the same 200 templates in F-0, W-0 and
  both controls. W-n-0 was dropped by a cost rule. R8 was written after
  gpt-6-luna's results were known and before any haiku output. Its tests
  are unadjusted and carry no confirmatory label.
- **ARC-AGI-3 configurations.** Gemini 3.8 Flash at high thinking, on the
  benchmark's own harness (upstream commit 4ca00e6 plus 18 patches).
  - S0, Standard: visible text only, notes line in the prompt, oldest
    turns dropped past 175k characters.
  - S1: as S0, but old turns are replaced with a model-written summary at
    175k provider tokens.
  - S2: S1 without the notes line.
  - S4, Provider Adapter: native replay of reasoning items with thought
    signatures, the same summary trigger, no notes line.

  The path S0 → S1 → S2 → S4 changes one component per step. Shares are
  each step's change in mean RHAE divided by the gap G, so they sum to 1.
- **Games.** FT09, SB26, R11L and CD82. A registered rule picked them as
  the only public games with two published high-thinking runs each
  costing at most $60, a published gap of at least 20 points, and a
  Standard run that ended normally. One counted run per configuration and
  game, under the 12-hour clock and the per-level action budgets. A run
  that ended on an API or agent error was re-run once.
- **Registration.** PREREG.md v1.0 (2026-10-01) was frozen before the
  first paid run. Amendments R1–R8 and a decisions log are dated, with
  reasons (Table 3). Several came after some results were seen.
- **Cost.** $750.11 in total: probe $73.75, replication $129.69, ARC
  runs $507.96, pilots $38.71.
- The paper discloses that the code, analyses and drafts were produced
  with Claude Code under the author's direction.

## Results

- **H1, gpt-6-luna (Fig. 1, Table 1).** In the observed, far cells: F-0
  200/200 both ways. W-0 15/200 when the fact is not recoverable, 200/200
  when it is. ΔH1 = +92.5 [89.5, 95.5]. Firth coefficient +7.09, and a
  Bayesian random-intercept model gives a posterior median of 5.25 [2.86,
  7.52]. Only the not-recoverable far cells separate the harnesses.
  Every near cell and every recoverable cell is at or near 100% under
  every harness.
- **Notes and summary, not-recoverable far cells (Fig. 1).**

  | Harness | Observed fact | Derived fact |
  |---|---|---|
  | F-0 | 100.0% | 99.5% |
  | W-0 | 7.5% | 8.0% |
  | W-n-0 (window + notes) | 28.5% | 26.5% |
  | Σ-n-0 (summary + notes, exploratory) | 43.0% | 92.0% |

  Under W-n-0, the notes held the fact in 21.5% of observed episodes, and
  the model then used it every time. So in this cell the loss comes from
  what is carried, not from using what is carried. Under Σ-n-0 the
  summary or notes held the fact in 92% of derived and 38.5% of observed
  episodes.
- **Effort and cost of the window.** In the recoverable case, need-step
  and lookup tokens were 1.31× full history's (45,875 vs 35,088 per
  episode). The registered near-episode ratio was 1.99× [1.97, 2.01],
  against a bar of 1.30. The window cost 2.9–4.0× as much as full history
  in dollars in every task cell while using 0.84–0.90× the tokens.
  The reason is caching: 94% of full history's input tokens were served
  from cache, against 48% for the window, and cached input is a tenth of
  the price. In the not-recoverable far cells the lossy harnesses spent
  3.7–4.3× the reasoning tokens of full history.
- **H3.** Single prompt 400/400 vs conversation 399/400, +0.25 [0.0,
  0.75]. Only one template shows any failure, so the model check adds
  nothing.
- **Disclosure.** H1 and H3 first read "inconclusive" because of code
  limits found after the results were read. The Firth step cap was raised
  from 100 to 1,000, and H3's check was allowed to fit one template. Both
  labels are reported.
- **Replication, claude-haiku-5-5.** W-0 not recoverable 0/200,
  recoverable 200/200. F-0 200/200 and 198/200. ΔH1 = +101.0 [100.0,
  102.5], unadjusted p < 0.0001. ΔH3 = −0.25 [−0.75, 0.0]. The larger ΔH1
  is not a larger memory loss. When the window had dropped the fact, haiku
  never picked any of the 10 options, in all 400 not-recoverable far
  episodes. gpt-6-luna picked an option in 387/400 and was right in 31
  (7.75%, near chance). Haiku also refused some steps (refusal category
  "bio", 1.1–1.6% of multi-turn episodes), 13 of 14 templates in the
  biobank setting. Refusals score 0. The window again cost 3.3–3.7× in
  dollars at 0.84–0.90× the tokens, because this provider bills a
  rewritten prompt as cache writes at 1.25× input.
- **ARC-AGI-3 (Table 2).**

  | Game | S0 | S1 | S2 | S4 |
  |---|---|---|---|---|
  | FT09 | 18.11 | 100.00 | 18.06 | 100.00 |
  | SB26 | 5.44 | 100.00 | 100.00 | 100.00 |
  | R11L | 6.45 | 11.21 | 14.29 | 14.29 |
  | CD82 | 6.94 | 41.11 | 12.95 | 73.02 |
  | Mean | 9.24 | 63.08 | 36.32 | 71.83 |

  G = 62.59 [29.51, 87.44]. σ_loss = 0.860 [0.541, 1.000], σ_notes =
  −0.427 [−0.879, 0.011], σ_replay = 0.567 [0.000, 0.960]. With four games
  there are only 35 distinct bootstrap resamples, so the intervals are
  coarse. The path is not monotone. The negative notes share inflates the
  other two.
- **How firm the ARC result is (the author's own list).**
  - FT09 S2's counted run is the R7 replacement (18.06). The run counted
    under the frozen rules scored 71.43. With that run, S2 recovers 65%
    of the gap instead of 43%, σ_replay falls to 0.354, and neither
    prediction holds. Leaving FT09 out gives σ_replay 0.357. Leaving CD82
    out gives 0.445.
  - FT09 S0 ran twice under the same configuration (a latency re-run) and
    scored 58.17 and 18.11, 40 points apart.
  - R7 raised the summary output budget from 8,192 to 65,536 tokens
    partway through. At high thinking the model's reasoning had used up
    the 8,192-token budget, and summary requests came back empty four
    times in a row. FT09 and SB26 S1 ran under the old budget and their
    S2 runs under the new one (R11L the reverse), so the S1 → S2 step
    also changes the budget in three games. Upstream later removed the
    cap for the same failure (Mazur 2026, PR 56).
  - Configurations ran on different days against a model with no dated
    snapshot.
  - Scoring each unit's original run gives G = 52.57 and σ_replay =
    0.671. At a matched 150 actions (R5, added after S0's first run hit
    the clock), G = 53.95 and σ_replay = 0.711 (0.464 with FT09 S2's
    replaced run).
- **H5.** On the benchmark's 25 published games, Standard cleared 39/47
  early levels it reached and 15/26 deep ones. The Adapter cleared 47/50
  and 63/75. C = +0.30 [0.16, 0.46], or +0.22 [0.05, 0.41] after
  dropping levels where a run ended on an error. On the four games run
  here, S0 cleared 0/2 deep levels and S4 11/11. On the logit scale the
  published difference is not distinguishable from zero.
- **Exploratory, compaction and unusable replies.** Summary
  configurations compact every 5.6–6.9 steps. Unusable replies (empty
  text, or a call to a tool not offered) occurred 22 times in S1 and 17 in
  S2, and never in S4 (737 steps). In S1 they were more frequent in the
  five steps after a compaction (3.4% of calls vs 1.3%). S2 showed no such
  pattern. The analysis was specified after most endings had been seen and
  is confounded with time into the run. The author reports it as a lead.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest's
  numbers are all in the PDF, and it flags the single runs. Four things
  it leaves out change how the result should be read.
  1. *"Truncation costs nothing when a fact can be looked up again"* is
     true for success only. In dollars the window cost 2.9–4.0×
     (gpt-6-luna) and 3.3–3.7× (haiku) because it breaks the prompt
     cache, and the lookup adds 1.31× need-phase tokens.
  2. *The 92.5-point gap is largely a manipulation check.* The paper
     says the success penalty "is expected by construction and only
     confirms that the window removed the fact". With W = 24k tokens and
     far evidence outside it, the not-recoverable cell has nothing to
     recover from. The informative part is that the recoverable cell
     costs zero, which shows the model actually uses the lookup (201
     lookups in 200 episodes).
  3. *"Replicated on a second provider"* overstates the paper's own label.
     It is a replication outside the Holm family that "neither confirms
     nor refutes", registered after the first model's results were known,
     and its larger Δ comes from abstention, not memory.
  4. *"Summary + notes recovers 86%"* is a point estimate from four games,
     one run each, picked for a large gap. A same-configuration rerun
     moved one game by 40 points. Which registered prediction held
     depends on a run replaced under an amendment adopted after outcomes
     were seen.
- **The two ingest checks.**
  - *Probe design.* The design is clean. Episodes are identical across
    harnesses, near/far and observed/derived are crossed, there is a
    validity gate, an oracle and memory-free baselines, and a
    restated-at-need ceiling. The limits: a K = 10 forced choice, a lookup
    that re-shows exactly the needed fact at one step's cost (an ideal
    address), one window size, and one model per provider. It shows that
    a single re-observable fact costs nothing. It does not show what
    happens when re-observation is partial, expensive or noisy, or when
    the agent has to know *that* it lost something before looking.
  - *How much rests on single runs.* All of the ARC part. The probe rests
    on 200 templates × 1 episode per cell with 20 repeats, which is
    adequate for a 92-point effect. The notes and summary numbers in the
    not-recoverable cells are descriptive or exploratory. The ARC shares
    have intervals that span most of [0, 1] (σ_replay [0.000, 0.960]),
    and the rerun spread on FT09 S0 is of the same order as the effects.
- **What the probe adds to the graph.** It is the matched test that
  [[concepts/lossless-context-offload]]'s scope condition asked for, from
  the other side. [[literature/papers/fan2026empirical]] found that an
  explicit recall handle adds nothing in a coding workspace, where the
  environment can re-derive the content. Here the recoverable arm costs
  zero success, and the non-recoverable arm falls to chance with
  everything else held fixed. Together they say an address only matters
  for content the environment cannot regenerate. This is consistent with
  [[literature/papers/nguyen2026cliffcompaction]] (the re-issued call as
  the address) and [[literature/papers/shen2026what]] (irreversible vs
  recoverable errors).
- **Derived beats observed under a summary (92% vs 43%).** The paper
  does not explain this. Our inference: a fact the model worked out
  appears in its own reasoning and replies, so a summarizer or notes
  writer treats it as task state. A fact merely shown in an observation
  looks like scenery until it is needed. That fits
  [[literature/papers/xu2026what]], where fixed fields carried more
  non-inferable state than a prose summary, and item 18 of the same
  digest (arXiv 2610.31778), where compaction drops rules and
  constraints first. Here the not-recoverable observed fact is the
  closest analogue of a constraint stated once at the top of a run.
- **The notes line works when the model uses it, and it rarely does.**
  Under the window the notes held the fact in 21.5% of episodes, and when
  they did, success was 100%. So the instruction is a carry channel the
  model under-uses, not a weak one. This bears on
  [[concepts/constraint-pinning]]: an invitation to carry state is not a
  pin. On ARC, removing the notes line cost 27 points on the mean, in a
  step confounded with the summary-budget change in three of four games.
- **Haiku's abstention is a proprioceptive signal.** When the fact was
  gone, claude-haiku-5-5 chose no option in 400/400 episodes. gpt-6-luna
  guessed in 387/400. Both spent 3.7–4.3× the reasoning tokens. One model
  per provider, so this says nothing about providers, but it is a
  measured case of an agent detecting that it lacks state rather than
  confabulating it ([[concepts/context-proprioception]]). In a forced
  choice scored 0 for no answer, abstaining and guessing score the same.
  In a research loop they do not.
- **The empty-summary failure is a concrete compaction bug.** A thinking
  model given an 8,192-token output budget for its summary call used it
  all on reasoning and returned nothing, four times in a row, and the run
  died. Upstream fixed it by letting compaction inherit the model's output
  limit. Any compaction call to a reasoning model needs an output budget
  that covers reasoning plus summary.
- **Not tested.** Reasoning replay in the probe (H2 stopped because the
  engagement checks found too little reasoning at clue steps). A
  configuration with both notes and replay. Window sizes other than 24k.
  Any real task.

## Trust signals

- **Credibility:** 3.
  - For: pre-registered (PREREG.md v1.0 frozen before the first paid
    run), with every amendment and decision dated and its effect on the
    paper stated (Table 3). Analysis bugs found after reading the results
    are disclosed and both labels reported. The frozen-rule reading is
    reported alongside the amended one. Every ARC run, including replaced
    ones, is listed (Appendix A). Code, pre-registration, decisions log,
    per-run metadata and a $750 cost ledger are released
    (github.com/raimondasl/persistgym, named in the PDF; not opened at
    ingest). The limitations section anticipates most of this critique.
  - Against: one author, not peer reviewed, no citations yet (8 days
    old). The confirmatory result is close to tautological by design. The
    probe ran below its registered power (T capped at 200 against 290).
    The replication was registered after the first model's results were
    known. ARC rests on one counted run per cell, four games picked for a
    large gap, a model with no dated snapshot, and an amendment adopted
    after outcomes that decides which prediction holds. The Microsoft
    affiliation is given, but the work is disclaimed as personal.
  - Not 4 because the substantive ARC claims are point estimates from
    single runs with intervals covering most of the range, and there is
    no peer review. Not 2 because the disclosure discipline is unusually
    complete, the artifacts are released, and the probe's central
    contrast is not in doubt.

## Follow-up

- **Relevance:** 4. It is the first controlled manipulation of
  re-observability in the graph, holding the episode, model and window
  fixed. It turns [[concepts/lossless-context-offload]]'s scope
  condition from an inference drawn from fan2026empirical's null into a
  measured contrast. It prices the cache break that
  [[concepts/context-eviction-policy]] has argued from bai2026how and
  mason2026missing. Its amendment record is a worked case for
  [[concepts/hce-evaluation]]'s preregistration-binding and rerun-arm
  rules. Not 5 because the probe is synthetic, one window size and a
  perfect lookup, and the ARC part is n = 1 per cell.
- **For this box.** `raw/` is the not-recoverable store that matters:
  web captures and paper snapshots that cannot be re-fetched in the same
  form. Long `/iterate` chains that compact should carry pointers into
  `raw/` and experiment `results/`, not paraphrases. The derived-vs-observed
  gap suggests that HCE rules stated once at session start are the
  observed-fact case and the most at risk. Compaction summaries should
  restate them in fixed fields (our inference).
- **Cheap local test.** PersistGym-lite is released and needs no GPU.
  Running W-0 vs F-0 with Opus on this box would price the cache-break
  dollar ratio under Anthropic's cache-write billing, which the haiku
  replication already shows differs from OpenAI's.
- **Candidates** (cited, not yet in the graph):
  - Mehta & Datta 2026, "Plans Fade Fast: Why Context Management Is
    Load-Bearing for LLM Agents" (arXiv 2606.22953). Naive eviction on
    ALFWorld drops pooled success from 56.7% to 22.0%, and re-surfacing
    the plan does not close the gap.
  - M. Wang et al. 2026, "When Can Agents Forget Their Reasoning?
    ICLR for Long-Horizon Agent Context Compression" (arXiv 2609.29875).
    Listed in the 2026-09-28 digest, not ingested. Reasoning matters
    less once its state has moved into code, files or tool outputs.
  - Z. Wang et al. 2026, "Turning Interaction History into Execution
    State: A Runtime Layer for Long-Horizon Coding Agents" (arXiv
    2608.00808).
  - Zeng et al. 2026, "LOCA-bench" (arXiv 2602.07962). Context growth
    with task semantics fixed.
  - Han et al. 2026, "VISTA: A Visual Harness for Reasoning in an
    Interactive World" (arXiv 2610.02200). Adds compaction to the
    ARC-AGI-3 harness one component at a time. This is a different VISTA
    from [[literature/papers/xu2026llm]].
  - Wu et al. 2026, "Tracing the Thoughts of a Coding Agent Playing
    ARC-AGI-3" (arXiv 2610.11450). Beliefs, rules and plans committed to
    notes and scripts.
  - Fox et al. 2026, PRO-LONG (arXiv 2607.20064), and Karten et al. 2026,
    Prime Agent (arXiv 2608.23552). ARC-AGI-3 agents with structured
    interaction logs or a persistent REPL.
  - Bigio & Sanders 2026 (OpenAI blog), retained reasoning plus
    compaction on ARC-AGI-3, with only the combined effect reported.
  - Same digest, item 18 (arXiv 2610.31778, "Benchmarks Are Surprisingly
    Compaction-Friendly"). Compaction drops anti-cheating rules first.
