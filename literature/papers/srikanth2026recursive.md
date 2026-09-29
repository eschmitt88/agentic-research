---
kind: paper
title: "Recursive self-improvement of AI research agents"
authors: ["Dhruv Srikanth", "Bingchen Zhao", "Dixing Xu", "Yuxiang Wu", "Zhengyao Jiang"]
institutions: ["Weco AI"]
year: 2026
venue: "arXiv (cs.AI / cs.LG / cs.SE) preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.26457"
code_url: null
citations: null
source: "raw/papers/srikanth2026recursive.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/compression-as-generalization-test]]"
tags: ["recursive-self-improvement", "harness-evolution", "bi-level-optimization", "public-private-split", "outer-loop-selection", "aide", "reward-hacking", "context-compaction", "bandit-search", "noise-floor", "ignition-test", "weco"]
---

# Recursive self-improvement of AI research agents

## TL;DR

AIDE² is a bi-level loop. An outer-loop agent (AIDE_human, Weco's production
research agent, on Claude Opus 4.7) rewrites the code of an inner-loop
research agent (starting from AIDE₀, a pared-down AIDE refactor, on Gemini 3
Flash). Each rewrite is graded by running the inner agent under a fixed
dollar budget on a selection benchmark of ML-, heuristic-algorithm- and
harness-engineering tasks. The inner agent optimizes against a **public**
score; the outer loop selects by `argmax` on the mean **private** held-out
score `g(a)`. One 8-day, 100-node run accepted **seven** rewrites (steps 2,
6, 28, 39, 47, 63, 85), raising the incumbent grade **0.703 → 0.778**
(AIDE_human: 0.749 on the same grade). Two checkpoints (AIDE₄₇, AIDE₈₅) beat
AIDE₀ on four external benchmarks. AIDE₈₅ matches or exceeds AIDE_human on
all four, though two of those four are within about one SE. On 38 KernelBench
(kernel, training-context) pairs the reward-hacking rate falls **55% → 39% →
32%** along the lineage (AIDE_human 39%). The "ignition test", in which the
discovered agent runs the outer loop, is inconclusive by the authors' own
account (0.780 vs 0.782, 3 seeds).

## Claims

- **Sustained improvement, not a one-off.** "seven accepted improvements at
  steps 2, 6, 28, 39, 47, 63, and 85, with the incumbent grade rising from
  0.703 to 0.778." "Two further complete runs of the same protocol also
  produced sustained improvements, accepting two and four rewrites." The
  authors themselves say the trace "is not meant to demonstrate
  generalization beyond the selection benchmark."
- **Signal separation is the anti-hacking design.** "separating the inner
  loop's optimization signal r_pub from the outer loop's selection signal g
  … prevents the inner-loop agent from directly optimizing the criterion
  used for outer-loop selection." A fixed per-task budget `b_t` "constrains
  improvements to arise from a better algorithm rather than from spending
  additional compute."
- **Second-order generalization.** "AIDE₈₅ matches or exceeds AIDE_human on
  all four external benchmarks." The gains are "positive throughout but not
  monotone across checkpoints": AIDE₄₇ is best on MLE-Bench and WeatherBench
  2, AIDE₈₅ on ALE-Bench and FML-Bench.
- **Emergent reduction in reward hacking.** "55% for AIDE₀, 39% for AIDE₄₇,
  and 32% for AIDE₈₅, compared to 39% for AIDE_human." The paper is careful
  about cause: the rates "establish a held-out behavioral change, though they
  do not identify which rewrites produced it."
- **What the loop discovered** (AIDE₈₅, §3.5):
  - *Search:* UCB1 over five fixed drafting-strategy arms, with 30% softmax
    sampling. Every five steps it forks the global best under a different arm.
  - *Context:* draft/improve prompts read "a compact summary of the root and
    recent candidates" rather than the full history. A failure memory injects
    up to three deduplicated error signatures when the bug rate is ≥ 15%. The
    median per-call prompt reduction reaches "7× on MLE-Bench, over 40× on
    WeatherBench 2, and about 50× on ALE-Bench and FML-Bench."
  - *Robustness:* a fixed prompt line saying solutions are scored on a
    private split, and a re-prompt when generated code is under 40
    characters. A median-distance selection penalty "never changed which
    candidate the agent selected". There is also a patch to one task's
    held-out scoring script, which "crashed on all of its test cases whenever
    any single test case failed." The authors read that last one as "rather
    than exploiting the broken evaluation, AIDE² repaired it."
- **Harness gains transfer across models** (App. C). On ALE-Bench at $20,
  "gemini 3 flash with AIDE₈₅ reaches 1858 ± 22 and exceeds fable 5 with
  AIDE₀ at 1796 ± 24." On MLE-Bench, "AIDE₈₅ with fable 5 … stays within one
  standard error of its AIDE₀ score."
- **Noise is the stated limit.** "If the noise is high enough, a falsely
  accepted rewrite becomes the new incumbent (eq. (4)), so a single noisy
  comparison can derail the outer loop's subsequent search."

## Methods

