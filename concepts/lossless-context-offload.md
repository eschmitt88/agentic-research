---
kind: concept
name: "lossless-context-offload"
status: seedling
added: "2026-08-10"
sources:
  - "[[literature/papers/li2026acm]]"
  - "[[literature/papers/semenov2026beyond]]"
  - "[[literature/papers/hao2026selfgc]]"
  - "[[literature/papers/dang2026addressable]]"
  - "[[literature/papers/xu2026llm]]"
  - "[[literature/papers/mason2026missing]]"
  - "[[literature/papers/cheng2026agenticsts]]"
  - "[[literature/papers/zhu2026lossy]]"
  - "[[literature/papers/badhe2026skill]]"
  - "[[literature/papers/song2026string]]"
used_by: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/file-as-bus]]"
related_experiments: []
tags: [memory, context-window, eviction, recall, addressability, runtime-policy, lossless]
---

# lossless-context-offload

## Definition

Eviction from the context window is an *offload*, not a deletion:
every compression step writes the raw evicted span to an external
store under a **stable address** (identifier, pointer, file path), and
the summary left in context carries that address so the agent can
recall the exact original content on demand. Eviction and retrieval
stop being independent policies and become the two halves of one
protocol — compress out / query back — whose invariant is that no
information is destroyed, only demoted.

## Why it matters here

[[concepts/context-eviction-policy]] establishes *what* leaves the
window and when; [[concepts/selective-memory-retrieval]] establishes
*when* to read from the substrate. This concept names the contract
that joins them: the evicted content must remain **addressable**, or
the read-side policy has nothing precise to read. Compaction without
addressable offload is the failure mode chen2026governance and
semenov2026beyond document — unpredictable lossiness, destroyed causal
structure, compression-induced hallucination — because the raw span is
gone by the time the agent discovers the summary dropped what
mattered.

Five independent attestations, five mechanisms, one invariant:

- **ACM** ([[literature/papers/li2026acm]]): `manage_context` assigns
  each summary a unique identifier mapping to the raw messages in
  external storage; `query_memory` recalls by identifier through a
  querier LLM. Explicitly billed as the "lossless" property that
  distinguishes it from ReSum/ACON/Mem1-style summary agents
  (Table 1), and the recall path is exercised in real trajectories
  (Fig. 6: `query_memory` probes interleaved with compression on a
  5-constraint multi-hop question the base model can never finish).
- **Self-GC** ([[literature/papers/hao2026selfgc]]): `fold` moves an
  exact payload to a sidecar and leaves a recovery pointer on the
  control plane; only `prune` is destructive, and it is reserved for
  provably obsolete content.
- **CWL** ([[literature/papers/semenov2026beyond]]): evicts action
  records whose effects are *already persisted in the environment* —
  the environment itself is the addressable store, and the episode
  annotation is the address.
- **ARC** ([[literature/papers/dang2026addressable]]): every tool
  observation hashed into an append-only content-addressed store;
  eviction leaves a `§id` citation the agent dereferences with a
  budgeted `_recall`. The only formal attestation: an (informal)
  theorem that the active view stays ≤ budget while every stored
  observation is exactly reconstructible, plus a lower-bound argument
  that linear external-memory growth is unavoidable for worst-case
  exact recovery. Also the first to price the invariant in serving
  cost: 38.8–73.5% HBM-traffic reduction alongside the accuracy gain,
  so lossless offload is cheaper to *serve*, not just safer.
- **VISTA** ([[literature/papers/xu2026llm]]): `archive(S, ρ)` leaves
  a handle carrying path, level, size, and checksum; recovery is an
  *ordinary file read* of the archive path — no third context tool,
  no retrieval oracle — and archiving is hierarchical (bundles
  re-archived into coarser handles as pressure grows). Its Prop. 1 is
  the invariant's formal floor: under budget pressure, any
  non-recovering method's correctness is information-theoretically
  bounded, regardless of how clever the compression is.

The disagreement between them is the open axis inherited from
context-eviction-policy — who initiates (trained agent vs
deterministic policy vs planner-under-harness) — but all five agree
on the invariant, which is therefore the more settled claim and the
one worth importing first. ARC sharpens the axis by *splitting* the
locus: its write side (compaction) is fully deterministic — no LLM
call — while its read side (recall) is agentic. The two policies the
protocol joins need not sit at the same locus, and ARC's framing
says why: "recovery is a decision that the agent makes," whereas
what-to-show under a hard budget is arithmetic a harness can do.

## Implementation guidance (provisional — seedling)

1. **Pair the tools.** An offload operation without its matching
   recall operation is truncation with extra steps. Ship
   `manage_context`-style and `query_memory`-style operations
   together, or neither.
2. **Addresses live on the control plane.** Self-GC attaches recovery
   pointers to user/harness messages, ACM to summary identifiers —
   never as assistant prose, which later turns will imitate.
3. **Recall is the residual lossiness.** ACM's querier LLM answers a
   query *about* the raw span rather than returning it verbatim;
   errors there are the new information loss. Audit the recall path,
   not just the storage path.
4. For this project: the `raw/` → `literature/` pipeline already
   implements the invariant at curation scale (raw immutable, notes
   addressable by citekey); the in-session analogue would be
   compaction summaries that cite the journal/transcript span they
   replaced.

## The handle resolves to *current* content, not archived content

