---
kind: paper
title: "Quantifying Claim Smoothing by LLM Research Agents"
authors: ["Tiasa Singha Roy", "Manya Wadhwa", "Greg Durrett"]
institutions: ["New York University"]
year: 2026
venue: "arXiv 2610.35793 (cs.CL), v1 2026-10-26, preprint ('Preprint. Under Review.')"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.35793"
code_url: "https://github.com/tiasa2/SmoothScore"
citations: null
source: "raw/papers/singharoy2026quantifying.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/hce-evaluation]]"
tags: ["deep-research", "claim-smoothing", "overclaiming", "calibration", "smoothscore", "llm-as-judge", "judge-validation", "skeptic-agent", "aspect-clustering", "repair-ladder", "document-engagement", "search-snippets", "full-text-vs-snippet", "calibration-instructions", "researcherbench", "no-human-baseline", "digest-overclaim"]
---

# Quantifying Claim Smoothing by LLM Research Agents

## TL;DR

The paper defines *smoothing*: a report's conclusion is partly supported by
its cited documents, but stated more broadly or more certainly than the
cited set as a whole allows. SmoothScore measures it. It extracts the
report's main conclusions. A support agent and a skeptic agent then pull
evidence for and against each claim from the full text of every cited
document. The evidence is grouped into aspects, and a GPT-5 judge scores
each claim 1–5 by the smallest repair that would make it faithful (none,
append, revise, recast, replace).

They score 868 reports and 5,674 claims from 7 systems on 4 datasets. One
report is generated per query. Three results matter for this graph:

1. **Every system has a real share of materially smoothed claims, but the
   spread is wide.** Claims scoring ≥ 3 range from 22.7% (Luna 5.6 on
   DRB-II) to 93% (Gemini Deep Research). The GPT systems score mostly 1–2.
   Severe smoothing (≥ 4) is 1–4% for Luna and Sol, and 22–35% for Gemini.
2. **Agents cite pages they never opened.** On ResearcherBench, Luna opened
   19% of the documents it cited (95/496), and Sonnet 5 opened 2%
   (34/1,458). Luna's median search snippet is 585 characters.
3. **Full text plus calibration instructions halves smoothing; either
   alone does much less.** On 61 ResearcherBench questions, the share of
   claims ≥ 3 fell from 29% to 12% (Luna) and from 61% to 25% (Sonnet 5).

There is no human-written baseline. So "agents consistently overclaim"
means "a GPT-5 judge finds a share of claims that needed a fix". It does
not mean "more than a human expert would".

## Claims

- LLM research reports "read as confident syntheses" but "fail to
  accurately reflect the underlying documents".
- Citation-level verification is not enough. A claim can be supported by
  every document it cites and still be smoothed, because other cited
  evidence narrows or contradicts it.
- Smoothing is pervasive: "On average, for each system at least 25% of
  generated claims received a SmoothScore ≥ 3" (intro). The results
  section says "at least 22% of claims affected in every case".
- Limited engagement with cited documents contributes to smoothing, and
  "actively engaging with the full documents can help reduce smoothing".
