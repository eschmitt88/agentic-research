---
kind: paper
title: "Learning What to Investigate Next: Meta-Reasoning for Long-Horizon Research Agents"
authors: ["Ankur Samanta", "Yonathan Efroni", "Paul Sajda", "Kaveh Hassani", "Anirudh Goyal"]
institutions: ["Meta AI", "Columbia University", "Tel Aviv University"]
year: 2026
venue: "arXiv 2610.02525 (cs.AI), v1 2026-10-01"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.02525"
code_url: null
citations: null
source: "raw/papers/samanta2026learning.pdf"
added: "2026-10-06"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["meta-reasoning", "outer-loop", "work-order", "fresh-executor", "handover", "git-substrate", "decision-level-credit", "generative-critic", "value-forecasting", "actor-critic", "proxy-vs-gold", "strategic-inertia", "stopping", "autoresearch", "theorem-proving", "architecture-search"]
---

# Learning What to Investigate Next: Meta-Reasoning for Long-Horizon Research Agents

## TL;DR

MIRA splits a research agent into two parts:

- an **outer-loop meta-reasoner** that, at each decision, reads a curated
  slice of a persistent Git repository and writes a **work order** (objective,
  approach, acceptance criteria, stopping condition), or chooses to terminate;
- a **fresh Codex executor** per work order, which does the work, commits
  results and leaves a **handover**.

Both contexts reset at each decision, and only the repo survives. That turns
each investigation, however long, into one RL transition at the decision
level. On top of it the authors train (a) a **decision-level generative
critic** that forecasts remaining return as a next-token distribution over
11 bins, and (b) **MIRA-AC**, a shared LoRA actor–critic on Qwen3.6-27B whose
policy loss touches only the meta-reasoning tokens (25.8% of output tokens).

For this graph the architecture is the main point. It is
`hierarchical-delegation` + `file-as-bus` + per-decision context curation,
the same shape as `/iterate`, from a major lab with a same-model,
same-harness comparison. The learned parts need 32 H200s and are evaluated
on the same task identities they were trained on. The training results are
thinner than the abstract suggests.

## Claims

- **Untrained MIRA improves long-horizon inference.** On IMOProofBench
  Advanced with GPT-5.5 it raises the mean grader score "from 67.1% under
  direct inference to 100%, solving all 30 problems in each of three
  trials." It also produces the best architectures in matched 24 h (RMT) and
  48 h (Loop Transformer) campaigns.
- **The gain is not only extra tokens.** Under output-token-matched forced
  continuation the baselines recover "most of the performance gap", with a
  residual concentrated on four problems. MIRA also pivots more: strategic
  inertia is 0.958 (direct), 0.862 (Codex), 0.772 (MIRA).
- **Decision-level value forecasting beats token-level.** The step-generative
  critic gets held-out RMSE 0.214 vs 0.324 for the token-level generative
  critic ("an absolute 0.110").
- **Cross-environment pretraining is a transferable prior.** Leave-one-out
  initialization reduces RMSE by 0.053 vs target-only after 5 support
  episodes.
- **MIRA-AC trained on proxy feedback improves gold outcomes "across all
  four" environments.** The actor also transfers to BNLearn, which was held
  out from actor training (but not from critic pretraining).
- **Training changes research behavior.** An LLM judge credits MIRA-AC's
  outer decisions in 203 of 400 pairs vs 131 for untrained MIRA. It sees more
  discriminating tests (16.8% → 28.0%) and less local-search stagnation
  (33.5% → 18.8%).

## Methods

- **Decision process.** The state H_t is the Git repo (artifact,
  experiments, analyses, handovers, remaining budget).
  - A *fixed* curator C builds the observation o_t = C(H_t). It has
    read-only access to files, diffs and commit history, plus a fixed
    portion: task, workspace summary, latest proxy eval, a summary of the
    previous episode with a pointer to its handover, and remaining budget.
  - The policy π samples the full response a_t (reasoning + work order, or
    terminate).
  - The executor's whole trace is part of the environment transition.
  - Only true termination sets d_t = 1. Context resets and batch boundaries
    do not (App. I "trajectory stitching").
- **Reward.** The proxy reward is the increase in best-so-far normalized
  proxy quality q̄_t. Gold-binary is 1 if the final artifact meets the
  environment's success criterion. Gold evaluation is hidden during
  research. An environment-maintained incumbent "protects the strongest
  validated artifact against later regressions."
