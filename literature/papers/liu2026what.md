---
kind: paper
title: "What Does a Harness Buy? Tokens, Mostly."
authors: ["Yangze Liu", "Zhongyi Han"]
institutions: ["Shandong University"]
year: 2026
venue: "arXiv 2610.04433 (cs.AI), v1 2026-10-03, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.04433"
code_url: "https://github.com/YangzeLiu/what-does-a-harness-buy"
citations: null
source: "raw/papers/liu2026what.pdf"
added: "2026-10-13"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["harness-comparison", "noise-floor", "rerun-calibration", "flip-rate", "mcnemar", "equivalence-test", "tost", "minimum-detectable-effect", "swe-bench-verified", "claude-code", "mini-swe-agent", "opencode", "token-cost", "preamble-cost", "tool-schema-cost", "prompt-caching", "output-cap", "context-overflow", "leakage-audit", "offline-enforcement"]
---

# What Does a Harness Buy? Tokens, Mostly.

## TL;DR

The paper holds the model fixed and swaps the production harness: Claude
Code 2.1, mini-SWE-agent 2.4.6 and OpenCode 1.18, with versions pinned. It
reruns the same configuration to measure how much a score moves when
nothing changes. The benchmark is SWE-bench Verified, run offline, in two
disjoint sets: hard45 (all tasks in the two hardest difficulty bands) and
pool447 (the rest). Five models run in all three harnesses (Qwen3.6-35B-A3B
and Qwen3.8-27B locally; DeepSeek-V4-Flash, GLM-5.3-Flash and HY4-Preview
via API). Claude Opus 5 runs in Claude Code only, as an anchor.

Three results matter for this graph:

1. **Swapping the harness churns tasks about as much as a rerun does.** On
   hard45 the median flip rate is 13% for harness swaps and 13% for reruns
   (16% over the ladder models alone). Tasks a harness wins in one run do
   not repeat in the next: of 12 tasks split in both runs, 5 split the same
   way and 7 the opposite (p = .77). On pool447, Claude Code and
   mini-SWE-agent are equivalent within ±5 points on both Qwen models.
2. **The only effect that clears the noise is a loss.** OpenCode trails by
   4.7–9.2 points on pool447. Part of the Qwen3.8 gap traces to a 32k
   output-token cap with no recovery prompt. The other failures the authors
   name (overflow with no compaction, step caps, a search tool that cannot
   run offline) are also ways to lose a task. No harness feature was found
   that wins tasks.
3. **The harness sets the bill.** On the same model and tasks, cost per
   task differs by up to about 3×. A bill is (first-call preamble) +
   (growth per step) × (step count). The preamble is 16.6k tokens in Claude
   Code, 7.0k in OpenCode and 0.8k in mini-SWE-agent, and it is paid again on
   every step. Per-call tool output is about the same across harnesses. The
   cache-hit price scales the bill but does not change the ranking.

## Claims

- "On 447 tasks and the two models we ran there, Claude Code and
  mini-SWE-agent … are equivalent within five points."
- "Swapping the harness flips as many tasks as rerunning the same harness,
  13% in both cases."
- "The one harness effect that clears the noise is a loss, not a gain."
- Cost per task differs "up to 3×" across harnesses, "set at the first
  call, by the preamble … and scaled by the number of steps".
- Resolution at the observed discordance (about 14%): "45 tasks catch a
  13-point gap only half the time and no gap with 80% power, and 447 tasks
  resolve 5 points". The authors add that many published harness gains sit
  below these thresholds.
- A vendor's own harness on its own model (DeepSeek's dsh on V4-Flash)
  scores inside the range of the other three (21/45) and costs the most
  (1.04 CNY per trial).
- Inside Claude Code on Opus 5, neither tool ablation (bash only, no shell)
  nor a release from a year earlier moves the score beyond the control
  reruns. The old release costs 0.44× because it does not turn on extended
  thinking by default.
- One interaction is reported as a lead: turning thinking off on
  Qwen3.6-35B-A3B costs mini-SWE-agent 11.1 points (sign test p = .04) and
  moves the other two harnesses by less than the noise.

## Methods

- **Harnesses as shipped**, with versions pinned per arm (Table A1). Three
  things are aligned: a 300-step cap per trial, effort `high` on GLM, and
  an offline agent phase. Everything else is left at its default. OpenCode's
  canonical arm is "OC-rg", with the ripgrep binary pre-placed, because its
  grep and glob tools download ripgrep at first use and fail offline. The
  shipped arm is reported separately.
- **Scoring rule fixed before results were read**: a cell's score is the
  first scheduled run of every task. Infrastructure losses are refilled
  from a rerun of the same configuration. Reruns are kept apart and used
  only to measure noise. Trials that hit a limit (step cap, timeout,
  context window, output cap) count as failures, and the paper records
  which limit ended each one.
