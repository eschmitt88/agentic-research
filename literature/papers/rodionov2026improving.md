---
kind: paper
title: "Improving Environment-Agnostic Coding Agents: The Role of Scaffolding"
authors: ["Sergey Rodionov"]
institutions: ["SingularityNET"]
year: 2026
venue: "arXiv 2610.22289 (cs.AI), v1 2026-10-15, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.22289"
code_url: "https://github.com/astroseger/envtraverse"
citations: null
source: "raw/papers/rodionov2026improving.pdf"
added: "2026-10-19"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/structured-world-model]]"
tags: ["envtraverse", "environment-agnostic-agent", "codex", "scaffolding", "return-of-control", "stop-and-report", "scripted-controller", "reset-policy", "controller-vs-prompt", "decision-relocation", "stuck-protocol", "recovery-supervisor", "arc-agi-3", "agi-maze", "terminal-bench", "curriculum", "tutorial-transfer", "inference-not-matched", "n-equals-3"]
---

# Improving Environment-Agnostic Coding Agents: The Role of Scaffolding

## TL;DR

One Codex agent (a thin scripted controller, an environment-agnostic client
and a Linux workspace) is deployed unchanged on ARC-AGI-3 (25 games), AGI Maze
(106 labyrinths) and Terminal-Bench 2.1 (89 tasks). Four model–effort
configurations are run: GPT-5.5 high/xhigh and GPT-5.6-sol xhigh/max. Each
condition gets 3 evaluation blocks.

- **One prompt line.** "Continue until you have completed your task" becomes
  "stop and report" when unsure. This raises mean ARC-AGI-3 RHAE by 1.4–7.3
  points and AGI Maze RHAE70 by 0.4–5.0. Terminal-Bench moves −1.5 to +2.25.
  Cost rises with it.
- **A scaffold gap remains.** An ARC-specific reference agent (`twma_v1.61`)
  still beats the general agent by 6.9–18.8 RHAE.
- **A scripted reset closes part of it.** The variant Controlled_RESET adds a
  prompt ban on calling RESET; the controller issues RESET at `GAME_OVER`
  instead. It recovers 34–60% of that gap.
- **No arm is matched on inference.** The author says so.

## Claims

- **An environment-independent change can help on several environments.**
  Stop-and-report beats Baseline on mean score in 4/4 ARC-AGI-3
  configurations and 8/8 AGI Maze comparisons. AGI Maze was not used to select
  the instruction. Terminal-Bench shows "small, mixed changes" that "do not
  establish" non-inferiority.
- **Supplied procedures still matter at the frontier.** `twma_v1.61` exceeds
  Stop-and-report by about 7 RHAE even at GPT-5.6-sol max. The author argues
  its advantage is "organization and control rather than supplying new game
  rules". The paper calls this "a qualitative comparison of the supplied
  game information, not a claim of identical prompts or control logic".
- **Implementing a stated convention in the controller helps.** The reset
  rule is already in the environment description given to every agent.
  Controlled_RESET "recovers roughly one-third to three-fifths of the gap",
  so "the intervention supplies control around available information rather
  than a new game rule."
- **More experience is not uniformly better.** Seven preceding tutorial
  labyrinths lower target RHAE70 for both GPT-5.5 variants at both efforts.
  They help both variants at GPT-5.6-sol xhigh, and are mixed at max.
- **The mechanism is not identified.** Gains may come from better use of
  information, more inference, or differently allocated inference: "the
  experiments do not separate them."

## Methods

- **Agent.** `agent_v1` is a simplified version of the author's earlier
  ARC-AGI-3 textual agent, with ARC-specific control and the world-model
  requirement removed. The main prompt points Codex at
  `environment_description.md` and a client (`client.py move '<json>'`). It
  allows any workspace use, including indexes and copied observations.
  A single Codex invocation can include many model calls and environment
  actions.
