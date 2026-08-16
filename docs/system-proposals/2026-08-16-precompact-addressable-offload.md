---
kind: system-proposal
slug: precompact-addressable-offload
added: "2026-08-16"
target: "claude/hooks/pre-compact.sh"
change_type: edit
adds_surface_area: false
evidence_citekeys: [li2026acm, xu2026llm, dang2026addressable, hao2026selfgc, semenov2026beyond]
evidence_strength: "code-released (li2026acm, xu2026llm); 5 independent attestations; credibility ~3.2 (4/4/3/3/2)"
status: proposed
recommendation: adopt
---

# PreCompact should hand back an address, not just an instruction to write

## The change

`~/claude-system/claude/hooks/pre-compact.sh` currently discards the hook
payload and emits a fixed reminder. Current text:

```bash
# Swallow stdin payload; we do not currently need its fields.
cat > /dev/null || true

cat <<'EOF'
PreCompact: before compaction, persist anything load-bearing to disk.

- If you are in a research project and have un-logged findings from this
  session, run /wrap to append Did/Findings/Next to NOTES.md now.
- If you generated experiment results that are not yet committed to
  metrics.json or results/, write them.
- Auto-memory is a hint, not a home — promote anything the next session
  must see into a tracked file.
EOF
```

Proposed text — parse the payload's `transcript_path` (already supplied by
the harness, currently thrown away) and append the address to the reminder:

```bash
payload="$(cat || true)"
transcript="$(printf '%s' "$payload" | jq -r '.transcript_path // empty' 2>/dev/null || true)"

cat <<'EOF'
PreCompact: before compaction, persist anything load-bearing to disk.

- If you are in a research project and have un-logged findings from this
  session, run /wrap to append Did/Findings/Next to NOTES.md now.
- If you generated experiment results that are not yet committed to
  metrics.json or results/, write them.
- Auto-memory is a hint, not a home — promote anything the next session
  must see into a tracked file.
EOF

if [ -n "$transcript" ]; then
  cat <<EOF

The full pre-compaction transcript stays on disk at:
  $transcript

Compaction demotes detail; it does not delete it. Write the address with
whatever you persist — cite this path (and the file + heading you wrote to)
so a later turn can recover the exact original instead of trusting the
summary. If after compaction the summary looks like it dropped something,
read this file rather than reconstructing from memory.
EOF
fi
```

Net: +9 lines in one existing hook. No new file, no new hook registration,
no new `settings.json` key, no new dependency (`jq` is already used by
`session-start.sh` and `session-end.sh`, with this exact
`// empty` / `2>/dev/null || true` guard idiom — phase 2 of the accepted
instruction-ablation program single-sourced that boilerplate, and this
follows it).

## Why (logical case)

The hook today implements *half* of a protocol. It tells the agent to flush
state to disk before the window is compressed — the write side. It leaves
nothing behind that points at what was compressed — the address side. After
compaction the agent holds a summary and no handle to the raw span the
summary replaced, so its only recovery move is to reconstruct from the
summary itself, which is precisely the content whose fidelity is in doubt.

This is not a hypothetical gap on this box. The transcript is already a
durable, content-complete, stably-addressed store: verified on disk at
`~/.claude/projects/<project-slug>/<session-id>.jsonl`, one file per
session, retained for months (this project's directory holds sessions back
to mid-July 2026). The harness already hands the hook that exact path in
the PreCompact payload. The hook throws it away with an explicit comment
saying the fields aren't needed. Everything the invariant requires exists
except one line of plumbing.

The project *already runs this protocol everywhere else*, which is the real
argument. `raw/` is immutable and every literature note is addressable by
citekey; `/ingest` writes notes that cite their `source:` file;
`citation-anchoring` is enforced as `/lint` check #12. The one place the
box compresses information without leaving an address is the one place the
compression is involuntary. This proposal makes the compaction path
consistent with the curation path rather than importing a foreign pattern.

## Why (reputable evidence)

The invariant — *eviction is offload, not deletion; the summary carries the
address of the raw span* — is the settled core of
`concepts/lossless-context-offload`, attested by five independent groups
using five different mechanisms. Two release code; one supplies a formal
guarantee with a matching lower bound. Gate 1 passes on all three of its
disjuncts at once.

