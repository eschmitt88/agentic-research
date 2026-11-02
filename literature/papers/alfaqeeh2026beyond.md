---
kind: paper
title: "Beyond fabrication: Misattributed numbers in papers written by AI research agents"
authors: ["Mosab Alfaqeeh", "Omar Hourani", "Muath Hourani", "Hashem Alfaqeeh"]
institutions: ["Al-Ahliyya Amman University", "Binghamton University (SUNY)"]
year: 2026
venue: "arXiv 2610.38691 (cs.AI), v1 2026-10-28, preprint (Elsevier journal format; Section 2.6 cites 'this journal', Data Science and Management)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.38691"
code_url: "https://github.com/mos3b-faqeeh/agent-integrity-analysis"
citations: null
source: "raw/papers/alfaqeeh2026beyond.pdf"
added: "2026-11-02"
relevance: 5
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["claim-verification", "provenance", "misattribution", "fabrication", "numeric-claims", "value-presence-check", "decoy-calibration", "deterministic-checker", "llm-verifier", "llm-as-judge", "judge-robustness", "second-judge", "into-the-unknown", "re-analysis", "human-labels", "non-verification-is-not-falsehood", "metrics-json", "wrap"]
---

# Beyond fabrication: Misattributed numbers in papers written by AI research agents

## TL;DR

A re-analysis of the public release of *Into the Unknown*
([[literature/papers/kim2026into]]): nine agents, fifteen research tasks,
135 submissions, each graded four times by GPT-6-Astra (540 judgments). The
authors run no agents. They use the released papers, working directories and
node-level judge verdicts.

Three results matter for this graph:

1. **An unsupported number is usually a real number in the wrong place.**
   Two co-author annotators labelled 256 numeric claims from 18 submissions.
   Of the 53 unsupported claims, 47 are *wrong source* and 6 are *not found*
   (81% wrong-source after reweighting). The value is in the agent's files,
   but from a different arm, metric, seed, statistic or condition.
2. **Checking that a value appears in the outputs accepts nearly all of
   these.** Value matching accepts 87–95% of unsupported claims and 88% of
   decoy values that are wrong by construction. A deterministic checker that
   scores whether the evidence key's tokens match the claim's context, and
   tests that score against ±2…±10 last-digit decoys, certifies 78 of 85
   claims correctly (0.92, exact CI 0.84–0.97). On held-out submissions it
   is 27 of 29. It settles only about a quarter of claims (23% coverage,
   recall about one third).
3. **"Not certified" does not mean "false".** Among claims the checker left
   uncertified, 57% (held-out) and 72% (development) were labelled
   traceable. Used as a rejector, it would be wrong more often than right.

A Claude Opus 5.5 verifier shown the same evidence has the same precision as
the checker on held-out claims and much higher recall. Its weakness is the
paper's topic: it calls 36% of non-traceable claims traceable. A Claude Opus
5.5 second judge on 30 submissions keeps the vendor-group order and passes 12
on integrity where the official judge passed 6.

## Claims

- Benchmark integrity verdicts say *that* a paper failed, not *how*.
  Fabrication (the number was never produced) and misattribution (produced,
  but reported for the wrong experiment) need different remedies. Detection
  and sanction fit the first. Record-keeping tooling fits the second:
  structured result files with explicit keys, and reports generated from
  those files rather than retyped.
- Because misattributed values exist in the outputs, the obvious automated
  test (is the value present?) "passes exactly the errors that matter most".
  Verification must establish provenance, not presence.
- The checker should be used asymmetrically. A certification is strong
  evidence of provenance. A non-certification routes the claim to human
  review and is not evidence of misconduct.
- Pass rates depend on the judge, rankings less so. The authors find no
  evidence that the official judge (GPT-6-Astra) favours agents from its own
  vendor.
- The authors "make no claim that agents in general misreport more often
  than they fabricate". The share is for this benchmark.

## Methods

