---
kind: paper
title: "LLM Evolutionary Search Strategies Should be Compared by Tokens"
authors: ["Tal Oved", "Roi Pony", "Oshri Naparstek", "Udi Barzelay"]
institutions: ["IBM Research"]
year: 2026
venue: "arXiv 2610.23977 (cs.NE), v1 2026-10-13, preprint (ACL-style short paper with appendices)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.23977"
code_url: null     # no release named; runs use the open SkyDiscover framework and ADRS-Bench tasks, logs not released
citations: null
source: "raw/papers/oved2026llm.pdf"
added: "2026-10-26"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/pass-at-k]]"
tags: ["compute-matching", "token-accounting", "evolutionary-search", "independent-sampling", "best-of-n", "openevolve", "evox", "adaevolve", "adrs-bench", "skydiscover", "scorer-exploit", "reward-hacking", "re-execution-audit", "order-statistics", "circle-packing", "heilbronn", "gpt-5-mini", "haiku-4.5", "gemini-2.5-flash", "budget-unit"]
---

# LLM Evolutionary Search Strategies Should be Compared by Tokens

## TL;DR

LLM evolutionary search (OpenEvolve, EvoX, AdaEvolve) is usually compared
with an independent-sampling ablation at an equal number of *proposals*.
The paper replays 600 logged runs (5 tasks × 3 strategies × 40 seeds × 200
iterations, gpt-5-mini) and re-prices every best-so-far curve in cumulative
tokens. It then compares each strategy with an 8,000-draw independent-sampling
pool at equal tokens, using exact order statistics, and re-executes winners
on the systems tasks to catch programs the scorer wrongly accepts.

Four results matter for this graph:

1. **A proposal is not a unit of compute.** One proposal costs 15.6k–38.6k
   tokens across cells (OpenEvolve 23.0k, EvoX 27.0k, AdaEvolve 33.9k on
   average) and 1.3–1.8 model calls. One fresh sample costs 4.1k–6.0k, so
   an equal-proposal sampling arm gets 4.6–7.2× less compute (mean 5.7×).
   Within one task, strategies differ by up to 1.9×. Counting model calls
   does not fix this: tokens per call still span 2.3×.
2. **The currency correction is real but small; the audit is what moved the
   headline.** After the audit, the mean evolve/sampling ratio is 1.09 at
   equal proposals and 1.07 at equal tokens. 11 of 15 pairs are already
   within 5% at equal proposals, and token matching brings in one more
   (Cloudcast·AdaEvolve 1.06 → 1.05). Sampling had saturated: its last
   doubling adds +0.00% to +3.63%. The large correction is a scorer exploit
   on transaction scheduling. A schedule placing 3 of 300 transactions
   scored 5.06× the sampling arm, and re-execution took it to 1.02. That
   one fix moves the equal-token mean from roughly 1.35 to 1.07 (our
   arithmetic from the stated pre-audit values).
3. **Where evolution pays depends on the task, and the size of the payoff
   depends on the model.** On gpt-5-mini, Heilbronn keeps 1.23–1.28 at equal
   tokens for all three strategies. The other four tasks sit at 0.99–1.05,
   and all three PRISM pairs are at 1.00. Haiku 4.5 (math tasks only)
   repeats the ordering: Heilbronn 1.17–1.26, circle packing 1.01–1.07. On
   gemini-2.5-flash (systems tasks only), evolution wins on all 9 pairs
   (1.06–1.59, mean 1.30), and token pricing barely moves that (1.31 → 1.30).
   Cloudcast goes from parity on gpt-5-mini to 1.53–1.59.
4. **Selection spreads exploits, and a cheap audit catches them.** In the
   worst transaction-scheduling seed, the top 69 distinct programs were all
   the same exploit, and over a third of candidates scored at that level.
   The audit walks each seed in order and re-executes a candidate only if
   it would beat the best program that has passed so far. That is about 6
   re-runs per seed instead of 200.

## Claims

- "LLM evolutionary search is currently evaluated by counting proposals
  … this metric hides the true cost of search." An evolve prompt carries
  the program being edited and part of the archive, and both grow during
  the run. A later proposal therefore costs more than an earlier one, and
  a method that shows more context costs more per proposal.
- "When we match token budgets … and remove scorer exploits, the
  evolutionary advantage largely vanishes." Evolution wins by ≤5% on 12 of
  15 pairs and by 23–28% on 3, all of them Heilbronn.
