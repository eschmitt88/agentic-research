---
kind: system-proposal
slug: instruction-ablation-program
added: "2026-07-31"
target: "claude/skills/** prose corpus; coordinator admission layer (plan, jobs/decisions/session_caps, pretooluse_cap.sh); rules loading scope"
change_type: removal + consolidation (phased)
adds_surface_area: false
evidence_citekeys: [khan2026token, madatha2026deterministic, wu2026hasbench, hambardzumyan2026aira]
evidence_strength: "primary evidence is box-local usage telemetry (3 months: state.db row counts, skill-mention grep across 18 projects, git churn) — stronger than any paper for a removal decision; literature corroborates direction (khan2026token code-released cred 3; wu2026hasbench cred 3; madatha2026deterministic cred 2, prevalence only)"
status: accepted
recommendation: adopt-in-phases
provenance: user-directed review (2026-07-31, Fable 5 session), not /elevate-gated — recorded here so the reasoning and the decision live in the same queue
---

# Instruction ablation: move invariants to code, delete the dead admission layer, single-source the contracts

## Framing

Anthropic's internal practice for system prompts — delete everything,
ablate pieces back in, keep only what the model still needs — cannot be
run literally here (no eval harness, no budget for N×2 comparisons).
This proposal substitutes the two next-best signals we do have:

1. **Three months of usage telemetry** as the ablation read-out: what
   was actually exercised is load-bearing; what never fired is not.
2. **This repo's own literature** as the design prior: deterministic
   code for hard invariants, model judgment for everything graded —
   and *no new hard machinery* at single-operator scale
   ([[concepts/typed-enforcement]], [[concepts/budget-as-ceiling]]).

The instruction corpus is ~18,400 words across CLAUDE.md, 2 rules, 20
skills, and templates. The 2026-07-31 review found it splits roughly
into: hard contracts (schemas, paths, invariants — keep), rationale
(the *why*, usually one citation-anchored line — keep), step-by-step
procedure (how to do things a frontier model does unprompted — delete),
and deterministic algorithms written as English (regex checks, date
arithmetic, halting grammars — convert to code). Target: ~7k words,
nothing load-bearing lost, with the mechanical checks *more* reliable
than today because they stop depending on in-context prose execution.

## Phase 1 — delete the coordinator admission layer (pure removal)

**Remove:** the `/plan` skill, `plan_cli.py`/`job_cli.py`, the `jobs`,
`decisions`, and `session_caps` tables, and `pretooluse_cap.sh` (plus
its settings.json wiring).

**Keep untouched:** the hardware poller, `token_events` + the Stop-hook
meter (just fixed, 2026-07-26 proposal), `/headroom`, the agency
verdict (`agency.py`, ccusage-based), and the dashboard.

Evidence this is dead weight, not latent capability:

- In 3 months: `jobs` has **1 row ever**, `decisions` **0**,
  `session_caps` **0**. `/plan` appears once in any project log.
- `pretooluse_cap.sh` has executed a SQLite query on **every tool call
  since April** to check a table that has never contained a row.
- The signals that actually govern autonomous spend — the agency
  verdict and `budget.yaml` ceilings read by `/iterate` — never touch
  this layer. The box ran `agency: max` bursts for weeks while `/plan`
  was deferring 100% of jobs, and nothing noticed.
- `khan2026token`: on single-agent workloads a 4-line counter matches
  the affine-typed budget system at 0/30 overshoot. The box's
  counter is `budget.yaml` + `/headroom`; the admission gate is the
  multi-tenant machinery the paper says single-operator boxes don't
  need. `typed-enforcement`'s open-questions section reaches the same
  conclusion independently.

The 2026-06-28 and 2026-07-19 holds on permission-gate hardening cited
"already enacted (PreToolUse cap + coordinator admission)" as a reason
not to add more gate surface. This phase updates that picture honestly:
those two mechanisms were decoration (empty cap table, always-defer
gate). Deleting them does not weaken a control that was never live.
If multi-project parallel autonomy ever materializes, the layer is one
`git revert` away.

## Phase 2 — single-source the contracts (text-only)

- **Project-root detection**: delete all ~15 prose restatements of
  "nearest ancestor with CLAUDE.md and `_meta/`" from skills. The
  model finds project roots without instruction; the three hooks that
  need it mechanically already implement it in bash.
- **HCE boundary**: `evaluation.md` is the single statement; the seven
  skills that currently *restate* it switch to a one-line
  `respects:` reference (the `/elevate`→`/promote-moc` cross-reference
  pattern, which the review found is the only place reference-not-
  restatement is practiced).
- **Diagnostics schema**: one canonical definition (in
  `implement/SKILL.md`, since its subagent writes it); fix the key
  drift (`/wrap` says `diagnostics.leakage_check`, `/implement` says
  `leakage_check`); `/wrap`, `/new-experiment`, `/lint`, and
  `session-end.sh` point at it.
