---
kind: paper
title: "Where Should a Stuck Agent Spend Its Next Tokens? Causal Attribution of Inference-Time Resources"
authors: ["Saiyue Lyu", "Jerrold Huang", "Yunbei Zhang", "Janet Wang", "Zhenting Wang", "Yingqiang Ge", "Haibo Zhang"]
institutions: ["University of British Columbia", "University of Toronto", "Tulane University", "Rutgers University", "Kyushu Institute of Technology"]
year: 2026
venue: "arXiv 2610.28834 (cs.SE), v1 2026-10-20; footer: NeurIPS 2026 Workshop on Long-Context Foundation Models"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.28834"
code_url: null
citations: null
source: "raw/papers/lyu2026where.pdf"
added: "2026-10-27"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
tags: ["inference-time-resources", "banzhaf-value", "shapley-value", "factorial-design", "frozen-checkpoint", "counterfactual-forks", "self-repair", "execution-feedback", "feedback-fixation", "specification", "retrieval", "self-allocation", "allocation-regret", "oracle-instrument", "placebo", "pre-registration", "swe-bench-verified", "evalplus", "redirect", "capability-ladder"]
---

# Where Should a Stuck Agent Spend Its Next Tokens? Causal Attribution of Inference-Time Resources

## TL;DR

The authors freeze a failed coding attempt and fork it into all 8 cells of a
2×2×2 factorial over three resources: **computation** (one extra
analysis-only planning call, 2048 tokens, no tools), **information** (a
task-level packet that never reads the artifact) and **feedback** (an
execution report on public tests). Every cell then gets one identical final
revision, scored on held-out tests. Because all 8 bundles are observed per
checkpoint, each resource's Banzhaf value (its average main effect) and the
pairwise interactions are computed exactly. They run 6 repair models on 194
function-level failures (MBPP+/HumanEval+, all from Qwen3-4B) and 126
repository-level failures (SWE-bench Verified, all from a Qwen3-14B
mini-SWE-agent), k = 3 rollouts per cell (k = 5 for two models at function
level).

Three results matter for this graph:

1. **The input that binds is a statement of what correct is, and it is an
   oracle instrument.** The "specification" packet is written by Kimi K2.5
   *from the reference solution or reference patch*, then audited for
   literal leakage. It is worth 0.064–0.136 held-out utility at function
   level and 0.048–0.103 on repositories. A length-matched placebo is worth
   nothing. The information a deployed agent actually gets, five BM25 code
   chunks, is worth −0.002 to 0.009 on repositories. Public-test feedback is
   worth 0.007–0.033 and barely grows with detail.
2. **Extra planning stops paying as models get stronger.** The planning
   call is worth 0.131 to Qwen3-4B and 0.007–0.010 to GPT-5.6 and Sonnet 4.5
   (CIs include zero), and this survives a 24-checkpoint matched-difficulty
   subset. All models run with hidden reasoning off.
3. **Asked to pick one resource, three of four probed models pick feedback
   95–100% of the time and do worse than a uniform random pick.** At
   function level the gap to random is 0.029–0.064 (p < 0.001) for
   Qwen3-14B, DeepSeek-V3.2 and Sonnet 4.5. GPT-5.6 is not worse than
   random. On repositories only DeepSeek-V3.2 is significantly worse. A
   fixed "always ask for information" rule beats every model's own choice.

The probe's menu includes the oracle-derived specification as one option.
"Worse than random" therefore mostly means "chose the oracle option less
than a third of the time". The whole design is one revision step, so
feedback's value as a stop or keep signal inside a loop is excluded by
construction.

## Claims

- Inference-time resources for agents differ in kind, not only in amount.
  End-to-end pipeline comparisons cannot isolate a resource, because the
  resource, the state and the next edit vary together along a trajectory.
- A frozen-state 2³ factorial with a common final revision identifies every
  main effect and interaction "without assumptions". AME_X equals the
  Banzhaf value; Γ_XY equals the Banzhaf (and, for n = 3, Shapley)
  interaction index (Proposition 1, proofs in App. C).
- Information "about what the code must do" is the binding resource at a
  failed step, including at the frontier.
- Scaling "moves inference bottlenecks rather than removing them": from
  computation to information.
- The three resources are substitutes, so "the first choice matters most".
  The pre-registered complementarity hypothesis (planning raises the value
  of information and feedback) is rejected.
- Tokens are not a common currency: which resources are in a cell predicts
  its outcome better than the cell's total token cost.
