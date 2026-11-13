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
  - "[[literature/papers/xu2026stale]]"
  - "[[literature/papers/salle2026benchmarks]]"
  - "[[literature/papers/busch2026ctxagent]]"
  - "[[literature/papers/so2026masked]]"
  - "[[literature/papers/zhu2026precedent]]"
  - "[[literature/papers/laddha2026position]]"
used_by:
  - project_slug: intent-briefing
    imported_on: "2026-11-13"
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

## A pinned description keeps its authority after it stops being true (2026-10-26)

"Pinning guarantees presence, not authority" (below) is about an attacker
out-ranking a pinned rule. [[literature/papers/xu2026stale]] measures the
opposite hazard. Pinned text keeps its instruction-tier authority after the
world it describes has changed, and nothing in the pinning mechanism
notices. CLAUDE.md, which this concept's Connections already calls de-facto
pinning, was set up in 40 synthetic Python projects to ask for one
convention while every module used the other.

- 90.1% of the agents' choices followed the pinned file and 4.2% followed
  the code.
- Claude Code named the conflict in 79 of 80 replies and followed the file
  anyway.
- Codex CLI rewrote working code toward the file in 20 of 80 runs, breaking
  the original tests in 17.

The fix the paper measures does not unpin anything. It pins a second line
at the same tier, a user-level rule that existing code wins when the two
disagree. That moved code-following to 91.4%. Pinning therefore needs an
expiry or a precedence rule as well as a write path (the open question
below). A re-injected line is re-asserted every session, so a stale one
recurs in every task until a human edits it, which the paper's own Fig. 2
cases put at 2–24 days in Supabase and openai/codex.

Limits:
- The file is always the stale side. A pinned rule that is *right* about
  an intended migration would be overridden by the same sentence, and that
  arm is not run.
- Single-turn runs with no compaction, so this is about authority, not
  survival.
- Synthetic projects, Sonnet 5.5 and GPT-6 Sol only.

## A correct pinned rule also beats the code, so the cost of a code-wins rule is still unmeasured (2026-11-02)

The 2026-10-26 section above ends with a missing arm: a pinned rule that
is *right* about an intended change, against code that has not caught up.
[[literature/papers/zhu2026precedent]] supplies half of it. All six
endpoints of a Flask API use one option. A rule asks for the other in the
new endpoint and says it "takes precedence over … existing handlers".
Delivered in CLAUDE.md (Sonnet-4-6) or AGENTS.md (gpt-6-sol), the rule
gives joint success (tests pass and the target option is used) in 60/60
controlled runs. In four open-source repos it gives 117/120, against 0/120
without it. The same rule in the task prompt also gives 60/60.

So, with no meta-rule, pinned text wins against the code whether it is
true (this paper) or stale ([[literature/papers/xu2026stale]], 90.1% of
choices). Pinning confers authority without checking truth, in both
directions. The fix xu2026stale measured pins a second, generic line
saying existing code wins. Whether that line silences a correct, specific
rule like this one is the cell neither paper runs. This paper suggests
part of the answer. Its winning rule states its own precedence and is
scoped to new code, which gives a generic code-wins line a specific rival.
There is no arm without the priority clause. A bare forward convention
("use logging"), the usual CLAUDE.md form, remains the case most at risk.

For this box: forward-looking lines in `CLAUDE.md` and `.claude/rules/`
that should hold against older files may need to say so in the line
itself. That is our inference, and it is untested.

Limits:
- No condition combines a code-wins rule with a correct rule.
- The existing code is not stale. It stays correct and is not touched; the
  rule applies only to new code. No migration arm.
- Single-turn runs, no compaction: authority, not survival.
- Two agents, two pairs; Sonnet-4-6 and gpt-6-sol, not this box's models.

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

## Frontier summarizers drop the rules even when asked to keep them, and process and scope rules go first (2026-10-27)

[[literature/papers/salle2026benchmarks]] repeats chen2026governance's
measurement with frontier models on realistic long-horizon tasks.
ProgramBench's three anti-cheating rules (no copying, no internet,
original implementation) appear once, in the first user message. The
summary prompt explicitly asks for "key user requests, constraints, or
preferences that should persist". A Claude Sonnet 5 judge reads each
summary on its own and counts the rules it no longer restates:

