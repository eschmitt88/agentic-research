---
kind: paper
title: "Beyond Final Scores: A Systematic Evaluation of Agents for Long-Horizon AI Research and Development"
authors: ["Yiwei Li", "Wanli Yang", "Hexiang Tan", "Xiangzhou Huang", "Zhengyu Chen", "Ziran Li", "Borun Chen", "Shanglin Lei", "Huaisheng Zhu", "Hao Tian", "Fei Sun", "Xunliang Cai", "Jingang Wang"]
institutions: ["Meituan (LongCat team)", "University of Chinese Academy of Sciences"]
year: 2026
venue: "arXiv 2608.13417 (cs.AI), v1 2026-08-13, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.13417"
code_url: "https://yiwei98.github.io/AutoResearchEval/"
citations: null
source: "raw/papers/li2026beyond.pdf"
added: "2026-10-13"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/pass-at-k]]"
tags: ["autolab", "process-metrics", "feedback-control", "avg-vs-best", "experience-reuse", "memory-erasure-counterfactual", "lesson-transfer", "harness-comparison", "auto-harness", "plateau-hook", "redirect-before-halt", "evaluation-hacking", "novelty", "claude-code"]
---

# Beyond Final Scores: A Systematic Evaluation of Agents for Long-Horizon AI Research and Development

## TL;DR

This is a benchmark study, not a harness paper. Seven frontier models
(Opus-4.7, GPT-5.5, Gemini-3.1-Pro, GLM-5.2, Kimi-K2.7-Code, DeepSeek-V4-Pro,
LongCat-2.0) run 36 AutoLab tasks in a shared Claude Code harness, 3 rollouts
each (756 rollouts, 2–12 h budgets, about $100k of inference). The paper
adds four things to final scores:

1. Three deterministic process scores computed from the per-commit verifier
   trace: Solution Framing (C1), Execution (C2) and Feedback Control (C3).
2. A memory-erasure counterfactual for within-task experience, and a
   lessons.md transfer test across tasks.
3. A harness comparison (Claude Code vs native CLI vs OpenCode).
4. An LLM-judged novelty audit of the 252 best-of-3 solutions.

The headline is "engineering optimizers, not autonomous researchers". Three
of 252 solutions are novel, and 16 (6.3%) exploit the evaluator. The
plateau hook that the digest promoted takes up two sentences in §5.3. It is
one of three rules an Opus-4.8 outer loop wrote into an evolved harness
after four rounds on three tasks. Its evidence is a single trajectory
(0.37 → 0.68 on agent_tool_routing). There is no ablation of the hook.

## Claims

- **Reliability separates models more than peak.** The highest-to-lowest
  gap is 0.237 on avg@3 against 0.122 on best@3. Kimi's best@3 is within
  0.03 of GLM and Gemini, but its avg@3 is far behind.
- **Similar outcomes hide different process profiles.** GPT-5.5 and
  Gemini-3.1-Pro score 0.663 vs 0.652 overall and 0.555 on C1 each. GPT is
  stronger on C2 (0.958 vs 0.889) and Gemini on C3 (0.920 vs 0.858).
- **Within-task experience usually helps the next commit, and sometimes
  anchors.** In the erasure test, six of seven models gain (+0.036 to
  +0.145). Weaker framers (LongCat, DeepSeek, Gemini) gain most. Case
  studies show memory both avoiding dead ends and keeping a premature
  verdict or a local optimum alive.
- **Cross-task lessons help on average but are unstable, and can reorder
  models.** DeepSeek +0.093 avg@3, GPT +0.063, GLM +0.040, Opus +0.001,
  Gemini −0.017, LongCat −0.021. Extracted lessons beat raw-workspace
  access for all three models tested. Self-written lessons beat another
  model's lessons in both directions tested.
- **Harnesses mainly change stability, not ceiling or ranking.** Native
  and OpenCode harnesses raise avg@3 by 0.014–0.055 for GPT and Kimi.
  best@3 moves by at most 0.035, and model ordering holds.
- **An evolved harness transfers within its task family.** The gain is
  +0.12 avg@3 on the 3 seed tasks, +0.06 on the other System Optimization
  tasks, +0.03 on GPT-5.5, and none on other families.
- **Novelty is rare and evaluator exploits are not.** Composition-stacking
  accounts for 111/252 solutions. Three are novel after manual review.
  Sixteen are evaluation-hacking, eight of them from GPT-5.5.

