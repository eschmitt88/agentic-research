---
kind: paper
title: "rTaste: Do Research Agents Have Taste? Evaluating Early Stage Research Judgment"
authors: ["Art Kanke"]
institutions: ["University of Minnesota"]
year: 2026
venue: "arXiv 2610.40488 (cs.CL), v1 2026-10-29, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.40488"
code_url: "https://github.com/ArtKanke/rTaste"
citations: null
source: "raw/papers/kanke2026rtaste.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evolutionary-expansion]]"
  - "[[concepts/spend-forecast-calibration]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/pass-at-k]]"
tags: ["research-taste", "experiment-proposal", "minimum-viable-experiment", "pre-registration", "outcome-forecasting", "brier-score", "proposal-ranking", "order-effect", "same-model-judge", "single-vs-team", "llm-auditors", "planted-defects", "novelty-judging", "obvious-follow-ups", "cost-per-useful-experiment", "model-only-labels"]
---

# rTaste: Do Research Agents Have Taste? Evaluating Early Stage Research Judgment

## TL;DR

Six frontier models each read seven recent NLP papers (the "seed papers")
with their code. Each model works once as a single researcher (SR) and once
as a research team (RT). It registers five minimum viable experiments
(MVEs), each with one pre-registered "event" (a statistic against a
threshold). It then gives each MVE a probability and ranks the five. Every
MVE runs, whatever its rank, under fixed limits (one A100, 45 min machine
time, 60 model calls, $6), and the result is rerun on a fresh machine.
Three LLM auditors judge the outcomes. That makes 84 conditions, 83 with
MVEs, 415 MVEs, 412 delivered. Each condition ran **once**. CIs are across
the **seven seed papers**. **No human labels anything.**

Three results matter for this graph:

1. **Proposals are useful; ranking them adds little.** 194 of 415 MVEs are
   judged sound, valuable and new. The agent's first choice is the
   auditors' most valued MVE in 38% of conditions (random: 20%). It beats
   the MVE the agent *wrote first* by only +3 pp (95% CI −8 to +15). The
   first choice *is* the one written first in 47% of conditions.
2. **The proposer's forecasts are no better than a fresh reader's.** A
   fresh instance of the same model, given only the seed paper and the five
   registrations, forecasts as well. Brier difference −0.006 (CI −0.021 to
   0.009). The mean of six such readers beats the agent (0.179 vs 0.192).
   Both beat a base rate (0.240).
3. **A team (two proposers, a critic, a lead) costs more for an unclear
   gain.** +0.71 sound, valuable and new MVEs out of five (CI −0.10 to
   +1.52), at $23.93 vs $15.31 per condition. The gain is mostly the two
   smaller models. Claude Opus 5.5 alone matches its team.

## Claims

- Agents "produce many worthwhile experiments", but evidence that their
  rankings improve the choice of what to run "remains limited".
- Agents are better at examining a paper and taking its obvious next step
  than at moving beyond it. This is a separate capability and should be
  measured separately.
- The agent's own reading and search do not measurably improve its
  outcome forecasts over a same-model reader who sees only the registrations.
- Agents rank largely in the order they wrote, and favour MVEs likely to
  pass.
- Choosing a model is choosing a kind of experiment. GPT-6 Astra and
  GPT-6.1 Sol go beyond the paper more often and register stricter tests.
  Claude Fable 5.1 and Opus 5.5 stay closer and register tests that pass
  more often.
- Because proposing cost varies about 100× across models while execution
  cost varies about 2×, a small-model team with a strong executor matches
  the best large model on sound, valuable MVEs per dollar.

## Methods

- **Seed papers (Table 5).** Seven arXiv cs.CL papers with code, posted
  3 Aug – 24 Sep 2026, from Hugging Face daily papers. No >8B models, no
  large GPU training. The agent gets full text, page images and read
  access to the repo. No summary, gap or hypothesis is given.
- **Models.** GPT-6 Astra, Claude Fable 5.1, Claude Opus 5.5, GPT-6.1 Sol,
  and the two smaller models GPT-6 Luna and Muse Spark 1.3. All run at
  maximum reasoning effort. They ran 6–8 Oct 2026. 30 of 415 registrations
  were written at medium effort (Opus 5.5 team lead, after output-limit
  cutoffs). On one seed paper the Opus 5.5 provider refused the
  single-researcher requests, so 83 of 84 conditions produced MVEs.
