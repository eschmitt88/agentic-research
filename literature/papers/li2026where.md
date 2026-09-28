---
kind: paper
title: "Where Does Exactly-Once Live? Model, Harness, and Tool-Contract Effects on Duplicate Side Effects in LLM Agents"
authors: ["Jiapeng Li"]
institutions: ["Microsoft"]
year: 2026
venue: "arXiv (cs.LG)"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.29095"
code_url: null
citations: null
source: "raw/papers/li2026where.pdf"
added: "2026-09-28"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/evidence-gated-completion]]"
  - "[[concepts/typed-enforcement]]"
tags: ["exactly-once", "idempotency", "side-effects", "tool-contract", "harness", "mcp", "fault-injection", "effect-ledger", "overclaim", "preregistered", "variance-decomposition", "placement"]
---

# Where Does Exactly-Once Live? Model, Harness, and Tool-Contract Effects on Duplicate Side Effects in LLM Agents

## TL;DR

A factorial experiment on one placement question: when a tool write times
out or errors, should exactly-once behaviour be enforced in the **model**,
the **harness**, or the **tool contract**? **Limbo** has six simulated
services and twelve fault modes injected at the service boundary, and it
grades every episode against a ledger of committed effects. Across
**25,930** episodes (nine models, a minimal scaffold plus three production
harnesses, two contracts, fifteen recovery conditions), the answer splits
by whether the hidden state is **observable from where the model stands**.
If an immediate read-back can reveal it (lost ack, misleading 500, partial
batch), frontier models verify first and almost never duplicate (**0.5%**
on lost acks), and the model takes **53%** of the explained variance. If it
cannot (a request still in flight, or a redelivered request), the same
frontier models duplicate in **56%** and **74%** of episodes, and the
contract takes **81%**. A proposition proves that no verification-only
policy is exactly-once under late commits without a known in-flight bound.
Offering an idempotency key on every write brings duplicates from **28% to
4%**. Three production harnesses behave like the minimal scaffold. Agents
reported `completed` in **90%** of the episodes where they had duplicated
an effect.

## Claims

- **Placement is decided by observability, not by capability.** "When an
  immediate read-back can reveal what happened, it lives in the model …
  When no read-back can—a request still in flight, a request delivered
  twice—it lives in the tool contract."
- **Verification alone is provably insufficient under late commits.**
  Proposition 1: if a policy completes the task in the world where the
  request never executed within time T, then it duplicates with probability
  one in every world where the request lands after T + λ. A known bound Δ
  restores a verify-only solution at a latency cost of Δ + λ per ambiguous
  write. "few APIs document such a bound, none of our tools do."
- **Keys suffice (Proposition 2).** "re-issue with the same key until
  acknowledged" is exactly-once in all five outcome states.
- **Agents use keys when offered.** Frontier models attach a key to a
  keyable write in 98% of committed-fault episodes without being told to.
  "Agents use keys when offered; they cannot use keys that do not exist."
- **Client-side middleware has a ceiling set by the contract.** With the
  native contract, even an outcome oracle that sees in-flight requests
  reaches only 87% EOS: "95% of the 105 episodes in which it still produced
  a duplicate were redelivery on writes without key support, which no
  client can prevent."
- **Transparent retries below the model are harmful.** "Harness-level
  retries sit below the model and are invisible to it". sdk-retry drops
  EOS from 72% to 50%, "because every lost acknowledgement becomes a
  duplicate before the model sees anything."
- **A safety layer can displace the model's own caution.** For gpt-6-sol
  under late commits, the guard's annotation that it would verify before
  any identical retry shifted escalation from 12% to 3% and
  verify-then-retry from 59% to 88%. Late-commit duplicates rose from 50%
  to 71%. "A safety layer that promises verification can displace the
  model's own, more conservative strategies."
- **A stale read path is worse than none.** "a read-back path that cannot
  see lagging or in-flight effects was worse than no read-back path at
  all, because agents trusted it instead of escalating."
- **Agents do not know when they duplicated.** 90% of duplicate-producing
  episodes ended `completed`, and 80% also listed no uncertain operation.
  This happened even though the `finish` tool had an explicit
  uncertain-operations field.
- **Normative ask to protocol designers.** MCP's idempotency declaration is
  "only as an advisory hint". The paper proposes "three normative
  additions: a standard idempotency-key argument for non-idempotent tools,
  a declared read-back (status) operation for every write, and documented
  visibility and in-flight bounds."

## Methods

- **Limbo.** Six services (social, billing, tickets, mail, data, deploy)
  with realistic contracts: optional Stripe-style idempotency keys (native
  contract: only billing and one social platform), eventually consistent
  read paths with documented lag (120–180 s), one endpoint with no
  read-back, and naturally idempotent operations. Twelve task templates
  cover ops workflows, from a two-platform release announcement to a
  ~10-write hotfix rollout. Tools are `wait`, `escalate_to_human` (a
  simulated operator who sees ground truth and answers after 15 simulated
  minutes) and `finish` (status + uncertain-operations list).
