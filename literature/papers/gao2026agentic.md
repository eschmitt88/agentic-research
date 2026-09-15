---
kind: paper
title: "Agentic ML Exploration (A-MLE) for Ads Ranking"
authors: ["Erwin Gao", "Vinodh Kumar Sunkara", "Jingyi Guan", "Qinjin Jia", "Hangjun Xu", "Xiang Ji", "Sherman Wong", "Surya Teja Chavali", "Pratik Vaishnavi", "Aryan Pandhi", "Xiaoyu Deng", "Zhaodong Wang", "Samarth Inani", "Fan Yang", "Jakob Moberg", "Zoe Zu", "Nicolas Bievre", "Sami Khenissi", "Amit Jaspal", "Ehsan Fakharizadi", "Srinidhi Viswanathan", "Dorothy Sun", "Abishek Vanam", "Sneha Iyer", "Sheela Yadawad", "Wenjie Chen", "Gaby Nahum", "Junhua Gu", "Peter Chu", "Yucheng Liu", "Xin Zhao", "Vitor Cid", "Chaorong Chen", "Vijay Pappu", "Ashwin Kumar", "Wenlin Chen", "Ben Schulte", "Deepak Chandra", "Ritwik Tewari"]
institutions: ["Meta Platforms, Inc."]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.08248"
code_url: null
citations: null
source: "raw/papers/gao2026agentic.pdf"
added: "2026-09-15"
relevance: 3
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["industrial", "ml-research-agent", "recsys", "ads-ranking", "cross-llm", "prompt-sensitivity", "human-in-the-loop", "staleness", "technique-transfer", "meta"]
---

# Agentic ML Exploration (A-MLE) for Ads Ranking

## TL;DR

A second Meta deployment report on a research agent for production ranking
models, from a team with no author overlap with
[[literature/papers/li2026autorecsys]]. **One** tool-using agent walks
hypothesis → exploration strategy → execution → result analysis, with a
human checkpoint at each stage boundary. It reads and writes a
git-versioned markdown "shared substrate" of per-technique and per-model
Track Records, so a technique validated on one model surfaces on
architecturally similar ones. The useful part is a **fixed-loop LLM swap**.
Under a basic prompt, Gemini 2.5 and GPT-5 explore hardest
(116.04×10⁻⁴ rMSE improvement each) and Sonnet plays safe (0 to 33.57).
Under a "stressful, competitive" prompt, **Sonnet 4.0 jumps to 255.87 and
GPT-5 drops to 68.05**, so the prompt effect changes sign by model family.
Domain skills lift a small L1 tool bench from 8% (generic LLM) and 16%
(generic ML agent) to **68%**. Almost every deployment claim (throughput,
training success, proposal acceptance) is given **without a number**. The
text also contradicts its own Figure 3 on GPT-5.

## Claims

- The bottleneck in an industrial ranking portfolio is **human iteration
  throughput**, not model capacity. The long tail of models rarely gets
  expert attention, so proven techniques spread slowly between models.
- **Chaining phases, not automating one, is the leverage.** A
  semi-automated baseline (engineer drives hypotheses, scripts run jobs and
  evals) gains less throughput, which they attribute to "the agent's ability
  to chain phases without engineer-mediated handoffs." No number is given.
- **Reliability comes from the harness, not the model**: "the skill
  library's coverage, the evaluation pipeline's statistical rigor, and the
  execution layer's resilience to infrastructure noise." The authors predict
  better LLMs "will close the hypothesis-quality gap faster than they close
  the orchestration harness gap."
- **Domain skills, not raw LLM capability, set the L1 ceiling** (Figure 2).
- **Technique transfer gave the best results.** The agent spots that a
  technique validated on one model will likely carry over to a structurally
  similar model that hasn't tried it. It struggles on models with a recent
  non-trivial baseline change, "where its hypotheses were calibrated to a
  prior version of the model."
- **Human checkpoints limit blast radius.** A hallucinated code change is
  caught before launch, and a miscalibrated evaluation before the proposal
  is written.

## Methods

- **Session.** Parameterized by a (model, objective, compute) triple. It ends
  in a documented proposal or a **documented null result**, and both count
  as a completed iteration.
- **Hypothesis generation.** The agent reads the live training config,
  baseline metrics and the rolling history of attempted techniques. Internal
  generators (model-internal-state analyzers, training-efficiency analyzers,
  literature retrievers) propose candidates, and an **LLM critic scores them
  for novelty and feasibility**. Grounding in live state rather than a
  snapshot is emphasized because hypotheses rarely survive a baseline
  refresh.
