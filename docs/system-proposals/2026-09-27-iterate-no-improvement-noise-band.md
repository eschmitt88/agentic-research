---
kind: system-proposal
slug: iterate-no-improvement-noise-band
added: "2026-09-27"
target: "claude/skills-experiment/iterate/SKILL.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [zhang2026agora, chan2024mle, yu2026primescientist, xing2026compute]
evidence_strength: "peer-reviewed (chan2024mle, seed variance) + code-released (yu2026primescientist, xing2026compute) + 1 direct measurement of the failure (zhang2026agora); credibility ~3.5"
status: proposed            # proposed | accepted | rejected | superseded
recommendation: adopt
---

# Define "improvement" for /iterate's no-improvement counter

## The change

One paragraph added to `claude/skills-experiment/iterate/SKILL.md`, in
*Before every cycle: budget check*, directly after the paragraph that ends
"…so the user can see the running numbers." Nothing else in the file changes
apart from one field in the step 7 log line.

Current text (the whole of what the skill says about the counter):

```bash
  --experiments-completed <n> --consecutive-no-improvement <n> \
```

Proposed addition:

```markdown
**What counts as an improvement.** `--consecutive-no-improvement` is the
number of consecutive cycles whose metric did not beat the chain's best by
more than the noise band. When either run used `--seeds N>1`, the band is the
larger of the two `std` values its Diagnostics report. A single-run metric
has no band, so a strictly positive delta counts as an improvement and the
log line says `band=none`. Reset the counter only on an improvement beyond
the band. A "new best" inside the band is a coin flip, and counting it
stops `max_consecutive_no_improvement` from ever firing.
```

Step 7's log line changes from `Δ<metric>=<delta>` to
`Δ<metric>=<delta> band=<std|none>`.

`scripts/chain_budget.py`, `budget.yaml` and the template stay as they are.
The script already compares whatever integer it is given against the
ceiling. The fault is that nothing defines the integer.

## Why (logical case)

`max_consecutive_no_improvement` is one of the five ceilings `agency.md`
calls "absolute". `chain_budget.py` enforces it faithfully, but the number it
checks is a chain-local counter the model computes. `/iterate` never says what
an improvement is: the counter is named once, as a CLI flag, and never
defined. So the model falls back on the obvious reading, `metric > best`. With
a noisy metric that reading has a known bad outcome. Any sub-noise gain resets
the counter, so the ceiling cannot trip and the chain runs on until a wall-clock
or token ceiling stops it. By then it has spent its whole budget on noise.

The system already produces the band this needs. `/implement --seeds N`
writes `mean ± std` into `metrics.json`, and the Diagnostics block records
`seeds_run` and `metric_aggregation`. The counter just never reads them. The
edit connects an existing measurement to an existing ceiling.

The one live chain on this box shows the exposure, though at small scale.
`mle-bench/_meta/iteration_log.md` (2026-04-24) crowned a champion on
`Δval_auc=+0.0128`, and it compared a later cycle at `Δ=-0.0003`, both from
single runs (`seeds_run: 1`). Whether +0.0128 clears noise on that validation
split is unknown, and nothing in the log shows that it is unknown. After this
edit the log line would say `band=none`.

## Why (reputable evidence)

- **`chan2024mle`** (MLE-bench, ICLR 2025, peer-reviewed, code released,
  credibility 5). This is the premise: the same config under different seeds
  finds materially different optima (pass@1 16.9% → pass@8 34.1%). It is
  already cited in `/implement` as the reason `--seeds` exists. Single-run
  deltas between agent-built ML solutions are not reliable, and the harness
  has acted on that for months, just not in the counter.
- **`zhang2026agora`** (credibility 3, no code, relevance 5). This is the
  failure mode, measured. Thirteen agents spent five days improving one recipe
  by about 1e-5 bpb per step. The paper's own cross-hardware reproduction noise
  was up to 1.3e-3, two orders of magnitude larger, and the last recorded
  change "moved the score by 9 x 10-6 , below cross-hardware variation."
  The leaderboard compared with a bare `>`, so every sub-noise delta counted
  as a new best. Agora had no no-improvement ceiling, but any stall counter
  over that comparison would have been reset on every step. The paper did
  not run this counterfactual; it is the direct logical consequence.
