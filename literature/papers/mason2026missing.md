---
kind: paper
title: "The Missing Memory Hierarchy: Demand Paging for LLM Context Windows"
authors: ["Tony Mason"]
institutions: ["University of British Columbia", "Georgia Institute of Technology"]
year: 2026
venue: "arXiv 2603.09023v1, cs.OS (preprint, 2026-03-09; PDF still carries an unedited SOSP '17 ACM template header)"
peer_reviewed: false
url: https://arxiv.org/abs/2603.09023
code_url: https://github.com/fsgeek/pichay
citations:
source: "raw/papers/mason2026missing.pdf"
added: "2026-08-17"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-proprioception]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/budget-as-ceiling]]"
tags: [memory, context-window, eviction, demand-paging, working-set, page-fault, virtual-memory, cost-model, prefix-cache, claude-code, production]
---

# The Missing Memory Hierarchy: Demand Paging for LLM Context Windows

## TL;DR

Argues the context window is not memory but **L1 cache**, and that the
field has built ever-larger L1 instead of the rest of the hierarchy.
Instruments 857 production Claude Code sessions (54,170 API calls, 4.45B
effective input tokens) to show **21.8% of input tokens are structural
waste**, then builds Pichay — a transparent proxy between client and
inference API that evicts stale content, detects *page faults* when the
model re-requests evicted material, and pins the working set from fault
history. The best part is theoretical: the LLM cost model is **inverted**
relative to classical virtual memory (keeping is expensive, faulting is
cheap), which changes the optimal replacement policy and makes
aggressive eviction correct by default — with a twist at high fill.

## Claims

- **The context window is L1, not RAM.** The hierarchy is L1 (generation
  window) → L2 (working set, demand-paged and pinned) → L3 (session
  history, compressed with declared losses) → L4 (cross-session
  persistent memory, retrieved by graph traversal or similarity) →
  Storage (the full corpus, archived and never resident unless
  demanded). The OS analogy is structural, not metaphorical: page table
  = retrieval handles/anchors, page fault = model re-requests evicted
  content, MMU = the proxy layer, working set = currently relevant
  context subset, thrashing = evict/fault cycle exceeding useful work,
  pinning = fault history preventing re-eviction. Enlarging the window
  is "building machines with more physical RAM instead of inventing
  virtual memory."
- **21.8% of input tokens are structural waste**, with a measured
  taxonomy: dead tool output 26.5% of request bytes, tool-definition
  schemas for unused tools 20.2%, static system-prompt re-send 11.0%,
  skill triplication 2.9% — 60.5% of request bytes addressable in the
  detailed 99-call sample, projecting to 21.8% of the corpus's input
  tokens (970.4M of 4.45B). Claude Code sends 18 tool definitions
  (63,088 bytes) on every call while the median session uses 3;
  ~52,500 bytes of never-invoked schemas are tokenized and attended on
  every call. The skills list is sent three times under different
  prefixes. "These are not implementation bugs. They are the inevitable
  consequence of managing a finite resource without the abstractions
  operating systems developed for the analogous problem fifty years ago."
- **The inverted cost model — the paper's most transferable idea.** In
  classical VM, keeping a page is free and faulting costs disk I/O, so
  every replacement algorithm minimizes faults. In LLM context, *keeping*
  costs tokens on every turn and *faulting* costs once. A 5,000-token
  file resident for 20 turns costs 100,000 input tokens; re-reading it
  when needed costs 5,000. The objective is therefore
  `min Σ [C_keep(p) + C_fault(p)]`, with break-even
  `|p|·T_until_next_ref·c_token > |p|·c_token`, i.e. **evict whenever the
  page will not be referenced for more than one turn**. This is why
  FIFO — the worst classical policy — works well here, and why
  sophistication is needed only to avoid evicting pages needed on the
  *very* next turn. Belady's MIN is not optimal under inverted costs; the
  optimal policy is cost-weighted.
- **Fault cost is non-linear in context size, which reverses the pressure
  instinct.** A fault requires an extra full inference pass, and
  attention is O(n²) in sequence length, so true fault cost scales with
  n² at the current fill, not with the page size. At low fill (~40K)
  faults are cheap and aggressive eviction is right; at high fill
  (>100K) faults become expensive, so **the eviction policy should become
  more conservative as context pressure rises** — the opposite of the
  naive instinct to evict harder under pressure.
