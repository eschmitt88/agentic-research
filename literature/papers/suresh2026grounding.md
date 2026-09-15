---
kind: paper
title: "Grounding Agent Memory: Environment-Probing Curation for Enterprise Agents"
authors: ["Susheel Suresh", "Hazel Mak", "Sahil Bhatnagar", "Chhaya Methani", "Alejandro Gutierrez Munoz"]
institutions: ["Microsoft Corporation"]
year: 2026
venue: "arXiv (cs.AI)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.11060"
code_url: null
citations: null
source: "raw/papers/suresh2026grounding.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/llm-wiki-pattern]]"
tags: ["memory", "curation", "write-policy", "verification", "staleness", "procedural-memory", "continual-learning", "cost"]
---

# Grounding Agent Memory: Environment-Probing Curation for Enterprise Agents

## TL;DR

A post-task curator that sees only the finished trajectory and its grade
writes memories that are faithful to that trajectory, and the trajectory is
a partial, often mistaken observation of the world. The authors give the
curator (and only the curator) **least-privilege, read-only environment
tools** and a two-block prompt addition telling it to probe before it
creates or updates a record. Nothing else changes: same task agent,
retriever, schema, and CRUD policy. On CLBench with schema drift (GPT-5.4),
the probing configuration reaches **73% pass vs 39% with no memory** at
**$1.68 vs $3.38** task-agent cost. Most of that gap is memory itself.
Trajectory-only memory already gets **70 ± 16%**, so probing's own
increment is **70 → 73% pass, 20.00 → 22.60 reward, $1.99 → $1.68**, with
overlapping CIs. The more distinctive result is qualitative: probed records
are **executable positive procedures** (join key, filter, grain, current
table name), while trajectory-only records are answer-anchored warnings,
under-specified maps, and table names that went stale after migration. The
curator's own cost is excluded from every cost figure.

## Claims

- **Trajectories bound what a curator can know.** A lemma from one
  trajectory can "(i) memorize an instance answer rather than its
  procedure, (ii) inherit an inefficient path, (iii) assert an unverifiable
  scope, (iv) leave blind spots in unvisited regions, or (v) go stale as
  the world changes." Stronger reflection over the same evidence can't
  recover states the task policy never observed.
- **Verification belongs at curation time, not task time.** Deferring it
  "forces the responding agent to spend scarce tool calls rechecking
  uncertain memories rather than directly solving the current task."
- **It deploys without new authority.** Probes use a read-only subset of
  connectors or MCP tools already registered for the task agent. They
  inherit auth and audit, stay off the user-facing path and the task
  budget, and the curator "receives no production write authority." With
  no safe read surface, the curator falls back to trajectory-only
  curation.
- **Gains come from better write-time evidence, not added task-agent
  capacity.** The task-time interface (read-only `memory_read`) is
  identical across both memory conditions.
- **Probing helps most where the trajectory leaves a gap** (an unresolved
  join, workbook location, or procedure) and adds little where the
  trajectory already supports an actionable record. The authors call this
  "a mechanism interpretation rather than a resolved subgroup effect."

## Methods

- **Roles and write authority.** A fresh task-agent session per task gets
  environment tools and read-only `memory_read`, with no write tool. After
  the task closes, a **non-writing distiller** (no feedback, no tools)
  compresses the raw trajectory into a tagged evidence packet. A separate
  **curator** session gets the packet, the terminal feedback, related
  existing records, and the staged raw trace, plus
  `memory_read/create/update/delete`. It is the **sole writer**, and it
  finishes before the next task is revealed.
- **Record contract.** Each record has a category (pattern | rule | trap |
  schema | policy | interaction), a confidence, an `applies_to` retrieval
  scope, and a one-claim lemma. The curator prompt is PROPOSE → CHECK
  (support, transfer, scope, actionability, current validity, redundancy)
  → RECONCILE (create / update-merge / narrow-correct-delete / skip) →
  COMMIT. It includes "A successful task does not validate every
  intermediate assumption."
