---
kind: paper
title: "Hypothesis-Driven Skill Optimization for LLM Agents"
authors: ["Fangxin Shang", "Yehui Yang"]
institutions: ["AI Lab, Qifu Technology"]
year: 2026
venue: "arXiv preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2606.22330"
code_url: null
citations: null
source: "raw/papers/shang2026hypothesis.pdf"
added: "2026-07-28"
relevance: 4
credibility: 3
status: read
related_concepts:
  - "[[concepts/skill-library-lifecycle]]"
  - "[[concepts/typed-claim-partition]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/verified-memory-writes]]"
  - "[[concepts/agent-native-memory]]"
related_experiments: []
tags: [skills, lifecycle, validation, falsifiability, ablation, train-free, promotion-gate]
---

# Hypothesis-Driven Skill Optimization for LLM Agents (HDSO)

## TL;DR

Treat every persistent skill update as a **falsifiable hypothesis** rather
than an accepted reflection. A frozen curator observes executor traces,
proposes a hypothesis with an explicit validation plan and falsification
conditions, instantiates it as a candidate skill package, tests it via
**paired control/treatment executions on matched task indices**, and
promotes only candidates whose evidence supports the proposed mechanism.
Rejected candidates persist as auditable negative evidence in a hypothesis
ledger.

## Claims

- The hard problem is not writing a skill but **deciding when a written
  skill should become durable knowledge**. A failed trajectory could
  reflect a missing domain rule, weak exploration, an invalid action
  format, a poor invocation condition, an executor limitation, or simply
  unreliable feedback — and reflection alone cannot distinguish them.
- Appending every reflection to the prompt produces a repository of
  over-general and contradictory rules. This is the failure mode of
  unconstrained memory accumulation.
- A candidate skill should state: what behavior it will change, when it
  applies, what evidence motivated it, what risks it introduces, and
  **what observations would falsify it**.
- Validation must be **prospective and paired**, not retrospective: run
  repository *S* against *S ∪ {candidate}* on the same task indices.
- Rejected hypotheses are first-class artifacts, not discarded — so future
  cycles do not rediscover them.

## Methods

- **Train-free**: both curator and executor are frozen inference
  endpoints. No curator fine-tuning, which is what makes it deployable
  where the serving model is fixed for cost/compliance/vendor reasons.
- **Lifecycle**: observe → hypothesize → validate → review → consolidate,
  with rejection as an explicit terminal state alongside consolidation.
- **Promotion gate** records paired outcomes: success delta, step delta,
  invalid-action delta. A paired-evidence-strength diagnostic is logged for
  audit but deliberately **not** used as a hard threshold — the gate
  reviews mechanism, not just significance.
- **Progressive disclosure** on the executor side: it first sees compact
  skill cards and requests detail only as needed, and the **executor-only
  path is preserved** when no skill is selected — so the skill layer can
  only help by being invoked, and its absence is always a live control.
- **Hypothesis ledger** holds approved skills, rejected hypotheses,
  validation artifacts, and paired behavior records.

## Results

On ALFWorld, against executor-only baselines:

- **+6.9** Avg. SR points for Qwen3-8B; **+4.0** for Qwen3.6-27B.
- Under **20% randomly flipped** success/failure feedback during both skill
  discovery *and* validation, HDSO preserves a **+7.1**-point gain for
  Qwen3-8B — the noise-robustness headline.
- Transfer diagnostics: validated repositories are useful beyond the run
  that produced them.
- Heterogeneous-pair diagnostics: cross-model curation succeeds **only when
  curator diagnosis, executor capability, and validation evidence align** —
  a curator that correctly diagnoses a problem the target executor cannot
  act on produces a skill that does not help.

## Critique / open questions

- **The noise result is suspiciously good and the paper does not
  interrogate it.** +7.1 under 20% flipped labels exceeds the +6.9 clean
  result for the same model. The natural reading is that run-to-run
  variance is large relative to the effect size, which would undercut both
  numbers; the alternative (paired validation genuinely filters label
  noise) is plausible but needs the variance reported to distinguish. No
  confidence intervals or seed counts are given for the headline deltas.
- Single environment (ALFWorld) and small open models (Qwen3-8B,
  Qwen3.6-27B). The gain is larger for the *weaker* model, which suggests
  skills partly substitute for capability — plausible, but it means the
  result may not survive at frontier scale.
- Paired validation is the expensive part and its cost is not accounted
  against the gain. Every candidate requires matched control and treatment
  runs; a repository that proposes many candidates pays a multiple of the
  base task cost, and there is no budget analysis.
- Authors scope out QA and pure-reasoning settings as future work — the
  method is specific to action-oriented agents with a clear success signal,
  which is also the setting where a promotion gate is easiest.
- No released artifact.

## Trust signals

- **Credibility:** 3 — coherent, well-motivated design with unusually
  careful experimental structure for a two-author industrial preprint
  (paired arms, preserved executor-only control, deliberate noise
  injection, transfer and heterogeneous-pair diagnostics). Held at 3 by the
  single environment, small models, absent variance reporting, no peer
  review, and no released code.

## Follow-up

- **Relevance:** 4 — supplies [[concepts/skill-library-lifecycle]]'s
  missing **enforcement half**. That concept has accumulated many sources
  on acquisition, composition, and retrieval, but the admission rule has
  mostly been policy. HDSO's rule is sharp and testable: *a skill is not
  accepted because it sounds plausible, but because a falsifiable
  hypothesis about it survived a paired test.*
- Directly answers the question [[literature/papers/tang2026memory]] left
  open — whether admission gating and consolidation are substitutes or
  complements. HDSO is the pruning/admission side, and the answer implied
  by the preserved executor-only path is **complements**: consolidation
  decides what a skill *says*, admission decides whether it *enters*.
- **Rejected-hypothesis records are [[concepts/typed-claim-partition]]
  applied to a skill library.** This is the most transferable idea in the
  paper and it is nearly free: the negative evidence costs nothing extra to
  retain once the validation ran, and it prevents the loop from
  rediscovering the same dead end. This project's own `/curate` declines
  with a recorded reason are the same pattern — a decline is a curation
  decision, not an absence.
- The **paired control/treatment design** is [[concepts/hce-evaluation]]'s
  logic applied at the granularity of a single skill: same tasks, one
  variable changed, promote on the delta. Worth noting that HDSO validates
  on the *same* task indices it later reports on, so its promotion
  decisions and its headline gains share a signal — the setup this project's
  own HCE rule exists to separate. A held-out task set for final scoring
  would strengthen the claim considerably.
- Pairs with [[concepts/verified-memory-writes]]: HDSO's promotion gate is
  a write gate for procedural memory, and its "mechanism review, not just
  significance threshold" stance is the semantic analogue of TrustMem's
  coverage/preservation/faithfulness triad.
