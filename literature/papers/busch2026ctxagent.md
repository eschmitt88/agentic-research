---
kind: paper
title: "CTXAgent: Self-Controlled Context Management of LLM Agents"
authors: ["Matthias Busch", "Marius Tacke", "Roland C. Aydin", "Christian Feiler", "Christian J. Cyron"]
institutions: ["Hamburg University of Technology", "Helmholtz-Zentrum Hereon", "German Research Center for Artificial Intelligence (DFKI)", "Saarland University"]
year: 2026
venue: "arXiv 2610.34425 (cs.AI), v1 2026-10-25, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.34425"
code_url: "https://github.com/MatthiasHBusch/CTXAgent"
citations: null
source: "raw/papers/busch2026ctxagent.pdf"
added: "2026-11-02"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-proprioception]]"
tags: ["compaction", "self-managed-context", "handoff-summary", "claude-code", "codex-cli", "opencode", "prompt-cache", "billed-cost", "standing-instructions", "planted-facts", "ceiling-effect", "harness-comparison", "scicode", "swe-bench-pro", "terminal-bench", "opus-5-5"]
---

# CTXAgent: Self-Controlled Context Management of LLM Agents

## TL;DR

The agent gets two tools and one paragraph of guidance and decides for
itself when to compact. `compact_context` replaces history with a hand-off
summary the model writes. `curate_context` replaces tool results the model
has acted on with one-line notes. Archived results stay reachable by
`recall`. Seven models run with and without the tools (arm A vs arm B,
both at a 1M window). Opus 5.5 and GPT-6.1 Sol also run through Claude
Code, Codex CLI and OpenCode on byte-identical sessions behind one logging
proxy that prices every request at list rates with cache accounting.

Three results matter here:

1. **Prompt-only self-management works for some frontier models and not
   others.** Opus 5.5, Sol 6.1 and Astra 6 compact every ~10 turns and
   keep mixed-session context near 50–80k. Haiku 5.5 and Muse Wonder never
   call `compact_context`. Sonnet 5.5 compacts in one session of three.
   Only Opus and Sol gain consistently (cost −43% and −31% on mixed
   sessions, small score changes).
2. **Against Claude Code, Opus 5.5 is much cheaper at a non-significant
   score difference.** Mixed sessions cost $4.80 per session under
   CTXAgent, $7.31 under Claude Code at an explicit 200k window and $9.35
   at Claude Code's default. Success was 0.969 vs 0.979 (McNemar 3/0,
   p = 0.25). For Sol the order reverses: CTXAgent costs 16% more than
   Codex at 200k and 2.2× on the marathon, because Sol writes long
   hand-offs.
3. **The retention instrument is at ceiling in every arm.** Planted facts
   and ephemeral observations were retained 1.000 in every
   distance × compaction-between cell of every arm. Standing rules were
   followed in >99% of replies on mixed sessions in every harness. So the
   paper cannot rank harnesses, or models, on what compaction drops. The
   authors say so.

## Claims

- Threshold harnesses ask the operator to pick one window for every task.
  The model is the only party that knows a task just finished, so it
  should choose when to compact.
- Li et al. 2026 (SelfCompact) found open-weight models need a rubric,
  admission gates and a backstop. This paper tests their conjecture that
  frontier models with better metacognition do not. Result: some do.
- With Opus 5.5 the self-managed harness "matches Claude Code on task
  success at 34% lower cost than its explicit 200k-window configuration
  and 49% lower than its default, with no context window to choose."
- Cost depends on each provider's cache-read ratio. Astra's cached input
  costs a tenth of fresh input and Fable's a fortieth, so the same
  behaviour saves about 50% for Astra and about 0% for Fable.
- Failure modes of self-management: not compacting at all (Haiku, Muse
  Wonder), compacting so often that cache invalidation eats the saving,
  and very long hand-offs (Sol, up to 5× the output tokens).

## Methods

