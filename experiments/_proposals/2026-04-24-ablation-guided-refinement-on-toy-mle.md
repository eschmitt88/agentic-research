---
kind: proposal
slug: ablation-guided-refinement-on-toy-mle
date: 2026-04-24
status: proposed
hypothesis: "On a scaled-down synthetic ML engineering task (single tabular classification, fixed data, <30 min per trial), MLE-STAR-style ablation-guided targeted refinement produces a higher final validation AUC than an equal-budget whole-solution revision loop at matched LLM operator and compute, with the gap widening as block count in the generated code grows."
rationale: "MLE-STAR's core experimental claim is that targeted refinement of individual code blocks outperforms whole-solution revision, evaluated on MLE-bench Lite. The mechanism — ablation identifies the highest-signal block, then refinement focuses exploration there — is domain-general, so it should replicate on a simpler, fully controllable task where we can vary the code's block count and measure the widening/narrowing of the gap. This is the minimum experiment that distinguishes the ablation-ranking mechanism from the targeted-refinement mechanism: in a 1-block solution, the two loops are equivalent; the gap should be a function of block count."
reads:
  - "[[literature/papers/nam2025mle]]"
  - "[[concepts/evolutionary-expansion]]"
expected_metric:
  name: final_val_auc_delta
  target: 0.01
  direction: higher-is-better
design_sketch:
  - Pick a public tabular dataset (e.g. Adult, Titanic) as the toy target.
  - Define three block counts — 1, 3, 5 — by varying how segmented the starting code is (monolithic vs. feature-engineering + model + training loop vs. each further split).
  - Arm-A (ablation-guided): per cycle, run the solution with each block stubbed, rank blocks by signal contribution, pick highest-signal block, generate 3 variants, keep best.
  - Arm-B (whole-solution): per cycle, regenerate the entire solution once based on previous result. Matched cycle count and token budget.
  - Fix seeds, LLM operator, and compute per arm. Run k=5 seeds each.
  - Primary metric is delta in final validation AUC at matched budget; secondary metric is delta as a function of block count.
risks:
  - Toy tabular tasks are LLM-easy; the revision loop may saturate quickly at high AUC and mask the gap. Pick a task known to be non-trivial for LLM-generated solutions, or inject a deliberately hard feature-engineering bottleneck.
  - Ablation-guided refinement needs the blocks to be independently stubbable; poorly chosen block boundaries would distort the ranking.
  - Whole-solution revision is the baseline but "matched compute" is ambiguous — token count matters more than cycle count for LLM cost.
related_prior: []
estimated_runtime: "~1-2h on CPU per arm per seed, 3 block-count variants × 2 arms × 5 seeds ≈ ~30h total."
---

# ablation-guided-refinement-on-toy-mle

MLE-STAR's headline — 64% medals on MLE-bench Lite — is an
impressive aggregate, but MLE-bench Lite is noisy per-competition and
the paper's ablation focuses on component additions (retrieval,
refinement, ensembling) rather than isolating ablation-guided
refinement against a matched-budget whole-solution baseline. The
mechanism is cleaner to evaluate on a controllable toy task where we
can hold everything except block count constant.

The interesting prediction is the gap-as-a-function-of-block-count
shape. If the gap is flat across block counts, ablation-guided
refinement is just "a better prompt" and could be folded into whole-
solution revision by prompt engineering. If the gap widens with block
count, the mechanism is genuinely capturing extra signal via ablation
rankings, and the pattern should be a first-class skill in any
downstream autonomous ML project — probably a `/refine-target <block>`
counterpart to `/iterate`.

The experiment is cheap (single CPU, tabular data, hours not days)
and feeds directly into whether `/iterate` should grow an
ablation-ranking step or stay whole-solution. A negative result is
informative: whole-solution revision is simpler to implement and, if
the gap is small, the operational win is to stay simple.

One hygiene note: "block count" is not a native concept in free-form
LLM-generated code; the three arms use different starting-code
templates that expose the same task at different granularities, and
"block" means "function in the starting code the model edits".
