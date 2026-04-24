---
kind: proposal
slug: hce-discipline-ablation
date: 2026-04-24
status: proposed
hypothesis: "On a controlled MLE-bench-Lite subset, an autonomous search loop with the HCE discipline enforced (no test-split reads during search) achieves final held-out percentile rank within 1 standard error of a loop that allows test-split reads during search for the first 48 hours; over 72+ hours, the no-HCE loop degrades while the HCE-disciplined loop stays flat or improves."
rationale: "AIRA_2 names validation-based selection overfitting as one of three dominant bottlenecks but reports aggregate 24h/72h percentile-rank numbers rather than an ablation of HCE in isolation. The paper's ablation confirms necessity but does not characterize when HCE bites. This proposal is the ablation worth running: short-horizon loops may not need HCE at all, which would simplify downstream projects' evaluation scaffolding; long-horizon loops should show divergence as the test-split-reading loop overfits. The finding would update downstream default search budgets and whether HCE is worth the complexity for smaller projects."
reads:
  - "[[literature/papers/hambardzumyan2026aira]]"
  - "[[concepts/hce-evaluation]]"
expected_metric:
  name: final_test_percentile_rank_delta_at_72h
  target: 5.0
  direction: higher-is-better
design_sketch:
  - Pick 5 MLE-bench-Lite competitions with stable human baselines.
  - Run two arms: arm-A strict HCE, arm-B allows test-split reads during search.
  - Fix seeds, LLM operators, compute budget, worker count — HCE enforcement is the only independent variable.
  - Run each arm to 24h, 48h, 72h and record final held-out percentile rank at each checkpoint.
  - Primary metric is the delta between arms at 72h; secondary metric is the point where the two arms diverge by more than 1 SE.
risks:
  - Arm-B may converge to the test split fast enough that validation-test agreement is high by 24h, making the ablation look like a wash until much longer horizons.
  - MLE-bench-Lite's 5-competition subset may be too noisy per-competition for 1 SE detection — may need k≥3 seeds per arm per competition.
  - Arm-B is deliberately violating the project's own evaluation rule; the experiment needs a fenced subdirectory outside the usual HCE-enforcing harness.
related_prior: []
estimated_runtime: "~72h wall per arm × 2 arms × k seeds — budget-heavy, needs multi-GPU async pool."
---

# hce-discipline-ablation

The AIRA_2 paper is the reason this project's evaluation rule treats
`test/` as off-limits during search. What the paper does not tell us
is *when* the HCE discipline starts mattering. The ablation in the
paper confirms HCE is one of three necessary components for the full
AIRA_2 system; the discipline's marginal value as a function of
search-loop horizon is what downstream projects actually need to know
when sizing their scaffolding.

The hypothesis is deliberately bracketed: at short horizons (≤24h on
MLE-bench-Lite), the HCE-enforced loop and the loop that allows test
reads should track each other; the test-reading loop has not had time
to overfit its selection signal to the test split. At longer horizons
(48h–72h), the test-reading loop should start to overfit — the same
failure mode AIRA_2 warns about — and the HCE-disciplined loop should
either plateau (because the search signal is exhausted) or continue
slow improvement (because the validation signal still has room).

If the divergence shows up at 48h already, HCE is clearly load-bearing
for any non-trivial autonomous run and we continue to enforce it
strictly. If the divergence never emerges within the 72h budget, HCE
still has value as a discipline, but for short projects the
enforcement complexity is probably not worth the overhead.

This is a ceiling-heavy experiment — two arms of ~72h each times k≥3
seeds is a multi-day campaign. It would likely run as a downstream
project rather than in this meta project, and benefits from the
[[concepts/async-worker-pool]] pattern to keep wall time manageable.
The experiment also needs careful hygiene: arm-B violates the
project's own HCE rule, so it must run in a fenced harness that does
not touch the default evaluation machinery.
