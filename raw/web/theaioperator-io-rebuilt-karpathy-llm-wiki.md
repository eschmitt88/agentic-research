# I Rebuilt Karpathy's LLM Wiki Gist: What's Missing

- source_url: https://theaioperator.io/p/i-rebuilt-karpathys-llm-wiki-heres
- fetched: 2026-07-14
- author: Eugeniu Ghelbur (The AI Operator, Substack)
- published: 2026-04-29
- capture_note: retrieved via WebFetch (summarizing fetcher); argument
  structure and key wording preserved, not a verbatim byte copy.

## Core Argument

Karpathy's LLM Wiki pattern is foundational — "the wiki as the product,
the chat as the interface" — but the gist is "a conceptual blueprint,
not a runnable spec," missing critical mechanics for production systems.

## Five Critical Gaps

1. **Append-only ingest becomes stale.** The original pattern never
   updates existing pages; new contradicting sources get backlinks, not
   revisions, so pages describe outdated states with no recency signal.
   Fix: ingest must rewrite — latest evidence at top, dated entries
   preserved below.
2. **Unresolved contradictions compound.** Karpathy flags
   contradictions for manual resolution; past a few hundred sources they
   accumulate faster than anyone addresses them and the vault goes
   internally inconsistent. Fix: automatic reconciliation across pages,
   resolved by source recency, source authority, and explicit confidence
   levels; losing claims archived with explanations.
3. **Patterns remain invisible without prompting.** The wiki answers
   what is asked but never surfaces unrecognized patterns (recurring
   unnamed themes; contradictions between stated values and actual
   decisions). Fix: unsolicited synthesis runs automatically, writing
   new synthesis pages.
4. **Maintenance requires manual triggers.** On-demand maintenance
   "happens only when remembered, effectively never." Fix: scheduled
   agents — nightly close-out and filing, weekly reconciliation and
   synthesis passes, health checks for orphans and dead links.
5. **Notes optimized for human reading, not AI retrieval.** The
   contrarian fix ("AI-First Vault Principle"): in a working vault the
   LLM does most of the reading, so notes should optimize for machine
   parsing over human scanning.

## The AI-First Vault Principle

Notes include: a "For future Claude" preamble, machine-readable YAML
frontmatter, mandatory wikilinks, dated recency markers per external
claim, preserved source URLs, confidence levels, and self-contained
context for standalone retrieval. "Harder for a human to scan" but
dramatically faster for LLM parsing.

## Implementation Details

Open-source Claude Code skill (MIT, github.com/eugeniughelbur/
obsidian-second-brain): 31 slash commands for vault operations, 5
research commands (~$0.04–$0.80 per call via xAI and Perplexity APIs),
scheduled agents logging changes to daily diffs with 24-hour
reversibility windows, plain markdown + YAML frontmatter,
Obsidian-native.

## Key Design Principle: Reversible Automation

Automation must be "reversible" rather than "everything always." After
a synthesis command wrote garbage to the vault (misconfigured API
models), the author added mandatory 24-hour review periods before
scheduled changes become permanent.
