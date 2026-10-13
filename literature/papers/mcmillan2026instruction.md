---
kind: paper
title: "Instruction Adherence in Coding Agent Configuration Files: A Factorial Study of Four File-Structure Variables"
authors: ["Damon McMillan"]
institutions: ["HxAI Australia"]
year: 2026
venue: "arXiv 2605.10039 (cs.SE), v1 2026-05-11, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.10039"
code_url: null
citations: 3
source: "raw/papers/mcmillan2026instruction.pdf"
added: "2026-10-13"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/typed-enforcement]]"
tags: ["claude-md", "agents-md", "configuration-files", "instruction-adherence", "file-size", "instruction-position", "file-architecture", "conflicting-instructions", "within-session-attenuation", "lost-in-the-middle", "claude-code", "factorial-design", "affirmative-null", "bayes-factor", "task-heterogeneity"]
---

# Instruction Adherence in Coding Agent Configuration Files: A Factorial Study of Four File-Structure Variables

## TL;DR

A fractional factorial over four CLAUDE.md structure variables: size
(25/100/250/500 lines), position of the target instruction (line 2 to line
250), architecture (CLAUDE.md alone, + AGENTS.md, + nested src/ and tests/
CLAUDE.md), and a contradicting instruction in AGENTS.md. It runs 1,650
headless Claude Code CLI sessions (mainly Sonnet 4.6) on two TypeScript
repos. The dependent variable is a trivial, AST-checked marker: every new or
modified function body must begin with `// @tracked`.

Three results matter for this graph:

1. **No structural variable moved compliance detectably.** Size and conflict
   are affirmative nulls (BF₁₀ 0.096 and 0.053). Position and architecture
   are only failures to reject, and the design cannot exclude effects below
   about 6 points.
2. **Compliance is low even when the rule is present and well placed.** In
   the reference cell (250 lines, rule at line 2, single file, no conflict)
   67.7% of functions comply. At 25 lines it is 60.0%. With no CLAUDE.md the
   rate is 0%, so the marker is purely instruction-driven.
3. **The large variance is within session and across tasks.** Each
   additional generated function carries OR 0.944 for compliance (pooled,
   exploratory, non-monotonic). Task means run from 45.1% to 84.4%.

## Claims

- **Structure does not matter in the tested range.** None of the four main
  effects or three two-way interactions survives BH correction. "A 25-line
  CLAUDE.md and a 500-line CLAUDE.md are indistinguishable."
- **A contradicting AGENTS.md instruction does not override CLAUDE.md**
  (C0 63.7% vs C1 64.1%, n = 2,044, BF₁₀ = 0.053).
- **No lost-in-the-middle U-shape across positions.** The descriptive
  pattern is "weak primacy followed by a small monotonic drift": P1 67.7%,
  P5 61.8%, raw p = 0.055, BH p = 0.554.
- **Within-session attenuation is the largest effect.** Slope −0.0578
  log-odds per generation step, OR 0.944 [0.937, 0.951], p ≈ 10⁻⁴⁹. The
  direction reproduces on Umami/Sonnet 4.6 and ixartz/Opus 4.6. It was found
  during analysis, not pre-specified.
- **Task identity beats structure.** T4 (sidebar refactor, modify existing)
  45.1% vs T5 (dashboard build, write new) 71.3%, a 26.2-point gap. The
  author declines a "modifying is harder than writing" reading because the
  tasks differ on many dimensions.
- **Practitioner advice.** Layer in-session mechanisms (hooks that
  re-surface the rule at PreToolUse/PostToolUse/Stop, agent-managed memory)
  and post-hoc enforcement (linters, CI). Do not split configuration across
  files as a remedy. Focus on content rather than length.

## Methods

- **Design.** Twenty-four conditions in four sub-blocks: main effects
  (ME-01–11, each IV varied with the others at S3/P1/A1/C0), size × position,
  size × architecture, conflict × position. There is also a no-config
  baseline, three Umami cells, and three cross-model cells each for Opus 4.6
  and Opus 4.7. Fifty runs per condition; each run is assigned one of five
  tasks, 10 runs per task.
- **Content.** Padding was authored to mirror the topical mix of surveyed
  real CLAUDE.md files, not extracted from any repo. The target instruction
  is fixed text. Size is manipulated with the target at P1 (line 2).
  Position is manipulated at 250 lines. In A2/A3 an `@AGENTS.md` import is
  prepended to load the second file.
- **Harness.** Claude Code CLI 2.1.92 pinned, headless single-turn
  `--print`, up to 100 agent turns (median 14, max 81). The repo is reset to
  a pinned commit between runs. Opus 4.7 needed CLI 2.1.123, which enables
  adaptive thinking by default, so it is confounded on three axes.
- **Measurement.** The post-session git diff is AST-parsed. A function
  complies if its first statement is exactly `// @tracked`. The unit is
  function-level: 16,050 observations, 43 no-code sentinel rows excluded.
