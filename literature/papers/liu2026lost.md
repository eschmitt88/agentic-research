---
kind: paper
title: "Lost in Implicit Routing: How LLM Agents Fail to Load Relevant Skills at Scale"
authors: ["Jinlin Liu", "Wei Chen", "Xiaojin Zhang"]
institutions: ["The University of Hong Kong", "Huazhong University of Science and Technology"]
year: 2026
venue: "arXiv 2610.18522 (cs.AI), v1 2026-10-13, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.18522"
code_url: "https://github.com/jinlinl/lost-in-implicit-routing"
citations: null
source: "raw/papers/liu2026lost.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
tags: ["skill-routing", "skill-loading", "progressive-disclosure", "distractor-scaling", "zero-load-rate", "loaded-skill-recall", "preselection", "hybrid-retrieval", "semantic-interference", "instruction-interference", "description-masking", "skillsbench", "process-metrics", "skill-scoping"]
---

# Lost in Implicit Routing: How LLM Agents Fail to Load Relevant Skills at Scale

## TL;DR

The authors hold SkillsBench tasks, workspaces, verifiers and each task's
golden skills fixed. They add s ∈ {0, 10, 100, 1,000, 10,000} irrelevant
skills sampled (seed 42) from a pool of 20,000+ skills scraped from a skill
market. Under *implicit routing* (Anthropic-style progressive disclosure:
the agent sees every skill's name, description and location, and decides
during execution whether to open one), agents load fewer skills, or none:

- Loaded-skill recall (|G ∩ L|/|G|) falls 73.6% → 27.4% from 0 to 1,000
  distractors, aggregated over 82 tasks × 3 open-weight models × 3 runs.
- The zero-load rate rises 9.3% → 51.6%.
- Substitution does not explain it. Non-golden loads occur in 2.32% of
  evaluations; near-duplicates of a missed golden skill (cosine > 0.8,
  e.g. `pdf-reader` for `pdf`) in 0.95%.
- Pass@3 falls less, 51.6% → 38.6%. On a 37-task subset Claude Opus 4.6
  keeps Pass@3 at 64.9 → 62.2 while its loaded recall falls 81.3 → 30.9.

Two diagnostics locate the cause, both on a 10-task subset at 1,000
distractors:

1. **Masking distractor metadata restores loading.** Replacing every
   distractor's name, description and location with `<mask>` tokens of
   about the original length raises loaded recall 18.6 → 63.9%. The
   no-distractor reference on that subset is 73.5%. Masking descriptions
   alone gives 63.3%; names alone 43.7%. So long context is not the
   cause; plausible-sounding descriptions are.
2. **Selection works when it is the only job.** Given the same full list
   and asked only to pick skills, the same models pick 73.9% of golden
   skills at 1,000 distractors (82.5 / 80.7 / 58.4 per model). A
   "plan skills first" instruction in the execution prompt recovers only
   44.6% loaded recall, and 39.9% planned recall vs 70.6% for isolated
   selection.

As mitigation, preselecting ≤5 skills before execution flattens the
curves. LLM preselection is best around 100 distractors; hybrid
(LLM-written queries → embedding retrieval → LLM final pick) is best at
1,000–10,000. Hybrid keeps loaded recall at 67.1% (1k) and 55.2% (10k),
and the zero-load rate at 9–11%.

## Claims

- **"Skill routing collapse."** Growing libraries make agents load fewer
  or no skills "rather than substitute other skills for golden ones."
- **Task success hides it.** "Strong executors can solve around missing
  skills, so final success alone hides a failing skill protocol."
  Benchmarks "should report loaded-skill recall and zero-load rate
  alongside task success."
- **Two interference mechanisms.** *Semantic interference*: distractor
  descriptions look like plausible alternative actions. *Instruction
  interference*: during execution, descriptions act as "conditional
  sub-instructions" that compete with the task. The authors flag the
  two-roles account as "an interpretation of behavioral evidence, not a
  measurement of internal model processing."
