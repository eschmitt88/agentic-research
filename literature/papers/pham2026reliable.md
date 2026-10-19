---
kind: paper
title: "Reliable Memory from Unstable Growth: Curating Self-Written Agent Playbooks with Randomized Experiments"
authors: ["Hoang Pham", "Cong-Duy Nguyen", "Dong Le", "Ponhvoan Srey", "Anh Tuan Luu"]
institutions: ["Nanyang Technological University", "VinUniversity"]
year: 2026
venue: "arXiv 2610.18427 (cs.CL), v1 2026-10-13, preprint; ACL-style format with an anonymized code link, so probably under double-blind review"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.18427"
code_url: "https://anonymous.4open.science/r/rbc"
citations: null
source: "raw/papers/pham2026reliable.pdf"
added: "2026-10-19"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/agent-native-memory]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/hce-evaluation]]"
tags: ["self-written-memory", "playbook", "ace", "dynamic-cheatsheet", "memory-curation", "randomized-inclusion", "banzhaf", "factorial-design", "self-judgement", "helpful-tags", "growth-run-variance", "best-full", "rsea", "no-playbook-fallback", "token-budget", "practice-split", "appworld", "webshop", "alfworld"]
---

# Reliable Memory from Unstable Growth: Curating Self-Written Agent Playbooks with Randomized Experiments

## TL;DR

Self-written agent playbooks (ACE's appended lessons, Dynamic Cheatsheet's
rewritten cheatsheet) are unstable across growth runs: the same procedure
on the same tasks in another order can produce a playbook that helps or
one that harms. The agent's own helpful/harmful tags do not identify the
harmful lessons. RBC replaces the "judge" step with a randomized experiment
on a practice split. Lessons are clustered into 8 blocks; on each practice
episode every block is included independently with p = 1/2. A
task-intercept ridge regression of success on the inclusion indicators
estimates each block's average effect (its Banzhaf value). Positive blocks
are kept, best first, under a 40% token cap. Over K = 3 growth runs, RBC
then scores 7 candidates (3 curated, 3 whole, none) on new practice
episodes and serves the best. Averaged over 19 settings (2 memory systems,
3 open models, 3 benchmarks), RBC scores 66.9, against 55.5 for the whole
playbook and 54.1 for no playbook. **The strongest baseline, Best-Full
(RSEA's rule: choose among the 3 whole playbooks and none on the same
practice episodes), scores 66.4.** Most of the headline gain therefore
comes from *choosing among runs with a no-playbook fallback*, not from
curating. Curation adds about 1 point on ACE, nothing on Dynamic
Cheatsheet, and 3.1 points when ACE playbooks grow long.

## Claims

- **Finding 1: serving the whole playbook can harm.** Of the first growth
  runs in 11 ACE settings, 5 help, 2 are level, and 4 harm (more than 5
  points below no playbook). The worst run is 69.6 points below no
  playbook (27B AppWorld, pass/fail feedback; every run there harms).
- **Finding 2: the growth run decides.** On 27B AppWorld with per-test
  feedback the three runs score 82.5, 58.9 and 30.0 against 78.4 with no
  playbook. Swings like this occur in 3 of the 6 settings that have a
  harmful run. Dynamic Cheatsheet shows the same thing (9B WebShop: one
  run +13.2, another −14.0; Gemma ALFWorld runs 37.3 points apart).
