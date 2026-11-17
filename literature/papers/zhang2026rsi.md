---
kind: paper
title: "The RSI Tax: When Recursive Self-Improvement Is Not Worth the Cost"
authors: ["Yitong Zhang", "Ang Li", "Yisen Wang"]
institutions: ["Peking University (School of Physics)", "Peking University (State Key Lab of General Artificial Intelligence, School of Intelligence Science and Technology)"]
year: 2026
venue: "arXiv 2611.13524 (cs.LG), v1 2026-11-10, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.13524"
code_url: null
citations: null
source: "raw/papers/zhang2026rsi.pdf"
added: "2026-11-17"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/spend-forecast-calibration]]"
tags: ["recursive-self-improvement", "rsi-tax", "end-to-end-cost", "token-accounting", "streaming-evaluation", "harness-evolution", "skill-evolution", "memory", "bounded-note", "dynamic-cheatsheet", "meta-harness", "evoskill", "rrsi", "reasoningbank", "verification-cost", "frontier-refresh", "subsequent-use", "decision-invariant-evaluation", "single-run", "no-ci", "terminal-bench", "swe-chain-evo"]
---

# The RSI Tax: When Recursive Self-Improvement Is Not Worth the Cost

## TL;DR

The paper counts the tokens a self-improving agent spends *on improving
itself* (the "RSI Tax") alongside the tokens it spends doing the tasks. It
runs this accounting on continual task streams where each task is scored
before its feedback can be used. Six methods (Static, ReasoningBank,
Meta-Harness, EvoSkill, RRSI and the authors' bounded-note baseline SRN)
run on two models (DeepSeek V4.1 Flash, GPT-6 Luna) and two streams (105
SWE-Chain-Evo positions in 7 chains; 89 Terminal-Bench 2.1 tasks in one
order). Three results matter here:

1. **The cost result is large and robust.** Meta-Harness, EvoSkill and RRSI
   spend more on self-improvement than on the tasks in every setting (Tax
   rate 109–393% of task tokens). SRN's Tax is under 0.4%. Mean total-token
   savings of SRN: 65.6% vs Meta-Harness, 71.0% vs EvoSkill, 68.2% vs RRSI.
   Those are 2–4× gaps, which no plausible run-to-run noise erases.
2. **The accuracy result is "not visibly worse" from one run, not a tested
   equivalence.** SRN has the highest observed accuracy in 3 of 4 settings
   and ties EvoSkill in the fourth. Its margins over the best RSI method are
   3, 1, 1 and 0 tasks. There are no seeds, CIs or tests.
3. **Verification is most of the Tax.** Re-scoring incumbents and candidates
   on support tasks is 74.8% (Meta-Harness), 78.1% (EvoSkill) and 93.6%
   (RRSI) of the Tax. EvoSkill's rolling re-scoring of its old frontier
   alone (497M tokens) costs 2.7× the 182M spent serving the 89 terminal
   tasks. That refresh is the authors' streaming adaptation, not EvoSkill's
   original fixed-validation design.

## Claims

- End-to-end cost of an RSI method is N_task + N_RSI. The Tax counts every
  update attempt, including rejected ones. Omitting it can reverse a cost
  comparison: on DeepSeek/terminal, Meta-Harness uses fewer task tokens than
  SRN (184.13M vs 214.37M) but 762.57M in total vs SRN's 214.65M.
- A one-note, no-search baseline is a counterexample to the need for a
  costly update procedure, on these streams (contribution 2, worded with
  care in the paper).
- The Tax decomposes into Verify / Explore / Write (+ Other), and each cost
  item can be linked to whether its state or artifact is used in a later
  task. 23.7–63.6% of non-SRN Tax is associated with state never used later.
- Cost-aware selection (HarnessPro, RRSI) prices *execution* cost of the
  selected harness but not the cost of *obtaining* updates. The authors call
  for update policies that jointly optimize the Tax and expected execution
  cost over the remaining stream. They do not build or test one.

## Methods

- **Protocol.** Task t runs under state s_t; outcome y_t ∈ {0,1} is recorded
  before any update uses it. Updates see public instructions, visible
  trajectories and scalar outcomes only (no reference solutions, hidden
  tests or future tasks). Re-executing completed tasks for validation is
  charged to the Tax and never replaces the original outcome.
- **Streams.** SWE-Chain-Evo: a *selected* subset of 7 of the 50 EvoArena
  chains ([[literature/papers/xu2026evoarena]]): Click, CoreDNS, NATS,
  Fiber, Loki, Istio, Prometheus, 15 positions each. State resets per chain;
  every task starts from its prescribed snapshot. The selection criterion is
  not stated. Terminal-Bench 2.1: all 89 tasks in one fixed order, state
  persists. Ordering independent tasks permits reuse but introduces no drift.
- **Shared agent.** Initial KIRA agent, high reasoning effort, nominal 262k
  context, 65k output per call, ≤100 turns, no per-task token cap. DeepSeek
  at T = 1, top-p 0.95; Luna sampling parameters unrecorded. 2,328 scored
  method–task pairs, one run each.
- **Update schedule.** SRN, Meta-Harness and EvoSkill update every 4
  completed tasks (3 opportunities per 15-task chain, 22 per terminal
  stream), with no final partial-window update. Support set = last 2 tasks
  on DeepSeek/SWE, last 4 elsewhere; the paper gives no reason for the
  difference. ReasoningBank updates per task. The paper does not say when
  RRSI updates.
- **Method adaptations (all by the authors).**
  - Meta-Harness: one native search iteration per update, smoke test plus
    support validation, deploys only on *strictly* higher mean validation
    accuracy (incumbent kept on ties). Narrower than the original
    population search.
  - EvoSkill: one iteration, frontier size 3, two samples. Frontier members
    are re-scored on each new support window (the original uses a fixed
    validation set). The proposer is denied ground truth, which the original
    may inspect.
  - RRSI: analyst, digesters, proposer, critic, validation.
  - ReasoningBank: judgment, extraction, top-bundle retrieval, no MaTTS;
    gemini-embedding-001 tokens charged to the Tax.
- **SRN.** At each update, if the window has failures, the same backbone
  rewrites one Markdown note (≤2,048 tokens, tokenizer-truncated) from the
  old note, instructions, scalar outcomes and head/tail excerpts (first 20k
  + last 10k characters) of failed traces. Any valid nonempty output is
  adopted, with no validation. The note is in every later task's context.
  The prompt (App. A.4) tells the writer to drop outdated claims, separate
  facts from hypotheses, and say that "a scalar reward of zero does not
  establish a specific root cause". No ablation of cap, prompt or window,
  and no account of how they were chosen.
- **Accounting.** Input + output tokens per recorded call, deduplicated by
  request ID. Cached input counted at par; reasoning tokens not
  double-counted. Cross-setting means weight the four settings equally, with
  token ratios formed within settings.
- **Subsequent use (Sec. 3.3, App. B.4).** Each Tax cost item, verification
  and exploration included, is attributed to the state or artifact it
  concerns. It counts as "used" if that state is used in a later task
  before reset or stream end: a harness is run, a skill is used, or a
  note or memory enters task context. The authors say the label records
  use, not effect on behaviour or accuracy, and is a share of tokens, not
  of updates.

## Results

Accuracy (%) and total tokens (M), one run per cell (Table 2/4). Task counts
are our conversion (n = 105 SWE, 89 terminal).

| Method | DS/SWE | Luna/SWE | DS/TB | Luna/TB | Tax rate (%) DS/SWE, Luna/SWE, DS/TB, Luna/TB |
|---|---|---|---|---|---|
| Static | 43.8 (46) · 488 | 38.1 (40) · 180 | 69.7 (62) · 186 | 66.3 (59) · 79 | 0 |
| ReasoningBank | 40.0 (42) · 521 | 37.1 (39) · 191 | 69.7 (62) · 199 | 67.4 (60) · 85 | 4.8, 6.0, 4.5, 7.4 |
| Meta-Harness | 45.7 (48) · 977 | 40.0 (42) · 496 | 77.5 (69) · 763 | 65.2 (58) · 318 | 109, 177, 314, 329 |
| EvoSkill | 41.0 (43) · 1,167 | 40.0 (42) · 612 | 71.9 (64) · 899 | 68.5 (61) · 359 | 144, 213, 393, 359 |
| RRSI | 44.8 (47) · 1,093 | 39.0 (41) · 574 | 74.2 (66) · 606 | 61.8 (55) · 454 | 126, 213, 223, 347 |
| SRN | **48.6 (51)** · 493 | **41.0 (43)** · **157** | **78.7 (70)** · 215 | **68.5 (61)** · 87 | 0.04, 0.25, 0.13, 0.36 |

- **Equal-weight means.** SRN 59.18%, Meta-Harness 57.10, EvoSkill 55.35,
  RRSI 54.94, Static 54.46, ReasoningBank 53.56. SRN vs Static: +4.71 pp
  for +3.6% tokens. SRN vs RRSI: +4.24 pp.
- **Self-improvement sometimes lost to doing nothing.** On Luna/terminal,
  Meta-Harness (65.2) and RRSI (61.8) are below Static (66.3).
  ReasoningBank is at or below Static in 2 of 4 settings.
- **SRN changes execution cost both ways.** On Luna/SWE its task tokens fall
  to 156.67M from Static's 179.69M, so total cost is 12.6% *below* Static
  with +2.9 pp. On DS/terminal it uses 15.5% more task tokens than Static
  for +9.0 pp (8 tasks). SRN's update calls total 1.21M over all four
  settings.
- **Tax by function, four-setting sums (Table 5a).**
  - Meta-Harness 1,648.86M: Verify 74.8%, Explore 18.0%, Write 5.6%,
    Other 1.6%.
  - EvoSkill 2,103.05M: Verify 78.1%, Explore 21.7% (455M extra rollouts
    and proposals), Write 0.2% (4.59M skill materialization).
  - RRSI 1,770.50M: Verify 93.6%.
  - ReasoningBank 48.98M: Explore 50.3%, Write 49.8%.
  - About 98.1% of Meta-Harness and EvoSkill Tax is input tokens.
- **Stage detail.** Meta-Harness re-validates its incumbent every update.
  Incumbent validation (615M over four settings) costs more than the
  proposal sessions themselves (415M). EvoSkill's old-frontier refresh is
  1,142M of its 2,103M Tax.
- **No-subsequent-use share of the Tax (Table 5b).** Meta-Harness 52.78%
  (564.85M of it verification), ReasoningBank 63.57%, EvoSkill 37.91%,
  RRSI 23.70%, SRN 0%. A narrower predicate for ReasoningBank (bundles
  never retrieved despite remaining tasks) gives 53.1–61.5%.
- **Trace diagnostics (App. D, single events; the paper warns prevalence
  cannot be read from them).** *Proposal-free search*: an EvoSkill round
  spent 58.2M tokens (44.9M frontier refresh, 13.3M on two samples that both
  passed) and proposed nothing. *Decision-invariant evaluation*: a
  Meta-Harness incumbent scored 100% under strict `>`, yet the candidate got
  604k tokens of evaluation that could not change acceptance. *Unretrieved
  memory*: a 242k-token ReasoningBank bundle went unretrieved in 34 tasks.

## Critique / open questions

- **"Matches or exceeds" is an observed ranking, not equivalence.** One run
  per method-setting, DeepSeek at T = 1, one terminal ordering, no CIs,
  no seeds, no paired test. SRN's margins over the best RSI arm are 3/105,
  1/105, 1/89 and 0/89 tasks. A binomial 95% half-width is about ±9.6 pp at
  n = 105 and ±8.5 pp at n = 89, and the SWE positions are clustered in 7
  chains, so the effective n is smaller (our arithmetic).
  wang2026rethinking measured ~2.5 pp run-to-run spread on the same
  Terminal-Bench 2.1. Even SRN's largest gain over Static (+8 tasks,
  DS/terminal) gives an unpaired z ≈ 1.4 (our arithmetic). The authors say
  "observed accuracy" throughout and list repeated runs under limitations.
  The abstract-level claim is therefore "SRN is not detectably worse and
  costs 3× less". The cost half carries the paper.
