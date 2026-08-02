---
kind: system-proposal
slug: lint-rule-consistency-pass
added: "2026-08-02"
target: "claude/skills/lint/SKILL.md"
change_type: edit
adds_surface_area: true
evidence_citekeys: [palumbo2026formal, mondl2026autoformalization, ye2026agent, khan2026token, louck2026securing]
evidence_strength: "peer-reviewed in cluster (ye2026agent) + code-released ×3; 7-source typed-enforcement convergence; direct attestations credibility 4 and 3"
status: proposed
recommendation: adopt
---

# Add a rule-consistency pass to /lint's judgment layer

The `typed-enforcement` concept (seeded 2026-07-28, 7 sources) names
this as the one piece of its cluster cheap enough to import at
single-operator scale: "this project's `CLAUDE.md` and `.claude/rules/`
are a natural-language policy that has never been checked for
contradiction, redundancy, or vacuity, and `/lint` checks graph hygiene
rather than rule consistency. That is the concrete proposal this concept
generates."

## The change

Add one bullet to the "Your job after the script runs" list in
`claude/skills/lint/SKILL.md`:

> - **Rule-consistency pass.** If any loaded instruction file (user
>   `CLAUDE.md`, project `CLAUDE.md`, `.claude/rules/*`, `budget.yaml`,
>   `@import`ed rules) changed since the last lint run (`git log` on
>   both repos), read the loaded set as one policy and flag:
>   contradictions between files, the same contract stated in more than
>   one place (drift risk — name the file that should own it), and
>   references to files, skills, scripts, or flags that no longer
>   exist. Report-only, like everything else here; skip silently when
>   nothing changed.

No change to `scripts/kg_lint.py` — semantic contradiction detection in
prose is not a deterministic check, so it belongs in the skill's
existing interpretation layer, not the script.

## Why (logical case)

This box's policy *is* prose: rules, skill files, and CLAUDE.md files
that load into context and are honored by the model. The
instruction-ablation program (accepted 2026-07-31, applied 2026-08-01)
demonstrated empirically what unchecked prose policy accumulates: its
phase 2 found the same root-detection contract drifted across skill
files, HCE clauses restated with variations, Diagnostics keys diverged,
and agency batch scales inconsistent — each a contradiction or
redundancy that no process was positioned to catch, and that took a
deliberate one-off audit to find. That audit fixed the stock but not the
flow: nothing today prevents the same drift from re-accumulating as
files are edited. `/lint` is the system's existing recurring health
check with exactly the right shape (weekly cadence, a deterministic
script plus a judgment layer, report-only), and the pass is gated on
"instruction files changed since last lint," so on most runs it costs
nothing.

## Why (reputable evidence)

The typed-enforcement cluster converges on this from four unrelated
starting points, which is what makes it a concept rather than one
paper's opinion. The two direct attestations of *this specific claim* —
that static consistency analysis of the policy artifact is a first-class
payoff:

- **palumbo2026formal** (credibility 4): chooses Datalog for its
  policies in significant part because it admits "tractable static
  analysis for contradiction, redundancy, subsumption, and conditional
  reachability — a policy author can find out that their rules conflict
  *before* deployment."
- **mondl2026autoformalization** (credibility 3, code released): its
  deterministic hard critic — syntax, schema, vacuity, rule-conflict
  checks — is the load-bearing quality gate of the pipeline; coverage
  and consistency, not soundness, were the binding constraints.

Supporting the cluster's credibility on balance: **ye2026agent**
(peer-reviewed, code), **khan2026token** (code), **louck2026securing**
(code + machine-checked theorem). Gate 1 passes on the peer-reviewed and
code-released anchors with seven attestations.

The honest caveat: the direct evidence concerns policies in formal
languages, where the analysis is decidable; transferring "check your
policy for conflicts before it runs" to a prose corpus checked by an
LLM is an analogical step. Two things ground the analogy. First, the
local evidence above — phase 2's findings are exactly the fault classes
palumbo2026formal's analyses target (contradiction, redundancy), found
by an LLM read, on this box. Second, madatha2026deterministic's warning
that LLM judgment cannot be an *enforcement* layer does not apply: this
is a report-only lint finding a human reviews, the same trust level as
every other judgment call `/lint` already makes.

## Simplicity assessment

Adds one bullet to an existing skill — no new skill, no new script, no
new schedule (rides the existing weekly cadence). Declared
`adds_surface_area: true` because it is a net-new recurring behavior,
and the higher bar for additions is met on three grounds: the simpler
form ("do nothing; the ablation already fixed drift") is refuted by the
drift having happened — a stock fix without a flow check re-pays the
audit cost indefinitely; the change-gate keeps the common case free; and
the alternative implementations (a standalone `/rule-lint` skill, a
deterministic checker for prose, autoformalizing the rules into Cedar
per mondl2026autoformalization) are all strictly heavier forms of the
same idea. This is also the natural companion to the ablation program's
phase 2: that commit made contracts single-sourced; this check is what
notices when they stop being.

## Risks & what could make this wrong

- **False positives.** An LLM reading ~16k words of rules may flag
  deliberate refinements (project rule tightening a global rule) as
  contradictions. Mitigation is the report-only framing plus the
  existing lint instruction to "drop or dismiss noise"; if noise
  dominates after a few runs, the bullet should be narrowed to
  contradictions-and-dead-references only.
- **Scope creep.** The pass could balloon into re-auditing the whole
  skill corpus each week. The bullet's change-gate and file list bound
  it explicitly.
- **Redundancy with /elevate.** This skill also reads system files
  weekly — but for importing literature ideas, not for internal
  consistency; the two do not overlap in what they flag.

## Recommendation

Adopt. The concept nominated exactly this edit, the evidence cluster is
the strongest multi-source convergence in the graph this cycle, the
failure class it targets has already occurred on this box at measurable
cleanup cost, and the form is the minimal one — a gated, report-only
bullet in the skill that already owns recurring health checks.
