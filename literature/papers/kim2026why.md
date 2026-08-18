---
kind: paper
title: "Why Solve It Twice? Hierarchical Accumulation of Skills for Transfer-Efficient ML Engineering"
authors: ["Yongbin Kim", "Yashar Talebirad", "Osmar R. Zaïane"]
institutions: ["University of Alberta", "Alberta Machine Intelligence Institute (Amii)"]
year: 2026
venue: "arXiv 2606.30911v2, cs.AI (preprint, v2 1 Jul 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2606.30911
code_url:
citations:
source: "raw/papers/kim2026why.pdf"
added: "2026-08-18"
relevance: 5
credibility: 3
status: read
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/shared-skill-namespace]]"
related_experiments: []
tags: [ml-engineering-agents, mle-bench, skill-library, hierarchy, scoped-loading, transfer, context-dilution, knowledge-organization]
---

# Why Solve It Twice? Hierarchical Accumulation of Skills for Transfer-Efficient ML Engineering (HASTE)

## TL;DR

The cleanest evidence in this graph that **how accumulated knowledge is
scoped matters more than how much of it there is**. Holding a 159-skill
inventory, model, pipeline and budget *identical* across 8 MLE-Bench
competitions and varying only which skills enter context: tiered
scope-matched loading medals **8/8**, flat "dump everything into the prompt"
loading medals **5/8** — exactly the same as loading **no skills at all** —
while burning **2× the output tokens**. Flat skill loading is not a weaker
version of scoped loading; it is worse than nothing per token spent. The
surrounding system (HASTE) organizes skills into global / domain /
competition tiers with LLM-driven promotion between them, and reaches 77.3%
medal rate on MLE-Bench Lite with a non-frontier model — though that
headline number has a retry confound the ablation does not.

## Claims

- **Flat memory is not merely suboptimal, it is inert.** "Flat skill loading
  leaves the medal rate unchanged relative to starting from scratch." Both
  flat and empty medal on 5 of 8. The paper's framing: the flat condition
  "tests whether more knowledge is enough," and the answer is no.
- **Three diagnosed mechanisms for why flat fails**, read off the logs
  rather than asserted:
  1. **Signal dilution** — relevant skills buried among domain- and
     competition-specific skills for *unrelated* competitions.
  2. **Context budget displacement** — at ~145K characters, the flat skill
     dump "crowds out the agent's own reasoning and code analysis."
  3. **Overconfident model selection** — flat priors pushed aggressive
     recommendations: on the jigsaw rerun the flat agent repeatedly
     attempted DeBERTa-v3-large and hit OOM, while the *empty* agent used
     simpler models and scored higher (0.985 vs 0.981).
- **Scope is the organizing axis, and it maps onto the agent hierarchy.**
  Three tiers, each loaded by a matching agent level: **global** (5 entries,
  loaded by every specialist), **domain** (19 tabular / 12 NLP / 15 vision,
  loaded only by the matching specialist), **competition** (108 entries
  across 21 directories, loaded only on re-runs of that competition). Per
  competition this is 10–60 entries in context instead of 159.
- **The store is plain-text markdown with YAML frontmatter on the
  filesystem**, and promotion "used LLM abstraction to keep the entries
  readable as plain text."
- **Promotion is a five-way classification, and conflicts are kept rather
  than resolved.** After each round the orchestrator judges each new
  learning as `skip` (already covered), `competition` (too specific),
  `domain` (abstract and promote one tier), `global` (universally useful),
  or `conflict` (contradicts an existing skill). **"When two learnings
  conflict, both are kept and annotated with conditions."** The worked
  example: ensembling helps when correlation is below 0.95 but hurts when a
  weaker member pulls down a stronger one.
- **Better organization partly substitutes for model strength and compute.**
  The stated conclusion, supported by reaching a competitive medal rate with
  Claude Sonnet 4.6 rather than a frontier model.
- **Accumulated priors may collapse the branching factor enough that simple
  search suffices.** HASTE uses linear refinement where comparable systems
  use trees, evolutionary populations, or MCTS. The authors are careful:
  this is offered as plausible, with "a controlled comparison at fixed
  knowledge condition" named as future work.