- **Controller (Fig. 3).** Each time Codex returns, the script reads only
  the terminal flag and the step index.
  - **Normal protocol:** resume the same conversation with "keep working
    toward the goal … re-read the file" plus the variant string.
  - **Stuck protocol:** if a continuation produced no new action, append a
    no-action reminder ("If you do not make progress, the run will stop").
    If Codex again returns with no action, the controller exits.
  - **Recovery supervisor:** restarts the controller after crashes, freezes
    *or* stuck-protocol exits, up to 10 times per run. It resumes the saved
    conversation. So the "halt" after the second no-progress return is in
    practice followed by up to 10 restarts. No arm ablates this.
- **The single difference between the two variants** is
  `continuation_string.txt`. It is sent at initialization and on every
  continuation. "Report it back to me" means the automated controller, which
  answers with the generic continuation and no task advice.
- **Controlled_RESET (§6.2).** This is Stop-and-report plus one prompt
  paragraph: "If the game is in the GAME_OVER state, you should report it,
  and I will give you another attempt on the same level. You are forbidden
  to call RESET by yourself."
  - The action interface is unchanged, so RESET is not masked. The ban is
    prompt-only. The controller issues RESET on a `GAME_OVER` return, then
    resumes with a reset notice.
  - Compliance with the ban is not reported.
- **`twma_v1.61`.** The textual agent from the author's 2607.15439 study,
  rerun in this campaign. It differs in several bundled ways: organization
  by level and attempt, a controller-managed RESET policy, a required
  persistent textual world model, and a more structured prompt sequence.
- **Environments.**
  - ARC-AGI-3: 25 public games. A level ends at WIN or after 1,500 actions,
    with RESETs counting toward the 1,500.
  - AGI Maze: identifiers 0005 and 0009 from CLASSIC and EXTENDED, 106
    targets. The action cap is 5× `max_steps`. The curriculum setting runs
    7 tutorials before each target.
  - Terminal-Bench 2.1: 89 tasks, `service_tier="fast"`.
  - The agent runs in Docker with an allowlisted proxy and no web search.
    Task IDs are replaced by random aliases.
- **Design.** 3 retained blocks per condition. Repetition indices are "not
  matched random seeds". The author states that "contrasts are descriptive
  differences between means", and no statistical tests are run.
  - All-or-nothing replacement: any block with an irrecoverable failure is
    rerun whole. One Maze batch, three Terminal-Bench blocks and one
    `twma_v1.61` block were replaced.
  - Cost is in normalized tokens: cached input /60, uncached input /6,
    output ×1.
  - Runs took place 21 Sep–10 Oct 2026 on eight subscription Codex accounts.
    Exploratory runs are excluded.
- **Metrics.** ARC-AGI-3 uses benchmark RHAE. AGI Maze uses RHAE70: 0 if
  unsolved, else 100·min(1, (0.7·m/a)²). RHAE70 was "adopted during analysis
  and manuscript preparation, after examining mean step fractions".

## Results

- **ARC-AGI-3 (Table 1), mean RHAE / cost (M):**

  | config | Baseline | Stop-and-report | Controlled_RESET | twma_v1.61 | CR share of S&R→twma gap |
  |---|---|---|---|---|---|
  | GPT-5.5 high | 27.57 / 9.36 | 34.86 / 14.19 | 45.92 / 42.19 | 53.19 / 64.09 | 60% |
  | GPT-5.5 xhigh | 50.45 / 16.08 | 52.36 / 20.18 | 61.44 / 45.52 | 71.11 / 63.64 | 48% |
  | GPT-5.6-sol xhigh | 70.81 / 16.30 | 77.33 / 16.57 | 84.59 / 18.36 | 92.24 / 27.03 | 49% |
  | GPT-5.6-sol max | 89.59 / 15.34 | 90.96 / 18.01 | 93.31 / 18.86 | 97.86 / 26.61 | 34% |

  The gap-share column is our arithmetic and matches the paper's "34–60%".
  Cost ordering matches score ordering in every configuration.
- **Controlled_RESET's cost relative to Stop-and-report** (our arithmetic):
  2.97× and 2.26× on GPT-5.5, but only 1.11× and 1.05× on GPT-5.6-sol. So
  at GPT-5.6-sol xhigh, +7.26 RHAE came for about +11% cost. That is the
  closest the paper gets to a matched-compute reset result, and it is not a
  designed match. On GPT-5.5 the reset arm spends roughly 3× more, and the
  gain cannot be separated from that spend.
