---
kind: post
title: "The Claude Code Leak: What the Harness Actually Looks Like"
author: "Paddo (paddo.dev)"
url: https://paddo.dev/blog/claude-code-leak-harness-exposed/
source: "raw/web/paddo-dev-claude-code-leak-harness-exposed.md"
added: "2026-05-15"
relevance: 3
related_experiments: []
related_concepts:
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hierarchical-delegation]]"
tags: [claude-code, leak, agent-harness, memory-tiers, context-management, permission-gates, kairos, mcp]
---

# The Claude Code Leak: What the Harness Actually Looks Like

## TL;DR

Synthesis writeup of the March 31, 2026 Claude Code source-map leak.
A `.map` artifact shipped inside `@anthropic-ai/claude-code` v2.1.88
exposed 512K lines of TypeScript across 1,906 files via a known Bun
bug (#28001) that served source maps in production builds. The
architecture-level findings (which are the part that matters for this
project) are:

- **19 permission-gated tools** — file I/O, shell, Git, web,
  notebook editing. Independent sandboxing per tool with configurable
  access controls.
- **Three-layer memory architecture** — short-term context,
  session-level persistence, long-term memory files. Tiered, not
  flat.
- **Context entropy management** — explicit subsystem deciding what
  stays in context and what is evicted. Paddo: "the hard problem
  that separates usable agents from toy demos."
- **6 MCP transport types** — Stdio, SSE, HTTP, WebSocket, SDK,
  ClaudeAiProxy. More transport flexibility than the public docs
  suggested.
- **44 unshipped feature flags** — built but not released; a roadmap
  hiding in plain sight.
- **KAIROS** (>150 references in source) — unreleased autonomous
  daemon mode, "always-on background agent."
- **undercover.ts** (~90 lines) — system-prompt injection stripping
  Anthropic attribution / AI-disclosure from external repo
  contributions. Activates for Anthropic employees, no force-off.

Operational fallout (less relevant to this project but worth flagging):
Sigrid Jin shipped a clean-room Python+Rust rewrite ("claw-code")
within 24h that hit 100K stars; Anthropic's DMCA takedown took out
~8,100 unrelated repos before being narrowed to 96; Anthropic's
billing model pivoted from subscription to pay-as-you-go for
third-party harnesses, moving "the moat from harness to model."

## Key points

- **"The harness is the product, not the model."** Paddo's core
  thesis — confirmed by the leak. The 19 tools, the memory tiers,
  and the context-management code are where Claude Code's
  capability advantage lives, not the underlying Opus/Sonnet
  weights. Cross-checks against this project's own framing in
  CLAUDE.md ("Research-about-research ... architectural concepts
  for autonomous research agents"): the architecture is the
  research object.

- **Three-layer memory is now confirmed for Claude Code.** Pairs
  with the same pattern in OpenHarness (CLAUDE.md / auto-compact /
  MEMORY.md), Hermes (Honcho profiles + FTS5 sessions + agent-
  curated memory), and ByteRover ([[literature/papers/nguyen2026byterover]]
  Domain→Topic→Subtopic→Entry). The hierarchical-memory pattern
  has now been observed in four independent harnesses; treat as
  consensus.

- **Context entropy management is its own thing.** Distinct from
  selective-memory-retrieval (when to *read* memory) and
  skill-library-lifecycle (when to *write* memory). Context-eviction
  is the question "when the working set overflows, what gets
  dropped from the active context window?" Seeded as a new concept
  here: [[concepts/context-eviction-policy]].

- **Permission gates are infrastructure, not a feature.** Each of
  the 19 tools has independent sandboxing. Bash specifically
  carries the most elaborate validation chain (per Savelis: 23
  sequential checks per command). OpenHarness's three permission
  modes (default / auto / plan) and PreToolUse/PostToolUse hooks
  are the open-source analogue. Pattern attested in three sources
  (Claude Code, OpenHarness, Bara's commentary); not promoted to a
  concept yet — needs one more direct attestation to be
  comfortable seeding.

- **6 MCP transports vs. public-doc claim of fewer.** Stdio + SSE
  + HTTP + WebSocket + SDK + ClaudeAiProxy. For this project,
  the practical implication is that MCP is a more general transport
  surface than it appeared — a downstream project building MCP
  tooling has wider integration options than the public docs
  suggest.

- **KAIROS is the missing piece for autonomous research loops.**
  150+ references to an unreleased daemon mode strongly suggests
  Anthropic is shipping a long-running background-agent capability.
  If/when KAIROS becomes public, the architectural fit with this
  project's curation cadence (Monday cron /digest, etc.) is direct
  — the daemon is the right substrate for the patterns this project
  already enacts via cron + manual session resumption.

- **Synthesis source, not primary.** Paddo is a quality blog
  writeup but cites the leaked source rather than being it. The
  primary artifact (512K lines of TypeScript) is closed-by-Anthropic
  and not fetchable here. This literature note's role is to anchor
  the *facts* extracted from the leak; deeper architectural claims
  should be confirmed against primary sources (the leaked source
  itself, claw-code's clean-room implementation, Anthropic's
  published Claude Code docs) before being load-bearing.

## Follow-up

**Relevance:** 3. A synthesis writeup, not primary evidence. Seeds
[[concepts/context-eviction-policy]] *jointly* with OpenHarness and
Hermes (which provide the open-source observations of the same
pattern); confirms [[concepts/agent-native-memory]] three-layer
substructure on Claude Code; provides the strongest available
public account of what's actually in the leaked harness short of
reading the leaked source. Not a 4 because the seeding is mostly
carried by other sources and the confirmations are corroborative
rather than novel.

- **Primary follow-up: claw-code.** Sigrid Jin's clean-room Rust+Python
  rewrite (mentioned earlier in `raw/_candidates/2026-05-15-agent-harnesses.md`
  entry #9) would be the more architecturally substantive thing to
  fetch next, since it's an open-source full implementation
  rather than a writeup. Worth fetching its GitHub README if the
  repo is still up post-DMCA.
- **KAIROS** is the most architecturally consequential of the
  44 feature flags for this project. If Anthropic ships it,
  re-evaluate the curation skills (`/digest`, `/lint`, `/wrap`)
  for daemon-mode compatibility.
- **Permission-gate-as-architecture** is now attested in three
  independent sources (Claude Code, OpenHarness, Bara). One more
  direct attestation and it's worth promoting to a concept.
- **MCP transport surface.** The 6-transport list is more than the
  public docs claim. If this project ever ships its own MCP
  server for the concept graph, knowing the full transport surface
  matters for integration choices.
- **Co-design claim.** Paddo's closing thought — that Claude Code's
  harness "was designed alongside the models it orchestrates" and
  that this co-design is the residual moat — is interesting but
  unverifiable from public artifacts. Worth tracking whether any
  Anthropic publication confirms or denies this.
