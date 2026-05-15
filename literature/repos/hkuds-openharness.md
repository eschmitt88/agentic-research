---
kind: repo
name: "OpenHarness (HKU Data Science Lab)"
url: https://github.com/HKUDS/OpenHarness
commit: HEAD (README @ 2026-05-15; codebase ≥ v0.1.7, 2026-04-18)
source: "raw/repos/hkuds-openharness.md"
added: "2026-05-15"
relevance: 4
status: scanned
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/shared-skill-namespace]]"
tags: [agent-harness, academic, open-source, hku, plugins, skills, format-interop, dry-run]
---

# OpenHarness (HKU Data Science Lab)

## Purpose

OpenHarness is an open-source Python agent harness from HKU Data Science
Lab — explicitly positioned as research infrastructure ("designed for
researchers, builders, and the community" — "Understand how production
AI agents work under the hood"). The README slogan summarizes the
stance: *"The model is the agent. The code is the harness."* Ships
alongside `ohmo`, a personal agent built on top of OpenHarness that
piggybacks on existing Claude Code or Codex subscriptions rather than
its own API key.

Currently at v0.1.7 (2026-04-18). 114 unit/integration tests passing,
plus several E2E suites including a "Real Skills + Plugins" suite that
runs against actual `anthropics/skills` and `claude-code` plugins —
format compatibility is verified, not just claimed.

## Shape

Explicit 14-subsystem layout under `openharness/`:

| Subsystem | Role |
|---|---|
| `engine/` | Agent loop (stream → tool-call → loop). Streaming + parallel tool execution + exponential-backoff retry + token/cost tracking. |
| `tools/` | 43+ tools. Pydantic-validated inputs, self-describing JSON Schema, permission + hook integration. |
| `skills/` | On-demand `.md` skill loading. Compatible with `anthropics/skills` SKILL.md format. |
| `plugins/` | Extensions. Compatible with `claude-code` plugin layout (commands/ + hooks/ + agents/ directories). |
| `permissions/` | Multi-level (default / auto / plan), path rules, command denylist. |
| `hooks/` | PreToolUse / PostToolUse lifecycle events. |
| `commands/` | 54 slash commands (`/help`, `/commit`, `/plan`, `/resume`, …). |
| `mcp/` | MCP client. HTTP transport added v0.1.5; JSON-schema-inferred tool inputs. |
| `memory/` | Persistent cross-session knowledge. CLAUDE.md discovery + auto-compact + MEMORY.md. |
| `tasks/` | Background task lifecycle. |
| `coordinator/` | Multi-agent — subagent spawning, team registry, ClawTeam roadmap integration. |
| `prompts/` | System prompt assembly, CLAUDE.md discovery, skill injection. |
| `config/` | Multi-layer config with migrations. |
| `ui/` | React/Ink TUI (backend protocol + frontend). |

The conceptual frame (from the README's section "What is an Agent
Harness?") is a five-element decomposition:

> Harness = Tools + Knowledge + Observation + Action + Permissions

The 14 subsystems implement these five elements with safety/observability
infrastructure underneath.

## Useful bits

- **Multi-root skill discovery — the cross-harness namespace.**
  OpenHarness reads skills from *three* user-level roots:
  `~/.openharness/skills/`, `~/.claude/skills/`, `~/.agents/skills/`
  (plus the project-level equivalents). Combined with Hermes's
  explicit agentskills.io compatibility, this defines a de-facto
  *shared skill namespace* — a skill written for Claude Code works
  in OpenHarness without porting. Promoted to a new seedling concept:
  [[concepts/shared-skill-namespace]].

- **Dry-run with verdict.** `oh --dry-run` resolves settings, auth
  state, skills, commands, tools, MCP servers without calling the
  model, executing tools, or spawning subagents. Returns one of
  `ready` / `warning` / `blocked` plus a list of `next actions`. This
  is a *pre-flight check* pattern — distinct from a preview because
  the verdict is actionable. Worth tracking as an idiom for safe
  autonomous loops; if `/iterate` ever gets a chain mode in this
  project, a dry-run-with-verdict step before each cycle is a clean
  failure-prevention mechanism. Not promoted to a concept yet (single
  source), but flagged.

- **Plan mode as a permission class.** Three permission modes:
  default (ask before write/execute), auto (allow everything),
  **plan** (block all writes — for large refactors and review-first
  workflows). Most harnesses treat "review mode" as an unwritten
  protocol the user enforces; OpenHarness compiles it into the
  permission system. Worth comparing against this project's own
  `respects:` declarations in skill frontmatter (e.g. HCE rule, which
  similarly enforces a "no test/ access during search" policy at
  the rule layer, not the model layer).

- **Pydantic-typed tools + JSON Schema self-description.** Strong
  evidence on the typed-tools side of Huang's comparison axis
  ([[raw/_candidates/2026-05-15-agent-harnesses]] entry #6).
  OpenHarness sits firmly with Claude Code on the typed-interface
  side; Hermes is the dynamic-registry counterpoint. Both approaches
  ship and pass tests — the choice is design philosophy, not
  capability.

- **Tested against the actual upstream ecosystems.** The "Real Skills
  + Plugins" E2E suite (12 plugins) verifies `anthropics/skills` and
  `claude-code` plugins work unmodified. This is a strong
  *compatibility-as-contract* discipline that distinguishes
  OpenHarness from harnesses that merely claim portability.

- **ohmo's file-based persona/identity/memory split.** `soul.md`
  (persona), `identity.md` (who the agent is), `user.md` (user
  profile), `BOOTSTRAP.md` (first-run ritual), `memory/` (running
  memory). Five distinct files for what other harnesses cram into
  one "system prompt" or one "memory file". The decomposition is
  consistent with this project's [[concepts/typed-claim-partition]] —
  different *types* of persistent state get different files, not
  just different sections of one file.

- **Provider workflows, not provider drivers.** Five named workflows
  (Anthropic-compatible API, Claude Subscription, OpenAI-compatible
  API, Codex Subscription, GitHub Copilot). The
  *workflow* framing — including OAuth device flow for Copilot and
  subscription-bridge for Claude/Codex — is more user-flow-shaped
  than backend-driver-shaped. Confirms the
  [[concepts/hybrid-model-backends]] pattern from the consumption
  side: model selection is a workflow choice, not a code change.

## Follow-up

**Relevance:** 4. OpenHarness is the closest academic / open-source
peer to this project's worldview — Markdown-first knowledge, skills
as `.md` files, explicit subsystem decomposition, compatibility with
the `anthropics/skills` and `claude-code` plugin formats. Strong
confirmation for [[concepts/skill-library-lifecycle]],
[[concepts/agent-native-memory]], and
[[concepts/hierarchical-delegation]]. Seeds one new concept
([[concepts/shared-skill-namespace]]) jointly with Hermes. Not a 5
because the README is breadth-over-depth — every subsystem gets a
paragraph, none gets the kind of architectural depth that anchors a
load-bearing concept the way ByteRover does for agent-native-memory.

- **Architecture doc not yet fetched.** The `docs/SHOWCASE.md`,
  CHANGELOG, and the `coordinator/` source would clarify the
  Agent-as-Tool vs. fixed-role question relative to AIBuildAI /
  AiScientist ([[concepts/hierarchical-delegation]]).
- **The 5-element decomposition (Tools + Knowledge + Observation +
  Action + Permissions)** could itself be a concept (the harness as
  a typed structure rather than a "framework"). Holding off — needs
  more sources to confirm it's a recognized abstraction and not
  just OpenHarness's pedagogical framing.
- **The "shared skill namespace" claim should be tested empirically**
  by writing a skill to `~/.claude/skills/` and verifying it loads
  under OpenHarness. If true at a path level, downstream projects
  could keep one canonical skill directory and use it across
  harnesses — meaningful operational claim.
- **Dry-run + verdict** is interesting as a pattern but isolated to
  OpenHarness in the current sample. If Hermes or Claude Code ship
  equivalents, promote to a concept.
