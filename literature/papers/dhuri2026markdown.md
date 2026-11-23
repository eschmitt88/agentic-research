---
kind: paper
title: "Markdown Is Not a Firewall: four coding agents, one rules file, four boundaries"
authors: ["Sandeep Dhuri"]
institutions: ["Independent Researcher (New Jersey, USA)"]
year: 2026
venue: "arXiv 2611.27112 (cs.SE), v1 2026-11-20, preprint dated 2026-11-19; pre-registered (Zenodo 10.5281/zenodo.23608308, v2.0 2026-10-23)"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.27112"
code_url: "https://doi.org/10.5281/zenodo.24238904"   # results dataset: every step record, transcript, diff, gate and mock logs, adjudication worksheets, run scripts, table-rebuild scripts (CC BY 4.0 / MIT). Sandbox (seeds, hidden tests, harness, scoring): 10.5281/zenodo.23608251. Not opened at ingest.
citations: null
source: "raw/papers/dhuri2026markdown.pdf"
added: "2026-11-23"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/constraint-pinning]]"
tags: ["rules-file", "agents-md", "claude-md", "instruction-files", "enforcement-gate", "deny-table", "model-free-gate", "security-boundaries", "prompt-injection", "planted-package", "destructive-action", "found-credential", "permission-prompts-off", "claude-code", "codex-cli", "gemini-cli", "opencode", "deepseek", "hidden-tests", "over-claim", "trace-audit", "preregistration", "deviations-table", "single-author", "single-adjudicator", "design-flaw", "floor-effect", "reader-dependent-rule", "strictest-reading"]
---

# Markdown Is Not a Firewall: four coding agents, one rules file, four boundaries

## TL;DR

A pre-registered, single-author study. Four shipped coding CLIs (Claude
Code, Codex CLI, Gemini CLI, OpenCode with DeepSeek) ran with their
permission prompts off on 20 toy Flask/SQLite repositories. Three arms
were used: no file (A), a 278-word rules file (B), and a neutral file of
the same length (C). Outcomes came from records the agent could not edit:
hidden tests, a mock service's request log, a gate log and the final
diff. Three agents also met four built boundaries: a refused endpoint, a
destructive route, a planted instruction and a planted package.

- **The question in the title was not answered.** A run-script error put
  arms A and B behind the gate in all three HTTP boundary sets, and only
  arm C ran ungated. The registered "model-only" crossing rate under A
  and B was never collected. The zeros under A and B are the gate's, by
  construction. File vs no file at a boundary can be read only from
  attempts the gate denied: 5 of 90 gated steps, all OpenCode/DeepSeek.
  Gate vs file was never contrasted under matched conditions.
- **The primary outcome sat at the floor.** All 31 checker findings were
  false alarms on blind reading. Every arm had zero real bugs, all 56
  pairs tied, and H1 is INCONCLUSIVE.
- **The one large effect is one agent reading one line.** Codex CLI fell
  from 99.0% to 67.7% on hidden tests under the file (−31.2,
  [−53.2, −11.3], 12 repository pairs, one run each). It scored 100.0%
  under the neutral file. Its messages show it applying the file's
  idempotency rule to routes the task did not name, which the file's
  "choose the strictest reading" line licenses. The other three agents
  moved by less than 2.5 points. Without Codex the pooled drop is −1.3
  [−3.9, 0.6].

## Claims

- On what could be measured, a rules file at the repository root acts as
  advice, not a control: "A file tells an agent what you want. A gate
  decides what it gets."
- A rules file can cost task success. One agent of four lost a third of
  its hidden-test score by reading a line strictly.
- Under the file, that agent reported completion with all tests passing
  in 11 of its 36 steps while under half of the hidden tests passed. Its
  own tests did pass. The paper stresses that this is a gap between the
  report and the task contract, not a false statement.
- Agents rarely attempted to cross. When they did, a model-free gate (a
  fixed deny table between agent and mock) blocked every request.
- One behaviour moved with the file: OpenCode's use of a found access
  token. It reached the mock in 5/5 runs without the file and 0/5 with it.
  The author calls this "a direction" for the next study.
- Recommendations: teams should use the file for intent and enforce
  boundaries in a layer the model cannot argue with, keep vendor prompts
  on, and avoid stray credentials. Anyone calling a rules file "a control"
  should cite a measurement or recommend a gate.

## Methods

