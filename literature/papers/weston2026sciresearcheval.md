---
kind: paper
title: "SciResearch-Eval: Evaluating Scientific Judgment in Autonomous Experimental Agents"
authors: ["Brian T. Weston", "Alondra Torres Contreras", "Aldair E. Gongora"]
institutions: ["Lawrence Livermore National Laboratory"]
year: 2026
venue: "arXiv 2611.21541 (cs.AI), v1 2026-11-16, preprint (16-page main text + 7-page appendix; report no. LLNL-JRNL-2026874)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.21541"
code_url: "https://github.com/Btwestyo/sciresearch-eval"    # named in the paper (CC BY 4.0), not opened at ingest; held-out split and its traces withheld
citations: null
source: "raw/papers/weston2026sciresearcheval.pdf"
added: "2026-11-24"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["scientific-judgment", "flawed-simulator", "model-discrepancy", "interval-score", "calibration", "experiment-design", "hint-ablation", "reasoning-effort", "pre-registration", "fresh-split-replication", "judge-free-scoring", "input-token-budget", "false-completion-claim", "benchmark", "claude-5.5", "fable-5.2", "gpt-6-astra"]
---

# SciResearch-Eval: Evaluating Scientific Judgment in Autonomous Experimental Agents

## TL;DR

An agent must measure a plate's conductivity k, heat capacity ρc and
rear heat-loss coefficient h. It gets one heating experiment, can design
four more, and can fit a supplied 1-D heat-conduction simulator. It
reports each property with a 95% interval. The hidden "real" sample is
simulated with richer physics, so the truth is exact. In some tasks the
data are weak or the default solver is too coarse. In others the
simulator lacks real physics: a glued joint, radiation, conductivity that
varies with temperature, or side heat loss. About 30% of tasks are
controls with nothing wrong. Scoring uses the proper interval score
(Gneiting–Raftery) and coverage against the truth, with no LLM judge.
The held-out split has 93 tasks, 9 models and 3 runs per task, 2,511
episodes in the main condition.

Five results matter for this graph:

1. **Procedure is solved; missing physics is not.** Current Claude
   models and GPT-6 Astra are at or near ceiling on weak data, coarse
   solvers and picking the right simulator from a menu. On the 38
   physics-wrong tasks, coverage of the nominal 95% intervals is **62%**
   (Opus 5.5), 56% (Sonnet 5.5), 49% (Haiku 5.5) and 20–39% (Sonnet 4.5
   and open-weight models). GPT-6 Astra reaches **94%** and Claude Fable
   5.2 **99%**. Controls are fine for everyone, so this is not blanket
   scepticism.
2. **The winners measure where the simulator is valid.** Astra's
   median estimate error is ~0.2% against ~17% for Opus. Its intervals
   are no wider. It runs gentle and short experiments and bases its
   answer on one dataset the simulator fits to 1.4× noise. Opus pools
   data the simulator cannot explain (final misfit 21× noise). Naming
   the missing mechanism is neither necessary nor sufficient. When Opus
   names it, coverage is 77%. When Astra does not, coverage is 82%.
3. **Telling the model to widen its intervals does nothing. Stating the
   method helps the score, not coverage.** These are one-paragraph
   hints appended to the prompt for the three Claude 5.5 models:
   - "Widen your ranges by the model's possible bias" (H): no
     detectable change for any of the three.
   - General scientific practice (B): 1.4× better for Opus only.
   - Astra's strategy stated outright (S): interval score **1.6–2.5×
     better**, replicated on a pre-registered fresh split (1.7–2.2×).
     Coverage moves **+0 to +7 points**. S fixes the k(T) tasks, where
     "heat gently" is a single action. It does not fix the glued joint
     or side loss.
4. **Reasoning effort did far more than any hint.** In a pre-registered
   effort ablation, Opus 5.5 at max is **23× better** [16–33×] than at
   its default (medium). At xhigh it is 28× better. Against Astra's
   default it shows no detectable difference (not an equivalence test).
   High effort alone gives 2.9× at 1.5× the tokens. Most of the gain
   comes between high and xhigh, where output grows from 5.1k to 47k
   tokens per episode. Sonnet and Haiku 5.5 at max improve 7.7× and 11×.
   Sonnet 4.5, Gemma 4 and Muse Glimmer gain nothing at their highest
   setting. Qwen3.8 writes 70k tokens by default and still covers 38%.
