---
kind: paper
title: "What Coding Agents Leave Unsaid: Measuring Task-State Gaps Between Execution and Reports"
authors: ["Guoyang Xu", "Rui Zhang", "Xiuyu Wu", "Hao Chen"]
institutions: ["Tencent"]
year: 2026
venue: "arXiv 2610.20003v2 (cs.AI), v2 2026-10-15, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.20003"
code_url: "https://github.com/Kaguraaaaaa/Ctxgap"
citations: null
source: "raw/papers/xu2026what.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["agent-reports", "task-state-recoverability", "supervision", "handoff", "report-only-reader", "omission", "mentioned-but-insufficient", "failed-attempts", "probe-qa", "swe-bench-verified", "claude-code", "pi-harness", "deepseek-harness", "matched-length-rewrite", "llm-judge", "measurement-chain-replication", "ctxgap"]
---

# What Coding Agents Leave Unsaid: Measuring Task-State Gaps Between Execution and Reports

## TL;DR

CTXGAP measures how much of a coding agent's task state a reader can
recover from the agent's user-facing reports, compared with a reader who
gets the full recorded execution context. An LLM extracts "state atoms"
(facts, failures, progress, risks, decisions, ...) from each checkpoint's
full context, writes a question and reference answer per atom, and a fresh
observer answers each question from one view. A judge grades the answers.

On 37 SWE-bench Verified tasks (one HY4-preview trajectory each, 4
checkpoints, 4,799 valid probes):

- Reports-only reader (U, ~6.6k chars): 52.0%.
- Full terminal scrollback (V, ~25k chars): 73.6%.
- Full context with reasoning (T, ~167k chars): 92.7%.
- The actor itself, forked at the checkpoint: 93.6%.
- Gap T − U = 40.7 points [37.3, 43.8]. On the final checkpoint of the
  30 tasks the hidden tests mark as fixed, it is 34.8 [28.2, 41.3],
  positive in 28/30.

The paper's own human audit finds that most of the missing state was
**mentioned but not specified well enough to answer the question**
(55.4% / 59.2% of gap mass for two annotators), not left out entirely
(42.1% / 33.7%). Adding a bare mention recovers nothing (−0.2 points).
Rewriting the report from full context at the same total length recovers
about 8 points, and a category-targeted rewrite does no better than a
natural one.

## Claims

- Task success, on-demand answerability and what reaches a supervisor
  through reports are three different things. Benchmarks measure the
  first.
- "Routine reports expose far less state than direct queries" (Finding 1):
  93.6% on demand vs 52.0% from reports.
- The gap includes "supervision-relevant states": 35.3 / 36.4 points on
  probes HY4-preview / DeepSeek label decision-relevant, 36.1 on the
  jointly relevant subset (Finding 2).
- "More terminal visibility reduces but does not close the gap": scrollback
  closes 53.0% [47.6, 57.8] of it and leaves 19.1 points (Finding 3).
- Context-informed rewrites at matched length raise recovery by 8.3
  (natural) and 8.5 (targeted) points, "without a detectable
  targeted-over-natural advantage".
- Positive gaps recur across three harnesses, an independent DeepSeek
  executor, and an independent DeepSeek measurement chain.
- Explicit scope statements: decision relevance is model-assessed;
  interventions measure offline recovery, "not human action or code
  outcomes"; no behavioural study with end users was run.

## Methods

- **Tasks and trajectories.** A 76-task inventory of SWE-bench Verified
  issues, one trajectory per issue. The headline uses 37 of them, chosen
  "following availability", "not a preregistered random 37-of-76 draw".
  The 37 over-represent easy tasks (40.5% under 15 min vs 23.7% in the
  76; 10.8% hard vs 25.0%). Each trajectory runs a fixed 4-turn script:
  investigate without edits, implement a fix, run tests and report, assess
  remaining risk.
- **Views.** U_k = user instructions plus the agent's final report for
  each completed turn. V_k = default-visible terminal scrollback. T_k =
  full context: tool calls, outputs, narration and reasoning. A reading
  ladder adds narration, execution headers and tool outputs between U and
  V (Table 14).
