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
  - "[[literature/papers/fan2026empirical]]"
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
  - "[[literature/papers/zhang2026correct]]"
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
  - "[[literature/papers/kim2026why]]"
  - "[[literature/papers/badhe2026skill]]"
  - "[[literature/papers/kassis2026scientific]]"
  - "[[literature/papers/lavrenko2026instruction]]"
  - "[[literature/papers/li2026autorecsys]]"
  - "[[literature/papers/piriyakulkij2026subagents]]"
  - "[[literature/papers/shen2026what]]"
  - "[[literature/papers/nguyen2026cliffcompaction]]"
  - "[[literature/papers/hu2026analyzing]]"
  - "[[literature/papers/srikanth2026recursive]]"
  - "[[literature/papers/samanta2026learning]]"
  - "[[literature/papers/cai2026forkandflush]]"
  - "[[literature/papers/li2026humanize]]"
  - "[[literature/papers/farahbakhsh2026agentdiscover]]"
  - "[[literature/papers/liu2026what]]"
  - "[[literature/papers/xu2026replaylens]]"
  - "[[literature/papers/sun2026scipivot]]"
  - "[[literature/papers/kumar2026turncost]]"
  - "[[literature/papers/xu2026what]]"
  - "[[literature/papers/lencevicius2026when]]"
  - "[[literature/papers/salle2026benchmarks]]"
  - "[[literature/papers/busch2026ctxagent]]"
  - "[[literature/papers/jain2026monitor]]"
  - "[[literature/papers/zha2026do]]"
  - "[[literature/papers/chu2026when]]"
  - "[[literature/papers/liu2026agents]]"
  - "[[literature/papers/passos2026one]]"
  - "[[literature/papers/chen2026compactbench]]"
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

   [[literature/papers/nguyen2026cliffcompaction]] is the first
   head-to-head that tests "compact, don't truncate" directly, and it
   **does not support the rule as a default.** Its truncate-only policy
   keeps verbatim fragments (tool results ≤500 chars, ~150-char call
   signatures, 300-char thoughts) and drops the rest. It is compared with
   LLM summarization (Claude Code's prompt), a reimplementation of Claude
   Code's microcompaction + summarization, and a sliding window. On
   SWE-bench Verified all four land within 2.6 points, and summarization is
   the weakest. On Terminal-Bench at 16K, swapping Terminus-2's native
   summarizer for truncation moves Kimi K2.6 from 55.45 to 61.42. On 1M-token
   KernelBench runs, truncation scores 3.58× and summarization 3.47×, with
   summarization slightly cheaper ($8.10 vs $8.32). So summarization buys no
   measurable quality over faithful truncation in coding workspaces, and the
   paper's re-read counts suggest one reason. A dropped span leaves a visible
   gap the agent refills from the workspace (+5.04 re-reads per instance),
   while a plausible paraphrase discourages the re-read (+2.35). Revised
   guidance: **where evicted content is re-derivable from the environment,
   truncate to verbatim fragments and let the agent re-fetch. Keep
   summarization for content that cannot be re-derived.** Its "never compact a
   compaction" rule (discard the previous block rather than nest it) is the
   design answer to summary-of-summary drift. That drift is argued, not
   measured. The evidence covers coding and terminal tasks on open-weight
   models only.

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

   [[literature/papers/nguyen2026cliffcompaction]] measures the cadence
   directly. Three policies ran on the same KernelBench runs: append-only
   growth with rare large resets ("cliffs"), frequent small prefix edits
   (microcompaction), and a per-step shift (sliding window). Cost per
   problem was $8.32, $12.84 and $21.52 respectively, and the per-step
   cache decomposition shows why: each microcompaction tooth is a fresh
   partial re-prefill. On Terminal-Bench the uncompacted bill is 78% cache
   reads, and the savings come entirely from cutting them (−80%). Two cautions
   carry over. Tightening the threshold lowers the provider cache-hit rate
   (79% → 47% for GLM 5.1), so savings are **non-monotonic in budget**: 53%
   at 16K and 37% at 8K for Kimi K2.6. And on a cheap, already well-cached
   model the saving can vanish: +1% at 8K on mini-swe-agent. Price a
   threshold on metered cost, not tokens.

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

A third category belongs next to those two: **re-reads that no eviction
caused.** [[literature/papers/hu2026analyzing]] finds that 11.05–18.10% of
all retrieved code lines on SWE-bench are re-reads of regions an earlier
read fully covered. It traces their causes to things other than paging:

- a subagent returned a summary instead of the code (50.15% of Claude
  Code's re-reads);
- an edit tool gave no feedback, so the agent re-read to check the result;
- a `cat` without line numbers prompted a zoom-in read;
- the content "becomes less salient" deep in a long debugging loop.

The trajectories are short (tens of calls), so most of this content was
plausibly still resident. That is our inference; the paper does not report
compaction events. A harness fault counter that logs every re-read as a
fault would therefore charge the eviction policy for salience and
interface failures it did not cause. The fix for those lies elsewhere: a
prompt rule ("reuse context; say why before re-reading"), line-numbered
reads, and edit tools that report what they changed.

The paper also reports the size of the saving. Task cost moved 2.83× as
much as the behavior-attributed cost on Verified, because a prevented
action also shortens every later cache read. This is bai2026how's
cache-read dominance, seen from the savings side.

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

## Truncation that ties summaries on task score loses a once-stated rule in most runs, and the summary budget has a cliff (2026-11-16)

This page's selective-vs-summary evidence so far is about task score or
cost. Rule-based dropping roughly ties summarization
([[literature/papers/nguyen2026cliffcompaction]],
[[literature/papers/kumar2026turncost]], and
[[literature/papers/salle2026benchmarks]]'s tool-output pruning).
[[literature/papers/zha2026do]] compares ten policies on a different
target: whether a rule stated once, early, still governs an action after
one compaction. The harness is an unmodified pi, with 17 cases, four
flash-tier models, and the same model compacting and acting. Grading is
deterministic from the tool call, against a matched full-history arm.

- **Verbatim selection drops the rule.** Share of runs whose kept turns no
  longer contain the rule:
  - FIFO: 100%.
  - LRU over turns later referenced by a repeated call or quoted literal:
    88.2%.
  - An LLM ranking turns by "likely to be needed again": 63.7%.

  violation@3 is 44.8–54.4% for these, against 9.0–42.6% for the seven
  generative strategies. The agent's-own share is flat across all ten
  (6.0–12.1%), so the spread is entirely in what the policy kept. A
  constraint is referenced by nothing until the moment it is needed, so
  recency and reference counts both evict it. This is
  [[literature/papers/lencevicius2026when]]'s observed-fact case, measured
  on behaviour.
- **Among summaries, a constraints slot does the work.** Pi and Structured
  (the same prompt, with a "Constraints & Preferences" field) retain the
  rule in 96.0% and 94.5% of runs. One added preserve-rules paragraph
  (Pi + RP) gives 99.0% and the only below-baseline violation rate (9.0%
  vs 11.9%). The ACON prompts, tuned for task utility or compression,
  retain 90.7–96.0% but are violated more often (20.6%, 31.8%).
- **The budget is a cliff, not a slope.** Under Hermes with volume filler
  (DeepSeek, GLM), retention is ≥97.1% from 2,500 down to 750 tokens and
  ≤4.9% at 300 and 200. violation@3 jumps from 24.2–28.1% to
  47.1–50.0%. Hermes's fixed 177-token preamble is clipped first, ahead of
  rule-bearing text, so at tight budgets boilerplate displaces the rule.
- **The wrapper around the summary is part of the policy.** Hermes's
  reference-only prefix ("NOT as active instructions … the latest message
  WINS") coincides with the largest violation excess on *retained* rules
  (26.8% vs 9.3% full). The paper never isolates it; see
  [[concepts/constraint-pinning]].
- **The evicted span is an injection surface for the summarizer.** One
  retrieved note telling the compactor to omit rules raises the
  generative family's rule-absent share from 0.9% to 16.3%. Selective
  compactors do not read content, so they are unaffected, but they
  already drop the rule. In this design volume filler is a null (37.8% →
  34.3%) because compaction is forced anyway. It does not test
  [[literature/papers/chen2026governance]]'s volume-to-trigger-compaction
  attack.

Rules (ours, extending the 2026-10-27 "carry stated-once observations in
explicit fields"):
- Do not switch a long-horizon agent to plain truncation on the strength of
  task-score parity if it carries standing rules. The parity does not
  extend to them.
- Give the summary a constraints field, and fill it before fixed preambles
  when clipping.
- Validate any summary budget below about 750 tokens on retention of the
  rules specifically.

Limits:
- One forced compaction over 12–16 benign messages. The
  re-compaction regime where salle2026benchmarks saw rules erode is
  untested.
- Flash-tier models only. No Claude Code compactor.
- Selective-policy retention is labelled mechanically (was the
  rule-bearing turn kept), generative retention by an LLM panel.
- The budget sweep is Hermes-only with filler in every run, and TRACE is
  an AppWorld prompt run out of domain.
- No interval is shown for any difference.

## Compaction fidelity has an additive failure, and deleting the bad sentence is not the repair (2026-11-16)

The "Compaction fidelity" open question, and the 2026-10-20 and
2026-10-27 sections, measure what a summary *loses*.
[[literature/papers/chu2026when]] measures what it *adds*: permissions or
requirements the record does not support. It is the first test in this
graph that holds the environment fixed at the compaction point and varies
only the summary (same-snapshot replay, 4 successor runs per arm, matched
seeds). The agents are open-weight (Qwen3-8B/14B, gpt-oss-20b) on
synthetic tasks with a ground-truth permission ledger.

- **Prevalence.** On fresh held-out instances, 1.2–2.4% of summaries carry
  an unsupported restriction and 0.1–1.3% an unsupported authorization
  (lower bounds; AI-confirmed).
- **The repairs differ by kind of error.** The arms are neutral filler,
  deletion, appending the true state (`restore`), rewriting the sentence
  with the truth (`claim_correct`, an oracle), and the full history.
  - *False approval:* neutralizing it cut unauthorized actions from 24 to
    7 of 120 (pooled, post hoc). The full history gave 0 of 120.
    Appending the true state only partly overrode a planted false approval
    (42 of 142 runs, against 53).
  - *Overgeneralized restriction* (a rejected request turned into a rule;
    59 of 76 held-out cases): neutralizing barely moved confirmation
    requests (197 → 190 of 236). Writing in the true permission with the
    allowed target brought them to 48. The successor was missing a fact,
    not just carrying a wrong one: in 192 of 197 such runs it asked about
    a target the ledger did not allow.
  - *Truth costs something too:* `claim_correct` raised rule violations
    in two sets (8 → 18, 10 → 26 of 148), and `restore` doubled them on
    Test (8 → 16) while lifting task success from 32 to 102 of 148.
- **The compaction prompt is not the lever.** Structured (with Rules and
  Approvals headings, "only include facts you saw") and brief-note prompts
  both still produced invented restrictions, at rates no lower than the
  default (exploratory). The intervention that worked was upstream, in the
  system prompt the writer ran under (5.0% → 1.8% development,
  1.8% → 0.3% held-out, pre-specified; see
  [[concepts/intent-level-briefing]]). In 46 of 64 development cases the
  invented rule was already in the agent's earlier reasoning before the
  summary was written.

Our reading: this extends the 2026-10-20 rule (fixed fields for
failures and rejected options). A rejected request should be carried as
"X on target A was rejected; B is allowed", with the allowed alternative.
Carried as prose, it becomes "X needs approval". The full-history arm's
0 of 120 is another vote for keeping the raw transcript addressable
([[concepts/lossless-context-offload]]). It is not a vote for every
policy: on the development split the full history still carried the
original guess (179 of 256 runs asked).

Rule: when evaluating a compaction change, count sentences the summary
*adds* that no source turn supports, as well as facts it drops. A
self-summary that restates permissions should carry the allowed target
and any withdrawal, not just the requirement.

Limits:
- Small open-weight agents and a harness where the successor never sees
  the original request. The summary's influence is near its maximum.
- Synthetic ledger tasks. The "wrong target is rejected" behaviour is
  one specific way a refusal arises.
- The restriction replay effect was null on the pre-specified held-out
  test (p = 0.58). Most repair comparisons are exploratory, four runs per
  arm.

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

A field instance of the failure the no-overflow contract avoids comes from
an industrial research harness. In [[literature/papers/li2026autorecsys]],
background agents monitoring multi-day training jobs "were dying silently
after roughly three to five hours because each polling cycle appended tool
results until the LLM context window overflowed." The replacement, designed
by the agent itself, was **cron-based ticks, each a fresh prompt with no
context accumulation**, reading job and idea state from files. Two points
transfer. For a polling loop, append-by-default fails by *silent death*,
not degraded output, so it goes unnoticed until someone checks. And the fix
is AgenticSTS's contract in its simplest form: nothing crosses a tick
boundary unless a store holds it. The evidence is a single anecdote, but it
bears directly on any practice of monitoring long jobs from one persistent
session.

## At a generous window the policy is a liveness device, not a selector

[[literature/papers/fan2026empirical]] adds a blunt fourth reading, with a
176-setting sweep behind it: **most of what a context-management policy buys
is not dying.** The managed-minus-unmanaged success gap collapses "from 35.7
to 15.9, 5.5, and 2.7 percentage points on SWE-Bench" as the window grows,
tracking overflow one-for-one — "the model-averaged T0 overflow rate falls
from 78.7% to 8.7% on SWE-Bench and from 61.0% to 12.1% on Terminal-Bench,
while all managed tiers have zero overflow failures throughout." The
trajectory analysis finds no behavioural change to explain the rest:
management "primarily extends execution trajectories without substantially
altering agent behavior."

Be precise about what this licenses. All four managed tiers share the same
selection *target* — preamble and a recent window pinned verbatim, only the
middle compacted — and there is no random-eviction arm, so the common
anchor-plus-recency rule is never itself measured. The supported statement is
narrower than "selection doesn't matter": **given a sane anchor-plus-recency
rule, the marginal return on a cleverer middle-region policy is near zero,
and the return on not overflowing is large.**

**One concrete recommendation: order the mechanisms cheap before expensive.**
The tier that elides at a soft threshold and invokes the LLM summarizer only
above a hard one matches every other tier on accuracy "with the lowest cost
in seven of eight model–benchmark panels", because early rule-based elision
keeps the metered summarizer from firing.

A caution on that cost table: the unmanaged arm's low cost at tight budgets
is the cost of dying early. No cost-per-task figure is interpretable without
its success rate beside it.

**A blind search lands on the same rule, and under a dollar cap the second
payoff is more steps (2026-09-29).** [[literature/papers/srikanth2026recursive]]
evolved a research agent's harness under fixed per-run dollar budgets. The
context policy it kept is anchor-plus-recency. Draft and improve prompts read
"a compact summary of the root and recent candidates rather than the full
history". A state-gated injection adds up to three deduplicated error
signatures only when the run's bug rate is ≥ 15%. The unmanaged starting
agent reproduces the liveness finding: it overflowed on five FML-Bench runs
and 48 ALE-Bench runs, and the managed agents never did. The new element is
the cost-capped regime. The median per-call prompt shrinks by 7× on
MLE-Bench and about 50× on ALE-Bench and FML-Bench, and the authors'
argument is that "shorter prompts … buy more search steps in cost-bound
runs." That is a benefit on top of not dying. Caveat: the policy shipped
bundled with a new search policy, and no rewrite was ablated. The
steps-per-dollar gain is argued, not isolated.

**A per-decision rebuild from a repo, with a fixed curator (2026-10-06).**
[[literature/papers/samanta2026learning]] (MIRA) takes the opposite
extreme to compaction. Nothing carries over between research decisions.
The outer context is rebuilt from a fixed portion (task, workspace summary,
latest proxy eval, a structured summary of the previous episode with a
pointer to its handover, remaining budget), plus evidence a fixed curator
selects from the Git record. This is still anchor-plus-recency: Fig. 7 shows
a stable task scaffold plus recent artifacts and the latest handover, about
10 files out of a repo of about 5k. The selection is not ablated against a
simpler rule, and the curator's outputs were "not fully logged". The
cost comparison against full-history replay (10.6M input tokens per episode
measured, vs higher *projected* replay costs) is a projection, not a run. It
adds a fourth locus to the open three-way split, *fixed LLM curator over a
persistent repo*, but no evidence that it beats the other three.

## Sufficiency is a property of the future, not of the current query

Every policy above is audited against the query in front of it.
[[literature/papers/zhang2026correct]] holds that answer fixed and varies
what comes *next*, and the two come apart inside the same system. Its
paired-history audit builds pairs that share a current answer and a future
update but require different subsequent answers, so a retained state can be
**correct now and insufficient later**.

The ablation is the contribution. On DeepSeek, frontier and latest-only both
score 96/96 on the current branch while reveal accuracy falls 96/96 → 32/96;
on GLM, latest-only is *better* on the current branch (84/96 vs 78/96) and
far worse after the update (23/96 vs 82/96). So **current-answer accuracy can
actively select the less update-capable policy** — it is not merely a weak
proxy for sufficiency, it is sometimes an inverted one. That complements
[[literature/papers/hao2026selfgc]]'s finding that optimizing the prune rate
picks the wrong operating point: here the *scoring target*, not the tuning
knob, is what misleads.

Scope this tightly. The study is a pilot — 24 synthetic pairs, four per
mechanism family, with mechanisms the author generated. Quote the ablation,
never the ranking: the 96/96 reveal contrast bundles delivery and schema
effects, GLM's deficit is a pure wrapper artifact (82/96 → 96/96 under a bare
-map rule), and frontier does **not** beat raw history (DeepSeek joint 45/48
against 46/48 for both archives).

Two further cautions the paper supplies against itself. Its reported
**identification intervals are not confidence intervals** — they are
partial-identification bounds over unresolved outcomes inside a fixed sample,
carrying no sampling-error information and licensing no inference to new
histories. One of them, the GLM reveal interval, has width exactly zero; the
widest is one ambiguous block (1/48). And its winning selector broke priority
ties lexicographically by identifier: under consistent renaming,
late-reference adequacy collapses 8/8 → 94/320. The label-equivariant repair
retains only 2/8 at identity and needs 4× the pilot's byte cap to recover.
The author's own conclusion is the one to carry — "eliminating a naming
shortcut does not solve unknown future relevance."

**What to do with it here:** treat it as an evaluation design, not a policy
recommendation. Before crediting any eviction rule, run one paired history
where the future update differs and the current answer does not.

## Compaction preserves the strategy; a flush is an exploration move (2026-10-12)

The sections above treat eviction as a way to keep the agent alive and
correct within a fixed task line. [[literature/papers/cai2026forkandflush]]
measures a side effect of what survives. Multi-day autoresearch runs that
compact their history periodically still stay in one region of approach
space. "One might expect this compaction to weaken dependence on the
earlier reasoning path and reopen exploration. We find that it is
insufficient." A summary keeps the run's strategic commitment, which is
what anchor-plus-recency is designed to do.

The paper's alternative is a full flush, (W, C) → (W, ∅): keep every file,
drop all conversation. Flushed branches must re-derive intent from the
workspace. At equal token spend, flush-only (K = 1, two flushes) lifts the
normalized mean from 0.45 to 0.55 over 11 tasks. It is erratic per task
(better on 6 of 11) and more reliable when combined with best-of-K
selection (0.71). For this concept the point is that eviction policy has
a third role beyond liveness and selection: **an aggressive eviction is a
search operator**, and a cleverer summary works against it. Any context
policy for a long `/iterate` chain should be judged on whether it lets the
chain change direction, not only on whether it preserves task state.

## A full flush on a schedule, used against deceleration rather than overflow (2026-10-12)

[[literature/papers/li2026humanize]] (HMA) uses the most extreme eviction
policy, a total flush that keeps nothing in the prompt, and adopts it for a
reason this page has not yet recorded. The context is not overflowing. The
policy is meant to restart a session that is slowing down. Each agent session
on MLE-bench is cut after k = 5 accepted experiments. Context resets to
empty, and only the shared workspace and the retained best result carry
over. The motivating measurement is "long-horizon deceleration": in
uninterrupted 6 h single-agent runs, output tokens per response fall by
29.8–45.0% from hours 0–2 to hours 3–5, across six models. Within a
session, the first five experiments hold 97.8% of the observed gains. With
the flush, HMA keeps 86.4% of its early response length, against 60.7% for
the single agents.

This adds a third role to the liveness reading
([[literature/papers/fan2026empirical]]) and the selection reading. The
policy acts as a **restart device**. The evidence is weak in exactly the
place this page cares about:
- The flush is always paired with a switch to a *different* model and
  harness, and no same-model reset arm exists.
- Response length is a confounded proxy. Early turns write pipelines and
  late turns poll jobs, and every reset restarts the code-writing phase.
- In the one cap ablation, flushing every 1 or 3 experiments scored below
  never flushing on a schedule.

Treat "flush to restart" as a hypothesis. The single-backend test of it is
fork-and-flush (arXiv 2610.07447), which keeps the workspace, flushes the
chat and holds the model fixed.

## A flush offered on a cost trigger, with the history behind a query (2026-10-13)

[[literature/papers/farahbakhsh2026agentdiscover]] (AgentDiscover) is a
third total-flush design. This one is triggered by price, not by schedule
or overflow. From the fifth candidate of a session, the server's reply
includes a cost note: the context has grown to L tokens, while "a fresh
session would start from ⟨c + i⟩". The agent decides whether to stop. A
fresh agent inherits the worktree, the `tools/` and `skills/` folders and a
graph database of every attempt, and it re-reads the history by query
rather than by replaying it. The flush is cheap because nothing has to be
re-read.

The evidence is observational. After a stall of more than five candidates,
the chance of a new best within three candidates is 21.2% when continuing
(n = 650), 25.0% after a reset (n = 32) and 26.2% after a reset plus a
meta-agent pass (n = 42). The reset arm's binomial SE is about 8 points,
and the agent chose when to reset. The direction agrees with
cai2026forkandflush (same model, flushed chat, kept workspace). It is a
third instance of "flush as exploration move", not an independent test of
it.

## Whole-harness check: an overflow mostly marks a task already lost (2026-10-13)

[[literature/papers/liu2026what]] tests the liveness reading
(fan2026empirical) on production harnesses rather than tiers in one loop.
mini-SWE-agent has no compaction and overflows a 131k window on 11 of 45
hard tasks and 18 of 447 pool tasks with Qwen3.8-27B. But the harnesses
that compact mostly failed those tasks too. Recovering every overflowed
task that another harness passed would move mini-SWE-agent by 6.7 points
on hard45 and 1.3 on the pool, and 7 of the 11 overflow again on rerun.
At a generous window, then, not dying is worth little when the overflow
is a symptom of a failing trajectory. Compaction did not make Claude Code
or OpenCode beat the uncompacted harness: CC vs mini is equivalent within
±5 points on both Qwen models. The cost side matters more. Each step
re-sends the transcript, so a harness's bill is its preamble and growth
line, and compaction's cliffs show up in the cost curve, not the pass rate.

## Eviction decides whether a store survives a restore (2026-10-13)

[[literature/papers/xu2026replaylens]] adds a property that this concept's
policy comparisons have not scored: whether the retained state depends on
*what* arrived or on *the order it arrived in*. Its pair-transport zero law
says a store that sums a position-independent function of each record
is invariant to reordering. FIFO retention is not, because arrival order
decides which entries survive the cap. With a 16-slot store and 8-slot
retrieval, replaying the same events in a different order changed FIFO's
stored summaries and retrieved text in 192/192 test blocks. It changed 0
for logical-time retention, an identifier-priority reservoir and per-action
binned means. So a FIFO-evicted memory cannot be safely rebuilt from an
unordered backup or a merged log. That is a migration hazard and needs no
accuracy loss to matter. Two caveats keep this small. The utility cost was
3 × 10⁻³, and the binned store also kept all 32 observations to FIFO's 16,
which confounds coverage with order. An endpoint-only selector picked the
same safe store anyway, and the paper's criterion for added audit value
was not met.

## The agent's own project notes lower its choice at a pivot (2026-10-19)

[[literature/papers/sun2026scipivot]] gives a paired measurement of what a
retained record does to a decision rather than to recall. Five frontier
models choose between an agent's original plan and a human correction. They
do it once with the project record (idea, proposal, the agent-written
STATE.md, code, logs) and once with only the two plans. At medium effort
every model scores lower with the record. The five-model mean is 39.2%
against 55.8%. The paired drop is −11.7 to −26.6 points for four models and
−4.2 [−10.9, +2.4] for Astra. In the worked cases the record carried the
agent's own interpretations, and the models adopted them. One was "earlier
checks ruled out an evaluation fault". Another was a parameter restriction
the agent had imposed on itself, which Opus 5 later called
"human-authorized".

This is the decision-level counterpart of cai2026forkandflush's finding that
compaction preserves the run's strategy. What anchors is the narrative
layer, which a summary keeps by design. It is in tension with
[[literature/papers/wang2026language]], where in-environment experiment
records *raised* Opus 5's ranking of unseen interventions (Spearman 0.506 →
0.774). One reconciliation is that records of measured outcomes help and
records of the agent's conclusions hurt. That split is our hypothesis.
Neither paper runs a notes-stripped arm. Here the record also adds length,
file retrieval and a provider-specific tool interface. Because cases were
selected on model disagreement, the size of the gap may be inflated, though
not its sign.

For a long `/iterate` chain: when a cycle has to choose a new direction,
give it measurements and code rather than a replay of the chain's own log,
and mark inherited claims as unverified.

## Coding-agent observations are small, so a compression call rarely beats truncation (2026-10-20)

[[literature/papers/kumar2026turncost]] measures the thing an eviction
policy acts on, in 600 published coding-agent trajectories (OpenHands and
SWE-agent × Qwen3.5-122B and Minimax-M2.5). The median tool observation is
118–286 tokens per cell, and the p99 is 4,274–6,208. The large build log
that motivates selective compression is about the top 1%. The largest
observations come from editor echo, not logs: edit turns are 29.9% of turns
but 51.8% of observation tokens (mean 787). That is the content the agent
is working on and the least safe to drop.

Two consequences for this page. First, mason2026missing's "size drives
eviction priority" is right in principle but has little to act on in coding
traces, because the large-page tail is thin. Second, at the paper's prices
a summary written by a cheaper model saves no more than deterministic
truncation. Across 20 cell × threshold combinations the gap is at most
0.17 pp, and both save 11–19% at aggressive thresholds. Savings go negative
at conservative thresholds in two of four cells: a late observation has too
few remaining turns to repay its own model call. This matches
nguyen2026cliffcompaction's rule-based drop tying summarization on quality.

The scaffold's truncation message also matters. After 468 truncations with
a recovery hint (e.g. narrow with `grep -n`), 76.9% of next actions
narrowed the query. After 50 bare truncations, 44.0% did. That is a
behavioural point for a visible gap with a pointer, as in
nguyen2026cliffcompaction and the [[concepts/lossless-context-offload]]
scope condition.

Caveats:
- Cost only. Trajectories are fixed and no quality arm is run. Truncation
  ≈ summary is a cost statement.
- Simulated ideal prefix cache, one tokenizer, and a corpus that excludes
  runs which hit the iteration limit. The excluded runs are where a huge
  re-read log would sit, so the tail is understated.
- The truncation-hint numbers are observational (n = 50 bare) and do not
  show whether the content was recovered.

Rule: before adding a model call to compress observations, compare against
plain truncation that tells the agent how to get the content back.

## A prose summary keeps about half the task state; fixed fields keep more of what cannot be re-derived (2026-10-20)

[[literature/papers/xu2026what]] gives the closest measurement in the
graph to the open "compaction fidelity" question, using an agent's
user-facing reports as the summary. On the same 4,799 probes (37 SWE-bench
tasks), recovery rises with what the reader keeps: latest report only
.239 (2.6k chars), all reports .520 (6.6k), terminal scrollback .736
(25k), full context without reasoning .890 (107k), full context .927
(167k). Reasoning accounts for only 3.7 points. Most of what the summary
loses is tool output and narration, which the workspace can often
regenerate ([[concepts/lossless-context-offload]]'s re-derivability
condition) but a report reader cannot.

Two results bear on how to write the carried summary. First, at matched
total length, a rewrite from full context recovers about 8 points over the
agent's own reports (natural +8.3, targeted +8.5), with no targeted over
natural advantage (+0.2 [−1.5, +1.8]). Second, on a held-out split, a
summary organized by fixed categories picked from dev data (failures,
facts, progress, environment) recovers more *non-inferable* state than a
natural summary at the same budget (.110 vs .031; +.077 [.030, .133]).
Unrecovered atoms also accumulate: 9.4 at the first checkpoint, 22.3 at
the fourth, 18.9 of them from earlier turns. That is the same direction as
[[literature/papers/zhang2026correct]]'s "correct now, insufficient
later", seen across turns of a real task. With
[[literature/papers/sun2026scipivot]] (above), the pattern is that a
carried summary should hold structured state and measurements. Free prose
both drops facts and carries the agent's interpretations forward.

Caveats:
- These are reports to a user, not compaction summaries the agent
  re-reads. The analogy is ours.
- The probe population is extracted from the full context, so the scale
  is relative to that view, not to what a next step needs.
- One executor model, one trajectory per task, a non-random cohort that
  skews easy.

Rule: when a long chain carries state forward as a summary, give it fixed
fields for failures and rejected options and keep the raw logs
addressable; do not rely on prose to carry them.

## Dropping turns saves tokens and costs 3-4x in dollars, and a summary carries derived facts far better than observed ones (2026-10-27)

[[literature/papers/lencevicius2026when]] gives two measurements this page
has so far argued without matched arms.

**The cache break, priced.** In a synthetic probe where every episode is
played under every harness, a 24k-token sliding window used 0.84–0.90× the
tokens of full history and cost 2.9–4.0× as much in dollars in every task
cell (gpt-6-luna). 94% of full history's input was served from cache,
against 48% for the window, and cached input is a tenth of the price. On
claude-haiku-5-5 the ratio was 3.3–3.7×, because that provider bills the
rewritten prompt as cache writes at 1.25× input. This is the matched-arm
version of bai2026how's "cache reads dominate" and mason2026missing's
batch-your-mutations rule. A drop-oldest window rewrites the prefix on
every turn, so it pays the invalidation every turn.

**What a lossy harness carries.** In the cells where the fact had left the
window and could not be looked up again:

| Harness | Observed fact | Derived fact |
|---|---|---|
| Full history | 100% | 99.5% |
| Window, drop oldest | 7.5% | 8.0% |
| Window + one-line notes instruction | 28.5% | 26.5% |
| Summary + notes (exploratory) | 43.0% | 92.0% |

When the notes held the fact (21.5% of observed episodes), the model used it
every time, so the loss comes from what gets carried, not from using what was
carried. On ARC-AGI-3 (Gemini 3.8 Flash, four games picked for a large
published gap, one run per cell), replacing drop-oldest with summary
compaction while keeping the notes line took the mean score from 9.2 to
63.1, against 71.8 for the provider's native-replay harness.

Our inference: a fact the agent derived sits in its own reasoning and
replies, so a summarizer treats it as task state. A fact shown once in an
observation does not look like task state until it is needed. That is the
same direction as [[literature/papers/xu2026what]] (fixed fields keep more
non-inferable state) and the 2026-10-26 digest's item 18 (compaction drops
rules first). A constraint stated once at session start is the observed-fact
case.

Caveats:
- The summary arm is exploratory. The ARC shares come from single runs, and
  a same-configuration rerun moved one game by 40 points (see
  [[concepts/hce-evaluation]]).
- One more failure mode for the implementation list: at high thinking an
  8,192-token output budget for the summary call was used up by reasoning.
  The summary came back empty four times in a row and the run died. The
  upstream harness fixed it by letting compaction inherit the model's
  output limit. A compaction call to a reasoning model needs a budget that
  covers reasoning plus summary.

Rule: evaluate a context policy on billed cost with caching on, not on
tokens. When compacting, carry stated-once observations (constraints,
one-off measurements) in explicit fields, because a summary keeps derived
conclusions by default and drops them.

## Default self-summarization was not beaten, and discarding more costs some models 16–20 points (2026-10-27)

[[literature/papers/salle2026benchmarks]] fixes one scaffold (OpenCode).
It compares five compaction strategies on PaperBench, ProgramBench and
PostTrainBench with Opus 4.6, GPT-5.4 and GPT-5.3-Codex. There are three
results for this page.

1. **For 1M-window models the threshold does not matter in the range
   tested.** Opus and GPT-5.4 are flat from 160K to 380K. That range is
   only 16–38% of their window, so the near-full regime where Claude Code
   compacts is untested. GPT-5.3-Codex (400K) drops from about 0.60 to
   0.38 on PaperBench at 160K, where it averages 27.7 compactions per run.
   Its accuracy declines from about 8 compactions onward. Opus and GPT-5.4
   show no relationship with compaction count.
2. **No strategy beats self-summarization beyond the CIs.** Tool-output
   pruning ties it. This is consistent with
   [[literature/papers/nguyen2026cliffcompaction]] and
   [[literature/papers/kumar2026turncost]] (cheap rule-based dropping ≈
   summary).
3. **Discarding more does not transfer across models.** On PaperBench:
   - Clear thinking takes GPT-5.4 from 0.59 to 0.43 and leaves Opus and
     Codex unchanged.
   - A Haiku 4.5 summarizer leaves Opus within CI, but takes GPT-5.4 from
     0.59 to 0.43 and Codex from 0.38 to 0.18.

   The authors suspect a same-provider effect. That means a cheap
   summarizer validated on one backbone is not validated for another,
   which is the same point as chen2026governance's summarizer-dominates
   crossing.

Caveats: 2 runs per task × configuration, 3 PaperBench papers judged by
gpt-4o-mini on the Code-Dev variant, and accuracy conditional on a valid
submission with failed runs relaunched. The nulls are low-powered, and
only the 16–20 point drops are clearly outside the noise.

Rule: validate a compaction change on the backbone that will run it. If
the summarizer is a different model, test it against the task model on
the same tasks first. The paper only shows a same-provider cheap
summarizer to be safe, and only for Opus.

## A prompt alone moves the compaction decision to the model, for some models only, and the saving is in the cache bill (2026-11-02)

The 2026-08-10 note below records li2026acm's finding that untrained
GPT-5.5 barely used its context tools, so the agent locus seemed to need
training. [[literature/papers/busch2026ctxagent]] tests the untrained,
prompt-only version on seven current models. Each gets two tools
(`compact_context` writes a hand-off summary, `curate_context` swaps
acted-on tool results for one-line notes) and one paragraph of guidance.
Sessions run 101–115 turns, 3 seeds, at a 1M window. Arm B removes the
tools.

- Opus 5.5, Sol 6.1 and Astra 6 compact every ~10 turns at task
  boundaries and keep context near 50–80k. Opus cost ratio A/B 0.57, Sol
  0.69, with SciCode steps 77 vs 80 and 71 vs 72 of 80.
- Haiku 5.5 and Muse Wonder never called `compact_context`. Sonnet 5.5
  compacted in one session of three and lost 8 of 80 steps (65 vs 73).
- Sol under no context pressure still cut 7 times in clean chains and
  passed 62/86 against Codex's 68/86. Opus made 0 cuts there.

Against Claude Code on the same Opus 5.5 sessions, the self-managed arm
cost $4.80 per session against $7.31 at an explicit 200k window (1 cut at
~168k) and $9.35 at the default window, where Claude Code never compacted
and the context reached 277k. Success was 0.969 vs 0.979 (McNemar 3/0,
p = 0.25). With Sol the order reversed: long hand-offs made CTXAgent 16%
dearer than Codex at 200k and 2.2× on the 212-turn marathon. The saving
also depends on the price list: the same behaviour saves about 50% where
cache reads cost a tenth of input and about 0% where they cost a
fortieth.

So the three-way locus question below now has a fourth data point. A
prompt-only agent locus works for the strongest models, and whether it
pays depends on the model's hand-off length and the provider's cache-read
ratio. This is the dollar side of bai2026how's "cache reads dominate".

Limits:
- 3 seeds per cell, one marathon session; success differences not
  significant; no test reported for the seven-model comparison.
- Each vendor CLI ran only its own vendor's model, so harness and model
  are confounded across vendors.
- The memory probes were at ceiling in every arm, so the comparison is on
  cost and task score, not on what was lost.
- The 49% figure is against a Claude Code arm that never compacted.

Rule: if the backbone is Opus-class, a hand-off tool plus one paragraph
is a credible alternative to a threshold. Validate it per model on billed
cost, since the same prompt made Sol more expensive.

## A flush triggered by protocol collapse replaced the failure label, not the failure (2026-11-03)

The flush sections above (cai2026forkandflush, li2026humanize,
farahbakhsh2026agentdiscover) lean the same way: dropping the chat
and keeping the workspace can help a stalled run, though the evidence is
weak or observational in two of three. Those flushes run on a
schedule or a cost trigger. [[literature/papers/jain2026monitor]] tests a
flush on a *failure* trigger, same model, at matched call budget, with a
frozen success bar.

- **The design.** mini-swe-agent with `gpt-5.6-luna` on four hard SymPy
  issues, 5 baseline and 5 treated runs each, a shared 60-call cap. After
  three consecutive invalid tool responses, the treatment keeps the
  repository, drops the conversation, opens a fresh client, and supplies
  the issue plus a deterministic `git status` and diff-stat. It fired in
  18 of 20 runs.
- **The result.** Success 0/20 against 1/20 for the baseline.
  Repeated-format terminals went from 90% to 0%, and limit terminals from
  5% to 100%. Mean cost rose 56.7% and wall time 65.7%, because treated
  runs spent all 60 calls. The frozen bar (+5 pp or +2 successes, at most
  +20% cost) was not met.

For this concept: a flush can fix the symptom that triggered it and leave
the task where it was. It also inherits any wrong edits in the kept
workspace. Judge a flush policy on verified outcome and cost, not on
whether the run's failure mode changed. This does not contradict the
scheduled-flush gains: these four tasks had near-zero recoverable value
(1 of 20 baseline runs solved one), so no intervention had much to win.

Limits:
- 4 tasks, 40 runs, one cheap model. Too small to rule out a modest gain.
- The trigger is a format collapse, not a search plateau, and the tasks
  were picked because the baseline failed them.
- No arm separates "drop the chat" from "keep possibly wrong edits".

## Rewriting a summary from the last summary carried most of a handoff's loss in toy worlds, and the writer's view of the record, not its instructions, recovered it (2026-11-23)

Implementation guidance item 3 records summary-of-summary drift as "argued,
not measured" ([[literature/papers/nguyen2026cliffcompaction]]), and the
2026-11-16 section notes that [[literature/papers/zha2026do]] left the
re-compaction regime untested. [[literature/papers/liu2026agents]] is the
first matched test in this graph that separates compressing once from
compressing repeatedly. 56 small text worlds with hidden, checkable rules.
One agent plays eight lives with its whole history (Memory). A chain of
eight fresh agents each plays one life and inherits a 300-char note
written from the previous note plus that life (Relay). Two further chains
change one thing each: the note's writer may read the raw record of every
earlier life (Compaction relay), or the next agent gets the raw records
and no note (Record relay). Life g is the same instance in every arm.

- **Every model loses at a handoff.** Memory − Relay is 0.26–0.67 marks of
  5 across six models (19–48% of what Memory learns), all intervals above
  zero. The eighth Relay life sits where Memory stood after about four
  lives for four models, and 2.5–3.0 for the two weakest.
- **On each model's 10 worst worlds, most of the loss is the
  note-from-note step.** Same 300 chars, writer reads the record:
  - gpt-6.1-sol recovers 1.82 of 2.59 marks (70%)
  - gpt-6-astra 1.08 of 1.31 (82%)
  - gpt-5.6-terra 0.81 of 2.03 (40%); for it one compression costs more
    (0.92)

  Over all 56 worlds sol recovers 0.39 of 0.64 (61%). A fresh agent given
  the raw record matches the agent that lived it (−0.49 to +0.30).
- **Stated content mostly survives a hop; what never got concluded is
  lost.** A fact right in one Relay note is right in the next 87–89% of
  the time. An absent fact appears in only 14–15%. Relay notes plateau at
  about 50% of a world's facts by life 5; written from the record they
  reach 47–54% against 24–40% on the same worlds. Failing chains' notes
  are mostly single-game records (65% for sol), not false rules. The loss
  sits in worlds a model needs 3+ lives to work out (0.62–1.01 marks
  against 0.03 for worlds learned within two).
- **Instructions to the writer did not help; length did.** Revise-the-note,
  a confirmed / ruled-out / open ledger, and "mark what was verified and
  record what you could not explain" all scored at or below the plain
  request (sol, 10 worlds). A plain Relay note at 1,500 chars (3.83)
  matched or beat a 300-char note written from the record (3.47).

This is the same direction as [[literature/papers/xu2026what]] (a rewrite
from full context beats the agent's own running reports at matched
length) and [[literature/papers/goyal2026does]] (most of a note's deficit
is lost at write time). It refines this page's drift worry. Restating a
summary did not degrade what it already said. A chain whose writer sees
only the last summary plus the newest span cannot assemble a conclusion
that needs evidence spread across spans. In this design that is what
"repeated compression" costs; paraphrase drift in the strict sense is not
isolated.

Rule for this project (our inference): where a summary is rewritten many
times and its record still fits the writer's window (subagent briefs,
session hand-offs, a `NOTES.md` "Next" carried session to session),
regenerate it from the record (journal, logs, git history, transcript),
not from the previous summary. At overflow compaction the record does not
fit by definition. There the usable forms are a longer summary and a
record the writer is made to consult, not one it is offered. Do not try
to fix the loss with writer wording.

Limits:
- Induced rules in toy worlds, not stated constraints or real tasks. The
  author does not claim the size of the loss transfers.
- A 300-char budget. At 1,500 chars the note-from-note penalty was gone
  for sol. The size of the effect at Claude Code's summary lengths is
  unknown.
- The 70–82% is on worlds selected by the outcome and partly reuses the
  selection runs. The all-worlds sol figure (61%) is the safer number.
- Single author, no code, worlds or prompts released. Variant and length
  results are one model, 10 worlds, no intervals. No Claude model.

## A harness slot carrying the explorer's last finding across compaction tied no memory (47 vs 48 of 98), cost about 2,000 tokens a request, raised compactions, and was filled at only 47–65% of compactions (2026-11-24)

Implementation guidance item 2 anchors a structural prefix across
compactions. The 2026-09-29 note records a state-gated injection that
[[literature/papers/srikanth2026recursive]]'s evolved harness kept.
[[literature/papers/passos2026one]] is the first pre-registered paired test
in this graph of a harness-side carry-over slot. In the Gemma 4 Developer
Agent harness (Gemma 4 31B QAT, compaction at a 14,336-token previous
prompt, 32k limit), two slots fill on every coder call after a compaction:
- the explorer sub-agent's latest answer (≤15 lines);
- a notes page the agent writes through a notes tool (≤1,500 tokens). The
  agent is asked to write after each explorer answer and each edit.

The arm began as agent-requested recall. Six smoke runs wrote no notes, so
before any result it became automatic, the pre-registered fallback.
- **No resolve-rate effect.** 98 paired SWE-bench-family tasks: 47 vs 48,
  −1.0 [−8.3, +6.3]. The first 48 gave +8.3 [−2.1, +18.3] and 50 fresh ones
  −10.0 (see [[concepts/pass-at-k]]). The registered secondary, repeated
  actions per compaction, was also null: +0.30 [−4.50, +5.98].
- **The agent did not write memory.** Notes appeared in 2 of 16 Modal and 4
  of 32 Kaggle runs. All 11 `plumbing_failed` Modal runs were adherence
  failures: 4 never ran the first recall, and 7 had nothing to inject.
- **Delivery was partial even when automatic.** Something was injected at 21
  of 45 compactions (Modal) and 50 of 77 (Kaggle). Nothing can be injected
  before the first explorer call.
- **The slot has a context price.** The plumbing adds about 2,000 tokens to
  every request (median first request 3,966 vs 2,311). Compactions rose from
  23 to 45 on Modal (post hoc, +1.38 per task [+0.31, +2.69]) and from 71 to
  77 on Kaggle. Kaggle overflows were 7 vs 5. The trigger reads the previous
  prompt's size, so one large step can overflow before compaction runs.

For this concept: a slot holding the last sub-agent answer is not the
content this page's evidence says summaries lose. That content is stated-once
constraints, failures, rejected options and non-re-derivable observations
(2026-10-20, 10-27 and 11-16 sections). So the null does not count against
carrying those. It does say three things about any carry-over. A slot filled
from agent output is empty wherever the agent produced nothing. A fixed
addition to every request moves a size-based trigger earlier and raises
overflow risk. And an intention-to-treat comparison then measures "feature
on", not "state carried". The adherence result is a fifth "offered, rarely
used" case after the [[concepts/lossless-context-offload]] series, here on
writes rather than reads, and on a small model. Opus-class models do write
hand-offs when asked ([[literature/papers/busch2026ctxagent]]). The paper's
own next step is the 11-23 rule from the other side: build the recap from
the harness's tool history, so it needs no cooperation and is present at
every compaction.

Rule for this project (our inference):
- A carry-over slot logs, per compaction, whether it was filled. Report the
  delivered share beside the outcome.
- Fill carry-over from the harness record (tool calls, diffs, failures), not
  from content the agent is asked to write, unless the backbone is shown to
  write it.
- Price the slot against the compaction trigger: report compactions and
  overflows per arm, and compare with a token-matched filler arm before
  attributing a result to the content.

Limits:
- One quantized 31B model, a 32k window, Python repository tasks, a
  subprocess sandbox rather than the official scorer.
- The design resolves only effects above about ±12 points combined.
- With delivery under 70%, per-protocol is 26 pairs (13 vs 11) and
  conditions on a post-treatment variable.
- No filler arm, so the content's value and the extra compactions' cost are
  not separated. The misleading-memory control was dropped for budget.

## Twelve frontier compactors at a 10% budget kept a successful run alive 46–69% of the time; most failures were omissions paid for in actions, and a carried fact's status mattered as much as its presence (2026-11-24)

The "Compaction fidelity" open question asks how much a compaction step
loses. The sections above answer it on recall probes
([[literature/papers/xu2026what]]), planted rules
([[literature/papers/zha2026do]]), invented policy
([[literature/papers/chu2026when]]) and toy-world handoffs
([[literature/papers/liu2026agents]]).
[[literature/papers/chen2026compactbench]] (CompAct-Bench) scores it on
whether real execution survives.

- **The design.**
  - Successful runs on BrowseComp, SWE-bench Verified and an internal
    workspace suite (200 instances) are cut at 25/50/75% of their actions.
  - Twelve frontier models compress the history before the cut to 10% of
    its characters, all with one shared prompt. An iterative calibration
    loop matches lengths: 86.7% of compactions land within ±3%.
  - A fixed executor resumes from the replayed environment, with exactly
    the actions the source run had left. It is Opus 4.8, or GPT-5.5 Codex
    for the workspace suite.
- **Survival is 46.0–68.5%, mean 56.7%.** Claude Opus 5 is best (68.5
  [62.0, 74.5]), and the top and bottom intervals do not overlap. Every
  source task had passed 3 of 3 screens. But **no full-history
  continuation under the same budget was run**, so the gap to 100% is an
  upper bound on what compaction costs, not a measurement of it.
- **Most failures are omission, then re-acquisition, then no actions
  left.**
  - 91.6–97.7% of failed continuations used up the budget. Early stops
    are about 2–8%.
  - An LLM annotator labels 25.3% of the units needed to continue as
    omitted and 5.6% as distorted.
  - Critical omission goes with +18.2 points more repeated work. The
    executor tries to recover lost state in 95.1% of applicable failures.

  This is the failure side of [[literature/papers/bai2026how]]'s redundant
  re-reading. A lossy summary turns into re-work, and under a fixed step
  budget the re-work is the failure.
- **A fact's status carries weight, not only its content.** "Premature
  commitment" (an unresolved hypothesis written as settled) is in 56.7% of
  early failures and 46.0% of budget exhaustions. Two case studies are the
  cleaner evidence.
  - Case F: all twelve summaries state the same disqualifying fact (8
    authors, 7 required). The three that write it as a decision all
    succeed, and so does one that lists it with no next step. Five of the
    eight that keep it open use up a 39-action budget re-checking it.
  - Case C: summaries that assert a half-matching candidate lead to
    two-round wrong submissions. Only the two that keep alternatives alive
    succeed.

  A fact-retention score would rate each set of twelve alike.
- **History length does not predict difficulty.** Within a domain,
  |ρ| ≤ 0.03. In Case E a 7-round, 35.8k-character prefix fails for every
  compactor. The answer sat only in results the agent had labelled spam,
  and nine summaries cut it along with the spam. That is
  [[literature/papers/lencevicius2026when]]'s observed-once fact, dropped
  here because the agent's own label marked it as noise.

What this supports:
- The 2026-10-20 rule (fixed fields for failures and rejected options)
  gets its first execution-level evidence with frontier compactors, from
  Case F. A rejection carried with its reason cost the reader nothing. The
  same fact carried as a question cost up to the whole budget.
- It agrees in direction with liu2026agents. It does not contradict that
  paper's null for a confirmed / ruled-out / open ledger instruction,
  because no prompt variant is tested here.

What it does not support: a loss rate for any production harness.
- The prompt is a generic "summarize concisely" at 10% of characters. That
  is far harsher than Claude Code's structured compaction prompt.
- The calibration loop's first stage re-compresses the model's own
  previous output before a full-history refinement stage. Calls per
  compaction differ by model (4.6–11.8).

Rules (ours, extending the 2026-11-16 "count what the summary adds"). When
evaluating a compaction change:
1. Run a full-history arm under the same action budget.
2. Score each carried candidate's status (confirmed, rejected with its
   reason, or open), not only whether it is present.
