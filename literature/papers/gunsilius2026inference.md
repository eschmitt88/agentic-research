---
kind: paper
title: "Inference After Coding Agents"
authors: ["Florian Gunsilius", "David Van Dijcke", "Kaspar Wüthrich"]
institutions: ["Emory University", "University of Virginia", "University of Michigan", "CESifo"]
year: 2026
venue: "arXiv 2611.23174 (stat.ME), v1 2026-11-17, preprint (35-page main text + 28-page appendix)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.23174"
code_url: "https://github.com/Davidvandijcke/inference_after_coding_agents_replication"    # named in the title-page footnote, not opened at ingest
citations: null
source: "raw/papers/gunsilius2026inference.pdf"
added: "2026-11-24"
relevance: 4
credibility: 4
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/intent-level-briefing]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["selective-inference", "post-selection-inference", "specification-search", "p-hacking", "coding-agents", "agentic-data-analysis", "replay", "subsampling", "selection-multiplier", "self-correction", "multiplicity-correction", "llm-judge-matching", "econometrics", "codex", "claude-code"]
---

# Inference After Coding Agents

## TL;DR

A coding agent that looks at the data before choosing its analysis
reports a p-value that is not valid for the analysis it chose. The
authors correct it by **replaying the agent**. They draw subsamples
without replacement and run the agent's own emitted analysis code on
each. Subsamples where that code's p-value is at least as small as the
reported one form the *tail*; the rest form the *body*. A fresh agent
with the same prompt runs on each subsample, and a commitment prompt
plus an LLM comparison decides whether it chose the same model and
null. The ratio of tail to overall reselection rate, the **selection
multiplier λ**, rescales the reported p-value. Under stated
assumptions the result is asymptotically valid conditional on
selection, with finite-replay confidence bands.

Five results matter for this graph:

1. **Under a "one compelling finding" prompt on null data, agents
   lead with p ≤ 0.05 most of the time.** On 100 simulated RCTs with
   every effect zero, the lead finding is significant for
   **97/99 GPT-6-Luna, 88/100 GPT-6.1-Sol, 74/100 Claude Sonnet 5.5
   and 52/97 Claude Opus 5.5** reports (Table 9). Every null-data
   condition uses a prompt that asks the agent to explore outcomes,
   follow-ups and subgroups and pick a lead. No condition fixes the
   analysis in advance or drops the request for a lead.
2. **The replay test brings false rejections near 5%, tested on one
   agent.** It rejects **4.2%** with a known selector, **5.1%** on
   simulated data and **5.9%** on three real studies with placebo
   treatments. The agentic rows are all GPT-6-Luna. On simulated data,
   65 of 99 corrected p-values sit at 1, because the agent almost never
   re-picks its analysis when it is not significant (0.31% of body
   replays). The near-5% is a test that mostly declines to reject. It
   is not a fine calibration.
3. **The correction costs power and hundreds to thousands of agent
   runs per p-value.** With a true effect the replay test rejects
   **49.5%** and **87.9%**, against **78.8%** and **99.0%** for the
   pre-specified test. The real-study design used 24,542 agent
   sessions. One Card–Krueger correction used 3,000 replays. Selecting
   an analysis took up to 157,000× as long as executing it. No dollar
   or token cost is reported.
4. **Agents' own multiplicity corrections are not a substitute.** The
   family an agent corrects over depends on the data. GPT-6.1-Sol
   counted 26 tests on the full Card–Krueger data (Holm p 0.027 →
   0.70). On subsamples it counted **6–194**, and on full-data placebos
   **12–1,252**. The headline p-value the agent returned was never the
   corrected one. Report openings gave the corrected p for only
   **7–25%** of significant leads.
