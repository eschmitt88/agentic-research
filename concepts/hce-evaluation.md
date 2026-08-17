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

6. **If the task is public and the agent can retrieve, close the
   retrieval surface too.** Per wang2026search: disable web search
   during scored runs, or pin retrieval to a fixed offline corpus, and
   log the full search trajectory (queries, retrieved URLs, visited
   pages) alongside `final_metrics.json` so a reviewer can check whether
   a result came from reasoning or from a retrieved answer. Audit for
   *answer-level* leakage, not corpus overlap. A held-out score from a
   search-enabled run on a public benchmark is not an unbiased estimate
   and should be reported with that caveat attached.

7. **Compose the holdout where the task allows, and don't read a
   validation plateau as convergence.** Per zhao2026specbench: prefer a
   held-out set built by composing validation-visible units over a random
   split, so an honest solution predicts a zero gap. And scale scrutiny
   with horizon — `max_consecutive_no_improvement` detects a validation
   plateau, which arrives *before* compliance stops degrading, so a long
   chain that stopped on that criterion warrants a held-out check rather
   than confidence.

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
