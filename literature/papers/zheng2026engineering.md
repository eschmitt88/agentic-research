---
kind: paper
title: "Engineering Reliable Commit Gates for Agentic AI: Cost-Aware Verification Portfolios under Common-Mode Data Failures"
authors: ["Zihao Zheng", "Baichuan Li", "Junyi Yao", "Jiayu Long"]
institutions: ["Washington University in St. Louis", "Southern Methodist University"]
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.10969"
code_url: "https://doi.org/10.6084/m9.figshare.33511441.v1"
citations: null
source: "raw/papers/zheng2026engineering.pdf"
added: "2026-09-14"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["verification", "commit-gate", "provenance", "common-mode-failure", "source-independence", "atomicity", "idempotency", "risk-control", "negative-result"]
---

# Engineering Reliable Commit Gates for Agentic AI: Cost-Aware Verification Portfolios under Common-Mode Data Failures

## TL;DR

"Add another verifier" quietly assumes independent evidence. In a fixed-call
2×2 factorial on frozen agent proposals, a second verifier that reads an
**independent data source** cuts false approval of unsafe actions by
**40.9pp**, while switching the verifier's **model family** cuts it by
**11.3pp** (pooled: 74.2% same-model/same-source → 22.9% cross-model/independent).
A live HTTP/SQLite arm then shows the limit of *any* pre-action
verification: under a forced after-check race every verifier-only policy
commits the same 12 unsafe effects, and only a full atomic guard inside the
write transaction records **0 unsafe effects in 216 episodes**.

## Claims

- **Evidence lineage matters more than model identity.** Two verifiers that
  read interfaces backed by the same stale replica or cache agree on the same
  wrong state; more votes do not repair a common-mode data failure.
- The effect sits in the **evidence plumbing, not in correlated LLM errors**:
  a rule-based arm whose verifier errors are independent *by construction*
  reproduces it (65.9% cross-model approval on a shared source vs 22.8% with
  an independent source).
- An independent read helps **only if it has an independent failure path**
  (when every source degrades, the best cell is still 41.4%) and **only if
  the verifier is competent** for that evidence (external FinQA check
  reverses the effect).
- A cost-aware, auditable **portfolio controller** (a lookup table over
  deployment-observable metadata) improves the risk–cost trade-off over fixed
  checks — but its calibration is approximate and fails to transfer to unseen
  fault families.
- **Verification is not enforcement.** Evidence acquired before a concurrent
  change cannot enforce the state at execution; only a check inside the write
  transaction does, and only for the clauses it covers. When the whole
  predicate is checkable at commit, a full atomic guard is preferable and extra
  model checks add nothing.
- Returning better evidence to the actor does **not** replace the gate: most
  unsafe proposals are re-committed after one replan.

## Methods

- **VP-Control benchmark:** 48 task templates (12 each: schedulers, data
  pipelines, storage, IAM), each with a precondition predicate Φ, effect,
  blast radius, reversibility, and the subset of clauses a transactional guard
  can express (39 templates support a guard). 10 hash-seeded worlds × 6 fault
  regimes (*clean, a_stale, common_stale, partial_update, ambiguous_commit,
  all_degraded*) = **2,880 scenarios**, split by template 24/8/16
  train/calibration/test. Non-clean regimes contain a hazardous latent change
  with p=0.5, so regime does not reveal the label.
- **Evidence views:** A (actor's read), A′ (second interface, same upstream
  lineage as A), B (independently replicated snapshot with its own lag, lags
  with p=0.10). Plus an exact transactional guard and human deferral, each with
  a unit cost (A′=1, B=3, guard=2, deferral=10).
- **Models:** actors Qwen3.5-9B and Gemma4-8B; verifiers Qwen3-4B (V1) and
  Phi-4-mini (V2); Llama-3.2-3B as a weak-verifier stress case. All Q4_K_M
  Ollama checkpoints on one 16 GB laptop. **5,760 frozen proposals, 28,362
  cached verifier calls**; every controller replayed over identical proposals
  and verdicts.
- **2×2 design:** start with V1 on A′, add one of {V1 new seed on A′, V2 on
  A′, V1 on B, V2 on B}. Reports false-approve rate on unsafe proposals and
  error correlation.
- **Portfolio controller:** 37 small decision trees over five mechanism
  families; contexts are a six-level back-off hierarchy over observable
  metadata (age of A, last-tool status, source health, guard availability).
  Plan selection minimises c − w·q + ν·r̂ (w=4); calibrated with
  Learn-Then-Test using a design-effect-adjusted Wilson upper bound. A
  `ControllerView` strips hidden fields; static and dynamic leakage audits
  check this.
