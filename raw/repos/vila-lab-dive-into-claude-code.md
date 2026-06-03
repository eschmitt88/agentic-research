<!-- Captured 2026-06-03 from https://github.com/VILA-Lab/Dive-into-Claude-Code (README.md, main branch). Companion to arXiv:2604.14228. Immutable raw snapshot — do not edit after ingest. -->

# Summary: Dive into Claude Code

This comprehensive analysis examines Claude Code v2.1.88 (~512K lines across 1,884 TypeScript files), revealing a fundamental insight: **only 1.6% comprises AI decision logic; 98.4% is deterministic infrastructure**.

## Core Architecture

The system addresses four critical design questions:

1. **Reasoning location**: Model reasons; harness enforces through permission gates and context management
2. **Execution engines**: One `queryLoop` async generator across all interfaces (CLI, SDK, IDE)
3. **Safety posture**: Deny-first default—strictest rule always wins
4. **Resource constraint**: Context window (~200K–1M tokens) addressed via 5 compaction stages

## Key Design Principles

Five human values translate into thirteen design principles:

- **Human Decision Authority**: Principals retain control; when approval fatigue reached 93%, the system restructured boundaries rather than add warnings
- **Safety, Security, Privacy**: Seven independent safety layers protect even when human vigilance lapses
- **Reliable Execution**: Gather-act-verify loops enable graceful recovery
- **Capability Amplification**: Infrastructure enables model performance
- **Contextual Adaptability**: Graduated trust spectrum evolves over time

## Critical Systems

**The Query Loop**: ReAct-pattern while-loop with nine-step pipeline per turn. Before each model call, five compaction shapers run sequentially (Budget Reduction → Snip → Microcompact → Context Collapse → Auto-Compact).

**Permissions**: Seven modes form a graduated spectrum (`plan` → `default` → `acceptEdits` → `auto` → `dontAsk` → `bypassPermissions`). Deny-first rule: broad denials override narrow allows.

**Context & Memory**: Nine ordered sources build context; CLAUDE.md hierarchy provides file-based, inspectable (not vector-embedded) memory. No embeddings—fully version-controllable.

**Subagents**: Six built-in types plus custom agents. Sidechain transcripts protect parent context; only summaries return, preventing verbosity explosion.

## Broader Implications

The analysis identifies six design signals reshaping agent architecture:

- Runtime and control plane become first-class design surfaces
- Context requires managed infrastructure (provenance, review, rollback)
- Execution boundary is the safety boundary
- Tools form a supply chain (registries, allowlists, versioning)
- Humans become managers and verifiers, not just operators
- Observability closes improvement loops

## Community Resources

The repository curates 50+ reimplementations, architectural analyses, and learning resources, plus comparisons with peer systems (OpenClaw, Hermes-Agent) revealing how deployment context drives design choices.

**License**: CC BY-NC-SA 4.0
**Citation**: arXiv:2604.14228
