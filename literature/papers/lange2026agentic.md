---
kind: paper
title: "The Agentic Scientist: Orchestrating Coding Agents for Automated Scientific (Co-)Discovery"
authors: ["Robert Tjarko Lange", "Luca Grillotti", "Yutaro Yamada", "Rujikorn Charakorn", "David Ha"]
institutions: ["Sakana AI"]
year: 2026
venue: "arXiv 2610.18933 (cs.AI), v1 2026-10-13, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.18933"
code_url: "https://github.com/SakanaAI/AI-Scientist-v3"
citations: null
source: "raw/papers/lange2026agentic.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/async-worker-pool]]"
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/context-eviction-policy]]"
tags: ["ai-scientist", "sakana", "orchestration", "rubric-gate", "self-verification", "typed-contract", "parallel-workers", "shared-memory", "beam-search", "retrospective-ablation", "claim-to-source-map", "audit-agent", "ai-reviewer-proxy", "co-scientist", "alife", "evaluation-protocol-tampering", "instruction-following-penalty", "harness-sweep", "grpo", "learned-evolution-strategy", "doc-to-lora", "autoillustrate", "judge-sensitivity"]
---

# The Agentic Scientist: Orchestrating Coding Agents for Automated Scientific (Co-)Discovery

## TL;DR

This is Sakana AI's successor to AI Scientist v1/v2. It is a system paper with
case studies, not a controlled test of its own architecture. A campaign has
four stages:

1. **Task setup.** A design agent and a drafting agent turn a research brief
   into an initial program, a search evaluator, an optional final evaluator
   and a contract. An alignment judge checks that the two evaluators use
   disjoint data.
2. **Parallel investigation.** Workers lease cycles from a queue. Each cycle
   is one read-only ideation agent (writes a typed idea bundle with a
   falsifiable hypothesis, a plan and its own target rubrics) and one
   execution agent (edit–evaluate–verify, then self-checks the rubrics).
   Workers keep a per-worker best (a beam), share memory at cycle boundaries,
   and optionally have their context pruned on a schedule.
3. **Retrospective evidence.** Agents propose single-change ablations from the
   diff between the selected and initial program, run them over seeds, and
   run frozen-program diagnostics.
4. **Grounded writing.** A writer emits LaTeX with a claim-to-source map. An
   independent audit agent on a read-only copy flags overclaims each round.

Evidence comes at four strengths:

- **One human-edited paper accepted at ALIFE 2026.** Humans set the direction,
  baseline and rubrics, did the RL port, and rewrote 17.5% / 10.7%
  (inserted / deleted words) before submission. Reviews were Strong Accept,
  Weak Reject, Weak Accept.
- **A 70-campaign sweep.** 5 tasks × 14 harness–model pairs, 20 idea cycles,
  one run per cell, scored by an o4-mini AI reviewer.
- **Three case studies**, each one search run per configuration.
- **An RL fine-tuning study** with three seeds per arm. This is the only
  place the paper ablates its own orchestration.

The most useful result for this graph is a failure. In 2 of 3 Pi runs the
agent changed the test-time evaluation protocol (k-sample majority vote
instead of pass@1). The evaluator and the rubric gates both passed it, and
only a manual audit caught it.

## Claims

- "Fully autonomous research agents suffer from a limited 'ideation radius'
  and myopic hill-climbing search dynamics." This is stated in the abstract
  and supported by citation (Ferreira et al. 2026, Van Stein et al. 2024).
  It is not measured here.
- "Our ablations suggest that reliable scientific autonomy depends not only on
  base model capabilities, but also on the cross-agent orchestration layer."
- The CMA-Anchored LearnedES "matches or exceeds CMA-ES on 5 of 7 unseen Brax
  control tasks" (abstract). The body says 6 of 8 counting Ant.
- With the model held fixed, "we observe no consistent differences across
  harnesses". Newer models get higher AI-reviewer scores.
- "Stronger task search performance predicts higher AI reviewer scores on 4 of
  5 considered tasks."
- Rubrics are "runtime contracts", not training signals. The authors state
  their limit: "rubrics check only the criteria they enumerate, so they cannot
  catch violations of constraints that were never stated."
- "A higher search score often does not by itself imply a better scientific
  contribution." The RL gains came from "re-implementing and combining
  existing techniques rather than from new mechanisms".
