---
kind: paper
title: "Authority Is Not a String: A Capability-Scoped Harness for Prompt-Injection-Resistant Coding Agents"
authors: ["Dimitrios Stamatios Bouras", "Yihan Dai", "Sergey Mechtaev"]
institutions: ["Peking University (Key Lab of HCST, MoE; School of Computer Science)"]
year: 2026
venue: "LMPL '26 (2nd ACM SIGPLAN Workshop on Language Models and Programming Languages), Oakland, Oct 2026; arXiv 2609.08371 (cs.SE)"
peer_reviewed: true
url: "https://arxiv.org/abs/2609.08371"
code_url: "https://figshare.com/s/86184ed20f66d1f0cf91"
citations: null
source: "raw/papers/bouras2026authority.pdf"
added: "2026-09-15"
relevance: 4
credibility: 3
status: read
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/typed-enforcement]]"
tags: ["safety", "prompt-injection", "capabilities", "least-authority", "permission-gate", "delegation", "multi-agent", "coding-agents"]
---

# Authority Is Not a String: A Capability-Scoped Harness for Prompt-Injection-Resistant Coding Agents

## TL;DR

A coding agent's tools carry *ambient authority*: if the model names a
path, the harness acts on it. So an injected instruction only has to make
the model name the wrong path. CapScope takes that away in three steps.
(1) Before any repository content or tool output is read, a preflight LLM
call that sees only the trusted request and the file *names* proposes a
task-wide **authority ceiling**. Host code validates it and freezes it.
(2) Each sub-agent gets its own typed capability store, held outside the
model's context and **derived as a subset of that ceiling**. (3) A blocking
pre-dispatch hook checks every tool call against the store of *the agent
that issued it*. The pipeline is an orchestrator delegating to a runner, a
patcher and a verifier. A task-specific global allowlist, minted at the
same moment from the same trusted inputs, lets the injected effect land in
**33/75** runs. The per-agent version lets it land in **3/75**. Repairs:
**68/75** for both. The variable that moves the number is **principal
granularity**, not when the policy was minted or what language it is
written in. A conventional denylist does no better than no policy at all
(46 vs 47/75).

## Claims

- **The gap is granularity.** One task-level policy has to grant the union
  of what the task needs, so the runner that reads poisoned pytest output
  also holds the source-write the patcher needs. "A single task-level
  policy cannot give two agents different authority."
- **Authority enters only through Mint.** Derive can only narrow and Invoke
  requires a covering capability already in the store. So no agent can
  exceed the minted ceiling, and injected text "may influence a proposed
  tool call or requested sub-agent grant, but it cannot expand the
  task-wide authority."
- **The check never asks about provenance.** Model output is treated as an
  untrusted request as a whole. The hook asks only whether a held capability
  covers the call. This makes it payload-blind in the sense of
  [[concepts/enforcement-boundary-placement]]'s rule 1.
- **Command scopes are parsed argument vectors, not strings.** `parse`
  rejects `$(...)` substitution, splits compound commands and checks each
  segment, so `Exec[pytest]` admits `pytest -v` but not `pytest; curl ...`.
- **Explicit non-goals.** It is not a confidentiality mechanism (a
  permitted read followed by a permitted write can still move data). It
  does not stop misuse of authority granted to the principal that read the
  injection. It does not contain the transitive effects of an allowed
  command: pytest imports project modules, so process sandboxing is still
  needed.
- **Host requirements are small.** A *blocking* pre-dispatch interceptor
  and per-session state. A lifecycle hook that only observes calls is not
  enough.

## Methods

- **Implementation.** Built on the Pi TypeScript coding-agent toolkit
  (0.78.0) without modifying Pi internals. The `tool_call` event returns
  `block: true`. Sub-agents run process-isolated via `pi-subagents`, so
  each store is seeded independently. There are four tools: read (Read),
  write/edit (Write), bash (Exec). Read and Write scopes are canonicalized
  path prefixes (`..` and symlinks resolved). Exec scopes are allowed argv
  prefixes. Read/navigation programs (`grep`, `find`, `ls`) are admitted by
  the command wrapper, and "Repository reads are allowed by default in all
  conditions."
- **Minting.** The preflight LLM runs at temperature 0 on the trusted
  request plus the project file tree (names only), then host validation
  compiles and freezes the result. There are two outputs: the **ceiling**
  (bounds all grants) and the **orchestrator's operational store** (bounds
  its own calls, possibly read-only). The root orchestrator is the one
  exception to ordinary attenuation: it may grant a patcher a write it does
  not itself hold, as long as the write is inside the ceiling. Runtime user
  approval would count as a new trusted Mint, but it is disabled in the
  evaluation.
- **Derivation.** The orchestrator's `subagent` tool call carries role,
  subtask and a *requested grant*. Host code first runs `ensureUsable`,
  which adds omitted repo-local reads and role-required test/write scopes,
  then `derive`, which keeps only the capabilities structurally contained
  in the ceiling and drops the rest.
