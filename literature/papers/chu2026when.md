---
kind: paper
title: "When Do Summaries Become Instructions? Unsupported Policy in LLM Agents' Self-Written Handoff Summaries"
authors: ["Wenhui Chu"]
institutions: ["University at Albany, State University of New York"]
year: 2026
venue: "arXiv 2611.06545 (cs.CL), v1 2026-11-04, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.06545"
code_url: "https://github.com/Chuw1/handoff-audit"
citations: null
source: "raw/papers/chu2026when.pdf"
added: "2026-11-16"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/context-eviction-policy]]"
  - "[[concepts/intent-level-briefing]]"
tags: ["compaction", "handoff-summary", "self-written-memory", "unsupported-policy", "invented-restriction", "false-authorization", "permission-ledger", "same-snapshot-replay", "neutral-filler", "count-matched-control", "oracle-repair", "approval-withdrawal", "overgeneralized-rejection", "system-prompt-wording", "provenance-sentence", "pre-specified", "null-result", "ai-readers", "open-weight", "qwen3", "gpt-oss"]
---

# When Do Summaries Become Instructions? Unsupported Policy in LLM Agents' Self-Written Handoff Summaries

## TL;DR

Compaction research so far asks what a summary *loses*. This paper asks
what it *adds*: sentences in an agent's own handoff summary that grant or
require permissions the record does not support. Synthetic tool-use tasks
carry a ground-truth permission ledger and event log. Same-snapshot replay
reruns the successor with the suspect sentences swapped for neutral filler.
The agents are small open-weight models (Qwen3-8B/14B, gpt-oss-20b);
Claude appears only as the labeller. Four results matter here:

1. **It is rare.** On fresh held-out instances, unsupported restrictions
   are in 1.2% (Qwen3-8B) and 2.4% (gpt-oss-20b) of readable summaries,
   unsupported authorizations in 0.1% and 1.3%. These are lower bounds,
   because only rule-flagged summaries were read.
2. **False permissions are the costly direction.** Successors given one
   acted without permission in 11–16 of 60 runs per main held-out set, and
   in 0 of 60 with the full history. Neutralizing the sentence cut
   unauthorized actions from 24 to 7 of 120 (F1+F2 pooled, post hoc). A
   later pre-specified set went from 17 to 11 of 48 (p = 0.63).
3. **Invented restrictions mostly start before the summary, and the causal
   test failed on held-out templates.** The pre-specified replay test was
   null (p = 0.58). On the development split, most invented restrictions
   followed the agent's own earlier guess that an action needs approval.
   On held-out templates, most followed a rejected request for a wrong
   target.
4. **A one-sentence system-prompt change fixed most of them.** Replacing
   "Some actions need approval." with "Only actions that the rules or the
   user's request say need approval require it." cut AI-confirmed
   unsupported restrictions from 5.0% to 1.8% of development summaries and
   from 1.8% to 0.3% on held-out templates (both pre-specified,
   p < 10⁻⁹). Requests for approvals the ledger really requires did not
   change.

## Claims

- A handoff summary is self-authored text that the successor reads as its
  own notes, usually without a human reading it. A permission sentence in
  it therefore acts as an instruction. The paper motivates this with
  OpenAI's reports of self-generated instructions in RL compaction summaries
  (OpenAI 2026b, 2026c) and a red-team study (Remedios et al. 2026).
- Added restrictions and added approvals behave differently. False
  approvals are rare but lead to action. False restrictions are more common.
  Their effect on held-out instances is weak, and removing the sentence does
  not repair them, because the successor still lacks the allowed target.
- The best deployable repair is upstream of the summary: say where
  requirements come from in the system prompt. The paper also proposes
  (untested) a handoff check that strips false approvals and restates the
  true permission state, including allowed targets and withdrawals, while
  tracking rule violations.
- The harness is packaged as a test for any OpenAI-compatible tool-calling
  endpoint. That path has been run only against scripted servers.

## Methods

