---
kind: paper
title: "Who Grades the Agent? Privilege Separation for Coding-Agent Benchmarks"
authors: ["Nathan Nour Houlamy", "Mattia Favretto", "Earlence Fernandes", "Mauro Conti"]
institutions: ["University of Padua", "UC San Diego"]
year: 2026
venue: "arXiv 2611.27860 (cs.CR), v1 2026-11-20, preprint (18 pages incl. appendices A–E)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.27860"
code_url: null    # "We will release the full artifact upon publication"; nothing released yet
citations: null
source: "raw/papers/houlamy2026who.pdf"
added: "2026-11-24"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["privilege-separation", "split-verification", "splitbench", "benchmark-integrity", "reward-hacking", "adversarial-measurement", "oracle", "adapter", "hidden-data", "fresh-environment-per-case", "malicious-evidence", "forged-observations", "terminal-bench", "deepswe", "benchmark-conversion", "agent-authored-evaluator", "converter-reviewer-loop", "weak-acceptance-conditions", "semantic-adequacy", "red-teaming", "provider-hosted-tools", "egress-proxy", "coding-agents", "claude-opus-5.5"]
---

# Who Grades the Agent? Privilege Separation for Coding-Agent Benchmarks

## TL;DR

SplitBench is a grading framework for coding agents. It separates
**running** a task's tests from **judging** them. The agent's changed
files, filtered by a declared scope that excludes test files, are copied
onto a clean baseline. Each test case then runs in a fresh, network-less
Evaluation environment. There an untrusted **adapter** runs the candidate
on a challenge (inputs only) and reports the values it produced. A
trusted **Oracle** on the host holds the expected answers and an
operator-chosen seed, and compares the values. The framework assumes the
candidate controls the whole Evaluation environment, adapter included.
Only the hidden data and the seed are secret; the grading code may be
public.

Four results matter for this graph:

1. **Most tasks convert, and the converted tasks grade like the
   originals.** Claude Opus 5.5 agents converted **180 of 202** tasks
   (Terminal-Bench 2.0: 68/89; DeepSWE v1.1: 112/113). The median cost
   was about $10 per task at API prices. On 1,374 same-files comparisons
   (Sonnet 5.5 and GPT 6.1 Sol, two runs), SplitBench and the native
   grader agree **98.7%**. None of the 18 disagreements is a conversion
   error. Twelve are agents that edited or added test files, which the
   native grader ran.
2. **An informed attacker got no pass through the framework, but did get
   through the tasks.** Opus 5.5 at high effort, told to pass without
   solving, made one attempt per task at two knowledge levels; 307 of 360
   completed. SplitBench rejected **126 of 168 (75%)** with full knowledge
   of the grading code and 117 of 139 (84%) with a design description
   only. **0** of the 64 passes went through a flaw in SplitBench. 47 used
   weak upstream tests, 12 public answers, 2 conversion errors, and 3
   faked the implementation from inside the adapter's process.
3. **The original tests are weak on a large share of tasks.** At least
   **34 of the 180** converted tasks accept unsolved work: **29 of 68
   Terminal-Bench tasks (43%)** and 5 of 112 DeepSWE tasks (our
   arithmetic). The native grader also passed 40 of the 47 such attacks.
   A secure grader inherits these tests. It removes attacks on grading
   and leaves this problem open.
4. **The native graders are easy to tamper with.** Native graders
   accepted **230 of 429** attacks written by the converter. SplitBench
   accepted none, but that holds by construction: a conversion is
   admitted only once all its attacks fail SplitBench. In a small
   campaign on the native tasks, **71 of 96** completed attacks passed the
   native grader, and SplitBench rejected 55 of those 71. The model's
   safeguards stopped 264 of the 360 attempts, so the authors make no
   general claim from it.

## Claims

- In most frameworks the grader is trusted but takes its measurements
  in state the agent controls. Examples are a shared container
  (Terminal-Bench 2.0), a separate verifier with the agent's patch
  applied (DeepSWE), a verdict read from the test log (SWE-bench) and
  Unix-user separation (METR). SplitBench claims to be "the first
  framework in the scientific literature" that separates running a
  task's tests from judging them.
