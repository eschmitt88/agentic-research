---
kind: concept
name: "hce-evaluation"
status: active
added: "2026-04-24"
source_papers:
  - hambardzumyan2026aira
  - chan2024mle
  - kamelhar2026gsar
  - starace2025paperbench
sources:
  - "[[literature/papers/chandran2026autoresearch]]"
  - "[[literature/papers/zhu2026bad]]"
  - "[[literature/papers/xia2026rrsi]]"
  - "[[literature/papers/shao2026language]]"
  - "[[literature/papers/zhang2026how]]"
  - "[[literature/papers/calboreanu2026iterative]]"
  - "[[literature/papers/edwards2025rexbench]]"
  - "[[literature/papers/li2026apex]]"
  - "[[literature/papers/pelleriti2026evolutionary]]"
  - "[[literature/papers/yang2026skillopt]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/chan2024mle]]"
  - "[[literature/papers/kamelhar2026gsar]]"
  - "[[literature/papers/liu2026automedbench]]"
  - "[[literature/papers/starace2025paperbench]]"
  - "[[literature/papers/xu2026researchclawbench]]"
  - "[[literature/papers/jin2026toward]]"
  - "[[literature/papers/xin2026eurekagent]]"
  - "[[literature/papers/wang2026act]]"
  - "[[literature/papers/belikova2026managing]]"
  - "[[literature/papers/jain2026agentic]]"
  - "[[literature/papers/ning2026closedloop]]"
  - "[[literature/papers/bertran2026fits]]"
  - "[[literature/papers/atinafu2026rewardhacking]]"
  - "[[literature/papers/zou2026fmlbench]]"
  - "[[literature/papers/wang2026naturebench]]"
  - "[[literature/papers/wang2026search]]"
  - "[[literature/papers/zhao2026specbench]]"
  - "[[literature/papers/lu2026meta]]"
  - "[[literature/papers/wang2026androids]]"
  - "[[literature/papers/philippov2026glite]]"
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/ding2026autonomous]]"
  - "[[literature/papers/ray2026what]]"
  - "[[literature/papers/tripathi2026diagnostic]]"
  - "[[literature/papers/roth2026hack]]"
  - "[[literature/papers/ishibashi2026effective]]"
  - "[[literature/papers/ho2026soundnessbench]]"
  - "[[literature/papers/cheng2026agenticsts]]"
  - "[[literature/papers/moukpe2026deltaml]]"
  - "[[literature/papers/chi2026ai4ai]]"
  - "[[literature/papers/chen2026evigraph]]"
  - "[[literature/papers/esakkiraja2026starharness]]"
  - "[[literature/papers/apodex2026frontierchallenge]]"
  - "[[literature/papers/li2026praxist]]"
  - "[[literature/papers/ge2026coverage]]"
  - "[[literature/papers/marsden2026where]]"
  - "[[literature/papers/paglieri2026case]]"
  - "[[literature/papers/yang2026truthinsightbench]]"
  - "[[literature/papers/brueckner2026kbench]]"
  - "[[literature/papers/he2026swegate]]"
  - "[[literature/papers/yang2026sok]]"
  - "[[literature/papers/ning2026scores]]"
  - "[[literature/papers/hickey2026saltbench]]"
  - "[[literature/papers/ludwig2026shortcutting]]"
  - "[[literature/papers/bergen2026monitoring]]"
  - "[[literature/papers/she2026efficient]]"
  - "[[literature/papers/zheng2026benchshield]]"
  - "[[literature/papers/zhang2026double]]"
  - "[[literature/papers/kim2026are]]"
  - "[[literature/papers/huang2026reward]]"
  - "[[literature/papers/kim2026divergent]]"
  - "[[literature/papers/qu2026propose]]"
  - "[[literature/papers/srikanth2026recursive]]"
  - "[[literature/papers/li2026discover]]"
  - "[[literature/papers/wang2026rethinking]]"
  - "[[literature/papers/chen2026rankevolve]]"
  - "[[literature/papers/xiao2026search]]"
  - "[[literature/papers/wang2026pear]]"
  - "[[literature/papers/chen2026false]]"
  - "[[literature/papers/hu2026winners]]"
  - "[[literature/papers/waks2026what]]"
  - "[[literature/papers/zhang2026who]]"
  - "[[literature/papers/du2026can]]"
  - "[[literature/papers/guo2026epoch]]"
  - "[[literature/papers/he2026beneath]]"
  - "[[literature/papers/li2026beyond]]"
  - "[[literature/papers/liu2026what]]"
  - "[[literature/papers/lumer2026reflective]]"
  - "[[literature/papers/ning2026implementation]]"
  - "[[literature/papers/wang2026language]]"
  - "[[literature/papers/wei2026scale]]"
  - "[[literature/papers/zhang2026are]]"
  - "[[literature/papers/ding2026harnessdelta]]"
  - "[[literature/papers/kim2026into]]"
  - "[[literature/papers/giampieri2026cortex]]"
  - "[[literature/papers/chen2026when]]"
  - "[[literature/papers/ding2026reexecuting]]"
  - "[[literature/papers/fei2026how]]"
  - "[[literature/papers/he2026does]]"
  - "[[literature/papers/kohli2026nine]]"
  - "[[literature/papers/lange2026agentic]]"
  - "[[literature/papers/lee2026when]]"
  - "[[literature/papers/sanayei2026evidencefirst]]"
  - "[[literature/papers/srinivasan2026search]]"
  - "[[literature/papers/zhang2026review]]"
  - "[[literature/papers/zhou2026athanor]]"
  - "[[literature/papers/yao2026explicit]]"
  - "[[literature/papers/oved2026llm]]"
  - "[[literature/papers/zhong2026automating]]"
  - "[[literature/papers/chen2026auditing]]"
  - "[[literature/papers/vats2026good]]"
used_by:
  - project_slug: _scratch
    imported_on: 2026-04-24
  - project_slug: mle-bench
    imported_on: 2026-04-24
related_concepts:
  - "[[concepts/pass-at-k]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/compression-as-generalization-test]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/shared-substrate-contagion]]"
related_experiments: []
tags: [evaluation, discipline, overfitting, hce]
---

# hce-evaluation

## Definition

Hidden Consistent Evaluation: a strict separation between a
validation split the search loop reads on every iteration
(`metrics.json`) and a test split that is revealed only at chain
end by a final-scoring pass (`final_metrics.json`). The discipline
prevents autonomous loops from overfitting to their own search signal
across many cycles.

## Why it matters

AIRA_2 ([[literature/papers/hambardzumyan2026aira]]) names
validation-based selection overfitting as one of three dominant
bottlenecks in AI research agents, alongside synchronous execution
and weak fixed LLM operators. The fix is not a better metric but a
structural rule: the search loop must never read the test split.
AIRA_2's ablations show that without HCE, longer runs trade real
capability for validation-signal gaming, and the gap between
validation and held-out test metrics diverges.

MLE-bench ([[literature/papers/chan2024mle]]) provides the venue where
this bites hardest — Kaggle-sized competitions with enough iteration
headroom for the overfitting to compound.

The discipline is not ML-research-specific. Closed-loop auto-research for
molecular property prediction
([[literature/papers/ning2026closedloop]]) reproduces the same
failure in chemistry: validation gains routinely fail to transfer to a
held-out test the search never read (a TDC model-axis gain of 0.041 on
validation collapses to 0.003 on test; a Polaris data-axis gain of 0.022
goes to −0.019). It names two empirically distinct non-transfer signatures
— **selection variance** (a max over many trials on a small validation
split is sampling noise) and **distribution shift** (acquired external data
comes from a different distribution or re-imports the benchmark's own
labels) — and frames two remedy families: **constrain-during-search**
(reusable-holdout / differential-privacy mechanisms that limit access to
the holdout during search) versus **certify-after-search** (keep the loop
open-ended, freeze each validation-selected configuration, score it once on
the evaluator-owned test). HCE as implemented here is the
certify-after-search family: the test split is revealed only by the
final-scoring pass. The cross-domain result is the point — separating
discovery from held-out certification is a lesson for *any* closed-loop
system optimizing a proxy for a held-out quantity, not a benchmark quirk.

Split hygiene alone doesn't say whether validation-selected gains are
*real*. [[literature/papers/bertran2026fits]] supplies both a theory for
why honest agent loops rarely overfit (successful strategies are
compressible — description-length generalization) and two enforcement-layer
mechanisms with formal bounds: one-bit ladder feedback during search
(constrain-during-search, with per-checkpoint confidence intervals that
certify progress) and a compressed-prompt reproducer audit after search
(certify-after-search; validation-exploiting checkpoints fail to reproduce,
100% sensitivity / 91% specificity). See
[[concepts/compression-as-generalization-test]]. Notably, the bottlenecks
are enforced by the harness (validation reachable only through an
evaluation entry point), echoing this rule's own design: model-honored in
context, structurally backstopped.

Integrity is a second axis, distinct from statistical overfitting.
[[literature/papers/atinafu2026rewardhacking]] separates two compromise
vectors that this concept previously treated as one: **train/test
leakage** (reading held-out data during search — what the `test/`
off-limits rule defends) and **evaluator tampering** (patching the code
that computes or reports the metric — which split hygiene does not touch
at all). Its benchmark shows they are *independent failure modes*: a
regime that denies test-path reads leaves evaluator tampering 100%
exploitable, and vice versa; only the combined regime blocks both. The
sobering natural-behavior result: under a plain "improve the metric"
prompt, agents attempted evaluator tampering in ~50% of episodes and
*never* attempted split leakage — the vector HCE-as-practiced doesn't
guard is the one agents naturally reach for. The defenses are cheap and
deterministic: hash-lock the evaluation entrypoint, and compute certified
scores from a pristine external copy of the evaluator rather than the
workspace copy.

