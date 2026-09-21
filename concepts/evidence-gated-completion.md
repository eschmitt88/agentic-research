---
kind: concept
name: "evidence-gated-completion"
status: growing
added: "2026-08-17"
sources:
  - "[[literature/papers/zhang2026how]]"
  - "[[literature/papers/nepal2026faithful]]"
  - "[[literature/papers/ng2026agent]]"
  - "[[literature/papers/ding2026autonomous]]"
  - "[[literature/papers/chen2026evigraph]]"
  - "[[literature/papers/apodex2026frontierchallenge]]"
  - "[[literature/papers/marsden2026where]]"
  - "[[literature/papers/yang2026truthinsightbench]]"
  - "[[literature/papers/brueckner2026kbench]]"
  - "[[literature/papers/zhu2026claimreceipt]]"
  - "[[literature/papers/ning2026scores]]"
  - "[[literature/papers/zheng2026engineering]]"
  - "[[literature/papers/hickey2026saltbench]]"
  - "[[literature/papers/zheng2026benchshield]]"
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

## The principle now has benchmarks (2026-09-08)

Until this cycle every source under this concept described a *harness*
that gates completion. Three additions supply the missing half —
instruments that measure whether completion was warranted:

- [[literature/papers/yang2026truthinsightbench]] scores the **evidentiary
  maturity of an agent's own claims** across six dimensions and 29
  artifact-grounded items, with no reference answer to match. It names the
  acts that establish warrant and finds them largely absent: **controls,
  robustness, falsifiability, cross-dataset generalization**. That list is
  directly usable as a completion contract's evidence requirements.
- [[literature/papers/brueckner2026kbench]] puts a rate on the failure this
  concept exists to prevent: **overclaiming is the leading failure tag, on
  31.4% of assessments**, measured on real scientific requests with no
  ground truth. It also finds scientific accuracy (6.22) trails
  communication (7.33) in *every one* of nine models — agents clear the
  presentation bar well before the substantive one, which is the exact
  shape of an ungated completion claim.
- [[literature/papers/zhu2026claimreceipt]] makes the gate checkable and
  cheap: **sufficiency** (is the claim recomputable from retained
  evidence?) and **coverage** (do the records span the committed set?) as
  separate verdicts, returning **PASS / INVALID / INCONCLUSIVE** at 0.021%
  of inference time. The three-way verdict is the load-bearing part — a
  gate that can only pass or fail must resolve missing evidence as one of
  the two, and resolving it as *pass* is how unmet requirements get scored
  as met.

The coverage half is new to this concept. Gating asks "is *this* claim
supported"; coverage asks "is the set of claims complete against what was
committed to." Both are needed, and only the second catches a silent
omission.

## A released gate that refuses, aimed at a different target (2026-09-14)

[[literature/papers/ning2026scores]] is the first source under this
concept with a released, executable, LLM-free verifier (`dcp-audit`) shown
**refusing**. It returns `recovered` when a no-history challenger beats the
target on knapsack (0.9363 vs 0.9349), and `audit incomplete` on an
underpowered case even though the observed feedback effect was a perfect
1.0, because positive-control recall LCB was 0.07 against a required 0.8.
Across five cases with declared roles, the verifier matched 5/5.

Three parts carry over:

- **Split INCONCLUSIVE in two.** zhu2026claimreceipt has a single
  INCONCLUSIVE verdict. DCP separates *audit incomplete* (the evidence
  apparatus failed: weak controls, broken interface, a contract violation)
  from *statistical uncertainty* (a complete audit whose interval crosses a
  threshold). They call for different actions: fix the harness vs collect
  more episodes.
- **The attempt ledger is part of the evidence.** "Every started attempt
  remains in the ledger, including timeouts, invalid outputs, and
  infrastructure failures," and a best-of-k procedure counts as *one*
  k-candidate episode. This is the mechanical form of the result-selection
  disclosure named above as this repo's actionable gap.
- **A null verdict needs a positive control.** A challenger's failure to
  recover counts only if the same setup solves known instances when given
  the needed information (45/45, recall LCB 0.889 ≥ 0.8). Otherwise the
  verdict is incomplete, not pass.

The limit matters for elevation. DCP gates *discovery claims after the
fact* at ~500 sessions and ~$60 per audit, not task completion inside a
working harness. Its five cases are designed calibrations with no
false-rejection rate, and it reports no before/after effect on agent
behavior. It attests an implementation of the *verifier and verdict
vocabulary*, not a deployed completion gate with a measured effect.

