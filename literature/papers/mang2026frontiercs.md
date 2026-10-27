---
kind: paper
title: "FrontierCS 2: A Benchmark for Computer Science R&D"
authors: ["Qiuyang Mang", "Wenhao Chai", "Runyuan He", "Dongruixuan Li", "Zihan Zheng", "Yujun He", "Xuanyi Zhou", "Hao Wang", "Siddhant Ray", "Bo Peng", "Ruyi Ji", "Shuo Yang", "Haocheng Xi", "Boru Chen", "Xinyang David Han", "Melissa Pan", "Mayank Mishra", "Boxi Yu", "Sijia Liu", "Kerem Akillioglu", "Yi Pan", "Zirui Wang", "Mert Cemri", "Diogo Antunes", "Yichuan Wang", "Aiwei Ivy Zhang", "Zhenyang Chen", "Qizheng Zhang", "Kaiyuan Liu", "Wenyuan Huang", "Sherry Ruan", "Shu Liu", "Dacheng Li", "Zerui Cheng", "Zeyu Shen", "Shang Zhou", "Hanfei Chen", "Yuzhong Zhang", "Bohan Lyu", "Qu Ao", "Han Zheng", "Chonghe Jiang", "Jingzhuo Hu", "Xiaokun Chen", "Kunjun Li", "Zhiqing Zhong", "Haoye Tian", "Tong Zhao", "Yunzhong He", "Hanchen Li", "Ofir Press", "Jiantao Jiao", "Kunle Olukotun", "Karthik R. Narasimhan", "Jingbo Shang", "Koushik Sen", "Matei Zaharia", "Alex Dimakis", "Ion Stoica", "Joseph E. Gonzalez", "Alvin Cheung"]
institutions: ["UC Berkeley", "Princeton University", "University of Waterloo", "New York University", "Georgia Institute of Technology", "University of Chicago", "University of Michigan", "University of Limerick", "Stanford University", "University of Washington", "University of Wisconsin-Madison", "UCSD", "UNSW", "MIT", "University of Pennsylvania", "Harbor Framework", "Aalto University", "Scale AI", "NVIDIA"]
year: 2026
venue: "arXiv 2610.29932 (cs.LG), v1 2026-10-21, preprint (31 pages; cover byline 'FrontierCS Team')"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.29932"
code_url: "https://github.com/FrontierCS/FrontierCS-2"
citations: null
source: "raw/papers/mang2026frontiercs.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/pass-at-k]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["benchmark", "research-agent-benchmark", "task-by-subtraction", "human-reference", "authors-code-as-bar", "private-judge", "development-suite", "held-out-final-suite", "agent-named-final", "conjunctive-pass", "score-vs-pass", "side-costs", "item-response-theory", "fci", "run-effect", "k-equals-1", "test-time-scaling", "cost-truncation", "harness-confound", "claude-code", "codex-cli", "opus-5.5", "gpt-6-astra", "systems-research"]
---

# FrontierCS 2: A Benchmark for Computer Science R&D

## TL;DR

48 tasks across 13 computer-science domains. Each is built by deleting a
published paper's contribution from the authors' own released repository;
the deleted code is the **human reference**, and pass thresholds are set
from the reference's own measurements on the paper's workloads. The agent
works for five hours in the gutted repository, may submit patches to the
allowed files at any time, and gets back a development-suite score plus a
sanitized per-metric count. The evaluator, a held-out final suite
(different seeds, queries or graphs) and the reference stay in a separate
judge sandbox. The agent names one patch as final; a run passes only if
that patch meets every metric's threshold on every held-out workload. Twelve
model+harness agents, **one run per task each**.

Three results matter for this graph:

1. **The headline is a k = 1 ranking.** Claude Opus 5.5 (Claude Code, effort
   max) passes 28/48 (58%; Wilson 95% CI about 44–71%, our arithmetic).
   GPT-6 Astra (Codex CLI) passes 21/48 (44%; 31–58%). Their FCI intervals
   overlap: Opus's lower whisker (about 46, read from Fig. 7) is below
   Astra's 50.7, and Astra's upper whisker (about 63) is above Opus's 55.7. The fitted run effect
   σu is 2.04 logits, i.e. 20 FCI points, four times the Opus–Astra gap of 5.
   No agent passes 16 of 48 tasks.
