---
kind: system-proposal
slug: hce-retrieval-boundary
added: "2026-08-02"
target: "claude/rules/evaluation.md"
change_type: edit
adds_surface_area: true
evidence_citekeys: [wang2026search, wang2026naturebench, wang2026evobrowsecomp]
evidence_strength: "code-released ×2 (wang2026search, wang2026naturebench); 3 attestations across three boundary types; credibility ~4"
status: proposed
recommendation: adopt
---

# Extend the HCE holdout boundary to web retrieval

This is the deferred re-raise of the 2026-07-26 hold ("wang2026naturebench
→ evaluation.md, held on process, not merit — re-raise once
hce-evaluator-integrity is decided"). That proposal was decided (accepted,
applied 2026-07-31, claude-system@e181a3b), and the evidence base has
since grown from one source to three.

## The change

Clause 1 of `claude/rules/evaluation.md` ("`test/` is off-limits during
search") currently draws the holdout boundary purely in file space:

> While proposing, implementing, iterating, expanding, or ensembling,
> agents must not read, list, glob, or sample from the project's
> `test/` directory. This includes derived forms: no `wc -l test/…`,
> no `head`, no `ls test/`, no `dvc pull test/*`, no notebook cell
> that touches `test/`.

Append one paragraph to clause 1:

> **The boundary includes retrieval, not just the filesystem.** During
> search-phase skills, do not web-search or fetch the task's own
> benchmark artifacts — its leaderboard, published solutions or
> write-ups, the dataset's hosting or discussion pages, or the withheld
> source method of a discovery-style task. Retrieved content that
> carries held-out labels or a published solution is holdout access by
> another route: the score stops measuring search and starts measuring
> lookup. Technique-level retrieval (API docs, library usage, general
> methods) is fine — the test is whether the query names the task or
> benchmark rather than the technique. If task-specific artifacts
> surface unexpectedly, record it in `log.md` and surface it when
> reporting metrics; don't bury it.

No other file changes. `/lint`'s HCE checks and the skills' `respects:`
declarations already point at this rule, so the clause propagates without
further edits.

## Why (logical case)

The HCE discipline exists so a held-out number means what it claims. Its
enforcement today covers exactly one leak channel: filesystem access to
`test/` (clause 1) plus, since 2026-07-31, tampering with the evaluator
(clause 4). Nothing constrains the agent's *retrieval surface* — and a
grep of the experiment-loop skills confirms no skill mentions web access
during search either way.

For the tasks downstream projects actually run this is not hypothetical.
MLE-bench tasks are Kaggle competitions with public leaderboards, winner
write-ups, and solution kernels indexed by every search engine. An agent
that searches "kaggle <competition> winning solution" mid-`/iterate`
defeats the entire split discipline without ever touching `test/` — the
validation and test scores both inflate, and no current check notices.
The rule is soft-specified by design ("a capable model with the rule
loaded into context is the first line of defense"), but today the rule
never states that retrieval is inside the boundary, so even a fully
compliant model has nothing to comply with.

## Why (reputable evidence)

- **wang2026search** (credibility 4, code released) measures exactly this
  channel: search-enabled agents retrieve benchmark artifacts — and on
  MedQA, Gemini Deep Research retrieves the gold label for 60% of sampled
  questions. Its central refinement is that only *answer-level* leakage
  reliably inflates scores (hazard ratios 2.20–8.92), while metadata-level
  exposure does not — but escalates into answer leakage over subsequent
  turns. This is why the proposed text targets solutions/labels rather
  than banning all task-adjacent retrieval.
- **wang2026naturebench** (credibility 4, code released) is the
  construction-side attestation: its information firewall is only sound
  because web search is disabled at eval time — the authors state the
  package-level firewall cannot hold otherwise.
- **wang2026evobrowsecomp** (credibility 3) adds the temporal boundary
  and shows contamination is a decay process, reinforcing that the
  boundary must be maintained per-task rather than assumed.

Gate 1 passes on two credibility-4 code-released sources plus a third
attestation. The acknowledged gap: leakage rates are measured for
clinical and general-web QA, not ML-research tasks specifically. The
mechanism transfers on its face — Kaggle solutions are *more* thoroughly
indexed than medical exam banks — but no paper has measured it there.

## Simplicity assessment

Adds one paragraph to an existing clause of an existing rule — no new
file, no new skill, no enforcement machinery. `adds_surface_area: true`
is declared because it is net-new prose in a loaded rule, but the load is
scoped: `evaluation.md` applies only to projects with an evaluation
holdout, so literature repos (where `/digest` and `/discover` web-search
constantly, by design) never see it. The scoping already in the rule's
"When this rule applies" section resolves the apparent tension with
web-grounded curation with zero extra text. A simpler form (do nothing;
trust the model) was considered and fails because the rule as written
affirmatively excludes retrieval from its own scope — clause 1 enumerates
filesystem verbs only. A stronger form (deterministic egress filtering,
sandboxed retrieval corpora per wang2026search's prescriptions) was
considered and rejected as net-new infrastructure; the prose clause is
the HCE discipline's native enforcement style.

## Risks & what could make this wrong

- **Over-blocking.** An agent may read the clause as banning all web use
  during experiments and stop looking up library docs, hurting
  implementation quality. The technique-vs-task test in the proposed text
  is the mitigation; if it proves too coarse, the clause should be
  tightened rather than dropped.
- **Domain transfer.** If ML-research tasks somehow leak much less than
  QA benchmarks, the clause buys little — but its cost is a paragraph,
  and the asymmetry (a silently inflated final score is unrecoverable;
  a skipped doc lookup is a minor tax) favors having it.
- **Unenforceability.** Like the rest of the rule, compliance is by the
  model, with `/lint` as backstop — and `/lint` today greps file access,
  not search queries. The clause is honest about this (record-and-surface
  rather than pretend-to-block). A transcript-grep lint check could
  follow later if violations are observed; do not build it preemptively.

## Recommendation

Adopt. This is the highest-leverage gap in the HCE discipline the graph
has identified: both existing clauses guard channels (filesystem,
evaluator code) that are narrower than the one left open. The evidence
is the strongest kind this project collects short of peer review — two
independent credibility-4 measurement/construction papers with released
code, converging from opposite directions — and the fix is one paragraph
in the rule file that already owns this concern.
