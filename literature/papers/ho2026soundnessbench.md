---
kind: paper
title: "SoundnessBench: Can Your AI Scientist Really Tell Good Research Ideas from Bad Ones?"
authors: ["Sy-Tuyen Ho", "Minghui Liu", "Huy Nghiem", "Furong Huang"]
institutions: ["University of Maryland, College Park"]
year: 2026
venue: "arXiv 2605.30329v1, cs.LG (preprint, 28 May 2026)"
peer_reviewed: false
url: https://arxiv.org/abs/2605.30329
code_url: https://huggingface.co/datasets/hosytuyen/SoundnessBench
citations:
source: "raw/papers/ho2026soundnessbench.pdf"
added: "2026-08-18"
relevance: 5
credibility: 4
status: read
related_concepts:
  - "[[concepts/refusal-cost-symmetry]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/information-firewall]]"
related_experiments: []
tags: [ai-scientist, evaluation-integrity, benchmark-design, optimism-bias, over-refusal, calibration, peer-review, scientific-triage, prompt-sensitivity]
---

# SoundnessBench: Can Your AI Scientist Really Tell Good Research Ideas from Bad Ones?

## TL;DR

Tests the **first gate** — can a model reject a methodologically unsound
research proposal *before* anyone spends compute on it — over 1,099 ML
proposals reconstructed from ICLR submissions and labeled by reviewer
soundness sub-scores. The answer is no, in a specific and unusually
well-controlled way. Under standard prompting, 12 frontier LLMs approve
**74.0%** of low-soundness proposals while catching 91.8% of good ones.
Told to be strict, they invert rather than improve: false approvals fall to
19.9% but **high-soundness recall collapses from 91.8% to 36.1%**, and Macro
F1 gets *worse* (54.9 → 49.3). Two frontier models end up at 0% false
approvals and ~0% good-proposal recall — they reject everything. And within
one model family from 2B to 122B, **scale makes the optimism worse**, not
better.

## Claims

- **Scientific triage is the under-tested decision point.** Existing
  agent benchmarks "primarily emphasize execution to reproduce existing
  results or deliver runnable artifacts. They do not test whether an agent
  can critically evaluate the viability of a proposal in the first place."
  The stakes are stated plainly: "without a robust upfront filter,
  autonomous agents do not necessarily accelerate science; they risk scaling
  'bad science' by automating the pursuit of unsound hypotheses."
- **Pervasive optimism bias.** Mean low-soundness recall is **26.0%** — a
  **74.0% false-positive rate** — against 91.8% high-soundness recall. Models
  are not weak judges in general; they are asymmetrically permissive.
