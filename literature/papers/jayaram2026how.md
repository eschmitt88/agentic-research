---
kind: paper
title: "How Much Should an Agent Remember? Deployment Lessons from Self-Distilled Memory for LLM Agents"
authors: ["K. R. Jayaram", "Vatche Isahagian", "Aswath Vaithinathan Aravindan", "Vinod Muthusamy", "Punleuk Oum", "Gaodan Fang", "Michal Ulewicz", "Gegi Thomas", "Evelyn Duesterwald"]
institutions: ["IBM"]
year: 2026
venue: "arXiv 2611.26361 (cs.AI), v1 2026-11-19; AAAI-format deployment-track paper (copyright line reads AAAI 2027; acceptance not stated)"
peer_reviewed: unknown
url: "https://arxiv.org/abs/2611.26361"
code_url: "https://github.com/AgentToolkit/altk-evolve"    # the library, named in the paper, not opened at ingest; experiment scripts not named
citations: null
source: "raw/papers/jayaram2026how.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["self-distilled-memory", "guidelines", "consolidation", "support-count", "dose-response", "lossless-consolidation", "retrieval", "provenance", "noise-floor", "scaffold-confound", "silent-failure", "appworld", "tau-bench", "ace", "deployment-report", "altk-evolve", "cuga"]
---

# How Much Should an Agent Remember? Deployment Lessons from Self-Distilled Memory for LLM Agents

## TL;DR

IBM's altk-evolve pipeline has the agent's own model write 1–3 short
lessons per train/dev trajectory, then merge near-duplicates into
**guidelines** that each carry a **support count** (how many lessons
merged into it). At inference the agent gets all guidelines, a
support-thresholded subset, or a high-support core plus top-k retrieved
by source-task similarity. The paper is a deployment report: the authors
say the method is not the contribution. The main instrument is AppWorld
`test_normal` (168 tasks), single runs, with configurations compared on
the test split (no held-out split was left).

Results that matter for this graph:

1. **A lossless store served whole never fell below no memory.** Under
   the rule fixed before any test run (inject all guidelines), gains
   were +9.5 (DeepSeek-V3.2, 79.8 → 89.3), +11.3 (gpt-oss-120b, 39.9 →
   51.2), +2.9 (GPT-5.5), +1.2 (Gemma-4-31B) and 0.0 (GLM-5). DeepSeek's
   gain holds on the 417-task hard split (60.4 → 72.9), and a second
   model in a second scaffold repeats it (GLM-5.3-Flash in CUGA, 87.8 →
   93.2, two runs per arm with non-overlapping ranges).
2. **Compressing to a short checklist kept about a third of the gain.**
   For DeepSeek, a 12-rule "small set of highly relevant rules" gave
   +2.9; the full 191 gave +9.5 (11 tasks apart). The 47 guidelines with
   support ≥ 2 already gave +5.9, so the 144 support-one guidelines
   added +3.6 (about 6 tasks). The paper's "the value lives in the long
   tail" is stronger than that step.
3. **The headline "dose is a property of the model" rests on two swept
   models and a gap inside the noise.** gpt-oss scored 55.4 with ~29
   retrieved guidelines (mean of 56.0 and 54.8) against 51.2 with all
   199 (one run): about 7 tasks. Identical DeepSeek runs flipped about
   32 of 168 tasks. On that figure a 7-task net difference is about 1.2
   SD of pure noise (our arithmetic). Each model also wrote and merged
   its own guidelines, so reader capability and writer capability are
   not separated.
