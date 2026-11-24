---
kind: paper
title: "Surfacing Disagreement Beats Resolving It: A Gated Ablation Study of Provenance-Weighted Memory for Multi-Agent Debate"
authors: ["Ziaullah Khan", "Abdul Rehman Khalid", "Shrestha Nischal Lal", "Nheng Vanchhay", "Md Ariful Islam Mozumder", "Hee-Cheol Kim"]
institutions: ["Inje University (Department of Digital Anti-Aging Healthcare)", "Inje University (Department of Computer Engineering)"]
year: 2026
venue: "arXiv 2611.22461 (cs.MA), v1 2026-11-17; NeurIPS 2026 workshop paper (PALM: Personalized, Aligned, Long-Term Memory for AI Systems); 8-page main text + 4-page appendix + checklist"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.22461"
code_url: null    # harness, attack set and per-query logs "released" (Sec. J, checklist 5), but the PDF gives no URL
citations: null
source: "raw/papers/khan2026surfacing.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hce-evaluation]]"
tags: ["memory-poisoning", "conflict-surfacing", "contradiction-handling", "write-gate", "supersession", "multi-agent-debate", "knowledge-graph-memory", "provenance", "bi-temporal", "locomo", "hotpotqa", "pre-declared-gates", "exposure-denominator", "negative-result", "gpt-oss"]
---

# Surfacing Disagreement Beats Resolving It: A Gated Ablation Study of Provenance-Weighted Memory for Multi-Agent Debate

## TL;DR

DebateGraph is a shared memory service for a three-agent, two-round
debate with a judge (gpt-oss-120b throughout). Five mechanisms from the
agent-memory literature sit behind one read/write/snapshot contract.
Each can be switched off alone, and each has a threshold declared
before the run. The attack set plants one contradicting edge per
eligible LoCoMo item: same subject and relation, a type-consistent wrong
object taken from elsewhere in the corpus, a fourth non-debating writer,
and a timestamp 30 days later so that recency favours the lie.

Four results matter for this graph:

1. **Showing the reader the whole ranked conflict set beats handing it
   the winner.** At the same retrieved set and budget, passing every
   competing claim with its writer, trust, evidence count and
   supersession status raises F1 on the 122 poisoned items from **0.116
   to 0.456 (+0.340, CI [+0.267, +0.414])**. Attack success falls from
   **22 to 7 of 122** (16 vs 1 discordant pairs, McNemar p = 0.0003).
   Tokens per query are within 2%. The collapse arm's ranking puts the
   lie first by construction whenever the conflict is retrieved
   (recency-first, the adversary sets the timestamp).
2. **That headline is from a regime chosen after the first run.** At the
   pre-declared retrieval cap (40 edges) only **12 of 51** planted
   contradictions were ever retrieved. There, surfacing gained **+0.043**
   overall and **+0.084** on poisoned items, and attacks fell 5 → 3 of
   51 (p = 0.63, not established). The rerun at cap 200 reaches 40 of
   51. Per item where a conflict actually reached the prompt, the effect
   was already **+0.329** at cap 40 (13 items). So the "eightfold"
   correction is mostly dilution: the cap changed how many items were
   exposed, not how much each exposed item gained.
3. **A groundedness-based write gate cannot see this attack, and its
   damage came from supersession, not admission.** Poison objects are
   entities already in the graph in **100%** of cases (by construction
   of a type-consistent distractor), against **53%** of gold objects, so
   endpoint groundedness scores the lie above a truthful first mention.
   The gate as specified rejects **0/122**; three repairs reach 3/122. At
   cap 200 it costs **−0.075 F1 (CI [−0.133, −0.018])** and moves
   attacks 7 → 9. Its write log shows **123 invalidations against 3
   rejections** over those items: it mostly marked true claims
   superseded.
4. **With contradictions reachable, two "inert" mechanisms became
   harmful.** The same cap change that grew the surfacing result turned
   the write gate (−0.019 n.s. → −0.075) and the router's provenance
   features (−0.002 → −0.096, CI [−0.149, −0.043]) from nulls into
   harms. The authors report both and keep the pre-declared cap-40
   numbers unchanged in Table 6.

Full context beats every retrieval configuration (conflict slice 0.609
vs DebateGraph 0.384), because every LoCoMo partition fits in 32K
tokens. The paper says so and motivates retrieval by cost only.

## Claims

