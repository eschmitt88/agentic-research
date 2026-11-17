---
kind: paper
title: "Do Coding Agents Follow AGENTS.md? The Effect of Policy Wording on Instruction Compliance"
authors: ["Rikuto Tsuchida", "Youmei Fan", "Kazumasa Shimari", "Raula Gaikovina Kula", "Kenichi Matsumoto"]
institutions: ["Nara Institute of Science and Technology", "Wakayama University", "Osaka University"]
year: 2026
venue: "arXiv 2611.04367 (cs.SE), v1 2026-11-03; preprint (Letter) submitted to IEICE Transactions on Information and Systems"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.04367"
code_url: "https://doi.org/10.5281/zenodo.23148524"
citations: null
source: "raw/papers/tsuchida2026do.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/constraint-pinning]]"
tags: ["agents-md", "claude-md", "agent-context-file", "instruction-compliance", "policy-wording", "conditional-phrasing", "rationale-clause", "prohibitive-phrasing", "claude-code", "qwen-code", "claude-opus-5", "claude-sonnet-5", "gpt-oss", "qwen3-coder", "ctxbench", "tracked-marker", "friedman-test", "rank-biserial", "short-paper"]
---

# Do Coding Agents Follow AGENTS.md? The Effect of Policy Wording on Instruction Compliance

## TL;DR

A four-page Letter. One mechanically checkable policy (put `# @tracked`
on the line below the `def` of every Python function the agent adds or
modifies) is appended to the real agent context file (ACF) of 86
CTXbench tasks. It is written in four styles: Prescriptive, Explanatory
(Prescriptive plus a reason), Conditional (an if-clause) and
Prohibitive. Each task × style is run three times on four harness–model
pairs: Claude Code with claude-opus-5 and claude-sonnet-5, and qwen-code
with gpt-oss-120b and Qwen3-Coder-Next.

- **Wording moved compliance in three of four pairs** (Friedman p < 0.001).
  The fourth, Qwen3-Coder-Next, stayed under 1.5% in every style.
- **Conditional lost to both Prescriptive and Explanatory in all three
  pairs that moved**, significantly after Holm correction (rank-biserial
  0.29–0.97). On Opus 5: 78.0 / 77.2 vs 64.2%.
- **Adding a reason did not help consistently.** On Opus 5 it changed
  nothing (78.0 vs 77.2). On Sonnet 5 it cost 22 points (71.6 → 49.4).
  On gpt-oss-120b it gained 14 (6.7 → 20.4).

There is one text per style, so style is confounded with the specific
sentence (the authors say so). The "Conditional" text states the *same*
scope as the Prescriptive one. It is not hedged and not narrower. What
differs is a leading if-clause and the loss of "every".

## Claims

- RQ1: writing style affects compliance. On Claude Code / Opus 5,
  Conditional is significantly below Prescriptive and Explanatory, and
  "adding a reason to a direct instruction made no difference".
- RQ2: the Conditional penalty is consistent across the pairs that
  responded, but the best style is not. Prescriptive is best on Sonnet 5,
  and Explanatory on gpt-oss-120b. The spread between styles grows from
  14 points (Opus 5) to 54 (Sonnet 5): "a style chosen for one agent
  cannot be assumed to transfer to another".
- Proposed mechanism (Discussion, untested): a conditional policy makes
  the agent "first judge whether its own action meets the condition, so
  the policy is more easily overlooked".
- "The less capable of the two models was the one whose compliance moved
  more with the writing style" (Sonnet 5 vs Opus 5, same harness).
- Wording matters only once the agent follows the policy at all. On
  qwen-code both open models stay under 21% in every style. Harness and
  model can't be separated.
- Advice: avoid conditional phrasing, and don't assume a style tuned for
  one agent will transfer to another.

## Methods

- **Tasks.** CTXbench (Gloaguen et al. 2026, "Evaluating AGENTS.md"), a
  SWE-bench-like set from repos with a developer-written ACF: 12 repos,
  138 tasks. Kept 86 tasks from 10 repos that have an ACF at the task
  commit. Where both AGENTS.md and CLAUDE.md existed, they kept the larger
  file. It is saved under the filename each harness reads.
