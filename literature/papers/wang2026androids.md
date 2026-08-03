---
kind: paper
title: "Do Androids Dream of Breaking the Game? Systematically Auditing AI Agent Benchmarks with BenchJack"
authors: ["Hao Wang", "Hanchen Li", "Qiuyang Mang", "Alvin Cheung", "Koushik Sen", "Dawn Song"]
institutions: ["UC Berkeley"]
year: 2026
venue: arXiv (2605.12673)
peer_reviewed: false
url: https://arxiv.org/abs/2605.12673
code_url: https://github.com/benchjack/benchjack
citations: null
source: "raw/papers/wang2026androids.pdf"
added: "2026-08-03"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
tags: [evaluation, reward-hacking, benchmark-auditing, red-teaming, trust-boundary, taxonomy]
---

# Do Androids Dream of Breaking the Game? Systematically Auditing AI Agent Benchmarks with BenchJack

## TL;DR

First quantitative audit of agent-benchmark hackability: an automated
red-teaming agent (BenchJack) built on an eight-class flaw taxonomy
synthesizes working reward-hacking exploits achieving near-perfect
scores on 9 of 10 popular benchmarks — including SWE-bench Verified
via a nine-line PyTest `conftest.py` hook and MLE-Bench via shipped
answers + logic gaps — without solving a single task. An iterative
attack-patch loop drives hackable-task ratio from ~100% to <10% on
benchmarks without fatal design flaws.

## Claims

- Reward hacking emerges spontaneously in frontier models without
  overfitting; benchmarks must be **secure by design**, and current
  evaluation pipelines "have not internalized an adversarial mindset."
- Eight recurring flaw classes: V1 agent/evaluator isolation failure,
  V2 answers shipped with the test, V3 remote code execution in the
  evaluator, V4 LLM-judge prompt injection, V5 weak string matching,
  V6 evaluation logic gaps, V7 trusting output of untrusted code,
  V8 excessive permissions. V1 and V7 are the most exploit-enabling:
  few in number but generalizing across every task at once.
- Post-hoc hack detection (LLM-judge trajectory monitors) is gullible
  and can only fire after the hack happens; proactive pre-release
  auditing shifts the check to before agent execution.
- Patching has a hard limit: benchmarks whose *design* violates trust
  boundaries (agent and evaluator sharing an environment) stay
  hackable after patching — "these flaws are not bugs to be patched...
  a code-only patch cannot move the trust boundary back into place."

## Methods

- Taxonomy distilled from real incidents (IQuest-Coder-V1's `git log`
  gold-patch copying; OpenAI's SWE-bench-Verified audit; METR's >30%
  spontaneous hack observations) into the **Agent-Eval Checklist**
  (30 binary questions, 7 categories) for pre-release use.
- **BenchJack** pipeline: (1) reconnaissance — map entry points,
  scoring functions, task configs, and trust boundaries; (2)
  taxonomy-guided flaw scan with a static-analyzer toolbox (semgrep
  rules, Dockerfile analyzer, AST-based trust mapper) feeding a flaw
  ledger; (3) exploit construction under a strict validity rule: run
  through the official entry point, default agent harness, no
  pre-patching — the exploit must achieve the score without solving
  or cheating outside what a model could do during evaluation.
  Also packaged as a coding-agent skill (`/benchjack <benchmark>`).
- Iterative refinement: BenchJack as adaptive attacker vs. a coding
  agent as defender patching against each verified exploit, looping
  until no exploit or non-patchable.
- Audited: SWE-bench Verified/Pro, FrontierSWE, MLE-Bench,
  SkillsBench, Terminal-Bench, OSWorld, WebArena, NetArena,
  AgentBench (thousands of tasks; Claude Code as the agent backend).

## Results

- 219 distinct flaws across 10 benchmarks, spanning six of eight
  classes; near-perfect exploit scores on 9/10 (AgentBench lowest
  because its exploit only fully hacks the dbbench subset).
- Major-flaw table: SWE-bench Verified V7, MLE-Bench V2&V6,
  Terminal-Bench V1, OSWorld V7, WebArena V2&V5 — the de facto
  standards of ML-agent evaluation are the flawed ones.
- Severity is bimodal: most findings cover one task or *all* tasks;
  a single all-task flaw causes a hack-rate explosion while dozens of
  task-specific bugs barely move it.
- Single-round patching kills the original exploit almost everywhere,
  but re-running BenchJack restores high hack rates on
  poorly-designed benchmarks; only 4 of 10 cut hack rate by more
  than half. With three iterative rounds, WebArena and OSWorld reach
  0% hackable; AgentBench and SWE-bench Pro <10%.

## Critique / open questions

- Exploit validity is verified by achieving the score, but flaw
  *counts* (219) depend on the scanning agent's judgment; no
  human-validation rate is reported for the ledger itself.
- Single agent backend (Claude Code) for both attack and defense;
  attacker-strength sensitivity is unexplored — a weaker auditor
  would certify a hackable benchmark as clean, so "BenchJack finds
  nothing" is evidence, not proof (same asymmetry as any red-team).
- The generative-adversarial patching loop shares failure modes with
  its GAN inspiration: convergence claims rest on three rounds on
  four benchmarks.
- The checklist is the transferable artifact; the exploit corpus is
  the evidence. For research-agent evaluation the missing extension
  is *data* leakage (contamination) rather than *scoring* exploits —
  complementary to wang2026search's search-time contamination.

## Trust signals

- **Credibility:** 4 — UC Berkeley group (Cheung, Sen, Song), code
  released, incident-grounded taxonomy, exploits verified end-to-end
  against real benchmark harnesses; preprint, not yet peer-reviewed,
  and headline flaw counts are agent-generated.

## Follow-up

- **Relevance:** 4 — the evaluator-side complement to this graph's
  agent-side reward-hacking sources ([[literature/papers/zhao2026specbench]]
  measures the agent's gap, [[literature/papers/atinafu2026rewardhacking]]
  the ML-engineering incident classes — cited here as related work):
  BenchJack audits whether the *oracle itself* is sound, which is the
  precondition [[concepts/programmable-evaluator-oracle]]'s "the
  final scorer is not allowed to be gamed" silently assumes. MLE-Bench
  V2&V6 findings directly concern this project's downstream mle-bench
  work.
- The trust-boundary framing (V1/V7 as boundary violations that
  patches cannot restore) is design-time evidence for
  [[concepts/information-firewall]]'s claim that separation must be
  structural, and for [[concepts/hce-evaluation]]'s enforcement arc
  (lu2026meta's separate-container holdout is exactly the design that
  BenchJack's worst flaw classes cannot touch).
- Actionable: run the Agent-Eval Checklist's isolation/scoring
  questions against any evaluator a downstream experiment proposes
  (the `/benchjack` skill packaging shows this fits a coding-agent
  harness).