- "The performance margin justifying these systems is often an accounting
  artifact." Reporting in proposals "flatters evolution twice": it does not
  charge for larger prompts, and nobody checks the best program's output.
- "Check whether a task rewards search before choosing a searcher."
  Heilbronn has "a long improvement path where editing a good program keeps
  paying". On the other tasks "a fresh sample is almost as good for a
  fraction of the cost".
- The cost spread is "closer to a property of the harness than of the
  model, without being a fixed ranking". AdaEvolve is the most expensive
  strategy on every backbone. OpenEvolve is the cheapest on gpt-5-mini but
  not on Haiku Heilbronn.
- Margins are lower bounds. The sampling pool caps the window at 28–48M
  tokens, about a fifth of evolution's logged corpus, and evolution is
  still rising there while sampling has saturated.
- Practice: report tokens alongside proposals (the logs already hold them),
  audit both sides, and record audit quantities such as a coverage number
  per candidate during the run.

## Methods

- **Engine and arms.** ADRS engine on SkyDiscover. gpt-5-mini; model,
  objective and evaluator fixed. Strategies: OpenEvolve (islands +
  archive, population 40), EvoX (co-evolves its selection rule),
  AdaEvolve (adaptive). 40 seeds × 200 iterations per cell (8,000
  proposals), 15 cells, 600 runs. One cell costs 108–309M tokens (mean
  194M). Every model call logs prompt, completion and reasoning tokens.
  Every evaluated program logs score and timestamp.
- **Tasks.** Three ADRS-Bench systems tasks: PRISM (LLM-serving scheduler,
  goodput), Cloudcast (broadcast planner, transfer cost) and transaction
  scheduling (throughput). Two math constructions: circle packing in a
  square (sum of radii) and the Heilbronn triangle problem (minimum
  triangle area). Each is optimized on its native `combined_score`.
- **Sampling baseline.** 8,000 independent programs per task, generated
  from the task description alone with no history or archive and scored
  by the same evaluator. On Cloudcast, 38% of draws return an error, which
  leaves 4,898 usable programs.
- **Equal-cost comparison.** Each seed becomes a best-score-vs-cumulative-
  tokens curve, charging every call including retries and meta-calls. For
  budget B, k seeds share it, each read at B/k. The paper takes the best
  over k seeds, then maximizes over k. Which k of the 40 seeds is drawn is
  averaged out with exact order statistics, which makes the result
  deterministic. The sampling side gets B / (cost per sample) draws, and
  its expected best is also an exact order statistic. Budgets are capped
  at what the sampling pool covers (28–48M tokens), so nothing is
  extrapolated. The paper reports equal proposals, equal calls (App. F)
  and equal tokens side by side.
- **Audit (App. G, Table 9).** Transaction scheduling: re-execute and count
  how many of 300 transactions are placed. Cloudcast: re-execute and check
  that the required partitions arrive, plus compare the declared price
  with the provided network. PRISM: a success-rate gate already in the
  logs. Circle packing and Heilbronn validate their own output and serve
  as controls. A seed is credited only with its best program that passes.
  Three transaction-scheduling seeds whose whole upper archive is the
  exploit are dropped.
- **Intervals (App. E).** 2,000 bootstrap replicates resampling both the 40
  seed curves and the sampling pool. Where the best k is 40 (a sample
  maximum), m = 20 of 40 subsampling is used instead.
- **Replications.** Haiku 4.5 on the two math tasks (6 cells, 40 × 200,
  plus an 8,000-attempt pool each). gemini-2.5-flash on the three systems
  tasks (9 cells, 40 × 200, plus pools), gated and audited the same way.

## Results

- **Cost per proposal (gpt-5-mini, Table 1).** OpenEvolve 22,985 tokens
  (1.29 calls, 8% reasoning), EvoX 26,992 (1.82, 9%), AdaEvolve 33,907
  (1.52, 14%). One sample costs 4,064–6,050 tokens (24–47% reasoning).
  Proposal/sample ratio: 4.6× circle packing, 4.8× PRISM, 5.8× Cloudcast,
  6.0× Heilbronn, 7.2× transaction scheduling. Across tasks, equal
  proposals give AdaEvolve 1.5× OpenEvolve's compute, and up to 1.9× on
  one task.