- **Probes.** HY4-preview extracts atoms from T_k, each with a category
  and an importance score (1–3). Only importance ≥ 2 is sampled.
  Importance is extractor-assigned and "not human-validated". One model
  call generates the question and the reference answer. A probe is kept
  only if its task-description-only score is below full credit and its
  full-context score is above zero: 4,799 of 4,855. The authors note this
  makes the population outcome-conditioned (+0.56 points on the gap).
- **Scoring.** Observer and judge are HY4-preview by default. Judge gives
  1 / 0.5 / 0. Three observer samples per probe per view. Probes are
  inverse-weighted by category sampling rate. CIs are issue-clustered
  bootstraps (10,000 draws, 37 clusters).
- **Validity audits.** Reference-paraphrase regrade (300 probes);
  question-aligned rescoring; dual human raters on 360 non-forced probes;
  a 195-atom human mention/sufficiency audit (κ = .632 / .638); LLM
  relevance labels (κ = .637); literal-detail labels; a generator ×
  observer swap with DeepSeek on 413 atoms; a fully independent DeepSeek
  extraction-to-grading chain on the same 37 trajectories.
- **Cross-harness and cross-executor.** 12 shared issues, one trajectory
  per harness: Claude Code 2.1.154, Pi, DSH, all on HY4-preview. A
  separate arm runs DeepSeek-v4-flash as executor under DSH on the same
  12.
- **Reporting interventions.** (E2) 129 checkpoint units: original
  reports vs natural vs targeted rewrites generated from T at matched
  total length. (Held-out) 46/30 dev/test task split; added summaries
  (natural, structured, targeted on dev-selected categories: failures,
  facts, progress, environment), scored on "non-inferable" probes.
- **Simulated supervision.** A model supervisor writes the next
  instruction from U, U + the omitted atom, or U + an already-known atom.
  The endpoint is whether the instruction changes.

## Results

- **Main table** (37 tasks, 4,799 probes; Tables 1, 5, 14):

  | View | chars | Acc | gap to T |
  |---|---|---|---|
  | Latest report only | 2.6k | .239 | .688 |
  | All reports (U) | 6.6k | .520 | .407 |
  | U + last tool call | 6.8k | .525 | .402 |
  | + exec headers | 13.1k | .639 | .288 |
  | Scrollback (V) | 25.5k | .736 | .191 |
  | Context w/o reasoning | 107k | .890 | .037 |
  | Full context (T) | 167k | .927 | — |
  | Actor fork (A) | — | .936 | — |

  Reasoning adds only 3.7 points (9.1% of the gap). Nearly all the gap is
  in tool outputs and narration, not hidden thinking.
- **Success-conditioned.** 30/37 tasks fixed at checkpoint 4. On those,
  U = .571, T = .919, gap .348. Checkpoint 4's instruction was itself
  "assess remaining risk", so this is residual loss after the agent was
  asked, not unprompted omission.
- **Persistence.** Unrecovered atoms per checkpoint grow 9.4 → 19.0 →
  21.0 → 22.3, of which 18.9 at k = 4 come from earlier turns. The
  normalized actor gap does not grow (.426, .472, .436, .359).
- **Anatomy.** By category, facts carry 42.8% of gap mass, failures
  16.6%, progress 15.9%, environment 8.2%, risks 5.0%, rejected options
  3.5%. The automated classifier labels 89.4% of the failure gap and
  73.0% of the rejected-options gap "not conveyed", the two highest
  shares. That classifier is an LLM judgment. Humans say "not conveyed"
  usually means "mentioned but insufficient" (Table 23: 47–57% of the
  frozen "omitted" stratum was actually mentioned).
- **Mention vs evidence** (99 human-screened candidates, 33 tasks, 3
  observers). Adding a mention: −0.002 [−0.027, +0.025]. Adding
  pre-checkpoint tool evidence selected with access to the reference:
  +0.189 [+0.137, +0.241]. The gain scales with lexical overlap between
  evidence and reference (slope +0.88; top quartile +0.44, bottom +0.03),
  so part of it is answer leakage.
- **Literal detail.** On 159 probes two LLM families agree on, the
  non-literal subset still has a .369 [.290, .444] gap; literal-dependent
  .479. The difference is not significant. Reference paraphrasing moves
  the gap by −0.010.
- **Matched-length rewrites (E2).** A .672, B natural .755, C targeted
  .757. C − B = +0.002 [−0.015, +0.018].
