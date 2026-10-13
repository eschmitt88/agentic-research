---
kind: paper
title: "Can AI Agents Make Open-Ended Scientific Discovery? Evidence from Station"
authors: ["Wenyu Du", "Stephen Chung"]
institutions: ["DualverseAI", "University of Hong Kong", "University of Cambridge"]
year: 2026
venue: "arXiv 2610.08927 (cs.AI), v1 2026-10-06, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.08927"
code_url: "https://github.com/dualverse-ai/station-open-reseach"
citations: null
source: "raw/papers/du2026can.pdf"
added: "2026-10-13"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["station", "open-ended-research", "rediscovery-benchmark", "oracle-paper", "sub-discovery-rubric", "supervisor", "commitment-floor", "pivot-blocking", "meta-reflection", "streetlight-effect", "judgment-misalignment", "decentralized-agents", "multi-model-ensemble", "ai-scientist-v2", "codex-multiagent", "contamination", "llm-grader", "consolidation-agent", "interpretability"]
---

# Can AI Agents Make Open-Ended Scientific Discovery? Evidence from Station

## TL;DR

Station is the authors' own multi-agent "research ecosystem": six agents
(3× Gemini 3.1 Pro, 1× GPT-5.5, 2× Claude Opus 4.8) living in rooms with a
shared archive of agent-written papers, mail, forums and a Codex sandbox,
for 300 discrete ticks. This paper adapts it to tasks with no metric and
adds two mechanisms. A **Supervisor** agent forces each agent to pre-commit
to a direction for at least 60 ticks and rejects almost every pivot. **Meta
Reflection**, every 50 ticks, swaps the agent's model to GPT-5.5 for three
turns of "external expert" review.

Evaluation is rediscovery. Three ICLR 2025/2026 oral papers (emergent
planning in an RL agent, low-rank LLM logits, temporal superposition in
RNNs) supply the research question and setup, with findings withheld and
web access off. Each paper's findings are pre-split into 10–12 binary
sub-discoveries, and a GPT-5.5 grader scores the final report. Station
recovers 62.7% ± 2.8 on average (41.9–75.8% by task). Codex Multiagent-v2
gets 15.4% and AI Scientist-v2 14.4–20.6%, at matched cumulative experiment
time. Removing both new mechanisms together, on one task, drops recovery
from 75.8% to 57.9%. Two exploratory tasks with no oracle paper produce
findings that match post-cutoff concurrent work.

## Claims

- A suitable environment lets agents "autonomously make meaningful progress
  in open-ended scientific discovery".
- Open-ended tasks have two specific failure modes: **judgment
  misalignment** (the agent's sense of value diverges from researchers') and
  the **streetlight effect** (easy questions pursued and justified as
  important).
- The Supervisor and Meta Reflection "together" improve coverage and
  continuity. Research projects last longer (67.0 vs 55.5 ticks), and 71.7%
  of 92 Meta Reflection events are followed within 10 ticks by work that
  deepens or validates the current direction.
- Prior exposure in pretraining cannot explain the result: the baselines run
  the same models, "direct prompting" fails, and the grader requires
  experimental evidence, not just the statement.
- The authors explicitly do **not** attribute the gain to decentralization.
  The systems also differ in model mix and workflow.
- A large gap to human researchers remains. One oracle-paper author rated
  75% of 24 non-overlapping agent proposals 1–2 out of 5.

## Methods

- **Station substrate (Appendix A).** Rooms: Research Center (natural
  language → Codex GPT-5.5 implements and runs), Archive (papers reviewed by
  a NeurIPS-workshop-style reviewer agent), Mail, Public Memory, Common Room,
  Reflection Chamber. Agents live at most 150 ticks, work in isolation for
  their first 20, and pass private notes down named **lineages**. Contexts
  are compacted to a summary near 300k tokens. There is no evaluator and no
  task-level score.