3. Report early stops and budget exhaustion separately. They have
   different causes.

Limits:
- No full-history arm. Budgets are set by one successful rollout, and can
  be as small as 3 actions.
- The error labels come from one LLM annotator (GPT-5.6-Sol), computed
  over failed continuations only. There is no rate for successes, and the
  human review is not quantified. The status contrasts are single selected
  cases.
- One prompt, one ratio, and one executor per domain. Opus 4.8 executes
  and Opus 5 is the best compactor, so a same-family match is possible.
  The workspace suite is internal, n = 25.

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
  **Partly measured (2026-09-15), on a capped store rather than the prompt
  buffer.** [[literature/papers/shen2026what]] splits errors under budgeted
  eviction into *irreversible* (evidence evicted) and *recoverable* (evidence
  survived, retriever missed it), and the split depends on pressure: on
  LongMemEval-S retrieval misses are ~27–40% of restore-fixable errors at an
  80k budget but ≤2% at 30k and 8k — a better retriever buys nothing until
  more is retained. Two rules follow for evaluating any policy here. A
  budget–accuracy curve is not comparable across papers unless the read-time
  retrieval regime is reported (forced-gold reading empties the recoverable
  bin by construction). And question-blind importance scoring is not a lever
  under pressure: an LLM rater that never sees the question is
  indistinguishable from FIFO and random on irreversible loss at matched
  accuracy at 30k and 8k (resolution 1.2–6 pp), though it leads at 80k where
  accuracies are not matched. That is the write-before-query barrier
  ([[literature/papers/zhu2026lossy]]) appearing in a policy comparison;
  dependency-aware policies ([[literature/papers/hao2026selfgc]]) were not
  tested.
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