- **Insertion.** The policy is appended as the last line of the ACF after
  a blank line. The rest of the file is unchanged, so position and
  surrounding text are identical across styles within a task.
- **The four texts (Table 1), verbatim:**
  - Prescriptive: "Add the comment "# @tracked" on the line directly
    below the def line of every Python function you add or modify."
  - Explanatory: the same sentence, plus ", because our code-audit system
    uses this marker to identify agent-touched functions."
  - Conditional: "If you add or modify a Python function, add the comment
    "# @tracked" on the line directly below its def line."
  - Prohibitive: "Never add or modify a Python function without the
    comment "# @tracked" on the line directly below its def line."

  Styles follow four of the five ACF policy styles in Mohsenimofidi et
  al. (MSR 2026), dropping "Descriptive". The marker policy is taken from
  [[literature/papers/mcmillan2026instruction]] (`// @tracked` in
  TypeScript there).
- **Pairs (Table 2).** Claude Code 2.1.227 × {claude-opus-5,
  claude-sonnet-5}: same harness, model capability varies. qwen-code
  0.21.2 × {gpt-oss-120b, Qwen3-Coder-Next}: same harness, architecture
  and training vary. No cross-over (no Claude model on qwen-code or the
  reverse).
- **Scoring.** A session complies if *every* added or modified Python
  function carries the comment directly below `def`. Sessions with no
  added or modified function are excluded. The per-task rate is the mean
  over the remaining runs (out of three). A task is kept if ≥1 of its
  three runs remains under every style. Each style has 1,032 sessions
  (86 × 3 × 4). Dropped sessions per style: 72 Prescriptive, 75
  Explanatory, 63 Conditional, 74 Prohibitive.
- **Statistics.** Per pair: a Friedman test across the four styles,
  blocked by task. If significant, pairwise Wilcoxon signed-rank tests
  (Pratt ties), Holm-corrected, with matched-pairs rank-biserial as the
  effect size. No confidence intervals on the rates. No model of repo
  clustering.

## Results

Table 3, compliance (%) by style; n = tasks analysed:

| Harness / model | n | Presc. | Expl. | Cond. | Prohib. | χ²(3) | p |
|---|---|---|---|---|---|---|---|
| Claude Code / opus-5 | 82 | 78.0 | 77.2 | 64.2 | 70.7 | 17.68 | <0.001 |
| Claude Code / sonnet-5 | 81 | 71.6 | 49.4 | 35.6 | 17.3 | 102.02 | <0.001 |
| qwen-code / gpt-oss-120b | 80 | 6.7 | 20.4 | 0.8 | 0.4 | 59.62 | <0.001 |
| qwen-code / Qwen3-Coder-Next | 80 | 1.3 | 0.8 | 0.4 | 0.0 | 3.33 | 0.343 |

Table 4, rank-biserial (positive = first style higher; * = p < 0.05 after Holm):

| Pair | P–E | P–C | P–Pr | E–C | E–Pr | C–Pr |
|---|---|---|---|---|---|---|
| opus-5 | +0.00 | +0.65* | +0.52 | +0.56* | +0.46 | −0.27 |
| sonnet-5 | +0.70* | +0.89* | +1.00* | +0.29* | +0.85* | +0.73* |
| gpt-oss-120b | −0.82* | +0.88* | +0.88* | +0.97* | +1.00* | +1.00 |
| Qwen3-Coder-Next | +0.20 | +0.50 | +1.00 | +0.33 | +1.00 | +1.00 |

- **Conditional vs the direct styles.** Below both Prescriptive and
  Explanatory in all three responsive pairs, all significant. The gaps are
  −13.8 / −13.0 points on Opus 5, −36.0 / −13.8 on Sonnet 5, and
  −5.9 / −19.6 on gpt-oss-120b.
