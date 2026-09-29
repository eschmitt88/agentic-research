---
kind: paper
title: "Scope Before You Persist: Preventing Cross-Family Interference in Agent Memory"
authors: ["Yezhou Cheng", "Runjia Du", "Zeming Liu", "Qibai Chen", "Hang Lyu", "Yilan Wei", "Yankai Zeng", "Bojun Lin"]
institutions: ["Independent (6 of 8 authors)", "Northwestern University", "Pinterest, Inc."]
year: 2026
venue: "arXiv (cs.AI); header reads \"Author-prepared manuscript; not a record of conference acceptance\""
peer_reviewed: false
url: "https://arxiv.org/abs/2609.29144"
code_url: null      # code, manifests, prompts and traces are said to be "included with the supplement"; no public URL given
citations: null
source: "raw/papers/cheng2026scope.pdf"
added: "2026-09-29"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/information-firewall]]"
tags: ["persistent-memory", "skill-editing", "scoped-retrieval", "certification-scope", "cross-family-interference", "admission-gate", "paired-design", "prespecified", "self-improvement", "frozen-model", "code-repair", "single-slot-memory", "toy-benchmark"]
---

# Scope Before You Persist: Preventing Cross-Family Interference in Agent Memory

## TL;DR

A frozen model (Qwen3-Coder-Next) edits its own 360-character skill after
each round of a 12-round code-repair stream over nine recurring task
families. A paired, execution-grounded gate (ORC) admits or rejects each
edit. The gate's admissions are locally sound, but deployed as the one
global skill they damage other families, so the gated agent ends *below*
never updating (0.713 vs 0.775 mean hidden trajectory utility). Storing
each admitted edit only under the family whose evidence certified it fixes
this: 0.816 when replaying the same eight edits, and +0.063 [0.037, 0.094]
across 27 prespecified paired streams, with 63 vs 12 admissions and
harmful/admitted 0/63 vs 6/12. The slogan: "certification determines
whether an edit is supported, while retrieval scope determines where that
evidence authorizes its use."

## Claims

- **Persistent memory takes two decisions**: whether an update is
  supported, and where it applies. Evidence from one family can justify the
  first without justifying global deployment.
- **Cross-family interference** is the named mechanism. A "locally valid
  edit" deployed through "one compact global skill" changes families
  outside the gate's evidence.
- **Gate-drift proposition (Prop. 1).** The expected deployed change is
  positive iff παG > (1 − π)βL. For an AND gate over conditionally
  independent channels, false acceptance is Πβ_j. If the channels'
  false-accept events are identical, adding them leaves β unchanged. The
  paper's own scope note: it "only covers outcomes represented in the gate
  evidence; it cannot protect an unscoped global edit from interactions
  with task families that have not yet appeared."
- **Scope matching supports repeated adaptation.** Scoped memory keeps
  admitting useful edits, while global memory stops after its first
  admission in most streams. The authors decline to claim *compounding*,
  since that "additionally requires a cumulative one-shot comparator".

## Methods

- **ProcStream-RSI v3.1.** A 12-round code-repair stream with the fixed
  family schedule boundary, rotation, ordering, normalization, boundary,
  rotation, nested, missing, prefix, chunk, tie-breaking, ordering (nine
  families, repeats so retention is visible). Each round has 4 discovery
  and 4 probe tasks. Fixed banks hold one checkpoint and two final tasks
  per family, 123 tasks per stream. Seeds vary tokens, names, program
  variants and cases, but "oracle semantics and family order are fixed
  across all streams". Each family gets a "seed-derived opaque protocol
  token".
- **Memory.** One natural-language skill K_t, padded to 360 characters, is
  the only inherited state. An accepted edit *replaces* it. The editor is
  the same frozen model. Nothing is trained. Qwen3-Coder-Next runs on a
  pinned Parasail BF16 endpoint at temperature 0.
- **ORC gate.** It compares incumbent and candidate on the same probe
  instances over three channels: 4 public examples, 4 private exact-output
  tests absent from the actor/editor prompt, and ≥6 answer-free
  metamorphic checks. It takes a one-sided 95% paired-bootstrap lower bound
  per (family-group × channel). A candidate deploys only if the current
  family gains on private tests (L > 0) and no channel regresses by more
  than 0.02 on the current family or any historical family. Evidence roles
  are separated. Probe tasks are never summarized for the editor. Private
  and metamorphic outcomes reach only the gate. Sealed hidden cases reach
  only a model-free auditor (App. C).
- **Seven arms**: Static, Frozen-compute (same calls, never deploys),
  Latest-only (deploy everything), Self-judge, Replay (public-test gate),
  ORC, and Batch-ORC (cumulative evidence, one final deploy).
