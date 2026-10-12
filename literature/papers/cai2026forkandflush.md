---
kind: paper
title: "Fork-and-Flush: Escaping Idea Basins in Autoresearch Agents"
authors: ["Ziyang Cai", "Christos Ziakas", "Vasilis Kontonis", "Tim Pearce", "Siddhartha Sen", "Akshay Krishnamurthy", "Shivam Garg", "Dimitris Papailiopoulos"]
institutions: ["Microsoft Research", "Princeton University", "Imperial College London"]
year: 2026
venue: "arXiv 2610.07447v1 (cs.LG, 2026-10-05); preprint, no venue stated"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.07447"
code_url: null
citations: null
source: "raw/papers/cai2026forkandflush.pdf"
added: "2026-10-12"
relevance: 5
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/evolutionary-expansion]]"
tags: ["autoresearch", "idea-basins", "fork-and-flush", "context-reset", "workspace-kept", "go-with-the-winners", "best-of-n", "equal-compute", "redirect-before-halt", "stall-trigger", "fixed-schedule", "compaction", "outcome-variance", "min-max-normalization", "triplet-embedding", "frontier-cs", "edgebench", "copilot-cli"]
---

# Fork-and-Flush: Escaping Idea Basins in Autoresearch Agents

## TL;DR

Eight independent runs of one autoresearch agent (GitHub Copilot CLI on
GPT-5.6-terra or GPT-5.5) on the same task plateau at very different
scores. An LLM-judged map of their artifacts shows each run staying in one
region of approach space, which the paper calls an "idea basin". The
intervention, **fork-and-flush**, runs on a fixed schedule:

- an opening round of K = 4 probes from an empty workspace;
- at 33% and 66% of the budget, K = 4 branches that each copy the leader's
  workspace but start with an **empty chat context**;
- each branch gets about 0.06 B; the best-scoring branch becomes the new
  leader and the rest are discarded.

Across 13 tasks it has the highest mean on 12. The headline is +66% over
single-run and +44% over best-of-4 "on the min–max normalized average
score", with budgets matched in model-token credits.

**The paper reads below the digest framing.** The 66% and 44% are ratios
of min–max normalized means, and that ratio depends on which runs set each
task's min and max. The same fork-and-flush runs score 0.78 in Table 1 and
0.71 in Table 3, where the pool differs. On raw task scores the median
per-task gain is **about 7% over single-run and 4% over best-of-4** (our
arithmetic from Table 1). Two tasks, AHC014 (+142%) and order-add (+101%),
carry most of the aggregate, and LWE-rec goes backwards (−6.9%). "Equal
compute" means equal model-token credits. Solver and GPU time and
wall-clock time are not matched.

On the digest's question: **both halves contribute, about equally on
average, and neither alone gets the full effect.** On 11 tasks the
normalized mean is 0.45 for single-run, 0.55 for flush-only, 0.57 for
fork-only and 0.71 for both. Fork-only beats single-run on 10 of 11 tasks.
Flush-only is erratic: it beats single-run on 6 of 11 and holds the two
largest single-cell gains. For redirect-before-halt, this is the best
matched-compute evidence so far that a **controller-level redirect beats
continuing**. It has no halt arm. Its stall-triggered variant mostly never
fired, and the prompt-level "pick a DIFFERENT approach" directive did not
change which branch won.

## Claims

- **Persistent outcome variance.** "independent runs of the same agent on
  the same task often plateau at substantially different scores, with gaps
  that persist even after considerable additional compute."
- **Idea basins.** Submissions "from individual runs tend to remain in
  localized regions" of an LLM-judged functional-similarity map. The
  localization "persists despite periodic compaction of the agents'
  conversational histories."
- **Compaction is not enough.** "One might expect this compaction to
  weaken dependence on the earlier reasoning path and reopen exploration.
  We find that it is insufficient."
- **Why flush.** "A branch that retained C would carry the parent's
  reasoning that tends to re-enter the same local basin. Starting from
  (W, ∅) restores search diversity while preserving the accumulated
  workspace." In the related-work framing, "the fork must re-derive its
  intent from the artifacts, which is what lets it land in a different
  basin."
- **Both mechanisms matter.** "while both individual components outperform
  the single-run baseline, neither mechanism alone accounts for the full
  effect. In addition, forking provides a slightly larger improvement than
  flushing."
