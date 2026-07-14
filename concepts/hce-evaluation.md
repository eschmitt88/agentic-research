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

## Open questions

- The rule is soft-specified: enforcement relies on the LLM honoring
  the rule in context plus `/lint` as backstop. A project that wants
  stronger guarantees can add pre-commit or CI checks that grep for
  `test/` access in the tool-call log.
- The right validation-split size is not specified. Too small and
  every iteration has high variance; too large and the test split
  shrinks. Projects should pick based on task-level noise and
  document the choice.
