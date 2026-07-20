---
kind: paper
title: "Securing LLM-Agent Long-Term Memory Against Poisoning: Non-Malleable, Origin-Bound Authority with Machine-Checked Guarantees"
authors: ["Yedidel Louck"]
institutions: ["Ariel University (Ariel Cyber Innovation Center)"]
year: 2026
venue: "arXiv preprint (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.24322"
code_url: "https://github.com/yedidel/mem-inv-bench"
citations: null
source: "raw/papers/louck2026securing.pdf"
added: "2026-07-20"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/agent-native-memory]]"
tags: [memory, write-policy, security, memory-poisoning, provenance, information-flow-control, formal-verification]
---

# Securing LLM-Agent Long-Term Memory Against Poisoning: Non-Malleable, Origin-Bound Authority with Machine-Checked Guarantees

## TL;DR

Proves (TLA+/TLC, machine-checked) that any memory defense whose
authority signal is derived from *content* or a *derivation edge* is
malleable — an attacker can launder untrusted origin through the agent's
own summarization, a trusted-tool echo, or manufactured corroboration —
and that binding authority non-malleably to *origin at write time*, with
elevation only via ≥2 independent trusted principals or a fresh
action-bound user authorization, is sufficient: TMA-NM hits 0% attack
success across 8 frontier models where the strongest baselines leak up
to 68–84%, at utility identical to the undefended agent.

## Claims

- Memory authority-to-act must be decided by a signal the adversary
  cannot transform. Content (detection/trust-scoring) and lineage
  (derivation edges) are both **malleable**: three LLM-specific
  laundering channels — self-summarization (L-a), trusted-tool echo
  (L-b), manufactured corroboration (L-c) — each erase or flip the
  signal while preserving behavior.
- Separation theorem (bounded model, TLC-checked over 3,270 reachable
  states, plus an inductive invariant toward unbounded executions):
  T1 — no content- or lineage-based defense is sound under laundering;
  T2 — write-time origin binding is necessary; T3 — non-malleable
  origin-bound authority with Sybil-resistant corroboration-gated
  elevation is sufficient.
- TMA-NM's four mechanisms: (M1) write-time origin binding sets
  `act_class = f(origin)`; (M2) non-malleable propagation — derived
  items inherit the *maximum untrust* of their inputs, closing L-a and
  L-b; (M3) elevation only via ≥2 *independent* trusted principals
  (distinct registries/tools, not repeated content) or a single-use
  action-bound user token, closing L-c and replay/TOCTOU; (M4) a
  tamper-evident hash-chained verdict log (auditability only — ablation
  shows it affects neither security nor utility).
- The act-gate is a deterministic check (~1.3 µs) — no extra model
  call — vs ~2,000 ms for an LLM content-judge, and introduces zero
  false blocks (authorizes the exact same legitimate actions as the
  undefended agent).

## Methods

- Formal model of the write → retrieve → act pipeline in TLA+; guarantees
  are conditional on an origin-labeling oracle (A1: origin comes from the
  authenticated *channel* — mTLS, audience-bound tokens, signed tool
  responses — never inferred from content).
- MEM-INV-Bench: 12 domains × 5 consequential tool types × attack
  scenarios (sleeper, control-flow, data-exfiltration; benign / static /
  adaptive / whitebox triggers), 5 defense classes (none, trust_score,
  prob_detect, lineage, capability_ifc, TMA-NM), 8 frontier models
  across 6 vendors, Wilson 95% CIs + permutation tests.
- External validity: reproduces 4 published poisoning pipelines
  (MemMorph, MemoryGraft, Trojan Hippo, conversational Trojan) as
  drop-ins (all blocked, 0/1152); multi-turn tool-using loop with
  value-level cross-turn origin propagation; end-to-end run over Mem0
  with a real vector store (origin labels survive consolidation).

## Results

- Theory↔benchmark correspondence: each defense class fails exactly on
  the channels the theorem predicts — trust_score laundered 68%,
  lineage resists corroboration but falls to summarization/echo (47%),
  capability_ifc admits the direct attack (84%). TMA-NM: 0% on every
  channel, model, and trigger style, at full legitimate utility.
- Content-detection threshold sweep: no τ separates laundered-malicious
  from legitimate (ASR ≤5% costs utility down to 33%); the whole
  frontier is dominated by TMA-NM's (0%, 100%) — the failure is
  structural, not calibration.
- Lineage default-policy sweep: neither default-allow nor default-deny
  reaches (0% ASR, 100% utility); binding to origin rather than the
  edge is what closes tool-echo.
- Corroboration threshold k generalizes (k∈{1..4}): ASR 0% for every
  k≥2; k is a security-vs-burden knob.

## Critique / open questions

- Everything is conditional on A1: a compromised trusted *channel* (not
  just content) defeats the scheme, and origin-labeling infrastructure
  (mTLS, signed tool responses) is assumed, not built — deployment cost
  lives exactly there.
- Single-author preprint; the machine-checked claim is a bounded model
  + inductive invariant, not a fully mechanized unbounded proof (stated
  honestly).
- Non-consequential answer-biasing is explicitly out of scope — poisoned
  memory can still bias *answers* with provenance surfaced; only
  consequential actions are protected.
- Design convergence worth noting: "≥2 independent trusted principals"
  is corroboration-as-quorum — structurally the same move as
  multi-attestation evidence bars elsewhere in this graph (e.g.
  /elevate's ≥3-attestation gate).

## Trust signals

- **Credibility:** 3 — single-author arXiv preprint from a real but
  non-frontier security center; substantially offset by full artifact
  release (benchmark, harness, TLA+ models) and an unusually careful
  eval (8 models, CIs, permutation tests, reproduced published attacks);
  no citations yet.

## Follow-up

- **Relevance:** 4 — strongest formal treatment in the graph of the
  memory write-gate: upgrades [[concepts/verified-memory-writes]] from
  empirically-motivated (TrustMem, FARMA/SENTINEL) to
  impossibility-theorem-backed ("content and lineage signals are
  malleable; bind authority to origin at write time"), and bridges to
  [[concepts/permission-gate-as-architecture]] — the act-gate here *is*
  a permission gate whose policy is data (origin labels + k-of-n
  corroboration), not prose.
- Complements karamchandani2026your precisely: FARMA shows the write
  path is the attack surface; this paper proves which write-path signal
  class can actually hold it, and that SENTINEL-style content scoring
  is in the provably-malleable class (its adaptive-paraphrase brittleness
  is T1 in the wild).
- Watch for: peer-reviewed version; whether MemLineage/CaMeL authors
  respond; value-level capability-token extension (Section IX) if the
  memory-security thread thickens toward its own concept.