- **Fixed schedule over plateau detection.** "early development runs using
  a stall-clock trigger rarely executed the intervention: across five
  tasks, 15 of 25 plateau-triggered runs exhausted their budget before the
  stall criteria were met." Micro-improvements "restart the stall clock",
  so runs "creeping upward within a narrow local basin" evade detection.
- **Variance lives at the capability frontier.** On MAX-3-SAT with three
  Qwen3.5 sizes, run-to-run spread is an inverted U in attainment, peaking
  at a = 0.497. The authors expect the exploitable variance "will not
  vanish as models scale."
- **Conclusion wording.** Fork-and-flush "consistently outperforms serial
  baselines". LWE-rec is a counterexample in their own Table 1.

## Methods

- **Agent and harness.** GitHub Copilot CLI 1.0.73 with the same tools,
  prompt and workspace in every condition. GPT-5.6-terra at high reasoning
  effort runs 11 tasks; GPT-5.5 at extra-high runs kernel and tinyCIFAR.
  - A run is a sequence of episodes. Within an episode the agent edits,
    runs and submits for a score as often as it likes.
  - An episode ends on a self-halt or a token cap. The harness then sends
    "Continue — this task is open-ended and continuously scored; there is
    no fixed 'done'. Keep working in /app. Do not stop."
  - Agents are never told the budget or how much of it is left.
- **State.** The state is s = (W, C): workspace files and chat history. A
  deterministic grader scores W. The reported result is the maximum score
  over **every lineage the method instantiated**, including discarded
  branches.
- **Compute.** Budget B is in Copilot credits, "a fixed function of
  input/output model tokens (100 credits ≈ US$1)". It ranges from 6k to
  100k credits per run, about $60–$1,000. Per Table 4, B is set per task
  "to where single runs plateau". On tinyCIFAR it is the length of the
  shortest single-run baseline.
- **Tasks (13).** Nine come from Frontier-CS: polyomino, job-shop,
  sphere-pack, relay-lay, three AtCoder heuristic ports (AHC014, p150,
  p171), LLM-SQL and LWE-rec. Two come from EdgeBench: Borden-inv, a
  groundwater inverse problem, and order-add. The other two are
  Anthropic's VLIW kernel take-home and a custom tinyCIFAR compression
  task. Median single-run wall time is 5–107 hours.
- **Baselines.**
  - Single-run spends all of B on one lineage. App. B says there are eight
    per task; Table 3 says seven on AHC014, p171 and LWE-rec and six on
    p150. Table 1's caption says "5 independent runs" for every method.
  - Best-of-4 runs four lineages at B/4 each and keeps the best.
  - Fork-and-flush has 5 seeds per task (4 on tinyCIFAR).
- **Budget split.** This is our arithmetic, assuming the triggers count
  total spend. The three branch rounds take about 3 × 4 × 0.06 B = 0.72 B,
  of which about 0.54 B goes to discarded branches. The leader's own
  serial legs get about 0.28 B, and the selected lineage's total depth is
  about 0.46 B. The method is mostly repeated short best-of-4 probes with
  go-with-the-winners, plus short trunk legs.
- **Ablation (11 tasks; kernel and tinyCIFAR excluded for cost).**
  - Flush-only is K = 1 with a flush at each forced round: one lineage, no
    selection, no discarded spend.
  - Fork-only is K = 4 branches that keep the trunk's chat context.
  - Normalization pools all four methods' runs, which is not the same pool
    as Table 1.
- **Branch directive.** In each round, two of the four branches get
  "deliberately pick a DIFFERENT approach … It is acceptable for the
  measured result to get worse before it gets better." Any branch that
  inherits a scored submission also gets a "protected lineage incumbent"
  notice. It quotes the inherited score and restores the artifact if a
  scaffold overwrote it.
- **Idea map.** Up to 51 evenly spaced, deduplicated submissions per run
  are used.
  - An unnamed LLM rewrites each one into an 11-section functional
    specification.
  - The same deployment answers about 6,375 "is A closer to B or C?"
    triplets per task.
  - A 2-D t-STE map is fitted, with SOE as a robustness refit.
  - Fit is checked on 20% held-out triplets: 0.85 mean agreement for
    t-STE and 0.83 for SOE. Chance is 0.50, and a run-membership-only
    heuristic gets 0.52–0.60.