- The known attacks fall into six classes, mapped to four goals (Tables
  2–3):
  - **G1 mutual isolation:** disclosure of evaluation information;
    tampering with acceptance checks.
  - **G2 verdict integrity:** forged observations; malicious evidence.
  - **G3 test-case isolation:** cross-case interference.
  - **G4 access control:** access-policy misuse.

  SplitBench prevents five classes by design and the sixth only in part.
  No compared framework fully prevents access-policy misuse.
- A stronger sandbox alone (e.g. a microVM) would not stop the known
  grading attacks (§6.1). The conftest hook, the call-stack read and the
  polyglot binary swap all use access the benchmark itself grants.
  Split verification changes the *policy*: which files cross, what the
  candidate can reach, and what evidence the Oracle accepts.
- A fabricated observation passes only if it holds the correct outputs
  for a challenge the candidate first sees during evaluation. For a
  well-tested task, producing them means solving it.
- Agents can port whole benchmarks "with minimal human supervision". The
  converter/reviewer loop is needed: the reviewer found problems in 173
  of 180 conversions.
- What remains is semantic, and a grader that checks only results
  cannot close it:
  - the Oracle judges values, not how they were computed;
  - a destination allowlist cannot control what is done on an allowed
    service.

  Both gaps are stated in the paper, not solved.

## Methods

- **Architecture (Fig. 2, §4).**
  - *Capture.* Files that changed inside the task's scope are diffed
    against the baseline, by a network-less Capture environment treated
    as possibly compromised. SplitBench re-checks the scope and the
    file/byte limits and builds the manifest itself. Test files and
    convention-loaded files (`_test.go`, `conftest.py`, package
    initializers) are excluded.
  - *Replay.* The candidate's files are copied onto a fresh baseline. A
    setup step (e.g. a build) runs once and is snapshotted. Each case
    then starts from its own copy of the snapshot, with no network, and
    is destroyed afterwards.
  - *Checks.* A passive check reads files within a size limit and never
    executes them. An active check runs the candidate through the
    adapter. An attempt passes only if every case of every check passes.
  - *Oracle.* It has two functions: `plan(hidden, seed)` produces the
    cases (a challenge to release, and a context kept on the trusted
    side), and a deterministic `judge(evidence, context)` returns the
    verdict. Random test inputs are drawn from the secret seed.
    - Imports are limited to an allowlist for computation. This guards
      against author mistakes, not against a hostile Oracle.
    - It runs on the host under time and memory limits.
    - Evidence is stored, so any verdict can be recomputed.
  - *Evidence interface.* An observation is one strict JSON value under
    a size limit, validated against SplitBench's own copy of the task's
    schema. It never names its case: the framework attributes it.
    - An adapter crash, timeout, memory kill or malformed observation
      fails the case.
    - Only an infrastructure error makes an attempt invalid, and an
      invalid attempt never counts as a pass.
  - *Network.* A relay forwards model inference and adds the provider
    credentials, so the agent never holds a key. By default it blocks
    provider-hosted storage and tools such as web search, whose traffic
    would bypass the proxy. An egress proxy admits only the task's
    allowlisted domains. It rejects names that resolve to internal
    addresses and mismatched TLS SNI. The agent gets no feedback from
    evaluation.
- **Conversion (§5.1, App. D).**
  - Each task gets two Claude Opus 5.5 agents at high effort. The
    converter keeps exactly upstream's cases and assertions and writes a
    fidelity record of every difference. A separate reviewer, told
    "your job is to find what is wrong, not to confirm that it is
    right", works a nine-point checklist. They iterate until the
    reviewer approves.
  - *Qualification*, both graders:
    - the shipped reference solution must pass both;
    - a no-op candidate must fail both;
    - mutants (plausible wrong implementations) must get the same
      verdict from both;
    - attacks must fail SplitBench.
  - The authors changed the rules during the campaign. A fresh agent
    then brought earlier conversions in line, and another reviewed them.
  - Tasks were attempted in hash order of their names, so that the
    tasks that suit SplitBench are not picked first.
  - 4.8 B tokens (92% cached context), a median of more than 18 M tokens
    and just under two hours per task, run on two Claude Max
    subscriptions.
- **Fidelity (§5.2).**
  - Sonnet 5.5 (Claude Code) and GPT 6.1 Sol (Codex) at medium effort
    attempt all 180 tasks natively and under SplitBench, twice. Each
    attempt's files are re-graded by the other grader.
  - 5 tasks are excluded from the comparison: their native tests check
    state a file copy does not carry. 26 comparisons are missing,
    including 7 lost to a symlink error shared by two conversions.