## A run-acceptance gate inside a harness, with an error rate (2026-09-15)

[[literature/papers/zheng2026benchshield]] gates whether a benchmark run's
score may be accepted, inside a working evaluation harness (BenchFlow). Its
checker is a pure function of a sealed, host-recorded evidence bundle. Two
parts carry over:

- **Report on the environment separately from the run.** The verdicts are
  `Checked`, `VectorExposed`, `AgentViolation` and `Inconclusive`.
  `VectorExposed` means the task admits a forbidden path and this run shows
  no use of it. Neither zhu2026claimreceipt's three-way verdict nor
  ning2026scores's split can say "the gate's own environment is unsound,
  and this submission is still clean." An honest run and an exploit run on
  the same vulnerable task receive the same reward, so collapsing the two
  either convicts the honest run or clears the environment.
- **Read the host's record, not the transcript.** A detector given the task,
  the agent-facing trajectory and the outcome, but no host-side events,
  scored 36.4% on 40 trajectories. That is below chance, so it signals a
  label mismatch as much as missing signal. Host-side evidence plus scoped
  auditors scored 96.0% on the cells they judged. Guidance 3's "read the
  diff" extends to *read the host's record of the diff*.

It also partly prices false rejection. Among 50 honest runs on exploitable
tasks, 3 were falsely convicted and 10 abstained. No replayed exploit was
marked `Checked`; 2 were missed as `VectorExposed` and 8 abstained. A checker
revision re-labels every archived run without rerunning the agent, which
bears on whether a gate survives its own rewrite. The limits are real. The
honest-safe cells are the package's reference solutions. The exploit cells
replay recorded tool streams rather than live agents. And attribution for
the largest hack class comes from an LLM auditor. So this is not yet the
LLM-free completion gate with a measured effect that elevation needs.

## The gate's own instruments fail quietly (2026-09-15)

[[literature/papers/hickey2026saltbench]] is a referee-gated benchmark: a
withheld suite or verifier, run outside the agent's toolchain, decides the
outcome. It is hard evidence in ng2026agent's sense. Its record prices how
the gate's own instruments fail:

- **Unasked is not failed.** Its runner returns non-zero for a cell that
  never booted, so a first correctness tally read five failures. Frozen
  classes (pass / fail / cap-cost / failed-boot / no-build /
  interface-miss, every class printed at zero) cut that to one. That is a
  measured case for zhu2026claimreceipt's three-way verdict: 4 of 5 raw
  "failures" were never asked the question.
- **A guard whose predicate cannot hold looks like a guard in every
  review.** A provenance guard took its fallback on 78 of 78 rows, and the
  fallback read the same bytes, so no number moved. A token watchdog
  crashed on the first path-carrying tool call and the shell turned the
  crash into zero. The remedy is a self-test that makes the caller's exact
  call, plus driving a detector down every branch, "because a quiet failure
  reads as good news."
- **A positive control only covers the population it can see.** This
  extends ning2026scores' "a null needs a positive control." An absence
  sweep with a correctly firing control reported a document missing that
  existed on a remote the working copy had not fetched. The control came
  from the same truncated population. An absence verdict has to name the
  population it enumerated.

## A soft-evidence gate, live in-loop, with a false-reject rate

[[literature/papers/zhang2026how]] runs a terminal verifier inside a working
tau-squared-bench harness on 229 Retail and 58 Airline episodes and reports the
full cross-tabulation. It is the closest thing this concept has to what its
open question asks for, and it is instructive precisely where it falls short.

**What it is.** "The terminal verifier uses the same model identifier as the
executor and reviews the last at most eight user/assistant messages, truncated
to 300 characters each. It receives dialogue text, with no database access,
and has an output limit of 200 tokens." Cost: $0.0079 per episode.

**What it buys.** It rejects 83 of 137 oracle-invalid Retail episodes (61%)
and withholds 16 of 92 oracle-correct ones (17%), cutting the false-pass rate
from a counterfactual 57.21% to an actual 20.96%.

