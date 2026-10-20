---
kind: paper
title: "ResearchFlow-Eval: Tracing the Execution Gap in End-to-End Multi-Claim Autoresearch"
authors: ["Peng Jiang", "Dongyang Li", "Quanying Liu", "Xiaoxuan Jia"]
institutions: ["Tsinghua University", "Southern University of Science and Technology", "Omni-Intelligence"]
year: 2026
venue: "arXiv 2610.18088 (cs.AI), v2 2026-10-14, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.18088"
code_url: "https://github.com/Jp-17/ResearchFlow-Eval"
citations: null
source: "raw/papers/jiang2026researchfloweval.pdf"
added: "2026-10-20"
relevance: 3
credibility: 2
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/typed-claim-partition]]"
tags: ["autoresearch", "benchmark", "execution-gap", "multi-claim", "frozen-claims", "success-criteria", "reported-vs-confirmed", "self-report", "stagewise-vs-end-to-end", "retry-trajectories", "early-exit", "skill-augmentation", "research-orchestration", "llm-judge", "anchor-paper-rubric", "integrity-gates", "aris", "orchestra", "deepscientist", "autoresearchclaw", "codex", "claude-code", "kimi-code", "falcon", "spint"]
---

# ResearchFlow-Eval: Tracing the Execution Gap in End-to-End Multi-Claim Autoresearch

## TL;DR

ResearchFlow-Eval scores an autoresearch run stage by stage against rubrics
derived from one published "anchor" paper. The run goes through seven stages:
literature review, claims, method design, experiment planning, experimental
execution, results analysis and paper writing. Before any system runs, the
task freezes ten required claims, each with an explicit success criterion,
and hides them from the system. A single LLM judge (claude-opus-5) rates 99
rubric items and issues a met/unmet verdict per claim.

The paper instantiates **one task** (SPINT, cross-session intracortical motor
decoding on FALCON M1/M2/H1). It runs **nine systems once each**, end to end,
with a 72-hour budget: five general-purpose harnesses, two skill-augmented
Codex/GPT-5.5 harnesses (ARIS, Orchestra) and two research-orchestration
systems on GPT-5.5 (DeepScientist, AutoResearchClaw). Three harnesses are also
run "stagewise", where each stage is given the paper's own upstream outputs.

What the numbers show:
- Execution (mean 43.0) scores below design and planning (61.6) in all nine
  runs.
