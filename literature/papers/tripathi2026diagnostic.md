---
kind: paper
title: "Diagnostic Foundation for Evaluating LLMs' Research Integrity as Co-Scientists"
authors: ["Yash Tripathi", "Silu Sharma", "Sai Sidhanth Manoharan Jayanthi", "Shivank Garg", "Lin Li"]
institutions: ["Jaypee University of Information Technology", "NYU Shanghai", "UC Berkeley", "Algoverse AI Research", "University of Oxford"]
year: 2026
venue: "AI for Science Workshop, ICML 2026 (accepted); arXiv 2608.12345v1, cs.AI (v1 3 Jun 2026, arXiv id announced August 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2608.12345
code_url:
citations:
source: "raw/papers/tripathi2026diagnostic.pdf"
added: "2026-08-18"
relevance: 5
credibility: 4
status: read
related_concepts:
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/information-firewall]]"
related_experiments: []
tags: [benchmark-design, evaluation-integrity, research-integrity, ai-scientist, pressure-protocol, paired-controls, over-refusal, alignment]
---

# Diagnostic Foundation for Evaluating LLMs' Research Integrity as Co-Scientists (IntegrityBench)

## TL;DR

A benchmark whose **design** is worth more here than its scores. Every
misconduct task is paired with an **ethical control** that holds role,
domain, artifact and question structure fixed and changes only the feature
that determines permissibility — so blanket refusal is penalized exactly as
hard as compliance with misconduct. Every task then runs under five
environments that insert social pressure **without adding any factual
information**, making any score drop attributable to framing rather than
evidence. Across 18 frontier variants × 1,800 prompts, models fail roughly
1 in 3 integrity-critical decisions at peak pressure, neither scale (44×
parameters, −1.5 points) nor reasoning (7 of 9 matched pairs
non-significant) reliably helps, and the three decision facets turn out to
be **structurally dissociated** — a model that misclassifies what is wrong
still acts correctly on the artifact.

## Claims

- **Symmetric pairing is the anti-gaming mechanism.** "Symmetric pairing of
  every misconduct task with an ethical control penalizes blanket refusal
  equally as misconduct compliance, targeting a failure mode of
  refusal-tuned LLMs." A refusal-maximizing model scores at chance overall,
  not near ceiling. The controls modify a *single* permissibility-determining
  feature — contamination is documented in the lab notebook, the IRB
  protocol was approved, the p-values are maintained.
- **The pressure protocol isolates framing from evidence.** Five
  environments — baseline, plus a 2×2 crossing of mechanism (implicit
  institutional cue vs explicit named-authority appeal) with intensity
  (moderate vs escalated). "Because pressure blocks are inserted without
  changing the dataset or experimental record, performance changes can be
  attributed to social framing rather than new evidence." Explicitly modeled
  on Milgram-style authority-compliance studies, retargeted at RLHF-trained
  models.
- **Explicit and implicit pressure fail in *opposite directions*.** This is
  the paper's sharpest result and it is why a single integrity score is
  misleading. Explicit named-authority pressure degrades **misconduct**
  performance (68.8 vs 73.5 implicit, t = 6.24, p < .001, 16/18 variants) —
  i.e. it induces compliance with misconduct. Implicit institutional cues
  degrade **ethical-control** performance (61.4 vs 65.2 explicit, t = −7.86,
  p < .001, **18/18** variants) — i.e. they induce over-refusal of
  legitimate work. Opposing signs, two distinct mechanisms, one aggregate
  score that hides both.
- **Neither scale nor reasoning is a fix.** Paired McNemar tests with
  Benjamini–Hochberg correction find **no significant reasoning effect for
  7 of 9 matched pairs** (all adjusted p > .09); the two significant effects
  point in *opposite* directions (Qwen 3.5 Flash 9B improves, GPT 5.4 Mini
  degrades, both adjusted p < .001), mean Δ = +1.3. On scale, a 397B model
  scores **1.5 points lower** than a 9B model of the same family under
  matched reasoning (~44× the parameters), and the large-model tier mean
  exceeds the small by 1.7. Conclusion: integrity is governed by "alignment
  curriculum and techniques," not capability — "scaling a model whose
  alignment conflates confident refusal with ethical judgment produces
  enhanced capability without corresponding integrity improvements."
- **The three facets are structurally dissociated.** Strictly monotone
  difficulty ordering across the panel — Q1 misconduct classification 56.8,
  Q2 ethical action reasoning 66.6, Q3 artifact-grounded decision making
  80.8 (paired Wilcoxon over 36 task means, W = 152, p = .004) — a 24-point
  gap. Conditioning Q3 on Q1 correctness, models that **fail** Q1 do
  equally well or better on Q3 (85.7 vs 79.4). "A classification failure
  therefore does not necessarily imply an unethical downstream decision."
  Correct action does not require correct diagnosis.