- **Size matters to eviction priority in a way it does not in classical
  VM,** where all pages are 4KB. A 10,000-token page costs ten times as
  much per turn as a 1,000-token one, so large pages should be evicted
  eagerly unless access frequency justifies the keep cost; recalled
  objects vary from a 3-line summary to a 200-line file. The paper argues
  for a **size-aware, fill-sensitive** replacement policy with no direct
  analogue in the VM literature.
- **Structural mutations pay a prefix-cache invalidation cost that can
  exceed their savings.** A collapse compressing 12 turns of orientation
  dialogue dropped the cache hit rate from 100% to 25% for one turn
  (~105K tokens of recompute, comparable to several page faults) before
  recovering. Hence: **batch** structural mutations and pay the
  invalidation once, and prefer infrequent large collapses to frequent
  small ones. "A collapse that saves 10KB of context but invalidates a
  100K-token cached prefix is a net loss."
- **Pins should decay.** One fault currently pins a page permanently, but
  a fault says the content was needed *then*, not forever; under the
  inverted cost model a pin's strength should halve every K turns since
  last access, giving LRU-like behavior with cost-weighted decay and
  preventing the monotonic working-set growth permanent pinning causes.
- **Cooperative memory management is a new design point.** Classical
  replacement algorithms assume a non-cooperative application; an LLM has
  an *incentive* to cooperate (cleaner context = better output = longer
  session). Pichay opens two side channels the framework never sees:
  **phantom tools** (proxy→model capabilities: `memory_release(paths)`
  voluntarily releases cold pages — the reference bit, supplied
  voluntarily; `memory_fault(paths)` requests restoration from the
  proxy's eviction cache, faster and cheaper than a real Read round-trip)
  and **cleanup tags** (model→proxy directives parsed from output text:
  `drop`, `summarize`, `anchor`, `collapse: turns N-M`).
- **Graduated pressure zones instead of a binary threshold.** Normal
  (<60K): observe only. Advisory (60K–100K): inject memory-pressure
  information into the model's context — fill percentage, the five
  largest resident blocks, available cleanup operations — the equivalent
  of a desktop "low memory" notification, chosen to leave ~40K tokens of
  runway so the model can finish a coherent thought and emit cleanup tags
  before losing agency. Involuntary (100K–120K): automatic FIFO eviction,
  model informed but not consulted. Aggressive (≥120K): emergency
  eviction, context survival over working-set preservation.
- **Retrieval handles are late-binding anchors, and they resolve to
  *current* content.** The eviction summary — `[Paged out: Read
  /path/to/file.py (12,450 bytes, 287 lines). Re-read if you need its
  content.]` — was designed as a space-saving marker (~200 bytes
  regardless of original size) but functions as an anchor storing minimal
  metadata. A file edited since eviction materializes at its *new* state
  when faulted, so **stale cached content is structurally impossible**.
  Behavioral evidence: a fresh model instance resuming a session with
  paged-out content unprompted said "Let me re-read the files I need
  since some were paged out" — the handle's format is self-describing
  and no instruction was needed.
- **Garbage collection and paging must be counted separately.** Ephemeral
  tool output (Bash results, search output, directory listings) has no
  stable identity and cannot be meaningfully re-requested — removing it is
  reclamation of dead content and *cannot* cause a fault. Only paging of
  addressable content (file reads, plan documents, specifications) can.
  Conflating them "inflates the eviction denominator and deflates the
  apparent fault rate."
- **Amplification factor quantifies reprocessing.** Median A = 84.4×
  (P75 217.9×, P90 570.8×) for main sessions, 12.8× for short-lived
  subagents, scaling linearly with session length at ratio ~0.5 —
  confirming quadratic cumulative cost. Agentic coding is overwhelmingly
  input-bound: 82,061 effective input tokens per call against 88 output
  tokens, a **933:1 ratio**, with a 93.5% cache hit ratio — cached tokens
  still occupy the window and still require attention for every output
  token.

## Methods

- **Corpus**: 857 Claude Code sessions across 15 software projects from a
  single power user, Nov 2025 – Mar 2026 (59 main, 567 subagent, 154
  compact continuations, 21 prompt-suggestion), 54,170 API calls, 4.45B
  effective input tokens. Frozen cohort with a released recount script
  and a fixed counting protocol; bias explicitly declared as
  single-user.
- **Three instruments**: `probe.py` (streaming JSONL analyzer over raw
  session transcripts, no API calls), `proxy.py` (transparent HTTP proxy
  between Claude Code and the Anthropic API — the "MMU", logging every
  decision to JSONL), and `pichay` (paired-run experiment framework;
  self-bootstrapping — built by Claude Code running through its own
  proxy).
- **Three treatments** at the proxy layer: baseline (observe only),
  trimmed (tool-definition stubbing + skill dedup), compact+trim (adds
  stale-result eviction). Same model (Claude Opus), temperature, and
  system prompts throughout; interposition transparent to both client and
  API.
- **Eviction policy under test**: deliberately minimal FIFO by user-turn
  age (τ = 4 user-turns, s_min = 500 bytes), replacing evicted content
  with a retrieval handle; error results never evicted.
- **Eviction safety**: offline replay of 29 proxy-captured sessions,
  1,393,000 simulated evictions with no API calls.
- **Quality check**: paired LLM-judged non-inferiority on 18 sessions
  (54 verdicts), treatment replacing consumed tool results outside a
  20-message recency window with tombstones; three judges (one Sonnet 4,
  two Haiku 4.5) scoring correctness/completeness/coherence 1–5 with
  randomized A/B position.

## Results

- **Eviction safety**: 354 page faults in 1,393,000 simulated evictions
  = **0.0254% fault rate** over 8.49 GB of evicted content. A rate of
  exactly zero would indicate over-conservative eviction; some faults are
  expected and acceptable.
- **Controlled treatment comparison**: 37.1% reduction in effective input
  tokens (114,222 → 71,816) under compact+trim, with cache reads down
  59.6% and the task completing correctly under all three conditions.
- **Corpus-scale projection**: 970.4M tokens addressable (21.8% of
  input) — tool stub trimming 487.5M (11.0%), static re-send 387.0M
  (8.7%), skill dedup 95.8M (2.2%); average 17,913 tokens saved per API
  call, translating to ~85 billion fewer token-token attention pairs
  across the corpus.
- **Production session A (steady-state coding)**: context free rose from
  7% (imminent exhaustion) to 43% — 36 percentage points of recovered
  capacity. 15 evictions, of which 11 were garbage collection and 4 were
  pageable Read evictions with 1 fault. The single fault is instructive:
  a plan file read early and evicted on age, needed for the whole session
  — "a hot page that FIFO treats as cold. This is the classic working set
  failure: the eviction policy measures age, not access pattern."
- **Production session B (681-turn multi-agent coordination) — the
  honest negative result.** 680 evictions, 659 page faults = **97% fault
  rate**, peak compression 5,038KB → 339KB. Three named patterns:
  a *thrashing cycle* (three files faulted repeatedly across turns
  163–170+, working set exceeding resident set), a *sequential scan*
  (planning across three repos re-reading the same 7 files, working set
  larger than the age threshold allowed), and **self-inflicted
  inflation** — the 5,038KB pre-compaction size *includes re-reads caused
  by previous evictions, so the proxy was measuring its own overhead as
  "bytes saved," a metric artifact of thrashing*. The session died on the
  API **rate limit**, not the context limit: at 659 faults (each an
  inference-priced round trip) thrashing consumed the rate budget faster
  than useful work. "Fault cost is not merely computational but
  monetary."
- **Quality non-inferiority**: judges preferred the treatment *more*
  often than baseline (37% vs 28%, 35% ties); mean correctness 3.89 →
  3.74, completeness 3.59 → 3.59, coherence 3.74 → 3.69, max score Δ
  0.15; detection rate 57% (p = 0.14, not significant) at mean 48%
  compression.
- **Why the waste exists** (four reinforcing causes): training data
  consists of monotonically growing conversations with no examples of
  intelligent content removal; the Messages API is a list whose natural
  operation is append and which has no way to say "this content is
  stale"; every orchestration framework appends by default; and token
  consumption is billed *after the fact*, so there is **no backpressure
  signal** — quality degradation from context bloat is invisible unless
  measured.

## Critique / open questions

- **Single user, single tool.** 857 sessions from one power user on one
  assistant. The author declares the bias, argues the waste patterns are
  structural consequences of the architecture rather than of the user,
  and identifies multi-user validation as future work — but the specific
  percentages should be treated as one system's profile, not a constant.
- **The headline production result is a pathology.** Session B's 97% fault
  rate is reported honestly and analyzed well, but it means the deployed
  policy failed on the paper's own most demanding workload. The
  93%-compression figure in the abstract comes from that session, and its
  denominator is partly self-inflicted re-reads — the paper says so, but
  the abstract does not.
- The good theory is not the implemented policy. The inverted cost model,
  size-aware/fill-sensitive replacement, pin decay, and collapse batching
  are all *argued* in the discussion; what ran in production is FIFO with
  permanent pinning. The strongest content is therefore a design
  prescription awaiting evaluation.
- L3 (rolling compaction) is implemented but not evaluated at scale; L4
  (cross-session memory) is design only.
- Quality evidence is a 54-verdict LLM-judged non-inferiority check, not
  a task-success benchmark. It is honestly framed as non-inferiority and
  the detection result is n.s., but "judges slightly prefer the treatment"
  on 18 sessions is thin.
- The PDF still carries an unedited SOSP '17 ACM template header ("SOSP
  '17, Shanghai, China, 2017"), which is a draft artifact rather than a
  venue claim — worth noting because it is easy to miscite.

## Trust signals

- **Credibility:** 3 — real production deployment with a released
  implementation (`github.com/fsgeek/pichay`), a frozen cohort plus a
  released recount script, an offline replay at 1.4M-eviction scale, and
  unusually candid reporting of its own worst case (the 97%-fault
  thrashing session, including that its savings metric was
  self-inflated). Held at 3: single-author preprint, no peer review,
  single-user single-assistant corpus, the strongest theoretical claims
  unevaluated, quality evidence limited to 54 LLM-judged verdicts, and
  stale template metadata suggesting an early draft.

## Follow-up

- **Relevance:** 5 — the canonical cost-model source for
  [[concepts/context-eviction-policy]] (the inverted objective, the
  non-linear fault cost that reverses the pressure instinct,
  size-awareness, pin decay, cache-invalidation batching), a sixth
  attestation for [[concepts/lossless-context-offload]] with the
  handle-resolves-to-current-content property, and the **second**
  attestation for [[concepts/context-proprioception]] — the graduated
  pressure zones are that concept deployed, and the NOTES watch item for
  a second source is now satisfied.
- Also the only source here whose corpus *is* Claude Code, which makes
  its waste taxonomy directly actionable rather than analogical: 18 tool
  definitions on every call against a median of 3 used, ~52.5KB of unused
  schemas per call, skills listed three times. Worth measuring on this
  box before assuming the numbers transfer.
- The `memory_release` / `memory_fault` phantom-tool pair is the
  cooperative counterpart to [[literature/papers/li2026acm]]'s
  agent-initiated `manage_context` / `query_memory`, and the two together
  make the case that the policy locus should be *both* — proxy informs,
  model directs. Neither alone.
- Directly strengthens the pending `precompact-addressable-offload`
  system proposal: the retrieval-handle format is a working design for
  what a pre-compact hook should leave behind, and the
  handle-resolves-to-current-content property is a reason to store a
  path plus content hash rather than the content itself.
- The "no backpressure signal" diagnosis is the sharpest statement of why
  this whole cluster exists: billing is post-hoc and quality degradation
  is invisible, so nothing in the loop pushes back. Compare
  [[concepts/spend-forecast-calibration]] — the missing signal is the same
  one a forecast would supply.
- Open thread worth tracking: the paper's own thrashing analysis implies a
  **fault-rate metric belongs in the harness**, since a 97% fault rate was
  only visible because the proxy logged it. A compaction hook that offloads
  without counting re-reads cannot tell savings from self-inflicted churn.