2. **A higher combined score is not a pass, and neither measure sees side
   costs.** Most runs whose final score beats the reference miss some
   metric or workload ("most" is not quantified), and some passing runs
   score at or below it. Case studies: BBQ's score ranks Opus 5.5's
   circuit first because it ignores circuit size, and the circuit does not
   serve BBQ's design sizes. QuarterPIR's best idea (1,280 vs 1,760 bytes
   per lookup) was found "only by reading the patches".
3. **The leader changes with the per-run cost budget.** Under post-hoc
   truncation, DeepSeek V4.1 Flash leads below about $0.2 per run, GPT-6.1
   Sol around $3 (35% pass at about $3/run) and Opus 5.5 at the top (about
   $20/run). Mean cost per run spans about 200×.

## Claims

- Existing open-ended benchmarks give a problem statement and grade one
  program after the run. Environment benchmarks grade against tests,
  leaderboards, rubrics or reimplementations. "None of these benchmarks sets
  its bar by measuring the original authors' code." MLS-Bench is the
  closest design (140 ML tasks, improve one named component); FrontierCS 2
  extends it beyond ML.
- Subtraction gives each task "an oracle that already runs in its
  repository on the paper's own workloads."
- Choosing the final patch "is part of a researcher's work", so the judge
  does not pick the best development submission.
- "A higher score is not the same as a pass." Benchmarks like this "will
  need to report such side costs next to the pass."
