---
kind: paper
title: "Does the Judge See What the Evaluation Requires? Input Assurance for LLM Agent Evaluation"
authors: ["Boming Xia", "Qinghua Lu", "Runze Li", "Dino Sejdinovic", "Liming Zhu"]
institutions: ["Responsible AI Research Centre, Adelaide University", "CSIRO", "UNSW Sydney", "Adelaide University"]
year: 2026
venue: "arXiv 2611.02676 (cs.SE), v1 2026-11-02, preprint (IEEE two-column format; artifact behind an anonymized link, so probably under double-blind review)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.02676"
code_url: "https://anonymous.4open.science/r/evaluation-artifact-342B/"
citations: null
source: "raw/papers/xia2026does.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/evidence-gated-completion]]"
tags: ["llm-as-a-judge", "input-assurance", "input-contract", "renderer", "trajectory-packet", "structured-tool-calls", "content-hash", "manifest", "staging", "time-of-check-time-of-use", "test-oracle", "mutation-testing", "n-version-checkers", "fault-injection", "recording-stub", "fail-stop", "abstention", "agentprocessbench", "bfcl", "tau2-bench", "case-study", "single-pipeline"]
---

# Does the Judge See What the Evaluation Requires? Input Assurance for LLM Agent Evaluation

## TL;DR

An LLM judge sees an agent step only through a text *packet* that a
renderer builds from the recorded trajectory. The paper asks what shows
that this packet carries what the evaluation needs. It proposes four
"input-assurance" obligations, each with a direct test:

1. required content survives rendering;
2. each checker covers the requirements assigned to it;
3. a failed check stops later use;
4. the judge reads the bytes that were last checked.

It audits one existing pipeline: the authors' own earlier step-level judge
study on 30 AgentProcessBench trajectories (240 packets, five judge
configurations). On the real packets, every hash matched, both independently
written checkers reported all ten requirements met, and staging succeeded.
Yet:

- The renderer had dropped the structured `tool_calls` field. **53 of 60
  judged steps were blank** (26/30 error targets, 27/30 comparison
  targets). The hashes matched because they were computed from the same
  defective text.
- Restoring the calls cut one judge configuration's (J0, `claude-opus-5`)
  later-context contrast from 13 to 2. The reduction was 11, or 0.37 per
  trajectory, 95% cluster-bootstrap [0.17, 0.53].
- No judge said a call was missing. J0 abstained on the blank steps. Three
  of the other four flagged most blank steps as errors, including correct
  ones.

Obligations 2–4 are shown with mutation cases, code inspection and
constructed integration tests, not on the real packets. Three added
controls (stop on failure, exact-byte checks before dispatch, reverse
call/result matching) stopped 18 of 19 constructed fault cases. The 19th, a
stale script, never ran.

## Claims

- A content hash shows only that bytes are unchanged. It does not show that
  the bytes contain what the evaluation requires. A manifest written by the
  same build that produced a defective packet "can match its defective
  output exactly, so a matching hash does not authorise use."
- The judge's input contract is part of the measurement process, along with
  source selection, rendering, checking and aggregating responses.
  Evaluations should report evidence that judge inputs met their
  requirements (construct validity, Ralph & Tempero 2018).
- Judge outputs cannot serve as the check. The judges' answers were
  affected by the omission but did not identify it, so the input must be
  validated directly against its source.
- Two checkers written separately from one plan cannot be assumed to fail
  independently (Knight & Leveson 1986). Here both shared the same
  one-directional matching blind spot.
- Each obligation needs its own kind of evidence: comparison with source
  records (1), violating cases per requirement (2), integration tests of
  failure handling (3), and byte checks at the point of use (4).
- The four tests can run without live judge calls, using a recording stub,
  and can serve as regression tests when representations or components
  change.

## Methods

- **Cohort.** AgentProcessBench BFCL and tau2 subsets, with per-step labels
  +1 / 0 / −1. Of 500 trajectories, 99 met the selection rules after
  excluding 32 used earlier. A previous study drew 30 at random with a
  fixed seed (14 BFCL, 16 tau2), and this paper reuses that cohort. The
  30 trajectories come from 22 distinct user requests, which form the
  bootstrap clusters.
- **Targets.** Each trajectory contributes an *error target* (the first
  tool-calling step that begins a run of −1 labels) and a *comparison
  target* (the nearest earlier tool-calling step labelled +1). A *prefix
  packet* ends at the target step, before its tool result. A *full packet*
  continues to the end of the trajectory. Benchmark labels select the
  targets but are not in the packets.