- **Scale-dependent recommendation.** Implicit routing for small curated
  libraries (with loading tracked). Explicit LLM preselection "around the
  hundred-skill scale". Hybrid preselection for thousands. Prompt
  reminders "complement, not replace, preselection."
- **Fixed preselection has costs.** It can omit skills needed later and
  adds 8–41 s median latency before execution.

## Methods

- **Tasks.** 82 of 87 SkillsBench tasks (5 dropped for infrastructure
  problems). 26 tasks have one golden skill, 36 have 2–3, 20 have 4+.
  A run passes only if the task's deterministic verifier passes; no LLM
  judge anywhere.
- **Harness.** A custom terminal agent that answers in JSON
  (`analysis`/`plan`/`commands`). The system prompt lists skills under
  `available_skills` and says "you MUST check if any skill matches the task
  BEFORE executing commands." A skill counts as loaded if the agent emits
  `load_skill` or reads its `SKILL.md`. This is not Claude Code or
  OpenHands.
- **Models.** Main study: GLM-5, Kimi K2.5, MiniMax M2.5, each task ×
  condition × 3 runs, >15,000 trajectories and 10B tokens. Validation:
  Claude Opus 4.6, Gemini 3.1 Pro, GPT-5.4 on 37 tasks, implicit routing
  only.
- **Preselectors** expose at most 5 skills to the executor, which must
  still open them. LLM (full metadata list to one call); RAG (Qwen
  text-embedding-v4, top 5); hybrid (LLM writes queries, quota-merged
  top-100 pool, LLM picks). Full-list LLM preselection is omitted at 10k
  as infeasible.
- **Metrics.** Pass@3 via the unbiased estimator with n = 3 runs, which
  equals "any of 3 passes". Loaded-skill recall, zero-load rate, average
  skills loaded, precision, and selected-skill recall |G ∩ P|/|G|.
- **Diagnostics.** 10-task subset at s = 1,000. Full masking and
  plan-first at 3 runs; field-level masking at 1 run per task and model.
- **Cost.** One implicit run averages 15.3 min and 485k input tokens at
  s = 0, and 17.0 min and 1.95M input tokens at s = 1,000.

## Results

- **Implicit routing by model (Tables 14–16).**

  | Model | Loaded recall 0 → 10 → 100 → 1k | Zero-load 0 → 1k | Pass@3 0 → 1k |
  |---|---|---|---|
  | GLM-5 | 79.6 → 71.8 → 55.0 → 20.9 | 7.8 → 64.9 | 57.3 → 42.7 |
  | Kimi K2.5 | 78.8 → 74.4 → 63.2 → 40.5 | 7.0 → 31.3 | 52.4 → 43.9 |
  | MiniMax M2.5 | 62.3 → 58.2 → 41.6 → 21.5 | 13.1 → 58.4 | 45.1 → 29.3 |

  With no skills at all, Pass@3 is 26.8 / 34.1 / 26.8. So even at 1,000
  distractors every model still beats the no-skill baseline. That differs
  from kim2026why, where flat loading tied empty.
- **The cliff is between 10 and 100.** Aggregate loaded recall is 73.6 →
  68.2 (s = 10) → 53.3 (100) → 27.4 (1k). For Opus 4.6 (37 tasks) it is
  81.3 → 77.7 → 43.3 → 30.9; Gemini 77.4 → 70.4 → 57.9 → 43.0; GPT-5.4
  76.2 → 62.9 → 51.8 → 49.3.
- **Strategy comparison (Tables 17–19), aggregate.**

  | Strategy | Loaded recall 0 / 100 / 1k / 10k | Zero-load 1k / 10k | Pass@3 0 / 1k / 10k |
  |---|---|---|---|
  | Naive (implicit) | 73.6 / 53.3 / 27.4 / – | 51.6 / – | 51.6 / 38.6 / – |
  | LLM preselect | 74.0 / 73.0 / 62.0 / – | 9.8 / – | 54.9 / 51.6 / – |
  | RAG | 74.6 / 69.3 / 58.6 / 48.5 | 13.2 / 18.0 | 52.4 / 46.5 / 41.5 |
  | Hybrid | 74.0 / 70.5 / 67.1 / 55.2 | 9.1 / 10.7 | 52.8 / 52.8 / 50.4 |