- **Statistics:** template is the unit; 2,000 cluster-bootstrap resamples,
  5,000 sign-flip permutations, three preregistered primary comparisons
  Holm-corrected, everything else labelled exploratory. A recomputation audit
  re-derived all 33 strategy rates from 900 sampled replay rows with zero
  mismatches.
- **Boundary checks:** domain and fault-family holdouts, verifier-prompt
  shift, weak-verifier swap, actor/verifier role swap, price variation, one
  bounded actor replan after receiving A′ or B, and a 240-question FinQA
  subset (960 scenarios) with controlled table corruption.
- **Live arm (preregistered):** loopback HTTP service over per-episode SQLite,
  a separate writer process, 12 business contracts in three families (schema
  migration, snapshot promotion, failed-job restart), six injected conditions
  (C0 clean, C1 benign change, C2 common stale cache, C3 after-check race via
  a synchronising barrier, C4 lost response with client retry, C5 degraded
  evidence). Eight policies, 216 episodes each; unsafe execution scored from
  the predicate at the transaction's linearization point.

## Results

- **RQ1, 2×2 (Table IV, pooled over 849 unsafe proposals):** same/same
  **74.2%**, cross-model/same-source **62.9%**, same-model/independent
  **33.3%**, cross/independent **22.9%**. Paired source effect at fixed model
  **−0.409 [−0.479, −0.331]** vs model effect **−0.113 [−0.215, −0.038]**;
  interaction 0.008, p=0.9056 (not detected). Under *common_stale*,
  cross-model approval falls from **81.7% → 7.3%** when one call uses B; under
  *all_degraded* the best cell stays at **41.4%**.
- **The verifiers are weak.** Single calls (Table III): V1 on A′ AUROC 0.555,
  false approve 75.0%; best single call V1 on B AUROC 0.666; V2 on B has
  62.8% false reject. A confidence-threshold gate is identical to always
  executing (44.2% unsafe).
- **RQ2, fixed policies (Table V, locked test):** always execute 44.2% unsafe;
  cross-model same-source vote 27.8% (cost 2); independent source 16.5% (V1)
  / 11.2% (V2) at cost 3; guard-then-B **0.8%** but defers 56.2% at cost
  8.82; random mechanism at matched cost 20.9%.
- **Portfolio @5% nominal:** **1.9% [1.0, 2.8] unsafe, 38.2% safe
  coverage, cost 4.96**. At the cross-model vote's budget it cuts unsafe
  27.8% → **12.7%** while raising coverage 40.6% → 51.7%; at the
  independent-source budget 16.5% → **5.7%** at similar coverage. All three
  preregistered comparisons significant (P1 −0.152, P2 −0.108, P3 −0.409;
  Holm p=0.0006). The 1% and 2% targets cannot be resolved with eight
  calibration templates — the controller defers on 99.9%.
- **RQ3, transfer is conditional:** domain holdouts — portfolio @5% stays in
  target on scheduler (1.0%) and storage (2.8%) but defers entirely on IAM and
  pipeline. Fault-family holdouts break it: unseen *ambiguous_commit* **26.2%**
  and *partial_update* **16.2%** unsafe at a 5% target. Stress refits stay at
  1.7–2.8%; pooling extra actor seeds makes the candidate family fail
  calibration, so it defers rather than extrapolates.
- **Replan does not replace the gate:** of 192 initially unsafe proposals,
  a same-lineage replan leaves **92.2%** unsafe; returning B lowers it to
  **69.8%**, with 30.2% abstention.
- **FinQA external check is negative:** actor wrong on 43.3% of uncorrupted
  tables; Phi-4-mini rejects every answer; Qwen3-4B approves 17% of unsafe
  answers from the corrupted shared rendering vs **20%** from the correct
  independent table — the source effect reverses. Zero-shot portfolio 5.1%
  unsafe at 9.4% coverage.
- **RQ4 live arm (Table IX):** plain commit 20.8% unsafe; shared-cache vote
  9.3%; independent-read vote 5.6% (L1 difference +0.037, Holm p=0.2667,
  **not significant** — Phi-4-mini rejects nearly every migration so the
  shared-cache vote completes only 7% of those safely). Partial guard as
  non-atomic preflight 16.2%, same clauses inside the write transaction
  13.0% (L2 Holm p=0.0960, not significant); **full atomic guard 0.0%
  (95% upper 1.4%) with 81.0% safe completion**, tied best.
- **After-check race (C3, 36 episodes):** every verifier-only policy commits
  the same **12** unsafe effects; preflight partial guard 16; atomic partial
  guard 9 (the remaining nine are clauses it does not express); full atomic
  guard 0. Uncontrolled random-timing runs (0–50 ms writer delays, 60 runs)
  produce only four after-check races — the forced race measures an exposed
  window, not its frequency.