- **Renderings.** O is the original archived packet, byte-identical and
  matching its recorded hash. R (call-restored) inserts `[TOOL CALL]
  name(arguments)` after each originating message, throughout the packet.
  R is not a complete rendering: tool results still lack tool names and
  call IDs, and tau2 packets lack tool definitions. 2 target types × 2
  extents × 2 renderings = 8 cells of 30, or 240 packets.
- **Judges.** Five configurations: J0 `claude-opus-5`, J1 `gpt-5.6-terra`,
  J2 `gpt-5.6-luna`, J3 `claude-sonnet-5`, J4 `qwen3.6:27b`. Two draws per
  packet (480 per configuration). Answers are `yes` / `no` /
  `cannot_decide`. A consensus error flag needs both draws to say `yes`.
  - J0 ran in fresh sessions that read the packet from a file, with O and
    R draws randomly interleaved.
  - J1–J4 ran later and got the packet text in the prompt, with no tools.
  - Model, prompt wrapper, schema enforcement, reasoning settings, client,
    delivery and date all differ between configurations. The authors
    therefore interpret differences only at the configuration level.
- **Contrast.** G = (F_E − P_E) − (F_C − P_C), where F and P count
  consensus flags on full and prefix packets, and E and C are error and
  comparison targets. G asks how much later context raises error flags on
  error steps beyond what it does on correct steps.
- **Pipeline.** The builder writes R packets, a manifest of expected hashes
  for all 240 packets, and a job list. Staging copies packets to a separate
  folder and checks their text against the manifest. Emission writes
  dispatch scripts. Two checkers were written separately from one plan that
  assigned each the same ten requirements:
  - the *inline checker* runs inside the builder, before staging;
  - the *disk checker* rereads source records and archived packets
    afterwards.
- **Checker audit.** The authors read both checkers' code and ran them on
  valid inputs and 38 constructed *mutation cases* (34 shared by both). A
  miss is a clean exit while the mutation is present.
- **Integration tests and controls.** 12 integration tests, then 19
  constructed fault cases (8 reused from the mutation cases or integration
  tests, 11 new) under four cumulative conditions:
  - P0: as built; later steps run after failures.
  - P1: stop when a step errors or a required checker does not report
    success.
  - P2: also record exact-byte hashes at staging and check archive
    against staged copies before dispatch. The stub re-checks the hash at
    read time.
  - P3: also reverse matching, so every tool result must answer a call,
    including calls by simulated users.

  A recording stub replaces the judge and hashes whatever packet it reads.

## Results

- **Rendering (obligation 1, real packets).** 26 of 30 error targets and 27
  of 30 comparison targets were blank in O. The agent had left its message
  text empty and put the action only in `tool_calls`. The other 7 had text
  but lost their calls. Every O packet matched its recorded hash.
- **Judge flags, consensus counts per 30 targets (Table III), O → R.**

  | Key | Model | Error prefix | Error full | Comp. prefix | Comp. full |
  |---|---|---|---|---|---|
  | J0 | claude-opus-5 | 2 → 20 | 17 → 22 | 0 → 2 | 2 → 2 |
  | J1 | gpt-5.6-terra | 22 → 19 | 13 → 18 | 15 → 9 | 6 → 3 |
  | J2 | gpt-5.6-luna | 22 → 20 | 12 → 21 | 23 → 7 | 8 → 5 |
  | J3 | claude-sonnet-5 | 8 → 21 | 9 → 15 | 7 → 9 | 4 → 3 |
  | J4 | qwen3.6:27b | 21 → 19 | 18 → 16 | 17 → 7 | 15 → 4 |

- **J0's contrast.**
  - With O, G = 13, and 14 of the 17 error-full flags were on targets that
    were blank in their prefix packets.
  - With R, G = 2. On error prefixes 18 targets gained a flag and none lost
    one.
  - Reduction 11 (0.37 per trajectory, 95% bootstrap [0.17, 0.53]). The
    restored contrast's interval is [−0.05, 0.21] per trajectory (point
    value 2/30 ≈ 0.07, our arithmetic).
  - With O, J1–J4 had contrasts of 0, 5, 4 and −1. From the table, R gives
    5, 3, 0 and 0 (our arithmetic; the paper does not state these).
- **What the judges did with blank steps.**
  - J0 had both draws abstain on all 26 blank error prefixes, and at least
    one draw abstain on each of the 27 blank comparison prefixes. No J0
    draw abstained on these targets in R.
  - On the same blank O prefixes, J1–J4 flagged 20, 20, 7 and 19 of 26
    error targets, and 13, 22, 7 and 17 of 27 *correct* comparison
    targets.
  - At least one draw abstained on 17 of 30 O error prefixes under J3, and
    on 3–6 under J1, J2 and J4.
  - No judge named the missing call. Flagging a blank error step does not
    show detection, because the label was assigned from the full
    trajectory.
