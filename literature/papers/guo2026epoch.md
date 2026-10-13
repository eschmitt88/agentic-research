---
kind: paper
title: "EPOCH: Reliable Discovery through Evidence-Governed Search"
authors: ["Binjie Guo", "Aisheng Mo", "Ruitong Li", "Xinle Deng"]
institutions: ["Zhejiang University", "The University of Hong Kong"]
year: 2026
venue: "arXiv 2610.06986 (cs.AI), v1 2026-10-04, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.06986"
code_url: null
citations: null
source: "raw/papers/guo2026epoch.pdf"
added: "2026-10-13"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["evidence-governance", "task-contract", "admission-gate", "three-valued-gate", "falsification", "typed-memory", "independent-replay", "claim-sealing", "claim-scope", "mutation-controls", "algotune", "agenthpo", "math-discovery", "lean", "evolutionary-search", "ablation", "min-max-normalization"]
---

# EPOCH: Reliable Discovery through Evidence-Governed Search

## TL;DR

EPOCH wraps an LLM discovery loop in an evidence protocol. A frozen task
contract fixes the editable artifact, the checks, the budget and the
admissible claim scope. Every trial, failed or not, goes into an
append-only typed ledger. Each proposal names a hypothesis, a predicted
effect, the evidence it relies on and a falsifier aimed at its mechanism.
A three-valued gate (pass / fail / unresolved) admits a candidate only
when every required check passes. The selected endpoint is then frozen,
replayed out of loop, and sealed together with its claim.

The architecture is a clean composition of patterns this graph already
tracks. The evidence that the governance layer is what produces the gains
is weak:

1. The AlgoTune "SOTA" (0.65 vs 0.53) is on a self-run 12-task subset with
   3 seeds and no intervals.
2. The held-out replay edge over the strongest baseline (+0.04) comes from
   one task.
3. The Math14 lead over OpenEvolve is +0.01, with a 4/3/7 losing task
   record.
4. Component ablations exist only on six AgentHPO tasks, scored with a
   min-max normalisation that turns 0.01 raw differences into tens of
   points.

The ten "discovery" cases are heterogeneous, uncontrolled dossiers. By the
paper's own Table S9, none of them has a complete final bridge to its
stated claim.

## Claims

- **Discovery needs an evidence protocol, not just an evaluator.** "A
  scalar reward can guide a search procedure toward better candidates, but
  it cannot by itself organize the generation of supporting evidence,
  specify the scope of a claim, or preserve the failed attempts."
- **Endpoint selection is constrained maximisation (Eq. 1).** The arg-max
  of the objective runs over proposals from the registered run only. Each
  must pass Correct, Challenge, Replay and Supports(ξ, σ), with
  Scope(σ) ⊆ S. Gates take values in {1, 0, ⊥}, and only 1 admits.
- **Safety invariant (D.5).** If the baseline is verified and every
  replacement needs every gate to pass, then by induction the retained
  endpoint always has its own passing receipt. The authors say this
  "establishes protocol compliance rather than external truth".
- **Results.** State-of-the-art aggregate on AlgoTune, highest mean on
  Math14, "favorable" held-out replay, the leading "descriptive aggregate"
  on AgentHPO, and ten discovery problems. The ten include 11 improved
  tensor border-rank bounds, V₁₂(23) ≤ 54 for median-of-23 selection, and a
  1,512-gate (14.10%) T-count reduction for an AES circuit. There is also a
  Lean-checked hypercube determinant inequality and a Lean-checked
  nine-element counterexample to an H_β coefficient ordering.

## Methods

- **Task contract.** C = (D_pub, D_term, E, B, A, S): public and terminal
  data views, evaluators, budget, editable artifact space, allowed claim
  scope. The public view can shape proposals. The terminal view only scores
  the frozen endpoint. If test feedback prompts a new candidate, that is a
  new experiment, not the original replay.
- **Typed evidence ledger.** Each event records artifact and evaluator
  hashes, score, correctness, resource use, failure attribution and
  rejection reason. **Infrastructure incidents are attributed separately
  from scientific failures.** An incident triggers bounded recovery and
  leaves admission unresolved, rather than counting as evidence against
  the method. The model sees a bounded projection of the ledger. The
  auditability claim is about preserving the full record, not about the
  model seeing all of it.
