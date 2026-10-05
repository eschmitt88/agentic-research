---
kind: paper
title: "Rethinking the Evaluation of Harness Evolution for Agents"
authors: ["Yike Wang", "Huaisheng Zhu", "Zhengyu Hu", "Yige Yuan", "Zhengyu Chen", "Shakti Senthil", "Hannaneh Hajishirzi", "Yulia Tsvetkov", "Noah A. Smith", "Pradeep Dasigi", "Teng Xiao"]
institutions: ["Allen Institute for AI", "University of Washington", "Independent"]
year: 2026
venue: "arXiv 2607.12227 (cs.AI), preprint; v3 dated 2026-10-01"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.12227"
code_url: "https://github.com/rethinking-harness-evolution"
citations: null
source: "raw/papers/wang2026rethinking.pdf"
added: "2026-10-05"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["harness-evolution", "matched-budget-baseline", "test-time-scaling", "parallel-sampling", "sequential-refinement", "held-out-tasks", "overfitting", "terminal-bench", "arc-agi-3", "edgebench", "ahe", "vista", "tool-writing", "negative-result"]
---

# Rethinking the Evaluation of Harness Evolution for Agents

## TL;DR

This paper is an evaluation-protocol critique of automatic harness evolution,
backed by experiments. It names two flaws in prior work (Meta-Harness, AHE,
AEVO). First, there is no comparison against parallel sampling or sequential
refinement **at a matched rollout budget**. Second, the harness is searched
using unit-test feedback from **the same tasks** the final score is reported
on. On Terminal-Bench 2.1 (89 tasks, 3 models, K = 5, mean of 2 runs), AHE
with its explore agent disabled comes in **below direct sampling on average
when no tests are available** (67.4 vs 68.2) and is last on both metrics when
tests are available. On a 45/10/34 train/val/test split its held-out gain is
**exactly 0.0** averaged over models. The positive result is narrower than
the abstract suggests. On ARC-AGI-3 and EdgeBench the method that wins is
**per-task "Harness Scaling"**, not the reusable cross-task evolution that
prior work claims. It is compared only against sequential refinement, and on
ARC-AGI-3 the winning arm is also the only one that can write and run code
tools over the frame. For this graph the main use is a rule:
**any evolution claim needs a same-budget test-time-search arm and a
task-disjoint test set**. Its own negative numbers sit inside an
unreported ~2.5-point run-to-run spread.

## Claims

- **The protocol is broken in two ways.** "prior work does not compare
  these approaches with simple task-level search baselines under matched
  feedback and inference budgets", and it "searches for harness
  configurations using verification signals (e.g., unit test cases) drawn
  from the same benchmarks on which it reports the final performance".
- **On Terminal-Bench, evolution loses to test-time scaling.** "automatic
  harness evolution fails to outperform simple test-time scaling methods
  both with and without test cases, and exhibits limited generalization."
- **The gains come from more attempts.** "the benefit only materializes when
  we can select among multiple trajectories … their gains largely stem from
  making multiple attempts, an effect that Parallel Sampling and Sequential
  Refinement achieve more directly and more effectively."
- **Evolved edits memorize instead of distilling.** "most edits memorize
  fixes rather than distilling strategies", while "the stable core of hard
  failures … remains unaffected", and "the growing volume of persistent
  prompt text introduces context bloat".
- **Self-feedback is too noisy to drive revision.** "without unit test
  cases, even strong agents struggle to extract reliable learning signals
  from their own trajectories … harness revision likely requires a reliable
  external correctness signal."
- **Where it does help.** Long-horizon games "are difficult enough to leave
  headroom, rely heavily on adaptation to out-of-distribution dynamics, and
  provide granular feedback by design". There, "task-specific harness
  evolution improves over the search baseline by 80.0% on ARC-AGI-3 and by
  10.9% on EdgeBench under matched budgets."

## Methods