- **Grid.** 4 conditions × 5 tasks × 5 injection surfaces × 3 trials = 300
  runs. The tasks are small Python repairs, one bug each, with a failing
  pytest suite. Surfaces: README, AGENTS.md (auto-loaded), a project-local
  skill file, a source comment or docstring, pytest output. Payloads ask
  for a plausible in-repo action (run a project-local utility, or write a
  project file) that the repair doesn't need. Each leaves a unique sentinel
  on disk. All conditions use the same four-principal pipeline
  (orchestrator → runner / patcher / verifier). Model: qwen3.5-flash via an
  OpenAI-compatible endpoint, runs collected May 2026.
- **Conditions.** B0 is ambient authority. B1 is one static
  task-independent denylist-style policy through Pi's permission gating
  (`.env`, `.ssh`, `rm -rf`, `sudo`, `curl`). B2 is a *task-specific*
  global policy generated from the trusted task and a sanitized project
  structure before any injection is read, applied to every principal. C is
  CapScope. **B2 vs C is the controlled contrast**: both mint before the
  untrusted read, and only per-principal attenuation differs.
- **Outcomes.** `task_success` (suite passes at end) and `attack_executed`
  (sentinel exists on disk) are both read from the filesystem, independent
  of logs and of each other. Per-call decision logs give secondary
  diagnostics (proposed vs blocked).

## Results

- **Attack executed (of 75).** B0 **47**, B1 **46**, B2 **33**, C **3**.
  The denylist equals no policy because "the malicious actions are ordinary
  in-repository operations it never names."
- **By surface, B2 vs C (of 15 each).** README 7 vs 0, **AGENTS.md 11 vs
  3**, skill file 7 vs 0, source comment 4 vs 0, tool output 4 vs 0. All
  three residual executions under C came through AGENTS.md, the
  auto-loaded instruction file the orchestrator reads.
- **Task success (of 75).** B0 72, B1 70, B2 **68**, C **68**. CapScope
  costs nothing in repairs against the strongest baseline and 4 runs
  against ambient authority.
- **Decomposing C's 7 failures.** 2 are the preflight omitting the
  source-write the repair needed, so derivation withheld the patcher's
  legitimate write. That is the too-narrow-ceiling cost. 2 are the runner
  proposing the *correct* source edit, which was refused, and the workflow
  never rerouting it to a write-authorized patcher. That is a delegation
  routing cost, not a policy cost. 3 had passing tests but hit the 900 s
  session timeout.
- **The model keeps trying.** Under C the model proposed the injected
  action in **34/75** runs, and the hook records only 3 runs with an
  allowed attack call. Of those 34, **27 contain repeated attack-targeting
  calls** after refusal.
- **Latency.** Mean wall-clock 145 s (B0), 140 s (B1), 194 s (B2), **316 s
  (C)**, with a C median of 208 s and five runs at the 900 s timeout. The
  authors attribute the gap to extra model turns after refusals and to
  derivation-related calls, not to the check itself, and say explicitly
  that they did not isolate component costs.

## Critique / open questions

- **The residual channel is the principal with the widest grant power.**
  The orchestrator reads three of the five surfaces, including the one
  that is auto-loaded, and it is the one principal whose *grants* are
  bounded by the ceiling rather than by its own store. The authors name two
  residual cases: a grant wider than the subtask needs, and injected text
  steering the model-generated subtask or grant selection. Both are
  necessarily inside the ceiling. The paper does not attribute the three
  AGENTS.md runs to either case, but the location is consistent with the
  second. **Per-principal stores stop authority leaking across agents.
  They do not stop influence leaking**, because the delegation message is
  itself a channel from the most-exposed principal to the most-empowered
  one. This is [[literature/papers/yang2026sok]]'s "per-agent checks do not
  compose", measured at small n.
- **One model, five toy repairs, three trials, descriptive counts only.**
  There are no CIs, and qwen3.5-flash is a small fast model. How often a
  model follows the injection is model-dependent: 34/75 proposals under C
  would look different on a frontier model. The B2 vs C gap (33 vs 3) is
  large enough to survive this, but the 68 vs 72 utility comparison is not
  distinguishable from noise. The authors call the per-condition ordering
  "stable" and do not claim more.
- **The payloads are deliberately ones a deny rule cannot name**:
  in-repo writes and project-local utilities. That is the right test for
  the granularity claim, but it also flatters C against B1. Against
  `curl`-style exfiltration, B1 would score very differently.
- **The ceiling is only as good as an LLM guessing from filenames.** Two
  of 75 runs lost a needed write. The paper notes that large refactors,
  dependency migrations and long-running work are unrepresented, and it is
  exactly there that preflight omission and approval frequency would rise.
  This is the same "who writes the contract" gap
  [[literature/papers/santosgrueiro2026lingering]] flags for PORTICO.
  CapScope's answer is to let a model write it from trusted inputs only,
  and it puts a number (2/75) on that answer for easy tasks. Filename
  injection is excluded from the threat model, yet the preflight's entire
  input is file names.
