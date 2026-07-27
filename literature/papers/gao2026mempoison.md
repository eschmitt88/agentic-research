---
kind: paper
title: "MemPoison: Uncovering Persistent Memory Threats and Structural Blind Spots in LLM Agents"
authors: ["Jifeng Gao", "Kang Xia", "Yi Zhang", "Xiaobin Hong", "Mingkai Lin", "Xingshen Wei", "Wenzhong Li", "Sanglu Lu"]
institutions: ["State Key Laboratory for Novel Software Technology, Nanjing University", "NARI Group Corporation / State Grid Electric Power Research Institute"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2607.14651"
code_url: null
citations: null
source: "raw/papers/gao2026mempoison.pdf"
added: "2026-07-27"
relevance: 5
credibility: 4
status: read
related_concepts:
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/selective-memory-retrieval]]"
  - "[[concepts/multi-granularity-memory]]"
related_experiments: []
tags: [memory, write-policy, security, memory-poisoning, benchmark, compositional-attack, mechanistic-analysis]
---

# MemPoison: Uncovering Persistent Memory Threats and Structural Blind Spots in LLM Agents

## TL;DR

Establishes that the write gate has a **structural ceiling**, not merely
an implementation weakness. A three-tier taxonomy separates harm that is
visible at write time (L1) from harm that is *deferred* — emerging only
when individually plausible fragments are co-retrieved (L2), or when a
dormant record is activated by a later natural context (L3). Write-time
consistency checking crushes L1 (45.4% → 4.8% corruption) and stalls on
the other two (L2 22.5%, L3 27.8%). A counterfactual analysis explains
why: the harm is not *in* any admitted record. 1,227 hand-validated
cases, ten model families including GPT-5 and Gemini-3 Flash, all
vulnerable.

## Claims

- **Deferred maliciousness is a distinct threat class**, and the write
  gate cannot see it. The stated dilemma: a write-time filter facing
  L2/L3 must "either admit the locally plausible components, thereby
  allowing the attack, or block benign information, destroying the
  utility of the memory system." Precision and utility trade off
  directly, because the fragments genuinely *are* benign in isolation.
- **Every model tested is substantially vulnerable undefended** —
  behavioral corruption rate 54.7–66.9% across ten families at 91–97%
  clean accuracy. Scale and recency do not help: GPT-5 shows the
  *highest* BCR at 66.9%. Corruption is not an artifact of weak models.
- **Difficulty inverts the intuition.** Undefended BCR by tier: L1
  45.4%, L2 51.7%, L3 **76.7%**. The dormant, trigger-activated attack
  is the most effective, and it is also the one a write-time filter is
  least equipped to detect. L2 lags L3 only because it is
  retrieval-completeness limited — the attack needs several fragments
  co-retrieved in one slate.
- **No defense family covers the pipeline.** Admission defenses handle
  L1; retrieval-aware reweighting handles specific co-retrieval failures
  (L2 17.0%, the best single result on that tier, while leaving L3 at
  71.8%); judge- and sanitization-based defenses lower the overall rate
  but leave residual corruption on both deferred tiers. The paper frames
  the result as a **behavioral defense frontier**, stage-specific by
  construction, rather than a solvable problem.
- **Mechanistic Influence Decomposition (MID) gives each tier a distinct
  causal signature**, which makes the taxonomy measurable rather than
  merely descriptive: L1 is concentrated single-record influence
  (Δs = 0.266, 95.7% of poisoned responses revert to clean when the one
  record is removed); L2 shows genuine interaction (Ωg = 0.176 — the
  pair's joint effect exceeds the sum of individual effects, 88.9%
  reversion); L3 shows trigger-conditioned activation (ActivationShift =
  0.242 — the record is behaviorally inert in ordinary context and
  influential only under its trigger, 92.2% reversion).
- **Memory architecture is itself a defense parameter.** Flat chunk
  storage is worst (67.9% BCR) because injected content survives intact;
  hierarchical notes attenuate somewhat (63.1%) via summarization; a
  decomposed fact store is best (56.6%) through "decomposition-induced
  dilution." Structural choices about how memory is stored change attack
  surface independently of any filter.

