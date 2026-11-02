---
kind: paper
title: "Does Your Generalist Skill Optimizer Really Incentivize Agentic Tasks? Rethinking Skill Optimization via Bipartite Attribution"
authors: ["Jiawei Zhou", "Bo Peng", "Yuxuan Huang", "Ming Zhang", "Youbang Sun", "Xiang Wang", "Chaochao Lu"]
institutions: ["Shanghai AI Laboratory", "Fudan University", "Shanghai Jiao Tong University", "University of Science and Technology of China", "Tsinghua University"]
year: 2026
venue: "arXiv 2610.37201 (cs.AI), v1 2026-10-27, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.37201"
code_url: "https://github.com/valleysprings/BiPA"
citations: null
source: "raw/papers/zhou2026does.pdf"
added: "2026-11-02"
relevance: 5
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["skill-optimization", "acceptance-gate", "selection-only-null", "winners-curse", "noise-band", "dev-test-gap", "token-cost", "turn-elasticity", "rule-accumulation", "skill-length", "gepa", "skillopt", "skilladam", "terminal-bench", "tau-bench", "automationbench", "opencode", "verification-before-completion", "elevate-candidate"]
---

# Does Your Generalist Skill Optimizer Really Incentivize Agentic Tasks? Rethinking Skill Optimization via Bipartite Attribution

## TL;DR

The authors run three published skill optimizers (GEPA, SkillOpt,
SkillAdam) from one hand-written seed skill. The seed is Superpowers'
`verification-before-completion`. The agent is OpenCode with
DeepSeek-V4.1-Flash or GPT-6-Luna, on Terminal-Bench 2.1, τ³-bench and
AutoBench. Each optimizer runs once per cell, which gives 18 runs and 16
learned skills (two runs returned the seed). Every skill is then tested
three times on held-out tasks.

Three results matter for this graph:

1. **The learned skills do not beat the seed.** Over all 16, test success
   is +0.1 points (one-sided 95% upper bound +1.6). Over the nine DeepSeek
   skills it is −1.4 (upper bound +0.6). Only one of 16 beats the seed by
   more than the seed's own run-to-run spread (6.4 vs 5.8 points).
2. **The reported dev gains are what selection alone produces.** BiPA
   simulates each optimizer's acceptance rule on candidates that are
   identical to the seed. Those null candidates reach the reported dev gain
   with probability 0.25–0.94 under DeepSeek and 0.21–0.99 under Luna.
   Much of the null gain comes from one detail: SkillOpt and GEPA compare
   against the seed's *lowest* dev run. Across nine DeepSeek skills, test
   difference vs reported dev gain has slope 0.02 and correlation 0.05.
3. **The skills cost more, through turns, not length.** Learned skills read
   1.16× the seed's input tokens (1.18× DeepSeek, 1.14× Luna). Editors
   almost always add text (58 of 59 DeepSeek edits). About half the added
   rules ask the agent to re-check work. None asks it to read less. The
   agent carries the rules out as extra turns, and each turn re-sends the
   whole history. Across edits, the change in turns explains 93% of the
   variance in token change; skill length explains 10%.

## Claims

- On long-horizon tasks, a gain on a small dev set does not show that an
  edit helps. Keeping the best of several noisy scores is a winner's curse
  (Smith and Winkler 2006).
- A success score hides cost. Extra turns raise input tokens faster than
  turns, because the harness re-sends the history every turn.
- The editor writes several rules per edit, and the gate scores the whole
  edit with one number. No rule is credited alone, so rules accumulate.
- Learned skills "sharpen" the agent (episodes on a task agree more) but do
  not "expand" it (cover new tasks).
- Four design principles (Section 6):
  1. accept an edit only if its gain exceeds what selection alone reports;
  2. charge an edit for the turns it adds, not for its length;
  3. let the editor delete and merge rules;
  4. show the editor what its earlier rules cost.
- BiPA is a measurement instrument. It applies to other optimizers, seeds,
  models and harnesses without change. These optimizers are static
  propose-verify-accept loops, not recursive self-improvement.

## Methods

