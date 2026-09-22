---
kind: paper
title: "Authorization Revocation for Long-Running AI Agents: Root-Scoped Quiescence under Delegation and Asynchronous Execution"
authors: ["Genliang Zhu", "Chu Wang"]
institutions: ["Accentrust (Vancouver, Canada)", "Georgia Institute of Technology", "University of Illinois Urbana-Champaign"]
year: 2026
venue: "arXiv 2609.21284v1 (cs.PL; CCS concepts: Access control / Distributed computing models), 18 Sep 2026"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.21284"
code_url: null
citations: null
source: "raw/papers/zhu2026authorization.pdf"
added: "2026-09-22"
relevance: 3
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hierarchical-delegation]]"
tags: ["safety", "revocation", "shutdown", "authorization", "capabilities", "distributed-systems", "formal-methods", "protocol", "three-valued-verdict", "design-paper", "no-released-code"]
---

# Authorization Revocation for Long-Running AI Agents: Root-Scoped Quiescence under Delegation and Asynchronous Execution

## TL;DR

A 39-page cs.PL protocol paper arguing that "stop the agent" is an
authorization question, not a process question. Cancellation
acknowledgement, token revocation and process exit each close one local
object while durable queue messages, derived credentials, cron triggers,
reservations and prepared provider effects keep the retiring authority
able to commit. The paper defines **root-scoped authorization quiescence**
— a certificate that, within a declared manifest, no protected sink will
accept a commitment whose selected authorization still contains the
retired root-epoch atom — and gives a protocol (durable epoch cut →
issuance freeze → per-sink fences → carrier closure or atomic rebind →
exact cross-provider channel-token accounting → signed leaf certificates
composed into a cutset). **What it actually contains: 3 lemmas + 6
theorems with hand proofs in a 5-page appendix, all conditional on an
explicit 11-premise table (A1–A9 safety, A10–A11 liveness); a
provider-free reference implementation (file-backed hash-linked ledger,
forked child-process race harness, separate trace checker); and a
self-authored 17-case conformance suite.** Its headline empirical
sentences — 17/17 registered outcomes, 17/17 separate-checker agreement,
44/44 rehashed regressions rejected — are an **artifact gate over a mock,
not a measurement of anything in the world**. No network request, no
credential, no MCP/A2A/OAuth/queue call is ever made; effects are "inert
typed records." **No code is released anywhere in the paper.** The one
causal result worth carrying is internal and small: the root cut alone
does *not* stop an already scheduled old-epoch effect — only the
sink-local fence does.

## What the paper contains — answered directly

Because the digest flagged "no evaluation is visible in the abstract,"
this is the first thing to settle.

- **(i) Proofs: yes, real ones, conditional.** Nine numbered results
  (Lemmas 1–3, Theorems 1–6) with proofs in Appendix A (pp. 33–37).
  Lemma 1 cut issuer non-expansion; Lemma 2 support-sound projection;
  Lemma 3 no root laundering by rebind; Theorem 1 compositional
  no-false-quiescence; Theorem 2 noncompositionality + opacity boundary;
  Theorem 3 independent-support preservation; Theorem 4 merge-order
  independence (⊔ associative/commutative/idempotent); Theorem 5
  crash/replay stability and unrelated-root locality; Theorem 6
  conditional convergence. They are **pen-and-paper proofs over a
  hand-defined transition system, not machine-checked** — no Coq, no TLA+,
  no Lean (contrast [[literature/papers/louck2026securing]], whose
  malleability theorem is TLA+-checked). Every safety result is stated
  "under A1–A9," and Table 2 spells those premises out at unusual length.
- **(ii) Implementation: yes, but provider-free and unreleased.** §6.1: a
  file-backed ledger with canonical JSON, domain-separated SHA-256
  digests, a contiguous hash-linked event log, write–fsync–rename–
  directory-fsync persistence, exclusive file locks with reload-under-lock.
  §6.2 forks "a real child process" as a cancellation-race worker. §7.2
  adds a **separately implemented checker** that "neither imports nor
  calls the ledger, runner, worker, or corpus" and recomputes every hash
  chain and digest from its own oracle. That separation is genuine
  methodological hygiene. But §6.1 also states: "This is a provider-free
  executable semantics: no network request, cloud resource, credential,
  payment, or production effect is performed. A protected effect is an
  inert typed record whose acceptance is decided at the local sink." And
  §8.4: "Endpoint names in its 17-case corpus are semantic categories, not
  assertions of runtime or remote-provider conformance."