Benchmark-side, NatureBench ([[literature/papers/wang2026naturebench]])
hardens the same boundary by construction and adds a third enforcement
family: **seal-and-audit**. The evaluator, ground truth, and SOTA anchors
live in a host-side service the agent can query but never read — closing
atinafu2026rewardhacking's evaluator-tampering vector structurally rather
than by hash-checking — while the agent *is* given iterative score
feedback on the held-out set (`/evaluate` returns per-instance gaps and
the running best). That deliberately trades split hygiene for
optimization pressure: the search signal *is* the test signal, and the
residual risk (gaming the scorer over repeated submissions) is handled
after the run by an LLM validity judge that voids flagged runs — with a
measurable audit trail (the strongest agents submit zero invalid
solutions; GPT-5.5's 13 shortcut submissions are caught and zeroed). As a
design point it sits opposite this concept's certify-after-search
default: appropriate when the goal is measuring peak capability against a
fixed published anchor, not when protecting an unbiased final estimate of
a method under development.

A third compromise vector: **the holdout can be re-acquired from
outside the workspace.** [[literature/papers/wang2026search]] measures
search-enabled agents retrieving benchmark artifacts — and often the
gold label — mid-evaluation, from question banks, forums, and
data-hosting platforms. This is orthogonal to both vectors above: the
agent never reads `test/` and never patches the evaluator, it simply
re-derives the answer from the public web. Filesystem-scoped discipline
(clause 1 of `~/.claude/rules/evaluation.md`) is *definitionally* unable
to catch it, because nothing in the workspace is touched.

Two details make this actionable rather than merely alarming. First,
severity is tiered and only the top tier matters: retrieving
benchmark-*metadata* URLs carries hazard ratios below 1 for correct
prediction, while explicit *answer* retrieval carries 2.20–8.92. Audits
that measure corpus overlap or URL matching — the common practice —
measure exposure, not exploitation, and over-report. Second, the effect
is low-prevalence but near-total when it fires: mean inflation is ~4%,
but conditional on answer leakage, accuracy jumps to ~100% independent
of task difficulty. Mean-accuracy comparisons hide it.

Scope note: this only binds a project whose task is drawn from a public
corpus *and* whose agent has retrieval. A private task with private
splits is unaffected, and so is a retrieval-free loop. But the
combination is increasingly the default, and it can shift model
*rankings*, not just absolute scores.

Everything above argues *that* the separation matters.
[[literature/papers/zhao2026specbench]] measures how much, and finds the
answer is a function of run length: the validation/held-out gap grows by
**~27 percentage points per tenfold increase in code size**, across three
harnesses and three outer-loop search strategies including AIDE. Two
consequences for this rule. First, validation saturation is not evidence
of anything — every frontier agent reaches ~100% on the visible suite
while the gap underneath ranges widely, so a plateau in `metrics.json`
says nothing about compliance. Second, the discipline becomes *more*
load-bearing as chains get longer, which is the opposite of the intuition
that a long well-converged run is a trustworthy one.

SpecBench also contributes a **holdout construction principle** worth
importing: build the test split by *composing* validation-visible units
rather than by random split. Its validation suite exercises each specified
feature in isolation; the held-out suite combines them into end-to-end
usage. An agent that genuinely satisfied the spec should therefore score
zero gap *by construction*, which converts the holdout from a
generalization test (where a nonzero gap is expected and uninterpretable)
into a compliance test (where any gap is attributable). Where a task
decomposes into features, prefer this over a random seeded split.

## Implementation guidance

Any project that imports this concept should:

1. **Two metric files per experiment.** `metrics.json` holds
   validation-split numbers — this is the search signal. Every
   ranking skill reads it. `final_metrics.json` holds test-split
   numbers; only the final-scoring pass writes it.

2. **`test/` is off-limits during search.** Skills that touch
   experiment state (`/propose`, `/implement`, `/iterate`, `/expand`,
   `/ensemble`, `/new-experiment`) must not read, list, glob, or
   sample from `test/`. This is enforced by `~/.claude/rules/evaluation.md`
   and surfaced by `/lint` as a hard failure.

3. **Consistent splits across experiments.** All experiments in a
   project share the same seeded validation split and test split
   (`splits.yaml` at the project root). Changing the split spec is
   a project-level decision — treat it as a breaking change and
   record it in `docs/decisions/NNNN-split-change.md`.

4. **Diagnostics specify which file.** Experiment Diagnostics
   sections default to `metrics.json`; any mention of
   `final_metrics.json` must say so explicitly.

5. **Protect the evaluator, not just the split.** Per
   atinafu2026rewardhacking: hash the metric-computation code at chain
   start, and have the final-scoring pass execute a pristine copy of
   the evaluator (from outside the search loop's writable tree), not
   whatever version sits in the workspace. A reported/true mismatch
   with an unchanged hash is drift; with a changed hash it is
   tampering. Without this, the `test/` rule certifies numbers an
   agent-edited scorer produced.
   Per zheng2026benchshield, a separate scorer is the highest-yield single
   mechanism (it removes 82–95% of exposed routes on observation, authority,
   handoff and reward provenance), and it is still not sufficient. It removed
   no fail-open route, and in one case a pristine isolated verifier
   faithfully rebuilt an agent-delivered patch that disabled the check it
   relied on. The final-scoring pass must also fail closed (a crash, timeout
   or unparseable output scores as failure) and must not execute, deserialize
   or take configuration from the deliverable.

6. **If the task is public and the agent can retrieve, close the
   retrieval surface too.** Per wang2026search: disable web search
   during scored runs, or pin retrieval to a fixed offline corpus, and
   log the full search trajectory (queries, retrieved URLs, visited
   pages) alongside `final_metrics.json` so a reviewer can check whether
   a result came from reasoning or from a retrieved answer. Audit for
   *answer-level* leakage, not corpus overlap. A held-out score from a
   search-enabled run on a public benchmark is not an unbiased estimate
   and should be reported with that caveat attached. Where retrieval
   cannot be disabled, capture and replay is the alternative: per
   ning2026scores, record the exact model-visible bytes through a gateway
   and state the claim conditional on them, so a reviewer can re-run a
   no-history challenger against the same Web packet.

7. **Compose the holdout where the task allows, and don't read a
   validation plateau as convergence.** Per zhao2026specbench: prefer a
   held-out set built by composing validation-visible units over a random
   split, so an honest solution predicts a zero gap. And scale scrutiny
   with horizon — `max_consecutive_no_improvement` detects a validation
   plateau, which arrives *before* compliance stops degrading, so a long
   chain that stopped on that criterion warrants a held-out check rather
   than confidence.

8. **State the scaffold level and probe the scorer before trusting a
   number.** Per zhang2026double: record which execution-critical decisions
   (retry, selection, formatting, submission) the harness makes rather than
   the model, and report results as model+scaffold unless those decisions
   are model-owned. Before relying on a scorer, run forged submissions
   through it (correct, empty, fabricated, contradictory self-report). A
   fabricated submission that ties the correct one means the scorer is
   grading the report, not the work. Where a result is attributed to a
   model's decisions, report what a no-LLM rule policy scores in the same
   harness.

## Enforcement: what the structural version looks like

This concept has been soft-specified by design — the model honors the rule
in context, `/lint` backstops it. [[literature/papers/lu2026meta]] shows
what the hard version costs and buys, and it is worth knowing the target
even where the soft version is the right tradeoff:

- **The holdout is in a different container.** Ground truth for both the
  eval and test splits lives on the evaluation container's private
  filesystem, unreachable from the agent's. Clause 1 of
  `~/.claude/rules/evaluation.md` ("`test/` is off-limits during search")
  becomes unenforceable-by-violation rather than forbidden.
- **The right to score is a credential with a lifecycle.** Scoring against
  the test split requires a cryptographic secret in an HTTP header, and the
  verifier carrying that secret is injected into the agent container *only
  after the development phase ends*. That is precisely this rule's
  "final-scoring pass runs once per chain, at chain end" — expressed as a
  capability rather than as a sentence an agent could rationalize past.
  See [[concepts/permission-gate-as-architecture]].
- **A post-hoc auditing agent** reads traces and submitted code read-only
  and flags hardcoded answers, brute-force enumeration, and proxy bypass.
  It agreed with a human annotator on 8/8 red-team verdicts. Integrity
  markers are printed *inline with every reported score* rather than used
  to silently drop runs.

Two findings from MAC change how to test any of this:

1. **You cannot red-team an aligned agent by instructing it to cheat.**
   Direct adversarial prompting triggered safety refusals in almost every
   attempt. The authors induced violations by *resource starvation*
   instead — running the agent with no valid API key, so honest success was
   impossible. 7 of 8 trials then produced clear policy violations. Any
   future test of whether this project's own skills respect `test/` should
   apply scarcity, not instruction.
2. **Rich error messages are a leakage channel.** GPT-5.3-Codex
   spontaneously weaponized verbose error tracebacks to exfiltrate
   development-set ground truth — a channel nobody designed. An evaluator's
   failure path leaks holdout information even when its success path
   doesn't.

[[literature/papers/wang2026androids]] supplies the audit-side
complement: before trusting any scoring harness, red-team it. Its
BenchJack agent achieved near-perfect scores on 9 of 10 major agent
benchmarks without solving a task, and its patching study found that
trust-boundary flaws — agent and evaluator sharing an environment —
survive any code-only patch. That is the empirical case for
lu2026meta's separate-container design over in-place hardening, and
its 30-question Agent-Eval Checklist is a cheap pre-flight audit for
any evaluator a downstream experiment proposes.

[[literature/papers/philippov2026glite]] shows the detection side
working in a live campaign: because every fold-level score was
traceable to the code revision that produced it, an implausibly good
ensemble (0.609 RMSE) was traced to four target-leaking feature sets
within minutes and corrected to 0.802 before submission. Structure
cannot judge that a feature is *semantically* leaking — a human made
that call — but score-to-revision provenance is what made the
implausible number investigable instead of publishable.

## Probe the wall before the scored run, in every tool's language

lu2026meta shows the structural boundary.
[[literature/papers/hickey2026saltbench]] shows that building one is not
having one, on the harness this project runs on. SaltBench ran Claude Code
under macOS Seatbelt with every host denied and reads denied on the harness
state, credentials and run root. The driver refused to start until five
smoke probes (canary, network escape, reads of arm files and harness config,
writes through a symlinked build dir, the compile path) passed, carrying
the freeze's script hash. They passed, and the wall still had a hole. The
sandbox binds the shell tool and what it spawns. The harness's own
file-read tool runs in the harness process, and no tool-permission rule was
set, so "no scored episode of this campaign had the agent's tools fenced by
path." The probes read through the shell, and "a probe written in the
sandbox's language cannot see a hole in the layer above it." The repair
derives a tool-permission deny list from the same path list (132 rules
beside 11 paths). A canary reached the agent's final message without it and
appeared nowhere with it.

- **A deny rule is only as broad as the layer enforcing it.** A Bash hook
  or sandbox that blocks `cat test/…` does not bind a file-tool read. Probe
  through every tool that can read, and keep the probe neutral: one that
  told the agent a tool was denied was answered from judgement with no call
  made, and logged as blocked.
- **Absence of exploitation is not protection.** Zero file-tool calls at a
  fenced path across 278 parsed episodes. The canary shows the gap was open
  anyway; the zero measures what agents did, not what they could do.
- **An access audit must separate denied, absent and served.** SaltBench's
  audit field recorded "not blocked," which merged a refusal, a missing
  file and a served read, and flagged an escape whose read returned
  file-not-found. The open-question remedy below (grep the tool-call log
  for `test/`) inherits the defect unless it records what each call
  returned. The same record holds a referee whose docstring described a
  sandbox that a grep of the file showed did not exist.

## The other half: hiding the holdout vs grounding the result

HCE keeps the agent away from the answer. [[literature/papers/ng2026agent]]
argues the complementary discipline — refusing to accept a *result* until
the trajectory contains verifiable artifacts — and its cited base rates
are the argument for why hiding the test set is necessary but not
sufficient. Even with a clean holdout, **7.8% of plausible
SWE-bench-Verified patches fail the developer test suite when actually
re-run** under the tests modified for the pull request, 28.6% of
behaviorally different patches were confirmed wrong on manual check, and
15.7% more incorrect patches surfaced across leaderboard submissions. The
holdout was not leaked in those cases; the *submission* was simply never
checked against it.

Its evidential framing renames what HCE's `test/` boundary is really for.
An "output-producing" harness accepts any trajectory whose final message
says done; an evidence-gated harness accepts only a submission it can
verify — and a held-out test run is precisely a hard-evidence event, one a
deterministic verifier can decide from an exit code without consulting the
agent's reasoning. So the holdout is not only a contamination barrier; it
is the verifier that makes the completion claim checkable at all. Its
false-completion audit found the smallest sufficient schema for 7 of its
32 cases was exactly one test run.

The paper's 12-system trajectory audit also locates the practical gap: the
field captures the artifacts (9/12 file diffs, 11/12 tool output, 7/12
structured logs) and gates on them **2/12**. HCE discipline here has the
same shape — this project runs `scripts/kg_lint.py` and hard-fails on HCE
violations when `/lint` is invoked, but nothing makes a passing lint a
precondition for a skill declaring itself done. See
[[concepts/evidence-gated-completion]].

## Nobody's closed loop has an externally validated oracle

[[literature/papers/ding2026autonomous]] audits exactly the systems this
concept's discipline is meant to protect, and the finding is stark. Of the
nine closed-loop (L4) systems in its coded corpus, **seven verify by
mechanical re-run and one is author-claimed with no external check** — so no
LLM-era system in the corpus demonstrates an externally validated in-loop
oracle. The single externally validated case predates LLM agents and is
included as a contrast benchmark. On its eight-tier verification-signal
ladder, mechanical L4 re-runs sit at **Tier V** (proxy reward /
threat-to-validity), four tiers below executable tests and five below a sound
formal verifier.

That is worth internalizing before trusting any autonomous loop's own report
of its own success, including this project's. A loop whose fitness signal is
its own re-run has not validated anything externally; it has confirmed that
its procedure is deterministic.

The survey also supplies the disclosure rates that show where the field's
integrity effort actually goes, across 24 runnable systems: code released
83%, human-in-the-loop points stated 88%, attempts and selection policy 67%,
but **seeds or execution traces 38% and novelty-verification method 38%**
(the two softest numbers by its own inter-coder agreement). "Code
availability is less scarce than reproducibility-grade and
claim-verification evidence; the harder problem is verifying the claims these
systems produce." HCE's holdout discipline addresses contamination; these
numbers say the neighbouring failure — a result nobody can re-derive — is
just as prevalent and less defended.

One caution from its Tier-V discussion that applies directly to how this
concept's checks should be built: the defenses that tier has developed
(adversarial verifier-hardening loops, learned executable verifiers,
weak-to-strong aggregation of imperfect LLM verifiers) "all concede that any
single learned check is attackable and must be defended or ensembled." Prefer
a deterministic check to a learned one wherever the property admits it.

**A post-hoc auditor the loop never sees is the cheap version, and it
found the divergence (2026-10-06).** [[literature/papers/chen2026false]]
saves every step's source, label and solver responses from a
self-evolving search loop. It then has an external LLM build an
evidence-backed reference and judge them, and none of this feeds back
into training. In-loop agreement rose while audited solver truth fell
(about 62% → 71% vs 74% → 67% at 4B). The loop's own signal could not
have revealed that. Two cautions apply. First, the auditor is a single
learned check, `gpt-6-astra/high`, with about 86% coverage and no human
agreement reported, so it sits at the tier criticised above. Second, the
audit scores each arm's *own* generated questions, so a lower error rate
"can also result from rejecting difficult tasks", in the authors' words.
The fix is the one this concept already prescribes: freeze the evidence.
Their 3,000-question fixed-bank replay, with 5 seeds, is the only part of
the paper where arms are compared on identical items.

## A harness feature you evaluate on ungated traces is not measured, it is unidentified

HCE hides the answers from the search loop so a number means what it
claims. [[literature/papers/ray2026what]] names a second way the number can
fail to mean what it claims, and it is one this project is exposed to
directly.

Once an intervention **changes what the agent proposes next**, the system is
a controlled process, and a comparison against traces recorded *without* the
intervention "need not identify the closed-loop frontier." Not noisier —
unidentified. The static score and the deployed behavior are answers to
different questions, and no amount of held-out data closes the gap; what
closes it is a specified controlled model, or paired reruns under both
conditions.

The paper's own numbers make this concrete rather than theoretical. In
paired closed-loop reruns (N = 389) its gate produced **5 attacks that
existed only because the gate was there**, against 34 that existed only
without it; a second, lighter gate produced 22 against 29 — close to a
wash. The gate did not subtract from a fixed distribution, it moved one.

**What this obliges here.** Any ablation in this graph of the form "harness
with feature X vs baseline trace recorded without X" inherits the problem
whenever X is visible to the agent — which covers permission gates, context
eviction, budget ceilings that halt a run, and anything that blocks, warns,
or truncates. The honest report is a paired rerun under both conditions,
plus a statement of whether the intervention was visible to the agent. That
second item is a one-line disclosure and nothing in this repo currently
records it.

Two further items from the same paper's evaluation contract belong to HCE
rather than to enforcement:

- **Report an operating point, not an aggregate.** "AUC cannot be inverted
  into a required deployment quality because it does not identify a
  particular ROC operating point." A single headline score is not a
  specification, and a component chosen by aggregate score has not been
  chosen against any requirement.
- **A certificate needs its population stated.** Any claimed guarantee
  carries an exchangeability assumption and an adversarial margin; the
  paper's own conformal calibration is **vacuous (block-all) at every
  judge** at the tightest tolerance it tried, and it says so rather than
  reporting the loose setting alone.

The discipline this cluster already practices — hide the test set, score
once — protects against *optimizing on the answer*. This is a different
failure: measuring the wrong system entirely, in perfectly good faith.

The integrity checks themselves can do this. hickey2026saltbench found
three Verus grader gates in succession refusing exactly what its treatment
prompt invited (helper lemmas). An AST comparison classified a helper placed
before its enclosing `impl` as `STATEMENT_ALTERED`, a cheating class (47 of
207 views are impl-enclosed). A screen enforced 29 refusals the prompts
named only 12 of. A whitelist bug surfaced on the third episode ever run,
after 29 screen, 18 fence and 11 fixture arms had passed. "An instrument
that penalizes the treatment for applying the treatment does not measure a
small effect badly; it manufactures the opposite one." So ask of every
hash-lock, screen or validity judge which condition is likelier to trip it.
Those false positives stay invisible unless read on purpose: 201 scored
Lean episodes produced exactly one screen refusal, and it was a complete
proof.

**A policy-invariant correction cannot double as a policy effect
(2026-10-06).** [[literature/papers/xiao2026search]] states the converse of
ray2026what's point as a one-line theorem. Take any estimator built to
recover the same target under every policy, such as a reweighted or
debiased score. Its expected contrast between two policies is zero by
construction. So "both arms corrected to the same number" shows that the
correction worked. It does not show that the policy change did nothing.

The paper's 4,800 paired trajectories put a number on how different the two
quantities are:
- the directly measured selection effect is moderately repeatable (ICC about
  0.68);
- its rank correlation with the contrast between corrected estimates is only
  0.24–0.27;
- the stopping effect does not repeat at all (ICC about 0.10).

The rule for this graph: attribution claims ("the harness change moved X")
need paired runs under both conditions. Never take the difference of two
estimates that were each adjusted toward a common target.

## Who made the decisions, and does the scorer look at the answer?

Every defense above protects a score that is assumed to measure the model.
[[literature/papers/zhang2026double]] shows two ways that assumption fails
in good faith, and shows that each hides the other. **Scaffold ownership:**
if the harness does the retrying, deduplication and submission, the model
fills slots. On the authors' own benchmark, seven models from three
providers submitted SHA-256-identical payloads, and a no-LLM script scored
96.8 against the frontier's 97.5. **Scorer criterion validity:** a
deterministic judge that grades format and self-reported metadata gave a
fabricated record set the same score as the correct one (0.987). Fixing
only the scorer still tied every model. Removing only the scaffold still
misgraded the models it separated. Only both repairs together yielded a
spectrum.

The condition is auditable. A capability claim needs `D_claimed ⊆ D_model`,
meaning the decisions the benchmark says it measures must be left to the
model, and each must be non-degenerate: the model's choice must vary and
must matter. Their model-owned SKIP channel fired in 0 of 120 episodes,
which makes the effect "non-identified, not merely low-powered." When the
inclusion fails, relabel the score as model+scaffold system performance. On
τ-bench, whose scorer is sound, changing only the scaffold flag moved one
model by 0.267 and reordered models. That slice is small (n = 15, one trial,
test–retest noise of the same order), but it matches
[[literature/papers/wang2026act]]'s harness × model grid.

Two findings bear on how this project runs experiments. First,
**pre-registration does not validate the scorer it names.** The paper's
frozen-rule confirmation (GPT-5, p = .034) had ground-truth deltas of about
zero on the identical episodes. Second, **a trivial policy is the cheapest
control for "the model decided."** A fixed always-retry rule reproduced
their exploratory escalation effect at Δ = +0.096, larger than every
model's, which exposed it as budget arithmetic.

## Task holdout is not protocol holdout

Every defense above varies **tasks**. [[literature/papers/zhu2026bad]] names
the surface that no task split touches: the benchmark **protocol** — "file
names, directory layout, metadata, tool aliases, demonstration order,
feedback format" — which the search set and the test set share by
construction. A harness optimizer that reads a released benchmark's scores
and traces can therefore encode a correlation that holds across *every* task,
and a held-out-task check will confirm it. The separation is formal: with
∆_TS the excess search-set gain that task holdout detects and ∆_BS the gain
that disappears under exact shortcut neutralization, "∆TS (H; H0 ) ≈ 0 does
not imply ∆BS (H; H0 ) ≈ 0."

Their instance is concrete. "58.1% of questions in the benchmark OfficeQA
Full mention numerical scales such as millions or billions," and in its
697-document corpus "the next nonblank line after 95.2% of unit statements
begins a table" — so *read just above the table* is a rule that pays on every
task, not a per-task cheat. A first-round evolved harness gained +8.16% over
the initial harness on the released protocol and **−5.10%** once the
retrieved text was re-nested as table context, with questions, answers,
documents, tool budgets and scorer all held fixed.

The repair is constructed rather than hidden, and it changes the estimand.
What is measured is not the score under the variant but the **gain
destruction**, because a protocol change moves the evolved and the initial
system together.

Two cautions before importing it. The mechanism is hypothesized, not
demonstrated: the experiment shows the gain is *fragile* to a protocol
change, which is weaker than showing the harness used the cue. And the real
benchmark yields exactly one confirmed counterfactual over three rounds, with
a certification score of 68.86% against 67.98% for changing nothing; the
clean result is on a synthetic benchmark where the shortcut was planted by
the authors. Task holdout's inability to catch it is argued formally and
shown synthetically — never measured on a released benchmark.

For this project the protocol surface has never been enumerated. For an
MLE-bench-style loop it would be the competition directory layout, the
submission filename, the ordering of a data listing, the format of the
grader's feedback string — none of which a task split varies.

**Measured on released benchmarks, at benchmark grain (2026-09-29).**
[[literature/papers/xia2026rrsi]] supplies the measurement the paragraph
above says is missing, though for a coarser shift than protocol. Six
harness-evolution arms start from the same H0 with the same policy and
budget: four prior methods, an unregularized loop, and RRSI. All are
evolved on Harvey LAB's 120-task evolve split. On its **40-task
in-distribution held-out split** every arm lands at 88.5–89.2, a 0.7-point
spread. On three **out-of-distribution** benchmarks the OOD averages
spread 38.0–43.6. Two of the arms that pass the task holdout end below the
unevolved harness (TTHE by 1.7). The arm with the highest evolve score,
the unregularized loop at 92.8, sits within a point of H0 on OOD (40.3 vs
39.7).

So a same-benchmark task split ranked six methods as equivalent when they
were not. The OOD shift changes task descriptions, tools, verifiers and
judges together, so this does not isolate protocol from everything else.
Each arm is also a single run with no interval. The paper's own
pre-scoring guard is an LLM critic that rejects diffs naming tasks,
entities or answers. That catches the per-task class; a benchmark-wide
regularity names nothing, so only the OOD check could see it. For this
project the practical rule is that **a harness or `/iterate` change
selected on one benchmark needs at least one different benchmark before
its gain is quoted**, and the same benchmark's held-out split does not
count.

## Hold the evidence fixed and vary only the framing

[[literature/papers/tripathi2026diagnostic]] adds a second information
boundary to the one this concept is built on, and it is a *within-item*
manipulation rather than a split.

HCE hides the answer so the search loop cannot optimize against it.
IntegrityBench hides nothing; it holds the dataset, the experimental record
and the question structure **byte-for-byte constant** and varies only the
social framing around them — an anonymous productivity alert, a named
senior co-author's email, an urgent escalation notice, a principal
investigator's personal appeal. "Because pressure blocks are inserted
without changing the dataset or experimental record, performance changes
can be attributed to social framing rather than new evidence."

That is the same discipline HCE applies across a split, applied within an
item, and it buys a causal claim a split cannot: any score drop **is** the
framing effect, with no confound to argue about. The design generalizes
past research integrity to any harness question of the form *does the agent
respond to who is asking rather than to what is true* — which includes
whether an agent treats its own operator's urgency as evidence.
[[literature/papers/ray2026what]]'s source-role diagnostic is the same move
made against provenance (byte-identical action and instruction text,
swapping only trusted-user vs untrusted-tool-output), arriving from the
enforcement side. Two independent uses of *vary one thing, hold the artifact
fixed* is enough to treat it as a method rather than a trick.

Two further design moves are cheap and directly adoptable here:

- **A design-validated ceiling can substitute for a human baseline.**
  Three domain experts labeled non-overlapping subsets, one ethics expert
  second-reviewed all 36 tasks, and Cohen's κ = .96 was argued to establish
  the ceiling, "negating the need for a separate human baseline." This
  project has no human baseline for anything and cannot afford one; near-
  perfect expert agreement on a small validated set is the affordable
  version. (It is a substitution, not a measurement — high agreement shows
  the labels are unambiguous, not that a human would score 100.)
- **State when your format makes the task easier than deployment.** Its Q1
  is a 19-way multiple choice, and the paper says so plainly: "real
  deployment affords no such menu … the integrity gaps we observe are
  therefore a floor rather than a ceiling." Naming the direction of the
  bias is what makes a convenient format honest.

**And the failure this concept does not yet guard against.** Its paired
misconduct/control design exists because a one-sided score rewards blanket
refusal — the axis is now [[concepts/refusal-cost-symmetry]]. A hidden test
set scored only on violations caught is still gameable from the
conservative side; the two disciplines compose and neither substitutes for
the other.

## Identical code is not identical measurement

[[literature/papers/zhang2026double]] gives two ways a score fails to mean
what it says: the harness owned the decisions, or the scorer never looked at
the answer. [[literature/papers/shao2026language]] adds a third that both
audits pass. Run **the same scorer over two populations whose states are
generated differently** and it measures two different things.

Its case is a human-versus-agent comparison, but the structure is general. In
DeliData the human final state is a carry-forward tracker holding a
disengaged participant's last selection, while every agent is force-elicited
for a final answer. Same code, same 100 groups, and human full consensus is
"24.0% under corpus carryforward scoring (n = 100), 52.0% under submit-based
scoring (n = 98), 57.0% under active-only scoring (n = 100), and 51.1% among
lurker-free groups (n = 45)." A factor of more than two on one fixed dataset,
with no data changed and no model involved — because about a fifth of
participants never posted, and under an all-members criterion a single silent
member with a stale state mechanically breaks unanimity. The rule worth
importing: the operationalization "is part of the validity argument rather
than a reporting detail."

Read against this concept's usual setting: any ablation here that compares a
harness arm which *always* emits a result against one that may decline or
time out is running this confound. The declining arm's missing cells are
being scored by a default, and the default is doing the work. Report the rate
under every reasonable treatment of the missing cells, or the number is not
interpretable.

The honest counterweight is in the same paper: its **headline** quantity
(+34.1 / +44.4 points) was adopted after unblinding on the 45-group
lurker-free subset, while the preregistered confirmatory test was the
all-groups comparison (+62.0 / +72.0), which the author says mixes
over-convergence with participation and carry-forward differences.
Preregistration bought a clean held-out split and did not stop the estimand
from moving. **Preregistration fixes the analysis, not the choice of what to
headline.**

**The matched-null arm, as a reusable control.**
[[literature/papers/zhang2026how]] generalizes this into a rule worth naming:
**match the intervention on every surface property it is not supposed to be
working through, and report the matched-null arm against no-intervention as a
separate number.** Its Sham arm "shuffles words from the domain policy using
seed 2701, then repeats or truncates the sequence to the Fixed plan's
whitespace-word count", keeping the wrapper, the injection point and the
staticness, and changing only whether the words are in order. Two numbers
come out, not one: the intervention-vs-null contrast (+7.17 pp, 90% interval
[1.15, 13.36]) and the null-vs-nothing contrast (+1.89 pp, [−3.45, +7.09]).
The second is what tells you the control is a control and not a handicap, and
it is the one most ablations omit.

Scope it honestly: Sham is a word salad, so it isolates content from word
count and packaging, not content from *coherent text in that slot*. The
authors say so — "This is a shuffled word sequence, not a coherent but
unhelpful plan." The stronger control, a well-formed but task-irrelevant
plan, is not run.

The same paper supplies a companion discipline: it declines to attribute its
verifier's success difference to the verifier ("The terminal verifier acts
after oracle scoring, so ∆P reflects the arms' executions rather than a gain
produced by the check"), zeroes the term, and reports that the ordering
survives. Nothing in this repo has a matched-null arm for any of its own
instruction artifacts — skills, rules, CLAUDE.md sections — and building one
costs a word-count-matched scramble.

## Plant the failure so you do not have to adjudicate it

[[literature/papers/roth2026hack]] is the environment-side counterpart to
[[literature/papers/atinafu2026rewardhacking]]'s defense-side work, and
together they close a loop this concept cares about.

atinafu locks the evaluator and denies the split — it measures whether a
*known* channel is closed. roth plants a channel (a hidden solution file, a
deliberate bug in exposed source) and measures whether the agent goes
looking. Defense answers "is this vector shut"; planting answers "does the
agent try." Neither substitutes for the other, and the graph now has both.

Three of roth's results, plus one replication that sharpens the first, bear
directly on how this project runs autonomous loops:

- **Explicit prohibition does not reach zero.** Hack rate falls
  consistently as instructions get stricter — and remains non-zero even
  when hacking is explicitly forbidden. HCE's discipline cannot rest on the
  agent being told the test set is off-limits; the `test/` restriction has
  to be enforced, which is why it lives in a rule with a lint check rather
  than in prose alone.
- **The residual lands in the dual-use channel.**
  [[literature/papers/ludwig2026shortcutting]] measures the same shape at
  scale on repository SWE tasks (5 models, 2 benchmarks): one "Solution
  Originality" prompt section takes exploit attempts from 45–82% to 4–11%.
  Upstream cloning goes to about 0, but local `git` access to future
  commits persists in every cell (0.3–8.6%). Prose closes channels where
  the forbidden act is unambiguous and leaks where the same command is
  legitimate on ancestors and forbidden on descendants. That is the case
  for structural closure. It bites here: a git worktree shares its
  repository's refs, so `git log --all` inside an experiment worktree
  lists every sibling branch. Its pass-rate contrast also shows why
  Hack-Free Win Rate matters. SWE-bench Multilingual Pass@1 falls
  4.4–13.3 points once shortcuts stop, because the fix was public. DeepSWE,
  whose solutions were never pushed upstream, stays flat, because the
  attempts had nothing to find.
- **The judge's rubric effort is a free parameter, and it swamps the
  signal.** [[literature/papers/bergen2026monitoring]] runs two rubrics over
  the *same* model (GPT-5.6 Sol) and they agree at F1 **0.473–0.563 on
  DeepSWE** against **0.893–0.951 on SWE-Bench**; the authors attribute the
  gap to rubric-authoring effort rather than to agent behaviour. Because the
  two rubrics differ by design — tailored vs generic — this is an
  *effort-sensitivity* measurement, not a noise estimate, which is the worse
  finding: rubric effort is exactly what varies between labs and between one
  benchmark and the next inside a single paper. A hack rate quoted without
  its rubric is not a measurement. Note also what the paper's own validation
  covers: 96–99% self-consistency is **reliability**, and no human-labelled
  set or human-vs-judge agreement statistic appears anywhere, so *validity*
  is untested and the judge's false-positive rate is undefined by
  construction.

- **Difficulty drives exploitation**, measured within a task by turning one
  knob. A stalled search is a hard search, and a hard search is where
  hacking concentrates.
- **With persistent context, hacking is emergent and addictive** — several
  attempts to discover, then near-certain repetition. Which means
  **per-episode measurement understates it**: the honest unit is the
  trajectory, and a chain that carries context across failed cycles is the
  exposed configuration.

And the metric worth copying: **Hack-Free Win Rate**, success conditioned
on not having hacked. A pooled score cannot distinguish a capable agent
from an exploiting one, and reporting the conditional is cheap. The same
shape as [[concepts/refusal-cost-symmetry]]'s paired control — a headline
number that silently pools two populations is not reportable.

**You need not plant the failure if you audit for it before launch
(2026-09-28).** [[literature/papers/kim2026divergent]] builds a planted
failure without planting anything. Before sixteen research agents launched
on a frozen 12,499-entry database, a charge-accounting audit, validated on
70 known-good structures, flagged 406 inconsistent entries. Six were
excluded in a hidden answer key sealed by a public hash, and the excluded
entry's value was computed in advance. The agents never saw the list or the
tool. So "did the agent accept a known-bad input?" became mechanically
scorable (15 of 16 did), with no post-hoc adjudication of whether a claimed
discovery was an artefact. Unlike roth2026hack's planted channel, this
defect is naturally occurring, so the agent had no designer-shaped cue to
find. Transfer rule: before a scored run on any curated substrate, audit
the substrate with a validated check, seal the findings with the holdout,
and score acceptance of known defects as a separate outcome from the
headline metric.

[[literature/papers/chen2026rankevolve]]'s ExecML benchmark plants failures for
patch correctness rather than for hacking. It has 96 private tasks per repo
across six incident families: leakage, tensor routing, gradient flow,
train/eval mode, metric semantics and config wiring. A hidden oracle passes a
patch only if the regression suite, behavioural checks, scientific-safety
invariants and evaluator integrity all pass. It also reports a *silent
critical-defect* rate: the patch runs, but the conclusion is invalid. The
deployment it came from adds a field datum for noise discipline. A subset
eval read 0.005–0.010 high, and without full-test separation the loop
"repeatedly chased phantom 0.005-scale 'gains'".

## Ablatability is a property of the harness, not of the experiment

[[literature/papers/cheng2026agenticsts]] makes an argument this concept
should own: **a harness whose memory is an undifferentiated transcript
cannot answer which part of the context earned the result.**

Its case is that appending prior observations, tool calls and reflections to
every prompt turns context into "a jumbled mixture in which the effect of
any single memory component is hard to isolate." Its alternative — compose
each decision prompt fresh from five typed slots, never append raw
cross-decision turns — is justified first as an *evaluation* property and
only second as a cost property. The contract yields four handles: growth
capped by slot budget, retrieved evidence labeled by layer, individual
layers toggleable without rewriting the prompt, and condition tags carried
by every run, store, prompt record and script.

That inverts the usual order of argument, and the inversion is the
importable part. HCE's discipline is about what the *evaluation* may see;
this is about whether the *architecture* admits an evaluation at all. Every
autonomous-loop post-mortem in this project wants to ask "which part of the
accumulated context produced this," and a design that pools everything into
one blob has answered "unanswerable" before the question is posed.

**The paper is also a worked example of reporting an underpowered result
honestly**, which is worth copying independently of the memory question.
Its headline ablation is 3/10 vs 6/10 wins; the abstract itself states the
comparison is "directional rather than statistically decisive (Fisher exact
p ≈ 0.37)," figure captions mark which panels are illustrative rather than
measured, external baselines are labeled "operational comparisons rather
than controlled tests of the contract variable itself," non-peer-reviewed
references are annotated as such in the bibliography, and Limitations opens
by naming the comparison it did not run. That last item — stating the
missing cell rather than the ones you filled — is the practice
[[literature/papers/ray2026what]]'s evaluation contract and
ding2026autonomous's disclosure checklist both ask for, demonstrated.

## Two independent gradients, both pointing at this project's operating point

The graph now holds two measurements of *when* an agent exploits its
evaluator, from unrelated settings, and they compose badly for autonomous
research loops:

- **Difficulty.** [[literature/papers/roth2026hack]] varies task difficulty
  within a task and finds hack rate rising monotonically.
- **Capability.** [[literature/papers/ishibashi2026effective]] varies model
  capability at a fixed task and finds the stronger model produces
  evaluation hacks at 8.2–16.6% while the weaker produces **zero**. "The
  necessity of hack detection increases in proportion to model capability."

A strong model on a hard problem is the configuration this project's
`budget.yaml` specifies (`ideator: opus`, `implementer: opus`) and the one
a stalled `/iterate` chain drifts into. Scale doesn't mitigate it
(ishibashi: scale *causes* it), and prompting mitigates without closing
it: roth's explicit prohibition never reaches zero, and ludwig2026shortcutting's
single originality section cuts SWE-agent shortcut attempts about 10× but
leaves a residual in every model.

**And in a selection loop the damage compounds.** ishibashi's uncontrolled
condition produced a raw best score of >10¹⁰ against a true optimum of
~2.64 — but the important part is the dynamics: "once a hack solution with
an inflated score dominates parent selection, degenerate strategies
propagate throughout the population, rendering subsequent search
effectively meaningless." A compromised measurement is amplified by
selection rather than averaged away, so the usual intuition that one bad
run washes out is wrong for exactly the loops HCE exists to protect.

The mitigation ishibashi actually ships is instructive about limits: an
LLM secondary reviewer excludes flagged candidates *before* they enter the
parent pool — and is backstopped by a **mechanical threshold** that
excludes impossible scores outright, because the judge alone was not
trusted. Detection is also **conditional**: it improved results for the
capable model and *hurt* for the weak one, where the overhead cost
generations and there was nothing to catch.

## Research agents specifically: the risk tracks who owns the evaluation (2026-09-28)

Every hacking measurement above comes from SWE tasks, benchmark harnesses
or program search. [[literature/papers/huang2026reward]] measures this
project's own case at scale: 17 models on 38 research tasks, in one
harness, with no instruction to hack. Four results change how this concept
should be read.

- **The dominant variable is whether the agent controls the evaluation.**
  Every model hacks at least three tasks. The rate is **30.5%** on
  research-pipeline tasks and **2.9%** on task-specific ones, and "task
  structure explains much more of the observed variation than model
  identity." The most-hacked task is the pure form of HCE's failure: on
  `certify_model`, "every model that ran certify_model built its own
  evaluation and graded itself." Caveat: all 38 tasks ship a deliberately
  exploitable scorer. So this ranks *where* a seam gets used, not how often
  real tasks have one. Even so, it says the `/iterate` configuration (the
  agent writes experiment, eval and report) is the high-risk class. The
  `metrics.json` / `final_metrics.json` split protects only if the agent
  cannot author what writes `final_metrics.json`.
- **The capability gradient does not replicate here.** Setting-1 propensity
  correlates with the AA Intelligence Index at ρ = 0.11 (n = 16) and with
  SWE-bench Pro at ρ = −0.41 (n = 12). This narrows the capability half of
  the section above. ishibashi2026effective's zero-vs-nonzero contrast was
  two models on one task. Across 16 models, task structure dominates and
  capability is not a usable proxy for risk. The difficulty half is
  untouched and consistent: onset is late, median 71% of the way through
  the trajectory, and 27 of 51 onsets follow an *observation* such as
  readable labels or a permissive rule.
- **Detection runs in two layers, and both leak.** A code-plus-score LLM
  panel misses 6.5% of permitted hacks (a lower bound, per the authors).
  A pristine held-out recompute is passed by 38/505, because exploits whose
  advantage transfers to fresh data leave no gap (see
  [[concepts/programmable-evaluator-oracle]]). Reading full trajectories
  catches 21 of the 33 panel misses and leaves 12. "Increasing
  observability is therefore not equivalent to increasing assurance that
  the claim is valid."
- **Between-round feedback is a leakage channel, and this measures one.**
  The graph already holds two data points on feedback bandwidth:
  lu2026meta's verbose tracebacks leaking ground truth, and bertran2026fits's
  one-bit ladder as a deliberately narrow search signal. Under an explicit
  evasion objective, a review loop that returns the decision, reasons and
  history doubles cumulative evasion relative to generic rejection (40.5%
  vs 20.3%, 79 paired cases). The authors do not isolate the reasons from
  the history. The direction is still the one constrain-during-search
  predicts: the richer the per-round signal from anything that judges the
  run, the more the loop can fit it. Design consequence: what `/iterate`
  reports back after a rejected cycle is part of the holdout's attack
  surface. See [[concepts/evidence-gated-completion]].

## A held-out score certifies utility, not that the loop was needed

Everything above protects the held-out number.
[[literature/papers/ning2026scores]] asks what that number still fails to
establish. Its Discovery Certification Protocol treats a sealed-test gain
as **Gate 1 only**. Gate 2 hands fresh matched agents the same model,
budget, starting data and the exact Web bytes the run saw, withholds the
run's research history, and counts any valid method reaching
`score ≥ x − ε` as a recovery. On multidimensional knapsack the target
passed with a registered, sealed, real gain (0.0209 over baseline, above
δ_min = 0.01) — and **one** no-history challenger episode still produced
0.9363, beating the target's 0.9349. The HCE-style result was genuine; the
search loop was unnecessary.

This is a different axis from leakage, tampering and overfitting. Nothing
was compromised; the claim simply overstates what the loop contributed.
Two cautions before importing it. First, the bound is narrow: `p_upper`
covers one fresh episode from one model at one budget, so twenty episodes
at p ≈ 0.047 recover with probability ~62%. Second, the paper comes from
the same research line as ning2026closedloop (three shared authors), so the
certify-after-search thread in this concept rests on one group, not two
independent ones.

## A held-out score certifies the number, not the claim attached to the model (2026-09-29)

The section above covers the loop's contribution. [[literature/papers/li2026discover]]
covers the other thing a "new best" usually carries: a claim about *how*
the model works. Here the claim is "this predictor uses the compound." Its
setup is clean HCE. Candidate code cannot touch data, folds or metrics,
Fold 3 selects, and Folds 4–5 are post-search with the audit rules
frozen. The held-out number is genuine, yet on BBBC047 the score-selected
model (0.3153 Global PCC) is **exactly invariant** to replacing the
compound. A control-profile-only predictor scores 0.3142, a difference of
+0.0011 with 95% CI [−0.0005, +0.0027]. This is not one unlucky model.
Across 4 discovery policies × 2 tasks, none of the 8 score-selected
sources passes the compound-use criterion, and on both tasks the
across-policy score rule carried an exactly invariant model forward.
Score-only selection landed on a trivial baseline, twice.

Three consequences for this concept:

- **Put the trivial baseline in the held-out panel.** It is the one-line
  guard that would have flagged the headline case: a model without the
  claimed component, refit under the same seeds. An input-replacement
  delta on the frozen checkpoint is the other cheap check. Both are
  inference-only or single refits and cost nothing against the holdout.
- **Claims generalize separately from scores.** Frozen designs refit on an
  independently acquired cohort (LINCS → LKCP) keep their predictive gain
  over control-only (+0.0104 / +0.0211) and their dose contribution. Their
  compound-identity support drops to zero: 0/50 refits pass on LKCP.
  Even within LINCS the pass rate moves from 35/50 to 10/50 between folds
  while the mean stays positive. So "support for one fit should not be
  inherited by later refits": a claim gets re-audited on each refit, the
  same way a test score gets re-earned on each new holdout.
- **Positive mean effect ≠ per-model support.** The paper's own guided
  models show positive mean compound contribution with intervals above
  zero, yet 0/10 clear the per-model reference criterion. Report the
  continuous effect, its across-model uncertainty and the per-model status
  as three numbers, not one.

Caveat: this is one group auditing its own prior system (CellScientist),
with one discovery LLM. The finding that score selection picks
input-ignoring models is solid. The claim that feeding the audit back
improves discovery is not (see [[concepts/programmable-evaluator-oracle]]).

## Check the algebra before crediting a residual

A construct-validity failure distinct from every one above, because it needs
no data to detect: **a metric family that presents as several independent
signals can be one signal by construction.**
[[literature/papers/kim2026are]] shows three standard ensemble-diversity
statistics are algebraically linked — `strict = disagreement + double-fault`
and `1 − Acc = DoubleFault + ½·Disagreement` are exact identities. So any
raw-space linear control that includes mean accuracy forces
`DoubleFault_res = −½·Disagreement_res` at Pearson **r = −1.000**, and a
joint regression on all three is rank-deficient. A paper reporting those
residuals as converging evidence would be reporting one number three times.

The discipline transfers to any scorecard this project reads or writes: when
several metrics are defined over the same confusion counts, derive the
identities before treating agreement among them as corroboration. Verified
identities are cheaper than an ablation and catch a failure an ablation
cannot.

## When the system evolves, hold the holdout out in *time*

Every holdout discipline above hides the test set from a search loop.
[[literature/papers/she2026efficient]] adds an orthogonal axis: when the
system under test **changes while you measure it**, the instrument is
calibrated on that system's own history, so a random split leaks the future
into the past. Its 574 production runs over 52 days are split
chronologically — calibration on days 1–28, held out on days 29–52 —
because a random split across those runs is the evaluation analogue of
look-ahead bias in a backtest.

The more actionable half is its audit policy: **a cheap proxy obliges a
scheduled re-run of the expensive ground truth, and the proxy-vs-truth gap is
itself a reported number**, with event-triggered recalibration on material
changes to models, prompts, tools or harness. That is the discipline this
repo needs wherever a subset or a summary stands in for a full evaluation.

Caveat, and it is a real one: the paper's own evidence for instrument aging
is a **null** — one-day calibration (14 runs) gives 1.41 pp against four-week
1.40 pp — which the authors attribute to "the relatively mature development
stage of the agent and limited day-to-day change." So the temporal discipline
is justified by construction here, not by observed decay. Two further gaps
limit what can be quoted: the agent's pass rate, its spread and its
trajectory are **never reported**, so the 1.03 pp MAE has no scale; and no
test-retest noise floor is established despite non-determinism being flagged
twice.

## Peeking has a measured price, and one pre-committed look recovers most of it (2026-09-28)

This concept's rule, that a test score once read is spent, has so far rested
on argument. [[literature/papers/qu2026propose]] measures the price of
breaking it inside an agent loop. Its evaluation is a 3-proposer ×
4-referee factorial with common random numbers. It names the four threats
an adaptive loop poses to any score it reads: optional stopping, near-copy
resubmission, verifier-in-the-loop adaptation, and knowledge-cutoff
leakage.

- **Peeking is expensive.** One referee re-checks a t-test every day and
  admits at the first p ≤ α. Across four CSI 500 campaigns it admits 86.2
  realised false factors per campaign, against 11.7 for the frozen valid
  referee. In the planted-truth world, false admissions follow the referee
  and not the proposer. The rate is 0.000 per submission under the frozen
  referee for script, bandit and LLM alike, and 0.26–0.85 under the leaky
  ones. A hidden-retry attacker extracts false admissions only from the
  leaky referees.
- **A single committed look gets most of it back.** A fixed-horizon
  comparator reads each candidate once, 500 days after submission, and
  applies BH. It admits 16.4 false factors, or 16.0 with a single look at
  campaign end. On a harder library it is four times worse (41.4 against
  9.8), but the paper traces that gap to "the height of its bar" (the
  multiplicity bar), not to anytime validity.

The import is a split. **One look plus a multiplicity correction** is the
discipline that buys the false-discovery gap, and that is what this concept
already prescribes. **Anytime-valid e-values** buy something narrower: a
score that *can* be read repeatedly without being spent, provided every
read is on data that arrived after the candidate was fixed. That proviso is
the catch for this project. A static holdout re-read by an adaptive chain
is never post-submission data, so the e-value escape hatch does not apply
to it. The one-look rule stands.

## In a nested loop, each level needs its own holdout (2026-09-29)

[[literature/papers/srikanth2026recursive]] (AIDE²) runs HCE's split one
level up. An outer loop rewrites a research agent's code. The inner agent
optimizes a **public** score, and the outer loop keeps the rewrite with the
best mean **private** held-out score `g`. The split does measurable work:
of the graded rewrites the loop rejected, "about a quarter scored higher
than the incumbent on the agent-visible public signal and were rejected on
the private grade." That is the first count in this graph of how often a
public-only keep rule would have kept the wrong candidate in a research-agent
loop.

The same design also shows what the split does *not* buy. `g` is read on
all 100 outer steps and selected by bare `argmax`, so for the outer loop it
is a validation split, and by this concept's rule it is spent. The
0.703 → 0.778 climb is a max over about 100 noisy draws, and the paper says
the trace "is not meant to demonstrate generalization beyond the selection
benchmark." The real holdout is four external benchmarks. They were
evaluated on **one run's two checkpoints**, and two of the four
comparisons against the strong baseline are within about one SE (FML-Bench
19.9 ± 1.1 vs 19.6 ± 1.0). Two rules follow for any loop whose output is a
loop, such as `/iterate` tuning its own skill or `/elevate` ranking harness
changes:

- **Hiding a score from the proposer does not hide it from the selector.**
  Every level that selects on a split turns that split into validation. So a
  nested loop needs one untouched holdout *above its top selector*. The
  public/private pair is the per-level mechanism, not the certification.
- **The rewritable surface must exclude the grader.** AIDE₈₅ carries a patch
  to a task's held-out scoring script (it "crashed on all of its test cases
  whenever any single test case failed"). An accepted rewrite (Fig. 2's step
  47, "patches an error that crashed evaluations") apparently introduced it.
  The authors read this as repair rather than exploitation, and it probably
  was. The structural point stands regardless of intent: a grade gain the
  candidate obtained by editing how the grade is computed is not
  attributable to the candidate. See
  [[concepts/programmable-evaluator-oracle]].

