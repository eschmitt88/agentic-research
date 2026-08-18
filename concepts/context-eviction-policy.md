---
kind: concept
name: "context-eviction-policy"
status: growing
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
  - "[[literature/papers/hao2026selfgc]]"
  - "[[literature/papers/bai2026how]]"
  - "[[literature/papers/li2026acm]]"
  - "[[literature/papers/dang2026addressable]]"
  - "[[literature/papers/xu2026llm]]"
  - "[[literature/papers/mason2026missing]]"
  - "[[literature/papers/cheng2026agenticsts]]"
used_by: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/lossless-context-offload]]"
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

   [[literature/papers/hao2026selfgc]] independently attests
   dependency-aware eviction at production scale and adds the
   measurement that settles the design question: **pruning rate and
   preservation trade against each other, and the heuristics win the
   wrong one.** Across 332 production-derived sessions, position- and
   type-based baselines prune 40–48% of prefix tokens but preserve
   only 78–87% of future dependencies; Self-GC prunes *less* (31–34%)
   and preserves 91–95%. The Hard Set is starker — baselines 62–70%
   prune / 55–70% preservation, Self-GC 44% / 85%. Optimizing for
   tokens removed selects the wrong operating point; the objective is
   removing low-value context *while retaining future anchors*.

   Its failure taxonomy is the practical payoff: what fixed heuristics
   lose is never a generic "bad summary" but a specific class of
   dependency — exact locators and handles, verbatim source text,
   behavioral contracts ("do not replace the table header"), live
   execution state. A chronological or type-based policy has no
   representation for these distinctions, which is why it can remove
   many tokens and still break the run.

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

7. **Split the policy by kind of judgment, not by who owns it.**
   The agent-governed vs harness-governed framing is a false binary.
   [[literature/papers/hao2026selfgc]] runs a side-channel planner that
   proposes `fold`/`mask`/`prune` actions over stable object
   identifiers, and a harness that rehearses the plan locally, drops
   invalid or protected edits, normalizes overlaps, and commits only at
   a safe turn boundary — rejected plans never touch the main agent
   loop. The model supplies *semantic judgment about future value*; the
   harness owns *recoverability, protocol validity, and commit timing*.
   That boundary, not the locus, is the design decision — see
   [[concepts/typed-enforcement]], where this paper supplies the
   measured violation rate.

   Two mechanisms are worth importing directly. **Distinguish the
   lifecycle actions**: `fold` moves an exact payload to a sidecar and
   leaves a recovery pointer, `mask` keeps structural boundaries while
   eliding low-signal middle content, `prune` removes obsolete content
   with no recovery guarantee. Collapsing these into one "compact"
   operation is what makes compaction lossy. **Keep recovery metadata
   on the control plane**: Self-GC attaches fold pointers to the
   relevant *user* message rather than authoring them as assistant
   prose, so later assistant turns don't imitate internal fold tags.

8. **Eviction commits are a cache decision.** Committing an edit to
   the active view invalidates part of the provider prefix cache, so a
   policy that evicts eagerly can cost more than it saves.
   [[literature/papers/hao2026selfgc]] commits incrementally and only
   when `CommitBenefit ≈ N_future(C − C′) − L_cache_break − L_GC` is
   positive, which in their deployment meant holding any plan pruning
   less than ~0.3 of the active view until cache expiry or the next
   task boundary. The 0.3 is fitted to one deployment and shouldn't be
   copied as a constant, but the shape generalizes: eviction policy is
   a spend policy ([[concepts/budget-as-ceiling]]), and the cache term
   is why "evict as soon as you can" is wrong.

## Cache reads are the dominant cost line

Eviction policies are usually argued in tokens; the bill is in dollars,
and [[literature/papers/bai2026how]] shows the two diverge. Decomposing
Claude Sonnet 4.5 trajectories into the four separately-priced categories
(non-cached input, output, cache creation, cache read), **cache reads
dominate both raw token volume and dollar cost in every phase** of the
trajectory — Setup, Explore, Fix, Validate, Closeout — even though output
tokens are priced roughly 80× higher per token. Accumulated reuse of
prior context simply outweighs generation. Per-round cost is
non-monotonic, and the spikes come from what the agent *adds* to context
that round (repository exploration, file creation, test execution, final
summarization), not from the steady accumulated cost of re-reading it.

Two things follow. First, hao2026selfgc's `L_cache_break` term is not a
rounding error — if cache reads are the largest cost category, then
invalidating the provider prefix cache is the most expensive thing an
eviction can do, and a policy tuned on token surface will misprice it.
Evaluate eviction on billed cost. Second, the behavioral signature of
expensive failure is measured here: repeated file *view* and *modify*
actions on the same file rise sharply with cost quartile, and accuracy
peaks at intermediate cost then saturates. Redundant re-reading is both
the thing eviction should target and the reason an expensive run is weak
evidence of a hard problem.

## The cost model is inverted, and it changes the optimal policy

