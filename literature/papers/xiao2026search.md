---
kind: paper
title: "Search Shapes Conclusions: Auditing Evidence Selection Bias in Deep Research Agents"
authors: ["Shuyao Xiao", "Shengling Wang", "Xuan Chen", "Ke Chao", "Ming Cui", "Feifei Qian", "Chaoyang Mei", "Fanlin Meng", "Lulu Wang", "Ziming Yu", "Junxi Yin"]
institutions: ["Beijing Normal University", "Ke Holdings"]
year: 2026
venue: "arXiv 2609.39026v1 (cs.AI), 2026-09-30; header reads 'Preprint'"
peer_reviewed: false
url: "https://arxiv.org/abs/2609.39026"
code_url: null
citations: null
source: "raw/papers/xiao2026search.pdf"
added: "2026-10-06"
relevance: 3
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/web-grounded-literature]]"
  - "[[concepts/citation-anchoring]]"
  - "[[concepts/hce-evaluation]]"
tags: ["deep-research-agents", "evidence-selection-bias", "adaptive-sampling", "inverse-propensity", "doubly-robust", "shrinkage", "partial-identification", "candidate-pool", "ranking-sensitivity", "correction-vs-attribution", "paired-intervention", "ms2", "perspectrum", "open-deep-research"]
---

# Search Shapes Conclusions: Auditing Evidence Selection Bias in Deep Research Agents

## TL;DR

A Deep Research agent reads an **adaptive** sample of the documents
available to it. Earlier hits steer later queries, picks and stopping, so
the average evidence direction of what it opened (the "Opened Mean") can
differ from the average over a prespecified **candidate pool** (θ*). This
can happen even when every citation is correct.

**CESS** (Causal Evidence Selection Correction) estimates θ* from one
trajectory. It is a standard augmented-IPW / doubly-robust estimator:
- an outcome model predicts every candidate's evidence score;
- each opened document's residual is reweighted by its **logged**
  selection probability p_t and its probability s_t of reaching that
  round;
- the correction is shrunk toward the full-pool prediction (λ) and
  clipped to [−1, 1];
- when some documents have zero selection probability, a sharp interval
  of width 2ρ replaces the point estimate, where ρ is the unreachable
  share of the pool.

A second, mostly conceptual result (Theorem 2): any estimator that recovers
θ* under every policy has an expected contrast of zero between policies. So
it **cannot** measure what a change of search policy did to the evidence
opened. That question needs paired interventions.

For this graph, the useful part is the framing plus one formal bound. The
method itself does not transfer to `/digest`. It needs a stochastic
controller that logs a positive probability for every document in a fixed,
enumerated pool, and our pipeline (and most real agents) ranks
deterministically over the open web. The headline numbers are the
most favourable cut: a 2-document budget, in a semi-synthetic harness, with
adversarial document orderings.

## Claims

- **Citation correctness is not sample representativeness.** A
  well-cited report "can still reach a misleading conclusion"; the
  existing checks on citation support, factuality and utility "may appear
  satisfactory even when the agent has assembled a one-sided evidence
  set."
- **Three distinct estimands.** θ* is what the pool supports. O(g) is what a
  typical run of policy g concludes from what it opened. τ_O(g, g′) is how
  swapping policies changes that. CESS targets only θ*.
- **Design unbiasedness (Theorem 1).** With logged probabilities and
  overlap, the raw estimator is unbiased for θ* even if the outcome model
  is misspecified. This is the textbook AIPW result applied to a
  sequential design, and the authors cite Cassel 1976 and Robins 1994.
- **Correction is not attribution (Theorem 2).** "a contrast of
  policy-invariant corrected targets identifies τ_O(g,g′) only when
  τ_O(g,g′)=0." The proof is linearity of expectation.
- **Abstract headline:** on MS2, CESS cuts MAE against θ* by 9.2% and
  ranking sensitivity by 39.4% relative to the Opened Mean. On a public
  Open Deep Research agent the cuts are 60.1% and 87.2%.

## Methods

