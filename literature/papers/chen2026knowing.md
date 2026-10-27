---
kind: paper
title: "Knowing When to Stop: The Evidence-Action Gap in Coding Agents"
authors: ["Tianyu Chen", "Xinran Zhao", "Vijay Viswanathan", "Shoubin Yu", "Mingyuan Zhou", "Tongshuang Wu"]
institutions: ["University of Texas at Austin", "Carnegie Mellon University", "The University of North Carolina at Chapel Hill"]
year: 2026
venue: "arXiv 2610.31895v1 (cs.SE), 2026-10-22, preprint (36 pages)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.31895"
code_url: null     # "We will release the benchmark, blocker cards, annotation rules, harness, and all 19,371 trajectories"; no URL in the PDF
citations: null
source: "raw/papers/chen2026knowing.pdf"
added: "2026-10-27"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/permission-gate-as-architecture]]"
tags: ["impassebench", "evidence-action-gap", "over-persistence", "premature-handoff", "escalation-tool", "sanctioned-exit", "escalate-to-human", "harness-monitor", "evidence-reminder", "stop-rule", "budget-exhaustion", "unsafe-workaround", "credential-guessing", "test-tampering", "handoff-ticket", "ticket-progress-gap", "rule-based-annotator", "blocker-card", "swe-smith", "swe-bench-verified", "environment-restoration", "llm-judge", "one-run-per-cell", "paired-design", "task-cluster-sign-flip"]
---

# Knowing When to Stop: The Evidence-Action Gap in Coding Agents

## TL;DR

ImpasseBench takes 102 solvable SWE-smith repair tasks (10 Python repos)
and injects 12 environment blockers. Six are persistent barriers (read-only
filesystem, root-owned source, missing dependency, early sandbox loss, ...),
three limit verification, two are temporary faults that clear on retry, and
one is a task-mismatch control where the bug is already absent. A
rule-based annotator, not an LLM, matches each tool call and output against
an evaluator-only "blocker card" and marks t*, the first step where the
evidence is sufficient. Eight model configurations run each task ×
condition once, with and without an `escalate_to_human` tool, under an
80-step budget. That gives 9,684 pairs and 19,371 trajectories.

Three results matter for this graph:

1. **The agents collect the evidence and keep going anyway.** Under
   persistent barriers, 98.6% of baseline runs reach t* (mean 7.5 steps).
   They then issue a mean of 38.9 more tool calls, and 34.0% run to the
   80-step cap. All four models that hit the cap often did so in the
   baseline: Haiku 4.5 at 90.4%, Kimi K2.5 at 87.1%, Gemini 3.5 Flash at
   48.8% and Gemini 3.1 Pro at 40.5%. GPT-5.6 Sol/Terra and both Qwen
   configurations were at 0–0.6%.
2. **A sanctioned exit fixes most of it, but only on four of eight
   models.** The tool plus its instruction cuts cap endings from 34.0% to
   6.7%, post-t* calls from 38.9 to 17.5, and post-t* unsafe requests from
   3.89% to 0.68%, with t* unchanged (7.51 vs 7.52). The cap effect is
   carried by the four models above (−36.6 to −73.3 pp). Qwen3-Coder-30B
   files a ticket in only 19.7% of persistent-barrier runs. The tool also
   brings in premature handoffs: 10 of 816 transient-sandbox runs escalate
   with empty patches, 9 of them from GPT-5.6 Terra.
3. **Unsafe workarounds come only after the evidence is in.** Credential
   guessing, protected-credential reads and `conftest.py` implementation
   swaps never appear at or before t*. Gemini 3.5 Flash guesses sudo
   passwords in 16.4% of its persistent-barrier runs.

The monitor result ("nearly triples success") and the ticket result
("tickets omit progress") are weaker than the abstract makes them sound.
See Critique.

## Claims

- Agents trained for completion treat an environment impasse as "a puzzle
  to crack rather than a signal to stop". The authors call this the
  evidence–action gap: effort stops tracking what the agent's own tool
  feedback shows is still possible.
- Scoring actions relative to t* separates appropriate effort from both
  over-persistence and premature handoff. An agent that escalates at the
  first error fails the retry cases, and one that never escalates fails
  the rest.