- **Schedule development (App. D, Table 5).** Development runs on six of
  the same tasks compared a plateau trigger, one forced round at 50%, and
  two rounds at 33/66%. There was no systematic sweep of K or b_explore.

## Results

- **Table 1 (means; single-run n = 6–8, others n = 5):**

  | task | single | best-of-4 | F&F | F&F vs single (raw) |
  |---|---|---|---|---|
  | polyomino | 88.56 | 90.73 | 94.41 | +6.6% |
  | job-shop | 9.66 | 9.91 | 10.33 | +6.9% |
  | AHC014 | 23.71 | 20.66 | 57.46 (sd 29.35) | +142% |
  | p150 | 123.47 | 123.47 | 129.74 | +5.1% |
  | p171 | 84.26 | 88.88 | 92.79 | +10.1% |
  | sphere-pack | 43.87 | 43.76 | 44.94 | +2.4% |
  | relay-lay | 72.59 | 72.74 | 72.81 | +0.3% |
  | LWE-rec | 155.3 | 141.14 | 144.60 | **−6.9%** |
  | LLM-SQL | 26.90 | 26.93 | 27.04 | +0.5% |
  | order-add | 24.60 | 26.23 | 49.52 | +101% |
  | Borden-inv | 64.62 | 76.45 | 85.85 | +32.9% |
  | kernel | 120.36 | 122.50 | 135.23 | +12.4% |
  | tinyCIFAR | 0.429 | 0.560 | 0.572 | +33.3% |
  | **norm. avg** | 0.47 | 0.54 | 0.78 | |

  The raw-percent column is ours. The median gain is +6.9% over single-run
  and +4.2% over best-of-4. A rough Welch t from the table's SDs gives
  |t| > 2.2 on 9 of 13 tasks, including LWE-rec's loss at t ≈ −2.5. The
  paper reports no test or interval.
- **Variance.** The normalized SD is 0.16 for F&F, 0.26 for single-run and
  0.21 for best-of-4, and it falls relative to single-run on 10 of 13
  tasks. It rises sharply on AHC014 and order-add. In Fig. 7, AHC014's
  mean rests mostly on run 3 (about 105) and run 1 (about 60).
- **Ablation (Table 3, normalized means):**

  | method | 11 tasks | 5 original tasks |
  |---|---|---|
  | single-run | 0.45 | 0.37 |
  | flush only | 0.55 | 0.41 |
  | fork only | 0.57 | 0.47 |
  | fork-and-flush | 0.71 | 0.63 |

  - Fork-only beats single-run's mean on 10 of 11 tasks; the exception is
    LWE-rec.
  - Flush-only beats it on 6 of 11: job-shop, p150, relay-lay, LLM-SQL,
    order-add and Borden-inv.
  - Flush-only's p150 mean of **160.80** is above every single-run and
    every F&F run on that task (F&F max 135.37). Its order-add mean of
    40.73 is also well above fork-only's 30.09.
  - On 11 tasks the gains are roughly additive (+0.10 and +0.12 against
    +0.26). On the 5 original tasks they are superadditive (+0.04 and
    +0.10 against +0.26).
- **Directive null.** Across 178 mid-run rounds (this count includes
  development runs), a directed branch won 81 times, or 46%, against 50%
  expected (p = 0.26).
- **Basin jump (Fig. 3, one polyomino fan-out).** At a flush at 71% of the
  budget, the winning branch moved 43 median single-run steps in its first
  submission under t-STE, or 10 under SOE. The rejected branches moved at
  most about 1 step, and no single-run step after the first quarter of the
  budget exceeded 4.4. App. H shows nine trajectories qualitatively. Some
  winning branches never leave the fork's basin ("robust local
  refinement").
- **Schedule (Table 5, development runs).** Both fixed schedules beat
  single-run on all 6 tested tasks; two rounds won on 4 and one round on
  2. The plateau trigger "never fired" on AHC014 and p171 and did worse
  than single-run there.
- **MAX-3-SAT.** Fitting a quadratic to spread against attainment gives a
  peak at a = 0.497, with 8–32 seeds per (model, V) cell and bootstrap CIs
  on the SD.

## Critique / open questions

