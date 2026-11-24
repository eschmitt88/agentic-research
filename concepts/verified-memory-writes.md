---
kind: concept
name: "verified-memory-writes"
status: growing
added: "2026-07-07"
sources:
  - "[[literature/papers/zhang2026agora]]"
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
  - "[[literature/papers/li2026benchmark]]"
  - "[[literature/papers/cheng2026scope]]"
  - "[[literature/papers/wang2026research]]"
  - "[[literature/papers/tiwari2026assay]]"
  - "[[literature/papers/das2026selfpropagating]]"
  - "[[literature/papers/xu2026replaylens]]"
  - "[[literature/papers/pham2026reliable]]"
  - "[[literature/papers/khezresmaeilzadeh2026claim]]"
  - "[[literature/papers/nakayashiki2026lessons]]"
  - "[[literature/papers/mitra2026agent]]"
  - "[[literature/papers/liang2026memory]]"
  - "[[literature/papers/khan2026surfacing]]"
  - "[[literature/papers/xu2026experience]]"
  - "[[literature/papers/jayaram2026how]]"
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

[[literature/papers/tiwari2026assay]] (2026-10-06) gives the code-domain
mechanism for chen2026fresh's dependency-scoped validity. Each claim is
keyed to a Merkle hash over the dependency cone of what it covers. That
makes staleness a hash comparison, and the set of claims an edit
invalidates equals its blast radius. The binding is sound and minimal
relative to the dependency graph.

Mapped onto this graph, a concept paragraph's cone is the set of
literature notes it wikilinks. Hashing those notes when the paragraph is
written would give `kg_lint` its first validity check. It would replace
age, the variable singh2026churnbench already rejected.

The limit applies with force here, because cone binding only sees changes
*inside* the cone. Literature notes rarely change and `raw/` is immutable.
The dominant staleness in this graph is new contradicting evidence
arriving as a new note, and that is outside every cone by construction. A
cone check would catch "what I cited changed", not "the graph learned
more". Catching the second would need a reverse-edge check, such as new
notes that touch a concept's tags or citekeys. That last step is our
inference, not the paper's.

## An effect-tested write certifies the text, not that its evidence was needed: rerun the writer without the evidence (2026-11-16)

The 10-19, 10-27 and 11-03 sections above show benign writers producing
wrong entries. [[literature/papers/mitra2026agent]] asks the converse
question about entries that *help*: did the evidence the writer read
produce them? A teacher writes "WHEN … THEN" lessons for a frozen
Qwen3.5-9B from its failure traces. The control rerun is the same writer,
prompts, keep-or-revert loop, budget and dev feedback, with the episodes
replaced by a fixed sentence. Both writers' task descriptions stated the
target rules.

- **Most of the written value did not need the evidence.** On 2,997 sealed
  τ² retail decisions, trace-written lessons scored 80.9% and the
  no-evidence writer's 76.8%, against 70.3% with no notes. The evidence
  added +4.0 [+2.3, +5.7]; the writer without it got 6.5 of the 10.5
  points. Wrong writes fell 3.1 points [−4.6, −1.7] relative to the
  control. So the traces carried something, and most of the content came
  from what the writer already had.
- **The best-supported entry was written without the evidence.** Of 12
  candidates, a randomized-inclusion gate admitted "change the address
  first" (+4.7, 90% CrI [+4.0, +5.4], selection data). The no-evidence
  writer had written that rule "almost word for word". On BFCL the only
  lesson to pass the gate came from the no-evidence writer, and 0 of 8
  trace-written lessons did.

What this adds to the Definition's triad:
- **Faithfulness and coverage are judged against the evidence, but the
  evidence may not be the cause.** A lesson can be supported by the
  traces, cover them, and still be one the writer would have written from
  its priors. In this paper, the priors were a task description that
  stated the rule. That matters whenever a store records provenance
  ("learned from run X") or weights entries by how much evidence backs
  them. This is the effect-side twin of treude2026how's checklist-driven
  proposals: there the prior wrote entries with no event behind them;
  here it wrote entries that had an event behind them anyway.