- **(iii) Empirical evaluation: a 17-case self-registered conformance
  suite, and nothing else.** No baseline against a deployed framework, no
  incident corpus, no cost or latency figure ("Performance is descriptive
  and does not serve as evidence for safety," §7.5). §7.3 is explicit that
  process exit, token revocation, subtree cascade and graph-targeted
  revocation "are analyzed through their stated semantics and the formal
  counterexamples in Sections 2 and 4; they are not reported as measured
  remote-service baselines."

So: **not a pure position paper** — there are proofs and there is running
code. But it is also **not an evaluated system**. It sits in a third
category the ingest rubric doesn't have a slot for: a formal protocol with
a self-authored regression suite standing in for evaluation.

## Claims

- **"Stopped" is an authorization predicate, not a process predicate.**
  The decision object is "whether one authorization root can still cause a
  new protected commit after a particular durable cut" (§2). Table 1
  enumerates what each adjacent mechanism establishes and what it leaves
  open: cancellation ack establishes "a task endpoint accepted or attempted
  a cancellation-state transition"; process exit, "one observed process
  terminated"; token revocation, "a named token is no longer accepted under
  the issuer's revocation semantics."
- **The cut is not the fence.** The durable root-epoch transition "is
  atomic at the authority ledger, not across all providers" (§5.1). An
  already authorized carrier can legitimately reach a sink after the cut;
  only that sink's own monotone barrier can refuse it. This is stated as
  design *and* demonstrated in the harness (below).
- **Blind subtree revocation is wrong, in both directions.** §1.1's
  counterexample: a shared worker supported by roots r_A and r_B. Deleting
  it over-revokes valid r_B work; keeping it without changing the selected
  support "could launder r_A's authority through the shared principal."
  The repair is an **antichain of minimal sufficient root-support
  witnesses** per live instance plus an **atomic rebind receipt** to a
  witness excluding the retired atom.
- **Local closure does not compose.** §2.4: provider P reports an empty
  outbound queue immediately before sending; provider Q reports an empty
  inbound queue immediately before accepting. "Both local snapshots can be
  individually true while the composed system still contains a live
  cross-provider obligation." Hence exact typed SEND/terminal-ACK channel
  tokens and the conservation requirement π_t(O(E)) = T_c = π_t(B(E)).
- **Three-valued verdict, and no promotion by timeout.** Quiescent /
  Not-Quiescent / Indeterminate. "Neither of the other verdicts is
  converted to success by timeout, retry exhaustion, majority vote, or an
  agent's assertion that it has stopped" (§3.5). N has priority over U so
  a known counterexample survives missing evidence.
- **An impossibility boundary (Theorem 2(2)).** An endpoint exposing
  "neither complete enumeration nor an enforceable fence/frontier" cannot
  contribute a positive leaf: worlds w0 (no residual carrier) and w1 (one
  delayed callback holding the retired atom) are observationally identical
  outside that endpoint, so "any deterministic or randomized verifier
  receiving the same observation has the same output distribution in the
  two worlds. If it returns Q, it is unsound in w1; if it never returns Q,
  it is incomplete in w0." The sound verdict is U.
- **The certificate is deliberately narrow.** §10.1 keeps four nearby
  statements distinct: it is *not* global idleness, *not* business
  completion, *not* rollback of pre-fence commits, *not* universal physical
  cessation.

## Methods

- **Formal model (§4).** Root-epoch atom a = ⟨r, e, h_g⟩ where h_g digests
  the authenticated grant; atoms are explicitly "not bearer strings."
  Machine state S = ⟨A, C, X, T, F, K, L, M, P⟩ (root ledger, cut ledger,
  carrier registry, channel-token ledger, fence map, provider
  commitment/frontier state, event log, manifests, adapter profiles).
  Cross-domain reasoning uses only a causal ≺, "never by comparing
  implementation-specific sequence numbers" — a deliberate refusal to
  compare heterogeneous provider clocks.
- **Carriers.** "Any durable or live object that can preserve, transmit, or
  exercise authority: a task, child process, delegated credential, session,
  queue record, timer, cron entry, webhook, reservation, retry record, or
  provider operation." This enumeration is the paper's most quotable
  contribution and does not depend on any of its machinery.
- **Protocol (§5), six steps.** (1) BeginShutdown durably retires a_c and
  advances to epoch e+1, binding cut id, manifest version, policy and
  adapter-profile digests, and the frozen token frontier T_c. (2) Project
  actual old-root dependence. (3) Install adapter-specific barriers.
  (4) Conserve local and transferred carriers. (5) Emit provider-frontier
  leaves binding root, cut, coverage manifest, configuration, fence and log
  watermarks, rebind receipts, local-open set, and outbound/accepted tokens.
  (6) Compose the cutset; the combiner accepts only when the manifest is
  complete, leaves agree on the cut, nothing is unknown or conflicting, and
  channel tokens balance exactly.
- **Assumption table (Table 2, A1–A11).** A1 issuer mediation and
  non-expansion; A2 authentic canonical evidence; A3 durable root
  linearization ("The root transaction does not itself install a provider
  fence"); A4 support-provenance completeness; A5 per-sink fence and
  frontier soundness; A6 registry/manifest completeness; A7 exact channel
  accounting; A8 atomic rebind; A9 crash-safe monotonicity. A10–A11
  (eventual evidence stabilization, finite drain and fair delivery) are
  used only for the liveness theorem — "safety remains fail-closed during a
  partition or permanent provider failure."
- **Adversary model (§3.3).** Controls the agent's planning and all
  non-trusted descendants; may build deep delegation chains, pick among
  roots, move work between endpoints, schedule delayed callbacks, retain
  renewable credentials, race cancellation against commitment, delay,
  duplicate, reorder, replay, crash a coordinator at any instruction
  boundary, present conflicting inventories, and "attempt to relabel
  root-dependent work as independently authorized." Cannot forge
  signatures/digests, rewrite a committed ledger record, violate fence or
  rebind atomicity, or commit outside every mediation point. Compromise of
  a trusted commit gateway or an effect path missing from the manifest is
  **outside the theorem domain** and handled "operationally by profile
  admission and manifest audit."
- **Correspondence contracts (§6.3, Table 4).** Per-endpoint obligations
  for MCP async tasks, A2A delegated tasks, credential issuers, durable
  queues/triggers, and effect reservation gateways — each with a required
  barrier/evidence column and a nonpositive condition. **These are
  specifications for adapters that do not exist.** §6.5: "A profile cannot
  treat the union of unrelated documents as one implementation."
- **Test suite (§7.2, Table 5), 17 cases.** Cancellation-ack only (2),
  root cut without sink fence (1), root cut + sink fence (2), restart and
  stale-process interleaving (2), alternate-root rebind (1), opaque
  endpoint / missing fence (2), typed pre-fence commitments (1), open
  obligation / in-flight token (2), exact retry / body conflict / remint
  (1), conjunctive witness enforcement (1), ghost leaf token (1), terminal
  channel reopen (1). Causal isolation: in all three arms the worker
  reports STARTED before any intervention, reports EFFECT_READY, then
  enqueues CANCEL_ACK before emitting the same old-atom effect, so the
  protected unit is scheduled before the arms diverge.
- **Adversarial checker suite (§7.4).** 44 semantically modified and
  **consistently rehashed** traces — mutations are resealed under the
  receiving case, and several splice well-formed fields from a *different*
  registered case, so rejection must come from semantic reconstruction
  rather than a stale outer hash.

## Results

All numbers below are from Table 6 ("Fresh-run evidence for the executable
17-case layer") and §8, and are outcomes of the authors' own registered
oracle. Every one is a pass/pass count; there is no measured rate, no
baseline system, and no cost.

- **Causal isolation — the one result with an internal control.** Both
  cancellation-only cases accepted the late effect (**2/2**). The cut-only
  case "prevented new authority expansion but accepted the already
  scheduled effect because no sink barrier existed" (**1/1**). Both
  cut-plus-fence cases rejected the identical effect (**2/2**), as did the
  restart case (**1/1**) and the stale-process case (**1/1**) — four
  mechanism rejections. Since the three arms share the same scheduled unit
  and the same protected-effect oracle, "the comparison therefore
  attributes rejection to the sink-local fence, rather than to cancellation
  timing, process exit, or post-cut work creation."
- **Verdict totals: Quiescent 6, Not-Quiescent 3, Indeterminate 2** (11 of
  the 17; the remaining 6 are negative-operation cases registered as
  `rejected` or `replay safe`, not root verdicts). "No nonpositive case
  carried a root-A quiescence certificate." The two insufficient-evidence
  cases stayed Indeterminate; the open obligation, in-flight channel and
  accepted cut-only effect stayed Not-Quiescent.
- **Runner 17/17, separate checker 17/17, and 44/44 rehashed regressions
  rejected.** The checker "reproduced the runner's result-set digest," and
  "agreement is therefore a distinct implementation check, not a second
  invocation of the controller's verdict function."
- **Alternate-root rebind (1/1).** Support antichain {{A},{B}} with an
  exact selected root-A witness; the execution advances root B to epoch 2,
  then atomically instantiates the alternative support as the **exact
  current root-B epoch-2 atom** before root-A certification. The
  conjunctive case "rejected an obligation selected under {A, B} when it
  presented only B."
- **Dual pre-fence accounting (2/2 admissions receipted).** One effect
  admitted before the cut and still outstanding at the cut, one admitted
  between cut and barrier; the case reaches Quiescent only after both
  post-barrier typed receipts exist, each binding cut, obligation,
  operation, effect, sink, exact selected atoms, admission sequence and
  barrier sequence.
- **Evidence-bounded positive leaves 11/11.** Each positive leaf binds an
  `evidenceThroughSequence` plus a digest of the endpoint-local snapshot,
  "so a later record cannot retroactively satisfy an earlier positive leaf."
- **Certificate identity 1/1 each.** Exact (requestId, candidateDigest)
  retry returns the identical committed certificate; same-ID/different-body
  is a request conflict; different-ID is a remint. Both rejected.
- **Artifact gate.** "17/17 registered runner outcomes, 17/17 separate
  checker decisions, rejection of all 44/44 semantically rehashed
  regressions, zero false certificates, and zero registered post-certificate
  old-root crossings."

## Critique / open questions

- **The empirical layer is a regression suite the authors wrote, ran, and
  graded, over a system they built to satisfy it.** 17 cases with
  *registered expected outcomes* and a 17/17 match is a conformance gate,
  not evidence about agents, frameworks, or providers. Compare what this
  graph already holds on the same theme:
  [[literature/papers/shen2026revoked]] ran 49,121 decision-model calls
  across five real memory stores and nine models to get 43.1%;
  [[literature/papers/bouras2026authority]] ran 75×2 paired runs against a
  real injected effect to isolate 33/75 vs 3/75;
  [[literature/papers/santosgrueiro2026lingering]] ran a
  revocation-ablated comparator to get 6/6 vs 0. This paper's comparable
  number is 2/2-accept vs 2/2-reject over inert records in a mock. The
  direction is certainly right; the sample is three cases per arm.
- **No code. Anywhere.** The entire empirical section rests on an artifact
  that is not linked, not deposited, not anonymized-for-review. There is no
  artifact-availability statement in 39 pages. The only repository URL in
  the paper is reference [24], someone else's spec. Every digest, hash
  chain, and "separately implemented checker" claim is therefore
  unverifiable by anyone outside the two authors — and the separate-checker
  design, which is the strongest methodological move in the paper, is
  exactly the claim that most needs independent inspection. This is the
  single biggest reason the credibility score is 2.
- **The abstract does *not* overstate — §8.4 is stricter than §1.** This is
  unusual enough to record. The abstract's last sentence pre-empts the
  obvious misreading ("not global idleness, rollback, or business
  completion"); §7.3 refuses to report unexecuted mechanisms as baselines;
  §8.4 "Claim discipline" states "a one-to-one mapping between every
  empirical sentence and the system that produced its receipt"; §6.1 says
  in plain words that no network request or credential is involved. The
  risk with this paper is not an inflated abstract, it is a reader who
  takes "17/17" and "44/44" for measurements because they are formatted
  like measurements. **They are self-graded conformance counts over a
  provider-free mock.**
- **Every theorem is conditional on premises that are exactly the hard
  part.** A5 requires every manifested effect sink to serialize admission
  behind a durable monotone barrier that "rejects every later local attempt
  selected under the retired atom" and enumerates its complete cut-relevant
  frontier. A6 requires the frozen manifest to cover *every* runtime,
  provider, sink and channel class reachable from work selected under the
  retired atom. A7 requires every cross-endpoint handoff to carry a
  globally unique retry-stable token with exactly one authenticated
  terminal disposition. A deployment that could satisfy A5–A7 across MCP,
  A2A, OAuth, a queue and a payments provider has already solved most of
  the problem; the protocol composes that capability, it does not create
  it. The paper is honest that missing capability yields Indeterminate
  rather than failure — but an all-Indeterminate deployment is what the
  premises predict for essentially every system shipping today.
- **Table 4's correspondence contracts are the most load-bearing unbuilt
  component.** Not one adapter is implemented. "A deployment claim requires
  concrete adapters to satisfy the correspondence contracts in Section 6;
  no such claim is inferred from this artifact." So the paper's whole
  cross-provider thesis — its actual novelty over
  [[literature/papers/santosgrueiro2026lingering]] and
  [[literature/papers/zheng2026continuity]] — is untested by construction.
- **Proofs are by hand, over a model the same authors defined.** There is
  no mechanization and no external model-checking. Theorem 1's proof is a
  case analysis over Definition 5's verdict priority; it is plausible, and
  it is also the kind of argument that TLA+ exists to catch errors in. The
  paper's own strongest relative — louck2026securing — machine-checked its
  central negative result.
- **The 44 mutations are also self-designed.** "Rejects 44/44" bounds the
  checker against the failure modes its authors thought of and then
  implemented. It says nothing about a mutation class they did not
  enumerate. The resealing discipline (mutations rehashed consistently,
  cross-case field splices) is genuinely better than a naive tamper test,
  and still self-scoped.
- **Unverifiable provenance.** "Accentrust" has no visible track record;
  the affiliations are dual-listed with Georgia Tech and UIUC but name no
  lab, group, or advisor, and there is no acknowledgements or funding
  section. The reference list leans heavily on very recent single-author
  arXiv preprints in the same narrow niche (three by Santos-Grueiro, one
  RISU technical note, one RISU GitHub spec), only two of which this graph
  has independently ingested.
- **Liveness is conceded, correctly.** §10.5: "A permanent partition or
  permanently opaque endpoint can therefore prevent a certificate while
  preserving safety." Combined with Theorem 2(2), the honest operational
  reading is that a real heterogeneous deployment will frequently sit at
  Indeterminate and the policy question — what to *do* with a
  never-certifying shutdown — is named ("policy may escalate or contain")
  and not answered. That is the same open axis
  [[literature/papers/elkoussy2026agentltl]] opened for violation response.

## Trust signals

- **Credibility:** 2. Two authors at a company with no visible track
  record, dual-affiliated to Georgia Tech and UIUC without naming a lab;
  arXiv preprint, not peer reviewed; citations unknown; **no released
  code or artifact**, which matters more here than usual because the
  artifact *is* the entire empirical section and cannot be inspected;
  hand proofs rather than machine-checked ones; an evaluation that is a
  self-authored, self-registered 17-case conformance suite over a
  provider-free mock with inert effect records and zero implemented
  adapters. Held *at* 2 rather than 1 by real countervailing discipline:
  an explicit 11-premise assumption table, a separately implemented
  checker that imports none of the system under test, adversarial
  mutations that are consistently rehashed and cross-case-spliced, an
  impossibility result the authors prove *against* their own protocol's
  reach, and a scope section (§10.1) that enumerates four things the
  certificate deliberately does not mean. The writing is scrupulous; the
  evidence is unavailable.

## Follow-up

- **Relevance:** 3, and at the low end of 3. Useful prior art on an active
  theme — it is the most complete formal statement in this graph of what
  "revoked" has to mean once work has crossed a provider boundary — and it
  is cite-worthy for two design rules below. It is **not** a 4: it shifts
  no concept's evidence base, because everything it demonstrates is
  demonstrated on a mock the authors built to demonstrate it, and the
  cross-provider composition that is its actual novelty is untested by
  construction. It is not a 2 because the cut-vs-fence isolation and the
  three-valued verdict discipline are directly importable design rules
  touching two load-bearing concepts, and the carrier enumeration is
  reusable prose independent of the machinery. The domain fit is also
  weak for this project specifically: research agents on this box are
  single-operator, single-box, with no delegated credentials, no
  multi-provider reservations, and no payments gateway. The threat model
  this protocol is built for is a multi-tenant one.
- **What this paper *is* the constructive counterpart to — and what it is
  not.** The digest framed it as "the constructive counterpart to
  shen2026revoked's 43.1% measurement and bouras2026authority's
  propagation result." That is **directionally right and specifically
  overstated.** [[literature/papers/shen2026revoked]] is about
  *memory-record* revocation — a contradicted fact still ranked first at
  retrieval — and its fix is retrieval-time validity filtering. This paper
  is about *authorization-root* revocation across asynchronous carriers,
  and never touches retrieval or memory. Different objects, different
  layers; the shared word is "revocation." The genuinely adjacent in-graph
  papers are [[literature/papers/santosgrueiro2026lingering]] (which this
  paper cites as [28] and positions itself *after*: lingering authority
  scopes a capability's lifetime within one principal; this scopes a root
  cut across endpoints) and [[literature/papers/zheng2026continuity]]
  (cited as [39]; both argue locally valid controls do not compose). The
  bouras2026authority link is real but oblique — that paper is about
  authority granularity *across principals during* a task, not retirement.
  The "cs.PL, so it is the kind of source typed-enforcement is built from"
  half is right in genre and wrong in evidence grade: typed-enforcement's
  existing sources mostly ship either a measured violation rate or a
  released artifact, and this has neither.
- **[[concepts/enforcement-boundary-placement]]** — the strongest edit, and
  the only one backed by an internal control. The concept already holds
  that the gate belongs "at the transaction that applies the effect." This
  paper adds a **temporal** version of the same rule with a clean
  three-arm isolation: retiring the authority at the *issuer* (the root
  cut) prevents new authority from being created but does **not** stop an
  already scheduled effect — the cut-only arm accepted the identical late
  effect that the cut-plus-fence arm rejected, and both restart and
  stale-process arms also rejected it. The importable sentence is *the
  issuance freeze and the admission barrier are two different enforcement
  points and you need both*, with the corollary that the retirement
  decision cannot be atomic across providers (§5.1: "atomic at the
  authority ledger, not across all providers"). Caveat to record with the
  edit: 1 case vs 2 cases, inert typed records, one process.
- **[[concepts/permission-gate-as-architecture]]** — two additions.
  (a) **The gate needs a third verdict.** Q / N / U with an explicit rule
  that neither nonpositive verdict "is converted to success by timeout,
  retry exhaustion, majority vote, or an agent's assertion that it has
  stopped," and with N prioritized over U so a known counterexample
  survives evidence loss. Every gate in this concept is currently binary;
  "I could not establish this" is not the same output as "this is
  forbidden," and conflating them is exactly how a partition becomes a
  release. (b) **Revocation must rebind, not delete.** The minimal-sufficient-
  support antichain plus atomic rebind receipt is a mechanism the concept
  does not have: it preserves work that has an independent current witness
  while refusing "root laundering" through a shared principal, which is the
  precise failure a subtree cascade creates. This sits directly beside the
  concept's existing santosgrueiro2026lingering (lifetime within a
  principal) and bouras2026authority (granularity across principals)
  sections as a third axis: *what happens to shared work when one grantor
  withdraws*.
- **[[concepts/typed-enforcement]]** — a smaller, honest addition. This
  belongs in the concept's **"upper bound: what a deterministic checker
  cannot enforce at all"** section, next to
  [[literature/papers/ray2026what]]. Theorem 2(2) is a second
  information-theoretic boundary of the same family: where ray2026what
  bounds the enforceable class by prefix-recognizability and
  irreversibility, this bounds it by **observability** — an endpoint
  exposing no sound query, fence, bounded expiry, or terminal receipt makes
  two worlds (residual callback vs none) observationally identical, so no
  verifier over the remaining evidence is both sound and complete, and the
  only sound output is "unknown." The design consequence is the same shape
  as ray2026what's: the checker is only ever as strong as its oracle
  interface. Worth adding *with* the caveat that this theorem is hand-proved
  and unreleased, where ray2026what's is the paper's central machinery.
- **[[concepts/evidence-gated-completion]]** — the certificate object is a
  worked example of the concept's own criterion: a signed, digest-bound,
  replay-safe artifact that a verifier checks *without calling the
  controller's decision procedure*, with issuance keyed to
  (requestId, candidateDigest) so an exact retry is idempotent and a remint
  is rejected. The evidence-bounded leaf (`evidenceThroughSequence` + local
  snapshot digest, "so a later record cannot retroactively satisfy an
  earlier positive leaf") is a reusable trick for any completion receipt
  this project might define. Design-grade evidence only.
- **[[concepts/hierarchical-delegation]]** — the carrier taxonomy is the
  reusable piece and needs none of the protocol: "a task, child process,
  delegated credential, session, queue record, timer, cron entry, webhook,
  reservation, retry record, or provider operation." That is the checklist
  for *what survives* when a delegating agent stops, and it is more complete
  than anything currently in that concept.
- **Cited work worth a future digest.** Eleven of the 39 references are
  2026 arXiv preprints on agent authorization that this graph does not
  hold. Ranked:
  1. **Khan, "Stop Means Stop: Measuring and Repairing the Enforcement Gap
     in Agent-Framework Control Primitives"** (arXiv:2607.14166, cs.SE,
     v3) — described here as *measuring* cancellation orphans, timeout
     zombies, replayed execution and sibling leakage in real agent
     frameworks and evaluating an external effect gate with fence-on-cancel.
     That is the empirical paper this one is the formal counterpart to, and
     it is the higher-value ingest of the two.
  2. **Santos-Grueiro, "When Does Authorization End? Effect Closure at
     Provider Boundaries"** (arXiv:2609.02866, cs.CR) — the provider-local
     effect-closure model (future-use / instance / lineage closure, explicit
     effect frontier) that this protocol consumes as a leaf. Same author as
     the in-graph santosgrueiro2026lingering.
  3. **Santos-Grueiro, "Temporary Authority, Permanent Effects: Commit-Time
     Authorization for LLM Agents"** (arXiv:2607.10487, cs.CR) — evaluates
     a fail-closed monitor on protected commit surfaces; the sink-adapter
     obligation this paper assumes. Directly feeds
     enforcement-boundary-placement.
  4. **Choi et al., "ResidualAuth: What Authorization State Must Language
     Agents Preserve under Revocable Delegation?"** (arXiv:2609.08062,
     cs.AI) and **Liu et al., "VERA: Authority-Preserving Edge Revocation
     for Federated AI-Agent Workflows"** (arXiv:2608.30091, cs.AI) — the
     exact-revocation pair behind the antichain/rebind design.
  5. **Wu et al., "The Authorization-Execution Gap Is a Major Safety and
     Security Problem in Open-World Agents"** (arXiv:2605.11003, cs.CR) —
     a framing/position paper naming delegation incompleteness, channel
     corruption and composition fragmentation; likely a 2 on its own but
     useful as the umbrella citation for this cluster.
  Lower priority: AID-Guard (2608.21159), AIRGuard (2605.28914), EBL-Core
  (2609.11596), Bounded Agents (2608.15888), Earned Authority (2607.23586),
  heartbeat-bound credentials (2605.20704). Note also that the
  RISU "Bounded Agent Closure" work is cited as a Zenodo technical note
  plus a frozen GitHub spec rather than a paper — not a digest candidate.
