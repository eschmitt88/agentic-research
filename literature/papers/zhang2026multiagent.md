---
kind: paper
title: "Multi-Agent Memory Inconsistency: Where Factual Conflicts Disappear in LLM Agents"
authors: ["Hejia Zhang", "Zhongming Yu", "Yimin Ding", "Wei-Ting Chen", "Hengjia Yu", "Wentao Ni", "Yijun Liu", "Cong Xu", "Xin Chen", "Matrix Yao", "Fan Zhao", "Jishen Zhao"]
institutions: ["UC San Diego", "Hewlett Packard Enterprise", "Intel"]
year: 2026
venue: "arXiv 2611.22245 (cs.AI), v2 2026-11-18, preprint (~8-page main text, no appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.22245"
code_url: null    # no code, data or profiler release named in the PDF
citations: null
source: "raw/papers/zhang2026multiagent.pdf"
added: "2026-11-24"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/lossless-context-offload]]"
tags: ["lead-worker", "orchestrator-worker", "handoff-omission", "worker-reports", "conflict-detection", "pair-recall", "evidence-flow-profiler", "partitioned-delegation", "same-vs-split", "single-agent-control", "harvey-lab", "planted-conflicts", "llm-judge", "budget-matched", "negative-result"]
---

# Multi-Agent Memory Inconsistency: Where Factual Conflicts Disappear in LLM Agents

## TL;DR

The authors plant 73 numerical conflicts (value A replaced by B in some
directories, kept elsewhere) into 11 Harvey LAB legal due-diligence tasks.
Agents get the ordinary "write a red-flags report" prompt; they are not
told conflicts were planted. Three policies run under the same per-task
dollar allowance:
- **None**: one agent reads and writes.
- **Optional**: the lead may read sources and may spawn up to ten workers,
  with overlapping reviews allowed.
- **Required**: the lead must delegate, workers read assigned document
  partitions, and the lead reads no sources.

A profiler tracks whether both claims of each conflict appear at four
stages: Source Reads, Worker Reports, Lead Evidence and Final Report
("Pair Recall"). An LLM judge (Claude Sonnet 5) scores whether the report
flags the conflict ("Judge Pass"). Models: GPT-5.6 Luna and GPT-6 Luna in
the Codex SDK, Claude Haiku 5.5 in the Claude Code SDK. Two runs each,
1,314 conflict observations.

Five results matter for this graph:

1. **Under Required, the loss is at the child's report, not in transport.**
   Of the 301 of 438 observations where workers collectively read both
   claims, only **106 (35.2%)** had both in some worker's outgoing
   messages. Worker Reports and Lead Evidence carry identical labels, so
   nothing was lost between sending and receiving. The workers chose not
   to report one side.
2. **Split provenance is what fails.** In matched cohorts defined by
   Required's source reads:
   - one worker read both claims (n = 111): Required **34.2%**, None 22.5%;
   - the claims were split across workers (n = 190): Required **10.0%**,
     None 18.9%.
   - Our arithmetic: a Judge Pass needs both values in the report, so
     passes sit inside "both read". Conditioned on having read both, None
     detects about 32% (Same cohort) and 37% (Split cohort), against 34%
     and 10% for Required. So one worker reading both is about as good as
     one agent reading both. The paper's "Required wins on Same" is
     mostly Required's retrieval advantage. The whole delegation penalty
     is the Split case.
3. **Even when both claims reach the lead, split provenance cuts
   detection.** Conditioned on Required's Lead Evidence: one contributor
   supplied both (n = 72) → **72.2%**; separate workers supplied them
   (n = 35) → **17.1%**. Under Split the lead wrote both values into the
   final report only 34.3% of the time. The paper does not say whether
   the single contributor in the Same case had already flagged the
   conflict in its report, so "the lead detected it" may be "the lead
   relayed a worker's flag".
4. **Discretionary delegation had the highest detection for every model,
   but only Haiku tests it.** Optional Judge Pass: GPT-5.6 23.3% (None
   15.8%, Required 10.3%), GPT-6 19.2% (17.1%, 11.6%), Haiku **30.1%**
   (15.8%, 17.8%). The GPT models spawned workers in only 9/22 and 10/22
   Optional executions, and their workers' reports held both claims for
   0.7% of conflicts. So the GPT Optional gains are not a delegation
   effect. Only Haiku (22/22 delegated, overlapping reads) supports
   "allow overlap and let the lead read".
