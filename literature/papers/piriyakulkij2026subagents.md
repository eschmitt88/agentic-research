---
kind: paper
title: "Subagents vs Agent Skills: Executing Reusable Knowledge for Long-Horizon Agentic Tasks"
authors: ["Wasu Top Piriyakulkij", "Rachel Lawrence", "Alicia Curth", "Sushrut Karmalkar", "Niranjani Prasad"]
institutions: ["Cornell University", "Microsoft Research Cambridge"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.09233"
code_url: null
citations: null
source: "raw/papers/piriyakulkij2026subagents.pdf"
added: "2026-09-14"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/lossless-context-offload]]"
tags: ["skills", "subagents", "context-isolation", "delegation", "io-contracts", "skillsbench", "peak-context", "lazy-loading"]
---

# Subagents vs Agent Skills: Executing Reusable Knowledge for Long-Horizon Agentic Tasks

## TL;DR

Keeps the skill package (`SKILL.md` + resources) fixed and changes only **how it
runs**: loaded into the main agent's context ("agent skill"), or run as a
subagent in a fresh context seeded with the skill body that returns only its
final message. On SkillsBench (64 of 87 tasks, OpenHands, 7 models), subagent
execution wins by a lot for weaker models **only** with the authors' synthesized
skills that state an expected input and output. Examples: Qwen3.5-9B 0.22 → 0.46,
Mistral-Large-3.1 0.26 → 0.54, Gemma-4-12B 0.17 → 0.40. For frontier models it
is a wash (gpt-5.3-codex 0.85 vs 0.85; Kimi-K2.6 0.80 vs 0.77) at about
**1.3–1.7× the total tokens**. Accuracy values here are read off bar charts
(±0.01); the paper gives no numeric tables.

## Claims

- What hurts long tasks is the **peak** context length of any single window, not
  the total tokens used. Subagents trade more total tokens for a lower peak.
- A skill package suits subagent execution when its description carries an
  **input contract and an output contract** (`d = (q_in, h, q_out)`) and its
  body is procedural. The paper frames this with the options framework: `q_in`
  plays the initiation set, the `SKILL.md` body the option policy, `q_out` the
  termination condition.
- Curated skills without contracts favour agent-skill execution. Skills with
  contracts favour subagents, and the gain is largest for small,
  "bandwidth-limited" models.
- Subagent execution degrades more gracefully as unrelated skills pile into the
  context.
- In a hierarchical library, **routing nodes should run inline and leaf skills
  as subagents** (the "hybrid" mode), which the authors compare to MCP
  lazy-loading.
- Existing harnesses (Claude Code, Codex) mostly use subagents for
  parallelism, not for encapsulation. Their citation for this is Anthropic's
  own subagent documentation.

## Methods

- **Formalism:** an agent skill returns `m_k` into the main context, so the
  main policy runs each skill step. A subagent is `π^(k)(·) = π_LLM(m_k ⊕ ·)`
  with a fresh context `c_0 = x_t` (the arguments the main agent passes), and it
  returns only its last response `r_T`. Both run on the same base LLM.
- **Benchmark:** SkillsBench, 87 long-horizon tasks with human-curated skill
  packages. Harness: OpenHands SDK in each task's Docker container, all LLM
  calls routed through a logging proxy. A minimal system prompt replaces the
  OpenHands default and tells the agent to prefer tools. Native OpenHands
  skills and auto-retrieval are disabled. Subagents are exposed as MCP tools
  with a 2-hour timeout.
- **Synthesized skill set:** GPT-5.3-codex ran every task 3 times. Copilot CLI
  (Claude Opus 5 as orchestrator, GPT-5.5 subagents) turned the **successful
  trajectories** into 3–5 packages per task. A human hand-fixed the first
  task's set (edit-pdf) so every description has "Expected input:" and
  "Output:" sections, and that set served as the template for the rest. This
  worked for **64/87 tasks**, and all results are on that subset.
- **Subagent prompt:** TASK + INPUT INFORMATION + KNOWLEDGE (the skill body) +
  the resource dir. The output must be stated in the final message, as an exact
  path if it is a file.
- **Models:** Ministral-8B, Gemma-4-12B, Qwen3.5-9B, Mistral-Large-3.1,
  gpt-5.4-mini, Kimi-K2.6, gpt-5.3-codex. Open weights served with vLLM on
  A100s, the rest through an API.
- **Conditions:** 5 in total: no skill, and {agent skill, subagent} × {curated
  no-contract, synthesized with contracts}. Plus a distractor sweep (0 / 100 /
  200 / 263 unrelated skills), peak context and total tokens, skill-call
  counts, and a library-organization study on Qwen3.5-9B only (appendix A.1).

## Results

