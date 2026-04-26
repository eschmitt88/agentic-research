---
kind: paper
title: "Skilldex: A Package Manager and Registry for Agent Skill Packages with Hierarchical Scope-Based Distribution"
authors:
  - Sampriti Saha
  - Pranav Hemanth
year: 2026
venue: "arXiv:2604.16911 [cs.SE]"
url: "https://arxiv.org/abs/2604.16911"
source: "raw/papers/saha2026skilldex.pdf"
added: "2026-04-26"
relevance: 2
status: skimmed
related_experiments: []
related_concepts: []
tags:
  - agent-infrastructure
  - package-management
  - claude-skills
  - mcp
  - hierarchical-scoping
  - format-validation
---

# Skilldex: A Package Manager and Registry for Agent Skill Packages with Hierarchical Scope-Based Distribution

## TL;DR

Skilldex is a TypeScript CLI + community registry for distributing
LLM agent *skill packages* (Anthropic's `SKILL.md` format). Two
novel pieces: (1) compiler-style format conformance scoring against
Anthropic's published spec (0–100, line-level diagnostics); and
(2) a *skillset* abstraction that bundles related skills with shared
assets (vocabulary, templates, references) to enforce cross-skill
coherence. Supporting infra: three-tier scope (global / shared /
project) with local-first precedence, an MCP server exposing all
ops to agents natively, and a metadata-only registry with skills
hosted on GitHub.

## Claims

- Existing skill-distribution tooling has two gaps: no
  spec-grounded conformance signal (a malformed skill installs the
  same as a well-formed one) and no mechanism for grouping skills
  whose behavior depends on shared context.
- Format conformance scoring is *not* a measure of functional
  quality — it's a measurable proxy for one impactful publisher
  lever (description specificity, frontmatter validity, structural
  adherence). The paper foregrounds this disclaimer.
- Skill-format ownership should stay with the spec author
  (Anthropic). Skilldex versions its scorer (`spec_version`) but
  does not make normative format decisions.
- Explicit human-in-the-loop checkpoint before capability expansion
  beats post-hoc capability review.

## Methods

- **CLI** (`skillpm` / `spm`) and **MCP server** share core modules
  (Installer, Scope Resolver, Validator, Manifest I/O, Suggest
  Agent, Registry Client).
- **Three scope tiers**: `global` (`~/.skilldex/`), `shared`
  (`~/.skilldex/shared/`), `project` (`<root>/.skilldex/`).
  Resolution is *local-first* — lower scope shadows higher.
- **Validator** scores 8 checks summing to 100 (frontmatter parse:
  25, name: 10, description ≥30 words: 10, line count ≤500: 15,
  allowed subdirs: 10, referenced resources exist: 15, resources
  in correct subdirs: 5). Missing frontmatter is the only fatal
  case.
- **Diagnostics** mimic compiler output: severity (error / warning
  / pass) + optional line number + message; `--json` flag for
  CI consumption.
- **Skillset** is a bundled directory of related skills + a shared
  `assets/` (vocab files, templates, reference docs).
- **Registry** is metadata-only: PostgreSQL via Supabase; skill
  files fetched from GitHub at install time. Two trust tiers:
  `verified` (manual, reserved for Anthropic-official) and
  `community`.
- **Suggestion loop** reads `README.md` (first 100 lines),
  `package.json`, agent config, then proposes scoped installs that
  the user approves interactively (or `--yes` for CI).

## Results

- The paper is a system-paper / artifact release; no benchmark
  numbers. Open-source TypeScript implementation.
- No quantitative evaluation of whether the conformance score
  correlates with skill effectiveness — left as future work.

## Critique / open questions

- Conformance scoring's biggest claim — that it's a useful proxy
  for skill quality — is asserted but not measured. A study
  correlating score with task-success rate would be the natural
  validation.
- The skillset abstraction is interesting but its incremental value
  over disciplined cross-references between standalone skills is
  unclear without usage data.
- The two-tier trust model (verified / community) leaves a
  large unstructured middle: a skill's score is the only quality
  signal once you're outside the verified set.
- Tangential to autonomous ML research per se; this is agent
  *infrastructure*, not agent capability.

## Follow-up

- This is the closest published artifact to compare against this
  project's `@import`-based concept-sharing contract. Both solve
  cross-project pattern reuse; Skilldex does it via packaged
  skill bundles + a registry, while this project does it via
  Markdown back-references appended to canonical concept files
  living in one meta-repo. Worth a side-by-side note if the
  meta-project ever considers a registry layer.
- The compiler-style diagnostic format (severity + line number +
  message + 0–100 summary) is a clean shape for `/lint` output —
  consider lifting that pattern into our own lint reporting.
- The local-first scope resolution (project shadows shared shadows
  global) is the same precedence Claude Code already uses for
  CLAUDE.md and skills; no new concept needed, but useful prior
  art if the project ever formalizes its `_scratch/` vs. shared
  vs. global concept-set boundaries.