- **Scope, three ways.** (1) *Fixed-output intervention* on the 8 main
  streams: every Global-ORC lineage admitted only a round-0 boundary rule,
  so "scoped" reuses the archived ORC completion on boundary tasks and the
  archived Static completion elsewhere. Proposals, decisions and scores are
  unchanged. (2) *18-stream randomized-entry replication*, one update
  round, two streams per possible first family. (3) *27-stream full
  12-round extension*, prespecified, three streams per first family, with
  a leave-one-seed-out character 3–5-gram Naive Bayes router.
- **Stats.** The stream is the unit. Seed bootstrap, paired sign-flip
  tests, and Holm correction over three prespecified contrasts. Pilot
  results (a saturated v2 generator) and a truncated Batch-ORC run were
  archived and excluded, with a dated deviation log (App. E). Total spend
  was $6.9965.

## Results

- **The prespecified main contrasts are all null** (Table 2).
  ORC − Replay = 0.006 and ORC − Latest-only = 0.010 (Holm p = 1.0).
  ORC − Batch-ORC final = −0.068 (Holm p = 0.2812). The gate itself did
  not beat accepting every proposal.
- **Global controls all fall below Static** (Table 1, mean trajectory):
  Static 0.775, Frozen-compute 0.775 at about five times the cost,
  Latest-only 0.703 (BWT −0.253), Self-judge 0.736, Replay 0.707, ORC
  0.713. Harmful-acceptance fractions are 52.1% / 51.9% / 49.2% for the
  permissive arms. ORC admits one proposal at round 0 in every stream, when
  no historical family exists to constrain it, and "then rejects all later
  proposals". Six of those eight admissions lower the next hidden
  checkpoint.
- **The mechanism, per family.** The admitted boundary rule adds +0.398 on
  boundary but changes missing / rotation / tie-breaking / chunk by
  −0.461 / −0.234 / −0.148 / −0.117. All eight ORC admissions are safe on
  same-family gate probes, but only two improve the next global
  checkpoint. PPV goes "from 1.00 locally to 0.25 globally" (App. A).
- **Fixed-output scope intervention** (Table 4, 8 streams): scoped 0.816
  vs Global-ORC 0.713, with 0 vs 6 harmful out of 8. The paired difference
  is +0.103 [0.052, 0.151] (p = 0.03125), and +0.041 [0.022, 0.056] over
  Static.
- **18-stream replication** (Table 5): 5 shared admissions, 3 vs 0
  harmful, checkpoint 0.761 vs 0.802. The **primary effect is not
  significant**: 0.041 [−0.004, 0.107], p = 0.1875. Final bank is 0.031
  [0.001, 0.070], p = 0.125. The non-current-family change is −0.047
  (global) vs 0.000 (scoped).
- **27-stream extension** (Table 6): trajectory 0.848 vs 0.785, difference
  0.063 [0.037, 0.094], p < 10⁻⁴ (Holm < 10⁻⁴). Final checkpoint is 0.900
  vs 0.789. Admissions per stream are 2.33 vs 0.44, and streams with ≥2
  admissions are 19/27 vs 2/27. Harmful/admitted is 0/63 vs 6/12. The
  learned router is 243/243 on held-out checkpoint prompts and matches
  oracle routing (0.848). **No Static arm is reported for this
  extension.**
- **Channels.** Over 479 proposal–round comparisons, public AUROC is
  0.752, and the private/metamorphic component is 0.998. Their errors
  correlate ϕ = 0.164. At the frozen threshold, "the private component and
  full conjunction make identical predictions."
- **Transfer.** On HumanEval+ (32 tasks), Global-ORC scores 0.895 vs
  Static 0.914, and scoped is not evaluated. On GPT-OSS-120B actor-only
  sealed banks, scoped scores 0.960 vs Global 0.951 (diff 0.010
  [−0.033, 0.055]) and +0.020 [0.004, 0.035] over Static.

## Critique / open questions

- **"Global memory" here is a single-slot overwrite.** Every admitted edit
  replaces the whole 360-character skill for every family. Scoped memory
  keeps nine slots, so it stores up to nine times the text at the same
  per-prompt budget. The finding is "one overwritten slot interferes",
  not "a globally loaded rule among many interferes". Additive libraries,
  which is what `~/.claude/` skills, rules and most skill stores are, are
  untested.
- **0/63 is mostly structural.** A scoped edit changes only its own
  family's slot, so its non-current-family effect is exactly 0.000 by
  construction. "Harmful" can then only come from an in-family false
  admission, which the private channel (AUROC 0.998) nearly eliminates.
  The comparison is not like-for-like exposure: a global edit is exposed
  to all nine checkpoint families, a scoped one to one.
- **The 8-stream 0.816 is archival recomposition, not a run.** Each stream
  has one admitted rule. Applying it only to boundary tasks, with Static
  everywhere else, is close to arithmetic given the per-family deltas. The
  27-stream extension is the real test, and it holds.