- **Four methods under one budget K**, all starting from AHE's minimal
  harness (a single bash tool, no skills, middleware or memory):
  - **Parallel Sampling**: K independent rollouts. The pick is made by a
    self-judge without tests, or by any test-passing rollout with tests.
  - **Sequential Refinement**: K rollouts, each conditioned on a summary
    Φ of the previous one (plus its outcome when tests are available).
  - **Harness Evolution**: AHE (Lin et al. 2026) with its **explore agent
    disabled**, because that agent "retrieves harnesses tuned for the
    benchmark under evaluation from external sources". m = 1 rollout per
    task per harness, across all 89 tasks. Without tests the final harness
    is h_K. With tests it is argmax of the batch pass rate.
  - **Harness Scaling**: the authors' per-instance analogue. A meta agent
    rewrites the harness *for one task* after every rollout.
  - Φ is AHE's Agent Debugger, and the same model serves as policy,
    debugger and meta agent. **Harness-editing cost is excluded** from the
    budget, which the authors say favours evolution.
- **Terminal-Bench 2.1.** 89 tasks; Claude Opus 4.6, GPT-5.4 and GPT-5.4
  mini at high reasoning effort with 128k max generation; K = 5; **two
  independent runs averaged**, with no CIs. An ablation at K = 10 uses
  Claude only. Held-out setting: 45 train / 10 val / 34 test, evolve on
  train, select on val, report test pass@1.
- **ARC-AGI-3.** All 25 public games, scored by RHAE (per level
  `min(1.15, (a_human/a_agent)^2)`, level-weighted). The budget is **2,000
  environment actions**. "internal reasoning and read-only inspection are
  free", so agent-written tool calls do not count against it. Models are
  Claude Opus 4.8 (xhigh) and GPT-5.6 Terra. Arms:
  - Sequential Refinement on plain Claude Code or Codex CLI;
  - Sequential Refinement on VISTA* (VISTA with memory disabled);
  - Harness Scaling on VISTA (memory evolves: GUIDE.md / WORKING.md);
  - Harness Scaling on [full harness], which adds `write_tool`,
    `run_tool`, `write_skill` and `write_hooks`.

  Number of runs per game is not stated. EdgeBench says two, so this
  appears to be one.
- **EdgeBench, Interactive Games & Simulators** (5 bot-writing tasks: DCSS,
  NetHack, OpenRCT2, OpenTTD, Wesnoth). Claude Code (Opus 4.8) and Codex
  (GPT-5.4) serve as the initial harnesses. The budget is **12 h
  wall-clock** for both arms, and harness-update time counts against it.
  A meta-agent update fires every 3 submissions or 120 min. The score is
  the **best-so-far max over every judge evaluation**, including host-side
  snapshots the agent never sees. Two runs.

## Results

- **Terminal-Bench, no tests (Table 1, pass@1, average of 3 models):**
  direct 68.2; Parallel 72.3; Sequential 69.3; **Harness Evolution 67.4**;
  Harness Scaling 71.8. GPT-5.4 drops 75.3 → 69.7 under evolution. Harness
  Scaling has the best single cell (Claude 76.0).
- **Terminal-Bench, with tests (Table 2, pass@1 / pass@5 averages):**
  direct 67.6; Parallel 82.0 / 82.0; **Sequential 76.9 / 87.1**; Harness
  Evolution 69.7 / 79.0; Harness Scaling 75.5 / 85.6.
- **Held-out test (Table 3):** direct 63.2 → evolved **63.2**. By model:
  Claude +1.2, GPT-5.4 +0.0, mini −1.4. At K = 10 on Claude: 63.3 → 63.0.
- **K = 10 ablation (Claude only, Tables 7–8):** with tests, **Harness
  Scaling is now the best arm** (90.5 pass@1 / 94.4 pass@10, against
  Sequential 89.6 / 93.1 and Parallel 88.7). Doubling K made the no-test
  arms *slightly worse* (Parallel 74.7 → 73.6, Harness Scaling 76.0 →
  73.6). The caption concedes the gap "remains within run-to-run variance."
- **ARC-AGI-3 (Table 4, mean RHAE):**

  | Model | Claude/Codex CLI | VISTA* | VISTA | full harness |
  |---|---|---|---|---|
  | Claude Opus 4.8 | 44.3 | 43.3 | 59.3 | **77.5** |
  | GPT-5.6 Terra | 4.6 | 21.1 | 25.9 | **38.2** |

  The headline "80%" is the mean of +79.0% and +81.0% relative, [full
  harness] over VISTA*. **Win rates (Table 11) move far less than RHAE for
  GPT:** VISTA 4.6/7.3 levels against full harness 4.8/7.3, so most of
  GPT's VISTA → full gain is action efficiency. For Claude, win rate goes
  4.5 → 5.4 → 6.2.
