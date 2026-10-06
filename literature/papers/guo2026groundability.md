---
kind: paper
title: "Groundability, Not Scale Alone: When Weak Reviewers Can Audit Strong Coding Agents"
authors: ["Junyu Guo", "Shangding Gu", "Ming Jin", "Javad Lavaei"]
institutions: ["University of California, Berkeley", "Virginia Tech"]
year: 2026
venue: "arXiv 2610.01023v1 (cs.SE; also cs.AI, cs.CL), 2026-10-01. Two-column format with a reproducibility-checklist appendix; no venue named"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.01023"
code_url: null
citations: null
source: "raw/papers/guo2026groundability.pdf"
added: "2026-10-06"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/refusal-cost-symmetry]]"
tags: ["weak-to-strong-review", "llm-as-judge", "scalable-oversight", "coding-agents", "swe-bench", "omission-defects", "grounded-evidence", "generated-tests", "fail-on-base", "review-cascade", "selective-prediction", "abstention", "over-rejection", "evidence-misreading", "mechanical-gate"]
---

# Groundability, Not Scale Alone: When Weak Reviewers Can Audit Strong Coding Agents

## TL;DR

The paper holds a weak reviewer fixed and varies only the **evidence** it is
shown about a coding-agent patch. Each reviewer returns
accept / reject / uncertain, and is scored on paired defect catch and
over-rejection.

- **Unchecked evidence moves the operating point but does not improve it.**
  This covers the raw trace, the patch, and a "structured" packet of
  organized but unverified notes. Each one trades catch against
  over-rejection.
- **The upper bound is near-tautological.** Showing the reviewer the
  **official hidden test results**, which are the same thing that defines
  the label, breaks the trade-off for five of six reviewers.
- **The deployable stand-in does not work as an auditor.** The authors
  build a frozen cascade: an empty-patch rule, a patch-attributable static
  check, and an LLM-generated test kept only if it fails on the unpatched
  repo. Unresolved cases go to Llama-8B. The cascade reaches 0.89 / 0.86
  coverage, but over-rejection is 0.66 / 0.67.

Our reconstruction, from the paper's own counts, is that on the held-out
set the cascade **correctly accepts about 4 of 32 acceptable patches**. Its
selective risk (0.33) is also **worse than a constant "reject everything"
policy** (32/121 = 0.26), which the paper never plots.

The durable, importable content is three design rules:

- consume decisive evidence mechanically rather than handing it to an LLM;
- treat a generated-test **failure** as evidence and a **pass** as weak;
- pair catch with over-rejection and keep an abstain slot.

## Claims

- **More context is not more reviewability.** "A raw trace still leaves the
  reviewer to infer whether the patch covers the missing behavior. A patch
  alone has the opposite problem."
- **Structured but unchecked evidence couples catch and false alarms.**
  "Organizing the evidence makes the work easier to read, but does not make
  it reliable." "Without a checkable signal for each case, catch and false
  alarms rise together."
- **Groundability is the property that matters.** It is defined as
  "whether an independent check can answer the disputed question."
- **Official execution evidence is an upper bound.** "Five of six reviewers
  improve both rates on 122 held-out traces; two classify every trace
  correctly."
- **Scale is not the lever.** "Across our heterogeneous ladder, larger
  parameter count does not consistently predict better review." The
  authors caveat that this is "an observational model comparison, not an
  estimate of the causal effect of scale."
- **A grounded fail proves more than a grounded pass.** "A patch-caused
  failure can … support rejection, but one passing test cannot establish
  correctness."
- **Small models misread even decisive evidence, so gate it
  mechanically.** "When grounded evidence gives a clear answer, use a
  mechanical gate … Do not make the smallest models read it again."
- **The authors' own hedge on the cascade.** It is "a high-coverage stress
  test, not yet a reliable deployed reviewer."

## Methods

- **Agents audited.** All three run in a minimal JSON-tool harness capped
  at 12 turns and 2048 output tokens per turn:
  - GPT-5.4 (gpt-5.4-2026-03-05);
  - Gemini-2.5-Pro;
  - Claude-Sonnet-5.

  Tasks come from SWE-bench Lite/Verified, plus SWE-bench Multilingual for
  one set.
- **Labels.** The official FAIL_TO_PASS / PASS_TO_PASS suites, run in the
  per-instance containers. Agent-added test hunks are stripped first.
  Labels are accept, or reject with a bucket: omission, regression, or
  no-change. No model is ever the labeler.