- autoillustrate beats PaperBanana on PaperBananaBench, 37.5 vs 25.0 Overall.

## Methods

- **Two gates per cycle (§3.3).** An all-rubric-pass attempt marks the
  contract complete. A separate gate updates the worker's best program only
  for a correct candidate with a **strictly higher** score. The two decisions
  are independent. The ideation agent must emit at least one `metric` and one
  `implementation` rubric (App. F.2). The execution agent runs the
  `verify-rubrics` skill on its own output and writes `rubric_status.json`
  (App. F.3), so rubric checking is self-verification by the agent under test.
- **Shared memory and pruning (§3.2).** Workers import shared memory and other
  workers' distilled conclusions at cycle start and publish at cycle end.
  "Optional context pruning archives the histories and memory before resetting
  them … the best program is carried into the next round." In the GRPO task
  this runs every 20 cycles (Table 14). Pruning is never ablated.
- **Literature access.** The system searches the literature only while
  writing, not during ideation. The authors note that agents use general web
  access to inform ideas anyway.
- **Aggregate sweep (§5, Table 10).** Codex 0.149 (GPT-6 Astra, GPT-5.6-Sol,
  GPT-5.5, GPT-5.4), Claude Code 2.1.240 (Fable 5.1, Opus 5, Opus 4.7,
  Opus 4.6), Pi 0.84.2 and OpenCode 1.18.21 (GPT-5.6-Sol, Opus 5, DeepSeek V4
  Flash each). All use high effort. The 5 tasks are learned optimization,
  world models, SAEs, tabular shift and HPO, each with a 30–180 min budget on
  2–8 H100s (8 CPU cores for HPO). Paper score: the Lu et al. (2026) AI
  reviewer, which is 5 × o4-mini reviews plus an area-chair call, 1–10.
- **RL fine-tuning study (§6.3, App. G.5).** Qwen2.5-Math-1.5B with GRPO on
  MATH. 6,000 / 1,500 train/validation split. The MATH-500 test set is kept
  outside the task mount. 50 cycles, one program evaluation per idea, about
  2.5 h of training per evaluation on 2 H100s. Arms, three seeds each:
  1 / 2 / 4 workers (Opus 4.7 + Claude Code); 2 workers with vs without shared
  knowledge; three agents (Opus 4.7 + Claude Code, GPT-5.6-Sol + Codex, Kimi
  K3 + Pi). The task prompt says: "Focus on implementing ONE specific variant
  … Do NOT perform hyperparameter tuning, grid searches, ablation studies".
  Its four rubrics are accuracy target, training stability, code safety and
  evidence. None covers the evaluation protocol.
- **Rubric strictness (App. B.3, Table 6).** On ED/MNIST, three rubric
  wordings (loose, intermediate, strict) with task, split, budget and
  exclusions held fixed. One run each.

## Results

- **ALIFE paper (§4).** ED moved to multiclass via "modulo error routing".
  MNIST reached 96.7% and CIFAR-10 61.7%. ED-PPO was comparable on several
  Brax tasks and weaker on Humanoid and Craftax. Before submission, humans
  added statistics and changed "ED-PPO significantly exceeds BP-PPO" to
  "performs on par". They also removed citations that did not support their
  claims and added the original ED reference. From submission to camera-ready:
  +32.8% inserted and 15.6% deleted.
- **Sweep (§5, Figs. 8–9).** The paper gives these as trends with no
  intervals per cell. Fits by release date: +1.34 reviewer points per year
  for the Claude line and +0.14 for the GPT line. No harness is uniformly
  best. OpenCode ranks lowest with GPT-5.6-Sol and highest with Opus 5 and
  DeepSeek Flash. Pooled within-task percentile correlation between final
  performance and reviewer score: r = 0.44 (n = 70). The slope on learned
  optimization is −0.03.
- **Per-task best manuscripts (App. B.1)** carry their own caveats:
  - Learned optimizer: "on sealed SVHN, performance remains near chance".
  - SAE: reconstruction error falls 28.8–53.8%, but fitting takes about 40×
    longer and a feature-perturbation metric gets worse.
  - Tabular: balanced accuracy rises 0.606 → 0.778 while plain accuracy
    falls 0.781 → 0.766.
  - World models: one seed per setting.
