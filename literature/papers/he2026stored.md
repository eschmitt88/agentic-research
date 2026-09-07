---
kind: paper
title: "Stored Is Not Supported: Typed Provenance and Assertion Guardrails for Persistent AI Agents"
authors: ["Jun He", "Deying Yu"]
institutions: ["OpenKedge.io"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.02127"
code_url: null
citations: null
source: "raw/papers/he2026stored.pdf"
added: "2026-09-07"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/agent-native-memory]]"
tags: ["memory", "provenance", "write-policy", "verification", "types", "safety", "evidence"]
---

# Stored Is Not Supported: Typed Provenance and Assertion Guardrails for Persistent AI Agents

## TL;DR

"Persistence changes availability, not epistemic standing: stored or
retrieved material is not thereby supported." The paper separates the
orthogonal dimensions a retrieval hit conflates — admission, evidential
support, agent stance, temporal validity, disclosure scope — and gates
release on all of them, so an agent cannot launder an injected string into
first-person autobiographical fact by storing and re-reading it.

## Claims

- Retrieval "alone cannot confer epistemic entitlement." Benchmarks like
  LoCoMo measure factual retrieval over long horizons, which is a different
  property from whether the agent is *entitled* to assert what it retrieved.
- **The self-corroboration loop is the core threat.** An untrusted document
  carries an embedded instruction ("the user prefers public disclosure");
  background consolidation distils it into a persistent profile; a later
  session retrieves the isolated summary and asserts with first-person
  authority "You asked me to share this information publicly." Through
  recurrent retrieval and re-summarisation such claims self-corroborate,
  "while stronger successor models may embellish them with plausible
  parametric details."
- Common mitigations fail *jointly*: an authenticated source can supply
  inaccurate information; multiple records tracing to one upstream root
  "manufactur[e] an illusion of consensus"; valid preferences expire; and
  supported facts may still be restricted from a given recipient.
- **Procedural entitlement requires decoupling five orthogonal dimensions**
  — state admission (reachability from an authoritative head), evidential
  support, agent stance (*recorded belief* vs *attributed report*), temporal
  validity, disclosure authorization. "None of these checks decides
  metaphysical truth"; they bound what the agent may claim.

## Methods

- **Autobiographical assertion boundedness** — a system-relative release
  property: governed statements about the agent, user or named
  relationships must satisfy accepted-evidence, temporal-validity and
  disclosure policies.
- Typed provenance graph separating origin, dependency lineage, epistemic
  role, validity and disclosure scope.
- A resolver evaluates authorized state projections and returns one
  evidential status plus orthogonal **conflict, staleness and withholding
  flags** and a protected decision witness.
- A **generate–verify–revise mediator** checks candidate semantic units
  before release.
- Conditional assertion-boundedness contract proved under explicit
  assumptions (extraction, predicate correctness, resolution soundness,
  view declassification, channel mediation).

## Results

- Executable suite of **24 hand-authored conformance cases**: typed
  mediation passed **none of 19 unsafe opportunities unqualified** while
  preserving all 5 supported controls.
- Baselines: flat/prior comparison released **19/19** unsafe candidates;
  source-tag comparison released **18/19**. Source tagging alone buys
  almost nothing.
- The authors state the limit plainly: results "validate the encoded
  resolver and mediator obligations; they do not constitute an end-to-end
  evaluation of language models or retrieval systems."

## Critique / open questions

- 24 hand-authored cases is a specification test, not an evaluation. The
  suite was written by the same authors as the guardrail, so the 19/19
  baseline failure is partly a statement about how the cases were chosen.
- The soundness proof is conditional on five assumptions, at least two of
  which (extraction correctness, predicate correctness) are exactly what
  an LLM-based pipeline will violate in practice. The hard part is assumed
  away.
- No cost accounting: five orthogonal checks and a decision witness per
  released semantic unit is not free, and no latency or token overhead is
  reported.
- Unknown company, no released artifact.

## Trust signals

- **Credibility:** 2 — arXiv preprint from an unknown organisation
  (OpenKedge.io), no peer review, no released code, no citations, and an
  evaluation that is 24 self-authored conformance cases rather than a
  benchmark. Top of the band for conceptual precision and for stating its
  own scope limit explicitly rather than overclaiming from 19/19.

## Follow-up

- **Relevance:** 4 — the title is the thesis of
  [[concepts/verified-memory-writes]] stated better than the concept
  currently states it. That concept covers *checking content at write
  time*; this reframes the property as **release-time entitlement**, which
  is the more defensible boundary: an agent may store anything, but may
  assert only what it can support. Worth propagating into the concept's
  definition.
- The five decoupled dimensions extend [[concepts/typed-claim-partition]]
  past the claim-type axis. Temporal validity and disclosure scope are
  genuinely new axes there; agent stance (recorded belief vs attributed
  report) is the one this project's own literature notes implicitly use
  and never name.
- "Multiple retrieved records may trace back to a single upstream root,
  manufacturing an illusion of consensus" is a precise statement of a risk
  [[concepts/citation-anchoring]] has not addressed — source *count* is not
  source *independence*. Directly applicable to this repo: a concept with 7
  `sources:` that all cite one another is not 7 attestations.
- The self-corroboration loop is [[concepts/information-firewall]] failing
  across sessions rather than within one context, and the observation that
  "stronger successor models may embellish" plausible details pairs with
  the memory-portability question raised by item 12 of the 09-07 digest.