- Three recurring patterns: *unsupported composition* (separately
  supported pieces combined into a stronger claim), *scope expansion*
  (one setup's result stated generally), and *premature closure*
  (conclusive wording: "fundamentally resolves", "definitively proves").
- Comparative and topic-overview questions invite more smoothing.

## Methods

- **Definition.** A claim is smoothed if (1) it is a conclusion drawn from
  aggregated evidence, (2) some evidence supports at least part of it, and
  (3) the conclusion is stronger or broader than the evidence warrants.
  The authors frame it as a special case of hallucination detection that
  needs graded, not binary, judgments.
- **Pipeline (Table 8).** GPT-5-mini for every stage except the judge,
  which is GPT-5; all calls use `reasoning_effort=medium`, default
  sampling.
  1. Claim extraction: the report's main conclusions (typically 4–8),
     lifted verbatim with their hedges and scope.
  2. Evidence extraction, per (claim, cited document): a support agent and
     a skeptic agent. The skeptic tags each item `counter`,
     `concession-match` (confirms a limitation the claim itself states, and
     does not count against it) or `rival-win` (another approach's success
     that only counts if the claim ranks or excludes it). Each item has a
     verbatim quote, an aspect, a note and a 1–5 strength.
  3. Aspect induction: per-document aspect proposals, merged into a 4–8
     node hierarchy per claim.
  4. Evidence assignment: each item goes to an aspect by topic, not by
     stance.
  5. Judge: the minimum-repair test. None → 1, append → 2, revise → 3,
     recast → 4, replace → 5. Its ground rules include "item counts and
     pass labels are not evidence of consensus" and "the claim's own
     logical force sets what can defeat it".
- **Evidence pool.** Only the documents the report cites, reconstructed as
  full text. Smoothing is therefore relative to the cited set, not to the
  literature.
- **Datasets.** All 65 ResearcherBench questions; all 25 ScholarQA-CS2
  expert-written questions; 18 of 66 English DeepResearch Bench II tasks;
  17 of 101 ResearchRubrics tasks. The last two are filtered to questions
  the authors labelled high-synthesis and open (Table 9).
- **Systems (Table 10).** GPT Luna 5.6 and GPT Sol 5.6 with OpenAI web
  search; Claude Sonnet 5 and Claude Opus 5.5 with Anthropic web search and
  web fetch; Gemini Deep Research (deep-research-preview-04-2026); DR Tulu
  (8B, local, vLLM + MCP); QUEST-35B-RL (local). The main-run prompt and
  any turn or time limits are not stated.
- **Human validation (§5.1, App. C).** Two authors independently scored 73
  claims with the judge's rubric, blind to system. The 73 claims come from
  three systems only: Luna, DR Tulu and Gemini. The dataset is not stated.
  Gemini 2.5 Pro and Claude Sonnet 4.5 were tried as alternative judges.
- **Evidence-usefulness check (§5.2).** Luna and Sonnet 5 ResearcherBench
  reports are revised with no web search under three contexts: none, all
  cited documents, or the SmoothScore evidence. GPT-5 scores the revisions
  against ResearcherBench's gold rubrics.
- **Engagement (App. F).** A cited link counts as opened if the agent
  fetched or opened that page. For Claude systems, search results arrive
  encrypted, so only web-fetch calls are counted. What the model saw from
  search alone cannot be observed. Gemini exposes no browsing steps.
- **Intervention (§6).** Luna and Sonnet 5 on 61 ResearcherBench
  questions: {web search, cited documents in context with search off} ×
  {original prompt, plus calibration instructions}. The instructions
  (E.2/E.3) say: state the conditions a result was shown under, say when
  sources disagree, do not combine separate findings into a stronger
  conclusion, avoid conclusive language unless warranted.
- **Statistics.** 95% CIs on per-cell means (method not stated). Spearman
  and Pearson correlations for generation features. One generation per
  query and one judge call per claim. No seeds, and no repeat-run variance
  is reported.

## Results

- **Main table (Table 1), mean SmoothScore by dataset:**

  | System | ResearcherBench | CS2-Expert | DRB-II | ResearchRubrics | share ≥ 3 |
  |---|---|---|---|---|---|
  | Luna 5.6 | 1.95 | 1.93 | 1.66 | 1.83 | 22.7–31.6% |
  | Sol 5.6 | 1.93 | 1.82 | 1.83 | 1.93 | 24.8–27.4% |
  | Sonnet 5 | 2.52 | 2.30 | 2.46 | 2.55 | 46.1–59.5% |
  | Opus 5.5 | 2.36 | 2.20 | 2.39 | 2.42 | 44.7–54.0% |
  | QUEST | 2.50 | 2.67 | 2.88 | 2.88 | 55.2–73.0% |
  | DR Tulu | 2.57 | 2.70 | 2.49 | 2.63 | 58.8–68.5% |
  | Gemini DR | 3.27 | 3.18 | 3.05 | 3.23 | 85.0–93.4% |

  Score 5 ("fundamentally misleading") is rare: at most 1.1% for the
  Claude systems, up to 7.4% for QUEST on DRB-II.
- **Features (Table 12, n = 28 system–dataset cells).** Across systems,
  mean SmoothScore rises with documents cited per report (ρ = 0.37,
  p = .056) and claim length (ρ = 0.43, p = .02). Within systems both are
  negligible (ρ = 0.03 and 0.02). Report length has a small, consistent
  within-system association (ρ = 0.12, p = .001).
- **Human agreement (Tables 3, 13).** On 73 claims: annotator 1 vs 2 has
  58% exact, 96% within one point, ρ = 0.72, ordinal α = 0.71. GPT-5 vs
  annotator 1 has 56%, 93%, ρ = 0.69, α = 0.68. Of claims the judge scores
  ≥ 3, 62% are also ≥ 3 for annotator 1. The two annotators agree at
  κ = 0.62 on the ≥ 3 threshold. Gemini 2.5 Pro (α = 0.41) and Sonnet 4.5
  (α = 0.11) agree much less well, and score systematically higher.
- **Judge dependence (Table 14), mean on the 73 claims:**

  | System | Annotator 1 | GPT-5 | Gemini 2.5 Pro | Sonnet 4.5 |
  |---|---|---|---|---|
  | Luna | 1.96 | 1.71 | 2.33 | 2.88 |
  | DR Tulu | 2.16 | 2.60 | 2.96 | 3.32 |
  | Gemini DR | 2.71 | 3.04 | 3.83 | 4.42 |

  All four raters give the same order. Absolute levels move by up to 1.7
  points with the judge.
- **Evidence usefulness (Table 4).** Revision with no context: +0.4 (Luna,
  p = .22), +1.1 (Sonnet, p = .38). With all cited documents: +4.1
  (p = .004) and +9.7 (p < .001). With the SmoothScore evidence: +4.4 and
  +12.0 (both p < .001), using 11k and 65k tokens of context against 117k
  and 324k.
- **Engagement (Table 17), cited links opened:** Luna 8.9–19.2%, Sol
  16.1–30.0%, Sonnet 5 1.1–3.5%, Opus 5.5 0.2–2.3%, DR Tulu 0–0.5%. QUEST
  29.4–73.8%, but its visit tool returns a GPT-5-mini extract filtered to
  the agent's goal, not the page. 45 of 65 Luna reports and 44 of 65
  Sonnet reports on ResearcherBench open no page at all.
- **Intervention (Table 6), mean [95% CI] and share ≥ 3:**

  | | Luna | Luna ≥ 3 | Sonnet 5 | Sonnet ≥ 3 |
  |---|---|---|---|---|
  | Web search | 1.95 [1.84, 2.07] | 29.4% | 2.54 [2.44, 2.63] | 60.9% |
  | + instructions | 1.88 [1.78, 1.98] | 25.7% | 2.23 [2.12, 2.34] | 44.2% |
  | Cited docs | 1.80 [1.70, 1.91] | 23.4% | 2.38 [2.27, 2.49] | 50.7% |
  | Docs + instructions | 1.52 [1.44, 1.60] | 12.1% | 1.78 [1.67, 1.88] | 25.1% |

  Score ≥ 4 falls to 0.5% (Luna) and 2.2% (Sonnet). Docs reports are 1.2–1.6×
  longer with slightly fewer claims, so the drop is not from emptier
  reports. Qualitatively the takeaway often stays the same and the
  conditions are added (Tables 7, 16).

## Critique / open questions

- **Against the digest framing: below framing (mild).** The digest's
  description of the method and its "7 systems and 4 datasets" are correct.
  The overstatement is in what the result means, plus what it leaves out.
  1. **"Agents consistently overclaim" has no baseline.** No human-written
     report is scored, so there is no answer to "more than a careful human
     would". DRB-II is built from expert reports (per its cited title) and
     CS2 has expert-written questions, so a human arm looks available but
     was not run. The related work cites Peters & Chin-Yee (2025): LLM
     science summaries are about five times more likely than human ones to
     overgeneralize. That is a single-document setting, not this one.
  2. **"Consistently" hides a fourfold spread.** For Luna and Sol, 68–77%
     of claims score 1–2 and 1–4% score ≥ 4. A threshold of ≥ 3 means
     "an important qualification is missing", as judged by GPT-5.
  3. **Absolute rates belong to the judge.** The order of systems is the
     same under all four raters. Levels are not. Sonnet 4.5 as judge
     rates Gemini DR 4.42 against the annotator's 2.71. "At least 22–25%
     of claims materially smoothed" is a GPT-5-calibrated number.
  4. **Omitted:** the document-engagement result and the docs-plus-
     instructions intervention, which are the parts a builder can act on.
  5. The digest's "Why" quotes a stale tally (82/94); the current one is
     98/116.
- **Is the GPT-5 judge kinder to the GPT systems?** On the 73 validated
  claims, GPT-5 scores Luna 0.25 below the annotator, and DR Tulu (+0.44)
  and Gemini (+0.33) above. The Luna–Gemini gap is 1.33 under GPT-5 and
  0.75 under the annotator. The paper does not discuss this. About 24
  claims per system cannot separate vendor leniency from noise. The
  Gemini 2.5 Pro judge rates Gemini DR as worst, which argues against a
  simple self-preference story. Still, the systems ranked cleanest are
  the judge's own vendor. The Claude systems' claims were never
  human-validated.
- **Internal inconsistencies.** Appendix C's text gives ordinal α "0.62
  vs. 0.64" for GPT-5 and the second annotator, and α 0.36 / 0.04 for the
  other judges. Table 13 gives 0.68 / 0.71 and 0.41 / 0.11. The intro says
  "at least 25%"; the results section and Table 1 say 22.7% at the
  minimum. Table 10 labels claude-opus-5-5 as "Claude Opus 5". The E.2
  calibration prompt is printed truncated. None of these change a
  conclusion, but they show the appendix was not proof-read against the
  tables.
- **The measure is relative to what was cited.** The skeptic can only find
  counter-evidence inside the cited documents. Citing fewer or friendlier
  sources therefore makes a claim *look* less smoothed. The across-system
  link between documents cited and smoothing (ρ = 0.37, p = .056) has two
  readings: more to compress, or more places for the skeptic to look. The
  within-system null argues against a pure artifact, but the paper does
  not test it. This is the complement of
  [[literature/papers/xiao2026search]]'s gap (a skewed cited set). SmoothScore
  catches overstatement *within* the cited set, and is blind to whether
  that set was representative.
- **The intervention teaches to the test.** The calibration instructions
  name the judge's failure modes almost word for word ("do not combine
  separate findings into a stronger conclusion" = unsupported
  composition). A drop under the same judge is expected even if readers
  would not see a difference. No human check of the regenerated reports is
  reported. It is also one generation per question per arm, with no
  seeds. Luna's instruction-only and docs-only CIs overlap the baseline.
  The combined arm's CIs do not.
