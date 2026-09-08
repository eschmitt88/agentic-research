---
kind: paper
title: "SoK: When Safe Agents Fail Together: The Security of Multi-Agent LLM Systems"
authors: ["Rui Yang", "Junjie Xu", "Zhengyu Liu", "Neil Fendley", "Yang Hong", "Ziyang Li", "Yinzhi Cao"]
institutions: ["Johns Hopkins University", "Johns Hopkins APL", "Nanyang Technological University"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.00595"
code_url: null
citations: null
source: "raw/papers/yang2026sok.pdf"
added: "2026-09-08"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["safety", "governance", "multi-agent", "security", "survey", "evaluation", "enforcement"]
---

# SoK: When Safe Agents Fail Together: The Security of Multi-Agent LLM Systems

## TL;DR

Systematises 197 works by treating **the complete multi-agent execution as
the unit of analysis**. Core move: trace adversarial influence from its
starting position (A) through interaction interfaces (I) to a system-level
failure (R). Central warning — without an execution-level view, *a failure
observed in a multi-agent setting is routinely mistaken for evidence of a
genuinely multi-agent effect*.

## Claims

- **Safe agents can fail together.** MAS move information, state, decisions
  and authority across principal boundaries; safeguards verified per-agent
  do not compose across those boundaries.
- Prior surveys organise by component (models, prompts, memory, tools) or by
  MAS property (communication, trust, topology). What none answer is *how an
  attack moves*: where it enters, which interface it uses, what failure it
  finally causes.
- Defences should be stated as a **five-part contract**: path target,
  observation, intervention, trust boundary, recovery. **Path closure and
  recovery are the under-addressed parts.**
- Evaluation claims need counterfactuals — otherwise a single-agent failure
  reproduced in a multi-agent harness reads as an interaction effect.

## Methods

- Execution-centred systematisation of **197 works**: six interaction
  interfaces, four adversary positions, seven system-level risks, eight
  recurring attack paths (P1–P8), eight configuration dimensions (C1–C8).
- A separate **audit of 44 evaluation and benchmark works**.

## Results

- The A→I→R framework unifies attack mechanisms that were fragmented across
  incompatible threat models and units of analysis.
- Benchmark audit surfaces four open problems: isolating interaction
  effects, comparable and diagnostic metrics, reuse across MAS designs, and
  evaluating **open-system** operation.

## Critique / open questions

- A systematisation, not new measurement — it inherits whatever the 197
  works got wrong, and the paper's own point about inconsistent evidence
  standards applies to its corpus.
- The five-part defence contract is a taxonomy, not yet a testable spec; no
  instantiation is evaluated.
- "Path closure and recovery are hard" is the honest finding, but it is
  stated as an open challenge rather than approached.

## Trust signals

- **Credibility:** 4 — arXiv preprint (not yet peer reviewed, no citations),
  but from a strong security group: Johns Hopkins + JHU APL + NTU, with
  Yinzhi Cao as senior author. SoK is a recognised form with an established
  bar, and the corpus size (197 + 44) is verifiable. No artifact released,
  which is the main thing keeping this off 5.

## Follow-up

- **Relevance:** 4 — restates [[concepts/enforcement-boundary-placement]] at
  the multi-agent scale: the placement question is not *which layer* but
  *which crossing*, and the answer is that per-agent checks do not compose.
  This complements the six single-site placements the concept now carries
  from marsden / zheng / ding without duplicating any of them.
- The methodological warning is the most transferable part and applies
  directly to this project: **observing a failure in a multi-agent setting
  is not evidence of a multi-agent effect without a counterfactual.** That
  is [[concepts/hce-evaluation]] discipline applied to architecture claims,
  and it is a bar much of the MAS literature in `literature/` does not meet.
- The benchmark audit is a second front on [[concepts/hce-evaluation]]:
  44 evaluation works, and the finding is that comparability and diagnostic
  power are broadly missing.
- "Information, state, decisions and authority crossing principal
  boundaries" is the general form of the mechanism
  [[concepts/shared-substrate-contagion]] was seeded from — see
  [[literature/papers/paglieri2026case]], where the crossing medium is a
  shared skill library. Counterpoint in
  [[literature/papers/yoon2026arcticswarm]], which restricts peer reads
  deliberately and gains accuracy.