- **Proposal schema.** Each proposal carries a candidate, a hypothesis, a
  predicted effect, evidence references (pointers to immutable events, not
  prose like "the previous attempt worked") and an intended falsifier. The
  falsifier targets the mechanism. For example, an FFT convolution must
  survive a wraparound-mismatch check even when its timing improves.
- **Phase controller.** Explore / refine / recover / finalize. The authors
  say it "does not imply an unreported learned optimal schedule".
- **Admission (D.3).** The gates are a conjunction, not an average. Any
  demonstrated contract violation fails. A failed or unresolved candidate
  never replaces the verified incumbent. "A candidate can be informative
  and still be ineligible for replacement."
- **Replay independence ladder (D.7).**
  - *Process*: rerun the same candidate in a clean process.
  - *Implementation*: rebuild the target with another program.
  - *Representation*: use a different computational model.
  - *Formal reflection*: move the finite checks into Lean.
  Each level checks a stated interface and no more. A QASM truth-table
  replay checks an emitted circuit, not global minimality.
- **Discovery evidence layers (B.4).** Semantic reconstruction, exact or
  exhaustive replay, mutation controls ("sensitivity is not
  completeness"), and a formal or analytic bridge to the stated claim.
  Finite-certificate sufficiency (S3) holds only if the checker is sound
  *and* the enumeration is complete. Both are premises, "not consequences
  of a large pass count".
- **Benchmarks.**
  - AlgoTune: a frozen 12-task subset; public search on the first 8 train
    rows; 48 "semantic calls" per method; DeepSeek-V4-Flash; 3 seeds.
    Replay uses the first 32 official test rows for "eligible" endpoints,
    and the reported replay covers 7 tasks.
  - Math14: 14 "project-frozen" objectives × 5 methods × 3 seeds, with no
    train/test partition.
  - AgentHPOBench: 6 "reported" environments, used only for EPOCH's
    component-removal variants. Seeds and budget are not stated.
  - Baselines are AI-Scientist v1/v2, OpenEvolve and AdaEvolve (via a
    SkyDiscover adapter), all run by the authors.

## Results

- **AlgoTune development (Table 1/S3).** 0.65 vs AdaEvolve 0.53, AI-Sci v1
  0.52, OpenEvolve 0.50, AI-Sci v2 0.50. There are no CIs or tests. Table S4
  gives "original reported effects" (+0.16 to +0.20) from "a separate
  cohort" next to "complete development effects" (+0.12 to +0.15). Task
  records against the four baselines are 6/0/6, 6/0/6, 10/0/2 and 8/1/3.
  Per the performance profile, EPOCH scores ≥ 0.60 on 5 of 12 tasks. Most
  tasks for every method sit at the 0.5 baseline.
- **AlgoTune replay (Fig. S3, 7 tasks).** The mean effect against
  AdaEvolve is +0.04. Its per-task effects are 0.000, 0.000, +0.048,
  −0.002, **+0.475** (maximum common subgraph), **−0.270** (NMF), 0.000.
  Without the one MCS task the mean is about −0.03 (our arithmetic). The
  Convolve2D gain ties AdaEvolve, which "later produced a clean generic
  solution". Appendix E says the AlgoTune gains are established techniques
  (zero-padded FFT, LSODA, Bron–Kerbosch clique search), "not counted among"
  the discoveries.
- **Math14 (Table S5).** EPOCH 0.57, OpenEvolve 0.56, AdaEvolve 0.53, AI-Sci
  v2 0.52, AI-Sci v1 0.51. Task-level W/T/L: 7/4/3 against AI-Sci v1, 7/5/2
  against v2, **4/3/7 against OpenEvolve, 5/3/6 against AdaEvolve.** The
  authors concede "task-level comparisons are mixed".
- **AgentHPO ablations (Table 3, S6, S7).**

  | Variant | Norm. mean | Full W/T/L | exact sign test (ours) |
  |---|---|---|---|
  | Full | 77.48 | – | – |
  | No memory | 64.32 | 4/0/2 | p ≈ 0.69 |
  | No proof admission | 40.21 | 6/0/0 | p ≈ 0.03 |
  | No falsifier | 35.74 | 4/1/1 | p ≈ 0.38 |
  | No phase controller | 21.89 | 5/0/1 | p ≈ 0.22 |

  The normalisation (Eq. 4) is min-max *across the five variants within
  each task*, so on every task the worst variant scores 0 by construction.
  The raw means (S6) show how much that inflates. On NoisyGL Cora, Full is
  0.71 and every ablation 0.70. SparseTSF is 0.33 against 0.33/0.33/0.33/0.34,
  and TabM is 0.52 against 0.53–0.55. Real separation appears on ART 2048
  (win rate 0.07 vs 0.03, but no-memory is 0.15) and RankUp (MAE 17.13 vs
  19.20–30.56). No-memory beats Full on 2 of 6 tasks. No-phase beats Full on
  MNLI.
- **Mechanism traces (Table S8).** Prior-evidence citation is 80.0% vs 42.5%
  without memory. Repeated configurations fall 10–16 pp with the falsifier,
  admission and phase controller. Admission rejects 34.67% of proposals,
  against 0% with admission removed.
- **Discovery cases (Table 4, S9).**
  - All ten have the statement, algebraic reduction and finite/exact
    verification in scope.
  - The formal kernel is in scope for only 3 of them.
  - The **final bridge** is "Partial" or "Open" for **all ten**. The
    median-of-23, Heisenberg and online-embedding bridges are "Open".
  - "External record priority remains a separate question." The
    median-of-23 comparator is Hadian–Sobel (1969), and the
    selection-literature review "remains separate from the certificate".
- **AES factor sharing.** 10,746 → 9,234 T gates under matched PyZX, all 256
  input/output cases checked and ancillas restored. Optimality holds "only
  to the registered factor library and cost model". Note 1,512/10,746 =
  14.07%, not the stated 14.10% (our arithmetic).

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing, and the
  abstract overstates.** The digest reported "SOTA on AlgoTune (0.65 vs
  0.53)", "the top score on an internal Math14 suite" and "held-out
  official-test replay" without qualification. Each is the
  most-favourable cut:
  1. **AlgoTune SOTA** is a self-run 12-task subset with one model, 3 seeds,
     no intervals, a 0–1 normalised evaluator score (reference = 0.5) rather
     than AlgoTune's native speedup, and baselines run through adapters by the authors.
  2. **"Favorable held-out behavior"** is 7 of the 12 tasks. Against the
     strongest baseline its sign rests on maximum common subgraph alone.
     Table 1's caption also says development and replay "use distinct
     endpoint populations", so the replayed endpoints are not the ones the
     0.65 figure was computed from.
  3. **"Highest mean on Math14"** is +0.01 over OpenEvolve, and EPOCH loses
     more tasks to OpenEvolve and AdaEvolve than it wins.
  4. **"Leads the descriptive aggregate on AgentHPO"** reads like a
     comparison with other systems. It is only EPOCH against its own
     ablations.

  This fits the graph's abstracts-overstate pattern.