- Memory systems treat consistency as a design default: when facts
  conflict, the store reconciles them (recency, confidence or an LLM
  merge) and the reasoner never sees the disagreement. In a debate
  system this spends the store's weakest evidence (a heuristic
  tie-break) to pre-empt its strongest capability. Under a writer who
  controls timestamps it is exploitable.
- "The storage layer should not adjudicate." Surfacing helps, ranking
  order inside the surfaced set barely matters (agents override a bad
  ranking), and the one component that adjudicates on write costs
  accuracy once it has live claims to act on.
- Groundedness ("does this connect to what I already know") is the wrong
  prior for write admission against a writer who builds lies from known
  entities. The discriminating signal is relative (candidate against
  incumbent on the same key). The authors suggest admission needs
  corroboration or source-level signals, not topology. That is a
  recommendation, not a tested arm.
- "A mechanism can only be measured where it is able to act, and
  checking that it can act is a separate obligation from checking
  whether it helps." Per-item conflict counts, logged whether or not any
  analysis used them, found the cap problem.

## Methods

- **Store.** LoCoMo: 10 conversations, 5,878 episodes (one per dialogue
  turn), 9,769 triple edges from a cached batched LLM extraction. Each
  edge carries writer, ingestion time, valid and invalid times, trust ρ,
  source type and support. Edges are never deleted; supersession sets
  `t_inval` and both versions persist.
- **Mechanisms (one switch each).**
  - *Graph as default backend* (vs vector ANN over local embeddings,
    1-hop, k-hop, hybrid).
  - *Write trust-gate*: ρ = c_writer · (w_g·g + w_b·b + w_r·κ) · (1 − γ),
    no LLM call. g is endpoint groundedness, b a BFS compatibility term,
    κ relation compatibility, c a writer credibility, γ a contradiction
    penalty. Commit iff ρ ≥ the running median of committed scores. The
    same switch governs invalidate-on-supersede.
  - *Conflict resolution*: group retrieved edges by (subject, relation);
    rank competing objects lexicographically by (not invalidated,
    t_valid, ρ, number of writers). Only single-valued relations can
    conflict; same-time objects are elaborations. **Collapse** passes the
    top claim; **surface** passes the ranked set with provenance.
    `source_type` is never rendered, since labelling an injection would
    give away the answer.
  - *Router*: heuristic vs disjoint LinUCB bandit over query features
    (including a conflict-present flag), reward u − λ·tokens/τ.
  - *Snapshot isolation (MVCC)*: each round reads a view pinned at the
    round boundary.
- **Attack (DebateMem-Conflict).** 122 of 603 items (20.2%) have a gold
  answer locatable as a single-valued edge object. Each gets one
  template-generated contradicting edge as described in the TL;DR.
- **Retrieval cap.** k-hop truncates to `graph_max_edges` candidates
  ordered by hop distance then path trust, never by relevance. Declared
  as 40 before the runs; 200 in Table 2 and the Phase 1 control. A
  1,500-token budget separately governs what reaches the prompt.
- **Debate.** N = 3 agents, R = 2 rounds, one judge: 7 calls per item.
  T = 0.2, p = 0.7. Token-F1 against gold. One shared retrieval per
  round for all agents. Every call is disk-cached, so runs replay
  byte-identically. 39 runs = 39 configurations, one sample each.
- **Statistics.** Paired on shared items: 10,000-resample bootstrap,
  two-sided Wilcoxon, exact McNemar on attack discordance. An effect
  counts only if the 95% bootstrap CI excludes zero. Two pre-specified
  subgroups: poisoned items, and discriminating items (a conflict
  reached the context, identified from the retriever alone).
- **Second corpus.** HotpotQA-distractor bridge questions, n = 100: one
  question's 10 paragraphs per partition, one sentence per episode,
  article title as writer. Event time is synthetic (one day per
  paragraph), so only a reduced form of the mechanism is tested.
- **Model choice.** The environment exposed two chat models. The 70B
  Llama endpoint sustained 0.02 req/s, and a second-family replication
  was abandoned when one debate item did not finish in 35 minutes.

## Results

- **Pre-declared gates (Table 1).** Phase 0 harness PASS; Phase 1 graph
  as default REFUTED; Phase 2 trust-gate INCONCLUSIVE; Phase 3 surfacing
  PASS; Phase 4 router FAIL; Phase 5 MVCC PASS but inert; Phase 6
  ablation grid FAIL. Recomputed by `scripts/gates.py`.
