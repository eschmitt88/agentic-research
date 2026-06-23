---
kind: paper
title: "Act As a Real Researcher: A Suite of Benchmarks Evaluating Frontier LLMs and Agentic Harnesses in the Research Lifecycle"
authors: ["Jiayu Wang", "Weijiang Lv", "Bowen Fu", "Jing Fu", "Jiayi Song", "Lingyu Zhang", "Lanxuan Xue", "Luodi Chen", "Zepeng Xin", "Kaiyu Li", "Xiangyong Cao"]
institutions: ["Xi'an Jiaotong University", "Xidian University"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2606.07462
code_url: https://github.com/AARR-bench/AARRI-bench
citations: null
source: "raw/papers/wang2026act.pdf"
added: "2026-06-23"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/hce-evaluation]]"
tags: [benchmark, research-agent, harness-vs-model, researcher-quality, lifecycle, multi-harness, scaffolding]
---

# Act As a Real Researcher: A Suite of Benchmarks Evaluating Frontier LLMs and Agentic Harnesses in the Research Lifecycle

## TL;DR

AARRI-Bench (the inaugural "Act As a Real Research **Intern**" benchmark
in the planned AARR series) is 82 manually-crafted research-intern tasks
designed to test whether agents emulate **researcher-quality behavior** —
integrity, uncertainty awareness, careful verification, knowing when to
disagree — rather than just task completion. Its defining structural move
is **multi-harness evaluation**: every task is run under the Harbor
framework across 16 harness×model combinations, **decomposing frontier-LLM
capability from agentic-harness contribution**. The best configuration
(Mini-SWE-Agent + Claude Opus 4.7) reaches only **68.3%**, and a striking
finding is that the *minimalist* harness beats feature-rich ones on the
strongest model.

## Claims

- Existing research-agent benchmarks measure task completion and final
  outcomes but **overlook researcher qualities** — integrity, awareness of
  uncertainty, careful verification, responsible scientific reasoning.
  AARRI targets these directly.
- Most benchmarks pose tasks that are *hard for humans*; AARRI deliberately
  uses the inverse design principle — **tasks that are easy for humans but
  where agents are highly likely to make mistakes** — to surface the
  human-agent gap.
- Agent quality is the **joint product of the underlying model and the
  harness/scaffolding**, and these must be measured separately. AARRI is
  built to evaluate "both the underlying model and the agent harness"
  simultaneously via the Harbor framework.
- **Complex scaffolding is not a prerequisite for superior performance.**
  The minimalist Mini-SWE-Agent + Claude Opus 4.7 (68.3%) outperforms the
  feature-rich Hermes Agent (64.6%) and Claude Code (62.2%) on the *same*
  frontier model. Heavyweight harnesses appear to add cognitive overhead /
  distraction for strong models.
- **The model is the primary bottleneck.** Across all harnesses,
  performance drops sharply with lower-tier models; the intrinsic reasoning
  of the backbone dominates.
- Even the best configuration leaves ~32% of intern-level research tasks
  unsolved — building "researcher-like AI requires further exploration of
  research *behavior*, rather than merely complex scaffolding."

## Methods

- **Two-dimensional taxonomy** over 82 tasks. *Horizontal* (task scenario):
  Context (field-sensitivity / intuitive scientific judgment), Mindset
  (academic self-awareness, courage to disagree, recognizing dead ends),
  Hands-on (coding / experimental setup / data processing), Interaction
  (tool use + collaboration). *Vertical* (agent scope, by autonomy level):
  S1-Adaptation 32%, S2-Integration 28%, S3-Innovation 27%, S4-Open-ended
  13%.
- **Manual construction**, three stages (free creation → guided expansion →
  consolidation) by a team of senior PhD students down to undergrad interns,
  each drawing on real pain points using LLM agents for research. Table 1
  positions AARRI as the only benchmark combining end-to-end tasks,
  fine-grained eval, *researcher-quality* eval, *manual* data generation,
  and *multi-harness* eval at once.
- **Harbor evaluation framework.** Each task is a standardized directory
  (`instruction.md`, `task.toml`, `environment/` with a Dockerfile,
  `solution/`, `tests/`) run in a clean containerized cloud environment
  (Daytona, Modal). Harbor "standardizes the format... and enables the
  simultaneous evaluation of both the underlying model and the agent
  harness."
- **16 harness×model combinations.** Harnesses: Claude Code, Hermes Agent,
  Mini-SWE-Agent. Models: Claude Opus 4.7, Claude Sonnet 4.6, GPT-5.3 Codex,
  Qwen 3.6 Plus (closed) and MiniMax-M2.7, Kimi-K2.6, DeepSeek-V4-Flash
  (open). Models sourced directly from providers/OpenRouter, no third-party
  relay.
