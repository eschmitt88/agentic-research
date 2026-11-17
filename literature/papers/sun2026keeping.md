---
kind: paper
title: "Keeping LLM Agents on Procedure: Executable Finite-State Control for Long-Horizon Instruction Fidelity"
authors: ["Yutao Sun", "Haokun Li", "Mingshuai Chen", "Guanyan Li", "Zhongyi Wang", "Shengchao Qin", "Jianwei Yin"]
institutions: ["Zhejiang University", "University of Oxford", "Xidian University", "Zhejiang Key Laboratory of Digital-Intelligence Service Technology"]
year: 2026
venue: "arXiv 2611.10884 (cs.AI), v1 2026-11-07, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2611.10884"
code_url: null
citations: null
source: "raw/papers/sun2026keeping.pdf"
added: "2026-11-17"
relevance: 5
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/lossless-context-offload]]"
tags: ["procedure-fidelity", "sop", "finite-state-machine", "runtime-enforcement", "execution-monitor", "stop-gate", "pretooluse-hook", "claude-code", "escape-routes", "drift", "long-horizon", "unattended-runs", "reminder-baseline", "auto-resume", "compaction", "pre-registered", "ablation", "claude-haiku", "claude-sonnet", "sop-bench", "tau2-bench", "swe-bench-verified"]
---

# Keeping LLM Agents on Procedure: Executable Finite-State Control for Long-Horizon Instruction Fidelity

## TL;DR

A procedure given as text drifts over a long unattended run, and the
usual fixes only move the drift. Restating the SOP raises compliance per
decision but the agent stops early. Auto-resuming it keeps it working but
it batches, interleaves or decides without looking. The paper puts the
procedure in an executable FSM held by a daemon outside the context
window. Claude Code hooks enforce it at three points: a `PreToolUse`
**tool-call gate**, a **transition guard** run before the state may
advance, and a `Stop` **stop gate** that refuses end-of-turn until the
workflow exits. On 200-account SOP-Bench batches (pre-registered, Claude
Haiku 4.5, 8 seeds) the FSM made 139.2 faithful-and-correct (FC)
decisions against 48.4 for the pre-designated strongest baseline
(Remind-loop): +90.9, 95% CI [46.2, 130.2]. **Strict accuracy did not
move significantly** (+7.6, CI [−16.1, 29.8]). The gain is in following
the procedure, which is largely what the FSM's guards check, and it
shrank or reversed on models that drifted less.

## Claims

- **Drift moves between escape routes.** Varying *information* (where the
  SOP appears) and *liveness* (resumption) separately, each prose
  condition leaves a route open and the agent takes it.
- **Each enforcement point closes one route; all three are needed.**
- **The gain tracks plain-agent drift** (large for Haiku 4.5, small for
  Qwen3.8-flash, not significant for Sonnet 4.6).
- **The runtime supplies reliability:** Haiku 4.5 + FSM averaged more FC
  than plain Sonnet 4.6 at 38% of the cost.
- **The guards are the work.** Useful ones check the *process* (tool
  order, git state, ledger fields), never the answer; failures were
  specification errors (over-strict guards, tight turn budgets).

## Methods

- **Formalism.** Workflow W = (S, s₀, s_exit, P, Δ): P(s) is the admitted
  tool set per state; each transition has a guard over the tool-call
  history (shell commands that must exit 0, optionally an independent
  model review). A daemon keeps per-agent frames (state, history,
  variables) on disk, so compaction cannot erase them and a fresh context
  reads them back; the agent reaches it via MCP tools and two hooks.
  Tool-call gate: u ∈ P(s_t) (`PreToolUse`). Transition guard: one-time
  token, guard runs, atomic commit. Stop gate: end turn only at exit or a
  declared pause (`Stop`); its message names remaining accounts and the
  next one, and comes from the same function as the baselines' resume.
- **Main workflow.** Three states: s1 investigate (five data tools plus
  the STEP 1–4 record), s2 decide (`submit_decision`), exit. The s1→s2
  guard requires the account undecided, all five data sets retrieved,
  and no other account's data fetched since. The guards never read data
  values or gold labels and do not encode the decision logic.
- **Task.** SOP-Bench v2 referral-abuse detection, unmodified, 200
  accounts, seven enforcement actions. Labels are skewed (134/200
  *No Action*), so guessing often scores correct.
