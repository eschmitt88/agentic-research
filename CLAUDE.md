# Project: agentic-research

Short orientation only. User-level `~/.claude/CLAUDE.md` holds the durable
principles; this file refines them for this project.

## What this project is about

Research-about-research: curated literature and importable architectural
concepts for autonomous ML research.

## Role

This project is the research hub for every other project on this box.
Downstream projects `@import` files from `./concepts/` and `./literature/`
by absolute path (see the import contract below), so concept notes and
literature notes here are a read interface, not only internal artifacts.
When `/ingest` processes a file on a downstream project that imports a
concept from here, it appends a `used_by:` back-reference to the target
concept so the meta project can see which concepts are load-bearing.
Evolution here propagates to downstream projects at their next session
start — no copy-paste.

## Import contract

Canonical downstream usage — add lines like these to any other
project's `CLAUDE.md`:

```
# In ~/projects/research/<other-project>/CLAUDE.md

## Inherited architecture
@import ~/projects/research/agentic-research/concepts/hce-evaluation.md
@import ~/projects/research/agentic-research/concepts/citation-anchoring.md
```

On the downstream side, `/ingest` detects these `@import` directives and
appends a `used_by:` list entry to each referenced concept note with:

```yaml
used_by:
  - project_slug: <downstream-slug>
    imported_on: <YYYY-MM-DD>
```

The append is idempotent — re-ingesting does not duplicate entries for
the same `project_slug`. If the target concept's `status:` is
`retired`, `/ingest` warns on the downstream side but allows the import.

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
- No `/implement` or `/iterate` in this project — the meta project
  curates literature and concepts; experiments live downstream.
