---
kind: system-proposal
slug: iterate-redirect-before-halt
added: "2026-10-18"
target: "claude/skills-experiment/iterate/SKILL.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [yan2026traceml, chandran2026autoresearch, zou2026fmlbench, xia2026rrsi, cai2026forkandflush, xu2026conflictguide, zhang2026judged, li2026humanize, he2026beneath, du2026can]
evidence_strength: "peer-reviewed (yan2026traceml NeurIPS E&D, chandran2026autoresearch ICDM) + code-released (yan, zou, xia, zhang, li, he, du) + 10 attestations across ~9 distinct groups; credibility ~3.4"
status: proposed            # proposed | accepted | rejected | superseded
recommendation: adopt
---

# /iterate: make the last cycle before a stall halt a forced pivot

This idea was held on 10-04 and again on 10-11, only because the 09-27
noise-band proposal was still undecided on the same file. That proposal was
accepted and applied on 10-11 (claude-system@60f9f4c), so the hold no longer
applies. The 10-12 and 10-13 ingests then changed the idea's simplest form.
Earlier holds proposed firing on any non-zero counter. This proposal fires
**once, on the cycle just before the halt.**

## The change

One paragraph goes into the `--chain` section of
`skills-experiment/iterate/SKILL.md`, right after **What counts as an
improvement**, plus one optional field on the log line.

Today, step 3 picks the candidate "most directly implicated by
`intended_effect_confirmed` and `delta_from_prior`". In a stall this keeps
proposing the same kind of tweak that is stalling. The chain then halts
when `consecutive_no_improvement` reaches `max_consecutive_no_improvement`
(3 in all 22 project `budget.yaml`s).

Proposed text:

> **Pivot before the stall halt.** When the counter is one below
> `max_consecutive_no_improvement`, the next cycle must change the axis,
> not refine it. In step 3, reject any candidate that keeps the stalled
> cycles' backbone, representation, objective and validation scheme. Pick
> one that changes at least one of them, or reopen a `next_candidate` an
> earlier cycle in this chain passed over. Name the changed axis in one
> sentence. Judge the pivot against the chain's best, under the same band.
> If it also fails to improve, the counter reaches the ceiling and the
> chain halts as before. With a ceiling of 1, or no `budget.yaml`, there is
> no pivot cycle.

Step 7's log line gets ` redirect=<axis>`, on the pivot cycle only.

Nothing else changes: no new knob, no change to `chain_budget.py`, the halt
point, or the keep rule.

## Why (logical case)

- **The stall-halt path ends on a refinement.** The chain spends its last
  cycle proposing more of what just failed twice. The literature below finds
  this is the typical failure: resampling a fixed move set mostly does
  nothing (`bobadillasuarez2026audit`: 69–81% of trajectories never move).
- **It costs no extra compute.** A pivot cycle replaces the third
  refinement. It is not added after it. A chain halts on the same cycle it
  halts on today, so `budget.yaml` ceilings and the `agency.md` hard limits
  are unaffected. Only the content of one cycle changes.
- **Its counter is now trustworthy.** The redirect needs a counter that
  sub-noise "new bests" cannot reset. Without one it never fires
  (`cai2026forkandflush`: a plateau trigger never fired in 15/25 runs). The
  10-11 noise band supplies that counter.
- **Firing once, late, matches the counter-evidence.** Frequent forced
  moves cost score (`li2026humanize`: forced handoffs every k = 1 or 3
  experiments scored below the stronger single agent, while uncapped
  hand-off-at-termination beat every cap). A commitment floor helps
  (`du2026can`). Firing at ceiling − 1 waits for two noise-banded failures
  and fires at most once per chain.
- **The rule is a constraint, not a suggestion.** "Reject candidates on the
  same axis" is checkable after the fact from the logged axis. "Consider a
  different idea" is not, and every source that tested the soft version
  found it ineffective.

## Why (reputable evidence)

Gate 1 passes on all three disjuncts.

- **`yan2026traceml`** (cred 4, **peer-reviewed** NeurIPS 2026 E&D, code;
  CMU). Expert humans pivot on backbone, representation, objective or
  validation on 25% of transitions, while a single-loop Codex agent pivots on
  9%. Humans reopen abandoned lines 9.1% of the time and win 78.5% of those
  returns. Codex reopened one in 658. A prompt self-check ("stuck 3+ steps?")
  went with *less* switching (7.1% → 2.2%). This paper supplies the pivot
  test the proposed text uses, and the "reopen" clause.
- **`chandran2026autoresearch`** (cred 3, **peer-reviewed** ICDM 2026;
  Amazon). Its Criticizer fires on stagnation, and the largest
  single-directive gain followed one firing. The algorithm has no halt
  branch.
- **`zou2026fmlbench`** (cred 4, code; NUS/Tsinghua) and **`xia2026rrsi`**
  (cred 3, code; Google/Stanford). Both wire a stall to a strategy switch
  (RRSI: progress ≤ δ over 3 rounds gives one slot to never-tried
  components).