- **Evidence scores come from dataset labels, not from the agent.**
  - MS2 (medical systematic reviews): Y_i = p⁺ − p⁻ from supplied
    significance probabilities.
  - PERSPECTRUM (claims): the mean of ±1 stance labels.
  - θ* is the unweighted mean over the pool.
  - The agent's written report is never scored. The "conclusion" in the
    title is the label average over the documents it opened.
- **Ranking sensitivity.** The absolute change in an estimate when the same
  pool is ordered supporting-first vs opposing-first. This is a
  deliberately adversarial ordering intervention.
- **MS2 main run.** 200 test questions with 3,169 candidate studies.
  Qwen2.5-32B-Instruct and OLMo3-7B generate the queries. Each run does 4
  rounds under 3 orderings, giving 1,200 trajectories. Shrinkage is
  tuned on separate questions (λ = 0.5484 for Qwen, 0.5649 for OLMo).
  CIs resample whole questions.
- **Public-agent transfer.** Open Deep Research, at a pinned revision,
  with Qwen2.5-32B.
  - The authors replace only its search tool, with a stochastic
    Gumbel-draw sampler over a fixed PERSPECTRUM pool that logs every
    probability.
  - 36 tasks × 2 orderings × 3 seeds gives 216 trajectories (108 pairs).
  - The primary analysis uses only the first **K = 2** selections.
- **Paired 2×2 intervention.** Agent vs uniform document selection, crossed
  with agent-controlled vs fixed 10-round stopping. 200 MS2 questions × 2
  models × 4 conditions × 3 repeats gives 4,800 trajectories.
- **Appendix runs.**
  - Five-round PERSPECTRUM (36 topics, λ = 0.20 anchored on the Opened
    Mean).
  - Ten-round PERSPECTRUM (267 tuning and 36 test questions).
  - Controlled simulations for identification, limited support and
    perturbations.

## Results

- **Controlled simulation (Fig 2a).** The *unshrunk* correction with
  *exact* probabilities cuts the gap between orderings from 0.7478 to
  0.0014 (99.8%). That is Theorem 1 working as designed.
- **Table 5.** Removing the selection weight under biased selection
  leaves bias of 0.080–0.088. With full correction it is ≤ 0.010.
- **MS2 (Table 1, averaged over both models).**

  | Method | MAE | Rank. sens. | Worst-ranking MAE |
  |---|---|---|---|
  | Tuning-set Mean | 0.2733 | 0 | 0.2733 |
  | Outcome Regression | 0.2571 | 0 | 0.2571 |
  | Opened Mean | 0.1743 | 0.2445 | 0.2522 |
  | Sequential DR (unshrunk) | 0.1893 | 0.2626 | 0.2763 |
  | CESS | 0.1583 | 0.1481 | 0.2165 |

  Without shrinkage the DR correction is *worse* than the naive mean.
  The main text gives no CI for the 9.2% MAE gain.
- **Gains shrink with budget (Fig 3, MS2, Task-wise CESS vs Opened
  Mean).** The MAE reduction is 41.5% → 23.4% → 15.8% → 9.1% over rounds
  1–4 for Qwen, and 40.7% → 15.8% for OLMo.
- **Matched-accuracy check (Table 2 / Table 11, 28 settings).**
  - At matched MAE, CESS has lower ranking sensitivity in **10**
    settings, no detected difference in 18, and higher in 0.
  - The reciprocal comparison is only in the appendix. At matched
    sensitivity, CESS has **higher** MAE in **10** settings and lower in
    **3**.
- **Public agent, K = 2 (Table 3).**

  | Method | MAE | Rank. sens. | Direction error |
  |---|---|---|---|
  | Opened Mean | 0.5727 | 0.5849 | 0.4907 |
  | Outcome Regression | 0.2404 | 0 | 0.3333 |
  | Sequential DR | 0.3053 | 0.1492 | 0.3287 |
  | CESS | 0.2287 | 0.0746 | 0.3102 |

  - The paired MAE reduction is 0.344 (95% CI 0.261–0.424).
  - The median effective sample size is **1.70** at K = 2 and 2.81 at
    K = 4.
  - Against outcome-regression shrinkage at matched MAE, the K = 2
    sensitivity difference CI includes zero. At K = 4, CESS has 0.046
    lower MAE at matched sensitivity (CI excludes zero).
  - By round 4, 63.9% of paired branches issue different queries and
    75.0% select different documents.
