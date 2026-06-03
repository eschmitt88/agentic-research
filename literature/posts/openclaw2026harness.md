---
kind: post
title: "Agent Harness Plugins — OpenClaw SDK Reference"
author: "OpenClaw / Anthropic"
institutions: ["Anthropic"]
year: 2026
venue: "OpenClaw SDK docs"
peer_reviewed: false
url: "https://docs.openclaw.ai/plugins/sdk-agent-harness"
code_url:
citations: null
source: "raw/_candidates/2026-05-15-agent-harnesses.md"
added: "2026-06-03"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - hierarchical-delegation
tags:
  - harness
  - separation-of-concerns
  - openclaw
  - orchestration
---

# Agent Harness Plugins — OpenClaw SDK Reference

## TL;DR

Official OpenClaw SDK reference that defines a "harness" as the low-level
executor of a single prepared agent turn, cleanly factored from the
orchestrator that owns provider/model selection, auth, thinking levels,
and context budgets. The cleanest formal separation-of-concerns statement
in the current harness literature — maps almost 1:1 onto this project's
curator/executor (ideator/implementer) split.

## Claims

- A harness executes *one prepared attempt* and nothing more; the
  OpenClaw core retains control of provider/model resolution, auth,
  thinking levels, context budgeting, and the canonical session
  transcript.
- The `runtimePlan` interface exposes shared host policies (tool
  normalization, transcript sanitization, delivery-suppression rules,
  run-outcome classification) as "host-owned attempt state" the harness
  may read but must not mutate.
- Harnesses must satisfy a **transcript-mirroring contract**: keep
  user-visible assistant/tool output synchronized with the OpenClaw
  transcript so sessions stay portable, searchable, and fallback-able to
  the embedded runtime.
- `reset(...)` clears sidecar bindings when an OpenClaw session resets,
  keeping native harness state consistent with the core transcript.

## Trust signals

- **Credibility:** 4 — primary/official SDK reference (OpenClaw docs,
  published under Anthropic), not a third-party hot-take. Authoritative
  on its own interface; not peer-reviewed and vendor-specific, hence not
  a 5.

## Follow-up

- Compare the host-owned-`runtimePlan` boundary against this project's
  `[[hierarchical-delegation]]` framing: who owns model/budget choice vs
  who executes the turn.
- The transcript-mirroring contract is a concrete instance of treating
  the transcript as the shared substrate — cross-check against
  `[[file-as-bus]]` if the transcript is file-backed.
