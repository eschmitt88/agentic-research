---
kind: paper
title: "Correct Now, Insufficient Later: Auditing Update Sufficiency in Context Compression"
authors: ["Guangzhe Zhang"]
institutions: ["Independent AI Researcher (unaffiliated)"]
year: 2026
venue: "arXiv (cs.LG)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.20045"
code_url: null
citations: null
source: "raw/papers/zhang2026correct.pdf"
added: "2026-09-22"
relevance: 3
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/hce-evaluation]]"
tags: ["memory", "context-compression", "eviction", "temporal-validity", "tombstone", "metamorphic-testing", "identification-bounds", "partial-identification", "synthetic-benchmark", "pilot", "negative-result", "single-author", "reproducibility"]
---

# Correct Now, Insufficient Later: Auditing Update Sufficiency in Context Compression

## TL;DR

A single-author pilot on a **synthetic symbolic event log** (CCA v4, 24
matched history pairs) that isolates one real failure: two histories that
give the *same* answer to the current query can require *different* answers
after the *same* future update, so current-answer accuracy cannot certify a
compressed memory. The clean demonstration is an ablation — DeepSeek
frontier and latest-only both score **96/96 on the current branch**, but
reveal accuracy falls **96/96 → 32/96** when older live records are dropped;
on GLM latest-only is *better* on current (84/96 vs 78/96) and far worse on
reveal (23/96 vs 82/96). The paper then dismantles its own headline: the
winning selector's late-reference success is partly a **lexical naming
shortcut** (8/8 → **94/320** under identifier renaming), and the
label-equivariant repair that removes the shortcut retains only **2/8** of
the original late-reference answers. Its "confidence intervals" are not
confidence intervals — the reported `[0.521, 0.542]` is a
**finite-sample identification bound whose entire width is one unresolved
block (1/48)**, carrying no sampling-error information at all. Scale is
4 pairs per mechanism, one synthetic grammar, development data, no held-out
or natural-task validation, and the paper says so repeatedly.

## Claims

- **Current-answer agreement is an inadequate stand-alone test of a
  compressed memory.** Formally: a pair `(h0, h1)` is *currently equivalent*
  when `A(h0,q) = A(h1,q)` and *future-distinguishable* when some update `u`
  makes `A(h0⊕u,q) ≠ A(h1⊕u,q)`. Sufficiency is defined relative to a
  **declared** query and update family — explicitly "not a promise to answer
  arbitrary future questions from bounded storage."
- **Collision obstruction (Observation 1).** If two currently-equivalent
  histories compress to identical states with identical side information and
  a shared future needs different answers, no deterministic updater/reader
  is correct on both; if the compressed-state *distributions* are identical
  and the variant is drawn uniformly, even a randomized downstream reader has
  mean correctness ≤ 1/2. The author calls this "elementary… a diagnostic
  motivation, not a new general information-theoretic theorem," and notes the
  converse fails: distinct memory strings can differ only in useless wording.
- **The current/update dissociation is empirically real on these histories**
  — but "this is not a claim that static quality and updateability are
  statistically independent."
- **Score gaps between a deterministic selector and a model writer are
  end-to-end pipeline contrasts, not content-selection effects.** The
  schematic is `arm → (content, delivery) → m → r → S_q(r)`; holding the
  backend fixed "does not equalize writer reliability, output length, or
  reasoning consumption."
- **A high score can be an artifact of the generator's naming convention.**
  Hence the metamorphic contract: rename every identifier and key before
  interpreting success as robust selection.
- **Removing the shortcut does not solve the problem.** "Eliminating a naming
  shortcut does not solve unknown future relevance" — the repaired selector
  is provably label-equivariant *and worse* on the original instances.
- **Arbitrary late-query limit (Observation 3).** `N` independently chosen
  values over alphabet `d`, any index queryable, no side channel ⇒ a
  deterministic memory with ≤ 2^b states needs `b ≥ N log2 d`. Again
  disclaimed as "not a tight bound for our JSON packer, nor an impossibility
  claim about likely queries or archive-assisted systems."
- **What is explicitly *not* claimed:** "a robust general compressor,
  external-benchmark superiority, an equivalent H/off writing policy, a
  repaired model result, or a pure semantic effect of frontier versus
  equally reliable structured writing."

## Methods

