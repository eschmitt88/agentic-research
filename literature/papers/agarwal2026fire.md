---
kind: paper
title: "FIRE: Failure-Informed Runtime Engineering for Reliable Language-Model Agents"
authors: ["Nikita Agarwal", "Nivedit Jain"]
institutions: ["Failproof AI"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.26048"
code_url: "https://huggingface.co/datasets/failproofai/fire-runtime-policy-reliability"   # policy source, per-attempt outcomes, analysis scripts (808 KB); no raw trajectories
citations: null
source: "raw/papers/agarwal2026fire.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["runtime-policy", "harness-hooks", "stop-hook", "sham-control", "randomized-arms", "pass-hat-k", "terminal-bench", "procedural-loss", "delivery-reliability", "self-correction-null", "codex-cli", "conflict-of-interest"]
---

# FIRE: Failure-Informed Runtime Engineering for Reliable Language-Model Agents

## TL;DR

Hand-written **runtime policies** run in a hook layer around the Codex CLI
(prompt-submit, pre-tool-use and stop events). Each pairs a lexical
eligibility predicate over the task text with a state predicate over the
command log. When the trajectory reaches a state that preceded observed
failures, the policy injects a short instruction (at stop, forcing another
turn) or denies a tool call. The paper's best evidence is a randomized five-arm
panel on 30 Terminal-Bench 2.1 tasks (Terra tier, 300 attempts). The real
portfolio passes **17/28** eligible attempts. A **timing-matched sham** that
fires at the same events with generic review text passes **10/28**, and
always-on "verify"/"reconsider" pass 11/28 and 12/28. So the *content* of
the message does the work, not the interruption. The effect is large but
imprecise. The pre-specified primary contrast is +25.0 pp, 95% CI [7.1, 46.4],
**p = 0.061**, on 14 tasks drawn from the same development population the
policies were derived on. The authors sell the runtime.

## Claims

- **Reliability is partly a harness property.** Weights fix what is
  reachable; the harness decides whether a reached solution survives
  delivery. A *procedural loss* is a failure where (i) there is evidence the
  agent can solve the task, (ii) an observable state precedes the failure,
  and (iii) a bounded intervention could change that transition without
  supplying the solution.
- **Content, not timing.** The sham fires on 26/28 eligible attempts, the
  same rate as the real policy (25/28), and ends 3.6 pp below baseline (CI
  [−17.9, 10.7]). "Being interrupted at the right moment does not help by
  itself."
- **Generic self-checking does not reproduce the gain**, in line with Huang
  et al. 2024 and Kamoi et al. 2024. Always-verify and reconsider fire on
  57 and 59 of 60 attempts and end at or near baseline.
- **Policies convert reachable into repeatable.** For Sol, across the full
  suite, pass∧2 (both attempts succeed) rises 64.4 → 73.6 while pass@2 rises
  86.2 → 87.4 (+1.2).
- **Model-tier leverage.** On the 14 tasks Terra's portfolio covers,
  policy-guided Terra passes 20/28 (71.4%) against unassisted Sol's 18/28
  (64.3%), at $7.19 vs $14.48.

## Methods

- **Policy abstraction** P = (E, G, A, R, S). E is a one-shot eligibility
  predicate over the task description (lexical regex in the frozen
  portfolio). G is a runtime predicate over action history plus policy
  state, evaluated at harness events. A is *instruct* (text, and at stop, a
  required further turn) or *deny* (block the tool call). R plus a nudge
  limit prevent loops. The Terra profile allows **one nudge per rule**. Only
  the database-evidence denial repeats until satisfied. Separating E from G
  "separates scope from timing: broad eligibility spends tokens and
  distracts successful trajectories, and a late runtime predicate cannot
  prevent an irreversible action."
- **Derivation.** Run the policy-free agent twice per task. Inspect every
  failed attempt of a 1/2 task, plus 0/2 failures that show a concrete
  procedural omission. Exclude provider/verifier faults and capability gaps.
  Cluster by mechanism, register a chain (state X → intervention A →
  behaviour B → verifier condition), screen on positive and nearby-negative
  tasks, retire inert or harmful candidates, then **hash-freeze**. The frozen
  Terra portfolio has 8 families and 11 rules, reached after 13 evaluated
  Terra configurations. Retired: broad verify-depth, repetition, generic
  scope and quantifier policies, "whose firing correlated with breakage".
  Six failure types got no policy because "no reliable harness-observable
  intervention point was found."
- **Families (Table 1).** Service persistence (at stop, require a detached
  relaunch plus a *separate later* probe, since "a live PID or a probe
  chained into the launch command is not enough"). Deployment preservation
  (deny destructive git/rm after a successful deploy probe). Database
  evidence (deny a mutating open until a byte copy including WAL sidecars
  exists). Exact replacement (3 rules). LaTeX final-log search. Binary
  equivalence (2 rules). Token accounting. Enumeration. Full JS source is in
  Appendix H.
- **Four designs, strongest causal claim first:** a randomized five-arm
  panel (Terra, 14 eligible + 16 pre-registered silent tasks, 300 attempts,
  hashed before the first attempt); matched service-persistence screens in
  Sol and Luna (7 tasks × 3 attempts, same-day baseline); the complete
  87-task suite in all three tiers (1,044 attempts, **cross-date**, a
  different portfolio per tier); and the 14-task Terra-vs-Sol cohort.
- **Stack.** Codex CLI v0.146.0 at medium reasoning. Three GPT-5.6 provider
  routes, Luna < Terra < Sol, identified by route and not by checkpoint
  hash. Harbor containers. Terminal-Bench 2.1 has 89 tasks; two qemu tasks
  are excluded because their images could not install the agent.
- **Inference.** Tasks are the unit. Two attempts are averaged per
  condition, with a 10,000-resample task bootstrap and a paired sign-flip
  permutation test (100,000 for the panel). Behaviour markers come from
  deterministic extractors over the command log, fixed before the panel and
  applied identically to every arm, with no LLM judge.

## Results

- **Panel (Table 3), real minus comparator, eligible tasks:** sham +25.0
  [7.1, 46.4] p = .061 (primary); baseline +21.4 [7.1, 35.7] p = .032;
  always-verify +21.4 p = .064; reconsider +17.9 [−3.6, 42.9] p = .280.
  Over all 30 tasks: vs sham +15.0 p = .046.
- **Funnel (Table 4).** The registered risky state arises in 24–26 of 28
  attempts in every arm. The registered behaviour follows in **22/24** coded
  real-policy attempts against 11–14/24 in the other arms (sham 13). Under
  the worst-case recoding of the 4 uncoded attempts, it is 22/28 against at
  most 18/28. **Behaviour is necessary, not sufficient**: of 22 real-policy
  attempts with the behaviour, 12 pass. Binary equivalence produces the
  measured comparison on both attempts and fails both, a capability gap.
- **Per family.** Service persistence fires 11/12, gives a detached launch
  plus probe 12/12, and passes 7/12 (baseline 4/12, sham 3/12). Database
  evidence passes 4/4 (baseline 3/4, sham 2/4).
- **Off-target.** The real portfolio never fires on the 16 silent tasks
  (21/32 vs baseline 20/32). **Always-verify hurts them: 16/32**. Real minus
  always-verify on silent tasks is +15.6, CI [3.1, 31.3], the one silent
  contrast whose interval excludes zero. The eligible-minus-silent
  interaction (+18.8, CI [−4.9, 42.9], p = 0.20) is imprecise, and the silent
  tasks are easier (baseline 62.5% vs 39.3%).
- **Cost of the real arm (panel):** $19.92 vs $23.22 baseline. Mean agent
  time 418 → 473 s, almost all of it on eligible tasks (190 → 273 s).
- **Replication (§5.3).** Service persistence rises 7/21 → 19/21 (Sol) and
  11/21 → 19/21 (Luna), with no pass-to-fail on any task. Exact sign-flip
  p = 0.063 and 0.125. Excluding the two qemu tasks later dropped from the
  suite, 4/15 → 13/15 and 8/15 → 14/15.
- **Complete suite (Table 5).** pass@1 Luna 59.8 → 63.2 (p = .43), Terra
  64.4 → 70.7 (p = .17), Sol 75.3 → 80.5 (p = .09). **None is significant.**
  pass@2 69.0 → 72.4 / 73.6 → 80.5 / 86.2 → 87.4. pass∧2 50.6 → 54.0 /
  55.2 → 60.9 / 64.4 → 73.6, with no interval reported. Fired on 12 / 15 /
  **81** of 87 tasks. Cost −5.2% / +0.9% / **+47.7%**. Give-ups fall
  61→54, 58→41, 36→25.
- **14-task cohort (Table 8).** Terra 10/28 → 20/28 ($5.85 → $7.19). Sol
  18/28 → 24/28 ($14.48 → $27.17). Terra+policy minus bare Sol is +7.1,
  CI [−21.4, 35.7], **p = 0.82**.

## Critique / open questions

- **Development-population panel.** The panel's 14 eligible tasks are the
  tasks the Terra portfolio was derived and screened on. The authors say
  so: the panel "establishes a content effect on this distribution, not
  generalization". The rules name no task, but their regexes were written
  against these task descriptions (for example `hello\.html`, `mystery\.c`,
  `port 8080`). Read the +25 pp as "text written for a failure fixes that
  failure", which is a narrower claim than "runtime policies transfer".
- **The primary contrast misses its own threshold.** p = 0.061 against a
  bootstrap CI that excludes zero. The paper reports both instead of
  choosing, which is to its credit. It is 14 tasks with two attempts each.
- **Missing arm.** No condition runs the *real* policy text always-on
  (unscoped). The panel therefore separates content from timing, but not
  targeting from content. Always-verify is generic text, so its harm to
  silent tasks does not show that the real rules would cause the same harm
  if unscoped. The planned leave-one-family-out ablation was not run.
- **"Converts reachable into repeatable" holds cleanly for one tier only.**
  The flat-pass@2 / +9.2 pass∧2 pattern comes from Sol's broad portfolio
  (fires on 81/87 tasks, +47.7% cost). Sol was frozen earlier, with five
  broad stop-time families, not the targeted design the paper argues for.
  Terra's targeted portfolio raises reach *more* than repeatability
  (+6.9 vs +5.7). The flat pass@2 is also net churn (Table 6, my
  reading): 4 Sol tasks gained reach (0/2 → ≥1/2), 3 lost it (1/2 → 0/2),
  and 1 dropped from 2/2 to 1/2.
- **Complete-suite noise floor, derivable from the paper.** Terra's
  portfolio fired on 15 tasks, yet 32 tasks changed their success count
  (21 improved, 11 regressed). At least 17 of 87 tasks moved with no
  intervention in the policy arm. For Luna, 25 changed and 12 fired, so at
  least 13 moved. That is resampling plus cross-date route drift, and it
  is the scale the non-panel numbers have to clear.
- **The 14-task Terra ≈ Sol claim is selection-favoured.** The cohort is
  the set of tasks chosen by Terra's own failure modes. Unassisted Terra
  starts at 35.7% there, and Sol *with its own* portfolio reaches 85.7%. So
  the policies do not erase the tier gap; both tiers gain. At p = 0.82 the
  authors disclaim general superiority.
- **Soft gate, one nudge.** The stop-time instruct refuses completion once
  and then lets the agent stop. It is an evidence *prompt* with a hard-
  evidence predicate, not an accept-only-with-evidence gate. The deny rules
  are real action gates, but the sham replaced deny with instruct, so the
  deny mechanism is not isolated.
- **Lexical routing is fragile, and the paper shows it.** "A phrase
  meaning 'one per line' alone falsely routed a chess task", and the
  database guard "initially missed Python SQLite opens". One earlier
  service hook "fired on 82/178 trials". Portfolio authoring "required
  expert judgement, and researcher hours were not logged". The cost of
  deriving the policies is unpriced.
- **Conflict of interest.** "Both authors are cofounders of Failproof AI,
  which develops the runtime infrastructure used to implement the
  policies". The source imports `from "failproofai"`.

## Trust signals

- **Credibility:** 3. An unknown two-person startup, an unreviewed arXiv
  preprint, days old, with a declared commercial interest in the result
  (that would place it at 2). Lifted to 3 by unusually disciplined design
  and disclosure: a pre-registered primary contrast, arm sources and the
  schedule hashed before the first attempt, a timing-matched sham,
  deterministic arm-blind behaviour coding with no LLM judge, a frozen
  attempt-replacement rule with a lineage audit, task-level inference that
  reports the unfavourable p beside the favourable CI, and a released
  artifact (policy source, 1,044 per-attempt outcomes, analysis scripts;
  checked live on HF, 808 KB, Apache-2.0 / CC BY 4.0). The artifact does
  not include raw trajectories. Trust the panel's direction and the
  sham/generic nulls. Discount the complete-suite and tier-leverage
  numbers, which are cross-date and not significant.

## Follow-up

- **Relevance:** 4. The cleanest controlled evidence in the graph that a
  harness hook's *message content*, delivered at a state-triggered moment,
  changes agent behaviour and outcome, while identical timing with generic
  content does not. It uses exactly the hook surface this box runs on
  (Stop / PreToolUse / UserPromptSubmit). Not a 5: Terminal-Bench
  delivery tasks, not research agents; one vendor, one harness, one model
  family; and the panel does not test generalization.
- **[[concepts/evidence-gated-completion]].** This is the first randomized
  stop-time refusal with a sham that holds the refusal constant and varies
  the message. The refusal alone is worth about zero. The message that
  names the missing hard evidence carries the effect. See the concept's
  2026-09-29 section.
- **[[concepts/enforcement-boundary-placement]].** This bears on the
  dai2026agentguard caveat ("conditional activation is unmeasured").
  Generic always-on text measurably harms untargeted tasks, but
  real-text-unscoped was never run. See the concept's 2026-09-29 note.
- **[[concepts/pass-at-k]].** Unfired tasks work as a within-experiment
  placebo, and the transition matrix is needed beyond the two marginals.
  See the 2026-09-29 section.
- **[[concepts/typed-enforcement]], [[concepts/permission-gate-as-architecture]]:**
  cite only. The regex predicates' false routing is another instance of
  the semantic escape hatch. The deny rules were not isolated.
- **For claude-system (a candidate for `/elevate`, not a proposal):** a
  Stop hook that checks a hard, command-log-observable predicate and
  returns one specific instruction is cheap and matches the measured
  design. Generic "verify your work" stop text is the arm that did nothing
  on eligible tasks and cost 4/32 on the silent ones.
- **Candidates.** AgentSpec (Wang, Poskitt, Sun 2026, ICSE) is the
  peer-reviewed rule language closest to this abstraction. Huang et al.
  2024 (ICLR, "LLMs cannot self-correct reasoning yet") anchors the
  generic-reconsider null.
