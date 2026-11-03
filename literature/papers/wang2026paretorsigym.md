---
kind: paper
title: "ParetoRSI-Gym: Can AI Agents Improve Language Models Across Multiple Domains?"
authors: ["Haojin Wang", "Yangzhen Wu", "Zeyi Chen", "Jian Xie", "Jiaying Ye", "Lili Meng", "Shuyun Tang", "Shangding Gu", "Dawn Song"]
institutions: ["University of Illinois Urbana-Champaign", "University of California, Berkeley", "The Ohio State University", "University of Washington", "Google"]
year: 2026
venue: "arXiv 2610.34386 (cs.LG), v1 2026-10-25, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.34386"
code_url: "https://github.com/haojinw0027/ParetoRSI-GYM"
citations: null
source: "raw/papers/wang2026paretorsigym.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["rsi", "post-training", "data-curation", "sft", "lora", "multi-domain", "pareto", "proxy-vs-held-out", "same-benchmark-holdout", "answer-format", "proxy-best-checkpoint", "keep-best", "final-vs-best", "budget-unit", "decontamination", "human-baseline", "harness-ablation", "three-seeds", "claude-code", "codex", "huggingface-hub"]
---

# ParetoRSI-Gym: Can AI Agents Improve Language Models Across Multiple Domains?

## TL;DR

A benchmark for agent-run post-training data curation. Each run gives an
agent a base model (Qwen3-4B-Base, Ministral-3-3B-Base, Olmo-3-7B) and five
rounds. In each round it searches and pulls Hugging Face datasets, commits
a pool of at most 15M tokens / 40k entries, picks a parent checkpoint and a
LoRA recipe, and reads back per-item results on five **proxy** sets (math,
science, medical, finance, code). Reported scores come from a separate
**generalization** set per domain that the agent never sees. Averaged over
three base models and five domains, Claude Code + Opus-5's proxy-best
checkpoint gains **+2.3 points** (Codex + GPT-5.6 Sol +0.6, CheetahClaw +
DeepSeek-v4-flash −0.3). The gain is uneven. Two Ministral cells (medicine
+14.8, code +12.6) carry about 80% of it, and the authors say part of the
medicine gain is the model learning the answer format. For four of five
domains the "generalization" set is a disjoint split of the same benchmark,
so a format gain shows up there too.

## Claims

- Existing curation benchmarks score one target. Real post-training is
  multi-domain, and a gain in one domain must be weighed against losses in
  others (Table 1 positions it against PostTrainBench, RSIBench-Data,
  Curation-Bench, DataPrep-Bench, Agent² RL-Bench).
- Held-out generalization benchmarks "separate capability gains from
  overfitting to the scores the agent optimizes against".
- Claude Code + Opus-5 does best: +2.3 points (+9.2% relative) at the
  proxy-best checkpoint. The largest single gain is +63% relative on
  medicine for Ministral-3-3B (23.65 → 38.46).
- Agents peak before they stop. In 7 of 9 configurations the proxy-best
  round is not the last. In 24 of 45 (configuration × domain) cells the
  final checkpoint is below the proxy-best one.
- No single round is best for every domain. In 43 of 45 cells the per-domain
  best round is not the proxy-best round, at a cost of 2.0 points on average
  (up to 6.9).
- "The harness matters as much as the model": with the backbone fixed to
  DeepSeek V4.1 Flash Max, the proxy-best gain ranges from +4.3 (Codex
  harness) to +0.2 (CheetahClaw).
- Each agent has its own dataset taste: Jaccard 0.24 between two runs of the
  same agent, 0.07 between different agents. Within-agent overlap does not
  rise when the base model is shared (0.23 vs 0.25), so agents do not adapt
  to the target model.
- Exploration decays: new datasets entering the pool fall from about 5–6 in
  round 1 to about 0 by round 5 (Fig. 3c).
- A human doing the commit step (agent still searches and pulls) "matches or
  exceeds the strongest agent on every base model".

## Methods

- **Loop (Alg. 1).** State carried across rounds: model pool M, training
  pool P (only grows), and written principles R (a SKILL.md capped at 8,000
  tokens of active bullets). Per round: ACQUIRE (SEARCH / SELECT / PULL on
  the Hub), COMMIT (parent checkpoint, training set, recipe), train, score
  on the proxy suite, rewrite R. All five rounds run in one resumed agent
  session. The run returns T_general(M_T) and T_general(M_t*), where
  t* = argmax of the mean proxy score. The generalization suite is "scored
  once, after the run".
