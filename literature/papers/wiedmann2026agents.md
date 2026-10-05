---
kind: paper
title: "Agents Are Systems, Not Models: Rethinking Agentic Evaluation"
authors: ["Luis Wiedmann", "Leander Girrbach", "Cordelia Schmid", "Zeynep Akata"]
institutions: ["Technical University of Munich", "MCML", "Helmholtz Munich", "Inria / École normale supérieure / CNRS / PSL"]
year: 2026
venue: "arXiv 2610.01618 (cs.AI), preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.01618"
code_url: "https://github.com/lusxvr/rethinking-agent-evaluation"
citations: null
source: "raw/papers/wiedmann2026agents.pdf"
added: "2026-10-05"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["evaluation", "variance", "run-to-run-noise", "icc", "hurdle-model", "noise-band", "configuration-axes", "self-verification", "oracle-tool", "prompt-vs-tool", "ai4s", "specialist-models", "trajectory-dataset", "calibration"]
---

# Agents Are Systems, Not Models: Rethinking Agentic Evaluation

## TL;DR

A full-factorial study of how an agent's *configuration* shapes its outcome.
A ReAct-style coding agent must find and operate a published specialist model
(AstroCLIP, AstroSage-8B, DNABERT-2, RiNALMo) on four small astrophysics and
genomics tasks. Five axes are crossed: Information × Reasoning × Verification
× Budget × Model, 432 cells. Every cell is run **5 times**, for 8,640 main runs
and 18,240 trajectories including ablations. Three findings matter here.
(1) Among runs that make a genuine attempt, **~54% of score variance is
within-cell** (same configuration, re-run). (2) Prompting the agent to verify
barely moves its verification behaviour (reference-based checks 19% → 22%).
Giving it an `oracle_check` tool roughly triples them. (3) Task information
dominates the other axes for the Qwen backbones. For this graph the paper
contributes **per-cell spread figures at n = 3–5** and a **hurdle
decomposition**. Both feed the pending `/iterate` noise-band proposal. Two
qualifications cut against the abstract's framing. The "verification tool"
scores against the **test labels**. And "information beats budget" **flips
for Claude Sonnet 5**.

## Claims

- **Outcomes are noisy.** "about 54% of the score variance among genuine
  attempts is run-to-run noise rather than an effect of the configuration."
  "A single run's score is therefore not a reliable signal on its own."
- **Information first.** "the information given to the agent has the largest
  effect, outranking both the backbone model and the budget". It also "lowers
  cost and runtime and improves calibration … with no trade-off".
- **Axes interact.** "additional budget mainly raises the score when the agent
  has enough information or a large enough model to use it productively, and
  can lower the score otherwise".
- **Behaviour is set by the system more than by the prompt.** "some agent
  behaviors, such as self-verification, barely respond to prompting by the
  user but shift strongly when the system provides a dedicated tool." The
  authors' own limitation: this "is demonstrated on one dimension
  (verification) and needs to be tested across other behavioral dimensions
  before it can be treated as a general principle."
- **Report variance as standard.** The results "reinforce recent calls for
  more rigorous reporting of completion rates, variance, and confidence
  intervals alongside raw scores" (citing Kapoor et al. 2025 and Rabanser
  et al. 2026).

## Methods

- **Agent.** A single loop with `read_file`, `write_file`, `run_bash`,
  `web_fetch` and `finish`, run in a sandbox with one GPU. Specialist models
  are mounted with READMEs and pre-built envs, not exposed as tools. The
  system prompt calls it an "autonomous research agent".
- **Axes.**
  - Information: none / identity / interface / protocol. Protocol is a full
    code recipe; see App. A.17.
  - Reasoning: act-only / think-act / ReAct.
  - Verification, prompt only: none / asked / reported / binding.
  - Budget, wall-clock: 5 / 10 / 20 min.
  - Model: Qwen3.5 35B-A3B / 122B-A10B / 397B-A17B, all FP8 on vLLM.
- **Tasks.** There are four, each scored against the authors' own reproduction
  of the specialist's paper result (R).
  - redshift (R²), promoter (MCC) and rna-folding (F1) are GAP_POSITIVE: the
    specialist beats the bare backbone.
  - mmlu-astronomy is GAP_NEGATIVE. The right move is to *decline* the weaker
    specialist.
- **Primary metric** is G = (S − B)/(R − B), the share of the
  backbone-to-specialist gap closed. Runs that time out are "incomplete" and
  have no score.
