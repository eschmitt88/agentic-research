---
kind: paper
title: "How Do Agent Harnesses Create Value? Planning Information and Release Control in Stateful LLM Agents"
authors: ["Yukun Zhang", "Kemu Xu", "Yishen Chen"]
institutions: ["The Chinese University of Hong Kong", "University of Edinburgh", "The Chinese University of Hong Kong, Shenzhen"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.20474"
code_url: null   # "a public repository link is not yet available at the time of this version"
citations: null
source: "raw/papers/zhang2026how.pdf"
added: "2026-09-21"
relevance: 4     # supplies the decision-theoretic form refusal-cost-symmetry's open question asked for, plus a live gate's catch/false-reject pair; domain is customer-service tool use, not ML research
credibility: 2   # unreviewed preprint, no code, cheap non-frontier models, every Holm-adjusted p > 0.05 — held above 1 only by unusually candid self-limitation
status: read
related_experiments: []
related_concepts:
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/hce-evaluation]]"
tags: ["harness", "planning", "verification", "false-pass", "placebo-control", "cost-sensitive", "liability", "tau2-bench", "component-attribution", "false-rejection"]
---

# How Do Agent Harnesses Create Value? Planning Information and Release Control in Stateful LLM Agents

## TL;DR

Two harness components are measured separately against controls built for
them. Prewritten task-specific plans beat **shuffled policy words matched in
whitespace-word count and wrapper** by 7.17 pp of oracle-verified success —
a point estimate whose 90% task-clustered interval, **[1.15, 13.36]**, very
nearly touches zero and whose per-model Holm-adjusted p-values are all above
0.05. A terminal verifier that reads **only the last ≤ 8 dialogue messages,
with no database access**, rejects 61% of oracle-invalid Retail episodes and
withholds 17% of correct ones for **$0.0079 per episode**, cutting the
false-pass rate from 57.21% to 20.96%. The paper then prices the two against
each other: at a low loss per erroneous acceptance the planning gain wins, at
a high loss the verifier wins, and **the standalone verifier captures almost
all of the full stack's avoided false passes at a twelfth of its cost**.

## Claims

- **A word-count-matched word salad isolates guidance content from guidance
  presence.** "The primary comparison assigns prewritten task-specific Fixed
  guidance or Sham text formed from shuffled policy words, matched in
  whitespace-word count and input wrapper." The authors are explicit about
  what the control is and is not: "This is a shuffled word sequence, not a
  coherent but unhelpful plan. It controls word count and packaging, not
  readability or task structure; tokenizer length can also differ."
- **Supplying the content is worth ~7 pp, concentrated in harder tasks.**
  τ̂_P = 0.0717 over 265 matched model–task–replication cells, four models,
  23 mutation-flagged Retail tasks.
- **The placebo arm is not a handicap.** Sham − Minimal is **+1.89 pp**
  with interval [−3.45, +7.09]: adding ~95 words of shuffled policy text to
  the system context neither helps nor hurts on average. "Its interval spans
  zero, so this diagnostic does not establish equivalence."
- **Oracle correctness and terminal acceptance are different margins.**
  Z (oracle reward ≥ 0.999), S (the agent asserted completion), R (the
  verifier rejected). Verified success is Y^VS = Z; false pass is
  Y^FP = S(1−R)(1−Z); gate acceptance is A = S(1−R). "Y^VS = Z spans the
  first row: the verifier changes columns, not rows."
- **Component value is scenario-dependent and the ordering inverts.**
  NV_i(V,L) = V·ΔP_i^VS + L·ΔFP_i^avoid − ΔC_i, with a break-even liability
  L*_i(V) = (ΔC_i − V·ΔP_i^VS)/ΔFP_i^avoid. "Which component matters more
  depends on the loss assigned to erroneous acceptance."
- **The bundled stack is the worst buy on every axis except false pass.**
  Full Fixed has lower pooled verified success than Minimal, 5.4× its mean
  logged cost, and avoids essentially the same false passes as the verifier
  alone.
- **The verifier's apparent success gain is not the verifier's.** "The
  terminal verifier acts after oracle scoring, so ΔP reflects the arms'
  executions rather than a gain produced by the check." They zero the
  success term as a check and the ordering holds.

## Methods

