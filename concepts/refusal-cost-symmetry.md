---
kind: concept
name: "refusal-cost-symmetry"
status: growing
added: "2026-08-18"
sources:
  - "[[literature/papers/tripathi2026diagnostic]]"
  - "[[literature/papers/ray2026what]]"
  - "[[literature/papers/ge2026governance]]"
  - "[[literature/papers/wu2026hasbench]]"
  - "[[literature/papers/ho2026soundnessbench]]"
  - "[[literature/papers/zhu2026stopping]]"
  - "[[literature/papers/rahman2026framing]]"
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/typed-enforcement]]"
related_experiments: []
tags: [evaluation, benchmark-design, over-refusal, false-positives, safety, permission-gate, governance]
---

# refusal-cost-symmetry

## Definition

A safety mechanism must be measured against **paired instances of the
legitimate case**, scored so that wrongly blocking correct work costs the
same as wrongly permitting incorrect work. An evaluation that counts only
the permissive failure does not measure safety — it measures conservatism,
and a mechanism that refuses everything tops it.

The paired instance is the load-bearing part. It is not enough to report a
false-positive rate alongside a catch rate; the legitimate case must be
constructed to be **superficially indistinguishable** from the violation,
differing only in the feature that actually determines permissibility.
Otherwise the "legitimate" control is easy and the reported false-positive
rate is an artifact of that easiness.

## Why it matters here

Five sources reach this from five directions — research-integrity benchmark
design, formal enforcement theory, judge calibration, human-in-the-loop
scheduling, and research-proposal triage — and each finds the conservative
failure to be *larger than the permissive one it was measuring*.

**The mechanism, stated as a design rule.**
[[literature/papers/tripathi2026diagnostic]] pairs each of 18 research-
misconduct tasks with an ethical control holding role, domain, artifact and
question structure constant and changing only the permissibility-determining
feature — contamination is documented in the lab notebook, the IRB protocol
was approved, the p-values are maintained. The stated purpose is to
"penalize blanket refusal equally as misconduct compliance, targeting a
failure mode of refusal-tuned LLMs." The pairing is what makes the result
interpretable, and the result is stark: on misconduct classification models
score **71.9 on the violation and 41.6 on its matched legitimate twin**
(Δ = 30.3), and p-hacking specifically inverts by **44.3 points** (86.3 as
misconduct, 42.0 as control). Models read *surface features* — a dropped
sample, an unusual p-value — rather than the procedural context that makes
those features permissible, so the same cue that catches misconduct
over-flags legitimate robustness checks, transparent data exclusions, and
approved human-subjects procedures. The paper's causal claim is available
only because the pair exists: everything but permissibility was held fixed.

**The same paper shows the two failures have different causes.** Explicit
named-authority pressure drives *compliance with misconduct*; implicit
institutional cues drive *over-refusal of legitimate work* — opposing signs
(t = 6.24 and t = −7.86, both p < .001, the second consistent in 18/18
model variants). An aggregate "integrity score" averages a permissive
failure against a conservative one and reports the mean as safety.

**The frontier, formally.** [[literature/papers/ray2026what]] gives the
theoretical reason this cannot be tuned away. A gate is *effective* only if
it is both **sound** (never permits a violation) and **transparent** (never
blocks or alters an action of a compliant run) — and soundness alone is
trivially satisfied by blocking everything, which is why the definition
needs both halves. Empirically, conformal calibration of 23 judges at the
tightest tolerance returns **block-all for every single judge**; at a looser
tolerance the best reaches a .010 miss rate with a **.780 false-block
rate**. Its closed-loop gate cut attack success from .047 to .004 while
blocking **78% of all calls** and dropping task success from .254 to .180.
Over-blocking is not a tuning failure — it is where the frontier sits when
the checker is this good.

**Why it gets worse in deployment, not better.**
[[literature/papers/ge2026governance]]'s base-rate analysis is the sharpest
single number in the cluster: at a realistic 1% attack prevalence, even the
best judge's precision collapses to **22.7%** — three of every four blocks
are false alarms — despite intercepting 93–98.5% of attacks. Catch rate and
usability diverge as the base rate falls, and the base rate in a real
single-operator workflow is very low. This is the argument that the judge
layer must not be the sole gatekeeper.

