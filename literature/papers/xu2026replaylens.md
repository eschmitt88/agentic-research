---
kind: paper
title: "ReplayLens: Auditing Agents' Use of Outcomes"
authors: ["Dong Xu", "Zhangfan Yang", "Jiantao Wu", "Shipeng Zhang", "Zexuan Zhu", "Jianqiang Li", "Jun Zhang", "Junkai Ji"]
institutions: ["Shenzhen University (School of Artificial Intelligence)", "EasternDawn", "University of Nottingham Ningbo China"]
year: 2026
venue: "arXiv 2609.34177 (cs.AI; cross-listed cs.LG, cs.MA), v1 2026-09-28, v2 2026-10-03, preprint, 76 pages"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.34177"
code_url: null
citations: null
source: "raw/papers/xu2026replaylens.pdf"
added: "2026-10-13"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/structured-world-model]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hce-evaluation]]"
tags: ["replay-audit", "memory-audit", "outcome-reassignment", "pair-transport", "record-order", "log-migration", "iteration-log", "key-score-binding", "order-invariance", "endpoint-equivalence", "fifo-eviction", "sealed-test", "experiment-planning", "code-debugging", "gpt-6-astra"]
---

# ReplayLens: Auditing Agents' Use of Outcomes

## TL;DR

ReplayLens is a black-box audit of what an agent takes from a logged
history of (action, score) records. It replays the same history to the
same agent with one relationship changed at a time and counts how many
future decisions change. There are four interventions:

- **Outcome (value) reassignment** permutes which score belongs to which
  action.
- **Pair transport** moves intact action–score pairs to new record slots.
- **Consistent renaming** relabels actions in both the history and the
  menu.
- **Key-slot reassignment** moves action names over unchanged scores.

Value and pair cells use the same permutation, so both show the agent the
same score sequence; only the binding differs. For GPT-6 Astra, swapping
scores changes decisions and moving intact pairs does not. In a small
sequential model-selection task, swapped initial scores cut final sealed-test
utility from 0.618 to 0.359. The paper never gives the agent a way to
re-measure a misattached record, so "despite fresh measurements" means the
three new experiments did not repair the damage. It does not mean the agent
ignored contradicting evidence.

## Claims

- **Endpoint equivalence does not determine replay behaviour.** For any
  stateful writer, a lookup table can copy its aligned decisions and its
  memory-on/off contrast while staying invariant to every replay (finite-
  support witness, A.5–A.6). Empirically, five fitted additive/recurrent
  writer pairs with identical aligned and empty-state actions disagree on
  0.000 vs 0.538–0.796 of holdout actions after pair transport (Table 1).
- **Pair-transport zero law (Eq. 4).** If a writer's state is a sum of a
  position-independent function of each (key, score) pair, reordering
  intact pairs leaves every decision unchanged (Deep Sets argument). A
  non-zero pair response therefore diagnoses order dependence.
- **Black-box LLMs follow score attachment, not record order** "on the
  tested support". GPT-6 Astra's choices agree with a historical-mean rule
  on 1.000 of full and value-swapped ledgers. Sum, count, latest and best
  agree on 0.18–0.64.
- **Altered historical scores redirect exploration and reduce final utility
  despite fresh measurements** (abstract), in both sequential experiment
  planning and code debugging.
- **Practical guideline**: before a log migration, run the replay cell that
  matches the transformation (reorder → pair transport; identifier change →
  key-slot; scores moved → value reassignment; joint relabel → consistent
  renaming). "A nonzero action response flags a compatibility risk."

## Methods

- **Readouts.** Action disagreement d(a, b) over future tasks, plus paired
  utility differences from a fixed external evaluator. For hosted models,
  T_gr subtracts within-condition disagreement (two fresh calls per cell)
  from between-condition disagreement (Eq. 3). Streams are the bootstrap
  unit.
- **Synthetic calibration (§4.2–4.3, App. E).** Writers with known laws
  (replay-invariant, key-only, commutative-pairing, order-sensitive), 3
  parameterisations each, 48 streams. All 12 are classified correctly. A
  family-blind challenge with a pre-fixed threshold misses one
  pair-displacing writer whose state changed without crossing the decision
  threshold.
- **Joint black-box matrix (§4.4, App. Q.2/T).** GPT-6 Astra at max
  reasoning, 48 streams × 2 namespaces (semantic / opaque labels) × 5
  ledger forms × 4 history/menu cells × 2 repeats = 3,840 requests (69M
  input tokens). Histories are *designed* so that the five reference rules
  disagree. This is "a retrospective test on designed histories", and group
  summaries rest on 5–10 unique streams. 14 capacity failures were replaced
  under a disclosed amendment.
- **Bounded memory (§4.5, App. Q.1).** Four 16-slot / 8-retrieved stores:
  FIFO, logical-time retention, identifier-priority reservoir, and per-action
  descriptor-bin means. Selection was frozen on 16 development streams, then
  tested on 48 with outcomes sealed until all decisions were recorded.