- **Phase 1 (Table 4).**

  | MULTIHOP-TEMPORAL | F1 | tokens/q | gold-evidence recall |
  |---|---|---|---|
  | full context | 0.503 | 147,702 | 1.00 |
  | vector | 0.304 | 6,405 | 0.50 |
  | graph (k-hop) | 0.154 | 11,199 | 0.30 |

  - Graph vs vector: −0.150 (CI [−0.216, −0.085]); SINGLE-HOP 0.048 vs
    0.288.
  - At cap 200, graph recall rises 0.30 → 0.76 and F1 moves 0.154 →
    0.155. More of the right triples did not help: the authors read the
    deficit as the form of delivery (triples vs verbatim dialogue), not
    coverage.
- **Phase 2, write gate (Sec. 6.2, App. D, Table 5).**
  - As specified: 0/122 rejected. Repairs (score corpus edges against the
    full graph; condition the threshold on displacing writes; make γ
    proportional to the displaced incumbent's trust) reach 3/122.
  - Poison ρ 0.583 (flat γ) or 0.440 (proportional) vs truthful
    restatement 0.825; the median-derived threshold (0.175–0.200) sits
    below both.
  - Cap 40: +2.7% latency, −15 tokens/query, rejects 1 of 51, F1 −0.019
    (CI [−0.047, +0.002]), ASR change 0.000.
  - Cap 200: F1 **−0.075** (CI [−0.133, −0.018], Wilcoxon p = 0.039),
    attacks 7 → 9 of 122, **123 invalidations vs 3 rejections**.
  - The authors did not weaken the attack (e.g. unseen-entity
    distractors) to make the gate look effective.
- **Phase 3, surfacing (Tables 2, 6, 7).**

  | Contrast (on − off) | cap 40 | cap 200 |
  |---|---|---|
  | surfacing, graph backend | +0.043* | **+0.340*** |
  | surfacing, router backend | +0.077* | +0.143* |
  | write trust-gate | −0.019 | **−0.075*** |
  | router provenance features | −0.002 | **−0.096*** |
  | trust-weighted retrieval | −0.059 | −0.065 |
  | temporal validity in ranking | +0.001 | −0.004 |
  | write gate (full-system ablation) | +0.012 | +0.004 |

  `*` = bootstrap CI excludes zero. The cap-40 column is on 100 items
  (51 poisoned); the cap-200 surfacing row is on the 122 poisonable
  items, all attacked. So the columns differ in item set as well as cap.
  - Reachability sweep (offline, no LLM calls): planted contradiction
    retrieved for 12/51, 27/51, 40/51 poisoned items at caps 40, 100,
    200.
  - Discriminating items: 13/100 at cap 40, 109/122 at cap 200.
  - Cap 40, poisoned subgroup: graph +0.084 (CI [+0.022, +0.160]);
    router +0.195 (CI [+0.108, +0.292]). Discriminating subset +0.329
    (CI [+0.115, +0.572]).
  - Cap 200: tokens 11,182 → 11,318 (collapse) and 11,224 → 11,550
    (surface). The collapse arm's F1 does not move (0.1165 → 0.1158);
    its ASR rises 0.098 → 0.180 once the lie is reachable.
  - Tie handling: at cap 40, 94 of 100 pairs are exactly zero. Keeping
    and splitting tied ranks gives Wilcoxon p = 0.2595; discarding them
    gives p = 0.0260, on the same data. The authors rest no claim on
    either and use the bootstrap CI on the pre-specified subgroup.
- **Ablations against the full system (Table 9, cap 40, n = 100).**
  Only removing surfacing moves F1 with a CI excluding zero (0.321 →
  0.243). Removing temporal validity raises the resolver's own
  conflict-resolution F1 from 0.000 to 0.387 and leaves answer F1 at
  0.320. Removing trust weighting raises F1 to 0.380 (CI [−0.013,
  +0.132]) and evidence recall 0.43 → 0.62.
- **HotpotQA (Table 7).** Discriminating subset +0.111 (CI [+0.010,
  +0.232]) on 33/100 items; all items +0.032 (CI [−0.005, +0.077]);
  attacks 3 → 2 of 67 (p = 1.00). The cap cannot bind there (34/67
  conflicts reachable at either cap). The collapse arm already scores
  0.388 there, against 0.116 on LoCoMo.
- **Router (Table 8).** The bandit beats the heuristic (+0.063, CI
  [+0.023, +0.111]) but not always-vector (+0.010, CI [−0.013, +0.040]),
  and it picks vector on 93% of queries. The λ sweep is flat (F1
  0.357–0.384).
- **Baselines (Table 9).** Full context 0.609 on the conflict slice,
  +0.225 over DebateGraph (bandit, 0.384); vector-only 0.374;
  graph-only 0.131.
- **AutoGen transfer (Table 10, n = 60).** The service runs unmodified
  in RoundRobinGroupChat. Vector 0.339 vs full system 0.251; ASR 5 → 4,
  not significant. Claimed as transfer of the abstraction, not a gain.

## Critique / open questions

- **Against the digest entry: at.** The summary's numbers are in the PDF
  and quoted correctly. It overreaches in framing:
  - The **+0.340 and 22 → 7** headline is the cap-200 rerun, chosen
    after the pre-declared run exposed the cap. The pre-declared result
    is +0.043 (all) / +0.084 (poisoned) with no established ASR effect.
    The paper is open about this; the summary is not.
  - "**Two mechanisms are worse than inert**" holds only in the cap-200
    regime. At the pre-declared cap both were nulls.
  - "**The gate fails structurally because poisoned objects are always
    entities already in the graph**" is true by construction of the
    attack (type-consistent distractors drawn from the corpus). It says
    groundedness gates fail against same-key lies built from known
    entities, not that write gates fail.
  - The summary misses the mechanism of the gate's harm: 123
    invalidations against 3 rejections. The harm is the gate's
    *supersession* side effect marking true claims superseded, which the
    surfaced prompt then shows the debate.
  - "**Only one survives**": MVCC passes its gate too, though the
    authors call it not load-bearing (agents write no triples). The
    bandit passes its pre-declared ship decision (beats the heuristic)
    and fails the efficiency gate.
- **The *Why* line, item by item.**
  - "*A counter-result for `verified-memory-writes` (a write gate that
    costs accuracy)*": partly. The gate is one specific non-LLM
    topology-plausibility gate with a median threshold. It is the
    content-scoring class that
    [[literature/papers/louck2026securing]] already argues cannot be
    sound. No origin-bound, corroboration-based or verifier-model gate
    is tested. What it does counter is the concept's Definition clause
    that trust belongs at consolidation "not left to retrieval-time
    filtering", and any write path that supersedes automatically.
  - "*An input to `shared-substrate-contagion`*": weak. The store is
    shared by three agents, but the poison is one planted record, and
    agents' own writes assert no triples (App. H). Nothing propagates
    between agents or sessions. It is a poisoning-and-presentation
    result, not a contagion one.
  - "*Shows what the hub does in its own notes: keep the contradiction
    visible*": as an analogy, yes. The tested reader is a gpt-oss debate
    over dialogue triples, not a Claude session over Markdown concepts.
    It bears most directly on [[concepts/llm-wiki-pattern]]'s open
    question, which proposes trialling an automatic reconciliation pass
    (recency × authority × confidence, losers archived). That is the
    collapse arm.
  - "*Pre-declared gates are an `/elevate`-style evidence discipline*":
    the thresholds are declared and recomputed by script, but nothing
    shows they were bound before the run (no hash or commit;
    [[literature/papers/waks2026what]]'s M2). And the headline came from
    a post-hoc regime. The discipline is in what they kept (pre-declared
    numbers reported unchanged, every null re-run in the new regime),
    not in a binding.
- **The collapse baseline loses by construction on every exposed item.**
  The ranking is (not invalidated, newest, ρ, writers), and the lie is
  always newer. So collapse hands the debate the lie whenever the
  conflict is retrieved. The gain measures "surface" against the worst
  collapse an adversary with timestamp control can force. That is the
  paper's stated threat model, and a recency tie-break is common. But no
  run compares surfacing with a collapse ranked by trust or support
  first. Table 9's "− temporal validity" row shows that ranking makes
  the resolver right 38.7% of the time, but it was run with surfacing
  on.
- **The surfaced lie carries weaker provenance by construction.** It has
  one writer, a non-speaker, and lower ρ (0.58 or 0.44 vs 0.83 for a
  restated truth). The debate may be reading those fields rather than
  reasoning about content. Two results fit that reading: ranking order
  inside the surfaced set did not matter (temporal validity inert at
  both caps), and the gate's false supersession marks on true claims
  hurt. No arm tests a lie with forged support (several writers, or
  FARMA-style amplification as in
  [[literature/papers/karamchandani2026your]]), or a surfaced set with
  the provenance fields stripped. "Let the reasoner reason" is supported
  only as "let the reasoner read honest provenance".
- **Debate is not isolated.** There is no single-agent arm, so whether
  the gain needs three agents and two rounds, or any reader shown both
  claims, is untested. The paper's argument ("a debate system already
  contains machinery for adjudicating") is stronger than its design.
- **One sample per configuration.** Cached calls at T = 0.2 make each
  run deterministic, and the bootstrap is over items. No run-to-run
  variance is estimated, so the CIs cover item sampling only.
- **One model family** (gpt-oss-120b), template-generated injections,
  20.2% of items eligible (entity-like factual answers), one judge, and
  low absolute F1. Full context wins everywhere, so the regime where
  memory overflows the window, and reconciliation becomes unavoidable,
  is not exhibited (the authors' own limit).
- **Small inconsistency.** The cap-40 surfacing CI is [+0.010, +0.084]
  in Sec. 6.3 and [+0.010, +0.082] in Table 7.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/li2026benchmark]]: a consumer with an uncontested
    false belief asserts it in 0.97–0.99 of probes, and a competing true
    belief cuts this to 0.15. This paper is the controlled intervention
    on the same lever: put the competing claim in front of the reader.
  - [[literature/papers/hu2026memory]]: removing a label amplifies
    over-trust, and a recency cue fools stronger models. Here recency
    ordering did not matter once both claims and their provenance were
    shown. Different models and settings, so this is a contrast to
    test, not a contradiction.
  - [[literature/papers/gao2026mempoison]]: write-time and
    retrieval-time defences fail on disjoint tiers. This paper adds that
    a heuristic write-time defence can be net harmful through its
    supersession side effect.
  - [[literature/papers/shen2026revoked]]: a store-level status filter on
    revoked records fixed its failure. Here a status mark (superseded)
    written by a bad heuristic caused one. Status fields are powerful in
    both directions.

