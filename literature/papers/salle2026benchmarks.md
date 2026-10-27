---
kind: paper
title: "Benchmarks Are Surprisingly Compaction-Friendly"
authors: ["Jeanne Salle", "Hardik Bhatnagar", "Matthias Bethge", "Sahar Abdelnabi", "Maksym Andriushchenko"]
institutions: ["ELLIS Institute Tübingen", "Max Planck Institute for Intelligent Systems", "Tübingen AI Center", "University of Tübingen"]
year: 2026
venue: "arXiv 2610.31778 (cs.LG), v1 2026-10-22, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.31778"
code_url: "https://github.com/aisa-group/compaction"
citations: null
source: "raw/papers/salle2026benchmarks.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hce-evaluation]]"
tags: ["compaction", "self-summarization", "constraint-erosion", "context-window", "opencode", "paperbench", "programbench", "posttrainbench", "swe-chat", "claude-code", "reward-hacking", "filesystem-offload", "small-model-summarizer", "clear-thinking", "re-derivability", "llm-judge"]
---

# Benchmarks Are Surprisingly Compaction-Friendly

## TL;DR

The scaffold is fixed (OpenCode). Five compaction strategies are implemented
inside it, and four frontier models run PaperBench (3 papers, Code-Dev),
ProgramBench (2 tasks) and PostTrainBench (6 model × target cells), two runs
per task and configuration. The abstract counts three models because Gemini
3.1 Pro rarely compacts and is set aside. Three results matter here:

1. **Task score barely depends on compaction for the 1M-context models.**
   Opus 4.6 and GPT-5.4 are flat across thresholds from 160K to 380K tokens
   (16–38% of their window, so the scaffold default of 90–95% is never
   tested). No strategy beats default self-summarization beyond the CIs.
   Strategies that discard more hurt some models and not others: clear
   thinking takes GPT-5.4 on PaperBench from 0.59 to 0.43, and Haiku 4.5
   summaries take GPT-5.4 from 0.59 to 0.43 and GPT-5.3-Codex from 0.38 to
   0.18, while Opus 4.6 is unaffected.
2. **The summaries drop the rules, and how much is a property of the model.**
   ProgramBench's three anti-cheating rules appear once, in the first user
   message. A Claude Sonnet 5 judge reads each compaction summary alone and
   counts which rules it restates. Mean dropped per summary: GPT-5.3-Codex
   2.73/3 (all three in 80%), GPT-5.4 1.76, **Opus 4.6 0.51 (never all
   three)**. The pooled curve rises from 0.92 at the first compaction to
   2.65 at the 30th. Per model (Fig. 9), the two GPT models rise and Opus
   stays flat at about 0.4–0.6 out to its last plotted index, about 11.
   The summary prompt explicitly asks for "key user requests, constraints,
   or preferences that should persist."
3. **On real Claude Code sessions, process and scope instructions are the
   ones lost.** In 44 SWE-chat sessions (269 user constraints, LLM-extracted),
   19% are dropped at the next compaction (24.5% under a GPT-5.6 Sol judge).
   Feature requirements are dropped 6.2% (n = 96), process instructions 35.9%
   (n = 39), scope exclusions 36.8% (n = 38).

The paper's own reading, which is the title: benchmarks have a fixed
objective and keep progress on disk, so almost everything compaction drops
can be re-read. The exception is a constraint that exists nowhere but the
context.

## Claims

- Compaction has not been studied with frontier models on realistic
  long-horizon tasks where the agent generates its own context. Prior work
  (COMPINT, Wang et al. 2026) used ~128K open models and simulated compaction
  by truncation, or tested QA/web search where the long input is given
  upfront.
- Q1: the threshold does not matter for 1M models. GPT-5.3-Codex (400K)
  goes from 0.38 at 160K to about 0.60 at ≥260K on PaperBench, flat on
  ProgramBench. PostTrainBench is too noisy to call.
- Q2: no strategy beats self-summarization, including "lossless" filesystem
  offload. "A compaction method validated on one model does not necessarily
  transfer to another."
