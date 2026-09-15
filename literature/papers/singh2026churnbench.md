---
kind: paper
title: "ChurnBench: A Drift-Aware Benchmark Demonstrating That Refresh Scheduling, Not Cache Age, Governs Staleness in Agentic AI"
authors: ["Vivek Kumar Singh", "Preeti Priyam"]
institutions: ["Independent Researcher (McKinney, TX, USA)"]
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.11515"
code_url: "https://github.com/vsingh45/churnbench"
citations: null
source: "raw/papers/singh2026churnbench.pdf"
added: "2026-09-15"
relevance: 3
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["staleness", "freshness", "retrieval", "benchmark", "evaluation-design", "caching", "negative-result"]
---

# ChurnBench: A Drift-Aware Benchmark Demonstrating That Refresh Scheduling, Not Cache Age, Governs Staleness in Agentic AI

## TL;DR

A frozen corpus cannot produce a stale answer, so retrieval benchmarks
cannot measure staleness. ChurnBench generates an enterprise data fabric as
a **timeline**. Every change goes to an append-only ledger, and gold answers
are resolved from the ledger, never from the stores. A wrong answer that was
right at its effective retrieval time is then labelled a **freshness error**,
separate from a reasoning error. Pointed at a grounding layer with tiered
scheduled refresh, it finds that **cache age does not predict staleness**:
**7 / 4 / 4** freshness errors at 1 / 14 / 28-day cache ages. Turning refresh
off raises the 28-day count **4 → 45** and leaves the 1-day count at 7.
Exposure is set by **tier width × entity mutation rate**, so a drift benchmark
should sweep TTL against change rate, not the drift window. The instrument
and the documented design error are the contribution. The staleness
mechanism itself is ordinary cache behavior.

## Claims

- **Staleness is unmeasurable under the usual evaluation design.** Benchmarks
  freeze the corpus, so an answer can be unfound but never out of date.
- **A freshness error can be defined and checked without a judge.** Wrong
  against gold at evaluation time T, correct against gold at T_eff. T_eff
  comes from the trace: T for live routes, and the last-refresh time of the
  entity read for staged routes, taking the minimum when several contribute.
  Matching gold at neither time is a reasoning error. Every reported
  freshness error was re-resolved at both timestamps.
- **Scheduled refresh bounds staleness by TTL, not by build time.** A 1-day
  entity is refreshed throughout any longer window and arrives about a day
  old whether the cache was built 1 or 28 days earlier. "Cache age … never
  actually reaches the retrieval layer."
- **Exposure is a product.** Neither tier width nor mutation rate alone
  predicts where errors land. Tiers should be assigned from **measured**
  mutation rates "rather than from intuitions about which data feels
  important."
- **Supersession does not cover derived objects.** Contrasting itself with
  MemStrata's bi-temporal supersession rule (arXiv:2606.26511), the paper
  argues that a supersession rule "cannot refresh a materialized aggregate or
  expire a cache tier, because those have no fact-level identity to
  supersede."
- **Lapse counters are a measurement trap.** The TTL-lapse count read zero in
  both configurations for opposite reasons. Under tiering nothing lapsed;
  without tiering the check never ran. Record per-entity refresh timestamps.
- **Scope, stated by the authors:** no architecture claim, no baseline under
  the final design, and accuracy numbers that should not be read as
  performance.

## Methods

- **Fabric.** Software asset management with four sources: PostgreSQL
  warehouse (history), MongoDB (current operational state), a mock SaaS REST
  API (350 ms latency, 60 req/min), and synthetic contract documents that
  exist only in prose. A seeded simulator draws daily Poisson event counts
  (hires, offboardings, reassignments, price changes, renewals, cost-center
  moves, dense consumption), with changes concentrated on the top 10% of
  entities. Offboarding and renewal rates are calibrated from BLS turnover
  and industry reports. Reassignment and repricing rates are practitioner
  judgment. Stores are idempotent folds of the ledger at a timestamp.