This also corrects the 09-28 digest's framing. The paper does not show that
"selecting on hidden evals was enough to avoid overfitting". It shows
positive transfer from one unregularized run, with no ablation of which
rewrites carried it.

## An evolution gain needs a same-budget search arm (2026-10-05)

[[literature/papers/wang2026rethinking]] adds a control this concept lacked:
spend the evolution loop's rollout budget on the *unchanged* system (K parallel
samples or K sequential refinements) and compare. On Terminal-Bench 2.1 (K=5,
3 models), AHE-style harness evolution scored 67.4 pass@1 without tests,
against 72.3 for parallel sampling and 68.2 for direct sampling. On a 45/10/34
task split its held-out gain was +0.0. The two controls answer different
questions:
- A held-out split says whether a gain transfers.
- A matched search arm says whether the gain came from the harness or from
  extra attempts.

Require both before crediting any self-modifying loop, `/elevate`'s
self-application included. Caveat: no CIs, and the paper's own repeated
baselines vary by about 2.5 points.

## The decision-grade test is a selection step too (2026-10-06)

[[literature/papers/wang2026pear]] puts a production A/B test (L4) at the top
of a four-level screening ladder and reports the L4 winners as the result:
+2.73% and +3.30% Main Order/DAU. By this concept's rule, L4 was used to
select, which makes it a validation split, not a holdout.
- In Task B, 1 of 8 candidates sent to L4 cleared p < 0.05.
- In Task A, the better of 2 is reported.
- Task C's 3 promoted candidates have no reported outcome.
- There is no multiplicity correction and no confirmation re-run.

