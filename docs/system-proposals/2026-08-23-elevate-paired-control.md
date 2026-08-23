---
kind: system-proposal
slug: elevate-paired-control
added: "2026-08-23"
target: "claude/skills/elevate/SKILL.md"
change_type: edit
adds_surface_area: true
evidence_citekeys: [ho2026soundnessbench, tripathi2026diagnostic, ray2026what, wu2026hasbench, ge2026governance]
evidence_strength: "code-released ×2 (ho2026soundnessbench HF dataset, ray2026what GitHub); 5 independent attestations across five literatures; credibility ~3.4 (4/4/4/3/2)"
status: proposed
recommendation: adopt
---

# /elevate should score a known-good control each cycle, so a quiet gate is distinguishable from a broken one

## The change

`~/claude-system/claude/skills/elevate/SKILL.md`, step 4. Current text:

> 4. **Test each candidate against both gates.** Ground it by reading the
>    actual target file in `~/claude-system`. Discard anything that fails
>    either gate; note the strongest one or two discards in the run log so the
>    reasoning is visible.

Proposed — the paragraph above is unchanged, with one added below it:

> 4. **Test each candidate against both gates.** Ground it by reading the
>    actual target file in `~/claude-system`. Discard anything that fails
>    either gate; note the strongest one or two discards in the run log so the
>    reasoning is visible.
>
>    **Then run the paired control.** Pick one proposal in
>    `docs/system-proposals/` with `status: accepted` — rotate oldest-first —
>    and re-test it against the two gates *as they read today*, using only the
>    evidence its `evidence_citekeys` cited at the time. It should pass: it was
>    accepted, applied, and the system is better for it. If it would now be
>    rejected, the gates have drifted strict — say so in the run log and name
>    which gate rejected it. A cycle where the control fails is not a quiet
>    cycle, it is a broken gate, and from the output alone the two are
>    identical.

And step 6's log line gains one field. Current:

> `YYYY-MM-DD HH:MM elevate proposals=<k> considered=<m>`

Proposed:

> `YYYY-MM-DD HH:MM elevate proposals=<k> considered=<m> control=<slug>:<pass|fail>`

Nothing else changes. No new file, no new skill, no new config knob.

## Why (logical case)

`/elevate` is a rejection gate whose stated design posture is that "most
cycles correctly produce zero proposals," guarded by two bars — reputability
and simplicity — both of which are *permissive-failure* guards. Gate 1 stops
weak evidence getting through. Gate 2 stops complexity getting through.
Nothing in the skill measures the conservative failure: a good idea held
back.

That asymmetry has a specific consequence for this skill's output. The
recent record is 0, 1, 1, 1, 3, 0, 1 proposals per cycle — mostly quiet, by
design. But a `/elevate` that had drifted into rejecting everything would
produce *exactly the same artifact*: a `proposals=0` log line with a
thoughtful paragraph on what was considered and held. Step 6 already
requires that paragraph, and it does not help — a block-all gate writes
excellent hold notes, because articulate rejection is what it does. The
existing "note the strongest discards" instruction records what the gate
said no to; it never tests whether the gate can still say yes.

A paired control closes that. The repo already holds seven proposals with
`status: accepted` and a recorded downstream outcome — known-good items, on
disk, free to read. Scoring one per cycle against the current gate text
turns "zero proposals" from an uninterpretable signal into a two-part one:
zero proposals **and** the control passed (quiet week) versus zero proposals
**and** the control failed (the gate has moved).

## Why (reputable evidence)

The concept is `concepts/refusal-cost-symmetry` (status `growing`, five
sources, seeded 2026-08-18). Gate 1 passes on two independent disjuncts —
code-released artifacts and ≥3 independent attestations — with credibility
4/4/4/3/2, comfortably ≥3 on balance.

- **`ho2026soundnessbench`** (credibility 4, code released as a HuggingFace
  dataset) is the closest measured analogue to this skill's job: can a model
  reject an unsound *research proposal* before compute is spent on it. Over
  1,099 ML proposals labeled from ICLR reviewer soundness sub-scores, a
  standard prompt approves 74.0% of unsound proposals while recalling 91.8%
  of sound ones. Told to be strict, the models do not become discerning —
  false approvals fall to 19.9% while recall on good proposals collapses to
  36.1%, and **Macro F1 falls, 54.9 → 49.3**. The headline safety number
  improves fourfold while overall judgment gets worse. Two models reach the
  degenerate corner explicitly: GPT-5.4 and GPT-5.4-Mini post 0% false
  approvals at 0.0% and 0.2% recall on good proposals. They reject
  everything, and on a one-sided metric that reads as perfect. This is the
  failure mode being guarded against, measured on this skill's own task type.
- **`tripathi2026diagnostic`** (credibility 4, ICML 2026 AI-for-Science
  workshop accepted) supplies the *mechanism* — the paired legitimate case —
  and the reason it must be paired rather than merely reported alongside.
  Each of 18 misconduct tasks has an ethical twin holding role, domain,
  artifact and question structure constant, changing only the
  permissibility-determining feature. Models score 71.9 on the violation and
  41.6 on its matched twin (Δ = 30.3); p-hacking inverts by 44.3 points. The
  paper's stated purpose is to "penalize blanket refusal equally as
  misconduct compliance." The design rule this proposal imports is exactly
  theirs: to detect over-strictness you need instances that *should* pass,
  scored by the same gate, in the same run.