- **`li2026acm`** (CMU + Meta; credibility 4; code, data, and checkpoints
  released at `lixiaochuan2020/agentic-context-management`). ACM's
  `manage_context` writes evicted raw messages to external storage under a
  unique identifier that the retained summary carries; `query_memory`
  recalls by that identifier. Billed explicitly as the "lossless" property
  distinguishing it from ReSum/ACON/Mem1 summary agents (Table 1). Measured:
  +27%/+16%/+8% on BrowseComp-Plus / DeepSearchQA / SWE-bench Verified over
  ReAct with ~20% lower peak tokens, and — the relevant number here —
  pass⁴ (consistency) 34.1 → 59.3 while pass@4 (capability) moves modestly.
  Addressable offload buys *reliability*, not raw ability, which is exactly
  the axis a session-continuity hook sits on.
- **`xu2026llm`** (CUHK + Tencent LightSpeed; credibility 4; code + project
  page at `binyxu/VISTA`). `archive(S, ρ)` returns a handle carrying path,
  level, size, and checksum, and **recovery is an ordinary file read of the
  archive path — no retrieval oracle, no third context tool**. That is
  precisely the shape proposed here: the address is a filesystem path the
  agent reads with the tools it already has. Its Prop. 1 is the formal
  floor — under budget pressure, any non-recovering method's correctness is
  information-theoretically bounded regardless of how good the compression
  is. Training-free and cross-backbone (Claude Sonnet 4.5, DeepSeek-V4-Pro,
  GLM-5, Gemini-3-Flash), so the result is a property of the interface, not
  of one model.
- **`dang2026addressable`** (Fujitsu Research of America × RIKEN AIP;
  credibility 3; formal appendix, no code). The strongest *formal*
  attestation: Theorem 1 guarantees the active view stays ≤ budget while
  every stored observation is exactly reconstructible, with an appendix
  argument that linear external-memory growth is unavoidable for worst-case
  exact recovery. Its framing is the cleanest statement of the logical case
  above — "what to keep" and "what to show" are separable decisions, and
  "recovery is then a decision that the agent makes, not a probability that
  the retriever gets right." Measured 99.40% vs 88.12% best-baseline recall
  on needle-in-a-haystack; McNemar paired tests throughout, p ≪ 0.001.
- **`hao2026selfgc`** (credibility 3). `fold` moves an exact payload to a
  sidecar and leaves a recovery pointer *on the control plane*; only
  `prune` is destructive and is reserved for provably obsolete content. The
  control-plane placement is why this proposal puts the address in hook
  output (a harness-authored channel) rather than asking the agent to write
  it in prose.
- **`semenov2026beyond`** (credibility 2 — weakest of the five, carried by
  the others, not carrying them). Evicts records whose effects are already
  persisted in the environment; the environment is the addressable store.
  Together with `chen2026governance` it documents the failure mode this
  fixes: unpredictable lossiness and compression-induced hallucination,
  because the raw span is gone by the time the agent discovers the summary
  dropped what mattered.

Two of the five (`li2026acm`, `dang2026addressable`) disagree sharply about
*who decides what to evict* — trained agent vs deterministic policy — and
all five agree on the invariant regardless. The concept note calls this out:
the invariant is the more settled claim and the one worth importing first.
This proposal touches only the invariant and leaves the contested
policy-locus question alone.

## Simplicity assessment

**Gate 2 is the reason this one is worth raising and the previous three
compaction ideas were not.**

- **It adds no new surface**: no skill, no hook, no rule, no config knob.
  One existing hook gains 9 lines and stops discarding a payload field.
- **It removes a discard rather than adding a mechanism.** The line
  `# Swallow stdin payload; we do not currently need its fields.` is deleted
  and replaced by a use. The information was already flowing into the hook.
- **A simpler form was considered and rejected as strictly worse**: leaving
  the hook alone and adding a sentence to `rules/agency.md` telling the
  agent to cite the transcript. That fails because the agent cannot reliably
  *know* the transcript path — it would have to guess a session-id filename.
  The whole point of `xu2026llm`'s Thm. 1 is that a manager acting on state
  it cannot perceive acts wrongly; a prose rule that names a path the agent
  can't resolve is that failure exactly. The harness has the path; the
  cheapest correct fix is for the harness to say it.
