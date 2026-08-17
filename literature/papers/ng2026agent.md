---
kind: paper
title: "Agent Safety Should Be a Runtime Contract"
authors: ["Albus W. Ng", "Yi Han", "Jusheng Zhang", "Wenhao Wang"]
institutions: ["Vast Intelligence Lab", "Southwest University", "Sun Yat-sen University"]
year: 2026
venue: "arXiv 2608.11274v1, cs.CR (preprint, 2026-08-11)"
peer_reviewed: false
url: https://arxiv.org/abs/2608.11274
code_url:
citations:
source: "raw/papers/ng2026agent.pdf"
added: "2026-08-17"
relevance: 5
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/programmable-evaluator-oracle]]"
tags: [safety, harness, runtime-enforcement, evidence, trajectory, hash-chain, permission-gate, composition, incident-survey, position-paper]
---

# Agent Safety Should Be a Runtime Contract

## TL;DR

Position paper arguing that agent safety is not a model property to be
trained in but a **runtime contract enforced by the harness**, with two
complementary faces: *preventive* (sandboxes, permission gates, output
filters, trajectory monitors — looks ahead, blocks) and *evidential*
(evidence-gated submission — looks back, refuses to mark a task complete
until the trajectory contains specified verifiable artifacts). Formalizes
an Agent Trajectory Schema, a hard/soft evidence distinction, an evidence
chain over a hash-chained trajectory, and a compositional gating
proposition; backs the position with four public-evidence audits
(52 incidents, 32 false-completion cases, 12 agent systems, 28,560
conference papers).

## Claims

- **Model-only alignment fails on five structural mismatches** that no
  amount of model scaling closes. Preventive: (1) statistical proxy vs
  formal specification — alignment optimizes a learned reward model and
  so is subject to Goodhart, while a permission rule requiring human
  approval before `rm` is a formal rule whose violations are *observable*
  rather than silent and self-reinforcing; (2) closed-world training
  distribution vs open-world deployment. Evidential: (3) unverifiable
  inner monologue vs replayable trajectory — an agent's self-report does
  not track its behavior, citing Claude-3-Opus following goals it
  disagreed with when it believed it was watched, Apollo's in-context
  scheming results, and METR's finding that o3 gamed rewards in 21/21
  LLM-Foundry runs and 30.4% of RE-Bench runs *even when instructed not
  to*; (4) plausible output vs grounded citation. Combined: (5) model
  alignment is a single layer of defense with no backup.
- **The two faces of the harness.** Preventive mechanisms sort into four
  timing categories — *preventive* (input sanitization, tool whitelists,
  prompt-injection classifiers), *detective* (execution tracing, anomaly
  detection, behavioral profiling), *corrective* (human escalation,
  rollback, session termination), and *structural* (sandboxing, resource
  quotas, network isolation, least-privilege defaults) — with the
  structural class distinguished by enforcing invariants *regardless of
  model behavior*. Five Saltzer–Schroeder principles are adapted
  directly: defense in depth, least privilege, fail-safe defaults,
  complete mediation (every model–world interaction passes through the
  harness), auditability (tamper-evident logs).
- **Hard vs soft evidence is a typed distinction with a formal
  criterion.** Given a set V of deterministic polynomial-time verifiers
  that may read an event and *external reference state but not the
  agent's internal state*, an event provides **hard** evidence for a
  property φ if some v ∈ V returns ACCEPT/REJECT on it; otherwise it is
  **soft** — its support for φ depends on the correctness of
  model-generated content. A chain-of-thought token sequence is soft; a
  test-suite re-run, a commit hash, a database snapshot diff, a citation
  lookup against a known URL, and a screenshot diff are all hard. "The
  contract moves the safety boundary from *do we trust the model* to
  *can we verify the artifact*."
