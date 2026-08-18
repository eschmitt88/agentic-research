---
kind: paper
title: "From Lossy to Verified: A Provenance-Aware Tiered Memory for Agents"
authors: ["Qiming Zhu", "Shunian Chen", "Rui Yu", "Zhehao Wu", "Benyou Wang"]
institutions: ["School of Data Science, The Chinese University of Hong Kong, Shenzhen"]
year: 2026
venue: "arXiv 2602.17913v1, cs.DB (preprint, 20 Feb 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2602.17913
code_url: https://github.com/FreedomIntelligence/Tiermem
citations:
source: "raw/papers/zhu2026lossy.pdf"
added: "2026-08-18"
relevance: 4
credibility: 4
status: read
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/citation-anchoring]]"
related_experiments: []
tags: [memory, provenance, tiered-memory, compression, retrieval, escalation, auditability, write-back, cost-fidelity]
---

# From Lossy to Verified: A Provenance-Aware Tiered Memory for Agents (TierMem)

## TL;DR

Names and measures a structural failure of summary-based agent memory: the
**write-before-query barrier** — compression commits to what to keep
*before* the system knows what a future query will hinge on, so "any
fixed-budget summary admits a worst-case query it cannot support with
traceable evidence." The fix is not to stop compressing but to make the
summary tier **provenance-linked** to an immutable raw store, and to let a
trained router decide per query whether summary evidence suffices
(`ANSWER`) or the raw source must be consulted (`ESCALATE`). Verified
findings are then written back as new summary units **still linked to their
raw sources**. On LoCoMo: 0.851 accuracy against a 0.873 raw-only upper
bound, at 54.1% fewer input tokens and 60.7% lower latency.

## Claims

- **The write-before-query barrier is structural, not a summarizer bug.**
  "Write-time compression forces irreversible retention decisions under
  uncertainty, requiring a preemptive bet on saliency before the query
  distribution is known." The worked example: compressing "severe peanut
  allergy" into "dietary preferences" creates an **unverifiable
  omission** — when later asked "is this snack safe?", the agent cannot
  trace or cite the constraint. "These failures are not merely occasional
  summarization errors; they are structural risks of lossy,
  write-before-query compression."
- **Raw-only grounding is auditable but wasteful, and not merely on cost.**
  Long inputs "can even degrade effective context utilization due to
  position-sensitive failures and attention dilution," and "most queries do
  not require raw-level evidence." So neither pole is right.
- **Retrieval is an inference-time evidence-allocation problem.** "Given a
  query, the agent should retrieve the lowest-cost memory granularity that
  still provides sufficient evidence for faithful, auditable answering." A
  practical system must (i) detect when retrieved summaries are
  evidence-insufficient, (ii) fall back to an auditable source of truth
  only then, and (iii) consolidate verified findings back into cheap memory.
- **A new metric: Unverifiable Omission Rate (UOR)** — the share of errors
  caused by evidence being *absent from* the summary tier rather than
  misread. It separates "the memory didn't have it" from "the model got it
  wrong," which a single accuracy number conflates.
- **Sufficiency is defined strictly, and trained for.** The router "must
  escalate if the summary lacks precise constraints (e.g. negations, exact
  values, attribution) required for a faithful answer." Its training
  objective explicitly includes a `λ_waste` term penalizing **false
  alarms** — escalating when summaries were in fact sufficient — so the
  goal "is not to maximize escalation."
- **Provenance pointers earn their keep on the escalation path
  specifically.** Tier-1 units carry links to the Tier-2 raw pages they
  came from, used to warm-start escalation instead of a global BM25 search.
- **Write-back is query-triggered and evidence-backed, unlike generic
  summarization.** "The system only commits what was needed and validated
  during escalation, revealing which details must persist under the
  write-before-query asymmetry." The new unit stays linked to the raw pages
  used, "maintaining the chain of custody." The controller supports
  `ADD` / `UPDATE` / `SKIP`.

## Methods

