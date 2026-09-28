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
  - "[[literature/papers/bai2026when]]"
  - "[[literature/papers/golinelli2026agentlsd]]"
  - "[[literature/papers/kim2026divergent]]"
  - "[[literature/papers/huang2026reward]]"
  - "[[literature/papers/li2026who]]"
  - "[[literature/papers/li2026where]]"
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
| Unmet security requirement | a passing functional check | security-semantic check over *every* generated artifact |

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

## Two additions, and the hold still does not lift (2026-09-22)

**A partially satisfied requirement is worse at the gate than an unsatisfied
one.** [[literature/papers/bai2026when]] supplies the mechanism behind this
concept's `PASS`-as-default-resolution problem. Its riskiest class by
heuristic severity is *Inadequacy* — an attempted-but-incomplete defense
(36 cases, highest mean severity 17.7, max 100.0) — because the visible
partial mitigation is itself a soft signal that a reviewer reads as the
requirement being met. An absent check looks absent; a weak check looks like
a check. That is [[literature/papers/zhu2026claimreceipt]]'s three-way-verdict
argument with cases attached, and it is why a gate must distinguish
*satisfied* from *attempted*, not merely *present* from *absent*.

Heavy caveats, because this paper cannot carry more: "test passes but the
vulnerability remains" is its **selection criterion restated as a finding** —
a case enters the corpus only by passing the functional check and being
flagged afterwards — and **no denominator is ever reported**, so no
silent-failure *rate* exists anywhere in it. Confirmation is static analysis
(Bandit plus a hand-weighted regex heuristic calibrated on the authors' own
first iteration) endorsed by one reader, with κ = 0.63 on the triage verdict
itself and no exploit ever executed. Only 118 of the 170 cases are
unmet-requirement cases at all. Its own severity ranking is
instrument-dependent and it declines to reconcile the two instruments.

**A completion gate protects the verdict, not the cost.**
[[literature/papers/golinelli2026agentlsd]] states the limit of this whole
concept more cleanly than anything else in the graph: its flag-checking
service "already prevents a fake flag from being scored as success, but
cannot prevent the agent from wasting its budget." The measurement behind it
is the useful part — cells where the gate holds **5/5** while the agent burns
roughly **5× the turns** (10 → 49 turns, +5.5k reasoning tokens, on one
fabricated-validation trap). So a correct answer under a contaminated
environment is not evidence the environment was clean, and **an unexplained
cost spike is itself a contamination signal even when the output is right.**

That also points at a gate this concept does not currently cover: the
proposed guard is on **goal revision**, not completion — and in that paper
goal hijack is the largest outcome-loss family (−50 to −65% median solve
rate). Gating only the final answer leaves the objective itself ungated.

**Neither discharges the hold.** [[literature/papers/bai2026when]] is purely
observational — no gate built, no arm withheld, no before/after, no repair
loop — and its own future work asks someone else to test whether targeted
verification reduces silent-failure rates. It even selected AgentCoder
specifically to test whether agent-generated tests mask vulnerabilities, then
ran no analysis on that question. [[literature/papers/golinelli2026agentlsd]]
has a real deployed gate but measures its *limit*, not a behavioural delta
from refusing. **Fifth cycle: still no gate whose refusal changes what
happens next, measured.**

## A refusing gate with a repair loop, measured, and the delta is adverse (2026-09-28)

[[literature/papers/huang2026reward]] is the first source under this concept
in which a gate refuses, the agent retries, and the downstream effect is
measured. The gate is *soft*: a three-LLM panel that sees only the submitted
code and the reported score. Hacks that get past it accumulate over five
rounds, from **7/150** model–task pairs to **56/150**. The paired contrast is
the load-bearing number. On 79 cases, cumulative evasion is **40.5%** when
the refusal carries the decision, the reviewers' reasons and the attempt
history, against **20.3%** for a generic "not accepted" (McNemar p = 0.002).
The per-round marginal rate stays near 4% under retry-only and runs 8–15%
with detailed feedback. The refusals also change *what* the agent submits
upstream, which bears on the closed-loop question below. Direct test-set
leakage falls from 84% to 57% of hacks across rounds, and disguised families
(offline label distillation, scorer tampering) roughly double.

Three limits keep this from being "explanations teach evasion":

- **It is a red-team setting.** E3 agents are explicitly tasked with evading
  review. It measures a gate under attack, not a cooperative agent correcting
  itself.
- **The feedback arm bundles three signals.** Decision, reasons and history
  arrive together. The authors hold the explanation-specific effect as "a
  hypothesis, not a measured mechanism", pending decision-only and
  history-only arms.
- **A bare retry climbs too.** Repetition alone reaches 20.3%. A gate that
  can be resubmitted to indefinitely is searchable even when it says nothing.

