---
kind: paper
title: "AgentGuard: Learning Execution Guardrails from Anomalous Coding-Agent Trajectories"
authors: ["Wuyang Dai", "Song Wang"]
institutions: ["York University (Lassonde School of Engineering)"]
year: 2026
venue: "arXiv (cs.SE); formatted as an AAAI 2027 submission"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.16287"
code_url: null
citations: null
source: "raw/papers/dai2026agentguard.pdf"
added: "2026-09-22"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/skill-library-lifecycle]]"
tags: ["coding-agent", "guardrails", "prompt-placed-constraint", "over-refusal", "conditional-activation", "skill-routing", "claude-code", "single-repository", "no-code-released", "unblinded-annotation", "no-ablation", "self-benchmark"]
---

# AgentGuard: Learning Execution Guardrails from Anomalous Coding-Agent Trajectories

## TL;DR

461 reviewed failure traces from coding-agent runs were distilled into **15
conditional behavioral rules**, grouped under **5 routing entries**, and
shipped as Claude Code subskills that a router loads per instruction. On 100
held-out tasks × 3 runs × 2 conditions (600 Docker-isolated executions,
Claude Code 2.1.19 + Claude Haiku 4.5), the guarded agent cut the Abnormal
Execution Rate from **69.0% to 26.7%** (−42.3 pp, task-clustered 95% CI
[−51.7, −33.0], p < 0.001) and raised Successful Task Completion from
**21.7% to 35.0%** (+13.3 pp, CI [5.3, 21.7], p = 0.0022), at +2.5% cost and
+3.9% wall time.

Two things the digest and the abstract both need walking back. **First, the
constraints are not enforced.** The paper says so twice, in its own words:
the design "requires no model fine-tuning, tool modification, or *runtime
enforcement mechanism*," and "The routing skill and guardrail subskill add
instructions to the model's context; **they do not change tool permissions or
intercept actions**." This is placement-in-the-prompt, measured. **Second,
the conditional-activation mechanism — the paper's headline novelty and the
reason this repo flagged it — is never ablated.** There is no always-on arm,
no routing-off arm, no rule-count sweep, no second model, no second
repository. The only comparison in the paper is guardrails-on vs
guardrails-off.

The paper's own primary finding against itself: over-refusal, **19.3%
(58/300) against a baseline of exactly zero**, which the authors name as "the
primary failure mode introduced by AgentGuard." The abstract's claim of
"minimizing unnecessary restrictions on normal execution" is the one thing
the body measures and refutes.

## Claims

- **Behavioral constraints can be induced automatically from reviewed
  anomalous traces** rather than hand-authored, and the induced rules
  transfer to held-out tasks. 461 grounded findings consolidate to 15
  guardrails.
- **Negative constraints ("do not X unless Y, instead Z") are the right
  form** — inherited from Zhang et al. 2026, cited as prior empirical
  evidence that "negative behavioral constraints consistently outperform
  prescriptive guidance for coding agents."
- **Conditional, per-instruction activation beats uniform enforcement**
  because uniform rule sets "may unnecessarily restrict benign behaviors that
  are irrelevant to the current execution context." **This is asserted in the
  introduction and motivated in §3.4 — and never tested.**
- **Task success does not certify execution.** An execution is abnormal when
  an action "violates task scope, repository or environment integrity, safe
  recovery, validation, artifact, or evidence-grounded reporting
  requirements" — so "a trace may be anomalous even when its final output
  appears plausible." This is the framing contribution and it is sound.
- **Over-refusal is the primary limitation of learned behavioral
  guardrails** — stated in the contributions list, not buried. Credit where
  due.
- **Claimed novelty:** "the first instruction-level framework that
  automatically learns conditional behavioral guardrails from historical
  anomalous coding-agent trajectories." The qualifier chain is doing heavy
  work; they cite Aggarwal & Ghalaty 2026 (self-improving coding agents via
  *accumulated behavioral rules*, closed-loop) in related work.

## Methods

- **Trace representation.** A case is `⟨(instruction_i, execution_i)⟩`, each
  execution an ordered sequence of `(response, action, result)` triples with
  metadata (tool names, command arguments, file paths, exit status,
  repository changes, artifacts). Two-level by design: triples give
  fine-grained evidence, the full trace preserves cross-instruction
  dependencies.
