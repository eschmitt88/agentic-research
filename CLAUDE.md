# Project: agentic-research

Short orientation only. User-level `~/.claude/CLAUDE.md` holds the durable
principles; this file refines them for this project.

## What this project is about

Research-about-research: curated literature and importable architectural
concepts for **autonomous research agents** — primarily ML-research
agents (the dominant focus, e.g. MLE-bench / PaperBench / AIRA) but
including general scientific-research automation (e.g. Kosmos in
biology) where the architectural patterns transfer. Pure
software-engineering agents (ChatDev, etc.) are out of scope unless
they directly inform research-agent architecture.

## Role

This project is the research hub for every other project on this box.
Concept and literature notes here are a read interface, not only
internal artifacts: downstream projects declare the concepts they
depend on, and `/sync-imports` appends `used_by:` back-references so
this project can see which concepts are load-bearing.

## Import contract

Downstream projects declare hub concepts in their `CLAUDE.md`:

```
## Inherited architecture
Read these by path before relying on them (they don't auto-load):
- ~/projects/research/agentic-research/concepts/hce-evaluation.md
```

This is a **declaration, not a load**: Claude Code `@` imports resolve
only inside the project tree, so a hub concept never enters a downstream
session by itself. `/sync-imports` turns each declared path into an
idempotent `used_by:` entry (`project_slug`, `imported_on`) on the
concept; a `retired` concept warns but is still recorded.

## Scoped rules

Detailed conventions live in `.claude/rules/` and are auto-loaded when you
touch matching paths:

@.claude/rules/experiments.md
@.claude/rules/notebooks.md
@.claude/rules/data.md

## Budget & compute

Ceilings and model roles live in `budget.yaml`. This project auto-pushes
to GitHub (`git.auto_push: true`) so overnight curation persists off-box.

@budget.yaml

## Housekeeping

- End sessions with `/wrap`. The SessionEnd hook backstops this.
- Run `/lint` weekly.
- Experiments live downstream — this meta project curates literature
  and concepts and does not link the experiment-loop skill group.

## Framework rules

Framework rules in `~/claude-system/claude/rules/` can't be `@`-imported
(imports resolve only in-tree): read `agency.md` before an autonomous
run. Skills with `respects:` do this automatically.