- **Optimizers** (released code, upstream prompts, Appendix A and G.1).
  - GEPA: 20 rounds; rewrites a parent after a batch of 8 training tasks;
    scores the candidate on the whole dev set if it beats the parent on the
    batch; returns the best on dev. Uses the generic `optimize_anything`
    template, not `gskill`.
  - SkillOpt: keeps a candidate whose dev score is strictly above the
    current skill. **One pass, where upstream makes four** (4 rounds of 40
    training tasks). Upstream SkillOpt also scores on test; here test never
    selects.
  - SkillAdam: 16 steps of 10 training tasks; keeps an edit only if it beats
    the current skill on 10 dev tasks drawn fresh each step (the same 9 on
    TB2); returns the last kept skill.
- **Seed.** `verification-before-completion` (Vincent 2025, Superpowers),
  580 words, full text in G.2. One shared seed for all optimizers.
- **Splits.** Each benchmark split 7:1:2 into train, dev, test. Dev is 30
  tasks (9 on TB2). Test is 18 / 75 / 120 tasks (TB2 / τ³ / AutoBench); 17
  on TB2 for Luna. One sample of 160 training episodes per benchmark.
- **Harness.** OpenCode v1.14.31, whole history kept (no compaction). The
  skill goes in the first user message, not the system prompt.
- **Runs.** One optimizer run per (optimizer, model, benchmark). Every
  skill, the seed and a no-skill agent run three times on test (avg@3,
  k = 3). The seed has two or three dev runs under DeepSeek and one under
  Luna.
- **Selection replay (4.1, C.3).** Each dev task gets a difficulty p_v from
  a Beta(a, b) prior fitted by maximum likelihood on the seed's dev and test
  episodes. A replay draws p_v from its posterior, draws outcomes for K
  unchanged candidates, and applies the optimizer's acceptance rule. The
  reported number is P(G₀ ≥ G) over 100k replays, checked under four task
  models. **The null candidates are simulated, not re-run and not
  re-scored from recorded episodes.** The model assumes episodes are
  independent given p_v (A1) and difficulties are exchangeable (A2).
- **Bipartite graph (4.2–4.4).** Each episode is a graph between turns and
  context components (skill text, rest of prompt, bash output, read output,
  messages). The change in log input tokens splits into four terms: more
  actions, turn-splitting, content per turn, skill text. Turn elasticity
  e_R = 1 + s_H·R/(R−1) lies between 1 and 2 (Proposition 1, proved in B).
- **Rule coding (E.1).** Each added rule is sorted into eight categories by
  keyword search and checked by hand. A second coder agreed at Cohen's
  κ = 0.76 on 277 rules.
- **Statistics.** Paired bootstrap per skill; cluster bootstrap over tasks
  for pooled numbers; sign test for coverage; Mann–Whitney for kept vs
  rejected edits.

## Results

- **Test success (Table 1).**

  | Model | Bench | Seed | No skill | Learned (GEPA / SkillOpt / SkillAdam) |
  |---|---|---|---|---|
  | DeepSeek | TB2 | 75.9 | 72.2 | 74.1 / 64.8 / 68.5 |
  | DeepSeek | τ³ | 76.4 | 73.8 | 75.6 / 79.1 / 75.1 |
  | DeepSeek | AutoBench | 45.0 | 36.4 | 44.2 / 41.7 / 44.7 |
  | Luna | TB2 | 68.6 | 70.6 | seed / seed / 74.5 |
  | Luna | τ³ | 69.3 | 64.9 | 65.3 / 63.1 / 64.9 |
  | Luna | AutoBench | 37.2 | 27.5 | 41.4 / 42.8 / 43.6 |

  The seed's spread (best minus worst of its three test runs) is 5.6 / 4.0
  / 8.4 points under DeepSeek and 5.9 / 8.0 / 5.8 under Luna. Seven of nine
  DeepSeek skills differ from the seed by less than that.
