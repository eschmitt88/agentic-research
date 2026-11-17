---
kind: paper
title: "From AutoResearch to Continual AutoResearch: Do Agents Learn Across Tasks?"
authors: ["Jeremy Qin", "Ruizhi Deng", "Mohamed Osama Ahmed", "Frederick Tung", "Leo Feng"]
institutions: ["RBC Borealis", "ELLIS Institute Tübingen", "Max Planck Institute for Intelligent Systems, Tübingen AI Center"]
year: 2026
venue: "arXiv 2611.05108 (cs.AI), v1 2026-11-03, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.05108"
code_url: null
citations: null
source: "raw/papers/qin2026autoresearch.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["continual-autoresearch", "cross-task-transfer", "stateful-vs-reset", "persistent-session", "restart-vs-continue", "time-convergence", "best-so-far-curve", "anytime-performance", "first-valid-latency", "insight-cards", "memory-ablation", "shuffled-memory-control", "selection-regret", "dev-overfitting", "operational-lesson-overapplied", "anchoring", "one-run-per-cell", "claude-code", "codex", "opencode", "opus-5", "sonnet-4-6", "gpt-5-6-sol", "kimi-k3", "mle-bench-lite", "sela", "symbolic-regression", "time-series-forecasting"]
---

# From AutoResearch to Continual AutoResearch: Do Agents Learn Across Tasks?

## TL;DR

The same CLI agent works through an ordered stream of 3–5 related ML tasks
twice: once **stateful** (one native session and workspace resumed across the
stream) and once **reset** (fresh session and clean workspace per task), at a
30-minute wall-clock budget per task. Nine streams, five agents (Opus 5 and
Sonnet 4.6 in Claude Code, GPT-5.4 and GPT-5.6-Sol in Codex, Kimi K3 in
OpenCode), 32 transfer tasks per agent, scored on hidden test sets after the
run. Four results matter here:

1. **Persistent state finds good solutions sooner.** The mean "time-
   convergence" gain (normalized area under the hidden-test best-so-far curve)
   is positive for all five agents: +21.3 Kimi K3, +20.9 Sonnet 4.6, +7.5
   Opus 5, +6.1 GPT-5.6-Sol, +3.0 GPT-5.4. The stateful lead in best reward
   averages +24.0 points at 25% of the budget and +1.2 at the deadline.
2. **It does not raise the final result**, except for Sonnet 4.6 (+7.6). The
   others are at +0.4, +0.2, −0.8 and −0.9 (Opus 5). Opus 5 wins only 34.3% of
   transfer tasks under persistence.
3. **Most of the speed-up is reaching the first valid submission earlier.**
   Matching candidate counts removes the gain for Opus 5 (−0.5) and
   GPT-5.6-Sol (+0.1); only Sonnet keeps a clear one (+6.5). Subtracting each
   run's first-valid reward makes the gain negative for every agent.
4. **Relevant memory moves the start, not the finish.** In a fresh-session
   ablation on one stream, agent-written "insight cards" from the same stream
   vs a shuffled stream change first-solution quality (+2.6 to +5.9 for three
   agents) but not final outcome (−3.2 to +1.0), and they *slow* time-
   convergence for three of four agents.

Each agent × stream cell is **one** matched stateful/reset pair, except three
repeats for GPT-5.6-Sol, whose time-convergence gain varied by up to 7.14
points between repeats.

## Claims

- Existing autoresearch evaluations reset the agent per task, so they cannot
  show whether lessons about experimental design, modelling choices or
  execution failures carry over.
- Retaining experience and using it well are separate capabilities.
  Persistence helps agents produce stronger solutions sooner, but weak
  follow-up search, anchoring and poor final selection can erase the benefit.
- Persistence is worth most when budgets are tight: reset agents partly
  catch up through within-task search as the budget grows.
- Two failure modes of carried experience: applying an earlier lesson too
  broadly, and finding a strong candidate early then replacing it on
  unreliable development evidence. The authors argue for provenance,
  explicit applicability conditions and evidence-based revision of memories,
  and for process-level evaluation, since final score alone cannot separate
  late discovery, execution failure, weak generation and bad selection.

