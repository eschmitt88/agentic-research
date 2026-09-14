---
kind: paper
title: "Scanning the Harness: An Empirical Study of Supply-Chain Defects in AI Coding-Agent Configurations"
authors: ["Benjamin Kapner", "Carmel Soceanu", "Alicia Petrunin", "Hofni Gartner"]
institutions: ["Red Hat", "Ben-Gurion University of the Negev (Stein Faculty of Computer and Information Science)"]
year: 2026
venue: "arXiv (cs.SE)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.07360"
code_url: "https://github.com/redhat-community-ai-tools/harness-eval"
citations: null
source: "raw/papers/kapner2026scanning.pdf"
added: "2026-09-14"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
tags: ["supply-chain", "harness", "skills", "mcp", "permissions", "static-analysis", "prevalence", "validation-protocol", "negative-result", "claude-code"]
---

# Scanning the Harness: An Empirical Study of Supply-Chain Defects in AI Coding-Agent Configurations

## TL;DR

A validated prevalence study of the **harness** (instruction files, skills,
commands, hooks, MCP declarations, subagents) across **3,171 public GitHub
repositories** (2,660 assembled setups, 511 skill collections). Once every
finding is re-derived at a pinned commit and read against its consequence,
**16.0% of setups carry a confirmed security defect** and 18.4% any confirmed
finding, against a raw detector rate of 25.5%. The defects are ordinary
drift, not attacks: unpinned MCP servers (9.8%), scoped-looking grants that
pre-approve arbitrary execution (3.1%), and skills whose `allowed-tools`
pre-approves the shell (3.8%). The composition-level exfiltration path that
motivated the tool has **zero confirmed instances**.

## Claims

- **The harness is a dependency layer without dependency hygiene.** It has
  no lockfile, no install-time check, and no vocabulary for what a component
  may do. A skill is a third-party dependency whose payload is prose that
  instructs a privileged agent.
- **Defects split by when they appear.** Unpinned servers and
  arbitrary-execution grants exist **only after assembly** (0.0% of
  collections). Shell pre-approval **ships inside published skills** (3.7% of
  collections) and travels to every installer. So supply-chain checking has
  to happen twice: once at publication, once at assembly.
- **The permission vocabulary misleads.** `Bash(python:*)`, `Bash(awk:*)`,
  `Bash(find:*)` and `Bash(sed:*)` look narrowly scoped but equal `Bash(*)`
  (`python -c`, awk `system()`, `find -exec`, GNU sed's `e` flag).
- **A spec rule the reference client does not enforce is one authors ignore.**
  The Agent Skills spec requires `name`/`description`. Claude Code loads a
  skill with neither and improvises the description from the body. A
  subagent with no description is skipped silently, with the reason written
  only to a debug log.
- **Only file-level rules survive validation.** Rules that compare two files
  (cross-assistant context drift, MCP divergence, an agent declaring an
  absent skill) re-derive mechanically, then fail on consequence: the
  difference is usually intended.
- **"A prevalence figure taken from unaudited detector output is a
  measurement of the detector."** Prior per-skill rates (e.g. 26.1% of 31,132
  marketplace skills, Liu et al.) are of the same order and subject to the
  same inflation.
- **Recommendation is not review.** The 2,100 setups found through
  community-curated lists carry confirmed defects at **18.9%**, against 18.4%
  overall.

## Methods

- **Instrument:** `harness-eval`, an open-source deterministic static
  analyzer (no model in any rule; terminal/JSON/SARIF output). It parses
  Claude Code, Cursor, Copilot, Gemini CLI, OpenCode, Windsurf, Cline and
  Codex layouts into a typed component graph. Edge extraction separates
  invocations from prose mentions.
- **Scope taxonomy (31 measured rules):** FILE (20), FILE_FS (7), PAIRWISE
  (4), SETUP (whole-graph flows). A per-file marketplace scanner is limited by
  construction to FILE and part of FILE_FS.
- **Rule admission:** the predicate must be decidable from bytes, never from
  a judgment about prose. The consequence must fall in one of four families:
  security (S, 15 rules), cannot-work (Q, 12), spec departure (P, 2) or
  cross-assistant inconsistency (C, 2).
- **Corpus:** 9,295 candidate repositories were found via topic queries,
  README references, 30 curated lists and the Claude Code plugin
  marketplaces. Excluded: 3,065 with no component, 1,633 instruction-file-only,
  1,322 with a single component type, 65 duplicates or forks, 22 failures,
  plus 17 dropped on rescan. Result: 3,171 repositories. The two strata are
  never pooled.
- **Four-pass validation:**
  1. The scanner reports.
  2. A second implementation, written from rule documentation rather than
     code, re-derives **every** finding: 8,547 findings across 1,115
     re-cloned repositories.
  3. Disagreements go to a Claude adjudicator with a released prompt and
     fixed reason codes: 744 pairs, 586 agreed, 158 adjudicated, 640 counted.
  4. A separate model session with web access, blind to earlier verdicts,
     re-reads all 743 counted pairs against the repository and current
     platform docs.
