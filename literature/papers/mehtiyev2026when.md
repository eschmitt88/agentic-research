---
kind: paper
title: "When Instruction Files Go Stale: Documentation Debt and Hidden Cost in Coding Agents"
authors: ["Tural Mehtiyev", "Biruk Tadesse", "Lawrence Arkoh", "Wesley Assunção"]
institutions: ["North Carolina State University"]
year: 2026
venue: "arXiv 2611.09802 (cs.SE), v1 2026-11-06, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.09802"
code_url: "https://doi.org/10.5281/zenodo.23992004"
citations: null
source: "raw/papers/mehtiyev2026when.pdf"
added: "2026-11-17"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["instruction-files", "claude-md", "agents-md", "stale-documentation", "documentation-debt", "stale-path", "stale-command", "corrected-vs-removed", "no-repair", "silent-success", "pipe-masks-exit-code", "no-tests-ran", "trajectory-analysis", "beyond-task-success", "claude-code", "claude-sonnet-5-5", "claude-haiku-5-5", "real-repos", "elevate-candidate"]
---

# When Instruction Files Go Stale: Documentation Debt and Hidden Cost in Coding Agents

## TL;DR

Three real stale lines from active open-source repos, each a different
kind of guidance: a **test command** (pyflow AGENTS.md, `pytest tests/ifds`
after the tests moved to `tests/analysis/ifds`), a **code location** (cog,
a file that moved), and a **procedure** (aghub CLAUDE.md, "create
`crates/core/src/agents/<name>.rs`" after the folder moved to its own
crate). 22 bug-fix tasks run under three versions of the line (stale /
corrected / removed), with the code and task held fixed, on Claude Code
2.1.316 with Claude Haiku 5.5 and Claude Sonnet 5.5: 732 runs. Four
results matter for this graph:

1. **A stale line binds only when the task routes through it.** With the
   stale line, agents used the dead path in 78/80 Command runs, 55/80
   Procedure runs and 0/84 Location runs. With the corrected line, 0 in
   every case.
2. **The cost is behavioural and invisible to task success.** Fix rates
   are flat across conditions (p = 1.0). The stale `pytest` command collects
   nothing and exits non-zero, but every run piped it through `tail` or
   `head`, so the shell reported success. 19 of Sonnet's 40 stale Command
   runs first "verified" the fix with a test run that ran no tests (0/40
   corrected). Sonnet paid about two extra tool calls per run (7.8 vs 5.8
   median).
