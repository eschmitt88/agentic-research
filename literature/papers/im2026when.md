---
kind: paper
title: "When Evaluators of AI Agents Disagree: An Empirical Study of Native Scoring, LLM Judges, and Commit-Boundary Checks on tau2-bench"
authors: ["Sungbin Im"]
institutions: ["Seoul School of Integrated Sciences and Technologies (aSSIST)", "AIM Intelligence"]
year: 2026
venue: "arXiv 2611.14581 (cs.SE), v1 2026-11-10, preprint (Springer journal template; target venue not stated)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.14581"
code_url: "https://doi.org/10.5281/zenodo.23683806"
citations: null
source: "raw/papers/im2026when.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["llm-as-a-judge", "judge-validity", "reliability-vs-validity", "construct-validity", "human-reference-labels", "three-axis-labels", "commit-boundary", "premature-commitment", "boundary-evaluator", "partial-gold", "tau2-bench", "policy-compliance", "constant-baseline", "false-pass-bias", "pre-write-gates", "gate-false-rejection", "evidence-digest-truncation", "cluster-bootstrap", "holm", "null-result", "test-oracle"]
---

# When Evaluators of AI Agents Disagree: An Empirical Study of Native Scoring, LLM Judges, and Commit-Boundary Checks on tau2-bench

## TL;DR

Four evaluators score the same 2,160 τ²-bench airline and retail episodes:
the benchmark's own gold-state scorer ("native"), a transcript-reading LLM
judge, the same judge given the policy document ("judge-P"), and a
"boundary evaluator". The boundary evaluator is a deterministic check of one
bit of per-task gold (is a write expected?) against one observed bit (did a
WRITE tool call succeed?). They are validated against two-annotator,
three-axis human labels on a stratified 120-episode sample. Four results:

1. **Stable is not valid.** The gpt-4.1-mini judge agreed with itself on
   98.2% of episodes [0.975, 0.988] but with native scoring on only 56.2%
   [0.496, 0.627]. The disagreement runs one way: of the 1,007 episodes
   native failed, the judge passed 929 (92.3%). Native passed and the judge
   failed only 17 of 2,160.
2. **Judges pass forbidden writes.** The humans confirmed 11 "premature
   commitments" (a write in a task whose gold expects none). The gpt-4.1-tier
   judges passed 9–11 of them, with or without the policy and with or without
   truncated input. gpt-5.6 Sol passed 6 without the policy. With the policy
   it passed 0, but it also failed 40 of 55 fully correct episodes.
3. **The one-bit check was not beaten.** The boundary evaluator caught all 11
   by construction. No judge configuration was significantly above it on task
   or commitment correctness, though the intervals are wide (Sol vs boundary
   on A1: −0.029 [−0.140, +0.082]). Native scoring, which uses full gold, beat
   it overall (+0.102*).
4. **Gates did not help, and the trend in retail was adverse.** Premature
   commitments occurred in 11.1% of ungated episodes (60/540; airline 19.3%,
   retail 0.8%). Cumulative verification / correction / safe-termination
   gates produced no Holm-significant change. Two traced retail episodes show
   the LLM gate verifier falsely rejecting valid writes because its evidence
   digest had cut the order status or the customer's authentication.

## Claims

- Native scores, judge verdicts and trace checks are not interchangeable
  estimates of one "success". Each has different information access (full
  gold / no gold / partial gold; Table 2), so each sees different failures.
  Choosing the evaluator chooses the conclusion (§7.1).
- A judge's error direction is a property of its model tier and its prompt
  contents, not a fixed bias. gpt-4.1-tier judges are false-pass dominated
  (A1 false pass 39.9%, false fail 1.8%). Sol without the policy is balanced
  (13.1% / 13.2%). Sol with the policy over-refuses (false fail 42.0%).
- Policy compliance (axis A2) is unsolved. Every evaluator's weighted
  agreement (0.595–0.672) was at or below the constant rule "always
  compliant" (0.688 [0.591, 0.776]). The two human annotators reached only
  κ = 0.306 on A2 after recalibration.
- Gates and evaluators are both software needing separate validation. The
  same trace format that scores the agent localized a gate defect that
  aggregate rates could not.
- Practice (§7.2): always log commit boundaries; never ship on an
  unvalidated judge score alone; budget for the oracle each evaluator needs
  (a write-expectation bit per request type is cheap, a full goal state is
  not). The boundary check is offered as a floor, not a technique, that
  runtime-enforcement systems (AgentSpec, Progent, ShieldAgent) should match.

## Methods

- **Benchmark.** τ²-bench at commit 363133a, unmodified. 50 airline tasks
  (all) + 40 retail (test split). Chosen because the tool schema separates
  READ from WRITE tools, each domain has a natural-language policy, and many
  tasks require refusal.
