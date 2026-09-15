---
kind: concept
name: "verified-memory-writes"
status: growing
added: "2026-07-07"
sources:
  - "[[literature/papers/yang2026trustmem]]"
  - "[[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]]"
  - "[[literature/papers/chhikara2025mem0]]"
  - "[[literature/papers/karamchandani2026your]]"
  - "[[literature/papers/louck2026securing]]"
  - "[[literature/papers/sharma2026smsr]]"
  - "[[literature/papers/gao2026mempoison]]"
  - "[[literature/papers/ravindran2026portable]]"
  - "[[literature/papers/zhu2026lossy]]"
  - "[[literature/papers/li2026remember]]"
  - "[[literature/papers/leong2026recognition]]"
  - "[[literature/papers/nakayashiki2026when]]"
  - "[[literature/papers/wu2026evomal]]"
  - "[[literature/papers/yu2026recursive]]"
  - "[[literature/papers/he2026stored]]"
  - "[[literature/papers/goyal2026does]]"
  - "[[literature/papers/hu2026memory]]"
  - "[[literature/papers/chen2026fresh]]"
  - "[[literature/papers/shen2026revoked]]"
  - "[[literature/papers/li2026autorecsys]]"
  - "[[literature/papers/singh2026churnbench]]"
  - "[[literature/papers/suresh2026grounding]]"
related_concepts:
  - "[[concepts/multi-granularity-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/shared-substrate-contagion]]"
related_experiments: []
tags: [memory, consolidation, write-policy, verification, trustworthiness, knowledge-organization]
---

# verified-memory-writes

## Definition

Every write to an agent's persistent memory — insert, revise, prune — is
verified as a *state transition* before it persists, judged on whether it
covers the new evidence (**coverage**), keeps valid prior memory intact
(**preservation**), and adds nothing unsupported (**faithfulness**).
Trust is enforced at consolidation time, not left to retrieval-time
filtering or terminal task feedback.

## Why it matters here

Once a flawed entry is consolidated, it is repeatedly retrieved and
amplified — persistent memory errors compound across sessions, and
terminal rewards can't localize which write introduced them (a severe
credit-assignment gap). [[literature/papers/yang2026trustmem]] shows that
scoring each transition with a verifier (and training the write policy on
those scores) cuts omission/corruption/hallucination in consolidated
memory by 40–79% while *improving* downstream utility — write-time
verification is not a tax on memory quality but a contributor to it.

The memory cluster previously covered trust only on the read side:
[[concepts/selective-memory-retrieval]] gates what gets *retrieved*;
[[concepts/multi-granularity-memory]] consolidates across tiers but
without an explicit trust gate on the consolidation step. This concept is
the missing write-side policy: what may enter the durable store, and on
what evidence.

The pattern generalizes beyond conversational memory:

- **Skill promotion** — [[concepts/skill-library-lifecycle]]'s core
  question (what gets promoted into the durable skill library, on what
  evidence) is a memory-write decision; the coverage / preservation /
  faithfulness triad is a directly reusable promotion rubric.
- **Knowledge-graph ingest** — this project's own `/ingest` discipline
  (trust-signal frontmatter, dedup against existing notes, immutable
  `raw/`) is a hand-rolled instance: verification happens at write time,
  and `raw/` immutability guarantees the evidence a write was judged
  against stays auditable.

## The read side: a recall you can check

This concept covers making a *write* trustworthy.
[[literature/papers/zhu2026lossy]] covers the symmetric gap — making a
**recall** checkable — and the graph was thin there.

TierMem keeps every compressed summary unit **linked to the raw pages it
was derived from**, in an immutable raw-log tier. A recalled fact is
therefore not merely retrieved but *traceable*: the agent can produce the
source that justified it. When a consolidated finding is written back after
an escalation, the new unit inherits pointers to the raw evidence used,
which the paper calls **maintaining the chain of custody**. That is
[[concepts/citation-anchoring]] turned inward — a claim ships with the
means to check it, where the claim is the agent's own memory rather than a
literature citation.

The measured payoff is specific and worth the detail. Ablating the
provenance links (escalation falls back to global BM25 retrieval) costs
1.5pp overall — but the loss is concentrated exactly where the mechanism is
supposed to act: accuracy on **escalated** queries falls 81.7% → 77.5%,
while the summary-only path is unchanged. Links do not change what the
memory contains; they change how reliably the decisive evidence is found
when the summary proves insufficient.

