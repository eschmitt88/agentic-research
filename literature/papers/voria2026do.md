---
kind: paper
title: "Do They Do as They Are Told? Investigating Coding Agents' Adherence to Explicit Instructions in Context Files"
authors: ["Gianmario Voria", "Yutaro Kashiwa", "Gemma Catolino", "Fabio Palomba"]
institutions: ["University of Salerno", "Nara Institute of Science and Technology"]
year: 2026
venue: "arXiv 2611.13709 (cs.SE), v1 2026-11-10; preprint in ACM TOSEM format (template placeholders, not accepted)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.13709"
code_url: "https://github.com/gianwario/Agents-Context-Files-Analysis"   # named in the paper; returned 404 on 2026-11-17
citations: null
source: "raw/papers/voria2026do.pdf"
added: "2026-11-17"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
tags: ["agents-md", "claude-md", "copilot-instructions", "agent-context-file", "instruction-compliance", "mining-study", "aidev", "observability", "measurement-scripts", "codet", "llm-generated-checks", "bimodal-compliance", "applicability-filter", "permissive-default", "claude-code", "copilot", "codex", "kruskal-wallis", "cliffs-delta", "single-annotator"]
---

# Do They Do as They Are Told? Investigating Coding Agents' Adherence to Explicit Instructions in Context Files

## TL;DR

An observational mining study. It joins an existing ACF corpus
(Chatlatanagulchai et al., TOSEM) with agent-authored commits from AIDev
(Li et al.): 621 unique ACF snapshots linked to 6,541 commits in 268
repos, for Copilot, Claude Code and OpenAI Codex. A local Qwen 3.5 9B
extracts candidate instructions and labels each explicit / actionable /
observable. A local Qwen2.5-Coder-14B writes a compliance script per
instruction, CodeT-style (10 scripts × 10 fixture pairs, accept on ≥2
fixtures passed plus ≥60% overlap with a second script). Scripts then run
on each "applicable" commit's before/after file contents.

- **The funnel is the most robust result.** 14,902 candidates → 6,552
  explicit + actionable + observable (44.0%, LLM labels; an upper bound)
  → 4,123 with an accepted script (62.9%) → 2,053 that ever meet an
  applicable commit. Only 13.8% of candidate instructions yield any
  compliance observation.
- **Compliance is bimodal per instruction.** The mean is 59.8% and the
  median 100%. 1,116 (54.4%) are followed in every applicable commit,
  766 (37.3%) in none, and 171 (8.3%) fall in between. At ≥5 applicable
  commits (n = 819), 86.2% are still at 0% or 100%.
- **The category and agent differences are small, and most are fragile.**
  Only Implementation Details > Testing survives every evidence threshold
  (Cliff's δ = 0.15). Development Process is lowest (mean 34.0%, median 0,
  n = 50), but its gap does not survive the sensitivity check. Copilot
  (66.0%) > Claude Code (52.2%), δ = 0.12–0.15, robust. Claude Code and
  Codex do not differ.

What it cannot separate: an agent ignoring a rule, a wrong script, a
pre-existing file state, and a rule that did not apply to that commit.
About three in ten accepted scripts are wrong on manual check (67/94).

## Claims

- Most observable ACF instructions can be turned into executable checks
  (62.9%; 62.1–63.5% by agent).
- Adherence is "a switch", not "a dial". Non-compliance concentrates in
  never-followed instructions, so spot-checking a few rules says little
  about the rest.
- ACF governance "works best for code-level conventions, and less well for
  testing and process-oriented guidance".
- Process-only directives ("run the linter before committing") leave no
  artifact trace. Authors should prefer artifact-checkable rules, and tool
  vendors should flag non-observable ones at authoring time.
- Tentative (flagged by the authors as not statistically solid): workflow
  conventions such as commit formatting and branching "may be better
  encoded in project tooling, such as commit hooks, branch-protection
  rules, or CI configuration, than by relying on prose".