- **Harness usage (Table 12):** about 11 (Claude) and 20 (GPT) tools are
  written per game, and tool calls make up 143 of 169 and 217 of 260 harness
  operations. Hooks are almost never written (0.2 and 0.0). Case studies
  (tr87, lp85, ls20) show the bottleneck being "perceptual rather than
  strategic". A ~20-line pixel-mask tool takes tr87 from 1/6 to 6/6 levels
  (11.8 → 100 RHAE). The memory-only run's GUIDE.md "records the search
  rather than the rule".
- **EdgeBench (Table 5, best-so-far average):** Claude 48.3 → **54.7**
  (+13.3%), GPT 45.9 → **49.8** (+8.5%); "10.9%" is the mean of the two.
  Harness Scaling wins **7 of 10** model×task cells. It loses NetHack/GPT
  (50.4 vs 49.0), OpenTTD/GPT (51.6 vs 50.5) and Wesnoth/Claude (88.0 vs
  86.0). On OpenTTD/GPT it trails badly early: 20.6 vs 49.5 at 2 h.

## Critique / open questions

- **The paper's own noise floor swallows several of its contrasts.** It
  never reports run-to-run variance, but it ran "direct sampling with the
  initial harness" in several tables, which gives one:
  - GPT-5.4 mini: 59.4 (Table 1) vs 56.9 (Table 2).
  - Claude: 69.9 / 69.5 / 68.5.

  That is a spread of about 1–2.5 points for an unchanged condition. On
  that scale, Table 1's Parallel vs Harness Scaling (72.3 vs 71.8), Harness
  Evolution vs direct (−0.8), and the held-out ±1.4 are all noise.
  "Evolution does not beat test-time scaling" is supported. "Test-time
  scaling beats evolution" is supported only against **Harness Evolution
  with tests** (69.7 vs 82.0 pass@1) and for GPT-5.4 without tests (−5.6).
- **The pass@1 column measures different things in different rows (our
  inference).** With tests, Parallel Sampling's pass@1 equals its pass@5
  in every cell (84.8 / 87.1 / 74.1), so for that arm it is test-oracle
  best-of-5. Sequential and harness arms show pass@1 < pass@5, which fits
  Eq. 1's mean over all k rollouts, early attempts included. The argument
  "if harness revision genuinely produced better harnesses, we would expect
  the improvement to be reflected in pass@1" therefore compares an
  oracle-selected number with an average along the trajectory. pass@5
  (Sequential 87.1 > Harness Scaling 85.6 > Parallel 82.0 > Evolution 79.0)
  is the cleaner comparison, and there the evolution-vs-search gap is small
  except for AHE.
- **One evolution method, handicapped.** Only AHE is run. Meta-Harness
  and AEVO are critiqued but never reproduced, and AHE runs without its
  explore agent. Removing that agent is principled, since it imports
  benchmark-tuned harnesses, but it means "harness evolution" in Tables 1–3
  is AHE-minus-a-component at m = 1, K = 5. Read the TB result as being
  about this configuration, not about the method family.
- **The positive result is not the method that was criticised.**
  §4 tests *cross-task* Harness Evolution, which produces a reusable
  artifact. §5 tests only *per-task* Harness Scaling, which by design
  optimizes on the scored instance and reports the best-so-far score on
  it. That is legitimate test-time adaptation, but it is the same
  "search on the evaluated task" the paper's second critique objects to.
  So nothing in the paper shows a harness evolved on some games transferring
  to others. Harness *evolution* has no positive held-out result here.
- **On ARC-AGI-3, "matched budget" means matched environment actions, not
  matched compute, and the winning arm gets a new capability.** Tool
  execution and reasoning are free under RHAE. [full harness] is the only
  arm with `write_tool`/`run_tool`, and the case study says VISTA "exposes
  no way to inspect pixels programmatically." The +80% therefore bundles
  *letting the agent write and run code over the frame* with *evolving the
  harness*. The missing control is a fixed harness that ships a generic
  code-over-frame tool. Plain Claude Code (44.3 ≈ VISTA* 43.3) does not
  settle it, because the paper does not say whether that arm could parse
  frames programmatically.
- **Only sequential refinement is the §5 baseline.** Parallel sampling,
  the most consistent arm on TB, is absent from both game benchmarks. On
  ARC the baseline variant behind the 80% (VISTA*) is not the best one for
  Claude (Claude Code 44.3), though the difference is negligible.
