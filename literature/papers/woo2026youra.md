---
kind: paper
title: "YouRA: A Persistent-State Architecture for Evidence-Traceable Autonomous Research Agents"
authors: ["Yoonkyu Woo", "Woojin Lee", "Jin-xia Huang"]
institutions: ["Electronics and Telecommunications Research Institute (ETRI), Republic of Korea", "University of Science and Technology (UST), Department of Artificial Intelligence, Republic of Korea"]
year: 2026
venue: "arXiv 2610.01097v1 (cs.AI), 2026-10-01; ACL-style format (Limitations / Ethical Considerations sections), no venue stated"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.01097"
code_url: "https://github.com/PrayPrey/Your-Research-Agent"
citations: null
source: "raw/papers/woo2026youra.pdf"
added: "2026-10-06"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/structured-world-model]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/file-as-bus]]"
tags: ["end-to-end-research-agent", "mlr-bench", "persistent-state", "verification-state", "yaml-state-file", "independent-controller", "control-execution-separation", "stateful-reflection", "failure-memory", "repair-redesign-reset", "redirect-before-halt", "mock-data-detector", "data-provenance", "hallucination-diagnostic", "llm-as-judge", "ablation", "tool-confound", "claude-code-hooks"]
---

# YouRA: A Persistent-State Architecture for Evidence-Traceable Autonomous Research Agents

## TL;DR

An end-to-end ML-research agent built on Claude Code. Three parts:

- **VSA**: a single `verification_state.yaml` holding hypotheses,
  MUST_WORK / SHOULD_WORK gates, evidence pointers, a failed-hypothesis
  registry and a timestamped history.
- **Independent Controller**: GPT-5.2 in a separate context. It reads only a
  compact VSA summary plus the latest artifact pointer, and drives stage
  transitions through a Claude Code Stop hook.
- **Stateful Reflection**: failure records written as files, plus a
  three-level recovery ladder (bounded repair → hypothesis redesign →
  problem reset).

On MLR-Bench's ten-task end-to-end subset with three Claude backbones, the
four-LLM-judge Overall beats MLR-Agent by +0.85 and AI Scientist V2 by +0.64
(pooled over 30 cells, p = 0.006 and 0.044). Each of four ablations costs
about 0.9–1.1 Overall.

For this graph, the useful content is the **ablations**. They give a second
ML-research ablation, after AiScientist's File-as-Bus, showing that durable
external state beats an in-context summary (−1.13 Overall). They also give
the first ablation of a control loop held in a separate context (−1.12). The
cross-system win is confounded with tool access, and the headline's weakest
backbone gap is +0.18. In practice the recovery ladder is almost entirely
**reset**: 81 of 86 escalations.

## Claims

- **The claim–evidence gap is structural.** "research state, failure
  histories, and claim–evidence alignment are not maintained as persistent,
  verifiable state across long-horizon pipelines."
- **Control should not live in the execution dialogue.** In current agents
  "the same task-execution dialogue often both performs the work and decides
  how the workflow should continue", so stop/retry/redesign decisions drift
  as the dialogue grows.
- **The wins are not from better ideas.** On the full 201-task pool at the
  Idea/Proposal stages, YouRA is "lower or statistically indistinguishable"
  from MLR-Agent. The authors therefore place the end-to-end gain in
  execution, verification and writing.
- **Every component contributes.** "every ablation significantly lowers
  Overall under Holm-corrected paired sign-flip permutation tests", and the
  drops cannot be ranked against each other (every pairwise Holm p = 1.000).
- **The gain is not bought with fabrication.** YouRA "sits in the
  low-hallucination group yet alone leads on Overall".
- **The paper also takes an evaluation stance.** A research agent "should be
  judged on whether its final claims remain traceable to executed evidence …
  and a persuasive manuscript alone should not settle that judgment."
- **The components are said to be non-portable.** They "cannot be grafted
  onto existing research agents as standalone modules". This is asserted,
  not tested.

