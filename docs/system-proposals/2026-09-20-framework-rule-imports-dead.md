---
kind: system-proposal
slug: framework-rule-imports-dead
added: "2026-09-20"
target: "claude/templates/project/CLAUDE.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [kapner2026scanning, madatha2026deterministic]
evidence_strength: "code-released (harness-eval, Red Hat + Ben-Gurion), credibility 4; plus a direct 6-variant measurement on this box against Claude Code 2.1.275"
status: proposed
recommendation: adopt
---

# Framework-rule `@imports` have never loaded anything

## The change

`claude/templates/project/CLAUDE.md`, lines 29–33, currently read:

```markdown
Framework rules load here (per-project, not globally — they only cost
context where they can apply):

@~/claude-system/claude/rules/evaluation.md
@~/claude-system/claude/rules/agency.md
```

Proposed:

```markdown
Framework rules live at `~/claude-system/claude/rules/`. They cannot be
`@`-imported — CLAUDE.md imports resolve only inside the project tree —
so read them at the point they apply: `evaluation.md` before designing
or judging an evaluation, `agency.md` before an autonomous run.
```

Two companion edits follow from the same fact and should land with it:

1. **The 19 existing project `CLAUDE.md` files** carry the same two dead
   lines (`agentic-research` carries one; the other 18 carry both). They
   need the same replacement. One `sed` block per repo; no other content
   moves.
2. **`install.sh` lines 50–53** assert the mechanism that does not exist:

   ```
   # rules/ is deliberately NOT linked: evaluation.md/agency.md load via
   # project-CLAUDE.md @imports so they cost context only where they apply
   # (instruction-ablation-program, phase 5). Skills reference them by
   # their stable ~/claude-system/claude/rules/ paths.
   ```

   The first sentence is false and should say what the second already
   implies: rules are reached by path, read on demand.

**Optional, and the reliable half** — the eight skills that declare
`respects: - ~/claude-system/claude/rules/<file>.md` in frontmatter
(`digest`, `discover`, `curate`, `elevate`, `fetch-paper`, `ingest`,
`lint`, `promote-moc`) do nothing with that field; Claude Code does not
read it. A one-line first step in each ("Read the files under
`respects:` before acting") turns a decorative field into a live one.
That is eight one-line edits and no new file. Recommended as
adopt-with-changes if the reviewer wants loading actually restored
rather than merely un-lied-about; see *Recommendation*.

## Why (logical case)

The instruction-ablation program's phase 5 (`claude-system@683b2a6`,
2026-08-01) moved `evaluation.md` and `agency.md` out of global scope on
the reasoning that they should cost context only in projects where they
apply. The mechanism chosen was a project-`CLAUDE.md` `@import` with a
tilde path. `install.sh` stopped linking `rules/` into `~/.claude/` on
that basis.

The mechanism does not work. Measured on this box today against Claude
Code 2.1.275, with a scratch project and marker files:

| Import line | Loaded? |
|---|---|
| `@rel.md` (relative, inside project) | **yes** |
| `@/tmp/imptest/abs.md` (absolute, inside project) | **yes** |
| `@~/imptest-tilde.md` | no |
| `@import ~/imptest-tilde.md` | no |
| `@/home/eschmitt/imptest-tilde.md` (absolute, outside project) | no |
| `@.claude/rules/linked.md` (in-project symlink → outside) | no |
| `@/home/eschmitt/.claude/imptest-claudedir.md` | no |
| absolute-outside, re-run with `--add-dir /home/eschmitt` | no |

Two independent facts fall out. `~` is not expanded in an `@` import,
and — more consequentially — **no `@` import resolves outside the
project tree**, whatever the form. Symlinks are resolved before the
check, and widening the permission set with `--add-dir` does not help.
So there is no spelling of these two lines that would work.

The consequence: since 2026-08-01, `rules/evaluation.md` (the HCE
discipline) and `rules/agency.md` (the autonomous-spend levels) have
been in **no** project session's context. The user-level `CLAUDE.md`
says these "load via each research project's `CLAUDE.md` `@import`s".
They do not. This session is itself an instance — `agency: max` governs
this run, `CLAUDE.md` line 79 imports `agency.md`, and the rule's text
is absent from context; the skill body happened to restate enough of it
to proceed.

Nothing failed loudly. That is the whole problem.

## Why (reputable evidence)

**`kapner2026scanning`** — *Scanning the Harness: An Empirical Study of
Supply-Chain Defects in AI Coding-Agent Configurations* (Red Hat +
Ben-Gurion University), credibility 4, relevance 4. Gate 1 passes on the
**code-released** disjunct: the instrument (`harness-eval`), corpus
manifest with pinned commits, adjudication prompt, per-finding verdicts
and regeneration scripts are all public, and every finding is re-derived
exhaustively by a second implementation. It is the strongest
reproducibility package in this cluster.

What it actually demonstrates, on 3,171 public repositories:

- **The broken-`@import` class is real and roughly half-genuine.** Its
  observation-tier rule "broken `@import` in context file" fired with
  11/23 (48%) of pairs confirming a real defect — the usual benign cause
  being a file created at runtime, which is not the case here.
- **The `cannot-work` consequence family exists because these failures
  are silent.** Its sibling finding — a subagent with no description is
  skipped, with the reason written only to a debug log — is the same
  shape: the configuration is accepted, the component never runs, and
  the author is never told.
- **"A spec rule the reference client does not enforce is one authors
  ignore."** The inverse holds here: a syntax the reference client
  silently narrows is one authors over-trust.
