---
kind: paper
title: "Stale Instruction Files Override the Code in Coding Agents: A One-Sentence Fix"
authors: ["Xiaonan Xu", "Wenjing Wu"]
institutions: ["Georgia Institute of Technology", "University of Colorado Boulder"]
year: 2026
venue: "arXiv 2610.30571 (cs.SE), v1 2026-10-22, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.30571"
code_url: "https://github.com/WenJing95/stale-docs-data"
citations: null
source: "raw/papers/xu2026stale.pdf"
added: "2026-10-26"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["instruction-files", "claude-md", "agents-md", "stale-documentation", "instruction-hierarchy", "tool-output-authority", "precedence-rule", "user-level-instructions", "conflict-reporting", "convention-drift", "rewrite-breakage", "claude-code", "codex-cli", "synthetic-repos", "elevate-candidate"]
---

# Stale Instruction Files Override the Code in Coding Agents: A One-Sentence Fix

## TL;DR

The authors built 40 small synthetic Python projects. In each, the
instruction file (CLAUDE.md or AGENTS.md) asks for one convention and every
existing module uses the other, e.g. `print` vs `logging`. Claude Code
(Sonnet 5.5) and Codex CLI (GPT-6 Sol) each added a feature to every
project, twice per condition, with and without one sentence in the
**user-level** instruction file telling the agent to follow the existing
code when documentation and code disagree. That gives 320 runs.

Three results matter for this graph:

1. **The file wins even when the agent has seen the code and named the
   conflict.** Without the sentence, 90.1% of choices (649/720) followed the
   stale file and 4.2% followed the code. Claude Code named the conflict in
   its final reply in 79 of 80 runs and still matched the file in 86.2% of
   its choices in those runs. Codex CLI named it in 2 of 80.
2. **One agent repaired the code toward the stale file.** In 20 of 80 runs
   Codex CLI rewrote existing modules to match AGENTS.md (median 198 lines
   removed from 9 files). When the original tests were put back, they failed
   in 17 of the 20. Claude Code never rewrote existing code.
3. **One instruction-tier sentence reverses the outcome.** With the
   sentence, 91.4% of choices (658/720) followed the code (+87.2 points,
   95% CI 83.6–90.9). No run rewrote existing code, and test pass went from
   89.4% to 100%. The authors' explanation is that the file is delivered as
   an instruction and the code only as tool output, which models are trained
   to rank lower. The data are consistent with that explanation but do not
   isolate it.

The arm that decides whether the sentence is safe to adopt was not run. In
every project the file is the stale side. No condition has the file right
and the code behind, which is the normal state of an intended migration.

## Claims

- Instruction files "describe the code in the form of commands". When the
  code moves, the description becomes false while keeping imperative
  wording and instruction-tier delivery.
- Claude Code inserts CLAUDE.md into a user message right after the system
  prompt, under the preamble "Codebase and user instructions are shown
  below. Be sure to adhere to these instructions. IMPORTANT: These
  instructions OVERRIDE any default behavior and you MUST follow them
  exactly as written." Codex CLI wraps AGENTS.md in an INSTRUCTIONS tag
  without that line. Both put the user-level file before the project file in
  the same message. (The preamble is verbatim the one in this box's own
  session context, so the delivery claim checks out here.)
- Under the instruction hierarchy (Wallace et al. 2024; IH-Challenge 2026)
  tool output carries no authority of its own. That ordering exists for
  prompt-injection defence, and the same ordering gives a stale line priority
  over the code it once described.
- Because the sentence is itself an instruction, the conflict becomes one
  between two instructions, and the one that addresses this exact case
  wins. The other lines of the file "keep their effect". This last part is
  asserted, not measured.
- The emphatic Claude Code preamble is not needed for the effect, since
  Codex CLI has no such preamble and followed the file in 93.9% of choices.
  This is a cross-agent comparison and is confounded with the model.
- Recommendations: users add the sentence; teams use the reported conflicts
  to fix the file; agent makers put the rule in the system prompt; model
  makers train models to tell *rules* ("run the tests before you commit")
  from *descriptions* ("we use print"), and to check descriptions against
  the code.

## Methods

