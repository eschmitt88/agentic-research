---
kind: paper
title: "Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers"
authors:
  - Pengfei Du
institutions: ["Hong Kong Research Institute of Technology"]
year: 2026
venue: "arXiv:2603.07670 [cs.AI]"
peer_reviewed: false
url: "https://arxiv.org/abs/2603.07670"
code_url:
citations:
source: "raw/papers/du2026memory.pdf"
added: "2026-06-03"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/selective-memory-retrieval]]"
tags:
  - agent-memory
  - survey
  - taxonomy
  - memory-evaluation
  - write-manage-read
  - reflection
  - forgetting
---

# Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers

## TL;DR

(Skim of abstract page.) A single-author survey of LLM-agent memory
covering 2022–early 2026. It formalizes memory as a
**write–manage–read** loop coupled to perception and action, then lays
out a three-dimensional taxonomy (temporal scope, representational
substrate, control policy) and examines five mechanism families. On
evaluation it traces the shift from static recall benchmarks to
multi-session agentic tests and analyzes four recent benchmarks that
expose persistent gaps.

## Claims

- Memory — persisting, organizing, and selectively recalling
  information across interactions — is what converts a stateless text
  generator into an adaptive agent.
- Agent memory is best formalized as a write–manage–read loop tightly
  coupled with perception and action.
- A three-axis taxonomy — temporal scope, representational substrate,
  control policy — spans the design space.
- Current systems have stubborn gaps that only multi-session,
  decision-interleaved evaluation exposes; static recall benchmarks
  hide them.

## Methods

- Survey methodology. Five mechanism families examined in depth:
  (1) context-resident compression, (2) retrieval-augmented stores,
  (3) reflective self-improvement, (4) hierarchical virtual context,
  (5) policy-learned management.
- Evaluation section analyzes four recent multi-session agentic
  benchmarks; applications section covers personal assistants, coding
  agents, open-world games, scientific reasoning, multi-agent teamwork.
- Discusses engineering realities: write-path filtering, contradiction
  handling, latency budgets, privacy governance.

## Results

- No experiments (survey). Output is the write–manage–read
  formalization, the three-axis taxonomy, the five-family mechanism
  map, and a list of open challenges: continual consolidation, causally
  grounded retrieval, trustworthy reflection, learned forgetting,
  multimodal embodied memory.

## Critique / open questions

- Single-author survey, no affiliation shown — coverage and
  authority are harder to gauge than a multi-lab survey; treat its
  taxonomy as one useful lens rather than a consensus.
- No code/resource repo noted; value is conceptual organization.
- "Learned forgetting" and "policy-learned management" overlap with
  this project's `context-eviction-policy` and
  `selective-memory-retrieval` — useful cross-check on framing.

## Trust signals

- **Credibility:** 2 — Single-author arXiv preprint with no affiliation
  shown and no Comments/venue field; not peer-reviewed. Useful as a
  conceptual map (the write–manage–read loop and three-axis taxonomy
  are clean), but it carries less corroboration weight than the
  multi-author surveys in the same cluster. Credibility here is about
  the survey's authority, independent of its on-mission relevance.

## Follow-up

- Second independent survey source for the memory cluster (alongside
  `yang2026graph`). Together they strengthen the case for promoting an
  agent-memory Map of Content — five+ related concepts already exist
  (`agent-native-memory`, `selective-memory-retrieval`,
  `context-eviction-policy`, `structured-world-model`,
  `skill-library-lifecycle`).
- The write–manage–read framing maps cleanly onto this project's own
  `/ingest` (write), `/lint` + consolidation (manage), and
  retrieval-on-demand (read) — a tidy vocabulary to adopt.
- The four analyzed multi-session benchmarks are candidate yardsticks
  for evaluating future memory papers (pairs with `hce-evaluation`).
</content>
