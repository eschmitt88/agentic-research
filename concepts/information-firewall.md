---
kind: concept
name: "information-firewall"
status: seedling
added: "2026-07-20"
sources:
  - "[[literature/papers/wang2026naturebench]]"
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
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

## Connections

- Complements [[concepts/hce-evaluation]]: HCE hides the *answers*
  (held-out labels) from the search loop; the firewall hides the
  *method*. Both are information boundaries drawn so a measured number
  means what it claims — one protects the estimate, the other the
  construct.
- [[concepts/programmable-evaluator-oracle]] holds that the evaluator
  defines what an agent *can* discover; the firewall defines what it
  *must* discover rather than recall.
- Single-source seedling: needs a second attestation (another
  method-withholding benchmark, or contamination-control work framing
  the same boundary) before promotion to growing.