- **Finding 3: the agent's own judgement misses the harm.** Cutting the
  playbook to 40% of its tokens by ACE's tags still leaves it harmful in 3
  of 4 harmful settings. Over 81 blocks, tags and measured effects have
  rank correlation 0.21. The blocks that harm by more than 5 points carry a
  median of 33 helpful tags per lesson, more than every group except the
  most helpful blocks (44). One example rule ("always pass a concrete,
  non-empty string literal as the answer argument" to `complete_task`) was
  tagged helpful 123 times and harmful 0 times. It raised the rate of
  answering tasks that expect no answer from 37.5% to 69.0%.
- **Proposition 1.** With independent p = 1/2 inclusion, the
  within-task difference of means is unbiased for every block's average
  marginal effect at once, with variance about 4σ²/n, whatever the
  interactions. Leaving out one block at a time would need (m+1)/2 = 4.5
  times as many episodes for the same precision. Ridge with a balanced
  design shrinks all estimates by about the same factor, so signs and
  order survive.
- **Proposition 2.** Choosing the best of C candidates on n independent
  practice episodes loses at most 2·sqrt(log(2C/δ)/(2n)) of true practice
  success against the best candidate, and so against no playbook. The
  authors say it is loose: about a quarter of the scale at C = 7, n = 171.
  It covers practice tasks only.
- **"An agent's memory should be measured, not judged."**

## Methods

- **Memory systems.** ACE in batched mode: a Reflector tags lessons as
  helpful/harmful/neutral, a Curator appends new lessons, near-duplicates
  are merged, nothing is removed, the whole playbook is served. Dynamic
  Cheatsheet cumulative mode: one call rewrites the cheatsheet per batch
  and counts each entry's successful uses, without the reward.
  ReasoningBank-style Top-k similarity retrieval is a baseline, not a
  system.
- **Models.** Qwen3.8-27B, Qwen3.5-9B, Gemma-4-31B (FP8), thinking off,
  each model playing every role. Actor temperature 0.7, other roles 0.
- **Benchmarks and splits.** Three disjoint task sets per benchmark:
  - AppWorld: 90 training tasks, practice = 57 dev-split tasks, evaluation
    = test_normal 168 tasks × 3 seeds.
  - WebShop: 500 training instructions, 300 practice instructions,
    evaluation = 500 test instructions × 3 seeds.
  - ALFWorld: 500 training games, practice = eval_seen 140 games,
    evaluation = valid_unseen 134 × 3 seeds.
  That gives 11 ACE settings (AppWorld counts twice: pass/fail and
  per-test feedback) and 8 Dynamic Cheatsheet settings. Evaluation tasks
  enter no method's selection.
- **RBC.** Embed lessons (all-mpnet-base-v2), Ward-cluster into m = 8
  blocks. One practice round (684 / 600 / 600 episodes) with Bernoulli(1/2)
  inclusion per block. Ridge fit with an unpenalized intercept per task,
  λ chosen by leave-one-task-out. Keep positive blocks in effect order
  until a 40% token budget is full; if none is positive, serve nothing.
  Then score 2K + 1 = 7 candidates on new practice episodes (171 / 150 /
  140 per candidate), which reuse none of the estimation episodes but do
  reuse the same practice tasks. RBC calls no language model.
- **Baselines.** All are cut to the same budget from the same playbooks:
  Own (ACE tags, or DC usage counts), Top-k similarity, Random. Best-Full
  is RSEA's acceptance rule applied across K runs. RBC-cut curates one run
  without choosing. Advantaged variants Tags* and Top-k* are tuned *on
  the evaluation tasks*. DC_self and DC_outcome are LLM curators, the
  second told each outcome. DG-Mem's subset-Shapley estimator is swapped
  into RBC's pipeline.
- **Tuning discipline.** K = 3, B = 40%, m = 8 were fixed on four ACE
  development settings, using held-out pools (AppWorld test_challenge and
  half of WebShop's practice tasks), never their evaluation tasks. A
  one-at-a-time tuning round under a pre-stated rule (move only if +1
  point on the four-setting mean) moved nothing.
- **Scoring.** Each cell averages 3 evaluation seeds and, for methods that
  serve one run, the 3 growth runs. There are no confidence intervals or
  significance tests on Table 1. The noise floor quoted is "re-evaluating
  a playbook moved its score by at most 5.0 points".

## Results

- **Table 1 averages (task success 0–100).**

  | | No pb | Whole | Own | Top-k | Random | Best-Full | RBC-cut | RBC |
  |---|---|---|---|---|---|---|---|---|
  | ACE (11) | 54.4 | 51.2 | 50.9 | 50.0 | 50.9 | 66.1 | 61.9 | 67.2 |
  | DC (8) | 53.8 | 61.3 | 57.7 | 58.9 | 58.0 | 66.9 | 61.1 | 66.5 |
  | Both (19) | 54.1 | 55.5 | 53.8 | 53.8 | 53.9 | 66.4 | 61.6 | 66.9 |
  | ACE largest drop below no pb | – | 69.6 | 72.6 | 60.3 | 60.6 | 5.4 | 12.2 | 0.8 |
  | ACE tokens served (% whole) | 0 | 100 | ≤40 | ≤40 | ≤40 | 78 | 36 | 60 |
  | DC tokens served (% whole) | 0 | 100 | ≤40 | ≤40 | ≤40 | 100 | 35 | 93 |

  RBC is above the whole playbook in 15 of 19 settings. On Dynamic
  Cheatsheet it shares Best-Full's two AppWorld drops below no playbook
  (largest 7.1).
- **Decomposition (Table 10, all 19 settings).** Curate one run 61.6;
  choose on one run 62.1; both on one run 62.9; choose among three runs
  (Best-Full) 66.4; both among three (RBC) 66.9. Choosing among runs is
  the large step (+4.3 over one run); curating on top of choosing adds 0.5.
- **Where curation pays.** On ACE, RBC beats Best-Full 67.2 vs 66.1 while
  serving 60% vs 78% of the tokens, and leads in 5 of the 6 settings where
  their choices differ. It gains most where every whole playbook harms:
  49.6 vs 43.8 (9B WebShop), 69.8 vs 65.3 (Gemma AppWorld pass/fail). On
  the seven ACE settings not used for tuning the lead is +1.9. On Dynamic
  Cheatsheet, whose lessons are less separable, RBC detects this and serves
  a whole playbook in 7 of 8 settings (66.5 vs 66.9).
- **Long growth (Table 2, up to ten passes, 9 ACE settings, new runs).**
  RBC 3.1 above Best-Full on average, all from curating. Examples: 84.9 vs
  78.4 (27B AppWorld per-test, where whole playbooks fall to 46.6 against
  78.4 with none); 32.3 vs 21.2 (9B). Where RBC serves a curated set it
  uses at most 44% of the tokens and scores 4.7 higher.
- **More runs do not replace curation.** Best-Full needs 5 runs (two more,
  weaker) to reach 66.8 vs RBC's 67.2 with 3, and serves 27% more tokens.
  At matched growth and practice cost, RBC with 2 runs vs Best-Full with 3
  is 65.7 vs 66.1, i.e. level.
- **Judges with an advantage still lose (Tables 7–8).** Tags* 46.1 vs
  RBC-cut 59.7 (9 settings); Top-k* 51.8 vs 59.7; DC_self 33.9 vs 48.3;
  DC_outcome 26.6 vs 37.4. Telling an LLM curator the outcome is not
  enough. Per setting, Tags* beats RBC-cut on 9B AppWorld pass/fail (41.3
  vs 22.2), Gemma ALFWorld (86.4 vs 83.1) and Gemma AppWorld per-test
  (73.8 vs 71.2). Tags* had been tuned on the evaluation tasks.
- **Estimator matters.** DG-Mem's subset-Shapley estimates from the same
  episodes give a worse curated set in 3 of 4 27B settings. Replacing task
  success with an LLM goal-check judge failed on 27B and 9B AppWorld
  pass/fail for every judge prompt tried.
- **The cap matters.** Serving every positive block uncapped: 63.8 vs 65.2
  at 40%. The loss is on Gemma ALFWorld and 9B AppWorld per-test, from
  small positive effects that do not add.
- **Pooling runs loses (Table 11).** Pooling the 3 runs' lessons into 8K
  blocks and curating once scores below Best-Full (41.1 vs 45.2 on four
  development settings), because a playbook's value partly lies in its
  lessons together.
- **One practice round is enough (Table 17).** Out-of-sample gain plateaus
  from 3/4 of a round (10.2 → 11.0 → 10.8 for 1/2, 3/4, 1 round), and a
  variance model predicts +0.8 from a round twice as large, +1.5 from 4×
  and +2.8 from perfect measurement.
- **Practice-to-evaluation transfer (§6.5, E.6).** Practice success ranks
  the 7 candidates much as evaluation success does (median Spearman 0.89).
  RBC picks the evaluation-best candidate in 6 of 11 ACE settings and is
  within half a point in 7. Misses concentrate on AppWorld, where practice
  is the dev split and evaluation is test_normal. Choosing on held-out
  evaluation halves (50 random halvings) adds 2.2 points on DC and 0.4 on
  ACE, and every conclusion holds under that check.
- **Robustness (Table 15, development settings).** Default 47.4; m = 4
  45.9; m = 16 48.0; K = 4 or 5 48.1; B = 20% or 60% 46.1. Best-Full is
  45.2 throughout.
- **Cost (Tables 12–13).** Per growth run, episodes are growth 450 /
  1,288 / 2,500, practice round 684 / 600 / 600, and practice score
  171 / 150 / 140 per candidate (AppWorld / WebShop / ALFWorld). Totals,
  relative to RBC-cut: RBC at K = 3 is 4.1× / 3.6× / 3.3×; Best-Full at
  K = 3 is 1.8× / 2.4× / 2.6×. By my arithmetic, RBC at K = 3 spends about
  3,250 / 2,850 / 2,780 practice episodes per setting. ACE curation tokens:
  RBC 3 runs 1,162M vs Best-Full 3 runs 814M (+43%) vs whole 227M. Serving
  tokens per task: RBC 138k vs Best-Full 166k vs whole 212k.

## Critique / open questions

- **Against the digest framing: BELOW (mildly).** The three numbers are
  exact. But "Average success 66.9 vs 55.5 whole, 54.1 none, 53.8
  agent-picked" omits the one baseline that matters. Best-Full, which
  does everything RBC does except curate, scores 66.4. The randomized
  block experiment, the paper's named contribution, adds 0.5 points on
  average (+1.1 ACE, −0.4 DC), with no significance test. Its real
  advantages are narrower than the headline: fewer served tokens on ACE,
  +3.1 under long growth, and a smaller worst case on ACE (0.8 vs 5.4).
  The digest's three checks:
  1. **69.6 is the single worst of 33 ACE growth runs** (11 settings × 3).
     It comes from a setting where all three runs harm (Table 5: 31.2,
     17.7, 8.7 vs 78.4), so it does not illustrate "same procedure helps
     or harms". That swing is 27B AppWorld per-test (82.5 vs 30.0). By my
     count of Table 5, 14 of 33 ACE runs harm by more than 5 points. The
     second-worst is 54.7 below (Gemma AppWorld pass/fail run 1). Dynamic
     Cheatsheet's worst is 14.0.
  2. **Cost:** one practice round of 600–684 episodes per growth run, plus
     140–171 episodes per candidate × 7, on 57–300 practice tasks. That is
     roughly 2,800–3,250 practice episodes per setting and 3.3–4.1× the
     episodes of curating one run, against 1.8–2.6× for Best-Full.
  3. **Selection is separate from the reported test set.** The final
     choice uses new episodes on the *practice* tasks, never evaluation
     tasks. It does reuse the practice tasks the block estimates came from
     (new episodes, same tasks), so the selection score carries some
     winner's curse (see [[literature/papers/hu2026winners]]). The
     evaluation numbers are unaffected.
- **No uncertainty on the headline.** Table 1 has no CIs, standard errors
  or paired tests. The stated re-evaluation noise is up to 5.0 points per
  playbook, and the RBC vs Best-Full gap is 0.5. "Leads in 5 of 6
  settings where choices differ" is the only per-setting evidence for
  curation over choosing on the main runs. Long growth (Table 2) is
  stronger, but it is one table of 9 settings with three runs each.
- **The headline mean is carried by catastrophes.** The 11-point lift over
  Whole is largely the no-playbook fallback rescuing ACE AppWorld
  pass/fail settings (Whole 19.2 → RBC 77.6 on 27B). Any rule that can
  choose "none" on a practice split gets most of this. That is the
  paper's own Best-Full result, and the design lesson is real but not
  new (RSEA).
- **"Agent-picked" is not a strawman, but it is ACE-specific.** ACE's
  Reflector here sees the whole playbook and is told untagged lessons are
  neutral, because the actors do not cite lessons (Appendix G, a
  deviation from ACE's design). Tags given without citation may be
  noisier than in native ACE. The 0.21 correlation is still a clean
  measurement for this setup. On 9B the harmful lessons were mostly
  tagged *harmful* (Table 4), so tags do not always point the wrong way;
  the failure is the tag-ranked cut, not the tags' sign everywhere.
- **The mechanism is checker mismatch.** All three AppWorld pass/fail
  harmful families tell the agent to pass an answer to `complete_task`.
  Under pass/fail feedback the Reflector never sees the checker and
  invents a validator; under per-test feedback it writes no such rule.
  That is a lesson about unverifiable feedback producing confident wrong
  rules, which bears on [[concepts/programmable-evaluator-oracle]] as much
  as on curation.
- **Scope.** Open models of 9–31B with thinking off (one control with
  thinking on, on 9B WebShop). Three household/web/API benchmarks with
  binary or near-binary success. Playbooks are frozen once grown; RBC is
  not tested on memory that grows while served. It needs a practice split
  with an observable outcome. An LLM judge as the outcome failed. For a
  research agent whose tasks are one-off and whose outcome is a noisy
  validation metric, a practice split of hundreds of comparable episodes
  does not exist.
- **Blocks, not lessons.** The unit of admission is a Ward cluster of
  10–21 lessons (Table 4). A harmful rule can only be dropped together
  with its cluster-mates. m barely matters (45.9–48.0), but no result
  shows lesson-level precision.
- **Open:** can a cheaper signal (e.g. per-lesson citation plus outcome)
  approach randomized inclusion? The paper's DC_outcome and LLM-judge
  tests say not easily. And does the advantage survive when practice and
  deployment tasks drift apart, which is the AppWorld miss pattern?

## Trust signals

- **Credibility:** 3.
  - For: a clean causal design (independent randomized inclusion, task
    intercepts) with short, correct proofs. Constants were fixed on four
    development settings with held-out pools, and a pre-stated tuning rule
    moved nothing. Evaluation tasks never enter selection. Baselines are
    deliberately advantaged (Tags*, Top-k* tuned on evaluation tasks; an
    outcome-informed LLM curator). The paper reports where it loses
    (Dynamic Cheatsheet, the AppWorld misses, the uncapped variant) and
    gives the full per-run tables (Table 5), a cost accounting in episodes
    and tokens, and a component decomposition. RBC uses no prompts of its
    own; the actor prompts are published in the appendix.
  - Against: not peer reviewed. The code link is an anonymous.4open.science
    review mirror, not verified and not permanent. There are no
    confidence intervals or significance tests anywhere in the main
    results, and the curation-specific margin (0.5) sits well inside the
    stated 5-point re-evaluation noise. Three growth runs per setting.
    Mid-size open models only. ACE's Reflector was modified (no lesson
    citation).
  - Not 4 because the main comparison that tests the named method (RBC vs
    Best-Full) is small and untested statistically, and the code is not
    yet public. Not 2 because the design and disclosure are well above
    typical memory-system preprints.

## Follow-up

- **Relevance:** 4. It is the first source in this graph that measures,
  with a randomized design, whether a self-written memory's own
  usefulness labels track its causal effect on task success, and finds
  they do not (rank correlation 0.21; harmful blocks carry many helpful
  tags). That is material new evidence for
  [[concepts/skill-library-lifecycle]]'s admission/retention gates and
  [[concepts/verified-memory-writes]]'s "the writer is the judge" problem.
  It also supplies a cheaper-than-leave-one-out estimator (all blocks from
  one round, 4.5× fewer episodes than LOO at m = 8) that improves on
  hu2026skillbrew's per-skill LOO and shang2026hypothesis's single paired
  comparison. Not 5: the practice-split requirement does not transfer
  cleanly to one-off research tasks, and the headline is mostly carried by
  choose-with-fallback, which was already known.
- **Design lessons to carry**, roughly in order of evidence strength:
  1. Keep "serve nothing" as a live, scored candidate. This was the
     largest single effect.
  2. Do not rank lessons by the writer's own usefulness tags.
  3. Value lessons by their effect on outcome, in groups large enough to
     move success measurably.
  4. Cap what is served even among measured-positive lessons.
- **Candidates** (cited, not in the graph):
  - Nguyen, Nguyen & Vuong 2026, "Recursive self-evolving agents via
    held-out selection" (RSEA, arXiv 2606.28374). The source of the
    Best-Full rule, and nearly as good as RBC.
  - Guo et al. 2026, "Self-authored verification is unreliable in
    heuristic self-improving agents" (arXiv 2607.24300).
  - Wang et al. 2026, "Dual-grained agent memory and Shapley context
    attribution" (DG-Mem, arXiv 2608.23268).
  - Zhao et al. 2026, "Coalition-aware skill reliability for
    self-evolving agents" (arXiv 2608.22610).
  - Li et al. 2026, "What is a skill worth? Structure-aware Shapley
    valuation of agent skills" (SkillSV, arXiv 2608.04562).
  - Zhang et al. 2026, ACE (ICLR 2026), and Suzgun et al. 2026, Dynamic
    Cheatsheet (EACL 2026). These are the base systems; neither has its
    own note in this graph.
