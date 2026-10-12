---
kind: paper
title: "Humanize MLE Agents: Multi-Agent Session Alternation Beyond Single-Agent Limits"
authors: ["Changye Li", "Zijian Zhang", "Muheng Li", "Yiduo Pang", "Tingjun Wu", "Menghan Li", "Yitong Liu", "Hongkai Guo", "Mengchi Chen", "Chenyu He", "Kaisheng Ma", "Wei Xu", "Song Han", "Sihao Liu", "Ligeng Zhu"]
institutions: ["Tsinghua University", "NVIDIA", "Zhejiang University", "MIT"]
year: 2026
venue: "arXiv 2610.13494v1 (cs.AI, 2026-10-09), preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.13494"
code_url: "https://github.com/humanfia/hma"
citations: null
source: "raw/papers/li2026humanize.pdf"
added: "2026-10-12"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/file-as-bus]]"
tags: ["mle-bench", "session-alternation", "context-reset", "heterogeneous-agents", "model-swap", "shared-workspace", "fixed-k-interruption", "natural-termination-handoff", "long-horizon-deceleration", "harness-staleness", "semi-mdp", "options", "final-review", "candidate-archive", "redirect-before-halt", "headline-cut"]
---

# Humanize MLE Agents: Multi-Agent Session Alternation Beyond Single-Agent Limits

## TL;DR

HMA runs two complete, unmodified agents on one shared workspace. Each is a
native model–harness pair: Claude Code, Codex, Kimi Code or DeepSeek Harness.
They take turns in sessions capped at k = 5 accepted experiments, the context
is wiped at every handoff, and the other agent spends the last 15 minutes
choosing the final candidate from an archive of everything evaluated. On the
full 75-competition MLE-bench, under 6 h on one A10, the best pair
(Opus 5 first, then GPT-5.6-sol) reaches 78.2% any-medal. Opus 5 alone in
Claude Code gets 72.4% under the same budget. The paper names two motivating
effects. *Long-horizon deceleration*: output tokens per response fall 30–45%
from hours 0–2 to hours 3–5. *Harness staleness*: three research harnesses
lose to the native DeepSeek harness when the model moves from V4-Flash to
V4.1-Flash.

For this graph the important part is narrower than the headline. HMA is a
**controller-enforced, schedule-triggered context flush plus model swap**,
and on average it beats continuing in one session at matched wall-clock.
But it **never separates the flush from the swap**: there is no same-model
self-alternation arm. In its only cap ablation, uncapped alternation at
natural termination beats every capped run. Its most useful datum for
redirect-before-halt is a side condition, NTA. There, an agent that would
have stopped hands the workspace to a different model instead.

## Claims

- **Single sessions decelerate.** "As single-session execution lengthens,
  the agent's reasoning intensity declines alongside slowing progress."
- **Custom harnesses age with the model.** "The benefits of these custom
  harnesses vanish as the underlying model improves." The proposed reason is
  "model–harness alignment acquired during training".
- **Alternation beats single-agent reference limits under stated
  conditions.** The paper models the workflow as a context-reset semi-MDP
  (CR-SMDP) and proves three results:
  - joint alternation exceeds the best single-agent ceiling (Thm 3.3);
  - a sufficient experiment budget scales as (2k/a)·log(1/(1−p₀)) (Cor. 3.4);
  - fixed-k interruption keeps the advantage when Δ + 𝒜 > 0 (Thm 3.5), and
    some finite k suffices if interruption losses decay geometrically
    (Cor. 3.6).
- **State of the art on MLE-bench.** "78.2%, exceeding the strongest
  open-source baseline by 8.0 points with a quarter of its time budget."
- **Cross-pair generality.** "Among six configurations spanning five
  heterogeneous agent pairs, HMA improves any-medal rate over the constituent
  baselines by 6.4 points on average."
- **Exploration persists.** "HMA retains 86.4% of its mean output tokens per
  response, compared with 60.7% for matched single-agent baselines."