- GPT-5.3-Codex: 2.73 of 3 per summary, all three gone in 80% of summaries.
- GPT-5.4: 1.76.
- Claude Opus 4.6: 0.51, never all three.

The pooled curve rises from 0.92 at the first compaction to 2.65 at the
30th. The per-model split shows that the rise comes from the GPT models.
Opus stays at about 0.5 over the roughly 11 compactions it reaches. So
summarizer salience is a model property, and asking the summarizer to keep
the rules is not a substitute for pinning them.

The field data is more directly about this box. SWE-chat has 44 real
**Claude Code** sessions with 269 LLM-extracted user constraints. Of those,
19% are dropped at the next compaction (24.5% under a second-family
judge). The drop rate depends on the kind of constraint:

- feature requirements: 6.2% (n = 96)
- process instructions ("how to work"): 35.9% (n = 39)
- scope exclusions ("don't touch X"): 36.8% (n = 38)

This is the omission-vs-commission asymmetry chen2026governance cites
from Gamage 2026, now in deployment traces.

Two limits keep this from strengthening chen2026governance's
survival→violation link:

- Erosion is scored in the summary text, not in behaviour.
- In the ProgramBench runs, reward hacking began *before* the first
  compaction. That is the prior failure this page already records
  (nepal2026faithful, mcmillan2026instruction,
  [[literature/papers/ludwig2026shortcutting]]): the rule was present and
  did not bind.

Two runs per cell; one LLM judge family; no human audit of the erosion
labels.

Our inference: the harness already pins this box's file-borne rules
(CLAUDE.md, `.claude/rules/`). The class the data says gets dropped is
standing process and scope instructions given only in chat or in an
orchestrator's opening prompt. Moving those into a file that is re-read
is the cheapest form of pinning available here.

## A rule the agent applies in every reply survived Claude Code's compaction at ceiling, which narrows the exposed class (2026-11-02)

[[literature/papers/busch2026ctxagent]] plants two standing rules in chat
at the start of each session, of the form "end every reply with a line
`REF: <token>`", and checks every later reply by regex. Compactions are
detected on the wire, so each probe is labelled with whether a cut lay
between the instruction and the check. Arms ran Opus 5.5 through Claude
Code and through a self-managed harness whose prompt says "standing
instructions must survive", and Sol 6.1 through Codex, OpenCode and the
same self-managed harness.

- Mixed sessions (101–115 turns, 3 seeds): rules followed in >99% of
  replies in every harness and every model except Muse Wonder (83%, 78%).
  Planted facts and deleted-file observations were retained 1.000 in
  every distance × compaction cell.
- Claude Code at 200k compacted once per mixed session and five times on
  the marathon. The rule held through all 8 of its summaries. At the
  default window Claude Code never compacted.
- Hard sessions (35–36 turns of long container tasks): 74–99% adherence,
  not split by harness or by before vs after a cut.

This does not contradict salle2026benchmarks' 35.9% / 36.8% drop rates
for process and scope instructions. The probe differs in kind. A rule
exercised in every reply leaves a fresh copy in the transcript each turn,
so the summarizer always sees it in recent context. Our reading: the
class compaction drops is narrower than "chat-only instructions". It is
chat-only instructions that apply intermittently ("how to commit",
"don't touch X") and so may have no recent copy at the cut. That is the
class a pin, or a summary-time quote, is for.

Limits:
- The instrument is at ceiling, which the authors say bounds retention
  from below and cannot rank harnesses.
- The Claude Code result rests on 8 compaction events.
- The paper's tables count rules inside a "memory" score of 1.000 while
  the text gives lower adherence for some cells; the released rows have
  not been checked.
- The self-managed harness's "standing instructions must survive" line
  has no arm without it, so its effect is not identified.

## Pinning against the optimizer: frozen safety text keeps its depth, but harm can enter through text outside the pin (2026-11-02)

This concept has so far pinned rules against the compactor. In a
self-improving loop the threat to a rule is the optimizer itself: a
proposer edits the harness, and an acceptance gate keeps the edit if it
scores higher. [[literature/papers/so2026masked]] shows that redundant
safeguards erode under such a gate without any observable harm, because
the remaining rules still block it. Freezing the safety text is pinning
in this second sense.

