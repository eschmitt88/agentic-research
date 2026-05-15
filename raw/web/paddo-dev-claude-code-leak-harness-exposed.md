---
url: https://paddo.dev/blog/claude-code-leak-harness-exposed/
fetched: 2026-05-15
kind: post
title: "The Claude Code Leak: What the Harness Actually Looks Like"
host: paddo.dev
---

# The Claude Code Leak: What the Harness Actually Looks Like

March 31, 2026, 4 AM. Security researcher Chaofan Shou tweets that Claude Code's source has leaked via a source map file in the npm registry. By sunrise, the entire agent harness - 512,000 lines of TypeScript across 1,906 files - is mirrored, forked, and being picked apart by every AI lab on earth.

By the end of the day, someone had rewritten the core from scratch in Python. That repo hit 100K stars in 24 hours. The fastest in GitHub history.

The leak itself is an ops failure. The interesting part is what it revealed.

## How It Happened

A `.map` file shipped inside `@anthropic-ai/claude-code` v2.1.88 on npm. Source maps are debugging artifacts that connect minified JavaScript back to the original TypeScript. This one pointed to a zip archive sitting on Anthropic's own Cloudflare R2 bucket. Publicly accessible.

The root cause: a known Bun bug (#28001) filed March 11 that serves source maps in production builds when documentation says they shouldn't be. The bug sat open for 20 days. Nobody at Anthropic caught it during release.

A nearly identical source map leak happened with an earlier Claude Code version in February 2025. Same mechanism, same packaging oversight. The fix clearly didn't stick.

Anthropic's response was measured: "a release packaging issue caused by human error, not a security breach." No customer data or credentials were exposed. They confirmed it was limited to the CLI source code and rolled out prevention measures.

Fair enough. But the damage was architectural, not operational.

## What the Source Reveals

The leaked code confirms what many suspected but couldn't prove: the harness layer is where the real product lives. Not the model. The harness.

Key findings from the source:

- **19 permission-gated tools**: File I/O, shell execution, Git operations, web scraping, notebook editing. Each independently sandboxed with configurable access controls.
- **Three-layer memory architecture**: Moves beyond traditional "store everything" retrieval. Short-term context, session-level persistence, and long-term memory files work together to manage what the agent knows and when.
- **44 unshipped feature flags**: Fully built capabilities that haven't been released yet. A roadmap hiding in plain sight.
- **6 MCP transport types**: Stdio, SSE, HTTP, WebSocket, SDK, and ClaudeAiProxy. Far more transport flexibility than the public docs suggest.
- **Context entropy management**: A sophisticated system for deciding what stays in context and what gets evicted. This is the hard problem that separates usable agents from toy demos.
- **KAIROS**: Referenced over 150 times in the source. An unreleased autonomous daemon mode where Claude operates as a persistent, always-on background agent.
- **undercover.ts**: ~90 lines that inject a system prompt instructing Claude to never mention it's an AI and strip all Co-Authored-By attribution when contributing to external repos. Activates for Anthropic employees. No force-off switch.

> First there was chat, then there was code, now there is claw.
>
> — Andrej Karpathy, replying on X

Paddo's prior post (`/blog/agent-harnesses-from-diy-to-product`) argues
that harness patterns are becoming products. The leak confirms the
thesis: Anthropic's competitive advantage in Claude Code isn't Opus
or Sonnet. It's the orchestration layer that makes those models
useful for real work. The 19 tools, the memory tiers, the context
management — that's the product.

## The Claw-Code Phenomenon

Korean developer Sigrid Jin saw the leak and did what any AI-native developer would do: used AI tools to rewrite the core agent harness from scratch before sunrise. The result, "claw-code," was explicitly positioned as a clean-room implementation. No copied code. Independent architecture inspired by the leaked patterns.

The numbers are absurd:

- **100K+ stars in 24 hours** (fastest repo in GitHub history)
- **136K+ stars** by April 2
- **101K forks**
- Rust port already in progress

The meta angle writes itself: AI tools used to reverse-engineer AI tooling, overnight, by one person.

Clean-room status matters: independent audits confirm claw-code contains no Anthropic source code, model weights, API keys, or user data. This matters for the inevitable legal questions. Clean-room reverse engineering has strong legal precedent. Hosting leaked source does not.

## DMCA Whack-a-Mole

Anthropic's legal response was swift. A DMCA takedown notice hit GitHub targeting repos hosting the leaked source. The problem: the notice targeted a fork network connected to Anthropic's own public Claude Code repository. GitHub processed the takedown against the entire network.

**8,100 repositories went down.** Most had nothing to do with the leak.

Developers started reporting DMCA notices for forks containing only skills, examples, and documentation. One developer got a takedown for simply forking the public repo. Gergely Orosz called it DMCA abuse. Boris Cherny (Anthropic's head of Claude Code) acknowledged the overreach was accidental and retracted the bulk of the notices, narrowing the final takedown to one repo and 96 forks.

But the damage was done. The Streisand effect kicked in hard:

- **Direct mirrors** went down, then reappeared on decentralized platforms Anthropic can't reach
- **Clean-room rewrites** (claw-code) survived entirely, since they contain no Anthropic source code
- **The claw-code repo** now has more stars than Anthropic's own Claude Code repo (136K vs 97K)

The whack-a-mole continues. Anthropic can take down direct copies, but 512,000 lines of source code are permanently in the wild. The architecture is public knowledge. No amount of DMCA notices changes that.

## The Security Angle Nobody's Talking About Enough

Within hours of the leak, threat actors were seeding trojanized repos. Zscaler documented fake "leaked Claude Code" repositories bundling:

- **Vidar Stealer** (credential harvesting malware)
- **GhostSocks** (proxy botnet)
- **Cryptocurrency miners**

A separate malicious Axios npm supply chain attack hit the same day (March 31, 00:21-03:29 UTC), creating a perfect storm for anyone updating packages.

If you downloaded anything claiming to be "the real Claude Code source" from unofficial repos: assume compromise. Run a credential audit. The official Claude Code installer from Anthropic is the only safe source.

## What This Changes

Three things shift:

- **The harness is no longer a black box.** Every competitor — Cursor, Windsurf, Cline, OpenCode — now has the architectural blueprint. The memory tiers, tool sandboxing patterns, context management strategies. Anthropic's execution speed and ecosystem lock-in are now the moat, not the architecture itself.
- **Open source alternatives have a head start.** Claw-code implementing 19 tools with MCP support and multi-provider backends means the harness layer is commoditizing faster than anyone expected.
- **Dependency hygiene matters more than ever.** A Bun bug open for 20 days caused a major IP leak. A concurrent npm supply chain attack poisoned the aftermath. If you're shipping production software through npm, your packaging pipeline is a security surface. Source maps, debugging artifacts, internal configs — anything that gets bundled accidentally is public.

> The moat was never the architecture. It was the speed of iteration on top of it.

## The Uncomfortable Question

The leaked source confirms why Claude Code is good: the engineering behind context management, tool sandboxing, and session persistence is genuinely sophisticated. This isn't a thin wrapper over an API.

But now it's public. And the 44 feature flags tell competitors exactly where Anthropic is heading next.

The question isn't whether open source will replicate the harness. It's whether Anthropic can ship faster than the community can clone. The walled garden strategy makes more sense in this context: if the architecture is public, controlling distribution becomes the only lever.

History suggests that's not enough. But Anthropic has one advantage the clones don't: tight integration between the harness and the model. The context entropy system was designed alongside the models it orchestrates. Replicating the architecture without that co-design is like having the blueprints for a Formula 1 car but fitting it with a different engine.

**Update:** Hours after this post went live, Anthropic emailed every subscriber: third-party harnesses now require pay-as-you-go billing instead of subscription access. The leak made technical enforcement futile, so they shifted to billing enforcement. The moat moved from harness to model.
