---
kind: concept
name: "hybrid-model-backends"
status: active
added: "2026-04-24"
source_papers:
  - zhang2026aibuildai
  - hambardzumyan2026aira
  - ouyang2026skillos
sources:
  - "[[literature/papers/zhang2026aibuildai]]"
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[literature/papers/ouyang2026skillos]]"
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