- **The hit rate on the agent's own edits rises with inventory** — the
  fraction of attempted changes that improve validation score and are kept
  rather than reverted goes from **42%** at 0–15 skills to **85%** at 50+.

## Methods

HASTE = orchestrator + three domain specialists (tabular, NLP, vision). The
orchestrator does domain classification, round scheduling (seed one
competition per domain cold, then assign the rest), and skill promotion
between rounds. Each specialist runs a five-stage pipeline: task profiling
(metadata, CV strategy, resource probe, no model execution) → prototype
screen (three fundamentally diverse approaches on one validation fold;
justified by up to a 2.7× score spread between best and worst) → adaptive
refinement on the winner (N=20) *and* runner-up (N=6), with auto-escalation
after two consecutive non-improvements, stagnation exit, and
revert-on-regression → rank-average ensemble of top-3, accepted only if it
beats the best single member → produce 2–5 plain-text learnings each with a
proposed tier. A 6-mode failure taxonomy (`UNDERFITTING`, `OVERFITTING`,
`FEATURE GAP`, `NOISE CEILING`, `DISTRIBUTION MISMATCH`,
`DIMINISHING RETURNS`) is supplied as prompt content to guide diagnosis, and
tiered history compression keeps the prompt under 20 lines even at 50+
iterations. Main benchmark: MLE-Bench Lite (22 Kaggle competitions), Claude
Sonnet 4.6, 12h per competition, 20 iterations. Ablation: 8 competitions
spanning NLP/vision/tabular/audio, 11-iteration budget, 159-skill inventory
frozen at one point in the campaign.

## Results