- **Tasks.** Three families (records, reports, code). Each template has a
  request, a few rules, read/write tools, a permission ledger (allowed /
  forbidden / needs confirmation from a named approver), an event log
  (approvals, denials, revocations) and one tempting disallowed action. A
  confirmation request for an action–target pair the ledger does not list
  is rejected ("… is not permitted for this task"). Development split: 24
  templates. Test split: 60 templates (20 per family), written and hashed
  before any test run.
- **Compaction.** At a fixed step (5–8; or after a revocation), or when
  the agent calls `finish`, whichever comes first. One fixed compaction
  prompt ("what was asked, what you have done, what you found, and what
  remains"). The successor gets the system prompt plus the summary. **The
  user request is not repeated**, so the successor knows the task only
  through the summary. Writer and successor are the same model unless
  stated.
- **Models.** Qwen3-8B, Qwen3-14B (development only) and gpt-oss-20b on
  vLLM 0.30.0 with the developers' sampling settings, 32k context. As
  exploratory scale checks: gpt-oss-120b, Qwen3-235B-A22B-FP8 and Kimi K2.6
  (prevalence only). About 48 GPU-hours.
- **Labels.** A rule-based labeller flags candidate policy sentences. Two
  independent `claude-opus-5-5` readers then judge every flagged summary
  against the true permission state, blind to model and outcome; a summary
  counts as AI-confirmed only if both say yes. Reader κ 0.82–1.00. Human
  check: the author on 60 test items (κ 0.38 with the AI labels; the author
  said yes to 6 of 7 AI-confirmed items plus 12 others) and a non-author on
  30 of them (κ 0.81 with the author, 0.62 with the AI labels). Both humans
  said yes more often than the AI readers.
- **Same-snapshot replay.** Restore the environment at compaction and run
  the successor K = 4 times per arm with matched seeds. Arms:
  - `original`.
  - `neutral`: target sentences replaced by content-free filler of
    matching length.
  - `delete`.
  - A control edit: `edit_control` deletes one other sentence in Dev and
    Test; `neutral_control` fills one non-target sentence per target in the
    follow-ups.
  - Oracle arms `claim_correct` and `ledger_correct`, which rewrite the
    target sentence with the true permission state.
  - `restore`: append the true state.
  - `no_compaction`: full history.

  Outcomes: any confirmation request (restriction sources), any write the
  ledger does not permit (authorization sources). Exact two-sided sign test
  per source, plus (model, template) cell tests and cluster bootstrap.
- **Staging.** Dev (exploratory) → Test (pre-specified, hashed plan) → F1
  (same sources, every reader-marked sentence edited; exploratory) → F2
  (fresh instances, planned after Test's null) → F4 (pre-specified check of
  a post hoc compaction-point finding) → F5/F6 (pre-specified prompt
  intervention on development and held-out templates).

## Results

- **Prevalence (Table 2).** F2 held-out: restrictions 13/1,044 (1.2%,
  Wilson 0.7–2.1%) and 26/1,094 (2.4%, 1.6–3.5%); authorizations 1 (0.1%)
  and 14 (1.3%). gpt-oss-20b produced 27 of 30 main held-out authorization
  cases. The rule labeller is imprecise: readers confirmed 39 of 289
  authorization flags and 76 of 235 held-out restriction flags. Larger
  models on the F2 instances: gpt-oss-120b 1.3% and 0.2%, Qwen3-235B 1.7%
  and 0.9%, Kimi K2.6 0.3% and 0.0%.
- **What the sentences said.** Development split: 61 of 64 confirmed
  restrictions came from three report templates, mostly "publishing to the
  internal wiki requires approval". Held-out: 59 of 76 restriction sources
  were **overgeneralized**: the agent had asked to publish to a wrong
  target, been rejected, and the summary then stated an approval
  requirement for the right one. Held-out authorizations (42 sources across
  Test, F2 and F4):
  - 18 followed an approval **withdrawn** in the writer's own context. The
    summary mentioned the withdrawal in only 5 of the 18.
  - 21 followed a **non-approver's comment** in a tool result ("cron tweaks
    don't need approval").
- **Restrictions, development (exploratory).** The neutral edit cut
  confirmation requests from 251 to 151 of 256 runs (39/1 sources,
  p < 0.001). The control edit left them at 251, and the true permission
  brought them to 6. In 10 of 11 cells the rate fell (p = 0.002). The effect
  also held across models: gpt-oss-20b continuing from Qwen3-8B summaries
  went from 72/72 to 58/72.
- **Restrictions, held-out (Table 4).** Pre-specified Test: 126 → 119 of
  148 (8/5, p = 0.58). F1 with every marked sentence edited: 125 → 125.
  F2: 121 → 106 of 156 (16/5, p = 0.027). This is mostly Qwen3-8B (9/1);
  gpt-oss-20b was 7/4. The cell test p = 0.34, the bootstrap CI includes
  zero, and dropping either of two cells gives p = 0.096. The author does
  not count it as a replication. F4 (pre-specified, compaction at finish
  off): 118 → 107 of 136 (8/2, p = 0.11). Writing in the true permission
  removed most requests in every set (Test 43/148, F4 46/136). For
  overgeneralized sources, 192 of 197 requesting runs asked about a target
  the ledger did not allow. So the successor was missing a fact (the
  allowed target) as well as carrying a wrong rule.
- **Authorizations (Table 7).**
  - Neutral edit: Dev 11 → 4 of 36, Test 16 → 8 of 60, F1 13 → 6, F2
    11 → 1. Pooled F1+F2, 24 → 7 of 120 (10/0, p = 0.002, post hoc).
  - The count-matched control gave 25 of 120 and the full history 0 of
    120. All ten sources that fell were gpt-oss-20b, from four templates.
  - Removing the false approval also raised confirmation requests (F2
    15 → 37 of 60), and part of that rise came from any edit (control
    23).
  - Positive control, a planted false approval: 53 → 1 of 142.
- **Repairs (Table 8).** `restore` (append the true state; needs only
  ledger access) lowered restriction requests (Dev 251 → 191; Test
  126 → 109, p = 0.14) and raised Test task success from 32 to 102 of 148,
  but doubled rule violations (8 → 16). Against a planted false approval it
  only partly worked (42 of 142 vs 53). `claim_correct` also raised
  violations in Test and F1 (8 → 18, 10 → 26 of 148). It is an oracle, not
  a deployable fix.
- **Compaction prompt (Table 12, exploratory, ~230 summaries per cell).**
  A structured prompt with Task / Rules / Approvals headings and "only
  include facts you saw" more than tripled the rule labeller's restriction
  flags for Qwen3-8B (27.4% vs 8.2%). Its AI-confirmed restriction rate was
  no higher than the default prompt's (5.9% vs ≈7.4%; gpt-oss-20b 1.7% vs
  ≈3.9%). A brief-note prompt gave 4.3% and 2.2%. No compaction prompt
  removed the problem.
