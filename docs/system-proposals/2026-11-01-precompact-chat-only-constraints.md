---
kind: system-proposal
slug: precompact-chat-only-constraints
added: "2026-11-01"
target: "claude/hooks/pre-compact.sh"
change_type: edit
adds_surface_area: false
evidence_citekeys: [salle2026benchmarks, chen2026governance, lencevicius2026when, nakayashiki2026when]
evidence_strength: "code-released (salle2026benchmarks, lencevicius2026when, nakayashiki2026when); 4 independent groups; credibility ~3 (3/3/3/3)"
status: proposed            # proposed | accepted | rejected | superseded
recommendation: adopt
---

# PreCompact should name the one class compaction loses: standing instructions given only in chat

## The change

One bullet in the reminder heredoc of `~/claude-system/claude/hooks/pre-compact.sh`
is replaced. No new file, field, or mechanism.

Current third bullet:

```
- Auto-memory is a hint, not a home — promote anything the next session
  must see into a tracked file.
```

Proposed:

```
- Standing instructions the user gave only in chat (how to commit, what
  not to touch, what not to run) are what compaction drops most. Quote
  the latest version of each verbatim in the summary. Anything the next
  session must see goes in a tracked file, not auto-memory.
```

The auto-memory clause is kept but shortened. `CLAUDE.md` (lines 19 and
159) already states it, and `CLAUDE.md` is in context after compaction.

## Why (logical case)

The hook already covers the two things compaction can lose that the
agent *produced*: findings (→ `/wrap`) and results (→ `metrics.json` /
`results/`). Since 10-11 it also gives the transcript address, so a
dropped detail can be recovered. That recovery only works when the agent
knows something is missing. A dropped "don't touch X" is a rule the agent
no longer knows it was given, so it never goes looking for it.

On this box, the rules in `CLAUDE.md`, `.claude/rules/` and a loaded
SKILL.md are re-injected or re-readable after compaction. The text that
exists only in the conversation is the user's own process and scope
instructions, given in chat during an interactive session. That is the
class the measurements below find dropped most often, and it is the one
class the current reminder does not mention.

The hook text reaches the summarizer. Two compaction summaries on this
box, in the `-mnt-projects` sessions `12c3e956…` and `145fda70…`, cite it
("A NOTES.md / wrap update is pending per PreCompact"; "Given the
PreCompact instruction, the first action should be…"). So the edit acts
on the step that does the dropping.

## Why (reputable evidence)

- **salle2026benchmarks** (cred 3, code `aisa-group/compaction`;
  Andriushchenko/Bethge/Abdelnabi, Tübingen). This is the direct anchor,
  measured on **Claude Code's own compaction**. In 44 real sessions with
  269 extracted user constraints, 19% are dropped at the next compaction
  (24.5% under a second-family judge). The rate depends on the kind of
  constraint:
  - feature requirements: 6.2% (n = 96)
  - process instructions: 35.9% (n = 39)
  - scope exclusions: 36.8% (n = 38)

  On ProgramBench, Opus drops 0.51 of 3 first-turn rules per summary even
  though the summary prompt asks for "constraints … that should persist".
  So a *generic* ask is not enough. The paper's own reading is that almost
  everything compaction drops can be re-read, except a constraint that
  exists nowhere but the context.
- **chen2026governance** (cred 3). Gives the behavioural consequence that
  salle does not measure. Violation goes from 0% with the policy visible
  to 30% pooled (up to 59%) after one compaction, and survival predicts
  violation almost binarily (survived → 0%, dropped → 38%). Re-presenting
  the constraint verbatim (~47 tokens) restores 0%.
- **lencevicius2026when** (cred 3, code `raimondasl/persistgym`,
  pre-registered). A controlled test of re-observability. With a
  truncated window, a fact the agent can look up again costs nothing
  (100% success). One it cannot look up drops success to 7.5% (15/200).
  A chat-only instruction is the not-re-observable case.
- **nakayashiki2026when** (cred 3, Zenodo artifact). Supports the "agent
  won't go looking" step. Agents with a reachable provenance link checked
  it in about one episode in five and made stale-consistent decisions
  ~75% of the time. A transcript address is therefore not a substitute
  for keeping the rule in the summary.

Gate 1 passes on all three counts: code is released for three of the
four sources, there are four independent groups, and credibility averages
3. This is the unlock recorded at the 2026-08-09 and 2026-08-16 holds,
which left constraint-pinning at "2 rated attestations (chen2026governance
cred 3, semenov2026beyond cred 2)". semenov2026beyond is not counted here.
The idea was held, never rejected, so this is a first proposal, not a
re-raise.

## Simplicity assessment

`adds_surface_area: false`. One bullet is replaced by a bullet two lines
longer, inside an existing heredoc. No new hook, knob or file. The
redundant half of the old bullet is cut.

Heavier forms considered and set aside:
- **Harness pinning** (chen2026governance's quarantine + re-inject +
  integrity check): the hook would have to parse `transcript_path` and
  classify user turns as standing constraints. That is new code with a
  classification step it cannot do deterministically.
- **Write chat-only constraints into a project file** (salle's own
  suggestion, our reading): this persists session-scoped instructions
  into tracked project state, which the user may not want, and it needs a
  convention for where they go.

Quoting them in the summary is the smallest form. It acts at the one
point where the loss happens.

## Risks & what could make this wrong

- **The fix form is untested.** salle shows a generic ask failing. This
  is a category-specific ask with examples, which no source measures.
  lavrenko2026instruction (cred 2) finds repetition raises procedural
  adherence only 90 → 93% and leaves accuracy unchanged. The benefit may
  be small.
- **Presence is not adherence.** In chandran2026autoresearch
  (peer-reviewed), a rule stayed in the program the agent re-read every
  iteration and was still overridden six times. This edit keeps the rule
  in the summary. It does not make the agent follow it.
- **A pinned instruction can be stale** (hu2026memory). A chat rule the
  user later revoked gets quoted with privileged position. "The latest
  version of each" is there for this, but the summarizer has to resolve
  the revocation correctly.
- **Summary bloat and over-capture.** One-off requests may be quoted as
  standing rules. The examples bound the category, but there is no hard
  limit.
- **Delivery is shown, not fully characterized.** The two summaries
  citing PreCompact show that the text reaches the summarizer. Whether
  that holds for auto-compaction as well as manual `/compact`, and for
  subagent compaction, is not established here.
- **salle's per-category ns are small** (39 and 38) and LLM-labelled. The
  process/scope vs feature gap is well outside the intervals, but the
  exact rates are not firm.

## Recommendation

**Adopt.** The failure is measured on this harness's own compaction at
about a one-in-three rate for exactly the instruction class the current
reminder omits, and chen2026governance ties dropped constraints to
violations. The edit swaps one bullet and costs nothing to maintain. The
upside is uncertain because the ask itself is untested. A cheap local
check uses salle's released code: plant three process/scope rules in the
first turn of a long Claude Code session, then count restatement per
compaction with and without the new bullet. It needs agent tokens only,
no GPU. Run it if the edit's effect is ever in doubt.
