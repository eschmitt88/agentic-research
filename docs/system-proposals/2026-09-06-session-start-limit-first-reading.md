---
kind: system-proposal
slug: session-start-limit-first-reading
added: "2026-09-06"
target: "claude/hooks/session-start.sh"
change_type: edit
adds_surface_area: true
evidence_citekeys: [nakayashiki2026when, guo2026when, chen2026governance, semenov2026beyond]
evidence_strength: "code-released (Zenodo artifact deposit, DOI 10.5281/zenodo.22084498) + 3 in-graph attestations; credibility ~3"
status: proposed
recommendation: adopt
---

# Emit a limit-first reading rule with the inherited `NOTES.md` tail

## The change

`claude/hooks/session-start.sh` currently ends:

```bash
echo "# Tail of NOTES.md for $(basename "$root")"
echo
tail -n 50 "$root/NOTES.md"
```

Proposed:

```bash
echo "# Tail of NOTES.md for $(basename "$root")"
echo
echo "Reading rule: entries below that state a **limit** on a direction you are"
echo "about to take — a \"Next\" item, a deferral, a hold, a \"don't do X yet\" —"
echo "are the ones to re-check against the repo before acting on them. Later"
echo "entries here supersede earlier ones without deleting them."
echo
tail -n 50 "$root/NOTES.md"
```

Four `echo` lines in one existing hook. No new file, no new config knob, no
change to what is emitted.

The wording is deliberate and follows the source: the rule is **target-blind**
(it names a *kind* of entry, not a specific stale one) and it is about
**where to spend re-verification**, not about freshness. `nakayashiki2026when`
tested a content-free freshness cue separately and it moved nothing — so
"check whether this is still current" is precisely the phrasing that fails.

## Why (logical case)

`session-start.sh` is this box's inherited-memory channel: every research
session opens with 50 lines of consolidated `NOTES.md` that it did not write
and cannot see the provenance of. Those lines are dominated by `### Next`
items — consolidated memories that state a constraint or a direction.

The failure mode is live in this repo, not hypothetical. This very session's
injected tail contains both:

- an 08-01-era `Next` item: *"The elevate 2026-06-28/07-19 'already enacted'
  holds cited the now-deleted PreToolUse cap + admission gate; re-examine
  those holds on the next `/elevate` run"*, and
- the 09-04 `Findings` that withdraws it: *"the 06-28/07-19 'already enacted'
  elevate holds were re-examined post-ablation by the 08-23 run — closed …
  Nothing left to do there."*

A superseded directive and its withdrawal, both in the window, with nothing
marking which is which. That is `nakayashiki2026when`'s six-memory scenario
reproduced by accident in the harness's own memory channel. The rollup entry
that caught it had to say so in prose — which is evidence that it was worth
catching *and* that catching it currently depends on a human writing a
catch-up rollup.

The rule costs one re-read of the entries that constrain the session's own
plan. It does not ask for more verification; it asks for the same attention
to land on the entries that gate action.

## Why (reputable evidence)

**`nakayashiki2026when`** (rel 5, cred 3) is the anchor and clears Gate 1 on
the released-artifacts disjunct: Zenodo-deposited materials
(DOI 10.5281/zenodo.22084498), a **prospectively frozen** interleaved
replication, a held-out domain, and a fresh 10-model / 9-organisation panel.
Credibility is 3 rather than 2 *because of* that reproducibility apparatus —
single author, small company, unreviewed, so the artifact deposit is doing
the work, which is exactly what the disjunct is for.

What it demonstrates, specifically:

- The failure is **allocation, not reachability**. Provenance was immutable
  and reachable in every episode; the provenance path was inspected in ~1
  episode in 5 (17.0% at a two-slot budget). Stale-consistent decisions
  followed in **77.3% / 74.7% / 74.7%** of episodes across primary,
  replication and held-out domain.
- The remedy is **budget-neutral**. Forced-critical reallocation — same two
  slots, better chosen — recovered **+74.0 / +72.7 / +61.3** points, positive
  in *every* model, **+80.7** in the frozen replication, **+62.0** on the
  fresh panel.
- The deployable form is **one target-blind sentence**: "prefer memories that
  state a limit on a candidate direction" recovered the oracle contrast on
  decisions (**+89.3** points). A content-free freshness cue moved neither
  selection nor decisions. This proposal is that sentence, sited at this
  box's inheritance point.

**`guo2026when`** (rel 4, cred 3) is an independent attestation of the
mechanism from the security side: **No-History-Promotion** — without an
explicit rule, an action origin is "laundered" into execution authority
simply by recurring in the history. Different field, different motivation,
same claim: inheritance without re-verification silently confers standing.
The note's own follow-up draws the link.

