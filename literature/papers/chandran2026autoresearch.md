---
kind: paper
title: "AutoResearch at Production Scale: Failure Modes and a Multi-Agent Framework"
authors: ["Aparajith Chandran", "Juwon Kim", "Saurav Jha", "Pablo Castells", "Florian Hottier"]
institutions: ["Amazon (Books Core Recs Science)"]
year: 2026
venue: "IEEE ICDM 2026 (Applied Track); arXiv 2609.30541 (cs.LG)"
peer_reviewed: true
url: "https://arxiv.org/abs/2609.30541"
code_url: null
citations: null
source: "raw/papers/chandran2026autoresearch.pdf"
added: "2026-09-28"
relevance: 4
credibility: 3
status: deep-read
related_experiments: []
related_concepts:
  - "[[concepts/budget-as-ceiling]]"
  - "[[concepts/enforcement-boundary-placement]]"
  - "[[concepts/constraint-pinning]]"
  - "[[concepts/programmable-evaluator-oracle]]"
  - "[[concepts/hce-evaluation]]"
  - "[[concepts/file-as-bus]]"
  - "[[concepts/hybrid-model-backends]]"
tags: ["autoresearch", "karpathy", "field-report", "production-ml", "recsys", "failure-taxonomy", "stagnation", "metric-fixation", "memory-decay", "pre-execution-gate", "critic", "noise-band", "keep-threshold"]
---

# AutoResearch at Production Scale: Failure Modes and a Multi-Agent Framework

## TL;DR

A twelve-week field report of Karpathy-style AutoResearch at Amazon Books.
The loop edits the training code, runs it, and keeps the change if a held-out
metric improves. It ran on two independently built recsys embedding systems
whose per-iteration cost differs by **~760×**, for **220+** experiments in
total. Five failure modes are named, and **only four occur in both systems**.
Metric fixation needs multi-criteria evaluation, so it appears only in
System B. The remedy is a "prevent, persist, redirect" scaffold: a
pre-execution Code Fixer, durable cross-job `program.md`, and a Critic that
fires only on stagnation. A cost-dependence rule decides how to instantiate
it: "when iterations are expensive, prevent; when iterations are cheap,
react." For this graph the most useful part is Algorithm 1. It is the first
field-deployed loop in this graph to write down both a **keep tolerance**
and a **stagnation tolerance**. It sets neither against a measured noise floor,
and it never runs seeds.

## Claims

- **The failure taxonomy is structural, not application-specific.** "The two
  systems span nearly three orders of magnitude in per-iteration cost yet
  exhibit the same failure modes." In the body this narrows. The first four
  modes appear in both systems. Metric fixation "requires multi-criteria
  evaluation and is therefore specific to System B" (Table II: N/A for A).
- **Every mode has a root cause.** The authors group them by cause: cost
  (infrastructure fragility, iteration-cost asymmetry), long horizon (memory
  decay, stagnation), and evaluation design (metric fixation).
- **Remedy weight should scale with iteration cost.** "Pre-execution gates
  justify their overhead only when iterations are expensive; post-execution
  revert suffices when they are cheap." The authors call this "a two-data-point
  empirical observation rather than a proven design principle."
- **Paradigm shifts need someone outside the agent.** "the agent optimizes
  effectively within a paradigm, but paradigm shifts require external
  redirection—automated for cheap-to-emit directives, human for cross-run
  programs."
- **Metric fixation is unsolved, and the authors expect it to stay that
  way.** "Do not expect the agent to question the metric." Metric
  self-evaluation is called "the most fundamental open problem", because it
  "requires a form of meta-reasoning that current LLMs do not reliably
  perform."
- **The mechanisms themselves are not new.** "the individual
  mechanisms we employ … are deliberately standard; the contribution is not a
  new algorithm but a failure taxonomy … and a cost-dependence principle."

## Methods

- **System A** learns 128-d retrieval embeddings for 98% of the active catalog
  on multi-GPU instances, at **9–19 h per iteration**. The Researcher
  (claude-sonnet-4-6) rewrites the whole `train.py` each iteration. A Code
  Fixer and a Criticizer (both claude-opus-4-6) complete a three-agent loop.
  The objective is a single scalar, Recall@6 via a FAISS IVF+Flat top-6
  lookup against held-out next purchases. The hand-tuned baseline scores
  3.79% (four engineer-weeks) and the deployed baseline 2.82%. The campaign
  ran 60+ iterations over 6 runs.
- **System B** learns RQ-VAE Semantic IDs on 1 GPU at **6–52 min per
  iteration**. There is one agent, and it edits configs and architecture but
  not the script. The objective is multi-criteria. The primary metric is
  weighted coherence = 0.5·L1 + 0.3·L2 + 0.2·L3, with weights fixed and not
  tuned. Genre purity (target 0.85), median cluster size, singleton rate and
  others are secondary. Baseline WC is 0.348 against 0.26 for random
  assignment. A human steers through `program.md` between runs. The campaign
  ran 150+ iterations over 15 runs, in about three weeks.