- **Supervisor (B.1).** The first GPT agent to publish a paper becomes
  Supervisor and can no longer run experiments or publish. Its prompt is
  narrow: it is forbidden to propose directions or give technical advice. It
  holds a 1:1 mail meeting every 10 ticks, approves proposals almost by
  default, enforces the 60-tick pre-commit clause, and rejects pivots unless
  "genuinely impossible". It keeps an Agent Record in private memory.
- **Meta Reflection (B.2).** A mandatory three-turn reflection every 50
  ticks, run on GPT-5.5 whatever the agent's own model ("GPT agents tend to
  be more rigorous", an untested observation). The prompt is drawn from a
  pool of 12: overclaiming, planning, exploration, diversity, "learning from
  human researchers", pivot speed, attractor tasks, subproblem
  decomposition, anti-reward-hacking, research taste, knowledge
  crystallization. Two of them ask the reviewer to "recall a human
  researcher solving a problem similar to the research task".
- **Consolidation Agent (B.3).** Codex GPT-5.5 reads the run's records and
  writes an ~8,000-word report, 3 times per run. It is told to present "the
  strongest findings ... confidently without unnecessary hedging" and may
  omit negative results.
- **Grading (C, D).** The rubrics were fixed before any runs. GPT-5.5 scores
  each claim 0/1 using an evidence-ledger prompt, 3 seeds per report. The
  run score is the mean of 9 (3 reports × 3 grades). Grader validation: one
  author of the emergent-planning oracle paper scored 6 outputs blind (66
  judgments), and the grader's majority vote agreed on 64 (97.0%). Only that
  task was validated by a human.
- **Baselines (E).** AI Scientist-v2 on each of the three models, and Codex
  Multiagent-v2 (GPT) with a prompt adapted from OpenAI's Cycle Double
  Cover prompt. Each runs until its cumulative experiment time matches
  Station's per-task average (≈44 h / 18 h / 64 h), and each is told to
  write ≥8,000 words. Three seeds per configuration per task. With the
  ablation and exploratory runs, that is 50 runs, >3,400 h of experiments
  and 760M output tokens.
- **Exploratory tasks (7, G).** Subliminal learning and VLM hallucination,
  one run each. The authors "manually screen for potentially interesting
  findings" and validate them externally.

## Results

- **Rediscovery (Table 3/4).** Station's three task totals are 75.8
  [71.7, 80.8] for planning, 70.4 [59.3, 83.3] for low-rank and 41.9
  [35.6, 47.8] for RNNs. The best baseline per task scores 19.2, 37.0 and
  11.1. Station wins every category of every task except low-rank's
  sequence-level structure, where all systems score 86–100.
- **The hard categories are where the gap lives.** On planning, causal
  intervention is 60.5 for Station and 0.0 for every baseline. Low-rank
  functional consequences are 18.5 vs 0–5.6. RNN nonlinear dynamics are 7.4
  even for Station, whose report omits the oracle's main mechanistic
  explanation.
- **Tick 100 ≈ tick 300.** Station at tick 100 averages 61.2 ± 4.1
  against 62.7 at tick 300. On low-rank both are 70.4, and on RNN nonlinear
  dynamics tick 100 is *higher* (25.9 vs 7.4). At tick 100 Station's output
  tokens (3.2–3.5M on planning) are about Codex's (3.3–4.3M), with a
  fraction of its experiment time (10.7–29.8 h vs 50.2–300.5 h). So the
  baseline gap is not a compute artifact.
- **Ablation (Fig. 3A), planning task only, 3 runs, both mechanisms removed
  together.** 75.8 → 57.9, mostly through intervention findings. The ablated
  Station's 57.9 is still about three times the best baseline (19.2).
  By our arithmetic, the two mechanisms account for about 32% of
  Station's gap over that baseline on that task. The remaining 68% is
  substrate, model mix and workflow, which the paper leaves unattributed.
- **Meta Reflection behaviour (Fig. 3B).** Of the 92 events: deepening
  28.3%, validation 43.5%, pivot 14.1%, other 14.1%. There is no comparison
  window, such as random 10-tick spans without a reflection.