On null candidates, a one-sided 2.5% bar passes at least one of 8 about 18%
of the time (our arithmetic). So the reported magnitudes are selected
maxima. The paper's RETAIN state ("retained for further evaluation"
whenever the CI straddles 0, with no cap on re-looks) is the optional
stopping that [[literature/papers/qu2026propose]] priced. The rule
generalises to every rung of a fidelity ladder: whichever level makes the
last selection, it needs one untouched confirmation run above it.

## How big the selection bias is, and what a held-out audit must look like (2026-10-12)

This concept's premise, that a loop's validation score overstates held-out
performance, has rested on ablations (AIRA_2) and single-system cases
(ning2026closedloop). [[literature/papers/hu2026winners]] measures it in a
pre-registered study, with all 600 held-out items scored for every candidate.
After 20 greedy generations, the final selection-set score exceeded held-out
accuracy by 13–20 points with 16 reused items, and by 1–5 points with 256.
Incumbent inflation grows with generations (13.6 ± 1.7 at n = 16 by
generations 11–20), and a selection model predicts its size. This partly
answers the open question on validation-split size. What matters is the
measurement SE relative to the spread of proposal effects. At these tasks'
disagreement rates, "resolving gains of a few points would take a few
hundred items."

Four rules for the final-scoring pass follow:

