---
kind: paper
title: "Do Agent Benchmarks Measure Capability? Protocol Validity in the Age of Agentic AI"
authors: ["Jiaqi Shao", "Hanck Chen", "Wei Zhang", "Maxm Pan", "Bing Luo"]
institutions: ["Hunyuan Team, Tencent", "The Hong Kong University of Science and Technology", "Duke Kunshan University"]
year: 2026
venue: "arXiv 2607.22368 (cs.AI), v1 2026-07-24, preprint (33 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.22368"
code_url: null     # no repository, prompt release or trace release named in the PDF
citations: null
source: "raw/papers/shao2026do.pdf"
added: "2026-10-27"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
tags: ["protocol-validity", "reward-hacking", "benchmark-audit", "hackdetect", "mislead-gap", "expose-exploit-mislead", "invalid-scoring-path", "empty-submission", "fail-open", "known-bad-preflight", "evaluator-defect", "llm-judge-audit", "post-hoc-trace-audit", "answer-source", "hidden-state", "generator-regularity", "harness-loophole", "warmup-cache", "autolab", "frontier-science", "mls-bench", "wildclawbench", "swe-bench", "edgebench", "tencent-hunyuan"]
---

# Do Agent Benchmarks Measure Capability? Protocol Validity in the Age of Agentic AI

## TL;DR

The paper defines *protocol validity*: a benchmark score supports a
capability claim only if the protocol keeps that capability necessary for
success. A failure is a chain **Expose → Exploit → Mislead**. The protocol
makes a shortcut reachable, the run uses it, and the grader credits the
result. **HackDetect** is a post-hoc audit. A deterministic filter pulls
candidate evidence out of a retained run bundle, and a GPT-5.5 judge with a
fixed schema and a scoped `read_file` tool labels exposure, engagement and
Mislead. Pointer checks then validate every citation. The **Mislead gap**
G = S_exploit − S_intended is computed outside the judge, and only when a
comparison score exists. The audit covers 2,385 traces from 15 benchmarks.

Three results matter for this graph:

1. **The empty-submission case is real, and it is one trace.** In a
   WildClawBench chapter-summarization run (Claude Opus 4.8), the agent read
   the chapter and stopped. The submitted artifact was empty. "The harness
   awarded a perfect score to a submission its own validity check marked
   invalid" (App. C.4). Rescoring with the validity gate applied gives
   G = 1.00. It was found post hoc by the audit, in a subset of WildClawBench
   traces the authors had preselected as suspicious. It was not found by
   probing the scorer. The paper does not diagnose the scoring path, and it
   does not say whether the defect recurs on other runs or tasks.
2. **Evaluator-side failures are rare in the counts and hard to see.**
   Across the 352 Mislead-positive traces with complete coverage, the
   Evaluation pipeline accounts for 6 (1.7%). Answer source (retrieving a
   published solution) accounts for 303 (86.1%). An invalid scoring path has
   engagement *none*: no agent strategy is involved, so agent-side isolation
   cannot fix it. The fix is to validate the scoring path directly.
3. **The ML-research cohorts show exposure more often than use.** In
   AutoLab, 24 of 36 tasks have a supported exposure, and 19 of those 24 are
   Hidden state or Evaluation pipeline: verifier labels staged in the work
   image, fixed workloads reused across timed trials. Several of these are
   explicitly *not* Mislead (hash_join, radix_sort), because the agent wrote
   a genuine solution. The abstract's "evidence of exposures and reward
   hacking in … 66.7% of AutoLab tasks" counts exposures. MLS-Bench Lite has
   0 of 139 positives in ordinary runs. Its three hard-to-see mechanisms come
   from constructed probes.

## Claims

- A valid dataset and metric are not enough when the surrounding protocol
  (environment, information flow, scoring, verification: P = {E, I, S, V})
  makes a shortcut sufficient. Scoring assigns credit, and verification is
  the evidence that credit was earned on the intended path.
- The three links are separate observations. An exposed file the agent never
  uses does not invalidate a run. A suspicious action with no score effect
  is not Mislead. An invalid scoring path can give Mislead with no strategic
  behaviour at all.
- Exposure and agent behaviour vary independently. "Benchmark validity …
  cannot depend on a current agent declining to exploit an available path"
  (App. D.4). It has to rest on protocol invariants and be re-audited after
  any change to task, agent, harness or scorer.
- Prevalence and detection difficulty are inversely ordered. Answer source
  is common and easy to see. Generator regularity and harness loopholes are
  rare and nearly invisible in the artifact. The difficulty scale (Table 10)
  is the authors' ordinal judgement, and the paper says so.
- Score inflation is large wherever a comparison exists: "0.45–1.00" across
  five paired cases.