- **Exploration strategy.** Under a run/compute/wall-time budget, the agent
  interleaves isolated validation (explore) with combining the best
  candidates (exploit), and negotiates the plan with a human.
- **Execution.** The agent edits in a sandboxed copy, then runs type checks
  and unit tests, builds an image, runs a smoke pass, and submits the job.
  It monitors, separates infrastructure errors from real divergence, and
  retries or fixes within a capped retry limit. It reroutes compute when a
  branch proves infeasible.
- **Analysis.** Significance is computed against a **rolling baseline**
  (the current reference config, not a frozen snapshot), with per-segment
  decomposition and an **automatic re-run when within-run variance exceeds a
  threshold**. The output is a leaderboard that either loops back to
  strategy or becomes a proposal.
- **Shared substrate.** "Long-lived markdown trees versioned in source
  control" hold per-technique and per-model knowledge. At session start the
  target model is matched against **structured eligibility annotations**. At
  session end, outcomes are committed back to the Track Record, "reviewable
  like any other source-control change."
- **Skill library** (Appendix A). Typed procedures with structured I/O
  schemas: codebase navigation, training config (propose edit, validate
  against schema), launch and monitoring, evaluation, proposal authoring. An
  **explicit waiting operator** suspends the loop until an external event
  (job completion, eval ready) and then resumes.
- **Evaluation tiers.** L1 is single-step questions about configs and
  infrastructure. L2 is four multi-step workflow tasks on one model *M\**
  (baseline refresh, variance test, config-change A/B, batch offline eval).
  L3 is open-ended exploration. Most detailed studies use *M\**, a
  lightweight regression-objective model. All results are **relative to
  baseline**. Models are anonymized.

## Results

- **L1 bench on *M\*** (Figure 2; values printed on the bars). Result
  Analysis: 12.5 / 12.5 / 37.5. Job Modification: 9 / 36 / 100. Efficiency:
  0 / 0 / 50. Overall: **8 / 16 / 68**. The three columns are generic LLM,
  generic ML agent with cross-portfolio tools, and domain-equipped A-MLE.
  Item counts aren't reported. The 12.5% steps imply about 8 Result
  Analysis items.
- **L2 task completeness** (Figure 3a): Sonnet 3.5 **93.3**, Sonnet 3.7
  **93.3**, Sonnet 4.0 **96.7**, Gemini 2.5 **100**, GPT-4 **13.3**, GPT-5
  **66.7**. The failing model hallucinated workflow IDs or didn't wait for
  async jobs.
- **L3 architecture exploration** (Figure 3b; rMSE improvement ×10⁻⁴, basic
  → stressful prompt). Sonnet 3.5: 0 → 69. Sonnet 3.7: 33.57 → 34.7.
  **Sonnet 4.0: 33.57 → 255.87.** Gemini 2.5: 116.04 → 156.08. **GPT-5:
  116.04 → 68.05.** GPT-4 was not run at L3.
- **Headline outcome on *M\*** (Table 1). Single-hypothesis arch scale-up
  +0.44% (QPS neutral). Multi-round arch exploration +0.58% (neutral).
  **Multi-source (arch + efficiency generators) +2.56%**, QPS +0.42%.
  Improvements were "measurable" on "a majority of evaluated models," with
  the largest gains on the long tail. No per-model numbers.
- **Deployment metrics, all qualitative.** Throughput is "multiple times"
  the manual baseline. Training success "meaningfully surpassed" baseline,
  with "a small minority" needing a human. Proposal acceptance was "much
  higher"; reviewers credited statistics, documentation of negative results,
  and segment decomposition.
- **Technique families** (Table 2, bucketed: many ≥ 8 models, several 3–7,
  few ≤ 2). SSL pretraining: many attempted, majority passed. Embedding
  features: several, majority. Optimizer/loss tweaks: many, mixed.
  Token-mixing: several, mixed. Architecture scaling: few, mixed.
- **Five failure modes.** (1) Hallucinated APIs: pre-flight checks catch
  them "but consume compute." (2) Baseline drift: a concurrent refresh erases
  a win, "mitigated but not eliminated." (3) Infrastructure incidents read as
  divergence and viable candidates get abandoned; the retry loop was extended
  to tell them apart. (4) Over-confident single-seed promotion, "materially
  reduced" by the auto re-run rule. (5) LLM-specific: weaker models fake
  completion, and some models react to stressful prompts in "non-monotone
  ways."