Two-tier store: Tier-1 is a fast provenance-linked summary index, Tier-2 an
**immutable** raw-log store. A lightweight learned policy `π_θ` inspects the
retrieved Tier-1 evidence and emits `ANSWER` or `ESCALATE`; on escalation
the system reads the linked raw pages and, if still insufficient, runs a
bounded Deep Search Loop capped at `T_max = 3` iterations. Router training
is staged: zero-shot → SFT (distilled from a GPT-5 teacher emitting a short
rationale plus decision) → GRPO. Backbone is GPT-4.1-mini throughout.
Benchmarks: LoCoMo and LongMemEval, against summary-centric baselines
(Mem0, LightMem, O-mem, MemOS) and raw/controller-centric ones (MemR3,
GAM). Token accounting is decomposed so the router's own cost is visible
(`Tok_in = Tok_QA + Tok_router`). **Online write-back is disabled for all
methods in the main comparison** and evaluated separately in an epoch-wise
replay protocol: Tier-1 frozen within an epoch, verified findings logged,
applied as one batch between epochs.

## Results

- **LoCoMo.** Summary-only 0.755 (UOR 15.4%), raw-only 0.873, **router
  0.851** — with input tokens down 54.1% and latency down 60.7% versus
  raw-only. Summary-centric baselines carry UOR of **14.7%–23.3%**,
  confirming that their errors are largely evidence-*unavailability*, not
  misreading. Best baselines: GAM 0.869, MemR3 0.860, at 15,774 and 4,586
  input tokens/query respectively against TierMem's 3,980.
- **LongMemEval.** The gap widens with long-range dependencies — static
  summaries fail to retain answerable evidence for nearly 30% of queries
  (Mem0 UOR 29.6%).
- **Router behavior.** SFT+GRPO escalates **39.0%** of queries, recalls
  **71.7%** of oracle-hard cases, at 678 tokens/query overhead (584 in /
  94.5 out) = 14.7% of input tokens. Training progression on hard-case
  recall: zero-shot 44.7% → SFT 65.8% → SFT+GRPO 71.7%, with overhead
  *falling* (901 → 680 → 678 tokens) as format adherence improves. It
  matches a GPT-4.1-mini router's end-to-end accuracy (both 0.851) while
  saving ~51% of total system cost versus raw-only.
- **Provenance ablation — the cleanest result in the paper.** Linked 85.1%
  vs No-Linked (global BM25 escalation) 83.6%, +1.5pp overall — but the
  gain is concentrated exactly where the mechanism should act: accuracy on
  escalated queries rises **77.5% → 81.7% (+4.2pp)** while the `ANSWER`
  path is **unchanged**, at similar routing rates. "They do not change what
  summaries contain, but they improve how quickly and reliably the system
  finds decisive Tier-2 evidence."
- **Consolidation amortizes.** Across three replay epochs, overall accuracy
  stays stable while more queries are answered on the cheap Tier-1 path.
  Two write-back strategies compared: No-Recall (`ADD`/`SKIP` only, decided
  from the finding and its triggering query alone) vs **Retrieve-and-Edit**
  (retrieve related Tier-1 units first, then choose `ADD`/`UPDATE`/`SKIP`),
  the latter enabling updates and avoiding "uncontrolled growth of
  redundant entries."

## Critique / open questions

