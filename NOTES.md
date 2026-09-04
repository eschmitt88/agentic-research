# NOTES

Running log of work sessions. `/wrap` appends a new dated section at the
end of each session with **Did / Findings / Next** subsections. The
SessionEnd hook backstops this if you forget.

<!-- entries go below this line, newest at bottom -->

## 2026-04-24 — Phase 4 scaffold

### Did

- Scaffolded `agentic-research` via `/new-project`; GitHub remote
  `eschmitt88/agentic-research` (private) created and pushed on `main`.
- Added the project's Role and Import contract sections to `CLAUDE.md`
  (trimmed to 70 lines to honor the 80-line rule); set
  `budget.yaml.git.auto_push: true` (aspirational — no skill reads it yet).
- Seeded 6 core papers in `literature/papers/`: MLE-STAR, MLE-bench,
  FM Agent, Kosmos, AIRA_2, AIBuildAI.
- Wrote 10 concept notes in `concepts/` plus `concepts/_README.md`
  documenting the import contract (`source_papers:`, `used_by:`,
  `status: active|experimental|retired`).
- Derived 4 seed proposals in `experiments/_proposals/` from the
  highest-signal lit notes.
- Updated `~/.claude/skills/ingest/SKILL.md` with the
  cross-project `@import` back-reference sub-contract (idempotent
  `used_by:` appends, retired-status warnings). Takes effect on
  next session start.
- Installed a weekly `/digest` cron job.
- Ran a seed `/digest` — 3 April-2026 arXiv candidates in-window,
  `_meta/last_digest` populated.
- Ran `/lint` inline: caught one dead wikilink
  (`concepts/ablation-guided-refinement` → folded into
  `evolutionary-expansion`) and fixed it.

### Findings

- Two literature notes (`chan2024mle`, `li2025fm`) are orphans under
  `/lint` check #1 (no `related_experiments:`). Expected by design —
  the setup doc asked for 3-5 proposals from 6 seed papers, so 1-3
  orphans are structural. Graduate them when an experiment actually
  uses them.
- The `@import` contract test (verification step 3) requires session
  restart before `/ingest` picks up the new sub-contract. The
  `_scratch/CLAUDE.md` `@import` line is in place; after restart,
  running `/ingest` on any file in `_scratch/raw/` should trigger
  the back-reference to `concepts/hce-evaluation.md` and also warn
  if the target's `status:` is `retired`.
- `snap`'s `yq` can't read files under `/mnt/projects` — used Python
  `yaml.safe_load` for the frontmatter validation step. If a CI check
  ever needs `yq`, install the binary outside the snap confinement.
- PDFs in `raw/papers/` are in git (~31 MB total), following the
  `/fetch-paper` skill's existing convention. If this grows, move
  to DVC.

### Next

- **Restart Claude** and run the import-contract verification
  (step 3): `/ingest` on `_scratch/raw/<anything>` and confirm
  `concepts/hce-evaluation.md`'s `used_by:` gains
  `{project_slug: _scratch, imported_on: 2026-04-24}`. Re-run to
  confirm idempotency.
- No `/implement` calls on the 4 seed proposals — Phase 4 stays in
  design space.
- Phase 5 (`setup_smoke_test.md`) is the next planned milestone.
- Curation cadence: Monday `/digest` drops candidates into
  `raw/_candidates/`; skim and promote to `/fetch-paper` as warranted.

## 2026-05-12

### Did

- Diagnosed and fixed a three-layer cron failure that had silently
  prevented `/digest` from running since installation:
  Fixed the headless cron invocation (PATH resolution and
  permission configuration) so `/digest` and `/discover` run
  unattended. (Details redacted from the public repo.)
- Re-ran `/digest` manually end-to-end. Produced 6 in-window
  arXiv candidates in `raw/_candidates/2026-05-11-digest.md`;
  `_meta/last_digest` updated to 2026-05-11T20:26:13Z.