5. **The single agent also loses most of what it read, at writing.**
   Under None, Source Pair Recall is 50.2% pooled but Final Report has
   both values for 18.3%. GPT-5.6 alone reads both for 69.9% and reports
   both for 21.2%. Detection is low everywhere (13.2–24.2% pooled).

## Claims

- Delegation creates a memory-consistency problem: facts retrieved by
  different workers may not reach the lead together, so no model can
  compare them.
- Pooled across models, Required raises Source Pair Recall from 50.2% to
  68.7% but lowers Judge Pass from 16.2% to 13.2% (headline).
- The largest measured loss is at the worker-to-lead handoff: workers
  "summarizing isolated partitions lack the cross-document context needed
  to recognize figures that appear unproblematic in isolation as
  conflicting", and omit one or both claims in 73.2% of split-evidence
  cases against 50.5% when one worker read both.
- Design advice: avoid "rigid, non-overlapping context partitioning on
  cross-document verification tasks"; overlapping access with lead source
  inspection (Optional) "consistently outperforms" both alternatives.
- End-to-end accuracy "masks opposing retrieval gains and coordination
  losses", so stage-wise profiling is needed. Structured reporting
  protocols and conflict-preserving memory are proposed, not tested.

## Methods

- **Tasks.** 11 Harvey LAB due-diligence tasks with their original
  instructions and corpora. 5–7 planted conflicts per task, 73 in total,
  all graded against the same report. The specs (values, locations,
  equivalent forms, rubric) stay with the evaluator.
- **Policies.** As in the TL;DR. Workers can also write to a shared report
  draft. The paper does not state who chooses the Required partitions or
  whether they follow the directories where B was planted.
- **Budget.** A common per-task dollar allowance ($59.60 per 11-task
  round). A run continues its session to revise the report while spend is
  under a set fraction. Mean spend is 0.95–1.02 × allowance per
  model/policy. Judging excluded.
- **Profiler.** A local proxy captures model traffic and correlates it
  with SDK events, native traces, tool outputs and report artifacts. A
  uniform matcher finds each claim's value via fact-specific anchors and
  verified equivalent forms. GPT-6 delegated runs use "plaintext
  collaboration traces" instead.
- **Metrics.** Pair Recall: the share of conflicts whose two values both
  appear at a stage, adapted from FEVER's complete-evidence-group rule.
  At the first three stages it is split into Same (one individual
  supplied both) and Split (only the union has both). Judge Pass: Sonnet
  5 requires an explicit flag of the specified conflict and an accurate
  quantitative comparison.
- **Scale.** 3 models × 3 policies × 11 tasks × 2 runs = 198 executions,
  146 observations per model/policy, 438 per pooled policy. Conflicts
  cluster within tasks and reports.
- **Statistics.** None. No intervals, no tests, no human agreement figure
  for the judge.

## Results

- **Table 1, per model** (Source Pair Recall / Lead Evidence / Final Both
  / Judge Pass, %):

  | Model | None | Optional | Required |
  |---|---|---|---|
  | GPT-5.6 Luna | 69.9 / 69.9 / 21.2 / 15.8 | 67.1 / 63.0 / 26.0 / 23.3 | 78.8 / 24.0 / 11.6 / 10.3 |
  | GPT-6 Luna | 59.6 / 59.6 / 17.8 / 17.1 | 65.1 / 63.7 / 19.9 / 19.2 | 61.6 / 18.5 / 12.3 / 11.6 |
  | Claude Haiku 5.5 | 21.2 / 21.2 / 15.8 / 15.8 | 56.2 / 37.7 / 31.5 / 30.1 | 65.8 / 30.8 / 23.3 / 17.8 |
  | **Pooled** | **50.2** / 50.2 / 18.3 / **16.2** | 62.8 / 54.8 / 25.8 / **24.2** | **68.7** / 24.4 / 15.8 / **13.2** |