5. **The harness was part of the measurement.**
   - The 600k cumulative input-token cap is the same for every model.
     The conversation is re-sent each turn, so it binds on long-reasoning
     settings: 33 of 198 Opus-max episodes ended unsubmitted, and **16
     of those 33 said they had submitted** without calling `submit`.
   - A rerun of those 33 at twice the budget, specified before it ran,
     removed the apparent max-below-xhigh dip.
   - Fable 5.2 at max hit the cap in 60 of 66 episodes and was not
     rerun.

## Claims

- Scientific judgment, meaning whether a good fit is good for the right
  reason, can be measured with exact truth and no LLM judge. That needs
  one deep domain with computed flaws, not a broad scraped collection.
- At default settings the failure lies in diagnosing and, above all,
  correcting for a wrong simulator, not in procedure.
- Calibration comes from better estimates, not wider intervals.
  "Asking for humility does not produce calibration."
- Stating the method helps only where it maps onto one action.
- The correction is latent in the Claude 5.5 models. Effort unlocks it,
  by a different route from Astra's: Opus at max selects which data to
  trust instead of designing valid data. More reasoning is not the skill
  in general.
- Defaults understate capability, so results should be reported across
  effort settings. Budget limits and harness rules can look like model
  behaviour, so they should be reported and their effect tested.
- For training (asserted): make the high-effort behaviour the default,
  "as Fable 5.2 appears to have done".

## Methods

- **Environment.** A 1-D finite-volume solver (θ stepping, Picard
  iteration). The hidden sample runs 4× finer in space and time, with a
  2-D axisymmetric solver for side loss. Noise σ ∈ {0.02, 0.05, 0.1} K.
  Tools:
  - `request_measurement` (≤4);
  - `run_simulation` and `fit_parameters` (compute budget; the fit
    returns misfit per sensor);
  - `analyze` (runs test, early/late refit, amplitude scaling);
  - free data and residual fetches;
  - `submit` once.

  The hidden prediction target lies outside the allowed experiment
  range (3× power and duration).
- **Task families.** Severity labels are computed:
  - T1: unidentifiable data.
  - T2 and T10: missing physics. T10 asks for "actual physical
    properties … including any inadequacy of the supplied model".
  - T3: under-resolved solver.
  - T5: under-resolved solver plus missing physics.
  - T6: a candidate menu with one right simulator, or none.

  Physics severity is the best-fit misfit to noiseless truth in noise
  units, stratified at 2, 5 and 15.
- **Validation before any model ran.**
  - The simulator is checked against exact solutions (0.01–1%).
  - Each numerical trap is shown to produce a real error.
  - A scripted-policy gate checks that oracle < heuristic < naive on
    every family, and that always-reject fails the controls.
  - Public task files are scanned for regime cues.
  - The held-out split (93 tasks) was generated from fresh seeds and
    frozen with checksums.
- **Scoring.** Interval score on log10 parameters (width + 40× miss
  distance), geometric mean over physics-wrong tasks. Coverage counts
  unsubmitted runs as misses. Paired bootstrap 95% intervals over
  tasks. Mechanism naming is a secondary measure judged by Opus 5.5 and
  Astra, blind to the author model. Both must agree, and they do on 98%
  of 1,684 conclusions. A keyword match had 64% precision against the
  judges' consensus.
- **Models.** Six via Amazon Bedrock (GPT-6 Astra, Claude Fable 5.2,
  Opus 5.5, Sonnet 5.5, Haiku 5.5, Sonnet 4.5). Three self-hosted with
  SGLang (Qwen3.8-27B, Muse Glimmer 30B, Gemma 4 31B).
  - Shared harness, prompt and limits: 60 tool calls, 80 turns, 32k
    output tokens per turn, 600k cumulative input tokens. At the limit,
    one final turn allows only `submit`.
  - Defaults differ by model: Opus and Haiku 5.5 medium, Sonnet 5.5
    high, Sonnet 4.5 no thinking.
  - Fable 5.2 was added later under a pre-registered addendum. It
    "rejects a changed tool list once its reasoning is in the
    conversation", so for it the final turn keeps every tool and refuses
    all but `submit`.