- **Dev gains are selection.** Under DeepSeek, SkillOpt and GEPA reported
  dev gains of 3 to 17 points and SkillAdam 20 to 40 (summed over small
  batches). A gate that accepts any candidate beating its parent in one dev
  run accepts an unchanged candidate 24–41% of the time. Every DeepSeek
  SkillOpt and GEPA run started from the seed's lowest-scoring dev run. The
  seed's other dev runs scored 1 to 5 tasks higher, about the size of the
  reported gains. If the baseline is drawn at random instead, the mean null
  gain falls to 0.6–2.3 tasks, and the τ³ gains become unlikely under the
  null (P 0.02–0.17).
- **Test does not follow dev.** On dev, the returned SkillOpt and GEPA
  skills newly solve 12.5% of tasks and newly fail 2.7%. On test it is 8.9%
  vs 6.9%, close to the 1.9–7.8% churn between two runs of the seed itself.
  Along the τ³ paths, the dev gain climbs 6.7 points after the first kept
  skill while the test difference stays within 2.7 (SkillOpt) and 1.3
  (GEPA). Luna on AutoBench is the exception: test gains (15.8% / 15.0%
  newly solved vs 7.5% / 4.2% newly failed) with little dev movement.
- **Sharpen, not expand (F.2).** Pooled over six model–benchmark pairs, the
  seed covers 36 tasks the no-skill agent misses and loses 13 (sign test
  p = 0.001). The learned skills cover 57 tasks the seed misses and lose 62.
  Where success rose, episodes agreed more (odds ratio 1.61, p = 0.03).
- **Cost (D.1–D.3).** All nine DeepSeek skills read 1.08–1.43× the seed's
  input tokens and take 1.03–1.22× its turns. On tasks both skills solve
  in every run the ratio is still 1.08–1.29. Within a task, 10% more turns
  read 14–16% more input tokens (turn slopes 1.35–1.58). Turn terms carry
  73% of the summed log-token increase and skill text 21%. Under cache
  pricing the skills cost 1.05–1.27×. Re-billed as if history were
  compacted, 1.05–1.36×.
- **What editors write (E.1).** 58 of 59 DeepSeek edits lengthen the skill
  (33 of 38 under Luna); one shortens it. Skills grow to 654–2.4k words from
  580. Of 337 added DeepSeek rules, 52% ask the agent to verify, 18
  serialize calls, 1 parallelizes reads, 0 limit reading. Under Luna: 81
  rules, 41% verify, 3 serialize, 2 parallelize.
- **Serialized writes (E.2).** On AutoBench, 56% of the seed's writes share
  a turn with another write, against at most 0.3% under each learned skill.
  Repacking calls at the seed's density halves AutoBench's log ρ (0.17 →
  0.08). The authors note the serialization has a reason: concurrent writes
  can overwrite each other.
- **The gate does not filter cost (E.3).** Kept and rejected edits raise
  log input tokens alike (0.100 vs 0.128, p = 0.46). Only 5 of 59 edits keep
  dev success without adding tokens.
- **Acceptance rules, simulated (C.4).** On the fitted task model:

  | Rule | Unchanged accepted | True two-task gains kept |
  |---|---|---|
  | (1) one dev win (SkillOpt, SkillAdam) | 24 / 41 / 39% | — |
  | (2) win ≥ critical value (α = 5%) | 2.7–3.2% | 14–21% (τ³, AutoBench) |
  | (3) win repeated on a second dev run | 6.4 / 17.1 / 15.2% | 94 / 55 / 62% |
  | (4) higher total over three dev runs | 34–45% | — |

  Rule (2) would have refused 29 of the 30 kept DeepSeek edits. Among 43
  DeepSeek edits, 9 changed tokens beyond noise and only 1 changed success
  beyond noise. Under Luna, tokens separated only 2 of 38.
- **Transfer (F.1).** 8 of 9 transferred skills score below the seed. Most
  of their extra cost goes with them: TB2 skills read 1.24–1.49× the seed
  on AutoBench. One run per transfer.
- **Weak seed (C.5).** From a three-word seed ("Solve the task.") under
  Luna, SkillOpt and GEPA raise AutoBench test success from 25.0 to 43.3 and
  44.4, about the level of the runs from the hand-written seed (42.8,
  41.4). On τ³ they add nothing; the null reaches their dev gains with
  P 0.27 and 0.74. SkillOpt re-derived the hand-written seed's
  verify-after-write rules. One run each.