- **Evidence extraction.** Each reviewed anomaly becomes
  `finding = (context, action, outcome, support, stage)`. A finding is used
  "only when its action, outcome, and trace support are observable. Broad
  summaries alone do not support rule induction." Both the *preceding*
  interactions (inherited constraints, prior decisions) and the *later* ones
  (downstream effects, recovery attempts) are read in.
- **Rule form.** `r = (When, DoNot, Unless, Instead, ApplyAt)`. `Unless` is
  populated only when task conditions can legitimately permit the same
  action; `Instead` is "the smallest alternative that avoids the outcome
  while preserving the valid task goal." Repository names, paths and error
  strings are generalized "only when doing so does not change when the rule
  applies." Every rule keeps a link to its supporting trace.
- **Consolidation** (Algorithm 1, iterated to fixpoint): duplicate removal
  (same stage, same prohibited action under compatible conditions, same
  prescribed response — evidence merged); subsumption (a general rule absorbs
  a specific one only if it "covers the same cases without restricting
  additional legitimate behavior," and the specific rule's exceptions are
  preserved); conflict separation (different responses to the same apparent
  action stay separate or get refined preconditions). Rules failing the
  validation criteria — grounded in trace evidence, observable *before* the
  action, actionable via a clear alternative, non-interfering with legitimate
  work — are dropped.
- **Routing.** 15 guardrail subskills under 5 routing entries: (1)
  understanding and path resolution, (2) command execution and recovery, (3)
  project changes, (4) filesystem / version control / external-state
  mutation, (5) artifacts, validation and reporting. A main skill identifies
  the requested type of work, selects the entry, loads the subskill. "A
  routing entry adds no new rule; it only organizes the existing guardrails."
- **Data.** 642 documented failures over 382 tasks from **ABTest (Dai et al.
  2026)** — the first author's own prior benchmark. Split *by task* to
  prevent leakage: 461 traces / 282 tasks for construction, 100 tasks held
  out. Each task is "a coherent multi-step repository workflow with one
  injected adversarial step designed to elicit abnormal behavior" — unsafe
  requests, missing commands/files/interfaces, or ambiguous instructions
  needing clarification.
- **Evaluation.** 100 tasks × 3 runs × 2 conditions = 600 executions. Claude
  Code 2.1.19 + Claude Haiku 4.5, fresh Docker container per run (2 CPUs,
  8 GB), **the same Click repository snapshot every time**. "Tool access,
  permissions, and runtime configurations are identical in both conditions;
  the only difference is the addition of AgentGuard."
- **Annotation.** "Three human reviewers independently assessed all 100 tasks
  and all 600 execution traces," blind to each other and to prior verdicts;
  disagreements adjudicated by joint re-examination of cited evidence. Fleiss'
  κ = **0.87** pre-adjudication on the outcome measures and **0.81** on the
  three-way effect categorization. Reviewer identity is never stated.
- **Disclosed unblinding.** "Since AgentGuard adds explicit guardrail
  instructions to the execution trace, reviewers could identify the
  evaluation condition during annotation."
- **Statistics.** Task-level nonparametric cluster bootstrap for 95% CIs,
  exact paired randomization tests for hypothesis testing. Correctly clusters
  the 3 runs within task.

## Results

### Headline table (Table 2; denominators are 300 executions per arm)

| Metric | CC | CC + AgentGuard | Δ (pp) | 95% CI | p |
|---|---|---|---|---|---|
| AER | 69.0% (207/300) | 26.7% (80/300) | −42.3 | [−51.7, −33.0] | < 0.001 |
| CAR | 25.0% (75/300) | 62.7% (188/300) | +37.7 | [28.3, 47.0] | < 0.001 |
| CBR | 49.0% (147/300) | 42.0% (126/300) | −7.0 | [−17.0, 3.0] | 0.199 |
| ORR | — (0) | 19.3% (58/300) | +19.3 | [12.7, 26.7] | < 0.001 |
| Success | 21.7% (65/300) | 35.0% (105/300) | +13.3 | [5.3, 21.7] | 0.0022 |

- **The safety effect is real and large.** −42.3 pp on AER, −61.4% relative,
  with a CI nowhere near zero. CAR more than doubles (+150.7% relative).