- **Conditional vs Prohibitive** is mixed. On Opus 5 Prohibitive is
  higher (70.7 vs 64.2, n.s.). On Sonnet 5 Prohibitive is the worst style
  of all (17.3%) and significantly below Conditional.
- **Reason clause (P–E).** Opus 5: +0.00, n.s. Sonnet 5: Prescriptive
  higher, +0.70*. gpt-oss-120b: Explanatory higher, −0.82*. That is the
  sign flip behind RQ2.
- **Capability.** On the same harness, the Opus 5 spread is 13.8 points
  and the Sonnet 5 spread is 54.3. Only two models, so this is one
  comparison, not a trend.
- **Floor.** Qwen3-Coder-Next never exceeds 1.3%. gpt-oss-120b peaks at
  20.4% with the reason clause.

## Critique / open questions

- **Style is confounded with one sentence.** There is one text per style
  (Threat 1). Conditional differs from Prescriptive in three ways at
  once: a leading if-clause, the action placed after the condition, and
  "a Python function … its def line" instead of "every Python function".
  The loss of the universal quantifier is as plausible a cause as the
  if-clause. No paraphrase replicates within a style.
- **"Conditional" is not a conditional requirement in the sense the
  writing guide uses.** All four texts carry the same scope ("add or
  modify a Python function"). None is hedged ("if appropriate", "where
  possible"), and none applies the rule only in a subset of situations
  that the other texts cover unconditionally. The finding is about the
  *syntax* of a requirement that is already stated. It is not about
  whether to state a requirement that only holds under a condition.
- **The mechanism is asserted, not measured.** "Must first judge whether
  its action meets the condition" applies equally to "every function you
  add or modify". No traces, acknowledgements or per-function data are
  analysed.
- **The Sonnet 5 reason penalty is unexplained.** A 22-point drop from
  adding a "because our code-audit system…" clause is the largest single
  effect on a Claude model. It contradicts the Opus 5 null and the
  gpt-oss gain. Plausible readings: the reason invites the model to judge
  the rule as irrelevant to its task, or the longer sentence dilutes the
  imperative. The paper does not investigate.
- **Harness and model are confounded across pairs** (authors' point). The
  qwen-code floor could be the harness's ACF injection, the models, or
  the pair. That limits RQ2's "doesn't transfer" to "these two pairs on
  these two harnesses differ".
- **The p-values are likely optimistic.** 86 tasks from 10 repos share an
  ACF and a codebase within a repo, but the tests treat tasks as
  independent blocks (Threat 4). Per-task values are means of at most 3
  runs, so there are many ties. Effects as large as Sonnet 5's would
  survive clustering. Opus 5's 13–14-point Conditional gap is the
  finding most at risk.
- **The all-or-nothing session criterion couples style with the number of
  functions.** McMillan found per-function compliance decays within a
  session (OR 0.944 per function). If style changes how many functions an
  agent writes, session compliance moves for reasons other than wording.
  Function counts per style are not reported. The excluded-session counts
  differ by style (63 for Conditional vs 72–75), which hints that
  workload is not identical.
- **One toy, syntactic marker.** The rule has no task-level purpose, so
  the reason is decoration from the agent's point of view. A reason may
  matter more, in either direction, for rules the agent can relate to its
  task (don't touch generated files, run the linter). Task success,
  quality and cost are not measured, so a style's side effects are
  unknown.
- **Not shown:** whether stating a requirement beats leaving it unstated
  (there is no absent-rule arm; McMillan's 0/524 baseline makes this
  moot for the marker); whether the effect holds for rules placed early
  in the file; anything about multi-rule files, compaction or long
  sessions; and anything about rules with real cost.
- **Comparison with McMillan.** McMillan's Claude Code compliance is
  function-level (Sonnet 4.6 67.7%, Opus 4.6 54.9%). This paper is
  session-level, which is stricter, and Opus 5 reaches 78% Prescriptive.
  So on Claude 5 models a plain imperative in CLAUDE.md binds at least as
  well as on 4.6, and Opus now leads Sonnet. That is across papers,
  languages and units, so treat it as a direction only.

## Trust signals

- **Credibility:** 3. NAIST's software-engineering group (Kula,
  Matsumoto) with Wakayama and Osaka, an established empirical-SE lab. A
  Zenodo replication package is released. The tests are non-parametric
  and appropriate, with Holm correction, and the threats are stated
  candidly. Against: an unreviewed four-page Letter (submitted to IEICE
  Trans.), two weeks old with no citations, one sentence per style, no
  CIs, and no clustering correction across 10 repos. Not 4: no peer
  review yet, and the design cannot separate style from the specific
  wording. Not 2: a reputable group with artifacts released.

## Follow-up

- **Relevance:** 4. It is the first controlled test in this graph of
  *wording*, rather than file structure, for a rule in an agent context
  file. It runs on this box's own channel (Claude Code reading
  CLAUDE.md/AGENTS.md) with current Claude models. It bears directly on
  the writing guide's P1 ("constraints with their reasons") and P2
  wording, and it extends [[concepts/constraint-pinning]]'s "present but
  missed" ceiling from [[literature/papers/mcmillan2026instruction]].
  Material for [[concepts/intent-level-briefing]]. Not 5: one toy
  policy, one sentence per style, and the decisive comparisons rest on
  80-task cells from 10 repos.
- **For the writing guide (`intent-briefing/docs/writing-guide.md`), our
  reading:**
  - **P2 is not undercut.** P2 says to *state* conditional requirements,
    because they go unmet unprompted (yang2025what, 22.9%). Every style
    here states the requirement. What it adds is a phrasing hint: put the
    scope inside an imperative ("Do X to every Y you Z") rather than a
    leading "If you Z, do X". That rests on one sentence pair and can't
    separate the if-clause from the dropped "every".
  - **P1's "with their reasons" is the claim this weakens.** For a
    mechanical rule, a reason added nothing on Opus 5 and cost 22 points
    on Sonnet 5. The guide's case for reasons is generalisation to cases
    the writer didn't foresee. That case is untested here, and it is not
    a compliance claim. The guide should not cite reasons as raising
    adherence. It could say reasons are for judgment rules and are
    optional, or even costly on mid-tier readers, for mechanical ones.
  - **P7 gets one more data point.** On the same harness, the weaker
    Claude reader moved more with wording (54- vs 14-point spread).
  - **P3 is reinforced.** The best wording on the best model was still
    78% session-level compliance on a trivial marker. A rule that must
    hold belongs in a hook or linter.
- **Local check (cheap).** The replication package plus Claude Code
  headless could rerun the four sentences on the box's own models. Add
  "Add the comment … to the line below the def of a Python function you
  add or modify" (Prescriptive without "every") to separate the
  quantifier from the if-clause.
- **Candidates** (cited, not yet in the graph):
  - Gloaguen et al. 2026, "Evaluating AGENTS.md" (arXiv 2602.11988),
    the CTXbench source. It is in the `intent-briefing` project's
    literature, not the hub.
  - Mohsenimofidi, Galster, Treude & Baltes 2026, "Context engineering
    for AI agents in open-source software" (MSR 2026), the taxonomy of
    ACF policy styles.
  - Galster et al. 2026, "Configuring agentic AI coding tools" (AIware
    '26). 90.6% of 2,853 repos have an ACF.
  - Chatlatanagulchai et al. 2026, "Agent READMEs" (TOSEM, accepted).
  - Ding et al. 2026, "OctoBench" (ACL 2026). Per-policy compliance in
    repo-grounded coding is 79.75–85.64%.
  - Yang, He & Zhou 2026, "A first look at coding agents' compliance with
    AI contribution rules" (arXiv 2607.26819). Compliance with rules
    requiring contribution rejection is 0%, even with the rule in the
    prompt.
  - Deaconu et al. 2026, "Do influence tactics matter?" (EMSE). It covers
    prompt framing effects on generated code.