- **Critic.** It reads the same curated o_t and answers "how much additional
  improvement … do you expect from here?" over letters A–K = 0.0…1.0. Value
  is the expectation of the renormalized next-token distribution. It is
  trained by KL to a two-hot projection of the Monte Carlo return.
- **MIRA-AC.**
  - One LoRA adapter (rank 32) serves as both actor and critic, initialized
    from a pretrained critic.
  - GAE uses γ = 1 and λ_T = 1 − 1/(20T), with a "bridge" term for
    critic-version changes mid-episode.
  - Asynchronous buffered single-rollout updates (after Hou et al. 2026).
    Up to 32 live episodes adopt new weights at decision boundaries.
  - Compute is 16 H200 for actor–critic plus 16 H200 for frozen executors.
- **Inference experiments (frozen weights).**
  - GPT-5.5 direct vs Codex (one persistent session) vs MIRA (fresh Codex
    per work order). Same model, harness and tools.
  - IMOProofBench Advanced: 30 problems × 3 trials, plus Qwen3.6-27B.
  - HillClimbBench-RMT: one 24 h GPT-5.5 campaign per condition.
  - HillClimbBench-Loop: one 48 h campaign each, MIRA vs Codex, "GPT-6
    Astra".
- **Training experiments (Qwen3.6-27B for every role).**
  - Environments: BNLearn causal discovery, Physics Discovery and SRBench
    (100 tasks × 1 rollout each), and HillClimbBench-CPU (50
    paper-derived architecture tasks × 2 rollouts). At most 20 decisions
    per rollout.
  - The ablation ladder: token regression (gold-binary, an SAO analogue) →
    proxy-marginal → step regression → step generative → MIRA-AC.
  - Each configuration is one run, one pass over 100 episodes. The
    cross-environment policy uses 300 episodes (SR, Physics, HCB-CPU) and
    is evaluated on all four.
  - **Training and evaluation use "the same fixed task identities and
    seeds"** for the scientific environments. They test "new research
    trajectories … within the same tasks, not generalization to unseen task
    identities" (App. B.1).

## Results

- **IMOProofBench (Fig. 4a).**

  | Condition | Natural stop | Token-matched continuation |
  |---|---|---|
  | GPT-5.5 direct | 67.1% | 93.3% |
  | Codex · GPT-5.5 | 73.8% | 90.0% |
  | MIRA · GPT-5.5 | 100% | — |
  | Qwen-27B direct | 21.0% | — |
  | Codex · Qwen-27B | 29.0% | — |
  | MIRA · Qwen-27B | 43.3% | — |

  Our arithmetic: token matching closes about **80%** of the GPT-5.5 gap
  (26.2 of 32.9 pp) and about 62% of Codex's. The residual is 6.7–10 pp,
  about 2–3 of 30 problems.
- **Architecture campaigns (one campaign per arm, gold NLL, lower is
  better).**
  - RMT: MIRA 2.975, Codex 3.028, GPT-5.5 3.054, reference 3.147.
  - Loop: MIRA 3.046, Codex 3.228, reference 3.265.
  - The RMT gate ablation is one seed at each scale (−0.009 and −0.021 CE).
    The ungated final-scale control "was not submitted to the gold
    evaluator", and the authors say the ablations "do not isolate it as the
    cause".
- **Context curation (Fig. 7, Loop campaign).** About 10 files are selected
  per decision from a repo that grows to about 5k files. Selected text stays
  at roughly 20–110k characters.
- **Context cost (Fig. 11).** MIRA-AC averages **10.6M input tokens per
  episode** over 400 training episodes. Replay with no compaction or with
  200k auto-compaction is *projected* to cost more. The authors note these
  replay costs are projections of rollout cost under stated assumptions, not
  measured runs.
- **Critic (Fig. 8a, in-domain RMSE).**

  | Critic | RMSE |
  |---|---|
  | Step generative | 0.214 |
  | Step classification | 0.222 |
  | Step regression | 0.248 |
  | Token regression | 0.290 |
  | Token generative | 0.324 |

  On cross-environment pretraining (Fig. 8b), at 50 target-support episodes
  LOO is 0.208, single-source 0.212 and base 0.214.
- **Policy training (Table 2, gold evaluation, single run each, no CIs).**

  | Variant | SR solved | Physics solved | HCB-CPU wins/100 | BNLearn solved |
  |---|---|---|---|---|
  | Untrained MIRA | 50 | 8 | — | 76 |
  | Gold-binary token regression (SAO analogue) | 55 | 9 | 49 | 79 |
  | Gold-binary MIRA-AC | 65 | 6 | 49 | 56 |
  | Proxy step generative | 59 | 8 | 53 | 76 |
  | Proxy MIRA-AC (env-specific) | 60 | 10 | 58 | 48 |
  | Proxy MIRA-AC (cross-env; BNLearn actor-held-out) | 63 | 10 | 55 | 84 |

