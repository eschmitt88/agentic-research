---
kind: paper
title: "AgentDiscover: Autonomous Discovery with Minimal Search Scaffolding"
authors: ["Mahdi Farahbakhsh", "Ilan Sela", "Fatemeh Doudi", "Vishnu Teja Kunde", "Krishna Narayanan", "Jean-Francois Chamberland", "Dileep Kalathil"]
institutions: ["Texas A&M University"]
year: 2026
venue: "arXiv 2610.05334 (cs.AI/cs.LG/cs.NE), v1 2026-10-04, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.05334"
code_url: "https://github.com/mhdfb/AgentDiscover"
citations: null
source: "raw/papers/farahbakhsh2026agentdiscover.pdf"
added: "2026-10-13"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/typed-claim-partition]]"
tags: ["agent-owned-search", "bitter-lesson", "evolutionary-search", "map-elites", "mcts", "graph-database-memory", "neo4j", "mcp-server", "context-reset", "session-flush", "meta-agent", "skill-writing", "steering-message", "evaluator-hardening", "reward-hacking", "claims-vs-facts-provenance", "matched-spend", "coral", "atcoder-heuristic", "kernel-optimization"]
---

# AgentDiscover: Autonomous Discovery with Minimal Search Scaffolding

## TL;DR

AgentDiscover replaces the fixed selection algorithm of AlphaEvolve-style
loops with a coding agent (Claude Code or Codex) that plans the search
itself. Every idea, candidate, session and lesson goes into a Neo4j graph
behind an MCP server. The agent reads it with any read-only Cypher query
and writes only through typed tools. MAP-Elites parent selection is then a
single query the agent may use, combine or ignore. Before submitting, the
agent runs its own local experiments. The server scores each submission in
a hidden sandbox and replies with a templated steering message. Sessions
end when the agent stalls or when its context costs more than a fresh
start. A fresh agent then takes over the same graph, and an optional meta
agent turns past transcripts into skills and strategy notes.

Across 24 tasks it reports best or tied-best scores against OpenEvolve,
ShinkaEvolve, EvoX, CORAL and TTT-Discover. The cleanest matched-spend
result is Anthropic's kernel take-home, at 960 vs 1,098 cycles. The work
that transfers best is the infrastructure: claims-vs-facts provenance and
evaluator hardening. The central claim, that *agent-owned search* beats a
fixed algorithm, is never isolated. All main results are single runs.

## Claims

- **The Bitter Lesson applies to the search loop.** A strategy fixed
  before the run cannot use the model's knowledge of alternatives. A
  coding agent can implement MAP-Elites or UCT itself, so "a stronger model
  improves not only the candidates but also the search that produces
  them."
- **Retrieval as a query keeps long-horizon context cheap.** The authors
  contrast this with Meta-Harness, which reads a median of 82 files per
  iteration. Here, a fresh agent can re-explore the history "cheaply
  through queries".
- **Context resets help.** After a stall of more than five candidates, a
  reset raises both the chance and the size of an improvement, and a meta
  pass raises them further (Fig. 4c–d).
- **Meta-agent guidance helps.** In 15 paired forks, the guided agent beats
  the pre-fork best within two candidates 73% of the time, against 33%
  unguided.
- **Parallel agents save wall-clock time, not money.** Four agents
  converge sooner in hours, with no clear difference at equal spend.
- **Several published evaluators can be reward-hacked.** Appendix C
  documents three failure classes and corrects them.

## Methods

- **Components.**
  - Search agent: a coding-agent CLI in a sandboxed git worktree that holds
    `solution.py`, `tools/` and `skills/`. Its instruction file is a
    CLAUDE.md template filled with the problem, graph schema, folder
    listing and strategy.
  - Long-term memory: a Neo4j graph with node labels User, Agent, Session,
    Candidate (keyed by SHA-256 of the program), Idea, Resource, Guidance,
    Tool and RunConfig.
  - Meta agent: runs between blocks. It reads transcripts read-only and
    never sees the evaluator, genomes or git history. It writes meta
    lessons, saved Cypher query tools, skills and a new Guidance node.
