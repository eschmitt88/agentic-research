---
kind: paper
title: "Cortex: Measurement-Gated Evolution and Pruning of Coding-Agent Skills and Rules"
authors: ["Samuele Giampieri"]
institutions: ["Independent researcher"]
year: 2026
venue: "arXiv 2610.14216 (cs.SE), v1 2026-10-09, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.14216"
code_url: "https://github.com/samugit83/cortex"
citations: null
source: "raw/papers/giampieri2026cortex.pdf"
added: "2026-10-19"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-skill-namespace]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/typed-enforcement]]"
tags: ["claude-code", "claude-md", "path-scoped-rules", "skills", "harness-evolution", "exposure-gating", "paired-gate", "pruning", "placebo-candidates", "known-answer-test", "skill-invocation", "loading-tier", "a-a-arm", "held-out-scenarios", "synthetic-lab", "haiku-4.5", "sonnet-5"]
---

# Cortex: Measurement-Gated Evolution and Pruning of Coding-Agent Skills and Rules

## TL;DR

Cortex is three Claude Code commands that treat a repository's `CLAUDE.md`,
`.claude/skills/` and path-scoped `.claude/rules/` (the "harness") as
something to measure. `/harvest` turns a session that went from broken to
working into a replayable task (broken commit, verbatim request, fix, check
script) and logs each developer correction as a one-line lesson. `/evolve`
writes one skill or rule per cycle for a recurring problem and keeps it only
if a scripted, paired comparison against the live harness passes five gates.
`/prune` removes live items under the same gates, reversed. The organising
rule: **an item is credited or blamed only on tasks where it entered the
agent's context** ("exposure"). A candidate that never fired is never kept;
an item that never fired is never deleted.

Evaluated in one purpose-built synthetic repository (`shopkit`, four
machine-checkable house rules plus a no-rule control family), with Claude
Haiku 4.5 in every role, a scripted developer, 4 independent runs, and 30
held-out scenarios:

1. Evolved harnesses beat no harness by **+55.0 points** on the rule
   families (22% → 77%, 95% CI [+34.1, +74.1], permutation p = 0.0001). The
   control family moved −0.8 [−5.0, 0.0].
2. **16 of 18 kept items were rules.** Path-gated skill candidates were
   opened in 28 of 66 rollouts that listed them; always-on skill candidates
   in **0 of 198**.
3. The gates kept 0/20 placebos and 0/5 harmful items, but none of the
   placebos or harmful skills was ever opened, and the harmful rules were
   never swept on tasks where harm could show. The test shows the gates keep
   no unfired text, not that they detect harm.
4. Under Claude Sonnet 5 the same harness added +5.6 [−2.8, +13.9]. On a real
   repository (structlog) nothing recurred and the loop learned nothing.

## Claims

- **Exposure-aware measurement semantics** (contribution 1). The same rule
  governs adding and deleting: compare with/without only on exposed tasks;
  unknown exposure in any valid rollout blocks the verdict (RERUN). The
  author claims the symmetric use (same rule for admission and removal) is
  new; SLIM applies it to retirement only.
- **A known-answer test of the acceptance pipeline** with planted placebo,
  harmful and known-good candidates. The author says they found no prior
  study planting placebo candidates into a coding agent's acceptance
  pipeline.
- **Loaded vs chosen is the main lesson:** "how advice is delivered rather
  than what it says". Rules (injected by Claude Code when a matching file is
  read) did most of the work; skills (description listed, body read only if
  the agent invokes) helped only when path-gated.
- **Value tracks how often the base model breaks the convention.** The gain
  shrinks under a stronger model, and one `/prune` pass after the upgrade
  flagged 2 of 4 items as removable.
- **Stated non-results (abstract and Section 7):** only 2/4 known-good
  candidates kept; `/prune` got 6/8 expected verdicts but only 2/4 on planted
  items; the optional LLM judge did not anticipate verdicts; no real-repo
  result; no independent replication.

## Methods