- **Checkers (obligation 2, mutation cases and inspection).**
  - Both checkers verified that each call had a result, but not that each
    result had a call. The inline checker's loop starts from calls and
    skips messages without them.
  - An added tool result with no matching call passed both. It was the
    only one of the 34 shared mutation cases that both accepted.
  - The inline checker exited cleanly despite 2 of the 34 violations, the
    disk checker despite 1. Their error output named the violated
    requirement in 23 and 30 cases.
  - The disk checker rejected an invalid JSON argument string that the
    inline checker accepted.
  - The staged-file requirement went to both checkers, but neither could
    check it. The inline checker runs before staging. The disk checker
    reads archived packets, not staged copies.
  - The disk checker's hash comparison built its expected hash from text
    it had loaded and then hashed the same text again, so it could never
    fail (a redundant assertion).
  - Simulated users made 40 of the cohort's 311 calls. A matcher that took
    calls only from assistant messages flagged all 40 of their results as
    unmatched.
- **Failure handling and use (obligations 3–4, constructed integration
  tests).**
  - A build that exited with an error still let staging and emission
    succeed, and the stub read the altered packet in both draws. Staging
    passed because the failed build had written the packet and its
    manifest hash from the same altered text.
  - A packet changed after staging was read as changed: a
    time-of-check-to-time-of-use gap.
  - A change from LF to CRLF made before staging passed, because staging
    hashes normalised text.
- **Controls (Table II, 19 fault cases).**
  - P1 stopped 11.
  - P2 refused dispatch in 4 more: 3 packets changed after staging and 1
    extra staged file.
  - P3 stopped the 3 unmatched-result cases, which had passed P2 because
    the files were unchanged.
  - The stale script was unused under every condition.
  - Under P0, 14 of the 18 cases stopped by P1–P3 had led to an altered or
    stale packet being read. Both no-fault cases completed under all
    conditions.
  - By inspection, P2/P3 would still accept the CRLF change. Exact-byte
    hashes recorded at build are recommended but were not tested.
- **The four proposed tests (§VI).**
  - Test 1, render required content: put an action only in a structured
    field with empty text, then check name, arguments and position
    against the source. Also check that excluded fields, such as labels or
    the target's tool result in a prefix packet, stay out.
  - Test 2, violate each requirement, at the object and pipeline step it
    covers. Check one-to-one relations in both directions and cover every
    participant.
  - Test 3, continue after a failure, with stale outputs present.
  - Test 4, change a checked file before use.

  Abstentions should be reported separately from `no`. The answer schema
  here had no category for defective input.

## Critique / open questions

- **Digest vs full text: slightly above.** Every digest number is in the
  PDF, but "coexisted with violations of all four obligations" drops the
  abstract's own qualifier ("the last two in integration tests"). On the
  real packets only obligation 1 was violated. The obligation-2 evidence
  is latent checker gaps found with mutation cases and code reading.
  Obligations 3–4 were shown on faults the authors constructed. The
  digest also leaves out the one measured consequence for judging (G
  13 → 2) and the finding that no judge noticed the omission.
- **The case was chosen because the defect was known.** The paper calls it
  "one pipeline selected after its rendering omission was found." That
  makes it an existence proof, not a prevalence estimate. We do not learn
  how often published judge pipelines drop structured fields.
