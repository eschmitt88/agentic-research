---
kind: paper
title: "Recognition Without Enforcement: Configuration-Dependent Failures in LLM Agent Instruction Arbitration and External Control"
authors: ["Jun Wen Leong"]
institutions: ["Independent Researcher"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.28502"
code_url: null   # InstructionArbitrationBench announced as "will be released"; no live link in v1
citations: null
source: "raw/papers/leong2026recognition.pdf"
added: "2026-08-31"
relevance: 5
credibility: 2
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/information-firewall]]"
  - "[[concepts/verified-memory-writes]]"
tags: ["agent-security", "prompt-injection", "provenance", "safety", "governance", "agent-architecture"]
---

# Recognition Without Enforcement: Configuration-Dependent Failures in LLM Agent Instruction Arbitration and External Control

## TL;DR

LLM agents *can* detect forged authority — source-channel identity is
linearly decodable from activations and models verbalize the forgery when
asked — yet under some configurations they still emit the conflicting tool
call. The paper names this the **recognition–enforcement gap**, argues that
model self-arbitration is therefore a *capability, not a security boundary*,
and builds an external reference monitor (authenticated source routing +
capability-gated execution) that closes the channel-forgery class by
construction.

## Claims

- **Shared-channel observation.** System prompt `S`, memory records `M`, and
  tool outputs `T` all enter the model as one token sequence. The model can
  decode source-marking delimiters, but has *no architectural mechanism* to
  enforce a trust boundary: every provenance signal is in-context tokens the
  attacker can also forge. The gap is between representational access
  (present) and enforceable boundaries (absent).
- **Behavioral Provenance Neglect (Def. 1)** — an architecture family
  exhibits BPN when (1) a linear probe recovers the source channel at >90%
  balanced accuracy, *and* (2) some instantiation still complies with
  policy-violating instructions from a forbidden channel at >0.5. Explicitly
  distinguished from *provenance blindness*: a blind model cannot represent
  the distinction; a neglectful one can and does not act on it.
- **BPN is a property of (architecture, prompt-policy, stimulus-distribution)
  tuples, not of weights.** The same weights fail deterministically under a
  permissive prompt with fixed stimuli and pass under a restrictive prompt
  with diverse stimuli.
- Prompt-layer defenses do not generalize across models or adaptive
  formulations — hence the move to external enforcement.
- Secure agents require external enforcement, not better recognition.

## Methods

- **Two paradigms.** *Authority spoofing*: a fabricated authority claim in
  persistent memory directs a destructive tool call conflicting with a
  co-present preservation policy; DV = verified structured tool-call
  emission (binary). *Memory-conflict arbitration*: an active system-prompt
  instruction conflicts with a stored memory rule; DV = text-action-plan
  scoring.
- **Fleet evaluation** — authority spoofing across 46 model endpoints / 6
  vendors (incl. open-weight); memory conflict across 48 models (40 API + 8
  open-weight).
- **Linear probes** on hidden states for source-channel classification, with
  *crossed-probe controls* to test whether the signal is a unified abstract
  trust representation (it is not — layer-0 near-ceiling accuracy indicates
  substantial chat-template positional contribution).
- **Prompt–policy 2×2 factorial** ({permissive, restrictive} ×
  {fixed, novel}, N=200) to separate prompt novelty from system-prompt
  strictness.
- **Reference monitor**: source-label routing + HMAC-SHA256 admission
  control over serialized record content + capability-gated tool execution
  requiring valid scoped capabilities.
- Statistics use **model-clustered bootstrap CIs**, which the author notes
  are 4.5× wider than naive trial-level pooling.

## Results

- Average execution under diverse novel attacks is **1.21% [0.5–2.1%]**
  (model-clustered CI over 14,294 analyzed spoofed trials from 29 models) —
  but the risk is *concentrated in reproducible cells*, with up to **47pp
  within-window per-fingerprint range**, and shifts across deployment
  windows.
- The motivating single cell reaches **100% destructive-call rate** on
  GPT-4.1-mini, GPT-5.1 and Gemini-2.5-Pro (N≥150/cell, permissive protocol)
  from a plain authority-spoofing notice — no jailbreak suffix, no "ignore
  previous instructions".
- Execution **resists linear steering** (0/7 models), i.e. the decodable
  provenance direction is not a lever on the action.
- Reference monitor rejects **0/900 test cases** across 3 model families for
  forged / tampered / replayed / unsigned requests — but the author is
  explicit that this is *functional-correctness testing*, deterministic by
  cryptographic construction under EUF-CMA, **not** an empirical robustness
  measure.
- A separate adaptive red-team found one implementation flaw (a since-patched
  clock-skew admission), not a cryptographic bypass.
- **Stated residual boundary**: semantic authorization. Authenticated user or
  deputy messages can still induce privileged requests unless independently
  constrained by scoped capabilities.

## Critique / open questions

- **Independent, unaffiliated author; not peer reviewed.** The artifact
  release (InstructionArbitrationBench, >256k trials, verbatim attack
  prompts, 90-day vendor disclosure) is announced but not linked in v1 —
  re-score credibility upward once it lands.
- Most probe↔behavior pairings are **architecture-family-level, not
  same-instantiation**, because probing ran at BF16/8-bit while behavioral
  testing ran at Q4_K_M. Only Qwen2.5-32B and Llama-3.3-70B close that gap.
  The BPN claim is correspondingly weaker than it first reads.
- The headline 1.21% average is a *reassuring* number attached to an
  *alarming* finding; the paper's own framing (concentration in reproducible
  cells, 47pp spread) is the load-bearing part, and an average is a poor
  summary of a concentrated hazard. Worth remembering when citing.
- The monitor's evaluation is correctness-of-implementation, not adversarial
  robustness. The interesting open question is the residual it names:
  authenticated-but-wrong requests.

## Trust signals

- **Credibility:** 2 — independent researcher, no institutional affiliation,
  arXiv preprint with no peer review, no live code link and no citations.
  Sits at the *top* of that band on methodology rather than provenance:
  model-clustered bootstrap CIs, crossed-probe controls, a prospectively run
  2×2 factorial, an adaptive red-team against its own defense, and unusually
  candid scope caveats (it labels its own monitor result functional-
  correctness rather than robustness). Re-score to 3–4 if
  InstructionArbitrationBench is released as promised.

## Follow-up

- **Relevance:** 5 — this is the canonical evidence for the premise under
  [[concepts/typed-enforcement]]. The project's other 17 sources mostly
  *build* enforcement layers; this one argues from fleet-scale failure data
  *why* the layer cannot live inside the model, and supplies the vocabulary
  (recognition vs enforcement; provenance neglect vs provenance blindness)
  to state the concept precisely.
- The shared-channel observation is the cleanest statement yet of
  [[concepts/information-firewall]]'s architectural premise — one token
  sequence means no boundary — arrived at from the security side.
- Memory-conflict paradigm + HMAC admission control over memory records
  extends [[concepts/verified-memory-writes]] past write-time content checks
  to *channel authentication*.
- "Capability-gated tool execution" is the same shape as
  [[concepts/permission-gate-as-architecture]]; the residual it names
  (semantic authorization through an authenticated channel) is precisely the
  gap that concept has not addressed.
- Pairs directly with [[literature/papers/rahman2026framing]] (same window),
  which independently concludes that robustness comes from constraining the
  destination or isolating the capability, not from the acting model
  recognizing the attack.