- **Hint ablation.** Three appended paragraphs (B, H, S; quoted in App.
  E), for the three Claude 5.5 models only. 66 not-at-ceiling tasks, 3
  runs each. S was written after inspecting Astra on the test split. It
  was then re-tested on a fresh split, pre-registered with one primary
  outcome: 63 tasks, 34 physics-wrong.
- **Effort ablation.** Pre-registered after a 20-episode development
  pilot. H1: Opus at max beats its default. H2: Opus at max is then not
  worse than Astra's default. Addenda: Opus high, other models' highest
  settings, Fable low/medium/max. Secondary rows have one run per task
  against three-run defaults.
- **Cost.** About $1,065 for main runs and hints, $1,097 for the effort
  study and $793 for Fable 5.2, plus ~$100 rerun after a harness error.

## Results

- **Main table (Table 3, physics-wrong, default).**

  | Model (default) | Interval score | Coverage | Best tested (setting) |
  |---|---|---|---|
  | Fable 5.2 (adaptive) | 0.020 | 99% | same |
  | GPT-6 Astra | 0.037 | 94% | 0.018 (max) |
  | Sonnet 5.5 (high) | 0.75 | 56% | 0.097 (max) |
  | Opus 5.5 (medium) | 0.77 | 62% | 0.028 (xhigh) |
  | Haiku 5.5 (medium) | 1.35 | 49% | 0.12 (max) |
  | Sonnet 4.5 | 1.39 | 39% | 1.33 (thinking) |
  | Qwen3.8-27B | 1.43 | 38% | – |
  | Muse Glimmer 30B | 1.76 | 20% | 1.73 (xhigh) |
  | Gemma 4 31B | 2.07 | 22% | same |

  - Fable beats Astra 1.9× [1.3–2.7×]. Astra beats Sonnet 5.5 20×
    [13–32×]. Sonnet 5.5 and Opus 5.5 cannot be separated.
  - Controls: every model's median interval score is ≤0.010.
  - T6 menu vs no-right-candidate (only 3 tasks): Opus coverage 1.00 vs
    0.30, Sonnet 5.5 1.00 vs 0.15.
  - Hidden-target prediction falls within range in every physics-wrong
    episode for the current Claude models and Astra. Prediction does not
    expose the failure.
- **Behaviour (Table 11, physics-wrong, default).**
  - Names the effect: Astra 0.66, Fable 0.89, Opus 0.35, Sonnet 5.5
    0.25, Haiku 0.14.
  - Low-power experiment: 0.70 / 0.50 / 0.08 / 0.42 / 0.07.
  - Answer from one dataset: 0.96 / 0.94 / 0.39 / 0.56 / 0.19.
  - Final misfit, in noise units: 1.4 / 1.5 / 21.2 / 16.3 / 25.8.
  - Experiments used (of 4): Astra 4.0, Fable 4.0, Opus 2.1, Sonnet 2.6,
    Haiku 2.6.
  - Sonnet 5.5 runs a weak experiment in 42% of episodes, but in 37 of
    those 48 its final fit is to data the simulator does not fit.
- **Hints (Table 12; fresh split, Table 14).** Paired against no hint:

  | Model | H: widen | B: practice | S: strategy (test) | S (fresh, prereg) | Δ coverage with S, test / fresh |
  |---|---|---|---|---|---|
  | Opus 5.5 | n.d. | 1.4× [1.2–1.8] | 2.5× [1.6–3.8] | 2.2× [1.5–3.3] | +4 [−2, +10] / +1 |
  | Sonnet 5.5 | n.d. | n.d. | 1.6× [1.2–2.3] | 1.7× [1.2–2.4] | +0 [−5, +6] / +0 |
  | Haiku 5.5 | n.d. | n.d. | 1.8× [1.4–2.5] | 1.7× [1.3–2.2] | +7 [+1, +14] / +4 |

  - n.d. = no detectable change.
  - Behaviour under S on the fresh split. Opus's low-power experiments
    went 6% → 93% and answers from one dataset 39% → 89%. Typical
    estimate error went 13% → 6%. The ranges narrowed with it.
  - By mechanism (Table 13), Opus coverage without S → with S:
    - k(T): 0.73 → 0.98.
    - Glued joint: 0.77 → 0.80.
    - Radiation: 0.70 → 0.63, with lower error.
    - Edge loss: 0.35 → 0.32.

    Control scores are unaffected.
