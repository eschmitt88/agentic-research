---
kind: paper
title: "How Do AI Agents Spend Your Money? Analyzing and Predicting Token Consumption in Agentic Coding Tasks"
authors: ["Longju Bai", "Zhemin Huang", "Xingyao Wang", "Jiao Sun", "Rada Mihalcea", "Erik Brynjolfsson", "Alex Pentland", "Jiaxin Pei"]
institutions: ["University of Michigan", "Stanford University", "All Hands AI", "Google DeepMind", "Microsoft AI", "Massachusetts Institute of Technology"]
year: 2026
venue: "arXiv 2604.22750v2, cs.CL (preprint)"
peer_reviewed: false
url: https://arxiv.org/abs/2604.22750
code_url: https://longjubai.github.io/agent_token_consumption/
citations:
source: "raw/papers/bai2026how.pdf"
added: "2026-08-17"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/context-eviction-policy]]"
tags: [budget, token-economics, spend-forecasting, self-prediction, cost-accounting, prefix-cache, agentic-coding, swe-bench, calibration]
---

# How Do AI Agents Spend Your Money?

## TL;DR

The first systematic measurement of where agentic token spend actually
goes — 8 frontier LLMs × 4 runs × 500 SWE-bench-Verified instances under
OpenHands — plus a formalized *pre-execution self-prediction* task that
asks the executing agent to forecast its own token cost before starting.
Both halves land on budget policy: spend is input-dominated,
heavy-tailed, weakly coupled to success, and model-idiosyncratic; and
agents cannot forecast it (Pearson r ≤ 0.39, systematically biased
**low**) even with privileged access to the repository and tools they
are about to use.

## Claims

- **Agentic tasks are a different cost regime, driven by input.**
  Agentic coding averages 4.17M tokens and $1.857 per task against
  3.39k / $0.023 for multi-turn code chat and 1.19k / $0.016 for
  single-turn code reasoning — ~1200× and ~3500× respectively. The
  input/output token ratio is 153.85 (vs 1.33 and 0.16). The gap is
  driven by input growth, not generation, and holds *with caching
  enabled*.
- **Spend is heavy-tailed within and across tasks.** Across 500
  problems the most expensive instance costs ~7M more tokens than the
  cheapest; on the *same* problem the most expensive of four runs
  roughly doubles the cheapest (per-model max/min ≈ 2×, up to 30× in
  total tokens on some tasks). High-cost problems also show the largest
  cross-run variance — behavior destabilizes on hard tasks.
- **More tokens do not buy accuracy.** Problems consuming more input
  tokens have lower accuracy (consistent across models); on the same
  problem, accuracy rises modestly from MinCost to LowerCost and then
  saturates, i.e. an inverse test-time-scaling regime. The behavioral
  explanation is measured, not assumed: repeated file *view* and
  *modify* actions on the same file rise sharply with cost quartile —
  expensive failures are redundant re-reading and re-editing that
  inflate context without progress.
- **Token efficiency is a property of the model, not the task.**
  Relative model ranking persists on both the shared-success (n=230)
  and shared-failure (n=100) subsets, so the gap is not "stronger
  models attempt harder problems." Kimi-K2 and Claude Sonnet 4.5
  consume >1.5M more tokens than GPT-5 on the same tasks. The
  success→failure cost increase is also model-specific (GPT-5/5.2
  <0.5M; Kimi-K2 ~2M), which the authors read as models lacking a
  reliable mechanism to recognize an unsolvable task and stop.
- **Human difficulty ratings are a weak proxy for agent spend.**
  Kendall τ_b = 0.32 (95% CI 0.25–0.38) between expert-estimated
  resolution time and actual token consumption; 6.7% of "<15 min"
  tasks cost more than the average ">1 hr" task and 11.1% of ">1 hr"
  tasks cost less than the average "<15 min" one.
- **Cache reads dominate both volume and dollar cost, in every
  phase.** Output tokens are priced ~80× higher per token than cache
  reads, yet accumulated cache-read volume outweighs output cost in
  aggregate at every stage of the trajectory. Per-round cost is
  non-monotonic: the accumulated cost of *reusing* context is steady
  and predictable, and the spikes come from what the agent chooses to
  *add* to context that round (repository exploration, file creation,
  test execution, final summarization).
- **Agents cannot predict their own spend.** Self-prediction (the same
  agent, full tool access, instructed to inspect and estimate rather
  than fix) reaches Pearson r ≤ 0.39 (output tokens, Sonnet 4.5); input
  tokens are harder for every model except Kimi-K2 (0.38); Gemini-3-Pro
  trails badly. Every model **systematically underestimates**, with the
  bias most pronounced on input tokens, and Appendix D reports the
  pattern persists without an in-context example.