## Methods

- **Lifecycle.** Ten stages:
  - Problem Scoping (a Socratic brainstorm adapted from BMAD-Method);
  - Literature Grounding;
  - Hypothesis Debate: six personas, up to 20 rounds, with convergence judged
    by the controller;
  - Verification Planning: a DAG of 3–7 gated sub-hypotheses;
  - a Hypothesis Loop: Experiment Design → Implementation Planning (PRD,
    architecture, `03_tasks.yaml`) → Coding & Validation with spec-driven
    tests and an independent Validator;
  - Evidence Check: each prediction labelled SUPPORTED / PARTIALLY_SUPPORTED /
    REFUTED / INCONCLUSIVE;
  - Manuscript Drafting in three "story groups", with the abstract written
    last;
  - Adversarial Fact Review: three reviewer personas, one of which checks
    numbers against a ground-truth registry;
  - Manuscript Refinement, repeated "until no major findings remain".
- **VSA invariants.**
  - Gate-driven progression: a hypothesis advances only when its MUST_WORK
    gates pass.
  - Auditable history: every mutation is timestamped and attributed.
  - Steering as mutation: researcher directives become field updates.
- **Controller.** GPT-5.2 via OpenRouter. Its context does not grow with
  pipeline length. A Stop hook either approves termination, emits a resume
  prompt, or escalates MUST_STOP to a human. If a controller call fails, the
  backbone LLM stands in for that round.
- **Recovery (Algorithm 1).**
  - Local runtime, test or fabricated-output failures get bounded in-loop
    repair. A hook-based mock-data / hard-coded-result detector runs after
    each execution.
  - A SHOULD_WORK failure is retried or documented as a limitation.
  - A MUST_WORK failure that survives repair is classified by reflection.
    "Recoverable" goes to Hypothesis Debate redesign. "Evidence contradicts
    hypothesis" goes to Problem Scoping reset, which archives the folder and
    starts "a new research direction".
  - Failure records (`failure_{hid}_run{n}.md`) go to Serena memory and are
    consulted before new debate rounds.
- **Tool stack.** An MCP layer: Archon (RAG / task store), Exa, Semantic
  Scholar, Serena (code navigation and memory), Clear Thought. The baselines
  run in their released configurations instead. MLR-Agent drives Codex /
  Claude Code with no MCP stack; AI Scientist V2 has Semantic Scholar. The
  authors label this "a released-configuration comparison … not a fully
  tool-normalized comparison".
- **Evaluation.**
  - MLR-Bench end-to-end rubric (Clarity, Novelty, Soundness, Significance,
    Overall; 1–10) on the benchmark's predefined 10-task subset.
  - Backbones: Sonnet 4.5, Opus 4.5, Sonnet 4.6. One run per (system,
    backbone, task); seeds are explicitly not a reproducibility guarantee.
  - Judges: Gemini 3.1 Pro, GPT-5.4, Grok 4.3 and Opus 4.6, averaged. Each
    receives the paper, code, results and logs. Code, results and logs are
    truncated from the end when they exceed the judge's context.
  - Pairwise preferences are judged in both orders, and a reversal counts as
    a tie.
  - Statistics: paired sign-flip permutation over the 30 (backbone, task)
    cells, with Holm correction.
- **Diagnostics.**
  - Hallucination: MLR-Bench's four-category taxonomy with four judges. 270
    flagged instances were human-validated by three annotators plus an
    auditor.
  - Data provenance: two analyzers (Claude Code/Opus 4.6 and Codex/GPT-5.4)
    label each run Real / Synthetic / Fabricated. A run counts as real only
    if both agree.
- **Hardware.** One workstation (2× EPYC 9354, up to 5× H100 NVL).
  Generated code runs locally, usually on one GPU. Token budgets and cost are
  not reported.

## Results