- **Data.** The *Into the Unknown* release at commit 612cdca: for each of 135
  submissions, four judge outputs plus consensus with node-level status and
  justification, the full working directory (LaTeX, code, configs, logs,
  result files) and, for some harnesses, the trajectory. 2,376 counted G1
  (integrity) nodes at consensus.
- **Re-analysis (RQ1).** Failure rates by rubric family, judge
  self-consistency across the four repeats, and an OLS of integrity share on
  log run hours with agent and task fixed effects (HC1 errors).
- **Checker (RQ3).** Claims are every number in the expanded LaTeX with a
  decimal point, percent sign or exponent and at least two significant
  digits. **Integers are excluded.** Context is 160 characters of prose, or
  row label, column header and caption for tables. Evidence is
  machine-written result files only (JSON, JSONL, CSV, TSV, logs, plain
  text). The paper's own source, rendered text, build logs, code and Markdown
  notes are excluded. Bulk entries (large arrays, long CSVs, number-dense log
  lines) are flagged. Three levels:
  1. *Value match.* The entry rounds to the claim at its reported precision.
     This is the baseline.
  2. *Provenance score.* Token overlap between claim context and evidence key
     path, line and file path, normalized to penalize long generic contexts.
  3. *Decoy calibration.* The same score for ten decoys (value ±2, 4, 6, 8,
     10 in the last digit). The p-value is the claim's rank.

  Verdicts are *certified* (S ≥ τ and p ≤ ρ), *unattributed* (value found,
  no source identified) and *untraced* (value absent). The strict point
  (τ = 0.12, ρ = 0.1) was fixed before any label was collected. One later
  change removed rendered paper text and build logs from the evidence index.
  The authors say this left development test-split results unchanged. Two
  relaxed points (balanced, broad) were frozen on the development tuning
  split before the confirmation sample was drawn. A held-out decoy at +7
  gives a label-free false-certification rate.
- **Labels (RQ2).** Four labels: *traceable*, *wrong source*, *not found*
  (absent from the artifacts, explicitly not a finding of fabrication), *not
  a claim* (a setting or extraction error).
  - *Development:* ten submissions spanning the three vendor groups (how
    they were chosen is not stated), up to 22 claims each, stratified so
    about 40% are checker-certified. After 10 practice claims, 184 remain,
    split alternately into tuning and test halves. Single annotator.
  - *Confirmation:* one unused submission per agent, nine claims each.
    astra-6 was dropped before viewing because the checker had not finished,
    leaving 72 claims from 8 submissions. The first annotator labelled all
    72. A second co-author, not involved in development labelling, labelled
    48 blind to the first annotator and the checker. The first annotator was
    present but silent. Disagreements were not adjudicated.
  - Annotators see the claim plus the checker's three best-matching evidence
    entries (never its verdict or score) and may open other files.
  - Estimates are reweighted by inverse sampling probability within
    submission and stratum. CIs are cluster-bootstrap by submission (2,000
    resamples), with Clopper-Pearson intervals on raw counts for precision.
- **Baselines.** A sentence-embedding score (all-MiniLM-L6-v2) in place of
  token overlap, tuned to the strict checker's development precision. An LLM
  verifier: Claude Opus 5.5 via CLI, one call per claim in an empty
  directory, given the labelling guide, the claim context and the same three
  evidence entries. A "traceable" answer counts as a certification. All are
  compared on the 129 labelled real claims not used for tuning.
- **Second judge (RQ4).** Claude Opus 5.5 run through the benchmark's own
  runner, prompt, rubric and grading code. The sample is 30 stratified
  submissions, three or four per agent, with up to two G1 passers per agent,
  limited to working directories ≤5 GB. Two submissions were graded four
  times, two twice and 26 once. Leniency difference per submission is tested
  with Wilcoxon, vendor effects with Kruskal-Wallis, permutation tests and a
  GEE logit.

## Results

