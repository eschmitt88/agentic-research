---
kind: paper
title: "Search-Time Contamination in Deep Research Agents: Measuring Performance Inflation in Public Benchmark Evaluation"
authors: ["Yongjie Wang", "Xinyue Zhang", "Kunhong Yao", "Zhiwei Zeng", "Kaisong Song", "Jun Lin", "Zhiqi Shen"]
institutions: ["Alibaba-NTU Global e-Sustainability CorpLab (ANGEL)", "Tongyi Lab, Alibaba Group", "Nanyang Technological University (College of Computing & Data Science)"]
year: 2026
venue: "arXiv preprint (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.05241"
code_url: "https://anonymous.4open.science/r/Search-Time_Contamination-25F2/"
citations: null
source: "raw/papers/wang2026search.pdf"
added: "2026-07-27"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/citation-anchoring]]"
tags: [evaluation, contamination, benchmark-design, search, deep-research-agents, holdout-integrity, measurement]
---

# Search-Time Contamination in Deep Research Agents: Measuring Performance Inflation in Public Benchmark Evaluation

## TL;DR

When a deep-research agent can search the web during evaluation, it
routinely retrieves the benchmark's own artifacts — and sometimes the
ground-truth label — turning a reasoning test into key-value lookup.
The paper defines a three-tier severity taxonomy (metadata → question
context → explicit answer), builds and human-validates a detector for
each tier, and shows across six medical QA benchmarks that only the
*answer-level* tier reliably inflates scores: the coarse URL-matching
proxy used by prior work is not a valid contamination signal. Gemini
Deep Research leaks the answer on 60% of 100 sampled MedQA questions.

## Claims

- **Search-time contamination (STC) is a distinct failure mode from
  training-data contamination.** The leak happens at inference, through
  the agent's own tool calls, so it survives any amount of care about
  what went into pretraining and cannot be fixed by releasing a newer
  benchmark. Recently-released sets (MedXpertQA, HLE-149) still show
  leakage via LLM-paper exposure, data-hosting platforms, and forum
  discussion.