- **Tasks.** A task is valid only if its check fails on the broken commit,
  passes with the fix, and passes with the fix minus its test-file edits
  (Eq. 1); otherwise quarantined, one repair, then deleted. `/harvest` is
  told to make the check also test every correction made in the session
  (e.g. linter clean), so the convention, not just the bug, is scored.
- **Sweep.** Each task is run k times under `base` (live harness) and `cand`
  (live ± item), interleaved, in per-worker sparse clones with `.evolve/`
  (task store, lessons, graveyard) excluded, reset and verified-failing
  before every rollout, 10-minute headless limit. Exposure is read from the
  Claude Code event stream: a skill is "visible" if listed and "fired" if
  invoked; a rule "fired" when a file matching `paths` was read or
  @-mentioned.
- **Scores on exposed tasks S.** Δ_t = p_c(t) − p_b(t); gain G = Σ max(Δ,0);
  net N = ΣΔ; worst drop D = max(−Δ). Protected set P = tasks the base
  passes in every run.
- **Admission gates (Table 3), applied by `score.sh`, not by a model.**
  Screen at k = 2 on the theme's failing tasks only (killed if G = 0 or never
  fired). Confirm at **k = 3 per task per arm over every valid task**:
  - Gate 0: scorable (complete, ≤10% invalid, every planned task has k valid
    runs, zero unknown exposure, one Claude Code version, ≤10% oversubscribed
    rollouts) and at least one exposed task could show a gain — else RERUN.
  - Gate 1: G > 0.
  - Gate 2 ("no task noticeably worse"): **D ≤ τ = 0.34**, i.e. no task loses
    more than 1 of its 3 runs.
  - Gate 3: every protected task passes **all 3** runs (zero tolerance).
  - Gate 4 ("net gain"): **kN ≥ m = 2 runs net**, "because with k = 3 a net of
    one run can come from luck alone".
  - Gate 5: fired f ≥ 1.
  - A KILL resting only on gates 2/3 triggers a RECHECK of the regressed
    tasks; KILL stands only if a drop repeats.
- **Theme bar.** A lesson logged ≥3 times, or ≥2 failing tasks, not covered
  by a live item; otherwise the cycle is BARREN (no rollouts). Two
  consecutive cycles without KEEP signal stop (raised to 3 in the lab).
- **Tier choice is heuristic.** Fixes clustered in one folder editing
  existing files → path-scoped rule; fixes only creating new files →
  path-gated skill (a rule cannot trigger on a file the agent never reads);
  fixes spread out → always-on skill tied to a moment. CLAUDE.md, settings,
  hooks are written and journaled but never measured.
- **Removal (Eq. 7).** `/prune` at k = 3 on tasks where the item can load:
  REJECT (keep) if kN ≤ −2, D > τ, or a protected task loses a run;
  UNMEASURED (keep) if it never fired in a base run; else ACCEPT (delete).
  Developer approves the plan first. After a model change every item is
  re-tested.
- **Lab.** 26 training scenarios played as sessions in 10 rounds; **30
  held-out scenarios** (6 per family; A changelog, B money, C exporters,
  E clock, D control) never played. The primary analysis uses the **23
  rule-family scenarios valid in every run**, `none` vs `evolved` at **k = 5**
  per scenario per run (920 rollouts), paired by scenario within run. CI by
  two-way bootstrap over runs and scenarios (B = 10,000); p by within-scenario
  label permutation; Holm across families. Hypotheses H0–H16 each with a
  pre-stated margin (not described as pre-registered).
- **Arms (all at k = 3 except none/evolved).** `none2` (A/A, R1), `kitchen`
  (CONTRIBUTING.md appended to CLAUDE.md, 2,519 chars), `ideal` (hand-written:
  always-on skills for A and E, a billing rule, a gated plugins skill),
  `flat` (evolved texts with `paths` removed), `desc-only` (skill bodies
  replaced by filler), `accept-all` (evolved + everything buried). All
  ablations ran in **one run** (R1, R2 for accept-all), in a later batch.