- **Added summaries, held-out** (Table 3): non-inferable recovery reports
  .017, natural .031 (1.6k chars), structured .281 (3.2k), targeted .602
  (5.7k), targeted at natural's budget .110 (1.5k). At matched budget,
  targeted beats natural by +.077 [.030, .133] on non-inferable state;
  the overall-recovery difference under realized-length matching is +2.4
  to +2.5 points.
- **Simulated supervision.** Restoring an omitted atom changes the next
  instruction 71.8% of the time vs 52.5% for a known atom (+18.9
  [10.2, 28.0]). In the continuation check, 4 of 14 pairs favour the
  informed arm and 0 the control. No task-outcome effect is
  measured.
- **Cross-harness** (12 issues, 36 trajectories): gaps .407 (Claude
  Code), .390 (Pi), .490 (DSH). All pairwise differences include zero.
  Executor swap under DSH: HY4 .522 vs DeepSeek .527 (single sample,
  task-equal).
- **Independent chain.** DeepSeek extraction-to-grading on the same 37
  trajectories: .398 [.348, .456] vs .407. Generator × observer swaps on
  413 atoms: .345–.400 in all four cells.
- **Cohort.** The 39 tasks outside the headline have a larger gap (.462);
  all 76 give .438.

## Critique / open questions

- **Against the digest framing: BELOW (mild).** Every number in the entry
  is in the paper and checks against Table 1 / Table 5. Two sentences go
  beyond it:
  1. "Failed attempts, workarounds and unresolved risks drop out of
     reports." The paper's own human audit says the largest share of
     missing state was *mentioned but insufficiently specified*, and the
     authors write that this "invalidates the literal reading of '71.1%
     not conveyed' as '71.1% never mentioned'". Failures being the
     least-conveyed category rests on the automated classifier only.
  2. "It holds across harnesses." This is 12 issues, one trajectory per
     harness, all on the same executor model. Each harness generates its
     own probes, so the authors themselves say the numbers are not a
     ranking. "Positive in all three" is supported; "holds" as a
     robustness claim is thin.

  The success-conditioned 34.8 is reported faithfully, but at checkpoint
  4 the agent had just been told to assess remaining risk.
- **The instrument favours the full-context reader by construction.**
  Atoms are extracted from T, questions and references are written from
  T, and probes are kept only if the T reader scores above zero. A reader
  of T is reading the source text of the question. Some references ask
  for exact commands, file names or line numbers (G.2). The paper tests
  this (paraphrase −0.010; non-literal subset still .369), which is
  better than most. But the T ≈ 93% ceiling means "the context holds it",
  not "the actor knows it". The actor fork has the same context, so A ≈ T
  says little about the agent.
- **There is no notion of a good report.** The denominator is everything
  the extractor scored importance ≥ 2. A report that recovered 92% would
  be close to the transcript in length. The paper partly addresses this
  with length-matched arms, but it never asks what fraction of state a
  supervisor *should* get, and decision relevance is labelled by the same
  model families, not by people who supervise. The supervision arm
  measures whether an instruction changes, not whether it improves.
- **Shared substrate.** HY4-preview is executor, extractor, probe writer,
  observer and judge in the main chain (Table 12: 8,258 observer and
  9,929 judge calls, all HY4). That is exactly the correlated-error
  setup [[concepts/shared-substrate-contagion]] warns about. The
  independent DeepSeek chain landing at .398 is a real check, and the
  right way to do it. It still runs on the same 37 frozen HY4
  trajectories, so the executor is not varied there.
- **Non-random cohort and single trajectories.** 37 of 76 tasks chosen by
  availability, skewed easy; one run per task; mixed prompt versions
  (v3.0 / v3.2; the v3.0 source is lost). The omitted 39 tasks have a
  *larger* gap, so selection does not obviously inflate the headline.
- **The evidence-addition result is partly leakage.** The +18.9 comes
  from evidence picked with sight of the reference answer, and it scales
  with lexical overlap. The clean finding is the null: mentioning a fact
  without the evidence does nothing.
- **Reasoning is not where the gap is.** Removing reasoning costs 3.7
  points. Tool outputs and narration carry most of what reports lose.
  That matters for designs that try to fix reporting by exposing chain of
  thought.