- **"Sonnet engages even less" is partly a measurement limit.** Anthropic
  search returns encrypted page content, so Claude systems may see more
  than web-fetch counts show. The paper says so in App. F. The 2% figure
  is a lower bound on exposure, not a measured one.
- **Human validation is thin for a headline instrument.** 73 claims, two
  author-annotators, three systems, unknown dataset. The judge's precision
  at the ≥ 3 threshold against annotator 1 is 62%. So about four in ten
  claims the judge counts as materially smoothed are not, by that
  annotator's reading.
- **What holds up.** The system ranking (stable across four raters), the
  engagement counts (measured from traces), and the evidence-usefulness
  check, which uses an external rubric the pipeline never saw. The
  SmoothScore evidence lifts rubric scores as much as full documents do,
  using one-fifth to one-tenth of the tokens.
- **The box's own instance.** This box's digest entries are written from
  abstracts and search pages, and 98 of 116 PDFs read below their entry.
  That is the snippet arm of Table 6. The `/curate` reader brief (read the
  whole PDF, verify each number, say what the evidence does not show) is
  the docs-plus-instructions arm. The paper's result predicts that the
  instruction alone, added to `/digest`, would help less than reading the
  PDF with the instruction. That is our inference, from two models on one
  benchmark.

## Trust signals