4. **Support is recurrence, not correctness.** A high-support
   pagination guideline ("loop until fewer than the page limit are
   returned") assumed a limit of 20 where the API default was 5. It
   caused a regression the baseline did not have. The pipeline has no
   contradiction, staleness or harm signal; the authors would not run it
   unattended without one.
5. **Engineering failures were silent.**
   - Consolidation once rebuilt metadata from a fixed field list and
     dropped the support count. Support attribution double-counted
     repeated indices.
   - A swallowed HTTP 400 failed every task after one model call behind
     a plausible log.
   - In CUGA the memory write path never fired, so a 50-task
     evaluation ran with an empty store and reported "no significant
     effect".
6. **A comparison with ACE nearly misled.** DeepSeek's 89.3 against
   ACE's 80.4 is mostly scaffold: ACE's ReAct variant has its own
   no-memory baseline of 72.0. As lift over own baseline the two tie
   (+9.5 vs +8.4 TGC), with ours at ~40% of ACE's per-task tokens.

## Claims

- The right amount of memory is a function of model capability, not of
  the memory. Strong models keep improving as guidelines are added; weak
  ones gain more from a selected subset (Insight 1).
- Compressing memory into a tidy rule set discards most of its value,
  which lives in one-off, situational lessons (Insight 2). Lossless is
  now the shipped default; compression is opt-in.
- Selection by support or source task needs those fields to survive
  consolidation. Any field a later stage ranks on must be an enforced,
  tested invariant of every earlier stage (Insight 3).
- Support measures how often a lesson was derived, which is evidence of
  generality, not correctness (Insight 4).
- Temperature 0 and a fixed seed do not make runs comparable;
  differences under ~5 tasks are treated as noise (Insight 5).
- Memory methods must be compared on a fixed scaffold, as deltas, with
  tokens reported (Insight 6).
- Silent failures cost more calendar time than any modelling problem:
  fail loudly per task, assert a minimum number of model calls per task,
  watchdog long runs, check that the store grows (Insight 7).
- The shipped default is retrieval with a high-support core, because
  too much memory on a weak model fails silently.

## Methods

- **Pipeline (altk-evolve).**
  - Trajectories are captured by instrumenting client libraries, syncing
    from an observability store, or an MCP `save_trajectory` call.
  - Extraction: an LLM writes 1–3 imperative lessons per trajectory.
    Success/failure polarity comes from the evaluator's verdict, never
    from the model's self-report. DeepSeek: 147 trajectories → 290
    lessons (62 from failures, 228 from successes).
  - Consolidation: embed, cluster on source-task embeddings, LLM-merge
    within a cluster only when following one lesson is equivalent to
    following the other. Invariant: total support out equals the number
    of lessons in; uncovered lessons pass through verbatim. DeepSeek:
    290 → 191 guidelines, 144 with support 1.
  - Delivery: static injection of a set, or retrieval: a core of
    support ≥ 3, plus top-k by source-task similarity, deduplicated
    against the core. "sup2" / "sup3" thresholds are applied at
    selection time and never delete from the store.
  - The same model runs the agent, extracts and consolidates. No teacher,
    no human labels.
- **Benchmark.** AppWorld (9 apps, 457 APIs). The reference ReAct code
  agent. `test_normal` (168 tasks) throughout, `test_challenge` (417)
  where noted. TGC: share of tasks passing all checks. SGC: share of
  scenarios where every task variant passes. Memory mined from train/dev
  only.
- **Models.** Eight baselined, six run with memory: GPT-5.5, Claude Opus
  4.6, GLM-5, DeepSeek-V3.2, Gemma-4-31B, gpt-oss-120b.
- **Configurations compared on test.** Five for DeepSeek, four for
  gpt-oss, two each for GPT-5.5, GLM-5 and Gemma, one for Opus (the
  12-rule compressed set only). Table 2 reports each model's best next
  to the fixed "inject all" rule.
- **Runs.** Single runs, except gpt-oss retrieval (two runs) and the
  CUGA re-runs (three trials with GPT-4.1; two runs per arm with
  GLM-5.3-Flash).
- **ACE comparison.** ACE run in-house on the same models and splits,
  inside its own ACE-ReAct scaffold.
- **Deployment.** CUGA (IBM multi-agent assistant; memory in three
  sub-agents) on AppWorld and the τ-bench retail split (40 test tasks).
  watsonx Orchestrate: one qualitative expense-claim trace.

## Results

- **Baselines and the fixed all-guidelines rule (Table 2, TGC).**

  | Model | Base | All (n) | Δ | Best config | Best TGC/SGC |
  |---|---|---|---|---|---|
  | GPT-5.5 | 92.3 | 95.2 (207) | +2.9 | all | 95.2 / 89.3 |
  | Opus 4.6 | 90.5 | not run | – | lossy (12) | 94.6 / 94.6 |
  | GLM-5 | 87.5 | 87.5 (72) | 0.0 | all | 87.5 / 80.4 |
  | DeepSeek-V3.2 | 79.8 | 89.3 (191) | +9.5 | all | 89.3 / 80.4 |
  | Gemma-4-31B | 63.4* | 64.6 (87) | +1.2 | all | 64.6 / 45.5 |
  | gpt-oss-120b | 39.9 | 51.2 (199) | +11.3 | retr. (≈29) | 55.4 / 37.5 |

  *Gemma's memory runs used a 164-task subset.
- **DeepSeek dose sweep (Fig. 1).** TGC 79.8 → 82.7 (12, compressed)
  → 83.9 (19, sup3) → 85.7 (47, sup2) → 89.3 (191, all). Adjacent
  steps are 2–4 tasks; endpoints 16. SGC 64.3 → 80.4. Hard split, all
  guidelines: 60.4 → 72.9 TGC, 41.7 → 58.3 SGC.
- **gpt-oss dose (Fig. 1).** 39.9 (none), 53.6 (24, sup3 core), 55.4
  (~29 retrieved; 56.0 and 54.8), 51.2 (199, all). Retrieval costs 116K
  tokens per task against 110K baseline.
- **By difficulty (Table 1).** DeepSeek's medium-tier SGC goes 62.5 →
  93.8. gpt-oss hard-tier SGC stays at 4.8 → 9.5. Opus 4.6 on the hard
  split's hard tier: 73.3 → 83.1 TGC, with the 12-rule set.
- **Noise (Insight 5).** Two identical gpt-oss runs: 56.0 and 54.8.
  About 32 of 168 tasks flip between identical DeepSeek runs (number of
  repeat runs not stated). Retrieval cosine did not predict improvement:
  0.463 for improved tasks, 0.468 for regressed ones.
- **ACE (Table 3).**
  - DeepSeek: ACE-ReAct 72.0 → 80.4 (634K tokens/task); ReAct 79.8 →
    89.3 (263K). Lift over own baseline +8.4/+16.2 vs +9.5/+16.1.
  - gpt-oss: ACE 54.8 (777K); ours, all 51.2 (166K); ours, retrieval
    55.4 (116K).
- **Cost.** Overhead is input-side; steps barely move (18.4 → 19.3).
  All 191 costs +78% tokens; the 47-guideline sup2 set +27% for +5.9 TGC.
- **Pagination guideline (Insight 4).** After a hand rewrite ("pass an
  explicit page limit and loop until a page comes back empty") the task
  recovered and DeepSeek hard-tier TGC rose 66.7 → 73.0.
- **CUGA, GPT-4.1, three trials (Tables 4–5).** AppWorld lossless 71.03
  → 72.42 TGC, 54.76 → 58.93 SGC. τ-bench retail, sup3 (12 guidelines)
  51.7 → 55.8 accuracy, pass^3 27.5 → 32.5. The authors note each delta
  is about two tasks per trial, below their noise floor.
- **CUGA, GLM-5.3-Flash, two runs per arm (Table 6).** None 87.8, sup3
  (29) 88.4, lossless (164) 93.2 TGC (ranges 87.5–88.1, 87.5–89.3,
  92.9–93.5). +36% tokens, steps unchanged.
- **watsonx Orchestrate.** One trace: a guideline learned from a HEIC
  receipt task led a later TIFF task to convert before submitting, once
  instead of twice (13 → 10 steps). Descriptive only.

## Critique / open questions

- **Against the digest entry: at.** The PDF is the work described
  (arXiv 2611.26361, IBM, CUGA / watsonx Orchestrate). The bullet's
  three facts are in the paper and correctly stated. But it repeats the
  abstract's headline at full strength, and the side findings are worth
  more than it says.
- **The headline is weaker than the abstract.**
  - Only DeepSeek and gpt-oss were swept over dose. The other four have
    1–2 configurations, so "the right amount is a property of the model"
    rests on two models.
  - The gains do not order by capability. GPT-5.5 (92.3) gains +2.9,
    GLM-5 (87.5) 0.0, DeepSeek (79.8) +9.5, Gemma (63.4) +1.2, gpt-oss
    (39.9) +11.3. "Headroom plus whether the model can use the lessons"
    fits as well as "strong vs weak".
  - The conclusion calls DeepSeek "mid-tier"; Insight 1 calls it the
    "strong" case. The one frontier model with a full set (GPT-5.5) gains
    about 5 tasks, at the noise floor.
  - The weak-model "selection wins" gap is about 7 tasks against a
    one-run full set, with the winner picked from four configurations on
    test. With ~32 discordant tasks between identical runs, the SD of a
    net difference is about √32 ≈ 5.7 tasks (our arithmetic, applying
    DeepSeek's flip count to gpt-oss). So 7 tasks is about 1.2 SD. The
    authors' own "under ~5 tasks is noise" threshold is about 0.9 SD:
    lenient.
  - "Not the memory" is not tested. Each model mined and consolidated
    its own guidelines. gpt-oss's 199 guidelines were written and merged
    by gpt-oss. No arm gives one model another model's set, so reading
    capability and writing capability are confounded. That matters:
    [[literature/papers/razmara2026one]] found a model merging for
    itself scores 5–25 points below a strong merger.
- **The long-tail claim is partly a dose claim.** The compressed 12
  rules (82.7) scored about the same as the 19 highest-support
  guidelines (83.9). Of the 11-task gap from 12 rules to all 191, the
  support ≥ 2 subset already carries about 5 tasks; the 144 support-one
  guidelines add about 6 tasks, ~1.1 SD in one run. The direction is
  consistent, and the endpoints (16 tasks, ~2.8 SD) are solid. "The
  value lives in the one-off lessons" overstates the attribution.
- **Opus is the counterexample nobody ran.** The only Claude model
  gained +4.1 on `test_normal` with the 12-rule compressed set, and
  moved the hard tier on the hard split. Its full set was never run, so
  "do not compress" is untested on the most capable model in the study.
- **The ACE "tie at one-seventh the tokens" on gpt-oss commits the
  confound Insight 6 warns about.** Table 3 gives no ACE-ReAct
  no-memory baseline for gpt-oss. On DeepSeek that scaffold was 7.8
  points below plain ReAct. So 55.4 vs 54.8 compares final scores
  across scaffolds. The token comparison is also scaffold-confounded:
  ACE-ReAct with no memory already costs 221K vs 148K on DeepSeek. As
  overhead over own baseline, ours adds 115K and ACE adds 413K, so the
  efficiency point survives in delta form.
- **The pagination fix is a hand edit after a test-set regression.**
  The 66.7 → 73.0 hard-tier figure comes after it. Whether the edited
  guideline is in the 191-guideline numbers is not stated.
- **No intervals, single runs, configuration chosen on test.** The
  authors say so and report the fixed rule. The fixed rule's results
  (no model below baseline) are the cleanest evidence in the paper.
- **Insight 3 and Insight 7 are incident reports**, not measurements.
  Two provenance bugs, three silent failures. They are credible and
  specific, and n is tiny.
- **CUGA GPT-4.1 deltas are below noise** (the authors' own statement).
  The GLM-5.3-Flash table is the only clean second-scaffold result.
- **Offline batch extraction differs from online growth.** Lessons were
  mined once from train/dev with evaluator-graded polarity, then frozen.
  [[literature/papers/pham2026reliable]] found ACE playbooks grown
  online often scored below no playbook. That is consistent here only
  if the difference is the growth regime and the graded labels; the
  paper does not test either.
- **Relevance to research agents is indirect.** AppWorld tasks are
  repeated, procedural API tasks with an authoritative verdict. Research
  tasks are mostly one-off, so a store mined from a train split has no
  direct analogue there.

## Trust signals

- **Credibility:** 3.
  - For:
    - An IBM Research group with prior work on the same pipeline
      (Fang et al. 2026, arXiv 2603.10600).
    - Open-source library with named PRs; deployed in two products.
    - Six models over four families, a hard split, a second scaffold
      and a second benchmark.
    - A pre-test fixed rule reported beside the test-selected best.
    - Candid failure reporting, including a confound that undercut its
      own first headline.
  - Against:
    - Peer review unknown. The AAAI copyright line is in the template,
      not proof of acceptance.
    - Single runs, no intervals, configurations chosen on test.
    - The weak-model headline gap is ~1.2 SD of the paper's own run-to-run
      noise.
    - Writer and reader are the same model in every arm.
    - The ACE gpt-oss comparison lacks the baseline its own Insight 6
      requires.
    - Experiment scripts not named.
  - Not 4: unknown review and an under-powered headline. Not 2: a
    reputable industry lab, released library, and the fixed-rule
    results are clean at their own scale.

## Follow-up

- **Relevance:** 4.
  - It gives this graph's first per-model dose sweep for a self-written
    store, measured against a pre-fixed rule. Under that rule a
    lossless store served whole stayed at or above no memory for all
    five models it was tried on. That bears directly on the 10-19
    "served whole can cost more than no store" section of
    [[concepts/agent-native-memory]].
  - Material new evidence for [[concepts/selective-memory-retrieval]]
    (selection versus whole-serving, by model) and
    [[concepts/verified-memory-writes]] (support counts are not
    correctness; provenance fields must be invariants).
  - Not 5: not a research agent; the central claim rests on two models
    and a confounded design.
- **A cheap test the paper leaves open (our proposal):** give gpt-oss
  DeepSeek's 191 guidelines, and DeepSeek gpt-oss's 199. That would
  separate "the reader cannot use a long set" from "the writer wrote a
  worse set".
- **For the box (our inference):** the silent-failure list (assert a
  minimum of model calls per task, check the store grows) is a cheap
  health check for scheduled jobs that write memory or literature. See
  the /elevate notes in the curation report.
- **Candidates** (cited, not yet in the graph):
  - Zhang et al. 2026, ACE (arXiv 2510.04618, ICLR 2026). Used by
    pham2026reliable and here; still no note of its own.
  - Fang et al. 2026, trajectory-informed memory generation (arXiv
    2603.10600). The pipeline's predecessor.
  - Zhao et al. 2024, ExpeL (AAAI). Up/down-vote insight maintenance.
  - Wang et al. 2025, Agent Workflow Memory (ICML).
  - Lin et al. 2025, sleep-time compute (arXiv 2504.13171).
  - Anthropic 2026, "Dreams" (Claude Managed Agents docs): recency-based
    contradiction resolution, no multiplicity.
