---
kind: system-proposal
slug: token-metering-cumulative-double-count
added: "2026-07-26"
target: "claude/hooks/token_logger.sh (primary); coordinator/coordinator/policy.py (required companion)"
change_type: edit
adds_surface_area: false
evidence_citekeys: [khan2026token, li2025fm, hambardzumyan2026aira, kamelhar2026gsar, jia2026finharness]
evidence_strength: "code-released (khan2026token artifact: crate + catalogue.csv + IRR sheets) + 5-attestations across distinct groups; credibility ~3.5 (khan 3, li 4, hambardzumyan 4)"
status: proposed
recommendation: adopt
---

# Fix the token meter: the Stop hook records cumulative totals, so every ceiling reads a ~51× inflated number

## The change

`claude/hooks/token_logger.sh` aggregates the **entire session transcript**
on every Stop and inserts that cumulative total as a new `token_events`
row. Every consumer sums those rows. Summing a sequence of cumulative
snapshots double-counts everything before the last turn.

Measured on this box, session `0c4638b7…` (18 Stop events, values
monotonically increasing, `id` 2103 → 2152):

| turn | input | output | cache_creation |
|------|-------|--------|----------------|
| 1    | 45    | 45,835 | 123,643 |
| 2    | 118   | 74,772 | 197,413 |
| …    | …     | …      | … |
| 18   | 776   | 507,500| 1,269,355 |

Each row is the running total, not the turn's spend. Over the last 7 days:

```
SUM(input+output+cache_creation) over rows      = 4,775,410,939
SUM(per-session MAX) ≈ true spend               =    93,102,217
```

A **~51× over-count**.

### Primary edit — `claude/hooks/token_logger.sh`

Current (lines 64–78), inserting the cumulative transcript aggregate:

```python
import json, sys
from coordinator.writers import insert_token_event
payload = json.loads(sys.argv[4] or "{}")
insert_token_event(
    session_id=sys.argv[1] or "unknown",
    project=sys.argv[2] or None,
    input_tokens=payload.get("input_tokens", 0),
    ...
)
```

Proposed — insert the **delta since what is already recorded for this
session**, which needs no schema change because after the fix the stored
rows are deltas, so their sum *is* the previously-recorded cumulative:

```python
import json, sys
from coordinator.db import connect
from coordinator.writers import insert_token_event

FIELDS = ("input_tokens", "output_tokens", "cache_read_tokens", "cache_creation_tokens")
payload = json.loads(sys.argv[4] or "{}")
sid = sys.argv[1] or "unknown"

# Transcript aggregates are cumulative per session; store the per-turn delta.
with connect() as c:
    row = c.execute(
        "SELECT COALESCE(SUM(input_tokens),0), COALESCE(SUM(output_tokens),0),"
        "       COALESCE(SUM(cache_read_tokens),0), COALESCE(SUM(cache_creation_tokens),0)"
        "  FROM token_events WHERE session_id = ?", (sid,)).fetchone()
prev = dict(zip(FIELDS, (int(v or 0) for v in row)))
delta = {f: max(0, int(payload.get(f, 0)) - prev[f]) for f in FIELDS}

insert_token_event(session_id=sid, project=sys.argv[2] or None, **delta,
                   tools_used=payload.get("tools_used_summary", {}), timestamp=sys.argv[3])
```

The same delta should be written to the legacy NDJSON line (line 51–52) so
`_meta/token_log.ndjson` also becomes per-turn spend. `/iterate` already
documents that field as *"`tokens_spent` is the running sum from
`_meta/token_log.ndjson`"* — that description becomes true only after this
change.

### Required companion — `coordinator/coordinator/policy.py`

Correct metering alone does not restore `/plan`. `_load_budget()` maps a
project's `budget.yaml: max_tokens` onto `max_tokens_weekly`:

```python
"max_tokens_weekly": int(data.get("max_tokens", DEFAULTS["max_tokens_weekly"])),
```

But `budget.yaml` documents `max_tokens: 5000000` under *"Hard ceilings for
autonomous runs. Chain halts when any is hit"* — a **per-chain** ceiling,
which `/iterate` reads as exactly that. Observed real spend is ~93M/7d, so
even with a correct meter a 5M "weekly" limit defers everything. Fix by
reading a distinct key (`max_tokens_weekly`, absent → coordinator default
50M) rather than overloading `max_tokens`. Both halves are needed; neither
alone makes the gate function, which is why they are one proposal.

