---
kind: paper
title: "Judged Useless, Queried Anyway: Tool-Using Agents Rarely Turn Their Own Evidence Judgments into Stopping Decisions"
authors: ["Chubin Zhang", "Zhenglin Wan", "Xingrui Yu", "Jingxuan Wu", "Yaxin Zhou", "Ivor Tsang", "Bo An"]
institutions: ["Nanyang Technological University", "National University of Singapore", "A*STAR (CFAR, IHPC)", "University of North Carolina at Chapel Hill", "Carnegie Mellon University"]
year: 2026
venue: "arXiv 2610.06191v1 (cs.AI), 2026-10-05"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.06191"
code_url: "https://github.com/bennidict23/judged-useless-queried-anyway"
citations: null
source: "raw/papers/zhang2026judged.pdf"
added: "2026-10-12"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["stopping", "knowing-doing-gap", "judgment-action-dissociation", "harness-enforced-rule", "run-length-counter", "action-masking", "prompt-vs-harness", "deadline-driven-stopping", "tool-failure", "redirect-vs-halt", "pre-registration", "time-matched-contrast", "hotpotqa", "fever", "small-open-models"]
---

# Judged Useless, Queried Anyway: Tool-Using Agents Rarely Turn Their Own Evidence Judgments into Stopping Decisions

## TL;DR

ReAct agents answer HotpotQA (and FEVER) questions over tiny 10-paragraph
knowledge bases. The harness swaps search results for another question's
paragraph on a controlled schedule: never, always, the first 1–3 calls then
recover, or from the third call on. A side-channel question on a copy of the
history ("was the LAST observation USEFUL or USELESS?") shows the agents
recognise failed results: 97–100% are called useless by the five models whose
judgments were recorded. Unaided, they almost never stop on that judgment.
After five consecutive self-judged useless results the open models answer on
0.3–7% of questions and otherwise search until the 8-action budget runs out.

Prompt cues (permission to answer from memory, a stated budget with a
counter, the stopping rule in words, a per-call price, the running count of
useless judgments) change *when* agents stop but mostly not *what* they stop
on. Stating the budget moves stops to the deadline, and the deadline moves
when the budget doubles. What makes stopping track the evidence is a harness
rule: after k = 5 consecutive USELESS judgments the harness removes `search`
and `lookup` so only `finish` is available. The models are Qwen2.5-7B,
Llama-3.1-8B, Qwen3-8B and Qwen3-32B for most arms, plus Claude Haiku 4.5,
Claude Sonnet 5 (100 questions, two arms) and Search-R1. The success gains are
modest: +2.6 to +9.7 points of six-regime mean success on HotpotQA, and a
stated budget matches the rule's success for three of the four open models.

For this graph, the rule is a **stop**, not a redirect. The closest redirect
arm (App. D.5) disables a failing primary tool and points the agent at a
backup. It adds little, because the larger models already switch early on
their own.

## Claims

- **Judgments are accurate.** "the seven agents we test call a failing
  source's results useless 97–100% of the time" (abstract, hedged "where we
  record their judgments"). Gold paragraphs are called useful 77–98% on first
  retrieval (Qwen2.5-7B 57%), so this is not a blanket "useless" bias.
- **Actions do not follow.** After five self-judged useless results in a row,
  unaided Qwen3-8B answers on 1 of 292 questions; the other Figure 1 models
  on at most 7%. Answering at that point would have paid: 15–17% success
  against at most 7% for continuing (exploratory, App. D.2).
- **Beliefs do not explain it.** Self-reported value of continuing vs
  answering is wrong in different directions per model, and where a model's
  own report favours answering, its action still does not follow.
- **Telling changes timing, not the trigger.** "Permission to answer from
  memory and a reasoning mode can bring early stops regardless of evidence, a
  stated budget moves the 7–8B models' stops to the deadline, and a stopping
  rule or call cost in the prompt is followed at most partly."
- **Enforced integration works.** "Stopping follows the evidence only when
  the harness enforces an integration step that makes the agent answer after
  five consecutive results it judged useless. This step raises
  failing-source success for every model, keeps the stopping point fixed when
  the budget doubles, and needs no extra judgment call when the agent states
  its judgments."
