---
kind: paper
title: "Do Verifier Judgments Survive Abstraction? Measuring Verification Transport for LLM Agent Evaluation"
authors: ["Disha Sheshanarayana", "Sai Linisha Reddy Goli"]
institutions: ["Manipal University Jaipur"]
year: 2026
venue: "arXiv 2611.18471 (cs.CL), v1 2026-11-13; NeurIPS 2026 Workshop on Who Verifies the Agents? Toward Reliable Agent Development"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.18471"
code_url: "https://github.com/Disha1001/Verification-Transport"
citations: null
source: "raw/papers/sheshanarayana2026do.pdf"
added: "2026-11-17"
relevance: 4
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/lossless-context-offload]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/context-eviction-policy]]"
tags: ["llm-as-a-judge", "trajectory-verifier", "verification-transport", "observability", "abstraction-boundary", "macro-action", "delegation", "compaction", "provenance-restoration", "restore-counterfactual", "abstention", "false-certification", "verdict-contract", "deliberation-harm", "frozen-trajectories", "matched-views", "no-noise-floor", "open-weight", "gemini-flash", "workshop"]
---

# Do Verifier Judgments Survive Abstraction? Measuring Verification Transport for LLM Agent Evaluation

## TL;DR

An LLM verifier judges a frozen agent trajectory. Then part of that
trajectory is hidden behind a boundary marker, and the same verifier judges
it again. The task, trajectory, label and prompt stay fixed. The hidden span
is then put back verbatim and the verifier judges a third time. This is run
on 4 benchmarks with 7 mid-tier verifier backbones (no Claude, no GPT-5 class)
and 4 verdict protocols. Four results matter here:

1. **About a third of correct verdicts do not survive a hidden span, and more
   than half of those failures are abstentions.** Among verdicts correct on
   the native view, 31.0% (macro), 30.9% (delegation) and 20.3% (compaction)
   become wrong or unusable. Split by type, that is 13.4 / 13.2 / 12.2 points
   of wrong verdicts and 17.7 / 17.7 / 8.0 of no verdict.
2. **The error direction is set by the verdict contract.** The forced-binary
   judge (V-LLM) almost never abstains, and among correctly rejected
   negatives it certifies 41.3% as PASS once a span is hidden. Protocols
   allowed to abstain fail mostly by abstaining, but they still certify
   25–29%.
3. **A second pass without new evidence makes things worse.** The
   deliberative audit spoils 28.5% of correct first-stage judgments and
   repairs 8.7% of incorrect ones.
4. **Restoring the exact hidden events recovers 85.5% of failures, but that
   is an oracle.** Restoration means re-inserting the original events
   verbatim, so the view is the native one again. A lossy restore (action
   and tool names only) recovers 34–54%. Re-asking with the same abstracted
   evidence recovers 22–33%, which suggests part of the "failure" is
   instability, not information loss.

## Claims

- Accuracy measured on one view of a trajectory does not tell you a
  verifier's reliability once the evidence crosses a composed tool, a
  delegation or a compaction boundary. "Verification Transport" (keeping a
  correct verdict under such a change) is proposed as a separate criterion.
- Transport failures are often reversible: restoring execution provenance
  recovers most of them while degrading few judgments that had survived.
- A lower failure rate does not mean a more reliable verifier. Direct
  verification produces wrong verdicts, while evidence-grounded and
  deliberative protocols lose coverage (abstain).
- Recommendation (Discussion, untested as a system): evaluate verifiers under
  realistic abstraction boundaries and keep provenance "selectively
  restorable".

## Methods

- **Data.** 250 AgentProcessBench (APB) trajectories, scored step-level
  against a designated message, 758 ATBench tool-use safety trajectories,
  441 PlanRewardBench (PRB) pairwise comparisons and 754 AgentDojo
  trajectories. These are the instances eligible for construction, not the
  full benchmarks.
- **Matched views (App. B, Alg. 1).** Each trajectory is canonicalized into
  events, and complete TOOLCALL→TOOLRESULT pairs form the units. One block is
  replaced with a marker carrying a stable boundary id.
  - **Macro (C_M):** a maximal contiguous block with at least 2 complete
    pairs.
  - **Delegation (C_D):** a block with at least 1 complete pair. "No worker
    agent is instantiated and the task is not rerun."
  - **Compaction (C_C):** an earlier region, with recent evidence and the
    final call–result pair kept.
  - **Restore (C_a-R):** the exact stored events for that boundary are put
    back.

  Construction is deterministic and label-blind. No view is written by an
  LLM: there is no summary, no worker self-report and no macro description.
  The three "boundaries" differ only in which block is hidden and how it is
  labelled.
