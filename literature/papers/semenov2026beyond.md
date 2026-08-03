---
kind: paper
title: "Beyond Compaction: Structured Context Eviction for Long-Horizon Agents"
authors: ["Andrew Semenov", "Svyatoslav Dorofeev"]
institutions: ["Kiz8"]
year: 2026
venue: arXiv (2606.11213)
peer_reviewed: false
url: https://arxiv.org/abs/2606.11213
code_url: null
citations: null
source: "raw/papers/semenov2026beyond.pdf"
added: "2026-08-03"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/agent-native-memory]]"
tags: [memory, context-window, eviction, compaction, episode-graph, long-horizon, deterministic-policy]
---

# Beyond Compaction: Structured Context Eviction for Long-Horizon Agents

## TL;DR

Context Window Lifecycle (CWL): the agent annotates its own trajectory
into typed, dependency-linked episodes as it works, and a deterministic,
LLM-free eviction policy strips content in graduated levels when a token
budget is exceeded — an explicit algorithmic alternative to
summarization-based compaction. Demo: one session, 89 sequential tasks,
80M tokens, no measurable accuracy degradation vs. per-task isolated
sessions.

## Claims

- Summarization-based compaction has four structural limitations:
  unpredictable lossiness, destruction of causal structure, blocking
  cost (a full LLM call mid-task), and compression-induced
  hallucination at the moment the agent is least able to catch it.
- The agent is the best-placed party to annotate its own trajectory
  (Principle 2: "the agent is the authority on structure") — it knows
  at time-of-work which content is live reasoning context vs. action
  records already persisted in the environment.
- Causal dependencies dominate recency (Principle 4): the right
  eviction order follows the dependency graph, not the timeline.
- Eviction must be deterministic and model-free (Principle 5) — this
  rules out compression-induced hallucination by construction and makes
  a compression pass effectively free.
- User turns are inviolable (Principle 3); budget misses surface as
  explicit conditions rather than silent degradation.

## Methods

- **Annotation protocol**: a single `delimiter` tool with
  `start`/`end`, episode `type` ∈ {`expl`, `act`}, `dependencies`
  (act episodes declare which closed expl episodes they consume), and
  a required `description` when closing an expl episode — the only
  content retained after full eviction.
- **Episode graph**: typed DAG, append-only; edges only expl→act;
  pre-first-annotation content is a protected prologue; active
  episodes never evicted.
- **Eviction policy**: loop while over budget — pick oldest eligible
  act episode (else oldest expl whose dependents are fully evicted),
  strip in graduated levels: reasoning traces → bulk tool outputs →
  intermediate artifacts → whole episode. Smallest increment that
  meets budget; exits when satisfied or nothing safe remains.
- Evaluation: single-session 89-task / 80M-token run compared against
  isolated per-task sessions; authors state larger evals and ablations
  are deferred to a follow-up version.

## Results

- 89 sequential tasks across 80M tokens in one session with no
  measurable task-accuracy degradation relative to per-task isolated
  sessions; active context held near a stable ceiling below the
  degraded-attention regime.
- No benchmark suite, run counts, or variance reported in v1 — the
  empirical section is a single demonstration, explicitly framed as a
  first release.

## Critique / open questions

- The empirical claim is one run on an unnamed task suite: no
  variance, no ablation of the dependency constraint vs. plain
  recency, no comparison against a strong compaction baseline. By
  this project's own [[concepts/pass-at-k]] standard this is a
  design paper, not an evidence paper.
- Annotation burden is acknowledged but unmeasured: mis-typed
  episodes (act vs. expl) shift eviction order, and there is no
  audit mechanism for mis-declared dependencies — the same
  fidelity-audit gap the compaction critique aims at, moved from the
  summarizer to the annotator.
- Two-type edge restriction (expl→act only) keeps the policy
  tractable but cannot express act→act pipelines (e.g. a build step a
  later deploy depends on); the paper argues effects persist in the
  environment, which assumes the environment is re-inspectable.
- Contrast with Context-Folding (RL-trained branch/return) is the
  most useful part of related work: who decides (learned model vs.
  deterministic algorithm), depth (2-level vs. arbitrary DAG),
  trigger (implicit vs. token accounting).

## Trust signals

- **Credibility:** 2 — two-author preprint from Kiz8 (small
  independent team, unknown track record), no peer review, no
  released code or artifacts, no citations yet; the design is
  clearly articulated but the empirical validation is a single
  unreplicated demo.

## Follow-up

- **Relevance:** 4 — [[concepts/context-eviction-policy]] noted that
  no source provides "the canonical algorithm"; CWL is exactly that:
  typed episodes + dependency DAG + graduated deterministic eviction,
  stated in pseudocode. It also challenges the concept's
  "compact, don't truncate" guidance with a third option — structured
  eviction that neither summarizes nor truncates blindly. Held to 4
  (not 5) on credibility: single-demo evidence from an unknown group.
- The deterministic, model-free policy over an agent-declared typed
  artifact is a clean attestation for [[concepts/typed-enforcement]]
  — enforcement (here: what survives in context) runs as code over a
  typed structure, not as model judgment.
- Watch for the promised follow-up version with benchmark suites and
  ablations; that release would settle whether the dependency
  constraint carries its weight.
