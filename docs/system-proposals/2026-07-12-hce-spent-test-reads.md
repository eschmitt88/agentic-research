---
kind: system-proposal
slug: hce-spent-test-reads
added: "2026-07-12"
target: "claude/rules/evaluation.md"
change_type: edit
adds_surface_area: false
evidence_citekeys: [ning2026closedloop, bertran2026fits, hambardzumyan2026aira, chan2024mle]
evidence_strength: "3+ independent attestations incl. peer-reviewed + code anchors (chan2024mle) and an explicit score-once certification protocol (ning2026closedloop); credibility ~4 on balance"
status: proposed
recommendation: adopt
---

# `evaluation.md` should say a test read is *spent* — the final pass runs once, not until the number looks good

## The change

Two small additions to `~/claude-system/claude/rules/evaluation.md`. No
new file, no new knob, no new hook — a tightening of the existing rule.

**Clause 1** currently ends its final-pass paragraph:

> The only permitted access is a **final-scoring pass**, invoked
> explicitly at chain end (not inside `/iterate --chain` cycles), whose
> single job is to run the held-out evaluation and write
> `final_metrics.json`.

Append one sentence:

> That pass runs **once per chain** — never re-run it after further
> changes "to double-check," and never because the first number
> disappointed; a repeated final pass is search-phase access wearing a
> final-pass costume.

**Clause 2** currently ends:

> - `final_metrics.json` — **held-out test-split** metrics. Written
>   only by the final-scoring pass. Nothing inside the search loop
>   reads it.

Append:

> A test score, once read, is **spent**: whoever plans the next chain
> has seen it, so any later chain on the same task is partially
> *selected on the holdout*, and its own final score is no longer an
> unbiased estimate. Keep final-scoring passes rare and deliberate.
> When a task has accumulated several across chains, say so wherever
> the held-out number is reported — it is exploratory at that point,
> not confirmatory.

## Why (logical case)

The rule as written closes the *within-chain* channel (the search loop
never reads `test/`) but leaves the *between-chains* channel wide open.
Nothing prevents this sequence, and each step is currently compliant:
chain ends → final pass writes `final_metrics.json` → the number is
disappointing → a new chain starts on the same task, its direction chosen
*because* of that number → another "chain end" → another final pass.
Iterate that a few times and the test split has become a slow-motion
validation split; the singular phrasing "a final-scoring pass" implies
once, but implication is exactly what a many-cycle autonomous loop erodes.
This is the same failure mode clause 1 exists to prevent, operating one
level up — per-chain instead of per-iteration — and it is *more* likely
here than in human research because `agency: max` repos chain autonomously
and the "chain end" trigger recurs.

The fix is two sentences that make the implicit explicit. The rule's own
design philosophy ("a capable model with the rule loaded into context is
the first line of defense") means an unstated norm is a hole: the model
honors what the rule says, and the rule currently doesn't say this.

## Why (reputable evidence)

- **ning2026closedloop** (credibility 3) — the direct anchor. Its
  certify-after-search protocol is explicitly *score-once*: "freeze the
  validation-selected config, retrain from scratch, **score once** on the
  held-out test partition." Its empirical core is that validation gains
  routinely fail to transfer (0.041 val → 0.003 test; 0.022 val → −0.019
  test), which is precisely why a re-run-until-happy final pass would
  select noise. Its own critique section flags that even honest one-shot
  reads accumulate multiple-comparisons exposure as endpoints multiply —
  the exact quantity the proposed text asks to be disclosed.
- **bertran2026fits** (credibility 3) — the formal frame. From the
  adaptive-data-analysis lineage (Roth co-authored the original
  reusable-holdout work): generalization guarantees degrade with the
  number of bits extracted from a holdout, which is why its mechanisms
  bound the feedback channel (one-bit ladder; 32-token reproducer). Each
  additional final-pass read is additional extracted bits. The paper also
  attests the enforcement philosophy: bottlenecks belong in the harness
  and its rules, not in good intentions.
- **hambardzumyan2026aira** (AIRA_2, cred 4, `hce-evaluation`'s founding
  anchor) — ablations show the val/test gap diverges when a loop optimizes
  any signal it can read. Between-chains test reuse hands the loop exactly
  such a signal at chain granularity.
- **chan2024mle** (peer-reviewed, code released, credibility 5) — the
  design precedent: MLE-bench's hidden test set is graded on submission,
  a structurally one-shot read. The concept `hce-evaluation` is `active`
  with 18 sources; this edit imports nothing the concept's evidence base
  doesn't already assert.

Gate 1 passes on the 3+-attestations arm with a peer-reviewed + code
anchor in the set and credibility ≥3 on balance.

## Simplicity assessment

`adds_surface_area: false`. Two sentences appended to an existing rule
file; nothing new to maintain, no config, no hook, no counter. A stronger
form was considered and rejected: a `test_reads: N` frontmatter field
per task that `/lint` could count and threshold. That adds a schema
field, a lint check, and a bookkeeping obligation for a quantity that is
currently near-zero on every project this box runs — machinery ahead of
need. If between-chains reuse ever becomes common enough to need
counting, that will be visible (decision records, README notes) and a
follow-up proposal can add the counter with evidence of need. The prose
form is the simplest viable form, and it strengthens rather than loosens
the holdout, as required for any proposal touching `evaluation.md`.

## Risks & what could make this wrong

- **It could over-chill legitimate re-scoring.** Some re-runs are honest:
  a bug in the scoring script, a corrupted artifact. The proposed text
  says "never re-run after further changes" — a scoring-bug fix *is* a
  change, though not to the model. If the reviewer wants, add "(re-running
  because the scoring harness itself was broken is fine; record it)" — at
  the cost of a longer rule. I'd omit it; the model can distinguish a
  broken scorer from a disappointing number, and the decision-record
  habit covers the edge.
- **"Spent" is not literally true at N=2.** One extra read costs little
  in bits; the text's force is directional, not a hard threshold. That is
  deliberate (matching the rule's soft-specified style) but a reviewer
  who wants a hard rule ("max 3 final passes per task, then freeze")
  should note the evidence doesn't pin any particular number.
- **Both new anchors are non-peer-reviewed preprints.** The score-once
  protocol specifically rests on ning2026closedloop (cred 3). The
  mitigation is that the *principle* (holdout reuse degrades guarantees)
  is textbook adaptive data analysis and independently attested by the
  concept's peer-reviewed anchors; the preprints supply the agentic-loop
  instantiation, not the statistics.

## Recommendation

**Adopt.** The edit is two sentences, removes an ambiguity in a rule
whose entire mechanism is "say the norm explicitly so the model honors
it," strengthens the holdout, and is backed by the graph's
best-attested concept plus two fresh cross-domain anchors that name this
exact channel. The cost of being wrong is a slightly stricter norm than
strictly necessary; the cost of the status quo is a compliant path to
adaptive test-set reuse in autonomous chains.