- **Table 1, end-to-end Overall** (mean ± task SD, n = 10 tasks per cell):

  | Backbone | MLR-Agent | AI Scientist V2 | YouRA |
  |---|---|---|---|
  | Sonnet 4.5 | 3.10 ± 0.64 | 3.62 ± 0.84 | 4.20 ± 1.45 |
  | Opus 4.5 | 3.62 ± 0.80 | 4.28 ± 1.16 | 4.45 ± 1.51 |
  | Sonnet 4.6 | 4.42 ± 1.11 | 3.88 ± 0.88 | 5.05 ± 0.92 |

  - YouRA's Soundness is 3.98 / 4.58 / 4.92. MLR-Agent leads on Novelty
    (5.95 vs 5.70) and Clarity (7.78 vs 7.72) on Sonnet 4.6.
- **Table 12, paired tests on Overall.**

  | Comparison | Δ | Perm. p | Holm p | Cell W/L/T | Sign p | Per backbone (S4.5 / O4.5 / S4.6) |
  |---|---|---|---|---|---|---|
  | vs MLR-Agent | +0.85 | 0.006 | 0.012 | 18/12/0 | 0.362 | +1.10 / +0.83 / +0.63 |
  | vs AI Scientist V2 | +0.64 | 0.044 | 0.044 | 20/8/2 | 0.036 | +0.58 / +0.18 / +1.18 |

- **Table 2, pairwise preferences** (W/T/L out of 120 per baseline): 72 / 33 /
  15 against MLR-Agent and 55 / 38 / 27 against AI Scientist V2.
- **Tables 3 and 16, ablations** (Overall; Δ pooled over 30 cells):

  | Removed | S4.5 | O4.5 | S4.6 | Δ | Perm. p | W/L/T | Sign p |
  |---|---|---|---|---|---|---|---|
  | (full) | 4.20 | 4.45 | 5.05 | — | — | — | — |
  | MCP tools | 2.83 | 3.05 | 4.42 | −1.13 | <0.001 | 20/5/5 | 0.004 |
  | Reflection (redesign + reset) | 3.08 | 3.60 | 4.42 | −0.87 | 0.007 | 17/11/2 | 0.345 |
  | VSA (→ in-context summary) | 2.88 | 3.33 | 4.10 | −1.13 | 0.002 | 22/8/0 | 0.016 |
  | Independent Controller | 2.38 | 3.25 | 4.70 | −1.12 | 0.001 | 22/6/2 | 0.004 |

  - Drops are smallest on Sonnet 4.6. The authors read this as "a stronger
    execution backbone partially compensates for a missing component."
  - In judge justifications, w/o VSA shows numbers that disagree across
    sections, code that implements a different experiment from the paper,
    and selective reporting.
  - w/o Controller shows "failed or crashed runs whose outputs are still
    reported as completed findings, validation gates relaxed post hoc
    instead of escalated, and pipelines that shrink far below their plans".
- **Tables 4 and 5, hallucination.**
  - Flag precision is 203/270 = 75.2% (CI 69.7–80.0%) and does not differ by
    system (χ² p = 0.686). Auditor κ = 0.54.
  - Precision-corrected counts: YouRA 317, MLR-Agent 429, AI Scientist V2
    298.
  - YouRA − MLR-Agent = −112 (p = 0.003). YouRA − AI Scientist V2 = **+19**
    (CI −45 to +86, p = 0.571).
- **Table 15, provenance** (real-data runs, both analyzers): YouRA 27/30, AI
  Scientist V2 17/30, MLR-Agent 11/30. Every non-real cell is *Synthetic*
  except one MLR-Agent run labelled Fabricated.
- **Tables 9–11, routing telemetry.** There were 86 archive-producing
  escalations (28 / 40 / 18 by backbone; up to 11 in one task): **5
  redesign, 81 reset**, 0 unclassified. Bounded repairs are excluded by
  design, and counts come from each task's final working directory only.
- **Tables 13–14, upstream.** On Sonnet 4.5, MLR-Agent's Idea-stage Overall
  is significantly higher (Δ −0.35, Holm p = 0.004). On Opus 4.5 there is no
  difference.

