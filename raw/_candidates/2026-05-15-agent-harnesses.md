---
kind: candidates
topic: "agent harnesses — Claude Code (leaked), OpenClaw, Hermes, and adjacent work"
discovered: 2026-05-15
source: discover
n_requested: 10
n_returned: 10
---

## 1. The Claude Code Leak: What the Harness Actually Looks Like (Paddo, 2026)

- url: https://paddo.dev/blog/claude-code-leak-harness-exposed/
- type: post
- summary: Technical walkthrough of the March-2026 source-map leak — names 19 permission-gated tools, a three-layer memory architecture (short-term / session / long-term files), 6 MCP transport types (Stdio, SSE, HTTP, WebSocket, SDK, ClaudeAiProxy), and 44 unshipped feature flags including the "KAIROS" always-on daemon mode.
- reason: Most concrete public account of what a production frontier-lab harness actually contains. Touches every architectural concept already in this project (`agent-native-memory`, `file-as-bus`, `hierarchical-delegation`, `skill-library-lifecycle`) and provides external grounding for them.

## 2. Hermes Agent — Nous Research (GitHub, v0.9.0 Apr 2026)

- url: https://github.com/nousresearch/hermes-agent
- type: repo
- summary: Open-source long-running personal-agent harness. Closed-learning-loop memory (Honcho dialectic profiles + FTS5 session search + agent-curated nudges), 40+ tools across 7 terminal backends (local/Docker/SSH/Singularity/Modal/Daytona/Vercel Sandbox), provider-agnostic model routing, autonomous skill creation compatible with the agentskills.io standard.
- reason: The clearest open-source counterpart to Claude Code, but optimized for persistence rather than coding sessions. Skill self-creation is a direct experimental analogue of `skill-library-lifecycle` and worth comparing against SkillOS.

## 3. OpenClaw — Agent Harness Plugin SDK (Official docs)

- url: https://docs.openclaw.ai/plugins/sdk-agent-harness
- type: post
- summary: Reference doc defining "harness" as the executor for one prepared agent turn — distinct from provider/channel/tool registry. Spells out the `runtimePlan` interface, transcript-mirroring contract, `reset(...)` lifecycle, and the policy that the core (not the harness) owns provider/model selection, auth, thinking levels, and context budgets.
- reason: This is the cleanest formal *separation-of-concerns* statement in the harness literature — explicitly factoring "what the executor decides per-turn" from "what the orchestrator decides upstream." Maps almost 1:1 onto this project's curator/executor separation.

## 4. OpenHarness + Ohmo — HKU Data Science Lab (GitHub)

- url: https://github.com/HKUDS/OpenHarness
- type: repo
- summary: Lightweight Python harness from an academic lab. Five-component model (Tools, Knowledge, Observation, Action, Permissions) implemented over ten subsystems (Engine, Tools, Skills, Permissions, Plugins, Memory, Coordinator, …). Format-compatible with anthropics/skills and claude-code plugins. Ships Ohmo, a personal agent that piggybacks on existing Claude Code / Codex subscriptions rather than its own API key.
- reason: Closest *research-lab* peer to this project's worldview — Markdown-first knowledge, skills-as-files, multi-agent coordinator. The compatibility-by-default design choice is a useful counterpoint to the build-your-own-stack reflex.

## 5. Learn Harness Engineering by Building a Mini OpenClaw (DEV.to tutorial)

- url: https://dev.to/truongpx396/learn-harness-engineering-by-building-a-mini-openclaw-bdm
- type: post
- summary: Ten-part pedagogical decomposition of a harness: (1) agent loop, (2) tool dispatch, (3) JSONL session persistence with overflow summarization, (4) channel adapters, (5) gateway router, (6) intelligence layer, (7) heartbeat/scheduling, (8) crash-safe delivery queue, (9) three-tier retry/auth-rotation/compaction, (10) named-FIFO concurrency lanes.
- reason: The most explicit *minimum viable harness* spec I've seen. Each of the ten components is a candidate concept name and a candidate failure mode — directly useful for cross-checking what this project's design assumes vs. omits.