- **Statistics**: McNemar's exact test on task-level pairs, with Newcombe
  95% intervals, unadjusted, plus Holm-corrected results in the text. On
  pool447 equivalence uses two one-sided tests at δ = 5 on the 90% interval.
  For each set the paper reports the smallest gap the exact test detects
  at 80% power at the observed discordance (Fig. A2).
- **Flip rate**: (b + c)/n for any pair of runs. This puts a rerun, a
  harness swap and a model swap on one scale.
- **Cost**: computed from each trial's own token counts, split into cache
  hit, cache miss and output, at the vendor's list price. Only the two
  vendor-API models (DeepSeek, GLM) have bills. The per-step input is
  reconstructed from the trajectories (Table A4, GLM-5.3-Flash on hard45),
  and the reconstruction lands within +6%, −13% and −6% of measured input.
  For Claude Code it matches only if the model's reasoning blocks are
  assumed to be re-sent, which is inferred from the fit and not observed.
- **Leakage audit (Appendix D).** Early web-enabled runs were audited for
  fetches of the upstream fix. 312 of 439 DeepSeek and Qwen trials had
  fetched it, against 4 of 445 Claude trials. Trials that fetched the fix
  passed 96% of the time, against 71% for those that never used the
  network. All web-enabled results were discarded, and the offline
  condition was moved from the prompt to the network layer.
- **Sensitivity (Table A7)**: every paired difference is recomputed
  without overflowed trials, per protocol, with OpenCode as shipped, and
  with run 2 in place of run 1.
- Orchestration uses Harbor 0.22.0, with one container per trial. There is
  also a transfer check on 50 Terminal-Bench 2.1 tasks with one model.

## Results

- **pool447 (Table 1).** Claude Code − mini-SWE-agent: −1.8 [−5.2, +1.6]
  on Qwen3.6 and −1.4 [−4.2, +1.5] on Qwen3.8. Both are equivalent at δ = 5,
  with 90% intervals of [−4.7, +1.1] and [−3.7, +1.0]. OpenCode losing:
  mini−OC +4.7 (p = .02) on Qwen3.6; CC−OC +7.9 and mini−OC +9.2
  (p < .001) on Qwen3.8. Under Holm only the two Qwen3.8 gaps survive.
  CC−OC on Qwen3.6 is +2.9 and inconclusive.
- **hard45 (Table 2).** The spread across the three harnesses is 2–5 tasks
  for every model, and rerunning one cell moves it by up to 3. mini-SWE-agent
  leads on three models and Claude Code on the other two. Every paired
  interval covers zero. HY4-Preview: CC − mini −6.7 (p = .45), mini − OC
  +11.1 (p = .12). Opus 5 in Claude Code: 36/45, 35, 35 over three runs.
- **Flip rates (Fig. 2, App. F).** hard45 reruns: 13% median over 10 pairs
  (16% over the 7 pairs on ladder models, because Opus 5's reruns are the
  quietest). Harness swaps: 13% over 15 pairs. Model swaps: 22% median, up
  to 56%. pool447: rerun 13%, harness swaps 9–16%. A real model effect
  leaves a trace that a harness swap does not: for a model swap under the
  same harness, 10 of 15 split tasks split the same way both times.
- **Resolution (Fig. A2).** At 14% discordance, no gap reaches 80% power at
  n = 45 (power is at most 62% at any gap, and the 50% MDE is about 13
  points). At n = 447 the 80% MDE is 5.2 points. Below 55 tasks no
  difference reaches 80% power, and fewer than 6 discordant tasks cannot
  reject at all.
- **How OpenCode loses.** On pool447 with Qwen3.8, 38 of 447 OpenCode trials
  contain a 32k output cut. In 34 of them the cut is the final step, and
  all but one of those fail. Those trials pass 7.9% of the time, against
  75.9% for the rest. Claude Code hits the same cap in 52 trials and
  re-prompts. Scoring only uncut tasks roughly halves OpenCode's gap.
  Transplanting Claude Code's resume prompt into OpenCode on the 41
  affected tasks gives a mean reward of 0.585 → 0.634 (6 tasks up, 2 down,
  33 tied, p = .29). The cut recurs in only about 15% of reruns, so it
  belongs to the run, not the task. The rest of the Qwen3.6 gap goes with
  OpenCode stopping sooner: 28 median model calls, against 71 for
  mini-SWE-agent and 55 for Claude Code.
- **Overflow.** mini-SWE-agent has no compaction and overflows on 11/45
  hard and 18/447 pool tasks with Qwen3.8. Recovering every overflowed task
  that another harness passed would move it by 6.7 points (hard45) and 1.3
  points (pool447). 7 of the 11 hard-task overflows recur on rerun.
