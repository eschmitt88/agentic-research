---
kind: paper
title: "Review Rarely Beats the Stronger Model: Separating Verification from Capability Transfer in LLM Pipelines"
authors: ["Yirui Zhang", "Yash Raj Shrestha"]
institutions: ["University of Lausanne"]
year: 2026
venue: "arXiv 2610.15897 (cs.CL), v2 2026-10-14, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.15897"
code_url: "https://github.com/Kamilaabb/verification-or-capability-transfer"
citations: null
source: "raw/papers/zhang2026review.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["llm-review", "verification", "capability-transfer", "anchoring", "deference", "conformity", "stronger-model-baseline", "equal-call-baseline", "selector", "self-review-control", "commit-first", "cost-accounting", "open-weight", "math", "mmlu-pro", "bigcodebench", "gpqa", "program-graded"]
---

# Review Rarely Beats the Stronger Model: Separating Verification from Capability Transfer in LLM Pipelines

## TL;DR

A generator G answers a question. A reviewer V reads the question and G's
full answer, keeps it if correct, otherwise fixes it, and returns the final
answer. Papers usually report the gain over G (ΔG). This paper also reports
the gain over the stronger of the two models answering alone (ΔS). Since
ΔG = γ + ΔS when the stronger model reviews (γ = capability gap), a large
ΔG can come entirely from the reviewer already being better.

Setup: 12 open-weight models (3B–72B), 48 pairs, both review directions, so
96 directions. Four program-graded domains: MATH, MMLU-Pro, BigCodeBench,
and a ReClor/FOLIO/GPQA reasoning mix. Greedy decoding, thinking off.

- Review beats the stronger model alone in **9 of 96** directions. In 37
  more it is statistically level with it, and pays 2.6–2.8× the cost.
- Stronger reviews weaker: ΔG = +7.6, ΔS = −3.1. Weaker reviews stronger:
  ΔS = ΔG = −1.6, and 22 of 48 directions end significantly below G.
- The mechanism is deference. On questions only the reviewer gets right
  alone, the final answer is right 40% of the time (stronger reviewer) and
  17% (weaker reviewer). A self-review control with the same prompt keeps
  99.8% (math) and 91% (knowledge) of those answers.
- With the same two calls, a trained selector that picks one of the two
  independent answers is +4.2 and +2.7 points above the two review
  directions, and cheaper in every pair. It is only +1.2 above the stronger
  model alone, and a label-free version is 3.7 below it.

## Claims

- Gain over the generator "confuses verification, which adds accuracy beyond
  either model alone, with capability transfer, where a stronger reviewer
  supplies answers it could have given on its own." Only a positive ΔS shows
  verification.
- "Review rarely beats the stronger model answering alone," and does worse
  the larger the gap.
- "The reason is anchoring: the reviewer mostly follows the generator,
  discarding its own correct answers."
- Review costs 2.6–2.8× the stronger model alone. A selector over the two
  independent answers is "2.7 to 4.2 points more accurate than review and
  cheaper."
- Recommendation: "Studies of verification should report the stronger model
  alone, not only the gain over the generator." Practitioners should check
  whether one call to the better model already wins, and if a second call is
  worth paying for, treat choosing between the two answers as the baseline to
  beat.

## Methods

- **Models.** Qwen3 4B/8B/32B, Qwen2.5 3B/7B/72B, Qwen2.5-Coder-7B,
  Seed-Coder-8B, Llama-3.2-3B, Llama-3.1-8B, Ministral-8B, Mistral-Small-24B.
  All instruction-tuned, served with vLLM.
- **Tasks (Table 2).** MATH levels 3–5 (2,500, Math-Verify), MMLU-Pro
  (2,500, option letter), BigCodeBench-Instruct (832, unit tests), reasoning
  (901: 500 ReClor, 203 FOLIO v2, 198 GPQA Diamond). No LLM judge anywhere.
  Unparseable counts as wrong; an unparseable review falls back to G's answer.
- **Pairs (App. A).** Per domain, models are ranked into strong, middle and
  weak tiers on a development set (1,000 questions in math and knowledge;
  all questions in code and reasoning). For each of the six tier
  combinations, the pair with the most and the pair with the fewest
  weaker-only-correct questions are taken. 12 pairs per domain. Gaps run from
  0.5 to 39.6 points. Qwen2.5-72B pairs were swapped out for compute;
  completed runs on them (131 directions total) are a robustness check.
