---
kind: post
title: "OpenClaw ACP Harness Explained — Running Coding Agents Inside an Orchestrator"
author: "Hex (OpenClaw Playbook)"
institutions: []
year: 2026
venue: "OpenClaw Playbook"
peer_reviewed: false
url: "https://www.openclawplaybook.ai/guides/openclaw-acp-harness-explained/"
code_url:
citations: null
source: "raw/_candidates/2026-05-15-agent-harnesses.md"
added: "2026-06-03"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - hierarchical-delegation
tags:
  - harness
  - nested-harness
  - acp
  - openclaw
  - orchestration
---

# OpenClaw ACP Harness Explained — Running Coding Agents Inside an Orchestrator

## TL;DR

Walkthrough of OpenClaw's Agent Client Protocol (ACP), a mechanism that
lets an outer orchestrator agent nest external coding agents (Claude Code,
Codex, OpenCode, Gemini CLI, Pi) as child harnesses, hand each a coding
task brief, and receive results programmatically without manual session
juggling. The nested-harness pattern is novel for this project and maps
directly onto `[[hierarchical-delegation]]`: a curator-style outer agent
could dispatch ML experiments to a Claude-Code-shaped inner harness rather
than spawning raw subagents.

## Claims

- The orchestrator calls `sessions_spawn()` with `runtime: "acp"`, an
  `agentId`, a `task` brief, an explicit `model`, and `timeoutSeconds`;
  the child agent then runs autonomously — writing files, running tests,
  creating commits.
- Results "auto-announce back to the main agent" without polling; the
  orchestrator synthesizes outcomes and reports.
- Supports single and parallel sub-agent sessions; `thread: true` enables
  a thread-bound mode for interactive Slack/Discord guidance.
- Effective task briefs must specify exact files, line numbers, and
  surgical modifications to avoid the child agent's exploration overhead.

## Trust signals

- **Credibility:** 3 — a vendor-adjacent playbook guide (not the official
  SDK reference), but concrete and interface-level rather than a
  speculative hot-take. Single-author, non-peer-reviewed, so capped at 3.

## Follow-up

- Decide whether ACP is a credible interface for the outer-curator /
  inner-implementer dispatch this project gestures at under
  `[[hierarchical-delegation]]`, vs. the raw-subagent approach currently
  assumed.
- Contrast `sessions_spawn()`'s explicit-model + task-brief contract with
  the project's own implement/iterate subagent boundary.