## Methods

- **Setting.** Ordered stream T1..TK. Each task: description, train and dev
  data, a visible dev scorer, a wall-clock budget B (default 30 min). The
  agent may submit many solutions and must pick a final one. Hidden test
  scores are never shown during the run (anti-reward-hacking). No harness
  memory or planning is added; each agent uses its native CLI.
- **Stateful** resumes the same native session and workspace across the
  stream: transcript, files, results, traces. **Stateless** starts a new
  session and clean workspace per task; everything else is matched. T1 is
  excluded from transfer results.
- **Streams (Table 1, App. A).** Nine families, four settings:
  - Synthetic AutoML (3 streams × 5 classification tasks on one shared
    6000 × 150 feature matrix: shared latent, shared sparse, shared regime).
  - Symbolic regression (1 stream, 5 "canonical coordinates" equations that
    simplify after a change of variables; task 4 is an asymmetric control).
  - Time-series forecasting (ETT cross-resolution, PEMS traffic, energy
    systems; RMSE on chronological test splits).
  - Tabular AutoML (5-task SELA subset; 3-task MLE-Bench Lite slice).
  Dataset names are anonymized to stop agents recognizing benchmarks.
- **Normalization.** Accuracy/F1 as is; RMSE → (1+m)⁻¹ on standardized
  data; R² → (2 − min(m,1))⁻¹. Gains are 100 × (stateful − stateless), first
  averaged within a stream, then across streams with equal weight.
- **Three metrics.** Outcome (final selected submission), initialization
  (first valid submission) and time-convergence: U = (1/B)∫ max-so-far hidden
  reward dt, zero before the first valid submission, last value carried to
  the deadline if a run ends early. This is the "research efficiency" of the
  abstract. It is computed post hoc on the **hidden test** scores of every
  submission, not on what the agent saw.
- **Repeats.** One matched pair per agent × stream. GPT-5.6-Sol was repeated
  three times (Fig. 4).
- **Ablations.**
  - Per-task budget 15/30/60 min, GPT-5.6-Sol, shared-regime stream, 3
    repeats each (Fig. 3).
  - Stream length 3/5/7 ending on a common weather-forecasting task,
    GPT-5.4, one run each (Table 11).
  - **Memory ablation** (Energy-systems stream, 4 transfer tasks, Opus 5,
    Sonnet 4.6, GPT-5.4, GPT-5.6-Sol; no Kimi). Every task starts a fresh
    session. Information transfers only through agent-authored "insight
    cards" (claim, evidence, confidence, scope, recommended actions, failure
    conditions). Arms: matched cards (all cards from earlier tasks in this
    stream), shuffled cards (same number, from an unrelated stream), no
    cards. "No memory" is described as an "independent no-memory reference".
- **Decompositions (App. B.1).** Time to first valid submission; valid
  candidate counts; gain at matched candidate count (first min(n_S, n_R)
  candidates); gain with each trajectory started at its first valid
  submission; gain with the first-valid reward also subtracted.

## Results

- **Model level (Table 2, 32 transfer tasks each; mean [median]).**

  | Agent | Reset outcome | Outcome gain | Init. gain | Time-conv. gain | Outcome win rate |
  |---|---|---|---|---|---|
  | Opus 5 | 79.9 | −0.9 [+0.0] | +1.3 [+0.5] | +7.5 [+10.3] | 34.3% |
  | GPT-5.6-Sol | 80.5 | −0.8 [+0.2] | −1.0 [+1.2] | +6.1 [+7.4] | 59.3% |
  | Kimi K3 | 73.1 | +0.4 [+2.0] | +1.8 [+2.0] | +21.3 [+27.5] | 62.0% |
  | Sonnet 4.6 | 70.3 | +7.6 [+1.3] | +7.0 [+7.0] | +20.9 [+20.6] | 66.7% |
  | GPT-5.4 | 72.9 | +0.2 [−0.2] | +3.0 [+3.1] | +3.0 [+2.0] | 51.9% |

  Negative-transfer rate (Table 4): Opus 5 65.7%, GPT-5.6-Sol 40.7%, Kimi
  38.0%, Sonnet 33.3%, GPT-5.4 48.1%.
