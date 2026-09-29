---
kind: paper
title: "Analyzing and Mitigating Cost-Inefficient Behaviors in Coding Agents"
authors: ["Yiran Hu", "Nan Jiang", "Shanchao Liang", "Anik Dey", "Yi Wu", "Lin Tan"]
institutions: ["Purdue University"]
year: 2026
venue: "arXiv (cs.AI; cs.SE); preprint under review (ICLR-style AI-use statement)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.30725"
code_url: "https://anonymous.4open.science/r/coding-agent-cost-study-4B4A"   # anonymized review repo; may move
citations: null
source: "raw/papers/hu2026analyzing.pdf"
added: "2026-09-29"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/scripted-tool-pipelines]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["cost-efficiency", "trajectory-analysis", "claude-code", "mini-swe-agent", "swe-bench", "redundant-retrieval", "subagent-return-loss", "agent-skills", "human-vs-synthesized-skills", "trace2skill", "structure-aware-retrieval", "noise-floor", "run-to-run-variance", "held-out-split"]
---

# Analyzing and Mitigating Cost-Inefficient Behaviors in Coding Agents

## TL;DR

The authors mine 1,200 SWE-bench Verified trajectories (300 tasks × 4
configurations: Claude Code v2.1.169 with Sonnet 4.6 + Haiku 4.5 subagents,
and Mini-SWE-Agent with Sonnet 4.6, MiniMax-M3 or Qwen-3.5 Plus). They
define three detectable wastes: **subsumed retrieval** (re-reading code a
prior read fully covered), **similar script generation** (near-duplicate
scripts, Jaccard ≥ 0.60), and **test re-execution** (same test, no patch in
between). They then test three fixes on a date-split held-out set
(Verified-200) and on Pro-100. The fixes are CodeGraph (structure-aware
retrieval), agent-synthesized skills (Trace2Skill) and seven
developer-written principles. Wastes are near-universal but cheap on
Claude Code: **79.00% of tasks and 6.86% of cost**, against 96.67–98.00%
and 21.16–22.75% on Mini-SWE-Agent. CodeGraph *raises* cost in four of
eight settings. Developer skills robustly cut cost in six of eight
settings (7.88–41.73%). Synthesized skills do so in three (8.86–22.32%),
and after amortizing their synthesis cost almost none survive. The
unadvertised contribution is the method: triplicated baselines, a derived
single-run robustness threshold (|Δ| ≥ 1.90 s), and a validation that the
noise floor transfers. It is the most fully specified calibrated noise band
in this graph.

## Claims

- **Three wastes, near-universal.** Together they touch 79.00–98.00% of
  tasks and cost 6.86–22.75% of task cost (Table 1). Subsumed retrieval is
  the most prevalent, at 64.33–92.33% of tasks and 5.01–11.41% of cost.
- **Causes are architecture-specific.** On Claude Code, **50.15% of
  subsumed retrieval is Cross-Agent**: the main agent re-reads code a
  subagent already read, "because subagents return summaries rather than
  the retrieved code". On Mini-SWE-Agent the causes are low-feedback
  editing (`sed -i` failing silently), `cat`-then-zoom localization (no
  line numbers), and long debugging loops.
- **Claude Code's system prompt already suppresses some of this.**
  Similar-script generation occurs 5.91–9.98× more often on Mini-SWE-Agent.
  CC's "NEVER create files unless they're absolutely necessary" pushes it
  to ephemeral `python -c` probes (0.21 draft files per task vs 2.72 to
  7.07), and those probes get regenerated. **No built-in instruction
  covers test re-runs**, and ReTest is the one behavior where CC shows no
  advantage (2.09 per task vs 2.51 / 2.10).
- **Structure-aware retrieval is not inherently cheaper.** Under CC,
  CodeGraph cuts subsumed retrieval by more than 75%, yet cost rises by
  8.30% (Verified) and robustly by 12.19% (Pro). Each CodeGraph query
  returns 8.2–16.6× more tokens, and **Haiku subagent calls drop from
  4.81 and 11.03 per task to zero**. The same token volume moves to Sonnet,
  "whose token price is 3× that of H45". Under MSA, agents use it on top of
  ordinary retrieval rather than instead of it.
- **Developer-written skills beat agent-synthesized ones.** Robust cost cuts
  in six of eight settings (7.88–41.73%) vs three (8.86–22.32%). The
  authors offer "one possible mechanism": seven trace-agnostic principles
  vs 23–41 trace-specific operational rules.