- **The router's classification quality is poor in absolute terms.** Best
  routing F1 is **40.6%** — it escalates 39% of queries to recall 71.7% of
  hard ones, meaning a lot of unnecessary escalation and a lot of missed
  hard cases. The paper concedes the oddity (an appendix explains "why
  similar end-to-end accuracy can arise despite different hard-case
  recall"). The honest reading: the *architecture* is doing the work, and
  a mediocre router still captures most of the benefit because the
  escalation path is forgiving. That is arguably good news for adoption,
  but it means "learned sufficiency detection" is not what's validated.
- **Accuracy is LLM-judge scored** on both benchmarks, so the headline
  0.851/0.873 inherits judge error — notable in a paper whose whole thesis
  is about verifiability of evidence.
- **Two conversational QA benchmarks** (LoCoMo, LongMemEval). Long-horizon
  personal-assistant dialogue is a very different query distribution from
  agentic tool-use traces, where the "raw log" is tool output and code
  rather than utterances, and where the decisive detail is often a stack
  trace or a config value.
- **UOR's own measurement is unspecified in the main text** — it is
  reported for baselines and for summary-only but dashed out for
  raw/controller-centric systems and for the router, which makes the
  cross-system column hard to read.
- **Immutability of Tier-2 is asserted, not enforced.** Nothing in the
  described system prevents raw-log mutation; the guarantee is a design
  convention. Compare the content-addressing that
  [[literature/papers/atinafu2026rewardhacking]]'s `evalhashlock` uses to
  make a similar claim checkable.
- Write-back necessarily re-introduces the write-before-query problem at
  one remove: the consolidated unit is a *new* lossy summary, chosen under
  a *past* query's notion of salience. Provenance links mean it is
  recoverable, which is the point — but the summary tier still drifts.
- Single-backbone (GPT-4.1-mini) for both memory system and router.

## Trust signals

- **Credibility:** 4 — code released; two benchmarks with six external
  baselines; token accounting decomposed so the router's own overhead is
  visible rather than hidden in the win; a genuinely well-designed
  provenance ablation whose effect lands precisely where the causal story
  predicts (escalated queries move, `ANSWER` path does not); a training
  ablation showing the staged router's progression; and a new metric (UOR)
  that isolates the failure mode being claimed. Held below 5: unreviewed
  preprint, LLM-judge accuracy, one backbone, conversational-QA domain
  only, and a router whose standalone classification quality is weak.

## Follow-up

- **Relevance: 4** — this is the *read*-side counterpart the memory cluster
  was thin on. [[concepts/verified-memory-writes]] covers making a write
  trustworthy; TierMem covers making a **recall** checkable, via a link
  from the compressed unit back to the raw evidence it was derived from.
  The "chain of custody" framing — a consolidated fact carries pointers to
  the raw pages that justified it — is [[concepts/citation-anchoring]]
  applied to the agent's own memory rather than to literature, and it is
  the same structural claim: a claim ships with the means to check it.
- **The write-before-query barrier is the sharpest available argument for
  [[concepts/lossless-context-offload]].** The concept holds that offloaded
  material must stay addressable; this says *why*, in a form that does not
  depend on cost — because compression is a bet on the future query
  distribution, and "any fixed-budget summary admits a worst-case query it
  cannot support with traceable evidence." That is an impossibility
  statement about summarization, and it means the addressability is not an
  optimization, it is what makes the summary safe to take at all.
- **UOR is worth stealing as a diagnostic.** Separating "the memory didn't
  contain it" from "the model got it wrong" is exactly the split this
  project cannot currently make when a session fails to recall something —
  and the baseline rates (14.7–23.3%, and ~30% at longer range) suggest
  omission dominates. Any evaluation of a compaction step here should
  report the omission share, not just accuracy.
- **The escalation router is a concrete instance of
  [[concepts/selective-memory-retrieval]] with a cost-aware objective.**
  What is new relative to that concept's existing sources is the `λ_waste`
  penalty on **false escalation** — the gating decision is trained against
  the cost of consulting memory unnecessarily, not only against the cost of
  failing to. That is the same symmetry
  [[concepts/refusal-cost-symmetry]] argues for, arriving in a training
  objective.
- **Two-tier + router is [[concepts/multi-granularity-memory]] with the
  granularity chosen at query time rather than at write time**, which is
  the concept's more interesting form. The provenance ablation is the
  useful evidence: the *links* between granularities matter more than the
  granularities themselves (+4.2pp on the escalated path, 0 on the cheap
  path). A tiered store without back-pointers gets most of the cost saving
  and little of the fidelity.
- **Retrieve-and-Edit write-back is the pattern for this repo's own
  ingest.** Retrieving related existing units before committing a new one —
  and choosing `ADD` vs `UPDATE` vs `SKIP` — "avoids uncontrolled growth of
  redundant entries," which is precisely the dedup discipline `/ingest`
  applies to concepts by hand. Worth noting that an automated system found
  the same design necessary.
- **Open thread worth watching.** Write-back re-creates a lossy summary
  under a past query's salience, so the tier drifts even though every unit
  stays traceable. Nobody measures whether a consolidated store degrades
  over many epochs — the paper runs three. That is the memory analogue of
  the audit-debt problem [[literature/papers/ding2026autonomous]] names.