- **Episodes.** 90 tasks × 2 agent models (gpt-4.1-mini, gpt-4.1) × 4 gate
  conditions × 3 repetitions = 2,160. Simulated customer: gpt-4.1-mini.
  Analysis unit is the task cluster (24 episodes per task). Cost ≈ USD 88
  plus USD 5.10 for judge-P (the ledger omits verifier calls, ≈13% low).
- **Gates (C0–C3, cumulative).** A `GatedAgent` wrapper intercepts WRITE
  calls. C0: none. C1: verification (a failure is logged and the write
  proceeds). C2: plus correction (up to 2 regenerations, then the last draft
  proceeds). C3: plus safe termination (a still-failing draft is blocked and
  the episode ends). The verifier is gpt-4.1-mini at temperature 0 with the
  policy in `<policy>` tags. **It sees a digest, not the conversation:** the
  last 20 messages, each cut to 800 characters, shrinking to the last 8 if
  the digest exceeds 6,000 characters. Rejected drafts never enter the
  transcript. Wrapper check: C0 vs stock agent on 50 airline tasks, mean
  native reward 0.36 in both, 7/7 discordant episodes.
- **Trace.** `policy_event_trace` v1.1 per episode: gate statuses (disabled /
  skipped / executed / not reached), each executed event carrying an
  `evidence_ref`, terminal state and `checkpoint_status`. A write counts as a
  commit only if the WRITE tool result is not an error. 2,160/2,160 traces
  passed schema validation.
- **Evaluators.**
  - Native: benchmark reward = 1.0.
  - Judge: gpt-4.1-mini, temperature 0.4, two calls, majority with a 1–1 tie
    counted as pass (ties ≈2%; alternative tie rules move agreement ≤0.018).
    Input is customer context plus transcript, tool results cut at 600
    characters, transcript head+tail kept beyond 12,000. No gold, no policy.
  - Judge-P: identical plus the full policy.
  - Boundary: write expected (gold action list has ≥1 WRITE) × write
    committed → correct_action / missed_action / premature_commitment /
    correct_refusal.
  - Extra judges on the 120 labelled episodes, untruncated: gpt-4.1 and
    gpt-5.6 Sol (no temperature control; 11/240 ties), each with and without
    the policy.
- **Human reference.** 120 episodes, stratified by domain × gate × disagreement
  pattern, with sampling weights summing to 2,160. Axes: A1 task correctness,
  A2 policy compliance (judged on end state after rubric v1.3), A3 commitment
  correctness. Two annotators, blind to model, gate condition, all evaluator
  outputs and gold. One is the author; the other is a product manager at the
  author's employer. Round-1 κ: A1 0.686, A2 0.317, A3 0.477. A pre-declared
  A2 recalibration raised raw agreement to 0.742 with κ 0.306 (AC1 0.595).
  62 episodes (80 axis items) were adjudicated; originals are preserved.
  Final: A1 68/52, A2 83/37, A3 67/53 yes/no.
- **Statistics.** Task-cluster bootstrap within domain (B = 10,000),
  weighted. Paired bootstrap for evaluator differences. C0 vs C3 uses a
  task-level sign-flip permutation test (B = 20,000) with Holm over 8 tests
  (2 pre-specified metrics × 4 domain-model cells).

## Results

- **Pairwise agreement, 2,160 episodes (Table 4).** Native–judge 0.562
  [0.496, 0.627]. Native–boundary 0.761 [0.700, 0.820]. Judge–boundary 0.782
  [0.732, 0.829]. Judge self 0.982. All three passed 52.6% and all failed
  2.7%. "Only native failed" was 23.0% and "only the judge passed" 20.1%.
  Native–judge agreement ran from 48.8% (airline, gpt-4.1-mini) to 68.3%
  (retail, gpt-4.1). Self-agreement was 96.5–99.4% in every cell.
- **Against human labels, weighted (Table 5).**

  | Evaluator | A1 | A2 | A3 | Overall | FP (A1) | FF (A1) |
  |---|---|---|---|---|---|---|
  | native | **0.818** | 0.653 | **0.758** | **0.766** | 0.070 | 0.112 |
  | judge | 0.584 | 0.672 | 0.575 | 0.496 | 0.399 | 0.018 |
  | judge-P | 0.622 | 0.671 | 0.600 | 0.521 | 0.378 | 0.000 |
  | boundary | 0.766 | 0.595 | 0.708 | 0.664 | 0.216 | 0.019 |

  Boundary − judge: A1 +0.182*, A3 +0.134*, overall +0.168*. Native −
  boundary: n.s. on each axis, +0.102* overall. Judge-P − judge: A1 +0.039*,
  A2 −0.001, A3 +0.025. The always-"yes" rule scores 0.568 on A1, 0.688 on A2
  and 0.556 on A3.