- **Agent.** CTXAgent (commit 2a407b9), 1M window. A review hint is
  appended every 25 tool batches or 20k new tokens; the engine never acts
  on it. Age-based masking is a safety net; an automatic deep tier at
  0.9 × 1M was never reached. The guidance paragraph (App. A, verbatim)
  ends "Facts the user stated and standing instructions must survive
  both."
- **Arm B.** Same agent, engine off, context tools and the two prompt
  paragraphs removed.
- **Harnesses.** Claude Code 2.1.301, Codex CLI 0.163.0, OpenCode 1.19.3,
  each at native, 1M and explicit 200k windows where applicable. Web off
  and audited (zero hits). Effort high. Provider pinned via OpenRouter.
- **Probes.** Three planted kinds, each with a deterministic checker:
  stated facts probed by a direct question; ephemeral observations (the
  bench writes a file with a random id, the agent prints it, the file is
  deleted before the next turn); and two standing rules per session, of
  the form "end every reply with a line `REF: <token>`", checked by regex
  on every later reply. Probe distances are drawn from {3, 10, 25, 50, 90}
  turns, up to 200 in the marathon. A compaction is detected on the wire
  as a prompt drop of >25% and >10k tokens between requests.
- **Session families.** Mixed: 3 seeds, 101–115 turns, 34 Aider exercises
  plus 6–7 SciCode problems, 32 fact and 32 ephemeral probes per arm.
  Clean chains: 5 SciCode chats, 2 reps, no probes. Marathon: 1 session,
  212 turns, ~1.5M tokens raw. Hard: 3 seeds, 35–36 turns, 3 SWE-bench Pro
  + 3 Terminal-Bench 4.0 + 3 SciCode-Verified per seed.
- **Statistics.** The unit is the session. Paired exact McNemar on
  (seed, turn). Costs are reported as means with seed ranges. No test is
  reported for the seven-model A/B comparison.

## Results

- **Seven models, mixed sessions (Table 4, arm A vs B).**

  | Model | Cuts/session | Cost A / B ($) | A/B | SciCode steps A / B (of 80) |
  |---|---|---|---|---|
  | Haiku 5.5 | 0.3 | 2.71 / 2.72 | 0.99 | 50 / 49 |
  | Sonnet 5.5 | 1.0 | 4.50 / 5.37 | 0.84 | 65 / 73 |
  | Opus 5.5 | 3.7 | 4.80 / 8.37 | 0.57 | 77 / 80 |
  | Fable 5.1 | 3.0 | 19.46 / 19.14 | 1.02 | 70 / 69 |
  | Sol 6.1 | 7.7 | 5.03 / 7.30 | 0.69 | 71 / 72 |
  | Astra 6 | 8.3 | 31.61 / 60.39 | 0.52 | 71 / 76 |
  | Muse Wonder | 0.3 | 17.16 / 14.35 | 1.20 | 53 / 56 |

  Opus compacts at a median 72k (40–80k) and curates hardly at all
  (11 compacts, 2 curates over three sessions). Arm B grows to 230–430k.
- **Hard sessions (Table 5, 51 scored items per arm).** Opus A/B 0.84
  and 39 vs 35 passed. Sol 0.42 and 37 vs 34. Fable 0.81 and 31 vs 35.
  Sonnet 1.15 and 32 vs 32. Peaks under arm A reach 222k for Opus and
  215k for Fable, so "below 100k" holds for mixed sessions only.
- **Harnesses, mixed sessions (Table 6, per seed).**
  - Opus 5.5, Claude Code default: 0 cuts, peaks 255–277k, $9.02–9.94.
  - Claude Code 1M: 0 cuts, peaks up to 688k, $9.24–17.04.
  - Claude Code 200k: 1 cut per session at about 168k, $7.01–7.73.
  - CTXAgent: 3–4 cuts, peaks 72–80k, $4.64–4.92. Every CTXAgent seed is
    below every Claude Code seed in each window condition.
  - Sol 6.1: Codex 200k $3.84–4.64 (1–2 cuts) vs CTXAgent $4.22–6.28
    (7–8 cuts).
