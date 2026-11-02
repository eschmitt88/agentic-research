---
kind: paper
title: "Precedent or Intent? What Coding Agents Follow When Evolving Existing Code"
authors: ["Haichao Zhu", "Geyang Xu", "Zhaorui Yang", "Jiyuan Wang", "Qian Zhang"]
institutions: ["University of California, Riverside", "Tulane University", "Independent Researcher"]
year: 2026
venue: "arXiv 2610.38244 (cs.SE), v1 2026-10-28, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2610.38244"
code_url: null
citations: null
source: "raw/papers/zhu2026precedent.pdf"
added: "2026-11-02"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/shared-skill-namespace]]"
tags: ["instruction-files", "claude-md", "agents-md", "repository-precedent", "instruction-vs-code", "priority-rule", "worked-example", "local-vs-sibling", "file-placement", "joint-success", "claude-code", "codex-cli", "flask", "open-source-validation", "reverse-arm"]
---

# Precedent or Intent? What Coding Agents Follow When Evolving Existing Code

## TL;DR

The authors built one small Flask task-manager API with three `tasks`
endpoints ("local") and three `projects` endpoints ("sibling-resource").
Each agent adds one new `tasks` endpoint. Six implementation pairs are
studied, e.g. `"id": 1` vs `"id": "1"`. Both options in a pair pass the
same hidden tests. Four agents are used: Claude Code with Sonnet-4-6 and
Opus-4-8, and Codex CLI with gpt-5.6-sol and gpt-6-sol. There are 15 runs
per cell and 5,625 runs in all, plus a check in four open-source REST repos
(Python, Go, TypeScript).

Three results matter for this graph:

1. **Code beats the agent's own default.** With no endpoints present, all
   four agents pick the same option in every pair (359/360). When every
   existing endpoint uses the other option, they follow the code in 359/360.
   But the task prompt in that stage tells the agent to "follow the
   conventions of the existing code". There is no arm without that line.
2. **Where the code sits matters more than how much of it there is.**
   Three local files using the alternative give 96% adoption (347/360).
   Three sibling-resource files give 4% (13/360). At a fixed count, moving
   the example between local files moves adoption from 0/15 to 15/15 in
   one setting. The effect shows in 3 of 4 tested settings; Opus shows none.
3. **A stated rule beats the code; a code example alone does not
   reliably.** When all six endpoints use the opposite option, a written
   rule with an explicit priority clause gives joint success in 59/60 runs.
   A code snippet alone gives 38/60, and no guidance gives 0/60. The rule
   works the same in the task prompt and in CLAUDE.md / AGENTS.md (60/60
   each). In the open-source repos a CLAUDE.md / AGENTS.md rule moves joint
   success from 0/120 to 117/120.

"Joint success" means the edit passes the hidden functional tests **and**
a classifier finds the target option in the new code.

This is half of the reverse arm that [[literature/papers/xu2026stale]]
lacked. It shows that a correct instruction beats conflicting code when no
meta-rule is present. It does not test xu2026stale's "existing code wins"
sentence against a correct instruction. That is the cell that decides
whether the sentence is safe to adopt.

## Claims

- Existing code records decisions but not why they were made. So an agent
  cannot tell a constraint to preserve from an accident to override.
- Agents share near-deterministic defaults. Each default is "the most
  direct expression" in Python/Flask, e.g. return `task.id` as stored, with
  no wrapper. An alternative needs information from outside the task.
- Repository precedent overrides those defaults. Agents mostly follow the
  resource they are extending, not analogous resources elsewhere.
- Example count does not explain precedent. Placement can matter as much,
  "or even more".
- "New intent must be stated, not only demonstrated." A code example acts
  as "one more precedent that competes with the repository", while a stated
  priority resolves the conflict.
- Functional tests alone cannot verify intent, because both options pass.
  Choice has to be measured separately.
- The authors call for "design-decision traceability" that keeps the
  rationale and priority of past choices. They propose no mechanism.

## Methods

- **Repository.** One purpose-built Flask Task Manager API. It has
  create/update/list endpoints for `tasks` (local) and `projects`
  (sibling-resource), one file each. The new endpoint is
  `GET /projects/<id>/tasks/<id>`.