- **Hurdle.** A completed run "clears the hurdle" if it beats the trivial
  score T by a margin µ, which is set per task from the metric's sampling
  noise. The idea follows Cragg 1971 and Rabanser et al. 2026: split a binary
  "genuine attempt" indicator from a continuous score given success.
- **Variance decomposition.**
  - A one-way random-effects ANOVA over cells with ≥3 completed runs gives
    ICC = σ²_between / (σ²_between + σ²_within). "54% noise" is 1 − ICC
    computed on **hurdle-clearing runs only**, pooled over the three
    GAP_POSITIVE tasks **with task means subtracted**.
  - The authors flag both sides of that choice. Without the hurdle split,
    "rare but severe failures inflate MS_within far out of proportion". With
    it, "this restriction selects runs by their outcome, it can bias the
    variance estimate, so we report both."
  - Outcome consistency is C_out = mean over cells of (2p̂ − 1)², where p̂ is
    a cell's share of hurdle-clearing runs (Rabanser et al.).
- **Axis effect** is the range of mean G across an axis's levels divided by
  the within-cell SD. Checks: a 1,000-draw within-cell bootstrap of the
  ranking, partial η² from a linear model with two-way interactions, and
  matched-configuration and median checks for the Budget selection effect.
- **Ablations.**
  - Step-3.7-Flash, open-weight, with no act-only level.
  - Claude Sonnet 5, closed, 144 cells per task.
  - An `oracle_check` tool that "score[s] its current submission against the
    task's reference solution", run over the full grid for Qwen-122B and Step.
- **Taxonomy.** DeepSeek-V4-Flash extracts free text along six dimensions per
  trajectory. Sentence-BERT + HDBSCAN cluster the text, and the judge labels
  and merges the clusters into ≤8 categories. Categories are associated with
  axes by Cramér's V. **No human-agreement check on the judge is reported.**

## Results

- **Variance (Table 8, Fig. 2, Fig. 5).**
  - Pooled ICC is **46.1%** on hurdle-clearing runs (54% noise) and **39.4%**
    on all completed runs (≈61% noise).
  - Per task, the hurdle-clearing ICC is: redshift 36.2%, mmlu 6.9%,
    promoter 49.7%, rna 35.2%. **Noise shares therefore run from 50% to
    93%.**
  - Across the hurdle-margin sensitivity sweep (0–2µ) the pooled ICC stays in
    45–47%.
  - Mean per-cell SD as a % of |R − T|: redshift 85.1, mmlu 17.6, promoter
    10.7, rna 12.8.
  - The 95% CI half-width of a cell mean at 5 runs: 91.9 / 20.8 / 12.9 /
    13.0% of |R − T|.
  - Pooled CI half-width in G units, read off Fig. 5 (right). All completed
    runs: ≈0.28 / 0.19 / 0.15 at 3 / 4 / 5 runs. Hurdle-clearing runs only:
    ≈0.15 / 0.10 / 0.08.
  - **Better cells are less noisy.** Cell mean G against cell SD has
    Spearman ρ = −0.79.
- **Completion.** 80.7% of runs complete, and 19.3% run out of budget. A large
  share of cells complete on some repeats but not others: 72% "mixed" on
  rna-folding.
- **Hurdle failure.** redshift 53.8%, mmlu 4.7%, promoter 0.5%, rna 8.7%.
  Pooled C_out = 0.86; 0.73 on redshift.
- **Axis effects (Table 3, Qwen grid).** Information ranks 1st on every task
  (effect 1.45–1.93; partial η² 0.26–0.41). Model's effect is 0.17–1.25.
  Reasoning, Budget and Verification are each ≲0.30 (Verification 0.08–0.13,
  always last). Information is first in ≥99% of bootstrap draws.
- **Protocol information can solve the task outright.** On redshift, every
  protocol-level run scores exactly the reference, R² 0.753, under every
  budget (Table 9).
- **Interactions (Table 19).** Going from short to long budget changes G by
  **−0.067** for Qwen-35B and **+0.025** for Qwen-397B. Part of the
  small-model drop is a selection effect: it vanishes under strict matching.
  The largest interaction is Information×Model (rna partial η² 0.13).
- **Cross-model (Tables 14–17).**
  - Step-3.7-Flash also ranks Information first.
  - **Claude Sonnet 5 ranks Budget first on all three tasks** (effect
    1.59–1.88), with Information second. It uses less than half the tool
    calls, makes about a fifth of the errors and is better calibrated (−0.016
    against 0.092). It still has a lower pooled G than Qwen-397B (0.828
    against 0.870).
