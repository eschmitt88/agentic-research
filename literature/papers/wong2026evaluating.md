---
kind: paper
title: "Evaluating Governed LLM-Driven ML Experimentation: Lifecycle, Provenance, and Failure Metrics"
authors: ["Dowling Wong"]
institutions: ["Karlsruhe Institute of Technology"]
year: 2026
venue: "arXiv 2611.23678 (cs.AI), v1 2026-11-18. Extended version of a paper at the KDD 2026 Workshop on Evaluation and Trustworthiness of Agentic AI (non-archival); this version rewords claims and corrects the workshop RBR counts"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.23678"
code_url: "https://github.com/dowlingwong/autoresearch_harness"    # MIT. Ledgers in release kdd-etaai-arxiv-v1 (commit 0455171), Zenodo DOI 10.5281/zenodo.24174991. Not opened at ingest.
citations: null
source: "raw/papers/wong2026evaluating.pdf"
added: "2026-11-23"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/pass-at-k]]"
tags: ["governed-harness", "control-plane", "append-only-ledger", "attempt-accounting", "trial-lifecycle", "pending-guard", "keep-discard", "failure-taxonomy", "provenance-completeness", "scope-whitelist", "node-state-hash", "reset-verification", "ungoverned-counterfactual", "repeated-bad-rate", "memory-ablation", "autoresearch", "nanochat", "mlagentbench", "openml", "deepseek-v4-flash", "qwen2.5-coder", "single-author", "workshop"]
---

# Evaluating Governed LLM-Driven ML Experimentation: Lifecycle, Provenance, and Failure Metrics

## TL;DR

A harness for autonomous hyperparameter/edit loops in which a fixed
**control plane**, not the proposing LLM ("manager") or the executor
("worker"), owns the trial lifecycle. It writes a pending-guard file
before each trial, checks the patch against a whitelist of editable
files, decides keep or discard with a pure function
(`candidate > current_best`), and appends exactly one JSONL record per
opened budget slot: `kept`, `discarded` or `failed_invalid` with a
failure category and five hash-derived provenance IDs. The paper proposes
reporting governance metrics (acceptance rate, invalid rate, repeated-bad
rate, provenance completeness, failure taxonomy) beside the task score.
Across six nodes and 1,445 trials, provenance completeness is 100%, but
that holds **by construction of the invariant**, not as an empirical
finding, and the paper says so. The empirically interesting parts are
smaller: a per-trial hash of the editable files caught two reset
failures that made "replicates" non-independent, and a 5-seed L40S
replication shows that putting trial history in the manager's context
lifts best AUC from 0.899 to about 0.94 on one node.

## Claims

- An ML-experiment agent should be judged on its process (bounded,
  auditable, classified, recomputable) as well as its best score. Five
  desiderata: D1 bounded edit scope, D2 complete attempt accounting, D3
  independent adjudication, D4 provenance linkage, D5 recomputability
  from the ledger without re-running training (Table 1).
- **Replacement principle:** with authority in the control plane, the
  manager (heuristic, prompt manager, LangGraph, AIDE-style search) can be
  swapped without changing what is recorded.
- **Lifecycle invariant:** every budget slot that enters `EXECUTING`
  produces exactly one terminal record before the next slot opens; none is
  skipped, duplicated or overwritten. A mid-trial crash leaves the guard
  file, and the next campaign refuses to start (`PendingTrialError`) until
  `recover_pending.py` appends a `failed_invalid` record.
- A valid non-improving trial is `discarded` (label `degraded_metric`),
  never `failed_invalid`. Treating a discard as a failure would inflate
  the invalid rate and blur "bad proposal" with "bad outcome" (App. E.3).
- Against AIDE (commit `ad73d22`): the journal snapshot is overwritten,
  not appended, there is no pending-trial guard, and a failed node carries
  only a `buggy` flag. A crash loses the step in progress.
- Memory of governed outcomes is a *diagnostic probe*, not a proposed
  method. The paper's Table 10 grades its own claims: the control-plane
  properties are "demonstrated"; repeated-bad-rate sensitivity to memory
  is "mixed"; "rationale reduces repeated-bad rate" is "not claimed".

## Methods

- **Roles.** Manager proposes one bounded change (summary, rationale,
  target, intended effect). Worker applies and runs it: `LocalWorker`
  executes "change PARAM from X to Y" directives; `ClawWorker` routes
  proposals through an external coding-agent path with a deterministic
  constant-patch bridge (Ollama). Control plane owns everything else.
- **Pre-run guards (pseudocode, Fig. S6):** scope check against
  `editable_paths` / `frozen_paths`, no-op check (byte-identical patch),
  precondition check (malformed fields, e.g. `new_value == old_value`).
  **Post-run:** runtime error, missing metric, invalid config, edit
  failure; then the decision rule. The manager is never consulted at any
  classification step.