- **The controls are graded on their own fault suite.** The 19 cases were
  written with knowledge of the code, on the same cohort the controls were
  developed on. 18/19 is the hce-evaluation 10-20 point ("a verifier's own
  fault suite rewards its own evidence types") again. The authors say so
  and plan a transfer study with an assessor who has not seen the
  checkers. That study is not run.
- **The judge effect is one contrast in one configuration.** The
  significant result is J0's G, a difference of differences, not accuracy
  against labels. The paper does not report how much the omission changed
  any configuration's agreement with the benchmark labels, and its
  "flags on blank steps don't show detection" argument makes that hard
  to read. J0 differed from J1–J4 in delivery (file read vs prompt) as
  well as model, so "Opus abstains on blank input and GPT flags it" is
  not a model-level finding. n = 30 trajectories in 22 clusters.
- **The supplementary material is not in the arXiv PDF.** Per-target
  changes (S9), the checker outputs (S3), the cohort list (S1) and the
  judge-configuration details (§V–VI) are cited but not included. The
  artifact sits behind an anonymized 4open link.
- **Does it conflict with [[literature/papers/im2026when]]? No.** Im
  re-ran τ²-bench judging untruncated (tool results had been cut at 600
  characters, and transcripts over 12,000 characters reduced to head +
  tail). Agreement with human labels moved at most +0.020, with every CI
  spanning zero. The two results differ in three ways:
  1. *What was lost.* Im cut tool-result tails and transcript middles, and
     the assistant's actions mostly survived. Xia lost the judged action
     itself in 53 of 60 targets, so there was nothing to judge.
  2. *What was measured.* Im measured episode-level validity against
     human labels. Xia measured a within-judge contrast and flag counts.
  3. *Floor.* Im's gpt-4.1-tier judges were already passing 9–11 of the 11
     forbidden writes, and more input cannot rescue a judge that misses
     the event anyway.

  Im's own pipeline also shows the harmful case. Its *gate verifier's*
  digest cut the deciding field (order status at character 950 of 1,304)
  and falsely refused valid writes. Read together: input loss matters
  when it removes the fact the verdict depends on. An aggregate "did
  truncation move agreement" test averages over items where nothing
  decisive was cut. Xia's test 1 instead checks that the contract-required
  content survives, item by item. That is the check Im's verifier lacked
  and its judge did not need.
- **Fit with [[literature/papers/liu2026silent]].** Liu's pipeline errors
  sat around the scorer: whether it ran, how runs were logged, priced and
  re-graded. This paper adds the stage *before* the scorer: rendering,
  staging, dispatch. Both failure classes produce a plausible number and
  raise no error, and neither is caught by checking the scorer's value on
  a do-nothing input.
- **Not research agents.** The agents are BFCL and tau2 tool-use runs. What
  transfers is the shape of the defect. An assistant turn with empty text
  and a structured tool call is exactly how Claude Code writes `tool_use`
  blocks in its JSONL transcripts. Any box script that renders transcripts
  by joining `text` blocks (as `claude_sessions_lost_at_boot.py` does for
  user turns, harmlessly there) would drop the agent's actions in the same
  way.

## Trust signals

- **Credibility:** 3.
  - For: CSIRO and Adelaide authors with a track record in responsible-AI
    engineering (Lu, Zhu) and statistics (Sejdinovic). The artifact
    (plans, observations, code, replay scripts) is released. The authors
    state the limits plainly: one pipeline, chosen post hoc; controls
    developed on the same cohort; fault cases written with knowledge of
    the code; configuration-level interpretation only. The bootstrap is
    clustered by user request. A recording stub gives direct evidence of
    which bytes were read, independent of any judge.
  - Against: a 6-page preprint, not peer reviewed, 15 days old, no
    citations. n = 30 trajectories, one pipeline (the authors' own). The
    supplement holding the per-target data is not in the PDF, and the
    artifact link is anonymized. No accuracy-against-labels analysis.
    Judge configurations are confounded with delivery mode and date.
  - Not 4: a single self-selected case with constructed faults and no
    external validation. Not 2: an established group, released artifacts,
    and claims kept within what the case shows.

## Follow-up

- **Relevance:** 4. Material new evidence for
  [[concepts/hce-evaluation]]. Clause 4 hashes the evaluator, and this
  paper shows that a judge's *input* needs the same treatment and more: a
  hash certifies only the bytes between where it was taken and where it
  was checked. It also adds to [[concepts/programmable-evaluator-oracle]]:
  checkers are oracles that need their own violating cases, N-version
  checkers from one plan share blind spots, and "input defective" should
  be its own return value. It completes the input side of the
  evaluator-pipeline pair with liu2026silent and im2026when. Not 5: one
  self-selected pipeline, constructed faults, no measured effect on
  validity, and no research-agent setting.
- **Box mapping (our inference).**
  - The hce-evaluation 11-17 rule moves `/lint`'s `test/` scan from
    `log.md` to the harness JSONL transcript. That renderer must keep
    `tool_use` blocks (name + input), because a read of `test/` exists
    only there. Its regression test is this paper's test 1: plant an
    assistant turn with empty text and a `tool_use` `Read` of a `test/`
    path, and require a flag.
  - Any downstream LLM-judged metric should record an exact-byte hash per
    judge packet at build, check it again at dispatch, and stop dispatch
    if any upstream step failed. Report abstentions or `cannot_decide`
    separately from `no`.
- **Candidates** (cited, not yet in the graph):
  - Fan et al. 2026, AgentProcessBench (KDD 2026): step-level process
    labels for tool-using agents, the cohort used here.
  - Lù et al. 2025, AgentRewardBench (COLM 2025, arXiv 2504.08942). The
    planned transfer target, and already a candidate in im2026when.
  - Dev et al. 2026, "Judge Reliability Harness: stress testing the
    reliability of LLM judges" (arXiv 2603.05399; ICLR 2026 Agents in
    the Wild workshop).
  - Tan et al. 2026, "AgentChaos: chaos engineering for agent systems via
    programmatic fault injection" (arXiv 2608.06790; ASE 2026).
  - Zhao et al. 2025, "One token to fool LLM-as-a-judge" (arXiv
    2507.08794).