- **Tiers:** a rule is *gating* at ≥97% agreement on ≥50 findings with ≥80%
  of pairs ending as defects (6 rules). *Provisional* (1 rule) and
  *observation* (6 rules) tiers carry no headline figure. Of the rest, 1 rule
  fell below the agreement bar, 9 had too few findings, and 8 had none.
- **Consequences are dated** against platform docs as of September 2026, and
  version changes are recorded.

## Results

- **Setups (n=2,660):**
  - Any security defect: 16.0% (95% CI 14.6–17.4).
  - Cannot-work: 0.8%.
  - Spec departure: 2.4%.
  - Any S or Q defect: 16.7% (15.4–18.2).
  - Any confirmed finding: 18.4% (17.0–19.9); 18.8% including provisional.
  - Raw gate output before audit: 25.5%.
- **Collections (n=511):**
  - Any security defect: 3.7%.
  - Any confirmed finding: 6.8%.
  - Raw rate: 9.6%.
- **Unpinned MCP package, 9.8% of setups** (typically `npx -y
  @scope/server`, or `uvx`/`docker` with no tag or digest):
  - 961 findings at 100.0% agreement; 272 pairs, 260 counted.
  - It is live: in a seeded sample of 40, **29** ship a component that names
    the server or its tools.
  - This is the pattern vendor docs themselves show.
- **Arbitrary-execution grant, 3.1%:**
  - 83 setups with 161 findings. By mechanism: 19 unrestricted shell, 54
    interpreter, 18 package runner, 58 shell-escape tool, 10 shell by name.
  - Of 65 re-cloned setups, **22** invoke the granted tool from an
    instruction context. The authors admit this cuts both ways: many wanted
    the interpreter run.
  - Grants on `curl`/`wget`, `make`/`docker`/`ssh`, and bare `Edit`/`Write`
    are advisory only. The rule is silent on `Bash(git:*)`.
- **Skill pre-approves shell, 3.8% of setups and 3.7% of collections:**
  - 898 `allowed-tools` entries re-derived; 121/121 pairs counted.
  - The only security class visible at publication time.
- **Below the figure bar:**
  - Committed `.claude/settings.local.json`: 0.7%, provisional (22 findings,
    19/22 pairs).
  - Committed `defaultMode: bypassPermissions`/`dontAsk`: **0.4%**. Claude
    Code honoured it from project scope **until v2.1.257**, which changed
    during the study. It still takes effect on older clients.
  - `enableAllProjectMcpServers`: 0.8%, advisory.
  - Committed hooks: 6.8%, reported as an execution surface rather than a
    defect.
- **Cannot work:** subagent with no description, 0.8% (22/23).
- **Spec departure:** skill with no frontmatter block, 2.3% of setups and
  3.5% of collections. The consequence was **corrected mid-study** from
  "never loads" to "loads on an improvised description" after the
  stage-four reviewer objected.
- **Observation tier (Table 3), share of pairs that were real defects:**
  - Broken `@import` in context file: 11/23 (48%). The usual benign cause was
    a file created at runtime.
  - Context drift across assistants: 29/71 (41%).
  - MCP divergence: 8/22 (36%).
  - Agent declares absent skill: 3/7 (43%).
  - MCP config invalid: 1/13 (8%), usually a shipped template.
