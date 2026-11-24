---
kind: paper
title: "AI Cannot Review What Humans Catch: A First Look at the Reviews of Agent Instruction Files"
authors: ["Wenhao Yang", "Runzhi He", "Minghui Zhou"]
institutions: ["Peking University"]
year: 2026
venue: "arXiv 2611.18582 (cs.SE), v1 stamped 2026-07-31, preprint (11 pages incl. references, no appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.18582"
code_url: null    # the PDF names no data, benchmark or code release
citations: null
source: "raw/papers/yang2026ai.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["instruction-files", "agents-md", "claude-md", "cursorrules", "skill-md", "code-review", "review-coverage", "rubber-stamp", "review-hollowing", "concern-taxonomy", "nugget-recall", "ai-reviewer", "claude-code-review", "codex-review", "opencode", "necessity", "staleness", "hallucinated-reference", "mining-software-repositories", "no-human-baseline", "elevate-candidate"]
---

# AI Cannot Review What Humans Catch: A First Look at the Reviews of Agent Instruction Files

## TL;DR

A census of 2,870 GitHub pull requests that change an agent instruction
file (AGENTS.md, CLAUDE.md, .cursorrules, SKILL.md, etc.) in 2,246
repositories with >100 stars, Feb 2021 – Apr 2026. Every non-author review
comment is hand-coded, identity-blind, for whether it touches the
instruction file and whether it raises a concern. The concerns form a
16-category taxonomy. The human concerns become a gold set, and three
product-native AI reviewers are scored on how many they recover.

Four results matter for this graph:

1. **Most instruction-file changes get no evaluation of the instruction
   file.** Only 792 of 2,870 PRs (27.6%) draw a substantive comment on it.
   Another 949 (33.1%) were reviewed, but only on their code. 67.1% of PRs
   only *add* instruction content, and 69.0% ride along with other files
   (median 4).
2. **Product reviewers recover almost none of what maintainers raise.**
   Codex review (GPT-5.5 high), OpenCode review (DeepSeek-v4-pro max) and
   Claude Code `/review` (Sonnet 4.6 high) each ran once on 446 PRs, with
   the full repository at review time and web tools off. Strict recall
   of 1,912 human concerns: **OpenCode 3.9%, Claude Code 3.2%, Codex
   1.3%.** All three pooled reach 6.9%. 77.8% of PRs get zero coverage
   from all three.
3. **The necessity questions are nearly untouched.** Triviality / low-value
   concerns were recovered 0, 1 and 0 times out of 146. Audience fit got 0
   of 32, and hallucinated / fabricated references 0 of 38 for every
   reviewer. Staleness was the *best* category in relative terms (4–5 of
   69, 5.8–7.2%). But no category exceeds 7.2% for any reviewer, and most
   per-category counts are single digits.
4. **Maintainers mostly flag what is missing.** Of 2,687 human substantive
   comments, 24.6% are Completeness (a rule, command, prerequisite or
   example the agent needs is missing). Conciseness is 2.5%. Triviality
   plus Overreach ("should this rule exist?") is 9.5%. Wrong mechanism
   ("this should be a hook / linter / CI check") is 0.8%.

## Claims

- Instruction files are "executable policy" with no per-change oracle. So
  review is their only quality gate, and the census shows the gate is
  "largely broken": review is mostly absent, and AI review where present
  is "structurally blind" to judgment-dependent concerns.
- Human and AI reviewers attend to different things, not the same things
  at different rates. In the wild, AI comments lean to Structure /
  formatting (19.0% vs 2.8% of human comments), Consistency (11.6 vs 4.5)
  and Executability (8.2 vs 5.1). Humans lean to Completeness (24.6 vs
  5.4), Factual accuracy (7.8 vs 2.5), Triviality, Overreach, Stale and
  Hallucinated.
- Five categories are new relative to code, documentation and prompt
  review taxonomies: Triviality, Overreach (together, "Warrant"), Audience
  fit, Discoverability / loading, and Hallucinated.
- The low recall is "a task ceiling, not a model ceiling", because the
  three reviewers miss largely different concerns (the triple
  intersection is 3).
- The dividing axis is "checkable against an external ground truth"
  versus "requires a normative judgment about whether the instruction
  should exist".
- **Contamination**, asserted but never measured: the reviewer loads the
  instruction file under review as its own context and "in several cases,
  obeys the very rules it is asked to audit". "A constitution cannot
  coherently question itself."
- Agents review source code competently. The blindness is specific to
  executable policy. This rests on cited benchmarks (c-CRAB 20–32%, Code
  Review Bench 73–77%), not on a same-protocol run in this paper.
- Future work, a sketch only: a reviewer tuned to instruction-file defect
  classes that mines project history. It would fix surface defects itself
  and escalate necessity, overreach and hallucination questions to the
  maintainer as a structured interview.

