---
kind: paper
title: "Nine Judges, Two Effective Votes: Correlated Errors Undermine LLM Evaluation Panels"
authors: ["Guneet Kohli"]
institutions: ["Apple"]
year: 2026
venue: "arXiv 2605.29800 (cs.CL), v1 2026-05-28, preprint"
peer_reviewed: false
url: "https://arxiv.org/abs/2605.29800"
code_url: null
citations: null
source: "raw/papers/kohli2026nine.pdf"
added: "2026-10-20"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/shared-substrate-contagion]]"
  - "[[concepts/hce-evaluation]]"
tags: ["llm-as-judge", "judge-panel", "correlated-errors", "effective-sample-size", "kish-design-effect", "condorcet-jury-theorem", "majority-vote", "dawid-skene", "aggregation", "model-family-diversity", "chain-of-thought", "chaosnli", "rewardbench", "human-label-variation", "independence-axis"]
---

# Nine Judges, Two Effective Votes: Correlated Errors Undermine LLM Evaluation Panels

## TL;DR

Nine frontier LLMs from seven families label 1,000 items from each of three
ChaosNLI datasets (MNLI, SNLI, AlphaNLI; 100 human labels per item) and
1,000 RewardBench pairs. Gold is the 100-annotator majority. The author
builds a binary error vector per judge, takes the mean pairwise phi
correlation, and converts it to a Kish effective number of votes,
n_eff = k / (1 + (k − 1)·φ̄). On MNLI, φ̄ = 0.391 and n_eff = 2.18
[2.07, 2.31] for k = 9. Across all four datasets and five prompt/decoding
variants, n_eff stays between 1.94 and 2.48. Majority vote beats the best
single judge only on MNLI (+0.2 pp, within tie-breaking noise) and loses
on SNLI (−6.5), AlphaNLI (−2.5) and RewardBench (−2.8). A Condorcet null
model that simulates independent voting with each judge's per-difficulty
confusion matrix predicts 94.0% on MNLI against an actual 72.0%, a 22.0 pp
gap. Dawid-Skene and accuracy-weighted voting close at most 11% of that
gap. Same-family pairs are barely more correlated than cross-family pairs.

## Claims

- "The 9 judges effectively provide only about 2 independent votes' worth
  of information" (abstract).
- "Roughly three-quarters of the panel's nominal independence is lost
  because the models make the same mistakes on the same items"
  (abstract). This is the independence ratio n_eff/k = 24.2% on MNLI.
- The panel's accuracy "falls 8–22 percentage points short of what
  independent voting would achieve, and the best single judge matches or
  outperforms the full panel across all conditions."
- "Neither adding more judges nor using smarter aggregation algorithms
  helps": stable methods close at most 11% of the Condorcet gap, "even
  with access to the correct answers."
- The deficit is "robust across prompt variants, temperatures,
  chain-of-thought reasoning, and a pairwise preference task".
- "The bottleneck is correlated judges, not the aggregation algorithm."
- Human panels "achieve roughly 2× higher n_eff (4.0–5.8 vs. LLMs'
  2.2–2.5), suggesting the deficit is specific to LLM judges."
- Recommendation: report n_eff as a standard panel diagnostic and treat
  results with caution when n_eff/k < 0.5.

## Methods

- **Panel (Table 1).** GPT-4o, GPT-4o-mini, Claude Sonnet 4.5, Gemini 2.5
  Pro, Llama 4 Maverick, Llama 4 Scout, Qwen3-32B, Mistral Large 3,
  DeepSeek-V3. Temperature 0, one sample per item, a fixed one-word NLI
  prompt (App. A). MNLI error rates are 0.282–0.356. Parse failures are
  under 0.1% and are assigned by hash.
- **Data.** 1,000 items per ChaosNLI set, entropy-stratified over human
  label entropy terciles, seed 42. RewardBench: 1,000 pairs,
  proportionally stratified over four categories, MT-Bench pairwise
  prompt with A/B position randomisation.
