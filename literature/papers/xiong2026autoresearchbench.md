---
kind: paper
title: "AutoResearchBench: Benchmarking AI Agents on Complex Scientific Literature Discovery"
authors: ["Lei Xiong", "Kun Luo", "Ziyi Xia", "Wenbo Zhang", "Jin-Ge Yao", "Zheng Liu", "Jingying Shao", "Jianlyu Chen", "Hongjin Qian", "Xi Yang", "Qian Yu", "Hao Li", "Chen Yue", "Xiaan Du", "Yuyang Wang", "Yesheng Liu", "Haiyu Xu", "Zhicheng Dou"]
institutions: ["Renmin University of China"]
year: 2026
venue: "arXiv 2604.25256"
peer_reviewed: false
url: "https://arxiv.org/abs/2604.25256"
code_url: "https://github.com/CherYou/AutoResearchBench"
citations:
source: "raw/papers/xiong2026autoresearchbench.pdf"
added: "2026-05-11"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/selective-memory-retrieval]]"
tags: [literature-discovery, benchmark, deep-research, wide-research, react, arxiv, multi-hop-reasoning, failure-modes, evaluation]
---

# AutoResearchBench: Benchmarking AI Agents on Complex Scientific Literature Discovery

## TL;DR

A 1,000-query benchmark (600 Deep Research + 400 Wide Research)
evaluating autonomous literature discovery on a 3M-paper arXiv
corpus. Frontier models max out at ~9% on both tracks (Claude-Opus-4.6:
9.39% Deep accuracy; Gemini-3.1-Pro-Preview: 9.31% Wide IoU) — a
~70 pp gap below their performance on general agentic browsing
(BrowseComp). Specialized academic search (DeepXiv full-text)
consistently beats general web search; longer trajectories and
"think mode" do *not* reliably help; test-time scaling helps Deep
(trajectory brittleness) much more than Wide (recall bottleneck).
The failure-mode taxonomy is the most actionable contribution:
specific named patterns (retrieval drift, evidence aggregation
failure, constraint literalism, precision-unconstrained expansion)
that map cleanly to design decisions in any literature-discovery
skill.

## Claims

- **Scientific literature discovery is a distinct capability frontier
  from general agentic web browsing.** SOTA on BrowseComp >80%; SOTA
  here <10%. The gap isn't "scientific search is harder web search";
  it's that scientific search requires *full-text verification of
  conjunctive constraints* that titles, abstracts, and snippet
  matching can't reach.
- **Frontier models converge on similar failure rates** across very
  different architectures (Claude-Opus-4.6, Gemini-3.1-Pro-Preview,
  GPT-5.4, Qwen3.5-397B, etc. all cluster in 6–9% on Deep). This is
  evidence that the bottleneck is methodological, not capacity.
- **More compute does not reliably help.** GPT-5.4 hits 7.44% in 6.1
  turns; Kimi-K2.5 burns 27 turns to reach 4.69%. Long trajectories
  often degrade into redundant queries and circular reasoning, not
  better evidence accumulation. "Think mode" is consistently
  *detrimental* on Wide Research.
- **Test-time scaling helps asymmetrically.** Pass@16 on Deep
  Research reaches ~24-28%; best@16 on Wide Research only ~10-13%.
  Implication: Deep failures are *trajectory-level brittleness*
  (right path exists, single run misses it); Wide failures are
  *recall bottlenecks* (repeated runs reproduce the same omissions).
- **Specialized academic search beats open web.** DeepXiv (full-text
  indexed arXiv) outperforms Jina open-web search by ~1.5pp Deep
  accuracy and ~3pp Wide IoU across four matched models. The
  difference is that decisive evidence lives in appendices,
  ablations, figure captions, and citation contexts — invisible to
  abstract/snippet-level web retrieval.

## Methods

- **Corpus:** 3M+ arXiv papers via DeepXiv-SDK (full-text indexed,
  agentic search tool). Eight CS domains: CV, ML, NLP, Multimodal,
  AI4Sci, Theory, Robotics, Safety.
- **Deep Research construction** (600 queries):
  (1) Sample under-exposed papers (10-100 citations, not surveys);
  (2) Extract verifiable clues from full-text — secondary methodology,
  proof details, appendix observations, author/affiliation
  relations (explicitly avoiding headline cues);
  (3) Constraint fuzzification + iterative minimal-sufficiency
  pruning (each remaining constraint must materially contribute to
  uniqueness);
  (4) 4-stage adversarial verification — multi-query shortcut
  screen, multi-agent stress test (Claude-Sonnet-4.6,
  Gemini-3-Flash), timed human search (10-min budget per item),
  corpus-level uniqueness audit. 10% are unsatisfiable
  (no-answer) cases.
