---
kind: paper
title: "Signed but Not Safe: Requirement Integrity in Multi-Agent LLM Handoffs"
authors: ["Xiaobo Chen", "Genliang Zhu", "Ziyuan Wang", "Chu Wang"]
institutions: ["University of Massachusetts Amherst", "Accentrust", "Georgia Institute of Technology", "Duke University", "University of Illinois Urbana-Champaign"]
year: 2026
venue: "arXiv 2611.07483 (cs.CR), v1 2026-11-05, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.07483"
code_url: null   # "available from the authors on request"
citations: null
source: "raw/papers/chen2026signed.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["handoff", "requirement-integrity", "signed-manifest", "hmac", "provenance-filter", "verbatim-quote", "checkable-predicate", "deterministic-verifier", "llm-checker", "same-trajectory", "fail-closed", "abstention", "extraction-fidelity", "oracle-manifest", "pre-signing-injection", "trust-boundary", "agent-in-the-middle", "prompt-injection", "preregistered", "open-weight", "qwen3", "gpt-oss"]
---

# Signed but Not Safe: Requirement Integrity in Multi-Agent LLM Handoffs

## TL;DR

An orchestrator reads a user request and hands requirements to two branch
workers and a merger. The paper compares six handoff forms on synthetic,
exactly scorable tasks under five threats. The forms run from free text
(S) and structured JSON (J) to a **signed requirement manifest** (MS). In
MS the orchestrator emits each mandatory requirement as a verbatim
`source_quote` plus a predicate in a tiny grammar. Code drops any quote
that is not a substring of the request, HMAC-signs the canonical JSON and
deterministically checks the merged decision, failing closed after one
repair. Four results matter here:

1. **Under attack, signing plus code checking works, and it works by
   abstaining.** Qwen3-8B: unsafe commits under T2a–T4 fell from 69.4% (J)
   to 9.0% (MS), a paired 60.4 pp [51.4, 69.4]. Part of that is the
   by-construction case (T2a). Excluding it, 58.4% vs 13.5%. Conformant
   commits did not rise. The gain is detection plus abstention, not better
   decisions.
2. **Structure alone did nothing; checking did the benign work; signing
   did only the tampering work.** J ≈ S without an adversary (56.2% vs
   62.5% unsafe at T0). The unsigned checked manifest (M) gives the benign
   gain. Signing adds 79.2 pp only against in-transit mutation.
3. **A deterministic verifier beat an LLM checker on the same
   trajectories.** It flagged 78.7% vs 59.0% of violating first decisions
   (n = 61), with far fewer false alarms (−42.9 pp, 28 pairs). The checker
   was Qwen3-8B with thinking off, so this is a weak baseline.
4. **The residual is extraction, and stronger models live only there.**
   An oracle manifest removed every remaining unsafe commit (11.2 pp). One
   adversarial line appended to the request before extraction pushed
   signed-manifest violations to 37.5% (Qwen3-8B), 63.9% (gpt-oss-20b) and
   75.0% (GPT-5.6-luna, 12 tasks). Every one traced to a corrupted manifest
   that the signature then authenticated. For gpt-oss-20b and GPT-5.6-luna,
   structured JSON was already safe without an adversary, and the manifest
   arms were slightly *worse* (extraction errors enforced as rules).

## Claims

- Two mechanisms give the same symptom, a decision that violates a user
  requirement. **Loss** (benign summarisation, relay compression, merger
  neglect) calls for explicit, checkable requirements. **Tampering**
  (agent-in-the-middle, tool-output injection, merger override) calls for
  integrity protection. Each defence alone is fooled by the other failure.
- Signing guarantees transit, not content. "Signing protects requirements
  in transit, not at extraction; the extraction step is the trust
  boundary." Deployments should extract from authenticated user input only,
  or have the user (or a second extraction) confirm the manifest before it
  is signed.
- The decisive check should be code, not a model. An LLM checker deciding
  (JL) cost about as much as MS and committed more violations under attack.
