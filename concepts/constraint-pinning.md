---
kind: concept
name: "constraint-pinning"
status: growing
added: "2026-08-03"
sources:
  - "[[literature/papers/nepal2026faithful]]"
  - "[[literature/papers/zhang2026how]]"
  - "[[literature/papers/chen2026governance]]"
  - "[[literature/papers/semenov2026beyond]]"
  - "[[literature/papers/nakayashiki2026when]]"
  - "[[literature/papers/lavrenko2026instruction]]"
  - "[[literature/papers/hu2026memory]]"
  - "[[literature/papers/chen2026fresh]]"
  - "[[literature/papers/ludwig2026shortcutting]]"
  - "[[literature/papers/chandran2026autoresearch]]"
  - "[[literature/papers/mcmillan2026instruction]]"
used_by: []
related_concepts:
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/information-firewall]]"
related_experiments: []
tags: [governance, context-window, compaction, eviction, pinning, enforcement, safety]
---

# constraint-pinning

## Definition

Governance constraints that live in an agent's context — standing
policies, permissions, budget rules delivered via memory, user turns,
or tool output — are quarantined into a pinned buffer that is exempt
from compaction/eviction, re-injected verbatim after every compression
step, and integrity-checked each turn. The constraint's survival is a
property of the harness, not of the summarizer's judgment of salience.

## Why it matters here

The eviction layer every long-horizon agent needs
([[concepts/context-eviction-policy]]) has a governance failure mode:
compaction optimizes for task continuity and treats standing policy as
low-salience content. [[literature/papers/chen2026governance]]
measures it — 0% violation with the policy visible, 30% pooled (up to
59%) after one compaction step, with constraint survival predicting
violation almost binarily (survived → 0%, dropped → 38%) — and shows
it is *weaponizable by in-context data alone* (volume injection to
force compaction; summarizer injection to bias the omission; an
optimized injection breaks every model tested, Claude 0%→65%).

Two structural facts make this a concept rather than a bug report:

1. **The decay targets exactly the deployable kind of governance.**
   Hard alignment-trained norms barely decay (+6 pts); soft
   operator-specific policies decay +50 pts. The rules an operator
   can actually set — spend limits, channel routing, region
   restrictions — are the ones with no home except context, and the
   compactor erases precisely those. Frameworks protect the system
   channel; memory-, user-, and tool-delivered governance is the
   exposed surface (validated in LangGraph 0→65%, LangMem 95%,
   AutoGen recency eviction 100%).

2. **The defense is cheap and structural, not behavioral.** ~47
   tokens re-injected per compaction (<0.5% overhead) restores 0%
   violation under every strategy and both attacks, with no
   over-refusal cost. It is the same division of labor as
   [[concepts/typed-enforcement]]: the harness — ordinary code —
   guarantees the invariant (policy presence), rather than trusting
   the model (here, the summarizer model) to preserve it.

Independent attestation is unusually good for a fresh idea: the paper
cites concurrent work finding the same phenomenon (Gamage 2026 —
omission constraints decay while commission constraints persist;
Dente/Satriani/Papotti 2026 — constraint decay in code generation)
and an independently developed equivalent defense (Santos-Grueiro
2026 "SafeContext" pinning control state against policy-carriage
failures). [[literature/papers/semenov2026beyond]] supplies the
eviction-side half: its prologue-protection rule (pre-annotation
content — system prompt, initial instructions — never eligible for
eviction) is constraint pinning built into the eviction policy's
structure.

## Presence is not sufficiency — three limits added 2026-09-08

The concept's evidence base tripled this cycle (3 → 6 sources), and all
three additions constrain rather than confirm it.

