---
kind: concept
name: "evidence-gated-completion"
status: growing
added: "2026-08-17"
sources:
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/ding2026autonomous]]"
  - "[[literature/papers/chen2026evigraph]]"
  - "[[literature/papers/apodex2026frontierchallenge]]"
used_by: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/hce-evaluation]]"
related_experiments: []
tags: [safety, harness, evidence, completion, verification, trajectory, enforcement, hce]
---

# evidence-gated-completion

## Definition

"Done" is a verdict the harness reaches, not a claim the agent makes. A
task's completion contract names a small set of **evidence
requirements**; the harness accepts the submission only when it can point
to concrete events in the trajectory that satisfy every requirement and
that a deterministic verifier can check without trusting the agent's
account of them. Absent such a chain, the work is not complete — however
confidently the final message says it is.

## Why it matters here

The default is the opposite. [[literature/papers/ng2026agent]] names it
precisely: "an output-producing harness accepts any terminating trajectory
whose final `model_message` payload says 'done'." That is exactly how
every skill in this project currently terminates. `/ingest` declares the
literature note written, `/promote-moc` declares the cluster unripe,
`/elevate` declares the gates evaluated, `/lint` declares the graph
healthy — and in each case the completion signal is the agent's own
sentence.

The cost of that default is measurable, and not in exotic cases. The
paper's false-completion audit collects 32 documented incidents where an
agent claimed correctness that ground truth contradicted (13 hallucinated,
8 broken, 5 side-effect, 4 partial, 2 reward-hacked), and its cited base
rates are worse than anecdote: 7.8% of *plausible* SWE-bench-Verified
patches fail the developer test suite when actually re-run, 28.6% of
behaviorally different patches were confirmed wrong on manual check, and
15.7% more incorrect patches surfaced across leaderboard submissions. The
striking part is the resolution: **every one of the 32 rows reduces to a
one-element evidence schema** — 8 needed a citation lookup, 7 a test run,
5 a human approval, 3 an external-state check, 1 a screenshot. None
needed a better model.

The audit also shows the field is not missing the artifacts, only the
gate. Of 12 public agent systems, 9 capture file diffs, 11 capture tool
output, 7 capture structured logs — and **2** document a submission gate.
Claude Code, the harness this project runs on, is scored yes/partial on
five artifact dimensions and **no** on the submit gate. The plumbing
exists; nothing consumes it.

## The typed criterion: hard vs soft evidence

The concept only bites if "evidence" is defined so the agent cannot
manufacture it. ng2026agent's criterion is a verifier: let V be a set of
deterministic polynomial-time verifiers, each able to read an event and
**external reference state but not the agent's internal state**. An event
provides *hard* evidence for a property φ if some v ∈ V returns
ACCEPT/REJECT on it; otherwise the evidence is *soft* — its support
depends on the correctness of model-generated content.

- **Soft:** a chain-of-thought trace, a summary of what was done, a
  self-assessment, a claimed diff.
- **Hard:** a test-suite exit code, a commit hash, a content-addressed
  file diff, a citation lookup against a known URL, a database snapshot
  diff, a screenshot diff.

"The contract moves the safety boundary from *do we trust the model* to
*can we verify the artifact*." That is the same substitution
[[concepts/typed-enforcement]] makes for policy and
[[concepts/citation-anchoring]] makes for prose claims — here applied to
the completion boundary itself.

Tamper-resistance comes from structure, not vigilance: the trajectory is a
hash chain (h_i = H(e_i, h_{i-1})), so the evidence chain inherits it and
editing any cited event invalidates every hash after it. Fabricating a
passing test result requires forging the chain, not merely writing a
convincing sentence.

## The other face of the gate

ng2026agent's real structural contribution is the pairing. A harness has
two faces over the same contract:

- **Preventive** — looks *ahead* and blocks: sandboxes, tool whitelists,
  permission gates, scope guards, behavioral monitors. This is
  [[concepts/permission-gate-as-architecture]].
- **Evidential** — looks *back* and refuses to accept: the evidence gate.
  This concept.

Neither substitutes for the other, and the asymmetry is the point: a
preventive layer can be bypassed (jailbreak, injection, an unanticipated
tool path) and the evidential layer still refuses the submission; an
evidential gate cannot undo a `drop database` that a preventive layer
should have stopped. The paper's worked code-patch example runs both —
five preventive layers (sandbox, tool whitelist, filesystem scope guard,
credential-read-then-write monitor, auto-rollback) and a four-event
evidence chain (`file_write` diff content-addressed against the pre-edit
blob, `shell_exec` of the test suite, `tool_result` exit code,
`commit` hash linking them) — so an exfiltration patch must defeat every
preventive layer *and* produce a consistent chain.

The composition is formally cheap only under a condition worth
remembering: monitors compose polynomially when their observation
alphabets are **pairwise disjoint**, and degrade to assume-guarantee
reasoning (exponential in general) when they share events. Real hook
layers share events routinely.

## Independent derivation: failure mode → hiding proxy → minimum evidence

[[literature/papers/ding2026autonomous]] arrives at this concept's core move
from the opposite direction — auditing the AI-scientist literature rather
than cataloguing safety incidents — and lands on the same structure: for each
recurring failure mode, name the **evaluation proxy that hides it today** and
the **minimum evidence** that would expose it.