All accuracy, token and call-count values in this section are read off
the paper's bar/line charts (≈±0.01 for accuracies) and are approximate;
only the peak-context percentages (printed as labels in Fig. 4) and the
263-distractor count are stated as exact numbers.

- **Main comparison (Fig. 2), accuracy as agent skill / subagent (≈, read
  from chart):**

  | Model | No skill | Curated (no I/O) | Synthesized (I/O contracts) |
  |---|---|---|---|
  | Ministral-8B | 0.02 | 0.00 / 0.01 | 0.03 / **0.10** |
  | Gemma-4-12B | 0.05 | 0.06 / 0.09 | 0.17 / **0.40** |
  | Qwen3.5-9B | 0.07 | 0.09 / 0.11 | 0.22 / **0.46** |
  | Mistral-Large-3.1 | 0.08 | 0.12 / 0.16 | 0.26 / **0.54** |
  | gpt-5.4-mini | 0.46 | **0.55** / 0.51 | 0.55 / **0.71** |
  | Kimi-K2.6 | 0.44 | 0.52 / 0.53 | **0.80** / 0.77 |
  | gpt-5.3-codex | 0.54 | **0.71** / 0.61 | 0.85 / 0.85 |

- **Distractor sweep (Fig. 3/6):** agent-skill accuracy falls steeply as
  distractors grow: Qwen 0.48 → 0.22, Mistral-Large 0.63 → 0.26, gpt-5.4-mini
  0.80 → 0.55. Subagent accuracy falls less: Qwen 0.62 → 0.46, Mistral-Large
  0.66 → 0.55, gpt-5.4-mini 0.82 → 0.71. gpt-5.3-codex stays flat at ~0.85 in
  both modes.
- **Peak context (Fig. 4 left):** share of tasks where subagent mode has the
  lower peak: gpt-5.3-codex 95.3%, Kimi 79.7%, gpt-5.4-mini 70.3%, Qwen 59.4%,
  Gemma 45.2%, Mistral-Large 32.8%, Ministral 28.1%. The authors put the low
  weak-model numbers down to agent-skill runs quitting early, so their contexts
  stay short.
- **Total tokens per task (Fig. 4 right):** subagent mode is always higher.
  gpt-5.3-codex goes from ~100K to ~172K (≈1.7×) and Kimi from ~122K to ~175K
  (≈1.4×). Weaker models mostly reach ~175–190K in subagent mode.
- **Skill calls (Fig. 7):** subagent mode makes more calls for every model.
  gpt-5.4-mini goes from 0.7 to 2.5 per task and Mistral-Large from 2.0 to 3.8.
  The authors read agent-skill mode as "underthinking" as the context grows.
- **Library organization (Fig. 5, Qwen3.5-9B only), accuracy as agent skill /
  fully subagent / hybrid:**
  - flat: 0.22 / 0.45 / 0.45
  - LLM tree h2: 0.45 / 0.03 / 0.41
  - tree h3: 0.28 / 0.06 / 0.32
  - LLM graph h2: 0.49 / 0.04 / 0.61
  - graph h3: 0.43 / 0.06 / 0.45
  - two-level task tree (no LLM): 0.50 / 0.07 / **0.64**

  **Fully-subagent mode collapses to 3–7% as soon as routing nodes are
  delegated.** Deeper hierarchies (h3) are worse than h2.

## Critique / open questions

- **The headline figure appears to be the maximum-distractor condition, and
  the paper never says so.** Each model's Fig. 2 bars match its 263-distractor
  endpoint in Fig. 6 to within reading error (Qwen 0.22/0.46, Gemma 0.17/0.40,
  Mistral-Large 0.26/0.55, Ministral 0.03/0.10). The likely explanation is that
  the "flat library" exposes the whole synthesized library for all 64 tasks.
  With **0 distractors** (only the task's own skills), the subagent advantage
  is much smaller: gpt-5.4-mini +0.02, Mistral-Large +0.03, gpt-5.3-codex
  −0.02, Gemma +0.10, Qwen +0.14, Ministral +0.21. So subagents help most under
  heavy context pressure, mainly for small models. That is narrower than the
  abstract's framing. (This reading is the note-writer's inference from the
  figures.)
- **The paper's central causal claim, that I/O contracts are what make
  subagents work, is not isolated.** The curated and synthesized sets differ in
  content, origin and interface at the same time. The authors admit the sets
  are "not a controlled comparison", but only when discussing overall quality.
  No ablation removes the contract sections from the synthesized skills, or
  adds contracts to the curated ones. On frontier models the synthesized
  content lifts both modes equally (codex 0.71 → 0.85 inline), which points to
  content as the main driver.