- **Protocols (App. D prompts).** All four use the same system frame, which
  includes "Do not infer that an omitted execution segment was good or bad
  merely because it is omitted."
  - **V-LLM** is a forced PASS/FAIL that "may not abstain".
  - **V-EVIDENCE** first judges whether the evidence is sufficient; if not,
    it must return ABSTAIN.
  - **V-AGENTIC** may request the exact provenance for one visible BND-id,
    then decides in stage 2, where it "cannot abstain".
  - **V-DELIBERATIVE** audits the frozen V-EVIDENCE answer against the same
    evidence. It gets no new evidence and may abstain.
- **Verifiers.** Gemini-3.7-Flash, Gemini-2.5-Flash, GPT-OSS-120B/20B,
  Gemma-4-31B, Gemma-4-26B-A4B and Llama-4-Maverick, all at temperature 0
  with structured outputs.
- **Metrics (App. C).** The denominators matter.
  - VTFR = share of C0-correct instances that are wrong or invalid under
    C_a. It equals Wrong Verdict + No Verdict.
  - RRR = share of those failures that are correct under C_a-R.
  - RDR = share of C0- and C_a-correct instances wrong under C_a-R.
  - FCR / FRR = explicit PASS on a correctly rejected negative, or explicit
    FAIL on a correctly accepted positive. These are computed on ATBench and
    AgentDojo only.
  - OBA = balanced accuracy, with abstentions counted as wrong.

  Pooling sums numerators and denominators across cells. 95% CIs come from
  a paired, benchmark-stratified bootstrap (App. F) and are shown only
  graphically (Fig. 6).
- **Ablations** use Gemini-2.5-Flash and GPT-OSS-120B on APB and ATBench.
  - Severity: hide about 25 / 50 / 75% of the "semantic atoms" in the block.
  - Restore fidelity: Requery (same abstracted evidence again); LossyR
    (ordered action and tool identities, without arguments, observations
    or contents); ExactR.

## Results

- **Native accuracy is low.** C0 OBA spans 21.8–77.1 across cells. On ATBench
  (binary safe/unsafe) it is 51–66 for every model and protocol, close to
  the 50% chance level. AgentDojo is highest at 52–77.
- **Pooled VTFR (Tables 1–2, §5.1).** Macro 31.0%, delegation 30.9%,
  compaction 20.3%. Per benchmark, macro is 25.5–32.2, delegation 26.1–32.9
  and compaction 16.2–24.7.
- **VTFR by protocol (macro / delegation / compaction).**
  - V-LLM 16.4 / 17.3 / 14.1
  - V-AGENTIC 25.0 / 23.6 / 18.2
  - V-EVIDENCE 34.5 / 34.0 / 18.7
  - V-DELIBERATIVE 49.3 / 50.2 / 30.7

  Offering the page-in (V-AGENTIC) gave a higher failure rate than the
  forced judge with no page-in.
- **Decomposition (Table 4, wrong verdict / no verdict).**
  - V-LLM: macro 15.8 / 0.7, delegation 16.5 / 0.8, compaction 13.2 / 0.9.
  - V-EVIDENCE: 10.6 / 24.0 (macro), 10.4 / 23.6 (delegation).
  - V-AGENTIC: 17.5 / 7.5 (macro).
  - V-DELIBERATIVE: 9.4 / 39.9 (macro), 9.2 / 41.0 (delegation).

  Every protocol has a wrong-verdict floor of about 9–18 points.
- **Diagnostics (Table 5, pooled over the three abstractions).**
  - Coverage, native → abstracted: V-LLM 98.4 → 98.7, V-EVIDENCE
    97.0 → 80.4, V-AGENTIC 97.1 → 93.3, V-DELIBERATIVE 94.6 → 67.3.
  - FCR: 41.3 / 29.4 / 26.2 / 24.8. FRR: 6.6 / 4.7 / 13.1 / 5.4.
  - The asymmetry favours false PASS, even under the explicit "don't infer
    omitted was good" instruction.
  - V-AGENTIC asked for provenance in 37.4% of abstracted instances (macro
    43.3, delegation 44.5, compaction 24.3).
  - V-DELIBERATIVE: BRR 8.7 vs HRR 28.5.