- **Skill loading (F.3).** Loading on demand costs the same as always
  resident (ρ 1.230 vs 1.226). Without an activation instruction, the
  agent never loaded SkillOpt's τ³ skill (0 of 75), whose description is
  the seed's.

## Critique / open questions

- **Against the digest framing: at framing.** Every number in the digest
  summary is in the PDF and is quoted correctly: 0.1 points, ~16% more
  input tokens, ≤30 dev tasks, three optimizers × three benchmarks × two
  models, "almost always add rules". The three ingest checks come out as
  follows.
  1. **Seeds.** One optimizer run per cell. Each skill is tested three
     times. The seed has 2–3 dev runs (DeepSeek) or 1 (Luna). So optimizer
     variance is unmeasured, and the 16 skills are single draws.
  2. **Dev/test sizes.** Dev is 30 tasks (9 on TB2). Test is 18 / 75 / 120.
     The TB2 test set is very small.
  3. **"Identical candidates" are neither re-run nor re-scored.** They are
     simulated: outcomes drawn from a Beta-Binomial task model fitted on the
     seed's own episodes. It is checked against observed solve shares
     (Fig. 6) and across four task models. But "selection alone reports" is
     a model-based probability, P(G₀ ≥ G). No real unchanged candidate went
     through a real gate.

  Three qualifications the digest leaves out. They narrow the result; they
  do not move the verdict.
  - Much of the selection-only gain comes from comparing against the
    seed's **minimum** dev run. That is a fixable implementation choice.
    With a random baseline, the τ³ gains would be unlikely under the null.
  - "One call at a time" is 18 of 337 rules (5%). It drives AutoBench's
    turn-splitting, and it has a correctness reason (overwrites).
  - The digest lists three principles; the paper has four (the fourth:
    show the editor what earlier rules cost). Principle 1 is supported by
    simulation on the task model. No principle was built into an optimizer
    and re-run.
- **The null is relative to a strong seed.** The hand-written seed already
  gains on no skill in 5 of 6 cells (36 vs 13 tasks covered). From three
  words, the same optimizers gain 18–19 points on AutoBench. So the result
  is "optimizers do not improve a competent hand-written skill", not
  "optimizers do nothing". The title's "generalist" is the right hedge.