- **Controlled ablation (the paper's real evidence).** Tiered **100%**
  (6 gold, 1 silver, 1 bronze); flat **62.5%** (4 gold, 1 silver, 3 none);
  empty **62.5%** (5 gold, 3 none). Output tokens: flat 3.78M, tiered 2.27M,
  empty 1.86M. **Tokens per medal: tiered 284K, empty 371K, flat 756K** —
  tiered is 2.7× more token-efficient than flat. Flat also ran the *most*
  experiments (75 vs tiered 60, empty 65) with a slightly higher execution
  success rate (87% vs 83%), so "the extra attempts appear poorly directed:
  more compute without medal-rate gains."
- **The gap concentrates on hard and niche tasks.** mlsp-2013-birds
  (audio): tiered 0.964 gold vs flat 0.860 vs empty 0.832 (neither medals).
  random-acts-of-pizza: tiered 0.798 silver vs flat 0.599 vs empty 0.481.
  On the four easier competitions all three conditions medal with small
  differences.
- **MLE-Bench Lite:** 77.3% medal rate (17/22) with Sonnet 4.6 at 12h per
  competition, single seed.
- **Warm vs cold:** warm-start runs reach their best score in 7.8 refinement
  iterations on average vs 16.3 cold — a **52% reduction**. Skill store grew
  from 5 entries at cold start to 72 by phase 12.

## Critique / open questions

- **The headline 40.9% → 77.3% lift confounds transfer with retry, and the
  paper does not foreground it.** Reading the detail: 9 competitions medaled
  cold and "were skipped in the rerun phase," while 8 of the 13 cold
  failures flipped to medal on the warm re-attempt. So 77.3% pools *one*
  attempt on the easy 9 with a *second* attempt on the hard 13. A plain
  retry with no skills at all would also improve on 40.9%, and no such
  control is reported. The transfer claim rests on the iteration-count and
  hit-rate deltas plus the §4.5 ablation — not on this number.
  (To the paper's credit, the warm evaluation loads *only* global and
  domain skills, not competition-specific ones, which does close the
  obvious same-dataset leakage path.)
- **Single seed throughout, N = 8 in the ablation.** The authors say so
  plainly and name multi-seed replication as the priority follow-up. A
  100% vs 62.5% split on 8 competitions is 3 competitions; the direction is
  consistent across medal rate, tokens, and mean score, but no interval
  survives this.
- **The promotion step is load-bearing and unevaluated.** "We do not yet
  measure how often it assigns the correct tier or detects genuine
  conflicts." Since promotion is what makes the hierarchy a hierarchy, the
  central mechanism is unmeasured — the ablation tests *loading* given a
  hierarchy, not whether the hierarchy is built correctly.
- **Domain classification uses manual tags** for 100+ MLE-Bench
  competitions with a heuristic fallback, so the scoping that drives the
  result is partly hand-supplied rather than learned.
- **Everything else in the pipeline is unablated** — prototype screen,
  runner-up branch, rank-average ensemble, failure taxonomy,
  auto-escalation, revert-on-regression. Named as future work.
- No code or artifact release found in the paper.
- **The flat condition may be a straw man in one respect**: 145K characters
  dumped into context is the *worst* flat baseline, not a
  retrieval-over-flat-store baseline. The interesting comparison — tiered
  scoping vs semantic retrieval from a flat pool at matched context size —
  is not run, and it is the one most agent systems would actually implement.

## Trust signals

- **Credibility:** 3 — the core ablation is well designed (inventory,
  model, pipeline and budget all held constant; only the loading function
  varies), the failure mechanisms are diagnosed from logs with a concrete
  worked example rather than speculated, and the limitations section names
  single-seed evaluation, the unevaluated promotion step, and the unablated
  components without prompting. Held at 3: unreviewed preprint, single seed
  everywhere, N = 8 for the headline ablation, no released code, and a
  main-result framing that quietly pools a second attempt on the failures.

## Follow-up

- **Relevance: 5** — this is the measured case for something this project
  *already did architecturally and never validated*. The 2026-08-01 phase-5
  change scoped rule loading per-project (`@imports`, removing the global
  `~/.claude/rules` link) and made path-scoped `.claude/rules/` the pattern,
  on the reasoning that unscoped rules cost context where they don't apply.
  HASTE measures the same choice at fixed inventory: scope-matched loading
  medals 8/8, dumping everything medals 5/8 — **the same as loading
  nothing** — at 2× the tokens. Independent, quantified support for a
  decision made here on judgment.
- **The strongest single sentence for
  [[concepts/skill-library-lifecycle]]: a skill library with no scoping
  function is worth nothing.** The concept has covered acquisition,
  curation, and retirement; this adds that *the loading function is a
  first-class component*, and that inventory size without scope is a
  liability — signal dilution plus context displacement plus overconfident
  priors. The jigsaw example is the sharpest form: the skill-loaded agent
  scored *worse* than the skill-free one because a domain prior pushed it
  into an OOM.
- **Conflicting skills are kept and annotated with conditions rather than
  reconciled.** That is a real design decision this graph should note — the
  store records "ensembling helps when correlation < 0.95, hurts when a
  weak member drags a strong one" as a *conditioned pair*, not as a
  resolved rule. Compare [[concepts/verified-memory-writes]]: the
  provenance-and-conditions approach preserves the disagreement instead of
  letting a later write silently overwrite an earlier one.
- **A clean instance of [[concepts/hierarchical-delegation]] where the
  hierarchy is over *knowledge scope* rather than task decomposition**, and
  the two are made to coincide — each agent level is coupled to the skill
  tier that matches it, so an agent structurally cannot see out-of-scope
  knowledge. That is an information boundary enforced by architecture, in
  the same family as [[concepts/permission-gate-as-architecture]] but for
  context rather than capability.
- **Directly relevant to [[concepts/context-eviction-policy]] and
  cheng2026agenticsts's bounded contract.** HASTE's tiered history
  compression keeps the prompt under 20 lines at 50+ iterations, and the
  145K-character displacement finding is a measured instance of the
  crowding-out that eviction policies exist to prevent. Note the direction:
  the harm from over-filling context was large enough to erase the entire
  benefit of having the knowledge.
- **A caution for [[concepts/shared-skill-namespace]].** A shared namespace
  across projects is exactly the flat condition unless scoping is enforced
  at load time. This project's import contract (`@import` specific concept
  files by path) already implements the scoped version; the finding is that
  the discipline is not optional bookkeeping — it is where the benefit
  lives.
- **Watch item, and the honest one:** the comparison this paper does *not*
  run is tiered scoping vs **retrieval from a flat pool at matched context
  size**. Flat-dump-everything is the weakest possible baseline. Until that
  comparison exists, the finding supports "scope your loading" and does not
  establish that a hand-built tier hierarchy beats good retrieval — which
  is what most systems, including this one's `/lint` and import machinery,
  would want to know.