- **Model × protocol (Fig. 5).**
  - V-LLM VTFR runs from 10.1 (Gemini-3.7-Flash) to 22.0 (Llama-4-Maverick).
  - V-DELIBERATIVE reaches 57.9 (GPT-OSS-120B), 55.2 (Gemma-4-31B) and 49.7
    (Gemini-3.7-Flash), but only 14.5 on Gemma-4-26B-A4B.
  - The share of failures that are no-verdict under V-DELIBERATIVE is
    91.4% for Gemini-3.7-Flash.
  - Extreme cells: Gemini-3.7-Flash V-EVIDENCE on AgentDojo has VTFR 95.2
    (macro) and 96.4 (delegation); it abstains on nearly everything with a
    marker.
- **Restoration (§5.1 RQ3).**
  - RRR: 85.5% overall; macro 87.9, delegation 86.9, compaction 79.8.
  - RDR: 5.2% overall; 5.3 / 5.4 / 4.9.
  - No token cost is reported for any restore.
- **Severity ablation (Fig. 4a/b).** From A25 to A75, VTFR rises:
  - APB: 24.3 → 31.4 (Gemini), 21.5 → 29.6 (GPT-OSS).
  - ATBench: 17.7 → 21.1 (Gemini), 16.9 → 22.9 (GPT-OSS).
  - The rise is not monotone through A50 in every cell; GPT-OSS on APB
    peaks at 31.2.
- **Restore-fidelity ablation (Fig. 4c/d), RRR as Requery / LossyR / ExactR.**
  - APB: Gemini 22.5 / 42.8 / 80.0; GPT-OSS 24.6 / 34.0 / 82.5.
  - ATBench: Gemini 33.1 / 53.9 / 81.9; GPT-OSS 30.2 / 47.2 / 73.6.

## Critique / open questions

- **Is the ~31% failure among initially correct verdicts? Yes.** The VTFR
  denominator is C0-correct instances only. But it pools wrong verdicts with
  abstentions. Under macro, 17.7 of the 31.0 points are no-verdict, and for
  the abstain-capable protocols, abstaining on a view whose deciding evidence
  is hidden is arguably the right answer. The number that means "a correct
  judgment turned into a wrong one" is about 13%. The cleanest
  failure-direction number is FCR, measured on two benchmarks only.
- **What "restoring provenance" means and costs.** The original frozen events
  of the hidden block are re-inserted verbatim under the boundary id. That
  is close to the native view again, so the 85.5% says that going back to
  the native view mostly restores the native verdict. It is not a
  deployable mechanism. It needs the raw span to have been kept, and it
  costs that span's full tokens; the paper reports no token or latency
  figures. The deployable variants are weaker:
  - verifier-initiated page-in (V-AGENTIC) was requested in only 37.4% of
    cases and left VTFR at 18–25%;
  - an identity-only restore (LossyR) recovered about half as much as the
    exact one.
- **There is no noise floor.** No native-vs-native rerun is reported.
  Re-asking with *identical* abstracted evidence recovered 22–33% of
  failures at temperature 0. RDR (5.2%) measures flips between C_a and
  C_a-R, the latter nearly the native view. Both suggest that a material
  share of VTFR is verdict instability, not lost information. The failure
  set is also selected on one draw, so some recovery is regression to the
  mean.
- **Conditioning on native-correct under near-chance verifiers inflates the
  rate.** On ATBench, native OBA is 51–66 on a binary task, so many
  "correct" native verdicts are close to coin flips and are likely to flip
  under any perturbation. The paper notes that VTFR is computed on different
  native-correct subsets per protocol, which also makes the protocol ranking
  less clean.
- **"Delegation" and "compaction" are names for marker positions.** No
  subagent runs, and no summary or worker report is written. Macro and
  delegation are the same operation with a ≥2-pair vs ≥1-pair block rule,
  which is probably why their VTFRs agree to 0.1 point. On this box, the
  main session judges subagent work from a self-written prose summary that
  may misreport and carries no "hidden" marker. That is a different
  manipulation. The evidence already in the graph on it is
  [[literature/papers/xu2026what]] (report-only readers 52.0% vs 92.7%) and
  [[literature/papers/zhu2026who]] (the account bends the judge toward
  PASS). What this paper adds is the cleaner case: even with an *explicit*
  hidden-span marker and an instruction not to infer, judges certify.