- **`yu2026primescientist`** (credibility 3, code released). An independent
  group arrives at the same shape: it prunes on `ρ < Q(parent) − δ`, a band
  rather than a bare `>`. It does **not** validate the band: `δ = 0.05` is
  asserted, never derived or ablated. It attests the design choice, not
  that it works.
- **`xing2026compute`** (credibility 4, code released). It finds that
  evolutionary-search papers report single favourable runs whose figures
  "characterize what is achievable on a favorable run, not what a
  practitioner should expect". This is the same noise premise, applied to the
  search-loop setting `/iterate` belongs to.

Gate 1 passes on two disjuncts (peer-reviewed premise and code-released
corroboration), with credibility 5/3/3/4. The concept note
`budget-as-ceiling` records the design rule as "must compare against a
measured run-to-run spread, not against zero."

## Simplicity assessment

This adds no new surface. It adds no file, knob, script or config key, and
`budget.yaml` is untouched. It is a *clarify* edit: it defines a term the
skill already uses and ties it to a measurement the system already produces.
The log-line field makes an existing blind spot visible.

Simpler forms considered:
- **Nothing in `/iterate`, a comment in `budget.yaml` instead.** Rejected.
  The template is the wrong place, because the counter is computed in
  `/iterate` and not in the file that sets the ceiling. The template is
  also blocked behind the pending 2026-08-02 `budget-ceiling-reserve`
  proposal. Earlier cycles held this idea for that reason (09-20
  `hickey2026saltbench`, the `roth2026hack` hold).
- **A `min_improvement` / `δ` knob in `budget.yaml`.** Rejected. It adds a
  config key, and a fixed δ is exactly what `budget-as-ceiling` criticises in
  PrimeScientist: asserted, never calibrated against the evaluator. The
  band should come from the run's own spread, not from a constant.

Heavier form considered and rejected: **require `--seeds ≥ 2` in chain
mode** so a band always exists. That would make the counter noise-aware in
every chain, but it roughly doubles experiment cost and changes how
`/implement` is used, not just how `/iterate` is worded. The reviewer
can choose it deliberately; it should not come in as a side effect.

## Risks & what could make this wrong

- **The single-run case is unchanged in behaviour.** With `band=none` the
  counter still resets on any positive delta, which is the Agora failure.
  The edit makes the exposure visible without closing it. Only
  `--seeds ≥ 2` closes it. This is a known, deliberate limit, not an
  oversight.
- **One std is a crude band.** With N=2–3 seeds the std is itself noisy, and
  "beats best by more than the larger std" is a heuristic, not a test.
  It is still strictly better than zero, and anything more rigorous (a
  t-test, a bootstrap CI) adds prose and arithmetic the skill does not need.
- **It may make chains halt earlier.** That is the point, but a user who
  wants long exploratory chains will hit the ceiling sooner. They can raise
  `max_consecutive_no_improvement`, which is the right knob to turn.
- **Low current usage.** `/iterate --chain` has one recorded chain on the
  box (mle-bench, 3 cycles, April), so the payoff is prospective. That
  keeps the cost of being wrong low too.
- **Champion selection has the same flaw.** It also compares single runs
  with a bare `>`. That is a separate decision about what the chain
  reports, not about when it halts, so it is left out here (one idea
  per proposal). A reviewer who adopts this may want to apply the same
  rule to champion selection.
- **Evidence gap.** No source shows that a noise-band counter *improves*
  outcomes. Agora shows the failure without one, PrimeScientist uses one
  without validating it, and MLE-bench supplies the noise. The case rests
  on the logic plus measured noise, not on a controlled before/after.

## Recommendation

**Adopt.** It is a one-paragraph clarification that defines a counter the
harness already treats as an absolute ceiling. It reuses a measurement the
system already takes, adds no surface area, and lands on a file no pending
proposal touches. The evidence for the premise is strong (peer-reviewed seed
variance, plus a measured five-day sub-noise stall). The evidence for the
remedy is only logical, which is acceptable for a clarify edit this
cheap. If the reviewer wants the counter fully noise-aware rather than only
honest about when it isn't, pair this with a deliberate decision on
`--seeds ≥ 2` for chain mode.
