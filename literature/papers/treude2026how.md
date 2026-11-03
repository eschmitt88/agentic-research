---
kind: paper
title: "How Do Coding-Agent Harnesses Self-Evolve? An Empirical Study of Retrospective Adaptation"
authors: ["Christoph Treude", "Sebastian Baltes"]
institutions: ["Singapore Management University", "Heidelberg University"]
year: 2026
venue: "arXiv 2610.34938 (cs.SE), v1 2026-10-26, preprint (5-page short paper)"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.34938"
code_url: "https://doi.org/10.5281/zenodo.23653700"
citations: null
source: "raw/papers/treude2026how.pdf"
added: "2026-11-03"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/shared-skill-namespace]]"
tags: ["harness-self-evolution", "retrospective", "retro-skill", "claude-code", "claude-md", "known-failures-file", "transfer", "coverage", "consulted-vs-acted", "prior-driven-proposals", "pre-commit", "swe-bench-verified", "human-in-the-loop", "small-n"]
---

# How Do Coding-Agent Harnesses Self-Evolve? An Empirical Study of Retrospective Adaptation

## TL;DR

The authors ran Claude Code 2.1.295 with Opus 5.5 on 30 SWE-bench Verified
issues. After each session they resumed it with `/retro`, a third-party
skill (mattpocock/skills, commit b0618bc) that reviews the finished session
and proposes numbered harness changes for human approval. They coded all
125 proposals, implemented 24, and evaluated 12 changes on later issues
from the same repository: 5 fresh sessions with the original harness and 5
with the changed one per event. This is the same agent and model this box
runs.

Three results matter for this graph:

1. **Most proposals cite the session; lint proposals mostly do not.** 101
   of 125 proposals were coded *experience-driven* (justified by an event
   in the trace). 19 were *prior-driven* (justified without any session
   event), and 18 of those 19 were lint checks. The cause is the skill's
   own text: it treats a repo with no pre-commit hook and no CI lint job
   as "a finding".
2. **A change helped only when it covered the instance the later session
   met.** Of 7 informative transfer events, the 2 changes that covered
   the encountered problem cut it to 0/5 (from 5/5 and 3/5). Of the 5
   that did not cover it, 1 reduced it (5/5 to 2/5) and 4 did not move
   it. In every one of those cases Claude Code read the change (4/4 or
   5/5 consulted). Reading a change is not acting on it.
3. **Changes can add work and fail to fire.** One Sphinx known-failures
   file told Claude Code to treat any unlisted failure as its own. On an
   unrelated control issue, 4/5 adapted sessions ran baseline comparisons
   against 0/5 originally. A bundled "run CI lint before finishing" line
   raised median session time from 88 to 156 s. The one author-written
   pre-commit config was present in 5/5 adapted sessions and was run in
   0/5. Only 9 of 38 proposed executable checks could fire in the task
   environment as written. Task success never differed by more than one
   session between harnesses.

The paper's claim is that harness self-evolution needs a way to show a
change helps on later tasks, and to revise or remove it. That is argued,
not tested: no removal or revision arm exists.

## Claims

- Retaining changes from completed sessions, without retraining, is
  "harness self-evolution". Whether a retained change helps is separate
  from whether it was generated and implemented.
- A change can describe one task rather than the repository, can stay
  outside a later session's context, or can be read without changing what
  the agent does.
- Retrospectives do more than translate recorded events into changes.
  They also recommend practices the skill considers good regardless of
  what happened.
- Instance-specific retained facts (a list of failing tests) transfer
  worse than repository-level facts (the test command).
- Evaluating an adaptation needs three checks: did the targeted problem
  arise, did the agent consult the change, and did its actions change.
  Retained changes need continued evaluation because outdated or
  incomplete instructions can stay ineffective or add work.

## Methods

- **Sessions.** Exploratory sample: 10 issues, two each from Django,
  SymPy, Matplotlib, Astropy and Sphinx, sampled at random from the "15
  min–1 hour" difficulty band. Second sample: 20 issues from Astropy,
  pytest, Sphinx and SymPy, **chosen non-randomly** because exploratory
  sessions there hit environment problems (pre-existing test failures)
  and later same-repo issues existed. Each issue ran non-interactively in
  a fresh Docker container with the repo snapshot and dependencies. The
  prompt gave the issue text and asked for a fix verified with existing
  tests. The same session was then resumed with `/retro`, whether or not
  the patch passed the hidden tests. For the exploratory sample the
  script committed the patch before `/retro`; three remarks about that
  commit were excluded.