Historical rows stay inflated: 7-day windows remain wrong until the old
rows age out, unless the reviewer also collapses pre-fix rows to
per-session deltas. Recommend the backfill; it is a one-time SQL pass.

## Why (logical case)

The defect is inert in exactly one place and live in five:

- **Not affected** — the agency verdict (`coordinator/agency.py`) reads
  `ccusage` directly, never `token_events`. This is why the box has run
  `agency: max` bursts for weeks without anyone noticing the meter is
  broken: the signal that authorizes autonomous spend is sound, and the
  signals that are supposed to *stop* it are not.
- **`/plan` admission — dead.** Verified live today, read-only:

  ```
  $ claude-coordinator-plan agentic-research ingest --est-tokens 30000 --json
  {"admit": false, "reason": "defer: <25% weekly token budget remains
   (-4,770,410,939 of 5,000,000)"}   # exit 2
  ```

  Every `research` / `ingest` / `digest` job with `est_tokens` is deferred,
  permanently, with a negative-billions remainder. A gate that always says
  no is not a gate — it is either ignored or it silently blocks the
  `agency: max` pipeline that `/digest` and `/curate` depend on.
- **`pretooluse_cap.sh` — fires ~51× early.** It denies when
  `SUM(input+output+cache_creation) > cap`, i.e. at roughly 1/51 of the
  intended session cap, and its deny message quotes the inflated number.
- **`/iterate --chain max_tokens` — halts early on a bogus total**, and
  writes that total into `_meta/iteration_log.md`'s `budget{...}` suffix,
  which is what a human reads to decide whether a chain was expensive.
- **`/lint` check #10** ("costly sessions without insight", >500k
  threshold) flags sessions on a number ~51× too large.
- **`/headroom`** prints `tokens_5h` / `tokens_7d` from the same sums
  beside the correct `ccusage` figures.

`budget-as-ceiling`'s whole claim is that the halt is structural: the user
edits one file and the skills obey. That claim is currently false on this
box — not because a ceiling is missing, but because the number compared
against it is fiction.

## Why (reputable evidence)

**Gate 1 passes on both available routes.** `budget-as-ceiling` (status
`active`) carries seven sources from independently-developed groups —
`li2025fm` (Meituan FM Agent, cred 4), `hambardzumyan2026aira` (Meta FAIR
AIRA_2, cred 5 relevance / cred 4), `kamelhar2026gsar`, `xin2026eurekagent`,
`jia2026finharness`, `zhao2026agenticos`, `khan2026token` — well past the
three-independent-attestation bar, with credibility ≥3 on balance. Separately,
`khan2026token` alone satisfies the code-released route: the full artifact is
public and auditable (`github.com/sajjadanwar0/token-budgets` — crate source,
`catalogue.csv` with per-row quoted evidence, IRR coding sheets and
adjudications), which is what carries its credibility 3 despite being a solo
arXiv preprint.

What each actually demonstrates, specifically:

- **`khan2026token`** — 63 confirmed production budget-overrun incidents
  across 21 frameworks / 18 ecosystems, each backed by a quoted GitHub
  issue; four-class case labels re-coded blind by an independent second
  rater at κ = 0.837 (95% CI [0.745, 0.919]); a keyword-neutral baseline
  cohort (3,461 issues, 186 body-read) finds the same clusters recur in
  12/20 unrelated projects. Two findings land directly:
  - **M-cost-observability is the second-largest mechanism cluster** (22
    rows across 7 frameworks) — accounting layers that misreport spend are
    a documented, recurring production failure class in their own right,
    not a cosmetic bug. This proposal is that cluster, reproduced on this
    box, at 51×.
  - **No case in the catalog was prevented before at least one user paid.**
    Every deployed mitigation was post-hoc. The paper's argument is for
    *pre-flight* enforcement — refuse the call whose projected spend
    exceeds the cap.
  - Worth recording honestly: the paper's affine-typed Rust crate is
    scoped by its own Forgetful-Operator experiment to *non-bypassability
    under operator error in multi-agent delegation* — on single-agent
    workloads **a 4-line Python counter matches it at 0/30 overshoot**.
    The transferable asset for this box is the catalog and the
    pre-flight principle, not the type system.
- **`li2025fm`, `hambardzumyan2026aira`** (both cred 4) — the design case
  that explicit wall/cost ceilings are what make an unbounded search
  practically runnable; AIRA_2 reports 24h and 72h results separately
  *because* its ceilings predict behavior reliably. Ceilings that predict
  nothing (because the meter is wrong) fail this standard directly.
