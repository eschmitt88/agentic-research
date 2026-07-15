---
kind: repo
name: "obsidian-second-brain"
url: https://github.com/eugeniughelbur/obsidian-second-brain
commit: HEAD @ 2026-07-14
source: "raw/repos/eugeniughelbur-obsidian-second-brain.md"
added: "2026-07-15"
relevance: 4
status: scanned
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: [second-brain, pkm, obsidian, claude-code, contradiction-reconciliation, scheduled-agents, freshness-policy]
---

# obsidian-second-brain

## Purpose

A cross-CLI skill (Claude Code, Codex, Gemini CLI, OpenCode, Hermes, Pi)
that turns an Obsidian vault into a self-maintaining "AI-first second
brain," explicitly framed by the author as an evolution of Karpathy's
LLM Wiki pattern (already ingested as
[[literature/posts/gist-github-com-karpathy-llm-wiki]]). 44 commands,
3,267 stars, MIT license, actively maintained (v0.12 as of July 2026,
134-test CI wall, a self-run 32-agent stress test with 175 findings /
24 fix PRs).

## Shape

- `_CLAUDE.md` — operating manual (schema file, same role as this
  project's `CLAUDE.md`).
- `raw/` — immutable source material (direct convergence with this
  project's `raw/` convention).
- `wiki/{entities,concepts,projects,daily,logs,reviews,tasks,decisions}/`
  — the LLM-maintained layer, split by note type rather than this
  project's `literature/`+`concepts/`+`mocs/` split.
- `boards/`, `templates/`, `scripts/freshness_lint.py`.
- Bootstrap script (`bootstrap_vault.py`) with four role presets
  (executive/builder/creator/researcher).
- Background agent fires on `PostCompact`; four cron-scheduled agents
  (morning/nightly/weekly/health).

## Useful bits

The README's own comparison table against "Karpathy's LLM Wiki"
names five deltas — the same five gaps the rebuild critique
([[literature/posts/theaioperator-io-rebuilt-karpathy-llm-wiki]])
argued the *original* pattern is missing, now claimed as *implemented*
here:

| Gap (critique) | This repo's mechanism |
|---|---|
| No rewrite of stale claims | `/obsidian-ingest` rewrites 5–15 existing pages per source instead of only appending |
| No contradiction reconciliation | `/obsidian-reconcile` — automatic, plus a nightly scheduled sweep |
| No unsolicited synthesis | `/obsidian-synthesize` finds unnamed cross-source patterns and writes synthesis pages unprompted |
| No scheduled maintenance | 4 cron agents (morning/nightly/weekly/health) + a background agent on every context compaction |
| Human-first notes | "AI-first" rule: every note gets a `## For future Claude` preamble; frontmatter is written for LLM retrieval, not human skimming |

Two mechanisms not named in the critique or in A-Mem
([[literature/papers/xu2025amem]]):

- **Bi-temporal facts** — every stored fact tracks both *when it was
  true* and *when the vault learned it* ("You believed X on Tuesday.
  After ingesting Y on Wednesday, you shifted to Z"), giving contradiction
  reconciliation a full audit trail instead of silent overwrite.
- **OKM (Open Knowledge Metabolism)** — a named, spec'd maintenance
  policy (`references/freshness-policy.md`, linted by
  `scripts/freshness_lint.py`): every stored fact must be *timeless*,
  *dated*, or *a pointer*. Fast-changing facts (counts, balances,
  statuses) are never copied in — they're linked to their source with
  an `as of` stamp. The author frames this as complementary to Google's
  OKF (Open Knowledge Format): OKF standardizes how agent knowledge is
  *written*, OKM standardizes how it *stays true*.

## Follow-up

**Relevance: 4** — this is the concrete implementation this project's
own `llm-wiki-pattern` note flagged as missing (see that note's "Open
questions": *"Which maintenance operations are essential vs
nice-to-have… only reconciliation has an academic co-attestation so
far"*). It doesn't resolve the question, but it's a second, independent
attestation that rewrite-on-ingest and contradiction reconciliation are
worth building, and the freshness-policy linter is a concrete design
this project doesn't currently have (`/lint` here checks graph hygiene,
not fact-staleness).

**Credibility: 3** — solo/independent build (author: AI Automation
Engineer, runs the `theaioperator.io` blog that produced item #2 in
this candidate batch), not peer-reviewed, no formal citation count. But
real evidence of production use: 3,267 stars, MIT license, active
development with a documented CI wall and a self-run adversarial stress
test (175 findings fixed), and a companion blog with weekly technical
posts. Institution/pedigree signal is weak; reproducibility/adoption
signal is comparatively strong for an independent tool.

- Consider whether `freshness-policy.md`'s timeless/dated/pointer
  taxonomy is worth citing directly in `llm-wiki-pattern`'s maintenance
  section as a concrete design for gap (1), separate from the vaguer
  "rewrite-on-ingest" framing currently there.
- Not explored: no local install/run performed, no verification of the
  contradiction-reconciliation or freshness-lint mechanisms beyond what
  the README documents.