- Models show "feedback fixation" and misallocate. The oracle-to-own gap
  (0.05–0.22 function level, 0.08–0.17 repository) bounds what a learned
  allocator could recover.
- Artifacts (frozen checkpoints, per-cell outcomes) are "released"
  (contribution 1). No URL is given, and the ethics statement calls them
  "possible releasing artifacts".

## Methods

- **Checkpoints.**
  - Function level: every failed first attempt of Qwen3-4B in a "quick
    draft" style (one call, 2048 tokens, T = 0.7) on MBPP+ v0.2.0 and
    HumanEval+ v0.1.10. N = 194 (131 + 63), mean U = 0.32. U is the
    fraction of EvalPlus extended tests passed. Public tests are the
    original assertions.
  - Repository level: a stratified 250-instance frame of SWE-bench
    Verified; a text-based mini-SWE-agent with Qwen3-14B (75 steps, T = 0)
    left 126 unresolved, non-degenerate patches (62 django, 21 sympy, 43
    other). In 57 the patch breaks the test run. U is the fraction of
    held-out FAIL_TO_PASS tests passed, and 0 on any PASS_TO_PASS
    regression. 33 instances with ≥2 target tests use a split-test
    protocol: one failing target test is shown as feedback, U scores the
    rest.
  - Native-failure panel: 30 checkpoints DeepSeek-V3.2 itself failed as the
    agent.
- **Repair models.** Qwen3-4B/14B/32B (within-family ladder), DeepSeek-V3.2,
  GPT-5.6, Claude Sonnet 4.5. All without hidden reasoning, T = 0.7.
  Qwen3-14B's thinking mode is a robustness arm.
- **Resources.**
  - C: one analysis-only call (2048 tokens main; dose 256–4096) that reads
    the task, artifact and the cell's other packets and returns a diagnosis
    and plan in prose. No tools.
  - I: generated once per task, identical across checkpoints. Rungs:
    retrieval (BM25 over the repo at base commit, ≤12,000 chars);
    location (files and functions the reference fix touches, extracted
    mechanically); behaviour (root cause and required behaviour written
    from the reference fix, no names); specification (location +
    behaviour; at function level an input–output contract and edge cases,
    80–140 words). Audits reject code fences, any reference line ≥12
    chars, test literals ≥6 chars and test identifiers. All 126 packets of
    each rung pass. Main runs use specification at function level and
    retrieval on repositories.
  - F: an execution report on public tests. Levels F1 pass counts, F2
    failing inputs, F3 + expected outputs (main, ≤5 cases), F4 + observed
    outputs.
- **Revision.** One call, same template and output limit across cells
  (2048 tokens function level, 4096 repository, as ≤6 search/replace
  blocks with a fixed fuzzy-match fallback order, App. E). Truncated
  revisions are scored, not retried.
- **Inference.** Contrasts per checkpoint from k-rollout cell means,
  averaged over checkpoints, 2000-resample checkpoint bootstrap; Holm
  correction across the three pairwise interactions only. Labels use a
  pre-fixed τ = 0.2 and ceiling 0.8. A canary test asserts per cell that
  held-out inputs, outputs and references never enter a prompt.
  Definitions, contrasts and manifests were hashed before outcomes were
  read, with an append-only deviations ledger (App. F, 8 entries).