## Critique / open questions

- **The digest pattern holds again (32/32).** The abstract says YouRA beats
  AI Scientist V2 "across all three matched backbones". That is true in sign
  only. The Opus 4.5 gap is **+0.18** against task SDs of 1.51 and 1.16 over
  10 single-run tasks, which is noise. The pooled AI Scientist V2 result sits
  exactly at the threshold (p = 0.044, Holm 0.044). Against MLR-Agent,
  YouRA wins only **18 of 30 cells** (sign p = 0.362). The permutation test
  is significant because of a few large wins, not consistent ones.
- **The cross-system gain cannot be credited to the persistent-state
  architecture.**
  - YouRA alone has the five-server MCP stack. Removing it costs −1.13, as
    much as removing the VSA or the controller.
  - Without MCP, YouRA (2.83 / 3.05) falls **below both baselines** on
    Sonnet 4.5 and Opus 4.5.
  - The headline also includes a second model family (the GPT-5.2
    controller) and unreported token budgets that the Limitations section
    admits differ.
  - Only the within-YouRA ablations isolate architecture, as the authors
    say. Those ablations are about a co-adapted pipeline: removing any one
    of four parts costs about the same (0.87–1.13) and pushes the system to
    baseline level. That fits "the pipeline was tuned with all four present"
    as well as it fits "each part is separately load-bearing".
- **The ablation headline is weaker than it reads.**
  - The abstract says removing a core-state component "drops YouRA below the
    full system". Any ablation that hurts does that.
  - The stronger body claim, "below both baselines", holds on 2 of 3
    backbones. On Sonnet 4.6, w/o Controller (4.70) still beats both
    baselines (4.42 and 3.88).
  - The Reflection ablation is the weakest: sign p = 0.345, 17/11/2. On Opus
    4.5 it ties MLR-Agent (3.60 vs 3.62).
  - The w/o Controller arm removes GPT-5.2 as well as the separation. It
    does not distinguish "a separate context" from "a second model family"
    or "more compute".
- **"Evidence-traceable" is not shown by the hallucination diagnostic against
  the strongest baseline.**
  - Corrected hallucination counts are nominally *higher* than AI Scientist
    V2's (317 vs 298, p = 0.571); the authors call this "absence of evidence
    for a difference".
  - Human validation measures precision on flagged instances only. Missed
    hallucinations are never counted.
  - So the claim–evidence alignment that names the paper beats MLR-Agent but
    is not measurably better than AI Scientist V2.
- **The provenance diagnostic measures real versus synthetic data, not
  fabrication.**
  - Only one of 90 runs was labelled Fabricated, and it was MLR-Agent's.
  - YouRA's pipeline contains a mock-data detector aimed at almost exactly
    the property this diagnostic scores, so 27/30 is partly
    design-to-the-metric.
  - Synthetic data is sometimes the legitimate choice for a task.
- **The recovery ladder is in practice "repair or reset".**
  - 94% of escalations restart Problem Scoping with "a new research
    direction"; only 5 of 86 go to redesign. The figure-4 example the paper
    shows is the rare case.
  - The judges score the final working directory. The paper never reports
    whether manuscripts disclose earlier refuted directions.
  - A loop that discards a failed hypothesis and re-scopes up to 11 times
    per task is structurally a **file drawer**. It may raise Soundness and
    Significance partly by selecting the direction that passed its gates.
    The Reflection ablation (−0.87, the weakest) is the only handle on this,
    and it cannot separate the two readings.
- **Absolute quality is low.** The best Overall is 5.05/10, and Soundness
  stays under 5 on every backbone. The architecture moves papers from
  "reject" toward "borderline" under an LLM rubric. It does not reach
  publishable work.
