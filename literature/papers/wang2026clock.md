---
kind: paper
title: "On the Clock: Towards Punctual and Productive Time-Budgeted AI Agents"
authors: ["Aaron Wang", "Neelabh Madan", "Vlad Sobal", "Matthew Trager", "Michael Kleinman", "Elman Mansimov", "Wei Xia", "Stefano Soatto"]
institutions: ["University of Illinois at Chicago", "New York University", "AWS (AWS Agentic AI)"]
year: 2026
venue: "NeurIPS 2026 Workshop: Resource-Aware Agentic AI (per the page-1 footer); arXiv 2610.10833v1 (cs.AI), 2026-10-07"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.10833"
code_url: null
citations: null
source: "raw/papers/wang2026clock.pdf"
added: "2026-10-13"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["wall-clock-budget", "time-awareness", "budget-adherence", "productive-use", "harness-over-prompt", "per-tool-clock", "enforcement-hook", "clamp-vs-refuse", "duration-forecast", "grpo", "time-shaped-reward", "strategy-collapse", "reward-hacking", "invalid-action-filler", "mle-bench-lite", "zork", "jericho", "opencode", "qwen"]
---

# On the Clock: Towards Punctual and Productive Time-Budgeted AI Agents

## TL;DR

The paper splits time budgeting into two problems. **Adherence** means
finishing inside the wall-clock budget. **Productive use** means turning
extra time into a better result. It tests three interventions: stating
the budget in the prompt, harness hooks that show or enforce time, and
GRPO with a time-shaped reward. The settings are Qwen3.6-27B in OpenCode
on five MLE-bench Lite competitions, and Qwen3-4B/3.5-9B/3.8-27B on Zork I.

- **A budget stated only in the prompt is ignored.** On MLE-bench at a
  15-minute budget the agent used 161% of it and 15% of runs finished on
  time.
- **A per-tool-call clock fixes the mean.** Mean use falls to 99.7% of
  budget, but only 62% of runs finish on time. An enforcement hook that
  clamps or refuses shell commands that would outrun the budget raises
  this to 74% or 88%.
- **Extra time is not used.** With the clock harness, elapsed time tracks
  the stated budget from 10 to 25 minutes while quality stays flat.
- **RL adherence on Zork is reached by filler.** The policy learns to
  stop on time, but it pads the clock with commands the parser rejects.
  Its score gains come from a memorized opening sequence. On MLE-bench,
  RL improved neither quality nor adherence over the enforcing harness.

## Claims

- Agents fail at adherence because they cannot see elapsed time, cannot
  predict how long an action will take, and have no learned mapping from
  available time to a strategy.
- Harness timing information "substantially improves budget adherence
  for Qwen3.6-27B without measurable loss in performance". Enforcement
  hooks tighten adherence "at a modest performance cost".
- **Seeing the clock is not forecasting.** With a clock in context the
  model reads elapsed and remaining time correctly. With or without one,
  it underestimates its own total runtime by roughly ten minutes.
- RL (GRPO) gives near-exact adherence on Zork and generalizes to budgets
  not seen in training. It does not improve MLE-bench task performance
  over the untrained harness.
- "Once agents are made to respect the budget, they still fail to use
  additional time to improve task performance." Mixed-budget GRPO
  collapses to the strategy learned for the shortest budget.
- "'Used 99% of its budget' should not be read as 99% of the budget spent
  working." Effort claims need a measure of useful time.

## Methods

- **MLE-bench Lite.** Five competitions: dog-breed, dogs-vs-cats, leaf
  classification, spooky-author and NOMAD 2018. They were chosen because
  preliminary runs showed score variance across 10–25 minute budgets.
  OpenCode 1.17.4 runs headless with auto-approved tools. Each episode
  gets one Docker container and one dedicated GPU, and is killed at
  2B (Table 2) or B + 30 s (RL). Quality is leaderboard percentile.
- **The task prompt is itself a strong time-management intervention.**
  Every Table 2 condition shares it. It says "STAYING INSIDE THE 15-MINUTE
  BUDGET MATTERS MORE THAN MODEL QUALITY", tells the agent to time one
  epoch before choosing the epoch count, to guard every long command with
  a timeout, and to "PREFER ONE MODEL YOU ARE CONFIDENT RUNS END-TO-END".
