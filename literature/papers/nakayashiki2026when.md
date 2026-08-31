---
kind: paper
title: "When Stale Constraints Go Unchecked: Budgeted Verification Failures in Inherited Agent Memory"
authors: ["Kazuki Nakayashiki"]
institutions: ["Glasp"]
year: 2026
venue: "arXiv (cs.IR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.25553"
code_url: "https://doi.org/10.5281/zenodo.22084498"
citations: null
source: "raw/papers/nakayashiki2026when.pdf"
added: "2026-08-31"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: ["memory", "provenance", "context-window", "evaluation", "agent-architecture"]
---

# When Stale Constraints Go Unchecked: Budgeted Verification Failures in Inherited Agent Memory

## TL;DR

Provenance links keep the evidence behind an inherited belief *reachable* —
but an agent with a verification budget still has to choose which links to
spend it on, and it reliably chooses wrong. Sixteen models inspected the
provenance path of a superseded constraint in about one episode in five, and
produced stale-consistent decisions ~75% of the time. Re-assigning **one of
the same two budget slots** to the critical path recovers most of that risk —
no extra budget required.

## Claims

- The failure is not missing provenance. Provenance is immutable and
  reachable; the failure is **allocation of a finite verification budget**.
- A constraint that "reads as settled" is precisely the one that does not get
  re-verified — staleness is invisible at exactly the moment it matters.
- The recoverable risk is large and the recovery is **budget-neutral**:
  reallocation, not more inspection.
- A **target-blind, one-sentence rule** — *prefer memories that state a limit
  on a candidate direction* — is enough to redirect the agent's own
  allocation. A content-free freshness cue is not.

## Methods

- Controlled six-memory scenario with a verification budget of two records.
  A consolidated memory states a decision constraint; its source record has
  since been superseded by one that withdraws it.
- Sixteen language models in the primary run, plus a replication, a held-out
  domain, a **prospectively frozen interleaved replication** with a repaired
  non-critical control, and a further panel of **10 models from 9
  organisations** new to the study.
- Oracle contrast ("forced-critical" policy) uses experimenter knowledge of
  the critical path — the author is explicit that this **quantifies
  recoverable risk and is not a scheduler**.
- Two further deposited experiments locate the failure (allocation) and test
  the target-blind remedy.

## Results

- Provenance path inspected in ~1 episode in 5 (17.0% at two slots; 88.7% at
  four of six — above uniform allocation).
- Stale-consistent decisions in **77.3% / 74.7% / 74.7%** of episodes across
  primary run, replication, and held-out domain.
- Forced-critical reallocation: **+74.0, +72.7, +61.3** points, positive in
  every model; **+80.7** in the frozen interleaved replication; **+62.0** on
  the fresh 10-model panel; **+73.3** on a corrected re-run of the held-out
  scenario.
- The one-sentence target-blind rule recovered the oracle contrast on
  decisions (**+89.3** points) where the constraint limits the tempting
  action. A content-free freshness cue moved neither selection nor decisions.

## Critique / open questions

- Single author at a small company (Glasp), not a research lab, unreviewed.
  Offset substantially by the artifact deposit and the frozen replication.
- The scenario is small and synthetic (six memories, two slots). Whether the
  17%-selection number survives at realistic memory-store sizes is untested —
  though the *direction* is replicated six ways.
- The target-blind rule is stated to work "where that constraint limits the
  tempting action". That conditional may be carrying more weight than the
  +89.3 headline suggests.
- The paper discloses a temporal inconsistency in one frozen scenario's text
  and re-runs it. That is good practice, and it is also a reminder that the
  frozen materials were not perfect.

## Trust signals

- **Credibility:** 3 — not a reputable research group and not peer reviewed,
  which would ordinarily cap this at 2. Raised to 3 by genuinely strong
  reproducibility signals: Zenodo-deposited artifacts, a *prospectively
  frozen* replication, a held-out domain, a fresh 10-model / 9-organisation
  panel, and self-disclosed correction of a flawed scenario.

## Follow-up

- **Relevance:** 5 — canonical evidence for [[concepts/constraint-pinning]],
  which the 2026-08-30 `/elevate` log records as stalled at two attestations
  since 08-09. This is a third, and it is the strongest: it isolates *why*
  pinning is needed (allocation, not reachability) and gives a budget-neutral
  remedy.
- Sharpens [[concepts/verified-memory-writes]] by moving the question past
  write-time admission: even correctly attested, immutably-linked memory goes
  stale, and read-time verification is where the budget gets misspent.
- Direct [[concepts/budget-as-ceiling]] result and an unusual one — the useful
  lever is *reallocation under a fixed ceiling*, not a higher ceiling. This is
  the opposite of the instinct in
  [[literature/papers/leong2026recognition]]-style "add a layer" defenses, and
  the same shape as the negative result in the 2026-08-31 digest's item 19.
- Directly actionable for this project: inherited `NOTES.md` "Next" items and
  MoC framing are exactly "consolidated memories stating constraints whose
  source records may have been superseded". The target-blind rule is cheap
  enough to test.
- Read with [[literature/papers/guo2026when]]'s No-History-Promotion — both
  say that inheritance without re-verification silently confers standing.