- **Five-round PERSPECTRUM (Table 6).** MAE −10.2% (paired CI excludes
  zero), ranking sensitivity −15.8%, direction-error rate **+0.23 pp**
  (CI includes zero).
- **Ten-round PERSPECTRUM (Tables 8–9).**

  | Method | MAE | S | Direction disagreement |
  |---|---|---|---|
  | Opened Mean | 0.4108 | 0.9574 | 0.4186 |
  | Tuning-set Mean | 0.2787 | 0 | 0.3889 |
  | Opened Mean shrunk to tuning mean | **0.2427** | 0.2393 | 0.3893 |
  | CESS | 0.3103 | 0.1608 | 0.5208 |

  - Dropping the selection weights *lowers* MAE (0.3071 vs 0.3103) but
    nearly triples S (0.4346 vs 0.1608).
  - Dropping the stopping correction changes MAE by −0.0042 (CI
    includes zero).
- **Correction vs attribution (Table 4).**
  - The directly measured selection effect is moderately repeatable
    (ICC 0.679 for OLMo, 0.685 for Qwen).
  - The *stopping* effect is not (ICC 0.099 and 0.102).
  - The rank correlation between CESS contrasts and the selection
    effect is 0.269 and 0.236 with adaptive queries, and −0.027 and 0.086
    with fixed queries.

## Critique / open questions

- **The digest-framing pattern holds here: 32 of 32.** The abstract's
  60.1% / 87.2% comes from the friendliest setting in the paper:
  - **K = 2** documents per trajectory, with a median ESS of 1.70;
  - an adversarial supporting-first vs opposing-first ordering, which
    makes the 2-document Opened Mean nearly a coin flip (direction error
    0.49);
  - a search tool replaced by a logged stochastic sampler.

  The main real-review result is **9.2%** (MS2, 4 rounds), and Fig 3
  shows gains decaying toward that as budget grows. The 18.1 pp
  direction-error gain appears only in this K = 2 cut. On five-round
  PERSPECTRUM it is +0.23 pp, and on ten-round PERSPECTRUM CESS is
  *worse* than the Opened Mean on direction (0.52 vs 0.42).
- **Simple baselines win in some settings, and the main text does not
  say so.**
  - On ten-round PERSPECTRUM, "Opened Mean shrunk to the tuning mean"
    (MAE 0.2427) and even a constant Tuning-set Mean (0.2787) beat CESS
    (0.3103).
  - On the K = 2 public agent, CESS beats Outcome Regression, which
    never reads a document, by only 0.012 MAE (0.2287 vs 0.2404). The
    paper reports no CI for that pair.
  - Main-text Table 2 shows only the favourable matched comparison
    (10–0). Appendix Table 11's reciprocal comparison runs 3–10 against
    CESS on accuracy.

  The fair summary is that CESS buys order-stability, sometimes at an
  accuracy cost. It does not dominate.
- **The "conclusion" is never the agent's conclusion.** The scores are
  dataset stance and significance labels averaged over opened documents.
  The paper does not test whether the Opened Mean predicts the direction
  of the synthesized report, so the title's "Search shapes conclusions"
  is assumed, not measured.
- **The paper does not show bias magnitude in natural use.** Every
  large Opened-Mean error is produced by an imposed adversarial
  ordering. There is no measurement of how one-sided a real agent's
  evidence set is under its native ranking. "Early findings steer
  later queries, picks and stopping" is the paper's causal model and
  motivation. Of the three, only selection has a repeatable measured
  effect. The stopping effect has ICC ≈ 0.10, which is not reliably
  measurable.
- **The method needs conditions real agents lack.**
  - Every document needs a known, positive selection probability, and
    the controller must log it.
  - The candidate pool must be fixed and enumerated in advance.
  - Real search APIs are deterministic top-k over an unenumerated web,
    so ρ ≈ 1 and the Eq. 10 interval is uninformative. In the paper's
    own limited-support test (3.6% of the pool reachable), the interval
    covers all 16,000 targets, which is trivially true when it is about
    as wide as the score range.
  - The public-agent "transfer" works only because the authors swapped
    in a sampler over a closed pool.