- **Sequential experiment planning (§4.6, App. R).** 48 resampled
  binary-classification tasks (Breast Cancer Wisconsin, Wine Quality; i.i.d.
  and covariate-shift strata). There are 96 configurations (sampling policy ×
  preprocessing × logistic/LDA × 3 regularisation levels × 12/24 labels).
  Each history starts with 8 measured configurations. The agent runs 3
  *untried* configurations through a tool, then submits one of the 11. The
  final test pool is sealed until every choice is recorded. Two trajectories
  per task × condition. References: random search with the same 3 calls, and
  the "initial incumbent" (top displayed initial score, no calls).
- **Code debugging (§4.7, App. S).** 24 deterministic bug-fix tasks from 8
  function families (offset, clamping, dedup, top-k, …), 12 candidate patches
  each, a 4-entry history of public-test pass fractions, 3 test calls, 1
  submission, and hidden tests. 192 trajectories.
- **Cross-model (§4.8).** Archival GPT-5.6 (Luna/Sol/Terra) and a frozen
  Qwen3-8B bridge. Rows are separate evidence tiers and are not pooled.

## Results

- **Joint black-box (Table 3).** T_gr against the full ledger: value
  reassigned 0.807, key-slot 0.766, pair 0.000, utility-masked 0.53. The
  same holds in both namespaces. Rekeying the history but not the menu gives
  1.000, and rekeying both gives 0.000. The *utility* consequences in this
  matrix are tiny: Figure 7's replay − full utility axis spans ±0.01.
- **Bounded memory (Tables 5, 6, 83, 84).** Reordering complete events
  changes FIFO's stored summaries and retrieved text in 192/192 test blocks.
  The other stores change in 0. Audit-selected binned-mean minus FIFO is
  +3.0 × 10⁻³ utility (97.5% CI [0.09, 6.15] × 10⁻³). **The audit and the
  endpoint-only selector chose the same pipeline**, so audit − endpoint is
  −0.29 × 10⁻³ [−1.08, 0.21]. The prespecified joint criterion for
  added audit value "is not met". The FIFO comparison also changes coverage:
  binned means summarise all 32 observations, while FIFO keeps 16.
- **Sequential planning (Tables 7, 87–91).**

  | Condition | First-choice T_gr | Submit T_gr | Final-test U (agent) | Random search | Initial incumbent |
  |---|---|---|---|---|---|
  | Aligned | — | — | 0.618 [0.598, 0.639] | 0.610 | 0.594 |
  | Value reassigned | 0.708 | 0.875 | 0.359 [0.303, 0.415] | 0.443 | 0.342 |
  | Pair transport | 0.031 | 0.000 | 0.617 | 0.610 | 0.594 |
  | Consistent renaming | −0.042 | −0.021 | 0.619 | 0.610 | 0.594 |

  Value − aligned = −0.259 (97.5% CI [−0.330, −0.190]). Pair − aligned =
  0.000 [−0.002, 0.001]. Random search minus agent is −0.008
  [−0.028, 0.014] when aligned and **+0.084 [0.036, 0.137] when values are
  swapped**. The agent therefore does not beat a same-budget random search
  on clean logs, and loses more than random search on corrupted ones. Its
  corrupted-log utility sits next to the initial-incumbent reference (0.342),
  which suggests it mostly submits the falsely top-scored initial record.
- **Code debugging (Table 93).** Value reassignment changes the first test
  choice in 0.604 of paired comparisons and the final submission in 0.167.
  Hidden utility −0.125 [−0.292, 0.000]. Aligned hidden utility is 1.000, at
  ceiling. Pair and renaming: 0 final and 0 utility response.