- **Pairwise gold comparisons for the cross-env policy vs untrained (App.
  K).**
  - SR: 58–32 (10 ties).
  - Physics: 65–26 (9 ties). Perfect scores went only "from eight to ten",
    so most of the gain is better partial recovery.
  - HCB-CPU: 55–45.
  - BNLearn: the judge's attribution is 25–22 with 53 ties.
- **Behavior (Fig. 9, LLM judge, 400 pairs).**
  - Calibrated stopping: 29 → 53 of 400 trajectories.
  - Premature stops: SR 29% → 13%, Physics 27% → 6%.
  - Continuing despite contrary evidence rose on HCB-CPU (10% → 26%).
  - BNLearn: MIRA-AC less often preserved the strongest current graph
    (32% → 19%).

## Critique / open questions

**This paper also reads below its digest framing, which makes it 32 of 32.**
The headline "improves gold performance across all four environments" holds
only for the most favourable row of Table 2. Several other numbers are
likewise the best cut.

- **"Across all four" depends on the cross-environment row.** The
  environment-specific MIRA-AC, trained *directly* on BNLearn, scores **48
  (proxy) and 56 (gold-binary) vs 76 untrained**. That is a 20–28-task
  regression, and §6.3 does not mention it; the text only says BNLearn
  "favors Step regression". Gold-binary MIRA-AC is also below untrained on
  Physics (6 vs 8). The positive BNLearn number (84) is for the policy that
  never received BNLearn actor updates. So the transfer result is the only
  BNLearn win, and in-environment training hurts.
- **Several "improvements" are within noise.**
  - Every configuration is a single run with no CIs in Table 2.
  - HCB-CPU's 55–45 split is not significant: a two-sided sign test on
    n = 100 gives p ≈ 0.37 (our arithmetic). Worse, the *process* judge,
    blind to outcomes, preferred untrained MIRA 53–45 on that environment.
  - Physics "solved" 8 → 10 is 2 tasks of 100.
  - SR (58–32, p ≈ 0.008) and Physics on partial scores (65–26, p < 0.001)
    are the robust wins.
- **No held-out tasks.** Policies are trained and evaluated on the same task
  identities and seeds (App. B.1). The "gold" outcome guards against proxy
  overfitting within a task, not against memorizing the task. For BNLearn,
  the gold evaluator "uses the same hidden target as the proxy", and the
  harness reports SHD after every iteration. So the proxy-to-gold story is
  weakest exactly where the transfer claim rests.
- **A harness confound the authors flag themselves.** Forced continuation
  "appeared in 57 MIRA-AC trajectories and 17 untrained MIRA trajectories"
  in SR, which "weakens a causal interpretation of the aggregate
  difference." The fewer premature stops may partly be this mechanism, not
  learned judgment.
- **The untrained-MIRA theorem-proving gain is mostly budget.** "67.1% →
  100%" compares natural stopping points, and MIRA spends far more output tokens
  (its curves run to about 1M per problem vs tens of thousands for the
  baselines, read from Fig. 4a). Token-matched continuation gets GPT-5.5 to
  93.3%. The architecture's own contribution is the residual of 2–3
  problems, plus the inertia shift. That residual is real and interesting,
  but it is not the headline number.
- **The NAS campaigns are n = 1.** One 24 h and one 48 h campaign per arm,
  no repeats, so the gold-NLL gaps (0.05 and 0.18) have no variance
  estimate. The Loop comparison uses a different model ("GPT-6 Astra") from
  everything else.
- **The critic's edge over the next-best design is small.** 0.214 vs 0.222
  (step classification), with overlapping CIs in Fig. 8a. The headline
  "absolute 0.110" is measured against the *worst* baseline. The LOO
  pretraining benefit at full support is 0.004–0.006 RMSE. An RMSE of 0.21
  on a 0–1 scale with 0.1-wide bins is coarse.