- **Controls.** C1: same sessions with `/evolve` off. D0: development run on
  an earlier version. Model change: R1's harness under Sonnet 5 at k = 3, and
  `/prune` with rollouts pinned to Sonnet 5. Second repo: hynek/structlog at
  73393f34b4.
- **Setup.** Claude Code 2.1.278, `claude-haiku-4-5-20251001` as agent,
  harvester, proposer and rollout model; sweep cap raised 60 → 200.
  6,906 lines of shell/Python + 1,341 lines of prompts.

## Results

- **Held-out, by family (Table 5).**

  | Family | none | evolved | Gain [95% CI] |
  |---|---|---|---|
  | A changelog | 0% | 97% | +97.0 [+91.0, +100.0] |
  | B money | 26% | 79% | +53.3 [+17.5, +88.3] |
  | C exporters | 2% | 52% | +50.0 [+25.0, +73.3] |
  | E clock | 55% | 82% | +26.7 [+1.7, +60.0] |
  | A+B+C+E | 22% | 77% | +55.0 [+34.1, +74.1] |
  | D control | 100% | 99% | −0.8 [−5.0, 0.0] |

  Per run, pooled rule-family pass rate rose 22.6% → 80.9–87.0% in R1, R2,
  R4 and to 52.2% in R3. Training-scenario gain was +78.8, mostly because the
  baseline was lower there (6.4% vs 21.5%).
- **Noise floor.** A/A (`none2` vs `none`, R1, all five families): −3.3
  [−7.3, 0.0], p = 0.13. In the later batch the A/A pair differed −4.2
  [−9.2, 0.0] on the rule families.
- **Failure mode.** Without a harness, 94% of failed rule-family rollouts
  (341/361) passed the tests and broke the house rule. The agent opened
  CONTRIBUTING.md, where the rules are written, in 0.5% of `none` rollouts.
- **Loop activity.** 39 cycles; 14 barren (13 after every rule family had an
  item); 26 candidates swept; 18 KEEP, 5 KILL, 3 RERUN buried by the agent.
  1,818 sweep rollouts, 1,494 (82%) on confirms. Every run kept an item for
  each rule family and none for D (H9).
- **Scope failures the gates cannot see.** R3 narrowed its money rule to 5
  named files; it loaded in **0 of 30** held-out money rollouts (gain 0.0).
  Its narrowed clock rule loaded in 10/30 (gain −6.7). Conversely 7 kept rules
  on `shop/**` fired on 93–100% of all tasks, including 97–99% of other
  families' rollouts (Table 7), so H3 (sensible tier) failed, 15/18.
- **Loaded vs chosen.** Kept: 16 rules, 2 skills (both exporter checklists on
  `shop/plugins/**`). Path-gated candidates: listed 66, opened 28. Always-on
  candidates: listed 198, opened 0. Placebos planted in the prune test:
  listed 1,263 times, opened 46.
- **R1 ablations (k = 3, single run, later batch).**

  | Arm | always-on chars | rule-family pass | arm − evolved [95% CI] |
  |---|---|---|---|
  | evolved | 559 | 87.5% | — |
  | kitchen | 2,519 | 76.4% | −11.1 [−22.5, 0.0] |
  | flat | 1,688 | 75.0% | −12.5 [−28.3, +1.7] |
  | desc-only | 559 | ≈73.6% | −13.9 (recovered 79% [58%, 100%] of evolved's gain) |
  | ideal | 728 | 48.6% | −38.9 [−58.1, −20.0] |

  `flat` passed every A, B and E rollout but **0% of C**: without `paths`
  the exporter skill became always-on and was opened in none (evolved 70%).
  `ideal`'s always-on skills for A and E were listed in all 90 held-out
  rollouts and opened in none; A and E stayed at `none` levels (0%/56% vs
  0%/57%). `desc-only` could rewrite only 1 of R1's 4 items (the rest are
  rules); on C it recovered 16% [0%, 56%].