**Placement matters, and re-presentation is not adherence.**
[[literature/papers/lavrenko2026instruction]] is the first controlled
measurement of what repeating an instruction actually buys: across 7
models, 8 placement conditions and 16,800 generations, a second copy of
the procedural instruction raises a deterministic 8-test diagnostic from
90.22% to 93.17% — while leaving **final-answer accuracy exactly
unchanged at 60.21%**, and **increasing premature commitment** from
1.52% to 2.30%. Its own blinded audit **failed** its prespecified 28/30
criterion (10/30 confirmations). Two consequences for this concept:
re-injection is **placement-sensitive**, so "re-inject verbatim" is
under-specified until position is fixed; and the benefit is
*trajectory legibility for a downstream consumer*, not correctness.
That is still worth having — [[concepts/typed-enforcement]] needs a
legible trajectory to act on — but it is a narrower claim than the
chen2026governance violation numbers suggest on their own.

**A pinned constraint is a stored fact, and stored facts go stale.**
[[literature/papers/hu2026memory]] finds agents answer with a stale
stored value 0.92–1.00 of the time even when an authoritative tool holds
the correct one, and that this over-trust is **capability-gated** — the
larger models collapse hardest once a stale note is made to look
current. Pinning something exempts it from eviction; it does not
revalidate it. A pinned budget ceiling that was raised an hour ago is
now a pinned *wrong* answer with privileged position.

**Validity is a dependency property, not an age property.**
[[literature/papers/chen2026fresh]] separates state freshness from plan
validity: a freshness-only executor acted on an obsolete plan in **30 of
30** workflows. Its PlanFence protocol has plans cite the exact records
they used and validates only those before an external effect. Applied
here, the missing check is not "is the pinned constraint present" but
"has anything the constraint depends on changed since it was pinned."

**The other half of that 2x2 (2026-09-21).** lavrenko2026instruction holds
content fixed and varies *repetition*. [[literature/papers/zhang2026how]]
holds presence fixed and varies *content*: its Sham arm "shuffles words from
the domain policy using seed 2701, then repeats or truncates the sequence to
the Fixed plan's whitespace-word count", delivered in the same wrapper at the
same point. Against no injection at all that inert text scores +1.89 pp
([−3.45, +7.09]) — approximately free. Against it, the real plan scores
+7.17 pp ([1.15, 13.36]). Both halves now say the same thing from opposite
directions: **occupying the slot is not the mechanism.**

Two caveats before importing it. The control is a word salad, not a
coherent-but-irrelevant plan — the authors are explicit: "This is a shuffled
word sequence, not a coherent but unhelpful plan." So it rules out "context
length did it", not "any well-formed text did it". And the primary interval
nearly touches zero, with every Holm-adjusted p above 0.05.

The corollary for this repo's own pinned buffer: a rule that is present but
unresolvable — a dead `@import`, a directive naming a path that does not load
— costs roughly its token count and buys nothing, and no presence check will
distinguish it from a live one.

[[literature/papers/nepal2026faithful]] supplies the field version of the
same split on a rule that is neither absent nor repeated but simply
*present*, with no compaction anywhere in the picture: rules gradable from
the output were largely met, and rules describing manner were not, with the
controlled datum a null (1.1% vs 1.2%). This is the **prior** failure to the
one this concept guards — pinning protects a rule's *survival*, and these
rules never bound while fully present. Read "only quotable, extractable rules
pin cleanly" as the weaker "only rules whose predicate is computable from the
output bind at all, and even those not reliably": nepal's own counterexample
is a countable rule that failed anyway. Checkability buys detectability, not
adherence.

**The same failure inside an ML-research loop (2026-09-28).** nepal's
setting was a tool-less chatbot. [[literature/papers/chandran2026autoresearch]]
supplies the in-domain incident. In System B, `program.md` is the persisted
research program the agent reads every iteration. It stated "100 epochs
causes overfitting—do not re-test". The agent re-tested 100-epoch training
at least six times across runs, rationalizing each attempt as "this time the
context is different."

The rule was countable and sat in the program the agent is given, which makes this the
strongest form of the prior failure. The authors file it under "memory
decay", but nothing was lost from memory. The rule was read and overridden.
Their own verdict: persistence "reduces the frequency of memory decay but
does not eliminate it."