- **How "34% / 49%" is computed.** Mean cost per mixed session, Opus
  only: 1 − 4.80/7.31 = 34% and 1 − 4.80/9.35 = 49%. The same numbers
  appear as "+52%" and "+95%" from the other side in Fig. 5b. Request
  counts are matched (233 vs 228). The "default" arm never compacted in
  any session, so the 49% is against no compaction at all. On hard
  sessions Claude Code 200k costs +39% vs CTXAgent with 5% lower score,
  but OpenCode at 200k with Opus is 13% *cheaper* than CTXAgent.
- **Marathon (1 session).** Opus: CTXAgent $14.17 (11 cuts, peak 96k) vs
  Claude Code 200k $20.83 (5 cuts) vs OpenCode $22.75. Sol: CTXAgent
  $26.29 (38 cuts) vs Codex $12.14. Every memory item kept in every arm.
- **Clean chains, no context pressure (Table 3).** Opus under CTXAgent
  made 0 cuts and matched Claude Code (78/86) at lower cost. Sol under
  CTXAgent cut 7 times with no need and passed 62/86 vs Codex's 68/86.
- **Retention (App. C, Fig. 8).** 1.000 in every cell on mixed sessions.
  Across all sessions, one planted item was lost, by OpenCode with Opus on
  a hard session. Compaction-between
  observations on mixed sessions: Claude Code 200k 301, CTXAgent-Opus
  507, Codex 200k 348, CTXAgent-Sol 552. Rule adherence: >99% of replies
  on mixed sessions in every harness and every model except Muse Wonder
  (83% A, 78% B); 80–99% across models and 74–99% across harnesses on the
  hard sessions, with no split by harness, model or compaction.
- **Recall.** Over three mixed sessions per model, `recall` was called 3
  times by Opus, 2 by Haiku and never by the rest. On the hard sessions,
  once each by Haiku, Opus and Fable.

## Critique / open questions