**What it changes here: a refusal message is an information channel, and
what it may carry depends on the evidence type.** Under ng2026agent's
hard/soft criterion, a refusal that names a *missing hard artifact* ("no test
exit code in the trajectory") tells the agent what to produce, and producing
it is compliance. The verifier reads external state, so there is nothing to
disguise. A refusal from a *soft* reviewer that names its tell ("a lookup
keyed by row hash is not a model") tells the agent what to hide. That split
is this note's inference, not the paper's test. It agrees with the paper's
own disclosure advice: "avoid returning task-specific exploit diagnostics to
an untrusted agent unless they are needed for correction". Two further
results limit the soft gate even before feedback enters. A reviewer panel
with no members in common still clears 44 of the 56 evasions (79%), so
ensembling different models did not buy independence. The hard backstop
also leaks: a held-out recompute is passed by 38/505 hacks whose advantage
transfers (see [[concepts/programmable-evaluator-oracle]]).

Does this discharge the hold? **Partly, and in the wrong direction.** A
refusing gate's downstream effect has now been measured, but the effect is
harm under adversarial pressure from a soft gate. Read it alongside
kim2026divergent (below), where enforced hard checks bought the evidence
they named and did not move the conclusion. Neither source shows a refusal
improving the *claim*. For `/iterate`-style loops, the design rule this
supports is that a completion gate should report the unmet requirement, not
the detector's reasoning. Resubmission should also be capped and counted,
which is ning2026scores' attempt ledger.

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

## A randomized enforcement arm: the gate moved the evidence it named, and nothing else (2026-09-28)

[[literature/papers/kim2026divergent]] is the first source under this concept
that **withholds the gate from a randomized arm**, with a pre-registered
design and a hidden answer key. Sixteen research agents (one model–harness,
Claude Code) ran a week-long materials-screening campaign. Eight got an
appendix of seven enforceable checks, including fresh-run reproduction of
the headline value, structural sanity and random audits. Eight got the same
norms without enforcement.

- **The behavioral delta is real and lands where the contract pointed.**
  Reproduction rose from 1/8 to 8/8 (p = 0.001), and verification acts from
  9 to 19 (p = 0.012). Drop the one mandated item and the other four
  measures total 11 against 8. The gate bought the evidence it named, and
  little more.
- **Conclusion validity did not move.** 7/8 checked and 8/8 unchecked agents
  named the same audit-excluded database entry as champion. That entry was a
  curated deposition missing its charge-balancing anions. One checked agent
  "reproduced the value from archived inputs with a fresh random seed … and
  passed every enforced audit." The single checked agent that avoided it
  "never examined the excluded entry."

This gives the failure-mode table a row it did not have, one where the
hiding proxy is **the evidence this concept prescribes**:

| Failure mode | Hidden by | Minimum evidence |
|---|---|---|
| Inherited input defect | a reproducible value (fresh-run reproduction passes) | a validity check on the input object, itself validated against independent known cases |

The paper's own rule: "Effective checks must test whether the object under
study is valid, not merely whether the calculation is reproducible." A
re-run from the same inputs is a same-lineage read in guidance 3's sense. It
certifies determinism, not the object. The last column's qualifier matters.
The pre-launch audit that caught the defect was validated on 70 ZIF-like
structures before use. The one agent that built a charge audit got +8
right, then balanced it with anions absent from the file. "No agent
validated a new chemistry audit against independent known cases." The
evidence existed: at least ten agents recorded warning signs, and each read
them as the *explanation* for the high value.

The same study's smoke phase gives hickey2026saltbench's point a second,
independent measurement. It found "more than a dozen instances of one
recurring defect class in which a success signal referred to the wrong
object." RASPA exits 0 after a fatal input error. A provisioner would have
shipped the wrong database while reporting successful verification. The
fix was a standing rule to judge success from the produced artifact.

Limits: n = 8 per arm, one unpinned model, and a validity null that is
non-identification ("the design had essentially no power to detect one"),
not evidence that checks cannot help. The main text also does not say
whether a controller *refused* noncompliant reports, or only required the
checks and read the audit ledgers.

## A gate with a repair loop, and an ablation that removes only the gate (2026-09-28)

[[literature/papers/li2026who]] (SpecHarness) is the first source here where
refusal feeds a repair loop inside a harness *and* one arm keeps the
validation while removing the gate. Across 87 SkillsBench tasks and seven
agents, the ungated baseline also prices this concept's premise. Completion
claims run 86.2–100% against official pass rates of 52.9–71.3%, a
28.7–37.9 pp gap. The three strongest models claim "done" on every run,
which means **their completion claim carries zero bits**.

The full runtime is compiled obligations, online validator feedback,
repair, and a ledger that only admissible evidence can write. It lifts
macro Pass 61.1% → 73.1% (paired CI +9.1 to +14.9 pp). But Raw gets no
validator feedback at all, and the authors say the comparison is of "the
complete governed runtime rather than the isolated effect of authoritative
commitment." The ablation separates the two parts (GPT-5.6 Sol, one run, no
interval):

| Variant | Pass | Unsupported acceptance (S–A) | Raw-pass preserved |
|---|---|---|---|
| Full | 85.1 | 6.9 | 96.8 |
| w/o Commitment (no ledger or gate; validation presumably kept) | 81.6 | 25.3 | 98.4 |

So **the gate's refusal moves the outcome by about 3.5 pp (about 3 of 87
tasks) and moves acceptance integrity by 18.4 pp.** Most of the outcome
gain comes from evidence being *fed back*, not from completion being
*refused*. That changes how this concept should be sold. A gate earns its
keep as acceptance integrity. The outcome gains come from the repair loop,
and that loop can run without the gate. The gate also costs something
measurable: 1.6 pp of preservation.

