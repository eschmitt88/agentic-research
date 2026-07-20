---
kind: concept
name: "verified-memory-writes"
status: seedling
added: "2026-07-07"
sources:
  - "[[literature/papers/yang2026trustmem]]"
  - "[[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]]"
  - "[[literature/papers/chhikara2025mem0]]"
  - "[[literature/papers/karamchandani2026your]]"
  - "[[literature/papers/louck2026securing]]"
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
- Second attestation, from practice
  ([[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]]):
  after an automated synthesis pass wrote garbage to a production
  vault, the author made every scheduled write **reversible** — daily
  diffs plus a 24-hour review window before changes become permanent.
  That is write-time verification implemented as *delayed commit +
  human backstop* rather than a verifier model: same principle
  (trust enforced at consolidation, not retrieval), opposite
  mechanism cost.
- **A second paper-grade source, at production scale but with a
  weaker gate.** Mem0 ([[literature/papers/chhikara2025mem0]], 60.8k
  GitHub stars, deployed) makes every write pass through an explicit
  four-way decision — ADD / UPDATE / DELETE / NOOP — chosen by an LLM
  tool-call conditioned on the top-*s* most similar existing memories.
  This confirms the *shape* of TrustMem's claim (writes should be
  gated, not appended unconditionally) at real deployment scale, but
  the gate itself is a single undifferentiated LLM judgment call, not
  a dedicated verifier scored against explicit coverage /
  preservation / faithfulness axes — i.e. evidence that *some* write
  gate helps, not evidence for TrustMem's specific verification
  design. TrustMem remains the only source with a *decomposed,
  measured* verification rubric (40–79% error reduction attributed to
  the coverage/preservation/faithfulness split specifically); Mem0
  and the practice report are corroborating but coarser. The concept
  stays `seedling` — still no independent replication of TrustMem's
  own ablation, and no source yet compares a coarse single-judgment
  gate against a decomposed one head-to-head. Adjacent evidence to
  watch: memory admission control and provenance-aware memory (cited
  in TrustMem's related work but not yet in this graph).
- **Fourth attestation — the adversarial case for the same boundary**
  ([[literature/papers/karamchandani2026your]]). FARMA forges an
  agent's *own reasoning traces* ("prior validation already complete")
  and amplifies them into false precedent, reaching 100% attack
  success against undefended agents and defeating retrieval-time and
  consensus defenses — amplification floods the store until forged
  traces *are* the consensus, so outlier detection inverts. The
  SENTINEL defense's first design principle is this concept stated as
  a security requirement: screen entries **before they are committed
  to memory**, "prevent[ing] forged entries from accumulating in the
  store and being legitimized through retrieval." Its write-path
  ablation (Reasoning Guard necessary and sufficient; 0% ASR, 0% FPR
  on 326 benign traces) is the strongest quantitative evidence yet
  that the write gate, not retrieval filtering, is where trust must
  be enforced. Prior sources motivated the gate from benign drift;
  this one shows the same boundary is the security perimeter — but
  note its gate is *structural* (provenance/format forensics on the
  entry), orthogonal to TrustMem's *semantic* triad
  (coverage/preservation/faithfulness). A production write gate
  plausibly needs both, and the paper's own adaptive-paraphrase
  result (the heuristic guard falls to an attacker who knows its
  patterns) marks the structural half as the brittle one.
- **Fifth attestation — the formal boundary on what the gate may check**
  ([[literature/papers/louck2026securing]]). A machine-checked (TLA+)
  separation theorem: any write/act authority signal derived from
  *content* or a *derivation edge* is **malleable** — laundered by the
  agent's own summarization, a trusted-tool echo, or manufactured
  corroboration — so content-scoring gates (SENTINEL's forensics, and
  in principle any LLM-judge trust score) cannot be sound against an
  adaptive attacker; FARMA's adaptive-paraphrase win over SENTINEL is
  this theorem in the wild. What holds is authority bound
  non-malleably to *origin at write time* (channel-authenticated, never
  inferred from content), elevated only via ≥2 independent trusted
  principals: TMA-NM reaches 0% attack success across 8 models at
  utility identical to undefended, where content and lineage baselines
  leak 47–84%. Sharpens this concept's internal division of labor:
  origin-binding is the *security* layer of the write gate, and
  TrustMem's semantic coverage/preservation/faithfulness triad is the
  *quality* layer — the theorem says the quality layer can never be
  asked to carry the security load. First formally-verified source in
  the cluster; artifacts released (benchmark, harness, TLA+ models).
