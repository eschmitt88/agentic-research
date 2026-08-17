---
kind: concept
name: "context-proprioception"
status: growing
added: "2026-08-11"
sources:
  - "[[literature/papers/xu2026llm]]"
  - "[[literature/papers/bai2026how]]"
  - "[[literature/papers/mason2026missing]]"
used_by: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/spend-forecast-calibration]]"
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

## Perception is not prediction

[[literature/papers/bai2026how]] marks this concept's boundary from the
outside. Its agents were given what proprioception provides and more —
full tool access, permission to inspect the repository and run
preliminary commands, and an explicit instruction to produce a
stage-decomposed cost estimate before acting — and still reached only
Pearson r ≤ 0.39 against their own actual token usage, with **every model
biased low**, worst on the dominant input-token term. Access to state did
not yield a usable forecast.

So the claim here should stay narrow: surfacing present resource state
(per-block cost, recency, archive status, remaining headroom) improves
meta-decisions about what to keep, archive, or recover — that is an
interface problem and xu2026llm shows the interface fixes it. Predicting
*future* spend is a different problem that the same interface does not
solve, because the trajectory's own branching sets the number. Expose
state; do not trust prediction. See
[[concepts/spend-forecast-calibration]].

## Second attestation: pressure zones as a deployed proprioceptive interface

[[literature/papers/mason2026missing]] is this concept's second source, and
unlike xu2026llm it is a production deployment rather than a benchmark
harness — on 857 Claude Code sessions. Its **graduated pressure zones** are
this concept's interface, built as a policy rather than a dashboard:

- **Normal** (<60K tokens): observe only; the agent is told nothing.
- **Advisory** (60K–100K): inject memory-pressure information into the
  model's context — current fill percentage, the **five largest resident
  blocks**, and the available cleanup operations. Explicitly "the graduated
  equivalent of the 'low memory' notification in desktop operating
  systems."
- **Involuntary** (100K–120K): automatic eviction proceeds; the model is
  informed but not consulted.
- **Aggressive** (≥120K): emergency eviction, context survival over
  working-set preservation.

Two design points are worth more than the thresholds. First, the 60K
advisory trigger was chosen to leave **~40K tokens of runway** — enough for
the model to finish a coherent thought and emit cleanup directives *before*
losing agency over the process. Proprioception with no time to act on it is
just a notification; the interface has to fire while the agent can still
choose. Second, the zones make the perception→action loop explicit and
bidirectional: the proxy informs the model of memory state (phantom tools
`memory_release` / `memory_fault`), and the model directs the proxy through
cleanup tags parsed from its output (`drop`, `summarize`, `anchor`,
`collapse: turns N-M`). The paper argues this two-way protocol is a genuinely
new design point, because hardware replacement algorithms assume a
**non-cooperative** application, whereas an LLM has an incentive to
cooperate — cleaner context means better output and a longer session. "The
processor wants to help manage its own cache."

The concept's core claim gets independent behavioral support too: without
any instruction, a model resuming a session recognized retrieval handles it
had never been told about and chose to fault content in before acting. Give
the agent legible state and it uses it.

## Open questions

- How noisy can the state estimates be before the benefit collapses?
  (Thm. 1 bounds the *rate* of the interface; real token accounting
  drifts.)
- Does proprioception help at scopes beyond the context window —
  disk, wall-clock, API spend — where the ledger is cheap but the
  action space (pause, delegate, downscale) is coarser?
- Adversarial surface: a dashboard the agent trusts is a prompt-
  injection target ([[concepts/verified-memory-writes]] adjacent).
