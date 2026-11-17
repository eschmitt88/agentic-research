---
kind: paper
title: "Unprompted coordination through shared memory in LLM agents: a locked two-implementation replication and extension"
authors: ["Pekka Abrahamsson", "Muhammad Waseem", "Esa Karjalainen", "Petrus Jussila", "Henri Hansen", "Jussi Rasku", "Bilhanan Silverajan", "Kai-Kristian Kemell"]
institutions: ["Tampere University (GPT-Lab)"]
year: 2026
venue: "arXiv 2611.12412 (cs.MA), v1 2026-11-09, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.12412"
code_url: "https://doi.org/10.5281/zenodo.23965306"
citations: null
source: "raw/papers/abrahamsson2026unprompted.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/information-firewall]]"
tags: ["stigmergy", "shared-state", "key-value-store", "unprompted-coordination", "no-channel-control", "private-store-control", "prepopulated-store", "recheck-sentence", "read-back", "key-collision", "cross-agent-access", "preregistration", "hashed-locks", "replication", "implementation-sensitivity", "parser-strictness", "request-path", "reasoning-effort", "multi-vendor", "toy-task"]
---

# Unprompted coordination through shared memory in LLM agents: a locked two-implementation replication and extension

## TL;DR

Four agents share one key-value store that each prompt describes as the
agent's own scratch space. Two of them hold complementary halves of one
arithmetic computation. No prompt mentions other agents. Across nine
commercial models, joint success happened only when one agent read the
other's intermediate from the store. In 1,214 main-study worlds without a
cross-agent read, joint success never occurred. Whether a model
"coordinated" was set by the **writer** (did it store its half at all) and by
**timing** (did the reader list the store again after the partner wrote). It
was not set by reader reluctance: once a listing exposed another agent's
entry, a read followed in 493 of 501 worlds. For gpt-6-astra, deleting
"check your workspace again before submitting" from the prompt took
cross-agent access from 50/50 to 1/50 in both implementations. A second
researcher rebuilt the environment from a written protocol under hashed locks.
The qualitative pattern recurred, but one cell moved 28 points, one was not
estimable, and the authors' own rewording (Phase B) coordinated less. The
paper's practical conclusion is "a model's reticence is not a safeguard;
separation is."

## Claims

- Agents never told that others exist read and use each other's
  intermediates from a store presented as their own (RQ1).
- Whether this happens depends on the model but is not ordered by vendor
  tier. Within the tested models, it depends on whether the agent writes its
  intermediate and whether the reader looks again after the partner's write
  (RQ2).
- The shared store is the channel. A private store with an identical
  interface removes joint success, and a no-store control confirms that no
  other route exists.
- A generic workflow sentence governs discovery: without "check your
  workspace again before submitting", gpt-6-astra's agents list once at
  step 0 (store empty) and never again.
- The evidence supports *access* and *causal influence*, not *intent*.
  Agents write at the same rate whether or not the store is shared, so
  nothing indicates they write in order to be read.
- Three implementation details changed measured behaviour enough to alter a
  cell: parser strictness, a reply-format difference between request
  paths, and key collisions that an access metric counts as coordination.
- Practice: treat any cache, scratch directory or memory service shared
  between agents as a communication channel. Separate it per agent where
  sharing is not intended, namespace it where it is, and log with
  authorship. Research: lock the parse rule, route, effort and temperature
  actually sent, and record the model identifier returned per call.

## Methods

- **World:** 4 agents, one key-value store, at most 20 steps (one operation
  each: list, read, write, submit). An asynchronous scheduler runs each
  agent in its own loop with no barrier. Each agent sees only its own step
  history. Two interdependent agents get halves of one computation. In
  *digits*, each is told the number whose digit sum is its half (C_A or
  C_B), and the answer is C_A × C_B. In *remainders*, the answer is
  R_A + R_B. Two self-contained agents get standalone arithmetic tasks and
  provide a collision baseline. Task order is shuffled by seed, so agent
  index carries no information.