- **Final review helps retention.** A cross-agent pick over the candidate
  archive raises retained medals from 311 to 319 of the 330 runs that ever
  produced one.

## Methods

- **Framework.** Built on Humanize (Liu et al. 2026, arXiv 2610.08900,
  same group). Each agent keeps its own harness, tools and context
  management unchanged. The controller only:
  - counts accepted experiments;
  - interrupts at k;
  - resets context with f(W, E, c) = (W, E, ∅), where W is the shared
    workspace and E the retained result;
  - archives every evaluated candidate with its author.

  No task-specific knowledge is injected.
- **Models (all at `max` reasoning effort).**

  | Model | Harness |
  |---|---|
  | GPT-5.6-sol | Codex |
  | Claude Opus 5 | Claude Code |
  | GLM-5.3 | Claude Code |
  | Kimi K3 | Kimi Code |
  | DeepSeek-V4-Flash, DeepSeek-V4.1-Flash | DeepSeek Harness |

  The AOPT case uses Opus 5.5 and GPT-6-astra.
- **Benchmark.** The full MLE-bench, 75 competitions: Lite 22, Medium 38,
  High 15. "Lite" in Table 2 is the 22-task low-complexity column, not the
  evaluated split.
  - Metrics: any-medal, gold, Med+ (above the leaderboard median) and mean
    percentile.
  - The returned candidate is graded offline; agents never see test
    scores.
- **Budget and hardware.** 6 h per task (21,600 s), including 900 s for
  final review. One A10 (24 GB), 30 vCPU and 220 GiB RAM are shared by both
  agents.
- **Baselines.**
  - *Single-agent `/goal` runs.* The authors ran each model in its native
    harness, continuously until natural termination or 6 h. These are
    like-for-like on model, budget and hardware.
  - *Open-source workflows, quoted from their papers.*

    | Workflow | Model | Budget | Hardware |
    |---|---|---|---|
    | ScienceFlow | DeepSeek-V4-Flash-Preview | 24 h | up to 2 GPUs |
    | MLEvolve | Gemini-3.1-Pro-preview | 12 h | H200 |
    | ML-Master 2.0 | DeepSeek-V3.2-Speciale | 24 h | 2× RTX 4090 |

    None of these is model-matched to HMA.
- **Repeats.** C.3 says "each configuration is evaluated in three
  independent repeats". That clearly holds for the single-agent baselines,
  which carry ± stds, and for the headline Opus→GPT row: 78.2% × 75 is not
  an integer. But the other five HMA rows are integer counts out of 22, 38,
  15 and 75 in every column. Examples: 73.3 = 55/75, 72.0 = 54/75,
  70.7 = 53/75, 69.3 = 52/75, 62.7 = 47/75. App. F.2 calls the GPT→Kimi
  k = 5 result "the existing k = 5 run". App. F.3 audits "one complete
  repeat per configuration, using the first Opus-first GPT repeat". Our
  reading: **five of the six HMA configurations are single runs compared
  against 3-repeat means.** No HMA std appears anywhere, although Table 2's
  caption points to C.3 for them.
- **Deceleration measurement (App. D.1).** Mean provider-reported output
  tokens per response, in a 30-minute trailing window. This excludes tool
  returns and input tokens. The data are six `/goal` configurations × 75
  tasks × 3 repeats, and the comparison is hours 0–2 against hours 3–5.
- **Staleness measurement (App. D.2).**
  - Three research harnesses against the native DeepSeek Harness, on a
    16-task MLE-bench subset, with DeepSeek-V4-Flash and V4.1-Flash.
  - Task-specific cold-start knowledge is disabled. For ML-Master 2.0 that
    removes "prior wisdom initialized from 407 Kaggle competitions".
  - Research harnesses: one run per cell, 12–24 h. Native harness: 6 h,
    3 repeats.
- **Cap selection (App. B).** "Prefix retention" is computed on recorded
  sessions: 6 goal and 5 natural-termination-alternation (NTA)
  configurations, 1,065 options. It is the share of a session's observed
  best-so-far gain reached within its first k experiments.