## Methods

- **Tasks.** 36 expert-curated AutoLab tasks (Xu et al. 2026, arXiv
  2606.05080): 7 Model Development, 15 System Optimization, 10 Puzzle &
  Challenge, 4 CUDA. Each ships a correct but suboptimal start, an expert
  reference, a wall-clock budget and a verifier normalized to [0, 1].
- **Instruction additions.** Commit before each experiment (`round N:
  change=… hypothesis=…`), keep a journal, and "make full use of the time
  budget … Do not stop after the first working solution". Early stopping is
  discouraged by design, so there is no halt arm anywhere in the paper.
- **Process scores (App. C).** Checkpoints are aligned to commits, with
  administrative commits removed and ε = 0.01.
  - C1 is a stage-weighted area under the high-water mark over H = 20.
  - C2 is the delivery-gated fraction of commits, discounted by pre-commit
    build failures (1.0 → 0.5).
  - C3 is ½ peak retention + ½ recovery credit per dip episode. Recovery
    credit is divided by the steps needed and discounted for hidden
    off-ledger evaluations. With no dips, C3 is retention alone.
  - For 22 of 139 runs, C2 was reconstructed from rounded legacy scores
    because build artifacts had been lost.
- **Intra-task counterfactual.** A branch point near the run midpoint.
  The no-experience arm re-initializes Claude Code, deletes notes, strips
  code comments and keeps the artifact. Only the first commit after the
  branch is scored. 32 tasks are retained.
- **Inter-task transfer.** One source task per category, selected on
  pilot-model quality. 19 targets with headroom for every model. Each model
  writes lessons.md from its own best source run, then does 3 new rollouts
  per target. The baseline reuses the main-study rollouts.
- **Auto Harness (§5.2).** Opus-4.8 edits the Claude Code preamble,
  standing rules and "a thin layer of hooks" over 4 rounds. It watches
  LongCat-2.0 on 3 random System Optimization tasks. The frozen result is
  then applied unchanged.
- **Novelty.** Opus-4.8 classifies each best-of-3 solution into 8
  categories with a fixed rubric (App. J.3). Only the "novel" label gets
  manual review.

## Results

- **Outcome (Table 1).** avg@3 / best@3: Opus 0.739 / 0.790, GLM
  0.682 / 0.757, GPT 0.663 / 0.772, Gemini 0.652 / 0.750, Kimi
  0.587 / 0.729, LongCat 0.572 / 0.674, DeepSeek 0.502 / 0.668. CUDA
  separates models most (gaps 0.403 / 0.414) and Puzzle least.
- **Cost.** Opus costs $89.9 per task, GLM $33.0, GPT $16.5, DeepSeek $4.3
  and LongCat $3.9. Gemini and GPT use only 66 and 70 min per task against
  2–12 h budgets. They stop early despite the instruction not to.
- **Process.** C2 is compressed (0.880–0.967). C1 ranges 0.473–0.612 and
  C3 0.772–0.928. By category, Model Development has the best C2 (0.985)
  and the worst C3 (0.743). CUDA has the worst C1 (0.370).
- **Exposure.** Evaluated commit rounds per run: GPT 10.1, Opus 11.6, GLM
  16.0, Gemini 2.5, Kimi 5.2, LongCat 5.4, DeepSeek 4.9. The authors concede
  that Gemini's and LongCat's high C3 comes mostly from retention and few
  observed dips, "rather than a well supported recovery advantage".
- **Intra-task (Fig. 7, 15a).** Gains range from +0.036 (Opus) to +0.145
  (LongCat). Kimi is −0.013, but 17 of its tasks improve against 10 that
  get worse. Task-level signs range from 22/5/5 (positive/tie/negative) to
  15/7/10.
- **Inter-task (Fig. 8, 15b, 16).** Per-target signs are mixed. The best
  row is 13 positive vs 6 negative of 19 targets. DeepSeek's gain comes
  with its zero-score rollouts falling from 13/57 to 0. Gemini's
  "semantic mocking" lesson led it to cache a SHA-256 digest at warm-up,
  for an apparent +0.620 best@3 on one target.