- **Data: 411 real traces, which are not pooled.**
  - **GPT-5.4 core (154):** 53 accept, 76 omission, 9 regression, 16
    no-change. It splits into a 32-trace Django design set and a 122-trace
    held-out set.
  - **Gemini transfer (59):** 15 accept, 33 omission, 1 regression, 10
    no-change.
  - **Accept/no-change only, with zero real defects:** GPT-5.4
    Multilingual (78: 38 accept, 40 no-change) and Claude-Sonnet (120: 23
    accept, 97 no-change).
- **Data: 101 synthetic/controlled cases.**
  - 37 reference-patch controls;
  - 34 ablations, each deleting one load-bearing piece of a gold patch (9
    import, 10 surface, 15 partial-logic);
  - a 30-case naturalistic pilot.
- **Reviewers.** Six models:
  - Llama-3.1-8B;
  - Qwen-2.5-Coder-32B;
  - Qwen-2.5-72B;
  - GPT-OSS-120B;
  - Qwen3-235B;
  - GPT-4.1.

  They run at temperature 0.8 with a k=5 majority vote. A tie becomes
  uncertain. Reviewers get no tools, except in one baseline.
- **Evidence conditions.** Each reviewer sees one of:
  - the patch only;
  - the full trace;
  - a structured (non-grounded) packet: hunks, requirement alignment,
    heuristic audit flags, and LLM "what is missing" items;
  - an empty-section control;
  - official execution evidence, in two formats ("removed" vs "kept"
    unchecked risk notes).

  For each reviewer, the format was chosen on the 32-trace design set
  (lowest over-rejection first), then frozen.
- **Metrics.** Catch = P(reject | defect). Over-rejection = P(reject |
  acceptable), with abstentions kept in the denominators. Coverage and
  selective risk follow Geifman and El-Yaniv.
- **Cascade (Algorithm 1).**
  - Stage 0: an empty patch is rejected.
  - Stage 1: a patch-attributable unbound symbol is rejected.
  - Stage 2: Gemini-2.5-Pro writes a test from the issue and patch (up to
    3 repair attempts). The test is kept only if it fails behaviourally on
    the base repo. Then fail on the patch → reject, pass → accept, the
    latter being "the frozen evaluation rule, not a recommended acceptance
    rule".
  - Stage 3: Llama-8B, k=5, may abstain.
- **Discipline.**
  - Design choices were made on 32 traces. The held-out (121) and Gemini
    (59) sets were evaluated once.
  - Bootstrap 95% CIs over traces.
  - Paired permutation tests restricted to commonly decided traces.
  - The leakage audit found zero failures.
  - Verbatim prompts, packets, probes and votes are in the appendix.

## Results

- **Patch-only review is blind to omissions.** Llama-8B accepts 9/9 real
  design-set omissions. GPT-4.1 catches 0.27 at over-rejection ≈ 0.
  Qwen-32B catches 0.17 of omissions from the full trace.
- **Structured evidence trades catch for false alarms.** On the design set,
  GPT-4.1 goes to catch 0.91 / over-rejection 0.86. In Fig. S7c the mean
  net-correct over the 154 core traces is **lower** under structured
  evidence (~0.38) than under patch-only (~0.42).
- **Naming the missing item (construction-known) drives catch to 1.00.**
  On 15 partial-logic ablations this held for every reviewer, e.g.
  Llama-8B 0.07 → 1.00. The empty-section control shows it is the content,
  not the format.
- **Upper bound (Table 2, held-out 122 = 90 defects / 32 accepts).**
  Structured → official, as catch / over-rejection:

  | Reviewer | Structured | Official |
  |---|---|---|
  | Llama-8B | 0.70 / 0.75 | 0.98 / 0.00 |
  | Qwen-32B | 0.69 / 0.66 | 0.96 / 0.09 |
  | Qwen-72B | 0.43 / 0.44 | 0.91 / **0.47** |
  | GPT-OSS-120B | 0.51 / 0.41 | 1.00 / 0.00 |
  | Qwen3-235B | 0.28 / 0.19 | 0.90 / 0.00 |
  | GPT-4.1 | 0.69 / 0.62 | 1.00 / 0.00 |

  On Gemini (44 defects / 15 accepts), all three tested reviewers catch
  44/44. Llama-8B wrongly rejects 1/15.
- **The format choice matters a lot (Fig. S5b, full set).** The
  non-selected official format, which keeps unchecked risk notes next to
  the official result, gives over-rejection of roughly 0.65–0.85 for
  Llama-8B, Qwen-32B and GPT-4.1.
