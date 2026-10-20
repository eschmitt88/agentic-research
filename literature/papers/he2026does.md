---
kind: paper
title: "Does Failure Feedback Direct Agent Self-Modification?"
authors: ["Rong He"]
institutions: []    # none stated in the PDF or on the arXiv listing; single, unaffiliated author
year: 2026
venue: "arXiv 2610.14662 (cs.SE, cs.MA), v1 2026-10-10, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.14662"
code_url: null     # code + data ship as arXiv ancillary files (listed on the abs page), no repo URL
citations: null
source: "raw/papers/he2026does.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/evolutionary-search-grain]]"
tags: ["self-modification", "directedness", "failure-feedback", "darwin-godel-machine", "dgm", "gepa", "reflective-prompt-optimization", "randomized-target", "e-value", "selection-vs-variation", "viability", "collateral-damage", "output-protocol", "edit-localization", "regression-screening", "planted-rule", "rule-persistence", "preregistered", "qwen3-4b", "phi-4-mini", "big-bench-hard"]
---

# Does Failure Feedback Direct Agent Self-Modification?

## TL;DR

A self-improving agent revises itself after seeing one of its failures. The
paper asks whether the revision actually repairs *that* failure more often
than the parent's other failures ("directedness"). Because DGM draws the
failure shown at random from the parent's failures, directedness can be
tested without a model of task difficulty, using an e-value that stays valid
in an adaptive population.

Three results matter for this graph:

