---
kind: paper
title: "Scale and Selection: What Makes Automatic Harness Evolution Work for Visual-Interface Robot Agents"
authors: ["Zhijie Wei", "Ferris Tan", "Jinghui Wang"]
institutions: ["Novaxbot"]
year: 2026
venue: "arXiv 2609.39304v2 (cs.RO), 2026-10-09, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.39304"
code_url: null
citations: null
source: "raw/papers/wei2026scale.pdf"
added: "2026-10-13"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["harness-evolution", "keep-if-better", "champion-challenger", "accept-all", "acceptance-rule", "evaluation-batch-size", "rollout-scale", "overfitting", "robotics", "libero", "robocasa", "visual-interface-agent", "coding-agent-optimizer", "single-run", "binary-outcome-noise"]
---

# Scale and Selection: What Makes Automatic Harness Evolution Work for Visual-Interface Robot Agents

## TL;DR

A coding agent (Codex, "GPT-5.6 Sol") drives a simulated arm through a
browser 3D GUI and a few MCP tools. A second Codex session, the optimizer,
reads full trajectories of B rollouts and commits one bounded change to the
task agent's harness (prompt, tool semantics, observation format, control
and termination rules) per round, for T = 30 rounds. The paper varies two
things:

1. **B, the rollouts per round** (5, 10, 20, 100), under keep-if-better.
   Final-champion held-out success is 47 / 54 / 51 / 67% against a 51%
   baseline. Only B = 100 is clearly above baseline. At small B, training
   score stops ranking held-out: the B = 10 champion hits 70% training
   (7 of 10 cases) and 54% held-out.
2. **The keep rule.** "Champion–Challenger" promotes a revision only if its
   success count on the same fixed B cases strictly beats the incumbent's.
   That is plain greedy keep-if-better. Accepting every revision instead
   rises for a few rounds, then falls on training score at every B (at
   B = 100: 42 → 47 → 34 → 38% over 10 rounds).

It also reports a 2 → 37/100 training-sample gain in a 4-round RoboCasa
campaign, and 71.2% on the RoboCasa365 leaderboard for GPT-6 Astra plus the
evolved harness plus a VLA-handoff tool.

## Claims

- **Rollout scale is "a necessary condition"** for harness evolution to
  generalize. With rounds fixed, held-out success rises with B, and small
  batches overfit the training cases.
- **"The optimizer agent must not be given free rein."** Unconditional
  acceptance "degrades performance within a few rounds". Champion–Challenger
  "turns the same loop into one that improves over 30 rounds".
- Larger batches also promote more often (1, 3, 2, 8 promotions in 30
  rounds), because with 5–10 cases most challengers tie the champion
  exactly and are rejected.
- The RoboCasa-evolved harness "transfers to a stronger model": under GPT-6
  Astra it is worth 46 points (37 → 83 of 100), more than under Sol.
- The evolved harness plus Xiaomi-Robotics-1 through one `vla_run` tool is
  a "new best" on RoboCasa365, 13.1 points over Paimon-0.

## Methods

- **Task agent.** Codex with GPT-5.6 Sol at highest reasoning effort,
  30-minute wall-clock cap per rollout, binary success from each task's
  built-in detector. Model, simulator, tasks, detectors and the evaluation
  control plane are frozen.
- **Optimizer.** Same model, separate persistent session per campaign. It
  reads an evidence manifest (full per-case timelines of turns, tool calls,
  controller feedback, errors, verdicts), must find "one systemic,
  task-general failure mechanism", and lands one commit that passes the
  repo's test suite and leaves the eval plane untouched. It also sees the
  reasons for recently rejected challengers.
- **Selection (Algorithm 1).** The challenger is evaluated on the same B
  cases. It is promoted iff s(H′) > s(H). Ties lose. On promotion the
  challenger's score becomes the stored incumbent score, and the incumbent
  is never re-evaluated.
- **Splits.** Training: 10 tabletop tasks (3 robosuite, 7 LIBERO), with up
  to 10 fixed seeds each (B = 100). Held-out: 10 different LIBERO tasks × 10
  seeds = 100 rollouts, measured once per selected harness after the
  campaign.
- **Design.** One campaign per B per rule. Beyond the final champion, four
  more harnesses per campaign "chosen from promoted and rejected challengers
  at different rounds" are scored held-out (21 nodes in all). The
  unconditional campaigns were stopped after 10 rounds (B = 100) or 20
  rounds (B ≤ 20) "once the downward trend was clear". Only training
  scores are reported for them.
- **RoboCasa.** 10 kitchen tasks × 10 seeds, 4 rounds, no held-out set.
  RoboCasa365 (50 tasks) is run with the round-2 champion plus a `vla_run`
  tool, under GPT-6 Astra.

## Results