- **Environment.** CCA v4, an append-only symbolic event log with unique
  identifiers: `set` / `copy` / `choose` records, `revoke` (permanent, incl.
  future replay), inclusive expiry against a clock equal to max retained
  `tick.now`. Per key, resolution takes the greatest-revision non-revoked,
  unexpired record. Dangling dependency of a valid record returns `null`
  rather than falling back to an older version.
- **Dataset.** Manifest `cca4-matched-prefix-v1`, split **dev**: **24 pairs**
  (4 each for rollback, expiry, choice switch, late reference, tombstone
  replay, mixed alias/expiry), 48 variants, **32 distractor records** per
  prefix, shuffled order. The writer is **target-aware but future-hidden** —
  it gets the query keys and public semantics, not the future text, gold
  answer, variant label, or mechanism label.
- **Two futures per prefix, two chunks each** (meaningful operation, then a
  distractor). The **reveal** future exposes the hidden distinction (revoke
  the latest value, advance time, switch a selector, introduce a
  previously-irrelevant reference, replay an invalidated record); the
  **override** future installs a new common highest-revision answer. "This is
  not a long sequence of independent reversals."
- **12 conditions.** Eight strict-memory (`tail`, `type_pin`, `frontier`,
  `frontier_latest`, `frontier_no_tombstone`, `prose_high`, `prose_off`,
  `structured_high`) + four archive references (full history, BM25,
  dependency-closure, direct-H prose). Strict methods see only the charged
  preceding memory plus the new chunk; **archive methods may re-inspect
  accumulated raw history** — flagged as "an explicit additional capability,
  not free bounded persistence."
- **Budget: 1,200 serialized UTF-8 bytes** including identifiers, controls,
  JSON syntax and ordering — "The cap is not 1,200 tokens." Whole-record
  packers skip records that do not fit; `full` is uncapped.
- **Backends.** Exported profiles request `deepseek-flash` and `glm-5.2` via
  Chat Completions; 4,096-token writer/reader allowance, 600 s timeout.
  DeepSeek H enables thinking at high effort, GLM H enables thinking with no
  explicit effort setting, so "equal policy labels… do not establish
  cross-vendor compute equivalence."
- **Acceptance policy is consequential and asymmetric.** An over-cap memory is
  replaced by the empty string; an incomplete-but-in-cap memory is retained and
  flagged; **no rollback to last-valid and no retry-until-success**. Strict
  reader success requires a stop-finished `{"values": {key: string_or_null}}`
  with exactly the required keys — wrong structure, wrong value, or truncation
  all score zero; infrastructure missingness stays NA.
- **Endpoint.** `J = ∏_{v∈{0,1}} ∏_{b∈{reveal,override}} Y` — success requires
  retaining the distinction *and* accepting the replacing update, for both
  variants. 48 pair-repeat blocks within **24 source-pair clusters**; repeats
  averaged within pair before averaging pairs. Current performance is reported
  separately and does not filter eligibility.
- **Uncertainty (see Critique).** Unresolved constituents give a block bounds
  `[0,1]`; arm bounds are averaged block lower/upper; the contrast is
  `[L_f − U_b, U_f − L_b]`. A separate pair-resampling bootstrap (2,000 draws,
  seed 20260911) resamples *within* each of the six fixed families.
- **Metamorphic audit.** 40 same-length random ASCII renamings and 40 prefix
  permutations per pair, values/revisions/topology/order/budget unchanged,
  full-history answers re-verified after every transformation. **13,440**
  stage-level equivariance comparisons for the repaired selector, **zero**
  violations — with the caveat that "their count is not a statistical sample
  size" and "a proof about labels is not a proof of update sufficiency."
- **Three evidence layers, never merged:** P (the paid pilot), D
  (retrospective diagnostics on the frozen artifacts), R (new offline repair
  tests). "Neither D nor R is an independent confirmation of P."

## Results

### The dissociation (the paper's one solid finding)

Strict scores, 96 planned reads per branch, 48 joint blocks (Table 1 / A1 / A2):

