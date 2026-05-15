---
kind: repo
name: "Hermes Agent"
url: https://github.com/nousresearch/hermes-agent
commit: HEAD (README @ 2026-05-15)
source: "raw/repos/nousresearch-hermes-agent.md"
added: "2026-05-15"
relevance: 4
status: scanned
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/scripted-tool-pipelines]]"
tags: [agent-harness, persistent-agent, skills, memory, multi-platform, open-source, nous-research]
---

# Hermes Agent

## Purpose

Hermes Agent is Nous Research's open-source long-running personal-agent
harness, shipped February 2026 and at v0.9.0 by mid-April 2026. It
positions itself as "the self-improving agent that grows with you" —
optimized for *persistence across sessions* rather than the bounded
coding-session shape of Claude Code. The harness wraps any LLM (200+
models via OpenRouter, plus first-party support for Nous Portal,
NovitaAI, NVIDIA NIM, Anthropic, OpenAI, local) and adds the
session/state/memory/skill/permission scaffolding the model itself
doesn't carry.

## Shape

The README does not expose a module tree, but the documentation index
plus the surfaced feature list imply the following subsystems:

- **Agent loop**: streaming TUI on top of the model's tool-call API;
  the architecture docs (linked but unfetched here) reference a
  `QueryEngine`-equivalent and an `AIAgent` class — see
  Huang's comparison summary in `raw/_candidates/2026-05-15-agent-harnesses.md`.
- **Memory**: agent-curated, with periodic *nudges* prompting the
  model to write to memory; FTS5 (SQLite full-text) session index;
  LLM summarization for cross-session recall; Honcho-based dialectic
  user modeling (the Plastic Labs library) for "build a model of who
  you are".
- **Skills**: autonomous creation after complex tasks; skills
  *self-improve during use*; format-compatible with the
  `agentskills.io` standard. CLI exposes them via `/<skill-name>`.
- **Tools**: 40+ built-in; configurable via `hermes tools`. Tools are
  callable from Python scripts over RPC — see the
  zero-context-tool-pipelines pattern below.
- **Execution substrate**: seven terminal backends — local, Docker,
  SSH, Singularity, Modal, Daytona, Vercel Sandbox. Daytona/Modal
  offer serverless persistence (environment hibernates when idle).
- **Gateway**: one process bridges Telegram, Discord, Slack, WhatsApp,
  Signal, Email, CLI. Voice memo transcription. Cross-platform
  conversation continuity.
- **Cron**: built-in scheduler with delivery to any platform; natural-
  language schedule specs.
- **Subagents**: spawn isolated workers for parallel workstreams.
- **Permissions**: command approval, DM pairing, container isolation.
- **MCP**: supports arbitrary MCP servers for extension.
- **Migration**: explicit `hermes claw migrate` from OpenClaw —
  imports SOUL.md persona, memories, skills, allowlist, messaging
  config, allowlisted API keys, TTS assets, AGENTS.md.
- **Research-ready**: batch trajectory generation; trajectory
  compression "for training the next generation of tool-calling
  models" (i.e., Hermes-style data as RL/SFT corpus).

## Useful bits

- **"Zero-context-cost turns" via tool RPC.** The most architecturally
  interesting line in the README: *"Write Python scripts that call
  tools via RPC, collapsing multi-step pipelines into zero-context-cost
  turns."* When a multi-step tool chain is wrapped in a script, the
  LLM pays context cost on the script call and the final return — not
  on every intermediate tool result. Promoted to a new seedling
  concept: [[concepts/scripted-tool-pipelines]].
- **Agent-curated memory with periodic nudges.** Hermes does not
  passively accumulate context; the model is *nudged* on a schedule
  to consolidate. This is a concrete write-side policy that pairs
  with [[concepts/skill-library-lifecycle]] (insert/update/delete
  triggered not just by task events but by time).
- **Skills self-improve during use, not only between tasks.** A
  skill executed today can be edited (by the agent) as a side-effect
  of execution. Distinct from SkillOS's batch curation pass between
  task groups; closer to ReasoningBank's continuous update. Worth
  flagging as a third operating mode for skill-library lifecycle:
  not just write-time vs. retrieval-time but also *execution-time*
  curation.
- **Pluggable execution substrate.** Seven backends through a single
  abstraction. Hermes alone doesn't justify a new concept — the
  pattern is also visible in OpenHarness (next ingest) and the
  Claude Code MCP transport list. Defer a `pluggable-execution-substrate`
  seeding decision until after the OpenHarness note.
- **Migration story as architectural commentary.** The fact that
  Hermes ships a one-shot migration *from* OpenClaw indicates these
  harnesses are converging on a common data model — personas,
  memories, skills, allowlists. The interop surface is a candidate
  for downstream standardization.
- **Sub-agents are not typed roles.** Unlike AIBuildAI's
  manager/designer/coder/tuner ([[concepts/hierarchical-delegation]]),
  Hermes subagents are anonymous isolated workers. Same primitive
  (spawn child context), different policy (typed roles vs. dynamic
  spawning). Worth tracking as the spectrum the concept lives on.

## Follow-up

**Relevance:** 4. Hermes is the most coherent open-source
existence-proof of the persistent-agent harness shape, with material
new evidence for [[concepts/agent-native-memory]] (Honcho dialectic
modeling + FTS5 + nudges) and [[concepts/skill-library-lifecycle]]
(execution-time self-improvement, agentskills.io standard). It seeds
one new concept ([[concepts/scripted-tool-pipelines]]). Not a 5
because it doesn't *anchor* a load-bearing concept the way ByteRover
or SkillOS do — it's a confirmatory peer rather than a defining
source.

- The README is shallow on internals. The `developer-guide/architecture`
  doc (https://hermes-agent.nousresearch.com/docs/developer-guide/architecture)
  has the `QueryEngine`/`AIAgent` class structure and would be the
  next thing to fetch if we want to compare typed-tool vs.
  dynamic-registry patterns at code level.
- The "trajectory compression for training next-gen tool-callers"
  thread is independently interesting — Hermes runs are training
  data for the next Hermes model. Open question: does this create a
  feedback dynamic where the harness's behavioral biases get baked
  into the next generation's pretrain?
- Compatibility with `agentskills.io` is worth a separate scan —
  if that becomes a de-facto standard, it shapes how this project's
  own skill files should be structured for export.