- **Twelve fault modes**, attached deterministically to the n-th call
  matching a focal write, so every model, harness and condition faces the
  identical world. Observation-equivalent groups are byte-identical in
  response. The new modes are: `timeout_late` (commit lands 90 s later),
  `timeout_late_tail` (log-uniform 40 s–2 h), `duplicate_delivery`
  (success response, executed twice) and `partial_timeout`.
- **Grader** reads only the committed-effects ledger and final state, with
  no LLM judge. It reports TS, EOS, duplicates (including compensated
  ones), collateral damage, unrequested writes and **overclaim** (finished
  `completed` although the task failed or a duplicate remains). A
  deterministic coder classifies the recovery behaviour after the focal
  fault.
- **Validity.** Thirty-one scripted-agent unit tests check fault semantics.
  Observation equivalence is checked empirically: first action after the
  timeout agreed in 87% of matched committed/not-committed pairs, versus
  82% across independent runs of the same world (TVD 0.07 = exchangeable
  baseline, permutation p ≥ 0.15 for every model). E1/E2 overlap
  reproducibility: 96% agreement, κ = 0.90 on 868 episodes.
- **Harness-agnostic via MCP.** A stdio MCP server forwards only
  `tools/list` and `tools/call`. Fault plan and ground truth stay in the
  runner. Harness recovery policies (vbr, guard, wait-Δ) run *inside the
  MCP server*, so they transfer to any harness unchanged.
- **Harnesses.** GitHub Copilot CLI 1.0.86, Hermes Agent 0.20.6, OpenAI
  Codex CLI 0.139, plus a minimal function-calling scaffold (≤40 calls).
  **Claude Code is not among them**, although claude-opus-5.5 is run under
  Copilot CLI and Hermes. Harnesses were "restrict[ed] … to the sandbox's
  tools" with a shared preamble telling them "Do not use shell, file, web
  or any other tools, and do not write code."
- **Models (via one gateway, provider defaults):** gpt-6-astra, gpt-6-sol,
  gpt-5.6-sol, gpt-5.4-mini, gpt-4.1, claude-opus-5.5, gemini-3.8-flash,
  grok-4.7, mai-code-1.1-flash. gpt-4.1 completed only 87% of E1 and is
  excluded from pooled statistics.
- **Experiments.** E1 is models × faults (7,484 episodes). E2/E2k is four
  models × nine conditions, plus the keys-everywhere contract (9,548). E3/E3k
  is harnesses (3,968). E4 is robustness (1,928). E5 is wait-then-verify
  vs keys (1,904). E6 is the "exactly once" cue ablation (1,098).