- **Wrong in the digest: the critic is not a stopping rule.** The digest's
  "Why" reads the critic as "a learned value-of-continuing estimate … an
  alternative to a fixed `max_consecutive_no_improvement`." In the paper:
  - The critic serves only as the GAE baseline and as an auxiliary loss on
    the shared adapter.
  - Termination is the *policy's* sampled action.
  - No experiment thresholds V(o_t) to stop, or compares learned stopping
    against a patience counter or a fixed horizon.
  - Every training rollout is capped at 20 decisions anyway.

  "Calibrated stopping" (29 → 53 of 400) is an LLM-judge label. It is not a
  measured stop-quality metric.
- **The behavioral analysis is descriptive and judge-based** (GPT-5.5 judging
  Qwen trajectories). The authors say as much: it does "not establish that
  any individual behavior … causes the associated performance difference."
- **Cost and reproducibility.** No code or benchmark release is named
  (HillClimbBench is new). Appendix J ("Additional MIRA-AC Results") is an
  empty heading. Training needs 32 H200s. The curator is frozen and its
  outputs were "not fully logged", so the context-cost accounting excludes
  it.
- **What survives cleanly:** the decomposition itself, and the evidence that
  a fresh-context outer loop over a Git record keeps context bounded and
  pivots more than one persistent session with the same model and tools.
  The practical counsel in App. K.8 is the most transferable output:
  - make each costly experiment name the hypotheses it separates;
  - test changes on shifted data before selecting them;
  - prefer the smallest edit set once only a few uncertainties remain.

## Trust signals

- **Credibility:** 3. Meta AI with Columbia and Tel Aviv, and a
  well-specified method (full hyperparameters, prompts and the stitching
  invariant are given). The authors are candid in places: the forced
  continuation confound, the one-seed RMT gate ablation, and "not
  generalization to unseen task identities" are all stated. Held back from
  4 because:
  - it is an unreviewed preprint with no code or benchmark release;
  - every policy configuration is a single run with no CIs;
  - the NAS campaigns are n = 1;
  - train and eval tasks are the same;
  - the abstract's "all four" claim hides an in-environment BNLearn
    regression;
  - Appendix J is empty.

## Follow-up

- **Relevance:** 4. It is the closest architectural match to `/iterate` in
  the graph: an ideator turn that writes a scoped work order, a fresh
  executor, a Git record as the only state, and a per-decision handover.
  The same model, harness and tools are run with and without the outer
  loop. It strengthens [[concepts/hierarchical-delegation]] and
  [[concepts/file-as-bus]] with an ablation and adds a curation data point
  to [[concepts/context-eviction-policy]]. It is not a 5 because:
  - the inference gain is mostly budget once token-matched;
  - the learned components are heavy and weakly evidenced;
  - it seeds no new concept.
- **Bearing on `max_consecutive_no_improvement` and redirect-before-halt
  (see [[concepts/budget-as-ceiling]]):**
  - *No evidence for replacing the patience counter with a learned value
    estimate.* The paper never uses the critic to stop.
  - *Weak support for redirect-by-architecture.* With weights frozen, a
    fresh-context outer loop pivots more (inertia 0.772 vs 0.862 for one
    persistent Codex session, matched by problem and trial with bootstrap
    CIs, App. D). That is a structural redirect, not a prompt one. It
    agrees with [[literature/papers/yan2026traceml]]'s finding that a
    prompt-level self-check does not produce pivots. Caveat: this is proof
    search, not ML transitions.
  - *Pivoting has a cost.* RL training on proxy reward raised "continuing
    despite contrary evidence" (10% → 26%) and lowered preserving the best
    graph (32% → 19%). The authors' K.8 summary is that training made
    decisions "more likely to change the final result, for better or
    worse".
- **For `/iterate`'s work-order format.** MIRA's work order carries
  objective, relevant evidence, resource guidance, **acceptance criteria and
  a stopping condition** (App. C.1). Acceptance criteria and a per-work-order
  stop condition are worth checking against the current ideator-turn
  template.
- **Trajectory stitching invariant (App. I).** "Only true environment
  termination may stop value bootstrapping; infrastructure boundaries never
  do." It is directly relevant to any downstream project doing RL over
  context-reset agent episodes.
- **Candidates:**
  - Samanta et al. 2026b, BayesBench (arXiv 2606.30850): LLM belief
    trajectories under multi-turn evidence. It is the source of the
    generative-critic elicitation.
  - Hou et al. 2026, single-rollout asynchronous agentic RL (SAO, arXiv
    2607.07508).
  - Kausik et al. 2026, "The context gathering decision process" (arXiv
    2605.07042).
  - Zou et al. 2026, FML-bench (arXiv 2605.17373). It is a controlled study
    of research-agent search dynamics, with Goyal as a co-author.
