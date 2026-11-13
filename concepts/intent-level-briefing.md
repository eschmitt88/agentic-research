---
kind: concept
name: "intent-level-briefing"
status: seedling
added: "2026-11-13"
sources:
  - "[[docs/system-proposals/2026-07-31-instruction-ablation-program]]"
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/lossless-context-offload]]"
related_experiments: []
used_by:
  - project_slug: intent-briefing
    imported_on: "2026-11-13"
tags: [prompting, delegation, briefing, instruction-design, governance, harness]
---

# intent-level-briefing

## Definition

Instructions to a capable model, whether a system prompt, subagent brief or
skill, state the **goal, the hard constraints, the reasons behind them, and
the context the author would otherwise take for granted**. They leave the
method to the reader instead of scripting it step by step. Hard invariants are
not written as prose in either style. They move into code: hooks, typed
checks, permission gates.

## Why it matters here

This concept complements [[concepts/typed-enforcement]] and
[[concepts/constraint-pinning]]. Those concepts decide what the *harness*
guarantees. This one decides what is left for *prose*, and how prose should be
written once the invariants have moved to code. The box has already acted on
it once: the 2026-07-31 instruction-ablation program cut claude-system's skill
prose from ~18k to ~7k words (keep contracts and rationale, delete procedure,
compile algorithms into scripts). That program said it could not run real
ablations. It also named the main risk: contract-only skills assume the reader
has frontier-level inference.

Open boundaries, which the empirical work lives downstream to test:

- **Capability:** below some implementer strength, scripts likely win.
- **Context, not capability, is the reader's deficit.** A brief fails when the
  author's working context isn't transferred
  ([[concepts/lossless-context-offload]]). Mission command also assumes shared
  doctrine, which here means rules and CLAUDE.md.
- **Horizon:** drift under long chained runs is still unmeasured in either
  direction.
- **Two axes:** intent vs procedure is a different axis from terse vs verbose.

## Connections

- [[concepts/hierarchical-delegation]]: the main place this applies, since
  every delegation is a brief.
- [[concepts/enforcement-boundary-placement]]: where the code/prose line falls.
- Downstream: the `intent-briefing` project (harness, experiments, writing
  guide).
