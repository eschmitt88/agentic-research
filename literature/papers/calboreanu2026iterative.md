---
kind: paper
title: "Iterative Audit Convergence in LLM-Managed Multi-Agent Systems: A Case Study in Prompt Engineering Quality Assurance"
authors:
  - Elias Calboreanu
institutions:
  - Swift (North) AI Lab, The Swift Group, LLC
  - Capitol Technology University
year: 2026
venue: MDPI Software (Special Issue on Software Reliability, Security and Quality Assurance)
peer_reviewed: true
url: https://arxiv.org/abs/2605.12280
code_url: null
citations: null
source: "raw/papers/calboreanu2026iterative.pdf"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hierarchical-delegation]]"
tags: [audit-loop, prompt-engineering, quality-assurance, multi-agent, convergence]
---

# Iterative Audit Convergence in LLM-Managed Multi-Agent Systems: A Case Study in Prompt Engineering Quality Assurance

## TL;DR

Empirical case study of nine sequential Claude-sub-agent audit rounds over AEGIS, a
production seven-lane orchestration pipeline with ~7150 lines of prompt
specifications; surfaced 51 consistency defects with non-monotonic convergence and a
seven-category defect taxonomy. (Skim from abstract + PDF intro.)

## Claims

- Prompt specifications carry data contracts and integration logic across many
  interdependent files but are rarely subjected to structured-inspection rigor —
  prompts are a typed artifact subject to QA.
- Per-round defect counts (15, 8, 12, 2, 8, 1, 4, 1, 0) show non-monotonic
  convergence consistent with cascading edits and audit-scope expansion.
- Single-file review missed defect classes surfaced only by later expanded-scope
  rounds — scope matters for what an audit loop can certify.

## Methods

- Checklist-driven walkthrough adapted from Weinberg & Freedman, executed by Claude
  sub-agents over seven lane PROMPT.md files plus a 245-line shared Ticket Contract.
- Companion preprint reports 51 STRIDE-categorized adversarial code findings,
  distinct from the prompt-spec findings here.

## Results

- 51 prompt-specification consistency defects across nine rounds, converging to 0.
- Seven-category post-hoc defect taxonomy with explicit coding rules; locked
  checklist released as a reproducibility appendix.

## Critique / open questions

- Self-audit caveat the author flags: the same LLM family authored and audited the
  specs; replication with dissimilar models and human reviewers is needed. This is
  precisely the `hce-evaluation` drift concern — can an autonomous loop certify its
  own output without an independent holdout?
- n=1 system, single author; generality unestablished.

## Trust signals

- **Credibility:** 3 — peer-reviewed (MDPI *Software* special issue), which lifts it
  above a bare preprint; but single author, single production system, and a
  self-audit design temper confidence. Institution is a minor prior.

## Follow-up

- Read the convergence section: is the non-monotonic pattern a stable audit-loop
  signature or an artifact of scope expansion? This is the read-side analogue of
  `hce-evaluation`'s drift question.
- "Prompt-specification surface as QA target" is a candidate extension of
  `typed-claim-partition` from outputs to prompts.
