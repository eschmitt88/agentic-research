---
kind: paper
title: "ClaimReceipt: Verifying Evidence Sufficiency and Coverage in Agent Evaluations"
authors: ["Peiying Zhu", "Sidi Chang"]
institutions: ["Blossom AI", "Blossom AI Labs"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.01992"
code_url: null
citations: null
source: "raw/papers/zhu2026claimreceipt.pdf"
added: "2026-09-08"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
tags: ["evaluation", "provenance", "audit", "evidence", "verification", "preregistration"]
---

# ClaimReceipt: Verifying Evidence Sufficiency and Coverage in Agent Evaluations

## TL;DR

Splits agent-evaluation auditing into **two** questions that generic logs
conflate: *sufficiency* (is this claim recomputable from what was retained?)
and *coverage* (do the retained records span the experiment set that was
committed to?). Returns **PASS / INVALID / INCONCLUSIVE** per claim, at
0.021% of inference time.

## Claims

- Sufficiency and coverage are **independent** failures. Hash-linked
  transcripts answer neither: they prove records weren't altered, not that
  the records present are enough, nor that none are missing.
- Omissions only become visible against a **committed universe** — you must
  commit the experiment set *before* inference for coverage to mean
  anything.
- A three-way verdict is necessary; INCONCLUSIVE is a real outcome, distinct
  from failure.

## Methods

- **Specification frozen before implementation**, published as a SHA-256
  digest (`18d109...b81`) — a genuine preregistration of the artifact.
- Retrospective CR-2 verifier over 1,392 historical buyer–seller records.
- Separate **prospective** CR-3 epoch: 30 assignments committed before
  inference, terminal receipts signed and chained, private evidence
  encrypted for an auditor.

## Results

- CR-2 reproduces all five manually labeled audit verdicts; exactly replays
  600 deterministic and 792 post-generation records.
- All 13 declared field groups **non-redundant** under ablation — nothing in
  the receipt spec is decorative.
- **11/11 semantic faults caught, 0/8 false positives.**
- CR-3: withholding one terminal receipt returns `INCONCLUSIVE_COVERAGE`;
  withholding all private openings preserves coverage and protocol
  verification but makes *economic* claims inconclusive — **matching a
  preregistered prediction**.
- Overhead: **0.021% of model-inference time, 9.9 KB per transaction.**
- Reported negative: a specification-legibility probe indicates "our own
  frozen specification is not yet unambiguous to an independent reader."

## Critique / open questions

- The evaluation domain is a **buyer–seller commerce simulation**, not a
  research-agent evaluation. The machinery is domain-general but the
  demonstration is not, so transfer to scientific claims is argued rather
  than shown.
- Two authors at a small startup, no released code — for a paper whose whole
  thesis is auditability, the artifact's own absence is an awkward gap. The
  frozen-spec hash partly compensates.
- Preregistered predictions and a self-reported legibility failure are
  strong method signals from a weak-credential source.

## Trust signals

- **Credibility:** 2 — arXiv preprint from a small unknown company
  (Blossom AI), not peer reviewed, no code released, no citations. Sits at
  the top of that band on method: a specification frozen behind a published
  hash before implementation, preregistered predictions that were met, and
  an honestly reported negative result about their own spec's legibility.

## Follow-up

- **Relevance:** 4 — PASS/INVALID/INCONCLUSIVE is
  [[concepts/typed-claim-partition]] applied to evaluation, and the third
  verdict is the load-bearing one: an evaluator that can only pass or fail
  must resolve missing evidence as one of the two, which is precisely how
  an unmet requirement gets scored as met.
- **Coverage-against-a-committed-set is a gap
  [[concepts/citation-anchoring]] does not currently name.** That concept
  checks each claim against a source; it has no notion of whether the set of
  claims is complete. This project's `/lint` has the same hole — it finds
  dead wikilinks and sourceless concepts, but cannot detect a *missing*
  concept the evidence should have produced.
- The commit-before-inference discipline is the mechanism
  [[concepts/evidence-gated-completion]] needs to become checkable rather
  than aspirational.
- Cheap enough to be non-optional: 0.021% overhead undercuts the usual
  argument that provenance instrumentation is too expensive to always run.