- **What the official judge fails (Fig. 2).** Of 540 judgments, 450 ended in
  an integrity failure. 469 of 2,376 G1 nodes failed. By family:
  code–paper alignment 52%, number traceability 38%, held-fixed disclosure
  32%, **test-set integrity 28% (23/81)**, pre-stated method 27%, execution
  claims 26%, baseline honesty 8%, selective reporting 5%, hard-coded values
  1%, external code 1%, replicating the published method 0%. Code–paper
  alignment fails in 0% of astra-6 nodes, 16% for sol-5.6, 28% for opus-5.5
  and 56–84% for the other six agents.
- **Judge self-consistency.** All four repeats agree on 90.0% of counted
  nodes (89.5% for G1). The G1 verdict differs across repeats for 11 of 135
  submissions.
- **Run length.** ρ = −0.37 raw. With agent and task fixed effects the log
  hours coefficient is −0.61 (p = 0.59). This matches kim2026into: slow
  agents are weak agents.
- **Labels (Table 4).**

  | Label | Development | Confirmation | Total |
  |---|---|---|---|
  | Traceable | 112 | 46 | 158 |
  | Wrong source | 35 | 12 | 47 |
  | Not found | 4 | 2 | 6 |
  | Not a claim | 33 | 12 | 45 |
  | Wrong source / unsupported | 90% | 86% | 89% raw, 81% weighted |

  Wrong-source patterns (Table 5): a standard deviation reported as a mean;
  the baseline's value reported for the proposed method; one seed reported
  as the mean over seeds; a value from another dataset or boundary condition;
  a matching number from an unrelated line (a timer print); a configuration
  setting reported as a result. The six not-found claims are mixed:
  literature values, quantities computed while writing and never saved, and
  some with no candidate source. None can be classed as fabricated from the
  artifacts.
- **Label reliability.** Inter-annotator agreement on 48 confirmation claims:
  93.8%, κ = 0.89 (four labels); κ = 0.96 (traceable vs not). Before four
  rule-based revisions to the first annotator's labels (prompted by a
  consistency review "performed with an AI assistant"), κ = 0.75 / 0.83. The
  first annotator's blind relabel of 20 development claims agreed 75% of the
  time (κ = 0.56; 0.69 binary), with every change toward wrong source.
- **Checker (Tables 6–7).**

  | Setting | Dev precision | Dev recall | Held-out precision | Held-out recall | Decoy accept |
  |---|---|---|---|---|---|
  | Value match | 0.77 | 1.00 | 0.69 | 1.00 | 88% |
  | Strict | 0.97 [0.81, 1.00] | 0.39 | 0.95 [0.85, 1.00] | 0.34 [0.24, 0.47] | 1.3% |
  | Balanced | 0.84 | 0.49 | 0.88 | 0.51 | 3.8% |
  | Broad | 0.86 | 0.71 | 0.89 | 0.55 | 8.7% |

  Strict, raw counts: development 51/56 (0.91), held-out 27/29 (0.93, exact
  CI 0.77–0.99), pooled 78/85 (0.92, 0.84–0.97). Coverage is 23% of real
  claims. Value match has 13% specificity on held-out (5% on development).
- **Ablation (Table 8, all 211 real claims).** Ignoring bulk data changes
  little against labels but removes a fifth of decoy matches. The context
  score is what separates misattribution: precision 0.77 → 0.92 at recall
  0.60. The decoy test adds no precision against labels but cuts decoy
  acceptance 18.9% → 1.3%. It guards against coincidence, not
  misattribution. Why traceable claims are missed (37 on development
  tuning): 13 just below threshold, 6 tied with decoys in dense grids, 18
  where another entry (a log line, a generic key) scored best. Recall does
  not differ by agent (sol-5.6 50% vs others 49%).