- Verbatim provenance is necessary but not sufficient. It blocked only 10
  of 149 extracted items. Most residual errors were genuine quotes with
  the wrong mandatory status. Provenance also cannot stop injected text
  that is itself verbatim in what the orchestrator received.
- The cryptography is standard and claimed as no contribution. The
  contribution is separating loss from tampering and pricing each defence.

## Methods

- **Pipeline.** Request q → orchestrator (only agent that reads q) →
  handoff → [relay, where T1/T2a act] → receipt check (manifest arms) →
  worker A, worker B (T3 injected into B) → merger (T4 injected) → check
  (none / LLM / verifier) → at most one repair → commit or abstain.
  Harness code is trusted and holds the HMAC key, which never appears in
  any prompt.
- **Arms.** S free text, no check. J `{requirements:[{id,text,settings_mentioned}], preferences}`,
  no check. JL = J plus a deciding LLM checker. M = unsigned manifest plus
  deterministic verifier. MS = M plus HMAC-SHA256, verified at receipt,
  plus a *non-deciding* shadow LLM checker on the first final decision.
  MSO = MS built from gold requirements (exploratory oracle). Every prompt
  starts with a byte-identical format reference (JSON Schemas plus worked
  example, App. D). Arms differ only in handoff schema and check.
- **Manifest construction (code).** Provenance filter (whitespace-normalised
  substring of q). Predicate grammar `true | eq | le | sum_le | not | and |
  or | implies` over integer settings. Normalise IDs. Bind task ID and
  nonce. RFC 8785-style canonicalisation, then HMAC. Empty manifest =
  workflow failure.
- **Threats.** T0 benign. T1 LLM relay compresses to ≤120 words, keeping
  format. T2a scripted rewrite of the target requirement in transit
  (weaken to advice, relax threshold, drop condition, remove), applied
  equivalently in every arm, HMAC left stale. T3 a "policy lookup" tool
  result to worker B claiming the requester revised the target. T4 a
  "business priorities" tool result telling the merger the target is
  advisory. T5 (follow-up) one fixed adversarial line appended to the
  request before extraction (withdraw, rewrite, or omit the target).
- **Tasks.** Generator with six lineages (chain, budget, exception, cover,
  match, scoped). Each task has three integer settings, two mandatory
  requirements (verbatim sentences), preferences built so that losing or
  mutating the target makes the preference-optimal decision violate it, a
  withdrawn-draft decoy, and six background sentences. The target always
  spans both branches. Confirmatory bank 48 tasks (6 × 8, three fictional
  enterprise domains), disjoint 6-task dev bank.
- **Models.** Confirmatory: Qwen3-8B (MLX 4-bit, thinking disabled), 6 arms
  × 5 threats × 48 = 1,440 roots. Second family gpt-oss-20b (reasoning low)
  at a reduced cut: J/JL/M/MS × T0/T2a/T3 × 36 tasks = 432 roots, no own
  pilot. It replaced the preregistered Llama-3.1-8B (amendment v1.5). The
  Llama family was deferred. Exploratory: GPT-5.6-luna through a stateless
  Codex CLI, J/M/MS × T0/T2a/T3 × 12 tasks = 108 roots. Temperature 0,
  seed 0, exact-request cache.
- **Endpoints and tests.** Primary: *unsafe commit* (committed, violates ≥1
  gold requirement). Guardrail: *conformant commit*. Paired exact McNemar,
  Holm over H1/H2/H3a. Task-clustered bootstrap 95% CIs (B = 10,000).
  H3b is a TOST at ±5 pp. H2 enters Holm only with ≥60 evaluable pairs
  (preregistered). Freeze trail with SHA-256 manifests (v1 2026-10-17 to
  stage 2 2026-10-21). A gold-free pilot gates launch (the first pilot
  failed and prompted v1.3).

## Results

