---
kind: concept
name: "verified-memory-writes"
status: seedling
added: "2026-07-07"
sources:
  - "[[literature/papers/yang2026trustmem]]"
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
related_experiments: []
tags: [memory, consolidation, write-policy, verification, trustworthiness, knowledge-organization]
---

# verified-memory-writes

## Definition

Every write to an agent's persistent memory — insert, revise, prune — is
verified as a *state transition* before it persists, judged on whether it
covers the new evidence (**coverage**), keeps valid prior memory intact
(**preservation**), and adds nothing unsupported (**faithfulness**).
Trust is enforced at consolidation time, not left to retrieval-time
filtering or terminal task feedback.

## Why it matters here

Once a flawed entry is consolidated, it is repeatedly retrieved and
amplified — persistent memory errors compound across sessions, and
terminal rewards can't localize which write introduced them (a severe
credit-assignment gap). [[literature/papers/yang2026trustmem]] shows that
scoring each transition with a verifier (and training the write policy on
those scores) cuts omission/corruption/hallucination in consolidated
memory by 40–79% while *improving* downstream utility — write-time
verification is not a tax on memory quality but a contributor to it.

The memory cluster previously covered trust only on the read side:
[[concepts/selective-memory-retrieval]] gates what gets *retrieved*;
[[concepts/multi-granularity-memory]] consolidates across tiers but
without an explicit trust gate on the consolidation step. This concept is
the missing write-side policy: what may enter the durable store, and on
what evidence.

The pattern generalizes beyond conversational memory:

- **Skill promotion** — [[concepts/skill-library-lifecycle]]'s core
  question (what gets promoted into the durable skill library, on what
  evidence) is a memory-write decision; the coverage / preservation /
  faithfulness triad is a directly reusable promotion rubric.
- **Knowledge-graph ingest** — this project's own `/ingest` discipline
  (trust-signal frontmatter, dedup against existing notes, immutable
  `raw/`) is a hand-rolled instance: verification happens at write time,
  and `raw/` immutability guarantees the evidence a write was judged
  against stays auditable.

## Connections

- Complements [[concepts/selective-memory-retrieval]]: retrieval-side
  gating catches bad memories late; write-side verification prevents them
  from persisting at all. A robust memory system likely needs both.
- [[concepts/agent-native-memory]] argues memory systems should be
  designed for agents' actual read/write workloads — this concept is the
  write-workload half made concrete.
- Single-source seedling: TrustMem is the only attestation so far.
  Adjacent evidence to watch: memory admission control and
  provenance-aware memory (cited in TrustMem's related work but not yet
  in this graph).