- **Ablations (App. F).** All run on the GPT-5.6-sol / Kimi K3 pair, one
  repeat each:
  - starting order (GPT-first vs Kimi-first);
  - NTA vs HMA k = 5. NTA has no cap: an agent's session ends when it
    naturally terminates, then the other agent starts with a reset context;
  - cap k ∈ {1, 3, 5};
  - a selection-only audit of final review over 450 task runs.

## Results

- **Main table (Table 2, any-medal %, 75 tasks).** "Δ stronger" is our
  arithmetic.

  | Configuration | HMA | Stronger single | Δ stronger | Avg. of single | Δ avg (paper) |
  |---|---|---|---|---|---|
  | Opus → GPT | **78.2** | Opus 72.4 ± 1.6 | **+5.8** | 70.2 | +8.0 |
  | GPT → Opus | 73.3 | Opus 72.4 ± 1.6 | +0.9 | 70.2 | +3.1 |
  | GPT → DeepSeek-V4.1 | 72.0 | GPT 68.0 ± 1.3 | +4.0 | 64.7 | +7.3 |
  | GPT → Kimi | 70.7 | GPT 68.0 ± 1.3 | +2.7 | 61.1 | +9.6 |
  | GPT → GLM | 69.3 | GPT 68.0 ± 1.3 | +1.3 | 63.8 | +5.5 |
  | DeepSeek-V4.1 → Kimi | 62.7 | DeepSeek 61.3 ± 4.1 | +1.4 | 57.8 | +4.9 |
  | **Mean** | | | **+2.7** | | **+6.4** |

  - Published workflows: ScienceFlow 70.2, MLEvolve 65.3, ML-Master 2.0
    56.4.
  - On the High split the gain is concentrated in the headline row: 62.2
    against 46.7 for both constituents. The other five rows are 40.0–53.3.
- **Over time (Fig. 4, Table 9).**
  - Two-hour any-medal across configurations is 42.7–60.0%.
  - The matched-weight HMA curve ends at about 71% against about 65% for
    the matched goal reference (read off the figure).
  - Output tokens per response, hours 0–2 → 3–5: HMA 610.9 → 527.6
    (86.4%); goal 650.3 → 394.9 (60.7%).
- **Deceleration (Table 6).** Every model's tokens per response fall from
  hours 0–2 to hours 3–5:

  | Model | Change |
  |---|---|
  | GPT-5.6-sol | −36.5% |
  | Opus 5 | −45.0% |
  | GLM | −41.0% |
  | DeepSeek-V4.1 | −38.8% |
  | Kimi | −33.4% |
  | DeepSeek-V4 | −29.8% |

  Prefix retention (Fig. 3): k = 1, 3 and 5 retain 63.2%, 93.4% and 97.8%
  of observed within-session gains while keeping 24.8%, 54.9% and 73.8% of
  the experiments. Within a session, gains are front-loaded.
- **Staleness (Table 1, 16 tasks, medal %).**

  | Harness | V4-Flash | V4.1-Flash |
  |---|---|---|
  | DeepSeek Harness (native, 6 h) | 16.7 | **77.1** |
  | ScienceFlow (24 h) | **25.0** | 68.8 |
  | ML-Master 2.0 (24 h) | 6.3 | 37.5 |
  | MLEvolve (12 h) | 18.8 | 50.0 |

  One ScienceFlow V4.1 run was disqualified because the checkpoint had seen
  the test answers on Chaii.
- **Starting order and cap (App. F.1–F.2; GPT-5.6-sol / Kimi K3; single runs
  unless noted).**

  | Arm | GPT-first | Kimi-first |
  |---|---|---|
  | Single agent `/goal` (3 reps) | GPT 68.0 | Kimi 54.2 |
  | NTA (no cap, handoff at natural termination) | **77.3** | 56.0 |
  | HMA k = 5 | 70.7 | 66.7 |
  | HMA k = 3 | 66.7 | — |
  | HMA k = 1 | 65.3 | — |

  - Under NTA the first model holds about 81–83% of the budget.
  - In GPT-first NTA the second agent responds by 3 h on 17/75 tasks and by
    5.75 h on 19/75. In Kimi-first NTA the figures are 15/75 and 47/75.
  - With HMA k = 5, every task gets a second-agent response, with a median
    first response at 1.89 h.