- **Run-to-run noise.** Across GPT-5.6-Sol's three repeats, outcome and
  init gains varied by at most 0.75 points, time-convergence by up to 7.14.
  Per stream (Fig. 4), time-convergence spreads reach roughly ±20 points
  (PEMS, ETT). GPT-5.4's +3.0 and GPT-5.6-Sol's +6.1 time-convergence gains
  are inside that range; Opus 5's +7.5 is at its edge.
- **Where the outcome gain comes from (Fig. 8).** Sonnet's +7.6 is mostly two
  streams: canonical coordinates +41.2 and energy +18.5; the other seven are
  −0.3 to +5.0. Kimi has −10.9 on canonical coordinates. Sonnet's reset runs
  are the weakest (70.3), and its reset runs failed far more often (failed
  attempts S/R 15/71; failure fraction 30.7% reset vs 7.7% stateful; App.
  B.3). Part of Sonnet's gain may be operational reliability, not insight.
- **Over the budget (Table 8).** Stateful-minus-reset best hidden reward at
  25/50/75/100% of budget: mean +24.0 / +15.7 / +6.5 / +1.2. Opus 5 goes
  +18.3 / +19.8 / −2.8 / −0.8; at 100% only Kimi (+1.7) and Sonnet (+6.9)
  are positive.
- **Decomposition (Table 7).** Time to first valid, stateful/reset (min):
  Opus 7.7/11.1, GPT-5.6-Sol 6.1/8.5, Kimi 9.5/18.7, Sonnet 7.7/16.3, GPT-5.4
  8.4/9.2. Stateful runs made fewer valid candidates for four of five agents
  (e.g. GPT-5.6-Sol 6.7 vs 11.8). Gain at matched candidate count: −0.5,
  +0.1, +1.7, +6.5, +1.2. Gain excluding first-solution latency: −0.3, −1.1,
  +1.5, +6.6, +0.5. Incremental area gain after subtracting first-valid
  reward (Table 10): −1.0, −0.3, −0.2, −0.2, −3.2. First-to-best improvement
  S/R is about equal or lower under persistence (Opus 1.8/3.3, GPT-5.4
  1.1/5.3).
- **Selection regret** (best hidden reward reached minus the selected one):
  the stateful-minus-reset difference is −0.7 to +1.3 points, within ±0.8 for
  four agents. Selection does not explain the gap between faster search and
  flat outcomes in aggregate, though single cases are severe (D.2).
- **Task position (Figs. 10–11).** On the synthetic streams the stateful
  search advantage grows over task position for four agents (not Sonnet).
- **By task nature (Table 5).** Classification: outcome +0.8, win 60%;
  regression: +0.8, win 50%; symbolic regression: +5.2 mean but −2.9 median.
- **Budget ablation (Fig. 3).** As the budget rises 15 → 30 → 60 min, the
  outcome gain falls (about +1.5 → +0.4 → +0.4), the init gain rises (about
  +2 → +7.5 → +10.7) and the time-convergence gain falls toward zero or
  below. Longer earlier tasks give the stateful agent more to bring to its
  first solution; longer current tasks let the reset agent catch up.
- **Stream length (Table 11, one run each).** 2/4/6 prior tasks: init
  +3.56 / +5.53 / +5.34; time-convergence −1.75 / −3.61 / −31.44; outcome
  −1.82 / −0.05 / −0.27.
- **Memory ablation (Table 12, 4 tasks per model; matched minus shuffled).**

  | Model | Outcome M/S | Outcome M/N | Init. M/S | Time-conv. M/S |
  |---|---|---|---|---|
  | Opus 5 | +1.0 | +0.2 | +2.6 | −15.5 |
  | GPT-5.6-Sol | −3.2 | +0.3 | +3.9 | −27.3 |
  | Sonnet 4.6 | −0.2 | +14.7 | −2.9 | +25.6 |
  | GPT-5.4 | +0.2 | +5.0 | +5.9 | −0.9 |

  Shuffled cards beat no memory by about as much as matched cards (+14.9
  Sonnet, +4.8 GPT-5.4), so the authors say the no-memory comparison shows
  no relevance benefit. Matched-minus-shuffled best reward by checkpoint
  (Table 13), mean −13.0 / −5.4 / −4.2 / −0.6 at 25/50/75/100%.