3. **Correcting beats removing.** With the line deleted, agents still hit
   the dead path in 1/80 Command and 20/80 Procedure runs (guessing the old
   folder from the code's layout), and Sonnet's effort on the Command case
   was the same as with the stale line (7.8 calls).
4. **No agent repaired the file.** Agents reached the current target in
   130/133 stale-path encounters, mostly by searching (98). Only 21/133
   final replies named the mismatch. 0/133 proposed a repair, and 0/732
   runs edited an instruction file. The task asked for a code fix only, so
   this is default behaviour, not behaviour under permission.

## Claims

- A stale instruction is documentation debt. The principal is a one-line
  edit; the interest is the extra work every later session pays when it
  meets the line. Because agents recover and tasks pass, nothing signals
  the debt, and the line survives later edits of the file (2, 13 and 4
  edits after going stale; Fig. 3).
- What decides whether a stale line costs anything is whether the task
  needs what the line names and offers no other starting point. Every
  Command task ends in running tests; the Procedure line mattered when the
  bug report named only an area; no Location task needed the file.
- Stale paths, unlike stale conventions ([[literature/papers/xu2026stale]],
  Mohammadi et al.), are contradicted by the repository itself: the agent
  hits an error and recovers. The harm shows in the trajectory, not the
  output.
- Agents take the stale path *silently*. Before the first stale call,
  agents cited the instruction file in 2 of 133 runs and opened it in
  none. Sonnet copied the documented command verbatim in 35/40 runs. A
  trajectory reader cannot tell that the instruction file misled the agent.
- Recommendations: maintainers should correct a stale line rather than
  delete it. Harnesses could flag instruction-file paths that do not exist,
  use `pipefail`, and treat "no tests ran" as a failure. Agents could name a
  missing path in the final summary and propose the one-line fix.
  Evaluations of instruction files should report actions, not only
  success, and compare against both corrected and removed versions. None
  of the four safeguards is evaluated.

## Methods

- **Case selection.** GitHub code search for repos with a root AGENTS.md
  or CLAUDE.md. Five requirements: active (not a fork or archived, ≥5
  stars, commits after the last edit to the instruction file); the file
  names a path that existed until a code change moved it and still
  presents it as current; the path *moved*, so a corrected version exists;
  the line asks the agent to act (run, go to, follow steps); a runnable
  test suite. A script extracted every path and checked it against the
  current tree and history. Claude and Codex reviews then confirmed
  whether each flagged line still presented the path as current. Three
  lines passed: one of each kind. The search finds cases; it is not a
  prevalence estimate.
- **Tasks (Fig. 2, Table I).** 22 bugs in the repo's own code, caught by
  its own tests: Command 8 (every fix checked with the stale-command
  tests; 4 reports name the function), Location 6 (bugs next to the stale
  file, none needing it; 3 name the function), Procedure 8 (4 need the
  descriptor code the line points to; 4 name the function). Bugs are
  historical fixes or seeded defects.
- **Conditions.** Stale (line as found), corrected (names the location
  the code moved to, per repo history), removed (line deleted, rest of the
  file kept). Corrected and removed versions change *every* mention of the
  stale path in the instruction files. In the Procedure repo a second
  instruction file already gave the new location in the stale condition.
- **Runs.** Claude Code 2.1.316, medium effort, 30-minute limit and a
  per-run budget cap. The prompt is the bug report only, the instruction
  files load natively, permissions are not prompted, and there are no host
  settings or memory. Each run uses a fresh container built from one fresh
  commit (no history). Blocks of one run per condition per task run
  together in shuffled order. Repeats: 5 (Command, Procedure) or 7
  (Location) per task × condition × model → 240 + 252 + 240 = 732.
  Failed runs were rerun from scratch.
- **Measures.** Fixed = target tests and full suite pass offline on the
  final workspace with test files restored. Stale-path use (ran, read or
  wrote the dead path) is the primary measure. Also tool calls and tokens
  (including cached), as per-task medians. Each stale-path run was coded
  with fixed rules: the purpose of the first stale call, its position
  relative to the first edit, whether the tool flagged an error, how the
  agent reached the current location, and whether messages named the path.
- **Statistics.** Exact tests stratified by task, Holm-corrected, for
  stale-path use and fixes. Effort comparisons use per-task medians with
  bootstrap CIs and are labelled exploratory. A detector bug (relative
  paths after `cd`) was found and fixed post hoc. It changed 19 Sonnet
  Procedure runs, and both versions are in the package.

## Results

- **Table II (stale-path runs / fixed / median calls / median tokens).**

  | Case | Cond. | Haiku path | Haiku fixed | Haiku calls | Sonnet path | Sonnet fixed | Sonnet calls | Sonnet tok (k) |
  |---|---|---|---|---|---|---|---|---|
  | Command | stale | 38/40 | 39/40 | 16.0 | 40/40 | 40/40 | 7.8 | 195 |
  | Command | corrected | 0/40 | 39/40 | 18.2 | 0/40 | 40/40 | 5.8 | 152 |
  | Command | removed | 0/40 | 39/40 | 14.0 | 1/40 | 40/40 | 7.8 | 222 |
  | Location | stale | 0/42 | 42/42 | 7.5 | 0/42 | 42/42 | 5.3 | 157 |
  | Location | corrected | 0/42 | 42/42 | 7.3 | 0/42 | 42/42 | 5.5 | 162 |
  | Location | removed | 0/42 | 42/42 | 7.7 | 0/42 | 42/42 | 5.3 | 157 |
  | Procedure | stale | 24/40 | 39/40 | 12.4 | 31/40 | 40/40 | 7.9 | 294 |
  | Procedure | corrected | 0/40 | 38/40 | 11.2 | 0/40 | 39/40 | 6.8 | 249 |
  | Procedure | removed | 13/40 | 39/40 | 10.5 | 7/40 | 40/40 | 7.8 | 283 |

  Stale vs corrected stale-path use: Haiku p = 2×10⁻²⁸, Sonnet
  p = 2×10⁻³³ (Holm). Fixes: p = 1.0 for both models. Pooled tokens for
  the Command case are 278k / 303k / 352k (stale / corrected / removed),
  and for Procedure 391k / 395k / 339k.
- **Effort.** The only resolved difference is Sonnet on the Command case:
  7.8 vs 5.8 calls. The stale median is higher in 7 of 8 tasks, and the 95%
  interval excludes zero after correcting for the number of estimates.
  Tokens 195k vs 152k, not significant. In 4 of these tasks all 15 runs
  across the three conditions produced byte-identical patches. On the
  Procedure case with a broad report, the stale line cost about two calls
  per task (Haiku 2.25, Sonnet 2.0), and almost nothing with a named
  function (0.0, 0.25). Haiku's runs varied too widely to resolve a
  difference: on the Command case its corrected runs used *more* calls and
  tokens than stale ones (18.2 vs 16.0; 648k vs 536k).
- **Silent failure.** In 64/80 stale Command runs at least one call on
  the stale path ran no tests. Claude Code flagged none as an error,
  because every such call was piped through `tail`/`head`. Haiku also
  suppressed stderr with `2>/dev/null` in 16 of its 38 stale-path runs. The
  missing Procedure folder was flagged as an error more often (20/24 Haiku,
  13/31 Sonnet).
- **Where the agent meets the line.** With a broad report, Sonnet met the
  stale command while reproducing the bug (20/20). With a named function
  it met it while checking the fix (19/20). With a broad Procedure report
  both models went to the stale folder in nearly every run (Haiku 20,
  Sonnet 19 of 20). With a named function, Haiku used it in 4/20 runs and
  Sonnet in 12/20, though the report named the function's current module.
- **Removed.** The Procedure-case stale-path runs under "removed" all had
  broad reports. Agents guessed an `agents` folder inside the core crate
  from the code's structure (Haiku 13, Sonnet 7 of 40).
- **Recovery and disclosure (Table III, n = 133 encounters).**

  | Case | Model | Enc. | Reached target | Via search | Final disclosure | Suggested repair | Edited file |
  |---|---|---|---|---|---|---|---|
  | Command | Haiku | 38 | 36 (95%) | 27 | 3 (8%) | 0 | 0 |
  | Command | Sonnet | 40 | 40 (100%) | 27 | 17 (43%) | 0 | 0 |
  | Procedure | Haiku | 24 | 23 (96%) | 16 | 1 (4%) | 0 | 0 |
  | Procedure | Sonnet | 31 | 31 (100%) | 28 | 0 (0%) | 0 | 0 |

  Recovery took a median of two calls after the first stale-path
  operation (case–model medians from 2 to 4). Sonnet's disclosure depended
  on when it met the line: 17/20 when it met it while checking the fix,
  0/20 when it met it while reproducing the bug. In 22 of Haiku's 120
  Procedure runs, across all three conditions, the final report pointed
  out *other* outdated parts of CLAUDE.md, often saying the file was left
  alone "because it was not part of the bug".

## Critique / open questions

- **What this adds to [[literature/papers/xu2026stale]].** Xu & Wu use
  synthetic projects with stale *descriptions* (conventions), where
  following the file yields valid but inconsistent code and nothing fails.
  Here the stale lines are real *operational* lines (command, path,
  procedure) that the repository contradicts with an error. The output is
  unaffected, and the cost moves into the trajectory. Four things are new:
  (a) the stale-vs-**removed** arm, which xu2026stale lacks; (b) a
  per-case gradient (78 / 55 / 0 of ~80) showing that authority depends on
  task routing; (c) the silent-success mechanism (`| tail` masking a
  no-tests exit); (d) a direct measure of repair (0/732). Xu & Wu's Claude
  Code named the conflict in 79/80 replies. Here Sonnet named the mismatch
  in only 17/71 encounters, which suggests that a stale path met in passing
  and routed around is far less reportable than a convention conflict that
  shapes every edit.
- **Is "removed" ever better than "corrected"?** Not on the primary
  measure: corrected gave 0/244 stale-path runs, removed 21/244. On effort,
  removed has the lowest medians in Procedure (Sonnet 7.8 vs 6.8 corrected
  calls is *higher*; pooled 8.9 vs 9.6 calls and 339k vs 395k tokens is
  lower). On Haiku Command, corrected was the most expensive arm. None of
  these effort gaps is resolved, so the paper gives no evidence that
  deletion beats correction. It also does not measure the maintainer's
  cost of finding the correct path, which is the case where deletion is
  the cheaper repair. The removed arm is also not neutral: the agent
  reconstructs the old structure from the code (20/80 Procedure).
- **Would an agent repair the file if told it may?** Untested. The prompt
  is a bug report and nothing grants or requests documentation upkeep. The
  authors state that this shows "what agents do by default, not what they
  would do if asked". Haiku's "left alone because not part of the bug" is
  scope discipline, not inability, which makes permission the obvious
  missing variable. xu2026stale's sentence moved *reporting* from 2/80 to
  80/80 for Codex, which makes it plausible that a sentence would move
  *proposing* a repair too. Editing is a different act and has no data.
- **Three cases, confounded.** The instruction kind, repo, language and
  task set vary together. Command = Python/AGENTS.md, Procedure =
  Rust/CLAUDE.md. "Kind of guidance" cannot be separated from the case;
  the authors say so. The Location null (0/84) is by design (no task needs
  the file), so it shows that an irrelevant stale line is harmless, not
  that stale locations are.
- **Channel details not given in the main text.** Two of the three lines
  live in AGENTS.md, and the paper says files "load natively" in Claude
  Code. How the AGENTS.md text reached context (a CLAUDE.md copy or
  import?) is not stated, though 78/80 stale-command runs show it did. In
  the Procedure repo a second instruction file already gave the new
  location, yet 55/80 stale runs went to the old folder. Whether that
  second file was loaded at session start or only on directory access
  decides how to read this. It may be the most interesting conflict in the
  paper and is not analysed.
- **Effort statistics are weak.** One resolved effort difference (Sonnet
  Command), per-task medians over 5 repeats, and an exploratory label.
  The "a third more" headline rests on that one cell. Token differences
  were not significant anywhere.
- **Easy tasks.** Small seeded or historical defects; medium effort; a
  single bug report per run. Longer sessions that meet the line several
  times, or a test command reused across a multi-step task, could compound
  the cost. Neither is measured.
- **One vendor, one harness.** Both models are Claude, inside Claude Code.
  The silent-success finding depends on the harness and model choosing
  `| tail`, which may differ on other agents.
- **The safeguards are proposals.** Path-existence flagging, `pipefail`,
  "no tests ran = failure", and a summary-time repair proposal are all
  unevaluated. A path check catches the Command and Location kinds. The
  Procedure line names a folder pattern (`<name>.rs`) that a simple
  existence check would need to resolve to its parent.
- **Honest reporting.** The post-hoc detector fix is disclosed with
  before/after data, the effort analyses are labelled exploratory, and the
  generalization section is restrained. That raises trust in the main
  stale-path result, which is very large and needs no statistics.

## Trust signals

- **Credibility:** 3 — single-university group (NCSU, Assunção's
  software-engineering lab; the same first author has prior agent-trajectory
  work), not peer reviewed, no citations yet (11 days old). Replication
  package on Zenodo (DOI 10.5281/zenodo.23992004; not opened at ingest)
  with search queries, tasks, transcripts and per-run codes. Real stale
  lines with documented history, a clean one-variable manipulation, exact
  tests stratified by task with Holm correction, and a disclosed detector
  correction. Not 4: three cases, one vendor, effort results mostly
  unresolved. Not 2: artifacts released, design clean, and the headline
  stale-path effect is overwhelming.

## Follow-up

- **Relevance:** 4 — the first real-repo measurement of stale lines in the
  instruction channel this box runs on (Claude Code, CLAUDE.md, Claude
  models). It gives the first direct evidence in the graph on correct vs
  delete vs leave, and on whether agents repair the file (no, by default).
  It adds material evidence to [[concepts/skill-library-lifecycle]]
  (removal and repair), [[concepts/typed-enforcement]] (path lines are
  checkable without deciding which side is right) and
  [[concepts/constraint-pinning]] (pinned operational text binds when the
  task routes through it, and its provenance is invisible). Not 5: three
  cases, and the permission arm that would answer this box's question is
  absent.
- **This box.** The user-level and project CLAUDE.md files and
  `.claude/rules/` contain exactly the Command and Location kinds, e.g.
  `J=~/claude-system/coordinator/.venv/bin/claude-coordinator-jobs`,
  `~/claude-system/claude/rules/{evaluation,agency}.md` and the
  writing-guide path. All three resolve today. `kg_lint.py` has no
  instruction-file path check: its staleness checks are date-based
  (candidates, literature, proposals).
- **`/elevate` candidate (flag only).** A deterministic lint pass that
  extracts paths and command targets from CLAUDE.md files and
  `.claude/rules/*.md` and flags those that do not exist. It is cheap and
  needs no judgment about which side is right. The paper names the
  safeguard but does not evaluate it, and three cases are thin evidence of
  frequency here. A second, smaller candidate: the box's hooks already use
  `set -o pipefail`, but agent-issued `pytest … | tail` in sessions does
  not. A PostToolUse check for pytest exit code 5 ("no tests collected")
  would catch the silent verification. This is our inference, not
  measured.
- **Cheap local test of the open question.** Rerun the released
  Command-case tasks with one added line in the project CLAUDE.md
  ("if an instruction in this file is wrong, fix it in the same change and
  say so"), and measure proposed repairs, edited files and collateral edits.
  Five repeats × 8 tasks × 2 conditions on Sonnet is about 80 short runs.
- **Candidates** (cited, not yet in the graph):
  - Treude & Baltes 2026, "Context rot in AI-assisted software
    development" (arXiv 2606.09090): stale references in 23.0% of 356
    repos (also flagged from xu2026stale).
  - Mohammadi et al. 2026, "The working set of a coding agent"
    (arXiv 2608.16630): conflicting standard → 0% preferred decisions.
  - Gloaguen et al. 2026, "Evaluating AGENTS.md" (arXiv 2602.11988).
  - Lulla et al. 2026, "On the impact of AGENTS.md files on the efficiency
    of AI coding agents" (arXiv 2601.20404).
  - Shepard & Albrecht 2026, "Probe-and-refine tuning of repository
    guidance" (arXiv 2606.20512): refines instruction files with
    synthetic bug-fix probes, which is the closest thing to a repair loop.
  - Sahoo et al. 2026, "AgentLens: the lucky pass problem" (arXiv
    2605.12925) and Shu et al. 2026, "TraceProbe" (arXiv 2607.06184):
    process-level evaluation beyond pass rates.