- **The harness is a dependency layer without dependency hygiene** — no
  lockfile, no install-time check. An import that resolves to nothing is
  exactly the defect that hygiene would catch.

Its limits are stated in the note and do not bite here: no human has
scored any verdict, κ=0.23 on the adjudicated subset, recall unmeasured.
None of that matters for this proposal, because the paper supplies the
*class* and the prior, while the decisive evidence is the local
measurement above — a controlled experiment on the exact client version
this box runs, which is stronger than any published rate.

**`madatha2026deterministic`** (credibility 2, Zenodo artifact) is a
second, independent attestation that the agent-config layer is an
unmanaged supply chain — 6,145 config files, 10.1% cross-repo
duplicates, under 1% declaring permission boundaries. It does not carry
credibility weight; it is cited because kapner explicitly excludes the
instruction-file-only repositories that are madatha's focus, and
project `CLAUDE.md` files are that layer.

The literature is not doing the heavy lifting here and should not
pretend to. It supplied the hypothesis — kapner's note flagged a
possible broken import in this repo's `CLAUDE.md` on 2026-09-14, and
`NOTES.md` carried it forward as an open item. The measurement settles
it, and corrects the note's guess in the process: the note blamed the
`@import` token, but the tilde and the project-tree confinement are the
operative causes, and the confinement is unfixable by re-spelling.

## Simplicity assessment

**Gate 2 passes on the strongest available reading: this removes a false
affordance.** Two lines that do nothing are deleted. The replacement is
three lines of prose that name a path — the same path the eight
`respects:` blocks and seven skill bodies already name. No new file, no
new skill, no new hook, no new config knob, no new script.

Simpler forms considered and rejected:

- **Re-spell the path** (`$HOME`, absolute, a symlink into the project).
  Measured: none work. The confinement is to the project tree.
- **Re-link `rules/` into `~/.claude/` and load globally**, reverting
  phase 5. This works, and is the one alternative that restores loading
  with zero per-project changes — but it pays ~12 KB of context in every
  session of every project, which is precisely the cost phase 5 removed,
  and the user's own `CLAUDE.md` states the not-globally requirement.
- **Copy the two rule files into each project's `.claude/rules/`.** This
  works and keeps loading path-scoped, but vendors 19 copies of a
  framework file and creates a drift surface with no owner. Rejected on
  Gate 2.
- **Do nothing and leave the lines.** Rejected: a dead line that reads
  as live is worse than no line, and this one has already misled one
  design decision (`install.sh`'s unlinking of `rules/`).

The optional skill-side `Read` step adds eight lines across eight
existing files and no new surface. It is the only option that both
restores loading *and* keeps the cost scoped to invocation — phase 5's
stated goal, reached by the mechanism Claude Code actually provides.

## Risks & what could make this wrong

- **The measurement could be version-specific.** It was taken against
  2.1.275 on 2026-09-20. Claude Code's import resolution has changed
  before — kapner records `bypassPermissions` changing meaning mid-study
  — and a future release could widen imports to arbitrary paths, which
  would make the two deleted lines correct again. Mitigation: the
  replacement prose is harmless if that happens, and the test above is
  eight lines to re-run. Re-run it before assuming this is permanent.
- **The deleted lines might be load-bearing somewhere I did not check.**
  `/sync-imports` scans for `@import` directives in project `CLAUDE.md`
  and `.claude/rules/**`, but it matches only the
  `agentic-research/concepts/` form, not these. Nothing else in
  `claude-system` greps for them. If some untracked cron wrapper in
  `~/.claude/schedule/` does, this would break it — those wrappers are
  deliberately outside the repo and I could not read them.
- **Prose is a weaker instrument than an import.** An import, if it
  worked, would be unconditional; "read `agency.md` before an autonomous
  run" is an instruction the agent may skip. `lavrenko2026instruction`
  (from the 09-06 thread) found instruction placement changes trajectory
  legibility while leaving final accuracy flat, which cuts against
  expecting much from prose alone. This is the argument for the optional
  skill-side `Read` step, and the reason I would not call the prose
  replacement a full fix.
- **The migration touches 19 repositories.** Mechanical and
  `git revert`-able per repo, but it is 19 commits, and two of those
  repos (`mle-bench`, `agentic-research`) also carry the separate
  concept-import defect — see
  [concept-import-contract-inert](2026-09-20-concept-import-contract-inert.md).
  Do them together per repo, not in two passes.
- **This proposal does not weaken the HCE holdout.** It is the reason
  the holdout rule has not been in context for seven weeks; adopting it
  is the only path by which `rules/evaluation.md` reaches a session at
  all. If the reviewer adopts nothing else this cycle, adopt the part
  that restores `evaluation.md`.

## Recommendation

**Adopt** — and adopt the optional skill-side `Read` step with it, which
makes this adopt-with-changes if the reviewer treats the template edit
alone as the proposal. The template edit on its own is still worth
making: it stops the system asserting a mechanism that does not exist,
and it corrects the false premise recorded in `install.sh` that caused
`rules/` to be unlinked in the first place. But the defect being fixed
is "two framework rules have not loaded since 2026-08-01," and prose in
a template does not fix that on its own — a `Read` at the top of the
eight skills that already declare `respects:` does. Both halves together
are eleven edited lines across ten existing files plus a mechanical
19-repo sweep, add no new surface, and restore a property the system has
believed it had for seven weeks. That is the cheapest true fix
available, and the evidence for the defect is a direct measurement, not
an inference.