- **Oracle (Table 12).**
  - Pooled G with the oracle: Qwen-122B **0.822 → 0.923**, Step
    **0.865 → 0.889**.
  - Calibration error falls from 0.093 to 0.002 (Qwen) and from 0.073 to
    0.026 (Step).
  - Cost per run rises by 34–112%.
  - Flagged hill-climbing (≥5 calls with ≥half following only file edits) is
    **4.4%** of runs, 10–15% on redshift, with up to 111 calls in one run.
    Excluding flagged runs leaves G nearly unchanged (0.922 / 0.878).
- **Verification behaviour (Fig. 4).**
  - Baseline, from none to binding: reference-based verification 19 → 17 →
    23 → 22%, format-only 76 → 77 → 71 → 72%.
  - With the oracle: reference-based 55–63% at every prompt level.
  - The authors concede "Part of this shift reflects the oracle calls
    themselves."
- **Agents almost never decline a weaker specialist.** On mmlu-astronomy,
  **99.8%** of runs use AstroSage. Accuracy is 0.685, against 0.967 for the
  bare backbone. The protocol level lowers accuracy further (0.71 → 0.62).
  The oracle barely helps (0.707 → 0.725).
- **Cost.** $0.11–0.42 per run. One 720-run grid costs $63–367 per task and
  model.

## Critique / open questions

- **"54%" is a property of this grid, not of agents.** ICC is a share, and its
  denominator includes between-cell variance. The grid deliberately spans
  extremes: no information to a full code recipe, and a 35B to a 397B model.
  A narrower grid would show a *larger* noise share. An `/iterate` cycle
  comparing two adjacent variants of one idea sits at the narrow end. The
  transferable quantities are the **absolute spreads**: per-cell SD of
  ~11–13% of |R − T| on the well-behaved tasks, and ~85% on redshift. Also
  transferable: a CI half-width that is still ~0.08 G among genuine attempts
  after 5 repeats.
- **The pooled figure hides a 50–93% per-task range.** mmlu-astronomy's 93%
  arises because the configuration barely matters when every run uses the
  wrong model. Noise share depends on task as much as on agent.
- **The noise measured is agent-trajectory noise on 5–20-minute runs.** Each
  repeat re-runs the whole agent, so it writes different code, picks
  different preprocessing and fetches different docs. This is not the noise
  of training a fixed program under different seeds. For an ML-research loop
  this is the more relevant and less-measured component. "Same hypothesis,
  re-implemented" is what this paper's within-cell variance captures.
  `/implement --seeds` holds the code fixed and captures none of it.
- **"Information" is close to tautological at its top level.** The protocol
  level is the solution recipe, with code (App. A.17). That giving the
  solution beats a bigger model is unsurprising. The more interesting
  contrast (identity → interface) is not separated out in the headline. The
  ranking also comes from the open-weight Qwen family. For the strongest
  model tested, Sonnet 5, **Budget ranks first**. The abstract's "exceeding
  both time budget and model size" does not mention this.
- **The Budget axis is very short.** 5/10/20 minutes, with tool execution
  taking 54–73% of wall clock. "Budget matters little" is a claim about
  minutes, not about the hours-to-days budgets of ML-research loops.
- **The "system-level verification" arm is test-label access.** The oracle
  scores against the task's reference solution. The authors call it "the
  strongest form of system-level verification" and say that "in practice, a
  system would offer weaker checks, such as a validation set". **That weaker
  check is never run.** So the paper shows that agents call a scoring tool
  when one exists and score higher with it, which partly follows from
  selection on the test set. It does not show that a deployable verifier
  changes behaviour. The hill-climbing flag is a heuristic lower bound, and
  any single oracle call is already a peek at the test set. Under
  [[concepts/hce-evaluation]] the oracle arm's G is not a clean score.
- **The prompt-side result is the solid half.** Four escalating verification
  prompts, up to "do not submit unless convinced", move reference-based
  checks by about 3 pp. Across 8,640 runs this is a well-powered null. It
  replicates [[literature/papers/agarwal2026fire]]'s finding that generic
  "verify" text does nothing, at much larger n.
- **The taxonomy is an unvalidated LLM judge.** No human labels and no
  inter-rater agreement are reported, and categories are induced by
  clustering. Treat the Fig. 4 shares as approximate. The direction of the
  oracle shift is still hard to doubt, since the shift is partly the oracle
  calls themselves.
- **Restricting to the hurdle conditions on the outcome.** The authors report
  both versions, which is good practice. The 54% is the more favourable of
  the two noise figures, and with all completed runs it is ≈61%.
- **Tasks are small.** They have 20–613 items and are re-sized so a 5-minute
  run can finish. The reference R is the authors' reproduction, not the
  published number.
