---
kind: paper
title: "Revoked but Still Authoritative: An Empirical Study of Revocation Enforcement in Agent-Memory Systems"
authors: ["Yi Ting Shen", "Kentaroh Toyoda", "Alex Leung"]
institutions: ["Vulcan Research, AIFT (Singapore)"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.08258"
code_url: "https://github.com/VulcanLab/Memory-Rebirth-Attack"
citations: null
source: "raw/papers/shen2026revoked.pdf"
added: "2026-09-14"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
tags: ["memory", "revocation", "staleness", "retrieval", "provenance", "multi-agent", "safety", "negative-result"]
---

# Revoked but Still Authoritative: An Empirical Study of Revocation Enforcement in Agent-Memory Systems

## TL;DR

Soft revocation means a contradicted fact is marked invalid but kept. The
authors test whether that mark is enforced when the store is read, in five
agent-memory systems (Graphiti, Zep CE, mem0, langmem, cognee), and it
mostly is not. Where the revoked record is returned, it **ranks first in
every scenario** and leads agents to the unsafe action in **43.1% of trials
(699/1,620)**. A store-level validity filter takes that to **0/1,620**. The
filter's guarantee then breaks as soon as the agent **writes its own
decision back**: the journal is a current record the filter has no reason to
remove, and filtered unsafe rates climb to **71.6–83.1%** at later hops.
The missing control is *retrieval-time validity*, not provenance. The
harmful record is the defender's own authorized policy, so every provenance
check passes it by construction.

## Claims

- Stores fail in **three ways**: (1) the revocation is never recorded
  (cognee, langmem, and mem0 under indirect insertion); (2) it is recorded
  and returned, but the status is hidden from the caller, so no application
  can filter on it (Zep); (3) it is recorded and visible but not enforced at
  retrieval (Graphiti by default; mem0 with its expiry filter disabled).
- **The failure is in retrieval, not in any one product.** Two independently
  built exposed systems can't be told apart on matched cells (p = 0.629), and
  mem0 flips safe→unsafe in 39/81 cells when a **single retrieval boolean**
  changes, never the other way (p < 0.001).
- **Provenance is the wrong check for this threat.** SMSR-style write-time
  signatures ([[literature/papers/sharma2026smsr]]) and TMA-NM origin binding
  ([[literature/papers/louck2026securing]]) certify the record's origin, and
  here the origin is the defender. "Origin binding does not constrain what an
  authorized record does once its authorization is revoked."
- **Prompt hardening can't work in this setup.** The agent gets fact text
  only, with no validity metadata, so asking it to "disregard superseded
  facts" asks it to judge without the information it needs.
- **The harm goes beyond a single read.** It persists through agent
  write-back, crosses roles that share a store, and reaches tool actions,
  all without an attacker.
- **The containment boundary that holds is the read.** A guardrail at the
  tool layer is indistinguishable from no defense.

## Methods

- **Setup.** Each store gets a revoked policy r1 and its replacement r2.
  *Direct* insertion writes both in native format with the revocation set
  explicitly (e.g. mem0 `expiration_date=yesterday`). *Indirect* insertion
  ingests plain contradicting prose, and the system's own extraction has to
  notice the supersession. All five systems share one embedding model and
  one extraction model. k = 10.
- **Grid.** 9 authored scenarios: 4 with no prompt rule, 4 with an explicit
  prohibition in the system prompt plus 2 distractors, and 1
  "stored access directive" (an instruction rather than a policy). 9 decision
  models from 6 vendors in three tiers: GPT-5.5, Grok-4, DeepSeek-V4-Flash /
  GPT-5-nano, Gemini-2.5-Flash, Grok-4-Fast / GLM-5.2, Gemma-3-27B-IT,
  Kimi-K2.6. 10 trials per cell at T = 0.7. Each trial is a binary
  safe/unsafe action choice.
- **Six defense conditions.** No defense; store-level status filter (before
  the read); prompt hardening (after the read); output filter (after the
  read, and deliberately *given* the revoked texts); filter plus prompt;
  and the authors' guard.
- **Two metrics, decomposed.** Pr[unsafe] = Pr[exposure] ×
  Pr[unsafe | exposure]. Exposure depends only on the store; the conditional
  term depends on the model. Cross-system claims use matched-pair exact tests
  on discordant (scenario, model) cells.
- **Guard** (`guard/stale_guard.py`). It sits between the agent and any
  backend. Stage 1 withholds records whose exposed status says revoked.
  Stage 2 is a lexical conflict test: containment ≥ 0.45 AND an opposition
  signal (antonym pair, disjoint quantities, or a difference in negation
  count). It withholds the *older* record only when timestamps prove order,
  and caps withholding at β = 0.6 of the result. Withheld facts come back
  with a reason. Nothing is deleted.
- **Beyond a single read** (Appendix C, Graphiti only). Agent write-back
  over hops 1–3 in *natural* mode (the system's ingestion) and *direct* mode
  (an explicit record). Three-role propagation (executor / reviewer /
  planner) over one store. A four-step tool loop with decoy tools: file
  read, shell, egress, sanctioned API.
- **Cost.** 49,121 decision-model calls, **$79.08**, run on one MacBook Air.

## Results

- **Exposure.** Graphiti and mem0 (exp.) return r1 for 81/81 pairs, ranked
  **first**. cognee, langmem, and default mem0 return it 0/81. Under indirect
  insertion, cognee, langmem, and mem0 record the revocation in **0/9**
  scenarios while keeping both policies. Graphiti records it in 4/9 and Zep
  in 6/9.
- **Unsafe action with no defense.** Graphiti 44.2% (358/810), mem0 (exp.)
  42.1% (341/810). Clean-context baselines: 0/2,430.
- **Defenses** (pooled over both exposed systems). Store filter **0%**;
  prompt hardening 43.1% → **37.2%**; output filter **18.1%**, even with the
  revoked texts in hand; filter plus prompt 0%.
- **By model, capability doesn't predict the rate.** Rates run from 15.0%
  (Grok-4-Fast, non-reasoning) to 60.0% (Gemini-2.5-Flash), and the flagship
  tier spans 33.3–58.9%. Prompt hardening *raised* Gemma-3-27B-IT from 38.9%
  to 43.3%.
- **By scenario.** With no prompt rule: **81.0%** (583/720), up to 94.4% on
  PII export. With an explicit prohibition: 12.9%, but backup deletion
  (46.1%) supplies 83 of the 116 bypasses, leaving **4.6%** for the other
  four. The two groups are not a controlled comparison, and the authors say
  so.
- **Guard.** Where a mark is exposed, it matches the filter (0/1,620;
  18/18 and 12/12 revoked records caught, no current record withheld). Under
  indirect insertion it does what no filter can: **0/810** on both mem0
  configurations (vs 5/810 and 24/810 for the store filter) and 315/810 vs
  364/810 on Zep. It is also *worse than no defense* on cognee (53 vs 44/810)
  and Graphiti (83 vs 81/810). It catches only 4/14 on Zep and withholds
  current facts (6/15 langmem, 8/25 mem0, 6/24 mem0 exp. under indirect
  insertion). An **earlier version broke ties by result order and turned a
  0% unsafe rate into 100%**: relevance rank is not chronology.
- **Write-back.** 714/1,620 hop-0 runs were poisoned. In *direct* mode every
  journal was stored as current (356/356). With default retrieval, later hops
  go unsafe 98.0 / 95.8 / 96.3%. **With the filter: 71.6 / 79.5 / 83.1%**,
  stable or rising, against 0.0% without write-back. In *natural* mode
  Graphiti's ingestion kept only 43/358 journals and marked 21 of those
  superseded unprompted, leaving 22 active. Filtered rates were still
  7.5 / 7.8 / 12.8%, and 19 of the 22 active-journal runs went unsafe at
  hop 1 under the filter.
- **Propagation.** All three roles retrieved r1 in 810/810 runs each. Given a
  poisoned executor, the reviewer agreed **96.3%** of the time (341/354), and
  some later role went unsafe in 99.4%. All three roles were unsafe in 41.6%
  of runs. Filtering every read brings every column to 0/810. Without the
  journal, reviewer agreement falls to 81.7% and mean unsafe roles to 1.23
  (1.32 with it). The journal compounds propagation but doesn't cause it.
- **Tools.** No defense: dangerous tool invoked 17.2% (62/360), secret read
  5.0%, exfiltration confirmed 2.5%. Filter: 2.2 / 0.3 / 0%. Guard:
  2.2 / 0.8 / 0%. **Tool-layer guardrail: 17.2 / 5.0 / 1.9%**, the same as no
  defense. It refused 9 commands, but the revoked policy licensed actions the
  guardrail didn't classify as dangerous. Relapse: after the first agent
  journals, a second agent repeats the dangerous action in 42/62 runs (67.7%)
  under default retrieval and 13/62 (21.0%) with the filter at its read.
  Interpreter one-liners of the agent's own design appeared in 2/90
  undefended trials.

## Critique / open questions

- **The headline "no system enforces revocation by default" is carried by
  indirect insertion and by a non-default configuration.** Shipped mem0
  withholds expired records (0/810 under every condition, Table 13). The
  "one boolean" result is mem0 working as shipped against mem0 with its
  filter turned *off*. The fair summary: marks that exist are enforced by at
  most one vendor, and under ordinary prose ingestion most systems never
  create the mark.
- **The agent is structurally denied the label.** The harness passes
  `txt(r)` only, never status or timestamp. That makes prompt hardening fail
  by design, and it leaves the cheapest mitigation untested: put validity
  metadata in the context. [[literature/papers/hu2026memory]] found that
  *removing* a label amplifies over-trust, so label-in-context is a real
  candidate defense, and this paper says nothing about it.
- **The exposure count is inflated by the model axis.** Retrieval doesn't
  depend on the model, so "81/81" is effectively 9/9 scenarios repeated nine
  times.
- **Scenario design dominates the between-cell variance, and the scenarios
  are authored.** There are 9 hand-written policies, a binary action choice,
  no CIs (an ICC analysis is said to be in the bundle), and pooled rates
  that swing on one outlier (backup deletion). The "revoked phrasing is more
  absolute, so similarity favors it" explanation for rank-1 is post hoc and
  not ablated.
- **The authors bound their own statistics honestly.** With 17 discordant
  pairs, the cross-system test could only detect roughly a 13-to-4 split, so
  "not distinguishable" means "not grossly different." They say this
  themselves, and they state that the guard's agreement with the filter is
  not independent evidence, since both read the same field.
- **Stage 2 of the guard is brittle.** It relies on antonym lists, negation
  counts, and quantity sets, is marginally harmful on two systems, and
  withholds current facts. It is a lexical heuristic that would fall to
  paraphrase in the way SENTINEL fell in
  [[literature/papers/karamchandani2026your]]. Its one independent win is
  under indirect insertion on mem0.
- **Appendix C runs on Graphiti only**, and the write-back rates are
  properties of Graphiti's ingestion pipeline. Direct-mode write-back is the
  controlled upper bound. Natural mode (12% of journals retained) is closer
  to deployment and much milder, though still non-zero under the filter.
- **The paper contradicts the documented behavior of an in-graph source.**
  [[literature/papers/chhikara2025mem0]] specifies that contradicted memories
  are deleted. This paper measured retention with an expiry marker (direct
  insertion) and *no revocation at all* under indirect insertion. langmem's
  documented update/remove also fired only inconsistently.
- **The disclosure is worth recording.** The manuscript was drafted with
  help from Claude Opus 4.8 and GLM-5.3-Flash, and the authors take
  responsibility for the content.
- **Open question.** Write-back defeats the read filter because the journal
  carries no dependency pointer to r1. [[literature/papers/chen2026fresh]]'s
  cite-the-records-you-used protocol would catch r3 when r1 is revoked. That
  composition is untested, and it is the obvious next experiment.

## Trust signals

- **Credibility:** 3. An arXiv preprint from an industry research group
  (Vulcan Research, AIFT, Singapore) with no prior track record in this
  graph; not peer-reviewed; no citations established. Raised to 3 by a
  **released code and results bundle** (the guard, per-trial run data,
  verbatim tool attempts), a large and cheap-to-reproduce grid (49k calls,
  $79), matched-pair statistics whose power the authors bound themselves,
  and forthright negatives: the guard is worse than no defense on two
  systems, the tie-breaking bug is reported, and the two scenario groups are
  admitted to be uncontrolled. Held at 3 because the scenarios are authored,
  every beyond-single-read result comes from one system, and the headline
  framing overreaches the mem0 default result.

## Follow-up

- **Relevance:** 4. It adds material new evidence that **constrains
  [[concepts/verified-memory-writes]]**. The concept's definition places
  trust at consolidation time "not left to retrieval-time filtering." This
  paper is a clean benign case where every write is correct and authorized
  and the failure lives entirely on the read side. Its write-back result
  then shows a read filter alone fails too, because the agent's own write
  launders the revoked conclusion into a current record. The two halves
  close into one loop. It doesn't seed a new concept, so it scores 4 rather
  than the digest's proposed 5.
- **Independence.** This is the third group, after
  [[literature/papers/hu2026memory]] (PayPal AI, a stale note beats an
  authoritative tool) and [[literature/papers/chen2026fresh]] (a fresh state
  still yields an obsolete plan), to find that superseded authority wins at
  read time. Each measures a *different layer*: store retrieval here, model
  trust in hu, plan dependency in chen. That is convergence, not
  restatement. It is not independent of [[literature/papers/louck2026securing]]
  or [[literature/papers/sharma2026smsr]] as framing, since it cites both,
  but its measurements are new. It is in tension with hu2026memory on scale:
  hu finds harm grows with capability, while here capability tier doesn't
  predict the unsafe rate. The setups differ (a dressed-up stale note vs two
  policies of equal standing with no metadata), so they don't contradict
  each other.
- **[[concepts/enforcement-boundary-placement]].** This paper runs the
  **head-to-head placement comparison** that concept's open question says no
  source has done. Before the read (filter or guard): 0% and 2.2% tool
  execution. After the read in context (prompt): 37.2%. After the read at
  output: 18.1%. At the tool: 17.2%, the same as no defense. It also restates
  that concept's rule 3 (evaluate at the moment of use) for memory:
  retrieval-time validity.
- **[[concepts/shared-substrate-contagion]].** This is measured,
  attacker-free evidence that "a second opinion drawn from the same store is
  not independent": the reviewer agrees 96.3% of the time with a poisoned
  executor. Here the shared channel carried no correction, which qualifies
  the symmetric-detection result from [[literature/papers/paglieri2026case]].
- **This repository's import contract.** Downstream projects may `@import`
  a `status: retired` concept with only a warning. That is soft revocation
  at the read interface, but it is *not* the paper's form (3). The status
  travels inside the imported file text, so the reader sees the label, which
  is the label-in-context condition the paper never tested. The
  finding that bites harder is write-back. Downstream NOTES, ADRs, and code
  derived from a concept before it was retired persist as current records,
  and nothing propagates the retirement. The `used_by:` back-references are
  exactly the dependency pointers that would let a retirement be pushed to
  every consumer. The same pattern exists inside concept files, where
  superseded prose ("treat sufficiency as open") sits beside the later
  section that settles it, with equal standing.
- **[[concepts/context-eviction-policy]]: declined.** That concept scopes
  itself to the in-context buffer, while this paper is about retrieval from
  the long-term store. The guard's "withhold with reason, delete nothing"
  resembles a fold more than a prune, but that is an analogy, not evidence.
- Same-week, same-surname paper, **different author and group**:
  arXiv:2609.08279, "What Eviction Destroys" (Chen Shen, Megagon Labs) —
  [[literature/papers/shen2026what]]. It is the actual eviction-side source.