5. **An agent's account of its own selection is not evidence about
   it.** In the immigration application GPT-6-Luna logged that its
   search was "a breadth check, not a result-driven decision". It
   re-picked the analysis **2.42×** as often when the analysis was
   significant. Claude Sonnet 5.5 declined to lead with "the most
   statistically compelling" result in 97.1% of Card–Krueger replays
   and showed no selection there (λ = 1.00). On the null simulation it
   still led with p ≤ 0.05 in 74%.

## Claims

- Coding agents create a new form of post-selection inference. They
  search an open, implicit space of analyses like a human, and they can
  be reset and replayed on counterfactual data like a program. The
  correction exploits the replayability.
- λ is all that is needed both to detect selection (a lower bound > 1)
  and to correct for it. The selective p-value is p · λ(p), using
  Sood (2024).
- The corrected p-value is asymptotically valid conditional on
  selection (Theorem 1). Under two-sided stability it is exact. It has
  oracle power against contiguous alternatives and inherits consistency
  (Prop. 3). Finite-replay Clopper–Pearson/Berger–Boos bands are valid
  (Lemma 1, Theorem 2), and validity is uniform over a null class under
  stronger assumptions (Corollary 1).
- The correction is learned from the agent's actual choices, not a
  model of its search. So it "remains valid even as AI technology
  continues to evolve rapidly". The paper backs this with its own
  replication: persona steering (Miao et al. 2026) works on Sonnet 4.6
  and disappears on Sonnet 5.5 and GPT-6-Luna.
- Subsampling without replacement is required, not the bootstrap. An
  n-out-of-n bootstrap sample sits at total-variation distance → 1 from
  a fresh dataset, and agents notice duplicated rows. Selection's trace
  on subsamples is bounded by √((m/2n) log(1/ρ)) (Prop. 1).
- Self-applied multiplicity corrections control a different error rate
  over a data-dependent family. They overcharge analyses with λ ≈ 1 and
  can undercharge rarely-reported ones with λ > K.

## Methods

- **Setup.** The prompt r is fixed and independent of the data. The
  agent A_r(D, Ξ) maps data and randomness to an analysis q = (model,
  null). Each q has a fixed p-value procedure that the agent's code
  implements. Human revision of the prompt is out of scope.
- **Assumptions.**
  - (1) Subsampling: i.i.d. sampling units. Clusters or panels are
    allowed as units; time-series and spatial dependence are excluded.
  - (2) "Trust your agent": correct implementation, faithful commitment
    report, correct match decision.
  - (3) Asymptotically exact fixed-analysis p-value.
  - (4) Selection stability between subsample size m and full size n.
  - (5) Recurrence: the chosen analysis's selection probability stays
    bounded away from zero.
  - Uniform validity needs uniform versions of these. The authors call
    uniform stability "a substantive restriction" that subsampling can
    fail along sequences of nulls.
- **Estimator.** Run the emitted code on many cheap subsamples and sort
  them into tail (p ≤ p\*) and body. Replay the agent on B₁ tail and B₀
  body subsamples, with B₁:B₀ set by a Neyman allocation from a
  discarded pilot (3–5× fewer replays in their experiments). Then
  Ĥ(u) = u·â₁ / (u·â₁ + (1−u)·â₀). If a stratum has no replay or no
  match, the correction is "unavailable" and p̂ = 1. The replay test
  rejects when the one-sided 95% upper band is below 0.05.
- **Matching.** After its run, every agent receives a commitment prompt
  to state its model, null, population, outcome and sample restrictions
  as JSON. Each replay then compares its own commitment with the
  original's. For Card–Krueger a separate GPT-6.1-Sol reviewer judges
  matches. No human audit of match decisions is reported.
- **Harness.** GPT-6-Luna in Codex CLI at high effort unless stated.
  Each replay runs in a fresh copy of the workspace with web search off
  and the codebook's numbers edited to match the subsample. Agents had
  noticed codebook/data mismatches in pilots. Refused replays were
  discarded unread and rerun.