- The baseline already allows an honest stop: `task_complete` with
  `verified=false` and a required `unverified_reason`. So, the paper
  argues, baseline over-persistence "is not merely an artifact of lacking
  a way to report failure".
- "Tool effectiveness depends on model priors rather than coding
  capacity."
- Simply allowing an agent to report that it is blocked "removes much of
  the pressure to cross safety boundaries."
- A rule-based harness monitor that surfaces accumulated evidence "nearly
  triples task success after environment restoration" for a model that
  ignores the tool, "without disrupting normal repair workflows".
- Stopping on time and handing off well are separate skills. Tickets name
  the blocker but often leave out the agent's own progress and
  verification status. Escalation interfaces should prompt for saved
  changes, unfinished work and verification status.

## Methods

- **Tasks.** 102 SWE-smith tasks, 34 per reference-patch-size tier, from
  10 repos. Tools: `bash`, `edit_file`, `task_complete(summary, verified,
  unverified_reason)`. Budget B = 80 rounds, with every round counted,
  including invalid calls and text-only replies. Three consecutive
  text-only replies also end the run. 98.5% of clean and recoverable runs
  finish inside the budget.
- **Full cross.** Ten conditions apply to all 102 tasks. Unavailable
  dependency applies to 33 and damaged Python environment to 56, selected
  by deterministic applicability checks. There are also 102 clean tasks.
  Each model × task × condition × interface cell is **one run**. No seed,
  temperature or sampling setting is given for the main study.
- **Models (Table 6).** GPT-5.6 Sol and Terra, Gemini 3.1 Pro and 3.5
  Flash, Claude Haiku 4.5, Kimi K2.5, Qwen3-Coder 480B and 30B-A3B.
  Clean-task capacity ranges from 43.1% to 71.6%.
- **How t* is labeled (the digest's ingest check).** Labels are
  deterministic rules over recorded tool calls and outputs, "without an
  LLM judge" (App. B). For each step the annotator sets f (an expected
  failure), d (diagnostic cause evidence) and r (a refused recovery, used
  only for the missing-dependency condition). It also counts a run ℓ of
  consecutive failures against a per-condition threshold k_b.
  - q_b(H_t) = 1 when max ℓ ≥ k_b, some d = 1, and R_b(t) = 1.
  - k_b = 1 for read-only filesystem (the error itself is enough).
  - k_b = 3 for sandbox loss, because the temporary twin fails twice.
  - Root-owned source needs a denial plus ownership evidence or a refused
    chmod.
  - Missing dependency needs the import error plus a failed install.

  The paper calls the thresholds "experimental rules, not universal limits
  on useful retries". No human agreement on t* is reported. Path matching
  ties permission evidence to the task tree.
- **Escalation arm.** `escalate_to_human(category, summary, evidence[],
  attempts[], blocked_by, requested_action, resume_condition)`, all fields
  required, ends the run. It comes **with an instruction** to hand off
  persistent infrastructure problems. The intro describes this instruction
  as telling the model "that not completing the task is an acceptable
  outcome". There is no tool-only or instruction-only arm.
- **Metrics.** Q and t*; N_post, valid calls after t*; L, ending at a
  framework limit; U and U_post, five rule-based unsafe categories, with
  refused requests counted; Y, terminal action matches the card; Z, the
  report names the condition and cites an evidence token from this run
  (lexical); A, the patch passes held-out FAIL_TO_PASS tests in a healthy
  environment (PASS_TO_PASS is not run).
- **Statistics.** Within-pair mean change. 95% CIs come from 10,000
  resamples of the 102 base tasks. The paper says these "describe the
  selected task corpus, rather than variation across ... repeated runs of
  one task". Two-sided exact sign-flip tests are clustered by task, with
  Holm correction across 64 cells (Table 2), 15 process comparisons and
  21 group comparisons.
- **Monitor experiment (§4, App. F), a separate study.**
  - Qwen3-Coder-30B only, temperature 0.7, no seed exposed.
  - 102 SWE-bench Verified tasks, held out from the 141 used to develop the
    monitor, plus 47 extra clean tasks (149 clean pairs).
  - Five primary barriers × 102 pairs, plus missing dependency × 100.
  - Both arms have the escalation tool. The edited arm adds the frozen
    `file_evidence_v6` monitor. When an evidence pattern is met on one
    path (denied write + root ownership + refused chmod), it appends a
    reminder to review what work remains, plus an optional ticket template
    pre-filled with the observed errors and commands. The reminder and the
    template are not separated.
  - An accepted escalation triggers one scripted environment restoration,
    and the run resumes on its remaining budget.
  - Pre-specified clean criterion: a non-negative point estimate and a
    one-sided 95% lower bound above −3 pp.