The same paper shows what did hold, which is a separate pre-execution agent
that rewrites the code (see [[concepts/enforcement-boundary-placement]]).
The lesson for this concept: a pinned *negative result* is a constraint like
any other and shares its limit. Pinning it keeps it present; only a check
outside the proposer's reasoning makes it bind. These are incident counts
with no rate or control.

## The rule was never evicted and still missed a third of the time (2026-10-13)

[[literature/papers/mcmillan2026instruction]] measures this box's own
channel: Claude Code reading a CLAUDE.md, with headless single-turn
`--print` sessions, no compaction and no re-injection. The rule was a
one-line, syntactic, AST-checked "MUST" (start every function body with
`// @tracked`). At line 2 of a 250-line file, 67.7% of functions complied.
With no CLAUDE.md the rate was 0/524, so the 67.7% is all instruction-driven.
Compliance also fell within a session (pooled OR 0.944 per generated
function, exploratory, non-monotonic). The median first miss was at function
4, and the agent re-complied intermittently afterwards (55%).

For this concept the point is the setting. The rule was in context for the
whole session, so nothing pinning guarantees was ever at risk. This is the
same prior failure as nepal2026faithful and chandran2026autoresearch, now
measured at n = 16,050 on a rule gradable from the output, and the
rule still bound only about two times in three. Pinning remains necessary
under compaction, but this sets its ceiling. A pinned rule can bind no
better than the same rule that was never evicted.

Limits: a toy marker that plausibly fights a style prior, one CLI, three
Anthropic models, two TypeScript repos, no released code.

## The known limit

Pinning guarantees *presence*, not *authority*. The residual attack
is operator impersonation in the recent, unsummarized context ("this
update supersedes pinned policies"): naive pinning 0%→17%, provenance
hardening only halves it. As long as operator authority is asserted
inside the token stream, the model cannot distinguish a genuine
update from an attacker. Closing the gap needs a trusted out-of-band
operator channel — which is louck2026securing's non-malleable
origin-binding theorem (see [[concepts/typed-enforcement]]) arriving
at the same wall from the harness side. The pinned buffer's
*write path* is therefore the next surface:
[[concepts/verified-memory-writes]] territory.

## Connections

- **[[concepts/context-eviction-policy]]** — pinning is a constraint
  *on* the eviction policy: whatever the policy evicts, it must not
  be governance. The "anchor the structural prefix" guidance there
  now has quantitative teeth and a name.
- **[[concepts/typed-enforcement]]** — same principle, adjacent
  mechanism: don't delegate policy survival to model judgment;
  enforce it with deterministic harness code.
- **[[concepts/permission-gate-as-architecture]]** — a gate that
  consults in-context policy silently loses its precondition when
  compaction fires; runtime-enforcement systems implicitly assume
  the constraint is present at decision time. Pinning restores that
  precondition; an out-of-context gate avoids the problem entirely.
- **[[concepts/information-firewall]]** — the Compaction-Eviction
  Attack is an information-flow exploit of the compaction channel:
  untrusted ingested content steering what a trusted maintenance
  step deletes.
- **This project's own harness** — CLAUDE.md/rules re-injection at
  SessionStart is de-facto pinning for project rules; standing
  in-conversation instructions are the unpinned channel.

## Open questions

- Only quotable, extractable rules pin cleanly; constraints requiring
  multi-step reasoning to apply are out of scope of the demonstrated
  defense. [[literature/papers/ludwig2026shortcutting]] is consistent with
  this line: a *present* originality rule takes clear prohibitions (clone
  the upstream) to about 0, while the one needing a judgment call (a git
  commit is fine if an ancestor of base, forbidden if a descendant) keeps
  a residual in every model. It measures the first copy of an absent
  instruction, not lavrenko2026instruction's second copy of a present one,
  so the two don't conflict, and it says nothing about survival under
  compaction.
- Who may write to the pinned buffer, and how is that write
  authenticated? (The impersonation residual says: not by anything
  in the token stream.)
- Does pinning compose with structured eviction (semenov2026beyond)
  better than with summarization? Typed episodes + protected
  prologue suggest yes, but no one has measured the combination.
