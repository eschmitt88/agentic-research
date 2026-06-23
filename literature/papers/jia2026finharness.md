---
kind: paper
title: "FinHarness: An Inline Lifecycle Safety Harness for Finance LLM Agents"
authors: ["Haoxuan Jia", "Yang Liu", "Bin Chong", "Yingguang Yang", "Yancheng Chen", "Jiayu Liang", "Qian Li", "Hanning Lu", "Kefu Xu", "Hao Zheng", "Chongyang Zhang", "Hao Peng", "Philip S. Yu"]
institutions: ["Peking University", "Nanyang Technological University", "Tsinghua University", "University of Science and Technology of China", "University of Chinese Academy of Sciences", "Soochow University", "Beijing University of Posts and Telecommunications", "University of Leeds", "Fullive Innovation (Beijing) AI Technology Co., Ltd.", "Beihang University", "University of Illinois Chicago"]
year: 2026
venue: arXiv
peer_reviewed: false
url: https://arxiv.org/abs/2605.27333
code_url: null
citations: null
source: "raw/papers/jia2026finharness.pdf"
added: "2026-06-23"
relevance: 4
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/budget-as-ceiling]]"
tags: [safety, guardrail, permission-gate, inline-monitoring, prompt-injection, finance, risk-cumulant, llm-judge, escalation]
---

# FinHarness: An Inline Lifecycle Safety Harness for Finance LLM Agents

## TL;DR

FinHarness is a safety **harness** that wraps a finance LLM agent
end-to-end and intercepts every prospective tool call *before* execution.
It fuses three components — a **Query Monitor** (single-turn intent + a
cross-turn drift signal accumulated into a session-level risk cumulant), a
**Tool Monitor** (deterministic permission / parameter / sequence priors
per call), and a **Cascade** (a sliding-window risk sum that adaptively
routes verification to a cheap or advanced LLM judge). Its sharp framing:
the harness is a **regulator over the agent's policy**, feeding fired risk
factors back into the agent input as *ex-ante evidence* so the agent can
refuse / re-plan / escalate on its own — not merely a gate positioned in
front of the agent. On FinVault it cuts attack success rate (ASR) from
38.3% to 15.0% while largely preserving benign approval (41.1% → 39.3%),
using 4.7x fewer advanced-judge calls than an always-advanced baseline.

## Claims

- A finance agent rarely fails on one bad turn; it fails because a long,
  plausible trajectory ends in an *irreversible* transfer or a leaked KYC
  record. Once a prompt-injected instruction passes the input boundary,
  every downstream tool call is a privileged operation where existing
  safeguards have no visibility.
- The two dominant deployment patterns both miss the loop: **boundary
  filters** (input/output guards) are stateless and blind to the tool-call
  series in between; **post-hoc LLM judges** intervene too late (the wire
  transfer has cleared by audit time) and cost grows linearly with trace
  length. The deeper problem is *positional* — both treat protection as an
  external supervisor observing a black-box agent.
- The fix is to operate **inline** within the execution loop, observing
  every intermediate state and feeding safety signals back into the
  agent's subsequent decisions — making the safeguard a regulator over the
  policy rather than a gate in front of it.
- A persistent weak signal defeats per-step judging: a five-step
  semantic-obfuscation attack keeps every per-step score at 0.22 (below
  any decisive band), so a per-step judge approves each step, yet the
  *cumulative* risk rises monotonically and crosses the escalation
  threshold — only an integrate-across-steps safeguard redirects the agent
  at the right moment.

## Methods

- **Problem setting.** Agent is a policy π selecting tool actions over
  history; a subset of tools `T_irrev` commit irreversible state (transfers,
  contract signing). Adversary controls user input (direct injection) or
  retrieved content (indirect), aiming to steer π into an unauthorized
  irreversible call. The harness `H` wraps π and, per proposed action,
  approves / blocks / injects advisory evidence.
- **Query Monitor (no LLM calls).** Deterministic compliance priors at each
  user turn. A **single-turn** head fuses verb-tier intent `Q1`, amount
  magnitude `Q2`, risk-product flag `Q3`, coercion/urgency `Q4`, injection
  lexicon `Q5` (manually specified ordinal severity priors, not fitted
  parameters). A **cross-turn drift** head compares the turn against history
  (`D1` false reference to unsupported entity, `D2` pseudo test/debug mode,
  `D3` phantom approval code, `D4` verb-tier jump, `D5` closing push). These
  combine into a **session-level risk cumulant** `C_query` with gravity
  decay (decay 0.7 normally; `γ=1` — non-decaying — once any *structural*
  drift `D1/D2/D3` has ever fired, so elevated risk persists). `C_query` maps
  to a four-band advisory: UNSAFE (≥0.5), UNCERTAIN ([0.25,0.5)), SAFE
  (≤0.1), and a deliberate dead-zone `⊥` ((0.1,0.25)). Advisories are
  rendered into the agent's system prompt as *evidence*, not verdicts.
- **Tool Monitor.** Before each prospective tool call, five heads `H1–H5`:
  permission-tier prior (per the FinVault tool registry), dangerous-parameter
  match, argument type/range anomaly, business-fact discrepancy (over prior
  observations), and tool-sequence anomaly (e.g. critical write without prior
  verification). Tool risk `c_tool = max(H1..H5)`. Query and tool risk fuse
  conservatively: per-step risk `s_t = max(C_query, c_tool)` — either a risky
  intent *or* a risky action suffices.
