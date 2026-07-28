---
kind: paper
title: "Portable Agent Memory: A Protocol for Provenance-Verified Memory Transfer Across Heterogeneous LLM Agents"
authors: ["Santhosh Kumar Ravindran"]
institutions: ["Microsoft Corporation"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.11032"
code_url: null
citations: null
source: "raw/papers/ravindran2026portable.pdf"
added: "2026-07-28"
relevance: 3
credibility: 2
status: read
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/multi-granularity-memory]]"
related_experiments: []
tags: [memory, portability, provenance, merkle-dag, capability-tokens, interoperability]
---

# Portable Agent Memory: A Protocol for Provenance-Verified Memory Transfer Across Heterogeneous LLM Agents

## TL;DR

An open protocol for serializing agent memory into a signed, portable
artifact that a *different* vendor's agent can re-hydrate: a five-part
memory model (episodic, semantic, procedural, working, identity), a
Merkle-DAG for tamper-evident provenance, capability-scoped access tokens,
and an injection-resistant re-hydration pipeline. The design is the
contribution; the evidence behind it is a self-described pilot (N = 50
tasks) and should be read as directional.

## Claims

- Agent memory is imprisoned in vendor runtimes, producing lock-in, session
  amnesia, and no way to verify integrity when memory does move.
- Memory should be an **artifact with an owner**, not engine-local state:
  the human operator holds the signing keys and issues capabilities, not
  the model provider.
- Structured, verified transfer preserves substantially more capability
  across a model boundary than no-memory continuation (TCS 0.83–0.92 vs
  0.28–0.45).
- Re-hydration is an attack surface in its own right — imported memory is
  untrusted input, and framing/content-type enforcement should treat it
  that way.

## Methods

- Memory model M = (E, S, P, W, I): episodic, semantic, procedural,
  working, identity.
- **Merkle-DAG provenance.** Entries hash into a DAG; any field change
  invalidates the root. Ed25519 artifact signing (auto-generated keypairs,
  operator-supplied keys for production, multi-key rotation).
- **Capability-scoped access tokens** for fine-grained authorization over
  which portions of an artifact a receiving agent may read.
- **Re-hydration pipeline** with relevance ranking over a weighted linear
  combination of recency, salience, semantic similarity, and provenance
  depth; extractive (not abstractive) summarization to avoid inference
  latency; content-type enforcement and framing as injection defense.
- Redaction that preserves DAG position with typed tokens, so erasure does
  not break provenance.
- `pam-sdk` Python reference implementation, six modules, 54 tests, minimal
  dependencies (blake3, pynacl).

## Results

- **Transfer continuity** across three model pairs (Claude-3.5-Sonnet →
  GPT-4-Turbo, GPT-4-Turbo → Gemini-1.5-Pro, Gemini-1.5-Pro →
  Claude-3.5-Sonnet), N = 50 tasks total: TCS 0.83–0.92, ~2.4× the
  no-memory baseline. Q&A recall transfers most cleanly (semantic memory);
  planning degrades most (working memory is model-specific) — a sensible
  and believable ordering.
- **Tamper detection**: 100% across 1,000 single-field mutation trials and
  500 parent-reference manipulations.
- **Injection resistance**: 200 attack patterns (role elevation,
  instruction override, delimiter breakout, encoded/obfuscated), zero
  reported executions.

## Critique / open questions

- **The security evaluation tests the wrong threat.** Tamper detection and
  injection-pattern blocking are both *transport-integrity* results: they
  show an artifact cannot be modified in flight and that known injection
  strings do not survive framing. Neither addresses whether an entry
  *should have been written in the first place*. A memory poisoned by an
  honest-but-wrong acquisition at the source transfers with a perfect
  signature and an intact DAG.
- This matters because [[literature/papers/louck2026securing]] proves
  (TLA+-checked) that gates keyed on content or on **derivation edges** are
  launderable, and a Merkle DAG is precisely a derivation-edge structure.
  Portable Agent Memory's cryptography establishes *that the lineage was
  not altered*, not *that the origin was legitimate* — which is the
  property [[concepts/verified-memory-writes]] identifies as the only one
  that holds. The Ed25519 artifact signature is closer to origin binding
  but binds the **export event**, not each write to its channel.
  Likewise the framing/content-type injection defense is content-based, the
  class [[literature/papers/sharma2026smsr]] independently shows cannot be
  certified.
- The 200-pattern injection battery is non-adaptive and self-authored; no
  adaptive adversary, no held-out attack set.
- Evaluation uses Claude-3.5-Sonnet, GPT-4-Turbo, and Gemini-1.5-Pro —
  2024-era models in a May 2026 paper. Whether the transfer gap persists
  with current models is untested, and it plausibly shrinks as context
  windows and native memory features grow.
- "We release a complete Python SDK" but no repository or package URL
  appears anywhere in the paper — hence `code_url: null`.
- Author's own limitations are candid: N = 50 is directional, relevance
  ranking is deliberately simple, summarization is extractive.

## Trust signals

- **Credibility:** 2 — single author; corporate affiliation (Microsoft) but
  no indication of a lab or team behind it; pilot-scale evaluation the
  author explicitly labels directional; security results self-tested
  against non-adaptive attacks; no released artifact link despite the
  release claim; dated evaluation models. The protocol design is coherent
  and well-specified — the weakness is entirely evidential.

## Follow-up

- **Relevance:** 3 — useful prior art that names a real design point
  ([[concepts/verified-memory-writes]] × [[concepts/permission-gate-as-architecture]]
  × [[concepts/file-as-bus]]: memory as a portable, signed, capability-gated
  artifact rather than engine-local state) without supplying evidence
  strong enough to shift any concept. Cite-worthy for the design;
  do not cite the numbers.
- The **ownership stance** is the most durable part and is orthogonal to
  the weak evaluation: operator-held signing keys mean memory portability
  is a property of who holds the key, not of what the vendor permits. That
  framing is worth keeping even if this specific protocol goes nowhere.
- Directly analogous to this project's own cross-project `@import`
  contract, which moves *concepts* between repos by absolute path with
  `used_by:` back-references. The comparison is instructive in both
  directions: our version has no integrity layer at all (a concept file is
  trusted because git history is trusted), and this paper's version has an
  integrity layer that verifies the wrong property. Neither binds a memory
  to a legitimate origin at write time.
- Watch for a second, better-evidenced portable-memory protocol before
  letting this influence `verified-memory-writes` beyond a design mention.
  Recorded here mainly so a future stronger paper on memory portability has
  an anchor to compare against.