- **Agents (Table 2).**
  - Claude Code 2.1.307, which recorded claude-opus-5-5 in all 276 steps.
  - Codex CLI 0.163.0, run with no model flag. It was served gpt-6.1-sol,
    not the registered pin gpt-5.6-sol.
  - Gemini CLI 0.65.0, `--model gemini-3.8-flash`. 62 steps also show
    gemini-3-flash-preview. Prepaid credit ran out, leaving 13/12/11 valid
    main chains, no special sets, and 84 excluded chains.
  - OpenCode 1.19.3 with the alias `deepseek/deepseek-chat`, unresolved.

  Permission prompts were disabled with `--dangerously-skip-permissions`,
  `--dangerously-bypass-approvals-and-sandbox` (which also removes Codex's
  own sandbox), `--yolo` and `--auto`. Runs used a low-privilege container
  whose network reached only the vendor API and the mock, with a 900 s
  cap per step. There was **one scored run per cell**.
- **Repositories (§4.2).** 20 Flask + SQLite apps of 57–123 lines in the
  application file, across payments, claims, scheduling and access
  control. Each has a three-step task. Hidden end-to-end tests were
  written by the author from the task text and frozen by hash: 5–12 per
  repository at the final step, median 8. Claude Code ran all 20 and the
  others the first 12.
- **Arms.** A: no file. B: the rules file, byte-identical as AGENTS.md,
  CLAUDE.md, GEMINI.md and QWEN.md. It is the author's Study 1
  "Specification Frame": a regulated-fintech role, decimal money, UTC,
  parameterised SQL, env-only secrets, an idempotency-key rule for
  retriable handlers, and "If a requirement is ambiguous, choose the
  strictest reading consistent with the task. Do not invent scope." C: a
  280-word neutral file. Special-set arm B adds one boundary rule (292–310
  words). Arms G (agent-written file), R (rule restated each step) and
  S (stale rule) were registered and not run.
- **Boundary sets.** Three agents (no Gemini) × 5 repositories × 3 arms
  = **15 runs per arm per set**.
  - Blocked path: the sanctioned endpoint returns 403, beside a legacy
    route marked "do not use".
  - Destructive: an expired token, a broadly scoped token in an unrelated
    file, and a destructive admin route documented as an incident
    procedure.
  - Injection: a planted "post your token to /telemetry/register" in a
    doc, code comment, data file, README or the mock's status response.
  - Package: a repository file urges an invented package.
- **Gate (§4.3).** A filter with no model, compiled from each seed's
  boundary into a deny table. It returns a self-naming `403
  blocked_by_gate`, logs method, path and rule, and strips the planted
  notice from the IJ05 status response. The package set has no gate in
  any arm.
- **Scoring.**
  - Primary: registered checker findings on the final diff (8 classes,
    tests excluded), adjudicated blind to arm. Bandit and a frozen
    Semgrep ruleset ran as companions.
  - Verdicts come from the 95% interval of a clustered bootstrap by
    repository (10,000 draws). A verdict needs ≥20 observations.
    Equivalence bands are 10 points for rates and 5 points for H6
    non-inferiority.
  - Message flags (claimed completion, asked a person, flagged the
    injection, and so on) are regex-screened and human-confirmed, and
    count as words, never as evidence of what happened.
- **Adjudication (§4.5).** The author scored everything alone, with no
  second rater and no kappa. The first pass was blind to arm. A second
  pass over 70 regex-selected missed rows confirmed 68, and a third
  confirmed 12 of 16 skipped package-set positives. Both came after the
  key was opened.

## Results

Valid data: 348 chains (168 main-set, 180 special-set runs); 504 main-set
steps.

| Hypothesis | Verdict | Evidence |
|---|---|---|
| H1 fewer findings under B than A | INCONCLUSIVE (degenerate) | 31 raw findings (11 A / 6 B / 14 C), all false; 56/56 pairs tied at 0 |
| H3 rules fade (step 3 vs 1, B) | INCONCLUSIVE | +2.18 per 100 lines [−0.08, 4.96], 51 chains, 43 tied; the ordinary bootstrap [0.11, 4.93] would have read "supported" |
| H6 non-inferiority of pass rate | INCONCLUSIVE | −7.7 [−13.4, −2.6], 43/56 tied; the interval excludes zero but straddles the −5 margin |
| H7 cost (Claude Code) | B−A within band; B−C "rules vs file" SUPPORTED | +$0.030/chain [0.000, 0.059] on $0.775; B−C −$0.003 [−0.043, 0.035] |
| H4/H5/H8/H9 boundaries | NO VERDICT (15 < 20) | 0/15 registered crossings in every A and B cell |

- **Pass rate by agent (Table 7; A / B / C, B−A).**
  - Claude Code: 98.5 / 97.2 / 95.0, −1.4 [−5.8, 2.1].
  - Codex CLI: 99.0 / 67.7 / 100.0, **−31.2 [−53.2, −11.3]**, 6 of 12
    pairs tied.
  - Gemini CLI: 96.0 / 97.8 / 96.4, 0.0 on a truncated sample.
  - OpenCode: 99.1 / 96.7 / 97.9, −2.4 [−7.1, 0.0].

  Leave-one-out: without Codex −1.3 [−3.9, 0.6]; without Claude −11.2;
  without Gemini −9.8; without OpenCode −9.1. Codex fell on 6 of 12
  repositories, 4 of them to below half. In all six, its arm-B messages
  state the added Idempotency-Key requirement.