- **Qwen3-8B outcomes (Table 2, % of 48 roots).**
  - T0 unsafe: S 62.5, J 56.2, JL 18.8, M 14.6, MS 14.6, MSO 0.0.
  - T0 conformant: S 37.5, J 43.8, JL 35.4, M 47.9, MS 43.8, MSO 60.4.
  - MS abstained in 39.6% of T0 runs. 64.6% of M's *first* decisions at
    T0 violated gold, so the benign gain is the repair/abstain loop.
  - By threat, J vs MS unsafe: T2a 91.7 vs 0.0, T3 64.6 vs 14.6, T4 52.1
    vs 12.5. MS T3/T4 equal its T0 rate, so the injections added nothing
    past the verifier.
- **Preregistered contrasts (Table 3).**
  - H1 J−MS unsafe, T2a–T4: 60.4 pp [51.4, 69.4], 144 pairs, Holm
    p < 0.001.
  - H2 verifier − shadow LLM detection on violating first decisions
    (T3/T4): 19.7 pp [6.7, 34.4], 61 pairs (one over the threshold).
  - H3a M−MS unsafe under T2a: 79.2 pp [66.7, 89.6]. The unsigned
    verifier faithfully enforced the forged predicate.
  - H3b M−MS conformant at T0: 4.2 pp, 90% CI [−4.2, 12.5], **not shown
    equivalent** at ±5.
  - S1 M−J conformant (T0/T1): 1.0 [−16.7, 18.8]. S2 MS−J conformant, all
    threats: −0.4 [−14.2, 13.3], which is "no detected difference", not
    equivalence. S3 JL−MS unsafe: 20.1 [11.1, 29.9]. S4 MS−MSO: 11.2
    [4.6, 19.6]. S6 false alarms: −42.9 [−71.4, −14.3], 28 pairs (abstract:
    7% vs 50%).
- **Brittleness (S5).** Under the T1 relay MS rejected 2.1% of manifests
  and kept 43.8% conformant. 16.7% of unsigned manifests failed the
  schema receipt check. The relay prompt asked to keep the schema, so a
  harsher compressor would cost more. MS turned tampering into refusal:
  97.9% abstention under T2a.
- **Extraction fidelity (Table 4, T0, n = 47).** Recall 85.1%, precision
  70.4%, target recalled 80.9%. The provenance filter dropped 10 of 149
  items (7 invalid predicates, 3 non-verbatim quotes). 54 of 139 kept items
  were preferences encoded as mandatory. That causes avoidable abstentions
  and enforces non-requirements.
- **Cost (Table 5, all threats).** Tokens per root: S 8,532, J 8,961, JL
  13,489, M 10,569, MS 11,215. That is 1.18× (M), 1.25× (MS) and 1.51× (JL)
  over J. At T0 (Discussion): M 1.22×, MS 1.51× including the shadow
  checker, JL 1.52×. The verifier itself costs microseconds.
- **gpt-oss-20b (Tables 6–7).**
  - H1 36.1 pp [27.8, 43.1], **entirely from T2a**. J unsafe 2.8% at T0
    and 0.0% at T3. M/MS 11.1% at T0, slightly worse, from enforced
    extraction errors.
  - H3a 50.0 pp. H2 untestable (4 pairs). H3b not equivalent.
  - S2 utility −28.7 pp [−39.8, −17.6]. 8 of 36 tasks (22.2%) produced no
    usable manifest. In 5, all items were dropped. In 3, a `<|constrain|>`
    harmony header broke parsing, and that would have failed the Qwen
    pilot's I2 criterion.
  - Usable manifests: recall 87.5%, precision 93.4%, target 78.6%. The
    deciding LLM checker (JL) did not stop T2a: 83.3%, same as J.
- **Pooled shared cells.** H1 52.8 pp [45.8, 59.7]. H3a 63.9 pp. Utility
  MS−J −13.9 pp [−23.6, −4.2]. The authors: "the benign-condition and
  injection benefits of deterministic checking appear only for the weaker
  model".