- **Wide Research construction** (400 queries):
  (1) Domain candidate sourcing via external search;
  (2) Structural attribute extraction → conjunctive query
  formulation;
  (3) Query refinement + human annotation of initial answer set;
  (4) Iterative expansion + 3-LLM unanimous-consensus verification
  + human audit. 96% of "false positives" confirmed as true
  negatives in manual audit.
- **Evaluation:** ReAct loop with DeepXiv search tool. Single tool,
  fixed schema. Deep metric: exact-match accuracy. Wide metric: IoU
  (penalizes both over-recall and under-recall harder than F1).
  All models share the same agent framework + prompt + tool.

## Results

- **Top Deep Research scores:** Claude-Opus-4.6 9.39%,
  Gemini-3.1-Pro-Preview 7.93%, GPT-5.4 7.44%, Qwen3.5-397B 6.97%,
  Claude-Sonnet-4.6 6.96%, Seed-2.0-Pro 6.80%. All else ≤5%.
- **Top Wide Research IoU:** Gemini-3.1-Pro-Preview 9.31%, GPT-5.4
  8.12%, Seed-2.0-Pro 7.87%, DeepSeek-V3.2 7.70%. Claude-Opus-4.6
  is unusual: very high turn count (27.1) but only 6.56% IoU
  because it over-expands and fails to filter (highest recall
  0.341, but lowest precision 0.059).
- **End-to-end systems** (Alphaxiv, GPT DeepResearch, AI-studio
  Gemini-3.1-Pro) tested on 50-query sample: GPT DeepResearch hits
  11/50 on Deep (22%, the best). Suggests well-tuned commercial
  systems modestly outperform off-the-shelf models in the same
  ReAct framework — but still nowhere near saturation.
- **Search tool ablation (Table 3):** Across 4 matched models,
  DeepXiv outperforms Jina open-web by an avg 1.45pp on Deep and
  2.55pp on Wide. The structured-arXiv advantage is consistent and
  largest on Wide (where comprehensive coverage matters).
- **Think-mode ablation (Table 4):** Across 3 models, THINK is
  *worse* than NOTHINK on Wide for all three; on Deep it's mixed
  (DeepSeek-V3.2 +1.5pp, others slightly worse). Reasoning helps
  only when it directly improves *evidence acquisition*, not when
  it just adds deliberation.
- **Failure-mode breakdown** is the most actionable result:
  - **Deep Research failures** (per Fig 11): Retrieval Drift (24-41%),
    Tool Failures (8-26%), Evidence Aggregation (4-19%), Ranking
    (12-40%), Other (6-29%) — varies sharply by model.
  - **Wide Research failures**: GT Boundary Misalignment dominates
    Gemini-3.1-Pro (68%); Precision-Unconstrained Expansion
    dominates Claude-Opus-4.6 (85.3%) — model-specific patterns
    that suggest model selection is a deployment choice, not a
    quality ranking.

## Critique / open questions

- **Synthetically-generated queries are themselves a confound.**
  Deep Research queries are constructed *to be solvable in
  principle* (a unique target paper exists; constraints can in
  theory isolate it). Whether the 9% ceiling reflects intrinsic
  task difficulty or *query-construction bias* (constraints written
  by humans, paraphrased by LLMs) is partly conflated. The 10%
  no-answer cases help calibrate but don't fully isolate this.
- **The metric structure is harsh on purpose, which compresses
  signal.** Deep uses exact-match (no partial credit); Wide uses
  IoU (penalizes both false positives and false negatives more than
  F1). Result: nearly every model scores in 5-9%, so absolute
  numbers are uninformative for ranking. The relative differentials
  (and the failure-mode taxonomy) carry more signal than the
  headline numbers.
- **DeepXiv is a proprietary search tool** (RUC-affiliated, per
  reference [11]). The published comparison vs. Jina open-web
  shows DeepXiv wins, but downstream reproducibility depends on
  either DeepXiv access or implementing an equivalent full-text
  indexed retrieval. Single-author/team paper from RUC; benchmark
  released open-source on GitHub, but the eval infrastructure
  isn't fully portable.
- **The error taxonomies are model-specific in distribution.** Fig 11
  shows that Gemini-3.1-Pro's Wide errors are 68% GT-misalignment
  while Claude-Opus-4.6's are 85% precision-unconstrained-expansion.
  This is interesting but also a warning: the taxonomy was
  constructed *from these models' errors*, so it's not necessarily
  a complete typology. A different model family (e.g., a new
  reasoning-focused model) might exhibit failure modes the taxonomy
  doesn't capture.