- **Six pairs (Table 1).** Field casing (`project_id`/`projectId`), Boolean
  naming (`completed`/`is_completed`), ID serialization (int/string),
  response envelope (flat/`{"data": …}`), error handling
  (`return …, 404`/`raise NotFound`), dispatch (if/elif/`LABELS[p]`).
  Hidden tests accept either option.
- **Stage 1 (RQ1).** All six endpoint files are removed, giving a
  "zero-precedent" repo; 360 runs.
- **Stage 2 (RQ2/3).** For each pair there are eight conditions: both
  groups baseline, both alternative, local support 1/3–3/3, and sibling
  support 1/3–3/3 (2,880 runs). A file-placement study holds support fixed
  and moves the example: 75 exploratory runs with an import-order control,
  then 270 replication runs. The task prompt says to follow the existing
  code's conventions without naming the choice.
- **Stage 3 (RQ4).** All six endpoints are set opposite a target. Only
  Sonnet-4-6 and gpt-6-sol are run, on two pairs (ID serialization and
  envelope). Four conditions are given in the task prompt: C0 none, C1 a
  written rule, C2 a code snippet, C3 both (240 runs). The C1 rule reads:
  "Serialize task identifiers as strings in the new endpoint. This
  instruction takes precedence over the ID representations used by
  existing handlers." A delivery experiment (180 runs) compares no rule,
  the rule in the task prompt, and the rule in the harness's native file
  (CLAUDE.md for Sonnet, AGENTS.md for gpt-6-sol). The rest of the task
  prompt is unchanged. The paper does not say whether the native file is
  at project or user level. It also does not say whether Stage 2's "follow
  the existing conventions" line is still in the Stage 3 task prompt.
- **Open-source check.** Four repos: FastAPI RealWorld, FastAPI Clean
  Example, Go REST API, ExpressJS API. Error handling: all four agents ×
  four local/sibling assignments × 15 runs = 960. Response envelope:
  Sonnet and gpt-6-sol, 420 runs. A native-file rule vs no rule for the
  envelope, with all local handlers flat: 240 runs.
- **Analysis.** Exact binomial or Wilson intervals, Cochran–Armitage trend
  tests, Fisher tests, and within-block randomization tests with Holm
  correction. The text mentions "six registered tests" but gives no
  registration link. Adoption is computed over classifiable outputs.
- **Not reported:** harness versions, time or token limits, permission
  modes, and any released artifact URL. The threats section refers to "the
  artifact" for trial accounting, but none is linked.

## Results

- **Defaults (Table 3).** 359/360 runs pick the default. 23 of 24
  pair–agent cells are unanimous. The one deviation is a Sonnet run on
  dispatch.
- **Precedent over default.** With all six endpoints opposite the default,
  359/360 runs follow the code (299/300 in five pairs, 60/60 in dispatch).
- **Local support (Fig. 5).** Pooled adoption is 0% at 0/3, 21% at 1/3,
  49% at 2/3 and 96% at 3/3. It never decreases with support in any of the
  24 pair–agent settings. Thresholds differ by agent and pair. Response
  envelope never saturates for the Claude agents at 3/3 (Sonnet 8/15, Opus
  9/15).
- **Sibling support.** 13 of 1,079 sibling-only runs (1.2%) adopt the
  alternative, all at 3/3 sibling support. Opus is 0/270. In 22 of 45
  Sonnet ID-serialization transcripts with sibling support, the agent
  discusses the conflicting format and still follows the local files.
  When siblings agree with local code they reinforce it: Sonnet envelope
  goes from 8/15 to 15/15 (p = 0.0063).
- **Placement (Fig. 6).** Exploratory, Sonnet ID: 1/15, 3/15 and 10/15 by
  file. Replications: gpt-6-sol ID at 2/3 goes from 0/15 to 15/15. Sonnet
  field casing at 1/3 goes from 0/15 to 11/15. Opus ID shows no effect
  (p = 0.148). The most influential placements all include `tasks_list`.
  One example in `tasks_list` (11/15) beats two in other local files (0/15).