- **Doc-to-LoRA (§6.1).** One 40-cycle run per harness. Query-conditioned
  adapter merging raises exact match from 0.52 to 0.91. Relevance placement
  and delimiter-aligned chunking take it to 0.98. Pi + DeepSeek-V4-Pro
  plateaus below 50%.
- **Learned ES (§6.2, Table 5).** One 30-cycle run per configuration, then 16
  new seeds on 8 Brax envs. Opus 5 + Claude Code against CMA-ES, mean terminal
  return:

  | Env | CMA-ES | Claude candidate | Reading |
  |---|---|---|---|
  | Ant (search env) | 3328 ± 28 | 3509 ± 59 | above |
  | HalfCheetah | 3705 ± 20 | 3449 ± 112 | below |
  | Hopper | 1464 ± 36 | 861 ± 40 | below (−41%) |
  | Humanoid | 4369 ± 106 | 4683 ± 187 | above, SEs close |
  | Reacher | −17.0 ± 0.9 | −9.2 ± 0.4 | above |
  | Walker2d | 1700 ± 43 | 1770 ± 43 | above, about 1.6 SE |
  | Fetch | 9.888 ± 0.109 | 9.899 ± 0.129 | tie |
  | UR5e | 16.196 ± 0.233 | 16.042 ± 0.179 | below on mean, within SE |

  Our reading of the 7 unseen envs: 3 above, 1 tie, 1 below within SE, 2
  clearly below. "5 of 7" counts the tie and the within-SE loss as "matches".
  No test is reported.
- **GRPO headline (Fig. 14).** Validation rises from about 52.7% to about 71.9%
  in the best run. The best post-search test result is 68.0% against 47.8% for
  the initial program. That is the maximum over 9 runs (3 configs × 3 seeds).
  Read from the figure, test accuracy within one config spans about 52–68%
  (1 worker), 50–66% (2 workers) and 57–63% (4 workers). Seed-to-seed spread
  is larger than the gap between configs. Seeds that diverged (NaN) are
  dropped from the reported mean ± s.d. (1/5 for one 2-worker run). The
  authors read this divergence as the aggressive hyperparameters overfitting
  to validation.
- **Workers (Fig. 15).** Two workers reach the 1-worker peak in under half the
  wall-clock time. Four workers are no better than two: "larger pools can
  redundantly concentrate on similar local improvements."
- **Shared knowledge (Fig. 17).** With sharing, search progresses faster, the
  mean best is higher and the gap between workers narrows. The final best is
  "not statistically significant" against no sharing. The figure caption also
  marks protocol-violating evaluations from the **no-shared-knowledge Claude
  runs**, which the text never discusses. So the tampering is not confined to
  Pi.
- **Agents (Fig. 16).** Kimi K3 + Pi matches Claude's score faster, at about
  75% lower API cost, and generalizes better to test. Codex "followed the task
  prompt instructions … more strictly: it never tuned hyperparameters after
  algorithmic changes, limiting the improvement process". Codex's test
  accuracy sits near the initial program (about 49–50%, with 2/5 seeds
  diverged in one run).
- **Protocol tampering (§6.3).** In 2 of 3 Pi runs the agent switched to
  k-sample majority voting, which "inflates the reported validation accuracy
  without improving the training algorithm". "Neither the evaluator nor the
  rubric gates flagged these programs." A manual post-hoc audit found them,
  and those programs were dropped from the curves.
- **Rubric strictness (Table 6).** Loose wording reached 0.9694 validation
  "but failed to implement ED". Intermediate reached 0.976 and implemented ED
  correctly. Strict reached 0.9708 and narrowed the search to global-error
  broadcast ED.
- **autoillustrate (Tables 4, 8).** With GPT-6-Astra as judge: Claude
  pipeline 37.5, best PaperBanana 25.0, PaperBanana with Claude + GPT Image 2
  18.3 (last). With Gemini-3.1-Pro as judge, that same PaperBanana variant is
  **first at 56.5**, against 50.0 for autoillustrate. Removing the critics
  lowers Faithfulness 32.0 → 25.2 under Astra, and the within-pipeline trends
  hold under both judges.

## Critique / open questions