- **Two metric granularities.** *Classic 0/1 reward* (SWE-bench / Terminal-
  bench style final-completion-only — the headline metric, chosen to avoid
  step-wise partial-credit misjudging valid exploratory behavior) and
  *fine-grained unit tests* (multiple hand-crafted tests per task, used for
  case-study and ablation analysis).

## Results

- **Best overall: Mini-SWE-Agent + Claude Opus 4.7 = 68.3%** (Table 2).
  Same model under Hermes Agent = 64.6%, under Claude Code = 62.2%. Lowest
  overall config = Claude Code + Kimi-K2.6 = 51.3%.
- **Harness-model synergy is highly non-uniform.** Lower-tier models
  cluster tightly (MiniMax-M2.7 spans only 56.1-58.1% across harnesses), but
  moving to Claude Opus 4.7 triggers disparate scaling: Mini-SWE-Agent gains
  **+11.5%** vs the minimum-score baseline while Claude Code gains only
  +6.1%. Rigid, over-engineered harnesses appear to *cap* the scaling
  headroom of intelligent models.
- **Trajectory statistics** reveal harness behavioral fingerprints: Claude
  Code has wide, long-tailed step distributions (up to 131 steps) prone to
  runaway / redundant paths; Hermes Agent is highly condensed (μ ≈ 8.4-9.0
  steps); Mini-SWE-Agent sits in a stable middle (μ ≈ 15.4-15.9) robust to
  model swaps — a predictable footprint that avoids both catastrophic
  looping and premature termination.
- **Case studies.** Task 26fb63 (fabricated-data detection in manuscript
  review): a dataset has identical trailing decimals across all results — an
  obvious-to-humans misconduct pattern; **almost all configs missed it**,
  only Claude Code + Claude Opus 4.7 caught it. Task 429504 (avoiding
  re-proposing already-rejected directions): Mini-SWE-Agent + Opus 4.7
  captured category boundaries accurately; Hermes Agent re-submitted a
  reworded duplicate of a rejected idea.

## Critique / open questions

- **Small and single-institution.** 82 manually-built tasks from two
  universities (Xi'an Jiaotong, Xidian). Researcher-quality judgments
  ("integrity", "courage to disagree") are inherently subjective; how
  inter-annotator agreement and the 0/1 reward map onto these soft
  qualities is not detailed in the skimmed sections.
- **AARRA / AARRS are vaporware so far.** Only the *intern*-level benchmark
  exists; the "Assistant" and "Scientist" tiers (and promised LLM-judge for
  open-ended questions, MCP/skill support, crowdsourced scale-up) are
  future work.
- **Harness comparison is confounded by harness-specific defaults.** "Same
  model, different harness" still varies tool sets, prompts, and step
  budgets; the conclusion "minimalist beats complex" is plausible but the
  causal attribution to scaffolding-as-distraction is interpretive, not
  ablated to a single controlled variable.
- **Best-config ceiling of 68.3% is the headline, but per-category numbers
  are noisy** (e.g. Mindset saturates at 76.9% for several configs),
  suggesting some categories have few discriminating tasks.

## Trust signals

- **Credibility:** 3 — a v1 arXiv preprint from two solid Chinese
  universities with **data/code released** (github.com/AARR-bench/
  AARRI-bench) and a clean, reproducible Harbor-containerized eval design.
  Held to 3 (not 4) by the small single-region task set, the not-yet-built
  remainder of the AARR series, and subjective researcher-quality labels
  whose annotation protocol is under-specified — short of a broad,
  fully-validated benchmark.

## Follow-up

- **Relevance:** 4 — squarely on the dominant front (benchmarking
  autonomous research agents) and a direct attestation of
  [[concepts/hybrid-model-backends]]: AARRI's whole Harbor design exists to
  **decompose model capability from harness contribution**, and its central
  empirical finding — a minimalist harness (Mini-SWE-Agent) lets a frontier
  model scale where a heavyweight harness caps it, while lower-tier models
  are harness-insensitive — is exactly the "use the right tier for the right
  role, and don't over-scaffold the strong model" thesis, recast as a
  *harness×model interaction matrix*. It also supports
  [[concepts/hce-evaluation]] as a lifecycle/researcher-quality holdout:
  manually-built tasks with hidden `tests/`, 0/1 final-completion scoring
  chosen specifically to avoid step-wise partial-credit gaming. Held at 4
  (not 5) because it strengthens existing concepts rather than seeding a new
  importable pattern, and the harness-vs-model attribution is observational
  rather than mechanistically controlled.