- **Not specific to small models.** "A larger open model, a reasoning mode
  and an RL-trained search agent still largely fail to stop on the evidence,
  and only Claude Sonnet 5 partly does so without being prompted."
- **Evaluation design claim.** Success alone cannot reveal what stopping
  responds to, because a deadline-driven agent and an evidence-driven one can
  score alike. Test instead whether stopping at a fixed step rises with the
  run of self-judged useless results.

## Methods

- **Environment.** HotpotQA distractor dev questions, each with its own
  10-paragraph knowledge base (2 gold, 8 distractors). Tools: `search` (title
  match, then word overlap), `lookup`, `finish`. 8-action budget. Correct
  means token F1 ≥ 0.6. Transfer task: 292 balanced FEVER claims.
- **Failure regimes.** A failed observation is replaced by another question's
  paragraph. Six regimes are averaged with equal weight ("mean6"): clean,
  persistent, recover_after_1/2/3, late_onset_from_3. Harder variants: own
  distractors ("plausible") and pages with supporting facts removed
  ("answerless"), plus recover_after_4/5.
- **Three separately measured quantities per step.**
  - *Judgment*: one-word USEFUL/USELESS on a scratch copy the agent never
    sees again. Judgments stated in the agent's own reasoning were labelled
    by LLM annotators (Claude Sonnet 5, GPT-5.6-luna; reference set by Claude
    Fable 5.1 and GPT-6-sol), first-author adjudication, binary κ ≈ 0.90.
  - *Belief*: self-reported p_continue and p_answer_now on another copy.
  - *Action*.
- **Primary measure Δ.** At the same step (decisions after 3–6 results), the
  probability of answering when every result so far was judged useless minus
  the same when an earlier result was judged useful. Mantel–Haenszel pooled
  over steps; final action excluded. Positive = stops on the useless run,
  zero = clock or deadline, negative = stops after useful evidence. Validated
  on simulated policies (App. Table 7). A hazard model separates the two
  explanations of negative Δ (App. C.2).
- **Arms (Table 1).** none; permit; budget (+ counter, "unanswered = wrong");
  stated rule; call cost (0.05 per call vs 1 per correct answer); decide
  (running count shown, agent asked CONTINUE/ANSWER_NOW); enforced rule;
  combo (budget + enforced rule). Controls: the rule fed a random signal at
  the model's own useless rate, and a lexical detector (useless if < 25% of
  question content words appear).
- **The enforced rule's mechanism.** At k = 5 the system prompt says search
  and lookup "are unavailable; the only available action is finish[answer]",
  and the observation is followed by "Do not make any further tool calls."
  It is action masking plus a forced answer from parametric memory. k = 5 was
  chosen on 100 development questions.
- **Stated-judgment reader (App. D.3).** A keyword classifier on the first
  two sentences of the agent's thought replaces the side-channel call
  (precision 0.971, recall 0.913 on Qwen3-8B dev labels).
- **Two-source redirect (App. D.5).** Adds `backup_search` over an intact
  copy, described as slower and costlier. The "switch rule" disables the
  primary after five useless judgments and says to use the backup or finish.
- **Protocol.** test300 for confirmatory runs; fresh300 (new seed, disjoint)
  re-tests Δ after it was defined in amendment 12. Two further runs with new
  seeds and failure draws. Paired bootstrap (2,000), Holm across models
  within a hypothesis. An append-only pre-registration with 20 amendments,
  each frozen before the runs it governs, **not deposited with a third-party
  registry**. About 350 GPU-hours on 4× RTX A5000. Claude models via API, in
  fewer arms: Haiku 4.5 none/rule on 300 questions and stated rule/cost on
  200; Sonnet 5 none/rule on the first 100 questions only.

## Results

