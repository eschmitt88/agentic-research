---
kind: paper
title: "Rethinking Verification in Agent Harnesses"
authors: ["Zhengxin Zhang", "Erkang Zhu", "Yin Lin", "Claire Cardie", "Bolin Ding", "Jingren Zhou"]
institutions: ["Cornell University", "Alibaba Group"]
year: 2026
venue: "arXiv 2610.34577v1 (cs.AI), 2026-10-25, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.34577"
code_url: null    # no code, prompts-only release; full intervention and critic prompts are reproduced in Appendix E
citations: null
source: "raw/papers/zhang2026rethinking.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["verification-placement", "completion-gate", "stop-hook", "independent-critic", "cheap-critic", "cross-model-critic", "self-verification", "execution-time-reminder", "message-role", "system-vs-user", "sham-refusal", "content-free-rejection", "review-depth", "review-width", "dollar-cost", "terminal-bench", "swe-bench-verified", "visualbench", "open-weight-agents", "no-intervals"]
---

# Rethinking Verification in Agent Harnesses

## TL;DR

The authors compare 17 verification designs placed at four points in an
agent's run:

- **L0**, a verification instruction at the start (system or user message);
- **L1**, reminders during the run (every third turn after the first edit,
  or after each edit; four at most);
- **L2**, a gate at the first stop (inspect, run-and-check, recheck, or a
  content-free "NOT complete, keep working" rejection);
- **L3**, a separate critic in a fresh context with the live sandbox.

The L3 critic is read-only. Its findings go back to the agent once, and the
agent's next delivery is accepted.

There are seven open-weight coding agents (Qwen3.7/3.8-Max, GLM-5.2,
DeepSeek V4-Pro/Flash, Kimi-K2.7-Code, MiniMax-M2.5). Runs use 3 trials per
model × task on a Terminal-Bench 2.1 subset (31 tasks stated) and a
SWE-bench Verified subset (about 30 tasks). An internal 18-task VisualBench
is rated by blinded in-house raters. **The critic is always the same
cheaper model, Qwen3.7-Plus**, for every agent. Cost is **dollars** at
Alibaba list prices with caching, not tokens.

Three results matter for this graph:

1. **The cheap separate critic is the best average trade on TB.** It gives
   +5.9 to +9.5 pp at 1.13–1.25× cost. On SWE it gives +3.9 to +6.6 pp at
   1.00–1.04×. The best L2 gate (early user rejection) gives +6.0 / +6.2 at
   higher cost. Rejection gates cost 1.60–1.76× (TB) and 1.48–1.56× (SWE).
2. **A refusal with no content still paid on these agents.** The L2
   "Reject completion" text names nothing. It fires on every first stop,
   whatever the state of the work. It still gave +4.0 TB / +5.3 SWE
   (automated marker), and +6.0 / +6.2 with the user role and an early
   trigger. Early user rejection was positive on SWE for all seven models.
   This differs from [[literature/papers/agarwal2026fire]]'s sham refusal,
   which was worth about zero.
3. **Instructions and reminders change checking more than outcomes.** In
   baseline TB runs, only 6.2–13.6% of editing turns are followed by a
   recognized check. After an after-edit user reminder, a check follows
   within two turns 58–72% of the time. Average accuracy moves only
   −1.1 to +5.4 pp, and the sign often flips by model and benchmark.

No interval or significance test appears anywhere. The critic's "who
checks" is confounded with a different model, a fresh context and a long
falsification protocol. No arm separates them.

## Claims

- Verification is "an axis of test-time scaling" in a harness. The design
  questions are when to intervene, who checks, and how findings lead to
  revision.
- L0 and L1 give "modest, model-dependent" gains without raising average
  output tokens. Recommendation: a short verification instruction in the
  system prompt, plus after-edit reminders.
- L2 completion gates "can yield larger gains but may substantially
  increase cost". Recommendation: "Avoid completion-time self-checking."
- L3 "delivers consistent improvement across all four designs on TB and
  SWE at modest additional cost". Recommendation: a "separate, low-cost
  critic" given the task and the agent's changes (not its trajectory).
- On VisualBench all four L3 designs beat the evaluated L0–L2 designs
  (net preference +13.2 to +23.8). Changes-based review beats
  trajectory-based review with and without required visual inspection.
- More review rounds (depth) or more critics (width) add up to 3.5 pp on
  TB. Four critics come close to four rounds at 19% lower total cost,
  because feedback is merged before a single revision.

## Methods

