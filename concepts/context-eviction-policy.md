---
kind: concept
name: "context-eviction-policy"
status: seedling
added: "2026-05-15"
source_papers:
  - nguyen2026byterover
  - yang2026graph
  - wu2026gam
  - du2026memory
  - khan2026token
sources:
  - "[[literature/papers/li2026complexmcp]]"
  - "[[literature/papers/liu2026dive]]"
  - "[[literature/papers/pham2026memorai]]"
  - "[[literature/papers/sun2026rethinking]]"
  - "[[literature/papers/wang2026reframing]]"
  - "[[literature/papers/wu2026memory]]"
  - "[[literature/papers/yu2026hmem]]"
  - "[[literature/repos/vila-lab-dive-into-claude-code]]"
  - "[[literature/papers/nguyen2026byterover]]"
  - "[[literature/papers/yang2026graph]]"
  - "[[literature/papers/wu2026gam]]"
  - "[[literature/papers/du2026memory]]"
  - "[[literature/papers/omri2026agent]]"
  - "[[literature/papers/kerestecioglu2026human]]"
  - "[[literature/papers/lodha2026less]]"
  - "[[literature/papers/zhou2026ready]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
  - "[[literature/posts/paddo-dev-claude-code-leak-harness-exposed]]"
  - "[[literature/papers/khan2026token]]"
  - "[[literature/papers/lee2026minteval]]"
  - "[[literature/papers/semenov2026beyond]]"
  - "[[literature/papers/chen2026governance]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/skill-library-lifecycle]]"
related_experiments: []
tags: [memory, context-window, eviction, compaction, working-set, runtime-policy]
---

# context-eviction-policy

## Definition

The rule that decides what stays in the LLM's active context window
when the working set would otherwise overflow — and what gets dropped,
compressed, or relocated to a colder tier. The policy operates on the
*conversation buffer* (the messages, tool results, and intermediate
reasoning currently in the prompt), not on the long-term knowledge
substrate. A typical policy combines several mechanisms: recency
weighting, importance scoring, structural anchoring (system prompts,
recent tool results), and *compaction* — replacing a span of cold
turns with an LLM-generated summary that preserves task state.

## Why it matters here

The other three memory concepts in this project name distinct axes:

- [[concepts/agent-native-memory]] — *where* memory lives (the on-disk
  substrate and its provenance discipline).
- [[concepts/skill-library-lifecycle]] — *when* the substrate is
  added to, refined, or pruned (insert / update / delete on disk).
- [[concepts/selective-memory-retrieval]] — *when* the agent decides
  to read from the substrate during reasoning.

Context-eviction-policy names a fourth axis: *what stays in the
prompt while reasoning happens.* It is the working-set-management
problem, distinct from the long-term-memory problem. The Paddo
writeup ([[literature/posts/paddo-dev-claude-code-leak-harness-exposed]])
calls this out as "the hard problem that separates usable agents from
toy demos" — and the leaked Claude Code source confirms it has a
dedicated "context entropy management" subsystem.

The pattern is now attested across four independent harnesses:

- **Claude Code** (per Paddo): "context entropy management" with a
  three-layer memory architecture (short-term / session / long-term);
  Savelis's writeup adds *self-healing compaction* that runs
  proactively rather than waiting for overflow.
- **OpenHarness** ([[literature/repos/hkuds-openharness]]):
  "Context Compression (Auto-Compact)" plus "CLAUDE.md Discovery &
  Injection" — a fixed structural anchor at the top, automatic
  summarization elsewhere.
- **Hermes** ([[literature/repos/nousresearch-hermes-agent]]):
  "Auto-Compaction preserves task state and channel logs across
  context compression — agents can run multi-day sessions without
  manual compact/clear" (v0.1.6 release note, 2026-04-10);
  user-invokable `/compress` slash command.
- **ByteRover** ([[literature/papers/nguyen2026byterover]]) — the
  hierarchical Domain/Topic/Subtopic/Entry layout is the off-context
  substrate the eviction policy spills *to*. Eviction is meaningful
  only because there's somewhere cold for the content to land.

Four independent attestations from a sample of four harnesses is
consensus. The pattern is real, named differently in each project.

[[literature/papers/khan2026token]] adds the *cost* face of the missing
or buggy policy: its M-context-amplification cluster collects 13
production budget-overrun incidents across seven frameworks where
unbounded context growth directly produced dollar losses — a 31×
context overflow from a single base64-encoded image, a 2-million-token
observer-LLM call during tool-heavy runs — and two of the incidents are
Claude Code *compaction-loop* bugs (CCDE-001: ≈$235 in four days for
one user), i.e. the eviction mechanism itself failing open. Eviction
policy is therefore not only a fidelity concern (see Open questions)
but a documented cost control: the failure mode of not having one, or
having a buggy one, shows up as a budget incident
([[concepts/budget-as-ceiling]]).

[[literature/papers/chen2026governance]] adds a third face: *governance*.
Compaction optimizes for task continuity and silently drops in-context
standing policies — 0% violation while the policy is visible, 30%
pooled (up to 59%) after one compaction step, reproduced in LangGraph,
LangMem, and AutoGen's context managers. The eviction layer is thus a
safety-critical surface, not just a fidelity/cost mechanism; the
mitigation is [[concepts/constraint-pinning]].