- **Nodes (Table 3), one YAML each, same contract:** `resnet_trigger`
  (detector waveform classification, `val_auc`, only `train.py` editable);
  `lr_synthetic` (NumPy logistic regression, four editable symbols);
  `openml_credit_g` and `openml_bank_marketing` (`config.yaml`
  hyperparameters only); `mlagentbench_vectorization` (a NumPy-convolution
  speed task wrapped as a node, `config.yaml` only, "a compatibility
  probe, not a full MLAgentBench benchmark claim"); `autoresearch_linux`
  (Karpathy autoresearch GPT pretraining on climbmix shards, `val_bpb`,
  ~6 min/trial, only `train.py` hyperparameters editable).
- **Models.** Cross-node campaigns: `deepseek-v4-flash` via API as
  manager. LangGraph memory ablation: `qwen2.5-coder:7b`, temperature 0.7,
  with an explicit AVOIDANCE RULE in the prompt when memory is non-empty.
  L40S replication: DeepSeek manager, `qwen2.5-coder:7b` worker at 0.2.
- **Memory modes:** `none` (only current editable values), `summary` (one
  line per prior trial: decision, summary, metrics), `rationale`
  (summary plus per-trial rationale and an `[AVOID: ...]` line when the
  repeated-bad detector fires).
- **Repeated-bad rate** (App. C.7): a proposal repeats a prior discarded
  or failed proposal's target and direction (Jaccard ≥ 0.6 on unigram
  summary tokens, direction by keyword regex) without a rationale that
  references the rejection. RBR = count / N.
- **Ungoverned counterfactual.** An independent matched campaign with the
  pending guard, failure-record appends and blocking scope enforcement
  disabled; a separate observation log (`*_ungoverned_obs.jsonl`) records
  the true denominator. Two nodes, chosen to span failure rates.
- **Validation probes:** two adapter reruns; a re-execution script that
  rebuilds kept-trial state from patches; a rule-based structural
  relabeler on 50 stratified `failed_invalid` records that sees only
  observable fields (scope flags, patch presence, training start, exit
  status), never `failure_category`.
- **Reset verification (App. A.6):** each record carries
  `node_state_hash`, the SHA-256 of the editable file set just before the
  worker runs. Equal hashes at `budget_index == 1` across seeds and arms
  are the check that a multi-arm comparison started from one state.
- Bootstrap CIs: 10,000 seed-level resamples, seed 42.

## Results

- **Scale (Table 7).** 1,445 primary trials: ResNet case study 20
  (AR 20%, IR 55%), ResNet L40S 225 (AR 46.7%, IR 0.4%), `lr_synthetic`
  450 (8.4% / 46.2%), credit-g 230 (13.5% / 26.1%), bank 230 (13.0% /
  32.2%), MLAgentBench adapter 50 (28% / 48%), autoresearch `none` 120
  (0% / 100%), autoresearch `summary` 120 (11.7% / 40.8%). Provenance
  completeness 100% on every row.
- **Outcome mix (§5.3).** 242 `runtime_error` (16.7%), 220
  `proposal_precondition_failed` (15.2%), 85 `invalid_config` (5.9%),
  662 `discarded` (45.8%); the remaining ~236 (16.3%) are kept (our
  arithmetic). `invalid_edit_scope` and `no_op_patch` occur only in the
  1-trial forced stress campaigns and one no-op pair in a non-primary
  ablation replicate. **No live proposal in the primary set ever tried to
  touch a frozen file.** `metric_missing` and `edit_failed` never occurred.
- **Ungoverned counterfactual (Table 9).** `autoresearch_linux`: governed
  completeness 1.00, ungoverned 0.00 (30 of 30 dropped, all trials failed).
  `openml_bank_mkt`: 1.00 vs 0.35 [0.15, 0.55] (13 of 20 dropped, the
  ledger keeps only the 7 valid trials). Combined 43/50 vs 0/50. The paper
  states that the dropped trials are "by construction, exactly that arm's
  failed trials" and that this "shows what such a loop loses, not an
  independent estimate of loss".
- **ResNet case study (Table 6).** 4 kept / 5 discarded / 11
  `proposal_precondition_failed`. Best `val_auc` 0.7747 → 0.7827
  (+0.008), inside the 5-seed baseline noise (bootstrap CI of the mean
  [0.775, 0.798], min–max [0.771, 0.811]). The paper calls the gain
  secondary evidence that the loop runs, not an optimization claim.
- **MLAgentBench adapter.** 2 kept, 4 discarded, 24 failed-invalid in 30
  trials; the high IR is bounded candidates running out
  (`proposal_precondition_failed`), not task failure.
