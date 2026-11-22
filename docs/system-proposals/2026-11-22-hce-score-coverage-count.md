---
kind: system-proposal
slug: hce-score-coverage-count
added: "2026-11-22"
target: "claude/rules/evaluation.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [liu2026silent, xia2026does, chen2026when]
evidence_strength: "code-released (liu2026silent, xia2026does, chen2026when); credibility ~3.3"
status: proposed
recommendation: adopt
---

# HCE §4: every score carries its scored count, because an ungraded item looks like a zero

## The change

Add one bullet to `rules/evaluation.md` §4, after the do-nothing preflight
bullet:

> - Write `n_scored` of `n_attempted` beside every score, with the reason
>   for each unscored item. A bad submission scores as a failure; an item
>   the pipeline never graded (timeout, API error, an episode that never
>   reached the grader) is missing, not zero. It sits at the floor, where
>   the do-nothing check is blind. Don't rank arms with unequal coverage
>   until the gap is explained.

Offsets in the same file:

1. "When this rule applies": cut "Projects with no holdout can ignore the
   rule, and `/lint` skips its checks there." The frontmatter
   `description` already says this, and the in-scope list implies the
   converse. (−14 words)
2. §1: cut "`/lint` treats it as a hard failure." "Enforcement" already says
   the clauses are hard and `/lint` is the backstop. (−7)
3. Optional: cut the citation parenthetical in the intro, "(AIRA2, arXiv
   2603.26499, names it one of three dominant bottlenecks)". The reason
   (loops overfit their own signal) stays, and only the provenance goes.
   (−10)

## Why (logical case)

§4 checks the scorer's *value*: the hash shows it is unchanged, and the
do-nothing preflight shows it doesn't reward nothing. Neither checks
*coverage*, meaning whether the scorer ran on every item. An item that
was never graded and is stored as 0 (or quietly dropped from the
denominator) gives a plausible number and raises no error. It also lands
exactly at the floor, so the preflight passes it by construction. In an
`/iterate` comparison, an arm that times out more often, or one whose
episodes end in a way the grader doesn't trigger on, then looks worse for
reasons that have nothing to do with the method. The rule separates two
things that a fail-closed habit merges: a bad submission, which is a
legitimate failure, and a missing grade, which is an instrument fault.

## Why (reputable evidence)

- **`liu2026silent`** (cred 4, code released, HAL maintainers confirmed
  the errors). On HAL's TAU-bench Airline, one scaffold was graded in 51%
  of its episodes against 98% for the other. Ungraded episodes were
  stored as failures. Grading them under the benchmark's own rule takes
  pooled accuracy from 0.30 to 0.43, and a 13-point published gap becomes
  a tie (48 of 351 run-pair orderings reverse). The paper's remedy is
  this one: publish a coverage number next to the score. Caveat from the
  note: the pooled gap closes, but 7 of 9 models still trail.
- **`xia2026does`** (cred 3, code released). The same defect class at the
  stage *before* an LLM judge: a renderer dropped `tool_calls`, leaving 53
  of 60 judged steps blank, and every manifest-hash check still passed.
  It shows that a hash and a staging check are blind to missing inputs,
  which is exactly §4's current coverage.
- **`chen2026when`** independently describes the same τ-bench scoring
  function (replay-into-record, a different failure mode). It
  corroborates that the code *around* a scorer is where silent faults
  sit, not the coverage figure itself.

Box instance: in `arc-agi-2` (HCE), `eval_gen.py` and `eval_heldout.py`
already exclude errored rows, retry them, and print `x/len(ok)`. That is
this rule, followed by local habit. `run_fable_gen.py` in the same
experiment writes an exception as a row with no `solved` field, and its
`metrics.json` reports `n` with no error count. No errors occurred there
(0 errored rows), so no number is known to be wrong. The habit is just
not carried by the rule.

## Simplicity assessment

- No new file, hook, knob or check. One bullet in an existing section,
  plus two fields in metrics files that already exist.
- **Net word change: +64 − 21 = +43**, or +33 with the optional third
  cut. The remaining positive balance is the cost of naming the
  bad-submission/missing-grade distinction. Without it, the bullet reads
  as "never score a crash as a failure", which would loosen §4 against an
  agent that games the scorer by crashing.
- Smaller forms considered:
  - **Append one clause to the preflight caveat** ("…and it cannot see
    items that were never graded"). That is about 10 words, but it names
    the blind spot without giving the fix. Most of the value is the count
    in the metrics file, so this form was rejected.
  - **Put it in `/lint` as a check that `n_scored == n_attempted`.** This
    adds code and only works once the fields exist. Rejected for now. It
    can follow if the fields get adopted.

## Risks & what could make this wrong

- **Weak live target.** Most of the box's evaluators score one artifact
  once (a CSV against AUC), where coverage is trivially complete or the
  evaluator crashes loudly. The defect needs per-item or per-episode
  grading: `arc-agi-2`'s generator arms, `arc-agi-3`, LLM-call scorers.
  The one live instance already follows the habit. If no project grades
  per item, this is 64 words loaded into every HCE session for nothing.
  (Counter: the fault is silent by construction, so not seeing it is weak
  evidence that it's absent.)
- **Liu's setting is public leaderboards comparing scaffolds**, not one
  chain comparing its own arms. The mechanism transfers. The prevalence
  doesn't, and none is measured on the box.
- **Ambiguity at the boundary.** Is an agent's own API timeout a "bad
  submission" or a missing grade? The text puts infra faults in
  "missing", which is right for comparing arms. But it gives a chain a way
  to discard its own failures. A reviewer may want to add "report both
  rates" to close this, though that costs words.
- **"Don't rank" could stall `/iterate`**, if one arm routinely loses one
  item to a flaky API. "Until the gap is explained" permits a logged
  explanation, so the chain needn't stop, but a cheaper reader might treat
  it as a halt.
- **The offsets are not free.** Cut 1 removes the only in-body statement
  that out-of-scope projects ignore the rule. If a reader sees the body
  without the frontmatter, scope is implied only by the in-scope list.

## Recommendation

Adopt. It is one bullet, net +33 to +43 words, and it closes a blind spot
that the existing hash and preflight both miss by construction. The
evidence is credibility 4 with code, and the fix is the one the source
proposes. If the reviewer judges the live target too thin, the fallback
is the 10-word clause on the preflight caveat alone (net about −11 with
the offsets).