- **Case studies (App. C–D; chosen for interpretability).**
  - Stateful Sonnet reused its whole ETT pipeline and recalibrated it; its
    first submission (RMSE 0.532) beat the reset run's final (0.596).
  - Matched cards gave Codex the seasonal-residual idea; it re-checked the
    period (lag 288, not 96). First RMSE 0.545 vs 1.406 with shuffled cards;
    final 0.413 vs 0.477.
  - With 21 matched cards closely describing the target schema, Sonnet
    anchored on a correction model (first = final 0.588). Shuffled cards
    gave 0.543, and the no-memory reference 0.538.
  - Kimi found a sin-product expression early (hidden R² 0.593), then
    dev-polished to dev R² 0.99979 and selected a version with hidden R²
    −1.354 (selection regret 41.3 points).
  - GPT-5.6-Sol found a good candidate in 4.5 min (hidden RMSE 0.545), then
    promoted a dev-tuned ensemble at 1.049 (reset final 0.523): +55.0
    time-convergence, −16.9 outcome.
  - Opus 5 lost task 1 of MLE-Bench by blocking on a 5-model CV suite, wrote
    the persistent policy "write a valid final within the first ~3 minutes,
    before any tuning", and then explored too little: 4 valid attempts
    across the stream vs 28 for reset; ROC-AUC 0.978 vs 0.998,
    time-convergence −85.4. Host contention also affected this stream.

## Critique / open questions