| Condition | DS current | DS reveal | DS joint | GLM current | GLM reveal | GLM joint |
|---|---|---|---|---|---|---|
| Frontier | 96/96 | 96/96 | 45/48 | 78/96 | 82/96 | 28/48 |
| Latest-only | 96/96 | **32/96** | 15/48 | **84/96** | **23/96** | 5/48 |
| No tombstones | 95/96 | 79/96 | 38/48 | 82/96 | 65/96 | 20/48 |
| Structured H | 76/96 | 62/96* | 19/48* | 64/96 | 56/96 | 13/48* |
| Full archive | 96/96 | 95/96 | 46/48 | 83/96 | 75/96 | 23/48 |
| Closure archive | 93/96 | 96/96 | 46/48 | 80/96 | 76/96 | 24/48 |

(`*` = one unresolved outcome/block, not an added success.) The dissociation
is sharpest on GLM, where **current-answer accuracy actively prefers the less
update-capable memory**: latest-only 84/96 current but 23/96 reveal, against
frontier's 78/96 and 82/96. Prose writers are at the floor throughout —
DeepSeek prose H reaches **1/48** joint, prose off **1/48**, GLM **0/48** for
both; `tail` and `type_pin` score **0/96 on the override branch** and 0/48
joint on both backends.

### Ablations localize to the mechanism they target — and only there

- **Tombstones.** Under `no_tombstones`, strict reveal success in the
  tombstone-replay stratum is **0/16 on both backends**, versus **16/16
  (DS)** and **15/16 (GLM)** for frontier: forgotten revocations let invalid
  records reappear. But the paper refuses the headline: "It would nevertheless
  be inaccurate to say the entire no-tombstone arm collapses" — overall reveal
  is 79/96 and 65/96, and every other mechanism is 12–16/16.
- **Latest-only** still works on choice-switch (16/16 DS) and *original*
  late-reference (16/16 DS). Its early memories collide exactly in **12 of 24
  pairs**, which the paper notes does *not* explain all its failures: losing a
  dependency invalidates an answer even when the strings still differ.
- The paper pre-empts the obvious inflation: each family has four source
  pairs, so "sixteen repeated/variant outcomes are not sixteen independent
  demonstrations."

### The record-level audit is the most useful contribution

Partitioning all 96 reveal records per arm into NA / unparsed / wrong-log /
correct-and-pass / wrapper-only / other-reader (Table 3, mutually exclusive):

- **Structured H writes valid-but-semantically-wrong logs**: **26** on
  DeepSeek, **25** on GLM. Its failures are *not* mainly missing or malformed
  output.
- **All 96 frontier reveal memories are adequate under the exact
  interpreter on both backends.** The 14 strict GLM frontier failures are
  **wrapper-only** — correct bare value map, missing required envelope.
- Applying the diagnostic bare-map rule uniformly moves **GLM frontier reveal
  82/96 → 96/96** and structured H **56/96 → 63/96**; joint frontier becomes
  **48/48 (DS) and 45/48 (GLM)**. So the apparent GLM content deficit is
  largely an answer-contract artifact.
- Counter-discipline in the other direction too: of GLM's **16 unparsed**
  structured memories, **7 still yield strict-correct answers**, so parser
  failure must not be read as semantic loss.

### The self-demolition: a naming shortcut, and a repair that makes things worse

- Original frontier breaks priority ties **lexicographically by identifier**.
  Under 40 consistent same-length renamings per pair, late-reference retained-
  state adequacy falls **8/8 → 94/320**. Every other mechanism stays
  **320/320**. A previously-irrelevant record survives only when residual
  capacity *and* the lexical tie-break happen to favor it.
- `frontier-alpha` removes lexical tie-breaks (Proposition 2: label
  equivariance, verified over 13,440 stage comparisons with zero violations).
  Its late-reference adequacy: **2/8** at identity, **80/320** renamed (exactly
  equivariant), **78/320** under prefix permutations — while original frontier
  holds **320/320** under permutations only because its lexical priority is
  untouched.
- Budget sweep on the 8 late-reference variants: alpha resolves **2/8 at 600,
  900, 1,200 and 1,800 bytes, 4/8 at 2,400, 8/8 at 4,800** — i.e. it needs 4×
  the pilot cap to match, and original frontier is 8/8 at every cap.
- **Closure archive resolves all late-reference answers** under the
  transformations, "because it can access raw history after the new reference
  appears; that result relies on a different storage capability."
- The author's own reading: "Correcting a shortcut is not evidence of a
  stronger compressor… An invariant algorithm can be consistently inadequate;
  a high-scoring algorithm can exploit an incidental generator convention."