- **Prediction has non-trivial overhead of its own,** and overhead
  doesn't buy accuracy: Sonnet 3.7 and Sonnet 4 spend >2× the task cost
  on prediction without achieving the best correlation, while GPT-5.2
  gets moderate correlation at <6% overhead.

## Methods

- OpenHands agent framework on SWE-bench-Verified (500 real GitHub
  issues with repos and tests); 4 independent runs per instance across
  Claude Sonnet 3.7 / 4 / 4.5, GPT-5, GPT-5.2, Qwen3-Coder-480B-A35B-
  Instruct, Kimi-K2, Gemini-3-Pro-Preview. Metrics parsed from the
  agent's structured JSON completion history: per-type token cost,
  monetary cost, action types, file-access patterns.
- Cost decomposed into the four separately-priced Anthropic categories
  (non-cached input, output, cache creation, cache read) at the 5-minute
  cache-write rate, analyzed at two granularities: **phase** (Setup
  9.98% / Explore 30.37% / Fix 33.53% / Validate 16.59% / Closeout
  9.53% of rounds) and **round** (a single traced trajectory).
- Self-prediction task: the executing agent keeps full tool-calling,
  may inspect the repo and run preliminary commands, and must output a
  stage-decomposed estimate of input tokens, output tokens, and total
  cost. Three predictions per model per instance; scored by Pearson
  correlation against actuals, plus a prediction-cost/task-cost
  overhead ratio.

## Results

See Claims — the load-bearing numbers are 153.85 input/output ratio,
τ_b = 0.32 for human difficulty, r ≤ 0.39 for self-prediction with
uniform downward bias, and cache-read dominance of dollar cost in all
five phases.

## Critique / open questions

- **Coding agents, not research agents.** SWE-bench-Verified trajectories
  are the measured object; whether the phase mix (Explore+Fix ≈ 64% of
  rounds) transfers to an ML-research loop with training runs in it is
  untested. The *direction* of the self-prediction bias should transfer
  more readily than its magnitude.
- Four runs per instance is thin for variance claims, though the
  variance findings are consistent across eight models.
- Self-prediction is measured only as *self*-prediction. The paper does
  not compare against a trained external predictor, which is exactly the
  comparison [[literature/papers/besanson2026green]] supplies from the
  other side — so the pair, not either paper, is what supports the
  "external and calibrated, not self-reported" conclusion.
- Prediction was run once per model at a fixed prompt; the authors
  concede better prediction may be reachable "with a reasonable amount
  of compute," so the ceiling on self-estimation is not established —
  only its current level.
- Prices move. The dollar figures are a snapshot; the token ratios are
  the durable part.

## Trust signals

- **Credibility:** 4 — strong multi-institution authorship
  (U. Michigan, Stanford, All Hands AI — who build OpenHands, Google
  DeepMind, Microsoft AI, MIT) with senior co-authors, a released
  project site with code and all agent trajectories, and a large
  measured trajectory corpus rather than a design argument. Not yet
  peer-reviewed, so held at 4 rather than 5.

## Follow-up

- **Relevance:** 5 — seeds [[concepts/spend-forecast-calibration]], the
  spend-policy concept the 2026-08-04→08-17 `/promote-moc` declines have
  been waiting on, and supplies the measured negative result that
  settles *whose* estimate a pre-flight gate may use. Also the strongest
  single-source evidence for two existing claims:
  [[concepts/hybrid-model-backends]] (token efficiency is inherent to
  the model and persists across success/failure subsets) and
  [[concepts/context-eviction-policy]] (cache reads dominate dollar
  cost, and redundant re-reads are the measured signature of expensive
  failure).
- The cache-read-dominance result is the missing price tag on
  [[literature/papers/hao2026selfgc]]'s commit rule: if cache reads are
  the dominant cost line, the `L_cache_break` term is not a rounding
  error, and an eviction policy should be evaluated on billed cost, not
  on token surface.
- "Models lack a reliable mechanism to recognize an unsolvable task and
  stop" is the same failure `max_consecutive_no_improvement` exists to
  patch from outside — and it is model-specific, which argues the
  ceiling should be tuned per backend rather than globally.
- Worth checking whether the released trajectories can be re-analyzed
  for a phase mix on a research-agent harness; that would make the
  Explore/Fix split importable rather than indicative.
