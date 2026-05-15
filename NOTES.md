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
- Installed weekly `/digest` cron: Mon 7am via
  `~/.claude/schedule/agentic-research-digest.sh`.
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
  1. Added `export PATH="$HOME/.local/bin:$PATH"` to
     `~/.claude/schedule/agentic-research-digest.sh` so cron's
     restricted PATH can find `claude`.
  2. Added `permissions.allow: ["WebSearch", "WebFetch"]` to
     user-level `~/.claude/settings.json` (via the symlink target)
     so headless `/digest` and `/discover` don't fail closed.
  3. Added `--permission-mode bypassPermissions` to the cron
     `claude -p "/digest"` invocation so file writes persist in
     headless mode.
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