- **Table 1 (Champion–Challenger, T = 30).**

  | B | training (base → final) | promotions | held-out final | held-out 5-node mean (range) |
  |---|---|---|---|---|
  | 5 | 20 → 40% | 1 | 47% | 44.6 (35–49) |
  | 10 | 30 → 70% | 3 | 54% | 51.4 (48–54) |
  | 20 | 40 → 55% | 2 | 51% | 52.8 (50–55) |
  | 100 | 41 → 60% | 8 | **67%** | **61.0** (51–67) |

  Baseline held-out: 51%. At B = 100 all five nodes are at or above
  baseline. At B = 5 all five are below, one by 16 points.
- **Train/held-out coupling.** Across 21 nodes r = 0.60, "carried by the
  B = 100 nodes". Within small-B campaigns training score does not rank
  the nodes. B = 100 nodes sit *above* the diagonal (held-out > training),
  and the baseline is 41% on training vs 51% on held-out. So the held-out
  tasks are easier.
- **Unconditional acceptance (training only, Fig. 3).** B = 100: 42% →
  peak 47% (round 5) → 34% (round 8) → 38% (round 10). B = 20: 40 → 15%
  in 20 rounds. B = 10: 30 → 10%. B = 5: 40 → 0%.
- **What the gate filtered at B = 100.** 22 of 30 challengers scored at or
  below the champion. Over the 20 evolved nodes, promoted harnesses average
  54.0% held-out and rejected ones 50.9%.
- **RoboCasa (training sample only).** 2 → 26 → 37/100 in two promotions.
  Rounds 3–4 (28, 33) were rejected. On the same 100 cases: baseline under
  Astra 37, evolved under Sol 37, evolved under Astra 83.
- **RoboCasa365 (Table 4).** Evolved harness + Xiaomi-Robotics-1: 71.2
  overall (93.9 / 71.2 / 45.6 on atomic-seen / composite-seen /
  composite-unseen). Policy alone: 57.4. Paimon-0: 58.1. On composite-unseen
  a third of successes involve the agent intervening.

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** The
  digest calls this "a clean accept-all vs gated ablation" and says it
  "complements item 2 (which finds gating rules don't beat greedy)". Both
  readings over-reach.
  1. **Champion–Challenger *is* greedy.** It promotes iff s′ > s on a
     reused fixed set, and the stored incumbent score is never re-measured.
     That is the greedy keep-if-better baseline of
     [[literature/papers/hu2026winners]], with K = 1. The paper's contrast
     is accept-all vs greedy, a weaker baseline than anything Hu tested. It
     neither complements nor tensions Hu's finding about thresholded rules
     vs greedy. It sits one rung below it.
  2. **The ablation is not clean.**
     - One campaign per arm.
     - Accept-all arms were stopped early on an observed trend (optional
       stopping).
     - They were scored on *training only*, with no held-out number.
     - Their baselines were "evaluated separately, so starting points
       differ slightly".
     - At B = 100, accept-all ends at 38% from a 42% start, a 4-point
       drop on 100 binary rollouts. The 13-point peak-to-trough
       (47 → 34) is a max-minus-min across rounds. Either is within about
       1–2 SE (about 7 points) of an unpaired difference.

     The small-B collapses (40 → 0% at B = 5 is 2/5 → 0/5) are more
     dramatic but rest on 5–10 cases.
  3. **The 51 → 67% headline is the B = 100 Champion–Challenger campaign,
     not the rule contrast.** The digest's sentence credits the rule for it.
     The rule contrast was never measured held-out.
- **Most-favourable-cut check.**
  - 67% is the pre-designated final champion, so it is not post-hoc
    picked. It is still the maximum of its campaign's five held-out nodes
    (range 51–67, mean 61.0), from a single run.
  - "Rises as rollouts grow from 5 to 100" is non-monotone in final
    champions (47, 54, 51, 67). B = 5, 10 and 20 are all within one held-out
    SE (about 5 points at 100 rollouts) of the 51% baseline. The authors
    concede "only the B = 100 campaign produces a clear gain". The scaling
    claim is therefore one point above baseline, not a curve.
  - 100 held-out rollouts are 10 tasks × 10 seeds, so the effective sample
    is closer to 10 clusters than 100 draws. The stated precision is
    optimistic (our inference).
- **No run-to-run noise is measured.** The task agent is a stochastic
  frontier model on 30-minute rollouts, but no harness is rerun on the same
  cases. The baseline's 51% is one draw, and so is every node. Under strict
  `>` on a reused set with the incumbent's lucky score stored, the champion's
  training climb (41 → 60%) is partly selection by construction. This is
  Hu's incumbent-inflation mechanism, and the paper does not discuss it.
  The promoted-vs-rejected held-out gap (54.0 vs 50.9) is about 3 points,
  inside the noise, so it is weak evidence that the gate selects for
  transfer.