- **The probing intervention** (App. D.3). The same prompt plus two
  blocks: a statement of read-only tool access, and a verification nudge:
  "Use the read-only environment tools to verify candidate and existing
  memories before Create or Update whenever correctness, scope, freshness,
  or actionability is uncertain. Probe counterexamples, untouched slices,
  stale mappings, required preconditions, and whether a shorter procedure
  yields the same evidence." Also: "Probe only to evaluate a candidate
  memory, not to solve a future task." That limit is an instruction, not
  an enforced property.
- **Benchmarks.** *CLBench* database exploration: 40 SQL questions over a
  hidden SQLite DB with format traps (dollars vs cents, epoch vs ISO) and
  an **unannounced migration after Q20** that renames tables, splits
  columns, and adds soft deletes. There is also a 30-question no-drift
  schedule. *Adapted APEX-Agents*: 90 management-consulting tasks regrouped
  into six document worlds (11–18 tasks each) with PDF/XLSX/DOCX/PPTX
  discovery via MCP-style Archipelago tools.
- **Systems.** GHCP (No Memory), GHCP + Full ICL (prepend prior
  trajectories), GHCP + Mem (trajectory-only curator), and GHCP + Mem (w/
  Env Probing). Harness: Python GitHub Copilot SDK driving headless Copilot
  CLI over JSON-RPC in containers. e5-base-v2 embeddings.
- **Models and protocol.** GPT-5.4 at xhigh effort for all roles in the
  main studies. The no-drift cross-model study uses Sonnet 4.6 (high) and
  Opus 4.7 (xhigh) for all roles. CLBench: five paired seeded runs per
  configuration, 95% Student-t CIs. APEX: five runs per memory
  configuration and **three** no-memory runs. No-drift: fixed order, five
  paired runs, **SD rather than CI**.
- **Metric.** Pass-discounted reward r = p·(1 − q/B), with B = 15 SQL
  queries (CLBench) or 100 tool calls (APEX), strict all-criteria pass.
  Memory-management calls are excluded from q, and **tokens and cost cover
  the task-agent phase only**. Distiller and curator usage "is tracked
  separately" and is not reported.

## Results

- **CLBench drift, GPT-5.4 (Table 1a).** Pass / total reward / queries per
  question / input tokens / task-agent cost:
  - No memory: 39 ± 4% / 8.60 / 8.8 / 3.14M / $3.38
  - Full ICL: 61 ± 11% / 21.39 / **3.0** / 5.42M / $2.01
  - Mem: 70 ± 16% / 20.00 ± 6.52 / 5.6 / 2.13M / $1.99
  - **Probe: 73 ± 5% / 22.60 ± 2.07 / 4.7 / 1.69M / $1.68**

  Probing is best on pass, reward, input tokens, and cost. Its clearest
  edge over Mem is **variance** (±16 → ±5 pass), not the mean.
- **Learning curve.** Probing already leads Mem at the migration boundary
  (running-mean reward 0.541 vs 0.486) and finishes at 0.565 vs 0.500. No
  memory ends at 0.215. Derived from those numbers, the post-migration
  segment means are ≈0.589 vs ≈0.514, a gap of ≈0.075 against ≈0.055
  before migration. The lead mostly **predates** the drift.
- **No-drift cross-model (Table 1b, ± SD).** Sonnet 4.6: Mem 0.673 ± 0.097,
  Probe **0.748 ± 0.030** (+0.075 over Mem). Opus 4.7: Mem 0.696 ± 0.036,
  Probe 0.721 ± 0.062 (+0.025, inside one SD). The paired no-memory
  baselines differ between the two arms (Opus 0.444 vs 0.458), so the lifts
  are not over an identical control.