- **Reference-does-not-resolve** fires on 33.1% of setups and 64.0% of
  collections. By consequence it splits roughly into 45% dead paths, 20% real
  files at the wrong path, and 40% **planned outputs** ("write the plan to
  `plans/current.md`"). No format lets a skill declare outputs, so the paper
  proposes a `creates:` frontmatter field.
- **Validated negative result:** the credential-to-network delegation rule
  fired on 6 repositories and every one was read. None was exfiltration: one
  was a token counter matching the pattern, and the others sent keys to the
  vendor that issued them.
- **Harness shape:** the median setup has 6 components and 7,057 tokens (max
  500 components, 1.61M tokens). Median always-loaded content is 1,198 tokens
  in setups vs 189 in collections. 17.5% configure more than one assistant.
  Claude Code is detected in 1,887 repos.
- **Reviewer agreement:**
  - Final table: 93.3% of 743 pairs (κ=0.76).
  - Pairs both implementations agreed on: 99.3%.
  - Adjudicated pairs: only **70.7% (κ=0.23)**.
  - On the 18 adjudicated pairs touching headline figures: 10/18. All
    disagreements are unpinned packages in templates or fixtures, and move
    the headline by at most 0.3pt.
  - The adjudicator made **nine** visible errors; the reviewing model
    reversed itself twice.

## Critique / open questions

- **No human has scored any verdict.** The consequence judgment rests on
  one model under one prompt, checked by a second model session. Agreement
  on the adjudicated subset is weak (κ=0.23). The protection is that
  adjudication touches only 158 of 744 pairs and the gating rules are mostly
  agreed-by-construction (4 of 6 gating rules had zero adjudications). The
  authors ship a scoring table and script for a human pass nobody has run.
- **Both implementations come from the same team.** A shared misreading of
  a format passes both passes. Agreement bounds implementation error, not
  specification error.
- **Recall is unmeasured.** Every figure is a lower bound on what 6 file-level
  rules can express, and the selection bar favours byte-decidable rules. "Only
  file-level rules survive" is partly produced by the method: a PAIRWISE rule
  is judged on intent, which bytes cannot show.
- **Two internal inconsistencies in a v1 preprint.** The introduction gives a
  raw instrument rate of **96.8%**, while the abstract, Table 2 and §4.1 give
  **25.5%**. §4.7 says the exfiltration rule fired on **6** repositories, then
  accounts for "one" plus "the other four". Neither changes a headline, but
  both are the kind of thing peer review catches.
- **The corpus is supply-side by design:** lists, marketplaces and topics.
  It over-represents configurations that are advertised for others to use.
  The authors expect the unpinned-server rate to *rise* in a path-based
  corpus. Private enterprise configs are unobserved.
- **Rates are version-dated.** `bypassPermissions` changed meaning during the
  study, so rates like these decay as clients change.
- **Static analysis cannot say whether a component is good.** The paper cites
  Kevin et al. (ACES, arXiv:2608.20614): across 145 skills, structural scans
  and measured live effect correlate at **0.14**. That number is second-hand
  and not re-verified here.
- **Source independence.** [[literature/papers/madatha2026deterministic]]
  already argued that the agent-config layer is an unmanaged supply chain:
  6,145 config files, 10.1% cross-repo duplicates, <1% declaring permission
  boundaries. This paper is an independent team with an independent
  instrument, and neither cites the other. But both sample **public GitHub
  agent configs**, so the populations likely overlap. This is corroboration
  by a stronger method, not an independent sample. Kapner also excludes
  instruction-file-only repositories (1,633), which are exactly Madatha's
  focus, so the two partly measure different layers.

## Trust signals

- **Credibility:** 4 — industry lab (Red Hat) plus a university co-affiliation.
  arXiv preprint, not peer-reviewed, no citations yet (published 7 Sep 2026).
  Raised by the strongest reproducibility package in this cluster: the
  instrument, corpus manifest with pinned commits, adjudication prompt,
  per-finding verdicts, the independent review table and regeneration
  scripts are all released. Also raised by an exhaustive (not sampled)
  re-derivation, and by candour: it corrects its own stated consequences,
  reports a null on the motivating threat, and lists the adjudicator's
  errors. Held below 5 by no human scoring, a same-team second
  implementation, and the v1 arithmetic inconsistencies.

## Follow-up

- **Relevance:** 4 — the first *validated* prevalence measurement of the
  exact artifact class `~/claude-system` distributes (skills, hooks, settings
  permissions, context-file `@`imports). It materially extends
  [[concepts/permission-gate-as-architecture]] (the grant language is itself
  a defect site, and a pre-approval is a path around the gate carried by a
  third-party artifact). It also answers two open questions in
  [[concepts/shared-skill-namespace]] with numbers (trust/permission
  verification, and spec vs client conformance). Not a 5: it is
  software-engineering-agent infrastructure, not research-agent
  architecture, and it does not anchor a load-bearing concept alone.
- **Check this repo's own harness against the gating rules.** A quick manual
  pass (2026-09-14) found claude-system clean on all six:
  - `settings.json` allows only `WebSearch`/`WebFetch`.
  - No skill declares `allowed-tools`.
  - No MCP servers are configured.
  - Every `SKILL.md` has a `description:`.

  Running the released `harness-eval` (SARIF, seconds) would make that
  check reproducible rather than eyeballed.
- **Possible broken import, flagged, not verified.** The import contract in
  this project's `CLAUDE.md`, and `mle-bench/CLAUDE.md` lines 23–26, write
  `@import ~/projects/...`. Claude Code's import syntax is `@<path>`, so the
  literal `@import` token may resolve to a file named `import` and silently
  load nothing. That is the paper's "broken @import" class, and exactly its
  "the author never hears about it" failure. If true, `/sync-imports` records
  `used_by:` for concepts that never reach the downstream context. Verify
  with `/memory` or a debug-log check in mle-bench before relying on
  `used_by:` as a load-bearing signal.
- **The planned-output finding applies to our skills.** `/digest`, `/wrap`,
  `/ingest` and others name paths they *create* (`raw/_candidates/...`,
  `journal/YYYY-MM-DD.md`). Any static reference check would misflag them.
  The proposed `creates:` field is a cheap convention worth watching.
- **Watch** for a human-scored pass over the released review table. The
  κ=0.23 on adjudicated pairs is the number that would move.
