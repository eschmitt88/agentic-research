---
kind: system-proposal
slug: concept-import-contract-inert
added: "2026-09-20"
target: "claude/skills/sync-imports/SKILL.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [kapner2026scanning]
evidence_strength: "code-released (harness-eval, Red Hat + Ben-Gurion), credibility 4; plus a direct 6-variant measurement on this box against Claude Code 2.1.275"
status: proposed
recommendation: adopt-with-changes
---

# The concept-import contract loads nothing; `used_by:` records intent, not reach

## The change

`claude/skills/sync-imports/SKILL.md`, step 3, currently teaches the
canonical downstream form:

```markdown
   Match lines of the form:
   ```
   @import ~/projects/research/agentic-research/concepts/<name>.md
   ```
   Expand `~` to the user's home. Deduplicate absolute paths.
```

Neither half of that line is a working Claude Code import: `@import` is
not the directive (`@<path>` is), `~` is not expanded, and — decisively
— an import pointing at `agentic-research/concepts/` from any *other*
project is by construction outside that project's tree, which no import
form reaches. So the mechanism cannot be repaired by re-spelling.

Proposed — keep the detection rule (it is the declared-dependency
record, and that is worth keeping), and stop calling it context loading:

```markdown
   Match lines of the form:
   ```
   @import ~/projects/research/agentic-research/concepts/<name>.md
   ```
   Expand `~` to the user's home. Deduplicate absolute paths.

   These lines are a **declaration, not a load**. Claude Code resolves
   `@` imports only inside the project tree, so a concept in
   `agentic-research/` never enters a downstream session's context.
   `used_by:` records which projects claim a concept, which is what the
   meta project needs in order to see what is load-bearing. It does not
   mean the concept text was read.
```

Two companion edits, same fact:

1. **`README.md` line 159**: "`concepts/` files are `@import`ed by
   downstream projects" — same correction.
2. **`agentic-research/CLAUDE.md`** (this repo, not `claude-system`)
   carries the Import contract block that mle-bench copied. It should
   say the same thing. That edit is mine to make once the reviewer rules
   on the semantics; it is not proposed here because the skill is
   forbidden from pre-empting the decision.

**What this proposal deliberately does not decide**: whether the four
dead `@import` lines in `mle-bench/CLAUDE.md` should be deleted
outright. See *Recommendation*.

## Why (logical case)

The import contract is the second of two things this repo's `CLAUDE.md`
claims about itself: concept notes are "a read interface, not only
internal artifacts," and "evolution here propagates to downstream
projects at their next session start — no copy-paste."

The propagation claim is false. Measured on this box today against
Claude Code 2.1.275 (full table in
[framework-rule-imports-dead](2026-09-20-framework-rule-imports-dead.md)):
an `@` import resolves only inside the project tree. Relative and
absolute in-tree paths load; `~/...`, `@import ~/...`, absolute
out-of-tree, an in-tree symlink pointing out, and out-of-tree even with
`--add-dir` all load nothing. A downstream project importing a concept
from `agentic-research/` is out-of-tree in every case.

The live scope is small and worth stating exactly. Four concepts carry
`used_by:` entries: `hce-evaluation` (`mle-bench`, `_scratch`),
`citation-anchoring`, `pass-at-k` and `budget-as-ceiling` (all
`mle-bench`), every one dated 2026-04-24. `mle-bench/CLAUDE.md` lines
23–26 hold the four `@import` lines those entries were derived from.
So the affected surface is four concept notes and one downstream repo —
not a systemic breakage, but a mechanism the project advertises in its
root `CLAUDE.md`, its README and a skill, and has never delivered.