## Methods

- **Corpus.** World of Code April 2026 snapshot (≈460k commits touching
  instruction-file paths) → GHArchive PR mapping → >100 stars (3,765
  candidates). Each PR was cross-checked against the GitHub API. Kept: PRs
  that change ≥1 instruction file, have ≥1 non-author review comment in
  the recovered data, and can be reconstructed at the reviewed revisions.
  Result: 2,870 PRs, 5,475 instruction-file instances.
- **Author origin.** Four classes from machine-emitted markers (co-author
  lines, generated-by trailers, bot accounts) and free-text
  self-reports. An audit of 150 PRs found 2.0% missed AI attribution. The
  AI share (963/2,870) is a lower bound.
- **Reviewer class.** Human / AI / Tool from account metadata. Validated on
  300 accounts: κ 0.983, 95.3% account-weighted and 98.5% instance-weighted
  accuracy.
- **Relevance.** Two deterministic rules cover 91.8% of 23,651 comments:
  an inline comment on the instruction file, or a bare approval on an
  instruction-only PR. The remaining 8.2% were hand-coded (κ 0.630).
- **Concern coding.** Two authors open-coded ~300 PRs to a codebook, then
  double-coded 99.3% of the 1,200 relevantly reviewed PRs, blind to
  reviewer identity. Substantive vs not: κ .918. Per-concern codes:
  macro κ .72, "a few judgment-heavy codes dip lower". A comment carries
  up to two concerns.
- **Gold.** Two coders marked each of 1,987 substantive comments
  KEEP / DROP / UNSURE against three evidence tiers:
  - outcome: did the author later change the flagged content?
  - discourse: "good catch" vs a correct rebuttal;
  - factual truth: checked against code, docs and web search.

  25 were dropped (1.26%), leaving 1,962 concerns across 468 PRs.
- **Benchmark.** Each PR was reconstructed at review time. Each product's
  *native* review ran on the full repository (base and head) with the
  archived PR title and body. Web tools were disabled and git history
  trimmed. One run per product. 446 PRs and 1,912 concerns were
  reconstructable.
- **Scoring.** Two human judges per AI review. "Covered" requires a
  contiguous verbatim quote raising the same concern in the same
  direction. Praise and neutral description do not count. The ranking was
  re-checked by a stricter independent re-judging (3.5%, 2.9%, 1.2%).
  Precision was not measured.

## Results

- **Funnel (Fig. 2, Table IV).** 2,870 → 2,149 reviewed by a non-author →
  1,200 relevantly reviewed (41.8%) → 792 substantive (27.6%).
  - By reviewer mix, substantive / relevant:
    - human-only: 242 / 411 of 882 PRs;
    - ai-only: 194 / 337 of 484;
    - human+ai: 353 / 446 of 530;
    - tool-only: 3 / 6 of 257;
    - silent: 0 of 717.
  - The paper's "71.9% receive no substantive review" does not match its
    own counts. 2,078/2,870 is **72.4%** (our arithmetic). The figure's
    "relevant but hollow" box says 394; 1,200 − 792 is 408.
- **Authorship (Table II).** Pure human 66.4%, AI-generated (declared)
  14.0%, AI agent / bot 12.8%, human with self-reported AI use 6.7%.
  417 PRs (15.6%) were both AI-written and AI-reviewed. In 148 (5.5%) an
  AI reviewer was the only reviewer.
- **Hollow review.** Of the 4,789 comments on relevant PRs, 66.9% are
  substantive. By reviewer type:
  - Humans are substantive 73.9% of the time. Their hollow mode is the
    rubber stamp: 18.9% of their comments.
  - AI reviewers are substantive 57.4% of the time. Their hollow mode is
    a walkthrough or PR summary, 39.5% of their comments: "review-shaped"
    with no evaluative judgment.
- **Human backstop on AI-authored files.** Among PRs reviewed by both, 139
  contained an AI-authored instruction file. In 109 (78%), the AI review
  missed at least one concern a human caught. The introduction and
  Discussion restate this as "78% of AI-authored changes still pull a
  human in". That is a different claim from the one measured.