- **Pre-signing injection T5 (Table 8 and §6.6).**
  - Qwen3-8B, T0 → T5 unsafe: J 56.2 → 79.2, M 14.6 → 41.7, MS 14.6 →
    37.5, S 62.5 → 83.3. H5b (MS) 22.9 pp [12.5, 35.4], Holm p = 0.002.
    The target was missing or altered in 62.5% of signed manifests, and all
    18 MS unsafe commits had a corrupted manifest. H5a (signing adds
    nothing): 4.2 pp, 90% CI [−2.1, 10.4], not shown equivalent at ±10.
    MS still beat J (37.5 vs 79.2), because the verifier caught violations
    of the untouched requirement and abstained (41.7%).
  - gpt-oss-20b: MS 11.1 → 63.9 (52.8 pp [30.6, 72.2]), J 2.8 → 72.2. 100%
    of manifests were corrupted, and so were all 23 MS unsafe commits.
  - GPT-5.6-luna (exploratory): J 0 → 75.0, MS 8.3 → 75.0. Every manifest
    was corrupted.
  - T0 baselines come from the earlier runs, not interleaved.
- **GPT-5.6-luna, T0/T2a/T3 (Table 11, 12 tasks).** J was 100% conformant
  at T0 and T3. Under T2a it followed the forged requirement (J 58.3%, M
  75.0%), while MS abstained 12/12. M/MS carried one enforced extraction
  error (8.3%) at T0 and T3.

## Critique / open questions

- **The headline effect is mostly a small-model effect plus a
  by-construction case.** The 69.4 → 9.0 headline pools T2a, where
  rejection is guaranteed (the authors say so). The benign and injection
  gains of checking hold for Qwen3-8B without thinking. For gpt-oss-20b and
  the frontier subset, J was already near-perfect without tampering, and
  the manifest cost utility (−28.7 pp; pooled −13.9). The abstract's "no
  net loss of conformant commits" is Qwen-only, and its CI is ±14 pp.
- **H2 bounds the verifier by extraction and pits it against a weak
  judge.** The verifier checks the *manifest*, not gold, so its 78.7% is
  capped by extraction recall. Its "false alarms" are mostly preferences
  extracted as mandatory items. The checker is Qwen3-8B with thinking off
  and a frozen prompt. The paper calls a stronger checker likely to narrow
  H2, and there is no evaluable H2 for gpt-oss-20b (4 pairs) or the
  frontier model.
- **Toy requirement form.** Three integer settings, two requirements, a
  predicate grammar in which gold is written (the circularity the authors
  flag). The promised secondary analysis, a verifier restricted to
  provenance and coverage checks without predicates ("to show how much
  depends on executability"), is announced in §8 and **never reported**.
  That is the analysis that would speak to prose requirements, which is
  the box's case.
- **Audit-trail gaps.** Table 9's freeze trail ends at v1.3/stage 2
  (2026-10-21). Appendix A's disclosed deviations list v1.1–v1.3 only. The
  v1.5 (gpt-oss swap, cut 4 → 5) and v1.7/v1.8 (T5) amendments are cited
  in the text but have no hash row, so their pre-run status rests on the
  text's word. **Table 7 is mislabelled.** It is captioned "pooled Qwen3 +
  gpt-oss-20b" with n = 36 per row, but its numbers (J T0 97.2% conformant,
  MS T2a 77.8% abstain, M/MS T0 11.1% unsafe) are the gpt-oss-20b-only
  results quoted in the §6.5 text.
- **Possible high-throughput study pipeline.** The manifest schema is named
  `p920-manifest-v1`. The text mentions "the authors' Paper 44 v5.1
  transport", "copied from Paper 34", and launch rules "simplified for all
  of the authors' studies". AI coding and writing assistance is disclosed.
  None of this invalidates the design, but it is a reason to weigh the
  unreleased code more heavily.
- **What the box cannot read off.** No Claude model and no prose-only
  requirements. Small models without reasoning carry every confirmatory
  claim. The decoy (a withdrawn-draft sentence) is in every task, but its
  extraction rate is not reported, although it is the closest analogue to
  a withdrawn chat instruction. The T5 injection is one fixed line at the
  end of the request, non-optimised. T5 relaxes the "honest orchestrator"
  assumption only through its input. A dishonest or compromised
  orchestrator is out of scope.