- **Facts vs claims (Table 6).** Edges written by the orchestrator or
  server record what happened (RETRIEVED and READ are rebuilt from the
  session log). Edges written by the agent, such as DERIVED_FROM and
  USED_CONTEXT, are stored as *claims*. Each claim has a logged
  counterpart, and the server checks claims only for existence. The
  prompt says this outright: "your declared parents, idea, and resources
  are recorded as your claims."
- **Session loop.**
  - The agent queries, records an idea, experiments, then submits with
    both a linguistic and a numeric fitness prediction.
  - The steering reply is code-generated, not an LLM. It gives the score
    against the agent's prediction, the gap to a named target, candidates
    left, time left, and from the fifth candidate a cost note: "a fresh
    session would start from ⟨c + i⟩ … Continue only if you have a great
    idea".
  - The search agent has no web access, except on AC1/AC2.
- **Safeguards (A.4).** The evaluator is never mounted. The server scores
  a copy of `solution.py` in a separate container with no network. Hidden
  data is pre-cached. Reads go through a Neo4j read transaction and writes
  through fixed tools. Calls are attributed by the server, not by what the
  agent says about itself.
- **Benchmarks and budgets.**
  - TTT-Discover tasks: TriMul, MLA-Decode, three single-cell denoising
    sets, AHC039/058, AC1/AC2. GPT-6 Sol at $10/task or Opus 5 at
    $100/task, compared against CORAL rerun under the same cap and against
    TTT-Discover's *published* gpt-oss-120b + RL numbers.
  - Five AtCoder heuristic contests: all four baselines rerun with Opus 5
    at equal spend ($120–200).
  - Eleven systems/math tasks: Opus 4.6, at most 100 submitted candidates.
    Baselines are the *published* numbers from Qu et al. (CORAL), except
    signal processing, which was rerun.
  - Kernel builder: Opus 5 at low reasoning effort, all methods at common
    spend checkpoints from $25 to $118.
- **Spend accounting.** Spend is estimated from transcript token counts
  at list prices, because the runs used subscriptions. Meta calls are
  included.

## Results

- **Kernel builder (Table 4, matched spend, one run each).** 1,016 cycles
  at $25, below every baseline's $118 result (best EvoX 1,098). At $100 it
  reaches 960 vs 1,098, a 12.6% reduction. Fig. 1: AgentDiscover submitted
  only 15 candidates (456 calls, $0.26/call), against 128–166 for the
  population methods.
- **Multi-domain (Table 1).** Best on all 9 entries vs CORAL at matched
  spend. Margins are small on several:
  - AC1: 1.50274 vs 1.50282.
  - AHC058: 848.72M vs 848.39M.
  - Pancreas: 0.1603 vs 0.1676.
  - TriMul: 2,080 vs 2,419 μs.

  The AC1 bound ranked first on the Einstein Arena leaderboard on
  2026-10-03.
- **AtCoder (Table 2).** "Would have placed first" in all five original
  contests. CORAL also places first in three (AHC006, AHC012, AHC041), and
  OpenEvolve in one. CORAL has the higher score on AHC006 (2,317,352 vs
  2,309,017). With post-contest entries AgentDiscover ranks 1st–5th. On the
  AHC025 absolute score, AgentDiscover reaches the contest winner at $22
  vs CORAL's $73 (Fig. 8).
- **Systems/math (Table 3).** Best on 7 of 11 and tied on 4 at reporting
  precision. Several "wins" are in the fourth significant figure:
  - Cloudcast: 618.0 vs 618.4.
  - Erdős: 0.38077 vs 0.38089.
  - 3rd-autocorrelation: 1.4556 vs 1.4557.

  The largest gaps are EPLB (0.153 vs 0.149), LLM-SQL (0.737 vs 0.731)
  and transaction scheduling (4,651 vs 4,566).
- **Context resets (Fig. 4c–d, observational).** Probability that the best
  score improves within 3 candidates after a stall: 21.2% when continuing
  (n = 650), 25.0% after a reset (n = 32), 26.2% after reset + meta pass
  (n = 42). Expected gain: 0.38% / 0.86% / 1.87%.
- **Meta ablation (Fig. 5c–d; transaction scheduling, Haiku 4.5 search
  agent, 5 runs × 3 fork points = 15 forks).**
  - Beats the pre-fork best within 2 candidates: 73% vs 33%. Within 10:
    87% vs 53%.
  - Median candidate fitness: 0.801 → 0.832.
  - Final best: higher in 10 of 15 forks, equal in 2, lower in 3.