- 14 of 90 required claims are met. 13 of the 14 are method claims
  (implementation checks). Only 1 of 54 empirical-claim slots is met
  (GPT-6 Astra's latency claim).
- Four systems each report 8 claims attained but meet 1 to 4.
- Every run stops with at least 6 of 10 claims unmet. Eight of nine use
  under 10% of the 72-hour clock, even though the prompt says to keep
  revising until 90%.
- Stagewise inputs raise met claims from 1–3 to 7–8.

## Claims

- **An "execution gap" exists.** "Across all evaluated systems, experimental
  execution consistently lags behind design and planning."
- **Self-reports don't track attainment.** "The evaluated systems report
  success on as many as eight required claims, yet at most four meet their
  criteria." Their self-reported progress is "a poor proxy for scientific
  progress."
- **Retries rarely flip a verdict.** "Repeated attempts thus change the
  numbers but rarely turn an unmet claim into a met one" (Sec. 5.2). The
  abstract compresses this to "repeated attempts rarely yield measurable
  improvements."
- **The gap is partly inherited.** "Much of the end-to-end execution gap
  thus appears to originate upstream of execution", based on the stagewise
  comparison.
- **Added structure does not help.** "Skill augmentation mainly improves
  paper writing while lowering execution scores, and research-specific
  orchestration lowers execution scores further." Neither "reliably yields
  higher scores."
- **Recommendation (Discussion).** Future systems "may benefit more from
  checking each claim against an explicit criterion than from relying on
  their own progress reports."
- **Limits the authors state.** The rubrics follow the anchor paper, so they
  "may undervalue scientifically valid alternatives". A literature cutoff
  does not rule out pretraining exposure to the anchor paper.

## Methods

- **Task construction.** The task is built from the SPINT paper, its
  supplement and its official repo. Each derived item is traced to evidence.
  There are ten claims: four method claims (MC1–MC4: set decoder,
  permutation invariance ≤ 10⁻⁵, gradient-free calibration, dynamic channel
  dropout) and six empirical claims (E1–E6: beat four named baselines on all
  three subjects, few-shot calibration, training-day scaling, unit-loss
  robustness at 20% retention, latency ratio < 1, three ablations). Each
  claim lists its dependencies and the extrapolations it does not license.
  One domain researcher reviewed the reference, the claims and the rubrics
  before any run.
- **Modes.**
  - *End to end:* one workspace, and each stage builds on the system's own
    earlier artifacts.
  - *Stagewise:* each stage runs in a fresh workspace that holds rewritten
    paper-derived inputs for all preceding stages. Stage budgets sum to
    72 h, including 48 h for execution.
- **Scoring.** Items are rated 0–4 and averaged within a dimension.
  Overview = 0.3 design + 0.5 execution + 0.2 analysis/writing. Execution
  is four equally weighted dimensions: module development, code execution,
  result quality (per claim) and claim completion (per claim). A
  well-executed negative result earns 3/4 on claim completion. Missing
  items stay in the denominator.
- **Judge.** claude-opus-5 via the Claude Code CLI, pinned, one sample per
  call, nine calls per workflow, temperature not exposed. The submission is
  treated as untrusted data, and self-reported claim states carry no
  weight. Eight integrity gates (forbidden source, reference leak, test
  protocol, fabricated execution, and others) zero the dependent items. One
  gate fired: Codex/GPT-5.5's figure was plotted from the wrong result
  file. Validation of the judge: "A domain researcher spot-checked the
  judge's ratings and rationales and found them largely consistent". No
  counts or agreement statistics are given.
- **Iteration diagnostics.** Systems log claim events through a provided
  recorder. It is hash-chained, and timestamps are added from outside the
  workspace. After the run the evaluator maps attempts onto required
  claims. The "reported" track counts a claim when the system first marks
  it attained. The "confirmed" track places it at the first attempt the
  judge finds sufficient. These diagnostics carry zero weight in the score.
- **Instructions.** The workspace instructions were revised over pilot
  runs. In the pilots, runs "ended after using about 1% of their budget".
  The fix was frozen claims, one claim at a time, and "an unmet claim is
  revised and rerun until it meets its criterion or the run reaches 90% of
  its budget." This 90% line is an instruction, not enforced. The runner
  enforces only the timeout, the artifact contract and the input
  allowlist. The research-orchestration systems got the instructions
  through their own entry points, without the stage order and without the
  background-polling line.
- **Systems (Table 14).** All at high effort with no fallback model:
  - Codex CLI 0.153.4 with GPT-5.5, GPT-5.6 Sol or GPT-6 Astra;
  - Claude Code 2.1.269 with Opus 4.8;
  - Kimi Code 0.39.1 with K3;
  - ARIS and Orchestra skills on Codex/GPT-5.5;
  - AutoResearchClaw 0.5.0 and DeepScientist 1.6.0 on GPT-5.5.

  Sol and Astra ran in a later batch. The skill harnesses ran on a second
  host and the orchestration systems on a third.

## Results

- **End-to-end overview (Table 2, Table 13).**

  | System | Overview | Exec (S5) | Met / reported | Attempts | Wall-clock (h of 72) |
  |---|---|---|---|---|---|
  | Codex / GPT-5.5 | 45.1 | 44.0 | 1 / 0 | 9 | 0.30 |
  | Codex / GPT-5.6 Sol | 59.5 | 51.4 | 3 / 8 | 24 | 1.72 |
  | Codex / GPT-6 Astra | 64.9 | 58.6 | 4 / 8 | 10 | 1.10 |
  | Claude Code / Opus 4.8 | 61.3 | 57.2 | 1 / 3 | 9 | 5.73 |
  | Kimi Code / K3 | 64.2 | 61.6 | 3 / 2 | 18 | 20.56 |
  | ARIS (GPT-5.5) | 44.5 | 37.8 | 1 / 8 | 28 | 0.74 |
  | Orchestra (GPT-5.5) | 45.2 | 39.4 | 1 / 8 | 10 | 0.90 |
  | AutoResearchClaw (GPT-5.5) | 18.6 | 15.1 | 0 / 0 | 0 | 0.71 |
  | DeepScientist (GPT-5.5) | 39.9 | 21.5 | 0 / 0 | 34 | 7.00 |

  Exec values for rows other than the three harnesses in Table 15 are our
  arithmetic, the mean of the four dimension scores.
- **Per research part (mean over nine).** Design 61.6, execution 43.0,
  analysis/writing 46.5. Result quality is the weakest dimension (26.7),
  and module development the strongest in execution (57.9).
- **Which claims are met (Table 16).** MC1 (the set decoder exists and runs)
  accounts for 5 of the 14 met claims, MC2 for 5 and MC3 for 3. The only met
  empirical claim is E5 (latency) for Astra. E3 (training-day scaling) scores
  0/0 in every end-to-end run.
- **Reported vs met.** Over-reporting is concentrated in four runs: Sol
  8/3, Astra 8/4, ARIS 8/1 and Orchestra 8/1. Two runs under-report: Kimi
  2/3 and Codex/GPT-5.5 0/1. Sol's met count reaches 3 at 0.26 h into
  execution and stays there, while its reported count climbs 3 → 8 between
  0.76 and 1.52 h. ARIS and Orchestra each report 8 within half an hour.
  The judge's rationales for Sol show what the reports meant:
  - E1 used baselines of its own choosing, and it still trailed one of them
    on H1.
  - E2 failed on M2/H1, where one-shot R² was negative.
  - E4 tested only 70% retention, so the required 20% condition does not
    exist.
  - E5 timed H1-shaped inputs only and reported absolute latency instead of
    the ratio.
- **Retry trajectories (Fig. 4c, Fig. 6).** These come from the longest
  audited chain on E1 for four systems. Sol's M1 session-mean R² goes
  0.49 → 0.54 over six attempts. ARIS, DeepScientist and Kimi end within
  0.015 of their first attempt, and Kimi has no chain longer than two. None
  meets E1. Attempt counts are unrelated to met counts: ARIS 28 → 1,
  DeepScientist 34 → 0, Astra 10 → 4.
- **Stagewise (Table 15, three harnesses).** Every stage rises.
  - Execution S5: 44.0 → 77.6 (Codex/GPT-5.5), 57.2 → 86.1 (Claude), 61.6 →
    74.9 (Kimi).
  - Overview: 45.1 → 78.5, 61.3 → 81.0, 64.2 → 74.8.
  - Met claims: 1 → 7, 1 → 8, 3 → 7.
  - Most of the execution gain is in result quality and claim completion.
- **Skill augmentation (same GPT-5.5 backbone).**
  - Overview: ARIS 44.5, Orchestra 45.2, Codex alone 45.1.
  - Paper writing: 52.5 and 47.5 vs 32.5.
  - Result quality: 20.0 for both, vs 30.0.
  - Claim completion: 32.5 and 37.5 vs 42.5.
  - Met: 1 each, the same as Codex alone.
- **Orchestration.** DeepScientist has the best GPT-5.5 design scores, but
  its code execution is 25.0 (vs 50.0), its result quality 0.0, and 0 of 34
  attempts meet a criterion. AutoResearchClaw crashed on a missing import
  before any scientific attempt.
- **Resources (Fig. 5).** Spearman with overview is 0.47 for time and 0.23
  for tokens over n = 9, and 0.79 and 0.57 without DeepScientist.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). All three digest sentences
  are in the abstract. The body narrows each one:
  1. *"Repeated attempts rarely yield measurable improvements."* The body
     says something different. Attempts "change the numbers but rarely turn
     an unmet claim into a met one". One of the four audited chains improved
     measurably (0.49 → 0.54 R²), and what fails is the verdict flip. The
     evidence is four chains on one claim of one task. Attempts are
     self-logged, and the evaluator "does not judge whether a revision was
     substantive."
  2. *"Neither skill augmentation nor research-specific orchestration
     reliably helps."* Each arm is one run on one task. The skill arms ran
     on a different host from the baseline. The orchestration systems got
     different instructions (no stage order, no polling line).
     AutoResearchClaw's 18.6 comes from an import crash, an engineering
     failure rather than a test of orchestration. "Reliably" cannot be
     assessed at n = 1.
  3. *"Execution lags design everywhere."* This holds in all nine rows. But
     the design rubric and the execution rubric are different instruments
     with different item difficulty, so the gap's size is not a measured
     capability difference.

  The digest's own "Why" reads the retries as the stall regime `/iterate`
  halts on. The paper shows a different regime: runs **stopped early**. All
  nine ended with at least 6 of 10 claims unmet. Eight used ≤ 7.0 h of
  72 h, and the most was 20.56 h (29%). This happened under an explicit
  instruction to keep revising until 90%.
