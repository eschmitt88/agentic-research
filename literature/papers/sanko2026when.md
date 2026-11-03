---
kind: paper
title: "When Skills Tell Agents Not to Ask: A Large-Scale Study of Oversight Directives in Agent Skills"
authors: ["Timofey Sanko", "Yuan Tian"]
institutions: ["Queen's University, Canada (School of Computing)"]
year: 2026
venue: "arXiv 2610.37734 (cs.CR), v1 2026-10-27, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.37734"
code_url: null
citations: null
source: "raw/papers/sanko2026when.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/refusal-cost-symmetry]]"
tags: ["agent-skills", "skill-md", "oversight", "human-in-the-loop", "permission-flags", "confirmation", "scope-clause", "prevalence-study", "gitskills", "near-duplicate-families", "skill-scanners", "codebook", "llm-coder", "mining-software-repositories", "agency-max"]
---

# When Skills Tell Agents Not to Ask: A Large-Scale Study of Oversight Directives in Agent Skills

## TL;DR

A mining study of the text of public `SKILL.md` files. The authors take
1,527,281 distinct English skills from GitSkills (3,301,422 copies across
public GitHub repos). They define an *oversight-loosening directive*: a
line that tells the agent to act with less human involvement ("proceed
without asking", "pass `--yes`", "don't stop between tasks", "commit with
`--no-verify`"). They label 2,097 units by hand and train a detector.

Four results:

1. **Prevalence.** In a 1,600-unit human-labelled sample, 13.3% of
   oversight-vocabulary units loosen oversight and 36.4% tighten it. By the
   detector, 5.8% of English skills (about 88k) have at least one loosening
   directive. 70.1% of loosening *units* carry no scope condition (such as
   "only in CI").
2. **Risk context.** At equal amounts of oversight language, skills with
   shell code, package installs or `sudo` have 1.74–1.82× the odds of a
   loosening directive. They do not have more tightening to offset it.
3. **Spread.** Copying does not make loosening more common. It is
   concentrated: 960 of 86,048 distinct loosening units (1.1%) account for
   half of all loosening occurrences. 93.4% of edited copies leave the
   ancestor's oversight text untouched.
4. **Scanners.** Two open-source skill scanners catch at most 46% of
   loosening directives, at precision below 0.4. The same LLM, asked the
   codebook's oversight question instead of a malware question, reaches
   F1 0.65.

The paper measures **what skills say, not what agents do**. No agent is
run, and no harm, incident or compliance rate is measured. The authors
say so.

## Claims

- Skills hold a second kind of checkpoint: one that exists only because an
  instruction asks for it (ask before pushing, report each commit). A
  skill's text can remove that kind. It can also steer the agent around the
  harness's own kind (e.g. telling it to pass `--force`).
- Loosening directives are "a convenience, not an attack". They show no
  sign of intent to harm. The risk is that a shared file carries them into
  contexts the author did not foresee, with no signal that oversight is
  being skipped.
- "Scope is a missing safeguard." Over two-thirds of loosening sets one
  autonomy level for every context. Authors could keep most of the
  convenience with a one-line scope clause ("pass `--yes` only in CI").
- Fixes should target upstream skills, not adopters. Editing the 100
  families with the most loosening would reach about a quarter of the
  loosening-carrying skills agents load.
- Scanners fail because of the question asked, not the model behind them.

## Methods

- **Population.** GitSkills via its Hugging Face mirror: 3,797,117
  `SKILL.md` files from 282,200 repos, 1,877,981 distinct contents. Kept if
  it is a valid skill (YAML frontmatter or in a skills directory), first
  committed no earlier than October 2025, not a symlink placeholder, and
  English by `langid.py`. Result: 1,527,281 distinct skills.
- **Units and lexicon.** Each body is split into units (sentence, bullet,
  table cell, code line), each read with its section heading and
  neighbours. A lexicon of 36 regex patterns (e.g. "without asking",
  `--no-verify`, "auto-merge") retrieves 1.37M candidate units (817,479
  distinct) in 513K skills. A repository-balanced frame keeps at most two
  distinct units per repo per stratum: 370,285 units.
- **Definition (final codebook v2).** A unit is a *directive* if deleting it
  would change how much control a human has over the agent. It is a
  *mention* if it refers to oversight without instructing the agent. A
  directive *loosens* if it lets the agent act with less human involvement.
  *Mechanism* is one of nine (Table 1): PERM, CONF, PERSIST, SAFE, AUTO,
  TRANS, ACT, INVOKE, PREC. *Scope* is the condition under which a
  loosening applies (e.g. in CI, in tests, only if the user authorized it),
  or "unscoped". Twelve decision rules resolve edge cases; they are not
  listed in the paper.
- **Two conventions inflate loosening.** Commands run with
  prompt-suppressing flags count as loosening. Persistence is broad: "don't
  stop at the first failure" counts as well as "don't check in". Counting
  only the narrow persistence kind gives 12.7% [11.0, 14.4] instead of
  13.3%.
- **Labelled sets (2,097 distinct units).** Pilot 100, calibration 400,
  reliability 200 and gold 400 (stratified), pre-labelled 500, and a census
  of the 500 most-copied candidate units. The first author labelled or
  reviewed every unit. **The second coder is an LLM, Claude Opus 5.5 at max
  effort**, on 1,000 units, with labels hashed before comparison. There is
  no second human coder. On the pre-labelled set and census the LLM
  proposed and the author reviewed (changes on 8.4% and 4.0%); these are
  not counted as agreement.
- **Agreement (Cohen's κ, Table 2).** Gold set (n = 400, v2): three-way
  label 0.82, polarity 0.92, mechanism 0.84, scope 0.85. Round 1 (v1,
  n = 400) had scope κ 0.17. This shows that scope only became codable
  after the v2 rules.
- **Detector.** A fine-tuned RoBERTa-base (two heads, three seeds averaged)
  plus Jev 1.13, a hosted multiple-choice decision model. Jev's class
  probabilities are reweighted (neither 1.0, loosening 0.25, tightening
  0.75), with weights grid-searched on 700 non-gold units. The two are
  averaged. The gold set is never used for training. But the authors chose
  to combine the two models *after* seeing both models' gold scores, and
  they say this "may flatter the combination slightly".
- **RQ2.** Three logistic regressions on 512,876 candidate skills, with
  detector labels. Outcome: at least one loosening (M1, M2) or tightening
  (M3) directive. Predictors: eight regex capability features plus bundled
  executables, log stars, copied to two or more repos. Controls: log
  length and log number of candidate units. SEs are clustered by repo.
  Five robustness refits are run, plus a direction check on the
  human-labelled prevalence sample.
- **RQ3.** Families are near-duplicates by MinHash Jaccard over 5-word
  shingles at ≥ 0.7, giving 118,076 families with two or more members
  (558,531 skills). Repeated at 0.6 and 0.8. The ancestor is the
  earliest-committed member, or alternatively the most-copied one. It is
  inferred, not observed.
- **RQ4.** Cisco AI Defense skill-scanner 2.2.1 and NVIDIA SkillSpector
  2.12.0, each offline (rules) and with GPT-5.4. Scored on the 400 gold
  units, 61 of which are loosening.

## Results

- **Detector validation (gold set, n = 400, 61 loosening).** Accuracy 0.80,
  κ 0.65, loosening precision 0.70, recall 0.61 [0.48, 0.72], F1 0.65.
  RoBERTa alone gets F1 0.52, Jev alone 0.62, GPT-5.4 with the codebook
  0.65 (McNemar p = 1.0 against the detector, at more than ten times the
  cost). Units with confidence ≥ 0.9 (31% of units) are 97% correct. The
  detector under-calls loosening (10.3% vs 13.3% human) and over-calls
  tightening (39.3% vs 36.4%).
- **Lexicon recall, the bigger gap.** In 250 random skills (28,311 units),
  a blind stratified sample of 319 units was labelled. Among candidate units
  the classifiers flag 94% of verified loosening. But **the lexicon itself
  captures an estimated 32% [14%, 75%] of loosening directives** (61% if one
  unit with sampling weight 28 is dropped), and 16% [10%, 25%] of
  tightening. Missed phrasings include "no approval needed". So all counts
  are conditional on the lexicon and are lower bounds.
- **RQ1, prevalence (Table 4, distinct units in the frame, human labels
  with 95% CIs).** Oversight directive 49.7% [46.8, 52.6]. Loosening 13.3%
  [11.7, 15.0], of which unscoped 70.1% [63.9, 75.8]. Tightening 36.4%
  [33.7, 39.2]. Mention only 11.6% [10.1, 13.1]. That is about 49,000
  distinct loosening units. Removing security-tool repos and test fixtures
  gives 13.2% → 13.3%. Individual labelled sets give 12.5–15.6%.
- **Mechanisms.** Loosening: permission flags 33.9%, confirmation 27.4%,
  persistence 16.0%, safeguards 7.1%, auto-action 6.2%, transparency 4.5%,
  consequential action 3.0%, invocation-as-approval 1.5%, precedence 0.5%.
  Tightening is mostly confirmation (46.3%) and consequential action
  (20.2%).
- **Copy-weighted.** Loosening is 17.9% [13.3, 24.4] of units weighted per
  skill file and 15.0% [11.2, 19.9] weighted per occurrence. Both intervals
  include 13.3%. In the 500 most-copied units the mix changes: "act without
  telling the user" (TRANS) is the top mechanism at 22.6%, against 4.5% in
  the sample.
- **Skill level (detector).** 5.8% of English skills have at least one
  loosening directive and 16.9% have at least one tightening directive.
- **RQ2 (Table 5, n = 512,876, loosening OR [95% CI]).** Shell code block
  1.82 [1.73, 1.92]. Package install 1.82 [1.69, 1.95]. `sudo` 1.74
  [1.61, 1.89]. `curl | sh` 1.31 [1.18, 1.45]. Network 1.19. Bundled
  executables 1.12 [1.00, 1.27]. Git write 1.09. Destructive command 0.98
  (n.s.). Secrets 0.77. Stars 0.99. Copied 1.13 [1.05, 1.22], or 1.18 when
  counted by distinct owners. Tightening ORs: package 0.59, `sudo` 0.75,
  `curl | sh` 0.77, shell 0.95 (n.s.). Git write is the exception at 2.01.
  12,484 skills (0.82%) pair loosening with destructive commands,
  `curl | sh` or `sudo`; 6,918 (0.45%) do so at p ≥ 0.7.
- **RQ2 robustness.** Shell, package and `sudo` hold in all five refits.
  `curl | sh` holds in four. Without the oversight-language control, the
  copying OR reverses (0.85). **On the human-labelled sample, the direction
  agrees for shell, network, package and git write, but not for secrets,
  `sudo` or `curl | sh`** (the last two rest on 39 and 42 units). All of
  this is association; nothing is causal.
- **RQ3, spread.** Copying does not amplify. 5.8% of distinct skills carry
  loosening against 5.4% of all occurrences (ratio 0.94). For tightening
  the ratio is 0.85. The top 100 families hold 23% of loosening-carrying
  occurrences but 1.4% of all occurrences. 2.4% of loosening-carrying
  families account for half of loosening-carrying copies. 960 of 86,048
  distinct loosening units (1.1%) account for half of loosening
  occurrences. 93.4% of variants leave oversight text untouched (92–97%
  across specifications). Where variants edit, the share of loosening
  among added units matches removed units: 20.3% vs 20.2% (+0.1 points
  [−5.3, +4.7]). With the most-copied ancestor rule, the shift is +1.4 to
  +5.5 points, significant in 2 of 5 specifications. The common edits are
  new releases of the same skill pack that add an auto-commit mode or drop
  an ask-before-step line.
- **RQ4 (Table 6, gold, precision / recall / F1 / share of flags on
  tightening).**

  | Method | P | R [95% CI] | F1 | %T |
  |---|---|---|---|---|
  | Keyword lexicon | 0.20 | 0.82 [0.71, 0.90] | 0.33 | 24% |
  | Cisco offline | 0.14 | 0.02 [0.00, 0.09] | 0.03 | 29% |
  | Cisco + LLM | 0.39 | 0.15 [0.08, 0.26] | 0.21 | 13% |
  | SkillSpector offline | 0.21 | 0.21 [0.13, 0.33] | 0.21 | 29% |
  | SkillSpector + LLM | 0.27 | 0.46 [0.34, 0.58] | 0.34 | 23% |
  | Either + LLM | 0.28 | 0.51 [0.39, 0.63] | 0.36 | 23% |
  | LLM + codebook | 0.71 | 0.61 [0.48, 0.72] | 0.65 | 4% |
  | Detector | 0.70 | 0.61 [0.48, 0.72] | 0.65 | 4% |

  On the full prevalence sample, offline Cisco flags 1.6% of loosening and
  SkillSpector 28%. 23–29% of scanner flags land on guardrails ("do not
  auto-approve" trips an excessive-agency rule). One LLM (GPT-5.4) was
  used, on 400 units, so the intervals are wide.

## Critique / open questions

- **Digest framing: at.** Every number in the digest entry is in the paper
  and quoted correctly. 1 in 17 skills is ~88k (5.8% of 1.53M). The 1.7–1.8×
  odds are for shell, package and `sudo`. 2.4% of families account for half
  of copies. "≤ 46% at precision < 0.4" matches Table 6 per scanner. Three
  qualifications, none of which the digest contradicts:
  1. "70.1% of these apply unconditionally" follows the abstract, but it is
     a share of loosening *units* in the human sample, not of the ~88k
     *skills*. Unit-level and skill-level estimates come from different
     label sources.
  2. "1 in 17 skills" is a detector estimate (loosening F1 0.65) on top of
     a lexicon that the authors estimate catches only 32% [14, 75] of
     loosening. The paper argues it is an underestimate. The interval says
     the true rate is poorly bounded.
  3. "Loosening spreads from a few upstream families" is right about
     concentration. The digest leaves out the paired finding that copying
     does *not* amplify loosening (ratio 0.94) and adopters rarely edit it.
     "Spreads" here means verbatim reuse, not growth.
- **The question the brief asked: is any harm measured? No.** Nothing is
  measured about agent behaviour, compliance, incidents or cost. The paper
  says "We measure what skills say, not how agents act" and lists "whether
  agents follow oversight directives and whether a scope clause changes
  their behaviour" as future work. So "loosening is a risk" rests on the
  definition (removing a checkpoint) plus OWASP's guidance, not on outcome
  data. The paper's own framing is moderate: "a convenience, not an
  attack".
- **The definition counts by control removed, not risk.** `--yes` to a
  package manager, "don't stop at the first failing test" and "push
  without confirmation" are all loosening. Reversibility and blast radius
  are not coded. The codebook has no axis for whether the skipped
  checkpoint guarded a reversible action. That is the axis this box's
  confirmation principle uses (see below).
- **Detector validation is honest but thin on positives.** It is held out
  from training. But the combination was chosen after seeing gold scores,
  and there are 61 positives, so recall ±0.12. The human-vs-LLM agreement
  (κ 0.82) is author vs Claude Opus 5.5, not two humans. The twelve
  decision rules and the full codebook are not printed. No labels, codebook
  or detector are linked in the PDF; the conclusion says they "would help"
  but gives no URL.
- **Keyword lexicon recall 0.82 on gold is unexplained.** Every gold unit
  was retrieved by the lexicon, so this baseline must use a narrower rule
  than retrieval did. The paper does not say what.
- **RQ2 partly fails its independent check.** Of the two headline
  capabilities in the digest, the `sudo` direction does not replicate on
  human labels (small n). Shell and package installs do.
- **Population limits.** Public GitHub only, English only, `SKILL.md` only.
  AGENTS.md, CLAUDE.md and rule files are excluded and named as future
  work. Private and marketplace skills may differ.

### Applying the lens: this box's own loosening directives

The task asked which of the box's directives are conditional and which are
unconditional, in the paper's terms. Scope = a stated condition on *when*
the loosening applies. Exceptions left to the agent's judgment and limits
by action class are not scope conditions in the paper's examples.

| Directive (where) | Mechanism | Paper's scope reading |
|---|---|---|
| "Commit and push automatically… Don't wait for permission… every git repo I work in" (`~/.claude/CLAUDE.md`, Git autonomy) | AUTO + ACT + CONF | **Unconditional.** Scope is explicitly widened to every repo. The "judgment still applies… flag it" carve-out is agent-judged, not a stated condition. |
| "act autonomously… without pausing for confirmation, **while the verdict permits and ceilings hold**" (`agency.md`, `max`) | CONF, PERSIST | **Conditional.** Per-repo opt-in, coordinator verdict, `budget.yaml` ceilings, `hold` absolute. |
| "Artifact writes… commit autonomously everywhere, in every repo" (`agency.md`, confirmation principle) | AUTO | **Unconditional** by context; limited by class (artifact writes, reversible in git). |
| "commit and `git push`" after every autonomous burst "in a `max` repo" (`agency.md`) | ACT | **Conditional** on repo opt-in only (not on verdict). |
| "Agentic workflow — no confirmation gate" (`/ingest`, `/fetch-paper`, `/discover`, `/wrap`, `/sync-imports` bodies) | CONF | **Unconditional.** The line carries no condition. The rationale lives in `agency.md`, which the skill loads via `respects:`, but the gate is not in the line. |
| "Under `agency: max`, do this autonomously, gated by the headroom verdict" (`/curate`) | CONF | **Conditional.** |
| "`--no-gate` bypasses" (`~/.claude/CLAUDE.md`, `/jobs`) | — | **Mention**, not a directive (documents a flag). |
| `skipDangerousModePermissionPrompt`, bypassed permissions for subagents (settings) | — | Harness checkpoint, not text. Outside the codebook. |

Of seven text directives, three are conditional and four unconditional.
The paper's 70% is a population share; with seven items this is
illustration, not a comparison. The box's split follows a rule the paper
does not code. The unconditional lines cover git-reversible artifact writes.
The conditional lines cover spending compute and choosing hypotheses. The
exception is Git autonomy's **push**. A push to a public remote with Pages
cannot be fully undone for information exposure, and the line is scoped to
"every git repo". The "don't commit secrets" tightening and "flag it"
carve-out sit beside it. This is a lens, not evidence of harm. The paper
measures none.

Two concrete findings from the check:

- **`/wrap` contradicts itself.** Its frontmatter `description:` (always in
  context) says "shows diff, waits for confirmation". Its body step 6
  (loaded on invoke) says "no confirmation gate". The tightening and the
  loosening of the same checkpoint are in the same skill. `/ingest` has a
  milder version: step 7 says "the same confirmation batch", step 8 says
  "no confirmation gate".
- **The box is an upstream publisher in the paper's sense.**
  `~/.claude/skills/*` are byte-identical to `claude-system/claude/skills/*`.
  That repo is public, and its first commit (2026-04-24) is within
  GitSkills' window. A copier gets the unconditional "no confirmation gate"
  lines. The conditions live in `~/claude-system/claude/rules/agency.md`
  and in a coordinator binary, and neither travels with a copied skill.
  This is the paper's "carried into contexts the author did not
  anticipate" case. Whether GitSkills actually contains this repo was not
  checked.

## Trust signals

- **Credibility:** 3.
  - For: a careful MSR-style design. Stratified, repository-balanced
    sampling with reweighting and bootstrap CIs. A three-version codebook
    with κ at each round, including the bad round (scope κ 0.17 in v1).
    The detector is held out from training on a gold set. A blind lexicon
    recall study that hurts the paper's own numbers (32%). Five RQ2
    refits plus a human-label direction check that partly fails and is
    reported. Several near-duplicate thresholds and two ancestor rules.
    Clear threats-to-validity. Yuan Tian's Queen's group publishes in
    MSR/SE venues.
  - Against: not peer reviewed, one week old, no citations. No codebook,
    labels or detector released (no URL in the PDF). The second coder is
    an LLM, and no second human was available. Detector combination chosen
    after seeing gold scores. 61 gold positives. Lexicon recall 32% with a
    wide CI. No behavioural outcome at all.
  - Not 4 because nothing is released and the reliability rests on one
    human plus an LLM. Not 2 because the design is careful, the CIs and
    caveats are reported, and the numbers that hurt the paper are printed.

## Follow-up

- **Relevance:** 4. It is the first large-scale measurement of oversight
  text in the skill format this box writes and publishes. It also gives a
  codable conditional/unconditional distinction that sorts the box's own
  directives. It adds material evidence to:
  - [[concepts/shared-substrate-contagion]]: spread is verbatim reuse
    from a few upstream families. It is not amplified, and adopters do not
    edit it.
  - [[concepts/permission-gate-as-architecture]]: instruction-held
    checkpoints are removed in text, mostly with no scope.
  - [[concepts/shared-skill-namespace]]: a copied skill carries the
    loosening line but not the condition that gated it, and scanners
    don't look.
  Not 5 because it measures text, not agent behaviour or harm.
- **Fronts.** Not on the redirect-vs-halt, known-bad-preflight,
  precompact or noise-band proposals. On `agency.md`: the two text fixes
  above (the `/wrap` description; a scope clause beside "no confirmation
  gate", e.g. "artifact writes only; spend is governed by `agency.md`") are
  cheap and make the condition travel with the skill. They are untested.
  Flag them, don't adopt them on this evidence.
- **Missing experiment, also missing in the paper.** Does a loosening
  directive in a skill change what Claude Code does when the harness gate
  is on? Does a scope clause change it? Example cells: {no directive,
  unscoped "push without confirmation", scoped "only in CI"} × {auto mode,
  bypass}, on a task with one consequential step. Pairs with
  [[literature/papers/qin2026llm]]'s gate-on/off design.
- **Candidates** (cited, not yet in the graph):
  - Destefanis et al. 2026, "GitSkills: A Dataset of Agent Skills on
    GitHub" (arXiv 2608.10906).
  - Liu et al. 2026, "Do Not Mention This to the User: Detecting and
    Understanding Malicious Agent Skills in the Wild" (arXiv 2602.06547).
  - Gao et al. 2026, "From Registry to Repository: How AI Agent Skills Are
    Written, Adapted, and Maintained" (arXiv 2607.00911).
  - Dhanorkar et al. 2026, "Human oversight of agentic systems in
    practice" (arXiv 2606.05391).
  - Jiang and Nam 2026, "Beyond the Prompt: An Empirical Study of Cursor
    Rules" (MSR '26).