Two caveats before importing. Tier-2 immutability is a **design
convention, not an enforced property** — nothing described prevents raw-log
mutation, where atinafu2026rewardhacking's `evalhashlock` makes the
analogous claim checkable by content-addressing. And write-back
re-introduces lossiness at one remove: the consolidated unit is a new
summary chosen under a *past* query's notion of salience, so the tier
drifts even while each unit stays traceable. Recoverability is the
mitigation, not a cure.

### Trusted too much, and stale by dependency

Traceability makes a recall *checkable*; it does not make the agent
*check*. [[literature/papers/hu2026memory]] measures what happens absent
the check: agents answer with a stale stored value **0.92–1.00 of the time
at every model scale**, even when an authoritative tool holds the correct
one — over-trust rather than confusion. Worse, the harm is
**capability-gated**: larger models collapse hardest once a stale note is
dressed to look current, and a **recency cue (stale note dated newer)
fools the larger models harder**. So a write gate plus a traceable recall
still leaves the decisive failure open, because the read path prefers the
store over live evidence by default.

One finding is directly actionable here: **removing a label amplifies
over-trust at every model size.** The frontmatter discipline this project
already runs is doing safety work, not just bookkeeping. The corollary is
uncomfortable — a concept whose `added:` date was refreshed by an unrelated
edit is exactly hu's "stale dated newer" trap, and touching a file is not
revalidating it.

[[literature/papers/chen2026fresh]] supplies the missing validity notion:
freshness of *state* does not establish validity of the *plan* derived from
it, and a freshness-only executor acted on an obsolete plan in **30 of 30**
workflows. Its dependency-scoped alternative — cite the exact records used,
revalidate only those before an external effect, block when validation is
incomplete — is the principled form of a staleness check, and it is
implementable in this repo today because concept notes already cite their
sources. This repo has no staleness check on concepts at all: `kg_lint`'s
age checks are backlog timers (uncurated candidates, unengaged high-relevance
literature), not validity checks.

[[literature/papers/singh2026churnbench]] closes off the obvious alternative
proxy, age. With scheduled refresh on, freshness errors were **7 / 4 / 4** at
cache ages of 1 / 14 / 28 days, because any entity whose TTL is shorter than
the window is refreshed repeatedly and reaches evaluation within one TTL.
Switching the scheduler off raised the 28-day count to **45** and left the
1-day count at 7. Where errors landed followed **tier width × mutation
rate**. The 30-day tier was stale by construction but produced one error in
all runs, because contracts barely change. The 1-day tier produced none until
refresh was removed, then 25 of the 45. Time since a record was built is
therefore the wrong staleness variable for any maintained store. What
matters is time since last revalidation, relative to how fast what it
depends on changes. Two details carry over. The residual errors under
tiering were all **aggregates** (cost-center spend totals and rankings over
a warm-tier price). The paper argues per-fact supersession cannot reach such
objects, which have "no fact-level identity to supersede". That is the
derived-artifact gap again, after chen's stale plan and shen2026revoked's
journal. And a TTL-lapse counter read zero in both configurations for
opposite reasons, so the instrument is a per-entity last-refresh timestamp,
not an expiry count. This graph has only `added:`, a build time. Thin
evidence: one model, synthetic data, single-digit counts under tiering, and
no mutation-rate sweep.

## Connections

- Complements [[concepts/selective-memory-retrieval]]: retrieval-side
  gating catches bad memories late; write-side verification prevents them
  from persisting at all. A robust memory system likely needs both.
- [[concepts/agent-native-memory]] argues memory systems should be
  designed for agents' actual read/write workloads — this concept is the
  write-workload half made concrete.
- Second attestation, from practice
  ([[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]]):
  after an automated synthesis pass wrote garbage to a production
  vault, the author made every scheduled write **reversible** — daily
  diffs plus a 24-hour review window before changes become permanent.
  That is write-time verification implemented as *delayed commit +
  human backstop* rather than a verifier model: same principle
  (trust enforced at consolidation, not retrieval), opposite
  mechanism cost.