- **Effort (Table 4, 38 physics-wrong tasks of the 66-task set).**

  | Model | Setting | vs own default | Coverage | Tokens/ep | $/ep |
  |---|---|---|---|---|---|
  | Astra | default | – | 0.94 | 3.3k | 1.04 |
  | Astra | max | 2.1× better | 1.00 | 19.2k | 3.10 |
  | Fable 5.2 | low / medium | 12× / 5.5× worse | 0.80 / 0.85 | 2.8k / 4.1k | 0.24 / 0.35 |
  | Fable 5.2 | default | – | 0.99 | 20.8k | 1.56 |
  | Fable 5.2 | max | 1.4× worse | 0.98 | 98.3k | 5.55 |
  | Opus 5.5 | default (medium) | – | 0.62 | 3.5k | 0.14 |
  | Opus 5.5 | high | 2.9× better | 0.76 | 5.1k | 0.28 |
  | Opus 5.5 | xhigh | 28× better | 0.97 | 47.2k | 1.54 |
  | Opus 5.5 | max | 23× better | 0.93 | 75.7k | 2.96 |
  | Sonnet 5.5 | max | 7.7× better | 0.89 | 86.1k | 1.61 |
  | Haiku 5.5 | max | 11× better | 0.90 | 95.8k | 0.09 |
  | Sonnet 4.5 / Gemma / Muse | highest | n.d. | 0.45 / 0.21 / 0.17 | 12.2k / 8.2k / 20.0k | – |

  - H1 holds: Δlog10 −1.36 [−1.52, −1.20].
  - H2's criterion is met: Opus max vs Astra default 0.78–1.65×. Astra
    could still be up to ~1.3× better. At both providers' max, Opus is
    1.9× worse [1.5–2.5×] than Astra.
  - Opus's diagnosis rate rises with effort: 0.35 (default), 0.65
    (high), 0.92 (xhigh), 0.93 (max). At max, 99% of answers come from
    one dataset, but only 18% use a low-power experiment (Astra 70%),
    and the kept dataset's misfit is 5.1× noise.
  - Haiku 5.5 at max (interval score 0.12, $0.09) beats Opus 5.5 at
    default (0.77, $0.14) by about 6× at about two-thirds of the price
    (our arithmetic). It writes 95.8k tokens per episode to do so, 27×
    Opus's default.
  - Billing differs by source. Fable's bill is ~70% output, Astra's ~80%
    cache writes. The authors treat tokens as the primary efficiency
    measure.
- **Budget censoring (§7).**
  - 33 of 198 Opus-max episodes, 7 of them on physics-wrong tasks,
    reached the 600k input cap and did not call `submit` in the final
    turn (submission rate 83%). 16 of the 33 state "I've submitted the
    results" or similar.
  - Opus xhigh and Astra max stayed within the cap.
  - The pre-specified 2× rerun of the 33: 22 now submit, submission rate
    94%, coverage 0.99, H1 unchanged, and max vs xhigh not detectably
    different [0.58–1.28×].
  - Fable max: 60 of 66 episodes reached the cap and submitted in the
    final turn.
  - Muse at xhigh failed to submit in 8% of physics-wrong episodes.
- **Run-to-run spread (Table 10).** The mean per-task SD of log10
  interval score is 0.05–0.21, small against the between-model gaps.

## Critique / open questions