- **Gate test (Table 6).** Placebos (20 always-on skills): 0 fired; 12 KILL
  by gate 5, 8 RERUN. Harmful: 2 always-on skills never fired (KILL); 3 rules
  fired on every screen rollout but all RERUN (screens run on failing tasks,
  where no harm can show; one screen included a passing task the rule broke
  and still returned RERUN). Known-good: rule KEEP, gated skill KEEP, 2
  always-on skills lost (never opened). Placebos 0/20 kept: Clopper–Pearson
  upper bound 17%. Fisher known-good vs placebo p = 0.022. **10 candidates
  showed a positive screen gain without ever firing, 9 of them from a single
  run on a single task.** Gate 5 turned them away.
- **Do the gates add value (H6)?** `accept-all` − evolved = −8.9 [−16.1,
  −1.7], p = 0.075, but it added one candidate that the `/evolve` agent
  buried by judgement (overlap) after a RERUN, so the result rests on one
  agent burial: inconclusive.
- **Corrections.** 60 in 104 evaluation sessions (0.58/session) vs 23 in 26
  control (0.88); descriptive only.
- **Cost.** Held-out rollout $0.121 (none) vs $0.149 (evolved), +23%; 22.1 vs
  18.2 turns. Per run: $32.76–$128.78 sweep rollouts, $2.39–$4.09 `/evolve`
  turns, $10.08–$11.78 scripted sessions.
- **Model change.** Sonnet 5 `none` already 88.9% (Haiku 20.8%); harness
  → 94.4%, +5.6 [−2.8, +13.9] vs +66.7 [+51.7, +80.8] under Haiku (same R1
  harness). `/prune` (k = 3, Sonnet 5 rollouts) flagged money rule (Sonnet
  passed every money scenario unaided) and exporter checklist (which still
  showed +11.1 [−11.1, +38.9] on C held-out).
- **Prune test.** Kept R1's 4 own items. Planted: deleted 1 placebo
  always-on skill, skipped the unreachable gated skill, kept the other
  placebo (REJECT: 2 tasks lost runs on removal, though it was opened in 0 of
  18 listings), and the never-firing always-on skill got no verdict
  (left out of the plan). No never-fired item deleted.
- **structlog.** 8 tasks harvested, 1 lesson; 7/8 sessions passed tests and
  linter first try; every cycle barren; `evolved` byte-identical to `none`.
- **Judge (Jev, off during runs, replayed after).** Relevance did not
  separate kept from regression-killed (lowest kept 17%, highest killed 75%);
  on only 2 regression kills.

## Critique / open questions

- **Against the digest framing: AT.** Each clause is supported by the body
  and mirrors the abstract, including the honest "keeps no unused text, not
  that it detects damage". The specifics the digest left open: "net gain" is
  **≥2 runs net summed over exposed tasks at k = 3 per arm**; "noticeably
  worse" is **any task losing 2 of 3 runs (τ = 0.34), or a base-always-pass
  task losing even 1**, with a recheck before a regression-only KILL;
  screen k = 2. Held-out is **30 scenarios (23 rule-family scenarios valid in
  all runs) × 4 runs × k = 5**. One place the body is narrower than the
  digest's "auto-loading rules did most of the work": the rules-vs-skills
  comparison is not a matched experiment (next bullet).
- **The rules-vs-skills comparison is mostly observational and confounded
  with family.** The tier was picked by a heuristic: rule if fixes edit
  existing files, gated skill if fixes create new files. Only family C
  (exporters) creates new files, so both kept skills are C's, and "16 of 18
  were rules" is in large part what the heuristic produces on this lab. The
  matched evidence is (a) `flat` (one run, k = 3) and (b) the gate test's 4
  known-good items (n = 1, 1, 2 by tier). The 0/198 always-on invocation rate
  is the robust datum; it is across candidates of mixed quality, but includes
  the hand-written `ideal` skills.
