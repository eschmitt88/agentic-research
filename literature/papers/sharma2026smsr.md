---
kind: paper
title: "SMSR: Certified Defence Against Runtime Memory Poisoning in Persistent LLM Agent Systems"
authors: ["Tarun Sharma"]
institutions: ["Independent researcher"]
year: 2026
venue: "arXiv preprint (cs.CR); submitted to IEEE"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.12703"
code_url: "https://github.com/tarun-ks/smsr"
citations: null
source: "raw/papers/sharma2026smsr.pdf"
added: "2026-07-27"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-claim-partition]]"
tags: [memory, write-policy, security, memory-poisoning, provenance, certified-robustness, formal-verification, randomised-smoothing]
---

# SMSR: Certified Defence Against Runtime Memory Poisoning in Persistent LLM Agent Systems

## TL;DR

Reaches the same conclusion as [[literature/papers/louck2026securing]] —
retrieval-time content filtering cannot be sound, write-time provenance
is necessary — by a completely different formal route (an
embedding-density impossibility argument plus a hypergeometric
robustness certificate, rather than TLA+ model checking). Two components:
HMAC-SHA256 signing at write time, which takes unsigned attacks from
93–100% to **0%**; and randomised memory ablation with verdict-based
majority voting for the *authenticated* adversary, which only bounds the
attack rather than eliminating it — 8.0% ASR [5.8, 10.9] against a
certified worst case of 10.4%. That residual is the paper's most useful
contribution to us: it marks precisely where origin-binding stops.

## Claims

- **MSMP (Multi-Session Memory Poisoning) is structurally outside what
  static-corpus RAG defenses can cover.** RobustRAG and ReliabilityRAG
  give certified bounds, but all three of their assumptions fail here:
  the corpus is not indexed offline (memory is a live append-only log),
  the adversary does not poison externally (it writes through the same
  path legitimate users do), and the number of poisoned documents in the
  retrieved set is not an a-priori constant.
- **Theorem 1 (impossibility).** Under a fluent-embedding-density
  assumption — for any target query embedding and similarity threshold
  there exists a fluent string within that ball — no *deterministic
  content-based* retrieval-time filter achieves a non-trivial worst-case
  certificate against an unbounded adaptive adversary. Keyword
  blacklists, perplexity proxies, and semantic-anomaly filters are all
  instances of the covered filter class. Provenance at **write** time is
  therefore necessary.
- **Component 1 (HMAC provenance) is exactly (0, 2⁻²⁵⁶)-secure against
  the unsigned adversary** under HMAC-SHA256 pseudorandomness. Cheap and
  total — but only against an attacker who cannot reach the signing
  oracle.