- **Agency batch-size table**: lives only in `agency.md`; `digest`
  references it (currently `digest` has a 3-tier table the "canonical"
  rule lacks — resolve in `agency.md`'s favor by completing it there).
- **Experiment file-set and worktree rule**: stated once
  (template scoped rule), referenced elsewhere.

Rationale beyond word count: every restatement is a copy that can
drift, and three of the four listed have *already* drifted. Redundancy
in a prose system is not belt-and-suspenders; it is a divergence
generator.

## Phase 3 — compile prose-programs into scripts

- **`/lint`** (1,334 words, 13 checks): all filesystem/regex/date/DVC
  checks move to one Python script (`scripts/kg_lint.py`) emitting a
  findings report; `SKILL.md` shrinks to "run it, then interpret,
  prioritize, and recommend actions." The HCE hard-failure check
  becomes deterministic — which is what `evaluation.md`'s "the
  file-system and lint checks are the backstop" already promises but
  prose execution cannot guarantee. Judgment-shaped checks (MoC
  ripeness, consolidation signals) stay with the model, fed by the
  script's cluster counts.
- **`/iterate` halting**: the `<key>:<op>:<value>` grammar and
  budget-arithmetic tables become a small helper the skill invokes;
  the SKILL.md keeps the semantics ("any condition halts; subagent
  hard failure always halts") and drops the parser spec.
- **`/new-project`**: the embedded command sequence becomes an actual
  script; the skill keeps the decisions (naming, `--private`,
  graduation pointers).

Direction is straight from [[concepts/typed-enforcement]]: "the
enforcement layer must be ordinary testable code... not further LLM
orchestration." The token-meter incident is the cautionary tale in the
other direction — mechanical layers need tests; each extracted script
should carry a smoke test the way `kg_lint.py` trivially can.

## Phase 4 — one confirmation principle, stated once

Today three regimes coexist with no stated rule: unconditional
auto-commit (`discover`, `fetch-paper`, `ingest`, `wrap`,
`new-experiment`), unconditional confirm (`propose`,
`derive-experiment`, `sync-imports`, `mle-task`), and correctly
agency-branched (`digest`, `curate`, `promote-moc`, `iterate`).
`agency.md` names `/propose` as governed by it; `/propose` step 6
hard-codes a confirm gate that never branches. Proposed principle, one
paragraph in `agency.md`, skills reference it:

> Artifact writes (ingesting sources, notes, scaffolds, logs) commit
> autonomously everywhere. Hypothesis selection (proposals, ensemble
> choices, MoC promotion) confirms under `standard` and auto-advances
> under `max`.

`wu2026hasbench` quantifies the cost of over-gating (+50% interaction
turns for diminishing or negative returns); the 2026-07-26 hold noted
the anti-over-gating direction as "already enacted," which the
`/propose` mismatch shows is only two-thirds true.

## Phase 5 — scope what loads where

- `evaluation.md` + `agency.md` (~1,500 words) currently load into
  every session on every project, including non-research repos where
  neither can apply (verified in the 2026-07-31 review session
  itself). Load them via research-project CLAUDE.md imports / scoped
  rules instead of globally.
- The candidates pipeline creates obligations that rot outside the
  cron-drained hub: 7 domain projects hold `raw/_candidates/` backlogs
  26–88 days stale; half the projects never used the pipeline at all.
  Make the backlog lifecycle opt-in (repos with `agency: max` or an
  explicit flag); elsewhere `/discover` writes a ranked triage note
  and creates no standing obligation, and `/lint` stops flagging
  staleness it structurally cannot cure.

## Simplicity assessment

Net-strongly-negative surface area: one skill, one hook, three tables,
and ~10k words of prose removed; added are two or three small scripts
that replace prose currently re-executed by the model at inference
time. No new knobs, no new rules files. The riskiest phase (3) replaces
soft checks with deterministic ones — the direction every accepted
proposal in this queue has moved.

## Risks & what could make this wrong

- **Telemetry under-counts silent value.** `/plan`'s existence might
  have shaped behavior without leaving rows. Unlikely — the agency
  verdict is what skills actually quote in logs — but Phase 1 is the
  cheapest to revert if withdrawal symptoms appear.
- **Prose deletion can remove context a weaker implementer model
  needs.** `budget.yaml` allows `haiku`/`sonnet` implementers; a
  contract-only SKILL.md assumes frontier-model inference of the
  "how." Mitigation: contracts and invariants stay; if a cheap
  implementer stumbles, the fix is a scoped hint in the subagent
  preamble, not global re-inflation.
- **Scripts ossify.** A prose check is trivially editable mid-session;
  a script invites "not my job" drift. Mitigation: keep scripts small,
  single-file, colocated in claude-system where `/elevate` proposals
  already target changes.
- **Phase 5's scoping could hide HCE from a project that needs it.**
  Mitigation: the project template imports it; graduation checklist
  step 3 already makes the holdout explicit.

## Recommendation

**Adopt in phases, in order (1→5), each as its own commit.** Phase 1 is
pure deletion with telemetry-backed evidence and one-command
reversibility. Phases 2 and 4 are text-only and low-risk. Phase 3 is
where reliability is actually gained and deserves the most care (smoke
tests per script). Phase 5 last, since it changes load behavior for
future sessions. Each phase leaves the system consistent if the next
never happens.
