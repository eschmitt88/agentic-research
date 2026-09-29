---
kind: paper
title: "Don't Read the Log: Execution Traces Contaminate Verifiers in Video-Generation Agents"
authors: ["Jian Xu"]
institutions: ["RIKEN"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.28564"
code_url: null      # "the generated clips, labels, judging harness, simulation code and all per-call outputs will be released" — no link yet
citations: null
source: "raw/papers/xu2026dont.pdf"
added: "2026-09-29"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/shared-substrate-contagion]]"
tags: ["llm-as-judge", "vlm-judge", "verifier-contamination", "execution-trace", "self-report", "least-privilege-judging", "evidence-typing", "repair-loop", "reward-hacking", "false-accept-ceiling", "checker-laundering", "video-generation", "negative-result-for-prompting"]
---

# Don't Read the Log: Execution Traces Contaminate Verifiers in Video-Generation Agents

## TL;DR

Video-generation agents show their multimodal judge the agent's own trace,
plan and narration alongside the clip, for diagnosis. Holding the frames
fixed and varying only that auxiliary text, three open-weight Qwen-VL
judges (7B, 8B, 32B) answer from the text: on 109 author-labelled
"near-miss" clips, a supportive execution log raises false accepts from
0.17 / 0.19 / 0.07 to 0.78 / 0.90 / 0.83, and "use only the frames" leaves
0.41–0.63. Frontier closed judges barely move. In a simulated repair loop
the judge's false-accept rate at round 0 caps the true pass rate of *any*
repair policy at 1 − α0 (8B naive: judge 1.00, true 0.28). The fix is
structural: **least-privilege judging**, where each requirement declares
its evidence type and the judge sees only that evidence. It restores true
pass to 0.86 / 0.92 at 3× cost, and process-requirement accuracy stays at
0.89–1.00.

## Claims

- **Auxiliary text is a contamination channel for visual requirements.**
  The ideal judge satisfies J(r; v, τ, n, s) = J(r; v). Measured by fixing
  v and flipping the text between "supports" and "contradicts".
- **The log dominates, and it does not shrink with scale inside the
  family.** The 32B judge has learned to distrust narration (0.17) and
  burned-in subtitles (0.10), "but it treats a tool log as ground truth."
- **Near-miss is the trigger.** A text shift moves the verdict most where
  the visual log-odds are near zero. On UCF101, with unambiguous frames, a
  supportive log barely moves false accepts. There a contradicting log
  produces false *rejects*: acceptance of correct clips falls from
  0.80–0.89 to 0.00–0.21, and accuracy drops to 0.50–0.56.
- **Plan-derived vs artifact-derived text.** A `status: success` or a
  synthesized narration is a function of the plan, identical whether the
  generator succeeded or failed, so "it can shift a judge's acceptance
  threshold, but it cannot help the judge separate a failed clip from a
  successful one." A checker verdict logged into the trace is
  artifact-derived and is trustworthy "exactly to the extent of the
  checker's accuracy."
- **It is not a task property but a learned-trust property.** Frontier
  judges' no-text baselines are no better than Qwen3-VL-8B's, yet they
  place no weight on the log. The author *conjectures* agentic training
  data teaches that tool outputs are authoritative.
- **Ceiling identity.** A clip accepted at round 0 never re-enters the
  loop, so true pass ≤ 1 − α0 for any repair policy, and always-regenerate
  attains it. It holds to two decimals in all five judge configurations
  (Table 4).
- **Laundering.** With stage false-accept rate α and deference p to an
  upstream PASS, α_k = α + p(1 − α)α_{k−1}. The observed 0.08 → 0.69 gives
  p = 0.92: "the 32B judge takes the 8B checker's word 92% of the time."
- **Least-privilege judging is a guarantee, not an empirical property.**
  Unlike "ignore the text" or "use a frontier judge", it "cannot be
  overridden by what the text says, and it holds for judges whose trust
  has never been measured."

## Methods

- **Channels.** Transcript (narration in the prompt), log (a tool-calling
  trace in the prompt), subtitle (narration burned into frames). Each is
  tested in a support and a contradict direction. **Note the static-audit
  log's content** (Appendix A): plan, `t2v_generate -> status: success`,
  `consistency_check -> passed`, `add_narration`, and in the two-event
  variant also `event_check(...) -> all events present: true`. It is
  therefore an explicit *verification claim*, not only a success status.
  The closed-loop initial trace is only `parse_request`, `plan` and
  `t2v_generate -> status: success`.