[[literature/papers/mason2026missing]] adds a property the offload sources
so far have left implicit, and it is the one that makes handles safer than
archives. Its eviction summary — `[Paged out: Read /path/to/file.py (12,450
bytes, 287 lines). Re-read if you need its content.]` — was designed as a
space-saving marker (~200 bytes regardless of the original size) but
functions as a **late-binding retrieval handle**: it stores minimal metadata
and resolves on demand. Because it stores a *path*, not a payload, a file
edited since eviction materializes at its new state when faulted, so **stale
cached content is structurally impossible**. An offload that stores the bytes
has to solve invalidation; an offload that stores an address does not.

Two further observations transfer. The format is self-describing enough to
need no instruction: a fresh model instance resuming a session containing
paged-out content stated unprompted, "Let me re-read the files I need since
some were paged out, then create the task list and start implementing" —
it recognized the handles, inferred the recovery mechanism from the summary
text alone, and chose to fault content in before acting. And the paper's
cooperative side channel gives the *agent-side* half of the same protocol:
phantom tools `memory_release(paths)` (the model voluntarily releases cold
pages — a reference bit supplied rather than inferred) and
`memory_fault(paths)` (the model requests restoration from the proxy's
eviction cache, cheaper than a real Read round-trip). Paired with
[[literature/papers/li2026acm]]'s agent-initiated `manage_context` /
`query_memory`, the design space now has both directions instrumented, which
is the practical argument that the policy locus should be *both* — the
harness informs, the agent directs — rather than either alone.

Scale evidence, on a corpus that happens to be this project's own harness:
1,393,000 simulated evictions at a **0.0254% fault rate** over 8.49 GB of
evicted content, and 37.1% fewer effective input tokens in a controlled
paired run with the task still completing correctly. Lossless offload is not
a research aspiration at this point; a deliberately minimal FIFO policy
achieves it.

## Connections

- **[[concepts/context-eviction-policy]]** — the parent axis; this
  concept is the invariant a good eviction policy maintains.
- **[[concepts/selective-memory-retrieval]]** — the read-side policy
  that addressable offload makes precise: the gate can target an
  identifier instead of a similarity search.
- **[[concepts/agent-native-memory]]** — the substrate the offload
  lands in; provenance discipline there is what keeps addresses
  stable.
- **[[concepts/file-as-bus]]** — same move at inter-agent scope:
  durable addressable artifacts instead of ephemeral message passing.

## Mandatory offload: the same mechanism with the opposite default

This concept moves material to a colder tier while keeping it addressable —
an offload taken when the window is under pressure.
[[literature/papers/cheng2026agenticsts]] runs the same mechanism with the
default inverted: **"any information that survives across decisions must
first be written into a bounded store."** Nothing is offloaded because
nothing was implicitly resident; the store is the only path across a
decision boundary, and the prompt is rebuilt from typed slots each time.

The pair is worth holding together because it isolates what this concept's
real variable is. Both designs are lossless in the sense that matters
(material remains reachable), both are deterministic, and both bound
growth. They differ only on whether **residence is the default and offload
the exception**, or offload is the default and residence must be earned by
an explicit write. mason2026missing's page-fault restore is the reactive
form; this is the pre-emptive form.

For this box, the pre-emptive version is the stricter and more auditable
discipline — but note it comes from a closed-rule game with an enumerable
action space, and the paper never established that it performs better than
accumulation (its win-rate differences are directional, p ≈ 0.37, and the
same-codebase accumulating-context comparison was not run).

## Why addressability is a safety property, not an optimization

[[literature/papers/zhu2026lossy]] gives this concept the argument it has
been making on cost grounds a much harder form, and names it: the
**write-before-query barrier**.

Compression "forces irreversible retention decisions under uncertainty,
requiring a preemptive bet on saliency before the query distribution is
known." The consequence is stated as an impossibility rather than a quality
problem: **any fixed-budget summary admits a worst-case query it cannot
support with traceable evidence.** No better summarizer escapes it, because
the deficiency is in the ordering — the write happens before the query
exists. The paper's example is a "severe peanut allergy" compressed to
"dietary preferences"; the later question "is this snack safe?" is then
unanswerable *with a citable source*, which is a different and worse
failure than answering it wrongly.

That is why keeping offloaded material addressable is not a nice-to-have:
addressability is what makes taking the summary **safe in the first place**.
A lossy compaction with a live pointer back to the source is a bet you can
lose and recover from; the same compaction without the pointer is a bet you
cannot audit.

The measured scale of the problem: summary-centric memory systems on LoCoMo
carry **Unverifiable Omission Rates of 14.7–23.3%** — errors caused by the
evidence being absent rather than misread — rising to ~30% on the
longer-range LongMemEval. So roughly one error in five to one in three is
an omission the summary tier cannot even flag.

TierMem's resolution is the same shape mason2026missing's page fault takes,
one level up: default to the cheap tier, detect insufficiency at query
time, escalate to the immutable raw store, and — the addition — **write the
verified finding back with its provenance links intact**. Across three
replay epochs that moves queries onto the cheap path without losing
accuracy, which is what amortization of an offload looks like when it
works.

## Open questions

- Identifier stability under long horizons and revision-heavy
  workloads (the lee2026minteval regime): what happens when the store
  itself is compacted or the identifiers outlive their summaries?
- Cost accounting: lossless storage is cheap, but the recall path
  (querier LLM per probe) is metered — is offload+recall actually
  cheaper than keeping content hot once cache effects
  (hao2026selfgc's CommitBenefit) are included?
- Does the invariant hold under adversarial content? An offloaded
  span recalled later re-enters the window without whatever screening
  applied at first ingestion ([[concepts/verified-memory-writes]]).