- **`ray2026what`** (credibility 4, code released) shows the block-all
  corner is where calibration actually lands, not a hypothetical: conformal
  calibration of 23 judges at the tightest tolerance returns block-all for
  **every single judge**. Its formal framing is the reason a one-sided bar is
  not a safety property — a gate is effective only if both *sound* (never
  permits a violation) and *transparent* (never blocks compliant work),
  because soundness alone is trivially satisfied by refusing everything.
- **`wu2026hasbench`** (credibility 3) instruments the conservative failure
  as a cost rather than an error: its heaviest human-control setting buys
  +50% interaction turns for diminishing-to-negative returns, and the
  tradeoff curve has an interior optimum that only a symmetric metric finds.
- **`ge2026governance`** (credibility 2 — supporting, not load-bearing)
  adds the base-rate argument: at 1% attack prevalence even the best judge's
  precision falls to 22.7%. Cited for direction only; the proposal does not
  rest on it.

What makes this cluster unusually strong for a Gate 1 test is that the five
sources come from five separate literatures — research-integrity benchmark
design, formal enforcement theory, judge calibration, human-in-the-loop
scheduling, and research-proposal triage — and independently find the
conservative failure to be *larger than the permissive one they set out to
measure*. `ho2026soundnessbench` and `ray2026what` reach the identical
block-all endpoint from completely disjoint methods.

## Simplicity assessment

**This adds surface area** — one paragraph of instruction and one log field
— so it owes the higher bar. Stating the cost plainly: `/elevate` grows by
about ten lines, and every cycle does one extra read of a file already in
this repo plus a short re-test. There is no new file, no new skill, no new
hook, no new config knob, and no new dependency. The control set already
exists as a by-product of the skill's normal operation and grows on its own.

Three simpler forms were considered and rejected:

1. **Do nothing; rely on step 6's existing hold notes.** This is the status
   quo and it is precisely what `ho2026soundnessbench` shows to be
   insufficient. Recording what the gate rejected is not a test of whether it
   can accept. A gate at 0% false approvals and 0.2% recall produces a
   fuller, more articulate hold list than a healthy one.
2. **Loosen the gates.** Wrong axis. The measured effect is that strictness
   *inverts* the error rather than reducing it, with the aggregate metric
   falling in both directions from the optimum. Moving the bar without
   instrumenting both sides just trades one unmeasured failure for another.
3. **A separate audit skill or a periodic human review pass.** Net-new
   surface area for something that fits in a paragraph of the skill that
   already owns the decision, and a standalone skill would need its own
   invocation, cron entry, and log. Rejected under this skill's own bias
   toward a smaller system.

The proposed form is the smallest thing that makes the two indistinguishable
outputs distinguishable, which is the whole benefit.

## Risks & what could make this wrong

- **The control set is not independent.** These seven proposals were written
  and accepted *by this same gate*, so "the gate still accepts them" is
  partly circular, and `tripathi2026diagnostic`'s requirement that the
  legitimate case be *superficially indistinguishable* from the violation is
  not met — accepted proposals are not near-misses. This is a **drift**
  detector, not a measure of absolute recall. It catches the gate getting
  stricter over time; it cannot tell you the gate was mis-calibrated from day
  one. That is a real limitation and the proposal should not be read as more
  than it is.
- **The control may pass for the wrong reason.** Re-testing a proposal whose
  evidence base has since grown is easier than the original call. The
  "using only the evidence its `evidence_citekeys` cited at the time"
  clause is there to prevent this, and it depends on the running agent
  actually honoring it.
- **It could become a rubber stamp.** If every cycle logs `control=pass`
  without real re-derivation, the field is noise that costs tokens. Mitigant:
  rotating oldest-first means the same proposal is not re-tested until the
  cycle wraps, and a `fail` is required to name the gate that rejected it, so
  the check has to be actually performed to produce a valid line.
- **It could be unnecessary.** If `/elevate` never drifts, this cost buys
  nothing. The counter-argument is that gate drift is invisible by
  construction — the cost of the check is bounded and known, the cost of
  undetected drift is not — and the recent run history is quiet enough that
  drift and health currently look the same.
- **`refusal-cost-symmetry` records that no source measures the conservative
  failure in a research-agent loop.** All five measure benchmarks,
  guardrails, or triage. The transfer to a weekly curation skill is this
  proposal's inference, not any paper's finding.

## Recommendation

**Adopt.** The gap is real and self-diagnosed — `refusal-cost-symmetry`
names `/elevate` explicitly, and this proposal is written by the very gate
it is about. The evidence clears Gate 1 on two independent disjuncts, with
`ho2026soundnessbench` measuring the failure mode on this skill's own task
type: research-proposal triage, where an aggressive prompt drove Macro F1
down while the headline number improved fourfold. The added surface is about
ten lines in a file that already owns this decision, reusing a control set
the skill produces as a by-product. Adopt with the limitation stated in the
first risk understood: this detects drift, not initial mis-calibration, and
should not be described in the skill as more than that.