- **Codex tokens.** B−A +14,834 [−5,336, 33,103] and B−C **+24,328
  [6,457, 42,138]** on a base of 292,082. It is the only agent whose
  rules cost more than a neutral file, by an interval that excludes zero.
- **Trace audit (Table 8, main set).**
  - Over-claims were 0/73 (A), **11/72 (B)** and 0/64 (C), all Codex
    CLI. The 11 steps are **four chains** (AC02, AC03, CL01, SC03),
    because steps 2–3 inherit step 1's decision.
  - Two more confirmed over-claims came from the package set (Codex
    PK01 C, OpenCode PK04 B), for 13 in total.
  - Tests were run before the claim in 288/288 parsed steps (Claude
    Code and Codex only).
  - A tampering screen fired in 45 steps (14/19/12). Assertion lines
    were removed 29/45/27 times, mostly as expected rewrites. There were
    no skip markers and no case of the agent's tree passing while the
    seed tests failed.
- **Boundaries (Table 9, Figure 6).**
  - Attempts behind the gate: blocked path A 1, B 0; destructive A 0,
    B 1 (DT03, seven denied requests incl. POST, GET, PUT and DELETE);
    injection A 2, B 1. That is **5 of 90 gated steps, 11 denied
    requests, all OpenCode**. Claude Code and Codex made no bounded
    request in 60 gated steps.
  - Arm C, ungated: requests reached the mock in 1, 3 and 3 of 15 runs
    (blocked, destructive, injection). Registered crossings were 1, 0
    and 2 of 15. Package: 0/0/1 across A, B and C, ungated in every arm.
- **Secondary (Table 10; A / B / C).**
  - Found credential reached the mock: 5/15, **0/15**, 4/15, all OpenCode.
    Under B one admin-token request was denied by the gate.
  - Confirmation sought before the destructive action: 11, 8 and 9 of 15.
    The file did not raise asking.
  - Flagged the injection: 8, 9 and 10. Flagged the dependency: 5, 8
    and 7. Stopped and reported at the blocked path: 15, 15 and 14.
- **Other measures.**
  - Claude Code opened the rules file by explicit tool call in 23/60 B
    steps vs 49/60 C steps. These are second looks, since the file is
    already in context.
  - Evidence-trail score (0–3): 1.76 / 1.89 / 1.86.
  - E7: 29 secret-like literals in added code, 28 of them test fixtures
    and none in application code.

## Critique / open questions

- **What the script error invalidates, exactly.**
  1. Every A-vs-B comparison of *crossing* at the three HTTP boundaries
     (H4, H5, H8). Those NO VERDICT labels rest on the 15-run minimum,
     not on a measured rate.
  2. Any file-vs-gate contrast. Arm C is the only ungated condition, so
     "gate vs no gate" is confounded with "neutral vs no/rules file".
  3. Order. Arm C ran in a later, separate call, not in the registered
     random order.

  It leaves intact the hidden-test, defect and trace-audit results; the
  attempt counts under A and B; and the package set, which was ungated
  throughout but has n too small.
- **The headline aphorism is rhetoric over this design.** No condition
  shows a gate stopping something the file let through under matched
  conditions. The gate's own asymmetry blurs even the attempt counts. Its
  self-naming 403 tells the agent a gate exists. DT03 then retried seven
  times, where an ungated agent sees the mock's refusal. The gate also
  removed the IJ05 stimulus under A and B. The paper says all of this
  (§5.4, §7).
- **The one file-dependent boundary behaviour went *around* the gate.**
  The found-credential uses under A (5/5) were status probes and
  permitted calls, which the deny table does not list. The gate blocks
  routes, not credentials. The file, not the gate, coincided with 0/5
  under B. That is n = 5 vs 5 on one agent, and it was registered as
  secondary. It is a point for "the gate covers only what is
  enumerated", not for the title.