- **Guidance (Fig. 7).** Joint success per cell (Sonnet ID, gpt-6-sol ID,
  Sonnet envelope, gpt-6-sol envelope):

  | Condition | Cells | Pooled |
  |---|---|---|
  | C0 none | 0, 0, 0, 0 | 0/60 |
  | C1 rule | 15, 14, 15, 15 | 59/60 |
  | C2 example | 0, 15, 8, 15 | 38/60 |
  | C3 both | 15, 15, 15, 15 | 60/60 |
  | Rule in task prompt | 15 each | 60/60 |
  | Rule in native file | 15 each | 60/60 |

  The rule-vs-example gap is entirely Sonnet. gpt-6-sol follows the
  example 30/30. 238 of 240 runs pass the hidden tests. The channel
  comparison is at ceiling, so it cannot rule out a gap of about 6 points
  either way (the paper says so).
- **Open-source (Tables 4–6).** Error handling: 948/960 pass, and all 945
  classifiable successes follow the local pattern. Sibling changes have no
  effect (all p = 1.0). Envelope: 0/207 adopt it with flat local handlers.
  With enveloped local handlers, gpt-6-sol adopts it in 11–15/15 per repo.
  Sonnet adopts it 15/15 in two repos, 3–6/15 in RealWorld, and 0/30 in Go
  REST. Native-file rule (Table 5): Sonnet 57/60, gpt-6-sol 60/60, against
  0/60 each without the rule.

## Critique / open questions