- **OpenML.** Bank marketing mean best AUC 0.9295 (b20) → 0.9341 (b30),
  95% CI [0.9337, 0.9345] at b30; credit-g flat (0.7671 → 0.7679). IR
  rises with budget (bank 17.5% → 40.0%) as bounded candidates exhaust.
- **LangGraph memory ablation (Table S4, ResNet CPU fast mode, budgets
  10/10/20).** Hypothesis RBR(none) > RBR(summary) > RBR(rationale).
  Summary has the lowest RBR in rep1 (0.50 vs 0.80/0.80) and rep3 (0.80
  vs 0.85/0.85). The full ordering holds only in rep2 (0.80/0.70/0.60),
  and `node_state_hash` shows rep2's arms were **not reset** between
  campaigns (the rationale arm started from the summary arm's best
  state). On `lr_synthetic` the ordering reverses (rationale 0.80, none
  and summary 0.70). A 50-token truncated rationale matched the full
  rationale arm (0.80), but both were stuck on an edit that has no effect
  in the CPU fast-search mode (`BATCH_SIZE`), so the comparison is weak.
  These campaigns are not in the 1,445 and their ledgers are only in an
  archive commit (`62e93a8`), not the tagged release.
- **L40S replication (Table S5; 3 modes × 5 seeds × 15 trials).**
  Trial-1 `val_auc` 0.886–0.890 for every arm and seed, shared hash
  `9c0118f33fcb`. Mean best AUC: none 0.899 [0.892, 0.907], σ 0.009;
  summary 0.939 [0.937, 0.940], σ 0.002; rationale 0.941 [0.940, 0.943],
  σ 0.002. Acceptance 17.3% / 64.0% / 58.7%. Memory arms separate by
  trial 5 (Fig. S4). Summary ≈ rationale (0.002 is not treated as
  meaningful). **In a first run, the `train.py` reset had failed** in all
  15 campaigns (s1–s3 started from a different file, s4–s5 from a
  modified baseline); those ledgers were deleted and all 15 rerun.
- **autoresearch_linux (Table S6).** `none`: all 4 seeds × 30 trials
  `runtime_error`, cause undetermined. `summary`: s1/s3/s4 improve
  `val_bpb` by −0.062, −0.061, −0.064; s2 is also a total worker failure
  (27 `runtime_error` + 3 precondition). These campaigns predate the
  frozen-baseline restore protocol and trial-1 hashes do not verify a
  common start, so the paper reports both failures as "observed failures
  of undetermined cause (an infrastructure or protocol anomaly cannot be
  excluded)", not memory evidence. One of the 30 s2 records was written
  by the crash-recovery tool (budget index 1 missing, index 2 twice).
- **Label reliability (App. E.2).** Structural relabeler vs control-plane
  labels on 50 stratified `failed_invalid` records: κ = 0.830 [0.703,
  0.958]. Perfect on `invalid_config`, `invalid_edit_scope`,
  `proposal_precondition_failed`; the six disagreements are
  `runtime_error` / `no_op_patch` edge cases whose evidence lives in the
  referenced artifact files, not the JSONL record.

## Critique / open questions

- **Against the digest entry: below it.** The entry quotes the abstract
  accurately, but three of its framings overstate what the full text
  supports:
  1. **"In a matched counterfactual, a loop that logs only successful
     trials left no record of 43 of 50 trials" is definitional.** The
     ungoverned arm is built to not append failures, so its completeness
     is 1 − failure rate, and the two nodes were picked to span failure
     rates (one at 100%). The paper says this is not an estimate of loss.
     It demonstrates the mechanism and measures nothing about prevalence.
  2. **"The ledger exposed failures that a task score hides: a no-memory
     arm where all 120 trials failed."** The ledger *recorded and
     classified* the 120 failures as `runtime_error`; it did not diagnose
     them. The cause is undetermined, the campaigns predate the reset
     protocol, and they were not rerun. A task score would also have shown
     no improvement. What the ledger adds is that the arm ran 120 trials
     and every one crashed, instead of an empty or flat curve.
  3. **"Memory ablations show that memory's effect depends on the task"**
     rests on three ResNet replicates (one non-independent) and one
     `lr_synthetic` run of 10 trials per arm. The robust memory result is
     the L40S one the entry omits: history in context lifted mean best AUC
     0.899 → ~0.94 with 4–5× smaller seed spread, on one node with one
     manager.
  - "All 1,445 trial ledgers released" is right for the primary set. The
    Table S4 ResNet ablation ledgers sit only in an archive commit, and
    patch and run-log artifacts are not released, so failure *causes*
    cannot be re-checked.