- **`flat` is the same-content scoped-vs-unscoped arm, and for rules it shows
  no harm from unscoping.** Removing `paths` from R1's rules (they became
  path-less, loaded every turn) left A, B, E at 100%. The whole −12.5 came
  from C, where the skill lost its gate and stopped being opened. In this lab
  path-scoping a rule bought context cost (1,688 vs 559 always-on chars), not
  compliance; path-gating a skill decided whether it was used at all. The CI
  crosses zero and H7's margin only asks "no better than evolved".
- **"Adds no always-on context" is a tier technicality.** Seven of the 16
  kept rules are scoped to `shop/**` and fire on 97–99% of every other
  family's rollouts. Measured by what is actually in context, much of the
  evolved harness is effectively always-on; only the Claude Code tier
  accounting says otherwise.
- **The gates certify scope only on harvested tasks.** R3's over-narrow money
  rule passed every gate and then loaded in 0/30 held-out rollouts. This is
  the converse of cheng2026scope's over-wide global slot: a paired gate on
  the tasks that motivated a candidate cannot see that it will not reach the
  next task. The author says so ("Nothing in them can see that a scope is too
  narrow for tasks the loop has not yet met").
- **The harm test never ran.** Harmful skills were never opened; harmful
  rules were screened only on tasks the live harness failed, where a drop is
  impossible, so `score.sh` returned RERUN even when a rule broke a passing
  task. Gates 2–3 (regression) were exercised only in the loop itself (2 of
  5 KILLs). Whether the no-regression gates catch a loaded harmful rule is
  untested. The placebos were all always-on skills, so no placebo that loads
  was ever tested either.
- **k = 3 is thin for a regression gate.** τ = 0.34 means one lost run of 3 is
  tolerated, and the recheck exists because one unlucky run looks like a
  broken task. The prune test shows the converse false positive: a placebo
  opened 0/18 times was kept (REJECT) because 2 tasks lost runs when it was
  removed. With an A/A gap of −3.3 to −4.2 points and per-task k = 3, single
  sweeps resolve only large effects (cf. liu2026what's 13% rerun flips).
- **Lab built to show a gain.** Scenarios the model solved unaided were
  redesigned until it broke the rule; family A is checked only by "changelog
  gained a line"; 15 of the 30 held-out scenarios were written after the
  development run, when kept items were known (from CONTRIBUTING.md and
  family definitions, not from evolved items). Held-out scenarios test the
  same 4 rules on new work, not new conventions: they show transfer within a
  convention, which is what a developer wants, but the +55 is an upper-end
  setting by design.
- **Haiku plays every role**, including proposer and the agent that buries
  RERUN candidates by judgement (3 of 26). The +55 is a Haiku number; under
  Sonnet 5 a single R1 harness added +5.6 with CI including zero.
- **Sandbox leakage is reported and partly quantified.** Git history is
  reachable from the clone. Under Haiku it was rare (2/1,809 loop rollouts
  printed a harvested fix; 112/595 held-out `none` rollouts saw item names
  via `git status`, 2 printed text). Under Sonnet 5, 19/40 benchmark rollouts
  and 1/8 training sessions read the upstream fix through git, so that run
  does not show unaided behaviour.
- **Few runs.** 4 evaluation runs and 1 control: the bootstrap over runs draws
  from few distinct sets, H4 cannot reach p < 0.2, and all ablations are one
  run at k = 3 in a later batch than the primary arms.
- **What it does not show:** that Cortex helps on a real repository (structlog
  had nothing to learn at its correction rate); that it handles conventions
  needing judgement to check; that the tier heuristic picks the right tier;
  that CLAUDE.md edits (written but never measured) help; anything about
  harm detection.
- **Open for this graph:** a matched-content rule-vs-skill arm (same text as
  path-scoped rule and as path-gated skill, on files the agent reads) would
  separate tier from family; a harm sweep that includes passing tasks; and
  whether the exposure rule survives harnesses whose "fired" signal is less
  observable than Claude Code's event stream.

## Trust signals

- **Credibility:** 3.
  - For: public code with the evaluated tag (`v1.0-eval`) and a data +
    regeneration tag (`v1.0.5`), both verified on GitHub on 2026-10-19;
    every verdict re-implemented and checked against `score.sh` on all 51
    sweeps; replicated runs (4) with a control run, an A/A arm, and
    pre-stated margins for 17 hypotheses with failures reported as failures
    (H3, H5, H10, H15 not supported; H6, H11, H14, H16 inconclusive);
    known-answer tests of both gates; quantified sandbox-leakage audit.
    Claims are scoped carefully in the abstract.
  - Against: single independent author, not peer reviewed, no citations
    yet, no independent replication (stated). One synthetic repository built
    and calibrated by the author, a scripted developer, one small model in
    every role. Raw session records are not released (only exported rows).
    The acknowledgement says AI participation was "substantial and
    continuous" across design, implementation, statistics and editing.
  - Not 4 because the setting is a single-author synthetic lab with one
    model; not 2 because the release, disclosure and controls are far above
    the typical preprint in this graph.

## Follow-up

- **Relevance:** 4. It is the first source in this graph measured on this
  box's exact substrate: Claude Code `CLAUDE.md`, path-scoped
  `.claude/rules/` and skills, with exposure read from the event stream. It
  supplies the same-content scoped-vs-unscoped arm that
  [[concepts/enforcement-boundary-placement]] recorded as missing (`flat`),
  an exposure-symmetric admission/removal rule for
  [[concepts/skill-library-lifecycle]], and a planted-placebo audit for
  [[concepts/hce-evaluation]]. Not 5: synthetic lab, Haiku-only gain, single-
  run ablations, and the tier result is confounded with task family.
- **Practical reads for this box (our inference):**
  - Always-on skills (no `paths`) were listed 198 times and opened 0 times.
    This box's user-global skills are always-on by this definition and rely
    on user invocation or description match; a skill whose value depends on
    the model choosing it unprompted should be checked against invocation
    logs, not assumed.
  - A convention the agent must follow while editing particular files is
    better delivered as a path-scoped rule than as a skill; a rule scoped to
    `**` is effectively always-on context.
  - Scoping to named files rather than folders can silently stop a rule
    loading on new work (R3: 0/30).
  - Any keep/retire decision about a rule or skill should be made only on
    sessions where it actually loaded; "never fired" means unmeasured, not
    useless.
- **Candidates cited, not in the graph:**
  - Zhang et al. 2026, "Guardrails Beat Guidance: A large-scale study of
    rules, skills, and persistent configuration for coding agents" (arXiv
    2604.11088) — randomly drawn rules did as well as curated ones in Claude
    Code on SWE-bench Verified.
  - Gloaguen et al. 2026, "Evaluating AGENTS.md" (arXiv 2602.11988).
  - Khatri 2026, "Do context files help coding agents?" (arXiv 2607.27250).
  - Kozyrev et al. 2026, "Skill Issue: lessons from optimizing repository
    SKILLs for coding agents" (arXiv 2609.12742) — gains not separable from
    run-to-run noise.
  - Han et al. 2026, "Skill-Use: can LLMs actually use skills in agentic
    harnesses?" (arXiv 2608.04828).
  - Ke et al. 2026, "EvoHarnessBench" (arXiv 2609.04280) — Claude Code
    invoked about one in seven offered skills.
  - Saha 2026, "Delivery, not storage: cue-anchored working memory" (arXiv
    2607.20972) — injected facts arrived, offered memory was never consulted.
  - Chakrabarti 2026, "Why does CLAUDE.md keep growing?" (arXiv 2608.11095).
  - Zhan et al. 2026, "Safe skill retirement for physical agents" (arXiv
    2609.29543) — the "unmeasured support gap" behind UNMEASURED.
  - Zhang et al. 2026, "DarwinX" (arXiv 2608.07545) — source of the
    net-gain/bounded-regression contract Cortex adapts.
