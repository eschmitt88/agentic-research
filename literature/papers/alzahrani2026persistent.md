---
kind: paper
title: "Persistent AI Agents in Academic Research: A Single-Investigator Implementation Case Study"
authors: ["Anas H. Alzahrani"]
institutions: ["King Abdulaziz University (Dept. of Preventive Medicine and Public Health, Faculty of Medicine)"]
year: 2026
venue: "arXiv preprint (cs.MA)"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.26870"
code_url: null
citations: null
source: "raw/papers/alzahrani2026persistent.pdf"
added: "2026-07-20"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/file-as-bus]]"
  - "[[concepts/agent-native-memory]]"
tags: [case-study, persistent-agents, memory, telemetry, token-economics, governance, human-agent-environment]
---

# Persistent AI Agents in Academic Research: A Single-Investigator Implementation Case Study

## TL;DR

A 115-day (Jan 31–May 25 2026) self-observed case study of one academic
physician-scientist running a persistent, multi-role agent environment
(Discord interface, durable memory files, scheduled jobs, 17 agent
directories, 57 skill files); introduces PARE-M, a descriptive
measurement framework, and reports a cache-dominant token profile
(82.9% cache-reads in May) as evidence that persistent agentic work
shifts the economic unit from cost-per-token toward
cost-per-completed-artifact.

## Claims

- Persistent, real-world agent deployments are a different object of
  study than benchmark/episodic evaluation; the unit of analysis should
  be the "human-agent environment" (researcher + agent runtime + memory
  + tools + repos + scheduled jobs + governance rules), not the model
  in isolation — episodic benchmarks (HELM, SWE-bench, AgentBench, GAIA,
  AI-Scientist-style evals) "do not directly measure whether a
  persistent agentic environment improves a researcher's longitudinal
  workflow, governance, reproducibility, or cost per artifact."
- PARE-M v0.1 (Persistent Agentic Research Environment Measurement)
  frames five metric domains — utilization, output, resource,
  reproducibility, governance — and requires every metric to declare a
  numerator, denominator, time window, and computation rule (e.g.
  Active-Day Fraction, De-duplicated Record Count, Active-Time
  Estimate, Cache-Dominance Ratio, Output-Proxy Rate, Governance-Event
  Rate, Artifact-Surface Breadth).
- Cache-Dominance Ratio (CDR): 82.9% of May 2026 recorded tokens were
  cache-reads (61,278,669 of 73,950,305), vs. 10,697,394 input,
  754,633 output, 1,219,609 cache-write. The authors read this as
  evidence that a mature persistent workflow depends on reused context
  rather than fresh inference, and that "cost per token" becomes a
  poor value proxy once caching dominates — future evaluation should
  price cost-per-artifact, cost-per-verified-action, cost-per-safe-
  external-workflow instead.
- Governance is not separable from capability in a persistent
  environment: the same memory layer that preserves task context also
  preserves safety rules, prior mistakes, citation requirements, and
  deployment/external-action checks. 889 failure/verification/
  correction/protocol-proxy events were recorded (Governance-Event
  Rate = 9.26/active day) — the governance layer is part of the
  operating environment, not an after-the-fact policy appendix.
- Aggregate interaction volume did not shrink as the system
  accumulated memory and procedures; scope of delegated work expanded
  instead. The authors explicitly frame the observed pattern as
  *capacity expansion*, not proven labor substitution, and refuse to
  claim causal productivity effects anywhere in the paper.

## Methods

- Structured self-observed implementation case study, Jan 31–May 25
  2026 (115-day window; recoverable telemetry starts Feb 2).
- Recoverable main-agent telemetry: 75,671 de-duplicated records across
  96 active days (8,059 user-role messages, 23,710 assistant-role
  messages, 18,596 tool-result messages, 2,385 tool-call events, 1,286
  model-completed events).
- Workspace inventory: 502 memory-related files, 17 configured agent
  directories, 57 skill files, plus session/JSONL-like file counts
  (4,309 main-session files; 3,194/4,388 recoverable main/all-agent
  JSONL-like files).
- Active-time estimation via a capped-gap algorithm over unique event
  timestamps (30-min cap → 579.7h; 60-min cap sensitivity → 674.1h) —
  explicitly labeled a system-activity estimate, not direct human labor
  time.
- Output-proxy events (482) and governance/correction events (889)
  extracted from memory and lesson files via keyword/rule-based
  coding — author-coded, and the paper itself flags this as requiring
  independent validation before being treated as a definitive rate.
- A separate, narrower token-telemetry subset (May 1–25 only: 627
  model-completed events, 73,950,305 tokens) is analyzed by category
  (input/output/cache-read/cache-write) across OpenAI Codex-route,
  OpenAI-route, and Anthropic-route usage.