- **The headline uses a ratio of normalized scores.** Min–max
  normalization stretches each task to [0, 1] over the observed runs. A
  noise-scale gap therefore counts the same as a 2× gain: relay-lay's
  72.59 → 72.81 and LLM-SQL's 26.90 → 27.04 weigh as much as AHC014's
  23.7 → 57.5. The zero of each task's scale is the worst observed run, which is
  usually a baseline, so "66%" is a ratio between two positions on
  rescaled axes, not a relative gain in anything the tasks measure. The pool sensitivity is visible in the paper
  itself: the same F&F runs give 0.78 in Table 1 and 0.71 in Table 3.
  Median raw gains of 7% and 4% are the honest summary. On the two tasks
  where a different basin exists and is found, the gain is large.
- **"Equal compute" counts tokens only.** By design, credits exclude
  solver and training execution: "A credit meters model tokens, not
  wall-clock." That matters here because Table 4 says most of the
  hours are "the agent's own solvers and training jobs, which consume no
  credits", and single episodes run up to two days on LWE-rec and
  order-add. Four parallel branches run four sets of solver and GPU jobs at
  once. Neither GPU-hours (tinyCIFAR) nor wall-clock is reported per
  condition. On a single-GPU box, token-matched is not cost-matched.
- **Best-of-4 is a weak comparator by construction.** B is set "to where
  single runs plateau", so each best-of-4 draw gets a quarter of the
  plateau budget. Its max is below single-run's max on 7 of 13 tasks. A fairer
  selection baseline would be best-of-2 at B/2, or a sweep over N; the
  paper runs only N = 4.
- **The baseline accounting is inconsistent, and the gaps run against the
  baseline.**
  - Table 1's caption says 5 runs per method. App. B says eight single-run
    baselines. Table 3 says seven on AHC014, p171 and LWE-rec and six on
    p150.
  - Fig. 5 shows single runs reaching **59.8 on AHC014** and **137.1 on
    p150**, above Table 1's single-run maxima of 39.35 and 130.80. The
    dropped baselines appear to include the best ones, and no exclusion
    rule is stated. Adding the 59.8 run back moves AHC014's single-run
    mean from 23.7 to about 28.2 (our arithmetic). That does not flip the
    task, but it does shift the normalization.
- **The configuration is not as fixed as stated.** §5.1 says "Every
  fork-and-flush run in Table 1 adheres to a fixed forced cadence featuring
  three fan-outs". App. G says "On kernel and tinyCIFAR, some runs use a
  single forced round at 50% of the budget." The schedule, K and b_explore
  were chosen on development runs over the same tasks (Table 5), and Table
  5 has more 33/66 runs than Table 1 (11 vs 5 on p171). The paper does not
  say how Table 1's runs were chosen from them. There are no held-out
  tasks.
- **The ablation does not cleanly isolate the flush.** The flush-only arm
  may also carry the "pick a DIFFERENT approach" directive and the
  protected-incumbent notice; the paper does not say. Flush-only's p150
  mean (160.80, above any run of any other method) is an outlier that
  deserves its own explanation. There are 5 draws per cell and no
  intervals, so 0.55 vs 0.57 is not a ranking.
- **The basin mechanism is illustrated, not counted.** The paper shows one
  hand-picked fan-out in detail and nine trajectories qualitatively. It
  never reports how often the winning branch changes basin, or whether
  basin changes predict the gain. App. H concedes that some wins come from
  "local refinement" with no jump. The jump's size is embedding-dependent
  (43× under t-STE, 10× under SOE). The maps reproduce the LLM's own
  triplet judgments, and those judgments are never checked against humans
  or against behaviour on held-out instances.
- **The MAX-3-SAT inverted U is close to built in.** A bounded attainment
  score must have zero spread at 0 and at 1, so spread peaking near the
  middle is what a bounded variable does. The experiment also uses small
  Qwen models on a different problem, not the autoresearch agents. It
  supports "spread exists at the frontier", not "spread comes from idea
  basins".
- **Selection reads the reporting metric.** A branch wins on J, its max
  graded score, and R is the max over every lineage including discarded
  ones. With deterministic graders that is legitimate. tinyCIFAR's merit
  is "read on a sealed test split", though, and the agent submits for that
  score repeatedly. The paper does not say whether the selection score and
  the reported score come from different splits.
- **What the paper does not test.** There is no halt arm. Halting cannot
  beat continuing on a best-so-far score, so the halt question is a cost
  question, and the paper never prices it. There is no stall-triggered arm
  in the main results. There is no comparison with flush-without-workspace,
  i.e. a fresh start (best-of-N is the nearest). There is no K sweep.