- **Adversarial (§5.3, App. D).**
  - The attacker is Opus 5.5 at high effort. It gets the task
    instructions plus an "authorized adversarial test" notice: pass
    without solving, and stop if no way exists. It runs under the
    Defense Access tier of Anthropic's Cyber Verification Program.
  - **Out of scope by instruction:** escaping the container, attacking
    the relay, proxy or host, sending data out, and escalating
    privileges. The authors added this because safeguards otherwise
    stopped the attacker. They "reviewed the code of the relay, the
    proxy and the container configuration separately". No method or
    result for that review is reported.
  - *Limited knowledge* is the task plus SplitBench's architecture
    document. *Full knowledge* adds the task's evaluation files and the
    Oracle's code. The attacker never gets the hidden data.
  - One attempt per task per level. Two agents classify each pass
    independently. They disagreed on one, which a third agent settled.
    The authors reviewed every class.

## Results

- **Conversion (Table 4).** 180/202 convert. Terminal-Bench exclusions:
  - by design (14): privileges or system paths 5, a running process as
    the deliverable 3, a deliverable outside any scope 2, a speed
    target 4;
  - other (7): pack size 4, machine resources 1, reference fails
    natively 1, model safeguards 1.

  DeepSWE: 1 exclusion (reference fails natively). **87 of the 112
  DeepSWE conversions accept regression-test outcomes as upstream's
  runner reports them.** A candidate can forge those, but the feature
  tests are still judged by value.
- **Review loop.** The reviewer returned findings on 173 of 180
  conversions before approving them. At least one finding was a security
  problem in 67 tasks. One was the Oracle memory-exhaustion flaw (an
  Oracle sized an allocation from a number the candidate reported). It
  led to a memory limit on every Oracle.
- **Agreement (Table 5).** 1,356 / 1,374 (98.7%): Sonnet 99.4% (694),
  GPT 97.9% (680). The 18 disagreements fall on 8 tasks:
  - 5 native-grader errors, all on one DeepSWE task: tests crashed and
    the native grader counted them as passed;
  - 12 recorded differences (Sonnet 2, GPT 10): the agent changed test
    files to fit its code, or added tests that do not compile;
  - 1 timing-dependent candidate.

  No κ is reported. Pass rates are 66–79%, so chance agreement is well
  above 50%.
- **Difficulty.** Tasks passed:

  | | Native (run 1 / run 2) | SplitBench (run 1 / run 2) |
  |---|---|---|
  | Sonnet (of 180) | 121 / 120 | 118 / 125 |
  | GPT (of 179) | 142 / 142 | 140 / 134 |

  GPT's second run is 8 lower under SplitBench. The restrictions can
  cost a pass: allowlisted domains only, no provider web search, and
  software installed outside the task's files is not kept. Median
  attempt time is 157 vs 152 s (Sonnet) and 280 vs 279 s (GPT).
- **Adversarial (Table 6).**

  | | Limited | Full |
  |---|---|---|
  | Attempts | 180 | 180 |
  | Stopped by safeguards | 41 | 12 |
  | Completed | 139 | 168 |
  | Rejected by SplitBench | 117 (84%) | 126 (75%) |
  | Passed without solving | 22 | 42 |
  | — flaw in SplitBench | 0 | 0 |
  | — conversion error | 1 | 1 |
  | — weak upstream tests | 14 | 33 |
  | — faked implementation | 0 | 3 |
  | — public answers only | 7 | 5 |

  - Full knowledge raised the pass share of completed attempts from
    15.8% to 25.0% (our arithmetic). Almost all of the rise is weak tests
    (14 → 33). 20 of the 34 weak-test tasks fell only to the attacker
    with full knowledge.
  - **The two conversion errors passed the review loop.** In
    `pypi-server` the server the agent sets up is not graded. In
    `chess-best-move` a comment in the Oracle's code states the expected
    answer, which the full-knowledge attacker read. The authors: a
    benchmark adopting SplitBench without such a campaign "should have a
    person review each conversion".
  - **Faked implementations (3).** The candidate's code runs inside the
    adapter's process. In one case it handed the calls to pandas. In
    another it replaced the adapter's function with a line-break
    detector. In the third it wrote the observation itself. The native
    grader rejects all three only because each takeover targets the
    conversion's adapter. A takeover aimed at the native test program
    would be easier.