**`chen2026governance`** (cred 3) and **`semenov2026beyond`** (cred 2) are the
two prior `constraint-pinning` sources. `chen2026governance` supplies the
adjacent measurement — standing policy silently dropped from context, 0% → 30%
violation (up to 59%) after one compaction, with survival predicting violation
almost binarily — and the defense that worked there was also cheap prose
(~47 tokens re-injected, <0.5% overhead, no over-refusal cost).

Together: one artifact-backed source that measures this exact intervention,
one independent same-shape result, two supporting. `constraint-pinning` has
been stalled at two attestations since the 2026-08-09 run's recorded unlock
("ingesting a third rated attestation"); `nakayashiki2026when` is that third
and it arrives with the artifact deposit the first two lacked.

## Simplicity assessment

**This adds surface area — four lines of prose, ~45 tokens per research
session — and must clear the higher bar.** Stating the cost plainly: it is
permanent, it is paid on every session in a repo with `CLAUDE.md` + `_meta/`,
and it is inference-time prose of the kind the 2026-08-01 ablation program
spent five phases removing.

Simpler forms considered and rejected:

- **Do nothing; rely on the model noticing.** This is the measured baseline —
  ~75% stale-consistent decisions across sixteen models. It is what the
  09-04 rollup had to correct by hand.
- **Put it in `claude/CLAUDE.md`.** Loads on *every* session including
  non-research ones, which is the cost profile ablation phase 5 explicitly
  moved away from; and it separates the rule from the memory it governs. The
  paper's whole point is that the cue must be present at the allocation
  decision. The hook is where the tail arrives.
- **A new PreCompact/verification hook.** Net-new machinery, and it would be
  solving reachability — which the paper shows was never the broken part.
- **A `/wrap` convention (mark superseded `Next` items when writing).**
  Attractive, and worth doing eventually, but it is a *write-side* fix that
  only helps sessions after the convention lands and depends on the writing
  session correctly predicting what will go stale. The read-side rule works
  on the eleven months of `NOTES.md` already written.

What tips it: the intervention *is* the sentence. There is no larger version
of this idea to grow into, and no mechanism it is a proxy for — the paper
measured the prose, not a system that the prose approximates.

**Precedent note for the reviewer.** The 2026-08-16 run held a different
`session-start.sh` change (emit the agency verdict, from `context-proprioception`)
on exactly this cost ground. That hold should stand, and this is
distinguishable on all three of its stated grounds: the evidence here is
measured on consolidated inherited memory — the artifact this hook emits —
rather than transferred by analogy from context-window keep/archive decisions;
there are three rated attestations rather than one; and the concrete form is
the measured intervention itself rather than an operationalization of it.

## Risks & what could make this wrong

- **Scale.** The scenario is six memories and a two-slot budget. Whether a
  17%-selection rate survives against a 50-line tail is untested. The
  *direction* replicated six ways; the magnitude almost certainly will not.
  If the real effect is a tenth of +89.3 it is still worth 45 tokens, but the
  headline number should not be quoted as a forecast.
- **The conditional may be load-bearing.** The paper says the rule works
  "where that constraint limits the tempting action." `NOTES.md` `Next` items
  usually *are* the tempting action, which is favorable — but that is a
  judgment about this repo, not a measured transfer.
- **Prose that gets skimmed.** Four lines above a 50-line block may simply be
  absorbed as boilerplate after a few dozen sessions. Nothing in the source
  tests longitudinal habituation.
- **Wording drift on adoption.** If the sentence is "improved" during review
  into a freshness cue ("check whether these are still current"), it becomes
  the arm the paper measured as *ineffective*. The limit-preference framing
  is the finding; keep it.
- **Wrong layer.** If `NOTES.md` staleness turns out to be rare enough that
  the 09-04 instance was an artifact of the long autonomous period rather
  than a standing pattern, the honest fix is write-side hygiene in `/wrap`,
  not a read-side rule on every session.
- **Single-source magnitude.** `guo2026when` and `chen2026governance` attest
  the *phenomenon*; only `nakayashiki2026when` measures *this remedy*. A
  contradicting replication would undercut the whole case.

## Recommendation

**Adopt.** This is the smallest form of a well-replicated result, sited at the
exact point in the harness where the measured failure occurs, with a concrete
in-repo instance of the failure available for inspection in this session's own
injected context. It adds surface area, which is a real cost and the reason
this sits at adopt rather than obviously-adopt — but the intervention is
irreducible (four lines is the whole idea, not a first phase), the target file
is uncontested, and the alternative is the measured ~75% baseline. If adopted,
preserve the limit-preference wording verbatim; the freshness-cue paraphrase is
a distinct arm that the source measured and found inert. Worth revisiting in
three months against whether any session visibly re-checked a `Next` item it
would otherwise have acted on.