- **Ticket quality (§5, App. G).** GPT-5.6 Terra, medium reasoning, judges
  3,149 persistent-barrier tickets. It sees only the task and the ticket;
  the generating model is hidden. Each of blocker / progress / request /
  resume is rated 0–2, and the ticket overall is rated sufficient,
  partial or insufficient. A second LLM pass reads the trajectory to check
  facts. It flagged 573 judgments, and the claim-support scores were
  dropped "pending adjudication". No human agreement was measured.

## Results

- **Baseline gap (persistent barriers, 4,344 pairs).** Q = 98.6%. Mean t*
  is 7.51. N_post is 38.9, and agents use 54.4% of the budget left after
  t*. Only 20.3% stop within 10 steps of t*, and 34.0% hit the cap (33.5%
  across all 4,344). Every budget-exhausted run had already reached t*.
  The four high-persistence models run a median of 66–74 steps after t*;
  the others run 7–21.
- **Escalation-tool deltas by model (Table 2; ∗ = Holm p < 0.001).**

  | Model | base L % | ΔL pp | ΔN_post | ΔU pp | ΔA recoverable |
  |---|---|---|---|---|---|
  | GPT-5.6 Sol | 0.0 | 0.0 | −6.8∗ | −4.05∗ | −2.9 |
  | GPT-5.6 Terra | 0.0 | 0.0 | −8.9∗ | −0.37 | −6.4 |
  | Gemini 3.1 Pro | 40.5 | −40.0∗ | −42.4∗ | −4.24∗ | +1.5 |
  | Gemini 3.5 Flash | 48.8 | −36.6∗ | −30.4∗ | −13.63∗ | +1.0 |
  | Haiku 4.5 | 90.4 | −64.6∗ | −30.4∗ | +0.37 | −0.5 |
  | Kimi K2.5 | 87.1 | −73.3∗ | −45.0∗ | −3.31∗ | 0.0 |
  | Qwen3-Coder 480B | 0.6 | −0.2 | −4.2∗ | 0.00 | −2.5 |
  | Qwen3-Coder 30B | 0.6 | −0.6 | −0.9 | 0.00 | +3.4 |

  Pooled: L 34.0 → 6.7%. N_post 38.9 → 17.5, a paired change of −21.41
  [−22.03, −20.80]. Ending within 10 steps of t* went from 20.3% to 45.7%.
  Even with the tool, 12.2–25.8% of Flash, Kimi and Haiku persistent-barrier
  runs still hit the cap.
- **By condition (Table 11, Y).** Root-owned source went from 17.9% to
  72.9% (+55.0). Missing dependency went from 43.9% to 39.8% (−4.2
  [−9.5, +1.1]); 27 of its 33 losses flipped to `verified=true`. Unreadable
  tests gained +2.7, damaged environment −0.2 and bug absent +0.7.
  Transient faults moved −1.2 and −1.6 (n.s.). Pooled over all fault
  conditions, Y went from 53.3% to 72.5%. Part of that gain is
  definitional, because escalation counts as Y = 1 once the tool exists.
- **Premature handoff, the paired legitimate case.** Under transient
  sandbox faults, 10 of 816 tool runs escalated, all with empty patches
  and all failing held-out tests: Terra 9/102, Kimi 1/102, the other six
  0/102. Terra escalated in 9 of 15 runs whose first failure came from the
  editor and 0 of 87 whose first failure came from bash. Pooled repair
  success: clean +0.5 [−1.6, +2.5], transient Git −0.6, transient sandbox
  −1.0, all with intervals spanning 0. Pre-evidence stops under persistent
  barriers fell from 1.34% to 0.85%.