## 6. Chapter 1: The Harness Paradigm — Claude Code vs. Hermes Agent (Ken Huang, Substack)

- url: https://kenhuangus.substack.com/p/chapter-1-the-harness-paradigm-claude
- type: post
- summary: Side-by-side architecture comparison. Claude Code: typed `Tool` interface with Zod schemas, async-generator streaming, mutable state owned by a `QueryEngine`. Hermes: dynamic registry with lambda handlers, sync loop, thread-safe `IterationBudget` shared across parent + subagents, runtime `check_fn` guards instead of compile-time declarations.
- reason: A rare apples-to-apples tradeoff analysis between two real frontier harnesses. The typed-vs-registry axis is exactly the kind of design choice this project should be tracking as a `[[concept]]`.

## 7. AI Harness Engineering: What 512,000 Lines of Claude Code Leak Taught Us (Savelis, Medium)

- url: https://medium.com/@savelis.pedro/ai-harness-engineering-what-512-000-lines-of-claude-code-leak-taught-us-e7809a9cef04
- type: post
- summary: Synthesis-style writeup. Highlights 14 prompt-cache invalidation vectors with "sticky latches" across mode switches, 23 sequential bash security checks (including Zig-computed client-side auth hashing), self-healing context compaction that runs proactively, an agent-edited `memory.md` index, and prompt-native (not framework-native) multi-agent coordination.
- reason: The "prompt cache as CPU register" framing and the "self-healing compaction" observation both deserve named concepts in this project; this article surfaces them cleanly enough to ingest as primary source for the patterns.

## 8. What Claude Code's Source Leak Actually Reveals (Marc Bara, Medium)

- url: https://medium.com/@marc.bara.iniesta/what-claude-codes-source-leak-actually-reveals-e571188ecb81
- type: post
- summary: Argues the underappreciated lessons are: (1) the permission-gate pattern as first-class architecture not bolt-on safety; (2) context compaction as a *memory hierarchy* — hot/cold tiers, not just token optimization; (3) anti-distillation defenses (fake tool definitions injected to poison competitor scraping); (4) tool isolation enforced at the architecture layer, not the feature layer.
- reason: The "anti-distillation" observation is genuinely novel and adversarial-thinking-flavored; the memory-hierarchy framing reinforces this project's `agent-native-memory` concept with a vocabulary borrowed from systems engineering.

## 9. Claw Code Killed Claude Code? (Mehul Gupta, Medium)

- url: https://medium.com/data-science-in-your-pocket/claw-code-killed-claude-code-02aab80b0838
- type: post
- summary: Profile of Sigrid Jin's clean-room reimplementation in Rust (~73%, conversation loop + tool execution + streaming) and Python (~27%, LLM provider abstraction + lifecycle + session persistence). Trait-based `ConversationRuntime<C, T>` decouples API client from tool execution. Hit 100K stars in 24 hours after the leak.
- reason: Provides a concrete data point for the question "what's the right *substrate* for a harness?" — most frontier examples are TypeScript or Python; the Rust-core/Python-edge split is a substantive design choice worth understanding.

## 10. OpenClaw ACP Harness Explained — Running Coding Agents Inside an Orchestrator

- url: https://www.openclawplaybook.ai/guides/openclaw-acp-harness-explained/
- type: post
- summary: Walkthrough of Agent Client Protocol (ACP) — OpenClaw's mechanism for nesting external coding agents (Pi, Claude Code, Codex, OpenCode, Gemini CLI) inside a parent orchestrator. The orchestrator hands a coding task to the child harness over ACP and receives results programmatically, without manual session juggling.
- reason: This *nested-harness* pattern is novel for this project. Maps directly onto `[[hierarchical-delegation]]` — a curator-style outer agent could in principle dispatch ML experiments to a Claude-Code-shaped inner harness rather than spawning raw subagents. Worth knowing whether ACP is a credible interface for that.