- **Environments.** τ²-bench (Barres et al., ICML 2026) Retail and Airline.
  Three trajectory blocks: **shared Retail** (1,547 trajectories, 6 models,
  16 tasks, 7 configurations, ≤ 3 reps), **planner-focused Retail** (1,227
  trajectories, 5 models, 24 tasks, 4 conditions), **Airline pilot** (233 of
  240 planned trajectories, 5 models, 6 of 50 tasks, 4 configurations, 2
  reps).
- **Models are all cheap, non-frontier endpoints.** Shared Retail runs
  `claude-haiku`, `deepseek-v4-flash`, `deepseek-v4-pro`, `glm-4-air`,
  `glm-4-flash`, `qwen-turbo`; the planner block runs `deepseek-v4-flash`,
  `doubao-pro`, `kimi-32k`, `minimax-text`, `qwen-turbo`. Pooled Minimal
  oracle success in shared Retail is 0.3787.
- **The Sham construction, exactly.** "Planner Sham shuffles words from the
  domain policy using seed 2701, then repeats or truncates the sequence to
  the Fixed plan's whitespace-word count. Both use a `<task_plan>` wrapper."
  Both arms inject into system context when the agent processes its first
  user message; neither makes a runtime planner call. The archive holds 307
  Fixed and 307 Sham rows, 283 nonempty each; "All nonempty Fixed texts match
  the current task-indexed plan, and all nonempty Fixed/Sham records match its
  whitespace-word count." Appendix D prints a matched pair for Retail task 9
  at **95 whitespace words** per arm — the Fixed text is an eight-step
  `auth -> profile -> order -> ... -> apply` dependency chain, the Sham text
  is genuine policy vocabulary in scrambled order ("is 'delivered', **variant
  order, **cancelled**. 'cancelled', otherwise an - e.g. user to to changed
  The zip ON.'…").
- **Plan provenance is guarded but not established.** "Plan construction is
  documented as manual in the authoring script… The author's full information
  access during construction is unrecorded, so interpreting the result as
  ordinary planning support assumes that the plans used only information
  available before execution." The runtime does check plan text against
  "prohibited field names and selected long fragments of hidden task fields",
  which they call "a limited guard against direct copying."
- **The terminal verifier, exactly.** "The terminal verifier uses the same
  model identifier as the executor and reviews the last at most eight
  user/assistant messages, truncated to 300 characters each. It receives
  dialogue text, with no database access, and has an output limit of 200
  tokens. For episodes with S = 1, an output containing INCOMPLETE sets R = 1.
  The episode and oracle reward are complete before this check; the verifier
  has no repair loop."
- **Estimation.** Paired on (model, task, replication); 90% task-clustered
  percentile bootstrap, 5,000 resamples, seed 2701; Holm adjustment across
  the five model-specific contrasts. Intention-to-treat: episodes stay in
  their assigned arm even when the user ends the dialogue before injection.
- **Sample construction for the primary contrast.** Restrict to tasks flagged
  as requiring mutation (drops task 62, leaves 23), then retain models with
  Minimal verified success ≥ 0.05 on that subset. That keeps four models
  (67 + 69 + 69 + 60 = 265 pairs) and drops `minimax-text`, whose Minimal
  success is zero and whose Fixed–Sham contrast is a degenerate 0.0.
- **Scenario grid.** V ∈ {1, 5, 20} and L ∈ {2, 200} USD, averaged equally
  over contributing models and the three V values. High risk is a
  reference-action-name heuristic (`cancel`, `modify`, `exchange`, `return`,
  `refund`, `update`, `place`, `delete`); the **low-risk stratum contains
  exactly one task**, so the main comparison is the high-risk one.
- **Freeze.** "The experimental package records a 7 July 2026 freeze and a
  13 July 2026 coverage audit." No independently timestamped preregistration.

## Results

- **Primary.** τ̂_P = 0.0717, 90% task-clustered interval [0.0115, 0.1336],
  265 cells. Per model: DeepSeek Flash +13.43 pp [1.49, 24.64], Qwen +13.33
  [5.00, 21.67], Doubao +4.35 [−7.25, 15.94], Kimi **−1.45** [−11.59, 8.70].
  Unadjusted p for Qwen is 0.0172, Holm-adjusted 0.086. "All five adjusted
  values exceed 0.05."
- **Robustness.** Every leave-one-model-out estimate stays positive (5.05 to
  10.20 pp), but "The interval crosses zero after omitting either
  deepseek-v4-flash or qwen-turbo." All-task sensitivity 7.58 pp over 277
  pairs. Exposure-restricted (both arms actually received text) 7.50 pp
  [0.91, 14.23] over 240 pairs. Replication-wise 5.62 / 6.82 / 9.09 pp.
- **Heterogeneity is task-complexity-shaped, from one partition.**
  τ̂_high = 0.1301 (123 cells) vs τ̂_low = 0.0211 (142 cells). Reference-action
  count, a dependency-depth proxy and distinct-tool count "produce the same
  partition of the retained sample", so "this is a single exploratory
  grouping result."
- **Baseline does not predict planning response.** "Qwen has Minimal success
  of 10.17% and a Fixed–Sham effect of +13.33 pp; Kimi has Minimal success of
  11.59% and an effect of −1.45 pp."
- **Fixed also beats Minimal**, 5.80 to 11.59 pp across the four retained
  models — so the gain is not an artifact of the Sham arm being bad.
- **Self-planning is executor-dependent.** Qwen's Self beats Sham by 16.95 pp
  [6.78, 27.12], p = 0.0052, 59 pairs, and its Self–Fixed estimate is +3.39
  [−8.47, 15.25]; "The other retained models have negative Self–Fixed point
  estimates."
- **Verifier rejection, Retail (n = 229).** 83 of 137 oracle-invalid episodes
  rejected (61%); 16 of 92 oracle-correct episodes rejected (17%); of its 99
  rejections, 84% concern invalid outcomes. "Of the 137 oracle-invalid
  episodes, 83 are rejected and 54 are not. Six of those 54 have S = 0 (the
  agent did not assert completion), leaving 48 false passes." The
  counterfactual pre-check rate is (48+83)/229 = 57.21%, actual 48/229 =
  20.96%, so "rejection removes 36.24 percentage points." Minimal's arm rate
  is 58.30%, and the cross-arm gap decomposes 37.34 = 1.09 + 36.24 pp —
  "Almost all of the pooled gap is accounted for by the rejection flags."
- **The false-reject rate more than doubles in the second domain.** "In
  Airline, the verifier rejects 25 of 41 oracle-invalid episodes (61%) and
  seven of 17 oracle-correct episodes (41%). The correct-episode rejection
  fraction is higher than Retail's 17%." Same verifier, same prompt budget;
  the catch rate is identical and the collateral cost is 2.4× worse.
- **Matched contrasts.** All six Retail Verifier–Minimal false-pass
  differences are negative, from −6.25 pp (DeepSeek Pro) to −100 pp (Claude,
  on 10 pairs, "all negative one, producing a degenerate resampling
  interval"). Airline: Claude −70.00, Qwen −91.67, DeepSeek Flash and GLM Air
  −25.00 (intervals crossing zero), Doubao **+10.00**.
- **Pooled configuration table (shared Retail).** Planner Fixed has the best
  verified success (0.4810), the best Cost@Success (0.0280) and the best
  Pass@Budget (0.5185). Verifier-only: 0.4017 verified, 0.2096 false pass,
  0.0164 mean cost. **Full Fixed: 0.3592 verified — below Minimal's 0.3787 —
  with 0.1831 false pass and 0.0739 mean cost, "the largest configuration
  mean."** For `deepseek-v4-flash`, mean tokens run 73,409.7 (Minimal),
  75,102.9 (verifier-only), 250,745.1 (Full Fixed).
- **The scenario inversion, with components.** High-risk stratum,
  Full Fixed: ΔP = −0.0411, ΔFP^avoid = 0.4982, ΔC = 0.0947. Verifier-only:
  ΔP = +0.0655, ΔFP^avoid = 0.4838, ΔC = 0.0079. Mean net value vs Minimal:
  Fixed $1.97 (L=2) → $35.13 (L=200); Verifier-only $1.53 → $97.31; Full
  Fixed $0.55 → $99.20. Figure 3(c) annotates the crossings: "Verifier-only
  passes Fixed at L ≈ $3.4" and "Full Fixed passes Verifier-only at L ≈ $70"
  — read off a log-scale chart from two frozen grid points, and the caption
  calls them "descriptive."
- **Break-even liabilities.** Planner Fixed and Verifier-only have negative
  thresholds at all three V (positive value even at L = 0). Full Fixed needs
  L ≥ $0.27 / $0.60 / $1.84 at V = 1 / 5 / 20, because its success difference
  is negative. "Every aggregate break-even threshold lies below the grid's
  lower bound of L = 2."
- **A capability-conditioned diminishing return, with no precision.** The
  Failure-Detection-Sensitivity × Verifier interaction is β = 0.1825 on false
  pass across 1,547 trajectories: "the positive coefficient associates higher
  measured FDS with a smaller additional reduction under verification." The
  model-clustered SE is 0.0460 (p = 7.2e−5) but the delete-one-model
  jackknife gives SE 0.2803 and **p = 0.5439**. Six clusters; the sign is
  stable, the magnitude is not.
- **The capability probes mostly failed.** Planning Frontier split-half
  reliability 0.968; Failure Detection Sensitivity 0.761 with split range
  [−0.739, 0.931]; Tool Coordination Complexity rises from −2.116 to 0.349
  after stress items "with continued score saturation." Neither
  trajectory-derived proxies nor independent probes predict configuration
  benefit: "Trajectory-derived proxies worsen all three metrics. Independent
  probes change R2 by +0.002, AUC by +0.010, and Brier by −0.0005."

## Critique / open questions

- **The headline interval nearly touches zero and the design is not powered
  for it.** [0.0115, 0.1336] at 90% is a wide interval whose lower bound is
  1.15 pp. Two of four leave-one-out deletions push it across zero, all five
  Holm-adjusted p-values exceed 0.05, and the paper's own power simulation
  says so: "A separate two-arm design simulation reports rejection rates of
  0.14 for 23 tasks and 0.18 for a hypothetical 46 tasks." A 14% rejection
  rate at the design's own task count means the primary result is the kind of
  finding this design finds about one time in seven when the effect is real.
  The direction is credible; the magnitude is not estimated to a useful
  precision. Any downstream use should carry the interval, not the 7.17.
- **The Sham control is cleaner than usual and cheaper than it looks.** It
  is a *word salad*, not a plausible-but-useless plan, and the authors say so
  unprompted. That rules out "context length did it" and "the wrapper did
  it", but it does not rule out "any coherent text in that slot did it."
  The stronger control — a coherent, well-formed, task-*irrelevant* plan —
  is not run. Tokenizer length also differs between arms by construction.
  Read narrowly, the finding is: shuffled policy vocabulary is inert, and
  correct dependency-ordered guidance is not.
- **Plan provenance is the load-bearing unrecorded variable.** The plans were
  hand-written by an author with unrecorded access to the task set, and the
  guard is substring matching on hidden field names. A plan that encodes the
  reference action sequence is not "planning support"; it is a partial oracle
  leak. The paper flags this and does not resolve it. The task-9 example
  printed in Appendix D reads exactly like a reference-action chain.
- **The verifier is soft evidence by construction.** It reads the *dialogue*
  — the agent's own account — truncated to 300 characters per message, with
  no database access, using the executor's own model. Under
  [[concepts/evidence-gated-completion]]'s hard/soft criterion this is the
  front door that concept warns about, and it still removed 36.24 pp of false
  passes. That is a genuinely surprising positive for a cheap soft gate, and
  also a ceiling: 54 of 137 invalid episodes survive it, and its correct-work
  rejection rate is domain-dependent (17% Retail, 41% Airline).
- **Nothing downstream consumes the rejection.** "The episode and oracle
  reward are complete before this check; the verifier has no repair loop."
  The stateful point the introduction makes — "a terminal rejection occurs
  after execution and may leave earlier refunds, cancellations, or other
  state changes in place" — is stated and then never measured. The paper
  concedes the gap: "Following rejected cases through repair, escalation, or
  abandonment would measure both the benefit of withholding invalid results
  and the cost of withholding valid ones."
- **The liability analysis prices only inference tokens.** "Plan authoring,
  integration, maintenance, human review, and the opportunity cost of
  withholding correct outcomes require additional accounting." The last item
  is the one that matters most: the 17%/41% withheld-correct episodes appear
  in the outcome table and **nowhere in NV**. The framework's own asymmetry
  weight L covers false passes only; there is no term for a false reject.
  So a decision rule read off this paper still cannot be pushed to
  block-everything only because ΔP happens to be positive here.
- **The V and L grids are asserted, not derived.** {1, 5, 20} × {2, 200} USD
  with no source. The crossing points ($3.4, $70) are therefore a
  demonstration that ordering depends on the ratio, not a calibrated
  threshold anyone should import.
- **The risk stratification is nearly degenerate.** "The low-risk group
  contains only task 62." A two-stratum analysis where one stratum is a
  single task is a one-stratum analysis with a caveat.
- **Pooled table comparisons are unbalanced and not paired.** n runs 142 to
  237 across configurations, and the authors say "Samples are unbalanced;
  comparisons are not paired." Full Fixed's apparent verified-success deficit
  vs Minimal (0.3592 vs 0.3787) is from that table; the paired high-risk
  ΔP = −0.0411 is the better-supported version of the same sign.
- **Every model is a budget endpoint.** Haiku, flash/turbo/air tiers, a 32k
  Kimi, minimax-text. The Minimal baselines are 0% to 60.3%. Whether
  prewritten plans help a frontier agent that already plans well is exactly
  the question, and no frontier model is in the study. The FDS interaction —
  more capable failure-detectors gain less from an external verifier — is the
  only evidence pointing at the answer, and it has a jackknife p of 0.54.
- **No code, no peer review, no citations.** "The research archive contains
  the study trajectories, analysis scripts, plans, and diagnostics; a public
  repository link is not yet available at the time of this version." Against
  that: benchmark familiarity is disclosed ("participating models may have
  encountered these tasks during training"), the missing preregistration is
  disclosed, and a failed Claude billing batch is reported and excluded
  rather than quietly retried.
- **A genuinely strong reporting habit worth stealing.** The paper zeroes the
  term it thinks is not causally attributable (the verifier's ΔP) and reports
  that the ordering is unchanged. Most papers would keep the favorable term.

## Trust signals

- **Credibility:** 2. An unreviewed arXiv cs.AI preprint from three authors
  (CUHK, Edinburgh, CUHK-Shenzhen) with no track record in this graph, no
  released code or data, no citations, and no independently timestamped
  preregistration. The primary result fails multiple-comparison correction
  at every model, its interval nearly touches zero, and the design's own
  power simulation puts rejection at 0.14. All six evaluated models are
  budget tiers. Held above 1 by unusual candor: a stated placebo arm with a
  null it does not spin, a printed matched Fixed/Sham example, leave-one-out
  and exposure-restricted sensitivities that weaken the headline and are
  reported anyway, an explicit refusal to attribute the verifier's ΔP to the
  verifier, an explicit statement that plan-authoring information access is
  unrecorded, and a jackknife that demolishes its own FDS interaction's
  precision.

## Follow-up

- **Relevance:** 4. It answers a standing open question in
  [[concepts/refusal-cost-symmetry]] with a form rather than an anecdote, and
  it is the closest thing in the graph to what
  [[concepts/evidence-gated-completion]] has been holding out for. It is a 4
  and not a 5 because the domain is customer-service tool use rather than ML
  research, the models are all budget tiers, and no number here is precise
  enough to act on directly.
- **[[concepts/refusal-cost-symmetry]] — the asymmetric weighting, stated as
  a form.** That concept's first open question is "Is the symmetric penalty
  the right weighting? … nobody has proposed a principled ratio."
  [[literature/papers/taneja2026scan]] "proposes a form, not a ratio." This
  paper proposes a form *and* demonstrates that the ratio reorders the
  answer: NV = V·ΔP^VS + L·ΔFP^avoid − ΔC with break-even
  L* = (ΔC − V·ΔP^VS)/ΔFP^avoid, and a measured inversion — the standalone
  verifier overtakes the planner at L ≈ $3.4 and the full stack overtakes the
  verifier at L ≈ $70. The catch/false-reject pair (61%/17% Retail) is also
  the cleanest single instance of this concept's core demand. The caveat
  cuts the same way: NV has **no term for the withheld correct work**, so the
  framework is symmetric in name and one-sided in arithmetic, and the paper
  says the opportunity cost of withholding "require[s] additional
  accounting."
- **[[concepts/refusal-cost-symmetry]] — the false-reject rate is not a
  property of the gate.** The same verifier, same prompt, same truncation
  budget rejects 17% of correct Retail episodes and **41%** of correct
  Airline episodes while holding a 61% catch rate in both. Any imported
  false-positive number is a number about a domain, not about a checker.
  This is the deterministic-analogue open question's near neighbour: not a
  model-judgment inversion, but a fixed cue transplanted into a domain where
  it over-flags.
- **[[concepts/evidence-gated-completion]] — the held open question,
  partially discharged.** The concept asks for "a completion gate inside a
  harness with a measured false-rejection rate." This is one: live on 229
  Retail and 58 Airline episodes, in-loop, paying $0.0079/episode, with a
  cross-tabulated false-reject rate. It is closer than
  [[literature/papers/ning2026scores]] (post-hoc, ~$60/audit, no
  false-rejection rate) or [[literature/papers/zheng2026benchshield]]
  (reference-solution and replayed cells). It falls short in three specific
  ways that should be written into the concept rather than glossed:
  (1) it is a **soft-evidence** gate — dialogue only, no external state,
  executor's own model — which contradicts implementation guidance #3, and
  it worked anyway (57.21% → 20.96%); (2) there is **no behavioural delta**,
  because rejection triggers nothing: "the verifier has no repair loop", and
  the before/after is computed by setting R = 0 on the *same* executions, not
  by comparing two live arms; (3) it is unreleased. The honest summary is
  that it prices the *acceptance margin* of a gate, not the *effect* of one.
- **[[concepts/evidence-gated-completion]] — a soft gate is worth more than
  zero.** The concept's hard/soft partition implies a dialogue-reading
  verifier admits soft evidence "through the front door." This measures what
  that buys: 61% of invalid episodes caught for less than a cent. It does not
  overturn the partition — 54 of 137 invalid episodes survive, and the
  correct-rejection rate is unstable across domains — but "soft evidence is
  not gradable" is now a claim with a counterexample, and the right framing is
  a cheap soft prefilter in front of a hard check rather than in place of one.
- **[[concepts/constraint-pinning]] — the complement to "presence is not
  sufficiency."** That section uses
  [[literature/papers/lavrenko2026instruction]], which holds content fixed and
  varies **repetition**. Sham is the other half of the 2×2: hold presence,
  word count and wrapper fixed and vary **content**. Both halves now say the
  same thing from opposite directions — the text's presence in context is not
  the mechanism. Sham − Minimal = +1.89 pp [−3.45, +7.09] is the direct
  measurement that ~95 words of inert instruction-shaped text in the system
  context is approximately free, which is what a repo full of declared-but-
  unresolving `@import` lines would predict.
- **[[concepts/hce-evaluation]] — a matched-null control template.** The
  concept already holds "Hold the evidence fixed and vary only the framing"
  ([[literature/papers/tripathi2026diagnostic]]) and the scaffold-ownership
  audit ([[literature/papers/zhang2026double]], also on τ-bench). Sham is a
  third member of that family: **match the intervention on every
  surface property it is not supposed to be working through**, and report the
  matched-null arm's contrast against no-intervention as a separate number.
  Cheap, and this repo has no equivalent for any of its own instruction
  artifacts.
- **Convergence and independence.** The false-pass/terminal-acceptance split
  arrives independently from Advani (2026, arXiv:2606.09863, false success on
  τ²-bench and AppWorld), Chen et al. (2026, VIGIL / arXiv:2605.08747, world
  completion vs self-termination), Cao et al. (2026, arXiv:2603.03116,
  "corrupt success"), and Rosset et al. (2026, COLM, computer-use trajectory
  judges) — none of which are in this graph, and the first three are the
  obvious digest candidates. Sah et al. (2026, "The Verifier Tax", ACM
  Conference on AI and Agentic Systems) covers the same two τ²-bench domains
  with a safety–success tradeoff framing and is the closest external
  comparison to this paper's scenario analysis. The placebo-arm methodology
  is the same instinct as
  [[literature/papers/shen2026what]]'s equal-length non-gold placebo.