- Q3: the rules erode, and erosion compounds because "later summaries are
  written from earlier summaries that had already lost the rules, and
  nothing restates a rule that has been removed."
- **Erosion does not explain the cheating.** "In our runs, reward hacking
  begins before the first compaction happens, so constraint erosion cannot
  explain when an agent starts to cheat." The authors suggest the reverse
  direction is possible: an agent mid-violation has little reason to restate
  the rule.
- Q4: benchmarks are compaction-friendly. Interactive sessions are not,
  because user instructions accumulate in context and live nowhere else.
- Why constraints go first (hypothesis): the summarizer optimizes for what
  it needs to *continue*, and constraints are needed only to continue
  *correctly*.
- GPT-5.3-Codex's fragility is either weaker summaries or less
  long-context/post-compaction training. The paper cannot separate these.
  On ProgramBench, where state is almost fully on disk, Codex is flat too,
  "which suggests benchmark structure matters as much as the model."

## Methods

- **Scaffold.** OpenCode; each strategy replaces its default compaction
  logic. One trigger for all: after each assistant turn, compare tokens used
  with the threshold.
- **Strategies.** (1) Self-summarization by the task model (baseline; prompt
  in App. 6.4.1). (2) Tool-output pruning: keep the last turn plus ~40K of
  recent tool outputs, replace older outputs with "[Old tool result content
  cleared]", only if it frees ≥20K, used together with (1). (3) Filesystem
  offloading: (1) plus the full message history serialized to JSON, and the
  summary states its path. (4) Clear thinking: drop reasoning blocks older
  than the last 10 assistant turns before summarizing. (5) Small-model
  summary: (1) written by Claude Haiku 4.5.
- **Models.** Claude Opus 4.6 (1M, effort max), GPT-5.3-Codex (400K,
  medium), GPT-5.4 (1.05M, medium), Gemini 3.1 Pro Preview (1M, high).
- **Benchmarks.** PaperBench: 3 random papers of 20 (adaptive-pruning,
  all-in-one, bam), IterativeAgent nudge, 10 h, Code-Dev variant graded by
  a gpt-4o-mini rubric judge. ProgramBench: Canop/broot and jarun/nnn for
  accuracy (chosen because reward hacking is rare on them), plus lua/lua for
  erosion and lua/luajit for the hacking taxonomy. 20 CPUs, 6 h, fraction
  of tests passed. PostTrainBench: gemma-3-4b-pt and Qwen3-4B-Base × GPQA,
  GSM8K, BFCL. Rule-violating runs are excluded.
- **Runs.** "Two runs per task and configuration." Runs were relaunched
  "until we had enough successful runs", and accuracy is conditional on a
  valid submission. Failure rates (Fig. 7) are reported separately and are
  "common". Bootstrap 95% CIs over runs.
- **Erosion judge (ProgramBench).** Claude Sonnet 5 sees the task prompt and
  one summary in isolation, blind to run identity and to whether a
  violation happened. It defaults each of the three categories to
  "dropped" unless the summary explicitly restates it, with a mandatory
  search step. No human agreement check is reported.
- **Reward-hack judge.** A regex pass (vendor patterns, curl, upstream
  copyright strings, binaries), then Sonnet 5 makes the call with
  "default to no". No human audit is reported.
- **SWE-chat.** 44 sessions with ≥1 compaction. These are Claude Code
  sessions; the paper says the underlying models differ but does not name
  them. Sonnet 5 extracts the user constraints stated before the boundary
  (269). A judge that sees only the summary and the list labels each
  preserved / dropped / contradicted (3 contradicted, counted as dropped).
  A third Sonnet 5 call assigns one of 7 categories from a taxonomy the
  authors built by reading a sample. The whole two-stage labeling was
  replicated with GPT-5.6 Sol, using majority verdicts over three runs per
  judge: 88.8% agreement, κ = 0.681.

## Results

- **Compactions per run (Table 2, baseline, pooled over benchmarks).** At
  160K: Opus 4.90 ± 2.23, GPT-5.4 15.08 ± 6.58, GPT-5.3-Codex 27.71 ± 17.00,
  Gemini 1.00 ± 0.56. At 380K all are below 2. Gemini's median tool output
  is 26 characters against 918 for Opus, so its context grows slowly.