- **Baselines (Table 9, 129 untuned claims, 94 traceable).**

  | Method | Certified | Traceable | Precision | Recall |
  |---|---|---|---|---|
  | Value match | 127 | 94 | 0.74 | 1.00 |
  | Embedding score | 21 | 20 | 0.95 | 0.21 |
  | Embedding + decoys | 54 | 40 | 0.74 | 0.43 |
  | LLM verifier (Opus 5.5) | 102 | 91 | 0.89 [0.82, 0.94] | 0.97 |
  | Checker and LLM agree | 52 | 51 | 0.98 | 0.54 |
  | Checker (strict) | 57 | 51 | 0.89 [0.78, 0.96] | 0.54 |

  Over all 211 real claims the LLM accepts 19 of 53 non-traceable (36%),
  including 19 of the 47 wrong-source ones. The checker accepts 7 of 53
  (13%). LLM–human κ = 0.55 (0.64 binary). Among claims the checker leaves
  uncertified, the LLM's "traceable" answers are right 80% of the time.
- **Benchmark-wide (Table 10).** The checker completed on 102 submissions
  (99 with claims, 14,577 claims). 15 hit a 20–40 min time limit, 2 failed
  and 16 exceeded the 10 GB storage limit. These are the largest ones.
  Certified share per agent: astra-6 75.6%, sol-5.6 62.8%, the other seven
  7.8–17.3%. It correlates with the official integrity score (ρ = 0.22,
  p = 0.028) and against the judge's number-traceability failure rate
  (ρ = −0.32). **opus-5.5** is the outlier: mean integrity score 92.9, but
  only 13.8% certified, and 99% of its values occur somewhere in its result
  files. The judge has about 2.4 number-traceability nodes per submission.
  The authors' hypothesis, untested, is that the judge checks a few headline
  numbers that opus-5.5 gets right.
- **Second judge (Table 11, Fig. 5).** Self-consistency 98–100% (second) vs
  93–99% (official) on multiply-graded submissions. Cross-judge per-repeat
  agreement is 82.3% of counted nodes (κ = 0.51) and 81.2% of G1 nodes
  (κ = 0.38). Majority G1 verdicts agree on 22 of 30. The second judge
  passes 12 on G1 vs the official 6: it passes 7 the official failed and
  fails 1 it passed. G1 node failure rates are lower by 5.4 points on average
  (Wilcoxon p = 0.014): OpenAI −2.2, Anthropic −8.6, open-weight −3.8.
  The vendor interaction is not significant (Kruskal-Wallis p = 0.26; GEE
  p = 0.075). Both judges put the OpenAI agents cleanest. Per-agent order is
  "similar" in the figure, with no rank statistic.

## Critique / open questions

- **Against the digest framing: at framing, with one item below.** Every
  number in the digest summary is in the PDF and is quoted correctly. The
  checks asked for at ingest:
  1. **Benchmark re-analysed:** *Into the Unknown*, release commit 612cdca,
     already in the graph as [[literature/papers/kim2026into]]. "All 540
     judgments" is 135 submissions × 4 repeats.
  2. **How the 256 / 18 were sampled:** 10 development submissions (choice
     method unstated) plus 8 confirmation submissions (one per agent, astra-6
     dropped). Claims were stratified to over-sample certified claims, then
     reweighted. The 256 include 45 "not a claim" items, so 211 are real
     claims. The headline "4 in 5" rests on 53 unsupported claims, 6 of
     them not found. Raw it is 89%. Weighted it is 81%.
  3. **Who the blinded annotators are:** two co-authors. The first labelled
     all 256 and was the only annotator on development. The second labelled
     48. Both were blind to the checker but saw its top-3 retrieved entries.
     The first annotator's self-consistency is only κ = 0.56. Four of their
     confirmation labels were revised after an AI-assisted rules review. The
     digest's "κ 0.89" is inter-annotator agreement on 48 claims, not
     checker–human agreement.
  4. **Fraction the checker can verify at all:** this is what the digest
     drops. Integers and one-significant-digit values are out of scope. The
     judge's own canonical failures ("400 epochs where the logs show 320",
     "ten seeds where nine were run") are integers, so the checker could not
     test them. Of in-scope real claims it certifies 23%, with recall
     0.34–0.39. It ran on 99 of 135 submissions, and the largest were the
     ones excluded. The abstract says "while verifying only a subset of
     claims"; the digest leaves it out.
  5. **"~3× as many misattributed": below.** It is 19 vs 7 of 53
     non-traceable claims, pooled over all 211 claims. On the 129 untuned
     claims of Table 9 it is 11 vs 6 (about 1.8×). There, precision is
     identical (0.89 vs 0.89) and the LLM's recall is 0.97 against 0.54. The
     LLM accepts more of everything, so the fair summary is lower
     specificity (64% vs 87% on the pool) at equal precision. The LLM also
     sees only the checker's top-3 entries, so it is not an independent
     method.
  6. **Second judge:** correct as stated (12 vs 6 of 30). "Keeps the agent
     ranking" holds at the vendor-group level. Cross-judge G1-node κ is
     0.38, 8 of 30 G1 verdicts differ, and 26 of 30 submissions were graded
     once against the official 4-repeat consensus. kim2026into's consensus
     rule scores a 2–2 tie as a fail, so a one-run grade may look more
     lenient for that reason alone. The paper does not say how d_i handles
     this.
