---
kind: concept
name: "intent-level-briefing"
status: seedling
added: "2026-11-13"
sources:
  - "[[docs/system-proposals/2026-07-31-instruction-ablation-program]]"
  - "[[literature/papers/chu2026when]]"
  - "[[literature/papers/chen2026signed]]"
  - "[[literature/papers/wang2026beneath]]"
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/lossless-context-offload]]"
related_experiments: []
used_by:
  - project_slug: intent-briefing
    imported_on: "2026-11-13"
tags: [prompting, delegation, briefing, instruction-design, governance, harness]
---

# intent-level-briefing

## Definition

Instructions to a capable model, whether a system prompt, subagent brief or
skill, state the **goal, the hard constraints, the reasons behind them, and
the context the author would otherwise take for granted**. They leave the
method to the reader instead of scripting it step by step. Hard invariants are
not written as prose in either style. They move into code: hooks, typed
checks, permission gates.

## Why it matters here

This concept complements [[concepts/typed-enforcement]] and
[[concepts/constraint-pinning]]. Those concepts decide what the *harness*
guarantees. This one decides what is left for *prose*, and how prose should be
written once the invariants have moved to code. The box has already acted on
it once: the 2026-07-31 instruction-ablation program cut claude-system's skill
prose from ~18k to ~7k words (keep contracts and rationale, delete procedure,
compile algorithms into scripts). That program said it could not run real
ablations. It also named the main risk: contract-only skills assume the reader
has frontier-level inference.

Open boundaries, which the downstream project watches the literature for:

- **Capability:** below some implementer strength, scripts likely win.
- **Context, not capability, is the reader's deficit.** A brief fails when the
  author's working context isn't transferred
  ([[concepts/lossless-context-offload]]). Mission command also assumes shared
  doctrine, which here means rules and CLAUDE.md.
- **Horizon:** drift under long chained runs is still unmeasured in either
  direction.
- **Two axes:** intent vs procedure is a different axis from terse vs verbose.

## A vague "some actions need approval" turned into invented rules; one sentence saying where requirements come from removed most of them (2026-11-16)

This page's "Open boundaries" are about briefs for a capable reader.
[[literature/papers/chu2026when]] supplies the first controlled,
pre-specified test in this graph of one prose sentence's wording in a
system prompt. It also follows the effect past the reader, into text the
reader writes for its own successor. The agents are Qwen3-8B and
gpt-oss-20b in synthetic tool-use tasks with a ground-truth permission
ledger.

