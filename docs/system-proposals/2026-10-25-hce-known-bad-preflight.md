---
kind: system-proposal
slug: hce-known-bad-preflight
added: "2026-10-25"
target: "claude/rules/evaluation.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [chen2026when, zhang2026double, he2026swegate, ding2026reexecuting]
evidence_strength: "code-released (chen2026when, zhang2026double, he2026swegate) + 4 attestations across 4 distinct groups; credibility ~3"
status: proposed            # proposed | accepted | rejected | superseded
recommendation: adopt
---

# HCE clause 4: a pinned evaluator must also fail a do-nothing submission

Clause 4 stops an agent from editing the evaluator. It does not check
that the evaluator is right in the first place. A hash shows the evaluator
is unchanged, not that it is correct. This proposal adds one chain-start
check that covers the gap: score a do-nothing submission and confirm it
lands at the floor.

## The change

`claude/rules/evaluation.md`, §4 "The evaluator is part of the holdout".

Current first bullet:

> - At chain start, record the content hash of the evaluator (the script
>   or function that computes `metrics.json` / `final_metrics.json`) in
>   the experiment's `log.md`.

Proposed: keep that bullet and add one directly after it.

> - Also at chain start, score a do-nothing submission with the search
>   evaluator and log the score next to the hash. Use the task's sample
>   submission, a constant or majority-class prediction, or an empty
>   output. It should land at the task's floor: chance AUC, the
>   majority-class rate, the empty-output score. If it lands clearly
>   above the floor, the evaluator is reading something other than the
>   submission. Fix it before the chain starts. A hash proves the
>   evaluator is unchanged, not that it is right.

Nothing else changes. There is no new file, knob, hook or skill step. The
check uses the validation split only, so it does not touch `test/`.

## Why (logical case)

- **Clause 4 covers tampering only.** The pristine copy run at the final
  pass is the same code that was pinned at chain start. If that code is
  wrong, every cycle ranks on a wrong number and the final pass repeats
  the error. Nothing in HCE or `/lint` would notice.
- **Downstream evaluators are often agent-written.** In MLE-bench-style
  projects the official grader is well tested. But the
  `metrics.json` evaluator for a validation split is usually written by
  `/implement` or the first experiment, not by the benchmark authors. That
  is the setting where the papers below found defects.
- **A do-nothing submission is the cheapest known answer.** Every task
  has one, and it needs no reference solution. Its correct score is known
  in advance, and one evaluator call finds out whether it gets that score.
- **It complements the 10-18 hold.** The `gomez2026can` escalation channel
  was held partly because "§4's hash check already catches the dangerous
  half (editing the evaluator)". This check covers the other half, an
  evaluator that is unchanged but wrong, and adds no new reporting channel.

## Why (reputable evidence)

All four sources are credibility 3 preprints from distinct groups. Three
have released code. Gate 1 passes on the code-released and the
≥3-attestation disjuncts.

- **`chen2026when`** (code: `JunjieChen0/SimGlot`, cred 3). τ-bench's
  `calculate_reward()` returns the right value but leaves the database
  holding the reference solution. The author's harness graded that state,
  which inflated success from 65.5% to 94.3% over 4,320 episodes. The
  harness was untouched, so a hash pin would not have caught it. Its
  known-answer control CTL-1, a do-nothing episode, *passed*, which was the
  defect showing. Across 7 planted defect classes, the controls and
  invariants catch all 7.
- **`zhang2026double`** (code: `yonghongzhang-io/comtrade-openenv`,
  cred 3). A deterministic 100-point scorer gives an empty submission with
  well-formed metadata **0.648**. A fabricated submission scores 0.987, the
  same as the correct one. On τ-bench, 5 of 115 retail tasks are satisfied
  by an empty do-nothing trajectory. Its stated pre-flight is to run canned
  empty and degraded submissions through the scorer and require the correct
  one to beat them all. "Determinism is not criterion validity."
- **`he2026swegate`** (code: `DeepSoftwareAnalytics/SWE-Gate`, cred 3).
  It ships a non-compliant patch next to the gold patch so a test's power
  to discriminate is checked, not assumed. This is the same idea at the
  test level.
- **`ding2026reexecuting`** (cred 3, no code). It treats "gold passes,
  empty fails" as the standard validation for a hermetic grader. Its point
  is that this is *necessary but not sufficient*: a grader that passed the
  check still rejected 6 of 15 correct alternative patches. That limits
  the claim below. It does not weaken the case for the check.

The concept trail is in `concepts/hce-evaluation.md` ("A pristine
evaluator is unchanged, not correct", 2026-10-20) and
`concepts/programmable-evaluator-oracle.md`. The 10-20 curate summary flags
this as an `/elevate` input.

## Simplicity assessment

- **Surface area.** It adds one bullet to an existing clause and one
  evaluator call per chain. No new file, field, hook or knob. Clause 4's
  "soft-specified" paragraph already leaves enforcement to the model and
  `/lint`, so no skill needs to change.
- **Simpler forms considered.** None is simpler than one bullet. The
  sources suggest more, and I dropped it:
  - *A known-good submission must pass* (chen2026when, ding2026reexecuting).
    Dropped: ML-research tasks usually have no reference solution.
  - *Score the same artifact twice* (chen2026when). Dropped: it catches
    stateful oracles, which are rare for file-based ML metrics, and the
    do-nothing check already caught chen's headline case.
  - *A degraded-submission battery* (zhang2026double). Dropped: it costs
    more and its main target is self-report scorers, which HCE projects
    don't use.
  - *Repeat the check at the final-scoring pass.* Dropped: the final pass
    runs the same pinned code, so a code defect shows up at chain start.
    It would also add a second number to `final_metrics.json`.

## Risks & what could make this wrong

- **No downstream incident.** No project on this box has been seen
  shipping a wrong evaluator. The case rests on base rates from other
  benchmarks: τ-bench, ComtradeBench and SWE-bench graders. None of them
  is an ML-research loop.
- **It certifies one failure mode.** `ding2026reexecuting` shows that an
  evaluator can pass "empty fails" and still misjudge real submissions.
  The check catches an evaluator that rewards nothing. It does not catch
  one that misranks real candidates. A passing pre-flight must not be
  reported as "evaluator validated".
- **The floor is not always defined.** For open-ended or discovery tasks,
  "do-nothing" and "floor" can be ambiguous. The bullet should be read as
  "skip with a logged reason" there, not as permission to invent a floor.
  If reviewers expect this often, add those five words.
- **"Clearly above" is a judgment call.** It is deliberately not tied to
  `/iterate`'s noise band, which lives in a different file. A threshold
  would add a knob.
- **The literature here overstates by default.** 76 of 88 PDFs read since
  09-22 came in below their digest entry. These four were read in full,
  and the numbers above are from the notes, not the abstracts.

## Recommendation

**Adopt.** It strengthens HCE: it adds a check and loosens nothing, as the
skill requires for proposals touching `evaluation.md`. It costs one
evaluator call per chain. It closes a gap that two cycles named
(10-18 escalation hold, 10-20 concept section) and that clause 4's hash
cannot close by construction. `evaluation.md` has no other pending
proposal, so this does not stack.