- **The evaluation itself is limited.**
  - One run per cell and 10 tasks; the paper uses MLR-Bench's predefined
    subset, so this was not a choice.
  - LLM judges whose configuration differs from MLR-Bench's
    human-aligned one; the authors flag that alignment "requires separate
    validation".
  - Tail-truncation of code and logs, which affects the systems that produce
    the most artifacts.
  - No cost per paper.
- **Several things count for it.**
  - The comparison does not overreach: the authors label it
    non-tool-normalized themselves.
  - The upstream control shows YouRA *worse* at ideation.
  - Holm-corrected paired tests are run on cells rather than on judge
    verdicts.
  - It includes human validation with an auditor, a two-analyzer provenance
    check, and released generated papers, VSA files and judge outputs.
  - The repo (created 2026-09-13, 1 star at ingest) has per-ablation
    directories, including a `YOURA_no_VSA_no_IC` arm the paper does not
    report.

## Trust signals

- **Credibility:** 3. It reaches 3 on a reputable group plus released
  artifacts:
  - a national research institute (ETRI), government-funded;
  - code, prompts, generated papers, VSA state files and raw judge outputs
    released;
  - careful paired statistics with Holm correction;
  - human validation of the automated diagnostic.

  It is held at 3 for these reasons:
  - not peer-reviewed;
  - single runs on 10 tasks with LLM-only quality judging;
  - a cross-system comparison confounded by tools, a second model and
    budget;
  - an abstract that rounds borderline and sign-only results up to
    "across all three".

## Follow-up

- **Relevance:** 4. It is an end-to-end ML-research agent with clean
  single-component ablations of the two architectural ideas this graph cares
  about most:
  - durable structured state in place of an in-context summary
    ([[concepts/structured-world-model]], [[concepts/file-as-bus]]);
  - control held in a separate context that reads only state, not the trace
    ([[concepts/hierarchical-delegation]]).

  It also gives a measured base rate for repair / redesign / reset routing
  ([[concepts/budget-as-ceiling]]'s redirect-before-halt question). It is
  not a 5 because the evidence is LLM-judged, single-run and n = 10, and
  because the system-level win is confounded.
- **Bearing on redirect-before-halt for `/iterate`.**
  - The w/o Reflection arm keeps running with failures recorded but takes
    no redesign or reset. It costs −0.87, a signal that is not robust (sign
    p = 0.345).
  - It is still not redirect-vs-halt: neither arm halts.
  - The ladder YouRA actually executes is almost all reset. A redirect rule
    for `/iterate` should log which rung fired and keep the abandoned branch
    visible in the write-up, or it inherits the file-drawer problem above.
- **Bearing on [[concepts/evidence-gated-completion]].** The w/o Controller
  failure profile is the clearest qualitative evidence in the graph so far:
  "gates relaxed post hoc instead of escalated" and crashed runs reported as
  findings. It shows what happens when the agent that does the work also
  decides whether its gate passed. This is judge-justification text, not a
  counted rate.
- **[[concepts/typed-claim-partition]], unmeasured.** Evidence Check's
  four-way per-prediction label is a typed partition that feeds the writer
  and the reviewer. It is not ablated separately, so the paper offers it as
  a design instance only.
- **The Claude Code mechanics transfer directly to this box.** The
  controller's verdict enters through a **Stop hook** that approves, resumes
  or escalates MUST_STOP. A YAML state file is the hook's input. That is a
  concrete pattern for moving a keep/stop decision out of the working
  session.
- **Candidates:**
  - Chen et al. 2026, MLR-Bench (NeurIPS 2026 D&B). The benchmark and its
    "~80% fabricated or unverified results" finding are not yet ingested.
  - Liu & Zhu 2026, "Agent-native research artifacts".
  - Baumgärtner & Gurevych 2026, SciCoQA (ACL 2026): paper–code
    discrepancy detection below 50%.
  - Krumdick et al. 2026, "No free labels" (COLM 2026).
  - Yan, Chen & Zhang 2026, "When the specification emerges" (arXiv
    2603.17104).