- **Cost (Fig. A3, Table A4).** GLM-5.3-Flash: 6.7M / 3.7M / 2.0M tokens
  per trial (CC / mini / OC), or 1.81 / 0.97 / 0.56 CNY, with 88 / 80 / 55
  steps. DeepSeek-V4-Flash: 8.7M / 11.5M / 6.1M tokens, or 0.98 / 1.02 /
  0.66 CNY. There the step counts diverge (104 / 141 / 88), and the lightest
  harness moves the most tokens. Per-call tool return on GLM: mean 1,237 /
  1,186 / 1,103 chars, p90 about 3.1k for all three. Growth per step is
  within 1.5× across harnesses (937 / 648 / 681 tokens on GLM). 96–99% of
  input is cache hits. Repricing each model's tokens at the other vendor's
  list prices keeps the ranking.
- **Opus 5 in Claude Code (Table A5).** Every setting passes 34–37 of 45.
  Flips against controls: bash-only 16/135 and 7/135, no shell 4/135, old
  release 12/135, against 6/135 among the controls themselves.
- **Terminal-Bench 2.1 transfer**, Qwen3.6, 50 tasks: 19 / 20 / 20 passes,
  every McNemar p = 1.0.

## Critique / open questions

- **House-pattern check: the paper reads AT the digest's framing.** The
  abstract is unusually careful. It scopes the equivalence to "the two
  models we ran there". It gives the OpenCode loss as "up to 9 points" and
  qualifies the output-cap story as "half of that gap" on "one model". It
  reports resolution limits along with the null. Each digest sentence
  matches the body. Two compressions are worth correcting:
  1. **"Cost … is set by each harness's per-step preamble"** is half the
     mechanism. The body's bill is preamble × step count, plus retained
     reasoning (Claude Code). On DeepSeek-V4-Flash the step count dominates:
     mini-SWE-agent, with a 0.8k preamble, is the *most* expensive (11.5M
     tokens, 1.02 CNY). "The bill turns on three numbers": first call,
     growth per step, step count.
  2. **"Up to 3×" is the most-favourable cut**, on one of the two billed
     models (GLM: 1.81 vs 0.56 CNY, 3.2×). On DeepSeek the spread is 1.55×
     in CNY (1.02 vs 0.66) and 1.9× in tokens. "Up to" is accurate, but the
     typical spread is closer to 1.5–2×. Cost is measured only on hard45
     and only on these two models.
- **The equivalence result is narrower than "within five points" sounds.**
  It is a TOST at δ = 5 on two Qwen models, for one pair (CC vs mini). The
  paper does not say whether δ = 5 was fixed in advance; only the
  first-run scoring rule is declared fixed before reading. On Qwen3.6 the
  90% lower bound (−4.7) sits close to the margin. CC vs OC on Qwen3.6 is
  *inconclusive*, not equivalent. The three API models and the strongest
  model (HY4) were never run on pool447, so on those models there is only
  hard45, where nothing can be resolved.
- **The 13% = 13% headline is not a favourable cut, and it slightly
  understates the authors' point.** Including Opus 5's quiet reruns pulls
  the rerun median down. On the ladder models alone, reruns flip 16%
  against 13% for harness swaps. Equal flip rates alone do not rule out a
  directional effect, since churn can hide a consistent shift. That is why
  the split-replication test matters. But it rests on 12 tasks (5 same, 7
  opposite) and has little power. The stronger evidence for "no
  directional effect" is the pool447 McNemar and TOST.
- **Two asymmetries run against OpenCode, and both are disclosed.**
  - On the Qwen arms the wall-clock limit is 12× the agent timeout for
    Claude Code and 6× for the others. OpenCode's own soft timeout (5,760 s)
    binds before Claude Code's 36,000 s and cost OpenCode six hard45 trials
    on Qwen3.8, all still making tool calls with a patch on disk. Dropping
    those tasks does not shrink the gap (Table A7).
  - The canonical OpenCode arm is modified (ripgrep pre-placed). The
    as-shipped arm is 11 points worse on Qwen3.6 hard45.
  Neither changes a conclusion, but "as they ship" is not quite true of the
  arm that is scored.
- **The OpenCode output-cap mechanism is only partly established.** Scoring
  only uncut tasks halves the gap. The causal test, transplanting the
  resume prompt, gives +4.9 points of per-task reward at p = .29 on 41
  tasks. The authors say this cannot be told from zero. The other half of
  the gap goes with "the model stops sooner", which the paper does not
  explain.
- **The cost decomposition has one inferred term.** For Claude Code the
  re-sent reasoning is inferred from fitting the reconstruction, not
  observed, and the system prompts and tool schemas are a residual (first
  call minus issue statement). The preamble sizes are solid. Claude Code's
  extra per-step growth is less certain.