- Reproducibility section: schema, parsing rules, de-duplication logic,
  active-time algorithm, and output-event extraction logic are declared
  reproducible in principle; raw conversations, credentials, and
  private project logs are withheld for privacy. A de-identified event
  ledger and parsing scripts are stated as "in preparation" for
  preprint release — not available at time of this ingest.

## Results

- Active-Day Fraction = 0.835 (96/115 days).
- De-duplicated Record Count = 75,671.
- Output-Proxy Rate = 5.02 events/active day (482 total, spanning 10
  artifact-surface categories: manuscripts, teaching artifacts, content
  drafts, scripts, operational documents, software repositories,
  calibration-study materials, deployed research tools).
- Governance-Event Rate = 9.26 events/active day (889 total).
- Cache-Dominance Ratio = 82.9% (May subset).
- Artifact-Surface Breadth = 10 distinct categories.
- ~US$1,961 in preliminary observed direct spend; invoice-level
  reconciliation explicitly incomplete.
- Environment ran on a DigitalOcean VPS with a verified, checksummed
  compressed local backup — described as recoverability infrastructure,
  not verified disaster-recovery maturity (no offsite schedule or
  restore drill confirmed).

## Critique / open questions

- The authors are unusually candid about the design's limits: single
  investigator who is simultaneously user, system designer, data
  source, analyst, and beneficiary; no control group; no matched
  non-agent workflow; no pre-registered output register; no baseline
  productivity period; no independent coder for the governance-event
  taxonomy. They explicitly disclaim causal productivity claims — a
  discipline this note preserves rather than upgrading their language.
- Artifact-surface counts are a *positive* inventory (what was
  completed), not a denominator of attempted work — abandoned, failed,
  or never-started efforts are not counted, so Output-Proxy Rate cannot
  be read as a completion rate.
- File counts are explicitly flagged as vulnerable to inflation from
  generated/software artifacts (projects, static sites, build
  products are "not commensurable" per the authors).
- Token telemetry covers only a 25-day window inside the 115-day case;
  the 82.9% cache-dominance figure should not be extrapolated to the
  full period.
- No code or data is released yet (in preparation), and no peer review
  — this is a preprint self-report, not an independently verified
  audit, and the governance-event counts in particular are author-coded
  without a second rater.
- What it's worth despite all that: this is one of very few published
  *in-vivo* accounts of a persistent, multi-role agent deployment
  running for months against real academic work, rather than a
  benchmark ablation or an architecture description. Most of this
  project's evidence for durable-memory and multi-role-workspace
  patterns comes from curated benchmarks (PaperBench, MLE-Bench,
  LoCoMo) or system papers describing an architecture's design; this
  paper instead reports what accumulated when someone actually ran the
  pattern for four months and counted what came out. That is exactly
  the trade a self-observed n=1 case makes: rich ecological detail
  (502 memory files, 17 role directories, 57 skill files, a governance
  layer that grew organically) in exchange for zero counterfactual
  control.

## Trust signals

- **Credibility:** 2 — single author outside CS/ML (a preventive-
  medicine/public-health physician-scientist), arXiv-only cs.MA
  preprint, not peer-reviewed, no code or data released yet ("in
  preparation"), no citations. Partially offsetting: unusually
  transparent methodology for a self-report (explicit operational
  definitions, numerator/denominator/window/rule for every metric, a
  dedicated Limitations section that pre-empts the obvious
  objections) — rigor above what the credibility signals alone would
  predict, but institutional/independent-verification signals remain
  weak.

## Follow-up

- **Relevance:** 4 — the first naturalistic (non-benchmark) attestation
  of [[concepts/file-as-bus]] and [[concepts/agent-native-memory]]
  operating together in an actual multi-month deployment: durable
  memory files as the system of record (502 files), specialized agent
  roles as durable-state substrate (17 directories), scheduled jobs,
  and a skill library (57 files) as reusable procedure store — the
  same shape this project itself uses, observed in someone else's
  environment rather than asserted architecturally.
- Also surfaces a token-economics observation (cache-dominance → the
  claim that the correct evaluation unit shifts from cost-per-token to
  cost-per-artifact) that neither existing concept currently models
  structurally. Left as an open-question note on
  [[concepts/agent-native-memory]] rather than promoted to its own
  concept file — it's a single data point from one 25-day window, not
  yet an atomic idea with independent corroboration.
- Watch for: the de-identified event ledger and parsing scripts the
  authors say are in preparation — if released, they would let a
  future ingest check the PARE-M computation rules directly rather
  than taking the aggregate numbers on trust.