## Methods

- **Construction is unusually disciplined.** 100 semantic units across
  five long-horizon agent domains × four attack types (fact,
  instruction, preference, state) → 900 family prototypes → 12,760 raw
  surface realizations → 2,575 sampled for human review against six
  criteria (realism, memory writability, non-leaky triggers, behavioral
  separability, harmfulness, label correctness) → **1,227 frozen cases**.
  Semantic families, not prompt variants, are the unit of coverage.
- Paired clean/poisoned protocol per case: the clean run executes the
  trigger query with no adversarial memory; the poisoned run first
  writes the injection through the real memory pipeline, then issues the
  same query. Five repeated runs throughout.
- **BCR (Behavioral Corruption Rate) deliberately replaces attack
  success rate**, measuring only final downstream behavior, because the
  benchmark separates admission, retrieval, and trigger-time
  manifestation as distinct stages. Auxiliary Ambiguous/Unclear rates
  confirm effects are genuine corruption rather than evaluator noise
  (5.1% / 4.6% vs 62.6% BCR).
- Three injection channels — user_input (60.9% BCR), tool_return
  (64.2%), cross_agent (64.1%) — all *legitimate* interaction paths.
- Ten models: Qwen3-8B/32B, Llama3.1-8B, Qwen2.5-14B, DeepSeek-V2-Lite,
  GLM4-32B, DeepSeek-V3, GPT-4o, GPT-5, Gemini-3 Flash. Local runs on
  4×A100 40GB.
- MID is run in two views: *intention-to-treat* (write-blocking counts
  as defense success) and *conditional* (only poisons that were admitted
  and retrieved), isolating residual causal influence from admission and
  retrieval effects.
- Scope explicitly excludes parametric poisoning, side channels, and
  weight tampering — this is about the external memory lifecycle only.

## Results

Overall BCR by defense (undefended baseline 62.55%, CleanAcc 94.40%):

| Defense | Overall | L1 | L2 | L3 | CleanAcc |
|---|---|---|---|---|---|
| None | 62.55 | 45.37 | 51.73 | 76.72 | 94.40 |
| Write-time consistency check | 20.09 | **4.77** | 22.54 | 27.80 | 93.71 |
| LLM-judge write gate | 17.84 | 12.11 | 22.42 | 19.23 | 93.81 |
| Source-reliability retrieval | 52.42 | 43.70 | **17.04** | 71.79 | 93.98 |
| Novelty anomaly filter | 43.39 | 41.07 | 20.41 | 54.07 | 94.02 |
| MIXED (best) | **10.70** | 3.94 | 11.64 | 14.16 | 93.77 |
| PromptGuard | 27.65 | 12.57 | 30.38 | 35.11 | 93.27 |
| Perplexity filter (PPL) | 59.14 | 50.92 | 45.57 | 69.34 | **61.28** |

- The write-time/retrieval-time split is stark and complementary:
  consistency checking is ~9× better than source-reliability retrieval
  on L1 (4.8 vs 43.7) and ~1.3× *worse* on L2 (22.5 vs 17.0). They fail
  on disjoint tiers, which is why the MIXED stack is the only one to get
  everything into single digits or low teens.
- Perplexity filtering is a cautionary result: it destroys clean utility
  (94.4% → 61.3%) while barely moving corruption (62.6% → 59.1%). A
  fluency proxy is not a safety signal.
- Even the best defense leaves 14.2% L3 corruption at full utility cost
  of only 0.6 pp — the residual is a detection ceiling, not a
  precision/recall tuning point.

## Critique / open questions

- **It does not directly test origin-binding.** Every "write-time"
  defense evaluated here is content-based — consistency checks, LLM
  judges, novelty/anomaly filters, perplexity. So this is not a direct
  measurement against
  [[literature/papers/louck2026securing]]'s TMA-NM or
  [[literature/papers/sharma2026smsr]]'s HMAC provenance, and should not
  be read as one. The transfer argument is structural rather than
  empirical (see Follow-up) and deserves a real experiment.