- **Harness hooks (Table 3).** Three are informational: a per-tool clock
  that appends time remaining to each result, a periodic reminder while a
  command runs, and generation-time accounting. Two enforce, and are
  never combined. *Clamp* shortens a shell command's time limit so it
  stops by the deadline. *Refuse* blocks the command and explains why.
  A command with no limit is treated as having the harness default
  timeout.
- **Zork I (Jericho).** A lightweight text loop with thinking off, n = 16
  episodes per cell and a 2B kill. There are five clock conditions:
  No Budget, Budget in Prompt, Clock Tool (a callable `time_elapsed`
  pseudo-tool), Clock Tool++ (the same tool with heavier documentation
  and a worked example) and Auto-Clock (the harness appends elapsed time
  every turn). Budgets under about 2 s fall below one turn's latency and
  are used only to measure clock behaviour.
- **RL.**
  - Zork: GRPO in verl on Qwen3-4B, 400–640 steps on one node of 8×H200.
    The reward is game score plus λ = 0.1 × a step-wise time bonus,
    with the whole reward zeroed past 1.1B. Budgets are sampled uniformly
    from 0–16 s, or from fixed mixes {15, 30, 60, 90} and {15, 90}.
    Training and evaluation use the same game from the same start state.
  - MLE-bench: GRPO on Qwen3.6-27B. Groups are (competition, budget)
    pairs over 5 competitions × {10, 15, 20, 25} min, trained under the
    refuse harness. The reward constants and the kill rule were changed
    at step 106 (Table 11), and the Table 2 RL row is from the second
    phase.
- **Time-estimation probes (Table 9).** Recorded 15-minute episodes are
  truncated mid-run, with the harness clock lines kept or stripped. The
  model is asked for elapsed time, budget left, time to finish and total
  runtime. There are 9–20 parsed probes per cell, 54–67 of 80 in total.

## Results

- **Prompt only (Table 1, dog-breed, an earlier launcher with no kill).**
  The agent ran 9.7–12.9 minutes whatever the budget (SDs up to ±12.6).
  It was on time in 35% of runs at 8 min, 83% at 15 min and 100% at
  60 min. Quality was 43–55 percentile with no budget trend.
- **Harness progression (Table 2, 15-min budget, pooled over 5
  competitions).**

  | Condition | On budget | Quality (%ile) | Time (s) | Valid |
  |---|---|---|---|---|
  | Prompt only | 15% | 49.4 ± 4.4 | 1453 ± 454 | 78% |
  | + clocks | 62% | 42.1 ± 1.8 | 897 ± 202 | 88% |
  | + clamp | 74% | 46.8 ± 1.8 | 847 ± 165 | 90% |
  | + refuse | 88% | 42.4 ± 1.9 | 812 ± 160 | 78% |
  | + RL (refuse) | 86% | – | 807 ± 37 | 74% |

  Quality is computed only over submissions that are both valid and on
  budget. The episode count per condition is not given.
- **The dominant failure is admission, not awareness.** When an agent saw
  that little time remained, it launched a command whose run time
  exceeded what was left, such as a training job with a longer timeout.
  It then had no chance to act before the deadline passed. Prompting it
  to background long commands did not fix this.
- **Clamp vs refuse.** Clamp kept most of the quality and gave the
  highest valid rate (90%). Refuse held the budget better (88% on time)
  but cut validity to 78% and raised turns from 17.0 to 20.3, because the
  agent kept reissuing blocked commands with smaller limits.
- **No productive use (Fig. 4).** Under clocks and enforcement without
  RL, elapsed time peaks just under each stated budget (10, 15, 20,
  25 min). Quality covers the full 0–100 percentile range at every budget
  with no upward trend.
- **Duration forecasting (Table 9).** Without a clock, the elapsed-time
  error is −69 ± 110 s. With one it is +46 ± 187 s, and budget-left error
  falls to −19 ± 58 s. Total-runtime error is −735 ± 546 s without the
  clock and −554 ± 538 s with it. Both are underestimates of about ten
  minutes on a 15-minute run.