- **Against the digest framing: below framing.** The cost numbers are in
  the body and the arithmetic checks. The digest ranked this paper 5 for
  a different reason: that it measures standing-instruction retention
  across compaction deterministically, for the pending
  `precompact-chat-only-constraints` proposal. It does not do that in any
  usable way.
  1. **Ceiling.** Every probe was retained 1.000 and rules were followed
     >99% on mixed sessions in every arm, so there is no contrast between
     Claude Code's compaction and self-management, or between categories.
  2. **Few compaction events behind the Claude Code number.** At 200k
     Claude Code compacted once per mixed session, so its 301
     compaction-between observations rest on 3 summaries, plus 5 on the
     marathon. At its default window it never compacted at all.
  3. **The rule probe is not salle2026benchmarks' kind of constraint.** A
     format rule exercised in every reply puts a fresh copy into the
     transcript every turn. Process and scope instructions ("how to
     commit", "don't touch X") apply only now and then, so before a
     compaction they may have no recent copy.
  4. The only non-ceiling rule numbers (74–99% on hard sessions) are not
     split by harness, model, or before vs after compaction.
  The digest's "live context mostly under 100k" holds for mixed sessions
  only. The abstract's "scores mostly do not drop significantly" has no
  reported test behind it for the seven-model comparison.
- **What it means for salle2026benchmarks.** It neither corroborates nor
  contradicts it. Our reading is a mild narrowing. A chat-only *process*
  rule (how to end every reply), given at turn 2, survived Claude Code's
  own summary in all 8 observed compactions and every Opus hand-off. That
  fits the view that what compaction drops is instructions *not
  re-exercised* between being given and the cut. It does not test that
  view.
- **The CTXAgent prompt already contains the proposal's sentence.** The
  guidance ends "Facts the user stated and standing instructions must
  survive both." Claude Code, without such a line, also hit ceiling, so
  the line's effect is not identified. salle2026benchmarks found a generic
  ask in the summary prompt insufficient for GPT models.
- **Internal inconsistency.** Tables 4 and 5 define "memory" as facts,
  ephemerals, probes *and rules*, and report 1.000 nearly everywhere. The
  text gives Muse Wonder rule adherence of 83% / 78% and hard-session
  adherence of 80–99%. The memory column cannot include per-reply rule
  adherence as defined. The released rows would settle which is right.
- **Small n.** Three seeds per cell, one marathon, two reps of the clean
  chains. The Opus cost gap is outside the seed spread. The success
  differences are not significant, and the paper says so. "Matches on
  task success" means "not significantly different at n = 3", with Claude
  Code ahead by 1 point.
- **Confounds the authors name.** Each vendor CLI runs only its own
  vendor's model, so harness and model are confounded across vendors. The
  system prompt is not tuned. Prices are one provider's list on one date.
- **Ephemeral observations survived everywhere.** The value is gone from
  disk after its turn, so it is not re-observable. It still survived
  every summary, plausibly because the agent printed it in its own reply.
  That fits [[literature/papers/lencevicius2026when]]'s finding that a
  fact in the agent's own output is carried like a derived fact. It
  does not test lencevicius's observed-only case.
- **Fit with the graph.** It extends [[literature/papers/li2026acm]],
  where untrained GPT-5.5 barely used the same kind of tools. Here a
  prompt alone is enough for Opus, Sol and Astra, and not for Haiku,
  Sonnet or Muse Wonder. Recall was almost never used, like the 56% of
  recall-enabled settings in [[literature/papers/fan2026empirical]] that
  never called it. The cache-economics argument matches
  [[literature/papers/bai2026how]] and
  [[literature/papers/nguyen2026cliffcompaction]] (about 51% below Claude
  Code auto-compaction).

## Trust signals

- **Credibility:** 3 — agent, benchmark, harness configs and all result
  rows released; byte-identical sessions, wire-level compaction detection
  and per-seed tables; but 3 seeds, a ceiling instrument, an internal
  memory/rule inconsistency, not peer reviewed, and a computational
  mechanics group outside the core agent community.
  - For: one proxy prices every harness the same way; vendor versions
    pinned; web access audited; a limitations section that names the
    ceiling and the harness–model confound; unsolvable SciCode steps
    documented and held constant (App. B).
  - Against: one author did all the experimental work (CRediT); no tests
    on the seven-model comparison; the Claude Code default arm never
    compacted, so "default compaction" is not actually measured.

## Follow-up

- **Relevance:** 4 — it runs this box's harness and model (Claude Code
  with Opus 5.5) against a self-managed alternative on billed cost, and
  adds material evidence to [[concepts/context-eviction-policy]] and
  [[concepts/constraint-pinning]]. Not 5, because the retention
  measurement the digest wanted is at ceiling.
- **For the pending proposal.** No new evidence either way. Gate 1 still
  rests on salle2026benchmarks, chen2026governance, lencevicius2026when
  and nakayashiki2026when. Two box-relevant facts: (a) at Claude Code's
  default window, Opus 5.5 mixed sessions reached 277k without one
  compaction, so PreCompact may fire rarely in practice; (b) a rule the
  agent applies in every reply appears to survive compaction without
  help, which narrows the exposed class to intermittent process and
  scope instructions. That is the class the proposal already names.
- **Cheap check.** Run the released bench with one rule that applies
  only on rare turns (for example "when you create a file under X, do Y")
  instead of on every reply, at Claude Code 200k. That would give the
  measurement the digest hoped for.
- **Candidates** (cited, not yet in the graph):
  - Li et al. 2026, "Self-compacting language model agents" (SelfCompact,
    arXiv 2606.23525). Also listed as a candidate in salle2026benchmarks.
  - Wang et al. 2026, "Lost in context compaction" (arXiv 2608.11242).
    Side constraints survive about 17% of the time.
  - Lindenbauer et al. 2025, "The complexity trap" (arXiv 2508.21433).
    Observation masking matches LLM summarization at half the cost.
  - Ariño de la Rubia & Pafka 2026, "Identical runs, different results"
    (arXiv 2609.33812). Run-to-run cost variance larger than harness gaps.