- **What the boundary check misses.** Of 53 human A3 = "no", it caught 24
  (weighted 43.7%): all 11 premature commitments plus 13 missed writes. Of
  the 29 it missed, 25 were expected writes that occurred with wrong content
  or timing. It failed 6 of 67 human A3 = "yes". The four gpt-4.1-mini judge
  configurations caught 6–9 of the 53, native 40.
- **Adjudication-free subset** (A1 n = 101, A2 89, A3 90): boundary − judge
  A1 +0.186*, A3 +0.129*; native − judge A1 +0.279*, A3 +0.218*. Both judges
  passed all 10 premature commitments there. A2 ordering shifted, so no A2
  ranking is claimed.
- **Truncation is not the cause (Table A2).** Untruncated re-judging (57/120
  transcripts exceeded 12,000 characters; median 11,797) moved agreement at
  most +0.020 with every CI spanning zero. Premature commitments passed: judge
  10/11, judge-P 11/11.
- **Judge tiers (Table A3, untruncated).** gpt-4.1 vs gpt-4.1-mini within
  ±0.022 on every axis. Sol (no policy) vs gpt-4.1-mini A1 +0.133
  [+0.016, +0.250]. Sol (no policy) vs boundary: A1 −0.029 [−0.140, +0.082],
  A3 −0.036 [−0.146, +0.078], overall +0.021 [−0.086, +0.134]. Sol vs native:
  A1 −0.081 [−0.164, −0.002], A3 −0.086 [−0.174, −0.002]. Sol + policy: A1
  0.580 (−0.156 [−0.270, −0.041] vs no policy), A2 0.460, caught 51 of 53 A3
  failures and failed 40 of 55 fully correct episodes. All six untruncated
  judge configurations were significantly below native on A1 and A3.
- **Premature commitments without gates (C0).** 60/540 = 11.1% [6.1, 16.5].
  Airline 58/300 (gpt-4.1 18.0%, mini 20.7%), retail 2/240 (0.8%).
- **Gates (Table 7, C0 vs C3).** Nothing survives Holm. Airline/mini
  premature commitments, discordant 11 vs 4 in C3's favour (p = 0.121). Retail/
  gpt-4.1 boundary success, 17 C0-only successes vs 4 C3-only (p = 0.030,
  Holm 0.240), the adverse direction. Safe termination fired 15 times, 14 of
  them for gpt-4.1.
- **Gate trace anatomy (§6.3, two episodes).** In TB-retail-60-C3-R1 the
  order-status field sat at character 950 of a 1,304-character tool result
  and was cut at 800. The verifier rejected five times ("not confirmed"), and
  C3 blocked a valid write. In TB-retail-32-C3-R2 the 20-message digest
  exceeded 6,000 characters, so the window shrank to 8 messages and dropped
  the authentication step. The verifier rejected ("not authenticated") and
  the agent gave up the write. Both: native fail, judge pass, boundary
  missed_action. Retail tool results average 976 characters (59.4% over 800),
  airline 663 (27.5%). The author did not fix the verifier and re-run, and
  did not measure how much of the aggregate the defect explains. The same
  truncation class was found and fixed in the annotation-packet builder
  before labelling.

## Critique / open questions

- **Digest vs full text: slightly above.** Every digest number is in the
  PDF. The digest says the gates failed to reduce forbidden writes
  *because* their evidence digest dropped fields. The paper shows something
  narrower. The digest defect produced false rejections of *valid* writes in
  two retail episodes. Retail barely has premature commitments (2/240), and
  the paper links the defect to the adverse retail trend, not to the airline
  null. It calls the cause "candidate" and leaves its share unmeasured. Why
  the gates did not cut airline forbidden writes is not explained.
- **"Caught all 11" is by construction.** A premature commitment is defined
  as human A3 = "no" with an unexpected write executed. The boundary
  evaluator's premature_commitment label is that same event read from gold.
  So 11/11 shows that human A3 agrees with gold write-expectation on those
  episodes, not that a cheap check found something hard. The judges'
  failure is the real finding. The boundary check's limits are equally
  clean: it is blind to wrong-content writes (25 of 29 misses).
- **The floor needs gold.** The boundary evaluator uses a per-task "write
  expected?" bit taken from the benchmark's gold action list. A deployment
  without that annotation cannot compute it. The paper says so (§3.3). The
  "no model call" framing invites the misreading that it is gold-free.
- **"Not significantly worse" is not "as good".** n = 120 with task-cluster
  CIs about ±0.11 wide. The author states the Sol–boundary intervals "do not
  show that the two methods are equivalent". No equivalence test was run.
- **The comparison rests on 11 events.** All premature-commitment counts
  are on 11 (or 10) episodes from essentially one domain (airline). Each
  "passed 6 of 11" carries a binomial interval of roughly ±0.3.