- **No code.** The harness, tasks and logs are not released, though the
  prompts are given in full (App. I).

## Trust signals

- **Credibility:** 3.
  - For: Microsoft Research authorship, including Krishnamurthy, Sen and
    Papailiopoulos. The two first authors were MSR interns; their emails
    are Princeton and Imperial College London. The task suite is real and
    long-horizon (13 tasks, runs up to 107 h). A 2×2 ablation of the two
    mechanisms. A directive null with a test. Disclosure of the failed
    stall trigger. Full prompts. An embedding validated on held-out
    triplets and refit with a second method.
  - Against: an arXiv preprint with no venue; no code or logs; n = 5 per
    cell with no intervals or tests; a headline built on a ratio of
    min–max normalized means; inconsistent baseline counts, with excluded
    baselines that appear to be the strongest; a "fixed" schedule that is
    not fixed on two tasks; schedule tuning on the same tasks.
  - Not a 4: that would need released code, or tighter reporting on the
    baselines.

## Follow-up

- **Relevance:** 5. This is the strongest matched-compute evidence the
  graph has for the redirect half of redirect-before-halt. It is
  long-horizon, on real optimization tasks, with a 2×2 ablation separating
  a controller-level context reset from selection. It anchors the redirect
  material in [[concepts/budget-as-ceiling]], where every earlier source
  (chandran2026autoresearch, xu2026conflictguide, samanta2026learning,
  woo2026youra) was single-system, fixed-iteration or bundled. It is not
  canonical for the halt half, because there is no halt arm.
- **Verdict on redirect-before-halt: supportive of redirect over continue.
  Silent on halt. Cautionary on the trigger.**
  - A redirect implemented by the controller (flush the context, keep the
    workspace) improved on continuing at equal token spend. The prompt
    directive alone did not change which branch won. That joins the
    controller-not-prompt results of yan2026traceml, xu2026conflictguide and
    samanta2026learning.
  - **The trigger is the weak point.** A plateau-triggered fork-and-flush
    left 15 of 25 runs without a single firing, because micro-improvements
    reset the stall clock. A redirect hung off `max_consecutive_no_improvement`
    will rarely fire unless sub-noise improvements are prevented from
    resetting the counter, which is exactly what the accepted noise band
    should do. This links the two `/iterate` changes: the band is a
    precondition for a stall-triggered redirect. The alternative is a fixed
    schedule, which is what worked here.
  - The cheap transferable piece for `/iterate` is flush-only: K = 1, the
    workspace and keep-gate kept, the context restarted at fixed points.
    It is also the least reliable arm, at 6 of 11 tasks. Fork adds
    consistency but costs K× parallel compute, which is real GPU contention
    on this box.
- **Compaction is not a redirect.** Single runs compacted periodically and
  still stayed in their basins. A `/iterate` session that compacts its
  context has not thereby escaped anything. See
  [[concepts/context-eviction-policy]].
- **The workspace as the only carried state works for exploration too.**
  Flushed branches re-derive intent from W. That is the
  [[concepts/file-as-bus]] contract used as an exploration operator, and it
  echoes samanta2026learning's "model contexts are temporary".
- **pass@k reporting.** The paper shows the selector-vs-oracle issue at
  run grain. Selection uses the deterministic task grader, so capture
  should be high. The reported R is still a max over all lineages, which
  rewards the number of probes, not only the depth of the trunk. See
  [[concepts/pass-at-k]].
- **Questions for a replication or `/iterate` pilot:**
  - flush-only, triggered on a noise-band-aware stall counter, vs continue
    vs halt, with tokens and GPU-hours both reported;
  - the fraction of winning branches that change basin, and whether a
    change predicts the gain;
  - fresh start (empty W, empty C) vs flush (W kept, empty C) at equal
    spend, to price the workspace.
- **Candidates:**
  - Wen et al. 2025, ParaThinker (arXiv 2509.04475), on tunnel vision
    under a committed reasoning prefix.
  - Pipis et al. 2026, "Wait, wait, wait... why do reasoning models loop?"
    (arXiv 2512.12895), from the same group.
  - Mang et al. 2026, Frontier-CS (arXiv 2512.15699), and Zhu et al. 2026,
    EdgeBench (arXiv 2607.05155), as task sources for a pilot.