- **The paper's text misdescribes its own figure.** It says that with curated
  skills "agent skills match or outperform subagents across all models". In
  Fig. 2, subagents are slightly *ahead* with curated skills for Gemma, Qwen and
  Mistral-Large (0.09 vs 0.06, 0.11 vs 0.09, 0.16 vs 0.12). Agent skills lead
  clearly only for gpt-5.4-mini and codex. With synthesized skills, Kimi
  *reverses* (0.80 vs 0.77) rather than "subagents outperform".
- **The skills are distilled from successful runs on the evaluation tasks
  themselves, and the 64-task subset is chosen by where GPT-5.3-codex
  succeeded.** Synthesized skills are therefore near-oracle procedures for
  exactly the tasks being scored. This is the holdout gap
  [[concepts/skill-library-lifecycle]] already flags ("skill promotion needs a
  holdout too"), and it is especially sharp for gpt-5.3-codex, whose own
  trajectories seeded the library.
- **The hierarchy study is thin: one model, and the best structure is close
  to an oracle.** The winning two-level "task tree" groups skills by their
  source task, which for a novel task would not exist. Its accuracies (0.50
  inline, 0.64 hybrid) match the zero-distractor Qwen numbers (0.48, 0.62), so
  "hierarchy" here mostly recovers perfect scoping. The prose also says
  hierarchy "gives additional gains", yet tree-h2 hybrid (0.41) and tree-h3
  (0.32) score *below* flat (0.45).
- **The fully-subagent collapse (3–7%) is unexplained.** It is either a real
  finding (delegating a node with no procedure fails badly) or an artifact of
  how routing subagents were prompted. The paper does not diagnose it.
- **Error bars are shown but the number of runs is never stated.** There are
  no significance tests. Token and peak-context numbers exist only as charts.
  Some sloppiness: "We possess that", "Figure 3 (right)" for a single-panel
  figure, and "distracting tools" in the text vs "distracting skills" in the
  plots.
- **Claude is not among the executors**, although Opus 5 orchestrated the
  skill synthesis. The frontier-model evidence is gpt-5.3-codex and Kimi, and
  for both the accuracy benefit is nil.
- **This is not evidence for [[concepts/lossless-context-offload]]; it is its
  opposite.** A subagent's internal trajectory is thrown away and only `r_T`
  comes back. The one addressable element is the prompt rule "if the output is
  a file, state its exact path", so the subagent's *artifact* stays reachable
  but its *reasoning* does not. Whether the main agent ever needs to go back
  into a subagent's trajectory is not studied.

## Trust signals

- **Credibility:** 3. Microsoft Research Cambridge plus a Cornell intern (a
  reputable group), 7 models including two frontier ones, error bars, and an
  internally consistent set of experiments across figures. Held down by:
  arXiv preprint with no peer review, no code or skill set released, no
  citations yet, results only as charts with the run count unstated, a key
  confound (contract vs content) not ablated, skills distilled from the
  evaluation tasks themselves, and text that misdescribes the paper's own Fig.
  2 in two places.

## Follow-up

- **Relevance:** 4. It is the first source in the graph to hold skill content
  fixed and vary only **inline vs fresh-context execution**. That makes it a
  controlled test of the context-isolation rationale in
  [[concepts/hierarchical-delegation]], which the concept has so far argued
  from AIBuildAI's system-level results. The answer is conditional: isolation
  pays when the delegated unit has a stated input/output interface and the
  main context is under pressure. It pays little for frontier models with a
  well-scoped library, and it costs 1.3–1.7× the tokens. It also gives the
  graph a second, independent attestation of [[literature/papers/kim2026why]]'s
  "scope your loading" result, from a different group, benchmark and model
  family. Agent-skill Qwen goes from ≈0.22 with all skills exposed to ≈0.50
  with task-scoped lazy loading (read from charts). Like kim2026why, the flat baseline is "dump
  everything", not retrieval at matched context size. Not a 5: the contract
  claim is confounded, and the hierarchy result rests on one model.
- **For this box:** the "routing inline, leaves delegated" result matches how
  workflow scripts already run here. The orchestrator plans inline, and spawned
  subagents return a schema-validated `StructuredOutput`, which is an explicit
  output contract. The transferable, cheap change is adding
  "Expected input / Output" lines to skill descriptions for leaf-like skills
  (`/fetch-paper`, `/ingest`, `/discover`), while leaving routing skills such
  as `/curate` and `/digest` inline. This does not conflict with the concise,
  goal-level skill style: a contract states the goal. The evidence does **not**
  support expecting accuracy gains on an Opus executor. The documented benefit
  there is lower peak context, at a token cost.
- Watch for: an ablation that removes contracts from the synthesized skills,
  released skill packages, and a held-out-task version of the synthesis
  pipeline.