- **Cascade.** A **Risk Window** sums the last `W=5` step scores; when the
  sum exceeds θ=1.0 it routes to an advanced judge (gpt-4o), otherwise to a
  cheap judge (gpt-4o-mini) — the two tiers are mutually exclusive, so at
  most one LLM call per step. A **Selective Episodic Memory** retrieves at
  most two prior steps (via risk-salience and a collusion / entity-overlap
  signal) for *bounded* judge context — independent of trajectory length. The
  judge returns SAFE → APPROVE, UNSAFE → hard BLOCK, UNCERTAIN → advisory
  feedback only.
- **Fired-Signal Dynamic Injection.** At every step the fired rule heads from
  all three components are re-written into the agent input as structured
  evidence (e.g. `[turn k fired: Q4_coercion(0.85), D1_false_reference(0.85)]`)
  across three contextual zones, enabling agent-initiated self-rejection. The
  judge is told *which* priors fired and at what magnitude, leaving room to
  override false positives.

## Results

- **FinVault** (107 benign + 107 attack, 31 scenarios, attack families
  V1–V4). Routed FinHarness: **ASR 38.3% → 15.0%**, Approve 41.1% → 39.3%,
  Net (Approve − ASR) +24.3. The always-advanced ablation FinHarness-AA
  reaches ASR 8.4% / Approve 37.4% (best descriptive Net +29.0) but uses
  646 advanced calls vs the routed config's 138 (**4.7x fewer**).
- FinHarness variants are the only configurations combining sub-20% ASR with
  benign Approve ≥37%. The B6/B7 external-guardrail adaptations (GuardAgent /
  InferAct) drive ASR to 2.8%/0.9% but **collapse benign Approve to 8.4%** —
  out of the deployable operating set.
- **Loop-coupling signature.** From the B2 per-step cheap judge to
  FinHarness, hard-stops drop (40→26) while agent *self-rejections* rise
  (9→32) and escalations rise (15→22) — interception shifts from the harness
  hard-stopping to the agent refusing on its own, the intended effect of
  feeding evidence back.
- On the 856-trace all-attack synthesis set, FinHarness reaches the lowest
  ASR (20.1% vs 22.1–24.1% for per-step judges). Largest per-family gains:
  emotional manipulation (−43.8 pp), document fabrication (−25.0); direct
  injection *regresses* (+16.7).

## Critique / open questions

- **No code released** (the note records `code_url: null`); the deterministic
  priors, severity numbers (e.g. `Q1 ∈ {0,0.10,0.30,0.55,0.80}`), window
  `W=5`, and θ=1.0 are stated as frozen, manually-specified constants —
  reproducibility and transferability off FinVault are untested.
- The authors concede the evaluation is **single-run and benchmark-bound**:
  no variance across agent prompts, model versions, seeds, or tool
  simulators; significance checks only for the main B2/B3 comparisons.
- **Adaptive-attacker fragility, admitted:** results "should not be read as
  adaptive robustness against an attacker who knows the risk heads, window
  length, threshold, or injection format." Several single-step syntactic
  families *regress* relative to the cheap per-step judge — a deployable
  harness would need a fast-reject path before the advanced tier.
- The manually-specified ordinal priors encode FinVault's permission schema
  and compliance taxonomy; whether the same severity ladder generalizes to
  another tool registry (or another domain) is unattested — the "regulator"
  is hand-tuned to one benchmark's ontology.
- Judges (gpt-4o / gpt-4o-mini) are proprietary and frozen; the cost claim is
  about advanced-tier *call count*, not end-to-end token billing.

## Trust signals

- **Credibility:** 3 — large multi-institution Chinese-academia collaboration
  (Peking, Tsinghua, USTC, Beihang, NTU) with **Philip S. Yu** (one of the
  most-cited authors in data mining) as senior author, which lifts it above a
  no-name preprint. But it is an **arXiv v1 preprint (not peer-reviewed)**,
  **releases no code or artifacts**, and is **single-run / benchmark-bound on
  one safety benchmark (FinVault)** with self-admitted gaps in adaptive
  robustness and variance estimation. The named-senior-author signal pulls up;
  the no-artifact, single-run, preprint status pulls down — net 3.

## Follow-up

- **Relevance:** 4 — a concrete fourth-domain (finance) attestation that
  completes the seed for [[concepts/permission-gate-as-architecture]]: safety
  is positioned *inline within the execution loop* as a per-tool-call
  permission/parameter/sequence gate (the Tool Monitor's `H1` permission-tier
  prior over a typed tool registry is a literal permission gate), and the
  paper's sharpest contribution — framing the harness as a **regulator over
  the agent's policy** that injects fired-rule evidence back into the agent
  rather than acting as an external supervisor — is exactly the architectural
  framing that distinguishes a gate-as-architecture from a bolted-on filter.
  This is enough to crystallize the concept. Held at 4 (not 5) because the
  mechanism is hand-tuned to one benchmark's permission ontology and the
  concept is being newly seeded rather than already load-bearing. The
  session-level **risk cumulant + fixed escalation threshold θ=1.0** is a
  genuine structural analog of [[concepts/budget-as-ceiling]]: a monotonically
  accumulating quantity crosses an explicit, project-frozen ceiling to trigger
  a deterministic halt/escalation action — not a heuristic deciding the work
  "looks risky" — the same accumulator-vs-ceiling shape as a token/wall-clock
  budget, applied to risk rather than spend.