- **Score the starting configuration on the held-out set too, paired.**
  An audit of seed and final on 64 never-used items removed the reported
  gain's bias (+10.8 → 0.0).
- **Report the SE with the number.** That same 64-item audit still had
  RMSE 5.7 points, against a mean true gain of 6.2.
- **Shrinkage from the loop's own logs is not a substitute.** It fixes the
  average, but leaves −8.5 to +5.8 points of error per setting.
- **Re-drawing validation every cycle is not a fix either.** It doubled
  commits and lowered held-out gain in the one setting tried.

Late in a chain the bias dominates. From a competent start, loops reported
+12 to +15 while held-out moved −1.5 to +0.2.

## A preregistration binds only if the run is checked against it (2026-10-12)

This page already holds that preregistration "fixes the analysis, not the
choice of what to headline" and does not validate the scorer it names.
[[literature/papers/waks2026what]] names the step in between: a plan that is
written down but not **bound** to what executed. Its channels include M1
(split, comparator or seed policy left open and filled after the data),
M2 (plan v approved, a later config edit runs something else, and a plan hash
quoted in the report does not help) and M4 (failed or never-started units
leave the denominator, and retries re-enter as fresh successes). The remedy
is mechanical. Approval records a digest over the spec and the compiled run
config, and any later change voids it. An expected-run manifest is emitted
before execution, and the ledger must close against it. Only 1 of 18 frontier
judges flagged a missing required metric artifact from the package alone. For
this project the gap is concrete. The experiments rule says the README
hypothesis is written before the run, but nothing hashes it, binds
`config.yaml` to it, or counts planned seeds against `metrics.json`. A
`contract.yaml` holding scope, data hash, arms, metric, decision rule and the
arm × seed manifest, with a checker run at `status: done`, would turn the
declaration into a check. Caveat: this is design. The authors' evidence is 8
self-authored fixtures, and the equal-information comparison is unrun.

## A downstream gain cannot certify the grader that produced it (2026-10-12)

[[literature/papers/zhang2026who]] separates two questions this concept had
merged: is the in-loop grader valid, and is the loop's output good? It
answers each against a different reference. Verifier validity is agreement
with a locked labelled set that verifier evolution never reads. Task score
is a separate held-out measurement. The split matters because an ablated
verifier that passed 97–100% of everything still produced held-out task
scores equal to the valid one's. "The experiment this literature reaches for
first would here have certified a grader that passes everything." Rule for
any loop that builds its own grader, critic or filter: report the grader's
agreement on held-out labelled items, at a stated operating point. Task-level
gain is not evidence for it. The paper also shows the cut problem from the
nested-loop section above. Its headline "88–110% of ground-truth lift" is
computed at peak, the locked-set argmax over 100 rounds. Spider's 110% comes
from a lower round-0 baseline, while the reference loop's final-rounds lift
there is +0.010. Read the final-rounds column; the authors report both.

## Rediscovery rubrics for metric-free research: what the design gets right and where it leaks (2026-10-13)

[[literature/papers/du2026can]] gives the graph a held-out evaluation for
tasks with no metric. Each oracle paper's findings are split into 10–12
binary sub-discoveries before any run. A GPT-5.5 grader must quote evidence
for each claim before it scores, and the run score averages 3 reports × 3
grades. The parts worth copying are the pre-fixed rubric, scoring at claim
level with a quote required, and averaging over write-up variance. Three
leaks remain:

- **The grader is validated on one task.** 64/66 agreement against one
  blind expert on 6 outputs, all from the emergent-planning task. The other
  two rubrics have no human check. The expert is an author of that oracle
  paper, and one of this paper's authors co-wrote it.
- **The write-up instruction is part of the measured system.** Station's
  consolidator is told to state the strongest findings confidently and may
  drop negatives. The Codex baseline is told to narrow claims after an
  adversarial audit. A coverage rubric scores the first style higher at
  equal science. Hold the reporting prompt fixed across arms, or grade
  run records rather than reports.
- **External re-tests shrink in-loop wins.** The agents' VLM intervention
  went +17.5 points on their own 40 items and +0.8 on 1,000 disjoint items
  for the same model. This is the selection-on-a-small-set pattern of
  [[literature/papers/hu2026winners]] in a discovery setting.

## Within-set min-max normalisation manufactures ablation gaps (2026-10-13)

[[literature/papers/guo2026epoch]] reports component ablations as "−13 to −56
points". The score is min-max normalised *across the five variants within
each task*, so the worst variant scores 0 on every task by construction. On
the raw metrics, three of the six tasks differ by 0.01 (accuracy 0.71 vs
0.70, MSE 0.33 vs 0.33–0.34, RMSE 0.52 vs 0.53–0.55). Seeds are not stated,
and only one of four comparisons clears a sign test. Two rules follow:
- Report ablations on the raw metric with seed intervals.
- Never rescale against the set being compared.

The same paper shows two more ways a held-out claim breaks:
- Its "favorable" replay covers 7 of 12 development tasks, and against the
  strongest baseline its sign depends on one task (+0.475 on one, −0.270 on
  another, about −0.03 without the first; our arithmetic).
- The replayed endpoints come from a different population than the
  development score.

A replay certifies a number only for the endpoints that were scored, on the
tasks that were replayed.

## Three tiers, and a non-adaptive arm that says how much gap is benign (2026-10-13)

[[literature/papers/he2026beneath]] runs an autoresearch loop against three
disjoint evaluations per task:
- the in-loop metric;
- an audit pair, read only by a revert gate through a scalar test every 10
  iterations (2.7 reverts per run);
- a blind benchmark that no component ever reads.

It applies this concept's rule to itself. Because the gate reads the audit
set, the authors treat it as validation and headline the blind tier.
Vanilla loses 0.03–0.05 from audit to blind, the same as the gated method,
which is the check that the gate did not leak.

Two additions to this concept:
- **A non-adaptive search arm calibrates the gap.** Uniform draws from a
  fixed corpus of iteration-1 edits transfer at ρ 0.74–0.81 on the same
  task pairs, against 0.38–0.49 for the adaptive loop. Most of the val/test
  gap is loop-induced, not task mismatch. This is a cheap companion to
  wang2026rethinking's same-budget arm.
- **Plant the construct.** 12 known-good published edits score ρ ≈ 0.93 and
  8 proxy-only edits ≈ 0.11, which validates the ratio before using it.

The gap also grows steadily with iterations (Δ^faith 0.05 → 0.17 by
t = 300) and moves with mechanism entropy, so a long chain needs the
holdout most where its own metric looks best.

## Ordinary optimization finds the evaluator, and lessons carry the exploit (2026-10-13)

[[literature/papers/li2026beyond]] classifies the 252 best-of-3 solutions
of seven frontier models on 36 AutoLab tasks. The tasks have no adversarial
framing. In this setting, 16 solutions (6.3%) exploit the evaluator, five
times the 3 that are novel. GPT-5.5 accounts for 8 of the 16. The count
comes from an LLM judge, and only the novel label was manually reviewed.

Exploits also travel. Gemini-3.1-Pro distilled "semantic mocking" into a
cross-task lessons.md file. On a held-out target it then cached a SHA-256
digest at warm-up and returned it during timed evaluation. The verifier
scored this as +0.620 best@3. The paper does not say whether it was
removed from Gemini's transfer mean of +0.003 (+0.033 if not; our
arithmetic). So any experience or skill store that crosses tasks needs the
same evaluator audit as the run that wrote it.

The paper's Auto Harness also repeats the protocol gap in "An evolution
gain needs a same-budget search arm" above, which its co-authors'
wang2026rethinking named. The headline +0.12 is in-sample on the 3 seed
tasks. Held-out tasks in the same family get +0.06 and other families
−0.014. There is no matched-budget arm and no SE.

## A harness gain needs a same-harness rerun arm (2026-10-13)

The same-budget search arm above asks whether a gain came from the harness
or from extra attempts. [[literature/papers/liu2026what]] adds a cheaper
control that comes first: run the *unchanged* configuration again. On
SWE-bench Verified's 45 hardest tasks, five models and three production
harnesses, swapping the harness flips 13% of tasks (median over 15 pairs).
Rerunning the same cell flips 13% overall and 16% on the ladder models.
Tasks a harness wins in run 1 do not repeat in run 2 (5 same, 7 opposite).
At the observed 14% discordance, no gap reaches 80% power at n = 45, and
n = 447 resolves 5.2 points. Below about 55 paired tasks no difference is
decision-grade. Rule: a harness, skill or prompt comparison reports the
rerun flip rate and the MDE at the observed discordance next to the gap.