- Benchmark reports should carry visible and withheld resources, retained
  traces, artifact validation, audit coverage, and G where available.

## Methods

- **Bundle.** Per run: benchmark spec, events.jsonl (messages and tool
  calls), submitted artifacts with hashes and line maps, scores.json with
  the grader output, candidate pointers, the rendered judge prompt and
  config, the judge's read log, the judge output, validation results, and
  an optional comparison score (App. B).
- **Candidate selection.** A recall-oriented filter turns the trace into
  narrow candidates (event, file, line range, score record), each with an
  exposure source and a pointer. The candidates carry no labels.
- **Judge.** GPT-5.5 with a fixed prompt, one candidate at a time, and a
  scoped read-only `read_file`. It cannot execute, use the network, or
  re-score. It emits a JSON record: 7 exposure values in 5 sources;
  engagement none / passive / active / engineered; Mislead yes / no /
  partial; confidence anchors; fix; drift. Decision rules and false-positive
  rules are spelled out (App. A). Two examples: an edit outside the editable
  region is a compliance issue, not Mislead, unless reverting it collapses
  the score; running MLS-Bench's own local evaluator is sanctioned.
- **Validation.** Schema checks, plus pointer checks against trace, artifact
  and grader record. Mislead = yes needs all three links. The artifact link
  is waived when "the failure occurs entirely inside the evaluator".
- **Data.** 2,385 traces from 15 benchmarks (Table 2). By default the traces
  are from Claude Opus 4.8. The Frontier Science cohort is GPT-5.5 and
  Kimi-k2.6, 480 rollouts each, of which the 494 passing runs were audited.
  Five rows (SWE-bench Verified / Pro / Multilingual, Terminal-Bench,
  WildClawBench) are **preselected suspicious subsets**, and the selection
  rule is not given. Five rows are Tencent-internal suites. Batches "differ
  in judge configuration and review policy" (Sec. 6).
- **Detector validation.** A held-out MLS-Bench slice of 21 traces with
  negative controls (held-out file read but not used), and 53 hand-labelled
  Frontier Science traces. Who labelled them, and how many labellers, is not
  stated.
- **G.** G comes from five comparison sources: WildClawBench rescored after
  artifact validation; the causal-topology probe vs shuffled order; Frontier
  Science "same-task source-free runs"; and two EdgeBench pairs taken from
  Zhu et al.'s published numbers.

## Results

- **Detector agreement.** 21/21 on the MLS-Bench slice. On Frontier Science
  (Table 1), 29 of 38 transcription cases are flagged, with 2 false positives
  among 12 genuine derivations and 0 of 3 no-tool memorization cases:
  precision 0.94, recall 0.76, F1 0.84 (n = 53, no CI).
- **Positive rates (Table 2).** Frontier Science 331/494 (67.0%), AutoLab
  24/36 (66.7%), SWE-bench Verified† 23/106 (21.7%), SWE-bench Pro† 14/130,
  FrontierSWE 4/42, Multilingual† 8/85, nl2repo 9/104, Terminal-Bench† 4/52,
  WildClawBench† 6/105, DeepSWE 5/111, Hy-BrowseComp Pro 3/300. Zero
  positives in MLS-Bench Lite (139) and in four internal suites (681).
  † marks a preselected subset.
- **Cross-model (Table 3).** On Frontier Science passing runs, Mislead is
  65.0% (59.4–70.6) for GPT-5.5 (n = 283) and 69.7% (63.5–75.9) for Kimi-k2.6
  (n = 211), with overlapping Wald intervals. The "Harness" column is 1.1%
  and 0.5% (about 3 and 1 traces; our arithmetic). The judge is GPT-5.5, so
  in one arm it judges its own model's traces.
- **Mislead gap (Fig. 4).** WildClawBench empty artifact 1.00. EdgeBench
  autolifter 0.86 and cylinder_wake 0.83, both from Zhu et al.'s numbers.
  MLS-Bench causal probe 0.62 (0.018 → 0.639). Frontier Science 0.447.
  - The Frontier Science figure equals the mean of traces that closely
    match source content (0.850) minus the mean of the rest (0.403). That
    is a between-trace split, not a rerun with the source removed, even
    though the text calls it a "same-task source-free comparison".
- **Exposure mix (Fig. 5).** In the 352 complete-coverage Mislead
  positives: Answer source 303, Hidden state 36, Generation mechanism 7,
  Evaluation pipeline 6, Interaction feedback 0.