- **APEX (Tables 2–3).** All 18 memory-vs-no-memory reward gains (6 worlds ×
  Full ICL / Mem / Probe) are positive. Task-agent calls fall from baseline
  means of 30.0–71.6 to 13.9–28.4. Probe has the best reward gain per
  task-agent dollar in 5/6 worlds and beats Mem on absolute reward in 5/6.
  The largest increments are +1.77 (2a87e5cb) and +1.09 (2f84c98b), and
  941eba66 is −0.04. Full ICL wins raw reward in two worlds (2a87e5cb 3.12,
  d1b705c7 5.47), and in one world it costs more than no memory. In
  941eba66, memory cuts calls from 71.6 to 17.7–19.3 and cost from $54.30
  to $7–9 per run.
- **Record quality (Fig. 3, App. E).** The trajectory-only trap reads: "Do
  not answer with AVG(items_g2.prc_usd) over non-null rows; that produces
  about 52.96, but the benchmark's correct result is 96.23." The probed rule
  reads: "Join items_g2 to taxn_g2 on ref_id; filter cat_lvl=1 and the
  exact cat_nm; keep items_g2.prc>0; then compare against the filtered
  AVG(prc)." A stale "Use attrs_g3" becomes "Use product_attributes_g3."
  In the three matched examples Mem used 4/9/8 queries and Probe 1/2/1.
  In the E.2 case, no memory answers 188 (wrong) in 7 queries, Mem answers
  267 in 9, and Probe answers 267 in 2. The authors say these are selected
  cases and "do not imply that every probed record is complete."

## Critique / open questions

- **The headline credits probing with memory's effect.** "39% → 73% and
  halves cost" compares probing against *no memory*. Against the right
  control, trajectory-only curation, the increment is 3 pass points and
  $0.31, with overlapping CIs on pass and reward (70 ± 16 vs 73 ± 5;
  20.00 ± 6.52 vs 22.60 ± 2.07). "All 18 APEX comparisons positive" is also
  against no memory and includes Full ICL. The probe-specific evidence is
  5/6 APEX worlds (the authors concede overlapping intervals), a Sonnet
  no-drift gain, a near-null Opus gain, and a consistent variance
  reduction.
- **Curator cost is never reported.** Every cost and token figure excludes
  the distiller and curator, and the probing curator issues extra
  environment queries against the same world future tasks will query. Part
  of the "halved" task-agent cost may simply be exploration **moved** from
  the task agent to the curator. Without a curation-phase total, the
  cost-efficiency claim (best reward gain per dollar in 5/6 worlds) is a
  claim about the task agent's bill, not the system's.
- **The check is anchored to an oracle label.** Terminal feedback includes
  the correct answer on failure. The trajectory-only trap quotes "the
  benchmark's correct result is 96.23", and the grader output shows
  "Correct answer: 267". A probing curator that knows the target value can
  search the live DB for a query that reproduces it. That is label-guided
  procedure search, stronger than "verification" in a deployment where no
  one supplies the right answer. How much probing helps when feedback is
  pass/fail only, or absent, is untested, and that is the enterprise case
  the title claims.
- **Stale-memory repair isn't isolated.** The drift schedule is the
  motivating case, but the probe lead is mostly in place *before* the
  migration (≈0.055 of the ≈0.075 post-migration gap). The paper never
  counts how many records probing refreshed, narrowed, or deleted, and
  never measures record-level correctness. Refresh is also event-driven:
  an existing record is only re-checked when it is retrieved as related to
  a *new* task's curation, so a stale record nobody's task touches stays
  stale. The one stale-name example is qualitative.
- **Tools and instruction are confounded.** The probing arm gets tools
  *and* a verification nudge naming counterexamples, untouched slices, and
  shorter procedures. The trajectory-only prompt already asks the curator
  to CHECK "current validity", but with nothing to check against. Neither
  "nudge without tools" nor "tools without nudge" is run.
- **Thin statistics.** Five runs per arm, three no-memory runs on APEX,
  SD instead of CI on the cross-model table, and different paired
  baselines per arm. Same base model for task agent, distiller, and
  curator throughout, so this says nothing about the SkillOS-style
  smaller-curator question ([[concepts/skill-library-lifecycle]]).