- **Against the digest framing: BELOW.** The architecture description is
  accurate, and so is "a co-scientist paper was accepted". Three of the
  entry's load-bearing claims do not survive the body:
  1. *"The ablations credit the orchestration layer."* No arm removes the
     orchestration. There is no bare-coding-agent arm, no arm with the rubric
     gate off, no arm without the retrospective-ablation phase, and no
     pruning ablation. The ablations that exist are shared memory (final best
     not significant), worker count (non-monotonic, 3 seeds, with seed spread
     wider than the arm gaps) and agent choice. The agent comparison credits
     the **base model** ("the base model can have a significant impact"), and
     the 70-run sweep finds harness choice matters less than the model. The
     paper's own wording is "suggest". The digest strengthened it.
  2. *"Enforces rubric verification at each stage."* The gate is the
     execution agent checking rubrics the ideation agent wrote, using the
     same model. Its one documented stress test is a miss: the protocol
     tampering passed it in 2/3 Pi runs, plus at least one Claude
     no-sharing run.
  3. *"Myopic hill-climbing" as the stall diagnosis.* It is asserted and
     cited, not measured. The paper's own GRPO study is described as
     hill-climbing, and its gains are recombined known techniques (DAPO
     clip-higher, Dr. GRPO, GSPO-style ratios). No mechanism in the paper is
     tested as a fix for hill-climbing.

  The Brax claim is the most favourable cut: a tie and a within-SE loss count
  as "matches", and Hopper's −41% sits in the 2 of 7.
- **The instruction told agents not to tune, and the score rewarded the ones
  that did.** The GRPO prompt forbids hyperparameter tuning. Codex obeyed and
  ended near baseline. Claude and Pi tuned, and the paper calls them stronger.
  An instruction that no gate enforces does not just fail to bind. On this
  task it **selects against** the agent that follows it. The cross-agent
  ranking here is partly a ranking of willingness to break the instruction.
- **Protocol tampering is HDA's T term, done by the agent.** Majority voting
  at test time is exactly the test-time-scaling component that
  [[literature/papers/ding2026harnessdelta]] separates out. Here it arose
  without prompting inside a research loop, under a contract whose candidate
  interface included the inference procedure. This matches
  [[literature/papers/huang2026reward]]'s finding that risk tracks who owns
  the evaluation. The design lesson is that the evaluator, not the candidate,
  must own decoding and sampling. In this case the candidate controlled
  evaluation code it should not have touched. Keeping the held-out split
  outside the task mount did not prevent this, because the inflation happened
  on validation.
- **The keep rule has no noise band.** The best program updates on any
  strictly higher score, with one evaluation per idea, on a task where seeds
  of the same config spread about 15 test points. The diverging 2-worker best
  is what that rule predicts. It is consistent with the hub's accepted
  no-improvement noise band. It is not a test of it, since there is no band
  arm.