- **Fit with the graph.** The same two open families (Qwen3-8B,
  gpt-oss-20b) as [[literature/papers/chu2026when]]. That paper found
  self-written handoffs *add* policy. This one finds that the
  orchestrator's extraction *mis-classifies* it (preferences as
  mandatory), so both locate the error at the writer of the handoff. It
  agrees with the cited Wang et al. 2026a ("Facts without rules") that
  typed JSON alone does not beat free text. It also gives
  [[concepts/typed-enforcement]]'s open question on prose→policy
  misreading a second measured extraction error rate, beside
  [[literature/papers/li2026who]]'s 77/81 compiler match. The AP2 lesson it
  cites (Louck et al. 2026, Aviv et al. 2026) is by the author of
  [[literature/papers/louck2026securing]], whose origin-bound authority is
  the memory-side counterpart of "extract only from authenticated input".

## Trust signals

- **Credibility:** 3.
  - For: preregistered with SHA-256 freeze manifests and a gold-free pilot
    gate (which failed once and was reported). Holm family, paired tests
    and task-clustered CIs. Nulls reported as nulls (H3b, H5a not
    equivalent). An oracle arm and a same-trajectory shadow checker. A
    second vendor family that partly *contradicts* the primary one, given
    equal space. Four universities plus an industry co-affiliation
    (Accentrust).
  - Against: no released code or data ("on request"). Not peer reviewed,
    12 days old, no citations. Synthetic toy tasks. 8B/20B models without
    reasoning. The second family ran at a reduced cut with no pilot.
    Amendment hashes are missing for v1.5/v1.7/v1.8, Table 7 is
    mislabelled, and one announced secondary analysis is absent.
  - Not 4: no code, no venue, and the contradicting second family limits
    generalisation. Not 2: the design is unusually disciplined and
    transparent, and the key mechanism claim (every T5 violation traces to
    a corrupted manifest) holds in all three families.

## Follow-up

- **Relevance:** 4. Material new evidence for three existing concepts.
  For [[concepts/typed-enforcement]], it gives the first same-trajectory
  deterministic-vs-LLM checker comparison in this graph and a measured
  prose→predicate extraction error. For [[concepts/hierarchical-delegation]],
  it is a direct test of guidance 2 ("free-text handoffs drift;
  field-indexed handoffs do not"), which it does not support. For
  [[concepts/intent-level-briefing]], it places the residual risk at the
  brief-writer's extraction, mainly mis-graded mandatory status. Not 5: an
  enterprise security framing, toy tasks, no Claude-class model, and the
  benign benefit disappears for stronger models.
- **For this box (our inference).** The box's delegations are not under
  attack in the T2a–T4 sense, so the measured lesson is the T0 and T5 one.
  The brief-writer's reading of the source is where requirements go wrong,
  and a check downstream of that reading cannot recover them. A cheap
  analogue for curation briefs and `/implement` handoffs: hard constraints
  carried as verbatim quotes from a named source, which a script checks
  for presence in that source (the provenance filter costs nothing). The
  quote should also be marked "must" vs "prefer", because that grading is
  where 54 of 139 items failed. Untested here on prose or on Opus-class
  writers.
- **Candidates** (cited, not yet in the graph):
  - Wang et al. 2026a, "Facts without rules: boundary metadata collapse in
    multi-agent LLM handoffs". Typed JSON did not beat free text, and a
    corrupted allowlist reintroduced leakage.
  - Li & Wang 2026, "Governance at the boundary: how agent decomposition
    degrades policy compliance". Attenuation 0% single loop → 85%
    orchestrator–subagent with a 32B model.
  - Xu et al. 2026, "MasDrift". Re-anchoring each call to the original
    request reduces unauthorized actions.
  - Yamafuji Lun et al. 2026, "MIRROR: multipath quorum integrity for LLM
    multi-agent communication". An LLM judge cost 35× tokens and blocked up
    to 44.2% of benign outputs.
  - Louck, Dvir & Stulman 2026, "Signing the transaction but not the
    decision: Whisper attacks and a binding defense for AP2".
  - Zhiqi Wang et al. 2026b, "Lost in context compaction: when LLM systems
    drop side constraints". A separate constraint channel restores them.