- **Review protocol.** Zero-shot, one pass. V is told the solution "may be
  correct or incorrect"; "If the proposed solution is correct, retain it",
  otherwise correct it, and return a full solution (App. B). Greedy, thinking
  off, 4,096 output tokens. 1.7–1.8% of review outputs and 1.8–3.3% of solo
  answers hit the limit.
- **Measures.** ΔG = f̄ − ḡ, ΔS = f̄ − s̄. Headroom h is the share of
  questions only the weaker model gets right; h = (δ − γ)/2 where δ is
  disagreement. ΔG is split into V-only-correct (capability transfer),
  G-only-correct (losses) and same-outcome (what review creates or breaks).
  Five outcome levels from paired 95% bootstrap intervals: worse than G, no
  different from G, above G but below S, level with S, beyond S.
- **Statistics.** Question-level paired bootstrap (2,000 replicates),
  domains weighted equally. Gap regressions with domain intercepts, pairs
  resampled. Leave-one-model-out, pair resampling, held-out-only reruns.
- **Selector (App. C).** Both models answer alone. A per-pair classifier
  (average of an L2 logistic regression and a small gradient-boosted tree,
  5 seeds) estimates from token log-probabilities, lengths, agreement and
  question features which answer is right. It is trained on the pair's
  1,000 labelled development questions (math, knowledge) or cross-fitted
  (code, reasoning). Section 5 numbers use 1,500 held-out questions in math
  and knowledge. JEV, a closed API judge that reads both answers, is a
  reference, not a contender.
- **Cost.** c = N·(t_in + t_out), parameters times tokens. Review pays
  c_G + c_V with G's whole answer in V's input. App. D repeats with output
  tokens weighted 4×.