- **Per-category recall (Table VI).** Recovered out of N, as
  OpenCode / Claude Code / Codex:

  | Concern | N | OpenCode | Claude Code | Codex |
  |---|---|---|---|---|
  | Structure / formatting | 76 | 2 | 0 | 0 |
  | Executability | 134 | 6 | 9 | 9 |
  | Consistency | 111 | 8 | 2 | 0 |
  | Factual accuracy | 199 | 8 | 10 | 6 |
  | Stale / outdated | 69 | 5 | 4 | 4 |
  | Hallucinated / fabricated | 38 | 0 | 0 | 0 |
  | Conciseness | 64 | 4 | 1 | 0 |
  | Specificity / actionability | 123 | 4 | 5 | 0 |
  | Completeness | 648 | 27 | 22 | 5 |
  | Discoverability / loading | 79 | 3 | 4 | 1 |
  | Wrong mechanism | 20 | 1 | 0 | 0 |
  | Security | 38 | 1 | 1 | 0 |
  | Scope / placement | 94 | 3 | 0 | 0 |
  | Audience fit | 32 | 0 | 0 | 0 |
  | Triviality / low-value | 146 | 0 | 1 | 0 |
  | Overreach / unwarranted | 98 | 5 | 5 | 1 |
  | **Unique concerns** | **1,912** | **75 (3.9%)** | **62 (3.2%)** | **25 (1.3%)** |

  Per-category N sums to 1,969 because concerns can carry two codes.
- **The misses mattered more than the hits.** Of concerns all three missed,
  71% (1,173/1,653 with a recorded outcome) were later acted on by the PR
  author, and 77% in the human-judgment categories. That compares with 55%
  of concerns some AI caught. The authors note that the ~70% base rate is
  partly by construction, since author action was a gold-validation
  signal.
- **Effort, not only capability.** 32% of Codex's reviews (141/446) are
  under 600 characters: single-sentence dismissals.
- **Confidently wrong (three hand-picked cases, no rate).**
  - OpenCode praised "correct folder structure" on the very nonexistent
    `services/` directory the human flagged.
  - Codex reversed two explicit author decisions.
  - Codex and Claude both asserted "Java 25 is the latest LTS" against a
    correct file.

## Critique / open questions

- **No human-vs-human baseline, so 1.3–3.9% has no reference point.** The
  gold is what one or more specific maintainers raised. Many concerns
  (Completeness, Overreach, Audience fit) turn on project intent. The
  paper never measures how many of maintainer A's concerns an independent
  maintainer B would raise on the same PR. Code-review studies typically
  find low overlap between human reviewers too. Without that arm, "AI
  cannot substitute for human attention" is not shown. What is shown is
  that default product reviews do not reproduce *these* maintainers'
  comments.
- **The tasking confound is not removed, despite the paper's claim.**
  Threats to validity says RQ4 removes "tasking rather than capability"
  by "explicitly tasking agents". But the agents ran their *native* PR
  review on PRs where 69% also change code, with the PR title and body
  only. Nothing in the method says they were told to review the
  instruction file, or to judge necessity. Product review modes are tuned
  to report a few high-confidence code defects. Codex's short dismissals
  fit that design. So this measures default product behaviour, not what a
  model given an instruction-file review rubric could do.
- **The asymmetry runs against the agents.** Web tools were off, while the
  gold's validators "search[ed] the web". Factual and Stale concerns
  (e.g., current LTS versions) often need external facts. Claude Code ran
  Sonnet 4.6, not an Opus-class model.
- **RQ4 does not show the selective blindness the abstract states.** The
  "catches surface structure" half comes from RQ3, the comment mix of
  *in-the-wild* AI bots, not from these three reviewers. In the
  benchmark, they recovered 2, 0 and 0 of 76 human Structure concerns.
  That is no better than their Overreach recall (5, 5, 1 of 98; 5.1% for
  OpenCode and Claude Code, the same as Factual accuracy). The clean
  zeros are Triviality, Audience fit, Hallucinated and Scope.
  Hallucinated is the most *checkable* category given the repository (does
  this path exist?), and it is 0/38. That contradicts the paper's own
  "checkable vs normative" axis. A fair summary of Table VI: recall is
  uniformly low, and single-digit counts with no intervals cannot rank
  categories.
- **"Task ceiling, not model ceiling" does not follow.** Low overlap among
  three low-recall reviewers is expected by chance. It says nothing about
  whether a differently tasked model would do better. The 6.9% ensemble
  ceiling is a property of these three runs.
- **Contamination is asserted, not reported.** The RQ4 summary box and
  Discussion cite "contamination effects observed in our controlled
  trial" and reviewers that "obey the very rules [they audit]". No count,
  case or method for this appears anywhere in the paper. It is a plausible
  hypothesis and should not be cited as a finding.
- **Cross-benchmark comparison is not like-for-like.** The "agents review
  code well" contrast cites c-CRAB, Code Review Bench and industrial
  precision figures, which differ in gold, metric and unit. The paper
  could have scored the same reviewers on the same PRs' code concerns. It
  did not.
