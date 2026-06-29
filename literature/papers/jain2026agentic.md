---
kind: paper
title: "Agentic AutoResearch for Space Autonomy: An Auditable, LLM-Driven Research Agent for Aerospace Control Problems"
authors:
  - Amit Jain
  - Richard Linares
institutions: ["Massachusetts Institute of Technology"]   # Dept. of Aeronautics and Astronautics
year: 2026
venue: "arXiv:2606.20394 [cs.RO]"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.20394"
code_url: null   # no repo/artifact link surfaced in the first pages
citations: null
source: "raw/papers/jain2026agentic.pdf"
added: "2026-06-29"
relevance: 3
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
tags:
  - autonomous-research-agent
  - evaluation-integrity
  - seed-variance
  - aerospace-control
  - in-loop-audit
  - credibility-gate
---

# Agentic AutoResearch for Space Autonomy

## TL;DR

**AutoResearch**: an LLM drives the full ML-research loop for spacecraft
control (proposes a config edit, runs the experiment, logs the outcome),
but the LLM never operates the vehicle — it produces an offline control
policy deployed onboard. Its contribution is a **credibility layer built
into the loop** that certifies each reported result against the problem's
own measured seed noise, so an autonomous agent that experiments fast
cannot over-claim. A 4th-domain (aerospace control) attestation that
high-variance research agents must be scored over many seeds and gated
before a result is believed.

## Claims

- An agent that "experiments faster only compounds the risk of mistaking
  noise for progress" — speed without statistical discipline is exactly
  what the reproducibility literature warns against.
- Research-loop rigor can be enforced **statistically and in-loop** (this
  work) as a complement to procedural rigor (Curie): denominate each
  reported number in measured seed noise and gate on it.
- The autonomous, auditable research *process itself* — producing a policy
  and certifying its reported gains — is the contribution, not any single
  control result.

## Methods

- **`family` contract**: a plain-language problem description, one editable
  training script with a delimited hyperparameter region, a single
  structured metric, and an append-only run log. One LLM-driven agent loop
  applies *unchanged* across problems whose physics differ sharply.
- **Credibility layer (the acceptance gate)** — three checks, applied
  uniformly, before any result is credited: (a) measured **per-problem
  seed noise**; (b) **reseeded verification** of the best configuration;
  (c) **leave-one-out pruning** of the agent's own edits (which edits
  actually carry the result). The loop is free to search optimistically
  because "nothing it reports is believed until the credibility layer has
  audited it."
- Baseline: a parallel **undirected random search** over the same editable
  surface, under a matched per-iteration budget, anchored to a shared
  first iteration so both arms start from a common configuration.
- Demonstrated on two aerospace control problems calibrated against a known
  optimal-control benchmark: Clohessy–Wiltshire relative rendezvous, and a
  safety-constrained collision-avoidance docking past a keep-out zone.

## Results

- The audited policy clears the measured seed noise by **many standard
  deviations**; the undirected search over the same parameters does not.
- On the docking problem the gap is **categorical**: undirected search
  yields no feasible policy, while the learned policy holds the hard safety
  constraint and stays outside the keep-out zone **on every seed**.
- Leave-one-out pruning identifies which of the agent's edits actually
  carry each result (and which are inert).

## Critique / open questions

- Narrow demonstration: two control problems, a single agent loop; the
  generality claim rests on "same loop, unchanged" rather than a broad
  benchmark sweep.
- No released code/artifacts surfaced — credibility leans on the MIT
  AeroAstro affiliation and clear methodology rather than reproducibility.
- The credibility gate certifies *empirical results* (seed-noise gating),
  not *claims-against-sources* — so it is an evaluation-integrity / variance
  mechanism (`pass-at-k`, `hce-evaluation`) more than a citation-grounding
  one. The "gate as a first-class in-loop layer" framing rhymes with
  `permission-gate-as-architecture` but governs result acceptance, not tool
  authorization.

## Trust signals

- **Credibility:** 3 — MIT Aeronautics & Astronautics (Linares, Assoc.
  Prof.; Jain, postdoc), clear and well-motivated methodology grounded in
  the RL-reproducibility literature (Henderson, Agarwal, NeurIPS repro
  program). But an arXiv preprint (not peer-reviewed), short (6 pp.), no
  released code/artifacts, no citations yet. Reputable group offset by
  thin reproducibility signal.

## Follow-up

- **Relevance:** 3 — a cross-domain (aerospace control) attestation that
  strengthens the evaluation-integrity cluster: it operationalizes
  `pass-at-k` variance-discipline as an **in-loop acceptance gate** (seed
  noise + reseeded verification) and embodies `hce-evaluation`'s
  honest-signal principle (the search loop may be optimistic; nothing is
  believed until audited). The optimal-control benchmark acts as a
  `programmable-evaluator-oracle`. Useful as a concrete "in-loop result
  certification" pattern; one more attestation rather than a new concept.
- Pairs with Curie (procedural rigor) and MLE-bench (high-variance →
  score over many seeds) already in the graph under `pass-at-k` /
  `hce-evaluation`.