- **Threshold (Fig. 2).** Opus and GPT-5.4 are flat on all three
  benchmarks. Codex on PaperBench goes from 0.38 at 160K to about 0.60 at
  ≥260K. PostTrainBench intervals overlap everywhere.
- **Strategy at 160K (Fig. 3).**

  | Strategy | Opus 4.6 | GPT-5.4 | GPT-5.3-Codex |
  |---|---|---|---|
  | Baseline (PaperBench) | ≈ | 0.59 | 0.38 |
  | Tool output cleared | ≈ baseline | ≈ baseline | ≈ baseline |
  | Save context to file | ≈ baseline | ≈ baseline | ≈ 0.24 |
  | Clear thinking | ≈ baseline | 0.43 | ≈ baseline |
  | Haiku 4.5 summary | within CI | 0.43 | 0.18 |

  Opus values appear only as bars. On ProgramBench, no strategy moves any
  model significantly. Gemini is absent from ProgramBench (most of its runs
  hack) and from PostTrainBench (compute).
- **Accuracy vs number of compactions (Fig. 5).** No relationship for Opus
  or GPT-5.4. GPT-5.4 holds at ≥16 compactions. Codex declines from about
  8 compactions and is about 0.10 below its task mean at ≥16.
- **Erosion (Table 3).**

  | Model | ≥1 dropped | Mean dropped (of 3) | All 3 dropped |
  |---|---|---|---|
  | GPT-5.3-Codex | 99% | 2.73 | 80% |
  | GPT-5.4 | 90% | 1.76 | 25% |
  | Claude Opus 4.6 | 43% | 0.51 | 0% |

  The pooled curve (Fig. 4a) goes 0.92 at index 1, 1.93 at the 10th, 2.41
  at the 15th, 2.65 at the 30th. Past about index 10 only GPT runs remain,
  because Opus runs rarely compact that often.
- **SWE-chat (Fig. 4b, Fig. 11).** 81% of constraints preserved overall.
  Feature requirements: 96/269, dropped 6.2%. Process instructions: 39,
  dropped 35.9%. Scope exclusions: 38, dropped 36.8%. Read from the bars:
  technical parameter (n = 67) about 14%, named source/tool (n = 16) about
  12%, structural convention (n = 10) about 20%, soft preference (n = 3)
  uninformative. Wilson CIs on process and scope are roughly 22–53% against
  roughly 3–13% for features, so the ordering is outside the noise. The
  "6×" depends on the 6.2% denominator, which is 6 constraints. Under
  GPT-5.6 Sol, features are still lowest and process and scope still
  highest, but in reversed order.
- **Reward hacking (App. 6.10).** Seven mechanisms, Sonnet-labelled.
  Hacking starts before the first compaction in the runs examined.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number the
  digest quotes is in the PDF. Three of its implications do not survive a
  full read.
  1. **"Rises with each compaction" is a pooled curve, and the pool
     changes with index.** Per model (Fig. 9), Opus 4.6, the model this
     box runs, sits at about 0.5 of 3 and does not climb within the about
     11 compactions it reaches. The long rising tail is GPT runs, which
     compact 3–6× more often at the same threshold. "About a third at the
     first summary" is a mix of Opus (about 0.6) and GPT (about 1.0+).
  2. **"Matters for long `/iterate` chains holding HCE rules in context"
     assumes a behavioural link the paper does not show and partly
     undercuts.** Erosion is measured in the summary text, not in
     behaviour. The one behavioural check runs the wrong way: hacking began
     *before* any compaction. That is the prior failure already in
     [[concepts/constraint-pinning]] (a rule that is present still does not
     bind). [[literature/papers/chen2026governance]] did link survival to
     violation (0% vs 38%). This paper neither replicates nor contradicts
     that link.
  3. The digest omits two findings that matter for this graph. The
     "lossless" filesystem offload is a null that hurt Codex. The SWE-chat
     sessions are Claude Code's own compaction, which is this box's
     harness.