- **The `hce-score-coverage-count` link is weaker than the entry's Why.**
  The proposal targets a *score* whose ungraded items sit at zero or drop
  out of the denominator. Wong's ledger is about *trial attempts* in a
  keep-best loop. Three differences (our reading):
  - In a keep-best loop, dropping failed trials does not move the best
    score at all, because the keep rule only sees valid trials. It biases
    the *rates* (acceptance, invalid) upward, the opposite direction from
    Liu's zeros. The proposal's "or quietly dropped from the denominator"
    branch covers it, but the effect is on process metrics, not the
    ranking of arms by score.
  - The supporting number (43/50) is by construction. It is a worked
    mechanism, not evidence of prevalence, and should not count toward an
    evidence gate as a measurement.
  - **It complicates the proposal's central distinction.** Wong's split is
    valid-but-worse (`discarded`) vs invalid (`failed_invalid`). Inside
    `failed_invalid` it puts agent-caused faults (malformed proposals,
    out-of-scope edits) beside crashes whose cause may be infrastructure.
    The one large crash set (120 `runtime_error`) could not be attributed
    to either, which is exactly the proposal's "is a timeout a bad
    submission or a missing grade?" risk, met in practice and left open.
    The workable form here was recording a typed category per attempt,
    not classifying the attempt as score-or-missing.
  - What it does support: n_attempted should be a structural count the
    harness writes per opened slot (pending guard plus one terminal
    record), not something reconstructed from what the scorer emitted.
- **The strongest transferable evidence is the start-state hash.** It
  caught two reset failures: rep2 of the LangGraph ablation (the only
  replicate that "confirmed" the hypothesis) and all 15 first-run L40S
  campaigns. The workshop version had reported the rep2 ordering. Without
  the hash, an arm comparison would have mixed different start states and
  been reported as replicated.
- **The scope guard was never tested by a real proposal.** Bounded
  workers (parameter directives, a constant-patch bridge, `config.yaml`
  only on three nodes) make out-of-scope edits nearly impossible, and
  `invalid_edit_scope` occurs only in a 1-trial forced campaign. Nothing
  here shows that the guard holds against a free-form coding agent under
  optimization pressure.
- The κ = 0.83 is a consistency check between two rule sets over the
  same fields, not human agreement; the paper says so.
- The governance metrics are not shown to help anyone. The paper lists
  a human-auditor study (do reviewers find seeded errors faster with the
  ledger?) as future work.
- AIDE comparison is by reading its code, not by running it under the
  same nodes.

## Trust signals

- **Credibility:** 3. Single author at Karlsruhe Institute of Technology,
  workshop (non-archival) and arXiv, no citations yet. In favour: MIT code
  and a Zenodo-archived ledger release that should make every governance
  number recomputable without GPUs, an explicit claim-strength table that
  downgrades its own memory claims, a published correction of the
  workshop RBR counts and replicate status, and reruns after its own hash
  check caught a reset failure.
  - Against: the headline 100% completeness and the 43/50 counterfactual
    hold by construction; task gains are small (case study inside seed
    noise); the memory results are one node, one manager, five seeds; the
    120-trial failure is unexplained; the author used AI assistants to
    write the harness and edit the paper.
  - Not 4: no major-lab or peer-reviewed archival signal and no
    independent reproduction. Not 2: code and ledgers are released and
    the self-correction record is good.

## Follow-up

- **Relevance:** 4. The paper's loop has the shape of this box's
  `/iterate` chain (propose, implement, compare to the prior best, one
  log line per cycle in `_meta/iteration_log.md`), and it supplies two concrete pieces for
  [[concepts/hce-evaluation]]: attempt accounting by a per-slot pending
  guard, and a start-state hash per seed that caught non-independent
  replicates twice. It is also a clean instance of the decision sitting
  outside the proposer, for [[concepts/enforcement-boundary-placement]].
  - Not 5: the headline numbers are design guarantees, and the paper's
    measured effects (memory, task gains) are small-sample and
    single-node. It seeds no concept.
- **Design implications (our inference).**
  - Write a `node_state_hash` (hash of the editable files at trial 1) into
    every seed's metrics row, and refuse to compare arms whose trial-1
    hashes differ.
  - Count attempts from the harness side: open a slot record before the
    run and close it with a typed terminal state, so a crash cannot drop a
    row.
  - Keep "valid but worse" distinct from "invalid", and record the
    failure category, because the infra-vs-agent split may not be
    decidable afterwards.
- **Candidates** (cited, not yet in the graph):
  - Birk et al. 2026, "A Scientific Human-Agent Reproduction Pipeline"
    (SHARP, arXiv 2604.18752). Governs reproduction rather than generation.
  - Zou et al. 2025, "FML-bench: Benchmarking Machine Learning Agents for
    Scientific Research" (arXiv 2510.10472).
  - Karpathy 2026, `autoresearch` (GitHub). The base of the
    `autoresearch_linux` node.
- Already in the graph and cited by the paper:
  [[literature/papers/du2026memory]] (memory survey, informs its memory
  modes); [[literature/papers/chan2024mle]] (MLE-bench, the outcome-only
  evaluation it positions against).
