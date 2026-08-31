---
kind: paper
title: "When Tool Outputs Become Commands: Separating Action Induction from Runtime Authorization in Tool-Augmented LLM Agents"
authors: ["Xiaokun Guo", "Zhen Xu", "Dongdong Huo", "Yanqiu Zhang", "Wei Wang", "Qinfu Yang", "Dongjin Yu", "Yu Wang"]
institutions: ["Institute of Information Engineering, Chinese Academy of Sciences", "School of Cyber Security, University of Chinese Academy of Sciences"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.27146"
code_url: null
citations: null
source: "raw/papers/guo2026when.pdf"
added: "2026-08-31"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/information-firewall]]"
tags: ["agent-security", "prompt-injection", "provenance", "safety", "agent-architecture"]
---

# When Tool Outputs Become Commands: Separating Action Induction from Runtime Authorization in Tool-Augmented LLM Agents

## TL;DR

When a tool output stops merely providing data and starts specifying an
action, it has become a command. SARA argues this risk comes from
**conflating action induction with execution authorization**, and separates
the two as distinct runtime roles: provenance of *what induced* an action is
recorded on the Observation side, while authorization is decided only against
the user objective and audited evidence from prior authorized successes.

## Claims

- The structural tension in open-ended agents: restricting untrusted external
  content improves security but weakens the runtime adaptability the tasks
  require — a direct security–utility trade-off. Separating induction from
  authorization is a way out that does not narrow the action space.
- **Action provenance and execution authority are different things** and
  should be tracked separately across steps.
- **No-History-Promotion**: without an explicit rule, an action origin can be
  "laundered" into execution authority simply by recurring in the history.
  Multi-step execution therefore needs an invariant preventing historical
  recurrence from conferring authority.
- Authorization should require goal-level, execution-chain-level, and
  argument-level support simultaneously.

## Methods

- A **context-isolated Action Probe** on the Observation side exposes
  action-inducing semantics and persistently records action-origin provenance
  across steps as a review signal.
- Execution side authorizes tool calls only against (a) the user objective and
  (b) audited evidence from authorized *successful* executions.
- No-History-Promotion invariant preserves the separation across multi-step
  runs.
- Evaluated on **AgentDojo** and **AgentDyn**, across four primary settings
  and multiple agent backbones.

## Results

- Attack success rate limited to **≤0.63%** across the four primary
  evaluation settings, with competitive task utility retained.
- ASR reductions hold consistently across additional agent backbones.

## Critique / open questions

- No released code, so the Action Probe's isolation boundary — the part that
  determines whether this actually works — cannot be independently checked.
- AgentDojo and AgentDyn are the standard benchmarks for this class, which is
  good for comparability but means the attacks are largely known ones; there
  is no adaptive adversary designed against SARA's specific invariant.
- "Audited evidence from authorized successful executions" is a bootstrapping
  story — the first authorization in a chain has no prior success to appeal
  to. The paper does not obviously address the cold-start case.

## Trust signals

- **Credibility:** 3 — IIE / CAS is a reputable security group and the
  evaluation uses standard, comparable benchmarks with multiple backbones.
  Held at 3 rather than 4 by the absence of released code and peer review.

## Follow-up

- **Relevance:** 4 — supplies the decomposition
  [[concepts/permission-gate-as-architecture]] has been missing. That concept
  argues the gate is architectural but does not say *what the gate
  distinguishes*; "induction vs authorization" is a crisp, implementable
  answer.
- No-History-Promotion is the sharpest idea here and generalises well beyond
  security: any system that treats "we did this before" as license is exposed
  to the same laundering. Worth holding against this project's own
  context-inheritance behavior — see
  [[literature/papers/nakayashiki2026when]] on inherited constraints.
- The context-isolated probe is an [[concepts/information-firewall]]
  construction: the component that reads untrusted content is denied the
  authority to act on it.
- Compare [[literature/papers/zhan2026auto]] (artifact-level typed policy) and
  [[literature/papers/leong2026recognition]] (external reference monitor) —
  three placements of the same enforcement boundary in one week.