- **Analysis.** Binomial GLMMs (lme4) with (1|run) + (1|task), one per
  sub-block, using LRTs. A Bambi/PyMC companion supplies Savage–Dickey Bayes
  factors and a prior sweep σ ∈ {0.5, 1, 2.5, 5}. BH-FDR for pairwise tests.
  A simr power analysis gives ≥98% power for a 15-point shift and 70–99%
  for 10 points, with size the weakest.
- **Within-session predictor.** Chronological generation order,
  reconstructed from the tool-call timeline. An AST-traversal index is a
  sensitivity check (OR 0.909, same qualitative shape).

## Results

- **Size (P1, A1, C0).** 25/100/250/500 lines: 60.0 / 65.2 / 67.7 / 64.0%.
  GLMM χ²(3) = 5.16, p = 0.16. Linear trend p = 0.625. BF₁₀ = 0.096, below
  1/3 under every prior tested ([0.019, 0.197]).
- **Position (250 lines).** P1–P5: 67.7 / 63.2 / 64.0 / 64.0 / 61.8%.
  χ²(4) = 1.47, p = 0.83. No polynomial component is detectable (quadratic
  p = 0.94).
- **Architecture.** A1 67.7%, A2 68.2%, A3 (nested) 61.7%. χ²(2) = 3.29,
  p = 0.19. A3 is −6.0 points vs A1 at raw p = 0.053, BH p = 0.079.
- **Conflict.** 63.7% vs 64.1%, χ²(1) = 0.10, p = 0.76. The interactions
  are all null (p = 0.81, 0.67, 0.13).
- **Baseline.** 0/524 functions without configuration (Wilson upper bound
  0.73%). Replicated at 0 on Umami and on both Opus models.
- **Tasks (Table 4).** T1 health endpoint 82.6%, T2 button 84.4%, T3
  NextAuth 72.7%, T4 sidebar refactor 45.1%, T5 dashboard 71.3%. The task
  random-effect variance is 0.310.
- **Attenuation by task.** T3 +0.005 (OR 1.005, p = 0.44, not detectable);
  T4 −0.046 (OR 0.955); T5 −0.185 (OR 0.831). The slope-by-task LRT is
  χ²(2) = 293. The pooled curve falls from about 88% at position 1 to about
  55% by position 9, then partly recovers. The author attributes the
  recovery to a shift toward page/layout files, which stay compliant.
- **First omission.** 756 of 1,150 non-baseline runs (65.7%) had at least
  one miss. The median first-omission position is 4 (AST metric: 2).
  Post-omission compliance is 55%, so runs re-comply intermittently rather
  than giving up.
- **Acknowledgement.** The agent mentioned the instruction in 112 of 1,150
  runs (9.7%). Acknowledged runs complied 72.0% vs 62.8%, and 28% of
  acknowledged functions still missed.
- **Cross-codebase and cross-model.** Umami at the reference config 56.7%
  (−11.0 points). Opus 4.6 54.9% (−12.8 vs Sonnet 4.6). CLI-matched slopes
  are −0.040 (ixartz Sonnet), −0.029 (ixartz Opus 4.6, CI upper 0.999) and
  −0.068 (Umami Sonnet); slope-equality LRT p = 0.327. Opus 4.7 asked a
  clarifying question instead of coding in 25.3% of trials and is not
  interpreted.
- **Quality.** After controlling for task, ICR does not predict test-suite
  pass rate (OR 1.25 [0.57, 2.73]). There is no detectable quality cost of
  the marker.

## Critique / open questions

- **House-pattern check: the paper reads AT the digest framing, and the
  abstract does not overstate.** The digest's "null on structure, with
  within-session compliance decay" matches the abstract, which already
  splits the nulls into affirmative (size, conflict) and failure-to-reject
  (position, architecture). The abstract also flags the attenuation as
  found in analysis, non-monotonic and detectable on two of three tasks.
  Checked against the most-favourable-cut pattern, both headline numbers are
  max or pooled cuts, but the paper labels them so:
  - OR 0.944 is the primary-pool average. It is driven by T5 (OR 0.831),
    with T3 flat.
  - 26.2 points is the single largest between-task contrast, reported "as a
    single-contrast observation".

  Where the digest over-reads is the word *decay*. Much of the attenuation
  mass sits in the first three or four functions (median first omission 4),
  and the late-session recovery is compositional. That looks as much like a
  loading or uptake failure as like gradual erosion of a loaded rule. The
  author says the two regimes are not separated.
- **Bearing on the 80-line CLAUDE.md rule: the rule cannot be justified by
  adherence to a top-placed rule, and is not refuted for anything else.**
  Between 25 and 500 lines a line-2 instruction is followed at the same rate
  (BF₁₀ ≈ 0.1). Size was manipulated only at P1, the most favourable
  position. The size × position block (S2/S4 × P3/P5) is also null, but
  only at four cells. What the paper does not touch: token cost per session
  (Gloaguen et al. 2026, cited, report >20% inference-cost inflation from
  context files), maintainability, and dilution at >500 lines. The 80-line
  rule's honest justification is cost and legibility, not compliance.