- **The refusal cost is real and large too.** ORR goes from an exact zero to
  19.3%. The authors state it plainly: this "identif[ies] over-refusal as the
  primary failure mode introduced by AgentGuard."
- **The benign-completion result is an underpowered null, not a
  preservation.** −7.0 pp, p = 0.199, CI [−17.0, 3.0]. The paper's own
  wording is careful — "providing no evidence of a systematic reduction in
  benign task completion" — but the **conclusion** then says AgentGuard
  "substantially reduces abnormal execution **while preserving benign
  behaviors**." The CI admits a 17-point loss. That is a walk-forward, not a
  walk-back.
- **Absolute performance stays poor in both arms.** 35.0% success means 65%
  of guarded executions still fail. The baseline's 69.0% AER is the behavior
  of a small, cheap model (Haiku 4.5) under a deliberately adversarial task
  distribution.
- **Metric definitions say "tasks," the table computes over executions.**
  All five metrics are defined in §4.4 as "the percentage of *tasks*…" but
  every denominator in Table 2 is 300 = 100 tasks × 3 runs. Table 3 is the
  only genuinely task-level view. Minor, but it means "69.0% of tasks" is not
  what was measured.

### Task-level transition (Table 3, n = 100 tasks)

| Status | n | % |
|---|---|---|
| Abnormal under CC only | 41 | 41.0 |
| Abnormal under both | 33 | 33.0 |
| Abnormal under CC + AgentGuard only | 3 | 3.0 |
| Abnormal under neither | 23 | 23.0 |

The 41-vs-3 asymmetry is the cleanest result in the paper and does not depend
on any rate arithmetic. It is also the honest bound: **33% of tasks stay
abnormal under the guardrails**, so "the guardrails do not suppress every
known failure mode" — the authors' phrase.

### Trace-level attribution (Table 4, 300 guarded executions, κ = 0.81)

| Category | Effect | n | % |
|---|---|---|---|
| Beneficial | Safety gain, no benign loss | 100 | 33.3 |
| Beneficial | Safety gain with benign loss | 29 | 9.7 |
| Neutral | No attributed effect | 139 | 46.3 |
| Adverse | Benign loss only | 25 | 8.3 |
| Adverse | Safety loss only | 6 | 2.0 |
| Adverse | Safety loss with benign loss | 1 | 0.3 |

- 129 safety gains vs 32 adverse effects. **100 of the 129 gains preserve all
  benign work** — the paper's argument that the improvement "cannot be
  explained merely by conservative or blanket refusals." That argument is
  good and it is the right one to make.
- But **55 executions (29 + 25 + 1) incur benign utility loss**, 18.3% of the
  guarded arm. The authors' own diagnosis: "the primary remaining challenge
  is enabling the agent to safely continue execution after a guardrail is
  triggered, rather than prematurely abandoning feasible work." That is a
  precise, useful statement of the failure mechanism — the rule fires, and
  the agent treats a local prohibition as a global stop.
- **46.3% of guarded executions have no attributed effect at all.** Nearly
  half the time the machinery is inert.

### Per-routing-entry effects (Figure 2; beneficial / neutral / adverse)

Understanding & path 63 / 91 / 27 · Execution & recovery 120 / 80 / 14 ·
Project changes 40 / 34 / 8 · Filesystem & external 22 / 58 / 17 ·
Artifacts & reporting 113 / 113 / 29. *(Read off a bar chart; the body text
independently corroborates 120, 29 and 27.)* Activations exceed executions
because one run can trigger several entries. Execution/recovery is the
workhorse; adverse effects concentrate in artifacts/reporting and
understanding/path — i.e. in the entries whose rules demand *evidence* before
a claim, which is exactly where an over-cautious agent stops working.

### Overhead (Table 5)

Mean cost $0.158 → $0.162 (+2.5%); mean execution time 93.3 s → 97.0 s
(+3.9%); mean tool calls 21.8 → 22.7 (+4.1%). Cheap, and the routing design
is the plausible reason — but see the ablation gap: this is the *benefit*
conditional activation was introduced to deliver, and it is measured only in
the arm where routing is on, with nothing to compare against.

## Critique / open questions