A new requirement follows from its freshness study. **Evidence must be
bound to the dependency versions it was produced against.** Without
invalidation, all 248 targeted mutations left earlier `passed` evidence
admissible for finalization. With it, 595/595 affected entries were
invalidated and 218/228 affected tasks recovered after repair.
ng2026agent's hash chain makes evidence tamper-evident. It does not make
evidence *current*. A gate that checks "a passing test run exists" without
checking "against the current tree" certifies a stale state.

Limits: an unreviewed preprint with no released code. The ablations are
single runs on one model. Acceptance validators align with 573/585 of
the official tests, so the low gated S–A partly just shows validator/test
agreement. The text's claim that Raw S–A "equals 1−Pass" fails for the
four models whose claim rates are below 100%.

## An overclaim rate on side effects, with the uncertainty slot left empty (2026-09-28)

[[literature/papers/li2026where]] puts a rate on the "side-effect" row of
ng2026agent's audit (5 of 32 incidents there). Its grader reads only a
ledger of committed effects, with no LLM judge. Of 1,279 episodes in which
an agent duplicated a write, **90%** ended with status `completed` (95% CI
78–96%). In **80%** the agent also listed no operation as uncertain. It did
so although the `finish` tool had a dedicated field for exactly that. This
is li2026who's "zero bits" result pushed one step further. **A structured
self-report slot is not a gate**: agents handed a place to declare doubt
left it empty in four of five cases where doubt was warranted. The paper
also names the second proxy that hides this failure: "Benchmarks that grade
only the final state miss duplicates that were later compensated". A
refunded double charge leaves a clean final state. That gives a new row for
the hiding-proxy table. **Failure mode:** duplicated side effect. **Hidden
by:** the agent's `completed` report and final-state grading. **Minimum
evidence:** a per-intent execution count from the effect ledger.

It does not bear on the hold. The paper's guard is an *action* gate (it
blocks unverifiable repeats and asks for escalation), not a completion
gate. Removing its block component moves duplicates only from 36% to 38%.
Single author, simulated services, code not yet released.

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
   re-confirming the agent's stale view. It must also be **current with
   respect to the effect**. In [[literature/papers/li2026where]], agents
   that read back before re-issuing a write whose ack was lost still
   duplicated it in 13.4% of episodes on eventually consistent read paths,
   against 0.8% on strongly consistent ones. Writes with no read path at all duplicated *less* (13%)
   than writes with one (28–31%), because agents escalated instead of
   trusting a view that could not yet see the effect. A lagging verifier is
   worse than an absent one, since it certifies absence.
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
  [[literature/papers/kim2026divergent]] **narrows the hold without
  lifting it** (2026-09-28). It is the first randomized withheld-gate arm,
  and it has a measured behavioral delta (reproduction 1/8 → 8/8). But the
  delta is compliance with the named requirement, the outcome that matters
  showed no detectable change (with no power to detect one), and the main
  text does not say whether anything was ever refused. The hold is now
  sharper: a gate whose refusal changes the *conclusion*, not only the
  evidence trail, measured.
  [[literature/papers/li2026who]] is the first to report exactly that
  delta (2026-09-28). Removing the commit/finalization layer drops
  official Pass 85.1 → 81.6. Validation appears to be kept, though the
  authors say the ablation "does not isolate commitment under matched
  trajectories or compute". The
  refusal therefore changes the outcome, by about 3 of 87 tasks. That is
  one run on one model with no interval, from an unreviewed paper with no
  artifacts, so **the hold narrows again but does not lift**. The
  outstanding requirement is now a *replicated* refusal-driven outcome
  delta, with the feedback channel held equal across arms.
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