- **Against the digest framing: below framing (mild).** Every number in
  the digest summary is in the body and is quoted correctly. The framing
  of what they show is the overstatement.
  1. **"This is the reverse arm xu2026stale lacked": half of it.** In
     Stage 3 the instruction is right by design and the code is opposite,
     and the instruction wins. But no condition contains an "existing code
     wins" rule like xu2026stale's sentence, so the safety question the
     digest names is not decided. The one ambiguity is Stage 2's task line
     "follow the conventions of the existing code". If that line stayed in
     the Stage 3 prompt, then C1 is a specific priority rule beating a
     generic follow-the-code line in the same prompt, which is close to the
     decisive cell. The paper does not say whether it stayed.
  2. **The rule is not a bare description.** C1 carries an explicit
     priority clause ("This instruction takes precedence over … existing
     handlers"). It is also scoped to the new endpoint only. There is no
     arm with the target stated without the priority clause. That is the
     form a CLAUDE.md migration line usually takes ("use logging"), and
     it is the form xu2026stale's sentence would hit.
  3. **The code is not stale.** Existing handlers stay correct and are
     not touched. The intent applies only to new code. Nothing tests a
     migration where the instruction marks the existing code as legacy.
  4. **"Written priority rules beat code examples (59/60 vs 38/60)."**
     This rests on two agents × two pairs, and the whole gap is Sonnet
     (0/15 and 8/15 on the example). The text tests C1 vs C0 and C3 vs C1,
     not C1 vs C2 directly.
  5. **"Placement alone moves adoption 0% → 100%."** This is one
     replication cell (gpt-6-sol, ID, 2/3). Opus shows no placement effect.
  6. **Relevance 5 → 4.** The rule-delivery result is direct and clean,
     but the adoption-relevant cell is missing.
- **How the two papers combine.** With no meta-rule, the instruction beats
  the code in both directions. A stale project-file line wins 90% of
  choices (xu2026stale). A correct, priority-stated line in the task
  prompt or native file wins 59–60/60 (this paper). So agents do not judge
  which source is true. They follow the instruction channel. The default is
  right when the instruction is right and wrong when it is stale.
  xu2026stale's user-level sentence fixes the stale case. Whether it breaks
  this paper's case is the cell neither paper runs: the sentence plus a
  correct target rule, with and without a priority clause, in project
  CLAUDE.md. There are reasons to expect the specific rule to survive. It
  names its own precedence, it is scoped to the new code, and xu2026stale's
  own alternative reading is "specific beats generic". But that is
  inference. A bare forward-looking convention line is the case most at
  risk.
- **"Any shift must come from the code itself" overstates Stage 2.** The
  task prompt tells the agent to follow existing conventions. So the
  359/360 "precedent over default" result is partly an instruction to
  follow precedent. The OSS study says "without guidance about the choice"
  and does not say whether a similar line is present. With no
  convention-following line, adoption could be lower. No arm measures that.
- **Fit with the example-as-channel result.**
  [[literature/papers/razmara2026one]] found that the worked example
  decides which of two equal-rank skills takes over, and that a priority
  line barely helped. Here a priority line wins and the example is the
  unreliable part. Both fit the rank reading already in the graph. Here the
  rival is the code, which arrives as tool output; there it was an
  equal-rank instruction. That is our inference, not either paper's. The
  paper's own reading ("an example is one more precedent") also fits:
  against six opposite files, one snippet is outnumbered.
- **Seen-and-ignored, again.** In 22/45 transcripts Sonnet discussed the
  conflicting sibling format and followed the local files. This matches
  xu2026stale's Claude Code, which named the file–code conflict in 79/80
  runs and followed the file. Noticing a conflict does not change which
  source wins.
- **Scale and setting.** One synthetic Flask task family. Fifteen runs per
  cell capture run-to-run variance, not task variance (the paper says so).
  Stage 3 uses two agents and two pairs. The OSS guidance check uses one
  pair, the envelope, which is also the pair where precedent binds least.
  Every pair is a style or shape choice that both options satisfy. No pair
  makes either choice incorrect. The models (Sonnet-4-6, Opus-4-8) are not
  the ones this box runs, and Opus is absent from Stage 3.
- **Process disclosures.** An Opus placement run was misconfigured,
  excluded and rerun. Classifications were revised after outcome audits.
  RQ4 had "technical interruptions" that needed retries. All of this is
  disclosed, but the artifact that documents it is not linked.

## Trust signals

- **Credibility:** 3.
  - For: a clean bidirectional design that varies one factor at a time
    with fixed hidden tests; choice is scored separately from success;
    15 runs per cell; exact tests with Holm correction; per-cell counts
    for every RQ1–RQ3 cell (Fig. 5) and every guidance cell (Fig. 7); an
    import-order control for placement; replication of the main contrast
    in four real repos in three languages; an honest threats section that
    names the excluded run and the revised classifications. The lead
    author's related work (Needle in the Repo) is in ASE 2026.
  - Against: not peer reviewed, 5 days old, no citations. No code or data
    URL in the PDF despite references to an artifact. No harness versions
    or run limits. The guidance study is small (two agents, two pairs).
    The Stage 2 "follow conventions" prompt confound is not discussed.
  - Not 4 because nothing is released and the key Stage 3 prompt content is
    under-specified. Not 2 because the design is clean, the effects are
    near-ceiling, and they replicate across repos.

## Follow-up

- **Relevance:** 4. It is the first measurement in this graph of a
  *correct* instruction against conflicting code, and the first to compare
  task-prompt and CLAUDE.md delivery of the same rule head to head. It adds
  material evidence to:
  - [[concepts/enforcement-boundary-placement]]: delivery channel is a null
    at ceiling, and code placement matters.
  - [[concepts/constraint-pinning]]: a correct pinned rule wins, so the
    sentence's cost is the open cell.
  - [[concepts/typed-enforcement]]: test suites cannot see choice, and a
    rule needs a stated priority.
  Not 5 because it does not test the adoption-relevant cell, uses one
  synthetic task family, and runs Stage 3 on two agents.
- **`/elevate` bearing.** This does not unblock the xu2026stale sentence.
  It does narrow what a safe variant needs to say. On this evidence,
  forward-looking lines in this box's CLAUDE.md and `.claude/rules/` are
  most robust when they state their own precedence over existing files,
  e.g. "applies to new folders even where older ones differ". Then a
  generic code-wins rule has a specific rival to lose to. This is untested,
  so flag it, do not adopt it.
- **Cheap local test (extends xu2026stale's).** Take this paper's Stage 3
  setup or xu2026stale's projects. Cross {no meta-rule, xu sentence in
  user-level CLAUDE.md} × {target rule with priority clause, target rule
  without it, none}, with the rule in project CLAUDE.md. Run Claude Code
  with Opus. That is six cells; at 15 runs each it is 90 runs per pair, no
  GPU. It decides the cell neither paper ran.
- **Candidates** (cited, not yet in the graph):
  - Gloaguen et al. 2026, "Evaluating AGENTS.md: Are repository-level
    context files helpful for coding agents?" (arXiv 2602.11988).
  - Zhu et al. 2026, "Needle in the Repo: Diagnosing maintainability
    failures in AI-generated repository edits" (ASE 2026). Same lead group.
  - Wang et al. 2025, "Beyond functional correctness: Investigating coding
    style inconsistencies in LLMs" (FSE 2025).
  - Same digest, item 17 (arXiv 2610.33201, WhoWins). Position and label
    decide instruction conflicts, which is relevant to where a priority
    clause sits.