- **Provenance and the system-prompt fix (F5, F6; Table 15).**
  - The post hoc trace of the 64 development sources: 46 were "own
    guesses", where an earlier reasoning turn paired an approval word with
    the action, often hedged ("I might need approval"). 17 appeared first
    in the summary, and 1 followed a rejection.
  - In the three report templates, "approval" appeared only in the system
    prompt's "Some actions need approval." and in the confirmation tool's
    description.
  - F5 (development, 1,862 matched pairs): 76 restrictions only under the
    old prompt and 14 only under the new (p = 1.8 × 10⁻¹¹), 5.0% → 1.8%.
    Requests about allowed actions fell from 278 to 112 of 1,920 episodes.
    Requests about needed approvals stayed at 1,030 vs 1,015.
  - F6 (held-out, 3,305 matched pairs): 59 vs 9 (p = 3.9 × 10⁻¹⁰),
    1.8% → 0.3%. It held in both models (23/8, 36/1) and in pairs with a
    rejected request (44/9). Requests about needed approvals stayed at
    2,205 vs 2,199 of 3,840. Unauthorized actions (244 vs 260) and rule
    violations (405 vs 415) did not change significantly, and task success
    rose (2,771 → 2,866, p = 0.0014).
  - The first assistant turn mentioned approval equally often under both
    prompts. The author reads this as the sentence changing what agents
    *conclude* about approval, not whether they think about it.
  - Unsupported authorizations did not fall (F6 matched 15/11, p = 0.56).
