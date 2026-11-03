---
kind: paper
title: "Coral Discovery: The Division of Cognitive Labor in Autonomous Research"
authors: ["Xinxing Ren", "Qianbo Zang", "Ziyan Wang", "Suman Deb", "Qiang Yang", "Caelum Forder", "Peter Carroll", "Gilead Gevero", "Jing Liu", "Yali Du", "Milind Tambe", "Zekun Guo"]
institutions: ["Coral AI Labs", "Università di Trento", "Harvard University", "University of Cambridge", "King's College London", "University of Hull"]
year: 2026
venue: "arXiv 2610.33281 (cs.MA), v1 2026-10-23, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.33281"
code_url: null    # "Code will be released shortly"; no repository named in the PDF
citations: null
source: "raw/papers/ren2026coral.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hce-evaluation]]"
tags: ["multi-agent-evolution", "division-of-labour", "kitcher", "direction-board", "direction-vs-method", "mechanism-label", "llm-verifier", "per-direction-credit", "ucb", "new-arm", "claim-cap", "herding", "leaderboard-following", "hhi-concentration", "mit-coral", "rerun-baseline", "best-of-3", "eval-count-budget", "no-token-accounting", "no-ablation", "kernel-take-home", "polyominoes", "opus-4.6", "redirect-before-halt"]
---

# Coral Discovery: The Division of Cognitive Labor in Autonomous Research

## TL;DR

Coral Discovery is a layer on top of MIT CORAL ([[literature/papers/qu2026coral]]).
It is by a different group (Coral AI Labs with Trento, Harvard, Cambridge,
KCL and Hull), despite the name. In MIT CORAL, four Claude Code agents share
a repository and a leaderboard. Coral Discovery takes the leaderboard away
and gives the agents a **direction board** instead:

- A **verifier** (a separate, tool-less, single-turn Opus 4.6 call at every
  eval) reads the diff and decides whether the candidate keeps its parent's
  **direction**, meaning the mechanism by which the program earns its score.
  If not, the candidate joins another direction or opens a new one. The
  agent writes the **method** (the change made within the direction) itself.
- **Credit** for a node is its gain over the best of its parent, any nodes it
  declares it borrowed from, and its own direction's best, divided by the
  global best. The board shows a per-direction yield (windowed mean credit),
  a UCB-style potential, a NEW arm for opening another direction, and a
  suggested head count.
- **Hard rules:** at most 2 agents per direction, 1 per direction in round 1,
  no duplicate mechanisms, and no copying another agent's method verbatim.
  An agent reads the board only when it judges its own line finished. The
  head-count suggestion is advisory.

All runs use Opus 4.6 (effort medium), with no internet and no grader
reading (Kernel excepted). There are nine MIT CORAL tasks and **3 runs per
configuration**, but **only the best of 3 is reported**, with no spread.
Against the better of two re-run baselines (1 agent; 4 agents with a
leaderboard), Coral Discovery wins 6, ties 1 and loses 2. Two of the six
wins are at the sixth decimal place, and the paper calls one of them "noise
level". Four wins have visible margins: Kernel 1095 vs 1138 cycles,
Polyominoes 91.78 vs 80.55, Signal 0.7904 vs 0.7718, and Erdős 0.380984 vs
0.381264. There is no ablation of any component. The budget is an eval cap
(100 evals, uncapped for Kernel), not tokens, and no token, dollar or
wall-clock figure appears anywhere.

The useful content for this graph is the design, not the scoreboard. It
offers a checkable definition of "a different direction", a credit rule
that scores a line against its own best, and a measured concentration
contrast (window HHI) between a leaderboard community and a board community.

## Claims

- A community that shares only a score "follow[s] the leader and
  converge[s] onto one approach". In the authors' baseline runs, three of
  four agents worked in one direction for most of the run, and whole thirds
  of a run fell into one direction.
- A leaderboard cannot answer when to dig deeper and when to move, because
  it has neither "approach" as a unit nor "rate of return" as a quantity.