## Trust signals

- **Credibility:** 3.
  - For:
    - Unusually disciplined reporting: pre-declared thresholds with
      verdicts recomputed by script, four of seven gates reported as not
      passing, pre-declared numbers kept beside the correction, and every
      null re-run in the corrected regime (two became harms, both
      reported).
    - Paired design, bootstrap CIs, exact McNemar, and Wilcoxon
      tie-handling sensitivity disclosed rather than chosen.
    - A second corpus picked so the confound cannot bind, and a
      framework-transfer check reported as no gain.
    - Code, attack set and per-query logs for all 39 runs claimed as
      released; byte-identical replay via a request cache.
  - Against:
    - A workshop paper (NeurIPS 2026 PALM), not peer reviewed at a main
      venue; one university group with no prior record in this graph;
      seven days old, no citations.
    - No artifact URL in the PDF.
    - One model family, one sample per configuration, synthetic
      single-writer attacks, and the headline from a post-hoc regime.
  - Not 4: workshop-level review, a single model and no located code.
    Not 2: the controls and the honesty of the reporting are well above
    most preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - Material new evidence for [[concepts/verified-memory-writes]]: a
    content-scoring write gate whose harm comes from automatic
    supersession, with the mechanism counted (123 vs 3).
  - The first controlled test in the graph of what the read path should
    do with a conflict: surface it with provenance, not resolve it. That
    bears on [[concepts/selective-memory-retrieval]] and on
    [[concepts/llm-wiki-pattern]]'s reconciliation open question.
  - Not 5: QA over dialogue memory with a debate reader, not research
    agents; one model; synthetic single-writer attacks.
- **Experiments that would settle the import.**
  - Surfacing against a collapse ranked by trust or support first, not
    recency.
  - Surfacing with provenance fields stripped, and against a lie with
    forged multi-writer support.
  - A single-reader arm against the debate.
- **Candidates** (cited, not yet in the graph):
  - Meng et al. 2026, "EquiMem: calibrating shared memory in multi-agent
    debate via game-theoretic equilibrium" (arXiv 2605.09278). Scores
    each write against global memory state, which this paper's gate
    diagnosis independently supports.
  - Tian et al. 2026, "Multi-agent debate with memory masking" (ICLR
    2026).
  - Cattan et al. 2025, "Dragged into conflicts" (arXiv 2506.08500): a
    knowledge-conflict taxonomy and benchmark.
  - Wang, Prasad, Stengel-Eskin and Bansal 2025, "Retrieval-augmented
    generation with conflicting evidence" (COLM 2025).
  - Rasmussen et al. 2025, Zep (arXiv 2501.13956): the bi-temporal
    knowledge-graph memory this paper's store resembles.