- FCI/FDI (an IRT model adapted from Epoch's Capabilities Index) are "the
  measures we recommend": they place agents and tasks on one scale, use
  every metric outcome and score margin, and come with intervals.
- The median task sits at about the strongest agent's FCI.
- A private set of more than 100 further tasks exists, "to be announced".

## Methods

- **Task building.** Locate the code implementing the paper's contribution,
  delete it, leave dependencies unchanged; deleted functions stay declared
  and called. The agent may edit only the files that held the
  contribution. One exception: Structured LWE is an original cryptanalysis
  challenge on 200 public instances, so its final suite has no held-out
  part. The 48 tasks come from 45 papers (three FlashLib tasks share one
  source, two FARGO tasks another).
- **Domains.** Database 9, Machine Learning 9, MLSys 6, HPC 4, Programming
  Languages 4, Architecture 3, Cryptography 3, Network 3, Security 2,
  Robotics 2, Bioinformatics 1, Software Engineering 1, Theory 1. So 15 of
  48 are ML or MLSys, the part closest to this project's ML-research focus.
- **Thresholds.** Per metric, from the reference's measurements. SVG-EAR:
  latency ≤ 0.96× reference, PSNR within 0.25 dB, SSIM within 0.01, LPIPS
  within 0.005. So "beats" means "meets the threshold". For quality metrics
  that is a tolerance band around the reference, not an improvement.
- **Judge (Fig. 4).** Separate sandboxes for agent and judge, linked by a
  per-run driver. The agent sandbox has no network except the driver, so it
  cannot download the authors' code. On GPU tasks the agent gets its own
  development GPUs; only the judge's GPU measures against the reference.
  Per patch the agent sees the development score and a message counting
  development workloads that meet each metric's threshold. It does not see
  the reference score or raw evaluator stdout/stderr. Development
  submissions are capped per task (3–100, Table 3); final-eval time limits
  run from 2 minutes to 30 hours.
- **Reference re-measured per run.** The reference score used in the fit is
  "the median, over the task's runs, of the human reference's final score
  measured within each run" (App. D Step 4).
- **Agents (Table 4).** Twelve models, each in one harness: Claude Code
  2.1.296 (Opus 5.5, effort max), Codex CLI (GPT-6 Astra and GPT-6.1 Sol,
  xhigh), Pi (Gemini 4 Argon, high), ZCode, Muse Code, DeepSeek Harness,
  Kimi Code, Qwen Code, Step Code. Muse Code adds side calls that watch the
  main agent and remind it. A 13th agent (Kimi K2.7) only anchors FCI = 0.
  "The harness is part of what we measure."
- **Runs.** One per agent per task: 13 × 48 = 624 runs in the fit. Two
  Gemini 4 Argon runs were refused by the provider and count as failures;
  so does any run without a saved final submission.
- **FCI/FDI (App. D).** Joint fit to per-metric-group outcomes (115 groups
  after merging metrics that every run beats or misses together) and to a
  normalized score margin over the reference. Runs that fail but score above
  the reference enter only as "margin ≥ 0". A per-run random effect
  u ~ N(0, σu²) is shared by both parts. MAP by Newton; hyperparameters by
  Laplace-approximate marginal likelihood; 90% intervals from 500 task
  bootstraps. FDI is the FCI at which an agent passes the task half the
  time.
- **Test-time scaling.** At budget B, a run that finished within B keeps its
  result; a run still going counts as having no final submission (score 0).
  Agents were not told B. The rule deliberately avoids development scores,
  because they "swing": the judge gives 0 on any correctness or quality
  failure, "yet most runs that drop to 0 after a positive score recover
  above 0 later".

## Results

- **Pass rate (Fig. 2, n = 48 each, k = 1).**

  | Agent (harness) | Pass | Wilson 95% (ours) | Mean margin | FCI | ~Cost/run |
  |---|---|---|---|---|---|
  | Opus 5.5 (Claude Code) | 58% (28) | 44–71 | +9.4 | 55.7 | ~$20 |
  | GPT-6 Astra (Codex) | 44% (21) | 31–58 | +7.1 | 50.7 | ~$17 |
  | Gemini 4 Argon (Pi) | 40% (19) | 27–54 | +2.1 | 39.5 | ~$9.5 |
  | GPT-6.1 Sol (Codex) | 35% (17) | 23–50 | +5.7 | 42.6 | ~$3 |
  | GLM 5.3 (ZCode) | 25% (12) | 15–39 | +1.8 | 36.6 | ~$10 |
  | Kimi K3 / Muse Wonder 1.0 | 21% (10) | 12–34 | −1.2 / +4.1 | 26.4 / 34.5 | ~$8 / ~$13 |
  | DeepSeek V4.1 Flash | 15% (7) | 7–27 | +0.5 | 30.7 | ~$0.11 |
  | Muse Spark 1.3 / Qwen 3.8 Max / GLM 5.3 Flash | 12% (6) | 6–25 | −3.3 / −6.4 / −8.4 | 22.1 / 21.9 / 22.9 | $1.4–$8 |
  | Step 5 Preview | 8% (4) | 3–20 | −10.6 | 9.8 | ~$7.5 |

  Pass counts are inferred from the percentages at n = 48. Costs are read
  off Fig. 5's log axis. DeepSeek's costs are lower bounds.
- **FCI intervals (Fig. 7)** "separate only agents that sit far apart".
  Pass rate and FCI order the agents almost alike. Gemini passes more often
  than Sol but sits below it on FCI.
- **Fitted noise (Table 5).** σu = 2.04 logits (20.4 FCI points); group
  difficulty spread τ = 3.95 logits; margin noise σe = 2.32 units.
- **Score vs pass (Fig. 6).** Per-run margins span roughly −100 to +60 for
  most agents. Most above-reference scores are non-passes (no count given).
- **Case studies (one run per agent).**
  - *QuarterPIR*: Muse Wonder 1.0 replaces per-group parity values with two
    RAID-6-style checksums: 1,280 bytes per lookup against the reference's
    1,760. The other passing runs sit at 1,664–1,752. GPT-6.1 Sol scores
    slightly higher than Muse Wonder with a far smaller saving.
  - *BBQ (FPGA priority queue)*: only Opus 5.5, Gemini 4 Argon and GPT-6.1 Sol
    pass. Opus moves every counter into registers: shortest clock period,
    among the largest circuits, and it "does not serve trees as large as
    those BBQ was designed for". The score ranks it first "because it
    rewards speed and ignores circuit size".
  - *SVG-EAR*: no run passes. GPT-6 Astra measures attention error online
    instead of predicting it, and is faster (729 vs 891 s per clip) and
    closer to full attention on average (PSNR 23.8 vs 21.7). It still fails
    image-quality bands on two clips. Most runs beat the reference's time;
    no run stays within every band.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  entry is in the paper, and "the HCE split as a benchmark design" is
  accurate. Three things the entry leaves out narrow it.
  1. Every number is k = 1. 58% vs 44% is 28 vs 21 tasks with overlapping
     binomial and FCI intervals. The model's own run effect (20 FCI points)
     is larger than every adjacent gap in the top five.
  2. "Beats the score but not a pass" is not a declared-vs-verified datum in
     this graph's sense. Both numbers come from the same judge on the same
     held-out run: one scalar and one conjunction of thresholds. No agent
     self-report is measured. It is a scalar-vs-conjunctive-oracle datum,
     and "most" is never counted.
  3. Each model runs in a different harness at a different effort setting.
     The ranking is of model+harness bundles, as the paper says, so it says
     nothing about Opus 5.5 vs GPT-6 Astra as models.
- **What the judge design gets right (for [[concepts/hce-evaluation]]).**
  - The evaluator, held-out suite and reference live in another sandbox,
    so clause 4 (evaluator is part of the holdout) holds by placement, not
    by hash.
  - The agent names the final patch, and the judge scores it once on unseen
    instances (clause 1).
  - Raw stdout/stderr is withheld from the feedback message, which is the
    return-path repair [[literature/papers/farahbakhsh2026agentdiscover]]
    called for.
  - The reference is re-measured inside every run, a per-run known-good
    measurement on the same hardware.
- **What it does not report, and should.**
  - *Development-to-final gap.* Development scores exist for every patch,
    and dev feedback is the channel an agent can overfit. The paper never
    reports how many runs met every threshold on the development suite and
    then failed the final, or how often the named final was the best
    development patch. It has both numbers in its logs.
  - *Reference run-to-run spread.* Thresholds like latency ≤ 0.96× are set
    from the reference, and the reference is measured in every run. Its
    spread across runs is never shown. Without it one cannot tell whether a
    4% latency margin over many workloads is a real bar or partly a coin
    flip (cf. [[literature/papers/ding2026reexecuting]] on grader-side
    noise).
  - *Gaming audit.* There is no reading of trajectories for evaluator
    probing, special-casing of development workloads, or test-sandbox
    exploits. DeltaML-Bench ([[literature/papers/moukpe2026deltaml]]) used
    the same "improve inside the paper's repo" setting and found gaming on
    up to 47.9% of tasks under one scaffolding. Held-out instances catch
    hard-coding of development inputs, but the paper does not say whether
    patched code running in the judge sandbox can read the held-out
    workloads or the reference. That is the candidate-runtime leak in
    [[concepts/information-firewall]].
  - *Contamination.* The deleted code is public, and many sources are
    years old (Edlib 2017, egg 2021, Diffusion Policy 2023, BBQ NSDI'24).
    Network isolation blocks download, not recall. Our inference: recall
    alone would tie the reference. That meets the tolerance-band metrics
    but not strict "beat" thresholds like 0.96× latency, so memorization
    plausibly helps on quality metrics more than speed. No recall probe is
    run.
  - *Do-nothing floor.* The starting repository is a natural known-bad
    submission, but its score is not reported.
- **Selection is inside the measurement.** Making the agent name its final
  patch is a defensible spec choice. But it means pass rate = generation ×
  selection, and the paper does not separate them. That is the
  [[concepts/pass-at-k]] selector point
  ([[literature/papers/iyengar2026sample]]): best-dev-patch pass rate vs
  named-final pass rate would show how much each agent loses to its own
  selection.
- **Test-time scaling is post-hoc truncation, not a budgeted run.** Agents
  never saw B, so a run cut at B is one that would have submitted later, not
  a budget-aware policy. Counting it as a 0 is the right number for "what
  you get if you kill it at B". It is the wrong number for "how would this
  agent do with B". Contrast [[literature/papers/hickey2026saltbench]],
  which scores a halt as unresolved rather than failed.
- **The dev-score swing remark is a useful datum with no count.** "Most
  runs that drop to 0 after a positive score recover above 0 later". If it
  holds, a stall or regression halt keyed on the development score would
  kill recoverable runs. No numbers or denominators are given.
- **IRT on 48 tasks × 13 agents × 1 run.** The fit has 13 strengths, 163
  task parameters and 5 hyperparameters on 624 runs. The 90% intervals come
  from task bootstrap with hyperparameters held fixed, so they understate
  uncertainty. Every task's FDI is too uncertain to order (the paper says
  so). "The median task sits at about the strongest agent's FCI" is
  therefore a statement about a noisy distribution.
- **Authors built tasks from their own papers.** SVG-EAR, Continuum, FlashLib
  and Flash-KMeans share authors with this paper. MLS-Bench, the closest
  prior design, shares authors too (Lyu, Han). That probably improves task
  fidelity. It also means part of the bar is the authors' own code, and
  thresholds were set by people who know where its slack is. No
  conflict-of-interest discussion.
- **Relation to NatureBench.** [[literature/papers/wang2026naturebench]]
  strips the source method so agents must invent rather than re-implement,
  and its best agent (Opus 4.7 + Claude Code) surpasses SOTA on 17.8% of 90
  tasks, also k = 1. FrontierCS 2 is the same subtraction idea with the
  authors' code as an executable oracle in place of a reported number. The
  two agree that the frontier sits at the median task, not the tail.

## Trust signals

- **Credibility:** 3.
  - For: a large, identifiable systems group (mostly UC Berkeley, with
    Princeton, Stanford, MIT, Georgia Tech, Scale AI) and senior
    advisors (Stoica, Gonzalez, Zaharia, Narasimhan, Press). Code and
    website are released. The judge architecture is described concretely,
    and Appendix D gives the fit step by step with fitted values. The
    paper is candid that the harness is part of what is measured, that FDI
    cannot order tasks, and that FCI intervals separate only distant
    agents. Refused runs count as failures rather than being dropped. It is
    a follow-up to an existing benchmark (FrontierCS, 2512.15699).
  - Against: preprint, one run per agent per task, harness and effort vary
    with the model, "most" left uncounted for the score-vs-pass claim, no
    dev-to-final gap, no reference noise, no gaming audit, no contamination
    probe. Case studies are single runs chosen after the fact. Effort settings
    differ (Opus 5.5 max, GPT agents xhigh, Gemini 4 Argon high).
  - Not 4 because the headline ranking rests on k = 1 with overlapping
    intervals, and the benchmark's integrity (gaming, leakage into the judge
    sandbox, recall) is asserted by architecture, not measured. Not 2
    because the design is released, structural and specified well enough to
    audit, and the statistical treatment is more careful than most k = 1
    benchmarks in this graph.