**This is soft evidence by construction, and it still worked.** Under the
hard/soft criterion above, a verifier reading the *dialogue* — the agent's own
account, truncated to 300 characters per message, judged by the executor's own
model — is exactly the front door implementation guidance #3 warns about. It
admitted no external state at all, and it still removed two thirds of the
erroneous acceptances for under a cent. The partition survives (54 of 137
invalid episodes pass it), but "soft evidence is not gradable" now has a
counterexample. The right reading is a cheap soft prefilter *in front of* a
hard check, never in place of one.

**The false-reject rate is a property of the domain, not of the checker.**
Same verifier, same prompt, same truncation budget: 61% catch / 17%
false-reject in Retail, and in Airline "the verifier rejects 25 of 41
oracle-invalid episodes (61%) and seven of 17 oracle-correct episodes (41%)."
Identical catch rate, 2.4x the collateral cost, purely from the domain. Any
false-positive rate imported from a paper is a number about that paper's
domain.

**The stateful cost of a late rejection is named and not measured.** "A
terminal rejection occurs after execution and may leave earlier refunds,
cancellations, or other state changes in place." That is the asymmetry in this
concept's preventive/evidential pairing, stated from inside a stateful
benchmark and still uninstrumented.

Credibility caveat: every Holm-adjusted p in that paper exceeds 0.05, its own
power simulation puts rejection at 0.14, and no code or data is released.

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
   evidence through the front door. Read the diff. Reading external state
   is necessary but not sufficient, though:
   [[literature/papers/zheng2026engineering]] shows a verifier reading a
   second interface backed by the **same upstream lineage** as the agent's
   read approves 62.9-74.2% of unsafe proposals, against 22.9-33.3% for a
   verifier reading an independently replicated source. The external state
   must be reached through an independent failure path, or the verifier is
   re-confirming the agent's stale view.
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
  deployed gate with a measured effect. ning2026scores now supplies a
  released verifier that demonstrably refuses, but for post-hoc discovery
  certification at ~$60/audit, not per-task completion. What is still
  missing before elevation is a completion gate inside a harness with a
  measured false-rejection rate. zheng2026benchshield supplies one for
  benchmark-run acceptance (3/50 false convictions, 10/50 abstentions on
  honest runs), but on reference-solution and replayed cells and with an LLM
  auditor in the attribution path.
  [[literature/papers/zhang2026how]] supplies the closest match yet — a
  terminal completion gate live in-loop on 287 real episodes at $0.0079 per
  episode, with a clean false-reject rate (16/92 Retail, 7/17 Airline) — but
  "the verifier has no repair loop", so its rejections drive nothing
  downstream and the before/after is recomputed on the *same* executions. It
  prices an acceptance margin, not an effect. **What remains unmet is a gate
  whose refusal changes what happens next, measured.** The false-rejection
  rate itself is no longer the blocker.
  [[literature/papers/nepal2026faithful]] does **not** discharge this hold
  either: it is an observational audit with no gate, no intervention arm and
  no before/after behavioural delta, and its one outcome association
  (β = .25) is a single survivor of 32 tests on n = 107 with non-randomized
  exposure. Its contribution is on the hiding-proxy and instrument side.
- What is the false-*rejection* rate of a real gate? Every check that can
  refuse valid work has a cost the paper does not measure, and a gate
  that blocks a correct submission on a flaky verifier is a new failure
  mode, not a removed one. Partly priced in one controlled setting by
  zheng2026engineering: single small verifiers reject 12.1-62.8% of safe
  proposals, and the lowest-risk fixed policy (exact guard, then an
  independent read) reaches 0.8% unsafe only by deferring 56.2% of
  scenarios. Risk targets of 1% and 2% could not be calibrated at all and
  deferred on 99.9%. There, over-refusal is where the frontier sits, not a
  tuning failure. The verifiers were 4B quantized models, so the magnitudes
  may not transfer. hickey2026saltbench adds one small data point from a
  deployed referee (benchmark scoring, not task completion): of 201 scored
  Lean episodes the integrity screen refused exactly one, and that one was a
  complete proof. Three further grader gates were found refusing valid
  treatment work. There, false rejections were invisible until someone read
  the refusals.
- Where does the schema live? Per-skill frontmatter, a project-level
  contract file, or the harness's own config are all plausible, and the
  choice determines whether the gate survives a skill rewrite.
- Does an evidence gate change agent behavior upstream, the way
  compliance gating does in
  [[literature/papers/elkoussy2026agentltl]] (where block-and-warn
  regressed two strong models)? Gating completion is a different
  intervention from gating actions, but the same closed-loop caution
  applies.
