---
kind: paper
title: "EviGraph: Towards Verifiable Evidence Construction for Information-Seeking Agents"
authors: ["Jiashun Chen", "Yirong Mao", "Wenhui Que"]
institutions: ["Tencent (WeChat)"]
year: 2026
venue: "arXiv (cs.IR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.24667"
code_url: null
citations: null
source: "raw/papers/chen2026evigraph.pdf"
added: "2026-09-01"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hce-evaluation]]"
tags: ["retrieval", "evaluation", "provenance", "grounding", "knowledge-organization"]
---

# EviGraph: Towards Verifiable Evidence Construction for Information-Seeking Agents

## TL;DR

Agentic search optimizes for final-answer correctness, so the evidential
basis of the answer stays implicit in a growing trace. EviGraph makes
**evidence recording a separate, separately-rewarded stage** from search
execution: a frozen verifier returns verbatim spans with an explicit
support/conflict **polarity**, and a **deterministic structural validator**
decides what enters the graph. The graph is simultaneously working memory
and the reward signal.

## Claims

- **Retrieval relevance is not evidential support.** A page may mention the
  right entity while referring to a different time period, setting,
  geographic scope, or numerical value.
- Answer-level rewards actively encourage shortcuts; supervising the
  *intermediate grounding* requires a reward attached to evidence
  construction itself.
- Separating roles matters: the verifier **never edits the graph**, and the
  validator checks **only syntax and graph invariants** — no semantic
  judgement at the commit boundary.
- The same structure can serve as **both** persistent working memory and a
  source of dense process rewards, unlike prior evidence graphs used as
  passive state.

## Methods

- **Role split.** An *executor* plans concise queries. A *frozen evidence
  verifier* inspects source pages and returns verbatim evidence items with
  a relevance explanation and a polarity. An *evidence-proposal policy*
  maps each item to an `add` request (new candidate) or a `support` request
  (existing candidate). A **deterministic structural validator** checks the
  request against graph invariants and commits it.
- **Graph schema**: nodes partition into documents, verbatim spans, and
  candidate–claim nodes; an edge links a span to a claim with polarity ∈
  {support, conflict}. Scope and source context are edge fields.
- **Span anchoring** plus explicit constraint tags make graph quality
  measurable at each step, yielding dense per-step rewards.
- **Reward-hacking safeguards** (the paper's most careful part): coverage
  counts only positive-polarity evidence; unresolved support–conflict pairs
  are *penalized*; a potential-difference reward in the policy-invariant
  form γΦ(G′) − Φ(G); canonical evidence signatures; a per-candidate–tag–
  polarity acceptance-bonus cap; and a KL penalty. Together these target
  reward farming from duplicate or superficially varied records.
- RL over a shared policy for the trainable roles; the verifier is frozen.

## Results

- **BrowseComp-Plus**, Qwen3-8B under a matched interaction budget:
  **35.9%** for EviGraph vs **26.9%** for the same dual-role architecture
  without RL vs **2.7%** for a monolithic agent.
- Achieves this while generating **fewer tokens per rollout**.
- Consistent gains carry over to BrowseComp, GAIA, and XBench.
- The inference-time loop transfers without the training.

## Critique / open questions

- No released code, and the results are on a single small open model
  (Qwen3-8B). Whether the process rewards matter for a frontier model that
  already grounds well is untested.
- The 2.7% monolithic baseline is weak enough to flatter the architecture;
  the meaningful comparison is 26.9% → 35.9%, i.e. what RL on evidence
  construction adds to a dual-role setup that already exists.
- The reward-hacking safeguards are extensive, which is admirable but also
  an admission that the dense-reward design has a large attack surface.
  Whether the six mechanisms are each necessary is not ablated.
- The evidence verifier is frozen but is still an LLM: "verbatim span with
  a polarity" is a strong contract, yet nothing verifies the polarity
  label itself.

## Trust signals

- **Credibility:** 3 — Tencent WeChat is a serious industrial group and the
  RL formulation is unusually careful about reward hacking (policy-invariant
  potential shaping, acceptance caps, canonical signatures). Held down by
  no released code, a single 8B model, and no ablation over the safeguard
  set.

## Follow-up

- **Relevance:** 4 — [[concepts/citation-anchoring]] and
  [[concepts/evidence-gated-completion]] both currently assume evidence is
  a *byproduct* of search. This is the mechanism for treating it as a
  first-class stage, and it is what [[concepts/web-grounded-literature]]
  would need to become verifiable rather than merely sourced.
- **"Retrieval relevance is not evidential support" is the sentence to
  carry into `/digest` and `/discover`.** Both currently rank candidates by
  topical match and record a one-line reason for inclusion. The scope
  mismatch this paper names — right entity, wrong time period or numerical
  scope — is exactly the failure mode a WebSearch-driven digest is prone
  to, and this curate pass found an instance of it (the digest's
  compute-allocation summary of
  [[literature/papers/moukpe2026deltaml]] generalized a GPT-5-only result).
- The **support/conflict polarity edge** is a concrete extension to
  [[concepts/typed-claim-partition]]. This project's concept notes list
  `sources:` as a flat array — every source implicitly *supports* the
  concept. There is no way to record a source that **conflicts** with a
  concept, and `/lint` cannot surface a concept whose sources disagree.
  Penalizing *unresolved* support–conflict pairs is the corresponding
  hygiene rule.
- The **deterministic validator checking only syntax and invariants**, with
  semantic judgement pushed to a separate frozen role, is the same
  architectural split as `scripts/kg_lint.py` (deterministic checks) versus
  the interpreting agent — independent attestation that this project's
  ablation phase 3 got the boundary right.
