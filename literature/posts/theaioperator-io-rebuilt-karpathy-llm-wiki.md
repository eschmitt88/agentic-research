---
kind: post
title: "I Rebuilt Karpathy's LLM Wiki Gist: What's Missing"
author: Eugeniu Ghelbur
url: https://theaioperator.io/p/i-rebuilt-karpathys-llm-wiki-heres
source: "raw/web/theaioperator-io-rebuilt-karpathy-llm-wiki.md"
added: "2026-07-14"
relevance: 4
related_experiments: []
related_concepts:
  - "[[concepts/llm-wiki-pattern]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/verified-memory-writes]]"
tags: [second-brain, llm-wiki, maintenance, contradiction-reconciliation, scheduled-agents, ai-first-vault]
---

# I Rebuilt Karpathy's LLM Wiki Gist: What's Missing

## TL;DR

The strongest practitioner critique of the LLM Wiki pattern: Karpathy's
gist is "a conceptual blueprint, not a runnable spec," and an
append-only wiki goes internally inconsistent past a few hundred
sources. Five gaps — no rewriting, unresolved contradictions, no
unsolicited synthesis, no scheduled maintenance, human-first notes —
plus two design principles (AI-First Vault, reversible automation),
backed by a shipped MIT implementation (obsidian-second-brain).

## Key points

- **Gap 1 — append-only goes stale**: new evidence gets backlinks, not
  revisions; fix is rewrite-on-ingest with latest evidence on top and
  dated entries preserved below.
- **Gap 2 — contradictions compound**: manual resolution stops scaling
  past a few hundred sources; fix is automatic reconciliation ranked by
  source recency, authority, and confidence, with losing claims
  *archived with explanations*, not deleted.
- **Gap 3 — no unsolicited synthesis**: a query-only wiki abandons half
  the value; scheduled synthesis passes should write pattern pages the
  user never asked for.
- **Gap 4 — manual maintenance never happens**: nightly filing, weekly
  reconciliation/synthesis, orphan/dead-link health checks as scheduled
  agents.
- **Gap 5 / AI-First Vault Principle**: in a working vault the LLM does
  most reading, so optimize notes for machine parsing — YAML
  frontmatter, mandatory wikilinks, per-claim recency markers,
  confidence levels, self-contained context — even at the cost of human
  scannability.
- **Reversible automation**: after a misconfigured synthesis run wrote
  garbage, all scheduled changes get daily diffs and a 24-hour review
  window before becoming permanent.
- Shipped as github.com/eugeniughelbur/obsidian-second-brain (31 vault
  commands, 5 research commands at ~$0.04–0.80/call).

## Scorecard against this repo

Gap 1 — partial: `/ingest` updates concepts on arrival (and git keeps
the superseded state per agent-native-memory guidance #8), but literature
notes are effectively append-only. Gap 2 — **missing**: no contradiction
reconciliation pass anywhere. Gap 3 — **mostly missing**: `/promote-moc`
is the only unsolicited synthesis; nothing writes cross-concept pattern
pages. Gap 4 — enacted (nightly `/curate`, weekly `/lint` + `/elevate`).
Gap 5 — enacted (frontmatter, wikilinks, provenance). Reversible
automation — enacted structurally (git revert; commit-per-change).

## Follow-up

- **Relevance:** 4 — materially extends [[concepts/llm-wiki-pattern]]
  with its maintenance-operations half and supplies the failure model
  (append-only inconsistency at scale); the scorecard above turns it
  into a concrete gap list for this repo. Single practitioner source,
  no peer review — weight accordingly; A-Mem's memory-evolution
  ablation is the academic echo of gap 1.
- Candidate `/elevate` input once attested further: a weekly
  contradiction-reconciliation pass (gap 2) is the one mechanism with
  no counterpart in this system.