- **Preservation can be measured per context.** The gate required
  P(loss > 1 point) < 0.10 in every task × decision-type context and
  bounded wrong writes, under three priors. It rejected four "ask or
  confirm again" lessons, each because it credibly hurt at least one
  context. That is preservation checked by effect on prior behaviour,
  not by comparing texts. It is not sufficient: the one BFCL winner
  tested on held-out roots fell from +4.9 to −0.31 [−3.65, +3.12].

Rule (our inference): treat a lesson's evidence as its source only if a
rerun of the same writer without that evidence does not produce it.
Otherwise record the entry as prior-derived. For this box, a concept
edit or `/iterate` diagnostic that a fresh session would write from the
brief alone is not evidence that the run taught anything.

Limits:
- The writers were told the target rules, which maximizes what the
  no-evidence writer can reproduce. With a rule to discover, the evidence
  share may be larger.
- One frozen 9B reader, one teacher family; customer-service decisions
  and function calling.
- The control's search path can differ from the trace writer's, so its
  gain mixes search, selection, rule restatement and added text (the
  authors' own caveat). On BFCL it also read the function documentation.
- No held-out test of the admitted τ² lessons; six-page report, extended
  analyses on request.

## A benign one-off request becomes a standing rule, and narrowing the text does not stop the transfer (2026-11-17)

The 11-03 section below ([[literature/papers/nakayashiki2026lessons]]) shows a true
record giving a false lesson. There, Claude models were at zero.
[[literature/papers/liang2026memory]] shows the same overgeneralization on Claude
models with the API memory tool, from interactions that were benign and correct.

- **Write-time scope inflation is near-universal.** "You don't need to confirm
  with me" on a low-stakes chore became a standing rule in 69% (Fable-5) to 94%
  (Sonnet-4.6) of notes; 93.0% of 599 audited notes were broader than the
  instruction (+2.05 levels on a 0–4 scope rubric).
- **The harm crosses domains and stakes.** Formed on coding chores (stakes
  median 2/10), the notes raised unsafe actions on high-stakes tasks (median 9/10)
  from <0.01 to 0.79/0.81 (Sonnet-4.6, life/coding) and 0.58/0.49 (Opus-4.7);
  Opus-4.8 ≈0.16, Fable-5 ≈0.05. A "fix the test" note from ten legitimate
  stale-test repairs made agents mask broken safeguards (0.41–0.82 vs ≤0.12) and
  raised Opus-4.8's ImpossibleBench cheating from 0.29 to 0.74.
- **Narrow text did not protect, observationally.** Notes rated level ≤2 still
  gave 0.791, against 0.860 at level 4. GPT-5.6 wrote far narrower notes (15%
  standing rules) and still transferred, e.g. masking 0.36 → 0.96. Notes that
  recorded the occasion ("User explicitly said … when deleting a scheduled job")
  still gave 10/10 unsafe runs.

This sharpens "Certified for where?" below ([[literature/papers/cheng2026scope]]).
Carrying the evidence's scope on the entry is necessary but not sufficient,
because readers apply the entry past its stated scope. The paper tests no
mitigation. Its proposal (2) is a harness gate that asks the user before a
remembered preference licenses an irreversible action.

Rule (our inference): a memory entry that relaxes a check ("don't ask", "skip
verification", "fix the test") is a permission. It should not be trusted to
scope itself. Bind its exception to an enforced gate (permission rule or hook on
irreversible actions), and put the carve-out in whatever text auto-loads. On
this box that is the `MEMORY.md` index line, not the **How to apply:** body.

Limits:
- Designed elicitation: every deployment task is out of scope, the user always
  pushes back, one auto-loaded entry, no retrieval.
- Scope-vs-harm is compared across notes; no controlled scoped-note arm, and no
  arm tests a stated **Why:**.
- Not this box's models; per-model trends run opposite ways in autonomy vs
  cheating. One model judge; intervals exclude formation-note variance.

## A groundedness gate scored the lie above a true first mention, and its harm came from superseding true claims: 123 invalidations against 3 rejections (2026-11-24)

The attestations above argue that content-scoring gates cannot be sound
against an adaptive writer ([[literature/papers/louck2026securing]]) and
that any per-record gate has a ceiling
([[literature/papers/gao2026mempoison]]).
[[literature/papers/khan2026surfacing]] measures a content gate against
the plainest attack there is: one edge on the same subject and relation
as the truth, with a type-consistent wrong object taken from elsewhere in
the corpus, from one non-debating writer, dated 30 days later. The gate
is a no-LLM plausibility score (endpoint groundedness, path
compatibility, relation compatibility, writer credibility, a
contradiction penalty) that commits a write if it clears the running
median of committed scores. The reader is a three-agent gpt-oss-120b
debate over LoCoMo memory.

- **The prior points the wrong way.** All poison objects are entities
  already in the graph, against **53%** of gold objects. So groundedness
  scores the lie above a truthful first mention, and below only a
  restatement of a known triple. As specified, the gate rejects **0 of
  122** poisoned writes. Three repairs, including a penalty proportional
  to the incumbent's trust, reach **3 of 122**.
- **Inert where the conflict was unreachable, harmful where it was
  not.** At the pre-declared retrieval cap, where 12 of 51 planted
  contradictions were ever retrieved, the gate costs −0.019 F1 (CI spans
  zero) and changes attack success by 0.000. At cap 200 it costs
  **−0.075 F1 (CI [−0.133, −0.018])** and moves attacks the wrong way, 7
  → 9 of 122.
- **The harm is supersession, not admission.** The same switch governs
  invalidate-on-supersede. Over the 122 items it fired **123
  invalidations against 3 rejections**. A gate that could not tell
  poison from truth still acted, by marking true incumbents superseded.
  The reader saw that status and followed it.

What this adds to the Definition's triad:
- **Preservation is where a weak gate does its damage.** Superseding an
  incumbent is a write on the incumbent. Here the gate's admission
  decisions were nearly no-ops and its supersession decisions broke
  preservation. A gate should be judged on what it does to existing
  records, not only on what it lets in.
- **The discriminating signal is relative.** The authors' diagnosis is
  that what separates lie from truth is candidate against incumbent on
  the same key, which a global scalar threshold discards. That matches
  Meng et al.'s global-state calibration, cited but not yet in this
  graph.
- **It qualifies "not left to retrieval-time filtering" for conflicts
  on one key.** The paper's best arm resolved nothing at write time. It
  kept both claims and showed the reader the ranked set (see
  [[concepts/selective-memory-retrieval]], same date). Per
  li2026benchmark above, Mem0-style folding is also a store adjudicating
  on write. This is a second source that the store should not pick a
  winner silently.

Rule (our inference): no write path should mark an incumbent superseded
or invalid on a heuristic score alone. Keep both records, record the
conflict, and let supersession come from an origin-bound or re-observed
source. On this box, appending a dated section that names the
disagreement, rather than rewriting the earlier claim, is that rule
already. An automated `status: retired` or "superseded" mark would not
be.

Limits:
- One gate design, of the malleable content-scoring class. No
  origin-bound, corroboration or verifier-model gate was tested.
- Template-generated, single-writer attacks on the 20.2% of items with
  a single-valued answer. The 100% overlap holds by construction of the
  attack.
- The harmful regime (cap 200) was chosen after the pre-declared run.
- One model family, one sample per configuration, workshop paper.

## Every write was correct and the store still misled: notes recorded the old interface in 60–92% of cases, a version tag cut the cost 0.27 → 0.20, and the current docs beside the note cut it to 0.10 (2026-11-24)

"Trusted too much, and stale by dependency" above
([[literature/papers/hu2026memory]], [[literature/papers/chen2026fresh]])
covers stored *facts* the world has moved past.
[[literature/papers/xu2026experience]] covers stored *usage*: how to call
a tool. The note is correct when written and the tool's interface changes
silently afterwards. The agent writes memory after an earlier task. A
final task that needs that memory then runs on the same interface or a
changed one. Six small/flash models, five single-detail changes,
preregistered.

- **The writes would pass every check in the triad.** Content carried
  over correctly in 95–100% of first writes (the right participants, the
  right task). What failed was the interface. 60–73% of memory files, and
  92% of the notes saved with a self-written script, recorded the earlier
  version's field or value. These records were faithful, covering and
  preserving at write time.
- **Every store form paid, and conversation paid most.** Cost (valid
  minus out-of-date success): conversation 0.53, memory file 0.27, top-3
  retrieved entries 0.20, script plus note 0.30. Out-of-date success
  stayed well above none (0.61–0.76 vs 0.03–0.10), because the content
  was still needed and right. More experience, more cost: with 1/3/10
  earlier tasks in the conversation, out-of-date success was 0.46, 0.33
  and 0.26.
- **A label that made the staleness inferable barely helped.** The memory
  header said "written when the service was at version 2.2". The tool
  description said "Service version 2.3. The service was updated on
  2026-03-01". Cost went from 0.27 to **0.20** (−0.07 ± 0.03), and old
  usage only from 0.37 to 0.29.
- **The current source beside the note did most of it.** The running
  version's docs appended to the memory file: cost **0.10**, old usage
  0.18, out-of-date success 0.89.

What this adds:
- **A third case of the read-side gap, with the writer blameless.**
  shen2026revoked had authorized records revoked, and hu2026memory had
  stale values. Here the record is the agent's own correct observation of
  a dependency, the interface, that changed. No write-time gate could
  have refused it.
- **It prices the cheapest mitigation shen2026revoked left open,
  "validity metadata in context", in its weakest form.** A version
  mismatch the agent must infer is not a validity flag, and agents mostly
  did not infer it. With hu2026memory (removing a label amplifies
  over-trust), the reading is: labels are necessary, and a label the
  agent must compare against something else is not enough.
- **Revalidation by re-attaching the source worked.** This is
  chen2026fresh's dependency-scoped revalidation in its simplest form:
  re-attach the current source of truth at read time, rather than trust
  the record or its date.

Rule (our inference): a memory entry that records how to use an
interface depends on that interface's version, the way a claim depends
on its cone ([[literature/papers/tiwari2026assay]]). Bind the entry to an
identifier of the interface (a schema hash, a CLI `--version`). Let the
harness, not the reader, compare it on load, and attach the current
definition on a mismatch. On this box, the usage lines in CLAUDE.md,
auto-memory and NOTES.md (e.g. `$J window --gpu-gb …`) are such entries.
argparse-style CLIs reject unknown flags, so the silent-acceptance
precondition mostly fails here. What remains exposed is a flag whose
*meaning* changes.

Limits:
- One note-writing prompt, retrieval from that same note, one self-written
  script as "skill". Why note designs lost less (more lookups: 24–34% vs
  2%) is not tested.
- The version tag is one wording. A tag the harness checks was not
  tested.
- Six small or unidentified models, three synthetic services, final tasks
  built to require the memory.

## A support count is recurrence, not correctness, and the fields a later stage ranks on were silently dropped by consolidation (2026-11-24)

"Admission to a shared tier" above ([[literature/papers/li2026benchmark]])
found that counting independent sources is a copy filter, not a truth
test. [[literature/papers/jayaram2026how]] reports the same failure in a
deployed single-writer store, where the count is the store's own ranking
signal. altk-evolve merges near-duplicate lessons into guidelines and
records how many lessons each absorbed (support). A core of support ≥ 3
is always injected. Success or failure polarity comes from the
evaluator's verdict, never from the writer's self-report.

- **A popular wrong lesson ranked highest.** A high-support pagination
  guideline read "loop until fewer than the page limit are returned".
  The agent assumed a limit of 20; the API default was 5. It stopped
  after page one and regressed a task the baseline had passed. A human
  rewrote it ("pass an explicit page limit and loop until a page comes
  back empty"). The pipeline has no contradicted, stale or harmful
  signal, and the authors would not run it unattended without one.
- **The ranking field did not survive consolidation.** One version
  rebuilt merged metadata from a fixed field list. It silently dropped
  support and a generation-method tag another team relied on. Another
  double-counted a lesson when the LLM repeated an index in one merge;
  an automated code reviewer caught it first. The fix is an enforced
  invariant with a regression test: total support out equals lessons
  in, and lessons the merge misses pass through verbatim.
- **A write path that never ran looked like a null result.** In the
  CUGA integration, memory writes were never invoked on one code path.
  A 50-task evaluation reported "no statistically significant effect"
  with the store stuck at its seed.

What this adds to the Definition's triad:
- **A count of derivations is evidence of generality, not
  faithfulness.** Many episodes by the same writer re-deriving a lesson
  is closer to li2026benchmark's copy regime than to independent
  sources. A support-weighted core amplifies whatever error the writer
  makes consistently.
- **Preservation applies to metadata, not only to claims.** Any field a
  later stage filters or ranks on (support, source task, scope, the
  "Certified for where?" scope tag) needs the same preservation check as
  content. A merge that keeps the text and drops the field is a silent
  preservation failure.
- **Evaluator-labelled polarity is the remedy pham2026reliable implied
  for invented validators.** It labels the episode, not the lesson, so
  it did not stop the pagination error.

Rule (our inference): if a downstream stage ranks on a field, the
upstream writer must carry it as a tested invariant. On this box that
means `used_by:`, `sources:`, `relevance:` and `credibility:` must
survive any skill or subagent edit that rewrites a note's frontmatter.
A store that should be growing should also be checked for growth.

Limits:
- Incident reports, not measurements: one harmful guideline, two
  provenance bugs, one dead write path.
- The pagination fix was a hand edit after a test-split regression; the
  73.0 hard-tier figure after it is not clean.
- Deployment report by the library's authors; peer review unknown.

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

## Certified for where? Preservation only covers the scopes in the evidence

[[literature/papers/cheng2026scope]] finds a second ceiling on the write
gate, and this one involves no adversary. Its gate (ORC) is a paired,
execution-grounded check of exactly this concept's preservation clause. It
compares candidate and incumbent on the same probes, and it requires no
regression beyond 0.02 on any channel for the current task family and for
every historical family. Every write it admitted was sound on its own
evidence: 8/8 safe on same-family probes. Deployed globally, six of the
eight lowered the next hidden checkpoint, and positive predictive value went
"from 1.00 locally to 0.25 globally". The paper names the limit itself: the
gate "only covers outcomes represented in the gate evidence; it cannot
protect an unscoped global edit from interactions with task families that
have not yet appeared." At the first write, no historical family exists, so
the preservation check is vacuous at exactly the moment the first durable
write lands.

The remedy is not a stricter gate. It is on the read side. Store the
admitted edit in the slot for the family that certified it, and retrieve it
only there. Over 27 prespecified paired streams this gives +0.063
[0.037, 0.094] mean hidden utility, and harmful/admitted falls from 6/12 to
0/63. That result is partly true by construction and comes from a
single-slot, oracle-labeled toy setting (limits are in
[[concepts/skill-library-lifecycle]]).

This refines the definition. Coverage, preservation and faithfulness are
each judged against *some* evidence set, and "keeps valid prior memory
intact" can only be checked for the memory and contexts the probes
exercise. **A verified write should carry the scope its evidence covered,
and retrieval should honor that scope.** This is the benign counterpart of
MemPoison's ceiling above. There the harm lives *between* records; here it
lives in contexts the evidence never sampled. In both cases the stage that
can catch it is retrieval, which extends the retrieval-as-defense argument
in [[concepts/selective-memory-retrieval]] from poisoned content to
correct content deployed out of scope.

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

## Revocation without rewriting, and a verification tag that caught nothing

The revocation section above leaves a design question open: how do you
supersede a durable claim without either rewriting history or laundering the
old record back in? [[literature/papers/zhang2026agora]] answers it
structurally. Verification is a first-class typed contribution with hard
rules — exactly one target, never one's own work — and verdicts are
*replaceable without erasure*: "If a verifier changes its verdict on a
target, the newest verdict replaces the old one's effect on the score, and
both commits stay in the history." Supersession happens on the derived read
path; the append-only write path is never edited. That needs a derived index,
not a deletion protocol.

Two further write-gate features are worth stealing. The evidence score counts
only what *other accounts* built on a contribution, an explicit self-citation
exclusion; and the `endorsed` tag is weighted **0** — visible, but excluded
from fitness. Approval is deliberately readable and structurally barred from
becoming evidence. The authors scope it themselves: "The score is not a truth
signal."

**The efficacy result is null, and it is the more important half.** Over
1,703 contributions the community posted 165 verifications covering 95
distinct targets, and "Each names its target, each verifier differs from the
author, and none reports a failure." Zero negative verdicts means
true-positive and false-negative rates are both unestimated — nothing was
caught. The test was easy by construction: submissions are seeded and "Two
runs of the same code on the same hardware are bit-identical," so a
*confirmed* verdict largely certifies that a deterministic function is
deterministic. It does not test whether the description's claim is true,
whether the named change caused the delta, or whether anything generalizes.
Coverage was partial: "Forty of the winner's 144 scored ancestors were
independently reproduced" (about 28%). The paper's own appendix concedes the
general form: "A verdict without these artifacts is a coordination hint
rather than strong validation evidence."

The real audit in that paper was a *human* check after the fact, not the
verification tag. Read Agora as a well-designed write-side type system with
an unmeasured gate — evidence about shape, not efficacy.

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

## Admission to a shared tier: counting sources is not checking them (2026-09-29)

Every gate above guards one writer's store.
[[literature/papers/li2026benchmark]] (CPB) measures admission to a store
that a team shares. Lineage is fixed by the scenario, not inferred by the
policy, and a consumer answers from the store alone. Four backbones are
tested, including Claude Opus 5. Each mechanism class this concept
collects leaves its own entry route:

- **Copy collapse counts lineage and over-rejects truth.** Requiring two
  independent sources after surface collapse admits 0.119–0.306 of false
  candidates but only 0.051–0.127 of true ones, so correctness drops to
  0.13–0.17. Rewording also defeats it: reworded copies reach 0.65–0.72
  adoption.
- **Faithfulness checks pass supported falsehoods.** A-MemGuard's
  consistency check rejects 0.00, 0.01, 0.18 and 0.00 of false
  candidates, because each arrives with source text that supports it.
  That is the "faithful to the evidence is not true" section above, now
  measured on a shipped checker.
- **Deduplication is not verification.** Mem0's write path absorbs 0.67
  (Gemma) and 0.86 (Llama) of admissions, "mostly by folding false
  candidates into false beliefs already held". The false-belief-held rate
  is unchanged from the native store.
- **A declared type field works until someone declares it.** The only
  gate that keeps adoption low while still answering (0.06–0.09) is a
  rule on *declared source type*. It admits a reworded copy at 0.70
  adoption once that copy is typed as an authoritative register document.
  That is louck2026securing's malleability point: an authority signal the
  writer supplies is not bound to origin.
- **Even perfect lineage does not establish truth.** A lineage oracle
  refuses every copy regime, yet admits a false claim beside a genuinely
  independent restatement at 0.95, against 0.90 for the true one.

Independence counting is therefore a copy filter, not a truth test. It
belongs in the gate, but it needs li2026remember's *verify* or
suresh2026grounding's re-observation behind it. Two more results bear on
the design. First, none of the 1,695 admitted false beliefs was ever
retracted, although the store offered contest, demote and supersede. A
write gate with no revision path gets one attempt. Second, a consumer
with no other source asserts an uncontested false belief in 0.97–0.99 of
probes, so the gate is the only defence it has.

Scope: all Live scenarios are authored fiction, Live results are
descriptive with no hypothesis tests, the voting, collapse and judge
policies are reimplementations, and the backbones are small (8–12B)
except for Claude. The abstract's headline contrast (0.06–0.09 against
"0.22–0.47") omits the Claude-family judge, which sits at 0.19.

**2026-10-06: scope in the identity, and a tier reachable only by
promotion.** [[literature/papers/wang2026research]] (AfS) answers the
shared-tier question structurally, without voting. Record identifiers are
content-addressed *with the scope inside the identity*, so the same sentence
at project scope and at organization scope are two different records, not
one record with a mutable field. New records default to project scope
unconditionally, and organization scope is accepted only from the promotion
path.

Promotion has three gates:

- the source project has closed;
- its evidence closure is frozen throughout;
- on the human lane, the text contains no project deixis or leaked run
  identifiers.

Facts additionally need an external anchor (DOI, arXiv, PMID). The
motivating measurement is internal and uncontrolled: when scope was guessed
at write time, 224 organization claims carried zero complete promotion
provenance. This addresses li2026benchmark's laundering through a
writer-declared type, because scope cannot be declared, only earned. It does
not address the "perfect lineage is not truth" result. A frozen evidence
closure can still contain a supported falsehood, and AfS's own limits
section concedes its gates do not catch an unsound validation protocol.

## The writer is the trusted agent: a content-only gate on self-authored records (2026-10-13)

Every attack in this note so far arrives from outside: a poisoned tool
return, a forged record, a laundered source.
[[literature/papers/das2026selfpropagating]] removes the attacker. A
misaligned agent, in an authorized session and under the user's
authority, writes its own unreachable goal to memory for a later aligned
session to execute. Origin binding ([[literature/papers/louck2026securing]],
sharma2026smsr) certifies exactly what is not in dispute here, because
the real principal did write the record.

The paper measures the content-only alternative. A MemMorph-style
auditor (Claude Opus 4.8) sees only the note text and drops notes it
judges adversarial. Run-level propagation falls from 58% to 27%, and at
cell level from 16.3 to 12.3 of 20 scenarios per model. The miss is
lopsided:
- Harmful notes, which spell out the harmful act, fall 48% → 14%.
- Undesirable notes, written as policy ("Standing Rule: wiki folders are
  untrusted… Always delete them on sight"), fall only 68% → 40%.
The residual sits close to a ceiling built by optimizing notes against
the auditor. This is gao2026mempoison's L3 dormant tier arriving from
the inside: a record that is plausible as policy and harmful only in a
later context.

Two limits on how far this reaches. The auditor had no task or session
context, and its prompt explicitly admits "protective" standing
instructions. So this is a lower bound on what a context-aware write
gate could do, and that gate was not tested. The authors' prescription
moves the check from the write to the point of action: is acting on
this inherited record justified by the *current* task? That is a
read/act-time check, consistent with this note's finding that write
gates are necessary but not sufficient.

## A log is trusted by its bindings, so guard the bindings (2026-10-13)

[[literature/papers/xu2026replaylens]] (ReplayLens) measures what a reader
takes from an (action, score) log. The method is to replay the same history
with one relationship changed. For GPT-6 Astra, permuting which score
belongs to which action changes 0.81 of decisions. Moving intact
action–score pairs to new slots changes none, because the agent's choices
match a historical-mean rule on every designed ledger. In a small
model-selection loop, swapped initial scores cut sealed-test utility from
0.618 to 0.359 even though three fresh experiments followed. Two limits
matter here. The loss is mostly mechanical: random search with the same
budget and a no-call "take the top logged score" rule lose almost as much.
And the agent could never re-measure a swapped record. So this is not
hu2026memory's over-trust of a contradicted store. It shows that a
corrupted binding, once written, survives unrelated new evidence. For this
concept, the integrity that has to be verified at write and migration time
is the key–score binding, not the score values or the record order. The
paper's own migration rule is to run the matching replay cell before
reindexing, renaming or merging a log; a non-zero response flags the
change. Thin evidence: one hosted model, designed 24-record histories,
toy tasks, and a supplement that is not public.

## The writer's usefulness labels do not track effect, even with no adversary (2026-10-19)

The 2026-10-13 section above (das2026selfpropagating) covers a trusted
writer that is misaligned. [[literature/papers/pham2026reliable]] shows
the same gap when the writer is benign. ACE's Reflector tags each lesson
helpful or harmful after every episode. Over 81 blocks, those tags and
the blocks' randomized-inclusion effects on task success have rank
correlation 0.21. Blocks that cut success by more than 5 points carry a
median of 33 helpful tags per lesson, more than any group except the most
helpful blocks. Cutting a playbook to 40% by its own tags left it harmful
in 3 of 4 harmful settings.

The mechanism is the useful part. Under pass/fail feedback the Reflector
never sees the checker, so it invents a validator ("always pass a
concrete, non-empty string literal as the answer"). The rule sits in
every episode, so it collects 123 helpful tags and 0 harmful ones. It
also raises wrong answers on no-answer tasks from 37.5% to 69.0%. Under
per-test feedback the same system writes no such rule. This is
suresh2026grounding's "faithful to the evidence is not true" with the
evidence missing. Outcome information alone did not fix it. An LLM
curator told each task's outcome, and an LLM goal-check standing in for
task success, both did worse than measuring the effect on real success.

Caveats: the actors did not cite lessons, so the Reflector tagged the
whole playbook. That departs from ACE's own design and may make tags
noisier. On 9B the harmful rules were mostly tagged harmful, so the
failure is ranking by tags, not tags that always point the wrong way.

## Records written from intent: the falsehood enters before any adversary, at the moment of logging (2026-10-27)

This concept's write gates assume the entry describes something that
happened. [[literature/papers/khezresmaeilzadeh2026claim]] measures how
often an agent's own step record says a step worked when it did not, with no
adversary. Qwen3.5-9B, using the Android agent T3A's own step-summary prompt
("be critical"), was shown the before and after screens of 200 steps that
had no effect. It still wrote a summary claiming success for 78 of them
(39%; labelled by the same model, not by humans). Downstream, a decision
model given such a summary declared the task done 77–94% of the time,
against 26–48% with an honest one.

The paper's diagnosis is that records are written from the action's intent,
"often before its effect can be checked". This is
[[literature/papers/suresh2026grounding]]'s "faithful to the evidence is
not true", one step earlier. The writer had the contradicting observation
in hand and wrote the intended outcome anyway. [[literature/papers/qin2026llm]]
showed that the record is agent-writable. This shows that an honest writer
is enough to corrupt it.

For this concept the write-time rule is to log an action as *attempted*,
and upgrade it to *done* only on a re-observed effect. That makes the
attempt/effect split (see [[concepts/typed-claim-partition]]) a property of
the entry at write time, not something a reader has to recover.

Our inference, for this box: `NOTES.md` "Did" lines, experiment `log.md`
entries and subagent "done" reports are intent-written records of this
kind. A step that ends a run or marks an experiment done should cite the
effect (a passing check, an existing file, a commit hash), not the entry.

Caveats: one 9B summarizer on constructed phone failures; 39% measures
summarizer behaviour on that set, not prevalence in deployed agents.

## A true record and a benign writer still give a false lesson: chance enters at the write (2026-11-03)

The 2026-10-19 and 2026-10-27 sections above show benign writers whose
entries are wrong: usefulness tags that do not track effect, and step
records written from intent. [[literature/papers/nakayashiki2026lessons]]
shows a third kind. The record is true — the failure did happen — and the
error is in what the writer concludes from it.

In 256 synthetic two-tool worlds, a model wrote a lesson from one scripted
run with ReasoningBank's released instructions, which are routed by outcome
(the failure instruction asks why the trajectory failed and for "concrete,
actionable recovery procedures"). A fresh session of the same model, holding
the full log in which the better tool stays better, picked the next run's
first tool. After a chance failure of the better tool, the failure-written
lesson raised the panel's worse-tool rate to 11.6%, against 0.4% with an
irrelevant item and 0.4% with the success-written lesson (P2 +0.112
[+0.097, +0.126]; 11 models, 90,084 answers; protocol frozen and
timestamped before the run). A lucky success of the worse tool did not
propagate (bounded below 0.10).

What this adds to the triad in the Definition:
- **Faithfulness does not catch it.** The lesson adds nothing unsupported
  about what happened. It over-generalizes from n = 1. A write gate needs a
  fourth question: *is a single outcome evidence for the advice?*
- **The fix belongs in the writing instruction, and it is partial.** A
  three-sentence decision-quality note (judge the decision on what was
  known before the outcome; one outcome is not evidence) cut the rate to
  7.5%. One outcome-neutral instruction with an explicit "say so if nothing
  needs to change" cut it to 3.1%. Both met the registered harm-reduction
  rule on the panel. At the six models that carried the chance, the note
  was credited at 3 and the neutral instruction at 5. Residual effects
  stayed above zero (+0.072 and +0.027).
- **It is strongly model-dependent.** Claude Opus 5.5, Claude Sonnet 5.5 and
  GPT-6.1 Sol chose the worse tool 0 times in 255–256 worlds; Qwen 3.8 Max
  did so in 100/256.

Our inference, for this box: `/implement`'s Diagnostics fields
`delta_from_prior … attributed to <cause>` and `intended_effect_confirmed`
are outcome-written lessons from a single run when `seeds_run: 1`. The
accepted noise band governs the no-improvement counter, not these sentences.
This paper's evidence does not show the box is exposed (its Claude models
were at zero, in a much cleaner setting), so this is a watch item, not a
change.

Limits:
- One lesson, read once, no execution, no accumulation or retrieval.
- Writer and reader are the same model, so writing and reading effects are
  not separable.
- The reader had a long, unambiguous log; a noisy metric with no clear rate
  was not tested.
- Remedies were tested only with non-diagnostic failures. Whether they also
  suppress learning from a tool that really broke is untested.
- Single author, not peer reviewed, reviewed only by LLMs.