- **The "not cheating" summary skips test-set integrity.** Page 4 lists
  "test-set leakage" among overt-cheating criteria that "almost never fire".
  Fig. 2 colours test-set integrity as overt misconduct and shows it failing
  in 28% of counted nodes (23/81). That is as high as pre-stated method
  (27%) and execution claims (26%). It matches kim2026into's "held-out data
  used in development" for every agent. Section 5.1's prose omits that row.
  The misattribution-over-fabrication result is about *numbers*. It does not
  show that agents rarely break protocol.
- **Wrong source is not shown to be inadvertent.** The paper reads it as
  record-keeping drift, citing InnovationEval's selective reporting. A
  baseline's number reported for the proposed method, or one seed reported as
  the mean, is also what a motivated misreport looks like. The artifacts
  cannot tell intent apart, and the paper says so for "not found" but not for
  "wrong source". Read "misattribution" as a provenance type, not a verdict
  on intent.
- **Labels share the checker's retrieval.** Annotators see the checker's
  three best-matching entries first. They may consult other files, but a
  traceable entry the retrieval missed is more likely labelled not found or
  wrong source. That biases both the wrong-source share and the "traceable
  among uncertified" figures, though in opposite directions for different
  claims.
- **The opus-5.5 gap is the most portable finding and the least tested.**
  A judge that checks about 2.4 headline numbers can rate an agent at 93%
  while 86% of its numbers are not certified. Whether those numbers are
  wrong is unknown: the manual read of 14 uncertified opus-5.5 claims found
  both traceable values in number-dense sentences and values computed while
  writing that appear in no file. This box runs Opus 5.5. The pattern to
  guard against is a report whose headline numbers trace and whose
  supporting numbers do not.