- Verification "cannot be agent-agnostic by default".
- Reported non-compliance is "a floor, not a ceiling", because scripts
  default to permissive. (Disputed below.)

## Methods

- **Data.** ACF snapshots are deduplicated by content and linked to every
  commit made while that version was active. 621 usable snapshots remain
  (median 638 words, 110 lines). Of 21,513 commits, 6,541 in 268 repos
  have both context and file-level data. The median commit changes 1 file
  (mean 3.95), across 26 languages.
- **Attribution.** AIDev heuristics: bot accounts, co-author trailers
  (Claude Code), branch naming (Copilot). Only the "strict agent" tier is
  used. No precision figure is given ("without claiming a validated
  precision figure"). The agent axis in RQ2/RQ3 is the *originating agent
  of the ACF snapshot* (Tables 8–9). The paper never states that the
  committing agent is the one whose file it is, or that the agent loads
  that file.
- **RQ1 extraction.** Calibrated on 15 snapshots. Validation: one
  annotator (the first author) on 84 stratified snapshots, 2,113 true
  positives. Precision 0.963, recall 0.999 (one miss). Label accuracy:
  explicit 0.955, actionable 0.973, category 0.966, **observable 0.850**
  (over 90% of errors LLM-yes → human-no).
- **RQ2 scripts.** Input is `files_before`, `files_after`, `message`,
  `branch`, with **no diff**. Both thresholds came from one pilot. The
  same annotator spot-checked 94 accepted scripts.
- **RQ3 applicability.** File extensions or basenames are taken from the
  winning fixtures. A commit is applicable if it touches any matching
  file. Fixtures with no files mean the rule concerns message or branch
  and applies to every commit.
- **Statistics.** One compliance rate per instruction, so pairs are not
  pseudo-replicated. Kruskal–Wallis, then Bonferroni-corrected
  Mann–Whitney U tests and Cliff's δ. A sensitivity rerun keeps only
  instructions with ≥3 or ≥5 applicable commits. There are no CIs on the
  rates and no repo-level clustering model.

## Results

Funnel (Tables 2, 6, 8): 14,902 candidates; explicit 10,324 (69.3%);
+ actionable 8,726 (58.6%); + observable 6,552 (44.0%); accepted script
4,123; ≥1 applicable commit 2,053, over 18,594 instruction–commit pairs.

- **The human cut observability far harder than the LLM did.** On the
  validated sample, 585 of 1,145 explicit + actionable instructions
  (51.1%) were observable. Corpus-wide, the LLM kept 6,552 of 8,726
  (75.1%). If the sample is representative, about a third of the
  instructions that entered RQ2 are not artifact-checkable by the
  authors' own standard (our arithmetic). Build and Run falls from 8.8%
  of explicit + actionable to 2.9% of the final set.
- **Exclusions (n = 2,429).** 1,006 had no script passing any fixture,
  997 no corroborating script. Operationalization is lowest for Management
  (47.7%) and Security (48.5%), highest for Config & Environment (75.6%).
- **Script validity.** 67/94 = 71.3% (Wilson 95% CI about 61–80%, our
  computation). The commonest error hard-codes an example from the ACF (a
  specific filename or type name) as a literal requirement. That makes
  the script stricter and biases compliance down. 2,569 of 4,123 accepted
  scripts (62.3%) return True on a commit with no relevant files.
- **Evidence per instruction.** The median is 3 applicable commits (mean
  9.1). 540 (26.3%) have exactly 1. Coverage by category ranges from
  Debugging 21.1% to Management 77.4%. By ACF agent it is 48.0–52.6%.

Compliance by category (Table 8; evaluated n, mean, median):

| Category | n | Mean | Median |
|---|---|---|---|
| Implementation Details | 1,022 | 63.6% | 100% |
| Documentation | 340 | 60.2% | 100% |
| Testing | 270 | 49.2% | 37.5% |
| Config & Environment | 99 | 56.3% | 100% |
| Build and Run | 62 | 69.9% | 100% |
| Development Process | 50 | 34.0% | 0% |
| Build | 48 | 67.5% | 100% |
| Architecture | 40 | 38.2% | 0% |
| Quality | 37 | 63.5% | 100% |
| Management | 24 | 44.6% | 0% |
| Security | 18 | 58.6% | 98.7% |
| Maintenance | 15 | 85.3% | 100% |
| Debugging | 4 | 21.9% | 0% |

- Across 18 categories, H = 59.13, p < .001. Four of 153 pairs survive
  Bonferroni. Development Process is below Build and Run (δ = −0.39),
  Documentation (−0.29) and Implementation Details (−0.32), and
  Implementation Details is above Testing (δ = 0.15). At ≥3 (n = 1,144)
  and ≥5 (n = 819) only Implementation Details > Testing survives.
  Development Process shrinks to 14 and then 7 instructions (means 25.0%,
  21.4%).
- **By agent (Table 9).** Copilot: n = 1,137, mean 66.0%, median 100%.
  Claude Code: n = 799, 52.2%, median 66.7%. Codex: n = 117, 51.2%,
  median 100%. H = 41.58, p < .001. Copilot > Claude Code at p_Bonf <
  .001 / .002 / .004 (full / ≥3 / ≥5). Copilot > Codex only in the full
  sample (.018; then .165, .102). Claude Code vs Codex δ = −0.0005.

## Critique / open questions

- **"Never followed" is not shown to mean "ignored".** The hub asked
  whether it could be "never applicable / not observable". The paper's own
  numbers say a large part of it plausibly is:
  1. *Wrong scripts.* 29% fail the spot-check, mostly by being stricter.
     A deterministic too-strict script scores 0% on every commit.
  2. *Final-state checks, not diffs.* A rule like "no print statements"
     fails every commit that touches a file with legacy prints the agent
     did not write. A file that already complies passes with no agent
     effort, and there is no counterfactual (the authors concede both).
     Commits in one repo touch the same files again, so verdicts repeat.
     That is a mechanism for bimodality that involves no agent behaviour.
  3. *Coarse applicability.* Any `.py` commit is "applicable" to every
     Python rule. A conditional rule ("add tests for new features") scored
     on a docstring fix fails if its script demands a test file. Testing's
     37.5% median fits this reading.
  4. *Not really observable.* By the human's standard, about a third of
     scripted instructions were not artifact-checkable (51% vs 75%).
  5. *Never loaded.* Nothing checks that the committing agent reads that
     ACF filename.
- **The "floor, not a ceiling" claim does not follow.** Leakage pushes
  compliance up only for the 62.3% of scripts that are permissive by
  default. The other 37.7% (about 1,554) do not return True on irrelevant
  commits, so leakage pushes them toward 0%. With the stricter-script
  error, the bias direction is unknown. Threats admits the net effect
  can't be quantified, yet §4.2.2 and §5.2 still say "floor".
- **No RQ3 verdict was checked by a human.** Validation covers extraction
  (84 snapshots) and scripts against their *spec* (94 scripts). No
  instruction–commit verdict was compared with a human reading of the
  commit. Both validations used the same single annotator.
- **The hub's question is mostly unanswered.** There are no examples of
  always- or never-followed instructions and no content analysis of the
  766 at 0%. The bimodality is *within* categories, not between them. The
  only robust category gap is small (δ = 0.15), and Development Process
  (the workflow rules the authors would move into hooks) rests on 50
  instructions and does not survive the check.
- **The agent comparison is confounded beyond what the authors list.**
  The agent axis is the ACF's type, not a verified committer. The
  attribution signals differ by agent. Claude Code commits found via a
  co-author trailer likely come from human-in-the-loop sessions where the
  human may have edited. Copilot's are PR-agent commits. Category mix is not
  controlled (acknowledged). "Copilot follows its file better than Claude
  Code" is an association at δ ≈ 0.15. No model versions are recorded.
- **The design advice is untested here.** "Imperative, one behaviour per
  rule, avoid 'when appropriate'" is offered, but wording is never related
  to compliance (tsuchida2026do tests wording, controlled).
- **Pipeline choices.** 9–14B quantized local models, both thresholds
  fixed on one pilot, no CIs, and no repo clustering in the tests (2,053
  instructions from at most 268 repos).
- **Not shown:** a trustworthy rate for any single instruction; a
  counterfactual (commits without the ACF, or before the rule was added);
  any effect of ACF length or rule position, though the data had it;
  whether hooks or CI on the same rules change anything; and process
  rules, which were filtered out by design.
- **Relation to the graph.** Field data beside the controlled single-rule
  studies ([[literature/papers/mcmillan2026instruction]],
  [[literature/papers/tsuchida2026do]]). Its 59.8% mean sits in their
  55–78% band, but the measures differ, so the match proves nothing. The
  funnel fits [[literature/papers/xu2026stale]]'s rules-vs-descriptions
  split: most ACF text is not a checkable rule.

## Trust signals

- **Credibility:** 3. Palomba and Catolino's SeSa lab (Salerno) and
  Kashiwa (NAIST, a co-author of the source ACF dataset) are established
  empirical-SE groups. Local models make it reproducible in principle, and
  the threats section is candid about the main measurement risks. Against: an unreviewed preprint
  in TOSEM format with template placeholders ("Received 20 February
  2007"). The named replication repo returned 404 on 2026-11-17. Single
  annotator throughout. The instrument is 71% valid on spot-check. No
  human check of verdicts and no CIs. One headline interpretation ("floor")
  contradicts the paper's own numbers. Not 4: no review, artifacts not
  reachable, and a measurement chain with an unvalidated last step. Not
  2: reputable groups, large corpus, sensitivity analysis and stated
  limits.

## Follow-up

- **Relevance:** 3. It is the only instruction-level, in-the-wild study of
  CLAUDE.md/AGENTS.md compliance in the graph, and it bears on the box's
  P3 rule ("a rule that must hold goes in code"). Its contributions are
  cite-worthy but shift little. (a) The funnel: only 13.8% of candidate
  ACF instructions can be audited from artifacts, and process rules
  ("run the linter") leave no trace. (b) Per-rule bimodality, which, if
  real, means spot-checks give no assurance. (c) The authors' tentative
  recommendation to move workflow conventions into hooks and CI. It does
  not say which categories reliably fail (one robust small gap), and its
  "never followed" bin cannot be separated from measurement error. Not 4:
  the controlled studies already carry the P3 case better, and this one's
  compliance numbers have unknown bias direction.
- **For the box.** The lesson is *where* a check can sit. A rule whose
  execution leaves no artifact ("run X before committing", "read Y
  first") can't be audited post hoc. It needs a check at the action (a
  hook that sees the command), or it is unenforceable. A post-hoc check
  suits artifact rules only, and should read the diff, not the final
  file state.
- **Cheap local check, if wanted.** Run this paper's design on the box's
  own git history. Commits carrying the Claude co-author trailer, against
  the CLAUDE.md version at that commit, with hand-written diff-level
  checks for the few artifact rules in the user-level CLAUDE.md (no
  `model.pt`/parquet committed, the attribution trailer present). Small
  n, but a verdict per rule you can trust.
- **Candidates** (cited, not yet in the hub):
  - Li, Zhang & Hassan 2025, "The Rise of AI Teammates in SE 3.0" (AIDev,
    arXiv 2507.15003), the attribution dataset.
  - Lulla et al. 2026, "On the impact of AGENTS.md files on the efficiency
    of AI coding agents" (arXiv 2601.20404): −28.6% runtime, −16.6% tokens.
  - Chatlatanagulchai et al. (Agent READMEs) and Yang et al.
    (RepoComplianceBench) are already tsuchida2026do candidates.