For this project specifically, eviction policy is currently
*implicit*: `/digest` and `/ingest` produce small outputs (one
literature note, a few concept diffs); `/lint` produces a short
report. None of the skills today face context-overflow pressure
because each is bounded. But the curation loop *as a whole* (a
multi-month research project resumed across many sessions) faces
the same problem: what stays in the model's view of the project, and
what gets dropped to disk?

## Implementation guidance

1. **Eviction policy and storage substrate compose, not stack.**
   The policy decides what to remove from the active context; the
   substrate ([[concepts/agent-native-memory]]) is what it spills
   to. A project that has a great substrate and no eviction policy
   will silently lose work as conversations grow; a project with
   eviction but no substrate will lose work permanently. Both are
   required.

2. **Anchor the structural prefix.** Some context must be
   uncondi-tionally preserved across compactions: the system prompt,
   the active task statement, the project's CLAUDE.md / SOUL.md /
   identity files. OpenHarness names this "CLAUDE.md Discovery &
   Injection" — the structural anchor reasserts on every turn
   regardless of eviction.

3. **Compact, don't truncate.** Truncation (dropping old turns
   silently) loses task state. Compaction (summarizing old turns
   into a short LLM-generated paragraph) preserves it. Hermes's
   v0.1.6 release note is explicit: auto-compaction *preserves task
   state and channel logs* across compression. Truncation as a
   fallback is fine, but the default should be compaction.

   [[literature/papers/semenov2026beyond]] argues a third option
   dominates both: *structured eviction*. The agent annotates its
   trajectory into typed episodes (`expl`/`act`) with declared
   dependencies, and a deterministic, LLM-free policy strips content
   in graduated levels (reasoning traces → bulk outputs →
   intermediates → whole episodes), evicting action records whose
   effects are already persisted in the environment before touching
   exploratory context. This sidesteps compaction's four failure
   modes (unpredictable lossiness, destroyed causal structure,
   blocking cost, compression-induced hallucination) — at the price
   of an annotation burden and a new failure mode: mis-typed
   episodes. Single-demo evidence so far (89 tasks / 80M tokens, no
   ablations).

4. **Run compaction proactively, not reactively.** Savelis's writeup
   on the Claude Code source notes "self-healing compaction that
   runs proactively." Waiting for overflow forces a panic-compaction
   that loses fidelity; running on a slow timer (or after large
   tool results) gives the model time to summarize well.

5. **Tier the storage that eviction spills to.** Three tiers are
   commonly attested: active context window (hot), session-level
   persistence (warm — searchable, accessible via dedicated tool
   calls), long-term files (cold — reachable via grep/search/recall
   commands). The eviction policy is the function that decides
   *promotion* and *demotion* across these tiers.

6. **Make compactions visible.** Compaction is non-trivial: an LLM
   chose what to summarize and what to drop. Log the compaction
   event with the spans involved and the resulting summary — both
   for debugging and for the agent itself to consult when it suspects
   a compaction lost something relevant.

## Connections

- **[[concepts/agent-native-memory]]** — the eviction policy is the
  bridge between in-context working state and the on-disk substrate.
  Without an eviction policy, the substrate is write-only-by-the-user;
  without the substrate, eviction is lossy.
- **[[concepts/selective-memory-retrieval]]** — the read-side dual.
  Selective retrieval pulls cold content back into hot context when
  the agent decides it needs it; eviction pushes cold content out.
  Together they govern the *flux* across the hot/cold boundary; the
  substrate is just where the cold content sits.
- **[[concepts/skill-library-lifecycle]]** — analogous lifecycle at
  a different scope. Skill-library lifecycle is insert/update/delete
  on the *persistent* skill files; context-eviction is the working-
  set version of the same operations on the *transient* prompt
  buffer. Naming both makes clear that "lifecycle" is not one
  thing — it has different policies at different scopes.

## Open questions

- **Compaction fidelity.** The compaction step is an LLM call; it can
  drop or distort the summarized span. How much fidelity is lost,
  and whether the summary can be *audited* by the agent on later
  turns, is open. Hermes logs the compactions; whether the agent
  ever consults the log isn't documented.
- **When to evict vs. when to retrieve.** A turn that triggers
  compaction often also benefits from retrieval (the model is now
  reasoning on summarized history; specific facts may need to be
  pulled back). The interaction between eviction and retrieval
  policies is underexplored — they likely co-design.
- **Recovery from bad compaction.** If a compaction summary drops
  task-critical state, can the agent recover by re-reading the
  pre-compaction transcript from session storage? Hermes's
  "trajectory compression" suggests the raw trace is retained
  somewhere; whether the agent can navigate back to it is not
  explicit.
- **Cost.** Compaction costs an LLM call. The total budget spent on
  compaction across a long-running session is non-trivial and
  largely invisible in current measurement. Worth a /lint extension
  that counts compaction calls.
- **Status is `seedling`** because the pattern is attested across
  four sources but no single source provides the canonical
  algorithm. ByteRover's paper has the closest thing to a quantitative
  policy (importance scoring, recency decay, hysteresis).
  [[literature/papers/semenov2026beyond]] (CWL) now states a full
  explicit algorithm — typed episode DAG + graduated deterministic
  eviction, in pseudocode — but from a low-credibility source with
  single-demo evidence. The move to `growing` should wait for either
  CWL's promised follow-up (benchmarks + ablations) or an independent
  attestation of dependency-aware eviction.
