---
kind: concept
name: "web-grounded-literature"
status: active
added: "2026-04-24"
source_papers:
  - nam2025mle
  - mitchener2025kosmos
  - xiong2026autoresearchbench
sources:
  - "[[literature/papers/wang2026parness]]"
  - "[[literature/papers/nam2025mle]]"
  - "[[literature/papers/mitchener2025kosmos]]"
  - "[[literature/papers/xiong2026autoresearchbench]]"
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

## Realistic SOTA bounds and failure modes

AutoResearchBench ([[literature/papers/xiong2026autoresearchbench]])
provides the strongest external calibration for what these skills
can actually achieve. Frontier models max out at **~9% accuracy**
on autonomous literature-discovery tasks (Deep Research: identify
one specific target paper from obfuscated clues; Wide Research:
exhaustively collect papers matching multi-constraint spec) — a
~70 pp gap below their performance on general agentic web
browsing benchmarks. The failure-mode taxonomy is more actionable
than the headline number:

- **Specialized academic search beats general web.** DeepXiv
  full-text arXiv search outperforms Jina open-web search by
  ~1.5–2.5 pp across matched models. Decisive scientific evidence
  lives in appendices, ablations, figure captions, and citation
  contexts — invisible to abstract/snippet-level retrieval. Our
  `/digest` uses general WebSearch; the empirical finding is that
  arxiv-specialized full-text retrieval is materially better.
- **More compute does not reliably help.** Long trajectories (27+
  turns) often *underperform* short ones (6 turns); "think mode"
  is consistently *detrimental* on Wide-style tasks. Reasoning
  helps only when it directly improves *evidence acquisition*,
  not when it just adds deliberation.
- **Long natural-language queries beat keyword piles.** Short
  web-query-style preferences degrade scientific search. Digest
  query synthesis should produce 3–5-sentence scenario-rich
  queries, not `topic1 topic2 topic3` strings.
- **Design against `precision-unconstrained-expansion`.** Claude
  Opus's dominant Wide-Research failure (85.3% of its errors) is
  over-recall — expanding the candidate set without applying all
  constraints. Better one well-grounded candidate than five
  loosely-grounded ones. Our digest's small-and-filtered output
  shape (3–6 candidates with explicit rationale) is the right
  default; resist comprehensiveness pressure.
- **Curator-in-the-loop is the right pattern given this regime.**
  At ~9% SOTA, full automation isn't credible. The project's
  design — automated recall (digest produces ranked candidates),
  human precision verification (user triages and promotes via
  `/fetch-paper`) — matches what the empirical evidence supports.

## Open questions

- `/digest` queries are synthesized from concept filenames + tags;
  whether query synthesis should use a stronger model (vs. string
  templates) to surface less-obvious connections is an open
  question. AutoResearchBench's finding that long scenario-rich
  queries outperform keyword piles suggests this is worth tuning.
- Deduplication against prior candidates is O(n) in candidate-file
  count today. At multi-year scales the dedup cost will dominate
  `/digest` latency — worth an index revisit then.
- Whether to swap `/digest`'s WebSearch backend for an
  arxiv-specialized full-text retriever (a la DeepXiv from
  [[literature/papers/xiong2026autoresearchbench]]) is the most
  concrete improvement lever surfaced by external evaluation.
  Worth a downstream experiment.
