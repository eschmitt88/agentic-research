---
kind: system-proposal
slug: system-review-intent-briefing
added: "2026-11-14"
target: "claude/hooks, claude/skills*, claude/rules, claude/CLAUDE.md, claude/templates/project, coordinator, scripts, docs/index.html"
change_type: edit + removal + consolidation
adds_surface_area: false
evidence_citekeys: []
evidence_strength: "box-local measurement for the code fixes (transcripts, a two-call rule-loading test, six months of hook output); the intent-briefing writing guide for the prose changes (P2, P3, P5 rated strong there)"
status: accepted
recommendation: adopt
provenance: user-directed review (2026-11-14), not /elevate-gated. Recorded here so the reasoning and the decision live in the same queue, as with the 2026-07-31 program.
---

# System review after the intent-briefing founding

The user asked for an intense review of claude-system, with authority to
update it. This records what changed and why, and what was looked at and
left alone. Applied as claude-system@1c23fbd..0d4b305.

## Defects found by measurement

1. **The Stop hook over-counted ~2.4x and missed subagents.** It summed
   usage over every transcript record. One response is written as several
   records that repeat its usage (11,934 records for 6,490 responses in the
   largest local transcript, identical usage on every duplicate), and a
   delegating session's subagent transcripts were never read. `/iterate`'s
   `max_tokens` ceiling and the dashboard read those sinks. The 2026-07-26
   fix made the rows per-turn deltas but kept the per-record sum. Counting
   now lives in `coordinator/token_log.py` on the deduped reader the quota
   anchor already used. Cost on the 327 MB transcript fell from 1.7 s and
   617 MB (`jq -s`) to 0.4 s and 20 MB.
2. **Scoped rules were never scoped.** Claude Code reads only `paths:` from
   a rule's frontmatter. The template used `applies_to:`, so every rule
   loaded in every session of every project, and `CLAUDE.md` imported each a
   second time. Confirmed with two headless calls on this box: an
   `applies_to:` rule loads at launch, a `paths:` rule loads only after a
   matching file is read. Template fixed; 23 project repos migrated.
3. **`status_emit.sh` was dead weight.** It ran on every `Write` and
   produced two records in six months. The lint check "costly sessions
   without insight" used those records as its only insight signal, so it
   flagged every large session. Both removed.
4. **The public system map described machinery removed in August** (`/plan`,
   the admission policy, the PreToolUse cap, `/ensemble`, `/expand`) and
   lacked `/curate`, `/promote-moc`, `/elevate`, `/jobs`, the job ledger and
   the agency verdict. Rewritten, and `smoke.sh` now fails when the map and
   the repo disagree on skills, hooks or CLIs.
5. **The scaffold left `budget.yaml` hardware hints as placeholders**
   (`gpu_count: 0`), which `/implement` hands to its subagent. The scaffold
   now reads them from the machine. `--agency-max` added.
6. **`auto-push.err` gained a header on every attempt**, successful or not.
   It now records failures only.
7. **Synced third-party skills landed untracked inside this public repo**
   (`claude/skills/synced/`, via the `~/.claude/skills` link). Gitignored.

Also found, outside the tracked repo: the nightly curate sweep aborted
silently on a project with no concepts yet (`ls glob | wc -l` under
`set -euo pipefail`). Fixed in the untracked wrappers, which now log an
abort line.

## Prose changes, by writing-guide principle

- **P2, state what the reader won't do by default.**
  - `CLAUDE.md` gains a short section on writing instructions for another
    model, pointing at the guide.
  - `/discover` marks each entry `verified: page | snippet`, and `/curate`
    re-checks snippet entries. The first sweeps reported that most entries
    rested on search excerpts and several arXiv IDs were unopened.
  - `/new-project` now says to fill the orientation, because `/ingest`
    scores relevance against it.
- **P3, rules that must hold go in code.** The map drift guard, the token
  logger test and the scaffold tests are code. `agency.md` now says plainly
  that nothing meters an ingest burst against `max_tokens`; the verdict is
  the limit there.
- **P6, don't restate; restated text drifts.** Contradictions removed where
  two statements of one contract had diverged:
  - `/ingest` and `/wrap`: "shows diff, waits" vs "no confirmation gate".
  - `/digest`: "never ingests" vs step 8.
  - `/implement`: "the only skill allowed to spawn a subagent" vs
    `/discover` and `CLAUDE.md`.
  - `/iterate`: a hard-coded pause vs the Confirmation principle.
  - `/discover`: a duplicated step, and a 14-day staleness rule against "no
    standing obligation".
  - `CLAUDE.md`: the "What not to do" list restated four rules above it.
- **Stage by path.** Seven skills ran `git add -A`. Scheduled jobs and
  other sessions keep these repos dirty, so a sweep commits their state.
  Skills now stage what they wrote and push. `/sync-imports` commits its
  hub changes, which nothing committed before.
- **P7, weaker readers get more structure.** `/implement` adds explicit
  steps to the preamble when the implementer is a smaller model than the
  ideator. This is the mitigation the July program named and deferred.

## Pending proposals decided

All three accepted and applied: `iterate-redirect-before-halt`,
`hce-known-bad-preflight`, `precompact-chat-only-constraints`.

## Looked at and left alone

- **The `respects:` read-first mechanism.** The docs say external `@`
  imports work after a one-time interactive approval, which unattended runs
  never get, so the 09-20 measurement stands and read-on-demand stays. The
  boilerplate sentence that gave the wrong reason was shortened.
- **A PreToolUse hook to block `test/` reads** (P3 applied to HCE). It
  cannot tell a search-phase read from the final-scoring pass without
  state, and it would run on every tool call. Held, as the June and July
  cycles held gate hardening.
- **Skipping the journal commit for unattended sessions.** The only signal
  is an undocumented environment variable. The neutral-cwd workaround stays.
- **`/implement` pasting `CLAUDE.md` into the subagent prompt.** Possibly
  redundant, but whether subagents receive `CLAUDE.md` was not verified.
- **Reading `evaluation.md` in literature-only repos.** About 1.3k tokens
  per skill run; not worth a conditional.

## Risks

- **In-flight sessions:** a session whose earlier turns were recorded under
  the old logger can show a zero delta until its true total passes the
  inflated recorded sum. The direction is under-count.
- **Path-scoped rules** now load only when a matching file is read, written
  or edited. A session that writes into `raw/` through a shell command
  alone will not load `data.md`; the same rule is in `CLAUDE.md`.
- **The guide is untested** on held-out tasks. The prose changes rest on its
  three strong principles plus removal of plain contradictions. The private
  brief audit is the planned check.
