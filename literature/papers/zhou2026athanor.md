---
kind: paper
title: "Athanor: An Agent-Native System for Autonomous Post-Training Research"
authors: ["Yufa Zhou", "Yichen Wang", "Jiakun Fan", "Ziqi Shu", "Zhichao Liu", "Anru R. Zhang", "Neiwen Ling", "Zijun Wei"]
institutions: ["ByteDance", "Duke University", "The University of Chicago", "Virginia Tech"]
year: 2026
venue: "arXiv 2610.21055 (cs.AI), v1 2026-10-15, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.21055"
code_url: null
citations: null
source: "raw/papers/zhou2026athanor.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/file-as-bus]]"
tags: ["autopostrain", "post-training", "recursive-self-improvement", "multi-agent-team", "sealed-holdout", "twin-suite", "frozen-evaluator", "decontamination", "research-record", "checkpoint-lineage", "recipe-convergence", "correlated-priors", "model-merging", "self-distillation", "first-seed-inflation", "stall-halt", "post-hoc-protocol", "claude-opus-5.5", "qwen3-4b", "n-equals-1"]
---

# Athanor: An Agent-Native System for Autonomous Post-Training Research

## TL;DR

Athanor is a research environment, not an agent. Researcher agents run in
their own harness (here Claude Code) inside a sandbox on one 8-GPU node each.
They act through one command (`athanor job|eval|data|artifact`). Services
outside the sandbox run jobs, decontaminate incoming data, score checkpoints
with a frozen evaluator, and write every request, result, checkpoint lineage,
transcript and note to a persistent record. A held-out "twin" suite exists
that the agents are never told about.

One campaign is reported. Four Claude Opus 5.5 agents (xhigh effort, no
skills, no research advice) post-trained Qwen3-4B-Base for 104 hours with
public data and no teacher model. Cost was about $11,800 ($10,800 GPU at
$3.49/GPU-h, $1,074 inference).

- **Headline (final measurement, 4 seeds):** dev 15.9 → 67.3 and held-out
  9.2 → 48.2, against 66.5 / 47.9 for Qwen3-4B-Instruct-2507. Paired 95%
  CIs on the differences contain zero on both suites.
- **Under the pre-declared campaign protocol (2 seeds):** held-out 46.7 ± 1.3
  against 48.0 ± 1.4, so the team is 1.3 points lower (CI −4.0 to +1.3).
  The "final measurement" (32k tokens, sampled decoding) was added *after*
  the campaign, because the vendor's reasoning releases ran out of tokens
  under the campaign limits.
- **Parity is carried by one task.** On multi-step action (ALFWorld,
  fine-tuned on expert trajectories) the team scores 98.9 / 78.8 against
  82.4 / 65.9. Without it the team trails Instruct-2507 by 3.7 (dev) and
  4.2 (held-out). It is 11–17 points below on logic and math.
