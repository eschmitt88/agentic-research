---
kind: paper
title: "The Framing Gap: Indirect Prompt-Injection Exfiltration Defeats Surface-Level Defenses in Tool-Using Agents"
authors: ["Md Habibur Rahman", "Jaeho Kim"]
institutions: ["Gyeongsang National University"]
year: 2026
venue: "arXiv (cs.CR)"
peer_reviewed: false
url: "https://arxiv.org/abs/2608.27092"
code_url: null
citations: null
source: "raw/papers/rahman2026framing.pdf"
added: "2026-08-31"
relevance: 5
credibility: 3
status: skimmed
related_experiments: []
related_concepts:
  - "[[concepts/information-firewall]]"
  - "[[concepts/permission-gate-as-architecture]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/typed-enforcement]]"
  - "[[concepts/enforcement-boundary-placement]]"
tags: ["agent-security", "prompt-injection", "safety", "evaluation", "agent-architecture"]
---

# The Framing Gap: Indirect Prompt-Injection Exfiltration Defeats Surface-Level Defenses in Tool-Using Agents

## TL;DR

Ten overtly-worded exfiltration injections are refused (gpt-4o: 0%). Reframing
the *identical* leak as a mandatory integrity signature, a runtime-config
field, or a look-alike "trusted" host drives gpt-4o to **100%**. What closes
the gap is not a better-behaved model but **payload-blind structural checks**:
a destination allow-list, or a capability-isolating planner/reader split —
both 0%.

## Claims

- **The framing gap**: refusal is keyed on the surface framing of the request,
  not on the act being requested. Same leak, different clothing, opposite
  outcome.
- **The reusable attack asset is the template, not the mechanism.** Cost is
  three-tiered: paraphrasing a known mechanism is trivial (96% at 3 wordings),
  swapping the field inside a known-effective template is also cheap (up to
  60%), but authoring a fresh page around a *new* mechanism is hard (0/130).
- The mechanism is **instruction/data confusion, not defeated alignment** —
  established by ablation, not asserted.
- **Robustness comes from constraining the destination or isolating the
  capability, not from the acting model recognizing the attack.**

## Methods

- A safe synthetic laboratory: a canary secret, mock tools that only record,
  and a matched clean-vs-poisoned success metric (so utility loss is
  measurable alongside attack success).
- Six models spanning five families; ten overt injection classes as the
  baseline; reframings as the treatment.
- **Ablation** removing the confidentiality policy, to separate
  instruction/data confusion from alignment failure.
- Positive controls run against a standard harness to rule out
  harness/template artifacts.
- Tested defenses: destination allow-list; planner/reader capability split;
  broad "in any form" policy clause; SecAlign (a published fine-tuning
  defense, CCS 2025); channel separation; an output-normalizing guard.

## Results

- Overt injections: 0% (gpt-4o). Reframed: **0% → 100%** on strongest
  wordings. Per-wording rates span 0–100% (mean 52%, SD 45).
- Removing the confidentiality policy leaves base attacks at 0% and moves
  reframing only **31.9% → 38.1%** — so the policy was never what was
  holding, and this is not an alignment failure.
- **Closes the gap**: destination allow-list **0%** (when destinations are
  closed); capability-isolating planner/reader split **0%**.
- **Brittle**: a broad "in any form" policy clause also reaches 0% at the
  acting model, but dropping the catch-all reopens it to **48.8%**.
- **Fails**: SecAlign 32.5% (positive-control-validated, so this is a real
  negative result, not a mis-run); channel separation 38.8%; an
  output-normalizing guard loses entirely to a held-out encoding (ROT13,
  100%).

## Critique / open questions

- Synthetic lab with mock tools and a canary — deliberately safe, but the
  external validity to real egress paths is asserted rather than shown. The
  100% figure is on the strongest wordings in that lab.
- No released code, which matters more than usual here because the negative
  result against a published defense (SecAlign) is a strong claim about
  someone else's work.
- "When destinations are closed" is doing real work in the allow-list result;
  the paper is honest about it, but an agent that must reach arbitrary hosts
  gets no protection from this.

## Trust signals

- **Credibility:** 3 — a national university group, unreviewed preprint, no
  code. Carried to 3 by unusually disciplined method work for a preprint:
  matched clean/poisoned metric, a mechanism-isolating ablation, positive
  controls, a held-out encoding, and testing a *published* defense rather
  than only strawmen.

## Follow-up

- **Relevance:** 5 — this is the second, independent attestation that
  [[concepts/information-firewall]] has been waiting for (NOTES has carried
  "watch for a second attestation" since 2026-08-01). The seedling was
  derived from the *evaluation* side — the agent must not reach the oracle.
  This arrives at the same structural conclusion from the *security* side —
  the agent must not reach the destination. Two independent derivations of one
  boundary is the evidence that moves it off single-source.
- Pairs with [[literature/papers/leong2026recognition]], which concludes from
  entirely different evidence (activation probes + fleet failure data) that
  the acting model cannot be the boundary. Same claim, disjoint methods, same
  week.
- The framing gap is a [[concepts/refusal-cost-symmetry]] result in disguise:
  a refusal keyed on surface form has an asymmetric error profile that the
  attacker, not the operator, gets to choose.
- Actionable here: "constrain the destination or isolate the capability" is
  the design rule, and it is cheap. Candidate for `/elevate` review against
  claude-system's own tool surface.
