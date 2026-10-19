---
kind: paper
title: "SciΠvot: Benchmarking Principal-Investigator Judgment at Pivotal Moments of Scientific AutoResearch Agents"
authors: ["Youran Sun", "Xingyu Ren", "Chugang Yi", "Kejia Zhang", "Di Wu", "Yingkang Cao", "Allen Zhou", "Jianping Ye", "Suying Liu", "Jiaxuan Guo", "Jianda Du", "Weicheng Ye", "Kecheng Liu", "Hefan Zhang", "Haixin Wang", "Jie Xiao", "Eric Yang", "Haizhao Yang"]
institutions: ["University of Maryland, College Park", "The Chinese University of Hong Kong", "Gradient Network", "Stanford University", "Dartmouth College", "University of California, Los Angeles"]
year: 2026
venue: "arXiv 2610.21082 (cs.AI), v1 2026-10-15, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.21082"
code_url: "https://github.com/AutoResearch-Factory/SciPIvot"
citations: null
source: "raw/papers/sun2026scipivot.pdf"
added: "2026-10-19"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
tags: ["pi-judgment", "pivot-decision", "redirect", "human-correction", "preference-pairs", "on-policy-anchoring", "project-record", "state-notes", "reconsideration", "re-trapping", "self-correction", "test-time-effort", "model-in-the-loop-selection", "challenge-set", "position-bias", "autoresearch", "benchmark"]
---

# SciΠvot: Benchmarking Principal-Investigator Judgment at Pivotal Moments of Scientific AutoResearch Agents

## TL;DR

The authors ran LLM "AutoResearch" agents on 26 of their own PhD-level
projects for about six months. Senior researchers reviewed the projects
several times a day and corrected the agents' plans. From the finished
histories they built 248 two-way choices. Each case has the project record
frozen at the commit before a human intervention, the agent's original plan
(A) and the human-corrected plan (B). Both plans were rewritten by Grok 4.6
into the same `Goal:`/`Plan:` format. A model reads the record (idea.md,
proposal.md, the agent-written STATE.md, code and logs, but no git history)
and picks one. Picking B is scored correct; chance is 50%, with each case
scored as the mean over both option orders.

Three results matter for this graph:

1. **At medium effort with the record, four of five frontier models pick the
   agent's plan more often than the correction.** Their accuracies are
   29.2–38.9%, and the upper ends of both their case and project-cluster CIs
   are below 50%. GPT-6 Astra reaches 59.5%. The five-model mean is 39.2%.
2. **The record makes every model worse.** The five-model mean is 55.8%
   with only the two plans and 39.2% with the record. The paired drop is
   −11.7 to −26.6 points for four models (CIs exclude zero) and −4.2
   [−10.9, +2.4] for Astra.
3. **Reconsidering without feedback flips answers both ways.** The first
   review repaired 422 answers and broke 302 (of 2,480 transitions). The
   second repaired 264 and broke 305. The mean went 39.2 → 44.0 → 42.3%.

The cases were **selected with the evaluated models in the loop**: a case was
kept only if those five models' 120 responses split between the two plans.
The absolute accuracies, the "below chance" result and the gap to humans are
therefore properties of a challenge set, not base rates.

## Claims

- **Scientific judgment, not execution, is the bottleneck.** "Keeping
  experiments running proved easier than choosing experiments that advanced
  the scientific question." Their name for this decision-making is PI
  Judgment.
- **Models prefer the original plan for structural reasons.** "The original
  LLM plan is on-policy text, while the human-corrected plan is off-policy. A
  systematic preference for the original plan is the structural signature of
  this asymmetry." This is stated as an explanation, not tested.
- **Project context hurts on this set.** "Every model nevertheless scores
  higher without the project record at medium effort". The authors attribute
  this to the agent's own explanations in the record, including mistaken
  interpretations, and to long-context degradation.
- **More thinking can return to the same mistake ("re-trapping").** Higher
  effort and reconsideration help some models and hurt others.