- **Against the digest entry: above.** The bullet's two numbers are
  right, but it describes a weaker and narrower paper than the PDF.
  - **"With a flawed simulator, models give 95% intervals that cover
    the truth 20–62% of the time."**
    - The bullet says "flawed simulator", but only the *physics-wrong*
      flaw breaks the models. Under-resolved solvers (T3) are handled at
      1.00 coverage by every current Claude model.
    - "Models" leaves out the two calibrated ones: GPT-6 Astra (94%) and
      Claude Fable 5.2 (99%). The range 20–62% is right for the other
      seven.
  - **"The Claude 5.5 correction is latent and appears only at 13–31×
    the reasoning."**
    - It is right for Opus. The 13–31× range spans the three models'
      top-setting-to-default token ratios: Haiku 12.9×, Opus 13.5×
      (xhigh) and 21.6× (max), Sonnet 31.9× (our arithmetic; the paper
      rounds to 31).
    - Only Opus reaches Astra's default. Sonnet and Haiku improve 7.7×
      and 11× and stay behind.
    - "Only" overstates. Opus at *high* (1.5× tokens) is already 2.9×
      better, which beats the best hint.
  - **What the bullet misses is what bears on this box:**
    - the pre-registered, fresh-split-replicated hint ablation on the
      Claude 5.5 family (concepts/intent-level-briefing);
    - effort as a lever bigger than any brief
      (concepts/hybrid-model-backends);
    - the input-token cap that censored the high-effort rows, and the
      16 false "submitted" claims (concepts/budget-as-ceiling).
  - **The cut reason does not hold on the full read.** No reason is
    given in the bullet's parenthetical. The near-miss rule offers
    overlap, small n or method-first.
    - Not small n: 2,511 main episodes, 3 runs per task, paired
      bootstrap CIs, two pre-registrations.
    - Not method-first: it is a benchmark whose results are
      ablations, not a system pitch.
    - No overlap with a listed item: nothing in items 1–20 tests
      calibration under model discrepancy, hint content, or effort
      settings.

    The one fair reason is domain: physical-parameter estimation, not
    an ML-research loop. The project scope admits it because the
    patterns transfer.
  - No check-at-ingest hint in the entry. The PDF is the work described
    (arXiv 2611.21541v1, 16 Nov 2026). The ID and both numbers check
    out.
- **"Every model's fits looked excellent" (Lesson 1) is visual, not
  statistical.** The failing models' final fits sit at 14–30× noise
  (Table 11). `fit_parameters` returns per-sensor misfit, and `analyze`
  offers a runs test. The evidence of inadequacy was in the agent's own
  tool output. The failure is acting on it, which is the
  "noticing without acting" pattern in evidence-gated-completion, more
  than not seeing it.
- **The default ranking compares provider defaults, not models.**
  - Opus 5.5 runs at medium by default, Sonnet 5.5 at high.
  - "Sonnet 5.5 = Opus 5.5" at default, and Fable 5.2 as the "most
    accurate model at default", are statements about effort defaults on
    Bedrock as much as about models.
  - Claude Code and the first-party API may set different defaults. The
    paper does not test them.
- **Several effort claims rest on one run per task against a three-run
  default.**
  - These are xhigh, Sonnet/Haiku/Astra max, all Fable non-default rows
    and the open-weight addenda.
  - The xhigh "28×" and "same gain at half the cost" are single-run
    rows.
  - The authors say the highest settings are not equivalent across
    vendors. For Sonnet 4.5 and Muse they barely changed the reasoning
    done.
- **Fable 5.2 at max is censored and was not rerun.** 60 of 66 episodes
  hit the cap. "At max it does not improve (1.4× worse)" is mostly a
  statement about the cap. The paper flags this, but its per-token
  conclusions about Fable rest partly on that row.
- **Hint S was written from the test split.** It replicated on a fresh,
  pre-registered split with similar sizes. It is still a strategy the
  authors supplied after watching a stronger model, one wording.
  - There is no length-matched neutral paragraph. B (general practice)
    is the nearest active control, and it did little except for Opus.
  - Hints were not crossed with effort. Whether S adds to xhigh is
    untested.
  - Fable was not hinted.
- **Some of the "missing physics" difficulty is task design.** For k(T)
  tasks the graded value is room-temperature k. That is the simulator's
  own parameter form, and a temperature average is defensible
  (limitation v). The v0.2 wording addresses it. Hard-heating answers
  are wrong partly by the grading convention.
- **Experiment budget.** Astra and Fable use all four experiments. Opus
  uses 2.1. Whether a larger budget would widen or narrow the gap is
  untested. The failing models were not limited by the cap.
- **Mechanism-naming judges are two of the tested models** (Opus 5.5,
  Astra), with no human labels. It is a secondary measure and in no
  score. The authors report no own-family favouritism.
- **Minor inconsistencies.**
  - Qwen's physics-wrong coverage is 38% in Table 3/Fig. 2 and 0.42 in
    Table 11. Muse is 20% vs 0.21. These are probably different
    denominators, given 11 Qwen endpoint timeouts.
  - The Fable-vs-Astra CI is [1.3–2.7×] in the text and [1.3–2.8×] in
    Table 9.
  - The 2,511 episodes are the main condition only. Table 6 lists 6,646
    episodes in all (our arithmetic).