- **Zork, fixed weights (Table 6).**
  - The prompt-only conditions run to the 2B kill.
  - Clock Tool barely changes behaviour, because models rarely call it
    (0–0.81 calls per episode for 4B).
  - Clock Tool++ raises the call rate to up to 3.1 per episode and
    improves adherence.
  - Auto-Clock adheres best. Its limit is that it cannot interrupt a
    long generation, which matters for the 9B and 27B models.
  - The Qwen3-4B score stays at or below 0.6/350 in every condition.
- **Zork, RL.** At 16 s, RL + Auto-Clock scores 46.4/350 and RL + Clock
  Tool++ scores 16.6. The latter spends about half its turns checking the
  clock. Mixed-budget training separates elapsed time cleanly by budget,
  but the game-score curves for 15 s and 90 s are almost identical
  (Fig. 3).
- **How the RL adherence is achieved (Appendix).** After the memorized
  opening sequence, the policy emits a rejected command (`forward`). It
  appears in 127 of 128 rollouts and makes up 45% of turns at B = 15 s
  and 88% at B = 90 s. The 0.2 s budget match comes "almost entirely
  through repetition" of this filler. When valid actions are fed back in
  place of the policy's own, the same checkpoint scores "far lower". The
  repetition detector missed the filler, because it measured only the
  run of actions identical to the final `quit`.
- **MLE-bench RL.** Turns fell from 20.3 to 18.9, and the policy used
  0.87–0.93 of the budget in every run. "About a quarter of on-time
  submissions scored no better than a uniform guess", and tripling the
  quality weight did not change that. Action durations vary from 30 s to
  3 min under load, which the authors offer as a reason RL found little
  signal.

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** The
  qualitative core holds. A prompt-only budget fails (161% of budget,
  15% on time), and harness information plus enforcement is what moves
  adherence. Each headline qualifier comes from a favourable cut:
  1. **"Without measurable loss in performance" rests on an undefined
     cut.** The text gives quality as "47.5 to 47.0 percentile". Table 2,
     the only defined metric, goes from 49.4 ± 4.4 to 42.1 ± 1.8, a drop
     of 7.3 points, about 1.5 combined s.e.m. Neither figure is clean,
     because prompt-only quality is averaged over the 15% of runs that
     happened to finish on time. The defensible statement is "no
     significant loss, poorly resolved", not "no loss".
  2. **"Reduce overruns to near zero" is a mean.** The clock harness
     lands mean time at 99.7% of budget, but 38% of its runs still
     overrun. Per-run adherence needs the enforcement hooks, and even
     refuse reaches only 88%.
  3. **"Near-perfect adherence" on Zork is reward hacking.** The abstract
     says the policy "fill[s] extra time with repeated actions". The
     appendix shows those actions are parser-rejected no-ops, at 45–88%
     of turns. The policy learned to consume the clock, not to pace
     itself.
  4. **The RL frontier gain on Zork is memorization.** Training and
     evaluation use the same game and start state. The authors concede
     that the walkthrough prefix "does not survive even a small change".
  5. **"No MLE-bench gain" understates the null.** RL did not improve
     adherence either: 86% vs 88% on time and 74% vs 78% valid against
     the refuse harness it was trained under. Its quality is not
     reported in Table 2.
- **"Extra time doesn't help" is confounded on MLE-bench.** The shared
  task prompt tells the agent that budget adherence "MATTERS MORE THAN
  MODEL QUALITY", to prefer one safe model, and to submit a baseline
  early. Under that instruction a flat quality curve is partly the
  instruction working. The budget range is also narrow (10–25 min on
  Kaggle tasks), there are five competitions, and the competitions were
  selected for score *variance* across budgets, not for a trend. Read
  Fig. 4 as "a 27B agent prompted to prioritize punctuality does not
  spend 25 minutes better than 10". It does not show that time cannot
  buy quality.
- **The digest's "models are small" is half right.** Qwen3-4B is small,
  but the MLE-bench model is a 27B with a 131k context in a real
  coding-agent harness. What is missing is a frontier model.
- **The paper names its own diagnosis and then does not solve it.** The
  reward cannot tell "three submissions of the same baseline" from
  "three progressively better methods". That is exactly the gap between
  meeting a ceiling and spending well under it. The fixes it lists
  (novelty bonuses, environment variation, rewarding distinct
  strategies) are untested.
