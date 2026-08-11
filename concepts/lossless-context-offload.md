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
