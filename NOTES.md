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