- **No comparison to RAG-style retrieval.** All evaluated systems
  use ReAct + search tool. The natural alternative — embed all
  papers, do similarity search, let the LLM filter — isn't
  evaluated. Probably because at 3M papers it's expensive, but the
  comparison would clarify whether the failures are agentic-loop
  failures or retrieval failures.

## Trust signals

- **Credibility:** 4 — Renmin University of China (RUC); arXiv
  preprint, not yet peer-reviewed; dataset, evaluation pipeline, and
  code publicly released at github.com/CherYou/AutoResearchBench.
  Reputable academic group with a full artifact release; the
  DeepXiv search backend it builds on is proprietary, which limits
  full end-to-end reproducibility.

## Follow-up

**Relevance: 4** — provides the strongest external empirical anchor
for [[concepts/web-grounded-literature]] (which this project enacts
via `/discover`, `/fetch-paper`, `/digest`) by establishing realistic
SOTA bounds (9%), comparing specialized vs. general search, and
producing a directly applicable failure-mode taxonomy.

**Direct implications for this project's `/discover` and `/digest`
skills:**

1. **SOTA is 9%. Our digest's 6 candidates → ~3 turn out to be on-mark
   is in the right ballpark.** Calibrate expectations: even
   frontier models miss ~91% of literature-discovery queries. The
   project's strategy of "digest produces ranked candidates, user
   curates" is the right loop given this regime — automation does
   the recall work, human does the precision verification.

2. **Investigate arxiv full-text search over WebSearch for `/digest`.**
   AutoResearchBench shows DeepXiv (full-text indexed arXiv) beats
   Jina open-web by 1.45pp Deep / 2.55pp Wide. Our `/digest` uses
   general WebSearch. A specialized arxiv search backend (arxiv.org
   API + full-text fetch via `/fetch-paper`'s machinery) plausibly
   improves digest quality. Worth a small experiment.

3. **The `precision-unconstrained-expansion` failure mode (Claude-Opus's
   dominant Wide failure: 85.3%) is the one to actively design
   against in our setup.** Our digest currently produces 3-6
   candidates per cycle. Maintain this small-and-filtered shape;
   resist the temptation to expand candidate pools to "be
   comprehensive." Better one well-grounded candidate than five
   loosely-grounded ones.

4. **Think-mode is not consistently helpful.** Our digest skill
   uses Claude's default reasoning, which is roughly "think mode."
   The Wide Research evidence suggests this may *hurt*
   discovery-style tasks where the bottleneck is evidence acquisition,
   not deliberation. Worth testing whether a no-think variant of
   `/digest` produces similar-or-better candidates faster.

5. **Long natural-language queries beat keyword piles.** The paper
   explicitly shows that short-query preferences from web tasks
   *degrade* scientific search. Our digest's queries are reasonably
   long but could be longer and more constraint-rich. Worth
   revising the digest skill's query-synthesis prompt to produce
   3-5-sentence scenario-rich queries rather than
   "topic1 topic2 topic3" strings.

6. **Test-time scaling asymmetry matters.** Deep Research benefits
   much more from pass@k than Wide Research does. For our digest
   (which is closer to Wide — exhaustive recall across a theme
   rather than precise paper identification), running more digest
   passes won't help much; running *better* single passes will.
   This is a design constraint that shapes how we'd parallelize.

**Pairing with the batch.** This is the only paper in the May-11
batch that directly evaluates *agents doing curation work*, not
*architectures for agents that do curation work*. The others —
GSAR, ExpWeaver, SkillOS, SkillRet — give us building blocks for
how to *structure* knowledge and decide *when* to use it.
AutoResearchBench gives us the empirical reality of *how well*
agents currently do at the discovery task in the first place. The
honest takeaway is: discovery is hard, the curator-in-the-loop
pattern (this project's design) is the right shape, and the
relevant frontier is "make individual passes better," not "do
more passes."

**Suggested follow-up** (not in this batch):
- Investigate AutoResearchBench's eval pipeline (publicly released
  on GitHub) as a candidate evaluation target for any
  literature-discovery skills downstream projects build on top of
  this meta project. Could anchor a future
  `literature-discovery-evaluation` concept if downstream demand
  emerges.
- The DeepXiv-SDK paper (ref [11] in this paper:
  `qian2026deepxiv`, arXiv 2603.00084) is the natural next read —
  it specifies the agentic data interface for scientific papers
  that AutoResearchBench treats as a primitive. If we want to
  improve `/digest`'s retrieval backend, that's where to look.