- **Consistent with neighbours in the graph.**
  - [[literature/papers/sun2026scipivot]] (cited as [34]): judgment at
    research pivots as a distinct capability from procedure.
  - [[literature/papers/hickey2026saltbench]]: a cap correlated with
    the condition censors the arm that spends more. Here that arm is
    the high-effort setting.
  - [[literature/papers/wang2026beneath]]: a generic restatement of
    what the prompt already says buys nothing. Here H restates the
    prompt's "too narrow intervals … are penalized heavily".

## Trust signals

- **Credibility:** 4.
  - For:
    - A US national laboratory (LLNL, LDRD-funded) and an
      environment, generator, graders, harness, analysis scripts and
      pre-registrations named as released under CC BY 4.0.
    - Judge-free primary scoring with a proper scoring rule. The
      simulator is verified against exact solutions, and a
      scripted-policy gate ran before any model.
    - A held-out split frozen with checksums.
    - Three runs per task with paired bootstrap CIs.
    - Two pre-registrations: the effort test, and the fresh-split
      re-test of S, committed before its tasks existed. Both were
      checked, and the deviations are stated.
    - A pre-specified sensitivity rerun for the budget artefact.
    - The abstract's numbers match the body.
  - Against:
    - Not peer reviewed (journal report number, arXiv preprint). Eight
      days old, no citations.
    - One physical domain.
    - Many secondary rows are single runs, and the effort settings are
      not equivalent across vendors.
    - Hint S was derived from the test split.
    - The mechanism judges are tested models.
    - The code was not opened at ingest, and the held-out traces are
      withheld.
  - Not 5: unreviewed and one domain. Not 3: the design controls
    (pre-registration, fresh-split replication, judge-free proper
    scoring, verified simulator, scripted gate, sensitivity rerun) are
    well above the graph's typical preprint.

## Follow-up

- **Relevance:** 4.
  - It is the first evidence in the graph that the reasoning-effort
    setting is a lever larger than prompt content, on the Claude 5.5
    models this box runs. Opus 5.5 went 23–28× from effort against
    2.2–2.5× from the best hint.
  - It carries a pre-registered, replicated hint ablation that
    separates a restated "be careful" from a stated missing action
    (concepts/intent-level-briefing, P2).
  - Material new evidence for concepts/hybrid-model-backends (effort as
    a per-role parameter) and concepts/budget-as-ceiling (a uniform
    input cap censors the high-effort arm; false "submitted" claims at
    the cap).
  - Not 5: physical-parameter estimation, not an ML-research loop. No
    concept here rests on it alone.
- **For `/elevate` (narrow).**
  - `budget.yaml`'s model roles name a model ("opus") and no effort
    level. On this task family the effort level changed Opus 5.5's
    result more than any brief did. Most of the change came between
    high and xhigh.
  - A clarifying change (record the effort a role runs at) passes a
    modest evidence gate. A rule ("raise effort before rewriting a
    brief") does not, on one paper in one domain.
- **Candidates** (cited, not yet in the graph):
  - Brynjarsdóttir & O'Hagan 2014, "Learning about physical parameters:
    the importance of model discrepancy" (Inverse Problems). The
    statistical statement of the failure.
  - Epstein et al. 2025, "FermiEval" (arXiv 2510.26995). Qin &
    Andriushchenko 2026, "QuantSightBench" (arXiv 2604.15859). Interval
    calibration of LLMs, non-agentic.
  - Oikonomou & Raonic 2026, "The verifier is part of the simulator:
    evidence-view gaming in agentic equation discovery" (Sim2Sci
    workshop). Xu et al. 2026, "Calibration, not compilation" (arXiv
    2606.31630): LLMs miss misspecification and gain confidence from
    passing tests.
  - Jaffe & Sherburn 2026, "TasteVal" (arXiv 2610.06824). Already a
    candidate in kanke2026rtaste.
  - Luo et al. 2026, "Agentic abstention" (arXiv 2606.28733). Liu et
    al. 2025, "CaRT" (arXiv 2510.08517). Ding et al. 2026,
    "Calibrate-then-act" (arXiv 2602.16699). Stopping and evidence
    sufficiency.
