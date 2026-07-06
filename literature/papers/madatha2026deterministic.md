---
kind: paper
title: "A Deterministic Control Plane for LLM Coding Agents"
authors: ["Padmaraj Madatha"]
institutions: ["Happiest Minds Technologies (AIP Centre of Excellence)"]
year: 2026
venue: "arXiv preprint (2606.26924)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.26924"
code_url: "https://doi.org/10.5281/zenodo.20780913"
citations: null
source: "raw/papers/madatha2026deterministic.pdf"
added: "2026-07-06"
relevance: 4
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [safety, governance, permission-gate, determinism, supply-chain, coding-agents, config-layer]
---

# A Deterministic Control Plane for LLM Coding Agents

## TL;DR

A prevalence study of 6,145 agent-configuration files across 10,008 GitHub
repositories finds that agent definitions (`CLAUDE.md`, `.cursor/rules`,
`AGENTS.md`) propagate as **undeclared shared components** — 10.1% of
tracked paths are exact cross-repo duplicates (75.5% crossing org
boundaries), configs are rarely revised, and <1% declare permission
boundaries. The paper argues the config-and-process layer around a coding
agent should be treated as a **managed software supply chain governed
deterministically** — hashing, state machines, blocklists in ordinary
testable code — and explicitly **not delegated to further LLM
orchestration**, because a non-deterministic component cannot be a
trustworthy control for another non-deterministic component.

## Claims

- **The governance surface is thin.** Modern coding harnesses pair a capable
  execution loop (indexing, MCP tools, file/shell access) with policy
  expressed only as natural-language config files injected into context —
  low-friction to author, unmanaged at enterprise scale.
- **Three structural gaps**: (1) configuration is unmanaged software
  (copy-pasted, no provenance, trusted without a "this is the approved,
  untampered version"); (2) execution is unbounded (no
  requirement→implementation→test trace, no hard cap on self-correction
  loops); (3) expertise does not port across tools' config dialects.
- **The determinism thesis** (load-bearing): governance of this layer must
  be deterministic and tool-agnostic. Every proposed mechanism is ordinary
  testable code (SHA-256 content addressing, HMAC-stamped lockfiles,
  hash-chained audit logs, regex blocklists, a phase state machine, set
  arithmetic) — never an LLM. *"A non-deterministic component cannot serve
  as a trustworthy control for another non-deterministic component."*
- Honest scope limit: the deterministic guarantee covers **pre-execution**
  controls (install-time gates, blocklists, phase transitions); the
  requirement→file→test trace is *cooperative and audited post-hoc*, not
  pre-execution enforced. Developer-outcome benefits remain future work.

## Methods

- Prevalence study (§7): 10,008 public GitHub repos, n=6,145 agent config
  files. Exact-duplicate propagation measured by SHA-256 (fork-adjusted,
  threshold-independent); permission-boundary declaration parsed (fragile
  parser, n=31 true positives); revision depth vs CI/CD workflows,
  age-normalized.
- Reference implementation **Rel(AI)Build**: content-addressed agent
  definitions, tiered permissions enforced before LLM invocation via IDE
  runtime hooks (Claude Code `PreToolUse`, Cursor `beforeShellExecution`),
  a post-delegation `scan-diff` gate, Jaccard-similarity prompt-drift
  detection, a phase-gated lifecycle blocking execution on invariant
  violation, and a define-once/compile-to-seven-IDE-targets transformer.
- Conformance tests (§6) on injected violations (N=10/15/20) confirm each
  mechanism enforces its stated invariant. Dataset + reproduction scripts
  released on Zenodo.

## Results

- Robust headline: 10.1% fork-adjusted exact-duplicate propagation, 75.5%
  cross-organizational; two weaker indicative proxies — 58% single-commit
  shallow version-control depth, and <1% of agent configs declaring
  permission boundaries vs 33% of GitHub Actions workflows.
- Conformance tests pass; the architecture is demonstrated to enforce its
  invariants but its *effect on developer outcomes is not measured*.

## Critique / open questions

- Single author, industry affiliation, no lab/venue peer review; the
  architecture's real-world benefit is explicitly deferred ("developer
  outcomes remain future work"). Strength is the prevalence dataset, not a
  validated system.
- The permission-boundary prevalence (<1%) rests on a "fragile parser" with
  only 31 true positives — the author flags this as indicative, not
  definitive.
- Determinism is only pre-execution; the traceability guarantee still
  depends on the agent cooperatively calling `trace-update`, so the hardest
  part (bounding execution) is not actually LLM-independent.

## Trust signals

- **Credibility:** 2 — a substantial 45-page empirical study with a released
  Zenodo dataset (10,008 repos) and conformance tests, but a single
  industry author, no peer review, a self-flagged fragile parser on the
  key permission stat, and an unvalidated reference architecture.

## Follow-up

- **Relevance:** 4 — adds the **determinism axis** that
  [[concepts/permission-gate-as-architecture]] lacked: the gate as ordinary
  testable code rather than an LLM judge governing another LLM, plus
  empirical grounding (<1% of real agent configs declare permission
  boundaries) for why the gate must be designed as architecture. Directly
  about this project's own substrate — `CLAUDE.md` / rules files and Claude
  Code `PreToolUse` hooks are the studied surface.
- Contrast with jia2026finharness and AgenticOS-style LLM-judge governors:
  this paper is the counter-argument that the governor itself should be
  deterministic.