| Failure mode | Hidden by | Minimum evidence |
|---|---|---|
| Hallucinated citation | fluent prose | resolvable bibliography |
| Novelty overclaim | idea-rating | stated novelty-verification method |
| Weak baseline | the headline metric | baseline provenance |
| Unreproducible run | a single score | seeds, execution traces |
| Result selection | best-of-n | attempt count + selection rule |
| Hidden labor | the word "autonomous" | per-stage human-in-the-loop points |
| Dual use | task focus | safety review |

The "hiding proxy" column is the addition worth importing. An evidence gate
is not only a missing check; it is a check that something *else* is currently
standing in for — and the substitute is always cheaper to produce and more
persuasive than the evidence. Fluent prose is easier than a resolvable
citation; a single score is easier than seeds. That is why the default drifts
toward the proxy without anyone deciding to.

The survey also measures how often the field supplies each disclosure, across
24 runnable systems: human-in-the-loop points stated **88%**, code released
**83%**, attempts and selection policy **67%**, seeds or execution traces
**38%**, novelty-verification method **38%** — with the caveat that
second-coder agreement was lower on the novelty and selection dimensions, so
those two are the softest numbers. The shape matches ng2026agent's 12-system
audit exactly: artifacts are commonly released, and the checks that would let
a reviewer *verify* a claim are not. "Code availability is less scarce than
reproducibility-grade and claim-verification evidence."

For this repo the actionable gap is result selection. A
`max_consecutive_no_improvement` chain is a best-of-n procedure, and nothing
in `/derive-experiment` or the experiment template requires recording n and
the selection rule — the disclosure that targets exactly that failure mode.

## Implementation guidance

1. **Declare the schema per skill, and keep it one or two elements.** The
   paper's evidence that this is affordable is that all 32 false
   completions were catchable by a single check. Candidates here:
   `/ingest` — the literature note exists, its wikilinks resolve, and
   `scripts/kg_lint.py` exits 0; `/promote-moc` — the decline rationale
   cites a mechanically re-derivable count; `/new-experiment` — the six
   required files exist.
2. **Gate on artifacts already produced.** This repo emits git commits,
   `_meta/log.md` lines, and a deterministic linter. Nothing new needs
   capturing; the missing piece is a skill that treats a failing check as
   *incomplete* rather than as a note in the report. `/lint` is currently
   a weekly report — as an acceptance condition on graph-writing skills
   it becomes a gate.
3. **Verify against external state, never against the agent's account.**
   A verifier that reads the agent's summary of the diff has admitted soft
   evidence through the front door. Read the diff.
4. **Degrade gracefully where no schema exists.** Open-ended work
   (synthesis, framing, a MoC's prose) has no checkable acceptance
   standard, and the honest response is to route non-idempotent actions
   to human approval rather than to invent a proxy check. Gate *effects*,
   not thought.
5. **A hard verifier is not an infallible one.** "A flaky test still
   produces wrong gates." The claim is about where the burden of proof
   sits, not about verifier perfection — so a gate needs its own failure
   reporting.

## Connections

- [[concepts/permission-gate-as-architecture]] — the preventive face;
  same contract, opposite time direction.
- [[concepts/typed-claim-partition]] — partitions *claims* by provenance;
  this partitions *completion* by verifiability. The hard/soft criterion
  is the type theory both want.
- [[concepts/citation-anchoring]] — the special case where the evidence
  requirement is a resolvable reference, and the largest single category
  (8 of 32) in the false-completion audit.
- [[concepts/programmable-evaluator-oracle]] — supplies the verifiers;
  the oracle scores a candidate, the evidence gate decides whether a
  submission may be scored at all.
- [[concepts/hce-evaluation]] — HCE hides the holdout so the agent cannot
  peek; this refuses the result until it is grounded. Complementary
  halves of the same integrity property.
- [[concepts/typed-enforcement]] — the schema is a machine-checkable
  artifact held outside the agent's reasoning.

## Open questions

- **Two sources, both unimplemented.** ng2026agent is a position paper with
  no deployed gate and no before/after measurement; ding2026autonomous is a
  survey coding other people's reporting. The convergence is real — two
  independent literatures (safety incidents, AI-scientist audits) derived the
  same failure-mode → minimum-evidence structure — but neither supplies a
  deployed gate with a measured effect, so what is still missing before
  elevation is an *implementation* attestation rather than a third argument.
- What is the false-*rejection* rate of a real gate? Every check that can
  refuse valid work has a cost the paper does not measure, and a gate
  that blocks a correct submission on a flaky verifier is a new failure
  mode, not a removed one.
- Where does the schema live? Per-skill frontmatter, a project-level
  contract file, or the harness's own config are all plausible, and the
  choice determines whether the gate survives a skill rewrite.
- Does an evidence gate change agent behavior upstream, the way
  compliance gating does in
  [[literature/papers/elkoussy2026agentltl]] (where block-and-warn
  regressed two strong models)? Gating completion is a different
  intervention from gating actions, but the same closed-loop caution
  applies.
