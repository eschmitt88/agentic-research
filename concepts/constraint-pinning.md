---
kind: concept
name: "constraint-pinning"
status: seedling
added: "2026-08-03"
sources:
  - "[[literature/papers/chen2026governance]]"
  - "[[literature/papers/semenov2026beyond]]"
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
  defense.
- Who may write to the pinned buffer, and how is that write
  authenticated? (The impersonation residual says: not by anything
  in the token stream.)
- Does pinning compose with structured eviction (semenov2026beyond)
  better than with summarization? Typed episodes + protected
  prologue suggest yes, but no one has measured the combination.
