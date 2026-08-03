---
kind: paper
title: "Glite ARF: Verifier-Driven Research with Parallel LLM Coding Agents"
authors: ["Vassili Philippov", "Pavel Katunin", "Dmitry Andreev", "Igor Ostanin", "Anton Nikolaev"]
institutions: ["Glite", "University of Sheffield"]
year: 2026
venue: arXiv (2606.27416)
peer_reviewed: false
url: https://arxiv.org/abs/2606.27416
code_url: https://github.com/GliteTech/glite-arf
citations: null
source: "raw/papers/philippov2026glite.pdf"
added: "2026-08-03"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/citation-anchoring]]"
tags: [research-campaign, verifiers, provenance, parallel-agents, worktrees, reproducibility, leakage, framework]
---

# Glite ARF: Verifier-Driven Research with Parallel LLM Coding Agents

## TL;DR

An open-source framework for multi-week research campaigns run by
parallel LLM coding agents, built on **verifier-driven research**:
the rules of the research process live in deterministic Python
scripts that fail loudly when violated, not in prose agents are
asked to follow. External anchor: 1st place (closed track, all three
languages) at the BEA 2026 shared task, 273 tracked tasks, up to 12
parallel agents on one laptop, ~$498 total cost, with per-fold
provenance catching four target-leaking feature sets.

## Claims

- Naive delegation doesn't scale: prompt-encoded rules keep a
  low-single-digit failure rate "across thousands of agent
  invocations," which compounds into dozens of broken artefacts over
  a multi-week campaign — "the natural response is to write better
  prompts. We tried." The fix is structural, not rhetorical.
- Three-role stack: the human chooses hypotheses (role 1), coding
  agents implement isolated tasks (role 2), deterministic verifiers
  enforce structure (role 3). Role 1 stays human *by design*
  (METR time-horizon argument; Beel et al.'s 42% failed AI-Scientist
  experiments), not as an interim measure.
- Seven principles, each crystallized from a logged incident: task
  isolation (folder + git worktree + branch; pre-merge verifier
  rejects out-of-folder writes — motivated by one step recomputing
  20,304 historical rows across 38 feature sets), immutability with
  a corrections overlay, aggregators-only cross-task reading,
  materialized human-facing overview, spec-verified artefacts
  (versioned specification + verifier per artefact kind),
  comprehensive command logging (a step cannot complete if its log
  is missing), and per-stage subagent isolation against context
  degradation.
- Verifiers judge structure, never semantics: experiment design
  quality stays with the human; the framework makes structural
  failures detectable and auditable.

## Methods

- Every artefact conforms to a versioned spec under
  `arf/specifications/`; verifier scripts emit stable error codes
  (e.g. PM-E003: task branch modified a file outside its folder) that
  block merge; warnings don't. Verifiers are deliberately redundant.
- Tasks are folders + branches + PRs; each runs a nine-step lifecycle
  (papers → internet → code → planning → implementation → analysis →
  literature comparison → reporting → merge), each step in its own
  subagent with scoped context.
- Corrections overlay: completed task folders are immutable;
  downstream fixes are correction files in the *fixing* task's
  folder, applied at read time by aggregators — original records
  never rewritten.
- Campaigns mined for evidence: BEA 2026 (247 tasks completed),
  word-sense disambiguation (167 tasks, 74 days, peak concurrency
  12), Ace-CEFR readability (public demo repo) — identical framework
  code across all three.

## Results

- BEA 2026: 1st closed track on ES/DE/CN (avg RMSE 0.855 vs 1.218
  baseline, −29.9%), 2nd open track (−35.9%); $449.69 LLM API spend,
  $498.31 total, ~100 local wall-hours.
- **The leakage catch**: an intermediate ensemble scored an
  implausible 0.609 K-fold RMSE; because every fold-level score was
  traceable to the code revision that produced it, four
  target-leaking feature sets were localized "within minutes,"
  quarantined via the corrections overlay, and the rebuilt ensemble
  scored 0.802. The structural layer cannot judge *semantic* leakage
  — that call was human — but provenance turned a silent
  contaminated submission into an auditable fix.
- Compliance at scale: per-column metadata let a mid-campaign rule
  clarification trigger a fine-grained re-audit (56% of feature sets
  rejected for the closed track, full pipeline rebuild in the final
  week, 1st place retained).
- Overhead measured: framework machinery is ~25% of logged commands
  but ~1% of wall-clock; across 305 merges in two mined campaigns,
  no task corrupted another's data and no merge conflict reached
  main.

## Critique / open questions

- **Verifiers are not agent-proof** (stated plainly): they are
  ordinary scripts in the repository — an agent instructed to bypass
  a rule can rewrite the verifier that checks it. ARF targets
  accidental drift, not adversarial agents. This is the same
  write-path/authority gap as constraint pinning's
  operator-impersonation residual: enforcement code that lives
  inside the agent-writable universe is only as strong as the
  agent's incentive not to touch it.
- The headline result is one shared task; the two supporting
  campaigns are author-run and one is author-held (logs not public).
- Single-operator assumption; no multi-human concurrency.
- Verifier detection rates are unmeasured (controlled incident-
  seeding study is named as future work) — currently the evidence is
  incidents caught, not a catch *rate*.

## Trust signals

- **Credibility:** 4 — small company (Glite) + University of
  Sheffield, not peer-reviewed, but the empirical anchor is a
  *refereed external shared task* whose labels and leaderboard the
  authors don't control — rare in autoresearch papers, most of which
  self-evaluate; framework + demo project released (Apache 2.0),
  campaign logs partially public.

## Follow-up

- **Relevance:** 5 — the first campaign-scale deployment of
  [[concepts/typed-enforcement]]'s thesis on the research *process*
  itself, with measured cost (~1% wall-clock) and an external
  result; its honest limit (in-repo verifiers are rewritable)
  sharpens the concept's escape-hatch section. Directly validates
  several pillars of this project's own architecture: worktree task
  isolation, file-based coordination with a materialized overview
  ([[concepts/file-as-bus]]), task-grain parallel agents
  ([[concepts/async-worker-pool]]), human-on-hypothesis-selection
  ([[concepts/hierarchical-delegation]] and the agency.md
  Confirmation principle), and score-to-revision provenance
  ([[concepts/citation-anchoring]], [[concepts/hce-evaluation]]).
- The corrections-overlay (immutable records + read-time overlay,
  "each datum has a single home") is a concrete mechanism this
  project's `raw/`-immutability + re-ingest rule approximates;
  worth citing if that discipline is ever formalized.
- Watch the promised incident-seeding study — a measured verifier
  catch rate would move verifier-driven research from
  well-engineered practice to quantified method.