- **What this means for the hub's live fronts.**
  - *Redirect-before-halt proposal* (`docs/system-proposals/2026-10-18-iterate-redirect-before-halt.md`).
    Its "reopen a `next_candidate` an earlier cycle passed over" clause
    assumes passed-over options survive in the chain's record. Here
    rejected options and failed attempts are the categories the reports
    convey worst (automated labels), and a bare mention does not make
    them answerable. Combined with [[literature/papers/sun2026scipivot]]
    (the agent's narrative notes lower the pivot choice), the record a
    pivot cycle reads should be structured fields (rejected candidates,
    failed runs with their measured result), not prose. This is our
    inference; neither paper tests a chain.
  - *Evidence-gated completion.* The pytest case is a direct instance: a
    baseline run aborted after reverting files, the patch was restored,
    and the report described only the later successful results. A gate
    or reviewer reading the final message cannot see this.
  - *HCE and the noise band.* No direct bearing.
  - *This hub's own pipeline.* Orchestrators that read only subagent
    final replies (curation, digest.log lines) are report-only readers in
    this paper's sense. Load-bearing state should go into files the
    orchestrator can open, not only into the reply.

## Trust signals

- **Credibility:** 3.
  - For: Tencent (the only listed affiliation; one author uses a USTC
    email). A public repo (verified
    2026-10-20, created 2026-10-14) with atoms, probes, report and
    scrollback views, 228,939 frozen judgement records, prompts,
    `reproduce_all.py` for the headline numbers and `evaluate.py` for new
    models. An unusually thorough audit trail: human dual-rating, a
    mention/sufficiency audit that contradicts the authors' own automated
    label (and they report it), a paraphrase audit, a fully independent
    second measurement chain, and candid limitation statements throughout.
  - Against: not peer reviewed. One in-house model (HY4-preview, not
    expanded in the paper) fills almost every role. The cohort is
    non-random, with one trajectory per task. Decision relevance is
    model-labelled. Full execution context and reasoning are not released,
    so the T arm cannot be rerun. The supervision endpoint is an
    instruction change, not an outcome. The repo license is
    "NOASSERTION".
  - Not 4 because the instrument is model-built from the favoured view
    and relevance is never human-grounded. Not 2 because the release and
    the cross-chain replication are well above the median preprint here.

## Follow-up

- **Relevance:** 4. This is the first measurement in the graph of what an
  agent's report loses relative to its trajectory, with matched-length
  rewrite arms. It adds material evidence to
  [[concepts/evidence-gated-completion]] (the report is not the
  evidence; a mention is not evidence) and to
  [[concepts/context-eviction-policy]] (a natural-language summary as
  the only carried state keeps about half of what the context held, and
  a structured or category-targeted summary recovers more of the
  non-inferable part). Not 5 because the tasks are SWE-bench issue fixes,
  not ML-research loops, relevance is model-labelled, and no downstream
  outcome is measured.
- **Design note for `/iterate` and `/wrap`.** The held-out arm suggests a
  cheap rule: a cycle's log line or NOTES entry should carry a fixed set
  of fields for failures, facts, progress and environment (the
  categories the targeted arm picked from dev data). That beat natural
  prose on non-inferable state at the same budget (+7.7 points), though
  not on overall recovery. Not proposed here.
- **Related in the graph:** [[literature/papers/sun2026scipivot]] (agent
  notes anchor the reader), [[literature/papers/xu2026dont]] (the
  agent's log contaminates the verifier),
  [[literature/papers/nguyen2026cliffcompaction]] (a plausible summary
  discourages going back to the source),
  [[literature/papers/kim2026into]] (self-assessed completion misses
  required records), [[literature/papers/zhu2026claimreceipt]]
  (sufficiency vs coverage of retained evidence).
- **Candidates** (cited, not yet in the graph):
  - Kraishan and Jitkajornwanich 2026, "Plans they abandon, reports they
    author: The narrative layer of autonomous agents" (arXiv 2609.12205).
    5,851 coding-agent sessions; self-report vs log reconstruction.
  - Wang and Wang 2026, "The observability gap: Why output-level human
    feedback fails for LLM coding agents" (arXiv 2603.26942).
  - Tang et al. 2026, "How coding agents fail their users" (arXiv
    2605.29442). 20,574 real-world sessions.
  - Podivilov et al. 2026, AgentLens (arXiv 2607.06624).