- **`jia2026finharness`, `kamelhar2026gsar`** — independent attestations
  of the same ceiling discipline in unrelated harnesses.

**On the pre-flight increment specifically** (the item NOTES.md flagged for
this run): the box already *has* the pre-flight shape — `/plan` estimates
before the job, `budget.yaml` declares the ceilings, the coordinator refuses
on `est_tokens > remaining`. The gap khan2026token exposes here is not a
missing mechanism; it is that the existing pre-flight gate is fed a fictional
number. Repairing the meter is strictly prior to, and strictly simpler than,
adding reservation machinery — and until it is repaired, any reservation
layer built on top would reserve against the same fiction.

## Simplicity assessment

**Removes surface area; adds none.** No new file, no new hook, no new
config knob, no new dependency. The primary edit is ~8 lines inside an
existing heredoc; the companion is one line in `_load_budget()` plus a
default. No schema migration — the delta form is self-referential
(`delta = transcript_total − SUM(recorded)`), which is why it needs no
"last cumulative" column.

Simpler forms considered and rejected:

- **Fix the readers instead** (`MAX` per session rather than `SUM`) —
  touches `readers.py` and every consumer's semantics, breaks time-window
  attribution (a session spanning two 5h blocks can no longer be split),
  and leaves the stored rows meaningless. The writer is the right locus:
  an event row should record an event, not a running total.
- **Add a `cumulative` column / a `session_totals` table** — schema
  surface area for something the delta form gets for free.
- **Do nothing and rely on `ccusage`** — that is the current de-facto
  state, and it is why `/plan`, the PreToolUse cap, `/iterate`'s
  `max_tokens`, and `/lint` #10 are all inert or wrong. It also concedes
  that four ceiling mechanisms are decoration.

This is the shape `/elevate` is supposed to prefer: it makes an existing
mechanism true rather than adding a new one.

## Risks & what could make this wrong

- **My cumulative reading could be an artifact of two long sessions.** The
  strongest counter-check: two sessions in `token_events` have 18 and 38
  rows with strictly increasing values, and single-turn sessions have one
  row. If Claude Code ever rotates or truncates `transcript_path`
  mid-session, the transcript aggregate would drop and `max(0, …)` would
  silently record 0 for that turn — under-counting instead of
  over-counting. That failure is safe-ish but should be checked before
  adoption; a `WARN` line when the delta clamps would catch it.
- **Session resumption / compaction.** If a resumed session gets a fresh
  `session_id` but the same transcript, the delta baseline is empty and
  the whole history is re-inserted under the new id. This is a *new*
  double-count path the current code also has; worth confirming against a
  `--resume` session before adoption.
- **Sub-agent sessions.** If `Agent`/subagent turns log under their own
  `session_id`, per-session deltas remain correct but a *session-scoped*
  cap (`pretooluse_cap.sh`) still cannot see delegated spend — khan's
  M-delegation-fanout cluster (11 incidents / 6 frameworks) in miniature.
  Not fixed here; flagged as the natural follow-up now that the meter is
  trustworthy. Do not bundle it into this change.
- **The `tools_used` histogram stays cumulative.** The delta of a
  histogram is messier and no consumer currently sums it; left as a known
  residual rather than over-engineered. Say so in a comment.
- **Backfill risk.** Collapsing historical rows is a destructive one-time
  SQL pass on `~/.claude/state.db`. If the reviewer skips it, 7-day
  windows self-heal in a week; if they run it, take a copy of the DB
  first.
- **The companion change is a judgment call about intent.** I read
  `budget.yaml: max_tokens` as per-chain because that is how the file
  documents itself and how `/iterate` consumes it. If the intended meaning
  really is weekly, then the correct fix is the opposite: raise the value
  and change `/iterate`. Either way the current overloading of one key for
  two different scopes is the defect.

## Recommendation

**Adopt.** This is the rare case where the evidence gate and the simplicity
gate point the same way: `khan2026token`'s second-largest incident cluster
is cost-observability failure, and the box is reproducing it at 51× with a
live, reproducible symptom (`/plan` deferring every ingest with a
negative-billions remainder). The fix deletes a wrong number rather than
adding a mechanism, needs no schema change, and restores four ceilings that
are currently decorative — including the PreToolUse hard stop, which is the
last line of defense against exactly the runaway-loop overruns the catalog
documents. Adopt the hook edit and the `policy.py` companion together;
verify the two risks above (transcript rotation, `--resume` session ids)
against a live session before landing; treat the backfill and the
sub-agent/delegation cap as separate, later decisions.