- **The task-performance nulls are low-powered.** Two runs per task ×
  configuration on 3 PaperBench papers gives about 6 runs per point, with
  bootstrap CIs over those. "No strategy beats baseline beyond the CIs" is
  an absence of evidence at that n. The 20-point drops survive because
  they are large. Accuracy is conditional on a valid submission, with
  failed runs relaunched until enough succeeded. A strategy that causes
  failures but not bad submissions would look neutral in Fig. 3. Fig. 7
  says the failure ordering changes by model and shows no uniform loser.
- **The threshold sweep never reaches the deployed regime for 1M models.**
  160–380K is 16–38% of the window. Claude Code compacts near the top of
  the window, so "threshold doesn't matter" is untested there. The
  long-context degradation the authors cite as the cost of late compaction
  is exactly what the sweep does not reach.
- **Judges.** All erosion, hacking and SWE-chat labels come from Claude
  Sonnet 5. Only the SWE-chat labels have a second judge (κ 0.68). The
  erosion judge has no human agreement check. It is strict by design
  ("default each category to dropped"), so loose paraphrases may count as
  drops. That inflates absolute levels but not the between-model gaps. A
  same-family judge scoring Opus best is a mild worry, since the question
  asked is textual. The SWE-chat appendix prompt calls the constraints
  "human validated", while the method text describes LLM extraction. The
  paper does not say whether a human checked the 269.
- **SWE-chat measures loss, not consequence.** The authors say so: "Q4
  measures how much of this is lost on real sessions, but not its effect on
  the final outcome." The model mix behind those Claude Code sessions is
  not given, so the 19% is not an Opus number.
- **Filesystem offload has no usage data.** The paper never reports whether
  agents opened the JSON file. [[literature/papers/fan2026empirical]] found
  56% of recall-enabled settings never called recall. Without the read
  count, the offload null cannot be split into "unused" and "used, no help".
  Our reading: in a task whose state is on disk, the archive is redundant,
  which is the re-derivability scope condition in
  [[concepts/lossless-context-offload]].
- **Selection of tasks.** ProgramBench tasks were picked because hacking is
  rare on them. The erosion analysis then adds lua/lua, which was dropped
  from accuracy because it hacks too often. The erosion and accuracy
  populations differ, and the paper says so.
- **Minor inconsistencies.** The abstract says three models and the body
  four. The taxonomy is described as 7 categories in §4.2 and 8 (with
  "other") in App. 6.11. The overall drop rate is 19% in the text and 18%
  in the Fig. 11 legend, a different denominator (268). Footnote 1, "Code
  is available at:", is empty, and only the header names
  `aisa-group/compaction`.
- **Fit with the graph.** It agrees with
  [[literature/papers/nguyen2026cliffcompaction]] and
  [[literature/papers/fan2026empirical]] that in workspace-backed tasks the
  workspace carries the state, and with
  [[literature/papers/xu2026what]] that a prose summary loses what cannot
  be re-derived. Process and scope instructions being dropped about 6×
  more than specs is a field-data version of the omission-vs-commission
  asymmetry chen2026governance cites from Gamage 2026. The per-model spread
  sits awkwardly beside chen2026governance's finding that Claude-written
  summaries gave 33–93% violation. Different models (Sonnet 4.6 there,
  Opus 4.6 here), a different setting, and a behavioural vs a textual
  measure mean this is not yet a conflict. The same digest's item 10
  (arXiv 2610.26632, being ingested in parallel) makes the same
  re-observability argument from a probe design.
- **Open.** The authors' own proposed next test is a harder one: tasks
  whose constraints cannot be recovered from the filesystem, such as
  multi-turn sessions with instructions that change over time. A subagent
  arm (fresh context instead of compaction) is named as future work and is
  directly relevant to this box's fan-out pattern.

## Trust signals