- **Task-cost change is larger than behavior-cost change.** Fitted slopes
  are 2.83 (Verified) and 1.33 (Pro). The attributed reason is that a
  prevented action also shortens the trajectory, which saves cache reads,
  and removes the reasoning that would have followed.

## Methods

- **Split.** Within each repo, SWE-bench Verified tasks are ordered by
  creation date. The earliest ~60% (300) are for analysis; the rest
  (Verified-200) are held out. Pro-100 is a stratified sample that
  over-samples costly tasks (terciles at 20/30/50%). CodeGraph ran on only
  87 of the Pro tasks, because 13 failed (mostly Alpine/musl images).
- **Labeling and detection.** 18 action labels are assigned by rules, with
  an LLM fallback (GPT-5.1) for 7.26% of steps overall and 31.17% of
  MSA-S46 steps. The detectors are thresholded and "manually calibrated"
  (e.g. 0.60 Jaccard, from 20 pairs per configuration). Cost is attributed
  per flagged action with the o200k tokenizer at list prices.
- **Skills.** SynSkills are built per configuration. A ReAct analysis agent
  handles the top 25% of trajectories by waste share, and a single LLM
  call handles the rest. Trace2Skill hierarchical consolidation then
  produces 23–41 rules. DevSkills are seven principles the authors wrote
  from the RQ1 findings, shared across configurations: hypothesis-before-read,
  reuse context, targeted patch, understand the test harness, capture test
  output and filter instead of rerunning, persist artifacts, break loops.
  All skill sets are **preloaded into the system prompt**, not loaded on
  demand.
- **Noise control (App. A.1.4), the part worth copying.** Triplicating
  every cell would have cost about $7,500. Instead, all 8 baselines run
  3×, and floors are set from them: SD of Pass@1 and CV of cost and CoP.
  A change is *robust* when |Δ|/(s·√(1/R + 1/3)) ≥ 1.645, a 95%
  sign-replication criterion, which is **|Δ| ≥ 1.90 s** for a single run.
  Equal-variance transfer was checked on 9 triplicated approach cells: cost
  SD is 0.28–1.38× the baseline in 8 of 9. The exception (CodeGraph,
  MSA-Q35+, Pro) gets its own 2.39× floor. Two robustness checks: every
  verdict survives a 1.5× floor inflation, and the expected number of sign
  reversals over the 13 robust cost effects is 0.06. **Behavior counts
  carry no verdicts**, because their per-task CV averages 15.68% (max
  44.47%) and exceeds the cost floor in 20 of the 24 combinations.

## Results

- **Noise floors for SWE-bench-scale coding agents (Table 7).** Pass@1 SD
  is 0.87–4.00 pp and cost CV is 2.44–8.35% at n = 100–200 tasks. The
  minimum robust single-run cost change is 4.64% (CC, Pro) to 15.87% (CC,
  Verified). Claude Code is the most stable configuration on Pass@1 and,
  on Verified, the noisiest on cost.
- **RQ1 baselines (Table 9).** CC 76.67% at $0.520 per task; MSA-S46 74.33%
  at $0.554; MSA-Q35+ 65.67% at $0.102.
- **Claude Code rows specifically.**

  | CC | Verified-200 cost | Pro-100 cost |
  |---|---|---|
  | CodeGraph | +8.30% (noise) | **+12.19%** |
  | SynSkills | −1.38% (noise) | **−8.86%** |
  | SynSkills + amortized synthesis | +12.51% | −0.49% |
  | DevSkills | **−13.94%** (Pass@1 −1.50 pp) | −1.51% (noise) |

  On Claude Code neither skill type wins on both benchmarks.
- **Amortization (Table 15).** The authors add synthesis cost ($22.76 for
  CC, $37.59 for MSA-S46) spread over 300 tasks. After that, SynSkills keep
  one robust reduction (MSA-MM3 Pro, −9.65%), turn into robust increases on
  MSA-Q35+, and the −22.32% headline cell shrinks to −2.70%. The authors
  note the surcharge would fall below 2.00% over 3,000 tasks.
- **Where re-reads come from.** 11.05–18.10% of all retrieved lines are
  re-reads. Volume doesn't predict the share: MSA-S46 reads the fewest
  lines (573) but re-reads 16.69%; CC reads 1,032 and re-reads 11.05%.

## Critique / open questions

