---
kind: paper
title: "Self-GC: Self-Governing Context for Long-Horizon LLM Agents"
authors: ["Xubin Hao", "Hongjin Meng", "Xin Yin", "Jiawei Zhu", "Chenpeng Cao"]
institutions: ["Xiaohongshu"]
year: 2026
venue: arXiv (cs.AI)
peer_reviewed: false
url: https://arxiv.org/abs/2607.00692
code_url: null
citations: null
source: "raw/papers/hao2026selfgc.pdf"
added: "2026-08-04"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/agent-native-memory]]"
tags: [memory, context-window, eviction, compaction, lifecycle, runtime-policy, prefix-cache, production-deployment, harness]
---

# Self-GC: Self-Governing Context for Long-Horizon LLM Agents

## TL;DR

Treats long-horizon agent context as a collection of **indexed runtime
objects with lifecycles** rather than a linear token buffer: a
side-channel planner proposes fold/mask/prune actions over stable object
identifiers, and the harness rehearses, validates, and commits them only
at safe turn boundaries with recoverable sidecars. On production-derived
traces it prunes *less* than heuristic baselines (31–44% vs 40–70%) while
preserving far more future dependencies (85–95% no-impact vs 55–87%), and
a deployed A/B split cuts daytime average input tokens 10–15%.

## Claims

- The failure mode of current context management is **not excessive
  length** but a *mismatch between token-buffer operations and
  object-level future dependencies*. Position-based heuristics cannot
  tell whether an old tool output holds the only URL, table value, file
  path, or editable body a later step still needs.
- Aggressive pruning is not sufficient for agent context management; the
  right objective is removing low-value context *while retaining the
  future anchors*. Baselines that prune harder score worse on
  continuation success.
- Context objects have **distinct lifecycle requirements**, so one action
  does not fit all: `fold` (move exact payload to a sidecar, leave a
  compact recovery pointer), `mask` (keep structural boundaries, elide
  low-signal middle), `prune` (remove obsolete content, no recovery
  guarantee).
- The safety mechanism is a **division of labor**: "the model supplies
  semantic judgment about future value, while the harness enforces
  runtime invariants such as recoverability and protocol validity."
- Committing a context edit is a **cost decision, not only a fidelity
  decision** — it invalidates part of the provider prefix cache, so it
  should be committed incrementally and only when expected future
  savings justify the disruption.
- Recovery metadata must be attached to *user* messages as control-plane
  reminders, not authored as assistant prose, so later assistant turns
  don't imitate internal fold tags.

## Methods

- **Indexed context objects.** The transcript maps to addressable objects
  with session-local monotonic ids: `conversation:user:k` for a user
  request and its following span, `function:<tool>:n` for independently
  editable tool results (carried as a lightweight XML boundary tag).
  Assistant turns are deliberately *not* first-class targets — they hold
  connective text and tool-call envelopes, and are normalized by the
  harness when adjacent objects change.
- **Reflective plan → rehearse → commit.** On token pressure, a turn
  boundary, or a policy trigger, the system forks the context prefix and
  appends planner-only instructions. The planner emits a structured plan
  of object actions (an "object-action contract", not a summary prompt).
  The harness then rehearses locally: resolves targets, drops invalid or
  cut-turn edits, normalizes overlapping actions, materializes the
  projected view, estimates savings. Accepted plans stay pending until a
  safe turn boundary; **rejected plans never affect the main agent loop**.
- **Cache-aware commit.** `CommitBenefit ≈ N_future(C − C′) −
  L_cache_break − L_GC`. A deployment regression over observed triggers
  found immediate commit is positive-value only once expected active-view
  pruning exceeds 0.3; below that the plan is held pending until cache
  expiry or the next task boundary. Stated as an operating policy, not a
  universal constant.
- **Harness interface (portability).** Requires only that the harness
  expose turn/tool-span boundaries, maintain object indices, let a
  side-channel planner read a forked prefix, replay candidate edits into
  a local projection, persist folded payloads to sidecar storage, and
  commit at a safe boundary. Explicitly a context-engine hook rather than
  a provider-specific agent rewrite.
- **Evaluation.** 15,141 raw trace rows → 9,075 compaction-triggered
  traces → a **332-session Production Suite** and a 33-session **Hard
  Set** (distilled from highest sustained tool-pressure sessions, skewed
  to browser/shell/web-fetch). Offline: counterfactual replay at matched
  cut points with mandatory last-turn retention; a GPT-5.5 judge checks
  whether exact URLs, row values, task identifiers, editable bodies, and
  source-backed evidence remain available in the retained prefix
  (Wilson 95% CIs). Online: account-level split by email first character,
  covering `context-gc` (interactive chat) and `skill-gc` (long-lived
  skill state).
- **Baselines**: oldest-turn fold, tool-prune greedy, tool-mask+prune,
  and a hybrid — replayed at the same cut points with the same token
  accounting.
- **Planner backbones**: Qwen3.6-Plus (default), Qwen3.7-Max, GLM-5.1.
  GPT-5.5 is used only as offline judge, never as the planner under test.

## Results

- **Hard Set (n=33).** Self-GC: 43.95% prune, **84.85% no-impact**
  [69.08, 93.35]. Baselines prune harder (61.90–69.87%) but land at
  54.55–69.70% no-impact. Gain of ≥15.15 points over the strongest
  completed heuristic.
- **Production Suite (n=332).** Self-GC across three planners: 31.04–
  33.98% prune, **91.27–94.58% no-impact**. Baselines: 40.19–47.76%
  prune, 77.71–87.46% no-impact — all below 90%.
