---
kind: paper
title: "Autonomous Research for Open-Ended Problems: A Case Study on Telecom Ticket Retrieval"
authors: ["Junghyun Min", "Huseyin Uzunalioglu", "Mohamed Trabelsi"]
institutions: ["Georgetown University", "Nokia Bell Labs"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.13073"
code_url: null
citations: null
source: "raw/papers/min2026autonomous.pdf"
added: "2026-09-15"
relevance: 4
credibility: 2
status: read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-search-grain]]"
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/budget-as-ceiling]]"
tags: ["ml-research-agents", "autoresearch", "search-space", "industrial-case-study", "retrieval", "multi-agent", "harness", "negative-result"]
---

# Autonomous Research for Open-Ended Problems: A Case Study on Telecom Ticket Retrieval

## TL;DR

A Nokia Bell Labs team points Karpathy-style autoresearch loops and a
multi-agent AutoScientists team at an industrial retrieval problem whose
search space goes beyond hyperparameters: document representation,
training-pair generation, loss and architecture. Across 8 campaigns and 3
agents (Claude Sonnet 5, Cursor Composer 2.5, local GPT-OSS 120B), the best
system is a **single fine-tuned MPNet at R@1 0.343**. That beats the human
fine-tuned model (0.200) and the human ensemble (0.251), but not the internal
SoTA (**0.380**), which uses re-ranking, LLM data augmentation and
ensembling. **No campaign attempted any of those three, even when the
documentation named them.** The agents found real wins *inside* the declared
axes, such as 2× oversampling of ticket-to-ticket pairs, and never stepped
outside them. Agent choice, team structure and informedness are all reported
as having "negligible impact". The paper's own Table 3 does not support that
for team structure.

## Claims

- **Autonomous research got to ~90% of SoTA in far less time.** R@1 0.34 vs
  0.38. It took 10 weeks end to end versus ~10 months of human work, though
  the authors say the two efforts are not independent.
- **Agents are good at "deep, narrow hyperparameter sweeps" but lack
  "human-like intuition or creativity."** Follow-ups are arbitrary rather
  than analysis-driven: going from 1 to 2 epochs instead of looking at the
  loss curve, and from 256 to "an arbitrary 384" tokens instead of looking
  at the length distribution.
- **Documentation does not induce structural change.** "Providing human
  insight in the form of documentation is insufficient to induce
  'outside-the-box' architectural changes." The best run was informed, but
  per footnote 7 "the agent did not make use of this information."
- **Without an explicitly declared search space, agents default to learning
  rate and epochs.** Declaring the space as pre-set variables fixes that, at
  a cost: "agents primarily toggle pre-defined candidate options rather than
  implementing novel code modifications."
- **None of the design axes moves the peak.** The authors report no
  substantial effect from single vs multi-agent, from model reasoning tier
  (Sonnet 5 > Composer 2.5 > GPT-OSS was the prior), or from informed vs
  uninformed.
- **Agents cannot keep themselves running.** Prompting them to "never stop"
  and Python heartbeat controllers both "proved unreliable", leaving stalled
  agents or redundant experiments. The fix is a deterministic outer bash
  `while` loop that resumes the agent session until a human stops it.
- **Cost is modest.** An estimated $150–200 per 10–20 experiment Cursor
  campaign, 2–3× that on Claude ($300–600), and ~$0 marginal on local
  GPT-OSS. These figures exclude the repository setup and harness
  development.

## Methods

- **Task.** Telecom ticket retrieval over 250k tickets (T), 204k fault
  analyses (FA) and 89k technical analyses (TA), grouped into incident
  clusters. A hit means the query and the retrieved document share a
  cluster. The evaluation has 7.6k held-out queries and 1.2k held-out gold
  documents, which compete against the whole corpus. The primary metric is
  R@1. The formal search space is joint over preprocessing P, sampling
  hyperparameters λ, and encoder weights θ.
- **Frameworks.** (a) A single-agent loop extended from Karpathy's
  autoresearch: propose, run, and commit if the metric improves, otherwise
  revert. (b) A multi-agent team extended from AutoScientists: an
  orchestrator plus sub-agents on a ClawInstitute message board, split into
  three teams (representation, training-data generation, and
  architecture & hyperparameters).
- **Harness, arrived at by trial and error.** (1) Task and environment
  documentation, including hardware and an instruction to maximize GPU use.
  (2) An explicit search-space definition: team assignment in the
  multi-agent setup, pre-set codebase variables in the single-agent one.
  (3) The scripted outer loop described above. The authors deliberately did
  **not** put the SoTA components into the declared space, because that
  "would defeat the purpose" of discovery.