- **Accept/no-change sets (Fig. S6).** Under structured evidence the three
  reviewers correctly accept only 12 / 5 / 7 of the 61 correct patches.
  Under official evidence that rises to 49 / 61 / 61. Patch-only Llama-8B
  accepts 64/137 empty submissions.
- **Misreading is the residual error, not weighing.**
  - Llama-8B: 10/61 over-rejections contradict the cited "passed" line.
  - Qwen-2.5-72B: a post-hoc replication confirms the exception. Of 198
    audited over-rejections, 173 contradict their own citation and 24
    cite selectively.
  - Calibration instructions do not remove it (Qwen-72B 40/53 and 45/53).
- **Votes barely help.** Grounded mean 0.953 at K=1 → 0.978 at K=5 →
  0.985 at K=9. Non-grounded conditions are flat in K.
- **Cascade, run once (Table 3).**

  | Set | Coverage | Risk | Catch | Over-rejection |
  |---|---|---|---|---|
  | held-out (121) | 0.89 [0.84, 0.94] | 0.33 [0.25, 0.43] | 0.76 [0.67, 0.85] | 0.66 [0.49, 0.81] |
  | Gemini (59) | 0.86 | 0.26 | 0.80 | 0.67 |

  Gemini values are point estimates.
- **Cascade flow (Fig. S4, held-out).**
  - Stage 0 rejects 15.
  - Stage 1 rejects 0.
  - Stage 2 accepts 10 and rejects 11.
  - Stage 3 rejects 63, accepts 9 and abstains on 13.
  - 19 of the 21 false rejections come from Stage 3.
  - Generated-test reject precision is 0.82; accept precision is 0.30.
- **Our reconstruction of the counts.** It is consistent with Table 3, the
  Fig. S4 counts and the §S4.1 conditional rates (0.82 / 0.84).
  - Overall: about 68 correct rejects, 21 false rejects, 15 accepted
    defects, **~4 correct accepts** and 13 abstentions.
  - Stage 3 alone, in the residual: Llama-8B rejects **19/27 acceptable vs
    44/58 defective** patches (0.70 vs 0.76) and accepts 1/27 vs 8/58. On
    what reaches it, **it does not discriminate.**
  - A constant reject would score coverage 1.00, risk 0.26 on held-out
    (0.25 on Gemini).
- **Against baselines.**
  - The cascade beats patch-only review (paired permutation p = 0.0002 to
    0.03).
  - It does **not** differ from structured review on commonly decided
    traces (p ≥ 0.12). "The cascade's measured advantage is coverage."
- **Tool-using reviewer (GPT-4.1, one sample, 12 turns, 40 + 20-trace
  subsets).** It decides 0.55 / 0.45 of traces, at risk 0.36 / 0.11.
  The cascade's risk on the same subsets is 0.46 / 0.33. The tool-user
  spends 58–84× the prompt tokens, and test-generation cost is excluded.

## Critique / open questions

- **Running finding: holds (32/32).** The digest entry is accurate number
  by number. The 0.86–0.89 coverage and 0.66 over-rejection figures are
  quoted correctly, and over-rejection is not hidden. But it leans on the
  favourable framing in two places:
  - **"Given execution evidence … five of six reviewers improve on both."**
    The "execution evidence" is the official hidden test result, the same
    signal that defines the label. Rendered as text, it is a line saying
    "Official FAIL_TO_PASS execution passed." The upper bound therefore
    measures whether a model can **read its label off the page**. That
    makes the failures more informative than the successes: Qwen-72B
    misses at 0.47, and Llama-8B contradicts its own citation 10/61 times.
    The win also holds only for the format that **removes** unchecked
    notes. With those notes kept, over-rejection returns to about 0.65–0.85
    for three reviewers.
  - **"A frozen cascade … reaches 0.86–0.89 coverage."** High coverage is
    cheap when the residual reviewer rejects by default. The paper has a
    74% defect base rate on held-out (89/121), and against it the cascade's
    selective risk (0.33) is **worse than reject-all (0.26)**. On Gemini it
    is about equal (0.26 vs 0.25). The paper's claim that "no tested
    deployable baseline has both at least as much coverage and no higher
    risk" is true only because reject-all was not tested. The cascade
    correctly accepts about 4 of 32 acceptable patches on held-out and
    about 3 of 15 on Gemini (our reconstruction). The Stage-3 Llama-8B is
    close to a reject-biased coin on the residual (0.70 vs 0.76 reject
    rates).