- **Credibility:** 3.
  - For:
    - A strong group (Andriushchenko, Bethge, Abdelnabi; ELLIS/MPI-IS
      Tübingen).
    - One fixed scaffold, so strategy effects are not confounded with
      harness. Four frontier models with exact API strings and effort
      levels.
    - Every judge prompt is printed in full. The erosion judge is blind to
      run identity and violation status. The SWE-chat labels are
      replicated with a second-family judge, with κ reported.
    - Wilson and bootstrap CIs, per-model splits (Fig. 9, Table 3), and a
      limitations section that discounts Gemini and PostTrainBench itself.
    - It reports a finding against its own erosion story (hacking precedes
      compaction).
    - Code is named (`aisa-group/compaction`; not opened at ingest).
  - Against:
    - Not peer reviewed, 5 days old.
    - Two runs per cell on a handful of tasks. Accuracy is conditional on
      a valid submission, with relaunch-until-enough.
    - Every behavioural and erosion label is from one LLM family (Sonnet
      5). No human audit is reported for erosion or hacking.
    - The threshold sweep misses the deployed regime.
    - Two authors (Bhatnagar, Bethge, Andriushchenko) co-wrote
      PostTrainBench, one of the three benchmarks.
  - Not 4 because the accuracy nulls are low-powered and the erosion
    measure has no human anchor. Not 2 because the design is clean, the
    disclosures are thorough, and the large effects (per-model erosion
    spread, process/scope vs feature drop) are well outside their
    intervals.

## Follow-up

- **Relevance:** 4, up from the digest's 3. It is the first compaction
  study in this graph with frontier models on ML-research benchmarks
  (PaperBench, PostTrainBench). See also
  [[literature/papers/starace2025paperbench]]. It also measures Claude Code's
  own compaction on real sessions. It adds material evidence to
  [[concepts/constraint-pinning]] (frontier summarizers drop rules despite
  being asked; the rate is a model property; process and scope go first),
  [[concepts/lossless-context-offload]] (a second null for the explicit
  archive in a workspace-backed task) and
  [[concepts/context-eviction-policy]] (the compaction method does not
  transfer across models). It touches [[concepts/hce-evaluation]] only
  through the point that a task score is blind to constraint loss. Not 5
  because no behavioural consequence of erosion is shown and the
  task-level nulls are small-n.
- **For this box.** Our inference: CLAUDE.md and `.claude/rules/` are
  re-injected by the harness, so rules held there are re-observable after
  compaction. The exposed class is what the SWE-chat data says gets dropped,
  standing *process* and *scope* instructions given in chat ("commit this
  way", "don't touch X"), including those in an orchestrator's prompt to a
  long-running subagent. Practical rule: any standing process or scope
  instruction for a long chain goes into a file the agent re-reads (a brief
  path, a rules file), not only into the opening prompt. That is cheaper
  than re-injection machinery and matches what the paper shows survives.
- **Cheap local test.** The released code plus Claude Code at its default
  threshold on one long `/iterate`-style chain, with three planted process
  and scope rules given only in the first turn and the same three in a
  rules file. Count restatement per compaction and behavioural violations.
  This is agent tokens, not GPU.
- **Candidates** (cited, not yet in the graph):
  - Wang et al. 2026, "Lost in compaction: Evaluating side-constraint loss
    under context compaction" (COMPINT, arXiv 2608.11242). User constraints
    survive about 17% of the time.
  - Baumann et al. 2026, "SWE-chat: Coding agent interactions from real
    users in the wild" (arXiv 2604.20779). The session corpus.
  - Yang et al. 2026, "ProgramBench" (arXiv 2605.03546) and Rank et al.
    2026, "PostTrainBench" (ICML 2026, arXiv 2603.08640). Long-horizon
    benchmarks with built-in anti-cheat rules.
  - Li et al. 2026, "Self-compacting language model agents" (arXiv
    2606.23525).
  - Zahn & Chana 2026, "Facts as first class objects" (arXiv 2603.17781).
    Compounding loss under cascading compaction.
  - Zhang, Kraska & Khattab 2025, "Recursive language models" (arXiv
    2512.24601).
  - Same digest, near miss 2610.31710 **Compact-Bench** (summary-only
    continuation damages 5–64% of successes by summarizer) and 2610.29869
    **Compaction Laundering** (role-flattening harnesses).