- **Harness.** E2B cloud sandboxes orchestrated with Harbor. Each trial
  starts in a fresh sandbox. A turn with no tool call is a stop. A run
  that hit a 45-minute limit is mentioned (cancel-async-tasks). The paper
  does not say whether all TB tasks share that limit.
- **L0 text (E.1)** is one paragraph: run what you changed at its real
  surface, read the real output and exit code, render visual work, and
  "Re-reading the code and not spotting anything isn't evidence."
- **L1 (E.2).** The shared reminder body adds "if you genuinely can't run
  it here, say 'unverified'". The system-role versions carry the marker
  "[AUTOMATED SELF-VERIFY CHECKPOINT - not a message from the user]". The
  user-role version has no marker. So the role comparison also changes
  the marker.
- **L2 (E.3).** Each gate is single-pass: one message, and the next stop
  is accepted. "Automated" designs are user-role messages named
  `system-reminder` with a "[… - not a message from the user]" tag. The
  rejection text is "An automated verification pass has determined this
  task is NOT complete. Keep working; do not finish yet." No verification
  pass exists. Early variants also fire after 80% of the task's time
  limit.
- **L3 (E.4).** The critic is Qwen3.7-Plus for every agent. It gets the
  task, the git diff and, in the trajectory arms, the agent's full
  session. It can use a shell in the live sandbox but must "NEVER MUTATE
  THE DELIVERABLE" and must not "hunt for hidden grading tests". The
  harness also denies it write permission. The protocol has three steps:
  a weighted requirement checklist, verify each item by running the
  deliverable, then a strict verdict with "PROXY" and "SHARED MISTAKE"
  traps. Output is JSON with `done`, `feedback` and a per-item `met` plus
  `evidence`. The verdict is the model's. An objection goes to the agent
  as a user message ("A strict verification pass determined the task is
  NOT yet complete. Unmet requirements / defects: {detail}").
  "Task-first" checks against the spec alone. "Claim-guided" treats the
  diff or session as a map of claims to falsify.
- **Depth/width (E.5).** These use claim-guided review with changes on TB.
  Depth n allows up to n critic rounds within the remaining budget. Width
  n runs n critics one after another in the shared sandbox, each blind to
  the others. Any objection sends the work back once.
- **Tasks (A.1).** For TB, 31 tasks were chosen from the 89-task registry
  using a baseline pilot on four of the seven models, balanced across
  "usually solved / inconsistent / usually failed". For SWE, 30 tasks were
  spread over three time-to-fix tiers, then "adjusted" after baseline
  evaluation.
- **Cost (A.2).** Dollars at Alibaba Model Studio rates, with uncached,
  cached and output prices per model. The critic's output price is $1.194/M
  against $5.373/M for the Qwen Max agents. It is $0.299/M for DeepSeek
  V4-Flash, so for that agent the critic costs 4× more per output token.
  Figure 2 reports "mean cost relative to its baseline". Output tokens are
  reported separately (B.3).
- **Baseline.** One no-verification baseline per model (3 trials per task)
  is shared by every design, so baseline noise moves all 16 deltas
  together.
- **VisualBench (C).** 18 tasks, 2 trials per task × model × condition. One
  representative design per L0–L2 plus four L3 designs. Raters are "members
  of an internal company data team" and choose win / tie / loss / both
  fail. Ethics approval is stated. **Not reported:** the number of raters,
  ratings per pair, and inter-rater agreement.
- **Not reported anywhere:** confidence intervals, significance tests,
  seeds, the critic's objection rate, its false-objection rate,
  pass→fail vs fail→pass flips, finish-request precision, and code.

## Results

- **Effective sample size.** Per-model TB accuracies are multiples of 1/84
  (1/75 for Qwen3.8-Max). SWE accuracies are multiples of 1/87. So the
  scored sets look like 28 TB tasks (25 for Qwen3.8-Max) and 29 SWE tasks
  × 3 trials, not the stated 31 / 30. A few values are off-grid (e.g. 53.0),
  which suggests some missing trials. The paper does not explain this.
  One task flipping in all three trials moves a per-model cell by
  3.6 pp (TB) or 3.4 pp (SWE).
- **Average deltas (Tables 2–3, Fig. 2), pp over baseline (TB 55.1%, SWE
  70.9%), with cost × baseline:**

  | Design | TB | SWE | Cost TB / SWE |
  |---|---|---|---|
  | L0 system / user | +3.3 / +1.2 | +3.9 / −1.1 | 1.08, 1.03 / 0.94, 0.93 |
  | L1 periodic / after-edit sys / after-edit user | +3.8 / +3.5 / +5.4 | +3.1 / +3.6 / −0.3 | 0.90–1.12 / 0.85–0.89 |
  | L2 inspect changes (auto) | +0.7 | +0.5 | — |
  | L2 run-and-check auto / system | +2.6 / +4.5 | −0.5 / +1.5 | — |
  | L2 reject auto / user / user-early | +4.0 / +2.1 / +6.0 | +5.3 / +4.1 / +6.2 | 1.60–1.76 / 1.48–1.56 |
  | L2 recheck user / user-early | +4.1 / +4.8 | −0.3 / +4.1 | early: 1.13 / 1.20 |
  | L3 (4 designs) | +5.9 to +9.5 | +3.9 to +6.6 | 1.13–1.25 / 1.00–1.04 |

  Per-design cost points for L2 non-rejection designs are only in the
  figure, so we do not quote them.
- **Per-model consistency of L3.** On TB, 24 of 28 model × design cells
  are positive, 3 are 0.0 and 1 is negative (Kimi, task-first changes,
  −1.2). On SWE, 21 of 28 are positive, 3 are 0.0 and 4 are negative
  (down to −3.4). Kimi-K2.7-Code supplies +13.8 to +20.7 on SWE. Without
  Kimi, the SWE L3 averages are +1.9 to +4.8, and early user rejection is
  +6.7 (our arithmetic from Table 3). So **the SWE claim that L3 matches
  the best L2 at lower cost rests on one model**. On TB the order holds
  without Kimi (L3 up to +9.9; early rejection +7.1).
- **L2 per-model.** Early user rejection is positive for all 7 models on
  SWE (+1.1 to +11.5). That makes it more consistent there than any L3
  design. On TB it is 5 positive and 2 at 0.0.
- **Design-14 baseline anomaly.** Claim-guided changes review has a
  different TB baseline: 56.8 average, and DeepSeek V4-Pro 63.1 against
  51.2 for every other design. Its DeepSeek delta is 0.0 against that
  baseline. Against the common baseline it would be +11.9, and the design
  average +7.6 (62.7 − 55.1) instead of +5.9. Output-token baselines
  differ too (49.2k vs 47.7k). This is not explained. The depth/width sweep
  starts from this design (62.7% at one round).
- **Tokens vs dollars.** Average L3 total output tokens (agent + critic)
  are 45.5–52.8k against 47.7k on TB (0.95–1.11×). On SWE they are
  40.0–43.3k against 37.8k (1.06–1.15×). Rejection gates use 67.0–69.6k on
  TB (about 1.4×). Trajectory-context critics cost 29–43% more critic
  dollars than changes-context critics.
- **Baseline token noise.** DeepSeek V4-Flash's TB baseline is 96.7k output
  tokens. Fourteen of its 17 designs come in below that (46.5–94.7k).
  Because the baseline is shared, that one baseline value drives part of
  the "L1 cuts output tokens 14–25%" result. That is our reading; the paper
  does not discuss it.
- **Behaviour (Tables 7–9, TB).** Baseline: 6.2–13.6% of editing turns are
  followed by a recognized check before the next edit. After-edit user
  reminders reach 80–100% of runs. A check follows within two turns in
  58–72% of reminder events, and any execution in 90–97%. The two
  denominators differ (editing turns vs reminder events). After a user
  rejection, agents take a median 12–19 more steps and change files in
  86–96% of triggered runs. After "Inspect changes" the figures are 4–6
  steps and 38–66%.
- **Depth/width (Fig. 4, B.4).** These are TB averages over seven models
  with no intervals:

  | Setting | Accuracy | Cost per trial |
  |---|---|---|
  | 1 critic, 1 round | 62.7% | $0.47 |
  | 2 rounds | 65.6% | $0.61 |
  | 2 critics | 64.4% | $0.60 |
  | 4 rounds | 66.1% | $0.77 |
  | 4 critics | 65.8% | $0.62 |

  Critic cost is about $0.14 per trial for both 4-settings. The gap comes
  from agent re-execution. A second round helps Qwen3.7-Max (59.5 → 70.2)
  and hurts Kimi (60.7 → 58.3).
- **VisualBench (Table 12).** Net preference over baseline: L0 −3.2, L1
  −3.2, L2 early recheck +2.7. L3: changes +21.2, changes + visual +23.8,
  trajectory +13.2, trajectory + visual +21.6. Qwen3.8-Max alone scores
  +71.1 to +92.0 under L3. Without it, the L3 means are about +3.6 to
  +12.7 (our arithmetic). So the "+13.2–23.8" range is mostly one model.
  Changes beats trajectory in every model-excluded comparison too.
- **Cases (D).** The critic finds an executable counterexample the agent
  missed: a date before the IP in regex-log, and COBOL unsigned underflow.
  Reopened work can also break a passing artifact. In git-multibranch, a
  test commit was left on served branches after an early recheck, and an
  L0-instructed run deleted the repo to retest. One critic diagnosed a
  real defect, but the run hit its 45-minute limit before the repair.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest summary is in the paper and quoted correctly. The digest says
  "seventeen designs over four layers, seven frontier coding models". That
  is right, but it leaves out what bounds the result:
  1. **"An independent critic" is a different, cheaper model with a long
     protocol.** Every agent is checked by Qwen3.7-Plus. No arm uses the
     agent's own model as critic. No arm runs the critic's protocol as an
     L2 self-check. No arm gives a fresh context with the same model. So
     "who checks" (independence) is not separated from which model checks,
     the fresh context, or the falsification protocol (roughly 700–900 words
     once assembled, by our count).
     [[literature/papers/zhang2026veriharness]] found the protocol, not
     environment access, carried most of its verifier's gain. The same
     could hold here.
  2. **"Cost" is dollars at a 4.5× price gap.** The 1.00–1.04× SWE figure
     depends on the critic's output costing 22% of a Qwen Max agent's.
     Measured in output tokens, L3 is 0.95–1.15×. A same-model critic would
     cost more. Neither the digest nor the paper prices one.
  3. **"Best trade" holds clearly on TB only.** On SWE the L3-vs-L2 claim
     depends on Kimi. Without Kimi, early user rejection beats every L3
     design on accuracy.
  4. **No intervals, 3 trials, about 28–29 tasks per model.** A naive
     binomial SE for a pooled seven-model difference is about 3 pp. For a
     single model it is about 7–8 pp (our estimate; task clustering and
     pairing pull in opposite directions). So most L0/L1 averages
     (+1.2 to +5.4) are within about 2 SE. Most per-model "model-dependent"
     swings are within the noise of 84 trials. L3's +9.5 on TB is about
     3 SE.
  5. **Open-weight agents with low baselines.** Baselines are 55% (TB) and
     71% (SWE). There is no Claude or GPT agent. This matters for the
     content-free-rejection result in particular (below).
  6. **Relevance stays 4.**
- **The content-free rejection is the paper's most useful accident.** The
  L2 "Reject completion" message is a sham refusal: it fires on every
  first stop and names nothing. Yet on these agents it gave +2.1 to +6.0 TB
  and +4.1 to +6.2 SWE. Compare [[literature/papers/agarwal2026fire]]
  (Codex, Terminal-Bench 2.1, 14 tasks × 2): there the triggered sham sat
  3.6 pp below baseline and the named-evidence refusal carried +25 pp.
  Two differences may explain it. Here the sham says "keep working" and
  the agents then take 12–19 more steps, so it acts as a budget extension
  on agents that stop early. Agarwal's sham asked for a generic review.
  Also, these agents' unforced stops are probably less often correct than
  Codex's. That is our inference; neither paper measures finish-request
  precision. Read with [[literature/papers/vats2026good]], where generic
  verify prompts were pure cost on an agent whose finish requests were
  87% right. The value of a generic refusal looks like a function of the
  agent's premature-stop rate. It is not a constant.
- **L3 vs L2-reject is the nearest real-vs-sham contrast here.** Both
  refuse once and both force more work. L3's refusal is conditioned on a
  verdict and carries named unmet requirements. Its margin over the best
  sham (early user rejection) is +3.5 pp on TB and +0.4 on SWE, at about
  0.7× the cost. That is our arithmetic from Tables 2–3 and Fig. 2. Model,
  context and timing also differ between the two. So on these agents,
  naming the defect bought cost-efficiency more than accuracy. One likely
  reason is that a critic that approves does not reopen work, while the
  sham always does. The critic's approval rate is not reported, so this
  cannot be checked.
- **Nothing on false rejection.** The critic is told "Be strict, no benefit
  of the doubt". We have no count of how often it objected to work that
  would have passed, or of how often revision broke a passing artifact.
  The case studies show reopening can break work (git-multibranch), so
  pass→fail flips exist. Only net deltas are reported.
- **Changes vs trajectory is weak evidence on "withhold the account".**
  [[literature/papers/zhu2026who]] and [[literature/papers/xu2026dont]]
  found a judge's false accepts rise when it sees the agent's own
  account. Here the trajectory critic saw the agent's session but was told
  to treat every claim as a hypothesis to falsify. On TB, trajectory arms
  did slightly better (+8.4 / +9.5 vs +7.6 corrected / +6.9). On SWE the
  arms were mixed. On VisualBench changes won (+21.2 vs +13.2), and
  trajectory cost 29–43% more. None of these TB/SWE differences clears the
  noise above. Read it as a null that does not overturn the
  withhold-the-account rule. The changes-only arm also sees the diff and
  any comments, so it is not account-free.
- **Message role is not a stable lever.** System beats user for L0 on both
  benchmarks (+3.3 vs +1.2; +3.9 vs −1.1). For L1 after-edit, user beats
  system on TB (+5.4 vs +3.5) and loses on SWE (−0.3 vs +3.6). The role
  arms also differ in the "not a message from the user" marker. The
  paper's "keep verification in the system prompt" rests on the L0 row
  alone.
- **The "automated" L2 format imitates Claude Code.** It is a user-role
  message named `system-reminder`, which is the shape Claude Code uses to
  inject harness text. The paper cites Claude Code hooks as the reference
  completion-check mechanism. Whether Claude agents treat that channel the
  same way is untested here.
- **Task selection used the agents under test.** Four of the seven models
  picked the TB tasks, and SWE was "adjusted" after baseline evaluation.
  That is reasonable for headroom, but it ties the subsets to these
  models. The critic prompt's development set is not described, so we
  cannot tell whether the tasks were held out from prompt tuning.
- **Stake.** Alibaba supplies the platform, the pricing, the critic model
  (Qwen3.7-Plus) and two of the seven agents. The headline recommends a
  cheap critic, and the cheap critic is theirs. The design does not depend
  on that, but no other critic model is tried.

## Trust signals

- **Credibility:** 3.
  - For: a reputable group (Cornell + Alibaba). A broad factorial over 17
    designs × 7 agents × 2 benchmarks, with per-model tables for every
    cell. Every intervention and critic prompt is reproduced verbatim
    (Appendix E). Cost is in billed dollars with caching, and output tokens
    are separated. Behavioural diagnostics (Tables 7–9) show what each
    intervention changed. The VisualBench protocol is blinded and
    side-randomized, with ethics approval.
  - Against: not peer reviewed, 9 days old, no citations, no code. No
    interval or test anywhere, with 3 trials on about 28–29 tasks per
    model. One shared baseline per model, plus an unexplained alternate
    baseline for design 14. The effective task counts do not match the
    stated ones. The critic is confounded with model, context and
    protocol. Rater count and agreement are not reported. The sponsor
    supplies the critic.
  - Not 4 because nothing is released and no uncertainty is reported for
    a result that lives in 3–6 pp differences. Not 2 because the design
    space is broad and fully specified, and the per-model tables let a
    reader recompute what the averages hide.

## Follow-up

- **Relevance:** 4. This is the first source in this graph to price four
  verification placements side by side, in billed dollars, across seven
  agents. It also contains an unplanned content-free refusal arm at scale.
  It adds material evidence to:
  - [[concepts/evidence-gated-completion]]: a refusal conditioned on a
    verdict versus a blind one, and the missing false-rejection count.
  - [[concepts/enforcement-boundary-placement]]: four points on the time
    axis, and reminders that move checking but not outcomes.
  - [[concepts/hybrid-model-backends]]: a cheap cross-family critic, and
    price-gap economics.
  Not 5 because no arm isolates independence, there are no intervals, and
  no agent is a model this box runs.
- **Bearing on this box.** Claude Code's Stop hook is an L2 placement. On
  this evidence, a Stop hook that blindly refuses buys accuracy only by
  buying turns (1.5–1.8× cost). A separate read-only critic run once at
  stop is the cheaper option. The open cell for us is the same-model
  critic. On a Max plan the critic would be Opus, so the dollar argument
  becomes a token argument, and the price gap that made L3 cheap is gone.
- **Cheap local test.** Take 20 TB 2.1 tasks that Claude Code with Opus
  fails sometimes. Run 3 trials each in five arms: none, content-free
  rejection, same-model fresh-context critic with this paper's E.4
  protocol, Haiku critic with the same protocol, and the E.4 protocol as a
  self-check stop hook. That is 300 runs, with no GPU. Log the critic's
  objection rate and pass→fail flips. This separates independence, model
  and protocol, which this paper does not.
- **Candidates** (cited, not yet in the graph):
  - Lopopolo 2026 (OpenAI engineering post on Codex self-review and
    iteration), cited as the practitioner baseline.
  - Kim et al. 2026, reuse of coding rollouts through structured summaries
    (cited in related work; arXiv ID not given in the main text).
  - Qiu & Gill 2026, structured disagreement in review protocols.