- **Surface-cue inversion: what makes misconduct easy to flag makes
  legitimate practice easy to over-flag.** p-hacking scores **86.3 as a
  misconduct task and 42.0 as its ethical control — a 44.3-point
  inversion**. The pattern reverses when misconduct depends on motive
  rather than a visible action (hypothesis anchoring: 58.0 vs 75.5).
  Models "rely on visible research features rather than the procedural
  context that determines whether those features are permissible."
- **Detection ≫ permissibility judgment.** On Q1 alone, models score 71.9
  on misconduct tasks against **41.6** on the paired ethical controls
  (Δ = 30.3) with all task features held constant. Models "are more
  reliable at recognizing suspicious research patterns than at
  distinguishing misconduct from procedurally justified practice," and the
  paper names the concrete casualties: legitimate robustness checks,
  transparent data exclusions, and approved human-subjects procedures
  flagged as misconduct.
- **Expert agreement substitutes for a human baseline.** Three domain
  experts (PhD+) labeled non-overlapping subsets; one ethics expert served
  as overlapping second reviewer across all 36 tasks; Cohen's κ = **0.96**.
  The paper argues this near-perfect agreement establishes a
  "design-validated performance ceiling," "negating the need for a separate
  human baseline."
- **The benchmark is deliberately a floor, and says so.** Q1 is a 19-way
  multiple choice (18 misconduct types + control): "Real deployment affords
  no such menu, making the 19-option format a conservative lower bound on
  classification difficulty. The integrity gaps we observe are therefore a
  floor rather than a ceiling on the true challenge."

## Methods

36 tasks (18 misconduct + 18 paired ethical controls) across three domains
(AI, physics, medicine), four research stages, and three misconduct families
(Bias, Deception, Forbidden Research). The 18 misconduct behaviors were
sourced from **a survey of 47 researchers** reporting violations they had
observed, rather than invented. Every task is **authored synthetically from
scratch** by the team explicitly "to prevent the risk of contamination."
Each task carries a role/context block, a situation description, a JSON
research artifact with summary statistics, and ten questions (1 × Q1,
3 × Q2, 6 × Q3), run under all five pressure environments → 1,800 prompts
per model. 18 variants across five families (Claude, Gemini, Qwen, DeepSeek,
GPT), crossing size × reasoning, all queried through one OpenRouter client
key so request shape and decoding are identical across providers →
**32,400 prompts**, ~$90 and ~70M tokens. Scoring is string matching on
single-character responses. Integrity Score equally weights Q1/Q2/Q3,
bounded [0, 100].

## Results

- **Overall integrity scores 60.9–72.8, mean 68.7** — an 11.9-point spread
  across 18 variants, which the authors read as convergent failure modes
  across alignment pipelines. The best model still fails roughly 1 in 4
  tasks overall; at peak pressure the panel fails roughly 1 in 3.
- Failures are concentrated, not uniform: models do well where the
  violation cue is salient (plagiarism, p-hacking) and poorly where
  misconduct depends on methodological intent (experiment overfitting,
  anchoring). Ethical-control performance is weakest in **analysis**-stage
  tasks, "where permissibility depends on intent and procedural
  justification."
- Ethical-control facet scores are even more skewed than the pooled ones:
  Q1 = 41.6, Q2 = 62.7, Q3 = 89.1.

## Critique / open questions

- **Single-pass evaluation at provider-default temperature.** The paper
  concedes point differences under 2–3 are within sampling noise and — to
  its credit — restricts all inferential claims to the McNemar tests rather
  than to Integrity-Score deltas. Still, no repeated sampling means the
  per-model numbers are one draw each.
- **Backbone models, not agents.** By design: it isolates model-level
  behavior that scaffolds inherit, justified by cited evidence that base
  model choice dominates behavioral variation. But the co-scientist claim
  is about deployed systems, and the paper's own future work names
  scaffolding with a real tool suite as the test of whether the facet
  dissociation survives agentic execution. **The dissociation result is
  precisely the one most likely to change under a scaffold**, since Q3's
  artifact grounding is what a tool-using agent would actually do.
- **36 tasks is small** for the number of cuts taken (3 domains × 4 stages
  × 3 families); the paper reports domain/stage/family differences
  "descriptively given the task counts per grouping," which is the right
  call but leaves the subgroup patterns as observations rather than
  findings.