- **Tokens (Table 5).** At s = 1,000, implicit routing uses 1,966k total
  tokens per run. LLM preselection uses 547k (36k of it on routing), and
  hybrid 554k. Narrowing the list cuts cost by about 3.5×.
- **Coverage cost (Table 30).** The average number of golden skills shown
  to the executor falls from 2.52 (all visible) to 1.78–1.97 at s = 1,000.
  RAG exposes more golden skills than LLM preselection but scores lower
  Pass@3 (46.5 vs 51.6).
- **Precision holds while recall falls.** When agents do load at s = 1,000,
  88.6 / 87.2 / 76.4% of loads are golden (Table 23). Opus 4.6 is 95.2%.
- **Per demand bin (Table 26).** At s = 1,000, naive loaded recall is
  38.0% on single-skill tasks and 14.8% on 4+-skill tasks, from 91.9% and
  58.8% at s = 0.

## Critique / open questions

- **Against the digest framing: AT, with one mild over-read.** Every number
  in the digest entry is in the paper and matches the abstract. The
  over-read is "explicit selection from the same library keeps 73.9%". That
  number is *selected*-skill recall in a routing-only call. It is not
  *loaded* recall during execution. The comparable preselect-plus-execute
  numbers at 1,000 distractors are 62.0% (LLM) and 67.1% (hybrid). The
  abstract makes the same cross-metric comparison, so the digest copied it
  rather than inflating it. The digest also leaves out two things that
  matter for this box: the frontier-model result (Opus Pass@3 barely
  moves), and that the steep part of the curve is 10–100 skills.
- **Task-level n and no uncertainty.** There are no CIs anywhere. The
  headline routing effects (73.6 → 27.4 recall, 9.3 → 51.6 zero-load) are
  large against 82 tasks × 9 model-runs. The Pass@3 comparisons between
  strategies are 1–5 points (e.g. hybrid 52.8 vs LLM 51.6 at 1k) and are
  within the noise this graph tracks (see the accepted no-improvement
  noise band, docs/system-proposals/2026-09-27-iterate-no-improvement-noise-band.md).
  Read the strategy ranking from the recall and zero-load columns, not
  from Pass@3. Pass@3 is any-of-3, which is lenient and compresses the
  differences further.
- **The diagnostics are small.** Masking and plan-first run on 10 tasks.
  Field-level masking uses 1 run per task and model (30 runs per cell). The
  authors say proximity between conditions "does not establish
  equivalence". The masking result (18.6 → 63.9) is large enough to
  survive that. The descriptions-vs-names split (63.3 vs 43.7) is less
  secure.
- **The harness amplifies the effect, or at least shapes it.** All skill
  metadata sits in the system prompt of a custom JSON terminal agent with
  a "you MUST check" instruction. Claude Code exposes skills through a
  `Skill` tool listing, and other harnesses differ again. The direction
  likely transfers; the size may not. The proprietary validation uses the
  same harness.
- **Distractors are sampled at random from a market.** A real library is
  curated and topically clustered. Random distractors are mostly
  unrelated. Topically similar distractors would probably interfere more,
  which the masking result predicts but nobody tests.
- **"Recognizable when selected" mixes two settings.** The routing-only
  prompt asks for at most 5 skills and allows an empty list. Execution has
  no cap, but picking is one decision among many. The gap between 82.5%
  selected and 20.9% loaded for GLM-5 is real. Part of it may still be
  that a call whose only output is a skill list nudges the model to name
  something. Selected precision at 1k (74.1% for LLM preselection,
  Table 25) shows it does name wrong ones too.
- **Code is named but not available.** The GitHub URL in the paper
  returned 404 on 2026-10-20. Trajectories and the 20,000-skill pool are
  not otherwise released.