- **Mean6 success, HotpotQA test300 (raw, Table 2/13).**

  | Model | none | budget | stated rule | enforced rule | combo |
  |---|---|---|---|---|---|
  | Qwen2.5-7B | .192 | .328 | .206 | .237 | .375 |
  | Llama-3.1-8B | .263 | .312 | .267 | .328 | .337 |
  | Qwen3-8B | .448 | .464 | .439 | .474 | .478 |
  | Qwen3-32B | .509 | .581 | .526 | .572 | .578 |
  | Claude Haiku 4.5 | .501 | – | – | .598 | – |
  | Claude Sonnet 5 (n = 100) | .632 | – | – | .653 | – |

- **Rule vs none:** +0.045, +0.064, +0.026, +0.063 for the open models,
  +0.097 for Haiku (Holm-significant). Sonnet 5: +0.022, p = 0.051, so the
  pre-registered S-H1 **failed**. Its persistent-regime gain (+0.11) passed.
  FEVER: +0.129 to +0.201 for the three 7–8B models.
- **Rule vs budget statement:** −0.091 (Qwen2.5-7B, worse), +0.016, +0.009,
  −0.009. Not significantly positive for any model. The rule's advantage over
  the budget cue is the stopping point, not success: with the budget doubled
  to 16, budget-cued 7–8B models move their median answer from action 8 to 16
  and make 61–71% more tool calls for no gain; the rule's median stays at 6
  (tool calls 2.88 → 2.91 for Qwen2.5-7B).
- **Δ (time-matched contrast).** Every unaided open model is negative
  (−0.02 to −0.11), as are permit and budget. Stated rule is positive only for
  Llama-3.1-8B (+0.09) and Qwen3-32B (+0.08). Call cost is positive only for
  Qwen3-32B (+0.09). Enforced rule +0.33 to +0.37, **positive by
  construction**. fresh300 replication: all 19 pre-registered tests pass
  (four open models only). Hazard model: each extra useless judgment changes
  unaided answering by −0.02 to −0.01, against +0.15 to +0.17 under the rule.
- **Does the signal matter?** Rule vs random signal at the same rate: +0.011
  to +0.029 (Holm-significant). Rule vs the lexical detector: +0.012, −0.001,
  −0.011, so a cheap word-overlap heuristic does as well as the agent's own
  judgment.
- **Cost.** At λ = 0.05 per call, charging the rule's side-channel judgment
  calls makes it the most expensive arm (utility −0.053 and −0.192 for
  Qwen2.5-7B and Llama-3.1-8B, vs +0.083 and +0.043 for budget). The
  stated-judgment reader removes those calls. Budget plus the reader beats
  the budget alone by +0.022 to +0.045 utility for three models and ties for
  Qwen3-32B (pre-registered E4 fails there).
- **Threshold vs recovery lag (App. D.5, Table 26).** When the source
  recovers after five failures, rule minus none is −0.213 (Qwen3-8B) and
  −0.137 (Qwen3-32B). k = 5 was tuned for recovery within three failures.
  The rule gives up just before the source comes back.
- **Frontier and near-frontier models.**
  - *Claude Haiku 4.5:* unaided it answered on 1 of 284 qualifying questions;
    forced at that point it was right on 114. Stated rule: partial
    (design-label contrast +0.05), mean6 −0.050 vs the enforced rule. **Call
    cost stated: mean6 +0.053 and utility +0.213 above the enforced rule**,
    though its stops are not evidence-driven (+0.01). That comparison is
    confounded in the new arms' favour: format-failure fixes were added only
    to those runs (2–3% of trajectories).
  - *Claude Sonnet 5:* the only model that partly integrates unaided
    (design-label contrast +0.12 [+0.05, +0.18]); it still exhausted its
    budget on 45% of questions where the rule fired.
- **Larger, reasoning and RL-trained agents.** Qwen3-32B with a stated budget
  matches the rule's success but stops by time (Δ = 0.00). Thinking mode
  makes Qwen3-32B answer from memory on 97% of failing-source questions and
  before any real result on 47% of recover_after_2 questions. Search-R1's
  judgments call 89% of real results useless (pre-registered validity
  failed), so its Δ is not interpreted. The rule still adds +0.024.