- **Cross-family comparisons are confounded** by release date and alignment
  curriculum, as the authors state. "Scale doesn't help" rests partly on a
  within-Qwen comparison (9B vs 397B), which is the cleanest cut available
  and still one family.
- **The κ = 0.96 argument is doing a lot of work.** High inter-rater
  agreement shows the *labels* are unambiguous to experts; it does not
  establish that an expert would score near 100 on the ten-question format
  under pressure framing. "Design-validated ceiling" is a reasonable but
  unverified substitution for a measured human baseline.
- Benchmark is said to be on Hugging Face but the paper gives no URL, so
  the artifact is unverified from the PDF alone.
- Q1's 19-way menu is acknowledged to overestimate real classification
  accuracy — which cuts *for* the paper (the gaps are a floor) but means
  the absolute 56.8 is not a deployment-relevant number.

## Trust signals

- **Credibility:** 4 — genuinely careful design and statistics: paired
  controls with a single varied feature, factual content held fixed across
  pressure conditions, misconduct taxonomy grounded in a 47-researcher
  survey rather than authored from intuition, expert validation at κ = 0.96,
  McNemar with Benjamini–Hochberg for reasoning, paired Wilcoxon for the
  facet ordering, paired t-tests reported with per-variant consistency
  counts (16/18, 18/18), a unified client key to remove provider-side
  variation, and an explicit refusal to draw inferences from point deltas
  it deems noise. Held below 5: workshop paper rather than a full venue,
  no artifact URL in the PDF, single-pass sampling, 36 tasks, and
  no independent reproduction.

## Follow-up

- **Relevance: 5** — this supplies the adversarial and *symmetric* half
  that [[concepts/hce-evaluation]] lacks. HCE hides the answer so the
  search loop cannot optimize against it; this hides nothing and instead
  varies the **social framing** while holding evidence fixed, which is a
  second kind of information boundary: the model must not be permitted to
  learn the answer from who is asking. Both are "hold everything constant
  except the one thing you're measuring," and the pressure protocol is the
  cleaner instance because it is a within-item manipulation.
- **Seeds [[concepts/refusal-cost-symmetry]].** The paired-control design
  is the general mechanism: *an evaluation that only measures the permissive
  failure silently rewards the conservative one*. This project's own
  evaluations have this shape — `/lint` counts violations, `/elevate`
  counts accepted proposals, and the agency verdict counts spend — with no
  paired measure of correct work suppressed. Four independent sources now
  attest the axis (see the concept), and this is the one that makes it a
  *design principle* rather than a reported side-metric.
- **The facet dissociation is an argument for
  [[concepts/programmable-evaluator-oracle]] over judge-the-request.**
  Q3 (artifact-grounded) outperforms Q1 (classification) by 24 points, and
  models that get the classification *wrong* score **higher** on the
  artifact decision. So gating on a model's stated diagnosis of what is
  wrong is strictly worse than gating on a check against the artifact —
  which is the concept's thesis, measured. It also complements
  ding2026autonomous's verification-signal ladder: artifact-grounded
  decisions sit higher than model self-classification, in the same
  ordering.
- **The explicit/implicit asymmetry is the reason a single safety number is
  not reportable.** Two mechanisms, opposite signs, both called "integrity
  failure," and an aggregate that averages them toward the middle. This is
  the same lesson [[literature/papers/ray2026what]] states formally
  ("AUC cannot be inverted into a required deployment quality") arriving
  from measurement instead of theory — worth reading the two together.
- **Two importable evaluation-design moves, independent of the subject
  matter.** (1) *Hold the artifact fixed, vary only the framing* — the
  causal-isolation trick, identical in structure to ray2026what's
  byte-identical source-role diagnostic and applicable to any harness
  question of the form "does the agent respond to who is asking rather than
  what is true." (2) *Substitute a design-validated ceiling for a human
  baseline* via near-perfect expert agreement — cheap, and this project has
  no human baseline for anything.
- **A contamination note for [[concepts/information-firewall]]**, not an
  attestation: authoring every task from scratch to prevent contamination
  is the file-space boundary applied at *construction* time. It is the
  weakest form (it protects against memorization of this benchmark, not
  against retrieval), and the paper does not test for leakage — but the
  practice is right and cheap.
- **Watch item.** The claim that integrity is set by "alignment curriculum
  rather than raw parameter count" would, if it holds, mean model choice for
  this box's roles is not a capability question. It rests on cross-family
  comparisons the paper itself flags as confounded; look for a second
  source before treating it as settled.