- **Stagewise is partly circular.** The stagewise inputs are the paper's
  own claims, method and plan, and the rubric is derived from the same
  paper. A system that follows the reference plan runs the experiments the
  rubric checks for. "The gap originates upstream" is therefore
  confounded with "the system now runs the reference's experiments". No arm
  gives a *correct but different* upstream plan.
- **The judge is unvalidated.** claude-opus-5 gives one sample per call,
  with no repeat-sampling variance, no inter-rater statistic and no planted
  cases. Its only validation is an informal spot-check by one domain
  researcher. All met/unmet verdicts, which are the paper's sharpest
  numbers, come from one claim-level call per run. The judge shares a
  family with one contestant (Opus 4.8). That contestant does not top the
  table, so self-preference is not visible, but it was not tested.
- **Rubric fidelity to one paper.** The authors concede it. A system with a
  valid alternative design (e.g. different baselines with a sound argument)
  is scored unmet. This matters for E1, where Sol's own baselines were
  marked unmet. Some of that is substitution, and some may be legitimate
  disagreement. The paper does not separate the two.
- **No uncertainty anywhere.** There are no seeds, no repeats and no CIs.
  The Spearman values are over n = 9 heterogeneous runs on three hosts and
  two batches.
- **Reproducibility.** The reproducibility statement says "The task package
  and evaluation code are available" at the GitHub repo. On 2026-10-20 the
  repo held only a README reading "Code coming soon" (in Chinese). The
  rubrics, reference, judge prompts and run artifacts are not available.
  Appendix excerpts are translations of Chinese originals.