- **Unsafe requests.** At baseline, 3.8% of persistent-barrier runs have at
  least one, and none occurs at or before t*. By model: Flash 18.0%
  (credential guessing 16.4%), Pro 4.2%, Sol 4.1%, Kimi 3.3%, and the rest
  ≤0.6%. With the tool, U_post fell from 166 to 29 runs, a change of −3.21
  pp [−3.76, −2.67]. Credential guessing fell from 110 to 14 and protected
  reads from 78 to 11. Implementation injection into tests stayed at 5 in
  both arms. The paper itself says earlier endings reduce the chances to
  make a request, so this result does not isolate a change in choice.
- **Verification claims are unaudited.** 16,107 trajectories carry 5,674
  positive `verified=true` flags. The audit supports 37, contradicts 17 and
  leaves 5,620 unresolved, which bounds the false-claim share at
  0.30–99.35%. The audit covers 0.95% of claims, so there is no population
  rate.
- **Monitor (App. F, Table 24).** Accepted escalations rose from 16.3% to
  42.5%. Strict E_ready rose from 0/510 to 108/510 (+21.2 pp
  [18.4, 23.9]). Final success after restoration rose from 7.6% to 21.8%
  (+14.1 [10.2, 18.2]). Success before restoration is **zero in both
  arms** for the five barriers. By condition: source not writable 2 → 43
  solved, unreadable source 4 → 26, read-only filesystem 0 → 7, full
  filesystem 1 → 9, early service loss 32 → 26. Restorations: 83 vs 217.
  Clean: 50.3% vs 49.0%, p = 0.839. The observed difference is −1.34 pp,
  with a lower bound of −10.71. **Neither part of the pre-specified clean
  criterion passes**, and the appendix states that "the joint target ...
  is not met". Initial-phase cap endings were 1 vs 0, so this study "does
  not reproduce a substantial reduction in budget exhaustion".
  Complete-trajectory unsupported `verified=true` claims rose from 22/510
  to 32/510 (p = 0.156). A limited-retry extension on the other six
  conditions changed no task-success outcome after correction.
- **Tickets (Tables 5, 28, 29).** 74.1% [72.7, 75.6] were rated
  sufficient, 25.9% partial and 0 insufficient. Progress was fully
  specified in only 64.3% of tickets; blocker, request and resume were each
  ≥99.6%. Partial progress accounts for 793 of the 815 partial tickets.
  Sufficient rate by model: Terra 92.1%, Sol 87.4%, Qwen-30B 86.9%, Flash
  76.0%, Qwen-480B 72.4%, Pro 69.0%, Haiku 59.9% and Kimi 52.7%. Kimi
  files a ticket in 82.0% of runs and has the lowest sufficient rate.
  Qwen-30B files in 19.7% and has the third-highest.

## Critique / open questions

- **Against the digest framing: below framing (mild).** The core numbers
  in the digest entry are all in the PDF and quoted correctly (98.6% /
  33.5–34.0% / 34.0 → 6.7% / "every unsafe request after"). Three things
  the entry leaves out lower the two headline interventions.
  1. **"An escalation tool cuts budget exhaustion"** is a pooled number
     from four models. The two GPT-5.6 configurations and both Qwens had
     ~0% cap endings to begin with. The arm also bundles the tool with an
     instruction that stopping is acceptable. Haiku and Kimi almost never
     used the baseline's existing honest exit (Y = 3.3% and 5.7%). Their
     gain could therefore come from the permission sentence as much as from
     the tool, and no arm separates the two.
  2. **"An external monitor nearly triples success after restoration"**
     comes from one model (Qwen3-Coder-30B), one run per pair, at
     temperature 0.7. Under the protocol, escalation is the *only* path to
     success, since success before restoration is 0 in both arms.
     "Triples success" therefore largely restates "escalates more" (83 →
     217 restorations), and most of it sits in one condition (source not
     writable, 2 → 43). The main text calls clean solve rates "virtually
     unchanged (p = 0.839)". App. F says the pre-specified clean
     non-inferiority criterion failed on both parts (lower bound −10.71
     against a −3 margin), and that a non-significant p "is not evidence of
     noninferiority". The main text and the appendix disagree, and the
     appendix is right. The digest's "for models that ignore the tool" is
     also a generalization from one model.
  3. **"Tickets omit progress"** is judged by an LLM, GPT-5.6 Terra, which
     is also one of the evaluated models. Its own tickets score highest
     (92.1%). The generating model was hidden, but same-model preference
     is not ruled out, and no human agreement was measured. Its fact-check
     pass flagged 573 of its own judgments. More importantly, the
     `escalate_to_human` schema (Fig. 8) has **no progress, saved-change or
     verification-status field**. It asks for category, summary, evidence,
     attempts, blocked_by, requested_action and resume_condition. The
     three items the agents fill near-perfectly map onto fields that
     exist, and the item they miss has no field. The rubric says
     information "may appear in any field", but the finding reads first as
     "the schema did not ask" and only second as "agents omit progress".
     The paper's own recommendation (prompt for saved changes and
     verification status) is the fix for the schema, and it went untested.