## Follow-up

- **Relevance:** 4. It is the cleanest benchmark-scale instance in the graph
  of the HCE split run structurally across many tasks: agent-side dev
  feedback, judge-held evaluator, held-out final, agent-named final, network
  isolation. It is also the first in the graph to set the bar as the
  authors' own code measured in the same run. It adds a scalar-vs-
  conjunctive-pass dissociation and two side-cost blind spots to
  [[concepts/programmable-evaluator-oracle]], a fitted run-effect size to
  [[concepts/pass-at-k]], and a budget-dependent leader to
  [[concepts/budget-as-ceiling]]. Not 5 because it measures agents, not
  loop policy, only 15 of 48 tasks are ML/MLSys, and the integrity
  questions this project cares about most are not measured.
- **For downstream projects (our inference).** If a downstream project runs
  FrontierCS 2, log per run: (a) best-dev vs named-final outcome; (b) the
  reference's in-run score, to build its noise band; (c) the starting
  repository's score as the do-nothing floor. All three come from the judge
  the benchmark already ships.
- **Ask of the authors.** Report dev-pass→final-fail counts, the
  reference's per-run spread per metric, and the count behind "most runs
  whose final score exceeds the human reference's miss a threshold".
- **Candidates** (cited, not in the graph):
  - Lyu et al. 2026, MLS-Bench (arXiv 2605.08678). Closest prior design, 140
    ML tasks, improve one named component.
  - Garikaparthi et al. 2026, ResearchGym (arXiv 2602.15112). Also removes a
    paper's method from its repository.
  - Wijk et al. 2024, RE-Bench (arXiv 2411.15114). Agents vs human experts at
    equal time budget.
  - Wang et al. 2026, RSI-Forge (arXiv 2610.09426). Papers → environments,
    baseline is an agent reimplementation.
  - Liu et al. 2026, "When agents slow down: Elo-per-token analysis"
    (arXiv 2609.15309). Test-time strategy and cost, same group.
  - Ho et al. 2025, "A Rosetta Stone for AI benchmarks" (arXiv 2512.00193).
    The Epoch Capabilities Index that FCI adapts.
  - Mang et al. 2025, FrontierCS (arXiv 2512.15699). Version 1.
  - Zhu et al. 2026, EdgeBench (arXiv 2607.05155). In-run feedback, scaling
    laws from real environments.