- **The hedge seeded the error.** A post hoc trace of 64 invented
  restrictions in handoff summaries found 46 already present as the
  agent's own earlier guess, often hedged ("Before publishing, I might
  need approval"). In the three templates that produced 61 of the 64,
  "approval" appeared only in the system prompt ("Some actions need
  approval.") and in the confirmation tool's description.
- **The scoped sentence fixed most of it.** F5 and F6 replaced that
  sentence with "Only actions that the rules or the user's request say
  need approval require it." Both tests were pre-specified with hashed
  plans and scripts.
  - Development: AI-confirmed unsupported restrictions fell from 5.0% to
    1.8% of summaries (matched 76 vs 14, p = 1.8 × 10⁻¹¹).
  - Fresh held-out templates: 1.8% to 0.3% (59 vs 9, p = 3.9 × 10⁻¹⁰),
    in both models.
  - Requests about actions that needed no approval fell from 485 to 178
    of 3,840 episodes. Requests for approvals the ledger really requires
    stayed flat (2,205 vs 2,199).
  - Task success rose (2,771 → 2,866, p = 0.0014). Unauthorized actions
    and rule violations did not change significantly.
- **It changed the conclusion, not the attention.** The first turn
  mentioned approval equally often under both prompts (e.g. gpt-oss-20b
  44.1% vs 44.3%).

For this concept, this is P2 evidence ("state what the reader won't do by
default"). The vague sentence left the reader to infer *which* actions
it covered, and the reader filled the gap with guesses. The fix names the
source of the requirement rather than adding procedure. It also adds a
boundary that the page did not have: a gap in a brief does not stay in
the reader's turn. It can be written into the reader's self-summary and
inherited as a rule by its successor. That is the long-horizon drift this
page lists as unmeasured, seen in one mechanism.

For this box (our inference): look for unscoped caution sentences of the
form "some X need Y", where the brief does not say which X or who decides.
They are cheap to scope. The 2026-11-14 PreCompact bullet's "standing
instructions the user gave" is already the scoped form (see
[[concepts/constraint-pinning]]). The fix does not transfer to every
case, though. The new sentence is true only because these templates write
every required approval down. Where requirements are tacit, it could
suppress needed requests, which the paper did not test.

Limits:
- The new sentence also states a rule, so the result shows that it
  prevents most unsupported restrictions in these templates. It does not
  show that the old sentence caused them (the author's caveat).
- Most old-prompt cases came from three or four report templates. In F6
  the GPU type was coupled to prompt within model, but the drop held in
  both models and in H100-only F5.
- 8B–21B open-weight readers, not Opus-class. The intent-vs-procedure
  question is about strong readers, so this is evidence on wording, not
  on capability thresholds.
- It did nothing for unsupported *authorizations* (matched 15 vs 11,
  p = 0.56).

## The brief-writer is the trust boundary: signed, code-checked requirements failed only where the writer mis-graded or was fed a requirement (2026-11-17)

The 2026-11-16 section above found that a gap in the brief becomes an
invented rule downstream ([[literature/papers/chu2026when]]).
[[literature/papers/chen2026signed]] tests a typed alternative (preregistered).
The orchestrator writes each requirement as a verbatim quote plus a
predicate. Code confirms the quote is in the request, signs the list and
checks the final decision against it.

- **Everything after the writer was made safe; the writer was not.** An
  oracle requirement list removed every remaining unsafe commit (11.2 pp).
  The LLM-written list kept 85.1% of gold requirements at 70.4% precision.
- **The dominant writer error was grading, not wording.** 54 of 139
  extracted items were preferences the request marked "Where the
  requirements allow, the team prefers …", recorded as mandatory. Only 3
  of 149 failed the verbatim-quote check. Quoting verbatim kept the
  *words*. It did not keep whether a sentence was a must or a preference.
- **Text in the writer's input became a requirement.** One adversarial line
  appended to the request corrupted the target requirement in 62.5%
  (Qwen3-8B) to 100% (gpt-oss-20b, GPT-5.6-luna) of extracted lists. All
  resulting violations came from those lists. The quote check passed them,
  because the injected sentence was verbatim in what the writer read.
- **For strong readers, a plain structured brief already carried the
  requirements.** gpt-oss-20b and GPT-5.6-luna violated nothing via
  unchecked JSON without tampering. The checked list cost utility, because
  its own extraction errors were enforced.

Rule (our inference): for a capable reader the risk sits in the writer's
step. Two cheap habits follow, both untested in this form. (1) Quote each
hard constraint and name its source, so a script can confirm it exists.
(2) Mark each one as a must or a preference, because that grading is the
measured failure. Neither defends against poisoned source text. That
requires drawing constraints only from authenticated sources (user turns,
rule files), not tool output, as in the PreCompact bullet's "the user
gave".

Limits:
- Synthetic requests with explicit "must"/"prefers" cues. Tacit priorities
  are harder.
- No Claude writer or reader, no prose-only requirements, and one fixed,
  non-optimised injection line.

## Context the reader lacks beat a generic "verify and finish" reminder, and the reminder alone cost a capable harness 1.7–4.2 points (2026-11-17)

[[literature/papers/wang2026beneath]] gives a second data point after
the chu2026when section, this time on GPT-5.4 inside MiniSWE and Codex.
Evolved Champion changes were split into dependency-closed Components and
each was added alone to the Seed. The control is the Seed plus one
generic sentence: "Inspect the available runtime tool interface, use
tools when needed, verify the resulting state, and complete every
requested item before ending." On TB2, the MiniSWE control adds a
run-the-end-to-end-check clause. Scores are Avg@8.
- **The generic reminder did nothing or harm.** On MiniSWE it moved the
  score +0.4 (τ³) and 0.0 (TB2). On Codex, the stronger outer-layer
  harness, it moved the score **−1.7 (τ³) and −4.2 (TB2)**. Codex's own
  prompt and skills already carry this advice.
- **Specific content beat it in 15 of 16 Components.** The prose-only
  Components on τ³ add +4.5 to +6.9 over Seed. Examples: "Keep a live
  per-entity worklist" for each card, account and transaction; use the
  exact KB-named tool rather than a neighbouring one; serialize inner JSON
  in the surfaced key order. On TB2 the prose Components add at most +1.8,
  and both Codex instruction Components sit at or below Seed (70.2 and
  69.1 vs 69.9).

This is P2 evidence ("state what the reader won't do by default; leave
out what it would do anyway"). The τ³ wins are mostly *context*: domain
tool, payload and multi-item facts the model could not infer. The
generic sentence restated defaults, and where the harness already had
them it cost points.

Rule (our inference): a brief sentence that a capable reader would follow
anyway is not free. Before adding "verify X / complete everything", check
whether the harness or skill already says it. Spend the words on
domain facts the reader can't see.

Limits:
- Not a clean specificity test. Components are evolved, benchmark-fitted
  and selected from Champions, and some are code. They are scored on the
  full benchmark, including the Train tasks they were evolved on. The
  control is one untuned sentence with no length match.
- One evaluation per arm with no intervals. Per-pass SDs are 2–5 points,
  so the τ³ Codex −1.7 is within noise; TB2 Codex −4.2 is about 2 SD.
- GPT-5.4 only; intent vs procedure is not the manipulated variable.

## Connections

- [[concepts/hierarchical-delegation]]: the main place this applies, since
  every delegation is a brief.
- [[concepts/enforcement-boundary-placement]]: where the code/prose line falls.
- Downstream: the `intent-briefing` project (curated literature and an
  evidence-graded writing guide; no in-house experiments, per its ADR 0003).