- **Harnesses.** SR: one context from reading to forecasts. RT: two
  proposers draft independently with search and code access; a critic
  without tools reviews the pooled drafts once; a lead without tools keeps,
  revises or merges them into five, then forecasts. Both use the same arXiv
  search tool, not providers' web tools.
- **Registration (Table 6).** JSON with hypothesis, closest work (arXiv
  IDs), inputs, method, one event (statistic, relation, threshold),
  evaluation, resources and a "full-scale study" field. 384 events use
  "greater". The prompt says all five will run under the same limits, so
  the ranking states a priority. It does not decide what runs.
- **Judges.** Each of the six models, in a fresh context, sees the seed
  paper and the five registrations in random order. It does not see the
  code, reasoning, forecasts or proposer. It cannot search. It forecasts
  and ranks. The judge from the agent's own model is the "same model
  judge". All judges commit before any MVE runs.
- **Novelty and obviousness.** Claude Opus 5.5 and GPT-6.1 Sol, with the
  same arXiv tool, label each MVE equivalent / close variant / distinct.
  "New" means neither finds close prior work. Separately, each of the six
  models lists ten follow-ups from the paper alone. An MVE is an
  "obvious follow up" if either novelty judge matches it to any list.
- **Execution.** Claude Opus 5.5 or GPT-6.1 Sol, alternating by seed
  paper. The executor sees only the seed paper, code and one registration.
  If one refuses or exceeds limits, the other makes one attempt. 58 of 412
  delivered MVEs were implemented by an instance of their own proposer.
  Results are deleted and the evaluation is rerun on a fresh machine (35
  min). That rerun value decides the event. Seed paper 7 ran 53 of 60 MVEs
  on H200s.
- **Audits.** Claude Opus 5.5, GPT-6.1 Sol and Grok 4.7 judge each
  delivered MVE: faithful to registration, valid evidence, and research
  value 0–3. They see the registration, executor notes, rerun output and
  files, but not the seed paper or the proposer. Majority vote for
  faithful/valid and median for value. **Sound** = faithful and valid;
  **valuable** = value ≥ 2. The auditors also rank each condition's five
  by value. The best mean rank is the **most valued MVE**.
- **Reviews (added after the study).** The same three models read the seed
  paper text, registration and what the auditors saw. They answer 11 fixed
  questions (Table 9): relation to the paper, obviousness ("on how many of
  ten researchers' lists"), what the evidence says, and the next step.
  Majority of two decides.
- **Auditor checks.** On seed paper 1 only, Claude Sonnet 5.5 planted one
  defect into copies of 10 delivered MVEs.
- **Statistics.** Every comparison is within seed paper, averaged over the
  seven, with a 95% CI across the seven.

## Results

- **Quality (Table 1).** Sound, valuable and new per five: SR 2.00, RT
  2.67. Beyond the paper and non-obvious: 57 (SR) and 69 (RT) MVEs; 73 of
  the 194 sound, valuable and new MVEs. The four larger models give about
  three sound, valuable and new per five, the smaller two about one. 369
  of the delivered MVEs are sound (Table 7).
- **Kind (Table 2).** 66% build on the paper, 24% check a claim, 5% open a
  new direction. 44% match an obvious-follow-up list. On one seed paper
  (Flash-dLLM), all eight conditions of the four larger models wrote the
  same correctness check first.
- **Model split.** GPT-6 Astra and GPT-6.1 Sol propose two thirds of the
  beyond-paper, non-obvious MVEs. Their share exceeds Fable 5.1 and Opus
  5.5 by 48 pp (CI 29 to 66) and is higher on all seven seed papers. Their
  events occur in 25% of sound MVEs, against 52% for Fable 5.1 and Opus
  5.5. They mostly compare a CI bound with the threshold (Table 7: 59 and
  63 of 70); the Claude models mostly use a plain threshold.
- **Events are not results.** Of sound MVEs whose event did not occur, 50%
  still gave informative evidence, mostly against the hypothesis. Of those
  whose event occurred, 20% were inconclusive. Small models' MVEs are as
  often sound as Sol's but informative half as often (GPT-6 Luna 38% vs
  Sol 74%).