What is actually at stake is the meaning of `used_by:`.
`concepts/_README.md` says concepts "earn their keep through `used_by:`"
and that long-term empty `used_by:` is a signal a concept is not
load-bearing. If `used_by:` records a declaration nobody reads, that
keep is unearned, and the `retired` status rule ("never earned
`used_by:`") is grading on a broken instrument. Fixing the semantics is
cheap; discovering later that four concepts were promoted on a false
signal is not.

## Why (reputable evidence)

**`kapner2026scanning`** — credibility 4, code released
(`harness-eval`, corpus manifest at pinned commits, per-finding verdicts
and regeneration scripts), Red Hat + Ben-Gurion University, exhaustive
re-derivation by a second implementation. Gate 1 passes on the
code-released disjunct. Its bearing here is specific rather than
general:

- Its observation-tier **broken-`@import`-in-context-file** rule fired
  at 11/23 (48%) real defects across 3,171 repositories. The benign 52%
  were files created at runtime — a category these four are not in, as
  every target exists on disk and always has.
- Its **`cannot-work` consequence family** is defined by silence: the
  configuration parses, the component never runs, no error surfaces.
  That is this defect exactly.
- It supplies the framing that makes the `used_by:` question sharp:
  **the harness is a dependency layer with no dependency hygiene**. A
  declared dependency that is never resolved is the canonical thing a
  lockfile would catch, and there is no lockfile.
- Its **`creates:` proposal** — a frontmatter field letting a skill
  declare outputs, because 40% of unresolved references in its corpus
  are planned outputs — is the same move in the other direction:
  distinguish "this path is an input I load" from "this path is
  something I merely name." `used_by:` needs that same distinction.

The note that flagged this (written 2026-09-14) guessed the cause was
the `@import` token and said so, flagged rather than verified. The
measurement confirms the defect and corrects the diagnosis: the token is
wrong *and* the tilde is wrong *and* neither matters, because the
project-tree confinement is unfixable by spelling. Recording that
correction is half the value of this proposal.

Gate 1 rests on one source, and it should be said plainly that the
source supplies the class and the prior, not the finding. The finding is
a local controlled measurement, which for a claim about one client
version on one box is the stronger instrument.

## Simplicity assessment

**Gate 2 passes: this is clarification, and the only alternative
readings both add surface area.** The proposed edit adds five lines to
one existing skill and deletes nothing that works. No new file, no new
check, no new field, no new script.

Forms considered:

- **Retire `/sync-imports` and delete the contract.** The smallest
  possible system, and genuinely tempting — the skill exists to service
  a mechanism that does not work. Rejected as *this* proposal because
  `used_by:` has a second, real use the propagation claim was
  obscuring: it is the only record of which concepts downstream projects
  claim, which is what `/lint` and `/promote-moc` need to judge whether
  a concept is load-bearing. Deleting the record to punish the false
  claim about the record would lose information. Retiring the skill is
  the reviewer's call, not a foregone one.
- **Make the imports work by vendoring concept copies downstream.**
  Rejected: it is `no-copy-paste` inverted, adds a sync obligation with
  no owner, and would make the four concepts' text drift from the hub.
- **Add a `kg_lint.py` check for unresolvable `@import` targets.**
  Rejected on Gate 2 and on the 09-13 precedent: net-new machinery,
  `/lint` is blocked behind a pending proposal, and the check would fire
  on exactly four known lines whose disposition this proposal settles.
  A one-time fix does not need a standing detector.
- **Say nothing and quietly change `used_by:`'s definition in
  `_README.md`.** Rejected: the false claim lives in three files, and
  leaving the skill teaching the wrong form guarantees it propagates to
  the next downstream project.

## Risks & what could make this wrong

- **I may be preserving a mechanism that should be deleted.** The
  honest reading is that the import contract's headline promise —
  propagation without copy-paste — is dead, and a declared-dependency
  ledger is a consolation prize invented after the fact to justify
  keeping the skill. If the reviewer finds `used_by:` has not actually
  informed a `/lint` or `/promote-moc` decision, the right change is
  removal, and this proposal is the wrong shape.
- **Version-specificity**, as in the companion proposal: measured
  against 2.1.275 on 2026-09-20. If a future release widens import
  resolution, the propagation claim becomes true and this edit becomes a
  stale caveat. Cheap to re-test, cheap to revert.
- **`_scratch`'s `used_by:` entry on `hce-evaluation` may be noise.**
  It is dated 2026-04-24 alongside the mle-bench entries; `_scratch` is
  the sandbox slug. It is cited above as live scope, which may overstate
  by one.
- **Four concepts' promotion history is now suspect, and this proposal
  does not audit it.** `hce-evaluation`, `citation-anchoring`,
  `pass-at-k` and `budget-as-ceiling` all carry `used_by:` and all sit
  at `mature`/`active`. Their status was probably earned on their
  sources rather than on `used_by:`, but I did not verify that, and the
  audit is real work the reviewer may want done before accepting the
  "keep the ledger" framing.
- **This touches no evaluation rule and weakens no holdout.** It makes
  the `hce-evaluation` concept's claimed downstream reach honest, which
  is strictly a tightening.

## Recommendation

**Adopt-with-changes.** Adopt the correction — the skill should not
teach a form that loads nothing, and `README.md` should not repeat the
claim. That part is uncontroversial and five lines.

The change I am asking the reviewer to decide is the one I deliberately
did not make: whether `used_by:` survives as a declared-dependency
ledger or the whole contract is retired. My recommendation is to keep it
for now and delete the four dead `@import` lines from
`mle-bench/CLAUDE.md` in the same sweep as the framework-rule migration,
replacing them with a plain prose line naming the four concepts and
where they live. That keeps the signal, removes the inert syntax, and
costs one line per project. But the case for retiring `/sync-imports`
outright is live, and it is stronger than it looks: a skill whose
stated purpose is servicing a mechanism that has never worked is a
skill the system would not add today. If the reviewer's instinct on
reading this is "delete it," that instinct is well founded and I would
not argue.