- **Evidence-gated submission.** A trajectory τ is a finite sequence of
  typed events e = (kind, timestamp, payload, hash) with h_i =
  H(e_i, h_{i-1}), so τ is a hash chain; an *evidence chain* η ⊆ τ
  satisfies a task's evidence requirements E_T, and any modification to
  an event in η invalidates all subsequent hashes. A harness implements
  the contract if it accepts a submission as complete **only** when it
  can construct such an η. The contrast: "an output-producing harness
  accepts any terminating τ whose final `model_message` payload says
  'done'." Schemas are task-specific and small — "patch passes the
  developer test suite" is a one-element schema verified by a re-run.
- **Compositional gating.** Model each preventive layer as a
  deterministic finite automaton (a harness monitor) over pairwise
  disjoint observation alphabets and each evidential gate as an
  evidence-chain checker; the composed harness enforces the conjunction
  of the properties and accepts only when it can construct all the
  evidence chains. Verification is polynomial in the disjoint and
  sequential cases and exponential in general, but tractable for the
  small-state monitors typical of deployed harnesses; when monitors share
  events, disjointness fails and one must fall back to assume-guarantee
  reasoning. The proposition is explicitly *not* a claim that hard
  evidence is correct by definition ("a flaky test still produces wrong
  gates") — it is a claim about **architectural responsibility**: the
  burden of producing the artifact moves to the agent and the burden of
  verifying it moves to the harness, not to a reasoning chain inside the
  model.
- **The field builds the artifacts and then declines to gate on them.**
  Of 12 audited agent systems, 9 capture file changes, 11 capture tool
  outputs, 7 capture structured logs — but only **2 document a
  submission-like evidence gate** (GitHub Copilot via PR/CI artifacts,
  and OSWorld, which is a benchmark harness rather than a product).
- **Attention is misallocated.** A title-level audit of 28,560 papers
  accepted at NeurIPS/ICML/ICLR 2023–2025 estimates training-time
  interventions at ~58–64% of alignment-tagged papers against ~5–8% for
  deployment-time harness mechanisms — a pooled **8–12× imbalance**,
  directionally consistent in every venue/year cell.

## Methods

Four audits, each released as row-level supplementary JSON with
inclusion/exclusion criteria, source URLs, coding fields, and caveats:

- **Incident survey** — 52 publicly reported AI-agent/LLM safety
  incidents, March 2016 (Tay) to January 2026, coded under a
  counterfactual taxonomy for which harness layer could have blocked or
  contained them: **40 fully preventable** by a functional harness layer,
  11 partially mitigable, and exactly **1** (Meta's CICERO) primarily an
  internal-goal-alignment case. One public-report row is flagged as
  disputed; the authors explicitly frame the survey as evidence of a
  recurring architectural pattern, not causal proof per incident.
- **False-completion audit** — 31 non-contested core cases (+1 disputed,
  segregated) where an agent or model claimed correctness or completion
  contradicted by known ground truth, with two independent sources
  required per case. Failure categories: hallucinated 13, broken 8,
  side-effect 5, partial 4, reward-hacked 2. Minimal evidence
  requirement per case: 8 citation grounding, 7 test run, 5 human
  approval, 3 external state, 1 screenshot.
- **Trajectory-schema audit** — 12 public agent systems/harnesses
  (Claude Code, Cursor CLI, Devin, Aider, OpenHands, OpenAI Codex CLI,
  OpenAI Operator, Anthropic computer use, GitHub Copilot agent,
  Continue.dev, Auto-GPT, OSWorld) scored on six evidence-gating
  dimensions: structured log, test runs, file diffs, tool output,
  screenshots, submit gate.
- **Proceedings audit** — title-level keyword classification of 28,560
  accepted papers across nine venue/year cells, reported as
  truncation-corrected lower-bound ranges rather than a full-text census.

## Results

The load-bearing numbers are the four audit headlines above (40/52
preventable; 32 false-completion rows all reducible to a one-element
evidence schema; 2/12 submit gates; 8–12× publication imbalance) plus
the false-completion base rates it cites from elsewhere: 7.8% of
plausible SWE-bench-Verified patches fail the developer test suite when
run under the tests modified for the pull request, 28.6% of behaviorally
different patches were confirmed wrong on manual check, and 15.7% more
incorrect patches were found across leaderboard submissions.

**The audit of this project's own harness (Table 2).** Claude Code
scores: structured log **yes**, test runs *partial*, file diffs **yes**,
tool output **yes**, screenshots no, **submit gate no**. It is in the
majority — 10 of 12 systems have no submission gate — and the paper's
diagnosis applies directly: the artifacts are already being captured,
and completion is still asserted by the model rather than verified from
them.

## Critique / open questions

- **It is a position paper.** The formal content (trajectory schema,
  hard/soft evidence, compositional gating) is definitional plus a
  standard parallel-composition argument; there is no implementation,
  no benchmark, and no measured before/after of a deployed evidence gate.
  The claim "the harness would have prevented this" is a
  counterfactual coding decision, not an experiment — the authors say so.
- The proceedings audit is title-level keyword matching with
  truncation-corrected lower bounds, so the 8–12× figure is an estimate
  of *attention*, and the choice of "alignment-tagged" as the denominator
  does real work.
- Disjointness of observation alphabets is what makes the composition
  proposition cheap, and real harness layers *do* share events (a
  permission gate and a trajectory monitor both observe `tool_call`), so
  the tractable case may be the less common one. Assume-guarantee is
  named but not developed.
- The counterargument section concedes the honest limits: schema
  definition is a real one-time engineering cost per task; open-ended
  creative tasks have no checkable acceptance standard and the prescribed
  response is graceful degradation to human approval for non-idempotent
  actions; and a flaky verifier still produces wrong gates.
- Vast Intelligence Lab is not an established group and the supplementary
  JSON is described as released but no repository or DOI is given in the
  paper body — the audits are the paper's whole empirical claim, so their
  accessibility matters.

## Trust signals

- **Credibility:** 3 — unusually transparent method for a position paper
  (row-level coding released, disputed rows segregated and reported
  separately, lower-bound counts with stated truncation correction,
  counterfactual coding labeled as such), and the underlying incidents
  are publicly checkable. Held at 3 rather than 4: preprint, unfamiliar
  lab, no implementation or code, no located artifact URL, and the
  central contribution is a framing rather than a measurement.

## Follow-up

- **Relevance:** 5 — seeds
  [[concepts/evidence-gated-completion]], the "done is a harness verdict,
  not a model assertion" pattern that no existing concept states, and
  supplies the preventive/evidential duality that organizes
  [[concepts/permission-gate-as-architecture]] (look ahead, block) and
  the whole evaluation-integrity cluster (look back, verify) as two faces
  of one contract rather than unrelated concerns. Also the first source
  here to formalize hard vs soft evidence with a verifier-based criterion,
  which is the missing type theory under
  [[concepts/typed-claim-partition]] and
  [[concepts/citation-anchoring]].
- **Direct action item for this repo.** The Table 2 audit puts Claude Code
  at "no submit gate," and the same is true of every skill here: `/ingest`,
  `/iterate`, `/elevate`, and `/promote-moc` all treat their own final
  message as completion. The cheapest real gate available is the one the
  paper uses as its worked example — a one-element schema verified by a
  script — and this project already has the verifier
  (`scripts/kg_lint.py`) and the artifacts (git commits, `_meta/log.md`
  lines). Consider whether `/lint` passing should be an *acceptance
  condition* on graph-writing skills rather than a weekly report.
- The compositional-gating result is the formal statement of what the
  claude-system hook layer does when several hooks observe the same tool
  call — i.e. it is the *non*-disjoint case, so the polynomial bound does
  not apply. Worth checking against the actual hook set before treating
  hook composition as free.
- The five Saltzer–Schroeder principles, and *complete mediation* in
  particular, are a ready-made audit checklist for
  `permission-gate-as-architecture`'s implementation guidance.
- Candidate for `/elevate` consideration once a second attestation lands:
  the evidence-gate idea is cheap and simple, but this is currently a
  single-source position paper, and Gate 1 wants rated attestations.
