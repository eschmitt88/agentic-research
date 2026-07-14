---
kind: system-proposal
slug: experiment-loop-skill-scoping
added: "2026-07-14"
target: "claude/skills/ (layout), skills/new-project, skills/propose, skills/iterate"
change_type: restructure
adds_surface_area: false
evidence_citekeys: [belikova2026managing, lodha2026less]
evidence_strength: "user-directed simplification grounded in the skill-library-lifecycle concept (merge/consolidate bias) and context-rot evidence; not literature-gated the way /elevate proposals are"
status: proposed
recommendation: adopt
---

# Scope the experiment-loop skills to experiment projects; consolidate /expand and /ensemble

Origin: user-directed (2026-07-14 session), not an `/elevate` cycle. The
user's framing: lit/web-research projects that only build a knowledge
base carry seven experiment-loop skills in every session's context —
superfluous skill descriptions are context rot — and some of those
skills are over-specific where "concise goal + a smart model" would do.

## The change

Three parts. Net surface area is negative: no new skill, two skills
removed, one directory boundary added.

### 1. Split the skill tree by loop shape, not domain

Two groups, drawn by what kind of loop a project runs — **not** by
ML-vs-science domain. ML-runnable experiments and external/wet-lab
science share the same propose → execute → diagnose → iterate
discipline; a capable model adapts the execution step (run it locally
vs. specify the external protocol). Splitting those would be the
over-specificity the user warned against.

- **Knowledge-base group** (stays user-global at `claude/skills/`):
  `discover`, `fetch-paper`, `ingest`, `curate`, `digest`, `lint`,
  `promote-moc`, `sync-imports`, `wrap`, `headroom`, `plan`,
  `new-project`, `elevate`. Every research project uses these.
- **Experiment-loop group** (moves to `claude/skills-experiment/`, not
  linked into `~/.claude/skills/`): `propose`, `implement`, `iterate`,
  `expand`, `ensemble`, `new-experiment`, `derive-experiment`.

Borderline call: `derive-experiment` reads literature but *writes* a
proposal — it belongs with the loop it feeds. `plan` stays global
because the coordinator admission gate is not experiment-specific.

### 2. Opt-in loading via the project template

`~/.claude/skills` is already a directory symlink into the repo
(install.sh), and Claude Code loads project-level skills from
`<project>/.claude/skills/`. So the mechanism is one symlink:

```sh
# in an experiment project
ln -s ~/claude-system/claude/skills-experiment .claude/skills
```

`/new-project` gains a one-line behavior: if the new project declares
experiments (user says so, or the template's `experiments/` dir is
kept), create the symlink; otherwise skip it. Existing experiment
projects get the symlink by hand once. A lit-only repo like
agentic-research then stops carrying `/implement`, `/iterate`, etc. in
context at all — and its CLAUDE.md line "No /implement or /iterate in
this project" (a rule that exists only to counteract global loading)
can be deleted.

### 3. Consolidate the two over-specific skills

- **`/expand` → `/propose --expand <proposal-path> [--n N]`.** Same
  behavior (N alternative implementations sharing a hypothesis), stated
  in a paragraph of `/propose`'s SKILL.md instead of a standalone file.
- **`/ensemble` → `/iterate --ensemble <slug...>`.** An ensemble is one
  more propose→implement cycle whose proposal is "combine these
  members"; `/iterate` already owns the cycle machinery. Strategy
  choice (voting/stacking/averaging) is exactly the kind of finer
  detail the model can decide — the current skill's `--strategy auto`
  default already admits this.

Both SKILL.md files are deleted; their essential contracts (output
locations, frontmatter fields like `parent:` and `members:`) move as
short sections into the absorbing skills. HCE `respects:` declarations
carry over unchanged — `evaluation.md`'s clause list already names
expand/ensemble behaviors generically ("proposing, implementing,
iterating, expanding, or ensembling").

## Why

- **Context rot is real and this is the cheap end of it.** Seven skill
  descriptions loaded into every lit-only session buy nothing and cost
  attention; [[context-eviction-policy]] (lodha2026less) attests that
  irrelevant context degrades agent behavior, and the repo-level
  workaround ("No /implement here") is evidence the current layout
  fights itself.
- **The library's own lifecycle concept prescribes this.**
  [[skill-library-lifecycle]] (belikova2026managing) carries a
  merge/consolidate bias for skill libraries; the deferred full
  merge/split detector was conditioned on graph size, but these two
  merges need no detector — they are visible by inspection.
- **Matches the user's standing authoring principle** (recorded
  2026-07-14): concise, goal-level skills; let a smart model determine
  finer details; scope specialized groups to the projects that need
  them.

## Simplicity assessment

Removes: two SKILL.md files, one CLAUDE.md counter-rule, seven
irrelevant descriptions from every lit-only session. Adds: one
directory boundary and one conditional symlink in `/new-project`. The
main cost is a **naming/muscle-memory break** (`/expand` and
`/ensemble` stop existing as top-level commands) and a one-time
migration touch on existing experiment projects.

## Risks & what could make this wrong

- **Flag-modes can bloat the absorbing skill.** If `/propose` grows
  many modes, we've traded skill-count rot for skill-length rot. Guard:
  the absorbed sections must stay goal-level (a paragraph each), per
  the authoring principle.
- **A project's shape can change.** A lit repo that later grows
  experiments needs someone to remember the symlink. Mitigation:
  `/new-experiment` (experiment group) is unreachable in such a repo,
  but `/lint` is global — add a one-line check: `experiments/` exists
  but no experiment-loop skills are linked → warn.
- **Skill-name references in existing docs** (iteration logs, NOTES,
  concept notes) will mention `/expand`//`/ensemble`; they remain
  readable as history. No rewrite needed.

## Recommendation

**Adopt.** Apply in one claude-system commit (move + two merges +
`/new-project` edit), then: symlink existing experiment projects,
delete the agentic-research CLAUDE.md counter-rule, and record the
layout change in claude-system's README.
