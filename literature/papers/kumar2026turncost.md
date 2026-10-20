---
kind: paper
title: "TurnCost: Turns, Not Observations: Cache-Aware Loop Engineering for Coding Agents"
authors: ["Pramod Kumar", "Raj Kumar Pandey"]
institutions: ["The Pennsylvania State University"]
year: 2026
venue: "POVC '26 (1st International Workshop on Prompt-Ops and Vibe Coding), co-located with ASE 2026, Munich; arXiv 2610.15093 (cs.SE), v1 2026-10-10; DOI 10.1145/3843779.3844635"
peer_reviewed: true
url: "https://arxiv.org/abs/2610.15093"
code_url: "https://doi.org/10.5281/zenodo.22216661"
citations: null
source: "raw/papers/kumar2026turncost.pdf"
added: "2026-10-20"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/lossless-context-offload]]"
tags: ["cost-accounting", "prefix-cache", "cache-aware-pricing", "turn-count", "observation-size", "context-reduction", "step-level-routing", "delegation-break-even", "truncation-baseline", "agentdiet", "trajectory-rescoring", "open-swe-traces", "openhands", "swe-agent", "coding-agents", "loop-engineering"]
---

# TurnCost: Turns, Not Observations: Cache-Aware Loop Engineering for Coding Agents

## TL;DR

A cost-accounting study. No agent is run. The authors take 600 published
coding-agent trajectories from `nvidia/Open-SWE-Traces`: 150 each from 2
scaffolds (OpenHands, SWE-agent) × 2 open-weight models (Qwen3.5-122B-A10B,
Minimax-M2.5), on SWE-rebench-V2 tasks in nine languages. They retokenize
each trajectory with `o200k_base` and simulate an exact append-only prefix
cache. Then they re-price five loop policies offline. Four findings:

1. **Observations are small.** The median tool observation is 118–286
   tokens per cell. The p99 is 4,274–6,208. The "huge build log" that
   motivates compression and routing work sits in about the top 1%.
2. **Flat input pricing overstates cost by 6.5–7.8× (mean 7.14×).** In the
   simulation, 98.1–99.0% of input tokens are cache hits. At 90% cache
   efficiency the gap falls to 4.0×, and at 50% to 1.45×.
3. **Delegating observations to a cheaper model saves no more than plain
   truncation.** Across 20 cell × threshold combinations, truncation
   matches or slightly beats weak-model delegation (largest gap 0.17 pp).
   Both save 11–19% at aggressive thresholds. Savings go negative at
   conservative thresholds in two of four cells.
4. **Turn count, not observation size, tracks cost.** On log–log axes, turn
   count alone gives R² = 0.844 (exponent 1.23). Mean observation size
   alone gives R² = 0.040 (exponent −0.31). Together they give 0.945.

They also derive a closed-form break-even rule. Out-of-band delegation pays
only when the remaining turns exceed roughly p_W^in / p_S^cache. The
comparison is with the strong model's *cached-read* price, not its list
price. Task success is never measured. Every result is about cost on fixed
trajectories.

## Claims

- "Turn count explains 84% of the variance in log trajectory cost,
  compared with 4% for mean observation size; the most expensive
  trajectories have the most turns despite having the smallest
  observations" (abstract).
- "98.6% of input tokens are served from a prefix cache, so flat input
  pricing overestimates loop cost by roughly 7× on average under
  near-ideal caching" (abstract). §5.7 asks that the 7.1× figure "be cited
  with that caveat attached".
- The critique of cache-unaware accounting applies to AgentRouter,
  SWE-Router and MTRouter. It does not apply to AgentDiet, which prices
  cached input. TRACE-Router reports no token-level cost model.
- A weak model swapped in for a whole turn (in-band) must beat the strong
  model's cached price, because it prefills the full transcript cold.
  "A weak model priced at 20% of the strong model's list price appears to
  offer an 80% saving under flat accounting but is a net loss once caching
  is counted."
- The bulk of observation tokens comes from edits, not logs. Edit turns
  are 29.9% of turns and 51.8% of observation tokens (mean 787 tokens).
  Read/exec turns are 67.4% and 43.7% (mean 294).
- The small median is set by agent–computer interface design, not by
  scaffold truncation limits. SWE-agent's cap is about 25,000 tokens, two
  orders of magnitude above the median.
- Recommendations: price the cache before optimizing; use the break-even
  rule as a deployment check; always compare against a truncation
  baseline. Removing unnecessary turns is named as the larger opportunity,
  "though we do not establish that turns can be removed while holding
  task success fixed".