- **Projects.** 40 synthetic Python projects, each with 3–8 working
  modules, a passing test suite, an instruction file and a feature request.
  Each feature forces 4–5 choices between two conventions from 16 kinds
  (Fig. 4): messages (logging/print), paths (pathlib/os.path), exception
  type (built-in/project classes), error signalling (raise/return
  (value, error)), test file names, test style, records
  (dataclass/namedtuple), function names (snake/camel), Optional vs
  `X | None`, absolute/relative imports, `typing.List`/`list`, f-strings vs
  `str.format`, docstring style, money (integer cents/Decimal),
  configuration (dict/dataclass), fixed sets of values (Enum/string
  constants). Each kind appears in 11–12 projects, 5–6 in each direction,
  so a model's prior for one convention cancels. Feature requests do not
  mention conventions.
- **The sentence** (user-level `~/.claude/CLAUDE.md` or
  `~/.codex/AGENTS.md`):

  > When a repository's documentation and its existing code disagree,
  > follow the conventions used in the existing code, and in your final
  > reply point out each inconsistency you found between the documentation
  > and the code.

- **Agents.** Claude Code 2.1.302 with Claude Sonnet 5.5
  (`claude -p --model sonnet --permission-mode acceptEdits`, auto-memory
  disabled, a dedicated `CLAUDE_CONFIG_DIR` per condition, python/pytest
  allowed, pip denied). Codex CLI 0.163.0 with GPT-6 Sol
  (`codex exec -s danger-full-access`). Non-interactive, a fresh project
  copy per run, the feature request as the only prompt, 1,800 s limit. Two
  runs per project × agent × condition = 320. All 320 valid (one
  sandbox-rejection rerun).
- **Measures.** A script classifies each added choice as file / code /
  neither. A blinded Claude Opus 5.5 judge (`--safe-mode`, no tools,
  criterion fixed before the first run, replies shuffled with opaque IDs)
  decides whether the final reply names a file–code conflict. A "rewrite" is
  removal of ≥20 lines from pre-existing Python files. After each run the
  original tests are restored and run with held-out feature tests. CIs come
  from 2,000 project-level resamples. Results are reported per agent and per
  kind.
- **Field evidence for staleness** (Section 2) is cited, not measured:
  73.8% of AI config files in 441 company repos were committed once and
  never modified (Denisov-Blanch et al.). 23.0% of 356 public repos have
  instruction-file references to code that no longer exists (Treude &
  Baltes). There are also three hand-picked 2026 cases from Supabase and
  openai/codex, stale for 2–24 days (Fig. 2).

## Results

- **Choice shares (Fig. 5a).**

  | Agent | Condition | Matches file | Matches code |
  |---|---|---|---|
  | Claude Code | without | 86.4% | 7.5% |
  | Claude Code | with | — | 94.4% [91.9, 96.9] |
  | Codex CLI | without | 93.9% [90.7, 96.7] | 0.8% |
  | Codex CLI | with | — | 88.3% [84.8, 91.8] |
  | Pooled | without | 90.1% (649/720) | 4.2% |
  | Pooled | with | — | 91.4% (658/720) |

  The remainder is "neither". File-share with the sentence is not given in
  the text.
- **By kind (Fig. 5b).** The file prevailed for both agents in 14 of 16
  kinds. The exceptions are **Money** and **Configuration**, where Claude
  Code followed the file in fewer than half of its choices. Per-kind numbers
  appear only as bars.
- **Conflict reporting.** Without the sentence: Claude Code 79/80, Codex CLI
  2/80. With it: both agents in all 160 runs. Codex's jump is what the
  sentence's second clause asks for, so it is not evidence about precedence.
  For Claude Code the clause adds nothing; it reported already.
- **Rewrites (Section 5.4, Fig. 7).** Codex CLI without the sentence
  rewrote in 20/80 runs across 12 of 40 projects (median 198 lines, 9
  files). Original tests failed in 17/20. Example: it replaced
  `class Column(Enum)` with module-level string constants as AGENTS.md
  asked, and the restored tests failed at `from kanbanlite.model import
  Column`. The agent's reply reported "14 passed", but those were tests it
  had rewritten itself. With the sentence there were 0 rewrites. Claude Code
  had 0 in both conditions.