## Critique / open questions

- **The text contradicts its own figure.** §6.7 puts "Sonnet ≥ 3.5, Gemini
  2.5, GPT-5" in a high-90s cluster, with "the rest" unable to run the loop.
  Figure 3a shows GPT-5 at **66.7**, and "the rest" is one model (GPT-4,
  13.3). The honest reading is three tiers, not two clusters. The same
  section says that under stress GPT-5 "gives back most of its basic-prompt
  gains". The figure shows 116.04 → 68.05, a 41% loss. It also frames the
  flip as Sonnet vs GPT, when Gemini 2.5 *also* gains under stress
  (116.04 → 156.08).
- **The L3 cells look like single runs.** Different models share identical
  values to two decimals (33.57 for Sonnet 3.7 and 4.0 basic, 116.04 for
  Gemini and GPT-5 basic), which suggests they reached the same discrete
  configuration. No seeds, variance or repeat count are given. The
  "aggressiveness" reading and the prompt sign flip therefore rest on one
  outcome per cell on one model.
- **The units don't reconcile.** Figure 3b's best cell (255.87×10⁻⁴ =
  2.559%, if the axis is relative) is labelled *architecture* exploration.
  Yet Table 1 gives arch-only exploration +0.44 to +0.58% and credits
  **+2.557%** to the *multi-source* configuration. Either the figure's axis
  is not relative improvement, or the headline and the Sonnet 4.0 stress
  cell are the same result under two labels. The paper doesn't say which.
- **The "harness beats model" thesis holds only where the paper tested it
  lightly.** At L1 and L2, the loop and skills make most recent models
  reliable. At L3, "the regime we ultimately care about," the harness was
  held fixed and outcomes still ranged from 0 to 255.87 across backend and
  prompt. On the paper's own data, backend and prompt dominate exploration
  *outcome* even if the harness dominates *reliability*. The prediction
  about future LLMs is asserted, not measured.
- **The generic LLM in the L1 bench is not identified**, nor is the backend
  used in deployment. So "domain skills, not raw LLM capability" can't be
  separated from which model was used.
- **Every deployment metric has no number and no control.** There's no
  throughput ratio, success rate or acceptance rate. The acceptance
  reviewers are the system's own organization, unblinded. How often the
  human checkpoints modified or terminated a session, and whether they were
  active during L2/L3, is never reported. So the checkpoints' blast-radius
  claim is design rationale, not a result.
- **The substrate is never ablated.** "Strongest results came from technique
  transfer" is the only evidence the cross-model Track Record helps, and it
  comes from a bucketed table with no no-substrate arm.
- **Sections disagree on stages and checkpoints.** The abstract and §4 count
  five stages including the shared substrate, while Figure 1 draws four
  stages over a substrate. Figure 1 places the last checkpoint after result
  analysis; Appendix B places it after proposal authoring. Minor, but it
  echoes li2026autorecsys's four-vs-two checkpoint inconsistency.
- **What this adds beyond [[literature/papers/li2026autorecsys]].** Same
  company and problem class, different team. The two papers make opposite
  architectural choices and report complementary evidence:
  - *Topology.* li2026autorecsys uses a state machine with a specialist
    agent per state and parallel per-idea state files. A-MLE uses a single
    sequential agent with a waiting operator. Neither compares against the
    other design.
  - *Outcome vs operations.* li2026autorecsys reports no model-quality
    result but a measured operational-reliability curve. A-MLE reports a
    model-quality headline (+2.56% on *M\**), a portfolio technique table, a
    fixed-loop LLM swap and a skill ablation, but no reliability numbers.
    Neither has a no-memory control.
  - *Substrate write policy.* li2026autorecsys updates per-model playbooks
    by agent judgment with no gate. A-MLE commits cross-model Track Records
    through source-control review. That is a gate on the substrate write,
    though what it catches is unmeasured.
  - *What transfers.* li2026autorecsys finds playbook *structure* transfers
    across models but *contents* do not, and its contents are operational
    recipes. A-MLE transfers modeling-technique *contents*, filtered by
    architectural eligibility. The two fit together once content type is
    separated: infra recipes are model-bound, while modeling techniques
    transfer conditional on architecture match.
  - *Staleness.* Both name baseline change as the dominant staleness
    source. li2026autorecsys measures a loud regression and recovery.
    A-MLE treats it as a standing failure mode and builds against it
    (rolling-baseline significance, live-state hypothesis grounding, auto
    re-run), but gives no numbers.