- **Store framing (reference):** "a persistent key/value store attached to
  this workspace", "so that long assignments do not have to recompute
  intermediate work". A listing returns key names only. Entries carry no
  author. The key names that matter (C_A, C_B, R_A, R_B) come from the task
  text. Every reference and Phase A prompt carries a persistence sentence
  ending "and check your workspace again before submitting".
- **Measures:** *cross-agent access* = any agent reads a key whose current
  value another agent wrote (Heylighen's minimal stigmergic event). *Joint
  success* = both interdependent agents submit the correct answer. The
  paper also counts *write* (an interdependent agent wrote) and *exposure*
  (a list returned another agent's entry). It separates access from
  influence (tested by intervention) and from intent, following Lowe et al.
  2019. Rates are counts of valid worlds with 95% Wilson intervals.
  Contrasts use Newcombe intervals, no multiplicity correction, and no
  equivalence claims.
- **Models (9, 3 providers):** gpt-6-astra, gpt-5.4, gpt-5.4-mini,
  gpt-5-nano, claude-haiku-4-5, claude-sonnet-5, gemini-3.1-pro-preview,
  gemini-3.8-flash, gemini-3.7-flash. Settings: 4,096-token output ceiling,
  effort low, temperature 1.0 (the reference omits it on the OpenAI route).
  A cell is 5 runs × 10 worlds = 50 worlds.
- **Two implementations.** The *reference lab* is a browser web app with
  Postgres, built with Lovable, calling providers via a Lovable gateway or
  directly. It runs a monitor model per step whose labels no agent or
  measure reads. The *replicator* is a command-line runner via OpenRouter,
  pinned to one upstream provider per model. A second researcher rebuilt it
  from a written protocol without access to the original system. **Phase
  B** (conceptual) used the replicator's own task wording and prompts.
  **Phase A** (direct) used the original wording. Controls: a *no-store*
  condition (the store paragraph and operations removed, task and
  persistence sentence unchanged).
- **Locks (Table 1):** SHA-256 manifests, with hashes emailed before data.
  L1 (Phase B design) 2026-10-08, L2 2026-10-09, L3 (Phase A) 2026-10-31,
  reference L0/L0a 2026-10-31, outcome exchange 2026-11-01.
- **Extension E1 (preregistered 2026-11-03, run 11-03/04 in both
  implementations):** store scope (shared vs private: same interface, own
  namespace per agent) × persistence sentence (with vs without the recheck
  clause), for gpt-6-astra, gpt-5.4-mini and claude-sonnet-5. A
  *prepopulated* arm seeds each interdependent agent's half under its
  task-named symbol before step 0, with no author (gpt-6-astra,
  claude-sonnet-5, gemini-3.7-flash, gpt-5.4-mini). It uses a lockstep
  **barrier** scheduler. 16 cells × 800 worlds per implementation.
- Cost: the replicator's 16 E1 cells cost USD 43.60 via OpenRouter.

## Results

- **Main study, coordinating / valid worlds (Table 2, reference):**
  gpt-6-astra 50/50, gpt-5.4 50/50, claude-haiku-4-5 40/40, gpt-5-nano
  45/49 (0.81–0.97), gpt-5.4-mini 26/49 (0.39–0.66), claude-sonnet-5 2/50,
  gemini-3.8-flash 6/96, gemini-3.7-flash 0/100, gemini-3.1-pro-preview
  0/50. No-store: 0/49 (gpt-5-nano), 0/50 (claude-sonnet-5). Within
  providers, tier does not order the rate: gpt-5-nano 45/49 against
  gemini-3.1-pro 0/50, and haiku 40/40 against sonnet 2/50. The Anthropic
  split is confounded with effort in the reference (haiku ran at provider
  default after an effort-parameter rejection) but recurs in Phase A, where
  both ran at low.
- **Joint success (Table 3, reference):** gpt-6-astra 48/50, haiku 30/40,
  gpt-5.4 34/50, gpt-5-nano 15/49, gpt-5.4-mini 12/49, the rest 0–1.
  **Without coordination it was 0 of 414 non-coordinating reference worlds
  and 0 of 800 across all 27 replicator files.** Removing the store took
  gpt-5-nano from 15/49 to 0/49 (reference) and from 8/50 to 0/50 (Phase A).
- **Phase A (direct replication):** every estimable 95% interval overlaps
  the reference except gpt-5.4-mini: 7/28 against 26/49, −28 points (95%
  −46 to −5). gpt-5.4 lost 49 of 50 worlds to invalid replies and is not
  estimable.
- **Phase B (own wording):** lower across the board. gpt-6-astra 41/50,
  gpt-5.4 2/33, gpt-5.4-mini 0/50, gpt-5-nano 17/50, haiku had no valid
  world (parser), and all Gemini and sonnet cells were 0/50. Seventeen
  gpt-5-nano worlds coordinated without a single list operation (see
  collisions below).
- **Writing, exposure, reading (Table 5, §5.5):**
  - *Looking is universal.* At least one agent listed the store in 533/534
    reference and 370/375 Phase A worlds.
  - *Reading is universal once exposed.* A read followed exposure in 213/215
    reference and 280/286 replicator worlds (493/501), for every model with
    any exposed world. Without exposure, 35 of 894 worlds coordinated.
  - *Writing is the variable.* Interdependent agents of gemini-3.7-flash
    wrote in 0/100 reference worlds, gemini-3.1-pro 1/50, sonnet 9/50. The
    partner never wrote for 80–100 of 100 agents in the low-writing cells.
    gpt-5-nano, gpt-6-astra and haiku wrote in every world.
  - *Timing mattered less than writing.* Agents that submitted before a
    later partner write numbered 0/100 for gpt-6-astra and haiku, 14/100
    for mini, and 8/100 for sonnet.
- **Effort (§6.2):** gpt-5.4-mini at effort none coordinated 22/30, against
  7/28 at low, on the same route, prompts and seeds. Writes went 26/30 →
  10/28. On 17 seeds valid in both runs: 12 vs 5, exact McNemar p = 0.039.
- **Key collisions (§6.3):** in Phase B, 61 of gpt-5-nano's 76 cross-agent
  reads were of a generic key (calc, temp) that the reader itself had
  written earlier and another agent overwrote. Counting only reads of keys
  the reader had not written takes that cell from 17/50 to 2/50, and the
  original Phase A nano cell from 19 to 15 of 22. In the reference, 18 of
  615 cross-agent reads were read-backs of the reader's own key, and 396
  used task-named keys.
- **E1 contrast a, sharing (Table 6):** making the store private took
  gpt-6-astra's joint success from 49/50 to 0/50 (reference; +98, 95% +87
  to +100) and from 47/50 to 0/50 (replicator; +94, +82 to +98). Writes
  stayed 50/50, and agents issued *more* operations re-listing an empty
  namespace (6.2 vs 5.0 per agent, reference). gpt-5.4-mini went 16 → 0
  (+32, +19 to +46) in the reference and 3/29 → 0/25 in the replicator (CI
  crosses zero).
- **E1 contrast b, the recheck clause:** gpt-6-astra access went 50/50 →
  1/50 in *each* implementation (+98, +87 to +100), and joint success 49 →
  1 and 47 → 0. Without the clause, all 100 gpt-6-astra agents listed only
  at step 0, before the partner's median step-1 write (Table 7: 99/97
  listed after the partner's write with the clause, 0/0 without). The
  clause also acted on writing: without it, astra's interdependent writes
  fell to 28/23 of 50, because three-quarters of its agents never wrote
  after finding the store empty. gpt-5.4-mini in the reference: access
  33 → 11 (+44, +25 to +59), joint 16 → 5, writes unchanged (33 vs 35). In
  the replicator, no effect (6/29 vs 6/39, +5, −13 to +25). claude-sonnet-5
  showed nothing to move (2 vs 1, 2 vs 4), because its partner never wrote
  in 86–90 of 100 agents.
- **E1 contrast c, prepopulated store:** with the partner's half already
  stored, all four models retrieved it: 100/100 first reads for gpt-6-astra,
  sonnet and gemini-3.7-flash in both implementations, and 100/100 and
  64/64 for mini. Joint success was 50, 48, 50, 48 of 50 (reference) and
  50, 50, 50, 32 of 32 (replicator). So near-zero access in low cells is not
  an inability to retrieve.
- **E1 contrast d, barrier schedule:** results stayed close to the
  asynchronous cells, e.g. astra 50/50 vs 50/50, and sonnet 2/50 vs 2/50
  (reference). The authors call this a robustness result, not a null: mini
  intervals admit 20–30-point differences.
- **Intent:** shared and private writing rates match (astra 50 vs 50 with
  the clause, 28/23 vs 27/20 without; mini 33/35 vs 33/31), so agents did
  not write differently when a reader could exist. A post hoc search of
  reasoning found explicit reference to another party in 21 of 381
  interdependent agents that did a cross-agent read. 14 of those 21 were
  haiku ("R_B (which should be in the cache from another worker)"), and 20
  of 21 came before any exposure. So the attribution came from the task
  text, not from seeing an entry.
- **Implementation sensitivity (§6):**
  - Phase B's whole-reply-must-be-JSON parser invalidated 309 haiku replies
    that a first-to-last-brace cut would have accepted in 308 cases, and
    removed the model.
  - On OpenRouter, gpt-5.4 (235) and gpt-5.4-mini (183) returned several
    chained JSON actions per reply. This never happened on the reference
    route (0/897, 0/690 steps). A matched-request diagnostic reproduced it:
    multi-object replies were 0/50 via api.openai.com, against 7/50 and
    8/50 via OpenRouter with and without temperature.

## Critique / open questions

- **The task hands the agents a shared vocabulary.** Both halves are named
  C_A/C_B in each task text, and 396 of 615 reference cross-agent reads used
  those task-named keys. "Unprompted" means no prompt mentions other
  agents. It does not mean the agents had to guess where the partner's
  value would be: the task says a value named C_A exists and is "missing".
  Phase B, which let agents choose key names, coordinated far less, and its
  one high-ish non-astra cell (nano 17/50) was mostly collisions. The paper
  says this itself (§9 External validity). The headline rates are rates for
  a world where the key names are pre-agreed by the task author.
- **The recheck sentence is in every main-study prompt.** All main-study
  rates are therefore rates *with* an invitation to look again. E1 removes
  it for only three models, two of which barely coordinate anyway. "One
  sentence switches it on and off" is shown cleanly for one model
  (gpt-6-astra). gpt-5.4-mini is partial in the reference and null in the
  replicator, and sonnet is untestable.
- **Toy scale.** 4 agents, 20 steps, single-number arithmetic, a key-value
  store with no authorship. It is far from the motivating METR incident
  (~1,200 agents over days, a package-cache namespace used as a message
  board). The paper shows the minimal mechanism exists. It shows nothing
  about prevalence, about defect propagation, or about stores where entries
  carry provenance.
- **Only benign, task-relevant information flows.** No entry is wrong,
  adversarial or off-task. This is information flow, not contagion. Whether
  agents would equally use a *wrong* partner value is untested, although
  the prepopulated arm's near-100% uptake of seeded values suggests no
  scrutiny.
- **Two implementations, one group.** The replicator is a co-author at the
  same lab. The reference author designed the original study and wrote the
  Phase A package, so Phase A tests implementation, not design
  independence. The lock order slipped. L0 (reference) was locked after L3,
  and 8 of 11 reference cells were run on 31 October, after an early Phase B
  table reached the reference author on 9 October. The prompts and
  parameters had been fixed earlier, and Phase B is the cell set that
  disagrees most, so the leak would not obviously favour agreement. But
  "locked before the data" is weaker than the abstract suggests for the
  reference side.
- **The reference environment's source is not released.** It "continues
  in development". Reproduction goes through the replicator's runner and
  protocol. The reference lab did not record the returned model identifier
  per call.
- **Wide intervals, no correction.** Cells are 50 worlds, and the
  between-implementation comparisons are descriptive only. The effort
  confound (§6.2, mini 22/30 vs 7/28) shows that one unintended parameter
  moves a cell more than most contrasts of interest, which limits what any
  single-cell rate means.
- **Not shown:** whether a store with authorship metadata, namespacing by
  default, or a "this is private" claim that agents can verify changes the
  read rate. Whether agents *write for* others under any design. Whether
  any of this holds for open-weight models (named as future work).

## Trust signals

- **Credibility:** 3. A university group (Tampere GPT-Lab), unreviewed arXiv
  preprint. Unusually strong process: hashed preregistration locks, an
  independent rebuild from a written protocol, a deviations log, and a
  replication package on Zenodo (runner, protocol, manifests, step logs,
  recount scripts). Held back from 4 by the same-group replication, the
  late reference locks, the unreleased reference source, and a toy task.
- Reports every exclusion and both valid-world and all-launched
  denominators (Table 4a). States that it makes no equivalence claim.
- Discloses AI use: the reference environment was built with Lovable, and
  Claude assisted the analysis scripts, recounts and drafting.
- Funded by Business Finland (ANSE) and the Jane and Aatos Erkko Foundation
  (EVIL-AI).

## Follow-up

- **Relevance:** 4. This is the cleanest controlled evidence for the
  premise under [[concepts/shared-substrate-contagion]]: a shared store is
  a channel whether or not anyone designed or announced it. Its private-store
  and no-store arms are the "no-channel arm" that concept demands. It adds a
  mechanism that matters for [[concepts/file-as-bus]]: the bus carries
  information only if writers persist intermediates and readers re-list
  *after* the write. A toy task and benign flow keep it from 5.
- **For this box (our inference):**
  - Sessions, scheduled jobs and subagents here share repos, `_meta/` and
    `/tmp` paths named by convention (`NOTES.md`, `raw/_candidates/`,
    `/tmp/curate-drafts/`). That is the paper's "task-named key" condition:
    a shared vocabulary makes discovery trivial.
  - The paper's discovery gate (look again after the partner writes) is
    open by construction here. CLAUDE.md, NOTES.md and imported concepts
    load at every session start, so every session re-lists. This matches
    [[literature/papers/das2026selfpropagating]]'s memory-open condition.
  - The paper's remedy maps to existing practice: per-agent namespaces
    (one drafts file per reader, as in this curate pass) and authorship
    logging (git). Its warning maps to `/tmp` scratch paths, which carry no
    author.
- **Measurement lesson:** an access metric over shared keys must exclude
  read-backs of the reader's own key, or collisions read as coordination
  (gpt-5-nano Phase B: 17/50 → 2/50). Any audit of cross-session reads here
  (for example, "did session B use session A's draft?") needs authorship
  per write, which git provides and `/tmp` does not.
- **Candidates** (cited, not yet in the graph):
  - METR (Greenblatt, Rao, Cotra, Wijk) 2026, "Brief independent
    investigation of agents' behavior, reasoning and collaboration in the
    OpenAI / Hugging Face hacking incident" (METR blog, 2026-08-26). This is
    the naturalistic instance; [[literature/papers/lee2026unfinishedbench]] also cites the
    incident.
  - Park et al. 2026, "Scaling discovery through test-time communication"
    (arXiv 2609.21032). Shared directory, agents told peers exist.
  - Jones & Hauert 2026, "Emergent culture in minimal LLM systems" (ALIFE
    2026, arXiv 2606.30668). A decaying shared text store.
  - Tessera et al. 2026, ALEM (arXiv 2606.08340). Coordination competence
    vs individual competence across 13 models.
  - Riedl 2026, "Emergent coordination in multi-agent language models"
    (ICLR 2026, arXiv 2510.05174).