- Per-direction credit (Kitcher's priority rule) keeps a lagging direction
  staffed. "A record inside a lagging direction is worth as much as a
  record inside the leading one."
- Under identical conditions, model and budget, it wins 6, ties 1 and loses
  2 against the better of the re-run single-agent and 4-agent MIT CORAL.
- Against all published numbers it is best or tied on 4 tasks. It improves
  MIT CORAL's reported Kernel result from 1103 to 1095 cycles, passing 1103
  at eval 275. It sets a Polyominoes state of the art of 91.78 against 87.0.
- "The leading scores rest on directions a score-only community never
  entered." The authors attribute this to the claim rules, the NEW arm,
  per-direction credit and the inspired-by threshold. This attribution is
  argued, not ablated.
- "Adding agents alone brings no improvement": the 4-agent baseline beats
  the single agent on only 3 of 9 tasks.
- Stated cost: spreading the budget lost LLM-SQL and Txn Sched.

## Methods

- **Setting.** MIT CORAL's autonomous multi-agent evolution. Four agents run
  asynchronously in their own workspaces. Native git is disabled, and all
  shared state goes through `coral` commands. Coral Discovery adds `board`,
  `direction`, `claim`, `insight` and `tree show`, and extends `eval` with
  `--direction` and `--inspired-by`. It withdraws `log` (leaderboard),
  `notes`, `revert` and `heartbeat` (App. A).
- **Direction definition.** A direction has three parts: the objective or
  proxy optimised, the solution object or decision unit, and the principle
  of search or construction. A method is a parameter change, variant,
  reordering, substitution or speed-up within it.
- **Verifier.** It reads the task, the direction table, the (direction,
  method) pairs of parent and candidate, the diff and the inspired-by nodes.
  It asks one question: is the scoring mechanism preserved? If unsure, the
  answer is "preserved". It also merges provisional duplicates. Calls run
  serially with a 5-minute timeout. Direction labels attach to nodes, not
  subtrees, so code lineage and direction are separate graphs.
- **Board (Eqs. 1–4).** Credit = max(0, s(X) − max(s(parent), max over
  inspired-by s(i), best of X's direction)) / |global best|. Yield is the
  windowed mean credit (W = 32) shrunk to the window mean with 4
  pseudo-evals. Potential = yield + κ·y⁺·√(ln N / (n_all + 1)), with
  κ = 0.05, where y⁺ is the run's highest yield (never decays). NEW is
  valued from each past opening's gain over the global best in its first
  k = 3 evals ("infancy"). Wanted head count is assigned by largest
  remainder over potentials. Failed evals get credit 0.
- **Round protocol.** The manager injects only the agent's own line
  (direction, method, last 5 scores and bests) and the other agents'
  current claims. If the agent has a line best or a concrete next step, it
  re-claims without reading the board. After 5 scored evals without a
  personal best, a "Diagnose" block asks whether the bottleneck is
  structural or removable. Otherwise the agent reads the board and claims.
  Each round has one eval and one written insight. The paper does not say
  whether agents can run the program locally between evals.
- **Tasks.** Nine of MIT CORAL's tasks in five groups: Kernel Eng.
  (Anthropic's VLIW take-home, cycles); Polyominoes and Circle-Pack.;
  Signal Proc. (held-out test signals) and Erdős min-overlap; EPLB and
  LLM-SQL; Txn Sched. and Cloudcast. All graders are deterministic.
- **Arms.** MIT CORAL re-run with 1 agent; MIT CORAL re-run with 4 agents
  and a leaderboard; 4 agents plus Coral Discovery. All use Opus 4.6
  (medium effort), WebSearch/WebFetch disabled, and the grader directory on
  the Read deny list (except Kernel). A post-hoc scan of all tool calls
  found no attempt at either. Cited columns (SOTA, OpenEvolve,
  ShinkaEvolve, EvoX, MIT CORAL) come from the MIT CORAL paper, whose runs
  allowed web access and grader reading.
- **Budget and reporting.** Each configuration ran 3 times. The paper
  reports the best of 3 ("the counterpart of MIT CORAL's Bo4"), its eval
  count and best@k. Only the first 100 evals count (20 for Cloudcast;
  Kernel uncapped). Realized counts differ: Kernel 600 / 600 / 596; Circle
  40 for all; Erdős 60 for all; EPLB 60 / 100 / 60; LLM-SQL 90 / 100 / 100;
  Cloudcast 18 / 20 / 18. Why a run ended at 40 or 60 is not stated.
  Verifier calls are extra model calls on the Discovery arm only, and they
  are not counted.
- **Attribution (§5.2).** Every submission in all three arms was
  re-labelled with the verifier's criterion (Opus 4.6 reading each diff),
  and directions were aligned per task. No human check or agreement
  statistic is given. The paper does not say which of the 3 runs per arm
  the attribution uses.

## Results

- **Main table (Table 2, best of 3).**

  | Task | 1 agent | 4 agents | Discovery | Discovery vs better baseline |
  |---|---|---|---|---|
  | Kernel ↓ | 1220 | 1138 | **1095** | 43 cycles better |
  | Polyominoes ↑ | 78.99 | 80.55 | **91.78** | +11.23 |
  | Signal ↑ | 0.7596 | 0.7718 | **0.7904** | +0.0186 |
  | Circle-Pack ↑ | 2.635977 | 2.635890 | **2.635983** | +0.000006 |
  | Erdős ↓ | 0.381264 | 0.382150 | **0.380984** | 0.00028 better |
  | EPLB ↑ | 0.145498 | 0.145346 | **0.145501** | +0.000003 |
  | LLM-SQL ↑ | **0.735769** | 0.733963 | 0.732704 | −0.003065 |
  | Txn Sched ↑ | **4608** | 4367 | 4566 | −42 |
  | Cloudcast ↓ | 618.0 | 618.0 | 618.0 | tie |

  The paper (§5.2d) says the EPLB arms "lie within 0.1453–0.1455, at noise
  level". That win still counts in the 6/1/2.
- **Against cited numbers.** Discovery is best or tied on Kernel,
  Polyominoes, Circle-Pack and Cloudcast. Against the cited MIT CORAL column
  it wins 4, ties 2 and loses 3 (Signal, Erdős, EPLB). The cited Signal
  0.8229 came with the grader readable, and that grader holds the test
  signals.
- **The re-run baseline is weaker than the cited one.** Kernel 1138 (4
  agents, 600 evals) against the cited 1103. Polyominoes 80.55 against the
  cited 84.2. Conditions (no web, no grader reading) and run selection
  both differ.
- **Speed.** On Kernel, Discovery reached the baseline's final 1138 at eval
  202; the baseline reached it at 594. On the other tasks it reached the
  baseline's final best between eval 3 (Erdős) and eval 75 (Signal), except
  LLM-SQL.
- **Directions (Table 3, Fig. 5).** Discovery opened 68 directions against
  the baseline's 41 on six tasks, and 34 vs 15 after the first third.
  Discovery-only directions per task: Kernel 25 (7 produced records), Poly
  8 (4), Signal 6 (4), Circle 4 (1), Erdős 6 (4), EPLB 6 (0), LLM-SQL 6 (2),
  Txn 3 (1), Cloudcast 1 (0). On 6 of the 7 tasks Discovery leads, the
  winning lineage contains a direction the baseline never ran (not EPLB).
  The share of the gain from records *in* Discovery-only directions is 100%
  on Kernel, Poly and Txn, 80% on Erdős and 0% on Signal and Circle. On the
  last two, the final record is in a shared direction but descends from
  Discovery-only ancestors.
- **Lagging lines survive (Fig. 6).** Local records (a direction's own best
  that is short of the global best): Discovery 9 / 4 / 15 on Signal /
  Circle / Erdős, against the baseline's 0 / 0 / 4. Window HHI (16 evals)
  over the last third, baseline vs Discovery: Signal 1.00 vs 0.66, Circle
  1.00 vs 0.36, Erdős 0.82 vs 0.29.
- **Champions open late.** The three champion directions on Signal, Circle
  and Erdős were opened at eval 62/100, 34/40 and 58/60. After the Signal
  champion's first global record, 76% of the remaining 38 evals went to it.
  The other two champions set their first records fewer than 8 evals
  before the end. The Polyominoes 91.78 came from a direction opened at
  eval 33.
- **Losses (Table 4).** LLM-SQL: Discovery opened 7 directions against the
  baseline's 2. 15% of its evals went to directions that never produced a
  record, against 1% for the baseline. Its champion (opened at eval 12, 72
  evals, 18 records) peaked at 0.732704, below the baseline's single
  locked-in direction. The single agent won with a direction opened at
  eval 45 that neither 4-agent arm found. Txn Sched: the single agent's
  simulated annealing, opened at eval 2, reached 4608 at eval 33. The
  authors conjecture that a single agent "carries the complete history"
  of a line, while multi-agent members "receive the code but not the
  process".

## Critique / open questions

- **Against the digest framing: BELOW (mild).** The 6/1/2 and the
  Kernel/Polyominoes numbers are the paper's own and quoted correctly.
  - The digest omits that the 6/1/2 is best of 3 runs against the better
    of two best-of-3 baselines, with no spread. Two of the six wins are at
    the sixth decimal (Circle +6e-6, EPLB +3e-6), and the paper calls the
    EPLB one noise in §5.2. That leaves four wins with visible margins,
    three effective ties and two losses.
  - "Seeds are unknown" is wrong in one direction and incomplete in the
    other. The run count is stated (3 per configuration). Only the maximum
    is shown, so variance is unknown.
  - "The board is advisory" is half right. The head count is advisory, but
    the claim cap (≤2 per direction, 1 in round 1) and the duplicate and
    verbatim checks are enforced. These binding rules are the most likely
    source of the spread, and nothing separates them from the board.
  - "Same model and budget" means the same eval cap and agent count. It
    does not mean tokens. Discovery adds a verifier call per eval, realized
    eval counts differ (EPLB 60 vs 100), and no cost is reported.
  - "Beats the strongest open multi-agent system" is against MIT CORAL
    re-run by these authors without web or grader access. Their re-run
    baseline is weaker than MIT CORAL's published numbers (Kernel 1138 vs
    1103, Polyominoes 80.55 vs 84.2).
- **No component ablation.** The intervention bundles at least seven
  changes: the verifier, the board statistics, the NEW arm, the credit
  rule, the claim caps, the "finish your own line first" protocol, and the
  removal of the leaderboard, notes and heartbeat. The prompt also changes
  (the Diagnose block replaces the heartbeat pivot). Section 6 assigns
  causes by narrative. A forced-distinct round 1 plus a 2-per-direction cap
  could explain much of Fig. 5 by themselves.
- **Direction labels are a model's reading of a diff, and the inputs
  differ by arm.** The 68 vs 41 count and the lineage shares in Table 3
  rest on Opus 4.6 re-labelling. The paper gives no human check, no
  agreement statistic, and no test of whether a second labelling pass
  gives the same partition. Discovery's submissions carry declared
  directions and inspired-by links into the labeller's context; the
  baseline's do not. "The baseline never entered" is relative to one
  baseline run (presumably its best) and to this labeller.
- **Max-of-max reporting.** Every number is the best eval of the best of
  three runs, scored on the same grader the agents optimised. The graders
  are deterministic, so evaluation noise is zero. Run-to-run noise is not,
  and it is unmeasured. Under the accepted noise band, the Circle, EPLB and
  arguably Erdős wins would not count. Only Signal Proc. has a held-out
  split, and the agents see its score.
- **Kernel context.** 1095 is not the best Kernel number in this graph.
  [[literature/papers/farahbakhsh2026agentdiscover]] reports 960 cycles at
  matched spend (Opus 5, low effort, one run). That paper also covers
  MIT CORAL's systems/math suite with Opus 4.6. Coral Discovery does not
  cite it. The models differ, so this is context, not a contradiction.
- **What holds.** Under one controlled setup, a 4-agent leaderboard
  community converged (last-third HHI 0.82–1.00) and the board community
  did not (0.29–0.66). The board community had larger leads on four tasks.
  The design is concrete enough to reimplement from §3 and App. A. The
  authors also report their two losses with an allocation breakdown.
- **For redirect-before-halt
  (docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md).**
  This is the candidate answer to "what should a forced pivot change", with
  three qualifications.
  1. *What changes.* A direction is the scoring mechanism: objective or
     proxy, solution object, or search/construction principle. This maps
     onto the proposal's objective / representation / backbone axes. The
     verifier is a cheap, separate check that a pivot really changed the
     mechanism. It reads the diff, not the agent's own label, and defaults
     to "preserved" when unsure. This is a stronger audit than the
     proposal's self-logged `redirect=<axis>`. Its accuracy is not
     reported.
  2. *Credit against the direction's own best is an allocation rule, not an
     acceptance rule.* The champion is still the global best. A new
     direction branched from the incumbent must still beat the incumbent
     to earn credit (the parent is in the threshold). The rule only helps
     lines that continue from their own, weaker nodes. So it does not
     support relaxing "judge the pivot against the chain's best". It
     supports *keeping a pivot line alive* while it sets its own records.
     That costs extra cycles, which the compute-neutral proposal does not
     have.
  3. *A new direction is valued over k = 3 evals, not 1.* The NEW arm's
     infancy window is 3 evals, and the champions came late from lines
     that lagged first. This agrees with the
     [[literature/papers/zhou2026athanor]] inference that one cycle
     favours the incumbent. It also shows the cost side: on LLM-SQL,
     per-direction credit kept agents in a line with no headroom.
  Nothing halts in this paper (fixed eval budget), so the redirect-vs-halt
  arm is still missing.
- **For the noise band.** The headline tally uses bare `>` and counts a
  +3e-6 EPLB win, while §5.2 calls the same gap noise. This is a live
  example of why a band belongs in the comparison rule, not only in prose.
- **For the known-bad preflight.** No do-nothing submission is scored. The
  seed program appears only as the start of the curves. The post-hoc
  tool-call scan for grader or internet access is a sandbox audit, not an
  evaluator check.

## Trust signals

- **Credibility:** 3.
  - For: authors from Harvard (Tambe), Cambridge, KCL, Trento and Hull.
    MIT CORAL is re-run under identical, stricter conditions
    rather than only quoted. Agent tool calls were scanned afterwards for
    grader or internet access. The method is specified to the equation and
    command level. Losses are reported with an allocation breakdown, and the
    authors flag their own EPLB win as noise.
  - Against: preprint, eleven days old, and code is only promised. The paper
    reports best of 3 with no spread, CIs or tests. There is no ablation of
    any component, and no token, dollar or wall-clock accounting. Direction
    labels come from an unvalidated LLM labeller with arm-dependent inputs.
    The paper does not say which run the attribution uses. "Coral AI Labs"
    shares a name with MIT CORAL but is a separate group, which may confuse
    citation.
  - Not 4 because nothing is released and the headline rests on max-of-3
    single numbers, two of them inside the sixth decimal. Not 2 because the
    baseline is a controlled re-run and the reporting of costs and losses
    is candid.

## Follow-up

- **Relevance:** 4. It is the first tested version in this graph of a
  herding remedy for a shared-memory agent community: a re-run
  leaderboard arm at equal eval caps, with a measured concentration
  contrast. It also gives the most concrete operational definition so far
  of "a different direction", with an independent check. That bears
  directly on [[concepts/evolutionary-expansion]],
  [[concepts/shared-substrate-contagion]] and the redirect-before-halt
  front in [[concepts/budget-as-ceiling]]. Not 5 because it has no
  ablation and reports best of 3 only, so it anchors design options rather
  than effect sizes.
- Related in the graph: [[literature/papers/qu2026coral]] (the base
  system; that note is only a skim and could be deepened against this
  paper's re-run numbers), [[literature/papers/zhang2026agora]] (the
  untested diversity-aware UCB this board resembles),
  [[literature/papers/zhou2026athanor]] (convergence from correlated
  priors; restart pivots lost in one round),
  [[literature/papers/he2026beneath]] (mechanism entropy over accepted
  edits; Coral's per-eval labels plus window HHI are the same
  instrument), [[literature/papers/farahbakhsh2026agentdiscover]] (same
  task suite, agent-owned selection), [[literature/papers/gurkan2026mutation]],
  [[literature/papers/shi2026weight]].
- **Wanted from the authors:** all three runs per arm with spread; an
  ablation that keeps the claim caps and removes the board statistics (and
  the reverse); token counts per arm including verifier calls; a human
  audit of a sample of direction labels.
- **Candidates** (cited, not yet in the graph):
  - Ren et al. 2026, AgentRadio: "Passive awareness for long-horizon
    multi-agent collaboration" (arXiv 2607.28430). Same group; gains
    concentrate on decomposable tasks.
  - Kitcher 1990, "The division of cognitive labor" (J. Phil.), and
    Weisberg & Muldoon 2009, "Epistemic landscapes and the division of
    cognitive labor" (Phil. Sci.). The theory the credit rule borrows.
  - Liu et al. 2026, EvoX: Meta-evolution for automated discovery (COLM
    2026).