- **Full text vs the digest entry: at, with one correction.** "Improves
  research efficiency for all five agents, but not final held-out
  performance" is the abstract's own claim, and the tables bear out the
  sign. Three qualifications the summary drops:
  1. "Efficiency" is anytime hidden-test reward over wall-clock, and most
     of it is time to the first valid submission. In units of candidates
     (closer to `/iterate`'s cycles) the gain is near zero for Opus 5 and
     GPT-5.6-Sol.
  2. The "all five" claim rests on one pair per cell. Two of the five gains
     (+3.0, +6.1) are inside the 7.14-point repeat spread the paper itself
     measured for one agent.
  3. Sonnet 4.6 *does* gain on final outcome (+7.6), concentrated in two
     streams and coinciding with a much higher reset failure rate.
- **The memory ablation does not separate insight transfer from session
  continuity.** All three arms are fresh sessions; it separates *relevant*
  from *irrelevant* cards, and cards from none. No arm compares cards
  against the persistent session on the same stream, so how much of the
  main stateful effect is the transcript, the files and code, or the
  insights is not measured. The authors say as much in Limitations ("future
  work should separately test insights, code, transcripts, and execution
  state"). It is also one stream, four transfer tasks, one run per cell,
  four models.
- **Wall-clock metrics on a shared host.** Time-convergence and time-to-
  first-valid are wall-clock, and the authors note host contention on at
  least one stream. Token or dollar cost per arm is not reported. A
  persistent session also carries a growing context; nothing reports
  whether any agent compacted, or what the stateful arm cost.
- **The stateful arm keeps the workspace, so "transfer" includes copying
  code.** C.1 is direct pipeline reuse. On streams built so that the
  previous solution is close (ETT, synthetic shared-data streams), earlier
  first-valid times are expected from file reuse alone.
- **Hidden-test curves are post hoc.** That is right for measurement, but
  it means "efficiency" counts candidates the agent itself had no way to
  know were good. A stateful run that generated a strong early candidate
  and then discarded it scores well on time-convergence and badly on
  outcome (D.2: +55.0 vs −16.9). The metric credits generation, not the
  research outcome.
- **No statistics.** No CIs, no tests; means and medians over 4-task
  families and equal family weights. Several family cells are 2–3 tasks.
- **Opus 5 is the agent this box runs, and it is the worst case here:**
  outcome −0.9, win rate 34.3%, negative transfer 65.7%, candidate-matched
  gain −0.5, and the D.3 overapplied-lesson failure. One run per cell, so
  this is suggestive, not a finding about Opus.
- **What it does not test.** Within-problem continuation (the same task
  resumed), which is what `/iterate` chains do. Long streams (≤ 5 tasks),
  long budgets (30 min; PostTrainBench-scale ~10 h named as future work),
  compaction, or any mechanism for selecting or forgetting memories.
- **Fit with the graph.** Agrees with [[literature/papers/pham2026reliable]]
  that a self-curated store served whole is not reliably positive, and
  adds the shuffled-memory control that pham lacks. The D.1/D.2 failures
  are dev-overfit selection, the failure the evolutionary-expansion
  "return the best" and repeated-draw sections address
  ([[literature/papers/wang2026paretorsigym]],
  [[literature/papers/wang2026beneath]]). The budget result echoes
  [[literature/papers/ellington2026automatic]]: a better start is not a
  higher finish when the budget lets the cold run catch up.

## Trust signals

- **Credibility:** 3.
  - For: RBC Borealis research lab; five current frontier agents through
    their native harnesses; nine streams including synthetic ones with
    known transferable structure; anonymized datasets; hidden test sets
    never shown to the agent; three complementary metrics with explicit
    robustness decompositions that cut against the headline (candidate-
    matched, post-first, incremental area); a shuffled-memory control;
    failure cases reported alongside successes; stream construction given
    in enough detail to rebuild.
  - Against: preprint, two weeks old, no citations. Code "will be released
    alongside the camera-ready version", not now. One run per cell for
    four of five agents, no CIs or tests, and the one repeat study shows
    noise as large as two of the five headline efficiency gains. Ablations
    are single-stream and mostly single-run. Wall-clock metrics on a host
    with acknowledged contention. LLM assistance in code, plots and writing
    disclosed.
  - Not 4: no code, no peer review, and the replication is one agent. Not 2:
    a known industry lab, a careful matched design, and the authors' own
    controls are reported even where they shrink the result.

## Follow-up

- **Relevance:** 4. It is the first matched persistent-vs-reset comparison
  of frontier ML-research agents scored on held-out data, which is the
  question behind the 11-14 decision that a forced `/iterate` pivot should
  keep the chain's state. It adds material evidence to
  [[concepts/budget-as-ceiling]] (carried state front-loads gains and its
  value falls toward zero as the budget is spent) and
  [[concepts/agent-native-memory]] (relevant agent-written memory moves
  the start, not the finish; an operational lesson without applicability
  conditions overapplies). Not 5: cross-task rather than within-chain
  state, one run per cell, and no code yet.
- **For `/iterate` (our inference).** The 11-14 decision rests on
  [[literature/papers/srinivasan2026continue]] and
  [[literature/papers/zhou2026athanor]], where a cold restart re-pays setup
  inside a shared cap. This paper is consistent with that, and adds that
  the advantage of keeping state is mostly *latency to the first valid
  result*. `/iterate` counts cycles, not wall-clock, and at matched
  candidate counts Opus 5's advantage was −0.5. So keeping state in a
  pivot is cheap insurance against re-paying setup, not a source of a
  higher final score. Nothing here argues for a cold restart either.
- **For the box's auto-loaded memory (our inference).** D.3 is the shape of
  an auto-memory feedback entry going wrong: a correct lesson from one
  failure ("ship a valid final first") written as an unconditional rule,
  then applied where it cost exploration. The paper's fix, record the
  conditions a lesson was formed under and what would revise it, is a
  cheap format rule for feedback entries.
- **Candidates** (cited, not yet in the graph):
  - Shu et al. 2026, AgentCL: controlled continual-learning streams across
    coding, deep research and language reasoning.
  - Asawa et al. 2026, "Continual learning bench" (arXiv 2606.05661).
  - Rank et al. 2026, PostTrainBench (long-budget autoresearch, ~10 h).
  - Gao, Fang & Zitnik 2026, "AutoScientists" (arXiv 2605.28655).
  - Goel et al. 2026, FutureSim (arXiv 2605.15188).
  - Zhao et al. 2024, ExpeL (AAAI) — the textual-experience-reuse baseline.