- **The digest's ingest question gets a no: nothing isolates the
  governance layer from search budget.**
  - AlgoTune is matched on 48 "semantic calls". The paper never says
    whether falsifier evaluations, challenge runs and replay compute count
    against that budget. If they do not, EPOCH gets more evaluator compute
    per call.
  - AlgoTune and Math14 have no component ablations.
  - The only ablations are on AgentHPO, with no stated budget or seeds and
    no same-budget unconstrained arm. That is the control
    [[literature/papers/wang2026rethinking]] showed can erase an evolution
    gain.
  - "Six reported environments" leaves open whether more AgentHPO tasks
    were run.
- **The ablation magnitudes are artefacts of the normalisation.** Within-set
  min-max over five variants guarantees a 0 on every task. The headline
  "−55.59 points without the phase controller" includes tasks where the raw
  gap is 0.01 RMSE or 0.01 accuracy, well inside any plausible seed noise.
  Only one comparison (no proof admission, 6/0/0) clears a sign test, at
  p ≈ 0.03 over 6 tasks.
- **"Proof admission" on hyperparameter tuning is unclear.** The gate
  rejects 35% of HPO proposals, but the paper does not say what an HPO
  "proof" obligation is. So the 6/0/0 cannot be tied to a specific check.
- **The discovery portfolio is not an evaluation of EPOCH.**
  - Budgets are "heterogeneous".
  - The content is "reproduced and organized from the supplied research
    dossier".
  - The AI-use statement says generative AI tools helped develop "candidate
    programs, constructions, mathematical arguments, and proofs", without
    separating EPOCH runs from other assistance or from human work.
  - There is no baseline system on any case and no novelty check against
    the literature. Yet the contributions call median-of-23 "record-setting".
  - The 25 auxiliary results in Appendix K add volume, not evidence about
    the architecture.