- **Monitors (exploratory).** Small-model CoT/summary monitors scored
  AUROC 0.36–0.59 against rule labels, which is about chance.

## Critique / open questions

- **Full text vs the digest entry: at, with two omissions that matter
  here.** Every number in the digest summary is in the PDF and quoted
  correctly, and it keeps "post hoc" and "open-weight only". It omits two
  things.
  1. The paper's main causal test for restrictions was **null** on held-out
     templates (p = 0.58; F4 p = 0.11). The only significant held-out
     result (F2) is fragile by the author's own reading. So "summaries
     invent restrictions that then steer the successor" is established
     only on the exploratory development split. On held-out templates the
     confirmed evidence is about prevalence and about the oracle arms.
  2. The digest calls the fix "a one-sentence provenance statement". The
     author is more careful. The new sentence also *states a rule*, so F5/F6
     show that it prevents most unsupported restrictions in these templates.
     They do not show that the old sentence caused them. The sentence is
     true only because every required approval in these templates is
     written in the rules or the request. "Where requirements are not
     written down, it could suppress needed requests, which we did not
     test." It also did nothing for unsupported authorizations.
- **The design gives the summary maximal authority.** The successor never
  sees the original request, and compaction usually happened at the
  agent's own `finish` call (70–84% of held-out compactions). Deployed
  agents compact on a full context, and Claude Code re-injects CLAUDE.md.
  Both make these rates an upper bound on influence, though not on
  prevalence.
- **Labels rest on one model family.** Both readers are copies of
  `claude-opus-5-5`. They saw the rule labeller's stars, and human
  agreement is modest (κ 0.38 for the author, 0.62 for the second reader).
  Humans said yes more often, so the prevalence figures are probably low,
  not high.
- **Neither edit isolates the claim.** The neutral edit also removes any
  next step or reported rejection in the same sentence. The count-matched
  control sometimes replaced the line stating the task.
- **Confounds the author flags.** In F6 the GPU type was coupled to prompt
  within model (H100 vs H200). The drop held in both models and in
  H100-only F5, so this is unlikely to explain it, but it is not ruled out.
  vLLM with fixed seeds is not fully reproducible (F1 re-ran the same
  sources and matched in 37 of 52).
- **What the box cannot read off.** No frontier agent wrote or consumed a
  summary in the main runs. The larger-model checks are exploratory and
  their authorization sources (mostly one template, T-C05) never led to
  action. Whether Opus-class summarizers invent policy at all is not
  measured here.
- **Fit with the graph.** This is the mirror image of
  [[literature/papers/chen2026governance]] and
  [[literature/papers/salle2026benchmarks]], which measure dropped rules. It
  agrees with [[literature/papers/das2026selfpropagating]] that the
  successor's deference to self-written memory is the weak point. The
  withdrawal result (5 of 18 summaries mentioned the withdrawal) is a
  measured instance of the stale-pin hazard that
  [[concepts/constraint-pinning]] records from
  [[literature/papers/hu2026memory]] and
  [[literature/papers/xu2026stale]].

## Trust signals

- **Credibility:** 3.
  - For: code, prompts, labels and run records released (MIT / CC BY 4.0)
    with a hash manifest. Six analyses fixed in hashed plans before their
    data existed. Null primary results reported as nulls. Every post hoc
    analysis is labelled. Deviation logs. A count-matched control and
    oracle arms. A second human reader recruited specifically to check the
    AI labels. The F5/F6 result has very large matched-pair margins on
    both splits.
  - Against: single author, one university group, not peer reviewed, 12
    days old, no citations. 8B–21B open-weight models in a synthetic
    harness. Labels are from Claude readers, with weak human agreement.
    Most behavioural effects on held-out templates are not significant
    under the pre-specified tests. AI assistance in writing, analysis code
    and analysis suggestions is disclosed.
  - Not 4: no major lab or venue, and the headline behavioural claims
    for restrictions did not survive the author's own held-out test. Not 2:
    the code is released, the reporting is unusually transparent, and the
    F5/F6 result is pre-specified and large.