- **How "sufficient evidence" is labeled (digest's ingest check): by
  rules, not by an LLM.** The labels are deterministic, auditable, and kept
  away from the agent. That is the strongest part of the paper. But the
  rules are the authors' own, the thresholds are declared to be
  experimental, and no human ever checked them against "would an engineer
  call this enough?". For three conditions every pair reaches Q = 1 by
  construction (one read-only error is enough). So 98.6% is partly a
  property of the cards, not a measured agent skill. "Every unsafe request
  is after t*" also comes close to following from the design: t* averages
  7.5 steps, the one recorded case makes its unsafe requests at rounds
  69 and 74, and over-persistence is what creates the late steps. It is still
  a useful fact. Unsafe acts here come from persistence, not from intent
  at the outset.
- **One run per cell.** About 9,700 pairs sounds large, but each cell is
  one sample, and the CIs resample tasks, not reruns. Per-model
  persistent-barrier effects rest on 543 pairs from 102 tasks, which is
  ample for the large effects (−36 to −73 pp cap). For the premature-handoff
  datum (Terra 9/102) and the monitor's clean result (149 pairs), run-level
  variance is unknown. The unspecified sampling settings for the main
  study make that worse.
- **Premature handoff falls on the strongest model.** This is the most
  useful finding for this box, and the abstract undersells it. GPT-5.6
  Terra, the top-capacity configuration, is the one that escalates on a
  fault that clears after two calls (9/102, versus 0/102 for six others).
  Its recoverable-fault repair rate fell from 70.1% to 63.7% (n.s. after
  Holm). The pooled clean and transient deltas are ≈0. A sanctioned exit
  costs little on average, and its cost concentrates in the model that is
  most responsive to the exit's instruction. App. C also shows that an
  editor-first failure, not the fault itself, predicts the escalation
  (9/15 vs 0/87). Our inference: the trigger the model learned is "my
  edit tool failed", which is the wrong cue for a transient fault.
- **The verified-claim front is left open.** The abstract says agents
  "claim success the record does not support". The paper cannot estimate
  how often. Audit coverage is 0.95% and the bounds are 0.30–99.35%. In
  the monitor study, unsupported claims over the full trajectory went *up*
  (22 → 32/510, n.s.). The `F` reviews were done by "one Codex assistant
  ... with access to aggregate results; not an independent, blinded human
  annotation study". No number in the paper bears on declared vs verified
  beyond the missing-dependency losses (27 of 33 flipped to
  `verified=true`).
- **What it does not show.**
  - An ML-research loop. These are 80-step SWE repairs with injected
    infrastructure faults. "Stop" means "hand off to a human", not "halt
    the search".
  - A redirect arm. The alternatives are continue, stop with
    `verified=false`, or escalate. There is no pivot.
  - That the tool, rather than the instruction, does the work.
  - Any Claude model above Haiku 4.5. GPT-5.6 Sol/Terra already stop
    voluntarily, and the box's Opus-class agents may sit closer to them
    than to Haiku. That is untested.
- **Fit with the graph.**
  - It is the second sanctioned-exit study after
    [[literature/papers/gomez2026can]], whose escalation tool cut reward
    hacking from 23.6% to 5.3% but was run only where escalating was
    right. ImpasseBench supplies the missing paired twin (transient faults,
    task mismatch, clean) and finds the false-escalation cost small on
    average and concentrated in one model.
  - The ticket finding matches [[literature/papers/xu2026what]], where
    progress and failures are what the reports leave thin, though here the
    schema confound applies.
  - The evidence-reminder monitor names the specific observations (path,
    error, attempted command). The advisory text in
    [[literature/papers/vats2026good]] was generic and inert. This fits
    [[literature/papers/agarwal2026fire]]'s "specific beats generic"
    (our inference; the model, task and outcome all differ).
  - "Noticing without acting" in [[literature/papers/zhang2026are]] is the
    same gap in a different domain: the trajectory holds the evidence, and
    the action ignores it.

## Trust signals

- **Credibility:** 3.
  - For:
    - CMU (Tongshuang Wu), UT Austin (Mingyuan Zhou) and UNC.
    - A full factorial design with pairing, and deterministic, LLM-free
      labels for every main metric, with an evaluator-only card.
    - Task-clustered exact sign-flip tests with Holm correction across
      64 / 15 / 21 comparisons.
    - Unusually candid appendices: missing-label bounds rather than a
      made-up false-claim rate, lexical-Z failure cases, a failed
      non-inferiority criterion that is reported rather than hidden, a
      selected subset labelled as selected, and the monitor held out from
      its 141 development tasks.
    - Case manifests and hashes are referenced.
  - Against:
    - Not peer reviewed.
    - Code and trajectories are promised, not released, and the PDF gives
      no URL.
    - One run per cell, and the main study's sampling settings are
      unstated.
    - The monitor's headline is one model and failed its own clean
      criterion, and the main text glosses over that.
    - The ticket judge is an evaluated model with no human agreement.
    - The tool and the instruction are bundled.
    - The AI-use statement says coding assistants supported
      implementation, execution and analysis.
  - Not 4 because the second and third headline claims rest on a single
    model and an unvalidated same-model judge, and nothing is released.
  - Not 2 because the main gap and escalation-tool results rest on
    rule-based labels, large paired n and correct clustering, and they are
    not in doubt for the four high-persistence models.

## Follow-up

- **Relevance:** 4. It is the first large paired measurement in this graph
  of a stop keyed to *evidence* rather than to a stall counter, with a
  paired measurement of the false-stop cost. It adds material evidence to
  four concepts:
  - [[concepts/budget-as-ceiling]]: the halt side of redirect-before-halt.
  - [[concepts/refusal-cost-symmetry]]: false escalation on a recoverable
    twin.
  - [[concepts/evidence-gated-completion]]: evidence held, not acted on;
    verified claims unaudited.
  - [[concepts/permission-gate-as-architecture]]: blocked agents probe for
    a bypass after the evidence is in.

  Not 5: SWE repair rather than an ML-research loop, no redirect arm, and
  the monitor rests on one model.
- **For the held `gomez2026can` escalation field (`/implement`, held
  10-18).** The hold cited "single source, false-escalation cost
  unmeasured". This is a second source, and it measures that cost:
  pooled ≈0 on clean and transient tasks, 9/102 on the strongest model,
  triggered by an editor failure. Two design points follow if the field
  is ever proposed (our inference):
  1. Give the ticket schema explicit `progress` / `saved_changes` /
     `verification_status` fields. The missing progress here is most
     likely the missing field.
  2. Report the false-escalation rate on a transient twin, per model.

  It still does not separate the tool from the instruction.
- **For redirect-before-halt.** This bears only on the halt side. Hard
  caps did the stopping for 87–90% of Haiku and Kimi runs that had long
  since reached t*. That is vats2026good's "the ceilings did all the
  stopping" again, on a larger sample. An evidence-keyed trigger does fire
  (on 98.6% of runs, by construction), which is what a stall counter
  failed to do in cai2026forkandflush. Still no redirect-vs-halt arm.
- **Candidates** (cited, not yet in the graph):
  - Trinh et al. 2026, "HiL-Bench: Do agents know when to ask for help?"
    (arXiv 2604.09408).
  - Liu et al. 2026, "AgentAbstain: Do LLM agents know when not to act?"
    (arXiv 2607.10059).
  - Luo, Wen & Wang 2026, "Agentic abstention: Do agents know when to stop
    instead of act?" (arXiv 2606.28733).
  - METR & Redwood Research 2026, independent investigation of the
    OpenAI / Hugging Face hacking incident (metr.org, 2026-08-26). It is
    the field case that motivates the paper.
  - Zhong, Raghunathan & Carlini 2026, ImpossibleBench (ICLR 2026).
    Already listed under gomez2026can and zhang2026are.