## Methods

- **Corpus.** `nvidia/Open-SWE-Traces` (207,489 trajectories, CC BY 4.0).
  The first 150 rows of one parquet shard per cell, taken in file order
  with no shuffling. Shard IDs are in the artifact README. Median turns per
  cell: 97 / 71 / 60 / 134. Median context: 46–73k tokens.
- **Cost model.** Cost = Σ_t [p_S^cache·h_t + p_S^in·(C_t − h_t) +
  p_S^out·o_t], with h_t = C_{t−1}. Tool schemas count as input. Reasoning
  tokens count as output and are not carried forward. Prices are
  normalized to p_S^in = 1, with p_S^cache = 0.1, p_W^in = 0.05 and
  compression ratio σ = 0.05 by default. All three are swept in §5.7.
- **Five conditions.** B0 is the unmodified baseline. A* swaps in the weak
  model for the turn (in-band, full transcript). R1 delegates large
  observations to the weak model (out-of-band summary). B2 is the same
  compression done by the strong model, to separate compression from
  routing. B3 is deterministic truncation, the analogue of AgentDiet's
  *Delete*.
- **Threshold sweep.** Observations above 200–5,000 tokens are delegated
  or truncated.
- **Truncation behaviour.** The authors scan raw observation text for
  scaffold truncation markers and classify the agent's next action
  (re-query, identical retry, terminate, other).
- **Regression.** Per-trajectory cost on turn count and on mean
  observation size, log–log, across all 600 trajectories.

## Results

- **Cache inflation by length.** The flat/cached ratio is 5.33× below 40
  turns and 7.99× above 130 turns. Short-trajectory routing results
  (AgentRouter's 4–12 steps) are therefore least affected.
- **Truncation vs delegation (Fig. 3).** B3 ≈ R1 in all four cells. B2
  (strong-model compression) is consistently worse. A compressed late
  observation has too few remaining turns to repay its own model call.
  This is Eq. 3 visible in data.
- **In-band swap (§5.5).** At the defaults (p_W^in 0.05 < p_S^cache 0.10)
  A* saves 14.7% at a 200-token threshold and 0.4% at 5,000. At the most
  aggressive threshold it beats R1 (14.7% vs 13.5%).
- **Turn deciles (§5.6).** From the first to the tenth decile, median turn
  count rises 36 → 168 and median cost rises about 7× (0.138 → 0.978).
  Median observation size *falls*, 655 → 334. Cost per turn is fairly
  stable (median 0.005, CV 0.28).
- **Ceiling (§5.7).** Observations are 66–74% of transcript growth. An
  oracle that compresses every observation to 5% at no cost saves
  47.8–60.3%. Across 16 vendor-price combinations the truncation-minus-
  delegation gap stays ≥ 0, up to +1.0 point.
- **Truncation markers (§5.8).** 518 of 56,164 observations (0.92%) are
  truncated, in 367 of 600 trajectories. In 468 cases the scaffold adds
  recovery guidance (e.g. narrow with `grep -n`). After guided truncation,
  76.9% of next actions narrow the query and 0.2% terminate. After the 50
  bare truncations, 44.0% narrow. No agent repeats the identical command.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). The three numbers in the
  digest are in the paper and match its figures. The entry leaves out
  three things that change how they read:
  1. *"98.6% hit the prefix cache" is a simulation, not a measurement.*
     The cache is reconstructed with h_t = C_{t−1}, so near-total reuse
     holds by construction for an append-only transcript. Real providers
     round to blocks, expire entries and lose reuse across backends. The
     authors' own sweep cuts the 7.1× to 4.0× at 90% efficiency.
     liu2026what's metered 96–99% cache hits are the better corroboration.
  2. *"84% vs 4%" is the most favourable cut for observations being
     unimportant.* The 4% is the univariate R² of mean observation size.
     Added to a model that already has turn count, observation size raises
     R² by 0.101, which is not negligible. Variance explained is also not
     cost share. Observations are 66–74% of transcript growth, and an
     oracle compressor would cut cost by about half. Observations are most
     of the bill. They just do not vary much between trajectories.
  3. *The turn–cost link is partly mechanical.* Cost is a sum over turns.
     The authors say so. Their two non-definitional points are fair: the
     exponent is above 1, and observation size slopes negative.
- **No quality arm.** Every result holds the trajectory fixed. "Truncation
  equals delegation" is a cost statement only. "Remove unnecessary turns"
  is a direction with no test of whether those turns can go without
  losing success. The truncation-guidance numbers are observational, with
  n = 50 bare truncations, and do not show whether content was recovered.