- **Adding rules is not the fix they propose.** Section 2.2 argues that
  growing instruction sets are followed less reliably (citing Jaroslawicz et
  al. 2025), and that "an instruction to check a measurement does not
  specify which assumption is wrong". Their proposed route is training: the
  pairs fit DPO, and a preference scorer could reward research actions. This
  is future work.
- **Scope limits stated by the authors.** The benchmark "measures recognition
  of corrective scientific decisions" from projects "selected using the
  procedure in Section 3". The cases "do not estimate how often these
  failures occur across all scientific research activity." Wet-lab work is
  excluded.

## Methods

- **Source projects.** There are 26 projects, grouped by primary field:
  applied math / scientific computing (8 projects, 85 cases), robotics (4,
  58), AI/ML (9, 59), biological imaging (1, 21), mechanics/materials (2, 20)
  and earth sciences (2, 5). Projects contribute 1 to 43 cases;
  `sequential-pinn` alone has 43. The work is computational only: theory,
  simulation and analysis of existing data on CPUs and GPUs.
- **Case construction.** The writer, Grok 4.6, reads the full history. It
  recovers the original plan, picks the commit before the intervention, and
  reconstructs the corrected plan from the intervention and the
  investigation that followed. It is told to exclude administrative actions,
  unrecoverable states and corrections not linkable to an error, and to
  match detail and length between the two options. The evaluator never sees
  the writer's explanations. The ground truth is the senior researcher's
  correction plus the later evidence for it. The original plan was never run
  to compare.
- **Case selection.** 120 responses per case were collected (5 models × 12
  conditions × 2 orders). Cases were kept "whose model responses vary
  across the two plans", with "a minimum share of responses on both plans".
  The threshold is not given. Four one-sided cases were kept after a
  documentation check. The size of the candidate pool before selection is
  not reported. The authors cite Dynabench (Kiela et al. 2021) as precedent
  and call the scores "diagnostic". Humans played no part in selection.
- **Judgment labels.** Each case has at least one judgment label, and
  categories overlap: planning 123, focus 122, anomaly awareness 88,
  adapting 65, evidence sufficiency 47, repetition 4. Adapting means the
  agent "keeps the same hypothesis or search strategy even when earlier
  experiments have shown what should change". Coding uses only the original
  plan and the correction, never the evaluated responses. 243 cases have a
  primary error label.
- **Models.** GPT-6 Astra, Claude Opus 5, Grok 4.6, and GPT-5.6 Sol and
  Terra. Opus and Grok run through one shared Claude-client harness with
  file-read/search tools only. The GPT models run through Codex with
  read-only shell. Network access is off.