- **Recovery conditions.** vanilla, aware (five reliability sentences),
  reflect, sdk-retry (3× backoff on timeout/5xx/429), rules, vbr
  (verify-before-retry, after Mansoor et al.), wait-Δ, and guard (vbr +
  auto keys + re-verify after lag + block unverifiable repeats with an
  escalation request + an "outcome unknown" annotation; "under two hundred
  lines"). There are two ground-truth reference conditions: a state oracle
  and an outcome oracle.
- **Statistics.** Paired worlds. Wilson intervals on Kish effective sample
  size, clustered by template (only twelve clusters). A Bayesian mixed
  model with GEE (Mancl–DeRouen) as a frequentist check. Exact McNemar with
  Holm, TOST for H4. Shapley decomposition of McFadden pseudo-R².
  **Preregistered before E1**, with deviations listed. E2k, E3k, E5, E6,
  the outcome oracle, the *stratified decomposition* and the revised
  interval method are post hoc and labelled exploratory.

## Results

- **RQ1, E1 (Table 3, Fig. 2).** Frontier models duplicate 0.5% of
  lost-ack writes and 8% after a misleading 500. The two weaker models
  (mai-code-1.1-flash, gpt-5.4-mini) duplicate 18% and 47%. Late commit:
  frontier models duplicate 25–72% by model (56% pooled). Redelivery: 74–75%
  for every model except gpt-4.1. "Not executed" worlds duplicate 0.6% with
  99% TS, explicit faults 0.4% and fault-free 0.0%, so caution is not being
  rewarded for its own sake.
- **How late commits defeat careful agents.** Of 297 late-commit episodes
  re-issued within a minute, 69% duplicated. **All 123 agents that waited
  ≥60 s before re-issuing duplicated (100%)**. They waited the documented
  *visibility* lag from the error rather than from the unknown commit
  time. Agents that never re-issued were exactly-once in all but 1% of 84.
- **RQ2, variance decomposition (Table 4).** On faults a read-back resolves
  (E1): model 53%, contract 30%, fault mode 18% (pseudo-R² 0.51). On faults
  it cannot resolve: contract 81%, mode 10%, model 8% (pseudo-R² 0.72). The
  *preregistered pooled* analysis gives contract 36%, mode 56% and **model
  9%**. In E3 (all frontier models) the resolvable split is model 39% vs
  contract 42%, and the harness share is 3% resolvable, 0% unresolvable, 0%
  pooled. The preregistered ≥10% harness share is not supported.
- **Contract class (Fig. 3).** Frontier duplicates: keyable 1.5%,
  idempotent 0.0%, non-idempotent/strong 28%, non-idempotent/eventual 31%.
  Mixed-effects OR 6.4 (5.2–7.9) for non-idempotent with an eventual or
  missing read path. The GEE check gives a much weaker 2.9 (1.4–6.0),
  p = 0.007.
- **Stale read paths.** Writes with no read-back were duplicated at 13%
  because frontier agents escalated in 87% of those episodes. With a read
  path they verified (96% strong, 89% eventual) and trusted it. Among
  lost-ack faults verified before re-issue, duplicates were 13.4% on
  eventual vs 0.8% on strong paths (n = 947, GEE p = 0.005).
- **Contract manipulation (Table 5).** Under keys-everywhere without the
  guard, vanilla agents attach keys in 98% of faulted episodes. Late-commit
  duplicates fall 61% → 9% and redelivery 74% → 7%. With the guard: 68% → 7%
  and 74% → 0%, and EOS 99%. The headline **"28% to 4%"** appears only in
  the abstract and introduction. No table reports that pair directly; it
  is presumably the E2/E2k pooled rate.
- **Waiting vs keys (E5, Table 6).** Fixed 90 s delay: wait-120 s reaches
  99% EOS at 3.8 simulated min, which beats keys+guard (93%). Heavy tail:
  wait-1 h reaches only 84% EOS at 49.9 min, because a one-hour bound
  covers the delay in only 82% of worlds. Keys+guard reach 94% at 1.5 min.
  All 432 same-key late-commit re-issues produced 0% duplicates. The
  residual came from 56 re-issues whose first attempt had no key (68%
  duplicated) and 2 where the agent minted a new key such as `-retry1`.
- **RQ3, overclaim.** Of 1,279 duplicate-producing E1 episodes, 90%
  finished `completed` (95% CI 78–96%, wide because of Kish clustering).
  80% listed no uncertain operation. No stakes sensitivity: irreversible
  writes were blindly re-issued in 9.6% of episodes vs 6.3% for reversible
  ones (TOST inconclusive).
- **RQ4, mitigations (Table 7).** For frontier models the prompt- and
  harness-level mitigations barely move EOS (claude-opus-5.5: 77% vanilla,
  77% guard). The guard helps the weakest model (mai-code-1.1-flash
  59% → 76%). Paired over the four models it gains +4.0 pp EOS over vanilla (Holm
  p = 0.00011). The state oracle reaches ~80% and the outcome oracle 87%.
- **Harnesses (Tables 8–9).** Native duplicate rate: Copilot CLI 27%,
  Hermes 29%, Codex CLI 30%, minimal 26%. With the guard in the MCP server:
  28/29/28/29%. Under keys-everywhere with gpt-5.6-sol, every harness goes
  to 0% duplicates with or without the guard, and to 100% EOS with keys +
  guard. Costs differ: Codex CLI used ~153k tokens per episode against 12k
  for the minimal scaffold.
- **Robustness.** Removing the "exactly once" cue raises resolvable-fault
  duplicates 12% → 22% (mostly in weaker models, 29% → 51%) and late-commit
  duplicates 58% → 71%. Removing consistency statements from tool
  descriptions raises duplicates 22% → 34%. An explicit "not idempotent"
  warning barely matters (22% → 24%). Guard ablations all fall within
  35–39%.

## Critique / open questions

- **The headline split is exploratory; the preregistered analysis says
  something weaker.** The 53%/81% stratified decomposition was "added
  afterwards". The preregistered pooled analysis attributes only 9% to the
  model and 56% to fault stage. The stratification is well motivated by
  Propositions 1–2, and the authors flag it honestly, but the abstract
  leads with the post-hoc numbers.
- **"The harness barely matters" is narrower than it reads.** Every
  harness was stripped to the MCP sandbox's tools with a shared preamble
  that forbade shell, files, web and code. The authors say this "removes
  some sources of harness variation by design". Only three harnesses were
  tested, all run non-interactively, and Claude Code was not among them.
  What was measured is that *harness scaffolding and system prompts* do not
  change recovery behaviour on MCP-only tool use. Harness *retry* behaviour
  does matter a lot, but it was measured through the sdk-retry condition,
  not through the production harnesses.
- **Simulated services, simulated operator.** The operator "always answers
  truthfully when available". The E5 success rates depend on the chosen
  delay distribution (the qualitative result does not).
- **Internal inconsistencies worth noting.** The heavy-tail median is "about
  9 minutes" in §4 but "median 6 min" in §6.3. Log-uniform over 40 s–2 h
  gives ≈9 min, so §6.3 is wrong. The introduction's "the duplicate rate
  of a shared model is at most 0% even without the guard, and exactly-once
  success is at least 100% with it" reads like a template left unfilled.
  The paper says "We release the benchmark, the harness adapters and every
  episode trace" and the conclusion says "available", but the footnote
  says "Code and data will be made available upon publication." **Nothing
  is released yet.** The manuscript and benchmark code were partly drafted
  with GitHub Copilot (disclosed).
- **Possible vendor interest, not visible in results.** The author is at
  Microsoft, and both GitHub Copilot CLI and mai-code-1.1-flash are
  Microsoft products. Neither is favoured: Copilot CLI matches the others,
  and mai-code-1.1-flash is one of the two weakest models.
- **The contract result is close to a theorem, which is its strength and
  its limit.** Proposition 2 guarantees keys work when reused. The
  empirical content is that models *do* reuse them (98% attach, 432/432
  same-key re-issues clean). The interesting residual is agent-minted
  retry keys, which motivates a harness that "pin[s] one key per intent".
- **Relation to its concurrent work.** It reproduces Mansoor et al.'s
  verify-before-retry wrapper as a baseline and adopts Sun (2026)'s
  counterfactual pairs. It corroborates IdempotencyBench's scripted-agent
  finding that keys protect only when stable across retries. None of the
  three is in this graph.

## Trust signals

- **Credibility:** 3. For: a single author at Microsoft; an unusually
  disciplined design (preregistered hypotheses with deviations listed,
  paired deterministic worlds, observation-equivalence checked empirically,
  31 unit tests on fault semantics, clustered intervals on an honest
  effective sample size, a GEE check that visibly shrinks the headline OR
  from 6.4 to 2.9, and two unsupported hypotheses reported as such).
  Against, each of which holds it below 4: no peer review, no released code
  or traces yet (the text contradicts itself on this), the headline
  stratified decomposition is post hoc, all services are simulated, a
  single author with no second analyst, and several internal numeric slips
  (the 6 vs 9 min median, the unfilled "at most 0% / at least 100%"
  sentence, and a "28%→4%" headline not traceable to a table).

## Follow-up

- **Relevance:** 4. This is the head-to-head placement comparison
  [[concepts/enforcement-boundary-placement]] asked for. It varies model,
  harness and contract in one factorial and supplies a criterion, the
  observability of the hidden state from the enforcing component, that the
  concept did not have. It is a 4 and not a 5 because the domain is
  operational tool writes rather than research agents, and because the
  harness null covers MCP-only use of three non-Claude harnesses.
- **[[concepts/enforcement-boundary-placement]]:** a fourth head-to-head
  comparison, and the first to include the model itself as a candidate
  placement. It yields an exception to the "enforcement cannot live in the
  model" consensus, scoped by observability. It also corroborates design
  rule 3 with a proof: the key is evaluated at the resource, inside the
  effect's application.
- **[[concepts/evidence-gated-completion]]:** a measured overclaim rate on
  side effects (90% `completed`, 80% with an empty uncertainty field) that
  a ledger-reading grader catches. There is also a guidance-3 datum: a
  lagging read path is worse than none. It does **not** discharge the
  concept's hold, because the guard is an action gate and its block
  component moves duplicates by ~2 pp.
- **[[concepts/typed-enforcement]]:** MCP's idempotency annotation is
  advisory, and a <200-line guard driven only by a machine-readable tool
  contract transfers across harnesses unchanged. The guard backfire is a
  second instance of elkoussy2026agentltl's "enforcement can hurt strong
  models", with a named mechanism: displacement of the model's own
  escalation.
- **For this repo.** The main external write the skills perform on their
  own is `git push` after an auto-commit, which is naturally idempotent.
  `gh` issue or PR creation would not be. The more
  relevant import is the overclaim finding: a skill's own `completed` is
  uninformative about side effects, which is this graph's standing
  argument for gating on the log/commit ledger.
- **Candidates.** Barbaste et al. 2026 (arXiv 2609.00006), a source-code
  study of eleven coding-agent harnesses, bears directly on
  harness-engineering concepts. Also Mansoor et al. 2026 (arXiv
  2608.02645) and Tang & Zhan 2026 (TMLR, budgeted verification by
  downstream harm).