- **Corpus filter.** Open-SWE-Traces excludes runs that hit the iteration
  limit or were terminated by the harness. This trims the long-turn tail.
  It also removes exactly the "agent re-reads a huge log until budget
  runs out" case that would fatten the observation tail. The medians are
  probably right. The tail is understated.
- **Scope.** Two open-weight models, two scaffolds, and one tokenizer
  (`o200k_base`, 10–20% off native counts). There is no frontier
  proprietary agent and no research or ML-engineering task. Sampling is
  deterministic but not random (the first 150 rows of one shard).
- **What it means for the hub's live fronts.**
  - *Budget ceilings.* This project's `budget.yaml` caps `max_tokens`. If
    that counter adds cached reads at par with fresh input, it overstates
    billed spend by several times on long sessions. That is a
    units question for [[concepts/budget-as-ceiling]] rather than an
    error, but a ceiling and a forecast should use the same units.
  - *Redirect-before-halt proposal.* Only indirectly. The proposal's
    pivot cycle replaces a refinement rather than adding one, so turn
    count and cost are unchanged. The paper's claim that unnecessary turns
    are the big lever is untested, so it cannot be cited for a gain from
    stopping earlier.
  - *Noise band, HCE evaluation, shared-substrate error, skill scoping.*
    No bearing. There is no evaluation of outcomes, and the only judge
    is arithmetic.

## Trust signals

- **Credibility:** 3.
  - For: published in ACM workshop proceedings with a DOI. Public CC BY
    corpus, deterministic sample with shard IDs listed, and a Zenodo
    archive of the tool, analysis code and derived results (not checked
    for this note). The cost model is a closed form that anyone can
    recompute. The limitations section is candid: it names the corpus
    filter, says the regression is associational, and asks that 7.1× be
    cited with its caveat.
  - Against: workshop-level review. Two authors from one university, with
    no senior-lab track record on this topic visible from the paper. All
    costs are simulated, not metered. No quality measurement. 150
    trajectories per cell from one shard each.
  - Not 4 because the core quantities come from a simulated cache on a
    filtered corpus with no outcome arm. Not 2 because the method is
    simple, fully specified, and reproducible from public data.

## Follow-up

- **Relevance:** 3. The two main findings confirm what the graph already
  holds. Cache reads dominate the bill (bai2026how), and step count can
  outweigh everything else (liu2026what). What is new is useful but
  narrow: a cache-aware break-even rule for delegating to a cheaper model
  ([[concepts/hybrid-model-backends]]); a measured observation-size
  distribution showing per-observation eviction mostly has nothing to bite
  on ([[concepts/context-eviction-policy]]); and a stable cost per turn
  that makes turn count the variable a forecaster should model
  ([[concepts/spend-forecast-calibration]]). Not 4 because none of it
  carries an outcome measurement, and the tasks are coding, not research.
- **Related in the graph.**
  - [[literature/papers/bai2026how]]: cache reads dominate cost in every
    phase. TurnCost extends this to an explicit flat-vs-cached ratio.
  - [[literature/papers/liu2026what]]: metered 96–99% cache hits.
    Preamble × step count drives the bill.
  - [[literature/papers/mason2026missing]]: argues size should drive
    eviction priority. TurnCost shows the large-page tail is about 1% of
    observations in coding traces.
  - [[literature/papers/nguyen2026cliffcompaction]]: a dropped observation
    kept as a re-issuable call signature. This fits TurnCost's
    truncation-with-guidance numbers and its truncation ≈ delegation
    parity.
  - [[literature/papers/fan2026empirical]]: the recall tool goes unused
    when the workspace can re-derive content. Same scope condition.
- **For this project.** Check whether the coordinator's token counters
  and `max_tokens` weight cache reads separately. If they do not, record
  that ceilings are in raw tokens, not billed cost.
- **Candidates** (cited, not yet in the graph):
  - Xiao et al. 2026, AgentDiet (FSE 2026, doi:10.1145/3797084). This is
    the closest prior work and the source of the Delete baseline.
  - Ahmad et al. 2026, Open-SWE-Traces (arXiv 2606.16038).
  - Paul & Nandy 2026, AgentRouter (ICML 2026 workshop).
  - Son et al. 2026, SWE-Router.
  - Zhang et al. 2026, MTRouter (ACL 2026).
  - Raj et al. 2026, TRACE-Router (arXiv 2607.22465).
  - Salim et al. 2026, Tokenomics (arXiv 2601.14470).