- **Run spread (Table 4, our reading).**
  - Stop-and-report vs Baseline separates cleanly only at GPT-5.5 high
    (33.3–37.1 vs 22.9–30.3). At GPT-5.5 xhigh one S&R run (58.48) carries
    the +1.9 mean, and the other two sit at 49.5 and 49.1. At GPT-5.6-sol max
    the runs interleave (B 87.7/92.2/88.9 vs S&R 92.4/87.3/93.2).
  - Controlled_RESET vs Stop-and-report separates at GPT-5.5 high, and
    mostly at GPT-5.5 xhigh and GPT-5.6-sol xhigh (8 of 9 cross-pairs favour
    CR). At GPT-5.6-sol max it does not: CR 92.1/94.8/93.0 vs S&R
    92.4/87.3/93.2, a mean gain driven by one low S&R run.
  - Unsolved games at GPT-5.5 high: S&R 17/17/18, CR 14/14/14.
- **AGI Maze (Table 2).** Stop-and-report − Baseline RHAE70: single +0.68,
  +3.37, +2.91, +2.52; curriculum +2.49, +0.37, +3.07, +4.95. Cost is up
  7–39%. Normalization changes the sign of one cell: RHAE80 favours Baseline
  in GPT-5.5 xhigh curriculum (67.14 vs 66.54).
- **Tutorials (curriculum − single, our arithmetic).**
  - GPT-5.5 high: B −4.17, S&R −2.36.
  - GPT-5.5 xhigh: B −4.97, S&R −7.97.
  - GPT-5.6-sol xhigh: B +2.27, S&R +2.43.
  - GPT-5.6-sol max: B −1.32, S&R +1.11.
  - The curriculum costs 1.5–3.6× more tokens, because tutorials count.
- **Terminal-Bench (Table 3).** Rewards are 79–87%. S&R − B: +1.50, −0.37,
  +2.25, −1.50. Over the 24 blocks, 22 executions hit the time limit, all
  with reward 0. Twelve GPT-5.6-sol executions ended on a cybersecurity-flag
  recovery failure after using all 10 recovery attempts.

## Critique / open questions

- **Against the digest framing: BELOW, mildly.** The numbers match the
  digest: a 34–60% recovery, unmatched inference, and tutorials lowering Maze
  scores on GPT-5.5. The abstract does not overstate. What reads above the
  paper is the digest's own label, "fifth instance of 'controller, not
  prompt'":
  - Controlled_RESET adds a prompt prohibition *and* a scripted action.
  - There is no arm that restates the reset convention more forcefully in
    the prompt without the controller, and compliance with the ban is not
    measured.
  - The paper never frames the result as controller vs prompt.
  - The Stop-and-report result is itself a prompt line that works on means,
    which cuts the other way.
  - Two of four ARC Stop-and-report gains, and the GPT-5.6-sol max reset
    gain, are within run spread.
- **The reset comparison is not "controller vs prompt"; it is "agent decides
  vs script decides".** Both arms are told the convention: reset only at
  `GAME_OVER`. What moves is who executes the reset and when. A cheaper
  sanctioned alternative is also added: report, and you get another attempt.
  That resembles gomez2026can's escalation tool more than zhang2026judged's
  action mask. The paper reports neither how often the general agent
  misused RESET nor how often it stalled at `GAME_OVER`, so the failure the
  controller fixes is inferred, not observed.
- **Matched compute.** No arm is matched, by design: "not at matched
  inference cost". On GPT-5.5 the reset arm's 2.3–3× cost dominates any
  reading. On GPT-5.6-sol the cost is within 5–11%, and there the reset gain
  is +7.3 (xhigh, mostly separated) and +2.4 (max, not separated). The
  GPT-5.6-sol xhigh cell is the one usable data point.
