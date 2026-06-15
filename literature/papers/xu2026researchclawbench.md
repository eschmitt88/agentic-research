---
kind: paper
title: "ResearchClawBench: A Benchmark for End-to-End Autonomous Scientific Research"
authors: ["Wanghan Xu", "Shuo Li", "Tianlin Ye", "Qinglong Cao", "Yixin Chen", "Hengjian Gao", "Yiheng Wang", "Qi Li", "Kun Li", "Sheng Xu", "et al."]
institutions: ["Shanghai Artificial Intelligence Laboratory"]
year: 2026
venue: "arXiv"
peer_reviewed: false
url: https://arxiv.org/abs/2606.07591
code_url: https://github.com/InternScience/ResearchClawBench
citations: null
source: "raw/papers/xu2026researchclawbench.pdf"
added: "2026-06-15"
relevance: 5
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
tags: [benchmark, autoresearch, scientific-discovery, evaluation, held-out, rubrics, re-discovery, multimodal]
---

# ResearchClawBench: A Benchmark for End-to-End Autonomous Scientific Research

## TL;DR

A benchmark for *end-to-end* autonomous scientific research: 40 tasks across
10 scientific domains, each grounded in a real published paper that supplies
related literature and raw experimental data while **hiding the target paper
during evaluation**. Expert-curated multimodal rubrics decompose each target
artifact into weighted, verifiable sub-criteria; scoring is anchored at 50
points = target-paper-level re-discovery, with higher scores indicating
genuine new discovery. The strongest autonomous agent (Claude Code) averages
only 21.5 and the best native LLM (Claude-Opus-4.7 via the lightweight
ResearchHarness) only 20.7 — current systems are far from reliable
re-discovery, failing mostly on experimental protocol mismatch, evidence
mismatch, and missing scientific core.

## Claims

- Existing benchmarks cover adjacent-but-incomplete settings (scientific QA,
  interactive science environments, paper reproduction) but none asks a system
  to start from *raw experimental data*, produce a *complete* research output,
  and be evaluated against *verifiable anchors* — the gap RCBench fills.
- Open-ended scientific outputs cannot be scored by exact-match or unit tests,
  and LLM-as-judge introduces bias; the fix is expert-curated rubrics that
  decompose the target artifact into weighted, checkable sub-criteria.
- Hiding the target paper while supplying its inputs makes the benchmark a
  re-discovery test with no leakage of the answer — a held-out evaluation
  frontier rather than a search signal.
- Scoring anchored at 50 = "matches the target paper" lets a single scale span
  both re-discovery (≤50) and novel discovery (>50).
- Both scaffolded agents and native LLMs struggle: best autonomous agent 21.5,
  per-task frontier mean 24.6, native-LLM frontier mean 26.5 — none crosses the
  50-point re-discovery threshold.
- Failures concentrate in three named modes: experimental protocol mismatch,
  evidence mismatch, and missing scientific core.

## Methods

- 40 expert-authored tasks spanning Astronomy, Chemistry, Earth, Energy,
  Information, Life, Materials, Mathematics, Neuroscience, Physics; each
  converted by domain experts from a real paper with clear question,
  accessible raw data, and practical value.
- Target paper kept hidden on the evaluation side; related literature + raw
  data given as inputs.
- Expert-curated **multimodal rubrics** decompose expected outputs into
  weighted verifiable/scored sub-criteria.
- **ResearchHarness**: a unified lightweight tool-use harness so native LLMs
  (without a full agent scaffold) can be evaluated under the same protocol.
- Unified evaluation of 7 autonomous research agents (incl. Claude Code, Codex
  CLI, OpenClaw, ARIS, Evo, Nanobot) + 17 native LLM baselines.

## Results

- Autonomous agents: Claude Code 21.5 (best), down through Codex CLI / OpenClaw
  / etc.; per-task frontier (best agent per task) 24.6.
- Native LLMs via ResearchHarness: Claude-Opus-4.7 20.7 (best), frontier mean
  26.5; no system reaches the 50-point re-discovery line.
- Error analysis: experimental protocol mismatch, evidence mismatch, missing
  scientific core dominate failures.

## Critique / open questions

- Like AutoMedBench and AutoResearchBench, the architectural contribution is
  the *evaluation rig*, not a new agent — but the rig here is unusually clean:
  hidden target + weighted verifiable rubric + a 50-point re-discovery anchor.
- The 50-point anchor presumes the target paper is the "correct" answer; how it
  handles tasks where the original paper is itself flawed or where a better
  solution exists below the anchor is open.
- ResearchHarness deliberately strips the agent scaffold to isolate native-LLM
  capability — a useful LLM-vs-harness decomposition (cf. AARRI-Bench), but the
  ~26 vs ~24 frontier gap suggests the scaffold currently *hurts* as often as
  it helps, which deserves its own study.

## Trust signals

- **Credibility:** 4 — Shanghai AI Laboratory (Intern Discovery / Agentic
  Science), with code (github.com/InternScience/ResearchClawBench), a project
  page, and a HuggingFace dataset released; arXiv preprint (v2 Jun 10 2026),
  not yet peer-reviewed.

## Follow-up

- **Relevance:** 5 — canonical fresh evidence for `hce-evaluation`: the
  hidden-target-paper design *is* a held-out evaluation with no answer leakage,
  exactly the discipline HCE encodes. The expert-curated weighted rubric is a
  `programmable-evaluator-oracle` instance (a checkable fitness oracle over an
  open-ended artifact), and the three named failure modes (protocol / evidence
  / missing-core) attest `typed-claim-partition` and `citation-anchoring`
  (evidence mismatch = unanchored claims). Dominant scientific-research-agent
  front (cf. AutoMedBench, AutoResearchBench, PaperBench).
- The ResearchHarness LLM-vs-agent decomposition is a clean design for a
  downstream experiment on how much an agent scaffold actually adds over a bare
  tool-use harness.
