---
kind: paper
title: "AI4AI-Bench: Benchmarking LLM Agents in Algorithmic Design for Recursive Self-Improvement"
authors: ["Yizhe Chi", "Wenyi Li", "Deyao Hong", "Xiaoqiu Wang", "Mingju Gao", "Kaisen Yang", "Bingxiang He", "Youjie Zheng", "Calvin Xiao", "Qinhuai Na"]
institutions: ["Navers Lab", "Einsia.AI", "Tsinghua University"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.20318"
code_url: "https://lab.einsia.ai/ai4ai"
citations: null
source: "raw/papers/chi2026ai4ai.pdf"
added: "2026-09-01"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["benchmark", "ml-research-agents", "self-improvement", "evaluation", "compute-economy"]
---

# AI4AI-Bench: Benchmarking LLM Agents in Algorithmic Design for Recursive Self-Improvement

## TL;DR

Recursive self-improvement turns on whether an agent can improve **the
training algorithm** — the thing that changes the compute-capability
exchange rate for every subsequent run. No prior benchmark isolates that:
existing suites are won by collecting data or tuning hyperparameters.
AI4AI-Bench freezes 10 repositories and asks for algorithmic change only.
The best of six systems reaches **0.250** on a scale where the shipped
algorithm is 0.1 and the optimum is 1.0.

## Claims

- RSI feasibility reduces to a measurable question — can an agent design
  training algorithms — and existing benchmarks **cannot tell a change to
  how a run is executed apart from a change to how the model learns**.
- Agents mostly **do not attempt** the thing being measured: most
  submissions never change how the model learns at all.
- **Reasoning effort buys willingness, not skill.** More effort moves
  agents toward algorithmic change; that shift is most of what effort buys.
- Incommensurable metrics across domains can be made comparable by anchoring
  each task's scale at the shipped algorithm and the task optimum.

## Methods

- **10 frozen research repositories** spanning 10 training-algorithm
  families. The repository, the starting model, and a cheap proxy metric
  are all frozen.
- Each task: the agent has **4 hours on one B300** to rewrite the training
  algorithm. Its code is then **rerun from scratch for up to 12 hours** and
  scored against the repository's original algorithm under the same
  procedure.
- **The evaluator is fixed before the first run and has no access to the
  agent's workspace** — the agent may evaluate itself as often as it likes,
  but the evaluation that decides its score is out of reach, "exactly as a
  held-out test set is out of reach".
- Normalization: every task maps onto one scale where **0 = uninformative
  model, 0.1 = the algorithm the repository ships, 1.0 = task optimum**.
- 29 configurations of 6 systems across all 10 tasks.
- **Released**: the task suite, the evaluators, and *every scored
  submission*.

## Results

- Mean score across 29 configurations: **0.166**. Best system: **0.250** —
  even the strongest closes under a fifth of the distance between the
  shipped algorithm and the optimum.
- **Most submissions never change how the model learns at all.** The
  minority that do average **0.226** against **0.126** for the rest.
- Raising reasoning effort takes that minority from **8% to 64%** of
  submissions, and the mean score from **0.094 to 0.196**.
- The authors note a scaling law in which every further halving of the loss
  gap costs several times more tokens than the halving before it.

## Critique / open questions

- 4 hours to write an algorithm and up to 12 hours to verify it is a hard
  budget; a low score may measure the budget as much as the capability. The
  paper's own framing (0.1 is what already ships) partly controls for this
  but does not separate "could not" from "did not have time to".
- Anchoring the shipped algorithm at 0.1 is a modelling choice that fixes
  the headline. A different anchor changes "under a fifth of the distance"
  into a different-sounding claim about the same data.
- "Best system reaches 0.250" is a max over 29 configurations, so it is
  subject to the usual selection optimism.
- Navers Lab / Einsia.AI is an unfamiliar group; the Tsinghua affiliation
  and the full artifact release are what carry the credibility.

## Trust signals

- **Credibility:** 4 — the artifact release is unusually complete (task
  suite, evaluators, **and every scored submission**, explicitly so the
  measurement can be repeated as systems change), the evaluator-isolation
  design is rigorous, 29 configurations × 6 systems × 10 tasks is real
  scale, and the central finding is a negative result about agents rather
  than a promotion of the authors' system. Tsinghua co-affiliation. Not
  peer reviewed; the lead organizations are not well known.

## Follow-up

- **Relevance:** 5 — **the evaluator design is the contribution to steal.**
  "An evaluator fixed before the first run, with no access to the agent's
  workspace; the agent may evaluate itself as often as it likes, but the
  evaluation that decides its score is out of reach" is the cleanest
  statement of [[concepts/information-firewall]] yet ingested, and it comes
  from the *measurement* side rather than the security side. Combined with
  [[literature/papers/rahman2026framing]] and
  [[literature/papers/esakkiraja2026starharness]] (whose validator forbids
  ground-truth-table and hidden-state access), the firewall is now attested
  by three independent constructions with three different motivations.
- Also a clean instance of [[concepts/programmable-evaluator-oracle]]: the
  oracle is fixed *before* the run, not co-evolved with it, which is the
  property that makes the score meaningful.
- **"Reasoning effort buys willingness, not skill" is the finding with the
  most direct bearing on this project's own operation.** `budget.yaml` sets
  `ideator` and `implementer` to Opus and treats spend as a capability
  dial. This says the first-order effect of more effort is that the agent
  *attempts the harder class of change at all* (8% → 64%) — which reframes
  [[concepts/budget-as-ceiling]]: the ceiling gates ambition before it
  gates quality. Compare [[literature/papers/ge2026coverage]] (same curate
  pass), which finds smarter *allocation* of a fixed budget buys nothing —
  together they suggest total effort matters and clever routing of it does
  not.
- Sits alongside [[literature/papers/moukpe2026deltaml]] as the second
  benchmark in this pass built on "improve a real frozen repository", and
  is the harder of the two: DeltaML-Bench allows any improvement,
  AI4AI-Bench scores only changes to *how the model learns*. The
  distinction is a sharper version of what
  [[concepts/evolutionary-search-grain]] calls the unit of mutation.
- Flagged as a recovered miss by the 08-31 digest note (the 08-24 window
  should have caught it). Ingested here to close that gap.
