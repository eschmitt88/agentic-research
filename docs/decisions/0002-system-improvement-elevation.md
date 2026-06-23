# 0002 — System-improvement elevation (meta repo → claude-system)

- **Date:** 2026-06-23
- **Status:** accepted

## Context

This repo is research-about-research: it curates literature and concepts on
how autonomous research agents are built. Some of that is directly about the
kind of system this box runs — the skills, hooks, and rules in
`~/claude-system`. There was no mechanism to periodically ask "should any of
these curated, well-evidenced ideas be adopted into the harness itself?",
so good ideas could sit in `concepts/` indefinitely without ever being
considered for the system.

## Decision

Add a `/elevate` skill (`~/claude-system/claude/skills/elevate/SKILL.md`)
that reads the knowledge graph, compares it against the live
`~/claude-system`, and writes **proposals for human review** under
`docs/system-proposals/`. Policy, per the user's directive:

- **Human review is the gate.** The skill never edits `~/claude-system` and
  never applies a change. It only writes proposal files in this repo. A
  human accepts (and makes the edit) or rejects.
- **Two acceptance gates, both required.** (1) *Reputable evidence* —
  peer-reviewed OR code-released OR ≥3 independent attestations, with
  supporting credibility ≥3. (2) *Simplicity* — strongly prefer changes
  that remove/consolidate/clarify; net-new surface area (a new skill/hook/
  rule/knob) must clear a higher bar and justify why a simpler form won't
  do. Default to rejecting complexity.
- **Restraint is normal.** Most cycles produce zero proposals; that is the
  healthy outcome, logged rather than padded with weak proposals.
- **Cadence.** Weekly cron, Sundays 05:00
  (`~/.claude/schedule/agentic-research-elevate.sh`) — after the 04:00
  nightly `/curate` (graph freshly drained) and before Monday 07:00
  `/digest`. Can also be run by hand with `/elevate`.
- **Location.** Proposals live in this meta repo (tracked alongside the
  concepts that justify them), keeping `~/claude-system` clean — it changes
  only when a human accepts a proposal.

## Consequences

- Curated findings now have a path to influence the harness, but only
  through an explicit human-reviewed step — no autonomous self-modification
  of the system.
- The `/elevate` skill is itself a change to `~/claude-system`; it was added
  under direct user instruction (not via its own review queue), which is the
  authorized exception. Future system changes go through the queue.
- Adds one (usually cheap / zero-proposal) `claude -p` invocation per week.
- Alternative considered: let proposals land in `~/claude-system` itself.
  Rejected — that adds churn to the core infra repo even for rejected ideas;
  keeping them here isolates review from the code they target.