- **Evolve / sampling ratio (Table 2, post-audit; equal proposals → equal
  tokens; share of above-parity advantage removed).**

  | Task | Method | prop. | tok. | cut |
  |---|---|---|---|---|
  | Heilbronn | EvoX | 1.33 | 1.28 | 13% |
  | Heilbronn | OpenEvolve | 1.31 | 1.27 | 10% |
  | Heilbronn | AdaEvolve | 1.32 | 1.23 | 27% |
  | Cloudcast | AdaEvolve | 1.06 | 1.05 | 27% |
  | Cloudcast | EvoX | 1.05 | 1.05 | 7% |
  | Circle | EvoX / AdaEvolve | 1.04 | 1.04 | 2% / 5% |
  | Txn (audited) | EvoX / OpenEvolve / AdaEvolve | 1.04 / 1.03 / 1.03 | 1.04 / 1.02 / 1.02 | 20–38% |
  | Circle | OpenEvolve | 1.02 | 1.01 | 44% |
  | PRISM | all three | 1.00 | 1.00 | 0% |
  | Cloudcast | OpenEvolve | 1.02 | 0.99 | >100% |
  | mean | | 1.09 | 1.07 | |

  95% CIs (Table 7): only Cloudcast·OpenEvolve spans parity (0.97–1.12).
  Heilbronn·OpenEvolve is 1.17–1.34. The "cut" percentages look large only
  because the advantages are small: Heilbronn loses 0.04–0.09 of ratio.
- **Calls are not a fix (Table 8).** Mean ratio 1.085 at equal proposals,
  1.083 at equal calls, 1.069 at equal tokens. Calls recover about an
  eighth of what tokens recover and leave 11 of 15 pairs within 5%, where
  tokens leave 12.
- **Scorer exploits (Table 9).** Transaction scheduling: "3 of 300 placed"
  in 3 of 120 re-executed seeds. AdaEvolve 5.06 → 1.02 and OpenEvolve
  1.14 → 1.02 at equal tokens; EvoX is untouched at 1.04 because all 40 of
  its winners place every transaction. The 5.06 reads the same in both
  currencies because the exploit is found early and cheaply. Cloudcast:
  "no partitions sent" in 2 of 113 (never a leader, <0.1% effect). Price
  misreported in 2 of 113: the simulator prices plans from submitted edge
  data, so re-execution reproduces the wrong price and only a graph
  comparison catches it. One was a cell ceiling (EvoX 1.08 → 1.05).
  PRISM: "2% of requests served" in 22 of 120 seeds, excluded by its gate.
  An ungated best-of-k would ship one under all three methods.
- **Sampling saturation.** The final doubling of the sampling budget adds
  +0.00% (circle packing), +0.38% (PRISM), +0.44% (transaction scheduling),
  +1.37% (Heilbronn) and +3.63% (Cloudcast).
- **Haiku 4.5 (Tables 3–4).** Per-proposal costs fall in the same band as
  gpt-5-mini for EvoX and AdaEvolve (within 3–19%). OpenEvolve is 41%
  cheaper on circle packing and 9% dearer on Heilbronn. AdaEvolve costs
  2.7× OpenEvolve per proposal on circle packing (1.6× on gpt-5-mini).
  Haiku logs no reasoning tokens on any of 58,615 calls. Ratios (prop. →
  tok.): Heilbronn 1.26–1.34 → 1.17–1.26; circle 1.02–1.07 → 1.01–1.07;
  mean 1.18 → 1.14.
- **gemini-2.5-flash (Tables 5–6).** One proposal costs 16.8k–65.4k tokens
  (3.9× spread, vs 2.5× on gpt-5-mini), with 32–62% reasoning. One sample
  costs 2.1–2.6× more than on gpt-5-mini, so the proposal/sample gap
  narrows to 3.3–4.4×. Equal-token ratios: Cloudcast 1.53–1.59,
  transaction scheduling 1.15–1.25, PRISM 1.06–1.15; mean 1.31 → 1.30. The
  audit found no transaction-scheduling exploit (118/118 winners place
  every transaction) and three Cloudcast price misreports.

## Critique / open questions