- **What the design gets right, apart from the numbers.** It separates
  write authority cleanly (the task agent can't write, the curator can't
  mutate the world, the distiller can't do either), falls back safely when
  no read surface exists, and releases full prompts (App. D). Those are
  importable even if the probe increment turns out small.
- **Open question.** Does an environment check add anything over
  **dependency-scoped revalidation** ([[literature/papers/chen2026fresh]])
  when the record carries pointers to the state it was derived from? A
  probe re-observes the world blind to what the record depended on. A
  dependency pointer would say exactly which table or file to re-read.

## Trust signals

- **Credibility:** 3. An industry preprint from Microsoft Corporation
  (the Copilot product org, so the harness is realistic), not
  peer-reviewed, no citations established, **no code released** (the
  GitHub Copilot SDK link is the harness dependency, not artifacts). It
  earns 3 from a reputable group, two external benchmarks rather than
  authored tasks, 95% CIs on the main tables, full prompt text in the
  appendix, and candid hedges (overlapping intervals and selected cases
  both acknowledged). It stays below 4 because there is no code, curation
  cost is unreported, the headline framing credits probing with memory's
  gain, and n is small (5 runs per arm, 3 for APEX baselines).

## Follow-up

- **Relevance:** 4. It adds a mechanism and an axis to
  [[concepts/verified-memory-writes]] without seeding a new concept.
  Every gate in that concept checks a candidate against the *incoming
  evidence* (TrustMem's faithfulness means adding nothing the evidence
  doesn't support), against its *origin* (TMA-NM, SMSR), or against its
  *form* (SENTINEL). This paper shows the incoming evidence can itself be
  the defect: a record perfectly faithful to a partial, mistaken
  trajectory is still wrong, incomplete, or stale. The fix is **an
  independent re-observation of the world at write time**, and it costs
  no write authority. It also operationalizes
  [[literature/papers/li2026remember]]'s *verify* action (the world is
  authoritative for changing facts) as a curator tool, with a measured
  downstream effect. The effect is modest and not cost-complete, which
  keeps it at 4.
- **Placement versus [[literature/papers/hu2026memory]].** hu shows task
  agents over-trust stored values even when an authoritative tool is one
  call away. This paper doesn't test that. Its architecture makes the
  point partly moot, though: it puts the re-check on a curator whose
  *only* job is checking, rather than hoping the task agent spends budget
  on it.
- **[[concepts/skill-library-lifecycle]].** A third admission mechanism
  beside HDSO's paired rollout ([[literature/papers/shang2026hypothesis]])
  and SkillBrew's held-out verification
  ([[literature/papers/hu2026skillbrew]]). It checks the *claim* against
  the world instead of the *utility* against tasks: cheaper (no rollouts),
  but it certifies correctness and currency, not usefulness. It answers
  [[literature/papers/tang2026memory]]'s "don't distill from raw
  trajectories" differently. Instead of governing a memory hierarchy, it
  re-observes. Its CRUD set also includes **narrow scope** as a named
  operation.
- **[[concepts/llm-wiki-pattern]].** It bears on that concept's open
  question "Where compile-time curation breaks down" for corpora that
  change underneath. This is compile-time curation (post-task, async,
  amortized) in a drifting environment. The compiled index beats Full ICL
  replay on pass rate at a third of the input tokens (1.69M vs 5.42M), and
  a compiler that can re-query the source is the proposed staleness
  answer. The drift-repair evidence is weak (see Critique), so this is a
  partial answer.
- **[[concepts/agent-native-memory]]: declined.** The memory is an
  embedding-indexed record store with a separate sole-writer curator and a
  read-only task agent. That is the opposite of "the same LLM that reasons
  over a task also curates", and there is no comparison against
  agent-writes-its-own-memory, so it offers neither evidence for nor
  against the concept.
- **This repository.** `/ingest` is a trajectory-bound curator in exactly
  this paper's sense: it writes concept prose from one paper's PDF and
  the existing concept file, and its "probe" is whatever the ingesting
  agent chooses to read. The paper's lesson maps onto a concrete
  practice: before a concept edit asserts what another source or the
  repo's tooling does, read it. That is a read-only world check at write
  time, not a recollection.