- **Subliminal learning (G.1).** The inverted-U with LoRA rank matches Nief
  et al., but a follow-up (Feng, LessWrong, July 2026) shows it disappears
  under rank-specific learning-rate tuning, as the appendix itself notes.
  The novel claim is that early-layer-only LoRA restores transfer. It
  replicates across 5 traits × 2 ranks × 3 seeds (cat 4.4 → 30.2%, eagle
  2.8 → 64.8%), which makes it the most robust finding in the paper.
- **VLM hallucination (G.2).** The headline in Fig. 5 is 28 → 35 of 40 on
  the agents' own 40-item set (+17.5 points). On a disjoint AMBER-1000 set,
  the *same* model (InternVL3.5-8B) moves 88.37 ± 0.83 → 89.17 ± 0.32
  (+0.8). The other models gain +3.7, +1.2 and +12.7, the last from a low
  58.5 baseline. The prose says "0.80 to 13.70", but the table gives +12.70.

## Critique / open questions

- **House-pattern check: the paper reads BELOW the digest's framing.** The
  rediscovery numbers themselves hold. 62.7% is an equal-weight task mean,
  not a best cut. The gap is large in every task and every hard category,
  and it survives compute matching even at tick 100. What is below the
  framing is the attribution and the discovery claims:
  1. **The paper's own mechanisms explain a minority of the gap.** The
     digest says "the ablation says both mechanisms together improve
     coverage". That is accurate, but it comes from one task (the
     best-scoring one), 3 runs, with both mechanisms removed jointly.
     Ablated Station still beats every baseline by about 3×. The headline
     comparison is Station-as-a-whole, three model families against one, and
     the authors themselves decline to attribute it.
  2. **The Supervisor is not a redirect mechanism.** The digest calls it
     "another external redirect mechanism". It is the opposite: a
     pivot-blocking **commitment floor**. It is forbidden to suggest
     directions and rejects almost all pivots for 60 ticks. Meta Reflection
     is the redirect-like piece, and it mostly produced *non*-pivots
     (14.1% pivot).
  3. **The exploratory "discoveries" shrink under external checks.** The
     VLM intervention's +17.5 points on 40 items becomes +0.8 on 1,000 for
     the same model. The rank inverted-U was explained away by a follow-up.
     Each task had one run and author-screened findings, with no denominator
     of how many findings were screened.
- **Contamination is argued, not measured.** Model cutoffs run from January
  2025 to January 2026, and the planning oracle paper appeared in April
  2025. The "direct prompting" control is asserted in Limitations with no
  numbers anywhere in the paper. The cross-system argument (same models,
  baselines fail) is real but weak: it shows recall is not *sufficient*,
  not that it does not contribute. Two of the 12 Meta Reflection prompts
  ask GPT-5.5 to "recall a human researcher solving a problem similar to
  the research task" and the sub-problems they solved first. On a
  rediscovery task, that is an explicit retrieval cue, and the joint
  ablation cannot separate it from the commitment effect. The highest score
  (75.8%) is on the oldest oracle paper.
- **Author proximity to the evaluation.** Stephen Chung, the second author,
  co-authored the planning oracle paper (Bush, Chung et al., ICLR 2025).
  The only human grader validation and the 24-proposal rating both come
  from "an author of the emergent-planning oracle paper", who is unnamed and
  may be Chung. The authors also built Station. Rubrics fixed before runs
  and public reports soften this, but the one task with a human-validated
  grader is also the one closest to the authors.
- **Grader validity covers one task.** 64/66 agreement on 6 outputs from
  the planning task. The low-rank rubric (claims R3/R4 with layered
  controls) and the RNN rubric are graded by GPT-5.5 with no human check.
- **Write-up instructions differ across systems, and the rubric rewards
  breadth.** Station's consolidator is told to state the strongest
  findings confidently and drop negatives. The Codex baseline's prompt asks
  for "a bounded empirical claim", adversarial audits and narrowed claims. A
  claim-coverage rubric scores the first style higher at equal science. The
  grader requires evidence, which limits but does not remove this.