- **The headline mixes directions.** Required vs None:
  - Source Pair Recall: GPT-5.6 +8.9, GPT-6 +2.1, Haiku **+44.5** pp.
  - Judge Pass: GPT-5.6 −5.5, GPT-6 −5.5, Haiku **+2.1** pp.
  - So the pooled retrieval gain is mostly Haiku, whose single agent read
    little (21.2%), and the pooled detection loss is the two GPT models.
    Each −5.5 pp is 8 of 146 observations. Pooled, 16.2% → 13.2% is 71 →
    58 of 438 (our arithmetic).
- **Where Pair Recall falls (Fig. 3c), in pp.** Required, Source → Lead:
  GPT-5.6 54.8, GPT-6 43.2, Haiku 34.9; then Lead → Final 12.3, 6.2,
  7.5. None loses everything at Lead → Final: 48.6, 41.8, 5.5.
- **Worker contribution (Fig. 3a).** Worker-report Pair Recall under
  Optional: 0.7% for both GPT models, 37.0% for Haiku. Under Required:
  24.0%, 18.5%, 30.8%.
- **Matched cohorts (Fig. 4).** By Required's Source Reads: Same n = 111,
  Split n = 190. Required lead held both in 49.5% of Same and 26.9% of
  Split; omission of one or both claims 50.5% vs 73.2%. Judge Pass as in
  TL;DR item 2. None read both in 69.4% of the Same cohort and 51.6% of
  the Split cohort. By Required's Lead Evidence: Same n = 72 (Required
  72.2%, None 33.3%), Split n = 35 (17.1%, 28.6%).
- **Shared-draft effects.** Reading the shared draft and arithmetic
  reconstruction can turn a source-level Split into a lead-level Same, so
  the two groupings are not nested.
- **Judge errors.** Manual inspection found three saved Judge Pass labels
  that credited a neighbouring comparison, not the planted conflict.

## Critique / open questions

- **Against the digest entry: below.** The numbers (50.2 → 68.7%, 16.2 →
  13.2%) are in the PDF and quoted correctly. Three overreaches:
  - **"Delegation ... lowers conflict detection"** is true only of
    *Required*, which bundles mandatory delegation, partitioned sources
    and a lead barred from reading. Discretionary delegation (Optional)
    had the *highest* detection for all three models (24.2% pooled). The
    bullet drops the paper's most usable contrast.
  - **The pooled pair is two different models' stories.** The retrieval
    gain is mostly Haiku (+44.5 pp); the detection loss is the GPT models
    (−5.5 pp each, 8 of 146), and Haiku's detection rose. The pooled −3.0
    pp is 13 of 438 clustered observations, with no interval.
  - **"The loss happens at worker-to-lead handoffs"** is right in place,
    but the mechanism is worker *omission*, not loss in transit (Worker
    Reports = Lead Evidence). And there is a second loss the bullet
    misses: with both claims at the lead from different workers,
    detection is 17.1% (n = 35).
- **The cut reason, after the full read.**
  - *Overlap with item 11 (xie2026do):* only partly holds. xie2026do
    fixes the child material, has no non-hierarchical control, and finds
    the *parent* dropping qualifiers. This paper has live workers, a
    single-agent control at matched spend, and finds the *child*
    dropping a claim. It also shows that split provenance at the parent
    is its own failure. The two are complementary, not redundant.
  - *Small n:* holds. 11 tasks, two runs, no uncertainty, and the
    headline differences are a handful of observations.
  - *Method-first:* partly. The profiler is a contribution, but it is not
    released, and the results carry the paper.
- **Required confounds three things.** Mandatory delegation, partitioned
  assignment and a source-blind lead change together (authors'
  limitation). The partition rule is not described. If partitions follow
  directories, the planted B values sit in one partition and A in
  another, so many Split cases are built in by the corpus.
- **The Same-cohort "advantage" is a retrieval effect** (our arithmetic,
  TL;DR item 2). It does not show that a narrower worker context detects
  better than one agent that read the same two claims.
- **Workers were not told to report figures for cross-checking.** The
  lead's briefs to workers are not analysed. Whether a brief that asks
  workers to return every in-scope figure with its location would close
  the gap is the obvious untested arm. The paper names "structured
  reporting protocols" as future work.