- **Suites (Table 2).** Proxy / generalization pairs: MATH500 ID / OOD,
  GPQA ID / OOD, MedBullets ID / OOD, SuperGPQA Finance ID / OOD, MBPP+ /
  HumanEval+. "Each pair targets the same capability but shares no item."
  Only code uses a different benchmark. The prompt (App. B) names proxy
  files of 200 / 200 / 150 / 200 / 200 items. Generalization-set sizes and
  how the ID/OOD splits were drawn are not stated.
- **Training.** LoRA SFT, AdamW, one epoch over the committed pool. The
  agent sets lr, LoRA rank, batch size, max length and warmup within fixed
  ranges. ≤ 15M tokens and ≤ 40k entries per round. Training and evaluation
  compute is paid by the harness and identical across systems.
- **Agent budget.** About 30 minutes and $3 of the agent's own model spend
  per round. A 12-minute planning deadline is enforced: read routes refuse
  until `plan.json` exists. Realized agent tokens or dollars are not
  reported.
- **Agents.** Claude Code + Opus-5, Codex + GPT-5.6 Sol (medium effort),
  CheetahClaw + deepseek-v4-flash. All tool calls auto-approved. §3.2 says
  network access is restricted to the Hugging Face Hub. The verbatim prompt
  says "Native `WebSearch` is still yours for DISCOVERY", and the App. C
  transcript shows a web search.
- **Contamination controls.** 13-gram decontamination of every registered
  row against every proxy *and* generalization item. Rejected rows are
  returned to the agent "with reasons". The Hub repos that publish the proxy
  sets are blocked. Searches containing proxy text are "recorded as
  violations". Gold answers are never shown. The agent may read every proxy
  question and its model's wrong answers. It may also register rows it
  wrote itself (`derived` or `author`).
- **Principle file checks.** Each bullet is tagged `prior`, `rN:axis+pts`
  or `retired:rN`. The harness checks cited score changes against the record
  and rejects the round if they disagree. A skill that fails validation is
  "silently reverted".
- **Repeats.** Claude Code and Codex: three training seeds per
  configuration, mean reported. CheetahClaw: one run per base model. The
  run count is stated three ways: "nine independent curation runs" (§1),
  three seeds per proprietary configuration (§3.3, implying 21 runs), and
  "50 rounds we logged" (App. C, i.e. 10 runs).
- **Analyses.** Per-round proxy vs generalization trend for one
  configuration (Qwen3-4B, Codex). Harness ablation with a fixed DeepSeek
  backbone on Qwen3-4B, one run each. Dataset overlap and pool composition.
  Three case studies. A human-supervised baseline (one run per base model).
  Seed SDs for Qwen3-4B only (App. G).

## Results