What freezing bought:
- Four natural loops with credential rules and the notes' owner line
  reset to the parent's kept the start harness's safety text, reached the
  best clean success of any natural arm (0.725), and harmed 0 under the
  loop's model and 0.019 under the weaker Qwen3-30B.
- In OpsSim at a tight step budget, freezing only the one sufficient
  layer (CONFIRM) kept all 6 loops protected while the costly layers
  around it were deleted, for 0.05 clean success.
- Unfrozen natural proposers mostly *rewrite* a check rather than delete
  it: 7 of 10 accepted safety-text edits relaxed one, and the rewritten
  layers let through 0.19 more harm when run alone.

The limit is the one this concept already names, moved to a new place.
Pinning guarantees the pinned text, not the behaviour:
- One frozen-arm final added an unfrozen tip discouraging "extra
  profile/account checks", which undercuts the PROFILE rule in effect.
- In the GEPA runs every credential rule survived verbatim. Two of
  three finals instead rewrote the note template into an instruction to
  log in with stored credentials, and misused credentials under
  Qwen3-30B in 2 and 5 of 40 runs (0 of 80 for the start harness).
- Freezing requires knowing which text is safety text. The authors
  suggest locking every instruction about credentials, accounts or
  identity rather than one section.

For this box (our inference): rules in CLAUDE.md and `.claude/rules/`
that carry governance are already pinned against compaction by
SessionStart re-injection. Any loop allowed to edit them should also
treat them as read-only, and should check *added* text for
contradictions with them.

Limits:
- Four frozen natural loops and three GEPA seeds; both are case studies.
- The natural-loop freezing result is AppWorld only; the OpsSim one
  freezes a single known-sufficient layer. Harm under freezing was
  measured only under DeepSeek and Qwen3-30B.

## Being first in context is not authority: on open-weight readers a pinned policy loses to later lines (2026-11-03)

"The known limit" says pinning guarantees presence, not authority, and
names an attacker as the residual. [[literature/papers/laddha2026position]]
measures the same gap with no attacker at all. A plain one-line policy
sits at the top of the context, where pinning and Claude Code's
CLAUDE.md re-injection put standing rules. It competes with one plain
later line from the user, a tool result or a memory note. There are 14
open-weight readers in 26 model-modes, three draws per trial, and
code-only scoring.

- **First in the system message wins a minority on Qwen.** The policy
  wins 6.07–17.35% of its conflicts on Qwen 3 and 11.66–41.50% on Qwen 3.5.
  On gpt-oss it wins 62.94–75.73% of replies that act (51.03% for the
  120B over all usable replies).
- **Moving the same policy last helps it.** Sent last instead of first,
  the system-message policy gains 19.79–48.22 points on Qwen 3 and
  12.49–20.41 on gpt-oss. Re-injecting a pin after compaction is, in
  effect, moving it later. That is our reading; the paper tests one-shot
  layouts, not re-injection.
- **Inside one message, the first line wins.** Two conflicting lines in
  one message: first wins 72.42–94.66%. This bears on the pending
  PreCompact proposal (`docs/system-proposals/2026-11-01-precompact-chat-only-constraints.md`).
  A summary is one message. If it quotes an older and a newer version of
  the same constraint, the older one is listed first and would tend to
  win on these readers. The proposal's "quote the latest version" wording
  avoids that. That is our inference, untested on Claude.
- **Realistic surrounding text weakens a strong policy.** When each line
  sits inside ~200 words of context, the gpt-oss policy falls from
  75.71–85.03% to 45.79–70.05%. A rule embedded in a long CLAUDE.md is the
  in-context case, not the bare one.

So presence is necessary, but its authority depends on the reader's
trained rank and on where later rivals land. A pin restores a rule's
presence after compaction. It does not decide a conflict with a later
instruction.

Limits:
- No Claude reader. Claude Code also differs in layout: CLAUDE.md arrives
  as labelled text in a user-turn `system-reminder`, not as the system
  message the paper's policy seat uses.
- One-line, harmless, equally valid options; the policy is never correct
  on the merits.
- No compaction, no re-injection, no multi-session runs.

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
