---
kind: concept
name: "information-firewall"
status: growing
added: "2026-07-20"
sources:
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/wang2026search]]"
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/web-grounded-literature]]"
related_experiments: []
tags: [evaluation, benchmark-design, discovery, reproduction, contamination]
---

# information-firewall

## Definition

Task construction that deliberately withholds the source method — its
identity, code, outputs, and every file specific to it — so an agent
evaluated on the task must *discover* a competitive solution rather
than *reproduce* the published one. The retained package contains only
what is needed "to define the task no matter which method is used";
scoring anchors to the withheld method's published result.

## Why it matters here

The discovery/reproduction distinction is the axis existing benchmark
families conflate: PaperBench-style replication hands the agent the
method and asks for a faithful re-implementation; MLE-bench-style
optimization asks for a good score but on engineering proxies with no
published scientific method to beat. NatureBench
([[literature/papers/wang2026naturebench]]) operationalizes the split
with a two-layer firewall — a file-level keep/exclude rule at dataset
acquisition (keep raw inputs preceding the algorithm and method-agnostic
preparation; exclude the algorithm's own preprocessing and outputs) and
a package-level rule that no file may reveal the source paper's identity
or method — plus web search disabled at eval time so the firewall
cannot be bypassed by retrieval.

The firewall is also what makes success-mode analysis *interpretable*:
because the source method is knowably absent from the agent's inputs,
matching SOTA via the same broad method family (37.7% of such runs) can
be read as rediscovery, and the dominant pathway — recasting scientific
tasks as supervised prediction (45.5% of successes) — as genuine
translation rather than leakage. Without the firewall, neither claim is
distinguishable from paraphrased reproduction.

## The firewall has to hold at retrieval time, or it does not hold

NatureBench simply *disables* web search at eval time so the firewall
cannot be bypassed. Search-time contamination
([[literature/papers/wang2026search]]) is the measurement of what
happens when that stipulation is dropped, and it supplies the second
attestation this concept was waiting for — from the opposite direction.
Where NatureBench builds the boundary, wang2026search measures it
leaking.

Three results sharpen the concept:

- **A firewall is a property of the agent's whole retrieval surface, not
  of the task package.** Curating the package is necessary and
  insufficient: with web search enabled, agents recover benchmark
  artifacts — and on MedMCQA, answers for nearly a quarter of
  questions — from question banks, forums, and data-hosting platforms
  that no package-level keep/exclude rule can reach.
- **Exposure is not the same as breach, and the distinction is
  measurable.** wang2026search's three-tier taxonomy (metadata →
  question context → explicit answer) shows only the answer tier
  reliably inflates scores (hazard ratios 2.20–8.92); merely retrieving
  benchmark-metadata URLs is associated with hazard ratios *below 1*.
  A firewall audit should therefore target answer-level leakage, not
  corpus overlap — the coarse proxy over-reports. But metadata leakage
  is still worth detecting as a *leading indicator*: it escalates into
  answer leakage over subsequent turns.
- **Restricting the corpus relocates the boundary rather than drawing
  it.** Valyu Deep Research leaks 0% on MedQA from its curated PubMed
  corpus and 78% on PubMedQA, which is *built from* PubMed. There is no
  globally safe source list; the firewall is only ever defined relative
  to a particular task's provenance.

The prescribed controls — isolated knowledge sandboxes so every agent
retrieves from one corpus, transparent search trajectories so an auditor
can see which retrieval produced an answer, and gated benchmark access
with anti-redistribution terms — are the operational form of the
firewall for search-enabled agents.

## Connections

- Complements [[concepts/hce-evaluation]]: HCE hides the *answers*
  (held-out labels) from the search loop; the firewall hides the
  *method*. Both are information boundaries drawn so a measured number
  means what it claims — one protects the estimate, the other the
  construct. wang2026search shows the two boundaries share a bypass:
  the agent's own search tool routes around both at once.
- [[concepts/programmable-evaluator-oracle]] holds that the evaluator
  defines what an agent *can* discover; the firewall defines what it
  *must* discover rather than recall.
- Tension with [[concepts/web-grounded-literature]]: continuous web
  intake is a capability this project deliberately builds, and the same
  channel is the contamination vector. The two are compatible only
  because literature curation has no score to inflate — a project that
  both web-searches *and* evaluates on a public benchmark has to choose,
  or sandbox the retrieval.
- Two attestations from opposite directions (construction, measurement).
  Promoted to `growing`. Next: evidence outside clinical QA —
  wang2026search's leakage rates come from an unusually redistributed
  domain, and whether ML-research tasks leak comparably is untested.