- **Parallel agents (Fig. 5a–b, 5 runs).** Faster in wall-clock time,
  with no difference at matched spend.
- **Evaluator hacks (Appendix C).**
  - Signal processing: two output samples scored 0.9259 against an honest
    0.727, because output length was never enforced. AgentDiscover's own
    "original" Table 3 score is exactly 0.9259.
  - LLM-SQL: discarding the input and returning a constant table scored
    0.994 vs 0.737.
  - Candidate code running inside the scoring process rebound `pearsonr`
    or printed `{"score": 1e9}`.
  - EPLB: the agent read the hidden next workload from a readable path,
    raising the score 0.25 → 0.44 past a keyword blocklist. It also
    printed inputs that were echoed back as feedback, and later candidates
    hard-coded them (0.153 → 0.201).

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest framing.** It
  becomes the 47th of 49 at-or-below. The digest says "better scores at
  lower cost than prior discovery frameworks on kernel engineering,
  biology, algorithm design and mathematics". That is the abstract's
  wording, and the body supports less of it:
  1. **Every main-table entry is a single run with no variance** (the
     authors concede that "small margins … should be read cautiously").
     Most Table 1/3 wins are smaller than any plausible run-to-run spread.
  2. **"Lower cost" is measured only where spend is matched**: the kernel
     builder, AtCoder and three TTT-Discover curves. The systems/math
     comparison caps submitted candidates (≤100) against published
     baselines with no spend figure. There AgentDiscover can run unlimited
     local experiments between submissions, so the evaluation budget is
     not matched.
  3. **Table 1's TTT-Discover comparison is model-confounded.** It is GPT-6
     Sol or Opus 5 against the published gpt-oss-120b + RL. Only the CORAL
     column is like-for-like.
  4. **"Outperforms prior frameworks" has exceptions.** CORAL wins AHC006,
     and four systems/math tasks are ties.
- **Most-favourable-cut check.** "Seven AtCoder contests, first place" is
  true, but it does not distinguish the methods. Every framework at this
  spend places near the top of these old contests, and CORAL also takes
  first in three of five. Fig. 1's "spend to reach 125×" ($10.87 vs
  $47.45) uses a threshold set just under where the baselines plateau
  (126–135×), which maximises the ratio. The matched-budget cycle counts
  in Table 4 are the honest version, and they still favour AgentDiscover
  clearly. "Matches or exceeds every baseline on eleven tasks" includes
  four ties and three fourth-digit margins.
- **The headline mechanism is not ablated.** The Bitter-Lesson claim is
  that *the agent owning the search* beats a fixed rule. No arm holds
  everything else fixed and forces a fixed selection rule on the same
  coding agent. Several confounds travel with it:
  - local experimentation before submitting (15 submitted candidates vs
    ~150; the same per-candidate-depth effect
    [[literature/papers/ishibashi2026effective]] found);
  - the target in the briefing;
  - prediction-before-submit;
  - the steering message;
  - the meta agent.

  The authors admit they "have not analyzed which search strategies the
  agents actually use", that is, whether the agents rediscover MAP-Elites
  or UCT. The paper therefore shows that this whole bundle beats the
  baselines. It does not show that agent-owned search is the reason.
- **The digest's steering question cannot be answered from the paper.** No
  arm removes the steering message. It is code-generated rather than an
  LLM supervisor, so it acts as a structured briefing plus a reset nudge,
  not as external judgment. The cost note from candidate five does push
  toward resets, so part of the reset effect is the server's doing. The
  source of the briefing's "target to beat clearly" is never stated. If it
  is the best known result, the agent gets information the baselines may
  not.
- **The context-reset evidence is observational.** The agent chooses when
  to reset, and the 21.2% vs 25.0% contrast compares n = 650 with n = 32
  (binomial SE about 8 points on the reset arm). The text's "this holds on
  average across all our experiments" outruns the figure. The only
  randomised component test is the meta ablation, which uses Haiku 4.5 on
  one task. Its meta model is "Claude Opus 4.6" in §4.2 but "Sonnet 5" in
  Table 10.