- **Small n, max statistics, no intervals.** ARC is 25 games with
  apparently one run each, and per-game swings between arms are huge
  (s5i5 Claude: 41.3 vs 3.0 across the two Sequential variants). EdgeBench
  is 5 tasks × 2 runs scored as a running max, and arms with more volatile
  trajectories gain more from a max. The 10.9% is a mean of two relative
  gains driven by low-base tasks (DCSS +31% and +34%).
- **What the TB failure analysis adds (Appendix A.2).** The harness edits
  are reasonable: finalization gates, turn-budget trackers, "do not weaken
  tests" rules. The meta agent climbs from advisory prompt text to runtime
  middleware when prose plateaus, which is
  [[concepts/enforcement-boundary-placement]] rediscovered by search. But
  the content is mostly per-task facts ("install Coq 8.16.1", "never open
  SQLite before backing up"). That is the memorize-not-distill failure
  [[concepts/compression-as-generalization-test]] predicts: none of it
  would survive a 128-token strategy channel.
- **The digest calls this v2. The PDF is v3** (arXiv stamp "v3 [cs.AI]
  1 Oct 2026").

## Trust signals

- **Credibility:** 3. Strong group (AI2 and UW: Hajishirzi, Tsvetkov,
  Smith, Dasigi), a public code organization, and a careful, conservative
  design on the negative side: harness-edit cost excluded, explore-agent
  leak removed, a K = 10 ablation reported even though it narrows the
  claim. Held at 3 for several reasons. It is an unreviewed preprint. A
  paper about evaluation rigor reports no variance or CIs, and its key
  contrasts sit inside its own visible ~2.5-point run-to-run spread. Only
  one evolution method is instantiated. The pass@1 definition appears to
  differ across arms. The headline positive result is confounded by a
  capability difference and uses a different method (per-task scaling)
  against a weaker baseline set.

## Follow-up

- **Relevance:** 4. This is the first paper in the graph to run
  harness/agent evolution against **same-budget test-time-search arms** and
  find it does not win on a standard agent benchmark. It is also the third
  independent finding that evolved harnesses add little or nothing on
  disjoint tasks: here held-out +0.0; [[literature/papers/xia2026rrsi]]'s
  unregularized loop OOD 40.3 vs 39.7 unevolved; and
  [[literature/papers/zhu2026bad]]'s +8.16% → −5.10% under protocol shift.
  It materially extends [[concepts/hce-evaluation]] and
  [[concepts/evolutionary-expansion]]. It is not a 5 because it seeds no
  concept and its own statistics are thin.
- **Counterweight to [[literature/papers/srikanth2026recursive]] (AIDE²).**
  AIDE² selects rewrites on a private split and shows external transfer,
  but it has no arm that spends the same 100-node budget on plain
  repeated runs of AIDE₀. This paper says that arm is the one to beat.
  Ask that of any recursive-improvement claim before crediting it.
- **The matched-baseline point is related to, but not the same as, the
  08-23 `elevate-paired-control` proposal.** That proposal adds a
  *known-positive control* so a quiet gate can be told apart from a broken
  one. This paper adds a *matched null-method control* (same budget, no
  harness change) so a gain can be attributed to evolution rather than
  compute. They are complementary controls. This paper is not evidence for
  that proposal as written.
- **For any `/iterate`-style loop that edits its own skills or prompts:**
  the default comparator should be "rerun the unchanged loop K times and
  keep the best", with the same rollout or token budget. A skill edit kept
  without beating that arm is unattributed.
- **Where evolution earned its keep, it wrote executable tools, not prose.**
  On ARC-AGI-3, memory-only evolution "can only accumulate prose about a
  misperception it cannot correct". Tool writing fixed perception and
  replaced manual search with BFS. This supports the executable-over-advisory
  line in [[concepts/skill-library-lifecycle]] and
  [[concepts/compression-as-generalization-test]] (a prose summary is
  compression without a check).
- **Candidates:** AHE (arXiv 2604.25850), Meta-Harness (arXiv 2603.28052),
  AEVO (arXiv 2605.13821), VISTA (Han et al. 2026,
  vista-research.github.io), EdgeBench (arXiv 2607.05155), and Li et al.
  2026, "Benchmark test-time scaling of general LLM agents" (arXiv
  2602.18998). None are ingested yet.