- **B confounds three things.** A larger B gives (a) more trajectories for
  the optimizer to diagnose from, (b) higher resolution on each promotion
  decision, and (c) 20× the rollout spend of B = 5. No arm separates (a)
  from (b), for example by showing 100 trajectories but selecting on 10.
  No arm holds total rollouts fixed, and there is no matched-budget search
  arm, which is [[literature/papers/wang2026rethinking]]'s required control.
  "More rollouts per decision" is the plausible mechanism, not the isolated
  one.
- **The 5-node sample is under-specified.** The four extra nodes per
  campaign were "chosen from promoted and rejected challengers at different
  rounds", with no stated rule.
- **RoboCasa claims are on the optimization sample.** There is no held-out
  set. The 2/100 baseline means the tabletop harness was simply broken in
  kitchens, so early fixes are easy. Table 3's "transfers to a stronger
  model" re-scores the *same 100 cases the harness was evolved on*. That is
  model transfer, not task transfer.
- **The RoboCasa365 result is a systems result, not an evolution result.**
  71.2 combines GPT-6 Astra, the evolved harness and a VLA-handoff tool. No
  arm runs the baseline harness + `vla_run` + Astra, so evolution's share
  of the 13.1 points is unidentified. The 10 evolution kitchen tasks may
  overlap RoboCasa365's 50 tasks, and the paper does not say. The
  comparison is a self-run frontier-model system against a published VLA
  entry.
- **What transfers to this graph.** Two qualitative points hold up.
  - Accept-all chains with no way back accumulate bad edits.
  - With 5–10 binary cases most challengers tie, so a strict `>` gate
    stalls (B = 5: 27 of 30 rejected), and the few promotions it makes
    overfit.

  Both already follow from Hu's selection model. The genuinely new setting
  is long-horizon, binary, expensive rollouts with a coding-agent optimizer
  that reads full trajectories, which is closer to `/iterate` than Hu's
  instruction rewrites. The evidence for it is a single run per arm.
- **Open.** Would a noise-aware gate (rerun spread, paired per-case
  disagreement) beat strict `>` at B = 20–100 here? Would re-scoring the
  incumbent each round (fresh rollouts of H alongside H′) remove the lucky
  stored score? Neither was tested.

## Trust signals

- **Credibility:** 2.
  - For: a clear task-disjoint held-out split that is never shown to the
    optimizer. Frozen eval plane. The authors honestly concede that only
    B = 100 is a clear gain and that RoboCasa has no held-out set.
    Trajectory figures for every arm.
  - Against:
    - Three authors at Novaxbot, a company with no track record visible in
      the paper.
    - No code, harnesses or logs released.
    - Preprint (v2), not peer reviewed.
    - One run per condition, with no seeds, CIs or rerun noise.
    - Early-stopped comparison arms scored on training only.
    - Leaderboard claim with no ablation of the evolution's contribution.
  - Not 1 because the design (fixed cases, disjoint held-out, frozen
    detectors) is sound and the core qualitative claims are internally
    consistent. Not 3 because the statistical support for every headline
    is a single campaign.

## Follow-up

- **Relevance:** 3. This is a harness-evolution loop with a coding-agent
  optimizer reading full trajectories and a keep-if-better gate on noisy,
  expensive, binary outcomes. Structurally it is close to `/iterate`. But
  its gated arm is the graph's existing default (bare `>`), and its
  baseline (accept-all) is one no concept proposes. So it corroborates
  rather than moves [[concepts/hce-evaluation]] and
  [[concepts/budget-as-ceiling]]. The evaluation-batch-size result is
  directionally useful for sizing `/iterate`'s validation split, but it
  rests on one campaign. Below the digest's 4 because the "clean ablation"
  is neither clean nor about the rule the graph is deciding.
- **For `/iterate`.**
  - The stored-champion-score pattern (Alg. 1, line 10) is the lock-in
    risk Hu describes. Re-scoring the champion alongside each challenger
    on the same cases would remove it at 2× eval cost.
  - With small binary eval sets, ties dominate under strict `>`. That is a
    stall signal, not a no-improvement signal, and the halting counter
    should not count ties at tiny n as evidence of convergence (our
    inference).
- **Candidates** (cited, not in graph):
  - Lee et al. 2026, "Meta-Harness: End-to-End Optimization of Model
    Harnesses" (arXiv 2603.28052).
  - Zhang et al. 2026, "Self-Harness: Harnesses That Improve Themselves"
    (arXiv 2606.09498).
  - Elmaaroufi et al. 2026, "RHO: Your coding agent is secretly a
    roboticist" (arXiv 2606.16458). Harness search for robotics with a
    held-out benchmark.
  - Jiang et al. 2026, "What Are We Actually Benchmarking in Robot
    Manipulation?" (arXiv 2606.04233). Claims that most LIBERO
    improvements are not statistically significant, which bears directly
    on this paper's 100-rollout held-out numbers.
  - Lee et al. 2026, "Recursive Harness Self-Improvement" (arXiv
    2607.15524).