- **Internal inconsistencies, suggesting an unpolished draft:**
  - 71.9% vs 72.4% by its own counts;
  - 394 vs 408 hollow PRs;
  - "remaining 12 PRs were unreconstructable" when 468 − 446 = 22;
  - "1,987 substantive comments" used interchangeably with concerns;
  - the misquoted 78%;
  - a truncated sentence in §VIII.A ("Today's agentic reviewers");
  - a leftover note in reference [6] ("verify final venue metadata before
    camera-ready").

  None changes the headline. Together they lower confidence in the
  unreported parts, such as the contamination claim.
- **What it does establish well.** The census and coding are careful:
  - identity-blind double coding with agreement reported per stage;
  - a validated reviewer classifier;
  - a gold checked against outcomes and facts;
  - verbatim-quote coverage judging by two judges, plus a stricter
    re-judge.

  The field base rates are the most reliable part. They are a lower bound
  on absence of review, since the corpus is gated to popular repositories.
  The taxonomy is a useful checklist for instruction-file defects.
- **Scope relative to this hub.** OSS software repositories and
  product-default PR review, not research agents. The box's own review of
  instruction text is different. A human accepts `/elevate` proposals,
  and the main agent reads subagent drafts against a brief. The
  transferable claim is narrow: a model's default review of an
  instruction-text change is not evidence that its necessity was judged.

## Trust signals

- **Credibility:** 3 (low end).
  - For:
    - An established empirical-SE group (Minghui Zhou, PKU).
    - A large, careful hand-coded census with agreement statistics at
      every stage.
    - Identity-blind coding and a gold validated against author action
      and facts.
    - Strict quote-backed recall, judged twice, with a stricter re-judge
      that kept the ranking.
    - Leakage controls (web off, history trimmed).
  - Against:
    - Unreviewed preprint with no citations.
    - No data, benchmark or code release named.
    - One run per product, no intervals.
    - No human-vs-human or instruction-tasked arm.
    - Several arithmetic inconsistencies and draft remnants.
    - A headline-level claim (contamination) with no reported evidence.
    - The arXiv ID (2611) does not match the v1 stamp (31 Jul 2026).
  - Not 4: no artifacts, and RQ4's interpretation outruns its design. Not
    2: the census methodology is well above the typical preprint in this
    graph, and the group is reputable.

## Follow-up

- **Relevance:** 4.
  - It is the first measurement in the graph of how well a model reviewer
    recovers human concerns about instruction-file changes, and the first
    field base rates for whether such changes are reviewed at all.
  - Material new evidence for [[concepts/typed-enforcement]]: reference
    existence has to be a code check, since product reviewers with the
    repository caught 0/38 fabricated references.
  - Material for [[concepts/skill-library-lifecycle]]: model review is
    not an admission gate for instruction text.
  - Descriptive evidence for [[concepts/intent-level-briefing]]: what
    maintainers flag.
  - Not 5: SE repositories rather than research agents. The recall
    number lacks the baseline needed to make it canonical.
- **For `/elevate`.** One preprint at credibility 3 with no artifacts, so
  it clears Gate 1 only alongside other sources.
  - It adds an independent attestation to the path/command existence-check
    candidate from [[literature/papers/mehtiyev2026when]].
  - It is weak support for keeping human acceptance of instruction changes
    rather than substituting model review.
  - A clean pass by the model-judged vacuity and consistency checks of
    the accepted 2026-08-02 `/lint` rule-consistency pass should not be
    read as coverage. Product reviewers recovered 0–1 of 146 Triviality
    and 0–8 of 111 Consistency concerns. The task differs (PR review, not
    a whole-file lint) and so does the model.
  - It does not support the contamination claim.
- **Candidates** (cited, not yet in the graph):
  - Gloaguen et al. 2026, "Evaluating AGENTS.md: are repository-level
    context files helpful for coding agents?" (ICLR 2026 MemAgents
    workshop). Context files do not significantly raise SWE-bench or
    CTXBENCH success, cost >20% more inference, and LLM-generated ones
    sometimes hurt. This bears directly on the text budget.
  - Galster et al. 2026, "Configuring agentic AI coding tools" (AIware
    '26). 90.6% of 2,853 configured repos ship an instruction file.
  - Arabat & Sayagh 2026, "Toward instructions-as-code" (MSR 2026 Mining
    Challenge).
  - Yang, He & Zhou 2026, "Beyond banning AI: GenAI governance in open
    source communities". Same group.
  - Zhang et al. 2026, "Code review agent benchmark" (c-CRAB). The
    code-review recall comparison point.
- **Test that would settle the headline (our inference, untested).** Rerun
  RQ4 with three more arms on the same PRs:
  - a second human maintainer's recall of the gold;
  - the same models given an instruction-file review rubric built from the
    16 categories, with web on;
  - the same models scored on the PRs' code concerns.

  Only the second arm says whether "structural blindness" survives
  tasking.