- **Things it does not touch.** The checker certifies provenance, not
  correctness, so a certified number can come from a flawed run. It does not
  check whether the run that wrote the result file ran the described code
  (kim2026into's paper/code mismatch is the largest family). There is no
  intervention: no agent was told to generate reports from result files, so
  the proposed remedy is untested.
- **Unremarked overlaps with kim2026into.** That paper's App. B already ran
  Claude Opus 5.5 as a judge, four times on 52 submissions, for consistency
  only. This paper does not cite that run, which could have enlarged the
  second-judge sample. The second judge and the LLM verifier are both Claude
  Opus 5.5, and the authors used Claude for analysis code, drafting and the
  label-consistency review. One vendor appears in every non-human check.
- **Fit with the graph.** This is the measured version of
  [[literature/papers/elkoussy2026agentltl]]'s κ_ground warning. A
  numeric-literal witness check is value match, and here value match accepts
  87–95% of wrong-source claims. It also explains kim2026into's
  "68/68 ALL TRACEABLE" agent self-checks, if those scripts matched values;
  kim2026into does not show the script. The asymmetric-use rule agrees with
  [[literature/papers/wang2026making]] and
  [[literature/papers/zhu2026claimreceipt]]: keep unverifiable out of the
  error rate. It is also the first deterministic claim gate in the graph
  with a measured false-rejection rate against human labels.

## Trust signals

- **Credibility:** 3.
  - For: the strict operating point was frozen before any label. A
    development/confirmation split with relaxed points frozen before the
    confirmation draw. A label-free decoy false-certification rate.
    Inverse-probability weighting with cluster-bootstrap CIs and exact
    intervals. A component ablation and two practitioner baselines. Honest
    reporting of the one post-hoc checker change, the label revisions and
    the weak self-consistency. Limitations stated plainly (one benchmark,
    internal annotators, recall one third, 99/135). Code, labels and derived
    data released (github.com/mos3b-faqeeh/agent-integrity-analysis, named in
    the PDF; not opened at ingest). Uses only public artifacts, so every
    number is re-checkable.
  - Against: two universities with no prior track record in this graph; not
    peer reviewed; no citations yet (5 days old). Annotators are co-authors.
    Development has a single annotator with κ = 0.56 self-consistency. The
    held-out set is 60 real claims and 29 certifications (exact lower bound
    0.77). The development submission choice is unstated. The second judge
    covers 30 submissions, mostly graded once. The text's "cheating almost
    never fires" omits a 28% overt-misconduct row. Every model-based check
    uses one vendor.
  - Not 4 because labels are internal and small, and the summary prose is
    selective in one place. Not 2 because the frozen-before-labels design,
    decoy control and released data make the main contrast (presence vs
    provenance) hard to doubt.

## Follow-up

- **Relevance:** 5. It is the first claim-level, human-validated
  measurement in the graph of how report numbers fail against the run's own
  artifacts. It audits a benchmark already in the graph and supplies the
  false-positive and per-claim checks that kim2026into's caveats said were
  missing. Material evidence for:
  - [[concepts/citation-anchoring]]: presence is not provenance; a validated
    key-context score plus decoys.
  - [[concepts/typed-claim-partition]]: *wrong source* as its own type
    between grounded and absent, and *not a claim* for settings.
  - [[concepts/evidence-gated-completion]]: a deterministic certifier with a
    measured false-rejection rate, usable only asymmetrically.
  - [[concepts/hce-evaluation]]: judge choice doubles pass rates at a stable
    group order; a few headline-number nodes miss most numbers.

  It also bears on [[concepts/verified-memory-writes]]. The proposed remedy,
  reports generated from result files rather than retyped, is a write-side
  binding, but the paper does not test it.
- **Cheap local test (no GPU).** Port the strict checker to experiment
  READMEs and `/wrap` NOTES entries against `metrics.json` and `results/`
  in downstream projects. Hand-label 50 claims. Expected outcome per the
  paper: high precision, low coverage, so certified claims pass and the rest
  route to review. Keys in `metrics.json` should be descriptive, since
  "values computed only in the prose cannot be traced at all".
- **`/elevate` candidate (flag only).** A rule in experiment READMEs: every
  number is inserted from `metrics.json`, not retyped. That is the paper's
  remedy and is simple. It does not clear the evidence bar yet, because the
  paper has no arm that tests it.
- **Candidates** (cited, not yet in the graph):
  - Owen 2026, "Can AI automate AI R&D yet? Early evidence from
    InnovationEval" (Epoch AI report, 2026-10-07). Selective reporting by
    agents.
  - Yin et al. 2026, "Artifact-centered claim-aware observability for
    autonomous scientific agents" (arXiv 2608.18312). The design spec this
    paper implements. Already cited in waks2026what.
  - Eulig 2026, "Correct answer, wrong mechanism" (arXiv 2606.23175).
  - Song 2026, "When does a second model help? Cross-model review in LLM
    verification" (arXiv 2610.01471).
  - Luo et al. 2026, "Maintaining benchmarks against increasingly capable
    agents: detection and remediation of unearned passes" (arXiv
    2609.34262).
  - Chen et al. 2025, MLR-Bench (NeurIPS D&B). About 80% fabricated or
    invalidated results in inspected cases.