- **Grid.** 3 axes: structure (2), agent (3), informedness (2). Only **8 of
  the 12 cells** were run, **one campaign per cell**, with 4 to 132
  experiments each. There are no seeds and no repeats.
- **Compute.** 4× RTX A6000 (48 GB) servers. The best 17-experiment
  campaign used about 1 GPU-week.

## Results

- **Table 2.** BM25 0.072; fine-tuned MPNet 0.200; human ensemble 0.251;
  internal SoTA 0.380 (R@10 0.783); discovered model 0.343 (R@10 0.630,
  R@50 0.768). The R@10 gap to SoTA (0.15) is much wider than the R@1 gap
  (0.04), so "90% of SoTA" holds only at k=1.
- **Table 3, best R@1 per campaign.**
  - Single-agent: Cursor uninformed 0.274 (33 exps); Cursor informed
    **0.343** (17); Claude uninformed 0.338 (9); GPT-OSS uninformed 0.335
    (12); GPT-OSS informed 0.324 (13).
  - Multi-agent: GPT-OSS uninformed 0.204 (4, orchestration failed);
    Cursor uninformed 0.265 (**132**); Claude informed 0.240 (18).
- **What the winning run found** (Appendix A). Fields are concatenated with
  explicit markers. Boilerplate is removed with a filter on lines that recur
  in ≥5 documents. T-to-T peer pairs are added and oversampled 2× alongside
  T-to-FA pairs, giving 760k instances. One hard negative is mined per pair
  from out-of-cluster tickets with the same product/feature metadata.
  Training: all-mpnet-base-v2, 2 epochs, batch 32, lr 2e-5, max length 384,
  MNRL with scale 20. The T-to-T pairing is a genuine data-generation
  finding that the human comparison systems lacked. It sits entirely inside
  the declared "training data generation" axis.
- **Model search was conservative too.** Single-agent campaigns started
  from MPNet and multi-agent campaigns from MiniLM, and "both rarely
  proposed" a larger encoder. The only other encoder tried was BGE.
- **Each agent failed in its own way.** All of them under-used GPU memory
  by proposing batch sizes smaller than would fit, and all of them went idle
  or stopped early. Cursor re-ran duplicate experiments. GPT-OSS could not
  spawn sub-agents ("succeeding only once across dozens of attempts"), and
  Qwen Coder 30B failed the same way. Claude sometimes debugged environment
  scripts instead of running experiments.

## Critique / open questions

- **Table 3 contradicts the claim that framework structure doesn't matter.**
  Every multi-agent campaign (0.204, 0.240, 0.265) scores below every
  single-agent campaign (0.274–0.343). The best-to-best gap of 0.08 is
  twice the gap to SoTA. The comparison is also confounded: multi-agent runs
  started from MiniLM-L6 and single-agent runs from MPNet, so the base
  encoder may explain the gap. The data therefore support "untested", not
  "no effect".
- **The noise floor swallows every factor.** The one repeated cell pair,
  single-agent Cursor informed vs uninformed, differs by **0.07**. The
  authors attribute that to nothing, since the agent ignored the
  information. It is larger than the whole cross-model spread among
  single-agent campaigns (0.324–0.343). With one campaign per cell, no
  factor claim survives: the null results on model tier and informedness
  are as unresolved as the positive ones would be.
- **"Documentation doesn't induce structural change" is partly built into
  the harness.** The declared search space, which the paper says is what
  pulls agents off lr/epochs, excluded ensembling, re-ranking and
  augmentation on purpose. The agents were told about those components but
  steered along axes that didn't contain them. The obvious ablation, adding
  them to the declared space, is the one the authors decline to run. The
  finding is really that **the declared space binds and informational
  context does not lift it**. That is narrower than "agents lack
  creativity", and more useful.
- **Tension with [[literature/papers/chi2026ai4ai]].** There, raising
  reasoning effort moved agents that attempt algorithmic change from 8% to
  64%. Here model tier made no difference. The two don't conflict: chi
  varied effort within a model on a benchmark that scores *only* structural
  change, while min varied the model inside a harness whose declared axes
  left out the structural class. Read together, the space definition may
  dominate willingness.
- **Autoresearch's keep/revert loop is greedy and single-lineage.** It
  keeps only the best state, so a system-level change that first *drops*
  R@1, such as adding a re-ranker stage before it is tuned, is reverted on
  arrival. The paper doesn't discuss this. It is a structural reason, apart
  from model creativity, why a greedy loop would stay local.
  [[literature/papers/zou2026fmlbench]] shows greedy is competitive on dense
  improvement landscapes, and this task's dense axes are exactly the ones
  that got explored.