- **Mechanism checks (App. F).** Self-review (V shown its own solo answer in
  place of G's, same prompt). A commit-first prompt with no retain
  instruction, on four directions × 500 held-out questions. Qwen3-8B with
  thinking on (sampled, 32,768 tokens) on two math directions.

## Results

- **Outcome levels (Table 7).**

  | | worse than G | no diff from G | above G, below S | level with S | beyond S |
  |---|---|---|---|---|---|
  | Stronger reviews (48) | 1 | 5 | 22 | 19 | 1 |
  | Weaker reviews (48) | 22 | 18 | — | — | 8 |

  When the weaker model reviews, G is the stronger model, so only three
  levels exist. Eight of the 9 "beyond" directions are a weaker model
  reviewing a close stronger one, all in math and knowledge. Of 26
  weaker-reviews-stronger directions with a gap under 8 points, 7 beat S and
  7 are significantly worse.
- **Averages (Table 7).** Stronger reviews: ΔG +7.6 [7.2, 7.9], ΔS −3.1
  [−3.5, −2.7]. By domain ΔS is −4.5 math, −1.7 knowledge, −3.5 code, −2.7
  reasoning. Weaker reviews: −1.6 [−1.8, −1.3]; knowledge is level (+0.1).
  Code is worst: a stronger reviewer lifts G by only 2.7 and no weaker
  reviewer improves G. The headline example: Qwen3-8B reviewing Llama-3.1-8B
  in math gives ΔG = +33.2 and ΔS = −6.4.
- **Gap, not headroom.** Per 10 points of gap, ΔS falls 1.8 [1.1, 3.0]
  (stronger reviews) and 1.5 [0.7, 2.5] (weaker). Headroom held fixed has no
  detectable effect. The two are correlated by construction (r = −0.73).
- **The room review leaves unused.** An oracle choice between the two solo
  answers is 7.5 [7.2, 7.8] above S. After review, an oracle over G's answer
  and the final answer is only +0.6 above S (stronger reviews) and +2.8
  (weaker). In 16 of 48 stronger-reviews directions that oracle is below S.
- **Deference (Fig. 4b, App. F).** On only-G-right questions the final
  answer stays right 76% / 85% (stronger / weaker reviewer). On only-V-right
  questions it is right 40% / 17%. 53% / 65% of the lost answers become G's
  wrong answer. Solo answers differ in correctness on 25.7% of questions;
  final vs G differ on 15.0% / 7.2%. The final answer equals G's on 57.7% of
  questions, against 41.8% for V's solo answer (stronger reviews). In code
  the reviewer's solo program never equals G's, yet its final program does
  on 21% of questions.
- **Decomposition (Fig. 4c).** The stronger reviewer's +7.6 is +8.4 from
  V-only questions, −1.8 from G-only, +0.9 elsewhere: almost all capability
  transfer. Review creates little where models agree: +0.9 / −0.5.
- **Self-review control.** Same prompt, V's own answer in G's place. Of
  answers only the reviewer had right, self-review kept 467/468 (math) and
  449/493 (knowledge); reviewing G kept 306 and 218. Accuracy under
  self-review stays within five questions of solo. So the second pass does
  not cause the losses; seeing G's answer does.
- **Commit-first prompt.** Of 421 reviewer-only right answers, the new
  prompt kept 231 (55%), the standard prompt 256 (61%), self-review 404
  (96%). The reviewer often skipped committing (in math, 211 and 310 of 500).
  When it committed and was right against a wrong G, it stayed right 143 of
  144 times.
- **Thinking on (Qwen3-8B, 500 MATH).** Review still ends 2.8 [1.0, 4.6]
  below the reviewer alone when reviewing Llama-3.1-8B and 3.0 [0.8, 5.2]
  when reviewing Qwen3-32B. Against the weaker G it now keeps 93% of its own
  right answers (77% thinking off). Against the stronger G it still loses
  about as many (21 vs 20), 14 of them to G's answer.
- **Cost and selection (Table 8, Fig. 5).** Review costs 2.8× S for −3.0
  and 2.6× for −1.6. The selector costs 1.9× and gains +1.2 [0.9, 1.4]: above
  S in 19 pairs, below in none, about 15% of the 7.6-point oracle room. It
  is level in code (+0.5 [−0.1, 1.0]). Against review it is +4.2 [3.8, 4.6]
  (higher in 34 of 48, lower in 0) at 0.69× cost, and +2.7 [2.4, 3.1] (28
  higher, 0 lower) at 0.82×. With output weighted 4×, the cost ratios become
  0.89 and 0.98. JEV is +4.1 above S for $0.04–$0.21 per 1,000 questions.
- **Label-free selector.** Picking the higher mean log-probability is 3.7
  [3.4, 4.1] below S, significantly below in 24 of 48 pairs, above in none.
- **Robustness (Tables 9–10).** No average changes sign when any one model
  is left out. Pair resampling widens intervals but keeps every sign.

## Critique / open questions

- **Against the digest framing:** AT. Every number in the digest entry is in
  the abstract and checks against the body: 48 pairs, 9 of 96, worse with
  gap, 2.6–2.8× cost, anchoring, selector beats review. The paper is careful,
  and the claims are stated at the right scope. Three omissions matter for
  how the hub uses it:
  1. *"A selector using the same two calls beats it"* is a per-pair trained
     classifier fitted on 1,000 labelled questions. Its edge over the
     stronger model alone is +1.2. Without labels it is 3.7 below. The
     robust result is "review loses", not "selection wins big". The authors
     argue the labels are fair because knowing which model is stronger
     needs them too. That holds for routing, but a label-free deployment
     gets the stronger model, not the selector.
  2. *Scope.* Open models up to 72B, thinking off, greedy, one review pass,
     no tools. The single thinking-on check removes most of the deference to
     a weaker G. Frontier models are untested.
  3. *"Rarely beats"* hides a split: 37 of 96 directions are level with the
     stronger model. That is a cost result, not an accuracy loss.
- **The retain instruction biases toward G.** The main prompt makes G's
  answer the default. The authors test this. The commit-first prompt does no
  better (55% vs 61% kept) and self-review keeps 96%, so the bias is not the
  whole story. But the alternative prompt ran on four directions only, and
  in math the reviewer often did not follow it.
- **Mechanism evidence is correlational plus one good control.** Self-review
  is the no-channel arm: same prompt, same selected questions, only the
  shown answer changes. That arm is clean and large (99.8% vs 65% in math).
  The App. F feature regressions (solo length, max logit) are correlations,
  as the authors say.
- **Review here has no evidence channel of its own.** The reviewer cannot
  run code or tests. In code, review does worst and no weaker reviewer helps.
  Agentic review with execution (chen2026rankevolve's implement → review →
  repair with a hidden oracle) is a different regime. Transfer to research
  agents is an inference.
- **Cost is a proxy.** N·(t_in + t_out) puts 3B and 72B on one scale. It is
  not dollars or latency, and the 4×-output variant narrows the selector's
  cost edge to near parity.
- **Development questions are in the main analysis.** Section 4 uses all
  2,500 math and knowledge questions, 1,000 of which picked the pairs. The
  held-out-only regression is unchanged (App. F), so this does not drive the
  result.
- **For the hub's live fronts.**
  - *Shared-substrate correlated error:* this is a cross-family, cross-size
    deference measurement with a no-channel control. It extends xu2026dont
    (p = 0.92) and shen2026revoked from shared stores and logged verdicts to
    plain review of a peer's answer. A stronger reviewer still loses 60% of
    the right answers only it had. Independence came from what the reviewer
    was shown, not from a different or stronger model.
  - *HCE evaluation:* a pipeline stage that adds a second model should be
    scored against its strongest single component at matched calls, not
    against its input. The identity ΔG = γ + ΔS is the same kind of
    check-the-algebra point as kim2026are.
  - *Redirect-before-halt proposal:* if the pivot cycle reads the stalled
    chain's proposals before generating its new axis, this paper predicts it
    will adopt them, even when told they may be wrong and even with a
    commit-first prompt in the same call. Generating the pivot candidate
    without the prior proposals in context, then choosing, is the
    selector-shaped design. This is our inference and agrees with
    [[literature/papers/sun2026scipivot]]'s warning about reading the
    chain's own narrative.
  - *No-improvement noise band and skill scoping:* no bearing.
- **Open:** does deference survive when the reviewer has its own evidence
  (tests, execution, retrieval)? Does thinking on remove deference to a
  weaker G in other models and domains? Neither is tested.

## Trust signals

- **Credibility:** 3.
  - For: all grading is by program, so no LLM judge sits in the measurement.
    96 directions over 12 models from 4 families and 4 domains. Paired
    bootstrap intervals throughout, plus leave-one-model-out, pair
    resampling, held-out-only reruns and 131 completed directions as checks.
    The self-review control is the right no-channel arm. Code and
    per-question results are public: the repo exists, is MIT-licensed, and
    has `results/`, `pairs/`, `protocols/`, `replicates/` and `scripts/`
    (checked 2026-10-20). Limits are stated plainly.
  - Against: not peer reviewed. Two authors from one university group. Open
    models only, thinking off, one review prompt in the main runs. The
    selector's advantage depends on labelled data per pair. Cost is a
    parameter-token proxy.
  - Not 4 because it is an unreviewed two-author preprint whose scope (small
    open models, no thinking, no tools) limits generalisation. Not 2 because
    the design, controls and release are well above the median preprint
    here.

## Follow-up

- **Relevance:** 4. Material new evidence for
  [[concepts/shared-substrate-contagion]]: deference to a peer's answer,
  measured across model families with a self-review no-channel control, and
  the structural fix (choose between independent answers rather than review)
  tested at equal calls. It also adds a baseline rule to
  [[concepts/hce-evaluation]]: credit a two-model stage only against the
  stronger model alone and an equal-call choice. Not 5 because the tasks are
  single-turn QA and code with small open models, not research loops, and
  the reviewer has no tools.
- **For [[concepts/hybrid-model-backends]].** Splitting work across a
  stronger and a weaker model pays only if the split beats one call to the
  stronger model. Here, putting the second model in a reviewer seat did not,
  at 2.6–2.8× the cost.
- **For the box.** Any review or judge step (e.g. `/elevate`'s gates, judge
  panels, an orchestrator re-reading a reader's note) should ask two
  questions. Would one call to the stronger model alone do as well? Does the
  reviewer see the first answer before forming its own? This paper says the
  second question decides most of the outcome. Not proposed here.
- **Related in the graph:** [[literature/papers/xu2026dont]],
  [[literature/papers/shen2026revoked]],
  [[literature/papers/chen2026rankevolve]],
  [[literature/papers/kim2026are]], [[literature/papers/wang2026rethinking]].
- **Candidates** (cited, not yet in the graph):
  - Xiang et al. 2026, "Cross-model LLM code review: Should you use Claude
    to review Codex or vice versa?" (arXiv 2607.21656).
  - Yang et al. 2026, "Precise but uncoupled: Reviewer precision does not
    guarantee critique uptake in multi-agent math reasoning" (arXiv
    2607.15388).
  - Gozel 2026, "Commit-first LLM judging inherits the judge's own errors"
    (arXiv 2609.00088).
  - Qu et al. 2026, "Easier to mislead than to correct: Harmful and
    beneficial revision in LLM conformity" (arXiv 2606.01637).
  - Li et al. 2026, "JEV-as-a-judge: Accept when confident, escalate when
    unsure" (arXiv 2609.26550).
  - Lu et al. 2026, "When does verification pay off?" (arXiv 2512.02304).