### Delivery is a live competing explanation

Raw-empty writer returns: DeepSeek prose H **115/480**, direct-H archive
**157/288**; GLM **85/480** and **47/288**; structured writers **28** and
**19**. "Thus the pilot has not structurally eliminated empty-summary
failures." The H-vs-off joint contrast is **zero** only because DeepSeek has
one passing joint block in each prose arm and GLM none — reveal counts differ
(15 vs 14 on DS; **11 vs 3** on GLM), and the paper states outright that "its
zero joint contrast is not an equivalence result."

### Provenance accounting

All **3,456** planned outcome records present per backend (DS 1 NA, GLM 3);
independent rescoring reproduces every score; **4,608** deterministic exported
memories match byte-for-byte on replay; a separately implemented interpreter
verifies every gold answer. Token receipts: DeepSeek 4,597,572 in /
4,196,900 out; GLM 4,412,518 in / 5,623,634 out — with no currency-cost or
compute-parity claim. The archive is noted as **lacking** live SQLite ledgers,
run-lock manifests, some raw writer texts, and deployed-source attestation.

## Critique / open questions

- **"Identification intervals, explicitly not confidence intervals" — and the
  distinction is bigger than it sounds.** These are **partial-identification
  bounds over the unresolved outcomes inside a fixed sample**. Each of the 48
  joint blocks is a product of four binaries: an observed zero identifies
  `J = 0`; a block containing an NA with no observed zero could be anything, so
  it contributes `[0, 1]`. The arm bound is the average of block lower/upper
  values, and the contrast is `[L_f − U_b, U_f − L_b]`. **No missing-at-random
  assumption is made and no population null is tested.** Concretely, the
  headline DeepSeek joint interval reconstructs exactly: frontier 45/48 =
  0.9375; structured H has 19 known successes and one unresolved block, so
  `[19/48, 20/48]`; the contrast is `[0.9375 − 0.416667, 0.9375 − 0.395833] =
  [0.520833, 0.541667]`. **Its entire width, 0.0208, is 1/48 — one ambiguous
  block.** The GLM interval `[0.291667, 0.312500]` is likewise 1/48 wide, and
  the GLM *reveal* interval is `[0.270833, 0.270833]` — **width zero**, because
  nothing was unresolved. A width-zero "interval" is the tell: this quantity
  carries *no* sampling-error information whatsoever. It says only "even under
  the worst assignment of the missing outcomes, the observed gap on these 24
  pairs is X." It licenses **no** inference to new histories. The quantity that
  resembles sampling uncertainty is the separate bootstrap envelope — and it is
  much wider and sometimes nearly uninformative: DeepSeek joint
  `[0.395833, 0.646354]`, **GLM joint `[0.104167, 0.479167]`** — computed from
  four pairs per family and labelled "an approximate conditional endpoint
  envelope, not a calibrated simultaneous confidence set."
- **The abstract does *not* overstate — this is the unusual case.** It
  pre-declares the pilot scale, the identification-interval caveat, the
  negative repair result, and "no independent held-out or natural-task
  validation is claimed." The body adds caveats but never retracts an abstract
  claim. The one place a careless reader will over-extract is
  "*a deterministic frontier selector obtains strict reveal accuracy of 96/96
  on DeepSeek and 82/96 on GLM*," which reads as a superiority result; the body
  says (a) the contrast with the structured writer bundles content, delivery
  and answer-schema effects, (b) the GLM deficit is a wrapper artifact
  (82 → 96/96 under the bare-map rule), and (c) frontier does **not** beat raw
  history — DeepSeek joint 45/48 for frontier versus **46/48** for both full
  and closure archive. Quote the ablation, never the ranking.
- **The generator and the winning selector share a grammar.** "Its selector
  understands that grammar directly," so interpreter-level success says nothing
  about natural conversations, repositories, or partially observed
  environments. The tombstone ablation is close to tautological: deleting the
  persistence rule the mechanism was built around fails that mechanism.
- **Scale.** 24 pairs, 4 per mechanism, two repeats, two update chunks, one
  synthetic grammar, development data reused for the diagnostics *and* the
  repair. "Format sensitivity and selector repair were developed after seeing
  the pilot." The leave-one-family-out check (joint lower endpoint stays
  positive, min 0.45 DS / 0.20 GLM) is explicitly "reuse of development data,
  not replication."