- **A/B calibration** on `oldest-turn-fold` vs Self-GC Qwen3.6-Plus: raw
  single-prompt judging gives 89.76% vs 92.47%; on 20 disagreement cases
  the A/B judge prefers Self-GC in 11, ties in 7, prefers the baseline in
  2 → calibrated 92.77% vs 87.46%.
- **Online.** Daytime average main-agent input tokens down 10–15% for the
  Self-GC group, peaks near 20%, over May 25–30 covered traffic.
- **Planner robustness.** All three backbones exceed 90% no-impact on the
  Production Suite — a mid-tier planner suffices *provided the harness
  enforces deterministic safety checks*. Directly measured: planners
  proposed touching the protected cut turn in **25/330 (Qwen3.6-Plus),
  15/330 (Qwen3.7-Max), 12/328 (GLM-5.1)** parsed plans — "the prompt
  usually works, but the residual risk justifies mandatory last-turn
  protection."
- **Workload asymmetry.** Tool pruning is competitive on BI-heavy Hard
  Set sessions (many dumps can be re-queried), while chronological turn
  folding is stronger on DOC/CODE-heavy Production sessions (short tool
  outputs often carry unique artifact identifiers). Coding tasks have Git
  history, build logs, and rerunnable commands as external memory;
  office-style workflows often lack an agent-accessible version
  substrate, so a short tool output may be the only durable business
  evidence.
- **Failure taxonomy of fixed heuristics**: losses are not generic "bad
  summaries" but specific missing dependencies — evidence details,
  locators and handles, behavioral contracts ("do not replace the table
  header"), verbatim source text, live execution state.

## Critique / open questions

- **No code or artifacts released.** The limitations section is candid:
  raw traces contain private user data, and reproduction would need
  sanitized fixtures, prompt templates, per-sample judge outputs, and
  recomputation scripts — all deferred to "future releases". Nothing is
  independently checkable today.
- **The primary offline metric is LLM-judged.** "No-impact" is a GPT-5.5
  judgment about whether required anchors survive, not measured task
  completion. The A/B calibration set is 20 disagreement cases — small.
  Judge-sensitivity analysis is listed as future work, i.e. the headline
  15-point gain rests on an uncalibrated judge.
- **The online result is not a clean experiment.** An account-level split
  by email first character is quasi-random at best, it isn't a matched
  billed-cost audit, and it excludes side-channel planner overhead. The
  paper says this plainly — input-token reduction is confirmed, *net*
  cost reduction and user-quality preservation are not.
- **The planner is an extra LLM call on a hot path.** Cost is
  acknowledged in the `L_GC` term but never reported as a standalone
  number, so the true overhead of governance is unquantified.
- **The 0.3 commit threshold is fitted to one deployment** and offered as
  operating policy; whether it transfers is untested.
- **Name overreach.** "Self-governing" oversells it — the model proposes,
  the harness disposes. That is the paper's actual strength (see
  Follow-up), but the framing invites the wrong reading.
- **Assistant turns are excluded from object targeting.** Reasonable for
  their traces, but in agents whose assistant turns carry long plans or
  derived analysis, that is where the bulk sits.

## Trust signals

- **Credibility:** 3 — real production deployment at a large consumer
  platform (Xiaohongshu) with 332 production-derived sessions plus a live
  account-level split is a strong evidence signal that most context
  papers lack; offset by no released code or artifacts, no peer review,
  an LLM-judge primary metric with a 20-case calibration set, and a
  non-randomized online split the authors themselves decline to call a
  quality experiment. Deployment evidence at scale, weak reproducibility.

## Follow-up

- **Relevance:** 4 — supplies the *independent attestation of
  dependency-aware eviction* that [[concepts/context-eviction-policy]]
  named as its precondition for leaving `seedling`, and does so with
  production-scale evidence where [[literature/papers/semenov2026beyond]]
  had a single demo. Also the sharpest quantitative statement yet of
  [[concepts/typed-enforcement]]'s division of labor.
- The real contribution is **not** agent-governed-vs-harness-governed
  context (the framing this candidate was surfaced under). Self-GC is a
  *split by kind of judgment*: the model decides semantic future value,
  the harness owns recoverability, protocol validity, and commit timing.
  That reframes the policy-locus axis from a binary into a boundary
  question — which is the version worth importing.
- The 25/330 · 15/330 · 12/328 cut-turn violation counts are a rare
  direct measurement of **how often a well-prompted model would breach an
  invariant the harness enforces deterministically** (~4–8% of plans).
  That is the missing empirical answer to typed-enforcement's open
  question "no head-to-head between a formal policy engine and a
  well-prompted frontier model" — partial, single-invariant, but real.
- Cache-aware commit closes the cost angle without needing a separate
  cache-economics source: eviction policy *is* a spend policy
  ([[concepts/budget-as-ceiling]]), and the prefix-cache term is why
  naive eviction can cost more than it saves.
- Worth reading against [[literature/papers/chen2026governance]]: Self-GC
  attaches recovery metadata to user messages as control-plane reminders
  and mandates last-turn retention — both are
  [[concepts/constraint-pinning]] mechanisms arrived at independently, for
  fidelity rather than safety reasons.
- Open thread for this project: the `skill-gc` variant (lifecycle pruning
  of long-lived *skill state*, as opposed to chat context) is mentioned
  in the online split but never evaluated separately. That is the variant
  closest to [[concepts/skill-library-lifecycle]] and it is unmeasured.