- **Credibility:** 3 (a high 3).
  - For: Durrett's NYU group, a strong NLP lab with prior claim-verification
    work (MiniCheck, Molecular Facts). Code and data released. All
    pipeline prompts printed in full. A blind human check with
    inter-annotator agreement, and a three-judge comparison. An external
    check of the evidence using gold rubrics the pipeline never saw.
    Engagement measured from traces, with the encrypted-search limit
    stated.
  - Against: preprint under review, 8 days old, no citations. Headline
    levels rest on one judge validated on 73 claims by two authors, from
    three of the seven systems. The judge shares a vendor with the two
    best-scoring systems. Appendix α values contradict Table 13. One
    generation per query and no repeat-run variance. No human baseline.
  - Not 4 because the validation is too small for the levels to be
    trusted and the appendix contradicts its own tables. Not 2 because
    the ranking is robust across raters, the artifacts are released, and
    the design is clean and fully specified.

## Follow-up

- **Relevance:** 4. It is the first measurement in this graph of
  over-assertion *relative to the whole cited set*. It adds material
  evidence to:
  - [[concepts/citation-anchoring]]: claims whose anchors all support them
    still overstate, and agents mostly cite pages they never opened.
  - [[concepts/typed-claim-partition]]: "grounded" does not mean
    "calibrated", and the repair ladder is a typed recovery action.
  - [[concepts/web-grounded-literature]]: snippet-level synthesis
    overclaims, and full text plus instructions is the measured fix.
  - [[concepts/hce-evaluation]]: judge choice moves levels, not order (as in
    [[literature/papers/alfaqeeh2026beyond]]), and there is no known-good arm.
  Not 5 because the levels are judge-calibrated, there is no human
  baseline, and the intervention is one run on two models.