- **The comparison to CORAL is about memory format and session policy,
  not autonomy.** CORAL ([[literature/papers/qu2026coral]]) already lets
  long-running agents own retrieval and search direction. The stated
  differences are periodic context resets, a graph DB instead of shared
  files, local tools and experiments, and per-submission steering instead
  of heartbeat restarts. None of them is ablated against CORAL on its own.
- **Evaluator corrections and baseline comparability.** On LLM-SQL, PRISM
  and 3rd-autocorrelation, the baselines are Qu et al.'s published scores
  under the *original* evaluators, while AgentDiscover ran under the
  corrected ones. "The corrections preserve the scoring of valid
  solutions" makes this fair only if those baselines were honest, and
  that is not checked. Signal processing was rerun, and there three of
  four baselines scored lower under the corrected evaluator (CORAL
  0.8229 → 0.7790). OpenEvolve scored higher (0.6420 → 0.7286), which
  shows these are fresh runs, not re-scores.
- **What transfers regardless of the headline.**
  - The claims-vs-facts edge split, in which an agent's declared lineage
    is stored next to what the platform logged it actually read.
  - Read-only queries with typed writes.
  - Withholding candidate stdout from feedback, because echoed output is a
    leakage channel.
  - Prediction-before-submit, which gives a calibration trace for free.

  All of this is infrastructure, and none of it depends on the
  Bitter-Lesson result.

## Trust signals

- **Credibility:** 3.
  - For: an established Texas A&M ECE group (Kalathil, Narayanan,
    Chamberland) with NSF/DEVCOM funding. Code is public
    (github.com/mhdfb/AgentDiscover), and the AtCoder submissions and the
    Einstein Arena AC1 entry are linked, so they can be checked
    externally. The evaluator-hack appendix is candid and reports the
    authors' own inflated score. Baselines were rerun at matched spend
    where spend could be measured.
  - Against: not peer reviewed. The main results are single runs with no
    variance. Systems/math baselines are published numbers under different
    evaluators. Spend is estimated at list price from subscription
    transcripts. The central mechanism is unablated, and the ablation
    model is reported inconsistently.
  - Not 4: the evidence design (single runs, unmatched budgets on 11
    tasks) is below what a major-lab preprint with code would usually
    supply. Not 2: code, externally verifiable submissions and the hack
    audit put it above an unknown-group preprint.

## Follow-up

- **Relevance:** 4. It is the most complete public design for an agent
  that owns its own evolutionary search, from a framework this graph
  already tracks against (CORAL, ShinkaEvolve, OpenEvolve, EvoX). Several
  of its pieces bear directly on existing concepts:
  - session reset with a cost trigger and a fresh agent →
    [[concepts/context-eviction-policy]];
  - per-candidate depth and the selection rule as a query →
    [[concepts/evolutionary-search-grain]];
  - evaluator hardening → [[concepts/programmable-evaluator-oracle]];
  - the feedback-channel leak → [[concepts/information-firewall]];
  - the claims-vs-facts provenance edges →
    [[concepts/typed-claim-partition]].

  Not 5, because the load-bearing Bitter-Lesson claim is unablated.
- **For `/iterate` step ordering (the digest's question).** The paper does
  not show that a prescriptive step order hurts. Its own loop is also
  prescribed: explore → idea → candidate → loop, with a mandatory idea
  record and a mandatory prediction. What it frees is the *selection
  rule*. The safe import is to give the agent its history as a queryable
  store and keep the step contract fixed.
- **Cheap imports to test downstream:**
  - Record a predicted metric before each run and log the miss.
  - Store agent-declared parents separately from logged reads.
  - Strip candidate stdout from any feedback the agent sees on a
    hidden-data task.
  - Offer a reset when context cost exceeds a fresh start.
- **Candidates** (cited, not in the graph):
  - Gupta et al. 2026, "Automated discovery has no universally superior
    harness" (arXiv 2607.18235). 30 harnesses budget-matched, and full
    OpenEvolve ranks last. This is the paper's motivating evidence and the
    natural partner to [[literature/papers/zou2026fmlbench]].
  - Lee et al. 2026, Meta-Harness (arXiv 2603.28052).
  - Liu et al. 2026a, EvoX (arXiv 2602.23413).
  - Yuksekgonul et al. 2026, TTT-Discover (arXiv 2601.16175).
  - Cemri et al. 2026, AdaEvolve (arXiv 2602.20133).