- **A second paper-grade source, at production scale but with a
  weaker gate.** Mem0 ([[literature/papers/chhikara2025mem0]], 60.8k
  GitHub stars, deployed) makes every write pass through an explicit
  four-way decision — ADD / UPDATE / DELETE / NOOP — chosen by an LLM
  tool-call conditioned on the top-*s* most similar existing memories.
  This confirms the *shape* of TrustMem's claim (writes should be
  gated, not appended unconditionally) at real deployment scale, but
  the gate itself is a single undifferentiated LLM judgment call, not
  a dedicated verifier scored against explicit coverage /
  preservation / faithfulness axes — i.e. evidence that *some* write
  gate helps, not evidence for TrustMem's specific verification
  design. TrustMem remains the only source with a *decomposed,
  measured* verification rubric (40–79% error reduction attributed to
  the coverage/preservation/faithfulness split specifically); Mem0
  and the practice report are corroborating but coarser. The concept
  stays `seedling` — still no independent replication of TrustMem's
  own ablation, and no source yet compares a coarse single-judgment
  gate against a decomposed one head-to-head. Adjacent evidence to
  watch: memory admission control and provenance-aware memory (cited
  in TrustMem's related work but not yet in this graph).
- **Fourth attestation — the adversarial case for the same boundary**
  ([[literature/papers/karamchandani2026your]]). FARMA forges an
  agent's *own reasoning traces* ("prior validation already complete")
  and amplifies them into false precedent, reaching 100% attack
  success against undefended agents and defeating retrieval-time and
  consensus defenses — amplification floods the store until forged
  traces *are* the consensus, so outlier detection inverts. The
  SENTINEL defense's first design principle is this concept stated as
  a security requirement: screen entries **before they are committed
  to memory**, "prevent[ing] forged entries from accumulating in the
  store and being legitimized through retrieval." Its write-path
  ablation (Reasoning Guard necessary and sufficient; 0% ASR, 0% FPR
  on 326 benign traces) is the strongest quantitative evidence yet
  that the write gate, not retrieval filtering, is where trust must
  be enforced. Prior sources motivated the gate from benign drift;
  this one shows the same boundary is the security perimeter — but
  note its gate is *structural* (provenance/format forensics on the
  entry), orthogonal to TrustMem's *semantic* triad
  (coverage/preservation/faithfulness). A production write gate
  plausibly needs both, and the paper's own adaptive-paraphrase
  result (the heuristic guard falls to an attacker who knows its
  patterns) marks the structural half as the brittle one.
- **Fifth attestation — the formal boundary on what the gate may check**
  ([[literature/papers/louck2026securing]]). A machine-checked (TLA+)
  separation theorem: any write/act authority signal derived from
  *content* or a *derivation edge* is **malleable** — laundered by the
  agent's own summarization, a trusted-tool echo, or manufactured
  corroboration — so content-scoring gates (SENTINEL's forensics, and
  in principle any LLM-judge trust score) cannot be sound against an
  adaptive attacker; FARMA's adaptive-paraphrase win over SENTINEL is
  this theorem in the wild. What holds is authority bound
  non-malleably to *origin at write time* (channel-authenticated, never
  inferred from content), elevated only via ≥2 independent trusted
  principals: TMA-NM reaches 0% attack success across 8 models at
  utility identical to undefended, where content and lineage baselines
  leak 47–84%. Sharpens this concept's internal division of labor:
  origin-binding is the *security* layer of the write gate, and
  TrustMem's semantic coverage/preservation/faithfulness triad is the
  *quality* layer — the theorem says the quality layer can never be
  asked to carry the security load. First formally-verified source in
  the cluster; artifacts released (benchmark, harness, TLA+ models).
- **Sixth attestation — an independent route to the same necessity, and
  the first honest account of what origin-binding does *not* cover**
  ([[literature/papers/sharma2026smsr]]). SMSR proves the impossibility
  half by an entirely different argument — embedding density rather than
  TLA+ model checking: because fluent text populates any neighbourhood
  of a target query's embedding, no deterministic content-based
  retrieval-time filter can carry a non-trivial worst-case certificate.
  Two independent formal routes converging on "write-time provenance is
  necessary" is a much stronger warrant than either alone, and the
  empirical companion is blunt: a keyword + perplexity + semantic-anomaly
  filter falls to **100% ASR** against fluent bypass text, while HMAC
  signing takes the same attacks to **0%**.

  The important part is the residual. SMSR's 0% covers only the
  **unsigned** adversary — one with no access to the signing oracle.
  Against the **authenticated** adversary, who writes through the normal
  path (which is what the agent itself does), signatures buy nothing and
  the defense falls back to a probabilistic bound: 8.0% ASR [5.8, 10.9]
  against a certified worst case of 10.4%, collapsing to no guarantee at
  all once the adversary's write budget approaches half the retrieval
  pool. This puts a **scope qualifier on the fifth attestation**:
  louck2026securing's laundering channels — self-summarization,
  trusted-tool echo, manufactured corroboration — all operate by making
  *the agent write the poison itself*, which is precisely SMSR's
  authenticated regime. TMA-NM claims 0% there; SMSR measures a real
  residual. The two agree that origin-binding is **necessary** and
  disagree on whether it is **sufficient**. Treat sufficiency as open
  until a third source adjudicates.

  Operationally the construction reduces to one rule that is
  [[concepts/permission-gate-as-architecture]] restated as a security
  requirement: **the signing oracle must be the only write path.** Any
  route into the store that bypasses it voids the whole construction —
  so the gate's worth is a property of the architecture around it, not
  of the gate itself.

## The write gate has a structural ceiling — necessary, not sufficient

[[literature/papers/gao2026mempoison]] settles the sufficiency question
the fifth and sixth attestations left open, and settles it against
sufficiency. Its taxonomy separates harm by *when it becomes visible*:

- **L1 — direct.** A single record is harmful on its face. This is what
  every write gate in the cluster is built for, and it works: write-time
  consistency checking takes L1 corruption from 45.4% to **4.8%**.
- **L2 — compositional.** The payload is split across several records,
  each individually plausible, that become harmful only when
  co-retrieved. Best write-time result: 22.5%.
- **L3 — dormant.** A record stays behaviorally inert until a later
  natural context activates it. Undefended this is the *most* effective
  attack (**76.7%** corruption, above L1's 45.4%), and the best
  write-time result is 27.8%.

The dilemma is structural, not a tuning failure: facing L2/L3 a
write-time filter must "either admit the locally plausible components,
thereby allowing the attack, or block benign information, destroying the
utility of the memory system." The fragments genuinely *are* benign
individually. Counterfactual analysis confirms the mechanism rather than
inferring it — L1 corruption is carried by one record (removing it
reverts 95.7% of responses), whereas L2 shows real pairwise interaction
(the pair's joint effect exceeds the sum of its parts) and L3 shows
influence that appears only under its trigger.

**Why this binds origin-binding too, not just content gates.** Every
defense MemPoison evaluates is content-based, so this is not a direct
measurement against TMA-NM or SMSR — but the argument transfers by
construction. L2/L3 payloads arrive through *legitimate, authenticated*
channels: user input, tool returns, cross-agent messages. Channel-bound
origin authority certifies precisely what is not in dispute — that a
real principal wrote this record — while the harm lives in the relation
*between* records, or in a future context. An origin-bound gate admits
L2/L3 for a different reason than a content gate does, and just as
reliably. Together with sharma2026smsr's measured 8% residual against
the authenticated writer, the resolution is:

> **Write-time origin binding is necessary but not sufficient.** TMA-NM's
> 0%-against-laundering result should be read as scoped to *single-record*
> laundering; it does not cover compositional or dormant corruption.

**The missing layer.** This concept has distinguished a *security* layer
(origin binding) from a *quality* layer (TrustMem's
coverage/preservation/faithfulness). Both are **per-record**, and
MemPoison shows any per-record gate has a ceiling. The layer neither
covers is **composition-time**: reasoning over the retrieved slate
rather than the incoming record. The evidence is direct — source-
reliability *retrieval* reweighting is the single best L2 defense
(17.0%, beating every write-time method) while being among the worst on
L1 (43.7%). Write-time and retrieval-time defenses fail on **disjoint**
tiers, which is why only a stacked defense reaches single digits overall
(10.7% at 93.8% clean accuracy). Design implication: a write gate is a
necessary first stage, not a complete architecture.

Two smaller findings worth carrying:

- **Scale does not save you.** All ten model families sit at 54.7–66.9%
  undefended corruption; GPT-5 is the *worst* at 66.9%. This is not a
  weak-model artifact that frontier models grow out of.
- **The substrate is a defense parameter.** Flat chunk storage is worst
  (67.9%), hierarchical notes intermediate (63.1%), a decomposed fact
  store best (56.6%) via dilution — see
  [[concepts/multi-granularity-memory]]. Though note the tension:
  decomposition dilutes L1 payloads while fragmentation is exactly the
  L2 attack.

## Revocation is a read-side property, and write-back launders it

The definition above puts trust at consolidation time, "not left to
retrieval-time filtering." [[literature/papers/shen2026revoked]] is a
clean case where that framing has nothing to act on. Every write is
correct and authorized: an organization's policy is superseded, and the
store soft-revokes the old record. Every provenance check passes the old
record by construction, because its origin is the defender. The harm is
entirely on the read side. Where the revoked record is returned (Graphiti
by default, mem0 with its shipped expiry filter disabled), it ranks
**first in every scenario** and leads agents to the unsafe action in
**43.1%** of trials. A store-level status filter takes that to
**0/1,620**. Prompt hardening only reaches 37.2%, because the agent
receives fact text without validity metadata. The paper sharpens this
concept's origin-binding layer: "origin binding does not constrain what an
authorized record does once its authorization is revoked."
louck2026securing and sharma2026smsr certify *who wrote* a record, not
*whether it is still current*.

The read filter is not sufficient either, which is why the two sides have
to be designed as one loop. Once an agent acts on the revoked record and
journals its decision, the journal is an ordinary current record that
nothing marks superseded. Filtered unsafe rates at later hops rise to
**71.6 / 79.5 / 83.1%** when journals are written directly; under the
system's own ingestion they are **7.5 / 7.8 / 12.8%**, with only 22 of 358
journals surviving as active (both measured on Graphiti only). So the
write gate has a job no source above assigns it: **a write derived from a
record must not outlive that record's revocation.** That is
chen2026fresh's cite-the-records-you-used rule applied to the agent's own
memory writes. The composition is untested.

Two caveats. The headline "no system enforces revocation by default"
overreaches: shipped mem0 does withhold expired records. The broad
failure is that under ordinary prose ingestion, cognee, langmem and mem0
record the revocation in **0/9** scenarios, so no mark exists to enforce.
And the tested agent is never shown the label, which leaves the cheapest
mitigation — validity metadata in context, which hu2026memory suggests
matters — unmeasured.

This also qualifies the Mem0 attestation under Connections.
chhikara2025mem0 documents that contradicted memories are deleted via its
UPDATE/DELETE decisions; measured behavior was retention with an expiry
marker (direct insertion) and no revocation at all under indirect
insertion. A documented write gate is not a measured one.

## Portability: the same gate, one trust boundary further out

[[literature/papers/ravindran2026portable]] asks what happens when memory
has to cross *between* agents — a different vendor, a different runtime —
and answers with a signed portable artifact: Merkle-DAG provenance,
Ed25519 artifact signing, capability-scoped read tokens, and a re-hydration
pipeline that treats imported memory as untrusted input. The design is
worth recording; the evidence is a pilot (N = 50, 2024-era models,
non-adaptive injection battery) and should not move this concept on its
own.

It is most useful here as a **clean illustration of the distinction this
concept turns on**, because it gets one half right and one half wrong by
the standard louck2026securing establishes:

- The Merkle DAG is a *derivation-edge* structure — the malleable class.
  It proves the lineage was not altered after the fact; it says nothing
  about whether an entry was legitimately acquired. A memory poisoned by an
  honest-but-wrong write at the source travels with an intact root hash and
  a valid signature.
- The framing / content-type injection defense at re-hydration is
  *content*-based — the class sharma2026smsr independently shows cannot be
  certified.
- The Ed25519 signature is the closest thing to origin binding, but it
  binds the **export event**, not each write to its channel. Signing the
  container is not signing the contents' provenance.

The general lesson: **transport integrity is not write integrity.** A
protocol can make memory perfectly tamper-evident in flight and still
faithfully deliver a poisoned entry. Any future portable-memory scheme this
project takes seriously should carry per-entry channel-authenticated origin
labels across the boundary, not just a hash chain over them.

This also reflects back on this project's own cross-project `@import`
contract, which moves concept files between repos with `used_by:`
back-references and *no* integrity layer at all — trusted because git
history is trusted. That is a defensible choice for a single-operator box
and an obvious gap if concepts were ever imported across trust boundaries.

## The decision before the write: commit, limit, verify, or ask

Every source above judges a transition once a candidate update already
exists — coverage, preservation, faithfulness, provenance. None of them
ask the prior question: *should this become a durable write at all?*
[[literature/papers/li2026remember]] supplies exactly that, with four
operationally distinct actions (persist / ephemeral / verify / clarify)
and a clean empirical result: models under-ask sharply more than they
under-verify. Bare Qwen3.5-9B verifies 12/18 freshness items but
clarifies 0/12 ambiguous ones, instead silently mapping ambiguity onto
`persist`, `verify`, or `ephemeral` — "the model may recognize
uncertainty yet consult the world rather than the user who alone can
resolve intent. It otherwise silently commits an interpretation." Both
tested model families under-ask; which wrong action they substitute is
family-specific (Qwen over-persists at 8x Claude Haiku's rate).

This adds a fourth axis to the coverage/preservation/faithfulness triad:
**source-of-truth routing** — is the world or the user the authority for
resolving this candidate update's uncertainty, and does the write policy
route to the right one before committing anything. It also supplies a
concrete asymmetric-cost design rule this concept can adopt directly:
"when persist and a weaker action remain tied, prefer the weaker
commitment... an unnecessary question is visible and recoverable, while
a wrong durable update can remain silent" — the same silent-failure-is-
worse logic [[concepts/refusal-cost-symmetry]] argues for a gate,
restated as a write-time tie-break rule.

A second finding compounds the write-gate's honest-limit theme this
concept already tracks: **a model's stated commitment decision does not
reliably survive translation into the tool call that would actually
execute it.** Forcing the same decision into a structured `memory_write`
/ `use_now` / `check_source` / `ask_user` call changes the outcome on
30/70 items for Claude (57% label-to-call agreement) and on the majority
of items for Qwen (23% agreement, accuracy collapsing 0.557 → 0.343). Any
write-gate evaluated only on what a model *says* it will do — rather than
on the tool call it actually emits — may be measuring a different, more
optimistic quantity than what a deployed system commits.

## Faithful to the evidence is not true: re-observe the world before writing

Every gate above judges a candidate against something already in hand:
the incoming evidence (TrustMem's faithfulness, "adds nothing
unsupported"), the writer's origin, or the entry's form.
[[literature/papers/suresh2026grounding]] targets the case where the
evidence in hand is the defect. A post-task curator that sees only one
trajectory writes records that are faithful to a partial, often mistaken
observation. They memorize an answer instead of the procedure, assert a
scope nothing tested, or keep a table name that a schema migration has
since removed. Each would pass a faithfulness check. The fix is an
**independent re-observation at write time**: the asynchronous curator,
the only agent allowed to write, gets a least-privilege *read-only*
subset of the task's environment tools and is told to probe
counterexamples, untouched slices, stale mappings and preconditions
before it creates or updates a record. This is li2026remember's *verify*
action turned into a curator tool, and it adds no write authority.

The measured effect is real but smaller than its headline. On CLBench
with an unannounced schema migration (GPT-5.4), no memory passes 39%,
trajectory-only curation **70 ± 16%**, and probing **73 ± 5%**, at
task-agent cost $3.38 / $1.99 / **$1.68**. Probing's clearest gain over
trajectory-only curation is variance, not the mean. The qualitative
difference is sharper: probed records are executable positive procedures
(join key, filter, grain, current table), and they cut matched-task
queries from 4/9/8 to 1/2/1. Three limits before leaning on it. Curator
cost is excluded from every cost figure, so "cheaper" means the task
agent's bill. Terminal feedback includes the correct answer, so probing
here is partly label-guided search, and the no-oracle case is untested.
The stale-repair mechanism isn't isolated either: probing's lead is
mostly in place before the migration, and existing records are only
re-checked when a new task's curation retrieves them. The same check
appears as a skill-admission gate in [[concepts/skill-library-lifecycle]].