- **Scale and limits are honest.** The authors state the four-task,
  two-domain limit and the single-dimension limit of the prompt-vs-system
  claim. No multi-agent or memory axes are varied.

## Trust signals

- **Credibility:** 4. The authors are TUM/MCML, Helmholtz Munich and Inria,
  with Akata and Schmid as senior authors. The design is full-factorial with
  5 repeats per cell, 18,240 trajectories and a named release repo (though
  the reproducibility statement says "will release"). The statistics are
  careful: random-effects ICC, a hurdle split reported both ways, Wilson
  intervals, bootstrap rank stability, partial η², and matched-configuration
  checks for selection effects. The limitations section is explicit. It is
  not a 5 for four reasons: an unreviewed preprint, four small tasks, an
  unvalidated LLM-judge taxonomy, and a verification-tool arm that uses the
  test labels. The variance and axis numbers deserve about 4. The "system
  beats prompt" generalization deserves about 3.

## Follow-up

- **Relevance:** 4. This is material new evidence for
  [[concepts/pass-at-k]] and for the noise-floor rule in
  [[concepts/budget-as-ceiling]]. It is the largest repeated-run agent
  dataset in this graph with per-cell spreads at n = 3–5 reported. It adds
  a well-powered null for prompted verification and a positive for a
  verification tool to [[concepts/enforcement-boundary-placement]]. It is
  not a 5: the tasks are short specialist-operation jobs rather than
  ML-research loops, it seeds no concept, and its headline variance share
  does not transfer.
- **Bearing on the pending 2026-09-27 `iterate-no-improvement-noise-band`
  proposal.**
  - *Premise: corroborated.* A single run is "not a reliable signal". Even
    5 repeats leave a ~0.08 G half-width among genuine attempts, and ~0.15
    with failures included.
  - *Repeat count.* The following is our arithmetic, not the paper's. The
    implied per-run SD among genuine attempts is ≈0.06 G (half-width ×
    √5 / t₄). The minimum detectable difference between two configurations
    (α = 0.05, 80% power) is ≈2.3 SD at n = 3 per arm and ≈1.8 SD at n = 5.
    Resolving a 1-SD difference needs ~16 runs per arm. So a band from
    `--seeds 3` can veto sub-noise "new bests", which is the proposal's
    claim. It cannot certify small real gains, and the proposal should not
    be read as claiming that.
  - *Band source: a gap the proposal does not cover.* The proposal's band is
    the `std` from `--seeds`, which is training-seed noise on fixed code.
    This paper's within-cell variance is re-running-the-agent noise. In
    `/iterate` that is the noise of implementing the same hypothesis again,
    and nothing in the harness measures it. A band built only from seeds
    **understates** the noise in a hypothesis-vs-hypothesis keep decision.
    A cheap probe: re-run `/implement` on the same plan once per chain and
    log the delta.
  - *Hurdle split.* A crashed or degenerate candidate should count as a
    hurdle failure and be excluded from the band's `std`. Otherwise one
    catastrophic seed inflates the band until nothing can ever beat it. This
    is the paper's MS_within inflation argument.
  - *The band should be per-comparison, not a global constant.* Higher-
    scoring cells have lower spread (ρ = −0.79). A band calibrated early in a
    chain will be too wide near the frontier. The proposal's "larger of the
    two runs' std" already handles this.
- **Prompt vs tool for verification.** This sits with
  [[literature/papers/agarwal2026fire]] (generic verify text has no effect;
  specific, timed content does) and with this graph's enforcement-placement
  line. Asking does not change verification behaviour, and giving the agent
  a scoring instrument does. The deployable version, a validation-split
  scorer the agent can call, is the untested arm. Under
  [[concepts/programmable-evaluator-oracle]] that is the version this repo
  would build.
- **The mmlu-astronomy result is a cheap behavioural probe.** In 99.8% of
  runs, agents use a weaker specialist because it is offered. This is a
  research-agent version of "authority bias" toward provided tools. It is
  relevant wherever a harness pre-mounts tools or baselines.
- **Candidates:**
  - Rabanser et al. 2026, "Towards a science of AI agent reliability" (ICML
    2026; source of C_out and the hurdle framing).
  - Khanal et al. 2026, "Beyond pass@1: A reliability science framework for
    long-horizon LLM agents".
  - Kapoor et al. 2026, Holistic Agent Leaderboard (ICLR 2026).
  - Kirgis et al. 2026, "Can AI agents conduct open-ended AI research?"
  - None of these is ingested yet.