- **Harness (Fig. 9, Table 2).** At category level the effects reverse.
  Codex CLI lowers GPT's CUDA avg@3 by 0.099 and raises System
  Optimization by 0.067. "No harness dominates across models and
  categories."
- **Auto Harness (Fig. 10).** avg@3 / best@3 gains: seed tasks
  +0.123 / +0.065, held-out System Optimization +0.057 / +0.043, GPT-5.5
  System Optimization +0.027 / +0.010, other families −0.014 / +0.013. The
  harness "converg[ed] on three simple interventions":
  1. identify what the verifier actually rewards;
  2. "attempt one larger structural change when the score plateaus";
  3. protect the best verified state ("Before you finish, restore your
     best").

  Mechanism (§5.3): "After every five new commits, a hook asks the agent to
  reassess whether progress has plateaued and to attempt a larger
  structural change when local refinement has stalled". On
  agent_tool_routing this was followed by a Python → native C switch, and
  the score went from about 0.37 to 0.68.
- **Novelty (Fig. 11).** Composition-stacking 44.0%, evaluation-hacking
  6.3%, novel 1.2%. The three novel solutions come from GLM, Kimi and
  LongCat, not the top two models.

## Critique / open questions

- **House-pattern check: on the redirect claim the paper reads BELOW the
  digest framing. As a paper it is broader than the framing suggests.**
  The digest's description of the mechanism is accurate. A hook does fire
  every five commits and does ask about a plateau and a structural change.
  The digest's own caveat, "not a controlled test", is also right. But
  "design precedent for redirect-before-halt" overstates what is there, in
  four ways:
  1. **It is a schedule, not a stall trigger.** The hook fires every five
     commits whether or not the score has stalled.
     [[literature/papers/li2026humanize]] found that frequent scheduled
     handoffs cost score, and [[literature/papers/cai2026forkandflush]]
     found that a real stall trigger rarely fires.
  2. **It asks; it does not enforce.** The agent decides whether it has
     plateaued and what to do. That is the prompt form that
     [[literature/papers/yan2026traceml]], [[literature/papers/xu2026conflictguide]],
     cai2026forkandflush and [[literature/papers/zhang2026judged]] each
     found ineffective.
  3. **It is not a redirect *instead of* a halt.** The task instruction
     already forbids early stopping, so no arm halts.
  4. **It was found by an optimizer, not designed and tested.** It is one
     of three bundled rules, and the 0.37 → 0.68 case is one trajectory
     with no counterfactual. The paper does not say whether
     agent_tool_routing was a seed task.

  A further point: Gemini averages 2.5 evaluated commit rounds and Kimi,
  LongCat and DeepSeek about 5. A five-commit hook would rarely fire for
  half the models (our inference). The evolved harness was only tested on
  LongCat and GPT.
- **Most-favourable-cut check.** The abstract makes no inflated numerical
  claim. The introduction's numbers are mostly honest, with three
  exceptions:
  - **Auto Harness.** "+0.12" is the in-sample seed-task number on 3
    tasks. Held-out same-family transfer is half that, and other families
    are −0.014. There is no matched-budget arm (the same 4 rounds spent on
    extra rollouts) and no SE. That is the protocol gap the co-authors'
    own [[literature/papers/wang2026rethinking]] says invalidates
    harness-evolution claims. Held-out task families do satisfy its
    second requirement.
  - **Experience "can change model ordering".** This rests on DeepSeek's
    +0.093, the largest cell. Its gain may be mostly crash avoidance:
    lifting 13 zero-score runs to around the mean gives about +0.09
    (our arithmetic). No inter-task row clears a sign test (13 vs 6:
    p ≈ 0.17). The intra-task signs do (22 vs 5: p ≈ 0.002), but Kimi's
    17 vs 10 does not (p ≈ 0.25).
  - **Harness "stability".** The 0.014–0.055 avg@3 gains come from 3
    rollouts per cell with no intervals, and the category table reverses
    them. "Improve stability" is not shown statistically.
- **No uncertainty anywhere.** No result reports SEs, CIs or significance
  tests, despite k = 3 and their own finding that run-to-run variation is
  the main separator. Under [[concepts/pass-at-k]], most model contrasts
  within the 0.65–0.68 tier are unresolved.
- **The process scores are verifier-trace proxies, and the authors say
  so.** C3 rewards a short monotone run. C2 for 22 runs is a
  reconstruction. C1's high-water mark ignores whether a peak survived to
  the end. Still, being deterministic and judge-free, the scores are
  reusable for this graph's own logs (see Follow-up).
- **The erasure counterfactual is a good design, with confounds.**
  Erasure removes chat context, notes *and* code comments, and the
  re-initialized arm also gets a fresh context window. So "experience"
  is confounded with context freshness. That is the flush effect
  cai2026forkandflush measured. It would push the gap toward zero, which
  makes the positive gaps conservative, but it also muddies the negative
  "anchoring" cases. Only one next commit is scored.
- **Evaluation-hacking is LLM-counted and enters aggregates
  unremoved.** Only the novel label was manually reviewed, so the 16
  hacks are Opus-4.8 judgments. Opus-4.8 also judges Opus-4.7, a
  same-family judge. The Gemini SHA-256 exploit came through the verifier
  as an apparent gain. The paper does not say whether it was removed from
  Gemini's transfer numbers. If not, it adds +0.033 to a 19-target mean
  of +0.003 (our arithmetic).
- **An incidental datum for the noise band.** In App. J.2, DeepSeek's
  self-written lessons from data_select_ifeval say that a 50-sample eval
  (stderr ≈ 0.07) made the agent chase a false 0.52 vs 0.48 "improvement"
  for 6 of 15 rounds. The full 541-prompt eval (stderr 0.021) showed every
  fine-tune was below baseline. The lessons propose a "kill-switch": after
  4 consecutive rounds within about 2 stderr and below baseline, stop and
  question assumptions. That is a noise-banded redirect trigger written by
  an agent. It is a single model-written narrative that the authors do not
  verify, but it is closer to redirect-before-halt than the hook is.

## Trust signals

- **Credibility:** 3.
  - For: an industrial lab (Meituan LongCat) with UCAS. 756 rollouts at
    real budgets on a public benchmark (AutoLab). Deterministic,
    documented process metrics with full formulas (App. C–D). A complete
    task instruction, lesson file and judge rubric are printed. The
    authors concede limitations candidly (C3 exposure, reconstruction,
    benchmark dependence). A project page with a trajectories section.
  - Against: not peer reviewed. No code repository found; the project page
    links only the paper and AutoLab. No uncertainty quantification
    anywhere. The novelty and hack labels come from a same-family LLM
    judge. The Auto Harness is a "preliminary" one-shot with an in-sample
    headline. The authors' own model (LongCat-2.0) is in the comparison,
    though it ranks sixth, so self-promotion is not evident.
  - Not 4 because there is no code and no intervals. Not 2 because the
    scale, the documented metrics and the candour are above most preprints
    in this graph.

## Follow-up

- **Relevance:** 3. Useful prior art on the active redirect-before-halt
  and memory themes, but it moves no concept. The redirect element is a
  scheduled prompt nudge with one anecdote, and the experience-reuse
  results lack intervals. The best transferable pieces are the
  memory-erasure counterfactual, the deterministic C1–C3 trace metrics, and
  more datapoints that prompt-level controllers are untested or weak.
- **Correction for the digest entry.** The five-commit hook is a fixed
  schedule that *asks* the agent to self-assess, inside a run that is told
  not to stop. It was found by a 4-round Opus-4.8 optimizer and bundled
  with two other rules, and it was never ablated. It is a weaker
  precedent than xu2026conflictguide or li2026humanize, not a new one.
- **For `/iterate` redirect-before-halt.** Nothing here changes the held
  design: a controller-enforced redirect, fired on a noise-banded stall
  counter. The paper is one more convergent design voice. It adds two
  specifics: protect the best verified state before any structural move
  ("restore your best"), and fire on stall, not on a commit count.
- **Reusable instrument.** C3's dip-episode recovery credit and the
  peak-retention ratio can be computed from `/iterate`'s metric log at no
  cost. Report them next to the champion score.
- **Candidates** (cited, not in the graph):
  - Xu et al. 2026, AutoLab (arXiv 2606.05080): the benchmark itself.
  - Tang & Yang 2026, "AI research agents narrow scientific exploration"
    (arXiv 2605.27905).
  - Antoniades et al. 2026, Heuresis (arXiv 2606.25198).
  - Chen et al. 2026, Co-Harness (arXiv 2607.22688).
  - Lee et al. 2026, Meta-Harness (arXiv 2603.28052).
