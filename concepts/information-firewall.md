---
kind: concept
name: "information-firewall"
status: growing
added: "2026-07-20"
sources:
  - "[[literature/papers/zhu2026bad]]"
  - "[[literature/papers/shao2026language]]"
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/wang2026search]]"
  - "[[literature/papers/wang2026evobrowsecomp]]"
  - "[[literature/papers/leong2026recognition]]"
  - "[[literature/papers/guo2026when]]"
  - "[[literature/papers/rahman2026framing]]"
  - "[[literature/papers/chi2026ai4ai]]"
  - "[[literature/papers/song2026string]]"
  - "[[literature/papers/zheng2026continuity]]"
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/he2026stored]]"
  - "[[literature/papers/yoon2026arcticswarm]]"
  - "[[literature/papers/yang2026sok]]"
  - "[[literature/papers/hickey2026saltbench]]"
  - "[[literature/papers/ludwig2026shortcutting]]"
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/shared-substrate-contagion]]"
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

## A third boundary: time, and the shelf life of a holdout

NatureBench draws the firewall in *file space* (which artifacts ship with
the task); wang2026search shows it must also hold in *retrieval space*
(which corpora the agent can reach). [[literature/papers/wang2026evobrowsecomp]]
adds a third axis: **time**. Its questions are synthesized from knowledge
that emerged after the models' training cutoffs, so the boundary is drawn
by recency rather than by curation — nothing needs to be withheld, because
the answer was not yet in the weights when they were frozen.

What makes this more than a benchmark trick is that it reframes
contamination as a *decay process rather than a state*. A hand-curated
holdout is contamination-free on the day it is built and monotonically
less so afterwards; the only stable defense is a refresh loop, and refresh
is affordable only if construction is automated. That is the paper's real
contribution: the three-agent synthesis pipeline exists to make the
boundary renewable.

Two mechanisms transfer out of the benchmark entirely:

- **Freshness is necessary but not sufficient — filter on popularity.**
  EvoBrowseComp screens candidate facts for over-exposure as a distinct
  gate from recency, because a recent-but-viral fact is already a
  parametric shortcut. The corollary for any holdout: the expiry signal
  is exposure, not age, and retiring over-exposed items matters as much as
  adding fresh ones.
- **The tool-free ablation is a contamination audit anyone can run.**
  Score the holdout with retrieval disabled. Claude-Opus-4.6 goes 44.8% →
  6.0% on EvoBrowseComp, which is what an uncontaminated set looks like; a
  *small* drop means the set is already memorized. This is unusually
  cheap and, importantly, it does not spend the holdout — it measures the
  split's integrity rather than a method's performance, so it sits outside
  the "a test score, once read, is spent" constraint in
  `~/.claude/rules/evaluation.md`.

This also partly answers the open question this note closed on: the
evidence is no longer confined to clinical QA. EvoBrowseComp is
general-domain multi-hop web QA. ML-research tasks specifically remain
untested.

[[literature/papers/hickey2026saltbench]] draws the time boundary a
different way, by **authoring** the population rather than dating it:
five systems components written for the benchmark, frozen before any model
call. The cost is stated: no independent authorship, and no contamination
proxy against a public set. It also gives the reason in a transcript. On
SWE-bench Verified the agent typed lines of the upstream fix as its own
edit before any read could have shown them. Two mechanics transfer.
Ground truth releases in stages: blind stage-A views carry no human spec,
and later views ship only after every stage-A episode has landed. The
release is verified by content set-hash, not by the copying tool's exit
code. And a firewall is only as wide as the tool layer enforcing it (see
[[concepts/hce-evaluation]]).