- **Judges.** Qwen2.5-VL-7B, Qwen3-VL-8B, Qwen3-VL-32B; 16 frames at 336
  px short side; greedy; JSON verdict. Two system prompts: default, and a
  visual-only prompt saying narration, logs and plans "are NOT evidence".
  Frontier: GPT-5.4-mini (all 109), GPT-5.5 (67 clips, 33/34), Claude
  Opus 5 (26 clips, 16/10), and DeepSeek-V4 reading Qwen3-VL-8B's
  frame-by-frame description (VideoWeaver's architecture).
- **Data.** A UCF101 control (10 classes, hard and easy splits, 100 clips
  each). A generated two-event benchmark from Wan2.2-T2V-A14B: 40 prompts
  of the form "e1, then e2", with a both / e1-only condition. The author
  labelled clips from 6-frame contact sheets. The clean set is 50 present /
  59 absent over 36 prompts. 13 ambiguous clips were excluded, and 4
  prompts were dropped as undecidable.
- **Closed loop.** A simulation over 36 labelled failures, at most 4
  rounds. Actions: regenerate (cost 1.0; draws a real labelled clip,
  present with probability 0.85), add narration (0.02), self verify
  (0.05). Planners: always-regenerate, best-of-K, cost-greedy, and an LLM
  planner (Qwen3-VL-8B). "+8B check" writes the 8B judge's verdict on the
  initial clip into the trace as a `vision_check` result.
- **Least-privilege routing.** Each requirement carries ev(r) ∈ {visual,
  process, audio, …}. The judge sees only π_ev(r). For visual requirements
  that means frames with EasyOCR-detected text Gaussian-blurred; process
  requirements see the trace; numeric artifact facts come from metadata
  tools.

## Results

- **Static audit (Table 1, default prompt, present / absent).** No text:
  0.96/0.17, 0.96/0.19, 0.82/0.07. Log supports: 1.00/0.78, 1.00/0.90,
  1.00/0.83. Log contradicts: 0.62/0.02, 0.08/0.00, 0.00/0.00. The Qwen3
  judges are at or below chance under a log.
- **Visual-only prompt.** It cuts supportive-log false accepts only to
  0.63 / 0.54 / 0.41, "still three to six times the no-text floor". It
  barely helps in the reject direction (Qwen3-VL-32B: 0.14 acceptance of
  present clips under a contradicting log).
- **Frontier judges.** GPT-5.4-mini false accepts go 0.12 → 0.14 under a
  supportive log, and it accepts 0.90 of correct clips under a
  contradicting one. GPT-5.5 and Opus 5 are flat on their subsets.
  DeepSeek-V4 via description: 0.06 → 0.09.
- **Multi-shot consistency (17/17 shot pairs).** With "asset locked" in
  production memory, 8B accepts every inconsistent pair (0.47 → 1.00) and
  32B goes 0.29 → 0.47. With "re-cast", 32B rejects every consistent pair
  (1.00 → 0.00). GPT-5.4-mini does not move. **Opus 5 does move here:**
  consistent-pair acceptance falls 1.00 → 0.76 under "re-cast" (Table 2),
  although the prose says it does not.
- **Closed loop (Table 3).** Naive 8B: 72% of failures are accepted at
  round 0, and every planner except cost-greedy ends at judge 1.00 / true
  0.28. The LLM planner chose regenerate 13/13 times; "it is not gaming
  the judge". Naive 32B: round-0 0.25, true pass 0.75. 32B + injected 8B
  check: round-0 0.69, true pass 0.31.
- **Least-privilege (LP).** Hacked rate 0.14 (8B) and 0.08 (32B), equal to
  each judge's no-text floor. True pass is 0.86 and 0.92, and the judge's
  report becomes informative again (judge 0.94–1.00). Cost per episode
  rises 0.36 → 1.11 because failures are actually regenerated.
  OCR-masking takes subtitle false accepts from 0.50 to 0.14 with present
  recall unchanged at 0.97. Process requirements score 0.89–1.00 under
  both NAIVE and LP (Table 6).
- **What does not work.** An "evidence-first" describe-then-judge prompt
  halves the subtitle effect (0.50 → 0.28) but cuts recall from 0.97 to
  0.77.
- **By-product: generator controllability.** A third of e1-only clips
  completed e2 anyway (12/34, 12/34, 12/40 per batch), and an explicit
  negation ("she never drinks") did not help.

## Critique / open questions

- **The 78–90% headline is for a log that asserts verification.** The
  static-audit trace includes `event_check -> all events present: true`,
  which is an artifact-derived claim by the paper's own taxonomy. The
  abstract's "a trace that reports a successful tool call" undersells
  that. The bare-status effect is measured only in the loop, where it is
  large for 8B (round-0 0.72 vs LP 0.14) and much smaller for 32B (0.25 vs
  0.08). The section heading says it: "Scale helps against the plan, not
  against a claim." For transfer, the dangerous line is a claimed check
  result, more than a success status.
- **The closed loop is a simulation, and its outcome is an identity.**
  Regeneration success is fixed at 0.85 by drawing from the labelled pool.
  The planner is irrelevant because the hack happens before it acts. So
  "honest agents exploit it anyway" means "the judge certifies failures at
  round 0". No agent behaviour is involved. The 0.28 → 0.86 LP gain is
  1 − α0 restated, not a measured harness outcome.
- **The laundering checker was injected by the experimenter.** Its verdict
  is the 8B judge's own. p = 0.92 comes from fitting a one-parameter model
  to one pair of rates (0.08 and 0.69, with α_{k−1} = 0.72).
- **"Frontier judges are unmoved" rests on thin subsets.** Opus 5 was run
  on 26 clips with only 10 absent. GPT-5.5 was run on 67. DeepSeek-V4 is
  not a vision judge at all. Opus 5 moved in the multi-shot re-cast arm.
  The mechanism story (learned trust from agentic training data) is a
  conjecture, and there is no training-data evidence.
- **Small, author-labelled, one open-weight family.** 109 clips from 36
  prompts, labelled by the author with 13 exclusions. Everything is Qwen.
  The limitations section says as much. An AI-use statement covers text,
  code and analysis.
- **Least-privilege presupposes separable requirements.** The author
  admits routing cannot remove evidence baked into the artifact (a
  generated whiteboard that spells out the plan). The same limit applies
  to ML artifacts: a results file the agent wrote *is* the agent's
  account. Evidence typing also assumes someone types the requirement
  correctly. "A one-token change" to the planner's checklist prompt puts
  that typing in the actor's hands.
- **Unaddressed:** the judge that reads a *correct* log is also moved
  (UCF101 false rejects). So the cost of contamination is not only false
  accepts but wasted regeneration. The paper prices this qualitatively
  ("a costly, pointless regeneration") and not in the loop.

## Trust signals

- **Credibility:** 3. RIKEN is a reputable national lab, but the paper
  has a single author with no track record visible in the PDF. It is an
  unreviewed cs.CR preprint, days old. Artifacts are promised ("will be
  released"), not linked. Labels are the author's own on a small set,
  frontier arms are budget-truncated, and an AI-use statement is
  attached. It is held at 3 by its internal discipline. The ceiling
  identity is checked against the simulation to two decimals, the paper
  reports what does not work (visual-only prompt, evidence-first
  prompting), it separates plan-derived from artifact-derived text, and
  it states its own limits. Trust the direction and the structural fix
  more than the specific magnitudes.

## Follow-up

- **Relevance:** 4. It is the cleanest measured case in the graph for
  "never let the verifier read the actor's account". It shows the account
  wins *even with the artifact in view*, that a prompt instruction does not
  remove the effect, and that a repair loop cannot outrun its gate's round-0
  false-accept rate. It supplies a cheap structural remedy (per-requirement
  evidence routing) that keeps process diagnosis intact. Not a 5: the domain
  is video, the open-weight judges are one family, and the frontier judges
  this box actually runs were barely moved.
- **Transfer argument (the digest said it "needs an argument").** The
  mechanism is not visual. It is a judge weighting a *provenance* channel
  (tool log) over a *content* channel on an uncertain case. An ML-research
  agent's trace is full of `status: success`, "tests passed" and
  "val_acc=0.93" lines. The analogue of a visual requirement is an
  artifact requirement ("the model beats the baseline on the held-out
  split"), and its least-privilege projection is a recomputation from the
  artifact, not the log line. Near-misses (a metric just below threshold, a
  partially run sweep) are exactly where the paper says the shift bites.
  The *magnitude* does not transfer: Claude-family judges were flat here.
  The *structural* argument does, since it holds whoever the judge is.
- **[[concepts/evidence-gated-completion]].** The main home. Guidance #3
  ("verify against external state, never against the agent's account")
  now has a measured violation with the artifact present, a ceiling
  identity for gate + repair loops, and a routing rule. See its
  2026-09-29 section.
- **[[concepts/programmable-evaluator-oracle]].** The access-restriction
  clause measured for model judges: a 7B → 32B increase does not remove
  the effect, and a changed training distribution (frontier) apparently
  does.
- **[[concepts/typed-claim-partition]].** Evidence type is a property of
  the (evidence, requirement) pair. A tool log is hard evidence for a
  process requirement and zero evidence for an artifact one. A
  `tool_match` weight that ignores the requirement overweights
  `status: success`.
- **[[concepts/shared-substrate-contagion]].** Within one pipeline the
  trace is the substrate. The paper measures a deference coefficient
  (p = 0.92) and runs the no-channel arm (the same judge without the
  injected line).
- **Not [[concepts/information-firewall]]** (the digest's framing). That
  concept governs what the *agent under evaluation* may see (method,
  answers, corpus, time). This paper governs what the *verifier* may see
  of the actor. It is the evaluator's access restriction, which
  programmable-evaluator-oracle and evidence-gated-completion already
  own. Filing it under the firewall would merge two opposite directions.
- **Repo read.** Orchestrated `/curate` runs are a checker → judge
  pipeline: subagents report, and the parent commits on the report. A
  subagent's "grep-verified" is a logged checker verdict in exactly the
  laundering sense. The least-privilege version is the parent re-running
  one cheap artifact check (note exists, `kg_lint.py` passes) instead of
  reading the claim.
- **Candidates.** Andrade et al. 2026 (ICLR, agreement bias in MLLM
  verifiers of agent trajectories) and Zou et al. 2026 (ACL,
  informativeness bias) are the peer-reviewed anchors for "VLM judges
  over-rely on text", which the paper treats as established. The sibling
  digest item qin2026 (agents can tamper with their own traces) compounds
  this one: the trace is both mutable and trusted.