**The cost is paid in human attention, and it is measurable.**
[[literature/papers/wu2026hasbench]] is the first source here to
instrument the conservative failure directly: its Control Request
Justification metric scores whether authorization requests are "well-timed
and warranted … rather than over-asking on trivial ones," and its heaviest
human-control setting costs **+50% interaction turns for diminishing or
negative returns**, while a mid setting gains +26.9 Safety Rate on
safety-critical tasks. The tradeoff curve has an interior optimum, which
only a symmetric metric can find.

## What this obliges in this repo

The graph's own evaluations are one-sided in exactly the way this concept
describes:

- `/lint` counts violations — orphans, dead wikilinks, sourceless concepts —
  with no paired measure of **correct structure it flags anyway**. A lint
  rule that fires on every legitimate illustrative double-bracket example in
  prose is indistinguishable, in the reported number, from a lint rule that
  works. (The 08-17 run already carried "the 4 shown are pre-existing
  illustrative wikilink text" as a manual annotation — a false positive
  tracked in prose because no metric holds it. Writing *this* paragraph
  produced a fifth one, which is the argument in miniature: the check has no
  way to tell an example from a mistake, and only the count of true hits is
  ever reported.)
- `/curate` counts ingests and declines but nothing measures **wrongly
  declined** items. A curation pass that declined everything would produce
  a clean, fast, fully-resolved backlog.
- `/elevate` "most cycles correctly produce zero proposals" is a stated
  design goal with no counterweight — nothing measures a good idea held
  back. The reputability and simplicity bars are permissive-failure
  guards only.
- The agency verdict gates spend on headroom, and a `hold` that fires when
  work should have proceeded costs a cycle nobody records.

The cheap importable move is tripathi2026diagnostic's, not ray2026what's:
**construct the paired legitimate case**. For `/lint` that means a small
fixture of documents that *should* pass but superficially resemble
violations, scored alongside the ones that should fail — the same file
count on both sides, so a rule that gets stricter cannot improve the
headline.

## The clearest instance: strictness inverts the error, it does not reduce it

[[literature/papers/ho2026soundnessbench]] is the fifth attestation and the
first on a **research** gate rather than a safety one — can a model reject a
methodologically unsound research proposal before compute is spent on it? —
and it exhibits this concept's pattern in its purest measured form.

Over 1,099 ML proposals labeled by ICLR reviewer soundness sub-scores, 12
frontier LLMs under a standard prompt approve **74.0%** of the unsound ones
while catching 91.8% of the sound ones. Told to be strict, they do not
become discerning:

| Prompt | False approvals (bad ideas passed) | Recall on good ideas | Macro F1 |
|---|---|---|---|
| Standard | 74.0% | 91.8% | 54.9 |
| Aggressive | **19.9%** | **36.1%** | **49.3** |

The aggregate metric gets **worse** while the headline safety number
improves fourfold. And the degenerate corner is reached explicitly: GPT-5.4
and GPT-5.4-Mini land at **0% false approvals with 0.0% and 0.2% recall on
good proposals** — they reject everything. That is the block-all rule
[[literature/papers/ray2026what]] shows conformal calibration actually
returns for every judge it tested, arrived at independently from a
completely different literature.

**Scale does not help and may hurt.** Within one model family from 2B to
122B under standard prompting, recall on good proposals rises (71.8% →
92.8%) while recall on bad ones *falls* (31.0% → 19.2%): "larger models
become more permissive toward weak proposals, not less."

**Models see blatant flaws and miss subtle ones.** Injecting severe
hypothesis–experiment mismatches into 100 sound proposals drops one model's
approval rate from 77.0% to 1.0%. So the failure is not inattention to
methodology — it is insufficient criticality toward flaws that look normal,
which is the same surface-cue dependence tripathi2026diagnostic measures
from the other side.