1. **In the public Darwin Gödel Machine logs, edits were not directed at the
   failure shown.** Over 145 eligible modifications in six runs, 12 repaired
   their target against 16.4 expected under the null (8.3% vs 11.6% for the
   parent's other failures; e-value 0.15, far below the 20 needed at 5%).
   A zero-mean variation model with DGM's selection rules reproduces all
   three SWE-bench archive curves (median best 0.51 vs 0.50 observed for
   DGM). That shows selection over undirected edits is *sufficient* for the
   trajectories, not that it produced them.
2. **When the editor sees the failed question and its answer, revisions are
   directed, but directed repair does not imply a score gain.** On a clean
   playbook (Qwen3-4B, 180 BBH questions), instance children repaired the
   question shown 51.9% vs 27.4% for other failures, yet lost 19.0 points on
   average. Most of the loss came from rewriting the response-length/format
   line. Gains appeared only when planted rules gave many failures one
   editable cause.
3. **In a pre-specified second study with a held-out split, relevant
   feedback raised target repair for one of two models (+20.0 points for
   Qwen3-4B, +6.7 n.s. for Phi-4-mini), and held-out gains came almost only
   from parents with a planted cause.** On the clean parent, no single-step
   method, eight-step loop or GEPA raised held-out accuracy on average
   (+0.4, +0.6, −0.3 points).

## Claims

- "Gains after selection do not establish target-specific repair or
  generalization." A higher retained score is compatible with directed
  edits, generally useful edits, or selection keeping edits that happen to
  work.
- Three quantities per revision should be reported separately:
  directedness (target vs the parent's other failures), category-level
  spread (other failures in the target's category vs elsewhere) and net
  score change (failures repaired minus correct answers broken, over N).
  Table 1 says what each does *not* establish.
- DGM: "no detectable target-specific advantage" at task, language or
  repository level; small mean edit-level score changes; and a higher
  *viability* when the agent edits itself than when a fixed initial agent
  edits every descendant.
- The DGM-vs-fixed-editor gap that the DGM paper credits to compounding
  self-improvement is, in these logs, a viability difference: better agents
  do not make better edits.
- Held-out gains concentrate where failures have "identifiable, repairable
  failure mechanisms" (planted rules, planted helper bugs, systematic
  tool-interface errors). The paper says explicitly that it does not
  establish failure structure as the cause.
- Practice recommendations (Section 7.3): randomize the failure shown;
  keep the answer protocol outside what the editor can change; report
  candidate quality before and after regression screening, "since
  screening improves the returned agent by discarding bad candidates
  rather than by producing better ones".

## Methods

- **Directedness test (Section 2.3, App. A).** Per modification m,
  Z_m = I_m(y_m − q_m), where y_m is target repair and q_m the repair rate
  over the analysis set (pool ∩ evaluated tasks). Under the null, Z_m is a
  martingale difference whatever the adaptive history. The primary test is
  an e-value averaging ∏(1 + λZ_m) over 20 values of λ in (0, 1); Ville's
  inequality gives validity at any sample size and stopping time.
  Auxiliary: martingale z, Mantel–Haenszel OR, parent-resampled intervals.
- **DGM data.** Six public runs: DGM, fixed editor ("w/o self-improve") and
  no-archive ("w/o open-ended exploration"), each on SWE-bench Verified and
  Polyglot. o1 writes a proposal from the parent's log on the target
  (including hidden-test results), instructed not to be issue-specific; a
  Claude 3.5 Sonnet coding agent implements it and never sees the failure.
  A modification is eligible if its entry was a task, the child works, and
  the target was evaluated.
- **Study 1 (controlled, exploratory).** Qwen3-4B-Instruct-2507 (4-bit, MLX)
  follows and edits its own playbook. 180 BBH questions from six
  categories. Clean playbook (79.4%, 37 failures) vs planted-rule playbook
  (53.3%, 84 failures) with three erroneous rules: round to the nearest
  ten, write counts in words, and an inert navigation rule. Evidence:
  one failure with the response and correct answer ("instance"), a named
  weak category, or none. 27/6/3 viable clean children, 28/6/4 planted.
  Exact Poisson–binomial test. Every child is scored on all 180 questions,
  which are also the diagnosis set.
- **Study 2 (pre-specified plan with SHA-256-dated versions, not an
  independent registry).** Qwen3-4B and Phi-4-mini (bf16, vLLM).
  - Setting A: BBH splits D 180 (diagnosis) / V 180 (selection) / T 240
    (held out); seven-module playbook; the output protocol is appended by
    the harness and is not editable; parents clean and clean + one planted
    rounding rule.
  - Setting B: tool-using agent writing Python over an editable helper
    library for a synthetic shop API; three planted helper bugs.
  - Factorial: feedback (target's own failure / matched same-category
    failure / other-category failure) × scope (whole rewrite / one module
    chosen with target knowledge, an oracle localization in Setting B).
    40/41 combinations per parent in A, 60 in B.
  - Hypotheses H1 (own failure > matched failure, for target repair),
    H2 (one module breaks less), H3 (feedback × scope on T net change),
    Holm-adjusted.
  - Method comparison: editor-chosen module, random non-target module,
    regression screening on V (up to 3 attempts). Eight-step loops (8
    trajectories) and GEPA with a matched 2,880-evaluation budget.
  - Noise floor: ten whitespace-only variants per parent.
- **Amendments.** A1–A8 before the data they concern; A9 after the data,
  fixing a bug that showed some clean-parent combinations the planted
  parent's response (20/40 Qwen, 18/40 Phi). Those were re-run with the
  same seeds; the old records are kept.
- **Compute.** About 22 GPU-hours on rented RTX 4090s for study 2.

## Results

- **DGM task-level directedness (Table 7).**

  | Run | M | Repaired | Expected | P(target) | P(other) |
  |---|---|---|---|---|---|
  | SWE · DGM | 23 | 0 | 2.8 | 0.000 | 0.125 |
  | SWE · fixed editor | 15 | 2 | 1.4 | 0.133 | 0.090 |
  | SWE · no archive | 19 | 1 | 1.2 | 0.053 | 0.061 |
  | Polyglot · DGM | 53 | 6 | 6.9 | 0.113 | 0.132 |
  | Polyglot · fixed editor | 16 | 1 | 1.8 | 0.062 | 0.118 |
  | Polyglot · no archive | 19 | 2 | 2.5 | 0.105 | 0.133 |
  | All | 145 | 12 | 16.4 | 0.083 | 0.116 |

  Difference −3.3 points (95% CI −8.2 to +2.0), pooled OR 0.69
  (0.31–1.18), z = −1.23. Five of six runs point the wrong way. Power: the
  test rejects with probability 0.72 / 0.89 / 0.96 for a directed boost of
  6 / 8 / 10 points. Group level (language, repository): +6.0 vs +5.4
  expected (p = 0.45) and −1.1 vs −0.1 (p = 0.75).
- **Human-named goals (Table 2).** Among surviving children, every
  empty-patch-goal child reduced the empty-patch rate (8/8, Fisher
  p = 0.03), but over all attempts, keeping randomization intact, the risk
  differences are +10 and +3 points (p = 0.21, 0.36). The context goal
  produced a surviving child in 18% of attempts vs 40% for other entries.
- **Edit-level score changes (SWE-bench).** Means +1.1 (DGM, n = 39),
  −0.7 (fixed editor, n = 24), +0.1 (no archive, n = 25), SDs 7.2 / 6.9 /
  3.3. A typical DGM edit repairs about 7 tasks and breaks about 6 of 58.
  The no-archive spread (3.3) bounds single-evaluation noise; real DGM edit
  spread is about 6.4 points. Parent–offspring slope 0.65. Edit gain does
  not grow with parent quality (slope −0.15, CI −0.35 to +0.02) and falls
  2.1 points per 10 generations (p = 0.03).
- **Viability.** Self-editing yields a working agent in 49% of SWE-bench
  attempts (43% → 55% across run halves) vs 33% for the fixed editor
  (42% → 23%); Polyglot 75% vs 23%. Pooled (n = 322), interaction
  +12.2 log-odds (z = 4.2).
- **Study 1 (Table 4).** Clean: target 0.519 vs other 0.274 (+24.5,
  CI +6 to +43, p = 0.007); category spread +3.6 (p = 0.09). Planted:
  0.607 vs 0.244 (+36.4, p < 10⁻⁴), spread +21.1 (p < 0.001). Net: clean
  instance children −19.0 points (4/27 above parent; repaired 28% of 37
  failures, broke 31% of 143 correct). Named/none children −38.4 / −35.4
  (0/9 above). Planted instance children +5.4 (21/28 above). Of 1,201
  broken clean answers, 42% were a different answer, 30% hit the 512-token
  limit, 16% used another answer format, 12% had no answer line. Prefix
  continuation and relaxed extraction recover about half the clean loss
  (−9.8 points remains).
- **Planted-rule persistence.** No diagnosis of the 18 rule-caused cases
  said a rule was wrong. Rounding: 5 of 6 diagnoses blamed misapplication,
  restricted the rule, and repaired the target. Number words: the rule was
  mentioned in 7 of 12 diagnoses, 6 of which said the agent had *not
  followed* it; none of the 12 neutralized it and two stated it more
  firmly. With the format rules made editable the number-words rule was
  neutralized in 1 of 11 (vs 0 of 11). The inert navigation rule was
  removed by 1 of 28 instance children and by all 4 no-evidence children.
- **Study 2 (Table 6).**

  | | Qwen3-4B (A, n = 80) | Phi-4-mini (A, n = 82) | Tool agent (B, n = 120) |
  |---|---|---|---|
  | H1 target repair, own − matched | +20.0 [+8.5, +31.8], p = 0.003 | +6.7 [−0.6, +14.0], 0.17 | −0.8 [−5.3, +3.6], 1.0 |
  | H2 breakage, whole − one module | +0.6 [+0.1, +1.0], 0.014 | −2.4 [−4.2, −0.6], 0.99 | −4.5 [−5.4, −3.7], 1.0 |
  | H3 feedback × scope on T | +0.4 [−1.1, +1.8], 0.63 | −1.9 [−4.2, +0.4], 0.24 | +0.5 [−1.3, +2.3], 1.0 |

  Whitespace-only parent variants moved T by −0.4 to +1.7 (Qwen) and −1.7
  to +4.2 (Phi), the same size as several reported effects. With the
  output protocol fixed, clean-parent children changed T by −1.0 to −0.2
  points (vs −19.0 in study 1). Planted parent: restricting the edit to the
  arithmetic module repaired 65% of rule-caused T items even with
  other-category feedback, vs 8% for a whole rewrite with the same
  feedback. Tool agent: oracle-localized edits +11.2 on T, repairing 90%
  of targeted-bug T items (26% with other-family feedback); 186 of 357
  viable whole rewrites reproduced the agent exactly.
- **Localization and screening (Qwen).** Localization + screening beat a
  whole rewrite by 2.5 points (CI +1.5 to +3.6), and by 1.8 over a
  screened whole rewrite. The gain came from the planted parent (+7.7 vs
  +3.6); on the clean parent every method stayed in −1.2 to −0.3. For Phi
  no comparison differed from zero; screening cut breakage (14.0% → 9.6%).
- **Loops and GEPA (Qwen, A).** Planted parent: +14.5 (whole-rewrite loop),
  +13.3 (localized loop), +14.4 (GEPA), each by removing the rounding rule.
  Clean parent: +0.4, +0.6, −0.3. Localized − whole loop −0.5 (CI −2.2 to
  +1.1). 8 trajectories per loop, 4 GEPA runs per parent.

## Critique / open questions

- **Against the digest framing:** AT for the summary; the "Why" reads past
  the paper. Every number in the digest summary is in the paper and the
  summary is fair. Three things it leaves out narrow it:
  1. "No target-specific advantage" is "no *detectable*" advantage at
     M = 145, with power 0.72 at a 6-point boost. Effects of a few points
     are not excluded.
  2. The +20-point result is one of two models; Phi-4-mini's +6.7 is not
     significant and the tool agent's H1 is −0.8, which the paper explains
     by shared causes rather than undirectedness.
  3. "Ended at similar held-out accuracy" holds on the planted parent,
     where all three methods just remove one planted rule; on the clean
     parent none of them gained anything.

  The digest's "Why" infers that refinement near the halt point is "close
  to a random walk". The paper does not support that in general. The DGM
  null is for one pipeline in which the proposal is instructed to be
  general and the implementing agent never sees the failure. The
  controlled studies, where the editor sees the failure and its answer,
  find clear directedness. And the zero-mean selection model is a
  sufficiency check calibrated on the same runs, not evidence that edits
  have zero mean.
- **What the paper means for the redirect-before-halt proposal.** It is
  indirect, and mostly a caution about *content*. The paper never tests an
  axis change or a halt. What it does show: a refinement aimed at the last
  failure can be directed and still lose (study 1 clean, −19.0), and on a
  parent with no shared editable cause neither single steps nor eight-step
  loops nor GEPA moved held-out accuracy. That is consistent with the
  proposal's premise that more same-axis refinement near a stall buys
  little. But the gains it did find came from locating one shared
  mechanism (65% vs 8% same-cause repair with the right module). A forced
  pivot that picks an axis without diagnosing a shared cause gets no
  support here. This sits beside [[literature/papers/sun2026scipivot]]'s
  warning that the agent's choice of pivot content is the weak step.
- **Noise band.** Whitespace-only parent variants moved held-out accuracy
  by up to 4.2 points on 240 items (Phi), and DGM's single evaluations
  carry about 3.3 points of spread. Both are of the order of the edit
  effects being judged, which supports the accepted no-improvement band
  and is a cheap instrument for setting it: a no-op perturbation arm
  measures the floor without a seed sweep. This is our inference; the
  paper does not test a band-gated stop.
- **HCE.** The study-1 loss is an instance of HCE clause 4 one step
  removed. The parser reads the "Answer:" line, and the editor could
  rewrite the instruction that produces it, so edits moved the score by
  breaking the protocol, not the task. Fixing the protocol outside the
  editable text cut collateral damage from −19.0 to about −1 point,
  though the two studies differ in several respects at once, so this is
  not isolated. The screened-vs-unscreened recommendation is also an HCE
  point: screening improves what is returned by selection, so reporting
  only the returned agent hides the edit distribution.
- **Shared-substrate correlated error.** In DGM the editor is the agent
  being edited, and that pairing is what raised viability. The paper reads
  this as the editor producing candidates that run, not ones that repair.
  Nothing here measures correlated *errors* between editor and agent.
- **Skill scoping.** The rule-persistence result bears on any loop that
  revises an instruction file from failures. Editors attributed rule-caused
  failures to non-compliance and sometimes restated the rule more firmly.
  An inert rule survived 27 of 28 failure-driven revisions because no
  failure points at it. Failure-reflection is a poor removal mechanism;
  see [[concepts/skill-library-lifecycle]].
- **What it does not show.**
  - Any result at frontier-model scale outside the DGM logs. The controlled
    models are 4B-class, the planted rules are artificial by design, and
    both models ignored planted rules that contradict their knowledge, so
    Setting A has a single planted cause.
  - Whether failure structure or editing strategy drives held-out gains;
    the paper says the shared cause was not manipulated independently.
  - An ML-research loop; tasks are SWE-bench, Polyglot, BBH and a synthetic
    shop API.
- **Internal checks are good but self-administered.** The plan is
  hash-dated but not registered externally, A9 was a post-data bug fix,
  editor prompts for study 2 were not archived, and the qualitative coding
  of 18 diagnoses was done by one coder (the author).
- **Related evidence in the graph.**
  [[literature/papers/hu2026winners]] shows reflective optimizers' selected
  gains inflate under reused selection sets; this paper's zero-mean
  sufficiency result is the population-level counterpart for DGM.
  [[literature/papers/bobadillasuarez2026audit]] finds refinement mostly
  goes idempotent; here 186 of 357 whole rewrites in Setting B reproduced
  the agent exactly. [[literature/papers/srikanth2026recursive]]'s
  recursive claim was already inconclusive; this paper finds no
  edit-quality compounding in DGM either.
  [[literature/papers/lumer2026reflective]] and
  [[literature/papers/ding2026harnessdelta]] attribute reflective-optimizer
  gains to artifacts and transfer gaps; this one adds that much of what
  feedback contributes is *where* to edit.

## Trust signals

- **Credibility:** 3.
  - For: a full rederivation package is in the arXiv ancillary files
    (verified on the abstract page 2026-10-20: ~240 files including
    `verify_claims.py`, `prereg/prereg_v2.md` and its hash log, DGM
    parsing and simulation scripts, every study-1 editor output and
    playbook, study-2 per-question scores). The DGM analyses start from
    public logs. The statistics are careful: a design-based e-value valid
    under adaptive sampling, power reported, parent-level resampling,
    sufficiency framed as sufficiency. Amendments, including the post-data
    bug fix, are disclosed. The limitations section is unusually direct
    and repeatedly declines causal claims the data cannot carry.
  - Against: single author with no stated affiliation; not peer reviewed;
    the plan is self-hashed, not registered; small open models in the
    controlled studies; one coder for the diagnosis coding; several
    effects (H2 for Qwen) sit inside the measured prompt-perturbation
    noise.
  - Not 4 because there is no institutional or peer-review signal and the
    controlled evidence is 4B-scale with artificial planted causes. Not 2
    because the released code and data let every number be re-derived,
    which is more than most lab preprints in this graph offer.

## Follow-up

- **Relevance:** 4. It is the first source in the graph to test, with a
  design-based statistic, whether a self-modification responds to the
  failure that prompted it, and it applies that test to real DGM logs. It
  adds material evidence to three concepts:
  [[concepts/evolutionary-expansion]] (selection over undirected
  variation is sufficient for DGM's curves; the editor's measurable
  contribution is viability), [[concepts/hce-evaluation]] (report target
  repair, spread and net change separately; keep the output protocol out of
  the editable surface; report pre-screening quality), and
  [[concepts/skill-library-lifecycle]] (failure-driven revision rarely
  removes the rule that causes failures). Not 5 because the controlled
  evidence is small-model and synthetic, and no task is an ML-research loop.
- **For `/iterate` step 3.** "Pick the candidate most directly implicated
  by the last result" is the directed-repair move this paper measures. It
  predicts a candidate can fix the implicated failure and still lose net.
  Logging failures repaired and correct cases broken per cycle, not only
  the delta, would make that visible. Not proposed here.
- **For [[concepts/budget-as-ceiling]].** A whitespace/no-op perturbation
  arm is a cheap way to measure the floor under the no-improvement band.
- **Candidates** (cited, not yet in the graph):
  - Zhang et al., Darwin Gödel Machine (arXiv 2505.22954).
  - Wang et al., Huxley–Gödel Machine (arXiv 2510.21614).
  - Melcer et al., "Textual gradients are a flawed metaphor for automatic
    prompt optimization" (arXiv 2512.13598). Wrong feedback works about as
    well as right feedback; the closest prior result.
  - Agrawal et al., GEPA (arXiv 2507.19457).
  - Ding et al. 2026, re-execution of 1,929 DGM archive evaluations (arXiv
    2610.14460), same digest, entry 17. Sizes the DGM evaluation noise this
    paper bounds at about 3.3 points.
