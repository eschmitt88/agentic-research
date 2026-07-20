---
kind: paper
title: "HAS-Bench: Evaluating LLM-Based Human-Agent Systems under Configurable Human Participation"
authors: ["Yaozu Wu", "Wei-Chieh Huang", "Jizhou Guo", "Dongyuan Li", "Renhe Jiang", "Henry Peng Zou", "Chunyu Miao", "Shanghao Li", "Weizhi Zhang", "WeiWei Ye", "Yankai Chen", "Meng Zhang", "Xue Liu", "Philip S. Yu"]
institutions: ["The University of Tokyo", "University of Illinois Chicago", "MBZUAI", "McGill University", "Zhejiang University"]
year: 2026
venue: "arXiv preprint (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.04329"
code_url: null
citations: null
source: "raw/papers/wu2026hasbench.pdf"
added: "2026-07-20"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/pass-at-k]]"
tags: [permission-gate, human-in-the-loop, governance, benchmark, evaluation, coordination, agent-architecture]
---

# HAS-Bench: Evaluating LLM-Based Human-Agent Systems under Configurable Human Participation

## TL;DR

Makes human participation in agent systems a *configurable, measurable
architectural parameter*: HAS-Framework puts humans and agents in one
typed interaction graph (explicit roles, permissions, communication
paths, action authority), and HAS-Bench (397 tasks, 6 domains) sweeps a
5-level Human Agency Scale to show equal-partnership participation lifts
Pass@1 by ~8.4 points, rescues 17–28% of autonomous failures, and adds
+26.9 points of Safety Rate on authorization-critical tasks — while
*stronger* human control (A4) brings diminishing or negative returns at
+50% interaction cost.

## Claims

- Existing frameworks support human involvement via "external supervision
  mechanisms such as runtime interrupts, approval gates, or
  reviewer-style checkpoints," which "keep humans outside the system,
  treating human participation as a late-stage supervision signal rather
  than a designable and schedulable component." The fix is graph-native:
  humans as nodes with the same role/permission/visibility schema as
  agents, typed edges routing requests to the right human *participant*
  rather than a generic "human input" interface.
- Human involvement decomposes into three channels acting at different
  loci: **clarification** (input), **feedback** (output), **control**
  (action authority — approve/veto/override/execute). Collapsing them
  into one "human input" channel obscures which one does the work.
- Human authority is a 5-point dial (A1 full automation → A5
  human-driven), each level compiled to a concrete runtime policy
  (available channels, triggering conditions, authority over actions) —
  so gate placement becomes a reproducible experimental variable.
- Process-level metrics matter as much as outcomes: Control Request
  Justification explicitly scores whether authorization requests are
  "well-timed and warranted for high-stakes, ambiguous, or
  policy-sensitive actions, rather than over-asking on trivial ones" —
  the false-refusal/over-escalation cost made measurable.

## Methods

- 397 tasks across Retail/Telecom/Airline (stateful, policy-constrained),
  Coding/Research (open-ended expert), Bargaining — distilled from 2,749
  source tasks (τ²-Bench, MultiAgentBench) via rule filtering + 4-LLM-judge
  adaptation + reviewer-panel and sampled human validation.
- Six problem patterns incl. Safety-Critical Authorization (n=35);
  formalized as a partially observable multi-party process over the HAS
  graph. GPT-4.1 user simulator and judge (deterministic settings, fixed
  rubrics); backbones GPT-4.1(-mini), Claude Sonnet 4, DeepSeek-V3,
  Llama-3.1-8B.
- Outcome metrics: pass@1, Task Score, Delivery Rate, Safety Rate, HAS
  Rescue Rate; process metrics: CQS, FUR, CRJ, Action Safety Rate,
  Initiative Entropy, Human Intervention Rate, plus interaction cost.

## Results

- A3 (equal partnership) vs A1 (full autonomy): +8.4 Pass@1 / +11.5 Task
  Score averaged over models; capable models gain most (GPT-4.1
  +16.9/+20.5). Rescue: 17–28% of A1 failures recover under A3, most in
  collaboration-intensive domains (Coding, Research).
- Safety Rate on the authorization pattern improves +26.9 average across
  all backbones — the largest single effect, and it lands exactly where
  the gate governs (protected writes executed without approval).
- More human control ≠ better: A4 adds >50% turns over A3 for
  diminishing, sometimes negative returns — the gains "depend on when,
  how, and by whom human input is exercised."

## Critique / open questions

- Human side is a GPT-4.1 simulator; robustness study across simulators
  exists but the persona/simulator realism ceiling is the main external
  validity question (sampled human verification mitigates, doesn't
  remove).
- No code/artifact release found in the paper — for a benchmark this is
  the key missing trust signal; watch for a repo.
- LLM-judged process metrics (CQS/FUR/CRJ) inherit judge bias even with
  fixed rubrics and deterministic decoding.
- A2/A5 omitted from main experiments (argued near-redundant); the dial's
  ends are thus asserted more than measured.

## Trust signals

- **Credibility:** 3 — reputable multi-institution group (U Tokyo, UIC/
  Philip S. Yu, MBZUAI, McGill, Zhejiang), careful multi-stage validation
  protocol, but arXiv-only, no released code/artifacts yet, no citations.

## Follow-up

- **Relevance:** 4 — supplies the missing evidence class for
  [[concepts/permission-gate-as-architecture]]: a benchmark that treats
  gate/authority placement as a swept variable and quantifies both the
  safety benefit (+26.9 Safety Rate on authorization tasks) and the
  over-gating cost (A4's +50% turns for diminishing returns) — /elevate
  held the concept partly for lack of exactly this.
- The clarification/feedback/control channel split and the CRJ
  (justified-request) metric are directly importable design vocabulary
  for harness gates (e.g. PreToolUse policies that must not over-ask).
- Watch for artifact release; also τ²-Bench (dual-control) as the
  adjacent source if the human-side-of-the-gate thread thickens.
