# agentic-research

**Curated literature and importable architectural concepts for building autonomous research agents.**

📚 **[Browse the knowledge graph →](https://eschmitt88.github.io/agentic-research/)** —
an interactive, always-live view of every concept, literature note, and map of content.

## What this is

A "research-about-research" knowledge base: a distilled, cross-linked graph of the
ideas, papers, and patterns behind autonomous research agents — primarily
ML-research agents (MLE-bench, PaperBench, AIRA) with selective coverage of general
scientific-research automation where the architecture transfers.

It also serves as the **hub** of a personal research framework: downstream projects
`@import` concept and literature notes from here by path, so these notes are a read
interface, not just internal artifacts. Each concept carries `used_by:`
back-references showing which ideas are actually load-bearing across projects.

## How it's organized

Plain Markdown + flat YAML frontmatter, cross-linked with `[[wikilinks]]`:

- `concepts/` — atomic, importable ideas; promoted to `mocs/` when ≥5 cluster on a theme.
- `literature/` — processed notes on papers, repos, and posts (0–5 relevance scored).
- `mocs/` — maps of content stitching concept clusters into a navigable theme.
- `experiments/` — dated runs (this repo is literature-led; most work is curation).
- `raw/` — immutable source captures · `docs/decisions/` — ADRs · `_meta/` — index, log, templates.

The [browsable site](https://eschmitt88.github.io/agentic-research/) renders all of
this live from the repo — no build step, no regeneration.

## Local use

```sh
make env    # uv sync
make lint   # knowledge-graph health check (orphans, dead wikilinks, sourceless concepts)
```

Part of a personal research framework
([claude-system](https://github.com/eschmitt88/claude-system)). See `CLAUDE.md` for
the agent-facing orientation and `~/.claude/CLAUDE.md` for the framework's durable
principles.