- **Conditions.** There are 12: {project, options-only} × {low, medium,
  high, xhigh, max} effort, plus two reconsideration turns after the
  medium-effort project answer ("Please reconsider … You may keep or change
  your previous answer", no correctness feedback). Effort labels are not
  matched token budgets.
- **Scoring.** Each case is scored as the mean over two option orders, with
  cases weighted equally. Unparsed responses count as wrong (8 of 29,760,
  none in the medium-effort primary cells). The paper reports 10,000-sample
  case bootstraps (seed 20261004), project-cluster bootstraps, and
  parent-child pairing for reconsideration.
- **Human baseline.** 13 PhD students and postdocs who had not seen or
  supervised the source projects. They used an interface with the record
  and both plans, with option order randomized. An agent assistant was
  available (GPT-5.6 Terra by default; Astra and Claude optional), and the
  human made the final choice. Every case has one recorded human response.
- **Writer-bias pilot.** On 13 cases, Astra, Opus and Grok each wrote both
  options, and the three models answered every version.

## Results

- **Main table (medium effort, Table 4/5).**

  | Model | Project | Options only | Paired diff [95% case CI] | Best observed project (setting) |
  |---|---|---|---|---|
  | Astra | 59.5 | 63.7 | −4.2 [−10.9, +2.4] | 63.5 (xhigh) |
  | Opus 5 | 38.9 | 58.9 | −20.0 [−26.8, −13.3] | 46.8 (max) |
  | Grok 4.6 | 33.1 | 54.0 | −21.0 [−27.8, −14.3] | 38.1 (high) |
  | Sol | 35.1 | 46.8 | −11.7 [−18.5, −4.8] | 37.7 (after review 2) |
  | Terra | 29.2 | 55.8 | −26.6 [−32.9, −20.0] | 52.0 (after review 1) |
  | Humans | 73.4 (182/248) | — | — | agent assistant available |

  "Best observed" is the maximum over seven conditions per model, so 63.5%
  is a selected cell. Project-cluster CIs are similar (Table 6). Astra's
  difference CI is [−11.8, +3.6].
- **Unanimity.** On 49 cases all five models chose the agent's plan in both
  orders. On 16 they all chose the correction.
- **By category (Fig. 3).** Astra scores 72.2% on anomaly-awareness cases,
  56.1% on planning and 54.9% on focus. The other models sit around or below
  50% on most categories.
- **Scale decisions go both ways.** In case 21 the correction was to pilot
  at 20 items per cell before 200 (3/10 model responses chose it with the
  record). In case 13 it was to run one seed for the full 8,000 iterations
  instead of three short gated runs (4/10). A fixed "go bigger" or "go
  smaller" bias does not explain the errors.
- **Effort (Fig. 5a, Table 9).** From low to max, four models improve and
  Sol falls from 36.1% to 29.2%. Options-only accuracy is nearly flat across
  effort: within about 3 points for each model, except Sol's medium-effort
  dip (46.8 against 53.8–55.8 at the other settings).
- **Reconsideration (Fig. 5b–c, Table 10).** The first review nets +120
  answers (+4.8 pp on the mean). It is driven mostly by Terra (29.2 → 52.0).
  The second review relative to the first is −1.7 [−3.5, +0.2]. Opus falls
  −5.4 [−8.7, −2.2] and Terra −9.3; Sol rises +5.6. In the worked Transformer
  case, Opus picks the wrong plan, then the right one, then the wrong one
  again. Its final reason is a premise the frozen code contradicts.
- **Order effects (Table 11).** These are large. Opus options-only scores
  44.0% with the correction first and 73.8% with it second. Grok with the
  record scores 41.9 / 24.2. Averaging over both orders removes the bias
  from the mean, but a single-order deployment would see these swings.
- **Writer pilot (Table 12, 13 cases).** Grok, which wrote the released set,
  is never the top scorer on its own cases (23.1% with the record on
  Grok-written cases, against Opus 34.6%). Writer identity shifts absolute
  difficulty: Astra scores 76.9% on Opus-written options-only cases.
- **The two worked cases (Fig. 2, App. B).**
  - *Check the measurement before scaling up.* A 0.8B model scored 0/480 on
    three LiveCodeBench groups. STATE.md claimed that earlier checks had
    ruled out an evaluation fault, and the agent proposed retesting on 2B.
    The correction was to keep 0.8B and fix the scorer, which had skipped
    answer extraction. Regrading recovered 70 correct answers.
  - *Question the fit before refining the search.* A cloth-simulator fit had
    four of six parameters fixed at zero, a restriction the agent had carried
    over from its own earlier runs. All ten minima sat at the corner of the
    search box. The agent proposed searching the remaining two parameters
    more finely. The correction reopened the six-parameter fit, keeping the
    two-parameter slice as a comparison. With the record, 1 of 10 responses
    chose the correction. With options only, 5 of 10 did. Opus called the
    restriction a "standing, human-authorized restriction".

## Critique / open questions

- **Against the digest framing:** BELOW. Every number in the digest entry
  is in the paper, but the entry drops the word the abstract keeps
  repeating: "selected". It also turns "does not consistently improve" into
  "doesn't help". Specifically:
  - *What "selected cases" means.* Cases were filtered so that the five
    evaluated models' responses split between the plans, across all 12
    conditions. This is adversarial filtering against those same models. It
    removes cases every model gets right, and it removes almost every case
    every model gets wrong. So "four of five below chance" describes a set
    where model disagreement was a requirement for entry. The direction is
    not forced by the filter, since it is symmetric in A/B. The level,
    though, says nothing about how often agents reject good corrections in
    general. The authors say so themselves.
  - *Chance* is 50%, a binary choice averaged over both orders. Because of
    the order effects (up to 30 points for Opus options-only), single-order
    numbers would be noisier than the averaged ones.
  - *Humans vs models is not like for like.* Humans were not selected
    against, had an LLM assistant, and gave one response per case with no
    CI. The 73.4 vs 63.5 gap compares an unfiltered human score with a
    best-of-seven model cell on a model-filtered set.
  - *"Reconsideration doesn't help"* overstates. The first review nets +4.8
    pp on the mean, and +22.8 for Terra. What fails is the *second* review
    and *consistency*: it flips about as many right answers to wrong as the
    reverse.
  - The abstract itself hedges correctly ("on the selected cases"). The
    over-read comes from the digest paraphrase, not from the paper.
- **The project-vs-options drop is a bundle.** The project condition adds
  observations, the agent's interpretations (STATE.md), length, file
  retrieval and a provider-specific tool interface, all at once. The paper
  cannot say which component hurts. Its mechanism, inherited explanations,
  is shown in two worked cases, not measured. Selection retained cases
  where conditions disagree, so it can also inflate the *size* of the
  paired gap, though not force its sign. Astra, the strongest model, shows
  no significant drop.
- **"On-policy anchoring" is an untested explanation.** The writer rewrote
  both plans in one style. The original plan came from an unspecified
  agentic system, not from each evaluated model, so "on-policy" is loose.
  No arm swaps authorship or style to test it.
- **Ground truth is the authors' own program's corrections.** The paper
  does not name the senior researchers. It describes the projects as "our
  AutoResearch program", so the labels come from inside the group. The
  corrected plan is "better" because of the later history that supports
  it. The rejected plan was never executed, and corrections not linkable to
  an error were excluded. That is a reasonable standard, but it is one
  group's judgment with no inter-rater check on the reference labels.
- **The writer is one of the evaluated models.** The 13-case pilot (26
  responses per cell) argues against self-favouring, but it is small and
  shows that writer identity shifts difficulty by tens of points.
- **Reproducibility of the project condition is incomplete.** Released
  (verified on GitHub on 2026-10-19): `data/questions.json` with 248 cases,
  options and answers, `snapshots.jsonl`, the ask, reconsider and
  case-writing prompts, aggregate result CSVs including reconsideration
  transitions, and scoring and run scripts. Per-response logs are not
  released. The paper says the frozen project records were "too large to
  upload" and will be hosted later; the repo has a `fetch_records.py`, but
  record availability was not checked. Until the records are available,
  only the options-only condition can be rerun.
- **What it does not show.**
  - It does not test whether agents *generate* the right pivot. Both plans
    are handed to the model.
  - No outcome for a model-chosen pivot is measured.
  - There is no comparison of redirect against halt, and no comparison of a
    forced pivot against a self-judged one.
  - There is no ML-loop cycle structure, keep rule or stall counter.
  - It does not show that the corrected plan beat the original in
    expectation, only that it was the human's choice and was later
    supported.
- **Open for this graph:** does the record effect come from the agent's
  *interpretive* notes (STATE.md narrative) or from *observations* (logs,
  metrics)? [[literature/papers/wang2026language]] finds that
  in-environment experiment records *raise* Opus 5's ranking of unseen
  interventions (Spearman 0.506 → 0.774). This paper finds that a project
  record *lowers* Opus 5's choice between two plans by 20 points. A
  notes-stripped record arm would separate the two. Neither paper runs it.

## Trust signals

- **Credibility:** 3.
  - For: there is a code and data release. The question set, prompts
    (including the case-writing prompt), aggregate results and scripts were
    verified to exist on 2026-10-19. The statistics are careful: paired case
    bootstraps and project-cluster bootstraps, both option orders, and
    parent-child pairing for reconsideration, with unparsed responses
    disclosed. A writer-bias pilot is reported. The limitations section
    states the selection caveat plainly. The cases come from real,
    months-long supervised projects, which is rare. The institutions are
    UMD, CUHK, Stanford, Dartmouth and UCLA, plus a company (Gradient
    Network).
  - Against: it is not peer reviewed (v1, four days old). Selection uses
    the evaluated models with an unreported threshold and an unreported
    pre-selection pool. The reference labels come from one group's own
    corrections, with no inter-rater check. The writer is an evaluated
    model. The human baseline is 13 people with an assistant, one response
    per case and no CI. Project records and response logs are not yet
    released, so the headline project condition cannot be reproduced. The
    model names (GPT-6 Astra, GPT-5.6 Sol/Terra) are provider-internal
    labels, so the results describe a snapshot.
  - Not 4 because selection and labelling both depend on the authors and the
    evaluated models, and the main condition is not yet reproducible. Not 2
    because the release, the paired statistics and the candour about
    selection are well above the median preprint here.

## Follow-up

- **Relevance:** 4. This is the first measurement in this graph of the
  decision a redirect rule exists to override: at a real pivot point, does
  the agent recognize the human's change of direction over its own
  continuation? On a hard, model-filtered set, it mostly does not. The
  agent's own record makes it worse, and reconsidering without feedback
  flips answers in both directions. That is material new evidence for
  [[concepts/budget-as-ceiling]]'s controller-not-prompt reading
  (yan2026traceml, cai2026forkandflush, zhang2026judged), extended from
  "agents don't pivot" to "agents don't pick the pivot when shown it". It
  is not a 5 because it measures recognition on selected cases. It does
  not compare redirect with halt and does not measure what a forced pivot
  yields.
- **Bearing on the `iterate-redirect-before-halt` proposal (2026-10-18):
  supports forcing, and warns about content.**
  - *Supports* making the pivot a controller constraint rather than a
    self-judgment. Shown both options, models favoured continuing. Asking
    them to reconsider is not a reliable repair (second review −1.7 mean;
    Opus −5.4).
  - *The proposal's "reopen" clause has a direct instance.* In the cloth
    case the correction reopened a restriction the agent had inherited from
    its own earlier runs and later treated as a rule. With the record, 1/10
    responses chose it.
  - *Warns about axis choice.* In the LiveCodeBench case the *wrong* plan
    was the backbone change (0.8B → 2B) and the right one was a validation
    check. A forced axis change that the agent fills in will sometimes pick
    exactly that kind of change. The proposal lists validation as an axis;
    this case argues for preferring "audit the measurement" when results
    are degenerate (all zeros, minima pinned at a boundary).
  - *Warns about inputs.* If the pivot cycle chooses its new axis by
    reading the chain's own narrative log, this paper predicts anchoring.
    The candidate list should be built from measurements and code, and
    inherited claims ("evaluator ruled out", "fixed by rule") should be
    treated as unverified. This is our inference, not tested here.
- **Candidates** (cited, not in the graph):
  - Sun et al. 2026, "Agon: An autonomous large-scale omnidisciplinary
    research system built on prompt economy" (arXiv 2606.24177). Same first
    author; probably the system that produced these projects.
  - Ríos-García et al. 2026, "AI scientists produce results without
    reasoning scientifically" (arXiv 2604.18805), Corral.
  - Tian et al. 2026, "ForeSci: Evaluating LLM agents for forward-looking
    AI research judgment" (arXiv 2606.00644).
  - Jaroslawicz et al. 2025, "How many instructions can LLMs follow at
    once?" (arXiv 2507.11538). A source for the instruction-load argument
    against prompt-level rules.
  - Already in the graph: [[literature/papers/ho2026soundnessbench]] and
    [[literature/papers/yang2026truthinsightbench]].