- **"Twice" is one cell.** 41.73% vs 22.32% are both MSA-S46 on Verified.
  On Claude Code, DevSkills win on Verified and SynSkills win on Pro. The
  mechanism ("trace-agnostic vs trace-specific") is offered as "one
  possible mechanism". It is confounded with **rule count (7 vs 23–41)**,
  all preloaded, which is the flat-loading dilution kim2026why measured.
  It is also confounded with **authorship after full analysis**: the
  developers wrote their skills having seen every RQ1 finding across all
  four configurations, while each SynSkill set saw one configuration's
  diagnoses.
- **The synthesized arm is an ungated pipeline.** Trace2Skill with no
  validation step. [[literature/papers/yang2026skillopt]] reports that
  validation-gated, edit-bounded optimization beats both human-written
  skills and Trace2Skill. So the finding is "ungated trace distillation
  loses to human abstraction", not "agents can't write skills".
- **The held-out split is real but mild.** A date split within the same
  repos (django is 92 of 200) is close to in-distribution. The
  cross-benchmark Pro-100 is where both skill types shrink, and DevSkills
  lose significance on CC and MSA-Q35+.
- **The detectors are the authors' own.** Thresholds were hand-calibrated,
  containment ignores clones (the authors say this undercounts), and cost
  uses an OpenAI tokenizer for Claude and MiniMax models. "Cost attributed
  to a behavior" is an accounting model, not a counterfactual. The Table 3
  cost deltas, which do not depend on the detectors, are the trustworthy
  numbers.
- **Pass@1 only, and 15 of 24 approach cells are single runs.** The
  robustness rule is one-sided per cell with no family-wise correction,
  though the 0.06 expected-reversal figure partly answers that.
- **Coding, not research.** SWE-bench tasks are roughly 20–100 LLM calls
  long. The authors defer long-horizon tasks to future work, which is
  where this project's loops live.

## Trust signals

- **Credibility:** 4. Lin Tan's software-engineering group at Purdue, with
  code and analysis in an anonymized review repo. It has the most careful
  variance protocol in the graph's cost cluster: floors derived rather than
  asserted, the equal-variance assumption tested and one exception handled,
  and behavior metrics explicitly denied verdicts. It reports its own
  negative results (CodeGraph; the amortized SynSkills). Held back by being
  an unreviewed preprint, zero citations, one benchmark family,
  author-designed detectors, and treatment skills written by the same
  people who analyzed the data. Trust the Table 3 and 15 cost deltas and
  the noise protocol more than the causal "because".

## Follow-up

- **Relevance:** 4. The digest scored it 3 as a skill-authoring datum. The
  full paper adds two things the abstract doesn't mention. One is a derived
  noise band, which bears directly on the open
  `iterate-no-improvement-noise-band` question. The other is a
  Claude-Code-specific anatomy of subagent return loss. Not a 5: coding
  tasks, efficiency rather than correctness, and short horizons.
- **[[concepts/budget-as-ceiling]].** It *derives* the tolerance from
  measured spread and states the estimator. RRSI, ingested alongside it,
  shows where a band goes in the loop but leaves its estimator unstated. The recipe fits
  `/iterate`: triplicate the baseline once, reuse its floor, call a single
  run "improved" only when |Δ| ≥ 1.90 s. It also carries a warning: an s
  from 3 runs has about 52% relative standard error.
- **[[concepts/skill-library-lifecycle]].** A held-out test of human
  abstraction vs ungated trace distillation. Read it with the confounds
  above. It weakly corroborates this user's "concise, goal-level skills"
  preference.
- **[[concepts/hierarchical-delegation]].** The return channel is lossy.
  The parent pays again to re-read what the child saw (50.15% of CC's
  re-reads), and a tool change that silently removes cheap-tier delegation
  raises cost.
- **[[concepts/scripted-tool-pipelines]].** Ephemeral scripts are
  regenerated, not reused. Claude Code's "NEVER create files" instruction,
  which this ingest agent is running under, is the named cause.
- **[[concepts/context-eviction-policy]].** Re-reads of content that was
  never evicted: a salience failure, not a paging fault.
- **Actionable for this repo's skills (untested here).** DevSkill 5 is
  "capture test output to a file; filter, don't rerun". DevSkill 7 is
  "break loops: when repeated actions produce no new evidence, take one
  materially different action". Both are cheap additions to `/implement`
  and `/iterate` prompts. **Candidates:** bai2026how is already in the
  graph. Trace2Skill (Ni et al. 2026) and Gong et al. 2026 (skills
  optimized for task success and inference cost) are not in the graph.
