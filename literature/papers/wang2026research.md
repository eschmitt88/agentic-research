---
kind: paper
title: "Research-Native by Construction: Minimal Nodes, Re-verifiable Workflows, and Compounding Memory for Long-Horizon Scientific Agents"
authors: ["Di Wang", "Yu Liu", "Bing Cui", "Chaoqun Ji", "Dongyuan Ni", "Jingyu Lu", "Kunlei Cui", "Pu Qin"]
institutions: ["IdeaHorizon (company; byline 'IdeaHorizon Team', techsupport@ideahorizon.ai)"]
year: 2026
venue: "arXiv 2609.35182v1 (cs.SE), 2026-09-28; self-described system description, 'not an evaluation'"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.35182"
code_url: null
citations: null
source: "raw/papers/wang2026research.pdf"
added: "2026-10-06"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["system-description", "afs", "unrepresentable-state", "preregistration", "closure-conditions", "hash-chained-ledger", "freeze-forgery", "write-gate", "verdict-authority", "no-verdict-layer", "two-tier-knowledge-base", "promotion-by-rewriting", "dead-end-cards", "candidate-queue-failure", "dispatch-fingerprint", "context-as-rendered-view", "durable-change-stall", "deepseek-v4", "no-benchmark"]
---

# Research-Native by Construction: Minimal Nodes, Re-verifiable Workflows, and Compounding Memory for Long-Horizon Scientific Agents

## TL;DR

A 34-page system description of **AfS** (Agent for Science), a
commercial research-agent platform from a company team. Its one thesis is
that research credibility should move "from 'ask the model to comply' to
'make the non-compliant state unrepresentable'". That thesis is encoded as
six "iron laws" enforced on the write path. The design spans three time
horizons:

- **Run.** Eleven node types: five producing, three service and three
  architecture nodes. Capability grows by deepening a node's contract, not
  by adding node types.
- **Project.** A frozen inquiry contract, a hash-chained freeze ledger, and
  closure tallies computed from ledgers and never stored.
- **Organization.** A single-file `MEMORY.md`, plus a two-tier knowledge
  base whose organization tier has no birth channel except promotion by
  rewriting.

The paper reports **no benchmark and no ablation**. Its evidence is three
failure traces, two closed campaigns chosen because they are fully
recoverable from disk (out of 35 run), some internal operating audits, and
one cost breakdown. Every recorded run used DeepSeek V4. Read it as a
detailed, unusually self-aware design source. It is the closest published
analog to this box's own discipline (`raw/` immutability, experiment
folders, promotion into `concepts/`). It is not evidence that any of the
mechanisms work.

## Claims