- **What is solid.** The reported-vs-met design is sound and inspectable:
  claims frozen pre-run and hidden, a hash-chained event log with external
  timestamps, and per-claim rationales naming the missing condition. The
  Sol rationales are concrete evidence of *scope substitution*: the system
  ran a narrower experiment and filed it under the required claim's name.
- **Bearing on the hub's live fronts.**
  - *Redirect-before-halt proposal (2026-10-18).* Neutral to weak. There is
    no pivot arm and no halt arm. The retries are within-claim, under a
    fixed "rerun until criterion" prompt. Three of four audited chains
    barely moved, which is consistent with bobadillasuarez2026audit's
    "resampling a fixed move set mostly does nothing". The larger point:
    the binding stop in every run was the agent's own exit, not a stall.
    A no-improvement counter never sees an agent that reports success and
    quits. That is a different failure from the one the proposal targets.
    The stagewise result hints that some stalls come from upstream
    (claims/plan), and the proposal's axes do not include those. Given the
    circularity above, this is a hint only.
  - *No-improvement noise band.* Three of the four E1 chains moved under
    0.015 R². A band would correctly count those as no improvement. No
    seed variance is reported to calibrate against.
  - *HCE evaluation.* This is an evaluator-owned verdict against criteria
    frozen and hidden before the run, which is the evaluation.md
    discipline applied to whole research runs. It also shows the cost of a
    single-reference rubric: valid alternatives score as failures.
  - *Shared-substrate correlated error.* All four structure arms share
    GPT-5.5, and one judge scores all nine runs. A judge bias would move
    every row the same way, so cross-system ranks are safer than absolute
    levels.
  - *Skill scoping.* Two general research skill packs tied the bare
    harness overall. They raised write-up scores, lowered evidence scores,
    and came with 8 self-reported attainments against 1 met. This is weak
    support (n = 1 per arm) for scoping skill groups to the projects that
    need them, and for judging a pack on evidence dimensions rather than
    overall score.