- Triaged the 6 candidates against the project's CLAUDE.md scope
  plus the user's expanded interest in *knowledge organization
  for research agents in practice*. Ingested 5 of 6:
  - `kamelhar2026gsar` (GSAR, relevance 5) — typed claim partition
    + tiered recovery as a control signal for grounding.
  - `zhao2026expweaver` (ExpWeaver, relevance 5) — uncertainty-gated
    `[Retrieve]` trigger as decision-time read-side policy.
  - `ouyang2026skillos` (SkillOS, relevance 5) — RL-trained skill
    curator over a markdown skill library; Skill1
    ([arXiv 2605.06130](https://arxiv.org/abs/2605.06130)) folded
    in as architectural-alternative reference, not separate note.
  - `cho2026skillret` (SkillRet, relevance 4) — 17.8K-skill
    retrieval benchmark; empirical scale anchor.
  - `xiong2026autoresearchbench` (AutoResearchBench, relevance 4) —
    1,000-query literature-discovery benchmark with SOTA at ~9%.
- Seeded 3 new concepts (one per architectural layer surfaced by
  the batch):
  - `concepts/typed-claim-partition.md` (write-side: typed
    grounding + tiered recovery, from GSAR).
  - `concepts/selective-memory-retrieval.md` (read-side:
    uncertainty-gated retrieval, from ExpWeaver).
  - `concepts/skill-library-lifecycle.md` (curation policy:
    insert/update/delete as learned operations, from SkillOS).
- Updated 6 existing concepts with new `source_papers:`,
  `related_concepts:`, and body extensions:
  `citation-anchoring`, `hce-evaluation`, `budget-as-ceiling`,
  `agent-native-memory`, `hybrid-model-backends`,
  `web-grounded-literature`.
- Promoted the cluster to `mocs/knowledge-organization-for-research-agents.md` —
  the project's first MoC. Ties 8 concepts across substrate /
  write-side / read-side layers; includes a paper-by-layer table
  showing no batch paper covers all three. `_meta/index.md`
  updated to point to the MoC.

### Findings

- **Stacked silent failures in the cron pipeline.** Three
  independent gating issues (PATH, permissions, write-mode) each
  individually plausible; together they explain weeks of "running"
  cron with zero output. The diagnostic value of running the
  script manually in an `env -i` cron-like environment was high —
  the failure was invisible from the cron logs alone.
- **The May-11 batch papers cluster cleanly into a 3-layer
  architecture** (substrate / write-side / read-side of agent
  memory). This emerged from reading them together, not from any
  individual paper's framing. None of the five papers attacks more
  than one layer — combining a trained write-side curator (SkillOS)
  with a trained read-side gate (ExpWeaver) over an agent-native
  substrate is the natural next research direction and shows up
  in none of the published work.
- **"Good curation is reader-specific"** (SkillOS's strongest
  finding: an 8B trained curator beats Gemini-2.5-Pro used directly
  as curator on the same executor) is the most generalizable single
  insight from the batch. For research-memory projects this implies
  notes should be calibrated to *what future-Claude consulting the
  graph actually retrieves and uses*, not to what looks clean to a
  current human reader. The project's `relevance:` and
  `related_concepts:` frontmatter are the current calibration
  mechanism; making them empirically-driven (which notes do future
  sessions actually pull?) is an open question.
- **AutoResearchBench validates the curator-in-the-loop pattern.**
  Frontier models max out at ~9% on autonomous literature
  discovery; full automation isn't credible at current SOTA. The
  project's design — automated recall (digest), human precision
  verification (user triage + `/fetch-paper`) — matches what the
  evidence supports.
- **Skill1 (2605.06130) is concurrent with SkillOS** (both
  2026-05-07) and takes the inverse architectural bet: unified RL
  policy over selection/application/extraction vs. decoupled
  curator/executor. Neither paper cites the other. A direct
  head-to-head doesn't exist in the literature yet.
- **MoC threshold was already met implicitly before this batch** —
  agent-native-memory, citation-anchoring, web-grounded-literature,
  file-as-bus, structured-world-model are already 5 concepts on
  the knowledge-org theme. The batch added 3 more and made the
  thematic coherence undeniable, which is what made this the right
  moment to promote.

### Next

- **Monday 2026-05-18 7am cron `/digest` validation.** The manual
  run works; cron path *should* work (env-stripped manual
  invocation succeeded), but worth a glance at
  `_meta/digest.log` and `_meta/last_digest` Monday morning.
- **Three high-leverage read-side tweaks for `/digest` and
  `/iterate`** (the highest-leverage improvement frontier per the
  MoC's cross-layer analysis):
  1. Revise `/digest`'s query synthesis to produce longer,
     scenario-rich queries (AutoResearchBench finding: short
     keyword queries degrade scientific search).
  2. Investigate swapping `/digest`'s WebSearch backend for
     arxiv-specialized full-text retrieval (AutoResearchBench
     finding: DeepXiv beats Jina open-web by 1.45pp Deep /
     2.55pp Wide). Natural follow-up read is the DeepXiv-SDK
     paper (`qian2026deepxiv`, arXiv 2603.00084).
  3. Add an explicit `[Retrieve]` trigger mechanism to sub-agents
     in `/iterate` and `/implement` — emit when proposals stall or
     when reasoning entropy spikes, rather than always loading
     top-5 related lit notes upfront (ExpWeaver finding).
- **`/lint` extension** to track insert/update/delete operation
  ratio over time as a knowledge-graph health metric (SkillOS
  finding: a library that's always insert-dominant signals
  under-pruning). Today's `/lint` catches structural defects
  (orphans, dead wikilinks); doesn't characterize lifecycle
  health.
- **No `/implement` calls** on any of the 4 seed proposals —
  Phase 4 of project setup remains in design space, per
  CLAUDE.md ("No `/implement` or `/iterate` in this project").
- **Curation cadence unchanged.** Monday `/digest` → skim
  candidates → promote to `/fetch-paper` as warranted.

## 2026-05-15

### Did

- **`/discover` on agent harnesses.** Wrote
  `raw/_candidates/2026-05-15-agent-harnesses.md` — 10 ranked
  entries covering Claude Code (leaked), OpenClaw, Hermes,
  OpenHarness, Claw Code, plus synthesis writeups. Triage commit
  `6a6e874`. Primary sources (Hermes / OpenHarness GitHub + the
  Paddo leak walkthrough) tagged top-tier; synthesis pieces
  ranked below them.
- **Fetched 3 of the 10 candidates** — the two open-source primary
  repos plus the best public account of the Claude Code leak.
  Skipped the seven synthesis/comparison/profile pieces as
  redundant once primary sources were in `literature/`.
- **Ingested all 3** into the knowledge graph. Three new seedling
  concepts: `scripted-tool-pipelines`, `shared-skill-namespace`,
  `context-eviction-policy`. Five existing concepts gained
  sources: `agent-native-memory`, `skill-library-lifecycle` (with
  a new "execution-time curation" subsection per the Hermes
  evidence), `hierarchical-delegation`, `hybrid-model-backends`,
  `selective-memory-retrieval`.
- **Commit chain:** `6a6e874` (discover) → `1250ca3` /
  `14bac19` (Hermes fetch + ingest) → `5c1452c` / `73b3508`
  (OpenHarness) → `0f271c2` / `5a18fed` (Paddo).

### Findings

- **`context-eviction-policy` is the missing fourth memory axis.**
  The project had `agent-native-memory` (where memory lives),
  `skill-library-lifecycle` (when to write), and
  `selective-memory-retrieval` (when to read). What was missing
  was *what stays in the prompt when the working set overflows*.
  Attested in all four harnesses surveyed today (Claude Code per
  Paddo, OpenHarness auto-compact, Hermes `/compress` + v0.1.6
  release note, ByteRover hierarchical store as the destination).
  Four independent attestations = consensus, not coincidence.
- **`shared-skill-namespace` is real and the project is already
  in it.** OpenHarness reads skills from `~/.claude/skills/`,
  `~/.agents/skills/`, and `~/.openharness/skills/` (and the
  project-level equivalents). Hermes declares agentskills.io
  compatibility. The project's own skills under `~/.claude/skills/`
  are portable across harnesses *by accident* — by virtue of
  following the SKILL.md schema and the canonical layout. This is
  inherited architecture worth being explicit about.
- **Typed-tools vs dynamic-registry is a stable design axis.**
  Claude Code (typed `Tool` interface, Zod) and OpenHarness
  (Pydantic + JSON Schema) sit on one side; Hermes (dynamic
  registry with lambda handlers) on the other. Both ship and pass
  tests — the choice is design philosophy, not capability. Worth
  flagging when the project considers its own tool surface
  evolution.
- **Permission-gate-as-architecture is 3-of-4 attested.** Claude
  Code (19 permission-gated tools), OpenHarness (three permission
  modes + PreToolUse/PostToolUse hooks), Bara's writeup framing
  it as "first-class architecture not bolt-on safety." Held off
  seeding as a concept — one more direct attestation and it's
  worth promoting.
- **Hermes's "trajectory compression for next-gen training" is a
  meta-feedback dynamic** I hadn't seen named: the harness's
  runs *become training data for the next harness model*. If this
  generalizes, the harness's behavioral biases get baked into
  successor models, and the curator/executor split blurs over
  generations. Worth tracking but not actionable here.
- **The harness-architecture cluster is at 3 concepts**
  (`scripted-tool-pipelines`, `shared-skill-namespace`,
  `context-eviction-policy`). Two short of the MoC threshold.
  If/when claw-code or the Claude Code architecture docs get
  fetched, expect a fourth or fifth concept to crystallize and
  trigger MoC promotion.

### Next

- **Test the `shared-skill-namespace` claim empirically.** Drop a
  no-op SKILL.md in `~/.claude/skills/` and verify it loads under
  OpenHarness. If true at the path level, downstream projects can
  rely on one canonical skill directory. If false, the concept
  needs a softer framing.
- **Watch for KAIROS.** Paddo flags 150+ source references to an
  unreleased Claude Code daemon mode. If/when it ships, re-examine
  this project's cron-based `/digest` cadence — KAIROS may be the
  right substrate for what's currently glued together with cron +
  manual session resumption.
- **Defer claw-code fetch.** It's the architecturally most
  substantive of the remaining 7 candidates (clean-room Rust+Python
  rewrite of Claude Code), but the DMCA status is uncertain. Check
  whether the repo is still up before fetching; if it is, the
  fourth concept seed likely lives there.
- **One more attestation and `permission-gate-as-architecture`
  promotes.** Most likely path: read claw-code's permission system
  or the Claude Code MCP transport list more carefully.
- **Items from prior wraps still standing:** Monday 2026-05-18 7am
  cron `/digest` validation; three read-side tweaks to `/digest` +
  `/iterate` (longer queries, arxiv-specialized retrieval,
  `[Retrieve]` triggers); `/lint` extension for insert/update/delete
  ratio. None addressed today.

## 2026-06-03

### Did

- Backfilled trust-signal frontmatter (institutions, peer_reviewed,
  code_url, credibility) on all 16 pre-existing `literature/papers/`
  notes, pulling affiliations from the source PDFs.
- Opted the repo into `agency: max` (`budget.yaml`).
- Ran `/digest`: wrote `raw/_candidates/2026-06-03-digest.md` (10
  candidates), then auto-advanced the top 5 under a GO/high headroom
  verdict — ingested yang2026graph, wu2026gam, du2026memory,
  qu2026coral, starace2025paperbench with trust signals + concept links.
- Auto-promoted `mocs/agent-architecture.md` (substrate / orchestration
  / execution; 5 concepts) via the new `/promote-moc` skill.
- Drained the candidate backlog with `/curate`: all 6 candidate files
  moved to `raw/_candidates/_done/`; 26 notes ingested (24 papers, 1
  repo, 2 OpenClaw posts); claw-code declined (DMCA) and the
  Claude-Code-leak hot-takes declined with reasons; 72 note->concept
  back-links added across 15 concepts.
- Net graph growth: papers 21->44, posts 1->3, MoCs 1->2; uncurated
  candidates 6->0.

### Findings

- The memory tag-cluster is NOT MoC-ripe (3 concepts, 2 already in the
  knowledge-organization MoC); `agent-architecture` was (5 concepts, 3
  un-mapped). `/promote-moc` correctly declined memory + evaluation
  (subsumed) and promoted agent-architecture.
- Two concept seeds now have enough attestations to crystallize:
  `multi-granularity-memory` (yu2026hmem + sun2026rethinking +
  wu2026memory) and `permission-gate-as-architecture` (liu2026dive +
  wang2026reframing — the 4th attestation prior NOTES were waiting on).
- `assumpcao2025codeevolve` `code_url` left null — the abstract's "this
  https URL" could not be resolved confidently from the abs page.
- Backlog contained several peer-reviewed venues (ACL: edwards2025rexbench,
  pham2026memorai; MDPI: calboreanu2026iterative) — set peer_reviewed: true.

### Next

- Seed `multi-granularity-memory` and `permission-gate-as-architecture`
  concepts; with the former, re-check whether the memory cluster reaches
  MoC-ripeness.
- Resolve and fill `assumpcao2025codeevolve` `code_url`.
- Consider widening `agency: max` to the other literature repos and a
  nightly `/curate` + `/promote-moc` sweep.

## 2026-06-23

### Did

- Diagnosed why `agency: max` had left 3 candidate files uncurated since
  2026-06-03: each `/digest` auto-advances only ~3 items under a
  slow/normal cap and *defers the rest, leaving the file in place*; nothing
  ran a follow-up `/curate` to drain the deferrals, so they piled up.
- Ran `/curate` over the full backlog under a GO/high verdict. Ingested all
  7 deferred keepers (fetch + parallel-subagent read/draft, main-agent graph
  wiring): kerestecioglu2026human, jia2026finharness, wang2026act (AARRI),
  pu2026skillops, liu2026evolvemem, lodha2026less, xu2026single (MemGAS,
  ICLR 2026 — peer-reviewed). 0 declines.
- Seeded the two concepts the prior NOTES "Next" flagged:
  `multi-granularity-memory` (6 attestations) and
  `permission-gate-as-architecture` (5 attestations, FinHarness the
  4th-domain anchor). Wired the 7 new notes into 8 existing concepts.
- Reconstructed the missing 2026-06-15 `/curate` log (that digest ingested
  its top 3 in git but never wrote its curation section), and archived all
  3 files to `raw/_candidates/_done/`. Uncurated candidates 3 → 0.
- `/promote-moc`: added `multi-granularity-memory` to the
  knowledge-organization MoC (8 → 9 concepts) rather than spawning a
  redundant memory MoC; declined a new agent-memory MoC and a safety MoC.
- Two commits pushed (1672700 curate batch, 7ae8835 MoC); plus index/log fixes.
- Set up a **scheduled `/curate` + `/promote-moc` cron sweep**
  to drain deferred digest
  items automatically — closes the structural leak. Recorded in
  `docs/decisions/0001-nightly-curate-sweep.md`.
- Removed 4 stale day-1 (2026-04-24) experiment proposals from
  `experiments/_proposals/` — they contradicted the repo charter (no
  experiments in the meta project) and `/lint` flagged them stale. Ideas
  preserved in the concept notes; not relocated, per user.
- Built **`/elevate`** (`~/claude-system/claude/skills/elevate/`): periodically
  evaluates whether well-evidenced ideas in the graph should be adopted into
  claude-system (skills/hooks/rules). Two gates — reputable evidence
  (peer-reviewed / code / >=3 attestations, credibility >=3) AND simplicity
  (prefer remove/consolidate; net-new surface area must justify itself).
  Writes proposals to `docs/system-proposals/` for **human review** only;
  never edits claude-system. Runs on a weekly cron. ADR 0002.

### Findings

- Root cause is structural, not a one-off: `/digest`'s per-run auto-advance
  cap + "defer and leave in place" + no scheduled `/curate` = a slow leak of
  deferred items. The agency.md `/curate` contract ("drain the standing
  backlog") only fires when invoked; it was last run 2026-06-03.
- MoC-ripeness re-check (prior NOTES asked): the memory cluster is still
  *not* ripe for its own MoC — only 4 memory-tagged concepts, and half
  (agent-native-memory, selective-memory-retrieval) are already mapped in
  knowledge-organization. Enrich-existing was the correct move.
- xu2026single (MemGAS) is the strongest of the batch (rel 4 / cred 4 —
  peer-reviewed ICLR 2026 + code): canonical anchor for
  multi-granularity-memory (entropy-routed multi-grain retrieval).

### Next

- **Prevent recurrence**: set up the nightly `/curate` + `/promote-moc`
  sweep the prior NOTES already flagged (cron/schedule), so deferred digest
  items are drained automatically instead of accumulating. Alternatively,
  raise `/digest`'s auto-advance cap under a GO/high verdict so it defers
  less in the first place.
- Resolve and fill `assumpcao2025codeevolve` `code_url` (still open).
- Consider widening `agency: max` to the other literature repos.

## 2026-07-20

### Did
- Cleaned up morning session leftovers: committed the dangling
  `_meta/digest.log` hook-failure line and pushed the unpushed
  session commit.
- Drained the 2026-07-20 digest backlog via `/curate` under a GO/high
  verdict (re-checked mid-burst): **7 ingested, 1 declined** —
  `louck2026securing` (TMA-NM origin-bound memory authority),
  `wang2026naturebench`, `lupidi2026airsbench`, `khan2026token`
  (budget-overrun incident catalog), `cao2026agentsk1`, `yu2026knows`,
  `alzahrani2026persistent`; declined Harness-MU (2606.21856) as a
  marginal 12th source for permission-gate-as-architecture. File
  archived to `raw/_candidates/_done/`. Six of the seven ran as
  sequential subagents (last two on Sonnet per new fan-out policy).
- Concept updates across the batch: verified-memory-writes (5th
  attestation, formal), permission-gate-as-architecture (11th source,
  authority-as-data axis), budget-as-ceiling (first empirical anchor),
  hce-evaluation, programmable-evaluator-oracle, pass-at-k (×2),
  llm-wiki-pattern, structured-world-model, typed-claim-partition,
  file-as-bus (×2), agent-native-memory, context-eviction-policy;
  seeded **concepts/information-firewall.md** (NatureBench's
  method-withholding boundary).
- `/promote-moc`: no un-mapped cluster ripe (memory 5/5, evaluation 7/7
  already covered); folded information-firewall into
  `mocs/evaluation-integrity.md` (now 7 concepts) instead.
- Repaired a pre-existing corruption in
  `concepts/permission-gate-as-architecture.md`: the zhao2026agenticos
  attestation bullet had lost its opening line, leaving an orphaned
  fragment.
- Resolved the open `assumpcao2025codeevolve` `code_url` →
  https://github.com/inter-co/science-codeevolve (verified 200).
- Updated `_meta/index.md` (evaluation-integrity count, 3 new
  highlight papers).

### Findings
- khan2026token is the batch's standout (rel 5): 63 real budget-overrun
  incidents, none prevented before a user paid — converts
  budget-as-ceiling from design stance to empirically-motivated, and
  its evidence clears /elevate's bar (flagged in the note's Follow-up:
  pre-flight spend reservation vs our halt-after-cycle counters).
- louck2026securing sharpens the memory-write story: content- and
  lineage-based write gates are *provably* launderable (TLA+-checked);
  only channel-authenticated origin binding holds. SENTINEL-style
  forensics (karamchandani2026your) is quality layer, not security
  layer — the two papers ingested today+this morning form a clean
  attack/impossibility/construction arc on verified-memory-writes.
- A typed-enforcement thread is accumulating across khan2026token,
  zhao2026agenticos, louck2026securing, madatha2026deterministic
  (flagged in khan note) — watch for concept-promotion ripeness.
- The scheduled curate sweep would have drained today's backlog
  anyway; this session mainly pulled that work forward under an idle
  GO/high window and validated the sequential-subagent ingest pattern.

### Next
- `/elevate` next weekly run should evaluate khan2026token's
  pre-flight spend-reservation idea against the coordinator's current
  halt-after-cycle ceilings.
- Watch information-firewall (single-source seedling) for a second
  attestation; watch the typed-enforcement thread (3+ sources) for
  concept ripeness.
- Still open from prior sessions: consider widening `agency: max` to
  the other literature repos.

## 2026-08-01

### Did
- Applied the full instruction-ablation-program (system-proposal
  2026-07-31, accepted): five phases as five claude-system commits
  (d8be0f7, f934124, dc323da, 4b85862, d7521bd).
- Phase 1 deleted the coordinator admission layer (/plan, job/plan
  CLIs, policy.py, jobs/decisions/session_caps tables incl. live DB,
  pretooluse_cap.sh, dashboard /queue). Telemetry untouched.
- Phase 2 single-sourced the drifted contracts (root-detection
  boilerplate, HCE restatements, Diagnostics key drift, agency batch
  scale).
- Phase 3 compiled prose-programs to scripts/kg_lint.py,
  chain_budget.py, new_project.sh with a 22-assertion smoke suite.
- Phase 4 added the Confirmation principle to agency.md (artifact
  writes auto-commit; hypothesis selection confirms under standard,
  auto-advances under max); propose/derive-experiment now branch on
  agency instead of hard-coding confirm.
- Phase 5 scoped rule loading per-project (@imports; global
  ~/.claude/rules link removed) and made the candidates backlog an
  obligation only in agency: max repos. Migrated 16 project repos and
  pushed all.

### Findings
- Live instruction corpus 18.4k → 16.4k words (-11%): dedup was offset
  by new principle/script-contract text. The bigger wins are
  structural: ~1.6k words of rules no longer load on non-research
  sessions, and lint/halting arithmetic now runs as tested code
  instead of prose executed at inference time.
- Dashboard survived the admission-layer removal cleanly (/ 200,
  /queue 404); status + agency CLIs verified.

### Next
- Watch /propose's agency-branch behavior in the max repos — the first
  autonomous proposal write is the real test of phase 4.
- Deeper prose cuts remain available in ingest/digest if the corpus
  number matters; deferred as conservative-by-default.
- The elevate 2026-06-28/07-19 "already enacted" holds cited the now-
  deleted PreToolUse cap + admission gate; re-examine those holds on
  the next /elevate run.

## 2026-09-04

Catch-up rollup for 2026-08-02 → 09-04 (autonomous-period activity lived
only in journal/ + _meta/log.md; a weekly rollup now runs from the Monday
digest cron so NOTES.md doesn't go stale again).

### Did
- Five weekly digest→auto-ingest→curate cycles (08-03..08-31): papers
  103 → 151. Highlight: tang2026wikiskill (Google Research, arXiv 08-27)
  caught by the 08-31 digest and ingested 09-01 at rel 5 / cred 4 —
  5-day paper-to-graph lag, no intervention.
- Concepts 33 → 34: `enforcement-boundary-placement` seeded 09-01,
  mapped into two existing MoCs (no new MoC ripened; 8 MoCs, 0 orphans).
- /elevate raised two proposals (08-16 precompact-addressable-offload,
  08-23 elevate-paired-control); 08-30 run correctly produced zero.
- Ops hardening (09-04, this session + claude-system@0fa249e): nightly
  /promote-moc now gated on a concept-set change (13 consecutive paid
  no-op declines 08-22..09-03); claude_chain filters the SessionEnd
  "Hook cancelled" leak from tracked logs; decline-logging capped to one
  line; elevate ntfy fails silent without NTFY_TOPIC; sweep heartbeat.

### Findings
- The 08-01 watch items resolved themselves: information-firewall got its
  second attestation (wang2026search, seedling → growing), and the
  06-28/07-19 "already enacted" elevate holds were re-examined post-
  ablation by the 08-23 run — closed with "the machinery should stay
  deleted" (docs/system-proposals/_index.md). Nothing left to do there.

### Next
- Five proposals sit `proposed`/undecided (3× 08-02, 08-16, 08-23) —
  human review queue; one later hold is explicitly blocked behind it.
- Still watching /propose's agency-branch first autonomous write.
- Deeper ingest/digest prose cuts remain available; still deferred.
