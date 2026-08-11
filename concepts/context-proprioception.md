---
kind: concept
name: "context-proprioception"
status: seedling
added: "2026-08-11"
sources:
  - "[[literature/papers/xu2026llm]]"
used_by: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/selective-memory-retrieval]]"
related_experiments: []
tags: [memory, context-window, self-management, dashboard, runtime-state, budget-awareness, meta-decision]
---

# context-proprioception

## Definition

An agent's runtime resource state — per-item token cost, recency,
archive status, remaining budget — is surfaced to the agent as an
explicit, machine-maintained ledger, so that meta-decisions about its
own working memory (keep / archive / recover / stop) are made *with
perception* rather than inferred unreliably from prompt text or
delegated to a rule the agent cannot inspect.

## Why it matters here

The single source ([[literature/papers/xu2026llm]], VISTA) makes an
unusually crisp claim: state visibility is a **distinct resource from
any policy that acts on it**. Its Theorem 1 (Fano-based) shows that
even with a perfectly lossless archive, a size-blind manager
over-archives and under-recovers in expectation; the ablation confirms
it empirically — removing the dashboard costs more than removing the
recovery tools (37.3 vs 45.3, full 50.7 on LOCA-Bench). And the RL
result inverts the usual framing: GRPO training *implicitly* teaches
the model to estimate its own context state, while simply exposing the
state gets most of the benefit zero-shot. Elicitation over training:
the competence is latent; the interface is what's missing.

Why seed at one source: the pattern is already load-bearing in
claude-system — `/headroom` (token windows, hardware, agency verdict
with a suggested session budget) is precisely a proprioceptive
dashboard for the *session* scale, and `budget.yaml` ceilings plus the
coordinator's GO/SLOW/HOLD verdict are the enforcement half. VISTA
supplies the missing argument for why the dashboard is not just
convenience: without it, the policy — trained, prompted, or scripted —
is acting under partial observability of the one state it most needs.

## Distinctions

- Not [[concepts/context-eviction-policy]]: that axis is *who decides
  and by what rule*; proprioception is *what the decider can see*.
  VISTA's evidence is that the seeing matters independently of the
  deciding (zero-shot gains before any policy change).
- Not [[concepts/budget-as-ceiling]]: a ceiling halts at a boundary
  the agent may never perceive; a proprioceptive budget line lets the
  agent adapt *before* the boundary. The two compose — VISTA keeps
  the hard harness-enforced budget and adds the perception.
- The dashboard is "a ledger over blocks, not a memory oracle": it
  exposes runtime state created by the harness and the agent's own
  earlier actions, never hidden task evidence. That boundary is what
  keeps proprioception from becoming a side-channel.

## Connections

- [[concepts/lossless-context-offload]] — VISTA's Prop. 1 / Thm. 1
  pair proves the two concepts jointly necessary: recovery preserves
  evidence after eviction; proprioception selects what to evict.
  Import both or expect the failure mode of the missing half.
- [[concepts/selective-memory-retrieval]] — the read-side gate is
  also a meta-decision under partial observability; a recency/usage
  ledger is the natural input it currently lacks.
- claude-system instances: `/headroom`, the hardware poller, and the
  agency verdict — session-scale proprioception feeding an explicit
  spend policy.

## Open questions

- How noisy can the state estimates be before the benefit collapses?
  (Thm. 1 bounds the *rate* of the interface; real token accounting
  drifts.)
- Does proprioception help at scopes beyond the context window —
  disk, wall-clock, API spend — where the ledger is cheap but the
  action space (pause, delegate, downscale) is coarser?
- Adversarial surface: a dashboard the agent trusts is a prompt-
  injection target ([[concepts/verified-memory-writes]] adjacent).