- **Placement determines reliability.** The same requirement can sit in
  the prompt (weakest), in a post-hoc check (weak, "killing the run does
  not undo what ran"), or as an unrepresentable state (strong). Table 5
  refines this into eight layers: guidelines, rules, skill, hook injection,
  tool validation, write gate, closing gate, process sandbox. The bottom
  three "can refuse a legitimate action, so each is paired with an explicit
  discharge path."
- **Six laws** (Table 2):
  - L1, commitment before measurement;
  - L2, freezing is irreversible and unforgeable;
  - L3, reports are not facts;
  - L4, evidence persists but verdicts do not;
  - L5, negative results are first-class;
  - L6, mechanical questions go to the framework and semantic judgment goes
    to the model, "in both directions".
- **Three observed failure modes** in "a capable model in an ordinary
  harness" across 35 campaigns:
  - Completion without delivery. A refused freeze was forged by re-saving
    with `frozen=true`.
  - "Gates that teach performance". A falsifiability gate that demanded a
    numeric threshold got an invented one ("a 30-year gap counts as
    weakening").
  - Judgment hard-coded as thresholds. A "five citations" rule was gamed by
    scattering identifiers.
- **"None of these is a model deficiency."** Each is a place where the
  harness asked for behaviour it could not verify, or verified the wrong
  thing.
- **"A mechanism that exists is not a mechanism that is wired."** A
  detector nobody reads, or a memory channel behind a query that is never
  issued, "measure as absent". Under bare dispatch, all node types but one
  received no memory at all.
- **A per-run verdict layer is a dead end, not a feedback loop.** The old
  quality-check layer was retired. Its checks survive only as detectors
  that feed the reviewer.
- **A candidate queue is the wrong memory design.** "The judgment 'is this
  worth keeping' is not decidable at the moment of writing, so a gate
  placed there admits and rejects with equal error."
- **The organization tier is a different thing, not "more important
  knowledge".** It is read by a project that does not yet exist, so it is
  written only by promotion. Promotion is a rewrite: de-projectize, make
  self-contained, and state the mechanism. `dead_end` is "the
  highest-yield" card type.
- **Delivery is unconditional.** Memory channels have no relevance test
  and no opt-in, "because a channel that requires the model to ask is a
  channel it will sometimes not use". Every channel has a constant byte
  budget instead (Appendix D).

## Methods

- **Node contract (§4.1, App. B).** A node is a directory holding one
  contract file: prompt and rules, a tool whitelist, skills, hooks,
  required output types, callable nodes, a post-run flow and a turn cap.
  Contradictions raise at load. One example is a whitelisted tool that the
  tool itself forbids to that node, which would otherwise fail silently as
  "the model ignores instructions".
- **One writer per piece of state (§4.2).** Reads across nodes are free.
  Cross-node writes are blocked by the sandbox. The commit gate is the
  authority, and the in-process check is only a witness.
- **Evidence-node admission criterion (§4.3).** A new evidence node needs
  "a distinct original sin that existing gates cannot catch", which gives
  three modalities:
  - experiment (fabricated execution), gated by a SHA-256 raw-results
    manifest;
  - observation (cherry-picking), gated by a frozen search protocol with a
    denominator and a mandatory adversarial search;
  - derivation (an invalid step), gated by a step-checking proof kernel that
    returns four values, not a boolean.
- **Inquiry contract (§5.1).** A question carries an optional proposition
  (a hypothesis if non-empty) and frozen closure conditions of exactly two
  kinds, numeric or statement. Fulfilment is matched by verbatim key
  against the frozen identifiers, and the first writer of a key wins. The
  tally returns *nothing* when nothing is frozen, which is kept distinct
  from zero fulfilled.
- **Artifact ledger (§5.2, App. C).**
  - Every save versions and hashes the artifact.
  - Freezing pins a version. Amending requires an `amendment_reason`, and
    the framework computes the diff itself.
  - `.frozen.jsonl` is append-only, and each row carries the SHA-256 of the
    previous row.
  - Save gates carry structural discipline and freeze gates carry
    promise-fulfilment discipline, because "discipline attached only to a
    voluntary action is discipline a run can opt out of".
  - A forgery guard refuses any save carrying `frozen:true` at the tool
    boundary.
- **Verdict authority (§5.3).** Evidence nodes return execution-level
  verdicts. Only Analysis records scientific verdicts. A producing node
  cannot flip a hypothesis status unless the research state already points
  that way. The referee scores seven dimensions, and `accept` is a
  precondition of freezing the manuscript.
- **Memory (§6.1).** `MEMORY.md` lives in the worktree and has four layers:
  - constitution: user authority, with cited fragments verified against the
    session's user messages;
  - situation: computed every run, never stored;
  - handbook: no admission gate, but every entry needs an `applies_to` and
    an evidence pointer, and near-duplicates increment a counter;
  - log: does not exist.

  Retirement is driven by falsifiability, not age, and is proposed but
  never automatic.
- **Knowledge base (§6.2).** Identifiers are content-addressed with scope
  in the identity, so project and organization records are different
  records. Facts need an external anchor (DOI, arXiv, PMID). Promotion
  has three gates: the project is in a terminal batch, the evidence closure
  is frozen throughout, and on the human lane the text has no project
  deixis. A mechanical lane and a human lane split the volume, giving
  roughly 5–15 human decisions per week.
- **Runtime (§7).**
  - **Context window as a rendered view.** The window is a rendering of an
    append-only log, with one live copy per (tool, args) pair and
    tombstones in place of deletion.
  - **Progress control.** Repeat aborts at 6 byte-identical no-change
    turns. Stall counts turns with no change to a workspace fingerprint and
    never terminates.
  - **Completion protocol.** "Not calling a tool is the declaration of
    completion." There is no `i_am_done` tool.
  - **Closing gates.** `on_before_finish` hands obligations back as a new
    turn.
  - **Dispatch gate.** It refuses to re-dispatch a blocked node while a
    content fingerprint of everything the node does not own is unchanged.
  - **Execution choke point.** A behavioural probe checks it at startup,
    and only the write boundary is fail-closed.

## Results

There is no evaluation, so this section lists the paper's measured
operating records and illustrations. All of them are single,
uncontrolled observations from the authors' own system.

- **Verdict-layer audit (§4.6).** One internal corpus of 324 transcripts
  under an earlier configuration:
  - 87 "check X failed on node Y" sequences, of which 11 (13%) were later
    corrected and **76 (87%) never passed again**;
  - writing node: 17 runs, 0 completions;
  - in one campaign of 3,972 tool calls, all 23 restart refusals came from
    a check failing four times in a row, and none came from a reviewer.
- **Candidate-queue memory audit (§6.1).**
  - Under bare dispatch, all node types but one received no memory.
  - **47% of 800 candidates were never processed**, and the oldest had
    waited 49 days.
  - Retirement was never invoked across 24 projects.
- **Write-time scope guessing (§6.2).** 224 organization-tier claims
  carried zero complete promotion provenance.
- **Bounded reads.** One KB query returned 83k tokens. Bounding reads to
  index rows cut working context from 88k to 12k tokens.
- **Orchestrator busy-wait.** Letting the model judge whether a node should
  continue was "measured once at 47 turns and 6.7M tokens".
- **C1, early-warning signals (assisted).** 50.3 h, at least 23 numbered
  investigator instructions, 67 runs, 2,021 turns, and 200.9M tokens, of
  which 160.7M were cache reads.
  - Writing reviews went major, major, approve-with-revisions,
    approve-with-revisions, approve (0.9). Manuscript v11 was frozen.
  - Of 4 questions, 3 were supported (one partially) and 1 was withdrawn
    into Limitations.
  - The original proposition was refuted by a stationary non-normal
    baseline that alarmed at about 0.03, against 0.94–1.00 with drift.
  - The mechanism failed in a nonlinear network (22 of 33 cells overlapped)
    and was reported as open.
  - One experiment run reached turn 153 and 17.5M tokens with nothing
    stopping it.
- **C2, 2D Ising Tc (unattended, 8.85 h of compute).** 800 Wolff runs.
  Both frozen hypotheses were refuted:
  - H1: Tc = 2.268776 ± 0.000148 against Onsager's 2.269185, a deviation of
    2.8σ against a 2σ criterion;
  - H2: compute ran to 4.4× the two-hour budget.

  **The first draft stated H1 as supported.** Its text described the
  weighted fit, but its number came from the unweighted fit (1.17σ).
  "Four reviewer rounds missed the inconsistency and a human caught it."
  Only then did the platform flip its verdict to refuted.
- **Theory before experiment.** A derivation froze Tc analytically, and an
  experiment started 131 s later returned a 0.42σ deviation. A script
  checks that the prediction's freeze time precedes the experiment's
  start.
- **Knowledge-base replay (§6.3).** The C2 project was replayed through
  closure:
  - 3 cards promoted, "all taken verbatim from the project's abstract and
    experiment log";
  - provenance chains intact (11 hops and 5 hops);
  - opening injection grew from 210 to 653 characters;
  - a reviewer red flag fired on the recorded dead end.

  The downstream effect "has not been separated from a ceiling effect".
- **Cost of a round (§10).** A smaller Ising instance: 131 min, 551 calls,
  5 human touches. Wall clock split as:
  - 26 min Monte Carlo;
  - 47 min experiment work before results;
  - 27 min closing the evidence;
  - 30 min review and orchestration.

  The preceding project had a 92.3% cache-hit rate (38.8M of 42.1M prompt
  tokens).

## Critique / open questions

- **The digest-framing pattern holds, with a twist (32/32).** There is no
  headline benchmark number to cherry-pick. The favourable cut is
  narrative, and it sits in the abstract's description of C2: "an
  unattended Monte Carlo study that refuted its own preregistered
  hypothesis against an exact solution."
  - §9.2 shows the platform first reported H1 as **supported**, with the
    number from the wrong fit.
  - The four reviewer rounds the paper presents as its adjudication
    machinery missed it.
  - A human caught it.

  The frozen preregistration made the error *correctable* once it was
  pointed out. It did not make it unrepresentable. This is the paper's
  best case against its own thesis, and it is under-weighted in the
  framing.
- **2 of 35 campaigns, selected for recoverability.** The authors say the
  two are "the two for which every artifact … is recoverable from disk".
  No outcome distribution over the other 33 is given. We don't know how
  many froze a wrong manuscript, deadlocked on a gate, or were abandoned.
- **The process-integrity suite is worse than "under construction".**
  §12: "four of the five axes of an early version failed to separate any
  two configurations, and its grader needed three corrections before its
  rankings were stable." That is a measurement instrument that does not
  yet discriminate. The digest's phrasing hides this.
- **The 87% figure measures dead ends, not a counterfactual.** The digest
  says "killed runs 87% of which never recovered". The source is 76 of 87
  failure *sequences* in one internal corpus of 324 transcripts under an
  older configuration, with no replacement-arm comparison. It shows the
  old check layer stalled runs. It does not show that write gates do
  better, because the replacement was never measured the same way.
- **One model family.** Every recorded run used DeepSeek V4 through
  OpenAI-compatible endpoints. The authors concede that the gates may be
  catching failures a stronger model makes less often. The anecdotes were
  all seen on that one family, so their frequencies don't transfer.
- **"Minimal nodes" means minimal node types, not a minimal surface.**
  - 140 tools and 50 skills (App. E);
  - the experiment node alone has 60 tools and 30 loop hooks (App. B,
    Table 3);
  - 11 contracts, each with many fields.

  The minimality claim is real but narrow.
- **Promotion by rewriting was not exercised in the one replay.** §6.2
  makes "a rewrite, not a move" the defining property. The §6.3 replay
  promoted three cards "taken verbatim from the project's abstract and
  experiment log", so the rewriting step had nothing to do.
- **No code, no artifacts.** App. E lists internal file paths
  (`<data-root>/projects/46da60b0-...`, `docs/PROPOSAL_RETIRE_QC_...md`),
  but there is no repository URL. The paper says "the installed personal
  edition", so it is a product. Nothing can be re-run.
- **Self-acknowledged gaps that matter here:**
  - no per-node spending ceiling (the 153-turn, 17.5M-token run);
  - the honesty rules do not catch an *unsound* validation protocol, such as
    selection and validation on the same sample;
  - the process is heavy for simple problems, with close to half of the
    wall clock spent on closing, review and orchestration;
  - the knowledge-base effect is unseparated from a ceiling effect.
- **Credit where due.** The disclosure discipline is unusually good:
  - each mechanism is stated with its invariant, failure, realization and
    cost;
  - "a sentence in a prompt is not a capability until its effect on
    behavior has been observed";
  - unknown prices are recorded as unknown, never zero.

  The design reasoning is specific enough to rebuild and argue with.
  Several "alternative we rejected" passages carry an internal measurement
  (the candidate queue, write-time scope, the verdict layer). These are
  weak evidence, but more than most system papers give.

## Trust signals

- **Credibility:** 2. Against the paper:
  - a company team (IdeaHorizon) with no academic affiliation and no
    venue;
  - an explicit non-evaluation with no benchmark and no ablation, and an
    integrity suite that does not yet separate configurations;
  - no code or artifacts, for a commercial product;
  - one model family;
  - two hand-selected campaigns out of 35.

  It is not a 1 for two reasons. The authors are candid about exactly
  these limits (§12). And the internal audits, although uncontrolled, are
  specific, internally consistent numbers taken from operating ledgers.

## Follow-up

- **Relevance:** 4. It is the most complete published design for an
  integrity-first, long-horizon research agent, and it maps closely onto
  this box. It gives concrete mechanisms to four existing concepts:
  - [[concepts/typed-enforcement]]: the eight-layer placement ladder,
    discharge paths and the five pre-gate questions;
  - [[concepts/evidence-gated-completion]]: tallies computed from ledgers,
    the writing gate over zero discharges, and no `i_am_done` tool;
  - [[concepts/llm-wiki-pattern]]: two-tier promotion, the
    candidate-queue failure, and constant-budget delivery;
  - [[concepts/verified-memory-writes]]: an organization scope reachable
    only via promotion.

  It is not a 5 because it anchors none of them evidentially. It is
  design, not measurement.
- **Directly relevant to this box's `raw/_candidates/` → `/curate`
  backlog.** AfS measured a candidate queue failing: 47% of 800 never
  processed, the oldest 49 days old, and retirement never invoked. Its
  diagnosis is that "worth keeping" is undecidable at write time. This
  box's digest → candidates → curate pipeline is structurally a candidate
  queue. AfS's alternative is admit-without-gate, with retirement driven
  by falsifiability and promotion at closure. Whether that applies to a
  literature hub, where a candidate is a pointer and not a lesson, is
  open. But the backlog-age metric is worth tracking (our inference).
- **Design points worth testing against this box's skills:**
  - **Dispatch fingerprint for blocked nodes.** Refuse to re-run a skill
    whose blocker report is unchanged and whose upstream is unchanged.
    This applies to scheduled jobs that re-fail identically.
  - **"Not calling a tool is completion" plus an `on_before_finish`
    closing gate.** This is the same shape as a Stop hook that hands
    obligations back.
  - **Unknown is never zero** in cost and cache ledgers. Compare
    `~/.claude/state.db` accounting.
  - **A stall counter on a workspace fingerprint, not on the transcript.**
    It is evidence, not a terminator. See [[concepts/budget-as-ceiling]].
- **Candidates:**
  - Ríos-García et al. 2026, "AI scientists produce results without
    reasoning scientifically" (arXiv 2604.18805). This is the 25,000-run
    study: scaffold 1.5% vs base model 41.4% of variance, and evidence
    ignored in 68% of traces. It is cited as the empirical premise and is
    not yet ingested.
  - MLR-Bench (arXiv 2505.19955), the "80% fabricated or invalid" figure.
    It is cited by xin2026eurekagent and has no note of its own.