- **Main table (Table 3), change vs M_0 averaged over 15 cells (our
  arithmetic, checks the paper's figures):**

  | System | Proxy-best | Final |
  |---|---|---|
  | Claude Code + Opus-5 | +2.3 | +1.0 |
  | Codex + GPT-5.6 Sol | +0.6 | +0.3 |
  | CheetahClaw + ds-v4-flash | −0.3 | −0.3 |

- **Per base model, Claude Code proxy-best (our arithmetic):** Qwen3-4B
  +1.6, Ministral-3-3B +5.3, Olmo-3-7B −0.0. Ministral medicine (+14.8) and
  code (+12.6) sum to 27.4 of the 34.5 summed points. Without those two
  cells the mean over the other 13 is +0.55. Treating base model as the
  unit, the +2.3 has an SE of about 1.6.
- **Seeds (Table 8, Qwen3-4B only).** Per-domain SDs mostly 1–3 points.
  Largest: Claude Code math Final ±5.62, Codex code ±6.88. No CIs or tests
  on Table 3. The only interval in the paper is Fig. 7 (one run, 95% paired
  item bootstrap): +3.7 on math is significant there, +3.2 on finance is not.
- **Math regresses for most systems.** Claude Code math Final is −6.7, −4.9
  and −2.4 across the three base models. CheetahClaw on Olmo loses 16.2. The
  case study (App. D.1, one Claude Code run on Olmo) traces a −20.1-point
  round-1 drop to short-answer data. The agent filtered NuminaMath to
  "concise math CoT (≤5k chars) to finish solutions within budget". The
  next round, with long `<think>` traces, recovered math "with a seventh of
  the entries".
- **Proxy vs generalization.** Table 3 has no proxy columns. The only
  per-round comparison is Fig. 2 (Qwen3-4B, Codex) and Fig. 6 (one math
  run). In Fig. 2 the two move together, and on code the generalization
  gain (~+11 at round 2) is *larger* than the proxy gain (≤ 3). Code is the
  one domain with a different held-out benchmark.
- **Harness ablation (Fig. 5, one run each, Qwen3-4B).** Codex harness +4.3,
  Claude Code +2.6, CheetahClaw +0.2. Code moves +13.2, +10.5 and +8.8
  (read off the plot). Both of the stronger harnesses hit their proxy best
  at **round 1**.
- **Restarts (Fig. 4).** Claude Code starts 36% of later rounds from M_0,
  18% from an earlier round, 46% from the previous one. Codex 47 / 14 / 39.
  CheetahClaw 67% from M_0 and never branches back to an older round.
- **No inspection.** "The zero in it holds for 49 of the 50 rounds we
  logged": agents fixed pools of tens of thousands of rows without opening
  one.
- **Pareto round (App. D.3, one run).** Claude Code on Ministral: round 2
  gained math and code and lost 7.8 on medicine. Round 3 branched back to
  round 1, and its checkpoint is ≥ M_0 on every domain (medicine +17.4,
  code +14.4). Rounds 4–5 kept medicine and code and gave back math.
- **Human baseline (Table 7, one run each; means are our arithmetic).**
  Qwen3-4B: human 46.5 vs Claude Code proxy-best 46.9 (M_0 45.2).
  Ministral: 39.1 vs 32.2 (M_0 26.9), driven by math 65.67 vs 34.64.
  Olmo: 39.7 vs Codex 38.8 (M_0 38.5). Science falls under the human on
  Qwen (−6.1) and Ministral (−8.0). Who the human was and how long they
  spent are not stated.

## Critique / open questions

- **Against the digest framing: below framing (moderate).** The digest
  line is "Generalization benchmarks separate gains from proxy overfitting;
  Claude Code + Opus-5 gains +2.3 points." The +2.3 is arithmetically
  right. The rest overstates:
  - **The separation is designed, not measured.** No table gives proxy
    scores per system, so no proxy-to-generalization gap is reported for
    any agent. The two plotted runs show no overfitting event, and on code
    the gap runs the "wrong" way. No planted overfit arm checks that the
    held-out set would catch one.
  - **Four of five held-out sets are the same benchmark.** They are
    disjoint items, so they catch item memorization, not benchmark-style
    gains. The paper's own example is the headline cell. Part of the +63%
    Ministral medicine gain is "the model learning the answer format". That
    gain passed the held-out check, because the held-out set uses the same
    format. The agent's prompt says the opposite: "Teaching the model to
    emit the answer format an axis happens to want will move that axis's
    score here and buy nothing where the run is decided." For 4 of 5 axes
    that sentence is false. This is [[literature/papers/ding2026harnessdelta]]'s
    "a same-benchmark holdout passes the artifact", now in a weight-updating
    loop.
  - **The +2.3 is thin.** It rests on 3 seeds for two systems and 1 run for
    the third, with no CI. One base model carries it (Ministral +5.3; Olmo
    −0.0), and two cells carry about 80% of the summed gain. Across base
    models it is about 1.5 SE from zero (our arithmetic).
  - **Omitted:** Codex +0.6 and CheetahClaw −0.3; math regresses for most
    systems; a single human doing the commit step matches or beats every
    agent; and the internal inconsistencies below.
- **Same first author as RSIHub? No.** [[literature/papers/wang2026rsihub]]
  is by Zimu Wang et al. (Berkeley, SJTU, BIGAI and others). This paper is
  by Haojin Wang et al. (UIUC, Berkeley). No author overlaps, and this
  paper does not cite RSIHub. The shared surname and Berkeley affiliation are
  coincidence as far as the PDFs show. The comparison is still useful,
  because both are RSI evaluation frames from late October 2026:
  - **Single-draw baseline: partly avoided.** M_0 is a fixed model scored
    once, so it has no rerun noise like RSIHub's stochastic agent seed.
    (Decoding settings are not stated, so "deterministic" is our
    assumption.) But the CheetahClaw arm, the harness ablation and the
    human baseline are each one run, with seed SDs of 1–7 points per domain.
  - **Pooled splits: avoided in the headline, recurs in the split design.**
    Table 3 reports generalization scores only, with no train/gate/sealed
    sum like RSIHub's 13.5 pp. But RSIHub's sealed split was drawn from the
    same benchmark, and so are four of five held-out sets here.
  - **Budget: better matched.** Training compute is fixed by the harness,
    and agent spend is capped at $3 and ~30 minutes per round for every
    system. That is dollar-matched, not token-matched: $3 buys far more
    DeepSeek tokens than Opus-5 tokens. Realized spend is not reported.
  - **Missing same-budget arm: recurs in a new form.** RSIHub had no
    best-of-K arm. Here there is no non-agent curation arm at the same
    15M-token budget, such as a fixed public SFT mixture or a random draw
    from the same Hub sources. There is also no one-round arm. So the
    paper cannot say how much of +2.3 needs an agent, or needs five rounds
    (cf. [[literature/papers/ellington2026automatic]]'s one-shot arm and
    [[literature/papers/chen2026auditing]]'s product-vs-process split).
- **Does proxy-best selection leak? Not directly.** t* is the argmax of the
  proxy mean, and the generalization suite is scored after the run. So the
  reported proxy-best number carries no winner's curse on the held-out set.
  It is cleaner than [[literature/papers/srikanth2026recursive]], whose
  outer loop selected on its "private" score. Four smaller channels remain:
  1. **The decontamination reply.** Rows rejected for 13-gram overlap with
     *held-out* items come back to the agent with reasons. That tells it
     which sources sit near the held-out set. The paper does not say
     whether the reply separates proxy from held-out hits.
  2. **Paraphrase.** The agent reads every proxy item and may register rows
     it wrote. A paraphrase passes a 13-gram filter, and because the
     held-out set is the same benchmark, it would likely transfer. No audit
     of `derived`/`author` rows is reported, and neither is the count of
     recorded proxy-text "violations".
  3. **Web access.** The prompt grants WebSearch "for discovery", which
     contradicts §3.2's Hub-only sandbox. Rows still must come from the Hub
     or be agent-written, so web reading feeds channel 2. Cf.
     [[literature/papers/liu2026what]] on agents fetching upstream answers.
  4. **Domain Best is selected on the held-out set.** The paper labels it an
     oracle, which is correct. But the "2.0 points average cost" of handing
     back one model is a max over five noisy held-out scores. If taken per
     seed, it is also a mean of maxima. Either way it carries an upward
     max-of-noise bias the paper does not correct. "43 of 45" partly
     follows from that construction.
- **Internal inconsistencies.**
  - Fig. 2 (Qwen3-4B, Codex) shows a mean code generalization gain near +10
    at round 2 and about +5 at round 5 (read off the plot). Table 3 gives
    Final −1.2 and Domain Best +5.3 for the same cell, and Domain Best
    cannot sit below a round's mean. The five-domain mean at round 5 is
    about +1.5 in Fig. 2 and +0.3 in Table 3.
  - The run count is given three incompatible ways (see Methods).
  - Network access: Hub-only in §3.2, WebSearch allowed in the prompt.
- **"The harness matters as much as the model" is under-powered.** It is one
  run per harness on one base model. Code supplies most of each
  average, and Codex's code SD in the main runs is ±6.9. The two "ranges"
  being compared (backbones in Table 3, harnesses in Fig. 5) come from
  different designs. One reading the paper does not make: in the same
  harness, DeepSeek did at least as well as the native backbone on Qwen3-4B
  (Codex harness +4.3 vs GPT-5.6 Sol +0.3; Claude Code harness +2.6 vs
  Opus-5 +1.6). That is one run against a three-seed mean.
- **What holds.** The loop is a real HCE split in an ML training loop. It
  has a feedback set the agent can read, a held-out set it cannot, harness-
  owned training and evaluation, enforced decontamination, and an enforced
  plan-before-read deadline. "Final < proxy-best" is consistent across
  systems: returning the proxy-best round instead of the last one adds
  +1.3 points for Claude Code and +0.3 for Codex (our arithmetic). That
  argues for keep-best selection on the feedback split. The authors are
  candid where it counts: they flag the format effect, call Domain Best
  non-deployable, and report relative gains next to absolute ones.
- **Budget unit shapes content.** App. D.1 shows the token-denominated data
  budget pushing the agent toward short answers. Its stated reason was to
  fit more solutions in the budget, and math fell 20.1 points. The prompt
  also says "Aim to use the full budget: tokens you do not schedule are
  simply lost." In the one transcribed round (App. C), Codex scheduled
  10.0M of 15M tokens anyway. Both are single-run observations.

## Trust signals

- **Credibility:** 3.
  - For: UIUC, Berkeley (Dawn Song's group; corresponding author Shangding
    Gu), OSU, UW and Google. Code is named in the PDF
    (github.com/haojinw0027/ParetoRSI-GYM; not opened at ingest). The
    prompts are reproduced verbatim. Three seeds for the two proprietary
    systems, with SDs for one base model. Three base models from three
    families. A human baseline. Plain caveats on relative gains, answer
    format and the oracle row.
  - Against: not peer reviewed and nine days old. No CIs on any headline
    number. CheetahClaw, the harness ablation and the human arm are single
    runs. Fig. 2 and Table 3 disagree on one cell, and the run count and
    network policy are stated inconsistently. Four of five held-out sets
    share their benchmark with the proxy. There is no non-agent curation
    arm, and realized agent spend is not reported.
  - Not 4 because the headline sits within cross-model noise, and there are
    internal inconsistencies a reviewer would catch. Not 2 because the
    setup is concrete and released, the training budget is matched, and
    the paper's reading of its numbers is mostly cautious.

## Follow-up

- **Relevance:** 4. It is the first agent-run, weight-updating
  post-training loop in the graph (by grep) with the HCE split built into
  the benchmark. It adds three concept-level points:
  - [[concepts/hce-evaluation]]: a same-benchmark held-out set passed a
    format gain the authors themselves identified.
  - [[concepts/evolutionary-expansion]]: across 9 configurations, final <
    proxy-best, agents rarely exploit their checkpoint archive, and new
    sources dry up by round 5.
  - [[concepts/budget-as-ceiling]]: the budget's unit (training tokens)
    selected the content (short answers).

  It also bears on [[concepts/information-firewall]] (decontamination
  replies and WebSearch as channels) and [[concepts/verified-memory-writes]]
  (harness-checked `rN:axis+pts` citations in the principle file). That
  check is a mechanism with no ablation. Not 5 because no single result is
  strong enough to anchor a concept.
- **For the `hce-known-bad-preflight` proposal.** M_0 is a measured do-
  nothing floor, scored through the evaluator before the run and shown to
  the agent. That is the proposal's floor check, done. But the floor was
  itself partly a format measurement (Ministral "often does not end its
  answer with an option letter"). A known-good probe, such as gold answers
  written in the base model's own style, would have shown how much of the
  floor is extraction rather than knowledge. This supports reviewers'
  request to restore the known-good half.
- **Noise band.** For `/iterate` on SFT-style loops, our inference from
  Table 8 and Fig. 7: single-run per-domain deltas under about 3 points sit
  inside seed or item noise. Fig. 7's paired item bootstrap draws the line
  between +3.2 (not significant) and +3.7 (significant).
- **Redirect-before-halt.** Still no redirect-vs-halt arm. The only datum is
  one run (App. D.3): branching back to round 1 after a regression produced
  the run's best round. Restart rates differ sharply by agent (36–67% from
  M_0), with no comparison of outcomes.
- **Cheap local check.** From the released run records, tabulate proxy vs
  generalization per round for all runs. That gives the per-system gap the
  paper does not report, and it would settle the Fig. 2 / Table 3 mismatch.
  Then check how often t* = T.
- **Candidates** (cited, not yet in the graph):
  - Rank et al. 2026, "PostTrainBench: Can LLM agents automate LLM
    post-training?" (arXiv 2603.08640). The nearest prior benchmark; its
    seed protocol is the one used here.
  - Meng et al. 2026, "RSIBench-Data" (arXiv 2607.25886). Single-benchmark
    data RSI.
  - Kang et al. 2026, "Can generalist agents automate data curation?"
    (Curation-Bench, arXiv 2606.04261). Shares Dawn Song.
  - Wang et al. 2026a, "CurateEvo: data-curation evolving for agentic
    post-training" (arXiv 2607.06140).
  - Gu 2026, "From model scaling to system scaling: scaling the harness in
    agentic AI" (arXiv 2605.26112). The corresponding author's own harness
    argument, cited for the harness claim.
  - Hebbar et al. 2026, "SIA: self-improving AI with harness & weight
    updates" (arXiv 2605.27276).