- **Relation to [[literature/papers/alfaqeeh2026beyond]].** That paper
  finds the dominant numeric defect is a real value from the wrong place.
  This one finds the dominant prose defect is a real finding stated more
  broadly than the cited set allows. Both are failures a presence check
  passes. Both papers also show judge choice moving levels but not order.
- **Cheap local test.** Run SmoothScore (released code) on this box's
  own digest entries, with the fetched PDF as the only cited document. It
  gives a judge-scored smoothing rate to set beside the hand-made
  98/116 tally. It also tests whether the paper's 1–5 scale tracks
  "below (mild / moderate)" verdicts. Score a sample of the literature
  notes' TL;DRs the same way as a known-good arm. This needs API calls,
  not GPU.
- **Candidates** (cited, not yet in the graph):
  - Onweller et al. 2026, "Cited but not verified: Parsing and evaluating
    source attribution in LLM deep research agents" (arXiv 2605.06635).
  - Narayanan Venkit et al. 2026, "DeepTRACE: Auditing deep research AI
    systems for tracking reliability across citations and evidence"
    (ICLR 2026).
  - Liu et al. 2026, "ConflictScore: Identifying and measuring how language
    models handle conflicting evidence" (arXiv 2606.26437).
  - Rasheed et al. 2026, "From fluent to verifiable: Claim-level
    auditability for deep research agents" (arXiv 2602.13855).
  - Zhu et al. 2026, "Is deep research reliable? Misleading knowledge
    induces false conclusions" (arXiv 2607.20891).
  - Peters & Chin-Yee 2025, "Generalization bias in large language model
    summarization of scientific research" (Royal Society Open Science).
    The only human-vs-LLM overgeneralization baseline cited.