- **Credit where due: the evidence-boundary discipline is careful and
  importable.** The paper separates finite computation, general argument,
  formal kernel and missing bridge. It refuses to let a finite pass count
  stand in for a theorem, says a primary-complete result whose independent
  Z3 exclusion timed out "must remain primary-complete rather than
  dual-verified", and keeps "unresolved" distinct from "failed". That is a
  stricter scope vocabulary than most sources in
  [[concepts/typed-claim-partition]]. It is a design contribution, and the
  benchmarks do not test it.
- **Compared with nearby graph sources.** [[literature/papers/waks2026what]]
  specifies the same frozen-contract and claim-scope-ceiling idea with a
  deterministic checker on fixtures. [[literature/papers/ning2026scores]]
  certifies discovery post hoc with matched no-history challengers. EPOCH
  is the first to put the contract *inside* a search loop, as the
  replacement rule. It lacks both of their controls: a fault-injection test
  of the gate, and a recovery arm asking whether the result needed the run.
- **Open:** does the three-valued admission gate change which endpoint wins
  at matched evaluator compute? Does infrastructure-vs-science attribution
  measurably reduce false rejections? Neither is tested.

## Trust signals

- **Credibility:** 2.
  - For: two reputable universities; a detailed protocol appendix; a
    registered model/seed schedule; candid scope language on the
    mathematical cases; Lean kernels for several finite claims.
  - Against: an unreviewed preprint with no code, artifact link or project
    page, although the reproducibility statement refers to "supplementary
    material". All baselines were run by the authors. There are no
    confidence intervals or tests on any headline benchmark. Development and
    replay endpoint populations differ, and Table S4 has "original" and
    "complete" effect cohorts. Math14 is internal. The ablation metric
    inflates small raw gaps. The discovery dossiers are uncontrolled, with
    no attribution of AI or human assistance.
  - Not 3, because a reputable affiliation does not offset missing code
    and missing uncertainty on every comparison.

## Follow-up

- **Relevance:** 3. This is the most complete *architecture* in the graph
  that combines [[concepts/evidence-gated-completion]],
  [[concepts/typed-claim-partition]] and in-loop falsification around an
  AlphaEvolve-style search ([[literature/papers/novikov2025alphaevolve]],
  [[concepts/evolutionary-expansion]]). Its design vocabulary is worth
  citing: three-valued gates, infrastructure-vs-science attribution, a
  replay independence ladder, and finite-certificate premises. It does not
  shift any concept's evidence grade. The one refusal-removal ablation is
  on six tasks with tiny raw deltas, an unstated budget, and the authors'
  own normalisation. It does not lift the evidence-gated-completion hold.
- **Importable now (design, not evidence):**
  - Record failure attribution on every evaluation (scientific vs
    infrastructure). Map infrastructure failures to *unresolved*, never to
    a score or a rejection.
  - Evidence references in proposals should point at immutable event IDs,
    not prose.
  - Name the replay level a check provides: process, implementation,
    representation or formal.
- **Would raise relevance to 4:** released code; CIs over seeds; a
  same-evaluator-compute unconstrained arm on AlgoTune; ablations on
  AlgoTune itself, scored on raw metrics.
- **Candidates** (cited, not in the graph): AdaEvolve (Cemri et al. 2026,
  arXiv 2602.20133) as a baseline system; AgentHPOBench (OpenMOSS 2026)
  as an HPO-agent benchmark; Dörrer et al. 2025 (SEA), exact selection
  lower bounds, which is what the median-of-23 priority question needs.