- **The central design claim is unablated.** Conditional activation is the
  novelty in the title, the abstract, the contributions list and §3.4, and
  the paper contains exactly two arms: guardrails off, guardrails on. There
  is no always-on arm, no routing-off arm, no rule-subset sweep. The only
  text in the paper on this axis is a motivation sentence — "Rather than
  loading all guardrails for every task, AgentGuard first selects the routing
  category most relevant to the current instruction" — and a rationale,
  "Loading every guardrail for every instruction would lengthen the prompt
  and require the agent to consider rules unrelated to the current task."
  Plausible; untested. Every number in this paper is equally consistent with
  "15 rules pasted in unconditionally would have done the same or better."
- **The abstract's design goal is refuted by the paper's own primary
  measurement.** "This design enables behavioral guidance while minimizing
  unnecessary restrictions on normal execution" vs ORR 19.3% against a 0%
  baseline, named in §5.1 as the primary introduced failure mode. The
  abstract does end honestly ("highlighting the remaining challenge of
  balancing safety and task completion"), so this is overstatement, not
  concealment — but a reader who stops at sentence three gets the opposite of
  the result.
- **No code, no rules, no data released.** There is no artifact-availability
  statement anywhere in the paper. Table 1 gives **one abstracted sentence
  per guardrail**, with the repeated caveat that "the implemented rules are
  substantially more detailed." The 15 rule texts — the entire contribution —
  are never shown. Nobody can reproduce, inspect, or reuse them.
- **One repository.** Every one of the 600 executions runs against "the same
  Click repository snapshot." A Python CLI library. Rules induced from traces
  over a corpus and evaluated on one held-out repo tell us the rules
  generalize across *tasks*, and nothing about whether they generalize across
  *codebases* — which is the claim the word "reusable" makes throughout.
- **One model, and a small one.** Claude Haiku 4.5 only. No frontier model,
  no cross-family check. Prompt-placed guidance is exactly the kind of
  intervention whose effect size should be expected to shrink as the base
  model's own judgment improves — a 69.0% baseline AER leaves enormous
  headroom that a stronger model may already occupy.
- **Self-benchmark.** The evaluation data is ABTest, Dai et al. 2026 — the
  first author's own prior work. Not disqualifying, but it means the task
  distribution, the adversarial-step design, and the definition of "abnormal"
  all come from the same people measuring the improvement.
- **Unblinded human grading of one's own intervention.** Disclosed, which is
  to the authors' credit, and κ = 0.87 is high. But the reviewers are never
  identified, the paper has two authors and claims three reviewers, and the
  outcome measures ("unnecessarily refuses," "all applicable benign
  requirements are completed," "claims are supported by trace evidence") are
  irreducibly judgment calls. High inter-rater agreement among reviewers who
  can all see the condition and share a stake measures consistency, not
  validity. **This is the single biggest threat to the 42-point effect, and
  the paper does not discuss it beyond the one disclosure sentence.**
- **No limitations or threats-to-validity section.** The paper runs
  Introduction → Related Work → Method → Experimental Design → Results →
  Conclusion. For an empirical SE paper this is a conspicuous omission; the
  candid statements that do exist (over-refusal, 33% residual) are scattered
  through Results.
- **100% of evaluation tasks contain an injected adversarial step.** So the
  19.3% over-refusal rate is measured on a distribution where something
  genuinely *should* be refused in every single task. On a benign-only
  distribution — the normal case in production — the over-refusal rate is
  unmeasured and there is no reason to assume it is lower. The paper has no
  clean-task control arm.
- **Novelty claim vs cited prior work.** Aggarwal & Ghalaty 2026 accumulate
  behavioral rules in a closed loop; Zhang et al. 2026 establish that
  guardrails beat guidance. The delta AgentGuard owns is *conditional
  routing* — the one thing not ablated.

## Trust signals

- **Credibility: 2.** Real institution with a real SE track record (Song
  Wang, Lassonde/York), a genuinely well-constructed experiment — 600
  isolated runs, task-level split with no leakage, task-clustered bootstraps,
  exact paired randomization tests, κ reported both pre- and post-adjudication,
  and a non-significant result reported as non-significant (CBR, p = 0.199).
  The statistical machinery alone would earn a 4. Held down to 2 by a
  reproducibility and scope profile that is close to worst-case for a claim
  of this size: **no code and no released guardrail texts** (the contribution
  itself is unavailable for inspection), **one repository**, **one small
  model**, the authors' **own benchmark**, **unblinded human annotators of
  unstated identity grading their own intervention on subjective criteria**,
  **no limitations section**, and **no ablation of the paper's headline
  mechanism**. An unverifiable 42-point effect from subjective unblinded
  grading of an unreleased artifact on a single repo is not a 3.

## Follow-up

- **Relevance: 4.** Domain is coding agents, not research agents, and the
  digest's stated reason for flagging it turns out not to exist. It scores 4
  anyway because it is the **first source in this graph that measures a
  prompt-placed constraint working**, in the exact harness this repo runs
  (Claude Code, `SKILL.md` subskills, a router), *and* pairs it with the
  refusal cost — which is precisely the pair
  [[concepts/refusal-cost-symmetry]] demands and
  [[concepts/enforcement-boundary-placement]]'s table has only negatives for.
  Not 5: no new concept, one repo, one model, no released artifact, and the
  mechanism that would have made it load-bearing here is unablated.

- **[[concepts/enforcement-boundary-placement]] — propose a new table row,
  and a correction to how the "Nowhere" rows read.** The concept's table
  currently has two prompt-placement entries and both are negatives
  ([[literature/papers/paglieri2026case]]'s unenforced anti-cheating prompt;
  [[literature/papers/nepal2026faithful]]'s dispositional rule at 1.2% vs
  1.1%). This paper is the measured **positive** case for the same placement,
  and its disclaimer is unusually explicit — "they do not change tool
  permissions or intercept actions" — so there is no ambiguity about which
  row it belongs in. Proposed row: *Placement:* In the agent's own context,
  conditionally loaded. *Mechanism:* 15 trace-induced `(When, DoNot, Unless,
  Instead, ApplyAt)` rules as routed subskills; no permission change, no
  interception. AER 69.0% → 26.7%, ORR 0% → 19.3%.

  The synthesis worth writing down is that this **does not contradict
  nepal2026faithful — it confirms nepal's own rule from the other side.**
  Nepal's finding was a split by *predicate*: "every rule the agent followed
  was one it could be graded on, and no rule about how to behave was
  followed." AgentGuard's 15 rules are, without exception, gradable
  predicates over observable actions — *do not change the assertion*, *edit
  only the necessary files*, *do not weaken the check*, *reopen and parse the
  artifact*. So prompt placement buys real compliance on action-predicates
  and none on dispositional ones. That sharpens the concept's current
  necessary-condition framing ("checkability buys detectability, not
  compliance") into something more precise: **checkability buys detectability
  always, and buys partial compliance when the predicate is about an action
  the agent is about to take.** But the ceiling is visible and low — 33% of
  tasks stay abnormal, and the constrained component can still reach the
  constraint, so nothing here survives an adversary. Prompt placement is a
  reliability intervention, not a security boundary, and this paper is the
  cleanest evidence in the graph for that exact distinction.

- **[[concepts/refusal-cost-symmetry]] — propose adding as a source, with a
  caveat about the pairing.** The concept's load-bearing requirement is a
  *paired* legitimate instance, superficially indistinguishable from the
  violation. AgentGuard partially satisfies it by construction: each
  evaluation task bundles one injected adversarial step with legitimate work
  that "remains independently feasible after an unsafe step is blocked," and
  over-refusal is scored on exactly that residual. That is a genuine paired
  design, and it produces the number the concept wants — **19.3% over-refusal
  against 0%, with the safety gain of −42.3 pp measured on the same runs.**
  The trace-level split is even better: 100 safety gains with no benign loss
  vs 29 with, which separates "the guardrail worked" from "the guardrail
  scared the agent off."

  Two caveats to record with it. (i) The pairing is *within-task*, not
  *between-task*: there is no clean task with no adversarial step, so the
  rate at which guardrails fire on wholly benign work is unmeasured. (ii) The
  authors compute the safety/utility trade as favorable ("the reduction in
  abnormal executions outweighs the loss caused by over-refusal") using
  Success as the arbiter — but Success is a conjunction that already contains
  both terms, so it is a definition, not a weighing. The concept's point is
  that the weights are a *choice*; this paper makes the choice implicitly and
  calls it a result.

- **[[concepts/permission-gate-as-architecture]] — record and decline to
  edit.** This is the concept's photographic negative and belongs in the
  reasoning, not the source list. The gate role here is filled by the model
  itself: the router selects rules, the model decides whether to obey, and
  nothing stateful regulates the policy. "Tool access, permissions, and
  runtime configurations are identical in both conditions." The 33% residual
  abnormality and the 6 net-new safety losses are what a "gate" that is
  really a suggestion costs. Useful as a contrast case when the concept is
  next revised; not evidence about gates.

- **[[concepts/constraint-pinning]] — record and decline to edit, but log the
  question.** AgentGuard injects constraints into context *per instruction*,
  which is structurally the scenario constraint-pinning exists for: a
  governance constraint living in a buffer that compaction can evict. The
  paper never measures survival across context compaction, never reports
  context length, and the runs are short (93–97 s, ~22 tool calls) — almost
  certainly too short to compact. So it supplies no evidence either way. It
  does supply a **sharpened open question for the concept**: per-instruction
  re-injection is a *stronger* discipline than pinning-once, because the rule
  is re-delivered at each decision point rather than defended from eviction.
  Whether re-injection or pinning is the better mechanism is now a concrete
  question with a plausible experiment behind it.

- **[[concepts/skill-library-lifecycle]] — record; weak bearing.** The
  induction→consolidation→routing pipeline (Algorithm 1: dedupe, subsume
  preserving exceptions, separate conflicts, iterate to fixpoint, drop rules
  failing validation) is a reasonable sketch of automated skill-library
  maintenance, and the fixpoint-consolidation step is the part this repo's
  concept is thinnest on. But it is run **once, offline, on a fixed corpus** —
  no growth over time, no measured library drift, no re-induction after
  deployment, and the output is 15 rules, small enough that consolidation
  quality is untested at scale. Mention as prior art; do not treat as
  evidence.

- **[[concepts/shared-skill-namespace]] — decline, no bearing.** The subskills
  are Claude-Code-specific and the paper makes no portability, registry or
  cross-harness claim. The only connection is that both involve `SKILL.md`
  files.

- **Direct bearing on this repo's 2026-09-20 `@import` finding.** The digest
  framed this paper as the head-to-head that would settle conditional
  activation vs always-on rules. It is not, and that gap matters more than
  the paper's positive result: this repo cannot cite anything here to justify
  path-scoped `.claude/rules/`. What the paper *does* supply is the harder
  prior — that a rule which reaches the context at the right moment moves
  behavior by 42 points on action-predicates, which means the 09-20 finding
  (rule files declared but never actually loaded into any session) is a
  materially costly bug rather than a tidiness issue. It also supplies the
  matching warning: the same mechanism produced 19.3% over-refusal, so rules
  that *do* land are not free. And the ceiling is the real lesson — 33% of
  tasks stayed abnormal with the rules present and correct. Text in context
  is worth fixing; it is not worth trusting.

- **Cited work worth a future digest,** none currently in this graph:
  - **Zhang et al. 2026, "Do Agent Rules Shape or Distort? Guardrails Beat
    Guidance in Coding Agents" (arXiv:2604.11088)** — the strongest candidate
    by a distance. This is the head-to-head-of-rule-forms paper AgentGuard
    leans on for its negative-constraint design, and it is plausibly the
    paper the digest actually wanted.
  - **Han et al. 2026, "SWE-Skills-Bench: Do Agent Skills Actually Help in
    Real-World Software Engineering?" (arXiv:2603.15401)** — AgentGuard's own
    related work concedes skill effectiveness "is inconsistent across tasks,"
    citing this. Directly relevant to [[concepts/skill-library-lifecycle]].
  - **Aggarwal & Ghalaty 2026, "Self-Improving AI Coding Agents Through
    Accumulated Behavioral Rules: A Closed-Loop Framework"
    (arXiv:2607.13091)** — the closed loop AgentGuard's one-shot offline
    induction lacks.
  - **Dai et al. 2026, ABTest (arXiv:2604.03362)** — the substrate for every
    number above; needed to audit the definition of "abnormal."
  - **Mehtiyev & Assunção 2026, "Beyond Resolution Rates: Behavioral Drivers
    of Coding Agent Success and Failure" (arXiv:2604.02547)** — trajectory
    analysis over end-state scoring; bears on [[concepts/hce-evaluation]].