- **Weak reference on policy.** A2 κ = 0.306 after recalibration, so no
  evaluator can be shown valid on policy compliance against these labels. The
  second annotator is a colleague at the author's employer; blinding and
  released pre-adjudication labels partly offset this.
- **One vendor.** All agents, the simulator, the verifier and three of four
  judges are OpenAI models, mostly gpt-4.1-mini. The judge prompt is fixed.
  The 2,160-episode disagreement figures use the weakest judge. No Claude
  judge was tested.
- **The gates are as implemented.** C1–C3 results describe one gpt-4.1-mini
  verifier with a lossy digest. They are not evidence about gates as a
  class. No deterministic gate (e.g. block a write when the request type
  expects none) was tested, although the paper's own floor suggests one.
- **Not research agents.** Customer-service tool use with a policy document.
  What transfers is the structure: log the typed commit event, check its
  expectation deterministically, and validate the judge on labels before
  trusting it.
- **Fit with the graph.** zhang2026how ran a terminal verifier on the same
  benchmark's retail domain that also read a truncated dialogue (last ≤ 8
  messages × 300 characters), with 17% / 41% false rejection. This paper gives
  a traced mechanism for that kind of false rejection. bergen2026monitoring
  reported judge self-consistency with no validity check, and this paper
  measures the gap that note warned about. ray2026what's block-all
  calibration outcome reappears as Sol + policy. The input-assurance failure
  is the theme of xia2026does (ingested in parallel). Note that here the
  lossy input hurt the *gate verifier*. The *judge's* truncation was tested
  and did not matter.

## Trust signals

- **Credibility:** 3.
  - For: full replication package on Zenodo (traces, harness, evaluator
    code, verbatim prompts, both annotators' original labels, adjudication
    record, analysis scripts). Benchmark pinned and unmodified, wrapper
    no-effect check, schema validation of all traces. Clustered inference
    and Holm correction. Pre-declared recalibration procedure. Nulls
    reported as nulls, a sensitivity analysis without adjudicated labels,
    and an unusually explicit threats section. AI assistance disclosed
    (Claude for drafting and scripts, Codex for number re-checks).
  - Against: single author, graduate work at aSSIST,
    no funding, not peer reviewed, 7 days old, no citations. The second
    annotator is a colleague at the author's employer (QueryPie AI). The key
    contrasts rest on 11 events and n = 120. OpenAI-only models, mostly
    gpt-4.1-mini.
  - Not 4: no established group or venue, small labelled sample. Not 2:
    artifacts released and the statistical reporting is careful.

## Follow-up

- **Relevance:** 4. This is the first source in the graph to put a
  deterministic event check, a gold-state oracle and LLM judges of three
  tiers on the same agent episodes against independent human labels.
  It adds material evidence to [[concepts/programmable-evaluator-oracle]]
  (a one-bit typed-event oracle as a floor judges did not beat; judge error
  direction moves with tier and prompt), [[concepts/hce-evaluation]] (self-
  agreement is not validity; validate a judge against a constant-verdict
  baseline per construct) and [[concepts/evidence-gated-completion]] (a soft
  gate's evidence assembler fails closed on valid work). Not 5:
  customer-service domain, OpenAI-only judges, 11 critical events, and the
  floor needs per-task gold.
- **Box mapping (our inference).** The `/lint` HCE scan for `test/` access
  is already a boundary evaluator in this paper's sense: a per-phase
  expectation ("no test read during search") against an observed event. The
  paper supports keeping such checks deterministic and extending the
  pattern to other typed events with an advance expectation (writes under
  `raw/`, pushes, evaluator-file edits) before any LLM review of
  transcripts.
- **Candidates** (cited, not yet in the graph):
  - Norman, Rivera & Hughes 2026, "Reliability without validity" (arXiv
    2606.19544): judge test-retest > 0.95 alongside systematic bias.
  - Chen et al. 2026, "A judge should know what changed: construct validity
    for LLM-as-a-judge evaluation" (arXiv 2608.24419).
  - Advani 2026, "From confident closing to silent failure" (arXiv
    2606.09863; ICML 2026 FAGEN workshop): judges' AUROC ≤ 0.65 on 9,876
    τ²-bench trajectories. Already listed as a candidate in three notes.
  - Rabinovich et al. 2026, "Near-Miss: latent policy failure detection in
    agentic workflows" (GEM 2026): 8–17% of airline state changes are latent
    policy violations even when the final state matches gold.
  - Lù et al. 2025, AgentRewardBench (COLM 2025, arXiv 2504.08942); Verma
    et al. 2026, AgentJudgeBench (arXiv 2608.26623); Yang et al. 2026,
    CompliBench (arXiv 2604.12312); Mohammadi 2026, trajectory-judge (arXiv
    2609.00038); Pisupati et al. 2026, Counsel (arXiv 2606.21627).