- **Strictness does not fix it, it flips it.** Under an aggressive prompt
  ("default to low soundness unless the idea and experimental design are
  clearly strong"), mean false positives fall 74.0% → **19.9%** while mean
  high-soundness recall falls 91.8% → **36.1%**. Ten of 12 models get below
  30% false positives; seven of 12 fall below 40% recall on good proposals.
  **Macro F1 decreases**, 54.9 → 49.3. "The tested stricter prompt does not
  jointly improve both classes; it mainly shifts errors from false positives
  to false negatives."
- **At the extreme, the frontier models degenerate to always-reject.**
  GPT-5.4 and GPT-5.4-Mini under aggressive prompting: **0% false positives
  on both**, with high-soundness recall of **0.0% and 0.2%**. Claude-Sonnet-4.6
  and Qwen3.5-122B reject 94.4% / 95.6% of bad proposals while correctly
  identifying only 18.8% / 16.8% of good ones.
- **Scale makes it worse — measured within one family.** Across six Qwen3.5
  models (2B → 122B) under standard prompting, high-soundness recall
  *improves* with scale (71.8% → 92.8%) while low-soundness recall
  *degrades* (31.0% → 19.2%). "Larger models become more permissive toward
  weak proposals, not less." Under aggressive prompting scale gives no
  recovery either — high-soundness recall ranges 1.4% (4B) to 32.4% (27B)
  with no monotonic trend.
- **Prompt sensitivity is itself a finding, and it is model-specific.**
  GPT-5.4 has the best Macro F1 under standard prompting (69.7%) and drops
  to 29.5% under aggressive. A benchmark that reports one prompt condition
  would rank these models differently — and wrongly.
- **Models catch blatant flaws and miss subtle real ones.** Injecting severe
  hypothesis–experiment mismatches into 100 high-soundness proposals drops
  GPT-5.4's approval rate from 77.0% to **1.0%**. So the failure is not
  inattention to methodology; it is insufficient criticality toward "subtler
  naturally occurring flaws."
- **Deliberately scoped claim.** "SoundnessBench should be interpreted as a
  benchmark for **recoverable proposal-stage soundness** rather than exact
  prediction of full-paper review outcomes," and soundness means
  "proposal-stage methodological integrity in ML research, not eventual
  impact, novelty, acceptance, or universal validity."

## Methods

Five-step construction from 137,940 expert reviews down to 1,099
hypothesis–experiment pairings (458 low-soundness). Labels come from ICLR
**reviewer soundness sub-scores** rather than acceptance decisions or
overall ratings — a deliberate choice to target methodological validity
rather than outcome. Extraction fidelity is audited with retrieve-then-verify
atomic claims traceable to the source manuscript; 66.93% of candidates pass
that filter. Evaluation covers 12 frontier LLMs under two prompt conditions,
reporting per-class recall and Macro F1, plus a six-model within-family scale
sweep (Qwen3.5, 2B–122B).

**Five robustness controls**, which are the reason to trust the headline:
1. **Human audit** — 92.3% of leakage checks return the expected "no
   outcome cues leaked," 84.6% of label-validity checks return the expected
   "yes, defensible."
2. **Contamination** — an ICLR-2026-only split evaluated with models whose
   documented training cutoffs precede that submission period: low-soundness
   proposals are still approved at 77.47%, versus 73.88% on the full
   dataset. The bias is *not* memorization.
3. **Identifier removal** — stripping paper-identifying phrases moves the
   aggregate confusion matrix by ~1pp.
4. **Surface features** — no-training baselines using proposal length,
   experiment count, and risk-factor count "fail in the opposite direction
   from LLMs: they over-reject high-soundness proposals." A confound that
   produced the observed pattern would have to act in the LLM-specific
   direction, and the obvious ones do not.
5. **Adversarial content** — the injected-mismatch control above.

Plus persistence checks across years, subfields, and writing-quality bands.

## Results

Summarized above. The structure worth carrying: **both** error directions
are reported for **every** model under **both** prompts, so the trade-off
curve is visible rather than a single headline. That is what makes the
"strictness just moves the error" claim legible at all.

## Critique / open questions

- **Reviewer sub-scores are a proxy, and the paper's own audit is
  qualified.** Only 84.6% of label-validity checks came back defensible on
  the audited subset, and the audit is described as "preliminary" on a
  subset rather than complete. So roughly one label in six may not be
  supportable from the proposal, against effects that are large — but it
  bounds how finely the numbers can be read.
- **ICLR reviewers are not a gold standard.** Peer review has known
  reliability problems, and a soundness sub-score is one reviewer's
  judgment. The paper handles this by scoping to "recoverable" soundness
  and by not claiming to predict outcomes, which is the right move, but the
  ground truth is human judgment with its own noise floor.
- **Two prompts, not a threshold sweep.** "Standard" and "aggressive" are
  two points on what is obviously a continuum. The interesting object is the
  ROC curve — is there *any* operating point with acceptable error on both
  classes? — and it is not reported. Compare
  [[literature/papers/ray2026what]], which argues precisely that a
  reportable claim about a judge is an operating-point constraint, not a
  score.
- **Proposals reconstructed from published papers may be systematically
  cleaner** than proposals as an autonomous agent would actually generate
  them: they were written by researchers who went on to submit, and passed
  an extraction filter that dropped a third of candidates. The gate an AI
  scientist needs is over *its own* ideas, whose failure distribution may
  differ.
- **No agentic condition.** Backbone LLMs judging text, no tool use, no
  literature retrieval, no ability to check whether a baseline exists. A
  real first-gate agent would search; whether retrieval fixes optimism bias
  or amplifies it (by finding support for anything) is untested and is the
  obvious next experiment.
- The 122B model is absent from the reported standard-prompt scale trend
  endpoints (71.8% at 2B to 92.8% at **35B**), so the "scale makes it worse"
  line is read across a partly reported range.

## Trust signals

- **Credibility:** 4 — the control suite is the strongest part: a
  contamination check using *cutoff-restricted models on a post-cutoff
  split* (rather than the usual hand-waving), identifier ablation, a
  surface-feature baseline that fails in the *opposite* direction, an
  adversarial injection control establishing that models can see blatant
  flaws, and persistence checks across years/subfields/writing quality. The
  scoping language is careful throughout ("imperfect but audited proxy,"
  "narrow the most plausible alternative explanations" rather than
  eliminate them), both error classes are reported for every condition, and
  the dataset is released. Held below 5: preprint; reviewer sub-scores as
  proxy ground truth with a preliminary audit at 84.6% label validity; two
  prompt points rather than a threshold sweep.

## Follow-up

- **Relevance: 5** — the fifth and sharpest attestation for
  [[concepts/refusal-cost-symmetry]], and the first on a *research* gating
  task rather than a safety one. The pattern is textbook: measure one error
  direction, tighten the policy, watch the other direction explode. 74.0%
  → 19.9% false approvals bought at 91.8% → 36.1% recall on good work, with
  aggregate F1 *falling*. And the degenerate endpoint is explicit — two
  frontier models at 0% false positives and ~0% true positives, i.e. the
  block-all rule [[literature/papers/ray2026what]] shows is what conformal
  calibration actually returns. Two independent literatures, same
  degenerate corner.
- **This is the measurement `/elevate`'s reputability gate has been missing,
  and it cuts against the skill's current design.** That skill's stated
  posture is that "most cycles correctly produce zero proposals" — a
  one-sided guard tuned to avoid bad adoptions, with nothing measuring good
  ideas held back. SoundnessBench says a model asked to be strict about
  research quality does not become discerning, it becomes **near-uniformly
  rejecting**, and that the aggregate quality metric gets *worse* while
  looking safer. `/elevate`'s two bars (reputability, simplicity) are
  exactly an aggressive prompt. Worth a concrete change: score a few known-
  good historical proposals alongside the candidates, so a cycle that
  rejects everything is distinguishable from a cycle with nothing to
  accept.
- **Strong support for [[concepts/programmable-evaluator-oracle]] over
  judge-the-idea, from the demand side.** Combined with
  [[literature/papers/tripathi2026diagnostic]]'s facet dissociation
  (artifact-grounded decisions beat model self-classification by 24 points)
  and ding2026autonomous's ladder placing model judgment at Tier VIII, the
  picture is consistent: model assessment of a *claim* is the weakest
  signal in the stack, and it is exactly what an autonomous research loop
  reaches for when deciding what to work on next.
- **"Scale makes it worse" is the finding to watch.** Within one family,
  2B→122B, permissiveness toward weak proposals *increases* monotonically.
  If that replicates it is a genuinely awkward result for the assumption —
  implicit in this project's `budget.yaml` choice of Opus for both roles —
  that a stronger model is a better filter. Note it converges with
  [[literature/papers/ishibashi2026effective]] (more capable models produce
  *more* evaluation hacks) and with tripathi2026diagnostic (44× parameters,
  −1.5 integrity points): three independent papers now report that
  capability and judgment-under-pressure move in opposite directions.
  That is a cluster worth a concept if a fourth arrives.
- **The contamination control is a method worth copying**, and it is better
  than what [[concepts/information-firewall]] currently documents: rather
  than curating a corpus or disabling retrieval, evaluate on a split that
  postdates the models' documented training cutoffs *and* restrict the model
  set to those with documented cutoffs. Cheap, and it converts the
  time-boundary idea from EvoBrowseComp into a control that any existing
  benchmark can bolt on.
- **The surface-feature baseline failing in the *opposite* direction is an
  underrated trick.** It rules out a whole class of confounds in one
  cheap experiment: if a dumb heuristic reproduces your effect, you have
  measured the heuristic; if it produces the mirror image, you have not.
  Applicable to any claim this project makes about agent behavior.