- **Option closures (Table 12, k = 5, 6 configs × 1 repeat).**
  - Initial options: 389 hit the cap, 57 end naturally, 4 hit the deadline.
  - Later options: 394 cap, 598 natural, 446 deadline.
  - 248 of the 655 natural exits produced no accepted experiment.
- **Final review (Table 13).**
  - It changes 40 of 450 selections (8.9%).
  - It retains 319/330 historical medals, against 311/330 for the
    pre-review standing candidate, and loses none.
  - It has zero net effect on the headline Opus→GPT repeat (58 → 58).
  - 35 GLM-pair reviews were silent and fell back to the standing
    candidate.
  - Example: on MBM-01 the returning GPT session *overwrote* a DeepSeek
    bronze with a worse candidate, and final review restored the bronze.
- **Case studies (App. G, selected runs).**
  - INGV: MAE 34,234, 46.5% below the best of six goal runs.
  - QUEST: Spearman 0.5075 against 0.4725.
  - PetFinder: gold, where neither constituent medalled in three goal runs.
  - Smartphone: bronze from a late Kimi repair.

  In 4 of 5 INGV returns the response length rises 1.6–3.6× after a
  handoff. The Smartphone gain comes without any length rebound.
- **AOPT kernel task (§4.5).** One 48 h run with Opus 5.5 and GPT-6-astra
  at cap 3 reaches 878 cycles. That is 10th on a community leaderboard,
  1.6% behind 864. The leaderboard uses a nine-seed judge the run did not
  go through.

## Critique / open questions

- **The digest framing overstates the paper (house pattern, 32nd of 32).**
  - The 78.2% is the best of six pairs, and it is the only multi-repeat HMA
    row.
  - The "+8.0 over the strongest open-source baseline" compares
    Opus 5 + GPT-5.6-sol against ScienceFlow's *published* number on
    DeepSeek-V4-Flash-Preview. Opus 5 alone in Claude Code (72.4%) already
    beats every published workflow by 2.2 points. On a like-for-like model
    comparison, HMA's own gain on its best pair is **+5.8 over Opus
    alone**.
  - Averaged over the six pairs, the gain over the stronger constituent is
    **+2.7**, not the abstract's 6.4. That figure is measured against the
    *mean* of the two constituents, which credits HMA for not doing as
    badly as the weaker model. Three of the six pairs gain less than
    1.5 points, and GPT→Opus (+0.9) is inside Opus's own ±1.6.
  - "A quarter of its time budget" is wall-clock only. HMA runs two frontier
    APIs at `max` effort. The paper says token counts "measure output
    volume … not a common monetary or computational cost", and no cost is
    reported.
- **Five of six HMA rows look like single runs set against 3-repeat
  baselines, with no HMA std.** The +0.9 to +2.7 rows are therefore
  inside plausible run-to-run noise. The size of that noise is shown by a
  datum the paper does not discuss: GPT-first NTA scores 77.3% even though
  Kimi responds on only 19/75 tasks, so most of that run is GPT working
  alone. That is 9.3 points above GPT's 3-repeat 68.0 ± 1.3. Either the
  natural-termination handoff on those 19 tasks is worth up to ~9 points,
  or single-run differences of this size are within noise for this setup.
  The NTA archives are also "historical", scored with corrected answer
  keys, and seven GPT-first tasks actually started with Kimi.