- **Self-allocation probe.** From the empty cell's context the model picks
  one of C (a planning pass), I ("a specification of the required
  behaviour"), F (the execution report) or nothing. 3 rollouts per
  checkpoint, 4 models (Qwen3-14B, DeepSeek-V3.2, GPT-5.6, Sonnet 4.5).
  The pick is scored on that checkpoint's own measured surface against
  always-X, a uniform random pick among C/I/F, and the per-checkpoint
  oracle. Option order is permuted in a robustness arm. The probe prompt
  text is not in the PDF.
- **Compute.** Qwen3-4B/14B local on 3× A100 40 GB; everything else via
  API. A function-level panel takes 1–2 h wall clock; a repository arm
  2–9 h, dominated by harness evaluation.

## Results

- **Function-level main effects (Table 4, N = 194).**

  | Model | q000 | q111 | AME_C | AME_I | AME_F | Het. |
  |---|---|---|---|---|---|---|
  | Qwen3-4B | 0.338 | 0.597 | 0.131 [0.097, 0.164] | 0.108 | 0.030 | 0.30 |
  | Qwen3-14B | 0.448 | 0.730 | 0.110 | 0.136 [0.105, 0.170] | 0.026 | 0.45 |
  | Qwen3-32B | 0.535 | 0.816 | 0.101 | 0.129 | 0.033 | 0.39 |
  | DeepSeek-V3.2 | 0.732 | 0.889 | 0.053 | 0.086 | 0.012 | 0.34 |
  | GPT-5.6 | 0.813 | 0.909 | 0.010 [−0.006, 0.026] | 0.064 [0.038, 0.091] | 0.019 | 0.29 |
  | Sonnet 4.5 | 0.817 | 0.898 | 0.007 [−0.010, 0.025] | 0.067 [0.043, 0.092] | 0.007 | 0.28 |

  "Het." is the share of checkpoints whose best single resource is not the
  population's best. Information's share of summed main effects rises from
  40% to 83%; computation's falls from 49% to 8%. Shapley and Banzhaf values
  differ by ≤0.006 and rank identically (Table 2).
- **Ceiling.** Degenerate checkpoints (q000 ≥ 0.8) are 20% for Qwen3-4B and
  74% for GPT-5.6 and Sonnet 4.5. On the 24 checkpoints all six models fail
  badly, information is worth 0.12–0.25 for every model and computation
  0.03 (n.s.) for GPT-5.6 and Sonnet. The authors note the subset is
  selected on q000, so effect sizes there are inflated.
- **Placebo (Table 7, two models only).** Placebo − nothing: −0.016
  (Qwen3-14B), −0.002 (DeepSeek). Specification − placebo: 0.195 and 0.126.
- **Doses (40 hardest checkpoints, DeepSeek, k = 5).** 256 planning tokens
  give 61% of the 2048-token gain and 4096 add nothing. Feedback-only rises
  from 0.362 (F1) to 0.402 (F4) against an empty cell of 0.298. Most of
  feedback's gain is in the first level (pass counts, +0.064). On
  repositories, 4096 or 8192 planning tokens and F4 output add nothing
  detectable.
- **Interactions.** 21 of 36 pairwise interactions (6 models × 3 pairs × 2
  samples) are significantly negative after Holm, none significantly
  positive. Away from the ceiling Γ_CI is −0.10 to −0.14. For GPT-5.6 and
  Sonnet the information-only cell is within 0.01 of the full bundle.
  Removing feedback from the full bundle costs nothing for any model
  (N_F −0.011 to 0.014). Feedback alone unlocks at most 1.5% of
  checkpoints for any model.
- **Repository level (Table 1, N = 126, k = 3).** With retrieval as I, no
  main effect is significant for any model. Swapping in the specification:
  AME_I 0.048 (Qwen3-14B), 0.083 (DeepSeek), 0.103 (GPT-5.6), 0.088
  (Sonnet), all p ≤ 0.002; q111 − q000 goes from 0.01–0.04 to 0.07–0.12.
  Computation is significant only for the two weaker models (0.034); feedback
  is nil throughout. Location alone is worth −0.03 to +0.02, behaviour alone
  0.013–0.047, both together the whole effect. The complementarity term is
  positive for all four models, with CI excluding zero for three. The native
  panel (30 checkpoints) shows the same pattern: specification 0.079,
  retrieval −0.004.
- **Self-allocation probe (Fig. 4, Sec. 5.5, App. H.3).**
  - Qwen3-14B, DeepSeek-V3.2 and Sonnet 4.5 request feedback in 95–100% of
    checkpoints on both families. Under permuted option orders this is
    63–100% (DeepSeek 63–99%). GPT-5.6 asks for information 49% of the time
    at function level and feedback 71% on repositories.
  - Function level: the three fixated models realise 10–27% of the oracle
    gain, trail always-information by 0.078–0.101, and trail random by
    0.029–0.064 (p < 0.001). GPT-5.6: 0.071 against random 0.050 (n.s.).
  - Repository level: always-information beats own choice for all four
    models (significant for three). Every model is below random in point
    estimate; only DeepSeek significantly.
- **Cost and allocation (Table 3).** Per 1000 added tokens, information
  returns 0.54–1.25 at function level and 0.11–0.31 on repositories, 4–20×
  computation's return. Cross-validated R² for predicting a checkpoint's
  eight cell means is 0.007–0.111 from token cost and 0.023–0.158 from the
  resource indicators. Both explain little. Greedy allocation by return
  reaches 0.80–0.99 of the optimal affordable bundle's gain.
- **Robustness.** The binary all-pass outcome keeps the ordering. Qwen3-14B
  thinking mode as C gives AME_C 0.120, level with information (0.116);
  95 of 2328 thinking cells exhausted the budget and scored 0. Keeping only
  clean-edit checkpoints raises the specification effect to 0.131 (GPT-5.6)
  and 0.145 (Sonnet).

## Critique / open questions

- **Against the digest framing: below framing (mild).** The numbers in the
  digest summary are all in the PDF and are quoted correctly. Four things
  the digest left out change what the paper supports.
  1. **"Task specs help" means oracle specs.** The specification is written
     from the reference solution or patch. The authors say plainly that
     rungs above retrieval "establish what information would do, not what
     a system can obtain". The deployable rung, retrieval, is worth
     nothing on repositories. So the paper shows that the missing input is
     knowledge of the target. It does not show any way an agent can get
     that knowledge.
  2. **"Worse than random" is measured against a menu with an oracle in
     it.** In the probe, I is the same oracle-derived specification. A
     uniform random pick takes it a third of the time. Since I dominates
     and F is near zero, "below random" is close to "picked I less than a
     third of the time". The paper does not show the probe prompt. A model
     told it may have "a specification of the required behaviour" could
     reasonably expect a restatement of the task it already has. That
     would be a correct prior about deployed systems. The result shows
     that the models' preference does not track this surface. It does not
     show that the preference is irrational.
  3. **Significance is narrower than "three of four".** Below random is
     significant at function level for three models. On repositories only
     DeepSeek-V3.2 is. GPT-5.6 is never significantly below random. The
     "four frontier models" include Qwen3-14B. That is the paper's own
     wording.
  4. **It is one revision step.** The limitations section excludes
     feedback's role "as a verifier or termination signal over a
     trajectory" by design. Feedback fixation may be well calibrated for
     multi-step loops, where a test run also tells the agent when to stop.
     The digest's "Why" says it is task level. It is also single-step,
     which matters more for a research loop.
- **The digest's "what a redirect should supply" does not transfer
  directly.** The paper's answer is "a specification of what correct is",
  derived from ground truth. In an `/iterate` chain the analogue of that
  specification is the target metric and hypothesis, which the agent
  already has. No deployable input in the paper (retrieval, a plan, test
  output) helped a frontier model on repositories. Our reading: the paper
  supports taking the *choice* away from the agent and gives no positive
  guidance on what a research-loop pivot should contain.
- **The capability ladder is confounded with the checkpoint source.** All
  function-level checkpoints are Qwen3-4B quick-draft failures. Strong
  models start at q000 ≈ 0.81 on them, and 74% are degenerate for them. The
  24-checkpoint matched subset and the 30-checkpoint native panel address
  this, but both are small and the first is selected on q000. The
  computation channel is a single prose plan with hidden reasoning
  disabled. "Computation is worth nothing to strong models" means this
  planning call is worth nothing when reasoning is off. Thinking mode was
  tested only on Qwen3-14B, where it matched information.
- **Post hoc arms.** Deviation 7: the repository oracle arm and the
  information dose ladder were added *after* the null retrieval result. The
  Qwen ladder, native panel and self-allocation probe were run as a
  "second stage". All are labelled as such, but the headline repository
  result and the probe are not covered by the pre-registration.
- **Rollout noise is not separated from checkpoint variance.** k = 3 per
  cell at T = 0.7, and CIs come from resampling checkpoints only. That is
  the right unit for population claims, but per-checkpoint labels
  (τ = 0.2 on a 3-rollout mean) and the 28–45% heterogeneity figure
  include revision noise. Part of the oracle-to-own gap is therefore the
  oracle picking the best cell after the noise is seen. This is the same
  selection effect [[concepts/pass-at-k]] warns about for best@k.
- **The placebo covers two models, and only at function level.** No placebo
  arm runs on repositories, where the specification effect carries the
  headline.
- **Fit with the rest of the graph.**
  - [[literature/papers/sun2026scipivot]]: frontier models shown the
    correct pivot kept their own plan 61–71% of the time. This paper shows
    the same kind of habit at the level of the next input. Both measure a
    one-shot choice, not an outcome inside a loop.
  - [[literature/papers/zhang2026judged]]: agents recognise a failed
    observation but do not act on it, and a masked action at k works where
    showing the count does not. Here a fixed rule beats the agent's pick.
    Both are evidence for moving the choice into the controller.
  - [[literature/papers/park2026when]] found sign-only gating about equal
    to full numeric feedback. Here F1 (pass counts) carries most of
    feedback's small value, and F2–F4 add ≤0.02 each. That agrees in
    direction. [[literature/papers/lumer2026reflective]] found that
    reason-bearing feedback transfers and a scalar does not. That is not a
    conflict: lumer's useful feedback names what was wrong with a citation,
    which is closer to "what correct is" than a failing test is. That
    reading is ours.
  - [[literature/papers/zhang2026veriharness]]: an agentic selector
    captured 28–30% of the oracle gap. Here fixated models capture 10–27%
    of the oracle gain, but for a choice of *input*, not of output.

## Trust signals

- **Credibility:** 3.
  - For:
    - A clean identification argument. The full factorial at a frozen
      state is the smallest design in which every interaction is
      estimable, and the game-theory identities are proved.
    - Definitions, contrasts and manifests hashed before outcomes were
      read, with an append-only deviations ledger that discloses post hoc
      arms.
    - A length-matched placebo, dose ladders for all three resources,
      leakage audits on every packet, and a per-cell canary for held-out
      leakage. Held-out tests are disjoint from feedback tests.
    - Six models including a within-family ladder; two task families; a
      native-failure panel; binary-outcome and edit-interface sensitivity
      checks.
    - A pre-registered hypothesis that the data rejected is reported as
      rejected.
    - The limitations section names the oracle rungs, the single step and
      the weak generators.
  - Against:
    - A NeurIPS workshop paper (Long-Context Foundation Models, an odd fit
      for the topic), not peer reviewed at main-track level. No citations
      yet (7 days old).
    - No code or data URL in the PDF, despite "released" in the
      contributions.
    - The headline repository result and the probe are post hoc or
      second-stage. k = 3 per cell. The probe prompt is not shown.
    - Every checkpoint comes from one weak generator per family.
    - Hidden reasoning is off for every model, including the frontier
      ones.
  - Not 4: workshop venue, no artifacts, and the two results most likely
    to be quoted (the repository specification effect and below-random
    choice) are outside the frozen plan and rest on an oracle menu. Not 2:
    the design and its disclosures are unusually careful, five
    institutions are named, and the main function-level effects have
    tight checkpoint-bootstrap CIs on 194 states.

## Follow-up

- **Relevance:** 3. Task-level, single-step code repair, not a research
  loop. It adds material evidence to [[concepts/budget-as-ceiling]]: a
  stuck agent's own pick of its next resource is a fixed habit that loses
  to a fixed rule, and equal tokens do not buy equal value. It also adds to
  [[concepts/programmable-evaluator-oracle]] (a pass/fail report says the
  attempt is wrong, not what right is, and extra failure detail adds
  little) and [[concepts/pass-at-k]] (an oracle-gap capture ratio for
  input choice). Not 4: the binding resource is an oracle instrument, there
  is no loop, and there is no redirect or halt arm.
- **For the redirect-before-halt proposal.** This is weak support for its
  controller-side framing, and it is the third source after
  [[literature/papers/yan2026traceml]] and
  [[literature/papers/sun2026scipivot]] to find that an agent's own choice
  of next move is habitual. It does not touch the halt point, the trigger
  or the axis list. Its "what to supply" answer does not transfer to a
  research loop, because the useful input is ground truth. It does not
  change the proposal's recommendation.
- **Cheap local check (optional).** The same frozen-state fork applied to
  one stalled `/iterate` cycle would measure what this paper cannot: a
  forced axis change against one more refinement from the same state, at
  equal tokens. It needs a logged stall first, and the proposal notes that
  none exists on this box yet.
- **Candidates** (cited, not yet in the graph):
  - Feng et al. 2026, "Scores are not decisions: cost-aware stopping for
    tool acquisition in LLM agents" (arXiv 2607.27083). Treats resource
    acquisition with a stop decision, the loop half this paper excludes.
  - Yuan, Su & Yao 2026, "Diagnosing retrieval vs. utilization bottlenecks
    in LLM agent memory" (arXiv 2603.02473). A factorial within one
    channel; relevant to [[concepts/selective-memory-retrieval]].
  - Wu et al. 2026, "EcoAgent-Bench: evaluating economic decision-making in
    budget-constrained LLM agents" (arXiv 2608.05519). Escalation and
    spend decisions.
  - Olausson et al. 2024, "Is self-repair a silver bullet for code
    generation?" (ICLR 2024). Self-repair gains are modest at matched cost
    and bottlenecked by feedback quality.
  - Zhu et al. 2025, "Scaling test-time compute for LLM agents" (arXiv
    2506.12928). End-to-end ablations of agent test-time strategies.
  - Already in the graph and cited here: [[literature/papers/fei2026how]]
    (failure taxonomy on 100 autoresearch tasks).
