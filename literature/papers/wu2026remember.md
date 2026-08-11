---
kind: paper
title: "Remember When It Matters: Proactive Memory Agent for Long-Horizon Agents"
authors: ["Yifan Wu", "Lizhu Zhang", "Yuhang Zhou", "Mingyi Wang", "Bo Peng", "Serena Li", "Xiangjun Fan", "Zhuokai Zhao"]
institutions: ["Meta AI"]
year: 2026
venue: "arXiv preprint (2607.08716, cs.AI)"
peer_reviewed: false
url: https://arxiv.org/abs/2607.08716
code_url: https://github.com/yifannnwu/proactive-memory-agent
citations: null
source: "raw/papers/wu2026remember.pdf"
added: "2026-08-11"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/context-eviction-policy]]"
tags: [memory, intervention, long-horizon, behavioral-state-decay, selective-injection, ablations, memory-agent, rl]
---

# Remember When It Matters: Proactive Memory Agent

## TL;DR

Names the failure mode **behavioral state decay** — decision-relevant
state (requirements, environment facts, failed attempts, diagnoses,
open subgoals) may still sit in the transcript or even in the window,
yet stops influencing the next action — and treats memory as an
*intervention policy*, not storage: a separate memory agent observes
the trajectory, maintains a structured bank, and at each step decides
to inject one targeted reminder or **stay silent**; ablations show
this selective intervention beats passive bank exposure, always-on
injection, advisor-only guidance, and Mem0-style general retrieval.

## Claims

- **Behavioral state decay ≠ context loss.** Information can remain
  physically present in the window and still lose behavioral control
  — so longer contexts and better retrieval don't solve it. The
  question shifts from "can it be stored/retrieved" to "when should
  remembered state re-enter the control loop."
- **Intervention is a stronger control question than
  summarization**: a summarizer asks what to retain; this asks
  whether any retained state should *interrupt* the next decision.
  Failure modes differ per task, so no fixed summarization or
  injection schedule can be right.
- **Silence is an explicit action.** Unnecessary interventions are
  harmful, not merely redundant — they add tokens, latency, and
  distraction. Intervention timing is part of the memory policy, not
  a consequence of every memory update.
- The architecture is plug-and-play: the action agent, its harness,
  tools, and decoding are unchanged; the only difference is an
  optional transient memory context at call time.

## Methods

- Memory bank B = (status, knowledge, procedural): a *private*
  status field (memory agent's working model of progress — never
  shown to the action agent), stable task/environment facts, and
  attempt/outcome records. Entries have identifiers, tagged content,
  creation/access metadata; explicit delete.
- **Two-phase step** at fixed intervals: Phase 1 edits the bank via
  constrained tool calls (update_status / save_knowledge /
  save_procedural / delete) — a sequence of bank edits, not a
  free-form summary. Phase 2, conditioned on the updated bank +
  recent window, emits either a `<context_for_action>` reminder or
  `<no_intervention/>`.
- Evaluated on Terminal-Bench 2.0 (85 paired tasks, official
  harness) and τ²-Bench (airline/retail/telecom, 278 tasks), action
  agents Claude Sonnet 4.5 and Opus 4.6, memory agent Opus 4.6.
- Preliminary open-weight branch: Qwen3.5-27B memory agent trained
  with SFT distillation + GRPO ("SETA"), action agent fixed; partial
  transfer to held-out Terminal-Bench.

## Results

- Terminal-Bench 2.0: Sonnet 4.5 37.6% → 45.9% (+8.3pp); Opus 4.6
  43.5% → 45.9% (+2.4pp) — gains shrink but persist for the stronger
  agent, so the benefit is not just compensating limited capacity.
  τ²-Bench task-weighted: Sonnet +6.8pp (55.0 → 61.8), Opus +2.5pp.
- **Ablations (τ²-Bench, Sonnet action agent)** — full agent 64.3
  macro vs: full-bank passive exposure 61.5 (maintaining a bank is
  useful but exposing everything is not enough), always-inject 63.5
  (competitive on micro, loses on domain-balanced macro — selective
  silence wins where injections can mislead, e.g. airline),
  injection-only advisor without a bank 61.0 (ungrounded guidance is
  unstable, hurts airline below baseline), Mem0 vector+BM25 top-10
  62.1 (retrieval returns relevant records but doesn't model *when*
  memory should enter the loop).
- Qualitative mechanism taxonomy: requirement/policy reactivation,
  environment grounding, failure-loop avoidance, diagnostic
  carryover, progress/entity tracking — memory helps when it
  reactivates state the agent saw but is about to ignore.

## Critique / open questions

- The memory agent is Opus 4.6 invoked every step-interval — the
  paper doesn't price the overhead against the +pp gains; the
  open-weight branch exists precisely because prompted frontier
  memory agents are expensive, but its transfer is only partial.
- Fixed-interval triggering was chosen to isolate the intervention
  policy; the obvious next step (trigger on tool errors, repeated
  commands, context shifts) is untested here — and would blur the
  line between this and gated retrieval.
- Ablation deltas (0.8–3.3 macro points) are modest relative to run
  variance the paper itself acknowledges for always-inject.
- Both benchmarks are agentic-execution, not research tasks; the
  behavioral-state-decay framing plausibly transfers, the numbers
  may not.

## Trust signals

- **Credibility:** 4 — Meta AI, code released, official benchmark
  harnesses, careful one-capability-at-a-time ablations including a
  production-system (Mem0) comparison; arXiv-only, no peer review or
  citations yet.

## Follow-up

- **Relevance:** 4 — the strongest ablated attestation yet for
  [[concepts/selective-memory-retrieval]]'s core claim (timing
  selectivity is load-bearing; both passive exposure and always-on
  lose), and it adds a genuinely new pole: **push-side** gating by a
  parallel observer with an explicit null action, versus the
  concept's existing pull-side triggers.
- "Behavioral state decay" is a better name than "forgetting" for
  the failure the runtime-memory cluster keeps circling — state
  present but behaviorally inert; worth adopting in concept prose.