- **Run.** One unattended `claude -p` session (Claude Code 2.1.292,
  default auto-compaction); batch and SOP in the first message ("one at a
  time"); the model never sees a budget. Caps: $25 (Haiku), $150 (Sonnet),
  ≤50 resumes, 6–12 h.
- **Conditions (Haiku 4.5, 8 seeds).** Information factor: Plain (SOP
  once); Sys (plus a copy in the system prompt, which compaction cannot
  remove); Remind (plus the SOP's decision section and critical-rules
  list verbatim, and an "unattended, one at a time, no need to stop,
  summarize, pause or rush" paragraph, attached to each account's first
  data result). Liveness factor: the `-loop` variants auto-resume up to
  50 times when the agent ends its turn with accounts left.
  **Remind-loop was designated the strongest baseline before any run.**
  FSM = Remind + the FSM. Ablations: Stop-only, Gates-only (3 seeds).
  There is no checklist arm in the main experiment.
- **Metric.** A decision is *compliant* if investigated (all 5 retrievals)
  ∧ recorded (STEP 1–4 record names the submitted action) ∧ focused (no
  other account's data in between). FC = #(compliant ∧ exactly correct),
  max 200. Strict accuracy is also reported.
- **Pre-registration.** Design, metric, predictions and analysis code
  frozen and committed before formal runs (seed 1 development only);
  paired bootstrap CI, sign and Wilcoxon tests; voiding rules for
  infrastructure failures written beforehand; deviations listed (App. A).
  Extension-model regime: Remind-loop compliance <80% = drift, ≥95% = none.
- **Three analyses (not pre-registered as a family):** SWE-bench
  Verified with a 4-step debugging skill (Sonnet 4.6); τ²-bench with
  Codex CLI + gpt-5.4-mini; staged protocols (AutoResearch classifier
  tuning, month-end close, DuckDB optimizer from
  Long-Horizon-Terminal-Bench).

## Results

**Main experiment (Table 1, means over seeds).**

| Model | Condition | FC | Compliance | Decided | Correct | Cost $ |
|---|---|---|---|---|---|---|
| Haiku 4.5 (8) | Plain | 9.4 | 11.9% | 143.8 | 100.0 | 0.96 |
| | Sys | 17.1 | 20.1% | 120.1 | 86.1 | 1.07 |
| | Remind | 14.6 | 48.6% | 53.0 | 40.6 | 0.65 |
| | Sys-loop | 40.9 | 28.9% | 200 | 144.0 | 2.21 |
| | Remind-loop | 48.4 | 27.8% | 200 | 138.9 | 3.01 |
| | **FSM** | **139.2** | **94.8%** | 200 | 146.5 | 10.64 |
| Sonnet 4.6 (3) | Plain | 108.3 | 66.8% | 200 | 170.3 | 20.94 |
| | Remind-loop | 132.7 | 66.7% | 200 | 197.0 | 20.89 |
| | FSM | 187.7 | 100% | 200 | 187.7 | 53.26 |
| Qwen3.8-flash (8) | Remind-loop | 173.4 | 98.1% | 200 | 176.2 | – |
| | FSM | 186.2 | 99.7% | 200 | 186.8 | – |

- **Haiku paired contrasts (Table 7).** FSM − Remind-loop FC +90.9
  [46.2, 130.2], 7/8 seeds, Wilcoxon p = 0.023, sign p = 0.070. FSM −
  best -loop baseline per seed +70.5 [27.4, 114.5]. FSM − Plain +129.9
  [104.1, 151.9], 8/8. **Correct: FSM − Remind-loop +7.6 [−16.1, 29.8],
  5/8, p = 0.48.**
- **Over the batch.** FSM held 31.9–37.6 FC per 50-account window. No
  baseline exceeds 12 per window after the first 50. Remind-loop falls
  from 29.9 to 5.4.
- **Sonnet 4.6 drifts by batching** (fetches many accounts before
  deciding). FSM − Remind-loop FC +55.0 [−15.0, 191.0], positive in 1/3
  seeds (+191, −15, −11). Correct −9.3 [−15.0, −2.0], 0/3 seeds. The
  pre-registered falsification rule triggered. With 3 seeds the smallest
  attainable p is 0.25.
- **Qwen3.8-flash does not drift** (98.1%), yet FSM − Remind-loop FC
  +12.9 [2.6, 28.5], 7/8, Wilcoxon p = 0.047. That is outside the ±10
  null band they predicted. +5.9 without one seed where Remind-loop
  interleaved 292 times. Correct +10.5 [1.2, 23.4].
- **Haiku + FSM vs Sonnet 4.6 (seeds 13–15).** vs plain Sonnet: FC +26.0
  [−40.0, 137.0], positive in **1/3** seeds (+137, −19, −40). vs
  Sonnet's best baseline: −59.0 [−75, −40], 0/3. Strict accuracy vs
  plain Sonnet: −32.7 [−43, −15]. Cost $7.93 vs $20.94 (38%).
- **Ablation (Table 2, Haiku seeds 2–4).** Remind-loop FC 76.3. Stop-only
  5.7: the agent cannot stop, so it decides without looking (108.3
  blind decisions per session; one session overflowed its context after
  fetching 160 accounts in one turn). Gates-only 104.0 at 93.0%
  compliance, but 44.7 attempts to end the turn per session, and 2/3
  sessions exhausted the 50 resumes. Both together: 128.0.
- **Escape routes (Table 8, sessions out of 8).** Remind ended its turn
  with accounts left 8/8. Remind-loop interleaved accounts 7/8 (≈100 per
  session) and decided without looking 3/8. FSM took the one route its
  static permissions leave open, submitting for an account other than the
  one just recorded, in 3/8 sessions (40, 40 and 4 decisions). This
  failed pre-registered P3's ≥95%-every-seed condition.
- **Self-imposed budget pressure.** Agents cite a token budget they were
  never given; FSM agents did so *earlier* (median 13.5 decided accounts
  vs 26.5 for Remind-loop), and in the first 50 accounts FSM accuracy on
  fully investigated accounts was 77.5% vs 85.4%.
- **SWE-bench Verified (Sonnet 4.6, one run/task).** Hard-100 (heuristic
  selection): 69 vs 62, 7/0 discordant, McNemar p = 0.016. Easy-100
  (random): 87 vs 86; plain already followed the procedure in 97/98.
  +23% time, $1.30 → $1.73.
- **τ²-bench (gpt-5.4-mini, frozen first evaluation).** Telecom 30/40 vs
  13/40 (17/0, p = 1.5×10⁻⁵; control failures were mostly hand-offs);
  Airline 17/17, Banking 6/6, Retail 9 vs 15 (over-strict guards). **The
  control had the same static checklist**, but the FSM controller emits
  62% of turns. Revised FSMs (13/17/22 iterations) are descriptive.
- **Staged protocols.** AutoResearch: valid ledger rows 100/100 vs 48/100,
  same best accuracy (99.33%); with helper scripts for both, 7/10 vs 6/10
  complete. Month-end close: 11/12 vs 8/12 at 40 turns, 1/6 vs 3/6 at 24.
  DuckDB: reward 0.75 vs 0.19 (4 pairs, after 28 design versions). **On
  other LHTB tasks FSM variants did not beat controls; a pre-registered
  ten-pair data-validation audit gave −0.009 against a checklist control.**
- **Authoring cost (Table 11).** 3–10 states, 61–160 YAML lines, 156–1,214
  lines of guard/check code per task; τ² Telecom is a ~1,300-line adapter.
  The main YAML is generated from the SOP (119-line script). Authoring
  time not recorded.

## Critique / open questions

- **FC is largely the FSM's own guard predicate.** Compliance is
  investigated ∧ recorded ∧ focused. The s1→s2 guard enforces
  "investigated" and "focused" by construction, and the record tool
  enforces "recorded". FSM's 94.8% compliance is therefore close to
  definitional. The non-trivial findings are that the agent keeps
  deciding all 200 accounts under the gates and that accuracy does not
  fall. **The outcome metric that does not share the guard, strict
  accuracy, shows no significant Haiku gain** (+7.6, CI spans 0). It was
  significantly *worse* for Sonnet (−9.3) and much worse than plain
  Sonnet (−32.7). The paper says this ("enforces how the agent works,
  not what it decides"), but the headline 139 vs 48 reads as a quality
  gain, and it is a fidelity gain. With 67% *No Action* labels, blind
  decisions often score correct, which the procedural metric rightly
  penalises but a user may not.
- **"Strongest baseline" means restated instructions plus resumption,
  not a checklist.** Remind-loop re-attaches the SOP's decision section
  and critical rules once per account, plus up to 50 auto-resumes. It
  has no externalised progress ledger, no todo list, no per-account
  verification step and no hook. Where a checklist control did exist, the
  results split. τ² Telecom: the FSM won, but the FSM controller also
  emits 62% of the turns. Data-validation audit (the only pre-registered
  ten-pair confirmation outside the main task): −0.009, a null. So the
  paper does not show that code beats a *good* prose scaffold. It shows
  that code beats restatement.
- **Per-task hand-authoring is substantial and its cost is unmeasured.**
  156–1,214 lines of guard code per workflow, a 1,300-line adapter for
  Telecom, and 13–28 iterations in the analysis domains. The successful
  τ² and DuckDB designs were revised after seeing results, so those
  numbers are descriptive. "Mostly transcription" holds for the YAML,
  not the guards, and the authors' own lesson ("check less, not more")
  came from over-strict guards that lost Retail tasks.
- **Cost is not small:** Haiku FSM $10.64 vs $3.01 (3.5×); Sonnet $53.26
  vs $20.89. "38% of Sonnet's cost" is vs plain Sonnet, 3 seeds, 1/3 positive.
- **The stop gate's message is controlled; its timing is not isolated
  cleanly.** The stop-gate text and the -loop resume text come from one
  function, so the content is matched. That is close to the sham control
  [[literature/papers/agarwal2026fire]] used in
  [[concepts/evidence-gated-completion]], which found the message is the
  active ingredient. Here the difference is *when* it arrives: a blocked
  stop inside the turn vs a resume after the turn has ended. Gates-only
  (external resume) vs FSM (stop gate) is 104.0 vs 128.0 FC on 3 seeds,
  with no CI.
- **One runtime, one main task**, Sonnet on 3 seeds; Hard-100's model
  version was not preserved.
- **The open route was found** (3/8 sessions): a hand-written FSM leaves
  gaps a long run will find.
- **Not shown:** auto-generated workflows without hand-tuned guards;
  adversarial or goal-conflicting instructions; models newer than Sonnet
  4.6; whether a todo/ledger tool *without* gates gets most of the
  benefit (the AutoResearch helper-script result hints it might).

## Trust signals

- **Credibility:** 3 — Zhejiang University with Oxford and Xidian. arXiv
  preprint, not peer-reviewed, 10 days old, no citations. The main
  experiment is unusually careful: pre-registered and frozen, the
  strongest baseline designated in advance, failed predictions and a
  triggered falsification rule reported (P3, Sonnet), voiding rules for
  infrastructure failures, and full escape-route counts. Code, workflows,
  pre-registration records and session logs are promised ("we will
  release") but no URL is given. The analyses are post hoc, often one run
  per task, and in places revised after seeing results.
- Negatives reported (Sonnet accuracy loss, the checklist null, the
  24-turn reversal). AI-assisted drafting disclosed.

## Follow-up

- **Relevance:** 5 — the canonical controlled evidence for "a rule that
  must hold goes in code" on the harness this box runs. The setup is
  Claude Code, Claude models and `PreToolUse`/`Stop` hooks, with a
  pre-registered prose-vs-code comparison and an ablation that assigns
  each failure route to an enforcement point. It changes
  [[concepts/typed-enforcement]], [[concepts/enforcement-boundary-placement]]
  and [[concepts/constraint-pinning]]. Read the scope with care: it
  establishes *procedural fidelity*, not better answers. The baseline it
  beats is restatement, not a checklist.
- **For this box (our inference).** `/curate` and `/digest` under
  `agency: max` are prose procedures run in long unattended sessions.
  The paper's three escape routes have direct analogues:
  - **stopping early:** draining part of a backlog and ending the turn;
  - **batching:** fetching many items, then ingesting them together;
  - **deciding without looking:** dispositions without reading the source.

  This box's `Stop` hook is a token logger, and there is no `PreToolUse`
  procedure gate. The cheap transfer is a Stop-hook check over files
  (e.g. "uncurated items remain and the verdict is not `hold`"), plus a
  per-item record check before archive. Not a full FSM. The paper's own
  data warns that the stop gate alone made Haiku decide blind (Stop-only
  FC 5.7), so a stop gate needs a matching per-item evidence check.
  Candidate for `/elevate`.
- **Self-invented budget pressure.** Agents cite a "token budget" they
  were never shown, and the FSM did not remove it (it appeared earlier).
  This bears on `agency.md` wording and the headroom verdict. Telling an
  agent there is headroom may not stop it rationing.
- **Candidates** (cited, not yet in the graph):
  - Zhang et al. 2026, "Formal Skill" (arXiv 2605.19604): skills as
    executable state machines with hook policies; closest prior system.
  - Xie et al. 2026, constant-context skill learning (arXiv 2605.05413).
  - Nandi et al. 2026, SOP-Bench (KDD 2026); Li et al. 2026,
    Long-Horizon-Terminal-Bench (arXiv 2607.08964).
  - Wang, Poskitt & Sun 2026, AgentSpec (ICSE 2026), cited in other
    notes, no note of its own.
- Already in graph: [[literature/papers/badhe2026skill]] (SKILL.state,
  explicit mutable execution state). This paper holds the procedure fixed
  and asks about fidelity, where SKILL.state optimizes efficiency.