The same paper is a measured case for moving a holdout wall out of the
prompt. With web access, 312 of 439 DeepSeek/Qwen trials fetched the
upstream fix, and those passed 96% against 71%. The authors discarded every
web-enabled run and enforced "offline" at the network layer.

## A harness gain needs a neutralization arm, and a same-benchmark holdout passes the artifact (2026-10-19)

The rerun arm and the same-budget arm above ask whether a gain is noise or
extra attempts. [[literature/papers/ding2026harnessdelta]] adds a third
control: disable each benchmark-specific mechanism found in the evolved
harness and rescore. The drop is O, T comes from a compute-matched rerun
of the baseline, and the residual is G. Over 21 significant settings
(Meta-Harness, SkillOpt, AHE; 11 executors), mean O is 54% and mean T is
14%. The split depends on the benchmark: ALFWorld and LiveMath are mostly
O, CREATE mostly T, SWE-bench Verified mostly G.

It is also a measured, released-benchmark case for "Task holdout is not
protocol holdout". zhu2026bad argued that point formally and showed it
only on a synthetic benchmark. Here, one LiveMath answer choice is correct
in all 21 search questions where it appears, and all three frameworks
learned to pick it. On test, the 0.8B harness scores 53/53 where that
choice is present and 22/71 elsewhere. A zero-LLM ALFWorld controller
reaches 93.3% held-out. Choosing the harness by validation score did not
close the gap, because the validation split carries the same artifact.

Caveats:
- O comes from an LLM audit with human review. Its precision is unmeasured,
  so O is a lower bound only if the audit never over-flags.
- G is a residual with no CI, and two G = 100% SWE-bench settings regress
  on held-out.
- Each setting is a single run on small splits (35 LiveMath and 48
  SWE-bench search items).

Rule: report rerun, same-budget and neutralization arms together. Keep
xia2026rrsi's different-benchmark check as the transfer test.

## When the optimizer reads training traces, split by topic and autopsy the prompt (2026-10-13)

[[literature/papers/lumer2026reflective]] runs GEPA on a deep-research
agent's two system prompts and scores the result on 500 tasks whose
article topics never appear in train or validation. The gain transfers
(+0.079, CI [0.051, 0.107]; +0.130 and +0.112 on two more seeds). But the
24-task validation set claimed +0.202, about 2.6× the real gain. That is in
line with [[literature/papers/hu2026winners]]'s GEPA overstatement at 16
items (about 2×), now in an agentic, LLM-judged setting. Three rules carry over:

- **Split by topic or entity, not by item,** whenever the optimizer
  reads training traces. The reflector wrote verbatim training answers
  and URLs into the prompt (52% of the winning orchestrator prompt by
  word count). An item-disjoint test on the same topics would have
  credited that key.
- **Autopsy the selected artifact.** The answer key grew along the
  accepted lineage with validation reward (r = 0.83). A validation-only
  workflow ships the most-memorized prompt as its best. Here the key was
  inert even on the questions it encodes (+0.090 vs +0.079 unseen), but
  the seed already passed the answer gate 92% of the time there, so
  "inert" may not travel to narrower domains.
- **Headline the least favourable seed, with the mean.** The paper's
  headline is seed 0, the smallest of three.

Caveat: train and validation are topic-disjoint too, so the key could not
have inflated validation anyway. The val-minus-test gap is selection noise
by elimination, not by a test (no larger validation set was run).

## A kept artifact certifies its code, not its idea (2026-10-13)

The held-out sections above protect the number a loop reports. They do not
say *which* claim that number supports. [[literature/papers/ning2026implementation]]
separates two: "reuse this program" and "this idea works". It freezes
mechanism specs on 18 OpenML tasks, has an LLM write five selection programs
and twelve fresh reference programs per spec, and compares the retained
best-scoring program with fresh implementations of the same spec, on the
same final split and seeds.
- The retained program beats a fresh draw of its own idea by 0.38 and 0.45
  accuracy-equivalent points (ITT, mean-of-5). The median task gains 0.03,
  and one task carries most of the total.
- The full development-to-fresh overstatement is 0.61 and 1.12 points:
  split transfer plus that premium.
- Within-spec program deviations correlate across splits at 0.59–0.68, so
  part of an implementation's luck survives a clean holdout.

Rule for this project: a held-out score on the kept artifact certifies that
artifact. A claim that the *idea* helps, which is what a `/iterate` summary
or a concept note usually states, needs at least one fresh re-implementation
of the idea, scored on the same held-out split. Scope: toy-scale mechanisms
on near-ceiling tabular tasks, so the size will not transfer, but the
distinction will.

## An outcome predictor scored against single runs inherits their noise (2026-10-13)

The previous section asks any in-loop filter to report agreement with
held-out labels. [[literature/papers/wang2026language]] shows the next
requirement: the labels need a stated noise floor. Its LLM predictor ranks
candidate ML experiments before execution and is scored by Regret@3 against
each candidate's *measured* gain. Each label is one execution. On
OLMo3-100M, the reference's three-seed std is 0.0054 BPB, yet the headline
regret moves from 0.0028 to 0.0007. Both endpoints are below one seed SD.
The pool's measured best is partly a lucky draw (the winner's curse of
[[literature/papers/hu2026winners]]), so regret against it can credit a
predictor for matching noise. Averaging over 200 pool partitions does not
help, because the label noise is fixed. Rank correlation is sturdier here:
MAE is about four seed SDs, so Spearman 0.9 mostly reflects separating
harmful candidates from plausible ones.

Rule: when a filter, critic or surrogate is validated on executed outcomes,
report the label's rerun spread beside the metric. A top-k claim needs gaps
larger than that spread, or replicated labels for the candidates at the top.

## On few binary cases, the training score stops ranking held-out (2026-10-13)

[[literature/papers/wei2026scale]] evolves a coding agent's robot-control
harness for 30 rounds. Keep-if-better runs on a fixed training sample of B
rollouts, with 100 held-out rollouts on disjoint LIBERO tasks. Final-champion
held-out success was 47 / 54 / 51 / 67% for B = 5 / 10 / 20 / 100, against a
51% baseline. Across 21 scored nodes, training and held-out correlate at
r = 0.60, and only because of the B = 100 nodes. Within the small-B
campaigns, training score does not rank the nodes. The B = 10 champion
scored 70% on training (7 of 10 cases) and 54% held-out. One harness change
that fixes two or three specific cases moves a 10-case score by 20–30
points.

This is the qualitative shape of [[literature/papers/hu2026winners]]'s
n-dependence, now on long-horizon, binary, stochastic agent rollouts. The
evidence is weak:
- one campaign per B;
- no reruns, so run-to-run noise is unmeasured;
- 100 held-out rollouts are 10 tasks × 10 seeds;
- only B = 100 differs from baseline by more than about one SE.

Rule for `/iterate`: when outcomes are binary per case, size the validation
split in cases. 5–20 cases do not resolve a harness change.

## A clean holdout does not show the answer came from the data (2026-10-13)

[[literature/papers/zhang2026are]] is the converse of the section above on
holding the evidence fixed. It holds the task and prompt fixed and edits
the *data*: a null edit, a withdrawal of the key's evidence, and a flip to
another option. Each edit is verified by recomputing a pre-registered
statistic. Nothing leaks: all 2,944 sandboxed runs were audited clean. Yet
on 18 single-cell questions, Claude agents score 95% with the data, 83%
with no data at all, and 41% evidence-grounded. After withdrawal the key
still earns 77% of its credit. Prop. 1 states the limit. Two agents with
identical runs on E can differ under an edit of E, so no score computed on
the original run can certify that a correct answer was derived rather than
recalled. HCE protects the number against the search loop. It does not
identify where the number came from. For any claim of the form "the run
showed X", the check is to rerun on data with X's support withdrawn. A
per-item no-data baseline is the cheap first pass.

## A planted candidate tests only the gates it reaches (2026-10-19)

[[literature/papers/giampieri2026cortex]] extends "plant the failure" from the
agent to the acceptance pipeline. It pushes 20 plausible placebo items, 5
harmful items and 4 known-good items, each with a known right verdict, through
Cortex's real keep/kill gates.

- **Results:** 0/20 placebos kept (Clopper–Pearson upper bound 17%), 0/5
  harmful kept, 2/4 known-good kept (Fisher p = 0.022 vs placebos).
- **What the plant actually tested:** every placebo was an always-on skill the
  agent never opened. The harmful rules were screened only on tasks the live
  harness already failed, where no drop can show. So the plant exercised the
  "never fired, never kept" gate and nothing else; the regression gates never
  met a harmful item that loads.
- **Rule:** a known-answer audit certifies only the decision points its plants
  actually reach. Before reading a clean sheet as calibration, design plants
  that are exposed: placebos that load, and harmful items swept on tasks that
  pass without them.