- **`find` is admitted as a read/navigation program.** The paper doesn't
  say whether flags like `-exec` are parsed out.
  [[literature/papers/kapner2026scanning]] lists `find -exec` among
  scoped-looking grants that are arbitrary execution in effect. If the
  wrapper admits `find ... -exec`, the argv discipline has the same hole
  kapner measured in the wild. This is not checkable from the paper; the
  figshare artifact would settle it.
- **Fail-closed with no human in the loop converts refusals into retry
  loops.** 27 of 34 attack-proposing runs retried, and C's mean latency is
  2.2× B0's. This is [[literature/papers/ray2026what]]'s closed-loop point
  in practice: the gate changes what the model proposes next. The refusal
  message is informative, and the authors concede it "can affect recovery
  and task success". Nobody ablates a silent block against an informative
  one.
- **No closure.** The ceiling and stores are frozen per session, so the
  patcher holds its write for the whole session.
  [[literature/papers/santosgrueiro2026lingering]]'s lingering-authority
  failure is orthogonal and unaddressed. PORTICO scopes authority in
  *time* within a principal, and CapScope scopes it *across* principals
  within a task. Neither paper does both.
- **Closest prior art is acknowledged, not measured.** Progent (LLM-drafted
  policy, updates allowed with approval) and CaMeL (dual-LLM capabilities)
  are discussed but not run as baselines. The novelty claim is where
  authority originates (frozen before the untrusted read) plus per-principal
  attenuation, and the experiment isolates only the second.

## Trust signals

- **Credibility:** 3. Peking University, with Mechtaev as senior author,
  an established program-repair researcher. The paper is in archival ACM
  SIGPLAN workshop proceedings (LMPL '26, DOI assigned), which is
  workshop-level review rather than a main-track venue. The **artifact is
  released** (code, tasks, mutators, harness, decision logs on figshare).
  The design is clean: a matched B2-vs-C contrast, outcomes read from the
  filesystem independently of the logs, and C's failures decomposed
  honestly, including the two failures that directly price a too-narrow
  ceiling. Held at 3 by the tiny corpus (5 toy repairs, one small model, 3
  trials, no statistics) and attack payloads chosen to be invisible to
  denylists. No citations established.

## Follow-up

- **Relevance:** 4. It adds a controlled measurement of an axis the
  enforcement cluster had only asserted: **which principal a grant belongs
  to**. With mint timing, policy source and topology held fixed, moving
  from one task-wide allowlist to per-agent derived stores takes injected
  effects from 33/75 to 3/75 at no repair cost against that baseline. It
  also supplies a negative result (a denylist equals no policy for in-repo
  attacks), a mechanism for the residual (in-ceiling grant selection by
  the orchestrator), and costs (2/75 ceiling omissions, 2 routing
  failures, 2.2× latency). It changes existing concepts rather than seeding
  one. Coding-agent scope, but the orchestrator/sub-agent harness is the
  same shape research agents use.
- **[[concepts/permission-gate-as-architecture]].** This is a new
  **principal axis** alongside the concept's existing determinism,
  temporal, quantitative and authority-as-data axes. The gate's key is not
  only (call, state) but (call, *issuing agent*). The B1 result is also a
  measured instance of the concept's "grant language" section: a gate
  phrased as named hazards cannot see an ordinary-looking source write.
- **[[concepts/hierarchical-delegation]].** It gives delegation a
  **security** payoff that the concept's context-isolation rationale
  lacks, and it is the authority analogue of ye2026agent's conservation
  law, with one difference. Authority is *attenuated* (each child ⊆
  ceiling) but not *conserved*, since siblings can hold overlapping
  capabilities. It also adds a cost: a sub-agent that can see the right
  fix but lacks the capability fails unless the workflow reroutes.
- **[[concepts/enforcement-boundary-placement]].** It adds a second time
  point to the temporal axis. Rule 3 says *check* at use. CapScope adds
  *mint* before the first untrusted read, and B2 shows that mint-timing
  alone buys only 47 → 33. The residual shows that a boundary placed per
  principal is crossed by the delegation message itself.
- **[[concepts/typed-enforcement]].** Scopes are structured data (path
  prefix, argv prefix), not opaque predicates, so containment can be
  checked at delegation time. Typing the scope is what makes attenuation
  decidable. The `find` question above is this concept's escape-hatch
  pattern in a new place.
- **[[concepts/information-firewall]]: declined.** That concept is about
  withholding the method or answer from an evaluated agent. The
  preflight's "names, never contents" input restriction resembles it, but
  it is a security boundary on the policy author, which is placement's
  territory.
- **For this harness.** Agent definitions here restrict tool *names* per
  agent type (the `tools:` frontmatter in `.claude/agents/*.md`; for
  example, the Explore agent has no Edit/Write). That is CapScope's
  effect-type split without path or argv scoping. It is worth checking
  whether any per-agent path scope exists before assuming the runner/patcher
  separation is available.