- No code or data release found in the paper. For a benchmark whose main
  contribution *is* a frozen 1,227-case evaluation pack, that materially
  limits the contribution until artifacts appear.
- BCR depends on an LLM evaluator matching responses to
  attacker-specified targets; calibration and thresholds are deferred to
  appendices, and no human validation of the evaluator is reported
  (contrast sharma2026smsr, which quantifies inter-judge agreement).
  The Ambiguous/Unclear diagnostics partly compensate.
- The MIXED defense is the headline mitigation but is a stack of the
  others rather than a new mechanism; its cost (multiple filters plus a
  judge on every write) is not reported in latency or token terms.
- L3's dominance may partly reflect benchmark construction: the trigger
  tasks are authored alongside the dormant payload, so the trigger is
  guaranteed to occur. In deployment an attacker must predict a natural
  context that will actually arise, which is a real constraint the
  76.7% figure does not price in.
- Three substrates is a reasonable spread but excludes the graph and
  vector-hybrid designs that much recent memory work uses.

## Trust signals

- **Credibility:** 4 — Nanjing University's State Key Laboratory for
  Novel Software Technology, a strong systems/software group, with State
  Grid EPRI. Large and carefully constructed empirical base: ten model
  families spanning open and closed weights, five repeated runs, a
  12,760 → 2,575-human-reviewed → 1,227-frozen construction pipeline
  with explicit inclusion criteria, and a counterfactual mechanistic
  analysis rather than attack-success numbers alone. Held at 4 rather
  than 5 by the absence of peer review and of any released artifact.

## Follow-up

- **Relevance:** 5 — this is the source that adjudicates the sufficiency
  question opened when [[literature/papers/sharma2026smsr]] was ingested
  earlier today, and it settles it against sufficiency. The argument:
  L2/L3 payloads arrive through *legitimate, authenticated* channels
  (user_input, tool_return, cross_agent) as *individually plausible*
  records. Channel-authenticated origin binding certifies exactly what
  is not in dispute — that a real principal wrote this — while the harm
  lives in the *relation between records*, or in a future context. An
  origin-bound gate admits L2/L3 by construction, just as a content-based
  gate does, and for a different reason. Combined with sharma2026smsr's
  measured 8% residual against the authenticated writer, the conclusion
  for [[concepts/verified-memory-writes]] is that write-time origin
  binding is **necessary but not sufficient**, and TMA-NM's
  0%-against-laundering claim needs a scope qualifier: it covers
  single-record laundering, not compositional or dormant corruption.
- **The concept needs a fourth layer.** It currently distinguishes a
  *security* layer (origin binding) from a *quality* layer (TrustMem's
  coverage/preservation/faithfulness). Both are per-record. MemPoison
  shows that a per-record gate of any kind has a structural ceiling, so
  the missing layer is **retrieval-time and composition-time** —
  reasoning over the *retrieved slate* rather than the incoming record.
  The empirical support is right there: source-reliability retrieval is
  the single best L2 defense (17.0%) and among the worst on L1.
- **MID looks reusable well beyond security.** Leave-one-out and
  leave-pair-out counterfactuals over a retrieved memory set, plus an
  activation-shift measure across contexts, is a general tool for asking
  "which memory actually drove this response?" — relevant to
  [[concepts/selective-memory-retrieval]] and to any claim that a
  retrieval policy is working. Worth remembering next time we want to
  justify an eviction or ranking choice with something better than
  end-task accuracy.
- Practical note for [[concepts/multi-granularity-memory]]: the
  substrate ranking (fact_store 56.6% < hierarchical_notes 63.1% <
  flat_chunk 67.9%) says decomposing entries into atomic facts dilutes
  injected payloads. That is a security argument for a design this
  project already prefers on retrieval grounds — but it cuts against
  raw-preservation instincts, and it is in tension with L2, where
  fragmentation is the *attack*. Whether decomposition helps or hurts
  net is unresolved and worth watching.