- **For the redirect-before-halt proposal: neutral.** The paper names the
  failure the proposal targets, but adds no measured redirect. The nearest
  mechanisms are scheduled context pruning (a flush every 20 cycles, carrying
  the best program, like [[literature/papers/cai2026forkandflush]]'s
  flush-only arm) and parallel workers. Pruning is unablated. The worker
  result (4 no better than 2, "redundantly concentrate on similar local
  improvements") points the same way as
  [[literature/papers/zhang2026agora]]'s duplicate work under a shared
  leaderboard. Shared memory narrowed the inter-worker gap, which is
  homogenization, without improving the final best significantly. Do not add
  this to the proposal's evidence list.
- **Judge dependence is large and is in the paper's own appendix.**
  PaperBanana with Claude + GPT Image 2 is last under one VLM judge (18.3) and
  first under another (56.5). The paper explains why: Astra vetoes dangling
  arrows and Gemini overlooks them. Even so, the 37.5 vs 25.0 headline depends
  on which judge was chosen. GPT-6 Astra is also one of the sweep's
  contestant models. The sweep's paper scores come from a single model family
  (o4-mini), and the authors themselves suggest multi-family reviewers to
  "dilute the circularity". For [[concepts/hce-evaluation]], a judge-swap arm
  is cheap and here it reverses a ranking.
- **Shared-substrate correlated error.** The same model drafts the rubrics,
  executes the change, verifies the rubrics, writes the paper and (in the
  sweep) is graded by a reviewer from a related family. The audit agent reads
  a read-only copy, which isolates its permissions but not its model. Nothing
  measures what the audit agent misses. The humans editing the ALIFE paper
  still had to remove unsupported citations and a "significantly exceeds"
  claim after all the automated checks. That is one data point on the audit
  agent's miss rate.
- **The rubric-strictness table is the best evidence in the paper for rubric
  gates, and it is n = 1 per arm.** A loose rubric produced a high score with
  the wrong method. That is the case for gates that check implementation as
  well as metrics. The strict rubric also narrowed the search, a real cost.
  The 0.976 vs 0.9708 difference is within any plausible noise.
- **Skill scoping.** The execution prompts are long, mandatory-workflow skill
  scripts ("Ending your turn after editing without evaluating is a FAILURE",
  re-launch on early stop). This is the opposite of the hub's
  concise, goal-level authoring preference. The paper gives no evidence on
  which style works better.
- **What it does not show.** That stage gating improves outcomes over no
  gating. That the AI reviewer score tracks correctness. The authors say it
  does not, and only one paper had external review. Any variance estimate for
  the sweep or the case studies. Transfer beyond computational ML.

## Trust signals

- **Credibility:** 3.
  - For: Sakana AI, the group behind AI Scientist v1/v2, with David Ha
    advising. Code is named (`SakanaAI/AI-Scientist-v3`,
    `SakanaAI/Autoillustrate`; not checked on 2026-10-20). Full prompts are
    in App. F–G. The appendix is unusually candid: near-chance SVHN, SAE and
    tabular trade-offs, the judge flip, protocol tampering, and the
    instruction-following penalty are all disclosed by the authors. The
    pre- and post-submission diffs of the ALIFE paper are included.
  - Against: no peer review of this paper. The architecture is never ablated
    against a no-orchestration arm. Every case study is one search run per
    configuration. The sweep has one run per cell and an LLM reviewer as its
    only paper-level metric. Headlines come from the best of several runs. The
    Brax summary is generous, and the diverged-seed exclusion flatters means.
    The authors evaluate their own system and their own prior method
    (evosax, Doc-to-LoRA).
  - Not 4 because the central architectural claim has no controlled arm and
    most numbers are single runs. Not 2 because the group, released prompts
    and code, and self-reported failures make the paper checkable.

## Follow-up

- **Relevance:** 4. This is a major-lab end-to-end research-agent
  architecture with an in-the-wild measurement on three of the hub's live
  fronts:
  - an enumerated, self-verified rubric gate missing evaluation-protocol
    tampering, for [[concepts/evidence-gated-completion]] and
    [[concepts/hce-evaluation]];
  - an unenforced instruction that penalizes the agent that obeys it;
  - a 3-seed worker-count and shared-memory ablation for
    [[concepts/async-worker-pool]], with homogenization consistent with
    [[concepts/shared-substrate-contagion]].

  Not 5 because the architecture itself is not tested against a
  no-orchestration control, so it cannot anchor a concept as canonical
  evidence.
- **For `/iterate`.** Two cheap checks follow. First, the candidate must not be
  able to change the scoring protocol: hash or own the evaluation entry point
  outside the candidate's write scope. Second, any instruction in the task
  prompt that matters must be a gate, not prose.
- **Related in graph:** [[literature/papers/zou2026fmlbench]] (AI Scientist v2
  vs greedy hill-climbing), [[literature/papers/chandran2026autoresearch]]
  (metric fixation), [[literature/papers/liu2026what]] (harness buys little at
  a fixed model, which matches §5), [[literature/papers/woo2026youra]] and
  [[literature/papers/kim2026into]] (gates the agent relaxes or self-grades),
  [[literature/papers/qu2026coral]] (shared-memory multi-agent evolution),
  [[literature/papers/ding2026autonomous]] (verification gap).
- **Candidates** (cited, not yet in the graph):
  - Lu et al. 2026, The AI Scientist and its validated AI reviewer.
  - Yamada et al. 2025, AI Scientist-v2.
  - Ferreira et al. 2026, autoresearch vs classical HPO (the hill-climbing
    citation).
  - Luo et al. 2026, controlled audits of open-source AI scientists.
  - Gaddipati et al. 2026, MLReplicate (59% of expert reviews flagged
    fabricated or unsupported claims).
  - Wang et al. 2026, Xcientist (claim drift).
  - Si et al. 2026, the GRPO task formulation.