- **Severity is three-tiered, and the tiers are not interchangeable.**
  Benchmark Metadata Leakage (BML: search returns dataset-hosting or
  exam-site URLs), Question-Context Leakage (QCL: retrieved text carries
  the question's source passage but not the label), Explicit Answer
  Leakage (EAL: retrieved content contains query *and* gold label).
- **Central negative result: BML alone does not inflate performance.**
  Cox hazard ratios for BML→correct-prediction are *below 1* on five of
  six benchmarks (MedMCQA 0.62 [0.56, 0.69]; MedQA 0.68 [0.60, 0.78];
  MMLU 0.45 [0.36, 0.55]). QCL is mixed and mostly not significant. Only
  EAL is consistently positive and significant (HR 2.20 on Medbullets5op
  → 8.92 [2.46, 32.35] on HLE-149). Since prior work (Han et al. 2025)
  detects contamination by repository URL matching — a BML-level
  signal — **it measures exposure, not exploitation**, and over-reports.
- **Weak leakage is an early-warning signal for strong leakage.** BML
  escalates into QCL and EAL over subsequent turns (Kaplan–Meier), and
  QCL sharply raises the hazard of later EAL (HR 2.50–6.74, p<0.005 on
  four of five). So BML is worth detecting as a *leading indicator* even
  though it is not itself an inflation mechanism.
- **Narrowing the retrieval corpus relocates the boundary rather than
  establishing it.** Valyu Deep Research, which retrieves from curated
  sources like PubMed, shows 0% leakage on MedQA — but 78% on PubMedQA,
  whose questions are curated from PubMed articles. A "safe" corpus is
  only safe relative to a particular benchmark's provenance.
- **Contamination can alter model rankings**, not just absolute scores,
  making leaderboard comparisons between search-enabled agents unsound.

## Methods

- **Detection, one mechanism per tier** — a deliberately typed split:
  BML by regular-expression URL matching against two site groups
  (data-hosting platforms like HuggingFace/GitHub, and medical exam
  sites like Quizlet) plus known dataset-source patterns; QCL by
  normalized longest-common-substring overlap between question and
  retrieved content; EAL by LLM-as-a-judge (DeepSeek V4 Pro) asked
  whether retrieved content directly evidences the answer.
- **Detectors are human-validated, and the paper reports the weaker
  number.** EAL detection on Medbullets5op: 83.3% recall, 100%
  precision against full human annotation. On MedQA (too large to
  annotate fully) only detected cases were post-checked, so precision
  alone is reported: 94.85%.
- **Two evaluation frames.** Question-level: partition the dataset by
  which STC types fired and compare accuracy across subsets. Turn-level:
  treat the search trajectory as a survival process, with correct
  prediction as the event and STC occurrences as time-dependent
  covariates in a time-varying Cox proportional-hazards model — this is
  what separates "the agent was exposed" from "the agent's answer
  changed after exposure."
- **Six clinical QA benchmarks** (MedQA, MMLU-medical, MedMCQA,
  MedXpertQA, HLE-medical, Medbullets5op), chosen because medical exam
  content is heavily redistributed across question banks and forums.
  Primary agent: Tongyi-DeepResearch (fully observable traces);
  generalization checked on Gemini, Step, and Valyu Deep Research.
- **Ablation that isolates the search channel**: same agent with web
  search disabled (Tongyi-DeepResearch*) vs enabled, both against the
  base model (Qwen3-30B-A3B).

## Results

- Search-on vs search-off vs base model on MedQA: 91.28% / 89.00% /
  83.58%. Fine-tuning genuinely helps (89.00 > 83.58); the *additional*
  search gain is the part partly attributable to STC.
- Headline inflation is "up to 4%" (HLE biological/chemical subsets) —
  modest in aggregate but concentrated: on MedMCQA nearly a quarter of
  questions have web-retrievable answers.
- Within the BML-triggered group, the subset that also hit EAL reaches
  95.65% (MedQA) and 100% (HLE-149, Medbullets5op); the BML+QCL+EAL
  co-occurrence subset hits 100% on nearly every benchmark. BML-only
  subsets sit at or below the no-BML baseline.
- Turn-level prediction shifts after EAL are dramatic — e.g. MedXpertQA
  8% → 48%, HLE-149 20% → 100%, Medbullets5op 0% → 80% before/after the
  detected event.
- Cross-agent leakage rates on the first 100 MedQA questions: Gemini
  Deep Research 60%, Step Deep Research 9%, Valyu 0% (attributed to
  Step's Chinese-oriented search infrastructure and Valyu's curated
  corpus, not to better safeguards).
- **Prescriptions**: (1) isolated knowledge sandboxes (e.g.
  ToolUniverse) so all agents retrieve from the same corpus and differ
  only in reasoning; (2) transparent search trajectories — expose
  queries, retrieved URLs, visited pages, and intermediate evidence so
  auditors can check whether an answer is grounded in reasoning or in a
  retrieved artifact; (3) controlled benchmark access — gated access,
  mandatory registration, data-use agreements prohibiting redistribution.

## Critique / open questions

- **Scope is clinical QA only**, and the authors say so. Medical exam
  content is unusually heavily redistributed, so these rates are likely
  an upper bound for domains with less question-bank culture. Whether
  ML-research tasks (this project's dominant interest) leak at
  comparable rates is untested — though MLE-bench-style tasks have their
  own severe exposure via public Kaggle solutions.
- **BML recall is acknowledged-incomplete**: the regex matches a
  predefined site list, so BML prevalence is underestimated. This
  weakens the BML→QCL/EAL cascade analysis (some escalations start from
  undetected BML) but, if anything, strengthens the negative result —
  the tier with the most measurement noise is the one prior work relied
  on.
- **Only three commercial systems**, for cost reasons, and for those the
  internal trajectories are opaque — leakage was inferred from final
  answers, summaries, and visited-URL lists. The 60% Gemini figure is
  therefore an outside-in estimate.
- The "up to 4%" headline undersells the finding. The interesting number
  is not the mean shift but the conditional one: given EAL fires,
  accuracy goes to ~100% regardless of task difficulty. STC is
  low-prevalence/high-impact, which is exactly the profile that mean
  accuracy hides.
- Detector tiers have asymmetric trust: BML/QCL are deterministic string
  operations, EAL is an LLM judge. The load-bearing conclusion rests on
  the *least* mechanical detector — mitigated by human validation, but
  the judge model (DeepSeek V4 Pro) is not independently ablated.
- No treatment of the adversarial case: everything here is *accidental*
  leakage. A benchmark author who wants to detect deliberate
  contamination-seeking behavior needs a different threat model, and the
  Ethics section flags the dual-use risk of the detection method.

## Trust signals

- **Credibility:** 4 — Tongyi Lab (Alibaba) with NTU; a major industrial
  lab on its own agent stack, which cuts both ways (deep trace access,
  but Tongyi-DeepResearch is also the primary subject). arXiv preprint,
  not peer reviewed. Code and full experiment results released, though
  via an `anonymous.4open.science` link that may not survive
  de-anonymization. Detectors validated against human annotation with
  reported recall/precision, and the headline finding is a *negative*
  result that undercuts a convenient prior method — a good sign.

## Follow-up

- **Relevance:** 5 — supplies the second attestation
  [[concepts/information-firewall]] explicitly asked for
  ("contamination-control work framing the same boundary"), and converts
  NatureBench's *stipulation* that web search be disabled at eval time
  into a measured necessity. Simultaneously opens a threat model
  [[concepts/hce-evaluation]] does not currently carry: the holdout can
  leak through the agent's own tool, so local `test/` discipline is not
  sufficient whenever the task is drawn from a public corpus.
- Direct implication for this repo: `/digest` and `/discover` implement
  [[concepts/web-grounded-literature]] over exactly the retrieval
  channel this paper indicts. Harmless here — literature curation *wants*
  to find the source, and there is no score to inflate — but any
  downstream project that both web-searches and evaluates on a public
  benchmark inherits the problem. Worth a note in the evaluation rule if
  a downstream project ever pairs web search with a public holdout.
- The transparent-search-trajectory prescription is
  [[concepts/citation-anchoring]] applied to evaluation auditing: log
  queries, retrieved URLs, and intermediate evidence so a claim can be
  traced to what produced it. Our ingest pipeline already keeps the raw
  artifact and cites it; the missing half is the *query* that surfaced it.
- Open thread: does the BML-is-not-exploitation result generalize? If it
  does, contamination audits that rely on corpus-overlap or URL matching
  (the common practice) are measuring the wrong thing across the board.
  Watch for a replication outside clinical QA.
