---
kind: system-proposal
slug: budget-ceiling-reserve
added: "2026-08-02"
target: "claude/templates/project/budget.yaml"
change_type: edit
adds_surface_area: false
evidence_citekeys: [ye2026agent, khan2026token]
evidence_strength: "peer-reviewed + code-released (ye2026agent); code-released 63-incident corpus (khan2026token); credibility ~3"
status: proposed
recommendation: adopt
---

# Document ceiling-overshoot semantics and reserve sizing in the budget template

This resolves the follow-up recorded in NOTES.md after the 2026-07-26
run: evaluate khan2026token's pre-flight spend-reservation idea against
the coordinator's halt-after-cycle ceilings. The verdict is that
halt-after-cycle **wins** — reservation is not proposed — but one small
consequence of that verdict is currently undocumented and should be.

## The change

The project template's `budget.yaml` opens:

```yaml
# Hard ceilings for autonomous runs. Chain halts when any is hit.
# Sized for a Max plan box running the latest Opus for both roles.
```

Replace with:

```yaml
# Hard ceilings for autonomous runs. Chain halts when any is hit.
# Enforcement is between actions: a tripped ceiling stops the NEXT
# call, never the one in flight, so real spend overshoots the ceiling
# by up to one worst-case call (40% overshoot measured in the wild).
# Size each ceiling 10-15% below your true tolerance so ceiling plus
# overshoot still lands inside it.
# Sized for a Max plan box running the latest Opus for both roles.
```

No code changes. `scripts/chain_budget.py` already implements exactly
these semantics (it evaluates ceilings between `/iterate` cycles); the
edit makes the semantics visible at the one place users set the numbers.
Existing projects' `budget.yaml` files can pick up the comment whenever
they are next touched — nothing depends on it.

## Why (logical case)

Every ceiling in `budget.yaml` reads as if it bounds spend. It does not:
token consumption is unknown during an LLM call and reported only at
completion, so no orchestration-layer mechanism can stop the call in
flight — a ceiling stops the *next* action. A user who sets
`max_tokens: 5000000` believing 5M is the worst case has an unbounded
misunderstanding encoded in a config file; a user who knows the ceiling
is ceiling-plus-one-call sizes it with margin. The fix is to state the
semantics where the number is set. This also closes out the
spend-reservation question honestly: the reservation design khan2026token
gestures at is impossible against current provider APIs, so the right
move is not new machinery but sizing guidance for the machinery we have.

## Why (reputable evidence)

- **ye2026agent** (peer-reviewed — COINE 2026 @ AAMAS, oral; code
  released; credibility 3) proves the enforcement limit: budgets are
  enforceable only between actions, true pre-flight reservation would
  require provider capabilities (interruptible generation, token
  reservation) that do not exist, and its own runaway-agent experiment
  shows a 40K budget halting at 56K consumed — 40% overshoot by design,
  not by bug. Its deployed systems carry a 10–15% reserve buffer for
  precisely this reason.
- **khan2026token** (code released; credibility 3) supplies the incident
  base: 63 confirmed production budget overruns across 21 frameworks,
  with *no case* in which an overrun was prevented before at least one
  user paid. Its affine-typed budget prevents in-process double-spend
  but, as the concept note records, cannot bound a single call's actual
  cost either — nothing can.

Gate 1 passes on a peer-reviewed source plus a second code-released
attestation; both are direct attestations of the specific claim (the
overshoot is structural), not the general ceilings-are-good claim.

## Simplicity assessment

Negative surface area in every sense that matters: no mechanism, no
knob, no new file — five comment lines replacing two, correcting a false
implication the template currently makes. The rejected alternatives were
all heavier: a `reserve_fraction:` config key read by `chain_budget.py`
(a knob whose only function is arithmetic the user can do while typing
the ceiling), pre-flight estimation in the skills (rebuilding the
reservation that ye2026agent shows cannot be sound), or a paragraph in
`agency.md` (wrong file — the reader who needs this is the one editing
`budget.yaml`).

## Risks & what could make this wrong

- The 40% figure comes from one adversarial experiment; typical
  overshoot on this box is far smaller (one call's usage). The comment
  says "up to one worst-case call," which is exact, and cites the 40% as
  a measured extreme rather than an expectation.
- A user could read "size 10-15% below tolerance" as the system
  auto-applying a margin. The wording ("Size each ceiling…") makes it an
  instruction to the human editing the file, not a described behavior.
- If provider APIs ever ship true reservation, the comment becomes
  stale — an acceptable maintenance cost for two lines.

## Recommendation

Adopt. It is the rare proposal where the evidence is peer-reviewed, the
change is comment-only, and it settles a question the project explicitly
queued for this run. Apply it to the template; optionally propagate the
same comment to this meta repo's own `budget.yaml` at next touch.