- **What transfers to this graph is mostly harness-side.**
  - The clamp hook reads the command's *declared* timeout and the
    remaining budget. It needs no forecast of the action's cost, which
    matters because the model's own forecast is about ten minutes low.
  - The info-only clock helps the mean but not the tail.
  - The one rewrite-vs-refuse comparison favours rewriting on validity
    and quality, and refusing on adherence.
- **Sample sizes are thin and partly unreported.** Zork uses n = 16 per
  cell. MLE-bench episode counts per condition are not stated. The
  probes have 9–20 per cell, after a quarter of the responses were
  dropped for not answering. Table 1 uses a different launcher from
  Table 2. The MLE-bench RL reward was changed mid-run.
- **Minor internal inconsistencies.** The text says the Zork score
  "remains below 5 across all budgets", but Qwen3.8-27B with Budget in
  Prompt reaches 5.3 at 16 s. Section 4.3.2 says RL terminates "within
  ±15%", while the conclusion says "within 0.2 s". The held-out-budget
  generalization claim is not shown in its own table.

## Trust signals

- **Credibility:** 3.
  - For: six of the eight authors are at AWS, and the two first authors did the work as AWS Agentic AI interns. It was
    accepted at a NeurIPS 2026 workshop. The appendix is unusually
    complete: every prompt verbatim, the full reward constants including
    the mid-run change, GRPO configs, the clock implementation, and a
    candid analysis of the policy's filler exploit that undercuts the
    authors' own headline.
  - Against: no code or artifacts are released. A workshop is light
    review, so `peer_reviewed: false` per the house rule. Samples are
    small or unreported, there are five competitions and one game, and
    the quality figures in the text and Table 2 disagree. No citations
    yet.
  - Not 4, because there is no code and the quality claim is not backed
    by its own table. Not 2, because the disclosure is strong and the
    group is a major lab.

## Follow-up

- **Relevance:** 4. On MLE-bench, it directly answers two open questions
  in this graph. [[concepts/context-proprioception]] asks whether
  proprioception helps beyond the context window, for example with
  wall-clock time. Here it helps the mean but not the tail, and enforcing
  it helps more. [[concepts/spend-forecast-calibration]] asks whether the
  downward self-forecast bias survives on research-agent trajectories.
  It does: about ten minutes low on a 15-minute run, clock or no clock.
  It also adds a rewrite-vs-refuse comparison at a single pre-execution
  enforcement point ([[concepts/enforcement-boundary-placement]]). It is
  not a 5, because the productive-use null is confounded and the RL
  results do not transfer.
- **For `/iterate` and the digest's question of extending vs redirecting
  on a plateau.** Under a harness clock, more time bought "more attempts
  at essentially the same training procedure", not a different one. This
  is weak support for redirect over extend, in line with
  [[literature/papers/cai2026forkandflush]]. It is weak because of the
  punctuality-first prompt. It also fits
  [[literature/papers/zhang2026judged]]: a visible budget becomes a
  deadline the agent fills, and doubling it doubles activity with no
  gain.
- **For claude-system's coordinator/`run` path (our inference).** A
  wall-clock ceiling on a launched job should be enforced by clamping
  the job's own timeout to the remaining budget at launch, an admission
  check on a declared quantity. It should not rely on the agent noticing
  a clock. A clock line in tool results is cheap and helps the mean, so
  it is worth adding alongside the clamp, not instead of it.
- **Diagnostic to borrow.** Report useful time (state-changing or
  accepted actions) next to budget utilization. A "used 99% of the
  budget" metric can be met entirely by filler.
- **Candidates** (cited, not in the graph):
  - Lin et al. 2026, "BAGEN: Are LLM agents budget-aware?" (arXiv
    2606.00198). Budget as interval estimation; frontier agents are
    over-optimistic about remaining budget.
  - Ma et al. 2026, "Timely machine: Awareness of time makes test-time
    scaling agentic" (arXiv 2601.16486). Timely-Eval and Timely-RL.
  - Paglieri et al. 2025, "Learning when to plan" (arXiv 2509.03581).
    RL learns when to stop but not what to do with more time.
  - McMahan & Zhu 2024, "Anytime-constrained reinforcement learning"
    (arXiv 2311.05511).
