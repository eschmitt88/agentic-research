---
kind: paper
title: "TruthInsightBench: An Evidence-Grounded Benchmark for Automated Evaluation of Open-Ended Scientific Discovery Agents"
authors: ["Zhibo Yang", "Chen Zhang", "Yuewei Zhang", "Hao Wang"]
institutions: ["TruthInsight-AI"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.05079"
code_url: "https://github.com/TruthInsight-stack/TruthInsightBench"
citations: null
source: "raw/papers/yang2026truthinsightbench.pdf"
added: "2026-09-08"
relevance: 5
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/typed-claim-partition]]"
tags: ["evaluation", "benchmark", "research-agents", "scientific-discovery", "oracle", "evidence"]
---

# TruthInsightBench: An Evidence-Grounded Benchmark for Automated Evaluation of Open-Ended Scientific Discovery Agents

## TL;DR

Existing AI-scientist benchmarks are **configured for reproduction** — they
build tasks around a hidden target study and reward recovering its result.
This one is configured for *discovery*: the agent gets a neutral objective
and frozen data, and the judge scores the **evidentiary maturity of the
agent's own claims** rather than agreement with a withheld answer. Four
coding agents plateau at 58.4–60.3/100 with no reliable separation.

## Claims

- Two abilities are routinely conflated: *executing a prescribed analysis*
  (an engineering capability) and *open-ended inquiry* — deciding which
  phenomena merit a claim at all. Benchmarks measure the first and are read
  as evidence for the second.
- Withholding the source conclusion is what makes the task a discovery task.
  Tasks expose only objective + frozen data; conclusions, expected values
  and analysis paths are all withheld.
- **The bottleneck is scientific judgment, not coding.** Agents "carry out
  analytical workflows whose conclusions they cannot adjudicate."

## Methods

- 40 blind tasks derived from 40 peer-reviewed studies across 10 domains.
- A **fixed** LLM judge scores six dimensions operationalised as **29
  artifact-grounded items**; aggregation is automated and deterministic,
  with *no per-instance human grading* — explicitly so the benchmark can be
  re-run automatically "as agents evolve in self-improving research loops."
- Four coding agents evaluated on one frozen base model, isolating the
  scaffold from the model.

## Results

- Total scores form a narrow plateau, **58.4–60.3 of 100**, with no
  statistically reliable pairwise separation between the four agents.
- Sub-scores split sharply: evidence auditability and novelty are
  comparatively strong; **controls, robustness, falsifiability and
  cross-dataset generalization are largely absent**.
- Genuine discovery is characterised as out of reach at this level.

## Critique / open questions

- **Authorship is effectively anonymous** — the only affiliation on the
  paper is "TruthInsight-AI" with a `163.com` correspondence address. The
  code is public, which is what carries the credibility here.
- A fixed LLM judge scoring "evidentiary maturity" is itself an oracle
  whose construct validity is unestablished; the 29 artifact-grounded items
  constrain it but the paper does not report judge-swap sensitivity. Compare
  [[literature/papers/brueckner2026kbench]], which runs three blinded judges
  and finds they disagree on the top of the table.
- A four-agent plateau with no reliable separation is equally consistent
  with "agents are all equally weak at discovery" and "the instrument does
  not resolve at this range."

## Trust signals

- **Credibility:** 2 — arXiv preprint, no institutional affiliation, no
  citations. Task data and scoring program are released, which is the
  strongest signal present and the reason this is not a 1. Re-score upward
  if the authorship resolves to a known group.

## Follow-up

- **Relevance:** 5 — the closest external analogue yet to this project's
  [[concepts/evidence-gated-completion]] + [[concepts/hce-evaluation]] pair:
  a held-out oracle that scores *claim warrant* rather than answer match.
  `evidence-gated-completion` had only five sources and none of them were
  a benchmark built on the principle.
- The reproduction/discovery distinction is the sharpest statement of the
  [[concepts/hce-evaluation]] thesis found so far, and it names *which*
  acts are missing: controls, robustness, falsifiability, cross-dataset
  generalization. That is a checklist a gate could enforce.
- Deterministic aggregation with no per-instance human grading is
  [[concepts/programmable-evaluator-oracle]] applied to open-ended science —
  the hardest case for that concept, since there is no reference answer.
- Pairs with [[literature/papers/brueckner2026kbench]]: both attack
  no-ground-truth scientific evaluation, and they disagree on whether a
  fixed single judge is sufficient.