- **Bearing on scoped rules: the architecture arm is not the design this
  repo uses.** A3 puts *complementary* conventions in nested src/ and tests/
  CLAUDE.md files, with the target always in the root. It never tests the
  claim that path-scoped `.claude/rules/` makes: that a rule landing only
  when its paths are touched binds better than the same rule always on. The
  suggestive A3 dip (−6.0, raw p = 0.053) is the only signal. It points
  toward extra loaded files costing a little, which is consistent with
  [[literature/papers/agarwal2026fire]]'s harm from untargeted text, not against scoping.
- **The dependent variable is the most favourable possible case, and it
  still fails a third of the time.** A one-line, syntactic, uniformly
  applicable "MUST" was missed on 32% of functions at line 2. That is
  strong evidence that presence is not adherence. It also limits transfer:
  the marker plausibly fights a style prior (no comments inside function
  bodies), which may inflate misses on modified functions (T4) and helper
  functions. The author concedes that function-type composition shifts with
  position and that a confound cannot be separated from mechanism.
- **"Within-session" means one `--print` turn, not a long session.** There
  is no compaction, no re-injection and no multi-turn conversation. The rule
  was in context the whole time. Attenuation here therefore cannot be
  eviction, which matters for [[concepts/constraint-pinning]]: pinning
  guarantees exactly what was already true in this setting.
- **The hook recommendation is advice, not a finding.** No arm re-surfaces
  the rule via PreToolUse, adds a linter, or re-injects mid-session. The
  practical-implications table recommends these, but the evidence for them
  lives elsewhere in this graph ([[literature/papers/agarwal2026fire]]'s hook-timed injection,
  [[literature/papers/zhang2026judged]]'s harness-side stop).
- **Measurement is deterministic; generation is not.** Fifty runs per cell
  with function-level GLMMs is reasonable. The run is the randomization
  unit, so per-cell n is 50, and pairwise power is about ±6 points. The
  position and architecture nulls are therefore weak.
- **Scope.** Three Anthropic models, one CLI, two TypeScript repos, five
  feature-addition tasks, a single instruction phrasing, padding authored
  by the experimenter. The Opus 4.7 arm is uninterpretable by the author's
  own account.

## Trust signals

- **Credibility:** 2.
  - For: a pre-specified GLMM plan with BH correction and a Bayesian
    companion with a prior-sensitivity sweep. Affirmative nulls are kept
    distinct from failures to reject. Post hoc findings are labelled
    throughout, the baseline replicates at 0 in four cells, and confounds
    (Opus 4.7 CLI/thinking, EV-03, T4 vs T5) are disclosed rather than
    spun. The reporting is unusually careful for a single-author preprint.
  - Against: a single author at a small consultancy (HxAI Australia), not
    peer reviewed, no released code, data or prompts (no URL in the PDF;
    the arXiv comments field lists only page counts), and 3 citations
    (Semantic Scholar, 2026-10-13). There is no pre-registration document,
    only an analysis plan "described in advance". The padding and task
    prompts are not reproducible from the paper.
  - Not 3 because nothing can be re-run or audited. Not 1 because the
    statistical practice and disclosure are well above the weak-preprint
    baseline.

## Follow-up

- **Relevance:** 4. It is the first controlled measurement in this graph of
  the box's own instruction channel: Claude Code reading a CLAUDE.md. It
  tests the exact design variables this framework legislates (file length,
  file splitting, conflicting files). It adds material evidence to
  [[concepts/constraint-pinning]] ("presence is not sufficiency", now without
  any compaction) and to [[concepts/enforcement-boundary-placement]]
  (prompt-placement rows). It is not 5 because the DV is a toy marker in
  single-turn mode, and the scoped-rules question is untested.
- **For the framework's CLAUDE.md rules.** Keep the 80-line ceiling but
  restate its rationale as cost and legibility. Do not cite adherence. Keep
  the most load-bearing rule near the top: the weak-primacy pattern is not
  excluded. Any rule whose violation matters should have a hook or a
  checker, because a present, top-placed, trivial rule ran at about 65%.
- **Untested arm worth running here.** The same rule placed in a
  path-scoped `.claude/rules/` file versus the root CLAUDE.md. Use a
  deterministic marker like this one, and add a PreToolUse re-surfacing arm.
  That is the same-content counterfactual
  [[concepts/enforcement-boundary-placement]] notes as missing.
- **Candidates** (cited, not in the graph):
  - Gloaguen et al. 2026, "Evaluating AGENTS.md: Are repository-level
    context files helpful for coding agents?" (arXiv 2602.11988). An ETH
    Zurich study: context files reduce task success and inflate cost
    >20%.
  - Chatlatanagulchai et al. 2025, "Agent READMEs: An empirical study of
    context files for agentic coding" (arXiv 2511.12884), a survey of 2,303
    files.
  - Geng et al. 2026, "Control illusion: the failure of instruction
    hierarchies in LLMs" (AAAI 2026, arXiv 2502.15851).
  - McMillan 2026, "Structured context engineering for file-native agentic
    systems" (arXiv 2602.05447), the author's earlier work.
