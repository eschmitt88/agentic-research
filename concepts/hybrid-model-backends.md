---
kind: concept
name: "hybrid-model-backends"
status: active
added: "2026-04-24"
source_papers:
  - zhang2026aibuildai
  - hambardzumyan2026aira
  - ouyang2026skillos
  - novikov2025alphaevolve
sources:
  - "[[literature/papers/banu2026harness]]"
  - "[[literature/papers/zhang2026aibuildai]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/ouyang2026skillos]]"
  - "[[literature/papers/novikov2025alphaevolve]]"
  - "[[literature/papers/zhang2026skillcomposer]]"
  - "[[literature/papers/zhao2026generative]]"
  - "[[literature/papers/wang2026act]]"
  - "[[literature/repos/nousresearch-hermes-agent]]"
  - "[[literature/repos/hkuds-openharness]]"
used_by: []
related_concepts:
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/skill-library-lifecycle]]"
related_experiments: []
tags: [model-selection, budget, ideator-implementer, agent-architecture]
---

# hybrid-model-backends

## Definition

Split agent work by role — strategic ideation versus mechanical
implementation — and let each role choose its own model family. A
single frontier model does not dominate every sub-task; a cheaper /
faster model on the implementer side often matches quality at a
fraction of the cost when the strategic decisions are made by a
stronger ideator.

## Why it matters

AIBuildAI ([[literature/papers/zhang2026aibuildai]]) splits ML
engineering into four roles (manager, designer, coder, tuner) — the
coder and tuner are implementation-heavy and do not need the same
model quality as the designer. AIRA_2
([[literature/papers/hambardzumyan2026aira]]) shows that weak fixed
single-turn LLM operators are a structural bottleneck; the inverse
is also true — using a frontier model for work that a cheaper model
can do is wasted budget.

In practice, the `budget.yaml` pattern already encodes this split:
`models.ideator: claude-opus-4-7` for strategic skills
(`/propose`, `/derive-experiment`), `models.implementer: claude-opus-4-6`
for execution-heavy skills (`/implement`). The pattern shows up
organically because the cost-per-quality curve flattens steeply on
mechanical tasks.

AlphaEvolve ([[literature/papers/novikov2025alphaevolve]]) attests
the same split at a different axis: an **explore/exploit** model
ensemble where Gemini Flash supplies high-volume cheap proposals
("maximizes the breadth of ideas explored") and Gemini Pro supplies
fewer but higher-quality refinements ("provides critical depth").
This is the same hybrid-backend principle (use the right tier for
the right work) re-cast as breadth-vs-depth rather than
ideator-vs-implementer. Downstream projects with evolutionary search
loops should consider both axes — role split *and* tier split within
a role.

## Implementation guidance

1. **`budget.yaml` encodes defaults.** A `models:` block lists
   named roles:
   ```yaml
   models:
     ideator: "claude-opus-4-7"
     implementer: "claude-opus-4-6"
   ```
   Skills read the appropriate role at invocation time.

2. **Skills declare their role.** `/propose`, `/derive-experiment`,
   `/discover`, `/expand` are ideator-side. `/implement`,
   `/new-experiment` (scaffolding bits), any execution subagent
   are implementer-side.

3. **CLI flags override.** Per-call model selection is always
   allowed; `budget.yaml` just sets the default. Overrides should
   be justified in NOTES.md.

4. **Measurement.** Track tokens per role in `_meta/token_log.ndjson`
   so the project can tell whether the split is earning its keep.
   A project where the implementer burns 10x the ideator's tokens
   but produces worse results should shift budget, not ratios.

## Open questions

- The frontier keeps moving. Today's sensible split (Opus 4.7 ideator,
  Opus 4.6 implementer) may invert when 4.8 lands and 4.6 gets
  cheap. Treat the specific model IDs as parameters, not defaults.
- Whether Haiku or a smaller model can serve as implementer for
  simple tasks (scaffolding, boilerplate edits) is an open question;
  downstream projects are encouraged to try it and report back.
- **Trained role-specific small models can beat frontier defaults.**
  SkillOS ([[literature/papers/ouyang2026skillos]]) reports that a
  GRPO-trained 8B *curator* (the ideator-role in their architecture)
  outperforms Gemini-2.5-Pro used directly as curator on the same
  executor. The standard "stronger model on the strategic side"
  default is wrong when the strategic side benefits from being
  *calibrated to the executor's actual usage patterns*. Worth
  testing in this project's own ideator/implementer split — a
  smaller model trained on this project's ideation patterns may
  outperform Opus 4.7 used out-of-the-box.
- **A small strategic model can lift a much larger executor.**
  SkillComposer ([[literature/papers/zhang2026skillcomposer]]) reports a
  **4B** skill composer improving a **27B** executor by up to +4.5 (agent)
  / +3.4 (code). The composer plays the ideator/curator role — it
  authors and evolves the skills the larger executor consumes — and is
  ~7x smaller than the model it improves. This is a second attestation
  (after SkillOS's 8B curator) that the strategic side does *not* have to
  be the bigger model; here the small side earns its keep not through
  executor-specific training but through a *specialized capability*
  (learned skill composition: create/improve/merge). The lesson for this
  project's ideator/implementer split: a role-specialized small model can
  beat a frontier default when the role is a *learnable skill*, not just a
  reasoning-horsepower contest.
- **Third attestation, sharpest ratio yet: a ~3.9M-trainable-param
  specialist beats a frontier API judge at skill selection.** Generative
  Skill Composition ([[literature/papers/zhao2026generative]]) trains a
  tiny decoder (frozen 0.6B encoder + 3-layer AR head) to compose skill
  plans for GPT-5.2-Codex and Gemini-3-Pro executors; it beats
  Gemini-2.5-flash-as-judge by +12.9 pp Set F1 while being two orders of
  magnitude faster, and *degrades more gracefully under distribution
  shift than SFT of a 154×-larger backbone* (−11 pp vs −27.5 pp on the
  real-task holdout). The added nuance over SkillOS and
  zhang2026skillcomposer: when the strategic sub-decision has *closed
  structure* (a fixed library, a bounded output vocabulary), the
  specialist's advantage comes from exploiting that structure — the
  generalist judge cannot even fit the full skill bodies in its context
  budget. Route structured sub-decisions to structured specialists; save
  the frontier model for open-ended reasoning.
