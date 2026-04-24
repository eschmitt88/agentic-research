---
kind: concept
name: "web-grounded-literature"
status: active
added: "2026-04-24"
source_papers:
  - nam2025mle
  - mitchener2025kosmos
sources:
  - "[[literature/papers/nam2025mle]]"
  - "[[literature/papers/mitchener2025kosmos]]"
used_by: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
related_experiments: []
tags: [retrieval, search, ingest, discovery]
---

# web-grounded-literature

## Definition

An autonomous research project continuously ingests primary sources
from the web — arXiv, GitHub, blog posts — rather than relying on the
LLM's training-time knowledge. `/discover`, `/fetch-paper`, and
`/digest` are the three skills that realize this as continuous
intake.

## Why it matters

MLE-STAR ([[literature/papers/nam2025mle]]) finds that LLM-only ML
agents are bottlenecked by stale training-time knowledge; seeding
an initial solution from web-retrieved papers materially improves
downstream performance. Kosmos ([[literature/papers/mitchener2025kosmos]])
operates at the next scale up — ~1,500 papers reviewed per 12-hour
run — and treats literature retrieval as a peer specialist to data
analysis, not an optional enrichment step.

The shared insight: model priors drift stale within months of
deployment, while the web does not. A research loop that reads only
its own memory converges on yesterday's ideas.

## Implementation guidance

1. **Three skills, one workflow.**
   - `/discover <topic>` — focused, user-initiated triage.
     Produces `raw/_candidates/YYYY-MM-DD-<slug>.md` with ranked
     candidates and one-line justifications. Does not download full
     PDFs.
   - `/fetch-paper <arxiv-id-or-url>` — downloads to `raw/papers/`
     with a derived citekey, then chains into `/ingest` to produce
     the literature note.
   - `/digest` — unattended periodic sweep. Reads the project's
     current active concepts and recent iteration log, runs queries
     against the web, drops new candidates into
     `raw/_candidates/YYYY-MM-DD-digest.md`. Updates
     `_meta/last_digest`. Never ingests — curation is the user's job.

2. **Schedule `/digest` on a cron.** The meta project runs it Monday
   7am weekly via `~/.claude/schedule/agentic-research-digest.sh`.
   Downstream projects inherit the pattern; tune cadence to project
   velocity.

3. **`raw/` is immutable.** Every ingested source lives here and is
   never edited. Re-ingest rather than mutate.

4. **Citations flow from literature to Diagnostics.** When a claim
   in an experiment's Diagnostics section traces to a paper, the
   anchor is a wikilink into `literature/`, not a URL. The URL
   lives in the literature note's frontmatter only.

## Open questions

- `/digest` queries are synthesized from concept filenames + tags;
  whether query synthesis should use a stronger model (vs. string
  templates) to surface less-obvious connections is an open
  question.
- Deduplication against prior candidates is O(n) in candidate-file
  count today. At multi-year scales the dedup cost will dominate
  `/digest` latency — worth an index revisit then.