- **Designs.**
  - *Fixed selector:* six N(0,1) outcomes; pick the smallest p with
    probability 0.8. The exact correction is known. 400 datasets,
    n = 1,000, m = 31, B ∈ {25, 100, 400, 1,000} per stratum.
  - *Open-ended simulated RCT:* 2,000 participants, four sites, nine
    outcomes at 6 and 12 months, every treatment effect exactly zero.
    100 datasets, m/n = 0.1, 40+200 and 240+400 replays.
  - *Real data:* three top-5 economics RCTs (workplace wellness,
    maternal labour supply, consumer survey) with permuted or invented
    treatment. 34 datasets each, 40+200 replays, studies de-identified.
    No study name appeared in 24,542 logs.
  - *Power:* the open-ended design with +6.2 or +9.6 pp on 12-month
    employment (80% / 99% power for the pre-specified test).
  - *Applications:* ISSP immigration/welfare (154,921 respondents, 46
    countries; 500+500 replays on 16-country subsamples), with a
    pro-immigration think-tank prompt. Card–Krueger (1994) DiD on 410
    restaurants (3,000 replays on 150-store subsamples). There is a
    known-rule fixed menu of 108 regressions (App. C.3.1).
  - *Self-correction audit (Table 9):* four agents, one run per null
    dataset. GPT-6.1-Sol blind-reads the reports.
- **Prompts (App. D.1).** The open-ended prompt: "The team needs one
  compelling, defensible finding to motivate its next study. Explore
  the data as you see fit, comparing outcomes, follow-up points and
  participant groups, and decide which finding you would lead with …
  explain your choice, and discuss the exploratory nature of the
  finding and the alternatives you considered." Card–Krueger: "select
  the most statistically compelling finding and lead the abstract with
  it … Retain a complete record of the analyses attempted and report
  limitations honestly." The codebook fixes the estimator (OLS with
  site indicators, HC2, two-sided t-test). It leaves outcome, follow-up,
  subgroup and covariates open.

## Results

- **Null simulations (Table 1, Fig. 3).** Rejection at 5%, agent's own
  p vs replay test:

  | Design | Agent's p | Replay test | λ̂ median [band] |
  |---|---|---|---|
  | Fixed selector, 100 replays/stratum | 24.0% | 4.2% | 3.5 [2.5, 4.6] |
  | Open-ended, 40+200 | 98.0% | 2.0% | 17 [6.9, 54] |
  | Open-ended, 240+400 | 98.0% | 5.1% | 19 [7.2, 44] |
  | Real data, pooled (102) | 97.1% | 5.9% | 15 [5.6, 41] |

  - Fixed selector: same decision as the oracle in 96.2% (25 replays)
    to 99.8% (1,000). RMSE 0.153 → 0.019. Band coverage of the oracle
    falls from 97.2% to **92.0%** as replays grow, which the authors
    attribute to asymptotic error.
  - Open-ended: tail reselection 10.1%, body 0.29–0.31%. Of 99
    analyses, 46 had no body match (p̂ = 1), 19 no match at all, 29
    not rejected and 5 rejected. Mean band width is 0.71 (0.44
    excluding tail-only cases).
  - Real data per study: 8.8% (3/34), 0.0% (0/34), 8.8% (3/34). Agent's
    own p rejects 100%, 97.1% and 94.1%.
  - The agents' emitted programs, re-run on 4,000 fresh null datasets,
    reject 4.1–5.9%. The code is a valid fixed-analysis test; the
    choice is what is selected.
- **Power (Table 1 D).** The replay test rejects 49.5% / 87.9%. The
  pre-specified test rejects 78.8% / 99.0%, and the agent's chosen
  program on new data 67.9% / 98.0%. Conditional on the agent picking
  an outcome with a true effect, the replay test rejects 56.3% / 88.8%.
  At the small effect, the agent chose a null outcome on 12 of 99
  datasets.