- **Forecasts (Table 3).** Brier, lower is better: agent 0.192; single
  judges 0.179–0.216; mean of six judges 0.179; base rate 0.240. Agent
  minus same-model judge: −0.006 (CI −0.021 to 0.009). Miscalibration
  direction is model-specific. Astra, Sol, Luna and Muse state higher
  probabilities than their event rates. Fable 5.1 and Opus 5.5 state
  lower ones.
- **First choices.** Most valued: 38% (random 20%). Advantage over the
  MVE written first: +3 pp (CI −8 to +15). Not shown better than the same
  model judge's first choice either. No CI is given for the 38% vs 20%
  contrast. The written-first MVE is on an obvious list in 61% of
  conditions, against 35% for the fifth. The first choice's event occurs
  in 47% of conditions, the last choice's in 29%. Judges also put the
  written-first MVE on top in 39% of conditions. Judges' first choices
  are beyond-paper and non-obvious *less* often than random. Auditors also
  rank event-occurred MVEs higher.
- **Teams.** +0.71 (CI −0.10 to 1.52) sound, valuable and new per five.
  Luna 0.71 → 2.00, Muse 0.14 → 1.00. Opus 5.5 3.33 (SR, six seed papers)
  vs 3.14 (RT). Rated non-obvious +20 MVEs, CI includes zero. Teams
  mainly change design: more CI-bound events, lower probabilities.
- **Cost (Tables 1, 4).** $15.31 SR vs $23.93 RT per condition. Sound,
  valuable and new per $100: SR 13, RT 11. GPT-6.1 Sol SR is best at 30.
  397 of 415 delivered on the first attempt; median implementation 12
  min. Auditors judge 18% of Opus-executor MVEs not sound vs 3% for the
  Sol executor, with evidence informative equally often under both.
- **Auditor reliability (Appendix B).** Planted defects: the majority
  flagged 8 of 10 defective copies and 0 of 10 originals. Pairwise Cohen's
  kappa: faithfulness 0.09–0.32, validity 0.44–0.66, value ≥ 2 0.56–0.70.
  Novelty judges agree at kappa 0.27. Opus labels 387 of 415 distinct,
  Sol 298. The rerun reproduced the executor's value in 335 of 407 and
  the event in 403 of 407.

## Critique / open questions