- **Against the digest framing: below framing.** Every number in the
  digest entry is in the paper. What it implies about the mechanism is
  not. The front asked how much of the gap comes from the scorer audit and
  how much from token matching. On gpt-5-mini almost all of it comes from
  the audit.
  - At equal proposals after the audit, 11 of 15 pairs are already within
    5% (mean 1.09). Token matching moves one pair across the 5% line and
    lowers the mean by 0.02. Sampling had saturated, so its extra 6× of
    attempts bought almost nothing.
  - The audit removes the only large "advantage" (transaction scheduling
    5.06 → 1.02, OpenEvolve 1.14 → 1.02), plus Cloudcast·EvoX
    1.08 → 1.05. It accounts for roughly 0.28 of the mean, against 0.02
    for tokens (our arithmetic).
  - The digest says the Haiku run "reproduces the split" but omits the
    third backbone. On gemini-2.5-flash, evolution beats sampling on all 9
    systems pairs (mean 1.30) and token pricing moves it by 0.01. That is
    the paper's own counterexample to "the evolutionary advantage largely
    vanishes": it holds for one model on four of five tasks.
  - The paper's main contribution is the cost census and the
    method, not a finding that evolution is an accounting artifact. Its
    abstract says "accounting artifact", and "accounting" here includes
    the scorer audit.
- **Search scores, not held-out scores.** Every ratio uses the same
  evaluator that drove selection. There is no held-out workload or trace.
  For circle packing and Heilbronn this is acceptable because the score is
  the construction's value. For PRISM, Cloudcast and transaction
  scheduling the "best" is a maximum over one fixed evaluation. Both arms
  take a maximum, but sampling's maximum is over 5–7× more draws.
- **Ratios of raw scores are not comparable across tasks.** "≤5%" is a
  ratio of raw `combined_score`s, with no offset or headroom
  normalization. On circle packing, 2.541 → 2.636 (OpenEvolve to past the
  human record, in [[literature/papers/ishibashi2026effective]]) is a
  ratio of 1.037. A 4% ratio there is not negligible. On PRISM the panel
  spans roughly 25.5–26. The paper never says whether a 1.04 is large or
  small for a given task. This is the same caution as the normalized-ratio
  section of [[concepts/pass-at-k]], run the other way.
- **The window is short.** It is capped at the sampling pool's 28–48M
  tokens, about a fifth of evolution's corpus, and the paper says
  evolution is still rising at the cap. Every "≤5%" is a statement about
  the first fifth of each run, which is the regime where fresh samples are
  most competitive. The paper calls these lower bounds. That is right for
  evolution's margin, and it means the takeaway "fresh sampling is almost
  as good" is not established at the budgets the runs actually spent.
- **Evolution picks k in hindsight.** The evolution side maximizes over
  the seed count k at each budget, and the sampling side has no knob. This
  favours evolution slightly and does not change the direction of the
  result.
- **Single-call operators only.** Every strategy uses stateless API
  proposals. The paper says nothing about coding-agent operators that
  iterate against the evaluator before submitting
  ([[literature/papers/ishibashi2026effective]],
  [[literature/papers/farahbakhsh2026agentdiscover]]). Its OpenEvolve
  circle-packing cost (22.8k tokens/proposal on gpt-5-mini) matches
  ishibashi's 26.5k on gpt-5.2-codex, which is a useful cross-check.
- **Exploits and HCE.** The evaluators were unchanged, so clause 4's hash
  would have passed every exploit. The exploits are cases of
  [[literature/papers/chen2026when]]'s "pristine is not correct": a
  3-of-300 schedule is a near-do-nothing submission the scorer accepted.
  Two points are new to the graph:
  - Re-execution itself can reproduce a wrong score. Cloudcast prices
    plans from the submitted edge data, so only a comparison with the
    provided graph catches the misreport.
  - Selection concentrates an exploit. A third of one seed's candidates
    and its top 69 distinct programs were the same exploit.
  This fits [[literature/papers/li2026beyond]] (6.3% of best-of-3
  solutions exploit the evaluator) and ishibashi's capability-scaled hack
  rate. The finding that EvoX never hit the transaction-scheduling exploit
  across 40 seeds is left unexplained.
- **What it means for compute-neutral comparisons here.** For the
  redirect-before-halt proposal and any `/iterate` arm comparison, the
  lesson is about *which unit is neutral*. A cycle that carries history
  costs more tokens than one that starts fresh, and later cycles cost more
  than earlier ones. Equal cycles are not equal tokens, and equal calls
  are not either. The paper also shows the size of the error depends on
  whether the cheap arm has saturated. When it has, the currency hardly
  matters. On this box, `/iterate`'s cost is dominated by GPU training
  runs, which the paper does not price. Its evaluators run in seconds.