- **Component 2 is deliberately outside Theorem 1's scope.** It is
  randomised and *content-agnostic*: over-fetch m ≫ k verified
  candidates, sample k uniformly without replacement across n_runs
  independent trials, and majority-vote. Theorem 2 derives a
  hypergeometric certificate δ(t', m, k, n_runs) on the probability of a
  malicious majority. The impossibility result applies to deterministic
  content-based filters, which this is not.
- **Consistent Minority Effect (CME) — the paper's sharpest practical
  finding.** String-based majority voting is *systematically* gameable
  in this setting, and not by accident: adversarial responses restate
  one specific, repeatable false claim while benign responses paraphrase
  freely ("the limit is $50,000" vs. five distinct ways of saying "I
  don't know"). So a numerical minority wins the string vote. Proposition
  2 bounds this by clean-response min-entropy: as paraphrase diversity
  rises, the adversary's win probability approaches 1. Measured at 93.3%
  ASR under string voting vs 13.3% under verdict voting on the *same*
  ablation runs (n=15), affecting 12 of 15 scenarios. **Fix: aggregate
  over typed verdict labels {correct, malicious, neither}, not raw
  strings.**
- Heuristic content defenses fail exactly as Theorem 1 predicts: a
  fluent, low-perplexity, no-blacklisted-keyword bypass attack achieves
  **100% ASR** against a keyword + entropy + semantic-anomaly filter, and
  0% against Component 1.

## Methods

- Nexora Corp simulated enterprise RAG agent; memory pre-seeded with 10
  signed memories; 15 scenarios across finance, compliance, IT, HR,
  procurement, incident response. Claude Haiku 4.5 as both agent and
  judge; m=20, k=5, n_runs=5, t ∈ {1,2,3}.
- Three attack classes: **unsigned** (MINJA-style direct DB write —
  direct/flooding/subtle variants), **authenticated** (legitimate user
  injecting through the normal agent path, entries get valid HMAC tags),
  **heuristic bypass** (unsigned but crafted to evade content filters).
  Four defense modes: none, heuristic, c1, c1c2.
- Scale: 3,150 repeated trials (six Component-2 configs × 15 scenarios ×
  30 reps) plus 450 production-scale trials; 585 single-run attack trials
  for the 39-config sweep.
- **Judge reliability is quantified rather than assumed**: 84 held-out
  responses re-judged by claude-sonnet-4-6, Cohen's κ = 0.955, 97.6% raw
  agreement.
- **The uniform-sampling premise of Theorem 2 is validated empirically**,
  not just asserted: a 3×10⁵-trial Monte-Carlo confirms the
  contaminated-run fraction matches 1 − p_clean to within ±0.0006 across
  t ∈ {1,2,3}.
- Cross-model generality: rerun with claude-sonnet-4-6 as agent gives
  *identical* 8.0% at m=20 — expected, since Component 2 operates at the
  retrieval layer and is agnostic to which LLM generates.
- E10 external-validity test: a **query-only** attack where the agent
  writes the poison itself through its own signed write path (true MINJA
  mechanism), rather than pre-seeding adversarial entries.

## Results

- Unsigned attacks, all six configs including the bypass variant:
  93–100% → **0.0%**. Heuristic defense manages only 93–100% → 87–100%.
- Authenticated t=1: 93–100% → 37.8% [33.4, 42.3] at the small
  evaluation pool (m′=11, δ=41.5%), and **8.0% [5.8, 10.9]** at the
  production pool (m=20, δ=10.4%). Certificate holds in both.
- Degrades as predicted with adversary budget: t=3 gives 92.7% ASR
  against δ=94.5% at m′=13. The certificate is honest about being nearly
  vacuous when t approaches m/2.
- E10 (query-only, live agent): injection reaches the retrieval pool in
  **100%** of trials; ASR 65.3% [57.4, 72.5] undefended → **5.3%**
  [2.7, 10.2] with c1c2 (n=150). ~12× reduction, non-overlapping CIs.
- Utility cost: Component 1 is free (90%, identical to undefended);
  combined defense 85% — a 5 pp cost from random subsampling of context.
- Overhead: 10 API calls per query vs 1 (n_runs × agent+judge), ≈2×
  latency with async batching, ≈$0.001/query at Haiku pricing.
- vs A-MemGuard (consensus fallback) at n=450, like-for-like 20-seed
  store: A-MemGuard 3.8% [2.4, 6.0], SMSR 8.0% [5.8, 10.9] — **CIs
  overlap, and the paper says so, calling the pre-registered outcome
  "comparable."** The claimed differentiator is the certificate and
  adaptive-adversary resistance, not the point estimate.
- Parameter guidance: m ≈ 19t for δ ≤ 0.10 at k=5, n_runs=5. t=1 → m=21;
  t=3 → m=57 (or 18/34/50 at n_runs=7).

## Critique / open questions

- **The headline "certified" claim covers less than it appears to.** The
  0% result is against the *unsigned* adversary, which is the easy case —
  it is what a signature trivially buys. Against the authenticated
  adversary, the one that matters for an agent that writes its own
  memory, the guarantee is a probabilistic bound with a real residual
  (8%), and it collapses entirely once t ≳ m/2. Read alongside
  louck2026securing this is clarifying rather than damning: TMA-NM
  claims 0% including against laundering, but louck's laundering
  channels (self-summarization, trusted-tool echo, manufactured
  corroboration) all involve *the agent itself writing*, which lands in
  SMSR's authenticated regime. The two papers agree on the necessity
  half and **disagree on whether origin-binding alone is sufficient**.
  That disagreement is the interesting object, not a defect in either.
- **Bounded-t is doing heavy lifting.** The certificate only means
  anything when the adversary's write budget is small, justified by
  per-user write quotas, near-duplicate detection, and audit logs.
  Those are exactly the controls a poisoning adversary would target
  first, and none are evaluated here.
- Independent single author, no institutional backing, not peer reviewed
  (submitted to IEEE). Offsetting this: proofs, released code and data,
  n=450 with confidence intervals, an inter-judge reliability study, a
  Monte-Carlo check of the theorem's own premise, and a pre-registered
  comparison the author reports as a tie. The methodology is more
  careful than the affiliation predicts.
- Evaluation is a single simulated enterprise-policy domain ("Nexora
  Corp") with an LLM judge and no human ground-truth labels. The judge is
  Claude judging Claude; κ against a second Claude model measures
  consistency within a family, not correctness.
- Acknowledged adaptive gap: an attacker who observes judge verdicts
  over time could craft responses the per-run judge scores "correct"
  while remaining harmful. Mitigations offered (rotate the judge,
  randomise its system prompt) are friction, not defense.
- 10× API calls per query is a real cost the abstract does not mention.
  Cheap at Haiku pricing; not obviously cheap at frontier-model pricing
  in a loop that queries memory constantly.

## Trust signals

- **Credibility:** 3 — independent researcher, no institution, preprint
  only (submitted to IEEE, not accepted). Pulled up to 3 rather than 2 by
  unusually strong reproducibility signals: code + data released,
  formal proofs with an empirically validated premise, n=450 with
  reported CIs, quantified judge reliability (κ=0.955), and a baseline
  comparison the author declines to claim a win on. Weighted per the
  rubric's instruction that reproducibility counts at least as heavily
  as affiliation.

## Follow-up

- **Relevance:** 4 — sixth attestation for
  [[concepts/verified-memory-writes]] and the second *independent* formal
  route to "write-time origin binding is necessary." Material new
  evidence rather than a new anchor: it does not reseat the concept
  (louck2026securing remains the stronger formal statement) but it
  supplies a numeric robustness certificate the concept lacked, and it
  identifies the authenticated-write regime as the open frontier.
- **The CME result generalizes past memory security.** Any system that
  aggregates several LLM samples by string agreement — self-consistency
  voting, ensemble judging, majority-vote evaluation — is vulnerable the
  same way: a specific repeatable claim beats diverse correct answers on
  string equality. Collapsing to a typed verdict before voting is the
  fix. Flagged into [[concepts/typed-claim-partition]]; worth checking
  whether any of our own aggregation ever compares strings.
- Direct tension to resolve: SMSR's 8% authenticated residual vs
  TMA-NM's claimed 0% against laundering. Candidate #7 from today's
  digest (MemPoison, 2607.14651) reports that write-time defenses
  "degrade sharply beyond direct corruption" — consistent with SMSR,
  not with TMA-NM. If MemPoison confirms it, the concept should record
  that origin-binding is necessary-but-not-sufficient, and the
  0%-against-laundering claim needs a scope qualifier.
- Deployment note if we ever gate our own memory writes: the paper's
  real operational lesson is that **the signing oracle must be the only
  write path**. Any route into the store that bypasses it voids the
  entire construction — which is [[concepts/permission-gate-as-architecture]]
  restated as a security requirement.