- **Optimizers at reduced or generic settings.** SkillOpt ran one pass
  where upstream runs four, and GEPA used its generic template. The
  authors give reasons (a shared seed and comparable edits). A longer run
  makes more selections on the same 30 dev tasks, so it is not clear more
  budget would help. Still, the paper does not test published SkillOpt
  configurations against the headline claims in
  [[literature/papers/yang2026skillopt]] ("zero added inference-time model
  calls", harness portability). Its data contradict both in a multi-turn
  harness: turns rise 1.03–1.22× and the skills do not transfer.
- **The verify seed shapes the edits.** Half the added rules are "verify",
  on top of a seed that is a verification procedure. With a different seed
  the rule mix, and so the turn cost, may differ. The three-word run
  suggests the editor re-derives verification on AutoBench anyway.
- **Model drift.** An updated DeepSeek-V4.1-Flash release appeared during
  the study. The seed rerun on it solved 5.8 points fewer τ³ tasks than the
  earlier three runs. The τ³ path analysis and the loading checks use the
  updated release with a new baseline. Within-figure comparisons hold;
  cross-release levels do not.
- **Cost is in tokens, not dollars or wall-clock.** The cache-pricing and
  compaction re-billing (D.3) are recomputations of recorded episodes, not
  runs under a compacting harness.
- **Fit with the rest of the graph.** It is the skill-optimizer counterpart
  of [[literature/papers/hu2026winners]] (winner's curse in instruction
  optimization, measured with ground truth) and of
  [[literature/papers/he2026does]] (selection over zero-mean edits
  reproduces DGM's curves). It also extends
  [[literature/papers/ding2026harnessdelta]], where a skill-only
  edit "never scales compute" in a one-call rollout. In a multi-turn
  harness, a skill-only edit does scale compute, through turns. The
  rule-accumulation result matches he2026does' finding that failure-driven
  revision almost never removes a rule.

## Trust signals

- **Credibility:** 4.
  - For: Shanghai AI Laboratory with Fudan, SJTU, USTC and Tsinghua
    co-authors. Code and the nine DeepSeek skills released
    (github.com/valleysprings/BiPA, named in the PDF; not opened at
    ingest). Optimizers run from their released code with the supplied
    prompts printed. Paired and cluster bootstraps, one-sided bounds, a
    second coder with κ reported. The null replay is checked across four
    task models. The limitations section is specific and honest
    (post-hoc, no-compaction harness, success not decomposed).
  - Against: not peer reviewed, 6 days old, no citations. One optimizer
    run per cell, three test runs per skill, an 18-task TB2 test set. The
    selection null is a parametric simulation, not real reruns. One seed.
    SkillOpt run at a quarter of its upstream passes. Model drift partway
    through.
  - Not 5 because there is no peer review or replication and every cell is
    a single optimizer run. Not 3 because the design is careful, the
    artifacts are released, and the main null (+0.1, upper bound +1.6 over
    16 skills) does not hinge on any one cell.

## Follow-up

- **Relevance:** 5. It measures directly what this graph's acceptance-gate
  and noise-band arguments have inferred. A keep-if-better gate on ≤30
  tasks and one or two runs reports gains like these with nothing changed.
  The `/iterate` keep rule has that shape. It is also the first measurement
  in the graph of what a *skill file's* extra rules cost per task, and the
  mechanism (turns, not length) applies to this box's own skills and
  CLAUDE.md. It adds material evidence to [[concepts/hce-evaluation]],
  [[concepts/budget-as-ceiling]], [[concepts/skill-library-lifecycle]] and
  [[concepts/evidence-gated-completion]].
- **Bears on the pending proposals.**
  - `hce-known-bad-preflight` (10-25): this is the gate-side twin of a
    known-bad control. Score the unchanged configuration through the same
    acceptance rule and see what gain it reports. Here it is simulated,
    and cheap once per-task solve rates exist.
  - The accepted 09-27 noise band: the C.4 simulation is the first evidence
    in the graph that a **repeat-the-win** keep rule beats a
    critical-value rule on power (55–94% vs 14–21% of true two-task gains
    kept) at a modest false-accept cost (6–17%). It also shows why the
    starting score must not be the seed's best-of or worst-of runs.
  - `iterate-redirect-before-halt` (10-18): no direct evidence. The
    optimizers never halt or redirect.
- **`/elevate` candidate (flag only).** Two cheap, testable rules:
  - In `/iterate`, measure the incumbent's baseline as a mean or a fresh
    draw, never as the min (or max) of stored runs.
  - Require a candidate's win to repeat on a second run before it becomes
    champion, when the margin is under the band.

  Both rest on one paper's simulation. Any proposal should say that.
- **Cheap local test.** Take a box skill or CLAUDE.md rule set, run a
  pinned task set 3× with and without one added "verify" rule, and log
  turns and input tokens per task. That tests whether Proposition 1's
  turn-elasticity cost shows up in Claude Code, which compacts history.
- **Candidates** (cited, not yet in the graph):
  - Liu et al. 2026, "Rethinking self-evolving agent skills: feedback
    dynamics over multiple rounds" (arXiv 2608.02636).
  - Lin et al. 2026, "Harness updating is not harness benefit" (arXiv
    2605.30621).
  - Xing et al. 2026, "What should a skill remember? Quality–cost
    trade-offs in cost-aware skill rewriting" (arXiv 2606.09421). Shorter
    skills can cost more.
  - Li et al. 2026, "SkillAdam: stable and efficient skill evolution for
    agents" (arXiv 2609.08944). Listed in the 2026-09-14 digest, not
    ingested.
  - Weinberger and Hozez 2026, "Token reduction is not cost reduction"
    (arXiv 2607.12161).
  - Gao et al. 2026, "SkillReducer" (arXiv 2603.29919), and Gong et al.
    2026, "SkillMOO" (ASE '26). Cost-aware skill optimizers.