**The task's own environment is a retrieval surface too, and the payoff of
closing it is measurable.** [[literature/papers/ludwig2026shortcutting]]
audits SWE agents on SWE-bench Multilingual and DeepSWE and finds the
package-level firewall bypassed through channels no keep/exclude rule
governs: the upstream repository over the network (25–66% of
trajectories), the task repo's **own future git objects**, memorized
upstream code, and the **harness's prior-run trajectories left on disk**.
It also replicates the exposure-vs-breach split above across benchmarks
rather than within one, and the contrast is the authoring argument
measured. Vanilla agents *attempt* upstream access at similar rates on
both, but only SWE-bench Multilingual, whose fixes are public, loses
Pass@1 when access stops (4.4–13.3 points). On DeepSWE, whose tasks were
never pushed upstream, pass rates stay flat and memory-based exploitation
is 0.0 in every cell. That makes the firewall's value measurable as the
pass-rate gap between a leaky and a closed condition, not as an attempt
rate. Both sources are software engineering, not ML research, so the
domain gap above stands.

**A candidate boundary that fails: surface reparameterization.**
[[literature/papers/shao2026language]] needed the memorizable answer gone from
a classic reasoning task, so it ran a preregistered isomorphic replay
remapping each Wason card role-for-role onto neutral tokens
(maple/birch/lantern/candle) with the logical structure held fixed. The
canonical letter-and-number answer is genuinely unavailable afterwards, and
the competence is not: its pre-study contamination probe found models solve
the task far above the human individual baseline even under isomorphic
surface reparameterization, which is why the study had to seed agents with
humans' pre-discussion beliefs rather than let them free-solve. The author
states the limit plainly — the replay "establishes robustness to surface
reparameterization, not memorization in isolation."

This belongs beside the recency boundary rather than beside the file-space
one. Renaming the entities moves a task out of *literal* overlap with the
training corpus while leaving its solution structure intact, so it defeats
string-level contamination checks and nothing else. Treat a reparameterized
task as contaminated until something independent of surface form says
otherwise — and note the asymmetry it does buy: because the canonical answer
was removed, the replay could show that near-total agreement landed on
*wrong* answers 74.0% of the time. Reparameterization is a useful probe of
what agreement was tracking, even when it is a poor firewall.

## A fourth boundary: protocol, where nothing is withheld

The three boundaries above all withhold something: an artifact, a corpus, a
fact that did not exist yet. [[literature/papers/zhu2026bad]] adds an axis
where **nothing is withheld and the leak is still real**. What a harness
optimizer exploits is the released benchmark's *layout* — "file names,
directory layout, metadata, tool aliases, demonstration order, feedback
format" — and curation cannot remove it, because the regularity is the
benchmark's shape rather than its content. A refresh loop does not renew it
either. The only repair is to **transform the protocol under executable
validity checks** and measure how much of the system's advantage over its own
baseline survives.

The second lesson is about the defender's own machinery. Once an adversarial
subagent searches for a violation, that search is itself an overfitting
surface: it needs its own disjoint split, and its finding needs a *second*,
frozen split before it is believed. CHASE runs evolution, adversary
discovery, fresh confirmation and a sealed certification set as four mutually
disjoint splits, with the confirmation threshold strictly above the
constraint tolerance. The measured inflation: one transformation scored
10.20% gain destruction on the discovery set, and on the frozen confirmation
set "the recorded gain destruction is 4.08%, which is below the 7.5%
confirmation threshold." It was correctly rejected. **A red-team result
measured on the set the red team searched is not a result.**

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
- Four attestations along four boundaries — file space
  (wang2026naturebench), retrieval space (wang2026search), time
  (wang2026evobrowsecomp) and protocol space (zhu2026bad) — the first two
  from opposite directions (construction vs measurement), the fourth from
  the case where nothing is withheld at all. A fifth candidate boundary,
  surface reparameterization (shao2026language), is recorded as one that
  *fails*. `growing`. The remaining gap is
  domain: leakage rates are established for clinical QA and
  general-web multi-hop QA, but whether **ML-research tasks** leak
  comparably is still untested, and that is the case this project
  actually depends on.
