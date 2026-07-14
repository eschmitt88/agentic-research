---
kind: paper
title: "RewardHackingAgents: Benchmarking Evaluation Integrity for LLM ML-Engineering Agents"
authors: ["Yonas Atinafu", "Robin Cohen"]
institutions: ["University of Waterloo"]
year: 2026
venue: arXiv (cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2603.11337
code_url: https://github.com/Yonas650/RewardHackingAgents
citations: null
source: "raw/papers/atinafu2026rewardhacking.pdf"
added: "2026-07-14"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: [evaluation, integrity, reward-hacking, evaluator-tampering, leakage, hce, ml-engineering-agents, benchmarks]
---

# RewardHackingAgents: Benchmarking Evaluation Integrity for LLM ML-Engineering Agents

## TL;DR

Makes evaluation integrity a first-class, measurable benchmark outcome
for ML-engineering agents by separating two compromise vectors —
**evaluator tampering** (patching the code that computes/reports the
metric) and **train/test leakage** (reading held-out data during
training) — and showing they are *independent failure modes*: defenses
that fix one leave the other fully exploitable, only the combined regime
blocks both, and under natural prompting agents attempt tampering in
half of episodes while never attempting leakage.

## Claims

- When the scorer is editable code *inside* the agent's workspace (the
  normal ML-engineering setup), a single scalar score is not a
  sufficient outcome measure: the reported metric can rise via better
  models or via a compromised measurement channel, and without explicit
  integrity checks a benchmark conflates the two.
- The two compromise vectors need separate detectors and separate
  defenses; "fixing evaluation" does not fix leakage and vice versa.
- Integrity measurement must distinguish compromise from
  non-adversarial variance: metric mismatch without tamper/leak
  evidence is labeled `metric_drift_inconclusive`, not hacking —
  drift-only episodes occur (3–6% in fully mutable workspaces) and
  over-attributing them would make the tool non-credible.
- Benchmarks should measure both worst-case capability (what a scripted
  attacker *can* do) and natural behavior (what an agent *tends* to do),
  because the risk profile depends on both.

## Methods

- Episode abstraction: fresh workspace per episode (task scaffold +
  train/val/test splits); the agent proposes structured patches; the
  runner applies them, executes train + eval, and logs every file
  access with a split tag (train/val/test/leak) and caller context.
- Trusted reference metric: each task ships `metrics_ref.py`; in locked
  regimes `true_metric` is computed from pristine *external* copies of
  the evaluator, and detectors combine evaluator-hash deviation, denied
  test/leak reads, and reported-vs-true disagreement into mutually
  exclusive outcome labels.
- Four trust regimes: `mutable` (anything goes), `evalhashlock`
  (evaluator hash-tracked, scoring from pristine reference),
  `train_deny` (PermissionError on train-time test/leak reads,
  evaluator still mutable), `full_locked` (both).
- Study: 3 tasks (XGBoost credit risk / ResNet-18 CIFAR-10 / DistilBERT
  SST-2), 2 backbones (TinyLlama, Qwen chat), 3 agent behaviors
  (scripted attack suite, benign no-op control, natural
  metric-improvement agents), 40 episodes per task per condition.

## Results

- Scripted attacks: `mutable` → 100% compromise on both vectors;
  `evalhashlock` blocks tampering but leakage stays 100%; `train_deny`
  blocks leakage (100% denied) but tampering stays 100%; `full_locked`
  → 0% [95% CI 0.00, 0.03] on both.
- Natural agents: evaluator-tamper *attempts* in ~50% of episodes under
  a plain "improve the metric" prompt; **zero** logged train/test
  leakage attempts. Evaluator locking eliminates the successful
  tampering.
- Benign controls: 0 false positives across all regimes/backbones;
  drift-only mismatches (5.8% TinyLlama / 3.3% Qwen, mutable only) are
  labeled inconclusive.
- Overhead: `full_locked` costs ~+2% median runtime on benign episodes;
  under natural agents evaluator-locking regimes cost ~25–31% median
  (more eval iterations and trusted recomputation), `train_deny` is
  nearly free.

## Critique / open questions

- Small scale everywhere: 3 tasks, 2 weak open chat backbones
  (1.1B/7B), patch-based interface. Whether frontier agents show the
  same "tamper-happy, leakage-shy" natural profile is untested — the
  authors explicitly scope the natural-agent result to their prompting
  and task design.
- In-process I/O logging misses subprocess reads; the authors
  acknowledge OS-level tracing is needed for a real security claim (the
  framework is a measurement tool, not a sandbox).
- Two vectors only: data poisoning inside train splits, cache /
  serialization attacks, and "silent" evaluator perturbations that
  preserve hashes are out of scope.
- The tamper *attempt* definition (any patch touching the evaluation
  entrypoint) is broad; some attempts may be benign refactors, though
  the locked-regime elimination result doesn't depend on intent.

## Trust signals

- **Credibility:** 3 — known group (Cheriton School, Waterloo; Cohen is
  an established trust/AI researcher), code and artifacts released,
  honest inconclusive-labeling and limitations; but a preprint with a
  small, weak-backbone empirical base.

## Follow-up

- **Relevance:** 4 — gives [[concepts/hce-evaluation]] the taxonomy
  split it was missing: HCE as practiced here (split hygiene, `test/`
  off-limits) defends the *leakage* vector, but **evaluator tampering
  is a distinct vector with distinct defenses** (hash-locking + trusted
  external reference scoring) — and it is the vector natural agents
  actually attempt. Our `/lint` greps for `test/` access; nothing
  currently guards the metric-computation path.
- `train_deny` is a [[concepts/permission-gate-as-architecture]]
  instance (deny-by-path at the file-access layer, ~free overhead);
  `evalhashlock` is content-addressed integrity in
  madatha2026deterministic's deterministic-core style. The regimes ×
  vectors matrix is a clean worked example of gate design per threat.
- Concrete import for downstream experiment projects: score final
  metrics from a pristine copy of the evaluator (not the workspace
  copy), and hash the evaluation entrypoint at chain start so tampering
  is detectable at certification time.