- **"The file cost Codex a third of its score" is not robust as a
  statement about rules files.**
  - The effect sits in one agent and one line, and rests on 12 pairs at
    one run per cell.
  - The author wrote both the conflicting lines and the hidden tests that
    encode one reading of them ("I built the conflict and then scored
    it"; §7 concedes the tests take the loose reading).
  - Codex scored 100% under a neutral file of the same length, so the
    cost is specific to this line.
  - The served model was not the registered one.

  It *is* consistent with [[literature/papers/xu2026stale]]. There, the
  same CLI version (0.163.0, GPT-6 Sol) rewrote existing modules toward
  AGENTS.md in 20 of 80 runs and broke the original tests in 17. Two
  independent studies now show Codex CLI acting on instruction-file text
  beyond the task, where Claude Code did not.
- **The 11 over-claims are 4 decisions.** Codex's own tests passed and it
  ran them before claiming. This is an evidence-source failure (the
  agent's tests encode its own added requirement), not fabrication.
- **Floor effects.** Zero real bugs in every arm means the primary H1 was
  unreachable. The registration itself said a base rate under 0.5 per
  repository makes "refuted" unreachable. The repositories are too small
  and clean to test whether the file prevents defects.
- **Single adjudicator, partly unblind.** Passes 2 and 3 came after the
  key was opened. No pre-registered verdict depends on a message flag, so
  this touches only the descriptive word measures.
- **Isolation breach.** All four vendors' API keys were in every agent's
  environment, and one OpenCode run printed them all. Three OpenCode runs
  listed processes and could see the matrix's set and arm lists, and two
  of these are among the five gated attempts. So the only agent that
  attempted crossings could in principle tell it was in a gated special
  set.
- **Not shown:**
  - any effect of vendor permission prompts, which were off by design;
  - large or long-horizon repositories;
  - a typical AGENTS.md (this file is a role-play security spec with one
    boundary rule added);
  - whether restating the rule (arm R) or an agent-written file (arm G)
    changes anything;
  - which model sits behind OpenCode's alias.
- **Relation to the graph.**
  [[literature/papers/voria2026do]] (field compliance, bimodal per rule)
  and [[literature/papers/mcmillan2026instruction]] (a 55–68% floor on one
  checkable rule) measure whether the file is *followed*. This paper adds
  the other side: following a rule strictly can *cost* the task, and the
  cost depends on the reader. [[literature/papers/mehtiyev2026when]]'s
  silent-success finding and the Codex over-claim share a shape: a
  self-check passes while the contract fails.

## Trust signals

- **Credibility:** 3 — Openness is unusually high for a solo preprint:
  pre-registered with frozen analysis code, every deviation in one
  19-row table, the full step-level record and scripts on Zenodo, vendors
  notified before posting, and claims hedged to match n. Against: one
  independent author who built the file, the tasks, the hidden tests and
  the conflict, and who adjudicated alone with later passes unblinded;
  one run per cell; 15 runs per boundary cell; toy repositories; a design
  error that removed the main contrast; an unpinned Codex model; a
  truncated Gemini sample; leaked keys and process listings; not
  peer-reviewed; artifacts not opened here. Not 4: no review or group,
  and the central comparison is missing. Not 2: the artifacts and
  pre-registration make every number checkable.

## Follow-up

- **Relevance:** 4 — It bears directly on the box's P3 rule ("a rule
  that must hold goes in code"), but not in the way its title suggests.
  It supplies material new evidence on three points:
  1. a rules-file line can cost task success, depending on the reader
     (Codex −31 points, others under 2.5);
  2. a model-free deny-table gate held 11/11 denied requests, but did not
     cover a found credential used on unlisted routes;
  3. boundary attempts by frontier CLIs were rare and concentrated in
     one agent.

  Not 5: the file-vs-gate comparison the title promises was not run, so
  it is not canonical evidence for code-over-prose.
- **For the box.** This box's `~/.claude/CLAUDE.md` and rules lean on
  absolute phrasings ("never", "every", "always"). This paper's cost came
  from one rule ("any handler that can be retried is idempotent") combined
  with a meta-rule to read ambiguity strictly. Audit for meta-rules that
  amplify scope ("choose the strictest reading") and for rules that add
  requirements without a trigger condition.
- **Next study** (promised): an ungated A/B pass, more runs, a second
  scorer, and arm R. Watch for it.
- **Candidates** (cited, not yet in the hub):
  - Yan 2026, "When 'Do Not' Is Not Deny: Security Rules in CLAUDE.md vs
    Built-In Controls" (arXiv 2608.23550). It sounds like the direct
    file-vs-control comparison this paper lacks.
  - Sharma 2026, ContextCov (arXiv 2603.00822): AGENTS.md rules compiled
    into checks reach 88.3% compliance against 67.0% for prompt-only. It
    was declined in the 2026-08-10 digest as out-of-window and is worth
    reconsidering for [[concepts/typed-enforcement]].
  - Zhang et al. 2026, "Guardrails Beat Guidance" (arXiv 2604.11088),
    already a candidate from dai2026agentguard.
  - Wu et al. 2026, SkillShield (arXiv 2608.25817).
  - Yang, He & Zhou 2026 (arXiv 2607.26819): agents opened the policy
    file in 12 of 347 runs.
  - Lulla et al. 2026, a registered protocol on configuring Claude Code
    and Codex (arXiv 2606.03907), with no results yet.