- **Known rules on a fixed menu (Table 5).** "Fixed model": λ = 1.00,
  corrected = exact = 0.026. "Frontier" (most negative t): λ 1.38,
  corrected 0.035 vs exact 0.038. "Client pressure": λ 14.31, corrected
  0.366 vs exact 0.350. Every replay agent followed its assigned rule.
- **Immigration (Fig. 4).** The original lead was 4.6 points lower
  support per 10 pp immigrant stock, p = 0.0123. λ̂ = 2.42 (band
  excludes 1), corrected p = 0.0299; the result still rejects. Matches
  were 79/500 tail and 32/500 body. With pro- and anti-immigration
  prompts, 13/20 and 11/20 full-data runs lead with the same analysis.
  The persona does not move the choice. The Miao m-value is 0.23, which
  rates the analysis unremarkable.
- **Persona replication (Table 2).** Sonnet 4.6 reproduces Miao et al.
  (59% significant in the persona's direction; Δz −4.5). Sonnet 5.5
  and GPT-6-Luna show 0% (Δz +0.4, +0.2). The authors also found a
  faulty merge in Miao et al.'s distributed data file and rebuilt it.
- **Card–Krueger (Fig. 5, Table 8).**

  | Agent | p | Tail / body reselect | λ̂ (lower) | corrected p (upper) |
  |---|---|---|---|---|
  | GPT-6-Luna | 0.025 | 12.6% / 7.2% | 1.72 (1.43) | 0.043 (0.052) |
  | GPT-6.1-Terra | 0.028 | 23.4% / 12.7% | 1.79 (1.57) | 0.051 (0.058) |
  | GPT-6.1-Sol | 0.027 | 18.1% / 8.9% | 1.98 (1.68) | 0.053 (0.062) |
  | Claude Sonnet 5.5 | 0.041 | 90.2% / 90.4% | 1.00 (0.98) | 0.041 (0.042) |
  | Sonnet, masked data | 0.040 | 77.7% / 77.2% | 1.01 (0.97) | 0.041 (0.042) |

  - Sonnet names Card and Krueger in 18.7% of sessions (10.6% masked).
    It declines the "most compelling" request in 97.1% (97.9% masked).
    GPT-6-Luna declines in 3 of 3,000 sessions.
  - Agents pick an employment measure or a fixed-effects option 2× to
    12.6× as often when it is the most significant (Fig. 6). The
    original's measure still acts as a default (39% when it is not the
    most significant).
  - Reported p-values bunch just below 0.05 (Fig. 8). 97.9% of replay
    agents ran an analysis before writing any script, and 23.5% edited
    the final analysis after starting the report.
  - Methodological soundness: 98.5% by a GPT-6.1-Sol reviewer, 99.5%
    after a Claude Opus 5.5 audit (Table 7). The selection happens among
    sound analyses.
- **Self-correction (Table 9, Figs. 9–10).** On the null simulation:

  | Agent | Leads p ≤ 0.05 | Opening: no doubt / warning / disclaimer | Tests stated (median) | Correction applied | Corrected p in opening |
  |---|---|---|---|---|---|
  | GPT-6-Luna | 97 | 85 / 14 / 1 | 90 | 58% | 7% |
  | GPT-6.1-Sol | 88 | 70 / 11 / 18 | 312 | 100% | 25% |
  | Claude Sonnet 5.5 | 74 | 31 / 4 / 65 | 342 | 100% | 7% |
  | Claude Opus 5.5 | 52 | 35 / 12 / 54 | 378 | 96% | 10% |

  - On 49 power-design datasets where Sol reported the same analysis,
    its own corrections kept 30 effects. The conditional correction kept
    39 (9 gains, 0 losses, exact p = 0.004). Three uncorrected runs kept
    findings in the same p range. On the same evidence, the verdict
    depended on whether the run happened to correct.
  - The Fig. 10 "conditional correction" ranks the nominal p among 51
    fresh null runs that reported the same analysis. That is a
    fresh-data oracle, not the subsample replay.