- **Against the digest framing: at.** Every number in the summary is in
  the paper and quoted correctly: six models, seven papers, 415 MVEs, 194
  and 73, first choice vs random and vs written-first, forecasts vs base
  rate and vs fresh instance, teams. The "Why" line ("agent rankings don't
  measurably improve which one runs") paraphrases the abstract. Three
  things the entry omits change how far it carries:
  1. **The ranking never chose what ran.** Every MVE ran regardless of
     rank, and the paper says so in its limitations: an agent whose
     ranking decides what runs "may rank with more reasoning". The
     `/propose` reading is an extrapolation.
  2. **"Not better than written-first" is a wide null.** +3 pp with a CI
     of −8 to +15, over seven seed papers and one run per condition. It
     does not exclude a 15-point advantage. The two choices also coincide
     in 47% of conditions, so the contrast rests on the other 53%. No
     interval is given for "beats random" (38% vs 20%).
  3. **Every label is model-made.** No human checks soundness, value,
     novelty or obviousness. Two of the three auditors are also the
     executors and are among the six models studied. "New" rests on
     novelty judges with kappa 0.27. Faithfulness agreement is poor
     (kappa as low as 0.09).
- **What "taste" scores here partly measures passing.** Auditors judge
  after seeing outcomes and rank event-occurred MVEs higher. Agents'
  first choices also pass more (47% vs 29% for last). So "first choice
  matches the most valued MVE" is partly agents and auditors both
  preferring MVEs that pass. That is the reverse of the paper's own MVE
  definition, which says an MVE "is useful only if its outcome is not
  obvious in advance". The judges' first choices are non-obvious *less*
  often than random. Pre-execution ranking, as measured, selects for
  predictability.
- **The forecast null may be built in.** Registrations had to be complete
  enough for a separate executor, so they carry most of what the proposer
  knows (the paper notes this). The fair reading is not "agents know
  nothing extra". It is "a full registration transfers the proposer's
  forecasting edge to any reader". That is a good property for a
  pre-registration and bad news for proposer-only forecasting.
- **Teams are not compute-matched.** RT costs 1.56× SR and has no
  equal-cost SR arm, such as SR best-of-two. The critic and lead have no
  tools, so the lead can't check the critic. The gain for small models
  (~1 of 5) is the one robust-looking contrast, and it has no per-model
  CI.
- **Cross-model comparisons are confounded by the event form.** Astra and
  Sol "pass less" mainly because they gate on a CI bound (Table 7). The
  occurred rate is a design choice, not a quality signal. The paper says
  this; the digest does not.
- **Scale.** One domain (NLP papers runnable on one GPU with ≤8B models),
  seven seed papers, one run per condition. The executor (Opus vs Sol)
  shifts the not-sound share sixfold (18% vs 3%). Executors alternate by
  seed paper, so executor and seed paper are confounded.
- **Fit with the graph.** [[literature/papers/ho2026soundnessbench]] found
  LLMs approve 74% of unsound proposals before execution. Here judges are
  not asked to reject, and every MVE runs, so the two do not conflict.
  Both say pre-execution judgment is the weak link.
  [[literature/papers/ning2026implementation]] found one implementation
  per idea was not detectably worse than five for choosing ideas. Here the
  executor itself moves soundness. Together they say the idea-level score
  carries implementation noise that a single run cannot separate.
  [[literature/papers/sun2026scipivot]] found the project record made
  models *worse* at picking the human-corrected plan. That fits the null
  here: the proposer's extra context did not help it forecast.
- **Not tested:** a budget-allocation arm (the paper's proposed next
  study), any redirect after first results, human raters, and repeat runs.

## Trust signals

- **Credibility:** 3.
  - For: a pre-registration design with one stated event per MVE,
    committed before execution; every MVE run, including those ranked
    last; a fresh-machine rerun that decides the event; blind judges and
    auditors who don't see the proposer; a planted-defect check on the
    auditors (8/10 caught, 0/10 false flags); kappa reported for every
    label; within-seed-paper CIs; verbatim prompts (Appendix D); and an
    honest limitations section. The harness and all 415 MVEs with
    forecasts and audits are released.
  - Against: single author, a 5-day-old preprint, no citations, not peer
    reviewed. Only model labels, with two auditors who are also
    executors and studied models. One run per condition. CIs on n = 7.
    The review stage was added after the study.
  - Not 4: no human anchor, one run, single unknown author. Not 2: the
    design is unusually clean and the artifact is released.

## Follow-up

- **Relevance:** 4. It is the first measurement in this graph of a
  proposer's *pre-execution* ranking and outcome forecast against
  executed results, with a fresh-reader control. That maps directly onto
  `/propose` (`expected_metric:` is a registered event) and `--expand`
  (N siblings). It adds material evidence to:
  - [[concepts/evolutionary-expansion]]: proposer rank tracks write order
    and favours passing, so run the population and let execution select.
  - [[concepts/spend-forecast-calibration]]: an outcome self-forecast is no
    better than a fresh reader of the registration, and the bias
    direction differs by model.
  - [[concepts/hierarchical-delegation]]: a propose–critique–merge team
    helped small models and cost 1.56× with no clear gain for large ones.
  Not 5 because labels are model-only, ranking never decided what ran, and
  each cell ran once.
- **For `/propose`.** Two cheap implications, both untested here as
  interventions:
  - If `--expand` siblings must be pruned before execution, don't treat
    the proposer's order or rank as a quality signal. On this evidence it
    mostly reflects write order and pass likelihood.
  - Make `expected_metric:` complete enough that a fresh reader could
    forecast it. Then a cheap second-reader forecast (another model, no
    context) is as good as the proposer's, and the mean of several is
    better.
- **Cheap local test.** On a past downstream proposal batch, compare the
  proposer's rank with order written and with which proposal's
  `expected_metric` target was met. Needs no GPU, only the logs.
- **Candidates** (cited, not yet in the graph):
  - Jaffe & Sherburn 2026, "TasteVal" (arXiv 2610.06824). Concurrent,
    against human experts.
  - Si et al. 2026a, "The ideation-execution gap" (ICLR 2026, arXiv
    2506.20803).
  - Tran & Kiela 2026, "Single-agent LLMs outperform multi-agent systems
    ... under equal thinking token budgets" (arXiv 2604.02460). The
    compute-matched team arm this paper lacks.
  - Wen et al. 2025, "Predicting empirical AI research outcomes with
    language models" (NeurIPS 2025, arXiv 2506.00794).
  - Foster et al. 2026, "AI research preference models" (arXiv
    2608.13940).