- **Model comparison is not a model comparison.** Returned aliases are checked
  against allowlists, but "returned aliases are not immutable weights"; GLM and
  DeepSeek differ in thinking-effort configuration; no compute parity.
- **The artifact has hashes but no address.** Public-data SHA-256
  `ef7984bc…`, gold `4bb8f233…`, pilot archive `d64e202c…`, plus a
  `python analysis/run_all.py` reproduction path — but **no repository, DOI or
  download URL appears anywhere in the paper**, so none of it is actually
  obtainable by a reader. Reproducibility is asserted, not accessible.
- **AI assistance "extended beyond language editing"** — to research design,
  analysis code, artifact auditing, the diagnostics and the repair — and the
  paper states that "automated tests and multiple author-side revision passes
  are not independent peer review." Taking that disclosure at face value, the
  usual "another mind checked this" prior is absent.
- **Open question the paper poses and does not answer:** whether the
  matched-current / shared-future audit *transfers* off a synthetic grammar.
  Its own §8 confirmation contract says increasing the unchanged generator to
  192 pairs would not do it — "new seeds alone supply new instances of an old
  mechanism" — and that independently authored mechanisms plus source-separated
  natural or executable tasks are required.

## Trust signals

- **Credibility:** 2 — single unaffiliated author ("Independent AI
  Researcher"), arXiv preprint with no peer review, **no resolvable code or
  artifact URL** (only SHA-256 hashes), no citations, 24 synthetic
  development-set pairs, and AI assistance disclosed beyond language editing;
  offset upward from 1 by unusually strong claim discipline — complete
  3,456-record grids with fixed denominators, byte-identical replay of 4,608
  deterministic memories, an independently implemented gold interpreter,
  refusal to drop failed writes (citing post-treatment selection), and every
  headline number reconciling internally on check (the identification bounds
  recompute exactly from Table A2).

## Follow-up

- **Relevance:** 3 — squarely on an active theme
  ([[concepts/context-eviction-policy]], [[concepts/lossless-context-offload]])
  and it supplies a genuinely importable *evaluation design* (matched-current
  pair + shared future; the adequacy / delivery / schema partition), but at 4
  pairs per synthetic mechanism, with its own headline selector shown to ride a
  naming shortcut, it cannot anchor or move a concept. Cite-worthy, not
  load-bearing.
- **The digest's framing is half right.** [[concepts/constraint-pinning]]
  already carries this idea — its "Presence is not sufficiency" section has
  `chen2026fresh` (freshness-only executor acted on an obsolete plan in 30/30
  workflows) and `hu2026memory` (stale pinned values). That concept does *not*
  assume sufficiency away. [[concepts/context-eviction-policy]] is the better
  target: its criterion of merit is future-dependency preservation
  (`hao2026selfgc`), and this paper adds the missing *evaluation* move —
  hold the current answer fixed and vary the future.
- **Strongest importable idea, if it ever gets scale:** the three-way
  diagnostic partition. "Memory was wrong" is routinely conflated with "the
  reader mis-formatted": here 26/25 valid-but-wrong logs versus 14 wrapper-only
  failures, and a formatting rule change that moves 82/96 → 96/96 without
  touching content. Any future memory eval in this project should score
  retained-state adequacy separately from response delivery.
- **Second importable idea:** metamorphic renaming as a cheap overfitting
  probe. 8/8 → 94/320 under identifier renaming is the kind of check that costs
  nothing and invalidates a headline — a natural companion to
  [[concepts/compression-as-generalization-test]].
- **Digest candidates from its bibliography, none present in this repo:**
  MEMAUDIT (Bhargava & Barrento 2026, arXiv 2605.02199 — the paper's own
  "closest neighbor," an exact package-oracle protocol for budgeted memory
  writing); The Compaction Cliff (Zerhoudi, Mitrovic & Granitzer 2026, arXiv
  2608.22752 — repeated-compression retention); TRACE (Min et al. 2026, arXiv
  2608.06503 — closed-loop compression-boundary evaluation); ACON (Kang et al.
  2025, arXiv 2510.00615) and The Complexity Trap (Lindenbauer et al. 2025,
  arXiv 2508.21433 — observation masking matches LLM summarization);
  LongMemEval-V2 (Wu et al. 2026, arXiv 2605.12493).