## Critique / open questions

- **Against the digest entry: at, with two overreaches and an
  unanswerable hint.**
  - ID 2611.23174, three authors, stat.ME and the 2026-11-17 date match
    the PDF. The 54–98% range is Table 9's lead-finding rate
    (52/97 Opus … 97/99 Luna).
  - **"Four frontier agents report significant p-values 54–98% of the
    time"** is the abstract's wording. It means the p-value of the
    returned lead finding. Claude's written reports say otherwise most
    of the time. The opening says the finding fails a correction or is
    consistent with chance in **65% (Sonnet)** and **54% (Opus)** of
    significant leads, against 1% for Luna and 18% for Sol. A human
    reading Claude's report first would be warned in most cases. A
    pipeline reading the JSON p-value would not.
  - **"In simulations it brings false rejections back to about 5%"**
    holds, but only GPT-6-Luna was corrected in the agentic designs.
    The other three agents have raw rates only. Two-thirds of the
    open-ended corrections are p̂ = 1 or unavailable. At the smaller
    budget (40+200) the rate is 2.0%. The 5% shows the test does not
    over-reject. It does not show the corrected p-values are informative
    under the null.
  - **"Asymptotically valid conditional on selection"** needs stability,
    recurrence and "trust your agent". The match step is an LLM judging
    an LLM's self-description, with no measured error rate.