- **Proposal coding.** Each numbered `/retro` entry is one proposal. Two
  codes per proposal: *trigger* (experience-driven / prior-driven / weak)
  and *mechanism* (six codes, Table I). Two **LLM coders**, not humans,
  worked independently with one codebook: Claude Code with
  `claude-fable-5-1` at default settings on the full traces, and GPT
  `gpt-6.1-sol` at high reasoning on trace excerpts. Agreement was 109/125
  on trigger (Cohen's κ = 0.67) and 109/125 on mechanism (κ = 0.84). The
  paper says this measures consistency across two inputs, not human
  inter-rater reliability. The first author resolved 91 disagreements,
  "counted per proposal and code". That count exceeds the 32 implied by
  the two agreement figures, so it must include other codes (e.g.
  eligibility); the paper does not break it down.
- **What "regardless of what had happened" means.** A proposal is
  *prior-driven* when it is "justified without relying on session
  events", e.g. "adding a lint hook solely because the repository lacked
  one". 43 proposals were lint-related, across 29 of 30 sessions
  (SymPy's `bin/test quality` counted as linting): 23 experience-driven,
  18 prior-driven, 2 weak.
- **Eligibility and implementation.** Eligible = experience-driven, a
  persistent change in permitted files (context files, hooks, lint/CI
  config), safe, not just fixing the original issue, and with a written
  criterion for spotting the repeated work later. The initial Claude
  coding found 33 eligible (14 exploratory, 19 second). The first author
  reviewed exploratory eligibility *before* implementation and
  second-sample eligibility only *after*, where coders disagreed. The
  authors picked 9 known-failures files and 1 test-command instruction
  from the second sample (types for which another issue reproduced the
  problem), plus all 14 exploratory proposals: 24 implemented, by resuming
  the original sessions in their original containers. 11 of the 24
  produced an evaluable adaptation. The other 13 changed no harness files,
  changed only the environment or non-permitted files, or had no later
  issue reproducing the problem. A 12th adaptation was **author-written**:
  a pre-commit config from a prior-driven proposal, written after Claude
  Code's permission system blocked its implementation. Resolving
  disagreements changed eligibility for 16 proposals, none among the 12.
- **Transfer.** One *transfer event* = one adaptation on one later issue
  from the same repo, chosen and verified in a fresh container to
  reproduce the targeted problem. 5 sessions under the original harness
  and 5 under the adapted one, same prompt, each in a fresh container.
  12 adaptations, 13 events (one adaptation on two tasks). Three
  adaptations also ran on control tasks (unrelated same-repo issues; the
  two Sphinx adaptations shared one control issue). By our count that is
  about 160 transfer sessions. One adapted session ended in an
  authentication error and was excluded.
- **Measures.** *Recurrence*: did the session do the action the change
  targets (e.g. try `pytest` when it is not installed; run a baseline
  comparison on unmodified code for a pre-existing failure).
  *Encountered*: the trace shows the class of problem. *Covered*: the
  change addressed the specific instances met. *Informative*: ≥3 of the 5
  original-harness sessions encountered the problem. *Loaded*: entered
  context. *Consulted*: Claude Code read a referenced file or ran a
  recommended command. Secondary: task success and elapsed time.
- **Labeling.** For the 9 events about pre-existing failures plus their
  controls, two LLM labelers and then the first author labeled all 120
  traces, without labels identifying the harness. The traces themselves
  can reveal the harness (the agent reading transferred files), so this
  is not fully blind. First-author labels matched the LLM labels on
  recurrence in 119/120 (κ = 0.98).

## Results

- **Proposals (RQ1, Table I).** 2–6 per session, 125 total.
  101 experience-driven, 19 prior-driven, 5 weak.

  | Mechanism | Proposals | Sessions | Example |
  |---|---|---|---|
  | Executable check | 38 | 29 | pre-commit hook running flake8 (prior-driven) |
  | Navigation pointer | 24 | 22 | CLAUDE.md line naming `bin/test` |
  | Instruction | 21 | 16 | bug fix must include a failing test |
  | Information access | 20 | 18 | known-failures file |
  | Tool or configuration | 12 | 10 | script printing one line per failing test |
  | None | 10 | 10 | use Edit instead of a Python replace script |

  17 of the 19 prior-driven proposals were executable checks. Only 9 of
  the 38 executable checks could report a failure in the task environment
  as written. 28 of the other 29 needed missing tools or were commit
  hooks, which never fired because task sessions made no commits.
- **Environment problems recur.** 5 of 10 exploratory and 19 of 20
  second-sample traces had a failed shell command, a repeated command, or
  more than one test runner, mostly from pre-existing test failures.
  Known-failures files were proposed in 15 of 30 sessions. Test-runner
  instructions and known-failures files supplied all 11 evaluated
  experience-driven adaptations.
- **Transfer (RQ2).** 7 of 13 events were informative. In 5 events the
  environment reproduced the problem but no original session met it in
  4, and 1 of 5 did in the fifth. The 13th was the pre-commit config
  (present 5/5, read 0/5, run 0/5). Table II, recurrence under the
  original vs adapted harness, and adapted sessions that consulted the
  change:

  | Repo | Adaptation | Original | Adapted | Consulted |
  |---|---|---|---|---|
  | *Covered the encountered instances* | | | | |
  | SymPy | CLAUDE.md line naming `bin/test` | 5/5 | 0/5 | 5/5 |
  | Astropy | known-failures file, 3 tests | 3/5 | 0/5 | 5/5 |
  | (control) | same file, unrelated issue | 0/5 | 0/5 | 0/5 |
  | *Did not cover them* | | | | |
  | Sphinx | known-failures file + AGENTS.md | 5/5 | 5/5 | 5/5 |
  | (control) | same file, unrelated issue | 0/5 | 1/5 | 5/5 |
  | Sphinx | known-failures file + CLAUDE.md | 5/5 | 5/5 | 5/5 |
  | (control) | same file, unrelated issue | 0/5 | 4/5 | 5/5 |
  | SymPy | 2-package known-failures file + script | 4/5 | 4/4 | 4/4 |
  | pytest | known-failures file, 3 tests | 5/5 | 5/5 | 5/5 |
  | Astropy | cache-repair plugin + script | 5/5 | 2/5 | 5/5 |

  In the four no-change events, sessions met failures absent from the
  file (other tests, other packages, other files), read the file, and ran
  baseline comparisons anyway. The Astropy plugin fixed the leap-second
  failure, but sessions hit NumPy failures outside the script's stored
  list, which it reported as new.
- **Significance (our computation, not the paper's).** Two-sided Fisher
  exact at 5 vs 5: SymPy 5/5 → 0/5, p = 0.008; Astropy 3/5 → 0/5,
  p = 0.17; Astropy plugin 5/5 → 2/5, p = 0.17; Sphinx control 0/5 → 4/5,
  p = 0.048. Only the `bin/test` line and the added-work control clear
  0.05 alone. The paper reports counts only, no tests.
- **Cost.** Task success differed by at most one session between harnesses
  in every event. The SymPy CLAUDE.md file also had a second line asking
  Claude Code to run the CI lint checks before finishing. Every adapted
  session ran them, and median elapsed time rose from 88 to 156 s. Time
  also rose in the SymPy and pytest uncovered events and on the Sphinx
  control (82 to 245 s). It fell by 5–28 s in the other four informative
  events.

## Critique / open questions

- **Against the digest framing: at framing.** Every number in the digest
  entry is in the paper and quoted correctly (30 issues, 125 proposals,
  24 implemented, 12 evaluated, 18 of 43, 2/1/4 of 7). "Some added work
  elsewhere" repeats the abstract. In the body that is one clear case
  (4/5 vs 0/5) and one marginal one (1/5 vs 0/5). The entry flags small
  n. What it omits, rather than overstates:
  1. The process has a human in it at every step. The authors chose which
     proposals to implement, chose the later issues, and labeled the
     outcomes. The paper calls it "self-evolution with human selection".
  2. One of the 12 evaluated changes was written by the authors, not by
     the agent.
  3. The coding was done by two LLMs, resolved by the first author. κ is
     LLM-vs-LLM consistency.
  4. The 20-issue second sample was picked for environment problems, and
     evaluation issues were picked to reproduce the targeted problem. The
     paper says this may overstate how often changes help.
  5. The finding that matters most for us is missing from the entry:
     the 4 no-effect changes were consulted in every adapted session.
- **The evidence is about environment problems, not code.** All 11
  experience-driven adaptations are test-runner pointers or known-failures
  lists. The outcome is a behavioural count (did it re-run a baseline
  comparison), not correctness or cost. The paper is careful that a lower
  count does not mean the action was unnecessary, and calls the
  comparison "a reasonable verification step". Several "successes" here
  replace a fresh check with a stored list. That is fine when the list is
  right; the Sphinx control shows a wrong instruction in the list
  creating checks instead.
- **n is tiny.** Seven informative events, 5 sessions per arm, one
  agent, one `/retro` version, four repos in transfer. Two of the three
  positive effects are not individually significant (our Fisher test).
  The covered/uncovered split was made after labeling, and the
  adaptations differ in several ways at once. The paper says the split
  does not establish that coverage explains the difference.
- **Prior-driven ≠ wrong.** The 18 prior-driven lint proposals may be
  good practice. The finding is that they come from the skill's text,
  not from the session. Only one was evaluated (author-written), and it
  never ran because the agent never commits. So the data show prior-driven
  checks were placed where they could not fire. They do not show such
  checks are useless in a workflow that commits.
- **Implementation leakage.** The original sessions, still holding their
  full context, implemented the changes. The written files may contain
  more than the proposal text. The paper names this.
- **No removal arm.** The conclusion ("revise or remove changes that do
  not") is a recommendation. No change was removed, revised, or aged.
- **Fit with the graph.** This is the field version, on our exact
  harness, of points the graph has from synthetic setups.
  [[literature/papers/he2026does]]: reflection does not remove rules.
  [[literature/papers/nakayashiki2026lessons]]: a one-run lesson can be
  noise. [[literature/papers/giampieri2026cortex]]: "never fired means
  unmeasured". Here the pre-commit file never fired, and the uncovered
  known-failures files fired every time and did nothing.

## Trust signals

- **Credibility:** 3.
  - For: two established empirical-SE researchers (Singapore Management
    University; Heidelberg University). Full artifact on Zenodo: scripts,
    prompts, traces, codebook, labels, harness changes. Exact harness,
    model and skill commit pinned. Fresh containers per session, a
    with/without design per event, control tasks for added work, and
    "informative" defined before reading outcomes. Honest threats section:
    selection, implementation leakage, imperfect blinding, grouping after
    labeling.
  - Against: not peer reviewed, 8 days old, no citations. 5-page short
    paper. Five sessions per arm, seven informative events, no statistical
    tests. LLM coders, a single human resolver/labeler (first author).
    Non-random second sample and problem-selected evaluation issues.
  - Not 4 because the effect estimates are anecdotal at this n and the
    paper is not reviewed. Not 2 because the artifact is released, the
    design is clean for its size, and the limits are stated.

## Follow-up

- **Relevance:** 4. It is the first study in this graph of agent-proposed
  harness changes on Claude Code with Opus 5.5, evaluated with and
  without the change on later tasks. It adds to:
  - [[concepts/skill-library-lifecycle]]: the insert step imports the
    proposing skill's own checklist, and loaded-and-consulted changes
    can be inert.
  - [[concepts/enforcement-boundary-placement]]: checks bound to an event
    the session never produces (a commit) do not fire.
  Weaker bearing, no edit drafted:
  - [[concepts/evidence-gated-completion]]: a retained "run CI lint before
    finishing" line was obeyed every session and added time with no
    success change. That fits the existing "generic verify is cost" rule
    at n = 5; it does not add to it.
  - [[concepts/shared-skill-namespace]]: a portable third-party skill
    carried its author's checklist ("no pre-commit or CI is a finding")
    into every repo; 29 of 30 sessions got a lint proposal. That is a
    content point, not a portability one.
  Not 5 because n is tiny, the transfer evidence covers only environment
  problems, and no removal or revision is tested.
- **`/elevate` bearing.** `/elevate` is "self-evolution with human
  selection" too, from literature rather than session traces. Its gates
  (evidence, simplicity, human review) decide *admission*. Nothing checks
  an accepted change afterwards. The paired control re-tests the gates
  on old evidence, not whether the applied change altered behaviour. This
  paper's three-part check is cheap to state: did the targeted problem
  arise after the change, was the change read, did the action change.
  The 16 accepted proposals have no such record. Two cautions from the
  paper apply directly: a change can be read every session and change
  nothing, and a change can add work on unrelated tasks. Also, Gate 1
  would pass this paper alone (code_url set, credibility 3), so a
  proposal resting only on it should be held for small n.
- **Cheap local check.** For one or two accepted proposals with a
  countable targeted behaviour, grep session transcripts before and after
  the apply date for that behaviour and for reads of the changed file.
  No new runs needed. It answers "did it fire, and did it change
  anything" for changes already on the box.
- **Candidates** (cited, not yet in the graph):
  - Lin et al. 2026, "Harness updating is not harness benefit:
    Disentangling evolution capabilities in self-evolving LLM agents"
    (arXiv 2605.30621).
  - Lin et al. 2026, "Agentic harness engineering: Observability-driven
    automatic evolution of coding-agent harnesses" (arXiv 2604.25850).
  - Wu et al. 2026, "ModularRSI: Modular and generalizable recursive
    harness self-improvement" (arXiv 2609.14857).
  - Zhou et al. 2026, "Self-evolving coding agents" (arXiv 2608.03392).
  - Zhang et al. 2026, "Guardrails beat guidance: A large-scale study of
    rules, skills, and persistent configuration for coding agents"
    (arXiv 2604.11088).
  - Lulla et al. 2026, "On the impact of AGENTS.md files on the
    efficiency of AI coding agents" (arXiv 2601.20404, JAWs at ICSE 2026).
  - Galster et al. 2026, "Harness engineering for agentic AI coding
    tools: An exploratory study" (arXiv 2602.14690).
  - Huang et al. 2026, "BenchTrace" (arXiv 2605.29225).