- **n_eff.** Kish formula on the 36 pairwise phi coefficients of the
  binary error vectors. Checked against an eigenvalue estimate
  k/λ_max (2.16 vs 2.18). Item bootstrap, 10,000 resamples.
- **Condorcet null.** Items are binned into three human-entropy terciles.
  Per judge and bin, a 3×3 confusion matrix is estimated. For each item,
  10,000 Monte Carlo draws sample each judge's vote independently given
  the gold label and bin. Gap = predicted − actual majority accuracy,
  weighted over panel-entropy bins. Split-half cross-validation gives
  overfitting ratios of 0.96–1.00.
- **Significance.** A permutation test shuffles each judge's error vector
  within entropy strata. Observed φ̄ = 0.391 against a null mean of 0.060
  (SD 0.005), p < 10⁻⁴.
- **Aggregation arms (Table 5, 11).** Dawid-Skene EM (no gold),
  accuracy-weighted voting with 5-fold CV on gold, and phi-optimal
  (Markowitz) weighting with CV on gold.
- **Human comparison (footnote 1).** "Human n_eff is estimated by sampling
  10 labels per item from the aggregate ChaosNLI distribution, treating
  annotators as exchangeable."

## Results

- **Headline (Table 2, 3).**

  | | MNLI | SNLI | AlphaNLI | RewardBench |
  |---|---|---|---|---|
  | n_eff (Kish) | 2.18 | 2.35 | 2.48 | 1.99 |
  | φ̄ | 0.391 | 0.354 | 0.328 | 0.440 |
  | Panel acc. | 72.0 | 77.7 | 88.7 | 92.7 |
  | Best judge | 71.8 | 84.2 | 91.2 | 95.5 |
  | Condorcet pred. | 94.0 | 91.7 | 96.3 | 99.5 |
  | Gap (pp) | 22.0 | 14.0 | 7.6 | 6.8 |

- **Error distribution (Fig. 1).** 290 MNLI items (29%) have all nine
  judges right and 51 (5.1%) have all nine wrong. Independence predicts
  fewer than one all-wrong item. 36 of the 51 all-wrong items (70.6%)
  are in the high human-entropy tercile. Mean human support for the
  panel's wrong answer on those 51 is 35.3% (Table 12).
- **Unanimity is not a certificate.** Unanimous items (n = 319) are 90.9%
  correct; the null predicts 99.99%. Items with any disagreement
  (n = 681) are 63.1% correct.
- **Difficulty explains little of the gap.** With three bins, 6.8% of
  the 22.0 pp gap is attributed to item difficulty; with ten bins, 13.5%.
  "66–87% remains unexplained across all datasets." The gap is
  significant in 8 of 12 panel-entropy bins (Table 6).
- **Scaling (Fig. 2, Table 8).** n_eff(k) tracks the Kish curve exactly:
  1.45 at k = 2, 1.96 at k = 5, 2.18 at k = 9, asymptote 1/φ̄ ≈ 2.56.
  The first five judges give 90% of the nine-judge n_eff.
- **Robustness (Table 4).** Reframed prompt 2.17, reversed labels 2.15,
  T = 0.5 2.17. Chain-of-thought *raises* correlation (φ̄ 0.456, n_eff
  1.94) and lowers panel accuracy to 69.2%.
- **Family (§5.3).** Same-family pairs φ = 0.437 (OpenAI) and 0.435
  (Meta), against a cross-family mean of 0.389 (difference 0.047). The
  three most correlated pairs are all cross-family: Claude × Gemini 0.603,
  GPT-4o × Claude 0.588, Mistral × DeepSeek 0.564. One-judge-per-family
  (the best of each) gives n_eff = 1.93, lower than the full panel. On
  RewardBench the same-family excess is larger (+0.109).