- **The sample is small and selection is in-sample on ARC.** Each mean is
  over 3 blocks. Stop-and-report "was developed through ARC-AGI-3
  experimentation", and exploratory runs are excluded from the report. So
  the ARC gains carry a selection effect, and AGI Maze is the only clean
  out-of-sample test: +0.4 to +5.0 RHAE70, under a metric chosen post hoc.
  Maze is per-target-heavy (106 targets × 3), but the per-block spread
  (Table 5) is several points.
- **The stuck protocol is a reminder-then-halt rule, and it is
  uninstrumented.** On the first no-progress return the controller sends a
  prompt reminder. On the second it exits, and the supervisor restarts it up
  to 10 times. This is the shape `/iterate` would have with a prompt-only
  redirect at ceiling − 1. The paper does not report how often it fired, or
  whether restarts after it ever produced progress.
- **The `twma_v1.61` gap is bundled.** Level/attempt organization, the reset
  policy, a mandatory world model and the prompt structure all differ. Only
  the reset policy is isolated. "Organization/control, not game knowledge" is
  the author's qualitative judgment.
- **Tutorial harm is GPT-5.5-only and confounded with context load.**
  Tutorials run in the same conversation and workspace, cost up to 3.6×, and
  the effect flips sign by model. The paper gives no mechanism, such as
  context crowding or negative transfer.
- **Open for this graph:** no arm compares a controller-owned restart with a
  controller-owned stop. No arm masks RESET at the interface, which would
  separate the ban from the scripted action. Neither is run here.

## Trust signals

- **Credibility:** 3.
  - For: code, documentation and "complete results and run artifacts" are
    claimed at the EnvTraverse repository (not verified here). Every
    individual run is tabulated (Tables 4–7). All-or-nothing block
    replacement is applied, with each replacement disclosed. The post-hoc
    metric choice and its one sign flip are disclosed. Prompts are
    reproduced verbatim. Isolation controls are described. The author is
    unusually explicit about what is not shown.
  - Against: single author at SingularityNET. Not peer reviewed, although
    the author's earlier ARC-AGI-3 paper appeared at AGI 2026 (LNCS). Three
    blocks per condition, unmatched seeds and no statistical tests. The
    headline instruction was selected on one of the three benchmarks, using
    unreported exploratory runs. Inference is unmatched.

## Follow-up

- **Relevance:** 3. It is useful prior art for redirect-before-halt and
  for [[concepts/enforcement-boundary-placement]]: a controller that owns a
  *restart* decision the agent was already told how to make, with cost
  reported.
  - It does not compare redirect with halt, and it does not cleanly
    separate controller from prompt.
  - The one near-cost-neutral cell (GPT-5.6-sol xhigh, +7.3 RHAE for +11%
    tokens) is the strongest datum.
  - Its stuck protocol is an unmeasured instance of the reminder-then-stop
    pattern that the pending proposal replaces.
- **For `/iterate`:** the trigger belongs to the controller, and the agent
  should get a sanctioned action at the trigger ("report and you get another
  attempt"), not only a ban. A prompt line that makes the agent *return
  control* at difficulty helped modestly. That suggests the counter's
  ceiling − 1 point is worth surfacing as a return-to-controller event even
  before any content constraint is enforced (our inference).
- **Candidates** (cited, none in the graph):
  - Rodionov 2026, "Do coding agents need executable world models,
    simplification, and verification to solve ARC-AGI-3?" (arXiv
    2607.15439). Source of `twma` and the runtime-recovery design.
  - Rodionov 2026, "Executable world models for ARC-AGI-3 in the era of
    coding agents" (AGI 2026, LNCS 16855).
  - Bandel et al. 2026, "General agent evaluation" (Exgentic, arXiv
    2602.22953). Five architectures × five backbones × six benchmarks;
    reports premature termination as a failure pattern.
  - Qin et al. 2026, "StateM" (arXiv 2608.15089). Workflow states and
    checked transitions on Terminal-Bench 2.1.
  - Lehmann et al. 2026, "Tycho" (arXiv 2607.28287). When a coding agent
    should build, repair, use or bypass a programmatic model.