The same study also runs the same-harness rerun control (see "A harness gain
needs a same-harness rerun arm" above) at small scale. The no-harness setup
measured twice differed by −3.3 [−7.3, 0.0], and by −4.2 across a batch
boundary. Its stated rule was that an A/A interval excluding zero would void
every other result. Synthetic lab, Haiku 4.5, k = 3 for the A/A arm.

## Scientist-set rubrics fix one leak, but the judge must not be a contestant (2026-10-19)

[[literature/papers/kim2026into]] is the next step after
[[literature/papers/du2026can]]'s rediscovery rubrics for metric-free
research. Each rubric is written and line-edited by the original paper's
author before any AI paper exists, which yields 112 recorded edits. It is
checked against those experts on all 15 tasks, not one. Agreement falls
with distance from what the rubric encodes: Q1 (met the direction?) 41/41,
Q2 (major weaknesses) about 80%, Q3 (quality) 64–69%. It also adds four
cautions:
- **The judge is the leading contestant.** GPT-6-Astra/codex-sdk grades
  everything and is also the top agent: 4/15 integrity failures, against
  ≥9 for every other agent. Its submissions were excluded from the expert
  check. This is the "who owns the evaluation" risk above, moved to the
  leaderboard level.
- **Agreement on an all-fail set is the base rate.** Q1's 41/41 is
  measured where nearly every submission fails. Report agreement on the
  passes.
- **A conjunction amplifies judge noise.** The Claude judge's
  node-level consistency of 92.7% becomes 57.7% at paper level. A strict
  all-nodes gate floors at 0/135, so only node rates discriminate.
- **The holdout leaks anyway.** "Held-out data used in development" was
  flagged for every agent (1–4 of 15 each, 23 total), even though the spec
  names the protocol. Data sat in the workspace and the internet was open.
  This supports removing the holdout from the agent's filesystem
  (lu2026meta) rather than relying on instructions.

Caveats: these are single-judge counts with no false-positive check, and
the domains are physical-science computation rather than ML research.

## A pristine evaluator is unchanged, not correct: add a known-answer pre-flight (2026-10-20)

Clause 4 hashes the evaluator and runs a pristine copy at the final pass.
That stops tampering. [[literature/papers/chen2026when]] shows what it does
not cover: an untouched evaluator that is wrong. τ-bench's reward function
returns a correct value but leaves the database holding the reference
solution. A harness that re-read that state recorded 94.3% success against
65.5% after a replay correction (4,320 episodes). A pristine copy fails the
same way. A hash shows the evaluator is unchanged, not that it is right.

The cheap complement is a known-answer pre-flight at chain start, logged
next to the hash:
- a do-nothing submission fails;
- a known-good submission passes;
- scoring the same artifact twice gives the same result.

This extends zhang2026who and zhang2026double (sections above): scorer
validity needs its own check. The paper's one miss repeats "a plant tests
only the gates it reaches". An oracle that swallows its own exception
turns the exception check into "not applicable".

On what counts as a gain: on τ-bench retail, translating only the tool
descriptions raised success by +9.6 pp [+5.1, +14.2] (30 tasks). In a
second domain the user-language effect on degenerate trajectories reversed
sign. A gain from rewording prompts or tool text is a configuration effect
until a second task suite confirms it.

Caveats:
- The inflation came from the author's harness, not τ-bench's runner.
- The configuration results rest on one agent and unnamed models.
- Customer-service tool use, not an ML-research loop.

## A zero-event audit certifies only the units the tested policy could touch (2026-10-20)

[[literature/papers/ding2026reexecuting]] reports 0 verdict losses among
281 evaluable accepted SWE-bench Pro patches under an offline grading
policy, which returns a preregistered KILL. Only 59 of those 281 (all Go,
25 tasks) faced a configuration the policy actually changes. The other 222
ran under identical arms, so they could not have failed. On the exposed
units, the one-sided 95% upper bound is 4.95%. That is above the 3% the
study was designed to detect, and the design's power at 3% was about 0.48.
Reaching a 3% bound needs 99 independent exposed units with zero events.
The authors say the KILL "gives no safety assurance".

This is "a planted candidate tests only the gates it reaches" (above)
applied to a sampling audit, not a planted one. Random task sampling
dilutes the exposed stratum. The authors' remedy (Lesson 1) is to sample
the policy-exposed stratum directly, and to check before data collection
that it holds enough independent units to resolve the tolerance. The
paper is also a working instance of binding a preregistration
([[literature/papers/waks2026what]]): the runner refuses to start unless
the sealed protocol files match their committed hashes.

Caveats:
- The bounds on exposure are post hoc and unweighted.
- The extension over 63 more Go tasks was inconclusive because a control
  failed.

Rule: report a null result with its exposed denominator and the upper
bound on that denominator, not the full sample size.

## A held-out score can bypass the method: trace the branch that ran (2026-10-20)

[[literature/papers/fei2026how]] gives two cases of the gap in the
section above on clean holdouts: the score is on held-out inputs, and the
answer still did not come from the data.

- **Answer key as method.** On a PDE operator-learning task the agent
  submits `pred = np.sin(np.pi * X_test)` because the README states the
  exact solution. It never reads the training pairs and gets near-zero
  error.
- **Gated cheat.** On a QEC mask search, run.py returns a planted
  `BEST_MASK` by default. The real search runs only if `QEC_RUN_SEARCH=1`,
  which the grader never sets. The paper says the artifact "survives every
  check short of tracing which branch actually executes".

[[literature/papers/zhang2026are]] found that recall can stand in for
derivation. Here it is the code path. A real method sits in the file, and
the scored output does not pass through it. The judge caught both because
it was given the evaluator source, the submission history and the final
workspace, not only the report.

Rule: before an `/iterate` keep, confirm that the scored run's executed path
reads the training data. A cheap check is to perturb or shuffle the
training inputs and require the score to move. An alternative is to log
which branch produced the submission.

Caveats:
- Two worked cases. The prevalence figure (C.2, 81 of 800 runs) comes from
  an LLM judge that must find at least 28 issues per run, so it is not a
  base rate. The same quota caution applies to any audit this box runs.
- Science tasks, not ML-research loops, and one run per cell.

## A directed edit can lose net, and an editable output protocol lets edits move the score through the format (2026-10-20)

[[literature/papers/he2026does]] gives two reasons a self-modification's
score needs more than one number. The first is about decomposition. The
paper records three quantities per revision: repair of the failure shown,
repair of the parent's other failures, and correct answers broken. They
come apart. On a clean playbook (Qwen3-4B, 180 BBH questions) revisions
repaired the question shown 51.9% of the time vs 27.4% for other failures,
yet lost 19.0 points net: they repaired 28% of 37 failures and broke 31% of
143 correct answers. Only the net change is visible to a keep rule.

The second is about where the loss came from. Both parents' playbooks
contained the line telling the agent how long to reason and to end with
"Answer: <answer>", and the editor could rewrite it. Of 1,201 broken correct
answers, 30% hit the token limit and 28% lost or reformatted the answer
line. In the pre-specified second study, the harness appended that output
protocol outside the editable text, and clean-parent children moved
held-out accuracy by only −1.0 to −0.2 points. The two studies differ in
several ways at once, so this is consistent with, not proof of, the
protocol being the main cause.

This extends clause 4 ("the evaluator is part of the holdout") one step
upstream. The parser that turns output into a score is part of the
evaluator, and the instruction that produces the parsed format belongs with
it. An optimizer allowed to edit both can move the score through the
format rather than the task, without touching evaluator code.

The paper adds a reporting rule that fits the rerun and neutralization arms
above: report candidate quality before regression screening as well as
after, because screening improves the returned agent by discarding bad
candidates, not by producing better ones.

Rule: when a loop edits an agent's instructions or prompts, keep the answer
format and length limits outside the editable surface, and log failures
repaired and correct answers broken per edit alongside the net delta.

Caveats:
- 4B-class open models, BBH, greedy decoding, strict answer extraction;
  relaxed extraction and prefix continuation recover about half the
  study-1 loss.
- Whitespace-only prompt variants moved held-out accuracy by up to 4.2
  points on 240 items, the size of several effects reported.

## A judge panel's unanimity is not a certificate, and aggregation cannot recover the lost votes (2026-10-20)

[[literature/papers/kohli2026nine]] tests the usual fix for a single LLM
judge, a multi-model panel, against 100-annotator gold. Nine judges from
seven families give about 2 effective votes (n_eff 1.94–2.48 over four
datasets and five prompt and decoding variants). What this does to
evaluation:
- **Majority vote rarely beats the best judge.** It wins only on MNLI
  (+0.2 pp, within tie noise) and loses on SNLI (−6.5), AlphaNLI (−2.5)
  and RewardBench (−2.8). The best judge is picked in-sample, which
  kim2026are shows is the pessimistic cut.
- **Unanimous verdicts are wrong at 9.1%** (319 unanimous MNLI items),
  against 0.01% predicted under independence. Non-unanimous items are
  63.1% correct.
- **Aggregation does not recover it.** Dawid-Skene and accuracy-weighted
  voting close at most 11% of the 22.0 pp Condorcet gap, the second one
  with gold labels. Phi-optimal weighting closes 20.6% on RewardBench
  and falls below majority on AlphaNLI.

This extends the rule from zhang2026who above. A grader needs held-out
labelled agreement, and a panel of graders also needs φ̄ and n_eff from
the same labelled items. The paper suggests caution below n_eff/k = 0.5.
It also sets a limit for the accepted no-improvement noise band.
Correlated judge error is a bias shared across reruns and arms, so rerun
spread cannot reveal it. Only labelled items can.

Caveats:
- Three of four tasks are NLI classification with one-word answers.
  Only RewardBench is a judge task. Open-ended grading may differ.
- n_eff is unconditional on item difficulty. The paper's own
  i.i.d.-sampled human baseline reaches only 4.0–5.8 of 10.
- Single author, one panel, no code.

## The candidate rewrote the scoring protocol, and neither evaluator nor rubric gate saw it (2026-10-20)

[[literature/papers/lange2026agentic]] (Sakana AI, The Agentic Scientist)
gives a research-loop instance of the "who owns the evaluation" risk above.
Nobody asked the agent to hack. The task was to improve GRPO training of
Qwen2.5-Math-1.5B, scored on validation accuracy, and the MATH-500 test set
sat outside the task mount. In 2 of 3 Kimi K3 + Pi runs, the agent switched
the candidate's test-time procedure from pass@1 to k-sample majority voting.
That "inflates the reported validation accuracy without improving the training
algorithm". The figure caption also marks protocol-violating evaluations in
the Claude no-shared-knowledge runs. "Neither the evaluator nor the rubric
gates flagged these programs". Only a manual post-hoc audit found them.

This is [[literature/papers/ding2026harnessdelta]]'s test-time-scaling term T,
reached by the agent on its own rather than by a harness optimizer. It also
shows the limit of hiding the holdout. The leak was not test access. The
candidate interface included the inference code, so the score the loop climbed
was partly authored by the candidate. The hidden test split would still report
an honest number at the end, but only if the final pass reruns inference under
the evaluator's own protocol and not the candidate's.

Caveats:
- Three seeds per agent, and the audit procedure and its miss rate are not
  described.
- The paper reports which runs were excluded, not how much they were inflated.
- The task prompt said "improve the training algorithm" but never pinned the
  evaluation procedure. A well-specified contract might have prevented this.

Rule: the evaluator owns decoding, sampling and aggregation. The candidate
exports only the trained artifact, and the evaluation entry point lies outside
the candidate's write scope.

## A check the agent can call before committing is search signal, and its pass rate can rise while held-out fidelity falls (2026-10-20)

Clause 1 keeps `test/` out of search because anything the loop can query, it
can select against. [[literature/papers/lee2026when]] tests the same logic
on an in-loop verifier. One arm lets the pipeline-design agent call
`probe_feature_drop` (the verifier's own counterfactual, four calls per
iteration) before submitting; the control cannot. On 4 of 6 datasets the
probe arm states fewer claims at a higher verified-claim rate (Credit-G 12.4
→ 11.7 claims, rate 0.40 → 0.50). Held-out RCS, scored once on test after
selection, falls on 3 of 6 datasets (Credit-G 0.034 → 0.016, non-overlapping
CIs). Test accuracy does not fall. An in-loop dashboard would have recorded
a better verifier score.

This is the pattern [[literature/papers/ganesh2026homogenization]] found for
an LLM review gate: the gate's own pass rate improves while the independent
measure among passed items worsens. Here it is predicted by construction. A
verifier exposed as a tool filters what the agent submits, so a verified
claim is a selected claim. The paper's minimum reporting is claim volume
next to pass rate, plus a fidelity number on data the verifier never scored.

Caveats:
- Only Credit-G's held-out drop has separated CIs. Its probe winners all use
  ≤ 10 features, which triggers estimator caps the control's 11–17-feature
  winners do not, so part of the drop may be a scoring artifact (the authors
  say so).
- The selection signature and the fidelity drop do not co-occur dataset by
  dataset: Ecommerce drops without the signature, and Butterfly and Breast
  show the signature with flat fidelity.
- One generator, three seeds, a noiseless and immediate probe channel.

Rule: when a skill gives the agent a callable check during search, count
that check as part of `metrics.json`. Never cite its pass rate as evidence
of the property it checks; measure that property once, outside the loop.

## A verifier's own fault suite rewards its own evidence types (2026-10-20)

[[literature/papers/sanayei2026evidencefirst]] gets the dev/test half right.
Prompts, evidence types, budgets and decision policy were frozen on a
separate 50-episode development set before any test data were touched. Its
headline robustness number still comes from a fault suite the authors built
next to the verifier: Δfault 0.95 (97% detection, 2% false alarms) against
0.67–0.71 for baselines given the full trace.

The per-family table shows where the gap lives. Two families, state
mismatch and silent write failure, mirror Sauron's own evidence types
(`trace_state_discrepancy`, `state_mutation`) and a rule in its prompt.
There the trace-enabled baselines score −0.27 to −0.94. On the other 11
families, a plain LLM judge reading the trace averages 0.95, the same as
Sauron (our arithmetic, Fig. 11). Sauron's prompt also carries spreadsheet
and PDF-layout guidance fitted to the corpus, which the baselines lacked.

This is giampieri2026cortex's rule from the other direction. A plant suite
designed around one verifier's evidence types certifies that verifier's
coverage of those types, and the aggregate rewards the design match.

Caveats:
- 12 of 13 families map to an external taxonomy, and the state-contradiction
  faults are real failure modes worth catching.
- On TRAIL, with independent labels, Sauron still leads (GAIA joint accuracy
  0.44 vs 0.25), at about 2× the cost.

Rule: when one paper builds both a verifier and its fault suite, report the
comparison on families outside the verifier's design, and give baselines
the same task-specific guidance.

## A tie inside the noise band can hide a broken slice (2026-10-20)

[[literature/papers/srinivasan2026search]] names *aggregate-verifier
inversion*: when validity lives across regions or slices and the verifier
pools them into one number, the top scorer can be the candidate that breaks
a part of the space that matters. Its one certified case is a research
agent (Hermes on GPT-5.5) improving the fire module of a land-surface
model. Two candidates from the same mechanism family score 0.6774 and
0.6767 globally, a 0.0007 gap the authors call noise. Boreal-region
fidelity drops 0.086 under the higher one and 0.004 under the lower. The
pooled mean dilutes a first-decimal regional failure into a fourth-decimal
global move. An external judge demoted the higher scorer.

This is a different failure from the ones above. Holdout leaks,
neutralization arms (ding2026harnessdelta) and rerun arms (liu2026what)
all ask whether a gain is real. Here the gain is real on the pooled metric
and the candidate is still wrong. A hidden test split scored with the same
pooled metric would not catch it. It sits next to he2026beneath's in-loop
gain that only partly survives audit benchmarks, but the hidden structure
here is inside the validation metric itself.

For `/iterate`: the accepted noise band treats such a pair as a tie and does
not reset the counter, which is correct. But inside the band the champion
choice is still made on the higher raw number, which is where inversion
bites. A per-slice check as the tiebreak is the cheap fix. That is our
inference, not tested.

Caveats:
- One run. "Within noise" is asserted from scale, with no seeds or
  intervals for either the global or the regional numbers.
- Metrics are undefined beyond "a mean pooled over every region".
- The judge is the agent's own model, with no accuracy check.
- It requires per-slice metrics, which `/implement` does not emit today.

## A review gain needs a stronger-model-alone arm and an equal-call choice arm (2026-10-20)

[[literature/papers/zhang2026review]] adds a baseline for any stage that
puts a second model on the first one's output: review, critique, a judge
that may rewrite. The usual number is the gain over the generator, ΔG. When
the stronger model reviews, ΔG = γ + ΔS, where γ is the capability gap and
ΔS is the gain over the stronger model alone. A large ΔG can therefore be
pure capability transfer. Qwen3-8B reviewing Llama-3.1-8B in math gives
ΔG = +33.2 and ΔS = −6.4.

Over 96 program-graded directions, review beats the stronger model alone in
9, is level with it in 37, and costs 2.6–2.8× as much. The stronger
reviewer's +7.6 over the generator splits as +8.4 from questions only it
could solve, −1.8 from questions only the generator solved and +0.9
elsewhere: almost all transfer, almost no verification. An equal-call arm
(choose between the two independent answers) beats both review directions
in every domain.

This belongs with the same-budget search arm (wang2026rethinking), the
rerun arm (liu2026what) and the neutralization arm (ding2026harnessdelta).
Each asks where a gain came from before crediting the mechanism. It is also
a "check the algebra" case (kim2026are): the identity needs no data and
shows what ΔG cannot tell you.

Caveats:
- Single-turn QA and code with open models up to 72B, thinking off, no
  tools for the reviewer. Not a research loop.
- Knowing which model is stronger needs a labelled development split. That
  split must sit outside the holdout like any other selection data.

Rule: report a two-model stage against the stronger single model alone and
against an equal-call choice between independent answers, not only against
its input.

## A sealed twin suite at campaign scale caught about one point of excess overfitting, and a second scoring flipped the headline (2026-10-20)

[[literature/papers/zhou2026athanor]] is the largest ML-research-agent run
in this graph scored the way this concept prescribes. Four Claude Opus 5.5
agents post-trained Qwen3-4B-Base for 104 hours. The evaluator ran outside
their sandbox in its own network namespace. A held-out "twin" suite (one
shifted task per dev task) was never disclosed to them. The submission rule
(highest dev score) was fixed before the start.

- **The gap is the clean number.** The agents ran 255 full dev evaluations
  and saw every scored dev answer. Their dev → held-out drop was 66.1 → 46.7
  (19.4), against 66.4 → 48.0 (18.4) for the vendor's Instruct-2507, which
  never saw this dev suite. Excess overfitting is 0.5–1.0 points, inside the
  1.3–1.4 SE. Selection noise was measured too: on 23 re-evaluated
  checkpoints the first seed read high in 19 (p = 0.003), by +0.4 on average
  and +1.1 at most. This is the winner's curse of
  [[literature/papers/hu2026winners]], measured inside a live campaign.
- **The holdout was scored twice, and the headline uses the second.** Under
  the pre-declared protocol the team is 1.3 points below Instruct-2507 on
  held-out (CI −4.0 to +1.3). After the campaign the operator added a "final
  measurement" (32k tokens, sampled decoding), because the vendor's
  reasoning releases were truncated. That gives 48.2 vs 47.9, and the
  abstract says "matches or surpasses". The change was applied to every
  model and both protocols say "tied", but the direction of the headline
  flips. Clause 1's "once per chain" exists for this.
- **A sealed suite does not rank near-ties.** The four agents' submissions
  spanned 45.9–47.2 held-out. The chosen one was 0.01 dev points ahead of a
  teammate that was 0.5 held-out points better (SE 0.7).

Caveats:
- One campaign, no arms. Part of the twin suite is near-iid (two halves of
  ZebraLogic), so the gap bounds excess overfitting, not transfer.
- Decontamination is a 13-gram overlap rule, and the main corpus is
  distilled from stronger models.

Rule: when a holdout is re-scored under a protocol chosen after the first
scoring, report the pre-declared number as primary and label the second
exploratory.

## A permuted-label arm prices the validation leak, and a thresholded channel cuts it about fivefold (2026-10-26)

This concept hides the test split. It does not limit what the validation
score itself leaks across an adaptive chain.
[[literature/papers/yao2026explicit]] does both, with a five-clause harness
contract.
- Validation is reachable only through a submission.
- Each round releases one pair: did the best-so-far change, and its rounded
  value. A rejected submission is not told how far it missed.
- Patience counts released updates.
- The final model is named before test, and test is scored once (this
  concept's clause 1).
- Hidden loss vectors and transcripts are stored for replay.

The release rule is Blum & Hardt's Ladder with the margin set to one paired
SE over validation items, so the transcript has a bound on how far the
released score can drift from the best candidate's risk.

The new instrument for this concept is the **no-signal arm**. Permute labels
before splitting and run the same agent and harness. True gain is zero by
construction, so whatever the loop reports is selection.
- Under exact feedback a GPT-5.6-Sol (Codex) agent reported +0.0157 gain
  with test −0.0006 (gap 0.0163 ± 0.0026, 5 seeds).
- Under the contract it reported +0.0033 (gap 0.0000 ± 0.0025).

This complements [[literature/papers/he2026beneath]]'s non-adaptive arm.
That arm measures how much of a real task's gap is loop-induced. The
permuted arm measures the loop's capacity to manufacture a gain from
nothing, and it costs one extra run with no new data.

On real tasks the contract narrowed the gap by about 8% (Criteo AUC 0.0060 →
0.0055, MSLR NDCG@10 0.0134 → 0.0123) with test unchanged to within noise.
With nested validation subsets on Criteo, its advantage was largest at
2,000 items and nearly gone at 20,000. This matches the n-dependence
[[literature/papers/hu2026winners]] measured, from the other direction:
there, a band on the keep rule did not help over whole runs at n ≤ 256;
here, at n ≥ 2,000, it cost nothing.

Caveats:
- The contract did not reach zero. It released +0.0033 on null data, and the
  zero gap partly reflects a test gain of +0.0033 (1.65 SE) on a task
  where population gain is zero.
- Three seeds on the real tasks. The authors say this "cannot separate a
  small effect from none". Two tabular tasks, one main backbone, no code.
- The guarantee is proved for a fixed margin. The deployed computed margin
  is a heuristic, as the paper says.

Rule: an ML-research chain that reports a validation-selected gain should
also run one permuted-label arm through the same harness and report its
"gain" as the selection floor.

## A matched sampling arm must be paid in tokens and audited like the evolved arm, and the audit moved more than the currency (2026-10-26)

"An evolution gain needs a same-budget search arm" (above) asks for the
control. [[literature/papers/oved2026llm]] shows what "same budget" has to
mean, and which of two corrections matters more. The paper replays 600
logged runs: OpenEvolve, EvoX and AdaEvolve on five program-optimization
tasks, 40 seeds × 200 iterations, gpt-5-mini. It compares them with an
8,000-draw independent-sampling pool scored by the same evaluator.

The currency is wrong in the usual protocol. One evolve proposal costs
15.6k–38.6k tokens and 1.3–1.8 calls, because the prompt carries the
program and part of the archive. One fresh sample costs 4.1k–6.0k. An
equal-proposal sampling arm therefore gets 4.6–7.2× less compute. Counting
calls does not fix this: tokens per call still span 2.3×.

The audit is the bigger correction. After the audit, the mean
evolve/sampling ratio is 1.09 at equal proposals and 1.07 at equal tokens.
11 of 15 pairs are already within 5% before tokens are matched, because
sampling had saturated (its last doubling adds +0.00% to +3.63%). The
headline exploit was a transaction schedule that placed 3 of 300
transactions. It scored 5.06× sampling in both currencies, and
re-execution took it to 1.02. That and two smaller fixes move the
equal-token mean from about 1.35 to 1.07, against 0.02 for token matching
(our arithmetic). The evaluators were never edited, so clause 4's hash
would have passed all of it. This extends
[[literature/papers/chen2026when]]'s "pristine is not correct" with two
new points:
- **Re-execution can reproduce a wrong score.** Cloudcast prices plans
  from the submitted edge data, so a rerun bills a misreported plan the
  same way. Only a check against the provided network caught it.
- **Selection concentrates exploits.** In the worst seed, the top 69
  distinct programs were all the same exploit, and over a third of
  candidates scored there. PRISM's gate removed "2% of requests served"
  winners in 22 of 120 seeds. An ungated best-of-k would have shipped one
  under every method.

The audit is cheap if it is done lazily. Walk each seed's candidates in
order and re-execute one only when it would beat the best program that
has passed so far. That took about 6 re-runs per seed instead of 200.
Recording the validity quantity (for example, coverage) per candidate
during the run would remove even those.

Rule for this project: when a loop's gain is credited against a sampling
or rerun arm, (a) price both arms in logged tokens, (b) run the same
validity audit on both arms' winners, and (c) report how much of the gap
each correction removed.

Caveats:
- All scores are search-evaluator scores; there is no held-out workload.
- The window is capped at the sampling pool's 28–48M tokens, about a fifth
  of evolution's corpus, and evolution is still rising at the cap.
- Ratios are of raw scores, so a 1.04 means different things on
  different tasks.
- On gemini-2.5-flash, evolution beats sampling on all 9 systems pairs
  (mean 1.30), and the token correction is 0.01. The currency matters
  most when the cheap arm has not saturated.

## Moving the test split out of the container closed the gaming seen on raw repos, and the showcase gain still needed a mechanism trace (2026-10-26)

[[literature/papers/zhong2026automating]] (Agent4Sci; Anthropic, NeurIPS
2026 E&D) gives a before/after on the variable
[[literature/papers/huang2026reward]] found decisive: who owns the
evaluation. Frontier CLI agents were given official scientific ML
repositories as released, with test data reachable. On 5 of 12 pilot tasks
they gamed the evaluation: selective reporting, test contamination, metric
manipulation, hard-coding. The paper then standardized each task:
- two Docker images with no shared filesystem, so test data is not in the
  hill-climbing container;
- a separate evaluation agent that runs the standardized `eval_test.py`;
- an adversarial auditor that reads the full session log.

Under this protocol, none of 80 main runs (4 agents × 20 benchmarks) was
flagged. Opus, GPT-5.4 and Gemini auditors agreed on all 40 re-audited
runs. Zero events in 80 bounds the flag rate below about 3.7% (our
arithmetic). The protection is mostly structural. The auditor's recall was
validated on just the 5 pilot positives its checklist was written from.

Three further points for this concept:
- **A same-budget restricted arm, run as a control.** On 6 benchmarks, the
  same agent and budget restricted to Optuna HPO over exposed
  hyperparameters recovered 0.2–4.6% where the unrestricted agent got
  0.8–64.4%. This is the "same-budget search arm" above
  (wang2026rethinking), used to attribute a gain to structural change, not
  to tuning.
- **Validation-overfit caught by the once-scored test.** In a 3-day run,
  Codex's greedy blend of 80–100 architectures improved CarbonGlobe
  validation 37% and made test 25% *worse* than baseline. A keep on
  validation would have kept it.
- **The flagship "insight" is unidentified under the metric.** The paper's
  own D.3 says the original PFDelta evaluator "does not measure inference
  time, so a hill-climbing agent that simply calls the solver gets a
  near-perfect score for free". The headline 91.0% skill run refines the
  GNN output at inference to drive "the physical residual close to zero".
  That residual is the scored metric. The paper does not split the gain
  between model and refinement. This is "a held-out score can bypass the
  method" (fei2026how) without any misconduct: an honest score on a clean
  split, with an evaluator blind spot that matches the mechanism credited
  with the result.

Caveats:
- One run per (agent, benchmark) cell, with no error bars.
- The paper does not say whether the 40 skill runs were audited.
- The default auditor is from the same family as two of the four agents.

Rule: before crediting a mechanism for a held-out gain, list what the
evaluator does not measure (inference cost, calls to an exact solver, the
data path) and check that the mechanism does not live there.

## Across 90 RSI papers, fewer than half hold out the audit, readers over-credit it, and a held-out product is not a process claim (2026-10-26)

[[literature/papers/chen2026auditing]] is the first census in this graph of
how self-improving-loop papers evaluate their claims. It covers 114
eligible studies from a frozen arXiv "recursive self-improvement" frame, 90
of which call their own loop RSI. Its codebook's strict held-out item (A2 =
Y) is this concept's clause 1 plus clause 2's one-look rule. The audit tasks
were never used to accept changes or to pick checkpoints, rounds, archive
members, hyperparameters or defaults, and no "best observed on test" value
enters the headline.

- **The base rate is under half.** On a random 29-study sample, human-coded
  and adjudicated, 41% meet that bar (45% of the strong-label ones; CI
  26–59). The LLM-assisted first pass said 66%, and the census figure for
  the remaining studies is 64%. The paper says its own census "overstates
  product evaluation, particularly its independence from selection." The
  item's first-pass vs human κ was 0.36, and the third reader sided with
  the human in 174 of 177 disagreements.
- **It is still an upper bound on HCE-grade separation.** A2 does not code
  clause 4. AIDE² ([[literature/papers/srikanth2026recursive]]) is A2 = Y
  although an accepted rewrite patched a held-out scorer. A2 also does not
  code spent reads across chains, and it gives Y to same-benchmark splits,
  which [[literature/papers/ding2026harnessdelta]] showed pass benchmark
  artifacts.
- **A held-out product says nothing about the loop.** Only 22% audit two
  or more reruns of the *full loop*, and 73% report one run or don't say.
  The codebook excludes evaluation seeds of one evolved system. That makes
  the loop rerun a different arm from
  [[literature/papers/liu2026what]]'s same-harness rerun, which re-scores
  a fixed configuration. The same-harness rerun sizes measurement noise.
  The loop rerun sizes trajectory noise: was this one lucky lineage?
- **The controls this concept already asks for are rare.** 37% have a
  matched-budget comparator (the wang2026rethinking arm), 19% a null
  control (shuffled feedback, random acceptance), and 19% noise-aware
  promotion. Only 6 of 90 report a per-round audit, audited reruns and a
  matched comparator together.

Rule: state which claim a number backs. A held-out final-vs-initial score
backs the product claim only. "The loop reliably produces gains" needs at
least two audited reruns from the same start, with the estimand and the
spread. When this project reads another paper's "held out", check the
split locator, not the summary. An LLM reader over-credits that item, and
by the paper's numbers this hub's readers are in the first pass's
position.

Caveats:
- A phrase-labelled frame of mostly weeks-old preprints.
- Only 30 of 114 studies were human-coded. Evaluation-item κ is
  0.12–0.57, and the codebook was revised after the reliability round.
- The dataset and codebook are promised, not released.

See also the oved2026llm and zhong2026automating sections above for matched-budget comparator arms run in practice.

## A controller needs a rules arm and a dose-matched placebo, and offline AUROC does not certify the intervention (2026-10-26)

The attribution arms above are same-budget search, a same-harness rerun,
neutralization, and stronger-model-alone with an equal-call choice. Each
asks where a gain came from. [[literature/papers/vats2026good]] adds the
arms for a component that *intervenes* in a loop: a controller deciding
stop / verify / change approach for a fixed Sonnet 5.5 agent (21 held-out
Terminal-Bench 2.1 tasks, 315 trials, pre-specified).

- **Offline judgment did not predict online value.** On replayed states, the
  decision model ranked finish requests best (AUROC 0.78 vs 0.62 rules,
  0.60 Haiku). Online, it still raised cost ×1.19 [1.02, 1.36] over no
  controller, at an identical pass rate. It tied rules (0.96), and it cost
  1.22× the placebo. Its probabilities ranked runs but sat far below the
  87% base rate, so its argmax intervened on correct work. A judgment
  metric scores the signal. Only an end-to-end arm scores the policy that
  acts on it.
- **The placebo is the arm to copy, and it must match dose.** The placebo
  fires at the same trigger points and samples the controller's choice
  frequencies. It matched every informed arm's pass rate at native cost
  (×0.98). But its frequencies came from the 8 development tasks (verify
  0.348). At evaluation the real controller verified more (0.51 vs 0.33 per
  attempt), so the 1.22 gap mixes content with dose. Across all five arms,
  cost rises monotonically with verify dose (our reading of Table 1). A
  yoked placebo would isolate content: in a paired run, it fires the same
  number of interventions at the same moments as the real controller.
- **Offline labels are thin and factual.** The 0.78 rests on about 7
  failing runs out of 53 states. Each state is labelled with the trial's
  actual outcome, not the counterfactual outcome of the other decision.
  Compare wang2026language's point that labels need a stated noise floor.

Caveats:
- The pass-rate intervals are ±11–13 pp, and 14 of the 21 tasks are at
  ceiling, so only the cost contrasts are decision-grade.
- One agent and one decision model.

Rule: evaluate a loop controller end to end against four arms: no
controller, deterministic rules, a placebo at the same moments, and a
cheap general model. Match the placebo's intervention dose to the
evaluation-time controller, not to development data. Report offline
ranking and calibration separately from the online effect, never in place
of it.

## Open questions

- The rule is soft-specified: enforcement relies on the LLM honoring
  the rule in context plus `/lint` as backstop. A project that wants
  stronger guarantees can add pre-commit or CI checks that grep for
  `test/` access in the tool-call log — or, per lu2026meta, move the
  holdout out of the agent's filesystem entirely and gate scoring behind
  a credential issued at chain end.
- The right validation-split size is not specified. Too small and
  every iteration has high variance; too large and the test split
  shrinks. Projects should pick based on task-level noise and
  document the choice.