- **Not shown.**
  - Any ML-research task. These are program-optimization tasks with
    cheap evaluators.
  - Released logs or code, so the order-statistic numbers cannot be
    re-derived.
  - Whether the method ranking changes within a task at equal tokens.
    Only ratios against sampling are given. In Heilbronn, AdaEvolve goes
    from tied to last (1.32 → 1.23).
- **Related evidence in the graph.**
  - [[literature/papers/wang2026rethinking]] asked for a same-budget
    sampling arm. This paper says what "same budget" has to mean.
  - [[literature/papers/xing2026compute]] budgets C = T·N in LLM calls.
    Here, calls carry 12.4k–28.1k tokens each, and the cost grows with
    archive size, so a depth-heavy split costs more tokens per call than a
    breadth-heavy one.
  - [[literature/papers/zou2026fmlbench]]'s "opportunity density decides
    the topology" is the same kind of claim as "the task decides whether
    search pays". The gemini result shows the model decides it too.
  - [[literature/papers/cai2026forkandflush]] matched model tokens for its
    flush/fork arms, which is the accounting this paper asks for.

## Trust signals

- **Credibility:** 3.
  - For: IBM Research authors (all four). A large, clean design: 600 full
    runs of 40 seeds × 200 iterations per cell on one backbone, plus 15
    replication cells on two more, with every call's token counts logged.
    Exact order statistics remove resampling noise from the point
    estimates. Intervals resample both sides, and subsampling is used
    where the bootstrap is inconsistent. Proposal, call and token budgets
    are reported side by side, so the reader can split the corrections
    apart (we did). The limitations section names the window cap. The
    paper reports the gemini backbone even though it cuts against the
    headline.
  - Against: an arXiv preprint (13 days old), not peer reviewed, with no
    released logs or code. The ADRS-Bench tasks and SkyDiscover are
    public, but the 600 runs are not. A short-paper format leaves many
    choices one sentence deep: why raw-score ratios, how Cloudcast's
    cost-minimizing objective becomes a ratio, and how the best k is
    chosen. The abstract's "accounting artifact" outruns Table 2's
    currency effect. A concurrent companion by the same authors
    (arXiv 2609.19799) supplies the sampling-baseline protocol.
  - Not 4: no code or logs, not peer reviewed, and the framing claims
    more than the tables show. Not 2: a major lab, a large logged corpus,
    sound statistics, and an internal decomposition that lets the reader
    check the framing.

## Follow-up

- **Relevance:** 4. It is the first source in the graph that measures what
  one evolutionary proposal costs against one fresh sample, and it shows
  that equal-proposal and equal-call budgets are not compute-matched. It
  adds material evidence to [[concepts/hce-evaluation]] (a matched sampling
  arm has to be priced in tokens and audited the same way as the evolved
  arm; selection concentrates exploits; re-execution can reproduce a wrong
  score), [[concepts/budget-as-ceiling]] (the unit in which a redirect or
  arm comparison is "compute-neutral"), [[concepts/evolutionary-expansion]]
  (whether evolution beats fresh sampling depends on task and model) and
  [[concepts/evolutionary-search-grain]] (the context an evolve prompt
  carries is a cost). Not 5: no ML-research task, a short window, and the
  currency effect it is named for is small once exploits are removed.
- **For `/iterate` and redirect-before-halt.** Log tokens per cycle,
  including retries, so an arm comparison can be read in tokens later.
  The paper's point that "the logs already hold them" applies here too.
  Not proposed here.
- **Candidates** (cited, not yet in the graph):
  - Oved et al. 2026, "Evolution or illusion? Rethinking evaluation in LLM
    evolutionary search" (arXiv 2609.19799). Same authors; the k-vs-t
    split at a fixed proposal budget and the sampling baseline protocol.
  - Beck, Ogren, Kobren 2026, "Hill sampling for test-time scaling"
    (arXiv 2609.25510). Sampling edits to the best verified solution as an
    alternative to archives.
  - Zhang et al. 2026, "Budget-aware LLM discovery via cost-calibrated
    frontier utility" (arXiv 2607.26828). A token-priced search controller.
  - Rajan 2026, "Auditing reward hackability in code RL training
    environments" (arXiv 2606.16062).
  - Cheng et al. 2025, "Barbarians at the gate" (ADRS-Bench, arXiv
    2510.06189).