- **System under measurement.** A declarative semantic registry gives each
  entity class a tier and TTL: assignments and user status hot at 1 day,
  prices and cost-center membership warm at 7, contract terms and vendor
  dimensions cold at 30, consumption live and never staged. A rule-based
  router picks one of five paths (staged template, federated, live warehouse,
  origin API, vector lookup) with no model call and logs every entity's
  last-refresh time.
- **Frozen-at-T evaluation.** Cache built at T′, questions asked at T, and
  the world does not drift during a run. The harness walks day by day from T′
  to T and refreshes any lapsed entity.
- **Tasks.** 180 templated tasks: 81 single-source, 63 cross-source, 36
  cross-modality, over four executive intents. An always-zero responder
  scores 1.7%, enforced by a test. Unanswerable event-history and per-license
  cost questions are deliberately kept in.
- **Runs.** One model (nvidia/nemotron-3-ultra-550b-a55b via NIM, T = 0,
  reasoning off for Table I), LangGraph. About 7,500 lines of Python with 316
  tests. **≈ $0.50 per 180-task run** with reasoning off. The ledger has
  56,370 events, 54,000 of them consumption, with 77 price changes and 27
  renewals. Every result file records commit, model, T′, T, seed and
  task-set hash.

## Results

- **Matched 2×2** (Table I; one configuration, one flag): tiered refresh
  **7 / 4** at 1 / 28 days, untiered **7 / 45**. At one day both give the same
  seven errors from the same seven tasks.
- **Mechanism, read directly from traces** (Table II, 28 days, tiered vs
  untiered): observed age for assignments and user status 1 vs 29 days,
  prices 5 vs 29, contract terms 29 vs 29. A 30-day TTL never lapses inside a
  28-day window.
- **Sweep** (Table III, tiered): 7 / 4 / 4 errors, accuracy 67.2 / 69.4 /
  65.0%, **0 TTL lapses** in every window. The 14-day run used reasoning mode
  on. The 28-day count is 4 under either setting.
- **Attribution.** Under tiering, **14 of 15** errors across the sweep came
  from **prices** (7-day tier): 9 on cost-center spend totals and 5 on
  highest-spending cost center, both aggregates. The 15th, the only cold-tier
  error in the data, came from a contract mutation that fell inside the
  window. Untiered at 28 days: user status 15, assignments 10, prices 20,
  contract terms 0. The hot tier supplies 25 of 45 once refresh is removed.
- **Accuracy by task tier at 28 days** (tiered vs untiered): single-source
  **80.2 vs 42.0%**, cross-source 47.6 vs 17.5%, cross-modality 55.6 vs 55.6%
  (the document index is rebuilt at T′ in both). Overall **65.0 vs 36.7%**.
- **Accuracy mostly measures registry coverage.** 13 of the 22 measures are
  unregistered, and 109 of 180 tasks fall through to model-generated SQL.
  Those tasks account for 49–53 of the 51–69 reasoning errors per run.
- **Transport failures.** Four runs, including the 45-error ablation, lost
  4–24 tasks to API instability. The ablation's failures surfaced only in a
  later audit. The affected tasks were re-run in isolation with the other
  results preserved byte-for-byte. Freshness counts did not change in any
  of the four.

## Critique / open questions

- **The headline mechanism is close to true by construction.** A scheduler
  that refreshes every lapsed entity bounds observed age by TTL, so a
  cache-age sweep of a maintained cache cannot move. The paper frames this as
  a design error others will repeat, which is fair. Its real content is the
  instrument, the 4 → 45 ablation, and the product-of-rates attribution, not
  the discovery that TTLs work.
- **T_eff = min(last-refresh) makes the freshness count a lower bound.** An
  answer that combines entities of different vintages, or live data at T
  with a staged entity, need not match gold at any single timestamp. Such an
  answer falls into "reasoning error." The authors don't discuss mixed-vintage
  answers, which are the cross-source tasks where staleness should
  concentrate.
- **Counts under tiering are single-digit**, and the 7 → 4 dip is unexplained.
  The authors rest the negative on the Table II traces and the ablation
  instead, which is the right call.