- **Theorem 2 is near-definitional.** If an estimator returns θ* under
  every policy, its policy contrast is zero by linearity. The
  4,800-trajectory experiment confirms a one-line identity. The
  distinction is still worth having (see Follow-up).
- **Inconsistent λ reporting.** §3.3 gives MS2 λ ≈ 0.55. Appendix A.5
  says the coefficient chosen on tuning questions is λ = 0.01, at which
  "removing selection or stopping probabilities changes MAE by only
  about 10⁻⁴". The simulations in Table 12 use λ = 0.05. It is unclear
  which experiments ran at λ ≈ 0, where CESS is essentially the
  outcome-model prediction. If any did, the probability correction did
  almost nothing there.
- **A metric change is disclosed.** Ranking sensitivity S was redefined
  between audits (CESS 0.2925 → 0.1608 from the metric change alone).
  This is flagged honestly, but it is a post-hoc metric choice.
- **Reproducibility is partial.** The text refers to a "released
  implementation", but no URL is given. The Open Deep Research revision
  hash is pinned, and seeds are given for the bootstrap.

## Trust signals

- **Credibility:** 3.
  - For: a university and industry team (BNU, Ke Holdings); a correct,
    standard estimator with proofs; question-level bootstrap CIs; tuning
    and test separation; and candid appendix disclosure (the reciprocal
    matched table, the metric change, the clip-order ablation).
  - Against: no peer review, no code URL, and an abstract that leads
    with the most favourable cut. Simple-shrinkage baselines beat the
    method in one evaluation, and that is not surfaced in the main text.
  - Its statistics are far stronger than a typical pilot (cf.
    park2026when).

## Follow-up

- **Relevance:** 3. The digest's "4" was set by its *Why* line, which
  reads more into the paper than the PDF supports.
  - Transferable: the target / attribution / opened-mean distinction,
    and the unreachable-mass bound.
  - Not transferable: CESS itself, which needs logged propensities over
    an enumerated pool. Our pipeline has neither.
  - The gains are measured on label averages, not on reports.

  It sharpens [[concepts/web-grounded-literature]] but supplies no
  evidence about our retrieval.
- **On the digest's "strongest argument yet for the complete OAI-PMH
  harvest".** The paper never discusses harvesting. The argument is ours,
  and it rests on Eq. 10:
  - With deterministic ranked queries, every document the queries can't
    surface has zero probability. The pool estimate is then bounded only
    to an interval of width 2ρ, however careful the reading.
  - Only enlarging the reachable set shrinks ρ. A complete category
    harvest makes the pool enumerable and every record reachable at the
    triage stage.
  - The bias then moves to the regex-then-read funnel inside the
    harvest. That funnel is a deterministic selection whose rule *is*
    logged (the regex), so the selection is fully known. It still has
    zero probability on unmatched records.

  So the paper supports the harvest *qualitatively*: coverage beats
  correction when propensities are unknown. It does not support it
  quantitatively.
- **Cheap transferable practice.**
  - Log each candidate's selection reason (query or regex hit) and record
    the size of the pool it was drawn from. That is the minimal
    "logged design" an audit needs, and it extends the
    `web-grounded-literature` "log the query, not just the hit" item.
  - To ask whether `/digest` is one-sided, audit the funnel by
    intervention: re-run triage with the regex relaxed or the ranking
    reversed, and compare what gets read. Theorem 2 says a corrected
    estimate cannot answer that question.
- **"Abstracts overstate" is a different bias.** It is reporting and
  selection by the *author* (headline cut), not evidence selection by the
  *agent*. The digest entry's link between the two is loose.
- **Candidates:**
  - Zhou et al. 2026, "Explore before committing: Hypothesis-guided search
    for deep research agents" (arXiv 2609.01294). The paper cites it for
    early search directions reinforcing themselves.
  - Huang et al. 2026, DeepFact (ACL 2026), on cited reports conflicting
    with broader evidence.