Every classical replacement algorithm — FIFO, LRU, Clock, Working Set,
Belady's MIN — minimizes *faults*, because in virtual memory keeping a page
is free and faulting costs disk I/O.
[[literature/papers/mason2026missing]] points out that LLM context inverts
both terms: **keeping costs tokens on every turn, faulting costs once.** A
5,000-token file resident for 20 turns costs 100,000 input tokens; re-reading
it when needed costs 5,000. So the objective is not fault minimization but

    min Σ_p [ C_keep(p) + C_fault(p) ],  C_keep = |p|·T_resident·c_token

with break-even `|p|·T_until_next_ref > |p|`, which reduces to a rule this
concept can state in one line: **evict whenever the page will not be
referenced for more than one turn.** Two consequences follow that are the
opposite of received wisdom. Aggressive eviction is *correct by default* —
which is why the paper's deliberately minimal FIFO policy (τ = 4 user-turns)
achieved a 0.0254% fault rate over 1,393,000 simulated evictions, despite
FIFO being the worst-performing classical policy. And Belady's MIN is not
optimal here: under inverted costs the optimal policy evicts a page whose
next reference is distant *immediately*, trading one fault for many turns of
keep cost.

Four refinements the paper argues (and does not yet evaluate) are worth
importing as guidance, because each contradicts an intuition:

1. **Get more conservative as pressure rises, not less.** Fault cost is not
   linear in page size: a fault requires an extra full inference pass, and
   attention is O(n²) in sequence length, so the true cost is proportional
   to n² at the *current* fill. At low fill (~40K) faults are cheap and
   aggressive eviction is right; above ~100K the extra pass costs roughly
   (n+|p|)² ≈ n², so the policy should evict only what it is highly
   confident will never be referenced again. The naive instinct — evict
   harder under pressure — is backwards.
2. **Size drives eviction priority.** Unlike classical VM's uniform 4KB
   pages, a recalled object may be a 3-line summary or a 200-line file, and
   a 10,000-token page costs ten times as much per turn as a 1,000-token
   one. Evict large pages eagerly unless access frequency justifies the keep
   cost. The paper calls for a *size-aware, fill-sensitive* policy with no
   analogue in the VM literature.
3. **Batch structural mutations; they invalidate the prefix cache.** A
   collapse of 12 orientation turns dropped the observed cache hit rate from
   100% to 25% for one turn — ~105K tokens of recompute, comparable to
   several page faults — before recovering. "A collapse that saves 10KB of
   context but invalidates a 100K-token cached prefix is a net loss." Prefer
   infrequent large collapses to frequent small ones and pay the
   invalidation once. This is independent confirmation of hao2026selfgc's
   `L_cache_break` term, measured on a different system.
4. **Pins should decay.** One fault currently pins a page for the session,
   but a fault says the content was needed *then*. Halve pin strength every
   K turns since last access and let the page become evictable when
   projected keep cost exceeds fault cost — LRU-like behavior with
   cost weighting, and a guard against the monotonic working-set growth
   permanent pinning causes.

## Count garbage collection separately from paging

The same paper supplies a measurement discipline this concept needs to state
its own fault rates honestly. Ephemeral tool output — Bash results, search
output, directory listings — has no stable identity and cannot be
meaningfully re-requested, so removing it is *garbage collection* and
**cannot** cause a fault. Only *paging* of addressable content (file reads,
plan documents, specifications) can. Conflating them "inflates the eviction
denominator and deflates the apparent fault rate." In the paper's own
steady-state production session, 11 of 15 evictions were garbage collection
and only 4 were pageable — a 3.75× difference in the denominator.

## The failure mode has a name, and a monetary cost

The concept's honest negative evidence is this paper's 681-turn multi-agent
session: 680 evictions, **659 page faults — a 97% fault rate**. Three
diagnosable patterns, all classical: a *thrashing cycle* (three files
faulted repeatedly across turns 163–170+ because the working set exceeded
the resident set), a *sequential scan* (planning across three repositories
re-read the same 7 files, a working set larger than the age threshold
allowed), and **self-inflicted inflation** — the 5,038KB pre-compaction size
included re-reads *caused by previous evictions*, so the proxy was measuring
its own overhead as "bytes saved." The session terminated on the API **rate
limit**, not the context limit: at 659 faults, each an inference-priced
round trip, thrashing consumed the rate budget faster than useful work.

Three things to carry from that. Age is the wrong signal for a hot page —
the single fault in the paper's *healthy* session was a plan file read early
and needed throughout, which FIFO treats as cold, "the classic working set
failure: the eviction policy measures age, not access pattern." Fault cost
is monetary, so an eviction policy is a spend policy
([[concepts/budget-as-ceiling]]). And a compaction mechanism that does not
**count its own faults cannot distinguish savings from churn** — the 97%
rate was visible only because the proxy logged every decision, which argues
a fault-rate counter belongs in the harness alongside the eviction itself.

## Why the waste exists at all