- **Redirect arm (two sources, Table 25).** Unaided, Qwen3-8B and Qwen3-32B
  switch to the backup by action 4 on 73% and 95% of persistently failing
  questions; Qwen2.5-7B 49%, Llama-3.1-8B 17%. Unaided leaving is not
  evidence-driven (Δ ≤ +0.02). Switch rule minus none (mean6): +0.011 (n.s.),
  +0.054, +0.016, +0.007 (n.s.). Llama's success on the failing source falls
  to 0.05 without the rule, against 0.50 with a working primary.
- **Harder failures.** On-topic failures lower judgment accuracy (84–97%
  plausible, 76–97% answerless). After five useless judgments the unaided
  agents answer on 2–23% of questions, and the rule still helps. On
  answerless sources the budget condition does better than the rule.

## Critique / open questions

- **The paper reads below the digest framing.** The digest item overstates
  in three places:
  - *"Seven agents judge … useless 97–100%."* Judgments were recorded for
    five. Sonnet 5's per-step judgments were not saved (App. B.4). Search-R1
    also calls 89% of *real* results useless, which failed the paper's own
    validity test. The abstract is hedged ("where we record their
    judgments"); the digest dropped the hedge.
  - *"Only a harness-enforced step makes stopping follow evidence."* True for
    the 7–8B models. Not cleanly true above that: the stated rule is partly
    followed by Llama-3.1-8B, Qwen3-32B and Haiku; Sonnet 5 partly integrates
    with no prompt at all; and Haiku with the price stated beats the enforced
    rule on both success and utility.
  - *"A pre-registered replication on 300 questions confirms it."* It covers
    the four open models only. No Claude model was re-run on fresh300.
- **The running overclaim finding holds, mildly.** The abstract is more
  careful than most in this graph. Its one favourable cut: "raises
  failing-source success for every model" uses the persistent regime. On the
  pre-registered mean6 test the strongest model, Sonnet 5, failed (p = 0.051,
  n = 100), and the vivid "114 of 284 vs 1" Haiku figure is a
  persistent-regime count, not a success rate across regimes.
- **The headline Δ for the rule is circular.** A rule that forces an answer
  after five useless judgments has a positive Δ on those judgments by
  construction; the authors say so. The informative evidence is the success
  and budget-invariance results, and those are small. A one-line budget
  statement reaches the same success for three of four open models.
- **The agent's own judgment is not what makes the rule work.** A lexical
  detector matches it, and a random signal at the same rate is only 1–3
  points behind. The architectural lesson is "a harness-owned run-length
  counter mapped to a masked action set", not "trust the agent's judgment".
  For a loop where the harness already computes usefulness (a metric
  comparison), the judgment half is unnecessary.
- **The model mix is mostly small.** Four of the seven agents are 7–8B. The
  two models nearest the frontier ran in few arms on 100–300 questions, and
  they are where the "prompts don't work" premise is weakest. No Opus-class,
  GPT or Gemini acting agent was tested (GPT models appear only as
  annotators).
- **The environment is a toy.** 10-paragraph knowledge bases, an 8-action
  budget, failures made by substituting unrelated paragraphs, and an
  8B-friendly ReAct format. Partial, stale or wrong tool outputs are untested
  (the authors say so). Placeholder answers (`finish[answer]`) hit hundreds of
  trajectories in some arms and were repaired post hoc. The paper shows the
  conclusions do not change, but it is a sign of how close to the format
  limits the small models run.
- **The threshold cuts both ways.** The −0.21 loss when recovery comes at
  failure 5 is the cost of any fixed k set below the recovery lag. The paper
  tunes k on the same recovery distribution it tests, so the clean-regime
  non-inferiority results flatter the rule.
- **Credit where due.** Judgment, belief and action are measured separately;
  Δ is validated on simulated policies before use; the paper reports every
  pre-registered failure (H3 for Qwen3-8B, H7 for Qwen2.5-7B, S-H1 for
  Sonnet 5, L-H9, Rp2, PP2, E4); there are random and lexical signal
  controls; and code, prompts and manifests are released.

## Trust signals

- **Credibility:** 4. In its favour:
  - established groups (Ivor Tsang at A*STAR CFAR/IHPC, Bo An at NTU) plus
    NUS, UNC-Chapel Hill and CMU;
  - code, prompts and question manifests released under MIT (URL named in
    the PDF; not checked here);
  - an append-only pre-registration with frozen amendments, a disjoint
    fresh300 replication, two extra seed/failure-draw runs, Holm correction,
    and reported failures;
  - controls that isolate the signal (random, lexical) and the stopping
    point (doubled budget).

  Against it:
  - an unreviewed v1 preprint, and a pre-registration that was self-hosted,
    with Δ itself defined after early results (re-tested on fresh data);
  - a synthetic environment with tiny knowledge bases and an 8-action budget;
  - near-frontier models run in few arms on 100–300 questions, and not in
    the replication;
  - an LLM-annotated stated-judgment set adjudicated only by the first
    author.

## Follow-up

- **Relevance:** 4. It is the cleanest evidence in this graph that a
  per-step "this didn't help" signal fed to a **consecutive-run counter that
  resets on a useful result** (the shape of `max_consecutive_no_improvement`)
  must have its action enforced by the harness, not stated in the prompt. It
  also adds a new hazard: a stated budget turns stopping into
  deadline-chasing. It strengthens [[concepts/budget-as-ceiling]] and
  [[concepts/enforcement-boundary-placement]], and limits
  [[concepts/context-proprioception]]. It is not a 5: the task is QA
  retrieval, the action tested is a stop, not a redirect, and the effects
  shrink at the frontier end of the model range.
- **Bearing on redirect-before-halt in `/iterate`.**
  - *Supports enforcement.* Showing the agent its running count and asking
    it to decide (the `decide` arm) did not make 7–8B models stop on it
    (Δ −0.14 to −0.24). This is the third source, after yan2026traceml and
    xu2026conflictguide, where the prompt-only version of a controller rule
    fails.
  - *Gives the mechanism.* The enforced rule is **action masking**: the
    harness removes the near-neighbour actions and leaves one legal move.
    The `/iterate` analogue is to reject any step-3 candidate in the same
    category as the stalled ones once the counter is non-zero, not to ask for
    a different one.
  - *But it is a stop.* The redirect arm (switch to a named backup tool)
    added +0.7 to +5.4 points, significant for two of four models. The
    model that already switched on 95% of failing questions gained nothing.
    The paper still does not run redirect against halt.
  - *Watch the threshold.* A counter set shorter than a line's real payoff
    lag kills recoverable lines (−0.21). The no-improvement threshold should
    come from how long real improvements take to show, which ties it to the
    09-27 noise-band proposal.
  - *Do not show the agent the ceiling as a deadline.* A stated budget moved
    stops to the last action and scaled spend with the budget. If `/iterate`'s
    prompt names `max_consecutive_no_improvement`, the agent may treat it as
    a schedule. Keep the counter in the controller.
  - *Frontier caveat.* Sonnet 5 partly integrates unaided, and Haiku with a
    stated price beat the enforced rule. For Opus-class proposers the case
    that "the prompt will not work" is plausible but not shown here.
- **No new concept seeded.** "Harness-side integration of per-step judgments
  into a masked action" is the paper's atomic idea, but budget-as-ceiling's
  metric-computed counter plus enforcement-boundary-placement's prompt-vs-
  harness rows already cover it. Revisit if a second source tests the same
  dissociation in an ML loop.
- **Candidates (cited, not in graph):**
  - Zhang et al. 2026b, "From Inertia to Objectivity: Improving Deep
    Research Agents with Noise Isolation" (arXiv 2608.23045): deep-research
    agents keep browsing results already judged irrelevant.
  - Soudani et al. 2026, "When Deep Research Agents Stagnate"
    (arXiv 2608.15191): a retrieval-aware controller for stagnation.
  - Liu 2026, "When May an Agent Stop? Evidence-Carrying Termination for
    Tool-Using LLMs" (arXiv 2608.23623).
  - Roh and Han 2026, HALT: verification-aware stopping (arXiv 2608.02009).
  - Schmied et al. 2026, "LLMs are Greedy Agents" (ICLR 2026): the
    knowing-doing gap in bandits.