- **Flush and swap are never separated.** The digest asked whether the gain
  comes from heterogeneity or from context reset alone. The paper cannot
  say. There is no A↔A arm (same model, k = 5, context reset), so every
  HMA–goal contrast bundles four things:
  - a different model;
  - a different harness;
  - a fresh context;
  - a final review by the other agent.

  The k ablation does not help either. On GPT→Kimi, k = 1 (65.3) and
  k = 3 (66.7) fall *below* GPT alone (68.0). k = 5 (70.7) is above it.
  Uncapped NTA (77.3) beats all three. So in the only pair where the cap
  is varied, the forced fixed-k flush is not shown to add anything for the
  stronger-first order. Its demonstrated benefit is robustness to starting
  order (Kimi-first 56.0 → 66.7). The paper says as much in App. F.2, but
  §3.3 and the abstract still present fixed-k interruption as the
  mechanism.
- **"Long-horizon deceleration" is measured by a confounded proxy.** Output
  tokens per response track what kind of work is going on as much as how
  hard the agent is reasoning. Early responses write data pipelines and
  training scripts; late ones poll jobs, tweak hyperparameters and blend.
  "Provider-reported output tokens" also mean different things across
  providers, with and without hidden reasoning. HMA's 86.4% "retention" is
  partly mechanical: each reset restarts the read-the-workspace,
  write-new-code phase. The paper's own Smartphone case improves with no
  length rebound. The stronger deceleration evidence is Fig. 3: 97.8% of
  within-session gains arrive in the first five experiments. But any
  best-so-far curve on a bounded metric looks like that, and the paper does
  not show that a fresh *same-model* session would bend the curve.
- **"Harness staleness" is one model step, 16 tasks and single runs, on
  ported harnesses.**
  - The native harness jumps from 16.7% to 77.1% between V4-Flash and
    V4.1-Flash, a 4.6× change from a point release. The V4 baseline may be
    the anomaly, rather than the research harnesses decaying.
  - The research harnesses run with cold-start knowledge disabled.
    ML-Master 2.0's defining prior is switched off.
  - The ports run far below the published numbers the main table quotes,
    on different subsets and models. For example, ScienceFlow scores 25.0%
    in the authors' V4-Flash port against a published 70.2% on
    V4-Flash-Preview.
  - The observation is real enough to note: research harnesses did not beat
    the native one on the stronger model, even with 2–4× the time. It is
    not a trend. The paper's remedy, a workflow-level wrapper around native
    harnesses, is itself a harness, and its own staleness is untested.
- **The theory is decorative.**
  - Assumption 3.2 posits a per-cycle probability a > 0 of reaching states
    beyond the single-agent ceiling. Theorem 3.3 then shows that failure
    decays geometrically. The assumption carries the conclusion.
  - Theorem 3.5 is a telescoping identity
    (Perf_k − U* = Δ + 𝒜), restated as a sufficient condition.
  - α, β, a, Δ and 𝒜 are never estimated. The cap was chosen
    empirically, k ≤ 5 from Fig. 3, as the paper concedes.
- **Agents overwrite the champion.** Assumption 3.1 (the best result is
  retained) is a modelling assumption that the agents violate. A returning
  agent can replace a medal candidate with a worse one (MBM-01, MBM-20). The
  workflow needs the archive and the final review to restore it, and 11
  historical medals are still lost. A shared workspace without a monotone
  keep gate erodes its own best, the same lesson as
  [[literature/papers/samanta2026learning]].
- **Case studies are selected.** They were chosen for "inspectable
  artifacts", and the paper says they "illustrate observed mechanisms
  rather than estimating their average causal effect". The AOPT run is n = 1
  on newer models than the main table and was not judged by the
  leaderboard's own procedure.
- **Self-halting in the baseline is never quantified.** `/goal` runs stop at
  natural termination, so the baseline *can* halt early. How often the
  single agents stop before 6 h, and how much budget that leaves unused, is
  not reported. The only indirect figure is Table 10's second-agent timing
  under NTA. The HMA–goal gap therefore mixes "redirect instead of
  continue" with "redirect instead of stop", in unknown proportions.

## Trust signals