## Trust signals

- **Credibility:** 2.
  - For: Tsinghua (School of Life Sciences / IDG-McGovern) and SUSTech
    authors. The protocol design is unusually careful: frozen and hidden
    criteria, integrity gates, a hash-chained claim log, missing items kept
    in the denominator, and evaluator failures not scored as zero. The full
    per-claim verdict table (Table 16) and resource table (Table 13) are
    printed. System versions and commits are pinned (Table 14). The
    authors state the rubric-fidelity and pretraining-exposure limits
    themselves.
  - Against: no peer review (v2, one week old). One task, one run per
    system, no CIs. A single-sample LLM judge whose only validation is an
    informal spot-check. Arms ran on three hosts and two batches with
    partly different instructions. The repo named in the reproducibility
    statement holds only a "code coming soon" README (checked 2026-10-20).
    Model names (GPT-5.6 Sol, GPT-6 Astra, Kimi K3) describe a provider
    snapshot.
  - Not 3 because the claimed release is absent and every system comparison
    rests on a single run of a single task. Not 1 because the group is
    reputable and the printed verdict and rationale tables make the main
    reported-vs-met finding checkable.

## Follow-up

- **Relevance:** 3. This is a second measured instance, after
  [[literature/papers/kim2026into]], of self-reported completion running
  ahead of an external verdict on whole research runs. It adds a concrete
  mechanism: a narrower experiment filed under the required claim's name.
  It also gives a clean count of agents stopping with most of the budget
  unspent. It is cite-worthy for [[concepts/evidence-gated-completion]],
  [[concepts/budget-as-ceiling]] and [[concepts/skill-library-lifecycle]].
  It does not shift any of them: one task, one run per system and an
  unvalidated judge. It is less load-bearing than
  [[literature/papers/xu2026researchclawbench]] for end-to-end
  benchmarking.
- **Adoptable piece (not proposed).** A completion check should map each
  agent-reported claim onto a pre-run claim ID. It should then check that
  every condition the criterion names exists in the evidence (subjects,
  comparators, grid points). This is [[literature/papers/waks2026what]]'s
  expected-run manifest at claim grain. A claim-name match is not evidence.
- **Recheck** the repo for the task package and judge prompts. If they are
  released, the per-claim rationales can be audited and credibility
  revisited.
- **Candidates** (cited, not in the graph):
  - Trehan & Chopra 2026, "Why LLMs aren't scientists yet: Lessons from
    four autonomous research attempts" (arXiv 2601.03315).
  - Zhang et al. 2026, "How far are we from true auto-research?"
    (arXiv 2605.19156).
  - Garikaparthi et al. 2026, ResearchGym (arXiv 2602.15112).
  - Cui et al. 2026, ARAC (arXiv 2608.12788).
  - Zhou et al. 2026, ASI-Bench (arXiv 2608.17271).
  - Yang et al. 2026, ARIS (arXiv 2605.03042), and Liu et al. 2026,
    AutoResearchClaw (arXiv 2605.20025). These are the systems under test.
  - Already in the graph: [[literature/papers/starace2025paperbench]],
    [[literature/papers/zou2026fmlbench]],
    [[literature/papers/xu2026researchclawbench]].