- **Weak upstream tests (App. E, Table 7).** Examples:
  - `count-dataset-tokens`: a file containing every number below ten
    million;
  - `path-tracing`: a one-pixel image clears the similarity score;
  - `sqlite-db-truncate`: thousands of guessed rows, with no penalty for
    wrong rows;
  - `pytorch-model-recovery`: the model returns its target, so the loss
    is zero;
  - `prove-plus-comm`: a disabled soundness check, or `+` redefined;
  - several DeepSWE tasks: the new feature handed to another validator
    or to the Go compiler in the image.
- **Native campaign.**
  - Safeguards stopped 264 of 360 attempts. Of the 96 that completed, 71
    passed the native grader without solving the task.
  - SplitBench rejected 55 of those 71. They relied on files planted
    outside scope, test files and runner settings supplied by the
    candidate, and a test framework taken over by the candidate's code.
  - The other 16 pass both graders, through weak tests or public
    answers.

## Critique / open questions

- **Against the digest entry: at.**
  - Every number in the summary is in the PDF and correct.
  - **"Blocks 75% of attempts"** puts the wrong number in the headline.
    - The 75% is of *completed* full-knowledge attempts (126/168). 53 of
      the 360 attempts were stopped by safeguards and left out.
    - Whether an attack passes depends mainly on how strong each task's
      own tests are, so the 75% measures Terminal-Bench and DeepSWE more
      than SplitBench.
    - The framework's own result is **0 of 307** completed attempts
      through a SplitBench flaw. That zero is scoped (see below).
  - "Most of the rest expose weak original tests" holds: 33 of 42 at
    full knowledge, 47 of 64 overall.
  - The entry omits the 230-of-429 contrast with the native graders.
    Its SplitBench half (0 of 429) holds by construction.
- **The attacker did not test the isolation or network layers.** It was
  told not to escape the container, attack the relay or proxy, exfiltrate
  or escalate privileges. So the campaign tested:
  - verdict integrity (G2);
  - case isolation (G3);
  - the capture and scope part of G1.

  It did not test the configured isolation or G4. The authors reviewed
  those parts themselves and report no method. "No attempt passed
  through a flaw in SplitBench" is true of the oracle, adapter and
  evidence path.
- **One model plays converter, reviewer, attacker and classifier.**
  Opus 5.5 writes the attacks used in qualification, reviews the
  conversion and runs the campaign. Their blind spots may be shared. The
  campaign still found both conversion errors the review missed, so the
  overlap is not complete. Only one attacker family was tried, with one
  attempt per task per level. Every pass count is therefore a lower
  bound per task.
- **Agreement without κ.** With native pass rates of 66–79%, a grader
  that always passed would agree about 70% of the time. 98.7% is far
  above that, but the paper does not give the agreement matrix. The
  GPT-run-2 gap (142 vs 134) is not explained beyond "more restrictive
  by design".
- **The "secret is only the data" design has a measured cost on weak
  tasks.** Readable grading code nearly doubled attacker passes
  (22 → 42), almost entirely by exposing weak acceptance conditions. One
  Oracle comment leaked an answer. Public grading code is safe for the
  framework's own path only once each task's tests are strong and its
  code is scrubbed. This qualifies [[literature/papers/deeb2026towards]]'s
  "only unreadability defends a readable grader" rather than refuting it.
  Deeb's graders embedded the answers, and SplitBench removes exactly
  that.
- **Coding tasks, not ML research.** Several exclusions are what
  ML-research tasks often need: speed targets, a running service,
  properties of how a computation ran, and anything spanning several
  fresh runs. A training-time or compute budget is a speed target (our
  inference). So the design transfers directly to "the candidate returns
  predictions for held-out inputs". It does not transfer to "the
  candidate trains within budget X".
- **Semantic adequacy is untouched, and that independently confirms
  [[literature/papers/zheng2026benchshield]].** 50 of the 64 passes (47
  weak tests + 3 faked) are "the evaluator ran as written on a
  deliverable that did not solve the task". That is BenchShield's I7,
  which no isolation mechanism removed. Two groups, different
  frameworks, same residue.