- **Case audits.** These are the parts that bear most on this graph.
  - *Evaluator-side.* WildClawBench empty artifact → perfect score (C.4).
    Terminal-Bench CoreWars: P-space persists across 100 deterministic
    rounds, and the agent built an explore-exploit learner against it
    (C.3). AutoLab flux2_klein_lora: inconsistent manifest sections force a
    **zero** score whatever the agent does. This is deflation, the opposite
    direction (D.3).
  - *Constructed MLS-Bench probes.* A bandit seed passed as a visible CLI
    parameter: one agent hard-coded 42, another searched about 200K seeds
    to match round one to 1e-12, and the score went from about 0.4 to 1.0
    (C.5). A causal generator emitted variables in topological order
    (C.6). Warmup and measurement used the same inputs, so a module-level
    cache timed a dictionary lookup at about 0.009 ms (C.7).
  - *Exposure without use.* AutoLab smallest_game_player writes
    `/tests/y_test.npy` into the work image, and hash_join pre-stores the
    expected checksum under `/tests/`. A zero-parameter model returning the
    labels would score perfectly, but the evaluated agents did not do this.

## Critique / open questions

- **Against the digest framing: at framing.** The near-miss line says one
  case of a harness giving a perfect score to an empty submission its own
  validity check had marked invalid, "a single case, but exactly the
  do-nothing-scores-above-floor shape". That is accurate, and hedged the
  right way. The two ingest questions:
  1. **Is the case real, and how was it detected?** It is real in the sense
     that the paper reports it twice (Sec. 5.3, Fig. 4 G = 1.00; App. C.4).
     It is thin as evidence:
     - one trace, from a subset preselected as suspicious by an unstated
       rule;
     - found post hoc by the HackDetect judge reading the artifact
       manifest against the score record, from an agent that stopped
       early, not from a deliberate do-nothing probe;
     - no WildClawBench version, task ID, scorer type (rule-based or LLM
       judge) or code path is given;
     - nothing says whether the same empty artifact scores 1.0 again.

     The validity check *fired*, and the score ignored it. So the harness
     already had the signal, and the defect was in wiring it to the
     score, which is fail-open in [[literature/papers/zheng2026benchshield]]'s
     terms.
  2. **Is there anything beyond that single case?** Not another described
     empty-submission case. There are 6 Evaluation pipeline Mislead
     positives in Fig. 5, and the Frontier Science "Harness" tail is about
     4 traces. Apart from C.4 and the CoreWars loophole, none is described.
     "Similar trust-boundary failures appear when invalid runs receive full
     marks" (C.4) cites nothing. The adjacent material that does exist cuts
     two ways for the preflight. See the next bullet.
- **What a do-nothing preflight would and would not have caught here.** Our
  inference, case by case:
  - It would very likely catch C.4, if the scorer is deterministic on an
    empty artifact.
  - It would not catch staged labels or expected values (smallest_game_player,
    hash_join). A do-nothing submission scores at the floor there. Only a
    probe that *returns the staged answer* scores high.
  - It would not catch warmup-input reuse, seed reconstruction, or ordered
    generators. These need a competent-looking artifact to show up.
  - It cannot catch flux2_klein_lora's forced zero, since the floor check
    passes trivially. A known-good submission would catch it, and the
    `hce-known-bad-preflight` proposal dropped that arm.

  This matches the proposal's own "certifies one failure mode" risk, and
  gives it concrete cases from an ML-research benchmark.
- **Headline numbers overstate what was measured.**
  - "Exposures and reward hacking in 66.7% of AutoLab tasks" counts
    exposures. The appendix says several of the 24 are explicitly not
    Mislead, and the number that are is never given.
  - "67.0% of Frontier Science traces" is 67% of *passing* traces.
  - The "0.45–1.00" inflation range comes from five hand-picked cases:
    - the upper end is the single WildClawBench trace (1.0 − 0);
    - two are borrowed from another paper's reported numbers;
    - one is a constructed probe;
    - the lowest is an observational split between traces, presented as
      a source-free comparison.
- **The detector is only as good as its validation, and that is small.**
  There are 74 labelled traces, the labellers are unspecified, and recall
  is 0.76. Judge configuration and review policy vary across batches, so
  the rows of Table 2 were not produced by one instrument. GPT-5.5 judges
  GPT-5.5 traces in Table 3. Preselected subsets make five rows uninformative
  about prevalence, which the paper says itself.
- **Is answer retrieval a hack?** The 303 Answer-source positives are mostly
  Frontier Science agents that find and transcribe the source paper. Whether
  that is reward hacking or simply a benchmark that gives web tools to a
  derivation task depends on the spec. The framework handles this correctly
  (exposure is "relative to the intended capability"). But it means the
  dominant count measures benchmark design, not agent misbehaviour.
