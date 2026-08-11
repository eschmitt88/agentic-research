# 0001 — Nightly `/curate` + `/promote-moc` sweep

- **Date:** 2026-06-23
- **Status:** accepted

## Context

`/digest` (weekly cron) auto-advances only ~3 top candidates per run
under a "slow/normal" headroom cap and **defers the rest, leaving the
candidate file in place**. Nothing ran a follow-up `/curate` to drain those
deferrals, so they accumulated: between 2026-06-03 and 2026-06-23, three
digest files leaked 7 deferred-but-on-mission items, which read as
"uncurated" on the dashboard / Pages site indefinitely. `agency: max` was
working per-digest; the gap was the absence of a periodic drain.

## Decision

Add a **nightly** sweep that runs `/curate` (when the backlog is
non-empty) followed by `/promote-moc` (auto-detect). Mechanism mirrors the
existing digest cron:

- Script: a schedule script under `~/.claude/` (OS-drive config,
  not tracked by this repo — same location as
  `agentic-research-digest.sh`).
- Crontab: `0 4 * * * .../agentic-research-curate.sh`.
- Logs to `_meta/curate.log`.
- Guards: skips the `/curate` invocation entirely when
  `raw/_candidates/*.md` is empty; `/promote-moc` still runs so a cluster
  that ripened from a prior ingest gets promoted even on a no-candidate
  night. Both skills are no-ops when there is nothing to do.

Timing: the nightly sweep is offset from the weekly digest, so digest-day
fresh candidates are drained on Tuesday's sweep — there is no urgency, and
the point is to prevent accumulation, not to drain within the hour.

## Consequences

- Deferred digest items can no longer pile up for weeks; the uncurated count
  trends to zero on its own.
- Costs one (usually cheap / skipped) `claude -p` invocation per night under
  headless permission configuration, gated by the same coordinator
  headroom verdict every skill respects (a `hold` verdict stops new
  autonomous work mid-sweep).
- Alternative considered: raise `/digest`'s auto-advance cap under a GO/high
  verdict so it defers less up front. Rejected as the *primary* fix because
  it couples backlog-drain to digest cadence (weekly) and to whatever the
  verdict happens to be at digest time; a dedicated nightly drain is
  cadence-independent. The cap increase remains a reasonable complementary
  tweak.