- **"Tier width × mutation rate" is qualitative.** It rests on three entity
  classes and one cold-tier error, with no fitted relationship and no
  TTL-grid sweep. The authors name that sweep as the next step. The claim
  that "the experiments below span a wide range around the defaults, so no
  conclusion rests on any one calibration choice" is not backed by any
  reported mutation-rate sweep. Every reported run uses one ledger.
- **The refresh ablation does not touch the document tier.** The index is
  rebuilt at T′ in both settings, so cross-modality accuracy is identical by
  design and says nothing about prose staleness.
- **The external validity is narrow**: one model, one framework, one
  synthetic domain, no cross-provider replication, and a non-agentic
  rule-based router doing most of the grounding. This is enterprise RAG
  grounding (cs.SE), not an ML-research agent. Transfer to this project is by
  analogy to maintained knowledge stores.
- **The supersession-vs-aggregate argument is asserted, not measured.**
  MemStrata is not run. The measured part is only that the residual errors
  under tiering were aggregates over a warm-tier entity.
- **The disclosures are candid.** Configuration mismatches, retries, a
  late-caught transport failure in the headline run, and "no baseline" are
  all stated up front. Generative AI was used for language editing and
  reviewed coding assistance.

## Trust signals

- **Credibility:** 2. Two independent researchers with no institutional
  affiliation and no track record in this graph; an arXiv cs.SE preprint
  that is not peer-reviewed; no citations established. Raised from 1 by a
  **released repository** (code, committed ledger and seed, per-error
  supplementary data, a reviewer guide mapping every number to a command;
  URL resolves) and unusually explicit provenance and limitation reporting.
  Held at 2 by the single model and synthetic domain, single-digit counts,
  no baseline, and an uncovered calibration-robustness claim.

## Follow-up

- **Relevance:** 3. Useful, cleanly instrumented prior art on staleness in
  maintained knowledge stores, the empirical partner to
  [[literature/papers/chen2026fresh]]. It adjusts two concepts at the margin
  but seeds nothing, and its domain is enterprise RAG rather than research
  agents. Chen shows freshness of state is not validity of what was derived
  from it. ChurnBench shows the opposite proxy, **age since build**, is also
  the wrong staleness variable for any store that is maintained. The
  operative quantity is time since last refresh relative to how fast the
  source changes.
- **[[concepts/verified-memory-writes]].** It adds a third form of the
  derived-artifact gap after chen's stale plan and
  [[literature/papers/shen2026revoked]]'s write-back journal: a
  **materialized aggregate** that per-fact supersession cannot reach. Under
  tiering, every warm-tier error was an aggregate. The concept's former
  sentence "`/lint` currently approximates staleness by file age" was
  **wrong** (corrected 2026-09-15). `~/claude-system/scripts/kg_lint.py` has no concept-staleness
  check. Its only age checks are backlog timers: uncurated candidates > 14
  days (managed repos), unengaged rel ≥ 4 literature > 30 days since
  `added:`, and in experiments mode, proposals > 14 days and expansions > 7
  days. The same false claim appears in chen2026fresh's Follow-up and its
  `_meta/index.md` line.
- **[[concepts/llm-wiki-pattern]].** It answers that concept's open question
  about where compile-time curation breaks down: when a compiled page's
  refresh interval overlaps its source's change rate, not merely when the
  source changes. The hot/warm/cold/live tiers line up with the OKM
  timeless / dated / pointer taxonomy already recorded there, plus the rule
  to classify from measured change rate.
- **[[concepts/selective-memory-retrieval]].** Its "retrieval cannot be
  evaluated on static stores" claim now has a benchmark behind it, and a
  judge-free diagnostic separating a stale retrieval from a reasoning fault.
  The benchmark-design warning carries over: check that the swept variable
  reaches the component under test before reading a null.
- **[[concepts/agent-native-memory]]: declined.** Its only time-based
  mechanism is ByteRover's recency decay for importance and maturity
  graduation, not validity, so the finding doesn't bear on it.
- **Candidates not in the graph:** MemStrata (Yadav, arXiv:2606.26511,
  bi-temporal supersession over agent memory, stale-fact-error rate) is the
  closer memory-side source. Liu et al. (arXiv:2604.05096, RAG vs learning
  under continuous knowledge drift) is also cited.