Worth recording because it explains why this is an architectural concern
rather than a bug queue. mason2026missing measures 21.8% of input tokens as
structural waste across 857 production Claude Code sessions (4.45B effective
input tokens) — dead tool output, tool-definition schemas for tools never
invoked (18 definitions sent per call against a median of 3 used, ~52.5KB
wasted), static system-prompt re-send, and skill lists sent three times —
and attributes it to four reinforcing causes: training data consists of
monotonically growing conversations with no examples of intelligent content
removal; the Messages API is a list whose natural operation is append, with
no way to mark content stale; every orchestration framework appends by
default; and **token consumption is billed after the fact, so there is no
backpressure signal**. Quality degradation from bloat is invisible unless
someone measures it. That last point is the reason this concept has to be
designed rather than discovered.

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

## Memory as a contract, and the version with no overflow event

[[literature/papers/cheng2026agenticsts]] reframes what this concept is
about, in one sentence worth adopting: "memory for a long-horizon LLM agent
is not a place to store text; it is **a contract about what each future
decision is allowed to see**."

Under that framing the definition above — a rule for what stays when the
buffer *would otherwise overflow* — describes only one family of contracts,
the ones that let material in by default and then reclaim it under
pressure. The alternative has no overflow event at all. AgenticSTS composes
every decision prompt fresh from five typed slots and **never appends the
raw message turns from earlier decisions**; the governing rule is that "any
information that survives across decisions must first be written into a
bounded store." Nothing crosses a decision boundary implicitly, so growth
is capped by slot budgets rather than by an eviction policy, and a
transcript interface's worst-case `Ω(d · s̄)` growth over `d` decisions
never arises.

This is the far deterministic pole of the policy-locus axis the 08-03/08-10
cycles opened: fixed slot budgets and typed retrieval, with the model given
no say whatsoever, sitting opposite model-initiated compaction. Between
mason2026missing's proxy (deterministic, but reactive — evict, then restore
on a page fault) and this (deterministic and *pre*-emptive — nothing is
admitted unless a store holds it), the deterministic half of the axis now
has two distinct designs, and they differ on whether the default is
inclusion or exclusion.

**The argument for it is about measurement, not cost.** Appending
everything makes context "a jumbled mixture in which the effect of any
single memory component is hard to isolate." A typed contract gives four
handles a raw prompt-history setup hides: growth capped by slot budget,
retrieved evidence labeled by layer, individual layers toggleable without
rewriting the prompt, and runs tagged by condition for reuse. That is an
*evaluation* property of a memory architecture, and it is the more
interesting half of the paper — see [[concepts/hce-evaluation]].

Two honest caveats before importing. The paper never ran the
accumulating-context comparison in its own codebase, so it establishes that
the contract is ablatable, **not that it performs better** — its win-rate
differences are directional at p ≈ 0.37. And its typed layers lean on an
enumerable action space (`L2` ships legal action formats for each state
type), which a closed-rule game has and an open-ended research loop does
not.

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
- **Status moved `seedling` → `growing` on 2026-08-04.** The stated
  precondition was "either CWL's promised follow-up (benchmarks +
  ablations) or an independent attestation of dependency-aware
  eviction." [[literature/papers/hao2026selfgc]] is that independent
  attestation: a different group, a different substrate (indexed
  runtime objects rather than typed episodes), a different motivation
  (production cost rather than hallucination avoidance), converging on
  the same core claim — that *future dependency*, not position or
  type, is what eviction must be keyed on. It also supplies the
  benchmarks CWL lacked (33- and 332-session suites with CIs, plus a
  deployed split), so the pair now covers both the algorithm
  ([[literature/papers/semenov2026beyond]], explicit pseudocode) and
  the evidence (hao2026selfgc, production scale).

  Not `mature`: neither source releases code, hao2026selfgc's headline
  metric is LLM-judged on a 20-case calibration set, and the two agree
  on the principle while differing on the mechanism — CWL evicts by a
  deterministic LLM-free policy over declared dependencies, Self-GC by
  a model-proposed plan under harness validation. Which of those is the
  right default is unresolved, and that is the question a third source
  should settle.

  **The third source arrived (2026-08-10) and voted for a third
  option.** [[literature/papers/li2026acm]] puts the locus in the
  *trained agent*: two tools (`manage_context`, `query_memory`), no
  external trigger, and a teacher–student pipeline that explicitly
  teaches both when to compress and when *not* to (dual-constraint
  annotation). Its evidence cuts two ways. For the agent locus: even
  untrained tool-equipped Qwen3.5-9B beats threshold-triggered ReSum
  and ACON, and post-training adds a further 27% relative on
  BrowseComp-Plus with ~20% lower peak tokens — with released code,
  data, and checkpoints, addressing the no-code caveat above. Against
  free-riding on scale: GPT-5.5 given the same tools makes near-zero
  management calls, so the agent locus only works *because* the policy
  is trained — which is exactly the annotation-burden objection CWL's
  deterministic policy was designed to avoid, relocated from inference
  time to training time. The three-way split (deterministic LLM-free /
  planner-under-harness-validation / trained-agent-initiated) is now
  the concept's central open question, but all three sources agree the
  eviction must be a lossless offload with addressable recall — that
  invariant is factored out to
  [[concepts/lossless-context-offload]].