- **Portfolio + atomic partial guard:** 4.2% unsafe (all nine in the race
  condition), 68.5% safe completion vs 2.8% / 41.2% for independent vote +
  guard — it buys availability (+0.27 safe completion), not lower risk (L3
  Holm p=0.5063).
- **Lost responses (C4):** idempotent request identifiers replay stored
  responses; 96 retried requests, no duplicate effects. Without them, retried
  snapshot promotions duplicate audit rows and pointer switches; retry count
  ≠ duplicate effects where DB constraints happen to act as an uncovered guard.
- **Cost of model diversity:** tool-only guards decide in milliseconds;
  shared-cache vote median 6.09 s, independent-read vote 6.38 s, portfolio
  3.44 s (2.44 / 2.43 / 1.34 s excluding model load). Live decisions matched
  cached verdicts 360/360.

## Critique / open questions

- **Toy verifiers.** 4B quantized models with AUROC 0.53–0.67 are barely
  above chance. The 11.3pp "model diversity" effect is a statement about
  *these* models; frontier verifiers might gain more from diversity (or less
  from an independent source they cannot interpret — FinQA already shows the
  latter). The rule-based arm protects the qualitative ordering, not the
  magnitudes.
- **Independence is supplied, not discovered.** B is labelled independent by
  the benchmark. The paper states plainly that lineage discovery and
  protection against forged provenance are outside the implementation — which
  is exactly the hard part for a deployed system, and the part
  [[literature/papers/he2026stored]]'s typed provenance graph proposes.
- **The headline source effect fails its only external test** (FinQA
  reversal). The authors report this and restrict the claim to domains where
  verifiers can interpret the evidence. That is honest and it also means the
  40.9pp number should not be quoted without that qualifier.
- **The live-arm comparisons that distinguish policies mostly miss
  significance** after Holm (L1 p=0.27, L2 p=0.096, L3 p=0.51) with 12
  contracts. The one clean live result is the zero for the full atomic guard,
  and that is a well-known database principle (the paper says so) rather than
  an agent finding — the contribution is the head-to-head evidence that
  verification portfolios do not substitute for it.
- **Calibration is approximate, not a guarantee.** Nominal 5% → observed 1.9%
  in-distribution, but 26% on an unseen fault family. A risk target learned
  on one incident population is not a property of the gate.
- Synthetic, controlled world; one writer process; loopback service; no
  distributed failures, adversarial writers, or long-horizon rollouts. Unit
  costs are modelled; human deferral is modelled as always safe.
- A small number of process deviations are disclosed (Stage-0 parameter
  retuning, a single verifier-prompt change after a training-only pilot,
  an injector crash repaired and resumed) — logged as preregistration
  deviations rather than hidden.

## Trust signals

- **Credibility:** 3 — arXiv cs.SE preprint, not peer reviewed; authors at
  Washington University in St. Louis and Southern Methodist University with no
  named lab; no citations yet. Raised by unusually strong reproducibility
  discipline: frozen proposals and cached verdicts, a Figshare archive with
  prompts, model digests, raw JSONL, calibration traces and per-episode SQLite
  logs, preregistered and Holm-corrected primary comparisons, leakage and
  recomputation audits (33 rates, zero mismatches), and forthright negative
  results (FinQA reversal, fault-holdout failure, non-significant live
  comparisons). Held down by tiny quantized verifiers and a fully synthetic
  benchmark.

## Follow-up

- **Relevance:** 4 — the **first controlled measurement** of the
  "source count is not source independence" point that
  [[concepts/shared-substrate-contagion]] currently carries only as an argument
  from [[literature/papers/he2026stored]]; this is an independent group with a
  factorial design, not a restatement. It also supplies the head-to-head
  placement comparison [[concepts/enforcement-boundary-placement]] lists as
  missing (pre-action verifier vs non-atomic preflight vs in-transaction
  guard, same proposals), and sharpens that concept's temporal rule from "when
  authority is used" to "inside the effect's transaction."
- **Repo implication:** a re-check of a concept update that reads the
  literature note (written by the same ingest pass) is an A′ read — same
  lineage. An independent check must go back to `raw/`. By this paper's
  numbers, swapping the reviewing model buys roughly a quarter of what
  swapping the evidence source buys.
- Candidates from this paper's bibliography, not yet in the graph: Bara, "Epistemic Sybil resistance" (arXiv 2609.01873) proves a
  report-only aggregator cannot distinguish independent corroboration from
  duplicated reports — the formal version of the same point. Also
  SafeCommit (arXiv 2608.04289) and Han, "Partially correlated verifier
  cascades" (arXiv 2607.13918).
- Open for [[concepts/evidence-gated-completion]]: the false-reject side is
  now priced for one setting (V2 on B: 62.8% false reject; guard-then-B defers
  56.2%), which is the cost that concept's open question asks for.