- **Family identity is free here.** The benchmark gives each family an
  opaque token and templated prompts, and the router is 243/243 "in
  distribution". Wrong-slot routing is explicitly untested: the robustness
  curve models misses as fallback to K₀, not as delivery to another slot.
  Real tasks do not arrive labeled, and choosing the scope key is the
  unsolved part.
- **Scoping also forbids positive transfer.** No scoped policy is tested
  off-benchmark (HumanEval+ has no family key). This contrasts with
  `tang2026memory`'s positive cross-domain transfer on all six pairs. The
  obvious middle design is to retrieve where certified and widen on
  cross-scope evidence. The paper does not test it.
- **Why global ORC froze is not explained.** The paper says only "the
  conservative gate freezes it" (ORC admitted 1 of 12 in the pilots too).
  Whether the frozen rule then blocks later candidates through the
  historical-family check, or the thresholds are simply strict, is not
  separated.
- **Narrow world.** One model at temperature 0, nine code-contract
  families with fixed semantics and order, and a $7 study. The authors say
  "Static's 0.775 mean trajectory utility is consistent with substantial
  prior competence". The prespecified 3-contrast family came out null. The
  scope result the title rests on comes from the added intervention and
  the later prespecified extension.
- **Gate independence (Prop. 1) is textbook.** It is the
  source-independence axis of [[concepts/shared-substrate-contagion]]. The
  measured datum is that adding the weakly correlated public channel
  changed no decision.
- **AI-use statement**: generative AI "design[ed] the benchmark and
  experiments, implement[ed] and test[ed] code, draft[ed] and edit[ed] the
  manuscript". The authors take responsibility.

## Trust signals

- **Credibility:** 2. Six of eight authors list "Independent"; the others
  are Northwestern and Pinterest. The paper is an unreviewed
  author-prepared manuscript with no public code URL (a "supplement" is
  referenced), days old, with extensive AI authorship. It sits at the top
  of 2 for method hygiene: paired within-stream design, stream as the
  unit, prespecified contrasts with Holm, a dated deviation log, excluded
  pilots disclosed, a transactional cost ledger, sealed hidden cases
  audited model-free, and **null primary contrasts reported rather than
  buried**. Trust the direction of the scope effect on this benchmark.
  Trust its magnitude and generality much less.

## Follow-up

- **Relevance:** 4. It adds a principle the graph did not state:
  *admission certifies for the scope its evidence sampled, and the loading
  function must not deploy wider*. The paired design isolates it from
  proposal and gate quality. It maps directly onto this box's
  global-vs-project memory layering. Not a 5: it is a toy single-slot
  benchmark with oracle-separable families, and the result is partly true
  by construction.
- **[[concepts/skill-library-lifecycle]]**: third attestation for scoped
  loading, with a different mechanism (interference, not dilution). It
  couples the admission gate to the loading function. See the concept's
  2026-09-29 paragraph.
- **[[concepts/verified-memory-writes]]**: a non-adversarial ceiling on
  the write gate. The preservation check covers only the scopes present in
  its evidence. See the concept's new section.
- **[[concepts/selective-memory-retrieval]]**: cited only. The concept is
  about *whether* to retrieve (uncertainty-gated), and this paper is about
  *which scope* to retrieve from. It does extend the MemPoison point
  already recorded there (retrieval as a necessary defense stage) to
  benign, fully certified content, and the verified-memory-writes section
  carries that link.
- **[[concepts/information-firewall]]**: cited only. App. C's evidence-role
  split (editor sees discovery only, gate sees private/metamorphic, auditor
  alone sees sealed hidden cases) is a clean instance but adds nothing new
  to the concept.
- **Bearing on this repo.** `~/.claude/CLAUDE.md`, user-global skills and
  auto-memory are the global slot. Project `CLAUDE.md` and path-scoped
  `.claude/rules/` are family slots. A lesson validated in one project and
  promoted straight to the global file is the Global-ORC move. This is
  weak but direct support for recording *where a rule was validated* when
  promoting it (e.g. in `/elevate` proposals). It is still not the
  conditional-activation vs always-on head-to-head that
  `dai2026agentguard`'s note says the 09-20 `@import` finding lacks,
  because the global arm here overwrites rather than adds.
- **Cited work worth a digest check** (none in this graph): Wang,
  Kattakinda & Feizi 2026, "Do agent optimizers compound?" (2607.14004,
  continual evaluation on Terminal-Bench 2.0, where only a regression-aware
  optimizer transfers). Asadolahi et al. 2026, "Memory reward inflation"
  (2608.00017). Iacob et al. 2026, Red Queen Gödel Machine (2606.26294).