- **The baseline is the authors', the competitors are the authors'
  adaptations.** SRN's 2,048-token cap, excerpt window and prompt have no
  ablation. The paper does not say whether they were set on these streams.
  The prompts are stream-specific ("a terminal agent learning across ...
  independent tasks"). The DeepSeek/SWE support size (2 instead of 4) is
  unexplained. On the other side, every RSI method runs at one iteration
  per update with small frontiers. Meta-Harness gets a strict-`>` selector
  that the original does not use. EvoSkill gets a rolling refresh the
  original lacks, and that refresh is the largest single item in its Tax.
  The verification share is partly a property of these adaptations.
  Neither side is tuned, and only one side was designed by the authors.
- **The horizon is short, which penalizes amortization.** State resets
  every 15 SWE tasks: three updates, the last one serving 3 tasks. Methods
  designed to optimize once offline and deploy for long (EvoSkill, RRSI)
  are run as per-window online learners. The authors grant that "repeated
  reuse may justify a larger initial investment". The 89-task terminal
  stream is the only longer horizon, and the ranking there is the same.
  Whether the result holds at 1,000 tasks is untested.
- **"Never used later" is a structural label, and it is not waste.** Use
  means the artifact entered a later context or ran. SRN scores 100% by
  construction, since its note is always injected and always adopted. For a
  validated method, most "no use" is the cost of *rejected* candidates,
  which is what a validation gate exists to produce (Meta-Harness: 565M of
  verification on unused candidates). The paper does not claim these calls
  are removable and says so. The digest phrase "never used later" is
  accurate, but it reads as waste and should not be taken that way.
- **Tokens are not dollars.** 98% of the Tax is input. Cached input is
  counted at par. Re-scoring the same support tasks is likely highly
  cacheable, so the billed ratio may be much smaller. The paper cites
  Weinberger & Hozez ("Token reduction is not cost reduction") and reports
  no priced figures. See kumar2026turncost in
  [[concepts/spend-forecast-calibration]] (flat pricing overstates the
  cached bill 6.5–7.8×).
- **No released code, logs or traces**, so neither the subsequent-use
  attribution nor the token totals can be checked.
- **What the paper does not show:** that SRN beats Static reliably
  (+4.7 pp mean, one run per setting). Which SRN design choice matters
  (cap, failure-only routing, no validation). That cheaper verification
  (e.g. adaptive support selection, deferring frontier refresh) keeps the
  RSI methods' accuracy. Any effect on accuracy of the artifacts that were
  used.
- **SRN as written is the paper's most portable artifact.** It is a
  dynamic-cheatsheet-style note with failure-only routing and no admission
  test. nakayashiki2026lessons measured exactly that failure-routed write
  as the channel through which chance failures become rules. SRN's prompt
  hedges this ("a scalar reward of zero does not establish a specific root
  cause"), but nothing tests it.

## Trust signals

- **Credibility:** 2. For: a clear, well-specified accounting framework.
  Stage-level token tables to six decimals reconcile with the totals. The
  authors list their own adaptations of each baseline and hedge throughout
  ("observed", "descriptive", "does not establish"). Against: a
  three-author Peking University preprint, one week old, no citations. No
  code, logs or traces released. One run per cell with no uncertainty.
  The authors built the baseline and adapted all competitors, and the SWE
  chain subset was selected by an unstated rule. Not 3: no peer review, no
  artifacts, and the headline accuracy comparison has no error bars. Not 1:
  named institutional authorship, and the cost result is large enough to
  survive the missing variance.

## Follow-up

- **Relevance:** 4. Material new evidence for
  [[concepts/budget-as-ceiling]]: a matched-*accuracy* comparison that
  charges the improvement loop's own spend, showing that verification
  (re-scoring incumbents and frontiers) is most of the bill. It also adds
  an end-to-end cost argument for [[concepts/skill-library-lifecycle]]: a
  validated skill frontier vs one unvalidated bounded note. The 11-02/11-16
  sections there price admitted rules, but not the admission process.
  It is the third "simple loop does as well at much lower cost" source after
  chen2026autoresearch and wang2026rethinking. Not 5: one run per cell, no
  code, and the competitors run in adapted configurations.
- **For `/iterate` (our inference; the paper tests no keep rule or halt).**
  - Report a chain's spend split into candidate runs vs incumbent re-runs
    and band runs. Here the re-runs were the larger share.
  - Skip a candidate evaluation that cannot change the decision, e.g. the
    incumbent already sits at the metric's ceiling under a strict-`>` rule.
  - Count the improvement loop's spend in the same ceiling as the work.
- **Candidates** (cited, not yet in the graph):
  - Hajimiri et al., "Are online skill and memory modules always worth
    their tokens?" (EMNLP 2026), a budget-constrained web-agent study.
  - Zheng et al., "ServeLearnBench" (arXiv 2610.07792): serving vs held-out
    tasks, learning calls in the resource accounting.
  - Miyai et al., "Task-CoEvolve" (arXiv 2608.20169): adaptive validation-
    task selection, the direct remedy for the verification share.
  - Pan et al., "HarnessPro" (arXiv 2610.26770); Singh et al., "Agent
    Plasticity" (arXiv 2610.08902); Chen et al., "Are self-evolving agents
    continual learners?" (arXiv 2610.25858); Weinberger & Hozez, "Token
    reduction is not cost reduction" (arXiv 2607.12161); Suzgun et al.,
    "Dynamic Cheatsheet" (EACL 2026), SRN's ancestor.