- **Credibility:** 3.
  - *Prior.* Tsinghua, NVIDIA and MIT (Song Han's group, with Ligeng Zhu
    and Sihao Liu of NVIDIA), and code released at humanfia/hma. That
    alone would be a 4.
  - *What pulls it to 3.*
    - It is a preprint.
    - Five of six HMA configurations appear to be single runs, contrary
      to the stated 3-repeat protocol, and no HMA variance is reported.
    - Every ablation is single-run on one pair.
    - The headline "+8.0" compares across models using published numbers.
    - The theory assumes its conclusion.
    - Staleness rests on one model step with ported, knowledge-disabled
      harnesses.
  - *To its credit.* The like-for-like single-agent baselines (3 repeats,
    same models, hardware and budget) are a sound design. The appendices
    honestly report the starting-order, cap and final-review details that
    undercut the headline. An AI-use statement is included.

## Follow-up

- **Relevance:** 4. Material new evidence for the stall-handling thread in
  [[concepts/budget-as-ceiling]], and a new axis for
  [[concepts/hybrid-model-backends]]: alternating whole model–harness pairs
  in time, rather than splitting roles. It is the first MLE-bench-scale
  test of a scheduled context flush that keeps the workspace
  ([[concepts/context-eviction-policy]], [[concepts/file-as-bus]]). Not a
  5: the mechanism is never isolated, and most of its rows are single runs.
- **Bearing on the held redirect-before-halt idea
  (`docs/system-proposals/_index.md`).**
  - *Supports "controller, not prompt", weakly.* The handoff is forced by
    the controller and carries no prompt-level nudge. On average it beats
    one continuous session at matched wall-clock, which is consistent with
    [[literature/papers/yan2026traceml]] and
    [[literature/papers/xu2026conflictguide]].
  - *The closest redirect-at-halt datum yet is a side arm.* NTA hands off
    exactly when an agent would stop. GPT-first gives 77.3 against GPT
    alone at 68.0; Kimi-first gives 56.0 against Kimi alone at 54.2. Both
    are single, historical runs against 3-repeat means, and the GPT-first
    gap is larger than the second agent's participation easily explains.
    So this is a direction, not an effect size. It is still the first source
    whose two arms differ in whether the run is allowed to stop.
  - *The redirect move it tests is a model swap plus a context flush.*
    `/iterate` runs one backend (Opus). The transferable rung is "on stall,
    hand the workspace to a *different* backend with a fresh context". HMA
    cannot say whether a fresh context on the *same* model would do as
    well. Fork-and-flush (arXiv 2610.07447, 10-12 digest item 1) is the
    same-model, fresh-context test and should settle that.
  - *A schedule is not a stall trigger.* HMA hands off every five
    experiments regardless of progress. In the one ablation, k = 1 and
    k = 3 *lose* to the strong single agent. So frequent forced redirects
    cost something. A stall-triggered redirect, firing only after the
    counter ticks, is the better-motivated design, and nobody has tested it
    against this.
  - *Keep the monotone champion.* Alternating agents overwrite the standing
    best. A redirect rung needs the existing keep-if-better gate plus an
    archive. HMA's cross-agent final review over that archive recovers
    medals the workspace had lost.
- **For [[concepts/context-eviction-policy]].** A full flush, keeping only
  the workspace and the retained result, is the extreme eviction policy. It
  is adopted here for a reason other than overflow: to restart a
  decelerating session. That is a third role for the policy, after
  liveness and selection, and it is not yet isolated.
- **Check at the next ingest.** Ganz et al. 2026, "The handoff tax"
  (arXiv 2608.24358), measures the quality cost of continuing another
  model's trajectory. That is the opposite prediction to HMA's, and the
  graph has no note on it. Also worth ingesting: ScienceFlow (Zhao et al.
  2026, arXiv 2608.14354), the quoted baseline; Humanize (Liu et al. 2026,
  arXiv 2610.08900), the framework; and Relay, don't route (Luo et al.
  2026, arXiv 2608.05651), population handoff.