- **Tests.** Pass rate 89.4% → 100%, entirely from Codex CLI
  (78.8% → 100%). Every failing Codex run was a rewrite. Claude Code passed
  80/80 in both conditions. Runs with the sentence used slightly fewer tool
  calls and less time (no numbers in the text).

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in the
  digest summary is in the PDF and is quoted correctly. What the digest
  calls a "fix" is validated on one side only. The two ingest checks the
  digest asked for come out as follows.
  1. **"Code precedence" when the file is right: not tested.** Each kind
     appears "in both directions", but that balances *which convention*, not
     *which source is stale*. In all 40 projects the file is wrong by
     construction. A team that writes "use logging" in CLAUDE.md *in order
     to* migrate away from `print` is exactly the case the sentence would
     silence, and the paper has no arm for it. Claude Code's
     reported-and-followed behaviour ("the existing modules … go against
     CLAUDE.md") is the right behaviour in that case. The paper scores it as
     an error only because the design fixes the ground truth. The scope
     section names "the instruction file is the outdated source" but does
     not discuss the reverse cost. Until that arm exists the result
     supports "precedence is steerable with one line", not "code-precedence
     is the right default".
  2. **Trivial style rules: partly.** About 10 of the 16 kinds are pure
     style (naming, imports, typing spellings, string formatting, docstrings,
     test file names). About 6 change interfaces callers depend on
     (exception type, error signalling, records, money, configuration,
     Enum vs constants). That is why Codex's rewrites broke tests. So this
     clears the mcmillan2026instruction toy-marker caveat better than that
     paper did. But no kind makes the *file-following* choice incorrect: both
     conventions are valid in every pair. Mohammadi et al.'s float-money
     case, where following the file introduces a bug, is cited but not
     included. The measured harm is inconsistency plus rewrite breakage, not
     wrong new code. Notably, the one correctness-flavoured kind (Money) is
     where Claude Code already resisted the file.
- **The mechanism is consistent-with, not isolated.** The only manipulation
  is adding the sentence. An alternative reading fits equally well: a more
  specific instruction that addresses the conflict directly beats a less
  specific one, whatever channel the competitor arrived in. Arms that would
  separate them are all absent: the sentence in the project file rather
  than the user file; the code's convention stated as an instruction; the
  file's line delivered as tool output; a placebo sentence of the same
  length. The "preamble not needed" inference compares two agents running
  two different models.
- **The sentence bundles two clauses** (precedence + report every
  inconsistency) with no ablation. For Codex the reporting effect (2 → 80)
  is just instruction-following on the second clause.
- **"Other lines keep their effect" is untested.** "Documentation" in the
  sentence is broad. Nothing checks whether build commands, process rules or
  forward-looking conventions in the same file lose force once the sentence
  is in place. No task in the design exercises such a line.
- **Held-out feature tests across conventions.** For interface-shaping kinds
  (raise vs return-tuple, Enum vs constants) the paper does not say how
  held-out feature tests stay convention-agnostic. Claude Code's 100% pass
  in both conditions suggests they do. The data package would settle it.
- **Scale and setting.** Small synthetic projects (3–8 modules), a uniform
  and perfectly consistent codebase (real stale repos are mixed), one
  unattended turn, two runs per cell, one model per agent (Sonnet, not
  Opus, for Claude Code). The Fig. 2 field cases are hand-picked
  illustrations, not a prevalence estimate.
- **Fit with the rest of the graph.** Two related results are not explained
  by the paper's mechanism story. [[literature/papers/mcmillan2026instruction]]
  found that a contradicting AGENTS.md instruction did not move CLAUDE.md
  compliance (BF₁₀ 0.053), and a trivial top-placed rule bound only about
  65% of the time. Here a file line contradicted by the code bound
  86–94%. Our reading, not the paper's: instruction-vs-instruction
  conflicts are near-ties, instruction-vs-tool-output conflicts are not, and
  conventions with a natural prior bind better than a marker that fights
  one. [[literature/papers/nepal2026faithful]]'s "faithful where it can be
  checked" also points the other way. These convention choices *are*
  checkable against the code, and the agent did check (Claude reported the
  conflict) and complied with the file anyway. Checkability by the agent is
  not the same as authority. The staleness half sits beside
  [[literature/papers/suresh2026grounding]] and
  [[literature/papers/li2026autorecsys]]: write-time truth, read-time
  invalidity. Here the stale entry gives no error at all for Claude Code.
  The agent reports it in prose and carries on.
- **Security tension.** The hierarchy the sentence leans on exists to keep
  tool output from redirecting the agent. The sentence asks the agent to let
  file contents (code read via a tool) override an instruction, though only
  for the narrow "documentation vs code conventions" case. The paper does not
  discuss whether a planted file could exploit this. Given
  [[literature/papers/bouras2026authority]] and
  [[literature/papers/leong2026recognition]], that question is worth
  asking before an agent maker puts the rule in a system prompt.

## Trust signals

- **Credibility:** 3.
  - For: clean, simple design with direction-balanced conventions; a
    blinded, separately configured judge with a criterion fixed before the
    first run; project-level resampled CIs; per-agent and per-kind
    reporting; exact agent versions and full command lines (App. A); every
    run valid. Projects, tasks, run records and analysis scripts are
    released (github.com/WenJing95/stale-docs-data, named in the PDF; not
    opened at ingest). The scope section is honest about what was covered.
  - Against: two authors corresponding from gmail addresses (Georgia Tech,
    CU Boulder affiliations), not peer reviewed, no citations yet (4 days
    old). It is the latest of at least seven 2026 preprints by the same pair
    on agent-harness behaviour, which is high throughput. The decisive
    reverse arm and any mechanism-isolating arm are absent. Synthetic small
    repos, two runs per cell.
  - Not 4 because there is no peer review or lab signal and the
    adoption-relevant arm is missing. Not 2 because the artifacts are
    released and the design is clean enough that the main effect (+87
    points) is not in doubt.

## Follow-up

- **Relevance:** 4. It is the second controlled measurement in this graph of
  the box's own instruction channel (after mcmillan2026instruction), and the
  first on conflict between that channel and the code. It names the channel
  rank (instruction over tool output) as a placement variable. It adds
  material evidence to [[concepts/enforcement-boundary-placement]] (a
  user-level line reorders authority), [[concepts/constraint-pinning]]
  (pinning confers authority on stale text, too),
  [[concepts/typed-enforcement]] (rules vs descriptions in an instruction
  file) and [[concepts/skill-library-lifecycle]] (staleness of
  instruction text is quiet). Not 5 because the reverse arm is missing and
  the projects are synthetic.
- **`/elevate` candidate (flag only).** A one-sentence change to the
  user-level CLAUDE.md in claude-system. The bar it clears is simplicity.
  The bar it does not yet clear is the reverse case: this box's CLAUDE.md
  and `.claude/rules/` mostly state *process rules and forward conventions*
  ("every experiment folder has…", "use `/new-experiment`"). Older folders
  may violate these on purpose-of-migration grounds, and a blanket "existing
  code wins" would invert them. Two narrower variants, neither tested in
  the paper:
  - **Report-only.** "When CLAUDE.md or rules disagree with existing files,
    say so in your final reply." Low risk, but Claude Code already reports
    79/80 unprompted, so it buys little here.
  - **Description-scoped precedence.** Precedence for lines that *describe*
    existing code; process rules and conventions marked as a migration
    target keep force.

  Any proposal should cite the missing arm explicitly.
- **Cheap local test.** The released projects plus a mirrored set where the
  file is right and a minority of modules are legacy would measure the
  sentence's cost. Run Claude Code with Opus, the model this box uses. It
  needs no GPU; the cost is agent tokens, about 160 runs of up to 30 min.
- **Candidates** (cited, not yet in the graph):
  - Mohammadi et al. 2026, "The working set of a coding agent: coherence
    debt in repository-scale tasks" (arXiv 2608.16630). First showed
    rule-over-code; includes the float-money correctness case.
  - Treude & Baltes 2026, "Context rot in AI-assisted software development"
    (arXiv 2606.09090). A consistency checker on instruction files; 23.0%
    stale.
  - Huang et al. 2026, "Harness-IF: evaluating instruction following across
    instruction surfaces in coding agents" (arXiv 2608.11727).
  - Wallace et al. 2024, "The instruction hierarchy" (arXiv 2404.13208) and
    Guo et al. 2026, "IH-Challenge" (arXiv 2603.10521). The mechanism's
    basis.
  - Denisov-Blanch et al. 2026, "A few pages of markdown" (arXiv
    2608.25241). 73.8% of AI config files are never modified after the
    first commit.
  - Same digest, 2610.32578 "Lessons Outlive Their Failures". Stale docs
    are the largest failure class in a markdown system of record edited by
    agents.