- **What it does not test.** It has no hierarchical/tiered loading arm
  (kim2026why, piriyakulkij2026subagents). Preselection is fixed before
  execution, with no re-routing during the run; the authors name that as
  next step. It does not test a skill's own description quality, only
  distractors'. And it has no measure of harm from loading a wrong skill.
- **Bearing on the hub's live fronts.**
  - *Skill scoping:* this is the strongest evidence so far that scoping is
    needed. It also gives a mechanism: under-loading from description
    interference, not dilution of a context budget. See the concept edits.
  - *HCE evaluation:* an outcome metric hid a broken component. Opus 4.6
    lost about 50 points of skill loading for 2.7 points of Pass@3. This
    is another case for logging process metrics (which skills fired)
    alongside the verifier result. It matches the exposure rule in
    [[concepts/skill-library-lifecycle]] (giampieri2026cortex: never-fired
    means unmeasured). A skill A/B on this box that reports only task
    success could read "no effect" when the skill never loaded.
  - *Redirect-before-halt proposal / shared-substrate error:* no direct
    bearing. All models face identical distractors, and they collapse
    together. But this is a shared input, not a shared judge, so it is not
    the correlated-error case.

## Trust signals

- **Credibility:** 3.
  - For: a clean controlled design. One variable (number of distractors)
    changes; tasks, verifiers, golden skills and the distractor sample are
    fixed across models. Scoring is by deterministic verifier, with no LLM
    judge. The scale is large (>15,000 trajectories, 10B tokens), with 3
    runs per cell. Three diagnostic arms target the mechanism directly
    (length-preserving masks, plan-first prompt, routing-only selection).
    The cross-check on three proprietary models agrees in direction. All
    plotted values are tabulated in the appendix, with exclusions,
    prompts and pseudocode. The limitations section is candid.
  - Against: it is not peer reviewed (v1, a week old). The groups (HKU,
    HUST) are reputable but not frontier-agent labs. There are no CIs or
    significance tests, and the mechanism diagnostics run on 10 tasks with
    1 run per field-level cell. The proprietary validation covers 37 tasks
    and implicit routing only. The code repo named in the paper returned
    404 on 2026-10-20. The single custom harness limits transfer.
  - Not 4 because nothing can be rerun yet and there is no uncertainty
    reporting. Not 2 because the design is controlled, the scoring is
    verifier-based, and the appendix is unusually complete.

## Follow-up

- **Relevance:** 4. This is material new evidence for the loading-function
  thread in [[concepts/skill-library-lifecycle]]. It is the fourth source
  after kim2026why, piriyakulkij2026subagents and cheng2026scope. It
  supplies the arm those sources lacked: retrieval at matched exposed size
  (≤5 skills) against the full list. It also separates description
  semantics from context length. It bears directly on
  [[concepts/shared-skill-namespace]], since a shared global namespace is
  exactly the growing-list condition. Not 5 because it does not seed a new
  concept, and the harness and distractor sampling limit how far its
  numbers transfer to this box.
- **Related in this graph.** [[literature/papers/kim2026why]] (flat vs
  tiered loading on MLE-Bench). [[literature/papers/piriyakulkij2026subagents]]
  (SkillsBench distractor sweep and lazy hierarchical loading).
  [[literature/papers/cho2026skillret]] (skill retrieval benchmark on
  17,810 Claude skills; a trained retriever is the obvious upgrade to the
  RAG arm here). [[literature/papers/cheng2026scope]] (scope as
  interference, not dilution). [[literature/papers/li2026who]] (same
  SkillsBench substrate; claim vs pass gap).
- **Check on this box.** The session skill listing here has ~40 entries
  (13 user skills plus built-in and plugin skills). That sits in the
  10–100 range where the paper's curves steepen. A cheap probe: log which
  skills each session actually invokes (skill-fire telemetry) and compare
  against the tasks where a skill should have fired. This is the
  loaded-recall metric in production.
- **Watch for** the code release, a re-routing (mid-execution) variant,
  and any replication inside Claude Code's actual `Skill` tool interface.