- **Cross-model (Table 9).** Value > pair in every row. Qwen3-8B has the
  largest pair response (0.154, against ≤ 0.017 for the GPT-5.6 variants).
  The authors decline to read this as a model-size effect.

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** The
  digest line ("decisions follow the logged score vs record order; altered
  historical scores redirect exploration despite fresh measurements") is
  literally supported. The headline also comes from the most favourable cut.
  - *"Despite fresh measurements" is a design property, not a finding about
    evidence weighting.* In the planning task the agent can only run
    *untried* configurations. It can never re-measure a record whose score
    was swapped, so no fresh measurement ever contradicts the altered log.
    The −0.259 shows that a falsified top score survives three unrelated
    experiments when the final pick is allowed to be that record. It does not
    show that the agent preferred a logged number over a conflicting fresh
    one. The /curate gloss ("agents following altered logged scores over
    fresh measurements") is not tested anywhere in the paper.
  - *Most of the loss is mechanical.* Any rule that trusts the log loses:
    the no-call initial-incumbent reference falls 0.594 → 0.342 and random
    search 0.610 → 0.443. The audit shows *that* the agent uses scores, which
    is the desired behaviour for a log reader. The interesting residual is
    that the LLM loses 0.084 more than random search with the same budget.
  - *The other settings in the abstract are weaker than its wording.* Code
    debugging "shows the same pattern" with a hidden-utility interval that
    touches 0, a 0.167 final-choice response and an aligned arm at ceiling.
    The bounded-memory study "exposes ingestion-order sensitivity that
    endpoint comparison misses". But endpoint selection picked the same
    pipeline, the paper's own joint criterion for audit value failed, and
    the effect is in 10⁻³ utility units. In the main black-box matrix,
    0.8-scale decision changes come with utility changes of order 0.01.
- **The ML-research setting is a toy.** 96 configurations of logistic
  regression / LDA on 12–24 labelled points, 3 experiments, one model
  (GPT-6 Astra). The aligned agent ties random search (−0.008, CI spans 0).
  Nothing here resembles a multi-cycle training loop with noisy metrics, so
  the transfer to `/iterate` rests on the log-reading mechanism, not on
  effect sizes.
- **Order invariance is measured on short, designed ledgers.** 24 records,
  four actions, histories built so reference rules disagree. Qwen3-8B's
  0.154 pair response shows that order invariance is model-specific. It
  cannot be assumed for long, append-only logs read by small models, and
  the paper tests no long horizons ("longer-horizon trajectories … remain
  extensions").
- **Reproducibility is asserted, not checkable.** The text repeatedly cites a
  "supplement" and a "reproducibility archive" with frozen prompts, tool
  trajectories and analysis code. The arXiv abs page lists no ancillary files
  and no repository (checked 2026-10-13). GPT-6 Astra and the archival
  GPT-5.6 readouts are hosted aliases that the authors note may drift.
- **Disclosure is unusually careful.** Sealed test scoring, frozen selection
  rules, amendments for capacity failures and host restarts, and token usage
  are all reported. Negative results (the Q.3 learned control fails its own
  quality gate; Q.1's audit criterion is not met) stay in the appendix
  rather than being dropped. Sixty-plus pages of appendix serve one fairly
  simple intervention design.
- **Open for this graph:** the test that matters for `iteration_log`
  integrity was not run. Give the agent a record whose logged score
  conflicts with a re-measurement it can trigger, and see which it believes.
  [[literature/papers/hu2026memory]] is the nearest datum: agents answer
  with a stale stored value 0.92–1.00 of the time even when an authoritative
  tool holds the correct one.

## Trust signals

- **Credibility:** 2.
  - For: a clear intervention design with stated preservation conditions,
    a falsifiable invariance law checked against writers with known
    update rules, sealed external evaluators, repeat-corrected
    disagreement, and itemised acquisition amendments.
  - Against: a university group (Shenzhen University, with EasternDawn and
    University of Nottingham Ningbo China), not peer reviewed. The
    supplement and analysis code it relies on are not public. It uses a
    single hosted model for every fresh black-box result, designed
    histories, and toy tasks. Utility effects outside the planning task are
    near zero or have intervals touching zero.
  - Not 3 because none of the claimed artifacts can be checked yet. The
    rigour of the protocol is real but unverifiable.

## Follow-up

- **Relevance:** 3. It is a cite-worthy audit method and the first
  measurement in this graph of how an LLM agent reads an (action, score)
  log. It reads the log as key–score bindings (historical mean), not as a
  sequence, and binding corruption propagates to the final pick through
  later experiments. It sharpens [[concepts/verified-memory-writes]] and
  the `iteration_log` reading of [[concepts/structured-world-model]] but
  does not shift either. The headline loss is close to "a log-following
  agent follows a corrupted log", and the setting is a toy.
- **For `iteration_log` integrity (our inference, not the paper's):**
  - The log line's `Δ<metric>` keyed by experiment slug is a *derived copy*
    of each experiment's `metrics.json`. Key–score misattachment is the
    failure ReplayLens shows propagating. Read the chain's best from
    `metrics.json` by folder, and treat the log line as a view.
  - Before declaring a final champion, re-measure it rather than trusting its
    logged score. This is also [[literature/papers/hu2026winners]]'s point:
    the stored champion score is a lucky draw.
  - Record order appears not to matter for strong models reading the log, so
    merging or reordering log lines is low risk. Renumbering or renaming
    experiments without rewriting the scores is the risky migration
    (key-slot cell: 0.766).
- **For HCE:** the aligned planning agent tied a same-budget random search.
  That is one more datum for the same-budget-search-arm section of
  [[concepts/hce-evaluation]] (no edit proposed: one toy setting).
- **Candidates** (cited, not in the graph): Xue et al. 2026, PAST-Bench
  (arXiv 2608.04003); Garg et al. 2026, MemFail (arXiv 2605.26667); Shah
  2026, Causal Agent Replay (arXiv 2606.08275); Lin et al. 2026, REFLECT
  (arXiv 2606.09071); Zhang 2026, "Credit without ground truth" (arXiv
  2608.19760); Rocchi 2026, CAMEO (SSRN 7160321).