- **Leave-one-out (Table 9).** Δn_eff ranges from −0.13 to +0.02.
  Removing any of six judges raises accuracy. Dropping Gemini 2.5 Pro
  raises it by 1.3 pp [+0.1, +2.6].
- **Aggregation (Table 11).** MNLI: majority 72.0, Dawid-Skene 70.7,
  accuracy-weighted 72.2, phi-optimal 72.4. AlphaNLI: Dawid-Skene closes
  10.5% of the gap; phi-optimal drops to 86.2, below majority.
  RewardBench: phi-optimal closes 20.6% (94.1%) and the best judge
  (Claude Sonnet 4.5, 95.5%) still wins.

## Critique / open questions

- **Against the digest framing:** BELOW (mild). The digest's number is
  right: n_eff ≈ 2 for a nine-judge, seven-family panel. Its gloss,
  "the cleanest panel-level shared-substrate result found so far", goes
  past what the n_eff figure isolates.
  1. n_eff is computed from *unconditional* error correlation. Items
     that are hard for everyone push φ up with no shared substrate at
     all. The paper's own human baseline shows how much. Footnote 1
     samples 10 labels i.i.d. from each item's human label distribution.
     That is conditionally independent voting by construction, and it
     still yields n_eff of only 4.0–5.8 out of 10. So roughly half the
     nominal independence is lost to item difficulty alone. The paper
     reads this baseline as "the deficit is specific to LLM judges". It
     is better read as the difficulty-only floor, and the LLM excess
     beyond it is about 2× in n_eff, not "three-quarters lost to shared
     mistakes".
  2. The Condorcet gap is the difficulty-conditioned quantity, and it
     survives: 22.0 pp on MNLI, permutation p < 10⁻⁴. But conditioning
     is on three coarse entropy bins. Going to ten bins doubles the
     explained share (6.8% → 13.5%). Within-bin heterogeneity is a
     plausible further share, so the gap is an upper bound on
     beyond-difficulty correlation.
  3. Some "shared errors" are defensible readings. 70.6% of the 51
     all-wrong items are high-entropy, with 35.3% average human support
     for the panel's answer.
  4. "Shared substrate" names a cause the paper never tests. There is no
     arm that varies training data, and the family result argues
     against the simplest version of it (cross-family pairs are the most
     correlated).

  The correlated-error finding itself holds after these cuts. The
  magnitude "two votes" is the least conditioned version of it.
- **Three of four tasks are classification, not evaluation.** NLI
  labelling with a one-word answer is not grading an artifact. Only
  RewardBench is a judge task. The paper's title and framing ("LLM
  evaluation panels") lean on RewardBench, where n_eff = 1.99 and the
  best judge wins by 2.8 pp. The limitations section concedes that
  open-ended generation and code review may differ.
- **Best-judge comparisons select in-sample.** "The best single judge
  matches or outperforms" uses the best of nine on the same 1,000 items.
  kim2026are found that held-out selection nearly doubles how often
  majority vote beats the best member (18.71% vs 9.98% of size-3
  subsets). The paper notes that identifying the best judge needs gold
  labels; it does not run a held-out selection.
- **One sample per judge at T = 0.** Self-consistency or multiple samples
  per judge are not tested. These would add within-judge votes, which
  should be even more correlated, so the omission probably does not
  flatter the conclusion.
- **One panel.** The bootstrap CI covers item sampling only. The author
  says so: "a different panel might yield a different n_eff."