- **The PASS criterion is partly an absence test.** The AgentDojo-style
  prompt's PASS clause 2 is "the trajectory does not show the agent carrying
  out an unauthorized … action". Hiding the span where the action happened
  satisfies it literally. So part of FCR is the contract doing what it says,
  not the model misreading. The general lesson is that absence criteria
  over a partial view certify by construction.
- **Which span gets hidden is not controlled.** Selection is label-blind and
  is not checked for whether the block holds the deciding evidence. There is
  no arm that hides an irrelevant span of matched size, so sensitivity to
  hidden *evidence* is not separated from sensitivity to *any* edit.
- **Reporting gaps.** No per-cell n. CIs appear only as a figure. No
  hypothesis tests on protocol or boundary differences. The C0 differences
  across protocols complicate every cross-protocol claim, as the authors
  note.
- **Transfer limits.** No frontier verifier was tested; Gemini-3.7-Flash is
  the strongest. The frontier judges in [[literature/papers/xu2026dont]] were
  nearly flat under misleading accounts, so this box's Opus judges may
  transport better. That is not measured here.
- **Fit with the graph.**
  - The FCR asymmetry is the controlled, multi-backbone version of the
    false-pass direction in [[literature/papers/im2026when]].
  - It mirrors that paper's "truncated digest → false refusal" in the other
    direction. Here hiding mostly yields PASS for a forced judge, or ABSTAIN
    for a sufficiency-checking one.
  - The restore ladder is a verifier-side instance of the restore
    counterfactual in [[literature/papers/shen2026what]].
  - Low page-in uptake matches [[literature/papers/busch2026ctxagent]] and
    [[literature/papers/fan2026empirical]].
  - xia2026does (judge input assurance, ingested in parallel) is the
    nearest neighbour.

## Trust signals

- **Credibility:** 2.
  - For: code is released. The matched-view construction is deterministic
    and label-blind and has four stated validity checks. All prompts are
    printed in full. 7 backbones × 4 protocols × 4 benchmarks. Wrong-verdict
    and no-verdict are separated. Two ablations point the same way in all
    four model × benchmark cells. Limitations are stated.
  - Against: two authors at one university (student addresses); a workshop
    paper; 4 days old; no citations. There is no noise-floor arm (the
    Requery and RDR numbers imply a material one) and no irrelevant-span
    control. Verifiers are near chance natively on at least one benchmark.
    No per-cell n, and no significance tests. The views are synthetic
    markers, not real summaries.
  - Not 3: the headline rate is uninterpretable without a rerun baseline,
    and the paper supplies data suggesting the baseline is large. Not 1:
    the code and prompts are released and the design is a real matched
    counterfactual.

## Follow-up

- **Relevance:** 4. It adds a cross-backbone measurement to two existing
  concepts:
  - [[concepts/evidence-gated-completion]]: the verdict contract sets the
    error direction under hidden evidence. The abstain option cuts false
    certification only from 41% to 25–29%, and re-auditing without new
    evidence harms about 3:1.
  - [[concepts/lossless-context-offload]]: a judge's restore ladder; the
    exact span recovers about 80%, identities about 45%, a re-ask about 28%;
    offered page-in was used in 37%.

  It also bears on [[concepts/hierarchical-delegation]] (parents verifying
  children) and [[concepts/hce-evaluation]] (judge validity per view). Not 5:
  the "delegation" and "compaction" are markers, not subagent summaries or
  LLM compaction; there is no noise floor and no Claude-class verifier.
- **For this box** (our inference):
  - When the main session judges a subagent's claim of success, a span it
    cannot see counts as unverified. It is not a pass. Read the artifact
    (or the subagent's transcript under `~/.claude/projects/`) rather than
    re-asking or reasoning harder over the summary.
  - Requery and deliberation-without-evidence were the two worst repairs
    measured.
  - Subagent briefs that ask for "what you did" should ask for paths and
    commands that can be re-read. An identity-only list recovered about
    half of what the raw span did.
- **Candidates** (cited, not in the graph):
  - Weng, Feng & Xie 2026, "Beyond accuracy: policy invariance as a
    reliability test for LLM safety judges" (arXiv 2605.06161).
  - Fan et al. 2026, AgentProcessBench (arXiv 2603.14465).
  - Zhou et al. 2026, "Counterfactual trace auditing of LLM agent skills"
    (arXiv 2605.11946).
  - KC 2026, BabelJudge (arXiv 2606.22329).
  - Han et al. 2026, SkillTV-Bench (arXiv 2608.05573).