- **Persistence did not buy rediscovery past tick 100.** The Supervisor's
  mechanism is longer projects, yet two-thirds of the run adds about 1.5
  points on average and loses nonlinear-RNN findings. It is not clear what
  the extra 200 ticks of enforced commitment buy on this metric.
- **What does transfer.** The negative results are candid and specific:
  75% of off-rubric proposals rated low, the streetlight effect persists,
  and agents optimise memory performance instead of explaining it because
  "a clear performance metric provides feedback". The Supervisor's prompt
  design is a clean, copyable spec for a controller-enforced minimum
  commitment, with a written pre-commit clause, a pivot request procedure
  and a record kept outside the agent.

## Trust signals

- **Credibility:** 3.
  - For: the code and all agent dialogues and reports are released in two
    repos (named in the paper; not checked here). Rubrics were fixed before
    runs and printed in full. All prompts are reproduced (Supervisor, all 12
    reflection prompts, consolidator, grader, Codex baseline). Compute is
    matched on experiment time and token usage is reported. The scale is
    real (50 runs, 3,400 h). Negatives and later refutations are disclosed
    in the appendix.
  - Against: a startup lab evaluating its own platform, with an author of
    one oracle paper on the author list and in the validation loop. Not peer
    reviewed. The ablation is joint, on one task, with 3 runs. The
    direct-prompting control is unreported. Human grader validation covers
    6 outputs on one task. The exploratory tasks are single runs with
    author-screened hits.
  - Not 4 because self-evaluation, the author conflict and the missing
    contamination numbers bear directly on the headline. Not 2 because the
    full release and pre-fixed rubrics make every claim checkable.

## Follow-up

- **Relevance:** 3. It is the first source in this graph that evaluates a
  research-agent system on metric-free tasks with a pre-fixed rediscovery
  rubric, against two named open baselines at matched experiment time. Its
  Supervisor is the graph's first measured **commitment floor**: a
  controller rule that blocks pivots. That is the counter-pole to the
  redirect-before-halt line in [[concepts/budget-as-ceiling]], which so far
  holds only redirect evidence (yan2026traceml, cai2026forkandflush,
  li2026humanize). The evidence for the floor is a joint ablation on one
  task, and the main comparison is unattributed, so it shifts framing rather
  than any accepted design. It is not a 4.
- **For `/iterate`'s redirect design.** The floor and the redirect address
  opposite failure modes: [[literature/papers/yan2026traceml]]'s
  under-pivoting Codex, and Station's premature pivot to easy questions. In
  both cases the rule worked through the controller, not the prompt. A
  design that fires a redirect on a stall should probably also enforce a
  minimum span before one is allowed. This paper does not test that
  combination.
- **For [[concepts/information-firewall]].** Reflection or "how would an
  expert do this" prompts are a retrieval channel into pretraining on any
  rediscovery or replay task. Any harness that replays published work
  should run the tool-free / direct-prompting audit and report it.
- **For [[concepts/hybrid-model-backends]].** The model-swapped reviewer
  (Meta Reflection always on GPT-5.5) is a cheap cross-family critic. Its
  value is not isolated here.
- **Related in graph:** [[literature/papers/mitchener2025kosmos]] (Kosmos,
  rediscovery plus new findings, closed source);
  [[literature/papers/wiedmann2026agents]] (agents are systems, which is the
  reading this paper's unattributed comparison demands).
- **Candidates** (cited, not in graph):
  - Chung & Du 2025, "The Station: An open-world environment for AI-driven
    discovery" (arXiv 2511.06309). The substrate, evaluated on metric tasks.
  - Kirgis et al. 2026, "Can AI agents conduct open-ended AI research? Early
    evidence from two case studies" (arXiv 2607.27191). Already listed as a
    candidate from wiedmann2026agents.
  - Nief et al. 2026, "Subliminal learning is a LoRA artifact" (arXiv
    2606.00831), and Feng 2026 (LessWrong). These are the independent check
    on the exploratory finding.