- The recipe is four known methods composed by the agents: SFT on a public
  distilled corpus, GRPO with a quarter of rollouts decoded greedily (an
  agent's fix), cross-agent weight merges with extrapolation, and
  rejection-sampling self-distillation.

## Claims

- "To the best of our knowledge, this is the first public result in which a
  team of AI agents, working autonomously, post-trains a raw base model into
  one cross-domain generalist model at the level of the vendor's instruct
  release, confirmed on the held-out suite."
- Design thesis: "as agent intelligence improves, the bottleneck in
  autonomous research shifts from generating plausible ideas to the friction
  of executing and verifying them." Three principles: separate research
  decisions from execution, externalize the research state, keep the loop
  productive (background jobs that push results into the session).
- No human-designed research scaffolding, citing the bitter lesson. "In our
  experience, earlier versions of Athanor that prescribed a form per recipe,
  an approval per experiment, and a central scheduler constrained stronger
  agents more than they helped them." This is stated, not measured.
- Four findings from the record (Table 2): (1) recombination of known
  methods; (2) team progress, with ideas and checkpoints moving between
  agents within hours; (3) recipe convergence on one corpus within the first
  hour; (4) uneven data sense: agents check format, length, scorability and
  difficulty but rarely answer correctness.
- "Once the friction of execution is removed, the clearest limit we observe
  is how many different ideas they try."
- Limitations stated by the authors: one campaign, one base model, one agent
  model; the four submissions share work and are not independent repeats;
  much of the public data was itself distilled from stronger models, so
  "no teacher" means "from public resources, not from the base model alone".

## Methods

- **Roles (Table 3).** Researchers decide data, method, hyperparameters,
  merges, when to evaluate, and which checkpoint to submit. The operator
  declares the measurement protocol before the start, controls nodes and
  the window, holds the sealed held-out suite, and gives no research advice.
  The environment writes the record; nobody decides it.
- **Isolation.** Evaluation runs outside the sandbox in its own network
  namespace with a frozen evaluator. Only dev scores and scored dev answers
  come back. The agents see every scored dev answer, and the reference
  models' per-item dev scores but not their answers.
- **Decontamination.** Lowercase, strip punctuation; drop a training row if
  at least half of a benchmark item's 13-word sequences appear in it.
  Items under 13 words are not matched. 40,113 of ~384M pool rows removed.
  Post-campaign audit: no session text matched a held-out item, and no agent
  visited a site hosting a held-out set. 61 of the 108 datasets on the
  submission's lineage were fully scanned; 30 were lost with node disks,
  17 exceeded scan limits.
- **Suites (Table 7).** Six tasks each, plain mean. Held-out twins: AIME/HMMT
  2026 vs 2025 competitions (later window); the other half of ZebraLogic
  (same distribution); LiveCodeBench 2025 vs 2024; SuperGPQA vs MMLU-Pro;
  IFBench vs IFEval; two unseen ALFWorld task types vs four seen. Held-out
  sizes 85–1,000.
- **Selection.** Fixed before the start: each agent submits one checkpoint,
  the team submission is the one with the highest dev score, and the
  held-out suite plays no part in the choice.
- **Operator interventions (Table 8).** Hour 15.8: notice that the result
  counts the team's submission (fixed in the protocol but missing from the
  agents' documents). Hour 59: window extended 72 → 144 h. Hour 73: Claude
  Code web search, wait and notification tools turned on. Hour 97: each
  agent given its own truncation statistics. Hour 104: campaign closed
  because the best dev score rose by less than one point in the previous day.
- **Scale.** 500 jobs, 255 full dev evaluations covering 174 checkpoints,
  57 cross-agent checkpoint fetches, 160 shared-note versions, 3,218 tool
  calls (97% Bash), 8,243 model calls.

## Results

- **Main table (Table 1 / 10).**

  | Model | Dev (campaign) | Held-out (campaign) | Dev (final) | Held-out (final) |
  |---|---|---|---|---|
  | Qwen3-4B-Base | 16.1 | 8.7 | 15.9 | 9.2 |
  | Default starter (no research) | 36.4 | 20.4 | — | — |
  | Team submission | 66.1 (3 seeds) | 46.7 | 67.3 | 48.2 |
  | Qwen3-4B-Instruct-2507 | 66.4 | 48.0 | 66.5 | 47.9 |
  | Qwen3-4B (hybrid) | 58.4 | 38.0 | 73.3 | 50.6 |
  | Qwen3-4B-Thinking-2507 | 56.8 | 41.0 | 80.5 | 63.0 |

  The two reasoning releases jump 15–24 dev points under the final
  measurement. The team moves +1.2 / +1.5 and Instruct-2507 +0.1 / −0.1.
- **Per task (final measurement, team vs Instruct).** Dev: math 38.4 vs
  44.2, logic 63.0 vs 79.5, code 51.1 vs 45.1, knowledge 70.4 vs 64.1,
  IF 85.2 vs 84.1, action 95.6 vs 81.8. Held-out: math 33.3 vs 42.5, logic
  62.5 vs 77.7, code 37.9 vs 33.6, knowledge 43.3 vs 36.5, IF 37.6 vs 30.3,
  action 74.7 vs 66.8.
- **Dev → held-out gap.** Team 66.1 → 46.7 (19.4) vs Instruct 66.4 → 48.0
  (18.4) under the campaign protocol; 67.3 → 48.2 (19.1) vs 66.5 → 47.9
  (18.6) under the final one. The excess gap after 255 dev evaluations with
  per-item answers visible is 0.5–1.0 points, inside the 1.3–1.4 SE.
- **Selection noise.** For 23 checkpoints re-evaluated on more seeds, the
  first seed was higher than the later mean in 19 (sign test p = 0.003), by
  0.4 on average and at most 1.1. The best-single-evaluation frontier
  reached 66.9; the multi-seed submission is 66.1.
- **The four submissions are not separated by held-out.** Campaign protocol
  held-out 45.9–47.2; Agent 2 was 0.01 dev points below the chosen Agent 3
  and 0.5 held-out points above (SE of difference 0.7).
- **Cost curve.** Frontier about 50 after $1,100, about 60 after $2,400; the
  last six points (60 → 66.1) cost about $8,000. Per task, action passes
  Instruct by hour 9, knowledge by 21, code by 32; logic climbs 49 → 65
  after day 1, math stays below throughout.
- **Execution.** Jobs hold GPUs 93% of available hours (3,086 of 3,315);
  measured utilization about 75%. Agents spend 87% of session time waiting
  on jobs, 3.6% reasoning. Under 1% of GPU hours pass with an idle GPU
  waiting on an agent. 41 of 2,611 athanor commands fail; 26 of 500 jobs end
  failed. Three node losses (hours 46, 70, 94) cost 23 GPU-h of work;
  sessions resumed from the record.
- **Team progress (Fig. 6–7).** Before hour 15.8 the agents read each
  other's files 37 times and wrote no notes and fetched no checkpoints.
  Within minutes of the notice they started; the first fetch came 56 s
  after it. From hour 21 every new best is a cross-agent merge. 139 of 287
  ancestry links pass through a teammate's checkpoint. Of the 32 checkpoints
  evaluated after hour 84, 28 contain training from all four agents.
- **Recipe convergence (Fig. 7c).** Within the first hour all four agents
  made AM-Thinking-v1-Distilled 76–87% of their first SFT mixture. They
  looked at 43 of 447 pool datasets and used 27. No agent ran a controlled
  comparison of the corpus against an alternative. Two diversifying lineages
  (other reasoning traces; shorter data) scored lower and were dropped after
  one round (lineage B stuck near 55 while the team's best was near 62).
- **The rollout fix (Trace 1–2).** Agent 3 classified cut-off validation
  answers and found greedy decoding falls into repetition loops that GRPO on
  sampled rollouts does not touch. Decoding a quarter of rollouts greedily
  took validation loops 86 → 18 and the full dev score 49.7 → 57.3. All
  three teammates adopted it within hours, crediting Agent 3.
- **Truncation (App. D.1).** 17.5% of the submission's logic answers and
  24.2% of math answers hit the 16k limit (Instruct: 0.8% and 9.8%). Raising
  the limit to 32k barely changes logic (63.4 → 63.0); the cause is large
  puzzles the model rarely solves.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). Every number in the digest
  entry is in the paper and the "n = 1" caveat is there. The entry leaves
  out three things that narrow "matches or surpasses":
  1. *The held-out tie uses the favourable protocol.* The protocol fixed
     before the campaign gives 46.7 vs 48.0 (team 1.3 lower). The abstract's
     48.2 vs 47.9 comes from a second scoring of the held-out suite under a
     protocol chosen after the first results. The motivation (the reasoning
     releases were truncated) is fair and it was applied to every model, and
     both protocols say "statistically tied". But the direction of the
     headline flips with the post-hoc choice.
  2. *The average hides a task split.* Parity rests on ALFWorld, where the
     team trained on in-domain expert trajectories. Without it the team is
     about 4 points behind on both suites, and 11–17 behind on math and
     logic. "Cross-domain generalist" is true of the training target, not of
     uniform parity.
  3. *"No research advice" had operator messages that changed behaviour.*
     The hour-15.8 notice is what started all sharing, and sharing is where
     every new best after hour 21 came from. The hour-97 message handed each
     agent a diagnostic. These are facts about the setup, not advice, but the
     team-progress finding is conditional on the first one.
- **The dev score is the selection target.** 67.3 vs 66.5 on dev is not
  evidence of anything beyond the held-out comparison. The team searched dev
  for 104 hours with every scored answer visible, and its first-seed numbers
  run high (19/23). The paper handles this correctly by reporting multi-seed
  means for the submission.
- **The clean measurement here is the gap, not the level.** The team's
  dev → held-out drop exceeds the vendor model's by only 0.5–1.0 points
  after 255 dev evaluations. That is the best evidence in the paper that the
  sealed suite caught little overfitting. Two limits: part of the twin suite
  is near-iid (the ZebraLogic halves), and the reference was not trained on
  this dev suite either, so the comparison bounds *excess* overfitting, not
  absolute transfer.
- **Decontamination is shallow.** A 13-gram, half-the-item overlap rule
  misses paraphrases and answer-only leakage, and short items are never
  matched. The main training corpus is distilled from stronger models whose
  own training data is unknown. 47 of 108 lineage datasets were not fully
  scanned. The held-out math items post-date the dev set, which is the
  strongest protection here.
- **No arms.** There is no single-agent arm, no no-sharing arm, no
  different-model team, no second campaign and no human-baseline campaign at
  equal compute. The four findings are descriptions of one run. The
  "scaffolding hurts" claim is an anecdote about earlier versions.
- **Code is not released.** The paper calls Athanor "open", but names no
  repository, and none was found in the PDF's links. The record itself (the
  paper's main asset) is not released either.
- **For redirect-before-halt (docs/system-proposals/2026-10-18):** two
  data points, both cautionary about one clause. The diversifying pivots that
  restarted a lineage from the base each lost after one round to a lineage
  with a day or more of refinement, and were dropped. The proposal judges
  the pivot "against the chain's best, under the same band", which
  structurally favours the incumbent when the pivot is a restart. The pivot
  that worked (greedy rollout share) changed one axis *on top of* the
  incumbent, and it came from auditing scored outputs, which matches
  sun2026scipivot's "audit the measurement first" ordering. n = 2, no
  counterfactual.
- **For the accepted noise band:** the campaign's own stall halt used a
  1-point band per day on a 6-task mean, and the measured first-seed
  inflation (mean +0.4, max 1.1) sits just inside it. This is one
  independent calibration of a band near one seed's maximum excursion.
- **For shared-substrate correlated error:** the convergence on one corpus
  happened in the first hour, before any notes or fetches. That is a
  correlated-prior effect (four identical Opus 5.5 instances), not
  contagion over the record. See the concept edit.
- **For skill scoping:** agents ran with no skills at all and reached this
  level. That is compatible with goal-level, minimal skills, but it is not
  a test: no with-skills arm exists.

## Trust signals

- **Credibility:** 3.
  - For: ByteDance with Duke, UChicago and Virginia Tech. The evaluation
    design is careful: evaluator out of reach, held-out suite sealed and
    undisclosed, selection rule fixed in advance, seeds and SEs reported,
    paired CIs on the headline differences, a post-campaign contamination
    audit, first-seed inflation measured and corrected for. The authors
    report the unfavourable campaign-protocol numbers in Table 1 alongside
    the headline, state the n = 1 limit and the distilled-data caveat, and
    give the operator interventions with timestamps.
  - Against: no peer review (v1, five days old). One campaign, no ablation
    arms. No code, data or record released despite "open". The abstract's
    parity claim uses a protocol added after the campaign. The four findings
    are descriptive. Model names (Opus 5.5) describe a snapshot.
  - Not 4 because nothing is released to reproduce and the headline rests
    on one run and a post-hoc protocol choice. Not 2 because the reporting
    is unusually complete and self-critical, and the unfavourable cut is in
    the main table.

## Follow-up

- **Relevance:** 4. This is the largest end-to-end ML-research-agent run in
  the graph that is scored on a sealed, undisclosed holdout with a frozen
  evaluator outside the sandbox. It is material evidence for
  [[concepts/hce-evaluation]] (the excess dev → held-out gap after 255
  evaluations is about one point; the post-hoc second scoring shows how a
  headline moves) and for [[concepts/shared-substrate-contagion]] (a natural
  no-channel window shows convergence from correlated priors, then measured
  reuse once the channel opens). It also gives operating numbers for
  [[concepts/async-worker-pool]] (GPUs held 93%, agents waiting 87%) and a
  live instance of [[concepts/file-as-bus]] (the record as the only state
  across three node losses). Not 5 because it is one campaign with no arms,
  so it anchors no concept by itself.
- Related in the graph: [[literature/papers/hambardzumyan2026aira]] (HCE and
  async workers; cited), [[literature/papers/cai2026forkandflush]] (idea
  basins; cited), [[literature/papers/zhang2026agora]] (git as shared memory;
  cited), [[literature/papers/li2026praxist]] (solution lineages; cited),
  [[literature/papers/sun2026scipivot]] and
  [[literature/papers/shao2026language]].
- **Candidates** (cited, not yet in the graph):
  - Park et al. 2026, "Scaling discovery through test-time communication"
    (arXiv 2609.21032). Sharing helps when a verifier says whose work to
    build on.
  - Chen, Wen, Kirchner 2026, "Automated researchers can mitigate
    well-characterized alignment failures" (arXiv 2608.28945), AAR.
  - Lim et al. 2026, "What is missing from AI post-training AI"
    (arXiv 2608.19072). PostTrainBench agents rarely change strategy.
  - Meng et al. 2026, RSIGym (arXiv 2610.10310).
  - Du et al. 2026, RSI-Master (arXiv 2609.35561).
  - Anthropic Frontier Red Team 2026, "Patterns and problems in emerging
    multiagent systems" (blog). Agents sharing a model, harness and context
    act alike.
