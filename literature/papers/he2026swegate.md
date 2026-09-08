---
kind: paper
title: "SWE-Gate: Passing Functional Tests Is Not Enough for Software Engineering Agents"
authors: ["Xin He", "Yanlin Wang", "Mingwei Liu", "Jiachi Chen", "Hongyu Zhang", "Guanbin Li"]
institutions: ["Sun Yat-sen University", "Zhejiang University", "Chongqing University"]
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.04167"
code_url: "https://github.com/DeepSoftwareAnalytics/SWE-Gate"
citations: null
source: "raw/papers/he2026swegate.pdf"
added: "2026-09-08"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: ["evaluation", "benchmark", "oracle", "acceptance-criteria", "code-agents"]
---

# SWE-Gate: Passing Functional Tests Is Not Enough for Software Engineering Agents

## TL;DR

Repository-level benchmarks score a patch by whether it passes functional
tests. Real acceptance also depends on **review constraints** derived from
PR review comments. Of 644 repairs that pass the functional tests,
**221 fail the review constraints** — functional-only evaluation
overestimates capability by roughly a third of its own successes.

## Claims

- The oracle is incomplete, not merely noisy: functional tests and review
  constraints measure **separable** things, and a benchmark that ships only
  the first reports success on work that would be rejected.
- Issue-resolution capability and constraint compliance should be scored
  separately rather than folded into one pass/fail.

## Methods

- Review constraints mined from **real pull-request review comments**, then
  repository-level repair instances synthesised around them.
- Each instance ships **separate functional and constraint tests**, plus
  both a **non-compliant patch and a gold patch** — so the two axes can be
  scored independently and the non-compliant patch validates that the
  constraint test actually discriminates.
- 303 repair instances across 75 open-source Python repositories.
- Four LLM backends of differing capability under a **common coding-agent
  scaffold**, isolating the model from the harness.

## Results

- **221 of 644 functionally-passing repairs fail review-constraint tests**
  (~34%).
- The gap persists across all four capability levels — this is not something
  a stronger backbone closes.

## Critique / open questions

- Review comments are a proxy for acceptance, not acceptance itself: they
  capture what a reviewer *said*, and mining them risks encoding reviewer
  idiosyncrasy as a requirement.
- Python-only, open-source-only; whether the ~34% rate transfers is untested.
- The paper stops at measurement — it does not test whether exposing
  constraint tests to the agent closes the gap, which is the obvious next
  experiment and the one that matters for harness design.
- Software-engineering agents are out of this project's primary scope; the
  paper is retained for what it says about **oracle design**, not about
  SWE-agent architecture.

## Trust signals

- **Credibility:** 3 — arXiv preprint, not peer reviewed, no citations yet,
  but from an established academic group (Sun Yat-sen / Zhejiang /
  Chongqing) with a **full replication package released** (code, data,
  results). Reproducibility carries this one.

## Follow-up

- **Relevance:** 4 — a **quantified instance of the exact failure
  [[concepts/hce-evaluation]] exists to prevent**: the oracle passes while
  the requirement is unmet. That concept has 43 sources but few of them put
  a number on the size of the gap; this one does, at ~34% of successes, on
  a common scaffold with the model held as a variable.
- Shipping a **non-compliant patch alongside the gold patch** is a design
  worth importing into [[concepts/programmable-evaluator-oracle]]: it makes
  the discriminating power of the test itself checkable, rather than
  assumed. `/lint` has no analogue — nothing verifies that its checks would
  actually fail on a bad input.
- Reinforces the [[literature/papers/brueckner2026kbench]] finding from a
  completely different domain and method: agents clear the presentation bar
  well before they clear the substantive one.