- **Algorithm 1.** `g(a) = (1/T) Σ_t r_priv(x̂_t)`, where each task is "run
  several times independently and the scores are averaged." The acceptance
  rule is a bare `a*_k = argmax_{a ∈ a≤k} g(a)`, with no tolerance term. The
  outer-loop agent reads all previous agents *and their grades* (eq. 3). So
  the private score is hidden from the inner agent and is the outer loop's
  optimization signal.
- **Models.** The outer loop runs on Claude Opus 4.7, the inner loop on Gemini
  3 Flash during selection. The held-out runs use Gemini 3 Flash (ALE, MLE,
  KernelBench), Gemini 3.1 Pro (WeatherBench 2) and GPT-5.4 (FML-Bench).
- **Held-out protocol** (Table 2). ALE-Bench lite: 10 tasks × 10 seeds, $5.
  MLE-Bench lite: 22 × 3, $5. WeatherBench 2: 1 task × 3 seeds, $15.
  FML-Bench: 18 × 3, 100 steps. KernelBench: 38 pairs × 3, 20 steps. A kernel
  "counts as reward hacking when its isolated speedup exceeds 1.02× and
  either less than half of that speedup survives inside training or the
  kernel crashes at runtime there."
- **Ignition test.** Two arms, 3 seeds × 50 steps each, both starting from
  AIDE₄₇. The outer loop is AIDE₄₇ in one arm and AIDE_human in the other.
- **Rejected proposals** (App. D, pooled over three runs). "about a quarter
  [of graded rejected rewrites] scored higher than the incumbent on the
  agent-visible public signal and were rejected on the private grade."
  Rejected deltas of "−0.004 to −0.007 are small relative to observed
  run-to-run variability." Ensembling "was explored … but never retained".

## Results

| Agent | ALE-Bench | MLE-Bench | WeatherBench 2 | FML-Bench (%) |
|---|---|---|---|---|
| AIDE₀ | 1536 ± 33 | 0.678 ± 0.006 | 0.262 ± 0.205 | 15.0 ± 0.9 |
| AIDE₄₇ | 1713 ± 26 | 0.730 ± 0.005 | 0.798 ± 0.003 | 19.7 ± 1.2 |
| AIDE₈₅ | 1790 ± 9 | 0.722 ± 0.011 | 0.793 ± 0.005 | 19.9 ± 1.1 |
| AIDE_human | 1511 ± 35 | 0.708 ± 0.007 | 0.404 ± 0.193 | 19.6 ± 1.0 |

- **AIDE₀ failed to stay alive.** Its prompts grow with history, and it hit
  the context window on "five of its FML-Bench runs … and 48 of its
  ALE-Bench runs at the larger per-run budget." The other agents did not.
  Against AIDE_human, the discovered agents' prompts are 2.6–5.7× smaller
  (13–14× on WeatherBench 2).
- **Ignition test.** "Averaged across seeds, the two arms reach similar mean
  endpoints … 0.780 and 0.782 for AIDE₄₇ and AIDE_human respectively." The
  treatment reached its plateau in about 20 steps against about 40, but the
  authors call the result "inconclusive".
- **Selected rejected proposals** (Table 3). Island GA migration −0.021.
  Pairwise LLM-judge tournaments −0.090. Majority-vote ensembles up to
  −0.031. "promote-second-best and robust near-tie overrides" (optimizer's
  curse) best −0.007, "within noise."

## Critique / open questions

- **The "hidden evaluation" is the outer loop's validation split, not a
  holdout.** `g` is read on every one of 100 steps and selected by `argmax`.
  In this graph's terms (see [[concepts/hce-evaluation]]) it is spent, and the
  0.703 → 0.778 trajectory is a max over about 100 noisy draws. The
  selection-benchmark size `T` and the per-task repeat count are not stated.
  The actual test split is the four external benchmarks, and there the claim
  rests on **one run's two checkpoints**. The two replicate runs were never
  taken to held-out evaluation.
- **"Matches or exceeds a production agent" is mostly "matches."** FML-Bench
  is 19.9 ± 1.1 vs 19.6 ± 1.0 and MLE-Bench is 0.722 ± 0.011 vs 0.708 ±
  0.007. On ALE-Bench the clear margin (1790 vs 1511) is over a baseline that
  scores *below* the unimproved AIDE₀ (1536). WeatherBench 2, billed as the
  out-of-distribution highlight, is **one task, 3 seeds**, with both
  baselines at ±0.19–0.21 SE. It shows a variance collapse (every evolved
  seed converges on the same numerics change) more than a mean effect. The
  paper's own App. B says that on FML-Bench "margins separating the leading
  agents are small relative to the seed-level standard error." AIDE_human is
  also the outer-loop agent, and both come from the same lab.
- **Part of the AIDE₀ gap is AIDE₀ being fragile.** A pared-down starting
  agent with no compaction and context-overflow crashes gives the loop easy
  wins that AIDE_human already had. How much of the lift over AIDE₀ is
  "discovery" and how much is "re-adding bounded history" is not ablated.
  None of the seven rewrites is ablated individually.
