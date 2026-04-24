---
kind: proposal
slug: per-role-model-downgrade
date: 2026-04-24
status: proposed
hypothesis: "In an AIBuildAI-style four-role agent (manager, designer, coder, tuner), replacing the coder and tuner backends with a cheaper model (e.g. Haiku 4.5 in place of Opus 4.6) while keeping manager and designer on a frontier model preserves MLE-bench medal rate within 3 percentage points on a 15-competition subset, with total token cost reduced by 40% or more."
rationale: "AIBuildAI reports rank-1 MLE-bench performance (63.1% medal) with hierarchical delegation but does not publish per-role model choice or ablate model cost. The hybrid-model-backends concept predicts that strategic roles (manager, designer) benefit from frontier models while execution roles (coder, tuner) saturate earlier on model quality. If the hypothesis holds, downstream projects can run AIBuildAI-style architectures within realistic home-lab budgets. If it fails, the result identifies which role is the token-cost floor — likely the coder, given the debugging churn — and informs where to put the marginal dollar."
reads:
  - "[[literature/papers/zhang2026aibuildai]]"
  - "[[concepts/hierarchical-delegation]]"
  - "[[concepts/hybrid-model-backends]]"
  - "[[concepts/budget-as-ceiling]]"
expected_metric:
  name: medal_rate_delta_at_matched_budget
  target: -0.03
  direction: higher-is-better
design_sketch:
  - Reproduce AIBuildAI's four-role scaffold as faithfully as the abstract permits (explicit manager/designer/coder/tuner split, structured handoffs through manager).
  - Arm-A (uniform): all four roles on Opus 4.6.
  - Arm-B (hybrid): manager + designer on Opus 4.6, coder + tuner on Haiku 4.5.
  - Evaluate on 15 MLE-bench competitions (mix of tabular, vision, NLP), single pass per arm.
  - Fix scaffolding code, random seeds, and compute-per-run budget. Track token usage per role.
  - Primary metric is medal-rate delta at matched cost budget; secondary metric is delta per role-downgrade (tuner only, coder only, both).
risks:
  - AIBuildAI's scaffold is not fully public; our reproduction may be worse than theirs in absolute terms, making the delta noisy rather than informative.
  - Coder role handles debugging; a cheaper coder may spin on errors the stronger one would unstick. The downgrade could manifest as "10x more debug cycles" rather than "lower quality output."
  - HCE discipline bites here — we must ensure both arms see the same validation/test split and that medals are scored only on the held-out test split.
  - 15 competitions × 2 arms × k=3 seeds is budget-heavy; start with 5 competitions and expand only if the delta looks promising.
related_prior: []
estimated_runtime: "~24h wall per arm on the 15-competition set at single-worker pace; halves with a 2-worker async pool."
---

# per-role-model-downgrade

The hybrid-model-backends concept is the load-bearing bet behind this
project's own `budget.yaml` — ideator on Opus 4.7, implementer on
Opus 4.6. The bet is reasonable on priors but has not been
systematically validated against a real ML-engineering workload.
AIBuildAI's four-role split is the most structured prior art to
evaluate against; if the split lets each role choose its model
without coordination loss, the cost-per-medal curve bends favorably
for every downstream project.

The prediction is a small but nonzero medal-rate regression (≤3
points) paired with a large token-cost reduction (≥40%). The
interesting *mechanism* question is which sub-agent carries the
brittleness. If downgrading the coder costs most of the delta, the
result is "coder role is the real bottleneck; invest token budget
there." If downgrading the tuner costs most of the delta, the result
is "tuner role owns the precision-work."

A clean negative — large medal-rate regression under downgrade —
would be informative too: it would suggest AIBuildAI's performance
does not decompose cleanly by role, and the hybrid-model-backends
pattern is weaker prior art than the `budget.yaml` default assumes.
In that case, downstream projects should default to uniform-frontier
staffing.

Reproducing AIBuildAI faithfully is the hardest part of the design;
the scaffold details in the paper are incomplete. The experiment
should open with a "can we match AIBuildAI's 63.1% with uniform Opus
4.6 staffing?" sanity check before running the downgrade arm. If the
sanity check fails far below 63.1%, the downgrade delta is
uninterpretable and the scaffolding is the confound.