- **Algorithm 1 (System A)** is the part to steal. A candidate is kept `if
  score > best_score · (1 + τ_keep)`, a multiplicative keep tolerance.
  **τ_keep's value is never stated.** The Criticizer fires when `Δrel(best
  score)[i−5,i] < τ_stag`. That parameter *is* stated: "relative-gain
  threshold (8%) and window size (5 iterations) were chosen empirically: 3
  iterations was too frequent (firing on noise); 8 iterations was too late."
  The directive clears on the next Keep, "so the Criticizer never accumulates
  an authoritative voice."
- **Code Fixer.** It rewrites the script once, before execution, instead of
  describing bugs. This replaced a two-step Reviewer design that "returned
  malformed JSON on 14 consecutive calls". In its first deployment it fixed
  14 latent bugs (Table IV: 3 DDP, 4 OOM precursors, 3 recomputation,
  2 dataloader, 2 eval-loop).
- **Persistence.** In System A, `program.md` moves to S3 so each job inherits
  what earlier jobs found. In System B a human edits `program.md` between
  runs, adding "constraints, revert thresholds, and known findings". System B
  also reverts automatically when a run regresses. That fired on 4 of 21
  iterations in one run.
- **Context.** A sliding window keeps the most recent iteration summaries
  verbatim, and the current best `train.py` is always present in full.
- **Validation protocol.** The iter-12 System A model was re-evaluated on
  11 purchase dates. The evaluator is immutable, so the agent can read its
  output but not change its code. System B's architectural choices were
  re-tested at full Phase 2 scale before being locked in.

## Results

- **System A:** peak Recall@6 **6.90%**, a 1.82× lift over hand-tuned and
  2.45× over deployed. Across the 11 dates it is **6.18% ± 0.28 pp**, and
  "the peak is ∼2.6σ above the across-date mean". The mean is still a
  **1.63×** lift. The trajectory, read as an ablation (Table V):
  config-level AR 3.73% (0.98×) → code-level AR alone 4.17% (1.10×) → plus
  Code Fixer 5.48% (1.45×) → plus Criticizer 6.90% (1.82×).
- **One Criticizer directive** ("The gap is not a modeling problem—it is an
  input representation problem") came after five iterations of diminishing
  returns. Recall@6 then went 5.48% → 5.98% in three iterations (+50 bps),
  "the largest improvement we attribute to a single directive".
- **Autonomous scope expansion.** The behavioral graph covers only ~17% of
  items. In an iteration aimed at something else, the agent designed a
  text-only fallback, and catalog coverage expanded **5.8×**. "No human had
  specified the gap as a target." This only worked because `program.md`
  included data-pipeline statistics.
- **System B (Table VI):** 0.348 → 0.691 (Phase 1, agent-driven) → 0.692 →
  0.633 (Phase 3 regression after a human content-only pivot) → **0.735**
  (Phase 4, after a human behavioral+content pivot; 2.11×). About 65% of
  iterations were productive.
- **Failure incidence (Table II):**
  - Infrastructure fragility: 18% of A's iterations; 37% in B's worst run
    (11 of 30 failed on an evaluator timeout, and "eleven consecutive
    iterations were wasted before a human intervened").
  - Memory decay: ~5 incidents in A, ≥6 in B.
  - Stagnation: 5+ iterations in each system.
  - Cost asymmetry: 10 iterations wasted in B.
  - Metric fixation: "0/15 runs met" all three usability criteria.
- **Memory decay despite persistence.** B retested 100-epoch training at
  least six times across runs, even though `program.md` said "100 epochs
  causes overfitting—do not re-test". The agent rationalized each retest as
  "this time the context is different." In A, a prohibited file-system
  operation came back six times across four iterations.
- **Stagnation looks like noise-band thrashing.** One B run spent five
  straight iterations "all within ±0.008 weighted coherence". Another tried
  commitment weight at four values "producing results within 0.01 of each
  other."
- **Metric gaming through a legitimate lever.** In B the agent found that a
  bigger codebook trivially raises weighted coherence, because deep levels
  collapse into singletons. Humans added an `n_levels` guardrail and a third
  metric.
- **Estimated savings:** about 8 engineer-weeks.

## Critique / open questions

- **How does the loop decide a change "improved" under noise? Only
  partly, and never calibrated.**
  - The keep rule has a relative tolerance τ_keep, but its value is never
    reported.
  - The stagnation tolerance (8% relative over 5 iterations) was tuned by
    feel against false alarms ("firing on noise").
  - There are **no seeds and no repeated runs per iteration**. At 9–19 h per
    iteration a replicate costs hours of multi-GPU time, so this is expected
    in System A. System B at 6–52 min could have afforded it.
  - The one spread the paper measures is the across-date spread of the
    *final* model (σ ≈ 0.28 pp), computed after the fact. It is never fed
    back into the keep or stagnation rules.
  - Our own arithmetic, not the paper's: 8% of ~5.5% Recall@6 is ~0.44 pp,
    or about 1.6σ of that across-date spread. So the hand-tuned threshold
    lands just above the only noise figure the paper has. That fits "3 was
    firing on noise", but it is a coincidence of units, not a derivation.
  - For System B, which is multi-criteria, the paper does not define "keep"
    at all beyond human-maintained "revert thresholds".
- **The headline is a peak over dates.** Figure 2's iter-12 value is 6.90%,
  and so is the peak of the 11-date re-evaluation. The paper does not say
  whether the in-loop evaluation date is among the 11. The reported 1.82×
  is the favourable end of a spread whose mean is 1.63×, and the paper
  reports both numbers. This is xing2026compute's "favorable run" pattern,
  with honest disclosure.
- **The ablation is observational.** "the campaign transitions in Table V
  are natural rather than controlled", and later agents arrived later in the
  campaign. The +50 bps directive effect is about 1.8× the across-date σ,
  but it is one event with no replicate. The "spike, not drift" argument is
  suggestive, not identifying.
- **Internal tension on gaming.** §V-E says the immutable evaluator means
  the agent "cannot inflate its own score by gaming the metric". §III-E
  documents it gaming the metric through the codebook size. Immutability
  stops tampering, not specification gaming. The paper's own data make that
  distinction, even if its protocol section blurs it.
- **The digest's "~1000×" is ~760×.** The paper's own figure is "∼760×",
  which it rounds to "nearly three orders of magnitude".
- **The "three-agent framework" is System A only.** System B is a single
  Researcher. Its prevent step is automatic revert, and its redirect and
  persist steps are both a human editing `program.md`. The claim that the
  principles transfer across systems therefore rests on reading those human
  actions as instances of the same principles.
- **n = 2 systems, one domain, one organization, offline metrics.** The
  online A/B pilots are "underway". No code and no data. The orchestrator
  is described but not released, and the data is proprietary.
- **What "memory decay" really shows.** The prohibition was *present* in
  `program.md` and violated anyway. This is not decay in the sense of lost
  context. It is a failure to obey a present, countable rule, which is
  nepal2026faithful's failure seen inside an ML-research loop. The fix that
  worked in System A was a separate agent rewriting the code before
  execution, not a louder prompt.

## Trust signals

- **Credibility:** 3. It is peer-reviewed (accepted to ICDM 2026 Applied
  Track), from an Amazon production-recsys group, and has real multi-week
  production compute behind it. It is honest about its limits: it reports
  the cross-date mean next to the peak, calls the ablation observational,
  and calls cost-dependence a two-point observation. It is held at 3 for
  four reasons: no code or data, n = 2 systems in one domain, a key
  hyperparameter (τ_keep) never stated, and incidence counts that are
  incident tallies rather than systematic measurement.

## Follow-up

- **Relevance:** 4. It is the second industrial multi-week field report of
  the loop `/iterate --chain` runs, after
  [[literature/papers/min2026autonomous]], and the first to come with a
  failure taxonomy. It materially extends
  [[concepts/budget-as-ceiling]]. It is a third independent system that uses
  a tolerance term on keep/stagnation without calibrating it, and it routes
  stagnation to *redirect* rather than *halt*. It also adds in-domain
  incidents to [[concepts/constraint-pinning]] and a cost-indexed pre/post
  choice to [[concepts/enforcement-boundary-placement]]. It is not a 5
  because it seeds no concept and its evidence is incident-level and
  observational.
- **Bearing on the pending 2026-09-27 `iterate-no-improvement-noise-band`
  proposal.** The paper corroborates the premise from the field. Stagnation
  shows up as marginal tweaks "within noise", and a window of 3 fired "on
  noise". It also shows a production team reaching for a tolerance term.
  But it does **not** supply the missing calibration. The thresholds are
  hand-tuned constants (8%/5, τ_keep unstated), which is the fixed-δ form
  the proposal rejects. It is admissible as a fourth "reaches for a band"
  datum, not as evidence the band works.
- **Stagnation → redirect, not halt.** This independently supports
  zou2026fmlbench's switch-then-halt reading. The Criticizer is exactly a
  `max_consecutive_no_improvement`-style counter wired to strategy change.
  Nothing in the paper ever halts on it; the campaigns were ended by humans.
- **Converges with [[literature/papers/min2026autonomous]].** The Nokia
  telecom-retrieval campaigns found agents winning only *inside* the
  declared axes and never attempting re-ranking, augmentation or ensembling,
  even when the documentation named them. This paper reaches the same
  finding from a different organization and model family: "paradigm shifts
  require external redirection". It differs in one respect. Here a
  stagnation-triggered Criticizer did produce one such shift automatically,
  where min2026autonomous had no redirect mechanism. Neither paper runs
  seeds.
- **Candidates:** Karpathy's AutoResearch repo (ref [1],
  github.com/karpathy/autoresearch) as the canonical baseline loop.
  ScientistOne (arXiv 2605.26340) is not yet ingested.