- **The cost numbers are estimates, not metered spend.** They come "due to
  the structure of the group's billing system", leave out harness
  development (itself "many trial-and-error loops"), and leave out the
  human "babysitting" the paper documents. The "up to $200" in the abstract
  is the Cursor figure; Claude ran $300–600.
- **The "10 weeks vs 10 months" figure is a framing device.** The agents
  started from the human team's documentation, dataset, codebase, evaluation
  pipeline and baselines, and a domain expert monitored and stopped every
  campaign. The authors say so in their Limitations.
- **Nothing is reproducible from outside.** The dataset is proprietary, the
  SoTA system is unpublished, there is no code release, and the campaign
  logs and trajectories are not released. The per-experiment trajectory
  isn't reported either, so it can't be checked whether the 132-experiment
  campaign plateaued early.
- **Open question.** Does putting the structural components into the
  declared search space (a "system composition" team, or ensemble and
  re-ranker variables) get agents to try them, and does a non-greedy loop
  keep them long enough to tune? This is the ablation that would separate
  "harness boundary" from "agent creativity".

## Trust signals

- **Credibility:** 2. An arXiv preprint (cs.AI) from Nokia Bell Labs, a
  reputable industrial lab, with the first author a Georgetown intern. Not
  peer-reviewed, and no citations established. The lab prior would support
  3, but reproducibility is essentially zero: proprietary data, an
  unpublished SoTA comparator, no code and no campaign logs. The evidence is
  8 uncontrolled single-shot campaigns, spend is estimated rather than
  metered, and the headline "framework structure had no substantial impact"
  is contradicted by the paper's own Table 3 and confounded with base
  encoder. The Limitations section is candid about the uncontrolled design
  and the dependence on human work, which keeps it from dropping to 1.

## Follow-up

- **Relevance:** 4. It supplies the evidence
  [[concepts/evolutionary-search-grain]] lacked on **where agentic search
  stops being narrow enough**, and it adds a grain axis that concept doesn't
  have: in a research loop, the effective grain is the **declared search
  space**, not the span of code the agent may edit. The single-agent loop
  could edit the whole repository, yet without declared axes it collapsed
  to lr/epochs. With declared axes, it toggled options. It never crossed
  from component-level to system-composition changes, even when told what
  those were. This is the research-design analogue of the operator
  attractor in [[literature/papers/gurkan2026mutation]]. It also adds a
  third halting failure to [[concepts/budget-as-ceiling]]: agents stop
  *early*, despite instructions not to. It stays at 4 rather than 5 because
  the evidence is one uncontrolled industrial case, and it seeds no new
  concept.
- **[[concepts/evolutionary-search-grain]].** FunSearch's "skeleton is a
  hard ceiling on the solution structure" is here observed in an LLM
  research loop, with one addition: telling the agent what lies beyond the
  skeleton doesn't lift the ceiling. Guidance 1's pairing of pipeline-level
  problems with whole-file grain gets a qualifier: a larger grain *permits*
  structural change but does not *produce* it.
- **[[concepts/evolutionary-expansion]].** Weak, source-only. The
  multi-agent team is breadth by partitioning the search space, not a
  population, and its discussions "rarely introduced effective experimental
  proposals beyond parameter tuning". The comparison is confounded with base
  encoder, so it doesn't test the concept's claim that population machinery
  supplies variation.
- **[[concepts/budget-as-ceiling]].** The paper's scripted outer loop hands
  *continuation* to deterministic code because the agents' own judgment
  about whether to keep going was unreliable. That is the concept's
  definition ("not because a heuristic decided the work looked done") seen
  from the premature-stop side. The loop has no ceiling, though: a human
  ends every campaign, and the one 132-experiment campaign scored below
  three 9–17-experiment campaigns. Experiment count did not buy peak
  performance.
- **[[concepts/spend-forecast-calibration]]: declined.** There is no
  forecast, only a retrospective per-campaign estimate that leaves out
  harness development. At most it is another example of the
  denominator-excludes-setup problem the concept already covers
  ([[literature/papers/panigrahy2026energy]]).
- **Independence.** This is the second group, after
  [[literature/papers/chi2026ai4ai]], to find agents defaulting to
  non-structural change. The settings differ (an industrial open-ended task
  vs a frozen-repo benchmark), so it counts as independent convergence. It
  is the first source in the graph where a greedy autoresearch loop runs on
  a real industrial problem with a human SoTA to compare against.