- **"Reviewer size is not a consistent predictor" is weak evidence either
  way.** n = 6 models from three vendors (Meta, Alibaba Qwen, OpenAI) and several vintages, one of them
  coder-specialised and one (GPT-4.1) of unknown size. Under official
  evidence the ceiling makes size irrelevant by construction. Under
  structured evidence the reviewers sit at different operating points:
  Qwen3-235B's low catch (0.28) comes with the lowest over-rejection
  (0.19). The authors call it observational, and that is the right
  reading. It is a null claim, not a negative one.
- **The defect population is shaped by a hobbled harness.** Agents get 12
  turns in a minimal JSON-tool harness. 101/154 GPT-5.4 patches fail, and
  Claude-Sonnet-5 submits nothing on 97/120. The worked trace (S3.7) shows
  environment-blocked tests, absolute paths rejected, and tool-protocol
  errors. "Omission" defects from a capped agent may differ from those of a
  strong agent in a real harness. The title's "strong coding agents" is the
  nominal model tier, not observed strength.
- **Denominators are small where it matters.** Over-rejection rests on 32
  acceptable held-out patches, with a CI of [0.49, 0.81], and on 15 Gemini
  patches, with no CI. The design set is all Django.
- **The deployable gap is the paper's real result, and the paper says so.**
  §5.5: "Producing equally reliable checks without official tests remains
  the main bottleneck." The abstract leads with the upper bound. The
  limitations section is candid that the cascade is a stress test.
- **No released code or artifacts.** The paper says the environment is
  "pinned with uv" but gives no URL. The appendix is unusually complete
  in verbatim artifacts, though.

## Trust signals

- **Credibility:** 3. In its favour:
  - a reputable group (UC Berkeley; Virginia Tech);
  - a careful protocol: a design/held-out split frozen before evaluation,
    run-once test sets, bootstrap CIs, paired permutation tests on commonly
    decided traces, a leakage audit, and versioned model IDs;
  - a post-hoc replication of its own anomaly, with verbatim failure
    artifacts.

  Not higher because:
  - it is an unreviewed preprint with no code URL;
  - the critical denominators are small (32 and 15 acceptable patches);
  - the harness is crippled at 12 turns;
  - the cascade's headline omits the reject-all baseline that its own risk
    metric makes obligatory.

## Follow-up

- **Relevance:** 4. It is the reviewer-side complement to
  [[literature/papers/park2026when]]: the judge is held fixed and only the
  evidence is varied. It adds three things to
  [[concepts/evidence-gated-completion]] and
  [[concepts/programmable-evaluator-oracle]]:
  - paired catch and over-rejection with abstention;
  - the pass/fail asymmetry of generated tests (reject precision 0.82 vs
    accept precision 0.30);
  - a measured case that a weak model reading decisive evidence still
    misreads it. That supports routing decisive evidence to a mechanical
    gate.

  It is not a 5 because the domain is SWE-bench patch review, not research
  loops. The deployable result is weak, and the upper bound is
  near-tautological.
- **Bearing on `/iterate` and a cheap reviewer
  ([[concepts/hybrid-model-backends]]).** A cheap reviewer is admissible
  only as a reader of mechanically decisive evidence, and even then the
  evidence is better consumed by code.
  - Where the keep decision has a metric file, the metric decides and no
    reviewer is needed.
  - Where it does not, a cheap LLM reviewer over unchecked summaries is
    about as good as a biased coin. The cascade's Stage 3 sits at 0.70 vs
    0.76 reject rates.
  - Do not plan a "small-model auditor" for agent-written narrative.
- **A design rule worth carrying into any gate on this box: never put
  unchecked risk notes next to decisive evidence.** The "kept" format undid
  the grounded gain for three reviewers. In a `/wrap` or `/curate`
  verification step, this argues for showing the check result alone.
- **An instrument to reuse.** Report a gate as (coverage, selective risk,
  catch, over-rejection), and always plot the constant-policy baselines
  (accept-all, reject-all) at the observed base rate. Without them, a
  high-coverage, reject-leaning gate looks better than it is.
- **Candidates:**
  - Chen et al. 2026, "Rethinking the Value of Agent-Generated Tests"
    (arXiv 2602.07900);
  - Cheng et al. 2026, dynamic co-generation of bug reproduction tests
    (arXiv 2601.19066);
  - Kale et al. 2025, "Reliable Weak-to-Strong Monitoring of LLM Agents"
    (arXiv 2508.19461);
  - Engels et al. 2025, "Scaling Laws for Scalable Oversight" (NeurIPS,
    arXiv 2504.18530). This is the controlled version of the
    scale-vs-review question this paper only touches observationally.