- **A more complex form was considered and rejected**: a hook that itself
  copies or snapshots the transcript into the project (a real archive
  operation). That duplicates a file the harness already retains, adds disk
  churn and a cleanup obligation, and buys nothing — the address is
  sufficient because the store already exists.
- **This does not re-open the eviction-policy question.** The 2026-06-23 run
  held "Last-K-tool-calls + summary eviction in the pre-compact hook" on
  evidence (a credibility-2 preprint), and the 2026-08-09 run held
  `context-eviction-policy` for having *no claude-system target* because
  compaction lives inside the Claude Code harness. Both holds stand and this
  proposal respects them: it does not try to choose what gets evicted, which
  genuinely is harness-internal. The address side is entirely outside the
  harness, which is why it has a target where the policy side does not.
- **HCE discipline is untouched.** The transcript is the agent's own prior
  context, not held-out data; nothing here reads `test/` or weakens
  `rules/evaluation.md`. It is a strictly read-back-your-own-work channel.
- **No target collision.** The three proposals still pending
  (`rules/evaluation.md`, `templates/project/budget.yaml`,
  `skills/lint/SKILL.md`) do not touch `hooks/pre-compact.sh`, so this does
  not stack an undecided edit on a file already under review — the process
  constraint that blocked several ideas on 2026-07-26 and 2026-08-09.

## Risks & what could make this wrong

- **The field name is unverified against a live payload.** `transcript_path`
  is the documented PreCompact field, but this proposal was written from the
  hook source and the harness docs, not from a captured payload. The
  reviewer should confirm before applying. The failure direction is safe by
  construction: `// empty` plus `2>/dev/null || true` means a wrong or
  missing field silently omits the extra block and leaves today's behavior
  exactly intact. It cannot break the hook. But it could silently do
  nothing, which is worse than a loud failure — so **verify, don't assume**.
- **The injected text may not survive compaction.** The reminder is emitted
  into pre-compaction context, so the summarizer may drop the path itself.
  This is why the wording tells the agent to *write the address into the
  durable file* (NOTES.md / journal) rather than relying on the injected
  line persisting. If it survives, the agent has the handle directly; if
  not, the handle is on disk where `/wrap` put it. The mechanism does not
  depend on which happens — but the weaker case means the benefit routes
  through the agent actually following the instruction, and prose
  instructions are the least reliable enforcement family this project
  recognizes (cf. `typed-enforcement`).
- **Recall is the residual lossiness.** All five sources store losslessly;
  none guarantee the read path. `li2026acm`'s querier LLM introduces its own
  errors, and `dang2026addressable` scopes its theorem explicitly to
  address-conditioned recoverability, *not* end-to-end success — the agent
  must still choose to look. A `.jsonl` transcript is verbose and awkward to
  read; an agent that dereferences it badly gets a worse answer than one
  that trusted a good summary. This proposal buys the *option* to recover,
  which is what the theory says is necessary, not sufficient.
- **Evidence transfer is by analogy on one axis.** All five papers measure
  agents making *explicit* archive/recall decisions inside a benchmark. The
  Claude Code agent does not choose to compact — the harness does it. The
  invariant transfers cleanly (it is about the store, not the decider), but
  no cited experiment measures a harness-initiated compaction with a
  handed-back address. Nobody has run this exact configuration.
- **A small permanent token cost** on every compaction, forever, for a
  benefit that only materializes when a summary actually loses something
  load-bearing. If the harness's own compaction is good enough in practice,
  this is ~90 tokens of dead weight per compaction event.

## Recommendation

**Adopt**, after confirming the payload field name against a live PreCompact
invocation. The evidence base is the strongest available for any idea in the
graph on this axis — five independent groups, two released artifacts, a
formal theorem with a matching lower bound, and an ablation
(`xu2026llm`: 37.3 without the ledger vs 45.3 without recovery vs 50.7 full)
showing the effect is real and separable. The change is nine lines in a file
that already exists, consuming a value the harness already computes and the
hook already receives and deliberately discards. It contradicts none of the
standing holds — it is orthogonal to the eviction-policy question those
holds correctly parked. The honest weakness is that the benefit routes
through a prose instruction the agent may ignore, and that the recall path
into a `.jsonl` is clumsy; both argue for adopting the cheap version now and
watching whether the address ever actually gets dereferenced, rather than
building anything heavier.