- **The reward-hacking result is narrower than "never optimized for."** 32%
  vs 39% on 38 pairs is roughly 12 vs 15 pairs, with no interval reported.
  The construct is also close to what *was* optimized. The kernel metric
  asks whether the proxy speedup survives in the real training loop. The
  outer loop selected on whether public-score gains survive on private data.
  Those are the same proxy-to-held-out gap (cf. [[literature/papers/zhao2026specbench]],
  same group) measured on another family. One discovered mechanism tells the
  model it is "scored on a private split it cannot see". It is a plausible
  direct cause, and the paper does not isolate it.
- **An accepted rewrite reached the grader.** §3.5 says AIDE₈₅ carries a
  patch to one task's *held-out scoring script*. Fig. 2 labels accepted step
  47 "patches an error that crashed evaluations", which is probably the same
  rewrite, though the paper never ties the two together explicitly. So agent
  code the loop can rewrite could change how the private grade was computed,
  and an accepted rewrite did so. The paper calls it a benign repair, and it
  may well be. For evaluation hygiene, a selection signal the candidate can
  modify is exactly the channel HCE exists to close, whatever the intent
  (see [[concepts/programmable-evaluator-oracle]]). How much of that
  rewrite's grade gain came from the scorer is not reported.
- **The acceptance rule has no noise band, and the loop cannot resolve one.**
  The authors state the derail risk and quote a run-to-run band of at least
  about 0.007. The mean accepted step is about 0.011 (0.075 over seven
  accepts). Near-tie and optimizer's-curse proposals were rejected "within
  noise". The discovered robust-selection rule turned out to be a no-op. So
  the loop could neither evaluate noise-robustness changes nor protect
  itself from noise-driven accepts.
- **No code, no cost figure.** There is no repository ("© 2026 Weco AI. All
  rights reserved"), and the total dollar cost of the 8-day run is not
  reported. The ignition test is null, so the "recursive" part (a better
  agent improving the improver) is untested. What is shown is one level of
  harness search driven by a fixed human-built optimizer.
- **Rediscovery of known moves.** Stall-triggered forking of the best node
  is close to FML-Bench's AdaptiveSearch (greedy until stagnation, then fork
  broader exploration; [[literature/papers/zou2026fmlbench]], which has a
  Weco co-author). The paper deliberately leaves AdaptiveSearch out of its
  App. B comparison. UCB over strategy arms is BaSE-like allocation
  (xing2026compute). This convergence is mild evidence that these are the
  right moves, and it is evidence against novelty.

## Trust signals

- **Credibility:** 3. Weco AI built AIDE, which was used in MLE-Bench and
  co-authored FML-Bench, so the group is credible on exactly this agent
  family. The evaluation has good structure: fixed budgets, four external
  benchmarks with SEs, cross-model transfer, and candid reporting of the
  inconclusive ignition test and the noise risk. It is held at 3 by an
  industry preprint with no peer review and **no code or artifacts**. There
  is one run behind all transfer claims, and no per-rewrite ablation. The
  baseline is the authors' own product and is also the outer-loop driver.
  Two of the four headline "matches or exceeds" comparisons are within
  about one SE.

## Follow-up

- **Relevance:** 4. This is the most direct research-agent instance in the
  graph of an evolutionary loop pointed at the research agent's own harness,
  with held-out benchmark transfer. It sharpens [[concepts/hce-evaluation]]
  (nested separation: each level needs a holdout the level above does not
  select on) and gives [[concepts/evolutionary-expansion]] and
  [[concepts/budget-as-ceiling]] a recursive case where a false accept
  changes the lineage and not just the champion. It does not reach 5: the
  transfer evidence is one run, the recursive claim is null, and there is no
  code to import.
- **[[concepts/hce-evaluation]].** The public/private split is measured to do
  work: about 1/4 of rejected graded rewrites won on public and lost on
  private. It moves the holdout up one level rather than removing the need
  for one, and an accepted rewrite patched a held-out scorer.
- **[[concepts/evolutionary-expansion]].** Self-application. The loop
  converges on bandit allocation plus stall-forking and rejects ensembling
  because it eats search budget.
- **[[concepts/budget-as-ceiling]].** A fourth system with a bare `>` on a
  noisy grade. In a recursive loop a false accept rewrites the code that
  proposes the next candidates.
- **[[concepts/context-eviction-policy]].** Search re-derived anchor-plus-
  recency with a gated failure memory, and AIDE₀'s overflow deaths match the
  liveness reading. Unablated.
- **Digest item 11 (RRSI) comparison.** This paper does *not* show that
  hidden-eval selection is enough to avoid overfitting. It shows positive
  transfer from one run without the regularizers RRSI adds, on four
  benchmarks where the margins over a strong baseline are mostly within
  noise.
- **Candidates cited, not in graph:** SGM, a statistical Gödel machine for
  risk-controlled self-modification (arXiv 2510.10232), which is the
  noise-gated acceptance rule this paper lacks. Also the Darwin Gödel Machine
  (Zhang et al. 2026a), HyperAgents (arXiv 2603.19461) and the Red Queen
  Gödel machine (arXiv 2606.26294; co-evolving evaluators).