## Trust signals

- **Credibility:** 2. Meta Platforms, 39 authors, describing a system
  actually deployed on production ads models, so the architecture account
  is first-hand. Institution is only a prior. Against it: an arXiv preprint
  (ACM format, no venue named), not peer-reviewed, no code or artifacts, no
  citations established. The three deployment metrics the paper defines are
  reported only as adjectives. Results are relative-only on anonymized
  models. The L3 cross-LLM figure shows no variance and looks like single
  runs. The text contradicts Figure 3a on GPT-5, and Table 1 and Figure 3b
  don't reconcile. li2026autorecsys got 3 from the same prior because it
  published real reliability numbers; this paper withholds its equivalents,
  so it scores lower.

## Follow-up

- **Relevance:** 3. It is a core-mission deployed ML-research agent, and it
  adds one axis the graph lacks: in a fixed loop, **the effect of prompt
  framing on exploration changes sign by backend**. That constrains
  [[concepts/hybrid-model-backends]], which treats a backend swap as a
  cost/capability choice under a fixed prompt. The evidence is one figure,
  apparently one run per cell on one model, so it is recorded as an
  observation to test rather than material new evidence. The other
  contributions (second sighting of baseline staleness, gated substrate
  write-back, content-type-conditional transfer) are design points without
  measurements. Not 4.
- **[[concepts/hybrid-model-backends]].** Extends
  [[literature/papers/bai2026how]]'s "token efficiency is a model property"
  to *exploration disposition*, with a prompt interaction attached. A prompt
  tuned on one backend is not a neutral constant when the backend is
  swapped. Gemini 2.5 and Sonnet 4.0 gain under pressure framing while GPT-5
  loses about 40% of its gain. It also bears on
  [[literature/papers/gurkan2026mutation]]'s finding that Sonnet 4 collapses
  to one structural form: here Sonnet's conservatism is the basic-prompt
  default and a framing change unlocks it. The system is single-agent, so it
  says nothing about the ideator/implementer *role* split itself.
- **[[concepts/skill-library-lifecycle]].** (a) Baseline staleness seen a
  second time, by a different Meta team. (b) Eligibility-annotated
  cross-model loading is an applicability-scope loading function in the
  sense of [[literature/papers/kim2026why]], keyed on architectural
  similarity. (c) It qualifies "structure transfers where contents do not":
  that holds for operational recipes, not for modeling techniques. (d)
  Write-back goes through code review, the opposite of li2026autorecsys's
  ungated playbook. None of these is ablated.
- **[[concepts/hierarchical-delegation]].** A deployed counter-design: one
  agent plus typed skills plus stage checkpoints, where the sibling Meta
  system uses per-state specialists. The semi-automated-baseline finding
  (leverage comes from removing handoffs) is qualitative, but it points at
  the coordination-overhead side of the trade, which the concept asserts is
  "strongly positive" only at MLE-bench scale.
- **[[concepts/shared-substrate-contagion]].** Its substrate is structurally
  this repository: git-versioned markdown, auto-consumed across targets,
  committed at session end. The one design difference is a review step on
  write-back. Nothing is measured on the defect side. The note that
  hypotheses were "calibrated to a prior version of the model" may be stale
  substrate records propagating, but the paper doesn't say whether they
  came from the Track Record or from the model's own history.
- **[[concepts/async-worker-pool]]: not proposed.** A-MLE's explicit waiting
  operator is the *serial* answer to multi-hour jobs, the opposite pole from
  li2026autorecsys's persisted per-idea state records. It has no throughput
  comparison, so it's worth a mention in the note only.
- **[[concepts/permission-gate-as-architecture]]: declined as prose.** The
  checkpoints are human approvals at phase boundaries, not tool-call gates,
  and nothing about them is measured (intervention rate, cost, catches).
  They are an instance of the phase-boundary firing schedule the concept
  already discusses via kang2026policyguide.
- **Evolutionary-search-grain: declined.** A-MLE does no population search
  and has no mutation unit. Its explore/exploit interleaving is plan-level
  scheduling.
- Worth watching for: any release of the L1/L2 bench items, repeat runs of
  the L3 cross-LLM cells, or actual throughput and acceptance figures.
