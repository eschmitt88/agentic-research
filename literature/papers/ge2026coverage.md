---
kind: paper
title: "Coverage, Not Credit: Failure-Credit Routing of Zeroth-Order Perturbation Budgets Does Not Improve On-Pool Sample Efficiency for LLM Agents"
authors: ["Yuxu Ge"]
institutions: ["University of York"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.28011"
code_url: null
citations: null
source: "raw/papers/ge2026coverage.pdf"
added: "2026-09-01"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/hce-evaluation]]"
tags: ["negative-result", "compute-economy", "evolutionary-search", "methodology"]
---

# Coverage, Not Credit: Failure-Credit Routing of ZO Perturbation Budgets Does Not Improve On-Pool Sample Efficiency

## TL;DR

**A negative result, carefully done.** The natural hypothesis — that
trajectory-level credit assignment should decide where a zeroth-order search
spends its perturbation budget — does not hold. Across three task families,
six allocation schemes, and paired seeds, credit-based routing shows **no
detectable improvement over uniform allocation**, and misrouting has a real
cost. What matters is **coverage**, not credit.

## Claims

- Credit-based allocation of a fixed perturbation budget gives **no gain**
  (defined as ≥2pp) over uniform allocation in **any** on-pool comparison.
- **Misrouting has an asymmetric, real cost** — up to −0.074 AUC in-house
  and −0.118 end-to-end — so the failure mode is worse than "no benefit".
- The harm is **linear in the bottleneck's starvation rate** (R²=0.94,
  described by the author as descriptive), and a **preregistered credit-free
  coverage floor removes the detected harm**.
- The harm resides in **insufficient cumulative parameter movement** of the
  bottleneck rather than in update frequency — supported by a
  matched-budget burst schedule and a step-compensating catch-up arm.
- Three failure modes are documented that **silently invalidate** ZO/ES
  experiments on frozen LLMs.

## Methods

- A synthetic environment plus frozen Qwen2.5-1.5B/3B and SmolLM2-1.7B
  agents with modular low-dimensional subspace injection.
- Three task families, six allocation schemes, a credit-noise sweep,
  **paired seeds**, and **exact sign-flip tests**.
- **Equivalence testing**, not just null-hypothesis testing: the joint
  soft+σ routing scheme is bounded within a **±0.02 equivalence margin** in
  AUC on both 1.5B and 3B — i.e. an active demonstration of no effect
  rather than a failure to find one.
- Inverse-propensity debiasing tried as a rescue; it does not rescue routing.
- A **preregistered** credit-free coverage floor.
- Primary estimand declared as optimization efficiency on a fixed task pool.

## Results

- No statistically detectable improvement over uniform in any on-pool
  comparison.
- Concentrating the whole budget on the credit argmax is *marginally
  equivalent* on 1.5B (where that module is the verified bottleneck) and
  **significantly worse** on 3B.
- Misrouting cost: up to −0.074 AUC in-house, up to −0.118 end-to-end on
  the BFCL-derived family.
- Across six fixed-step schedules, loss is linear in starvation rate
  (R²=0.94).
- **One exception, reported rather than absorbed**: on held-out unseen BFCL
  functions, soft routing *exceeds* uniform (+0.047, p=0.031, n=6). The
  author offers a "plausible but untested" reading — that caller
  improvements favoured by routing are what transfer, while uniform's
  on-pool gains sat in a harness-specific synthesizer behaviour (relaying a
  constant gold answer).

## Critique / open questions

- Single author, no institution beyond a university affiliation and an
  ORCID, no released code, no peer review. On credibility signals this
  scores poorly — and on **methodology** it is better than most papers in
  this collection, which is exactly the tension the trust rubric is meant
  to hold.
- Small models (1.5B–3B) with subspace injection. Whether the null holds at
  frontier scale, or for non-ZO search, is untested and not claimed.
- The held-out exception is n=6 at p=0.031 — fragile, and the author says
  so, which is the right handling.
- The result is about ZO/ES parameter search specifically. Generalizing it
  to *any* budget-routing decision is the reader's inference, not the
  paper's claim.

## Trust signals

- **Credibility:** 3 — the score is a genuine composite tension. Against:
  single independent author, no code, no peer review, no citations, small
  models. For: preregistration, equivalence margins rather than bare null
  results, paired seeds, exact sign-flip tests, a credit-noise sweep, an
  attempted rescue (inverse-propensity) reported as failing, an
  inconvenient exception reported rather than buried, and a documented set
  of silent-invalidation failure modes. Per the ingest rubric,
  reproducibility and method are weighted at least as heavily as
  affiliation; the method here is strong enough to hold a 3 despite every
  institutional signal pointing at 1–2.

## Follow-up

- **Relevance:** 4 — this project has **very few negative results**, and
  the 08-30 `/elevate` log records the recurring problem that holds stack
  up because nothing ever argues *against* a mechanism. Both
  [[concepts/budget-as-ceiling]] and
  [[concepts/evolutionary-search-grain]] lean toward smarter allocation;
  this is a clean, preregistered negative on exactly that instinct, and it
  is load-bearing against over-engineering.
- **The operational rule is "guarantee coverage before optimizing
  allocation".** The credit-free coverage *floor* removes the harm; the
  clever routing adds nothing. Applied here: `/digest` and `/curate` should
  ensure every candidate gets *a* disposition before any effort goes into
  ranking which deserve the most attention — which is, not coincidentally,
  what the `/curate` skill already mandates ("the goal is that every item
  has a recorded disposition"). Independent support for an existing rule.
- Sharpens [[concepts/spend-forecast-calibration]]: if allocation cleverness
  buys nothing but starvation costs are linear and measurable, the useful
  forecast is *minimum per-unit coverage*, not predicted marginal value.
- **This note is itself the argument for a `diagnostic` lane.**
  [[literature/papers/li2026praxist]] (same curate pass) separates
  `confirmed` / `candidate` / `diagnostic` / `validation` evidence. This
  paper is a `diagnostic` artifact — a verified failure that should shape
  future exploration — and the current flat `sources:` array will file it
  next to sources that *support* the concepts it undercuts, with nothing
  recording the difference. Reading `budget-as-ceiling`'s source list, a
  future pass cannot tell that one of its sources argues against a core
  instinct.
- Read alongside [[literature/papers/chi2026ai4ai]] (same curate pass):
  that paper finds *more total effort* raises the class of change attempted
  (8%→64%); this one finds *smarter routing* of a fixed budget buys
  nothing. The pair points at total spend as the live variable and
  allocation cleverness as a distraction.