## What this obliges of `/elevate` specifically

The obligation this concept states generally has a named target now.
`/elevate`'s design posture is that "most cycles correctly produce zero
proposals," guarded by two bars (reputability, simplicity) and nothing
measuring what was wrongly held back. That is an aggressive prompt in
skill form, and SoundnessBench is the measurement of what an aggressive
prompt does to research judgment: near-uniform rejection, with the
aggregate quality metric falling while the safety number improves.

The cheap fix is this concept's standard one — construct the paired
legitimate case. Score a handful of **known-good historical proposals**
alongside each cycle's candidates, so that "zero proposals this week" is
distinguishable from "the gate is now rejecting everything." Without that
control, a correctly-quiet cycle and a broken gate produce identical
output.

**A second, structurally different lever on the same problem.**
[[literature/papers/zhu2026stopping]] attacks the false-approval/
false-rejection tradeoff from **panel composition** rather than
threshold strictness — which of several available judges to call, drop,
or route, holding each judge's own strictness fixed. Its redundant-copy
stress test (adding four exact-copy judges leaves a well-designed
policy's risk and cost unchanged, while a naive full-call jury gets both
more expensive and *less* accurate) is the composition-side analogue of
this concept's core claim: an evaluation mechanism should be judged by
what it does under conditions designed to break it cheaply, not by its
headline number on the easy case. Worth weighing against the
paired-control fix above before `/elevate`'s proposal is implemented —
they are not mutually exclusive, but a panel-routing policy is a
different (and in principle cheaper, since it needs no dedicated control
set) way to buy the same robustness the paired control targets.

## Connections

- Sharpens [[concepts/hce-evaluation]] along a different axis. HCE protects
  the *estimate* by hiding the answer from the search loop; this protects
  the *construct* by ensuring the score cannot be gamed from the
  conservative side. They compose: a hidden test set scored one-sidedly is
  still gameable by refusing.
- Answers a standing open question in
  [[concepts/permission-gate-as-architecture]] ("False-refusal cost … the
  tradeoff curve for automated gates remains uncharacterized"). It is now
  characterized in three places — ray2026what's 78% gating fraction,
  ge2026governance's 22.7% PPV at 1% base rate, wu2026hasbench's +50%
  turns — and the answer is that the cost is large, not marginal. Worth
  folding back into that concept's open questions as partially resolved.
- Constrains [[concepts/programmable-evaluator-oracle]]: a deterministic
  oracle is preferable partly *because* its false-positive behavior is
  inspectable and stable, where a judge's moves with the prompt. But
  determinism is not immunity — a deterministic rule can be systematically
  over-strict, and only a paired control detects that.
- Relevant to [[concepts/typed-enforcement]]'s open question on
  autoformalized policy: mondl2026autoformalization "explicitly evaluates
  coverage rather than utility, which leaves the failure mode most likely
  to sink the approach unmeasured." That failure mode is this one.
  [[concepts/constraint-pinning]] is the counter-example worth keeping —
  it reports restoring 0% violation "with no over-refusal cost," which is
  the right thing to report and rare.

## Open questions

- **Is the symmetric penalty the right weighting?** All four sources score
  the two failures equally, which is a modeling choice, not a finding. In a
  research-agent loop the two errors have genuinely different costs (a
  wrongly-permitted fabricated result may be unrecoverable; a wrongly-blocked
  experiment costs a retry), and nobody has proposed a principled ratio.
- **Does the surface-cue inversion have a deterministic analogue?** The
  effect is documented for model judgment. Whether a deterministic checker
  exhibits the same "the cue that catches the violation over-flags the
  legitimate twin" pathology — or whether it merely has a fixed, knowable
  over-strictness — is untested and matters for whether determinism is a
  real mitigation.
- **No source measures the conservative failure in a *research* agent
  loop.** The evidence is from benchmarks, guardrails, and human-approval
  scheduling. The case this project depends on — an autonomous curation or
  experiment loop that quietly declines good work — is uninstrumented
  everywhere, including here.