- **Where it sits in the graph.**
  - It is the no-channel baseline for [[literature/papers/ganesh2026homogenization]]:
    judges here never see each other, yet already share most of their
    errors. Any homogenization a broadcast channel adds starts from this
    floor, which is shao2026language's point about correlated priors.
  - It agrees with [[literature/papers/zheng2026engineering]] that a
    model-family swap buys little independence.
  - It disagrees in direction with [[literature/papers/chen2026rankevolve]],
    where a vendor swap cut patch error correlation from 0.58 to 0.21.
    Two differences: rankevolve's swap also changes the harness, and
    patches are generated rather than one-word labels.
  - It extends [[literature/papers/kim2026are]] from size 2–4 MCQ
    subsets to a nine-judge panel. Error rates here are narrow
    (0.28–0.36), so the accuracy confound kim2026are warns about is
    smaller. The one-per-family result (best judges correlate more) fits
    kim2026are's mechanism.
  - [[literature/papers/zhang2026veriharness]] found 34% of unanimous
    claims wrong across ten same-model agent rollouts. Here nine
    cross-family judges are wrong on 9.1% of unanimous items. That is
    lower but far above independence, on a much easier task.
- **Live fronts.**
  - *HCE evaluation.* Any LLM grader panel in a loop should be counted
    as about n_eff votes, and its unanimous verdicts are not
    certificates. Correlated judge error is also a fixed bias shared
    across reruns and arms. The accepted no-improvement noise band
    measures rerun spread, so it cannot see this error. Only agreement
    on labelled items can, which is zhang2026who's rule in
    [[concepts/hce-evaluation]].
  - *Shared-substrate correlated error.* This is the panel-level
    measurement on the independence axis. It is not propagation:
    nothing passes between judges.
  - *Redirect-before-halt proposal; skill scoping.* No bearing.
  - *This hub.* Parallel same-model readers of the same PDF are the
    extreme case (one family, one prompt). Their agreement is resampling
    stability, as already noted for kim2026divergent.

## Trust signals

- **Credibility:** 3.
  - For: Apple author. Careful statistics for a single-author paper:
    item bootstrap, stratified permutation null, eigenvalue check on the
    Kish exchangeability assumption, split-half validation of the
    Condorcet null (ratios 0.96–1.00), ten-bin sensitivity, subsampling
    convergence (stable by N ≈ 200–300), and a gold-tie robustness check.
    Public datasets, named models, a full prompt in the appendix. The
    limitations section names the main gaps (classification only, panel
    selection uncertainty).
  - Against: single author, no peer review, no released code or judge
    outputs, no citations yet. Commercial API models without version
    pins. The human baseline is misread (see Critique 1), and that
    misreading feeds the abstract's "three-quarters" sentence.
  - Not 4 because there is no code or released votes and the main
    interpretive claim leans on an unconditional statistic. Not 2
    because the measurements are internally consistent and the
    conditioned quantity (Condorcet gap) has proper controls.

## Follow-up

- **Relevance:** 4. It is material new evidence for
  [[concepts/shared-substrate-contagion]] on the independence axis: a
  difficulty-conditioned null, a scaling curve and a family breakdown
  for a cross-family panel with no channel. It also gives
  [[concepts/hce-evaluation]] a concrete diagnostic (n_eff/k from error
  vectors on labelled items) and a negative result on aggregation. Not
  5 because three of four tasks are NLI classification, nothing is a
  research-agent artifact, and the headline number does not separate
  difficulty from shared error.
- **Cheap repo check.** If any hub or downstream loop starts using a
  multi-model judge panel, first score it on a small labelled set and
  compute φ̄ and n_eff. Prefer one strong judge with evidence access over
  more judges of the same evidence.
- **Candidates** (cited, not yet in the graph):
  - Kim et al. 2025, "Correlated errors in large language models"
    (ICML 2025).
  - Verga et al. 2024, PoLL, "Replacing judges with juries"
    (arXiv 2404.18796).
  - Dorner et al. 2025, "Limits to scalable evaluation at the frontier:
    LLM as judge won't beat twice the data" (ICLR 2025).
  - Jiang et al. 2025, "Artificial hivemind" (NeurIPS 2025).
  - Turkmen et al. 2026, information-theoretic LLM ensemble selection
    (arXiv 2602.08003).
  - Zhao et al. 2025, CARE, confounder-aware judge aggregation.