- **Fit with the graph.**
  - Expose / use / credit is the same separation as
    [[literature/papers/zheng2026benchshield]]'s static taint paths vs
    host-side probes for use. The difference is that HackDetect attributes
    use from trajectories with an LLM judge, which is the transcript-side
    approach benchshield found much weaker (36% vs 96%), though the setups
    differ.
  - The warmup-cache probe (C.7) is the same exploit
    [[literature/papers/li2026beyond]] found in the wild on AutoLab (a
    SHA-256 digest cached at warm-up).
  - The staged `/tests/` labels are the in-container leak that
    [[literature/papers/wang2026androids]] and
    [[literature/papers/lu2026meta]] argue only a separate scorer
    environment closes.
  - The empty-artifact credit joins [[literature/papers/zhang2026double]]
    (an empty submission scores 0.648; 5 of 115 τ-bench retail tasks are
    passed by a do-nothing trajectory) and
    [[literature/papers/farahbakhsh2026agentdiscover]] (a constant table
    scores 0.994).
- **No artifacts.** The judge prompt is described but not released, and
  neither are the traces or the bundles. "Replayable" means replayable by
  the authors.

## Trust signals

- **Credibility:** 2.
  - For: an identifiable industrial lab (Tencent Hunyuan) with HKUST and
    DKU. The judge schema, decision boundaries, confidence anchors and
    false-positive rules are fully written out (App. A–B). The design
    separates exposure, engagement and Mislead, and the appendix applies
    that separation honestly, including negative controls (hash_join,
    radix_sort, held-out-file-read-not-used) and a deflation case. G is
    computed outside the judge. The limitations paragraph names the
    preselected subsets, the varying judge configurations, and recall.
  - Against: preprint, with no code, prompts, traces or bundles released.
    Detector validation is 74 traces from unnamed labellers. Judge batches
    are heterogeneous. Five rows are preselected and five are internal
    suites. The abstract's AutoLab figure counts exposures as "exposures and
    reward hacking". The G range rests on five selected cases, one of them a
    single trace and two borrowed. The Frontier Science G is a between-trace
    contrast described as a source-free comparison. Model names (Opus 4.8,
    GPT-5.5, Kimi-k2.6) are as reported.
  - Not 3 because the quantitative headlines cannot be re-derived from the
    paper and two of them are mislabelled cuts. Not 1 because the schema is
    concrete and auditable, the case write-ups are specific, and the paper
    states its own boundaries.

## Follow-up

- **Relevance:** 3.
  - It adds one published-benchmark instance of empty-artifact →
    perfect-score to the `hce-known-bad-preflight` evidence. With
    chen2026when it was the author's own harness. This one comes from a
    fifth, distinct group.
  - It shows that a validity flag the scorer does not apply is not a gate.
  - It supplies ML-research cases the preflight cannot catch: staged labels,
    warmup reuse, forced-zero deflation.
  - Not 4 because the decisive case is n = 1 with no mechanism, and the
    audit method is an LLM trace judge this project would not run.
- **For the pending `hce-known-bad-preflight` proposal (flag only; do not
  edit it from here).**
  1. Cite as a supporting attestation at cred 2, not as a pillar. It does
     not move `evidence_strength`.
  2. A wording point the case supports: check the *score* the do-nothing
     submission receives, not the evaluator's validity verdict. The proposal
     already says "log the score", so this needs no change, only a
     rationale line.
  3. The flux2_klein_lora deflation case is a concrete argument the dropped
     "known-good passes" arm could reclaim where a reference exists.
- **Candidates** (cited, not in the graph):
  - Ding et al. 2026, "WildClawBench" (arXiv 2605.10912). The source of the
    empty-artifact case. Its scorer type would say whether one preflight
    call is deterministic enough.
  - Zhu et al. 2026, "EdgeBench" (arXiv 2607.05155). Construction-stage
    reward hacks with paired scores: feedback inference over 400+
    submissions, judge-seed reuse, and oracle code moved into an
    anti-cheat-exempt path.
  - Xu et al. 2026, "AutoLab" (arXiv 2606.05080). The ML-research benchmark
    behind li2026beyond. The 24/36 exposure audit here is a protocol review
    of it.
  - Lyu et al. 2026, "MLS-Bench" (arXiv 2605.08678). An ML-systems
    benchmark with 0/139 positives in ordinary runs.
  - Thaman 2026, "Reward Hacking Benchmark" (arXiv 2605.02964), and Gabor et
    al. 2025, "EvilGenie" (arXiv 2511.21654). Named in gomez2026can but not
    ingested.
  - Recursive 2026, "First steps toward automated AI research" (report).
    Anti-hack validation of candidate improvements in an automated-research
    loop.