- **Retrospective choices on a load-bearing cell.** Budgets were set from
  observed spending, and "GPT-5.6 Required checkpoints were selected
  retrospectively" (authors' limitation). GPT-5.6 Required is one of the
  two cells that carry the detection drop.
- **Measurement is uneven.** GPT-6 delegated runs use a different trace
  source. The judge is Sonnet 5, the same family as one subject, with no
  agreement statistic and three known false passes.
- **Draft artifacts.** The v2 introduction still contains an editor's
  note: "(NOT SURE the definition is widely used. See my comment)".
- **Small, cheap models only.** No Opus- or Sonnet-class agent. Absolute
  detection is low in every arm, so the deltas sit on a floor.
- **Consistent with neighbours in the graph.**
  - [[literature/papers/xie2026do]]: the parent's synthesis loses
    qualifiers from fixed child material. Here the child's report loses
    one side of a conflict, and the parent misses conflicts that arrive
    split.
  - [[literature/papers/hu2026analyzing]]: subagents return summaries,
    not retrieved content, so the parent re-reads. Here the parent cannot
    re-read (Required), and what the summary dropped is lost.
  - [[literature/papers/yoon2026arcticswarm]]: isolate while gathering,
    share while integrating. This paper shows the integration channel
    has to carry the evidence, not each worker's judgment of it.
  - [[literature/papers/liu2026agents]]: a writer given the raw record
    compresses far better than one given the previous summary.

## Trust signals

- **Credibility:** 2.
  - For:
    - A known systems group (Jishen Zhao, UC San Diego) with Intel and
      HPE co-authors.
    - A stage-wise profiler built on captured traffic, not self-report.
    - A single-agent control at matched dollar spend, and matched-instance
      cohorts.
    - Candid limitations, including the retrospective checkpoint
      selection and the judge errors.
  - Against:
    - Not peer reviewed, no citations, no code, data or profiler
      released.
    - No intervals or tests; 11 tasks and two runs, with conflicts
      clustered in reports.
    - Retrospective checkpoint selection and budget calibration on the
      cells that carry the headline.
    - Judge from a subject's family, no human agreement figure.
    - An unresolved editor's note in v2.
  - Not 3: none of the signals the rubric weights over institution (code,
    review, reproducibility) are present, and the headline rests on a few
    observations. Not 1: the authorship is verifiable and the design is
    sound in outline.

## Follow-up

- **Relevance:** 4.
  - The first evidence in the graph that locates loss in a lead–worker
    hierarchy stage by stage, with a single-agent control at matched
    spend. Material new evidence for [[concepts/hierarchical-delegation]]
    (the child's report omits; split provenance defeats the parent) and
    for the detection half of [[concepts/shared-substrate-contagion]]
    (partitioning removes cross-partition detection).
  - Not 5: legal documents rather than ML research, small cheap models,
    small n, credibility 2.
- **Untested arm worth running here or downstream (our inference):**
  Required with a reporting contract (every in-scope figure with its
  path), against Required with the default brief, on the same corpora.
  That separates a brief fix from a topology fix.
- **Candidates** (cited, not yet in the graph):
  - Wang et al. 2026, "Facts without rules: boundary metadata collapse in
    multi-agent LLM handoffs" (EMNLP 2026 Findings). Constraints lost at
    handoff; peer reviewed. Also a candidate in chen2026signed.
  - Volkov, Li and Luo 2026, "StateFuse: deterministic conflict-preserving
    memory for multi-agent systems" (arXiv 2607.05844).
  - Li, Naito and Shirado 2026, "Systematic failures in collective
    reasoning under distributed information in multi-agent LLMs" (ICML
    2026; HiddenBench).
  - Tran and Kiela 2026, single agents beat multi-agent systems on
    multi-hop reasoning at equal thinking tokens (arXiv 2604.02460). Also a
    candidate in kanke2026rtaste and shi2026weight.
  - Jwalapuram et al. 2026, "The illusion of multi-agent advantage"
    (arXiv 2606.13003).
  - He et al. 2026, an information-theoretic view of compressor–predictor
    agent systems (ICLR 2026).
  - Yu et al. 2026c, multi-agent memory from a computer-architecture
    perspective (same group).
