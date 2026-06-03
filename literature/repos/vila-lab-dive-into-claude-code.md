---
kind: repo
name: "Dive-into-Claude-Code (VILA-Lab)"
url: https://github.com/VILA-Lab/Dive-into-Claude-Code
commit: HEAD (README @ 2026-06-03; companion to arXiv:2604.14228, paper v1 2026-04-14)
source: "raw/repos/vila-lab-dive-into-claude-code.md"
added: "2026-06-03"
relevance: 5
status: scanned
related_experiments: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [agent-harness, claude-code, reverse-engineering, design-guide, resource-index, mbzuai]
---

# Dive-into-Claude-Code (VILA-Lab)

## Purpose

Companion artifact to the paper `liu2026dive` (arXiv:2604.14228). Where the paper is
the analysis, this repo is the *extended, maintained* deliverable: a source-level
reverse-engineering of Claude Code v2.1.88 (~1,900 TypeScript files, ~512K lines)
plus a "build your own AI agent" design guide and a large curated bibliography of
the agent-harness ecosystem. Ingested as a repo (not just a paper mirror) because it
contains material beyond the PDF — a builder-facing decision guide and a ~100+ entry
resource index that the paper does not reproduce.

## Shape

- **Architecture documentation** — markdown across 7 safety layers, the seven-mode
  permission system (+ ML classifier), the five-layer context-management/compaction
  pipeline, four extensibility mechanisms (MCP/plugins/skills/hooks), subagent
  delegation with isolation, append-oriented session storage.
- **Design-principles framework** — five human values traced through thirteen
  implementation principles.
- **Comparative analysis** — Claude Code vs. OpenClaw vs. Hermes-Agent across six
  design dimensions, source-grounded.
- **"Build Your Own AI Agent" guide** — six critical design decisions.
- **Resource index** — 40+ open-source reimplementations, 25+ technical blog
  analyses (~100+ referenced projects/papers total).

## Useful bits

- **Headline quantitative claim:** 98.4% of the codebase is deterministic
  infrastructure, only 1.6% AI decision logic — the hard datum grounding this
  project's working assumption that harness engineering, not model choice, is the
  locus of agent capability.
- **Permission system as a first-class component** (seven modes + ML classifier) —
  the load-bearing attestation for permission-gate-as-architecture, alongside the
  AHI-security survey `wang2026reframing`, OpenHarness, and Hermes.
- **Five-layer compaction pipeline** — a concrete reference implementation to anchor
  `context-eviction-policy`.
- **Resource index as prior-art map** — the 40+ reimplementations list overlaps the
  harness-architecture cluster's sources (OpenHarness, Hermes, claw-code) and is a
  cross-check for this project's concept-index coverage.

## Follow-up

**Relevance:** 5. The most architecturally substantive single source the
harness-architecture cluster has on Claude Code itself, and the likely trigger
(with `ning2026code` and `wang2026reframing`) for promoting the cluster to a MoC.

- Read the "Build Your Own AI Agent" six-decision guide against this framework's own
  harness choices (permission gates, compaction, subagent delegation).
- The comparative six-dimension table (vs. OpenClaw, Hermes) is the cleanest
  cross-harness comparison yet — reconcile with the OpenHarness and Hermes repo notes.
- License is CC BY-NC-SA 4.0 — note the non-commercial clause if any content is reused.