- **The check-at-ingest hint ("how much the 'compelling finding'
  prompt drives the rate") cannot be answered from this paper.**
  - Every null-data condition asks the agent to "explore … comparing
    outcomes, follow-up points and participant groups" and pick one
    lead. There is no neutral-prompt arm and no pre-specified-prompt
    arm on null data. So 54–98% is a rate under *instructed* search,
    not a base rate of agentic analysis. The paper's "relatively mild
    selection pressure" understates a prompt that names the search
    axes.
  - Indirect evidence cuts both ways:
    - The persona sentence did not move which analysis was chosen
      (13/20 vs 11/20).
    - Sonnet 5.5 refused "most statistically compelling" in 97% of
      Card–Krueger replays and showed no selection. On the null
      simulation it still led with p ≤ 0.05 in 74%.
    - "Fixed model" gives λ = 1 by construction (Table 5).
    - Stated test counts (median 90–378) show the search was
      extensive. A min over 18 outcome × follow-up tests alone would
      give ~60% under independence (our arithmetic).
  - The share driven by the request for a lead, as against looking at
    the data at all, is open. A cheap follow-up arm: the same null
    datasets with "report the treatment effect on <named outcome>" and
    with "report what you find" (no lead requested).
- **Cost is the practical limit, and it is unreported in money.**
  Hundreds to thousands of full agent sessions buy one corrected
  p-value. Neyman allocation cuts this 3–5×. That is affordable for one
  headline claim in a paper, not for routine use inside a loop.
- **Recurrence fails exactly where selection is strongest.** With open
  prompts the agent rarely re-picks the same analysis (tail 10%, body
  0.3%). The correction then returns p = 1 or nothing. The authors
  concede that "find me something interesting" may make any particular
  q hard to reproduce. The method is most informative for moderately
  constrained prompts, such as Card–Krueger with 7–23% reselection.
- **Matching is unvalidated.** The JSON commitment and the comparison
  prompt are careful. Yet no human-coded sample checks match decisions,
  and Card–Krueger needed a stronger reviewer model. A false match in
  the tail or body moves λ directly.
- **Scope is classical hypothesis tests on i.i.d. units.** The method
  needs a fixed analysis with an asymptotically exact p-value and
  resampleable units. ML-research claims ("method A beats B on this
  benchmark") would need a test framing and a resampling unit (tasks,
  seeds). The paper does not try this.
- **Economics data and econometric analyses only.** GPT models dominate.
  Claude 5.5 models appear only in the self-correction audit, the
  persona replication and the Card–Krueger robustness table, and Claude
  behaves differently in each (refusals, disclaimers, no persona
  effect). The authors stress that agent behaviour findings are
  short-lived, and that applies to their own 54–98%.
- **One original run per dataset per agent** in Table 9, so each rate
  has roughly ±10 pp sampling error at n ≈ 100 (our arithmetic). The
  real-data per-study rates rest on 34 datasets each.

## Trust signals

- **Credibility:** 4.
  - For:
    - Three econometricians at Emory, Virginia and Michigan (Wüthrich
      is an established name in causal inference). A public replication
      package.
    - Formal results with proofs (Props. 1–3, Theorems 1–2, Corollary
      1). A known-truth design that checks the estimator against the
      oracle.
    - A fixed-menu design where corrected and exact p-values agree.
    - A careful harness: fresh workspaces, codebook numbers edited per
      subsample, de-identified studies, logs searched for memorisation,
      a masked-data rerun for Claude.
    - The authors replicated a competing paper and found and fixed a
      data error in it. Limits (recurrence, uniformity, cost, trust
      assumptions) are stated in the text.
  - Against:
    - Not peer reviewed, one week old, no citations.
    - The corrected test is run on one agent model in the agentic
      designs. Match decisions have no human validation.
    - No prompt ablation behind the headline rate, and no cost in money.
  - Not 5: unreviewed, one agent for the correction, and unvalidated
    matching. Not 3: theory plus a known-truth check plus released code
    goes well beyond most preprints in the graph.

## Follow-up

- **Relevance:** 4.
  - It is the first evidence in the graph that an agent choosing its
    own analysis on data with no holdout produces a selected statistic.
    That is HCE's failure mode in the setting
    `~/claude-system/claude/rules/evaluation.md` exempts ("analysis
    one-shots can ignore this rule").
  - It also gives the first black-box, replay-based correction that
    needs neither a holdout nor a model of the agent's search.
  - It is material new evidence for [[concepts/hce-evaluation]]. It
    adds a selection-provenance type to [[concepts/typed-claim-partition]],
    and a scope limit to [[concepts/intent-level-briefing]].
  - Not 5: the domain is econometric hypothesis testing, not ML-research
    agents. The correction is not applied to benchmark claims, and its
    cost keeps it out of routine loops. If a second source arrives (Miao
    et al. or Asher et al. below), a selective-inference concept could
    be seeded and this would be its canonical evidence.
- **Relation to [[literature/papers/zhou2026does]].** Both are
  "selection replays", but they answer different questions:
  - BiPA *simulates* null candidates through a *known* acceptance rule
    under a fitted task model. It asks whether nothing changing could
    report the gain. It is cheap.
  - This paper *re-runs the black-box agent* on resampled *real* data.
    It asks how much more often the agent picks this analysis when the
    analysis looks good, and it corrects the p-value. It is expensive.
  - Neither substitutes for the other.
- **Candidates** (cited, not yet in the graph):
  - Miao, Pritchard and Zou 2026, "The agentic garden of forking paths"
    (arXiv 2607.01507). The agentic bootstrap and m-value; the closest
    competitor.
  - Asher et al. 2026, "Do Claude Code and Codex p-hack? Sycophancy and
    statistical analysis in LLMs" (Stanford working paper).
  - Sood 2024, "Selective inference is easier with p-values" (arXiv
    2411.13764). The p-value correction this builds on.
  - Bertran, Fogliato and Wu 2026, "Many AI analysts, one dataset"
    (PNAS 123(29)). Gao and Xiao 2026, "Nonstandard errors in AI agents"
    (arXiv 2603.16744).
  - Queen and Zou 2026, "The shelf life of LLM findings" (web). Relevant
    to how long any agent-behaviour number in this graph stays true.
  - Herdağdelen, Joren and Tamkin 2026, "Who's aligned to whom? How AI
    agents shape experts' research judgment calls" (Anthropic report).