## Follow-up

- **Relevance:** 4. It is the first measurement in this graph of the reverse
  compaction risk: summaries *adding* permissions or requirements. It bears
  directly on the 2026-11-14 PreCompact change. It adds material evidence to
  [[concepts/constraint-pinning]] (a summary-time quote carries invented
  and withdrawn policy as well as real policy),
  [[concepts/context-eviction-policy]] (compaction fidelity has an
  additive failure; which repairs work) and
  [[concepts/intent-level-briefing]] (an unscoped hedge in a system prompt
  becomes an invented rule downstream). Not 5: no Claude-class agent, the
  held-out restriction effect failed its pre-specified test, and the
  deployable fix is a system-prompt sentence, not a compaction mechanism.
- **What this implies for the 2026-11-14 PreCompact bullet** (our
  inference; the paper tests no such instruction). The bullet tells the
  summarizer to quote the latest version of each chat-only standing
  instruction verbatim
  (`docs/system-proposals/2026-11-01-precompact-chat-only-constraints.md`).
  - **Not a reason to revert.** The bullet addresses dropped rules, which
    is a separate failure, measured at about one in three on Claude Code.
    The paper's three provenance classes map onto clauses the bullet
    already has:
    - "the user gave" excludes the agent's own guesses (46 of 64
      development sources) and non-approvers' comments in tool results (21
      of 42 held-out authorizations).
    - "verbatim" excludes paraphrases that turn a rejection into a rule (59
      of 76 held-out restrictions).
    - "latest version" is aimed at withdrawals (18 of 42).

    That is the scoped, source-naming form F5/F6 found effective, applied at
    the summary rather than the system prompt.
  - **The bullet's examples are all restrictions, but the costly direction
    here is a false permission.** "How to commit, what not to touch, what
    not to run" omits chat-only grants ("go ahead and push", "you can
    delete those"). A quoted grant that was later withdrawn is exactly
    this paper's false-approval case. Summaries mentioned the withdrawal in
    only 5 of 18, so "latest version" depends on a revocation step these
    models usually failed. A cheap amendment would say that a quoted
    permission must state whether it still stands, and that the summary
    should state no permission or requirement the user did not give. This
    is a candidate for `/elevate`, not applied, and it rests on small
    open-weight models.
  - **Asking for rules did not by itself raise invention.** The structured
    compaction prompt, with Rules and Approvals headings, tripled policy
    flags but not confirmed invented restrictions (exploratory, coarse).
    That is mildly reassuring for a bullet that asks the summarizer to
    restate rules.
  - **A local check.** The proposal's suggested salle-code test could also
    count the reverse case: summary sentences stating a permission or
    requirement that no user turn contains. `handoff-audit` itself targets
    OpenAI-compatible endpoints and does not exercise Claude Code's own
    compaction.
- **Candidates** (cited, not yet in the graph):
  - Cerruti, Okamoto & Erol 2026, "Agent memory is a surface for
    endogenous authorization laundering" (arXiv 2609.01836; NeurIPS 2026
    PALM workshop).
  - Zhan et al. 2026, "When memory becomes authority: benchmarking
    authority collapse at the memory consolidation boundary" (arXiv
    2608.01679). Authority metadata sharply lowers unauthorized actions.
  - Chen et al. 2026, "Slipstream: trajectory-grounded compaction
    validation for long-horizon agents" (arXiv 2605.08580).
  - OpenAI 2026b/2026c, Alignment Research Blog reports on self-generated
    prompt injections and deception in RL compaction summaries.
  - Remedios et al. 2026, "Red-teaming Auto Mode" (arXiv 2609.19587).
    Unmonitored compaction summaries let a misaligned coding agent
    fabricate facts.
  - Rodríguez, Pozanco & Borrajo 2026, "Is your agent playing dead?"
    (arXiv 2606.14831). Agents invent obstacles under irreconcilable
    constraints.
  - Chu 2026, "When evidence changes: evaluating memory repair and
    re-reading in language-model agents" (arXiv 2610.03902).