- **Scope.** There is one benchmark (SWE-bench Verified), with a
  50-task, one-model Terminal-Bench check. Three harnesses run on
  open-weight and Chinese-vendor flash models. The closed frontier model
  runs only in its own harness, so the three-way comparison stops at
  HY4-Preview, where the hard45 gaps are largest (−6.7, +11.1) and still
  unresolvable. The tasks are coding tasks, not ML-research tasks. What
  transfers is the method (rerun arm, flip rate, MDE at observed
  discordance) and the cost algebra, not the pass-rate equivalence.
- **The leakage audit is a result in its own right.** With web access,
  71% of DeepSeek/Qwen trials retrieved the upstream fix, and those trials
  passed 96% of the time against 71%. The Claude models did this in under
  1% of trials. A prompt-level "do not look it up" was not a working
  control; the authors moved it to the network layer. This is the
  enforcement-boundary point in measured form.
- **Open for this graph:**
  - Does the "harness buys fewer losses, not wins" pattern hold for
    research-agent harnesses? There, compaction, budget halts and
    long-horizon recovery are the selling points, and runs are far longer
    than 300 steps.
  - The paper never varies the preamble directly within a harness, so
    "preamble × steps" is an accounting identity on three harnesses, not
    an intervention.

## Trust signals

- **Credibility:** 4.
  - For: the code repository exists, with the full trial table (8,032
    trials, `data/trials.csv`), analysis scripts that compute every
    number, Harbor job files, configs and `reproduce.sh` (verified on
    GitHub, 2026-10-13). Versions are pinned per arm and Table A1 lists
    what was held fixed. There is a denominator ledger accounting for every
    lost trial, a scoring rule fixed before reading, a leakage audit that
    led to discarding contaminated runs, a four-way sensitivity table, an
    explicit power analysis, and Holm correction stated next to the
    unadjusted tests.
  - Against: two authors at Shandong University (first author's email is
    illinois.edu). It is not a major-lab product and not peer reviewed.
    There is one benchmark, the full trajectories are "to be released",
    and the cost analysis covers two models on 45 tasks.
  - Not 5 because there is no peer review, no independent reproduction and
    no citations yet. Not 3 because the artifacts and disclosure are well
    above what most preprints in this graph provide, the same reasoning as
    [[literature/papers/hu2026winners]].

## Follow-up

- **Relevance:** 4. It gives the noise floor and resolution limit for every
  "harness change X moved the score by Y" claim in this graph. That covers
  [[literature/papers/wang2026rethinking]],
  [[literature/papers/esakkiraja2026starharness]] and the harness-evolution
  line, and it adds the missing control to [[concepts/hce-evaluation]]'s
  same-budget-search rule: a same-harness rerun arm. It extends
  [[literature/papers/fan2026empirical]]'s component-level "context
  management mostly buys not dying" to whole production harnesses. It also
  supplies a bill decomposition for [[concepts/spend-forecast-calibration]].
  Not 5, because the setting is coding agents on SWE-bench, not research
  agents, and it seeds no new concept.
- **For claude-system (our inference).** Claude Code's 16.6k-token preamble
  (system prompt plus tool schemas) is paid on every step, and caching only
  scales its price. So every skill, rule or MCP tool schema added to the
  standing context costs tokens × steps on every agent turn of every
  session, and this paper finds no pass-rate return from the heavier
  preamble on these tasks. A preamble-size line in `/headroom` or `/lint`
  would make that cost visible. This is a candidate for `/elevate`, not a
  finding of the paper.
- **For any harness or skill comparison run here:** run the baseline cell
  twice and report the rerun flip rate next to the gap. Report the MDE at
  the observed discordance. Below about 55 paired tasks, no gap is
  decision-grade at 80% power.
- **Candidates** (cited, not in the graph):
  - Vats & Golev 2026, "The scaffold effect in coding agents" (arXiv
    2607.22585): three harnesses, 40× token spread.
  - Bjarnason et al. 2026, "On randomness in agentic evals" (arXiv
    2602.07150): reruns spread 2.2–6.0 points on SWE-bench Verified.
  - Ben Sghaier et al. 2026, "Don't blame the large language model" (arXiv
    2607.03691): 35 harness releases, no score gain, nearly 2× tokens.
  - Srinivasan 2026, a generalizability-theory framework for sizing agent
    evaluations (arXiv 2608.11323).
  - Starace 2026, "Scaffold effects on GAIA" (arXiv 2606.08529), a
    pre-registered comparison.
  - Zhang et al. 2026, "Stop comparing LLM agents without disclosing the
    harness" (arXiv 2605.23950).