- **`cai2026forkandflush`** (cred 3; Microsoft Research). This is the first
  matched-compute separation of a controller-level redirect. Normalized means
  are 0.45 for continuing and 0.55 for flush-only, which wins 6/11 tasks.
  Its "pick a DIFFERENT approach" directive did nothing (46% vs 50%). The
  stall-trigger failure is the reason this proposal depends on the band.
- **`xu2026conflictguide`** (cred 3; NUS). A matched branch-point test: an
  enforced scope change beat plain continuation in most of 15 cells at equal
  proposal budget. Probe feedback applied from iteration 0 hurt, which
  supports redirecting late.
- **`zhang2026judged`** (cred 4, code; NTU/NUS). This is the exact shape of
  the counter. Showing the agent its count and asking it to decide fails.
  Masking the action at k = 5 works. A k smaller than the true payoff lag
  costs up to 21 points, which supports firing late.
- **`li2026humanize`** (cred 3, code; Tsinghua/NVIDIA/MIT). It is the first
  source whose arms differ on whether a run may stop: handoff at natural
  termination scored 77.3 against 68.0 alone. It is also the source of the
  "not on a schedule" constraint.
- **`he2026beneath`** (cred 3, code; MBZUAI/McGill) and **`du2026can`**
  (cred 3, code; Cambridge/HKU). He2026beneath adds one more
  controller-not-prompt redirect (CCR/PEM, blind faithfulness 0.41 → 0.67).
  Du2026can supplies the counter-pole: a minimum span before any pivot.

Credibility balance across the ten is about 3.4, with three at 4.

## Simplicity assessment

`adds_surface_area: false`, with one caveat. The change adds one paragraph
to an existing section and one optional log field. It adds no file, hook,
knob or script change, and it reuses the counter and ceiling `/iterate`
already computes. The caveat is that it does add a behaviour: one cycle per
stalled chain is chosen differently.

Simpler forms considered:
- **Fire on any non-zero counter** (the 10-04 form). This has the same text
  length but fires up to twice per stall and goes against `li2026humanize`.
  Rejected.
- **Soft wording** ("prefer a different mechanism"). This is the prompt form
  four sources found ineffective. Rejected.

Heavier forms considered:
- **Enforce the axis check in `chain_budget.py`.** This would need an axis
  label schema in proposals. It adds code. Not proposed.
- **Mechanism-entropy trigger** (`he2026beneath`). This needs a category
  label on every accepted edit plus a windowed statistic. It is net-new
  surface on one paper. Held.

## Risks & what could make this wrong

- **No paper runs redirect against halt.** Every arm above compares redirect
  with *continuing*. `li2026humanize`'s NTA arm is closest, and it is a
  single historical run against 3-repeat means. This is the main reason to
  doubt the proposal. The defence is that the change is compute-neutral: it
  swaps one refinement for one pivot at a point where the refinements have
  already failed twice by more than noise.
- **The rule is prose read by the orchestrator, not a mask.** The evidence
  says controller rules work and prompt nudges fail. `/iterate`'s step 3 is
  the controller for `/propose`, but it is still an LLM reading a skill. The
  `redirect=<axis>` field is the cheap audit: a reviewer can see whether the
  logged axis really changed. If pivots turn out to be relabelled
  refinements, the escalation is the `chain_budget.py` check above.
- **A pivot can erode the champion.** `samanta2026learning` and
  `li2026humanize` both saw redirects overwrite good state. Here the pivot is
  judged against the chain's best and the champion is unchanged, so a failed
  pivot cannot lower the reported result.
- **The noise band may itself bias toward micro-tunings.** `he2026beneath`
  predicts this but did not test it. In that case the stall never registers
  and this rule never fires. That is a limit of the rule, not a harm from it.
- **Usage is thin.** The only downstream `iteration_log.md` (mle-bench) has
  one 3-cycle chain from 2026-04, which halted on `--chain 3`. There is no
  observed stall halt to point to. The case is from design and the
  literature, not from an incident on this box.
- **"Axis" is ML-shaped.** Backbone, representation, objective and
  validation fit ML experiments. A non-ML downstream loop would need its own
  reading. The text says "at least one of them", which leaves room for
  that.

## Recommendation

**Adopt.** The idea has the strongest evidence of anything waiting in the
graph. Two peer-reviewed sources and seven with code agree that agents
under-pivot, and four found the prompt form of the fix ineffective. This
form costs no extra compute, adds one paragraph, and fires at most once per
chain. Its one dependency, a counter that sub-noise gains cannot reset,
landed on 10-11. The missing redirect-vs-halt ablation is real. Because the
change replaces a cycle rather than adding one, the downside is bounded by
the value of a third refinement after two banded failures, which the
literature puts near zero. If you want to be conservative, adopt it and
check the first few `redirect=` log lines to confirm the axis really
changed.