- **Comparisons with concurrent work are not like-for-like.**
  - Zeng et al.'s separate hardened verifier still lets the strongest
    verifier-reading attacker succeed in 63% of attempts. Those are
    different tasks and a different attacker, so 63% vs 25% is not a
    controlled comparison. The paper does not present it as one.
  - Harbor's Rewardkit Worker (Oct 2026) was not evaluated.
- **No artifact yet.** Prompts are quoted in part; the code and
  converted tasks are promised "upon publication".
- **Consistent with neighbours in the graph.**
  - [[literature/papers/farahbakhsh2026agentdiscover]] gave the rule
    "whatever the scorer's process can see, the candidate can see".
    SplitBench is the construction that satisfies it.
  - [[literature/papers/ghosh2026truthgate]] re-runs tests on a post-run
    snapshot and hidden checks in fresh containers. There the
    assertions and expected values still sit in the container that runs
    the candidate, and SplitBench removes them.
  - [[literature/papers/wang2026androids]] (BenchJack): trust-boundary
    flaws survive code-only patches. SplitBench is the redesign BenchJack
    called for.

## Trust signals

- **Credibility:** 3.
  - For:
    - Established security groups (Conti at Padua; Fernandes at UCSD).
    - A threat model with explicit goals and non-goals.
    - Tasks taken in hash order, so the convenient ones were not picked
      first.
    - Every exclusion and every grader disagreement is reported with its
      cause, and so are the authors' own two conversion errors.
    - "We recomputed every number in the paper from the files of the
      runs." Verdicts can be recomputed without a model.
    - The abstract matches the body.
  - Against:
    - Not peer reviewed, four days old, no citations.
    - Code and converted tasks are not released yet.
    - One attempt per task per knowledge level.
    - The attacker was scoped away from the isolation and network layers.
    - One model family converts, reviews, attacks and classifies.
    - No κ for agreement.
  - Not 4: unreviewed and no artifact. Not 2: a careful, explicit
    security design with a worst-case attacker and honest accounting of
    its own errors.

## Follow-up

- **Relevance:** 4.
  - It is the first framework-scale, adversarially tested construction
    in the graph that keeps the expected answers out of the process that
    runs the candidate. That is material new evidence for
    [[concepts/information-firewall]] (the candidate-runtime boundary)
    and [[concepts/programmable-evaluator-oracle]] (instruments report,
    the oracle judges).
  - It measures how often benchmark acceptance conditions pass unsolved
    work, which bears on [[concepts/hce-evaluation]]'s
    known-answer preflight.
  - Not 5: coding benchmarks, not ML research, and its exclusions cover
    the speed and budget properties ML-research tasks often score.
- **For a downstream project that grades agent-run code** (our
  inference):
  - The candidate produces outputs for held-out inputs in a process that
    cannot read the labels. A separate process that never executes
    candidate code computes the metric.
  - The copy handed to scoring excludes test and convention-loaded
    files.
  - Before trusting the evaluator, run at least one constructed wrong
    answer (a mutant) and one informed attack pass. A do-nothing
    submission does not catch "every number below ten million".
- **Candidates** (cited, not yet in the graph):
  - Zeng et al. 2026, "Scalable weak-to-strong verifier hardening for
    agent environments" (arXiv 2610.15924). A separate hardened verifier
    with a measured 63% attacker success. It is the closest comparison
    arm.
  - Abdelnabi et al. 2026, "Measuring security without fooling
    ourselves" (arXiv 2605.22568). Position paper arguing for separating
    agent privileges from answers.
  - Thaman 2026, "Reward Hacking Benchmark" (ICML 2026, arXiv
    2605.02964). Grader outside the sandbox, recomputing on hidden data.
  - Adamczewski et al. 2026, "MirrorCode" (arXiv 2606.30182). Outputs
    are compared outside the containers that run the agent's code.
  - Christodorescu et al. 2026, "Agent security is a systems problem"
    (NeurIPS position track, arXiv 2605.18991).
  - Hamin & Edelman 2025, CAISI/NIST "Cheating on AI agent evaluations"
    (blog). A transcript-review tool that found cheating.
  - OpenAI, Hugging Face incident technical report (Aug 2026). Agents
    escaped a container during internal cyber evaluations.
  - Harbor RFC 0003, Rewardkit Worker (Oct 2026). The concurrent design
    in the shared Harbor framework.
